---
layout: post
title: Spark Mlib 提取转化选择特征
categories: [Spark,大数据]
description: Spark Mlib 作为 Spark 机器学习的核心组件
keywords: 大数据,Spark,Java
---

# Extracting, transforming and selecting features

本节介绍使用特征的算法，大致分为以下几类：

- 提取：从“原始”数据中提取特征
- 转换：缩放，转换或修改特征
- 选择：从较大的一组特征中选择一个子集
- 局部敏感哈希（LSH）：这类算法将特征变换的方面与其他算法相结合。

Table of Contents

- Feature Extractors
  - TF-IDF
  - Word2Vec
  - CountVectorizer
- Feature Transformers
  - Tokenizer
  - StopWordsRemover
  - nn-gram
  - Binarizer
  - PCA
  - PolynomialExpansion
  - Discrete Cosine Transform (DCT)
  - StringIndexer
  - IndexToString
  - OneHotEncoder
  - VectorIndexer
  - Interaction
  - Normalizer
  - StandardScaler
  - MinMaxScaler
  - MaxAbsScaler
  - Bucketizer
  - ElementwiseProduct
  - SQLTransformer
  - VectorAssembler
  - QuantileDiscretizer 
- Feature Selectors
  - VectorSlicer
  - RFormula
  - ChiSqSelector
- Locality Sensitive Hashing
  - LSH Operations
    - Feature Transformation
    - Approximate Similarity Join
    - Approximate Nearest Neighbor Search
  - LSH Algorithms
    - Bucketed Random Projection for Euclidean Distance
    - MinHash for Jaccard Distance

## Feature Extractors
Spark MLlib 提供三种文本特征提取方法，分别为TF-IDF、Word2Vec以及CountVectorizer其各自原理与调用代码整理如下：

###  TF-IDF
词频－逆向文件频率（TF-IDF）是一种在文本挖掘中广泛使用的特征向量化方法，它可以体现一个文档中词语在语料库中的重要程度。

词语由t表示，文档由d表示，语料库由D表示。词频TF(t,,d)是词语t在文档d中出现的次数。文件频率DF(t,D)是包含词语的文档的个数。如果我们只使用词频来衡量重要性，很容易过度强调在文档中经常出现而并没有包含太多与文档有关的信息的词语，比如“a”，“the”以及“of”。如果一个词语经常出现在语料库中，它意味着它并没有携带特定的文档的特殊信息。逆向文档频率数值化衡量词语提供多少信息：

$$
IDF(t, D) = \log \frac{|D| + 1}{DF(t, D) + 1}
$$

其中，|D|是语料库中的文档总数。由于采用了对数，如果一个词出现在所有的文件，其IDF值变为0。

$$
TFIDF(t, d, D) = TF(t, d) \cdot IDF(t, D).
$$

匹配频率和文档频率的定义有几个变体。在MLlib中，我们分离TF和IDF，使其灵活。

**TF:**HashingTF 和 CountVectorizer 均可用于生成单词频率向量。

HashingTF 是一个 Transformer ,它采用一个集合,并转换这个集合到固定长度的特征向量.在文本处理中,一个 "一条术语" 可能是一个词包.HashingTF利用哈希技巧,原始特征通过应用散列函数映射到索引（术语）。这里使用的哈希函数是MurmurHash 3。然后根据映射的索引计算术语频率。这种方法避免了计算全局术语对索引映射的需要，这对于大型语料库来说可能是昂贵的，但是它具有潜在的哈希冲突，其中不同的原始特征可以在散列之后变成相同的术语。为了减少碰撞的机会，我们可以增加目标特征维度，即哈希表的桶数。由于使用简单的模数将散列函数转换为列索引，建议使用两个幂作为特征维，否则不会将特征均匀地映射到列。默认特征维度为 $  2^{18} = 262,144 $.可选的二进制切换参数控制术语频率计数。当设置为true时，所有非零频率计数设置为1.对于模拟二进制而不是整数计数的离散概率模型，此操作特别有用。

