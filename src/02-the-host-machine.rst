The host machine
================

Setting up LXC containers will require you to have a dedicated server to run
your containers on. While this book targets Ubuntu Server 14.04 as the host
system, any Linux distribution with a modern kernel should be able to host your
containers.

Basic setup
-----------

In these steps, we'll review the basic setup for your host server. This is not
related to LXC in any way but these are common steps usually taken after the OS
has finished installing.

First you'll probably need a few packages, install whatever tools you need to
feel at home. For example:

::
    apt-get install git curl vim-nox zsh

Then create a user, since your provider probably gave you root access and we
don't want to directly log in as root. Add this user to the sudo group in order
to have admin rights with the newly created user.

::
    adduser <username> --shell /usr/bin/zsh
    usermod -a -G sudo <username>

You can now terminate your root SSH session and log back in as this user.

::
    # exit
    Connection to hostname.com closed.
    $ ssh username@hostname.com

If you have SSH and GPG keys attached to your identity, now is a good time to
import them in ~/.ssh and ~/.gnupg.

::
    # Run this on your host server:
    mkdir ~/.ssh
    mkdir ~/.gnupg

    # Run this on you machine where you have your keys:
    cd ~/.ssh
    scp id_rsa* username@hostname:~/.ssh
    cd ~/.gnupg
    scp *.gpg username@hostname:~/.gnupg

If you want to log in you server without needing a password, copy your SSH
public key in the ~/.ssh/authorized_keys file on your host. This command will
take care of the process just fine:

::
    ssh-copy-id username@hostname

**Fixing locales issues**

If you encounter error messages such as:

::
    perl: warning: Falling back to the standard locale ("C").
    locale: Cannot set LC_ALL to default locale: No such file or directory

Check your locale settings in /etc/environment and /etc/default/locale, update
the LANG setting if needed then update your locales:

::
    sudo locale-gen en_US en_US.UTF-8
    sudo dpkg-reconfigure locales

If you still get errors, check your locales settings:

::
    $ locale
    LANG=en_US.UTF-8
    LANGUAGE=
    LC_CTYPE=en_US.UTF-8
    LC_NUMERIC=fr_FR.UTF-8
    LC_TIME=fr_FR.UTF-8
    LC_COLLATE=en_US.UTF-8
    LC_MONETARY=fr_FR.UTF-8
    LC_MESSAGES=en_US.UTF-8
    LC_PAPER=fr_FR.UTF-8
    LC_NAME=fr_FR.UTF-8
    LC_ADDRESS=fr_FR.UTF-8
    LC_TELEPHONE=fr_FR.UTF-8
    LC_MEASUREMENT=fr_FR.UTF-8
    LC_IDENTIFICATION=fr_FR.UTF-8
    LC_ALL=

and install missing language packs:

::
    sudo apt-get install language-pack-fr

CGroups
-------

Your kernel should have cgroup support enabled for LXC to work properly.
This should already be the case if you are running a stock Ubuntu kernel
but bear in mind that some dedicated servers ship with a custom kernel
that don't support cgroups. If this is the case, the simplest way of
getting a supported kernel is installing the stock kernel:

::
    apt-get install linux-image-server

If your dedicated server came with an custom kernel, then there probably
is a grub config file to tell the boatloader to load this kernel. In the
following example, we see that the French provider OVH ships with a custom
kernel incompatible with LXC, you should move the corresponding grub
config file out of the way:

::
    mv /etc/grub.d/06_OVHkernel ~

Once you are done with that, you'll have to regenerate your grub config
then reboot:

::
    update-grub
    reboot

If everything when as expected, you should now run the stock kernel:

::
    # uname -a
    Linux lxchost 3.11.0-17-generic #31-Ubuntu SMP Mon Feb 3 21:52:43 UTC 2014 x86_64 x86_64 x86_64 GNU/Linux

Check that cgroup is correctly mounted, you should see something similar
to the following:

::
    # mount | grep cgroup
    none on /sys/fs/cgroup type tmpfs (rw,relatime,size=4k,mode=755)
    cgroup on /sys/fs/cgroup/cpuset type cgroup (rw,relatime,cpuset,clone_children)
    cgroup on /sys/fs/cgroup/cpu type cgroup (rw,relatime,cpu)
    cgroup on /sys/fs/cgroup/cpuacct type cgroup (rw,relatime,cpuacct)
    cgroup on /sys/fs/cgroup/memory type cgroup (rw,relatime,memory)
    cgroup on /sys/fs/cgroup/devices type cgroup (rw,relatime,devices)
    cgroup on /sys/fs/cgroup/freezer type cgroup (rw,relatime,freezer)
    cgroup on /sys/fs/cgroup/blkio type cgroup (rw,relatime,blkio)
    cgroup on /sys/fs/cgroup/perf_event type cgroup (rw,relatime,perf_event)
    cgroup on /sys/fs/cgroup/hugetlb type cgroup (rw,relatime,hugetlb)
    systemd on /sys/fs/cgroup/systemd type cgroup (rw,nosuid,nodev,noexec,relatime,name=systemd)

HTTP Server
-----------

Your host machine can act as a HTTP proxy for your containers. While any
HTTP should get the job done, we'll be using nginx which is a good
candidate for this task.
