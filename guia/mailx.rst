Instalaer Mailx para hacer pruebas con Postfix
================================================


Enviar emails con mailx y Gmail en CentOS
+++++++++++++++++++++++++++++++++++++++

Mailx necesita un servidor local SMTP (MTA : Mail Transfer Agent) para poder enviar los mails.
Lo primero actualizamos el sistema::

	yum -y update
 
instalamos mailx::

	yum install -y mailx

Creamos un enlace simbólico::

	ln -s /bin/mailx /bin/email
 
Ahora tenemos que configurar el servidor SMTP para realizar envíos, y usar el de Google::

	vi /etc/mail.rc

Editamos el fichero de configuración para enviar emails con nuestra cuenta de gmail, y lo configuramos así::

	set smtp=smtps://smtp.gmail.com:465
	set smtp-auth=login
	set smtp-auth-user=usuario@gmail.com
	 
	set smtp-auth-password=PASSWORD
	set ssl-verify=ignore
	set nss-config-dir=/etc/pki/nssdb/

Y ahora enviamos un email de esta forma por medio de la terminal::

	echo "cuerpo del mail" | mail -v -s "Message Subject" destinatario@correo
 

Para omitir los \n o salto de lineas y evitar que mail envie el contenido en un adjunto::

	tr -cd "[:print:]\n" < $BODY |  mail -s "$SUBJECT" $email
	
Si solo queremos enviar correos interno solo debemos editar el archivo de configuración /etc/mail.rc y agregar::

	set smtp=mail.e-deus.cf
	set from="admin@e-deus.cf"
	
	
	
Este es un ejemplo de un Script
==============================
::

  FECHA=$(date +%d-%m-%Y_%H:%M)
  EMAILS="soporte.aplicaciones@e-deus.cf Monitoreo1@e-deus.cf monitoreo@e-deus.cf  Monitorcc@e-deus.cf Andres.Pineda@e-deus.cf Edgar.Duran@e-deus.cf"
  SUBJECT="Reporte Estatus plataforma Soporte Web $FECHA"
  BODY="/usr/local/bin/BODY_EMAIL.TXT"
  ATTACH="/usr/local/bin/Check-List_aplicaciones.xlsx"


  for email in $(echo $EMAILS)
  do
    tr -cd "[:print:]\n" < $BODY |  mail -r soporte.aplicaciones@e-deus.cf -a $ATTACH -v -s "$SUBJECT" $email
  done


Y este es el archivo BODY_EMAIL.TXT
======================================
::

  Buenas.


          Se anexa archivo en el correo con el Check List de la verificacion de los Servicios Administrados por Soporte WEB.




          Muchas gracias por su atencion

  Atte.
  Vicepresidencia de Plataforma y Servicios Tecnologicos.
  Gerencia de Soporte Plataforma.
  Coordinacion Soporte Web.
  Especialista TI Soporte Web
  Telf.: 58 (0212)9554207
  Soporte Aplicaciones <Soporte.Aplicaciones@e-deus.cf>


Este es otro ejemplo de script 
=================================

En este caso se requiere de un HEADER y de un FOODER para ir armando el BODY::

  #!/bin/bash

  BODY="/CDC/scripts/bin/.BODY_EMAIL.TXT"
  HEADER="/CDC/scripts/bin/.HEADER_EMAIL.TXT"
  FOODER="/CDC/scripts/bin/.FOODER_EMAIL.TXT"

  function send_mail {

          FECHA=$(date +%d-%m-%Y_%H:%M)
          EMAILS="soporte.aplicaciones@e-deus.cf Monitoreo1@e-deus.cf monitoreo@e-deus.cf  Monitorcc@e-deus.cf Andres.Pineda@e-deus.cf Edgar.Duran@e-deus.cf Servicios.Web@e-deus.cf Jean.Bautista@e-deus.cf Rafael.Barreta@e-deus.cf Soporte.BD@e-deus.cf Soporte.AS400@e-deus.cf Avedis.Khajikian@e-deus.cf"
          SUBJECT="Estado de Salud de la Replica IBM CDC para SMI $FECHA"
          #ATTACH="/usr/local/bin/Check-List_aplicaciones.xlsx"


          for email in $(echo $EMAILS)
          do
                  #tr -cd "[:print:]\n" < $BODY |  mail -r soporte.aplicaciones@e-deus.cf -a $ATTACH -v -s "$SUBJECT" $email
                  tr -cd "[:print:]\n" < $BODY |  mail -r Replica.CDC.SMI@e-deus.cf -v -s "$SUBJECT" $email
          done
  }

  cat $HEADER > $BODY
  /CDC/scripts/bin/status_table_parked.sh S | grep -v Repl >> $BODY
  cat $FOODER >> $BODY

  send_mail
  echo > $BODY

  exit 0
  
  
  
