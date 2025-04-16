# Database-Systems_CMU15445
This is the CMU15445-Database Systems course project, consisting of five lab assignments. 

在本课程项目中，需要完成关系数据库bustub的磁盘页面缓冲池、索引功能、查询执行功能以及并发控制功能。并在课程中学习关如下知识：

关系数据库的基本概念，以及基于SQLite简易关系数据库的SQL语句语法及其实践；
关系数据库中页面存储、页面布局及元组布局等数据库物理存储模型，以及DBMS中的页面缓冲池的功能及原理；
关系数据库中索引的功能及分类，以及基于B+-tree和可扩展哈希表的可磁盘备份索引的具体实现及并发控制；
关系数据库中元组排序、聚合、连接功能的具体实现和性能分析，连接、聚合、插入等物理查询计划的具体执行策略，以及基于启发式和成本模型的查询计划优化方法；
关系数据库中事务的基本概念及其所应达到的标准ACID(原子性、一致性、隔离性、持久性)的定义及要求；
关系数据库中并发控制协议（即执行多个事务的方式）的悲观及乐观实现策略，如两阶段锁、基于时间戳并发控制、多版本并发控制；
关系数据库中保证原子性、隔离性、持久性的崩溃恢复算法的策略，包括日志、检查点、重做和撤销操作；

In this course project, it is required to complete the implementation of key functionalities in the relational database Bustub, including disk page buffer pool, indexing, query execution, and concurrency control. The course covers the following topics:

- Basic concepts of relational databases, as well as SQL syntax and practical applications based on SQLite, a simple relational database;
- Physical storage models in databases, such as page storage, page layout, and tuple layout, as well as the functionality and principles of the page buffer pool in a DBMS;
- The functionality and categories of indexes in relational databases, along with the implementation and concurrency control of disk-backed indexes based on B+-tree and extendible hashing;
- Implementation and performance analysis of tuple sorting, aggregation, and join functionalities in relational databases, execution strategies for physical query plans like joins, aggregations, and insertions, as well as query optimization methods based on heuristics and cost models;
- Basic concepts of transactions in relational databases and the standards they should meet (ACID: Atomicity, Consistency, Isolation, Durability) along with their definitions and requirements;
- Pessimistic and optimistic concurrency control protocols (i.e., methods for executing multiple transactions) in relational databases, such as two-phase locking, timestamp-based concurrency control, and multi-version concurrency control;
- Crash recovery algorithms to ensure atomicity, isolation, and durability in relational databases, including strategies like logging, checkpoints, redo, and undo operations.



---


## Project 0 : C++ Primer

本实验的目的为编写一组简单的矩阵运算类，以检验实验者的C++编程能力，并帮助实验者熟悉本课程实验的环境配置及提交流程。


### 代码实现

在本实验中需要实现三个类，其中Matrix作为矩阵类型的基类，存放指向实际矩阵元素数组的指针；RowMatrix 为 Matrix 的子类，存放指向各矩阵行的指针，并提供按行列数访问矩阵元素的接口；RowMatrixOperations提供矩阵相加、相乘函数的接口。本实验的逻辑比较简单，主要考察对C++继承和模板的使用。


### Matrix
```
 28 template <typename T>
 29 class Matrix {
 30  protected:
 31   /**
 32    * TODO(P0): Add implementation
 33    *
 34    * Construct a new Matrix instance.
 35    * @param rows The number of rows
 36    * @param cols The number of columns
 37    *
 38    */
 39   Matrix(int rows, int cols) : rows_(rows), cols_(cols) { linear_ = new T[rows * cols + 1]; }
...
 95   /**
 96    * Destroy a matrix instance.
 97    * TODO(P0): Add implementation
 98    */
 99   virtual ~Matrix() { delete[] linear_; }
100 };
```
在构造函数和析构函数中分别分配、销毁用于存放矩阵元素的内存即可。


### RowMatrix
```
106 template <typename T>
107 class RowMatrix : public Matrix<T> {
108  public:
109   /**
110    * TODO(P0): Add implementation
111    *
112    * Construct a new RowMatrix instance.
113    * @param rows The number of rows
114    * @param cols The number of columns
115    */
116   RowMatrix(int rows, int cols) : Matrix<T>(rows, cols) {
117     data_ = new T *[rows];
118     for (int i = 0; i < rows; i++) {
119       data_[i] = &this->linear_[i * cols];
120     }
121   }
...
197   ~RowMatrix() override { delete[] data_; }
```
构造函数中，使得行指针分别指向矩阵各行的首元素指针即可，并在析构函数中释放行指针。


<strong>为什么这样设计？</strong>

内存局部性：linear_ 是连续分配的，适合缓存优化，提高访问效率。

灵活性：RowMatrix 可以通过 data_ 提供行指针，方便某些算法（如行交换只需交换指针，无需复制数据）。

避免多层 new：如果直接 new T[rows][cols]，可能会涉及多次内存分配，而这种方式只需一次大块分配 + 一个行指针数组。


