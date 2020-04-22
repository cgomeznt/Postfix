Postfix utilizando Gmail como relay
===================================

Antes debes recordar que para que esto funcione debes ir a la cuenta de Gmail que vas a utilizar como relay y debes configurar que permita la conexiòn de equipos inseguros.

Configuramos Postfix abrimos el archivo /etc/postfix/main.cf y agregamos las siguientes lineas al final::

relayhost = [smtp.gmail.com]:587
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_CAfile = /etc/ssl/certs/ca-bundle.crt
smtp_sasl_security_options = noanonymous
smtp_sasl_tls_security_options = noanonymous


Configurar Postfix SASL Credentials. Creamos el archivo /etc/postfix/sasl_passwd con las siguientes lineas::

	[smtp.gmail.com]:587 username:password


Debemos generar ahora el sasl_passwd.db::

	postmap /etc/postfix/sasl_passwd
	
El archivo debe estar con permisos restringidos::

	chown root:root /etc/postfix/sasl_passwd*
	chmod 640 /etc/postfix/sasl_passwd*
	
Listo, recargamos el Postfix::

	systemctl reload postfix
	
Pruebas del Relay::

	ls -la / | mail -s"prueba de envio con Gmail como Relay"
	
Esto es lo que debemos ver en el log::

	Apr 22 18:03:12 lab01 postfix/pickup[2396]: 245A68B3E69: uid=0 from=<root>
	Apr 22 18:03:12 lab01 postfix/cleanup[2404]: 245A68B3E69: message-id=<20200422220312.245A68B3E69@mail1.cursoinfraestructura.com.ve>
	Apr 22 18:03:12 lab01 postfix/qmgr[2397]: 245A68B3E69: from=<root@cursoinfraestructura.com.ve>, size=1586, nrcpt=1 (queue active)
	Apr 22 18:03:23 lab01 postfix/smtp[2406]: 245A68B3E69: to=<cgomeznt@gmail.com>, relay=smtp.gmail.com[172.217.203.108]:587, delay=12, delays=0.24/0.31/9/1.9, dsn=2.0.0, status=sent (250 2.0.0 OK  1587593003 d83sm207547vka.34 - gsmtp)
	Apr 22 18:03:23 lab01 postfix/qmgr[2397]: 245A68B3E69: removed

Y en la cuenta de Gmail llegara el correo diciendo que fue enviado por el usuario que configuramos en el sasl_passwd, en realidad esta tecnica no me gusta mucho.