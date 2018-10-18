---
title: "Leetcode 508 Most Frequent Subtree Sum"
date: 2018-10-07T22:40:28-04:00
draft: false
tags: ['leetcode', 'recursion']
---

Given the root of a tree, you are asked to find the most frequent subtree sum. The subtree sum of a node is defined as the sum of all the node values formed by the subtree rooted at that node (including the node itself). So what is the most frequent subtree sum value? If there is a tie, return all the values with the highest frequency in any order.

Examples 1
Input:
```
  5
 /  \
2   -3
```
return [2, -3, 4], since all the values happen only once, return all of them in any order.
Examples 2
Input:
```
  5
 /  \
2   -5
```
return [2], since 2 happens twice, however -5 only occur once.
Note: You may assume the sum of values in any subtree is in the range of 32-bit signed integer.

常见递归求解树相关问题。注意点：想清楚在当前递归循环汇总和处理子树的结果还是当前节点的结果。两种都可行，但是如果混用就不行了，思路别乱。

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode(int x) { val = x; }
 * }
 */
class Solution {
    public int[] findFrequentTreeSum(TreeNode root) {
        Map<Integer, Integer> sumFreqMap = new HashMap<>();
        
        recursiveWorker(root, sumFreqMap);
        
        int maxFreq = 0;
        for(Integer i : sumFreqMap.keySet()) {
            if(sumFreqMap.get(i) > maxFreq) {
                maxFreq = sumFreqMap.get(i);
            }
        }
        
        List<Integer> result = new LinkedList<>();
        for(Integer i : sumFreqMap.keySet()) {
            if(sumFreqMap.get(i) == maxFreq) {
                result.add(i);
            }
        }
        
        int[] resultArr = new int[result.size()];
        for(int i = 0; i < result.size(); i++) {
            resultArr[i] = result.get(i);
        }
        
        return resultArr;
    }
    
    private int recursiveWorker(TreeNode root, Map<Integer, Integer> sumFreqMap) {
        
        if(root == null) {
            return 0;
        }
        
        int left = recursiveWorker(root.left, sumFreqMap);
        int right = recursiveWorker(root.right, sumFreqMap);
        int sum = root.val + left + right;    
        
        sumFreqMap.put(sum, sumFreqMap.getOrDefault(sum, 0) + 1);
    
        return sum; 
    }
}
```