---
title: Java集合框架-ArrayList源码解析
categories: [Java] 
date:   2021-04-25 16:18:38 +0800
tags: [Java集合框架]
---

ArrayList类实现了List接口，允许放入null元素，是顺序存放的一个容器，底层通过数组实现。对于容器来说，都会有一个容量，用来表示最多能保存多少个元素。当向容器中添加元素时，如果容量不足，容器会自动增大底层数据结构的大小。前面的博文已经说过，Java的泛型会做类型擦除，所以ArrayList底层是一个Object数组，以便能够容纳任何类型的对象。





### ArrayList的实现

#### 底层数据结构

ArrayList底层由一个Object[]数组保存数据。因为书数组，所以支持随机访问，get()、set()方法均能在常数时间内完成，同样的size()和isEmpty()也是常数时间。add()方法的时间消耗跟元素的插入位置有关。为了追求效率，ArrayList没有实现同步，所以不支持多线程操作。

```java
/**
 * The array buffer into which the elements of the ArrayList are stored.
 * The capacity of the ArrayList is the length of this array buffer. Any
 * empty ArrayList with elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
 * will be expanded to DEFAULT_CAPACITY when the first element is added.
*/
transient Object[] elementData; // non-private to simplify nested class access
```



#### 构造函数

```java
/**
 * Constructs an empty list with the specified initial capacity.
 *
 * @param  initialCapacity  the initial capacity of the list
 * @throws IllegalArgumentException if the specified initial capacity
 *         is negative
 */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {
        this.elementData = EMPTY_ELEMENTDATA;
    } else {
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}

/**
 * Constructs an empty list with an initial capacity of ten.
 */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
 * Constructs a list containing the elements of the specified
 * collection, in the order they are returned by the collection's
 * iterator.
 *
 * @param c the collection whose elements are to be placed into this list
 * @throws NullPointerException if the specified collection is null
 */
public ArrayList(Collection<? extends E> c) {
    Object[] a = c.toArray();
    if ((size = a.length) != 0) {
        if (c.getClass() == ArrayList.class) {
            elementData = a;
        } else {
            elementData = Arrays.copyOf(a, size, Object[].class);
        }
    } else {
        // replace with empty array.
        elementData = EMPTY_ELEMENTDATA;
    }
}
```



#### 自动扩容

每当向ArrayList添加元素时，都会检查元素个数是否会超出当前数组的长度。如果超出，数组将会进行扩容，以满足添加数据的需求。数组扩容通过一个public方法ensureCapacity(int minCapacity)实现。在添加大量元素前，我们也可以手动调用该方法来增加ArrayList的容量，以减少递增式再分配的次数。



数组进行扩容时，会将旧数组中的元素重新拷贝一份到新的数组中，每次数组容量的增长大约是原容量的1.5倍。这种操作的代价是很高的，因此在实际使用时，我们应该尽量避免数组容量的扩张。当我们可预知要保存的元素是多少时，要在构造ArrayList时，就指定容量，以免数组扩容的发生。或者根据实际需求，通过调用ensureCapacity方法来手动增加ArrayList实例的容量。



```java
/**
 * Increases the capacity of this {@code ArrayList} instance, if
 * necessary, to ensure that it can hold at least the number of elements
 * specified by the minimum capacity argument.
 *
 * @param minCapacity the desired minimum capacity
 */
public void ensureCapacity(int minCapacity) {
    if (minCapacity > elementData.length
        && !(elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA
             && minCapacity <= DEFAULT_CAPACITY)) {
        modCount++;
        grow(minCapacity);
    }
}

/**
 * The maximum size of array to allocate (unless necessary).
 * Some VMs reserve some header words in an array.
 * Attempts to allocate larger arrays may result in
 * OutOfMemoryError: Requested array size exceeds VM limit
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
 * Increases the capacity to ensure that it can hold at least the
 * number of elements specified by the minimum capacity argument.
 *
 * @param minCapacity the desired minimum capacity
 * @throws OutOfMemoryError if minCapacity is less than zero
 */
private Object[] grow(int minCapacity) {
    return elementData = Arrays.copyOf(elementData,
                                       newCapacity(minCapacity));
}

private Object[] grow() {
    return grow(size + 1);
}

/**
 * Returns a capacity at least as large as the given minimum capacity.
 * Returns the current capacity increased by 50% if that suffices.
 * Will not return a capacity greater than MAX_ARRAY_SIZE unless
 * the given minimum capacity is greater than MAX_ARRAY_SIZE.
 *
 * @param minCapacity the desired minimum capacity
 * @throws OutOfMemoryError if minCapacity is less than zero
 */
private int newCapacity(int minCapacity) {
    // overflow-conscious code
    int oldCapacity = elementData.length;
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity <= 0) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return minCapacity;
    }
    return (newCapacity - MAX_ARRAY_SIZE <= 0)
        ? newCapacity
        : hugeCapacity(minCapacity);
}

private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    return (minCapacity > MAX_ARRAY_SIZE)
        ? Integer.MAX_VALUE
        : MAX_ARRAY_SIZE;
}
```



