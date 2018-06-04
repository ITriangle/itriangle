---
layout: post
title: Spark Mlib ML Pipelines
categories: [Spark,大数据]
description: Spark Mlib 作为 Spark 机器学习的核心组件
keywords: 大数据,Spark,Java
---

# ML Pipelines
在本节中,我们介绍了ML管道的概念. ML管道提供了一组统一的高级API,它们构建在DataFrames之上,可帮助用户创建和调整实际的机器学习流水线

Table of Contents

- 管道中的主要概念
    - DataFrame
    - Pipeline 组成
        - Transformers
        - 估计
        - 管道组件的性能
    - Pipeline
        - 管道是如何工作的
        - 细节
    - 参数
    - 保存和加载管道
- 代码示例
    - 示例:估计,转化,参数
    - 示例:管道
    - 模型选择:高级参数调用 

## 一. 管道中的主要概念(Main concepts in Pipelines)
MLlib为机器学习算法标准化API,以便更容易将多个算法合并到单个管道或工作流中. 本部分涵盖Pipelines API引入的关键概念,其中管道概念主要受 [scikit-learn](http://scikit-learn.org/)项目的启发.


- **DataFrame**:ML API使用Spark SQL中的DataFrame作为ML数据集,它可以容纳各种数据类型.例如,DataFrame 有不同的列,可以存储文本,特征向量,真实标签和预测.
- **Transformer**:Transformer是一种可以将一个DataFrame转换为另一个DataFrame的算法.例如,ML模型是将具有特征的DataFrame转换为具有预测的DataFrame.
- **Estimator**:Estimator 是一个算法,可以适应DataFrame来生成Transformer的算法.例如,学习算法 Estimator 是通过训练 DataFrame 产生的算法模型.
- **Pipeline**:Pipeline 将多个 Transformer 和 多个 Estimator 链接在一起,使 ML 按照指定的流程工作.
- **Parameter**:所有的 Transformer , Estimator 现在共享用于指定参数的公共API.

### 1. 数据帧(DataFrame)
ML 可以应用于各种各样的数据类型,例如向量,文本,图像和结构化数据. 该API采用Spark SQL的DataFrame,以支持各种数据类型. 

DataFrame支持许多基本和结构化类型; 请参阅Spark SQL数据类型引用以获取支持的类型列表. 除了Spark [SQL指南中列出的类型](http://spark.apache.org/docs/latest/sql-programming-guide.html#data-types)之外,DataFrame还可以使用[ML Vector](http://spark.apache.org/docs/latest/mllib-data-types.html#local-vector)类型. DataFrame可以从常规RDD隐式或显式创建. 有关示例,请参阅下面的代码示例和[Spark SQL编程指南](http://spark.apache.org/docs/latest/sql-programming-guide.html).

DataFrame中的列被命名. 下面的代码示例使用名称,如"文本" ,"特征" 和"标签" .

### 2. Pipeline 组成(Pipeline components)

#### 2.1 转化(Transformers)
Transformer 是一种抽象,包括 feature transformer 和 learning mode.在技​​术上,Transformer 实现一种方法 transform(),它通常通过追加一列或多列来将一个DataFrame转换成另一个.例如：

- feature transformer 可以使用 DataFrame,读取列(例如文本),将其映射到新列(例如,特征向量)中,并输出附加了映射列的新DataFrame.
- learning model 可以使用 DataFrame,读取包含特征向量的列,预测每个特征向量的标签,并输出一个新的DataFrame,而且这个 DataFrame 附加预测标签作为列.


#### 2.2 估计(Estimators)
Estimator 抽象学习算法的概念,抽象适合或训练数据的任何算法.技术上,一个 Estimator 实现一个方法 fit(),它接受一个 DataFrame 并产生一个 Model ,它是一个 Transformer.例如,诸如Logistic回归之类的学习算法是一个 Estimator,调用fit()训练一个 Logistic回归模型,它是一个Model,因此是一个Transformer.

#### 2.3 管道组件的性能(Properties of pipeline components)
Transformer.transform()和 Estimator.fit()都是无状态的.在未来,可以通过替代概念来支持有状态算法.

Transformer 或 Estimator 的每个实例都有一个唯一的ID,可用于指定参数(如下所述).

### 3. Pipeline
在机器学习中,通常运行一系列算法来处理和学习数据.比如:简单的文本处理工作流可能包括几个阶段：

1. 将每个文档的文本分割成单词. 
2. 将每个文档的单词转换为数字特征向量. 
3. 使用特征向量和标签学习预测模型. 
 
MLlib的工作流程表现为一个 Pipeline ,该 Pipeline 由一组 PipelineStages (Transformers and Estimators)运行在特定的顺序组成.我们将使用这个简单的工作流作为本节的运行示例.

#### 3.1 管道是如何工作的
管道是指定阶段的序列,每个阶段不是 Transformer 就是 Estimator.这些阶段是按照顺序执行的,输入的 DataFrame 在通过每个夹断是被转化.对于 Transformer 每个阶段, 是在 DataFrame 上调用 transform() 方法. 对于 Estimator 夹断,调用 fit() 方法来生成一个 Tranformer(它成为 PipelineModel的一部分,或者适合的 Pipeline),并且在 DataFrame 上调用 Transformer 的 transform() 方法.


我们为简单的文本文档工作流程进行了说明.下图显示了管道的训练时间.
<center>![](http://spark.apache.org/docs/latest/img/ml-Pipeline.png)</center>

以上,上面行代表一个三级管道.前两个(Tokennizer 和 HashingTF) 是 Tranformer (蓝色), 第三个(LogisticsRegression)是 Estimator(红色).下面行表示流经管道的数据,其中柱面指示DataFrames.在原始DataFrame上调用Pipeline.fit()方法,它具有原始文本文档和标签. Tokenizer.transform()方法将原始文本文档分割成单词,向DataFrame添加一个带有单词的新列. HashingTF.transform()方法将单词列转换为特征向量,向DataFrame添加带有这些向量的新列.现在,由于 LogisticsRegression 是 Estimator, 管道首先调用 LogisticRegression.fit() 来生成一个 LogisticsRegressionModel.如果管道有更多的阶段,那么在将DataFrame传递到下一个阶段之前,它将会在 DataFrame 上调用 LogisticRegressionModel 的 transform() 方法.

A Pipeline is an Estimator. Thus, after a Pipeline’s fit() method runs, it produces a PipelineModel, which is a Transformer. This PipelineModel is used at test time; the figure below illustrates this usage.
中文(简体)
管道是一个 Estimator.因此,在Pipeline的 fit()方法运行之后,它会生成一个PipelineModel,它是一个Transformer.此管道模型在测试时使用;下图说明了这种用法.
<center>![](http://spark.apache.org/docs/latest/img/ml-PipelineModel.png)</center>

在上图中,PipelineModel 具有与原始Pipeline相同的级数,但原始Pipeline中的所有Estimators 都已成为 Transformers .当在测试数据集上调用PipelineModel的 transform()方法时,数据按顺序传递通过合适的管道.每个阶段的transform()方法更新数据集并将其传递到下一个阶段.

Pipelines 和 PipelineModels 有助于确保训练数据和测试数据通过相同的功能处理步骤.

#### 3.2 细节

DAG Pipelines: 管道的状态是有序的队列.这儿给的例子都是线性的管道,也就是说管道的每个阶段使用上一个阶段产生的数据.我们也可以产生非线性的管道,数据流向为无向非环图(DAG).这种图通常需要明确地指定每个阶段的输入和输出列名(通常以指定参数的形式).如果管道是DAG形式,则每个阶段必须以拓扑序的形式指定.

Runtime checking: 因为管道可以运行在多种数据类型上,所以不能使用编译时间检查.管道和管道模型在实际运行管道之前就会进行运行时间检查.这种检查通过 DataFrame schema,它描述了数据框中各列的类型.

Unique Pipeline stages: 管道的的每个阶段需要是唯一的实体.如同样的实体 "myHashingTF" 不可以进入管道两次,因为管道的每个阶段必须有唯一的ID.当然 "myHashingTF1"  和 "myHashingTF2" (都是 HashingTF)可以进入同个管道两次,因为他们有不同的ID.

### 4. 参数
MLlib Estimators 和 Transformers 使用统一的API来指定参数.

Param 是具有独立文档命名的参数. ParamMap 是一组(参数,值)对.

将参数传递给算法有两种主要方法：

1. 给实体设置参数.比如,lr是一个逻辑回归实体,通过lr.setMaxIter(10)来使得lr在拟合的时候最多迭代10次.这个接口与spark.mllib包相似.
2. 传递ParamMap到fit()或者transform().所有在ParamMap里的参数都将通过设置被重写.

参数属于指定 Estimators 和 Transformers 实体过程.因此,如果我们有两个逻辑回归实体lr1和lr2,我们可以建立一个ParamMap来指定两个实体的最大迭代次数参数：ParamMap(lr1.maxIter -> 10, lr2.maxIter -> 20).这在一个管道里有两个算法都有最大迭代次数参数时非常有用.

### 5. 保存和加载管道
我们经常需要将管道存储到磁盘以供下次使用.在Spark1.6中,模型导入导出功能新添了管道接口,支持大多数 Transformers .请到算法接口文档查看是否支持存储和读入.

## 二. 代码示例
下面给出上述讨论功能的代码示例：

### 1. 示例:估计,转化,参数
此示例涵盖了Estimators,Transformers 和参数的概念

```java
import java.util.Arrays;
import java.util.List;

import org.apache.spark.ml.classification.LogisticRegression;
import org.apache.spark.ml.classification.LogisticRegressionModel;
import org.apache.spark.ml.linalg.VectorUDT;
import org.apache.spark.ml.linalg.Vectors;
import org.apache.spark.ml.param.ParamMap;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.RowFactory;
import org.apache.spark.sql.types.DataTypes;
import org.apache.spark.sql.types.Metadata;
import org.apache.spark.sql.types.StructField;
import org.apache.spark.sql.types.StructType;

// Prepare training data.
List<Row> dataTraining = Arrays.asList(
    RowFactory.create(1.0, Vectors.dense(0.0, 1.1, 0.1)),
    RowFactory.create(0.0, Vectors.dense(2.0, 1.0, -1.0)),
    RowFactory.create(0.0, Vectors.dense(2.0, 1.3, 1.0)),
    RowFactory.create(1.0, Vectors.dense(0.0, 1.2, -0.5))
);
StructType schema = new StructType(new StructField[]{
    new StructField("label", DataTypes.DoubleType, false, Metadata.empty()),
    new StructField("features", new VectorUDT(), false, Metadata.empty())
});
Dataset<Row> training = spark.createDataFrame(dataTraining, schema);

// Create a LogisticRegression instance. This instance is an Estimator.
LogisticRegression lr = new LogisticRegression();
// Print out the parameters, documentation, and any default values.
System.out.println("LogisticRegression parameters:\n" + lr.explainParams() + "\n");

// We may set parameters using setter methods.
lr.setMaxIter(10).setRegParam(0.01);

// Learn a LogisticRegression model. This uses the parameters stored in lr.
LogisticRegressionModel model1 = lr.fit(training);
// Since model1 is a Model (i.e., a Transformer produced by an Estimator),
// we can view the parameters it used during fit().
// This prints the parameter (name: value) pairs, where names are unique IDs for this
// LogisticRegression instance.
System.out.println("Model 1 was fit using parameters: " + model1.parent().extractParamMap());

// We may alternatively specify parameters using a ParamMap.
ParamMap paramMap = new ParamMap()
  .put(lr.maxIter().w(20))  // Specify 1 Param.
  .put(lr.maxIter(), 30)  // This overwrites the original maxIter.
  .put(lr.regParam().w(0.1), lr.threshold().w(0.55));  // Specify multiple Params.

// One can also combine ParamMaps.
ParamMap paramMap2 = new ParamMap()
  .put(lr.probabilityCol().w("myProbability"));  // Change output column name
ParamMap paramMapCombined = paramMap.$plus$plus(paramMap2);

// Now learn a new model using the paramMapCombined parameters.
// paramMapCombined overrides all parameters set earlier via lr.set* methods.
LogisticRegressionModel model2 = lr.fit(training, paramMapCombined);
System.out.println("Model 2 was fit using parameters: " + model2.parent().extractParamMap());

// Prepare test documents.
List<Row> dataTest = Arrays.asList(
    RowFactory.create(1.0, Vectors.dense(-1.0, 1.5, 1.3)),
    RowFactory.create(0.0, Vectors.dense(3.0, 2.0, -0.1)),
    RowFactory.create(1.0, Vectors.dense(0.0, 2.2, -1.5))
);
Dataset<Row> test = spark.createDataFrame(dataTest, schema);

// Make predictions on test documents using the Transformer.transform() method.
// LogisticRegression.transform will only use the 'features' column.
// Note that model2.transform() outputs a 'myProbability' column instead of the usual
// 'probability' column since we renamed the lr.probabilityCol parameter previously.
Dataset<Row> results = model2.transform(test);
Dataset<Row> rows = results.select("features", "label", "myProbability", "prediction");
for (Row r: rows.collectAsList()) {
  System.out.println("(" + r.get(0) + ", " + r.get(1) + ") -> prob=" + r.get(2)
    + ", prediction=" + r.get(3));
}
```

### 2. 示例:管道
该示例遵循上述附图中所示的简单文本文档管道

```java
import java.util.Arrays;

import org.apache.spark.ml.Pipeline;
import org.apache.spark.ml.PipelineModel;
import org.apache.spark.ml.PipelineStage;
import org.apache.spark.ml.classification.LogisticRegression;
import org.apache.spark.ml.feature.HashingTF;
import org.apache.spark.ml.feature.Tokenizer;
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;

// Prepare training documents, which are labeled.
Dataset<Row> training = spark.createDataFrame(Arrays.asList(
  new JavaLabeledDocument(0L, "a b c d e spark", 1.0),
  new JavaLabeledDocument(1L, "b d", 0.0),
  new JavaLabeledDocument(2L, "spark f g h", 1.0),
  new JavaLabeledDocument(3L, "hadoop mapreduce", 0.0)
), JavaLabeledDocument.class);

// Configure an ML pipeline, which consists of three stages: tokenizer, hashingTF, and lr.
Tokenizer tokenizer = new Tokenizer()
  .setInputCol("text")
  .setOutputCol("words");
HashingTF hashingTF = new HashingTF()
  .setNumFeatures(1000)
  .setInputCol(tokenizer.getOutputCol())
  .setOutputCol("features");
LogisticRegression lr = new LogisticRegression()
  .setMaxIter(10)
  .setRegParam(0.001);
Pipeline pipeline = new Pipeline()
  .setStages(new PipelineStage[] {tokenizer, hashingTF, lr});

// Fit the pipeline to training documents.
PipelineModel model = pipeline.fit(training);

// Prepare test documents, which are unlabeled.
Dataset<Row> test = spark.createDataFrame(Arrays.asList(
  new JavaDocument(4L, "spark i j k"),
  new JavaDocument(5L, "l m n"),
  new JavaDocument(6L, "spark hadoop spark"),
  new JavaDocument(7L, "apache hadoop")
), JavaDocument.class);

// Make predictions on test documents.
Dataset<Row> predictions = model.transform(test);
for (Row r : predictions.select("id", "text", "probability", "prediction").collectAsList()) {
  System.out.println("(" + r.get(0) + ", " + r.get(1) + ") --> prob=" + r.get(2)
    + ", prediction=" + r.get(3));
}
```


### 3. 模型选择:高级参数调用 
使用ML管道的一大优点是超参数优化.有关自动模式选择的更多信息,请参阅"[ML调整指南](http://spark.apache.org/docs/latest/ml-tuning.html)" .

## 参考
1. [Machine Learning Library (MLlib) Guide](http://spark.apache.org/docs/latest/ml-guide.html)