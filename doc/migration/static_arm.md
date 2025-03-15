# Armazenamento de Arquivos Estáticos no Amazon S3

### O porquê da escolha?

Escolhemos o **Amazon S3** em vez do **Amazon EBS** porque o S3 é ideal para armazenar e compartilhar arquivos de forma simples, segura e econômica. Ele é como um "armazenamento na nuvem" onde você pode guardar documentos, imagens, vídeos, backups, entre outros, sem se preocupar com espaço físico ou manutenção.

O **EBS**, por outro lado, é mais usado para "armazenar dados de disco" dentro de servidores, como se fosse um HD de computador, e é mais adequado para sistemas que precisam de armazenamento rápido e constante para funcionar. No entanto, o custo e a complexidade são mais altos, e ele não é tão flexível quando se trata de armazenar arquivos em quantidade de forma eficiente.

Portanto, o **S3** oferece uma solução mais econômica, escalável e fácil de gerenciar para armazenar e acessar os arquivos que você precisa de qualquer lugar, sem se preocupar com o desempenho do servidor.

| **Serviço** | **Custo de Armazenamento** | **Escalabilidade** | **Custo de Transferência de Dados** |
| --- | --- | --- | --- |
| **S3** | Muito mais barato (variável conforme a classe de armazenamento) | Escalabilidade automática | Custo de saída de dados para fora da AWS |
| **EBS** | Mais caro (dependendo do tipo de volume) | Escalabilidade limitada (precisa aumentar manualmente) | Custo de saída de dados para fora da AWS |

### **Qual usar S3 ou EBS?**

- **Use S3 se:**
    - Necessidade de **armazenamento de dados de longo prazo** (backup, arquivamento).
    - Está lidando com **grandes volumes de dados** que não exigem acesso rápido.
    - Deseja **custos mais baixos**.
    - Seu uso envolve arquivos como **imagens, vídeos, logs** ou dados compartilhados por múltiplas instâncias.
- **Use EBS se:**
    - Necessitar de **alto desempenho e baixa latência** para acesso constante aos dados (como em bancos de dados).
    - Precisar de **acesso contínuo** a arquivos de sistema ou dados de aplicativos específicos em **instâncias EC2**.
    - O **desempenho** (com IOPS e throughput elevados) é uma prioridade.

### Exemplos de preços aproximados (2024):

- **S3 Standard**: cerca de **$0.023 por GB/mês**
- **S3 Glacier** (para arquivamento a longo prazo): cerca de **$0.004 por GB/mês**
- **EBS (SSD)**: cerca de **$0.08 por GB/mês** para volumes GP2
- **EBS (HDD)**: cerca de **$0.045 por GB/mês** para volumes ST1

### Custos entre S3 e EBS

- **S3** é geralmente **mais barato** para armazenamento de grandes volumes de dados e para **arquivos estáticos**. Ele é ideal para **backup, arquivamento**, e **arquivos não estruturados** (como imagens, vídeos, logs, etc.). Além disso, o S3 oferece diferentes classes de armazenamento com diferentes faixas de preço (como S3 Standard, S3 Infrequent Access, S3 Glacier, etc.), o que pode ajudar a reduzir ainda mais os custos, dependendo da necessidade de acesso aos dados.
- **EBS**, por outro lado, é **mais caro** e é voltado para **armazenamento de blocos** de dados, como volumes de discos rígidos usados por **instâncias EC2** para sistemas de arquivos. EBS é melhor para **aplicações que exigem acesso rápido aos dados** (como bancos de dados e sistemas de arquivos de instâncias). Se você usa volumes EBS, o custo aumenta com base no **tipo de volume** (SSD vs HDD), tamanho do volume e IOPS provisionadas.

## Implementação

1. **Criar um Bucket no S3**
    - No **AWS S3 Console**, crie um bucket para armazenar os arquivos estáticos.
    - Configure permissões adequadas com **AWS IAM** e **AWS WAF**.
2. **Modificar a Configuração do Nginx**
    - Atualize a configuração para que o Nginx use os arquivos do **Amazon S3**.
    - Se necessário, utilize o **AWS CloudFront** para caching e melhoria de desempenho.

## Passo a passo da implementação técnico

### **1. Criar um Bucket no S3**

**Objetivo:** Criar um bucket para armazenar os arquivos estáticos que serão acessados pelo Nginx.

### Passos:

