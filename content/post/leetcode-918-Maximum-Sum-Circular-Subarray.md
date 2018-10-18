---
title: "Leetcode 918 Maximum Sum Circular Subarray"
date: 2018-10-07T00:30:43-04:00
draft: false
tags: ["sum array sum", "leetcode"]
---

这道题看的答案。

在circular的情况，出现最大值有两种可能，一是subarray是连续的，在原数组的中间某处；另一种情况是subarray出现原数组的结尾和开头，也就是由两部分组成。

对于第一种情况，就按照正常方法计算。第二种情况可以由cumulate sum 减去 minimum subarray sum 得到。

最终结果是这两种情况结果里面的最大值。

```java
class Solution {
    public int maxSubarraySumCircular(int[] A) {
        int len = A.length;
        
        //计算cumulate sum
        int sum = 0;
        for(int i : A) 
            sum += i;
        
        // non-circular min sub array sum
        int cumulateSum = 0;
        int minSubarraySum = Integer.MAX_VALUE;
        for(int i : A) {
            cumulateSum += i;
            minSubarraySum = Math.min(cumulateSum, minSubarraySum);
            if(cumulateSum > 0) 
                cumulateSum = 0; 
        }
        
        //non-circular max sub array sum
        cumulateSum = 0;
        int maxSubarraySum = Integer.MIN_VALUE;
        for(int i : A) {
            cumulateSum += i;
            maxSubarraySum = Math.max(cumulateSum, maxSubarraySum);
            if(cumulateSum < 0) 
                cumulateSum = 0;
        }
        
        // 如果数组中元素都为负
        if(maxSubarraySum < 0) 
            return maxSubarraySum;

        return Math.max(maxSubarraySum, sum-minSubarraySum);
    }
}
```