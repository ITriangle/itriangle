---
layout: post
title: Spark Mlib 提取转化选择特征
categories: [Spark,大数据]
description: Spark Mlib 作为 Spark 机器学习的核心组件
keywords: 大数据,Spark,Java
---




## Feature Selectors

### VectorSlicer
VectorSlicer是一个转换器输入特征向量，输出原始特征向量子集。VectorSlicer接收带有特定索引的向量列，通过对这些索引的值进行筛选得到新的向量集。可接受如下两种索引

1. 整数索引，setIndices()。
2. 字符串索引代表向量中特征的名字，此类要求向量列有AttributeGroup，因为该工具根据Attribute来匹配名字字段。

指定整数或者字符串类型都是可以的。另外，同时使用整数索引和字符串名字也是可以的。不允许使用重复的特征，所以所选的索引或者名字必须是没有独一的。注意如果使用名字特征，当遇到空值的时候将会报错。

输出将会首先按照所选的数字索引排序（按输入顺序），其次按名字排序（按输入顺序）

**Example**
假设我们有一个DataFrame含有userFeatures列：

```
 userFeatures
------------------
 [0.0, 10.0, 0.5]
```

userFeatures是一个向量列包含3个用户特征。假设userFeatures的第一列全为0，我们希望删除它并且只选择后两项。我们可以通过索引setIndices(1,2)来选择后两项并产生一个新的features列：

```
 userFeatures     | features
------------------|-----------------------------
 [0.0, 10.0, 0.5] | [10.0, 0.5]
```

假设我们还有如同["f1","f2", "f3"]的属性，那可以通过名字setNames("f2","f3")的形式来选择：

```
 userFeatures     | features
------------------|-----------------------------
 [0.0, 10.0, 0.5] | [10.0, 0.5]
 ["f1", "f2", "f3"] | ["f2", "f3"]
```

```java
import java.util.List;

import com.google.common.collect.Lists;

import org.apache.spark.ml.attribute.Attribute;
import org.apache.spark.ml.attribute.AttributeGroup;
import org.apache.spark.ml.attribute.NumericAttribute;
import org.apache.spark.ml.feature.VectorSlicer;
import org.apache.spark.ml.linalg.Vectors;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.types.*;

Attribute[] attrs = new Attribute[]{
  NumericAttribute.defaultAttr().withName("f1"),
  NumericAttribute.defaultAttr().withName("f2"),
  NumericAttribute.defaultAttr().withName("f3")
};
AttributeGroup group = new AttributeGroup("userFeatures", attrs);

List<Row> data = Lists.newArrayList(
  RowFactory.create(Vectors.sparse(3, new int[]{0, 1}, new double[]{-2.0, 2.3})),
  RowFactory.create(Vectors.dense(-2.0, 2.3, 0.0))
);

Dataset<Row> dataset =
  spark.createDataFrame(data, (new StructType()).add(group.toStructField()));

VectorSlicer vectorSlicer = new VectorSlicer()
  .setInputCol("userFeatures").setOutputCol("features");

vectorSlicer.setIndices(new int[]{1}).setNames(new String[]{"f3"});
// or slicer.setIndices(new int[]{1, 2}), or slicer.setNames(new String[]{"f2", "f3"})

Dataset<Row> output = vectorSlicer.transform(dataset);
output.show(false);
```

### RFormula
RFormula通过R模型公式来选择列。支持R操作中的部分操作，包括‘~’, ‘.’, ‘:’, ‘+’以及‘-‘，基本操作如下：

1. ~分隔目标和对象
2. +合并对象，“+ 0”意味着删除空格
3. :交互（数值相乘，类别二值化）
4. . 除了目标外的全部列

假设a和b为两列：



### ChiSqSelector
ChiSqSelector代表卡方特征选择。它适用于带有类别特征的标签数据。ChiSqSelector根据类别的独立卡方2检验来对特征排序，然后选取类别标签主要依赖的特征。它类似于选取最有预测能力的特征。

**Example**
假设我们有一个DataFrame含有id,features和clicked三列，其中clicked为需要预测的目标：

```
id | features              | clicked
---|-----------------------|---------
 7 | [0.0, 0.0, 18.0, 1.0] | 1.0
 8 | [0.0, 1.0, 12.0, 0.0] | 0.0
 9 | [1.0, 0.0, 15.0, 0.1] | 0.0
```

如果我们使用ChiSqSelector并设置numTopFeatures为1，根据标签clicked，features中最后一列将会是最有用特征：

```
id | features              | clicked | selectedFeatures
---|-----------------------|---------|------------------
 7 | [0.0, 0.0, 18.0, 1.0] | 1.0     | [1.0]
 8 | [0.0, 1.0, 12.0, 0.0] | 0.0     | [0.0]
 9 | [1.0, 0.0, 15.0, 0.1] | 0.0     | [0.1]
```

```java
import java.util.Arrays;
import java.util.List;

import org.apache.spark.ml.feature.ChiSqSelector;
import org.apache.spark.ml.linalg.VectorUDT;
import org.apache.spark.ml.linalg.Vectors;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.types.DataTypes;
import org.apache.spark.sql.types.Metadata;
import org.apache.spark.sql.types.StructField;
import org.apache.spark.sql.types.StructType;

List<Row> data = Arrays.asList(
  RowFactory.create(7, Vectors.dense(0.0, 0.0, 18.0, 1.0), 1.0),
  RowFactory.create(8, Vectors.dense(0.0, 1.0, 12.0, 0.0), 0.0),
  RowFactory.create(9, Vectors.dense(1.0, 0.0, 15.0, 0.1), 0.0)
);
StructType schema = new StructType(new StructField[]{
  new StructField("id", DataTypes.IntegerType, false, Metadata.empty()),
  new StructField("features", new VectorUDT(), false, Metadata.empty()),
  new StructField("clicked", DataTypes.DoubleType, false, Metadata.empty())
});

Dataset<Row> df = spark.createDataFrame(data, schema);

ChiSqSelector selector = new ChiSqSelector()
  .setNumTopFeatures(1)
  .setFeaturesCol("features")
  .setLabelCol("clicked")
  .setOutputCol("selectedFeatures");

Dataset<Row> result = selector.fit(df).transform(df);

System.out.println("ChiSqSelector output with top " + selector.getNumTopFeatures()
    + " features selected");
result.show();
```

## 参考
1. [Machine Learning Library (MLlib) Guide](http://spark.apache.org/docs/latest/ml-features.html)