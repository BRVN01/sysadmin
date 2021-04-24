[toc]



# Introdução ao DNS

O Domain Name System é responsavel pela tradução de Nome para IP e IP para nome. Criado e mantido pela ISC (Internet System Consortium), mesmo grupo que mantem o DHCP e NTP.

É dividido em gTLD (Dominio Generico como: .com, .edu, .gov, .mil e etc) e ccTLD (codigo de paises ou country-code), sempre com duas letras ".br". 

A ICANN delega, de acordo com os tratados internacionais, a responsabilidade pela administração de um "ccTLD".



Resolução Recursiva



Tomando  um  navegador  web  como  exemplo,  a  resolução para  acesso  a  um  "website"  tem  as seguintes etapas:

1) Usuário solicita acesso a "www.exemplo.com.br";

2) Navegador checa se já conhece o endereço IP do "hostname", solicitado (cache do"browser");

3) Se não conhece, o navegador passa a solicitação para a biblioteca de resolução – o resolver";

4) O "resolver" procura o "hostname" solicitado no arquivo "/etc/hosts" local;

5) Se não encontrar, ele checa o arquivo "/etc/resolv.conf" para saber quais "name-servers" deve solicitar a informação;

6) O "resolver" repassa a solicitação ao primeiro "nameserver" da lista, e logo após para o próximo até o fim da lista, aguardando por uma resposta de qualquer um deles;

7) O servidor de nomes acionado consulta seu "cache", se houver;

8) Se não  encontrar em  seu  "cache",  o servidor  em  questão  vai diretamente ao  servidor raiz  e transfere a consulta - www.exemplo.com.br;

9) O servidor raiz não faz "cache", e também não é autoridade sobre zonas de baixo;

10) Nível, então ele apenas responde uma parte da questão: "Não sei quem é, mas sei quem pode responder melhor: br.";

![Selection_146](/home/bruno/Desktop/4Linux - 4752/Semana 1/Aula 2/IMG/Selection_146.png)



11) O servidor de nomes reenvia a consulta para o servidor ".br-www.exemplo.com.br;

12) .".br" retorna o mesmo tipo de resposta, porém como uma dica mais próxima: "Não sei quem é, mas sei quem pode responder melhor: com.br.";

13) Passos 11 e 12 são efetuados mais uma vez, e agora a resposta é "Não sei quem é, mas sei quem pode responder melhor: exemplo.com.br.";

14)  Após  repetir  o  passo  10,  finalmente  a  resposta será  da  autoridade  sobre  o  domínio exemplo.com.br. Vai ser respondido o IP, juntamente ao TTL do registro, ou será respondido "inexistente";

15)  O  servidor  de  nomes  fará  "cache"  da  resposta,  ao  mesmo  tempo  que  repassa  para  o resolvedor original;

16) O resolvedor repassa a resposta para o navegador;

17) O navegador inicia uma conexão "HTTP" com o IP descoberto.



### Server autoritativo
Server autoritativo de um dominio, possui os registros originais que associam aquele dominio a seu endereço IP. Tem sua responsabilidade na humanindade.



### Server de Cache

Um cache DNS guarda localmente os resultados das pesquisas para utilização futura, evitando a repetição de pesquisas e aumentando drasticamente a velocidade de respostas.



## Tipos de registros DNS

S

| Registro | Descroção                                                    |
| -------- | ------------------------------------------------------------ |
| SOA      | Start of Authority - Indica onde começa a autoridade da zona; |
| NS       | Name Server - Indica um servidor de nome para a zona;        |
| A        | Endereço IPv4                                                |
| AAAA     | Endereço IPv6;                                               |
| MX       | Servidor de E-mail;                                          |
| CNAME    | Canonical Name - Mapeia um nome alternativo;                 |
| PTR      | Pointer Record (IP reverso)                                  |
| TXT      | Permite incluir uma entrada de texto curto, mais usado para SPF. |

