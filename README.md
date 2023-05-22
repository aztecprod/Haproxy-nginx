# Балансировка нагрузки. HAProxy/Nginx - Александр Шевцов
![image](https://github.com/aztecprod/Haproxy-nginx/assets/25949605/34b30466-7a47-4387-a19b-2bd00909b26c)
Балансировка нагрузки (load balancing) — это процесс распределения нагрузки на пул серверов. 

Распределение происходит на L4(Транспортный) или L7(Прикладной) уровнях модели OSI.
![image](https://github.com/aztecprod/Haproxy-nginx/assets/25949605/2cfbe097-2c38-426a-a46d-f7832caeb782)
Round Robin - Запросы распределяются по пулу сервером последовательно. Лучше всего применять в случае, когда у нас имеются все сервера одинаковой мощности.

Weight Round Robin - Запросы распределяются по пулу сервером в зависимости от его веса. Чем больше вес, чем чаще туда будет распределяться нагрузка. Лучше всего применять, когда сервера имеют разные мощности.
![image](https://github.com/aztecprod/Haproxy-nginx/assets/25949605/2f1f8a9f-6a89-4b3e-8df1-02b0e2a142f4)
![image](https://github.com/aztecprod/Haproxy-nginx/assets/25949605/d2da618f-59d5-40c5-8c79-f9c74de62a2a)
![image](https://github.com/aztecprod/Haproxy-nginx/assets/25949605/16a86125-d818-4409-bc53-310122f30ca5)
![image](https://github.com/aztecprod/Haproxy-nginx/assets/25949605/c3d98320-8af8-4c9f-9298-d4d0bc95085a)
![image](https://github.com/aztecprod/Haproxy-nginx/assets/25949605/92e81722-6f2f-43cb-a388-0172e611205e)

Настройка nginx:
```

user  nginx;
worker_processes  auto;

error_log  /var/log/nginx/error.log notice;
pid        /var/run/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    include /etc/nginx/conf.d/*.conf;

server {
listen localhost:8088; # простая директива
location /ping {
return 200 'Nginx is configured correctly';

}
}

}

```
![image](https://github.com/aztecprod/Haproxy-nginx/assets/25949605/5b798151-72e8-48fc-b4a6-ba87eaa6a533)

![image](https://github.com/aztecprod/Haproxy-nginx/assets/25949605/88a4dc5a-d413-4228-95a1-cca6d19aa6ad)

Настройка Haproxy:

```
global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

        # Default SSL material locations
        ca-base /etc/ssl/certs
        crt-base /etc/ssl/private

        # See: https://ssl-config.mozilla.org/#server=haproxy&server-version=2.0.3&config=intermediate
        ssl-default-bind-ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256:DHE>
        ssl-default-bind-ciphersuites TLS_AES_128_GCM_SHA256:TLS_AES_256_GCM_SHA384:TLS_CHACHA20_POLY1305_SHA256
        ssl-default-bind-options ssl-min-ver TLSv1.2 no-tls-tickets

defaults
        log     global
        mode    http
        option  httplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000
        errorfile 400 /etc/haproxy/errors/400.http
        errorfile 403 /etc/haproxy/errors/403.http
        errorfile 408 /etc/haproxy/errors/408.http
        errorfile 500 /etc/haproxy/errors/500.http
        errorfile 502 /etc/haproxy/errors/502.http
        errorfile 503 /etc/haproxy/errors/503.http
        errorfile 504 /etc/haproxy/errors/504.http

frontend nginx
        bind localhost:8080
        default_backend mynginx

backend mynginx
        balance roundrobin
        mode http
#       option httpchk
        server web1 localhost:8088
#       server web1 192.168.0.108:80

```
![image](https://github.com/aztecprod/Haproxy-nginx/assets/25949605/e224defd-064c-46ef-a5b4-f0dec2344075)
