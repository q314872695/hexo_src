---
title: 一文读懂Hibernate Validator 数据校验框架
date: 2021-01-22 15:55:51
tags: hibernate-validator 
---

数据校验是一项常见的任务，在没有引入框架之前，我们通常的做法都是一堆的`if` 、 `else`等等，在代码中非常不美观，只用Hibernate Validator可以简化代码。
# 环境搭建
## javaSE版本
### 1.引入pom.xml文件
```xml
<!--hibernate-validator-->
<dependency>
	<groupId>org.hibernate.validator</groupId>
	<artifactId>hibernate-validator</artifactId>
	<version>6.1.6.Final</version>
</dependency>
<!--hibernate-validator el表达式依赖，版本必须对应否则会报错-->
<dependency>
	<groupId>org.glassfish</groupId>
	<artifactId>jakarta.el</artifactId>
	<version>3.0.3</version>
</dependency>
<!--hutool工具包，本案例中提供aop代理工具-->
<dependency>
	<groupId>cn.hutool</groupId>
	<artifactId>hutool-all</artifactId>
 	<version>5.5.6</version>
</dependency>
```
### 2.编写ValidatorUtil工具类
```java
public class ValidationUtil {
    private static Validator validator;
    private static Validator faliFastValidator;
    private static ExecutableValidator executableValidator;

    static {
        validator = Validation.buildDefaultValidatorFactory().getValidator();
        // 快速失败
        faliFastValidator = Validation.byProvider(HibernateValidator.class).configure().failFast(true).buildValidatorFactory().getValidator();
        // 验证非bean入参
        executableValidator = validator.forExecutables();
    }

// 方法约束校验
    public static void validateParameters(Object var1, Method var2, Object[] var3, Class<?>... var4) {
        Set<ConstraintViolation<Object>> constraintViolations = executableValidator.validateParameters(var1, var2, var3, var4);
        constraintViolations.forEach(i -> {
	// 常见是使用异常来传递错误信息
            throw new RuntimeException(i.getMessage());
        });
    }
// bean约束校验
    public static void valid(Object o) {
        Set<ConstraintViolation<Object>> validate = validator.validate(o);
        validate.stream().forEach(v -> {
            System.out.println(v.getMessage());
//            throw new RuntimeException(v.getMessage());
        });
    }
    /*
    * 与valid方法的区别，只要有一个不成立就停止循环;valid方法会打印所有不满足的信息
    * */
    public static void fastValid(Object o) {
        Set<ConstraintViolation<Object>> validate = faliFastValidator.validate(o);
        validate.stream().forEach(v -> {
            System.out.println(v.getMessage());
//            throw new RuntimeException(v.getMessage());
        });
    }
}
```
### 3.声明和验证bean约束
**编写bean类UserInfo.java**
```java
public class UserInfo {
    @NotBlank(message = "姓名不能为空")
    private String name;
    @Min(value = 18, message = "未成年人禁止使用")
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```
**编写测试类Main.java**
```java
public class Main {
    public static void main(String[] args) {
        UserInfo userInfo = new UserInfo();
        userInfo.setAge(12);
        userInfo.setName("");
// 没有if语句，进行数据校验
        ValidationUtil.valid(userInfo);
//        ValidationUtil.fastValid(userInfo);

// 运行结果：
//
//一月 22, 2021 4:37:06 下午 org.hibernate.validator.internal.util.Version <clinit>
//INFO: HV000001: Hibernate Validator 6.1.6.Final
//        姓名不能为空
//        未成年人禁止使用
}
```
### 4.声明和验证方法约束
**编写service接口和类**(为什么要使用接口呢？因为要使用动态代理)
```java
public interface IUserInfoService {
    // 校验方法参数
    void add(@NotBlank(message = "用户名不能为空") String username);
    // 校验bean约束
    void add(@Valid UserInfo userInfo);
}
```
**@Valid** ：注解于参数为bean的情况
```java
public class UserInfoService implements IUserInfoService {
    public void add(String username) {
        System.out.println("添加成功["+username+"]");
    }

    @Override
    public void add(UserInfo userInfo) {
        System.out.println(userInfo);
    }
}
```
**编写代理类**
```java
public class ValidatorAspect extends SimpleAspect {
    // 动态代理，在调用被代理的方法前调用
    @Override
    public boolean before(Object target, Method method, Object[] args) {
        ValidationUtil.validateParameters(target, method, args);
        return true;
    }
}
```
**编写测试类Main.java**
```java
public class Main {
    public static void main(String[] args) {
	UserInfo userInfo = new UserInfo();
        userInfo.setAge(12);
        userInfo.setName("");
        // 通过动态代理创建一个对象
        IUserInfoService proxy = ProxyUtil.proxy(new UserInfoService(), ValidatorAspect.class);
        // 校验方法约束
	proxy.add("");
	// 校验bean约束
	proxy.add(userInfo);
    }
}
```
**本案例只是对hibernate-validator方法约束的入门使用，在实际使用中都是通过spring aop等工具来完成，spring已经提供的提供了和hibernate-validator的整合，在后面的案例会有**

