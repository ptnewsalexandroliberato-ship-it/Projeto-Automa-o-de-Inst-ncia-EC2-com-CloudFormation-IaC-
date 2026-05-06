# Projeto: Automação de Instância EC2 com CloudFormation (IaC)

## 🎯 Objetivo
Demonstrar a aplicação de **Infrastructure as Code (IaC)** para o provisionamento automatizado de uma instância EC2 na AWS, incluindo a configuração de um servidor web funcional e regras de segurança (Security Groups). Este projeto faz parte do desafio de laboratório da **DIO**.

---

### 📂 Estrutura do Repositório
*   `/templates/network-and-ec2.yaml`: Template principal com a definição da infraestrutura.
*   `/scripts/user-data.sh`: Script de automação para configuração do servidor (Apache).
*   `README.md`: Documentação técnica do projeto.

---

### 📝 Detalhes do Projeto

#### **1. Arquitetura da Stack**
O template desenvolvido neste repositório utiliza o AWS CloudFormation para realizar as seguintes ações:
*   **Provisionamento de Instância:** Criação de uma instância EC2 utilizando Amazon Linux 2.
*   **Segurança:** Configuração de um **Security Group** permitindo tráfego nas portas 80 (HTTP) e 22 (SSH).
*   **Automação de Software:** Uso da seção **UserData** para instalar, iniciar e habilitar o servidor Apache (`httpd`) automaticamente no boot.

#### **2. Tecnologias Utilizadas**
*   **AWS CloudFormation**: Orquestração e gerenciamento da infraestrutura.
*   **YAML**: Linguagem utilizada para a escrita do template.
*   **Bash Script**: Automação do setup pós-instalação via UserData.

#### **3. Insights de Implementação**
*   **Segurança (Least Privilege):** O Security Group foi configurado para abrir apenas as portas estritamente necessárias para o funcionamento do servidor web e gerenciamento remoto.
*   **Flexibilidade:** O uso de **Parameters** permite que o usuário escolha o tipo de instância no momento do deploy, facilitando o controle de custos.
*   **Saídas Dinâmicas:** A seção **Outputs** fornece automaticamente o DNS público da instância, facilitando o acesso imediato após o provisionamento.

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
    Description: "Tipo de instância EC2"

  KeyName:
    Type: AWS::EC2::KeyPair::KeyName
    Description: "Nome do seu KeyPair para acesso SSH"

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
      ImageId: ami-0c101f26f147fa7fd # Amazon Linux 2 (Verificar Região)
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

 O projeto foi desenvolvido seguindo todas as boas práticas e sintaxe do CloudFormation. Devido a limitações de acesso ao console AWS no momento, a validação foi feita via análise estática do template.
  PublicDNS:
    Description: 'URL do Servidor Web'
    Value: !GetAtt MyEC2Instance.PublicDnsName
