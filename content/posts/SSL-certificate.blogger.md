---
title: "SSL Certificate Mutual Authentication"
date: 2019-07-17T21:35:50+01:00
draft: true
tags: ["SSL", "openssl", "Digital Certificates", "pki", "SSL mutual authentication", "x509", "CA", "curl"]
---
<!--- Below style are also defined in static/css/my.css file.
They are repeatedly defined here so that pandoc can generate
the final HTML with all necessary css styles.
Note: draft: true above. This prevents publishing it to GitHUB.
--->
<style>
/* To highlight text in Green in pre tag */
.hl {color: #008A00;}
/* To highlight text in Bold Green in pre tag */
.hlb {color: #008A00; font-weight: bold;}
/* To highlight text in Bold Red in pre tag */
.hlbr {color:#e90001; font-weight: bold;}
/* <code> tag does not work in blogger. Use following class with span tag */
.code {
    color:#7e168d; 
    background: #f0f0f0; 
    padding: 0.1em 0.4em;
    font-family: SFMono-Regular, Consolas, "Liberation Mono", Menlo, Courier, monospace;
}
</style>

# Introduction
We will be creating our own CA (Certificate Authority), CSR (Certificate Signing Request), signed certificate. We will be creating server side and client side certificate and will verify them using `openssl` and `curl`.

# Prepare environment

## os used
```bash
$ cat /etc/fedora-release
Fedora release 30 (Thirty)

$ uname -a
Linux localhost.localdomain 5.0.9-301.fc30.x86_64 #1 SMP Tue Apr 23 23:57:35 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```

## Prepare openssl conf files
A reference conf file can be found in `/etc/pki/tls/openssl.cnf` location and by default openssl uses this file. We will be creating our own conf files. One for CA and other one to generate CSRs.

**Note:** You do not need to be `root` to run following commands.
```bash
$ mkdir -p myPKI
$ cd mkPKI
$ mkdir -p caCert  caDatabase  CSRs  newCreatedCerts  opensslConf  privateKeys
$ touch caDatabase/index.txt
$ touch caDatabase/serial
$ echo "10" > caDatabase/serial
```
### Create following four config files.
* `opensslConf/CA_openssl.cnf`
* `opensslConf/csr_openssl.cnf`
* `opensslConf/server_san_names.cnf`
* `opensslConf/client_san_names.cnf`
```
$ cat opensslConf/CA_openssl.cnf 
HOME			= .
oid_section		= new_oids

openssl_conf = default_modules
[ default_modules ]
ssl_conf = ssl_module

[ ssl_module ]
system_default = crypto_policy

[ crypto_policy ]
.include = /etc/crypto-policies/back-ends/opensslcnf.config

[ new_oids ]
tsa_policy1 = 1.2.3.4.1
tsa_policy2 = 1.2.3.4.5.6
tsa_policy3 = 1.2.3.4.5.7

[ ca ]
default_ca	= CA_default		# The default ca section

[ CA_default ]
dir		= ./		        # Where everything is kept
new_certs_dir   = $dir/newCreatedCerts
database	= $dir/caDatabase/index.txt	# database index file.
unique_subject	= no			# Set to 'no' to allow creation of
serial		= $dir/caDatabase/serial 		# The current serial number
name_opt 	= ca_default		# Subject Name options
cert_opt 	= ca_default		# Certificate field options
copy_extensions = copy

default_days	= 365			# how long to certify for
default_crl_days= 30			# how long before next CRL
default_md	= sha256		# use SHA-256 by default
preserve	= no			# keep passed DN ordering
policy		= policy_match

# For the CA policy
[ policy_match ]
countryName		= match
stateOrProvinceName	= match
organizationName	= match
organizationalUnitName	= optional
commonName		= supplied
emailAddress		= optional

# For the 'anything' policy
[ policy_anything ]
countryName		= optional
stateOrProvinceName	= optional
localityName		= optional
organizationName	= optional
organizationalUnitName	= optional
commonName		= supplied
emailAddress		= optional

[ req ]
default_bits		= 2048
default_md		= sha256
default_keyfile 	= privkey.pem
distinguished_name	= req_distinguished_name
attributes		= req_attributes

# The extensions to add to the self signed cert (This CA will be signing its own certificate, hence self signed)
x509_extensions	= v3_ca	
string_mask = utf8only

[ req_distinguished_name ]
countryName			= Country Name (2 letter code)
countryName_default		= XX
countryName_min			= 2
countryName_max			= 2
stateOrProvinceName		= State or Province Name (full name)
localityName			= Locality Name (eg, city)
localityName_default		= Default City
0.organizationName		= Organization Name (eg, company)
0.organizationName_default	= Default Company Ltd
organizationalUnitName		= Organizational Unit Name (eg, section)
commonName			= Common Name (eg, your name or your server\'s hostname)
commonName_max			= 64
emailAddress			= Email Address
emailAddress_max		= 64

[ req_attributes ]
challengePassword		= A challenge password
challengePassword_min		= 4
challengePassword_max		= 20
unstructuredName		= An optional company name

[ v3_ca ]
subjectKeyIdentifier=hash
authorityKeyIdentifier=keyid:always,issuer
basicConstraints = critical,CA:true
nsCertType = sslCA, emailCA
```

```
$ cat opensslConf/csr_openssl.cnf 
[ req ]
default_bits		= 2048
default_md		= sha256
default_keyfile 	= privkey.pem
distinguished_name	= req_distinguished_name
attributes              = req_attributes
string_mask = utf8only

[ req_distinguished_name ]
countryName			= Country Name (2 letter code)
countryName_default		= XX
countryName_min			= 2
countryName_max			= 2
stateOrProvinceName		= State or Province Name (full name)
localityName			= Locality Name (eg, city)
localityName_default		= Default City
0.organizationName		= Organization Name (eg, company)
0.organizationName_default	= Default Company Ltd
organizationalUnitName		= Organizational Unit Name (eg, section)
#organizationalUnitName_default	=
commonName			= Common Name (eg, your name or your server\'s hostname)
commonName_max			= 64
emailAddress			= Email Address
emailAddress_max		= 64

[ req_attributes ]
challengePassword               = A challenge password
challengePassword_min           = 4
challengePassword_max           = 20
unstructuredName                = An optional company name

[ server ]
basicConstraints=CA:FALSE
nsCertType			= server
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
nsComment			= "OpenSSL Generated Certificate"
subjectKeyIdentifier=hash
extendedKeyUsage = serverAuth
subjectAltName = @san_names

[ client ]
basicConstraints=CA:FALSE
nsCertType			= client, email
keyUsage = nonRepudiation, digitalSignature, keyEncipherment
nsComment			= "OpenSSL Generated Certificate"
subjectKeyIdentifier=hash
extendedKeyUsage = clientAuth
subjectAltName = @san_names
```

```
$ cat opensslConf/server_san_names.cnf 
[ san_names ]
DNS.1 = myhost.full.domain.name
```

```
$ cat opensslConf/client_san_names.cnf 
[ san_names ]
DNS.1 = client.full.domain.name
```

## Create self signed CA certificate.
```bash
$ cd myPKI
$ openssl req -config opensslConf/CA_openssl.cnf \
-new -x509 \
-out caCert/myCA.crt \
-newkey rsa:2048 -keyout caCert/myCAkey.pem \
-extensions v3_ca \
-days 1000 \
-subj "/emailAddress=user@junkemail.com/C=GB/ST=London/L=City of London/O=My Organization/OU=Engg/CN=root CA"

Generating a RSA private key
...........................+++++
.....................+++++
writing new private key to 'caCert/myCAkey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
```
```bash
$ ls -1 caCert/
myCA.crt
myCAkey.pem
```

## Check CA certificate
<pre>
$ openssl x509 -in caCert/myCA.crt -text -noout

<span class="hlb">Certificate:</span>
    Data:
        Version: 3 (0x2)
        Serial Number:
            67:4e:dd:00:82:65:95:e0:92:91:70:52:2a:35:cc:bb:33:57:78:22
        Signature Algorithm: sha256WithRSAEncryption
        Issuer: emailAddress = user@junkemail.com, C = GB, ST = London, L = City of London, O = My Organization, OU = Engg, <span class="hlb">CN = root CA</span>
        Validity
            Not Before: Aug  3 10:43:28 2019 GMT
            Not After : Apr 29 10:43:28 2022 GMT
        Subject: emailAddress = user@junkemail.com, C = GB, ST = London, L = City of London, O = My Organization, OU = Engg, CN = root CA
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:99:05:47:02:ec:ab:1c:12:46:4e:fa:65:94:fd:
                    72:99:bc:81:e7:12:ba:5c:f0:17:46:45:72:dd:c1:
                    3d:7f:58:4b:f4:85:6e:99:4d:dc:24:e4:c4:01:0b:
                    c3:64:48:8a:89:3f:a8:ef:66:d6:d7:8d:83:58:07:
                    a2:09:69:de:36:06:de:04:1d:40:5a:1e:8b:31:e1:
                    7e:2a:8f:79:d8:f0:5d:84:6d:c9:20:80:4a:83:af:
                    60:ec:ff:0c:76:26:bb:30:3d:5b:93:21:f8:1c:0c:
                    5c:ab:4a:07:f7:2f:b2:ca:fd:0c:a7:0f:62:65:78:
                    43:84:98:63:12:6c:59:09:9f:df:4e:7c:00:91:a1:
                    e8:52:3c:54:9f:c2:6a:0a:c5:2c:85:fe:35:f5:83:
                    e6:40:2f:8f:ae:19:84:d8:fc:fc:4c:35:5f:34:8a:
                    9f:f8:2f:76:7a:d4:ba:4b:c5:5b:0a:06:5b:52:27:
                    d1:f8:c5:de:69:3c:0b:80:e1:53:b3:b5:7b:ca:33:
                    9c:d4:ae:72:04:f3:3c:14:e7:8d:71:91:62:52:f8:
                    0e:5a:67:8c:a7:1e:fb:8a:1a:b4:75:0a:68:32:57:
                    ed:ff:6a:93:b8:a8:90:fa:0b:21:c6:8a:71:88:b3:
                    8e:23:c1:71:af:41:4f:4d:96:a3:14:0a:76:3e:05:
                    c1:8f
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Subject Key Identifier: 
                E2:E6:68:05:8F:90:98:4F:36:E7:60:1A:B5:C8:95:FE:14:D8:84:20
            X509v3 Authority Key Identifier: 
                keyid:E2:E6:68:05:8F:90:98:4F:36:E7:60:1A:B5:C8:95:FE:14:D8:84:20

            X509v3 Basic Constraints: critical
                <span class="hlb">CA:TRUE</span>
            Netscape Cert Type: 
                <span class="hlb">SSL CA, S/MIME CA</span>
    Signature Algorithm: sha256WithRSAEncryption
         30:f7:ec:d8:ac:81:55:89:d4:51:89:a7:4a:58:87:b0:7a:3d:
         97:dc:96:4e:1f:77:77:ed:b8:33:af:ee:4e:b6:3c:66:d0:b4:
         0b:d5:a0:82:f2:34:14:da:32:f7:7b:fb:84:34:39:49:e6:ab:
         c6:23:19:9a:0e:f1:f0:dc:f7:d5:12:ad:9b:c2:7a:f6:c5:89:
         f0:c1:c1:77:1c:03:85:6e:fa:b0:3f:a7:63:72:a5:43:a7:c5:
         ca:fb:61:a8:7c:4a:14:bf:59:ab:a9:37:cd:6c:57:f1:3f:0c:
         36:69:1d:e7:8e:40:95:c8:a7:29:88:72:1c:95:ae:fc:10:93:
         4b:62:01:6f:52:7d:c4:37:ad:23:1a:e5:62:fb:9c:d5:63:89:
         7c:16:d6:1f:f0:41:58:8b:1d:44:5e:c5:8e:29:df:d2:61:e4:
         74:16:9f:73:2d:1d:f7:c0:13:42:fc:80:95:0d:33:30:1a:46:
         4f:cd:16:ef:3c:31:ad:2c:e8:d7:a9:c6:c8:55:8f:7f:1a:67:
         ee:22:b3:01:9d:61:fa:91:c6:54:ac:e2:d4:fb:a5:ef:39:7c:
         59:40:4f:93:81:f6:ee:4a:19:fa:05:7c:78:13:84:b9:17:25:
         26:20:ca:37:84:04:41:61:22:f3:87:fc:76:3c:72:ef:f7:d6:
         f7:e6:61:b7
</pre>

## Verify CA certificate
<pre>
$ openssl verify -verbose  caCert/myCA.crt

emailAddress = user@junkemail.com, C = GB, ST = London, L = City of London, O = My Organization, OU = Engg, CN = root CA
error 18 at 0 depth lookup: self signed certificate
<span class="hlbr">error caCert/myCA.crt: verification failed</span>
</pre>

**Note:** Above verification failed because this CA cert is self signed and is NOT issued by certified authority.

Run following command to tell `openssl` to use our signed CA cert.
```bash
$ openssl verify -CAfile caCert/myCA.crt -verbose  caCert/myCA.crt

caCert/myCA.crt: OK
```

## Create a CSR request for server.
<pre>
$ openssl req \
-config <(cat opensslConf/<span class="hlb">csr_openssl.cnf</span> opensslConf/<span class="hlb">server_san_names.cnf</span>) \
-new \
-newkey rsa:2048 \
-nodes \
-days 365 \
-out CSRs/server.csr \
-keyout privateKeys/serverKey.pem \
-reqexts <span class="hlb">server</span> \
-subj "/emailAddress=user2@junkemail.com/C=GB/ST=London/L=City of London/O=My Organization/OU=Engg/<span class="hlb">CN=myhost.full.domain.name"</span>

Ignoring -days; not generating a certificate
Generating a RSA private key
...........+++++
........+++++
writing new private key to 'privateKeys/serverKey.pem'
-----
</pre>

```bash
$ ls -1 CSRs/
server.csr
```

## Create a CSR request for client.
<pre>
$ openssl req \
-config <(cat opensslConf/<span class="hlb">csr_openssl.cnf</span> opensslConf/<span class="hlb">client_san_names.cnf</span>) \
-new \
-newkey rsa:2048 \
-nodes \
-days 365 \
-out CSRs/myClient.csr \
-keyout privateKeys/myClientKey.pem \
-reqexts client \
-subj "/emailAddress=user3@some.com/C=GB/ST=London/L=City of London/O=My Organization/OU=Engg/<span class="hlb">CN=client.full.domain.name</span>"

Ignoring -days; not generating a certificate
Generating a RSA private key
................................................................+++++
.............+++++
writing new private key to 'privateKeys/myClientKey.pem'
-----
</pre>

```bash
$ ls -og privateKeys/
total 8
-rw-------. 1 1704 Aug  3 13:03 myClientKey.pem
-rw-------. 1 1704 Aug  3 12:17 serverKey.pem

$ ls -og CSRs/
total 8
-rw-rw-r--. 1 1350 Aug  3 13:03 myClient.csr
-rw-rw-r--. 1 1358 Aug  3 12:17 server.csr
```

### Check Server CSR request
<pre>
$ openssl <span class="hlb">req</span> -in CSRs/server.csr -text -noout

<span class="hlb">Certificate Request:</span>
    Data:
        Version: 1 (0x0)
        Subject: emailAddress = user2@junkemail.com, C = GB, ST = London, L = City of London, O = My Organization, OU = Engg, <span class="hlb">CN = myhost.full.domain.name</span>
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:9b:39:32:a0:0c:c0:66:2a:b3:a2:0f:17:66:8e:
                    8a:60:b2:1b:5b:14:af:87:a3:af:06:a7:d0:81:cd:
                    87:fa:54:1f:d0:51:8f:d0:82:40:1a:56:f7:3a:de:
                    4a:a9:44:e5:fb:5e:26:59:2e:d0:74:d6:70:8e:79:
                    8f:22:22:07:7b:09:75:73:fb:db:64:b3:33:ef:85:
                    8e:c3:96:a4:aa:12:7a:2d:09:db:1f:f7:3a:eb:e5:
                    c0:c7:67:dc:ff:d0:02:50:dd:6a:7f:26:95:52:6a:
                    ed:ca:f5:b4:18:0b:c7:1a:33:1b:e5:40:96:6d:74:
                    c3:c3:a2:81:01:96:7b:b2:d3:6c:d1:ad:9c:59:46:
                    32:f8:b2:71:ea:57:77:4c:fe:b5:a6:a1:33:21:09:
                    73:b6:f0:72:bf:b4:be:b6:b7:b3:82:81:91:15:7c:
                    16:30:0c:2f:30:45:8c:25:37:53:3b:0f:eb:4b:46:
                    2b:4d:4e:2b:bc:e9:62:fe:10:1b:26:bc:ca:8f:20:
                    c2:34:a0:19:50:08:97:70:4e:82:5c:e8:53:80:7d:
                    7c:fc:61:14:f5:e1:ea:b8:7b:57:64:57:c0:85:a3:
                    e3:97:02:70:c8:84:82:5e:de:56:11:f4:23:0a:76:
                    44:d7:dd:ce:d7:46:56:53:0e:cc:03:94:b5:a2:75:
                    af:a7
                Exponent: 65537 (0x10001)
        Attributes:
        Requested Extensions:
            X509v3 Basic Constraints: 
                <span class="hlb">CA:FALSE</span>
            Netscape Cert Type: 
                <span class="hlb">SSL Server</span>
            X509v3 Key Usage: 
                Digital Signature, Non Repudiation, Key Encipherment
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                10:AB:8E:14:C4:C3:01:06:E8:77:4B:30:A3:B9:B0:18:8C:51:13:D7
            X509v3 Extended Key Usage: 
                <span class="hlb">TLS Web Server Authentication</span>
            <span class="hlb">X509v3 Subject Alternative Name:</span> 
                DNS:myhost.full.domain.name
    Signature Algorithm: sha256WithRSAEncryption
         43:6f:f5:64:34:87:2d:69:d0:80:f2:0d:6b:ec:ae:86:00:0c:
         0f:03:4e:6d:70:2d:f5:42:a0:7f:12:bf:27:f7:74:9e:85:5d:
         02:ac:bc:31:d0:99:73:87:ad:ea:6a:76:e1:4e:51:4c:f7:77:
         bd:04:07:05:97:31:ee:c9:b2:79:69:32:96:86:5f:6d:1e:3d:
         e8:e2:9d:d8:fb:16:ec:06:c3:43:eb:f8:ee:30:0c:da:e1:6a:
         f3:c6:54:21:06:a1:81:18:34:8e:c6:6b:db:e7:d5:a4:06:09:
         b4:46:fd:6a:23:84:66:21:4d:bc:18:f0:e3:f5:bb:6b:32:f0:
         72:62:2c:b4:0d:ef:cc:5c:e4:69:31:3a:7b:b6:c8:44:94:b9:
         5c:4f:a9:2b:14:57:4e:f7:5c:5a:a5:a8:e5:61:90:88:3f:30:
         0b:2c:5e:90:08:11:73:7e:c2:94:30:42:3d:8c:41:94:55:56:
         9b:ab:d3:c3:71:ed:64:ea:10:47:d6:2b:0d:3c:9f:f1:a6:64:
         a5:8a:ca:86:25:d3:17:b5:12:b8:47:ea:34:6b:a4:05:17:0f:
         91:dd:e5:e0:24:ae:a6:1d:ef:10:60:af:72:15:5e:eb:98:30:
         63:f8:b5:2a:1e:fc:1d:b5:1f:01:2c:93:72:48:78:b8:ec:2c:
         ca:44:5a:e8
</pre>

### Check client CSR request
<pre>
$ openssl <span class="hlb">req</span> -in CSRs/myClient.csr -text -noout

<span class="hlb">Certificate Request:</span>
    Data:
        Version: 1 (0x0)
        Subject: emailAddress = user3@some.com, C = GB, ST = London, L = City of London, O = My Organization, OU = Engg, <span class="hlb">CN = client.full.domain.name</span>
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:b7:bc:80:ee:fb:da:4b:40:04:b5:37:9a:d9:6a:
                    39:c3:b9:e9:c9:52:68:68:e0:76:4f:35:36:00:8e:
                    b8:37:70:49:17:29:c5:7f:6a:99:50:6b:84:dc:e3:
                    97:6f:a4:b4:83:b9:88:a4:3a:01:68:b4:75:07:45:
                    63:4c:5b:2c:b4:07:c3:98:10:1e:a0:f8:46:e8:0e:
                    a6:71:04:61:50:aa:8a:31:91:d1:9c:73:ad:e4:3c:
                    f9:40:de:43:f1:f5:5d:37:13:ab:c3:cc:2c:eb:46:
                    66:81:4a:27:06:c5:bf:8b:07:fd:67:67:58:8b:59:
                    b2:15:db:88:ff:11:23:f8:42:e3:fc:18:55:95:52:
                    30:19:3e:eb:81:04:ec:e0:94:bf:6a:f6:63:dd:34:
                    00:54:71:ec:bd:eb:13:1d:b3:10:52:6c:d2:22:32:
                    9d:e8:aa:54:66:3c:7b:5a:bb:42:eb:5d:34:57:51:
                    05:49:fd:86:e0:1c:91:02:75:af:87:96:bf:be:f7:
                    de:13:78:a2:61:9d:14:06:32:1d:33:03:61:c9:0f:
                    5d:db:0d:33:51:ec:cb:c2:d1:4b:64:40:da:05:6f:
                    c5:41:3a:7a:68:7a:23:0a:71:2d:81:31:c8:e7:a4:
                    fb:fc:30:a6:27:63:51:ca:38:7a:84:12:9d:44:e1:
                    3b:53
                Exponent: 65537 (0x10001)
        Attributes:
        Requested Extensions:
            X509v3 Basic Constraints: 
                <span class="hlb">CA:FALSE</span>
            Netscape Cert Type: 
                <span class="hlb">SSL Client, S/MIME</span>
            X509v3 Key Usage: 
                Digital Signature, Non Repudiation, Key Encipherment
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                7C:9D:A8:26:60:12:31:DE:88:21:FB:EA:9C:77:B9:B1:38:E0:52:67
            X509v3 Extended Key Usage: 
                <span class="hlb">TLS Web Client Authentication</span>
            <span class="hlb">X509v3 Subject Alternative Name:</span> 
                DNS:client.full.domain.name
    Signature Algorithm: sha256WithRSAEncryption
         26:69:46:d5:5b:69:af:a5:16:69:03:0f:e7:2a:ca:f1:38:e0:
         07:2c:ff:50:85:b9:91:a8:23:86:0c:2a:e9:64:da:8a:9e:31:
         d9:fd:0b:02:ef:67:14:09:65:be:14:b2:93:70:7a:2b:0b:96:
         38:57:dc:ba:ea:70:91:20:18:dc:fa:d2:49:f1:b4:5b:7c:41:
         c6:c1:2a:6d:7a:49:3f:18:42:49:3d:dc:d2:26:e0:71:9a:44:
         bb:5d:f6:de:d4:c9:20:96:bc:1e:0e:e5:36:ff:37:9d:92:bd:
         88:5e:1e:af:d4:72:dc:6a:27:41:51:a3:4b:a5:28:11:b1:bc:
         92:48:c5:75:48:79:92:eb:7e:d8:f9:71:03:8d:b3:cb:62:fd:
         76:7f:2d:70:95:93:4c:db:2a:90:0a:e2:61:f8:00:c1:a6:fa:
         83:3b:dd:76:a2:73:57:66:4c:b2:8a:6f:6e:78:b7:1e:c4:70:
         d5:26:76:83:b6:47:44:70:50:be:aa:7b:d8:f2:45:6e:57:dc:
         92:04:8d:78:fd:7d:70:84:35:52:9a:b4:68:ef:09:29:96:fb:
         0c:ae:51:e3:0c:d0:f3:bd:91:3e:99:7d:f8:d3:eb:e4:86:da:
         08:01:ec:46:2b:0c:76:65:61:c9:4e:cb:17:60:bf:72:55:6d:
         44:73:25:66
</pre>

## Sign Server CSR and generate server certificate
<pre>
$ openssl ca \
-config <span class="hlb">opensslConf/CA_openssl.cnf</span> \
-policy policy_anything \
-cert caCert/myCA.crt \
-keyfile caCert/myCAkey.pem \
-in CSRs/server.csr \
-out newCreatedCerts/serverCert.pem

Using configuration from opensslConf/CA_openssl.cnf
Enter pass phrase for caCert/myCAkey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 16 (0x10)
        Validity
            Not Before: Aug  3 11:40:34 2019 GMT
            Not After : Aug  2 11:40:34 2020 GMT
        Subject:
            countryName               = GB
            stateOrProvinceName       = London
            localityName              = City of London
            organizationName          = My Organization
            organizationalUnitName    = Engg
            commonName                = myhost.full.domain.name
            emailAddress              = user2@junkemail.com
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Cert Type:
                SSL Server
            X509v3 Key Usage:
                Digital Signature, Non Repudiation, Key Encipherment
            Netscape Comment:
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier:
                10:AB:8E:14:C4:C3:01:06:E8:77:4B:30:A3:B9:B0:18:8C:51:13:D7
            X509v3 Extended Key Usage:
                TLS Web Server Authentication
            X509v3 Subject Alternative Name:
                DNS:myhost.full.domain.name
Certificate is to be certified until Aug  2 11:40:34 2020 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
</pre>

```bash
$ ls -og  newCreatedCerts/
total 16
-rw-rw-r--. 1 5035 Aug  3 12:40 10.pem
-rw-rw-r--. 1 5035 Aug  3 12:40 serverCert.pem
```
### check signed server certificate
<pre>
$ openssl <span class="hlb">x509</span> -in newCreatedCerts/serverCert.pem -text -noout

<span class="hlb">Certificate:</span>
    Data:
        Version: 3 (0x2)
        Serial Number: 16 (0x10)
        Signature Algorithm: sha256WithRSAEncryption
        <span class="hlb">Issuer:</span> emailAddress = user@junkemail.com, C = GB, ST = London, L = City of London, O = My Organization, OU = Engg, <span class="hlb">CN = root CA</span>
        Validity
            Not Before: Aug  3 11:40:34 2019 GMT
            Not After : Aug  2 11:40:34 2020 GMT
        <span class="hlb">Subject:</span> C = GB, ST = London, L = City of London, O = My Organization, OU = Engg, <span class="hlb">CN = myhost.full.domain.name</span>, emailAddress = user2@junkemail.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:9b:39:32:a0:0c:c0:66:2a:b3:a2:0f:17:66:8e:
                    8a:60:b2:1b:5b:14:af:87:a3:af:06:a7:d0:81:cd:
                    87:fa:54:1f:d0:51:8f:d0:82:40:1a:56:f7:3a:de:
                    4a:a9:44:e5:fb:5e:26:59:2e:d0:74:d6:70:8e:79:
                    8f:22:22:07:7b:09:75:73:fb:db:64:b3:33:ef:85:
                    8e:c3:96:a4:aa:12:7a:2d:09:db:1f:f7:3a:eb:e5:
                    c0:c7:67:dc:ff:d0:02:50:dd:6a:7f:26:95:52:6a:
                    ed:ca:f5:b4:18:0b:c7:1a:33:1b:e5:40:96:6d:74:
                    c3:c3:a2:81:01:96:7b:b2:d3:6c:d1:ad:9c:59:46:
                    32:f8:b2:71:ea:57:77:4c:fe:b5:a6:a1:33:21:09:
                    73:b6:f0:72:bf:b4:be:b6:b7:b3:82:81:91:15:7c:
                    16:30:0c:2f:30:45:8c:25:37:53:3b:0f:eb:4b:46:
                    2b:4d:4e:2b:bc:e9:62:fe:10:1b:26:bc:ca:8f:20:
                    c2:34:a0:19:50:08:97:70:4e:82:5c:e8:53:80:7d:
                    7c:fc:61:14:f5:e1:ea:b8:7b:57:64:57:c0:85:a3:
                    e3:97:02:70:c8:84:82:5e:de:56:11:f4:23:0a:76:
                    44:d7:dd:ce:d7:46:56:53:0e:cc:03:94:b5:a2:75:
                    af:a7
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints: 
                <span class="hlb">CA:FALSE</span>
            Netscape Cert Type: 
                <span class="hlb">SSL Server</span>
            X509v3 Key Usage: 
                Digital Signature, Non Repudiation, Key Encipherment
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                10:AB:8E:14:C4:C3:01:06:E8:77:4B:30:A3:B9:B0:18:8C:51:13:D7
            X509v3 Extended Key Usage: 
                <span class="hlb">TLS Web Server Authentication</span>
            X509v3 Subject Alternative Name: 
                <span class="hlb">DNS:myhost.full.domain.name</span>
    Signature Algorithm: sha256WithRSAEncryption
         2d:f2:ec:1d:96:98:d7:4c:4e:f7:5d:83:e1:f6:b7:99:77:cc:
         31:30:5e:e1:7f:75:a1:b9:aa:b1:f2:e9:20:8c:fe:87:be:01:
         3c:a4:ee:cb:b8:f4:88:34:eb:3e:29:79:9f:2e:a0:db:3e:50:
         9f:5b:a2:cf:d7:24:b8:12:da:56:be:c7:69:0e:0c:d9:22:e7:
         2b:d1:dd:cf:c6:17:0a:d2:70:84:a4:ba:fb:27:0c:a7:33:ff:
         94:ed:4e:ea:be:04:b9:30:61:ad:8a:ee:29:81:fa:b1:28:3c:
         f2:63:c4:23:c6:63:46:09:a7:bf:94:69:ee:cb:39:3f:dd:f6:
         dc:41:96:9c:e8:83:a0:93:e7:21:11:57:d4:b9:29:aa:6d:93:
         42:a1:b9:dc:b0:3b:81:c4:6d:9c:51:5d:71:31:b8:e1:c8:43:
         3a:41:bb:47:73:b7:3b:3c:46:ba:e4:53:f3:f4:75:96:25:ad:
         8a:ed:f0:26:64:80:1d:b8:eb:8a:f2:6c:63:09:6d:19:e5:60:
         70:7a:d3:71:17:5c:d3:d1:c0:d0:1a:ec:82:e6:3e:fb:ac:44:
         26:38:d7:63:15:4b:6e:82:6f:fe:fe:9e:fc:e5:3a:6c:37:06:
         72:1f:c9:dc:e4:69:da:1e:31:30:63:d0:2c:cd:a1:e8:f0:ce:
         e3:9d:36:a8
</pre>

### Verify server certificate
```bash
$ openssl verify  -CAfile caCert/myCA.crt newCreatedCerts/serverCert.pem

newCreatedCerts/serverCert.pem: OK
```

## Sign client CSR and generate client certificate
<pre>
$ openssl ca \
-config <span class="hlb">opensslConf/CA_openssl.cnf</span> \
-policy policy_anything \
-cert caCert/myCA.crt \
-keyfile caCert/myCAkey.pem \
-in CSRs/myClient.csr \
-out newCreatedCerts/myClientCert.pem

Using configuration from opensslConf/CA_openssl.cnf
Enter pass phrase for caCert/myCAkey.pem:
Check that the request matches the signature
Signature ok
Certificate Details:
        Serial Number: 17 (0x11)
        Validity
            Not Before: Aug  3 12:18:30 2019 GMT
            Not After : Aug  2 12:18:30 2020 GMT
        Subject:
            countryName               = GB
            stateOrProvinceName       = London
            localityName              = City of London
            organizationName          = My Organization
            organizationalUnitName    = Engg
            commonName                = client.full.domain.name
            emailAddress              = user3@some.com
        X509v3 extensions:
            X509v3 Basic Constraints:
                CA:FALSE
            Netscape Cert Type:
                SSL Client, S/MIME
            X509v3 Key Usage:
                Digital Signature, Non Repudiation, Key Encipherment
            Netscape Comment:
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier:
                7C:9D:A8:26:60:12:31:DE:88:21:FB:EA:9C:77:B9:B1:38:E0:52:67
            X509v3 Extended Key Usage:
                TLS Web Client Authentication
            X509v3 Subject Alternative Name:
                DNS:client.full.domain.name
Certificate is to be certified until Aug  2 12:18:30 2020 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
</pre>

```bash
$ ls -og newCreatedCerts/
total 32
-rw-rw-r--. 1 5035 Aug  3 12:40 10.pem
-rw-rw-r--. 1 5034 Aug  3 13:18 11.pem
-rw-rw-r--. 1 5034 Aug  3 13:18 myClientCert.pem
-rw-rw-r--. 1 5035 Aug  3 12:40 serverCert.pem
```
### check client certificate
<pre>
 $ openssl <span class="hlb">x509</span> -in newCreatedCerts/myClientCert.pem -text -noout 

<span class="hlb">Certificate:</span>
    Data:
        Version: 3 (0x2)
        Serial Number: 17 (0x11)
        Signature Algorithm: sha256WithRSAEncryption
        <span class="hlb">Issuer:</span> emailAddress = user@junkemail.com, C = GB, ST = London, L = City of London, O = My Organization, OU = Engg, <span class="hlb">CN = root CA</span>
        Validity
            Not Before: Aug  3 12:18:30 2019 GMT
            Not After : Aug  2 12:18:30 2020 GMT
        <span class="hlb">Subject:</span> C = GB, ST = London, L = City of London, O = My Organization, OU = Engg, <span class="hlb">CN = client.full.domain.name</span>, emailAddress = user3@some.com
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                RSA Public-Key: (2048 bit)
                Modulus:
                    00:b7:bc:80:ee:fb:da:4b:40:04:b5:37:9a:d9:6a:
                    39:c3:b9:e9:c9:52:68:68:e0:76:4f:35:36:00:8e:
                    b8:37:70:49:17:29:c5:7f:6a:99:50:6b:84:dc:e3:
                    97:6f:a4:b4:83:b9:88:a4:3a:01:68:b4:75:07:45:
                    63:4c:5b:2c:b4:07:c3:98:10:1e:a0:f8:46:e8:0e:
                    a6:71:04:61:50:aa:8a:31:91:d1:9c:73:ad:e4:3c:
                    f9:40:de:43:f1:f5:5d:37:13:ab:c3:cc:2c:eb:46:
                    66:81:4a:27:06:c5:bf:8b:07:fd:67:67:58:8b:59:
                    b2:15:db:88:ff:11:23:f8:42:e3:fc:18:55:95:52:
                    30:19:3e:eb:81:04:ec:e0:94:bf:6a:f6:63:dd:34:
                    00:54:71:ec:bd:eb:13:1d:b3:10:52:6c:d2:22:32:
                    9d:e8:aa:54:66:3c:7b:5a:bb:42:eb:5d:34:57:51:
                    05:49:fd:86:e0:1c:91:02:75:af:87:96:bf:be:f7:
                    de:13:78:a2:61:9d:14:06:32:1d:33:03:61:c9:0f:
                    5d:db:0d:33:51:ec:cb:c2:d1:4b:64:40:da:05:6f:
                    c5:41:3a:7a:68:7a:23:0a:71:2d:81:31:c8:e7:a4:
                    fb:fc:30:a6:27:63:51:ca:38:7a:84:12:9d:44:e1:
                    3b:53
                Exponent: 65537 (0x10001)
        X509v3 extensions:
            X509v3 Basic Constraints: 
                <span class="hlb">CA:FALSE</span>
            Netscape Cert Type: 
                <span class="hlb">SSL Client, S/MIME</span>
            X509v3 Key Usage: 
                Digital Signature, Non Repudiation, Key Encipherment
            Netscape Comment: 
                OpenSSL Generated Certificate
            X509v3 Subject Key Identifier: 
                7C:9D:A8:26:60:12:31:DE:88:21:FB:EA:9C:77:B9:B1:38:E0:52:67
            X509v3 Extended Key Usage: 
                <span class="hlb">TLS Web Client Authentication</span>
            <span class="hlb">X509v3 Subject Alternative Name:</span> 
                DNS:client.full.domain.name
    Signature Algorithm: sha256WithRSAEncryption
         7d:20:3f:08:50:20:83:2d:27:d2:2e:1c:2d:50:f5:d4:bd:47:
         23:bf:5a:a2:6c:61:7d:3e:67:fd:3a:62:be:50:18:75:84:9b:
         d2:a4:1f:a2:ad:de:59:cd:fe:db:f3:95:80:12:c9:62:a3:79:
         73:c1:01:ed:18:25:1c:b6:4e:10:e9:60:54:18:37:0a:04:be:
         0f:9c:76:ff:63:32:69:d9:26:f4:25:46:c5:88:07:ea:f3:50:
         04:3a:f9:8b:47:02:27:8b:f0:5f:64:59:01:7b:45:8a:6d:9e:
         03:75:95:93:f0:d7:83:c8:c2:bf:c4:a5:4e:fb:81:ed:54:68:
         80:ef:8d:07:a0:dc:ab:e8:e7:5d:30:cf:26:7f:c1:3d:92:64:
         03:c4:52:56:a1:e2:c3:dd:a8:79:ba:4a:a4:54:d7:f8:50:2b:
         95:d0:4b:95:c1:d7:f9:d5:a4:23:dc:f0:84:f6:35:3b:d4:f8:
         cb:b6:e4:59:da:a3:27:1d:d7:5c:38:96:0c:a6:d1:d9:81:71:
         e6:fb:f8:41:13:c9:4d:9b:81:fb:33:b3:09:76:5b:1d:f7:d0:
         81:5f:94:14:4f:49:b8:8d:1a:ad:29:bd:fd:cd:a1:46:71:d9:
         0b:9a:50:d3:97:8e:13:a5:d0:5a:1a:65:44:f1:d9:01:1a:65:
         27:16:d1:f0
</pre>

## Verify Client/Server connectivity using certs.
Please note that our server certificate was created using `CN=myhost.full.domain.name`. This is imaginary hostname but client must use `myhost.full.domain.name` to connect to server.

Create a test html file `atest.html` that will be served by openssl server.

```bash
$ cd myPKI
$ mkdir -p serveContent
$ touch serveContent/atest.html
$ ( cat <<EOF
openssl page served.
EOF
) > serveContent/atest.html
```

## Attemps to connect and fetch a page from server (NO mutual auth)
Start a SSL server.
```bash
$ host myhost.full.domain.name
Host myhost.full.domain.name not found: 3(NXDOMAIN)
```
```bash
$ openssl s_server \
-accept 9999 \
-CAfile caCert/myCA.crt \
-cert newCreatedCerts/serverCert.pem \
-key privateKeys/serverKey.pem \
-state \
-WWW

Using default temp DH parameters
ACCEPT
```
#### Fetch `atest.html` file `insecurely`.
insecure fetch
<pre>
$ curl https://localhost:9999/serveContent/atest.html <span class="hlb">--insecure</span>
openssl page served.
</pre>

#### Fetch `atest.html` file `securely`.
secure fetch
<pre>
$ curl https://localhost:9999/serveContent/atest.html <span class="hlb">--cacert caCert/myCA.crt</span>

curl: (60) SSL: no alternative certificate subject name matches target host name 'localhost'
More details here: https://curl.haxx.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
</pre>

**Note:** The error above is due to the mismatch between client calling the server name as `localhost` and certificate's `SAN` name as `myhost.full.domain.name`

#### Fetch `atest.html` file with correct server hostname
using correct server name
```bash
$ curl https://myhost.full.domain.name:9999/serveContent/atest.html --cacert caCert/myCA.crt

curl: (6) Could not resolve host: myhost.full.domain.name
```

#### Fixing DNS resolution for `myhost.full.domain.name`
fix DNS resolution
As `root` user enter the following line in `/etc/resolv.conf` file above all `nameserver` lines.
```
nameserver 127.0.0.1
```
#### Start a temp `DNS` server as following
DNS server starting.

<pre>
$ sudo dnsmasq --no-daemon --listen-address=127.0.0.1 --address=/<span class="hlb">full.domain.name</span>/127.0.0.1 --log-queries

dnsmasq: started, version 2.80 cachesize 150
dnsmasq: compile time options: IPv6 GNU-getopt DBus no-i18n IDN2 DHCP DHCPv6 no-Lua TFTP no-conntrack ipset auth DNSSEC loop-detect inotify dumpfile
dnsmasq: reading /etc/resolv.conf
dnsmasq: ignoring nameserver 127.0.0.1 - local interface
dnsmasq: using nameserver 192.168.97.2#53
dnsmasq: read /etc/hosts - 2 addresses
</pre>

```bash
$ host myhost.full.domain.name
myhost.full.domain.name has address 127.0.0.1
```
#### Fetch `atest.html` file securely after DNS resolution fix.
secure fetch.
<pre>
$ curl https://myhost.full.domain.name:9999/serveContent/atest.html --cacert <span class="hlb">caCert/myCA.crt</span>
openssl page served.
</pre>

## Attemps to connect and fetch a page from server (WITH mutual auth)
### Start openssl server in client verification mode.
<pre>
$ openssl s_server \
-accept 9999 \
-CAfile caCert/myCA.crt \
-cert newCreatedCerts/serverCert.pem \
-key privateKeys/serverKey.pem \
-state \
-WWW \
<span class="hlb">-Verify 1</span>

verify depth is 1, must return a certificate
Using default temp DH parameters
ACCEPT
</pre>

#### Fetch `atest.html` in `insecure` way
insecure fetch
```bash
$ curl https://myhost.full.domain.name:9999/serveContent/atest.html \
--insecure

curl: (56) OpenSSL SSL_read: error:1409445C:SSL routines:ssl3_read_bytes:tlsv13 alert certificate required, errno 0
```
#### Fetch `atest.html` secure way
secure fetch
```bash
$ curl https://myhost.full.domain.name:9999/serveContent/atest.html \
--cacert caCert/myCA.crt

curl: (56) OpenSSL SSL_read: error:1409445C:SSL routines:ssl3_read_bytes:tlsv13 alert certificate required, errno 0
```

#### Fetch `atest.html` secure way using client certs (mutual auth)
```bash
$ curl https://myhost.full.domain.name:9999/serveContent/atest.html \
--cacert caCert/myCA.crt \
--cert newCreatedCerts/myClientCert.pem \
--key privateKeys/myClientKey.pem

openssl page served.
```