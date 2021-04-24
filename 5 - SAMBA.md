[toc]



# Introdução ao Samba

Samba é um software desenvolvido por Andrew Tridgell, seu desenvolvimento teve inicio em 1991. Mais tarde esse protocolo tornou-se Server Message Block (SMB), o predecessor do CIFS. Um tempo depois, Andrew expandiu o SMB e começou a distribuí-lo como um produto gratuito na Internet sob o nome de "Servidor SMB". No entanto, ele não conseguiu manter esse nome porque já pertencia a outro produto de uma outra empresa (Microsoft), então foi dado o nome de Samba.

Samba é um pacote de software que permite que um sistema baseado em Unix apareça e funcione como um servidor Microsoft Windows quando visualizado por outros sistemas em uma rede, como AD (Active Directory), File Server (Servidor de arquivo que pode ser usado para máquinas Windows e Linux/Unix) dentre outras funcionalidades.

Com o passar do tempo, o Samba vem criando mais e mais funcionalidades para substituir o AD do Windows Server como uma maneira gratuita de se obter tais funcionalidades, não só isso, traz a integração no compartilhamento de recursos entre sistema Unix/Linux com sistemas Windows.



## Instalação do Samba 4

Para instalar o Samba 4, nós devemos compilar a aplicação, versões anteriores eram disponibilizada já compilada e estava nos repositorios da maioria das aplicações.

[Wiki do Samba](https://wiki.samba.org/index.php/Build_Samba_from_Source#Introduction)



### Instalando Dependências

Antes de compilar o Samba, vamos instalar suas dependências.

```bash
$ sudo apt-get install acl attr autoconf bind9utils bison build-essential \
  debhelper dnsutils docbook-xml docbook-xsl flex gdb libjansson-dev krb5-user \
  libacl1-dev libaio-dev libarchive-dev libattr1-dev libblkid-dev libbsd-dev \
  libcap-dev libcups2-dev libgnutls28-dev libgpgme-dev libjson-perl \
  libldap2-dev libncurses5-dev libpam0g-dev libparse-yapp-perl \
  libpopt-dev libreadline-dev nettle-dev perl perl-modules-5.30 pkg-config \
  python3-all-dev python3-crypto python3-dbg python3-dnspython \
  python3-gpg python3-markdown \
  python3-dev xsltproc zlib1g-dev liblmdb-dev lmdb-utils libsystemd-dev libdbus-1-dev -y
```

Você pode encontrar essa lista [aqui](https://wiki.samba.org/index.php/Package_Dependencies_Required_to_Build_Samba#Operating_System-independent_Overview)



```bash
# Usar em ultimo caso:
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

```bash
# Baixe a ultima versão do Samba:
wget https://download.samba.org/pub/samba/stable/samba-4.14.3.tar.gz

# Descompate o arquivo:
tar -zxvf samba-4.14.3.tar.gz

# Entre na pasta do Samba4:
cd samba-4.14.3

# O script de configuração (chamado configure), vai criar um arquivo chamado Makefile que é usado pelo comando make.

# Se não formos usar a funcionalidade de AD-DC, podemos desativar esse recurso usando a opção '--without-ad-dc'.

# O padrão para os binários do Samba é ficar em /usr/local/samba/sbin/, mas podemos mudar isso, basta usar a opção "--sbindir=Novo Local"

# --sysconfdir=/etc/samba/ muda o local dos arquivos de configuração do Samba.
```



Mostrado os passos iniciais, vamos rodar o script de configure do Samba mudando alguns parametros:

```bash
# Vou mudar os diretórios dos binários e dos arquivos de configuração:
$ sudo ./configure --sysconfdir=/etc/samba/ --sbindir=/sbin/
```

