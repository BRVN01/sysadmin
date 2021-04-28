# Sumário

[toc]





## **Introdução**

O NFS foi desenvolvido pela Sun Microsystem com o propósito de compartilhar diretórios na rede local. Seu funcionamento é cliente/servidor, tendo pacotes separados para cada um, um conjunto de pacotes para o Servidor e outro para o Cliente.



Vamos a uma rápida introdução dos pacotes:

| Pacotes           | Distro      | Descrição                                     |
| ----------------- | ----------- | --------------------------------------------- |
| nfs-common        | Debian like | Utilitário para o Cliente NFS.                |
| nfs-kernel-server | Debian like | Pacote para o Servidor NFS.                   |
| nfs-utils         | RedHat like | Utilitário para o Cliente NFS e Servidor NFS. |
|                   | FreeBSD     |                                               |
|                   |             |                                               |
|                   |             |                                               |



## NFS sobre TCP e UDP



showmount -e = mostra as montagens locais.

rpcinfo -p = mostra um status do serviço.



```
$ sudo rpcinfo -p
   program vers proto   port  service
    100000    4   tcp    111  portmapper
    100000    3   tcp    111  portmapper
    100000    2   tcp    111  portmapper
    100000    4   udp    111  portmapper
    100000    3   udp    111  portmapper
    100000    2   udp    111  portmapper
    100005    1   udp  38132  mountd
    100005    1   tcp  37493  mountd
    100005    2   udp  46421  mountd
    100005    2   tcp  37803  mountd
    100005    3   udp  36482  mountd
    100005    3   tcp  37963  mountd
    100003    3   tcp   2049  nfs
    100003    4   tcp   2049  nfs
    100227    3   tcp   2049
    100003    3   udp   2049  nfs
    100227    3   udp   2049
    100021    1   udp  55676  nlockmgr
    100021    3   udp  55676  nlockmgr
    100021    4   udp  55676  nlockmgr
    100021    1   tcp  37471  nlockmgr
    100021    3   tcp  37471  nlockmgr
    100021    4   tcp  37471  nlockmgr
```



Antigo portmap, agora chama rpcbind (roda na porta 111), informa o cliente qual porta deve usar para se comunicar com o NFS ou MOUNT (conhecido como rpc.mountd).





mountd (rpc.mountd) roda em portas aleatorias, ele que faz a montagem dos diretorios remotos



confs em /etc/default{nfs-common,nfs-kernel-server}

no redhat fica em /etc/sysconfig/nfs



Ao mudar a porta default do NFSD, o rpcbind (portmap) entra em ação para saber qual porta está ativa, tanto do NFSD como do MOUTND, mas a comunicação é feita via NFSv3.

Com a porta default do NFS, a comunicação é feita via NFSv4, e não existe a necessidade de usar rpcbind (portmap). O rpcbind é usado apenas na versão 2 ou 3.





Exportar temporariamente 

exportfs IP:folder -o options

exportfs 192.168.0.0/24:/var/log -o ro,sync,no_subtree_check





Remote procedure calls
Quando a Sun desenvolveu as primeiras versões do NFS na década de 1980, eles perceberam que muitos dos problemas relacionados à rede que precisavam ser resolvidos para o NFS se aplicavam a outros serviços baseados em rede também. Eles desenvolveram uma estrutura mais geral para chamadas de procedimento remoto conhecido como RPC ou SunRPC, e construíram o NFS em cima disso. Este trabalho abriu a porta para aplicativos de todos os tipos para executar procedimentos em sistemas remotos como se estivessem sendo executados localmente.

O sistema RPC da Sun era primitivo e um tanto hackeado; sistemas muito melhores existem hoje para preencher essa necessidade. 1 No entanto, o NFS ainda depende de RPCs do tipo Sun para grande parte de sua funcionalidade. As operações que leem e gravam arquivos, montam sistemas de arquivos, acessam metadados de arquivos e verificam permissões de arquivos são implementadas como RPCs.



Protocolos de transporte
O NFS versão 2 originalmente usava UDP porque era o que tinha o melhor desempenho em LANs e computadores da década de 1980. Embora o NFS faça sua própria remontagem de sequência de pacotes e verificação de erros, UDP e NFS não possuem os algoritmos de controle de congestionamento que são essenciais para um bom desempenho em uma grande rede IP.

