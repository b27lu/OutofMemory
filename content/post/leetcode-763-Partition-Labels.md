---
title: "Leetcode 763 Partition Labels"
date: 2018-10-04T00:03:26-04:00
draft: false
tags: ["greedy", "leetcode"]
---

A string S of lowercase letters is given. We want to partition this string into as many parts as possible so that each letter appears in at most one part, and return a list of integers representing the size of these parts.

Example 1:
```
Input: S = "ababcbacadefegdehijhklij"
Output: [9,7,8]
Explanation:
The partition is "ababcbaca", "defegde", "hijhklij".
This is a partition so that each letter appears in at most one part.
A partition like "ababcbacadefegde", "hijhklij" is incorrect, because it splits S into less parts.
```
Note:

1. S will have length in range [1, 500].
2. S will consist of lowercase letters ('a' to 'z') only.

------

Greedy 方法，对于每一个char c 找到它最右边的出现位置，那么从当前char c的位置到最右边的char c的位置都是处于同一区间。

corner case: “abab" `b`在这里会使区间增长。

```java
class Solution {
    public List<Integer> partitionLabels(String S) {
        int[] map = new int[26];       //map[0] is the index of right most 'a' and so on
        
        for(int i = 0; i < S.length(); i++) {
            char c = S.charAt(i);
            map[c - 'a'] = i;
        }
        
        List<Integer> result = new LinkedList<>();
        int start = 0, end = 0; //start and end of the current part
        for(int i = 0; i < S.length(); i++) {
            char c = S.charAt(i);
            if(map[c - 'a'] == end && i == end) {   //注意这个判断条件，非常重要，如果没有｀i == end｀ 那么”abcab"遍历到c时，也会进入这个block
                result.add(end - start + 1);
                start = end + 1;
                end = end + 1;
            } else if(map[c - 'a'] > end) {
                end = map[c - 'a'];
            }
        }
        
        return result;
    }
}
```

时间复杂度`O(N)`，空间复杂度`O(1)`, 数组长度是26，是constant