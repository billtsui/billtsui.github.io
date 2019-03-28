---
layout: post
title: 从计算 Fibonacci 数列说起
categories: 算法
date:   2019-03-26 22:43:38 +0800
tags: Fibonacci
---
### 初识斐波那契数列    
80后那批程序员，第一次把斐波那契数列和编程联系到一起，最大的可能就是谭浩强版C语言讲递归的那个地方。这么多年过去了，不得不说用这个例子讲递归对学习算法是很不好的。先给出斐波那契数列的定义。    

$$
f(n) =
\begin{cases}
f(n-1) + f(n-2)  & \text{$n$ >= 3} \\
1 & \text{$n$ = 2 or $n$ = 1}
\end{cases}
$$
      

用递归方式计算斐波那契数列的JAVA代码如下:    
```java    
class Fib {
	public static void main(String[] args) {
		System.out.println(Fib.fib(46));
	}
	
	public static int fib(int n){
		if(n == 1 || n == 2){
			return 1;
		}else{
			return fib(n-1) + fib(n-2);
		}
	}
}
```    
这里计算第46项的原因是第47项的值超过了int类型的最大值。第46项的值是1836311903，在我的i7-7820HQ上跑了5000ms，CodeRunner统计用了31M的内存，真是又慢又耗内存。    


### 斐波那契数列初级优化    
采用递归的方式计算斐波那契数列慢是有原因的。以计算F(5)为例子，    

$$    
f(5) = f(4) + f(3) \\
f(4) = f(3) + f(2) \\
f(3) = f(2) + f(1)
$$   
  

在计算f(4)的时候，我们需要递归计算f(3)，在计算f(5)的时候，我们需要递归计算f(3)，f(3)被计算了两次，我们完全可以计算一次f(3)然后保存起来，用的时候直接拿来就好了吗。于是就有了以下的代码:    
```java
class Fib {
	public static void main(String[] args) {
		System.out.println(Fib.fib(46));
	}
	
	public static int fib(int n){
		if(n == 1 || n == 2){
			return 1;
		}else{
			int[] array = new int[n];
			array[0] = 1;
			array[1] = 1;
			for(int i = 0; i < n-2 ; i++){
				array[i+2] = array[i] + array[i+1];
			}
			return array[n-1];
		}
	}
}
```    
这个版本的代码用了185ms就计算出了第48项的值，简直是飞快啊，太有成就感了。原因是，每一项只计算了一次，并且没有递归就没有调用栈的保存，不仅内存消耗少，而且速度也快。但是，这个方法为了优化时间，把每一项都存在了数组里，实际上我只需要第48项，前面47项都是多余的啊。

### 斐波那契数列中级优化

**于是上面版本的代码又有了如下的优化:**    
```java
class Fib {
	public static void main(String[] args) {
		System.out.println(Fib.fib(46));
	}
	
	public static int fib(int n){
		if(n == 1 || n == 2){
			return 1;
		}else{
			int prefix1 = 1;
			int prefix2 = 1;
			int current = 0;
			
			for(int i = 0; i < n-2; i++){
				current = prefix1 + prefix2;
				prefix1 = prefix2;
				prefix2 = current;
			}
			return current;
		}
	}
}
```    
每次计算当前项的时候，就拿出前面的两项相加，然后移动第n-1个值去第n-2个位置，current值去第n-1个位置，空出current进行下一次循环，这样降低了空间复杂度，用了常量个空间，达到了我们减小空间的目的。    

### 斐波那契数列终极优化    
对于聪明好学的程序员来说，一定会想一个问题，有没有什么通项公式，能直接算出来某一项的值。就像等差数列知道任一项和公差就能得到其他项呢？有的，高中数学知识，构造一个等比数列可以推导出斐波那契数列的通项公式：    

$$
a_n= {\frac 1 {\sqrt5}}[(\frac {1+{\sqrt5}}{2})^n - (\frac {1-{\sqrt5}}{2})^n]
$$    

这样优化，可以在常数的时间和空间内计算出斐波那契数列的任意项。所以，    
## 要想程序性能好，关键数学要学好!!!

