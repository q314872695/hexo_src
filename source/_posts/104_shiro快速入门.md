---
title: shiro快速入门
tags: shiro
abbrlink: 368a6765
date: 2020-06-22 21:57:05
---

> Apache Shiro从一开始就设计为支持任何应用程序-从最小的命令行应用程序到最大的群集Web应用程序。本案例从java程序开始
# 简易版（读取配置文件的用户名密码）
## 需求
- 在控制台输入用户名和密码实现简单登录
## 创建maven项目并导入依赖
```xml
<!--shiro核心jar包-->
<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-core</artifactId>
	<version>1.5.3</version>
</dependency>
<!--shiro日志-->
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-log4j12</artifactId>
	<version>1.7.26</version>
</dependency>
```
## 创建shiro.ini配置文件
在resources目录下创建
```ini
[users]
# username = password, role1, role2, ..., roleN
# 用户名=密码,角色(自定义的)
zhangsan=123,admin
lisi=456,student
wangwu=789,teacher
```
- 用户名是zhangsan，密码是123，角色是管理员，其他类似。

## 创建log4j日志配置文件(可选)
```properties
# Set root category priority to INFO and its only appender to CONSOLE.
#log4j.rootCategory=INFO, CONSOLE            debug   info   warn error fatal
log4j.rootCategory=info, CONSOLE

# Set the enterprise logger category to FATAL and its only appender to CONSOLE.
log4j.logger.org.apache.axis.enterprise=FATAL, CONSOLE

# CONSOLE is set to be a ConsoleAppender using a PatternLayout.
log4j.appender.CONSOLE=org.apache.log4j.ConsoleAppender
log4j.appender.CONSOLE.layout=org.apache.log4j.PatternLayout
log4j.appender.CONSOLE.layout.ConversionPattern=%d{ISO8601} %-6r [%t] %-5p %30.30c %x - %m%n
```

## 测试类 
```java
public class Demo1 {
    private static final Log log = LogFactory.get();

    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        // 创建安全管理器对象
        DefaultSecurityManager securityManager = new DefaultSecurityManager();
        // 给安全管理器设置realm，并读取配置文件
        securityManager.setRealm(new IniRealm("classpath:shiro.ini"));
        // 给全局安全工具类设置安全管理器
        SecurityUtils.setSecurityManager(securityManager);
        // 创建对象subject主体，可以在程序中的任意位置调用
        Subject subject = SecurityUtils.getSubject();

        // 从控制台获取用户名和密码
        System.out.print("用户名：");
        String username = scanner.next();
        System.out.print("密码：");
        String password = scanner.next();
        // 创建令牌
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);
        try {
            // 尝试登录，成功则继续执行，失败就报异常
            subject.login(token);

            // 登录成功则为true
            System.out.println("认证状态："+subject.isAuthenticated());

            // 判断当前用户的角色
            if (subject.hasRole("admin")) {
                System.out.println("用户身份：admin");
            } else if (subject.hasRole("teacher")) {
                System.out.println("用户身份：teacher");
            } else if (subject.hasRole("student")) {
                System.out.println("用户身份：student");
            }
        } catch (UnknownAccountException e) {
            System.out.println("用户名错误");
        } catch (IncorrectCredentialsException e) {
            System.out.println("密码错误");
        }
    }
}
// 运行结果：
用户名：zhangsan
密码：123
2020-06-23 22:12:37,538 0      [main] INFO  stractValidatingSessionManager  - Enabling session validation scheduler...
认证状态：true
用户身份：admin
```
# 升级版(从数据库中获取用户信息)
## 导入依赖
在上一个案例的基础上添加新的坐标
```xml
<!--shiro核心jar包-->
<dependency>
	<groupId>org.apache.shiro</groupId>
	<artifactId>shiro-core</artifactId>
	<version>1.5.3</version>
</dependency>
<!--shiro日志-->
	<groupId>org.slf4j</groupId>
	<artifactId>slf4j-log4j12</artifactId>
	<version>1.7.26</version>
</dependency>
<!--java工具类封装了各种方法简化操作-->
<dependency>
	<groupId>cn.hutool</groupId>
	<artifactId>hutool-all</artifactId>
	<version>5.3.7</version>
</dependency>
<!--mysql驱动-->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<version>5.1.49</version>
</dependency>
```
## 在resources目录下创建db.setting数据库配置文件
```properties
## db.setting文件

url = jdbc:mysql://localhost:3306/shiro
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
## 自定义Realm
之前案例中用户信息是通过`IniRealm`加载配置文件中的用户信息，本案例需要从数据库中获取，所以要自定义一个Realm需要继承抽象类`AuthorizingRealm`
```java
public class MyRealm extends AuthorizingRealm {
    // 授权，获取（判断）用户角色信息时执行该方法,暂不实现
    protected AuthorizationInfo doGetAuthorizationInfo(PrincipalCollection principals) {
        return null;
    }

