# 内置函数

## print函数

```python
# a+：打开一个文件用于读写。如果该文件已存在，文件指针将会放在文件的结尾。文件打开时会是追加模式。如果该文件不存在，创建新文件用于读写。
fd = open("/Users/yud1/code/pycode/study/demo.txt","a+")
# 输入到文件当中
print("helloword",file=fd); 
fd.close()
# 不换行输出
print("a","b","c")
```

## 类型转换函数

```python
str()
int() # 字符串转int 必须为整数 int('98.17') -> 会报错 int(98.17)-> 98 截取整数部分
float()
```



## range函数

1. range函数用于生成一个整数序列。

2. 创建range函数的三种方式

```python
#一个参数
range(stop) 
#两个参数
range(start,stop)
#三个参数 start：从几开始 stop：从几结束 step：步长 
range(start,stop,step) 
```

3. 返回值是一个**迭代器对象**

4. 优点：不管表示的整数序列有多长，所有range对象占用的内存空间都是相同的，只会在内存中保存三个变量
5. in与not in 可以判断整数序列中是否存在（不存在）指定的整数

# 基础知识

## 转义字符

```python
# \n 换行
# \t 制表符
# \r 回车 重新输入这一行
# \b 删除一个字符
# \\ 取消转义
# r或R代表后面的字符串不转义
print(r"hello\nword") # hello\nword
```



## 编码规范

Unicode：为世界上所有字符都分配了一个唯一的数字编号。它是一种规定，Unicode本身只规定了每个字符的数字编号是多少，并没有规定这个编号如何存储。

UTF-8：实现它的一种规范，英文字母一字节存储，中文三字节存储。

GB2312：包含简体中文

GBK：包含繁体中文

GB18030：包含少数民族的语言

python文件默认使用utf-8编码，也可以用如下代码写在文件的第一行，来告诉解释器该文件使用的编码

```py
#codeing:utf-8
```



## 数据类型

### 浮点数

浮点数在计算机中可能表示的不精确，所以在进行运算时，会出现误差。使用Decimal函数来解决。

```python
from decimal import Decimal
a = 1.1
b = 2.2
print(1.1+2.2) # 3.3000000000000003
print(Decimal("1.1") + Decimal("2.2")) # 3.3
```

### 布尔类型

布尔类型可以转成0或1来使用

```python
f1 = True
f2 = False
print(f1 + 1) # 2
print(f2 + 1) # 1
```

### 字符串

三引号可以在多行之间使用

## 运算符

### 算数运算符

```python
a = 11/2  
b = 11//2 # 整除，不保留小数
print(a,type(a)) # 5.5 <class 'float'>
print(b,type(b)) # 5 <class 'int'>
c = 11%2 # 1 取余
d = 2**3 # 8 幂运算
print(9%-4) # -3 公式 = 被除数 - 除数*商 9-（-4）*（-3）=-3
print(-9%4) #  3 公式 = 被除数 - 除数*商 -9-（4）*（-3）=3

```

### 赋值运算符

```python
a=b=c=20 #链式赋值 且a,b,c的id相同
a,b,c = 10,20,30 #支持解包赋值
a,b =b,a # 交换两个变量的值，不用定义中间变量
```

### 比较运算符

==：比较的是对象的值

is：比较的是对象的id

### 位运算符

```python
# python中负数用‘+-’号加原码表示
print(bin(-2)) #-0b10
```

左移右移都添0



### 运算符优先级顺序

算数运算符 > 位运算符 > 比较运算符 > 布尔运算符 > 赋值运算符

## 条件表达式

x if 判断条件 else y

判断条件为True，返回x，否则返回y



## 循环结构

### for-in循环

#### 结构

for 自定义的变量 in 可迭代对象

​		循环体

#### 注意

当循环体内不需要访问自定义变量，可以将自定义变量代替为下划线



### 循环结构中的else

```python
while:
  pass
else:
	pass

for _ in 循环体：
	pass
else：

# 在循环结构 while和for中 else是在没有碰到break时执行

```



### break

python中没有跳出指定循环的语法，break只能跳出当前循环

# 列表

列表相当于java中的数组，但是可以存储多种类型的值。

## 列表特点

1. 列表元素按顺序有序排列
2. 索引映射唯一一个数据
3. 列表可以存储重复数据
4. 任意数据类型混存
5. 根据需要动态分配和回收内存

## 查询操作

### 获取单个元素

```python
# index() : 获取列表中的索引，不存在会报错
# list[] : 获取列表中指定索引的元素 索引不存在会报错
```

### 切片操作

获取列表中的多个元素

```python
'''
list[start:end:step]
返回的是对原列表片段的拷贝
start:默认是第一个元素
stop：默认是最后一个元素
step：步长默认为1

当step为负数时
		切片的第一个元素默认为原列表最后一个元素
		切片的最后一个元素默认为原列表的第一个元素
		从start开始往前计算切片
''' 
```

### 判断指定元素在列表中是否存在

```python
'''
元素 in 列表名
元素 not in 列表名
'''
```

## 添加

append()：添加一个元素

注意：当参数是list时，把list作为一个对象，添加进去

```python
list1 = [10,20,30,40]
list2 = [50,60,90,100]
list1.append(list2) # [10, 20, 30, 40, [50, 60, 90, 100]]
```

extend()：添加多个

注意：当参数是list时，把list里面的多个对象，依次添加进去

```python
list1 = [10,20,30,40]
list2 = [50,60,90,100]
list1.extend(list2) # [10, 20, 30, 40, 50, 60, 90, 100]
```

insert(index, obj)：在指定的索引位置 插入obj

```python
list1 = [10,20,30,40]
list2 = [50,60,90,100]
list1.insert(1,30)
print(list1) # [10, 30, 20, 30, 40]

```

切片操作

```python
list1 = [10,20,30,40]
list2 = [50,60,90,100]
list1[1:] =  list2 # 将1索引之后的元素删除，添加list2的所有元素
print(list1) # [10, 50, 60, 90, 100]
```

## 删除

Remove(obj):删除指定元素

Pop(index):删除指定索引元素，不加index，默认删除最后一个元素

切片删除

```python
list1 = [10,20,30,40]
list1[1:3] = []
print(list1) # [10,40]
```

Clear()：清除列表

del ：删除列表

## 排序

```python
'''
sort（）：默认升序,对原列表进行排序
sorted():生成新的列表，不对原列表进行排序
'''
lst.sort(reverse = True) # 降序
lst1 =  sorted(lst) 
```

## 列表生成式

 语法格式：[表示列表元素的表达式 for 自定义变量 in 可迭代对象]

```python
lst = [i*i for i in range(1,10)]
```

# 字典

## 字典特点

