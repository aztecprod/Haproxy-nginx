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

![image](https://github.com/aztecprod/Haproxy-nginx/assets/25949605/514ecc8b-0709-4954-8653-4a493093c7ee)
