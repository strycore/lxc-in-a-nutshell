*********************
Setting up containers
*********************

Container caveats
=================

Running Fedora, CentOS containers
---------------------------------

Distributions like Fedora, CentOS, Red Hat or Oracle don't run AppArmor
which is likely to cause some issues when starting a container on host
that do use it.

Before starting such system check your container config for the lines::

    # When using LXC with apparmor, uncomment the next line to run unconfined:
    lxc.aa_profile = unconfined

Uncomment the line if needed, then start you container. This behavior
should become the default in future versions of LXC.


Automating updates
==================

::
    sudo apt-get install unattended-upgrades

By default, automatic upgrades won't be enabled, you can do that by
running::
    sudo dpkg-reconfigure -plow unattended-upgrades

Or simply by creating the file /etc/apt/apt.conf.d/20auto-upgrades with
the following contents::
    APT::Periodic::Update-Package-Lists "1";
    APT::Periodic::Download-Upgradeable-Packages "1";
    APT::Periodic::AutocleanInterval "1";
