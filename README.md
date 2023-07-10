#  Atividade - AWS - Docker - Senac

Nesta atividade avaliátiva do programa de bolsas da Compass.UOL iremos colocar em prática os conhecimentos de AWS, Linux e Docker. O repositório abaixo contém a documentação e os passos necessários para realização da tarefa.

##  Objetivo
Construir o ambiente proposto abaixo:
![github](https://github.com/potnza/Atividade-Compasso/assets/113041172/521089cc-f914-4fca-8fb3-df535889c0a8)

##    Requisitos
1. Instalação e configuração do DOCKER ou CONTAINERD no host EC2; Ponto adicional para o trabalho utilizar a instalação via script de Start Instance (user_data.sh)
2. Efetuar Deploy de uma aplicação Wordpress com: container de aplicação RDS database Mysql
3. configuração da utilização do serviço EFS AWS para estáticos do container de aplicação Wordpress
4. configuração do serviço de Load Balancer AWS para a aplicação Wordpress

## Pontos de Atenção
- Não utilizar ip público para saída do serviços WP (Evitem publicar o serviço WP via IP Público)
- Sugestão para o tráfego de internet sair pelo LB (Load Balancer)
- Pastas públicas e estáticos do wordpress sugestão de utilizar o EFS (Elastic File System)
- Fica a critério de cada integrante (ou grupo) usar Dockerfile ou Docker-compose;
- Necessário demonstrar a aplicação wordpress funcionando
- (tela de login)
- Aplicação Wordpress precisa estar rodando na porta 80 ou
- 8080;
- Utilizar repositório git para versionamento;
- Criar documentação.

## Criando Security Groups

Security group do Load Balancer

| Tipo | Protocolo | Intervalo de portas | Origem                                        |
|------|-----------|---------------------|-----------------------------------------------|
| Custom TCP  | TCP       | 80                 | 0.0.0.0/0         |

Security Group Bastion Host

| Tipo | Protocolo | Intervalo de portas | Origem                                        |
|------|-----------|---------------------|-----------------------------------------------|
| Custom TCP  | TCP       | 22                 | Meu IP     | 
| Custom TCP  | TCP       | 80                 | Meu IP         |

Security Group das instancias

| Tipo | Protocolo | Intervalo de portas | Origem                                        |
|------|-----------|---------------------|-----------------------------------------------|
| Custom TCP  | TCP       | 22                 | sg-BastionHost    | 
| Custom TCP  | TCP       | 80                 | sg-LoadBalancer         |

Security Mount Targets

| Tipo | Protocolo | Intervalo de portas | Origem                                        |
|------|-----------|---------------------|-----------------------------------------------|
| Custom TCP  | TCP       | 2049                 | sg-instancias    | 
| Custom TCP  | TCP       | 80                 | sg-BastionHost         |

Security Group do RDS

| Tipo | Protocolo | Intervalo de portas | Origem                                        |
|------|-----------|---------------------|-----------------------------------------------|
| Custom TCP  | TCP       | 3306                 | sg-instancias    | 
| Custom TCP  | TCP       | 3306                 | sg-BastionHost         |

Security Group do EFS

| Tipo | Protocolo | Intervalo de portas | Origem                                        |
|------|-----------|---------------------|-----------------------------------------------|
| Custom TCP  | TCP       | 2049               | sg-Bastion   | 
| Custom TCP  | TCP       | 2049                 | sg-instancias         |

## Criando uma VPC
- Digite no menu de pesquisa VPC e acesse o menu de criação da VPC.
- Vá em **Create VPC**
- Selecione **VPC Only**
- Coloque um nome de sua preferência.
- Em IPv4 CIDR coloque 10.0.0.0/16
- Crie a VPC.
- Depois clique sobre ela e vá em **Edit VPC setting** e ative a opção **Enable DNS hostnames**

## Criando Internet Gateway
- Vá ate o menu de Internet Gateway e clique em **Create Internet Gateway**
- De um nome de sua preferência, e associe-o á nossa VPC criada anteriorimente.

## Criando NAT gatewat
- Ainda no menu de VPC, clique **NAT Gateway** e depois em **Create Nat gateway**
- De um nome de sua escolha, selecione uma sub-net publica, criada anteriormente e em **Connectivity type** deixe como público.
- Por fim associe um elastic IP e crie a NAT.

## Criando Sub-nets
- No menu de VPC ainda, vá em subnets e depois em **Create subnet**
- Crie duas sub-redes, uma pública e uma privada, elas precisam estar na mesma zona de disponibilidade. Repita o processo para a segunda zona de diponibilidade;

## Criando tabela de rotas
- Crie uma Route table em **route tables** e depois em **create route table**
- Crie duas tabelas, uma para sub-nets privadas e outra para sub-nets públicas.
- Depois associe cada sub-net a sua respectiva tabela, privada na tabela privada e publica na tabela publica.
- Selecione a tabela privada e clique em **Edit Routes** o Destinatinatio deve ser 0.0.0.0/0 e o Target deve ser o **internet gateway**, o mesmo que criamos a pouco.
- Selecione a tabela publica e clique em **Edit Routes** o Destinatinatio deve ser 0.0.0.0/0 e o Target deve ser o **NAT Gateway**, o mesmo que criamos a pouco.

## EFS (Elastic File Sistem)
- Vá até o serviço de EFS na aws e clique em **Create File System**;
- Selecione a VPC criada anteriormente e clique em **next**;
- Defina as duas zonas de disponibilidade e mude o security group para o criado anteriormente;
- Selecione, novamente, Próximo para as duas etapas seguintes e crie seu Elastic File Sistem.

## RDS MySQL (Relational Database Service)
 - Abra o console AWS e selecione o serviço de RDS e clique em **Create Data Base**;
- Selecione a opção **free tier** e **Mysql**
 - Abaixo, faça as configurações que achar necessário. Neste caso, manteremos como padrão.
- Altere a VPC para a criada anteriormente, bem como o Grupo de segurança e a preferência pela zona de disponibilidade.
- Finalize a criação do DB.

## Bastion Host
- Vá até o painel EC2 da Amazon
- Selecione **Launch Instance**
- Selecione a imagem **Amazon Linux 2**
- Selecione o tipo **t3.small**
- Selecione a VPC que criamos anteriormente
- Crie uma nova chave .pem
- Clique em **edit network**, selecione a VPC anteriormente já criada;
- Selecione a Subnet pública 1a e habilite o endereçamento de ip público;
- Após, selecione o Security Group do Bastion Host;
- No user data que fica em Advanced Details iremos adicionar o seguinte script.
```shell
#!/bin/bash
# Instalar Docker-CE ( Container Engine): 
sudo yum update
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
sudo usermod -aG docker ec2-user
# Instalar Docker Compose:
sudo curl -SL https://github.com/docker/compose/releases/download/v2.19.1/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
# EFS
sudo yum install amazon-efs-utils -y
sudo mkdir -m 666 /home/ec2-user/efs
sudo echo "${ID_EFS}   /home/ec2-user/efs    efs     defaults,_netdev    0   0" | sudo tee -a /etc/fstab
sudo mount -a
# Cria o docker-compose.yaml
sudo mkdir /home/ec2-user/docker-compose
sudo echo -e "version: '3.1'\n 
services:\n
  wordpress:\n
    image: wordpress:latest\n        
    restart: always\n
    ports:\n
      - 80:80\n
    environment:\n
      WORDPRESS_DB_HOST: ${ENDPOINT_RDS}\n
      WORDPRESS_DB_USER: admin\n
      WORDPRESS_DB_PASSWORD: 12345678\n
      WORDPRESS_DB_NAME: wordpress\n
    volumes:\n
      - /home/ec2-user/efs:/var/www/html\n
" | sudo tee /home/ec2-user/docker-compose/docker-compose.yml
sudo sed -i '/^$/d' /home/ec2-user/docker-compose/docker-compose.yml
```

## Criação da AMI a partir da EC2
- Vá até o serviço de EC2 no console AWS e acesse as instancias.
- Selecione a instância previamente criada, nosso bastion host, clique com o botão direito sobre e vá em > "Image and Templates" > "Create Image"; Nomeie e finalize a criação.


 ## Criação e configuração do Target Group
 - No menu de **Load Balancing**, abaixo dele clique em **Target Groups**
 - Depois em ** Create target Group**
 - Selecione **Instances**
 - Selecione um nome 
 - Selecione a VPC criada anteriormente e o resto deixaremos como está
 - Clique em **next** e **create**
 
 ## Configuração do Load Balancer
 - Clique no menu a esquerda **Load Balancing** e depois em **Create Load Balancer**
 - Depois em **Application Load Balancer**
 - De um nome que desejar.
 - Na opção **scheme** deixe em **Internet-facing**
 - Em **IP address type** deixe em IPv4
 - Associe a VPC criada anteriormente.
 - Selecione duas AZs
 - Selecione o SG criado anteriormente e por fim confirme a criação do LB.
 
 ## Configuração do Auto Scalling
 - Clique em **Auto Scalling**
 - Depois em **Create Launch Template**
 - De um nome para o template.
 - Uma descrição rápida
 - Selecione a AMI que criamos anteriormente.
 - O tipo de instancia que desejar, eu irei utilizar a **t3.small**.
 - Selecione o seu par de chaves.
 - Selecione o SG criado anteriormente.
 - Clique em **Advanced Details** e adicione o seguinte script:
```shell
#!/bin/bash
sudo yum update -y
sudo systemctl start docker
cd /home/ec2-user/docker-compose/
docker-compose up -d
```
 - Finalize a criação do template.
 - Volte para o menu de Auto Scalling e selecione o template criado anteriormente e clique em **next**
 - Depois selecione **Attach to a new Load Balancer**
 - De um nome de sua preferencia e selecione **internet facing**
 - Verifique se a VPC é a certa e se as subnetes são as públicas das duas zonas de disponibilidade.
 - Na opção de **listeners**, crie um novo.
 - Mantenha o resto como está e clique em **next**
 - Define as capacidades que deseja. Neste caso, faremos da seguinte maneira (Capacidade desejada: 2, Capacidade mínima: 2, Capacidade máxima: 4);
 - Clique em **next** e confirme a criação do template de **Auto Scalling**
