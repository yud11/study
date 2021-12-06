# 注册组件

## @Configuration

**@Configuration**：告诉spring这是个配置类，作用等同于配置文件的作用，可以给容器添加组件等。、

## @Bean

**@Bean**给容器里面添加一个组件，只要是spring容器能扫描的地方都能起到作用。value可以指定bean的id

## @ComponentScan

**@ComponentScan**：添加包扫描，该注解中有以下属性

 1）**basePackages**：扫描该基础包下的所有带有**@Component，@Service，@Controller，@Repository**注解的类

2）**excludeFilters**：排除掉指定的类，里面的属性为**@Filter**注解，**@Filter**的属性如下

​      1，type：指定要排除的类型

​      2，classes：指定要被排除的类

3）**includeFilters**：只扫描如下类型的类，属性和**excludeFilters**类似，includeFilters生效的条件，关闭默认过滤规则，useDefaultFilters属性要为false，

```java
@Configuration //配置类 作用等同于配置文件
@ComponentScan(value = "com.yudi",excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {Controller.class, Service.class})
})
//@ComponentScan(value = "com.yudi",includeFilters = {
//        @ComponentScan.Filter(type = FilterType.ANNOTATION,classes = {
//                Controller.class
//        })
//},useDefaultFilters = false)
public class AppConfig {
   @Bean("yudi") //给容器中添加一个bean,类型为返回值类型，id默认为方法名
    public Person person01() {
        return new Person("yudi",111);
    }
}
```

### 创建包扫描自定义规则

创建一个类实现TypeFilter，重写match方法

```java
public class MyFilter implements TypeFilter {

    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        String className = classMetadata.getClassName();
        if(className.contains("er")) {
            return true;
        }
        return false;
    }
}
```

主配置类上加入如下配置

```java
@ComponentScan(value = "com.yudi",includeFilters = {
    @ComponentScan.Filter(type = FilterType.CUSTOM,classes = {MyFilter.class})
},useDefaultFilters = false)
```

## @Scope

@Scope：指定bean的作用域，有以下几种

**singleton**：单实例，每次获取都返回同一个对象，容器创建，默认会创建bean

**prototype**： 多实例，容器创建并不会创建对象，调用时，才会创建，每次调用返回的都是一个新的对象

**request**：同一个请求创建一个实例

**session**：同一个session创建一个实例

## @Lazy

单实例bean默认情况下，容器创建，对象会被创建，使用**@Lazy**注解，在第一次获取bean时，创建对象，以及初始化。

## @Conditional

@Conditional：根据一定的条件给容器中加入bean。里面是实现了Condition接口的实现类

```java
@Bean("billgate") //给容器中添加一个bean,类型为返回值类型，id默认为方法名
@Lazy
@Conditional(WindowsCondition.class)
public Person person01() {
    return new Person("billgate",111);
}

public class WindowsCondition implements Condition {
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment environment = context.getEnvironment();
        String property = environment.getProperty("os.name");
        System.out.println(property);
        if(property.contains("Windows")) {
            return true;
        }
        return false;
    }
}
```

## 容器导入bean的三种方式

1. 包扫描+@Component，@Controller等注解（适用于自己写的bean）

2. @Bean，适用于第三方jar包

3. @Import，快速导入类，id默认为全类名。**必须有无参方法**

   1）@Import（要被导入的类）

   2）@Import（ImportSelector），返回要导入的类的全类名

   3） @Import（ImportBeanDefinitionRegistrar）：将类注入到容器中

4. 使用spring提供的FactoryBean（工厂bean）

   1）默认返回的是工厂bean调用getObject创建的对象

   2）要获取工厂bean本身，需要给id加一个&

```java
@Configuration //配置类 作用等同于配置文件
@Import({Color.class, MyImpotSelector.class, MyImportBeanDefinitionRegistrar.class})
public class AppConfig {

    @Bean("billgate") //给容器中添加一个bean,类型为返回值类型，id默认为方法名
    @Lazy
    @Conditional(WindowsCondition.class)
    public Person person01() {
        return new Person("billgate",111);
    }
}
```

```java
public class MyImpotSelector implements ImportSelector {
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{"com.yudi.bean.Blue","com.yudi.bean.Red"};
    }
}

public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {

    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        boolean red = registry.containsBeanDefinition("com.yudi.bean.Red");
        boolean blue = registry.containsBeanDefinition("com.yudi.bean.Blue");
        if (red && blue) {
            RootBeanDefinition beanDefinition = new RootBeanDefinition("com.yudi.bean.RainBow");
            registry.registerBeanDefinition("raiwBow",beanDefinition);
        }
    }
}
```

# bean生命周期

我们可以自定义初始化方法和销毁方法，容器在bean进行到当前生命周期时，会回调我们写的初始化方法和销毁方法。

