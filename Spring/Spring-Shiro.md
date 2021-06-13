# Shiro 入门

## Shiro 核心API

**Subject**: 用户主体 （把操作交给**SecurityManager**）

**SecurityManager**: 安全管理器 （关联Realm）

**Realam**: 连接数据的桥梁



## SpringBoot整合Shiro

引入依赖

```xml
<dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring-boot-starter</artifactId>
            <version>1.4.0</version>
        </dependency>
```

###  自定义Realm

定义个类，继承自AuthorizingRealm类，并实现其中的两个方法

doGetAuthorizationInfo：执行授权逻辑

doGetAuthenticationInfo：执行认证逻辑

```java
public class UserRealm extends AuthorizingRealm {
    //执行授权逻辑
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        return null;
    }

    //执行认证逻辑
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        return null;
    }
}
```



## 编写Shiro 配置类

- 创建ShiroFilterFactoryBean
- 创建DefaultWebSecurityManager
- 创建自定义Realm

```java
@Configuration
public class ShiroConfig {

    /**
     * 创建ShiroFilterFactoryBean
     */
    @Bean
    public ShiroFilterFactoryBean getShiroFilterFactoryBean(DefaultWebSecurityManager defaultWebSecurityManager){
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        //设置安全管理器
        shiroFilterFactoryBean.setSecurityManager(defaultWebSecurityManager);

        /**
         * Shiro内置过滤器，可以实现权限相关的拦截器
         * 常用的过滤器:
         * **anon**：无需认证（登录）即可访问
         *
         * **authc**：必须认证才可以访问
         *
         * **user**：如果使用rememberme功能才可以访问
         *
         * **perms**：该资源必须得到资源权限才可以访问
         *
         * **role**：该资源必须得到角色权限才可以访问
         */
        Map<String,String> filterMap = new LinkedHashMap<>();
        filterMap.put("/user/**","authc");
        filterMap.put("/login/**","anon");
        //设置上传下载权限
        filterMap.put("/uploadFile","perms[file:upload]");
        filterMap.put("/downloadFile","perms[file:download]");

        shiroFilterFactoryBean.setLoginUrl("/login/test");
        //设置过滤规则
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);
        return shiroFilterFactoryBean;
    }

    /**
     *创建DefaultWebSecurityManager
     */
    @Bean(name = "defaultWebSecurityManager")
    public DefaultWebSecurityManager getDefaultWebSecurityManager(UserRealm userRealm){
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        defaultWebSecurityManager.setRealm(userRealm);
        return defaultWebSecurityManager;
    }

    /**
     *创建自定义Realm
     */
    @Bean(name = "userRealm")
    public UserRealm getUserRealm(){
        return new UserRealm();
    }
}
```



## 使用Shiro内置过滤器实现请求拦截

Shiro内置过滤器，可以实现权限相关的拦截器

常用的过滤器：

**anon**：无需认证（登录）即可访问

**authc**：必须认证才可以访问

**user**：如果使用rememberme功能才可以访问

**perms**：该资源必须得到资源权限才可以访问

**role**：该资源必须得到角色权限才可以访问

修改shiro.config ，增加拦截配置

```java
Map<String,String> filterMap = new LinkedHashMap<>();
        filterMap.put("/user/**","authc");
        filterMap.put("/login/**","anon");

        shiroFilterFactoryBean.setLoginUrl("/login/toLogin");
        //设置过滤规则
        shiroFilterFactoryBean.setFilterChainDefinitionMap(filterMap);
        return shiroFilterFactoryBean;
```



## 整合mybatis

```xml
<dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.2</version>
        </dependency>

        <!-- https://mvnrepository.com/artifact/com.baomidou/mybatis-plus-boot-starter -->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus-boot-starter</artifactId>
            <version>3.3.0</version>
        </dependency>



        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.16</version>
        </dependency>
```

application.properties

