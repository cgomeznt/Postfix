Algunas formas de trabajar con Postfix mail queue
=====================================================

Cómo revisar el mensaje de correo electrónico que su servidor está intentando enviar.

Hay dos colas dentro de Postfix: pending y deferred. La cola pending incluye todos los mensajes que se han enviado a postfix que aún no se han enviado y entregado al servidor del destinatario. La cola de correo deferred contiene todos los mensajes que han fallado por software y deben ser retirados (falla temporal). Postfix volverá a intentar enviar la cola diferida en intervalos establecidos (esto es configurable, pero está configurado en 5 minutos de forma predeterminada).

Los siguientes comandos le permitirán revisar estas colas:

1- Mostrar los mail queues, deferred y pending::

	mailq

	o

	postqueue -p

2- Ver un message (contents, header and body) en Postfix queue, para ver el message con el ID XXXXXXX::


	postcat -vq XXXXXXXXXX


3- Indicarle al Postfix que procese las colas ahora. Esto causa que Postfix inmediatamente envie nuevamente los mensajes que estan en cola::

	postqueue -f

	O

	postfix flush


4- Borrar todas las queued mail::


	postsuper -d ALL

Borrar solo las differed mail queue ::

	postsuper -d ALL deferred

5- Borrar mail que sean por una seleccion::



	#!/usr/bin/perl

	$REGEXP = shift || die “no email-adress given (regexp-style, e.g. bl.*\@yahoo.com)!”;

	@data = qx;

	for (@data) {

	  if (/^(\w+)(\*|\!)?\s/) {

		 $queue_id = $1;

	  }

	  if($queue_id) {

		if (/$REGEXP/i) {

		  $Q{$queue_id} = 1;

		  $queue_id = “”;

		}

	  }

	}

	 #open(POSTSUPER,”|cat”) || die “couldn’t open postsuper” ;

	open(POSTSUPER,”|postsuper -d -“) || die “couldn’t open postsuper” ;

	 foreach (keys %Q) {

	  print POSTSUPER “$_\n“;

	};

	close(POSTSUPER);

	#########################################

Ejemplo: Borrar todas mensajes encolados de o para el dominio llamado spamers.com::


	./postfix-delete.pl spamers.com

orrar todas mensajes encolados que contengan la palabra spam en el e-mail address::

	./postfix-delete.pl spam