---
title: Java基础-泛型机制详解
categories: [Java] 
date:   2021-04-15 15:58:38 +0800
tags: [Java核心技术]
---

泛型在Java中有很重要的地位，在面向对象编程及各种设计模式中有非常广泛的应用。Java在1.5之后加入了泛型的概念。泛型，即“**参数化类型**”。泛型的本质是为了参数化类型(将类型参数化传递)（在不创建新的类型的情况下，通过泛型指定的不同类型来控制形参具体限制的类型）。

### 为什么会有泛型？

假设你有一个需求，需要计算两个数字之和，这两个数字是int，float，double中的一种。在没有泛型的时候，你会这样写:

```java
public int sum(int a, int b) {
    return a + b;
}

public float sum(float a, float b) {
    return a + b;
}

public double sum(double a, double b) {
    return a + b;
}
```

如果没有泛型，要实现不同类型的加法，每种类型都需要重载一个方法，通过泛型，我们可以将参数的类型传入，复用一个方法。

```java
public static <T extends Number> double sum(T a, T b) {
    return a.doubleValue() + b.doubleValue();
}
```

**所以，泛型的本质是为了参数化类型。也就是说，在泛型使用过程中，操作的数据类型被指定为一个参数，这种参数类型可以用在类、接口和方法中，分别叫做泛型类、泛型接口、泛型方法。**



### 泛型类

#### 简单泛型类

```java
public class Generic<T> {
    private T generic;

    public T getGeneric() {
        return generic;
    }

    public void setGeneric(T generic) {
        this.generic = generic;
    }
}

public class TestGeneric {
    public static void main(String[] args) {
        Generic<String> generic = new Generic<String>();
        generic.setGeneric("AAAAA");
        //输出aaaaa
        System.out.println(generic.getGeneric().toLowerCase());
    }
}
```



#### 多元泛型类

```java
public class MultiGeneric<T, K, V> {
    private T title;
    private K key;
    private V value;

    public T getTitle() {
        return title;
    }

    public void setTitle(T title) {
        this.title = title;
    }

    public K getKey() {
        return key;
    }

    public void setKey(K key) {
        this.key = key;
    }

    public V getValue() {
        return value;
    }

    public void setValue(V value) {
        this.value = value;
    }
}

public class TestGeneric {
    public static void main(String[] args) {
        MultiGeneric<String, Integer, String> multiGeneric = new MultiGeneric<>();
        multiGeneric.setTitle("Title");
        multiGeneric.setKey(1);
        multiGeneric.setValue("Value");
        System.out.println(multiGeneric.getTitle() + ":" + multiGeneric.getKey() + ":" + multiGeneric.getValue());
    }
}
```



### 泛型接口

```java
public interface GenericInterface<T> {
    T sum(T a, T b);
}

public class GenericInterfaceClass implements GenericInterface<Integer> {
    @Override
    public Integer sum(Integer a, Integer b) {
        return a + b;
    }
}
```



### 泛型方法

#### 在普通类中声明一个泛型方法

```java
/*
 * 一定要在方法前使用<T>声明当前是一个泛型方法
 * Class<T> 类型的变量c，指明当前需要的类型，可以用来传入对应类型的对象
 */
public <T> T getObject(Class<T> c) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
    return c.getDeclaredConstructor().newInstance();
}
```



#### 在泛型类中声明一个泛型方法

```java
public T getObject(Class<T> c) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
    return c.getDeclaredConstructor().newInstance();
}
```



需要创建泛型类型的对象时，我们无法用new操作符。因为我们不知道具体的类型是什么，也不知道构造方法如何，因此没有办法去new一个对象。但可以使用上面的代码创建一个对象，也就是利用反射创建对象。



### 泛型约束

#### 类型约束

拿一个常用的泛型接口 **List\<T\>** 举例，当我们不指定类型的时候，List可以放多种类型的对象，就像下面这样：

```java
public class TestGeneric {
    public static void main(String[] args) {
        List test = new ArrayList();
        test.add(1111);
        test.add("aaa");
        test.add(1.23);
    }
}
```

这样的写法，List<T>会默认传入类型为Object，如果我们传入一个自定义类型，在类型强制转换的时候会导致异常。所以，在使用泛型容器的时候通常指定类型，约束只能传入该类型的对象。

```java
List<String> test = new ArrayList();
test.add("aaa");
test.add(1.23); // 编译器检查会直接报错，因为List<T>是String类型
        
```



