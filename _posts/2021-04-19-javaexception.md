---
title: Java基础-异常机制详解
categories: [Java] 
date:   2021-04-20 15:24:38 +0800
tags: [Java核心技术]
---

编程语言的异常提供了一种识别以及响应错误的机制。异常机制可以使程序中出现错误的代码和正常的代码分离，提高程序的健壮性。



### Java异常的层次结构

#### Throwable

Throwable是Java中所有错误与异常的父类。

Throwable包含两个直接子类：Error和Exception，他们通常用于指示发生了异常情况。

Throwable同时还包含了其创建线程时线程执行栈的快照，可以使用printStackTrace()获取相关信息。

#### Error类及其子类

程序中无法处理的错误，表示运行引用程序中出现了严重的错误。此类错误一般表示JVM出现了问题。常见的有StackOverflowError、OutOfMemoryError等。

#### Exception类及其子类

程序可以捕获并且可以处理的异常。Exception又分为**运行时异常**和**非运行时异常**。

- 运行时异常： 

    是RuntimeException类及其子类的异常，如NullPointerException（空指针异常）、IndexOutOfBoundsException（下标越界异常）等。程序中可以捕获这些异常，也可以不处理。这些异常一般都是由程序逻辑错误引起的，在编写代码的时候应该从逻辑角度尽可能避免这类异常的发生。

- 非运行时异常：

    是RuntimeException之外的异常，一般都是必须进行处理的异常。如果不处理，程序就不能编译通过。如IOException、SQLException等以及用户自定义的Exception异常，一般情况下不自定义检查异常。



### 异常相关的关键字

- try：用于监听。将可能发生异常的代码放在try语句块之内，当try语句块内发生异常时，异常就被抛出。
- catch：用于捕获异常。catch用来捕获try语句块中发生的异常。
- finally：finally语句块总是会被执行。它主要用户回收在try块里打开的物理资源(数据库连接、网络连接和磁盘文件等)。**只有finally块执行完成以后，才会回来执行try或者catch快中的return或者throw语句，如果finally中使用了return或者throw等终止方法的语句，则直接停止**。
- throw：用于抛出异常。
- throws：用在方法签名中，用于声明该方法抛出的异常。



#### throws

如果方法中存在检查异常，但不对其捕获，那必须在方法上显示声明该异常，以便于告知调用者此方法有异常产生，需要进行处理。

```java
public static void metnod() throws IOException, FileNotFoundException{
    
}
```

注意：如果父类的方法没有声明异常，则子类继承后重写方法，也不能声明异常。

通常，应该捕获哪些知道如何处理的异常，将不知道如何处理的异常继续向上传递。传递异常可以在方法签名处使用throws关键字。

Throws抛出异常的规则：

- 如果是不可检查异常，即Error、RuntimeException或他们的子类，那么可以不用throws关键字来声明要抛出的异常，编译仍然能顺利通过，但在运行时会被系统抛出。
- 必须声明方法可抛出的任何可检查异常。即如果一个方法可能有可检查异常，要么用try-catch语句捕获，要么用throws子句将其抛出，否则会导致编译错误。
- 如果要重写一个方法，则不能声明与覆盖方法不同的异常。声明的任何异常必须是被覆盖方法所声明异常的同类或其子类。



#### throw

如果代码可能会引发某种异常，可以创建一个合适的异常类型并抛出它。

```java
public static double divide(int value){
    if(value == 0){
        throw new ArtthmeticException("参数不能为0");//抛出一个运行时异常
    }
    return 10.0 / value;
}
```



#### 自定义异常类

在业务系统中，如果需要自定义异常类，一般包含两个构造函数，一个无参构造函数和一个带有详细描述信息的构造函数。Throwable 的 toString 方法会打印这些详细信息，调试时很有用）, 比如上面用到的自定义MyException。

```java
public class MyException extends Exception{
    public MyException(){}
    public MyException(String msg){
        super(msg);
    }
}
```



#### 异常常见的处理方法

