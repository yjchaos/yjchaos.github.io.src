---
title: nodejs学习(一)-javascript基础
date: 2018-04-23 13:31:48
tags: nodejs
categories: 
---

# 语法

## 标识符

标识符的要求：
* 大小写敏感
* 使用下划线、字母、Unicode中的语言字符、美元符号开头，可以包含数字
* 不能使用保留字和关键字

## 严格模式

作用：消除ECMAScript老版本中的不合理、不严谨、不安全之处，提升效率，为ECMAScript的新版本做准备

范围：函数范围的严格模式，整个脚本的严格模式

Node.js中建议全部使用严格模式

严格模式的影响：
* 变量的严格声明（必须使用var声明）
``` js
//严格模式下的变量声明
var strTemp='string temp';
//会被定义成全局变量，严格模式下不允许
strTemp2='string temp2';
```
* 禁止动态绑定：不得使用with，eval作用域
* 不能删除变量，即不能 delete strUserDesc
``` js
//严格模式下的变量声明
var strTemp='string temp';
Object.getOwnPropertyDescriptor(global,'strTemp');
delete strTemp;
//会被定义成全局变量，严格模式下不允许
strTemp2='string temp2';
Object.getOwnPropertyDescriptor(global,'strTemp2');
delete strTemp1;
```
执行结果：
``` js
{ value: 'string temp',
  writable: true,
  enumerable: true,
  configurable: false }

false

{ value: 'string temp',
  writable: true,
  enumerable: true,
  configurable: true }

true
```
可以看到var申明变量的configurable为false所以它是无法删除的，而直接申明变量的configurable为true，所以它是可以删除的

* 不得重名：属性名、参数名
* 禁止使用八进制表示数字
* 保留字不得用做标识符

# 数据类型

## 变量

弱类型（变量可以表示任何类型的值）
性质：保存值的占位符
定义：使用var来定义变量，省略它会定义为全局变量（不推荐）

## 数据类型

* 简单数据类型：Undefined/Null/String/Number/Boolean

* 复杂数据类型：Object
<table>
    <tr>
        <td>数据类型</td>
        <td>要点</td>
    </tr>
    <tr>
        <td>Undefined</td>
        <td>
            Undefined类型只有一个值就是undefined，那么如何定义Undefined类型呢：
            <ul>  
                <li>值定义后没有初始化</li>  
                <li>值定义后，使用 undefined 来初始化</li>
                <li>函数没有返回值，会返回undefined</li>
                <li>函数的参数没有传值时，参数值为undefined</li> 
            </ul> 
            建议：变量定义时尽量给出一个初始值 
        </td>
    </tr>
    <tr>
        <td>
            Null
        </td>
        <td>
            Null与Undefined的区别
            <ul>  
                <li>Number(null) = 0, Number(undefined) = NaN</li>  
                <li>null：没有对象，undefined：缺少值</li>
            </ul> 
        </td>
    </tr>
    <tr>
        <td>Boolean</td>
        <td>
            <p>只有两个值：true和false</p>
            <p>其他类型均可以转成布尔值，使用Boolean()函数</p>
        </td>
    </tr>
    <tr>
        <td>Number</td>
        <td>
            <p>在JavaScript中，数值类型只有一种，内部表示为64位浮点数。所以在JavaScript中实际上是没有整数类型的。然而需要注意的是，数组索引、位操作符等使用的是基于32位的整数。</p>
            <p>三种进制(十进制：10，八进制：010，十六进制：0x10)</p>
            <p>E表示法</p>
            <p>Number.MAX_VALUE / MIN_VALUE / Infinity / -Infinity / isFinite()</p>
            <p>NaN / isNaN()</p>
            <p>Number()函数转成数值型</p>
            <p>parseInt()</p>
            <p>parseFloat()</p>
        </td>
    </tr>
    <tr>
        <td>String</td>
        <td>
            <p>单引号与双引号完全相同</p>
            <p>.toString() / String()</p>
        </td>
    </tr>
    <tr>
        <td>Object</td>
        <td>
            <p>成员：值或函数方法</p>
            <p>无序</p>
        </td>
    </tr>
    <tr>
        <td>typeof</td>
        <td>
            <p>是操作符，不是函数</p>
            <p>检查数据类型或函数</p>
            <p>检查结果：undefined / boolean / string / number / object / function</p>
        </td>
    </tr>
</table>

