****************
The host machine
****************

Setting up LXC containers will require you to have a dedicated server to
run your containers on. While this book targets Ubuntu Server 14.04 as the
host system, any Linux distribution with a modern kernel should be able to
host your containers.

Basic setup
===========

In these steps, we'll review the basic setup for your host server. This is
not related to LXC in any way but these are common steps usually taken
after the OS has finished installing.

First you'll probably need a few packages, install whatever tools you need
to feel at home. For example::

    apt-get install git curl vim-nox zsh

Then create a user, since your provider probably gave you root access and
we don't want to directly log in as root. Add this user to the sudo group
in order to have admin rights with the newly created user.

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

Check your locale settings in /etc/environment and /etc/default/locale,
update the LANG setting if needed then update your locales:

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

Dotfiles
========

What dotfiles refer to are a bunch of files, starting with a dot (making
them hidden by default on Unix systems), used to configure various
programs. Here are some examples of common dotfiles with their usage:

- .zshrc: Settings for the zsh shell, includes environment variables,
  aliases, prompt customisation, functions and other useful setups.
- .bashrc: Same as .zshrc but for the bash shell
- .gitconfig: Global configuration for git that will be used on all of
  your projects. It usually contains your name and email along with some
  handy aliases.
- .tmux.conf: Configuration file for the tmux screen multiplexer
- .vimrc: Configuration file for the vim editor

It is highly encouraged that you keep your dotfiles under version
control. That way you can very quickly deploy a working environment you
are confortable with in a matter of minutes and you will have the same
environment across all your machines. You can keep your dotfiles in a
public git repository such as GitHub or keep them in your own private
git repo (we will explain how to setup a private git repository later on
in this book). Once you have your dotfiles versionned in a single folder,
you can create symbolic links to their real emplacement. Let's stay you
have stored your dotfiles in ~/.dotfiles/, you can use your versionned
.zshrc by linking it::

    ln -s ~/.dotfiles/zshrc ~/.zshrc

You can even make the link creation process into a small shell script to
automate this tedious task.

CGroups
=======

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
===========

Your host machine can act as a HTTP proxy for your containers. While any
HTTP should get the job done, we'll be using nginx which is a good
candidate for this task.

If you have any SSL certificates on your domains, you'll have to set them
on your host. See the paragraph below on how to setup a SSL certificate
for one of your domains.

SSL Certificates
================

Regarding SSL certificates, you have two options available. You either
generate a self-signed certificate or you get a certificate from a
trusted authority, which you usually pay for. While self-signed
certificates are ok for websites that you use privately such as your
Redmine or Jenkins instance, it's better to get a verified certificate
for your domains accessible to the public.
There are a lot of offers available in the world of SSL certificates
ranging from very affordable (like Gandi) to ridiculously expensive
(Verisign/Symantec).

For basic needs (securing a single domain), you should find offers below
$30 per year. If you are paying more, you are probably getting ripped off
by some unscrupulous authority.

Also note that certificates for a star domain (\*.example.com) are way
more expensive than certificates for a single domain (which usually
includes the www. subdomain as well). You might want to take this into
account when building your website if you're not willing to spend a
fortune on certificates and avoid creating subdomains such as
api.mydomain.com or admin.mydomain.com.

It should also be noted that you have two free options to have a
Certificate delivered by an authority. The first one is StartSSL, which
will give you 1 year valid certificates for free.
The other one is CACert, which is a community-driven Certificate
Authority. While CACert certificates are free, their authority is not
recognised by default by all major web browser, which offer little
advantage over a self-signed certificate. It is therefore recommended not
to use CACert for public-facing websites (unless you want your visitors
to get a big fat nasty warning when they visit your website).

Creating the Certificate Signing Request
----------------------------------------

Once you've decided which solution is better for you website, let's get on
with setting this thing up. Whether you have chosen to create a
self-signed certificate or to get one from an authority, the process
always starts by creating a Certificate Signing Request (CSR) and a key::

    openssl req -nodes -newkey rsa:2048 -keyout myserver.key -out myserver.csr

You will be asked a bunch of questions about you and your website, this
information will be attached to your certificates so your visitors will
be able to verify your identity.

::
    Generating a 2048 bit RSA private key
    .....+++
    ........+++
    unable to write 'random state'
    writing new private key to 'myserver.key'
    -----
    You are about to be asked to enter information that will be incorporated
    into your certificate request.
    What you are about to enter is what is called a Distinguished Name or a DN.
    There are quite a few fields but you can leave some blank
    For some fields there will be a default value,
    If you enter '.', the field will be left blank.
    -----
    Country Name (2 letter code) [AU]:US
    State or Province Name (full name) [Some-State]:California
    Locality Name (eg, city) []:Los Angeles
    Organization Name (eg, company) [Internet Widgits Pty Ltd]:My company
    Organizational Unit Name (eg, section) []:
    Common Name (e.g. server FQDN or YOUR name) []:myserver.com
    Email Address []:me@myserver.com

    Please enter the following 'extra' attributes
    to be sent with your certificate request
    A challenge password []:
    An optional company name []:My company


Setting up a secure site in nginx
---------------------------------

Gather your certificate files someplace safe on your server, I like to
keep mine in /etc/ssl/sites/. You should at least have a .crt and a .key
file and also maybe a .pem file (If your Certificate Authority gave you
an intermediate certificate).

Once you got your certificate ready, open the nginx config file
corresponding to your domain name and make sure you have the following::

    listen x.x.x.x 443 ssl;

Replace the dummy IP address x.x.x.x with your public facing IP, then add
the following lines in the same server section::

    ssl_certificate /etc/ssl/sites/myserver.crt;
    ssl_certificate_key /etc/ssl/sites/myserver.key;

In the event where your authority has issued an intermediate certificate
(usually as a .pem file), you can use it by appending it to your main
certificate::

    cat myserver_intermediate.pem >> myserver.crt

When everything is correctly setup, you can restart nginx::

    servcice nginx restart
