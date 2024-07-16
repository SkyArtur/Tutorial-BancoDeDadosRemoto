# Configurando acesso remoto para banco de dados

Requisitos:
- Sistema Operacional Linux Ubuntu:
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
### PostgreSQL
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