```properties
## 数据源配置
spring.datasource.url=jdbc:mysql://xx.xxx.xx.xx:3306/shiro?characterEncoding=utf8 & useSSL=false & serverTimezone=UTC & rewriteBatchedStatements=true
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

mybatis 配置

```properties
##mybatis 配置
mybatis.mapper-locations=classpath:mapper/*.xml
#别名配置
mybatis.type-aliases-package=com.he.shiro.model
#打印日志
logging.level.com.he.shiro.dao=debug
```

## 在自定义realm中进行认证与授权逻辑

```java
public class UserRealm extends AuthorizingRealm {

    @Autowired
    private IUserService userService;

    @Autowired
    private PermisionService permisionService;

    //执行授权逻辑
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();

        Subject subject = SecurityUtils.getSubject();
        User user = (User) subject.getPrincipal();
        List<Permision> permisions = permisionService.queryByUserId(user.getId());

        for (Permision permision : permisions) {
            info.addStringPermission(permision.getPermison());
        }
        //info.addStringPermission("user:add");
        return info;
    }

    //执行认证逻辑
    @Override
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {

        UsernamePasswordToken token = (UsernamePasswordToken)authenticationToken;
        User user = userService.findByUserName(token.getUsername());
        if(user==null){
            return null;
        }
        return new SimpleAuthenticationInfo(user,user.getPassword(),"");
    }
}
```



那么所登陆的账号有了对应的权限（permission）才能访问响应的接口



## Shiro注解

Shiro常用注解

**RequiresAuthentication**

使用该注解标注的类，实例，方法在访问或调用时，当前Subject必须在当前session中已经过认证。

**RequiresGuest**

使用该注解标注的类，实例，方法在访问或调用时，当前Subject可以是“gust”身份，不需要经过认证或者在原先的session中存在记录。游客

**RequiresPermissions**

当前Subject需要拥有某些特定的权限时，才能执行被该注解标注的方法。如果当前Subject不具有这样的权限，则方法不会被执行。

```java
//符合index:hello权限要求
@RequiresPermissions("index:hello")
 
//必须同时复核index:hello和index:world权限要求
@RequiresPermissions({"index:hello","index:world"})
 
//符合index:hello或index:world权限要求即可
@RequiresPermissions(value={"index:hello","index:world"},logical=Logical.OR)
```

**RequiresRoles**

当前Subject必须拥有所有指定的角色时，才能访问被该注解标注的方法。如果当天Subject不同时拥有所有指定角色，则方法不会执行还会抛出AuthorizationException异常。

```java
RequiresRoles("user")
 
//必须同时属于user和admin角色
@RequiresRoles({"user","admin"})
 
//属于user或者admin之一;修改logical为OR 即可
@RequiresRoles(value={"user","admin"},logical=Logical.OR)
```



**RequiresUser**

当前Subject必须是应用的用户，才能访问或调用被该注解标注的类，实例，方法。



## Shiro密码的比对

自定义Rralm是继承自**AuthorizingRealm**，**AuthorizingRealm**继承自**AuthenticatingRealm**

密码比对时会调用**AuthenticatingRealm**的CredentialsMatcher（凭证匹配器来进行比对）

```java
public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
        Object tokenCredentials = getCredentials(token);
        Object accountCredentials = getCredentials(info);
        return equals(tokenCredentials, accountCredentials);
    }
```

**HashedCredentialsMatcher** 的密码比对（最终实际上也是使用了SimpleHash进行加密，返回加密的值与数据库中查出来的值进行比对）

```java
@Override
    public boolean doCredentialsMatch(AuthenticationToken token, AuthenticationInfo info) {
        Object tokenHashedCredentials = hashProvidedCredentials(token, info);
        Object accountCredentials = getCredentials(info);
        return equals(tokenHashedCredentials, accountCredentials);
    }

 protected Object hashProvidedCredentials(AuthenticationToken token, AuthenticationInfo info) {
        Object salt = null;
        if (info instanceof SaltedAuthenticationInfo) {
            salt = ((SaltedAuthenticationInfo) info).getCredentialsSalt();
        } else {
            //retain 1.0 backwards compatibility:
            if (isHashSalted()) {
                salt = getSalt(token);
            }
        }
        return hashProvidedCredentials(token.getCredentials(), salt, getHashIterations());
    }

protected Hash hashProvidedCredentials(Object credentials, Object salt, int hashIterations) {
        String hashAlgorithmName = assertHashAlgorithmName();
   		 //实际也是使用了SimpleHash
        return new SimpleHash(hashAlgorithmName, credentials, salt, hashIterations);
    }


```



## Shiro密码的加密

需要使用到**CredentialsMatcher**（凭证匹配器），需要为我们的自定义Rleam设置一个凭证匹配器（如果不设置，那么就走默认最简单的匹配）。

这里我们创建一个**HashedCredentialsMatcher**,并指定加密算法MD5，迭代次数2次，对密码使用加盐（在这里不用具体指定盐的值，在Realm中指定具体的值即可）

```java
@Bean(name = "hashedCredentialsMatcher")
    public HashedCredentialsMatcher hashedCredentialsMatcher(){
        HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
        hashedCredentialsMatcher.setHashAlgorithmName("MD5");
        hashedCredentialsMatcher.setHashIterations(2);
        hashedCredentialsMatcher.setHashSalted(true);
        return hashedCredentialsMatcher;
}
```

将**HashedCredentialsMatcher**交给我们的自定义Realm

```java
@Bean(name = "userRealm")
    public UserRealm getUserRealm(CredentialsMatcher hashedCredentialsMatcher){
        UserRealm userRealm = new UserRealm();
        userRealm.setCredentialsMatcher(hashedCredentialsMatcher);
        return userRealm;
    }
```

在执行认证逻辑的时候记得给出具体的盐值，在自定义Realm中的doGetAuthenticationInfo认证时指定

```java
 return new SimpleAuthenticationInfo(ShiroUser,ShiroUser.getPassword(), ByteSource.Util.bytes("hening"),"");
```

如果在Shiro验证的时候指定了加密和加盐，那么当用户注册的时候就必须也要进行加密处理才能入库

实现一个简单的加密，将加密后的值作为密码存入数据库中，Shiro在进行认证就能成功认证了

```java
@Test
    private void simple(){
        String password= "123456";
        Integer times = 2;
        SimpleHash simpleHash = new SimpleHash("MD5",password, ByteSource.Util.bytes("hening"),times);
        System.out.println(simpleHash.toString());
    }
```

**MD5 迭代两次加密** 结果

4280d89a5a03f812751f504cc10ee8a5

4280d89a5a03f812751f504cc10ee8a5

**MD5 迭代两次 salt=hening** 结果

8d708c4f41acac7040e9a1ea8e863ac6

8d708c4f41acac7040e9a1ea8e863ac6

## Session 的监听

Shiro提供了**SessionListener**接口实现对Session的监听，其中有三个方法分别对创建、销毁、过期进行监听，一般我们使用时直接继承实现了**SessionListener**接口的**SessionListenerAdapter**，然后重新我们需要的方法即可

```java
/**
 * Session 的监听
 */
