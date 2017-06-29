Use case: Redmine issue tracker
===============================

Install some required dependencies::

    sudo apt-get install mysql-server libmysqlclient-dev \
        libcurl4-openssl-dev libxml2-dev libxlst-dev libpq-dev \
        libmagickcore-dev libmagickwand-dev git subversion


Installing RVM::

    sudo -s
    curl -L https://get.rvm.io | bash -s stable --ruby=2.1.2
    echo "source /usr/local/rvm/scripts/rvm" >> ~/.bashrc

We will use `Passenger`_ to serve the Redmine application. Passeger is
able to download and compile nginx for us (with its own module enabled)

To install Passenger along with Nginx run::

    gem install passenger --no-ri --no-rdoc
    passenger-install-nginx-module

.. _Passenger: https://www.phusionpassenger.com

Install an init script to handle nginx::

    curl https://raw.githubusercontent.com/jnstq/rails-nginx-passenger-ubuntu/master/nginx/nginx -o /etc/init.d/nginx
    chmod +x /etc/init.d/nginx
    update-rc.d nginx defaults

Configure nginx::
   server {
      listen 80;
      server_name www.yourhost.com;
      root /somewhere/public;   # <--- be sure to point to 'public'!
      passenger_enabled on;
   }

Get latest stable version of Redmine


Install Redmine dependencies
    cd /srv/redmine
    bundle install

Troubleshooting
---------------

Passenger errors
^^^^^^^^^^^^^^^^

To enable detailled error messages from Passenger, you can add the
following directive to your nginx config::

    passenger_friendly_error_pages on;

If you activate friendly error pages, don't forget to set the directive
to `off` or to completely remove it when your Redmine instance is ready
for production!

Getting errors reports from nginx logs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you get some errors with your Redmine instance you can probably get
detailled information in nginx error logs::

    tail -n 200 /opt/nginx/logs/error.log