    // 登录认证时会执行该方法
    protected AuthenticationInfo doGetAuthenticationInfo(AuthenticationToken token) {
        // 获取用户名
        String principal = (String) token.getPrincipal();

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
                    this.getName());    // 当前类名
        }
        return null;
    }
}
```
## 修改测试类
把Realm设置成自定义的Realm
- 修改前`securityManager.setRealm(new IniRealm("classpath:shiro.ini"));`
- 修改后`securityManager.setRealm(new MyRealm());`

其他不变
**该案例简化了对角色信息的控制（因为还要创建一张表麻烦）**

# 再升级版（对明文密码加密+角色(权限)控制）
在生产环境中，数据库的密码都是密文保存的，该案例实现了使用shiro对用户密码加密的登录
## 数据表关系图
![image.png](https://halo-1257208482.image.myqcloud.com/202204051756880.png!webp)
- t_user:用户表
- t_role:角色表
- t_perms:权限表
- t_user_role:用户角色表，用户和角色是多对多关系
- t_role_perms:角色权限表，角色和权限是多对多关系
## 关于加密问题
本案例使用的是md5+salt(随机盐)的加密方式，只有md5加密，加密程度不够高，例如你要加密123456，加密后是e10adc3949ba59abbe56e057f20f883e,在百度中的在md5加解密网站中很容易被穷举出来。

md5+salt后，在123456的后面加上几个随机的字符，例如123456#&*%￥,这样再去加密后就很难被穷举出来了，也就增加的安全性。

而且md5还可以多次加密，即再已经加密的密文上在进行加密，就更安全了。

## 修改自定义的Realm
```java
// 把数据的来源转为数据库实现
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

**一般来说权限控制分为：**
- 基于角色的权限控制：判断是你哪种角色，然后执行相应操作
- 基于资源的权限控制：判断你拥有哪种资源，然后执行相应操作
	- 格式：`资源标识符:操作:资源类型`
	- 例如：
		- `*:*:*`：表示拥有全部资源（权限），一般是超级管理员
		- `teacher:add:*`：表示拥有教师的添加权限
		- `student:*:1`：表示只能对1号学生进行操作（CRUD）

