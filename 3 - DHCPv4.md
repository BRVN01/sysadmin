## **2 - SERVIDOR DHCPv4**

O DHCP (Dynamic Host Configuration Protocol ou Protocolo Dinâmico de Configuração de Host) fornece uma estrutura que passar informações de configuração para hosts em uma rede TCP/IP. O DHCP é uma ferramenta extremamente útil para poupar o tempo dos administradores de Rede.

A aplicação quie roda em cima do protocolo DHCP é mantido pela ISC (Internet System Consortium), eles também mantém o BIND e NTP.



### Funcionamento do DHCP

O DHCP é construído em um modelo cliente-servidor, em que os hosts designados do servidor DHCP alocam endereços de rede e entregam parâmetros de configuração para hosts configurados dinamicamente (clientes).

O DHCP consiste em dois componentes: um protocolo para fornecer parâmetros de configuração específicos de host de um servidor DHCP para um host (serviço DHCP) e um mecanismo para alocação de endereços de rede para hosts (Leasing).

#### Origem de leasing

Quando o cliente inicializa (ou quer ingressar em uma rede), ele começa um processo de quatro etapas (DHCPDISCOVER, DHCPOFFER, DHCPREQUEST e DHCPACK).

Um cliente inicia o processo de busca do servidor DHCP com uma mensagem de broadcast (DHCPDISCOVER) com seu próprio endereço MAC para descobrir os servidores DHCPv4 disponíveis.



- 1. **Descoberta do DHCP (DHCPDISCOVER)**

     A mensagem de broadcast *DHCPDISCOVER* encontra os servidores DHCPv4 na rede. Como o cliente não tem informações válidas de IPv4 durante a inicialização, ele usa endereços de broadcast de Camada 2 e Camada 3 (FF:FF:FF:FF:FF:FF  e 255.255.255.255) para se comunicar com o servidor.



- 2. **Pacote de DHCP Offer (DHCPOFFER)**

     Quando o servidor DHCPv4 recebe uma mensagem *DHCPDISCOVER*, reserva o endereço IPv4 disponível para alugar para o cliente. O servidor também cria uma entrada *ARP* que consiste no endereço *MAC* do cliente solicitante e o endereço IPv4 alugado do cliente. O servidor DHCPv4 envia mensagem de vinculação *DHCPOFFER* ao cliente solicitante. A mensagem de *DHCPOFFER* é enviada como unicast, usando o endereço MAC de Camada 2 do servidor como o endereço origem e o endereço MAC de Camada 2 do cliente como destino.

     

- 3. **Solicitação de DHCP (DHCPREQUEST)**

     Quando o cliente receber o *DHCPOFFER* do servidor, enviará uma mensagem *DHCPREQUEST*. Esta mensagem é usada para geração e renovação do aluguel. Quando usado para geração de aluguel, *DHCPREQUEST* atua como um aviso de aceitação para o servidor selecionado e uma recusa implícita a todos os outros servidores que possam ter fornecido ao cliente uma oferta de vinculação.

     

     Muitas redes corporativas usam vários servidores DHCPv4. A mensagem *DHCPREQUEST* é enviada na forma de um broadcast para informar esse servidor DHCPv4 e todos os outros servidores DHCPv4 sobre a oferta aceita.

     

- 4. **Reconhecimento de DHCP (DHCPACK)**

     Ao receber a mensagem *DHCPREQUEST*, o servidor verifica as informações de aluguel com um ping  (ICMP) para esse endereço para garantir que ele não esteja sendo usado atualmente, cria uma nova entrada *ARP* para o aluguel do cliente e envia uma mensagem unicast *DHCPACK*. 

     A mensagem *DHCPACK* é uma cópia de *DHCPOFFER*, exceto por uma mudança no campo do tipo de mensagem. Quando o cliente recebe a mensagem DHCPACK, registra informações sobre configuração e realiza uma pesquisa ARP para o endereço atribuído. Se não houver resposta ao ARP, o cliente sabe que o endereço IPv4 é válido e começa a usá-lo.

     ![image-20200416164646084](/home/bruno/.config/Typora/typora-user-images/image-20200416164646084.png)

#### Renovação de leasing

- 1. **Solicitação de DHCP (DHCPREQUEST)**

     Antes do aluguel expirar, o cliente envia uma mensagem DHCPREQUEST diretamente para o servidor DHCPv4 que originalmente ofereceu o endereço IPv4. Se um DHCPACK não for recebido dentro de um período especificado, o cliente envia outro DHCPREQUEST, de modo que um dos outros servidores DHCPv4 possa estender o aluguel.

