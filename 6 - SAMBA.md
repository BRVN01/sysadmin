[toc]



# Introdução ao Samba

Samba é um software desenvolvido por Andrew Tridgell, seu desenvolvimento teve inicio em 1991. Mais tarde esse protocolo tornou-se Server Message Block (SMB), o predecessor do CIFS. Um tempo depois, Andrew expandiu o SMB e começou a distribuí-lo como um produto gratuito na Internet sob o nome de "Servidor SMB". No entanto, ele não conseguiu manter esse nome porque já pertencia a outro produto de uma outra empresa (Microsoft), então foi dado o nome de Samba.

Samba é um pacote de software que permite que um sistema baseado em Unix apareça e funcione como um servidor Microsoft Windows quando visualizado por outros sistemas em uma rede, como AD (Active Directory), File Server (Servidor de arquivo que pode ser usado para máquinas Windows e Linux/Unix) dentre outras funcionalidades.

Com o passar do tempo, o Samba vem criando mais e mais funcionalidades para substituir o AD do Windows Server como uma maneira gratuita de se obter tais funcionalidades, não só isso, traz a integração no compartilhamento de recursos entre sistema Unix/Linux com sistemas Windows.

Boa parte das funcionalidades do Samba são implementadas por 2 daemons, sendo eles o `smbd` e `nmbd`. O `smbd` implementa serviço de arquivo impressão assim como autenticação e autoriação. O `nmbd` é responsável por um componente fundamental do SMB, o serviço de DNS e anúncio de serviço. 



## Instalação do Samba 4

Para instalar o Samba 4, nós devemos compilar a aplicação, versões anteriores eram disponibilizada já compilada e estava nos repositorios da maioria das aplicações.

