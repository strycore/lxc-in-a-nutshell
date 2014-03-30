********
Usecases
********


Use case: Wordpress blog
========================

Setting up MySQL

Setting up the Apache

Use case: Django project
========================

Based on http://michal.karzynski.pl/blog/2013/06/09/django-nginx-gunicorn-virtualenv-supervisor/

Setting up nginx in your container

Setting up nginx on the host

Use case: email server with Mailpile
====================================

* postfix
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

Use case: Redmine issue tracker
===============================

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

    sudo lxc-create -n airtime -t ubuntu -v FIXME