- 2. **Reconhecimento de DHCP (DHCPACK)**

     Ao receber a mensagem DHCPREQUEST, o servidor verifica as informações de aluguel retornando um DHCPACK.
     
     

### Mecanismo de Leasing (alocação)

O DHCP suporta três mecanismos para alocação de endereços IP, são eles:

- 1. **Alocação Automática**

     O DHCP atribui um endereço IP permanente a um cliente.
     
     
     
     A alocação dinâmica é o único dos três mecanismos que permite a reutilização automática de um endereço que não é mais necessário para o cliente ao qual foi atribuído (uso mais comum de servidor DHCP).
     A alocação dinâmica é útil para atribuir um endereço a um cliente que será conectado apenas temporariamente à rede ou para compartilhar um pool limitado de endereços IP entre um grupo de clientes que não precisa de endereços IP permanentes.

  

- 2. **Alocação Dinâmica**

     O DHCP atribui um endereço IP a um cliente por um período limitado de tempo (ou até que o cliente abandone explicitamente o endereço IP). 


     A alocação dinâmica é uma boa opção para atribuir um endereço IP a um novo cliente permanentemente conectado a uma rede, em que os endereços IP sejam escassos o suficiente para que seja importante recuperá-los quando clientes antigos forem aposentados.

  






- 3. **Alocação Manual**

     O endereço IP de um cliente é atribuído pelo administrador e o DHCP é usado para transmitir o endereço atribuído ao cliente.
     Uma rede pode usar um ou mais desses mecanismos, dependendo das políticas do administrador.
  
  
     A alocação manual permite que o DHCP seja usado para eliminar o processo propenso a erros de configuração manual de hosts com endereços IP em ambientes onde (por qualquer motivo) é desejável gerenciar.



### Instalando o serviço do DHCP

A instalação e configuração será feita no Ubuntu, mas tanto sua configuração e instalação é identica para sistemas base Debian e Ubuntu.

Instalação a aplicação do servidor DHCP:

```bash
╼ \# apt install isc-dhcp-server
```

Após isso, vamos informar ao serviço do DCHP qual a placa de rede do servidor que vai receber/atribuir os IPs da rede.

```bash
╼ \# vim /etc/default/isc-dhcp-server
# Remova a linha abaixo se não for utilizar IPv6:
INTERFACESv6=""

# Altere as variáveis conforme abaixo:
DHCPDv4_CONF=/etc/dhcp/dhcpd.conf # Arquivo de configuração para o IPv4.
DHCPDv4_PID=/var/run/dhcpd.pid # Onde ficara gravado o PID do serviço.
INTERFACESv4="enp0s8"

# Nesse caso, a segunda placa de rede do servidor é a 'enp0s8', a primeira placa fica
# dedicada ao link de Internet.

# Se for usar duas ou mais placas, pode deixar no padrão abaixo:
INTERFACES="enp0s3 enp0s8"
```

Como nosso servidor é apenas IPv4, só vou deixar as linhas que referenciam o IPv4. Vou também remover o arquivo de configuração relacionado ao IPv6.

```bash
╼ \# rm /etc/dhcp/dhcpd6.conf
```



### Configurando servidor DHCPv4

Antes de irmos para as configurações, é importante conhecer algumas opções de configuração, boa parte de exemplos está no próprio arquivo de configuração `/etc/dhcp/dhcpd.conf`.

Por isso vamos fazer backup desse aquivo.

```bash
╼ \# mv /etc/dhcp/dhcpd.conf /etc/dhcp/dhcpd.conf.bak && touch /etc/dhcp/dhcpd.conf
```



#### Arquivo `/etc/dhcp/dhcpd.conf` traduzido

