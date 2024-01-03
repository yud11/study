## 基于Redis的分布式锁

### 分布式锁

​	随着业务量越来越大，不同的业务被划分到不同的服务中，每个服务都是一个独立的存在，当并发量很大时，不同服务中的不同线程操作同一条数据时，容易产生脏数据，但是传统的jvm内部锁只能对单个服务内部锁有效，因此需要将锁的范围扩大，可以使用基于Redis、数据库、zookeeper等加锁，使得不同服务访问数据时需要先获取锁才能操作，这种锁称为分布式锁。

### 基于Redis的分布式锁

#### 基于Redis单机分布式锁

##### 利用setnt+expire方式实现

```java
/*
*  加锁
*/
public boolean tryLock(String key, String request, int timeOut) {
	Long result = jedis.setnx(key, request);
    if (1L == result) {	//设置成功了就将key添加一个过期时间
        return 1L == jedis.expire(key, timeOut);
    } else {
        return false;
    }
}
```

###### 存在的问题：

​	a)、setnx和expire的操作是分两步进行的，并不是一个原子性的操作。假如线程A执行完setnx操作后，线程所属服务崩了，那么这个key就不会过期，导致其他服务无法获取到这个key的锁。

​	b)、如何key的过期时间到了，获取到锁的线程A还没执行结束，这时候线程B就可以拿到锁，当线程A执行结束后，释放了锁，导致后续线程C拿到锁，这样就会容易造成有两个线程同时持有锁。

###### 解决问题a的方法

​	1、使用**Lua脚本保证setnx和expire操作的原子性**

```java
public boolean tryLock_with_lua(String key, String UniqueId, int seconds) {
    String lua_scripts = "if redis.call('setnx',KEYS[1],ARGV[1]) == 1 then" +
            "redis.call('expire',KEYS[1],ARGV[2]) return 1 else return 0 end";
    List<String> keys = new ArrayList<>();
    List<String> values = new ArrayList<>();
    keys.add(key);
    values.add(UniqueId);
    values.add(String.valueOf(seconds));
    Object result = jedis.eval(lua_scripts, keys, values);
    //判断是否成功
    return result.equals(1L);
}
```

​	2、使用**set key value [EX seconds][PX milliseconds][NX|XX] 命令**

```java
/*
在Redis2.6.12版本后，为set命令添加了很多选项
    EX seconds: 设定过期时间，单位为秒
    PX milliseconds: 设定过期时间，单位为毫秒
    NX: 仅当key不存在时设置值
    XX: 仅当key存在时设置值
*/
public boolean tryLock_with_set(String key, String UniqueId, int seconds) {
    return "OK".equals(jedis.set(key, UniqueId, "NX", "EX", seconds));
}
```

###### 解决问题b的方法

​	在加锁时对value值进行唯一性标记，然后解锁时需要判断这个锁是否是自己所持有的锁

```java
public boolean releaseLock_with_lua(String key,String value) {
    String luaScript = "if redis.call('get',KEYS[1]) == ARGV[1] then " +
            "return redis.call('del',KEYS[1]) else return 0 end";
    return jedis.eval(luaScript, Collections.singletonList(key), Collections.singletonList(value)).equals(1L);
}
```

###### 性能问题

​	在上面方式实现的分布式锁中，当客户端A拿到一把锁之后，在客户端A执行任务期间，如果客户端B也想获得该锁，它需要不断的发送set命令去尝试获得锁，实际上这是客户端B的一个自旋的过程，但是这种自旋对性能是不友好的，因为每次尝试获取锁时，客户端都必须与服务器建立连接，发送命令，都是IO操作，这个操作是很消耗性能的。

​	为了解决这个问题，可以使用Redis的**发布订阅**功能来提高性能。**Redisson客户端底层实现通过这种方式实现**。

#### 基于Redis集群分布式锁

​	使用 `set key value [EX seconds][PX milliseconds][NX|XX]` 命令看上去是可以的，但是当使用Redis集群时，可能会出现如下问题：A客户端获取在Redis的master节点上拿到锁，然后进行主从复制，将这个锁信息复制到slave节点上时，master节点挂掉，发生故障转移，slave节点升级成master节点，这个时候B客户端也能拿到这个锁，导致出现多个客户端拿到同一把锁的状况。**可以使用Redisson客户端已经实现的接口解决**。

### Redisson客户端实现的分布式锁

#### 通过发布订阅提高性能

​	当客户端A获取锁在执行任务期间，若客户端B来尝试获取锁失败时，它会订阅一个redisson_lock_channel_xxx（xxx代表锁的名称）频道，并使用异步线程监听频道的消息，然后利用Java中的Semaphore使当前工作线程阻塞。当客户端A任务执行完释放锁的时候会往redisson_lock_channel_xxx频道发布一条解锁消息，当异步线程收到解锁消息后，会调用Semaphore释放信号量，让客户端B被阻塞的线程唤醒然后加锁。

​	 **通过发布订阅机制，被阻塞的线程可以及时被唤醒，减少无效的空转的查询，有效的提高的加锁的效率。** 

#### 可重入锁

​	Redisson客户端实现的分布式锁是可重入的，具体实现是当需要加锁时，给value的值加1，解锁时给value的值减1。

​	