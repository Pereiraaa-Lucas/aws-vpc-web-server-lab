# 🌐 Laboratório AWS: Criação de VPC Customizada e Servidor Web

Este repositório documenta a realização do laboratório prático da AWS focado em redes em nuvem, onde foi construída uma arquitetura de rede personalizada (VPC) do zero e implantada uma instância Amazon EC2 configurada como um servidor web.

---

## 🏗️ Arquitetura da Solução

A infraestrutura foi projetada para atender aos requisitos de alta disponibilidade e segurança, dividindo recursos entre duas Zonas de Disponibilidade (AZs).

### Componentes Implantados:
* **Amazon VPC**: Bloco CIDR `10.0.0.0/16`.
* **Sub-redes Públicas (2 AZs)**: 
  * `Public Subnet 1` (10.0.0.0/24) com Gateway NAT[cite: 24].
  * `Public Subnet 2` (10.0.2.0/24)[cite: 25].
* **Sub-redes Privadas (2 AZs)**:
  * `Private Subnet 1` (10.0.1.0/24)[cite: 24].
  * `Private Subnet 2` (10.0.3.0/24)[cite: 25].
* **Internet Gateway (IGW)**: Para prover acesso externo à VPC[cite: 24].
* **Gateway NAT**: Para permitir que instâncias nas sub-redes privadas acessem a internet de forma segura[cite: 24].
* **Tabelas de Rotas**: Rotas configuradas para tráfego local (`10.0.0.0/16`) e externo (`0.0.0.0/0`)[cite: 24].
* **Grupo de Segurança (`Web Security Group`)**: Firewall virtual configurado para permitir tráfego HTTP (`Porta 80`) de qualquer origem (`Anywhere-IPv4`).
* **Instância Amazon EC2 (`Web Server 1`)**:
  * Tipo: `t3.micro`.
  * Sistema Operacional: Amazon Linux 2[cite: 26].
  * Configurada com um script de inicialização (*User Data*) para instalação automática do servidor web Apache e PHP[cite: 26].

---

## 🛠️ Passo a Passo Realizado no Laboratório

1. **Criação da VPC e Recursos Iniciais (Tarefa 1):** Utilização do assistente de VPC para gerar a estrutura base com sub-redes públicas e privadas na Zona de Disponibilidade A[cite: 24].
2. **Expansão para Múltiplas AZs (Tarefas 2 e 3):** Criação da `Public Subnet 2` e `Private Subnet 2` na Zona de Disponibilidade B, associando-as às respectivas tabelas de rotas para garantir alta disponibilidade[cite: 25].
3. **Configuração de Segurança (Tarefa 4):** Criação do Grupo de Segurança da Web liberando tráfego HTTP na porta 80[cite: 26].
4. **Implantação do Servidor Web (Tarefa 5):** Lançamento da instância EC2 conectada à nova VPC, associada ao grupo de segurança correto e provisionada com o script de dados do usuário (*User Data*)[cite: 26]:
   ```bash
   #!/bin/bash
   #Install Apache Web Server and PHP
   yum install -y httpd mysql php
   #Download Lab files
   wget [https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RESTRT-1/267-lab-NF-build-vpc-web-server/s3/lab-app.zip](https://aws-tc-largeobjects.s3.us-west-2.amazonaws.com/CUR-TF-100-RESTRT-1/267-lab-NF-build-vpc-web-server/s3/lab-app.zip)
   unzip lab-app.zip -d /var/www/html/
   #Turn on web server
   chkconfig httpd on
   service httpd start
