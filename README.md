# Criação de Infraestrutura de Rede

### Topologia da Rede

![image](https://github.com/user-attachments/assets/34536a57-64fe-41a4-a2f8-53ed8fa49c36)

### Pré-requisitos
* Três instâncias EC2 com NGINX instalado, onde o frontend (html) está hospedado.
* Uma instância EC2 para o Load Balancer.
* Uma instância EC2 para o Proxy Reverso.
  

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
