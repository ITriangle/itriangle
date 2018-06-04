---
layout: post
title: Spark Mlib
categories: [Spark,大数据]
description: Spark Mlib 作为 Spark 机器学习的核心组件
keywords: 大数据,Spark,Java
---

# Spark 机器学习库简介
Spark 机器学习库提供了常用机器学习算法的实现,包括聚类,分类,回归,协同过滤,维度缩减等.使用 Spark 机器学习库来做机器学习工作,可以说是非常的简单,通常只需要在对原始数据进行处理后,然后直接调用相应的 API 就可以实现.但是要想选择合适的算法,高效准确地对数据进行分析,您可能还需要深入了解下算法原理,以及相应 Spark MLlib API 实现的参数的意义.

需要提及的是,Spark 机器学习库从 1.2 版本以后被分为两个包,分别是：

**spark.mllib**
Spark MLlib 历史比较长了,1.0 以前的版本中已经包含了,提供的算法实现都是基于原始的 RDD,从学习角度上来讲,其实比较容易上手.如果您已经有机器学习方面的经验,那么您只需要熟悉下 MLlib 的 API 就可以开始数据分析工作了.想要基于这个包提供的工具构建完整并且复杂的机器学习流水线是比较困难的.

**spark.ml**

Spark ML Pipeline 从 Spark1.2 版本开始,目前已经从 Alpha 阶段毕业,成为可用并且较为稳定的新的机器学习库.ML Pipeline 弥补了原始 MLlib 库的不足,向用户提供了一个基于 DataFrame 的机器学习工作流式 API 套件,使用 ML Pipeline API,我们可以很方便的把数据处理,特征转换,正则化,以及多个机器学习算法联合起来,构建一个单一完整的机器学习流水线.显然,这种新的方式给我们提供了更灵活的方法,而且这也更符合机器学习过程的特点.

从官方文档来看,Spark ML Pipeline 虽然是被推荐的机器学习方式,但是并不会在短期内替代原始的 MLlib 库,因为 MLlib 已经包含了丰富稳定的算法实现,并且部分 ML Pipeline 实现基于 MLlib.而且就笔者看来,并不是所有的机器学习过程都需要被构建成一个流水线,有时候原始数据格式整齐且完整,而且使用单一的算法就能实现目标,我们就没有必要把事情复杂化,采用最简单且容易理解的方式才是正确的选择.

## Date Type
MLlib 支持 local vectors 和 matrices 存储在单个机器上,以及由一个或多个RDD支持的 distributed matrices. 局部向量和局部矩阵是用作公共接口的简单数据模型. 基本线性代数运算由Breeze提供. 在监督学习中使用的训练示例在MLlib中被称为“标记点(labeled point)”.

### 知识前提
稀疏向量和密集向量都是向量的表示方法

密集向量和稀疏向量的区别：

1. 密集向量的值就是一个普通的`Double`数组
2. 稀疏向量由两个并列的数组`indices`和`values`组成

例如:向量`(1.0,0.0,1.0,3.0)`分别用密集向量和稀疏向量表示

1. 用密集向量表示为 `[1.0,0.0,1.0,3.0]`
2. 用稀疏格式表示为 `(4,[0,2,3],[1.0,1.0,3.0])` 第一个`4`表示向量的长度(元素个数),`[0,2,3]`就是`indices`数组,`[1.0,1.0,3.0]`是`values`数组 表示向量`0`的位置的值是`1.0`,`2`的位置的值是`1.0`,而`3`的位置的值是`3.0`,其他的位置都是`0`.

**稀疏向量** 
通常用两部分表示：一部分是顺序向量,另一部分是值向量.例如稀疏向量(4,0,28,53,0,0,4,8)可用值向量(4,28,53,4,8)和顺序向量(1,0,1,1,0,0,1,1)表示.

