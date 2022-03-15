---
title: spring mvc整合shiro
tags:
  - spring mvc
  - shiro
abbrlink: 2292b57d
date: 2020-06-22 21:57:51
---

> shiro整合spring mvc,配置比较麻烦，先搭建spring mvc环境，然后再整合，本文主要是介绍shiro至于使用mybatis访问数据库，请参考之前文章 
# 创建maven项目并修改pom.xml文件导入jar包
```xml
	<!--servlet-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>
        <!--jsp-->
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>2.3.1</version>
            <scope>provided</scope>
        </dependency>
        <!--jstl标签库-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <!--spring mvc-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.2.6.RELEASE</version>
        </dependency>
        <!--shiro整合spring以及shiro-core、shiro-web-->
        <dependency>
            <groupId>org.apache.shiro</groupId>
            <artifactId>shiro-spring</artifactId>
            <version>1.5.3</version>
        </dependency>
        <!--java工具类封装了各种方法简化操作(主要用来访问数据库)-->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.3.7</version>
        </dependency>
        <!--shiro日志-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.26</version>
        </dependency>
        <!--mysql驱动-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
        </dependency>
```
# 修改web.xml文件
```xml
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee
         http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">
    <!-- 配置Spring的监听器,tomcat启动时创建spring容器 -->
    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>
    <!-- 配置加载类路径的配置文件 -->
    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>classpath:ApplicationContext.xml</param-value>
    </context-param>

    <!-- SpringMVC的核心控制器 -->
    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 配置Servlet的初始化参数，读取springmvc的配置文件，创建spring容器 -->
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value>classpath:SpringMVC.xml</param-value>
        </init-param>
        <!-- 配置servlet启动时加载对象 -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/</url-pattern>
    </servlet-mapping>

    <!-- 配置过滤器，解决中文乱码的问题 -->
    <filter>
        <filter-name>characterEncodingFilter</filter-name>
        <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
        <!-- 指定字符集 -->
        <init-param>
            <param-name>encoding</param-name>
            <param-value>UTF-8</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>characterEncodingFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>

    <!--以上是spirng mvc环境搭建，以下是shiro环境搭建-->
    
    <!--shiro过滤器-->
    <filter>
        <filter-name>shiroFilter</filter-name>
        <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
        <init-param>
            <param-name>targetFilterLifecycle</param-name>
            <param-value>true</param-value>
        </init-param>
    </filter>
    <filter-mapping>
        <filter-name>shiroFilter</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```
# 在resources目录下创建ApplicationContext.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">
    <!--开启注解扫描，处理service和dao 而不处理controller-->
    <context:component-scan base-package="zone.lxy">
        <!--不处理@controller注解的类-->
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!--以下开始为spring整合shiro-->

    <!-- shiro过滤器bean，id要和web.xml中filter-name一致 -->
    <bean id="shiroFilter" class="org.apache.shiro.spring.web.ShiroFilterFactoryBean">
        <property name="securityManager" ref="securityManager"/>
        <property name="loginUrl" value="/login"/>
        <!--配置要拦截的url-->
        <property name="filterChainDefinitionMap">
            <map>
                <!--配置静态资源可以匿名访问-->
                <entry key="/css/**" value="anon"/>
                <entry key="/js/**" value="anon"/>
                <entry key="/images/**" value="anon"/>
                <entry key="/fonts/**" value="anon"/>

                <!--登录页面不过滤-->
                <entry key="/login" value="anon"/>
                <!--退出页面-->
                <entry key="/logout" value="logout"/>
                <!--剩下的所有资源都被拦截-->
                <entry key="/**" value="authc"/>
            </map>
        </property>
    </bean>
    <!--配置web安全管理器-->
    <bean id="securityManager" class="org.apache.shiro.web.mgt.DefaultWebSecurityManager">
        <!--注入自定义的realm-->
        <property name="realm" ref="myRealm"/>
    </bean>
    <bean id="lifecycleBeanPostProcessor" class="org.apache.shiro.spring.LifecycleBeanPostProcessor"/>

    <!--自定义realm-->
    <bean id="myRealm" class="zone.lxy.realm.CustomerRealm">
        <!--注入密码匹配器-->
        <property name="credentialsMatcher" ref="credentialsMatcher"/>
    </bean>
    <!--配置密码匹配器-->
    <bean id="credentialsMatcher" class="org.apache.shiro.authc.credential.HashedCredentialsMatcher">
        <property name="hashAlgorithmName" value="md5"/>
        <property name="hashIterations" value="1024"/>
    </bean>
