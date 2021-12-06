# java8 stream流对集合的常用操作

* (中间操作) foreach，filter，map，flatmap，peek，distinct，skip

* (终端操作) allMatch，anyMatch，findfirst，findany，min，max，count

### 数据准备

```java
// 用来测试的实体类
public class User {
    public static List<User> init() {
        User user = new User("张三", 18, "男");
        User user1 = new User("李四", 28, "女");
        User user2 = new User("王五", 38, "男");
        User user3 = new User("赵六", 48, "女");
        return Arrays.asList(user,user1,user2,user3);
    }
    private String name;
    private Integer age;
    private String sex;
}
```

### filter-数据过滤

#### 代码

筛选出年龄大于30岁的人并打印

```java
@Test
public void filterTest() {
  List<User> users = User.init();
  // 筛选出年龄大于30岁的人
  users.stream()
    .filter(e -> e.getAge() >30)
    .forEach(e -> System.out.println(JSONObject.toJSONString(e,true)));
}
```

#### 输出结果

```json
{
	"age":38,
	"name":"王五",
	"sex":"男"
}
{
	"age":48,
	"name":"赵六",
	"sex":"女"
}
```

### map-数据映射

#### 代码

将User对象映射成User的姓名

```java
@Test
public void mapTest() {
  List<User> users = User.init();
  // 将User对象映射成User的姓名
  users.stream()
    .map(User::getName)
    .forEach(e -> System.out.println(JSONObject.toJSONString(e,true)));
}
```

#### 输出结果

```json
"张三"
"李四"
"王五"
"赵六"
```

### 中间操作 peek、sorted

peek 方法类似于 foreach ，对 stream 中的每一个对象进行操作，区别是 peek 是一种无状态的中间操作，而foreach 是终端操作，所以 peek 方法只能在 stream 流处理的过程中使用，而不能作为结尾的方法

#### 代码

```java
// peek 与 foreach 并行
@Test
public void peekTest() {
  List<User> users = User.init();
  users.stream()
    .peek(e -> System.out.println(e.getName()))
    .forEach(e -> System.out.println(JSONObject.toJSONString(e,true)));
}

// sorted 导致串行
@Test
public void peekTest() {
  List<User> users = User.init();
  users.stream()
    .peek(e -> System.out.println(e.getName()))
    // 根据年龄进行排序
    .sorted(Comparator.comparing(User::getAge))
    .forEach(e -> System.out.println(JSONObject.toJSONString(e,true)));
}
```

#### 输出结果

输出结果可以看出来 peek 和 stream 是并行输出的，这是因为 peek 无状态的操作，不会改变后续操作的结果，所以stream 的遍历并不需要按照串行的方式来进行，如果在 peek 和 foreach 中插入一个有状态操作（比如sorted）那么流就需要按照串行的方式来执行了,需要注意的是这里的串行或并行指的是执行顺序，和多线程无关。

```json
// 并行
张三
{
	"age":18,
	"name":"张三",
	"sex":"男"
}
李四
{
	"age":28,
	"name":"李四",
	"sex":"女"
}
王五
{
	"age":38,
	"name":"王五",
	"sex":"男"
}
赵六
{
	"age":48,
	"name":"赵六",
	"sex":"女"
}

// 串行并排序
张三
李四
王五
赵六
{
	"age":18,
	"name":"张三",
	"sex":"男"
}
{
	"age":28,
	"name":"李四",
	"sex":"女"
}
{
	"age":38,
	"name":"王五",
	"sex":"男"
}
{
	"age":48,
	"name":"赵六",
	"sex":"女"
}
```

### distinct 去重

#### 代码

对数据进行去重,相同性别去重

```java
@Test
public void distinctTest() {
    List<User> users = User.init();
    // 将User对象映射成User的性别
    users.stream()
            .map(User::getSex)
            .distinct()
            .forEach(e -> System.out.println(JSONObject.toJSONString(e,true)));
}
```

#### 输出结果

```java
"男"
"女"
```

### skip 跳过数据

#### 代码

跳过第一个元素

```java
@Test
public void skipTest() {
  List<User> users = User.init();
  users.stream()
    .skip(1)
    .forEach(e -> System.out.println(JSONObject.toJSONString(e,true)));
}
```

#### 输出结果

```json
{
	"age":28,
	"name":"李四",
	"sex":"女"
}
{
	"age":38,
	"name":"王五",
	"sex":"男"
}
{
	"age":48,
	"name":"赵六",
	"sex":"女"
}
```

### anyMatch、allMatch 数据匹配

#### 代码

anyMatch 是一个短操作，所以找到第一个符合断言的元素就会返回，而 allMatch 是一个长操作，需要遍历完整个流才会返回

```java
// anyMatch 如果有任意一个元素符合断言，那么返回 true
@Test
public void anyMatchTest() {
  List<User> users = User.init();
  boolean match = users.stream()
    .anyMatch(e -> "张三".equals(e.getName()));
  System.out.println(match);
}

//结果
true

  
// allMatch 如果全部元素符合断言，那么返回true
@Test
public void allMatchTest() {
  List<User> users = User.init();
  boolean match = users.stream()
    .allMatch(e -> "张三".equals(e.getName()));
  System.out.println(match);
}

//结果
false
```

### findFirst、findAny

返回第一个元素和返回任意第一个元素

#### 代码

```java
// findFirst
@Test
public void findFirstTest() {
  List<User> users = User.init();
  Optional<User> first = users.stream()
    .findFirst();
  System.out.println(first);
}

// findAny
@Test
public void findAnyTest() {
  List<User> users = User.init();
  Optional<User> first = users.stream()
    .findAny();
  System.out.println(first);
}
```

#### 输出结果

```json
// findFirst
Optional[User(name=张三, age=18, sex=男)]

// findAny
Optional[User(name=张三, age=18, sex=男)]
```



### Min、Max 返回最小项，最大项

```java
// min
@Test
public void minTest() {
  List<User> users = User.init();
  Optional<User> min = users.stream()
    .min(Comparator.comparing(User::getAge));
  System.out.println(min);
}

// 结果
{
	"age":18,
	"name":"张三",
	"sex":"男"
}

// max
@Test
public void maxTest() {
  List<User> users = User.init();
  Optional<User> min = users.stream()
    .max(Comparator.comparing(User::getAge));
  System.out.println(JSONObject.toJSONString(min.get(),true));
}

// 结果
{
	"age":48,
	"name":"赵六",
	"sex":"女"
}
```

### count 计数

对 stream 流中的元素进行计数

```java
@Test
public void countTest() {
  List<User> users = User.init();
  long count = users.stream()
    .count();
  System.out.println(count);
}

// 结果
4
```

