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

---

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