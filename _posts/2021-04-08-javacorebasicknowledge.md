---
title: Java基础-常用知识点
categories: [Java] 
date:   2021-04-08 15:42:38 +0800
tags: [Java核心技术]
---

这些知识点都是日常编码中经常使用的特性，熟记这些知识点不仅可以编写出质量更优的代码，还可以轻松面对一些基本的面试题。

#### 数据类型

- 包装类型
- 缓存池

##### 包装类型

Java中有8个基础数据类型(byte/short/int/long/float/double/char/boolean)，每一个基础数据类型都有对应的包装类型。基本类型与其对应的包装类型之间的赋值使用boxing与unboxing完成。

```java
Byte
Short
Integer
Long
Float
Double
Character
Boolean
```



##### 缓存池

Integer类型有一个默认的缓存池，缓存-128 ~ 127之间的数字。

```java
Integer x = Integer.valueOf(123);
Integer y = Integer.valueOf(123);
System.out.println(x == y)//true 因为都会使用缓存池中的对象，多次调用会取得同一个对象的引用
    
Integer z = new Integer(123);
Integer a = new Integer(123);
System.out.println(z == a);// false,new操作符会新分配存储空间
```

valueOf()方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容。

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];//cache是从-128作为起始元素的
    return new Integer(i);
}
```

在缓冲池范围内的基本类型自动装箱的话，编译器会默认调用valueOf()方法，因此多个Integer对象使用自动装箱来创建并且值相同，那么就会引用相同的对象。

```java
Integer a = 123;
Integer b = 123;
System.out.println(a == b) // true
```



Java的设计者为了提高String的性能，为其提供了一个字符串常量池。

```java
String str = "abc";
String str1 = new String("abc");
String str2 = str.intern();
```

- 直接使用双引号声明出来的String对象会直接存储在常量池中
- 使用构造函数声明的String对象是在堆中创建了一个新的对象
- 如果使用xxx.intern()创建字符串对象，则如果运行时常量池中已经存在一个等于此String内容的字符串，则返回常量池中该字符串的引用；如果没有，则在常量池中创建一个，并返回引用。



#### String

##### 概览

String类被声明为final，因此它不可以被继承。String类内部使用char[]存储数据，该数组也是final的，这意味着value数组一旦初始化之后就不能再引用其他数组。并且String类内部也没有改变value数组的方法，因此String是不可变的。

##### 不可变的好处

- 方便计算hash值
- String常量池可以提高性能
- 线程安全

#### String, StringBuffer and StringBuilder

1. 可变性
    - String不可变
    - StringBuffer和StringBuilder可变
2. 线程安全
    - String不可变，因此是线程安全的
    - StringBuilder不是线程安全的
    - StringBuffer是线程安全的，全类使用synchronized进行同步

#### 参数传递方式

Java的参数是以值传递的形势传入方法中，而不是引用传递。这个概念不是很好理解，尤其是针对先学C语言再学Java的程序员。

Java中将对象参数传入方法中，会复制其引用，然后传入。这个复制的引用和原始的引用指向同一个内存地址。对于基本数据类型，也是复制一个值然后再传入。请看下面的例子：

```java
class Untitled {
	public static void main(String[] args) {
		int a = 1;
		Untitled.changeValue(a);
        
        //这里打印出来的结果还是1，因为基础数据类型的值被复制一份传到了方法里。方法内部修改并不影响原始值。
		System.out.println(a);
	}
	public static void changeValue(int a){
		a = 10;
	}
}
```

```java
class Untitled {
	static class A{
		Integer age;
	}
	public static void main(String[] args) {
		A a = new A();
		a.age = 20;
		Untitled.changeValue(a);
        //这里打印出来是10
		System.out.println(a.age);
        
        //由于方法内部的引用是复制的，所以这里会打印false
		System.out.println(a == null);
	}
	public static void changeValue(A a){
		a.age = 10;
		a = null;
	}
}
```



#### switch

从JDK 7开始，可以在switch条件判断语句中使用String 对象。但switch不支持long，因为switch的设计初衷是对那些只有少数的几个值进行等值判断。

```java
switch(x){
   //char, byte, short, int, Character, Byte, Short, Integer, String, or an enum
}
```



#### 访问权限

|          | default | public | protected | private |
| -------- | :-----: | :----: | :-------: | :-----: |
| 同一个类 |    √    |   √    |     √     |    √    |
| 同一个包 |    √    |   √    |     √     |         |
| 子类     |         |   √    |     √     |         |
| 全局     |         |   √    |           |         |



#### 抽象类与接口

##### 抽象类

抽象类之所以叫抽象类，是因为它不是具体的，同理还有抽象方法。他们都使用abstract关键字声明，抽象类一般会包含抽象方法，抽象方法一定位于抽象类中。

抽象类因为抽象，所以不能被具体化，因此不能使用new 操作符声明它的实例，只能实例化继承它的子类。

##### 接口

接口定义行为。Java 8 之前，接口和抽象类一样，不能有具体的方法实现。从 Java 8开始，接口有默认的方法实现，默认方法需要使用default关键字声明。**为什么会有默认方法的出现？因为 Java 8之前如果接口新增了一个方法，那么所有实现它的类都需要实现这个方法，成本太高了。**



##### 抽象类与接口的比较

- 可以继承多个接口，但不能继承多个抽象类
- 接口的成员只能是public的，抽象类的成员可以有多种访问权限
- 接口的字段只能是static和final类型的，抽象类的字段没有这种限制
- 接口和抽象类都不能实例化，只能实例化继承它们的子类。



##### 使用场景

###### 接口

- 需要让不相关的类都实现一个方法，例如Compareable接口中的compareTo()方法
- 需要使用多重继承

###### 抽象类

- 需要在几个相关的类中共享代码
- 需要能控制继承来的成员的访问权限，而不是都为public
- 需要继承非静态和非常量字段

#### super

在使用父类的构造函数或者方法的时候，可以使用super关键字。

```java
class Parent {
	private int age;
	private String name;

