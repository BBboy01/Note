# 安装

```bash
yum install yum-utils net-tools -y
```

## 配置 nginx 源

```bash
cat > /etc/yum.repos.d/nginx.repo << EOF
[nginx-stable]
name=nginx stable repo
baseurl=http://nginx.org/packages/centos/\$releasever/\$basearch/
gpgcheck=1
enabled=1
gpgkey=https://nginx.org/keys/nginx_signing.key
module_hotfixes=true
EOF
```

## 安装 nginx

```bash
yum install nginx -y
```

## 常用命令

```bash
#立即停止
nginx -s stop

#执行完当前请求再停止
nginx -s quit

#重新加载配置文件，相当于restart
nginx -s reload

#将日志写入一个新的文件
nginx -s reopen

#测试配置文件
nginx -t
```

日志默认存储在`/var/log/nginx/`

> 注意：在centos 7中，用systemctl启动nginx可能出现如下错误，
>
> nginx: [emerg] bind() to 0.0.0.0:8000 failed (13: Permission denied)
>
> 这是由于selinux的安全策略引起的。解决方法如下：
>
> - setenforce 0 （临时）
> - 修改/etc/selinux/config，设置SELINUX=disabled （永久有效，需重启）

```bash
systemctl start nginx

systemctl status nginx

#产看日志
journalctl -xe

systemctl stop nginx

systemctl reload nginx

#配置开机启动
systemctl enable nginx
```

## 配置文件

配置文件位于 `/etc/nginx/nginx.conf` , 下列命令会引用 `/etc/nginx/conf.d` 目录下所有的 `.conf` 文件，这样可以保持主配置文件的简洁，同时配个多个 `.conf` 文件方便区分，增加可读性。

```nginx
include /etc/nginx/conf.d/*.conf;
```

默认配置文件 `/etc/nginx/conf.d/default.conf`

```nginx
server {
    listen       80; #监听端口
    server_name  localhost;

    location / {
        root   /usr/share/nginx/html; #根目录
        index  index.html index.htm; #首页
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   /usr/share/nginx/html;
    }
}
```

## 配置文件结构

```nginx
http {

  server { # 虚拟主机
     
    location {
      listen 80；
      server_name localhost;
    }
    location {
       
    }
      
  }

  server {
  
  }

}
```

# 静态网页

配置文件示例:

```nginx
server {
  
    listen 8000;
    server_name localhost:
    
    location / {
        root /home/AdminLTE-3.2.0;
        index index.html index2.html index3.html;
    }
  
}
```

虚拟主机 server 通过 listen 和 server_name 进行区分，如果有多个 server 配置，listen + server_name 不能重复

## listen

​	监听可以配置成`IP`或`端口`或`IP+端口` 

​	listen 127.0.0.1:8000; 
​	listen 127.0.0.1;（ 端口不写,默认80 ） 
​	listen 8000; 
​	listen *:8000; 
​	listen localhost:8000;

## server_name

​	server_name 主要用于区分，可以随便起。

​	也可以使用变量` $hostname `配置成主机名。

​	或者配置成域名：` example.org ` ` www.example.org ` ` *.example.org `

​	如果多个 server 的端口重复，那么根据`域名`或者`主机名`去匹配 server_name 进行选择。

​	下面的例子中：

```bash
curl http://localhost:80 会访问 /usr/share/nginx/html
curl http://nginx-dev:80 会访问 /home/AdminLTE-3.2.0
```

```nginx
# curl http://localhost:80 会访问这个
server {
    listen       80;
    server_name  localhost;

    #access_log  /var/log/nginx/host.access.log  main;

    location / {
        root   /usr/share/nginx/html;
        index  index.html index.htm;
    }

 # curl http://nginx-dev:80 会访问这个
server{
    listen 80;
    server_name nginx-dev;  #主机名
    
    location / {
        root /home/AdminLTE-3.2.0;
        index index.html index2.html index3.html;
    }
  
}
```

## location

​	`/` 请求指向 root 目录

​	location 总是从 `/` 目录开始匹配，如果有子目录，例如 `/css`，他会指向 `/static/css`

```nginx
location /css {
  root /static;
}
```

# HTTP反向代理

## 正向代理与反向代理

### 正向代理

在**客户端**代理转发请求称为**正向代理**。例如VPN。

