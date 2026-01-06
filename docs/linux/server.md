---
title: Server
icon: fontawesome/solid/server
---

## Setup proxy and certbot

1) Setup domain to target your server IP.

2) Nginx proxy config (without SSL):

```
server {
    listen 80;
    server_name <domain>.eu;

    # Logování (volitelné)
    access_log /var/log/nginx/<app_name>_access.log;
    error_log /var/log/nginx/<app_name>_error.log;

    location / {
        proxy_pass http://127.0.0.1:<port>;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    client_max_body_size 20M;
}
```

3) Apply

```console
# nginx -t  
# sudo systemctl reload nginx
```

4) Certbot SSL

```console
# certbot --nginx -d <domain>.eu
```

5) Apply again
