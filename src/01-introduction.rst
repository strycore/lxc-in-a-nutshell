Introduction
============


Before you start
----------------

 - Ubuntu Server
 - New (and stock) kernel

Installing LXC
--------------

::
    sudo apt-get install lxc

Creating your first container
-----------------------------

Create the container

::
    sudo lxc-create -t ubuntu -n mycontainer

Start the container

::
    sudo lxc-start -n ubuntu

Login in container

::
    sudo lxc-console -n ubuntu