#### 添加元素

ArrayList通过add(E e)和add(int index,E e)这两个方法添加单个元素。add(E e)方法向末尾添加一个元素，add(int index,E e)向指定的位置添加元素。添加元素会导致capacity不足，因此在添加元素之前，都需要进行空间检查，如果需要则自动扩容。

add(int index,E e)方法需要先对index ~ last element 整体向后移动一个位置，然后再完成插入操作，这表示index的数值越小，所需要移动的元素越多，消耗的时间也就越多。



```java
/**
 * Appends the specified element to the end of this list.
 *
 * @param e element to be appended to this list
 * @return {@code true} (as specified by {@link Collection#add})
 */
public boolean add(E e) {
    modCount++;
    add(e, elementData, size);
    return true;
}

/**
 * Inserts the specified element at the specified position in this
 * list. Shifts the element currently at that position (if any) and
 * any subsequent elements to the right (adds one to their indices).
 *
 * @param index index at which the specified element is to be inserted
 * @param element element to be inserted
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public void add(int index, E element) {
    rangeCheckForAdd(index);
    modCount++;
    final int s;
    Object[] elementData;
    if ((s = size) == (elementData = this.elementData).length)
        elementData = grow();
    System.arraycopy(elementData, index,
                     elementData, index + 1,
                     s - index);
    elementData[index] = element;
    size = s + 1;
}   
```



也可以使用addAll()和addAll(int index,Collection<? extends E> c)批量添加元素。跟add()方法类似，在插入之前也需要进行容量检查。

```java
 /**
  * Appends all of the elements in the specified collection to the end of
  * this list, in the order that they are returned by the
  * specified collection's Iterator.  The behavior of this operation is
  * undefined if the specified collection is modified while the operation
  * is in progress.  (This implies that the behavior of this call is
  * undefined if the specified collection is this list, and this
  * list is nonempty.)
  *
  * @param c collection containing elements to be added to this list
  * @return {@code true} if this list changed as a result of the call
  * @throws NullPointerException if the specified collection is null
  */
  public boolean addAll(Collection<? extends E> c) {
    Object[] a = c.toArray();
    modCount++;
    int numNew = a.length;
    if (numNew == 0)
        return false;
    Object[] elementData;
    final int s;
    if (numNew > (elementData = this.elementData).length - (s = size))
        elementData = grow(s + numNew);
    System.arraycopy(a, 0, elementData, s, numNew);
    size = s + numNew;
    return true;


/**
 * Inserts all of the elements in the specified collection into this
 * list, starting at the specified position.  Shifts the element
 * currently at that position (if any) and any subsequent elements to
 * the right (increases their indices).  The new elements will appear
 * in the list in the order that they are returned by the
 * specified collection's iterator.
 *
 * @param index index at which to insert the first element from the
 *              specified collection
 * @param c collection containing elements to be added to this list
 * @return {@code true} if this list changed as a result of the call
 * @throws IndexOutOfBoundsException {@inheritDoc}
 * @throws NullPointerException if the specified collection is null
 */
public boolean addAll(int index, Collection<? extends E> c) {
    rangeCheckForAdd(index)

    Object[] a = c.toArray();
    modCount++;
    int numNew = a.length;
    if (numNew == 0)
        return false;
    Object[] elementData;
    final int s;
    if (numNew > (elementData = this.elementData).length - (s = size))
        elementData = grow(s + numNew)

    int numMoved = s - index;
    if (numMoved > 0)
        System.arraycopy(elementData, index,
                         elementData, index + numNew,
                         numMoved);
    System.arraycopy(a, 0, elementData, index, numNew);
    size = s + numNew;
    return true;
}
```



#### 删除元素

