Postfix SMTP AUTH SASL TLS Almalinux 8
=======================================

Instalar los paquetes requeridos::

  dnf install cyrus-sasl cyrus-sasl-devel cyrus-sasl-gssapi cyrus-sasl-md5 cyrus-sasl-plain postfix


Respaldar el archivo de configuración::

  cp /etc/postfix/main.cf /etc/postfix/main.cf_orig


Configurar el SMTP-AUTH y el TLS usando postconf::

  /usr/sbin/postconf -e 'smtpd_sasl_local_domain ='
  /usr/sbin/postconf -e 'smtpd_sasl_auth_enable = yes'
  /usr/sbin/postconf -e 'smtpd_sasl_security_options = noanonymous'
  /usr/sbin/postconf -e 'broken_sasl_auth_clients = yes'
  /usr/sbin/postconf -e 'smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination'
  /usr/sbin/postconf -e 'inet_interfaces = all'
  /usr/sbin/postconf -e 'mynetworks = 127.0.0.0/8, 10.50.1.0/24, 186.185.84.193/32'


Configurar postfix para permitir el LOGIN y los logins PLAIN::

  vi /usr/lib/sasl2/smtpd.conf (32-bit)
  vi /usr/lib64/sasl2/smtpd.conf (64-bit)

  pwcheck_method: saslauthd
  mech_list: plain login


Crear la Key para el SSL del request del certificado auto firmado::

  mkdir /etc/postfix/ssl
  cd /etc/postfix/ssl/
  openssl genrsa -des3 -rand /etc/hosts -out smtpd.key 1024
  chmod 600 smtpd.key


Crear el request auto firmado con la Key::

  openssl req -new -key smtpd.key -out smtpd.csr


Crear el SSL certificado con el request y la Key::

  openssl x509 -req -days 3650 -in smtpd.csr -signkey smtpd.key -out smtpd.crt


Crar RSA key::

  openssl rsa -in smtpd.key -out smtpd.key.unencrypted
  mv smtpd.key.unencrypted smtpd.key


Crear la CA key y el cert::

  openssl req -new -x509 -extensions v3_ca -keyout cakey.pem -out cacert.pem -days 3650


Configurar postfix para TLS::

  /usr/sbin/postconf -e 'smtpd_tls_auth_only = no'
  /usr/sbin/postconf -e 'smtp_use_tls = yes'
  /usr/sbin/postconf -e 'smtpd_use_tls = yes'
  /usr/sbin/postconf -e 'smtp_tls_note_starttls_offer = yes'
  /usr/sbin/postconf -e 'smtpd_tls_key_file = /etc/postfix/ssl/smtpd.key'
  /usr/sbin/postconf -e 'smtpd_tls_cert_file = /etc/postfix/ssl/smtpd.crt'
  /usr/sbin/postconf -e 'smtpd_tls_CAfile = /etc/postfix/ssl/cacert.pem'
  /usr/sbin/postconf -e 'smtpd_tls_loglevel = 1'
  /usr/sbin/postconf -e 'smtpd_tls_received_header = yes'
  /usr/sbin/postconf -e 'smtpd_tls_session_cache_timeout = 3600s'
  /usr/sbin/postconf -e 'tls_random_source = dev:/dev/urandom'


Configurar hostname y mydomain en postfix::

  /usr/sbin/postconf -e 'myhostname = vsv01.atbnet.local'
  /usr/sbin/postconf -e 'mydomain = atbnet.local'


Verificamos la configuracion del postfix config::

  more /etc/postfix/main.cf


Crear el  DNS::

  smtp IN A 10.50.1.50


Reiniciar los servicios de postfix, saslauthd
systemctl stop sendmail
systemctl start postfix
systemctl start saslauthd

Vemos el LOG errors/failures::

  tail /var/log/maillog
  ....
  Mar 10 04:21:55 vsv01 sendmail[6074]: alias database /etc/aliases rebuilt by andy
  Mar 10 04:21:55 vsv01 sendmail[6074]: /etc/aliases: 76 aliases, longest 10 bytes, 765 bytes total
  Mar 10 04:21:55 vsv01 postfix/postfix-script: starting the Postfix mail system
  Mar 10 04:21:55 vsv01 postfix/master[6120]: daemon started -- version 2.3.3, configuration /etc/postfix
  ....


Realizamos el Test con postfix iniciado, para validar que este trabajando las peticiones SMTP-AUTH/TLS::

  # telnet e-deus.cf 25
  Trying 190.114.9.23...
  Connected to e-deus.cf.
  Escape character is '^]'.
  ehlo 220 mail.e-deus.cf ESMTP No me jodas
  server
  250-mail.e-deus.cf
  250-PIPELINING
  250-SIZE 10485760
  250-ETRN
  250-STARTTLS
  250-AUTH PLAIN LOGIN
  250-AUTH=PLAIN LOGIN
  250-ENHANCEDSTATUSCODES
  250-8BITMIME
  250-DSN
  250 SMTPUTF8
  mail from:<cgomeznt@e-deus.cf>
  250 2.1.0 Ok
  rcpt to:<cgomeznt@gmail.com>
  504 5.5.2 <server>: Helo command rejected: need fully-qualified hostname
  quit
  221 2.0.0 Bye


  Si muestra los siguiente quiere decir que el TLS y el PLAIN/LOGIN logins estan configurados::
  
    250-STARTTLS
    250-AUTH PLAIN LOGIN
    
  Si nos muestra el 504 5.5.2 es evidencia que no permite conexión porque no se esta autenticando o porque no esta en la red de mynetworks



Verificando la Autenticación
+++++++++++++++++++++++++++++++++

Creamos una cuenta::

  useradd cgomeznt
  echo Betania | passwd --stdin cgomeznt 


Encode Plain Text to Base64::

  $ perl -MMIME::Base64 -e 'print encode_base64("cgomeznt");'
  dnNhbmNoZXo=
  $ perl -MMIME::Base64 -e 'print encode_base64("Betania");'
  QmV0YW5pYTIx


Verificamos que este trabajando la autenticación::

  # telnet e-deus.cf 25
  Trying 190.114.9.23...
  Connected to e-deus.cf.
  Escape character is '^]'.
  220 mail.e-deus.cf ESMTP No me jodas
  ehlo server
  250-mail.e-deus.cf
  250-PIPELINING
  250-SIZE 10485760
  250-ETRN
  250-STARTTLS
  250-AUTH PLAIN LOGIN
  250-AUTH=PLAIN LOGIN
  250-ENHANCEDSTATUSCODES
  250-8BITMIME
  250-DSN
  250 SMTPUTF8
  AUTH LOGIN
  334 VXNlcm5hbWU6
  dnNhbmNoZXo=
  334 UGFzc3dvcmQ6
  QmV0YW5pYTIx
  235 2.7.0 Authentication successful
  mail from:<cgomeznt@e-deus.cf>
  250 2.1.0 Ok
  rcpt to:<cgomeznt@gmail.com>
  250 2.1.0 Ok
  data
  subject:Postfix con AUTH y TLS
  250 2.1.5 Ok
  354 End data with <CR><LF>.<CR><LF>

  Buenas. Cuerpo del correo
  .
  250 2.0.0 Ok: queued as 1EA311400CA




https://www.vmadmin.co.uk/linux/44-redhat/146-linuxpostfix


https://sysadmins.co.za/setup-smtp-authentication-with-tls-ssl-on-postfix/
