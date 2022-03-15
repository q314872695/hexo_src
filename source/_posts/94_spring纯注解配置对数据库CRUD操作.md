---
title: spring纯注解配置对数据库CRUD操作
tags:
  - java
  - spring
abbrlink: 7cd4b75b
date: 2020-06-08 18:04:44
---

> 该案例和上一个案例项目结构一摸一样，只是删除了spring配置文件，以及添加的两个配置类来代替xml配置文件
# 项目结构
![image.png](https://halo-1257208482.image.myqcloud.com/image_1591610963351.png!webp)
config包下为新添加的配置类。
# 相关配置
## 用到的新注解：
- `@Configuration`
	- 作用：指定当前类是一个配置类
	- 细节：当配置类作为`AnnotationConfigApplicationContext`对象创建的参数时，该注解可以不写。
 - `@ComponentScan`
	- 作用：用于通过注解指定spring在创建容器时要扫描的包，
	- 属性：
		- `value`：指定创建容器时要扫描的包。我们使用此注解就等同于在xml中配置了:`<context:component-scan base-package="com.itheima"></context:component-scan>`
 - `@Bean`
	- 作用：用于把当前方法的返回值作为bean对象存入spring的ioc容器中
	- 属性:
		- `name`:用于指定bean的id。当不写时，默认值是当前方法的名称
	- 细节：
		- 当我们使用注解配置方法时，如果方法有参数，spring框架会去容器中查找有没有可用的bean对象。
		- 查找的方式和`@Autowired`注解的作用是一样的
- `@Import`
	- 作用：用于导入其他的配置类
	- 属性：
		- `value`：用于指定其他配置类的字节码。当我们使用Import的注解之后，有Import注解的类就父配置类，而导入的都是子配置类,此时被导入的子配置类可以省略`@Configuration`注解
- `@PropertySource`
	- 作用：用于指定properties文件的位置
	- 属性：
		- `value`：指定文件的名称和路径。关键字：classpath，表示类路径下

## 配置类
**有关jdbc的子配置类**
```java
@Configuration
@PropertySource("classpath:jdbcConfig.properties")
public class JdbcConfig {
    @Value("${jdbc.driver}")
    private String driver;

    @Value("${jdbc.url}")
    private String url;

    @Value("${jdbc.username}")
    private String username;

    @Value("${jdbc.password}")
    private String password;
    /**
     * 创建jdbctemplate对象
     * @param dataSource
     * @return
     */
    @Bean("jdbcTemplate")
    public JdbcTemplate createJdbcTemplate(DataSource dataSource) {
        return new JdbcTemplate(dataSource);
    }

    /**
     * 配置数据源
     * @return
     */
    @Bean("dataSource")
    public DataSource createDataSource(){
        ComboPooledDataSource comboPooledDataSource = new ComboPooledDataSource();
        try {
            comboPooledDataSource.setDriverClass(driver);
//            comboPooledDataSource.setDriverClass("com.mysql.jdbc.Driver");
            comboPooledDataSource.setJdbcUrl(url);
//            comboPooledDataSource.setJdbcUrl("jdbc:mysql://localhost:3306/spring");
            comboPooledDataSource.setUser(username);
//            comboPooledDataSource.setUser("root");
            comboPooledDataSource.setPassword(password);
//            comboPooledDataSource.setPassword("123456");
        } catch (PropertyVetoException e) {
            e.printStackTrace();
        }
        return comboPooledDataSource;
    }
}
```
**主配置类**
```java
@Configuration
@ComponentScan("com.itheima")
public class SpringConfig {

}
```
# 测试
```java
public class AccountServiceTest {
    private ApplicationContext applicationContext;
    private IAccountService accountService;

    @Before
    public void init() {
        applicationContext = new AnnotationConfigApplicationContext(SpringConfig.class);
        accountService = applicationContext.getBean("accountService", IAccountService.class);
    }

    @Test
    public void test1() {
        List<Account> accounts = accountService.findAccountAll();
        for (Account account : accounts) {
            System.out.println(account);
        }
    }

    @Test
    public void test2() {
        Account account = accountService.findAccountById(2);
        System.out.println(account);

    }

    @Test
    public void test3() {
        Account account = new Account();
        account.setId(4);
        account.setName("ddd");
        account.setMoney(1500f);
        accountService.saveAccount(account);

    }

}
```