```
O que é o SPF?

SPF (Sender Policy Framework), é um sistema que evita que outros domínios, enviem e-mails não autorizados em nome de um domínio.

Domínio pode ou não autorizar que outros IP’s fora desta relação enviem e-mails em seu nome. O administrador configura esta relação na entrada TXT da zona de DNS seguindo as regras da RFC 4408.

A entrada SPF, pode ter 4 atributos diferentes que alteram o modo como o servidor de destino tratará mensagens enviadas de servidores que não estejam presentes na mesma.

 Tipos de Atributos

 + Pass
Significa que o IP está autorizado a enviar mensagens em nome do domínio, sendo que o domínio consultado pode então, ser considerado responsável pelo envio da mensagem.

 – Fail
Significa explicitamente que o IP não está autorizado a enviar mensagens em nome do domínio consultado. Este resultado pode ser utilizado para rejeitar a mensagem.

 ~ SoftFail
Deve ser tratado como um resultado intermediário entre os níveis fail e neutral. Neste caso, o domínio consultado informa que acha que o IP não está autorizado, mas que não pode fazer uma afirmação taxativa. A mensagem não deve ser rejeitada apenas com base neste resultado, mas é recomendável submetê-la a outros testes.

 ? Neutral
O dono do domínio não tem como ou não quer definir se um determinado endereço IP está ou não autorizado a enviar mensagens em nome do domínio. Este resultado deve ser tratado exatamente como se não existisse um registro SPF.
```





# Executar Diagnostico usando DNS externo

```bash
# Consultando o servidor de e-mail do google
host -t mx google.com

# Qual é o endereço do servidor webmail.4linux.com.br
host -t A webmail.4linux.com.br

# quais são os servidores DNS da 4linux?
host -t NS 4linux.com.br
```



# Instalação do DNS

```bash
# Instalando o bind9 e dnsutils:
sudo apt install bind9 dnsutils -y

## Iniciar e ativar o bind9 na inicialização:
# Iniciar o daemon:
sudo systemctl start bind9

# Iniciar na inicialização:
sudo systemctl enable bind9

# Configurar nosso servidor dns para usar a sí mesmo
# como dns:
sudo vim /etc/resolv.conf
nameserver 172.16.100.203

# Arquivo que armazenam os endereçõs dos root servers:
cat /etc/bind/db.root

# Arquivos principais:
cat /etc/bind/named.conf
cat /etc/bind/named.conf.local
# Sendo que o primeiro é usado para personalizar as opções referentes ao funcionamento do próprio "BIND", enquanto o segundo serve para declarar as zonas pelas quais este servidor deve responder.


# Definições do bind:
cat /etc/bind/named.conf.options

# Arquivo de zonas:
cat /etc/bind/named.conf.default-zones
```



# Configurando o servidor master

```bash
# Vamos configurar nossa zona interna:
vim /etc/bind/named.conf.local

zone "dexter.com.br" {
    type master;
    file "db.dexter";
};

# Verificando se existe algum erro:
named-checkconf
```



## TIpos de zonas

| Zonas           | Descrição                                                    |
| --------------- | ------------------------------------------------------------ |
| MASTER          | Zona de autoridade sobre o domínio. Os dados da "zona" serão criados, publicados e administrados a partir deste ponto. |
| SLAVE           | Basicamente uma “cópia” da zona original, nenhuma criação ou alteração respectiva a essa "zona" será feita diretamente neste DNS. |
| STUB            | Tipo de "zona" similar a Slave não é previsto em nenhuma "RFC", foi implementado apenas no "BIND". |
| HINT            | Específica para o "BIND" onde ele deve começar uma busca recursiva, quando estiver operando como "cache". |
| FORWARD         | Serve para orientar o "BIND", e encaminhar a consulta sobre uma determinada "zona" para outro servidor em especial. |
| DELEGATION-ONLY | Utilizada para evitar abusos de algumas autoridades, sobre domínios de primeiro. |



Configurar o registro do DNS