[Wiki do Samba](https://wiki.samba.org/index.php/Build_Samba_from_Source#Introduction)



### Instalando Dependências - Samba AD DC

Antes de compilar o Samba, vamos instalar suas dependências.

```bash
$ sudo apt-get install -y acl apt-utils attr autoconf bind9utils binutils bison build-essential ccache chrpath curl debhelper dnsutils docbook-xml docbook-xsl flex gcc gdb git glusterfs-common gzip heimdal-multidev hostname htop krb5-config krb5-kdc krb5-user language-pack-en lcov libacl1-dev libaio-dev libarchive-dev libattr1-dev libavahi-common-dev libblkid-dev libbsd-dev libcap-dev libcephfs-dev libcups2-dev libdbus-1-dev libglib2.0-dev libgnutls28-dev libgpgme11-dev libgpgme-dev libicu-dev libjansson-dev libjs-jquery libjson-perl libkrb5-dev libldap2-dev liblmdb-dev libncurses5-dev libpam0g-dev libparse-yapp-perl libpcap-dev libpopt-dev libreadline-dev libsystemd-dev libtasn1-bin libtasn1-dev libunwind-dev lmdb-utils locales lsb-release make mawk mingw-w64 nettle-dev patch perl perl-modules perl-modules-5.30 pkg-config procps psmisc python3 python3-all-dev python3-crypto python3-cryptography python3-dbg python3-dev python3-dnspython python3-gpg python3-iso8601 python3-markdown python3-matplotlib python3-pexpect python3-pyasn1 python3-setproctitle rng-tools rsync sed sudo tar tree uuid-dev wget xfslibs-dev xsltproc zlib1g-dev
```

Você pode encontrar essa lista [aqui](https://wiki.samba.org/index.php/Package_Dependencies_Required_to_Build_Samba#Operating_System-independent_Overview), porém, a lista acima foi removido os pacotes relacionados ao Python 2.



```bash
# Contém pacotes com python 2, usar somente se necessário:
$ sudo apt-get install acl attr autoconf bind9utils bison build-essential \
  debhelper dnsutils docbook-xml docbook-xsl flex gdb libjansson-dev krb5-user \
  libacl1-dev libaio-dev libarchive-dev libattr1-dev libblkid-dev libbsd-dev \
  libcap-dev libcups2-dev libgnutls28-dev libgpgme-dev libjson-perl \
  libldap2-dev libncurses5-dev libpam0g-dev libparse-yapp-perl \
  libpopt-dev libreadline-dev nettle-dev perl perl-modules-5.30 pkg-config \
  python-all-dev python-crypto python3-dbg python-dev python-dnspython \
  python3-dnspython python3-gpg python-markdown python3-markdown \
  python3-dev xsltproc zlib1g-dev liblmdb-dev lmdb-utils libsystemd-dev libdbus-1-dev -y
```






### Baixar o código fonte do Samba 4

Para instalar o Samba, primeiro temos que baixar o código fonte para poder compilar, podemos obter o código fonte [aqui](https://download.samba.org/pub/samba/stable/).

Vamos ver algumas opções do script `configure`:

| Opção                                  | Descrição                                                    |
| -------------------------------------- | ------------------------------------------------------------ |
| --without-ad-dc                        | Desativa a funcionalidade de Active Directory Domain Controller. |
| --systemd-install-services             | Habilita a instalação de serviços do SystemD (use a opção abaixo também). |
| --with-systemddir=/lib/systemd/system/ | Coloca os serviços no diretório informado.                   |
| --sbindir=\<place>                     | Muda o local padrão para os binários (`smbd` e `samba`) do Samba, por default eles ficam em `/usr/local/samba/sbin/`. |
| --prefix=/etc/samba/                   | Local de instalação do Samba.                                |
| --sysconfdir=/etc/samba/               | Onde ficará o arquivo `smb.conf`.                            |
| --bindir=/usr/bin/                     | Onde ficarão os binários como: `smbpasswd`, `smbstatus` entre outros. |
| --libdir=\<place>                      | Onde ficarão as bibliotecas usadas pelo Samba.               |
| --libexecdir=\<place>                  | Onde ficarão os binários de bibliotecas.                     |
| --mandir=/usr/share/man/               | Onde ficarão os manuais, se você não especificar, eles ficarão no local de instalação do Samba. |
| --localstatedir=/var/samba/            | Onde ficarão os dados varíaveis do Samba.                    |
| --includedir=/usr/include/             | Onde ficarão os arquivos de cabeçalho, como códigos em C.    |



### Instalando o Samba AD-DC 

Aqui vamos instalar o samba completo, incluindo o AD-DC dele.

```bash
# Baixe a ultima versão do Samba:
wget https://download.samba.org/pub/samba/stable/samba-4.14.6.tar.gz

# Descompate o arquivo:
tar -zxvf samba-4.14.6.tar.gz

# Entre na pasta do Samba4:
cd samba-4.14.6

# O script de configuração (chamado configure), vai criar um arquivo chamado Makefile que é usado pelo comando make.
```



Mostrado os passos iniciais, vamos rodar o script de configure do Samba mudando alguns parametros:

```bash
### Vou mudar os diretórios dos binários e dos arquivos do Samba!

# Para samba ad dc, use o comando abaixo:
$ sudo ./configure --bindir=/usr/bin --sbindir=/usr/sbin --prefix=/etc/samba --sysconfdir=/etc/samba --libdir=/lib/samba --libexecdir=/lib/samba --mandir=/usr/share/man --localstatedir=/var/samba --includedir=/usr/include --pythonarchdir=/lib/samba --systemd-install-services --with-systemddir=/lib/systemd/system -j 3

# Faça a compilação real:
$ sudo make -j 3

# Agora vamos instalar o nosso código compilado:
$ sudo make install -j 3

# Final da instalação:
'install' finished successfully (1m59.085s)

# Agora vamos provisionar o Samba (Colocar ele como controlador do nosso domínio):
sudo samba-tool domain provision --realm=sysnetbr.eng.br --domain=sysnetbr --adminpass='p@ssw0rd' --server-role=dc --use-rfc2307 --dns-backend=SAMBA_INTERNAL

# server-role=dc - Nosso samba vai ser um controlador de domínio.
# --use-rfc2307 - Permite as extensões NIS.
# --dns-backend=SAMBA_INTERNAL - Qual backend para DNS o samba vai usar. O primeiro DC num AD deve ser instalado usando backend de DNS.
# realm=... - Nosso domínio completo.
# domain=... = Apenas a porção do domínio.
```



Vamos fazer algumas alterações relacionadas ao DNS:

```bash
sudo vim /etc/hosts
127.0.0.1 localhost
192.168.121.229 samba-host.sysnetbr.eng.br samba-host

# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters


# Setar ip fixo, no caso, o ip do samba é .229
```



Fazendo um backup do arquivo original do `krb5.conf`

```bash
sudo mv /etc/krb5.conf /etc/krb5.conf.bkp

# Faça o link apontando para o novo arquivo:
sudo ln -sf /etc/samba/private/krb5.conf /etc/krb5.conf
```



Inicie o serviço do Samba:

```bash
# Habilite no boot:
sudo systemctl enable samba.service 

# inicie o serviço:
sudo systemctl start samba.service 
```



Vamos fazer alguns testes:

```bash
# Verifique se o mapeamento do localhost está funcionando:
$ smbclient  -L localhost -U%

	Sharename       Type      Comment
	---------       ----      -------
	sysvol          Disk      
	netlogon        Disk      
	IPC$            IPC       IPC Service (Samba 4.14.6)
SMB1 disabled -- no workgroup available

# Vamos dar um 'ls' no mapeamento do netlogon:
smbclient  //localhost/netlogon -Uadministrator -c 'ls'
Enter SYSNETBR\administrator's password: 
  .                                   D        0  Fri Jul 16 18:57:03 2021
  ..                                  D        0  Fri Jul 16 18:57:11 2021

		129125532 blocks of size 1024. 116616928 blocks available
		

# Testar outro compartilhamento:
$ smbclient  //localhost/devops -Uadministrator -c 'ls'
Enter SYSNETBR\administrator's password: 
  .                                   D        0  Fri Jul 16 19:21:02 2021
  ..                                  D        0  Fri Jul 16 19:20:20 2021
  test                                N        0  Fri Jul 16 19:21:02 2021

		129125532 blocks of size 1024. 116616920 blocks available
```

### 

## Acertando as bibliotecas

```bash
# Verifique o seu diretório de libs:
smbd -b | grep LIBDIR
   LIBDIR: /lib/samba

# Criando links entre as bibliotecas do samba com as bibliotecas do sistema:
sudo ln -s /lib/samba/libnss_winbind.so.2 /lib/x86_64-linux-gnu/
sudo ln -s /lib/x86_64-linux-gnu/libnss_winbind.so.2 /lib/x86_64-linux-gnu/libnss_winbind.so

# Recarregando as bibliotecas:
sudo ldconfig

# Editando o arquivo /etc/nsswitch.conf:
sudo vim /etc/nsswitch.conf

# Deixe como abaixo:		
	passwd: files winbind
	group:  files winbind
	shadow: files

# Recarregando o samba:
sudo smbcontrol all reload-config

# Executando os testes a seguir:
$ getent passwd administrator
SYSNETBR\administrator:*:0:100::/home/SYSNETBR/administrator:/bin/false

$ wbinfo -u
SYSNETBR\administrator
SYSNETBR\guest
SYSNETBR\krbtgt
SYSNETBR\vagrant

$ wbinfo -g
SYSNETBR\cert publishers
SYSNETBR\ras and ias servers
SYSNETBR\allowed rodc password replication group
SYSNETBR\denied rodc password replication group
SYSNETBR\dnsadmins
SYSNETBR\enterprise read-only domain controllers
SYSNETBR\domain admins
SYSNETBR\domain users
SYSNETBR\domain guests
SYSNETBR\domain computers
SYSNETBR\domain controllers
SYSNETBR\schema admins
SYSNETBR\enterprise admins
SYSNETBR\group policy creator owners
SYSNETBR\read-only domain controllers
SYSNETBR\dnsupdateproxy
```







### Instalando Dependências - Samba Only FileServer

Vamos instalar o Samba com algumas funcionalidades a menos, vamos usar ele apenas para compartilhamento de arquivos.

```bash
# Instalando as dependencias:
sudo apt install -y acl apt-utils attr autoconf bind9utils binutils bison build-essential ccache chrpath curl debhelper dnsutils docbook-xml docbook-xsl flex gcc gdb git glusterfs-common gzip heimdal-multidev hostname htop language-pack-en lcov libacl1-dev libarchive-dev libattr1-dev libavahi-common-dev libblkid-dev libbsd-dev libcap-dev libcephfs-dev libcups2-dev libdbus-1-dev libglib2.0-dev libgnutls28-dev libgpgme11-dev libicu-dev libjansson-dev libjs-jquery libjson-perl libkrb5-dev libldap2-dev liblmdb-dev libncurses5-dev libpam0g-dev libparse-yapp-perl libpcap-dev libpopt-dev libreadline-dev libsystemd-dev libtasn1-bin libtasn1-dev libunwind-dev lmdb-utils locales lsb-release make mawk mingw-w64 patch perl perl-modules pkg-config procps psmisc python3 python3-cryptography python3-dbg python3-dev python3-dnspython python3-gpg python3-iso8601 python3-markdown python3-matplotlib python3-pexpect python3-pyasn1 python3-setproctitle rng-tools rsync sed sudo tar tree uuid-dev wget xfslibs-dev xsltproc zlib1g-dev

# Baixe a ultima versão do Samba:
git clone -b v4-14-stable git://git.samba.org/samba.git

# Descompate o arquivo:
tar -zxvf samba-4.14.6.tar.gz

# Entre na pasta do Samba4:
cd samba-4.14.6

# Para samba somente file server, use o comando abaixo:
$ sudo ./configure --without-ad-dc --bindir=/usr/bin --sbindir=/usr/sbin --prefix=/etc/samba --sysconfdir=/etc/samba --libdir=/lib/samba --libexecdir=/lib/samba --mandir=/usr/share/man --localstatedir=/var/samba --includedir=/usr/include --pythonarchdir=/lib/samba --systemd-install-services --with-systemddir=/lib/systemd/system --enable-debug --enable-selftest

# Corrija um bug
echo "^samba4.rpc.echo.*on.*ncacn_np.*with.*object.*nt4_dc" >> selftest/knownfail

sudo make quicktest

# Faça a compilação:
$ sudo make

# Agora vamos instalar o nosso código compilado:
$ sudo make install
```





smbd responsavel pelos compartilhamentos





sudo mount -t cifs //192.168.121.153/financeiro -o username='matias' /mnt/financeiro/
sudo mount -t cifs -o username=anderson //192.168.121.229/TI /mnt/myFolder/

precisa instalar: smb4k



```
/lib/systemd/system/smbd.service 

[Unit]
Description=Samba SMB Daemon
Documentation=man:smbd(8) man:samba(7) man:smb.conf(5)
Wants=network-online.target
After=network.target network-online.target nmbd.service winbind.service

[Service]
Type=notify
NotifyAccess=all
PIDFile=/run/samba/smbd.pid
LimitNOFILE=16384
EnvironmentFile=-/etc/default/samba
ExecStartPre=/usr/share/samba/update-apparmor-samba-profile
ExecStart=/usr/sbin/smbd --foreground --no-process-group $SMBDOPTIONS
ExecReload=/bin/kill -HUP $MAINPID
LimitCORE=infinity


[Install]
WantedBy=multi-user.target
```



```bash
[global]
	dns proxy = No
	log file = /var/log/samba/message.log
	logging = file
	map to guest = Bad User
	max log size = 1000
	obey pam restrictions = Yes
	pam password change = Yes
	panic action = /usr/share/samba/panic-action %d
	passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
	passwd program = /usr/bin/passwd %u
	server role = standalone server
	server string = %h server (Samba, Ubuntu)
	unix charset = ISO-8859-15
	unix password sync = Yes
	usershare allow guests = Yes
	idmap config * : backend = tdb

        vfs objects = recycle full_audit
#        full_audit:prefix = %U|%I|%m|%S
        full_audit:prefix = %u|%I
#        full_audit:success = connect, open, opendir, write, unlink, rename, mkdir, rmdir, chmod, chown
        full_audit:success = connect, write, unlink, rename, mkdir, create_file, rmdir, chmod, chown, disconnect
        full_audit:failure = none
        full_audit:facility = LOCAL7
        full_audit:priority = ALERT

        unix charset = ISO8859-1
        unix extensions = No

[Public]
	comment = Todos acessam
	guest ok = Yes
	path = /Public
	read only = No
	vfs objects = recycle
	recycle:repository = /Public/.recycle
	recycle:keeptree = yes
	recycle:versions = yes

[TI]
	comment = Somente TI acessa
	force group = ti
	path = /TI
	read only = No
	write list = root @ti
	vfs objects = recycle
	recycle:repository = /TI/.recycle
	recycle:keeptree = yes
	recycle:versions = yes

```







## Fontes

https://dev.tranquil.it/wiki/SAMBA_-_Fichier_systemd_samba4

https://unix.stackexchange.com/questions/291759/install-samba4-4-4-from-source-on-centos7

https://wiki.samba.org/index.php/Build_Samba_from_Source#Creating_the_systemd_Service_File

https://wiki.samba.org/index.php/Build_Samba_from_Source

https://wiki.samba.org/index.php/Build_Samba_from_Source#Creating_the_systemd_Service_File

https://wiki.samba.org/index.php/Setting_up_Samba_as_a_Standalone_Server

https://download.samba.org/pub/samba/stable/

https://wiki.samba.org/index.php/Distribution-specific_Package_Installation

https://wiki.samba.org/index.php/Build_Samba_from_Source#Customizing_the_Configure_Options (mostra como aplicar patch tbm)

https://www.samba.org/samba/docs/current/man-html/vfs_full_audit.8.html

https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html

https://www.samba.org/samba/docs/current/man-html/vfs_recycle.8.html