异常捕获处理的方法通常有：

- try-catch
- try-catch-finally
- try-finally
- try-with-resource

##### try-catch

在一个try-catch语句块中可以捕获多个异常类型，并对不同类型的异常做出不同的处理。

```java
public static void readFile(String filePath) {
    try {

    } catch (FileNotFoundException e1) {

    } catch (IOException e2) {

    }
}
```

同一个catch也可以捕获多种类型异常，用|隔开

```java
public static void readFile(String filePath) {
    try {

    } catch (FileNotFoundException | IOException e) {

    }
}
```



#### try-catch-finally

```java
try {
    //执行程序代码
} catch (Exception e) {
    //捕获异常并处理
} finally {
    //必须执行的代码
}
```

- 当try没有捕获到异常时：try语句块中的语句逐一执行，程序将跳过catch语句块，然后执行finally语句块和后面的代码。
- 当try捕获到异常，catch语句块里没有处理此异常：此时异常将会抛给JVM处理，finally语句块里的语句还是会被执行，但finally语句块后的语句不会被执行。
- 当try捕获到异常，catch语句块有处理此异常的情况：当执行到try块中的某条语句时发生了异常，程序将跳到catch语句块，并与catch语句块逐一匹配，找到对应的处理程序，其他的catch语句块将不会被执行，而try语句块中，出现异常之后的语句也不会被执行，catch语句块执行完成后，执行finally语句块里的语句，最后执行finally后的语句。



#### try-finally

**可以直接使用try-finally，省略catch**。

```java
//以Lock加锁为例，演示try-finally
ReentrantLock lock = new ReentrantLock();
try {
    //需要加锁的代码
} finally {
    lock.unlock(); //保证锁一定被释放
}
```

try-finally可以用在不需要捕获异常的代码，可以保证资源在使用后被关闭。例如IO流中执行完相应的操作后，关闭相应资源;使用Lock对象保证线程同步，通过finally块可以保证锁会被释放;数据库使用结束后，关闭相应的连接操作等。

finally遇见如下情况不会执行

- 在try代码中用了System.exit()退出程序
- finally语句块中发生了异常
- 程序所在线程死亡



#### try-with-resource

这个是在JDK 7中引入的，使用频率低，很容易被忽略。

我们可以使用finally释放相关资源，但如果finally中出现了异常，则相关的资源可能得不到释放。JDK 7 提供了一种更优雅的方式来实现资源的自动释放，自动释放的资源需要是实现了AutoCloseable接口的类。

```java
try (Scanner scanner = new Scanner(System.in)) {

} catch (Exception ex) {

}
```

try代码块退出时，会自动调用scanner.close()方法。和把scanner.close()方法放在finally代码块中不同的是，如果scanner.close()抛出异常，则会被抑制，抛出的仍为原始异常。被抑制的异常会由addSusppressed 方法添加到原来的异常，如果想要获取被抑制的异常列表，可以调用 getSuppressed 方法来获取。



### 异常实践

#### 只针对不正常的情况才使用异常

> 异常只应该被用于不正常的条件，他们永远不应该被用于正常的控制流。Java类库中定义的可以通过预检查方式规避的RuntimeException异常不应该通过catch的方式来处理，比如：NullPointerException，IndexOutOfBoundsException等等

举个例子：

```java
if(null != object){
    //正确
}

try{
    obj.method();
} catch (NullPointerException e){
    //不正确的
}
```

主要原因是，异常设计的初衷是用于不正常的情况，所以JVM没有对异常进行性能优化。所以，创建、抛出和捕获异常的开销是很昂贵的。



#### 在finally块中清理资源或者使用try-with-resource语句

只有try块中的语句正常执行，才会执行类似close()方法。一旦close()前有异常，则资源就无法释放了。所以应该再finally块或者使用try-with-resource语句清理资源。



#### 尽量使用标准的异常

主要是为了代码重用，顺带提几个常用的异常。

