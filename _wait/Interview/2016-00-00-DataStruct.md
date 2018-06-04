[TOC]


# 数据结构

## Linked List

- 链表即是由节点（Node）组成的线性集合，每个节点可以利用指针指向其他节点。它是一种包含了多个节点的，能够用于表示序列的数据结构。
- Singly-linked list: 链表中的节点仅指向下一个节点。
- Doubly-linked list: 链表中的节点不仅指向下一个节点，还指向前一个节点。
- 时间复杂度:
    - 索引: O(n)
    - 搜索: O(n)
    - 插入: O(1)
    - 移除: O(1)

## Stack

- 栈是元素的集合，其包含了两个基本操作：push 操作可以用于将元素压入栈，pop 操作可以将栈顶元素移除。
- 遵循后入先出（LIFO）原则。
- 时间复杂度:
    - 索引: O(n)
    - 搜索: O(n)
    - 插入: O(1)
    - 移除: O(1) 

## Queue

- 队列是元素的集合，其包含了两个基本操作：enqueue 操作可以用于将元素插入到队列中，而 dequeeu 操作则是将元素从队列中移除。
- 遵循先入先出原则 (FIFO)。
- 时间复杂度:
    - 索引: O(n)
    - 搜索: O(n)
    - 插入: O(1)
    - 移除: O(1)

## Hashing

- 哈希能够将任意长度的数据映射到固定长度的数据。哈希函数返回的即是哈希值，如果两个不同的键得到相同的哈希值，即将这种现象称为碰撞。
- Hash Map: Hash Map 是一种能够建立起键与值之间关系的数据结构，Hash Map 能够使用哈希函数将键转化为桶或者槽中的下标，从而优化对于目标值的搜索速度。
- 碰撞解决
    - 链地址法（Separate Chaining）: 链地址法中，每个桶是相互独立的，包含了一系列索引的列表。搜索操作的时间复杂度即是搜索桶的时间（固定时间）与遍历列表的时间之和。
    - 开地址法（Open Addressing）: 在开地址法中，当插入新值时，会判断该值对应的哈希桶是否存在，如果存在则根据某种算法依次选择下一个可能的位置，直到找到一个尚未被占用的地址。所谓开地址法也是指某个元素的位置并不永远由其哈希值决定。