![](https://cdn.nlark.com/yuque/0/2022/svg/28915315/1659069721078-51b03bd0-f4ae-4441-a8b1-deef20ce8e05.svg)

### 反向代理

在**服务器端**代理转发请求称为**反向代理**。例如 nginx

![](https://cdn.nlark.com/yuque/0/2022/svg/28915315/1659069737327-ed7e214e-aace-45b1-afe3-32a776b94b8f.svg)

## 配置代理服务

![](https://cdn.nlark.com/yuque/0/2022/png/28915315/1659084328663-32955bb1-82f0-40f3-b94a-f233b4a5793f.png)

启动 dio 后台服务，端口为 `8088`

nginx配置文件：

```nginx
server {
  
  listen 8001;
  
  server_name dio.localhost;
  
  location / {
    proxy_pass http://localhost:8088;
  }

}
```

### proxy_pass配置说明：

```nginx
location /some/path/ {
    proxy_pass http://localhost:8080/;
}
```

- 如果`proxy-pass`的地址只配置到 `**/**`，不包括 URI，那么location将被追加到转发地址中，如上所示，访问 `http://localhost/some/path/page.html` 将被代理到 `http://localhost:8080/some/path/page.html` 

```nginx
location /some/path/ {
    proxy_pass http://localhost:8080/zh-cn/;
}
```

- 如果`proxy-pass`的地址包括URI，那么location将不会被追加到转发地址中，如上所示，访问 `http://localhost/some/path/page.html` 将被代理到 `http://localhost:8080/zh-cn/page.html`。

## 设置代理请求 headers

用户可以重新定义或追加 header 信息传递给后端[‎](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass_request_headers)‎服务器。可以包含文本、变量及其组合。默认情况下，仅重定义两个字段：

```nginx
proxy_set_header Host       $proxy_host;
proxy_set_header Connection close;
```

由于使用反向代理之后，**后端服务无法获取用户的真实 IP（获取到的为 nginx 的 IP 127.0.0.1）**，所以，一般反向代理都会设置以下header信息。

```nginx
location /some/path/ {
    # nginx 的主机地址
    proxy_set_header Host $http_host;
    # 用户端真实的 IP，即客户端 IP
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    proxy_pass http://localhost:8088;
}
```

常用变量的值：

- `$host`：nginx主机IP，例如192.168.56.105

- `$http_host`：nginx主机IP和端口，192.168.56.105:8001

- `$proxy_host`：localhost:8088，proxy_pass里配置的主机名和端口

- `$remote_addr`:用户的真实IP，即客户端IP

## 非HTTP代理

如果要将请求传递到非 HTTP 代理服务器，可以使用下列指令：

- [fastcgi_pass](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_pass) 将请求转发到FastCGI服务器（多用于PHP）
- [scgi_pass](https://nginx.org/en/docs/http/ngx_http_scgi_module.html#scgi_pass) 将请求转发到SCGI server服务器（多用于PHP）
- [uwsgi_pass](https://nginx.org/en/docs/http/ngx_http_uwsgi_module.html#uwsgi_pass) 将请求转发到uwsgi服务器（多用于python）
- [memcached_pass](https://nginx.org/en/docs/http/ngx_http_memcached_module.html#memcached_pass) 将请求转发到memcached服务器

# 动静分离

## 动静分离的好处

Apache Tocmat 严格来说是一款java EE服务器，主要是用来处理 servlet请求。处理css、js、图片这些静态文件的IO性能不够好，因此，将静态文件交给nginx处理，可以提高系统的访问速度，减少tomcat的请求次数，有效的给后端服务器降压

## 分离静态文件

![img](https://cdn.nlark.com/yuque/0/2022/png/28915315/1659339381949-41af5d82-eb67-4db3-aaa3-524a61e3a52f.png)

## 静态文件目录

```bash
mv WEB-INF/classes/static /home/www/
```

除了js、css、图片文件之外，还有字体文件和一个ie提示页面

![WechatIMG30.png](https://cdn.nlark.com/yuque/0/2022/png/28915315/1659403384054-77a861b5-99f9-45a6-81e1-f657aac53b40.png?x-oss-process=image%2Fresize%2Cw_768%2Climit_0)

```nginx
server{

  location / {
    proxy_pass http://localhost:8080/;
  }
  
  location = /html/ie.html {
    root  /home/www/static;
  }
  
  location ^~ /fonts/ {
   
    root  /home/www/static;
  }
  
  location ~ \.(css|js|png|jpg|gif|ico) {
    root /home/www/static;
}
```

### location 修饰符

- location可以使用修饰符或正则表达式

#### 修饰符：

- `= `  等于，严格匹配 ，匹配优先级最高。

- `^~`  表示普通字符匹配。使用前缀匹配。如果匹配成功，则不再匹配其它 location。优先级第二高。

- `~` 区分大小写

- `~*`  不区分大小写

#### 优先级:

优先级从高到低依次为：

1. 精确匹配（=）
2. 前缀匹配（^~）
3. 正则匹配（~和～*）
4. 不写

如上所示：

- /images/1.jpg 代理到  http://localhost:8080/images/1.jpg
- /some/path/1.jpg 代理到 http://localhost:8080/some/path/1.jpg

# 缓冲(buffer)和缓存(cache)

## 缓冲（buffer）

缓冲一般放在内存中，如果不适合放入内存（比如超过了指定大小），则会将响应写入磁盘临时文件中。

启用缓冲后，nginx 先将后端的请求影响（response）放入缓冲区中，等到整个响应完成后，再发给客户端。

![img](https://cdn.nlark.com/yuque/0/2022/png/28915315/1659585056819-df82befd-d847-4a0b-bffc-84cf626c2fed.png)

客户端往往是用户网络，情况复杂，可能出现网络不稳定，速度较慢的情况。

而 nginx 到后端 server 一般处于同一个机房或者区域，网速稳定且速度极快。

![img](https://cdn.nlark.com/yuque/0/2022/png/28915315/1659585159748-308fa557-418b-4bda-9f3f-a16efccaa673.png)

如果禁用了缓冲，则在客户端从代理服务器接收响应时，响应将同步发送到客户端。对于需要尽快开始接收响应的快速交互式客户端，此行为可能是可取的。

这就会带来一个问题：因为客户端到 nginx 的网速过慢，导致 nginx 只能以一个较慢的速度将响应传给客户端；进而导致后端 server 也只能以同样较慢的速度传递响应给 nginx，造成一次请求连接耗时过长。

在高并发的情况下，后端 server 可能会出现大量的连接积压，最终拖垮 server 端。

![img](https://cdn.nlark.com/yuque/0/2022/png/28915315/1659585458893-c276995d-f299-444b-bd33-cee5933ab878.png)

开启代理缓冲后，nginx可以用较快的速度尽可能将响应体读取并缓冲到本地内存或磁盘中，然后同时根据客户端的网络质量以合适的网速将响应传递给客户端。

这样既解决了server端连接过多的问题，也保证了能持续稳定的像客户端传递响应。

使用[proxy_buffering](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffering)启用和禁用缓冲，nginx默认为 on 启用缓冲，若要关闭，设置为 off  。

```nginx
proxy_buffering off;
```

[proxy_buffers](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffers) 指令设置每个连接读取响应的缓冲区的`大小`和`数量` 。默认情况下，缓冲区大小等于一个内存页，4K 或 8K，具体取决于操作系统。

来自后端服务器响应的第一部分存储在单独的缓冲区中，其大小通过 [proxy_buffer_size](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_buffer_size) 指令进行设置，此部分通常是相对较小的响应headers，通常将其设置成小于默认值。

```nginx
location / {
    proxy_buffers 16 4k;
    proxy_buffer_size 2k;
    proxy_pass http://localhost:8088;
}
```

如果整个响应不适合存到内存里，则将其中的一部分保存到磁盘上的‎‎临时文件中‎‎。

‎[‎proxy_max_temp_file_size‎](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_max_temp_file_size)‎设置临时文件的最大值。

‎[‎proxy_temp_file_write_size‎](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_temp_file_write_size)‎设置一次写入临时文件的大小。

## 缓存（cache）

启用缓存后，nginx将响应保存在磁盘中，返回给客户端的数据首先从缓存中获取，这样子相同的请求不用每次都发送给后端服务器，减少到后端请求的数量。

![img](https://cdn.nlark.com/yuque/0/2022/png/28915315/1659599505974-889b4227-80eb-4deb-8bae-303828cc3bc4.png)

启用缓存，需要在http上下文中使用 [proxy_cache_path](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path) 指令，定义缓存的本地文件目录，名称和大小。

缓存区可以被多个server共享，使用[proxy_cache](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache) 指定使用哪个缓存区。

```nginx
http {
    proxy_cache_path /data/nginx/cache keys_zone=mycache:10m;
    server {
        proxy_cache mycache;
        location / {
            proxy_pass http://localhost:8000;
        }
    }
}
```

缓存目录的文件名是 [proxy_cache_key](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_key) 的MD5值。 

例如：`/data/nginx/cache/**c**/**29**/b7f54b2df7773722d382f4809d650**29c**`

[proxy_cache_key](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_key) 默认设置如下：

```nginx
proxy_cache_key $scheme$proxy_host$uri$is_args$args;
```

也可以自定义缓存的键，例如:

```nginx
proxy_cache_key "$host$request_uri$cookie_user";
```

缓存不应该设置的太敏感，可以使用[proxy_cache_min_uses](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_min_uses)设置相同的key的请求，访问次数超过指定数量才会被缓存。

```nginx
proxy_cache_min_uses 5;
```

默认情况下，响应无限期地保留在缓存中。仅当缓存超过最大配置大小时，按照时间删除最旧的数据。

## 示例

```nginx
proxy_cache_path /var/cache/nginx/data keys_zone=mycache:10m;

server {

    listen 8001;
    server_name ruoyi.localhost;
    
    location / {
        #设置buffer
        proxy_buffers 16 4k;
        proxy_buffer_size 2k;
        proxy_pass http://localhost:8088;        

    }


    location ~ \.(js|css|png|jpg|gif|ico) {
        #设置cache
        proxy_cache mycache;
        proxy_cache_valid 200 302 10m;
        proxy_cache_valid 404      1m;
        proxy_cache_valid any 5m;

        proxy_pass http://localhost:8088;  
    }

    location = /html/ie.html {

        proxy_cache mycache;
        proxy_cache_valid 200 302 10m;
        proxy_cache_valid 404      1m;
        proxy_cache_valid any 5m;

        proxy_pass http://localhost:8088;  
    }

    location ^~ /fonts/ {

        proxy_cache mycache;
        proxy_cache_valid 200 302 10m;
        proxy_cache_valid 404      1m;
        proxy_cache_valid any 5m;

        proxy_pass http://localhost:8088;  
    }

}
```

