Instalar Postfix solo para el envio SMTP CentOS7
=================================================

Cómo configurar Postfix como un servidor SMTP de solo envío en CentOS 7. Primero le mostraré cómo hacerlo para un solo dominio, luego puede aplicar los pasos para múltiples dominios si lo necesita para.

Caso de uso
+++++++++++++

Tiene un sitio web/aplicación web que necesita enviar correos electrónicos transaccionales a los usuarios (como correo electrónico de restablecimiento de contraseña) o para un proceso informativo de batch. Lo más probable es que los usuarios no tengan que responder a estos correos electrónicos o, si responden, los correos electrónicos de respuesta se enviarán a su servidor de correo dedicado. En este caso, puede configurar un servidor SMTP de solo envío en el servidor web utilizando Postfix, que es un software de servidor SMTP popular.

Prerrequisitos
Para enviar correos electrónicos desde su servidor, el puerto 25 (saliente) debe estar abierto. Muchos ISP y empresas de alojamiento, como DigitalOcean, bloquean el puerto 25 para controlar el spam. Aunque Hostwinds no bloquea el puerto 25 (saliente).

Primero, necesitamos configurarlo para un dominio, luego configurarlo para múltiples dominios si así lo necesitamos.

Paso 1: Establecer el nombre de host y el registro PTR y SPF
+++++++++++++++++++++++++++++

Adquirimos un dominio publico
++++++++++++++++++++++++++++++

En https://my.freenom.com adquirimos el Dominio y en realizamos las siguientes configuraciones, en este ejemplo el dominio se llama **e-deus.cf**:

+-----------------------------------------------------------------------------+
|**Records**						      		      | 
+------------------+----+-------+---------------------------------------------+
|Name	           |Type|TTL	|Target					      | 	
+------------------+----+-------+---------------------------------------------+
|.		   |A	|300	|190.114.9.23	 	      	    	      |
+------------------+----+-------+---------------------------------------------+
|MAIL		   |A	|3600	|190.114.9.23		  		      |
+------------------+----+-------+---------------------------------------------+
|WWW		   |A	|300	|190.114.9.23		      		      |
+------------------+----+-------+---------------------------------------------+
|.		   |MX	|3600	|mail.e-deus.cf	Priority:5	       	      |
+------------------+----+-------+---------------------------------------------+
|.		   |TXT	|3600	|v=spf1 a:mail.e-deus.cf ip4:190.114.9.23 ~all|
+------------------+----+-------+---------------------------------------------+
|_dmarc.e-deus.cf. |TXT	|3600	|v=DMARC1; p=none; rua=mailto:admin@e-deus.cf |
+------------------+----+-------+---------------------------------------------+

De forma predeterminada, Postfix utiliza el nombre de host de su servidor para identificarse cuando se comunica con otros servidores SMTP. Algunos servidores SMTP rechazarán su correo electrónico si su nombre de host no es válido. Debe establecer un nombre de dominio completo (FQDN) como se muestra a continuación.::

	hostnamectl set-hostname mail.e-deus.cf

Verificamos el hostname del servidor::

	hostname -f

Debe cerrar la sesión y volver a iniciarla para ver el cambio de nombre de host en el símbolo del sistema. Este nombre de host debe tener un registro DNS A que apunte a la dirección IP de su servidor.

Establecer un registro PTR, que asigna una dirección IP a un FQDN. Muchos servidores SMTP rechazarán correo electrónico si la dirección IP de su servidor no tiene registro PTR.

Para ver si su registro PTR está configurado correctamente, ejecute el siguiente comando. ::

	host 190.198.55.89

	dig -x 190.198.55.89

Este es un ejemplo del SPF que se debe publicar en los registros de DNS Publico::

	  @            IN   TXT    "v=spf1 a:e-deus.cf ip4:190.36.229.66/23 -all"