## 生命周期时机

### 构造：

​	单实例：在容器创建时，创建对象

​	多实例：每次获取时，创建对象

### 初始化：

​	对象创建完成，并赋好值，调用初始化方法

### 销毁：

​	单实例：容器关闭时

​	多实例：容器不会调用该销毁方法。因为多实例容器不负责管理

## 生命周期钩子函数的指定

### 1，通过@Bean指定init-method和destroy-method

```java
@Scope("prototype")
@Bean(initMethod = "init",destroyMethod = "destroy")
public Car car() {
    return new Car();
}


public class Car {

    public  Car() {
        System.out.println(" car constructor ...");
    }

    public void init() {
        System.out.println("car init .....");
    }

    public void destroy() {
        System.out.println("car destroy......");
    }
}
```

### 2,通过实现InitializingBean, DisposableBean接口，回调初始化和销毁方法。

```java
@Component
public class Cat implements InitializingBean, DisposableBean {

    public Cat() {
    }

    public void destroy() throws Exception {
        System.out.println("cat destroy.....");
    }

    public void afterPropertiesSet() throws Exception {
        System.out.println("cat afterPropertiesSet.....");
    }
}
```

### 3,使用JSR-250规范

​		@PostConstruct：在bean创建完成并且属性赋值完成，来执行初始化方法

​		@PreDestroy：在容器销毁bean之前通知我们进行清理工作

```java
@Component
public class Dog {

    public Dog() {
    }

    @PostConstruct
    public void init(){
        System.out.println("dog  @PostConstruct ....");
    }

    @PreDestroy
    public void destroy(){
        System.out.println("dog @PreDestroy ..... ");
    }
}
```

同时使用上面三种方式来为bean设置生命周期函数，结果如下

```java
public class Dog implements InitializingBean, DisposableBean {

    public Dog() {
    }

    @PostConstruct
    public void init(){
        System.out.println("dog  @PostConstruct ....");
    }

    @PreDestroy
    public void destroy1(){
        System.out.println("dog @PreDestroy ..... ");
    }

    public void afterPropertiesSet() throws Exception {
        System.out.println("dog InitializingBean .......... ");
    }

    public void destroy() throws Exception {
        System.out.println("dog DisposableBean .......... ");
    }

    public void init1() {
        System.out.println("dog @bean init-method .......... ");
    }

    public void destroy3(){
        System.out.println("dog @bean destroy-method .......... ");
    }
}

@Bean(initMethod = "init1",destroyMethod = "destroy3")
public Dog dog(){
    return new Dog();
}
------------------------------------------------------
dog  @PostConstruct ....
dog InitializingBean .......... 
dog @bean init-method .......... 
dog @PreDestroy ..... 
dog DisposableBean .......... 
dog @bean destroy-method ..........    
```

### 4,实现BeanPostProcessor接口

BeanPostProcessor：bean的后置处理器，负责在bean的初始化前后进行一些处理工作

```java
@Component
public class MyPostProcessor implements BeanPostProcessor {
    
    //在bean的初始化之前调用
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("MyPostProcessor before .... " + beanName);
        return bean;
    }
	//在bean的初始化方法之后调用
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("MyPostProcessor after .... " + beanName);
        return bean;
    }
}
```

调用结果如下

```java
MyPostProcessor before .... appConfig
MyPostProcessor after .... appConfig
MyPostProcessor before .... dog
dog  @PostConstruct ....
dog InitializingBean .......... 
dog @bean init-method .......... 
MyPostProcessor after .... dog
```

# 属性注入

## @Value

@Value：给属性注入值

```java
@Getter
@Setter
@ToString
@NoArgsConstructor
public class Person {

    //注入字面量
    @Value("yudi")
    private String name;

    //写SpEL表达式
    @Value("#{10-2}")
    public  Integer age;

    //从配置文件中取值
    @Value("${person.nikeName}")
    public String nikeName;

    public Person(String name, Integer age) {
        this.name = name;
        this.age = age;
        System.out.println("person init");
    }
}
```

# 自动装配

## @AutoWired

@AutoWired：自动注入依赖

1）默认按照类型进行装配

2）如果找到多个类型相同的bean，再将属性作为bean的id，去容器中找

3）@Qualifier（“bookDao”），使用@Qualifier指定需要装配的组件的id，而不是使用属性名

4）自动装配默认一定要将属性赋值好，没有就会报错。可以使用`@Autowired(required = false)`

5) @Primary:让spring自动装配时，默认使用首选的bean，也可以使用@Qualifier去找指定的id的bean

```java
@Primary //当容器中有多个Dog类型的bean时，首选如下的bean，当使用@Qualifier时，该注解无效
@Bean(initMethod = "init1",destroyMethod = "destroy3")
public Dog dog(){
    return new Dog();
}
```

