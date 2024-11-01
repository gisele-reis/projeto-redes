# Criação de Infraestrutura de Rede

### Topologia da Rede

![image](https://github.com/user-attachments/assets/9ec07df6-7814-43fa-84a4-1af6f6b81197)

### Pré-requisitos
* Três instâncias EC2 onde o frontend está hospedado.
* Uma instância EC2 para o Load Balancer.
* Uma instância EC2 para o Proxy Reverso.
* Uma instância EC2 para o Banco de Dados.
* Uma instância EC2 onde o backend está hospedado.
  

## Configuração do Load Balancer
1. **Instalação do NGINX**
    * Acesse a máquina 4 (Load Balancer).
    * Execute o comando:
      
     <br>

     ```
   
     sudo apt update
     sudo apt install nginx
   
     ```
   

2. **Configuração do NGINX como Load Balancer**
    * Criando um novo arquivo de configuração:

     <br>

     ```

     sudo nano /etc/nginx/sites-available/load-balancer

     ```

    * Adicione a seguinte configuração:
    
     <br>

     ```
     
     upstream servidores{
        server # IP da máquina 1
        server # IP da máquina 2
        server # IP da máquina 3
     }
     
     server {
       listen 80;
     
       location / {
         proxy_pass http://servidores;
       }
     }
     
      ```

    * Habilite a configuração:

     <br>

     ```
     
     sudo ln -s /etc/nginx/sites-available/load-balancer /etc/nginx/sites-enabled/
     
     ```

 3. **Remoção da Configuração Padrão**
    * Remova o link para a configuração padrão:

     <br>

     ```

     sudo rm /etc/nginx/sites-enabled/default

     ```
     
 4. **Teste e Reinicie o NGINX**
    * Teste a configuração:

     <br>

      ```

      sudo nginx -t

      ```

    * Reinicie o NGINX:
   
     <br>

      ```

      sudo systemctl restart nginx

      ```

 ## Configuração do Proxy Reverso
 1. **Instalação do NGINX**
    * Acesse a máquina 5 (Proxy Reverso).
    * Execute o comando:
      
     <br>

      ```
   
      sudo apt update
      sudo apt install nginx
   
      ```
   

2. **Configuração do NGINX como Proxy Reverso**
    * Criando um novo arquivo de configuração:

     <br>

     ```

     sudo nano /etc/nginx/sites-available/proxy-reverse

     ```

    * Adicione a seguinte configuração, apontando para o Load Balancer:
    
     <br>

     ```
     
     server {
       listen 80;

       location / {
         proxy_pass http://<endereco-IP-da-maquina-4>;
         proxy_set_header Host $host;
         proxy_set_header X-Real-IP $remote_addr;
         proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
         proxy_set_header X-Forwarded-Proto $scheme;
       }
     
     }
     
     ```

    * Habilite a configuração:

     <br>

     ```
     
     sudo ln -s /etc/nginx/sites-available/proxy-reverse /etc/nginx/sites-enabled/
     
     ```

 3. **Remoção da Configuração Padrão**
    * Remova o link para a configuração padrão:

     <br>

     ```

     sudo rm /etc/nginx/sites-enabled/default

     ```
     
 4. **Teste e Reinicie o NGINX**
    * Teste a configuração:

     <br>

      ```

      sudo nginx -t

      ```

    * Reinicie o NGINX:
   
     <br>

      ```

      sudo systemctl restart nginx

      ```

## Configuração do Banco de Dados
1. **Instalação do Docker**
    * Acesse a máquina 6 (Banco de Dados).
    * Execute o comando:

    <br>

    ```

    sudo apt update
    sudo apt install docker.io -y

    ```
    
    * Inicie e habilite o Docker para iniciar automaticamente
  
    <br>

    ```

    sudo systemctl start docker
    sudo systemctl enable docker

    ```

2. **Configuração do PostgreSQL em um container Docker**

    * Puxe a imagem do PostgreSQL:

    <br>

    ```

    sudo docker pull postgres

    ```
    
    * Configure o nome do banco de dados, usuário e senha:
    
    <br>

    ```

    sudo docker run --name my_postgres_db -e POSTGRES_USER=login_user -e POSTGRES_PASSWORD=password123 -e POSTGRES_DB=logins_db -p 5432:5432 -d postgres

    ```
    
    * Explicação dos parâmetros:
      * `` --name my_postgres_db ``: Nome do container. 
      * `` -e POSTGRES_USER=login_user ``: Define o usuário do PostgreSQL.
      * `` -e POSTGRES_PASSWORD=password123 ``: Define a senha do usuário.
      * `` -e POSTGRES_DB=logins_db ``: Nome do banco de dados.
      * `` -p 5432:5432 ``: Mapeia a porta do container para a porta do host (EC2).
      * `` -d ``: Roda o container em segundo plano.

    <br>
    
    * Verifique se o container está rodando:

    <br>

    ```

    sudo docker ps

    ```

    > Se, por algum motivo, o container parar (não estiver listado), reinicie-o com o comando ``` docker start <container_id> ```

3. **Conectar ao PostgreSQL**

  * Acesse o container do PostgreSQL:

    <br>

    ```

    sudo docker exec -it my_postgres_db psql -U login_user -d logins_db

    ```
    
  * Crie uma tabela no banco de dados:

    <br>

    ```

    CREATE TABLE users (
      id SERIAL PRIMARY KEY,
      username VARCHAR(50),
      password VARCHAR(50)
    );

    ```

  * Insira dados na tabela:

    <br>

    ```

    INSERT INTO users (username, password) VALUES ('user1', 'password1');
    INSERT INTO users (username, password) VALUES ('user2', 'password2');

    ```

4. **Configuração do Acesso Externo**
   * Para acessar o banco de dados de fora da instância é necessário permitir o acesso externo.
   * Edite o arquivo de configuração `` postgresql.conf `` para permitir acessos externos.
   * Entre no container:

   <br>

   ```

   sudo docker exec -it my_postgres_db bash

   ```

   * Edite o arquivo de configuração:

   <br>

   ```

   nano /var/lib/postgresql/data/postgresql.conf

   ```

   * Altere o `` listen_addresses `` para permitir qualquer IP:

   <br>

   ```

   listen_addresses = '*'

   ```
