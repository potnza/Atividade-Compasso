#  :tw-270f: Atividade - AWS - Docker - Senac

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
-Não utilizar ip público para saída do serviços WP (Evitem publicar o serviço WP via IP Público)
- Sugestão para o tráfego de internet sair pelo LB (Load Balancer)
- Pastas públicas e estáticos do wordpress sugestão de utilizar o EFS (Elastic File System)
- Fica a critério de cada integrante (ou grupo) usar Dockerfile ou Docker-compose;
- Necessário demonstrar a aplicação wordpress funcionando
- (tela de login)
- Aplicação Wordpress precisa estar rodando na porta 80 ou
- 8080;
- Utilizar repositório git para versionamento;
- Criar documentação.
