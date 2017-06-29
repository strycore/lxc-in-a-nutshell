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