```bash
# Depois de aplicar, precisa reiniciar o serviço (rndc reload).
vim /var/cache/bind/db.dexter

$TTL	604800
@	IN	SOA	ns1.dexter.com.br. root.dexter.com.br. (
			 2018080801	; Serial
			 604800		; Refresh
			 86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@		IN	NS	ns1.dexter.com.br.
ns1		IN	A	172.16.100.203
@		IN	A	172.16.100.203

security	IN	A	172.16.100.201
storage		IN	A	172.16.100.202
web01		IN	A	172.16.100.203
web02		IN	A	172.16.100.204
proxy		IN	A	172.16.100.205
mail		IN	A	172.16.100.206
devops		IN	A	172.16.100.207

www		IN	A	172.16.100.205
intranet	IN	A	172.16.100.204

webmail		IN	CNAME	www
sarg		IN	CNAME	security

@		IN	MX	10 mail.dexter.com.br.
smtp		IN	CNAME	mail
smtps		IN	CNAME	mail
pop		IN	CNAME	mail
pops		IN	CNAME	mail
imap		IN	CNAME	mail
imaps		IN	CNAME	mail

ldap		IN	CNAME	storage

@	IN	TXT	"v=spf1 a mx ip4:172.16.100.0/24 -all"

```



### Informações encontradas em um Registro de Zona

| Registro             | Descrição                                                    |
| -------------------- | ------------------------------------------------------------ |
| Dono                 | É o nome do registro. Quando substituído pelo símbolo "@", o dono é o próprio domínio. Caso o dono fique em branco, o "BIND"assume o nome do registro imediatamente superior; |
| TTL                  | Um valor em segundos, para a permanência dos dados deste registro no "cache"de um servidor. Raramente utilizado. classe - Podem ser "CH" (Chaos), "HS" (Hesiod) ou "IN" (Internet). |
| Tipo                 | No momento existem mais de 30 tipos de registro, dentre os quais veremos "SOA", "NS", "MX", "A", "CNAME", "TXT" e "PTR". |
| Serial               | É a referência para os "slaves" saberem se a "zona" sofreu alterações; |
| Refresh              | Tempo que o servidor secundário vai aguardar até checar se há atualizações no servidor primário; |
| Retry                | Em caso de falha do "refresh", o tempo até a próxima verificação; |
| Expire               | O tempo que o secundário aguardará o primário voltar, se esgotar, o secundário para de responder por essa zona; |
| Negative caching TTL | Se a zona expirar, esse será o tempo pelo qual um servidor "cache" armazenará a informação "NXDOMAIN", antes de iniciar uma nova busca recursiva. O máximo são 3 horas. |



Diagnosticos

```bash
# Verificar os logs:
tail /var/log/daemon.log

# testar onde começa a autoridade da zona do DNS para dexter:
dig -t soa dexter.com.br

# Vamos usar o comando host:
host ns1.dexter.com.br
	ns1.dexter.com.br has address 172.16.100.203

host dexter.com.br
	dexter.com.br has address 172.16.100.203
	dexter.com.br mail is handled by 10 mail.dexter.com.br.

host intranet.dexter.com.br
	intranet.dexter.com.br has address 172.16.100.204
```



### Usando a ferramenta RNDC

O comando rndc (Remote Named Daemon Control) é uma ferramenta de gerenciamento do named. A vantagem dessa ferramenta, é que ela permite controlar o named muito facilmente, sem ter que ficar enviando sinais ao processo do mesmo.

```bash
# Exibir o status do servidor DNS:
rndc status

# Recarregar o arquivo de configuração e zonas:
rndc reload

# Atualizar o cache:
rndc dumpdb -cache

# Limpa entradas no cache do servidor:
rndc flush
```





# Configurando Slave e Reverso

DNS reverso é um recurso que permite que outros servidores verifiquem a autenticidade do seu servidor. Para isso, ele checa se o endereço IP atual bate com o endereço IP informado pelo servidor DNS. Os servidores de e-mail irão recusar seus e-mails ou classificá-los como spam, caso o DNS reverso não esteja configurado.

