### 数据类型 基于RDD的API

* [Local vector](#local-vector)
* [Labeled point](#Labeled point)
* [Local matrix](#local-matrix)
* [Distributed matrix](#Distributed matrix)
  * [RowMatrix](#RowMatrix)
  * [IndexedRowMatrix](#IndexedRowMatrix)
  * [CoordinateMatrix](#CoordinateMatrix)
  * [BlockMatrix](#BlockMatrix)

MLlib supports local vectors and matrices stored on a single machine, as well as distributed matrices backed by one or more RDDs. Local vectors and local matrices are simple data models that serve as public interfaces. The underlying linear algebra operations are provided by [Breeze](http://www.scalanlp.org/). A training example used in supervised learning is called a “labeled point” in MLlib.

MLlib支持存储于单机的本地向量和矩阵，就像分布式矩阵在后台被分成一个或者多个RDD那样。本地向量和本地矩阵都是一种简单的数据模式，他们提供公共接口服务。spark中的基本的线性代数运算是由 [Breeze](http://www.scalanlp.org/) 提供的。在MLlib中监督学习（学习分为监督学习和非监督学习）中的一个训练样本叫做“labeled point”。

## Local vector (本地向量)

A local vector has integer-typed and 0-based indices and double-typed values, stored on a single machine. MLlib supports two types of local vectors: dense and sparse. A dense vector is backed by a double array representing its entry values, while a sparse vector is backed by two parallel arrays: indices and values. For example, a vector `(1.0, 0.0, 3.0)` can be represented in dense format as `[1.0, 0.0, 3.0]` or in sparse format as `(3, [0, 2], [1.0, 3.0])`, where `3` is the size of the vector.

本地向量有三种数据类型，分别是整型，零型和浮点型，都是单机存储方式。MLlib支持两种类型的本地向量：紧凑型和松散型。紧凑型向量使用浮点型数据的一个数组代表他的值，而松散型的向量使用两个平行的数组：索引数组和数据数组，例如，一个向量是`(1.0, 0.0, 3.0)`既能用紧凑型的格式`[1.0, 0.0, 3.0]`又能用松散型的格式`(3, [0, 2], [1.0, 3.0])`表示,这里面的3表示向量的大小，没有数据的地方统一使用0.0替换。

The base class of local vectors is [`Vector`](http://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/Vector.html), and we provide two implementations: [`DenseVector`](http://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/DenseVector.html) and [`SparseVector`](http://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/SparseVector.html). We recommend using the factory methods implemented in [`Vectors`](http://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/Vectors.html) to create local vectors.

Refer to the [`Vector` Java docs](http://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/Vector.html) and [`Vectors` Java docs](http://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/Vectors.html) for details on the API.

本地线程对应的基本的类(接口)是[`Vector`](http://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/Vector.html)，提供两种实现：[`DenseVector`](http://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/DenseVector.html) and [`SparseVector`](http://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/SparseVector.html). 我们建议你使用一个实现[`Vector`](http://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/Vector.html)方法的工厂的方法来创建本地向量。

详细信息请参考 [`Vector` Java docs](http://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/Vector.html) and [`Vectors` Java docs](http://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/Vectors.html) 

```java
import org.apache.spark.mllib.linalg.Vector;
import org.apache.spark.mllib.linalg.Vectors;

// Create a dense vector (1.0, 0.0, 3.0).
Vector dv = Vectors.dense(1.0, 0.0, 3.0);
// Create a sparse vector (1.0, 0.0, 3.0) by specifying its indices and values corresponding to nonzero entries.
Vector sv = Vectors.sparse(3, new int[] {0, 2}, new double[] {1.0, 3.0});
```

## Labeled point 

A labeled point is a local vector, either dense or sparse, associated with a label/response. In MLlib, labeled points are used in supervised learning algorithms. We use a double to store a label, so we can use labeled points in both regression and classification. For binary classification, a label should be either `0` (negative) or `1` (positive). For multiclass classification, labels should be class indices starting from zero: `0, 1, 2, ...`.

labeled point也是一种本地向量，但是它既不是紧凑型的又不是松散型的，与 label/response有关。在MLlib中，labeled points被用在监督学习算法中。我们使用浮点型数据来存储label，所以我们可以在回归和分类中使用labeled points,对于二进制的分类，label只能是0（表示负）或者1（表示正）。对于多类分类，labels应该是从0开始的类别索引:`0`,`1`,`2`,...

A labeled point is represented by [`LabeledPoint`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/regression/LabeledPoint.html).

Refer to the [`LabeledPoint` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/regression/LabeledPoint.html) for details on the API.

在spark中使用[`LabeledPoint`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/regression/LabeledPoint.html)这个类来表示labeled point，详细信息请参考[`LabeledPoint` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/regression/LabeledPoint.html) 

```java
import org.apache.spark.mllib.linalg.Vectors;
import org.apache.spark.mllib.regression.LabeledPoint;

// Create a labeled point with a positive label and a dense feature vector.
LabeledPoint pos = new LabeledPoint(1.0, Vectors.dense(1.0, 0.0, 3.0));

// Create a labeled point with a negative label and a sparse feature vector.
LabeledPoint neg = new LabeledPoint(0.0, Vectors.sparse(3, new int[] {0, 2}, new double[] {1.0, 3.0}));
```

## Sparse data

稀疏数据

It is very common in practice to have sparse training data. MLlib supports reading training examples stored in `LIBSVM` format, which is the default format used by [`LIBSVM`](http://www.csie.ntu.edu.tw/~cjlin/libsvm/) and [`LIBLINEAR`](http://www.csie.ntu.edu.tw/~cjlin/liblinear/). It is a text format in which each line represents a labeled sparse feature vector using the following format:

使用稀疏的训练数据来来练习是很常见的。MLlib支持读取以`LIBSVM`存储的格式的数据，这种格式被[`LIBSVM`](http://www.csie.ntu.edu.tw/~cjlin/libsvm/) and [`LIBLINEAR`](http://www.csie.ntu.edu.tw/~cjlin/liblinear/)默认使用。他是一种text格式的，它当中的每一行代表着使用以下格式标记的稀疏特征向量：

```
label index1:value1 index2:value2 ...
```

where the indices are one-based and in ascending order. After loading, the feature indices are converted to zero-based.

这些索引是按1开始，升序排序的，在被加载之后，其中的特征索引就被转换成了0.

[`MLUtils.loadLibSVMFile`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/util/MLUtils.html) reads training examples stored in LIBSVM format.

Refer to the [`MLUtils` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/util/MLUtils.html) for details on the API.

[`MLUtils.loadLibSVMFile`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/util/MLUtils.html)读取训练数据存储成`LIBSVM`格式。

详细信息请参考[`MLUtils` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/util/MLUtils.html)

```java
import org.apache.spark.mllib.regression.LabeledPoint;
import org.apache.spark.mllib.util.MLUtils;
import org.apache.spark.api.java.JavaRDD;

JavaRDD<LabeledPoint> examples = 
  MLUtils.loadLibSVMFile(jsc.sc(), "data/mllib/sample_libsvm_data.txt").toJavaRDD();
```

## Local matrix 

A local matrix has integer-typed row and column indices and double-typed values, stored on a single machine. MLlib supports dense matrices, whose entry values are stored in a single double array in column-major order, and sparse matrices, whose non-zero entry values are stored in the Compressed Sparse Column (CSC) format in column-major order. For example, the following dense matrix

​                                                                $$\begin{pmatrix}  1.0 & 2.0 \\ 3.0 & 4.0 \\  5.0 & 6.0  \end{pmatrix}$$

is stored in a one-dimensional array `[1.0, 3.0, 5.0, 2.0, 4.0, 6.0]` with the matrix size `(3, 2)`.

一个本地矩阵有一个整数类型的行和索引列和浮点型的值，存储与单机。MLlib支持所有数据都存储在单个浮点型的数组中按列为主要顺序的密集矩阵，和没有0的存储于压缩松散列(CSC)格式以列为主要顺序的松散型矩阵。例如，以下是一个密集型的矩阵

​                                                                $$\begin{pmatrix}  1.0 & 2.0 \\ 3.0 & 4.0 \\  5.0 & 6.0  \end{pmatrix}$$

它存放于一个一维空间的数组中`[1.0, 3.0, 5.0, 2.0, 4.0, 6.0]` 矩阵的大小是`(3,2)`

The base class of local matrices is [`Matrix`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/Matrix.html), and we provide two implementations: [`DenseMatrix`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/DenseMatrix.html), and [`SparseMatrix`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/SparseMatrix.html). We recommend using the factory methods implemented in [`Matrices`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/Matrices.html) to create local matrices. Remember, local matrices in MLlib are stored in column-major order.

Refer to the [`Matrix` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/Matrix.html) and [`Matrices` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/Matrices.html) for details on the API.

本地矩阵对应的类是 [`Matrix`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/Matrix.html), 它提供了连个实现类[`DenseMatrix`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/DenseMatrix.html), and [`SparseMatrix`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/SparseMatrix.html). 

我们建议使用工厂方法的方式实现[`Matrices`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/Matrices.html)来创建一个本地矩阵。记住，在MLlib中本地矩阵以列为主要顺序存储。

```java
import org.apache.spark.mllib.linalg.Matrix;
import org.apache.spark.mllib.linalg.Matrices;

// Create a dense matrix ((1.0, 2.0), (3.0, 4.0), (5.0, 6.0))
Matrix dm = Matrices.dense(3, 2, new double[] {1.0, 3.0, 5.0, 2.0, 4.0, 6.0});

// Create a sparse matrix ((9.0, 0.0), (0.0, 8.0), (0.0, 6.0))
Matrix sm = Matrices.sparse(3, 2, new int[] {0, 1, 3}, new int[] {0, 2, 1}, new double[] {9, 6, 8});
```

## Distributed matrix

分布式的矩阵

A distributed matrix has long-typed row and column indices and double-typed values, stored distributively in one or more RDDs. It is very important to choose the right format to store large and distributed matrices. Converting a distributed matrix to a different format may require a global shuffle, which is quite expensive. Four types of distributed matrices have been implemented so far.

一个分布式的矩阵有long类型的行和索引列和浮点型的值，以分布式的方式存储在一个或者多个RDD中。对于存储大量的分布式的矩阵选择一个对的格式很重要。将一个分布式的矩阵转换成不同的格式可能需要一个全局的重洗，这个操作是代价是很大的。目前我们实现了四种类型的分布式矩阵。

The basic type is called `RowMatrix`. A `RowMatrix` is a row-oriented distributed matrix without meaningful row indices, e.g., a collection of feature vectors. It is backed by an RDD of its rows, where each row is a local vector. We assume that the number of columns is not huge for a `RowMatrix` so that a single local vector can be reasonably communicated to the driver and can also be stored / operated on using a single node. An `IndexedRowMatrix` is similar to a `RowMatrix` but with row indices, which can be used for identifying rows and executing joins. A `CoordinateMatrix` is a distributed matrix stored in [coordinate list (COO)](https://en.wikipedia.org/wiki/Sparse_matrix#Coordinate_list_.28COO.29)format, backed by an RDD of its entries. A `BlockMatrix` is a distributed matrix backed by an RDD of `MatrixBlock` which is a tuple of `(Int, Int, Matrix)`.

基本类型是`RowMatrix`，`RowMatrix`是一种面向行的分布式矩阵，其中不存在有意义的行索引，例如：一组特征向量。它以它的行的RDD为支撑，其中每一行就是一个本地向量。我们假设`RowMatrix`的列的长度不会很大，所以单个的本地向量能够合理的于驱动沟通，也可以在单个节点上读取和操作。`IndexedRowMatrix`很像`RowMatrix`但是他有行索引，能够用它来定义行，和执行join操作，`CoordinateMatrix`是一种分布式矩阵，以 [coordinate list (COO)](https://en.wikipedia.org/wiki/Sparse_matrix#Coordinate_list_.28COO.29)的方式存储，它由它的每一项数据组成的RDD来支撑。`BlockMatrix`也是一种分布式矩阵，它以`MatrixBlock`组成的RDD为支撑，他是一个`(Int, Int, Matrix)`类型的元组。

**Note**

The underlying RDDs of a distributed matrix must be deterministic, because we cache the matrix size. In general the use of non-deterministic RDDs can lead to errors.

注意

底层的分布式矩阵的RDDs一定要是确定性的，因为我们会缓存矩阵的大小，一般情况下，使用不确定性的RDDs会导致报错。

## RowMatrix

A `RowMatrix` is a row-oriented distributed matrix without meaningful row indices, backed by an RDD of its rows, where each row is a local vector. Since each row is represented by a local vector, the number of columns is limited by the integer range but it should be much smaller in practice.

`RowMatrix`是一种面向行的分布式矩阵，其中不存在有意义的行索引，例如：一组特征向量。它以它的行的RDD为支撑，其中每一行就是一个本地向量，它的列数被限制在整数范围内，但是实际使用中应该要很小。

A [`RowMatrix`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/RowMatrix.html) can be created from a `JavaRDD<Vector>` instance. Then we can compute its column summary statistics.

Refer to the [`RowMatrix` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/RowMatrix.html) for details on the API.

 [`RowMatrix`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/RowMatrix.html) 能够从 `JavaRDD<Vector>` 中创建一个实例。然后我们就能计算它的列的汇总统计。

详细信息请参考 [`RowMatrix` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/RowMatrix.html) 

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

## IndexedRowMatrix

An `IndexedRowMatrix` is similar to a `RowMatrix` but with meaningful row indices. It is backed by an RDD of indexed rows, so that each row is represented by its index (long-typed) and a local vector.

`IndexedRowMatrix`很像`RowMatrix`但是他有有意义的行索引，能够用它来定义行，它由索引化的行的RDD来支撑，所以它的每一行都代表着它的long类型的索引和一个本地向量。

An [`IndexedRowMatrix`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/IndexedRowMatrix.html) can be created from an `JavaRDD<IndexedRow>` instance, where [`IndexedRow`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/IndexedRow.html) is a wrapper over `(long, Vector)`. An `IndexedRowMatrix` can be converted to a `RowMatrix` by dropping its row indices.

Refer to the [`IndexedRowMatrix` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/IndexedRowMatrix.html) for details on the API.

 [`IndexedRowMatrix`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/IndexedRowMatrix.html) 能够通过 `JavaRDD<IndexedRow>` 来创建一个实例，[`IndexedRow`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/IndexedRow.html) 是一个`(long, Vector)`格式的包装器，`IndexedRowMatrix` 能够通过丢掉行索引之后转换成`RowMatrix` 。

更多信息去请参考 [`IndexedRowMatrix` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/IndexedRowMatrix.html) 

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

## CoordinateMatrix

A `CoordinateMatrix` is a distributed matrix backed by an RDD of its entries. Each entry is a tuple of `(i: Long, j: Long, value: Double)`, where `i` is the row index, `j` is the column index, and `value` is the entry value. A `CoordinateMatrix` should be used only when both dimensions of the matrix are huge and the matrix is very sparse.

 `CoordinateMatrix` 是一个由它的项组成的RDD支撑的. 每一项都是 `(i: Long, j: Long, value: Double)`格式的元组, q其中的`i`是行索引, `j是列索引,  `value` 是每一项的值.  `CoordinateMatrix` 应该仅仅使用在维数很大和非常松散的矩阵中。

A [`CoordinateMatrix`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/CoordinateMatrix.html) can be created from a `JavaRDD<MatrixEntry>` instance, where [`MatrixEntry`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/MatrixEntry.html) is a wrapper over `(long, long, double)`. A `CoordinateMatrix` can be converted to an `IndexedRowMatrix` with sparse rows by calling `toIndexedRowMatrix`. Other computations for `CoordinateMatrix` are not currently supported.

Refer to the [`CoordinateMatrix` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/CoordinateMatrix.html) for details on the API.

 [`CoordinateMatrix`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/CoordinateMatrix.html) 能够通过 `JavaRDD<MatrixEntry>` 来创建一个实例，其中的 [`MatrixEntry`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/MatrixEntry.html) 是一个`(long, long, double)`格式的包装器。  `CoordinateMatrix` 能够通过调用`toIndexedRowMatrix`被转换成包含有散列行的 `IndexedRowMatrix` . 其他算法的 `CoordinateMatrix` 目前不支持。

更多详情请参考 [`CoordinateMatrix` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/CoordinateMatrix.html) 。

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

## BlockMatrix

A `BlockMatrix` is a distributed matrix backed by an RDD of `MatrixBlock`s, where a `MatrixBlock` is a tuple of `((Int, Int), Matrix)`, where the `(Int, Int)` is the index of the block, and `Matrix` is the sub-matrix at the given index with size `rowsPerBlock` x `colsPerBlock`. `BlockMatrix` supports methods such as `add` and `multiply` with another `BlockMatrix`.`BlockMatrix` also has a helper function `validate` which can be used to check whether the `BlockMatrix` is set up properly.

 `BlockMatrix` 是一个由多个 `MatrixBlock`组成的RDD支撑的分布式矩阵, 其中的 `MatrixBlock` 是一个 `((Int, Int), Matrix)`类型的元组，其中`(Int, Int)` 是一个块的索引, `Matrix`是一个给出了索引大小的`rowsPerBlock` 乘以`colsPerBlock`矩阵的子矩阵. `BlockMatrix` 支持两个`BlockMatrix`之间的加法和乘法等方法。`BlockMatrix` 有一个 `validate` 方法，它可以用来检查 `BlockMatrix` 是否设置正确.

A [`BlockMatrix`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/BlockMatrix.html) can be most easily created from an `IndexedRowMatrix` or `CoordinateMatrix` by calling `toBlockMatrix`.`toBlockMatrix` creates blocks of size 1024 x 1024 by default. Users may change the block size by supplying the values through `toBlockMatrix(rowsPerBlock, colsPerBlock)`.

Refer to the [`BlockMatrix` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/BlockMatrix.html) for details on the API.

 [`BlockMatrix`](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/BlockMatrix.html) 能够很容易的通过 `IndexedRowMatrix` 或者 `CoordinateMatrix` 通过访问`BlockMatrix`来创建实例。`toBlockMatrix` 默认创建的块的大小是 1024 x 1024 . 用户能够通过`toBlockMatrix(rowsPerBlock, colsPerBlock)`方法来改变块的大小  。

Refer to the [`BlockMatrix` Java docs](https://spark.apache.org/docs/latest/api/java/org/apache/spark/mllib/linalg/distributed/BlockMatrix.html) for details on the API.

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

