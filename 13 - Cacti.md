# Cacti



[toc]



## Introdução

O Cacti é uma ferramenta que obtém dados e mostra esses dados em forma de gráficos, normalmente obtendo dados de uma ou mais redes, mas podendo exibir dados do host local também. Basicamente qualquer método que possa exibir um valor de tempos em tempos pode ser adicionado ao Cacti para que se tenha um gráfico.

O Cacti é uma ferramenta feita em PHP que usa como base o RRDTool, ele quem coleta, armazena e gera os gráficos, o Cacti em sí é apenas uma interface gráfica que torna toda a administração mais fácil. Em sua maior funcionalidade, o Cacti permite carregar scripts para fazer coleta, mas seu grande funcionamento é através do snmp.

Outro grande fator que contribui para o crescimento do Cacti é o uso de plugins, com eles podemos expandir a funcionalidade e usuabilidade do Cacti. 

> Um plugin que não pode faltar é o PHP Network Weathermap, que permite criar mapas da rede e mostra o estado de cada elemento nele.



## Instalação

Existem duas formas de instalar o Cacti, uma delas é pelo repositório da distribuição e outra é instalando direto do código fonte, eu vou optar por usar o código fonte do site.

> No momento da criação desse tutorial, a versão que está no repositório do Ubuntu 20.04 é a 1.2.10, enquanto que a versão via código fonte é a 1.2.18.

