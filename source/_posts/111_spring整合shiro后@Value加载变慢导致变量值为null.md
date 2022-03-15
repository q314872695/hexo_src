---
title: spring整合shiro后@Value加载变慢导致变量值为null
tags:
  - spring
  - shiro
abbrlink: ebdf593d
date: 2021-01-12 21:51:55
---

具体情况如下：
`Caused by: java.lang.ClassNotFoundException: ${jdbc.driverClass}`
而通过Controller中调用可以正常打印，由此可知是@Value加载的太慢了。

经过百度了好久，终于找到原因，由于Spring 和Shiro整合时，生命周期有某种冲突，需在spirng配置类中修改以下代码即可。**（程序使用纯java配置无xml）**
```java
 // 不加static,@Value加载properties文件太慢导致为null
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }
    // shiro 生命周期函数，不加static,@Value加载properties文件太慢导致为null
    @Bean
    public static LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }
```
**Spring完整配置类如下：**
```java
@Configuration
@ComponentScan(value = "zone.lxy", excludeFilters = @ComponentScan.Filter(Controller.class)) //除了controller注解其他都扫
@PropertySource("classpath:jdbc.properties")
@EnableTransactionManagement // 开启事务控制
@MapperScan("zone.lxy.dao")
public class SpringConfig {
    @Value("${jdbc.driverClass}")
    private String driverClass;

    @Value("${jdbc.jdbcUrl}")
    private String jdbcUrl;

    @Value("${jdbc.user}")
    private String user;

    @Value("${jdbc.password}")
    private String password;


    // 数据源
    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        try {
            dataSource.setDriverClassName(driverClass);
//            dataSource.setDriverClassName("com.mysql.jdbc.Driver");
            dataSource.setUrl(jdbcUrl);
//            dataSource.setUrl("jdbc:mysql://localhost:3306/emp?characterEncoding=utf8&useSSL=false");
            dataSource.setUsername(user);
//            dataSource.setUsername("root");
            dataSource.setPassword(password);
//            dataSource.setPassword("123456");
        } catch (Exception e) {
            e.printStackTrace();
        }
        return dataSource;
    }

    // 事务管理器
    @Bean
    public PlatformTransactionManager transactionManager(DataSource dataSource) {
        return new DataSourceTransactionManager(dataSource);
    }

    // 配置mybatis-plus的bean工厂
    @Bean
    public MybatisSqlSessionFactoryBean sqlSessionFactoryBean(DataSource dataSource) {
        MybatisSqlSessionFactoryBean sqlSessionFactoryBean = new MybatisSqlSessionFactoryBean();
        sqlSessionFactoryBean.setDataSource(dataSource);
        // mybatis-plus分页插件
        PaginationInterceptor paginationInterceptor = new PaginationInterceptor();
        paginationInterceptor.setCountSqlParser(new JsqlParserCountOptimize(true));
        sqlSessionFactoryBean.setPlugins(paginationInterceptor);
        return sqlSessionFactoryBean;
    }

    // shiro filter工厂
    @Bean("shiroFilter")
    public ShiroFilterFactoryBean shiroFilterFactoryBean(SecurityManager securityManager) {
        ShiroFilterFactoryBean shiroFilterFactoryBean = new ShiroFilterFactoryBean();
        shiroFilterFactoryBean.setSecurityManager(securityManager);
        Map<String, String> map = new LinkedHashMap<>();
        map.put("/css/**", "anon");
        map.put("/js/**", "anon");
        map.put("/img/**", "anon");
        map.put("/login", "anon");
        map.put("/register", "anon");
        map.put("/code", "anon");
        map.put("/logout", "logout");
        map.put("/**", "authc");
        shiroFilterFactoryBean.setLoginUrl("/login");
        shiroFilterFactoryBean.setFilterChainDefinitionMap(map);
        return shiroFilterFactoryBean;
    }

    // shiro web安全管理器
    @Bean
    public DefaultWebSecurityManager securityManager(CustomerRealm customerRealm) {
        DefaultWebSecurityManager defaultWebSecurityManager = new DefaultWebSecurityManager();
        defaultWebSecurityManager.setRealm(customerRealm);
        return defaultWebSecurityManager;
    }

    // 不加static,@Value加载properties文件太慢导致为null
    @Bean
    public static PropertySourcesPlaceholderConfigurer propertySourcesPlaceholderConfigurer() {
        return new PropertySourcesPlaceholderConfigurer();
    }
    // shiro 生命周期函数，不加static,@Value加载properties文件太慢导致为null
    @Bean
    public static LifecycleBeanPostProcessor lifecycleBeanPostProcessor() {
        return new LifecycleBeanPostProcessor();
    }

    // 开启shiro注解
    @Bean
    @DependsOn("lifecycleBeanPostProcessor")
    public DefaultAdvisorAutoProxyCreator defaultAdvisorAutoProxyCreator(){
        return new DefaultAdvisorAutoProxyCreator();
    }
    @Bean
    public AuthorizationAttributeSourceAdvisor authorizationAttributeSourceAdvisor(SecurityManager securityManager) {
        AuthorizationAttributeSourceAdvisor advisor = new AuthorizationAttributeSourceAdvisor();
        advisor.setSecurityManager(securityManager);
        return advisor;
    }
}
```