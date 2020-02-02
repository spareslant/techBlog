---
title: "Apache ReverseProxySSL SSLClientAuth"
date: 2019-08-04T14:54:01+01:00
draft: false
tags: ["apache", "SSL", "proxy", "reverse proxy", "SSL reverse proxy", "gunicorn", "SSL authentication", "flask" ]
---
<!--- Below style are also defined in static/css/my.css file.
They are repeatedly defined here so that pandoc can generate
the final HTML with all necessary css styles.
--->
<style>
.hl {color: #f155f1;}
.hlb {color: #f155f1; font-weight: bold;}
.hlbr {color:#e90001; font-weight: bold;}
/* <code> tag does not work in blogger. Use following class with span tag */
.code {color:#f20101; background: #f0f0f0; padding: 0.2em;    
</style>

# Introduction
We will be configuring `Apache` in reverse proxy mode. `Apache` will be accepting connections on secure port with SSL client authentication and forward that request to a backend application server `Gunicorn`. Communication between `apache` and `gunicorn` will also be on secure port and SSL authenticated. `Gunicorn` will run a simple `flask` app that will be writing some text to file.

Purpose of this excersize is to introduce end to end secure communication from client to apache to app server with SSL client authentication at every stage. Please note that this exercise represents just the minimal configuration to work and no other authentication mechanism will be mentioned.

**Note:** we will be using self-signed CA authority and all certificates will be signed by this authority.

# Environment Used
```bash
$ cat /etc/fedora-release
Fedora release 30 (Thirty)

$ uname -a
Linux localhost.localdomain 5.0.9-301.fc30.x86_64 #1 SMP Tue Apr 23 23:57:35 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
```
# Prepare Environment
## Create python virtual environment
```bash
$ python3 -m  venv apacheSLL
$ source apacheSLL/bin/activate
$ pip install flask
$ pip install gunicorn
```
## Create all digital certificates
We will be using a PKI setup mentioned in [**this link**]({{< ref "/posts/SSL-certificate.md" >}})
We will be creating following certificates.

* client-side certificate used by `curl`
* apache server-side certificate used in interfacing with client.
* apache client-certificate used in interfacing with `gunicorn`
* `gunicorn` server-side certificate.

### Create client-side certificate
create CSR
```bash
openssl req \
-config <(cat opensslConf/csr_openssl.cnf opensslConf/client_san_names.cnf) \
-new \
-newkey rsa:2048 \
-nodes \
-days 365 \
-out CSRs/aClient.csr \
-keyout privateKeys/aClientKey.pem \
-reqexts client \
-subj "/emailAddress=client@somecompany.com/C=GB/ST=London/L=City of London/O=client Organization/OU=Engg/CN=client.full.domain.name"
```
Sign CSR with CA
```bash
openssl ca \
-config opensslConf/CA_openssl.cnf \
-policy policy_anything \
-cert caCert/myCA.crt \
-keyfile caCert/myCAkey.pem \
-in CSRs/aClient.csr \
-out newCreatedCerts/aClientCert.pem
```
### Create `apache` server-side certificate
If you have followed the structure of PKI mentioned in [**this link**]({{< ref "/posts/SSL-certificate.md" >}}), then create a san file for apache like following.
```bash
$ cat opensslConf/apache_san_names.cnf
[ san_names ]
DNS.1 = apache.server.name
```
create CSR

```bash
openssl req \
-config <(cat opensslConf/csr_openssl.cnf opensslConf/apache_san_names.cnf) \
-new \
-newkey rsa:2048 \
-nodes \
-days 365 \
-out CSRs/apacheServer.csr \
-keyout privateKeys/apacheServerKey.pem \
-reqexts server \
-subj "/emailAddress=apacheAdmin@admin.com/C=GB/ST=London/L=City of London/O=My Organization/OU=Engg/CN=apache.server.name"
```
Sign CSR with CA

```bash
openssl ca \
-config opensslConf/CA_openssl.cnf \
-policy policy_anything \
-cert caCert/myCA.crt \
-keyfile caCert/myCAkey.pem \
-in CSRs/apacheServer.csr \
-out newCreatedCerts/apacheServerCert.pem
```
### Create `apache` client-side certificate
create CSR
```bash
openssl req \
-config <(cat opensslConf/csr_openssl.cnf opensslConf/apache_san_names.cnf) \
-new \
-newkey rsa:2048 \
-nodes \
-days 365 \
-out CSRs/apacheAsClient.csr \
-keyout privateKeys/apacheAsClientKey.pem \
-reqexts client \
-subj "/emailAddress=apacheAdmin@admin.com/C=GB/ST=London/L=City of London/O=My Organization/OU=Engg/CN=apache.server.name"
```
Sign CSR with CA

```bash
openssl ca \
-config opensslConf/CA_openssl.cnf \
-policy policy_anything \
-cert caCert/myCA.crt \
-keyfile caCert/myCAkey.pem \
-in CSRs/apacheAsClient.csr \
-out newCreatedCerts/apacheAsClientCert.pem
```
### Create `gunicorn` server-side certificate
If you have followed the structure of PKI mentioned in [**this link**]({{< ref "/posts/SSL-certificate.md" >}}), then create a san file for gunicorn like following.

```bash
$ cat opensslConf/gunicorn_san_names.cnf

[ san_names ]
DNS.1 = gunicorn.app.server
```

```bash
$ openssl req \
-config <(cat opensslConf/csr_openssl.cnf opensslConf/gunicorn_san_names.cnf) \
-new \
-newkey rsa:2048 \
-nodes \
-days 365 \
-out CSRs/gunicornServer.csr \
-keyout privateKeys/gunicornServerKey.pem \
-reqexts server \
-subj "/emailAddress=apacheAdmin@admin.com/C=GB/ST=London/L=City of London/O=My Organization/OU=Engg/CN=gunicorn.app.server"
```
Sign CSR

```bash
$ openssl ca \
-config opensslConf/CA_openssl.cnf \
-policy policy_anything \
-cert caCert/myCA.crt \
-keyfile caCert/myCAkey.pem \
-in CSRs/gunicornServer.csr \
-out newCreatedCerts/gunicornCert.pem
```
In whole, following files will should be created.

```bash
$ ls -1 caCert newCreatedCerts privateKeys/
caCert:
myCA.crt
myCAkey.pem

newCreatedCerts:
12.pem
13.pem
14.pem
15.pem
aClientCert.pem
apacheAsClientCert.pem
apacheServerCert.pem
gunicornCert.pem

privateKeys/:
aClientKey.pem
apacheAsClientKey.pem
apacheServerKey.pem
gunicornServerKey.pem
```
**Note:** you will never need `myCAkey.pem` file except for signing CSRs. Also ignore pem files having numerals in their names. Each certification generation process creates a cert file and a corresponding numeral file. Contents of both the files are same. Hence can be ignored.

## Apache Preparation
Make sure `apache` is installed. In case of fedora apache is recognized as `httpd` rpm package.

Install apache `ssl` module.
```bash
$ sudo dnf -y install mod_ssl
```

### Create Certificate Directory
```bash
$ sudo mkdir /etc/httpd/sslCerts
$ sudo cp newCreatedCerts/apacheServerCert.pem newCreatedCerts/apacheAsClientCert.pem \
privateKeys/apacheServerKey.pem privateKeys/apacheAsClientKey.pem \
caCert/myCA.crt \
/etc/httpd/sslCerts/
```

### prepare certificate to be used by apache reverse proxy SSL engine.
```bash
$ openssl rsa -in apacheAsClientKey.pem > apacheAsClientKeyWithRSAHeader.pem
$ openssl x509 -in apacheAsClientCert.pem > apacheAsClientCertPEMformat.pem
$ cat apacheAsClientCertPEMformat.pem apacheAsClientKeyWithRSAHeader.pem > apacheAsClientCertAndKeyCombined.pem
$ chmod 400 apacheServerKey.pem apacheAsClientCertAndKeyCombined.pem
```
**Note:** We do NOT need `apacheAsClientKey.pem`, `apacheAsClientCert.pem`, `apacheAsClientKeyWithRSAHeader.pem` and `apacheAsClientCertPEMformat.pem` files and they can be deleted.

### Prepeare apache config file
Create a file `/etc/httpd/conf.d/reverseProxy.conf` with following contents.
```bash
$ cat /etc/httpd/conf.d/reverseProxy.conf

Listen 7771
<VirtualHost *:7771>
    ServerName apache.server.name
    # LogLevel debug

    SSLEngine On
    SSLCACertificateFile /etc/httpd/sslCerts/myCA.crt
    SSLCertificateFile /etc/httpd/sslCerts/apacheServerCert.pem
    SSLCertificateKeyFile /etc/httpd/sslCerts/apacheServerKey.pem
    SSLVerifyClient require
    SSLVerifyDepth 5

    SSLProxyEngine On
    SSLProxyVerify require
    SSLProxyVerifyDepth 5
    # SSLProxyCheckPeerCN Off
    SSLProxyCheckPeerName Off
    SSLProxyCACertificateFile /etc/httpd/sslCerts/myCA.crt
    SSLProxyMachineCertificateFile /etc/httpd/sslCerts/apacheAsClientCertAndKeyCombined.pem

    <Location "/">
        ProxyPreserveHost On
        ProxyPass "https://gunicorn.app.server:6662/"
        # SSLRequireSSL
        # SSLRequire %{SSL_CLIENT_S_DN_CN} eq "some.host.name"
        # SSLRequire %{SSL_CLIENT_SAN_DNS_0} eq "some.host.name"
        # SSLRequire %{REMOTE_HOST} in {"some.host.name", "another.host.name"}
    </Location>

</VirtualHost>
```

**Note**: You may need to inrease `SSLVerifyDepth` and `SSLProxyVerifyDepth` to higher number. This represents the depth of you certificate chain (Root and intermediary certs). Number need to be exact. It can be higher but no lower. Lower number will result in error.

### Start Apache
```bash
$ sudo systemctl restart  httpd
```
**Note:** You may need to tweak selinux policy for apache to run at 7771 port. I had disabled selinux for this excersize.

### Make client, apache and gunicorn addresses resolvable
As `root` user enter the following line in `/etc/resolv.conf` file above all `nameserver` lines.
```
nameserver 127.0.0.1
```

Run a temp DNS server in foreground.
```bash
$ sudo dnsmasq --no-daemon \
--listen-address=127.0.0.1 \
--address=/client.full.domain.name/127.0.0.1 \
--address=/apache.server.name/127.0.0.1 --address=/gunicorn.app.server/127.0.0.1  \
--log-queries
```

### Create a Simple flask app
```bash
mkdir flaskApp
mkdir flaskApp/certs
touch flaskApp/myFlaskApp.py
```

Contents of `flaskApp/certs` should be following
```bash
$ ls -og flaskApp/certs
total 16
-rw-rw-r--. 1 5021 Aug  4 17:37 gunicornCert.pem
-rw-------. 1 1704 Aug  4 17:37 gunicornServerKey.pem
-rw-rw-r--. 1 1493 Aug  4 17:37 myCA.crt
```

Contents of `myFlaskApp.py` are below.

```bash
$ cat myFlaskApp.py
from flask import Flask, jsonify
app = Flask(__name__)

api_url = "/"

@app.route(api_url, methods=["GET"])
def write_to_file():
    return jsonify({"success": "request received successfully"}), 201
```

start serving flask app using following command.

```bash
$ cd flaskApp/
$ gunicorn --bind 0.0.0.0:6662 myFlaskApp:app \
--ca-certs certs/myCA.crt \
--certfile certs/gunicornCert.pem \
--keyfile certs/gunicornServerKey.pem \
--cert-reqs 2

[2019-08-04 18:20:48 +0100] [37404] [INFO] Starting gunicorn 19.9.0
[2019-08-04 18:20:48 +0100] [37404] [INFO] Listening at: https://0.0.0.0:6662 (37404)
[2019-08-04 18:20:48 +0100] [37404] [INFO] Using worker: sync
[2019-08-04 18:20:48 +0100] [37407] [INFO] Booting worker with pid: 37407
```
**Note:** argument 2 for `--cert-reqs` corresponds to `ssl.CERT_REQUIRED`

### Create client environment
```bash
$ mkdir client
$ mkdir client/certs
```
contents of `client/certs` are below
```bash
$ ls -og client/certs/
total 16
-rw-rw-r--. 1 5062 Aug  4 17:40 aClientCert.pem
-rw-------. 1 1704 Aug  4 17:41 aClientKey.pem
-rw-rw-r--. 1 1493 Aug  4 17:41 myCA.crt
```

### Test the whole setup
#### Test `gunicorn` server connectivity directly
Gunicorn server connectivity.
```bash
$ curl https://gunicorn.app.server:6662 \
--cacert certs/myCA.crt \
--cert certs/aClientCert.pem \
--key certs/aClientKey.pem 

{"success":"request received successfully"}
```
#### Test `gunicorn` server connectivity directly (no certs, insecure)
```bash
$ curl https://gunicorn.app.server:6662 \
--cacert certs/myCA.crt \
--insecure

curl: (56) OpenSSL SSL_read: error:1409445C:SSL routines:ssl3_read_bytes:tlsv13 alert certificate required, errno 0
```

#### `insecure` connection attempt
insecure connection
```bash
$ cd client/
$ curl https://apache.server.name:7771/ --insecure

curl: (56) OpenSSL SSL_read: error:1409445C:SSL routines:ssl3_read_bytes:tlsv13 alert certificate required, errno 0
```
Above connection failed because server requires authentication. Client also need to provide its certificate.

#### Authenticated connection attempt.
```bash
$ curl https://apache.server.name:7771/ \
--cacert certs/myCA.crt \
--cert certs/aClientCert.pem \
--key certs/aClientKey.pem 

{"success":"request received successfully"}
```
**Note**: You do not need to generate so many certificates. You can generate certificate having both the extensions of `TLS Web Server Authentication` and `TLS client Authentication` and that certificate can be deployed at all places.