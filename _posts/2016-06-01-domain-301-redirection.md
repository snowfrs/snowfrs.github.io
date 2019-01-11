---
title: 301 redirection
tags:
  - Nginx
---
<!--more-->
```
    server {
            server_name mysite.com;
            return 301 $scheme://www.mysite.com$request_uri;
    }
    server {
            server_name     www.mysite.com;
    
            root /usr/share/nginx/mysite.com;
            index index.html index.htm index.php;
    
            location / {
                    try_files $uri $uri/ /index.html index.php;
            }
    
            location ~ \.php$ {
                    fastcgi_split_path_info ^(.+\.php)(/.+)$;
                    fastcgi_pass unix:/var/run/php5-fpm.sock;
                    fastcgi_index index.php;
                    include fastcgi_params;
            }
    }
```
