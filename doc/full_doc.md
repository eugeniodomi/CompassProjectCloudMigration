![banner](https://vetores.org/d/compass-uol.svg)
# Introdução

Este projeto representa a entrega final do estágio *Scholarship* na Compass UOL. Durante o programa, aplicamos conceitos e práticas fundamentais em tecnologia, segurança e desenvolvimento, consolidando conhecimentos adquiridos ao longo da jornada. O projeto reflete nosso aprendizado e dedicação, demonstrando habilidades técnicas e metodologias utilizadas no ambiente profissional.

## Case

A empresa fictícia está enfrentando um crescimento acelerado em seu E-commerce, tornando a infraestrutura atual incapaz de lidar com a alta demanda de acessos e compras. Para solucionar esse problema, a empresa TI SOLUÇÕES INCRÍVEIS alocou um time de dentro dos estúdios de Dev Sec Ops, para solucionar o desafio.
A equpe da TI SOLUÇÕES INCRÍVEIS foi designada para realizar uma consultoria no caso de uma migração para a **AWS do seu serviço completo** (todas as maquinas on-premise clonadas para cloud), da empresa Fast Enginnering S/A, garantindo maior escalabilidade, segurança e eficiência na infraestrutura, onde a solução atual de arquitetura da empresa não está atendendo mais a alta demanda de acessos e compras, assim sendo este o requisito principal do projeto. 
Posteriormente, após toda a migração para a AWS, tendo como a segunda etapa, foi solicitado a modernização com kubernetes: tendo a aplicação das melhores práticas de arquitetura em Cloud Computing para otimização e escalabilidade.

## Objetivo

O projeto visa realizar a migração da infraestrutura atual para a **AWS**, iniciando com um processo de **"lift-and-shift"**, garantindo uma transição rápida e segura, contando com sistema de failover. 

Em seguida, será realizada a **modernização da infraestrutura com Kubernetes**, aplicando as melhores práticas de arquitetura em Cloud Computing. Para saber mais sobre essa segunda etapa, acesse a página dedicada à **modernização com Kubernetes.** 



---

# MIGRATION OVERVIEW

Aqui será descrito o processo de migração da infraestrutura atual para a AWS, mantendo o máximo de compatibilidade e disponibilidade dos serviços durante a transição. 
Dentro deste, terá a versão resumida do que foi feito e selecionado para solucionar e construir a arquitetura do projeto, tendo links  adicionados para versão completa de cada etapa.

> Esta documentação contém links de acesso rápido para facilitar a navegação e garantir um acesso completo ao conteúdo. Sempre que precisar de mais detalhes, basta seguir os links indicados ao longo do texto 💡

---

## Estrutura Atual

![Arquitetura de operação utilizada pelo cliente](../../images/over1_structure_original.png)

Arquitetura de operação utilizada pelo cliente

- **Banco de Dados:** Servidor MySQL (500GB de dados, 10GB de RAM, 3 Core CPU).
- **Frontend:** Servidor React (5GB de dados, 2GB de RAM, 1 Core CPU).
- **Backend:** Servidor com 3 APIs, Nginx como balanceador de carga e armazenamento de arquivos estáticos (5GB de dados, 4GB de RAM, 2 Core CPU).

## Estruturas Propostas na AWS

Arquitetura Inicial As-Is:

![Arquitetura de operação proposta inicial](../../images/arc1.png)

Arquitetura Modernização Kubernetes:

![Arquitetura de operação proposta modernização](../../images/faq2_diagram_arq.png)


A migração inicial será feita com mínimas alterações, utilizando os seguintes serviços AWS:

- **Banco de Dados:** Amazon RDS MySQL Multi-AZ.
- **Frontend:** EC2 + Load Balancer.
- **Backend:** EC2 + Load Balancer.
- **Armazenamento de Arquivos:** Amazon S3.
- **Segurança:** AWS IAM, AWS WAF e Grupos de Segurança AWS.

## Etapas da Migração

### Migração do Banco de Dados com AWS DMS

Utilizaremos o **AWS Database Migration Service (AWS DMS)** para transferir os dados do MySQL local para o **Amazon RDS MySQL Multi-AZ**.

### Passos:

1. Criar uma instância de replicação do AWS DMS antes de configurar os endpoints.
2. Criar um **Endpoint de Origem** no AWS DMS apontando para o banco MySQL atual.
3. Criar um **Endpoint de Destino** para o novo banco no Amazon RDS.
4. Configurar uma **Tarefa de Migração** com opção de **Full Load + CDC** (Change Data Capture) para replicação contínua.
5. Iniciar a migração e monitorar no **AWS DMS Console**.
6. Validar a integridade dos dados e realizar o switch para o novo banco.

🔗 [Migração do Banco de Dados (MySQL) com AWS DMS](migration_bd.md) 
___

### Migração dos Servidores com AWS MGN

Utilizaremos o **AWS Application Migration Service (AWS MGN)** para migrar os servidores **frontend** e **backend** para **instâncias EC2** na AWS.

### Passos:

1. Instalar o **AWS MGN Agent** nos servidores on-premises.
2. Configurar a **Replicação Contínua** no console do AWS MGN.
3. Criar instâncias **EC2 de teste** para validar a migração.
4. Configurar o **AWS Load Balancer** para balanceamento de carga entre as instâncias.
5. Redirecionar o tráfego para os novos servidores AWS e desligar os antigos.

🔗 [Migração dos Servidores (Frontend e Backend) com AWS MGN](migration_ec2.md) 


___

### Armazenamento de Arquivos Estáticos no Amazon S3

Os arquivos estáticos serão movidos para o **Amazon S3** para melhorar desempenho e reduzir custos de infraestrutura.

### Passos:

1. Criar um **bucket S3** para armazenar os arquivos.
2. Configurar permissões adequadas com **AWS IAM e AWS WAF**.
3. Atualizar a configuração do **Nginx** para servir os arquivos do **S3**.
4. Implementar **AWS CloudFront** para caching e melhoria de performance.

🔗 [Armazenamento de Arquivos Estáticos no Amazon S3](static_arm.md) 


___

### Segurança e Configurações Adicionais

Para garantir a segurança do ambiente na AWS, utilizaremos:

- **AWS IAM** para controle de permissões e acesso.
- **AWS WAF** para proteção contra ataques como SQL Injection e XSS.
- **Security Groups** para restringir acessos às instâncias EC2.
- **AWS CloudTrail**: Ative o CloudTrail para registrar e monitorar todas as chamadas de API feitas em sua conta. Isso permite auditar a atividade do usuário e detectar ações suspeitas.
- **AWS Key Management Service (KMS)**: Utilize o KMS para gerenciar chaves de criptografia de forma segura, protegendo dados sensíveis armazenados em S3, RDS, EBS, e outros serviços.

🔗 [Segurança do Sistema](../security_aws.md) 

---
# Migração do Banco de Dados (MySQL) com AWS DMS

Brevemente, sobre o **AWS DMS** (Database Migration Service): permite migrar bancos de dados com pouco ou nenhum tempo de inatividade, permitindo migrações contínuas para manter a origem e o destino sincronizados.

### O porquê da escolha?

A escolha da arquitetura utilizando o **AWS Database Migration Service (DMS)** para a migração do banco de dados MySQL oferece diversas vantagens, especialmente em termos de custo-benefício, escalabilidade e eficiência operacional. Vamos abordar as principais razões para essa escolha:

1. **Baixo Custo de Inatividade**: O **DMS** permite migração com *mínimo ou nenhum tempo de inatividade*, o que significa que seu banco de dados pode continuar operando enquanto os dados são transferidos para o Amazon RDS. Isso reduz a necessidade de períodos de manutenção prolongados, o que é crucial para operações sensíveis ao tempo, como em ambientes de produção.
2. **Escalabilidade e Alta Disponibilidade**: Usar o **Amazon RDS MySQL Multi-AZ** como destino oferece **alta disponibilidade** e **failover automático**. Ou seja, em caso de falha no banco de dados principal, o RDS automaticamente muda para uma réplica, garantindo que o serviço continue funcionando sem interrupções. Isso é vital para empresas que não podem se dar ao luxo de ter longos períodos de inatividade.
3. **Facilidade e Eficiência**: O **DMS** automatiza a maior parte do processo de migração, como mapeamento de tabelas e sincronização contínua de dados com a funcionalidade **CDC (Change Data Capture)**. Isso reduz significativamente o tempo e o esforço manual necessários para mover grandes volumes de dados.
4. **Custo e Flexibilidade**: Ao optar por **AWS DMS e RDS**, você paga apenas pelos recursos que usa, como transferência de dados e o tempo de execução das instâncias de banco de dados. Isso é mais econômico em comparação a outras soluções que podem exigir hardware dedicado ou licenciamento adicional.
5. **Alternativas**: Outras ferramentas, como **dump e restore**, podem ser mais baratas em termos de licenciamento, mas não oferecem a mesma flexibilidade e recursos de **replicação contínua** e **mínima inatividade**. Além disso, ferramentas como o **AWS SCT** (Schema Conversion Tool) e **Aurora** podem ser usadas em casos específicos, mas podem exigir mais configuração e têm custos mais altos devido à natureza dos serviços.

Em resumo, a escolha do AWS DMS proporciona uma migração mais rápida, segura e com menos impacto no desempenho das operações, garantindo uma transição suave para o **Amazon RDS** e uma infraestrutura escalável no futuro.

---


### Fluxo da Arquitetura do AWS Database Migration Service (DMS) 

1. O **Source Database** fornece os dados a serem migrados.
2. O **Source Endpoint** conecta o banco de dados de origem ao **Replication Instance**.
3. A **Replication Instance** gerencia a migração através da **Replication Task**.
4. A **Replication Task** executa a migração de dados entre o **Source Endpoint** e o **Target Endpoint**.
5. O **Target Endpoint** envia os dados para o **Target Database** de destino.

Esse fluxo é orquestrado pelo AWS DMS, garantindo uma migração eficiente e contínua dos dados, sem causar interrupções nos sistemas de origem ou destino.


O fluxo de dados é orquestrado pelo DMS, garantindo que os dados sejam migrados de forma eficiente e sem interrupções significativas nos sistemas de origem e destino.

![Arquitetura de operação utilizada pelo cliente](../../images/faq1_on-premise_rds.png)
---


---
## Implementação

1. **Criar um Endpoint de Origem (Source Endpoint)**
    - No console do **AWS DMS**, vá para *Endpoints* e crie um endpoint para o seu banco de dados MySQL on-premises.
    - Defina as credenciais e o IP público (ou VPN/Direct Connect, se aplicável).
2. **Criar um Endpoint de Destino (Target Endpoint)**
    - Crie um endpoint para o **Amazon RDS MySQL Multi-AZ**.
    - Configure as credenciais e certifique-se de que o RDS pode receber conexões.
3. **Criar uma Tarefa de Migração**
    - Vá para *Database Migration Tasks* e crie uma nova tarefa.
    - Escolha entre migração completa (*full load*), replicação contínua (*CDC* – Change Data Capture) ou ambas.
    - Se precisar sincronizar as alterações feitas no banco de dados original após a migração, ative o **CDC**.
4. **Executar e Monitorar a Migração**
    - Inicie a migração e monitore no **AWS DMS Console**.
    - Valide os dados no **Amazon RDS** após a conclusão.

## Passo a passo técnico da implementação

### 1. **Criar um Endpoint de Origem (Source Endpoint)**

### Passos Técnicos:

- **Acesse o Console AWS DMS**: No console do AWS, vá até a seção **Database Migration Service (DMS)**.
- **Criação do Endpoint de Origem**:
    - Na barra lateral do DMS, selecione **Endpoints** e clique em **Create Endpoint**.
    - Selecione **Source Endpoint** como tipo de endpoint.
    - Escolha o tipo de banco de dados como **MySQL**.
    - Forneça o **nome do endpoint**, uma descrição e insira o **endpoint de conexão** (endereço IP público ou nome DNS do banco de dados MySQL on-premises).
    - **Credenciais de Conexão**: Insira o **usuário** e a **senha** com privilégios suficientes para acessar e migrar dados do banco de dados de origem.
    - Se estiver usando uma **VPN ou Direct Connect** para conectividade privada, adicione as informações de rede (você pode usar uma VPC para configurar a conectividade).
    - **Testar a Conexão**: Clique em **Test Connection** para verificar se o AWS DMS consegue conectar ao banco de dados de origem.
    - **Salvar** o endpoint.

### Considerações

- **Privilégios do Usuário**: O usuário configurado precisa ter privilégios de leitura e de reposta para execução de comandos de consulta.
- **Firewall/Segurança**: Certifique-se de que as regras de segurança (grupos de segurança e firewalls) permitam a conexão com o IP público ou privado do DMS.

### 2. **Criar um Endpoint de Destino (Target Endpoint)**

### Passos Técnicos

- **Acesse o Console AWS DMS**: No console, vá até a seção **Endpoints**.
- **Criação do Endpoint de Destino**:
    - Clique em **Create Endpoint**.
    - Escolha **Target Endpoint**.
    - Selecione **Amazon RDS** como o banco de dados de destino e, em seguida, escolha **MySQL**.
    - Defina o **nome do endpoint**, a descrição e insira as credenciais (usuário e senha) de acesso ao RDS.
    - **Configuração de Conexão**: Insira o **endpoint do RDS** que pode ser encontrado no painel do RDS (na seção de instâncias do banco de dados) e configure as portas de conexão.
    - Certifique-se de que o **grupo de segurança do RDS** permite a conexão com o **DMS**.
    - **Configuração de Multi-AZ**: Durante a criação do **Amazon RDS MySQL Multi-AZ**, certifique-se de ativar a alta disponibilidade, o que proporcionará failover automático em caso de falha.
    - **Testar a Conexão**: Teste a conexão para garantir que o DMS possa se conectar ao banco de dados de destino.

### Considerações

- **Conectividade VPC**: Caso seu RDS esteja dentro de uma VPC privada, verifique se a configuração de rede está correta e se o DMS tem acesso à VPC.
- **Backup e Retenção**: Certifique-se de que a política de backup do RDS está configurada adequadamente para proteger os dados após a migração.

### 3. **Criar uma Tarefa de Migração**

### Passos Técnicos

- **Criação da Tarefa**:
    - Vá até a seção **Database Migration Tasks** no console DMS e clique em **Create Task**.
    - Dê um **nome** para a tarefa de migração e adicione uma descrição se necessário.
    - Escolha entre as opções de migração:
        - **Full Load**: Migra todos os dados do banco de dados de origem para o destino.
        - **Change Data Capture (CDC)**: Sincroniza as alterações feitas no banco de dados de origem após a migração completa.
        - **Full Load + CDC**: Realiza a migração completa e, em seguida, continua a sincronizar as alterações feitas após a migração.
    - **Configuração de Tarefa**:
        - Selecione o **endpoint de origem** e o **endpoint de destino** que você criou previamente.
        - **Mapeamento de Tabelas**: O DMS pode mapear automaticamente as tabelas do banco de dados de origem para o banco de dados de destino, mas também permite configurar regras de mapeamento personalizadas.
        - **Modo de Replicação**: Se você escolheu a opção **CDC**, habilite a **replicação contínua**. Esse modo permite que as alterações no banco de dados de origem sejam aplicadas em tempo real no banco de dados de destino.
    - **Configuração de Tarefa Avançada**:
        - **Transformação de Dados**: Você pode configurar transformações de dados, como modificações de nomes de tabelas e colunas.
        - **Manejo de Erros**: Defina como o DMS lida com erros (retries, falhas, etc.).

### Considerações

- **Consistência de Dados**: Certifique-se de que a consistência de dados é preservada durante a migração, especialmente se o banco de dados de origem estiver em uso durante a migração (com CDC ativo).
- **Performance**: Aplique a configuração de **throttling** ou limite de taxa de replicação para evitar sobrecarga do banco de dados.

### 4. **Executar e Monitorar a Migração**

### Passos Técnicos

- **Iniciar a Tarefa**: Depois de configurar a tarefa de migração, clique em **Start Task** para iniciar o processo de migração.
- **Monitoramento da Migração**:
    - **Console DMS**: No console DMS, você pode acompanhar o progresso da migração, verificando os logs e a quantidade de dados transferidos.
    - **CloudWatch Logs**: Configure **AWS CloudWatch** para monitorar os logs de migração, o que ajuda a identificar problemas de desempenho e falhas.
    - **Indicadores de Progresso**: No console, é possível ver o status da tarefa (executando, concluída, com falhas, etc.), além de detalhes sobre a transferência de dados.
- **Validação dos Dados**: Após a conclusão, valide os dados no **Amazon RDS** para garantir que tudo foi migrado corretamente e sem falhas.

### Considerações

- **Validação Pós-Migração**: Execute queries de validação no banco de dados de destino para garantir que os dados foram migrados corretamente.
- **Verificação de Consistência**: Realize uma verificação para confirmar que os dados não foram corrompidos ou perdidos durante o processo de migração.

---

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



---

# Migração dos Servidores (Frontend e Backend) com AWS MGN

O **AWS MGN** (AWS Application Migration Service) permite migrar servidores completos (máquinas físicas ou VMs) para **instâncias EC2** na AWS.

## Por que a escolha do MGN?

O **AWS MGN** é uma ferramenta projetada para facilitar a migração de servidores físicos ou virtuais (máquinas on-premises) para a nuvem da AWS. A grande vantagem de usar o AWS MGN é que ele permite uma migração **simples**, **rápida** e **com mínima interrupção** no funcionamento do seu sistema. Veja os principais motivos:

1. **Simplicidade na instalação e configuração**:
    - O processo de migração envolve a instalação de um agente nos seus servidores. Isso significa que você não precisa se preocupar com configurações complexas. A ferramenta lida com a maioria do trabalho.
    - Após a instalação, a AWS cuida da replicação dos dados automaticamente, reduzindo a chance de erro.
2. **Escalabilidade**:
    - A AWS é conhecida por sua **escalabilidade**. Isso significa que, à medida que sua empresa cresce, você pode facilmente aumentar a capacidade dos seus servidores sem precisar realizar grandes mudanças na infraestrutura.
3. **Segurança**:
    - A AWS oferece uma infraestrutura de **segurança robusta**. Com o AWS MGN, todos os dados replicados são protegidos com criptografia durante o processo de migração, o que garante que suas informações estejam seguras.
4. **Facilidade na validação da migração**:
    - Você pode criar instâncias de **teste** na AWS para garantir que tudo esteja funcionando corretamente antes de transferir definitivamente os serviços. Isso ajuda a evitar surpresas e problemas depois da migração.
5. **Flexibilidade**:
    - O AWS MGN é flexível em relação ao tipo de **armazenamento** e **infraestrutura** que você deseja usar na AWS. Se precisar de mais poder de processamento ou mais armazenamento, basta ajustar as configurações conforme necessário.
    
    **Comparação de Preços**
    A AWS oferece uma **opção de pagamento por uso**, ou seja, você paga somente pelo que utilizar. Isso é vantajoso porque não é necessário fazer um grande investimento inicial. Aqui está uma comparação simples com outras soluções de migração e seus preços aproximados:
    
    1. **AWS MGN**:
        - A AWS cobra pela **replicação de dados** e pelo uso das instâncias EC2. O custo vai depender do tamanho das instâncias EC2 que você escolher e da quantidade de dados que será replicada.
        - **Vantagem**: Você paga conforme o uso, sem taxas fixas. E a replicação é eficiente, o que pode economizar tempo e recursos.
    2. **AWS Server Migration Service (SMS)**:
        - O **SMS** também permite a migração de servidores, mas tem mais foco em migrações de servidores simples. O custo é similar ao do MGN, mas o MGN tem mais recursos para migrações mais complexas e contínuas.
        - **Vantagem**: O MGN é mais completo, com maior flexibilidade e recursos extras.
    3. **Outras Ferramentas do Mercado** (como Carbonite, CloudEndure, etc.):
        - Algumas ferramentas de migração de terceiros também estão disponíveis no mercado, mas elas podem ser mais caras e oferecer menos **integração com a AWS**.
        - **Desvantagem**: Você pode precisar configurar e gerenciar essas ferramentas manualmente, o que demanda mais tempo e recursos da sua equipe.
        

## Implementação

### **Passos**

1. **Instalar o Agente do AWS MGN**
    - No servidor on-premises (frontend e backend), instale o **AWS MGN Agent**.
    - O agente se comunica com a AWS e permite replicar os dados para a nuvem, sincronizando-os.
2. **Configurar a Replicação**
    - No **AWS MGN Console**, adicione os servidores que deseja migrar.
    - Configure as opções de replicação (como tipo de armazenamento e VPC na AWS).
3. **Criar Instâncias EC2 na AWS**
    - Após a replicação, inicie uma instância de teste para validar a migração.
    - Ajuste os tamanhos das instâncias EC2 para corresponder às necessidades do frontend e backend.
4. **Configurar o Load Balancer**
    - No **AWS ELB (Elastic Load Balancer)**, configure um balanceador de carga para distribuir o tráfego entre as instâncias do frontend e backend.
5. **Redirecionar o Tráfego**
    - Após testar, redirecione o tráfego para os novos servidores AWS.
    - Finalize a migração desligando os servidores antigos, se necessário.

## Passo a passo técnico

### **Passo 1: Instalar o Agente do AWS MGN**

1. **Acessar a Instância do Servidor Local**:
    - Conecte-se ao servidor on-premises (tanto frontend quanto backend) via SSH (para servidores Linux) ou RDP (para servidores Windows).
2. **Baixar o Agente do AWS MGN**:
    - Acesse o console do **AWS MGN** e obtenha a versão mais recente do agente para o sistema operacional do servidor.
    - No console do AWS MGN:
        - Navegue até **Replications** > **Agents**.
        - Selecione **Install Agent** e copie o link de download do agente para o seu sistema operacional.
3. **Instalar o Agente no Servidor**:
    - Para **Linux**, execute o comando de instalação:

```json
sudo wget https://link_de_download_do_agente -O aws-mgn-agent.rpm
sudo rpm -ivh aws-mgn-agent.rpm

```

- Para **Windows**, execute o instalador `.exe` baixado, seguindo as instruções na tela.
1. **Verificar a Instalação**:

Após a instalação, inicie o agente:
- Para **Linux**, execute:

```json
sudo service aws-mgn-agent start

```

- Para **Windows**, o serviço será iniciado automaticamente após a instalação.
- No **AWS MGN Console**, verifique se o servidor foi detectado corretamente na seção **Replications**.

### **Passo 2: Configurar a Replicação**

1. **Adicionar Servidores ao Console AWS MGN**:
    - No **AWS MGN Console**, clique em **Replications** > **Add Server**.
    - Selecione os servidores (frontend e backend) que você deseja migrar.
    - Após adicionar, o console iniciará a replicação dos dados para a nuvem.
2. **Configurar Opções de Replicação**:
    - Selecione o **tipo de armazenamento** desejado (SSD, HDD, etc.).
    - Defina a **VPC (Virtual Private Cloud)**, sub-rede e as opções de segurança (grupos de segurança).
    - Configure o **IAM Role** (caso necessário) para permitir que o AWS MGN gerencie a replicação dos dados.
3. **Iniciar Replicação**:
    - Após a configuração, clique em **Start Replication** para começar a sincronização dos dados para as instâncias EC2 na AWS.

### **Passo 3: Criar Instâncias EC2 na AWS**

1. **Verificar a Replicação**:
    - No **AWS MGN Console**, confirme se os servidores locais foram replicados corretamente. Você verá as instâncias representando o frontend e o backend.
2. **Criar Instâncias EC2**:
    - Após a replicação, selecione a opção **Create Instance** para criar instâncias EC2 na AWS.
    - Ajuste o tipo e tamanho das instâncias conforme necessário para os requisitos de frontend e backend. Se necessário, altere o tipo de instância para melhorar o desempenho.
    - Escolha uma **AMIs (Amazon Machine Image)** baseadas na replicação e **dê start** nas instâncias.
3. **Testar a Instância EC2**:
    - Acesse a nova instância EC2 via SSH ou RDP e verifique se o servidor replicado está funcionando corretamente. Teste as funcionalidades do frontend e backend.

### **Passo 4: Configurar o Load Balancer**

1. **Criar Elastic Load Balancer (ELB)**:
    - Acesse o **AWS Console** e vá até **EC2** > **Load Balancers**.
    - Selecione **Create Load Balancer** e escolha o tipo de balanceador de carga desejado (Application Load Balancer para tráfego HTTP/HTTPS).
    - Defina as opções como **VPC**, **sub-redes** e **grupo de segurança**.
2. **Configurar Regras de Roteamento**:
    - Configure o balanceador de carga para distribuir o tráfego entre as instâncias EC2 do frontend e backend.
    - Defina regras de roteamento adequadas para o tráfego de rede, como distribuir tráfego HTTPS para o servidor de frontend e tráfego HTTP para o backend.
3. **Associar Instâncias EC2 ao Load Balancer**:
    - No console do ELB, adicione as instâncias EC2 criadas anteriormente ao balanceador de carga.

### **Passo 5: Redirecionar o Tráfego**

1. **Redirecionar o Tráfego para a AWS**:
    - Após a validação do funcionamento das instâncias EC2 e do balanceador de carga, atualize as configurações de DNS para redirecionar o tráfego para o Elastic Load Balancer.
    - Alterar os registros DNS para apontar para o endpoint do ELB.
2. **Finalizar a Migração**:
    - Teste a migração final e verifique se o frontend e backend estão funcionando corretamente na AWS.
    - Após a migração bem-sucedida, desligue os servidores antigos (se necessário) para liberar recursos.

Este processo garante que a migração seja realizada de maneira controlada, com validações contínuas e mínima interrupção dos serviços.


---


# MODERNIZAÇÃO: KUBERNETES

À medida que as aplicações crescem e demandam maior escalabilidade, confiabilidade e eficiência operacional, a migração para o Kubernetes (K8s) se torna uma solução estratégica. O Kubernetes permite a orquestração automatizada de contêineres, garantindo melhor disponibilidade, escalabilidade e gerenciamento dos serviços. Na AWS, o Elastic Kubernetes Service (EKS) facilita essa transição ao oferecer um ambiente gerenciado, reduzindo a complexidade operacional e permitindo que as equipes foquem no desenvolvimento e entrega contínua.

A seguir, detalharemos as etapas essenciais para a migração do projeto para o Kubernetes na AWS.

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


---

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


---


# FAQ do Projeto

Bem-vindo à seção de **Perguntas Frequentes (FAQ)** do nosso projeto. Aqui, você encontrará respostas diretas e objetivas para as questões levantadas pela **Compass**, abordando os principais aspectos técnicos e funcionais do desenvolvimento.

Nosso objetivo é fornecer esclarecimentos detalhados sobre as decisões de arquitetura, tecnologias utilizadas, funcionalidades implementadas e demais requisitos solicitados. Caso precise de informações adicionais, entre em contato com a equipe responsável pelo projeto.

---

# ETAPA AS IS

## Quais as ferramentas vão ser utilizadas?

AWS MGN

AWS DMS

AWS-EC2

AWS-ALB

AWS-RDS

AWS-S3

CloudWatch

NAT

EKS

## Qual o diagrama da empresa atual?

![On-premise para RDS](../images/over1_structure_original.png)

## Qual o diagrama da infraestrutura na AWS?

#### Etapa As-Is

![On-premise para RDS](../images/arc1.png) 

#### Etapa Modernizção

![On-premise para RDS](../images/faq2_diagram_arq.png) 


## Como serão garantidos os requisitos de Segurança?

- **AWS IAM**: Configure usuários e permissões de acesso.
- **AWS WAF**: Proteja o aplicativo contra ataques como SQL Injection e XSS.
- **CloudWatch**: Monitore logs, métricas e eventos do ambiente AWS, configurando alarmes para detectar atividades suspeitas ou falhas no sistema.
- **Security Groups**: Controle o tráfego permitido para as instâncias EC2.
- **AWS Key Management Service (KMS)**: Utilize o KMS para gerenciar chaves de criptografia de forma segura, protegendo dados sensíveis armazenados em S3, RDS, EBS, e outros serviços.

## Quais atividades são necessária para a migração?

As etapas de migração estão resumidas abaixo para facilitar o entendimento do processo. Cada etapa contém uma visão geral das atividades envolvidas. Para mais detalhes, links específicos estão disponíveis ao longo da documentação, proporcionando uma explicação mais aprofundada de cada fase. Recomendamos a leitura dos links para um melhor acompanhamento.

### Migração de Servidores e APIs para AWS EC2 com Application Migration Service

Este processo detalha os passos para a migração de servidores utilizando o **AWS Application Migration Service** (MGN). Ele envolve a instalação do agente de replicação, sincronização dos dados, testes de aceitação e a execução do cutover final para garantir uma transição segura e eficiente para a infraestrutura da AWS.

1. Inicializar o AWS Application Migration Service na região de destino.
2. Instalar o AWS Replication Agent no servidor de origem.
3. Aguardar a conclusão da sincronização inicial. Após instalar o agente, é necessário aguardar a conclusão do processo de sincronização inicial, que realiza a replicação em nível de bloco do servidor de origem para o servidor de replicação na área de preparação.
4. Lançar instâncias de teste. Após a sincronização inicial, é possível iniciar uma máquina de destino no Modo de Teste, permitindo a realização de testes de aceitação e a verificação do funcionamento correto do ambiente migrado.
5. Realizar testes de aceitação nos servidores. Após testar com sucesso a instância de teste, finalize o teste e exclua a instância.
6. Configurar ações pós-lançamento (se necessário).
7. Aguardar a janela de cutover.
8. Confirmar que não há atraso (lag).
9. Parar todos os serviços operacionais no servidor de origem.
10. Lançar uma instância de cutover. Iniciar a máquina de destino no Modo de Cutover, iniciando o processo final de migração.
11. Confirmar que a instância de cutover foi lançada com sucesso e, em seguida, finalizar o cutover.
12. Arquivar o servidor de origem.

### Banco de dados on-premise para RDS

![On-premise para RDS](../images/faq1_on-premise_rds.png)

>Link de acesso a documentação oficial 💡
>[Set up replication for AWS Database Migration Service](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_GettingStarted.Replication.html)

## Como será realizado o processo de Backup?

- O RDS realiza backups contínuos do banco de dados dentro do período de retenção configurado.
- Os arquivos estáticos armazenados no S3 são versionados e seguem um ciclo de vida.

## Qual o custo da infraestrutura na AWS (AWS Calculator)?

### Migração

Estimativa de três dias para migração:

- Custos da migração do banco de dados (72 horas de uso t3.large)
    - 10.51 USD
- Custos de armazenamento DMS
    - 57.50 USD

### Estrutura

- Custo mensal
    - 504.22 USD
- Custo total anual
    - 6,050.64 USD

![Estimativa Etapa 1](../images/estimate_faq1.png)

>Link de acesso ao orçamento na AWS (March 15, 2025) 💡
>[My Estimate - AWS](https://calculator.aws/#/estimate?id=934bc019249d8f441fd11dabf67c648a8cb619b5)


---

## ETAPA 2: MODERNIZAÇÃO COM KUBERNETES

## Quais atividades são necessária para a migração?

### 1. Converter estrutura em microserviços docker

- **Identificar componentes do sistema** (ex: backend, frontend, banco de dados, autenticação).
- **Separar funcionalidades monolíticas** em serviços independentes.
- **Criar imagens Docker** para cada serviço.
- **Publicar as imagens** no **Amazon ECR** (Elastic Container Registry).

### **2. Preparar o Cluster Kubernetes**

- Criar um **Cluster EKS** no AWS Management Console ou via CLI.
- Configurar **VPC, sub-redes e Auto Scaling** para os Worker Nodes.

### **Segurança e Configurações Adicionais**

- Configurar **IAM roles** para permissões do EKS.
- Implementar **AWS WAF e Security Groups** para proteção.

## Quais as ferramentas vão ser utilizadas?

AWS MGN

AWS DMS

AWS-EC2

AWS-ALB

AWS-RDS

AWS-S3

CloudWatch

NAT

EKS

CloudFront

ECR

## Qual o diagrama da infraestrutura na AWS?

![On-premise para RDS](../images/faq2_diagram_arq.png)

## Como serão garantidos os requisitos de Segurança?

- **AWS IAM**: configura usuários e permissões de acesso.
- **AWS WAF**: protege o aplicativo contra ataques como SQL Injection e XSS.
- **CloudWatch**: monitora logs, métricas e eventos do ambiente AWS, configurando alarmes para detectar atividades suspeitas ou falhas no sistema.
- **Security Groups**: controla o tráfego permitido para as instâncias EC2, restringir o tráfego externo para o cluster.
- **AWS Key Management Service (KMS)**: gerencia chaves de criptografia de forma segura, protegendo dados sensíveis armazenados em S3, RDS, EBS, e outros serviços.

Visando que o projeto esta em produção, será adicionado camadas extras de proteção e segurança:

Segurança de Containers e Kubernetes:  

- **PodSecurity Policies e RBAC**: Controla o acesso e as permissões de execução dentro do cluster Kubernetes, assegurando que os pods tenham as permissões mínimas e evitando o uso de práticas inseguras (como containers executando como root).
- **Escaneamento de Imagens de Containers (Trivy, Clair)**: Assegura que as imagens de containers estejam livres de vulnerabilidades conhecidas antes de serem implantadas no Kubernetes.

**Ameaças Internas e Zero Trust**

- **Network Policies no Kubernetes**: Implemente políticas de rede para garantir que apenas os pods autorizados possam comunicar entre si. Isso minimiza os riscos de movimento lateral em caso de comprometimento de um pod.
- **Segurança de API (API Gateway + Rate Limiting)**: Proteja suas APIs com um **API Gateway**, implemente **rate limiting** para evitar abuso e **autenticação/autorização** (OAuth, JWT).

**Segurança de Rede**

- **AWS Shield (Proteção contra DDoS)**: Ative o AWS Shield para proteger suas aplicações de ataques de negação de serviço distribuída (DDoS).
- **VPC com subnets privadas**: Configure sua infraestrutura de rede de forma que os componentes sensíveis (como bancos de dados) fiquem em subnets privadas e inacessíveis diretamente pela internet.

## Como será realizado o processo de Backup?

- O RDS realiza backups contínuos do banco de dados dentro do período de retenção configurado.
- Os arquivos estaticos armazenados no S3 serão versionados e seguirão um ciclo de vida

## Qual o custo da infraestrutura na AWS (AWS Calculator)?

- Custo mensal
    - 977.84 USD
- Custo total anual
    - 11,734.08 USD


![Estimativa Etapa 1](../images/estimate_faq1.png)


>[My Estimate - AWS](https://calculator.aws/#/estimate?id=1deb627dd7529d6a7b4e341042305ca2bb0cd456)
>Link de acesso ao orçamento na AWS (March 15, 2025) 💡

## Orçamento

A seguir, apresentamos a estimativa de custos para a implementação do projeto. A tabela abaixo detalha os valores correspondentes a cada item necessário para a execução, garantindo maior transparência e controle financeiro.

![Estimativa Etapa 1](../images/table_budget.png)


## Conclusão

Este projeto visa garantir uma migração eficiente e segura da infraestrutura on-premise para a AWS, utilizando serviços gerenciados para otimizar desempenho, escalabilidade e segurança. A transição ocorre em duas etapas: primeiro, replicamos a estrutura atual na nuvem com serviços como AWS MGN, RDS, EC2 e DMS; em seguida, modernizamos a aplicação com EKS e microserviços Docker, promovendo maior eficiência e flexibilidade.

A implementação adota boas práticas de segurança, incluindo IAM, WAF, criptografia via KMS e políticas de segurança para Kubernetes, garantindo um ambiente resiliente contra ameaças. Além disso, a estimativa de custos permite um planejamento financeiro detalhado, proporcionando previsibilidade no investimento.

Com essa abordagem estruturada, a migração não apenas reduz a complexidade operacional, mas também prepara a infraestrutura para suportar o crescimento futuro da aplicação com alto desempenho e confiabilidade.

---


# Fluxo de funcionamento

Para facilitar a compreensão, apresentamos dois elementos essenciais para o entendimento do projeto:

#### **📌 Estrutura Completa da Arquitetura**  
Descreve os componentes principais e a organização do sistema, detalhando como as tecnologias e serviços se conectam.

#### **📌 Fluxo de Funcionamento**  
Explica a dinâmica do sistema, desde a entrada de dados até a entrega da funcionalidade ao usuário, garantindo uma visão clara do processo.


# **Estrutura Completa da Arquitetura 📌**

### **Nível Global (Fora das Regiões da AWS)**

- **Route 53 (DNS)** → Resolve o domínio e direciona o tráfego.
- **CloudFront (CDN) com WAF** → Protege contra ataques, melhora a performance e reduz carga nos servidores.

### **Nível Regional (us-east-1, em Múltiplas AZs)**

### **(A) Tráfego Web**

- **Application Load Balancer (ALB)** → Distribui as requisições entre os nós do **EKS (Kubernetes)**.

### **(B) Camada de Aplicação (EKS - Kubernetes em Múltiplas AZs)**

- **Node 1 (Frontend - React/Vue/Angular)** → Hospeda o frontend da aplicação.
- **Node 2 (Backend - APIs)** → Contém 3 APIs para processamento de lógica de negócios.

### **(C) Camada de Banco de Dados**

- **Multi-AZ Cluster Database (Amazon RDS - Aurora, PostgreSQL, MySQL, etc.)**
    - **Primary DB (Escrita/Leitura)** → Localizado em uma AZ primária.
    - **Read Replica 1** → Distribuída em outra AZ para balancear leituras.
    - **Read Replica 2** → Outra réplica para leitura, garantindo escalabilidade.
    - **Failover automático** → Se o banco principal falhar, uma réplica é promovida automaticamente.

### **(D) Armazenamento**

- **Amazon S3 (Armazenamento de objetos)**
    - **Usado para arquivos estáticos**, como imagens, vídeos e documentos.
    - **Acessado via CloudFront** para melhorar performance.

---

# **Fluxo de Funcionamento📌**


### **Cliente acessa o site**

🔹 O usuário digita `siteempresa.com` → Route 53 resolve o domínio.

### **Segurança e distribuição de tráfego**

🔹 O tráfego passa pelo **CloudFront (com WAF ativado)**:

- Filtra ataques DDoS, SQL Injection, etc.
- Se for um **arquivo estático**, CloudFront busca no S3.
- Se for uma **requisição dinâmica**, CloudFront encaminha para o ALB.

### **Load Balancer distribui para o EKS**

🔹 O **ALB** recebe a requisição e redireciona:

- Se for frontend → Envia para o **Node 1 (Frontend)** no EKS.
- Se for backend → Envia para o **Node 2 (APIs Backend)** no EKS.

### **Backend processa a requisição e acessa o banco**

🔹 As APIs no Node 2 processam a lógica de negócio e acessam o banco de dados:

- Escritas vão para o **Primary DB**.
- Leituras são balanceadas entre **Read Replica 1 e 2**.

### **Resposta é enviada de volta para o cliente**

🔹 O backend retorna os dados ao frontend → O frontend exibe as informações na interface.

## **📌 Benefícios da Arquitetura**

**Alta Disponibilidade**:

- EKS rodando em **múltiplas AZs**.
- **Banco de dados Multi-AZ** com failover automático.

**Escalabilidade**:

- Load Balancer e Kubernetes escalam os pods automaticamente.
- Read Replicas distribuem as consultas ao banco.

**Segurança**:

- **CloudFront + WAF** bloqueiam ataques antes de chegarem ao backend.
- **RDS Multi-AZ** protege contra falhas no banco.

**Performance Otimizada**:

- **CloudFront faz caching** de arquivos estáticos.
- **Read Replicas aliviam a carga do banco**.

### **📌 Conclusão**

**Essa arquitetura garante resiliência, eficiência e segurança, sendo uma das melhores opções para aplicações escaláveis na AWS.**

---


# SECURITY AWS

Para garantir a segurança e conformidade de um ambiente na AWS, é essencial configurar corretamente os serviços de gerenciamento de identidade, controle de acesso, monitoramento e proteção contra ameaças. A implementação de boas práticas de segurança envolve a restrição de permissões, o monitoramento contínuo das atividades, a proteção contra ataques cibernéticos e o uso de criptografia para proteger dados sensíveis. A seguir, destacamos alguns serviços fundamentais da AWS que ajudam a fortalecer a segurança da infraestrutura em nuvem.

- **AWS IAM**: Configure usuários e permissões de acesso.
- **AWS WAF**: Proteja o aplicativo contra ataques como SQL Injection e XSS.
- **CloudWatch**: Monitore logs, métricas e eventos do ambiente AWS, configurando alarmes para detectar atividades suspeitas ou falhas no sistema.
- **Security Groups**: Controle o tráfego permitido para as instâncias EC2.
- **AWS Key Management Service (KMS)**: Utilize o KMS para gerenciar chaves de criptografia de forma segura, protegendo dados sensíveis armazenados em S3, RDS, EBS, e outros serviços.


---

## Navegação


### 🚀 Modernização
- [Kubernetes Moderno](../doc/modern/modern_k8s.md)
- [Setup e Técnicas](../doc/modern/step_tecnicas.md)

### 🔄 Migração
- [Visão Geral](../doc/migration/migration_overview.md)
- [Migração de Banco de Dados](../doc/migration/migration_bd.md)
- [Migração para EC2](../doc/migration/migration_ec2.md)
- [Migração de Arquitetura Estática para ARM](../doc/migration/static_arm.md)

### 🔒 Segurança
- [Segurança na AWS](../doc/security_aws.md)

### ❓ FAQ
- [Perguntas Frequentes](../doc/faq.md)
- [Fluxo de Funcionamento](../doc/flow.md)


### 📚 Documentação Completa
- [Ver Documento Completo](../doc/full_doc.md)
