Use case: znc IRC bouncer
=========================

Installing znc::

    sudo apt-get update
    sudo apt-get install wget build-essential libssl-dev libperl-dev
    # For Python support
    sudo apt-get install pkg-config python3-dev
    wget http://znc.in/releases/znc-1.4.tar.gz
    tar xvzf znc-1.4.tar.gz
    cd znc-1.4
    ./configure --enable-python
    make

ZNC-push
--------

ZNC-push allows you to receive realtime notifications from ZNC to your
Android devices. It will send you push notifications everytime someone
highlights you or sends you a private message.

Installing ZNC-push::

    sudo apt-get install libcurl4-openssl-dev
    git clone https://github.com/jreese/znc-push.git
    make
    sudo make install

