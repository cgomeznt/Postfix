Como configurar Postfix SMTP relay
==================================


Para hacer **relay** (retransmitir) correo a través de otro servidor smtp, el archivo /etc/postfix/main.cf debe configurarse con el parámetro **relayhost** que especifica el servidor smtp de destino.

Instalar postfix::

  # dnf install postfix
  
Realice una copia de seguridad de la configuración predeterminada::

  # cp /etc/postfix/main.cf /etc/postfix/main.cf.bkp_default
  
Edite /etc/postfix/main.cf y agregue/modifique el siguiente parámetro en la configuración::

  myhostname = [nombre de host del servidor]
  relayhost = [Dirección IP/Nombre de host del servidor de retransmisión]

Ejemplo; hay un servidor de SMTP de e-deus.cf y nuestros servidores lo van utilizar como relay::

  myhostname = srv-api
  relayhost = mail.e-deus.cf
  
Reinicie el servicio postfix para que los cambios surtan efecto::

  # systemctl restart postfix

Instalar **mailx** https://github.com/cgomeznt/Postfix/blob/master/guia/mailx.rst

Verifique la retransmisión de correo mediante los comandos telnet/mail(mailx)::

  # echo "Correo de ip from postfix" | mailx -v -s "Probando relay de postfix" -r "cgomeznt@e-deus.cf"  -S smtp="mail.e-deus.cf:25" cgomeznt@gmail.com

En el servidor de Postfix SMTP, en el log vera::

  Mar  5 02:40:12 c946 postfix/smtpd[50882]: warning: hostname 186-185-53-27.genericrev.telcel.net.ve does not resolve to address 186.185.53.27
  Mar  5 02:40:12 c946 postfix/smtpd[50882]: connect from unknown[186.185.53.27]
  Mar  5 02:40:12 c946 postfix/smtpd[50882]: discarding EHLO keywords: CHUNKING
  Mar  5 02:40:12 c946 postfix/smtpd[50882]: discarding EHLO keywords: CHUNKING
  Mar  5 02:40:12 c946 postfix/smtpd[50882]: C1FB114007B: client=unknown[186.185.53.27]
  Mar  5 02:40:12 c946 postfix/cleanup[50890]: C1FB114007B: message-id=<64040105.f8Y2hmi0wXbp/VqL%cgomeznt@e-deus.cf>
  Mar  5 02:40:12 c946 postfix/qmgr[45470]: C1FB114007B: from=<cgome1@e-deus.cf>, size=688, nrcpt=1 (queue active)
  Mar  5 02:40:12 c946 postfix/smtpd[50882]: disconnect from unknown[186.185.53.27] ehlo=2 starttls=1 mail=1 rcpt=1 data=1 quit=1 commands=7
  Mar  5 02:40:13 c946 postfix/smtp[50891]: C1FB114007B: to=<cgomeznt@gmail.com>, relay=gmail-smtp-in.l.google.com[108.177.12.26]:25, delay=0.96, delays=0.18/0.03/0.3/0.45, dsn=2.0.0, status=sent (250 2.0.0 OK  1677984013 j5-20020a67f3c5000000b00414568d954dsi1716902vsn.204 - gsmtp)
  Mar  5 02:40:13 c946 postfix/qmgr[45470]: C1FB114007B: removed
