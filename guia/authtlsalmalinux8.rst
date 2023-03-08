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

