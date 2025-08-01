# OpenSSL Setup
## Introduction
Now that the https protocol and encrypted connection have become widely democratized on the web, it is crucial to have a reliable model to use for development before going into production. This tutorial aims to set up a self-signed SSL certificate using OpenSSL and Apache on Wamp for Windows (for other systems such as Linux this tutorial is to be adapted).

This steps permit to setup multiple SSL certificates for multiple virtual hosts, which allows to work on multiple projects simultaneously with an encrypted protocol.

Here after we give an example to setup ssl for a virtual host called "forum.prog" (it is possible to setup multiple virtual hosts in the same time).

This requires WampServer > 3, and installation of OpenSSL light as administrator on your system, and then activate OpenSSL module in Apache files.
To download and install the lastest executable for Windows (Win64 OpenSSL v3.5.1 Light): https://slproweb.com/products/Win32OpenSSL.html
## Creation of your virtual hosts, activation of SSL, adding Apache directives to ssl vhost file
* First, create your virtual host(s) for your different projects by using WampServer tool (localhost -> Tools -> Add a virtual host), restart Wamp services, restart DNS

Normally, Wamp should automatically update apache as well as windows HOST files

* Then activate the OpenSSL module in apache by uncommenting following lines in the file c:\wamp64\bin\apache\apache\conf\httpd.conf:

`LoadModule socache_shmcb_module modules/mod_socache_shmcb.so`

`LoadModule ssl_module modules/mod_ssl.so`

`Include conf/extra/httpd-ssl.conf`

* Add the following directive to c:\wamp64\bin\apache\apache\conf\extras\httpd-ssl.conf (add one directive per virtual host):

<VirtualHost *:443>
    ServerName forum.prog
    ServerAlias forum.prog
    ServerAdmin admin@forum.prog
    DocumentRoot "c:/wamp64/www/forum"

    <Directory "c:/wamp64/www/forum/">
        SSLOptions +StdEnvVars
        Options +Indexes +Includes +FollowSymLinks +MultiViews
        AllowOverride All
        Require local
        Require ip 192.168
    </Directory>

    SSLEngine on

    SSLCertificateFile "${INSTALL_DIR}/bin/certs/forum.prog/server.crt"
    SSLCertificateKeyFile "${INSTALL_DIR}/bin/certs/forum.prog/server.key"

    LogFormat "%L [%{%a, %d-%b-%g %T}t %{%z}t] %H %m \"%U%q\" (%b bytes) %>s" access
    CustomLog "${INSTALL_DIR}/bin/logs/forum.prog/access.log" access

    ErrorLogFormat "%L [%t] [%-m:%l] [pid %P:tid %T] %E: %a %M"
    ErrorLog "${INSTALL_DIR}/bin/logs/forum.prog/error.log"

    LogFormat "%L [%{%a, %d-%b-%g %T}t %{%z}t] %H %{SSL_PROTOCOL}x %{SSL_CIPHER}x %m \"%U%q\" (%b bytes) %>s" ssl
    CustomLog "${INSTALL_DIR}/bin/logs/forum.prog/ssl_request.log" ssl

    SSLProtocol all -SSLv3 -TLSv1 -TLSv1.1
    Protocols h2 http/1.1
</VirtualHost>

## Creation of self-signed SSL certificates
* Create the following directory: c:\wamp64\bin\certs\forum.prog (create one directory per virtual host)

* Open cmd as administrator and enter following commands:

* pointing the dir of installation of OpenSSL on your computer (edit if necessary)
`cd C:\Program Files\OpenSSL-Win64\bin`

* creating a dir for your website to certificate
`md forum.prog`

* Create the server key file
`openssl genrsa -out forum.prog\server.key 2048`

* Generate the Certificate Signing Request (CSR) file
`openssl req -new -key forum.prog\server.key -out forum.prog\server.csr`

* Generate the Certificate Signing Request (CSR) file ONLY IF YOU USE CONFIG FILE (no answers to give then, see the openssl.cnf file):
`openssl req -new -key forum.prog\server.key -out forum.prog\server.csr -config forum.prog\openssl.cnf`

* Enter the following answers and validate each question by Enter (if no config file given previously):

Country: FR
State/Province name: France
City: Paris
Organization: Business
Organizational Unit Name: IT Department
Common name: forum.prog (enter the domain name of the virtual host you want to certificate here)
Email Address: webmaster@forum.prog
Let a blank answer for the "Challenge password" and "Optional company name"

* Finally, self-sign your certificate for a duration of 3650 days (can be replaced by another number)
openssl x509 -req -days 3650 -in forum.prog\server.csr -signkey forum.prog\server.key -out forum.prog\server.crt

* Move your signed certificate to the directory: c:\wamp64\bin\certs\forum.prog

* Import your self signed certificate to the Windows Trusted Certificates Store (Win+R > certmgr.msc to see certificates)
certutil -f -addstore "root" "c:\wamp64\bin\certs\forum.prog\server.crt"

* Repeat these steps for all your virtual hosts

## Finalization
* Add the domain of your virtual hosts to the host file of Windows (C:\windows\system32\drivers\etc\hosts) (it is done automatically in Wamp3 virtual host utility, but you can check it)
* Then launch Cmd in admin and type following command: ipconfig /flushdns
* Restart all services for changes to take in account (click on Wampmanager Tray Icon -> Restart Services) and Restart DNS (right click on Tray Icon -> Tools -> Restart DNS)