remove(int index)和remove(Object c)都可以删除元素。前一个是删除对应位置的元素，后一个是删除第一个c.equals方法返回true的元素。删除操作是添加操作的逆过程，所以需要将删除点之后的元素向前移动一个位置。特别需要注意的是，为了让GC起作用，必须显示的为最后一个位置赋null值。

```java
/**
 * Removes the element at the specified position in this list.
 * Shifts any subsequent elements to the left (subtracts one from their
 * indices).
 *
 * @param index the index of the element to be removed
 * @return the element that was removed from the list
 * @throws IndexOutOfBoundsException {@inheritDoc}
 */
public E remove(int index) {
    Objects.checkIndex(index, size);
    final Object[] es = elementData;

    @SuppressWarnings("unchecked") E oldValue = (E) es[index];
    fastRemove(es, index);

    return oldValue;
}

/**
 * Removes the first occurrence of the specified element from this list,
 * if it is present.  If the list does not contain the element, it is
 * unchanged.  More formally, removes the element with the lowest index
 * {@code i} such that
 * {@code Objects.equals(o, get(i))}
 * (if such an element exists).  Returns {@code true} if this list
 * contained the specified element (or equivalently, if this list
 * changed as a result of the call).
 *
 * @param o element to be removed from this list, if present
 * @return {@code true} if this list contained the specified element
 */
public boolean remove(Object o) {
    final Object[] es = elementData;
    final int size = this.size;
    int i = 0;
    found: {
        if (o == null) {
            for (; i < size; i++)
                if (es[i] == null)
                    break found;
        } else {
            for (; i < size; i++)
                if (o.equals(es[i]))
                    break found;
        }
        return false;
    }
    fastRemove(es, i);
    return true;
}

/**
 * Private remove method that skips bounds checking and does not
 * return the value removed.
 */
private void fastRemove(Object[] es, int i) {
    modCount++;
    final int newSize;
    if ((newSize = size - 1) > i)
        System.arraycopy(es, i + 1, es, i, newSize - i);
    es[size = newSize] = null; //清除最后一个位置的引用，让GC起作用
}
```



#### 缩容

有扩容就有缩容，这里的缩容指的是将底层数组的容量调整为当前实际保存的元素个数，需要手动触发。

```java
/**
 * Trims the capacity of this {@code ArrayList} instance to be the
 * list's current size.  An application can use this operation to minimize
 * the storage of an {@code ArrayList} instance.
 */
public void trimToSize() {
    modCount++;
    if (size < elementData.length) {
        elementData = (size == 0)
          ? EMPTY_ELEMENTDATA
          : Arrays.copyOf(elementData, size);
    }
}
```



#### 查找

indexOf()，lastIndexOf()。indexOf()获取元素第一次出现的位置，lastIndexOf()获取元素最后一次出现的位置。

```java
/**
 * Returns the index of the first occurrence of the specified element
 * in this list, or -1 if this list does not contain the element.
 * More formally, returns the lowest index {@code i} such that
 * {@code Objects.equals(o, get(i))},
 * or -1 if there is no such index.
 */
public int indexOf(Object o) {
    return indexOfRange(o, 0, size);
}

int indexOfRange(Object o, int start, int end) {
    Object[] es = elementData;
    if (o == null) {
        for (int i = start; i < end; i++) {
            if (es[i] == null) {
                return i;
            }
        }
    } else {
        for (int i = start; i < end; i++) {
            if (o.equals(es[i])) {
                return i;
            }
        }
    }
    return -1;
}

/**
 * Returns the index of the last occurrence of the specified element
 * in this list, or -1 if this list does not contain the element.
 * More formally, returns the highest index {@code i} such that
 * {@code Objects.equals(o, get(i))},
 * or -1 if there is no such index.
 */
public int lastIndexOf(Object o) {
    return lastIndexOfRange(o, 0, size);
}

int lastIndexOfRange(Object o, int start, int end) {
    Object[] es = elementData;
    if (o == null) {
        for (int i = end - 1; i >= start; i--) {
            if (es[i] == null) {
                return i;
            }
        }
    } else {
        for (int i = end - 1; i >= start; i--) {
            if (o.equals(es[i])) {
                return i;
            }
        }
    }
    return -1;
}
```



### Fail-Fast机制

非线程安全的容器都会采用快速失效机制来检测错误。通常用来检测多线程操作的错误，还可以检测在循环的时候操作集合的错误。在操作元素的时候，modCount参数会++，面对并发的修改时，迭代器很快就会失败(modCount != expectedModCount)。s