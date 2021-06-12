---
title: Java集合框架-Stack、Queue和ArrayDeque源码解析
categories: [Java] 
date:   2021-05-09 13:46:38 +0800
tags: [Java集合框架]
---

JDK中有一个叫Stack的类，一个叫Queue的接口。当需要使用栈时，推荐使用ArrayDeque。ArrayDeque也实现了Queue接口，所以需要使用队列的时候也推荐优先使用ArrayDeque。但Stack和Queue作为JDK早期版本的针对栈和队列的默认实现，其学习价值还是很高的。



### Stack

栈是一种先入后出的数据结构，JDK的Stack类继承自Vector类。Vector使用object类型的数组作为底层数据结构，这点和ArrayList相同。Stack类很简短，只有不到200行代码，几个主要的方法见下表。

|     方法     |         说明         |
| :----------: | :------------------: |
| push(E item) |    向栈顶插入元素    |
|    pop()     |  获取并删除栈顶元素  |
|    peek()    | 获取但不删除栈顶元素 |



#### push(E item)

由于是继承自Vector，所以push方法很简单，只有短短的几行。

```java
public E push(E item) {
    addElement(item);

    return item;
}
```

调用Vector类的addElement方法添加一个元素，addElement方法是一个同步方法，可见是线程安全的。addElement方法会在底层数组的最后一个位置添加元素。

```java
public synchronized void addElement(E obj) {
    modCount++;
    add(obj, elementData, elementCount);
}
```

任何一个容器，在添加元素的时候，首先都要考虑容量的问题，Vector也不例外。和ArrayList不同的是，ArrayList默认1.5倍扩容，Vector默认2倍扩容。

```java
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    //这里没有>>1 而是直接oldCapacity + oldCapacity
    //capacityIncrement参数在构造函数里面指定，如果不指定默认就是0
    int newCapacity = oldCapacity + ((capacityIncrement > 0) ?
                                     capacityIncrement : oldCapacity);
    if (newCapacity - minCapacity <= 0) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return minCapacity;
    }
    return (newCapacity - MAX_ARRAY_SIZE <= 0)
        ? newCapacity
        : hugeCapacity(minCapacity);
}
```





#### pop()

pop()方法返回栈顶元素并将其删除，源码也很简单。其中peek()是获取栈顶元素的方法，removeElementAt删除栈顶元素。由于底层是用数组实现的，所以只需要删除最后一个元素即可，这里的传参是len-1，也就是最后一个元素的下标。

```java
public synchronized E pop() {
    E       obj;
    int     len = size();

    obj = peek();
    removeElementAt(len - 1);

    return obj;
}
```





#### peek()

peek()方法返回第一个元素，也很简洁。

```java
public synchronized E peek() {
    int     len = size();
    if (len == 0)
        throw new EmptyStackException();
    return elementAt(len - 1);
}
```





### Queue

队列是一种先进先出的数据结构，Queue接口定义了6个方法，说明可见下表:

|        方法        |                           说明                           |
| :----------------: | :------------------------------------------------------: |
|  boolean add(E e)  |           向队尾插入元素，超出队列容量抛出异常           |
| boolean offer(E e) |            向队尾插入元素，超出容量返回false             |
|     E remove()     | 移除队列中的第一个元素并返回该元素，如果队列为空抛出异常 |
|      E poll()      | 移除队列中的第一个元素并返回该元素，如果队列为空返回null |
|    E element()     |       获取队列中的第一个元素，如果队列为空抛出异常       |
|      E peek()      |       获取队列中的第一个元素，如果队列为空返回null       |





### Deque

双端队列是一种特殊的队列，即队列的头尾都可以出入元素，而普通的队列只能在尾部添加元素，在头部弹出元素。Deque接口在Queue接口的基础上，增加了双端操作的方法，可见下表。

|          方法           |                  说明                  |
| :---------------------: | :------------------------------------: |
|   void addFirst(E e)    |      向队首添加元素，失败抛出异常      |
| boolean offerFirst(E e) |     向队首添加元素，失败返回特殊值     |
|    void addLast(E e)    |      向队尾添加元素，失败抛出异常      |
| boolean offerLast(E e)  |     向队尾添加元素，失败返回特殊值     |
|      removeFirst()      |       移除队首元素，失败抛出异常       |
|      E pollFirst()      |  移除队首元素，如果是空队列则返回null  |
|     E removeLast()      |       移除队尾元素，失败抛出异常       |
|      E pollLast()       |  移除队尾元素，如果是空队列则返回null  |
|      E getFirst()       |  获取队首元素，但不移除。失败抛出异常  |
|      E peekFirst()      | 获取队首元素，但不移除，空队列返回null |
|       E getLast()       |  获取队尾元素，但不移除，失败抛出异常  |
|      E peekLast()       | 获取队尾元素，但不移除，空队列返回null |





### ArrayDeque

ArrayDeque和LinkedList是Deque的两个通用实现，由于ArrayDeque是JDK 1.6版本新增的，所以可以看成是Java更推荐使用ArrayDeque。

从名字可以看出，ArrayDeque底层是用数组实现的。为了实现双端队列的需求，该数组还必须是循环的，即**循环数组**。循环数组的任何一点都可能被看做起点或者终点。ArrayDeque是非线程安全的，同时不允许插入null元素。



ArrayDeque使用循环数组作为底层数据结构，循环数组的优点是资源利用率比较高。在频繁出队的情况下，普通数组无法利用第一个元素之前的存储空间，而循环数组可以充分利用各存储空间。

