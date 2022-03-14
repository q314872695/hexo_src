---
title: java8新特性之Stream流
date: 2020-05-16 08:55:08
tags: java
---

# 引言
## 传统集合的多步遍历代码
几乎所有的集合（如`Collection `接口或`Map `接口等）都支持直接或间接的遍历操作。而当我们需要对集合中的元 素进行操作的时候，除了必需的添加、删除、获取外，最典型的就是集合遍历。例如：
```java
public class Demo01ForEach { 
	public static void main(String[] args) { 
		List<String> list = new ArrayList<>(); 
		list.add("张无忌"); 
		list.add("周芷若"); 
		list.add("赵敏"); 
		list.add("张强"); 
		list.add("张三丰"); 
		for (String name : list) { 
			System.out.println(name); 
		} 
	}
}
```
这是一段非常简单的集合遍历操作：对集合中的每一个字符串都进行打印输出操作。
## 循环遍历的弊端
Java 8的Lambda让我们可以更加专注于做什么（What），而不是怎么做（How），这点此前已经结合内部类进行 了对比说明。现在，我们仔细体会一下上例代码，可以发现： 
- for循环的语法就是“怎么做” 
- for循环的循环体才是“做什么” 

为什么使用循环？因为要进行遍历。但循环是遍历的唯一方式吗？遍历是指每一个元素逐一进行处理，而并不是从 第一个到最后一个顺次处理的循环。前者是目的，后者是方式。 
试想一下，如果希望对集合中的元素进行筛选过滤： 
1. 将集合A根据条件一过滤为子集B
2. 然后再根据条件二过滤为子集C

那怎么办？在Java 8之前的做法可能为：
```java
public class Demo02NormalFilter { 
	public static void main(String[] args) { 
		List<String> list = new ArrayList<>(); 
		list.add("张无忌"); 
		list.add("周芷若"); 
		list.add("赵敏"); 
		list.add("张强"); 
		list.add("张三丰"); 
		List<String> zhangList = new ArrayList<>(); 
		for (String name : list) { 
			if (name.startsWith("张")) { 
				zhangList.add(name); 
			} 
		}
		List<String> shortList = new ArrayList<>(); 
		for (String name : zhangList) { 
			if (name.length() == 3) { 
				shortList.add(name); 
			} 
		}
		for (String name : shortList) { 
			System.out.println(name); 
		} 
	} 
}
```
这段代码中含有三个循环，每一个作用不同： 
1. 首先筛选所有姓张的人； 
2. 然后筛选名字有三个字的人； 
3. 最后进行对结果进行打印输出。 

每当我们需要对集合中的元素进行操作的时候，总是需要进行循环、循环、再循环。这是理所当然的么？不是。循 环是做事情的方式，而不是目的。另一方面，使用线性循环就意味着只能遍历一次。如果希望再次遍历，只能再使 用另一个循环从头开始。 
那，Lambda的衍生物Stream能给我们带来怎样更加优雅的写法呢？

## Stream的更优写法
下面来看一下借助Java 8的Stream API，什么才叫优雅：
```java
public class Demo1 {
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("张无忌");
        list.add("张三丰");
        list.add("周芷若");
        list.add("张强");
        list.add("赵敏");
        list.add("赵敏");

        list.stream().filter(name -> name.startsWith("张"))
                .filter(name -> name.length() == 3)
                .forEach(name-> System.out.println(name));
    }
}
```
直接阅读代码的字面意思即可完美展示无关逻辑方式的语义：获取流、过滤姓张、过滤长度为3、逐一打印。代码 中并没有体现使用线性循环或是其他任何算法进行遍历，我们真正要做的事情内容被更好地体现在代码中。
# 流式思想概述
**注意：请暂时忘记对传统IO流的固有印象！** 

