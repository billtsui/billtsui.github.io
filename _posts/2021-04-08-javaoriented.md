---
title: Java面向对象
categories: [Java] 
date:   2021-04-08 10:56:38 +0800
tags: [Java]
---



面向对象的编程语言有三大特性:

- 封装
- 继承
- 多态   



#### 封装

利用抽象数据类型将数据和基于数据的操作封装在一起，使其构成一个对外不可见，不可分割的实体。数据被保护在抽象数据类型的内部，隐藏掉了内部的细节，只提供一个开放的接口使之与外部发生联系。使用方无需知道对象内部的细节，使用对象对外提供的接口来访问对象。



优点:

- 减少耦合：可以独立的开发、测试、使用、修改
- 减轻维护负担：独立的模块调试起来不影响其他模块
- 有效调节性能：可以通过分析确定哪些模块影响了系统的性能
- 提高软件可重用性
- 降低了构建大型系统的风险：即使整个软件不可重用，但是这些独立的模块有可能是可用的

以下Person类封装了name和age两个属性，外界可以通过set方法对属性的值进行设置。在setAge这里，用if来过滤掉不符合的年龄，外部使用者是不知道这个逻辑的。

```java
class person{
    private String name;
    private Integer age;
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
    public Integer getAge() {
        return age;
    }
    public void setAge(Integer age) {
        if(age < 18 && age >= 50){
            System.out.println("这个年龄段的人不适合工作");
        }else{
            this.age = age;
        }
    }
}
```

#### 继承

继承实现了一种 A  IS-A B的关系。例如Student和Person就是一种  IS-A 关系。因此Student可以继承自Person，从而获得Person的非private的属性和方法。

继承应该遵循里氏替换原则，子类对象必须能够替换掉所有父类对象。

Student 可以当做Person来使用，也就是说可以使用Person引用Student对象。父类引用指向子类对象成为向上转型。

```java
Person person = new Student();
```



#### 多态

多态分为编译时多态和运行时多态：

- 编译时多态主要指方法的重载
- 运行时多态指程序中定义的对象引用所指向的具体类型在运行期间才确定



运行时多态需要满足三个条件：

- 继承
- 覆盖(重写)
- 向上转型

下面的代码中，Animal类有两个子类：Duck和Lion，他们都覆盖了父类的run()方法，并且在main()方法中使用父类Animal来引用Duck和Lion对象。在Animal引用调用run()方法时，会执行实际引用对象所在类的run()方法，而不是Animal类的方法。

```java
public class Animal(){
    public void run(){
        System.out.println("Animal is running...");
    }
}

public class Duck extends Animal{
    public void run(){
        System.out.println("Duck is running...");
    }
}

public class Lion extends Animal{
    public void run(){
        System.out.println("Lion is running...");
    }
}

public class TestAnimal{
    public static void main(String[] args){
        List<Animal> animalList = new ArrayList();
        animalList.add(new Duck());
        animalList.add(new Lion());
        
        for(Animal animal : animalList){
            animal.run();
        }
    }
}
```





#### 抽象类

在面向对象的概念中，所有的对象都是通过类来描绘的。但是如果一个类中没有包含足够的信息来描绘一个具体的对象，这样的类就是抽象类。

抽象类除了不能实例化对象之外，类的其他功能依然存在。成员变量、成员方法和构造方法的访问方式和普通类一样。由于抽象类不能实例化对象，所以抽象类必须被继承才能被使用。



继承抽象类的子类，必须实现抽象类中的抽象方法。抽象方法顾名思义，只有方法声明，没有具体的实现。有抽象方法的类必须是抽象类。由于抽象类的特征，设计模式中有一个模板方法模式高度依赖抽象类。



#### 接口

接口和类非常相似，当他们属于不同的概念。类描述对象的属性和方法，接口则定义其实现的方法，也可以理解成类定义对象，接口定义行为。



一个类继承了某个接口，那么就一定要实现接口中定义的方法，抽象类除外。



接口与抽象类的相似点：

- 接口和抽象类都不能被实例化
- 接口和抽象类都可以有多个方法
- 实现接口或者继承抽象类的子类都必须实现各自的方法

接口与抽象类的不同点：

- 抽象类可以有普通方法，接口不能有普通方法(这里的普通方法不是指接口的默认方法)
- 接口内不能定义成员变量
- 接口不能有构造方法，抽象类可以有构造方法，但是抽象类的构造方法不用于构造对象，而是让其子类调用，从而完成抽象类的初始化。
- 接口有默认方法，使用default关键字修饰，让继承其的子类对象调用
- 一个类可以继承多个接口，一个类只能继承一个抽象类