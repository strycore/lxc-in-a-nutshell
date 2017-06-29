Use case: email server with Postfix
===================================

Setting up postfix
------------------

Using port 587 to send email
----------------------------

Some ISP might block access to port 25, disabling you from using your
mail server to send email. You can workaround this by using port 587
instead. Edit the file /etc/postfix/master.cf and uncomment the line
starting with `submission`::

    submission inet n      -       n       -       -       smtpd

Then restart your mail server and check if it's available using telnet::

    telnet mail.example.com 587
    Trying 1.2.3.4...
    Connected to mail.example.com.
    Escape character is '^]'.
    220 mail.example.com ESMTP Postfix (Ubuntu)

Creating additional users
-------------------------

You may want to create users that have no shell access to the machine and
just be able to send and receive email. You can still use unix accounts
but you'll want to set their shell to `nologin`::
     useradd --create-home -s /usr/sbin/nologin emailuser
     passwd emailuser

* dovecot
* mailpile