Para remediar esses problemas (e outros), o NFS migrou para uma opção de UDP ou TCP na versão 3 e para o TCP apenas na versão 4. 2 A opção TCP foi explorada pela primeira vez como uma forma de ajudar o NFS a trabalhar por meio de roteadores e pela Internet . Com o tempo, a maioria dos motivos originais para preferir o UDP ao TCP evaporaram na luz quente de CPUs rápidas, memória barata e redes de alta velocidade.



Estado
Um cliente deve montar explicitamente um sistema de arquivos NFS antes de usá-lo, assim como um cliente deve montar um sistema de arquivos armazenado em um disco local. No entanto, as versões 2 e 3 do NFS não têm estado e o servidor não controla quais clientes montaram cada sistema de arquivos. Em vez disso, o servidor simplesmente revela um “cookie” secreto na conclusão de uma negociação de montagem bem-sucedida. O cookie identifica o diretório montado para o servidor NFS e, assim, abre uma maneira para o cliente acessar seu conteúdo. Os cookies persistem entre as reinicializações do servidor, portanto, uma falha não deixa o cliente em uma confusão irrecuperável. O cliente pode simplesmente esperar até que o servidor esteja disponível novamente e reenviar a solicitação.
O NFSv4, por outro lado, é um protocolo com monitoração de estado: tanto o cliente quanto o servidor mantêm informações sobre arquivos abertos e bloqueios. Quando o servidor falha, os clientes auxiliam no processo de recuperação, enviando ao servidor suas informações de estado anterior à falha. Um servidor de retorno espera por um período de carência predefinido para que os clientes anteriores relatem suas informações de estado antes de permitir novas operações e bloqueios. O gerenciamento de cookies de V2 e V3 não existe mais no NFSv4.



Filesystem exports
Os servidores NFS mantêm uma lista de diretórios (chamados de “exportações” ou “compartilhamentos”) que disponibilizam aos clientes na rede. Por definição, todos os servidores exportam pelo menos um diretório. Os clientes podem então montar essas exportações e adicioná-las aos seus arquivos fstab.

Em V2 e V3, cada exportação é tratada como uma entidade independente que é exportada separadamente. Na especificação V4, um servidor exporta um único pseudo-sistema de arquivos hierárquico que incorpora todos os seus diretórios exportados. Essencialmente, o pseudo-sistema de arquivos é o próprio namespace do sistema de arquivos do servidor esqueletizado para remover qualquer coisa que não seja exportada.

Por exemplo, considere a seguinte lista de diretórios, com os diretórios a serem exportados em negrito:

**/www/domain1**
**/www/domain2**
/www/domain3
**/var/logs/httpd**
/var/spool



No NFS versão 3, cada diretório exportado deve ser configurado separadamente. Os sistemas cliente devem executar três solicitações de montagem diferentes para obter acesso a todas as exportações do servidor.

