## RDB

### 简介

​	通过保存数据库中的键值对到RDB文件来记录数据库状态。

​	有两个命令可以用于生成RDB文件，一个是SAVE，另一个是BGSAVE。

​	SAVE命令会阻塞Redis的主进程，直到RDB文件创建完毕为止，在服务器进程阻塞期间，服务器不能处理任何命令请求

​	BGSAVE命令会fork出一个子进程，然后子进程负责创建RDB文件，主进程依然可以处理其他命令请求。

### 设置SAVE

​	Redis允许用户通过服务器配置的save选项，让服务器每隔一段时间自动执行一次BGSAVE命令。举例如下：

​	save 900 1

​	save 300 10

​	save 60 1000

那么只要满足以下三个条件中的一个就会执行BGSAVE命令：

​	服务器在900秒之内，对数据库进行了至少1次修改

​	服务器在300秒之内，对数据库进行了至少10次修改

​	服务器在60秒之内，对数据库进行了至少1000次修改

### 服务器状态数据结构

​	当服务器中配置了上面的save选项后，在内存中结构如下图：

![1627991260264](C:/Users/zxw/AppData/Roaming/Typora/typora-user-images/1627991260264.png)

​	其中saveparams数组中存了save条件，每个save条件存了seconds以及changes，除此之外，服务器状态还维持一个dirty计数器，记录了距离上一次SAVE或BGSAVE命令成功之后服务器对数据库状态进行了多少次修改；lastsave属性则记录了上一次SAVE或BGSAVE命令执行成功的时间。

### 检查SAVE条件是否满足

​	Redis服务器周期性函数serverCron默认每隔100ms就会执行一次，它的其中一个工作就是检查save选项（遍历saveparams数组）所设置的条件是否已经满足，如果满足的话就执行BGSAVE命令。

### BGSAVE原理

​	首先会从主进程中fork一个子进程，fork之后，kernel会把主进程的所有内存页的权限都设置为read-only，然后主进程与子进程的都指向同一片内存空间，当两个进程都只进行读操作时，相安无事；当在备份过程中，主进程对k2这个键的值进行了修改操作，cpu硬件检测到该内存片只有read-only的权限，便会触发页异常中断(page-fault)，陷入kernel的一个中断例程，中断例程中，kernel就会把触发异常的页复制一份，父子进程各自持有独立的一份，这就是写时复制。

![1627993033577](C:/Users/zxw/AppData/Roaming/Typora/typora-user-images/1627993033577.png)

####  CopyOnWrite的好处

1、减少分配和复制资源时带来的瞬时延迟；
2、减少不必要的资源分配；
CopyOnWrite的缺点：
1、如果父子进程都需要进行大量的写操作，会产生大量的分页错误（页异常中断page-fault）; 