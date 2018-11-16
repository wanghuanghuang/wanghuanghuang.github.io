---
title: Nginx 转发 + 域名配置
date: 2018-10-22 17:34:37
tags:
---
![猫咪老师](/images/页面图片/5.jpeg)
## 服务器中nginx 配置文件位置：
![nginx图片](/images/Nginx 转发_域名配置/nginx图片.png)
### 1.nginx.conf 
```
   a)文件内容：
   worker_processes  1;
   events {
       worker_connections  1024;
   }
   http {
       include       mime.types;
       default_type  application/octet-stream;
       log_format  main  '$remote_addr - $remote_user [$time_local] "$request" 
                         '$status $body_bytes_sent "$http_referer" '
                         '"$http_user_agent" "$http_x_cookie" "$request_time" "$upstream_response_time" "$upstream_cache_status" "$guid"';
       access_log  logs/access.log  main;
       sendfile        on;
       keepalive_timeout  65;
       gzip  on;
       gzip_min_length  1k;         #最小压缩大小
       gzip_buffers     4 16k;        #压缩缓冲区
       gzip_http_version 1.0;       #压缩版本
       gzip_comp_level 2;            #压缩等级
       gzip_types       text/plain application/x-javascript text/css application/xml;           #压缩类型
   
       #引用配置
       include /usr/local/nginx/conf/localhost.conf;
   }
```
### 2.localhost.conf
```$xslt
    a)	文件内容：
    upstream http_webapi_test {
        server 127.0.0.1:8889;
        keepalive 16;
    }
    upstream http_quartz_test {
        server 127.0.0.1:8011;
        keepalive 16;
    }
    upstream http_auth_test {
        server 127.0.0.1:8051;
        keepalive 16;
    }
    upstream http_webapi_insurance {
        server 127.0.0.1:8041;
        keepalive 16;
    }
    upstream http_webapi_risk {
        server 127.0.0.1:8071;
        keepalive 16;
    }
    upstream http_brand_test {
        server 127.0.0.1:8021;
        keepalive 16;
    }
    upstream http_wechat {
        server 127.0.0.1:8061;
        keepalive 16;
    }
    upstream http_policy {
        server 127.0.0.1:8077;
        keepalive 16;
    }
    upstream http_intelligent_cabinet {
        server 127.0.0.1:8201;
        keepalive 16;
    }
    upstream http_intelligent_wechat {
        server 127.0.0.1:8301;
        keepalive 16;
    }
    server {
      listen       80;            若为端口号80访问进入服务
      #server_name  p.boiotech.com;
      server_name  localhost;
      #charset koi8-r;
      set $guid "-";
      #if ( $http_cookie ~* "uid=(\S+)(;.*|$)"){
      if ($http_cookie ~* "CSID=([A-Z0-9]*)"){
      #if($http_cookie ~* "(.*)$"){
          set $guid $1;
      }
      access_log /var/log/nginx/localhost/host.access80_new.log  main;  转发日志
       location ~ /api(\/)+ {			url中带有api转发至此  下面同理
            proxy_redirect          off;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size    10m;
            client_body_buffer_size 128k;
            proxy_buffers           32 4k;
            proxy_connect_timeout   3;
            proxy_send_timeout      60;
            proxy_read_timeout      60;
            proxy_pass http://http_webapi_test;
           #proxy_http_version 1.1;
            proxy_set_header        Connection      "";
      }
      location ~ /api-insurance(\/)+ {
            proxy_redirect          off;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size    10m;
            client_body_buffer_size 128k;
            proxy_buffers           32 4k;
            proxy_connect_timeout   3;
            proxy_send_timeout      30;
            proxy_read_timeout      30;
            proxy_pass http://http_webapi_insurance;
            proxy_http_version 1.1;
            proxy_set_header        Connection      "";
        }
       location ~ /quartz(\/)+ {
            proxy_redirect          off;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size    10m;
            client_body_buffer_size 128k;
            proxy_buffers           32 4k;
            proxy_connect_timeout   3;
            proxy_send_timeout      30;
            proxy_read_timeout      30;
            proxy_pass http://http_quartz_test;
            proxy_http_version 1.1;
            proxy_set_header        Connection      "";
      }
      location ~ /auth(\/)+ {
            proxy_redirect          off;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size    10m;
            client_body_buffer_size 128k;
            proxy_buffers           32 4k;
            proxy_connect_timeout   3;
            proxy_send_timeout      30;
            proxy_read_timeout      30;
            proxy_pass http://http_auth_test;
            proxy_http_version 1.1;
            proxy_set_header        Connection      "";
      }
      location /webSocketServer/ {
            proxy_pass http://http_intelligent_cabinet;
            # WebScoket Support
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }	
      location ~ /api-cabinet(\/)+ {
            proxy_redirect          off;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size    10m;
            client_body_buffer_size 128k;
            proxy_buffers           32 4k;
            proxy_connect_timeout   18000;
            proxy_send_timeout      18000;
            proxy_read_timeout      18000;
            proxy_pass http://http_intelligent_cabinet;
            proxy_http_version 1.1;
            proxy_set_header        Connection      "";
      }
      location ~ /api-intelligent-wechat(\/)+ {
            proxy_redirect          off;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size    10m;
            client_body_buffer_size 128k;
            proxy_buffers           32 4k;
            proxy_connect_timeout   18000;
            proxy_send_timeout      18000;
            proxy_read_timeout      18000;
            proxy_pass http://http_intelligent_wechat;
            proxy_http_version 1.1;
            proxy_set_header        Connection      "";
      }
       location ~ /favicon\.ico$ {
                root   html;
            }		
      location / {
            root   /usr/local/nginx/html-rfid;     前端PC代码发布地
            try_files $uri $uri/ /index.html;
             }
    }
    server {
      listen       5600;
      server_name  localhost;
      #charset koi8-r;
      set $guid "-";
      #if ( $http_cookie ~* "uid=(\S+)(;.*|$)"){
      if ($http_cookie ~* "CSID=([A-Z0-9]*)"){
      #if($http_cookie ~* "(.*)$"){
          set $guid $1;
      }
      access_log /var/log/nginx/localhost/host.access_5600.log  main;
       location ~ /api(\/)+ {
            proxy_redirect          off;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size    10m;
            client_body_buffer_size 128k;
            proxy_buffers           32 4k;
            proxy_connect_timeout   3;
            proxy_send_timeout      60;
            proxy_read_timeout      60;
            proxy_pass http://http_webapi_test;
           #proxy_http_version 1.1;
            proxy_set_header        Connection      "";
      }
      location ~ /auth(\/)+ {
            proxy_redirect          off;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size    10m;
            client_body_buffer_size 128k;
            proxy_buffers           32 4k;
            proxy_connect_timeout   3;
            proxy_send_timeout      30;
            proxy_read_timeout      30;
            proxy_pass http://http_auth_test;
            proxy_http_version 1.1;
            proxy_set_header        Connection      "";
      }
       location /webSocketServer/ {
            proxy_pass http://http_intelligent_cabinet;
            # WebScoket Support
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
        }		
       location ~ /api-cabinet(\/)+ {
            proxy_redirect          off;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size    10m;
            client_body_buffer_size 128k;
            proxy_buffers           32 4k;
            proxy_connect_timeout   18000;
            proxy_send_timeout      18000;
            proxy_read_timeout      18000;
            proxy_pass http://http_intelligent_cabinet;
            proxy_http_version 1.1;
            proxy_set_header        Connection      "";
      }
        location / {
            root   /usr/local/nginx/html-rfid;
            try_files $uri $uri/ /index.html;
            #index  index.html index.htm;
        } 
       location ~ /favicon\.ico$ {
                root   html-rfid;
            }
      error_page   500 502 503 504  /50x/50x.html;
      location = /50x.html {
        root   html-rfid;
      }
    }
    
    
    server {
      listen       5700;
      server_name  localhost;
      #charset koi8-r;
      set $guid "-";
      #if ( $http_cookie ~* "uid=(\S+)(;.*|$)"){
      if ($http_cookie ~* "CSID=([A-Z0-9]*)"){
      #if($http_cookie ~* "(.*)$"){
          set $guid $1;
      }
      access_log /var/log/nginx/localhost/host.access_5700.log  main;
       location ~ /api(\/)+ {
            proxy_redirect          off;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size    10m;
            client_body_buffer_size 128k;
            proxy_buffers           32 4k;
            proxy_connect_timeout   3;
            proxy_send_timeout      60;
            proxy_read_timeout      60;
            proxy_pass http://http_webapi_test;
           #proxy_http_version 1.1;
            proxy_set_header        Connection      "";
      }
      location ~ /auth(\/)+ {
            proxy_redirect          off;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size    10m;
            client_body_buffer_size 128k;
            proxy_buffers           32 4k;
            proxy_connect_timeout   3;
            proxy_send_timeout      30;
            proxy_read_timeout      30;
            proxy_pass http://http_auth_test;
            proxy_http_version 1.1;
            proxy_set_header        Connection      "";
      }		
       location ~ /api-cabinet(\/)+ {
            proxy_redirect          off;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size    10m;
            client_body_buffer_size 128k;
            proxy_buffers           32 4k;
            proxy_connect_timeout   18000;
            proxy_send_timeout      18000;
            proxy_read_timeout      18000;
            proxy_pass http://http_intelligent_cabinet;
            proxy_http_version 1.1;
            proxy_set_header        Connection      "";
      }
      location ~ /api-intelligent-wechat(\/)+ {
            proxy_redirect          off;
            proxy_set_header        Host            $host;
            proxy_set_header        X-Real-IP       $remote_addr;
            proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size    10m;
            client_body_buffer_size 128k;
            proxy_buffers           32 4k;
            proxy_connect_timeout   18000;
            proxy_send_timeout      18000;
            proxy_read_timeout      18000;
            proxy_pass http://http_intelligent_wechat;
            proxy_http_version 1.1;
            proxy_set_header        Connection      "";
      }
        location / {
            root   /usr/local/nginx/html-rfid-weixin;
            try_files $uri $uri/ /index.html;
            #index  index.html index.htm;
        } 
       location ~ /favicon\.ico$ {
                root   html-rfid-weixin;
            }
      error_page   500 502 503 504  /50x/50x.html;
      location = /50x.html {
        root   html-rfid-weixin;
      }  
    }
```

域名转发 端口号要为8080 端口 域名指向端口号为80的 ip   nginx文件中 要对外开放80端口
一个服务器中 一个端口号不可对应两个工程 若有两个域名转发 另一个要用负载均衡转发 ip + 80端口 指向端口号不为80的工程ip 
配置完 服务器中的配置 需要改为域名

![规则](/images/Nginx 转发_域名配置/规则.png)

[参考链接1](https://www.cnblogs.com/knowledgesea/p/5175711.html)  

[参考链接2](https://www.jb51.net/article/45076.htm )