CountVectorizer将文本文档转换为术语计数的向量。有关详细信息，请参阅C[ountVectorizer](http://spark.apache.org/docs/latest/ml-features.html#countvectorizer)。


**IDF:**IDF是一个适用数据集并生成IDFModel的 Estimator 。 IDFModel获取特征向量（通常由HashingTF或CountVectorizer创建）并缩放每列。直观地说，它下调了在语料库中频繁出现的列。

**Note:**spark.ml不提供文本分割的工具.我们将引导用户到  [Stanford NLP Group](http://nlp.stanford.edu/) 和 [scalanlp/chalk](https://github.com/scalanlp/chalk).

**Example**

在下面的代码段中，我们从一组句子开始。我们使用Tokenizer将每个句子分成单词。对于每个句子（包的单词），我们使用HashingTF将该句子哈希成特征向量。我们使用IDF来重新缩放特征向量;这通常会在使用文本作为特征时提高性能。然后，我们的特征向量可以被传递给学习算法。

```java
import java.util.Arrays;
import java.util.List;

import org.apache.spark.ml.feature.HashingTF;
import org.apache.spark.ml.feature.IDF;
import org.apache.spark.ml.feature.IDFModel;
import org.apache.spark.ml.feature.Tokenizer;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.types.DataTypes;
import org.apache.spark.sql.types.Metadata;
import org.apache.spark.sql.types.StructField;
import org.apache.spark.sql.types.StructType;

List<Row> data = Arrays.asList(
  RowFactory.create(0.0, "Hi I heard about Spark"),
  RowFactory.create(0.0, "I wish Java could use case classes"),
  RowFactory.create(1.0, "Logistic regression models are neat")
);
StructType schema = new StructType(new StructField[]{
  new StructField("label", DataTypes.DoubleType, false, Metadata.empty()),
  new StructField("sentence", DataTypes.StringType, false, Metadata.empty())
});
Dataset<Row> sentenceData = spark.createDataFrame(data, schema);

Tokenizer tokenizer = new Tokenizer().setInputCol("sentence").setOutputCol("words");
Dataset<Row> wordsData = tokenizer.transform(sentenceData);

int numFeatures = 20;
HashingTF hashingTF = new HashingTF()
  .setInputCol("words")
  .setOutputCol("rawFeatures")
  .setNumFeatures(numFeatures);

Dataset<Row> featurizedData = hashingTF.transform(wordsData);
// alternatively, CountVectorizer can also be used to get term frequency vectors

IDF idf = new IDF().setInputCol("rawFeatures").setOutputCol("features");
IDFModel idfModel = idf.fit(featurizedData);

Dataset<Row> rescaledData = idfModel.transform(featurizedData);
rescaledData.select("label", "features").show();
```


###  Word2Vec

Word2Vec is an Estimator which takes sequences of words representing documents and trains a Word2VecModel. The model maps each word to a unique fixed-size vector. The Word2VecModel transforms each document into a vector using the average of all words in the document; this vector can then be used as features for prediction, document similarity calculations, etc. Please refer to the MLlib user guide on Word2Vec for more details.
中文(简体)
Word2Vec是一个 Estimator，它采用表示文档的单词序列，并训练一个 Word2VecModel。该模型将每个单词映射到一个唯一的固定大小向量。 Word2VecModel使用文档中所有单词的平均值将每个文档转换为向量;然后该向量可用作预测，文档相似性计算等功能。有关更多详细信息，请参阅有关[Word2Vec的MLlib](http://spark.apache.org/docs/latest/mllib-feature-extraction.html#word2vec)用户指南。

```java
import java.util.Arrays;
import java.util.List;

import org.apache.spark.ml.feature.Word2Vec;
import org.apache.spark.ml.feature.Word2VecModel;
import org.apache.spark.ml.linalg.Vector;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.types.*;

// Input data: Each row is a bag of words from a sentence or document.
List<Row> data = Arrays.asList(
  RowFactory.create(Arrays.asList("Hi I heard about Spark".split(" "))),
  RowFactory.create(Arrays.asList("I wish Java could use case classes".split(" "))),
  RowFactory.create(Arrays.asList("Logistic regression models are neat".split(" ")))
);
StructType schema = new StructType(new StructField[]{
  new StructField("text", new ArrayType(DataTypes.StringType, true), false, Metadata.empty())
});
Dataset<Row> documentDF = spark.createDataFrame(data, schema);

// Learn a mapping from words to Vectors.
Word2Vec word2Vec = new Word2Vec()
  .setInputCol("text")
  .setOutputCol("result")
  .setVectorSize(3)
  .setMinCount(0);

Word2VecModel model = word2Vec.fit(documentDF);
Dataset<Row> result = model.transform(documentDF);

for (Row row : result.collectAsList()) {
  List<String> text = row.getList(0);
  Vector vector = (Vector) row.get(1);
  System.out.println("Text: " + text + " => \nVector: " + vector + "\n");
}
```

###  CountVectorizer
CountVectorizer和CountVectorizerModel旨在帮助将文本文档集合转换为令牌计数向量。当先验词典不可用时，CountVectorizer可以用作 Estimator 来提取词汇表，并生成CountVectorizerModel。该模型通过词汇生成文档的稀疏表示，然后可以将其传递给其他算法，如LDA。

在拟合过程中，CountVectorizer将选择通过语料库按术语频率排序的顶级vocabSize字。可选参数minDF还通过指定术语必须出现以包含在词汇表中的文档的最小数量（或小于1.0）来影响拟合过程。另一个可选的二进制切换参数控制输出向量。如果设置为true，则所有非零计数都设置为1.对于模拟二进制而不是整数的离散概率模型，这是非常有用的。

**Examples**
假设我们有以下DataFrame,具有列:ID和文本

```
 id | texts
----|----------
 0  | Array("a", "b", "c")
 1  | Array("a", "b", "b", "c", "a")
```

文本中的每一行都是Array [String]类型的文档。调用CountVectorizer的拟合产生一个具有词汇表（a，b，c）的CountVectorizerModel。然后变换后的输出列“vector”包含:

```
 id | texts                           | vector
----|---------------------------------|---------------
 0  | Array("a", "b", "c")            | (3,[0,1,2],[1.0,1.0,1.0])
 1  | Array("a", "b", "b", "c", "a")  | (3,[0,1,2],[2.0,2.0,1.0])
```

每个向量表示文档在词汇表上的令牌计数。

```java
import java.util.Arrays;
import java.util.List;

import org.apache.spark.ml.feature.CountVectorizer;
import org.apache.spark.ml.feature.CountVectorizerModel;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.sql.types.*;

// Input data: Each row is a bag of words from a sentence or document.
List<Row> data = Arrays.asList(
  RowFactory.create(Arrays.asList("a", "b", "c")),
  RowFactory.create(Arrays.asList("a", "b", "b", "c", "a"))
);
StructType schema = new StructType(new StructField [] {
  new StructField("text", new ArrayType(DataTypes.StringType, true), false, Metadata.empty())
});
Dataset<Row> df = spark.createDataFrame(data, schema);

// fit a CountVectorizerModel from the corpus
CountVectorizerModel cvModel = new CountVectorizer()
  .setInputCol("text")
  .setOutputCol("feature")
  .setVocabSize(3)
  .setMinDF(2)
  .fit(df);

// alternatively, define CountVectorizerModel with a-priori vocabulary
CountVectorizerModel cvm = new CountVectorizerModel(new String[]{"a", "b", "c"})
  .setInputCol("text")
  .setOutputCol("feature");

cvModel.transform(df).show(false);
```

## 参考
1. [Machine Learning Library (MLlib) Guide](http://spark.apache.org/docs/latest/ml-features.html)