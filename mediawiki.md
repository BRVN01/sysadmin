[toc]



## Instalar o mediawiki

Exite duas formas de instalar o mediawiki, uma delas é através do repositório oficial da distribuição e outra é atraves do código fonte. No dia em que esse tutorial está sendo feito, o pacote mais atual no repositório do Ubuntu 20.04 é a versão `1.31.7` e a versão do código fonte mais atual é a `1.36.1` (estável).



### O que você precisa saber antes de instalar sua Media Wiki

Caso você deseje usar mais de uma Wiki usando a mesma instancia (uma única instalação do Media Wiki) no mesmo servidor, hoje em dia existem 3 formas, sendo uma delas uma gamb para deixar seu funcionamento como antigamente.

- **Drupal Style**
  Funciona somente com Subpages, como: `yourdomain.com/wiki1`, `yourdomain.com/wiki2` e etc. [Link](https://www.mediawiki.org/wiki/Manual:Wiki_family/Drupal-style_LocalSettings.php) para pegar o exemplo de configuração do método Drupal.

- **Giant Switch Statement**
  Você pode escolher entre usar com `vhost` (Virtual Hosts do Apache) ou usar com `Subpage`, aparentemente usar os dois métodos ao mesmo tempo não funciona.

- **Várias instancias**

  Para burlar isso os métodos acima, e poder usar `vhost` ou `subpage` não deve ser usado link simbólicos como era feito em versões mais antigas do Media Wiki, temos que copiar um diretório para cada wiki, ná prática você está criando várias instancias (várias wikis, cada uma com seu próprio código fonte).



A própria Media Wiki informa que o método de Link Simbólico não funciona mais:

> Fonte: https://m.mediawiki.org/wiki/Manual:Wiki_family
>
> Symlink method
>
> The symlinks method was formerly a method that eliminated the need to have redundant copies of MediaWiki on your filesystem while providing flexibility and customization for each wiki. WARNING: As of recent MediaWiki versions, this doesn't work anymore. If you try this, your MediaWiki will break. Instead, it's highly recommended to follow its spiritual successor, the method used by the Wikimedia Foundation (and the method that's used to power Wikipedia, this wiki, and many more!).



#### Usando o método Giant Switch Statement

Crie um arquivo `LocalSettings.php` apenas e adicione o conteúdo abaixo, alterando os domínios de cada Wiki.

```bash
#### Giant Switch Statement com VHOSTS ####

# Crie o arquivo padrão:
vim /etc/mediawiki/LocalSettings.php 

### CONTEÚDO ###
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
### FIM CONTEÚDO ###

# ---------------------------------------------------------------------------------------- #


#### Giant Switch Statement com SUB-PAGES ####

# Crie o arquivo padrão:
vim /etc/mediawiki/LocalSettings.php 

### CONTEÚDO ###
<?php
// Include common settings to all wikis before this line (eg. database configuration)

$callingurl = strtolower( $_SERVER['REQUEST_URI'] ); // get the calling url
if ( strpos( $callingurl, '/maddogs' )  === 0 ) {
        require_once '/etc/mediawiki/LocalSettings_maddogs.php';
} elseif ( strpos( $callingurl, '/taskforce171' ) === 0 ) {
        require_once '/etc/mediawiki/LocalSettings_taskforce171.php';
} else {
        header( 'HTTP/1.1 404 Not Found' );
        echo "This wiki (\"" . htmlspecialchars( $callingurl ) . "\") is not available. Check configuration.";
        exit( 0 );
}
### FIM CONTEÚDO ###
```

Você ainda deve configurar `$wgUploadDirectory = "/var/lib/wikis/<NOME>/images";` em cada `LocalSettings.php` de suas Wikis para não ter problema com o Uploads de conteúdos.



#### Usando o método Drupal Style

Obtenha o código [nesse link](https://www.mediawiki.org/wiki/Manual:Wiki_family/Drupal-style_LocalSettings.php), crie o arquivo em `/etc/mediawiki/LocalSettings.php` contendo esse código.

Na variável `$confdir`, coloque o diretório de onde estão as suas Wikis, por exemplo:

```bash
# A localização dos diretórios das minhas Wikis:
ls /var/lib/wikis
maddogs  taskforce171

# Então minha variável fica assim:
$confdir = '/var/lib/wikis';
```

Agora você deve criar a  configuração do `apache` para apontar para a subpage.

Você ainda deve configurar `$wgUploadDirectory = "/var/lib/wikis/<NOME>/images";` em cada `LocalSettings.php` de suas Wikis para não ter problema com o Uploads de conteúdos.






### Instalando pelo código fonte

Vamos ver como instalar usando o código fonte. 

```bash
# Atulize o repo:
sudo apt update

# Instale as dependencias:
sudo apt -y install php php-apcu php-json php-intl php-mbstring php-xml php-mysql mariadb-server apache2 imagemagick

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
sudo ln -s /usr/share/mediawiki/* .

# Apague alguns arquivos/pastas, não podemos ter eles como links:
rm LocalSettings.php AdminSettings.php images config

# Crie o LocalSettings da sua Wiki (apenas o link simbólico):
ln -s /etc/mediawiki/LocalSettings_taskforce171.php LocalSettings.php

# Crie as pastas:
sudo mkdir {images,upload,config}

# De as permissões corretas apenas para os arquivos abaixo:
sudo chown www-data. -R {images,config,upload}
```



### Instalando usando o repositório

No caso do Ubuntu/Debian use o comando abaixo:

```bash
sudo apt -y install php php-apcu php-json php-intl php-mbstring php-xml php-mysql mariadb-server apache2 imagemagick
```





### Criando o Banco de Dados

A partir daqui, o processo é o mesmo tanto para instalações feitas a partir do repositório como instalações usando o código fonte. 

Agora vamos criar o banco de dados que nossa Wiki vai usar.

```bash
# Entre no Mysql (mariadb):
mysql 

# Crie o banco:
CREATE DATABASE taskforcewiki;

# Crie o usuário:
CREATE USER 'taskforcewikiuser'@'localhost' IDENTIFIED BY 'for1inWiK1D0';

# De permissão para o usuário no banco que criamos:
GRANT ALL PRIVILEGES ON taskforcewiki.* TO 'taskforcewikiuser'@'localhost' WITH GRANT OPTION;

# Recarregue as permissões:
FLUSH PRIVILEGES;
```



### Apache

Essa parte é opcional, e você deve escolher entre instalações usando `vhosts` (Virtual Host, onde cada Wiki terá uma URL com domínio diferente) ou usando subpages (onde cada Wiki será uma sub página de uma página principal, como `wiki.org/wiki1`, `wiki.org/wiki2`, `wiki.org/wiki3` e assim por diante) 



#### Configurando Vhosts

Vamos configurar o vhost para nossa Wiki.

```bash
# Crie o arquivo abaixo:
cat /etc/apache2/sites-enabled/taskforce.conf 

###### CONTEÚDO ######
<VirtualHost *:80>
        ServerAdmin taskforce@wiki.teste.com.br
        ServerName taskforcewiki.teste.com.br
        DocumentRoot /var/lib/wikis/taskforce171

	<Directory /var/lib/wikis/taskforce171/>
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
	<Directory /var/lib/wikis/taskforce171/config>
    	Options -FollowSymLinks
        AllowOverride None
    	<IfModule mod_php7.c>
        	php_admin_flag engine off
    	</IfModule>
    	<IfModule mod_php5.c>
        	php_admin_flag engine off
    	</IfModule>
	</Directory>
	<Directory /var/lib/wikis/taskforce171/images>
        Options -FollowSymLinks
        AllowOverride None
    	<IfModule mod_php7.c>
        	php_admin_flag engine off
    	</IfModule>
    	<IfModule mod_php5.c>
        	php_admin_flag engine off
    	</IfModule>
	</Directory>
	<Directory /var/lib/wikis/taskforce171/upload>
        Options -FollowSymLinks
        AllowOverride None
    	<IfModule mod_php7.c>
        	php_admin_flag engine off
    	</IfModule>
    	<IfModule mod_php5.c>
        	php_admin_flag engine off
    	</IfModule>
	</Directory>

	ErrorLog /var/log/apache2/error-taskforce171.log
    
    # Possible values include: debug, info, notice, warn, error, crit,
    # alert, emerg.
    LogLevel warn
    
    CustomLog /var/log/apache2/access-taskforce171.log combined
    AddDefaultCharset UTF-8

</VirtualHost>
###### FIM CONTEÚDO ######
```



#### Configurando Subpages

Configure uma página em branco da página principal do apache ou desative ela.

```bash
# Crie uma página principal em branco ou coloque algo nela (caso nao tenha), porque as wikis só serão acessada via subpage:
vim /etc/apache2/sites-enabled/000-default.conf

###### CONTEÚDO ######
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

###### FIM CONTEÚDO ######
```



Agora adicione a configuração da página no diretório abaixo:

```bash
# Acesse o arquivo abaixo:
vim /etc/mediawiki/apache.conf 

###### CONTEÚDO ######
Alias /taskforce171 /var/lib/wikis/taskforce171

<Directory /var/lib/wikis/taskforce171/>
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
<Directory /var/lib/wikis/taskforce171/config>
   	Options -FollowSymLinks
    AllowOverride None
  	<IfModule mod_php7.c>
       	php_admin_flag engine off
   	</IfModule>
   	<IfModule mod_php5.c>
       	php_admin_flag engine off
   	</IfModule>
</Directory>
<Directory /var/lib/wikis/taskforce171/images>
	Options -FollowSymLinks
    AllowOverride None
   	<IfModule mod_php7.c>
     	php_admin_flag engine off
   	</IfModule>
   	<IfModule mod_php5.c>
       	php_admin_flag engine off
   	</IfModule>
</Directory>
<Directory /var/lib/wikis/taskforce171/upload>
    Options -FollowSymLinks
    AllowOverride None
   	<IfModule mod_php7.c>
       	php_admin_flag engine off
   	</IfModule>
   	<IfModule mod_php5.c>
       	php_admin_flag engine off
   	</IfModule>
</Directory>
###### FIM CONTEÚDO ######
```



##### Configuração de redirecionamento

Não esqueça de modificar o `LocalSettings.php` antes de acessar via Web, caso não tenha alterado, [veja aqui](#Usando-o-método-Giant-Switch-Statement).





## Criando outras Wikis

Caso você precise criar outras Wikis, siga o passo a passo abaixo, isso pode tornar as páginas das Wikis existentes inácessíveis Durante o processo de criação:

```bash
# Crie a pasta onde vao ficar as Wikis:
sudo mkdir -p /var/lib/wikis/<NOME>

# Entre na pasta:
cd /var/lib/wikis/<NOME>

# Crie os links
sudo ln -s /usr/share/mediawiki/* .

# Apague alguns arquivos/pastas, não podemos ter eles como links:
rm LocalSettings.php AdminSettings.php images config

# Crie o LocalSettings da sua Wiki (apenas o link simbólico):
ln -s /etc/mediawiki/LocalSettings_<NOME>.php LocalSettings.php

# Crie as pastas:
sudo mkdir {images,upload,config}

# De as permissões corretas apenas para os arquivos abaixo:
sudo chown www-data. -R {images,config,upload}

###################################
###### Crie a conf no apache ######
###################################

# Renomeie o arquivo principal para que possa criar uma nova Wiki pela interface Gráfica:
mv /etc/mediawiki/LocalSettings.php /etc/mediawiki/LocalSettings.php-old

### Acessa a página pelo navegador ###
### Crie o LocalSettings.php com o arquivo fornecido ###

# Desfaça o rename que tinha feito para que as outras wikis volte a funcionar:
mv /etc/mediawiki/LocalSettings.php-old /etc/mediawiki/LocalSettings.php
```



Criando uma Wiki sem usar os recursos Web, isso diminui o tempo em que você vai deixar as wikis inacessíveis.

```bash
# Crie a pasta onde vao ficar as Wikis:
sudo mkdir -p /var/lib/wikis/producao

# Entre na pasta:
cd /var/lib/wikis/producao

# Crie os links
sudo ln -s /usr/share/mediawiki/* .

# Apague alguns arquivos/pastas, não podemos ter eles como links:
rm LocalSettings.php AdminSettings.php images config

# Crie o LocalSettings da sua Wiki (apenas o link simbólico):
ln -s /etc/mediawiki/LocalSettings_producao.php LocalSettings.php

# Crie as pastas:
sudo mkdir {images,upload,config}

# De as permissões corretas apenas para os arquivos abaixo:
sudo chown www-data. -R {images,config,upload}

# Entre no Mysql (mariadb):
mysql 

# Crie o banco:
CREATE DATABASE producaowiki;

# Crie o usuário:
CREATE USER 'producaowikiuser'@'localhost' IDENTIFIED BY 'for1inWiK1D0';

# De permissão para o usuário no banco que criamos:
GRANT ALL PRIVILEGES ON producaowiki.* TO 'producaowikiuser'@'localhost' WITH GRANT OPTION;

# Recarregue as permissões:
FLUSH PRIVILEGES;

###################################
###### Crie a conf no apache ######
###################################

# Configure a Wiki:
cd /usr/share/mediawiki/maintenance
mv /etc/mediawiki/LocalSettings.php /etc/mediawiki/LocalSettings.php-old && \
\
php install.php --confpath='/var/lib/wikis' --dbname=producaowiki --dbserver="localhost" --dbuser=producaowikiuser --dbpass='for1inWiK1D0' --server="http://wiki.test.br.com.br" --scriptpath=/producao --lang=en --pass='4d7tx:h.;Ojv#yngEm2&p%Urk' "producao" "admin" && \
\
mv /etc/mediawiki/LocalSettings.php-old /etc/mediawiki/LocalSettings.php

# Pegue o arquivo LocalSeetings criado:
mv /var/lib/wikis/LocalSettings.php /etc/mediawiki/LocalSettings_producao.php

# Agora restarte o Apache:
systemctl restart apache2.service
```





## Migrando uma Wiki de Servidor

Vamos demonstrar como migrar uma Wiki de servidor, o mesmo processo funciona para mais de uma wiki também, sendo que alguns passos só serão preciso ser realizados para a primeira Wiki (preparar o ambiente), tornando a migração das demais Wiki mais fácil.



### Antes de começar

Antes de começar a migração, você vai precisar do dump de seu banco de dados e uma cópia dos arquivos do MediaWiki, principalmente da pasta `upload` (caso tenha) e `image`.

```bash
# Fazendo dump do banco de dados (backup):
mysqldump --user=wikidb_user --password=wikidb_userpassword wikidb > file.sql

# Faça backup do LocalSettings.php e das pastas: image, upload e config.
```



### Instalando as dependências

```bash
# Instalando as dependências:
sudo apt -y install php php-apcu php-json php-intl php-mbstring php-xml php-mysql mariadb-server apache2 imagemagick
```



### Criando o Banco de Dados

Agora vamos criar o banco de dados que a nossa nova Wiki vai usar.

```bash
# Entre no Mysql (mariadb):
mysql 

# Crie o banco:
CREATE DATABASE maddogs;

# Crie o usuário:
CREATE USER 'maddogswikiuser'@'localhost' IDENTIFIED BY 'uZD?%WK+wBqr,G3vQbMnEP&9)';

# De permissão para o usuário no banco que criamos:
GRANT ALL PRIVILEGES ON maddogswiki.* TO 'maddogswikiuser'@'localhost' WITH GRANT OPTION;

# Recarregue as permissões:
FLUSH PRIVILEGES;
```



### Fazendo o Tunning do Mysql

Vamos fazer o tunning do Mysql para deixar ele melhor, segue o que usei:

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



### Importando o Dump do DB

Vamos importar o banco de dados que fizemos backup anteriormente.

```bash
sudo mysql -u root maddogswiki < bkp_db.sql
```



### Instalando o Media Wiki

Para isso, siga os processos descritos nos links abaixo, dependendo do tipo de instalação que você deseja.

[Instalação usando o código fonte](#Instalando-pelo-código-fonte)

[Instalando usando o repositório](#Instalando-usando-o-repositório)



### Crie seu LocalSettings.php

Você deve usar o mesmo arquivo que utiliza no servidor antigo e deve deixar no mesmo local em que está o servidor antigo.

Crie também o LocalSettings padrão, seja usando método `Giant Swtch Statement` ou `Drupal Style`.



### Apache

No meu caso, escolhi utilizar subpage que fica mais fácil, todas as Wikis são uma sub-página de um mesmo domínio.

Como de costume, na minha Wiki existe um arquivo chamado `/etc/mediawiki/apache.conf`, em versões mais novas (usando o pacote pelo repositório) esse arquivo vem com outro nome, mas vou manter este nome por questões práticas.

Vamos criar a entrada da nossa Wiki lá:

```bash
sudo vim /etc/mediawiki/apache.conf

### CONTEÚDO ###

###
#### WIKI producao
###

Alias /producao /var/lib/wikis/producao

<Directory /var/lib/wikis/producao/>
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
<Directory /var/lib/wikis/producao/config>
        Options -FollowSymLinks
        AllowOverride None
    <IfModule mod_php7.c>
        php_admin_flag engine off
    </IfModule>
    <IfModule mod_php5.c>
        php_admin_flag engine off
    </IfModule>
</Directory>
<Directory /var/lib/wikis/producao/images>
        Options -FollowSymLinks
        AllowOverride None
    <IfModule mod_php7.c>
        php_admin_flag engine off
    </IfModule>
    <IfModule mod_php5.c>
        php_admin_flag engine off
    </IfModule>
</Directory>
<Directory /var/lib/wikis/producao/upload>
        Options -FollowSymLinks
        AllowOverride None
    <IfModule mod_php7.c>
        php_admin_flag engine off
    </IfModule>
    <IfModule mod_php5.c>
        php_admin_flag engine off
    </IfModule>
</Directory>
### FIM CONTEÚDO ###
```



Agora na página default do apache `000-default.conf` inclua o caminho para o nosso arquivo de Wikis (`Include /etc/mediawiki/apache.conf`):

```bash
# Editando a página padrão do Apache:
vim /etc/apache2/sites-available/000-default.conf

### CONTEÚDO ###
<VirtualHost *:80>
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html
        ServerName wiki.test.br.com.br

        #LogLevel info ssl:warn

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined

        Include /etc/mediawiki/apache.conf
</VirtualHost>

# vim: syntax=apache ts=4 sw=4 sts=4 sr noet

### FIM CONTEÚDO ###

# Reinicie o Apache:
sudo apachectl -k graceful
```





### Finalizando a migração

Volte o backup das pastas: image, upload e config do backup de arquivos:

```bash
# Entre na pasta da Wiki:
cd /var/lib/wikis/producao

# Crie os links
sudo ln -s /usr/share/mediawiki/* .

# Apague alguns arquivos/pastas, não podemos ter eles como links:
rm LocalSettings.php AdminSettings.php images config

# Crie o LocalSettings da sua Wiki (apenas o link simbólico):
ln -s /etc/mediawiki/LocalSettings_taskforce171.php LocalSettings.php

# Copie os arquivos:
cp -r /home/vagrant/var/lib/wikis/producao/{images,upload,config} .

# De as permissões corretas apenas para os arquivos abaixo:
sudo chown www-data. -R {images,config,upload}
```



Atualize o banco da sua Wiki:

```bash
# Entra na pasta de manutenção:
cd /usr/share/mediawiki/maintenance

# Faça o Update:
php update.php --conf /etc/mediawiki/LocalSettings_producao.php
```



Acerte o diretório de Upload:

```bash
# Etique o LocalSettings da Wiki:
vim /etc/mediawiki/LocalSettings_producao.php

# Adicione a linha abaixo (não esqueça de acertar o caminho correto):
$wgUploadDirectory = "/var/lib/wikis/<NOME>/images";
```

Por fim reinicie o Apache. Com isso sua Wiki estará migrada.





## Criando um Usuários e Modificando Senha

Siga o passo a passo abaixo para criar/trocar a senha de usuários:

```bash
## Criando um usuário:

cd /usr/share/mediawiki/maintenance
cfg=/etc/mediawiki/LocalSettings_<NOME>.php
user_login='USERNAME'
pass_login='SENHA'

php createAndPromote.php --conf=${cfg} ${user_login} ${pass_login}



## trocando a senha

cd /usr/share/mediawiki/maintenance
cfg=/etc/mediawiki/LocalSettings_<NOME>.php
user_login='USERNAME'
pass_login='SENHA'

php changePassword.php --conf=${cfg} --user=${user_login} --password=${pass_login}
```





## LocalSettings.php padrão

Segue uma versão do localsettings padrão para a instalação do MediaWiki 1.31.7:

```bash
<?php
# This file was automatically generated by the MediaWiki 1.31.7
# installer. If you make manual changes, please keep track in case you
# need to recreate them later.
#
# See includes/DefaultSettings.php for all configurable settings
# and their default values, but don't forget to make changes in _this_
# file, not there.
#
# Further documentation for configuration settings may be found at:
# https://www.mediawiki.org/wiki/Manual:Configuration_settings

# Protect against web entry
if ( !defined( 'MEDIAWIKI' ) ) {
	exit;
}

## Include platform/distribution defaults
require_once "$IP/includes/PlatformSettings.php";

## Uncomment this to disable output compression
# $wgDisableOutputCompression = true;

$wgSitename = "taskforce171";
$wgMetaNamespace = "taskforce171";

## The URL base path to the directory containing the wiki;
## defaults for all runtime URL paths are based off of this.
## For more information on customizing the URLs
## (like /w/index.php/Page_title to /wiki/Page_title) please see:
## https://www.mediawiki.org/wiki/Manual:Short_URL
$wgScriptPath = "/taskforce171";

## The protocol and server name to use in fully-qualified URLs
$wgServer = "http://wiki.test.br.com.br";

## The URL path to static resources (images, scripts, etc.)
$wgResourceBasePath = $wgScriptPath;

## The URL path to the logo.  Make sure you change this from the default,
## or else you'll overwrite your logo when you upgrade!
$wgLogo = "$wgResourceBasePath/resources/assets/wiki.png";

## UPO means: this is also a user preference option

$wgEnableEmail = true;
$wgEnableUserEmail = true; # UPO

$wgEmergencyContact = "apache@wiki.test.br.com.br";
$wgPasswordSender = "apache@wiki.test.br.com.br";

$wgEnotifUserTalk = false; # UPO
$wgEnotifWatchlist = false; # UPO
$wgEmailAuthentication = true;

## Database settings
$wgDBtype = "mysql";
$wgDBserver = "localhost";
$wgDBname = "NAME-DB";
$wgDBuser = "USER-DB";
$wgDBpassword = "SENHA";

# MySQL specific settings
$wgDBprefix = "";

# MySQL table options to use during installation or update
$wgDBTableOptions = "ENGINE=InnoDB, DEFAULT CHARSET=binary";

## Shared memory settings
$wgMainCacheType = CACHE_NONE;
$wgMemCachedServers = [];

## To enable image uploads, make sure the 'images' directory
## is writable, then set this to true:
$wgEnableUploads = false;
$wgUseImageMagick = true;
$wgImageMagickConvertCommand = "/usr/bin/convert";

# InstantCommons allows wiki to use images from https://commons.wikimedia.org
$wgUseInstantCommons = false;

# Periodically send a pingback to https://www.mediawiki.org/ with basic data
# about this MediaWiki instance. The Wikimedia Foundation shares this data
# with MediaWiki developers to help guide future development efforts.
$wgPingback = true;

## If you use ImageMagick (or any other shell command) on a
## Linux server, this will need to be set to the name of an
## available UTF-8 locale
$wgShellLocale = "C.UTF-8";

## Set $wgCacheDirectory to a writable directory on the web server
## to make your wiki go slightly faster. The directory should not
## be publically accessible from the web.
#$wgCacheDirectory = "$IP/cache";

# Site language code, should be one of the list in ./languages/data/Names.php
$wgLanguageCode = "pt-br";

$wgSecretKey = "ca14996bd2d4d6ffc6282050f8893cf920e641ace81e0bad35fff255a095ed06";

# Changing this will log out all existing sessions.
$wgAuthenticationTokenVersion = "1";

# Site upgrade key. Must be set to a string (default provided) to turn on the
# web installer while LocalSettings.php is in place
$wgUpgradeKey = "a03458a91eeeedf6";

## For attaching licensing metadata to pages, and displaying an
## appropriate copyright notice / icon. GNU Free Documentation
## License and Creative Commons licenses are supported so far.
$wgRightsPage = ""; # Set to the title of a wiki page that describes your license/copyright
$wgRightsUrl = "";
$wgRightsText = "";
$wgRightsIcon = "";

# Path to the GNU diff3 utility. Used for conflict resolution.
$wgDiff3 = "/usr/bin/diff3";

## Default skin: you can change the default skin. Use the internal symbolic
## names, ie 'vector', 'monobook':
$wgDefaultSkin = "vector";

# Enabled skins.
# The following skins were automatically enabled:
wfLoadSkin( 'MonoBook' );
wfLoadSkin( 'Timeless' );
wfLoadSkin( 'Vector' );


# End of automatically generated settings.
# Add more configuration options below.
```







## Fontes

https://www.mediawiki.org/wiki/Manual:Wiki_family
https://www.mediawiki.org/wiki/Manual:Installation_guide
https://www.mediawiki.org/wiki/Manual:Installation_requirements
https://sharkysoft.com/wiki/how_to_configure_multiple_MediaWiki_instances_on_a_single_host
https://www.mediawiki.org/wiki/Manual:Upgrading/pt-br
https://www.mediawiki.org/wiki/Manual:Running_MediaWiki_on_Debian_or_Ubuntu
https://www.mediawiki.org/wiki/Manual:$wgLogo