在数值分析中,**稀疏矩阵(Sparse matrix)**,是其元素大部分为零的矩阵.反之,如果大部分元素都非零,则这个矩阵是 **稠密矩阵(Dense  matrix)**的.在科学与工程领域中求解线性模型时经常出现大型的稀疏矩阵.

### Local vector
局部向量是数组的结构,值为 `double`,保存在单个机器上. Mlib 支持两种类型的局部向量: 密集向量和稀疏向量.

局部向量的基类是Vector,我们提供了两个实现：DenseVector和SparseVector.我们建议使用在Vectors中实现的工厂方法创建局部向量.

```java
import org.apache.spark.mllib.linalg.Vector;
import org.apache.spark.mllib.linalg.Vectors;

// Create a dense vector (1.0, 0.0, 3.0).
Vector dv = Vectors.dense(1.0, 0.0, 3.0);
// Create a sparse vector (1.0, 0.0, 3.0) by specifying its indices and values corresponding to nonzero entries.
Vector sv = Vectors.sparse(3, new int[] {0, 2}, new double[] {1.0, 3.0});
```

### Labeled point
Labeled point 是与标记/响应相关联的稠密或稀疏的局部向量. 在MLlib中,在监督学习算法中使用 Labeled point . 我们使用 double 来存储标签,因此我们可以在回归和分类中使用标记点. 对于二进制分类,标签应为0(负)或1(正). 对于多类分类,标签应该是从零开始的类索引：0,1,2,.... 

标记点由 `LabeledPoint` 表示.

```java
import org.apache.spark.mllib.linalg.Vectors;
import org.apache.spark.mllib.regression.LabeledPoint;

// Create a labeled point with a positive label and a dense feature vector.
LabeledPoint pos = new LabeledPoint(1.0, Vectors.dense(1.0, 0.0, 3.0));

// Create a labeled point with a negative label and a sparse feature vector.
LabeledPoint neg = new LabeledPoint(0.0, Vectors.sparse(3, new int[] {0, 2}, new double[] {1.0, 3.0}));
```

*Sparse data*
在实践中非常常见的是具有稀疏的训练数据. MLlib支持阅读以LIBSVM格式存储的训练示例,LIBSVM格式是LIBSVM和LIBLINEAR使用的默认格式. 它是一种文本格式,其中每行代表使用以下格式:*带标签的稀疏特征向量*

```
label index1:value1 index2:value2 ...
```

其中索引是基于1的并且以升序排列.加载后,特征索引将转换为零.

`MLUtils.loadLibSVMFile` 读取以 LIBSVM 格式存储的训练示例.

```java
import org.apache.spark.mllib.regression.LabeledPoint;
import org.apache.spark.mllib.util.MLUtils;
import org.apache.spark.api.java.JavaRDD;

JavaRDD<LabeledPoint> examples = 
  MLUtils.loadLibSVMFile(jsc.sc(), "data/mllib/sample_libsvm_data.txt").toJavaRDD();

```

### Labeled Matrix
局部矩阵具有整数类型的行和列索引,double 类型的value,存储在单个机器上. MLlib支持稠密矩阵,其实体值存储在单个 double类型 数组中,存储顺序以列为主.稀疏矩阵的非零实体值以列为主的顺序存储在压缩稀疏列(CSC)格式中. 例如,以下稠密矩阵:
$$
  \begin{Bmatrix}
   1.0 & 2.0 \\
   3.0 & 4.0 \\
   5.0 & 6.0
  \end{Bmatrix} 
$$

被存储在具有矩阵大小`(3,2)`的一维数组 `[1.0,3.0,5.0,2.0,4.0,6.0]`中.


局部矩阵的父类是 Matrix, 并且我们提供了两种实现: DenseMatrix 和 SparseMatrix.我们建议使用官方的工厂方法分实现去创建局部矩阵.记住,局部矩阵在 MLib 中是按照列为主(column-major)存储的.