public class MySessionLisener extends SessionListenerAdapter {

    @Override
    public void onStart(Session session) {
        //super.onStart(session);
        System.out.println("###测试SessionListener:Start");
    }

    @Override
    public void onStop(Session session) {
        //super.onStop(session);
        System.out.println("###测试SessionListener:Stop");
    }
}
```

创建好我们自己的MySessionLisener后，需要将它注入到Spring容器中，或者直接newc出来交由**SessionManager**管理，当然最后**SessionManager**最后也会交由**SecurityManager**管理

```java
@Bean
    public SessionManager sessionManager() {
        MySessionManager mySessionManager = new MySessionManager();
        //mySessionManager.setSessionDAO();
        //mySessionManager.setSessionDAO(redisSessionDAO());
        //设置Session监听器
        List<SessionListener> sessionListeners = new ArrayList<>();
        sessionListeners.add(sessionListener());
        mySessionManager.setSessionListeners(sessionListeners);

        return mySessionManager;
    }
```



## Session的参数修改

也可以认为是对返回给浏览器的Cookie的修改，主要是设置Cookie名称、过期时间等信息

一般使用默认的SimpleCookie即可，new一个SimpleCookie出来，然后为其设置值，最后同样交给SessionManager管理

```java
@Bean
    public SimpleCookie simpleCookie(){
        SimpleCookie simpleCookie = new SimpleCookie();
        //设置Cookie的属性值
        return simpleCookie;
    }
