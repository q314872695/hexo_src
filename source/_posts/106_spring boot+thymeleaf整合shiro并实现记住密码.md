---
title: spring boot+thymeleaf整合shiro并实现记住密码
tags:
  - shiro
  - spring boot
abbrlink: 1ce08af
date: 2020-06-24 17:25:23
---

> 至于在spring boot中使用shiro,配置起来就简单多了，具体如下
# 创建一个spirng boot项目，导入依赖
在创建时除了勾选web依赖、lombok依赖外，还要手动添加以下依赖
```xml
	<!--spring boot 中使用shiro-->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring-boot-web-starter</artifactId>
            <version>1.5.3</version>
        </dependency>
        <!--引用ehcache缓存-->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-ehcache</artifactId>
            <version>1.5.3</version>
        </dependency>
        <!--数据库驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
        </dependency>
        <!--thymeleaf模板-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-thymeleaf</artifactId>
        </dependency>
        <!--thymelead使用shiro标签-->
        <dependency>
            <groupId>com.github.theborakompanioni</groupId>
            <artifactId>thymeleaf-extras-shiro</artifactId>
            <version>2.0.0</version>
        </dependency>
	<!--java工具类封装了各种方法简化操作(主要用来访问数据库)-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.3.7</version>
        </dependency>
```

# 有关shiro的配置类
```java
@Configuration
public class ShiroConfig {
    // 创建web安全管理器
    @Bean
    public DefaultWebSecurityManager getDefaultWebSecurityManager(CustomRealm realm,CookieRememberMeManager cookieRememberMeManager) {
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        // 设置自定义Realm
        defaultWebSecurityManager.setRealm(realm);
        // 设置记住密码管理器      
        defaultWebSecurityManager.setRememberMeManager(cookieRememberMeManager);
        return defaultWebSecurityManager;
    }
    // 配置url过滤器
    @Bean
    public ShiroFilterChainDefinition shiroFilterChainDefinition() {
        DefaultShiroFilterChainDefinition chainDefinition = new DefaultShiroFilterChainDefinition();
        // 静态资源直接访问
        chainDefinition.addPathDefinition("/css/*","anno");
        chainDefinition.addPathDefinition("/js/*","anno");
        chainDefinition.addPathDefinition("/image/*","anno");
        // 登录注册页直接访问
        chainDefinition.addPathDefinition("/login", "anon");
        chainDefinition.addPathDefinition("/register", "anon");
        chainDefinition.addPathDefinition("/logout", "logout");
        // 只有登录或者勾选记住密码的用户才能访问
        chainDefinition.addPathDefinition("/**", "user");
        return chainDefinition;
    }

    // 密码匹配器
    @Bean
    public HashedCredentialsMatcher getHashedCredentialsMatcher() {
        HashedCredentialsMatcher credentialsMatcher = new HashedCredentialsMatcher();
        credentialsMatcher.setHashAlgorithmName("md5");  // 散列算法，这里使用更安全的sha256算法
        credentialsMatcher.setHashIterations(1024);  // 散列迭代次数
        return credentialsMatcher;
    }
    // Eh缓存管理器
    @Bean
    public EhCacheManager getEhCacheManager(){
        return new EhCacheManager();
    }

    // 加入shiro的方言配置，否则thymeleaf不解析shiro标签
    @Bean
    public ShiroDialect getShiroDialect(){
        return new ShiroDialect();
    }

    // cookie记住密码
    @Bean
    public SimpleCookie getSimpleCookie(){
        SimpleCookie simpleCookie = new SimpleCookie();
        simpleCookie.setName("rememberMe");
        simpleCookie.setMaxAge(60*60*24);
        return simpleCookie;
    }
    // 记住密码管理器
    @Bean
    public CookieRememberMeManager getCookieRememberMeManager(SimpleCookie simpleCookie){
        CookieRememberMeManager cookieRememberMeManager = new CookieRememberMeManager();
        cookieRememberMeManager.setCookie(simpleCookie);
        return cookieRememberMeManager;
    }
}
```
**最主要的还是这个shiro配置类，这个和spring mvc有所不同，其他的都差不多，不得不说springboot真香，很多配置都不用写了**

**在之前spring mvc整合shiro的基础上又添加了记住密码的新功能，以及模板语言换成了thymeleaf(spirngboot中用的较多，且spirngboot中不推荐使用jsp)**


