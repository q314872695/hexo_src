---
title: Map接口及其子类介绍
tags: java
abbrlink: f2167a39
date: 2020-05-11 10:05:37
---

# Map接口
现实生活中，我们常会看到这样的一种集合：IP地址与主机名，身份证号与个人，系统用户名与系统用户对象等，这种一一对应的关系，就叫做映射。Java提供了专门的集合类用来存放这种对象关系的对象，即`java.util.Map`接口。

1. Map集合是一个双列集合,一个元素包含两个值(一个key,一个value)
2. Map集合中的元素,key和value的数据类型可以相同,也可以不同
3. Map集合中的元素,key是不允许重复的,value是可以重复的
4. Map集合中的元素,key和value是一一对应

## Map接口中的常用方法
* `public V put(K key, V value)`:  把指定的键与指定的值添加到Map集合中。
* `public V remove(Object key)`: 把指定的键 所对应的键值对元素 在Map集合中删除，返回被删除元素的值。
* `public V get(Object key)` 根据指定的键，在Map集合中获取对应的值。
* `boolean containsKey(Object key)  ` 判断集合中是否包含指定的键。
* `public Set<K> keySet()`: 获取`Map`集合中所有的键，存储到`Set`集合中。
* `public Set<Map.Entry<K,V>> entrySet()`: 获取到`Map`集合中所有的键值对对象的集合(Set集合)。

## Map常用子类
通过查看`Map`接口描述，看到`Map`有多个子类，这里我们主要讲解常用的`HashMap`集合、`LinkedHashMap`集合。

### HashMap<K,V>
存储数据采用的哈希表结构，元素的存取顺序不能保证一致。由于要保证键的唯一、不重复，需要重写键的`hashCode()`方法、`equals()`方法。

1. `HashMap`集合底层是哈希表:查询的速度特别的快
	- JDK1.8之前:数组+单向链表
	- JDK1.8之后:数组+单向链表|红黑树(链表的长度超过8):提高查询的速度
2. `hashMap`集合是一个无序的集合,存储元素和取出元素的顺序有可能不一致
### LinkedHashMap<K,V>
`HashMap`下有个子类`LinkedHashMap`，存储数据采用的哈希表结构+链表结构。通过链表结构可以保证元素的存取顺序一致；通过哈希表结构可以保证的键的唯一、不重复，需要重写键的`hashCode()`方法、`equals()`方法。

1. `LinkedHashMap`集合底层是哈希表+链表(保证迭代的顺序)
2. `LinkedHashMap`集合是一个有序的集合,存储元素和取出元素的顺序是一致的