```java
import org.apache.spark.mllib.linalg.Matrix;
import org.apache.spark.mllib.linalg.Matrices;

// Create a dense matrix ((1.0, 2.0), (3.0, 4.0), (5.0, 6.0))
Matrix dm = Matrices.dense(3, 2, new double[] {1.0, 3.0, 5.0, 2.0, 4.0, 6.0});

// Create a sparse matrix ((9.0, 0.0), (0.0, 8.0), (0.0, 6.0))
Matrix sm = Matrices.sparse(3, 2, new int[] {0, 1, 3}, new int[] {0, 2, 1}, new double[] {9, 6, 8});
```



### Distributed matrix
分布式矩阵有 long 类型的行和列索引,以及 double 类型的 values,分布式的保存在一个或者多个 RDD 中.选择合适的格式去存储大的分布式的矩阵,这是重要的.转化一个分布式的矩阵到另外一种格式也许需要一个全局的 shuffle,这需要高昂的消耗.到目前为止,有三种分布式的矩阵已经别实现了.


基础的类型叫做 RowMatrix, RowMatrix 是行索引没有意义的面向于行做分布式的矩阵.例如特征向量的集合,它有行 RDD 支持,其中每行都是一个局部向量.我们假定 RowMatrix 的列数据不是很大,面向于行做分布式之后,每一个局部向量都可以被驱动,并且在单个节点上面保存和执行.

IndexRowMatrix 是具有行索引的 RowMatrix,行索引可以用于识别行和执行关联操作.

CoordinateMatrix 是存储在坐标列表(COO)格式中的分布式矩阵,其中的实体被 RDD 支持.

注意: 分布式矩阵的基础RDD,必须是确定性的.因为我们需要缓存矩阵大小.一般使用不确定的 RDD可能会导致错误.

#### RowMatrix
A RowMatrix is a row-oriented distributed matrix without meaningful row indices, backed by an RDD of its rows, where each row is a local vector. Since each row is represented by a local vector, the number of columns is limited by the integer range but it should be much smaller in practice.

RowMatrix 是行索引没有意义,面向行做分布式的矩阵.它的行通过 RDD 支持.每一个行是一格局部向量.虽然每一个通过局部向量呈现,但是列大小应该被限定在一个整数范围内,实际情况会小得多.

RowMatrix 通过 `JavaRDD<Vector>` 创建实例.然后我们可以计算其列摘要统计.

```java
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.mllib.linalg.Vector;
import org.apache.spark.mllib.linalg.distributed.RowMatrix;

JavaRDD<Vector> rows = ... // a JavaRDD of local vectors
// Create a RowMatrix from an JavaRDD<Vector>.
RowMatrix mat = new RowMatrix(rows.rdd());

// Get its size.
long m = mat.numRows();
long n = mat.numCols();

// QR decomposition 
QRDecomposition<RowMatrix, Matrix> result = mat.tallSkinnyQR(true);
```

#### IndexedRowMatrix

An IndexedRowMatrix is similar to a RowMatrix but with meaningful row indices. It is backed by an RDD of indexed rows, so that each row is represented by its index (long-typed) and a local vector.

IndexRowMatrix 与 RowMatrix 及其相似,只是 IndexRowMatrix 的列索引是有意义的.它被索引行的 RDD 支持.以便每行通过索引(整型) 和 局部向量表示. 

IndexedRowMatrix 可以从 `JavaRDD <IndexedRow>`创建实例,其中 `IndexedRow` 是一个包装器(long,Vector). IndexedRowMatrix 可以通过删除其行索引转换为 RowMatrix.

```java
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.mllib.linalg.distributed.IndexedRow;
import org.apache.spark.mllib.linalg.distributed.IndexedRowMatrix;
import org.apache.spark.mllib.linalg.distributed.RowMatrix;

JavaRDD<IndexedRow> rows = ... // a JavaRDD of indexed rows
// Create an IndexedRowMatrix from a JavaRDD<IndexedRow>.
IndexedRowMatrix mat = new IndexedRowMatrix(rows.rdd());

// Get its size.
long m = mat.numRows();
long n = mat.numCols();

// Drop its row indices.
RowMatrix rowMat = mat.toRowMatrix();
```

