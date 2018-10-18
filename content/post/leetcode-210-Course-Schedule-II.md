---
title: "Leetcode 210 Course Schedule"
date: 2018-10-08T18:19:22-04:00
draft: false
tags: ['leetcode', 'topological sort']
---


There are a total of n courses you have to take, labeled from 0 to n-1.

Some courses may have prerequisites, for example to take course 0 you have to first take course 1, which is expressed as a pair: [0,1]

Given the total number of courses and a list of prerequisite pairs, return the ordering of courses you should take to finish all courses.

There may be multiple correct orders, you just need to return one of them. If it is impossible to finish all courses, return an empty array.

Example 1:
```
Input: 2, [[1,0]] 
Output: [0,1]
Explanation: There are a total of 2 courses to take. To take course 1 you should have finished   
             course 0. So the correct course order is [0,1] .
```
Example 2:
```
Input: 4, [[1,0],[2,0],[3,1],[3,2]]
Output: [0,1,2,3] or [0,2,1,3]
Explanation: There are a total of 4 courses to take. To take course 3 you should have finished both     
             courses 1 and 2. Both courses 1 and 2 should be taken after you finished course 0. 
             So one correct course order is [0,1,2,3]. Another correct ordering is [0,2,1,3] .
```

Note:

The input prerequisites is a graph represented by a list of edges, not adjacency matrices. Read more about how a graph is represented.
You may assume that there are no duplicate edges in the input prerequisites.

这道题和207 Course Schedule 非常相似，唯一的区别就是要打印出一个valid的顺序，因此每当遇到入度为0的节点就把它加到结果集里面就行了。

```java
class Solution {
    public int[] findOrder(int numCourses, int[][] prerequisites) {
        List<List<Integer>> graph = new ArrayList<>();
        for(int i = 0; i < numCourses; i++) {
            graph.add(new LinkedList<Integer>());
        }
        
        int[] indegree = new int[numCourses];
        for(int[] prerequisite : prerequisites) {
            graph.get(prerequisite[1]).add(prerequisite[0]);
            indegree[prerequisite[0]] += 1;
        }
        
        int[] result = new int[numCourses];
        int pointer = 0; 
        
        Deque<Integer> dq = new LinkedList<>();
        for(int i = 0; i < numCourses; i++) {
            if(indegree[i] == 0) {
                dq.add(i);
                result[pointer] = i;
                pointer += 1;
            }
        }

        while(!dq.isEmpty()) {
            int course = dq.poll();
            for(int i : graph.get(course)) {
                indegree[i] -= 1;
                if(indegree[i] == 0) {
                    dq.add(i);
                    result[pointer] = i;
                    pointer += 1;
                }
            }
        }
        
        if(pointer < numCourses) {
            return new int[0];
        }
        
        return result;
    }
}
```
