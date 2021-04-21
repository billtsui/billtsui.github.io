---
title: Java基础-注解机制详解
categories: [Java] 
date:   2021-04-19 14:19:38 +0800
tags: [Java核心技术]
---

**注解**是JDK 1.5引入的一个新特性，用于对代码进行说明，可以对包、类、接口、字段、方法参数、局部变量等进行注解。目前大部分框架(如Spring)都使用了注解简化代码并提高编码的效率，所以学习框架和设计必须要掌握注解。

### 注解基础

注解主要有以下四方面作用：

- 生成文档，通过代码里标识的元数据生成javadoc文档。
- 编译检查，通过代码里标识的元数据让编译器在编译期间进行检查验证。
- 编译时动态处理，编译时通过代码里标识的元数据动态处理，例如动态生成代码。
- 运行时动态处理，运行时通过代码里标识的元数据动态处理，例如使用反射注入实例。



- **标准注解**，常用的有 **@Override**、**@Deprecated**、**@SuppressWarnings**，分别用于标明重写某个方法、标明某个类或方法过时、标明要忽略的警告。用这些注解标明后，编译器就会进行检查。
- **元注解**，用来定义注解的注解，包括**@Retention**、**@Target**、**@Inherited**、**@Documented**等，**@Retention**用于标明注解的保留阶段，**@Target** 用于标明注解使用的范围，**@Inherited** 用于标明注解可以继承，**@Documented**用于标明是否生成javadoc文档。
- **自定义注解**，根据自己的需求定义注解，可以用元注解对自定义注解进行注解。



```java
public class Parent {
    public void test(){

    }
}

public class Child extends Parent {

    /**
     * 重写父类的test方法
     */
    @Override
    public void test() {
        super.test();
    }

    /**
     * 标明被弃用的方法
     */
    @Deprecated
    public void oldMethod(){

    }

    /**
     * 抑制警告
     */
    @SuppressWarnings("rawtypes")
    public List process(){
        List lst = new ArrayList();
        return lst;
    }
}
```



#### @Override

先看看这个注解的定义:

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}
```

从源码可以看出，这个注解用来修饰方法，并且这个注解只在源码级别有效，意味着在class文件中就不存在了。当然，这个注解是使用率非常高的注解，告诉编译器被修饰的方法是重写父类中同名方法。编译器对此检查，如果发现父类中不存在这个方法或是存在的方法签名不同，则会报错。



#### @Deprecated

这个注解源码如下：

```java
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, MODULE, PARAMETER, TYPE})
public @interface Deprecated {