#### 上限约束

泛型的上限约束指只能传入对应类型及其子类型或者是继承该接口的类型，保证类型转换正常。用 **\<T extends E\>** 表示 ，可以在类声明中直接限定，也可以在声明变量时限定。**注意这里不管是接口还是类，都用*extends*表示上限约束**。

###### 在类声明中限定上限

```java
//T类型必须实现Comparable接口
public class Generic<T extends Comparable> {
    private T generic;

    public T getGeneric() {
        return generic;
    }

    public void setGeneric(T generic) {
        this.generic = generic;
    }

    public T getObject(Class<T> c) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
        return c.getDeclaredConstructor().newInstance();
    }
}

//IDE直接提示错误 Type parameter 'java.util.Arrays' is not within its bound; should implement 'java.lang.Comparable'
Generic<Arrays> generic1 = new Generic<>();



//T必须是String或其子类
public class Generic<T extends String>{
    
}

//IDE直接提示错误 Type parameter 'java.lang.Double' is not within its bound; should extend 'java.lang.String'
Generic<Double> generic1 = new Generic<>();
```



##### 在形参中限定上限

```java
public class TestGeneric {
    public static void main(String[] args) {
        List<String> lst = new ArrayList<>();
        test(lst);
    }

    /**
     * 只允许Number类及其子类对象
     * @param list
     */
    public static void test(List<? extends Number> list){

    }
}
```



#### 下限约束

既然有上限约束，那肯定也会有下限约束，用 **\<T super E\>** 表示，只能传入E及其父类。

```java
public class TestGeneric {
    public static void main(String[] args) {
        List<String> lst = new ArrayList<>();
        test(lst);
    }

    /**
     * 只允许String类及其父类对象
     * @param list
     */
    public static void test(List<? super String> list){

    }
}
```



### 泛型上下限约束的奇特地方

#### 变量声明的时候能不能指定上下限呢？

我们看到，泛型的上限约束可以在Class级别添加，也可以在形参中添加，那么能不能在变量声明的时候添加呢？像下面这样:

```java
List<? extends Number> lst = new ArrayList();//这一行没有问题
lst.add(Integer.valueOf("1"));//这一行报错了

List<? super String> lst1 = new ArrayList();
lst1.add("lst");//看起来好像都行
lst1.add(new Object());//IDE直接报错
```



#### 类声明的时候能不能指定下限呢？

泛型的上限约束可以在Class级别添加，那么能不能在Class级别指定下限呢？像下面这样:

```java
public class Human<T super Number>{
    //很遗憾，IDE直接报错
}
```



### 深入理解泛型

首先需要说明的是：**Java中的泛型是*伪泛型***。

> Java的泛型是从JDK 1.5引入的，因此为了兼容之前的版本，Java泛型的实现采用了“伪泛型”的策略。即，Java在语法上支持泛型，但在编译阶段会进行所谓的“类型擦除”，将所有的泛型表示(尖括号中的内容)都替换为具体的类型(对应的原生态类型)，就像完全没有泛型一样。理解了类型擦除，一些看起来奇奇怪怪的问题就能迎刃而解了。

泛型的类型擦除原则

- 消除类型参数声明，即删除<>及其包围的部分。
- 根据类型参数的上下界推断并替换所有的类型参数为原生态类型：如果类型参数是无限制通配符或没有上下界限定，则替换为Object，如果存在上下界限定则根据里氏替换原则取类型参数的最左边限定类型(即父类)。

- 为了保证类型安全，必要时插入强制类型转换代码。
- 自动产生“桥接方法”以保证擦除类型后的代码仍具有泛型的“多态性”。



#### 类定义中的无限制类型擦除

当类定义中的类型参数没有任何限制时，在类型擦除中直接被替换为Object，即形如<T>和<?>的类型参数都被替换为Object。

```java
public class Info<T>{
    private T value;
    public T getValue(){
        return value;
    }
}

//类型擦除后

public class Object{
    private Object value;
    private Object getValue(){
        return value;
    }
}
```



#### 类定义中的有限制类型擦除

当类定义中的类型参数存在限制时，在类型擦除中替换为类型的上界。例如<T extends Number>和<? extends Number>的类型参数被替换成Number。

```java
public class Info<T extends Number>{
    private T value;
    public T getValue(){
        return value;
    }
}

//类型擦除后

public class Info{
    private Number value;
    public Number getValue(){
        return value;
    }
}
```



