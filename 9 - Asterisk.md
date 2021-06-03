[toc]



# Introdução

O Asterisk implementa em software os recursos encontrados em um PABX convencional, usando a tecnologia VOIP.

O VoIP (Voz sobre IP) é uma tecnologia que faz as chamadas telefônicas via internet de banda larga ao invés de termos que usar uma linha de telefone analógica. Usa o protocolo TCP/IP para trafegar as informações.

Sua principal função é redução de custos e integração de todas as plataformas que usam essa tecnologia. Alguns exemplos que temos hoje é o Whatssap, Telegram e Skype, onde todos fornecem um meio de fazer chamadas de voz usando a Internet.



## Protocolos

O VoIP usa alguns protocolos para sinalização de chamadas, como: SIP, H.323, H.248, IAX dentro outros. A função desses protocolos é cuidar da sessão estabelecida entre origem e destino, sendo assim, podemos saber se uma chamada foi estabelecida, quanto tempo durou dentre outras informações.



### SIP

O Asterisk não é um SIP Proxy nem Redirector, ele é uma Meida Gateway, ou seja, ele conecta dois canais de SIP como se fossem canais de um PABX.



### IAX

Foi desenvolvido com a finalidade de estabelecer comunicação entre servidores Asterisk e suas derivações.

Com ele podemos economizar banda, ele aproveita um único cabeçalho IAX para multiplas ligações simultaneas e facilita o transporte de redes com Firewall e/ou NAT.



## Codecs

Sua função é codificar e decodificar sinais, tranformam um sinal analógico em digital e vice-versa. Sua outra utilidade é a de poder comprimir (reduzir) o tamanho de um sinal digital, com isso podemos otimizar o consumo de banda.

### G711

- G711a

  Padrão mais usado nas operadoras do Brasil.

- G711a (alaw)

  Usa 64 kbps (um pouco alto), mas seu perfil é de baixa compressão, o que não necessita de muito poder de CPU.

- G711u (ulaw)

  Usa 64 kbps (um pouco alto), mas seu perfil é de baixa compressão, o que não necessita de muito poder de CPU.

Esses codecs são indicados para rede LAN.



### GSM

Mesmo padrão usado pelos telefones celulares. Usa 13,2kbps por chamada, com isso já podemos ver que ele usa compressão e possui uma certa latência.

Recomendado para redes WAN.



### Outros Codecs

- G729a (Esse codec é pago)

  Codec de propriedade da Digium.

  Usa apenas 8kbps, possui maior compressão e requer mais uso de CPU e pode ter uma latencia um pouco maior.

  

- ILBC

  Sua maior habilidade é ser resistente a perda de pacotes, usa apenas 13,3 ou 15 Kbps.

  

- G.723.1

  Usa apenas 5,3 ou 6,3 Kbps, acaba exigindo muito poder de processamento.



Caso os Codes usados pela origem e destino sejam diferentes, o Asterisk faz a transcodificação, ou seja, ela vai converter o formato do Codec e isso pode gerar latência.

A prioridade dos codecs é sempre definido pelo dispositivo que inicia a chamada.



## Instalação no Ubuntu

Precisamos baixar e instalar os pacotes: Dahdi, Libpri e Asterisk. Ambos devem ser compilados.

