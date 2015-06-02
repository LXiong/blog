# 排序算法 - Sorting algorithm

所谓排序，就是使一串记录，按照其中的某个或某些关键字的大小，递增或递减的排列起来的操作。

排序(Sorting) 是计算机程序设计中的一种重要操作，它的功能是将一个数据元素(或记录)的任意序列，重新排列成一个关键字有序的序列。

## 算法列表

算法有一些指标，例如：时间复杂度，空间复杂度，稳定性等。

排序算法有很多，我们这里简单的列举部分。

### 稳定的

- 冒泡排序(bubble sort) — O(n^2)
- 鸡尾酒排序(Cocktail sort，双向的冒泡排序) — O(n^2)
- 插入排序(insertion sort) — O(n^2)
- 桶排序(bucket sort) — O(n); 需要 O(k) 额外空间
- 计数排序(counting sort) — O(n+k); 需要 O(n+k) 额外空间
- 合并排序(merge sort) — O(nlog n); 需要 O(n) 额外空间
- 原地合并排序 — O(n^2)
- 二叉排序树排序(Binary tree sort) — O(nlog n)期望时间；O(n^2)最坏时间；需要 O(n) 额外空间
- 鸽巢排序(Pigeonhole sort) — O(n+k); 需要 O(k) 额外空间
- 基数排序(radix sort) — O(n·k); 需要 O(n) 额外空间
- Gnome 排序 —  O(n^2)
- 图书馆排序 — O(nlog n) with high probability，需要 (1+ε)n 额外空间

### 不稳定的

- 选择排序(selection sort) — O(n^2)
- 希尔排序(shell sort) — O(nlog n) 如果使用最佳的现在版本
- 组合排序 — O(nlog n)
- 堆排序(heapsort) — O(nlog n)
- 平滑排序 — O(nlog n)
- 快速排序(quicksort) — O(nlog n) 期望时间，O(n^2) 最坏情况； 对于大的、乱数列表一般相信是最快的已知排序
- Introsort — O(nlog n)
- Patience sorting — O(nlog n+ k) 最坏情况时间，需要 额外的 O(n+ k) 空间，也需要找到最长的递增子串行(longest increasing subsequence)

### 不实用的

- Bogo排序 — O(n×n!) 期望时间，无穷的最坏情况。
- Stupid sort — O(n^3); 递归版本需要 O(n^2) 额外存储器
- 珠排序(Bead sort) — O(n) or O(√n),但需要特别的硬件
- Pancake sorting — O(n),但需要特别的硬件
- Stooge sort — O(n^2.7),很漂亮但是很耗时

## 常用算法

排序算法有很多，我们目前搜集到得有下面的一些。


