---
layout: post
title: Base-Algorithms
categories: [Base]
description: 简单工厂属于创造模型类
keywords: Base
---

## Sorting Algorithms

- 比较排序包括：
    - 快速排序
    - 堆排序
    - 归并排序
    - 插入排序
    - 选择排序
    - 冒泡排序
- 非比较排序包括：
    - 基数排序
    - 计数排序
    - 桶排序

### Bubble Sort
冒泡排序（英语：Bubble Sort，台湾另外一种译名为：泡沫排序）是一种简单的排序算法。每次求出未排序部分的最大值或者最小值.冒泡排序算法的运作如下：

1. 比较相邻的元素。如果第一个比第二个大，就交换他们两个。
2. 对每一对相邻元素作同样的工作，从开始第一对到结尾的最后一对。这步做完后，最后的元素会是最大的数。
3. 针对所有的元素重复以上的步骤，除了最后一个。
4. 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

### Insertion Sort
插入排序（英语：Insertion Sort）是一种简单直观的排序算法。它的工作原理是通过构建有序序列，对于未排序数据，在已排序序列中从后向前扫描，找到相应位置并插入。

一般来说，插入排序都采用in-place在数组上实现。具体算法描述如下：
1. 从第一个元素开始，该元素可以认为已经被排序
2. 取出下一个元素，在已经排序的元素序列中从后向前扫描
3. 如果该元素（已排序）大于新元素，将该元素移到下一位置
4. 重复步骤3，直到找到已排序的元素小于或者等于新元素的位置
5. 将新元素插入到该位置后
6. 重复步骤2~5

