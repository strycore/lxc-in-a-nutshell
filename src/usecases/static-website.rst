Use case: Static website
========================

This container will host a very simple website consisting only of static
files. We will be using Nginx as the webserver.

Setting up nginx
----------------

First step is to install nginx::
    sudo apt-get install nginx

The default install will provide everything you need to get everything
running without any hassle. The web server's configuration is located
under /etc/nginx/nginx.conf but we won't need to tweak that.
What does need a bit of configuration is the virtual host you'll be
hosting in your container, we will use the default one, which is located
under /etc/nginx/sites-available/default.

If you look into /etc/nginx you'll notice two different folders:
sites-available and sites-enabled. The sites-enabled contains symlinks to
virtual hosts stored in sites-available, allowing you to enable or
disable a website by creating or removing a symlink.

A basic nginx configuration is quite short and simple to understand,
here's one as an example.

::
    server {
        listen 80 default_server;
        root /srv/mywebsite;
        index index.html;
        server_name example.com www.example.com;

        location / {
            try_files $uri.html $uri $uri/ =404;
        }
    }

Let's break this configuration apart to see what's going on here.
The whole configuration is contained in a server {} block, try to keep a
single one server block per file in order to take advantage of the
sites-available/enabled system (if your website has both HTTP and HTTPS
and requires two `server` blocks, you can keep those in the same file).

The first directive `listen 80` tells nginx on which port the website
should be made available, it's usually always 80 for HTTP and 443 for
HTTPS. The `default_server` tells nginx to choose this website if there
are ambiguities (for example, a user access the server by its IP address
instead of the domain name and several websites hosted on a single IP
address). Since we are using a container with a single website, this
parameter could be removed.

The `root` directive is very important to set to a valid value. It should
point to a valid directory containing your website files.

`index` point to the file used when a request is made on the base url.
Here, a request to `http://example.com` would serve the
`/srv/mywebsite/index.html` file.

`server_name` usually contains your domain names or aliases (the domain
with a www. prefix or other subdomains).

Next comes another block which specifies the configuration used for a
particular URL prefix. Having a single `location` block for `/` tells
nginx to use this for any URL on the website. You could have more
`location` block, pointing to prefixes such as `/blog` or `/wiki` and
providing a totally different configuration.

Our `location` block contains a single directive which will translate
URLs requested to a file on the server. If one makes a request to
`http://example.com/mypage` then nginx will try to serve (in that order)
the files /srv/mywebsite/mypage.html, /srv/mywebsite/mypage and
/srv/mywebsite/mypage/index.html. If none of these files exist on the
server then nginx will return a 404 error.

Listing files of a folder
-------------------------

By default, if you request the URL `http://example.com/images/` and
/srv/mywebsite/images is a folder not containing an `index.html` file (or
whatever the value of the `index` directive is) then nginx will return a
403 Forbidden error. If you wish to authorize viewing the contents of a
folder then add a `autoindex on` directive inside your main `server` block
or inside a particular `location` block.

