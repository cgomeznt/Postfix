Enabling Postfix SMTPUTF8 support
====================================

There is more to SMTPUTF8 than just Postfix itself. The rest of your email infrastructure also needs to be able to handle UTF-8 email addresses and message header values. This includes SMTPUTF8 protocol support in SMTP-based content filters (Amavisd), LMTP servers (Dovecot), and down-stream SMTP servers.

Postfix SMTPUTF8 support is enabled by default, but it may be disabled as part of a backwards-compatibility safety net (see the COMPATIBILITY_README file).

SMTPUTF8 support is enabled by setting the smtputf8_enable parameter in main.cf::

	# postconf "smtputf8_enable = yes"
	# postfix reload