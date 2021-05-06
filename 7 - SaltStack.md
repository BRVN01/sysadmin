[toc]



# Salt Stack

Salt (às vezes chamado de SaltStack) é um software de código aberto baseado em Python para automação de TI orientada a eventos, execução remota de tarefas e gerenciamento de configuração (Semelhante ao Ansible). 

As duas peças principais do Salt são: Salt Master e Salt Minion. O master é a central de gerenciamento, todos os minions se conectam ao master para receber instruções. 

A partir do master, voce pode rodar comandos e aplicar configurações para muitos minions. O Salt foi construido em cima de dois grandes conceitos, são eles: execução remota e gerenciamento de configuração.

Na execução remota de sistemas, Salt aproveita o Python para realizar tarefas complexas. O sistema de gerenciamento de configuração no Salt, denominado States, se baseia na base de execução remota para criar configurações repetíveis e executáveis para os minions.



Link para a instalação em vários sistemas [Clique aqui](https://docs.saltstack.com/en/latest/topics/installation/index.html);

Link para a instalação usando o script oficial [Clique aqui](https://repo.saltstack.com/);



## Firewall

O Salt Master fica escutando em duas portas, 4505 e 4506 em todas as interfaces (0.0.0.0).

A porta 4505 é usada pelo Master, onde o salt minion vai estabelecer uma conexão persistente com o Master. 
Os comandos são enviados de forma assíncrona para todas as conexões por meio dessa porta, o que permite que os comandos sejam executados em um grande número de sistemas simultaneamente.

A porta 4506 também no Master, é usada pelos minions do Salt que se conectam ao server conforme necessário para enviar resultados ao master, essa porta também é usada pelo Master para solicitar arquivos e valores de dados específicos de um minion com segurança (chamados pilar do Salt). As conexões com esta porta são de 1:1 entre o master e o minion (não assíncronas).



Portanto, quando necessário, devemos liberar essa porta, tanto no firewall de host como no firewall de rede.

## Instalando o Salt

Vou rodar todos os exemplos em servidores Debian, escolhi instalar tanto o Master como Minion usando o script oficial, assim não teremos métodos diferentes para cada sistemas.



### Instalar o Master

Vamos instalar o serviço do Salt Master:

```bash
root@saltmaster:~# wget -q https://bootstrap.saltstack.com -O install_salt.sh
root@saltmaster:~# sudo sh install_salt.sh -P -M -x python3
```



### Instalar o Minion

Vamos instalar o serviço do Salt Minion:

```bash
root@saltminion:~# wget -q https://bootstrap.saltstack.com -O install_salt.sh
root@saltminion:~# sudo sh install_salt.sh -P -x python3
```



#### Vamos configurar o server no Minion

[Link oficial de como configurar o Salt Minion](https://docs.saltstack.com/en/latest/ref/configuration/minion.html#configuration-salt-minion)!

```bash
# Vamos editar o arquivo de configuração do Minion:
root@saltminion:~# vim /etc/salt/minion

# Coloque o IP do master dentro do arquivo,
# A linha 'master:' já existe, então edite ela:
master: 192.168.0.51

# Podemos configurar o ID do nosso Salt Minion também, para isso basta editar a linha:
#id:

# E deixa-lá como abaixo:
id: Minion1
# Outra forma de configurar o ID do minion é no arquivo /etc/salt/minion_id.

# Se for deixado comentado, o id será o nome do host conforme retornado pela chamada 
# python: socket.getfqdn()
# Como o salt usa ids separados, é possível executar vários minions na mesma máquina, mas com 
# ids diferentes, isso pode ser útil para clusters de computação salt.

# Pelo fato explicado acima, eu geralmente nao configuro o ID, vamos ter 2 solicitações 
# do mesmo minion com IDs diferentes.
```



Seu arquivo de configuração do Salt Minion deve estar igual a saída abaixo,

somente com 1 linha ativa (sem comentários):

```bash
# O comando abaixo remove as linhas comentadas e as linhas em branco,
# para assim, podermos ver somente as linhas ativa no arquivo de 
# configuração!
root@saltminion:~# sed '/^#/d; /^$/d' /etc/salt/minion
master: 192.168.0.51

# Agora reinicie o serviço:
root@saltminion:~# systemctl restart salt-minion

# Agora verifique se o serviço está ativo:
root@saltminion:~# systemctl status salt-minion
```



Outra forma, mais simples até, é colocar o no host do Minion o IP do Master.

```bash
root@saltminion:~# vim /etc/hosts

# Adicione a linha abaixo:
192.158.0.51 salt
```



#### Vamos configurar a interface do Master

[Link oficial de como configurar o Salt Master](https://docs.saltstack.com/en/latest/ref/configuration/master.html#configuration-salt-master)!

Dessa forma, vamos limitar a interface no qual nosso servidor vai ficar escutando requisições dos minions:

```bash
# A configuração está na linha 15:
root@saltmaster:~# vim +15 /etc/salt/master

# Agora descomente essa linha e coloque o IP do servidor (interface)
# que vai atender ficar ouvindo requisições de minions:
root@saltmaster:~# interface: 192.168.0.51

# Agora reinicie o serviço:
root@saltmaster:~# systemctl restart salt-master

# Agora verifique se o serviço está ativo:
root@saltmaster:~# systemctl status salt-master
```



## Executar como um usuário sem privilégios (não root)

Para executar o Salt com outro usuário, defina o parâmetro [`user`](https://docs.saltstack.com/en/latest/ref/configuration/master.html#std-conf_master-user) no arquivo de configuração do Master (servidor principal que executa o Salt Master).

Além disso, a propriedade e as permissões precisam ser definidas de forma que o usuário desejado possa ler e gravar nos seguintes diretórios (e em seus subdiretórios, quando aplicável):

- /etc/salt

- /var/cache/salt

- /var/log/salt

- /var/run/salt

Mais informações sobre como executar o salt como um usuário não privilegiado podem ser encontradas [aqui](https://docs.saltstack.com/en/latest/ref/configuration/nonroot.html#configuration-non-root-user).



### Chave do Master

Podemos exibir a chave privada/pública que o Master está usando, isso ajuda em ambientes com mais de um Master na rede.

```bash
root@saltmaster:~# salt-key -F master

Local Keys:
	master.pem: 7a:21:b3:8a:05:3d:23:1d:9d:81:8d:ce:9b:17:b2:dc:6a:68:a8:e5:5c:43:4d:45:a7:0c:c6:79:d2:d4:0a:b8
	master.pub: e6:f5:46:37:cc:f1:12:e7:34:25:85:dd:59:61:49:e7:a3:79:88:47:a9:d5:64:b0:a2:39:81:66:9d:2b:64:25
```



### Chave do Minion

Para verificar a chave do Minion, execute o comando abaixo (o mesmo comando pode ser executado no master para ver a chave dele):

```bash
root@saltminion:~# salt-call --local key.finger

local:    56:5e:e5:7c:1c:a2:e2:d6:c1:cb:d7:74:57:b2:9e:fa:93:3b:2b:11:2c:02:1c:d8:e6:ee:aa:f9:eb:b3:07:1c
```

Salt usa criptografia AES para todas as comunicações entre o Mestre e o Minion. Isso garante que os comandos enviados aos Minions não possam ser adulterados e que a comunicação entre o Mestre e o Minion seja autenticada por meio de chaves aceitas e confiáveis.

Antes que comandos possam ser enviados a um Minion, sua chave deve ser aceita no Mestre. 




## Aceitando a chave do Minion no servidor Salt

A seguir vemos algumas opções mais utilizadas do comando `salt-key`.

- salt-key é usado para gerenciar chaves de autenticação Salt

| Opções                     | Descrição                                                    |
| -------------------------- | ------------------------------------------------------------ |
| --version                  | Exibe a versão do Salt.                                      |
| -V, --versions-report      | Exibe as dependencias do Salt, mais o numero da versão.      |
| -h, --help                 | Exibe a ajuda.                                               |
| -u USER, --user=USER       | Especifica um usuário para rodar o salt-key.                 |
| -y, --yes                  | Responde "Yes" para todas as perguntas. Padrão: False.       |
| -l ARG, --list=ARG         | Lista as chaves publicas. <br />Os argumentos 'pre', 'un', e 'unaccepted' vão listar as chaves unaccepted/unsigned.<br /><br />'acc' ou 'accepted' vai listar as chaves accepted/signed. <br /><br />'rej' ou 'rejected' vai listar chaves rejeitadas.<br />'den' ou 'denied' vai listar as chaves negadas. Finalmente, 'all' vai listar todas as chaves. |
| -L, --list-all             | Lista todas as chaves publicas. Descontinuado: use "--list all". |
| -a ACCEPT, --accept=ACCEPT | Aceita uma chave publica especifica. <br />Use --include-rejected e --include-denied para corresponder a chaves rejeitadas e negadas, além das chaves pendentes. Globs são suportados. |
| -A, --accept-all           | Aceita todas as chaves pendentes.                            |
| -r REJECT, --reject=REJECT | Rejeita uma chave publica especifica. <br />Use --include-accepted e --include-denied para corresponder a chaves aceitadas e negadas, além das chaves pendentes. Globs são suportados. |
| -R, --reject-all           | Rejeita todas as chaves.                                     |
| -p PRINT, --print=PRINT    | Exibe uma chave especifica.                                  |
| -P, --print-all            | Exibe todas as chaves publicas.                              |
| -d DELETE, --delete=DELETE | Deleta uma chave publica especifica. Globs são suportados.   |
| -D, --delete-all           | Deleta todas as chaves publicas.                             |
| -f FINGER, --finger=FINGER | Exibe a impressão digital da chave especificada.             |
| -F, --finger-all           | Exibe a impressão digital de todas as chave.                 |



Execute o comando `salt-key --list-all` para listar as chaves conhecidas pelo Salt Master:

```bash
# Listando todos os pedidos de conexão:
root@saltmaster:~# salt-key --list-all
Accepted Keys:
Denied Keys:
Unaccepted Keys:
saltminion
Rejected Keys:
```

Este exemplo mostra que o Salt Master está ciente de 1 minion, mas nenhuma chave foi aceita. Para aceitar uma chave e permitir que os minion seja controlado pelo Mestre, use novamente o comando `salt-key`:

```bash
# Aceitar um minion:
root@saltmaster:~# salt-key -a saltminion
The following keys are going to be accepted:
Unaccepted Keys:
saltminion
Proceed? [n/Y] y
Key for minion saltminion accepted.

# Veja se o minion foi aceito:
root@saltmaster:~# salt-key --list-all
Accepted Keys:
saltminion
Denied Keys:
Unaccepted Keys:
Rejected Keys:
```



### Primeiro comando remoto

Vamos fazer um teste de ping para verificar qual minion está "vivo", para isso vamos rodar o comando `salt '*' test.ping`:

```bash
root@saltmaster:~# salt '*' test.ping
saltminion:
    True
```



Nós podemos usar `cmd.run` para rodar comandos livres no shell dos minions:

```bash
# Rodando um 'ls -lh':
root@saltmaster:~# salt '*' cmd.run 'ls -lh'
saltminion:
    total 352K
    -rw------- 1 root root 2.8K Aug 31 11:47 calc.sh
    -rw------- 1 root root  50K Aug 31 16:15 dirs.tar
    -rw-r--r-- 1 root root 294K Oct 21 09:01 install_salt.sh

# Rodando 'echo "123" '
root@saltmaster:~# salt '*' cmd.run 'echo "123"'
saltminion:
    123
```



## Módulos e Funções

O salt vem com funções prontas para que possamos executar comando remotos, essas funções são agrupadas por módulos, por exemplo, temos inumeras funções, das quais temos função para: transferencia de arquivos, ping, atualizar o sistema, instalar alguma aplicação dentre outros, cada função fica dentro de um módulo, entao quando for executar algum comando a sintaxe será `modulo.função`.

Por exemplo, o comando que fizemos acima `salt '*' test.ping`, chama o módulo **test** e dentro dele usa a função **ping**.

Para listar todas as funções de um módulo, nós usamos a função `list_function` que fica dentro do módulo `sys`, execute o comando abaixo para ver as funções do módulo `test`:

```bash
root@saltmaster:~# salt 'saltminion' sys.list_functions test
saltminion:
    - test.arg
    - test.arg_clean
    - test.arg_repr
    - test.arg_type
    - test.assertion
    - test.attr_call
    - test.collatz
    - test.conf_test
    - test.cross_test
    - test.echo
    - test.exception
    - test.false
    - test.fib
    - test.get_opts
    - test.kwarg
    - test.module_report
    - test.not_loaded
    - test.opts_pkg
    - test.outputter
    - test.ping
    - test.provider
    - test.providers
    - test.raise_exception
    - test.rand_sleep
    - test.rand_str
    - test.random_hash
    - test.retcode
    - test.sleep
    - test.stack
    - test.true
    - test.try
    - test.tty
    - test.version
    - test.versions
    - test.versions_information
    - test.versions_report

# Para ver todos os módulos podemos usar o comando abaixo:
root@saltmaster:~# salt '*' sys.list_modules
```



Para ver o menu de ajuda de uma função, seja ela qual for, use a função `doc` que fica dentro do módulo `sys`, passando para eles o `modulo.função` como argumento (esse modulo.função é o que você quer consultar o menu de ajuda).



```bash
# Vamos fazer um exemplo exibindo o menu de ajuda da função Fibonacci,
# liste seu menu de ajuda:
root@saltmaster:~# salt 'saltminion' sys.doc test.fib
test.fib:

    Return the num-th Fibonacci number, and the time it took to compute in
    seconds. Used for performance tests.

    This function is designed to have terrible performance.

    CLI Example:

        salt '*' test.fib 3
# Agora podemos ver como essa função trabaha, e tem até exemplo!

# Execute essa função 'fib' passando o valor 20:
root@saltmaster:~# salt 'saltminion' test.fib 20
saltminion:
    - 6765
    - 1.430511474609375e-06
```

Para ver todos os menus de ajuda do módulo test basta omitir a função: `salt '*' sys.doc test`



Diferença entre `salt '*'` e `salt 'saltminion'`:

- Quando usamos o simbolo do asterisco `*`, mais conhecido no linux como um dos coringas, sua funcionalidade é dar match com todo conteúdo de uma lista, por exemplo, quando usamos ele no salt, ele quer dizer *rode esse comando em todos os minions conhecidos*.
- Outra alternativa é especificar o minion em que vamos executar o comando ou criar um grupo e rodar o comando para esse grupo.



## Executando funções localmente com Salt-call

O comando `salt-call` é usado para executar funções de módulo localmente no salt minion que está no servidor, ou seja, que está no mesmo host que o salt master, enquanto o comando `salt` executa comando remotos, em minions que estão em outros hosts.

Dessa forma, tudo que formos rodar nos minions podemos rodar somente no minion que está no master, basta usar o comando `salt-call`. Apesar de parecer não muito importante ter o salt-call, ter ele é como ter endereços de loopback, sua nobre função fica ofuscada perto de sua grande utilidade.

Esse comando foi concebido para que se possa executar comandos em um minion sem a necessidade de um master, dessa forma, ao invés de enviar comandos remotamente para os minions, pode ser que você esteja na cli de um dos minios e necessite executar algum comando, com `salt-call` isso será possível.



## Grains

O Salt vem com uma interface para obter informações sobre sistemas subjacentes (nossos minions). Isso é chamado de **grains interface** (interface de grãos). Os grains são coletados do sistema operacional, como:  nome de dominio, endereço IP, Kernel, Tipo de sistemas operacional, memoria, e muitos outras propriedades que um sistema pode oferecer.

A interface de grãos `é disponibilizada para os módulos e componentes do Salt para que os comandos corretos do salt minion estejam automaticamente disponíveis` nos sistemas corretos.

Os dados de Grain são relativamente estáticos, embora caso as informações do sistema sejam alteradas (por exemplo, se as configurações de rede forem alteradas) ou se um novo valor for atribuído a uma grain personalizada, os dados de grain serão atualizados.

Os minions têm um grain chamado `os_family`, que descreve a família de sistemas operacionais aos quais um minion pertence. Por exemplo, as máquinas Ubuntu são membros da `os_family Debian`, para Fedora ou CentOS, o minion seria membro da RedHat. Veja como os grains podem ser recuperados na linha de comando:

```bash
root@saltmaster:~# salt '*' grains.item os_family
saltminion:
    ----------
    os_family:
        Debian
```

Para saber ver o tipo de sistema que nossos minions estão executando, podemos usar o grain `os` ou `osfinger`, segue exemplos:

```bash
root@saltmaster:~# salt '*' grains.item os
saltminion:
    ----------
    os:
        Debian

root@saltmaster:~# salt '*' grains.item osfinger
saltminion:
    ----------
    osfinger:
        Debian-9
```



Tendo essa informação, podemos efetuar comandos apenas em tipos específicos de sistema, por exemplo, para executar um ping apenas em Sistemas Debian, ficaria assim:

```bash
root@saltmaster:~# salt --grain 'os_family:Debian' test.ping

# Troque Debian pelo tipo de sistemas desejado!
```

Podemos chamar `--grain` na linha de comando ou apenas `-G`, como abaixo:

```bash
root@saltmaster:~# salt -G 'os_family:Debian' test.ping
saltminion:
    True
```



Para **ver toda a informação** de um minion que é armazenada pelo Grain, rode o comando abaixo:

```bash
root@saltmaster:~# salt '*' grains.items
```

Nós também podemos definir nossos Grain personalizado, como abaixo:

```bash
# Criando uma entrada no Grain:
root@saltmaster:~# salt '*' grains.setval teste funcionou

# Listando nossa entrada:
root@saltmaster:~# salt '*' grains.item teste
saltminion:
    ----------
    teste:
        funcionou

# Podemos usar JSON para criar uma estrutura de dados
# mais complexa e organizada!
```

Para deletar usamos a função `grains.delval VARIAVEL`:

```bash
# Apagando a informação do Grain:
root@saltmaster:~# salt '*' grains.delval teste destructive=True
saltminion:
    ----------
    changes:
        ----------
        teste:
            None
    comment:
    result:
        True

# Consultando para ver se a informação existe:
root@saltmaster:~# salt '*' grains.item teste
saltminion:
    ----------
    teste:
```

Os Grains personalizados são armazenados nos minions no caminho `/etc/salt/grains`!



## Pillar

Os dados do pilar são semelhantes aos grãos, exceto que podem ser definidos de forma mais dinâmica, é um armazenamento seguro de dados. Um pilar é usado para gerar e armazena dados altamente confidenciais específicos para um minion em particular, como chaves criptográficas e senhas. Ele armazena dados em um par chave/valor e os dados são gerenciados de maneira semelhante à Árvore de Estado do Salt.



### Combinação composta 

Com a correspondência composta, somos capazes de especificar correspondências muito mais granulares e complexas de nossos minions. Uma correspondência composta pode ter a seguinte aparência:

```bash
root@saltmaster:~# salt -C '* and G@os:Debian and not L@minion1' test.ping
saltminion:
    True
    
# O exemplo abaixo remove 1 minion da lista de execução:
root@saltmaster:~# salt -C '* and G@os:Debian and not saltminion' test.ping
No minions matched the target. No command was sent, no jid was assigned.
ERROR: No return received

# Nesse caso deu erro porque removemos da lista de execução o único
# minion que temos!
```

Após ativado o salt minion no servidor master, vamos rodar de novamente o comando acima. Eu desativei para algumas documentações aqui, mas ele deve estar sempre ativado para que se possa rodar o comando `salt-call` localmente no master.

```bash
root@saltmaster:~# salt -C '* and G@os:Debian and not saltminion' test.ping
saltmaster:
    True
```

Perceba que ele executou em todos os minions `*`, exceto o que nós pedimos para excluir `and not saltminion`.

Vamos dissecar um pouco mais a string enviada pelo comando salt:

```
'* and G@os:Debian and not L@yourminion,theirminion'
```

Basicamente, o matcher composto combina uma série de strings de correspondência usando os operadores booleanos `and`, `or`, `and not` entre as strings de correspondência. Ele também usa uma letra e o símbolo `@` para especificar o tipo de correspondência para cada seção.

Por padrão, as strings de correspondência são correspondidas usando a correspondência glob padrão. Assim, a parte com o `*` especifica todos os minios. 
Quanto ao resto dos tipos de correspondência, aqui está uma [tabela](http://docs.saltstack.com/en/latest/topics/targeting/compound.html), diretamente da documentação oficial do Salt , que enumera os possíveis tipos de correspondência e sua letra associada:

| Letra | Tipo de partida     | Exemplo                                  |
| ----- | ------------------- | ---------------------------------------- |
| G     | Grains glob         | G@os:Ubuntu                              |
| E     | ID de minion PCRE   | E@web\d+\.(dev\|qa\|prod)\.loc           |
| P     | Grains PCRE         | P@os:(RedHat\|Fedora\|CentOS)            |
| L     | Lista de minions    | L@minion1.example.com,minion3.domain.com |
| I     | Pillar Glob         | I@pdata:foobar                           |
| S     | Endereço de Rede/IP | S@192.168.1.0/24 or S@192.168.1.100      |
| R     | Range Cluster       | R@%foo.bar                               |



Vamos lá:

```
* and G@os:Debian and not saltminion

* = Selecione todo os minions da nossa lista (que foram aceitos)

and = E

G@os:Debian = Desses minions, pegue aqueles que tem o SO Debian

and not saltminion = Exclua saltminion da lista

Agora colocando toda a explicação numa única linha:
	Selecione todo os minions da nossa lista (que foram aceitos),
	e desses minions, pegue aqueles que tem o SO Debian, e exclua 	
	saltminion da lista.
```



## Adicionando usuário

Vamos ver como funciona o módulo/função para criar um usuário:

```bash
root@saltmaster:~# salt '*' sys.doc user.add
user.add:

    Add a user to the minion

    name
        Username LOGIN to add

    uid
        User ID of the new account

    gid
        Name or ID of the primary group of the new account

    groups
        List of supplementary groups of the new account

    home
        Home directory of the new account

    shell
        Login shell of the new account

    unique
        If not True, the user account can have a non-unique UID

    system
        Create a system account

    fullname
        GECOS field for the full name

    roomnumber
        GECOS field for the room number

    workphone
        GECOS field for the work phone

    homephone
        GECOS field for the home phone

    other
        GECOS field for other information

    createhome
        Create the user's home directory

    loginclass
        Login class for the new account (OpenBSD)

    nologinit
        Do not add the user to the lastlog and faillog databases

    root
        Directory to chroot into

    usergroup
        Create and add the user to a new primary group of the same name

    CLI Example:

        salt '*' user.add name <uid> <gid> <groups> <home> <shell>

# Podemos ver que somente o argumento nome é obrigatório!
```



Agora vamos criar um usuário chamado marcos:

```bash
root@saltmaster:~# salt 'saltminion' user.add marcos
saltminion:
    True

# Agora vamos pegar informações dele no sistema:
root@saltmaster:~# salt 'saltminion' user.info marcos
saltminion:
    ----------
    fullname:
    gid:
        1001
    groups:
        - marcos
    home:
        /home/marcos
    homephone:
    name:
        marcos
    other:
    passwd:
        x
    roomnumber:
    shell:
        /bin/bash
    uid:
        1001
    workphone:

# Vamos definir uma senha para nosso usuário:
root@saltmaster:~/openssl# salt '*' shadow.set_password marcos "$(openssl passwd -6 123)"

# Caso voce rode "openssl passwd -6 123" e de erro, você precisa atualizar o pacote do openssl ou compilar um pacote mais recente caso o seu seja o mais atual do seu repositorio.
# Esse comando vai criar um hash SHA256/SHA512, que é usado pelo linux atualmente.
```



## Instalando pacotes

Como o gerenciamento de pacotes num sistema é muito importante e uma das tarefas mais comuns em ambientes Linux/Unix, o Salt possui um módulo chamado **pkg**, ele possui várias funções para se trabalhar com pacotes.

```bash
# Vamos ver as funções disponível para o módulo pkg:
root@SaltMaster:~\# salt 'slave1' sys.list_functions pkg
slave1:
    - pkg.add_repo_key
    - pkg.autoremove
    - pkg.available_version
    - pkg.del_repo
    - pkg.del_repo_key
    - pkg.expand_repo_def
    - pkg.file_dict
    - pkg.file_list
    - pkg.get_repo
    - pkg.get_repo_keys
    - pkg.get_selections
    - pkg.hold
    - pkg.info_installed
    - pkg.install
    - pkg.latest_version
    - pkg.list_downloaded
    - pkg.list_pkgs
    - pkg.list_repo_pkgs
    - pkg.list_repos
    - pkg.list_upgrades
    - pkg.mod_repo
    - pkg.normalize_name
    - pkg.owner
    - pkg.parse_arch
    - pkg.purge
    - pkg.refresh_db
    - pkg.remove
    - pkg.services_need_restart
    - pkg.set_selections
    - pkg.show
    - pkg.unhold
    - pkg.upgrade
    - pkg.upgrade_available
    - pkg.version
    - pkg.version_cmp

# Para ver "help" da função install, use o comando abaixo:
salt 'slave1' sys.doc pkg.install

# Vamos instalar o unscd:
root@SaltMaster:~# salt '*' pkg.install unscd
slave1:
    ----------
    unscd:
        ----------
        new:
            0.53-1build4
        old:
slave2:
    ----------
    unscd:
        ----------
        new:
            0.53-1build4
        old:
```



Para ver os pacotes instalados, podemos usar a função **list_pkgs**.

```bash
# vou minimizar a saida por ser muito grande:
root@SaltMaster:~\# salt 'slave1' pkg.list_pkgs
slave1:
    ----------
    accountsservice:
        0.6.55-0ubuntu12~20.04.4
    adduser:
        3.118ubuntu2
    alsa-topology-conf:
        1.2.2-1
    alsa-ucm-conf:
        1.2.2-1ubuntu0.6
    amd64-microcode:
        3.20191218.1ubuntu1
    apparmor:
        2.13.3-7ubuntu5.1
    apport:
        2.20.11-0ubuntu27.16
    apport-symptoms:
        0.23
    apt:
        2.0.5
    apt-utils:
        2.0.5
```



Removendo, Atualizando, Consultando e marcando pacotes.

```bash
# Para remover um pacote e deixar os arquivos de configuração, usamos: 
root@SaltMaster:~\# salt 'slave2' pkg.remove unscd

# Para remover um pacote e apagar também os arquivos de configuração, usamos: 
root@SaltMaster:~\# salt 'slave2' pkg.purge

# Para ver informações sobre os pacotes instalados:
root@SaltMaster:~\# salt 'slave2' pkg.info_installed

# Similar a apt-cache, retorna a ultima versão disponível de um pacote, consultando os repositórios:
root@SaltMaster:~\# salt 'slave2' pkg.latest_version tmux

# Para listar todos os pacotes disponíveis para atualização, use:
root@SaltMaster:~\# salt 'slave2' pkg.list_upgrades

# Verificar se existe uma atualização para um pacote:
root@SaltMaster:~\# salt 'slave2' pkg.upgrade_available <pacote_nome>

# Lista os pacotes atualmente instalados, mostrando a versão:
root@SaltMaster:~\# salt 'slave2' pkg.list_pkgs

## Marcar um pacote como hold (impedir que esse pacote seja atualizado).
# No Linux:
root@slave2:~\# sudo apt-mark hold <pacote_nome>
alsa-ucm-conf set on hold.

# No SaltStack:
root@SaltMaster:~\# salt 'slave2' pkg.hold <pacote_nome>

## Desmarcar um pacote como hold (impedir que esse pacote seja atualizado).
# No Linux:
root@slave2:~\# sudo apt-mark unhold <pacote_nome>

# No SaltStack:
root@SaltMaster:~\# salt 'slave2' pkg.unhold <pacote_nome>

# Similar a apt-get update, Atualiza a base de dados do APT:
root@SaltMaster:~\# salt '*' pkg.refresh_db

# Similar a apt-get upgrade:
root@SaltMaster:~\# salt '*' pkg.upgrade
```



## Gerenciando serviços

Para gerenciar serviços, inclusive baseado no Systemd, usamos o módulos **service**.

```bash
# Vamos ver se o serviço está rodando:
root@SaltMaster:~\# salt 'slave2' service.status sshd
slave2:
    True

# Verificar se está ativo no boot:
root@SaltMaster:~\# salt 'slave2' service.enabled sshd
slave2:
    True

# Para pegar uma lista com todos os serviços, use:
root@SaltMaster:~\# salt 'slave2' service.get_all

# Para pegar uma lista com todos os serviços desabilitados, use:
root@SaltMaster:~\# salt 'slave2' service.get_disabled

# Funciona como o systemd no quisito (start, stop, reload, restart).
```

Pag 42