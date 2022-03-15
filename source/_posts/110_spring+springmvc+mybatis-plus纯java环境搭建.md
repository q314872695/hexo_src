---
title: spring+springmvc+mybatis-plus纯java环境搭建
tags:
  - spring
  - spring mvc
  - mybatis-plus
abbrlink: 9b96df1
date: 2020-06-26 16:32:55
---

# 修改pom.xml
```xml
<dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>3.1.0</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>2.3.1</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.2.7.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.2.7.RELEASE</version>
        </dependency>
        <!--整合mybatis-plus-->
        <dependency>
            <groupId>com.baomidou</groupId>
            <artifactId>mybatis-plus</artifactId>
            <version>3.3.2</version>
        </dependency>

        <!--解析事务控制中的切入点表达式-->
        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.5</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.49</version>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.12</version>
            <scope>provided</scope>
        </dependency>
        <!--使用log4j日志-->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.30</version>
        </dependency>
        <!--解析json-->
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.11.0</version>
        </dependency>
```
# java配置类相关
由于没有了xml配置文件，所有配置类都以java的形式表现

**MyWebAppInitializer.java**
```java
// 等价于web.xml文件
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    // spring配置类
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class[]{SpringConfig.class};
    }
    // spring mvc配置类
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }

    // spring mvc拦截路径
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }

    // 添加过滤器
    @Override
    protected Filter[] getServletFilters() {
        // 防止中文乱码的过滤器
        CharacterEncodingFilter characterEncodingFilter = new CharacterEncodingFilter();
        characterEncodingFilter.setEncoding("UTF-8");
        characterEncodingFilter.setForceEncoding(true);
        return new Filter[]{characterEncodingFilter};
    }

    // 文件上传配置
    @Override
    protected void customizeRegistration(ServletRegistration.Dynamic registration) {

        registration.setMultipartConfig(new MultipartConfigElement(""));
    }
}
```
**spring的配置类**
```java
@Configuration
@ComponentScan(value = "zone.lxy", excludeFilters = @ComponentScan.Filter(Controller.class)) //除了controller注解其他都扫
@PropertySource("classpath:jdbc.properties") //读取数据库配置
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
            dataSource.setUrl(jdbcUrl);
            dataSource.setUsername(user);
            dataSource.setPassword(password);
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
}
```
**spring mvc配置类**
```java
@Configuration
@EnableWebMvc
@ComponentScan(value = "zone.lxy",includeFilters = @ComponentScan.Filter(Controller.class)) //只扫描controller注解
public class WebConfig implements WebMvcConfigurer {
    // jsp视图解析器
    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {
        registry.jsp("/WEB-INF/templates/", ".jsp");
    }

    // 设置默认Servlet，用来响应静态资源
    @Override
    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
    }
}
```
# 数据库连接和日志配置
**jdbc.properties**
```properties
jdbc.driverClass=com.mysql.jdbc.Driver
jdbc.jdbcUrl=jdbc:mysql://localhost:3306/mybatis?characterEncoding=utf8&useSSL=false
jdbc.user=root
jdbc.password=123456
```
**log4j.properties**
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