    public Parent(int age, String name) {
        this.age = age;
        this.name = name;
    }
    

    public void getNameAndAge(){
        System.out.println("My name is:"+getName()+" and I'm "+getAge());
    }


    public int getAge() {
        return age;
    }


    public void setAge(int age) {
        this.age = age;
    }


    public String getName() {
        return name;
    }


    public void setName(String name) {
        this.name = name;
    }
}
```

```java
class Child extends Parent {
	
    public Child(int age, String name) {
        super(age,name);
    }
    
    public void getNameAndAge(){
        super.getNameAndAge();
    }
}
```

#### 重写与重载

##### 重写(override)

在继承中，指子类实现了一个与父类在方法声明上完全相同的一个方法。

既然是继承，那么必然需要满足里氏替换原则，所以重写也有限制：

- 子类方法的访问权限必须大于等于父类的方法
- 子类方法的返回类型必须是父类方法返回类型或其子类型

使用 @Override 注解，可以让编译器帮忙检查是否满足上面的两个限制条件。



##### 重载(overload)

overload有过载的意思，形象的说明一个方法存在多个同名的方法，但参数类型、个数、顺序至少有一个不同。如果返回值不同，那不构成重载。



#### Object类的一些方法

##### equals()

一个比较常用的方法，用来比较两个对象是否相等。这里的相等，指两个对象的引用是否相等。

```java
x.equals(x) //true
x.equals(y) == x.equals(y); //true 一致性
x.equals(y) == y.equals(x); //true

if (x.equals(y) && y.equals(z)){
    x.equals(z); //true;
}

x.equals(null); //false;
```

##### equals()与==

对于基本类型，==判断两个值是否相等，基本类型没有equals()方法。

对于引用类型，==判断两个变量是否引用同一个变量，equals()判断引用的对象是否等价。

不同的类会重写equals()方法，采用自己的逻辑来判断两个对象是否相等，例如String类。



#### hashCode()

hashCode()方法返回散列值，equals()方法用来判断两个对象是否等价。等价的两个对象散列值一定相等，但散列值相等的对象不一定是同一个对象。

在重写 equals() 方法时应当总是重写 hashCode() 方法，保证等价的两个对象散列值也相等。



#### 浅拷贝、深拷贝和clone()

##### 浅拷贝

被复制的对象的所有成员属性都有与原来的对象相同的值，而所有的对其他对象的引用仍然指向原来的对象。换言之，浅拷贝仅仅拷贝所考虑的对象，而不拷贝它所引用的对象。



##### 深拷贝

被复制对象的所有变量都含有与原来的对象相同的值，除去那些引用其他对象的变量。那些引用其他对象的变量将指向被拷贝过的新对象，而不是原有的那些被引用的对象。换言之，深拷贝将拷贝的对象引用的对象都复制一遍。



##### clone()

Object类中的 clone()方法被protected修饰符修饰。这也意味着如果要应用 clone()方法，必须继承Object类，在 Java中所有的类是缺省继承Object类的，也就不用关心这点了。然后重载 clone()方法。还有一点要考虑的是为了让其它类能调用这个clone类的clone()方法，重载之后要把 clone()方法的属性设置为 public。



Object.clone()方法返回一个Object对象。我们必须进行强制类型转换才能得到我们需要的类型。同时Object.clone()方法是一个对对象进行浅拷贝的方法，但是调用这个方法的类需要依赖去实现Cloneable接口,否则会抛出CloneNotSupportedException异常。



如果你要实现某个自定义类型的clone()方法，而且是深拷贝。那么你必须连带着一起实现它所引用的所有对象的clone()方法，同时还必须都是深拷贝。

```java
 protected native Object clone() throws CloneNotSupportedException;
```





#### final和static

##### final

声明一个变量为不可变，或者一个类为最终类，或者一个方法不能被override。

对于基本类型，final使数值不变。

对于引用类型，final保证其引用不变，但是引用类型对象的某个值是可变的。

```java
final A a = new A();
A a1 = new A();
a = a1;//错误
a.age = 30;//允许
```



##### static

静态变量、静态方法、静态代码块，这些都是一个类与生俱来的。

静态变量是绑定在类上的，所有类成员共享一份。



静态方法是类在加载时就存在了，所以它必须不能是抽象方法，并且也不能有非静态成员(静态方法不能调用非静态方法，不能有非静态变量，也不能使用this和super)。



静态代码块是在类初始化的时候执行一次。



使用静态变量和静态方法的时候，可以使用import static 关键字导包，但可读性就降低了。



#### **类完整的初始化顺序**

1. 父类(静态变量、静态代码块)
2. 子类(静态变量、静态代码块)
3. 父类(实例变量、普通语句块)
4. 父类(构造函数)
5. 子类(实例变量、普通语句块)
6. 子类(构造函数)