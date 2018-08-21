## nginx.conf配置
```shell

http {
    log_format  main  '$remote_addr - [$time_local] $http_host $request_method "$uri" "$query_string" '
                  '$status $request_length $body_bytes_sent $upstream_status $upstream_addr $request_time $upstream_response_time '
                  '"$http_user_agent" "$http_cookie" "$http_x_forwarded_for" "$http_referer"' ;

    access_log  /var/log/nginx/access.log  main;
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    gzip  on;
    gzip_min_length 1k;
    gzip_vary on;
    gzip_types text/plain text/css application/javascript application/x-javascript application/json application/vnd.ms-excel;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;
}
```
## conf.d目录的子文件配置
例子1：
接口只支持https协议的访问
有2个实例做负载！

```shell
upstream user {
    server 192.168.1.175:9995;
    server 192.168.1.176:9995;
}
server {
    listen       443 ssl;
    server_name  api.xxxxxx.cn;

    ssl_certificate   cert/api.pem;
    ssl_certificate_key  cert/api.key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

    location ^~ /user/ {
        proxy_pass http://user/user/;
        proxy_read_timeout 30s;
        proxy_send_timeout 30s;
        proxy_set_header Host            $host;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```
例子2：
文件下载
```shell
server {
        listen       80;
        server_name  file.xxxxx.cn;
        location ^~ / {
            alias   /local/file/;
        }
    }
```
例子3：
静态资源访问
```shell
server {
    listen       80;
    server_name  front.xxxxxx.cn;

    location ^~ /refer/ {
        rewrite '(.*)' /index.html last;
    }

    location ^~ /home {
        alias   /local/html/home;
        index  index.html index.htm;
    }

    location ^~ /map {
        alias /local/html/map;
        index  index.html index.htm;
        if ($request_filename ~* .*.(html|htm)$)
        {
          add_header Cache-Control no-cache,no-store;
        }
    }
}
```
例子4：
websocket协议
```shell
server {
    listen       80;
    server_name  front.xxxxxx.cn;

    location ^~ /sock {
           proxy_pass http://127.0.0.1:80;
           proxy_http_version 1.1;
           proxy_set_header Host            $host;
           proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
           proxy_set_header Upgrade $http_upgrade;
           proxy_set_header Connection "upgrade";
    }
}
```