```tex
# dhcpd.conf
Arquivo de configuração de amostra para o ISC dhcpd

Atenção: Se o /etc/ltsp/dhcpd.conf existir, ele será usado como arquivo de configuração em vez deste arquivo.

definições de opções comuns a todas as redes suportadas ...
# option domain-name "example.org";
# option domain-name-servers ns1.example.org, ns2.example.org;

tempo de concessão padrão 600; (default-lease-time 600; (10 minutos))
tempo máximo de locação 7200; (max-lease-time 7200; (2 horas))

O parâmetro ddns-updates-style controla se o servidor tentará ou não fazer uma atualização DNS quando "ceder um IP" for confirmada. O padrão é o comportamento dos pacotes da versão 2 ('none', pois o DHCP v2 não tinha suporte para DDNS.)
# ddns-update-style none;

Se este servidor DHCP for o servidor DHCP oficial da rede local, a diretiva autorizada deve ser descomentada.
#authoritative;

Use isso para enviar mensagens de log dhcp para um arquivo de log diferente (você também precisa hackear o syslog.conf para concluir o redirecionamento).
#log-facility local7;

Nenhum serviço será fornecido nesta sub-rede, mas declarar que ajuda o servidor DHCP a entender a topologia de rede.
#subnet 10.152.187.0 netmask 255.255.255.0 {
#}

Esta é uma declaração de sub-rede muito básica.

#subnet 10.254.239.0 netmask 255.255.255.224 {
#  range 10.254.239.10 10.254.239.20;
#  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
#}

Essa declaração permite que os clientes BOOTP obtenham endereços dinâmicos, o que realmente não recomendamos.

#subnet 10.254.239.32 netmask 255.255.255.224 {
#  range dynamic-bootp 10.254.239.40 10.254.239.60;
#  option broadcast-address 10.254.239.31;
#  option routers rtr-239-32-1.example.org;
#}

Uma configuração ligeiramente diferente para uma sub-rede interna.
#subnet 10.5.5.0 netmask 255.255.255.224 {
#  range 10.5.5.26 10.5.5.30;
#  option domain-name-servers ns1.internal.example.org;
#  option domain-name "internal.example.org";
#  option subnet-mask 255.255.255.224;
#  option routers 10.5.5.1;
#  option broadcast-address 10.5.5.31;
#  default-lease-time 600;
#  max-lease-time 7200;
#}

Os hosts que requerem opções de configuração especiais podem ser listados nas instruções do host. Se nenhum endereço for especificado, o endereço será alocado dinamicamente (se possível), mas as informações específicas do host ainda virão da declaração do host.

#host passacaglia {
#  hardware ethernet 0:0:c0:5d:bd:95;
#  filename "vmunix.passacaglia";
#  server-name "toccata.example.com";
#}

Endereços IP fixos também podem ser especificados para hosts. Esses endereços também não devem ser listados como disponíveis para atribuição dinâmica.
Os hosts para os quais foram especificados endereços IP fixos podem inicializar usando BOOTP ou DHCP. Os hosts para os quais nenhum endereço fixo é especificado podem ser inicializados apenas com DHCP, a menos que exista um intervalo de endereços na sub-rede à qual um cliente BOOTP esteja conectado e que tenha o sinalizador dynamic-bootp definido.

#host fantasia {
#  hardware ethernet 08:00:07:26:c0:a5;
#  fixed-address fantasia.example.com;
#}

Você pode declarar uma classe de clientes e, em seguida, fazer a alocação de endereços com base nisso. 
O exemplo abaixo mostra um caso em que todos os clientes de uma determinada classe obtêm endereços na sub-rede 10.17.224/24 e todos os outros clientes obtêm endereços na sub-rede 10.0.29/24.

#class "foo" {
#  match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
#}

#shared-network 224-29 {
#  subnet 10.17.224.0 netmask 255.255.255.0 {
#    option routers rtr-224.example.org;
#  }
#  subnet 10.0.29.0 netmask 255.255.255.0 {
#    option routers rtr-29.example.org;
#  }
#  pool {
#    allow members of "foo";
#    range 10.17.224.10 10.17.224.250;
#  }
#  pool {
#    deny members of "foo";
#    range 10.0.29.10 10.0.29.230;
#  }
#}
```



Opções que serão usadas no arquivo de configuração:

| Opção                    | Descrição                                                    |
| ------------------------ | ------------------------------------------------------------ |
| authoritative            | Indica que esse servidor é autoritativo.                     |
| ddns-update-style        | Controla se o servidor tentará ou não fazer uma atualização DNS quando  um IP for emprestado. DDNS (Dynamic Domain Name System) atualizará o nome do host para o novo IP fornecido no DNS. |
| db-time-format local     | Faz com que o arquivo /var/lib/dhcp/dhcpd.leases exiba os empréstimos de IP mostrando a hora do servidor. |
| default-lease-time 86400 | Tempo de empreśtimo em segundos (um dia = 86400).            |
| max-lease-time 604800    | Tempo máximo que pode usar a configuração (Uma semana).      |



Agora vamos configurar o serviço de DHCP:

