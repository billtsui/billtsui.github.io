---
layout: post
title: 从计算 Fibonacci 数列说起
categories: 算法
date:   2019-03-26 22:43:38 +0800
tags: Array
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

### 斐波那契数列矩阵优化   
对于聪明好学的程序员来说，一定会想一个问题，斐波那契数列本质上是一个方程，那我们可不可以用线性代数的知识来解决这个问题呢?    

首先我们可以得到这样一个等式:    

$$
\left\{
  \begin{matrix}
   F_{n+1} \\
   F_n \\
  \end{matrix}
  \right\}  =  
  \left\{
  \begin{matrix}
    F_{n} + F_{n-1} \\
    F_{n} + 0 \\
  \end{matrix}
  \right\}
$$    

然后进一步可以得到:

$$
	\left\{
  \begin{matrix}
    F_{n} + F_{n-1} \\
    F_{n} + 0 \\
  \end{matrix}
  \right\}    = 
  \left\{
  \begin{matrix}
    1 & 1 \\
    1 & 0
  \end{matrix}
  \right\}    
  \left\{
  \begin{matrix}
    F_{n} \\
    F_{n-1}\\
  \end{matrix}
  \right\}    
$$    

于是我们可以得到这样一个等式:

$$
\left\{	
  \begin{matrix}
   F_{n+1} \\
   F_n \\
  \end{matrix}
  \right\}  =
  \left\{
  \begin{matrix}
    1 & 1 \\
    1 & 0
  \end{matrix}
  \right\}    
  \left\{
  \begin{matrix}
    F_{n} \\
    F_{n-1}\\
  \end{matrix}
  \right\}
$$    

继续提取右侧的等式直到化简成F(0)和F(1)，我们最终可以得到这样一个等式:

$$
\left\{	
  \begin{matrix}
   F_{n+1} \\
   F_n \\
  \end{matrix}
  \right\}  =
  \left\{
  \begin{matrix}
    1 & 1 \\
    1 & 0
  \end{matrix}
  \right\} ^ n
  \left\{
  \begin{matrix}
    F_{1} \\
    F_{0}\\
  \end{matrix}
  \right\}
$$  

我们令

$$
A = 
\left\{
\begin{matrix}
1 & 1 \\
1 & 0 \\
\end{matrix}
\right\}= 
\left\{
\begin{matrix}
F_2 & F_1 \\
F_1 & F_0 \\
\end{matrix}
\right\}
$$


用数学归纳法推导出

$$
A ^ n = 
\left\{
\begin{matrix}
F_{n+1} & F_{n} \\
F_{n} & F_{n-1} \\
\end{matrix}
\right\}
$$    

到这里，我们发现只要计算出A的n次方这个矩阵，然后副对角线就是我们要求的值。然后进一步想到**数的幂的快速运算**:    

$$
x ^ n = 
\begin{cases}
 x ^ \frac{n-1}{2} \cdot x ^ \frac{n-1}{2} \cdot {x}   & & {n为奇数} \\
 x ^ \frac{n}{2} \cdot x ^ \frac{n}{2}  & & {n为偶数} \\
\end{cases}
$$

同理我们看看矩阵的幂运算可不可以快速化，推导出下列等式    

$$
A ^ {2m} = A ^ m \cdot A ^ m = 
\left\{
\begin{matrix}
F_{m+1} & F_{m} \\
F_{m} & F_{m-1} \\
\end{matrix}
\right\} 
\cdot  
\left\{
\begin{matrix}
F_{m+1} & F_{m} \\
F_{m} & F_{m-1} \\
\end{matrix}
\right\}
\\=
\left\{
\begin{matrix}
{F_{m+1}} ^ 2 + {F_m} ^ 2  & {F_{m+1}{F_m}} + {F_m}{F_{m-1}}\\
{F_	m}{F_{m+1} + {F_m}{F_{m-1}}} & {F_m} ^ 2 + {F_{m-1}} ^ 2
\end{matrix}
\right\}
\\ =  
\left\{
\begin{matrix}
F_{2m+1} & F_{2m} \\
F_{2m} & F_{2m-1}
\end{matrix}
\right\}
$$

所以我们可以得出    

$$
F_{2m+1} = {F_{m+1}} ^ 2 + {F_m} ^ 2 \\
F_{2m} = {F_m} \cdot (F_{m+1} + F_{m-1})
$$


所以我们可以得到这样一个等式

$$
\left\{	
  \begin{matrix}
   F_{n+1} \\
   F_n \\
  \end{matrix}
  \right\}  = 
  \begin{cases}
  	\left\{	
  \begin{matrix}
   F_{2m+2} \\
   F_{2m+1} \\
  \end{matrix}
  \right\}    
  = 
  \left\{	
  \begin{matrix}
   (2{F_m} + F_{m+1}) \cdot F_{m+1} \\
   {F_m} ^ 2 + {F_{m+1}} ^ 2 \\
  \end{matrix}
  \right\}  
  & & n为奇数，n = 2m+1 & & 此时 m = \frac{n-1}{2} \\
  \left\{	
  \begin{matrix}
   F_{2m+1} \\
   F_{2m} \\
  \end{matrix}
  \right\} 
  = 
  \left\{
  \begin{matrix}
  {F_{m+1}} ^ 2 + {F_m} ^ 2 \\
  {F_m} \cdot (F_{m+1} + F_{m-1})
  \end{matrix}
  \right\}
  = 
  \left\{
  \begin{matrix}
  {F_{m+1}} ^ 2 + {F_m} ^ 2 \\
  {F_m} \cdot (2{F_{m+1}} - F_{m})
  \end{matrix}
  \right\} & & n为偶数，n = 2m & & 此时 m = \frac{n}{2}
  \end{cases}
$$
    
至此，我们推导出来使用矩阵递快速幂递归的方式计算斐波那契数列。只需要计算出Fib(n/2)即可，时间复杂度为对数时间。

## 要想程序性能好，关键数学要学好!!!

