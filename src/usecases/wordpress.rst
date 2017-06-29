Use case: Wordpress blog
========================

In this setup we'll use a classic LAMP architecture with apache + php and
mysql separated in two different containers.

Setting up MySQL
----------------

There's only one mandatory package to install in the databse container,
the mysql server::
    sudo apt-get install mysql-server

Once you have your mysql server up and running, you might have some
backups you want to re-import::
    mysql -u root -p < mysql-dump.sql

If your legacy database was running on the same host as your PHP
installation, is it likely that your mysql users will be restricted to
localhost. You'll have to modify permissions for your users and authorize
access from your PHP container.
First, edit your MySQL config file located in /etc/mysql/my.cnf and
listen on all network interfaces (since we are using containers, the
mysql server won't be visible to the outside world). Change the
bind-address directive::

    bind-address    = 0.0.0.0

Restart your MySQL server and your server will be visible to other
containers::
    nc -vz mysql 3306
    Connection to mysql 3306 port [tcp/mysql] succeeded!
Now that your server is reachable, you must now authorize your users to
connect and use their databases. Connect to your mysql server command
line::
    mysql -uroot -p

If you need a reminder of which database and which users you have, you
can list them::
    # List all available databases
    show databases;

    # List all users and the host they are authorized on.
    select User, Host from mysql.user;

Now you can change the host from which a user is able to connect::
    update mysql.user set Host='%' where User='myusername';

You also have to authorize the user to access the appripriate database::
    grant all on mydb.* to myusername@'%' identified by 'mypassword';
    flush privileges;

Note that setting the host to '%' will allow the user to connect from
anywhere. If that's not what you want, you can substitute '%' by an IP
address or a hostname.

Setting up Apache
-----------------

TODO
