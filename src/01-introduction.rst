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



Creating a container for different architectures
------------------------------------------------

**i386 containers on 64bit hosts:**



**Other architectures:**

While LXC is not a virtualization technology, it is possible to combine it 
with qemu's user space CPU emulation to create containers for architectures 
different than your own.

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

Since the CPU will be emulated by qemu, don't expect performances even close to
the real hardware. 
