*********************
Setting containers up
*********************

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
