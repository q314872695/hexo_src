---
title: 使用c++实现矩阵相加、相乘、转置、求幂
tags: 算法笔记
abbrlink: aca33483
date: 2022-04-02 15:34:36
---

# 引言

对于矩阵我们应该并不陌生吧，在大学线性代数里面每天对它相加，相乘，转置等等，那么在计算机中该怎么实现呢？

首先，先定义一个矩阵的结构体。

```cpp
const int MAXN = 100; 	//矩阵最大的行和列
// 定义一个矩阵
struct Matrix{
    int row,col; //行，列
    int matrix[MAXN][MAXN];
    Matrix(){}
    Matrix(int r,int c):row(r),col(c){}	// 初始化时同时给行列赋值
};
```

# 矩阵相加

```cpp
Matrix Add(Matrix x,Matrix y){
    Matrix answer = Matrix(x.row, x.col); 	//定义一个新矩阵
    for (int i = 0; i < x.row; ++i) {
        for (int j = 0; j < y.col; ++j) {
            answer.matrix[i][j] = x.matrix[i][j] + y.matrix[i][j]; //遍历矩阵中的每个元素然后赋值相加
        }
    }
    return answer;
}
```

# 矩阵相乘

```cpp
Matrix Multiply(Matrix x,Matrix y){
    // 初始化的新矩阵的行是第一个矩阵的行，列的第二个矩阵的列
    Matrix answer = Matrix(x.row, y.col);
    for (int i = 0; i < x.row; ++i) {
        for (int j = 0; j < y.col; ++j) {
            answer.matrix[i][j] = 0;
            for (int k = 0; k < x.col; ++k) {
                answer.matrix[i][j] += x.matrix[i][k] * y.matrix[k][j];
            }
        }
    }
    return answer;
}
```

# 矩阵转置

```cpp
Matrix Transpose(Matrix x){
    Matrix answer = Matrix(x.col, x.row);//行列互换
    for (int i = 0; i < x.row; ++i) {
        for (int j = 0; j < x.col; ++j) {
            answer.matrix[i][j] = answer.matrix[j][i];
        }
    }
    return answer;
}
```

# 矩阵求幂

```cpp
Matrix QuickPower(Matrix x,int n){
    Matrix answer = Matrix(x.row, x.col);
    // 把answer初始化为单位阵，等价于求快速幂中的answer=1
    for (int i = 0; i < x.row; ++i) {
        for (int j = 0; j < x.col; ++j) {
            if (i == j) {
                answer.matrix[i][j] = 1;
            } else {
                answer.matrix[i][j] = 0;
            }
        }
    }
    // 类似于快速幂的算法
    while (n != 0) {
        if (n % 2 == 1) {
            answer = Multiply(answer, x);
        }
        n /= 2;
        x = Multiply(x, x);
    }
    return answer;

}
```