<center>**开地址法**</center>
<center>![开地址法](https://github.com/kdn251/interviews/raw/master/Images/hash.png?raw=true)</center>


## Graph
图是一种数据元素间为多对多关系的数据结构,加上一组基本操作构成的抽象数据类型。

- 无向图（Undirected Graph）: 无向图具有对称的邻接矩阵，因此如果存在某条从节点 u 到节点 v 的边，反之从 v 到 u 的边也存在。
- 有向图（Directed Graph）: 有向图的邻接矩阵是非对称的，即如果存在从 u 到 v 的边并不意味着一定存在从 v 到 u 的边。
- 如果一个有向图从任意顶点出发无法经过若干条边回到该点，则这个图是一个有向无环图（DAG图）。因为有向图中一个点经过两种路线到达另一个点未必形成环，因此有向无环图未必能转化成树，但任何有向树均为有向无环图。

<center>**有向有环图**</center>
<center>![有向有环图](https://github.com/kdn251/interviews/raw/master/Images/graph.png?raw=true)</center>

<center>**有向无环图**</center>
<center>![有向无环图](https://upload.wikimedia.org/wikipedia/commons/thumb/3/39/Directed_acyclic_graph_3.svg/356px-Directed_acyclic_graph_3.svg.png)</center>

## Tree
树即是无向非循环图。

- 无序树:树中任意节点的子节点之间没有顺序关系，这种树称为无序树，也称为自由树；
- 有序树:树中任意节点的子节点之间有顺序关系，这种树称为有序树；
    + 二叉树:每个节点最多含有两个子树的树称为二叉树
        * 完全二叉树：对于一颗二叉树，假设其深度为d（d>1）。除了第d层外，其它各层的节点数目均已达最大值，且第d层所有节点从左向右连续地紧密排列，这样的二叉树被称为完全二叉树；
            * 满二叉树：所有叶节点都在最底层的完全二叉树；
        * [平衡二叉树（AVL树）](#jump1)：当且仅当任何节点的两棵子树的高度差不大于1的二叉树；
            * [红黑树（英语：Red–black tree）](#jump3):是一种自平衡二叉查找树
        + [排序二叉树(二叉查找树（英语：Binary Search Tree），也称二叉搜索树、有序二叉树)](#jump2)
    + [霍夫曼树](#jump4)：带权路径最短的二叉树称为哈夫曼树或最优二叉树；
    + [B树](#jump5)：一种对读写操作进行优化的自平衡的二叉查找树，能够保持数据有序，拥有多余两个子树。

### 1.<span id="jump1">平衡二叉树</span>
<center>![](http://pic002.cnblogs.com/images/2012/214741/2012072218213884.png)</center>

### 2.<span id="jump2">二叉查找树（英语：Binary Search Tree）</span>
二叉查找树（英语：Binary Search Tree），也称二叉搜索树、有序二叉树（英语：ordered binary tree），排序二叉树（英语：sorted binary tree），是指一棵空树或者具有下列性质的二叉树：

- 若任意节点的左子树不空，则左子树上所有结点的值均小于它的根结点的值；
- 若任意节点的右子树不空，则右子树上所有结点的值均大于它的根结点的值；
- 任意节点的左、右子树也分别为二叉查找树；
- 没有键值相等的节点。

<center>![](https://upload.wikimedia.org/wikipedia/commons/thumb/d/da/Binary_search_tree.svg/150px-Binary_search_tree.svg.png)</center>

二叉查找树相比于其他数据结构的优势在于查找、插入的时间复杂度较低。为O(log n)。二叉查找树是基础性数据结构，用于构建更为抽象的数据结构，如集合、multiset、关联数组等。

时间复杂度:

- 索引: O(log(n))
- 搜索: O(log(n))
- 插入: O(log(n))
- 删除: O(log(n))


### 3.<span id="jump3">红黑树</span>
红黑树是每个节点都带有颜色属性的二叉查找树，颜色为红色或黑色。在二叉查找树强制一般要求以外，对于任何有效的红黑树我们增加了如下的额外要求：

- 节点是红色或黑色。
- 根是黑色。
- 所有叶子都是黑色（叶子是NIL节点）。
- 每个红色节点必须有两个黑色的子节点。（从每个叶子到根的所有路径上不能有两个连续的红色节点。）
- 从任一节点到其每个叶子的所有简单路径都包含相同数目的黑色节点。
<center>![](https://upload.wikimedia.org/wikipedia/commons/thumb/6/66/Red-black_tree_example.svg/450px-Red-black_tree_example.svg.png)</center>

在很多树数据结构的表示中，一个节点有可能只有一个子节点，而叶子节点包含数据。用这种范例表示红黑树是可能的，但是这会改变一些性质并使算法复杂。为此，本文中我们使用"nil叶子"或"空（null）叶子"，如上图所示，它不包含数据而只充当树在此结束的指示。这些节点在绘图中经常被省略，导致了这些树好像同上述原则相矛盾，而实际上不是这样。与此有关的结论是所有节点都有两个子节点，尽管其中的一个或两个可能是空叶子。

这些约束确保了红黑树的关键特性：从根到叶子的最长的可能路径不多于最短的可能路径的两倍长。结果是这个树大致上是平衡的。因为操作比如插入、删除和查找某个值的最坏情况时间都要求与树的高度成比例，这个在高度上的理论上限允许红黑树在最坏情况下都是高效的，而不同于普通的二叉查找树。

要知道为什么这些性质确保了这个结果，注意到性质4导致了路径不能有两个毗连的红色节点就足够了。最短的可能路径都是黑色节点，最长的可能路径有交替的红色和黑色节点。因为根据性质5所有最长的路径都有相同数目的黑色节点，这就表明了没有路径能多于任何其他路径的两倍长。

### 4.<span id="jump4">霍夫曼树</span>
霍夫曼树又称最优二叉树，是一种带权路径长度最短的二叉树。所谓树的带权路径长度，就是树中所有的叶结点的权值乘上其到根结点的路径长度（若根结点为0层，叶结点到根结点的路径长度为叶结点的层数）。树的路径长度是从树根到每一结点的路径长度之和，记为WPL=（W1*L1+W2*L2+W3*L3+...+Wn*Ln），N个权值Wi（i=1,2,...n）构成一棵有N个叶结点的二叉树，相应的叶结点的路径长度为Li（i=1,2,...n）。可以证明霍夫曼树的WPL是最小的。

霍夫曼编码（英语：Huffman Coding），又译为哈夫曼编码、赫夫曼编码，是一种用于无损数据压缩的熵编码（权编码）算法。

<center>![](https://upload.wikimedia.org/wikipedia/commons/thumb/8/82/Huffman_tree_2.svg/350px-Huffman_tree_2.svg.png)</center>

<center>这个句子“this is an example of a huffman tree”中得到的字母频率来建构霍夫曼树。句中字母的编码和频率如图所示。编码此句子需要135 bit（不包括保存树所用的空间）</center>

### 5.<span id="jump5">B树</span>
B树（英语：B-tree）是一种自平衡的树，能够保持数据有序。这种数据结构能够让查找数据、顺序访问、插入数据及删除的动作，都在对数时间内完成。B树，概括来说是一个一般化的二叉查找树（binary search tree），可以拥有多于2个子节点。与自平衡二叉查找树不同，B树为系统大块数据的读写操作做了优化。B树减少定位记录时所经历的中间过程，从而加快存取速度。B树这种数据结构可以用来描述外部存储。这种数据结构常被应用在数据库和文件系统的实作上。

<center>![](https://upload.wikimedia.org/wikipedia/commons/thumb/6/65/B-tree.svg/400px-B-tree.svg.png)</center>

## Heap
堆是一种特殊的基于树的满足某些特性的数据结构，整个堆中的所有父子节点的键值都会满足相同的排序条件。堆更准确地可以分为最大堆与最小堆:

- 在最大堆中，父节点的键值永远大于或者等于子节点的值，并且整个堆中的最大值存储于根节点；
- 而最小堆中，父节点的键值永远小于或者等于其子节点的键值，并且整个堆中的最小值存储于根节点。

时间复杂度:

- 访问: O(log(n))
- 搜索: O(log(n))
- 插入: O(log(n))
- 移除: O(log(n))
- 移除最大值 / 最小值: O(1)

<center>**最大堆**</center>
<center>![](https://github.com/kdn251/interviews/raw/master/Images/heap.png?raw=true)</center>

## 参考
1. [Data Structures](https://github.com/kdn251/interviews)
2. [6天通吃树结构—— 第二天 平衡二叉树](http://www.cnblogs.com/huangxincheng/archive/2012/07/22/2603956.html)
3. [树结构](https://zh.wikipedia.org/wiki/Category:%E6%A0%91%E7%BB%93%E6%9E%84)
4. [图论](https://zh.wikipedia.org/wiki/Category:%E5%9B%BE%E8%AE%BA)
5. [算法实践](https://zhuanlan.zhihu.com/p/25719965)