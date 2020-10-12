---
title: Java高级特性-反射
date: 2020-10-12 08:53:51
tags:
---

![pexels-disha-sheta-3489514.jpg](https://p.130014.xyz/2020/10/12/pexels-disha-sheta-3489514.jpg)

# 定义

Java反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意方法和属性；这种动态获取信息以及动态调用对象方法的功能成为java语言的反射机制

# 反射涉及的类

## Class类

代表类的实体，在运行的Java应用程序中表示类和接口

### 相关方法

|  方法   | 用途  |
|  ----  | ----  |
| forName(String className)  | 根据类名返回类的对象 |
| getName()  | 获得类的完整路径名字 |
| newInstance() | 创建类的实例 |
| getPackage() | 获得类的名字 |
| getSimpleName() | 获得类的名字 |
| getField(String name) | 获得某个公有的属性对象 |
| getFields() | 获得所有公有的属性对象 |
| getDeclaredField(String name) | 获得某个属性对象 |
| getDeclaredFields() | 获得所有属性对象 |
| getConstructor(Class...<?> parameterTypes) | 获得该类中与参数类型匹配的公有构造方法 |
| getConstructors() | 获得该类的所有公有构造方法 |
| getDeclaredConstructors() | 获得该类所有构造方法 |
| getMethod(String name, Class...<?> parameterTypes) | 获得该类某个公有的方法 |
| getMethods() | 获得该类所有公有的方法 |
| getDeclaredMethod(String name, Class...<?> parameterTypes) | 获得该类某个方法 |
| getDeclaredMethods() | 获得该类所有方法 |
| isInstance(Object obj) | 如果obj是该类的实例则返回true |

## Field类

代表类的成员变量(属性)

### 相关方法

|  方法   | 用途  |
|  ----  | ----  |
| equals(Object obj)  | 属性与obj相等则返回true |
| get(Object obj) | 获得ojb中对应的属性值 |
| set(Object obj, Object value) | 设置obj中对应属性值 |

## Method类

代表类的方法

### 相关方法

|  方法   | 用途  |
|  ----  | ----  |
| invoke(Object obj, Object... args)  | 传递object对象及参数调用该对象对应的方法 |

## Constructor类

代表类的构造方法

### 相关方法

|  方法   | 用途  |
|  ----  | ----  |
| newInstance(Object... initargs)  | 根据传递的参数创建类的对象 |

# 示例

```
public class Person {
    private final static String classString = "com.wang.Person";

    private String name;

    private Integer age;

    public Person() {
    }

    private Person(String name, Integer age) {
        this.name = name;
        this.age = age;
    }

    private String getName() {
        return name;
    }

    // 创建实例
    public static void reflectNewInstance() {
        try {
            Class<?> aClass = Class.forName(classString);

            Object object = aClass.newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // 反射私有的构造方法
    public static void reflectPrivateConstructor() {
        try {
            Class<?> aClass = Class.forName(classString);
            Constructor<?> declaredConstructor = aClass.getDeclaredConstructor(String.class, Integer.class);
            declaredConstructor.setAccessible(true);
            Object object = declaredConstructor.newInstance("wangweiye", 23);
            System.out.println(object.toString());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // 反射私有属性
    public static void reflectPrivateField() {
        try {
            Class<?> aClass = Class.forName(classString);
            Object object = aClass.newInstance();

            Field field = aClass.getDeclaredField("classString");
            field.setAccessible(true);
            System.out.println(field.get(object));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    // 反射私有方法
    public static void reflectPrivateMethod() {
        try {
            Class<?> aClass = Class.forName(classString);
            Constructor<?> declaredConstructor = aClass.getDeclaredConstructor(String.class, Integer.class);
            Object lbj = declaredConstructor.newInstance("lbj", 35);

            Method method = aClass.getDeclaredMethod("getName");
            method.setAccessible(true);

            System.out.println(method.invoke(lbj).toString());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        reflectPrivateConstructor();

        reflectPrivateField();

        reflectPrivateMethod();
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```