En Debian
===========

Instalamos ::

	apt-get install mailutils bsd-mailx

Configuramos el archivo mail.rc::

	# cat /etc/mail.rc
	set ask askcc append dot save crt
	ignore Received Message-Id Resent-Message-Id Status Mail-From Return-Path Via Delivered-To
	set hold
	set append
	set ask
	set crt
	set dot
	set keep
	set emptybox
	set indentprefix="> "
	set quote
	set sendcharsets=iso-8859-1,utf-8
	set showname
	set showto
	set newmail=nopoll
	set autocollapse
	set markanswered
	ignore received in-reply-to message-id references
	ignore mime-version content-transfer-encoding
	fwdretain subject date from to


	set smtp=e-deus.cf
	set from="no-reply@e-deus.cf
	
Hacer una prueba::

	mailx -r test@domain.com -s "SUBJECT" [EMAIL_ADDRESS]

	# mailx -r admin@e-deus.cf -s "SUBJECT" cgomeznt@gmail.com
	Unknown command: "fwdretain"
	Saludos todo marcha muy bien
	.
	Cc:

Ver log::

	# tail -f /var/log/mail.log
	Mar  7 11:19:22 SRVPROIMPRENTA postfix/pickup[3614]: 3C859802C8B: uid=0 from=<admin@e-deus.cf>
	Mar  7 11:19:22 SRVPROIMPRENTA postfix/cleanup[5837]: 3C859802C8B: message-id=<20230307151922.3C859802C8B@SRVPROIMPRENTA.credicard.com.ve>


Probar el envio de correo y certificarlo con Gmail
----------------------------------------------------

Nos ayudamos con mailx::
	
	$ echo "Buenas. de esta forma es que usted puede enviar correos con Postfix" | mailx -v -s "Guia de envio de correo por Postfix" -r "cgomeznt@e-deus.online"  -S smtp="mail.e-deus.online:25" soporte.aplicaciones@credicard.com.ve cgomez@gmail.com
	


Esto es lo que debemos ver en el log::

		Jul  7 20:46:01 c946 postfix/smtpd[1045701]: connect from c946.gconex.com[190.114.9.23]
		Jul  7 20:46:01 c946 postfix/smtpd[1045701]: F35F91400C5: client=c946.gconex.com[190.114.9.23]
		Jul  7 20:46:01 c946 postfix/cleanup[1045705]: F35F91400C5: message-id=<64a87989.TDLKuidz/b1bgl5T%cgomeznt@e-deus.online>
		Jul  7 20:46:02 c946 postfix/smtpd[1045701]: disconnect from c946.gconex.com[190.114.9.23] helo=1 mail=1 rcpt=2 data=1 quit=1 commands=6
		Jul  7 20:46:02 c946 postfix/qmgr[895556]: F35F91400C5: from=<cgome1@e-deus.online>, size=612, nrcpt=2 (queue active)
		Jul  7 20:46:02 c946 postfix/smtp[1045706]: F35F91400C5: enabling PIX workarounds: disable_esmtp for credicorreo2.credicard.com.ve[200.109.231.217]:25
		Jul  7 20:46:03 c946 postfix/smtp[1045706]: F35F91400C5: to=<soporte.aplicaciones@credicard.com.ve>, relay=credicorreo2.credicard.com.ve[200.109.231.217]:25, delay=1, delays=0.02/0.03/0.32/0.67, dsn=2.0.0, status=sent (250 2.0.0 367Kk2fP017589-367Kk2fQ017589 Message accepted for delivery)
		Jul  7 20:46:03 c946 postfix/smtp[1045707]: F35F91400C5: to=<cgomeznt@gmail.com>, relay=gmail-smtp-in.l.google.com[172.217.203.27]:25, delay=1.1, delays=0.02/0.06/0.37/0.66, dsn=2.0.0, status=sent (250 2.0.0 OK  1688762763 b9-20020a0561020b0900b004435af1fc44si378866vst.550 - gsmtp)
		Jul  7 20:46:03 c946 postfix/qmgr[895556]: F35F91400C5: removed

