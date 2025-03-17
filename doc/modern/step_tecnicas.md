# Etapas técnicas

Nesta página, são apresentadas as etapas técnicas fundamentais para o início do processo de migração para Kubernetes. A transição para um ambiente orquestrado exige a configuração adequada de segurança, controle de acesso, monitoramento e proteção contra ameaças. Aqui, destacamos os principais serviços da AWS que auxiliam na implementação de um ambiente seguro e eficiente, garantindo que a migração ocorra de forma estruturada e com as melhores práticas de segurança em nuvem.

### 1. **Preparar o Cluster Kubernetes**

- **Criar um Cluster EKS**:
Utilize o **Amazon EKS (Elastic Kubernetes Service)** para gerenciar o cluster Kubernetes. O EKS facilita a criação, a configuração e o gerenciamento do Kubernetes.
    - Acesse o **AWS Management Console**.
    - Navegue até **EKS** e crie um novo cluster.
    - Selecione a versão do Kubernetes, região, VPC, e sub-redes apropriadas.
    - Garanta que as instâncias EC2 para o Kubernetes Worker Nodes estejam em **Auto Scaling** e com a configuração adequada para os seus requisitos de carga.
    
    **Exemplo de comando para criar o cluster via CLI**:
    

```bash
aws eks create-cluster \
  --name my-cluster \
  --role-arn arn:aws:iam::<account-id>:role/<eks-role> \
  --resources-vpc-config subnetIds=<subnet-ids>,securityGroupIds=<sg-id>

```

### 2. **Implantar o Projeto Backend no Kubernetes**

- **Containerizar o Projeto Backend**:
    - Primeiro, crie uma **imagem Docker** do seu backend (API ou serviço). Use um `Dockerfile` e garanta que a imagem esteja pronta para rodar.
    - **Suba a imagem para o Amazon ECR** (Elastic Container Registry) para facilitar a gestão das imagens Docker.
    
    **Exemplo de comando para criar um repositório no ECR**:
    

```bash
aws ecr create-repository --repository-name my-backend
```

**Subir a imagem Docker para o ECR**:

```bash
docker build -t my-backend .
docker tag my-backend:latest <aws_account_id>.dkr.ecr.<region>.amazonaws.com/my-backend:latest
$(aws ecr get-login --no-include-email --region <region>)
docker push <aws_account_id>.dkr.ecr.<region>.amazonaws.com/my-backend:latest

```

**Criar os Manifests do Kubernetes**:

- Crie arquivos de manifesto Kubernetes para o backend, como `deployment.yaml` e `service.yaml`.
- Um exemplo básico de **Deployment** para o seu backend:

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-backend-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-backend
  template:
    metadata:
      labels:
        app: my-backend
    spec:
      containers:
      - name: my-backend
        image: <aws_account_id>.dkr.ecr.<region>.amazonaws.com/my-backend:latest
        ports:
        - containerPort: 8080

```

**Criar o Serviço Kubernetes (Service)**:

```bash
apiVersion: v1
kind: Service
metadata:
  name: my-backend-service
spec:
  selector:
    app: my-backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8080
  type: LoadBalancer

```

**Aplicar os Manifests no EKS**:
Após a criação dos arquivos YAML, aplique-os no EKS com o comando:

```bash
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml

```

### 3. **Implantar o Projeto Frontend no Kubernetes**

- **Containerizar o Projeto Frontend**:
    - Da mesma forma que o backend, crie uma imagem Docker do seu frontend, com um `Dockerfile` adequado para a aplicação, e faça o upload para o ECR.
- **Criar os Manifests Kubernetes**:
    - Crie o `deployment.yaml` e `service.yaml` para o frontend, alterando os detalhes de imagem para corresponder ao seu repositório no ECR.
    
    **Exemplo de Deployment para o frontend**:
    

```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-frontend-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: my-frontend
  template:
    metadata:
      labels:
        app: my-frontend
    spec:
      containers:
      - name: my-frontend
        image: <aws_account_id>.dkr.ecr.<region>.amazonaws.com/my-frontend:latest
        ports:
        - containerPort: 80

