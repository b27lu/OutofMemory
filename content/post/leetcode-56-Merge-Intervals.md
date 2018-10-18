---
title: "Leetcode 56 Merge Intervals"
date: 2018-10-08T23:33:44-04:00
draft: false
tags: ['leetcode', 'comparator']
---

Given a collection of intervals, merge all overlapping intervals.

Example 1:
```
Input: [[1,3],[2,6],[8,10],[15,18]]
Output: [[1,6],[8,10],[15,18]]
Explanation: Since intervals [1,3] and [2,6] overlaps, merge them into [1,6].
```

Example 2:
```
Input: [[1,4],[4,5]]
Output: [[1,5]]
Explanation: Intervals [1,4] and [4,5] are considerred overlapping.
```

需要自定义comparator 排序的题 `O(NlogN)`

自定义comparator对interval按照start生序排列，然后遍历排序后的数组进行merge。

注意点：merge时，要用`Math.max(intervals.get(i).end, result.get(result.size()-1).end);` 因为待merge的interval有可能完全被前面的interval包含。例如 `［1，5］，［2，3］` 所有选最大的end作为merge后interval的end。


```java
/**
 * Definition for an interval.
 * public class Interval {
 *     int start;
 *     int end;
 *     Interval() { start = 0; end = 0; }
 *     Interval(int s, int e) { start = s; end = e; }
 * }
 */
class Solution {
    public List<Interval> merge(List<Interval> intervals) {
        Collections.sort(intervals, new Comparator<Interval>(){
            @Override
            public int compare(Interval i1, Interval i2) {
                return i1.start - i2.start; 
            }
        });
        
        List<Interval> result = new ArrayList<>();
        for(int i = 0; i < intervals.size(); i++) {
            if(result.size() == 0 || result.get(result.size()-1).end < intervals.get(i).start) {
                result.add(intervals.get(i));
            } else {
                result.get(result.size()-1).end = Math.max(intervals.get(i).end, result.get(result.size()-1).end);
            }
        }
        
        return result;
    }
}
```