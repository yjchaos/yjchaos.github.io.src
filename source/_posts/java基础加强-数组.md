---
title: java基础加强-数组
tags: java
categories: array
date: 2018-03-29 10:10:54
---


# 数组的定义和初始化

两种定义方式：
>int[] a;
或者
int a[];

定义并初始化：
>int[] a = new int[100];

# 数组的赋值

* 初始化同时赋值

``` java
    int[] smallPrimes = { 2, 3, 5, 7, 11, 13 };
```

* for循环赋值

``` java
    int[] a = new int[100];
    for (int i = 0; i < 100; i++)
        a[i] = i; // fills the array with numbers 0 to 99
```

* 用匿名数组赋值

初始化一个匿名数组
``` java
new int[] { 17, 19, 23, 29, 31, 37 }
```

再用这个匿名数组重新初始化其他变量
``` java
smallPrimes = new int[] { 17, 19, 23, 29, 31, 37 };
```

这种写法是下面这种写法的简化
``` java
int[] anonymous = { 17, 19, 23, 29, 31, 37 };
smallPrimes = anonymous;
```

# 数组的拷贝

* 引用拷贝

``` java
int[] luckyNumbers = smallPrimes;
luckyNumbers[5] = 12; // now smallPrimes[5] is also 12
```
<div align="left">
    <img src="/images/java基础加强-数组/拷贝数组变量.png" alt="container-magic"/>
</div>

这样两个变量指向同一段堆空间

* 值拷贝

``` java
luckyNumbers = Arrays.copyOf(luckyNumbers, 2 * luckyNumbers.length);
```
>1.第一个参数luckyNumbers是源数组
2.第二个参数是拷贝的长度
3.拷贝长度大于原数组长度时，多出来的元素被赋予该类型的默认值，如上例，后面的元素全是0
4.拷贝长度大于目标数组的长度，会新开辟一段堆空间，赋值，然后目标数组变量指向新建的堆

# 命令行参数

考虑如下代码：
``` java
public class Message
{
    public static void main(String[] args)
    {
        if (args.length == 0 || args[0].equals("-h"))
            System.out.print("Hello,");
        else if (args[0].equals("-g"))
            System.out.print("Goodbye,");
            // print the other command-line arguments
            for (int i = 1; i < args.length; i++)
                System.out.print(" " + args[i]);
            System.out.println("!");
    }
}
```

当你用命令行去执行时：
``` 
java Message -g cruel world
```

那么你的`args`数组将会是：
```
args[0]: "-g"
args[1]: "cruel"
args[2]: "world"
```

程序将会打印：
```
Goodbye, cruel world!
```

# 数组排序

* 自己实现算法对数组进行排序

* 利用`java.util.Arrays.sort()`对数组进行排序

数组数据类型 | 排序方法
---| ---
基本类型 | java为各种基本类型提供了快速排序，`sort()`方法会调用各个基本类型的快速排序方法将数组做升序排列
类类型 | 对于用户自定义的类，需要实现`Comparable`接口的`compareTo`方法

# 多维数组

我们只看二维数组

* 二维数组的定义和初始化

定义一个二维数组

``` java
double[][] balances;
```

定义并初始化

``` java
balances = new double[NYEARS][NRATES];
```

* 二维数组的赋值

初始化并赋值

``` java
int[][] magicSquare =
{
    {16, 3, 2, 13},
    {5, 10, 11, 8},
    {9, 6, 7, 12},
    {4, 15, 14, 1}
};
```

* for循环赋值

for loop
``` java
for (int i = 1; i < balances.length; i++)
{
    for (int j = 0; j < balances[i].length; j++)
    {
        balances[i][j] = i+j;
    }
}
```

foreach loop
``` java
for (double[] row : a)
    for (double value : row)
        value = newValue;
```

* 用匿名二维数组赋值

``` java
balances = new int[][] { 
    {1,2,3,4,5},
    {6,7,8,9,10},
 };
```

> tip:要打印多维数组所有的元素，可以使用`System.out.println(Arrays.deepToString(a));`,打印的结果如下`[[16, 3, 2, 13], [5, 10, 11, 8], [9, 6, 7, 12], [4, 15, 14, 1]]`

# 交错数组

为了解释交错数组，首先要知道java没有多维数组（suprise！！！），所谓多维数组其实是数组的数组，也就是一维数组中每个元素的值还是一个数组，那么这就是个二维数组。

交错数组和二维数组的差别在于数组的数组它的长度不一样

``` java
int[][] balances = new int[][]{
    {1, 2, 3},
    {4, 5},
};
```

><font color=red>所以说是二维数组还是交错数组决定权在我们，首先在定义和初始化的时候使用`int[2][]`或者`int[][]`而不定义死成`int[2][2]`,这样我们就可以等到者赋值的时候再来决定它是否是交错的</font>