</beans>
```
shiro在运行是会创建一些默认的过滤器用于拦截url，常用过滤器如下:
- `anon`：可以匿名访问
- `authc`：登录后才能访问
- `logout`：实现退出功能
- `roles`：需要指定角色才能访问
- `perms`：需要指定权限才能访问
- `user`：需要已登录或“记住我”的用户才能访问

# 在resources目录下创建SpringMVC.xml
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:mvc="http://www.springframework.org/schema/mvc"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/mvc
        https://www.springframework.org/schema/mvc/spring-mvc.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
    <!--开启注解扫描-->
    <context:component-scan base-package="zone.lxy.controller">
        <!--只扫描controller注解-->
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>
    <!-- 配置jsp视图解析器 -->
    <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
        <property name="prefix" value="/WEB-INF/page/"/>
        <property name="suffix" value=".jsp"/>
    </bean>
    <!-- 设置静态资源不过滤-->
    <mvc:default-servlet-handler/>
    <!--配置spring开启注解mvc的支持-->
    <mvc:annotation-driven/>

    <!--开启shiro注解，只能放在spring mvc配置文件中，放在spring配置文件不会生效 -->
    <bean class="org.springframework.aop.framework.autoproxy.DefaultAdvisorAutoProxyCreator"
          depends-on="lifecycleBeanPostProcessor"/>
    <bean class="org.apache.shiro.spring.security.interceptor.AuthorizationAttributeSourceAdvisor">
        <property name="securityManager" ref="securityManager"/>
    </bean>


</beans>
```
# 在resources目录下创建db.setting配置数据库信息(临时)
以后应该使用的都是mybatis，暂时使用简单的工具类操作数据库，减少学习成本
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
# 在resources目录下创建log4j.properties日志配置文件
测试阶段所有信息均打印在控制台
```properties
# Set root category priority to INFO and its only appender to CONSOLE.
#log4j.rootCategory=INFO, CONSOLE            debug   info   warn error fatal
log4j.rootCategory=debug, CONSOLE

# Set the enterprise logger category to FATAL and its only appender to CONSOLE.
log4j.logger.org.apache.axis.enterprise=FATAL, CONSOLE

# CONSOLE is set to be a ConsoleAppender using a PatternLayout.
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} %-6r [%t] %-5p %30.30c %x - %m%n
```
# 编写控制器类
```java
@Controller
public class UserController {
    private final static Log log= LogFactory.get();

    // 跳转到登录页面
    @RequestMapping(value = "/login",method = RequestMethod.GET)
    public String login(){
        return "login";
    }
    // 处理登录请求
    @RequestMapping(value = "/login",method = RequestMethod.POST)
    public String login(String username, String password, HttpServletResponse response) throws IOException {
        response.setContentType("text/html;charset=UTF-8");
        // 获取主体对象
        Subject subject = SecurityUtils.getSubject();
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);
        try {
            // 登录验证
            subject.login(token);
        } catch (UnknownAccountException e) {
            // 给页面返回提示信息
            response.getWriter().write("用户名不存在");
            log.info("用户名不存在");
            return null;
        }catch (IncorrectCredentialsException e) {
            // 给页面返回提示信息
            response.getWriter().write("密码错误");
            log.info("密码错误");
            return null;
        }
        // 登录成功则重定向到首页
        return "redirect:/";
    }

    // 进入首页
    @RequestMapping("/")
    @RequiresRoles("admin") // 只有管理员才能进入首页，其他身份会报错
    public String index(Model model){
        log.info("进入首页");
        // 测试jstl标签
        ArrayList<String> list = ListUtil.toList("张三", "李四", "王五", "赵六");
        model.addAttribute("list", list);
        return "index";
    }
}
```
**整合spring mvc中可使用注解来控制权限，常用注解：**
- `@RequiresRoles`：只有对应角色才能访问，其他角色访问就把报错
- `@RequiresPermissions`：只有拥有该权限的用户才能访问，否则报错

# 自定义Realm类
```java
public class CustomerRealm extends AuthorizingRealm {
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
# 编写index.jsp页面
```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<%@taglib prefix="shiro" uri="http://shiro.apache.org/tags" %>
<html>
<head>
    <title>首页</title>
</head>
<body>
<h1>登录成功</h1>
你好<shiro:principal/>，身份：
<shiro:hasRole name="admin">管理员 </shiro:hasRole>
<shiro:hasRole name="teacher">教师 </shiro:hasRole>
<shiro:hasRole name="student">学生 </shiro:hasRole>
<br>
<a href="logout">退出</a><br>
</body>
</html>
```
**关于shiro的jsp新标签（常用）：**
- `<shiro:guest>`：用户未登录时标签中的内容就会显示
- `<shiro:principal/>`：输出当前用户的用户名
- `<shiro:hasRole name="admin">`：如果该用户的角色是admin，则标签中的内容会显示
- `<shiro:hasPermission name="user:create">`：如果该用户具有该权限就显示标签中的内容

更多标签信息点击[这里](http://shiro.apache.org/web.html#Web-taglibrary)


# 编写login.jsp页面
```jsp
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
<head>
    <title>登录</title>
</head>
<body>
<form action="login" method="post">
    用户名：<input type="text" name="username"><br>
    密码：<input type="password" name="password"><br>
    <input type="submit" value="登录">
</form>
</body>
</html>
```
# 性能优化——配置Eh缓存管理器
登录成功进入首页后，每次刷新页面都会从数据库获取角色（权限）信息，会对数据库造成很大压力，而且角色信息一般也很少变动，故应该使用缓存。**这里也可以使用redis做缓存相关信息可以百度**
## 在之前基础上导入新jar包
```xml
<!--Eh缓存管理器-->
<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-ehcache</artifactId>
	<version>1.5.3</version>
</dependency>
```
## 修改ApplicationContext.xml文件
```xml
<!--自定义realm-->
<bean id="myRealm" class="zone.lxy.realm.CustomerRealm">
        <!--注入密码匹配器-->
        <property name="credentialsMatcher" ref="credentialsMatcher"/>
        <!--设置缓存管理器-->
        <property name="cacheManager" ref="ehCacheManager"/>
        <!--开启全局缓存-->
        <property name="cachingEnabled" value="true"/>
        <!--开启认证缓存-->
        <property name="authenticationCachingEnabled" value="true"/>
        <!--开启授权缓存-->
        <property name="authorizationCachingEnabled" value="true"/>
</bean>
<!--把Eh缓存管理器加入spirng容器-->
<bean id="ehCacheManager" class="org.apache.shiro.cache.ehcache.EhCacheManager"/>
```