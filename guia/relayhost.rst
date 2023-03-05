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

  myhostname = srv-jboss
  relayhost = mail.e-deus.cf
  
Reinicie el servicio postfix para que los cambios surtan efecto::

  # systemctl restart postfix
  
Verifique la retransmisión de correo mediante los comandos telnet/mail(mailx)::

  # echo "Correo de ip from postfix" | mailx -v -s "Probando relay de postfix" -r "cgomeznt@e-deus.cf"  -S smtp="mail.e-deus.cf:25" cgomeznt@gmail.com

