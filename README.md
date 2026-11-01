# Projeto: Criação de VPC Personalizada e Instância de Servidor Web na AWS

Este repositório documenta o passo a passo e a arquitetura desenvolvida durante o laboratório prático **"Crie sua VPC e inicie um servidor web"** da AWS Cloud Computing Foundations Academy. O objetivo principal foi projetar e implementar uma rede isolada, segura e altamente disponível na nuvem da AWS.

---

## 🏗️ Arquitetura da Solução

A infraestrutura provisionada contempla os seguintes componentes de rede e computação:

* **Amazon VPC**: Bloco CIDR `10.0.0.0/16` configurado com isolamento de rede.
* **Zonas de Disponibilidade (AZs)**: Utilização de duas AZs (`us-west-2a` e `us-west-2b`) para redundância.
* **Sub-redes Públicas**: 
  * `Public Subnet 1`: `10.0.0.0/24` (na AZ A)
  * `Public Subnet 2`: `10.0.2.0/24` (na AZ B)
* **Sub-redes Privadas**: 
  * `Private Subnet 1`: `10.0.1.0/24` (na AZ A)
  * `Private Subnet 2`: `10.0.3.0/24` (na AZ B)
* **Gateways**: 
  * **Internet Gateway (IGW)** para tráfego externo/público.
  * **NAT Gateway** posicionado na sub-rede pública para prover acesso seguro à internet para instâncias nas sub-redes privadas.
* **Tabelas de Rotas**: Roteamento configurado separadamente para o tráfego público e privado.
* **Segurança (Security Group)**: `Web Security Group` permitindo o tráfego de entrada HTTP (porta 80) a partir de qualquer origem (`0.0.0.0/0`).
* **Computação (Amazon EC2)**: Instância `t3.micro` rodando Amazon Linux 2, provisionada na Sub-rede Pública 2, atuando como servidor web através do script de inicialização (UserData) do Apache.

---

## 🛠️ Passo a Passo da Implementação

### 1. Criação da VPC e Redes Iniciais
- Utilização do assistente de criação da VPC ("VPC e muito mais") para configurar o bloco CIDR principal `10.0.0.0/16`.
- Configuração de 1 NAT Gateway e sub-redes iniciais na Zona de Disponibilidade A.

### 2. Expansão para Alta Disponibilidade (Multi-AZ)
- Criação manual da **Public Subnet 2** (`10.0.2.0/24`) e da **Private Subnet 2** (`10.0.3.0/24`) na segunda Zona de Disponibilidade.
- Associação da *Public Subnet 2* à **Public Route Table** e da *Private Subnet 2* à **Private Route Table**.

### 3. Configuração do Grupo de Segurança
- Criação do grupo de segurança restrito à *Lab VPC*.
- Adição de regra de entrada (*Inbound Rule*) liberando a porta **HTTP (80)** para o mundo externo (`Anywhere-IPv4`).

### 4. Lançamento do Servidor Web (Amazon EC2)
- Instância criada do tipo `t3.micro` baseada em **Amazon Linux 2**.
- Associação à *Public Subnet 2* com atribuição automática de IP público.
- Inserção do script de automação no campo **Dados do Usuário (UserData)** para instalar e iniciar o servidor Apache e baixar os arquivos do laboratório:
  ```bash
  #!/bin/bash
  yum install -y httpd mysql php
  wget [https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RESTRT-1/267-lab-NF-build-vpc-web-server/s3/lab-app.zip](https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RESTRT-1/267-lab-NF-build-vpc-web-server/s3/lab-app.zip)
  unzip lab-app.zip -d /var/www/html/
  chkconfig httpd on
  service httpd start
