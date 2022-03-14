---
title: JDBC各类详解以及工具类JDBCUtils的抽取
date: 2020-05-18 22:06:03
tags: java
---

# 概念
JDBC的全称为`Java DataBase Connectivity`，是官方（sun公司）定义的一套操作所有关系型数据库的规则，即接口。各个数据库厂商去实现这套接口，提供数据库驱动jar包。我们可以使用这套接口（JDBC）编程，真正执行的代码是驱动jar包中的实现类。

# 快速入门
> 点击[这里](https://mvnrepository.com/artifact/mysql/mysql-connector-java)下载最新jar包
**步骤**
1. 导入驱动jar包 `mysql-connector-java-5.1.37-bin.jar`(本人使用idea)
	1. 复制`mysql-connector-java-5.1.37-bin.jar`到项目的libs目录下(没有libs)
	2. 右键-->`Add As Library`

2. 注册驱动
3. 获取数据库连接对象 Connection
4. 定义sql
5. 获取执行sql语句的对象 Statement
6. 执行sql，接受返回结果
7. 处理结果
8. 释放资源

**代码实现**
```java
public class JDBCDemo1 {
    // MySQL 8.0 以下版本 - JDBC 驱动名及数据库 URL
    static final String JDBC_DRIVER = "com.mysql.jdbc.Driver";
    static final String DB_URL = "jdbc:mysql://localhost:3306/db";

    // MySQL 8.0 以上版本 - JDBC 驱动名及数据库 URL
    //static final String JDBC_DRIVER = "com.mysql.cj.jdbc.Driver";
    //static final String DB_URL = "jdbc:mysql://localhost:3306/db?useSSL=false&serverTimezone=UTC";


    // 数据库的用户名与密码，需要根据自己的设置
    static final String USER = "root";
    static final String PASS = "123456";
    public static void main(String[] args) throws Exception {
        Connection conn = null;
        Statement stmt = null;
        // 注册 JDBC 驱动
        Class.forName(JDBC_DRIVER);
        // 打开链接
        conn = DriverManager.getConnection(DB_URL,USER,PASS);
        // 定义sql
        String sql = "update account set balance = 500 where id = 1";
        // 获取执行sql语句的对象 Statement
        stmt = conn.createStatement();
        // 执行sql，接受返回结果
        int count=stmt.executeUpdate(sql);
        // 处理结果
        System.out.println(count);
        // 完成后关闭
        stmt.close();
        conn.close();
    }
}
```
# 详解各类对象

**DriverManager：驱动管理对象**

**功能：**
1. 注册驱动，告诉程序该使用哪一个数据库驱动jar
	1. 通过方法`registerDriver(Driver driver)`来注册给定的驱动程序，不过它被包裹在静态静态块中，执行`Class.forName("com.mysql.jdbc.Driver")`会自动执行该方法。
	2. 可通过查看`com.mysql.jdbc.Driver`类的源码可得。
2. 获取数据库连接
	1. 方法：`getConnection(String url,String user, String password)`
	2. 参数：
		1. url：指定连接的路径`jdbc:mysql://ip地址(域名):端口号/数据库名称`
		2. user：用户名
		3. password：密码

 **Connection：数据库连接对象**

**功能**
1. 获取执行sql 的对象
	* `Statement createStatement()`：执行静态的SQL语句
	* `PreparedStatement prepareStatement(String sql)`：执行动态的SQL语句（推荐，安全性高）  
2. 管理事务：
	* 开启事务：`setAutoCommit(boolean autoCommit)`：调用该方法设置参数为false，即开启事务
	* 提交事务：`commit()`
	* 回滚事务：`rollback()` 

**Statement：执行静态sql的对象**

**常用方法**
1. `int executeUpdate(String sql)`：执行数据操纵语言DML（insert、update、delete）语句、数据定义语言DDL(create，alter、drop)语句
	* 返回值：影响的行数，可以通过这个影响的行数判断DML语句是否执行成功 返回值>0的则执行成功，反之，则失败。
2. `ResultSet executeQuery(String sql) `：执行数据查询语言DQL（select)语句，并返回一个结果集`ResultSet`

**PreparedStatement：执行动态sql的对象**

安全性高，效率也高，能够解决sql注入的问题，推荐使用这个。

**步骤：**
1. 定义sql，如：`select * from user where username = ? and password = ?`注意：sql的参数使用？作为占位符
2. 获取执行sql语句的对象 `PreparedStatement  Connection.prepareStatement(String sql) `
3. 通过`setXxx(参数1，参数2)`方法给占位符`?`赋值
	1. 参数1：`?`的位置编号 从1 开始
	2. 参数2：`?`的值

**ResultSet：结果集对象**

**常用方法**

* `boolean next()`: 游标向下移动一行，判断当前行是否是最后一行末尾(是否有数据)，如果是，则返回false，如果不是则返回true
- `getXxx(参数)`:获取数据
	- Xxx：代表数据类型   如： `int getInt() ,	String getString()`
	- 参数：
		- int：代表列的编号,从1开始   如： `getString(1)`
		- String：代表列名称。 如： `getDouble("balance")`

# 抽取JDBC工具类 ： JDBCUtils
* 目的：简化书写
* 分析：
	1. 注册驱动也抽取
	2. 抽取一个方法获取连接对象
		* 需求：不想传递参数（麻烦），还得保证工具类的通用性。
		* 解决：配置文件
	3. 抽取一个方法释放资源

**代码实现**

首先在src目录下创建一个文件名为`jdbc.properties`的配置文件
```properties
url=jdbc:jdbc:mysql://localhost:3306/db
user=root
password=123456
driver=com.mysql.jdbc.Driver
```
然后编写工具类
```java
/**
 * JDBC工具类
 */
public class JDBCUtils {
    private static String url;
    private static String user;
    private static String password;
    private static String driver;
    /**
     * 文件的读取，只需要读取一次即可拿到这些值。使用静态代码块
     */
    static{
        //读取资源文件，获取值。

        try {
            //1. 创建Properties集合类。
            Properties pro = new Properties();
            
            //2. 加载文件
            pro.load(JDBCUtils.class.getClassLoader().getResourceAsStream("jdbc.properties"));

            //3. 获取数据，赋值
            url = pro.getProperty("url");
            user = pro.getProperty("user");
            password = pro.getProperty("password");
            driver = pro.getProperty("driver");
            //4. 注册驱动
            Class.forName(driver);
        } catch (IOException | ClassNotFoundException e) {
            e.printStackTrace();
        }
    }


    /**
     * 获取连接
     * @return 连接对象
     */
    public static Connection getConnection() throws SQLException {

        return DriverManager.getConnection(url, user, password);
    }

    /**
     * 释放资源
     * @param stmt
     * @param conn
     */
    public static void close(Statement stmt, Connection conn){
        close(null,stmt,conn);
    }


    /**
     * 释放资源
     * @param stmt
     * @param conn
     */
    public static void close(ResultSet rs, Statement stmt, Connection conn){
        if( rs != null){
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if( stmt != null){
            try {
                stmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }

        if( conn != null){
            try {
                conn.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }

}
```