#### CoordinateMatrix

A CoordinateMatrix is a distributed matrix backed by an RDD of its entries. Each entry is a tuple of (i: Long, j: Long, value: Double), where i is the row index, j is the column index, and value is the entry value. A CoordinateMatrix should be used only when both dimensions of the matrix are huge and the matrix is very sparse.

CoordinateMatrix 是被实体 RDD 支持的分布式矩阵,每一个实体是一个元组(i: Long, j: Long, value: Double), i 是行索引, j 是列索引, value 是一个实体.  只有当矩阵的两个维度都很大并且矩阵非常稀疏时,才应使用 CoordinateMatrix.

CoordinateMatrix可以从`JavaRDD <MatrixEntry>`创建实例,其中MatrixEntry是一个包装器(long,long,double). CoordinateMatrix可以通过调用 `toIndexedRowMatrix` 转换为具有稀疏行的 IndexedRowMatrix. 当前不支持 CoordinateMatrix 的其他计算.

```java
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.mllib.linalg.distributed.CoordinateMatrix;
import org.apache.spark.mllib.linalg.distributed.IndexedRowMatrix;
import org.apache.spark.mllib.linalg.distributed.MatrixEntry;

JavaRDD<MatrixEntry> entries = ... // a JavaRDD of matrix entries
// Create a CoordinateMatrix from a JavaRDD<MatrixEntry>.
CoordinateMatrix mat = new CoordinateMatrix(entries.rdd());

// Get its size.
long m = mat.numRows();
long n = mat.numCols();

// Convert it to an IndexRowMatrix whose rows are sparse vectors.
IndexedRowMatrix indexedRowMatrix = mat.toIndexedRowMatrix();
```

#### BlockMatrix
BlockMatrix 是由 MatrixBlocks 的RDD支持的分布式矩阵,其中MatrixBlock是(Int,Int),Matrix)的元组,其中(Int,Int)是块的索引, Matrix 是一个给定大小为rowsPerBlock x colsPerBlock 的子矩阵. BlockMatrix 支持诸如与另一个BlockMatrix 相加和相乘的方法. BlockMatrix还有一个帮助验证函数,可用于检查BlockMatrix是否设置正确.

BlockMatrix可以通过调用 `toBlockMatrix` 从 IndexedRowMatrix 或 CoordinateMatrix 中最轻松创建. `toBlockMatrix` 默认创建大小为1024 x 1024的块. 用户可以通过设置` toBlockMatrix(rowsPerBlock,colsPerBlock)`来更改块大小.

```java
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.mllib.linalg.distributed.BlockMatrix;
import org.apache.spark.mllib.linalg.distributed.CoordinateMatrix;
import org.apache.spark.mllib.linalg.distributed.IndexedRowMatrix;

JavaRDD<MatrixEntry> entries = ... // a JavaRDD of (i, j, v) Matrix Entries
// Create a CoordinateMatrix from a JavaRDD<MatrixEntry>.
CoordinateMatrix coordMat = new CoordinateMatrix(entries.rdd());
// Transform the CoordinateMatrix to a BlockMatrix
BlockMatrix matA = coordMat.toBlockMatrix().cache();

// Validate whether the BlockMatrix is set up properly. Throws an Exception when it is not valid.
// Nothing happens if it is valid.
matA.validate();

// Calculate A^T A.
BlockMatrix ata = matA.transpose().multiply(matA);
```



## 参考
1. [Machine Learning Library (MLlib) Guide](https://spark.apache.org/docs/2.0.0-preview/mllib-guide.html)
2. [Data Types - MLlib](https://spark.apache.org/docs/2.0.0-preview/mllib-data-types.html)
3. [Spark 实战,第 4 部分: 使用 Spark MLlib 做 K-means 聚类分析](https://www.ibm.com/developerworks/cn/opensource/os-cn-spark-practice4/#ibm-pcon)