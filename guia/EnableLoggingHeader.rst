Postfix – Enable logging of email’s subject in maillog
=========================================================


Step:1 Edit ‘/etc/postfix/main.cf’ file & uncomment below line::

	#header_checks = regexp:/etc/postfix/header_checks
	
	
Step:2 Append the below line in ‘/etc/postfix/header_checks’::

	/^Subject:/     WARN

Step:3 Restart the postfix server::

	#service postfix restart

Con esto verificamos que tengamos la configuración::

	#postmap /etc/postfix/header_checks
	
En el log debera ver **warning: header Subject** algo como esto::

	Jul 31 16:20:28 lrkprdappsmtprelay postfix/smtpd[528880]: connect from w12prdappspooli.credicard.com.ve[10.132.0.40]
	Jul 31 16:20:28 lrkprdappsmtprelay postfix/smtpd[528880]: E324FC01379: client=w12prdappspooli.credicard.com.ve[10.132.0.40]
	Jul 31 16:20:28 lrkprdappsmtprelay postfix/cleanup[528886]: E324FC01379: message-id=<43bfa9ec5256e0066804bd270016757c@credicard.com.ve>
	Jul 31 16:20:28 lrkprdappsmtprelay postfix/cleanup[528886]: E324FC01379: warning: header Subject: SpooliT Server -Test from w12prdappspooli.credicard.com.ve[10.132.0.40]; from=<spoolit@credicard.com.ve> to=<carlos.gomez@credicard.com.ve> proto=ESMTP helo=<w12prdappspooli.local>
	Jul 31 16:20:28 lrkprdappsmtprelay postfix/qmgr[528747]: E324FC01379: from=<spoolit@credicard.com.ve>, size=9571, nrcpt=1 (queue active)
	Jul 31 16:20:28 lrkprdappsmtprelay postfix/smtpd[528880]: disconnect from w12prdappspooli.credicard.com.ve[10.132.0.40] ehlo=2 starttls=1 mail=1 rcpt=1 data=1 quit=1 commands=7
	Jul 31 16:20:28 lrkprdappsmtprelay postfix/smtp[528887]: E324FC01379: enabling PIX workarounds: disable_esmtp for 10.136.0.90[10.136.0.90]:25
	Jul 31 16:20:29 lrkprdappsmtprelay postfix/smtp[528887]: E324FC01379: to=<carlos.gomez@credicard.com.ve>, relay=10.136.0.90[10.136.0.90]:25, delay=0.22, delays=0.04/0.04/0/0.14, dsn=2.0.0, status=sent (250 2.0.0 36VKKS9J026559-36VKKS9K026559 Message accepted for delivery)
	Jul 31 16:20:29 lrkprdappsmtprelay postfix/qmgr[528747]: E324FC01379: removed

	
