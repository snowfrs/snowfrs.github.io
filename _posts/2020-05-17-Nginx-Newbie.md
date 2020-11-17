---
title: Nginx Newbie
tags: Web Nginx
---
[Nginx][nginx]

Nginx 已被证明具有极高的稳定性、高性能和高可靠性，并提供了一套强大的工具.
<!--more-->
与 [Apache][apache] 相比, Nginx 具有的优势:

+ 轻量
+ 占用内存少
+ 配置项更丰富
+ 高负载下性能更好

## Installation
官方 repo 配置 请参考 [nginx.org](http://nginx.org/en/linux_packages.html)

## Configuration
`/etc/nginx/nginx.conf` 包含全局配置文件

`/etc/nginx/conf.d` 目录下包含站点配置文件

## Performance
```txt
1. 启用gzip压缩
2. 添加upstream响应时间
3. 为静态内容设置适当的过期时间
4. 在启用HTTPS的站点总是启用HTTP/2
```

## Caching

## Security
Nginx 有许多提供Web Application Firewall(WAF)的模块，但是几乎所有模块都需要从源码编译 Nginx.

Naxsi 和 modsecurity 是两个流行的选择

## 典型配置
Mollzia提供了一个非常优秀的[Nginx配置生成工具](https://ssl-config.mozilla.org/)

`/etc/nginx/nginx.conf`
```txt
user nginx;  #Use for CentOS
#user www-data;  #Use for Debian
worker_processes auto; #Or set to number of CPU cores
error_log /var/log/nginx/error.log warn;
pid /run/nginx.pid;
include /usr/share/nginx/modules/*.conf;
events {
    worker_connections 1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"'
                      '"$request_time" "$upstream_response_time" $upstream_cache_status';
    access_log  /var/log/nginx/access.log  main;
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;
    server_tokens       off;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    include /etc/nginx/conf.d/*.conf;
}
```

`/etc/nginx/conf.d/example.conf`
```txt
upstream www {
	server unix:/run/php-fpm/www.sock;
}
server {
	listen 80;
	server_name example.com www.example.com;
	return 301 https://$server_name$request_uri;
}
server {
	listen 443 ssl http2;
	server_name example.com www.example.com;

	access_log	/var/log/nginx/example.com.access.log main;
	error_log	/var/log/nginx/example.com.error.log;

	root /path/to/your/web/root;
	index index.php index.html index.htm;
	
        #SSL settings
        #https://ssl-config.mozilla.org/#hsts=false
	ssl_certificate /etc/letsencrypt/live/exmaple.com/fullchain.pem;
	ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;
	ssl_session_timeout 1d;
        ssl_session_cache shared:SSL:50m;
        ssl_session_tickets off;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';
        ssl_prefer_server_ciphers on;
        resolver 8.8.8.8;

        #Performance settings
	server_tokens	off;
	gzip on;
	gzip_vary on;
	gzip_types text/plain text/css text/xml text/javascript application/json application/x-javascript application/javascript application/xhtml+xml application/xml application/atom+xml application/xml+rss font/opentype image/svg+xml image/x-icon;
	gzip_proxied no-cache no-store private expired auth;
	gzip_min_length 1000;
	gzip_disable "msie6";
        gzip_comp_level 6;
        gzip_buffers 16 8k;
        keepalive_timeout 65;
        sendfile on;

	location / {
		try_files $uri $uri/;
		gzip_static on;
	}

	location ~ \.php$ {
		try_files $uri =404;
		fastcgi_intercept_errors on;
		fastcgi_index  index.php;
		include        fastcgi_params;
		fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
		fastcgi_pass   www;
	}
	
       location ~* ^.+\.(ogg|ogv|svg|svgz|eot|otf|woff|mp4|ttf|rss|atom|jpg|jpeg|gif|png|ico|zip|tgz|gz|rar|bz2|doc|xls|exe|ppt|tar|mid|midi|wav|bmp|rtf|js|css|pdf)$ {
	access_log off; 
        log_not_found off; 
        expires max;
        }

	client_max_body_size 8m;
	client_body_timeout 60;

}
```

[nginx]: http://nginx.org/
[apache]: https://httpd.apache.org/