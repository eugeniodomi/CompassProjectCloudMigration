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
