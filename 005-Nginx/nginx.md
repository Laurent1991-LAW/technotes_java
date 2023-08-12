# Nginx





## 1. 安装及基本命令



### 1.1 安装

nginx解压后需要 

- ./config 进行配置及安装 make && make install ；
- 配置环境变量



脚本案例：

![190CD901A26BF6C288DE21689D8E8333](.\images\190CD901A26BF6C288DE21689D8E8333.png)





### 1.2 基本命令

```
nginx -s <SIGNAL>
```

where `<SIGNAL>` can be one of the following:

- `quit` – Shut down gracefully (the `SIGQUIT` signal)
- `reload` – Reload the configuration file (the `SIGHUP` signal)
- `reopen` – Reopen log files (the `SIGUSR1` signal)
- `stop` – Shut down immediately (or fast shutdown, the `SIGTERM` singal)



## 2. 应用场景



### 2.1 健康检查

In the following example, if NGINX fails to send a request to a server or does not receive a response from it 3 times in 30 seconds, it marks the server as unavailable for 30 seconds:

```
upstream backend {
    server backend1.example.com;
    server backend2.example.com max_fails=3 fail_timeout=30s;
    keepalive 2048;
}
```



### 2.2 缓存

定义缓存保存的位置

```
http {
    # ...
    proxy_cache_path /data/nginx/cache keys_zone=mycache:10m;
}
```



定义哪些请求使用缓存 、至少有几次请求后会将相应加入缓存、缓存哪些方法

```
proxy_cache_key "$host$request_uri$cookie_user";
proxy_cache_min_uses 5;
proxy_cache_methods GET HEAD POST;
```



某些响应码的响应被缓存的时间

```
proxy_cache_valid 200 302 10m;
proxy_cache_valid 404      1m;
```



在server中引入缓存

```
http {
    # ...
    proxy_cache_path /data/nginx/cache keys_zone=mycache:10m;
    server {
        proxy_cache mycache;
        location / {
            proxy_pass http://localhost:8000;
        }
    }
}
```





### 2.3 压缩

Gzip压缩是Nginx提供的一个<u>优化网页性能和提高用户体验</u>的重要功能。Gzip压缩是一种在传输过程中对数据进行压缩，减小响应体积的方法。Nginx在接收到HTTP请求后，会<u>根据客户端请求头中的Accept-Encoding字段判断是否开启Gzip压缩</u>。若客户端支持Gzip压缩，则Nginx会对响应数据进行压缩，并<u>在响应头中增加Content-Encoding字段，将压缩方式指定为gzip</u>。

```nginx
gzip on;
gzip_disable "msie6"; # 客户端的Accept-Encoding字段？
gzip_proxied expired no-cache no-store private auth;
gzip_buffers 32 8k; # number为指定Nginx服务器要向服务器申请的缓存空间的个数，size 每个缓存的空间大小
gzip_comp_level 1;  # 1-9 越大占用的CPU越大, 一般1-2已足够
gzip_min_length 1k; # 指定请求的数据超过某个阈值后才开启Gzip压缩功能
gzip_types text/plain text/css application/json application/x-javascript text/xml application/xml application/xml+rss text/javascript;

```





### 2.4 定义日志格式

```nginx
http {
    include       mime.types;
    default_type  application/octet-stream;
 
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
 
    access_log  logs/access.log  main;
    sendfile        on;
    keepalive_timeout  65;
 
    server {
        listen       80;
        server_name  localhost;
 
        location / {
            root   /data/software/nginx/html/dist;
            index  index.html index.htm;
        }
    }
}
```



### 2.5 利用正则rewrite

![F3DF7F9C62FD1DDAB2A73E732B8CF768](.\images\F3DF7F9C62FD1DDAB2A73E732B8CF768.png)



### 2.6 超时终止连接



![326D2CF24F12F2BFDA6DCEB86850776B](.\images\326D2CF24F12F2BFDA6DCEB86850776B.png)







## 3. 运维常用脚本

获取关键字‘listen’及之后的10行（-A 10）

```shell
cat /app/wlqz/nginx/conf/nginx.conf | grep -A 10 listen
```

若为 之前 则为 -B ； 前后则为 -C









