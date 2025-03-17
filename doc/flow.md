# Fluxo de funcionamento

Para facilitar a compreensão, apresentamos dois elementos essenciais para o entendimento do projeto:

### **📌 Estrutura Completa da Arquitetura**  
Descreve os componentes principais e a organização do sistema, detalhando como as tecnologias e serviços se conectam.

### **📌 Fluxo de Funcionamento**  
Explica a dinâmica do sistema, desde a entrada de dados até a entrega da funcionalidade ao usuário, garantindo uma visão clara do processo.


## **📌 Estrutura Completa da Arquitetura**

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

## **📌 Fluxo de Funcionamento**


### **1️⃣ Cliente acessa o site**

🔹 O usuário digita `siteempresa.com` → Route 53 resolve o domínio.

### **2️⃣ Segurança e distribuição de tráfego**

🔹 O tráfego passa pelo **CloudFront (com WAF ativado)**:

- Filtra ataques DDoS, SQL Injection, etc.
- Se for um **arquivo estático**, CloudFront busca no S3.
- Se for uma **requisição dinâmica**, CloudFront encaminha para o ALB.

### **3️⃣ Load Balancer distribui para o EKS**

🔹 O **ALB** recebe a requisição e redireciona:

- Se for frontend → Envia para o **Node 1 (Frontend)** no EKS.
- Se for backend → Envia para o **Node 2 (APIs Backend)** no EKS.

### **4️⃣ Backend processa a requisição e acessa o banco**

🔹 As APIs no Node 2 processam a lógica de negócio e acessam o banco de dados:

- Escritas vão para o **Primary DB**.
- Leituras são balanceadas entre **Read Replica 1 e 2**.

### **5️⃣ Resposta é enviada de volta para o cliente**

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
