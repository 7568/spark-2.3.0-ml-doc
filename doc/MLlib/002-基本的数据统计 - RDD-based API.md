# 基本的数据统计 - RDD-based API

- [Summary statistics](#summary-statistics)
- [Correlations](#correlations)
- [Stratified sampling](#stratified-sampling)
- Hypothesis testing
  - [Streaming Significance Testing](#streaming-significance-testing)
- [Random data generation](#random-data-generation)
- [Kernel density estimation](#kernel-density-estimation)

## Summary statistics

We provide column summary statistics for `RDD[Vector]` through the function `colStats` available in `Statistics`.

我们提供通过访问方法`Statistics`中的方法 `colStats` 来对 `RDD[Vector]`进行列汇总统计 。

[`colStats()`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/stat/Statistics.html) returns an instance of [`MultivariateStatisticalSummary`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/stat/MultivariateStatisticalSummary.html), which contains the column-wise max, min, mean, variance, and number of nonzeros, as well as the total count.

Refer to the [`MultivariateStatisticalSummary` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/stat/MultivariateStatisticalSummary.html) for details on the API.

[`colStats()`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/stat/Statistics.html) 返回一个 [`MultivariateStatisticalSummary`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/stat/MultivariateStatisticalSummary.html)的实例对象，它包含每一列的最大值，最小值，平均值，方差, 和非零数据的数量, 以及总的数量。

详情请参考 [`MultivariateStatisticalSummary` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/stat/MultivariateStatisticalSummary.html) 

```java
import java.util.Arrays;

import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.mllib.linalg.Vector;
import org.apache.spark.mllib.linalg.Vectors;
import org.apache.spark.mllib.stat.MultivariateStatisticalSummary;
import org.apache.spark.mllib.stat.Statistics;

JavaRDD<Vector> mat = jsc.parallelize(
  Arrays.asList(
    Vectors.dense(1.0, 10.0, 100.0),
    Vectors.dense(2.0, 20.0, 200.0),
    Vectors.dense(3.0, 30.0, 300.0)
  )
); // an RDD of Vectors

// Compute column summary statistics.
MultivariateStatisticalSummary summary = Statistics.colStats(mat.rdd());
System.out.println(summary.mean());  // a dense vector containing the mean value for each column
System.out.println(summary.variance());  // column-wise variance
System.out.println(summary.numNonzeros());  // number of nonzeros in each column
```

Find full example code at "examples/src/main/scala/org/apache/spark/examples/mllib/SummaryStatisticsExample.scala" in the Spark repo.

详细使用例子请看[SummaryStatisticsExample.scala](https://github.com/apache/spark/tree/master/examples/src/main/scala/org/apache/spark/examples/mllib/SummaryStatisticsExample.scala)

## Correlations

Calculating the correlation between two series of data is a common operation in Statistics. In `spark.mllib` we provide the flexibility to calculate pairwise correlations among many series. The supported correlation methods are currently Pearson’s and Spearman’s correlation.

在统计学中统计两组数据之间的相关性是一个很常见的操作。在  `spark.mllib` 中我们提供了灵活的计算很多数据当中的两两之间相关性。支持相关性的方法目前有 Pearson’s （皮尔森）和 Spearman’s （斯皮尔曼）两个相关性。

 Pearson’s  ： 两个变量$(X, Y)$的皮尔森相关性系数$(ρX,Y)$等于它们之间的协方差$cov(X,Y)$除以它们各自标准差的乘积$(σX, σY)$

##                                               $(ρX,Y)$  = $\frac{cov(X,Y)}{\sigma X\sigma Y}$

 Spearman’s ：假设两个随机变量分别为X、Y（也可以看做两个集合），它们的元素个数均为N，两个随即变量取的第i（1<=i<=N）个值分别用Xi、Yi表示。对X、Y进行排序（同时为升序或降序），得到两个元素排行集合x、y，其中元素xi、yi分别为Xi在X中的排行以及Yi在Y中的排行。将集合x、y中的元素对应相减得到一个排行差分集合d，其中di=xi-yi，1<=i<=N。随机变量X、Y之间的斯皮尔曼等级相关系数可以由x、y或者d计算得到，其计算方式如下所示：

## 					$\rho$ = $1- \frac{6\sum d_i^2}{n(n^2-1)}$

也可以表示成

## 				 $\rho$ =$\frac{\sum_i(x_i-\overline{x})(y_i-\overline{y})}{\sqrt{\sum_i(x_i-\overline{x})^2\sum_i(y_i-\overline{y})^2}}$

[`Statistics`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/stat/Statistics.html) provides methods to calculate correlations between series. Depending on the type of input, two `JavaDoubleRDD`s or a `JavaRDD<Vector>`, the output will be a `Double` or the correlation `Matrix` respectively.

Refer to the [`Statistics` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/stat/Statistics.html) for details on the API.

[`Statistics`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/stat/Statistics.html) 提供计算两组数相关性的方法. 根据输入类型是两个`JavaDoubleRDD` 还是`JavaRDD<Vector>`，输出将分别是一个浮点型数据或者是一个相关矩阵。

更多信息请查看[`Statistics` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/stat/Statistics.html) 。

```java
import java.util.Arrays;

import org.apache.spark.api.java.JavaDoubleRDD;
import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.mllib.linalg.Matrix;
import org.apache.spark.mllib.linalg.Vector;
import org.apache.spark.mllib.linalg.Vectors;
import org.apache.spark.mllib.stat.Statistics;

JavaDoubleRDD seriesX = jsc.parallelizeDoubles(
  Arrays.asList(1.0, 2.0, 3.0, 3.0, 5.0));  // a series

// must have the same number of partitions and cardinality as seriesX
JavaDoubleRDD seriesY = jsc.parallelizeDoubles(
  Arrays.asList(11.0, 22.0, 33.0, 33.0, 555.0));

// compute the correlation using Pearson's method. Enter "spearman" for Spearman's method.
// If a method is not specified, Pearson's method will be used by default.
Double correlation = Statistics.corr(seriesX.srdd(), seriesY.srdd(), "pearson");
System.out.println("Correlation is: " + correlation);

// note that each Vector is a row and not a column
JavaRDD<Vector> data = jsc.parallelize(
  Arrays.asList(
    Vectors.dense(1.0, 10.0, 100.0),
    Vectors.dense(2.0, 20.0, 200.0),
    Vectors.dense(5.0, 33.0, 366.0)
  )
);

// calculate the correlation matrix using Pearson's method.
// Use "spearman" for Spearman's method.
// If a method is not specified, Pearson's method will be used by default.
Matrix correlMatrix = Statistics.corr(data.rdd(), "pearson");
System.out.println(correlMatrix.toString());
```

运行结果是

#### 			$M_x^y$= $\begin{bmatrix}1.0         &        0.9788834658894731 & 0.9903895695275673 \\  0.9788834658894731  &1.0        &         0.9977483233986101  \\0.9903895695275673 & 0.9977483233986101  &1.0   \end{bmatrix}$

完整的案例代码请看"examples/src/main/java/org/apache/spark/examples/mllib/JavaCorrelationsExample.java" 

## Stratified sampling

Unlike the other statistics functions, which reside in `spark.mllib`, stratified sampling methods, `sampleByKey` and `sampleByKeyExact`, can be performed on RDD’s of key-value pairs. For stratified sampling, the keys can be thought of as a label and the value as a specific attribute. For example the key can be man or woman, or document ids, and the respective values can be the list of ages of the people in the population or the list of words in the documents. The `sampleByKey` method will flip a coin to decide whether an observation will be sampled or not, therefore requires one pass over the data, and provides an *expected* sample size. `sampleByKeyExact` requires significant more resources than the per-stratum simple random sampling used in `sampleByKey`, but will provide the exact sampling size with 99.99% confidence. `sampleByKeyExact` is currently not supported in python.

不像其他的统计方法, 它存在于`spark.mllib`中，两个分层抽样方法, `sampleByKey` and `sampleByKeyExact`, 他们能够用于RDD的key-value键值对上，对于分层抽样，它的key能够被想想成一个lable和value组成的特殊属性。例如key是男人或者女人，或者id，那么代表value的值可以是人口当中的人的一组年龄段或者是一个文档中的一组单词。 `sampleByKey` 方法 将会通过翻银币的方式来决定是否一个观察可以被当成抽样数据，因此它需要被传递一个参数和提供一个期望数据的大小。`sampleByKeyExact`比`sampleByKey`中使用的分层的随机样例数据需要更多的资源(就是说耗性能)，但是能提供99.99%自信度的样例数据大小。 `sampleByKeyExact` 目前不支持 python.



[`sampleByKeyExact()`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/api/java/JavaPairRDD.html) allows users to sample exactly ⌈$f_k⋅n_k$⌉∀k∈K items, where $f_k$ is the desired fraction for key $k$,$n_k$ is the number of key-value pairs for key $k$, and $K$ is the set of keys. Sampling without replacement requires one additional pass over the RDD to guarantee sample size, whereas sampling with replacement requires two additional passes.

[`sampleByKeyExact()`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/api/java/JavaPairRDD.html) 允许使用一个精确的满足 ⌈$f_k⋅n_k$⌉∀k∈K 表达式的一列数据，其中的 $f_k$ 是一个k期望的值，$n_k$ 是key-value键值对中的k，K是所有的key。样例数据不存在替换值所以它需要一个附加的RDD参数来保证样例数据的大小，而这样样例数据需要两个额外的通行证。

```java
import java.util.*;

import scala.Tuple2;

import org.apache.spark.api.java.JavaPairRDD;

 SparkConf conf = new SparkConf().setAppName("JavaStratifiedSamplingExample").setMaster("local");
    JavaSparkContext jsc = new JavaSparkContext(conf);

List<Tuple2<Integer, Character>> list = Arrays.asList(
    new Tuple2<>(1, 'a'),
    new Tuple2<>(1, 'b'),
    new Tuple2<>(2, 'c'),
    new Tuple2<>(2, 'd'),
    new Tuple2<>(2, 'e'),
    new Tuple2<>(3, 'f')
);

JavaPairRDD<Integer, Character> data = jsc.parallelizePairs(list);

// specify the exact fraction desired from each key Map<K, Double>
ImmutableMap<Integer, Double> fractions = ImmutableMap.of(1, 0.1, 2, 0.6, 3, 0.3);

// Get an approximate sample from each stratum
JavaPairRDD<Integer, Character> approxSample = data.sampleByKey(false, fractions);
// Get an exact sample from each stratum
JavaPairRDD<Integer, Character> exactSample = data.sampleByKeyExact(false, fractions);
```

详细信息请参考例子中的"examples/src/main/java/org/apache/spark/examples/mllib/JavaStratifiedSamplingExample.java"

## Hypothesis testing

假设检验

Hypothesis testing is a powerful tool in statistics to determine whether a result is statistically significant, whether this result occurred by chance or not. `spark.mllib` currently supports Pearson’s chi-squared ($\chi^2$) tests for goodness of fit and independence. The input data types determine whether the goodness of fit or the independence test is conducted. The goodness of fit test requires an input type of `Vector`, whereas the independence test requires a `Matrix` as input.

`spark.mllib` also supports the input type `RDD[LabeledPoint]` to enable feature selection via chi-squared independence tests.

Hypothesis testing 在统计学中是一个很强的工具，用它来决定一个结果是否具有统计意义，和检验这个结果是否被修改过, `spark.mllib` 目前支持 Pearson’s chi-squared ($\chi^2$) tests 以确定数据的适应性和独立性。输入数据的类型决定了是适应性测试还是独立性测试。适应性测试需要的输入类型是`Vector`, 而独立性测试需要的输入类型是 `Matrix`。

`spark.mllib` 同样也支持 `RDD[LabeledPoint]` 格式的输入，它通过选择 chi-squared 独立性测试的方式来实现。

[`Statistics`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/stat/Statistics.html) provides methods to run Pearson’s chi-squared tests. The following example demonstrates how to run and interpret hypothesis tests.

Refer to the [`ChiSqTestResult` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/stat/test/ChiSqTestResult.html) for details on the API.

[`Statistics`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/stat/Statistics.html) 提供方法来执行 Pearson’s chi-squared 测试。下面的例子演示了运行和interpret hypothesis tests.

更多信息请参考 [`ChiSqTestResult` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/stat/test/ChiSqTestResult.html) 。

```java
import java.util.Arrays;

import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.mllib.linalg.Matrices;
import org.apache.spark.mllib.linalg.Matrix;
import org.apache.spark.mllib.linalg.Vector;
import org.apache.spark.mllib.linalg.Vectors;
import org.apache.spark.mllib.regression.LabeledPoint;
import org.apache.spark.mllib.stat.Statistics;
import org.apache.spark.mllib.stat.test.ChiSqTestResult;

// a vector composed of the frequencies of events
Vector vec = Vectors.dense(0.1, 0.15, 0.2, 0.3, 0.25);

// compute the goodness of fit. If a second vector to test against is not supplied
// as a parameter, the test runs against a uniform distribution.
ChiSqTestResult goodnessOfFitTestResult = Statistics.chiSqTest(vec);
// summary of the test including the p-value, degrees of freedom, test statistic,
// the method used, and the null hypothesis.
System.out.println(goodnessOfFitTestResult + "\n");

// Create a contingency matrix ((1.0, 2.0), (3.0, 4.0), (5.0, 6.0))
Matrix mat = Matrices.dense(3, 2, new double[]{1.0, 3.0, 5.0, 2.0, 4.0, 6.0});

// conduct Pearson's independence test on the input contingency matrix
ChiSqTestResult independenceTestResult = Statistics.chiSqTest(mat);
// summary of the test including the p-value, degrees of freedom...
System.out.println(independenceTestResult + "\n");

// an RDD of labeled points
JavaRDD<LabeledPoint> obs = jsc.parallelize(
  Arrays.asList(
    new LabeledPoint(1.0, Vectors.dense(1.0, 0.0, 3.0)),
    new LabeledPoint(1.0, Vectors.dense(1.0, 2.0, 0.0)),
    new LabeledPoint(-1.0, Vectors.dense(-1.0, 0.0, -0.5))
  )
);

// The contingency table is constructed from the raw (feature, label) pairs and used to conduct
// the independence test. Returns an array containing the ChiSquaredTestResult for every feature
// against the label.
ChiSqTestResult[] featureTestResults = Statistics.chiSqTest(obs.rdd());
int i = 1;
for (ChiSqTestResult result : featureTestResults) {
  System.out.println("Column " + i + ":");
  System.out.println(result + "\n");  // summary of the test
  i++;
}
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/mllib/JavaHypothesisTestingExample.java" in the Spark repo.

Additionally, `spark.mllib` provides a 1-sample, 2-sided implementation of the Kolmogorov-Smirnov (KS) test for equality of probability distributions. By providing the name of a theoretical distribution (currently solely supported for the normal distribution) and its parameters, or a function to calculate the cumulative distribution according to a given theoretical distribution, the user can test the null hypothesis that their sample is drawn from that distribution. In the case that the user tests against the normal distribution (`distName="norm"`), but does not provide distribution parameters, the test initializes to the standard normal distribution and logs an appropriate message.

另外, 为了概率分布的平等性，`spark.mllib` 提供了1-sample, 2-sided 两种对 Kolmogorov-Smirnov (KS) test的实现。 通过提供一个理论分布的名称(目前只支持常规的分布)和它的参数，或者是通过给定的理论分布提供一个计算积累分布的方法，用户可以测试一个从分布中抽取出来的假设null的数据。如果用户想测试一个正态分布 (`distName="norm"`)，但是没有提供该分布的参数，那么系统在测试的时候就会初始化一个标准的正态分布然后记录适当的日志消息。

[`Statistics`](https://spark.apache.org/docs/2.1.1/api/java/org/apache/spark/mllib/stat/Statistics.html) provides methods to run a 1-sample, 2-sided Kolmogorov-Smirnov test. The following example demonstrates how to run and interpret the hypothesis tests.

Refer to the [`Statistics` Java docs](https://spark.apache.org/docs/2.1.1/api/java/org/apache/spark/mllib/stat/Statistics.html) for details on the API.

[`Statistics`](https://spark.apache.org/docs/2.1.1/api/java/org/apache/spark/mllib/stat/Statistics.html) 提供来进行 1-sample, 2-sided Kolmogorov-Smirnov 测试的方法，下面的例子中以代码的形式告诉你如何进行 hypothesis 测试。

更多详情请参考 [`Statistics` Java docs](https://spark.apache.org/docs/2.1.1/api/java/org/apache/spark/mllib/stat/Statistics.html) 。

```java
import java.util.Arrays;

import org.apache.spark.api.java.JavaDoubleRDD;
import org.apache.spark.mllib.stat.Statistics;
import org.apache.spark.mllib.stat.test.KolmogorovSmirnovTestResult;

JavaDoubleRDD data = jsc.parallelizeDoubles(Arrays.asList(0.1, 0.15, 0.2, 0.3, 0.25));
KolmogorovSmirnovTestResult testResult =
  Statistics.kolmogorovSmirnovTest(data, "norm", 0.0, 1.0);
// summary of the test including the p-value, test statistic, and null hypothesis
// if our p-value indicates significance, we can reject the null hypothesis
System.out.println(testResult);
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/mllib/JavaHypothesisTestingKolmogorovSmirnovTestExample.java" in the Spark repo.

### Streaming Significance Testing

performed on a Spark Streaming `DStream[(Boolean,Double)]` where the first element of each tuple indicates control group (`false`) or treatment group (`true`) and the second element is the value of an observation.

Streaming significance testing supports the following parameters:

- `peacePeriod` - The number of initial data points from the stream to ignore, used to mitigate novelty effects.
- `windowSize` - The number of past batches to perform hypothesis testing over. Setting to `0` will perform cumulative processing using all prior batches.

在 Spark Streaming `DStream[(Boolean,Double)]` 中第一个元素中false表示控制组，true表示治疗组，第二个元素是一个观察数据。

Streaming significance testing 支持下面的参数

- `peacePeriod` - 流数据忽略的初始数据点的数量，用来缓解奇异效应
- `windowSize` - 跳过之前批量假设测试的数量，设置成0，表示使用之前所有的批量测试的积累数据。

[`StreamingTest`](https://spark.apache.org/docs/2.1.1/api/java/index.html#org.apache.spark.mllib.stat.test.StreamingTest) provides streaming hypothesis testing.

[`StreamingTest`](https://spark.apache.org/docs/2.1.1/api/java/index.html#org.apache.spark.mllib.stat.test.StreamingTest) 提供了流式的假设测试。

```java
import org.apache.spark.mllib.stat.test.BinarySample;
import org.apache.spark.mllib.stat.test.StreamingTest;
import org.apache.spark.mllib.stat.test.StreamingTestResult;

JavaDStream<BinarySample> data = ssc.textFileStream(dataDir).map(
  new Function<String, BinarySample>() {
    @Override
    public BinarySample call(String line) {
      String[] ts = line.split(",");
      boolean label = Boolean.parseBoolean(ts[0]);
      double value = Double.parseDouble(ts[1]);
      return new BinarySample(label, value);
    }
  });

StreamingTest streamingTest = new StreamingTest()
  .setPeacePeriod(0)
  .setWindowSize(0)
  .setTestMethod("welch");

JavaDStream<StreamingTestResult> out = streamingTest.registerStream(data);
out.print();
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/mllib/JavaStreamingTestExample.java" in the Spark repo.

## Random data generation

Random data generation is useful for randomized algorithms, prototyping, and performance testing. `spark.mllib` supports generating random RDDs with i.i.d. values drawn from a given distribution: uniform, standard normal, or Poisson.

随机数生成是一个非常有用，它对随即算法，原型和性能测试都很有用。 `spark.mllib` 支持生成随机的 RDD， 以 i.i.d. 为结果，它是从这几个给定的分布中获取的: 均匀分布, 标准正态分布, 或者是泊松分布。

[`RandomRDDs`](https://spark.apache.org/docs/2.1.1/api/java/index.html#org.apache.spark.mllib.random.RandomRDDs) provides factory methods to generate random double RDDs or vector RDDs. The following example generates a random double RDD, whose values follows the standard normal distribution `N(0, 1)`, and then map it to `N(1, 4)`.

Refer to the [`RandomRDDs` Java docs](https://spark.apache.org/docs/2.1.1/api/java/org/apache/spark/mllib/random/RandomRDDs) for details on the API.

[`RandomRDDs`](https://spark.apache.org/docs/2.1.1/api/java/index.html#org.apache.spark.mllib.random.RandomRDDs) 提供方法工厂来随机的浮点型和RDD和向量类型的RDD。下面的例子是生成随机的浮点型的RDD，它的值遵循标准正态分布 `N(0, 1)`, 然后再map到第二个正态分布`N(1, 4)`.

更多信息请参考 [`RandomRDDs` Java docs](https://spark.apache.org/docs/2.1.1/api/java/org/apache/spark/mllib/random/RandomRDDs) 。

```java
import org.apache.spark.SparkContext;
import org.apache.spark.api.JavaDoubleRDD;
import static org.apache.spark.mllib.random.RandomRDDs.*;

JavaSparkContext jsc = ...

// Generate a random double RDD that contains 1 million i.i.d. values drawn from the
// standard normal distribution `N(0, 1)`, evenly distributed in 10 partitions.
JavaDoubleRDD u = normalJavaRDD(jsc, 1000000L, 10);
// Apply a transform to get a random double RDD following `N(1, 4)`.
JavaDoubleRDD v = u.map(
  new Function<Double, Double>() {
    public Double call(Double x) {
      return 1.0 + 2.0 * x;
    }
  });
```

## Kernel density estimation

核密度估计

[Kernel density estimation](https://en.wikipedia.org/wiki/Kernel_density_estimation) is a technique useful for visualizing empirical probability distributions without requiring assumptions about the particular distribution that the observed samples are drawn from. It computes an estimate of the probability density function of a random variables, evaluated at a given set of points. It achieves this estimate by expressing the PDF of the empirical distribution at a particular point as the the mean of PDFs of normal distributions centered around each of the samples.

[Kernel density estimation](https://en.wikipedia.org/wiki/Kernel_density_estimation) 是一个用来可视化经验概率分布的技术，且不用提供关于特殊的用于生成数据的分布假设。它计算一个随机变量概率密度函数的一个估值，用来评估给定的集合中的点。它通过压缩PDF来实现这个估值，这个PDF是通过对特殊点的经验分布，然后从实际点中以正态分布中心周围的数据的平均点中找出来的。

[`KernelDensity`](https://spark.apache.org/docs/2.1.1/api/java/index.html#org.apache.spark.mllib.stat.KernelDensity) provides methods to compute kernel density estimates from an RDD of samples. The following example demonstrates how to do so.

Refer to the [`KernelDensity` Java docs](https://spark.apache.org/docs/2.1.1/api/java/org/apache/spark/mllib/stat/KernelDensity.html) for details on the API.

[`KernelDensity`](https://spark.apache.org/docs/2.1.1/api/java/index.html#org.apache.spark.mllib.stat.KernelDensity) 提供方法来通过RDD的样本中计算核密度估算，下面以代码的形似告诉了你怎么做.

更多信息请参考[`KernelDensity` Java docs](https://spark.apache.org/docs/2.1.1/api/java/org/apache/spark/mllib/stat/KernelDensity.html)。

```java
import java.util.Arrays;

import org.apache.spark.api.java.JavaRDD;
import org.apache.spark.mllib.stat.KernelDensity;

// an RDD of sample data
JavaRDD<Double> data = jsc.parallelize(
  Arrays.asList(1.0, 1.0, 1.0, 2.0, 3.0, 4.0, 5.0, 5.0, 6.0, 7.0, 8.0, 9.0, 9.0));

// Construct the density estimator with the sample data
// and a standard deviation for the Gaussian kernels
KernelDensity kd = new KernelDensity().setSample(data).setBandwidth(3.0);

// Find density estimates for the given values
double[] densities = kd.estimate(new double[]{-1.0, 2.0, 5.0});

System.out.println(Arrays.toString(densities));
```

Find full example code at "examples/src/main/java/org/apache/spark/examples/mllib/JavaKernelDensityEstimationExample.java" in the Spark repo.







