## Instalar o mediawiki

Exite duas formas de instalar o mediawiki, uma delas é através do repositório oficial da distribuição e outra é atraves do código fonte.



### Instalando pelo código fonte

 

```bash
# Atulize o repo:
sudo apt update

# Instale as dependencias:
sudo apt -y install php php-apcu php-intl php-mbstring php-xml php-mysql mariadb-server apache2 imagemagick

# Baixe o mediawiki:
sudo wget https://releases.wikimedia.org/mediawiki/1.36/mediawiki-1.36.1.zip

# Descompacte-o:
sudo unzip mediawiki-1.36.1.zip

# Renomeie ele:
sudo mv mediawiki-1.36.1 mediawiki

# Mova para outro diretório:
sudo mv mediawiki /usr/share/

# Crie um dir no /etc:
sudo mkdir /etc/mediawiki

# Crie a pasta onde vao ficar as Wikis:
sudo mkdir -p /var/lib/wikis/taskforce171/

# Entre na pasta:
cd /var/lib/wikis/taskforce171/

# Crie os links
ln -s /usr/share/mediawiki/* .

# Apague alguns arquivos/pastas:
rm LocalSettings.php AdminSettings.php images config

# Crie as pastas:
sudo cp -a /var/lib/mediawiki/config .
```



## Criando o Banco de Dados

Agora vamos criar o banco de dados que nossa Wiki vai usar.

```bash
# Entre no Mysql (mariadb):
mysql 

# Crie o banco:
CREATE DATABASE brunowiki;

# Crie o usuário:
CREATE USER 'brunowikiuser'@'localhost' IDENTIFIED BY 'for1inWiK1D0';

# De permissão para o usuário no banco que criamos:
GRANT ALL PRIVILEGES ON brunowiki.* TO 'brunowikiuser'@'localhost' WITH GRANT OPTION;

# Recarregue as permissões:
FLUSH PRIVILEGES;
```



## Apache

Vamos configurar o apache.

```bash
cat /etc/apache2/sites-enabled/bruno.conf 
<VirtualHost *:80>
        ServerAdmin bruno@wiki.teste.com.br
        ServerName brunowiki.teste.com.br
        DocumentRoot /var/lib/wikis/bruno

	<Directory /var/lib/wikis/bruno/>
	    	Options +FollowSymLinks
	        AllowOverride All
	        <IfVersion >= 2.3>
	                Require all granted
	        </IfVersion>
	        <IfVersion < 2.3>
	                order allow,deny
	                allow from all
	        </IfVersion>
	</Directory>
	
	<Directory /var/lib/wikis/bruno/config>
	        Options -FollowSymLinks
	        AllowOverride None
	    <IfModule mod_php5.c>
	        php_admin_flag engine off
	    </IfModule>
	</Directory>
	
	<Directory /var/lib/wikis/bruno/images>
	        Options -FollowSymLinks
	        AllowOverride None
	    <IfModule mod_php5.c>
	        php_admin_flag engine off
	    </IfModule>
	</Directory>
	
	<Directory /var/lib/wikis/bruno/upload>
	        Options -FollowSymLinks
	        AllowOverride None
	    <IfModule mod_php5.c>
	        php_admin_flag engine off
	    </IfModule>
	</Directory>


        ErrorLog /var/log/apache2/error-bruno.log
    
        # Possible values include: debug, info, notice, warn, error, crit,
        # alert, emerg.
        LogLevel warn
    
        CustomLog /var/log/apache2/access-bruno.log combined
    
        AddDefaultCharset UTF-8

</VirtualHost>
```



```bash
cat /etc/apache2/sites-enabled/000-default.conf
<VirtualHost *:80>

	ServerName wiki.teste.com.br
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html/

	<Directory />
                Options FollowSymLinks
                AllowOverride None
        </Directory>

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

Include /etc/mediawiki/apache.conf

</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet
```