```



## Shiro Remeberme功能





## Thymeleaf 整合 Shiro

引入依赖

```xml
<dependency>
    <groupId>com.github.theborakompanioni</groupId>
    <artifactId>thymeleaf-extras-shiro</artifactId>
    <version>2.0.0</version>
</dependency>
```

在ShiroConfig中注入Bean

```java
@Bean
public ShiroDialect shiroDialect() {
    return new ShiroDialect();
}
```

编写Role 实体类和查询方法等

在自定义Realm中完成角色授权

```java
//执行授权逻辑
    @Override
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        SimpleAuthorizationInfo info = new SimpleAuthorizationInfo();

        Subject subject = SecurityUtils.getSubject();
        User user = (User) subject.getPrincipal();
        List<Permision> permisions = permisionService.queryByUserId(user.getId());
        Set<String> roles = roleService.queryByUserId(user.getId()).stream().map(Role::getRole).collect(Collectors.toSet());

        for (Permision permision : permisions) {
            info.addStringPermission(permision.getPermison());
        }
        info.setRoles(roles);

        //info.addStringPermission("user:add");
        return info;
    }
```



在html页面使用shiro标签进行权限管理

首先要引入命名空间

```html
<html lang="en" xmlns:th="http://www.thymeleaf.org"
      xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">
```



在需要权限控制的地方写上标签

通过角色控制 shiro:hasRole="admin"

```html
<li shiro:hasRole="admin" class="layui-nav-item"><a>产品</a></li>
<li shiro:hasRole="test" class="layui-nav-item"><a >大数据</a></li>
```

通过资源权限控制 shiro:hasPermission="user:update"

```html
<li shiro:hasPermission="user:update" class="layui-nav-item layui-nav-itemed">
    <a href="javascript:;">默认展开</a>
    <dl class="layui-nav-child">
        <dd><a th:href="@{/user/add}" th:class="${currUri=='add'?'layui-this':''}">add</a></dd>
        <dd><a th:href="@{/user/update}" th:class="${currUri=='update'?'layui-this':''}">update</a></dd>
        <dd><a >跳转</a></dd>
    </dl>
</li>
<li shiro:hasPermission="user:add" class="layui-nav-item">
    <a href="javascript:;">解决方案</a>
    <dl class="layui-nav-child">
        <dd><a >移动模块</a></dd>
        <dd><a >后台模版</a></dd>
        <dd><a >电商平台</a></dd>
    </dl>
</li>
```

### 常用标签





## 前后端分离使用Shiro进行认证

### 实现自己的Session管理器

```java
@Slf4j
public class MySessionManager extends DefaultWebSessionManager {

    private String authorization = "Authorization";

    /**
     * 重写获取sessionId的方法调用当前Manager的获取方法
     *
     * @param request
     * @param response
     * @return
     */
    @Override
    protected Serializable getSessionId(ServletRequest request, ServletResponse response) {
        return this.getReferencedSessionId(request, response);
    }

