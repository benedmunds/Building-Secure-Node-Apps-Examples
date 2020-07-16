# Chapter Two - HTTPS/SSL/BCA/JWH/SHA and Other Random Letters; Some of Them Actually Matter.

## Implementing HTTPS

### Apache Setup
```
<VirtualHost *:443>
  DocumentRoot "/path/to/your/app/htdocs"
  ServerName yourApp.com
  SSLEngine on
  SSLCertificateFile /usr/bin/ssl/yourAppSigned.crt
  SSLCertificateKeyFile /usr/bin/ssl/yourApp.key
</VirtualHost>
```

### Nginx Setup

```
server {

  listen   443;

  server_name yourApp.com;
  location / {
    root   /path/to/your/app/htdocs;
  }

  ssl    				  on;
  ssl_certificate 	  /usr/bin/ssl/yourAppSigned.crt
  ssl_certificate_key   /usr/bin/ssl/yourApp.key

}
```