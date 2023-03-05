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

	set smtp=10.133.0.63
	set from="Zabbix@credicard.com.ve"
	
	
	
Este es un ejemplo de un Script
==============================
::

  FECHA=$(date +%d-%m-%Y_%H:%M)
  EMAILS="soporte.aplicaciones@credicard.com.ve Monitoreo1@credicard.com.ve monitoreo@credicard.com.ve  Monitorcc@credicard.com.ve Andres.Pineda@credicard.com.ve Edgar.Duran@credicard.com.ve"
  SUBJECT="Reporte Estatus plataforma Soporte Web $FECHA"
  BODY="/usr/local/bin/BODY_EMAIL.TXT"
  ATTACH="/usr/local/bin/Check-List_aplicaciones.xlsx"


  for email in $(echo $EMAILS)
  do
    tr -cd "[:print:]\n" < $BODY |  mail -r soporte.aplicaciones@credicard.com.ve -a $ATTACH -v -s "$SUBJECT" $email
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
  Soporte Aplicaciones <Soporte.Aplicaciones@credicard.com.ve>


Este es otro ejemplo de script 
=================================

En este caso se requiere de un HEADER y de un FOODER para ir armando el BODY::

  #!/bin/bash

  BODY="/CDC/scripts/bin/.BODY_EMAIL.TXT"
  HEADER="/CDC/scripts/bin/.HEADER_EMAIL.TXT"
  FOODER="/CDC/scripts/bin/.FOODER_EMAIL.TXT"

  function send_mail {

          FECHA=$(date +%d-%m-%Y_%H:%M)
          EMAILS="soporte.aplicaciones@credicard.com.ve Monitoreo1@credicard.com.ve monitoreo@credicard.com.ve  Monitorcc@credicard.com.ve Andres.Pineda@credicard.com.ve Edgar.Duran@credicard.com.ve Servicios.Web@credicard.com.ve Jean.Bautista@credicard.com.ve Rafael.Barreta@credicard.com.ve Soporte.BD@credicard.com.ve Soporte.AS400@credicard.com.ve Avedis.Khajikian@credicard.com.ve"
          SUBJECT="Estado de Salud de la Replica IBM CDC para SMI $FECHA"
          #ATTACH="/usr/local/bin/Check-List_aplicaciones.xlsx"


          for email in $(echo $EMAILS)
          do
                  #tr -cd "[:print:]\n" < $BODY |  mail -r soporte.aplicaciones@credicard.com.ve -a $ATTACH -v -s "$SUBJECT" $email
                  tr -cd "[:print:]\n" < $BODY |  mail -r Replica.CDC.SMI@credicard.com.ve -v -s "$SUBJECT" $email
          done
  }

  cat $HEADER > $BODY
  /CDC/scripts/bin/status_table_parked.sh S | grep -v Repl >> $BODY
  cat $FOODER >> $BODY

  send_mail
  echo > $BODY

  exit 0
