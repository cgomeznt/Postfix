Instalar Postfix solo para el envio SMTP CentOS7
=================================================

Cómo configurar Postfix como un servidor SMTP de solo envío en CentOS 7. Primero le mostraré cómo hacerlo para un solo dominio, luego puede aplicar los pasos para múltiples dominios si lo necesita para.

Caso de uso
+++++++++++++

Tiene un sitio web/aplicación web que necesita enviar correos electrónicos transaccionales a los usuarios (como correo electrónico de restablecimiento de contraseña) o para un proceso informativo de batch. Lo más probable es que los usuarios no tengan que responder a estos correos electrónicos o, si responden, los correos electrónicos de respuesta se enviarán a su servidor de correo dedicado. En este caso, puede configurar un servidor SMTP de solo envío en el servidor web utilizando Postfix, que es un software de servidor SMTP popular.

Prerrequisitos
Para enviar correos electrónicos desde su servidor, el puerto 25 (saliente) debe estar abierto. Muchos ISP y empresas de alojamiento, como DigitalOcean, bloquean el puerto 25 para controlar el spam. Aunque Hostwinds no bloquea el puerto 25 (saliente).

Primero, necesitamos configurarlo para un dominio, luego configurarlo para múltiples dominios si así lo necesitamos.

Paso 1: Establecer el nombre de host y el registro PTR
+++++++++++++++++++++++++++++

De forma predeterminada, Postfix utiliza el nombre de host de su servidor para identificarse cuando se comunica con otros servidores SMTP. Algunos servidores SMTP rechazarán su correo electrónico si su nombre de host no es válido. Debe establecer un nombre de dominio completo (FQDN) como se muestra a continuación.::

	hostnamectl set-hostname mail.e-deus.cf

Verificamos el hostname del servidor::

	hostname -f

Debe cerrar la sesión y volver a iniciarla para ver el cambio de nombre de host en el símbolo del sistema. Este nombre de host debe tener un registro DNS A que apunte a la dirección IP de su servidor.

Establecer un registro PTR, que asigna una dirección IP a un FQDN. Muchos servidores SMTP rechazarán correo electrónico si la dirección IP de su servidor no tiene registro PTR.

Para ver si su registro PTR está configurado correctamente, ejecute el siguiente comando. ::

	host 190.198.55.89

	dig -x 190.198.55.89

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

**Reiniciar Postfix**
Finalmente, necesitamos reiniciar Postfix para que los cambios surtan efecto.::

	systemctl restart postfix

Este documento esta inconcluso aun falta..!!!

