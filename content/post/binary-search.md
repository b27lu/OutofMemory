---
title: "Binary Search 总结"
date: 2018-09-30T15:53:11-04:00
draft: false
---

binary search 最关键的问题就是注意右边界的位置。有两种写法

1. 右边界 `hi ＝ arr.length - 1`, 即右边界是最后一个参与binary search的元素。从而while loop的判断条件是 `lo <= hi`，同时 `lo = mid + 1` `hi = mid - 1`。 注意这里`hi = mid - 1`是因为右边界必须是参与binary search的元素，所以不是`mid` 而是 `mid-1`
```
    int lo = 0, hi = nums.length-1;
    while(lo <= hi) {
        int mid = (lo + hi) / 2;
        if (nums[mid] + temp > target) {
            hi = mid - 1;
        } else if (nums[mid] + temp < target) {
            lo = mid + 1;
        } else {
            List<Integer> group = Arrays.asList(nums[i], nums[j], nums[k], nums[mid]);
            result.add(group);
            break;
        }
    }
```

2. 右边界 `hi ＝ arr.length`, 即右边界是最后一个参与binary search的元素的下一个。从而while loop的判断条件是 `lo < hi`，同时 `lo = mid + 1` `hi = mid`。 注意这里`hi = mid`是因为右边界是参与binary search的元素的下一个，所以是`mid` 而不是 `mid-1`
```                    
    int lo = 0, hi = nums.length;
    while(lo < hi) {
        int mid = (lo + hi) / 2;
        if (nums[mid] + temp > target) {
            hi = mid;
        } else if (nums[mid] + temp < target) {
            lo = mid + 1;
        } else {
            List<Integer> group = Arrays.asList(nums[i], nums[j], nums[k], nums[mid]);
            result.add(group);
            break;
        }
    }
```

对于第一种方法，为什么`lo`一定要`mid+1`，能不能是`lo=mid`呢？不能。因为当只有一个或两个元素的情况下，`lo=mid` 和`hi = mid`会导致死循环。除非加判断`if(lo == hi)` 如何如何 还有 `if (hi - lo == 1 如何如何

对于binary search 时 target 找不到的情况下，最终lo会等于理想的插入位置，有可能是`arr.length` 因为是理想的插入位置。java jdk源程序中的处理是返回 `- insertionPoint - 1 `作为返回值。java collections binary search code

```
int low = 0;
int high = list.size()-1; 
while (low <= high) { 
    int mid = (low + high) >>> 1; 
    Comparable<? super T> midVal = list.get(mid); 
    int cmp = midVal.compareTo(key); 
    if (cmp < 0) 
        low = mid + 1; 
    else if (cmp > 0) 
        high = mid - 1; 
    else
        return mid; // key found
    } 
    return -(low + 1); // key not found
```

Problem list:
Search in Rotated Sorted Array II  判断哪边是有序的，然后缩小查找范围。注意有可能两边全无序！e.g. ［1，1，1，1，2，1，1］则只能左右各缩小1，前提是左右边界都不是target

Find Minimum in Rotated Sorted Array II 这里并不是对找到的mid如何如何，关键是缩小范围，同时避免lo == hi 和 hi - lo == 1时出现死循环。还有就是注意有可能两边全无序！e.g. ［10，1，10，10，10，10］