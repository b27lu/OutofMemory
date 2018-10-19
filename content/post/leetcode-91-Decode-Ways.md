---
title: "Leetcode 91 Decode Ways"
date: 2018-10-17T23:53:42-04:00
draft: false
tags: ["leetcode", "dfs", "dp"]
---

A message containing letters from A-Z is being encoded to numbers using the following mapping:

'A' -> 1
'B' -> 2
...
'Z' -> 26
Given a non-empty string containing only digits, determine the total number of ways to decode it.

Example 1:
```
Input: "12"
Output: 2
Explanation: It could be decoded as "AB" (1 2) or "L" (12).
```

Example 2:
```
Input: "226"
Output: 3
Explanation: It could be decoded as "BZ" (2 26), "VF" (22 6), or "BBF" (2 2 6).
```

注意以`0` 开头不是一个valid的encode way。

我自己写的方法1: DFS  O(2^N)

```java
class Solution {
    public int numDecodings(String s) {
        if(s == null || s.length() == 0 || s.startsWith("0")) {
            return 0;
        }
        
        List<Integer> result = new ArrayList<>(1);
        result.add(0);
        recursiveWorker(s, 0, result);
        return result.get(0);
    }
    
    private void recursiveWorker(String s, int index, List<Integer> result) {
        if(index == s.length()) {
            result.set(0, result.get(0) + 1);
            return;
        }
        
        if(s.charAt(index) != '0') {
            recursiveWorker(s, index+1, result);
        }
        
        if(index < s.length()-1) {
            char c = s.charAt(index);
            char c1 = s.charAt(index+1);
            
            if(c == '1' || (c == '2' && c1 >= '0' && c1 <= '6')) {
                recursiveWorker(s, index + 2, result);
            }
        }
        
    }
}
```

DP O(N)时间 O(N)空间

注意的关键点在于，初始化`dp[s.length()] = 1;`，这可以理解为我们只有一种方法decode 字符串右侧“虚无”的字符。

从尾到头扫，原因是对例如`2000`的字符串，遇到`0`时可以继续向头方向扫描；如果是从头到尾扫描，那么遇到`0`时需要判断前一个字符是不是`1` 或者`2`。总之是更麻烦一些。

```java
class Solution {
    public int numDecodings(String s) {
        if(s == null || s.length() == 0 || s.startsWith("0")) {
            return 0;
        }
        
        int[] dp = new int[s.length()+1];
        dp[s.length()] = 1;
        for(int i = s.length()-1; i >= 0; i--) {
            if(s.charAt(i) == '0') {
                dp[i] = 0;
            } else {
                dp[i] = dp[i+1];
                if(i < s.length()-1 && (s.charAt(i) == '1' || (s.charAt(i) == '2' && s.charAt(i+1) < '7'))) {
                    dp[i] += dp[i+2];
                }
            }
        }
        
        return dp[0];
    }
}
```

进阶DP O(N)时间 O(1)空间

因为`dp[]`数组其实只需要保留最近的两个元素即可，因此可以用两个变量代替。这也是优化dp的常见方法。