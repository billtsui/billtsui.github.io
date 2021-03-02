---
title: 二分查找
categories: [Algorithm] 
date:   2021-03-01 21:31:38 +0800
tags: [Search,Algorithm]
---

在计算机科学中，**二分查找算法**（英语：binary search algorithm），也称**折半搜索算法**（英语：half-interval search algorithm）、**对数搜索算法**（英语：logarithmic search algorithm），是一种在有序数组中查找某一特定元素的搜索算法。

#### **特点**

1、对待查找的序列要求有序。

2、比较次数少。



#### **缺点**

1、数组必须要有序，使用前需要先排序。



#### **时间复杂度**

$$
O(\log_2n)
$$



#### **空间复杂度**

O(1)，不需要额外的空间，所以为常数空间复杂度



#### **常规的二分查找示例代码**

```java
class SequentialSearch {
	public static void main(String[] args) {
		int[] nums = {0,2, 3, 6, 7, 9,11, 12};
		int index = binary_search(nums, 6);
		int index1 = binary_search(nums,13);
		System.out.println(index);//3
		System.out.println(index1);//-1
	}
	
	public static int binary_search(int nums[],int target){
		//左闭右闭区间[left,right]
		int left = 0,right = nums.length -1;
		while(left <= right){
			//防止两个大数相加溢出
			int middle = left + (right - left)/2;
			if(nums[middle] == target){
				return middle;
			}else if(nums[middle] > target){
				right = middle -1;
			}else if(nums[middle] < target){
				left = middle + 1;
			}
		}
		return -1;
	}
}
```



#### 常规二分查找的缺陷

针对以下输入，常规二分查找存在一定的缺陷。

```java
//input
{0,2, 3, 6, 8, 8, 8, 7, 9,11, 12}

//target
8

//return
5
```

没错，虽然返回值也符合，但是我们想要得到第一个索引，常规写法就不满足了。当然，也可以在找到target之后采用线性查找找到左边第一个数，但这样破坏了二分查找的时间复杂度，得不偿失。所以就有寻找左侧边界的二分查找。



#### 寻找左侧边界的二分查找

寻找左侧边界，就是在target找到的时候，右侧边界向左移，继续查找。

```java
class SequentialSearch {
	public static void main(String[] args) {
		int[] nums = {0,2, 3, 6, 8, 8, 8, 9, 10, 11, 12};
		int index = left_bound(nums, 8);
		int index1 = left_bound(nums,13);
		System.out.println(index);//4
		System.out.println(index1);//-1
	}
	
	
	public static int left_bound(int nums[],int target){
		int left = 0, right = nums.length - 1;
		while(left <= right){
			int middle = left + (right - left)/2;
			if(nums[middle] == target){
				right = middle - 1;
			}else if(nums[middle] > target){
				right = middle - 1;
			}else if(nums[middle] < target){
				left = middle + 1;
			}
		}
		
        /**
        	循环结束的条件是left = right + 1
        	如果right走到最后一个元素，都没有找到target，那么left就会越界
        **/
		if(left >= nums.length || nums[left] != target){
			return -1;
		}
		
		return left;
	}
}
```



#### 寻找右侧边界的二分查找

寻找右侧边界的二分查找，即找最后一个元素，逐渐收缩左边界即可。

```java
class SequentialSearch {
	public static void main(String[] args) {
		int[] nums = {0, 2, 3, 6, 8, 8, 8, 9, 10, 11, 12};
		int index = right_bound(nums,8);
		System.out.println(index);//6
	}
	
	public static int right_bound(int[] nums,int target){
		int left = 0,right = nums.length - 1;
		while(left <= right){
			int middle = left + (right - left)/2;
			if(nums[middle] == target){
				left = middle + 1;
			}else if(nums[middle] > target){
				right = middle - 1;
			}else if(nums[middle] < target){
				left = middle + 1;
			}
		}
		
        /**
        	循环结束的条件是left = right + 1;
        	如果left跑到最左边一个，都没有找到target，这个时候right下标就越界了
        **/
		if(right < 0 || nums[right] != target){
			return -1;
		}
		return right;
	}
}
```



