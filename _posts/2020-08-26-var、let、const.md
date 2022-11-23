---
layout: post
title: var、let、const
date: 2020-08-26 23:50:19
tags: [编程]
toc:  true
---
ES6 之前是没有块级作用域的概念的，块作用域有 { } 包括，if语句和 for语句里面的{ }也属于块作用域。  

**1、var声明的变量会挂载在window对象上而let和const声明的变量不会**  

因此，var 的这一特性，可能会造成 window 全局变量的污染。  

**2、var 声明的变量存在变量提升，let 和 const 声明的变量不存在变量提升**  

**3、var 声明不存在块级作用域，let 和 const 声明存在块级作用域**  
```js
{
        var a = 1;
        let b = 2;
    }

    console.log(a);
    console.log(b);
```
错误信息：Uncaught ReferenceError: b is not defined.  
var声明的变量直接提升至全局变量  

**4、同一作用域下，var 可以重复声明同名变量，let 和 const 不能重复声明变量**  

**5、let 和 const 的暂时性死区（DTC）**  
```js
const a = 1;
    function test() {
        console(a);
        const a = 2;
    }
    test();
```
错误信息：Uncaught ReferenceError: Cannot access 'a' before initialization  
如果在当前块级作用域中使用了变量a，并且当前块级作用域中通过 let/const 声明了这个变量，那么，  
声明语句必须放在使用之前，也就是所谓的 DTC（暂时性死区）。  

**6、const：一旦声明必须赋值,并且声明后不能再修改。**  
这里的“不能再修改”指的是不能改变内存地址的引用，因此：  
1.const 声明基本数据类型，则无法被修改；  
2.const 声明引用数据类型（即“对象”），对象里的内容是可以被修改的。  
```js
function test() {
        const b = [];
        b[0] = 2;
        console.log('b[0]=' + b[0]);

        const a = 1;
        a = 2;
        console.log('a=' + a);
    }
```
结果：  
打印：b[0]=2  
异常：TypeError: Assignment to constant variable.  
不可给常量赋值  

**7、总结：**  
1）能不用 var的时候都不用var  
2）能用 const 的地方都用 const  
3）不能用 const 的地方，就用 let  