```

**Aplicar os Manifests no EKS**:

```bash
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml

```

### 4. **Configurar Armazenamento Estático no S3 para o Frontend**

- **Configuração do Frontend**:
Configure seu frontend para consumir os arquivos estáticos diretamente do **Amazon S3**. Isso pode ser feito alterando as URLs de recursos estáticos (imagens, vídeos, arquivos CSS/JS) para os links públicos do S3.
- **Configuração de CORS no S3**:
Certifique-se de habilitar **CORS** no bucket do S3 para que o frontend possa fazer requisições corretamente.
    
    **Exemplo de política CORS no S3**:
    

```bash
[
  {
    "AllowedOrigins": ["*"],
    "AllowedMethods": ["GET"],
    "AllowedHeaders": ["*"],
    "MaxAgeSeconds": 3000
  }
]

```

### 5. **Configurar Banco de Dados no RDS**

O **Amazon RDS MySQL Multi-AZ** já está configurado, então você precisa apenas garantir que suas instâncias de backend no Kubernetes possam se conectar ao banco de dados:

- **Configuração de Conexão**:
Adicione variáveis de ambiente no **Deployment** do backend para configurar a string de conexão com o **Amazon RDS MySQL**.
    
    **Exemplo de variável de ambiente no Kubernetes**:
    

```yaml
env:
- name: DB_HOST
  value: <rds-endpoint>.amazonaws.com
- name: DB_PORT
  value: "3306"
- name: DB_USER
  value: <db-user>
- name: DB_PASSWORD
  valueFrom:
    secretKeyRef:
      name: db-secret
      key: password

```

- Use o **AWS Secrets Manager** para armazenar credenciais sensíveis e garanta que as variáveis de ambiente do Kubernetes possam acessá-las de forma segura.

### 6. **Segurança e Configurações Adicionais**

- **AWS IAM e Kubernetes**:
Configure o IAM para garantir que o EKS tenha permissões apropriadas para acessar os serviços da AWS (S3, RDS, etc.).
- **AWS WAF e Security Groups**:
Certifique-se de que o AWS WAF esteja configurado para proteger o tráfego de entrada e configure os **Security Groups** para restringir o acesso às instâncias EC2 conforme necessário.

### 7. **Escalabilidade e Monitoramento**

- **Escalabilidade**:
Configure **Horizontal Pod Autoscaling** no Kubernetes para garantir que os pods escalem conforme necessário.
- **Monitoramento**:
Utilize o **Amazon CloudWatch** para monitorar logs e métricas do Kubernetes, bem como o **AWS CloudTrail** para auditoria de API.

### Conclusão

Agora o projeto deve estar pronto para rodar no Kubernetes com o EKS, aproveitando os serviços AWS que você já configurou. A chave é garantir que todas as dependências do serviço (como S3, RDS, IAM e WAF) sejam integradas corretamente no seu ambiente Kubernetes.

## Navegação

### 🚀 Modernização
- [Kubernetes Moderno](doc/modern/modern_k8s.md)
- [Setup e Técnicas](doc/modern/step_tecnicas.md)

### 🔄 Migração
- [Visão Geral](doc/migration/migration_overview.md)
- [Migração de Banco de Dados](doc/migration/migration_bd.md)
- [Migração para EC2](doc/migration/migration_ec2.md)
- [Migração de Arquitetura Estática para ARM](doc/migration/static_arm.md)

### 🔒 Segurança
- [Segurança na AWS](doc/security_aws.md)

### ❓ FAQ
- [Perguntas Frequentes](doc/faq.md)
- [Fluxo de Funcionamento](doc/flow.md)

### 📚 Documentação Completa
- [Ver Documento Completo](doc/full_doc.md)


