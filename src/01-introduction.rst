************
Introduction
************


Before you start
================

 - Ubuntu Server
 - New (and stock) kernel

Installing LXC
==============

::
    sudo apt-get install lxc

If you're on an older version of Ubuntu than 14.04, you can get the latest
version by using the PPA providing the latest stable release:

::

    sudo add-apt-repository ppa:ubuntu-lxc/stable
    sudo apt-get update
    sudo apt-get install lxc

Creating your first container
=============================

In order to create a container, you have to give it a name (with the -n
parameter) and a template (with the -t parameter). If this is the first
time you use a template, it may take a few minutes to download system
files and set them all up, but as these will all be cached, any container
based off the same template will be created almost instantly.

::
    sudo lxc-create -t ubuntu -n mycontainer

Start the container

::
    sudo lxc-start -n ubuntu -d

Login in container

::
    sudo lxc-console -n ubuntu



Creating alternative containers
===============================

LXC is not a virtualization system but it is not limited to a single Linux
distribution or architecture. You have a wide arrary of choices when
creating your containers.

Installing a different distributions
------------------------------------

LXC provides a whole set of Linux distributions ready to install with
numerous template scripts. All major distributions (Ubuntu, Fedora,
Debian, Gentoo, Archlinux, Opensuse) are included and even
some lesser known ones such as Plamo, Altlinux or Alpine.

Choosing one of those templates is a mandatory step in the creation of a
template, given by the `-t` argument of the `lxc-create` command.

Most templates provide their own set of options, to use them write them
following two dashes `--` after passing all necessary arguments to
`lxc-create`. For example, let's create a debian container running the
unstable branch::

    sudo lxc-create -t debian -n mycontainer -- -r sid

Installing a different version of your host
-------------------------------------------

This will probably be the most useful feature when creating containers
that differ from your host. While many Linux distributions share a lot in
common at any given time, there will be big differences between the same
distrbitions taken at two different points in time. Some software may not
be updated with the latest technology and may require you to run previous
version of your OS, or, on the contrary, you may want to try a bleeding
edge technology on an OS that is still in beta.

To create a container with a different version of Ubuntu, run::

    sudo lxc-create -n mycontainer -t ubuntu -- -r precise

This command will create a container with Ubuntu 12.04 LTS (codenamed
Precise Pangolin). As you can see, the -r parameter is specific to the
Ubuntu template, hence it is given after the -- part.

i386 containers on 64bit hosts
------------------------------



Other architectures
-------------------

While LXC is not a virtualization technology, it is possible to combine it
with qemu's user space CPU emulation to create containers for
architectures different than your own.

First you'll have to install the qemu-user-static package:

::
    sudo apt-get install qemu-user-static

Then passing the right arguments to lxc-create will build an ARM container.
The following command will create an armfs container with Ubuntu 12.04:

::
    sudo lxc-create -n myarmcontainer -t ubuntu -- -a armhf -r precise

Possible architectures are:

- armhf (ARM hard float or ARMv7)
- armel (ARM)
- arm64 (64bit ARM)
- powerpc (TODO confirm this)

Since the CPU will be emulated by qemu, don't expect performances even
close to the real hardware.

Unprivileged containers
-----------------------

Be careful we you mount your unprivileged containers!
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You may want to symlink or mount --bind your ~/.local/share/lxc on
another drive with more space. If you do so, make sure that the
destination drive is mounted without the `nosuid` flag otherwise you
won't be able to get root access in your container.
