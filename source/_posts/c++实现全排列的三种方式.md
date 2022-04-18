---
title: c++实现全排列的三种方式
tags: 算法笔记
abbrlink: 69d8f6f
date: 2022-04-18 14:30:59
---



# 递归方式

```cpp
#include <cstdio>
#include <iostream>
#include <algorithm>
#include <string>
using namespace std;
const int MAXN = 10;

bool visit[MAXN];//判断某个元素是否被访问过
char sequence[MAXN];//存放找到的全排列

void GetPermutation(string str, int index){
    // 找到结果并打印
    if (index == str.size()) {
        // 打印结果
        for (int i = 0; i < str.size(); ++i) {
            putchar(sequence[i]);
        }
        printf("\n");
    }
    for (int i = 0; i < str.size(); ++i) {
        if (visit[i]) {//被访问过就跳过
            continue;
        } else {
            visit[i] = true;
            sequence[index] = str[i];
            //接着查找下一位
            GetPermutation(str, index + 1);
            visit[i] = false;
        }
    }
}
int main(){
    string str;
    while (cin >> str) {
        sort(str.begin(), str.end());// 输入的字符串排序，保证以字典序输出
        GetPermutation(str, 0);
        printf("\n");
    }
    return 0;
}
```



# 非递归方式

```cpp
#include <cstdio>
#include <iostream>
#include <algorithm>
#include <string>
using namespace std;

//非递归方式
//依次给出该排列的下一个排列
bool GetNextPermutation(string &str){
    int n = str.size();
    int index = n - 2;// 指向倒数第二个字符的下标
    //如果当前字符比后面的字符大就前移
    while (index >= 0 && str[index] >= str[index + 1]) {
        index--;
    }
    // 已经是字典序最大了
    if (index < 0) {
        return false;
    }
    for (int i = n - 1; i > index; --i) {
        // 找到第一个大于index的字符，然后交换
        if (str[i] > str[index]) {
            swap(str[index], str[i]);
            break;
        }
    }
    reverse(str.begin() + index + 1, str.end());
    return true;
}
int main(){
    string str;
    while (cin >> str) {
        sort(str.begin(), str.end());
        do {
            cout << str << endl;
        } while (GetNextPermutation(str));
        cout << endl;
    }

    return 0;
}
```



# 使用系统函数

`next_permutation()`和非递归方式求全排列的用法一样，是计算当前序列的下一个序列，该函数位于`algorithm`头文件中。

它有三个参数：

- 序列的首地址
- 序列的尾地址
- 比较函数（可选），默认是字典序排列

```cpp
#include <iostream>
#include <algorithm>
#include <string>
using namespace std;

int main(){
    string str;
    while (cin >> str) {
        sort(str.begin(), str.end());
        do {
            cout << str << endl;
        } while (next_permutation(str.begin(), str.end()));
        cout << endl;
    }

    return 0;
}
```

