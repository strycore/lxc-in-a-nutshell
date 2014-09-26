********
Usecases
********

Use case: Static website
========================

This container will host a very simple website consisting only of static
files. We will be using Nginx as the webserver.

Setting up nginx
----------------

First step is to install nginx::
    sudo apt-get install nginx

The default install will provide everything you need to get everything
running without any hassle. The web server's configuration is located
under /etc/nginx/nginx.conf but we won't need to tweak that.
What does need a bit of configuration is the virtual host you'll be
hosting in your container, we will use the default one, which is located
under /etc/nginx/sites-available/default.

If you look into /etc/nginx you'll notice two different folders:
sites-available and sites-enabled. The sites-enabled contains symlinks to
virtual hosts stored in sites-available, allowing you to enable or
disable a website by creating or removing a symlink.

A basic nginx configuration is quite short and simple to understand,
here's one as an example.

::
    server {
        listen 80 default_server;
        root /srv/mywebsite;
        index index.html;
        server_name example.com www.example.com;

        location / {
            try_files $uri.html $uri $uri/ =404;
        }
    }

Let's break this configuration apart to see what's going on here.
The whole configuration is contained in a server {} block, try to keep a
single one server block per file in order to take advantage of the
sites-available/enabled system (if your website has both HTTP and HTTPS
and requires two `server` blocks, you can keep those in the same file).

The first directive `listen 80` tells nginx on which port the website
should be made available, it's usually always 80 for HTTP and 443 for
HTTPS. The `default_server` tells nginx to choose this website if there
are ambiguities (for example, a user access the server by its IP address
instead of the domain name and several websites hosted on a single IP
address). Since we are using a container with a single website, this
parameter could be removed.

The `root` directive is very important to set to a valid value. It should
point to a valid directory containing your website files.

`index` point to the file used when a request is made on the base url.
Here, a request to `http://example.com` would serve the
`/srv/mywebsite/index.html` file.

`server_name` usually contains your domain names or aliases (the domain
with a www. prefix or other subdomains).

Next comes another block which specifies the configuration used for a
particular URL prefix. Having a single `location` block for `/` tells
nginx to use this for any URL on the website. You could have more
`location` block, pointing to prefixes such as `/blog` or `/wiki` and
providing a totally different configuration.

Our `location` block contains a single directive which will translate
URLs requested to a file on the server. If one makes a request to
`http://example.com/mypage` then nginx will try to serve (in that order)
the files /srv/mywebsite/mypage.html, /srv/mywebsite/mypage and
/srv/mywebsite/mypage/index.html. If none of these files exist on the
server then nginx will return a 404 error.

Listing files of a folder
-------------------------

By default, if you request the URL `http://example.com/images/` and
/srv/mywebsite/images is a folder not containing an `index.html` file (or
whatever the value of the `index` directive is) then nginx will return a
403 Forbidden error. If you wish to authorize viewing the contents of a
folder then add a `autoindex on` directive inside your main `server` block
or inside a particular `location` block.


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

Use case: Django project
========================

Based on http://michal.karzynski.pl/blog/2013/06/09/django-nginx-gunicorn-virtualenv-supervisor/

Setting up nginx in your container

Setting up nginx on the host

Use case: email server with Mailpile
====================================

Setting up postfix
------------------

Creating additional users
^^^^^^^^^^^^^^^^^^^^^^^^^

You may want to create users that have no shell access to the machine and
just be able to send and receive email. You can still use unix accounts
but you'll want to set their shell to `nologin`::
     useradd --create-home -s /usr/sbin/nologin emailuser
     passwd emailuser

* dovecot
* mailpile

Use case: Jenkins contiuous integration server
==============================================

Jenkins has an Ubuntu repository which will provide the latest vesion to
date. Install Jenkins by running the following commands:

::
    wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
    echo "deb http://pkg.jenkins-ci.org/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list
    sudo apt-get update
    sudo apt-get install jenkins

Use case: znc IRC bouncer
=========================

Installing znc::

    sudo apt-get update
    sudo apt-get install wget build-essential libssl-dev libperl-dev
    # For Python support
    sudo apt-get install pkg-config python3-dev
    wget http://znc.in/releases/znc-1.4.tar.gz
    tar xvzf znc-1.4.tar.gz
    cd znc-1.4
    ./configure --enable-python
    make

ZNC-push
--------

ZNC-push allows you to receive realtime notifications from ZNC to your
Android devices. It will send you push notifications everytime someone
highlights you or sends you a private message.

