---
layout: post
title:  "Angular 2 and ExpressJS with Nginx"
date:   2017-02-20 00:00:00 -0500
---
Content of `/etc/nginx/sites-enabled/default`

{% highlight bash %}
# HTTP - redirect all requests to HTTPS:
server {
        listen 80;
        listen [::]:80 default_server ipv6only=on;
        return 301 https://$host$request_uri;
}

# HTTPS - proxy requests on to local Node.js app:
server {
  listen 443;
  server_name tobymurray.ca;

  # Serve the Angular 2 application directly as static files
  root /home/toby/client/dist;

  # add Strict-Transport-Security to prevent man in the middle attacks
  add_header Strict-Transport-Security "max-age=31536000";

  ssl on;
  # Use certificate and key provided by Let's Encrypt:
  ssl_certificate /etc/letsencrypt/live/tobymurray.ca/fullchain.pem;
  ssl_certificate_key /etc/letsencrypt/live/tobymurray.ca/privkey.pem;
  ssl_session_timeout 5m;
  ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
  ssl_prefer_server_ciphers on;
  ssl_ciphers 'EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH';

  # Pass all requests for /api/ to the Express application at localhost:3000
  location /api/ {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-NginX-Proxy true;
    proxy_pass http://localhost:3000/;
    proxy_ssl_session_reuse off;
    proxy_set_header Host $http_host;
    proxy_cache_bypass $http_upgrade;
    proxy_redirect off;
  }
}
{% endhighlight %}
