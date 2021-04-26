[toc]



# Server NIS - Network Information Service

O Network Information Service, ou NIS (originalmente chamado de *Yellow Pages*ou YP), é um protocolo de serviço de diretório cliente/servidor para distribuição de dados de configuração do sistema, foi desenvolvido pela Sun Microsystems em meados da década de 1980.

Como a *British Telecom PLC* possuí o nome "**Yellow Pages**" como marca registrada no Reino Unido para sua lista telefônica comercial baseada em papel, a Sun teve que mudar o nome de seu sistema para NIS, embora todos os comandos e funções ainda comecem com *yp*. 

O NIS foi desenvolvido com a finalidade de disseminar informações em uma rede, simplificando o gerenciamento de informações como: grupos de usuários, usuários, hosts e etc. 

O uso do NIS é muito importante em redes com a qual tem se um ambiente com várias máquinas. Isso é importante para assegurar aos usuários acessos a estas maquinas de uma forma uniforme, ou seja, garantir os mesmos níveis de acesso em todas as máquinas.

O*servidor*NIS é um host que fornece informações de configuração para outros hosts na rede. Os servidores retêm um conjunto de mapas e executam o daemon[**ypserv**](https://www.ibm.com/support/knowledgecenter/ssw_aix_71/com.ibm.aix.cmds6/ypserv.htm?view=kc), que processa solicitações de clientes para obter informações contidas nos mapas. Existem dois tipos de servidores: um servidor***mestre***e um servidor***escravo***.

- *Servidor master* - Um servidor *mestre* é o único host em um domínio específico que mantém os mapas autorizados. O servidor master executa o daemon [**ypupdated**](https://www.ibm.com/support/knowledgecenter/ssw_aix_71/com.ibm.aix.cmds6/ypupdated.htm?view=kc), que solicita que os servidores escravos atualizem suas cópias dos mapas (todos os outros hosts no domínio devem obter suas informações de mapa do servidor master, direta ou indiretamente). O servidor master também executa o daemon [**yppasswdd**](https://www.ibm.com/support/knowledgecenter/ssw_aix_71/com.ibm.aix.cmds6/yppasswdd.htm?view=kc), que processa solicitações para alterar as senhas dos usuários.

- *Servidor slave* - O servidor slave mantém uma base de dados transferidos de um servidor master, este serviço é geralmente usado com redes mistas ou segmentado com vários hosts, pois mantém um tráfego menor de rede.



Segue os comandos mais usados para configurar os clientes:

| Comandos     | Descrição                                            |
| ------------ | ---------------------------------------------------- |
| ypdomainname | Mostra o domínio do cliente NIS.                     |
| ypbind       | Mostra o servidor de domínio NIS.                    |
| ypmake       | Cria um mapeamento entre o cliente e o servidor NIS. |
| ypset        | Relaciona um cliente NIS ao um servidor.             |
| ypcat        | Mostra a base de dados do servidor NIS.              |
| ypwhich      | Mostra o nome da máquina do servidor NIS.            |
| yppoll       | Mostra a versão do servidor NIS.                     |
| ypmatch      | Faz uma procurar na base de dados do NIS.            |
| yppasswd     | Muda a senha do usuário na base de dados NIS.        |
| ypchsh       | Muda o shell da base de dados NIS.                   |
| ypchfn       | Muda o nome do usuário da base de dados NIS.         |



## Clientes

Os *clientes* NIS constituem a maioria dos hosts em um domínio NIS. Os clientes executam o daemon [**ypbind**](https://www.ibm.com/support/knowledgecenter/ssw_aix_71/com.ibm.aix.cmds6/ypbind.htm?view=kc), que permite que os processos do cliente obtenham informações de um servidor. Os clientes não mantêm mapas, mas consultam os servidores em busca de informações do sistema e da conta do usuário (Os clientes não fazem distinção entre consultar o servidor mestre ou um servidor escravo).

Para acessar as informações do sistema contidas em um mapa, um cliente faz uma Chamada de Procedimento Remoto ([RPC](https://www.ibm.com/support/knowledgecenter/ssw_aix_71/com.ibm.aix.progcomc/ch8_rpc.htm?view=kc)) para um servidor. O servidor pesquisa seu banco de dados local e retorna as informações solicitadas ao cliente.

Observe que cada solicitação de informações do sistema requer um contato com o servidor, e a velocidade da rede pode afetar o tempo de resposta. Embora uma recuperação local seja geralmente mais rápida do que uma recuperação de rede, os benefícios do NIS superam o comprometimento do tempo de acesso.



## Domínio

Um domínio NIS é uma coleção de sistemas agrupados logicamente. Um grupo de hosts que compartilha o mesmo conjunto de mapas NIS pertence ao mesmo domínio. Os hosts geralmente são agrupados no domínio por um motivo comum; por exemplo, ao trabalhar no mesmo grupo em um local específico. Cada host NIS é atribuído a um domínio quando o sistema é iniciado. O nome de domínio deve ser definido em todos os hosts que pretendem usar o NIS.

Há um servidor mestre por domínio NIS e os sistemas do domínio geralmente estão na mesma rede. No entanto, o acesso aos dados servidos pelo NIS é independente das localizações relativas de um cliente e servidor NIS. Todos os sistemas dentro do domínio NIS usam o servidor mestre para recuperar informações do sistema e o número de sistemas em um domínio deve ser limitado para fins de eficiência. Conforme o número de sistemas aumenta, o tempo de resposta do servidor mestre aumenta devido ao aumento da carga de trabalho. Por design, você não pode adicionar outro servidor mestre a um domínio porque haveria duas fontes autoritativas para os mapas. Para reduzir a carga do servidor mestre, você pode adicionar servidores escravos ao domínio ou definir mais de um domínio. Cada novo domínio, é claro, tem seu próprio servidor mestre.



## Mapas

Mapas NIS são bancos de dados que especificam certas informações do sistema, como nomes de usuário, senhas e nomes de host, em um formato de banco de dados denominado DBM (DataBase Management - Gerenciamento de Banco de Dados). 
Cada mapa é construído a partir de um arquivo de texto padrão, associando uma chave de índice a um valor. Por exemplo, as informações no arquivo `/etc/hosts` do servidor mestre são usadas para criar um mapa que usa cada nome de host como uma chave e o endereço IP como o valor. Os pares de chave e valor (também conhecidos como registros ) que são criados a partir das entradas no arquivo `/etc/hosts` abrangem o mapa `hosts.byname`.

Um registro NIS tem um tamanho máximo de 1024 bytes. Essa limitação se aplica a todos os arquivos de mapa NIS. O NIS não pode operar corretamente com arquivos de mapa que excedem este tamanho máximo. Os mapas mais comumente usados têm apelidos que alguns comandos podem traduzir em nomes de mapas. 
Por exemplo, quando você insere:

```
$ ypcat hosts
```

A saída que você recebe é na verdade o conteúdo do mapa `hosts.byname`, porque não há nenhum mapa chamado hospedeiros no banco de dados NIS. (O comando ypcat -x produz uma lista de apelidos disponíveis.)

Por padrão, os mapas listados na tabela a seguir são criados se seus arquivos correspondentes estiverem disponíveis no servidor mestre:

| Mapa                   | Nickname  | Arquivo             |
| ---------------------- | --------- | ------------------- |
| **passwd.byname**      | passwd    | **/etc/passwd**     |
| **passwd.byuid**       | passwd    | **/etc/passwd**     |
| **group.byname**       | group     | **/etc/group**      |
| **group.bygid**        | group     | **/etc/group**      |
| **hosts.byaddr**       | hosts     | **/etc/hosts**      |
| **hosts.byname**       | hosts     | **/etc/hosts**      |
| **ethers.byaddr**      | ethers    | **/etc/ethers**     |
| **ethers.byname**      | ethers    | **/etc/ethers**     |
| **networks.byaddr**    | networks  | **/etc/networks**   |
| **networks.byname**    | networks  | **/etc/networks**   |
| **rpc.bynumber**       |           | **/etc/rpc**        |
| **services.byname**    | services  | **/etc/services**   |
| **protocols.byname**   | protocols | **/etc/protocols**  |
| **protocols.bynumber** | protocols | **/etc/protocols**  |
| **netgroup**           |           | **/etc/netgroup**   |
| **netgroup.byhost**    |           | **/etc/netgroup**   |
| **netgroup.byuser**    |           | **/etc/netgroup**   |
| **bootparams**         |           | **/etc/bootparams** |
| **mail.aliases**       | aliases   | **/etc/aliases**    |
| **mail.byaddr**        | aliases   | **/etc/aliases**    |
| **publickey.byname**   |           | **/etc/publickey**  |
| **netid.byname**       |           | **/etc/passwd**     |
| **netid.byname**       |           | **/etc/group**      |
| **netid.byname**       |           | **/etc/hosts**      |
| **netid.byname**       |           | **/etc/netid**      |
| **netmasks.byaddr**    |           | **/etc/netmasks**   |
| **ypservers**          |           |                     |



### YPSERVERS uma mapa especial

Observe que nenhum arquivo corresponde ao mapa **ypservers**. **YPSERVERS** é um mapa especial que contém os nomes dos servidores NIS, slaves e masters, no domínio. Os clientes usam o mapa **ypservers** para encontrar o servidor disponível mais próximo. O servidor mestre se refere a ele para determinar os nomes dos servidores escravos que precisam obter cópias atualizadas dos mapas NIS. As informações sobre como especificar a entrada para o mapa **ypservers** são discutidas em [Configurando o Servidor NIS Mestre](https://www.ibm.com/support/knowledgecenter/ssw_aix_71/com.ibm.aix.nisplus/doc/nisplus/nis_install.htm?view=kc#a38c012245) e [Adicionando um Novo Servidor NIS Escravo](https://www.ibm.com/support/knowledgecenter/ssw_aix_71/com.ibm.aix.nisplus/doc/nisplus/nis_maint.htm?view=kc#a26c0100bd) .



### Netgroups: grupos de máquinas e usuários em toda a rede

Além dos bancos de dados padrão de senha, grupo e arquivo de host, o NIS apresenta um novo banco de dados para a criação de conjuntos de usuários e hosts chamados de mapa de grupos. Os campos de usuário e nome de host são usados para definir grupos (ofhosts ou usuários) para fins administrativos. Por exemplo, para definir um subconjunto de usuários no mapa passwd que deve ter acesso a uma máquina específica, você pode criar um grupo de rede para esses usuários.

Netgroups são normalmente usados para aumentar outros mapas e arquivos (por exemplo, adicionar um grupo selecionado de usuários ao arquivo de senha).

```bash
## Alguns exemplos uteis:
# (hostname, username, domain name)
source (-,stern,nesales), (-,julie,nesales), (-,peter,nesales) 
trusted-hosts (bitatron,,), (corvette,,) 
trusted-users (bitatron,stern,), (corvette,johnc,) 
dangerous-users (,jimc,), (,dave,)

# Se a entrada for especificada como um traço (-), o campo não pode assumir nenhum valor.

# O local do arquivo fica em /etc/netgroup
```



### Criando Mapas: makedbm e Makefile

Os mapas NIS são criados pelo comando [**makedbm**](https://www.ibm.com/support/knowledgecenter/ssw_aix_71/com.ibm.aix.cmds3/makedbm.htm?view=kc), convertendo arquivos de texto em arquivos no formato DBM. Para simplificar a manutenção de seus mapas, o NIS fornece um *makefile*para uso com o comando **make**. O makefile padrão (**/var/yp/Makefile**) contém todas as instruções necessárias para criar todos os mapas padrão. Você pode adicionar estrofes a **/var/yp/Makefile** para criar mapas adicionais. No entanto, o makefile padrão é suficiente para atender às necessidades básicas da maioria das instalações NIS.

Quando o comando **makedbm** gera um mapa NIS, ele cria dois arquivos:*map.key.pag* e *map.key.dir*. Por exemplo, o mapa *host.byname* consiste nos arquivos **hosts.byname.pag** e **hosts.byname.dir**. O arquivo com a extensão **.pag** contém os pares de chave e valor, enquanto o arquivo com a extensão **.dir** é o índice do arquivo **.pag** . Todos os mapas de um domínio são armazenados nos servidores em um subdiretório do diretório **/var/yp** . O subdiretório tem o mesmo nome do domínio. Por exemplo, mapas para o domínio literatura estão localizados no subdiretório **/var/yp/literatura**.



## Layout da rede

Com o tutorial completo, vamos ter 2 servidores para o NIS; um Master e outro Slave, vamos ter apenas um cliente. 

Vamos começar apenas com o servidor Master e o cliente, depois de tudo funcionando, vamos inserir o servidor Slave na rede.

| Nome dos Hosts | Domínio    | IP                 |
| -------------- | ---------- | ------------------ |
| nismaster      | domaintest | 192.168.122.140/24 |
| nislave        | domaintest | 192.168.122.141/24 |
| nisclient      | domaintest | 192.168.122.10/24  |



## Instalando o servidor NIS

Para ambientes Red Hat, execute o comando abaixo:

```bash
$ sudo dnf -y install ypserv
```



Para ambientes Debian, execute o comando abaixo:

```bash
$ sudo apt-get -y install nis
```



Alguns tutoriais informam que depois da instalação devemos informar o domínio do nosso server NIS, isso não será preciso porque informamos o domínio durante a instalação, para consultar o domínio rode o comando `ypdomainname`, assim poderá ver se o domínio foi configurado.

Para mudar o domínio use o comando abaixo (funciona **momentaneamente**):

```bash
# Esse comando vai reconfigurar o pacote nis:
$ sudo dpkg-reconfigure nis
```



Para ativar um domínio definitivo edite o arquivo abaixo:

```
/etc/defaultdomain
```



### Configurando o servidor NIS

Vamos configurar nosso servidor como servidor **master** para o serviço do NIS, para isso, siga o passo a passo abaixo:

```bash
# Editar o arquivo de configuração:
$ sudo vim /etc/default/nis

# Edite a linha 'NISSERVER' e 'NISCLIENT', deixando igual abaixo:
NISSERVER=master
NISCLIENT=false
```



Agora vamos configurar quem deve ter acesso ao serviço do NIS:

```bash
# Primeiro acesse o arquivo de configuração usando o comando abaixo:
$ sudo vim /etc/ypserv.securenets

# Após isso, comente a última linha, ela deverá ficar assim:
#0.0.0.0                0.0.0.0
# Essa linha acima informa que qualquer host de qualquer rede poderá ter acesso a nosso NIS.

# Agora vamos configurar uma rede de exemplo para nosso NIS:
255.0.0.0       127.0.0.0
255.255.255.0   192.168.122.0

# Depois é só salvar e sair!
```

No arquivo de "permissões" eu tenho apenas duas linhas, uma permitindo o próprio servidor (usando a interface de loopback) e outra permitindo uma outra rede, veja apenas as linhas sem comentário nesse arquivo:

```bash
$ grep -v "^#" /etc/ypserv.securenets 
255.0.0.0	127.0.0.0
255.255.255.0	192.168.122.0

# Esse comando remove os comentario do arquivo apenas para exibição!
```



### Problema de segurança

Antes de o serviço do NIS funcionar, o `ypbind` procura um servidor NIS na rede, usando mensagens de broadcast, isso é um problema, um usuário mal intencionado poderia subir um servidor NIS na rede, e os clientes iriam usar esse servidor.

Para resolver esse problema, vamos adicionar o servidor que deve ser usado, dessa forma, não serão enviados mensagens de Broadcast, e vamos usar um servidor que sabemos que é confiável.

Vamos acertar a configuração do `/etc/hosts`:

```bash
# Vamos editar o arquivo /etc/hosts
$ sudo vim /etc/hosts

# Segue meu exemplo:
127.0.0.1 localhost
127.0.1.1 nismaster
192.168.122.140 nismaster
```

Isso faz com que o `ypbind` procure e encontre nosso server pelo IP, isso faz com que você pare de ver a mensagem de broadcast no status do serviço `ypbind` e `ypserv`.

A mensagem é `broadcast: RPC: Timed out.`



Agora vamos dizer ao servidor NIS qual servidor usar, no caso de servidores, eles devem usar a sí mesmos:

```bash
# Edite o arquivo:
$ sudo vim /etc/yp.conf

# Configure o servidor para o domínio especificado:
ypserver 127.0.0.1
```

Para um maior troubleshooting você pode usar o comando: `sudo ypbind -d -broadcast`



### Crias a base de dados do NIS

Agora vamos rodar um script do NIS para construir nossa base de dados:

```bash
# Esse é nosso primeiro e úncio servidor NIS, então ele é o servidor master da rede!

# A opção '-m' que vamos usar serve para criar uma base de dados no servidor Master, rode o comando abaixo:
$ sudo /usr/lib/yp/ypinit -m

# Após isso ele vai pedir para adicionar o próximo host
# Agora você pode adicionar mais servidores ao domínio ou simplesmente
# finalizar com um 'ctrl + d'.
# Observe que isso pode mudar, entao leia a mensagem na tela!!

# Após aceitar, o script vai criar a base de dados.
```



Segue abaixo a saída que me apareceu após finalizar:

```
At this point, we have to construct a list of the hosts which will run NIS
servers.  nismaster is in the list of NIS server hosts.  Please continue to add
the names for the other hosts, one per line.  When you are done with the
list, type a <control D>.
	next host to add:  nismaster
	next host to add:  
The current list of NIS servers looks like this:

nismaster

Is this correct?  [y/n: y]  y
We need a few minutes to build the databases...
Building /var/yp/domaintest/ypservers...
Running /var/yp/Makefile...
make[1]: Entering directory '/var/yp/domaintest'
Updating passwd.byname...
Updating passwd.byuid...
Updating group.byname...
Updating group.bygid...
Updating hosts.byname...
Updating hosts.byaddr...
Updating rpc.byname...
Updating rpc.bynumber...
Updating services.byname...
Updating services.byservicename...
Updating netid.byname...
Updating protocols.bynumber...
Updating protocols.byname...
Updating netgroup...
Updating netgroup.byhost...
Updating netgroup.byuser...
Updating shadow.byname...
make[1]: Leaving directory '/var/yp/domaintest'

nismaster has been set up as a NIS master server.

Now you can run ypinit -s nismaster on all slave server.
```



## Criando usuário para o Domínio

Nesse ponto, qualquer usuário que tenha o UID maior ou igual a 1000 já está na base de dados do nosso NIS, para ver a base de dados (usuários) atual, digite o comando abaixo: 

```bash
# Exibe o passwd do nosso NIS:
$ ypcat passwd
```



Agora vamos criar um usuário e atualizar o banco de dados do NIS.

```bash
# Criar um usuário chamado Marcelo com diretorio home 
# e o shell=/bin/bash
$ sudo useradd -m -s /bin/bash marcelo

# Trocar a senha desse usuário:
$ sudo passwd marcelo
```



Agora rode o comando `cat /etc/passwd` e veja que o usuário está la, depois rode o comando `ypcat passwd` e veja que o usuário não está lá, pois bem, vamos atualizar o banco de dados do NIS.

```bash
# Atualizar os mapas do NIS:
$ cd /var/yp/
$ sudo make
```

A saída do comando `sudo make` pode ser vista abaixo:

```
make[1]: Entering directory '/var/yp/domaintest'
Updating passwd.byname...
Updating passwd.byuid...
Updating group.byname...
Updating group.bygid...
Updating netid.byname...
Updating shadow.byname...
make[1]: Leaving directory '/var/yp/domaintest'
```

Agora rode novamente o comando `ypcat passwd` e perceba que o usuário está lá, com isso podemos validar o acesso de um usuário, depois o usuário deve trocar a senha para uma de sua escolha, usando o comando `sudo yppasswd -p USUARIO`.

> Se você obter o erro abaixo ao tentar trocar a senha de um usuário no servidor master, isso significa que a senha de root usada está incorreta!
>
> O erro exibido pela saída do comando yppasswd será a seguinte:
>
> ​		Error while changing the NIS password.
> ​		The NIS password has not been changed on pxe-teste.

## Configurando o cliente

Para ambientes Red Hat, execute o comando abaixo:

```bash
$ sudo dnf -y install ypserv
```



Para ambientes Debian, execute o comando abaixo:

```bash
$ sudo apt-get -y install nis
```



Agora vamos colocar o cliente no mesmo domínio do server; informa qual servidor NIS usar, explicado mais acima:

```bash
# Edite o arquivo:
$ sudo vim /etc/yp.conf

# Configure o servidor para o domínio especificado:
domain domaintest server 192.168.122.140

# Agora reinicie o serviço:
$ sudo systemctl restart nis
```

Agora vamos editar o arquivo `nsswitch.conf` para incluir o server NIS na busca de autenticação do linux:

```bash
# Edite o arquivo:
$ sudo vim /etc/nsswitch.conf

# Adicione a palavra 'nis' nas linhas abaixo:
# /etc/nsswitch.conf
#
# Example configuration of GNU Name Service Switch functionality.
# If you have the `glibc-doc-reference' and `info' packages installed, try:
# `info libc "Name Service Switch"' for information about this file.

passwd:         files systemd nis
group:          files systemd nis
shadow:         files nis
hosts:          files mdns4_minimal [NOTFOUND=return] dns nis

# O seu arquivo pode ser diferente do meu, mas apenas acrescente o 'nis' nas linhas que estão sendo exibidas acima.

# O arquivo de configuração nsswitch.conf do Linux controla como a resolução de nomes funciona ao pesquisar vários tipos de objetos, como endereços de host e senhas.
```



Segue abaixo, apenas para curiosidade o arquivo `nsswitch.conf` completo, após ser editado:

```
$ cat /etc/nsswitch.conf 
# /etc/nsswitch.conf
#
# Example configuration of GNU Name Service Switch functionality.
# If you have the `glibc-doc-reference' and `info' packages installed, try:
# `info libc "Name Service Switch"' for information about this file.

passwd:         files systemd nis
group:          files systemd nis
shadow:         files nis
gshadow:        files

hosts:          files mdns4_minimal [NOTFOUND=return] dns nis
networks:       files

protocols:      db files
services:       db files
ethers:         db files
rpc:            db files

netgroup:       nis
```



Agora por padrão configure o endereço do server NIS com o nome do host:

```bash
# Editar o /etc/hosts:
$ sudo vim /etc/hosts

# Coloque aqui o Endereço IP relacionando com o nome do servidor:
192.168.122.140 nismaster

# 192.168.122.140 = Endereço IP do servidor NIS
# nismaster = Nome do servidor NIS (é o hostname)
```



Agora vamos reiniciar os serviços:

```bash
# Agora reinicie o serviço:
$ sudo systemctl restart rpcbind nis
```



### Para ambientes sem NFS

Agora vamos acrescentar uma linha PAM de sessão para criar um diretório para o usuário assim que ele logar:

```bash
# Editando o arquivo:
$ vim /etc/pam.d/common-session

# Acrescente a linha abaixo ao final do arquivo:
session optional       pam_mkhomedir.so skel=/etc/skel umask=077
```



## Configurando o servidor Slave

Vamos configurar o servidor NIS Slave.

Toda a configuração abaixo deve ser feita no server Slave!

```bash
## Configure o nome mais IP do server em /etc/hosts:
# Edite o /etc/hosts:
$ sudo vim /etc/hosts

# Adicione as linhas abaixo:
192.168.122.140 nismaster

## Agora vamos configurar o domínio
# Edite o /etc/yp.conf
$ sudo vim /etc/yp.conf

# Adicione a linha abaixo:
ypserver 127.0.0.1

## Vamos configurar servidor como Slave:
# Edite o /etc/default/nis:
$ sudo vim /etc/default/nis

# Altere algumas linhas e deixe como abaixo:
NISSERVER=slave
NISCLIENT=false

# Configure as permissões:
$ sudo vim /etc/ypserv.securenets
255.0.0.0	127.0.0.0
255.255.255.0   192.168.122.0

# Agora vamos setar nosso server
$ sudo /usr/lib/yp/ypinit -s nismaster
```

A saída do comando acima pode ser vista abaixo:

```
/usr/lib/yp/ypinit -s nismaster
We will need a few minutes to copy the data from pxe-master.
Transferring group.bygid...
Transferring netgroup...
Transferring netid.byname...
Transferring ypservers...
Transferring passwd.byname...
Transferring protocols.byname...
Transferring hosts.byname...
Transferring group.byname...
Transferring rpc.byname...
Transferring services.byservicename...
Transferring shadow.byname...
Transferring netgroup.byhost...
Transferring passwd.byuid...
Transferring rpc.bynumber...
Transferring netgroup.byuser...
Transferring services.byname...
Transferring protocols.bynumber...
Transferring hosts.byaddr...

pxe-slave's NIS data base has been set up.
If there were warnings, please figure out what went wrong, and fix it.

At this point, make sure that /etc/passwd and /etc/group have
been edited so that when the NIS is activated, the data bases you
have just created will be used, instead of the /etc ASCII files.
```



### Configuração do server Master

**Agora o passo abaixo deve ser feito no servidor master**

```bash
## Vamos adicionar o server slave em nosso /etc/hosts:
# Edite o arquivo:
$ sudo vim /etc/hosts

# Adicione a linha abaixo:
192.168.122.141 nislave

# Deve ter essas duas entradas abaixo:
192.168.122.140 nismaster
192.168.122.141 nislave
```

Agora vamos editar o arquivo `/var/yp/Makefile`, nossa mudança irá replicar automaticamente os mapas do banco de dados para todos os servidores em `/var/yp/ypservers` enquanto executa o comando make no servidor mestre NIS.

Para isso precisamos mudar a entrada NOPUSH!

```bash
# Edite o arquivo /var/yp/Makefile
$ sudo vim /var/yp/Makefile

## Agamos mude a opção 'NOPUSH=true' para 'false'
# O comando abaixo mostra como está nossa opção:
$ sed -n '/^NOPUSH/p' /var/yp/Makefile
NOPUSH=false
```



Agora vamos adicionar nosso servidor slave no arquivo `/var/yp/ypservers`:

```bash
# Adicione o nome do server slave ao arquivo abaixo:
$ sudo vim /var/yp/ypservers 
nismaster

# Deve ficar assim:
$ sudo cat /var/yp/ypservers 
nismaster
nislave

# Aplique a configuração:
$ cd /var/yp/
$ sudo make
make[1]: Entering directory '/var/yp/domaintest'
make[1]: 'ypservers' is up to date.
make[1]: Leaving directory '/var/yp/domaintest'
make[1]: Entering directory '/var/yp/domaintest'
Updating netid.byname...
make[1]: Leaving directory '/var/yp/domaintest'
```



### Configuração do Cliente

**Os próximos passos devem ser realizados no Cliente:**

```bash
# Vamos adicionar nosso slave na entrada do cliente.
# Para isso, edite o arquivo /etc/yp.conf e insira a linha abaixo:
domain domaintest server 192.168.122.141

# Você deve estar com duas saídas agora, veja as linhas ativas abaixo:
domain domaintest server 192.168.122.140
domain domaintest server 192.168.122.141
```



## Montando o /home automaticamente

Vamos começar o processo pelo servidor onde estão os diretórios dos usuários, nesse caso no servidor master do NIS.

O correto é ter uma máquina separada do NIS para montar os diretórios *home* dos usuários, assim você pode trabalhar com backups nesse máquina, e não precisa se preocupar em ter o compartilhamento no master e no slave. Mas para nosso teste, irei deixar apenas no servidor master.

```bash
# Instale o nfs no servidor:
$ sudo apt install nfs-kernel-server -y

## Agora vamos configurar o diretório que será compartilhado:
# Edite o arquivo /etc/exports
$ sudo vim /etc/exports

# Agora adicione a linha abaixo ao seu arquivo:
/home/    192.168.122.0/24(rw,sync,no_subtree_check)

# Ative o compartilhamento:
$ sudo exportfs -a

# Nesse caso estou limitando o compartilhamento apenas para essa rede!
```



Agora vamos fazer as mudanças no cliente.

```bash
# Instale o nfs e o autofs:
$ sudo apt install autofs nfs-common unscd -y

# Agora vamos criar os arquivos /etc/auto.{home,master}
$ sudo touch /etc/auto.{home,master}

# Edite o arquivo /etc/auto.master e adicione a linha abaixo:
/home/ /etc/auto.home

# Agora edite o arquivo /etc/auto.home e adicione a linha abaixo:
*    -rw,sync    nismaster:/home/

# Reinicie o serviço do autofs
$ sudo systemctl restart autofs

# Faça um teste usando o comando abaixo:
$ su - USER
# Troque USER pelo nome do usuário!
```



Ao invés de usar o autofs, podemos montar o /home no inicio do boot, coloque a entrada abaixo no `/etc/fstab`:

```bash
# Colocar a linha abaixo em /etc/fstab:
192.168.122.140:/home/      /home/   nfs4    _netdev,async,noatime,nolock,fsc  0       2

# Rode o fstab
$ sudo mount -a

# Se der algum erro, rode de novo!

# Para verificar se deu certo, vamos listar os pontos de montagem no sistema:
$ df -h -x tmpfs
Filesystem                         Size  Used Avail Use% Mounted on
udev                               1.9G     0  1.9G   0% /dev
192.168.122.140:/home               14G  5.5G  7.6G  42% /home


# Perceba o IP e o ponto de montagem em /home
```



## Trocando o banco de dados de usuário

Por algumas questões de segurança, não é bom deixarmos os usuários no servidor, visto que mesmo sem acesso root ou com sudo, eles ainda podem logar no servidor, é bom que limitarmos todos os pontos de acesso possível para diminuir ainda mais as brechas, por isso, vamos trocar o banco de dados do server NIS para outro arquivo, assim os usuários vão existir no NIS, e no sistema do servidor, vamos impedir os usuário de logarem no mesmo.

Para exemplificar:

- Mudar o mapname passwd, shadow, group e gshadow para outro arquivo de origem;
- Impedir o usuário de fazer login no servidor (irá logar apenas pelo NIS).



Para isso, vamos seguir os passos abaixos:

```bash
# Fazer backup do /var/yp/Makefile por segurança
$ cp /var/yp/Makefile /var/yp/Makefile.bkp

# Vamos criar um diretório para colocar os novos arquivos de origem:
$ mkdir /etc/nismaps

# Adicione a permissao 0700 para essa pasta:
$ chmod 0700 /etc/nismaps

# Agora copie os arquivos passwd, shadow, group e gshadow para /etc/nismps
$ cp /etc/passwd /etc/nismaps/passwd
$ cp /etc/shadow /etc/nismaps/shadow
$ cp /etc/gshadow /etc/nismaps/gshadow
$ cp /etc/group /etc/nismaps/group

# Por fim, mude a permissão deles por segurança, é a mesma permissão que os arquivos originais:
$ chmod 0644 /etc/nismaps/group
$ chmod 0644 /etc/nismaps/passwd
$ chmod 0640 /etc/nismaps/shadow
$ chmod 0640 /etc/nismaps/gshadow

## Entre em cada arquivo acima e deixe apenas as linhas referentes ao usuário/grupo que você deseja no NIS.
```

Agora vamos editar o nosso Makefile:

```bash
# Edite o arquivo /var/yp/Makefile:
vim /var/yp/Makefile

# Agora localize as linhas:
GROUP       = $(YPPWDDIR)/group
PASSWD      = $(YPPWDDIR)/passwd
SHADOW	    = $(YPPWDDIR)/shadow
GSHADOW     = $(YPPWDDIR)/gshadow

## Deixe as linhas acima, igual as linhas abaixo:
GROUP       = $(YPPWDDIR)/nismaps/group
PASSWD      = $(YPPWDDIR)/nismaps/passwd
SHADOW      = $(YPPWDDIR)/nismaps/shadow
GSHADOW     = $(YPPWDDIR)/nismaps/gshadow

# $(YPPWDDIR) é o diretório /etc (está no arquivo de configuração, mais acima)
# /nismaps/ é o diretorio que criamos para colocar os mapnames.
```



### Procedimento para usuários novos e antigos

Se o seu servidor NIS ja estiver implantado ou se estiver fazendo do zero, os procedimentos a seguir sempre devem ser realizados.

 - Sempre que criar um usuário ou grupo para o NIS

   ```bash
   # Sempre que for criado um usuário ou grupo deve-se pegar a linha referente ao que foi criado e ser inserido nos arquivos dentro de /etc/nismaps/
   
   ## Agora vamos atualizar o banco de dados do NIS, para isso rode os comandos abaixo.
   # Entrar no mesmo diretório do Makefile
   $ cd /var/yp/
   
   # Agora rode o comando make para atualizar o banco de dados
   $ sudo make
   
   ## Agora devemos desativar as contas do nosso servidor, assim esses usuários não poderão acessar o servidor, para isso rode os comando abaixo:
   $ sudo usermod -L USER
   $ sudo usermod -s nologin USER
   ```

   Isso deve ser feito sempre que um usuário ou grupo dor adicionado ao NIS.



## Algumas curiosidades

Buscar um usuário na base de dados:

`sudo ypmatch USUARIO passwd`



Trocar a senha de um usuário que ja esteja na base de dados:

`sudo yppasswd -p USUARIO`



Listar base de usuários:

`sudo ypcat passwd`



Os mapas se propagam automaticamente a cada poucos minutos se o servidor mestre estiver executando o daemon `ypupdated`. Se você parar e reiniciar o NIS no servidor mestre, todos os mapas se propagam para os servidores escravos. 

Se você inserir o comando `yppush` no servidor principal, as alterações se propagam. O comando `yppush` notifica todos os servidores escravos que um mapa deve ser transferido. 

O daemon `ypserv` em cada servidor escravo executa o comando `ypxfr` para obter o mapa atualizado. Um servidor escravo que está fora de serviço quando você insere o comando yppush retém a versão anterior do mapa quando ele retorna à rede. Para evitar tais situações, use o  daemon para definir cada servidor escravo para solicitar mapas atualizados do servidor mestre em intervalos regulares.



Para ler o*man*do Network Information Service, use os comandos abaixo:

```bash
# Para verificar quais mans estão disponíveis:
root@debian:~# whatis nisdomainname
nisdomainname (1)    - show or set the system's NIS/YP domain name
nisdomainname (8)    - set or display name of current NIS(YP) domain

# Agora é só rodar o comando 'man' para ler esse manuais:
root@debian:~# man nisdomainname
root@debian:~# man 8 nisdomainname
```

 

## Fontes

- [passwd(4)-Docs.Oracle](https://docs.oracle.com/cd/E19253-01/816-5174/6mbb98uhh/index.html)

- [What does a plus + at the beginning of a line in the /etc/passwd UNIX / Linux password file signify?-nixCraft](https://www.cyberciti.biz/faq/plus-minus-sign-in-unix-linux-passwd-file/)

- [What does a plus + at the beginning of a line in the /etc/passwd UNIX / Linux password file signify?-InfoSec4Humans](https://infosec4humans.wordpress.com/2010/01/25/in-etcpasswd-file/)

- [What does a plus + at the beginning of a line in the /etc/passwd UNIX / Linux password file signify?-SXI.IO](https://sxi.io/what-does-a-plus-at-the-beginning-of-a-line-in-the-etc-passwd-unix-linux-password-file-signify/)

- [Book - Managing NFS and NIS](https://www.amazon.com.br/Managing-NFS-NIS-Hal-Stern/dp/1565925106)

- [Network Information Service - Wikipedia](https://en.wikipedia.org/wiki/Network_Information_Service)

- [Configure NIS - Server-world](https://www.server-world.info/en/note?os=CentOS_7&p=nis)

- [Sistema de identificação em rede (NIS) - VivaoLinux](https://www.vivaolinux.com.br/artigo/Sistema-de-identificacao-em-rede-(NIS))

  