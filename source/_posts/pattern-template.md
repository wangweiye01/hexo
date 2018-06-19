---
title: 设计模式-模板方法模式
date: 2018-06-16 15:47:39
tags:
---

# 什么是模板方法模式

定义一个操作中算法的骨架，而将这些步骤延迟到子类中，模板方法使得子类可以不改变一个算法的结构即可重新定义该算法的某些特定步骤

好抽象的概念啊，文绉绉的东西就是不讨人喜欢，下面我用一个生活中常见的例子来举例说明吧

饮料自动售卖机，生产饮料的过程大致分为四个步骤

1. 烧水
2. 冲泡饮料
3. 把饮料倒入杯中
4. 加入调料

例如：
咖啡：烧开开水->加入咖啡粉冲泡->把饮料倒入杯中->加入少许糖
奶茶：烧开开水->加入奶茶粉冲泡->把饮料倒入杯中->加入珍珠

不难发现，饮料制作过程中的步骤中的1烧水、3把饮料倒入杯中是重复工作，制泡哪种饮料都一样，那么也就是重复工作，我们可以把它设定为通用性操作

我们只需要去关心步骤2和步骤4即可

由于制泡饮料的步骤就是这4步，所以我们可以把它抽象成一个"制作饮料模板"出来，下面就以上面这个例子，我用代码来说明

DrinkTemplate.java（模板类）

由于避免继承它的子类去修改整体制作架构，所以这个方法用了final修饰符来修饰

```
public abstract class DrinkTemplate {
    public final void drinkTemplate() {
        // 烧水
        boilWater();
        // 冲泡饮料
        brew();
        // 把饮料倒入杯中
        pourInCup();
        // 加调料
        addCondiments();
    }

    // 加调料
    protected abstract void addCondiments();

    protected void pourInCup() {
        System.out.println("把饮料倒入杯中");
    }

    // 冲泡饮料
    protected abstract void brew();

    protected void boilWater() {
        System.out.println("烧水中...");
    }
}
```

抽象类中的抽象方法必须重写，抽象类中的非抽象方法可以选择重写

MakeCoffee.java（制作咖啡）

```
public class MakeCoffee extends DrinkTemplate{
    @Override
    protected void addCondiments() {
        System.out.println("加糖...");
    }

    @Override
    protected void brew() {
        System.out.println("加入咖啡粉冲泡...");
    }
}
```


MakeOrange.java（制作橙汁）

```
public class MakeOrange extends DrinkTemplate{
    @Override
    protected void addCondiments() {
        System.out.println("加柠檬...");
    }

    @Override
    protected void brew() {
        System.out.println("加入橘子粉冲泡...");
    }

    protected void pourInCup() {
        System.out.println("慢慢倒入");
    }
}
```

这样只需要复写必须复写的方法，选择复写方法即可，大大提高了代码的复用性`
