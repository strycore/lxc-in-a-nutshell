LXC Web Panel
-------------


Install it with:

::
    wget http://lxc-webpanel.github.io/tools/install.sh -O - | bash

What this script does:
    - Downloads LWP and installs it in /srv/lwp/
    - Installs a service to automatically start LWP at boot time.


Update:

::
    wget http://lxc-webpanel.github.io/tools/update.sh -O - | bash


Setting up nginx as a reverse proxy with custom subdomain and HTTPS

SSL certificate creation is explained in the "Setting up HTTP servers"
chapter.

::
    upstream lxc {
        server localhost:5000 fail_timeout=0;
    }

    server {
        listen 80;
        server_name lxc.mydomain.com;
        return 301 https://$host$request_uri;
    }

    server {
        listen 0.0.0.0:443;
        server_name lxc.strycore.com;
        access_log /var/log/nginx/lxc.mydomain.com.access.log;
        error_log  /var/log/nginx/lxc.mydomain.com.error.log;

        ssl on;
        ssl_certificate /etc/nginx/ssl/lwp.crt;
        ssl_certificate_key /etc/nginx/ssl/lwp.key;

        root       /usr/share/nginx/html;
        index      index.html;

        location / {
            proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
            proxy_redirect off;
            proxy_buffering off;
            proxy_set_header        Host             lxc.mydomain.com;
            proxy_set_header        X-Real-IP        $remote_addr;
            proxy_set_header        X-Forwarded-For  $proxy_add_x_forwarded_for;
            proxy_redirect http:// https://;
            if (!-f $request_filename) {
                proxy_pass http://lxc;
                break;
            }
        }
    }