<center>![](https://upload.wikimedia.org/wikipedia/commons/thumb/0/0f/Insertion-sort-example-300px.gif/220px-Insertion-sort-example-300px.gif)</center>
<center>使用插入排序为一列数字进行排序的过程</center>

### Selection sort
选择排序（Selection sort）是一种简单直观的排序算法。它的工作原理如下。首先在未排序序列中找到最小（大）元素，存放到排序序列的起始位置，然后，再从剩余未排序元素中继续寻找最小（大）元素，然后放到已排序序列的末尾。以此类推，直到所有元素均排序完毕。

<center>![](https://upload.wikimedia.org/wikipedia/commons/9/94/Selection-Sort-Animation.gif)</center>
<center>使用插入排序为一列数字进行排序的过程</center>

### Quicksort
快速排序使用分治法（Divide and conquer）策略来把一个序列（list）分为两个子序列（sub-lists）。步骤为：

1. 从数列中挑出一个元素，称为"基准"（pivot），
2. 重新排序数列，所有元素比基准值小的摆放在基准前面，所有元素比基准值大的摆在基准的后面（相同的数可以到任一边）。在这个分区结束之后，该基准就处于数列的中间位置。这个称为分区（partition）操作。
3. 递归地（recursive）把小于基准值元素的子数列和大于基准值元素的子数列排序。

- 稳定: 否
- 时间复杂度:
    + 最优时间: O(nlog(n))
    + 最坏时间: O(n^2)
    + 平均时间: O(nlog(n)

<center>![](https://upload.wikimedia.org/wikipedia/commons/6/6a/Sorting_quicksort_anim.gif)</center>



### Mergesort
归并排序（英语：Merge sort，或mergesort）是采用分治法（Divide and Conquer）的一个非常典型的应用，且各层分治递归可以同时进行。指的是将两个已经排序的序列合并成一个序列的操作。归并排序算法依赖归并操作。

<center>![](https://upload.wikimedia.org/wikipedia/commons/c/cc/Merge-sort-example-300px.gif)</center>
<center>使用插入排序为一列数字进行排序的过程</center>

<center>![](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c5/Merge_sort_animation2.gif/220px-Merge_sort_animation2.gif)</center>
<center>一个归并排序的例子：对一个随机点的链表进行排序</center>

### Heapsort
堆排序（Heapsort）是指利用堆这种数据结构所设计的一种排序算法。堆积是一个近似完全二叉树的结构，并同时满足堆积的性质：即子结点的键值或索引总是小于（或者大于）它的父节点。

#### 堆节点的访问:
通常堆是通过一维数组来实现的。在数组起始位置为0的情形中：

- 父节点i的左子节点在位置(2*i+1);
- 父节点i的右子节点在位置(2*i+2);
- 子节点i的父节点在位置floor((i-1)/2)

<center>![](https://upload.wikimedia.org/wikipedia/commons/1/1b/Sorting_heapsort_anim.gif)</center>
<center>堆排序算法的演示。首先，将元素进行重排，以匹配堆的条件。图中排序过程之前简单的绘出了堆树的结构。</center>

#### 堆的操作
在堆的数据结构中，堆中的最大值总是位于根节点(在优先队列中使用堆的话堆中的最小值位于根节点)。堆中定义以下几种操作：

- 最大堆调整（Max_Heapify）：将堆的末端子节点作调整，使得子节点永远小于父节点
- 创建最大堆（Build_Max_Heap）：将堆所有数据重新排序
- 堆排序（HeapSort）：移除位在第一个数据的根节点，并做最大堆调整的递归运算

### Bucket sort
桶排序（Bucket sort）或所谓的箱排序，是一个排序算法，工作的原理是将数组分到有限数量的桶里。每个桶再个别排序（有可能再使用别的排序算法或是以递归方式继续使用桶排序进行排序）。桶排序是鸽巢排序的一种归纳结果。当要被排序的数组内的数值是均匀分配的时候，桶排序使用线性时间（Θ(n)）。但桶排序并不是比较排序，他不受到O(n log n)下限的影响。

桶排序以下列程序进行：

1. 设置一个定量的数组当作空桶子。
2. 寻访序列，并且把项目一个一个放到对应的桶子去。
3. 对每个不是空的桶子进行排序。
4. 从不是空的桶子里把项目再放回原来的序列中。

时间复杂度： 

- 最佳情况：Ω（n + k） 
- 最坏情况：O（n ^ 2） 
- 平均情况：Θ（n + k）

<center>![](https://github.com/kdn251/interviews/raw/master/Images/bucketsort.png?raw=true)</center>

### Counting sort
计数排序（Counting sort）是一种稳定的线性时间排序算法。计数排序使用一个额外的数组C，其中第i个元素是待排序数组A中值等于i的元素的个数。然后根据数组C来将A中的元素排到正确的位置。

由于用来计数的数组C的长度取决于待排序数组中数据的范围（等于待排序数组的最大值与最小值的差加上1），这使得计数排序对于数据范围很大的数组，需要大量时间和内存。例如：计数排序是用来排序0到100之间的数字的最好的算法，但是它不适合按字母顺序排序人名。但是，计数排序可以用在基数排序算法中，能够更有效的排序数据范围很大的数组。

通俗地理解，例如有10个年龄不同的人，统计出有8个人的年龄比A小，那A的年龄就排在第9位，用这个方法可以得到其他每个人的位置，也就排好了序。当然，年龄有重复时需要特殊处理（保证稳定性），这就是为什么最后要反向填充目标数组，以及将每个数字的统计减去1的原因。算法的步骤如下：

1. 找出待排序的数组中最大和最小的元素
2. 统计数组中每个值为i的元素出现的次数，存入数组 C 的第 i 项
3. 对所有的计数累加（从C中的第一个元素开始，每一项和前一项相加）
4. 反向填充目标数组：将每个元素i放在新数组的第C(i)项，每放一个元素就将C(i)减去1

### Radix sort
基数排序（英语：Radix sort）是一种非比较型整数排序算法，其原理是将整数按位数切割成不同的数字，然后按每个位数分别比较。由于整数也可以表达字符串（比如名字或日期）和特定格式的浮点数，所以基数排序也不是只能使用于整数。

它是这样实现的：将所有待比较数值（正整数）统一为同样的数位长度，数位较短的数前面补零。然后，从最低位开始，依次进行一次排序。这样从最低位排序一直到最高位排序完成以后，数列就变成一个有序序列。

## Graph Algorithms

### Depth-First-Search
深度优先搜索算法（英语：Depth-First-Search，简称DFS）是一种用于遍历或搜索树或图的算法。沿着树的深度遍历树的节点，尽可能深的搜索树的分支。当节点v的所在边都己被探寻过，搜索将回溯到发现节点v的那条边的起始节点。这一过程一直进行到已发现从源节点可达的所有节点为止。如果还存在未被发现的节点，则选择其中一个作为源节点并重复以上过程，整个进程反复进行直到所有节点都被访问为止。属于盲目搜索。

时间复杂度：  O(b^m)
空间复杂度：  O(bm)

### Breadth-First-Search
广度优先搜索算法（英语：Breadth-First-Search，缩写为BFS）是优先遍历邻接节点而不是子节点的图遍历算法。

<center>![](https://github.com/kdn251/interviews/raw/master/Images/dfsbfs.gif?raw=true)</center>

时间复杂度：O(|V|+|E|) = O(b^d)
空间复杂度：O(|V|+|E|) = O(b^d)

## Topological Sort

## 参考
1. [Data Structures](https://github.com/kdn251/interviews)
2. [排序算法](https://zh.wikipedia.org/wiki/Category:%E6%8E%92%E5%BA%8F%E7%AE%97%E6%B3%95)