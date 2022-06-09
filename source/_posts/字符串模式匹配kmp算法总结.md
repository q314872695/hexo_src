---
title: 字符串模式匹配kmp算法总结
tags: 算法笔记
abbrlink: 65380b26
date: 2022-05-25 09:56:43
---


# 暴力匹配算法

首先呢先展示一下万能的暴力匹配算法，它的思想最简单，只是时间复杂度高了那么一点点，不过问题不大。

**算法思想：**

从主串的第一个字符起，与模式串的第一个字符比较，若相等，则继续逐个比较后续字符；否则从主串的下一个字符起，重新和模式串的字符比较；以此类推，直至模式串中的每个字符依次和主串中的一个连续的字符序列相等，则称匹配成功，函数值为与模式串中第一个字符相等的字符在主串中的序号，否则称匹配不成功，函数值为-1。

**代码：**

```java
// 暴力匹配算法
public static int indexBF(String text, String pattern) {
    int i = 0, j = 0, k = 0;
    while (i < text.length() && j < pattern.length()) {
        // 模式串和主串的每个字符一次比较
        if (text.charAt(i) == pattern.charAt(j)) { // 匹配成功就后移就继续比较后续字符
            i++;
            j++;
        } else { // 未匹配成功
            k++;
            i = k; // 主串后移一个字符
            j = 0; // 模式串重新从头开始
        }
    }
    if (j == pattern.length()) { // j到末尾就匹配成功 了
        return k;
    } else {
        return -1;
    }
}
```

# KMP算法

## 求next数组

**PM表**

**字符串的前缀、后缀和部分匹配值**

- 前缀：是指除最后一个字符外，字符串的所有头部子串。
- 后缀：是指除第一个字符外，字符串的所有尾部子串。
- 部分匹配值：字符串的前缀和后缀的最长相等前后缀长度。

以`ababa`为例说明：

| 子串  | 前缀          | 后缀          | 最长相等的前后缀长度 |
| ----- | ------------- | ------------- | -------------------- |
| a     | 无            | 无            | 0                    |
| ab    | a             | b             | 0                    |
| aba   | a,ab          | a,ba          | 1                    |
| abab  | a,ab,aba      | b,ab,bab      | 2                    |
| ababa | a,ab,aba,abab | a,ba,aba,baba | 3                    |

故字符串`ababa`的部分匹配值为00123。

将部分匹配值写成数组形式就得到了**部分匹配值表**，**PM表**

| 下标 | 0    | 1    | 2    | 3    | 4    |
| ---- | ---- | ---- | ---- | ---- | ---- |
| val  | a    | b    | a    | b    | a    |
| PM   | 0    | 0    | 1    | 2    | 3    |

将pm表右移一位就等到了**next数组**，左边的空缺用-1来填充。

| 下标 | 0    | 1    | 2    | 3    | 4    |
| ---- | ---- | ---- | ---- | ---- | ---- |
| val  | a    | b    | a    | b    | a    |
| next | -1   | 0    | 0    | 1    | 2    |

> 数据结构书上的字符串下标是从1开始的，实际编程中下标是从0开始，所以稍微有些出入

以下用代码来求next数组

```java
// 以-1开始
public static int[] getNext(String pattern){
    int[] next=new int[pattern.length()];
    int i=0; // 后缀序列，主串
    int j=-1; // 前缀序列，模式串 也等于最长的相等前后缀
    next[0]=-1; // 公式next第一位-1
    while(i<pattern.length()-1){
        if(j==-1||pattern.charAt(i)==pattern.charAt(j)){ // 前后缀匹配上就给后一位的next数组赋值
            i++;
            j++;
            next[i]=j; // next[j+1]=next[j]+1
        }else{
            j=next[j];
        }
    }
    return next;
}
```

## 实现KMP

```java
public static int indexKMP(String text, String pattern) {
    int i = 0, j = 0;
    int[] next=getNext(pattern); // 先得到next数组
    while (i < text.length() && j < pattern.length()) {
        if (j==-1||text.charAt(i) == pattern.charAt(j)) { // 匹配成功就后移就继续比较后续字符
            i++;
            j++;
        } else { // 未匹配成功,j回退
            j=next[j];
        }
    }
    if (j == pattern.length()) { // j到末尾就匹配成功 了
        return i-pattern.length();
    } else {
        return -1;
    }
}
```

