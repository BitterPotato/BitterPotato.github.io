---
layout: post
title: QuickSort
date: 2016-12-13 12:05:57
comments: true
tags: 
	- 算法
	- 面试
---
#### 概述
快速排序是面试中的常见题，每次简述一遍快速排序的原理便觉得仿佛已经掌握了它。不是挺简单的吗？然而实际实现的时候还是会遇到一些坑。于是，之前一次面试就跪了，很尴尬。所以，还是需要对它进行深入理解！


#### 基本步骤
0. 检查范围，(以及终止条件)
> **注意递归终止条件**
1. 选择基准(pivot)，分割序列
> 即筛选不大于pivot的元素和不小于pivot的元素），将pivot放至正确位置

2. 对pivot的左半段和右半段序列分别进行快速排序

参考[1], 我将快速排序的函数签名定为
`public <T extends Comparable<? super T>> void executeProcess(T[] sequ, int left, int right) { }`


#### 1.小数组优化
递归的快速排序终会落入对于小数组的排序。而对于小数组的排序，快速排序不如插入排序。

|                    | quicksort | insertsort |
| ------------------ | --------- | ---------- |
| Size：10 Range：0~10 | 2222223   | 623112     |
| Size：20 Range：0~20 | 1433779   | 487111     |
| Size：30 Range：0~30 | 3454668   | 540889     |

> 单位：nanosecond
> 环境：CPU 4核8线程 2.30GHZ


#### 2.检查范围，以及终止条件
    // 检查下标是否越界
    super.rangeCheck(sequ.length, left, right);
    
    // 数组个数为0或1，已排序（终止条件）
    int size = right - left + 1;
    if (size < 2) {
        return;
    }


#### 3.选择基准(pivot)，分割序列
##### 3.1 选择基准
基准选择常见的有以下三种方法。
1. 序列首/序列尾
   对于有序序列分割极不平衡
2. 随机选择
   优于序列首，但开销不小
3. 三数中值分割
   它将考虑序列中`left, right, (left + right) / 2`这三个位置的元素值，选择它们的中位数作为基准
> 进一步，可在三数中值分割的基础上将三个位置上的较小值和较大值分别置于`left`位置、`right`位置。在使用下文的`分割方式1`时可保证两个指针不越过序列端点。


    // 三数排序决定基准，left/right/中位
    int middle = (left + right) / 2;
    if (sequ[left].compareTo(sequ[middle]) > 0) {
        swap(sequ, left, middle);
    }
    if (sequ[left].compareTo(sequ[right]) > 0) {
         swap(sequ, left, right);
    }
    if (sequ[middle].compareTo(sequ[right]) > 0) {
        swap(sequ, middle, right);
    }
    // 数组仅有2个或3个元素，此时已经排好序
    //（若对小数组使用插入排序，则该语句没有必要）
    if (!super.insertSortOptimized && middle == right - 1) {
        return super.InvalidPoint;
    }
    // 将基准（三数中值）放至right-1位置
    swap(sequ, middle, right - 1);


