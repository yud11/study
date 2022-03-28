## 准备工作

### 创建日志实体类

```java
@Entity
@Getter
@Setter
@Table(name = "sys_log")
@NoArgsConstructor
public class Log  implements Serializable {

    @Id
    @Column(name = "log_id")
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    /** 操作用户 */
    private String username;

    /** 描述 */
    private String description;

    /** 方法名 */
    private String method;

    /** 参数 */
    private String params;

    /** 日志类型 */
    private String logType;

    /** 请求ip */
    private String requestIp;

    /** 地址 */
    private String address;

    /** 浏览器  */
    private String browser;

    /** 请求耗时 */
    private Long time;

    /** 异常详细  */
    private byte[] exceptionDetail;

    /** 创建日期 */
    @CreationTimestamp
    private Timestamp createTime;

    public Log(String logType, Long time) {
        this.logType = logType;
        this.time = time;
    }
}
```

### Repository

```java
@Repository
public interface LogRepository extends JpaRepository<Log, Long> , JpaSpecificationExecutor<Log> {
    /**
    * @Description: 根据日志类型删除日志 
    * @Param: [logType]
    * @Return: void
    * @Date: 2020/6/26
    */
    @Modifying
    @Query(value = "delete from sys_log where logType = ?1",nativeQuery = true)
    void deleteByLogType(String logType);
}

```

### Service

> 保存日志的方法为异步调用

```java
public interface LogService {
    /**
     * 分页查询
     * @param criteria 查询条件
     * @param pageable 分页参数
     * @return /
     */
    Object queryAll(LogQueryCriteria criteria, Pageable pageable);

    /**
     * 查询全部数据
     * @param criteria 查询条件
     * @return /
     */
    List<Log> queryAll(LogQueryCriteria criteria);

    /**
     * 查询用户日志
     * @param criteria 查询条件
     * @param pageable 分页参数
     * @return -
     */
    Object queryAllByUser(LogQueryCriteria criteria, Pageable pageable);

    /**
     * 保存日志数据
     * @param username 用户
     * @param browser 浏览器
     * @param ip 请求IP
     * @param joinPoint /
     * @param log 日志实体
     */
    @Async
    void save(String username, String browser, String ip, ProceedingJoinPoint joinPoint, Log log);

    /**
     * 查询异常详情
     * @param id 日志ID
     * @return Object
     */
    Object findByErrDetail(Long id);

    /**
     * 删除所有错误日志
     */
    void delAllByError();

    /**
     * 删除所有INFO日志
     */
    void delAllByInfo();

}

```

### 查询辅助类

#### @Query

> 该注解类用于给查询字段添加属性信息

```java
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Query {

    //  基本对象的属性名
    String propName() default "";
    //  查询方式
    Type type() default Type.EQUAL;

    /**
     * 连接查询的属性名，如User类中的dept
     */
    String joinName() default "";

    /**
     * 默认左连接
     */
    Join join() default Join.LEFT;

    /**
     * 多字段模糊搜索，仅支持String类型字段，多个用逗号隔开, 如@Query(blurry = "email,username")
     */
    String blurry() default "";

    enum Type {
        //  相等
        EQUAL
        //  大于等于
        , GREATER_THAN
        //  小于等于
        , LESS_THAN
        //  中模糊查询
        , INNER_LIKE
        //  左模糊查询
        , LEFT_LIKE
        //  右模糊查询
        , RIGHT_LIKE
        //  小于
        , LESS_THAN_NQ
        //  包含
        , IN
        // 不等于
        ,NOT_EQUAL
        // between
        ,BETWEEN
        // 不为空
        ,NOT_NULL
        // 为空
        ,IS_NULL
    }

    /**
    * @Description: 适用于简单连接查询，复杂的请自定义该注解，或者使用sql查询 
    * @Param: 
    * @Return: 
    * @Date: 2020/6/29
    */
    enum Join {
        /**  2019-6-4 13:18:30 左右连接 */
        LEFT, RIGHT
    }

}
```

#### LogQueryCriteria 查询条件类

> blurry 模糊查询的字段，不带任何参数的@Query注解等同于精确匹配查询

```java
@Data
public class LogQueryCriteria {

    @Query(blurry = "username,description,address,requestIp,method,params")
    private String blurry;

    @Query
    private String logType;

    @Query(type = Query.Type.BETWEEN)
    private List<Timestamp> createTime;
}

```