```bash
## Vamos configurar o reverso
# Apontamento da base de dados:
vim /etc/bind/named.conf.local

# Adicione a conf ao arquivo 
zone "100.16.172.in-addr.arpa" {
	type master;
 file "rev.dexter";
};

# verificando a conf
named-checkconf

# Configurando a zona reversa:
vim /var/cache/bind/rev.dexter

$TTL	604800
@	IN	SOA	ns1.dexter.com.br. root.dexter.com.br. (
			 2018080801	; Serial
			 604800		; Refresh
			 86400		; Retry
			2419200		; Expire
			 604800 )	; Negative Cache TTL
;
@		IN	NS	ns1.dexter.com.br.
ns1		IN	A	172.16.100.203
206		IN	PTR	mail.dexter.com.br.

# Recarregue o serviço
rndc reload

# verificando erro de sintaxe nesse aquivo:
named-checkzone 172.16.100.206 /var/cache/bind/rev.dexter
	zone 172.16.100.206/IN: loaded serial 2018080801
	OK

# Usando o dig para fazer consulta reversa:
dig -x 172.16.100.206
```



# Configurar o Slave

Os servidores DNS Secundários ajudam a fornecer equilíbrio de carga e tolerância a falhas. Os servidores DNS secundários, mantêm uma cópia somente leitura dos dados da zona transferidos periodicamente do servidor DNS Primário.

```bash
# Reconfigure a zona dexter para ter um servidor slave:
vim /etc/bind/named.conf.local

zone "dexter.com.br" {
	type master;
        file "db.dexter";
	allow-transfer { 172.16.100.204; };
	notify yes;
	also-notify { 172.16.100.204; };
};

zone "100.16.172.in-addr.arpa" {
	type master;
        file "rev.dexter";
	allow-transfer { 172.16.100.204; };
	notify yes;
	also-notify { 172.16.100.204; };
};

# no arquivo abaixo, adicione a linha no final:
vim /etc/bind/named.conf.options
	allow-query { localhost; 172.16.100.0/24; 192.168.200.0/24; };
	
# Verifique erros:
named-checkconf

# Restart all
rndc reload
```



### Transferência de Zona consiste no Servidor DNS

O servidor primário passa toda a configuração de uma determinada Zona. Pensando em segurança, essa ação sempre deve ser restrita apenas a servidores DNS autorizados a receber as suas configurações. É muito comum SysAdmins deixar a Transferência de Zona aberta no Servidor, causando uma brecha de segurança. Para testar essa brecha, execute o seguinte comando:

```bash
dig dexter.com.br axfr
```



### Diretiva de transferência de Zona

- `allow-transfer` – Restringir a transferência de zona, apenas para Servidores Autorizados;

- `notify` e `also-notify` – As opções determinam se o servidor primário notifica servidores secundários, quando a informação de zona for atualizada.



Agora no servidor DNS secundario, execute:

```bash
# Instalando o DNS
yum install bind bind-utils -y

# Configure a zona:
vim /etc/named.conf

zone "dexter.com.br" {
        type slave;
        masters { 172.16.100.203; };
        file "/var/named/slaves/db.slave.dexter";
};

zone "100.16.172.in-addr.arpa" {
        type slave;
        masters { 172.16.100.203; };
        file "/var/named/slaves/rev.slave.dexter";
};

vim /etc/resolv.conf
nameserver 172.16.100.204
```



## Liberando regras no FirewallD

```bash
# Liberar consultas DNS:
firewall-cmd --zone=public --add-service=dns --permanent 
firewall-cmd --reload

# Iniciar e ativar na inicialização o serviço de DNS:
systemctl start named
systemctl enable named

# Testar a transferencia e verificar se os registros foram
# copiados, a partir do DNS primario:
dig dexter.com.br axfr
ls /var/named/slaves/
```



**Diretiva `allow-recursion`**

A  utilização  da  diretiva  `allow-recursion  {  clientes;  };}`,  faz  com  que  o  servidor  ao  receber  uma consulta recursiva de uma rede externa, responda com o conteúdo do cache ou com os endereços dos servidores, onde a resposta à consulta pode ser obtida. Em alguns casos, essa resposta chega a 500 bytes, tendo potencial para ser abusada para negação de serviço, pois poderia gerar uma amplificação de até 10 vezes.



*Criação de ACL no BIND*

O bind tem uma funcionalidade que é a criação de acls, uma forma de deixar seus arquivos de configuração mais limpos e organizados.



Exemplo de Criação de ACL:

```
acl rede_dexter {
	127.0.0.1; 172.16.100.0/24;
};
allow-recursion { rede_dexter; };
allow-query { rede_dexter; }; 
```