```bash
╼ \# vim /etc/dhcp/dhcpd.conf
authoritative;
db-time-format local;
ddns-update-style none;
default-lease-time 86400;
max-lease-time 604800;

subnet 192.168.0.0 netmask 255.255.255.0
{
        range 192.168.0.15 192.168.0.250;
        option routers 192.168.0.11;
        option domain-name-servers 8.8.8.8, 8.8.4.4;

}


# Subnet inicia um bloco de configuração de rede
# 192.168.100.0 netmask 255.255.255.0 = Rede

# Range define o inicio e o fim dos IPs que poderão ser emprestados.
# option routers informa o IP do gateway.
# option domain-name-servers informa os servidores DNS.

# Caso queire fazer atualizações DNS, pode usar o parametro 
# 'ddns-update-style standard;'.
```

Feito isso, vamos reiniciar o serviço:

```bash
╼ \# systemctl restart isc-dhcp-server
```

Veja o status do serviço, se estiver ativo (Rodando), você provavelmente terá um servidor DHCP configurado.

```bash
╼ \# systemctl status isc-dhcp-server
● isc-dhcp-server.service - ISC DHCP IPv4 server
   Loaded: loaded (/lib/systemd/system/isc-dhcp-server.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2020-04-17 11:00:33 -03; 1s ago
     Docs: man:dhcpd(8)
 Main PID: 2603 (dhcpd)
    Tasks: 1 (limit: 1108)
   CGroup: /system.slice/isc-dhcp-server.service
           └─2603 dhcpd -user dhcpd -group dhcpd -f -4 -pf /run/dhcp-server/dhcpd.pid -cf /etc/dhcp/dhcpd.conf

Apr 17 11:00:33 ub dhcpd[2603]: Sending on   LPF/enp0s8/08:00:27:c9:bf:8f/192.168.0.0/24
Apr 17 11:00:33 ub dhcpd[2603]: 
Apr 17 11:00:33 ub dhcpd[2603]: No subnet declaration for enp0s3 (10.0.2.15).
Apr 17 11:00:33 ub dhcpd[2603]: ** Ignoring requests on enp0s3.  If this is not what
Apr 17 11:00:33 ub dhcpd[2603]:    you want, please write a subnet declaration
Apr 17 11:00:33 ub dhcpd[2603]:    in your dhcpd.conf file for the network segment
Apr 17 11:00:33 ub dhcpd[2603]:    to which interface enp0s3 is attached. **
Apr 17 11:00:33 ub dhcpd[2603]: 
Apr 17 11:00:33 ub dhcpd[2603]: Sending on   Socket/fallback/fallback-net
Apr 17 11:00:33 ub dhcpd[2603]: Server starting service.

# Tive um erro apresentado, informando que não foi declarada uma sub-rede 
# para a interface ENP0S3 e que por isso, o servidor DHCP vai ignorar requisições
# nessa interface (não quero receber requisições nessa interface e nem devemos...).
```



### Verificando o cliente

Assim que o host client ligou, ele ja pegou um IP.

Teste feito no cliente:

```bash
root@ub:~\# ip address show enp0s3
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:ef:e4:6e brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.16/24 brd 192.168.0.255 scope global enp0s3
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:feef:e46e/64 scope link 
       valid_lft forever preferred_lft forever
```



Agora vamos verificar o arquivo de leasing para ver o IP que foi emprestado.

Teste feito no servidor DHCP:

```bash
╼ \# cat /var/lib/dhcp/dhcpd.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.3.5

# authoring-byte-order entry is generated, DO NOT DELETE
authoring-byte-order little-endian;

server-duid "\000\001\000\001&,s\001\010\000'\311\277\217";

lease 192.168.0.15 {
  starts epoch 1587132148; # Fri Apr 17 11:02:28 2020
  ends epoch 1587218548; # Sat Apr 18 11:02:28 2020
  cltt epoch 1587132148; # Fri Apr 17 11:02:28 2020
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:ef:e4:6e;
  uid "\377\3424?>\000\002\000\000\253\021\361\015'C\303\000\204\232";
  client-hostname "ub";
}

# Por algum motivo o IP acima foi emprestado e logo depois outro IP foi emprestado:

lease 192.168.0.16 {
  starts epoch 1587132149; # Fri Apr 17 11:02:29 2020
  ends epoch 1587218549; # Sat Apr 18 11:02:29 2020
  cltt epoch 1587132149; # Fri Apr 17 11:02:29 2020
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:ef:e4:6e;
  client-hostname "ub";
}
```



Na maquina cliente, solicite uma renovação de endereço IP:

