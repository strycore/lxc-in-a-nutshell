*******
Ansible
*******


Inventory
=========

List your containers in /etc/ansible/hosts

::
    [containers]
    container1
    container2
    container3
    container4

Running commands
================

You can run ad-hoc commands to test your inventory or run quick one shot 
tasks.

::
    ansible containers -a "cat /etc/hostname"

When running commands requiring sudo, add add some parameters to be able
to give the password to ansible::
    ansible containers -a "apt-get update" --sudo --ask-sudo-pass


