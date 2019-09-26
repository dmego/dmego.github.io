---
title: 决策模型(一)：不确定型决策法
date: 2018-11-4 20:26
author: Dmego
categories:
- 技术 
tags:
- Operations Research
mathjax: true
---

<!--more -->

<center><img src="decision-making-under-uncertainty/supplyimg.png" width="200px" /></center>

## 前言

所谓的不确定型的决策是指决策者对环境情况一无所知。这时决策者是根据自己的主观倾向进行决策，由决策者的主观态度的不同基本可分为四种准则：悲观主义决策准则、乐观主义决策准则、等可能性准则、最小机会损失决策准则。下面将以一个例子来说明这几种决策准则。

> 设某工厂是按批生产某产品并按批销售，每件产品的成本为30元，批发价为每件35元，若每月生产的产品当月销售不完，则每件损失1元。工厂每投产一批是10件，最大月生产能力是40件，决策者可选者的方案可以是 0 件、10 件、20 件、30 件、40 件。假设决者对其产品的需求情况一无所知。问该决策者该如何决策？

要想解决上诉问题，必须先知道决策矩阵。从问题中我们知道决策者可选的决策方案有五种，这是他们的策略集合，记做 {S<sub>i</sub>}，i = 1，2，···，5。经过我们分析，可断定将发生五种销售情况：销售 0 件、10 件、20 件、30 件、40 件，但是不知道他们发生的概率。这就是事件集合。记做{E<sub>j</sub>>}，j=1，2，···，5。而对于每个 ”策略—事件“ 对都可以计算出相应的收益值或损失值。例如单选择月产量为 20 件时，销售为 10 件。这时收益值为：

> 10 x (35 - 30) - 1 x (20 - 10) = 40 元

因此可以将每一个 ”策略—事件“ 对对应的收益值或损失值求出，记做 a<sub>i</sub><sub>j</sub>，将这些数据汇总在一个矩阵中，如下所示：

| （策略\事件）          | E<sub>1</sub> = 0 | E<sub>2</sub> = 10 | E<sub>3</sub> = 20 | E<sub>4</sub> = 30 | E<sub>5</sub> = 40 |
| ---------------------- | ----------------- | ------------------ | ------------------ | ------------------ | ------------------ |
| **S<sub>1</sub> = 0**  | 0                 | 0                  | 0                  | 0                  | 0                  |
| **S<sub>2</sub> = 10** | -10               | 50                 | 50                 | 50                 | 50                 |
| **S<sub>3</sub> = 20** | -20               | 40                 | 100                | 100                | 100                |
| **S<sub>4</sub> = 30**  | -30               | 30                 | 90                 | 150                | 150                |
| **S<sub>5</sub> = 40** | -40               | 20                 | 80                 | 140                | 200                |


这就是决策矩阵，根据决策矩阵中元素所示的含义不同，可称为收益矩阵，损失矩阵，风险矩阵，后悔矩阵等。



## 悲观主义（max min）决策准则

##### 定义

悲观主义决策又被称为保守主义决策准则，他分析各种最坏的可能结果，然后从中选择最好的，以它对应的策略为决策策略，用符合表示为 max min 决策准则。

##### 计算步骤

在收益矩阵中先从各策略所对应的结果中选出最小值，将他们至于表的最右列，然后从此列中选出最大值，以他对应的策略为决策者应选的决策策略。

##### 计算公式

> S<sup>*</sup><sub>k</sub> $\rightarrow$ max min (a<sub>i</sub><sub>j</sub> )

##### 计算结果

| （策略\事件）          | E<sub>1</sub> = 0 | E<sub>2</sub> = 10 | E<sub>3</sub> = 20 | E<sub>4</sub> = 30 | E<sub>5</sub> = 40 | min                    |
| ---------------------- | ----------------- | ------------------ | ------------------ | ------------------ | ------------------ | ---------------------- |
| **S<sub>1</sub> = 0**  | 0                 | 0                  | 0                  | 0                  | 0                  | 0 $\longleftarrow$ max |
| **S<sub>2</sub> = 10** | -10               | 50                 | 50                 | 50                 | 50                 | - 10                   |
| **S<sub>3</sub> = 20** | -20               | 40                 | 100                | 100                | 100                | - 20                   |
| **S<sub>4</sub> = 30**  | -30               | 30                 | 90                 | 150                | 150                | - 30                   |
| **S<sub>5</sub> = 40** | -40               | 20                 | 80                 | 140                | 200                | - 40                   |

