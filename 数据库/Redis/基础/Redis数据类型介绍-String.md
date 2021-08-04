## String

### 字符串

set、get、getset

![1627480846396](C:/Users/zxw/AppData/Roaming/Typora/typora-user-images/1627480846396.png)

set [key] [value] nx：表示当这个key不存在时设置此key的值为value

![1627480962682](C:/Users/zxw/AppData/Roaming/Typora/typora-user-images/1627480962682.png)

set [key] [value] xx：表示当这个key存在时设置此key的值为value

![1627481073213](C:/Users/zxw/AppData/Roaming/Typora/typora-user-images/1627481073213.png)

### 数值

incr/decr [key]：使当前数值类型的key的value自增/自减1

![1627481315443](C:/Users/zxw/AppData/Roaming/Typora/typora-user-images/1627481315443.png)

append keyname value： 追加value值到指定字符串末尾
getrange keyname start end ：获取start到end范围的所有字符组成的子串，包括start和end在内
setrange keyname offset value ：从偏移量 offset 开始， 用 value 参数覆写(overwrite)键 keyname 储存的字符串值。

### bitmap

​	setbit [key] [offset] [value]：设置当前key的二进制位的偏移量为offset的值为value（只能为0或1）

![1627548955549](C:/Users/zxw/AppData/Roaming/Typora/typora-user-images/1627548955549.png)

​	getbit [key] [offset] ：获取当前key的二进制位的偏移量为offset的值

![1627549149398](C:/Users/zxw/AppData/Roaming/Typora/typora-user-images/1627549149398.png)

​	bitpos [key] [bit] [start] [end]：从当前key的第start位置字节到end位置的字节中找出第一个bit的下标

![1627550048200](C:/Users/zxw/AppData/Roaming/Typora/typora-user-images/1627550048200.png)

​	bitcount [key] [start] [end]：统计当前key的第start位置字节到end位置字节中1的数量

![1627550309979](C:/Users/zxw/AppData/Roaming/Typora/typora-user-images/1627550309979.png)

​	bitop [operation] [destkey] [key] [key ...]：将前面的key值与后面的key值进行operation运算（逻辑与或非等）并赋值给destkey

![1627550750912](C:/Users/zxw/AppData/Roaming/Typora/typora-user-images/1627550750912.png)

#### bitmap实际应用

需求1：在一个用户系统中，统计用户的登录天数，并且随机窗口，即统计随机日期区间的用户登录天数

解决方案：将用户id作为key，一年365天作为每一个bit位，如果哪一天登录了，就将这一个bit位设置成为1，最终根据需求统计具体区间内1的个数。这种方案需要花费的内存空间很小，最大只需要46个字节存储单个用户一年的登录情况。

![1627558233120](C:/Users/zxw/AppData/Roaming/Typora/typora-user-images/1627558233120.png)

需求2：京东618做活动，需要给登录的用户送礼物，库存需要备多少礼物。
此问题实际是需要知道在某个区间内活跃用户的大致数量

解决方案：将日期作为key，每个用户映射到二进制位的每一个bit位，当这一天某个用户登录时，就将这个用户对应的二进制位设置为1，如果需要统计某个区间的活跃用户数，只需要将这些区间的key作逻辑或运算得出一个目标key，统计目标key上的二进制位的1的个数即可。

![1627559665622](C:/Users/zxw/AppData/Roaming/Typora/typora-user-images/1627559665622.png)

