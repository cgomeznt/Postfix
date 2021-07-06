Instalar Postfix CentOS7
=========================

Introducción
+++++++++++++++

Postfix es uno de los Mail Transfer Agent (MTA) de código abierto más populares que enruta y entrega correos. Es una alternativa a Sendmail MTA que viene preinstalada en todas las versiones anteriores a Centos/RHEL 5. La instalación de CentOS Postfix es un proceso que requiere mucha precisión.

Veamos la definición de Wikipedia de Postfix, que dice:

"Postfix es un agente de transferencia de correo gratuito y de código abierto que enruta y entrega el correo electrónico. Se publica bajo la Licencia pública de IBM 1.0, que es una licencia de software libre. Alternativamente, a partir de la versión 3.2.5, está disponible bajo el Eclipse Licencia pública 2.0 a elección del usuario ". - Wikipedia

El trabajo principal del postfix (CentOS) es retransmitir correos localmente o a un servidor de destino fuera de la red. Para instalar postfix y evitar conflictos, debe eliminar sendmail si ya está instalado.

Antes de comenzar, también puede actualizar sus conceptos sobre cómo funciona el correo electrónico con Postfix como referencia. Esto te ayudaría a llegar más lejos con este contenido.

Configuración de un `SplitDNS <https://github.com/cgomeznt/Zimbra/blob/main/guia/SplitDNS.rst>`_.(Opcional) 

Verificar y remover el Sendmail (opcional)::

	# rpm -qa | grep sendmail

Si el Sendmail esta instalado los removemos::

	# yum remove sendmail*
	
Instalar Postfix::

	# yum install -y postfix mailx cyrus-sasl-plain

Verificamos nuestro archivo hosts::

	# cat /etc/hosts
		127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
		192.168.1.20    lab01.dominio.local lab01 dominio.local
		190.36.229.66   mail1.cursoinfraestructura.com.ve cursoinfraestructura.com.ve
		::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

La IP 192.168.1.20  es la asignada a la unica interfaz del servidor y la IP 190.36.229.66 es la WAN asociada al dominio cursoinfraestructura.com.ve, es decir, previo tienes configurado un DNS Publico.


Configurar Postfix, editamos el archivo /etc/postfix/main.cf::

	# vi /etc/postfix/main.cf
		...
		myhostname = smtp.cursoinfraestructura.com.ve
		...
		mydomain = cursoinfraestructura.com.ve
		...
		myorigin = $mydomain
		...
		inet_interfaces = all
		...
		inet_protocols = all
		...
		mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
		...
		mynetworks = $config_directory/mynetworks
		...
		home_mailbox = Maildir/
		...

Activamos las ip que estarán autorizadas a usar el smtp para envió. creamos el archivo /etc/postfix/mynetworks::

	# vi /etc/postfix/mynetworks
		# localhost
		127.0.0.0/8

		#cursoinfraestructura.com.ve
		192.168.1.0/24
		190.36.229.66
		192.168.43.16

Agregamos nuestra ip a access::

	# vi access
		192.168.1.20    OK

Activamos el cambio del archivo access, en realidad crea un archivo access.db::

	# postmap /etc/postfix/access
	

Habilitamos el servicio del postfix y lo iniciamos::

	# systemctl enable postfix
	# systemctl restart postfix
	# systemctl status postfix 

Para ver los LOGs::

	# tail -f /var/log/maillog

Realizamo un test de envio local::

	# useradd postfixtester
	# passwd postfixtester

Hacemos el test con telnet::

	# telnet localhost smtp
	Trying ::1...
	telnet: connect to address ::1: Connection refused
	Trying 127.0.0.1...
	Connected to localhost.
	Escape character is '^]'.
	220 mail1.cursoinfraestructura.com.ve ESMTP Postfix
	ehlo esteserver
	250-mail1.cursoinfraestructura.com.ve
	250-PIPELINING
	250-SIZE 10240000
	250-VRFY
	250-ETRN
	250-ENHANCEDSTATUSCODES
	250-8BITMIME
	250 DSN
	mail from:postfixtester
	250 2.1.0 Ok
	rcpt to:postfixtester
	250 2.1.5 Ok
	data
	354 End data with <CR><LF>.<CR><LF>
	Subject:Test de email local
	Buenas, esto es una prueba
	.
	250 2.0.0 Ok: queued as A5B448B3E68
	quit
	221 2.0.0 Bye
	Connection closed by foreign host.

En el log veremos algo como esto::

	Apr 22 16:18:11 lab01 postfix/smtpd[2011]: connect from localhost[127.0.0.1]
	Apr 22 16:18:44 lab01 postfix/smtpd[2011]: A5B448B3E68: client=localhost[127.0.0.1]
	Apr 22 16:19:14 lab01 postfix/cleanup[2017]: A5B448B3E68: message-id=<20200422201844.A5B448B3E68@mail1.cursoinfraestructura.com.ve>
	Apr 22 16:19:14 lab01 postfix/qmgr[1547]: A5B448B3E68: from=<postfixtester@cursoinfraestructura.com.ve>, size=416, nrcpt=1 (queue active)
	Apr 22 16:19:14 lab01 postfix/local[2020]: A5B448B3E68: to=<postfixtester@cursoinfraestructura.com.ve>, orig_to=<postfixtester>, relay=local, delay=37, delays=37/0.08/0/0.05, dsn=2.0.0, status=sent (delivered to maildir)
	Apr 22 16:19:14 lab01 postfix/qmgr[1547]: A5B448B3E68: removed
	Apr 22 16:19:18 lab01 postfix/smtpd[2011]: disconnect from localhost[127.0.0.1]

Teniendo instalado mailx, esta es otra forma rapida de hacerlo::

	# ls -la / | mail -s"prueba de envio" postfixtester
	
Ahora para ver los e-mail enviados nos vamos al homedirectory del usuario::

	# ls /home/postfixtester/Maildir/new/

	1587586754.Vfd00Ic845ceM218117.lab01.dominio.local
	1587587237.Vfd00Ic845cfM672412.lab01.dominio.local
	
	# cat /home/postfixtester/Maildir/new/1587586754.Vfd00Ic845ceM218117.lab01.dominio.local
	Return-Path: <postfixtester@cursoinfraestructura.com.ve>
	X-Original-To: postfixtester
	Delivered-To: postfixtester@cursoinfraestructura.com.ve
	Received: from esteserver (localhost [127.0.0.1])
			by mail1.cursoinfraestructura.com.ve (Postfix) with ESMTP id A5B448B3E68
			for <postfixtester>; Wed, 22 Apr 2020 16:18:37 -0400 (-04)
	Subject:Test de email local
	Message-Id: <20200422201844.A5B448B3E68@mail1.cursoinfraestructura.com.ve>
	Date: Wed, 22 Apr 2020 16:18:37 -0400 (-04)
	From: postfixtester@cursoinfraestructura.com.ve

	Buenas, esto es una prueba

Nos conectamos a Gmail o Yahoo, para enviar correos de prueba a nuestro dominio. Hacemos el mismo troubleshooting anterior.

Para hacer las pruebas a los dominios externos, hacer lo mismo y colocar la rutas validas, ejemplo, cgomez@gmail, cgomez@yahoo.

No olvidemos para que pueda ser aceptado por los dominios externos el envío de email, debemos cumplir con las convenciones de correo, como tener un DNS el registro MX y su tipo A, el PTR, tener un SPF, tener una IP estática, no estar en listas negras, etc...etc.