Configuración de un `SplitDNS <https://github.com/cgomeznt/Zimbra/blob/main/guia/SplitDNS.rst>`_.(Recomendado) 

Paso 2: Instale Postfix en CentOS 7
+++++++++++++++++++++

Instalar Postfix desde el repositorio predeterminado de CentOS 7::

ntes de comenzar, también puede actualizar sus conceptos sobre cómo funciona el correo electrónico con Postfix como referencia. Esto te ayudaría a llegar más lejos con este contenido.

Verificar y remover el Sendmail (opcional)::

	# rpm -qa | grep sendmail

Si el Sendmail esta instalado los removemos::

	# yum remove sendmail*
	
Instalar Postfix::

	# yum install postfix mailx cyrus-sasl-plain

Paso 3: configurar Postfix
++++++++++++++++

**Configuración del nombre de host de Postfix**
De forma predeterminada, el servidor Postfix SMTP utiliza el nombre de host del sistema operativo para identificarse cuando se comunica con otro servidor SMTP. Sin embargo, el nombre de host del sistema operativo puede cambiar, por lo que es una buena práctica establecer el nombre de host directamente en el archivo de configuración de Postfix con el siguiente comando.

	postconf -e "myhostname = mail.yourdomain.com"

**Configuración del parámetro $mydomain**
El parámetro $mydomain especifica el nombre de dominio de Internet local. El valor predeterminado es usar $myhostname menos el primer componente. Puede mostrar el valor actual de $mydomain con:

	postconf mydomain

Debe ser su nombre de dominio principal, como::

	e-deus.cf

Si no muestra su nombre de dominio ápice, configure el parámetro $ mydomain con::

	postconf -e "mydomain = e-deus.cf"

**Configuración del parámetro $ myorigin**
El parámetro $myorigin especifica el nombre de dominio predeterminado que se agrega a las direcciones del remitente y del destinatario que no tienen una parte @domain. El valor predeterminado es usar el valor de $myhostname, como se puede ver con::


	postconf myorigin

La salida sera::

	myorigin = $mydomain


Puede cambiar su valor a e-deus.cf::

	sudo postconf -e "myorigin = e-deus.cf"

Consultamos si esta atendiendo por todas las interfaz::

	postconf inet_interfaces

Aseguramos que solo pueda atender por la inet lo::

	postconf -e "inet_interfaces = loopback-only"

Si queremos agregar las IP que tiene que tener permisos en la variable $mynetworks::

	# postconf -e "mynetworks = 127.0.0.0/8, 190.203.180.247/32, 190.120.248.40/32, 190.114.9.23/32"

**Reiniciar Postfix**
Finalmente, necesitamos reiniciar Postfix para que los cambios surtan efecto.::

	systemctl restart postfix

Para ver los LOGs::

	# tail -f /var/log/maillog 


Ha instalado y configurado correctamente Postfix como un servidor MTA de solo envío. Para probar la entrega de correo electrónico, use el comando de correo como se muestra a continuación::

	echo "Postfix Send-Only Server" | mail -s "Postfix Testing" cgomez@e-deus.cf
	
Esta prueba me gusta::

	# echo "Postfix Send-Only Server" | mailx -v -s "Postfix Probando" -r "cgomeznt@e-deus.cf"  -S smtp="mail.e-deus.cf:25" carlos.gomez@credicard.com.ve
	Resolving host mail.e-deus.cf . . . done.
	Connecting to 190.114.9.23:25 . . . connected.
	220 c946.gconex.com ESMTP Postfix
	>>> HELO c946.gconex.com
	250 c946.gconex.com
	>>> MAIL FROM:<cgomeznt@e-deus.cf>
	250 2.1.0 Ok
	>>> RCPT TO:<carlos.gomez@credicard.com.ve>
	250 2.1.5 Ok
	>>> DATA
	354 End data with <CR><LF>.<CR><LF>
	>>> .
	250 2.0.0 Ok: queued as 6F3C0140093
	>>> QUIT
	221 2.0.0 Bye

