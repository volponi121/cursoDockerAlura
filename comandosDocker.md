##Docker

##Comandos

#Mostrar Containers
 * Ativos: 
   - docker ps
 * Criados e parados:
   - docker ps -a
#Conectar no container
 * ex. Ubuntu
   - docker run -it <NomeImagem>
#Startar o Container
 * Normal
   - docker start <ContainerID>
 * Conectando INPUT/OUTPUT (Modo ATTACH)
   - docker start -a -i <ContainerID>
 * Attach
   - -a (STDOUT/STDER, Standart Output do terminal,Standart Error do terminal)
   - -i (STDIN, Standart Input do terminal)
# Parar o Container                              
   - docker stop <ContainerID>
   - docker stop -t 0 <ContainerID>
   - -t (O tempo para matar o container, o default é 10 segundos) 
# Remover o Container do computador
  * Unitário
   - docker rm <ContainerID>
  * Todos inativos
   - docker container prune
# Remover a imagem do computador
  - docker rmi <ContainerID>   
# Dando run na imagem baixada sem travar o terminal
  - docker run -d <Imagem DockerHub ex. dockersamples/static-site>
# Linkar uma porta interna do container para meu computador
  - docker run -d -P dockersamples/static-site
  - -P (O seu host poder falar com o seu container, o docker mapeia automatico)
  - docker run -d -p ipMeuComputador:80 dockersamples/static-site
  - -p (Define a porta que você quer utilizar manualmente)
# Visualizar portas do container
  - docker port <ContainerID>
# Dando o nome do container no docker run
  - docker run -d -P --name meu-site dockersamples/static-site
  - --name (Coloca o nome que você quer no container e a imagem dele)
# Dando o nome do autor no container no docker run
  - docker run -d -P -e AUTHOR="Fabio Volponi" --name helloFabio dockersamples/static-site
# Dando stop em todos containers containers
  - docker ps -q (Retorna apenas os id's)
  - docker stop $(docker ps -q)

## Volumes

# Criando o volume onde vai salvar arquivos do container
  - docker run -it -v "/home/fabio:/var/www" ubuntu
  - -v (Volume)
# Startando uma aplicacao em node no container
  - docker run -d -p 8080:3000 -v "$(pwd):/var/www" -w "/var/www" node npm start
  - docker run <-d terminal não trava> <-p porta> <pwd é o caminho da aplicação na maquina, var/www é o caminho no container> <-w especifica o caminho no container> <Imagem> <Comando>


## Criando a imagem
.dockerfile

FROM node:latest   //A imagem e sua versão
LABEL MAINTAINER  Fabio // Nome de quem da manutenção na imagem
COPY . /var/www  // Copia todos arquivos(.) da pasta atual para o /var/www do container
WORKDIR /var/www // Diretorio que vai ser trabalhado
RUN npm install // Comando quando o container é startado
ENTRYPOINT [ "npm", "start"] // Mesma coisa que o de cima
EXPOSE 3000 // Expôe a porta que o container vai expor


## Criando a rede para os containers

 # Rede Default
  - Já tem ip default, não permite atribuir o hostname a determinado container.
 # Criar a rede
  - docker network create --driver bridge <nome-da-rede>
 # Colocando ping 
  - apt update && apt install iputils-ping
  - ping <nome-container>


## Docker Compose

# docker-compose.yml

# version: '3' // Versão do yaml 
# services: // Serviçõs
# nginx: //Nome do serviço  
#    build: //configuração do build
      dockerfile: ./docker/nginx.dockerfile
      context: . 
#    image: douglasq/nginx // Imagem
#    container_name: nginx // Nome do seu container
#    ports:  // Lista de portas que você vai utilizar
     - "80:80"
#    networks: // Lista de redes que você vai utilizar 
     - production-network 
#    depends_on: // Dependências para construir o build(Ou seja tem que ter node1...node3 primeiro para construir). 
      - "node1" 
      - "node2"
      - "node3"
  
  mongodb:
    image: mongo
    networks: 
      - production-network

  node1:
    build:
     dockerfile: ./docker/alura-books.dockerfile
     context: .
    image: douglasq/alura-books
    container_name: alura-books-1
    ports: 
      - "3000"
    networks: 
      - production-network
    depends_on: 
      - "mongodb"   

  node2:
    build:
     dockerfile: ./docker/alura-books.dockerfile
     context: .
    image: douglasq/alura-books
    container_name: alura-books-2
    ports: 
      - "3000"
    networks: 
      - production-network
    depends_on: 
    - "mongodb"    

  node3:
    build:
     dockerfile: ./docker/alura-books.dockerfile
     context: .
    image: douglasq/alura-books
    container_name: alura-books-3
    ports: 
      - "3000"
    networks: 
      - production-network 
    depends_on: 
      - "mongodb" 

# networks: // Onde você cria suas redes
  production-network: 
    driver: bridge