整体来看，流式思想类似于工厂车间的“生产流水线”。
![image.png](https://halo-1257208482.image.myqcloud.com/image_1589589041686.png!webp) 
当需要对多个元素进行操作（特别是多步操作）的时候，考虑到性能及便利性，我们应该首先拼好一个“模型”步骤 方案，然后再按照方案去执行它。
![image.png](https://halo-1257208482.image.myqcloud.com/image_1589589105679.png!webp)
这张图中展示了过滤、映射、跳过、计数等多步操作，这是一种集合元素的处理方案，而方案就是一种“函数模 型”。图中的每一个方框都是一个“流”，调用指定的方法，可以从一个流模型转换为另一个流模型。而最右侧的数字 3是最终结果。 

这里的`filter`、`map`、`skip`都是在对函数模型进行操作，集合元素并没有真正被处理。只有当终结方法 count 执行的时候，整个模型才会按照指定策略执行操作。而这得益于Lambda的延迟执行特性。 

> 备注：“Stream流”其实是一个集合元素的函数模型，它并不是集合，也不是数据结构，其本身并不存储任何 元素（或其地址值）。 

Stream（流）是一个来自数据源的元素队列 
- 元素是特定类型的对象，形成一个队列。 Java中的Stream并不会存储元素，而是按需计算。 
- 数据源 流的来源。可以是集合，数组 等。 

和以前的Collection操作不同， Stream操作还有两个基础的特征： 
- Pipelining: 中间操作都会返回流对象本身。 这样多个操作可以串联成一个管道， 如同流式风格（fluent style）。 这样做可以对操作进行优化， 比如延迟执行(laziness)和短路( short-circuiting)。
- 内部迭代： 以前对集合遍历都是通过Iterator或者增强for的方式, 显式的在集合外部进行迭代， 这叫做外部迭 代。 Stream提供了内部迭代的方式，流可以直接调用遍历方法。 当使用一个流的时候，通常包括三个基本步骤：获取一个数据源（source）→ 数据转换→执行操作获取想要的结 果，每次转换原有 Stream 对象不改变，返回一个新的 Stream 对象（可以有多次转换），这就允许对其操作可以 像链条一样排列，变成一个管道。
# 获取流
`java.util.stream.Stream<T>`是Java 8新加入的最常用的流接口。（这并不是一个函数式接口。）
获取一个流非常简单，有以下几种常用的方式：
- 所有的 Collection 集合都可以通过 stream 默认方法获取流
- Stream 接口的静态方法 of 可以获取数组对应的流。

# 常用方法
流模型的操作很丰富，这里介绍一些常用的API。这些方法可以被分成两种：
- 延迟方法：返回值类型仍然是 Stream 接口自身类型的方法，因此支持链式调用。（除了终结方法外，其余方 法均为延迟方法。）
- 终结方法：返回值类型不再是 Stream 接口自身类型的方法，因此不再支持类似 StringBuilder 那样的链式调 用。本小节中，终结方法包括 count 和 forEach 方法

## 逐一处理：forEach
虽然方法名字叫 forEach ，但是与for循环中的“for-each”昵称不同。
```java
void forEach(Consumer<? super T> action);
```
该方法接收一个 Consumer 接口函数，会将每一个流元素交给该函数进行处理。

**基本使用：**
```java
public class Demo12StreamForEach { 
	public static void main(String[] args) { 
		Stream<String> stream = Stream.of("张无忌", "张三丰", "周芷若"); 
		stream.forEach(name‐> System.out.println(name)); 	
	} 
}
```
## 过滤：filter
可以通过 filter 方法将一个流转换成另一个子集流。方法签名：
```java
Stream<T> filter(Predicate<? super T> predicate);
```
该接口接收一个 Predicate 函数式接口参数（可以是一个Lambda或方法引用）作为筛选条件。

**基本使用:**
```java
public class Demo07StreamFilter { 
	public static void main(String[] args) { 
		Stream<String> original = Stream.of("张无忌", "张三丰", "周芷若"); 
		Stream<String> result = original.filter(s ‐> s.startsWith("张")); 
	} 
}
```
在这里通过Lambda表达式来指定了筛选的条件：必须姓张。
## 映射：map
如果需要将流中的元素映射到另一个流中，可以使用 map 方法。方法签名：
```java
<R> Stream<R> map(Function<? super T, ? extends R> mapper);
```
该接口需要一个 Function 函数式接口参数，可以将当前流中的T类型数据转换为另一种R类型的流。

**基本使用:**
```java
public class Demo08StreamMap { 
	public static void main(String[] args) { 
		Stream<String> original = Stream.of("10", "12", "18"); 
		Stream<Integer> result = original.map(str‐>Integer.parseInt(str)); 
	} 
}
```
## 去重：distinct
```java
Stream<T> distinct()
```
根据`equals()`方法去重

## 排序：sorted
```java
Stream<T> sorted()
```
返回由该流的元素组成的流，按自然顺序排序。如果该流的元素是不 Comparable，一 java.lang.ClassCastException可能当终端操作执行。 
```java
Stream<T> sorted(Comparator<? super T> comparator)
```
返回一个包含该流的元素流，根据提供的 Comparator排序。

## 统计个数：count
正如旧集合 Collection 当中的 size 方法一样，流提供 count 方法来数一数其中的元素个数：
```java
long count();
```
**基本使用：**
```java
public class Demo09StreamCount { 
	public static void main(String[] args) { 
		Stream<String> original = Stream.of("张无忌", "张三丰", "周芷若"); 
		Stream<String> result = original.filter(s ‐> s.startsWith("张")); 
		System.out.println(result.count()); // 2 
	} 
}
```
## 取用前几个：limit
如果希望跳过前几个元素，可以使用 skip 方法获取一个截取之后的新流：
```java
Stream<T> skip(long n);
```
如果流的当前长度大于n，则跳过前n个；否则将会得到一个长度为0的空流。

**基本使用：**
```java
public class Demo11StreamSkip { 
	public static void main(String[] args) { 
		Stream<String> original = Stream.of("张无忌", "张三丰", "周芷若"); 
		Stream<String> result = original.skip(2); 
		System.out.println(result.count()); // 1 
	} 
}
```
## 组合：concat
如果有两个流，希望合并成为一个流，那么可以使用 Stream 接口的静态方法 concat ：
```java
static <T> Stream<T> concat(Stream<? extends T> a, Stream<? extends T> b)
```

**基本使用：**
```java
public class Demo12StreamConcat { 
	public static void main(String[] args) { 
		Stream<String> streamA = Stream.of("张无忌"); 
		Stream<String> streamB = Stream.of("张翠山"); 
		Stream<String> result = Stream.concat(streamA, streamB); 
	} 
}
```