1. **Acesse o AWS S3 Console**:
    - Vá para [AWS S3 Console](https://console.aws.amazon.com/s3/).
2. **Criação do Bucket**:
    - Clique em **Create bucket**.
    - **Bucket name**: Escolha um nome exclusivo para o seu bucket.
    - **Region**: Selecione a região mais próxima de onde os usuários ou servidores acessarão os dados.
3. **Configurações de Permissões**:
    - **Public Access Settings**: Desmarque a opção de bloqueio de acesso público, se os arquivos precisam ser acessados publicamente.
        - Configure a política de acesso conforme necessário (por exemplo, para permitir o acesso somente a usuários específicos ou para tornar os arquivos públicos).
    - **Bucket Policy (opcional)**: Se necessário, crie uma política para restringir o acesso ao bucket, como apenas permitir acesso de certos IPs ou apenas de determinadas contas IAM.
4. **Definir a Política do Bucket**:
    - Se você precisar de acesso restrito, defina políticas de IAM, garantindo que apenas os serviços e usuários com as permissões adequadas possam acessar o bucket.
5. **Configuração de Versionamento (opcional)**:
    - Se desejar manter versões dos arquivos, habilite o **versionamento** do bucket.
6. **Finalizar a Criação**:
    - Clique em **Create** para criar o bucket.

### **2. Configurar Permissões com IAM e WAF**

### **Permissões com IAM**:

1. **Criar uma Role IAM**:
    - Acesse o **IAM Console** no AWS.
    - Crie uma **Role IAM** que permita o acesso ao bucket S3 e conceda permissões adequadas de leitura (e gravação, se necessário) para o serviço Nginx.
        - **Policy de exemplo** para a role de leitura no S3:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::nome-do-bucket/*"
    }
  ]
}
```

1. **Anexar a Role ao Nginx**:
    - Se você estiver executando o Nginx em uma **instância EC2**, anexe a Role IAM criada à instância EC2.

### **Configuração do AWS WAF (Web Application Firewall)**:

1. **Acesse o Console do WAF**:
    - Vá para o **AWS WAF & Shield** no console AWS.
2. **Criar uma Web ACL**:
    - Crie uma **Web ACL** para proteger o bucket S3 ou a distribuição CloudFront, dependendo da sua necessidade de segurança.
    - Configure regras para bloquear tráfego indesejado ou proteger contra ataques (como SQL Injection, XSS, etc.).

### **4. Usar AWS CloudFront para Melhorar o Desempenho (opcional, mas recomendado)**

**Objetivo:** Configurar o CloudFront para distribuir e armazenar em cache os arquivos do S3 para melhorar o desempenho e reduzir a latência.

### Passos:

1. **Acesse o Console do CloudFront**:
    - Vá para o **Console do AWS CloudFront**.
2. **Criar uma Distribuição do CloudFront**:
    - Crie uma nova distribuição e selecione o **S3** como origem.
    - Selecione o bucket S3 que você criou como a **origem** e configure as opções de cache conforme necessário.
3. **Configurações de Distribuição**:
    - **CNAMEs**: Se desejar usar um nome de domínio personalizado para o CloudFront (por exemplo, `cdn.exemplo.com`), configure o CNAME apropriado e adicione um registro DNS.
    - **Configurações de Cache**: Escolha o tempo de cache desejado, o que ajudará a melhorar o desempenho ao reduzir a quantidade de requisições feitas diretamente ao S3.
4. **Ajustar o Nginx para Usar o CloudFront**:
    - Modifique o arquivo de configuração do Nginx para apontar para a URL da distribuição do CloudFront, em vez de acessar diretamente o S3.

```json
location /static/ {
    proxy_pass https://d123abc4xyz.cloudfront.net/;
    proxy_set_header Host d123abc4xyz.cloudfront.net;
}
```

1. **Testar o CloudFront**:
- Verifique se a distribuição do CloudFront está funcionando acessando corretamente os arquivos via a URL da distribuição.

### **5. Monitoramento e Manutenção**

### **Monitoramento**:

1. **Monitoramento com AWS CloudWatch**:
    - Ative métricas do **CloudWatch** para monitorar o uso do S3 (por exemplo, taxa de transferência, armazenamento utilizado).
    - Configure alarmes para monitorar o tráfego de CloudFront e garantir que os arquivos estão sendo entregues corretamente.

### **Backups e Versionamento**:

1. **Configurar Backups e Versionamento no S3**:
    - Se necessário, habilite o versionamento no bucket S3 para garantir que os arquivos possam ser restaurados em caso de falhas.
    - Configure regras de **lifecycle** para mover arquivos antigos para classes de armazenamento mais baratas (por exemplo, S3 Glacier).

### **Conclusão**

Esses são os passos detalhados para migrar arquivos para o S3, configurar o Nginx para servir esses arquivos e usar o CloudFront para melhorar o desempenho. 
Assim, ajustando as permissões e configurando corretamente as políticas de segurança para garantir o acesso apropriado aos arquivos.