如果在类级别使用**\<T super String\>**来声明，会怎么样呢？答案无疑是否定的。类级别不能使用super来描述泛型。按照泛型的定义，这里可以使用String以及它的父类，String的父类只能是Object，所以如果要类型擦除，那么这里只能是Object。如果是多重继承呢？A继承B，B继承C，C继承D，那么我们要写**\<T super A\>**，类型擦除以后还必须是Object。**所以类级别不能使用super来描述泛型**。

```java
public class Info<T super String>{//语法错误，IDE会直接提示
    private T value;
    public T getValue(){
        return value;
    }
}
```



#### 方法定义中的类型擦除

方法声明中的无限制类型擦除和类中的无限制类型擦除一样，都会将<T>转为Object。

同样，泛型方法中也不能使用super来描述泛型，原因和上面说的一样，没有任何意义。



#### 声明带有上下界的泛型变量

讲完了类和方法的上下界声明，讲讲变量级别的上界声明。



##### ? extends XXX

```java
List<? extends Number> lst = new ArrayList();//可以声明
lst.add(Integer.valueOf("10")) //报错的
Number number = lst.get(0);//可以的
```

###### 读取

编译器知道lst里面存储的是Number类或者其子类，所以读取是OK。

###### 写入

因为编译器只知道lst是某个继承自Number类的子类，但是它不知道具体是哪一个类，有可能是Integer，有可能是Double，也有可能是Float。为了类型安全，编译器阻止向extends描述的泛型添加对象。



##### ? super XXX

```java
List<? super Number> lst = new ArrayList();
Object object = lst.get(0);
lst.add(Integer.valueOf("10"));
lst.add(Double.parseDouble("12.3"));
```

这里可能有混淆的地方，为什么? super Number，但是可以添加Integer和Double类型？super只是限定T的类型下界，那么T的子类是完全符合条件的。

###### 读取

lst的下界是Number，所以有可能是Integer，也有可能是Double。所以这里用Object对象来获取是OK的。

###### 写入

Number的子类，和Number都可以写入。



#### 类型擦除的证明

1. 比较不同泛型类型的class是否相等
2. 通过反射添加其他类型的元素



ArrayList<T>是无上下界的泛型，所以这里会擦除成Object。

```java
public class TestGeneric {
    public static void main(String[] args) {

        ArrayList<String> list = new ArrayList<String>();
        list.add("abc");

        ArrayList<Integer> list1 = new ArrayList<Integer>();
        list1.add(123);

        System.out.println(list.getClass() == list1.getClass()); // true
    }
}
```



```java
public class TestGeneric {
    public static void main(String[] args) throws NoSuchMethodException, InvocationTargetException, IllegalAccessException {

        ArrayList<String> list1 = new ArrayList<String>();
        list1.add("abc");

        list1.getClass().getDeclaredMethod("add",Object.class).invoke(list1,"111");

        list1.forEach(e -> System.out.println(e));
    }
}
```



#### 类型擦除和多态

现在有这样一个泛型类

```java
public class Parent<T> {
    public T value;

    public T getValue() {
        return value;
    }

    public void setValue(T t) {
        this.value = t;
    }
}
```



然后我们有一个子类继承它

```java
public class Child extends Parent<String> {
    @Override
    public String getValue() {
        return super.getValue();
    }

    @Override
    public void setValue(String s) {
        super.setValue(s);
    }
}
```

在这个子类中，我们设定父类的泛型为Parent\<String\>。在子类中，我们重写了父类的两个方法，按照自然的理解，将父类的泛型限定为String，那么父类里面的两个方法参数都为String类型。但是我们有讲到类型擦除，所以编译后父类的类型全部变为了原始类型Object。

```java
public class person.billtsui.generic.Parent<T> {
  public T value;

  public person.billtsui.generic.Parent();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method java/lang/Object."<init>":()V
       4: return

  public T getValue();
    Code:
       0: aload_0
       1: getfield      #2                  // Field value:Ljava/lang/Object;
       4: areturn

  public void setValue(T);
    Code:
       0: aload_0
       1: aload_1
       2: putfield      #2                  // Field value:Ljava/lang/Object;
       5: return
}
```