    String since() default "";
    boolean forRemoval() default false;
}
```

从源码中可以看到，使用这个注解后，被会文档化。编译器处理完注解信息后，存储在class中，随后由VM读入，保留到runtime。能够修饰构造方法、属性、局部变量、方法、包、参数、类型。这个注解的作用是告诉编译器被修饰的程序元素已被“废弃”，不再建议用户使用。



#### @SuppressWarnings

```java
@Target({TYPE, FIELD, METHOD, PARAMETER, CONSTRUCTOR, LOCAL_VARIABLE, MODULE})
@Retention(RetentionPolicy.SOURCE)
public @interface SuppressWarnings {
    String[] value();
}
```

这个注解也是经常用的。能够修饰类型、属性、方法、参数、构造器、局部变量，编译成字节码时就抛弃掉，同时，值是一个String[]。它的作用是告诉编译器，忽略指定的警告信息，value可以取以下的值。

|           参数           |                        作用                        | 原描述                                                       |
| :----------------------: | :------------------------------------------------: | :----------------------------------------------------------- |
|           all            |                    抑制所有警告                    | to suppress all warnings                                     |
|          boxing          |            抑制装箱、拆箱操作时候的警告            | to suppress warnings relative to boxing/unboxing operations  |
|           cast           |                 抑制映射相关的警告                 | to suppress warnings relative to cast operations             |
|         dep-ann          |                 抑制启用注释的警告                 | to suppress warnings relative to deprecated annotation       |
|       deprecation        |                  抑制过期方法警告                  | to suppress warnings relative to deprecation                 |
|       fallthrough        |          抑制确在switch中缺失breaks的警告          | to suppress warnings relative to missing breaks in switch statements |
|         finally          |           抑制finally模块没有返回的警告            | to suppress warnings relative to finally block that don’t return |
|          hiding          |         抑制与隐藏变数的区域变数相关的警告         | to suppress warnings relative to locals that hide variable（） |
|    incomplete-switch     |              忽略没有完整的switch语句              | to suppress warnings relative to missing entries in a switch statement (enum case) |
|           nls            |                忽略非nls格式的字符                 | to suppress warnings relative to non-nls string literals     |
|           null           |                  忽略对null的操作                  | to suppress warnings relative to null analysis               |
|         rawtype          |        使用generics时忽略没有指定相应的类型        | to suppress warnings relative to un-specific types when using |
|       restriction        |        抑制与使用不建议或禁止参照相关的警告        | to suppress warnings relative to usage of discouraged or     |
|          serial          | 忽略在serializable类中没有声明serialVersionUID变量 | to suppress warnings relative to missing serialVersionUID field for a serializable class |
|      static-access       |            抑制不正确的静态访问方式警告            | to suppress warnings relative to incorrect static access     |
|     synthetic-access     |       抑制子类没有按最优方法访问内部类的警告       | to suppress warnings relative to unoptimized access from inner classes |
|        unchecked         |           抑制没有进行类型检查操作的警告           | to suppress warnings relative to unchecked operations        |
| unqualified-field-access |             抑制没有权限访问的域的警告             | to suppress warnings relative to field access unqualified    |
|          unused          |             抑制没被使用过的代码的警告             | to suppress warnings relative to unused code                 |



### 元注解

#### @Target

Target注解的作用是：描述注解的使用范围(即被修饰的注解可以用在什么地方)

Target注解用来说明那些被它所注解的注解可修饰的对象范围。在定义注解的时候使用@Target能够更加清晰的知道它能够被用来修饰哪些对象，它的取值范围定义在ElementType枚举中。

```java
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,

    /** Field declaration (includes enum constants) */
    FIELD,

    /** Method declaration */
    METHOD,

    /** Formal parameter declaration */
    PARAMETER,

    /** Constructor declaration */
    CONSTRUCTOR,

    /** Local variable declaration */
    LOCAL_VARIABLE,

    /** Annotation type declaration */
    ANNOTATION_TYPE,

    /** Package declaration */
    PACKAGE,

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE,

    /**
     * Module declaration.
     *
     * @since 9
     */
    MODULE
}
```



#### @Retention

Retention注解用来限定被它修饰的注解的声明周期。一共有三种策略，定义在RetentionPolicy枚举中。

```java
public enum RetentionPolicy {
    SOURCE, //被编译器忽略
    CLASS,  //编译期保留
    RUNTIME //运行时保留
}
```

为了理解这三种策略的注解有什么区别，我们做一个测试。分别自定义三个注解，然后观察测试类的字节码。

```java
@Retention(RetentionPolicy.SOURCE)
public @interface SourcePolicy {
}

@Retention(RetentionPolicy.CLASS)
public @interface ClassPolicy {
}

@Retention(RetentionPolicy.RUNTIME)
public @interface RuntimePolicy {
}


public class RetentionTest {

    @SourcePolicy
    public void sourcePolicy(){

    }

    @ClassPolicy
    public void classPolicy(){

    }

