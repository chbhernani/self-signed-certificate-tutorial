# Self-Signed Certificate Tutorial
This repository aims to guide creation of a Self-Signed Certificate with localhost development purpose.

## Prerequisites

- Have a Java JDK installed
- Have OpenSSL installed

## Steps

**NOTE:** The below steps consider the very essential content in order to get everything in place and working.

1. Create a Private Key (_domain.key_) and CSR (_domain.csr_) file entering `localhost` value for `Common Name` and leaving all the rest with default values.
```
openssl req -newkey rsa:2048 -nodes -keyout domain.key -out domain.csr
```
```
...
...
...
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:localhost <<<<<<<<-----------------
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

2. Create a self-signed certificate (_domain.crt_) with existing private key and CSR files:
```
openssl x509 -signkey domain.key -in domain.csr -req -days 730 -out domain.crt
```

3. Create self-signed certificate authority (CA) files: private key (_rootCA.key_) and self-signed root CA certificate (_rootCA.crt_). When asked enter any password (take not of it!) and `localhost` value for `Common Name`:
```
openssl req -newkey rsa:2048 -x509 -sha256 -days 1825 -keyout rootCA.key -out rootCA.crt
```
```
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:localhost
Email Address []:
```

4. Create a configuration text-file (_domain.ext_) with the following content:
```
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
subjectAltName = @alt_names
[alt_names]
DNS.1 = localhost
```

5. Sign the previously generated CSR (_domain.csr_) file with the root CA certificate and its private key. When asked enter the password for root CA certificate (previous step):
```
openssl x509 -req -CA rootCA.crt -CAkey rootCA.key -in domain.csr -out domain.crt -days 730 -CAcreateserial -extfile domain.ext
```
```
Certificate request self-signature ok
subject=C=AU, ST=Some-State, O=Internet Widgits Pty Ltd, CN=localhost
Enter pass phrase for rootCA.key:
```

6. In order to avoid `PKIX path building failed` error, import the root CA certificate to local `cacerts` keystore. When asked enter `yes`:
PS: keystore password may be different. `changeit` is the default one.
```
sudo keytool -import -alias rootCA -keystore $JAVA_HOME/lib/security/cacerts -file rootCA.crt -storepass changeit
```
```
...
...
Trust this certificate? [no]:  yes
Certificate was added to keystore
```

7. Check `rootCA` alias exists in `cacerts` list:
```
keytool -list -keystore $JAVA_HOME/lib/security/cacerts -storepass changeit > list.txt
```
```
...
...
rootca, Mar 7, 2024, trustedCertEntry, 
Certificate fingerprint (SHA-256): 84:D9:1C.....
...
...
```

8. If necessary, delete the entry with the below command:
```
keytool -delete -alias rootCA -keystore $JAVA_HOME/lib/security/cacerts -storepass changeit
```

9. Finally, for local development use the now self-signed _domain.crt_ and _domain.key_ files (maybe also the password set on steps above).

Sources:
- https://www.baeldung.com/openssl-self-signed-cert
- https://www.baeldung.com/jvm-certificate-store-errors
- http://java.globinch.com/enterprise-java/security/pkix-path-building-failed-validation-sun-security-validatorexception/

