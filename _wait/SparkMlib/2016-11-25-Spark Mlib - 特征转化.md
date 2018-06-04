---
layout: post
title: Spark Mlib Feature Extractors
categories: [Spark,大数据]
description: Spark Mlib 作为 Spark 机器学习的核心组件
keywords: 大数据,Spark,Java
---

# Feature Extractors

## Feature Transformers

### Tokenizer
Tokenization将文本划分为独立个体（通常为单词）。下面的例子展示了如何把句子划分为单词。

RegexTokenizer基于正则表达式提供更多的划分选项。默认情况下，参数“pattern”为划分文本的分隔符。或者，用户可以指定参数“gaps”来指明正则“patten”表示“tokens”而不是分隔符，这样来为分词结果找到所有可能匹配的情况。

```java
import java.util.Arrays;
import java.util.List;

import scala.collection.mutable.WrappedArray;

import org.apache.spark.ml.feature.RegexTokenizer;
import org.apache.spark.ml.feature.Tokenizer;
import org.apache.spark.sql.api.java.UDF1;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.types.DataTypes;
import org.apache.spark.sql.types.Metadata;
import org.apache.spark.sql.types.StructField;
import org.apache.spark.sql.types.StructType;

List<Row> data = Arrays.asList(
  RowFactory.create(0, "Hi I heard about Spark"),
  RowFactory.create(1, "I wish Java could use case classes"),
  RowFactory.create(2, "Logistic,regression,models,are,neat")
);

StructType schema = new StructType(new StructField[]{
  new StructField("id", DataTypes.IntegerType, false, Metadata.empty()),
  new StructField("sentence", DataTypes.StringType, false, Metadata.empty())
});

Dataset<Row> sentenceDataFrame = spark.createDataFrame(data, schema);

Tokenizer tokenizer = new Tokenizer().setInputCol("sentence").setOutputCol("words");

RegexTokenizer regexTokenizer = new RegexTokenizer()
    .setInputCol("sentence")
    .setOutputCol("words")
    .setPattern("\\W");  // alternatively .setPattern("\\w+").setGaps(false);

spark.udf().register("countTokens", new UDF1<WrappedArray, Integer>() {
  @Override
  public Integer call(WrappedArray words) {
    return words.size();
  }
}, DataTypes.IntegerType);

Dataset<Row> tokenized = tokenizer.transform(sentenceDataFrame);
tokenized.select("sentence", "words")
    .withColumn("tokens", callUDF("countTokens", col("words"))).show(false);

Dataset<Row> regexTokenized = regexTokenizer.transform(sentenceDataFrame);
regexTokenized.select("sentence", "words")
    .withColumn("tokens", callUDF("countTokens", col("words"))).show(false);
```

### StopWordsRemover
停用词为在文档中频繁出现，但未承载太多意义的词语，他们不应该被包含在算法输入中。 

StopWordsRemover的输入为一系列字符串（如分词器输出），输出中删除了所有停用词。停用词表由stopWords参数提供。一些语言的默认停用词表可以通过StopWordsRemover.loadDefaultStopWords(language)调用。布尔参数caseSensitive指明是否区分大小写（默认为否）。

**Examples**
假设我们有如下DataFrame，有id和raw两列：

```
 id | raw
----|----------
 0  | [I, saw, the, red, baloon]
 1  | [Mary, had, a, little, lamb]
```

通过对raw列调用StopWordsRemover，我们可以得到筛选出的结果列如下：

```
 id | raw                         | filtered
----|-----------------------------|--------------------
 0  | [I, saw, the, red, baloon]  |[saw, red, baloon]
 1  | [Mary, had, a, little, lamb]|[Mary, little, lamb]
```

其中，“I”, “the”, “had”以及“a”被移除。

```java
import java.util.Arrays;
import java.util.List;

import org.apache.spark.ml.feature.StopWordsRemover;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.types.DataTypes;
import org.apache.spark.sql.types.Metadata;
import org.apache.spark.sql.types.StructField;
import org.apache.spark.sql.types.StructType;

StopWordsRemover remover = new StopWordsRemover()
  .setInputCol("raw")
  .setOutputCol("filtered");

List<Row> data = Arrays.asList(
  RowFactory.create(Arrays.asList("I", "saw", "the", "red", "balloon")),
  RowFactory.create(Arrays.asList("Mary", "had", "a", "little", "lamb"))
);

StructType schema = new StructType(new StructField[]{
  new StructField(
    "raw", DataTypes.createArrayType(DataTypes.StringType), false, Metadata.empty())
});

Dataset<Row> dataset = spark.createDataFrame(data, schema);
remover.transform(dataset).show(false);
```

### nn-gram

### Binarizer

### PCA

### PolynomialExpansion

### Discrete Cosine Transform (DCT)

### StringIndexer

### IndexToString

### OneHotEncoder

### VectorIndexer

### Interaction

### Normalizer

### StandardScaler

### MinMaxScaler

### MaxAbsScaler

### Bucketizer

### ElementwiseProduct

### SQLTransformer

### VectorAssembler

### QuantileDiscretizer 

## 参考
1. [Machine Learning Library (MLlib) Guide](http://spark.apache.org/docs/latest/ml-features.html)