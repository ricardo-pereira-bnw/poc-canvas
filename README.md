# Poc Canvas

## Passos para instalação do zero

### Preparação do Docker

Criação do arquivo 'docker-compose.yml':

```
version: "3.1"
services:
  php:
    image: ricardopedias/docker-project:php80
    container_name: canvas
    volumes:
      - .:/application
    ports:
      - "8088:80"
    networks:
      - canvas-network
  mysql:
    image: ricardopedias/docker-project:mysql80
    container_name: canvas-db
    volumes:
      - ~/.bnw/projects/canvas/storage/mysql:/var/lib/mysql
    environment:
      - MYSQL_ROOT_PASSWORD=devel
      - MYSQL_DATABASE=devel
      - MYSQL_USER=devel
      - MYSQL_PASSWORD=devel
    healthcheck:
      test: ["CMD", "mysqladmin" ,"ping", "-h", "localhost"]
      timeout: 20s
      retries: 10
    ports:
      - "8089:3306"
    networks:
      - canvas-network
networks:
  canvas-network:
    driver: bridge
```

Configurar o conteiner:

```
# subir os conteiners em primeiro plano, para ver as informações na primeira vez
docker-compose up -d
```

Os conteiners serão levantados e o banco de dados será gerado automaticamente.
Nesta primeira execução, é importante usar o comando acima (sem o argumento '-d') 
e aguardar até que o mysql tenha sido iniciado completamente.

Você saberá disso quando ver uma mensagem parecida com esta:

```
canvas-db | 2021-10-11T11:27:28.143832Z 0 [System] [MY-010931] [Server] /usr/sbin/mysqld: ready for connections. Version: '8.0.24'  socket: '/var/run/mysqld/mysqld.sock'  port: 3306  MySQL Community Server - GPL.
```

### Criação do projeto Laravel

Criação de novo projeto Laravel:

```
# subir os conteiners em plano de fundo
docker-compose up -d

# entrar no terminal da aplicação PHP
docker exec -it canvas bash

# move o arquivo docker-compose.yml para permitir ao composer fazer instalações
mv docker-compose.yml /docker-compose.yml

# instala um novo projeto Laravel
composer create-project laravel/laravel .

# volta o arquivo docker-compose.yml
mv /docker-compose.yml .
```

Setar as permissões corretas:

```
chmod -Rf 777 storage bootstrap/cache
```

Configurar a conexão com o banco de dados

No arquivo '.env', defina as segites informações de conexão:

```
DB_CONNECTION=mysql
DB_HOST=canvas-db
DB_PORT=3306
DB_DATABASE=devel
DB_USERNAME=devel
DB_PASSWORD=devel
```

### Instalação do canvas

```
composer require austintoddj/canvas
php artisan canvas:install
```

O terminal exibirá o email e a senha padrão para acesso à aplicação 'canvas', como no exemplo abaixo:

```
Installation complete.
+-------------------+------------------+
| Default Email     | Default Password |
+-------------------+------------------+
| email@example.com | password         |
+-------------------+------------------+
First things first, head to http://localhost/canvas/login and update your credentials.
```


## Passos para instalação deste projeto

```
docker exec -it canvas bash
composer install
php artisan canvas:install
```