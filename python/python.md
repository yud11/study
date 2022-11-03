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