Página de [Downlaod/Instalação](https://www.cacti.net/info/downloads).



### Requisitos

Antes de instalarmos o Cacti, temos que satisfazer alguns requisitos da instalação dele, os requisitos são:

-  RRDTool 1.0.49 ou mais recente, a versão 1.4 ou maior é recomendada;
-  MySQL 5.x ou mais recente;
-  PHP 5.1 ou mais recente;
-  Web Server que suporte PHP, como: Apache ou IIS.



Vamos instalar os requisitos então:

```bash
apt install -y apache2 libapache2-mod-php mysql-server php php php-gd php-gmp php-json php-ldap php-mbstring php-snmp php-xml php-zip php-gd php-gmp php-ldap php-mbstring php-mysql php-snmp php-xml rrdtool snmp snmpd

php-mc
```



### Configuração do ambiente

Antes de compilarmos a aplicação, precisamos preparar o ambiente para que tudo funcione.

```bash
# Iniciando o script do mysql_secure:
mysql_secure_installation

## Respostas:
y, 1, "SENHA", "SENHA", y, y, y, y, y
BvI1Tbx2VX6DH980piOJIGFq

# Agora precisamos criar o banco de dados:
mysql
mysql> CREATE DATABASE cacti;
mysql> CREATE USER 'cactiuser'@'localhost' IDENTIFIED WITH mysql_native_password BY 'TDfwLqYhG+2b/9Xlr3eE3bsz';
mysql> GRANT ALL PRIVILEGES ON cacti.* TO "cactiuser"@"localhost";
# Dando acesso ao usuário do Cacti poder acessar o timezones do mysql:
mysql> GRANT SELECT ON mysql.time_zone_name TO "cactiuser"@"localhost";
mysql> FLUSH PRIVILEGES;

# Agora popule a tabela de zonas no mysql (horas):
mysql_tzinfo_to_sql /usr/share/zoneinfo | mysql -u root mysql

# Crie o user do Cacti no sistema, toda a aplicação vai rodar atras desse usuário:
useradd -s /bin/bash -d /dev/null cactiuser
```



Feito isso, vamos iniciar a instalação do Cacti:

```bash
# Entre na pasta:
cd /usr/src

# Baixe o código:
wget https://files.cacti.net/cacti/linux/cacti-1.2.18.tar.gz

# Descompacte o arquivo:
tar xvf cacti-1.2.18.tar.gz

# Entre na pasta:
cd cacti-1.2.18/

# Importe o esqueleto do banco de dados para o que acabamos de criar:
mysql -u root cacti < cacti.sql

# Mude o collation do banco do Cacti:
mysql cacti
ALTER DATABASE cacti CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;

# Mude as informações de acesso ao banco de dados no arquivo abaixo:
vim include/config.php
# Se você fez todos os passos como os meus, será necessário mudar apenas a senha.
# Mude também a opção 'url_patch' deixando-a assim: $url_path = '/';
```



Agora vamos finalizar a instalação:

```bash
# Edite o serviço do Cacti, mudando a localização do diretório root dele, modifique também o user/group e local da variável 'EnvironmentFile':
vim service/cactid.service
#### No meu caso, deixei assim  ####
[Unit]
Description=Cacti Daemon Main Poller Service
After=network.target

[Service]
Type=forking
User=cactiuser
Group=cactiuser
EnvironmentFile=/etc/default/cactid
ExecStart=/var/www/cacti/cactid.php
Restart=always
RestartSec=5s

[Install]
WantedBy=multi-user.target
#### FIM #### 

# Copie o 'service/cactid.service' para '/etc/systemd/system/':
cp service/cactid.service /etc/systemd/system/

# Embora não seja necessário, em algumas ocasiões você pode precisar usar o arquivo '/etc/defaut/cactid', então crie ele:
touch /etc/default/cactid

# Recarregue o daemon do SystemD:
systemctl daemon-reload

# Copie esse dir nosso (do cacti) para '/var/www/':
cd ..
cp -r cacti-1.2.18 /var/www/cacti

# Mude o dono/grupo de tudo para o usuário do cacti:
chown cactiuser. -R /var/www/cacti

# Permite que o apache escreva em apenas algumas pastas apenas durante a instalação:
## chown -R cactiuser rra/ log/ cache/
chown -R www-data.www-data /var/www/cacti/resource/snmp_queries/
chown -R www-data.www-data /var/www/cacti/resource/script_server/
chown -R www-data.www-data /var/www/cacti/resource/script_queries/
chown -R www-data.www-data /var/www/cacti/scripts/
chown -R www-data.www-data /var/www/cacti/log/
chown -R www-data.www-data /var/www/cacti/rra/
chown -R www-data.www-data /var/www/cacti/cache/boost/
chown -R www-data.www-data /var/www/cacti/cache/mibcache/
chown -R www-data.www-data /var/www/cacti/cache/realtime/
chown -R www-data.www-data /var/www/cacti/cache/spikekill/

# Inicie e habilite o Cacti no boot do Sistema:
systemctl start cactid
systemctl enable cactid

# Verificando se tudo está em ordem:
systemctl status cactid
● cactid.service - Cacti Daemon Main Poller Service
     Loaded: loaded (/etc/systemd/system/cactid.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2021-08-02 15:12:14 UTC; 30s ago
   Main PID: 11997 (php)
      Tasks: 1 (limit: 2305)
     Memory: 11.8M
     CGroup: /system.slice/cactid.service
             └─11997 php /var/www/cacti/cactid.php

Aug 02 15:12:14 cacti systemd[1]: Starting Cacti Daemon Main Poller Service...
Aug 02 15:12:14 cacti cactid.php[11983]: Starting Cacti Daemon ... [OK]
Aug 02 15:12:14 cacti systemd[1]: Started Cacti Daemon Main Poller Service.
```

> Como o Cacti passou a usar o SystemD, o arquivo que antes ficava rodando no crontab hoje não é mais necessário.
>
> 
>
> $ cat service/README.md | grep -A1 ^7
> 7. Comment out the cacti crontab file or simply remove the /etc/cron.d/cacti
>    file in place today



#### Proteção de lista de permissão de validação de entrada

Os métodos de entrada de dados do Cacti que chamam um script podem ser explorados de maneiras que um não administrador pode danificar os arquivos pertencentes à conta do poller e, nos casos em que alguém executa o poller do Cacti como root, pode comprometer o sistema operacional, permitindo que os invasores explorar sua infraestrutura.

Portanto, várias versões atrás, o Cacti foi aprimorado para fornecer recursos de lista branca nesses tipos de métodos de entrada de dados. Embora isso proteja os Cacti mais profundamente, aumenta a quantidade de trabalho exigida pelo administrador do Cacti para importar e gerenciar modelos e pacotes.

A maneira como a Whitelisting funciona é que quando você importa um Método de entrada de dados pela primeira vez, ou reimporta um Método de entrada de dados, e o script e / ou agumentos mudam de alguma forma, o Método de entrada de dados e todas as fontes de dados correspondentes ser desativado imediatamente até que o administrador valide que o Método de entrada de dados é válido.

Para identificar os métodos de entrada de dados neste estado, fornecemos um script de validação no diretório CLI do Cacti que pode ser executado com as seguintes opções:

```bash
php -q input_whitelist.php --audit 
# This script option will search for any Data Input Methods that are currently banned and provide details as to why.

php -q input_whitelist.php --update 
# This script option un-ban the Data Input Methods that are currently banned.

php -q input_whitelist.php --push 
# This script option will re-enable any disabled Data Sources.

# Para ativar, descomente a seguinte linha do arquivo '/var/www/cacti/include/config.php':
$input_whitelist = '/usr/local/etc/cacti/input_whitelist.json';
```


### Configurando o apache

Se estivesse colocado o diretório do Cacti em `/var/www/html`, poderia acessar o Cacti usando o pelo navegador assim `http://192.168.121.150/cacti`, mas como não coloquei, vou criar um vhost para nossa página do Cacti.

```bash
vim /etc/apache2/sites-available/cacti.conf
#### CONTEÚDO ####
<VirtualHost *:80>
        ServerAdmin root@localhost
        DocumentRoot /var/www/cacti

        <Directory /var/www/cacti>
                Options Indexes FollowSymLinks
                AllowOverride None
                Require all granted
        </Directory>

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
#### FIM CONTEÚDO ####

# Agora ative nosso vhost e desative o vhost padrão:
a2dissite 000-default.conf
a2ensite cacti.conf

# Agora reinicie o apache:
systemctl restart apache2
```



## Tunning do MySQL

Para que o mysql opere sem problemas, vamos fazer o tunning dele:

```bash
# Edite o arquivo abaixo e adicione as informações fornecidas para fazer o tunning do mysql:
vim /etc/mysql/my.cnf

#### INÍCIO DO CONTEÚDO ####
[mysqld]
collation_server = utf8mb4_unicode_ci
character_set_server = utf8mb4

innodb_buffer_pool_size = 1G
innodb_log_buffer_size = 256M
innodb_log_file_size = 1G
innodb_read_io_threads = 32
innodb_write_io_threads = 16
innodb_buffer_pool_instances = 2
innodb_flush_log_at_timeout = 3
innodb_buffer_pool_instances = 13
innodb_io_capacity = 5000
innodb_io_capacity_max = 10000

join_buffer_size = 100M
tmp_table_size = 100M
max_heap_table_size = 100M

long_query_time = 1
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slowquery.log
log_queries_not_using_indexes = 1
#### FIM DO CONTEÚDO ####

# Reinicie o Mysql:
systemctl restart mysql
```



#### Error: A valid timezone that matches MySQL and the system

Para esse erro, altere o timezone na cli e na parte web do php:

```bash
# php no apache
vim /etc/php/7.4/apache2/php.ini
[Date]
; Defines the default timezone used by the date functions
; http://php.net/date.timezone
date.timezone = America/Sao_paulo

# Ainda nesse arquivo, altere também, os valores de: 'memory_limit' e 'max_execution_time':
max_execution_time = 60
memory_limit = 400M
## Esses são valores padrões!

vim /etc/php/7.4/cli/php.ini
[Date]
; Defines the default timezone used by the date functions
; http://php.net/date.timezone
date.timezone = America/Sao_paulo

# Reinicie o apache:
systemctl restart apache2
```



Agora acesse o navegador `http://IP`, quando aparecer o site do Cacti, o usuário é `admin` e senha `admin`. Faça a instalação via interface gráfica, para qualquer erro que possa ter, veja a Sessão [Debug](#Debug) mais adiante.

```bash
nova senha:

hiUO7DEpOQKfIj2j092f/Z5l
```



## Após finalizar a instalação via Web

Agora que a instalação foi finalizada, temos que rodar os comandos abaixo para criar a whitelist:

```bash
# Crie a pasta do Cacti no local indicado:
mkdir /usr/local/etc/cacti

# Entre na pasta:
cd /var/www/cacti/cli/

# Rode o update para criar o arquivo:
php -q input_whitelist.php --update

# Agora rode o audit:
php -q input_whitelist.php --audit

# Volte as permissoes como devam ser:
chown -R cactiuser.cactiuser /var/www/cacti/
chown -R cactiuser.www-data /var/www/cacti/resource/snmp_queries/
chown -R cactiuser.www-data /var/www/cacti/resource/script_server/
chown -R cactiuser.www-data /var/www/cacti/resource/script_queries/
chown -R cactiuser.www-data /var/www/cacti/scripts/
chown -R cactiuser.www-data /var/www/cacti/log/
chown -R cactiuser.www-data /var/www/cacti/rra/
chown -R cactiuser.www-data /var/www/cacti/cache/boost/
chown -R cactiuser.www-data /var/www/cacti/cache/mibcache/
chown -R cactiuser.www-data /var/www/cacti/cache/realtime/
chown -R cactiuser.www-data /var/www/cacti/cache/spikekill/
```





#### When using Cacti with languages other than English, it is important to  use the utf8mb4_unicode_ci collation type as some characters take more  than a single byte.

ALTER TABLE search_index CONVERT TO CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci;

ALTER DATABASE cacti CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;



Ver o charset + collation do database:

SELECT SCHEMA_NAME 'database', default_character_set_name 'charset', DEFAULT_COLLATION_NAME 'collation' FROM information_schema.SCHEMATA;

Muda o charset da tabela:

ALTER DATABASE cacti CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;







## Fontes

https://www.mediawiki.org/wiki/Manual:Wiki_family
https://www.mediawiki.org/wiki/Manual:Installation_guide
https://www.mediawiki.org/wiki/Manual:Installation_requirements
https://sharkysoft.com/wiki/how_to_configure_multiple_MediaWiki_instances_on_a_single_host
https://github.com/netniV/cacti/wiki/Testing-Environment:-CentOS#change-default-permissions-1
https://docs.cacti.net/Requirements.md
https://docs.cacti.net/