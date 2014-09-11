*********************
Network configuration
*********************


Setting the IP address
======================

You can set the IP adress from you host by editing the configuration file
for your container:

::

    sudo editor /var/lib/lxc/containername/config
    # Add the following line and put the desired IP:
    lxc.network.ipv4  = 10.0.3.42


Firewall setup
==============

With iptables

Iptables is the default application provided on Linux systems to control
the firewall features provided by the kernel.

You can keep your iptables rules in a file where each command will be run
sequentially. Keep in mind that the order of these commands matter and a
bad sequence could possibly lock yourself out of your own server.

Configure the filter table:

::
    * filter

    # Accept all loopback trafic
    -A INPUT -i lo -j ACCEPT
    # Accept the ICMP protocol (your server will respond to ping)
    -A INPUT -p icmp -j ACCEPT

    # Accept all inbound traffic from already established connections
    -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT

    # By default accept all packets to be forwarded
    -A FORWARD -j ACCEPT
    # By default accept all outgoing traffic
    -A OUTPUT -j ACCEPT

    # Allow incoming traffic from SSH
    -A INPUT -p tcp --dport ssh -j ACCEPT

    # Allow HTTP traffic
    -A INPUT -p tcp --dport 80 -j ACCEPT
    -A INPUT -p tcp --dport 443 -j ACCEPT

    COMMIT

Configure the nat table:

::
    *nat

    # Allow routing from eth0 (the interface connected to the internet)
    -A POSTROUTING -o eth0 -j MASQUERADE

    # Forward SSH connections to your containers.
    -A PREROUTING -i eth0 -p tcp --dport 22101 -j DNAT --to-destination 10.0.3.101:22
    -A PREROUTING -i eth0 -p tcp --dport 22102 -j DNAT --to-destination 10.0.3.102:22
    -A PREROUTING -i eth0 -p tcp --dport 22103 -j DNAT --to-destination 10.0.3.103:22
    -A PREROUTING -i eth0 -p tcp --dport 22104 -j DNAT --to-destination 10.0.3.104:22

    COMMIT


One common way to manage your iptables settings and have them running as a
service is binding them with your network interface in the
/etc/network/interfaces file.

::
    auto eth0
    iface eth0 inet static
        # Your network interface config here
        pre-up iptables-restore < /etc/iptables.up.rules

You can also reload your firewall after you have modified them::
    sudo iptables-restore < /etc/iptables.up.rules

With UFW

ufw (Uncomplicated Firewall) is a wrapper around iptables providing a user
friendly interface to manipulate your firewall.

By default, UFW drops all forwarding traffic. As a result you will need to
enable UFW forwarding:

::

    sudo editor /etc/default/ufw
    # Change:
    # DEFAULT_FORWARD_POLICY="DROP"
    # to
    DEFAULT_FORWARD_POLICY="ACCEPT"


Setting up a DNS server
=======================

Make sure you have bind9 installed on your host machine and that you have
the vanilla config from the Ubuntu packages. Some server providers (like
kimsufi) will ship with a custom config that will only make things harder.
Save yourself some headaches and reinstall the bind9 package with the
stock config.

::

    sudo apt-get remove --purge bind9
    sudo apt-get install bind9

You will probably want your DNS server to act as a forwarder, using your
provider's nameservers. In that case you can add to your bind9
configuration the following:

::

    sudo editor /etc/bind/named.conf.options

    # These are the IPs for Google DNS servers, substitute them with the
    # ones provided by your ISP.
    forwarders {
        8.8.8.8
        8.8.4.4
    };

Restart the bind9 service to apply your changes, if anything goes wrong
check your logs in /var/log/syslog:

::

    sudo service bind9 restart
    * Stopping domain name service... bind9
    waiting for pid 2321 to die
    ...done.
    * Starting domain name service... bind9
    ...done.

Once your DNS server is set up, you probably don't want it to be visible
to the outside world. You can block incoming request with your firewall.

::

    sudo ufw deny from any to 94.23.12.211 port 53


SSH
===

For your deployment purposes and other tasks, you will probably need
direct access to your lxc containers via ssh. You can achieve this by
setting up a SSH tunnel on your host, making your container accessible
from anywhere.  You will need a different port for each of your containers
you want to access.  It is sadly not possible to route your SSH connection
based on the hostname like a webserver does.

Let's say your container's IP is 10.0.3.101 and you wish to access to your
container via the port 22101. On your host machine, run the following:

::

    ssh -L 0.0.0.0:22101:10.0.3.101:22 localhost
    # Don't forget to open the port to the outside
    sudo ufw allow 22101

Then from the outside you can run:

::

    ssh myusername@domain.of.host -p 22101

For further convinience, edit your .ssh/config file to set up default
values for this connection. This assume that you give your containers
unique domain names either in your DNS configuration or in /etc/hosts.

::

    editor ~/.ssh/config

    # Add the following
    Host mycontainer.domain
        User myusername
        Port 22101

From now on you can log in you container with:

::

    ssh mycontainer.domain


Mosh
====

SSH is a fine tool for accessing your servers, it's installed everywhere,
it's secure and has become the standard on Unix servers. Unfortunately,
SSH is unfit for a mobile usage. Changing you IP, waking up from suspend,
or losing your Wifi signal for too long will likely kill your SSH session
and you'll have to reconnect to your server. Now enter the world of Mosh.
Mosh stands for Mobile Shell and does not suffer from these issues, it's
the perfect tool for staying connected to your server while travelling on
a train for example.

A recent version of Mosh (1.2.4 at the time of this writing) is provided
in Ubuntu.

::
    sudo apt-get install mosh

You will have to install mosh on both your servers and your clients in
order to make a connection.

Once mosh is installed on both, you can connect the same way you would
using SSH::

    mosh domain.com

Mosh connections are kept alive as long as both the client and the server
are running. If you close a client then the following message will appear
on your server when you reconnect::

    Mosh: You have a detached Mosh session on this server (mosh [32015]).

While this vocabulary sounds similar to what you can read with screen or
tmux, you can't reconnect to this detached session for security reasons.If
you want to keep sessions at all times, you can combine mosh with tmux for
best results.

When you get a detached session on your host, you can get rid of it by
killing the pid indicated by the warning message::

    sudo kill 32015