    @RuntimePolicy
    public void runtimePolicy(){

    }
}
```

使用javap -v RetentionTest后得到字节码，我们可以看到如下内容：

```java
{
  public person.billtsui.annotation.RetentionTest();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 7: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lperson/billtsui/annotation/RetentionTest;

  public void sourcePolicy();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 12: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       1     0  this   Lperson/billtsui/annotation/RetentionTest;

  public void classPolicy();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 17: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       1     0  this   Lperson/billtsui/annotation/RetentionTest;
    RuntimeInvisibleAnnotations:
      0: #14()
        person.billtsui.annotation.ClassPolicy

  public void runtimePolicy();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=0, locals=1, args_size=1
         0: return
      LineNumberTable:
        line 22: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       1     0  this   Lperson/billtsui/annotation/RetentionTest;
    RuntimeVisibleAnnotations:
      0: #17()
        person.billtsui.annotation.RuntimePolicy
}
```

其中，runtimePolicy()方法下有 **RuntimeVisibleAnnotations**记录了**person.billtsui.annotation.RuntimePolicy**注解。classPolicy()方法下有**RuntimeInvisibleAnnotations**记录了**person.billtsui.annotation.ClassPolicy**注解。而编译器并没有记录下sourcePolicy()方法的注解信息。



#### @Documented

Documented注解的作用是：在使用javadoc工具类生成文档时保留其注解信息。

以下代码在使用Javadoc可以生成**@TestDocumentd**注解信息。

```java
@Documented
public @interface TestDocumentd {
    String value() default "default";
}

@TestDocumentd(value = "myDoc")
public void testDoc(){

}
```



#### @Inherited

Inherited注解的作用是：声明当前注解具有继承性。如果某个类使用了被@Inherited修饰的注解，那么其子类会自动具有该注解。我们来测试一下这个注解：

```java
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD,ElementType.TYPE})
public @interface TestInheritedAnnotation
{
}

@TestInheritedAnnotation
public class Parent {
    public void test(){

    }
}

public class Child extends Parent {
    public static void main(String[] args) {
        Class childClass = Child.class;
        Annotation[] annotations = childClass.getAnnotations();
        for (Annotation annotation : annotations) {
            System.out.println(annotation.toString());
        }
    }
}
```

输出：

```java
@person.billtsui.annotation.TestInheritedAnnotation()
```



### 反射获取注解内容

定义注解以后，如何获取注解中的内容呢？我们可以通过使用反射，来获取。注意：只有被定义为RUNTIME的注解，才能在运行时可见，当class文件被装载时，保存在class中的Annotation才会被虚拟机读取。



反射包java.lang.reflect下的AnnotatedElement接口是所有程序元素(Class、Method和Constructor)的父接口，所以程序通过反射获取了某个类的AnnotatedElement对象之后，就可以调用该对象的方法来访问Annotation信息。我们看下接口内定义的方法：

```java
default boolean isAnnotationPresent(Class<? extends Annotation> annotationClass) {
    return getAnnotation(annotationClass) != null;
}

//判断该程序元素上是否包含指定类型的注解，存在则返回true，否则返回false。
```



```java
<T extends Annotation> T getAnnotation(Class<T> annotationClass);

//返回该程序元素上存在的、指定类型的注解，如果该类注解不存在，则返回null
```



```java
Annotation[] getAnnotations();

//返回该程序元素上存在的所有注解，如果没有注解，返回长度为0的数组
```



```java
default <T extends Annotation> T[] getAnnotationsByType(Class<T> annotationClass);

//返回该程序元素上的指定类型的注解数组。没有对应类型时，返回长度为0的数组。如果当前类找不到注解，且该注解为可继承的，则会去父类上检测是否有对应的注解。JDK 1.8以后有了默认实现。
```



```java
default <T extends Annotation> T getDeclaredAnnotation(Class<T> annotationClass);

//返回直接存在于此元素上的指定类型的注解。与此接口中的其他方法不同，该方法将忽略继承的注释。如果没有注释直接存在于此元素上，则返回null
```



```java
default <T extends Annotation> T[] getDeclaredAnnotationsByType(Class<T> annotationClass);

//返回直接直接存在于此元素上的所有注解。于接口中的其他方法不同，该方法将忽略继承的注解。
```



```java
Annotation[] getDeclaredAnnotations();

//返回直接存在于此元素上的所有注解，同时忽略继承的注解。如果没有注解直接存在于此元素上，则返回长度为零的一个数组。
```



### 自定义注解

#### 定义自己的注解

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyMethodAnnotation {
    String title() default "";
    String description() default "";
}
```



#### 使用注解

