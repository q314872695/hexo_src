---
title: 使用Java操作json字符串
tags:
  - java
  - json
abbrlink: d81a6229
date: 2020-05-27 18:07:11
---

# JSON解析器：
本例中使用得json解析器是jackson，点击[这里](https://mvnrepository.com/artifact/com.fasterxml.jackson.core/jackson-databind/2.11.0) 可以下载最新版jar包，
	
# JSON转为Java对象
1. 导入jackson的相关jar包
	- `jackson-databind-2.2.3.jar`
	- `jackson-annotations-2.2.3.jar`（依赖包）
	- `jackson-core-2.2.3.jar`（依赖包）
2. 创建Jackson核心对象`ObjectMapper`
3. 调用`ObjectMapper`的相关方法进行转换
	- `readValue(String json,类名.class)`
```java
class Person{
    private String name;
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

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
public class Demo1 {
    public static void main(String[] args) throws IOException {
        ObjectMapper objectMapper = new ObjectMapper();
        String json = "{\"name\":\"张三\",\"age\":18}";
        // json 转换实体类
        Person person = objectMapper.readValue(json, Person.class);
        System.out.println(person);
        // json 转换map对象
        Map map = objectMapper.readValue(json, Map.class);
        System.out.println(map);
        // json 转换list对象
        String json2 = "[{\"name\":\"张三\",\"age\":18},{\"name\":\"李四\",\"age\":19}]";

        List list = objectMapper.readValue(json2, List.class);
        System.out.println(list);
    }
}
结果：
Person{name='张三', age=18}
{name=张三, age=18}
[{name=张三, age=18}, {name=李四, age=19}]
```

# Java对象转换JSON
1. 导入jackson的相关jar包(同上)
2. 创建Jackson核心对象`ObjectMapper`
3. 调用`ObjectMapper`的相关方法进行转换
	* `writeValue(参数1，obj)`:
		- 参数1：
			- `File`：将obj对象转换为JSON字符串，并保存到指定的文件中
			- `Writer`：将obj对象转换为JSON字符串，并将json数据填充到字符输出流中
			- `OutputStream`：将obj对象转换为JSON字符串，并将json数据填充到字节输出流中
	* `writeValueAsString(obj)`:将对象转为json字符串

```java
public class Demo2 {
    public static void main(String[] args) throws IOException {
        ObjectMapper objectMapper = new ObjectMapper();
        // 实体类转换json
        Person person = new Person("张三", 18);
        String s = objectMapper.writeValueAsString(person);
        System.out.println(s);
        // list 对象转换json
        ArrayList<Person> list = new ArrayList<>();
        list.add(new Person("古力娜扎", 18));
        list.add(new Person("迪丽热巴", 19));
        list.add(new Person("马儿扎哈", 20));
        System.out.println(objectMapper.writeValueAsString(list));
        // map对象转换json
        HashMap map = new HashMap();
        map.put("name", "葫芦娃");
        map.put("age", "200");
        System.out.println(objectMapper.writeValueAsString(map));
    }
}
```
# 注解：
1. `@JsonIgnore`：排除属性。
2. `@JsonFormat`：属性值得格式化，例如`@JsonFormat(pattern = "yyyy-MM-dd")`
```java
class Person{
	private String name;
    
	private int age;
    
	@JsonFormat(pattern = "yyyy-MM-dd")
	private Date birthday;

	// getter ...setter ...
}
```

