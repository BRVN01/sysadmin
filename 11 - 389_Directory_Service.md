[toc]



# Introdução



kinit

klist

### Instalação

```bash
[domain/in.registro.br]

id_provider = ipa
ipa_server = _srv_, idm.in.registro.br, idm-bkp.in.registro.br
ipa_domain = in.registro.br
ipa_hostname = tuxaua.in.registro.br
krb5_realm = IN.NIC.BR
auth_provider = ipa
chpass_provider = ipa
access_provider = ipa
cache_credentials = True
ldap_tls_cacert = /etc/ipa/ca.crt
krb5_store_password_if_offline = True
[sssd]
services = nss, pam, ssh, sudo

domains = in.registro.br
[nss]
homedir_substring = /home

[pam]

[sudo]

[autofs]

[ssh]

[pac]

[ifp]

[secrets]

[session_recording]
```



```bash
# Serviços para habilitar no boot:
sudo systemctl enable dirsrv@idm.service 
sudo systemctl enable httpd
sudo systemctl enable pki-tomcatd.target
sudo systemctl enable dirsrv.target
sudo systemctl enable ipa.service
sudo systemctl enable pki-tomcatd@pki-tomcat.service
sudo systemctl enable dirsrv@LOCALBRUNODOMAIN-COM.service
sudo systemctl enable kadmin.service
sudo systemctl enable krb5kdc.service
```





ipa user-add bruno --first=bruno --last=silva --email=bruno@localbrunodomain.com --password

ipa-client-install --server idm.localbrunodomain.com --domain localbrunodomain.com

ipa host-del servercups.localbrunodomain.com

ipa host-show servercups

Mudando o shell

ipa user-mod bruno --shell=/bin/bash

ipa user-mod bruno --rename="dias"

ipa user-mod dias --first="dias"

# Fontes

https://www.howtoforge.com/tutorial/install-and-configure-freeipa-server-on-centos-8/

https://www.techsupportpk.com/2020/04/how-to-set-up-389-directory-server-centos-rhel-8.html

https://directory.fedoraproject.org/docs/389ds/legacy/install-guide.html

https://computingforgeeks.com/how-to-install-and-configure-freeipa-server-on-rhel-centos-8/

https://computingforgeeks.com/how-to-install-and-configure-freeipa-server-on-rhel-centos-8/

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/5/html/configuring_identity_management/installing_the_ipa_client_on_linux

https://access.redhat.com/products/identity-management#server

https://www.freeipa.org/page/Build

https://access.redhat.com/products/identity-management#getstarted

https://ubuntu.com/server/docs/service-sssd

https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/linux_domain_identity_authentication_and_policy_guide/activating_and_deactivating_user_accounts