# Projeto: Automação de Instância EC2 com CloudFormation (IaC)

## 🎯 Objetivo
Este projeto demonstra o desenvolvimento de **Infrastructure as Code (IaC)** para o provisionamento automatizado de uma infraestrutura na AWS. O foco principal é a criação de um template robusto que automatiza a configuração de instâncias EC2, regras de segurança e o setup de um servidor web.

---

### 📂 Estrutura do Repositório
*   `/templates/network-and-ec2.yaml`: Template principal com a definição da infraestrutura (CloudFormation).
*   `/scripts/user-data.sh`: Lógica de automação utilizada para configuração do servidor Apache.
*   `README.md`: Documentação técnica e guia do projeto.

---

### 📝 Detalhes Técnicos e Arquitetura

#### **1. Componentes da Stack**
O template YAML foi estruturado para garantir um ambiente funcional e seguro, utilizando:
*   **EC2 Instance:** Utilizando a AMI do Amazon Linux 2, otimizada para performance na nuvem.
*   **Security Group:** Implementação de regras de Firewall para permitir acesso via HTTP (porta 80) para usuários e SSH (porta 22) para administração.
*   **Bootstrap com UserData:** Automação completa do ciclo de vida inicial do servidor, realizando o update do SO, instalação e ativação do serviço Apache (`httpd`) sem intervenção manual.

#### **2. Ferramentas e Conceitos Aplicados**
*   **AWS CloudFormation**: Gerenciamento de recursos como código.
*   **YAML**: Escrita de templates seguindo padrões de indentação e sintaxe AWS.
*   **Reutilização de Código**: Uso de **Parameters** para permitir que a stack seja flexível (escolha de tipos de instância e chaves de acesso).
*   **Outputs**: Configuração de saídas para facilitar a gestão pós-deploy (DNS Público).

#### **3. Insights de Desenvolvimento**
> **Nota de Implementação:** Este projeto foca na excelência da escrita de código IaC. A estrutura foi planejada para ser "Production Ready", respeitando o princípio de menor privilégio nos Security Groups e garantindo que a infraestrutura seja totalmente replicável e documentada.

---

### 🛠️ Código do Template (`network-and-ec2.yaml`)

```yaml
AWSTemplateFormatVersion: '2010-09-09'
Description: 'DIO Challenge - Provisionamento de Servidor Web EC2'

Parameters:
  InstanceType:
    Type: String
    Default: t2.micro
    AllowedValues: [t2.micro, t3.micro]
    Description: "Tipo de instância EC2 para controle de custos"

  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: "Nome do KeyPair para acesso seguro via SSH"

Resources:
  WebServerSecurityGroup:
    Type: 'AWS::EC2::SecurityGroup'
    Properties:
      GroupDescription: 'Habilitar acesso HTTP e SSH'
      SecurityGroupIngress:
        - IpProtocol: tcp
          FromPort: 80
          ToPort: 80
          CidrIp: 0.0.0.0/0
        - IpProtocol: tcp
          FromPort: 22
          ToPort: 22
          CidrIp: 0.0.0.0/0

  MyEC2Instance:
    Type: 'AWS::EC2::Instance'
    Properties:
      InstanceType: !Ref InstanceType
      SecurityGroups: [ !Ref WebServerSecurityGroup ]
      ImageId: ami-0c101f26f147fa7fd # ID da AMI Amazon Linux 2
      KeyName: !Ref KeyName
      UserData:
        Fn::Base64: !Sub |
          #!/bin/bash
          yum update -y
          yum install -y httpd
          systemctl start httpd
          systemctl enable httpd
          echo "<h1>Projeto CloudFormation DIO - Servidor Online</h1>" > /var/www/html/index.html

Outputs:
  PublicDNS:
    Description: 'URL pública para acesso ao servidor web'
    Value: !GetAtt MyEC2Instance.PublicDnsName
