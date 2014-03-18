Usecases
========


Use case: Wordpress blog
------------------------

Setting up MySQL

Setting up the Apache

Use case: Django project
------------------------

Based on http://michal.karzynski.pl/blog/2013/06/09/django-nginx-gunicorn-virtualenv-supervisor/

Setting up nginx in your container

Setting up nginx on the host

Use case: email server with Mailpile
------------------------------------

* postfix
* dovecot
* mailpile

Use case: Jenkins contiuous integration server
----------------------------------------------

Jenkins has an Ubuntu repository which will provide the latest vesion to
date. Install Jenkins by running the following commands:

::
    wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
    echo "deb http://pkg.jenkins-ci.org/debian binary/" | sudo tee /etc/apt/sources.list.d/jenkins.list
    sudo apt-get update
    sudo apt-get install jenkins

Use case: znc IRC bouncer
-------------------------

Use case: Redmine issue tracker
-------------------------------

Use case: etherpad lite
-----------------------

Installing Node

Use case: git repository with gitlab
------------------------------------

Use case: Minecraft server
--------------------------

Setting up the Java runtime

* Overviewer
* Bukkit ?

Use case: Managing your backups
-------------------------------

* Sparkleshare
* RSync
* OwnCloud
* cryptfs

Use case: Groupware with Roundcube? / Sogo?
-------------------------------------------

* LDAP
* Caldav
* Webdav

Use case: Chat server with WebRTC and Movim
-------------------------------------------

* Tox ?
* Movim?
* WebRTC

Use case: Maps server with OpenStreetMap
----------------------------------------

* openstreetmap
* leaflet
* varnish