##### 3.2 分割策略
###### 3.2.1 分割方式1
forePoint从前往后找大于pivot的元素，backPoint从后往前找小于pivot的元素，并交换。
当forePoint与backPoint相遇后，将pivot放至正确位置。
![分割方式1](https://pic.tanuki233.com/quicksort_ver1.png?imageView/2/w/344)
> 之后以此类推
> 标红的元素为pivot

    // 对left+1和right-2之间的范围进行分割
    int forePoint = left;
    int backPoint = right - 1;
    T pivot = sequ[right - 1];
    
    while (true) {
        while (sequ[++forePoint].compareTo(pivot) < 0) {
        }
        while (sequ[--backPoint].compareTo(pivot) > 0) {
        }
    
        if (forePoint >= backPoint) {
            // 将基准放到合适位置
            swap(sequ, forePoint, right - 1);
            break;
        } else {
            swap(sequ, forePoint, backPoint);
        }
    }

> **相等元素的处理**
> 当遇到和基准值相等的值时，应该如何处理？是往左半段移动？还是往右半段移动？
> 特别地，对于forePoint和backPoint同时分别遇到与基准值相等的元素时，应该如何处理？
> 按照[1]中所说，forePoint和backPoint的地位应是等价的，那么它们对于与基准值相等的元素的处理方式也应相同。否则，则会有左半段与右半段不均衡的情况出现，降低快速排序的效率。
> 那么我们还剩下forePoint和backPoint均停止(进行交换)和均不停止(不进行交换)的选择。 [1]中推荐前种做法。那么对于后种做法，可不可行呢？
> 针对上述的三种基准选择方法分别进行分析：
- 前两种选择的基准均有可能是该序列中的最大值或者最小值，序列中可能存在其他与该值相同的元素，也可能不存在。因此，必须考虑forePoint和backPoint越过序列端点的情况，停止与不停止并没有差别。
- 而对于三数中值分割，它所选择的基准，最大仅可能是该序列中的次大值（可能等于最大值），或者最小仅可能是次小值（可能等于最小值）。若在三数中值分割的基础上将三个位置上的较小值和较大值分别置于left位置、right位置，那么，forePoint和backPoint则无法越过序列端点。但考虑到right位置与基准值相等的情况，若采用不停止的方式，则需要再次考虑forePoint越过序列端点的情况，
  因此，遇到与基准相等的元素，forePoint或者backPoint停止并且交换的做法相对更佳。
>`其实值等于pivot的元素在该次快速排序中，既可以随便出现在左半段，也可以随便出现在右半段，不用恰好紧挨在该次被作为pivot的元素周围。因为，随着之后对于左半段和右半段调用的快速排序，它们会各自被放到正确的位置上，这并不属于该次快速排序的职责。`

###### 3.2.2 分割方式2
curPoint从前往后遍历序列，parPoint指向小于基准与大于等于基准的序列的分割位置 —— 大于等于基准的序列的第一个元素。
当curPoint遍历结束，将pivot与parPoint位置的元素交换。

![分割方式2](https://pic.tanuki233.com/quicksort_ver2.png?imageView/2/w/358)
> 之后以此类推
> 标红的元素为pivot

    // 对left+1和right-2之间的范围进行分割
    int curPoint = left + 1;
    int parPoint = left + 1;
    T pivot = sequ[right - 1];
    
    while(curPoint < right - 1) {
        if(sequ[curPoint].compareTo(pivot) < 0) {
            swap(sequ, curPoint, parPoint);
            parPoint++;
        }
        curPoint++;
    }
    swap(sequ, parPoint, right - 1);


#### 4. 对pivot的左半段和右半段序列分别进行快速排序
##### 4.1 递归

    int partionPoint = partition(sequ, left, right);
    if(partionPoint < 0) {
        return;
    }
    executeProcess(sequ, left, partionPoint - 1);
    executeProcess(sequ, partionPoint + 1, right);

##### 4.2 非递归
采用栈保存下次要进行分割的序列首尾位置，深度优先。

    Stack<Integer> stack = new Stack<Integer>();

    int partionPoint = partition(sequ, left, right);
    if(partionPoint < 0) {
        return;
    }
    
    stack.push(partionPoint + 1);
    stack.push(right);
    
    stack.push(left);
    stack.push(partionPoint - 1);
    
    while(!stack.isEmpty()) {
        int sRight = stack.pop();
        int sLeft = stack.pop();
    
        partionPoint = partition(sequ, sLeft, sRight);
        if(partionPoint < 0) {
            continue;
        }
    
        stack.push(partionPoint + 1);
        stack.push(sRight);
    
        stack.push(sLeft);
        stack.push(partionPoint - 1);
    }


#### About Error
- 若产生无限循环，则问题可能出在两个方面：一个是递归终止条件；另一个是分割序列处的循环，尤其注意forePoint和backPoint同时分别遇到与基准值相等的元素时，forePoint和backPoint的移动情况
- 若没有正确排序，由于结果基本有序，我们可以从错误序列中看出端倪。如以下情况：

    Before:
    8 3 15 13 2 0 0 5 10 2 1 9 7 3 9 10 15 5 8 2 9 12 1 8 10 
    After:
    0 0 1 1 2 2 2 3 3 5 5 7 8 8 9 8 9 9 10 10 10 12 13 15 15

共有25个元素，下标14和15位置的元素没有正确排序。25的分割沿着出错位置依次为
0~11 12 13~24； 13~17 18 19~24; 13~14 15 16~17。 即可知道是13~17这次快速排序发生差错，从而进行仔细调试。


#### More
[3] 中通过尾递归对快速排序C语言版优化。关于尾递归，[4]讲述得比较明了。然而Java没有实现尾递归优化。相对的，我们只能采取避免递归过深或者用迭代取代递归的方式。
> It's important to note that this isn't a bug in the JVM. It's an optimization that can be implemented to help functional programmers who use recursion, which is much more common and normal in those languages. I recently spoke to Brian Goetz at Oracle about this optimization, and he said that it's on a list of things to be added to the JVM, but it's just not a high-priority item. For now, it's best to make this optimization yourself, if you can, by avoiding deeply recursive functions when coding a functional language on the JVM.

##### [DualPivotQuicksort](http://codeblab.com/wp-content/uploads/2009/09/DualPivotQuicksort.pdf)
Java中对于基本数据类型的排序算法通过DualPivotQuicksort实现。它有如下特性：This algorithm offers O(n log(n)) performance on many data sets that cause other quicksorts to degrade to quadratic performance, and is typically faster than traditional (one-pivot) Quicksort implementations.
![DualPivotQuicksort](https://i.stack.imgur.com/NhkeU.png)

排序方式具体如下：
1. For small arrays (length < 17), use the Insertion sort algorithm.
2. Choose two pivot elements P1 and P2. We can get, for example, the first element a[left] as P1 and the last element a[right] as P2.
3. P1 must be less than P2, otherwise they are swapped. So, there are the following parts:
* part I with indices from left+1 to L–1 with elements, which are less than P1,
* part II with indices from L to K–1 with elements, which are greater or equal to P1 and less or equal to P2,
* part III with indices from G+1 to right–1 with elements greater than P2,
* part IV contains the rest of the elements to be examined with indices from K to G.
4. The next element a[K] from the part IV is compared with two pivots P1 and P2, and placed to the corresponding part I, II, or III.
5. The pointers L, K, and G are changed in the corresponding directions.
6. The steps 4 - 5 are repeated while K ≤ G.
7. The pivot element P1 is swapped with the last element from part I, the pivot element P2 is swapped with the first element from part III.
8. The steps 1 - 7 are repeated recursively for every part I, part II, and part III.

##### 性能比较

|      | 基准选择   | 分割策略  | 递归？  | 插排优化？ |
| ---- | ------ | ----- | ---- | ----- |
| ver1 | 三数中值分割 | 分割方式1 | 递归   | 否     |
| ver2 | 三数中值分割 | 分割方式2 | 递归   | 否     |
| ver3 | 三数中值分割 | 分割方式1 | 非递归  | 否     |

|        | ver1    | ver2     | ver3    |
| ------ | ------- | -------- | ------- |
| Round1 | 8283115 | 11229782 | 2312889 |
| Round2 | 2574668 | 4175557  | 2521779 |
| Round3 | 2995112 | 2246667  | 3599113 |

> 单位：nanosecond
> 环境：CPU 4核8线程 2.30GHZ
> 测试序列：长度100范围0~100的随机数序列

[代码地址](https://github.com/BitterPotato/Algorithm/tree/master/src/sort)

#### 参考文献
1 Mark Allen Weiss[美]. 数据结构与算法分析: Java语言描述:第2版[M]. 机械工业出版社, 2012.
2 ThomasH.Cormen…. 算法导论:第2版[M]. 机械工业出版社, 2007.
3 http://blog.csdn.net/insistgogo/article/details/7785038
4 http://www.ruanyifeng.com/blog/2015/04/tail-call.html
5 http://stackoverflow.com/questions/20917617/whats-the-difference-of-dual-pivot-quick-sort-and-quick-sort