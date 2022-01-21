---
title: List去掉重复数据的几种方式
date: 2020-03-29 20:26:43
tags: java
---



总结了去重的几种方式。

<!--more-->

### 使用LinkedHashSet删除arraylist中的重复数据(有序)

```java
List<String> words= Arrays.asList("a","b","b","c","c","d");
HashSet<String> set=new LinkedHashSet<>(words);
for(String word:set){
      System.out.println(word);
}
```
### 使用HashSet去重(无序)
``` java
//去掉List集合中重复的元素
List<String> words= Arrays.asList("a","b","b","c","c","d");
//方案一：
for(String word:words){
    set.add(word);
}
for(String word:set){
    System.out.println(word);
}
```
### 使用java8新特性stream进行List去重
``` java
List<String> words= Arrays.asList("a","b","b","c","c","d");
words.stream().distinct().collect(Collectors.toList()).forEach(System.out::println);
```
### 利用List的contains方法循环遍历
```java
List<String> list= new ArrayList<>();
        for (String s:words) {
            if (!list.contains(s)) {
                list.add(s);
            }
        }
```
注：当数据元素是实体类时，需要额外重写equals()和hashCode()方法。
例如：
以学号为依据判断重复
```java
public class Student {
    String id;
    String name;
    int age;

    public Student(String id, String name, int age) {
        this.id = id;
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Student student = (Student) o;

        return Objects.equals(id, student.id);
    }

    @Override
    public int hashCode() {
        return id != null ? id.hashCode() : 0;
    }

    @Override
    public String toString() {
        return "Student{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```

