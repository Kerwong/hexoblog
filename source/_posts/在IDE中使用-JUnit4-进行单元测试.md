---
title: 在IDE中使用 JUnit4 进行单元测试
date: 2016-10-17 00:43:39
tags:
- JUnit
- Java
---
## Eclipse 下的 JUnit 使用
### 新建项目JUnitTest，创建Calculator类
编写Calculator类
为了方便测试，在Calculator类 中添加一些 Bugs
```java
package com;
public class Calculator {
    private static int result; // 静态变量，用于存储运行结果
    public void add(int n) {
        result = result + n;
    }
    public void substract(int n) {
        result = result - 1; // Bug: 正确的应该是 result =result-n
    }
    public void multiply(int n) {
    } // 此方法尚未写好
    public void divide(int n) {
        result = result / n;
    }
    public void square(int n) {
        result = n * n;
    }
    public void squareRoot(int n) {
        for (;;)
            ; // Bug : 死循环
    }
    public void clear() { // 将结果清零
        result = 0;
    }
    public int getResult() {
        return result;
    }
}
```
### 将JUnit4单元测试包引入这个项目
在JUnitTest 项目上，单击右键，选择 Properties （属性）

在弹出的属性窗口中，首先在左边选择“Java Build Path ”，然后到右上选择“Libraries ”标签，之后在最右边点击“Add Library… ”按钮，如下图所示：

