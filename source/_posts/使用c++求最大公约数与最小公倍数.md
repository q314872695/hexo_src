---
title: 使用c++求最大公约数与最小公倍数
date: 2022-03-29 20:51:15
tags: 算法笔记
---

# 分析

**最大公约数**

最大公约数是指两个或多个整数共有约数中，最大的一个约数。常用的方法是欧几里得算法，也叫辗转相除法。

假如需要求 1997 和 615 两个正整数的最大公约数,用欧几里得算法，是这样进行的：

1997 / 615 = 3 (余 152)

615 / 152 = 4(余7)

152 / 7 = 21(余5)

7 / 5 = 1 (余2)

5 / 2 = 2 (余1)

2 / 1 = 2 (余0)

至此，最大公约数为1

以除数和余数反复做除法运算，当余数为 0 时，取当前算式除数为最大公约数，所以就得出了 1997 和 615 的最大公约数

**最小公倍数**

知道了最大公约数，那么求最小公倍数就迎刃而解了，因为有这样一个公式：a,b的最小公倍数=a*b/(a和b的最大公约数)

# 代码

**最大公约数**

```cpp
#include <cstdio>

using namespace std;

// 使用辗转相除法求最大公约数
int GCD(int a,int b){
    if (b == 0) {
        return a;
    }else{
        return GCD(b, a % b);
    }
}
int main(){
    int a,b;
    while (scanf("%d%d",&a,&b)!=EOF){
        printf("%d\n",GCD(a,b));
    }
    return 0;
}
```

**最小公倍数**

```cpp
#include <cstdio>

using namespace std;

// 使用辗转相除法求最大公约数
int GCD(int a,int b){
    if (b == 0) {
        return a;
    }else{
        return GCD(b, a % b);
    }
}
// 求最小公倍数
int LCD(int a,int b){
    return a * b / GCD(a, b);
}
int main(){
    int a,b;
    while (scanf("%d%d",&a,&b)!=EOF){
        printf("%d\n", LCD(a,b));
    }
    return 0;
}
```

