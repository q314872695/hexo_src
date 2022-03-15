---
title: java常用的数据库连接池C3P0、Druid以及Spring JDBC总结
tags: java
abbrlink: 7fc31dd5
date: 2020-05-19 18:46:05
---

# 数据库连接池简介
**概念**

数据库连接池就是一个容器(集合)，存放数据库连接的容器。当系统初始化好后，容器被创建，容器中会申请一些连接对象，当用户来访问数据库时，从容器中获取连接对象，用户访问完之后，会将连接对象归还给容器。

**好处**

1. 节约资源
2. 用户访问高效

**实现**

1. 标准接口：`javax.sql.DataSource`
	1. 方法：
		* 获取连接：`getConnection()`
		* 归还连接：`Connection.close()`如果连接对象Connection是从连接池中获取的，那么调用Connection.close()方法，则不会再关闭连接了。而是归还连接

	2. 一般我们不去实现它，有数据库厂商来实现
		1. C3P0：数据库连接池技术
		2. Druid：数据库连接池实现技术，由阿里巴巴提供的

# C3P0
> 可以在这里下载 [这里](https://mvnrepository.com/artifact/com.mchange/c3p0/0.9.5.5) 最新版jar包 

**步骤**
1. 导入jar包（不要忘记导入数据库驱动jar包）
	- `c3p0-0.9.5.2.jar` 
	- `mchange-commons-java-0.2.12.jar`(c3p0的依赖jar包)
2. 定义配置文件：
	- 名称： `c3p0.properties` 或者 `c3p0-config.xml`
	- 路径：**直接将文件放在src目录下即可**。

3. 创建核心对象 数据库连接池对象 `ComboPooledDataSource`
4. 获取连接： `getConnection`

**代码**
1. 定义配置文件`c3p0-config.xml`
```xml
<c3p0-config>
  <!-- 使用默认的配置读取连接池对象 -->
  <default-config>
  	<!--  连接参数 -->
    <property name="driverClass">com.mysql.jdbc.Driver</property>
    <property name="jdbcUrl">jdbc:mysql://localhost:3306/db</property>
    <property name="user">root</property>
    <property name="password">123456</property>
    
    <!-- 连接池参数 -->
    <!--初始化申请的连接数量-->
    <property name="initialPoolSize">5</property>
    <!--最大的连接数量-->
    <property name="maxPoolSize">10</property>
    <!--超时时间-->
    <property name="checkoutTimeout">3000</property>
  </default-config>

  <named-config name="otherc3p0"> 
    <!--  连接参数 -->
    <property name="driverClass">com.mysql.jdbc.Driver</property>
    <property name="jdbcUrl">jdbc:mysql://localhost:3306/db</property>
    <property name="user">root</property>
    <property name="password">123456</property>
    
    <!-- 连接池参数 -->
    <property name="initialPoolSize">5</property>
    <property name="maxPoolSize">8</property>
    <property name="checkoutTimeout">1000</property>
  </named-config>
</c3p0-config>
```
2. 编写测试类
```java
public class C3P0Demo1 {
    public static void main(String[] args) throws SQLException {
        // 创建数据库连接池对象，不传参数使用默认配置
        DataSource ds = new ComboPooledDataSource();
	// 传参时，只用指定的配置
	// DataSource ds = new ComboPooledDataSource("otherc3p0");
        // 获取连接对象
        Connection connection = ds.getConnection();
        PreparedStatement ps = connection.prepareStatement("select * from student");
        ResultSet resultSet = ps.executeQuery();
        ArrayList<Student> students = new ArrayList<>();
        while (resultSet.next()) {
            Student stu = new Student(
                    resultSet.getInt("id"),
                    resultSet.getString("name"),
                    resultSet.getInt("age"),
                    resultSet.getString("sex"),
                    resultSet.getString("address"),
                    resultSet.getInt("math"),
                    resultSet.getInt("english")
            );
            students.add(stu);
        }
        // 把连接对象归还数据库连接池
        connection.close();
        students.forEach(System.out::println);
    }
}
```

# Druid
> 可以在这里下载 [这里](https://mvnrepository.com/artifact/com.alibaba/druid/1.1.22) 最新版jar包 ，推荐使用这里，由阿里巴巴提供国产的

**步骤**
1. 导入jar包 `druid-1.0.9.jar`(不要忘记导入数据库驱动jar包)
2. 定义配置文件：
	- 是`xxx.properties`形式的
	- 可以叫任意名称，可以放在任意目录下
3. 加载配置文件。
4. 获取数据库连接池对象：通过工厂来来获取`DruidDataSourceFactory`
5. 获取连接：`getConnection`

**代码**

1. 定义配置文件
```properties
driverClassName=com.mysql.jdbc.Driver
url=jdbc:mysql://127.0.0.1:3306/db
username=root
password=123456
initialSize=5
maxActive=10
maxWait=3000
```
2. 编写测试类
```java
public class DruidDemo1 {
    public static void main(String[] args) throws Exception {
        //1.加载配置文件
        Properties pro = new Properties();
        InputStream is = DruidDemo1.class.getClassLoader().getResourceAsStream("druid.properties");
        pro.load(is);
        //2.获取连接池对象
        DataSource ds = DruidDataSourceFactory.createDataSource(pro);
        //3.获取连接
        Connection connection = ds.getConnection();
        
        PreparedStatement ps = connection.prepareStatement("select * from student");
        ResultSet resultSet = ps.executeQuery();
        ArrayList<Student> students = new ArrayList<>();
        while (resultSet.next()) {
            Student stu = new Student(
                    resultSet.getInt("id"),
                    resultSet.getString("name"),
                    resultSet.getInt("age"),
                    resultSet.getString("sex"),
                    resultSet.getString("address"),
                    resultSet.getInt("math"),
                    resultSet.getInt("english")
            );
            students.add(stu);
        }
        // 把连接对象归还数据库连接池
        connection.close();
        students.forEach(System.out::println);
    }
}
```

**定义工具类**
1. 定义一个类 `JDBCUtils`
2. 提供静态代码块加载配置文件，初始化连接池对象
3. 提供方法
	1. 获取连接方法：通过数据库连接池获取连接
	2. 释放资源
	3. 获取连接池的方法(使用`Spring JDBC`时需要)

```java
public class JDBCUtils {

    //1.定义成员变量 DataSource
    private static DataSource ds ;

    static{
        try {
            //1.加载配置文件
            Properties pro = new Properties();
            pro.load(JDBCUtils.class.getClassLoader().getResourceAsStream("druid.properties"));
            //2.获取DataSource
            ds = DruidDataSourceFactory.createDataSource(pro);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 获取连接
     */
    public static Connection getConnection() throws SQLException {
        return ds.getConnection();
    }

    /**
     * 释放资源
     */
    public static void close(Statement stmt, Connection conn){
        close(null,stmt,conn);
    }


    public static void close(ResultSet rs , Statement stmt, Connection conn){

        if(rs != null){
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }


        if(stmt != null){
            try {
                stmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if(conn != null){
            try {
                conn.close();//归还连接
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

    /**
     * 获取连接池方法
     */

    public static DataSource getDataSource(){
        return  ds;
    }
}
```
# Spring JDBC
> 点击[这里](https://mvnrepository.com/artifact/org.springframework/spring-jdbc/5.2.6.RELEASE)，下载最新版jar包

Spring框架对JDBC的简单封装。提供了一个JDBCTemplate对象简化JDBC的开发
**步骤**
1. 导入jar包
	- `spring-jdbc-5.0.0.RELEASE-sources.jar`
	- `spring-beans-5.0.0.RELEASE-sources.jar`(依赖包)
	- `spring-core-5.0.0.RELEASE-sources.jar`(依赖包)
	- `spring-tx-5.0.0.RELEASE-sources.jar`(依赖包)


2. 创建JdbcTemplate对象。依赖于数据源（数据库连接池）DataSource
	- `JdbcTemplate template = new JdbcTemplate(ds)`

3. 调用JdbcTemplate的方法来完成CRUD的操作（常用）
	- `update(String sql,Object... args)`:执行DML语句。增、删、改语句
	- `queryForMap(String sql,Object... args)`:查询结果将结果集封装为map集合，将列名作为key，将值作为value 将这条记录封装为一个map集合
		- 注意：这个方法查询的结果集长度只能是1
	- `queryForList(String sql,Object... args)`:查询结果将结果集封装为list集合
		- 注意：将每一条记录封装为一个Map集合，再将Map集合装载到List集合中
	- `query(String sql,RowMapper<T> rowMapper,Object... args)`:查询结果，将结果封装为JavaBean对象
		- query的参数：`RowMapper`
			- 一般我们使用`BeanPropertyRowMapper`实现类。可以完成数据到JavaBean的自动封装
			- `new BeanPropertyRowMapper<类型>(类型.class)`
	- `queryForObject(String sql,Class<T> requiredType,Object... args)`：查询结果，将结果封装为对象
		- 一般用于聚合函数的查询

> 注意：写sql语句时用？作为占位符，在调用方法时，传入sql语句，占位符要赋的值即可
例如：`template.queryForList("select * from student where id=?",1)`,参数不限，有多个参数时，占位符赋值方法参数列表最后


**案例**

**Student.java**
```java
public class Student {
    private int id;
    private String name;
    private int age;
    private String sex;
    private String address;
    private int math;
    private int english;

    // 当有其他构造方法时，必须又个空参的构造方法，否则spring jdbc会报错，或者都不写也可以
    public Student() {
    }

    public Student(int id, String name, int age, String sex, String address, int math, int english) {
        this.id = id;
        this.name = name;
        this.age = age;
        this.sex = sex;
        this.address = address;
        this.math = math;
        this.english = english;
    }

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

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

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    public int getMath() {
        return math;
    }

    public void setMath(int math) {
        this.math = math;
    }

    public int getEnglish() {
        return english;
    }

    public void setEnglish(int english) {
        this.english = english;
    }

    public String getSex() {
        return sex;
    }

    public void setSex(String sex) {
        this.sex = sex;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id=" + id +
                ", name='" + name + '\'' +
                ", age=" + age +
                ", sex='" + sex + '\'' +
                ", address='" + address + '\'' +
                ", math=" + math +
                ", english=" + english +
                '}';
    }
}

```

**测试类**

```java
public class SpringJDBCDemo{


    public static void main(String[] args) {
        JdbcTemplate template = new JdbcTemplate(JDBCUtils2.getDataSource());
        List<Map<String, Object>> maps = template.queryForList("select * from student where id=?",1);
        for (Map<String, Object> map : maps) {
            System.out.println(map);
        }
        System.out.println("----------------------------");
        // Student 需要一个空参构造方法
        List<Student> query = template.query("select * from student where id=? or id=?", new BeanPropertyRowMapper<Student>(Student.class),1,3);
        query.forEach(System.out::println);

        System.out.println("----------------------------");
        Integer integer = template.queryForObject("select count(id) from student where sex=?", Integer.class,"男");
        System.out.println(integer);

        System.out.println("----------------------------");

    }
}
运行结果：
五月 19, 2020 6:37:52 下午 com.alibaba.druid.pool.DruidDataSource info
信息: {dataSource-1} inited
{id=1, name=马云, age=55, sex=男, address=杭州, math=66, english=78}
----------------------------
Student{id=1, name='马云', age=55, sex='男', address='杭州', math=66, english=78}
Student{id=3, name='马景涛', age=55, sex='男', address='香港', math=56, english=77}
----------------------------
5
----------------------------

Process finished with exit code 0
```


