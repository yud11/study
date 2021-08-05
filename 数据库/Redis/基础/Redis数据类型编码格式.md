## Redis数据类型内部编码概况

 `Redis` 的每个键值内部都是使用一个名字叫做 `redisObject` 这个 C语言结构体保存的，其代码如下： 

 ![redisObject 结构体](https://user-gold-cdn.xitu.io/2018/8/9/1651c0ee3d531fb6?imageView2/0/w/1280/h/960/format/webp/ignore-error/1) 

解释如下：

- `type`：表示键值的数据类型，包括 String、List、Set、ZSet、Hash

- `encoding`：表示键值的内部编码方式，从 Redis源码看目前取值有如下几种：#define 

  ```c
  #define OBJ_ENCODING_RAW 0        /* Raw representation */
  #define OBJ_ENCODING_INT 1        /* Encoded as integer */
  #define OBJ_ENCODING_HT 2         /* Encoded as hash table */
  #define OBJ_ENCODING_ZIPMAP 3     /* Encoded as zipmap */
  #define OBJ_ENCODING_LINKEDLIST 4 /* No longer used: old list encoding. */
  #define OBJ_ENCODING_ZIPLIST 5    /* Encoded as ziplist */
  #define OBJ_ENCODING_INTSET 6     /* Encoded as intset */
  #define OBJ_ENCODING_SKIPLIST 7   /* Encoded as skiplist */
  #define OBJ_ENCODING_EMBSTR 8     /* Embedded sds string encoding */
  #define OBJ_ENCODING_QUICKLIST 9  /* Encoded as linked list of ziplists */
  ```

- `refcount`：表示该键值被引用的数量，即一个键值可被多个键引用

## String的编码方式

Redis中字符串对象的编码可以是int、raw、embstr中的某一种：

- int编码： 保存long 型的64位有符号整数 
  ![1627479620135](E:\GithubNote\数据库\images/1627479620135.png)

- embstr编码：保存长度小于44字节的字符串
  ![1627479755904](E:\GithubNote\数据库\images/1627479755904.png)

- raw编码：保存长度大于44字节的字符串
  ![1627479702834](E:\GithubNote\数据库\images/1627479702834.png)

  

## 二进制安全

　　二进制安全是指，在传输数据时，保证二进制数据的信息安全，也就是不被篡改、破译等，如果被攻击，能够及时检测出来。

　　二进制安全包含了密码学的一些东西，比如加解密、签名等。

　　举个例子，你把数据11110000加密成10001000，然后传给我，就是一种二进制安全的做法。虽然数据库一般用于保存文本数据，但使用数据库来保存二进制数据的场景也不少见，因此，为了确保Redis可以适用于各种不同的使用场景，**SDS(simple dynamid string )的 API都是二进制安全的（binary-safe），所有SDS API都会以处理二进制的方式来处理SDS存放在buf数组里的数据**，**程序不会对其中的数据做任何限制、过滤、或者假设**，数据在写入时是什么样的，它被读 取时就是什么样。

　　这也是我们将SDS的buf属性称为字节数组的原因——Redis不是用这个数组来保存字符，而是用它来保存一系列二进制数据。

[]: https://www.cnblogs.com/jing99/p/11687308.html

