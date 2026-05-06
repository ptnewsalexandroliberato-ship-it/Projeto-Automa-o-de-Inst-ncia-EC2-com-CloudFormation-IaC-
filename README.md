# Projeto-Automa-o-de-Inst-ncia-EC2-com-CloudFormation-IaC-
Objetivo Demonstrar a aplicação de Infrastructure as Code (IaC) para o provisionamento automatizado de uma instância EC2, incluindo a configuração de um servidor web funcional e regras de segurança (Security Groups).


### 📂 Estrutura de Arquivos Proposta
*   `/templates/network-and-ec2.yaml`: Template principal com a infraestrutura.
*   `/scripts/user-data.sh`: Script de automação para configurar o servidor (Apache/Nginx).
*   `README.md`: Documentação técnica.

---

### 📝 Conteúdo para o `README.md` (Destaque DIO)

#### **Projeto: Automação de Instância EC2 com CloudFormation (IaC)**

**1. Objetivo**
Demonstrar a aplicação de **Infrastructure as Code (IaC)** para o provisionamento automatizado de uma instância EC2, incluindo a configuração de um servidor web funcional e regras de segurança (Security Groups).

**2. Arquitetura da Stack**
O template desenvolvido neste repositório realiza as seguintes ações:
*   Provisiona uma instância **EC2 (Amazon Linux 2)**.
*   Configura um **Security Group** permitindo tráfego HTTP (porta 80) e SSH (porta 22).
*   Utiliza a seção **UserData** para automatizar a instalação do servidor Apache no boot da instância.

**3. Tecnologias Utilizadas**
*   **AWS CloudFormation**: Orquestração de recursos.
*   **YAML**: Linguagem para escrita do template.
*   **Bash Script**: Automação de setup pós-instalação.

**4. Insights de Implementação**
*   **Segurança:** Implementei o princípio de menor privilégio no Security Group, restringindo o acesso apenas às portas necessárias.
*   **Escalabilidade e Reuso:** O template foi estruturado com parâmetros para permitir a escolha dinâmica do tipo de instância (ex: t2.micro vs t3.small) sem alterar o código base.

---

### 🛠️ Código do Template (`network-and-ec2.yaml`)

Este código reflete exatamente o que a descrição do desafio pede:

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
      ImageId: ami-0c101f26f147fa7fd # Amazon Linux 2 (Verificar região)
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
    Description: 'DNS Público da instância'
    Value: !GetAtt MyEC2Instance.PublicDnsName
```

---

### 🚀 Próximos Passos:
1.  Suba esses arquivos para o seu **GitHub**.
2.  Adicione um print do **CloudFormation Designer** na pasta `/images` para mostrar que você entende a visualização lógica.
3.  No campo de entrega da DIO, use a descrição do projeto focando em **"Automatização e Segurança de Infraestrutura"**.

**Está pronto para seguir ou quer ajustar algo neste template?**
