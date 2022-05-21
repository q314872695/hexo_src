---
title: 使用c++求质数、质因数个数和约数个数
abbrlink: d5d27c37
date: 2022-03-30 14:46:17
tags: 算法笔记
---

# 质数的判定

质数也称素数，是指只能被其自身和1整数的正整数。

那么如何判断一个数是否为质数呢？可以用所有小于该数的正整数去试着除该数，若能整数，则不是质数；若这些数都不能整除它，则该数是质数。

## 要求

给定一个数n，要求判定是否为质数（0，1和负数都不是质数），若是则输出yes，否则输出no

## 代码

```cpp
#include <cstdio>
#include <cmath>
using namespace std;

/*
 * 判断一个数是否是质数（素数）
 * */
bool Judge(int n){
    if(n<2){	//小于2的数都不是质数
        return false;
    }
    int bound = sqrt(n); // 代码优化，只需循环到根号n即可
    for (int i = 2; i <= bound; ++i) {
        if (n % i == 0) { 	//如果能被整除则不是质数，返回false
            return false;
        }
    }
    return true;
}
int main(){
    int n;
    while (scanf("%d",&n)!=EOF){
        if(Judge(n)){
            printf("yes\n");
        } else {
            printf("no\n");
        }
    }
    return 0;
}
```

# 质数筛法

知道如何判定一个质数后，那么如何找出0~100000之间的所有质数呢？依次枚举每个数，然后按照上文中的判断方法确定其是否为质数。这样做是可行的，但是时间复杂度太高了，这里有一种更好的方法，那就是质数筛法。

找到一个质数，它的所有倍数均标记为非质数；这样，当遍历到一个数时，若它未被任何小于它的数标记为非质数，则确定其为质数

例如：

有这样20个数（粗体为质数）：

- **0，1，2，3，4，5，6，7，8，9，10，11，12，13，14，15，16，17，18，19，20**  （初始全部为标记为质数）

- 0，1，**2，3，4，5，6，7，8，9，10，11，12，13，14，15，16，17，18，19，20** （0，1为非质数排除）
- 0，1，**2**，**3**，4，**5**，6，**7**，8，**9**，10，**11**，12，**13**，14，**15**，16，**17**，18，**19**，20（2是质数，它的所有倍数均为非质数）
- 0，1，**2**，**3**，4，**5**，6，**7**，8，9，10，**11**，12，**13**，14，15，16，**17**，18，**19**，20（3是质数，它的所有倍数均为非质数）

这样很快就算出来[0,20]之间的质数了

## 要求

输出第k个质数（k<10000），如k=3时，输出5

## 代码

```cpp
//
// Created by lxy on 2022/3/15.
//
#include <cstdio>
#include <vector>
using namespace std;

const int MAXN = 1e5 + 10;
vector<int> prime;  //保存质数
bool isPrime[MAXN]; //用来判断该数是否为质数

/*
 * 质数筛法
 * */
void Initial(){
    fill(isPrime, isPrime + MAXN, true);// 先假设所有数是质数
    isPrime[0] = false; //  0不是质数
    isPrime[1] = false; //  1不是质数
    for (int i = 2; i < MAXN; ++i) {
        if (!isPrime[i]) { // 不是质数就继续
            continue;
        }
        prime.push_back(i);//是质数就添加到向量中
        if (i > MAXN / i) { // 等价于i*i>MAXN预防i*i越界int，提前判断一下
            continue;
        }
        for (int j = i*i; j < MAXN; j+=i) { //质数的倍数肯定是非质数
            isPrime[j] = false;
        }

    }
}
int main(){
    Initial();
    int k;
    while (scanf("%d", &k) != EOF) {
        printf("%d\n", prime[k - 1]);
    }
    return 0;
}

```

# 质因子分解

什么是因子分解呢？

30=2 * 15

​    =5 * 6

​    =2 * 3 * 5	(所有因子都是质数时就称质因子分解)

通常使用短除法来求质因子：一个数不断的除以质数，直到等于1为止。例如：

120 / 2 = 60 (从第一个质数开始整除)

