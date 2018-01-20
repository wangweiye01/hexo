---
title: java新特性讲义
date: 2018-01-20 22:03:18
tags:
---

# Predicate操作集合

Java 8为Collection集合新增一个removeIf(Predicate filter)方法，该方法将会批量删除符合filter条件的所有元素。该方法需要一个Predicate对象作为参数，Predicate也是函数式接口，因此可使用Lamada表达式作为参数。

```
Collection books = new HashSet();
books.add(new String("123"));
books.add(new String("1234"));
books.add(new String("12345"));
books.add(new String("123456"));
// 使用Lamada表达式过滤集合
books.removeIf(ele -> ((String)ele).length() < 3);
System.out.println(books);
```

上面程序中调用了Collection集合的removeIf()方法批量删除集合中符合条件的元素，程序中传入了一个Lamada表达式作为过滤条件。

Predicate就是一个函数式接口，可以把它当做C语言中函数指针来使用；

```
public class Test {

    public static void operate(Collection c, Predicate p) { // 满足谓词条件p的元素都打印出来
        for (Object ele: c) {
            if (p.test(ele)) {
                System.out.println(ele);
            }
        }
    }

    public static void main(String[] args) {
        Collection c = new ArrayList();
        for (int i = 0; i < 10; i++) { // 加入0 ~ 9的字符串
            c.add(String.valueOf(i));
        }

        operate(c, ele -> Integer.valueOf((String)ele) > 3); // 大于3的打印出来
    }
}
```
