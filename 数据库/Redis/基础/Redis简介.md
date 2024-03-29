## 常识

应用在获取数据时，无论数据是存在硬盘还是内存中，读取的速度都受到几个维度的影响：寻址和带宽

寻址：找到数据所在的位置

带宽：每次能传输的数据大小

## 数据存储发展进程

数据全部存储在硬盘中，未区分数据的格式及类型，读取速度很慢；

数据存储在关系型数据库中，数据存储在表中，同时建立索引，并将索引存储起来，读取时先读取索引，再根据索引找出具体的数据，读取速度变快，但是当数据量达到千亿级别时，增删改的速度必然变慢，读取数据需要从两个角度考虑：

1、读取数据量较少时，速度相对不会变慢

2、并发较大，读取数据量较大时，由于硬件及网络的影响，读取速度必然变慢

## 关系型数据库小知识点

关系型数据库建表时，必须给出schema的类型，这样就能确定每一行数据的宽度，在插入数据时，如果有某个字段是空的，数据库会默认在此字段填充0。这样做的好处就是在以后增加或减少数据时不用移动原先数据的位置，增加了性能。

## Redis简介

 Redis 是一种开源（BSD 许可）、内存中数据结构存储，用作数据库、缓存和消息代理。Redis 提供数据结构，例如字符串、散列、列表、集合、带有范围查询的排序集合、位图、超级日志、地理空间索引和流。Redis 内置复制、Lua 脚本、LRU 驱逐、事务和不同级别的磁盘持久化，并通过 Redis Sentinel 和 Redis Cluster 自动分区提供高可用性。 

## Redis与Memcache比较

Redis与Memcache同样是key-value型数据库，但是Memcache的value没有数据类型，这样就会带来一个问题，当value的值比较复杂，Memcache的server端通过网卡I/O返回的数据需要Memcache的client端实现的代码去解码，client端对这些未知的数据解码比较费力，但是redis的value都确定类型，server端对每种类型都有自己的方法，index()、lpop()等，这样client端接受到的数据就无需解码。

![1627386226717](E:\GithubNote\数据库\images/1627386226717.png)