```bash
### Faça todo o processo com root!

# Instalando todas as depedencias necessárias para o Asterisk funcionar corretamente:
add-apt-repository multiverse && apt update

# Instale as dependencias:
apt install -y autoconf automake build-essential git libedit-dev libjansson-dev libncurses5-dev libnewt-dev libsqlite3-dev libssl-dev libtool libxml2-dev linux-headers-$(uname -r) pkg-config sqlite3 subversion uuid-dev wget 

# ---------------------------------------------- #
### Instalando o DAHDI              	         #
# ---------------------------------------------- #

# Entrando do diretório /usr/src:
cd /usr/src/

# Clonar o repo do Dahdi do Git:
git clone -b next git://git.asterisk.org/dahdi/linux dahdi-linux

# Entrar na pasta:
cd dahdi-linux

# Iniciar a compilação:
make -j3

# Instalar:
make install

# Final da compilação:
###################################################
###
### DAHDI installed successfully.
### If you have not done so before, install the package
### dahdi-tools.
###
###################################################

# ---------------------------------------------- #
### Instalando o Dahdi Tools        	         #
# ---------------------------------------------- #

# Entrando do diretório /usr/src:
cd /usr/src/

# Clonar o repo do Dahdi-tools do Git:
sudo git clone -b next git://git.asterisk.org/dahdi/tools dahdi-tools

# Entrar na pasta:
cd dahdi-tools

# Compilar:
autoreconf -i
./configure
make -j3
make install
make install-config
dahdi_genconf modules

# ---------------------------------------------- #
### Instalando o libpri na versão 1.6"	         #
# ---------------------------------------------- #

# Entrando do diretório /usr/src:
cd /usr/src/

# Baixando o código fonte do Git:
git clone https://gerrit.asterisk.org/libpri libpri

# Entrando na pasta:
cd libpri

# Compilar a libpri:
make -j3
make install

# ---------------------------------------------- #
### Instalando o Asterisk           	         #
# ---------------------------------------------- #

# Entrando do diretório /usr/src:
cd /usr/src/

# Baixando a versão 16.8.0 do Asterisk
git clone -b 18 https://gerrit.asterisk.org/asterisk asterisk-18

# Entrando no diretório Asterisk:
cd asterisk-18 

# Rode o script abaixo, instala o que for necessário para criar o módulo MP3, assim podemos usar arquivos .mp3 no asterisk:
contrib/scripts/get_mp3_source.sh

# Rode o script para instalar as dependencias necessarias:
contrib/scripts/install_prereq install

# Compilação e instalação completa do Asterisk:
./configure --with-jansson-bundled 
make menuselect 
make -j3
make install 
make config
ldconfig

# Menuselect é um menu onde podemos escolher colocar ou não alguns módulos no Asterisk.

# Criando exemplos de Arquvios de configurações do Asterisk em /etc/asterisk como sip.conf, extensions.conf, entre todos os outros.(comando opcional):
make samples && cd 

# Iniciando o Dahdi e Asterisk:
systemctl restart asterisk

# Entrando em modo verbose do Asterisk
asterisk -vvvvvvc

# Habilitar no boot:
systemctl enable asterisk

# Reiniciar o serviço:
systemctl restart asterisk

# Entrar na cli do asterisk usando o verbose:
asterisk -vvvvvvvvvvvvr
```



## Instalação no CentOS