```bash
# Libera a concessão atual (IP emprestado) e para o cliente DHCP em execução.
root@ub:~\# dhclient -r

# Peça um novo endereço IP ao servido DHCP:
root@ub:~\# dhclient -v enp0s3
Internet Systems Consortium DHCP Client 4.3.5
Copyright 2004-2016 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/enp0s3/08:00:27:ef:e4:6e
Sending on   LPF/enp0s3/08:00:27:ef:e4:6e
Sending on   Socket/fallback
DHCPREQUEST of 192.168.0.16 on enp0s3 to 255.255.255.255 port 67 (xid=0x309ec2dd)
DHCPACK of 192.168.0.16 from 192.168.0.11
RTNETLINK answers: File exists
bound to 192.168.0.16 -- renewal in 41516 seconds.
```



Agora verifique no servidor novamente:

```bash
╼ \# cat /var/lib/dhcp/dhcpd.leases
# The format of this file is documented in the dhcpd.leases(5) manual page.
# This lease file was written by isc-dhcp-4.3.5

# authoring-byte-order entry is generated, DO NOT DELETE
authoring-byte-order little-endian;

server-duid "\000\001\000\001&,s\001\010\000'\311\277\217";

lease 192.168.0.15 {
  starts epoch 1587132148; # Fri Apr 17 11:02:28 2020
  ends epoch 1587218548; # Sat Apr 18 11:02:28 2020
  cltt epoch 1587132148; # Fri Apr 17 11:02:28 2020
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:ef:e4:6e;
  uid "\377\3424?>\000\002\000\000\253\021\361\015'C\303\000\204\232";
  client-hostname "ub";
}
lease 192.168.0.16 {
  starts epoch 1587132149; # Fri Apr 17 11:02:29 2020
  ends epoch 1587218549; # Sat Apr 18 11:02:29 2020
  cltt epoch 1587132149; # Fri Apr 17 11:02:29 2020
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:ef:e4:6e;
  client-hostname "ub";
}

# Repare abaixo que o mesmo IP (192.168.0.16) foi emprestado novamente.
lease 192.168.0.16 {
  starts epoch 1587134037; # Fri Apr 17 11:33:57 2020
  ends epoch 1587220437; # Sat Apr 18 11:33:57 2020
  cltt epoch 1587134037; # Fri Apr 17 11:33:57 2020
  binding state active;
  next binding state free;
  rewind binding state free;
  hardware ethernet 08:00:27:ef:e4:6e;
  client-hostname "ub";
}
```



### Setando IP fixo com base no MAC

Vamos atribuir sempre o mesmo IP a um endereço MAC específico:

```bash
╼ \# vim /etc/dhcp/dhcpd.conf
# Adicione a linha abaixo:
Host cliente
{
        hardware ethernet 08:00:27:ef:e4:6e;
        fixed-address 192.168.100.5;
}


#################### Como o arquivo irá ficar ####################
authoritative;
db-time-format local;
ddns-update-style none;
default-lease-time 86400;
max-lease-time 604800;

subnet 192.168.0.0 netmask 255.255.255.0
{
        range 192.168.0.15 192.168.0.250;
        option routers 192.168.0.11;
        option domain-name-servers 8.8.8.8, 8.8.4.4;

	Host cliente # Cliente é um nome, pode escolher qualquer nome.
	
	# Voce pode colocar um IP fixo em hosts que não estão dentro do range de 
	# atribuição, por exemplo o IP 192.168.0.5 que não será atribuido a nenhum host
	# na rede.
	{
        hardware ethernet 08:00:27:ef:e4:6e;
        fixed-address 192.168.0.5;
	}

}
##################################################################


# Reinicie o serviço:
╼ \# systemctl restart isc-dhcp-server

# No cliente, Libera a concessão atual e peça um novo endereço IP:

╼ \# dhclient -r
╼ \# dhclient -v enp0s3
Internet Systems Consortium DHCP Client 4.3.5
Copyright 2004-2016 Internet Systems Consortium.
All rights reserved.
For info, please visit https://www.isc.org/software/dhcp/

Listening on LPF/enp0s3/08:00:27:ef:e4:6e
Sending on   LPF/enp0s3/08:00:27:ef:e4:6e
Sending on   Socket/fallback
DHCPREQUEST of 192.168.0.5 on enp0s3 to 255.255.255.255 port 67 (xid=0x28f88d71)
DHCPACK of 192.168.0.5 from 192.168.0.11
RTNETLINK answers: File exists # Como eu tive que fazer isso para poder logar com SSH
# ele fala que já existe.
bound to 192.168.0.5 -- renewal in 36598 seconds.
```
