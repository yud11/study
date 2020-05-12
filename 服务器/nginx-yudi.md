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