No NFS versão 4, entretanto, o pseudo-sistema de arquivos conecta as partes desconectadas da estrutura de diretório para criar uma única visualização para clientes NFS. Em vez de solicitar uma montagem separada para cada um de **/www/domain1**, **/www/domain2** e **/var/logs/httpd**, o cliente pode simplesmente montar todo o diretório pseudo-raiz do servidor e navegar na hierarquia (**mount /**).

Os diretórios que não são exportados, **/www/domain3** e **/var/spool**, não aparecem durante a navegação. Além disso, os arquivos individuais contidos em **/**, **/var**, **/www** e **/var/logs** não são visíveis para o cliente porque a parte do pseudo-sistema de arquivos da hierarquia inclui apenas diretórios. Assim, a visão do cliente do sistema de arquivos exportado NFSv4 é:

/
├─ var
│ └─ logs
│   └─ httpd
└─ www
├─ domain1
└─ domain2

O servidor especifica a raiz dos sistemas de arquivos exportados em um arquivo de configuração conhecido como arquivo de exportação, geralmente mantido em **/etc**. Os clientes NFSv4 puros não podem examinar a lista de montagens em um servidor remoto. Em vez disso, eles simplesmente montam a pseudo-raiz e, em seguida, todas as exportações disponíveis tornam-se acessíveis por meio desse ponto de montagem.

Essa é a história de acordo com as especificações RFC. Na prática, a situação é um tanto confusa. A implementação do Solaris está em conformidade com esta especificação. O Linux fez uma tentativa indiferente de oferecer suporte ao pseudo-sistema de arquivos no código NFSv4 anterior, mas posteriormente o revisou para suportar o esquema de forma mais completa; a versão de hoje parece respeitar a intenção da RFC. O FreeBSD não implementa o pseudo-sistema de arquivos conforme descrito pela RFC. A semântica de exportação do FreeBSD é essencialmente a mesma da versão 3; todos os subdiretórios em uma exportação estão disponíveis para os clientes.



Bloqueio de arquivo
O bloqueio de arquivo (conforme implementado pelas chamadas de sistemas flock, lockf ou fcntl) tem sido um ponto sensível nos sistemas UNIX há muito tempo. Em sistemas de arquivos locais, sabe-se que não funciona perfeitamente. No contexto do NFS, o terreno é ainda mais instável. Por design, as primeiras versões dos servidores NFS não têm estado: eles não têm ideia de quais máquinas estão usando um determinado arquivo. No entanto, para implementar o bloqueio, são necessárias informações de estado. O que fazer?

A resposta tradicional era implementar o bloqueio de arquivos separadamente do NFS. Na maioria dos sistemas, dois daemons distintos, lockd e statd, tentaram fazer isso. Infelizmente, a tarefa era difícil por uma variedade de razões sutis, e o bloqueio de arquivos NFS sob lockd e statd geralmente não é confiável.
O NFSv4 removeu a necessidade de lockd e statd dobrando o locking (e, portanto, statefulness e tudo o que isso implica) no protocolo central. Essa alteração introduz uma complexidade significativa, mas evita muitos dos problemas relacionados das versões anteriores do NFS.

Infelizmente, lockds e statds separados ainda são necessários para oferecer suporte aos clientes V2 e V3 se seu site os tiver. Nossos sistemas de exemplo são fornecidos com as versões anteriores do NFS habilitadas, portanto, os daemons separados ainda são executados por padrão.



Preocupações com segurança

De muitas maneiras, NFS V2 e V3 são filhos de cartaz de tudo o que está ou sempre esteve errado com a segurança do UNIX e Linux. O protocolo foi originalmente projetado basicamente sem nenhuma preocupação com a segurança, e a conveniência tem seu preço. O NFSv4 abordou as questões de segurança de versões anteriores exigindo suporte para serviços de segurança fortes e estabelecendo melhores meios de identificação do usuário.

Todas as versões do protocolo NFS devem ser independentes do mecanismo de segurança, e a maioria dos servidores oferece suporte a vários “tipos” de segurança. Alguns dos sabores comuns incluem:

• AUTH_NONE - sem autenticação
• AUTH_SYS - controle de acesso de usuário e grupo no estilo UNIX
• RPCSEC_GSS - um sabor mais forte que permite esquemas de segurança flexíveis

Historicamente, a maioria dos sites usava a autenticação AUTH_SYS, que depende dos identificadores de usuário e grupo UNIX. Nesse esquema, o cliente simplesmente envia o UID e o GID local do usuário que está solicitando acesso ao servidor. O servidor compara os valores com aqueles de seu próprio arquivo **/etc/passwd** e determina se o usuário deve ter acesso. Assim, se os usuários mary e bob compartilharem o mesmo UID em dois clientes diferentes, eles terão acesso aos arquivos um do outro. Além disso, os usuários que possuem acesso root em um sistema podem usar o su para qualquer UID que desejarem; o servidor então dará a eles acesso aos arquivos do usuário correspondente.

Impor a consistência do arquivo passwd entre os sistemas é essencial em ambientes que usam AUTH_SYS. Mas mesmo isso é apenas uma folha de figueira de segurança; qualquer host desonesto (ou Deus me livre, máquina Windows) pode “autenticar” seus usuários da maneira que quiser e, assim, subverter a segurança NFS.

Para evitar esses problemas, a maioria dos sites pode usar um mecanismo de segurança mais robusto, como Kerberos, em combinação com a camada NFS RPCSEC_GSS. Essa configuração requer que o cliente e o servidor participem de um domínio Kerberos. O reino Kerberos autentica clientes centralmente, evitando os problemas de auto-identificação descritos acima. O Kerberos também pode fornecer criptografia forte e integridade garantida para arquivos transferidos pela rede. Todos os sistemas NFS versão 4 em conformidade com o protocolo devem implementar RPCSEC_GSS, mas é opcional na versão 3.

O NFS versão 4 requer TCP como protocolo de transporte e se comunica pela porta 2049. Como a V4 não depende de nenhuma outra porta, abrir o acesso por meio de um firewall é tão simples quanto abrir a porta TCP 2049. Como ocorre com todas as configurações de lista de acesso, certifique-se de especificar endereços de origem e destino, além da porta. Se o seu site não precisa fornecer serviços NFS para hosts na Internet, bloqueie o acesso por meio do firewall ou use um filtro de pacote local.

O serviço de arquivos em redes de longa distância com NFSv2 e V3 não é recomendado devido ao longo histórico de bugs nos protocolos RPC e à falta de mecanismos de segurança fortes.
Os administradores de servidores NFS versão 3 devem bloquear o acesso às portas TCP e UDP 2049 e também à porta portmap, 111.

Dadas as inúmeras e óbvias deficiências da segurança AUTH_SYS, recomendamos fortemente a descontinuação de todo o uso do NFSv3. Se você tiver sistemas operacionais antigos que não podem ser atualizados para compatibilidade com NFSv4, pelo menos implemente filtros de pacotes para restringir a conectividade de rede.



NFS do lado do servidor
Diz-se que um servidor NFS “exporta” um diretório quando o torna disponível para uso por outras máquinas. As exportações são apresentadas aos clientes NFSv4 como uma única hierarquia de sistema de arquivos por meio do pseudo-sistema de arquivos.

No NFS versão 3, o processo usado pelos clientes para montar um sistema de arquivos é separado do processo usado para acessar os arquivos. As operações usam protocolos separados e as solicitações são atendidas por diferentes daemons: mountd para descoberta e solicitações de montagem e nfsd para serviço de arquivo real. Em alguns sistemas, esses daemons são chamados de rpc.nfsd e rpc.mountd como um lembrete de que eles dependem do RPC como um mecanismo subjacente (e, portanto, requerem que o daemon do portmap esteja em execução). Neste capítulo, omitimos o prefixo rpc para facilitar a leitura.

O NFSv4 não usa o mountd de forma alguma. Se você absolutamente deve executar clientes antigos que suportam apenas NFSv3, o mountd deve permanecer ativo.

Tanto o mountd quanto o nfsd devem iniciar quando o sistema inicializa e ambos devem permanecer em execução enquanto o sistema estiver ativo. Tanto o Linux quanto o FreeBSD executam automaticamente os daemons quando você habilita o serviço NFS.

O NFS usa um único banco de dados de controle de acesso que informa quais sistemas de arquivos devem ser exportados e quais clientes podem montá-los. A cópia operativa deste banco de dados é normalmente mantida em um arquivo chamado xtab e também em tabelas internas ao kernel. xtab é um arquivo binário mantido para uso pelo daemon do servidor.

Manter um arquivo binário manualmente não é muito divertido, então a maioria dos sistemas assume que você prefere manter um arquivo de texto, geralmente **/etc/exports**, que enumera os diretórios exportados do sistema e suas configurações de acesso. O sistema pode então consultar esse arquivo de texto no momento da inicialização para construir automaticamente o arquivo xtab.
**/etc/exports** é a lista canônica, legível por humanos, de diretórios exportados. Seu conteúdo é lido por exportfs -a no Linux e com uma simples reinicialização do servidor NFS no FreeBSD. Portanto, ao editar **/etc/exports**, execute exportfs -a para ativar suas mudanças no Linux ou execute `service nfsd restart` no FreeBSD. Se você atende clientes V3 do FreeBSD, reinicie o mountd também (`service mountd reload`).

O NFS lida com a camada lógica do sistema de arquivos. Qualquer diretório pode ser exportado; não precisa ser um ponto de montagem ou a raiz de um sistema de arquivos físico. No entanto, para segurança, o NFS presta atenção aos limites entre os sistemas de arquivos e exige que cada dispositivo seja exportado separadamente. Por exemplo, em uma máquina que configurou **/chimchim/users** como uma partição separada, você pode exportar **/chimchim** sem exportar implicitamente **/chimchim/users**.

Os clientes geralmente têm permissão para montar subdiretórios de um diretório exportado se desejarem, embora o protocolo não exija esse recurso. Por exemplo, se um servidor exporta **/chimchim/users**, um cliente pode montar apenas **/chimchim/users/joe** e ignorar o resto do diretório de usuários.



Exportações de Linux
No Linux, o arquivo de exportação consiste em uma lista de diretórios exportados na coluna mais à esquerda seguida pelos hosts que têm permissão para acessá-los e opções associadas à direita. Um espaço em branco separa o sistema de arquivos da lista de clientes e cada cliente é seguido imediatamente por uma lista de opções separadas por vírgulas entre parênteses.

As linhas podem ser continuadas com uma barra invertida. Por exemplo, a linha

`/home *.users.admin.com(rw) 172.17.0.0/24(ro)`

permite que **/home** seja montado como leitura/gravação por todas as máquinas no domínio `users.admin.com` e somente leitura por todas as máquinas na rede classe C **172.17.0.0/24**. Se um sistema no domínio **users.admin.com** residir na rede **172.17.0.0/24**, esse cliente irá ter acesso somente leitura. **A regra com menos privilégios vence**.

Os sistemas de arquivos listados no arquivo de exportação sem um conjunto específico de hosts geralmente podem ser montados por todas as máquinas. Essa é uma falha de segurança considerável.
O mesmo buraco de segurança considerável pode ser criado acidentalmente por um espaço extraviado. Por exemplo, a linha

`/home *.users.admin.com (rw)`
permite qualquer acesso de leitura/gravação de host, exceto para `*.users.admin.com`, que tem permissão somente leitura, o padrão. Opa.

Infelizmente, não há como listar várias especificações do cliente para um único conjunto de opções. Você deve repetir as opções para todos os clientes desejados. A Tabela 21.1 lista os tipos de especificações do cliente que podem aparecer no arquivo de exportação.

|               |             |                                                              |
| ------------- | ----------- | ------------------------------------------------------------ |
| Hostname      | hostname    | Individual hosts                                             |
| Netgroup      | @groupname  | Grupos de rede NIS (raramente usados)                        |
| Wild cards    | * and ?     | FQDNs a com curingas; *não corresponderá a um ponto          |
| IPv4 networks | ipaddr/mask | Especificações de estilo CIDR (por exemplo, 128.138.92.128/25) |
| IPv6 networks | ipaddr/mask | Endereços IPv6 com notação CIDR (2001:db8::/32)              |



nfsd: servir arquivos
Depois que a solicitação de montagem de um cliente é validada, o cliente pode solicitar várias operações do sistema de arquivos. Essas solicitações são tratadas no lado do servidor pelo nfsd, o daemon de operações do NFS. O nfsd não precisa ser executado em uma máquina cliente NFS, a menos que o cliente exporte seus próprios sistemas de arquivos.

nfsd não tem arquivo de configuração; opções são passadas como argumentos de linha de comando. Você inicia e interrompe o nfsd com os mecanismos de serviço padrão do seu sistema, ou seja, **systemctl** em sistemas Linux executando o **systemd** e o comando **service** no FreeBSD.
Em sistemas Linux, execute `systemctl restart nfs-config.service nfs-server.service` para habilitar mudanças de configuração nfsd. No FreeBSD, use `service nfsd restart` e` service mountd restart`.
A opção **-N** para **nfsd** desativa a versão especificada do NFS. Por exemplo, para desativar as versões 2 e 3, adicione -N 2 -N 3 ao arquivo apropriado e a opção especificada na Tabela 21.4 e reinicie o serviço. Essa é uma boa ideia se você tiver certeza de que não precisa oferecer suporte a clientes mais antigos.

**nfsd** recebe um argumento numérico que especifica quantos threads de servidor bifurcar. Selecionar o número apropriado de nfsds é importante e, infelizmente, é uma espécie de arte negra. Se o número for muito baixo ou muito alto, o desempenho do NFS pode ser prejudicado.

O número ideal de threads nfsd depende do sistema operacional e do hardware em uso. Se você notar que o ps geralmente mostra o nfsds no estado D (ininterrupto hibernação) e que alguma CPU ociosa está disponível, considere aumentar o número de threads. Se você encontrar a média de carga (conforme relatado pelo tempo de atividade) aumentando à medida que adiciona nfsds, você foi longe demais; afaste-se um pouco desse limite.

Execute o **nfsstat** regularmente para verificar os problemas de desempenho que podem estar associados ao número de threads do nfsd. Veja a página 811 para mais detalhes sobre o nfsstat.

No FreeBSD, as opções **--minthreads** e **--maxthreads** para nfsd permitem o gerenciamento automático do número de threads dentro dos limites especificados. No FreeBSD, veja man rc.conf e consulte as opções prefixadas com **nfs_** para mais configurações do servidor NFS.