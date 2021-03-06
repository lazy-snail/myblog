---
title: Reflection
date: 2018-06-13 13:46:45
categories: java
tags: [java]
---
[toc]
# Reflection
反射，是 java 语言的特征之一，它允许运行中的 java 程序获取自身的信息，并且可以操作类对象或类的内部属性。反射的核心是 JVM 在运行时才动态加载类或调用方法/访问属性，它不需要事先（写代码的时候或编译期）知道运行对象是谁。
Oracle 官方解释：
Reflection enables Java code to discover information about the fields, methods and constructors of loaded classes, and to use reflected fields, methods, and constructors to operate on their underlying counterparts, within security restrictions.
The API accommodates applications that need access to either the public members of a target object (based on its runtime class) or the members declared by a given class. It also allows programs to suppress default reflective access control.
即，通过反射，可以在运行时获得程序或程序集中每一个类型的成员和成员的信息。一般，程序中对象的类型都是在编译期就确定下来的，而 Java 反射机制可以动态地创建对象并调用其属性，这样的对象的类型在编译期是未知的。所以可以通过反射机制直接创建对象，即使这个对象的类型在编译期是未知的。

## “反射”获取类对象
一般，用户使用某个类，应该从这个类定义出发，通过这个类产生实例化对象，这是类对象的使用方式；而“反射”，则是“相反”：通过对象找到这个类。具体就是，通过 instance.getClass() 方法，得到对象所在的“package.class”名称，getClass() 作为发起一切反射操作的开端（Object 的一个方法）：
```java
// 泛型都定义为 ?，返回值都是Object。
public final native Class<?> getClass();
```
除了使用 getClass() 方法（事实上一般不使用这个方法）获取类的实例化对象，还有：
class.class：
```java
Class<?> cls = Person.class; // 取得Class对象
System.out.println(cls.getName()); // 获取打印 package.class
```
以及，使用 Class 类内部定义的一个 static 方法：forName()
```java
Class<?> cls = Class.forName("package.class") ; // 取得Class对象
System.out.println(cls.getName()); // 获取打印 package.class
```

## 实例化
```java
public T newInstance() throws InstantiationException, IllegalAccessException {}
```

# 功能
反射框架主要提供以下功能（主要强调是在运行时，而非编译期间）：
* 在运行时判断任意一个对象所属的类；
* 在运行时构造任意一个类的对象；
* 在运行时判断任意一个类所具有的成员变量和方法（甚至包括 private 方法）；
* 在运行时调用任意一个对象的方法。

## 获取 class 对象
常用以下 3 种方式获取。
### 使用 class 类的 forName 静态方法
```java
public static Class<?> forName(String className)
```
JDBC 开发中，通常使用这种方式加载数据库驱动：
```java
 Class.forName(driver);
 ```

### 直接获取某一个对象的 class
```java
Class<?> klass = int.class;
Class<?> classInt = Integer.TYPE;
 ```

### 调用某个对象的 getClass() 方法
```java
StringBuilder str = new StringBuilder("123");
Class<?> klass = str.getClass();
```

## 判断是否为某个类的实例
一般，用 instanceof 关键字来判断是否为某个类的实例。同时也可以使用 isInstance() 方法判断是否为某个类的实例，这是一个 Native 方法：	
```java
myClass instanceof Object;
public native boolean isInstance(Object obj);
```

## 创建实例
通过反射生成对象。

## 获取构造器信息
通过 Class 类的 getConstructor 方法得到 Constructor 类的一个实例，而 Constructor 类有一个 newInstance 方法可以创建一个对象实例。
* getConstructor(Class… parameterTypes)	
获得指定的构造方法，注意只能获得 public 权限的构造方法，其他访问权限的获取不到；
* getDeclaredConstructor(Class… parameterTypes)	
获得指定的构造方法，可以获取到任何访问权限的构造方法；
* getConstructors()
获得所有 public 访问权限的构造方法；
* getDeclaredConstructors()
获得所有的构造方法，所有权限的构造方法。

## 获取方法
* getDeclaredMethods()
返回类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法；
* getDeclaredMethod()
返回一个特定的共有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法；
* getMethods()
返回某个类的所有公用（public）方法，包括其继承类的公用方法；
* getMethod()
返回一个特定的共有方法，其中第一个参数为方法名称，后面的参数为方法的参数对应 Class 的对象；

## 获取类的成员变量（字段）信息
主要是这几个方法，和获取方法类似：
* getFiled: 访问公有的成员变量；
* getDeclaredField：所有已声明的成员变量。但不能得到其父类的成员变量；
* getFileds 和 getDeclaredFields。

## 调用方法
从类中获取了一个方法后，就可以用 invoke() 来调用这个方法。invoke() 原型为:
```java
    public Object invoke(Object obj, Object... args)
        throws IllegalAccessException, IllegalArgumentException,
           InvocationTargetException {}
```

# 修改 private 属性/方法
通过以上方法可以获取实例的任何域/方法，然后通过相应的方法设置语言访问检查权限，即可修改属性值/调用方法
```java
// 修改域值，非 public 需要修改访问权限
field.setAccessible(true);
field.set(instanceOfClass, newVvalue);

// 调用方法(实例名后是参数列表的实参值)
// 非 public 需要修改访问权限
method.setAccessible(true);
method.invoke(instanceOfClass, newValue);
```

# 应用场景
java 中很重要的两个概念：注解和动态代理，都是使用反射技术有着很紧密的联系。

反射最重要的用途是开发各种通用框架。比如使用动态代理的 Spring、MyBatis 等框架都是配置化的，为了保证框架的通用性它们可以根据配置文件加载不同的类/对象，调用不同的方法，此时就必须使用反射——运行时动态加载需要加载的对象。



https://www.sczyh30.com/posts/Java/java-reflection-1
https://blog.csdn.net/gdutxiaoxu/article/details/68947735
https://www.cnblogs.com/rollenholt/archive/2011/09/02/2163758.html
https://www.zhihu.com/question/24304289/answer/38218810