根据 max min 决策准则有

> max (0 , - 10 , - 20, - 30 , - 40) = 0

对应的决策策略为 S<sub>1</sub>，为决策者选择的策略。在本例中为 “什么也不生产”，这个结论似乎很荒谬，但是在实际生产中表示先看一看，以后再做决定。

##### 计算代码

```java
/**
 * 悲观主义决策
 * @matrix 决策矩阵
 * @row 决策矩阵行数
 * @col 决策矩阵列数
 */
public static void maxMin(double[][] matrix, int row, int col){
    double[] maxMar = new double[row];
    for (int i = 0; i < row; i++) {
        double min = matrix[i][0]; //让第一个最小
        for (int j = 1; j < col; j++) {
            if(matrix[i][j] < min){
                min = matrix[i][j];
            }
        }
        maxMar[i] = min;
    }
    System.out.println(Arrays.toString(maxMar));
    double max = maxMar[0];
    for (int i = 0; i < row; i++) {
        if(maxMar[i] > max){
            max = maxMar[i];
        }
    }
    System.out.println("悲观主义决策结果："+max);
}
```



## 乐观主义（max max）决策准则

##### 定义

持有乐观主义决策准则的决策者对待风险的态度与悲观主义者不同，他不会放过任何一个获得最好结果的机会。来争取好中之好的乐观态度来选择他的决策策略。

##### 计算步骤

决策者在分析收益矩阵各”策略—事件“对的结果中选出最大者，记在表的最右列。再从该列数值中选出最大者，以它对应的策略为决策策略。

##### 计算公式

> S<sup>*</sup><sub>k</sub>  $\rightarrow$ max max (a<sub>i</sub><sub>j</sub> )

##### 计算结果

| （策略\事件）          | E<sub>1</sub> = 0 | E<sub>2</sub> = 10 | E<sub>3</sub> = 20 | E<sub>4</sub> = 30 | E<sub>5</sub> = 40 | min                     |
| ---------------------- | ----------------- | ------------------ | ------------------ | ------------------ | ------------------ | ----------------------- |
| **S<sub>1</sub> = 0**  | 0                 | 0                  | 0                  | 0                  | 0                  | 0                       |
| **S<sub>2</sub> = 10** | -10               | 50                 | 50                 | 50                 | 50                 | 50                      |
| **S<sub>3</sub> = 20** | -20               | 40                 | 100                | 100                | 100                | 100                     |
| **S<sub>4</sub> = 30**  | -30               | 30                 | 90                 | 150                | 150                | 150                     |
| **S<sub>5</sub> = 40** | -40               | 20                 | 80                 | 140                | 200                | 200 $\longleftarrow$ max |

根据 max max 决策准则有

> max (0 , 50 , 100, 150 , 200) = 200

对应的决策策略为 S<sub>5</sub>，为决策者选择的策略。也就是选择每月生产 40 件。

##### 计算代码

```java
/**
 * 乐观主义决策
 * @matrix 决策矩阵
 * @row 决策矩阵行数
 * @col 决策矩阵列数
 */
public static void maxMax(double[][] matrix, int row, int col){
    double[] maxMar = new double[row];
    for (int i = 0; i < row; i++) {
        double max = matrix[i][0]; //让第一个最大
        for (int j = 1; j < 5; j++) {
            if(matrix[i][j] > max){
                max = matrix[i][j];
            }
        }
        maxMar[i] = max;
    }
    System.out.println(Arrays.toString(maxMar));
    double max = maxMar[0];
    for (int i = 0; i < row; i++) {
        if(maxMar[i] > max){
            max = maxMar[i];
        }
    }
    System.out.println("乐观主义决策结果："+max);
}
```



## 等可能性（Laplace）准则

##### 定义

等可能性（Laplace）准则是19世纪数学家 Laplace 提出的。该准则认为所以事件发生的概率是相等的。也就是每一事件发生的概率都是 1 / 事件数。 

##### 计算步骤

决策者先计算各策略的收益期望值，然后在所有这些期望值中选择最大者。以它对应的策略为决策策略。

##### 计算公式

> S<sup>*</sup><sub>k</sub>  $\rightarrow$ max { E ( S<sub>i</sub> ) }

