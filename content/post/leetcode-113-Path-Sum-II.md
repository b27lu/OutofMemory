---
title: "Leetcode 113 Path Sum II"
date: 2018-10-07T14:17:59-04:00
draft: false
tags: ['tree', 'leetcode']
---

Given a binary tree and a sum, find all root-to-leaf paths where each path's sum equals the given sum.

Note: A leaf is a node with no children.

Example:
```
Given the below binary tree and sum = 22,

      5
     / \
    4   8
   /   / \
  11  13  4
 /  \    / \
7    2  5   1
```
Return:
```
[
   [5,4,11,2],
   [5,8,4,5]
]
```

关于树的典型题目。
注意要点1: 本层recursion完成之后，要从current的末尾移除当前元素，恢复现场。
注意要点2: `List<Integer> temp = new LinkedList(current);` 是deep copy可以直接用

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
    public List<List<Integer>> pathSum(TreeNode root, int sum) {
        List<List<Integer>> result = new LinkedList<>();
        if(root == null) {
            return result;
        }
        
        recursiveWorker(root, sum, new LinkedList<Integer>(), result);
        return result;
    }
    
    private void recursiveWorker(TreeNode root, int sum, List<Integer> current, List<List<Integer>> result) {
        if(root == null) {
            return;
        }
        
        if(root.left == null && root.right == null && sum == root.val) {
            List<Integer> temp = new LinkedList(current);
            temp.add(root.val);
            result.add(temp);
            return;
        }
        
        current.add(root.val);
        recursiveWorker(root.left, sum-root.val, current, result);
        recursiveWorker(root.right, sum-root.val, current, result);
        
        current.remove(current.size()-1);
    }
}
```

