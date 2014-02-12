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

    sudo vim editor /etc/bind/named.conf.options

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