|              异常               |               使用场合               |
| :-----------------------------: | :----------------------------------: |
|    IllegalArgumentException     |            参数的值不合适            |
|      IllegalStateException      |           参数的状态不合适           |
|      NullPointerException       |                空引用                |
|    IndexOutOfBoundsException    |               下标越界               |
| ConcurrentModificationException | 在禁止并发修改的情况下检测到并发修改 |
|  UnsupportedOperationException  |       对象不支持客户请求的方法       |



#### 对异常进行文档说明

当方法升声明抛出异常时，也需要进行文档说明。目的是为了给调用者提供尽可能多的信息，从而可以更好地避免或处理异常。



#### 优先捕获最具体的异常

> 大多数IDE都可以实现这点，当你尝试首先捕获较不具体的异常时，他们会报告无法访问代码块。

下面的例子就是一个经典的例子。第一个catch块处理了所有的异常，第二个catch处理空引用的异常。但是NullPointerException是Exception的子类，所以无法被捕捉到。

```java
public static void method() {
    try {

    } catch (Exception e) {

    } catch (NullPointerException e1) {
        //这里编译器会提示异常已经被捕捉了
    }
}
```



#### 不要捕获Throwable类

如果你catch了Throwable类，那么Error也将被捕捉到。JVM的错误不应该由应用程序处理。典型的例子是OutOfMemoryError或者StackOverflowError。所以，最好不要捕获Throwable，除非你很确信自己能够处理错误。





**关于异常的最佳实践，《Effective Java》这本书有很好的讲解**。

### 深入理解异常

#### 异常处理机制

说到JVM处理异常的机制，就需要提到Exception Table，简称异常表。我们先看一个简单的异常处理代码：

```java
public class TestException {
    public static void method() {
        try {
            System.out.println("Hello World");
        } catch (Exception e) {

        }
    }
}
```

这段代码已经是简单的不能再简单了。下面我们反编译一下它的字节码。

```java
public person.billtsui.exception.TestException();
    descriptor: ()V
    flags: (0x0001) ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LineNumberTable:
        line 12: 0
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   Lperson/billtsui/exception/TestException;

  public static void method();
    descriptor: ()V
    flags: (0x0009) ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=1, args_size=0
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String Hello World
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: goto          12
        11: astore_0
        12: return
      Exception table:
         from    to  target type
             0     8    11   Class java/lang/Exception
      LineNumberTable:
        line 15: 0
        line 18: 8
        line 16: 11
        line 19: 12
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
      StackMapTable: number_of_entries = 2
        frame_type = 75 /* same_locals_1_stack_item */
          stack = [ class java/lang/Exception ]
        frame_type = 0 /* same */
```

终于在第26行看到了Exception Table。异常表中包含了异常处理着的信息，这些信息包含如下：

- **from** 可能发生异常的起始点
- **to** 可能发生异常的结束点
- **target** 上述from和to之前发生异常后的异常处理者的位置
- **type** 异常处理者处理的异常的类信息

JVM什么时候用异常表呢？答案是异常发生的时候，当一个异常发生时

1. JVM会在当前出现异常的方法中，查找异常表，是否有合适的处理者来处理
2. 如果当前方法异常表不为空，并且异常命中处理者的from 至 to 节点，并且type也匹配，则JVM调用位于target的调用者来处理
3. 如果当前方法的异常表无法处理，则向上查找刚刚调用该方法的调用处，并重复步骤1和2
4. 如果所有的方法栈都没有处理，则抛给当前的Thread，Thread则会终止
5. 如果当前的Thread为最后一个非守护线程，且未处理异常，则会导致JVM终止运行。

#### Retrun和Finally问题

```java
public static void main(String[] args) {
    try {
        //throw  new Exception();
        System.out.println("OK");
        return;
    } catch (Exception e) {
        System.out.println("Error");
        return;
    } finally {
        System.out.println("tryCatchReturn");
    }
}
```

无论try块或者是catch块内有没有return，finally都会执行。当有return时，finally会优先于return执行。具体内容可以查看字节码文件。