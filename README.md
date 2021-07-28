# ansible-role-postfix-dovecot
An Ansible role that automates the installation and configuration of Postfix and Dovecot with MySQL authentication on Ubuntu. The MySQL schema is derived from the following
[Digital Ocean tutorial](https://www.digitalocean.com/community/tutorials/how-to-configure-a-mail-server-using-postfix-dovecot-mysql-and-spamassassin).
You can view the MySQL schema used in [schema.sql](schema.sql).

## Role Variables

#### Required Variables
* **dovecot_ssl_cert** - the path to the SSL certificate used by Dovecot. Note that if you need to provide a certificate chain,
it must be concatenated after the certificate in the same file.
* **dovecot_ssl_key** - the path to the SSL key used by Dovecot.
* **postfix_ssl_cert** - the path to the SSL certificate used by Postfix. This should include the intermediary CA as well if applicable.
* **postfix_ssl_key** - the path to the SSL key used by Postfix.
* **postfix_dovecot_mysql_password** - the password to the user that has permission to query the database on the SQL database server used for authentication.

#### Optional Variables
* **postfix_dovecot_mysql_host** - the FQDN or IP address to the MySQL server for authentication. This defaults to `127.0.0.1`.
* **postfix_dovecot_mysql_db_name** - the database name on the MySQL server used for authentication. This defaults to `servermail`.
* **postfix_dovecot_mysql_user** - the user that has permission to query the database on the MySQL server used for authentication. This defaults to `usermail`.
* **postfix_dovecot_mysql_password_scheme** - the password scheme used to encrypt passwords in the database. This defaults to `SHA512-CRYPT`.
* **postfix_default_domain** - the value to set the default domain used by Postfix, particularly when Postfix determines the sender's domain when sending bounce messages. This sets the contents of `/etc/mailname`.
* **postfix_inet_protocols** - the protocol that Postfix should listen on. To have only IPv4, set this value to `ipv4`. This defaults to `all`.
* **postfix_submission_smtpd_client_restrictions** - a list of client restrictions on the mail submission port (587). For more information visit the [Postfix documentation](http://www.postfix.org/postconf.5.html#smtpd_client_restrictions).
This defaults to `permit_sasl_authenticated` and `reject`.
* **postfix_smtpd_tls_auth_only** - whether to only allow SASL authentication over SSL/TLS. This defaults to `yes`.
* **postfix_smtpd_recipient_restrictions** - a list of restrictions of recipients of incoming email. For more information visit the [Postfix documentation](http://www.postfix.org/postconf.5.html#smtpd_recipient_restrictions).
This defaults to `permit_sasl_authenticated`, `permit_mynetworks`, and `reject_unauth_destination`.
* **postfix_smtpd_relay_restrictions** - a list of relay restrictions. For more information visit the [Postfix documentation](http://www.postfix.org/postconf.5.html#smtpd_relay_restrictions).
This defaults to `permit_mynetworks`, `permit_sasl_authenticated`, and `defer_unauth_destination`.
* **postfix_mynetworks** - a list of trusted SMTP clients. For more information visit the [Postfix documentation](http://www.postfix.org/postconf.5.html#mynetworks).
This defaults to `127.0.0.0/8`, `[::ffff:127.0.0.0]/104`, `[::1]/128`.
* **postfix_mydestination** - a list for the Postfix configuration value of `mydestination`. For information on visit the [Postfix documentation](http://www.postfix.org/postconf.5.html#mydestination).
This defaults to `localhost`.
* **postfix_mysql_alias_query** - the query used to find the destination of an alias when the source is supplied. This defaults to `SELECT destination FROM virtual_aliases WHERE source='%s';`.
* **postfix_mysql_domains_query** - the query used to determine if a domain is valid. This defaults to `SELECT 1 FROM virtual_domains WHERE name='%s';`.
* **postfix_mysql_users_query** - the query used to determine if an email address is valid. This defaults to `SELECT 1 FROM virtual_users WHERE email='%s';`.
* **dovecot_mysql_password_query** - the query used to authenticate a user on the MySQL server used for authentication. This defaults to `SELECT email as user, password FROM virtual_users WHERE email='%u';`.
* **dovecot_protocols** - a list of protocols to be enabled. This defaults to `lmtp` and `imap`. To enable POP3, add `pop3` to this variable. (note: `apt install dovecot-pop3d` on the target to use pop3)  
* **dovecot_mail_privileged_group** - the group that owns the folder defined in `dovecot_mail_location`.
This gives Dovecot's mail process the ability to write in the folder. This defaults to `mail`.
* **dovecot_disable_plaintext_auth** - determines if authentication without SSL is enabled. This defaults to 'yes'.
* **dovecot_auth_mechanisms** - a list of authentication mechanisms allowed by Dovecot. This defaults to `plain` and `login`.
For more informationm read Dovecot's [Authentication Mechanisms](http://wiki2.dovecot.org/Authentication/Mechanisms) documentation.
* **dovecot_force_imaps** - determines whether or not to disable IMAP and force IMAPS. This defaults to `true`.
* **dovecot_force_pop3s** - determines whether or not to disable POP3 and force POP3S. This defaults to `true`.
Note that to also enable POP3S, you need to add pop3 to the `dovecot_protocols` list variable.
* **dovecot_ssl** - determines whether or not SSL is enforced across all protocols. This defaults to `required`.
For more information, read Dovecot's [SSL Configuration](http://wiki.dovecot.org/SSL/DovecotConfiguration) documentation.
* **dovecot_listen** - a list of IP or host addresses where Dovecot listens for connections. This defaults to `*` (all IPv4) and '::' (all IPv6).
* **dovecot_add_example_users** - when set to `true`, adds example users to the database

## Requirements

* This role must be run with sudo/become or as root, otherwise the role will fail.
* The MySQL server needs to be pre-configured, and the user should already have the appropriate permissions to the database (see [defaults/main.yml] for default values).
* On Red Hat servers, you need to pre-install PyMSQL (python{2,3}-PyMySQL, which ever is more appropriate to you)

## Example Playbook

_requirements.yml_
```yaml
roles:
  - name: stackfocus.postfix-dovecot
```

_site.yml_
```yaml
- hosts: all
  become: yes
  gather_facts: true
  roles:
    - stackfocus.postfix-dovecot
  vars:
    postfix_dovecot_mysql_db_name: mailserver
    postfix_dovecot_mysql_user: mailuser
    postfix_dovecot_mysql_password: mailpass
    postfix_default_domain: example.com
    dovecot_protocols:
      - imap
      - pop3
      - lmtp
    dovecot_mail_privileged_group: vmail
    dovecot_ssl_cert: /etc/ssl/certs/dovecot.pem
    dovecot_ssl_key: /etc/ssl/private/dovecot.pem
    postfix_ssl_cert: /etc/ssl/certs/postfix.pem
    postfix_ssl_key: /etc/ssl/private/postfix.pem

```


```
$ ansible-galaxy install -r requirements.yml
$ ansible-playbook -i inventory site.yml --ask-become-pass
```
