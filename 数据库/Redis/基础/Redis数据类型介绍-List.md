## List

list数据类型中，value是一个双向链表，key的内部维护了一个head和tail的指针分别指向链表的头部和尾部，同时，链表有正向索引和反向索引。(3.2版本以后就是quickList)

![1627563626713](E:\GithubNote\数据库\images/1627563626713.png)

### 实现栈

后进先出：相同的方向进行push和pop的操作即可

![1627564608959](E:\GithubNote\数据库\images/1627564608959.png)

### 实现队列

先进先出：相反的方向进行push和pop操作即可

![1627564802195](E:\GithubNote\数据库\images/1627564802195.png)

### 模拟数组

通过索引定位元素

![1627565020848](E:\GithubNote\数据库\images/1627565020848.png)

### 实现阻塞队列

通过b开头命令实现。当使用blpop [key] [timeout] 时，该进程会判断是否存在key，并且key中是否能pop出一个元素，如果没有元素，则会一直阻塞在此，直到到了超时时间或者出现了key的list并且可以pop出一个元素。

![1627565367104](E:\GithubNote\数据库\images/1627565367104.png)

