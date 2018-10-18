---
title: "Leetcode 209 Minimum Size Subarray Sum"
date: 2018-10-01T22:20:43-04:00
draft: false
tags: ["binary search", "leetcode"]
---

Given an array of n positive integers and a positive integer s, find the minimal length of a contiguous subarray of which the sum ≥ s. If there isn't one, return 0 instead.

Example: 
```
Input: s = 7, nums = [2,3,1,2,4,3]
Output: 2
Explanation: the subarray [4,3] has the minimal length under the problem constraint.
```

Follow up:
If you have figured out the O(n) solution, try coding another solution of which the time complexity is O(n log n). 

---

滑动窗口典型题目，一种做法是`O(N)`的滑动窗口做法，另一种是为了用binary search而用binary search的方法 

窗口长度最小是`1`， 最长是数组长度。从最左边元素开始，把其加入到当前的`sum`中，如果`sum`大于 target `s`， 则收缩左边界，同时更新`sum` 否则继续扩张右边界。

时间复杂度 `O(N)` 其中`N`是数组长度。 

第一次自己写的答案，代码啰嗦，可读性不好。
```java
class Solution {
    public int minSubArrayLen(int s, int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }
        
        int result = nums.length+1;
        
        int sum = nums[0];
        int i = 0, j = 0;
        
        while(j < nums.length) {
            if(sum < s) {
                if(j < nums.length - 1) {
                    j += 1;
                    sum += nums[j];
                } else {
                    break;
                }
            }
            
            if(sum >= s) {
                result = result > j - i + 1 ? j - i + 1 : result;
                sum -= nums[i];
                i += 1;
            }
        }
        
        if(result == nums.length + 1) {
            result = 0 ;
        }
            
        return result;
    }
}
```

参考答案后，自己写的程序，时间复杂度一样，但是代码可读性更好

```java
class Solution {
    public int minSubArrayLen(int s, int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }
        
        int left = 0; //left boundary
        int result = nums.length + 1;   // initialize window length as max possible answer + 1
        int sum = 0;    //current sum
        
        for(int i = 0; i < nums.length; i++) {
            sum += nums[i];
            while(sum >= s) {
                result = result > i - left + 1 ? i - left + 1 : result;
                sum -= nums[left];
                left += 1;
            }
        }
        
        if(result == nums.length + 1) {
            result = 0;
        }
        
        return result;
    }
}
```

－－－

binary search 方法

```java
class Solution {
    public int minSubArrayLen(int s, int[] nums) {
        if(nums == null || nums.length == 0) {
            return 0;
        }
        
        int[] cumulateSum = new int[nums.length+1];  // cumulate sum 数组是解决subarray sum 问题的常见方法
        for(int i = 1; i < cumulateSum.length; i++) {
            cumulateSum[i] = cumulateSum[i-1] + nums[i-1];  //cumulateSum[i] is the sum of all elements before nums[i]
        }
        
        int result = nums.length + 1;   //初始化为一个不可能的数值
        for(int i = 0; i < nums.length; i++) {  //从左向右遍历 nums[]
            // 为什么start 是 i 呢？ 因为cumulateSum[i] 正对应nums[i] 左侧不包含nums[i]的所有元素的和
            // 为什么target是s + cumulateSum[i]呢? 因为假设存在target cumulateSum[j] 满足 cumulateSum[j] >= s + cumulateSum[i]
            //  则nums[] 中对应的满足条件的区间是 从 nums[i] 到 nums[j-1]，此区间的长度正是下面的 right - i
            int right = findRight(cumulateSum, i, nums.length, s + cumulateSum[i]);
            if(right != cumulateSum.length+1) {
                result = result > right - i ? right - i : result;
            }
        }
        
        if(result == nums.length + 1) {
            result = 0;
        }
        
        return result;
    }
    
    private int findRight(int[] cumulateSum, int start, int end, int target) {
        while(start < end) {      //此处 start < end 则退出时 start == end 为true
            int mid = (start + end) / 2;
            // 此处即使cumulateSum[mid] > target时，end 也没有收缩到end = mid-1的原因是
            // 要找的是cumulateSum[x] >= target的x, 此时cumulateSum[mid-1] 不一定 >= target
            if(cumulateSum[mid] >= target) {   
                end = mid;
            } else {
                start = mid + 1;
            }
        }
        
        // at this moment, start always equals to end, 
        // however, cumulateSum[start] is not necessarily great than target
        // eg. cumulateSum is [3,5] and target is 8
        if (cumulateSum[start] >= target)
            return start;
                
        return cumulateSum.length + 1;
    }
}
```