### 使用地方

@AutoWired可以标注在方法，构造方法，参数，属性等，都是从容器中获取值

1）在方法参数上：@Bean+方法参数，参数从容器中获取，默认不写@AutoWired，结果一样

2）在构造器上：如果只有一个有参构造器，那么@AutoWired可以不写

## @Resource

@Resource是JSR250规范中的一个注解，它也是用于自动注入属性的，默认是按照属性名作为id去查找bean，也可以指定id去容器中查找。不能和@Primary注解配合使用，也不支持`required = false`的功能。

## @Inject

@Inject是JSR330规范中的一个注解，需要导入javax.inject依赖。它也是用于自动注入属性的，它跟@AutoWired类似。不支持`required = false`的功能。

|            | 注解类型   | 是否能和@Primary配合使用 | 属性的值是否能为null |
| ---------- | ---------- | ------------------------ | -------------------- |
| @AutoWired | spring注解 | 是                       | 是                   |
| @Resource  | JSR250注解 | 否                       | 否                   |
| @Inject    | JSR330注解 | 是                       | 否                   |

## 获取spring底层的对象

组件实现xxxAware，spring会将特定对象注入进来

```java
@Component
public class Blue implements ApplicationContextAware, BeanNameAware, EmbeddedValueResolverAware {

    private ApplicationContext applicationContext;

    public void setBeanName(String name) {
        System.out.println(name);
    }

    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;

        System.out.println("blue de application context");
        String[] names = this.applicationContext.getBeanDefinitionNames();

        for (String name : names) {
            System.out.println(name);
        }
    }

    public void setEmbeddedValueResolver(StringValueResolver resolver) {

        String value = resolver.resolveStringValue("系统是${os.name} #{10*20}");
        System.out.println(value);
    }
}
```

## @Profile

1. 加了环境标识的bean，只有这个环境被激活时才能被注册到容器中，默认是default环境
2. 写在配置类上，只有是指定的环境时，整个配置类的配置才能生效
3. 没有标注的环境标识的bean在任何环境下都是加载的

# AOP

## AOP Demo

1，导入aop模块；Spring AOP：(spring-aspects)

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-aspects</artifactId>
    <version>5.2.1.RELEASE</version>
</dependency>
```

2,定义一个业务逻辑类（MathCalculator）；在业务逻辑运行的时候将日志进行打印（方法之前、方法运行结束、方法出现异常，xxx）

```java
public class MathCaculate {
    //业务逻辑方法
    public int div(int i,int j) {
        System.out.println(i/j);
        return i/j;
    }
}
```

3,定义一个日志切面类（LogAspects）：切面类里面的方法需要动态感知MathCalculator.div运行到哪里然后执行；**通知方法**分为如下

 * 			前置通知(@Before)：logStart：在目标方法(div)运行之前运行
 * 			后置通知(@After)：logEnd：在目标方法(div)运行结束之后运行（无论方法正常结束还是异常结束）
 * 			返回通知(@AfterReturning)：logReturn：在目标方法(div)正常返回之后运行
 * 			异常通知(@AfterThrowing)：logException：在目标方法(div)出现异常以后运行
 * 			环绕通知(@Around)：动态代理，手动推进目标方法运行（joinPoint.procced()）

4，给切面类的目标方法标注何时何地运行（通知注解）；

5,必须告诉Spring哪个类是切面类(给切面类上加一个注解：@Aspect)

```java
@Aspect
public class LogAspect {

    @Pointcut("execution(public int com.yudi.aop.MathCaculate.*(..))")
    public void joinCut(){}


    @Before("joinCut()")
    public void logStart() {
        System.out.println("log start..............");
    }

    @After("joinCut()")
    public void logEnd(){
        System.out.println("log end ...................");
    }

    @AfterReturning(value = "joinCut()",returning = "result")
    public void returning(JoinPoint joinPoint,Integer result){
        System.out.println(joinPoint.getSignature().getName() +"log afterreturning {"+result+"}");
    }

    @AfterThrowing(value = "joinCut()",throwing = "ex")
    public void logException(JoinPoint joinPoint,Exception ex){
        System.out.println("异常。。。异常信息：{"+ex+"}");
    }

}
```

6,将切面类和业务逻辑类（目标方法所在类）都加入到容器中;

7,给配置类中加 **@EnableAspectJAutoProxy** 【开启基于注解的aop模式】

```java
@Configuration
@EnableAspectJAutoProxy
public class MainConfigOfAop {

    @Bean
    public MathCaculate calculate(){
        return new MathCaculate();
    }

    @Bean
    public LogAspect logaspects(){
        return  new LogAspect();
    }
}
```

