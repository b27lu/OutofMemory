---
title: "Leetcode 647 Palindromic Substrings"
date: 2018-10-05T00:10:49-04:00
draft: false
tags: ["palindrome", "leetcode"]
---


Given a string, your task is to count how many palindromic substrings in this string.

The substrings with different start indexes or end indexes are counted as different substrings even they consist of same characters.

Example 1:
```
Input: "abc"
Output: 3
Explanation: Three palindromic strings: "a", "b", "c".
```

Example 2:
```
Input: "aaa"
Output: 6
Explanation: Six palindromic strings: "a", "a", "a", "aa", "aa", "aaa".
```

Note:
The input string length won't exceed 1000.


------

方法1: brute force 加 hashmap 存储中间结果加速， 居然也过了 O(N^3)时间
```java
class Solution {
    private static Set<String> hs = new HashSet<>();
    public int countSubstrings(String s) {
        if(s.length() <= 1) {
            return s.length();
        }
        
        int result = 0;
        for(int i = 0; i < s.length(); i++) {
            for(int j = i+1; j <= s.length();j++) {
                String substr = s.substring(i,j);
                if(hs.contains(substr)) {
                    result += 1;
                } else if(isPalindrom(substr)) {
                    result += 1;
                    hs.add(substr);
                }
            }
        }
        
        return result;
    }
    
    private boolean isPalindrom(String str) {
        if(str.length() == 1) {
            return true;
        }
        
        for(int i=0, j=str.length()-1; i < j; i++, j--) {
            if(str.charAt(i) != str.charAt(j)) {
                return false;
            }
        }
        
        return true;
    }
}
```

方法2: palindrome有两种，一种类似于`aba` 另一种`abba`。 对于字符串中的每一个字符，都把它视为palindrome的`aba`类型的中心和`abba`类型的左中心 各检查一遍。时间复杂度O(N^2)

```java
class Solution {
    public int countSubstrings(String s) {
        if(s.length() <= 1) {
            return s.length();
        }
        
        int result = 1; //last character won't be included below, so make up here
        for(int i = 0; i < s.length() -1; i++) {
            result += countPalindrome(s, i, i);
            result += countPalindrome(s, i, i+1);
        }
        
        return result;
    }
    
    private int countPalindrome(String s, int i, int j) {
        int count = 0;
        
        while(i >= 0 && j < s.length()) {
            if(s.charAt(i) == s.charAt(j)) {
                count += 1;
            } else {
                break;
            }
            i -= 1;
            j += 1;
        }
        
        return count;
    }
}
```

另有专用算法可以做到O(N)，反正也记不住，不管了。