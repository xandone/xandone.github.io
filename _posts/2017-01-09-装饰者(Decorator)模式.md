---
layout: post
title: 装饰者(Decorator)模式
date: 2017-01-09 18:17:03
tags: [编程]
toc:  true
---
装饰者(Decorator)模式  

装饰模式又名包装(Wrapper)模式。装饰模式是继承关系的一个替代方案。装饰模式是在不必改变原类文件和使用继承的情况下，动态的扩展一个对象的功能。它是通过创建一个包装对象，也就是装饰来包裹真实的对象。  
举一个生活中的例子，  
去奶茶店点奶茶，奶茶的种类有很多种，珍珠奶茶、绿茶、红茶...然后每一种茶都可以有不同的配置方案，比如常见的：去冰、加糖、打包等。  
那么，假设每种类型的奶茶都是一个实体类，可以在每种奶茶包装不同的属性（配置方案）。  

**1.创建父类**  
这里设定为接口，每种奶茶都实现了该接口  
```java
public interface Tea {
    void describe();
}
```
describe()函数简单的模拟该tea包含的属性。  

**2.定义每种奶茶的实体类**  
```java
/**
 * 珍珠奶茶
 */
public class PearlTea implements Tea {

    @Override
    public void describe() {
        System.out.print("珍珠奶茶\t");
    }
}

/**
 * 红茶
 */
public class BlackTea implements Tea {

    @Override
    public void describe() {
        System.out.print("红茶\t");
    }
}

/**
 * 绿茶
 */
public class GreenTea implements Tea {

    @Override
    public void describe() {
        System.out.print("绿茶\t");
    }
}
```

**3.定义装饰基础类**  
处理装饰者公共逻辑部分，在构造函数中，传入需要包装的Tea对象，对其进行“装饰操作”  
```java
/**
 * tea装饰基础类
 */
public abstract class CommenTeaDecorate implements Tea {

    protected Tea tea;

    public CommenTeaDecorate(Tea tea) {
        this.tea = tea;
    }

    @Override
    public void describe() {
        tea.describe();
    }
}
```

**4.装饰属性的实现类**  
```java
/**
 * 去冰
 */
public class NoIceTeaDecorate extends CommenTeaDecorate {

    public NoIceTeaDecorate(Tea tea) {
        super(tea);
    }

    @Override
    public void describe() {
        super.describe();
        System.out.print("去冰\t");
    }
}

/**
 * 加糖
 */
public class SugarTeaDecorate extends CommenTeaDecorate {
    
    public SugarTeaDecorate(Tea tea) {
        super(tea);
    }

    @Override
    public void describe() {
        super.describe();
        System.out.print("加糖\t");
    }

}

/**
 * 打包
 */
public class PackTeatDecorate extends CommenTeaDecorate {
    public PackTeatDecorate(Tea tea) {
        super(tea);
    }

    @Override
    public void describe() {
        super.describe();
        System.out.print("打包\t");
    }
}
```
5.调用方法  
例子1：分别包装三种类型的奶茶，  
```java
    PearlTea pearlTea = new PearlTea();
    GreenTea greenTea = new GreenTea();
    BlackTea blackTea = new BlackTea();

    NoIceTeaDecorate noIceTeaDecorate = new NoIceTeaDecorate(pearlTea);
    SugarTeaDecorate sugarTeaDecorate = new SugarTeaDecorate(greenTea);
    PackTeatDecorate packTeatDecorate = new PackTeatDecorate(blackTea);
    
    noIceTeaDecorate.describe();
    sugarTeaDecorate.describe();
    packTeatDecorate.describe();
```
打印：  
```
珍珠奶茶    去冰  
绿茶      加糖  
红茶      打包
```
例子2：针对一种奶茶多重属性包装，  
```java
    PearlTea pearlTea = new PearlTea();
    NoIceTeaDecorate noIceTeaDecorate = new NoIceTeaDecorate(pearlTea);
    SugarTeaDecorate sugarTeaDecorate = new SugarTeaDecorate(noIceTeaDecorate);
    PackTeatDecorate packTeatDecorate = new PackTeatDecorate(sugarTeaDecorate);
    
    sugarTeaDecorate.describe();
    packTeatDecorate.describe();
```
输出结果：  
```
珍珠奶茶    去冰  加糖  
珍珠奶茶    去冰  加糖  打包  
```
很明显，针对一种奶茶，可以包装很多种属性，相当于文前所说“动态的扩展一个对象的功能”。  