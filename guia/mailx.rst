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
	Mar  7 11:19:22 SRVPROIMPRENTA postfix/qmgr[3615]: 3C859802C8B: from=<admin@e-deus.cf>, size=438, nrcpt=1 (queue active)
	Mar  7 11:19:23 SRVPROIMPRENTA postfix/smtp[5839]: 3C859802C8B: to=<cgomeznt@gmail.com>, relay=mail.e-deus.cf[190.114.9.23]:25, delay=1.3, delays=0.01/0.01/1.2/0.16, dsn=2.0.0, status=sent (250 2.0.0 Ok: queued as 787731400D0)
	Mar  7 11:19:23 SRVPROIMPRENTA postfix/qmgr[3615]: 3C859802C8B: removed
