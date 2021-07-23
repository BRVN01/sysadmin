## Making a root Certificate

We recommend using a private certificate authority (CA).  While it can be difficult to install this CA on multiple client machines, it is (in general) more secure.

```
$ vi ca.cnf
```

Edit the `input_password` and `output_password` fields to be the password for the CA certificate.

qsNlEL50kVK0DjzJpLr83Mns

Edit the `[certificate_authority]` section to have the correct values for your country, state, etc.

Create the CA certificate:

```
$ make ca.pem
```

Then the `DER` format needed by Windows:

```
$ make ca.der
```



## Making a Server Certificate

The following steps will let you create a server certificate for use with TLS-based EAP methods, such as EAP-TLS, PEAP, and TTLS.  Follow similar steps to create an `inner-server.pem` file, for use with EAP-TLS that is tunneled inside of another TLS-based EAP method.

```
$ vi server.cnf
```

Edit the `input_password` and `output_password` fields to be the password for the server certificate.

Edit the `[server]` section to have the correct values for your country, state, etc.  Be sure that the `commonName` field here is different from the `commonName` for the CA certificate.

Create the server certificate:

```
$ make server
```



### Making a certificate for a public CA

If you wish to use an existing certificate authority, you can create a certificate signing request for the server certificate, edit `server.cnf` as above, and run the following command.

```
$ make server.csr
```

This step creates a "Certificate Signing Request" suitable for submission to a public CA.



## Making a Client certificate

Client certificates are used by EAP-TLS, and optionally by EAP-TTLS and PEAP.  The following steps outline how to create a client certificate that is signed by the CA certificate created above.  

You will have to have the password for the CA certificate in the `input_password` and `output_password` fields of the `ca.cnf` file.

kgkmb1pkBerpE7QbWn1yhiPT

```
$ vi client.cnf
```

Edit the `input_password` and `output_password` fields to be the password for the client certificate.  You will have to give these passwords to the end user who will be using the certificates.

Edit the `[client]` section to have the correct values for your country, state, etc.  Be sure that the `commonName` field here is the `User-Name` which will be used for logins!

```
$ make client
```

The users certificate will be in `emailAddress.pem`, e.g. `user@example.com.pem`.

To create another client certificate, just repeat the steps for making a client certificate, being sure to enter a different login name for `commonName`, and a different password.



## Performance

EAP performance for EAP-TLS, TTLS, and PEAP is dominated by SSL calculations.  That is, a normal system can handle PAP authentication at a rate of 10k packets/s.  However, SSL involves RSA calculations,  which are very expensive.  To benchmark your system, do:

```
$ openssl speed rsa
```

or

```
$ openssl speed rsa2048
```

to test 2048 bit keys.

The number that is printed is the **maximum** number of authentications per second which can be done for EAP-TLS (or TTLS, or PEAP).  In practice, you will see results much lower than this number, i.e. the  actual EAP-TLS performance may be half of the number printed here.

The reason is that EAP requires many round-trip packets, whereas `openssl speed rsa2028` only does RSA calculations, and nothing else.

## Compatibility

The certificates created using this method are known to be compatible with ALL operating systems.  Some common issues are:

* iOS and macOS have requirements on certificates.  See:
  https://support.apple.com/en-us/HT210176

* Many systems require certain OIDs in the certificates (`id-kp-serverAuth` for `TLS Web server authentication`).
  If the certificate does not contain these fields, the system will stop doing EAP.  The most visible effect is that the client starts EAP, gets a few Access-Challenge packets, and then a little 
  while later re-starts EAP.  If this happens, see the FAQ, and the comments in `raddb/mods-available/eap` for how to fix it.
  
* All systems requires the root certificates to be on the client PC. 
  If it doesn't have them, you will see the same issue as above.

* Windows XP post SP2 has a bug where it has problems with certificate chains.  i.e. if the server certificate is an intermediate one, and not a root one, then authentication will silently fail, as above.
  
* Some versions of Windows CE cannot handle 4K RSA certificates. 
  They will (again) silently fail, as above.

* In none of these cases will Windows give the end user any reasonable error message describing what went wrong.  This leads people to blame the RADIUS server.  That blame is misplaced.
  
* Certificate chains of more than 64K bytes are known to not work.
  This is partly a problem in FreeRADIUS.  However, most clients cannot handle 64K certificate chains.  Most Access Points will shut down the EAP session after about 50 round trips, while 64K certificate  chains will take about 60 round trips.  So don't use large certificate chains.  They will only work after everyone upgrades everything in the network.
  
* All other operating systems are known to work with EAP and FreeRADIUS.  This includes Linux, the BSDs, macOS, iOS, Android, Solaris, Symbian, along with all known embedded systems, phones, WiFi devices, etc.



## Security Considerations

The default certificate configuration files uses SHA256 for message digests for security.







https://serverfault.com/questions/872314/how-do-i-force-freeradius-to-check-certificates-validity



```
root@freeradius:/etc/freeradius/certs# make client
openssl ca -batch -keyfile ca.key -cert ca.pem -in client.csr  -key 'qsNlEL50kVK0DjzJpLr83Mns' -out client.crt -extensions xpclient_ext -extfile xpextensions -config ./client.cnf
Using configuration from ./client.cnf
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 4 (0x4)
        Validity
            Not Before: Jul 23 17:47:22 2021 GMT
            Not After : Jul 23 17:47:22 2022 GMT
        Subject:
            countryName               = BR
            stateOrProvinceName       = Sao Paulo
            organizationName          = NIC.br
            commonName                = dlucas
            emailAddress              = dlucas@registro.br
        X509v3 extensions:
            X509v3 Extended Key Usage: 
                TLS Web Client Authentication
            X509v3 CRL Distribution Points: 

                Full Name:
                  URI:http://www.example.com/example_ca.crl

Certificate is to be certified until Jul 23 17:47:22 2022 GMT (365 days)

Write out database with 1 new entries
Data Base Updated
openssl pkcs12 -export -in client.crt -inkey client.key -out client.p12  -passin pass:'ze7c9xi' -passout pass:'ze7c9xi'
chmod g+r client.p12
cp client.p12 'dlucas@registro.br'.p12
openssl pkcs12 -in client.p12 -out client.pem -passin pass:'ze7c9xi' -passout pass:'ze7c9xi'
chmod g+r client.pem
cp client.pem 'dlucas@registro.br'.pem

```

