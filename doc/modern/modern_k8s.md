# MODERNIZAÇÃO: KUBERNETES

A migração do projeto para o Kubernetes na AWS envolve a adaptação e implantação dos serviços já configurados para rodar em um ambiente **Kubernetes (EKS)**.  Explicando as etapas:

### 1. **Cluster EKS (Elastic Kubernetes Service)**:

- **EKS** gerencia o Kubernetes na AWS. A migração começa com a criação de um cluster EKS, que irá abrigar suas aplicações (frontend e backend) e orquestrar os contêineres de maneira eficiente.

### 2. **Containerização dos Serviços**:

- O **backend** e o **frontend** são convertidos em **imagens Docker** e armazenadas no **Amazon ECR (Elastic Container Registry)**, um repositório seguro para suas imagens.

### 3. **Implantação de Contêineres com Kubernetes**:

- Para cada serviço (backend e frontend), criamos arquivos de **Deployment** e **Service** no Kubernetes. O Deployment define como o serviço será escalado e o Service expõe o serviço para comunicação com outros componentes ou externamente (via LoadBalancer).

### 4. **Armazenamento de Arquivos Estáticos no S3**:

- Os arquivos estáticos do frontend são movidos para o **Amazon S3**, um serviço de armazenamento de objetos. O frontend é configurado para consumir esses arquivos diretamente de lá, melhorando a performance e reduzindo os custos de infraestrutura.

### 5. **Banco de Dados no RDS MySQL**:

- O **Amazon RDS MySQL Multi-AZ** foi configurado para garantir alta disponibilidade do banco de dados. O backend no Kubernetes será configurado para se conectar ao RDS via variáveis de ambiente, garantindo que a aplicação continue a acessar o banco de dados sem problemas.

### 6. **Segurança**:

- Utilização de **IAM** para controlar permissões de acesso aos serviços AWS.
- **AWS WAF** para proteger contra ataques.
- **Security Groups** para segurança de conexões.

### 7. **Escalabilidade e Monitoramento**:

- O Kubernetes facilita a **escalabilidade** dos seus serviços por meio de **Horizontal Pod Autoscaling**.
- **CloudWatch** é usado para monitorar logs e métricas, garantindo que o ambiente esteja funcionando de maneira eficiente e segura.



Neste link, encontra-se as etapas técnicas necessárias para time de DevSecOps, auxiliando na implementação
🔗 [Etapas Técnicas](step_tecnicas.md) 

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

### 📚 Documentação Completa
- [Ver Documento Completo](doc/full_doc.md)