# 自定义Realm
Realm还是当年的Realm，只是之前是通过xml配置的密码匹配器和缓存管理器现在要通过构造方法来进行依赖注入和配置
```java
public class CustomerRealm extends AuthorizingRealm {
    // 通过构造方法传入密码匹配器和Eh缓存管理器
    public CustomRealm(CacheManager cacheManager, CredentialsMatcher matcher) {
        super(cacheManager, matcher);
        this.setCachingEnabled(true);// 开启全局缓存
        this.setAuthenticationCachingEnabled(true); // 认证缓存
        this.setAuthorizationCachingEnabled(true);  //  授权缓存
    }

    // 授权
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principalCollection) {
        // 获取用户名
        String primaryPrincipal = (String) principalCollection.getPrimaryPrincipal();
        // 根据用户名获取角色信息(t_user、t_role、t_user_role三表联合查询)
        List<Entity> roles=null;
        try {
             roles = Db.use().query("select t_role.* from t_user " +
                     "join t_user_role on t_user.id=t_user_role.userid " +
                     "join t_role on t_user_role.roleid=t_role.id " +
                     "where t_user.username=?", primaryPrincipal);
        } catch (SQLException e) {
            e.printStackTrace();
        }
        SimpleAuthorizationInfo simpleAuthorizationInfo = new SimpleAuthorizationInfo();
        for (Entity role : roles) {
            simpleAuthorizationInfo.addRole(role.getStr("name")); // 把角色信息加入
            // 根据角色id 获取对应的权限信息
            List<Entity> perms = null;
            try {
                perms = Db.use().query("select t_perms.* from t_role join t_role_perms on t_role.id=t_role_perms.roleid join t_perms on t_role_perms.permsid=t_perms.id where t_role.id=?", role.getInt("id"));
            } catch (SQLException e) {
                e.printStackTrace();
            }
            for (Entity perm : perms) {
                simpleAuthorizationInfo.addStringPermission(perm.getStr("name")); // 把该角色对应的角色信息加入
            }

        }
        // 返回
        return simpleAuthorizationInfo;
    }

    // 登录认证时会执行该方法
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken authenticationToken) throws AuthenticationException {
        // 获取用户名
        String principal = (String) authenticationToken.getPrincipal();
        // 根据用户名查询用户信息
        Entity entity = null;
        try {
            // Entity相当于一个map对象，该处使用了hutool工具类操作数据库
            entity = Db.use().get(Entity.create("t_user").set("username", principal));
        } catch (SQLException e) {
            e.printStackTrace();
        }

        // 如果从数据库中找到了
        if (entity!=null) {
            return new SimpleAuthenticationInfo(
                    entity.getStr("username"),  // 用户名
                    entity.getStr("password"),  // 正确的密码
                    ByteSource.Util.bytes(entity.getStr("salt")),  // 用户注册时生成的随机盐
                    this.getName());    // 当前类名
        }
        return null;
    }
}
```
**这两个文件是有所不同的，其他的都差不多，不管你使用什么模板语言都是一样的。**

# 修改配置文件application.yml
```yml
spring:
  # 默认开启的，所以把它关闭
  thymeleaf:
    cache: false

# shiro默认的登录页面是login.jsp,需要修改
shiro:
  loginUrl: /login
```

# 创建数据库配置文件db.setting
该配置文件只适用于hutool-db模块，要使用mybatis的话就不需要了，直接在application.yml配置即可。
```properties
## db.setting文件

url = jdbc:mysql://localhost:3306/shiro?useSSL=false
user = root
pass = 123456

## 可选配置
# 是否在日志中显示执行的SQL
showSql = true
# 是否格式化显示的SQL
formatSql = false
# 是否显示SQL参数
showParams = true
# 打印SQL的日志等级，默认debug，可以是info、warn、error
sqlLevel = debug
```
# 静态页面
**注意：在使用thymeleaf模板时需要引入标签库，和与thymelead相对应的shiro标签库**
`<html lang="en" xmlns:th="http://www.thymeleaf.org" xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">`
## login.html
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org" xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>用户登录</h1>
<form action="login" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    记住密码：<input type="checkbox" name="rememb"><br>
    <input type="submit" value="登录">
</form>
</body>
</html>
```
## index.html
```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org" xmlns:shiro="http://www.pollix.at/thymeleaf/shiro">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1>系统主页v1.0</h1>
<a href="logout">退出登录</a>
<br>
用户名：<span shiro:principal=""></span>
<br>
<ul>
    <li><a href="#">用户管理</a></li>
    <div shiro:hasRole="admin">
        <li><a href="#">商品管理</a></li>
        <li><a href="#">订单管理</a></li>
        <li><a href="#">物流管理</a></li>
    </div>
</ul>
</body>
</html>
```
shiro标签功能和jsp中的都是一样的，只是写法稍微有一些不同，需要时html标签的属性中写