Con telnet es bien::

	➤ telnet e-deus.cf 25
	Trying 190.114.9.23...
	Connected to e-deus.cf.
	Escape character is '^]'.
	220 c946.gconex.com ESMTP Postfix
	ehlos server
	502 5.5.2 Error: command not recognized
	ehlo server
	250-c946.gconex.com
	250-PIPELINING
	250-SIZE 10240000
	250-VRFY
	250-ETRN
	250-STARTTLS
	250-ENHANCEDSTATUSCODES
	250-8BITMIME
	250-DSN
	250 SMTPUTF8
	mail from:cgomeznt@e-deus.cf
	250 2.1.0 Ok
	rcpt to:carlos.gomez@credicard.com.ve
	250 2.1.5 Ok
	data
	354 End data with <CR><LF>.<CR><LF>
	subject: Esto es una prueba de un Postfix

	Buenas.

	Por favor omitir este correo de prueba
	.
	250 2.0.0 Ok: queued as A4ABD140122
	quit


También puede cargar datos existentes al correo::

	mail -s "Mail Subject" cgomez@e-deus.cf < /home/jmutai/file.txt

En el log deberá ver algo como esto::

	Mar 31 19:03:39 debian postfix/pickup[28648]: 9B37C46CD6: uid=0 from=<root@debian.example.local>
	Mar 31 19:03:39 debian postfix/cleanup[28700]: 9B37C46CD6: message-id=<20210331230339.9B37C46CD6@mail.e-deus.cf>
	Mar 31 19:03:39 debian postfix/qmgr[28649]: 9B37C46CD6: from=<root@debian.example.local>, size=378, nrcpt=1 (queue active)
	Mar 31 19:03:39 debian postfix/local[28721]: 9B37C46CD6: to=<cgomez@e-deus.cf>, relay=local, delay=0.13, delays=0.06/0.01/0/0.06, dsn=2.0.0, status=sent (delivered to maildir)
	Mar 31 19:03:39 debian postfix/qmgr[28649]: 9B37C46CD6: removed

Consultamos el Maildir del usuario::

	ls -ltr /home/cgomez/Maildir/new/
	total 16
	-rw------- 1 cgomez cgomez  472 mar 31 19:03 1617231819.Vfe02I17612bfM673812.debian

Leemos el correo::

	cat  /home/cgomez/Maildir/new/1617231819.Vfe02I17612bfM673812.debian
	Return-Path: <root@debian.example.local>
	X-Original-To: cgomez@e-deus.cf
	Delivered-To: cgomez@e-deus.cf
	Received: by mail.e-deus.cf (Postfix, from userid 0)
		id 9B37C46CD6; Wed, 31 Mar 2021 19:03:39 -0400 (-04)
	Subject: This is the subject line
	To: <cgomez@e-deus.cf>
	X-Mailer: mail (GNU Mailutils 3.5)
	Message-Id: <20210331230339.9B37C46CD6@mail.e-deus.cf>
	Date: Wed, 31 Mar 2021 19:03:39 -0400 (-04)
	From: root <root@debian.example.local>

	This is the body of the email

Esta configuración, la dirección en el campo **FROM** para los correos electrónicos será yourusername@mail.e-deus.cf, donde yourusername es su nombre de usuario de Linux y mail.e-deus.cf es el dominio configurado en el nombre de host de su servidor. Si cambia su nombre de usuario, la dirección **FROM** también cambiará.

Para hacer las pruebas a los dominios externos, hacer lo mismo y colocar la rutas validas, ejemplo, cgomez@gmail, cgomez@yahoo.

No olvidemos para que pueda ser aceptado por los dominios externos el envío de email, debemos cumplir con las convenciones de correo, como tener un DNS el registro MX y su tipo A, el PTR, tener un SPF, tener una IP estática, no estar en listas negras, etc...etc.
