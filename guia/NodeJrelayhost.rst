


Como Nodemailer y SMTP
++++++++++++

El Protocolo simple de transferencia de correo (SMTP) es una tecnología para enviar correos electrónicos salientes a través de redes y es el método de transporte más común. Sirve como un servicio de retransmisión para enviar correos electrónicos de un servidor a otro::

  mkdir email-nodeapp && cd email-nodeapp 
  npm init -y

Instalamos el nodemailer, con npm ::

  npm install nodemailer
  
Creamos el archivo  **email.js**  y agregamos::

  process.env["NODE_TLS_REJECT_UNAUTHORIZED"] = 0;
  const nodemailer = require('nodemailer');
        let transporter = nodemailer.createTransport({
               host: 'mail.e-deus.cf',
               port: 25,
               secure: false
       })
  message = {
           from: "cgomez@e-deus.cf",
           to: "cgomeznt@calheta.com",
           subject: "1 Prueba desde Nodej",
           text: "Esto es un Hello SMTP Email, desde Nodej"
      }
      transporter.sendMail(message)

Deshabilitamos la verificación de TLS::

  export NODE_TLS_REJECT_UNAUTHORIZED='0'
  
Y luego solo ejecutamos::

  node email.js

How to Send Emails with Node.js [3 Different Ways + Code Tutorials]
+++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

https://www.courier.com/blog/how-to-send-emails-with-node-js/


nodejs - error self signed certificate in certificate chain
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

https://stackoverflow.com/questions/45088006/nodejs-error-self-signed-certificate-in-certificate-chain