```bash
# Atualize os pacotes e instale o wget e vim:
yum update &&  yum -y install wget vim

# Desabilite o SELinux, editando o arquivo /etc/selinux/config:
vim /etc/selinux/config

## Deixe SELINUX igual abaixo:
	SELINUX=disabled

# Agora, vamos desativar o firewall:
systemctl stop firewalld ; systemctl disable firewalld ; init 6

# Agora, vamos instalar o EPEL:
wget http://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm

# Instale o pacote:
rpm -ivh epel-release-latest-7.noarch.rpm

# Instalando todas as depedencias necessárias para o Asterisk funcionar corretamente:
yum install openssl-devel ncurses-devel newt-devel libxml2-devel sqlite-devel libuuid-devel git subversion libsqlite3x-devel autoconf automake libtool libedit-devel kernel-devel gcc-c++ bzip2 patch -y ; yum groupinstall 'Development Tools'

# ---------------------------------------------- #
### Instalando o Dahdi              	         #
# ---------------------------------------------- #

# Entrando do diretório /usr/src:
cd /usr/src/

## Baixando o Dahdi na versão 3:
wget downloads.asterisk.org/pub/telephony/dahdi-linux-complete/dahdi-linux-complete-3.0.0+3.0.0.tar.gz

# Descompactando o Dahdi:
tar zxvf dahdi-linux-complete*

# Entrando no diretório do Dahdi:
cd dahdi-linux-complete

# Instalando o Driver Dahdi:
make && make install && make config

# ---------------------------------------------- #
### Instalando o libpri na versão 1.6"	         #
# ---------------------------------------------- #

# Instalando o libpri na versão 1.6"
cd /usr/src/

# Baixando o pacote:
wget downloads.asterisk.org/pub/telephony/libpri/libpri-1.6.0.tar.gz

# Descompactando:
tar zxvf libpri*

# Entrando na pasta:
cd libpri*

# Compilando:
make && make install 

# ---------------------------------------------- #
### Instalando a Libjanssons           	         #
# ---------------------------------------------- #

# Instalando o libjanssons:
cd /usr/src/ && wget www.digip.org/jansson/releases/jansson-2.11.tar.gz

# Descompactando:
tar -zxf jansson-*

# Entrando na pasta:
cd jansson*

# Compilando:
./configure --prefix=/usr/ && make clean && make && make install && ldconfig

# ---------------------------------------------- #
### Instalando o Asterisk           	         #
# ---------------------------------------------- #

# Entrando do diretório /usr/src:
cd /usr/src/

# Baixando a versão 16.0.1 do Asterisk
wget downloads.asterisk.org/pub/telephony/asterisk/asterisk-16.1.1.tar.gz

# Descompactando o Asterisk:
tar zxvf asterisk*  

# Entrando no diretório Asterisk:
cd asterisk* 

# Compilação e instalação completa do Asterisk:
./configure &&  make menuselect && make && make install && make config

# Criando exemplos de Arquvios de configurações do Asterisk em /etc/asterisk como sip.conf, extensions.conf, entre todos os outros.(comando opcional):
make samples && cd

# Reiniciando o Asterisk:
systemctl restart asterisk

# Entrando em modo verbose do Asterisk
asterisk -vvvvvvr
```



## Criando um usuário para o Asterisk

Por padrão o Asterisk roda sobre o usuário root, o que abre muitas brechas de segurança, por isso vamos colocar ele para rodar sobre outro usuário.

```bash
sudo adduser --system --group --home /var/lib/asterisk --no-create-home --gecos "Asterisk PBX" asterisk
```

Com o usuário criado, precisamos configurar isso nele, assim ele saberá que deve rodar usando outro usuário.

```bash
# Edite o arquivo:
vim /etc/default/asterisk

### Deixe as opções como abaixo:
AST_USER="asterisk"
AST_GROUP="asterisk"
```



Adicione o usuário `asterisk` aos grupos `dialout` e `audio`:

```
sudo usermod -a -G dialout,audio asterisk
```



Agora vamos alterar o User e Group dono dos arquivos do Asterisk:

```bash
sudo chown -R asterisk: /var/{lib,log,run,spool}/asterisk /usr/lib/asterisk /etc/asterisk
sudo chmod -R 750 /var/{lib,log,run,spool}/asterisk /usr/lib/asterisk /etc/asterisk
```



Reinicie o serviço:

```bash
sudo systemctl restart asterisk
```



## Firewall

Por padrão o SIP usa a porta 5060 UDP, se você estiver usando o RTP (Real time Protocol) você vai precisar liberar o range de porta de 10000 até 20000.



## Arquivos de configurações

O comando mais usado para trabalhar com Asterisk é o `asterisk`, vejamos algumas opções dele:

| Comando                                    | Descrição                                                    |
| ------------------------------------------ | ------------------------------------------------------------ |
| -r                                         | Ao invés de tentar rodar um novo processo do Asterisk, essa opção tenta se conectar a um processo que já esteja rodando. |
| -v                                         | Habilita o verbose, quantos mais `v` nós colocarmos, mais verbose teremos, o máximo é `vvv`. |
| -V                                         | Mostra a versão.                                             |
| **Dentro da CLI**                          |                                                              |
| core show version                          | Além de ser similar ao `-V`, mostra algumas informações a mais. |
| core show applications                     | Mostra todos os comandos disponiveis para uso e mostra também uma pequena introdução do que ele faz. |
| core show application Dial                 | Mostra uma documentação do comando Dial.                     |
| core set verbose                           | Configura o nivel de verbosidade.                            |
| core show channels                         | Mostra quantas chamadas estão em andamento e em processo.    |
| core restart \<gracefully ou now ou when\> | Reiniciar o Asterisk.<br />now = Reinicia agora;<br />when = Quando não houver mais chamadas;<br />gracefully = Não vai permitir novas chamadas, e quando finalizar todas as chamadas, ele vai reiniciar. |
| module show like sip                       | Mostra os módulos que estão rodando.                         |
| module load chan_sip.so                    | Carrega o módulo chamado chan_sip.so                         |
| sip show peer 2001                         | Exibe informações do ramal 2001                              |
| sip reload                                 | Recarrega as informações do SIP.                             |

