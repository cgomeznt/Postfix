Cómo cambiar el encabezado De para los mensajes enviados por Postfix
=====================================================================

Agregue la siguiente línea en todos los correos electrónicos salientes tendrán esta dirección en el campo DE, pero el nombre del remitente no se modificará. Reemplace <FQDN> con su nombre de dominio completo./etc/postfix/main.cf::

  sender_canonical_maps = static:no-reply@<FQDN>
  
Para modificar también el nombre, debe crear un archivo que  /etc/postfix/header_checks contenga esta línea::

  /^From:[[:space:]]+(.*)/ REPLACE From: "Your Name" <email@company.com>

Luego ejecute los siguientes comandos::

  postmap /etc/postfix/header_checks
  postconf -e 'smtp_header_checks = regexp:/etc/postfix/header_checks'
  systemctl restart postfix
