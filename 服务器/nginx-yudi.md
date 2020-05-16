# nginx安装

## 1，下载依赖包

```shell
yum -y install make zlib zlib-devel gcc-c++ libtool openssl openssl-devel pcre pcre-devel
```

## 2，下载nginx，并解压

```she
tar -xvf nginx-1.16.0.tar.gz
```

## 3，编译

```shell
./configure
make 
make install
```

# nginx启动

```shell
#默认方式启动：

./sbin/nginx

#指定配置文件启动

./sbing/nginx -c /tmp/nginx.conf

#指定nginx程序目录启动

./sbin/nginx -p /usr/local/nginx/
```

# nginx停止方式

```shell
#快速停止
./sbin/nginx -s stop

#优雅停止
./sbin/nginx -s quit
```

# 其他命令

```shell
# 热装载配置文件 ，不用停止可以刷新配置（一定要熟练，这是用的最多的命令）
./sbin/nginx -s reload

# 重新打开日志文件（下面单说）
./sbin/nginx -s reopen

# 检测当前使用的是哪个配置文件，配置是否正确（可以在配置文件加点乱码测试一下）（这个命令也经常使用）
./sbin/nginx -t
```

# nginx配置反向代理

```shell
#每个server代表一个虚拟主机
server {
        listen       80; # 监听的端口
        server_name  192.168.197.10;#监听的域名和地址

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / { #配置的路径
                proxy_pass http://192.168.197.1:8080; #代理的路径
         # root   html;
           # index  index.html index.htm;
         #  ngx_fastdfs_module;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #

```

## 配置两个代理

```shell
server {
        listen       80;
        server_name  192.168.197.10;

        location ~ /edu/ {
                proxy_pass http://192.168.197.1:8080;
        }
        location ~ /vod/ {
                proxy_pass http://192.168.197.1:9080;
        }
        error_page   500 502 503 504  /50x.html;
    }


```

## location 指令说明

```shell
location [= | ~ |~* | ^~]
```

1.  **=** ：用于不含正则表达式的 uri 前，要求请求字符串与 uri 严格匹配，如果匹配
   成功，就停止继续向下搜索并立即处理该请求。
2.  **~**：用于表示 uri 包含正则表达式，并且区分大小写。
3.  **~***：用于表示 uri 包含正则表达式，并且不区分大小写。
4.  **^~**：用于不含正则表达式的 uri 前，要求 Nginx 服务器找到标识 uri 和请求字
   符串匹配度最高的 location 后，立即使用此 location 处理请求，而不再使用 location
   块中的正则 uri 和请求字符串做匹配。  

  **注意：如果 uri 包含正则表达式，则必须要有 ~ 或者 ~* 标识。**  

# 负载均衡

## 轮询

每个请求按照时间顺序逐一分配到不同的后端服务器，如果后端服务器down掉，自动剔除

## 权重

weight代表权重，默认为1，权重越高，分配的客户端请求越多。适用于服务器性能不均匀的情况。轮询则配置每台服务器的权重一样。

```shell
upstream myserver {
	server 192.168.197.1:8080 weight=10;
	server 192.168.197.1:9080 weight=11;
}

server {
listen       80;
server_name  192.168.197.10;

location ~ /edu/ {
	proxy_pass http://myserver;
	proxy_connect_timeout 10;
}
```

## iphash

每个请求按访问 ip 的 hash 结果分配，这样每个访客固定访问一个后端服务器，可以解决 session 的问题。

```
upstream server_pool {
	ip_hash;
	server 192.168.5.21:80;
	server 192.168.5.22:80;
}
```

## fair

按后端服务器的响应时间来分配请求，响应时间短的优先分配。

```shell
upstream server_pool {
	server 192.168.5.21:80;
	server 192.168.5.22:80;
	fair;
}
```