##### 计算结果
| （策略\事件）          | E<sub>1</sub> = 0 | E<sub>2</sub> = 10 | E<sub>3</sub> = 20 | E<sub>4</sub> = 30 | E<sub>5</sub> = 40 | E (S<sub>i</sub>) = $\sum_{j} pa_{ij}$ |
| ---------------------- | ----------------- | ------------------ | ------------------ | ------------------ | ------------------ | ---------------------------------------- |
| **S<sub>1</sub> = 0**  | 0                 | 0                  | 0                  | 0                  | 0                  | 0                                        |
| **S<sub>2</sub> = 10** | -10               | 50                 | 50                 | 50                 | 50                 | 38                                       |
| **S<sub>3</sub> = 20** | -20               | 40                 | 100                | 100                | 100                | 64                                       |
| **S<sub>4</sub> = 30**  | -30               | 30                 | 90                 | 150                | 150                | 78                                       |
| **S<sub>5</sub> = 40** | -40               | 20                 | 80                 | 140                | 200                | 80 $\longleftarrow$ max                  |

在本例中 P = $\frac{1}{5}$，期望值

>  max { E ( S<sub>i</sub> ) } = max {0, 38, 64, 78, 80 } = 80

对应的策略 S<sub>5</sub> 为决策策略  

##### 计算代码

```java
/**
 * 等概率准则决策
 * @matrix 决策矩阵
 * @row 决策矩阵行数
 * @col 决策矩阵列数
 */
public static void laplace(double[][] matrix, int row, int col){
    double[] maxMar = new double[row];
    for (int i = 0; i < row; i++) {
        double sum = 0;
        for (int j = 0; j < col; j++) {
            sum += matrix[i][j];
        }
        maxMar[i] = sum / col;
    }
    System.out.println(Arrays.toString(maxMar));
    double max = maxMar[0];
    for (int i = 0; i < row; i++) {
        if(maxMar[i] > max){
            max = maxMar[i];
        }
    }
    System.out.println("等概率准则决策结果："+max);
}    
```



## 最小机会损失决策准则

##### 定义

最小机会损失决策策略又被称为最小遗憾值决策准则或 Savage 决策准则。首先要将收益矩阵中的各元素变换为每一 “策略—事件” 对的机会损失值（遗憾值，后悔值）。其含义是：当某一事件发生后，由于决策者没有选用收益最大的策略，而形成的损失值。

##### 计算步骤

首先计算出当发生 k 事件后，各策略的收益最大值

> a<sub>i</sub><sub>k</sub> = max ( a<sub>i</sub><sub>k</sub> )

这时各策略的机会损失值为

>  $a'_{ik}$ = { max ( a<sub>i</sub><sub>k</sub> ) - a<sub>i</sub><sub>k</sub> }

从所有最大机会损失值中选取最小者，它对应的策略为决策策略。

##### 计算公式

> S<sup>*</sup><sub>k</sub>  $\rightarrow$ min  max $a'_{ik}$ 

##### 计算结果（该矩阵为损失矩阵）
| （策略\事件）          | E<sub>1</sub> = 0 | E<sub>2</sub> = 10 | E<sub>3</sub> = 20 | E<sub>4</sub> = 30 | E<sub>5</sub> = 40 | max                    |
| ---------------------- | ----------------- | :----------------- | ------------------ | ------------------ | ------------------ | ---------------------- |
| **S<sub>1</sub> = 0**  | 0                 | 50                 | 100                | 150                | 200                | 200                    |
| **S<sub>2</sub> = 10** | 10                | 0                  | 50                 | 100                | 150                | 150                    |
| **S<sub>3</sub> = 20** | 20                | 10                 | 0                  | 50                 | 100                | 100                    |
| **S<sub>4</sub> = 30**  | 30                | 20                 | 10                 | 0                  | 50                 | 50                     |
| **S<sub>5</sub> = 40** | 40                | 30                 | 20                 | 10                 | 0                  | 40 $\longleftarrow$ min |

决策结果为

>  min {200, 150, 100, 50, 40 } = 40

对应的策略 S<sub>5</sub> 为决策策略。在分析产品废品率时，应用本决策准则就比较方便。

##### 计算代码

