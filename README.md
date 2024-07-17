# Configurando acesso remoto para banco de dados

Requisitos:
- Linux Ubuntu:
  - [WSL - Windows Subsystem for Linux](https://learn.microsoft.com/pt-br/windows/wsl/install#install-wsl-command)
  - [VirtualBox](https://www.virtualbox.org/wiki/Downloads) + [ISO Ubuntu Server](https://ubuntu.com/download/server)
- SGBD - Sistema de Gerenciamento de Banco de Dados
  -  [MySQL](https://www.mysql.com/)
  - [PostgreSQL](https://www.postgresql.org/)

### WSL
- #### Instalação:
```shell
wsl --install -d Ubuntu
```
- #### Verificando a versão instalada:
```shell
 cat /etc/os-release | grep VERSION_ID
```
Realize a atualização do sistema:
```shell
sudo apt update && sudo apt upgrade -y
```
- #### Conferindo se o *systemd* está habilitado:
```shell
sudo nano /etc/wsl.conf
```
```
[boot]
systemd=true
```

### Linux
- #### Atualizando:
```shell
sudo apt update && sudo apt upgrade -y
```
- #### Realizando instalações:
```shell
sudo apt install postgresql postgresql-contrib mysql-server
```
# PostgreSQL
- #### Iniciando o service:
```shell
sudo service postgresql start
```
- #### Alterando o arquivo postgresql.conf:
*OBS.: substitua **{version}** pelo número da versão instalada, ex.: 14, 16, etc.*
```shell
sudo nano /etc/postgresql/{version}/main/postgresql.conf
```
*Remova a cerquilha(#) de **listen_addresses** e altere 'localhost' para ' * '.*
```shell

#------------------------------------------------------------------------------
# CONNECTIONS AND AUTHENTICATION
#------------------------------------------------------------------------------

# - Connection Settings -

listen_addresses = '*'                  # what IP address(es) to listen on;
                                        # comma-separated list of addresses;
                                        # defaults to 'localhost'; use '*' for all
                                        # (change requires restart)
```
- #### Alterando o arquivo pg_hba.conf:
```shell
sudo nano /etc/postgresql/{version}/main/pg_hba.conf
```
*Edite a conexão IPv4 para **0.0.0.0/0**.*
```shell
# Database administrative login by Unix domain socket
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD

# "local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             0.0.0.0/0               scram-sha-256
# IPv6 local connections:
host    all             all             ::1/128                 scram-sha-256
# Allow replication connections from localhost, by a user with the
# replication privilege.
local   replication     all                                     peer
host    replication     all             127.0.0.1/32            scram-sha-256
host    replication     all             ::1/128                 scram-sha-256
```
- #### Altere a senha do usuário postgres:
```shell
sudo -u postgres psql
```
No Shell do PostgreSQL, digite:
```shell
postgres=# ALTER ROLE postgres PASSWORD 'password';
```
Para sair do shell do postgresql:
```shell
postgres=# \q
```
- #### Reiniciar o serviço:
```shell
sudo service postgresql restart && sudo systemctl restart postgresql
```
- #### Verificar o endereço IP no WSL:
```shell
ip addr show eth0 | grep inet | awk '{print $2;}' | sed 's/\/.*$//'
```
- #### Verificar o endereço IP na VM:
```shell
ip addr show enp0s3 | grep inet | awk '{print $2;}' | sed 's/\/.*$//'
```
# Mysql
- #### Iniciando o service:
```shell
sudo systemctl start mysql.service
```

- #### Configurando o usuário root e a instalação segura do mysql:
```shell
sudo mysql
```
Alterando a senha do usuário root:
```shell
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY 'password';
```

Saindo do shell do MySQL:
```shell
mysql> exit
```

- #### Rodando o script de segurança:
```shell
sudo mysql_secure_installation
```
Nesta etapa é interessante que se tenha a atenção para alterar a opção que permite que o usuário root realize
conexões remotas. Para as demais, pode-se ignorar:

```shell
Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.

Disallow root login remotely? (Press y|Y for Yes, any other key for No) : y
Success.
```
- #### Retornando o método de autenticação padrão do usuário root:
```shell
mysql -u root -p
```
Alterando o método de autenticação do root:
```shell
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH auth_socket;
```
Criando um usuário para utilização:
```shell
mysql> CREATE USER 'usuario'@'%' IDENTIFIED BY 'password';

mysql> GRANT ALL PRIVILEGES ON *.* TO 'usuario'@'%' WITH GRANT OPTION;

mysql> FLUSH PRIVILEGES;

mysql> exit
```
- #### Alterando o arquivo mysqld.cnf:
```shell
sudo nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
*Edite bind-address para **0.0.0.0**.*
```shell
# Instead of skip-networking the default is now to listen only on
# localhost which is more compatible and is not less secure.
bind-address            = 0.0.0.0
mysqlx-bind-address     = 127.0.0.1
```
#### - Reinicie o serviço:
```shell
sudo systemctl restart mysql.service
```
# Docker
Utilizando uma VM para servir os bancos de dados a partir de containers Docker:
- [Instalar o Docker](https://docs.docker.com/engine/install/ubuntu/)

## Container PostgreSQL
- Docker Compose:
```yaml
services:
  postgres:
    container_name: postgres
    image: postgres:16.1
    restart: always
    environment:
      - POSTGRES_DB=postgres
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=212223
      - TZ=America/Sao_Paulo
    ports:
      - "5432:5432"
    volumes:
      - pgData:/var/lib/postgresql/data/
volumes:
  pgData:
    name: 'pgData'
  ```
```shell
docker compose up -d
```
- Comando:
```shell
docker run --name postgres -e POSTGRES_PASSWORD='212223' -d -p 5432:5432 -v pgData:/var/lib/postgresql/data postgres:16.3
```

- Executar bash do container:
```shell
docker exec -it postgres bash
```
- Atualizando a distro do container:
```shell
root@be1fa4b5e7b7:/# echo "host all all 0.0.0.0/0 md5" >> /var/lib/postgresql/data/pg_hba.conf
```
- Reiniciando o container:
```shell
docker restart postgres
```
- Abrindo a porta 5432:
```shell
sudo ufw allow 5432/tcp
```

## Container MySQL
- Comando:
```shell
docker run --name mysql -e MYSQL_ROOT_PASSWORD='212223' -d -p 3306:3306 -v myData:/var/lib/mysql mysql:8.4
```
- Docker Compose:
```yaml
services:
  mysql:
    image: mysql:8.4
    container_name: mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=212223
    ports:
      - "3306:3306"
    volumes:
      - myData:/var/lib/mysql/
volumes:
  myData:
    name: 'myData'
  ```
```shell
docker compose up -d
```