Installing ZNC-push::

    sudo apt-get install libcurl4-openssl-dev
    git clone https://github.com/jreese/znc-push.git
    make
    sudo make install

Use case: Redmine issue tracker
===============================

Install some required dependencies::

    sudo apt-get install mysql-server libmysqlclient-dev \
        libcurl4-openssl-dev libxml2-dev libxlst-dev libpq-dev \
        libmagickcore-dev libmagickwand-dev git subversion


Installing RVM::

    sudo -s
    curl -L https://get.rvm.io | bash -s stable --ruby=2.1.2
    echo "source /usr/local/rvm/scripts/rvm" >> ~/.bashrc

We will use `Passenger`_ to serve the Redmine application. Passeger is
able to download and compile nginx for us (with its own module enabled)

To install Passenger along with Nginx run::

    gem install passenger --no-ri --no-rdoc
    passenger-install-nginx-module

.. _Passenger: https://www.phusionpassenger.com

Install an init script to handle nginx::

    curl https://raw.githubusercontent.com/jnstq/rails-nginx-passenger-ubuntu/master/nginx/nginx -o /etc/init.d/nginx
    chmod +x /etc/init.d/nginx
    update-rc.d nginx defaults

Configure nginx::
   server {
      listen 80;
      server_name www.yourhost.com;
      root /somewhere/public;   # <--- be sure to point to 'public'!
      passenger_enabled on;
   }

Get latest stable version of Redmine


Install Redmine dependencies
    cd /srv/redmine
    bundle install

Troubleshooting
---------------

Passenger errors
^^^^^^^^^^^^^^^^

To enable detailled error messages from Passenger, you can add the
following directive to your nginx config::

    passenger_friendly_error_pages on;

If you activate friendly error pages, don't forget to set the directive
to `off` or to completely remove it when your Redmine instance is ready
for production!

Getting errors reports from nginx logs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you get some errors with your Redmine instance you can probably get
detailled information in nginx error logs::

    tail -n 200 /opt/nginx/logs/error.log



Use case: etherpad lite
=======================

Installing Node

Use case: git repository with gitlab
====================================

Use case: Minecraft server
==========================

Setting up the Java runtime

* Overviewer
* Bukkit ?

Use case: Managing your backups
===============================

* Sparkleshare
* Seafile http://seafile.com/en/home/
* RSync
* OwnCloud
* cryptfs

Use case: Groupware with Roundcube? / Sogo?
===========================================

* LDAP
* Caldav
* Webdav

Use case: Chat server with WebRTC and Movim
===========================================

* Tox ?
* Movim?
* WebRTC

Use case: Maps server with OpenStreetMap
========================================

* openstreetmap
* leaflet
* varnish

Use case: Building your own radio
=================================

Setting up a web radio is a realy simple task, within a matter of minutes
you can be broadcatsing your favorite song to the whole world. This
chapter will be divided into two parts. In the first part we will see how
to setup a simple radio using Liquid Soap and IceCast. In the second part
we will build a radio with more advanced setup using Airtime.

Simple setup with Liquid Soap and IceCast
-----------------------------------------

Using this method, you will be able to broadcast a playlist and maybe put
a few jingles in between. This may not allow very advenced control of your
stream, if you want more control please see next part.

After creating a new LXC container, you have to install two packages that
will run your radio. First one is Liquid Soap. This piece of software will
be responsible for build the stream you want to broadcast. Next one is
Icecast and will be responsible for broadcasting your audio stream on the
internet. Let's install those two and then we'll see how to use them.

::
    sudo apt-get install icecast2 liquidsoap

Icecast will ask a few questions during the install process but not sure
the are actually useful since I had to change the config file which was
still filled with the default values.

TODO: Write Liquidsoap script

TODO: Change Icecast config file

Advanced radio station with Airtime
-----------------------------------

TODO: Present Airtime (yes it is open source and Free)

Once small gotcha when installing Airtime, is that the package is not yet
compatible with Apache 2.4 with is the version shipped with Ubuntu 14.04.
Luckyly, since we're using LXC, it's trivial to create a container using a
previous version of Ubuntu. For Airtime, we will create a container with
Ubuntu 12.04 LTS::

    sudo lxc-create -n airtime -t ubuntu -- -r precise

Once the container is created, you can go on and install Airtime. Th
website provides a Debian / Ubuntu package which will setup everything
nicely for you::

    sudo apt-get install wget
    wget http://apt.sourcefabric.org/misc/airtime-easy-setup.deb
    sudo apt-get -f install
    sudo apt-get update
    sudo apt-get install airtime



