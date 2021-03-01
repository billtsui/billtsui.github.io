---
title: 顺序查找
categories: [Algorithm] 
date:   2021-03-01 21:31:38 +0800
tags: [Search,Algorithm]
---

## **顺序查找**



##### **基本思想**

顺序查找，就是从第一个元素开始，按照索引顺序遍历待查找序列，直到找出给定目标或者查找失败。



##### **特点**

1、对待查找的序列无要求。既可以是有序序列，也可以是无序序列。

2、从第一个元素开始。

3、需要遍历整个待查找序列。

4、如果遍历到最后一个元素还没找到，则查找失败。



##### **缺点**

1、效率低，需要遍历整个待查找序列。



##### **时间复杂度**

O(n)，平均查找时间 = 集合长度/2



##### **空间复杂度**

O(n)，集合的长度+1个目标元素



#### **示例代码**

```java
//input:{3,6,7,2,12,9,0,11}
//target:0
//return:6

class SequentialSearch {
	public static void main(String[] args) {
		int[] nums = {3,6,7,2,12,9,0,11};
		int index = sequential_search(nums,0);
		
		int[] nums1 = {};
		int index1 = sequential_search(nums1, 0);
		System.out.println(index);
		System.out.println(index1);
	}
	
	public static int sequential_search(int nums[],int target){
		if(nums.length <= 0) return -1;
		
		for(int i = 0; i < nums.length; i++){
			if(nums[i] == target){
				return i;
			}
		}
		
		return -1;
	}
}
```