```bash
cat /etc/mediawiki/apache.conf 
# Uncomment this to add an alias.
# This does not work properly with virtual hosts..
Alias /mediawiki /var/lib/mediawiki

<Directory /var/lib/mediawiki/>
        Options +FollowSymLinks
        AllowOverride All
        <IfVersion >= 2.3>
                Require all granted
        </IfVersion>
        <IfVersion < 2.3>
                order allow,deny
                allow from all
        </IfVersion>
</Directory>

# some directories must be protected
<Directory /var/lib/mediawiki/config>
        Options -FollowSymLinks
        AllowOverride None
    <IfModule mod_php5.c>
        php_admin_flag engine off
    </IfModule>
</Directory>
<Directory /var/lib/mediawiki/images>
        Options -FollowSymLinks
        AllowOverride None
    <IfModule mod_php5.c>
        php_admin_flag engine off
    </IfModule>
</Directory>
<Directory /var/lib/mediawiki/upload>
        Options -FollowSymLinks
        AllowOverride None
    <IfModule mod_php5.c>
        php_admin_flag engine off
    </IfModule>
</Directory>


# Wiki bruno
Alias /bruno /var/lib/wikis/bruno

<Directory /var/lib/wikis/bruno/>
        Options +FollowSymLinks
        AllowOverride All
        <IfVersion >= 2.3>
                Require all granted
        </IfVersion>
        <IfVersion < 2.3>
                order allow,deny
                allow from all
        </IfVersion>
</Directory>

# some directories must be protected
<Directory /var/lib/wikis/bruno/config>
        Options -FollowSymLinks
        AllowOverride None
    <IfModule mod_php5.c>
        php_admin_flag engine off
    </IfModule>
</Directory>
<Directory /var/lib/wikis/bruno/images>
        Options -FollowSymLinks
        AllowOverride None
    <IfModule mod_php5.c>
        php_admin_flag engine off
    </IfModule>
</Directory>
<Directory /var/lib/wikis/bruno/upload>
        Options -FollowSymLinks
        AllowOverride None
    <IfModule mod_php5.c>
        php_admin_flag engine off
    </IfModule>
</Directory>
```





Acesse a página.





Criar varias Wikis:

```bash
# cat /etc/mediawiki/LocalSettings.php 
<?php
// Include common settings to all wikis before this line (eg. database configuration)

        switch ( $_SERVER['SERVER_NAME'] ) {
                case 'maddogswiki.sysnetbr.eng.br':
                        require_once '/etc/mediawiki/LocalSettings_maddogs.php';
                        break;

                case 'taskforce171wiki.sysnetbr.eng.br':
                        require_once '/etc/mediawiki/LocalSettings_taskforce171.php';
                        break;

                default:
                        header( 'HTTP/1.1 404 Not Found' );
                        echo 'This wiki is not available. Check configuration.';
                        exit( 0 );
        }

```









## Wiki no NIC

```bash
# Entre no Mysql (mariadb):
mysql 

# Crie o banco:
CREATE DATABASE juridicowiki;

# Crie o usuário:
CREATE USER 'juridicowikiuser'@'localhost' IDENTIFIED BY 'rpeT1WZumgFZrwjI7VqDxKSK';

# De permissão para o usuário no banco que criamos:
GRANT ALL PRIVILEGES ON juridicowiki.* TO 'juridicowikiuser'@'localhost' WITH GRANT OPTION;

# Recarregue as permissões:
FLUSH PRIVILEGES;

cd /usr/share/mediawiki
mv LocalSettings.php LocalSettings.php.bkp

mkdir /var/lib/wikis/<NOME>
cd /var/lib/wikis/<NOME>

ln -s /usr/share/mediawiki/* .
rm LocalSettings.php AdminSettings.php images config
sudo cp -a /var/lib/mediawiki/config .
sudo mkdir images
    chown www-data. {images,config}

ln -s /etc/mediawiki/LocalSettings_juridico.php LocalSettings.php

# Para uma pagina com sub, ex.: /juridico
vim /etc/mediawiki/apache.conf

apachectl -k graceful

# Acessar a nova página para configuração

# Depois de configurado:
cd /usr/share/mediawiki
mv LocalSettings.php.bkp LocalSettings.php

Fabricio Torres (ftorres@nic.br)
Nádilla (nadilla@nic.br)
Lucas Ferreira (lucasferreira@nic.br)


## incluindo usuários, ex. sysadminwiki

cd /usr/share/mediawiki/maintenance
cfg=/etc/mediawiki/LocalSettings_cetic.php
 user_login='lucasferreira'
 pass_login='MudarPorFavor123'

php createAndPromote.php --conf=${cfg} ${user_login} ${pass_login}

## trocando a senha

php changePassword.php --conf=${cfg} --user=${user_login} --password=${pass_login}
```





## Fontes

https://www.mediawiki.org/wiki/Manual:Wiki_family
https://www.mediawiki.org/wiki/Manual:Installation_guide

https://www.mediawiki.org/wiki/Manual:Installation_requirements

https://sharkysoft.com/wiki/how_to_configure_multiple_MediaWiki_instances_on_a_single_host