## springMVC版本（纯java配置）
### 1.引入jar包
```xml
<!--servlet与jsp-->
        <dependency>
            <groupId>javax</groupId>
            <artifactId>javaee-api</artifactId>
            <version>7.0</version>
        </dependency>
        <!--springmvc相关-->
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-webmvc</artifactId>
            <version>5.3.2</version>
        </dependency>
        <dependency>
            <groupId>com.fasterxml.jackson.core</groupId>
            <artifactId>jackson-databind</artifactId>
            <version>2.12.0</version>
        </dependency>
        <!--hibernate-validator 相关，版本必须和spirngmvc匹配，否则报错-->
        <dependency>
            <groupId>org.hibernate.validator</groupId>
            <artifactId>hibernate-validator</artifactId>
            <version>6.1.6.Final</version>
        </dependency>

        <dependency>
            <groupId>org.glassfish</groupId>
            <artifactId>jakarta.el</artifactId>
            <version>3.0.3</version>
        </dependency>
```
### 2.springmvc相关配置
```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {
    @Override
    protected Class<?>[] getRootConfigClasses() {
        return null;
    }

    // 加载springmvc配置类
    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class[]{WebConfig.class};
    }
    @Override
    protected String[] getServletMappings() {
        return new String[]{"/"};
    }
}
```
```java
@Configuration
@EnableWebMvc
@ComponentScan(value = "zone.lxy",includeFilters = @ComponentScan.Filter(classes = {Controller.class}))
public class WebConfig implements WebMvcConfigurer {
    /*
    * 配置hibernate-validator，默认已经配置
    * */
//    @Override
//    public Validator getValidator() {
//        LocalValidatorFactoryBean localValidatorFactoryBean = new LocalValidatorFactoryBean();
//        localValidatorFactoryBean.setProviderClass(HibernateValidator.class);
//        return localValidatorFactoryBean;
//    }
    /*
    * 配置方法级别的数据校验，（非bean入参）
    * */
    @Bean
    public MethodValidationPostProcessor methodValidationPostProcessor() {
        return new MethodValidationPostProcessor();
    }
}
```
### 3.在controller中使用
```java
// 使用方法验证时，需要的被校验的类上添加@Validated注解
@Validated
@RestController
public class IndexController {
    
    
    /*
    * 方法约束校验
    * */
    @GetMapping("/")
    public Object index(@NotEmpty String s) {
        return s;
    }

    /*
    * bean约束校验
    * */
    @RequestMapping("/user")
    public User info(@Valid User user) {
        return user;
    }
}
```
**注：** 此时不满足条件会报异常，还需要配置一个全局异常拦截器就好了

## springboot版本
- 在springboot 2.3以下时已经默认给你整合好了，直接用就即可。
- 在使用2.3及以上版本时，需要添加额外jar包

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-validation</artifactId>
</dependency>
```


# 常见内置约束
Hibernate Validator包含一组基本的常用约束。[更多](https://docs.jboss.org/hibernate/validator/6.2/reference/en-US/html_single/#validator-defineconstraints-spec)

| 名称 | 说明 | 支持的数据类型 |
| ------- | ------- | ------- |
| `@AssertFalse` | 检查被注解的元素是否为false | Boolean， boolean |
|`@DecimalMax(value=, inclusive=)`|inclusive= false时，检查被注解的值是否小于指定的最大值。否则，该值是否小于或等于指定的最大值。|BigDecimal，BigInteger，byte，short，int，long和原始类型的相应的包装; CharSequence的任何子类型，Number和的javax.money.MonetaryAmoun任何子类型|
|`@DecimalMin(value=, inclusive=)`|inclusive= false时，检查被注解的值是否大于指定的最小值。否则，该值是否大于或等于指定的最小值。|同上|
|`@Digits(integer=, fraction=)`|检查被注解的值是否是最多由integer数字和fraction小数位组成的数字|同上|
|`@Max(value=)`|检查被注解的值是否小于或等于指定的最大值|同上|
|`@Min(value=)`|检查被注解的值是否大于或等于指定的最小值|同上|
|`@Range(min=, max=)`|检查带注释的值是否介于（包括）指定的最小值和最大值之间|同上|
|`@Negative`|检查元素是否严格为负。零值被视为无效。|同上|
|`@Positive`|检查元素是否严格为正。零值被视为无效。|同上|
|`@NegativeOrZero`|检查元素是否为负数或零。|同上|
|`@PositiveOrZero`|检查元素是否为正数或零。|同上|
|`@Future`|检查被注解的日期是否是将来的日期|java.util.Date，java.util.Calendar，java.time.Instant，java.time.LocalDate，java.time.LocalDateTime，java.time.LocalTime，java.time.MonthDay，java.time.OffsetDateTime，java.time.OffsetTime，java.time.Year，java.time.YearMonth，java.time.ZonedDateTime，java.time.chrono.HijrahDate，java.time.chrono.JapaneseDate，java.time.chrono.MinguoDate，java.time.chrono.ThaiBuddhistDate; |
|`@FutureOrPresent`|检查被注解的日期是否为现在或将来的日期|同上 |
|`@Past`|检查带注释的日期是否是过去的日期|同上|
|`@PastOrPresent`|检查带注释的日期是否为过去或现在的日期|同上|
|`@NotEmpty`|检查带注释的元素是否不为null或为空|CharSequence，Collection，Map和数组|
|`@Size(min=, max=)`|检查带注释的元素的大小是否介于min和之间max（包括）| 同上|
|`@NotNull`|检查注释的值不是 null|任意种类|
|`@Null`|检查注释的值是 null|同上|
|`@Pattern(regex=, flags=)`|检查带注释的字符串是否与正则表达式匹配|CharSequence|
|`@URL(protocol=, host=, port=, regexp=, flags=)`|根据RFC2396检查带注释的字符序列是否为有效URL。|同上|
|`@Email`|检查指定的字符序列是否为有效的电子邮件地址。|同上|
|`@NotBlank`|检查被注解的字符序列是否不为null以及修剪后的长度是否大于0。与的区别@NotEmpty在于，此约束只能应用于字符序列，并且尾随空白将被忽略。|同上|