    /**
     * 获取sessionId从请求中
     *
     * @param request
     * @param response
     * @return
     */
    private Serializable getReferencedSessionId(ServletRequest request, ServletResponse response) {
        String id = this.getSessionIdCookieValue(request, response);
        if (id != null) {
            request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_SOURCE, "cookie");
        } else {
            //如果为空，可以设置不用下面这步骤，直接从id = WebUtils.toHttp(request).getHeader(this.authorization)请求头中获取
            //根据自己需要，进行编写
            id = this.getUriPathSegmentParamValue(request, "JSESSIONID");
            if (id == null) {
                // 获取请求头中的session
                id = WebUtils.toHttp(request).getHeader(this.authorization);
                if (id == null) {
                    String name = this.getSessionIdName();
                    //从请求参数中获取
                    id = request.getParameter(name);
                    if (id == null) {
                        id = request.getParameter(name.toLowerCase());
                    }
                }
            }
            if (id != null) {
                request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_SOURCE, "url");
            }
        }

        if (id != null) {
            request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID, id);
            request.setAttribute(ShiroHttpServletRequest.REFERENCED_SESSION_ID_IS_VALID, Boolean.TRUE);
        }

        return id;
    }

    // copy super
    private String getSessionIdCookieValue(ServletRequest request, ServletResponse response) {
        if (!this.isSessionIdCookieEnabled()) {
            log.debug("Session ID cookie is disabled - session id will not be acquired from a request cookie.");
            return null;
        } else if (!(request instanceof HttpServletRequest)) {
            log.debug("Current request is not an HttpServletRequest - cannot get session ID cookie.  Returning null.");
            return null;
        } else {
            HttpServletRequest httpRequest = (HttpServletRequest) request;
            //这里
            return this.getSessionIdCookie().readValue(httpRequest, WebUtils.toHttp(response));
        }
    }

    // copy super
    private String getUriPathSegmentParamValue(ServletRequest servletRequest, String paramName) {
        if (!(servletRequest instanceof HttpServletRequest)) {
            return null;
        } else {
            HttpServletRequest request = (HttpServletRequest) servletRequest;
            String uri = request.getRequestURI();
            if (uri == null) {
                return null;
            } else {
                int queryStartIndex = uri.indexOf(63);
                if (queryStartIndex >= 0) {
                    uri = uri.substring(0, queryStartIndex);
                }

                int index = uri.indexOf(59);
                if (index < 0) {
                    return null;
                } else {
                    String TOKEN = paramName + "=";
                    uri = uri.substring(index + 1);
                    index = uri.lastIndexOf(TOKEN);
                    if (index < 0) {
                        return null;
                    } else {
                        uri = uri.substring(index + TOKEN.length());
                        index = uri.indexOf(59);
                        if (index >= 0) {
                            uri = uri.substring(0, index);
                        }

                        return uri;
                    }
                }
            }
        }
    }
    // copy super
    private String getSessionIdName() {
        String name = this.getSessionIdCookie() != null ? this.getSessionIdCookie().getName() : null;
        if (name == null) {
            name = "JSESSIONID";
        }
        return name;
    }
}
```

### 将MySessionManager 交由Spring管理

在Shiro统一配置类中，配置MySessionManager 

```java

    //自定义sessionManager
    @Bean
    public SessionManager sessionManager() {

        //mySessionManager.setSessionDAO(redisSessionDAO());
        return new MySessionManager();
    }
```

将SessionManager交由SecurityManager管理

```java
 @Bean(name = "defaultWebSecurityManager")
    public DefaultWebSecurityManager getDefaultWebSecurityManager(UserRealm userRealm){
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        defaultWebSecurityManager.setRealm(userRealm);
        //将SessionManager交由SecurityManager管理
        defaultWebSecurityManager.setSessionManager(sessionManager());
        return defaultWebSecurityManager;
}
```



这样Shiro在进行认证的时候就会使用我们实现的SessionManager，Shiro能够拿到我们传过去的Token（JSESSIONID），而这个值是登录时使用Shiro认证后，Shiro生成的token（SessionId）（**SecurityUtils.getSubject().getSession().getId()8**），返回给前段后，由前端存储在Cookie中，发送其他请求时会携带Cookie，那么就能成功通过认证了。

```java
map.put("id",shiroUser.getId());
        map.put("code","200");
        map.put("username",username);
        map.put("token",SecurityUtils.getSubject().getSession().getId());
        return map;
```