```
linear_ = [0,1,2,3, 4,5,6,7, 8,9,10,11, X]  // X 是多余的元素
data_ = [
    &linear_[0] → [0,1,2,3],
    &linear_[4] → [4,5,6,7],
    &linear_[8] → [8,9,10,11]
]
---------
data_ = [
    &linear_[0],  // 第 0 行的起始地址 → 指向 (0,0)
    &linear_[4],  // 第 1 行的起始地址 → 指向 (1,0)
    &linear_[8]   // 第 2 行的起始地址 → 指向 (2,0)
]
---------
data_[i][j] == *(data_[i] + j) == linear_[i * cols + j]

```
以上是举了一个例子；

```
147   T GetElement(int i, int j) const override {
148     if (i < 0 || j < 0 || i >= this->rows_ || j >= this->cols_) {
149       throw Exception(ExceptionType::OUT_OF_RANGE, "GetElement(i,j) out of range");
150     }
151     return data_[i][j];
152   }
...
164   void SetElement(int i, int j, T val) override {
165     if (i < 0 || j < 0 || i >= this->rows_ || j >= this->cols_) {
166       throw Exception(ExceptionType::OUT_OF_RANGE, "SetElement(i,j) out of range");
167     }
168     data_[i][j] = val;
169   }
...
182   void FillFrom(const std::vector<T> &source) override {
183     int sz = static_cast<int>(source.size());
184     if (sz != this->rows_ * this->cols_) {
185       throw Exception(ExceptionType::OUT_OF_RANGE, "FillFrom out of range");
186     }
187     for (int i = 0; i < sz; i++) {
188       this->linear_[i] = source[i];
189     }
190   }
```
利用行指针提取和设置矩阵元素，在提供的行列值超出范围时抛出异常。


### RowMatrixOperations
```
225   static std::unique_ptr<RowMatrix<T>> Add(const RowMatrix<T> *matrixA, const RowMatrix<T> *matrixB) {
226     // TODO(P0): Add implementation
227     if (matrixA->GetRowCount() != matrixB->GetRowCount() || matrixA->GetColumnCount() != matrixB->GetColumnCount()) {
228       return std::unique_ptr<RowMatrix<T>>(nullptr);
229     }
230     int rows = matrixA->GetRowCount();
231     int cols = matrixB->GetColumnCount();
232     std::unique_ptr<RowMatrix<T>> ret(new RowMatrix<T>(rows, cols));
233     for (int i = 0; i < rows; i++) {
234       for (int j = 0; j < cols; j++) {
235         ret->SetElement(i, j, matrixA->GetElement(i, j) + matrixB->GetElement(i, j));
236       }
237     }
238     return ret;
239   }
...
248   static std::unique_ptr<RowMatrix<T>> Multiply(const RowMatrix<T> *matrixA, const RowMatrix<T> *    matrixB) {
249     // TODO(P0): Add implementation
250     if (matrixA->GetColumnCount() != matrixB->GetRowCount()) {
251       return std::unique_ptr<RowMatrix<T>>(nullptr);
252     }
253     int rows = matrixA->GetRowCount();
254     int cols = matrixB->GetColumnCount();
255     int tmp = matrixA->GetColumnCount();
256     std::unique_ptr<RowMatrix<T>> ret(new RowMatrix<T>(rows, cols));
257     for (int i = 0; i < rows; i++) {
258       for (int j = 0; j < cols; j++) {
259         T element = 0;
260         for (int k = 0; k < tmp; k++) {
261           element += matrixA->GetElement(i, k) * matrixB->GetElement(k, j);
262         }
263         ret->SetElement(i, j, element);
264       }
265     }
266     return ret;
267   }
...
277   static std::unique_ptr<RowMatrix<T>> GEMM(const RowMatrix<T> *matrixA, const RowMatrix<T> *matr    ixB,
278                                             const RowMatrix<T> *matrixC) {
279     // TODO(P0): Add implementation
280     if (matrixA->GetColumnCount() != matrixB->GetRowCount()) {
281       return std::unique_ptr<RowMatrix<T>>(nullptr);
282     }
283     int rows = matrixA->GetRowCount();
284     int cols = matrixB->GetColumnCount();
285     int tmp = matrixA->GetColumnCount();
286     if (rows != matrixC->GetRowCount() || cols != matrixC->GetColumnCount()) {
287       return std::unique_ptr<RowMatrix<T>>(nullptr);
288     }
289     std::unique_ptr<RowMatrix<T>> ret(new RowMatrix<T>(rows, cols));
290     for (int i = 0; i < rows; i++) {
291       for (int j = 0; j < cols; j++) {
292         T element = matrixC->GetElement(i, j);
293         for (int k = 0; k < tmp; k++) {
294           element += matrixA->GetElement(i, k) * matrixB->GetElement(k, j);
295         }
296         ret->SetElement(i, j, element);
297       }
298     }
299     return ret;
300   }
```
以上三个矩阵操作函数的逻辑都比较简单，唯一需要注意的是在运算前应检查两矩阵的规格是否匹配。



