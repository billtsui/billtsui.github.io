---
layout: post
title: 一点一滴学Swift-常量变量的声明方式和基础数据类型
categories: Swift 
date:   2020-07-25 12:10:38 +0800
tags: 移动开发
---


## 声明方式  
```swift
let maxSize = 100 // let 关键字声明 常量
var currentSize = 0 //var 关键字声明 变量
```    

## 数据类型   

### 整型 
```swift
/**
	整数
	Int8
	UInt8
	Int16
	UInt16
	Int32
	UInt32
	Int64
	UInt64
	另外还有一个Int/UInt类型，它拥有与当前平台字长相同的长度。32位总线就是32bit，64位就是64bit
*/

var age:UInt8
var globalPopulation:UInt32
var globalCarNumber:UInt64
var temperature:Int8
print(UInt8.max)
print(Int8.min)
```

### 浮点型
```swift
/**
	浮点数
	Float(6位精度)
	Double(15位精度)
*/

var weight:Float
var price:Double
```

### bool型
```swift
/**
	bool型
	true false
*/

var weight:Float
```

### 给类型起别名
```swift
/**
	类型别名
	typealias 通常用在更需要代表业务类型的时候
*/

typealias myAge = UInt8
let age : myAge
age = 200
```

### 元组
```swift
/**
	Tuple
*/

//非命名元组用下表来访问
let errorInfo = (1,"密码错误")
print(errorInfo.0)
print(errorInfo.1)

//命名元组可以用 .名称 来取值
var successInfo = (successCode:1,successMessage:"登录成功")
print(successInfo.successCode)
print(successInfo.successMessage)

//元组分解到变量
var (code,message) = successInfo
print(code)
print(message)

```