```java
public class TestMethodAnnotation {

    @Override
    @MyMethodAnnotation(title = "toStringMethod",description = "override toString method")
    public String toString() {
        return "Override toString method";
    }

    @Deprecated
    @MyMethodAnnotation(title = "old static method",description = "deprecated old static method")
    public static void oldMethod(){
        System.out.println("old method, don't use it.");
    }


    @SuppressWarnings({"unchecked","deprecation"})
    @MyMethodAnnotation(title = "test method",description = "suppress warning static method")
    public static void test(){
        List lst = new ArrayList();
        lst.add("abd");
        oldMethod();
    }
}
```



#### 使用反射接口获取注解信息

```java
public static void main(String[] args) {
    //获取所有methods
    Method[] methods = TestMethodAnnotation.class.getMethods();
    for (Method method : methods) {
        //方法上是否有MyMethodAnnotation注解
        if (method.isAnnotationPresent(MyMethodAnnotation.class)) {
            for (Annotation declaredAnnotation : method.getDeclaredAnnotations()) {
                System.out.println("Annotation in method " + method + ":" + declaredAnnotation);

                MyMethodAnnotation annotation = method.getAnnotation(MyMethodAnnotation.class);
                System.out.println("title:" + annotation.title() + " " + "description:" + annotation.description());
            }
        }
    }
}
```

输出如下：

```shell
Annotation in method public static void person.billtsui.annotation.TestMethodAnnotation.oldMethod():@java.lang.Deprecated(forRemoval=false, since="")
title:old static method description:deprecated old static method
Annotation in method public static void person.billtsui.annotation.TestMethodAnnotation.oldMethod():@person.billtsui.annotation.MyMethodAnnotation(title="old static method", description="deprecated old static method")
title:old static method description:deprecated old static method
Annotation in method public java.lang.String person.billtsui.annotation.TestMethodAnnotation.toString():@person.billtsui.annotation.MyMethodAnnotation(title="toStringMethod", description="override toString method")
title:toStringMethod description:override toString method
Annotation in method public static void person.billtsui.annotation.TestMethodAnnotation.test():@person.billtsui.annotation.MyMethodAnnotation(title="test method", description="suppress warning static method")
title:test method description:suppress warning static method
```





### 深入理解注解

#### 注解不支持继承

不能使用关键字extends或者implements来继承某个注解。但注解在编译后，编译器会自动继承 **java.lang.annotation.Annotation** 接口。

区别于注解的继承，被@inherited修饰的注解，如果父类使用了该注解，那么子类会自动继承该注解。

#### 注解实现的原理

我们自定义一个注解，然后通过IDEA的查看继承关系功能，观察到如下的继承规则。

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface MyMethodAnnotation {
    String title() default "";
    String description() default "";
}
```



![](https://github.com/billtsui/billtsui.github.io/blob/master/javabasic/annotation.png?raw=true)

可以看到@MyMethodAnnotation继承了Annotation接口。现在我们知道，注解是一个继承了Annotation接口的东西，那么注解究竟是接口，还是类，还是抽象类？我们查看了一下字节码文件，如下：

```java
public interface person.billtsui.annotation.MyMethodAnnotation extends java.lang.annotation.Annotation
    
{
  public abstract java.lang.String title();
    descriptor: ()Ljava/lang/String;
    flags: (0x0401) ACC_PUBLIC, ACC_ABSTRACT
    AnnotationDefault:
      default_value: s#7
        ""

  public abstract java.lang.String description();
    descriptor: ()Ljava/lang/String;
    flags: (0x0401) ACC_PUBLIC, ACC_ABSTRACT
    AnnotationDefault:
      default_value: s#7
        ""
}
```

我们知道了注解是一个接口，一个继承自Annotation的接口。 里面每一个属性，其实就是接口的一个抽象方法。那么新的问题来了，如果注解是接口，那么其何时实例化，怎么实例化？

我们debug一下这行代码:

```java
 MyMethodAnnotation annotation = method.getAnnotation(MyMethodAnnotation.class);
```

发现了这样的结果，很明显是一个代理对象，里面还有一个叫AnnotationInvocationHandler的类，这就是注解的代理逻辑封装。

