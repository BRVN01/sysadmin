## **1 - FIREWALL LINUX**

Existem basicamente 2 tipos de Firewalls, são eles:

- **Firewall de Rede**

  Nosso Firewall de borda, tem a função de Gerenciar todos os pacotes que entram e saem da nossa rede (FORWARD / INPUT / OUTPUT / NAT / PREROUTNG / POSROUTING);

- **Firewall de Host**

  Configurado localmente em cada servidor, gerencia todos os pacotes do host com a rede interna e/ou externa (INPUT e OUTPUT).

O firewall do iptables é desenvolvido pelo[*Netfilter Project*](http://www.netfilter.org)e está disponível como parte do Linux desde o lançamento do kernel Linux 2.4 em janeiro de 2001.

O iptables funciona a nível de pacote, sempre se baseando no endereço/porta de origem/destino e prioridade, sempre comparando as regras para saber se um pacote pode ou não passar.

- **Possui 2 políticas**:
  ACCEPT, DROP e REJECT;

- **Possui 5 tabelas**: 
  Filter, NAT, MANGLE, RAW e Security;

- **Possui 5 chains**: 
  INPUT, OUTPUT, FORWARD, PREROUTING, POSTROUTING.



Analisar as regras:

```
sudo iptables -P INPUT -j DROP
sudo iptables -P FORWARD -j DROP
sudo iptables -P OUTPUT -j ACCEPT

sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
sudo iptables -A INPUT -m conntrack --ctstate INVALID -j DROP
sudo iptables -A INPUT -p icmp --icmp-type destination-unreachable -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type echo-request -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type time-exceeded -j ACCEPT
sudo iptables -A INPUT -p icmp --icmp-type parameter-problem -j ACCEPT
sudo iptables -A INPUT -p udp -m multiport --dport 33434:33690 -j ACCEPT

```





### Tabelas

Cada regra do IPTABLES é realizada numa tabelta específica, isso ajuda a manter a organização das regras criadas, segue as regras abaixo:

| Tabela   | Descrição                                                    |
| -------- | ------------------------------------------------------------ |
| Filter   | Regra responsável por determinar tudo o que entra e sai da máquina local (firewall de host); |
| NAT      | Usado para dados que geram outra conexão, mascarar a internet, redirecionar requisições (firewall de rede); |
| Mangle   | Usado para alterações especiais de pacote, como marcação para QOS e balanceamento de link. |
| RAW      | Marcar pacote para um rastreio, configurar exceções (é a primeira tabela a ser consultada); |
| Security | Usado para regras de rede, para controle obrigatório de acesso, específica para integração com o SELinux. |



### Chains

Cada cadeia (Chain) é apenas um diretório (fazendo uma analogia), dentro da Chain será criado as regras de Firewall. O IPTABLES possuí algumas Chains default, que são INPUT, OUTPUT e FORWARD, mas você pode criar suas próprias Chains.

Por exemplo, Entrada_Funcional (regras apenas de entrada que estão estabelecidas por exemplo) ou gerencia (Rede de gerencia da empresa), dessa forma, você irá sempre saber do que se trata as regras que estão dentro da Chain.

Segue abaixo as Chains de cada Tabela (por padrão):

| Tabela   | Chains default de cada tabela                  |
| -------- | ---------------------------------------------- |
| Filter   | INPUT, OUTPUT, FORWARD                         |
| NAT      | PREROUTING, POSTROUTING, INPUT, OUTPUT         |
| Mangle   | REROUTING, POSTROUTING, INPUT, OUTPUT, FORWARD |
| RAW      | PREROUTING, OUTPUT                             |
| Security | INPUT, OUTPUT, FORWARD                         |

 

#### Descrição das Chains

- **Input** - Aqui temos os pacotes destinados ao nosso sistema Linux, após o Kernel fazer um cálculo de roteamento, sempre depois de ter feito o roteamento.
- **OUTPUT** -  É reservado para pacotes gerados pelo próprio Linux, ou seja, pacotes que saem da nossa própria máquina.
- **FORWARD** -  São os pacotes roteados pelo nosso Linux (ou seja, quando o firewall do iptables é usado para conectar mais de uma rede).
- **PREROUTING** - É usada para aplicar regras na tabela `nat` a pacotes que ainda não passaram pelo algoritmo de roteamento no kernel, a fim de determinar a interface na qual eles devem ser transmitidos. Os pacotes processados nesta chain também ainda não foram comparados com as chain INPUT ou FORWARD na tabela de filtros.
- **POSTROUTING** - É responsável pelo processamento de pacotes depois que eles passam pelo algoritmo de roteamento no kernel e estão prestes a ser transmitidos na interface física. Os pacotes processados por essa chain passaram pelos requisitos das chains OUTPUT ou FORWARD na tabela de filtros.



### Políticas e Regras

- **ACCEPT** - Aceita os pacotes; 
- **DROP** - Barra (Dropa) um pacote silenciosamente;
- **REJECT ** - Barra (Dropa) um pacote mas devolve uma mensagem de erro ao remetente;
- **MASQUERADE** - Mascara o pacote (geralmente saindo da rede);
- **LOG** - Cria uma registro de LOG.



### Opções do Iptables:

| Opção                | Descrição                                                    |
| -------------------- | ------------------------------------------------------------ |
| -p (Protocol)        | (**Protocol**) Específica o protocolo                        |
| -L                   | (**list**) Lista as regras                                   |
| -t (table)           | (**Table**) Define 1 das 4 tabelas                           |
| -P (Chain)           | (**Policy**) Define a política da Chain                      |
| -S                   | (**list-rules**) Lista a política de todas as Chains ou uma em específico |
| -A                   | (**Append**) Adiciona a regra no final da lista de regras    |
| -I                   | (**Insert**) Adiciona a regra no inicio da lista de regras   |
| -d (IP)              | (**Destination**) É o destino                                |
| -s (IP)              | (**Source**) É a origem                                      |
| --line-numbers       | Adiciona um número a cada regra de cada Chain                |
| -D (Chain) (Number)  | (**Delete**) Deleta uma regra                                |
| -R (chain) (rulenum) | (**Replace**) Altera uma regra já existente                  |
| -F                   | (**Flush**) Deleta todas as regras de uma Chain              |
| -N (chain)           | (**new**) Crie uma nova Chain definida pelo usuário          |
| -n                   | (**Numeric**) Saída em forma de número de endereços e portas |
| -i (interface)       | (**in-interface**) Interface da qual um pacote foi recebido  |
| -o (interface)       | (**out-interface**) Interface da qual um pacote será enviado |
| --sport (porta)      | (**Source-port**) Porta de origem do pacote                  |
| --dport (porta)      | (**Destination-port**) Porta de destino do pacote            |
| -m (módulo)          | (**Comment**) Comentario na regra (uso: **-m comment --comment "Comentario"**) |
|                      | (**Multiport**) Podemos colocar várias portas                |
| LOG --log-prefix     | (**LOG**) Cria um sistema de log                             |



Observações sobre as opções `-i` e `-o`:

```tex
-i, --in-interface
Nome da interface em que o pacote entrou, a opção '-i' é utilizada para as Chains
em que o pacote entra, como as Chains: INPUT, FORWARD e PREROUTING.

Quando o cifrão  "!"  é usado como argumento antes do nome da interface, inverte-se o sentido. Se o nome da interface terminar com o sinal de adição "+", então qualquer interface que comece com esse nome corresponderá. 

Se a opção '-i' for omitida, qualquer interface corresponderá.

-o, --out-interface
Nome da interface em que o pacote será enviado, a opção '-o' é utilizada para as Chains
em que o pacote será enviado, como as Chains: FORWARD, OUTPUT e POSTROUTING.

Quando o cifrão  "!"  é usado como argumento antes do nome da interface, inverte-se o sentido. Se o nome da interface terminar com o sinal de adição "+", então qualquer interface que comece com esse nome corresponderá. 

Se a opção '-o' for omitida, qualquer interface corresponderá.

### Resumindo: ###

Usamos as Chains INPUT, FORWARD e PREROUTING quando estivermos usando a opção '-i';
Usamos as Chains FORWARD, OUTPUT e POSTROUTING quando estivermos usando a opção '-o'.
```



### Estrutura das definições de regras

<span style="color:#0000ff">iptables -t</span>  <span style="color:#008080">filter -A INPUT </span> <span style="color:#808000">-s 127.0.0.1 -d 127.0.0.1</span> <span style="color:#008000"> -j ACCEPT</span>

- <span style="color:#0000ff">Prefixo que "sempre será usado";</span>
- <span style="color:#008080">Onde a regra é armazenada, é o local onde o kernel irá verificar a regra (Chain);</span>
- <span style="color:#808000">A regra que será aplicada, nesse caso IP Origem e Destino, pode conter porta, protocolo e etc;</span>

- <span style="color:#008000">A ação que será aplicada, se o pacote será aceito, dropado, rejeitado, direcionado e etc.</span>



### Gravando as regras

Como as do iptables são criadas na memória, toda vez que o host for reiniciado as regras serão perdidas, para isso temos um comando para salvar as regras que estão na memória e um outro comando que ativa essas regras salvas anteriormente.

- **iptables-save**  - Salva as regras num arquivo;
- **iptables-restore** - Ativa as regras salvas.

```bash
# Salvando as regras (Backup):
╼ \# iptables-save > /backup/firewall.rules

# Fazendo o restore das regras:
╼ \# iptables-restore /backup/firewall.rules
```



### Construindo um Firewall

Para isso, é importante ter conhecimento das portas e protocolos de serviço de rede, entender a conexão baseada em Cliente e Servidor.



#### Arquitetura Cliente e Servidor

Arquitetura Cliente/Servidor trata-se de uma estrutura de rede entre dois sistemas finais, que **solicitam e recebem** (cliente) um serviço de um programa servidor. O programa Cliente normalmente roda em um computador (PC ou servidor) e o programa servidor, em outro, aplicações Cliente/Servidor são, por definição, **aplicações distribuídas**.

![Mitigating a SYN Flood Attack - root@opentodo#](https://opentodo.files.wordpress.com/2012/12/syn-flood.jpg?resize=418%2C305)

Esse processo da imagem acima é conhecido como Three-way Handshake, ele é usado para estabelecer comunicação entre cliente e servidor, no ultimo processo, após o cliente enviar um ACK para o servidor, ambos vão ter uma sessão aberta para poderem se comunicar, e logo após isso, o cliente pode solicitar dados do servidor que o transmitirá caso possível.

As portas usadas na comunicação variam de acordo com cada range e necessidade, segue abaixo:

| Portas      | Tipo de porta | Descrição                                                    |
| ----------- | ------------- | ------------------------------------------------------------ |
| 0-1023      | Reservada     | Portas do sistema ou portas conhecidas (atribuídas pela IANA) |
| 1024-49151  | Livre         | Portas de usuário ou portas registradas (atribuídas pela IANA) |
| 49152-65535 | "Livre"       | Portas dinâmicas (nunca atribuídas), usadas aleatoriamente em conexão cliente/servidor |

As portas podem estar em 3 estados diferentes, são eles:

**Atribuído**: atualmente, os números de porta atribuídos estão atribuídos ao serviço indicado no registro.

**Não atribuído**: atualmente, os números de porta não atribuídos estão disponíveis para atribuição mediante solicitação, conforme os procedimentos descritos neste documento.

**Reservado**: os números de porta reservados não estão disponíveis para atribuição regular; eles são "atribuídos à IANA" para fins especiais.



Quem cuida da atribuição de cada porta e serviço é a IANA, você pode ler a [RFC 6335](https://tools.ietf.org/html/rfc6335) para entender melhor esse processo ou a [RFC1340](https://tools.ietf.org/html/rfc1340) onde explica melhor a atribuição de portas.

O Linux tem um arquivo contendo as principais portas e serviços vinculadas a essas portas, o arquivo fica localizado em `/etc/services`.



#### Definindo as políticas das Chains da tabela Filter

O jeito mais fácil de proteger sua rede ou host é bloqueando tudo e permitindo somente o necessário, para isso vamos alterar as políticas das Chains da tabela Filter, que por padrão é aceitar tudo, nesse caso, vamos negar tudo e passaremos a liberar somente o necessário.

```bash
# Defininfo a politica da Chain INPUT como DROP:
╼ \# iptables -t filter -P INPUT DROP

# Defininfo a politica da Chain OUTPUT como DROP:
╼ \# iptables -t filter -P OUTPUT DROP

# Defininfo a politica da Chain FORWARD como DROP:
╼ \# iptables -t filter -P FORWARD DROP
```



#### Liberando acesso a interface Loopback (127.0.0.1)

Quando falamos de endereço loopback, o primeiro IP que vem a mente é `127.0.0.1`, porém, esse é o endereço IP mais utilizado, a [RCF 3330](https://tools.ietf.org/html/rfc3330) explica melhor os Blocos de endereços reservados.

IANA.IETF. Disponível em <https://tools.ietf.org/html/rfc3330>. Acesso em 12 Abril de 2020.

> 127.0.0.0/8 - This block is assigned for use as the Internet host loopback address.  A datagram sent by a higher level protocol to an address anywhere within this block should loop back inside the host.
>    This is ordinarily implemented using only 127.0.0.1/32 for loopback, but no addresses within this block should ever appear on any network anywhere [RFC1700, page 5].

> 127.0.0.0/8 - Este bloco é atribuído para uso como endereço de host loopback da Internet. Um datagrama enviado por um protocolo de nível superior para um endereço em qualquer local dentro desse bloco deve ser retornado no host.
> Isso geralmente é implementado usando apenas 127.0.0.1/32 para loopback, mas nenhum endereço dentro desse bloco deve aparecer em nenhuma rede em qualquer lugar [RFC1700, página 5].

Dessa forma, para que no futuro possa utilizar esse bloco sem preocupações, vamos liberar o acesso a toda rede `128.0.0.0/8`.

```bash
# Liberando os pacotes que entram:
╼ \# iptables -t filter -A INPUT -s 127.0.0.0/8 -d 127.0.0.0/8 -j ACCEPT

# Liberando os pacotes que saem:
╼ \# iptables -t filter -A OUTPUT -s 127.0.0.0/8 -d 127.0.0.0/8 -j ACCEPT

```



#### Resumo das regras (para script (Firewall de HOST))

```bash
# Defininfo a politica da Chain INPUT como DROP:
iptables -t filter -P INPUT DROP

# Defininfo a politica da Chain OUTPUT como DROP:
iptables -t filter -P OUTPUT DROP

# Defininfo a politica da Chain FORWARD como DROP:
iptables -t filter -P FORWARD DROP

# Liberando os pacotes que entram:
iptables -t filter -A INPUT -s 127.0.0.0/8 -j ACCEPT -m comment --comment "Libera pacotes na rede loopback (127.0.0.0/8) que estiver entrando."

# Liberando os pacotes que saem:
iptables -t filter -A OUTPUT -s 127.0.0.0/8 -j ACCEPT -m comment --comment "Libera pacotes na rede loopback (127.0.0.0/8) que estiver saindo."

# Liberando consulta ICMP do tipo 8 (echo request (pergunta)) entrando no meu host:
iptables -t filter -A OUTPUT -p icmp --icmp-type 8 -s IP -d 0/0 -j ACCEPT -m comment --comment "Liberando pacotes ICMP do tipo 8 (echo request (pergunta)) que estiver saindo."

# Liberando consulta ICMP do tipo 0 (echo reply (resposta)) entrando no meu host:
iptables -t filter -A INPUT -p icmp --icmp-type 0 -s 0/0 -d IP -j ACCEPT -m comment --comment "Liberando pacotes ICMP do tipo 0 (echo reply (resposta)) que estiver entrando."

# Desa forma meu host consegue efetuar perguntas icmp e obter respostas, mas
# meu host não vai responder a requisições de outros hosts.


# Liberando consulta DNS:
iptables -t filter -A OUTPUT -p udp -s IP -d 0/0 --dport 53 -j ACCEPT -m comment --comment "Liberando consulta DNS".

# Liberando resposta DNS:
iptables -t filter -A INPUT -p udp -s 0/0 --sport 53 -d IP -j ACCEPT -m comment --comment "Liberando resposta DNS."

## Após liberar o DNS, não vamos conseguir acessar a sites, mesmo tendo o IP,
## o acesso a sites é feito geralmenta na porta 80(http) e 443(https),
## portanto, precisamos liberar essas portas.

# Liberando requisição HTTP:
iptables -t filter -A OUTPUT -p tcp -s IP -d 0/0 --dport 80 -j ACCEPT -m comment --comment "Liberando requisição HTTP."

# Liberando resposta HTTP:
iptables -t filter -A INPUT -p tcp -s 0/0 --sport 80 -d IP -j ACCEPT -m comment --comment "Liberando resposta HTTP."

# Liberando requisição HTTPS:
iptables -t filter -A OUTPUT -p tcp -s IP -d 0/0 --dport 443 -j ACCEPT -m comment --comment "Liberando requisição HTTPS."

# Liberando resposta HTTPS:
iptables -t filter -A INPUT -p tcp -s 0/0 --sport 443 -d IP -j ACCEPT -m comment --comment "Liberando resposta HTTPS."


## Regra mais inteligente (HTTP e HTTPS) - Usando módulo MULTIPORT

# Liberando requisição HTTP e HTTPS:
iptables -t filter -A OUTPUT -p tcp -m multiport -s IP -d 0/0 --dport 80,443 -j ACCEPT -m comment --comment "Liberando requisição HTTP e HTTPS."

# Liberando resposta HTTP e HTTPS:
iptables -t filter -A INPUT -p tcp -m multiport -s 0/0 --sport 80,443 -d IP -j ACCEPT -m comment --comment "Liberando resposta HTTP e HTTPS."
```



### Alterando uma regra ja criada

Para alterar uma regra que já está em produção (Funcionando), ao invés de apagar e criar de novo, nós podemos editar essa regra.

```bash
╼ \# iptables -nL --line-numbers
Chain INPUT (policy DROP)
num  target     prot opt source               destination         
1    ACCEPT     tcp  --  10.0.2.2             10.0.2.15            tcp dpt:22
2    ACCEPT     udp  --  10.0.2.2             10.0.2.15            udp dpt:22

Chain FORWARD (policy DROP)
num  target     prot opt source               destination         

Chain OUTPUT (policy DROP)
num  target     prot opt source               destination         
1    ACCEPT     tcp  --  10.0.2.15            10.0.2.2             tcp spt:22
2    ACCEPT     udp  --  10.0.2.15            10.0.2.2             udp spt:22
```

Após listar as regras, usei a opção `--line-numbers` para saber a linha de cada regra, dessa forma poderemos alterar ou até mesmo apagar as regras a partir do número da regra de cada Chain.

Alterando uma regra de SSH, vamos colocar um comentario na regra ja existente:

```bash
# Mudando as regras da Chain INPUT:
iptables -t filter -R INPUT 1 -p tcp -s 10.0.2.2 -d 10.0.2.15 --dport 22 -j ACCEPT -m comment --comment "SSH da VBox entrando - TCP"

iptables -t filter -R INPUT 2 -p udp -s 10.0.2.2 -d 10.0.2.15 --dport 22 -j ACCEPT -m comment --comment "SSH da VBox entrando - UDP"

# Mudando as regras da Chain OUTPUT:
iptables -t filter -R OUTPUT 1 -p tcp -s 10.0.2.15 --sport 22 -d 10.0.2.2 -j ACCEPT -m comment --comment "SSH da VBox saindo - TCP"

iptables -t filter -R OUTPUT 2 -p udp -s 10.0.2.15 --sport 22 -d 10.0.2.2 -j ACCEPT -m comment --comment "SSH da VBox saindo - UDP"
```

Listando novamente:

```bash
╼ \# iptables -nL --line-numbers
Chain INPUT (policy DROP)
num  target     prot opt source               destination         
1    ACCEPT     tcp  --  10.0.2.2             10.0.2.15            tcp dpt:22 /* SSH da VBox entrando - TCP */
2    ACCEPT     udp  --  10.0.2.2             10.0.2.15            udp dpt:22 /* SSH da VBox entrando - UDP */

Chain FORWARD (policy DROP)
num  target     prot opt source               destination         

Chain OUTPUT (policy DROP)
num  target     prot opt source               destination         
1    ACCEPT     tcp  --  10.0.2.15            10.0.2.2             tcp spt:22 /* SSH da VBox saindo - TCP */
2    ACCEPT     udp  --  10.0.2.15            10.0.2.2             udp spt:22 /* SSH da VBox saindo - UDP */
```



### Servidor de Internet (NAT)

Um servidor de Internet é um servidor que libera Internet para o restante da rede, normalmente, ele pode ser um proxy transparente (juntamente com um Firewall), porque além de liberar a Internet, o intuíto é bloquear alguns acessos, podemos fazer isso apenas com um proxy, mas criando um Servidor de Internet  que faz isso, algumas coisas ficam mais interessantes, como registro de Logs (presente no proxy), marcação de pacotes (apenas em roteadores não caseiros e em firewalls), detalhar qual tipo de pacote pode entrar ou não na rede, qual tipo de pacote pode sair ou não da rede, entre outros tipos de controle de acesso.



#### O que á NAT?

O NAT converte endereços IP privados em endereços IP públicos. Cada cliente pode receber um único ou um pequeno intervalo de endereços IP públicos para suportar centenas a milhares de máquinas internas.

Dessa forma, precisamos apenas de 1 IP público e podemos N hosts dentro da rede privada, apesar disso parecer muito bom e realmente trouxe alguns benefícios, ele também tem muitos problemas e mascara outros problemas como a falta de endereços IPv4.



Para a criação de um servidor de Internet, primeiro precisamos trabalhar com a tabela NAT e com a função `MASQUERADE`.

Primeiro, precisamos habilitar o roteamento entre interfaces no kernel:

```bash
╼ \# vim +28 /etc/sysctl.conf

# Encontre a linha 'net.ipv4.ip_forward' e certifique-se que ela esteja com valor 1
# (net.ipv4.ip_forward=1), geralmente está na linha 28, mas isso pode mudar.

# Exibe o valor fa variável net.ipv4.ip_forward
╼ \# sysctl -p

# Definir o roteamento no Kernel até a proxima reinicialização:
╼ \# echo 1 > /proc/sys/net/ipv4/ip_forward
```



#### Os dois tipos de NAT

- NAT de origem (SNAT)

  O NAT de origem é quando você muda o endereço de origem do pacote, ou seja, você está mudando de onde o pacote está vindo. O NAT de origem sempre é feito após o roteamento no kernel do Linux (`POSTROUTING`), pouco antes do pacote ser enviado pela interface física. Mascaramento (`MASQUERADE`) é uma forma especializada de SNAT, quando mudamos a origem.




- NAT de destino (DNAT)

  O NAT de destino é quando você altera o endereço de destino do pacote, ou seja, você está mudando o destino da conexão. O destino é sempre feito antes do roteamento no kernel. O encaminhamento de porta, o compartilhamento de carga e o proxy transparente são todas as formas de DNAT (quando voce muda o destino para ter uma análise do tráfego ou pacote).

Exemplo de mudanças de Origem:

```bash
# Para mudar a origem, usamos comando '-j SNAT', e a opção '--to-source' pode especificar 
# um endereço IP, um intervalo de endereços IP (com ou sem porta, com ou sem intervalo de
# portas) apenas para os protocolos UDP e TCP.

## Mudando a origem para 10.0.13.44:
╼ \# iptables -t nat -A POSTROUTING -o enp0s3 -j SNAT --to 10.0.13.44

## Mudando a origem para 10.0.13.44 até 10.0.13.56:
╼ \# iptables -t nat -A POSTROUTING -o enp0s3 -j SNAT --to 10.0.13.44-10.0.13.56

## Mudando a origem para 10.0.13.44, portas de 1 até 3500
╼ \# iptables -t nat -A POSTROUTING -p tcp -o enp0s3 -j SNAT --to 10.0.13.44:1-3500
```



Exemplo de mudanças de Destino:

```bash
# Para mudar o destino, usamos comando '-j DNAT', e a opção '--to-destination' pode
# especificar um endereço IP, um intervalo de endereços IP (com ou sem porta, com ou 
# sem intervalo de portas) apenas para os protocolos UDP e TCP.

## Mudando o destino para 10.0.13.28
╼ \# iptables -t nat -A PREROUTING -i enp0s3 -j DNAT --to-destination 10.0.13.28

## Mudando o destino para 10.0.13.28, 10.0.13.29 ou 10.0.13.30
╼ \# iptables -t nat -A PREROUTING -i enp0s3 -j DNAT --to-destination 10.0.13.28-10.0.13.30

## Mudando o destino do tráfego HTTP para 10.0.13.28, port 8080.
╼ \# iptables -t nat -A PREROUTING -p tcp --dport 80 -i enp0s3 -j DNAT --to-destination 10.0.13.28:8080
```



#### Masquerading

Nas redes locais, cada máquina não precisa de um endereço IP (válido) para estar conectado a Internet. Ele pode acessar a Internet utilizando o mascaramento de IP ou IP Masquerading. É uma configuração que permite com que as máquinas sem um endereço de rede (válida), possam se conectar a outras redes com endereços válidos. Esse método deve ser usado apenas para endereços IP atribuídos dinamicamente, se o IP for estático, use o SNAT.

Exemplo de MASQUERADING:

```bash
## Mascarando tudo que vai sair pela interface ENP0S3 (link de Internet):
╼ \# iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE
```



#### Redirecionamento de porta

Uma dos recursos mais usados em servidores de Internet (não somente nele), é o redirecionamento de portas, ao receber uma consulta externa, podemos redirecionar essa consulta para dentro da rede, melhor dizendo, para um host em específico, e redirecionar requisições internas para fora da rede (Internet). Para fazer o  redirecionamento de portas usamos a table a NAT.

```bash
## Exemplo
# Redirecionando uma requisição de DNS externa para meu DNS na rede Interna:
╼ \# iptables -t nat -A PREROUTING -p tcp --dport 53 -i enp0s3 -j DNAT --to 192.168.5.7:53

# Redirecionando requisições externas para o Web Server interno:
╼ \# iptables-t nat -A PREROUTING -p tcp --dport 80 -i enp0s3 -j DNAT --to 192.168.5.8:80
╼ \# iptables -t nat -A PREROUTING -p tcp --dport 443 -i enp0s3 -j DNAT --to 192.168.5.8:443

# Redirecinando tudo da porta 80 para porta 4156
╼ \# iptables -t nat -A PREROUTING -i enp0s3 -p tcp --dport 80 -j REDIRECT --to-port 4156
```

> NOTAS
> Para uma explicação completa de como o iptables faz NAT, consulte http://www.netfilter.org/documentation/HOWTO/NAT-HOWTO.html



#### Carrier Grade NAT

Robert_Keith.a10networks. Disponível em <https://www.a10networks.com/blog/carrier-grade-nat/>. Acesso em 15 Abril de 2020.

>  A CGN (Carrier Grade NAT) foi desenvolvida para permitir que os provedores de serviços da Internet usem endereços IP públicos para dar suporte a cada vez mais clientes finais. 
>
> NAT padrão ou NAT44 traduz endereços IP privados para públicos. Uma função principal do CGN é o NAT444. NAT444 converte endereços IP privados em privados e depois em endereços IP públicos. Os ISPs que usam CGN conseguiram substituir endereços IP públicos por endereços IP privados nos dispositivos CPE do cliente. Isso permite que várias redes de clientes compartilhem um endereço IP público comum.

> O diagrama abaixo mostra três redes de clientes usando endereços IP privados na Internet, voltados para portas de rede. O recurso NAT444 (Privado, Privado, Público) do CGN permite que vários clientes finais compartilhem um único Endereço IP Público.

![Implementação CGN do NAT444, NAT privado para privado e público](https://www.a10networks.com/wp-content/uploads/private-to-private-to-public-nat.png)



#### NAT64

Robert_Keith.a10networks. Disponível em <https://www.a10networks.com/blog/carrier-grade-nat/>. Acesso em 15 Abril de 2020.

> NAT64  é uma tecnologia em que clientes somente IPv6 ainda podem acessar conteúdo herdado somente IPv4. O dispositivo NAT64 medeia entre as solicitações DNS do cliente (usando DNS64) e sintetiza uma resposta DNS IPv6, se uma não existir. Em seguida, o NAT64 fará a mediação entre o servidor da Web somente IPv4 e o cliente somente IPv6.
>
> Para obter mais informações sobre a transição do IPv6 para operadoras, consulte a especificação IETF (Internet Engineering Task Force) - NAT Incremental Carrier-Grade (CGN) para transição do IPv6 .



### Colocando as regras na inicialização do sistema

Vamos adicionar um script para rodar na inicialização do sistema, sempre que o servidor iniciar, esse script vai rodar, inicializando assim as regras do nosso firewall.

Verifique se o serviço `rc-local.service` do SystemD está rodando:

```bash
╼ \# systemctl status rc.local
```

Caso não esteja rodando, vamos criar o arquivo abaixo, caso já esteja rodando, verifique se o arquivo abaixo já exista:

```bash
# Para ativar o serviço precisamos criar o arquivo /etc/rc.local:
╼ \# > /etc/rc.local
# ou
╼ \# touch /etc/rc.local
```

Agora vamos editar o arquivo para que ele possa subir nossos script na inicialização.

Rode o comando `vim /etc/rc.local` com root para editarmos o arquivo:

```bash
#!/bin/bash

/etc/init.d/firewall.sh start
exit 0 # Tem que terminar com 0

# nesse caso, coloquei o script em ''/etc/init.d/'.

# O parâmetro 'exit  0' é um item obrigatório, ele informa que o
# script 'rc.local' foi executado com sucesso.
```

Script Firewall.sh:

```shell
#!/bin/bash

stop () {
	# Limpando as regras ta tabela Filter:
    iptables -t filter -F

    # Defininfo a politica da Chain INPUT como DROP:
    iptables -t filter -P INPUT ACCEPT

    # Defininfo a politica da Chain OUTPUT como DROP:
    iptables -t filter -P OUTPUT ACCEPT

    # Defininfo a politica da Chain FORWARD como DROP:
    iptables -t filter -P FORWARD ACCEPT
}

start () {
	# Limpando as regras ta tabela Filter:
    iptables -t filter -F

    # Defininfo a politica da Chain INPUT como DROP:
    iptables -t filter -P INPUT DROP

    # Defininfo a politica da Chain OUTPUT como DROP:
    iptables -t filter -P OUTPUT DROP

    # Defininfo a politica da Chain FORWARD como DROP:
    iptables -t filter -P FORWARD DROP

    # Liberando os pacotes que entram:
	iptables -t filter -A INPUT -s 127.0.0.0/8 -j ACCEPT -m comment --comment "Libera pacotes na rede loopback (127.0.0.0/8) que estiver entrando."

	# Liberando os pacotes que saem:
	iptables -t filter -A OUTPUT -s 127.0.0.0/8 -j ACCEPT -m comment --comment "Libera pacotes na rede loopback (127.0.0.0/8) que estiver saindo."

	# Liberando consulta ICMP do tipo 8 (echo request (pergunta)) entrando no meu host:
	iptables -t filter -A OUTPUT -p icmp --icmp-type 8 -s IP -d 0/0 -j ACCEPT -m comment --comment "Liberando pacotes ICMP do tipo 8 (echo request (pergunta)) que estiver saindo."

	# Liberando consulta ICMP do tipo 0 (echo reply (resposta)) entrando no meu host:
	iptables -t filter -A INPUT -p icmp --icmp-type 0 -s 0/0 -d IP -j ACCEPT -m comment --comment "Liberando pacotes ICMP do tipo 0 (echo reply (resposta)) que estiver entrando."

    # Liberando consulta DNS:
	iptables -t filter -A OUTPUT -p udp -s IP -d 0/0 --dport 53 -j ACCEPT -m comment --comment "Liberando consulta DNS".

	# Liberando resposta DNS:
	iptables -t filter -A INPUT -p udp -s 0/0 --sport 53 -d IP -j ACCEPT -m comment --comment "Liberando resposta DNS."

    # Liberando requisição HTTP e HTTPS:
	iptables -t filter -A OUTPUT -p tcp -m multiport -s IP -d 0/0 --dport 80,443 -j ACCEPT -m comment --comment "Liberando requisição HTTP e HTTPS."

	# Liberando resposta HTTP e HTTPS:
	iptables -t filter -A INPUT -p tcp -m multiport -s 0/0 --sport 80,443 -d IP -j ACCEPT -m comment --comment "Liberando resposta HTTP e HTTPS."

    ### (Virtual BOX) Liberando requisição SSH:
    iptables -t filter -I INPUT -p udp -s 10.0.2.2 -d 10.0.2.15 --dport 22 -j ACCEPT
    iptables -t filter -I INPUT -p tcp -s 10.0.2.2 -d 10.0.2.15 --dport 22 -j ACCEPT

    # (Virtual BOX) Liberando resposta SSH:
    iptables -t filter -I OUTPUT -p udp -s 10.0.2.15 --sport 22 -d 10.0.2.2 -j ACCEPT
    iptables -t filter -I OUTPUT -p tcp -s 10.0.2.15 --sport 22 -d 10.0.2.2 -j ACCEPT

}

case "$1" in
    start)
	    echo "Iniciando serviço..."
	    start

	    ;;
    stop)
	    echo "Parando serviço..."
        stop

	;;
    restart)
        echo "Reiniciando serviço..."
	    stop
	    start

	;;
    *)
        echo "Operação inválida"
        exit 1
        ;;
esac
```



### Criando log para as regras

Para criar um sistemas de log, precisamos usar o parametro `LOG --log-prefix '[Pacote entrando SSH]'`.

Isso é apenas um exemplo de log para pacotes entrantes na porta 22.

Configurando o Rsyslog para capturar os logs do iptables:

```bash
╼ \# vim /etc/rsyslog.d/10-iptables.conf

# Colocar o texto abaixo:
kern.warning      /var/log/iptables.log

# O iptables usa a facilidade de nivel kern.warn. Podemos encaminhar os logs do iptables
# para um arquivo específico.

# Agora reinicie o serviço:
╼ \# /etc/init.d/rsyslog restart
```

Para aplicar os log na regra, precisamos criar a regra duas vezes, uma com a ação de registrar o log e outra com a ação de aceitar ou dropar.

Forma errada de criar um sistema de log:

```sh
iptables -t filter -I OUTPUT -p tcp -s 10.0.2.15 --sport 22 -d 10.0.2.2 -j ACCEPT -m comment --comment "Liberando saida SSH (rcp)." 

iptables -t filter -A OUTPUT -p tcp -s 10.0.2.15 --sport 22 -d 10.0.2.2 -j LOG --log-prefix '[SSH TCP OUTPUT]'

# Assim que o iptables faz uma ação {ACCPET, REJECT ou DROP} ele aplica a ação e não
# testa as outras regras, por isso temos que ter cuidado com as regras, por exemplo:
iptables -t filter -A INPUT -s 192.168.0.0/24 -j DROP
iptables -t filter -A INPUT -s 192.168.0.15/24 -j ACCEPT

# A primeira regra irá descartar tudo da rede 192.168.0.0./24, mas logo abaixo eu tenho
# uma regra que permite apenas um host dessa rede, dessa forma esse host nunca irá
# se comunicar através do firewall, após aplicar o DROP, o iptables não vai verificar as
# outras regras para poder saber se pode ou não. Forma correta:
iptables -t filter -A INPUT -s 192.168.0.15/24 -j ACCEPT
iptables -t filter -A INPUT -s 192.168.0.0/24 -j DROP

# Dessa forma se for o IP 192.168.0.15/24 o firewall irá aceitar, caso não seja ele
# o pacote será descartado.

# O sistemas de logs funciona da mesma forma, portanto, a regra do log tem que estar 
# no topo da lista de regras, caso contrário, nao vai funcionar.
```

Forma correta de crias os logs:

```shell
iptables -t filter -A OUTPUT -p tcp -s 10.0.2.15 --sport 22 -d 10.0.2.2 -j LOG --log-prefix '[SSH TCP OUTPUT]'

iptables -t filter -I OUTPUT -p tcp -s 10.0.2.15 --sport 22 -d 10.0.2.2 -j ACCEPT -m comment --comment "Liberando saida SSH (rcp)." 
```



### Criando um Firewall de Rede

Já implementamos boa parte das regras de host no nosso firewall, agora precisamos implementar as regras de rede, vamos controlar o acesso de outras rede, do jeito que está, apenas implementando o `masquerading` os outros host de outras redes vão vão ter acesso pois bloqueamos tudo.

Sempre que um pacote deve trafegar entre "redes", usamos a Chain FORWARD.

```bash
# Liberando ping para a rede 192.168.0.0/24
iptables -t filter -A FORWARD -p icmp -s 192.168.0.0/24 -d 0/0 -j ACCEPT
iptables -t filter -A FORWARD -p icmp -s 0/0 -d 192.168.0.0/24 -j ACCEPT

# Liberando DNS para a rede 192.168.0.0/24
iptables -t filter -A FORWARD -p udp -s 192.168.0.0/24 -d 0/0 --dport 53 -j ACCEPT
iptables -t filter -A FORWARD -p udp -s 0/0 --sport 53 -d 192.168.0.0/24 -j ACCEPT

# Liberando HTTP e HTTPS para a rede 192.168.0.0/24
iptables -t filter -A FORWARD -p tcp -m multiport -s 192.168.0.0/24 -d 0/0 --dport 80,443 -j ACCEPT
iptables -t filter -A FORWARD -p tcp -m multiport -s 0/0 --sport 80,443 -d 192.168.0.0/24 -j ACCEPT
```



Script Firewall.sh (**Para nosso Firewall de borda** (rede)):

```bash
#!/bin/bash

stop () {
	# Limpando as regras ta tabela Filter:
    iptables -t filter -F

    # Defininfo a politica da Chain INPUT como DROP:
    iptables -t filter -P INPUT ACCEPT

    # Defininfo a politica da Chain OUTPUT como DROP:
    iptables -t filter -P OUTPUT ACCEPT

    # Defininfo a politica da Chain FORWARD como DROP:
    iptables -t filter -P FORWARD ACCEPT
}

start () {
	# Limpando as regras ta tabela Filter:
    iptables -t filter -F

    # Defininfo a politica da Chain INPUT como DROP:
    iptables -t filter -P INPUT DROP

    # Defininfo a politica da Chain OUTPUT como DROP:
    iptables -t filter -P OUTPUT DROP

    # Defininfo a politica da Chain FORWARD como DROP:
    iptables -t filter -P FORWARD DROP

    # Liberando os pacotes que entram:
	iptables -t filter -A INPUT -s 127.0.0.0/8 -j ACCEPT -m comment --comment "Libera pacotes na rede loopback (127.0.0.0/8) que estiver entrando."

	# Liberando os pacotes que saem:
	iptables -t filter -A OUTPUT -s 127.0.0.0/8 -j ACCEPT -m comment --comment "Libera pacotes na rede loopback (127.0.0.0/8) que estiver saindo."

	# Liberando consulta ICMP do tipo 8 (echo request (pergunta)) entrando no meu host:
	iptables -t filter -A OUTPUT -p icmp --icmp-type 8 -s 10.0.2.15 -d 0/0 -j LOG --log-prefix '[ICMP saindo]'
	iptables -t filter -A OUTPUT -p icmp --icmp-type 8 -s 10.0.2.15 -d 0/0 -j ACCEPT -m comment --comment "Liberando pacotes ICMP do tipo 8 (echo request (pergunta)) que estiver saindo."


	iptables -t filter -A OUTPUT -p icmp --icmp-type 8 -s 192.168.0.11 -d 192.168.0.0/24 -j LOG --log-prefix '[ICMP 192.168.0.0/24]'
        iptables -t filter -A OUTPUT -p icmp --icmp-type 8 -s 192.168.0.11 -d 192.168.0.0/24 -j ACCEPT

	# Liberando consulta ICMP do tipo 0 (echo reply (resposta)) entrando no meu host:
	iptables -t filter -A INPUT -p icmp --icmp-type 0 -s 0/0 -d 10.0.2.15 -j LOG --log-prefix '[ICMP ENTRANDO]'
	iptables -t filter -A INPUT -p icmp --icmp-type 0 -s 0/0 -d 10.0.2.15 -j ACCEPT -m comment --comment "Liberando pacotes ICMP do tipo 0 (echo reply (resposta)) que estiver entrando."

        iptables -t filter -A INPUT -p icmp --icmp-type 0 -s 192.168.0.0/24 -d 192.168.0.11 -j LOG --log-prefix '[ICMP ENTRANDO]'
        iptables -t filter -A INPUT -p icmp --icmp-type 0 -s 192.168.0.0/24 -d 192.168.0.11 -j ACCEPT


    # Liberando consulta DNS:
	iptables -t filter -A OUTPUT -p udp -s 10.0.2.15 -d 0/0 --dport 53 -j ACCEPT -m comment --comment "Liberando consulta DNS".

	# Liberando resposta DNS:
	iptables -t filter -A INPUT -p udp -s 0/0 --sport 53 -d 10.0.2.15 -j ACCEPT -m comment --comment "Liberando resposta DNS."

    # Liberando requisição HTTP e HTTPS:
	iptables -t filter -A OUTPUT -p tcp -m multiport -s 10.0.2.15 -d 0/0 --dport 80,443 -j ACCEPT -m comment --comment "Liberando requisição HTTP e HTTPS."

	# Liberando resposta HTTP e HTTPS:
	iptables -t filter -A INPUT -p tcp -m multiport -s 0/0 --sport 80,443 -d 10.0.2.15 -j ACCEPT -m comment --comment "Liberando resposta HTTP e HTTPS."

    ### (Virtual BOX) Liberando requisição SSH:
    iptables -t filter -I INPUT -p udp -s 10.0.2.2 -d 10.0.2.15 --dport 22 -j ACCEPT
    iptables -t filter -I INPUT -p tcp -s 10.0.2.2 -d 10.0.2.15 --dport 22 -j ACCEPT

    # (Virtual BOX) Liberando resposta SSH:
    iptables -t filter -I OUTPUT -p udp -s 10.0.2.15 --sport 22 -d 10.0.2.2 -j ACCEPT
    iptables -t filter -I OUTPUT -p tcp -s 10.0.2.15 --sport 22 -d 10.0.2.2 -j ACCEPT

    iptables -t filter -I OUTPUT -p tcp -o enp0s8 -d 192.168.0.0/24 --dport 22 -j ACCEPT
    iptables -t filter -I INPUT -p tcp -i enp0s8 -s 192.168.0.0/24 --sport 22 -j ACCEPT

    ######################## FORWARD #########################

    iptables -t filter -A FORWARD -p icmp -s 192.168.0.0/24 -d 0/0 -j LOG --log-prefix '[ICMP rede 192.168.0.0/24 para INTERNET]'
    iptables -t filter -A FORWARD -p icmp -s 0/0 -d 192.168.0.0/24 -j LOG --log-prefix '[ICMP INTERNET para rede 192.168.0.0/24]'

    iptables -t filter -A FORWARD -p icmp --icmp-type 8 -s 192.168.0.0/24 -d 0/0 -j ACCEPT -m comment --comment "Liberando consulta ICMP para rede 192.168.0.0/24."
    iptables -t filter -A FORWARD -p icmp --icmp-type 0 -s 0/0 -d 192.168.0.0/24 -j ACCEPT -m comment --comment "Liberando resposta ICMP para rede 192.168.0.0/24."

    iptables -t filter -A FORWARD -p udp -s 192.168.0.0/24 -d 0/0 --dport 53 -j LOG --log-prefix '[DNS rede 192.168.0.0/24 para INTERNET]'
    iptables -t filter -A FORWARD -p udp -s 0/0 --sport 53 -d 192.168.0.0/24 -j LOG --log-prefix '[DNS INTERNET para rede 192.168.0.0/24]'

    iptables -t filter -A FORWARD -p udp -s 192.168.0.0/24 -d 0/0 --dport 53 -j ACCEPT -m comment --comment "Liberando consulta DNS para rede 192.168.0.0/24."
    iptables -t filter -A FORWARD -p udp -s 0/0 --sport 53 -d 192.168.0.0/24 -j ACCEPT -m comment --comment "Liberando resposta DNS para rede 192.168.0.0/24."


    iptables -t filter -A FORWARD -p tcp -m multiport -s 192.168.0.0/24 -d 0/0 --dport 80,443 -j LOG --log-prefix '[HTTP(S) rede 192.168.0.0/24 para INTERNET]'
    iptables -t filter -A FORWARD -p tcp -m multiport -s 0/0 --sport 80,443 -d 192.168.0.0/24 -j LOG --log-prefix '[HTTP(S) INTERNET para rede 192.168.0.0/24]'

    iptables -t filter -A FORWARD -p tcp -m multiport -s 192.168.0.0/24 -d 0/0 --dport 80,443 -j ACCEPT -m comment --comment "Liberando consulta HTTP(S) para rede 192.168.0.0/24."
    iptables -t filter -A FORWARD -p tcp -m multiport -s 0/0 --sport 80,443 -d 192.168.0.0/24 -j ACCEPT -m comment --comment "Liberando resposta HTTP(S) para rede 192.168.0.0/24."


    ######################## NAT #########################

    iptables -t nat -A POSTROUTING -o enp0s3 -j MASQUERADE -m comment --comment "Mascarando os pacotes de outras redes saindo para internet."

}

case "$1" in
    start)
	    echo "Iniciando serviço..."
	    start

	    ;;
    stop)
	    echo "Parando serviço..."
        stop

	;;
    restart)
        echo "Reiniciando serviço..."
	    stop
	    start

	;;
    *)
        echo "Operação inválida"
        exit 1
        ;;
esac
```
