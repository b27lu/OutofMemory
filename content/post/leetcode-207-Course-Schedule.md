---
title: "Leetcode 207 Course Schedule"
date: 2018-10-08T12:19:22-04:00
draft: false
tags: ['leetcode', 'topological sort', 'dfs']
---

There are a total of n courses you have to take, labeled from 0 to n-1.

Some courses may have prerequisites, for example to take course 0 you have to first take course 1, which is expressed as a pair: [0,1]

Given the total number of courses and a list of prerequisite pairs, is it possible for you to finish all courses?

Example 1:
```
Input: 2, [[1,0]] 
Output: true
Explanation: There are a total of 2 courses to take. 
             To take course 1 you should have finished course 0. So it is possible.
```
Example 2:
```
Input: 2, [[1,0],[0,1]]
Output: false
Explanation: There are a total of 2 courses to take. 
             To take course 1 you should have finished course 0, and to take course 0 you should
             also have finished course 1. So it is impossible.
```
Note:

The input prerequisites is a graph represented by a list of edges, not adjacency matrices. Read more about how a graph is represented.
You may assume that there are no duplicate edges in the input prerequisites.

方法1: topological sort

    步骤1: 建立邻接矩阵用来表示图，建立入度数组
    步骤2: 把所有入度为`0`的节点加入`queue
    步骤3: 依次取出queue中节点，并更新入度数组，把入度刚变为`0`的节点加入到queue中

```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        int[][] matrix = new int[numCourses][numCourses];
        int[] indegree = new int[numCourses];
        
        for(int[] prerequisite : prerequisites) {
            int pre = prerequisite[1];
            int cur = prerequisite[0];
            matrix[pre][cur] = 1;
            indegree[cur] += 1;
        }
        
        Deque<Integer> dq = new LinkedList<>();
        for(int i = 0; i < numCourses; i++) {
            if(indegree[i] == 0) {
                dq.offer(i);
            }
        }
        
        int count = 0;
        while(!dq.isEmpty()) {
            int course = dq.poll(); //取出节点，可以理解为把这个课上了
            count += 1;
            for(int i = 0; i < numCourses; i++) {
                if(matrix[course][i] == 1) {
                    indegree[i] -= 1;
                    if(indegree[i] == 0) {
                        dq.offer(i);
                    }
                }
            }
        }
        
        return count == numCourses;
    }
}
```

方法2: DFS

对于每一个课程，递归检查它的prerequisite课程，如果它的prerequisite课程里面有在当前递归路径出现的（stacked[course] == true）那么就是有相互依赖的关系－－有向图中存在环。

这道题的本质就是检查有向图中是否存在环。因此下面注释处的位置如果写成`graph.get(pre).add(cur);`也不影响结果，但是从意义上就不准确了。我是要沿着每一个课程的prerequisite追本溯源，而不是相反方向。

```java
class Solution {
    public boolean canFinish(int numCourses, int[][] prerequisites) {
        List<List<Integer>> graph = new ArrayList<>();
        for(int i = 0; i < numCourses; i++) {
            graph.add(new LinkedList<Integer>());
        }
        
        for(int[] prerequisite : prerequisites) {
            int pre = prerequisite[1];
            int cur = prerequisite[0];
            graph.get(cur).add(pre);    //本题中此处方向不影响结果
        }
        
        boolean[] visited = new boolean[numCourses];
        boolean[] stacked = new boolean[numCourses];
        
        for(int i = 0; i < numCourses; i++) {
            if(isCyclic(i, visited, stacked, graph)) {
                return false;
            }
        }
        
        return true;
    }
    
    private boolean isCyclic(int course, boolean[] visited, boolean[] stacked, List<List<Integer>> graph) {
        if(visited[course]) {
            return false;
        }
        
        if(stacked[course]) {
            return true;
        }
        
        stacked[course] = true;
        for(Integer i : graph.get(course)) {
            if(isCyclic(i, visited, stacked, graph)) {
                return true;
            }
        }
        
        stacked[course] = false;
        visited[course] = true;
        return false;
    }
}
```