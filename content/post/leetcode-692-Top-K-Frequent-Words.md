---
title: "Leetcode 692 Top K Frequent Words"
date: 2018-10-05T23:13:50-04:00
draft: flase
tags: ["Top K", "leetcode"]
---

Top K 问题
维护一个大小是K的min heap， 不断把element加入到heap中，如果size大于K，则取出heap顶的最小元素。最后从heap中依次取出K个元素，依据结果集是否需要排序来加入到结果集中的相应位置。

类似题目 347 Top K Frequent Elements

```java
class Solution {
    public List<String> topKFrequent(String[] words, int k) {
        List<String> result = new LinkedList<>();
        if(words == null || words.length == 0 || k > words.length) {
            return result;
        }
        
        PriorityQueue<Pair> minHeap = new PriorityQueue<>(new Comparator<Pair>(){
            @Override
            public int compare(Pair p1, Pair p2) {
                if(p1.freq != p2.freq) {
                    return p1.freq - p2.freq;
                } else {
                    return p2.word.compareTo(p1.word);
                }
            }
        });
        
        Map<String, Integer> wordFreqMap = new HashMap<>();
        for(String word : words) {
            wordFreqMap.put(word, wordFreqMap.getOrDefault(word, 0) + 1);
        }
        
        for(String str : wordFreqMap.keySet()) {
            minHeap.add(new Pair(str, wordFreqMap.get(str)));
            if(minHeap.size() > k) {
                minHeap.poll();
            }
        }
        
        while(k > 0) {
            result.add(0, minHeap.poll().word);
            k -= 1;
        }
        
        return result;
    }
    
    class Pair {
        String word;
        int freq;
        public Pair(String word, int freq) {
            this.word = word;
            this.freq = freq;
        }
    }
}
```

