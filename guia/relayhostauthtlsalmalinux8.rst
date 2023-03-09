Postfix usando relayhost AUTH SASL TLS Almalinux 8
======================================================

Para configurar postfix para retransmitir correo utilizando otro MTA, puede realizar los siguientes pasos::

  postconf -e 'relayhost = e-deus.cf'
  postconf -e 'smtp_sasl_auth_enable = yes'
  postconf -e 'smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd'
  postconf -e 'smtp_sasl_security_options='

e-deus.cf es el nombre de host MTA original que va a usar para la retransmisión. Ahora, crea el sasl_passwd archivo /etc/postfix con lo siguiente dentro::

  smtp.to.relay.com smtp_username:smtp_password

Donde seria esto::

  mail.e-deus.cf cgomeznt:Betania21

Ahora generamos el postfix hash db::

  postmap /etc/postfix/sasl_passwd

Podemos verificar, debe retornar el nombre del realay, el usaurio:clave ::

  postmap -q smtp.to.relay.com /etc/postfix/sasl_passwd

Reiniciamos el servicio::

  systemctl restart postfix


Hacemos una prueba::

  # mailx -r no-reply@e-deus.cf -s "Postfix AUTH SASL y TLS" cgomeznt@gmail.com
  Unknown command: "fwdretain"
  Buenas. Esto es una prueba
  .
  Cc:
  
En el LOG del servidor de Relay debemos ver algo como esto::

  Mar  9 00:22:39 c946 postfix/smtpd[58436]: connect from unknown[193.42.33.16]
  Mar  9 00:22:39 c946 postfix/smtpd[58436]: discarding EHLO keywords: CHUNKING
  Mar  9 00:22:39 c946 postfix/smtpd[58436]: warning: unknown[193.42.33.16]: SASL LOGIN authentication failed: authentication failure
  Mar  9 00:22:40 c946 postfix/smtpd[58436]: disconnect from unknown[193.42.33.16] ehlo=1 auth=0/1 rset=1 quit=1 commands=3/4
  Mar  9 00:23:02 c946 postfix/smtpd[58436]: warning: hostname 186-185-3-41.genericrev.telcel.net.ve does not resolve to address 186.185.3.41
  Mar  9 00:23:02 c946 postfix/smtpd[58436]: connect from unknown[186.185.3.41]
  Mar  9 00:23:02 c946 postfix/smtpd[58436]: discarding EHLO keywords: CHUNKING
  Mar  9 00:23:03 c946 postfix/smtpd[58436]: 15F1A140088: client=unknown[186.185.3.41], sasl_method=LOGIN, sasl_username=cgomeznt
  Mar  9 00:23:03 c946 postfix/cleanup[58441]: 15F1A140088: message-id=<20230309002258.04FD2802C8B@SRVPROIMPRENTA>
  Mar  9 00:23:03 c946 postfix/qmgr[57111]: 15F1A140088: from=<cgome1@e-deus.cf>, size=584, nrcpt=1 (queue active)
  Mar  9 00:23:03 c946 postfix/smtpd[58436]: disconnect from unknown[186.185.3.41] ehlo=1 auth=1 mail=1 rcpt=1 data=1 quit=1 commands=6
  Mar  9 00:23:03 c946 postfix/smtp[58442]: 15F1A140088: to=<cgomeznt@gmail.com>, relay=gmail-smtp-in.l.google.com[173.194.210.27]:25, delay=0.73, delays=0.12/0.03/0.28/0.3, dsn=2.0.0, status=sent (250 2.0.0 OK  1678321383 z12-20020a67f54c000000b004120559131csi5015066vsn.559 - gsmtp)
  Mar  9 00:23:03 c946 postfix/qmgr[57111]: 15F1A140088: removed


