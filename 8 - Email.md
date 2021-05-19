[toc]



# Configurando servidor de e-mail

O **sendmail** era o servidor de email mais utilizado, mas por conta da evolução, começaram a surgir muitas falhas de segurança, muitas delas foram corrigidas, mas outras nem tanto, sua complexidade também é muito prejudicial, é um ótima ferramenta, mas é muito complexa.

Outra ferramenta muito boa é o **Exim**, ele foi um concorrente muito forte do **sendmail**, e agora podemos dizer que é um concorrente do **Postfix**, o **exim** não é muito conhecido, mas é tão bom quanto.

Uma outra aplicação que se mostrou promissor também foi o **Qmail**, acontece que ele esta morrendo lentamente, manter um servidor desse é muito trabalhoso por causa do funcionamento, como ele possui uma licença, para manter, você deve instalar patchs, muito desses patchs possuem patch para correções deles mesmos, isso torna a administração muito trabalhosa.

Por fim temos um dos mais utilizados, o **Postfix**. É uma das ferramentas mais rápidas e com maior simplicidade na configuração, também é bastante conhecido e possui muita documentação.



## Instalando o Postfix

```bash
sudo apt install postfix
```

Alguns pacotes adicionais:

```bash
sudo apt install postfix-ldap
# Permite obter a lista de logins (user and password) do servidor ldap.

sudo apt install postfix-mysql postfix-pgsql
# Permite armazenar a lista de usuários e senha no mysql ou postgree-sql
```



Assim que instalarmos o pacote principal o postfix, será exibido algumas perguntas, dessa forma o postfix consegue fazer uma configuração inicial.

**1° - Tipo de servidor de email**

Será perguntado o tipo de configuração geral para o servidor, normalmente temos 5 opções:

| Opção                   | Descrição                                                    |
| ----------------------- | ------------------------------------------------------------ |
| No configuration        | Não vai ocorrer a configuração inicial.                      |
| Internet Site           | Esse tipo é para um servidor de email que vai enviar e receber mensagens diretamente, é o tipo mais comum. |
| Internet with smarthost | Recebe mensagens normalmente, mas é outra máquina que vai enviar as mensagens, opção conhecida como `relayhost`. |
| Satellite System        | A opção mais limitada que temos para o funcionamento do servidor de email. Envia email através de outra máquina e não recebe mensagens de emails, muito útil para monitorar servidores na rede interna que precisam enviar notificações por email. |
| Local Only              | Usada apenas em redes de terminais leves, os emails são trocados entre os usuário na mesma máquina, é como um email, mas fica somente nesse máquina, não sai para a Internet. |



**2° - Redirecionamento de destinatário**

Como o usuário root nunca é usado, apenas para manutenção, os emails enviados para ele devem ser redirecionados para outros usuários, essa etapada pede que você informe o nome do usuário que vai receber os emails do root.



**3° - Nome de domínio**

Aqui voce deve inserie apenas o domínio que você possui. Se for teste, pode apenas usar o nome da máquina.



**4° - Destinos aceitos pelo nosso servidor**

Aqui devemos inserir os destinos que nosso servidor aceitará, essas mensagens são para destinos dentro do próprio servidor, assim ele sabe quando deve ser entregue para ele mesmo e quando deva ser enviado externamente.

Aqui devemos colocar: *hostname, domain_name, localhost.localdomain, localhost*. Veja o exemplo abaixo:

```bash
mx2, sysnetbr.eng.br, localhost.localdomain, localhost
```

Dessa forma, emails recebeidos pelo servidor com destinatário *fulano@mx2*, *fulano@sysnetbr.eng.br*, *fulano@localhost* serão enviados para o *fulano*.



**5° - Synchronous updates**

Permite ativar ou desativar a otimizações no envio de mensagens, fazendo com que o emails sejam enviados conforme são recebidos e em ordem. Isso aumenta um pouco a confiabilidade, mas diminuiu o desempenho, use com sabedoria, para um servidor pequeno ou sem muitos usuários, podemos deixar desativado.



Depois disso o postfix já está configurado e meio que pronto para uso.





```bash
# See /usr/share/postfix/main.cf.dist for a commented, more complete version


# Debian specific:  Specifying a file name will cause the first
# line of that file to be used as the name.  The Debian default
# is /etc/mailname.
#myorigin = /etc/mailname

smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no

# appending .domain is the MUA's job.
append_dot_mydomain = no

# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h

readme_directory = no

# TLS parameters
smtpd_tls_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
smtpd_tls_key_file=/etc/ssl/private/ssl-cert-snakeoil.key
smtpd_use_tls=yes
smtpd_tls_session_cache_database = btree:${data_directory}/smtpd_scache
smtp_tls_session_cache_database = btree:${data_directory}/smtp_scache

# See /usr/share/doc/postfix/TLS_README.gz in the postfix-doc package for
# information on enabling SSL in the smtp client.

smtpd_relay_restrictions = permit_mynetworks permit_sasl_authenticated defer_unauth_destination
myhostname = vicus.in.registro.br
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
myorigin = /etc/mailname
mydestination = $myhostname, vicus.in.registro.br, localhost.in.registro.br, localhost
relayhost = clone.registro.br
mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = loopback-only
inet_protocols = all

```

