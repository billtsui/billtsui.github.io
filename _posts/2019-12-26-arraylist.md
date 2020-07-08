---
layout: post
title: ArrayList源码阅读
categories: 技术
date:   2019-12-26 21:29:38 +0800
tags: Java
---
### ArrayList简单介绍    
ArrayList源码的第一行注释就写到，这是一个可变长度的列表，并且继承于List接口,***可以添加null***。JDK集合类的作者Josh Bloch在1.2版本中加入的，1.5版本又引入了泛型。    

#### 1、构造方法    
```java    
public ArrayList(){
	//构造一个初始容量为 10(默认值) 的空列表。
}

public ArrayList(int initialCapacity){
	//造一个具有指定初始容量的空列表。
}

public ArrayList(Collection<? extends E> c){
	//构造一个包含指定 collection 的元素的列表，可能很少有人用过这个构造方法。这里用伪代码举个例子
	SubClazz extends Clazz
	Collection<SubClazz> subclazzCollection;
	ArrayList<Clazz> clazzList = new ArrayList<Class>(subclazzCollection);
}
```    

#### 2、核心方法    
```java
 public boolean add(E e) {
        // 增加一个元素，增加前会进行容量检查。如果容量不够，那么将进行扩容操作。
    }

    /**
     * 扩容方法
     **/
    private int newCapacity(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length; // 旧容量
        int newCapacity = oldCapacity + (oldCapacity >> 1); // 设置新容量= 旧容量 * 1.5
        if (newCapacity - minCapacity <= 0) { // 如果新容量小于最小容量
            if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) // 如果是空ArrayList
                return Math.max(DEFAULT_CAPACITY, minCapacity); // 就返回最小容量和默认容量中较大的一个
            if (minCapacity < 0) // overflow 最小容量超出下边界就返回异常
                throw new OutOfMemoryError();
            return minCapacity; // 返回最小容量
        }
        return (newCapacity - MAX_ARRAY_SIZE <= 0) // 如果新容量小于最大容量
                ? newCapacity
                : hugeCapacity(minCapacity); // 进行超大容量分配
    }

    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) // 如果最小容量大于最大容量
                ? Integer.MAX_VALUE // 就给Integer的最大值
                : MAX_ARRAY_SIZE; // 这个值 == Integer.MAX_VALUE - 8
    }

    /**
     * 在指定位置新增元素
     * 
     * @param index
     * @param element
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);// 合法性检查
        modCount++;
        final int s;
        Object[] elementData;
        if ((s = size) == (elementData = this.elementData).length)
            elementData = grow(); // 如果容量满了就扩容，这里直接size + 1
        System.arraycopy(elementData, index, elementData, index + 1, s - index);// 这里就是将原数组index位置后面的元素往后挪一个位置，空出index的位置
        elementData[index] = element;// 将元素插入到index的位置
        size = s + 1;
    }

    /**
     * Remove的核心方法
     * 
     * @param es
     * @param i
     */
    private void fastRemove(Object[] es, int i) {
        modCount++;
        final int newSize;
        if ((newSize = size - 1) > i)
            System.arraycopy(es, i + 1, es, i, newSize - i);// 将被删除位置之后的元素往前挪一个位置，然后将最后一个置为null等待垃圾回收
        es[size = newSize] = null;
    }
```