Outra forma de reiniciar é usando o systemd (`sudo systemctl restart asterisk`).



A maioria dos arquivos de configuração do Asterisk fica em `/etc/asterisk/`. Os arquivos principais são: `iax.conf`, `sip.conf` e `ext*.conf` (são os arquivos extensions).

Todos os arquivos de configuração seguem o padrão abaixo:

```bash
[sessão]
Parametro = Valor

### Existem 2 sessões importantes!

[global]
# Define as variáveis globais para o arquivo de configuração inteiro.

[gerenal]
# Define apenas para as sessões seguintes a ele, não define nada para sessão que for criada antes dele, apenas depois.
```



## Criando um canal de comunicação SIP

Aqui vamos trabalhar com o arquivo `sip.conf`.

```bash
# Renomeie esse arquivo (no meu caso é um arquivo de exemplo).
sudo mv /etc/asterisk/sip.conf /etc/asterisk/sip.conf.old

# Crie um arquivo novo:
sudo bash -c "> /etc/asterisk/sip.conf"

# Atribua o dono e grupo correto do arquivo:
sudo chown asterisk. /etc/asterisk/sip.conf

# Aplique as permissoes corretas:
sudo chmod 750 /etc/asterisk/sip.con
```



Agora vamos editar esse arquivo, a saída abaixo será o código dentro do arquivo.

```bash
[general]
allowguest=no
; Não criar um tipo de canal sip automaticamente:
autocreatepeer=no
awayssauthreject=yes
; Qual IP:porta que ele vai aceitar (qualquer IP no caso)
; Tente mudar a porta por sec!
udpbindaddr=0.0.0.0:5060
; Contexto do extension.conf chamado ramais:
context=ramais
; IP do seu linux (IP local):
localnet=192.168.122.217/255.255.255.0
; Caso queira contratar um operadora VoIP, use o register:
; register=senha:login@hostoperadoravoip:5060
externip=meuip.com.br:5060
;
;; Criando um Ramal
;
; Nome do nosso ramal:
[2001]
; Nome do user:
defaultuser=2001
; Senha:
secret=12345
; Nosso telefone faz e recebe chamadas:
; Outros tipos são: peer (só faz chamada), user (Só recebe chamadas)
type=friend
; O host (end-point) sempre vai ter o IP alterado, fica mudando:
host=dynamic
; Não vai ter nat no meio da conf:
nat=no
dtmfmode=auto
canreinvite=no
regext=2001
disallow=all
allow=ullaw,gsm
callgroup=1
pickupgroup=1
```

Após alterar algumas configurações, vamos aplicar elas:

```bash
# Entre na cli do asterisk:
sudo asterisk -vvvr

# Agora recarregue as mudanças:
sip reload

## Se não funcionar, rode o comando abaixo:
sudo systemctl reload asterisk.service
```



### PJSIP

Para obter o conversor de `sip.conf` para `pjsip.conf` siga o passo a passo abaixo:

```bash
# Copie a pasta para o asterisk, isso não é necessário, mas eu acho mais fácil:
cp -r /usr/src/asterisk-18/contrib/scripts/ /etc/asterisk/

# Aplique as confs de user, grupo e permissão:
sudo chown -R asterisk: /etc/asterisk/scripts
sudo chmod -R 750 /etc/asterisk/scripts
```















































































































































































# Fontes

https://linuxize.com/post/how-to-install-asterisk-on-ubuntu-20-04/