```java
/**
 * 最小机会损失决策
 * @matrix 决策矩阵
 * @row 决策矩阵行数
 * @col 决策矩阵列数
 */
public static void savage(double[][] matrix, int row, int col){
    //损失矩阵
    double[][] loss = new double[row][col];
    for (int j = 0; j < col; j++) {
        double max = matrix[0][j]; //先定每一列的第一个最大
        for (int i = 1; i < row; i++) {
            if(matrix[i][j] > max){
                max = matrix[i][j];
            }
        }
        //损失矩阵中对应位置的值 = 决策矩阵中列最大值 - 决策矩阵中对应位置值
        for (int i = 0; i < row; i++) {
            loss[i][j] = max - matrix[i][j];
        }
    }
    //此时损失矩阵已经求出
    double[] maxMar = new double[row];
    for (int i = 0; i < row; i++) {
        double max = loss[i][0]; //让第一个最大
        for (int j = 1; j < col; j++) {
            if(loss[i][j] > max){
                max = loss[i][j];
            }
        }
        maxMar[i] = max;
    }
    System.out.println(Arrays.toString(maxMar));
    double min = maxMar[0];
    for (int i = 0; i < row; i++) {
        if(maxMar[i] < min){
            min = maxMar[i];
        }
    }
    System.out.println("最小机会损失决策结果："+min);
}
```



## 折中主义准则

##### 定义

当用 min max 决策准则或 max max 决策准则来处理问题时，有的决策者认为这样它极端了。于是提出把这两种决策准则给予综合，令 a 为乐观系数，且 0 < a < 1。

##### 计算步骤

设 $a^i_{max}$，$a^i_{min}$分别表示第 i策略可能得到最大收益值与最小收益值。根据下列关系式

> $H_i$= $a*a^i_{max}$ + $(1-a)a*a^i_{min}$

将计算出的 $H_i$ 记在矩阵表右侧，然后选择其中的最大者，对应的策略即为决策策略

##### 计算公式

> S<sup>*</sup><sub>k</sub>  $\rightarrow$ max { H<sub>i </sub> }

##### 计算结果（设 a = 1/3）
| （策略\事件）          | E<sub>1</sub> = 0 | E<sub>2</sub> = 10 | E<sub>3</sub> = 20 | E<sub>4</sub> = 30 | E<sub>5</sub> = 40 | $H_i$                   |
| ---------------------- | ----------------- | ------------------ | ------------------ | ------------------ | ------------------ | ----------------------- |
| **S<sub>1</sub> = 0**  | 0                 | 0                  | 0                  | 0                  | 0                  | 0                       |
| **S<sub>2</sub> = 10** | -10               | 50                 | 50                 | 50                 | 50                 | 10                      |
| **S<sub>3</sub> = 20** | -20               | 40                 | 100                | 100                | 100                | 20                      |
| **S<sub>4</sub> = 30**  | -30               | 30                 | 90                 | 150                | 150                | 30                      |
| **S<sub>5</sub> = 40** | -40               | 20                 | 80                 | 140                | 200                | 40 $\longleftarrow$ max |

决策结果为

>  min {0, 10, 20, 30, 40 } = 40

对应的策略 S<sub>5</sub> 为决策策略。

##### 计算代码

```java
/**
 * 折中主义决策
 * @matrix 决策矩阵
 * @row 决策矩阵行数
 * @col 决策矩阵列数
 * @a 乐观系数
 */
public static void eclecticism(double[][] matrix, int row, int col,double a){
    double[] H = new double[row];
    for (int i = 0; i < row; i++) {
        double max = matrix[i][0]; //让第一个最大
        double min = matrix[i][0]; //让第一个最小
        for (int j = 1; j < col; j++) {
            if(matrix[i][j] > max){
                max = matrix[i][j];
            }
            if(matrix[i][j] < min){
                min = matrix[i][j];
            }
        }
        //对运算结果四舍五入,保留两位小数
        H[i] = new BigDecimal(a*max + (1-a) * min).setScale(2, RoundingMode.UP).doubleValue();
    }
    System.out.println(Arrays.toString(H));
    double max = H[0];
    for (int i = 0; i < row; i++) {
        if(H[i] > max){
            max = H[i];
        }
    }
    System.out.println("折中主义准则决策结果："+max);
}
```



## 总结

在不确定型的决策中是因人、因地、因时选择决策准则的，但是在实际中当决策者面临不确定型决策问题时，首先是获取有关各事件发生的信息。使不确定型决策问题转化为风险决策。下篇将讨论风险决策。

## 参考

- [运筹学（第4版）本科版](http://www.tup.tsinghua.edu.cn/booksCenter/book_04835901.html)