既然父类已经类型擦除了，那么子类为什么还能重写父类的两个方法？并且返回值还是String？我们的本意是子类进行重写，实现多态，可是类型擦除以后，就没办法实现多态了啊。看起来类型擦除和多态之间有了冲突。但是，我们也确确实实实现了重写，奥秘就在JVM采用了一种特殊的机制：**桥接模式**。

我们看看Child的字节码，结果如下：

```java
public class person.billtsui.generic.Child extends person.billtsui.generic.Parent<java.lang.String> {
  public person.billtsui.generic.Child();
    Code:
       0: aload_0
       1: invokespecial #1                  // Method person/billtsui/generic/Parent."<init>":()V
       4: return

  public java.lang.String getValue();
    Code:
       0: aload_0
       1: invokespecial #2                  // Method person/billtsui/generic/Parent.getValue:()Ljava/lang/Object;
       4: checkcast     #3                  // class java/lang/String
       7: areturn

  public void setValue(java.lang.String);
    Code:
       0: aload_0
       1: aload_1
       2: invokespecial #4                  // Method person/billtsui/generic/Parent.setValue:(Ljava/lang/Object;)V
       5: return

  public void setValue(java.lang.Object);
    Code:
       0: aload_0
       1: aload_1
       2: checkcast     #3                  // class java/lang/String
       5: invokevirtual #5                  // Method setValue:(Ljava/lang/String;)V
       8: return

  public java.lang.Object getValue();
    Code:
       0: aload_0
       1: invokevirtual #6                  // Method getValue:()Ljava/lang/String;
       4: areturn
}
```

我们发现Child的字节码有4个方法，但我们的源代码里只有2个方法。编译器在最后生成了桥接方法，可以看到桥接方法的参数都是Object。也就是说，子类中真正覆盖父类两个方法的，是我们看不到的桥接方法。java源码中的@Override只是一种假象，桥接方法的内部实现，就是去调用我们自己重写的两个方法。



同时，我们发现Child字节码中有一些奇怪的现象，getValue()方法有两个版本存在。一个返回Object，一个返回String。如果我们自己写这样的两个方法，是根本无法通过检查的，但是虚拟机却可以这么干，因为虚拟机通过参数类型和返回类型确定一个方法，所以编译器为了实现泛型的多态允许自己做这个看起来不合法的事情。



#### 泛型类型不能被实例化

```java
T t = new T();
```

这样的写法是通不过检查的。因为Java在编译器没法确定泛型参数化类型，此外，由于类型擦除，T变成了Object，如果可以new T()，那么就变成了new Object()，毫无意义。如果确实想实例化一个泛型，只能用反射的方法。

```java
public <T> T getObject(Class<T> c) throws NoSuchMethodException, IllegalAccessException, InvocationTargetException, InstantiationException {
    return c.getDeclaredConstructor().newInstance();
}
```



#### 异常中不能使用泛型

不能使用泛型扩展Throwable。

```java
public class Problem<T> extends Exception{
    //不会通过编译
}
```

假设可以通过编译，那么试想一下下面的代码:

```java
try {

} catch (Problem<Integer> ex) {

} catch (Problem<Double> ex1) {

}
```

这样的代码毫无意义，因为类型擦除的存在，最后Integer和Double都被擦除成了Object。

同样，也不能在catch中捕获泛型类型的异常。试想一下下面的代码：

```java
public static <T extends Throwable> void doWork(Class<T> t){
    try {

    } catch(T e) { //编译错误

    } catch(IndexOutOfBounds e) {

    }                         
}
```

根据异常的捕获原则，子类异常应该在父类异常之前捕获。因为类型擦除，导致T被编译成了Throwable，这样就违背了上述原则。



#### 如何获取泛型类型的参数类型

既然有类型擦除，那么如何获取泛型类型的原始类型呢？可以通过反射 **java.lang.reflect.Type** 获取。

**java.lang.reflect.Type** 是Java中所有类型的接口，代表Java中的所有类型。Type体系中类型的包括：数组类型(GenericArrayType)、参数化类型(ParameterizedType)、类型变量(TypeVariable)、通配符类型(WildcardType)、原始类型(Class), 以上这些类型都实现Type接口。

```java
public class TestGeneric {
    public static void main(String[] args) {
        //注意这里一定要用匿名内部类的方式
        Generic<String> genericType = new Generic<>(){};
        Type actualTypeArgument = ((ParameterizedType) genericType.getClass().getGenericSuperclass()).getActualTypeArguments()[0];
        System.out.println(actualTypeArgument);
    }
}
```