本案例角色和资源都可以控制。
## 测试类
```java
public class Demo3 {
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        // 创建安全管理器对象
        DefaultSecurityManager securityManager = new DefaultSecurityManager();

        CustomerRealm customerRealm = new CustomerRealm();

        // 设置密码匹配器
        HashedCredentialsMatcher hashedCredentialsMatcher = new HashedCredentialsMatcher();
        hashedCredentialsMatcher.setHashAlgorithmName("md5");   //加密方式
        hashedCredentialsMatcher.setHashIterations(1024); // 散列次数
        // 给自定义的realm设置密码匹配器
        customerRealm.setCredentialsMatcher(hashedCredentialsMatcher);
        // 给安全管理器设置realm
        securityManager.setRealm(customerRealm);
        // 给全局安全工具类设置安全管理器
        SecurityUtils.setSecurityManager(securityManager);
        // 创建对象subject主体
        Subject subject = SecurityUtils.getSubject();

        // 从控制台获取用户名和密码
        System.out.print("用户名：");
        String username = scanner.next();
        System.out.print("密码：");
        String password = scanner.next();
        // 创建令牌
        UsernamePasswordToken token = new UsernamePasswordToken(username, password);


        try {
            subject.login(token);
            Console.log("认证状态：" + subject.isAuthenticated());

        } catch (UnknownAccountException e) {
            Console.log("用户名不存在");
        } catch (IncorrectCredentialsException e) {
            Console.log("密码错误");
        }
        // 判断当前用户的角色
        if (subject.hasRole("admin")) {
            Console.log("用户身份：admin");
        } else if (subject.hasRole("teacher")) {
            Console.log("用户身份：teacher");
        } else if (subject.hasRole("student")) {
            Console.log("用户身份：student");
        }
        // 基于权限,格式：资源标识符:操作:资源类型
        Console.log("[admin:*:*]: {}", subject.isPermitted("admin:*:*"));
        Console.log("[teacher:*:*]: {}", subject.isPermitted("teacher:*:*"));
        Console.log("[student:*:*]: {}", subject.isPermitted("student:*:*"));
    }
}

// 运行结果：
用户名：xiaocheng
密码：123
认证状态：true
用户身份：admin
[admin:*:*]: true
[teacher:*:*]: true
[student:*:*]: true
```
## 用户注册Demo(md5+salt加密)
```java
public class Demo4 {
    /**
     * 生成salt的静态方法
     * @param n
     * @return
     */
    public static String getSalt(int n){
        char[] chars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz1234567890!@#$%^&*()".toCharArray();
        StringBuilder builder = new StringBuilder();
        for (int i = 0; i < n; i++) {
            char c = chars[new Random().nextInt(chars.length)];
            builder.append(c);
        }
        return builder.toString();
    }
    public static void main(String[] args) throws SQLException {
        // 模拟用户注册，并对密码进行加密

        Scanner scanner = new Scanner(System.in);
        System.out.println("-----注册新用户-----");
        System.out.print("新用户名：");
        String username = scanner.next();
        System.out.print("新密码：");
        String password = scanner.next();

        StaticLog.debug("密码加密前：{}",password);
        // 获取生成的随机盐
        String salt = getSalt(8);
        StaticLog.debug("生成的随机盐：{}",salt);
        // 对密码进行md5+salt加密,哈希次数要和密码匹配器中的哈希次数一致，否则无法登录成功
        Md5Hash hash = new Md5Hash(password,salt,1024);
        // 生成的密文密码
        String password1 = hash.toHex();
        StaticLog.debug("密码加密后：{}",password1);
        // 存入数据库
        Db.use().tx(db->{   // 进行事务控制，要么都成功，要么都失败
            // 插入后返回主键
            Long id = db.insertForGeneratedKey(Entity.create("t_user").set("username", username).set("password", password1).set("salt", salt));
            // 默认给该用户赋予学生角色
            db.insert(Entity.create("t_user_role").set("userid",id).set("roleid",3));
        });
        StaticLog.debug("注册成功！");
    }
}
// 运行结果：
-----注册新用户-----
新用户名：zhangsan
新密码：123
2020-06-24 09:39:31,112 0      [main] DEBUG       cn.hutool.log.LogFactory  - Use [Slf4j] Logger As Default.
2020-06-24 09:39:31,115 3      [main] DEBUG                 zone.lxy.Demo4  - 密码加密前：123
2020-06-24 09:39:31,115 3      [main] DEBUG                 zone.lxy.Demo4  - 生成的随机盐：AO2U5CrO
2020-06-24 09:39:31,135 23     [main] DEBUG                 zone.lxy.Demo4  - 密码加密后：5b8d7a430747d63154c543223fdf5116
2020-06-24 09:39:31,712 600    [main] DEBUG                 zone.lxy.Demo4  - 注册成功！
```
