Network configuration
=====================


Setting the IP address
----------------------

You can set the IP adress from you host by editing the configuration file for
your container:

::

    sudo editor /var/lib/lxc/containername/config
    # Add the following line and put the desired IP:
    lxc.network.ipv4  = 10.0.3.42


Firewall setup (UFW)
--------------------

By default, UFW drops all forwarding traffic. As a result you will need to
enable UFW forwarding:

::

    sudo editor /etc/default/ufw
    # Change:
    # DEFAULT_FORWARD_POLICY="DROP"
    # to
    DEFAULT_FORWARD_POLICY="ACCEPT"


Setting up a DNS server
-----------------------

Make sure you have bind9 installed on your host machine and that you have the
vanilla config from the Ubuntu packages. Some server providers (like kimsufi)
will ship with a custom config that will only make things harder. Save yourself
some headaches and reinstall the bind9 package with the stock config.

::

    sudo apt-get remove --purge bind9
    sudo apt-get install bind9

You will probably want your DNS server to act as a forwarder, using your
provider's nameservers. In that case you can add to your bind9 configuration
the following:

::

    sudo editor /etc/bind/named.conf.options

    # These are the IPs for Google DNS servers, substitute them with the ones
    # provided by your ISP.
    forwarders {
        8.8.8.8
        8.8.4.4
    };

Restart the bind9 service to apply your changes, if anything goes wrong check
your logs in /var/log/syslog:

::

    sudo service bind9 restart
    * Stopping domain name service... bind9
    waiting for pid 2321 to die
    ...done.
    * Starting domain name service... bind9
    ...done.

Once your DNS server is set up, you probably don't want it to be visible to the
outside world. You can block incoming request with your firewall.

::

    sudo ufw deny from any to 94.23.12.211 port 53


SSH
---

For your deployment purposes and other tasks, you will probably need direct
access to your lxc containers via ssh. You can achieve this by setting up a SSH
tunnel on your host, making your container accessible from anywhere.
You will need a different port for each of your containers you want to access.
It is sadly not possible to route your SSH connection based on the hostname
like a webserver does.

Let's say your container's IP is 10.0.3.101 and you wish to access to your 
container via the port 22101. On your host machine, run the following:

::

    ssh -L 0.0.0.0:22101:10.0.3.101:22 localhost
    # Don't forget to open the port to the outside
    sudo ufw allow 22101

Then from the outside you can run:

::

    ssh myusername@domain.of.host -p 22101

For further convinience, edit your .ssh/config file to set up default values for this
connection. This assume that you give your containers unique domain names either 
in your DNS configuration or in /etc/hosts.

::

    editor .ssh/config

    # Add the following
    Host mycontainer.domain
        User myusername
        Port 22101

From now on you can log in you container with:

::

    ssh mycontainer.domain
