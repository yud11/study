# Session 会话管理

---

## 一、Session 生成策略

### 生成策略介绍：

Spring Security 对于Session 的管理有 4 种生成策略：

* Alawys：如果当前请求没有 Session 存在，那么就创建一个新的 Session
* Never：Spring Security 永远不会主动创建 Session，只会使用已经存在的 Session 
* If_requeired（默认）：Spring Security 只在需要的时候创建 Session
* Stateless：Spring Security 不会创建和使用任何 Session，适合于接口型的无状态应用

这 4 种创建 Session 的策略可以在枚举类 `SessionCreationPolicy` 中查看

```java
public enum SessionCreationPolicy {
	/** Always create an {@link HttpSession} */
	ALWAYS,
	/**
	 * Spring Security will never create an {@link HttpSession}, but will use the
	 * {@link HttpSession} if it already exists
	 */
	NEVER,
	/** Spring Security will only create an {@link HttpSession} if required */
	IF_REQUIRED,
	/**
	 * Spring Security will never create an {@link HttpSession} and it will never use it
	 * to obtain the {@link SecurityContext}
	 */
	STATELESS
}
```

### 配置生成策略

> Session 的生成策略在 Security 的配置类中进行配置

```java
.authenticated()
    .and()
 //启动 Session管理   
.sessionManagement()
 //配置生成策略
.sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED);
```

## 二、Session 会话超时配置

> Spring boot 对于 Session的会话超时配置可以在 yml 文件中进行配置，方式如下
>
> Spring boot 配置的 Session 默认失效时间为 30min，并且如果小于 60s 的话一律按照 1min 进行计时 

```yaml
server:
  port: 8080
  servlet:
    session:
      timeout: 10s
```

### 过期的 Session处理

> 我们需要在 Security 配置类中对 Session 的过期问题进行处理，如果之前的 `SESSIONID`过期了，那么此时再次访问服务器资源就会重定位到`处理页面`

```java
.sessionManagement()
                .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
    			//当 Session 无效的时候，让浏览器重定向到 reLogin.html 进行重新登录
                .invalidSessionUrl("/reLogin");
```

### Session 保护

> 默认情况下，Spring Security 启用了 migrationSession 保护方式。即对同一个cookies的 `SESSIONID`用户，每次登录验证将创建一个新的 HTTP 会话
>
> 旧的 HTTP 会话将无效，并且旧会话的属性将被复制

* newSession()：不保留原来的 session 信息，并创建一个新的 session 会话
* migrateSession()：保留原来的 session信息，并创建一个新的 session会话
* changeSessionId()：指定使用 Servlet 提供的 session 会话服务，修改 SESSIONID 并保留之前所有的 session信息
* none()：不启用会话保护

### 限制登录用户数量

> 一般情况下，我们会有两种需求
>
> 1、用户登录后，相同的账户只能在一台电脑上登录，其他电脑上登录的话，之前的电脑会被强制下线
>
> 2、用户登录后，无法在其他电脑上再次登录该账号

#### Spring Security 配置类

```java
.and()
                .sessionManagement()
                .invalidSessionUrl("/reLogin")
                .sessionCreationPolicy(SessionCreationPolicy.IF_REQUIRED)
                .sessionFixation()
                .migrateSession()
                //限制登录用户为 1
                .maximumSessions(1)
                //设置用户登录或其他设备上是否还可以登录，true 为不可以登录，false 为可以登录，之前的设备会强制下线
                .maxSessionsPreventsLogin(false)
    			.expiredSessionStrategy(new MySessionExpiredStrategy());
```

#### 实现 SessionInformationExpiredStrategy 接口

> 实现 SessionInformationExpiredStrategy 接口，定义处理规则和返回参数，当 session 过期的时候触发SessionInformationExpiredEvent 事件

```java
public class MySessionExpiredStrategy implements SessionInformationExpiredStrategy {

    @Override
    public void onExpiredSessionDetected(SessionInformationExpiredEvent sessionInformationExpiredEvent) throws IOException, ServletException {
        //当用户在其他设备登录时，提示当前设备已经下线
        String s = JSON.toJSONString(new Result().error(403, "你已经在另一个设备登录"));
        sessionInformationExpiredEvent.getResponse().setContentType("application/json;charset=UTF-8");
        sessionInformationExpiredEvent.getResponse().getWriter().write(s);
    }
}
```