60 / 2 = 30

30 / 2 = 15

15 / 3 = 5	（此时2无法整除了，换下一个质数3）

5 / 5 = 1 	（此时3也无法整除了，换下一个质数5）

所以 $120=2^3 * 3 * 5$
$对一个数x分解质因数就是确定质数p_1,p_2,···p_n，使其满足x=p_1^{e_1}*p_2^{e_2}*···*p_n^{e_n}$


## 要求

求正整数N(1<N<10^9)的质因子的个数，相同的质因子需要重复计算。例如120=2 * 2 * 2 * 3 * 5，有5个质因子

## 代码

```cpp
#include <cstdio>
#include <vector>
using namespace std;

const int MAXN = 4e4;
vector<int> prime;
bool isPrime[MAXN];

/*
 * 质数筛法
 * */
void Initial(){
    fill(isPrime, isPrime + MAXN, true);// 先假设所有数是质数
    isPrime[0] = false;
    isPrime[1] = false;
    for (int i = 2; i < MAXN; ++i) {
        if (!isPrime[i]) { // 不是质数就继续
            continue;
        }
        prime.push_back(i);//是质数就添加到向量中
        if (i > MAXN / i) { // 预防i*i越界，提前判断一下
            continue;
        }
        for (int j = i*i; j < MAXN; j+=i) {
            isPrime[j] = false;
        }

    }
}
/*
 * 求质因子个数
 * */
int NumberOfPrimeFactors(int number){
    int answer = 0;
    for (int i = 0; i < prime.size(); ++i) {
        int factor = prime[i];
        if (number < factor) {
            break;
        }
        int exponent = 0;   //记录被除以的次数，即它的指数
        while (number % factor == 0) { //不停的除以这个质数，直到不能整除为止
            exponent++;
            number /= factor;
        }
        answer += exponent;
    }
    if (number > 1) {// 还有一个大于根号number质因子
        answer++;
    }
    return answer;
}
int main(){
    Initial();
    int number;
    while (scanf("%d", &number) != EOF) {
        printf("%d\n", NumberOfPrimeFactors(number));
    }
    return 0;
}
```

# 约数的个数

$质因数分解 x=p_1^{e_1}*p_2^{e_2}*···*p_n^{e_n}$
$数x约数的个数=(e_1+1)*(e_2+1)*···*(e_n+1)$


120=2^3 * 3 * 5

120约数的个数有（3+1） * （1+1）*（1+1）= 16

## 要求

输入一个整数n，输出它约数的个数。（1<n<10^9）

## 代码

```cpp

/*
 * 求约数的个数
 * */
#include <cstdio>
#include <vector>
using namespace std;

const int MAXN = 4e4;
vector<int> prime;
bool isPrime[MAXN];

/*
 * 质数筛法
 * */
void Initial(){
    fill(isPrime, isPrime + MAXN, true);// 先假设所有数是质数
    isPrime[0] = false;
    isPrime[1] = false;
    for (int i = 2; i < MAXN; ++i) {
        if (!isPrime[i]) { // 不是质数就继续
            continue;
        }
        prime.push_back(i);//是质数就添加到向量中
        if (i > MAXN / i) { // 预防i*i越界，提前判断一下
            continue;
        }
        for (int j = i*i; j < MAXN; j+=i) {
            isPrime[j] = false;
        }

    }
}
/*
 * 求约数个数
 * */
int NumberOfFactors(int number){
    int answer = 1;
    for (int i = 0; i < prime.size(); ++i) {
        int factor = prime[i];
        if (number < factor) {
            break;
        }
        int exponent = 0;   //记录被除以的次数，即它的指数
        while (number % factor == 0) { //不停的除以这个质数，知道除不动
            exponent++;
            number /= factor;
        }
        answer *= exponent + 1;
    }
    if (number > 1) {
        answer *= 2;
    }
    return answer;
}
int main(){
    Initial(); 
    int n;
    scanf("%d",&n);
    printf("%d\n", NumberOfFactors(n));
    return 0;
}
```

