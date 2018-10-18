---
title: "Leetcode 658 Find K Closest Elements"
date: 2018-10-07T23:24:54-04:00
draft: false
tags: ['leetcode', 'binary search', '滑动窗口']
---

Given a sorted array, two integers k and x, find the k closest elements to x in the array. The result should also be sorted in ascending order. If there is a tie, the smaller elements are always preferred.

Example 1:
```
Input: [1,2,3,4,5], k=4, x=3
Output: [1,2,3,4]
```
Example 2:
```
Input: [1,2,3,4,5], k=4, x=-1
Output: [1,2,3,4]
```

Note:
The value k is positive and will always be smaller than the length of the sorted array.
Length of the given array is positive and will not exceed 104
Absolute value of elements in the array and x will not exceed 104

方法1: binary search ＋ two pointers 左右扩展  O(logN + k)

```java
class Solution {
    public List<Integer> findClosestElements(int[] arr, int k, int x) {
        int index = binarySearch(arr, x)；
        
        List<Integer> result = new LinkedList<>();
        
        int left = index-1, right = index; //arr[index] 不一定是结果，因为target可能不在数组中，arr[index]可能离target更远
        while(k > 0) {
            if(left < 0) {     //左边到头，则扩展右
                result.add(arr[right]);
                right += 1;
            } else if(right >= arr.length) {    //  右边到头，则扩展左
                result.add(0, arr[left]);
                left -= 1;
            } else {       //否则，比较差的绝对值来决定
                if(Math.abs(arr[left] - x) > Math.abs(arr[right] - x)) {
                    result.add(arr[right]);
                    right += 1;
                } else {
                    result.add(0, arr[left]);
                    left -= 1;
                }
            }
            k -= 1;
        }
        
        return result;
    }
    
    //binary search 返回target的index，如果target不在数组中，则返回其左或右的index
    private int binarySearch(int[] arr, int target) {
        int lo = 0, hi = arr.length-1;
        
        while(lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if(arr[mid] == target) {
                return mid;
            } else if(arr[mid] > target) {
                hi = mid;
            } else {
                lo = mid+1;
            }
        }
        
        return lo;
    }
}
```

方法2: 滑动窗口 O(N)

```java
class Solution {
    public List<Integer> findClosestElements(int[] arr, int k, int x) {
        List<Integer> result = new LinkedList<>();
        
        for(int i = 0; i < arr.length; i++) {
            if(result.size() < k) {
                result.add(arr[i]);
            } else {
                if(Math.abs(result.get(0) -x) > Math.abs(arr[i] -x)) {
                    result.remove(0);
                    result.add(arr[i]);
                }
            }
        }
        
        return result;
    }
}
```