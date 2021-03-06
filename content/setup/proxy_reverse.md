+++
date = "2015-12-05T16:00:21-08:00"
draft = false
title = "Reverse Proxy"
weight = 11
menu = "installation"
toc = true
+++

# Overview

Using a reverse proxy with Drone is entirely optional. When using a reverse proxy with Drone it is extremely important to properly configure the `X-Forwarded-Proto` and `X-Forwarded-For` header variables. You must also change the Drone container's port mapping from `--publish=80:8000` to `--publish=8000:8000`.

# Caddy

Example [caddy server](https://caddyserver.com/) reverse proxy configuration:

```
drone.mycomopany.com {
    proxy / localhost:8000 {
        proxy_header X-Forwarded-Proto {scheme}
        proxy_header X-Forwarded-For {host}
        proxy_header Host {host}
    }
}
```

# Nginx

Example nginx reverse proxy configuration:

```
location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $remote_addr;
    proxy_set_header X-Forwarded-Proto $scheme;
    proxy_set_header Host $http_host;
    proxy_set_header Origin "";

    proxy_pass http://127.0.0.1:8000;
    proxy_redirect off;
    proxy_http_version 1.1;
    proxy_buffering off;

    chunked_transfer_encoding off;
}
```

# Apache2

Example Apache2 reverse proxy configuration:

```
<VirtualHost *:80>

	ServerName drone.example.com
	ServerAlias drone

	ServerAdmin webmaster@localhost

	ErrorLog ${APACHE_LOG_DIR}/drone-error.log
	CustomLog ${APACHE_LOG_DIR}/drone-access.log vhost_combined

	#-#ProxyPass / http://127.0.0.1:8080/
	#-#ProxyPassReverse / http://127.0.0.1:8080/
	ProxyPass         / http://localhost:8080/ nocanon
	ProxyPassReverse  / http://localhost:8080/
	RequestHeader set X-Forwarded-Proto "http"
	RequestHeader set X-Forwarded-Port "80"
	ProxyRequests     Off
	#ProxyRequests     On	#not needed for reverse proxy
	ProxyPreserveHost On

	AllowEncodedSlashes NoDecode
	SetOutputFilter INFLATE;proxy-html;DEFLATE
	ProxyHTMLURLMap http://drone.example.com/ /
</VirtualHost>

```
