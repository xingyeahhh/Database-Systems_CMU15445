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

矩阵乘法示意：

<img src="https://github.com/user-attachments/assets/95ebb8c8-7415-44d0-93f2-a72a3ddc8e4d" 
     alt="image" 
     style="width:30%; max-width:600px;">



---

## Project 1 : BUFFER POOL

在本实验中，需要在存储管理器中实现缓冲池。缓冲池负责将物理页面从磁盘中读入内存、或从内存中写回磁盘，使得DBMS可以支持大于内存大小的存储容量。并且，缓冲池应当是用户透明且线程安全的。

### Task1 : LRU REPLACEMENT POLICY
本部分中需要实现缓冲池中的LRUReplacer，该组件的功能是跟踪缓冲池内的页面使用情况，并在缓冲池容量不足时驱除缓冲池中最近最少使用的页面。其应当具备如下接口：

- Victim(frame_id_t*)：驱逐缓冲池中最近最少使用的页面，并将其内容存储在输入参数中。当LRUReplacer为空时返回False，否则返回True；
- Pin(frame_id_t)：当缓冲池中的页面被用户访问时，该方法被调用使得该页面从LRUReplacer中驱逐，以使得该页面固定在缓存池中；
- Unpin(frame_id_t)：当缓冲池的页面被所有用户使用完毕时，该方法被调用使得该页面被添加在LRUReplacer，使得该页面可以被缓冲池驱逐；
- Size()：返回LRUReplacer中页面的数目；

```
 28 class LRUReplacer : public Replacer {
 29  public:
 30   /**
 31    * Create a new LRUReplacer.
 32    * @param num_pages the maximum number of pages the LRUReplacer will be required to store
 33    */
 34   explicit LRUReplacer(size_t num_pages);
 35   
 36   /**
 37    * Destroys the LRUReplacer.
 38    */
 39   ~LRUReplacer() override;
 40   
 41   bool Victim(frame_id_t *frame_id) override;
 42 
 43   void Pin(frame_id_t frame_id) override;
 44   
 45   void Unpin(frame_id_t frame_id) override;
 46   
 47   size_t Size() override;
 48 
 49   void DeleteNode(LinkListNode *curr);
 50 
 51  private:
 52   // TODO(student): implement me!
 53   std::unordered_map<frame_id_t, std::list<frame_id_t>::iterator> data_idx_;
 54   std::list<frame_id_t> data_;
 55   std::mutex data_latch_;
 56 };
```

在这里，LRU策略可以由哈希表加双向链表的方式实现，其中链表充当队列的功能以记录页面被访问的先后顺序，哈希表则记录<页面ID - 链表节点>键值对，以在O(1)复杂度下删除链表元素。实际实现中使用STL中的哈希表unordered_map和双向链表list，并在unordered_map中存储指向链表节点的list::iterator。

```
 21 bool LRUReplacer::Victim(frame_id_t *frame_id) {
 22   data_latch_.lock();
 23   if (data_idx_.empty()) {
 24     data_latch_.unlock();
 25     return false;
 26   }
 27   *frame_id = data_.front();
 28   data_.pop_front();
 29   data_idx_.erase(*frame_id);
 30   data_latch_.unlock();
 31   return true;
 32 }
```

对于Victim，首先判断链表是否为空，如不为空则返回链表首节点的页面ID，并在哈希表中解除指向首节点的映射。为了保证线程安全，整个函数应当由mutex互斥锁保护，下文中对互斥锁操作不再赘述。

- 链表头部（front） 通常存放 最近最少使用（LRU） 的页面（即最久未访问的）
- 链表尾部（back） 通常存放 最近使用（MRU） 的页面（即最新访问的）
- 参数 frame_id_t *frame_id 的作用是一个 输出型参数（output parameter），它的作用是： 调用方（比如缓存管理器）传入一个 frame_id_t 变量的地址; Victim 方法 找到要淘汰的 frame 后，把结果 写入这个地址指向的内存,这样调用方就能拿到被淘汰的 frame ID.
- 为什么用指针而不是返回值？ 返回值 bool 只用来表示成功/失败（是否有 victim）,实际的 victim frame ID 通过指针参数返回,这是 C/C++ 中常见的**多值返回**设计模式.

```
 34 void LRUReplacer::Pin(frame_id_t frame_id) {
 35   data_latch_.lock();
 36   auto it = data_idx_.find(frame_id);
 37   if (it != data_idx_.end()) {
 38     data_.erase(it->second);
 39     data_idx_.erase(it);
 40   }
 41   data_latch_.unlock();
 42 }
```

对于Pin，其检查LRUReplace中是否存在对应页面ID的节点，如不存在则直接返回，如存在对应节点则通过哈希表中存储的迭代器删除链表节点，并解除哈希表对应页面ID的映射。

```
 44 void LRUReplacer::Unpin(frame_id_t frame_id) {
 45   data_latch_.lock();
 46   auto it = data_idx_.find(frame_id);
 47   if (it == data_idx_.end()) {
 48     data_.push_back(frame_id);
 49     data_idx_[frame_id] = prev(data_.end());
 50   }
 51   data_latch_.unlock();
 52 }
```

对于Unpin，其检查LRUReplace中是否存在对应页面ID的节点，如存在则直接返回，如不存在则在链表尾部插入页面ID的节点，并在哈希表中插入<页面ID - 链表尾节点>映射。

**Pin (固定)**

- 当一个页面被"Pin"时，意味着它正在被某个事务或操作使用

- 被Pin的页面不能被缓冲池替换算法(如LRU)驱逐

- 在代码中，Pin操作会从LRU替换器中移除该页面

**Unpin (取消固定)**

- 当一个页面被"Unpin"时，意味着它不再被任何事务或操作使用

- 被Unpin的页面可以被缓冲池替换算法考虑驱逐

- 在代码中，Unpin操作会将页面添加回LRU替换器

**这与数据库的事务安全和并发控制有关：**

保护正在使用的页面：当一个事务正在读取或修改页面时，我们不希望这个页面被意外地从缓冲池中移除，否则会导致数据不一致或操作失败。

引用计数机制：实际上，一个页面可能被多个事务同时访问，所以通常会有一个引用计数。只有当引用计数降为0时(即没有事务再使用它)，才会真正执行Unpin操作。

LRU替换策略：只有那些没有被Pin的页面(即当前没有被任何事务使用的页面)才能参与LRU替换决策。

```
 54 size_t LRUReplacer::Size() {
 55   data_latch_.lock();
 56   size_t ret = data_idx_.size();
 57   data_latch_.unlock();
 58   return ret;
 59 }
```

对于Size，返回哈希表大小即可。

### Task2 : BUFFER POOL MANAGER INSTANCE

在部分中，需要实现缓冲池管理模块，其从DiskManager中获取数据库页面，并在缓冲池强制要求时或驱逐页面时将数据库脏页面写回DiskManager。

```
 30 class BufferPoolManagerInstance : public BufferPoolManager {
...
134   Page *pages_;
135   /** Pointer to the disk manager. */
136   DiskManager *disk_manager_ __attribute__((__unused__));
137   /** Pointer to the log manager. */
138   LogManager *log_manager_ __attribute__((__unused__));
139   /** Page table for keeping track of buffer pool pages. */
140   std::unordered_map<page_id_t, frame_id_t> page_table_;
141   /** Replacer to find unpinned pages for replacement. */
142   Replacer *replacer_;
143   /** List of free pages. */
144   std::list<frame_id_t> free_list_;
145   /** This latch protects shared data structures. We recommend updating this comment to describe     what it protects. */
146   std::mutex latch_;
147 };
```

缓冲池的成员如上所示，其中pages_为缓冲池中的实际容器页面槽位数组，用于存放从磁盘中读入的页面，并供DBMS访问；disk_manager_为磁盘管理器，提供从磁盘读入页面及写入页面的接口；·log_manager_为日志管理器，本实验中不用考虑该组件；page_table_用于保存磁盘页面IDpage_id和槽位IDframe_id_t的映射；raplacer_用于选取所需驱逐的页面；free_list_保存缓冲池中的空闲槽位ID。在这里，区分page_id和frame_id_t是完成本实验的关键。

```
 28 class Page {
 29   // There is book-keeping information inside the page that should only be relevant to the buffer     pool manager.
 30   friend class BufferPoolManagerInstance;
 31 
 32  public:
 33   /** Constructor. Zeros out the page data. */
 34   Page() { ResetMemory(); }
 35 
 36   /** Default destructor. */
 37   ~Page() = default;
 38 
 39   /** @return the actual data contained within this page */
 40   inline auto GetData() -> char * { return data_; }
 41 
 42   /** @return the page id of this page */
 43   inline auto GetPageId() -> page_id_t { return page_id_; }
 44 
 45   /** @return the pin count of this page */
 46   inline auto GetPinCount() -> int { return pin_count_; }
 47 
 48   /** @return true if the page in memory has been modified from the page on disk, false otherwise     */
 49   inline auto IsDirty() -> bool { return is_dirty_; }
...
 77  private:
 78   /** Zeroes out the data that is held within the page. */
 79   inline void ResetMemory() { memset(data_, OFFSET_PAGE_START, PAGE_SIZE); }
 80 
 81   /** The actual data that is stored within a page. */
 82   char data_[PAGE_SIZE]{};
 83   /** The ID of this page. */
 84   page_id_t page_id_ = INVALID_PAGE_ID;
 85   /** The pin count of this page. */
 86   int pin_count_ = 0;
 87   /** True if the page is dirty, i.e. it is different from its corresponding page on disk. */
 88   bool is_dirty_ = false;
 89   /** Page latch. */
 90   ReaderWriterLatch rwlatch_;
 91 };

```

Page是缓冲池中的页面容器，data_保存对应磁盘页面的实际数据；page_id_保存该页面在磁盘管理器中的页面ID；pin_count_保存DBMS中正使用该页面的用户数目；is_dirty_保存该页面自磁盘读入或写回后是否被修改。下面，将介绍缓冲池中的接口实现：

```
 51 bool BufferPoolManagerInstance::FlushPgImp(page_id_t page_id) {
 52   // Make sure you call DiskManager::WritePage!
 53   frame_id_t frame_id;
 54   latch_.lock();
 55   if (page_table_.count(page_id) == 0U) {
 56     latch_.unlock();
 57     return false;
 58   }
 59   frame_id = page_table_[page_id];
 60   pages_[frame_id].is_dirty_ = false;
 61   disk_manager_->WritePage(page_id, pages_[frame_id].GetData());
 62   latch_.unlock();
 63   return true;
 64 }
```

FlushPgImp用于显式地将缓冲池页面写回磁盘。首先，应当检查缓冲池中是否存在对应页面ID的页面，如不存在则返回False；如存在对应页面，则将缓冲池内的该页面的is_dirty_置为false，并使用WritePage将该页面的实际数据data_写回磁盘。在这里，需要使用互斥锁保证线程安全，在下文中不再赘述。

**page_id（磁盘页面ID）：**

- 唯一标识磁盘上的一个物理页面

- 由 DiskManager 分配和管理

- 即使系统重启，相同的 page_id 仍然指向磁盘上相同的物理位置

**frame_id（内存帧ID）：**

- 唯一标识缓冲池（内存）中的一个槽位（slot）

- 是 pages_ 数组的索引（pages_[frame_id] 就是具体的 Page 对象）

- 缓冲池大小固定，frame_id 的范围是 [0, pool_size)（例如缓冲池有100个页面，frame_id 范围就是0~99）

**pages_ (Page数组指针)**

- 这是缓冲池的核心存储区域，指向一个Page对象的数组

- 每个Page对象代表内存中的一个页面(frame)，可以存放从磁盘读取的数据页

- 数组大小等于缓冲池的容量(frame数量)
```
frame_id = page_table_[page_id];  // 通过page_id获取frame_id
pages_[frame_id]...              // 直接使用frame_id作为索引
```

```
 66 void BufferPoolManagerInstance::FlushAllPgsImp() {
 67   // You can do it!
 68   latch_.lock();
 69   for (auto [page_id, frame_id] : page_table_) {
 70     pages_[frame_id].is_dirty_ = false;
 71     disk_manager_->WritePage(page_id, pages_[frame_id].GetData());
 72   }
 73   latch_.unlock();
 74 }
```
FlushAllPgsImp将缓冲池内的所有页面写回磁盘。在这里，遍历page_table_以获得缓冲池内的<页面ID - 槽位ID>对，通过槽位ID获取实际页面，并通过页面ID作为写回磁盘的参数。

```
 76 Page *BufferPoolManagerInstance::NewPgImp(page_id_t *page_id) {
 77   // 0.   Make sure you call AllocatePage!
 78   // 1.   If all the pages in the buffer pool are pinned, return nullptr.
 79   // 2.   Pick a victim page P from either the free list or the replacer. Always pick from the free list first.
 80   // 3.   Update P's metadata, zero out memory and add P to the page table.
 81   // 4.   Set the page ID output parameter. Return a pointer to P.
 82   frame_id_t new_frame_id;
 83   latch_.lock();
 84   if (!free_list_.empty()) {
 85     new_frame_id = free_list_.front();
 86     free_list_.pop_front();
 87   } else if (!replacer_->Victim(&new_frame_id)) {
 88     latch_.unlock();
 89     return nullptr;
 90   }
 91   *page_id = AllocatePage();
 92   if (pages_[new_frame_id].IsDirty()) {
 93     page_id_t flush_page_id = pages_[new_frame_id].page_id_;
 94     pages_[new_frame_id].is_dirty_ = false;
 95     disk_manager_->WritePage(flush_page_id, pages_[new_frame_id].GetData());
 96   }
 97   page_table_.erase(pages_[new_frame_id].page_id_);
 98   page_table_[*page_id] = new_frame_id;
 99   pages_[new_frame_id].page_id_ = *page_id;
100   pages_[new_frame_id].ResetMemory();
101   pages_[new_frame_id].pin_count_ = 1;
102   replacer_->Pin(new_frame_id);
103   latch_.unlock();
104   return &pages_[new_frame_id];
105 }

```

NewPgImp在磁盘中分配新的物理页面，将其添加至缓冲池，并返回指向缓冲池页面Page的指针。在这里，该函数由以下步骤组成：

- 检查当前缓冲池中是否存在空闲槽位或存放页面可被驱逐的槽位（下文称其为目标槽位），在这里总是先通过检查free_list_以查询空闲槽位，如无空闲槽位则尝试从replace_中驱逐页面并返回被驱逐页面的槽位。如目标槽位，则返回空指针；如存在目标槽位，则调用AllocatePage()为新的物理页面分配page_id页面ID。
- 值得注意的是，在这里需要检查目标槽位中的页面是否为脏页面，如是则需将其写回磁盘，并将其脏位设为false；
- 从page_table_中删除目标槽位中的原页面ID的映射，并将新的<页面ID - 槽位ID>映射插入，然后更新槽位中页面的元数据。需要注意的是，在这里由于我们返回了指向该页面的指针，我们需要将该页面的用户数pin_count_置为1，并调用replacer_的Pin。


```
107 Page *BufferPoolManagerInstance::FetchPgImp(page_id_t page_id) {
108   // 1.     Search the page table for the requested page (P).
109   // 1.1    If P exists, pin it and return it immediately.
110   // 1.2    If P does not exist, find a replacement page (R) from either the free list or the replacer.
111   //        Note that pages are always found from the free list first.
112   // 2.     If R is dirty, write it back to the disk.
113   // 3.     Delete R from the page table and insert P.
114   // 4.     Update P's metadata, read in the page content from disk, and then return a pointer to P.
115   frame_id_t frame_id;
116   latch_.lock();
117   if (page_table_.count(page_id) != 0U) {
118     frame_id = page_table_[page_id];
119     pages_[frame_id].pin_count_++;
120     replacer_->Pin(frame_id);
121     latch_.unlock();
122     return &pages_[frame_id];
123   }
124 
125   if (!free_list_.empty()) {
126     frame_id = free_list_.front();
127     free_list_.pop_front();
128     page_table_[page_id] = frame_id;
129     disk_manager_->ReadPage(page_id, pages_[frame_id].data_);
130     pages_[frame_id].pin_count_ = 1;
131     pages_[frame_id].page_id_ = page_id;
132     replacer_->Pin(frame_id);
133     latch_.unlock();
134     return &pages_[frame_id];
135   }
136   if (!replacer_->Victim(&frame_id)) {
137     latch_.unlock();
138     return nullptr;
139   }
140   if (pages_[frame_id].IsDirty()) {
141     page_id_t flush_page_id = pages_[frame_id].page_id_;
142     pages_[frame_id].is_dirty_ = false;
143     disk_manager_->WritePage(flush_page_id, pages_[frame_id].GetData());
144   }
145   page_table_.erase(pages_[frame_id].page_id_);
146   page_table_[page_id] = frame_id;
147   pages_[frame_id].page_id_ = page_id;
148   disk_manager_->ReadPage(page_id, pages_[frame_id].data_);
149   pages_[frame_id].pin_count_ = 1;
150   replacer_->Pin(frame_id);
151   latch_.unlock();
152   return &pages_[frame_id];
153 }
```

FetchPgImp的功能是获取对应页面ID的页面，并返回指向该页面的指针，这个函数实现了缓冲池管理的核心功能之一，即将页面从磁盘加载到内存，同时处理缓冲池满的情况下的页面替换逻辑。其由以下步骤组成：

- 首先，通过检查page_table_以检查缓冲池中是否已经缓冲该页面，如果已经缓冲该页面，则直接返回该页面，并将该页面的用户数pin_count_递增以及调用replacer_的Pin方法；
- 如缓冲池中尚未缓冲该页面，则需寻找当前缓冲池中是否存在空闲槽位或存放页面可被驱逐的槽位（下文称其为目标槽位），该流程与NewPgImp中的对应流程相似，唯一不同的则是传入目标槽位的page_id为函数参数而非由AllocatePage()分配得到。
- **概括：**
   - 查找页面：通过page_table_哈希表检查请求的页面是否已在缓冲池中，如果页面已存在，则获取其帧ID，增加引用计数，将其标记为被固定，然后返回页面指针
   - 如果页面不在缓冲池中，需要加载：首先检查是否有空闲帧（free_list_），如果有空闲帧，从空闲列表中获取一个帧，如果没有空闲帧，则使用替换策略（replacer_）找一个可以替换的帧，如果无法找到可替换的帧（所有页面都被固定），则返回nullptr
   - 准备帧存放新页面：如果选中的帧包含脏页，将其写回磁盘，从页表中移除旧页面ID到帧的映射，添加新页面ID到帧的映射，更新帧中页面的元数据（页面ID、引用计数等）
   - 加载页面内容：从磁盘读取请求的页面内容到选中的帧，将帧标记为被固定，设置引用计数为1
   - 释放锁并返回：释放互斥锁，返回加载好的页面指针


```
155 bool BufferPoolManagerInstance::DeletePgImp(page_id_t page_id) {
156   // 0.   Make sure you call DeallocatePage!
157   // 1.   Search the page table for the requested page (P).
158   // 1.   If P does not exist, return true.
159   // 2.   If P exists, but has a non-zero pin-count, return false. Someone is using the page.
160   // 3.   Otherwise, P can be deleted. Remove P from the page table, reset its metadata and return it to the free list.
161   DeallocatePage(page_id);
162   latch_.lock();
163   if (page_table_.count(page_id) == 0U) {
164     latch_.unlock();
165     return true;
166   }
167   frame_id_t frame_id;
168   frame_id = page_table_[page_id];
169   if (pages_[frame_id].pin_count_ != 0) {
170     latch_.unlock();
171     return false;
172   }
173   if (pages_[frame_id].IsDirty()) {
174     page_id_t flush_page_id = pages_[frame_id].page_id_;
175     pages_[frame_id].is_dirty_ = false;
176     disk_manager_->WritePage(flush_page_id, pages_[frame_id].GetData());
177   }
178   page_table_.erase(page_id);
179   pages_[frame_id].page_id_ = INVALID_PAGE_ID;
180   free_list_.push_back(frame_id);
181   latch_.unlock();
182   return true;
183 }
```

DeletePgImp的功能为从缓冲池中删除对应页面ID的页面，并将其插入空闲链表free_list_，其由以下步骤组成：
- 首先，检查该页面是否存在于缓冲区，如未存在则返回True。然后，检查该页面的用户数pin_count_是否为0，如非0则返回False。在这里，不难看出DeletePgImp的返回值代表的是该页面是否被用户使用，因此在该页面不在缓冲区时也返回True；
- 检查该页面是否为脏，如是则将其写回并将脏位设置为False。然后，在page_table_中删除该页面的映射，并将该槽位中页面的page_id置为INVALID_PAGE_ID。最后，将槽位ID插入空闲链表即可。

```
185 bool BufferPoolManagerInstance::UnpinPgImp(page_id_t page_id, bool is_dirty) {
186   latch_.lock();
187   frame_id_t frame_id;
188   if (page_table_.count(page_id) != 0U) {
189     frame_id = page_table_[page_id];
190     pages_[frame_id].is_dirty_ |= is_dirty;
191     if (pages_[frame_id].pin_count_ <= 0) {
192       latch_.unlock();
193       return false;
194     }
195     // std::cout<<"Unpin : pin_count = "<<pages_[frame_id].pin_count_<<std::endl;
196     if (--pages_[frame_id].pin_count_ == 0) {
197       replacer_->Unpin(frame_id);
198     }
199   }
200   latch_.unlock();
201   return true;
202 }
```

这个UnpinPgImp函数实现了缓冲池管理器中"解除固定页面"的功能。它为提供用户向缓冲池通知页面使用完毕的接口，用户需声明使用完毕页面的页面ID以及使用过程中是否对该页面进行修改。其由以下步骤组成：

- 首先，需检查该页面是否在缓冲池中，如未在缓冲池中则返回True。然后，检查该页面的用户数是否大于0，如不存在用户则返回false；
- 递减该页面的用户数pin_count_，如在递减后该值等于0，则调用replacer_->Unpin以表示该页面可以被驱逐。
- 流程如下：1.获取互斥锁以确保线程安全 ->2.检查请求的页面ID是否在页表中存在 ->3.如果存在，获取对应的帧ID ->4.更新脏状态 ->5.检查并减少引用计数 ->6.如果引用计数降为0，通知替换器该帧可以被替换 ->7.释放锁并返回结果
- pages_[frame_id].is_dirty_ = pages_[frame_id].is_dirty_ | is_dirty;
  - 这是一个非常巧妙的操作，它实现了"一旦脏就永远脏"的逻辑：
  - 如果页面已经是脏的(is_dirty_ = true)，无论传入的is_dirty是什么，结果仍然是true
  - 如果页面不是脏的(is_dirty_ = false)，那么最终结果取决于传入的is_dirty参数

### Task3 : PARALLEL BUFFER POOL MANAGER
上述缓冲池实现的问题在于锁的粒度过大，其在进行任何一项操作时都将整个缓冲池锁住，因此几乎不存在并行性。在这里，并行缓冲池的思想是分配多个独立的缓冲池，并将不同的页面ID映射至各自的缓冲池中，从而减少整体缓冲池的锁粒度，以增加并行性。

```
 25 class ParallelBufferPoolManager : public BufferPoolManager {
...
 93  private:
 94   std::vector<BufferPoolManager *> instances_;
 95   size_t start_idx_{0};
 96   size_t pool_size_;
 97   size_t num_instances_;
 98 };
```

并行缓冲池的成员如上，instances_用于存储多个独立的缓冲池，pool_size_记录各缓冲池的容量，num_instances_为独立缓冲池的个数，start_idx见下文介绍。

```
 18 ParallelBufferPoolManager::ParallelBufferPoolManager(size_t num_instances, size_t pool_size, Disk Manager *disk_manager, LogManager *log_manager)
 19                                                      
 20     : pool_size_(pool_size), num_instances_(num_instances) {
 21   // Allocate and create individual BufferPoolManagerInstances
 22   for (size_t i = 0; i < num_instances; i++) {
 23     BufferPoolManager *tmp = new BufferPoolManagerInstance(pool_size, num_instances, i, disk_manager, log_manager);
 24     instances_.push_back(tmp);
 25   }
 26 }
 27 
 28 // Update constructor to destruct all BufferPoolManagerInstances and deallocate any associated me    mory
 29 ParallelBufferPoolManager::~ParallelBufferPoolManager() {
 30   for (size_t i = 0; i < num_instances_; i++) {
 31     delete (instances_[i]);
 32   }
 33 }
```
在这里，各独立缓冲池在堆区中进行分配，构造函数和析构函数需要完成相应的分配和释放工作。

```
 35 size_t ParallelBufferPoolManager::GetPoolSize() {
 36   // Get size of all BufferPoolManagerInstances
 37   return num_instances_ * pool_size_;
 38 }
 39 
 40 BufferPoolManager *ParallelBufferPoolManager::GetBufferPoolManager(page_id_t page_id) {
 41   // Get BufferPoolManager responsible for handling given page id. You can use this method in your other methods.
 42   return instances_[page_id % num_instances_];
 43 }
```

- 需要注意的是，GetPoolSize应返回全部缓冲池的容量，即独立缓冲池个数乘以缓冲池容量。
- GetBufferPoolManager返回页面ID所对应的独立缓冲池指针，在这里，通过对页面ID取余的方式将页面ID映射至对应的缓冲池。

```
 45 Page *ParallelBufferPoolManager::FetchPgImp(page_id_t page_id) {
 46   // Fetch page for page_id from responsible BufferPoolManagerInstance
 47   BufferPoolManager *instance = GetBufferPoolManager(page_id);
 48   return instance->FetchPage(page_id);
 49 }
 50 
 51 bool ParallelBufferPoolManager::UnpinPgImp(page_id_t page_id, bool is_dirty) {
 52   // Unpin page_id from responsible BufferPoolManagerInstance
 53   BufferPoolManager *instance = GetBufferPoolManager(page_id);
 54   return instance->UnpinPage(page_id, is_dirty);
 55 }
 56 
 57 bool ParallelBufferPoolManager::FlushPgImp(page_id_t page_id) {
 58   // Flush page_id from responsible BufferPoolManagerInstance
 59   BufferPoolManager *instance = GetBufferPoolManager(page_id);
 60   return instance->FlushPage(page_id);
 61 }
...
 82 bool ParallelBufferPoolManager::DeletePgImp(page_id_t page_id) {
 83   // Delete page_id from responsible BufferPoolManagerInstance
 84   BufferPoolManager *instance = GetBufferPoolManager(page_id);
 85   return instance->DeletePage(page_id);
 86 }
 87 
 88 void ParallelBufferPoolManager::FlushAllPgsImp() {
 89   // flush all pages from all BufferPoolManagerInstances
 90   for (size_t i = 0; i < num_instances_; i++) {
 91     instances_[i]->FlushAllPages();
 92   }
 93 }
```

上述函数仅需调用对应独立缓冲池的方法即可。值得注意的是，由于在缓冲池中存放的为缓冲池实现类的基类指针，因此所调用函数的应为缓冲池实现类的基类对应的虚函数。并且，由于ParallelBufferPoolManager和BufferPoolManagerInstance为兄弟关系，因此ParallelBufferPoolManager不能直接调用BufferPoolManagerInstance对应的Imp函数，因此直接在ParallelBufferPoolManager中存放BufferPoolManagerInstance指针也是不可行的。

```
 63 Page *ParallelBufferPoolManager::NewPgImp(page_id_t *page_id) {
 64   // create new page. We will request page allocation in a round robin manner from the underlying
 65   // BufferPoolManagerInstances
 66   // 1.   From a starting index of the BPMIs, call NewPageImpl until either 1) success and return     2) looped around to
 67   // starting index and return nullptr
 68   // 2.   Bump the starting index (mod number of instances) to start search at a different BPMI each time this function
 69   // is called
 70   Page *ret;
 71   for (size_t i = 0; i < num_instances_; i++) {
 72     size_t idx = (start_idx_ + i) % num_instances_;
 73     if ((ret = instances_[idx]->NewPage(page_id)) != nullptr) {
 74       start_idx_ = (*page_id + 1) % num_instances_;
 75       return ret;
 76     }
 77   }
 78   start_idx_++;
 79   return nullptr;
 80 }
```

在这里，为了使得各独立缓冲池的负载均衡，采用轮转方法选取分配物理页面时使用的缓冲池，在这里具体的规则如下：
- 从start_idx_开始遍历各独立缓冲池，如存在调用NewPage成功的页面，则返回该页面并将start_idx指向该页面的下一个页面；
- 如全部缓冲池调用NewPage均失败，则返回空指针，并递增start_idx。
- 轮询策略： 函数实现了一个轮询(round-robin)机制来平均分配页面创建请求，从start_idx_开始，依次尝试每个缓冲池实例，如果循环了一整圈都没有找到可用的实例，则返回nullptr
- 具体执行流程：1.循环遍历所有缓冲池实例；2.计算当前要尝试的实例索引：(start_idx_ + i) % num_instances_；3.调用该实例的NewPage方法尝试创建新页面；4.如果成功创建（返回非空指针），则更新start_idx_并返回页面指针；5.更新start_idx_的方式很特别：(*page_id + 1) % num_instances_，使用新分配的页面ID来影响下一次的起始位置；
- 实现了负载均衡，使得页面分配请求均匀分布到各个底层缓冲池实例，通过轮询策略减少了资源竞争，即使某些实例已满，只要还有实例可用，系统就能继续服务
- start_idx_是并行缓冲池管理器的一个成员变量，在初始化时设定，它跟踪的是下一次应该从哪个实例开始尝试创建新页面，在函数最后start_idx_++是在所有实例都无法创建页面的情况下，简单地移动起始位置到下一个实例，以便下次调用时从不同的实例开始尝试
- start_idx_++： 如果不更新start_idx_，下次调用仍然从相同位置开始，可能会造成某些实例被频繁访问而其他实例相对闲置
通过轮换起始位置，可以更均匀地分配请求，避免"饥饿"情况

**结构**

<img src="https://github.com/user-attachments/assets/30dc7a14-aa59-4882-9e32-054b79b0cc5c" 
     alt="image" 
     style="width:90%; max-width:600px;">

<img src="https://github.com/user-attachments/assets/692bbe79-e24f-4bdc-9dd3-0ae2be41a8a9" 
     alt="image" 
     style="width:50%; max-width:600px;">


---


## Project 2 : EXTENDIBLE HASH INDEX
在本实验中，需要实现一个磁盘备份的可扩展哈希表，用于DBMS中的索引检索。磁盘备份指该哈希表可写入至磁盘中，在系统重启时可以将其重新读取至内存中使用。可扩展哈希表是动态哈希表的一种类型，其特点为桶在充满或清空时可以桶为单位进行桶分裂或合并，尽在特定情况下进行哈希表全表的扩展和收缩，以减小扩展和收缩操作对全表的影响。

大致图像为如下，或可看后边详细解释：
![image](https://github.com/user-attachments/assets/a9360929-c986-4409-a358-90a51f390069)


### 可扩展哈希表实现原理
在进行实验之前，我们应当了解可扩展哈希表的具体实现原理。在这里，其最根本的思想在于通过改变哈希表用于映射至对应桶的哈希键位数来控制哈希表的总大小，该哈希键位数被称为全局深度。下面是全局深度的一个例子：

![image](https://github.com/user-attachments/assets/99b679c1-1bfa-4362-92dd-7e4633f1d175)

上图为通过哈希函数对字符串产生哈希键的一个示例。可见，当哈希键的位数为32位时，不同的哈希键有2^32个，这代表哈希表将拥有上述数目的目录项以将哈希键映射至相应的哈希桶，该数目显然过于庞大了。因此，我们可以仅关注哈希键的低几位（高几位亦可，但使用低位更易实现）以缩小哈希表目录项的个数。例如，当我们仅关注哈希键的后三位时，不同的哈希键为...000至...111共8个，因此我们仅需为哈希表保存8个目录项即可将各低位不同的哈希键映射至对应的哈希表。

![image](https://github.com/user-attachments/assets/e0d58b89-f80a-45ba-a146-4dda24cc6b40)

除了用于控制哈希表大小的全局深度外，每个哈希表目录项均具有一个局部深度，其记录该目录项所对应的哈希桶所关注的哈希键位数。因此，局部深度以桶为单位划分的，某个目录项的局部深度即为该目录项所指的桶的局部深度。例如，如上图可示，当表的全局深度为3，第001个目录项的局部深度为2时，哈希键为...01的所有键均将被映射至该目录项所对应的哈希桶中，即001和101两个目录项。因此，当哈希表的全局深度为i，某目录项的局部深度为j时，指向该目录项所对应的哈希桶的目录项个数为2^(i-j)。

下面，我将使用一个例子来展示可扩展哈希表的桶分裂/合并，表扩展/收缩行为。在说明中，将使用i代表表的全局深度，j代表目录项的局部深度：

![image](https://github.com/user-attachments/assets/f38f0dad-02fe-4ee5-a154-527bbe7ebcb8)

如上图所示，当哈希表刚被创建时，其全局深度为0，即哈希表仅有一个目录项，任何一个哈希键都将被映射到同一个哈希桶。当该哈希桶被充满时，需要进行桶的分裂，在这里，桶分裂的方式有两种，其对应于桶对应目录项的局部深度小于全局深度、桶对应目录项的局部深度等于全局深度两种情况。

当桶对应目录项的局部深度等于全局深度时，指向该桶的目录项仅有一条，因此需要进行表拓展。表拓展后，将表的全局深度加一，并将指向原被分裂桶和新桶的两个目录项置为当前的全局深度，并将原哈希桶的记录重新插入至新哈希桶或原哈希桶。对于其他目录项，表扩展后低i-1位相同的目录项指向同一桶页面，低第i位相反的两个页面互为分裂映像（实验中的自命名词汇）。

注意，可能分裂后的记录仍然均指向同一哈希桶，在这种情况下需要继续扩展哈希表，为了方便讲解，在本章节中不考虑这种特殊情况。因此，当上图中的哈希桶充满时，哈希表将更新至下图所示形式：

![image](https://github.com/user-attachments/assets/9a144928-bd58-48fd-94e2-6d3871b066c2)

在这里，表的全局深度由0变为1、两个目录项的局部深度被置为当前全局深度1。下面，当...0目录项所对应的桶被充满时，由于全局深度和该目录项的局部深度仍然相同，因此仍需进行表扩展：

![image](https://github.com/user-attachments/assets/86158304-d755-4d8d-aa3c-a4ad81e0870d)

下面，当...00目录项所对应的桶充满时，由于全局深度和该目录项的局部深度仍然相同，因此仍需进行表扩展：

![image](https://github.com/user-attachments/assets/2c7070db-b6a1-43a8-9b4b-04b3bee32f41)

此时，当...001目录项所对应的桶充满时，由于该目录项的局部深度j小于全局深度i，因此有2^(i-j)个目录项指向所需分裂的哈希桶，因此不必进行表的拓展，仅需将桶分裂，并将原哈希桶映射的目录项的一半指向原哈希桶，另一半指向新哈希桶，最后将指向原哈希桶和新哈希桶的所有目录项的局部深度加一即可。划分的规则为低j+1位相同的目录项在分裂后仍指向同一个桶，这种分裂规则保证了局部深度的语义，即分裂后桶关注哈希键的低j+1位。

另一个可能的问题是，如何找到与该目录项指向同一哈希桶的其他目录项。在这里，对于全局深度为i，局部深度为j的目录项，与其共同指向同一哈希桶的目录项（下面将其称为兄弟目录项）的低j位相同，且通过以下三个特性可以方便的遍历所有兄弟目录项：

- 兄弟目录项中的最顶端（位表示最小）目录项为低j位不变、其余位为0的目录项；
- 相邻两个目录项的哈希键相差1<<j；
- 兄弟目录项的总数为1<<(i - j)；

上述操作代码实现见下文，分裂后的哈希表如下所示：

![image](https://github.com/user-attachments/assets/93a3f7ae-83ec-4263-91e4-6a559b5fb4d0)

可以看出低j+1 = 2位相同的目录项在分裂后指向同一哈希桶，即以...01和...11为结尾的目录项分别指向两个不同的哈希桶。当一个目录项所指的哈希桶为空时，需要判断其是否可以与其目标目录项所指的哈希桶合并。一个目录项的目标目录项可由其低第j位反转得到，值得注意的是，由于目录项间的局部深度可能不同，因此目标目录项不一定是可逆的。例如，上图中...010目录项的目标目录项为...000，而...000的目标目录项却为...100。目录项及其目标目录项所指的两个哈希桶的合并的条件如下：（1）两哈希桶均为空桶；（2）目录项及其目标目录项的局部深度相同且不为0。此时，若...001和...011目录项所指的两个哈希桶均为空，则可以进行合并（代码实现见下文）：

![image](https://github.com/user-attachments/assets/e5f5427d-abfe-421f-aabe-d6acf31a7c9c)

合并后，需要将指向合并后哈希桶的所有目录项的局部深度减一。此时，若...000和...100所指的哈希桶均为空，则可以进行合并：

![image](https://github.com/user-attachments/assets/f41bcbc9-c85a-40b9-87fe-0837ba496b91)

当哈希桶合并后使得所有目录项的局部深度均小于全局深度时，既可以进行**哈希表的收缩**。在这里可以体现低位可拓展哈希表，即**收缩哈希表**仅需将全局深度减一即可，而不需改变其余任何哈希表的元数据。下图展示了哈希表收缩后的形态：

![image](https://github.com/user-attachments/assets/c567b100-078e-45cb-b23f-1eca0873f5ed)


### Task1 : PAGE LAYOUTS

为了能在磁盘中写入和读取该哈希表，在这里需要实现两个页面类存储哈希表的数据，其使用上实验中的Page页面作为载体，以在磁盘中被写入和读取，具体的实现原理将在下文中介绍：

**HashTableDirectoryPage**

```
 25 /**
 26  *
 27  * Directory Page for extendible hash table.
 28  *
 29  * Directory format (size in byte):
 30  * --------------------------------------------------------------------------------------------
 31  * | LSN (4) | PageId(4) | GlobalDepth(4) | LocalDepths(512) | BucketPageIds(2048) | Free(1524)
 32  * --------------------------------------------------------------------------------------------
 33  */
 34 class HashTableDirectoryPage {
 35  public:
 ...
189  private:
190   page_id_t page_id_;
191   lsn_t lsn_;
192   uint32_t global_depth_{0};
193   uint8_t local_depths_[DIRECTORY_ARRAY_SIZE];
194   page_id_t bucket_page_ids_[DIRECTORY_ARRAY_SIZE];
195 };

```

该页面类作为哈希表的目录页面，保存哈希表中使用的所有元数据，包括该页面的页面ID，日志序列号以及哈希表的全局深度、局部深度及各目录项所指向的桶的页面ID。在本实验中，GetSplitImageIndex和GetLocalHighBit两个与分裂映像相关的概念并未用到，个人认为此概念并不关键。下面将展示一些稍有难度的函数实现：

```
 29 uint32_t HashTableDirectoryPage::GetGlobalDepthMask() { return (1U << global_depth_) - 1; }
...
 47 bool HashTableDirectoryPage::CanShrink() {
 48   uint32_t bucket_num = 1 << global_depth_;
 49   for (uint32_t i = 0; i < bucket_num; i++) {
 50     if (local_depths_[i] == global_depth_) {
 51       return false;
 52     }
 53   }
 54   return true;
 55 }
```
GetGlobalDepthMask通过位运算返回用于计算全局深度低位的掩码；CanShrink()检查当前所有有效目录项的局部深度是否均小于全局深度，以判断是否可以进行表合并，哈希表的收缩上文图表有演示：
- 计算当前目录中的桶数量：bucket_num = 1 << global_depth_（即2的global_depth_次方）
- 遍历所有桶
- 检查每个桶的本地深度（local_depths_[i]）是否等于全局深度（global_depth_）
- 如果有任何一个桶的本地深度等于全局深度，则返回false，表示不能缩小
- 如果所有桶的本地深度都小于全局深度，则返回true，表示可以缩小

This code generates a bitmask where the lowest global_depth_ bits are set to 1, and all higher bits are 0. Here’s a detailed breakdown:

 - (1) 1U << global_depth_
   - 1U is an unsigned integer literal (unsigned int), ensuring safe bit shifting without unexpected sign-bit behavior.
   - << global_depth_ performs a left shift of 1U by global_depth_ bits.
   - Example: If global_depth_ = 3: Binary of 1U: 000...0001 (32 bits). After left shift by 3: 000...1000 (decimal 8, since 1 << 3 = 8).

- (2) - 1
  - Subtracting 1 from the shifted value sets the lowest global_depth_ bits to 1:
  - Continuing the global_depth_ = 3 example: 8 - 1 = 7, binary 000...0111 (lowest 3 bits are 1).

- Purpose of the Mask
Mask Structure: 00...011...1 (leading 0s + global_depth_ 1s).
- Why Use Unsigned 1U?
  - Avoids Undefined Behavior: If 1 (signed) is used, left-shifting by large values (e.g., 31) could trigger undefined behavior (UB). 1U ensures safe shifting.
  - Consistent Wrapping: Unsigned subtraction wraps around predictably (e.g., 0U - 1 = 0xFFFFFFFF), unlike signed integers.


**HashTableBucketPage**
```
 37 template <typename KeyType, typename ValueType, typename KeyComparator>
 38 class HashTableBucketPage {
 39  public:
...
141  private:
142   // For more on BUCKET_ARRAY_SIZE see storage/page/hash_table_page_defs.h
143   char occupied_[(BUCKET_ARRAY_SIZE - 1) / 8 + 1];
144   // 0 if tombstone/brand new (never occupied), 1 otherwise.
145   char readable_[(BUCKET_ARRAY_SIZE - 1) / 8 + 1];
146   // Do not add any members below array_, as they will overlap.
147   MappingType array_[0];
```

用到了bitmap, 该页面类用于存放哈希桶的键值与存储值对，以及桶的槽位状态数据。occupied_数组用于统计桶中的槽是否被使用过，当一个槽被插入键值对时，其对应的位被置为1，事实上，occupied_完全可以被一个size参数替代，但由于测试用例中需要检测对应的occupied值，因此在这里仍保留该数组；readable_数组用于标记桶中的槽是否被占用，当被占用时该值被置为1，否则置为0；array_是C++中一种弹性数组的写法，在这里只需知道它用于存储实际的键值对即可。

在这里，使用char类型存放两个状态数据数组，在实际使用应当按位提取对应的状态位。下面是使用位运算的状态数组读取和设置函数；

- **char occupied_[(BUCKET_ARRAY_SIZE - 1) / 8 + 1]**
  - 作用：标记某个槽位（slot）是否被占用（即使数据已被删除或无效）;
  - 位图(Bitmap)的原理: 在这种实现中，每个槽位的状态只需要1个比特位就可以表示（0或1），为了节省内存，代码使用了位压缩技术 - 每个char(8位)可以存储8个槽位的状态;
  - 实现：
    - 是一个位图（bitmap），每个 char（1字节 = 8位）表示 8 个槽位的占用状态;
    - (BUCKET_ARRAY_SIZE - 1) / 8 + 1 计算需要多少字节才能覆盖所有槽位（向上取整）;
    - 例如，BUCKET_ARRAY_SIZE=10 → 需要 2 字节（16位），其中 10 位有效;
    - 例如，(BUCKET_ARRAY_SIZE - 1) / 8 + 1 这个公式计算需要多少个字节来存储所有槽位的状态，假设BUCKET_ARRAY_SIZE是100: (100-1)/8 = 12.375，向上取整得13，这样用13个字节(13*8=104位)就可以表示所有100个槽位的状态;

- **char readable_[(BUCKET_ARRAY_SIZE - 1) / 8 + 1]**
  - 作用：标记某个槽位的数据是否有效（非墓碑/非未初始化状态）;
    - 0：表示该槽位是墓碑（tombstone）或从未被占用（逻辑删除或未使用）;
    - 1：表示该槽位的数据是有效的（可读取）;
  - 和 occupied_ 的区别：
    - occupied_ 标记物理占用（即使数据被删除也会保持 1）;
    - readable_ 标记逻辑有效性（删除后设为 0）;

- **MappingType array_[0]**
  - 作用：实际存储键值对（KeyType + ValueType）的柔性数组（flexible array）。
  - 特点：
    - array_[0] 是 C/C++ 中的零长度数组技巧，表示数组大小在运行时确定。
    - 实际内存布局中，array_ 会紧跟在 occupied_ 和 readable_ 之后，动态扩展以存储数据。
    - 通过指针算术访问具体槽位（如 array_[i]）。

- **如何检查槽位**
  - 比如要检查第n个槽位是否被占用，代码会:
  - 计算这个槽位对应的是哪个字节: byte_index = n / 8
  - 计算这个槽位对应的是字节中的哪一位: bit_index = n % 8
  - 通过位操作检查: (readable_[byte_index] & (1 << bit_index)) != 0
    - 这种设计的优点是非常节省内存 - 如果有10000个槽位，只需要约1250字节(10000/8)来存储所有槽位的状态，而不是10000字节。

- **内存布局**
- **| occupied_ (bitmap) | readable_ (bitmap) | array_ (flexible K/V pairs) |**
  - 这样设计允许哈希桶在一块连续内存中存储管理信息(occupied_和readable_)和实际的键值对数据(array_)
  - occupied_ 和 readable_ 是紧凑的位图，用于高效管理槽位状态。array_ 是变长部分，实际存储所有键值对。
    - 插入数据：找到一个 occupied_=0 的槽位（或 readable_=0 的墓碑位置）。设置 occupied_=1 和 readable_=1，写入 array_[i]。
    - 删除数据：不实际擦除数据，而是设置 readable_=0（逻辑删除，成为墓碑）。
    - 查询数据：检查 readable_=1 的槽位，比较键值。

```
 87 template <typename KeyType, typename ValueType, typename KeyComparator>
 88 void HASH_TABLE_BUCKET_TYPE::RemoveAt(uint32_t bucket_idx) {
 89   readable_[bucket_idx / 8] &= ~(1 << (7 - (bucket_idx % 8)));
 90 }
 91 
 92 template <typename KeyType, typename ValueType, typename KeyComparator>
 93 bool HASH_TABLE_BUCKET_TYPE::IsOccupied(uint32_t bucket_idx) const {
 94   return (occupied_[bucket_idx / 8] & (1 << (7 - (bucket_idx % 8)))) != 0;
 95 }
 96 
 97 template <typename KeyType, typename ValueType, typename KeyComparator>
 98 void HASH_TABLE_BUCKET_TYPE::SetOccupied(uint32_t bucket_idx) {
 99   occupied_[bucket_idx / 8] |= 1 << (7 - (bucket_idx % 8));
100 }
101 
102 template <typename KeyType, typename ValueType, typename KeyComparator>
103 bool HASH_TABLE_BUCKET_TYPE::IsReadable(uint32_t bucket_idx) const {
104   return (readable_[bucket_idx / 8] & (1 << (7 - (bucket_idx % 8)))) != 0;
105 }
106 
107 template <typename KeyType, typename ValueType, typename KeyComparator>
108 void HASH_TABLE_BUCKET_TYPE::SetReadable(uint32_t bucket_idx) {
109   readable_[bucket_idx / 8] |= 1 << (7 - (bucket_idx % 8));
110 }
```

**RemoveAt（逻辑删除槽位）**
- readable_[bucket_idx / 8] &= ~(1 << (7 - (bucket_idx % 8)));
  - 作用：将指定槽位标记为逻辑删除（墓碑）（readable_位设为 0）。
  - 操作：
    - bucket_idx / 8：找到目标槽位所在的字节索引（每字节管理8个槽位）。
    - bucket_idx % 8：计算槽位在字节中的位偏移（0~7）。
    - 1 << (7 - (bucket_idx % 8))：生成掩码（如 bucket_idx=2 → 00100000）。
    - ~(...)：按位取反（如 00100000 → 11011111）。
    - &=：清除目标位（其他位保持不变）。


**IsOccupied（检查槽位是否被占用）**
- return (occupied_[bucket_idx / 8] & (1 << (7 - (bucket_idx % 8)))) != 0;
- 作用：判断槽位是否被物理占用（即使数据已删除）。
- 操作：
  - 同上计算字节索引和位偏移。
  - occupied_[...] & (1 << ...)：提取目标位，（如 bucket_idx=2 → 00100000）。
  - != 0：若结果为 1，返回 true。

 
**SetOccupied（标记槽位为占用）**
- occupied_[bucket_idx / 8] |= 1 << (7 - (bucket_idx % 8));
- 作用：强制标记槽位为已占用（用于插入新数据）。
- 操作：
  - |=：将目标位置 1（其他位不变）。

 
**IsReadable（检查槽位是否有效）**
- return (readable_[bucket_idx / 8] & (1 << (7 - (bucket_idx % 8)))) != 0;
- 作用：判断槽位是否有效（非墓碑且数据可读）。
- 逻辑：与 IsOccupied 类似，但操作的是 readable_ 位图。


**SetReadable（标记槽位为有效）**
- readable_[bucket_idx / 8] |= 1 << (7 - (bucket_idx % 8));
- 作用：标记槽位为有效（用于插入或恢复数据）。
- 操作：与 SetOccupied 类似。


😺对于对应索引的键值读取直接访问arrat_数组即可：
```
 77 template <typename KeyType, typename ValueType, typename KeyComparator>
 78 KeyType HASH_TABLE_BUCKET_TYPE::KeyAt(uint32_t bucket_idx) const {
 79   return array_[bucket_idx].first;
 80 }
 81 
 82 template <typename KeyType, typename ValueType, typename KeyComparator>
 83 ValueType HASH_TABLE_BUCKET_TYPE::ValueAt(uint32_t bucket_idx) const {
 84   return array_[bucket_idx].second;
 85 }
```

```
 22 template <typename KeyType, typename ValueType, typename KeyComparator>
 23 bool HASH_TABLE_BUCKET_TYPE::GetValue(
    KeyType key,                    // 要查找的键
    KeyComparator cmp,              // 键比较器（用于比较键是否相等）
    std::vector<ValueType> *result  // 输出参数：存储匹配的值
    ) {
 24   bool ret = false;
 25   for (size_t bucket_idx = 0; bucket_idx < BUCKET_ARRAY_SIZE; bucket_idx++) {
 26     if (!IsOccupied(bucket_idx)) {
 27       break;                    //若遇到未占用的槽位，说明后续槽位均为空，直接终止遍历（因为插入时是从低索引到高索引填充的
 28     }
 29     if (IsReadable(bucket_idx) && cmp(key, KeyAt(bucket_idx)) == 0) {

          //IsReadable(bucket_idx)：检查槽位是否有效（非墓碑状态）。
          //KeyAt(bucket_idx)：获取槽位中存储的键（实际是 array_[bucket_idx].first）
          //cmp(key, KeyAt(...)) == 0：用比较器判断键是否相等
          //只有有效且键匹配的槽位会被处理

 30       result->push_back(array_[bucket_idx].second);
 31       ret = true;

          //将匹配的值（array_[bucket_idx].second）存入结果向量 result。
          //设置返回值 ret = true（表示至少找到一个匹配项）。
 32     }
 33   }
 34   return ret;
 35 }
```

GetValue提取桶中键为key的所有值，实现方法为遍历所有occupied_为1的位，并将键匹配的值插入result数组即可，如至少找到了一个对应值，则返回真。

```
 37 template <typename KeyType, typename ValueType, typename KeyComparator>
 38 bool HASH_TABLE_BUCKET_TYPE::Insert(KeyType key, ValueType value, KeyComparator cmp) {
 39   size_t slot_idx = 0;                                 // 记录可用槽位的索引
 40   bool slot_found = false;                             // 是否找到可用槽位

 41   for (size_t bucket_idx = 0; bucket_idx < BUCKET_ARRAY_SIZE; bucket_idx++) {
 42     if (!slot_found && (!IsReadable(bucket_idx) || !IsOccupied(bucket_idx))) {
 43       slot_found = true;
 44       slot_idx = bucket_idx;
 45       // LOG_DEBUG("slot_idx = %ld", bucket_idx);

          //条件：未找到可用槽位 且 当前槽位是墓碑（!IsReadable）或未占用（!IsOccupied）
          //操作：记录第一个可用槽位的索引 slot_idx，并标记 slot_found
 46     }
 47     if (!IsOccupied(bucket_idx)) {
 48       break;          //遇到未占用的槽位（即完全空闲的槽位）时终止遍历（因为插入是从低到高填充的）
 49     }
 50     if (IsReadable(bucket_idx) && cmp(key, KeyAt(bucket_idx)) == 0 && value == ValueAt(bucket_idx)) {   //槽位有效，键匹配，值匹配
 51       return false;   //操作：如果完全相同的键值对已存在，直接返回 false（拒绝重复插入）
 52     }
 53   }

 54   if (slot_found) {
 55     SetReadable(slot_idx);                          // 标记为有效
 56     SetOccupied(slot_idx);                          // 标记为占用
 57     array_[slot_idx] = MappingType(key, value);     // 存储数据
 58     return true;                                    // 表示插入成功
 59   }

 60   return false;                                     // 遍历完所有槽位仍未找到可用位置（桶已满），返回 false
 61 }
```
<img src="https://github.com/user-attachments/assets/d4a8152e-fc10-4075-98dc-b023b282d0a2" 
     alt="image" 
     style="width:50%; max-width:600px;">


Insert向桶插入键值对，其先检测该键值对是否已经被插入到桶中，如是则返回假；如未找到该键值对，则从小到大遍历所有occupied_为1的位，如出现readable_为1的位，则在array_中对应的数组中插入键值对。由于此种插入特性，因此occupied_为1的位是连续的，因此occupied_的功能与一个size参数是等价的。在这里仍然采用occupied_数组的原因可能是提供静态哈希表的实现兼容性（静态哈希表采用线性探测法解决散列冲突，因此必须使用occupied_数组）。

```
 63 template <typename KeyType, typename ValueType, typename KeyComparator>
 64 bool HASH_TABLE_BUCKET_TYPE::Remove(KeyType key, ValueType value, KeyComparator cmp) {
 65   for (size_t bucket_idx = 0; bucket_idx < BUCKET_ARRAY_SIZE; bucket_idx++) {
 66     if (!IsOccupied(bucket_idx)) {
 67       break;
 68     }
 69     if (IsReadable(bucket_idx) && cmp(key, KeyAt(bucket_idx)) == 0 && value == ValueAt(bucket_idx)) {
 70       RemoveAt(bucket_idx);
 71       return true;
 72     }
 73   }
 74   return false;
 75 }
```

Remove从桶中删除对应的键值对，遍历桶所有位即可。

```
112 template <typename KeyType, typename ValueType, typename KeyComparator>
113 bool HASH_TABLE_BUCKET_TYPE::IsFull() {
114   return NumReadable() == BUCKET_ARRAY_SIZE;   //检查桶中有效键值对的数量是否达到容量上限
115 }
116 
117 template <typename KeyType, typename ValueType, typename KeyComparator>
118 uint32_t HASH_TABLE_BUCKET_TYPE::NumReadable() {
119   uint32_t ret = 0;
120   for (size_t bucket_idx = 0; bucket_idx < BUCKET_ARRAY_SIZE; bucket_idx++) {
121     if (!IsOccupied(bucket_idx)) {
122       break;        //遇到第一个 !IsOccupied 的槽位时直接 break（因为插入是顺序填充的，后续槽位必然为空）
123     }
124     if (IsReadable(bucket_idx)) {
125       ret++;        //仅当 IsReadable(bucket_idx) 为 true 时（非墓碑），计数器 ret 增加
126     }
127   }
128   return ret;
129 } 
130     
131 template <typename KeyType, typename ValueType, typename KeyComparator>
132 bool HASH_TABLE_BUCKET_TYPE::IsEmpty() {
133   return NumReadable() == 0;  //若 NumReadable() 返回 0，表示桶中无有效数据（可能全为墓碑或完全空闲），返回 true
134 }
```

NumReadable()返回桶中的键值对个数，遍历即可。IsFull()和IsEmpty()直接复用NumReadable()实现。

**Page与上述两个页面类的转换，页面（Page）与具体页面类型（如HashTableDirectoryPage/HashTableBucketPage）之间的类型转换机制，这是数据库存储引擎设计的核心技巧之一**

在本部分中，有难点且比较巧妙的地方在于理解上述两个页面类是如何与Page类型转换的。在这里，上述两个页面类并非未Page类的子类，在实际应用中通过reinterpret_cast将Page与两个页面类进行转换。在这里我们回顾一下Page的数据成员：

```
 77  private:
 78   /** Zeroes out the data that is held within the page. */
 79   inline void ResetMemory() { memset(data_, OFFSET_PAGE_START, PAGE_SIZE); }
 80 
 81   /** The actual data that is stored within a page. */
 82   char data_[PAGE_SIZE]{};
 83   /** The ID of this page. */
 84   page_id_t page_id_ = INVALID_PAGE_ID;
 85   /** The pin count of this page. */
 86   int pin_count_ = 0;
 87   /** True if the page is dirty, i.e. it is different from its corresponding page on disk. */
 88   bool is_dirty_ = false;
 89   /** Page latch. */
 90   ReaderWriterLatch rwlatch_;
 91 };
```

可以看出，Page中用于存放实际数据的data_数组位于数据成员的第一位，其在栈区固定分配一个页面的大小。因此，在Page与两个页面类强制转换时，通过两个页面类的指针的操作仅能影响到data_中的实际数据，而影响不到其它元数据。并且在内存管理器中始终是进行所占空间更大的通用页面Page的分配（实验中的NewPage），因此页面的容量总是足够的。

- 核心设计思想
  - 统一内存管理：数据库系统使用通用的 Page 类作为所有页面的基础容器，负责内存分配、磁盘I/O和并发控制。
  - 类型擦除：具体的页面类型（如哈希表目录页、桶页）通过强制类型转换复用 Page 的 data_ 空间，实现"一片内存，多种解释"
 
```
// 将Page*转换为具体页面类*
HashTableBucketPage* bucket_page = reinterpret_cast<HashTableBucketPage*>(page->GetData());
```
- GetData() 返回 data_ 的起始地址（即 Page 对象起始地址）。
- 通过 reinterpret_cast 将 char[] 内存重新解释为具体页面类。
- Page 类：负责物理层管理（内存/磁盘、并发控制、脏页标记）。
- 具体页面类：负责逻辑层数据结构（如哈希表的目录/桶逻辑）。
- 避免多重继承带来的开销。
- 所有页面统一分配/释放，减少内存碎片。
- 类型安全: 通过模板和静态断言确保类型转换的安全性： static_assert(offsetof(HashTableBucketPage, array_) == 0); // 必须与data_对齐

**哈希桶的序号（Bucket Index）与Page的物理存储之间的关系确实需要明确**
- 1. 哈希桶序号的本质
  - 哈希桶的序号（即 bucket_idx）是逻辑标识符，它通过以下步骤与物理页面关联：
  - 计算哈希值：对键（Key）哈希得到整数值。
  - 提取低N位：用全局深度（global_depth）决定取哈希值的低几位作为 bucket_idx
  - uint32_t bucket_idx = Hash(key) & GetGlobalDepthMask(); // 例如 mask=0b11（global_depth=2，0b代表二进制）
  - 映射到物理页面：通过目录页（DirectoryPage）将 bucket_idx 转换为实际的 page_id

- 2. 目录页的核心作用
  - 目录页（HashTableDirectoryPage）存储了逻辑桶序号到物理页面的映射表：
  - class HashTableDirectoryPage {

    page_id_t bucket_page_ids_[DIRECTORY_ARRAY_SIZE]; // 每个bucket_idx对应的物理page_id

    uint8_t local_depths_[DIRECTORY_ARRAY_SIZE];      // 每个桶的局部深度

    // ...
};

- 3. 物理页面中的桶数据存储
  - 转换后的 HashTableBucketPage 在 Page::data_ 中的布局如下：

  - class HashTableBucketPage {
    
      char occupied_[];    // 位图，标记槽位是否被占用
    
      char readable_[];    // 位图，标记槽位是否有效
    
      MappingType array_[]; // 实际键值对数组
    
  };

  - array_ 的索引：即桶内的槽位序号（slot_idx），与 bucket_idx 无关。 例如：bucket_idx=3 的物理页面内部可能有多个槽位（slot_idx=0,1,2,...）
 
   

- 4. 完整查询流程示例
 
<img src="https://github.com/user-attachments/assets/2c91aa43-5ba7-48aa-b6ba-7aff3c6723a7" 
     alt="image" 
     style="width:70%; max-width:600px;">

- 5. 分裂与合并时的序号变化
  - 桶分裂（Split）
    - 当桶满时，根据局部深度（local_depth）增加位数：原 bucket_idx=0b10（local_depth=2）→ 分裂为 0b010 和 0b110（local_depth=3）。
    - 目录扩展：全局深度增加时，目录项数量翻倍，但物理页面可能共享（直到实际分裂）。

  - 桶合并（Merge）
    - 当两个兄弟桶的 local_depth 相同且可合并时：
      - 例如 0b010 和 0b110（local_depth=3）→ 合并为 0b10（local_depth=2）。
    - 目录收缩：如果所有桶的 local_depth < global_depth，可以缩减全局深度。
   
- 6. 为什么需要逻辑与物理分离？
  - 动态扩展：目录可以动态增长/收缩，而物理页面独立分配。
  - 空间优化：多个逻辑桶可能指向同一物理页面（分裂初期共享页面）。
  - 并发控制：通过 Page 类的锁（rwlatch_）保护物理页面，与逻辑结构解耦。

- bucket_idx：逻辑序号，由哈希值和全局深度计算得到。
- page_id：物理页面标识，通过目录页映射。
- slot_idx：桶内槽位序号，与 bucket_idx 无关。
- 转换链：
  - Key → bucket_idx → page_id → Page → HashTableBucketPage → slot_idx
- **这种设计实现了逻辑层（可扩展哈希）与物理层（页面存储）的优雅解耦，是数据库系统高效管理的核心机制之一**

### Task 2,3 : HASH TABLE IMPLEMENTATION + CONCURRENCY CONTROL

在这两个部分中，我们需要实现一个线程安全的可扩展哈希表。在对可扩展哈希表的原理清楚后，将其实现并不困难，难点在于如何在降低锁粒度、提高并发性的情况下保证线程安全。下面是哈希表的具体实现：

```
 24 template <typename KeyType, typename ValueType, typename KeyComparator>
 25 HASH_TABLE_TYPE::ExtendibleHashTable(const std::string &name, BufferPoolManager *buffer_pool_manager, const KeyComparator &comparator, HashFunction<KeyType> hash_fn)
 27 : buffer_pool_manager_(buffer_pool_manager), comparator_(comparator), hash_fn_(std::move(hash_fn)) {
      // name：哈希表的名称（字符串）
      // buffer_pool_manager：缓冲池管理器的指针，用于管理页面
      // comparator：键比较器，用于比较键的大小
      // hash_fn：哈希函数，用于计算键的哈希值
      // 将各个初始化，并传入function中
      // 例如，hash_fn_使用std::move被初始化为传入的hash_fn，这表示所有权转移（移动语义）

 28   // LOG_DEBUG("BUCKET_ARRAY_SIZE = %ld", BUCKET_ARRAY_SIZE);
      // 打印桶的大小，但是被注释掉了

 29   HashTableDirectoryPage *dir_page = reinterpret_cast<HashTableDirectoryPage *>(buffer_pool_manager_ -> NewPage(&directory_page_id_));
      // 这两行代码创建了一个新的目录页面：
      // buffer_pool_manager_->NewPage(&directory_page_id_)向缓冲池管理器请求一个新页面，并将页面ID存储在directory_page_id_中
      // reinterpret_cast将返回的通用页面指针转换为HashTableDirectoryPage类型的指针，以便能够调用目录页面特有的方法
 31   dir_page->SetPageId(directory_page_id_);
      // 这行代码将目录页面的ID设置为之前获取的ID，确保页面自身知道自己的ID。

 32   page_id_t new_bucket_id;
 33   buffer_pool_manager_->NewPage(&new_bucket_id);
      // 这两行代码创建了一个新的桶页面，并将其ID存储在new_bucket_id中。

 34   dir_page->SetBucketPageId(0, new_bucket_id);
      // 这行代码将目录的第一个槽位（索引0）指向新创建的桶页面，建立了目录和桶之间的关联。在初始状态下，哈希表只有一个桶。

 35   assert(buffer_pool_manager_->UnpinPage(directory_page_id_, true, nullptr));
 36   assert(buffer_pool_manager_->UnpinPage(new_bucket_id, true, nullptr));
      // 这两行代码解除对目录页面和桶页面的固定：
      // UnpinPage的第一个参数是页面ID
      // 第二个参数true表示页面已被修改（脏页）需要写回磁盘
      // 第三个参数nullptr是一个可选的锁跟踪器
      // assert用于确保UnpinPage调用成功，如果失败会触发断言错误

 37 }
```

- 为什么需要SetPageId(directory_page_id_)：
  - buffer_pool_manager_->NewPage(&directory_page_id_) 确实分配了一个ID并将它存储在directory_page_id_ 变量中，但这只是在缓冲池管理器的角度完成了这个操作。SetPageId(directory_page_id_) 是让 HashTableDirectoryPage 对象自身也知道自己的ID。
  - 这类似于数据库系统中的双向引用：缓冲池管理器知道这个页面的ID是什么，页面对象自身也需要记录这个ID（可能用于内部操作或引用） 


- 目录和桶的关系
  - 在可扩展哈希表中，目录和桶是两种不同的页面类型：
  - 目录页面(Directory Page)：
    - 存储哈希表的元数据
    - 目录记录了每个哈希前缀对应的桶
    - 包含指向实际存储数据的桶页面的指针/ID
    - 管理全局深度和扩展策略，维护哈希表的全局深度，处理桶分裂和目录扩展
    - 提供快速查找：当查找某个键时，通过哈希函数计算出哈希值，使用哈希值的前几位作为目录索引来定位到对应的桶，然后在桶中查找实际的键值对

- 桶页面(Bucket Page)：
  - 实际存储键值对数据
  - 每个桶有本地深度和存储容量

- buffer_pool_manager：
  - buffer_pool_manager 是一个通用的页面缓存管理器，能存储目录和桶，它可以管理不同类型的页面。它不关心页面的具体内容是什么，只负责页面的分配、缓存、读写和释放。在这个系统中：
    - 所有数据库对象（包括目录和桶）都被序列化为页面
    - 每个页面有一个唯一的ID
    - 缓冲池管理器通过ID来管理这些页面
    - 将目录页(directory page)和所有桶页(bucket page)都放入缓冲池(buffer pool)是经过精心设计的架构选择。
        1. **统一内存管理（核心优势）**--- 缓冲池是唯一的权威数据源：数据库系统要求所有磁盘页面必须通过缓冲池访问，这是为了：确保并发控制（通过Pin/Unpin机制），实现可靠的崩溃恢复（通过脏页标记），统一管理内存生命周期（示例：即使目录页很小，也必须通过缓冲池才能保证事务ACID特性）
        2. **性能优化考量** --- 目录页高频访问：可扩展哈希表的目录页是"热点数据"，缓存局部性；
        3. **持久化保证的必要性** --- 目录页是元数据：必须保证其持久化才能， 在崩溃后能正确恢复哈希表结构，确保全局深度(global depth)等关键信息不丢失
        4. **并发控制的实现基础** --- 锁粒度控制：通过缓冲池的PinCount实现隐式锁，避免内存重复加载：缓冲池保证同一页面在内存中只有一份副本
   
 - 三者总结：
   - buffer_pool_manager 就像是内存/硬盘管理系统
   - 目录页面就像是文件系统的目录结构
   - 桶页面就像是实际存储文件内容的数据块
   
在这个构造函数中，代码初始化了一个最小的哈希表结构：一个目录页面和一个初始桶页面，建立了它们之间的引用关系，为后续的扩展操作做好了准备。重复来讲就是为哈希表分配一个目录页面和桶页面，并设置目录页面的page_id成员、将哈希表的首个目录项指向该桶。最后，不要忘记调用UnpinPage向缓冲池告知页面的使用完毕。


```

 54 template <typename KeyType, typename ValueType, typename KeyComparator>
 55 uint32_t HASH_TABLE_TYPE::KeyToDirectoryIndex(KeyType key, HashTableDirectoryPage *dir_page) {
 56   uint32_t hashed_key = Hash(key);
 57   uint32_t mask = dir_page->GetGlobalDepthMask();
 58   return mask & hashed_key;
 59 }
    //功能：将键转换为目录索引,确定键应该映射到目录的哪个位置
    //调用Hash(key)计算键的哈希值 -> 从目录页获取全局深度掩码(GetGlobalDepthMask()) -> 使用掩码与哈希值进行按位与运算，得到目录索引
    //目录页包含多个槽位(slot)，每个槽位指向一个桶页面 -> 多个目录槽位可能指向同一个桶页面（在桶未分裂时）, 这个索引是目录页的索引，不是桶的直接编号
 60 
 61 template <typename KeyType, typename ValueType, typename KeyComparator>
 62 page_id_t HASH_TABLE_TYPE::KeyToPageId(KeyType key, HashTableDirectoryPage *dir_page) {
 63   uint32_t idx = KeyToDirectoryIndex(key, dir_page);
 64   return dir_page->GetBucketPageId(idx);
 65 }
    //功能：通过键找到对应的桶页面ID,将键最终映射到实际存储数据的桶页面（只是获取ID，不实际加载页面）
    //先调用KeyToDirectoryIndex获取目录索引 -> 通过目录页的GetBucketPageId方法获取对应桶的页面ID
 66 
 67 template <typename KeyType, typename ValueType, typename KeyComparator>
 68 HashTableDirectoryPage *HASH_TABLE_TYPE::FetchDirectoryPage() {
 69   return reinterpret_cast<HashTableDirectoryPage *>(buffer_pool_manager_->FetchPage(directory_page_id_));
 70 }
    //功能：获取目录页
    //通过缓冲池管理器获取目录页(FetchPage) -> 将返回的通用页面指针转换为HashTableDirectoryPage*类型
    //特点：会增加页面的引用计数(pin count), 使用后必须调用UnpinPage释放
 71 
 72 template <typename KeyType, typename ValueType, typename KeyComparator>
 73 HASH_TABLE_BUCKET_TYPE *HASH_TABLE_TYPE::FetchBucketPage(page_id_t bucket_page_id) {
 74   return reinterpret_cast<HASH_TABLE_BUCKET_TYPE *>(buffer_pool_manager_->FetchPage(bucket_page_id));
 75 }
 76
    //功能：获取指定ID的桶页面（从缓冲池加载页面）
    //参数：bucket_page_id - 要获取的桶页面ID
    //过程：与FetchDirectoryPage类似，但针对任意桶页面
    //注意：同样需要在使用后调用UnpinPage
```
上面是一些用于提取目录页面、桶页面以及目录页面中的目录项的功能函数：

 - **哈希映射流程：Key → Hash → Directory Index → Bucket Page ID → Bucket Page**
 - 缓冲池管理：所有页面访问都通过缓冲池管理器，确保内存安全
 - 类型安全：使用reinterpret_cast进行页面类型转换
 - 全局深度：目录的全局深度决定了掩码大小，影响索引计算
 - 页面生命周期：获取的页面必须在使用后正确释放(unpin)

**典型使用流程**
```
// 示例使用流程
auto dir_page = FetchDirectoryPage();
uint32_t dir_idx = KeyToDirectoryIndex(key, dir_page); //是否增加pin count	- 否
page_id_t bucket_id = KeyToPageId(key, dir_page);      //是否增加pin count	- 否
auto bucket_page = FetchBucketPage(bucket_id);         //是否增加pin count	- 是

// 对bucket_page进行操作...

buffer_pool_manager_->UnpinPage(bucket_id, true);
buffer_pool_manager_->UnpinPage(dir_page->GetPageId(), false);
```
 - 通过KeyToDirectoryIndex找到键对应的目录槽位
 - 通过KeyToPageId获取桶页面ID
 - 通过FetchBucketPage实际加载桶页面进行操作(桶页面指针)
 - 操作完成后UnpinPage释放桶页面

**遇上段流程比较**
```
先前流程： Key --→ bucket_idx(即 Directory Index) → page_id(即 Bucket Page ID) → Page → HashTableBucketPage → slot_idx
                                             │                   
                                             ↓                 
本流程：   Key → Hash → Directory Index → Bucket Page ID → Bucket Page   ---→   (隐含的桶内查找过程)

提醒：
1. Page → HashTableBucketPage: 将通用页面对象转换为哈希表桶页面对象，代码: HashTableBucketPage* bucket = reinterpret_cast<HashTableBucketPage*>(page->GetData())

2. 在桶内查找(slot_idx): 在桶页面内遍历槽位，寻找匹配的键， 代码: for (slot_idx = 0; slot_idx < BUCKET_SIZE; slot_idx++) {...}                                                     
```


```
 80 template <typename KeyType, typename ValueType, typename KeyComparator>
 81 bool HASH_TABLE_TYPE::GetValue(Transaction *transaction, const KeyType &key, std::vector<ValueType> *result) {
 82   HashTableDirectoryPage *dir_page = FetchDirectoryPage();
      //通过缓冲池获取哈希表目录页，这会增加目录页的pin count，确保页面不会被置换出缓冲池

 83   table_latch_.RLock();
      //获取哈希表级别的读锁（共享锁）
      //防止在查找过程中目录结构被修改（如桶分裂）

 84   page_id_t bucket_page_id = KeyToPageId(key, dir_page);
 85   HASH_TABLE_BUCKET_TYPE *bucket = FetchBucketPage(bucket_page_id);
      //KeyToPageId：根据键计算对应的桶页面ID
      //FetchBucketPage：从缓冲池获取实际的桶页面

 86   Page *p = reinterpret_cast<Page *>(bucket);
 87   p->RLatch();
      //将桶页面转换为通用的Page类型以使用锁机制
      //获取桶页面的读锁，允许并发读取但阻止写入

 88   bool ret = bucket->GetValue(key, comparator_, result);
      //调用桶页面的GetValue方法实际执行查找
      //使用比较器comparator_进行键的比较
      //结果存储在result中，返回值表示是否找到

 89   p->RUnlatch();
      //释放桶页面读锁
 90   table_latch_.RUnlock();
      //释放表级读锁

 91   assert(buffer_pool_manager_->UnpinPage(directory_page_id_, false, nullptr));
 92   assert(buffer_pool_manager_->UnpinPage(bucket_page_id, false, nullptr));
      //缓存管理：每个Fetch对应一个Unpin
      //取消固定(unpin)桶页面（false表示页面未被修改，不是脏页）
      //取消固定(unpin)目录页面
      //使用assert确保Unpin操作成功
 93 
 94   return ret;
 95 }
```
 - 并发控制设计
   - 锁层级：
     - 表级读锁(table_latch_)：保护目录结构
     - 页面级读锁(RLatch)：保护单个桶内容
   - 锁获取顺序：
     - 总是先获取表级锁，再获取页面级锁
     - 避免死锁的关键设计
   - 锁释放顺序：
     - 与获取顺序相反（页面锁→表锁）
   - 表级读锁保护目录结构
   - 页面级读锁保护单个桶
   - 这种设计最大化了并发性 
       
GetValue从哈希表中读取与键匹配的所有值结果，其通过哈希表的读锁保护目录页面，并使用桶的读锁保护桶页面。具体的操作步骤为先读取目录页面，再通过目录页面和哈希键或许对应的桶页面，最后调用桶页面的GetValue获取值结果。在函数返回时注意要UnpinPage所获取的页面。加锁时应当保证锁的获取、释放全局顺序以避免死锁。


```
100 template <typename KeyType, typename ValueType, typename KeyComparator>
101 bool HASH_TABLE_TYPE::Insert(Transaction *transaction, const KeyType &key, const ValueType &value    ) {
//功能：向哈希表插入键值对
//参数：
//transaction：事务对象（虽然未直接使用，但保留接口一致性）
//key：要插入的键
//value：要插入的值
//返回值：bool类型，表示插入是否成功

102   HashTableDirectoryPage *dir_page = FetchDirectoryPage();
103   table_latch_.RLock();
//获取目录页（固定页面）
//获取表级读锁（共享锁），允许并发读取但阻止结构修改

104   page_id_t bucket_page_id = KeyToPageId(key, dir_page);
105   HASH_TABLE_BUCKET_TYPE *bucket = FetchBucketPage(bucket_page_id);
106   Page *p = reinterpret_cast<Page *>(bucket);
107   p->WLatch();
//计算键对应的桶页面ID
//获取桶页面并转换为Page类型以使用锁机制
//获取桶页面的写锁（排他锁），阻止其他读写操作

108   if (bucket->IsFull()) {
109     p->WUnlatch();
110     table_latch_.RUnlock();
111     assert(buffer_pool_manager_->UnpinPage(directory_page_id_, true, nullptr));
112     assert(buffer_pool_manager_->UnpinPage(bucket_page_id, true, nullptr));
113     return SplitInsert(transaction, key, value);
114   }
//桶满处理：
//释放桶写锁和表读锁
//取消固定目录页和桶页（标记为脏页）
//调用SplitInsert处理桶分裂和重新插入

115   bool ret = bucket->Insert(key, value, comparator_);
116   p->WUnlatch();
117   table_latch_.RUnlock();
118   // std::cout<<"find the unfull bucket"<<std::endl;
119   assert(buffer_pool_manager_->UnpinPage(directory_page_id_, true, nullptr));
120   assert(buffer_pool_manager_->UnpinPage(bucket_page_id, true, nullptr));
121   return ret;
//插入流程：
//尝试向桶中插入键值对
//释放桶写锁和表读锁
//取消固定目录页和桶页（标记为脏页）
//返回插入结果

122 }
```

Insert向哈希表插入键值对，这可能会导致桶的分裂和表的扩张，因此需要保证目录页面的读线程安全，一种比较简单的保证线程安全的方法为：在操作目录页面前对目录页面加读锁。但这种加锁方式使得Insert函数阻塞了整个哈希表，这严重影响了哈希表的并发性。可以注意到，表的扩张的发生频率并不高，对目录页面的操作属于读多写少的情况，因此可以使用乐观锁的方法优化并发性能，其在Insert被调用时仅保持读锁，只在需要桶分裂时重新获得读锁。

Insert函数的具体流程为：

1. 获取目录页面和桶页面，在加全局读锁和桶写锁后检查桶是否已满，如已满则释放锁，并调用UnpinPage释放页面，然后调用SplitInsert实现桶分裂和插入；
2. 如当前桶未满，则直接向该桶页面插入键值对，并释放锁和页面即可;

**乐观锁 vs 悲观锁**

这里我是一直使用悲观锁的：

<img src="https://github.com/user-attachments/assets/267edd7a-62de-4492-a45d-2bf72ae2c1d5" 
     alt="image" 
     style="width:90%; max-width:600px;">

<img src="https://github.com/user-attachments/assets/b0b8aea6-f2dd-4447-a66b-fb578a6585ce" 
     alt="image" 
     style="width:90%; max-width:600px;">

**SplitInsert功能**

```
124 template <typename KeyType, typename ValueType, typename KeyComparator>
125 bool HASH_TABLE_TYPE::SplitInsert(Transaction *transaction, const KeyType &key, const ValueType &value) {
126   HashTableDirectoryPage *dir_page = FetchDirectoryPage();
127   table_latch_.WLock();
      //Directory Page: Fetches the directory page that manages the hash structure
      //Write Lock: Acquires exclusive table lock for structural modifications

128   while (true) {
      //while(true) 开始一个无限循环，直到成功插入或返回错误
129     page_id_t bucket_page_id = KeyToPageId(key, dir_page);
130     uint32_t bucket_idx = KeyToDirectoryIndex(key, dir_page);
131     HASH_TABLE_BUCKET_TYPE *bucket = FetchBucketPage(bucket_page_id);

       //KeyToPageId: 计算键应该存储在哪个桶页面，返回页面ID
       //KeyToDirectoryIndex: 计算键对应的目录索引
       //FetchBucketPage: 获取对应的桶页面

       //桶满时的处理逻辑
132     if (bucket->IsFull()) {
133       uint32_t global_depth = dir_page->GetGlobalDepth();
134       uint32_t local_depth = dir_page->GetLocalDepth(bucket_idx);
135       page_id_t new_bucket_id = 0;
136       HASH_TABLE_BUCKET_TYPE *new_bucket =
137           reinterpret_cast<HASH_TABLE_BUCKET_TYPE *>(buffer_pool_manager_->NewPage(&new_bucket_id));
138       assert(new_bucket != nullptr);
        //检查桶是否已满
        //获取目录的全局深度和当前桶的局部深度
        //创建一个新的桶页面用于分裂


//124-138行：首先，获取目录页面并加全局写锁，在添加全局写锁后，其他所有线程均被阻塞了，因此可以放心的操作数据成员。
//不难注意到，在Insert中释放读锁和SplitInsert中释放写锁间存在空隙，其他线程可能在该空隙中被调度，从而改变桶页面或目录页面数据。
//因此，在这里需要重新在目录页面中获取哈希键所对应的桶页面（可能与Insert中判断已满的页面不是同一页面），并检查对应的桶页面是否已满。如桶页面仍然是满的，则分配新桶和提取原桶页面的元数据。在由于桶分裂后仍所需插入的桶仍可能是满的，因此在这这里进行循环以解决该问题。

          //情况1: 局部深度等于全局深度时的处理
139       if (global_depth == local_depth) {
140         // if i == ij, extand the bucket dir, and split the bucket
141         uint32_t bucket_num = 1 << global_depth;
142         for (uint32_t i = 0; i < bucket_num; i++) {
143           dir_page->SetBucketPageId(i + bucket_num, dir_page->GetBucketPageId(i));
144           dir_page->SetLocalDepth(i + bucket_num, dir_page->GetLocalDepth(i));
145         }
           //操作图解：
           //扩容前目录：[A, B, C, D] (global_depth=2, bucket_num=4)
           //扩容后目录：[A, B, C, D, A, B, C, D] (global_depth=3)
           //具体行为：
           //将目录大小加倍（每个原有条目复制一份到新位置）
           //i + bucket_num：新条目的位置（原位置+原目录大小）
           //复制桶指针和局部深度值

146         dir_page->IncrGlobalDepth();
            //global_depth += 1，目录条目数量变为原来的2倍，global_depth从2→3，目录条目从4个变为8个

147         dir_page->SetBucketPageId(bucket_idx + bucket_num, new_bucket_id);
            //bucket_idx：原桶的目录索引，bucket_idx + bucket_num：对应的新位置（镜像位置）
            //原bucket_idx=2 (二进制10)，bucket_num=4
            //新位置=2+4=6 (二进制110)

148         dir_page->IncrLocalDepth(bucket_idx);
149         dir_page->IncrLocalDepth(bucket_idx + bucket_num);
            //操作：将这两个位置的局部深度都+1
            //为什么：原桶和新桶现在都需要更精细的区分，局部深度增加意味着需要多1位哈希位来区分

150         global_depth++;
            //保证后续代码使用正确的global_depth值

            //当局部深度等于全局深度时，意味着目录必须扩容才能继续分裂桶：
            //计算当前目录中的桶数量 bucket_num = 1 << global_depth
            //复制现有目录项到新的扩展部分
            //增加全局深度
            //设置新桶的页面ID和增加相关桶的局部深度
            //在这种情况下，目录大小会翻倍

            //完整示例演示
            假设初始状态：
            //global_depth=2, local_depth=2
            //目录：[A, B, C, D]（4个条目）
            //要分裂的桶：bucket_idx=1 (01)
            操作步骤：
            //扩容目录→[A, B, C, D, A, B, C, D]
            //global_depth→3
            //设置新桶：bucket_idx=1→镜像位置1+4=5
            //dir_page[5] = 新桶
            更新局部深度：
            //dir_page[1].local_depth=3
            //dir_page[5].local_depth=3
            最终目录：
            //[A, B, C, D, A, 新桶, C, D]
            //其中1和5位置的local_depth=3，其余保持2

            //局部深度的意义：数据正确分流，避免无限分裂，目录一致性
            //当 global_depth == local_depth 时：
            分裂前状态：
            //目录有4个条目（global_depth=2）
            //某个桶的local_depth=2（如bucket_idx=01）
            //所有哈希值以01结尾的键都路由到这个桶
            分裂后操作：
            //现在需要3位哈希位来区分这两个桶（而之前只需要2位）

            具体工作原理
            //示例：插入键 K（假设 Hash(K)=101）
            分裂前：
            //取低2位 01 → 路由到bucket_idx=1的桶
            分裂后：
            //取低3位：
            //如果是 101 → 路由到 新桶（目录索引 101=5）
            //如果是 001 → 仍留在 原桶（目录索引 001=1）
            目录结构变化：
            [0] → 桶A (local_depth=2)
            [1] → 桶B (local_depth=3) ← 原桶
            [2] → 桶C (local_depth=2)
            [3] → 桶D (local_depth=2)
            [4] → 桶A (local_depth=2)
            [5] → 桶B_new (local_depth=3) ← 新桶
            [6] → 桶C (local_depth=2)
            [7] → 桶D (local_depth=2)

          //情况2: 局部深度小于全局深度时的处理
151       } else {
152         // if i > ij, split the bucket
153         // more than one records point to the bucket
154         // the records' low ij bits are same
155         // and the high (i - ij) bits are index of the records point to the same bucket
            // 含义：当全局深度 > 局部深度时，说明有多个目录项指向同一个物理桶，现在要拆分这些"共享桶"的指针
156         uint32_t mask = (1 << local_depth) - 1;
            //作用：生成一个低local_depth位全1的掩码

157         uint32_t base_idx = mask & bucket_idx;
            //操作：用掩码保留bucket_idx的低位
            //物理意义：找到这个桶的"逻辑起始位置"
            //bucket_idx=5 (0b101), local_depth=2
            //base_idx=0b101 & 0b11 = 0b01 = 1

158         uint32_t records_num = 1 << (global_depth - local_depth - 1);
            //公式解释：需要处理的目录项数量 = 2^(全局深度-局部深度-1)
            //为什么-1：因为要处理的是"镜像对"的数量
            //global_depth=3, local_depth=1
            //records_num=2^(3-1-1)=2
            //一共多少对指向原来那个桶，然后分一半给新桶，原来1老桶也为一半

159         uint32_t step = (1 << local_depth);
            //物理意义：找到下一个"镜像组"的间隔
            //示例：local_depth=1 → step=2

160         uint32_t idx = base_idx;
161         for (uint32_t i = 0; i < records_num; i++) {
162           dir_page->IncrLocalDepth(idx);
163           idx += step * 2;
164         }
            //初始目录：[A,B,A,B] (global_depth=2, local_depth=1)
            //分裂bucket_idx=1 (base_idx=1): 更新索引1的local_depth (1→2)

165         idx = base_idx + step;
166         for (uint32_t i = 0; i < records_num; i++) {
167           dir_page->SetBucketPageId(idx, new_bucket_id);
168           dir_page->IncrLocalDepth(idx); 
169           idx += step * 2;
              //将部分指针重定向到新桶
              //同样增加这些指针的局部深度
170         }
171       }

             //当局部深度小于全局深度时，不需要扩展目录：
             //计算掩码和基础索引
             //计算需要更新的记录数和步长
             //更新指向原桶的所有目录项的局部深度
             //将一半的目录项指向新创建的桶
             //这种情况下，目录大小不变，只是更新了部分目录项。


             //完整示例演示
             初始状态：
             //global_depth=3, local_depth=1
             //目录：[A,B,A,B,A,B,A,B]（8个条目）
             //要分裂的桶：bucket_idx=3 (0b011)
             执行过程：
             //base_idx=0b011 & 0b1=1
             //records_num=2^(3-1-1)=1
             //step=2
             更新原桶指针：
             //idx=1 → 更新目录[1]的local_depth
             设置新桶指针：
             //idx=1+2=3 → 目录[3]指向新桶，local_depth+1
             最终：
             //目录[1]和[3]的local_depth=2
             //目录[1]指向原桶，[3]指向新桶

             //终极例子：
             原来:
             //[0] → 桶A (local_depth=1)
             //[1] → 桶B (local_depth=1) ← 要分裂这个桶
             //[2] → 桶A (local_depth=1)
             //[3] → 桶B (local_depth=1)
             //[4] → 桶A (local_depth=1)
             //[5] → 桶B (local_depth=1)
             //[6] → 桶A (local_depth=1)
             //[7] → 桶B (local_depth=1)

             //所有指向原桶B的目录项：
             //索引1：001
             //索引3：011
             //索引5：101
             //索引7：111

             应按低2位分组处理：

             组1（低2位=01）：索引1(001)和5(101)
             保持指向原桶B_old
             设置 local_depth=2

             组2（低2位=11）：索引3(011)和7(111)
             重定向到新桶B_new
             设置 local_depth=2

             变换后
             //[0] → 桶A (depth=1)
             //[1] → 桶B_old (depth=2) ← 处理哈希位结尾01
             //[2] → 桶A (depth=1)
             //[3] → 桶B_new (depth=2) ← 处理哈希位结尾11
             //[4] → 桶A (depth=1)
             //[5] → 桶B_old (depth=2) ← 处理哈希位结尾01
             //[6] → 桶A (depth=1)
             //[7] → 桶B_new (depth=2) ← 处理哈希位结尾11

```

139-171行：在这里，需要根据全局深度和桶页面的局部深度判断扩展表和分裂桶的策略。当global_depth == local_depth时，需要进行表扩展和桶分裂，global_depth == local_depth仅需进行桶分裂即可。原理介绍见上文所示：表扩展及分裂桶、仅分裂桶，在这里不再赘述。

```
173       // rehash all records in bucket j
174       for (uint32_t i = 0; i < BUCKET_ARRAY_SIZE; i++) {
            //BUCKET_ARRAY_SIZE：每个桶的固定容量（如128个槽位）
            //作用：检查桶中的每一个可能存在的键值对

175         KeyType j_key = bucket->KeyAt(i);       // 获取第i个位置的键
176         ValueType j_value = bucket->ValueAt(i); // 获取对应的值
177         bucket->RemoveAt(i);                    // 从原桶移除该键值对
            //注意：这里先移除再重新插入，是为了避免重复数据
            //RemoveAt只修改桶的槽位数组，不影响目录页的路由信息
            //仅清除槽位标记（如设置为空位），该槽位变为可复用状态

178         if (KeyToPageId(j_key, dir_page) == bucket_page_id) {
179           bucket->Insert(j_key, j_value, comparator_);
180         } else {
181           new_bucket->Insert(j_key, j_value, comparator_);
182         }
          //KeyToPageId：根据最新的目录状态计算键应属的桶
          //会使用更新后的local_depth进行路由判断
          //分流逻辑：
          //如果键仍属于原桶 → 重新插回原桶; 否则 → 插入新桶

183       }
184       // std::cout<<"original bucket size = "<<bucket->NumReadable()<<std::endl;
185       // std::cout<<"new bucket size = "<<new_bucket->NumReadable()<<std::endl;
186       assert(buffer_pool_manager_->UnpinPage(bucket_page_id, true, nullptr));
187       assert(buffer_pool_manager_->UnpinPage(new_bucket_id, true, nullptr));
          //UnpinPage：释放对物理页面的占用
          //参数true：标记页面已被修改（脏页需要写回磁盘）
          //assert：确保释放操作成功（生产环境应处理错误）

          //这种"先移除再判断"的设计实现了：
          //无临时内存分配的rehash
          //写时复制(Copy-on-Write)的变体
          //最小化数据移动的优化策略
```

173-187行：在完成桶分裂后，应当将原桶页面中的记录重新插入哈希表，由于记录的低i-1位仅与原桶页面和新桶页面对应，因此记录插入的桶页面仅可能为原桶页面和新桶页面两个选择。在重新插入完记录后，释放新桶页面和原桶页面。

```
188     } else {
189       bool ret = bucket->Insert(key, value, comparator_);
          //尝试将键值对插入桶中，ret：返回插入是否成功（可能因重复键失败）

190       table_latch_.WUnlock();
          //释放之前获取的表级写锁（在SplitInsert入口处获取）
          //此时其他线程可以访问哈希表

191       // std::cout<<"find the unfull bucket"<<std::endl;
192       assert(buffer_pool_manager_->UnpinPage(directory_page_id_, true, nullptr));
193       assert(buffer_pool_manager_->UnpinPage(bucket_page_id, true, nullptr));
194       return ret;
          //释放目录页和桶页的占用
          //true：标记页面为脏页（因插入操作修改了内容）
          //assert：确保释放操作成功（生产环境需错误处理）
          //返回插入操作的布尔结果：true：插入成功； false：插入失败（如键已存在）
195     }
196   }
          //当目标桶未满时，直接插入键值对并完成资源释放。
197 
198   return false;
199 }
```

桶最初检测为满，进入SplitInsert -> 在分裂并rehash后再次检查，发现桶现在未满 -> 此时直接插入新键值对，避免不必要的再次分裂

188-198行：若当前键值对所插入的桶页面非空（被其他线程修改或桶分裂后结果），则直接插入键值对，并释放锁和页面，并将插入结果返回Insert。


```
204 template <typename KeyType, typename ValueType, typename KeyComparator>
205 bool HASH_TABLE_TYPE::Remove(Transaction *transaction, const KeyType &key, const ValueType &value) {

206   HashTableDirectoryPage *dir_page = FetchDirectoryPage();          // 获取目录页
207   table_latch_.RLock();                                             // 加表级读锁
208   page_id_t bucket_page_id = KeyToPageId(key, dir_page);            // 计算桶页面ID
209   uint32_t bucket_idx = KeyToDirectoryIndex(key, dir_page);         // 计算目录索引
210   HASH_TABLE_BUCKET_TYPE *bucket = FetchBucketPage(bucket_page_id); // 获取桶
      //并发控制：使用表级读锁保护目录结构
      //双重定位：先找到目录项，再获取物理桶

211   Page *p = reinterpret_cast<Page *>(bucket);
212   p->WLatch();                                                   // 加桶级写锁
213   bool ret = bucket->Remove(key, value, comparator_);            // 执行删除
214   p->WUnlatch();                                                 // 释放桶锁

215   if (bucket->IsEmpty() && dir_page->GetLocalDepth(bucket_idx) != 0) {
216     table_latch_.RUnlock();                  // 先释放读锁
217     this->Merge(transaction, key, value);    // 触发合并
218   } else {
219     table_latch_.RUnlock();                  // 常规释放锁
220   }
221   assert(buffer_pool_manager_->UnpinPage(bucket_page_id, true, nullptr));
222   assert(buffer_pool_manager_->UnpinPage(directory_page_id_, true, nullptr));
      //页面释放：标记桶页和目录页为脏页（true）
      //结果返回：告知调用方是否删除成功
223   return ret;
224 }
```

Remove从哈希表中删除对应的键值对，其优化思想与Insert相同，由于桶的合并并不频繁，因此在删除键值对时仅获取全局读锁，只在需要合并桶时获取全局写锁。当删除后桶为空且目录项的局部深度不为零时，释放读锁并调用Merge尝试合并页面，随后释放锁和页面并返回。

```
229 template <typename KeyType, typename ValueType, typename KeyComparator>
230 void HASH_TABLE_TYPE::Merge(Transaction *transaction, const KeyType &key, const ValueType &value) {

231   HashTableDirectoryPage *dir_page = FetchDirectoryPage();
232   table_latch_.WLock();              // 获取表级写锁
233   uint32_t bucket_idx = KeyToDirectoryIndex(key, dir_page);
234   page_id_t bucket_page_id = dir_page->GetBucketPageId(bucket_idx);
235   HASH_TABLE_BUCKET_TYPE *bucket = FetchBucketPage(bucket_page_id);
      //加写锁：合并需要独占访问目录结构
      //定位目标桶：通过key找到对应的桶

236   if (bucket->IsEmpty() && dir_page->GetLocalDepth(bucket_idx) != 0) {
      //双条件：桶必须为空，不能是初始桶（local_depth>0）

237     uint32_t local_depth = dir_page->GetLocalDepth(bucket_idx);
238     uint32_t global_depth = dir_page->GetGlobalDepth();
239     // How to find the bucket to Merge?
240     // Answer: After Merge, the records, which pointed to the Merged Bucket,
241     // have low (local_depth - 1) bits same
242     // therefore, reverse the low local_depth can get the idx point to the bucket to Merge
243     uint32_t merged_bucket_idx = bucket_idx ^ (1 << (local_depth - 1));
        //位运算技巧：通过异或找到"镜像索引"
        //例如：bucket_idx=01 (local_depth=2) → merged_bucket_idx=11
        //获取兄弟桶：检查是否同样可合并

244     page_id_t merged_page_id = dir_page->GetBucketPageId(merged_bucket_idx);
245     HASH_TABLE_BUCKET_TYPE *merged_bucket = FetchBucketPage(merged_page_id);
        //获取该桶

246     if (dir_page->GetLocalDepth(merged_bucket_idx) == local_depth && merged_bucket->IsEmpty()) {
          //兄弟桶要求：相同local_depth，同样为空
247       local_depth--;
248       uint32_t mask = (1 << local_depth) - 1;
249       uint32_t idx = mask & bucket_idx;
250       uint32_t records_num = 1 << (global_depth - local_depth);
251       uint32_t step = (1 << local_depth);
252 
253       for (uint32_t i = 0; i < records_num; i++) {
254         dir_page->SetBucketPageId(idx, bucket_page_id);
255         dir_page->DecrLocalDepth(idx);
256         idx += step;
            //深度减少：local_depth减1
            //目录更新：将所有相关指针指向合并后的桶
257       }
258       buffer_pool_manager_->DeletePage(merged_page_id);  // 删除空桶
259     }
260     if (dir_page->CanShrink()) {
261       dir_page->DecrGlobalDepth();                       // 可能收缩目录
262     }
263     assert(buffer_pool_manager_->UnpinPage(merged_page_id, true, nullptr));
264   }
265   table_latch_.WUnlock();
      // 释放所有页面和锁
266   assert(buffer_pool_manager_->UnpinPage(directory_page_id_, true, nullptr));
267   assert(buffer_pool_manager_->UnpinPage(bucket_page_id, true, nullptr));
268 }

**合并过程示例**

初始状态：
目录索引  二进制  桶   local_depth
[00]    00   → A     1
[01]    01   → B     2 ← 要合并的空桶
[10]    10   → A     1
[11]    11   → C     2 ← 兄弟桶(空)

合并后：
[00]    00   → A     1
[01]    01   → B     1 ← 深度减少
[10]    10   → A     1
[11]    11   → B     1 ← 指向B且深度减少
删除桶C

不需要检查所有步长桶是否为空，因为可扩展哈希表的合并遵循严格的低位一致性原则。这是由哈希映射的数学特性保证的：

所有同组镜像桶本质是同一个物理桶
当local_depth=2时，索引001/101/011/111指向的物理桶，实际上只有两个：

001和101 → 指向同一个物理桶B
011和111 → 指向同一个物理桶C

检查一个即代表全部
只要确认了bucket_idx和merged_bucket_idx对应的两个物理桶为空，就等同于确认了所有镜像桶都为空。

当然对其他local_depth的也适用的！！
```

当桶变空且满足条件时，将其与"兄弟桶"合并，并可能收缩目录大小。

在Merge函数获取写锁后，需要重新判断是否满足合并条件，以防止在释放锁的空隙时页面被更改，在合并被执行时，需要判断当前目录页面是否可以收缩，如可以搜索在这里仅需递减全局深度即可完成收缩，最后释放页面和写锁。


## Project 3 : QUERY EXECUTION

在关系数据库中，SQL语句将被转换为逻辑查询计划，并在进行查询优化后转化为物理查询计划，系统通过执行物理查询计划完成对应的语句功能。在本实验中，需要为bustub实现物理查询计划执行功能，包括顺序扫描、插入、删除、更改、连接、聚合以及DISTINCT和LIMIT。

### 查询计划执行
<img src="https://github.com/user-attachments/assets/d1998a22-7dc0-4b0d-b0a1-440b887db3b8" 
     alt="image" 
     style="width:90%; max-width:600px;">

在关系型数据库中，物理查询计划在系统内部被组织成树的形式，并通过特定的查询处理模型（迭代器模型、生产者模型）进行执行。在本实验中所要实现的模型为迭代器模型，如上图所示，该模型的每个查询计划节点通过NEXT()方法得到其所需的下一个元组，直至NEXT()方法返回假。在执行流中，根节点的NEXT()方法最先被调用，其控制流向下传播直至叶节点。

谓词在计划中的存储方式
```
1.在 Bustub 中，谓词以表达式树的形式存储在计划节点中：

// 示例：WHERE age > 20 AND name = 'Alice' 的存储结构
       AND
      /   \
     >     =
    / \   / \
 age 20 name 'Alice'

2.Bustub 中常见的计划节点类型：
//plan具体的类型
计划类型	对应SQL	功能
SeqScanPlanNode	SELECT * FROM table	顺序扫描
IndexScanPlanNode	SELECT * FROM table WHERE id=1	索引扫描
InsertPlanNode	INSERT INTO ...	数据插入
AggregationPlanNode	SELECT COUNT(*) ...	聚合计算
NestedLoopJoinPlanNode	SELECT * FROM a,b WHERE a.id=b.id	嵌套循环连接

plan内存结构示意图
plan_ (SeqScanPlanNode对象)
│
├── table_oid_ : 0x1234
├── output_schema_ : Schema对象
├── 插入模式（raw/non-raw)， 如果是raw insert，包含值列表(插入特有)
└── predicate_ : AbstractExpression* (指向表达式树根节点)
       │
       ↓
       AND (LogicExpression)
      /   \
     >     =  (ComparisonExpression)
    / \   / \
 age  20 name 'Alice' (ColumnValue/ConstantValueExpression)
```
在bustub中，每个查询计划节点AbstractPlanNode都被包含在执行器类AbstractExecutor中，用户通过执行器类调用查询计划的Next()方法及初始化Init()方法，而查询计划节点中则保存该操作所需的特有信息，如顺序扫描需要在节点中保存其所要扫描的表标识符、连接需要在节点中保存其子节点及连接的谓词。同时。执行器类中也包含ExecutorContext上下文信息，其代表了查询计划的全局信息，如事务、事务管理器、锁管理器等。

**一些基本概念**
- table_oid_t oid = plan->TableOid() 获取更新计划节点中存储的表对象标识符(OID)，这是表的唯一标识
- auto catalog = exec_ctx->GetCatalog() 从执行上下文中获取数据库目录(Catalog)，目录是存储所有数据库对象元数据的地方
- table_info_ = catalog->GetTable(oid) 使用表的OID从目录中检索该表的完整信息，并将结果存储在 table_info_ 成员变量中
- ExecutorContext *exec_ctx (执行上下文):
  - 包含执行SQL语句所需的环境信息
  - 提供对事务管理器、目录、存储引擎等系统组件的访问
  - 存储事务状态、授权信息等
- const XXXPlanNode *plan (更新计划节点):
  - 由查询优化器生成的更新操作的计划
  - 包含如何执行更新操作的详细信息
  - 存储目标表OID、要更新的列等信息
- std::unique_ptr<AbstractExecutor> &&child_executor (子执行器):
  - 提供要更新的行(tuples)的来源
  - 通常是查询执行计划的另一部分，负责找出满足更新条件的记录
  - 使用右值引用(&&)和unique_ptr实现所有权转移，确保资源管理安全
  
### SeqScanExecutor

SeqScanExecutor执行顺序扫描操作，其通过Next()方法顺序遍历其对应表中的所有元组，并将元组返回至调用者。在bustub中，所有与表有关的信息被包含在TableInfo中：

```
 40 struct TableInfo {
 41   /**
 42    * Construct a new TableInfo instance.
 43    * @param schema The table schema
 44    * @param name The table name
 45    * @param table An owning pointer to the table heap
 46    * @param oid The unique OID for the table
 47    */
 48   TableInfo(Schema schema, std::string name, std::unique_ptr<TableHeap> &&table, table_oid_t oid)
 49       : schema_{std::move(schema)}, name_{std::move(name)}, table_{std::move(table)}, oid_{oid} {    }  
 50   /** The table schema */
 51   Schema schema_;
 52   /** The table name */
 53   const std::string name_;
 54   /** An owning pointer to the table heap */
 55   std::unique_ptr<TableHeap> table_; //通过 unique_ptr 管理的 TableHeap 对象，提供表数据的访问
 56   /** The table OID */
 57   const table_oid_t oid_;            //用于从目录中查找正确的表信息
 58 };

```

表中的实际元组储存在TableHeap中，其包含用于插入、查找、更改、删除元组的所有函数接口，并可以通过TableIterator迭代器顺序遍历其中的元组。在SeqScanExecutor中，为其增加TableInfo、及迭代器私有成员，用于访问表信息和遍历表。在bustub中，所有表都被保存在目录Catalog中，可以通过表标识符从中提取对应的TableInfo：

```
SeqScanExecutor::SeqScanExecutor(ExecutorContext *exec_ctx, const SeqScanPlanNode *plan)
    : AbstractExecutor(exec_ctx),    //执行器上下文，提供执行过程中需要的各种资源
      plan_(plan),                   //顺序扫描计划节点，包含执行计划信息
      iter_(nullptr, RID(INVALID_PAGE_ID, 0), nullptr),
      end_(nullptr, RID(INVALID_PAGE_ID, 0), nullptr) {
      //初始化 iter_ 和 end_为默认构造的迭代器(空指针和无效RID)
      //是一个 构造函数（Constructor），与类名完全相同，无返回类型，冒号 : 后的部分（用于初始化成员变量和基类）
  table_oid_t oid = plan->GetTableOid();
  //从计划节点获取要扫描的表的OID(对象标识符)

  table_info_ = exec_ctx->GetCatalog()->GetTable(oid);
  //通过执行上下文获取目录(catalog)，然后从目录中获取表的信息(TableInfo结构)

  iter_ = table_info_->table_->Begin(exec_ctx->GetTransaction());
  //获取表的起始迭代器，用于开始扫描表数据
  //table_info_->table_ 是 TableHeap 类型的表数据
  //Begin() 返回指向表中第一条记录的迭代器

  end_ = table_info_->table_->End();
  //获取表的结束迭代器，用于判断扫描何时完成
}
```

在Init()中，执行计划节点所需的初始化操作，在这里重新设定表的迭代器，使得查询计划可以重新遍历表：
- 构造函数只在执行器创建时调用一次
- Init() 方法会在每次执行开始前调用（包括第一次执行和可能的重复执行）
- Init()：负责可重复的初始化（如重置迭代器）

```
void SeqScanExecutor::Init() {
  iter_ = table_info_->table_->Begin(exec_ctx_->GetTransaction());
  end_ = table_info_->table_->End();
}
```

在Next()中，计划节点遍历表，并通过输入参数返回元组，当遍历结束时返回假：

```
bool SeqScanExecutor::Next(Tuple *tuple, RID *rid) {
  //返回值：bool 表示是否获取到了下一个有效元组
  //参数：
  //tuple：输出参数，用于返回找到的元组
  //rid：输出参数，用于返回元组的记录ID

  const Schema *out_schema = this->GetOutputSchema();
  Schema table_schema = table_info_->schema_;
  //获取输出模式(out_schema)和表模式(table_schema)

  while (iter_ != end_) {  //遍历表中的所有元组，直到结束迭代器(end_)
    Tuple table_tuple = *iter_;
    *rid = table_tuple.GetRid();
     //解引用迭代器获取当前元组
     //保存当前元组的RID(记录ID)

    std::vector<Value> values;
    for (const auto &col : GetOutputSchema()->GetColumns()) {
      values.emplace_back(col.GetExpr()->Evaluate(&table_tuple, &table_schema));
    }
    *tuple = Tuple(values, out_schema);
    //为输出模式中的每一列计算表达式值
    //用计算出的值构造新的输出元组
    //谓词过滤(WHERE条件)
    //假设查询是：
                SELECT score + 10 AS adjusted_score FROM students;
    //对于 adjusted_score 列：
    //col.GetExpr() 返回 score + 10 这个表达式
    //Evaluate() 会计算当前元组的 score 值并加10
    //结果值被存入 values 向量

    auto *predicate = plan_->GetPredicate();
    if (predicate == nullptr || predicate->Evaluate(tuple, out_schema).GetAs<bool>()) {
      ++iter_;
      return true;
      //如果没有谓词(predicate == nullptr)或谓词评估为真，则：
      //移动迭代器到下一个位置
      //返回true表示找到有效元组
      //在返回当前结果前，先将迭代器移动到下一个位置
      //这样下次调用Next()时，迭代器已经指向正确位置
    }
    ++iter_;
  }
  return false;
}
```

- 什么是谓词(Predicate)?
  - 在数据库系统中，谓词是指返回布尔值(true/false)的条件表达式，用于过滤数据。例如：
    - SELECT * FROM students WHERE age > 20 AND grade = 'A';
    - 这里的 age > 20 AND grade = 'A' 就是一个谓词

auto *predicate = plan_->GetPredicate();
  
if (predicate == nullptr || predicate->Evaluate(tuple, out_schema).GetAs<bool>())
- predicate 就是从查询计划中获取的过滤条件。
- 如果谓词为 nullptr (没有过滤条件)，自动视为"真"
- 否则调用 Evaluate() 计算表达式结果，然后 GetAs<bool>() 转换为布尔值
  - 结果为 true: 元组满足条件，会被返回
  - 结果为 false: 元组被过滤掉，继续检查下一个

**out_schema 与 table_schema 的详细对比**

- table_schema (原始表模式)
  - 来源：table_info_->schema_
  - 内容：表的完整物理结构
  - 示例：
```
// grades 表的 schema
(student_id INT, 
 course_id INT, 
 score DOUBLE,
 teacher VARCHAR(20))
```
- out_schema (输出模式)
  - 来源：查询计划节点的 GetOutputSchema()
  - 内容：查询结果的逻辑结构
  - 示例：
```
-- 对应查询
SELECT student_id, score+10 AS adjusted_score 
FROM grades
cpp
// 输出模式：
(student_id INT,adjusted_score DOUBLE)
```

```
二者关系图解
table_schema (物理存储)           out_schema (逻辑视图)
┌─────────────┬─────────┐        ┌─────────────┬─────────────────┐
│ student_id  │ INT     │        │ student_id  │ INT             │
├─────────────┼─────────┤        ├─────────────┼─────────────────┤
│ course_id   │ INT     │ ──────▶│ adjusted_   │ DOUBLE          │
├─────────────┼─────────┤        │ score       │ (score+10)      │
│ score       │ DOUBLE  │        └─────────────┴─────────────────┘
├─────────────┼─────────┤
│ teacher     │ VARCHAR │
└─────────────┴─────────┘
```
**为什么数据库执行器要逐条获取结果（调用多次Next()），而不是一次性返回所有匹配的元组？**

```
    SeqScan
      |
      v
   Filter(谓词)
      |
      v
   Project(列投影)
      |
      v
   Sort/GroupBy...
```
- 流式处理（Pipeline Processing）的优势
  - 数据库采用这种"懒加载"方式的主要原因包括：
  - 内存效率
    - 降低内存压力：对于百万级的大表，一次性加载所有结果会耗尽内存
    - 支持分页：客户端可以随时停止获取更多结果（如只取前100条）
  - 执行效率
    - 早期过滤：可以在获取第一条结果后就返回，不用等待全部处理完成
    - 并行处理：上层操作（如连接、排序）可以与其他操作流水线并行执行
  - 系统响应性
    - 快速首响应时间：用户能更快看到第一批结果
    - 可中断性：查询可以中途取消而不用完成全部工作

在这里，通过迭代器iter_访问元组，当计划节点谓词predicate非空时，通过predicate的Evaluate方法评估当前元组是否满足谓词，如满足则返回，否则遍历下一个元组。值得注意的是，表中的元组应当以out_schema的模式被重组。在bustub中，所有查询计划节点的输出元组均通过out_schema中各列Column的ColumnValueExpression中的各种“Evaluate”方法构造，如Evaluate、EvaluateJoin、EvaluateAggregate。

对于具有特定out_schema的计划节点，其构造输出元组的方式为遍历out_schema中的Column，并通过Column中ColumnValueExpression的Evaluate方法提取表元组对应的行：

```
// 列值表达式求值，上面的方程中会调用这个；
 36   auto Evaluate(const Tuple *tuple, const Schema *schema) const -> Value override {
 37     return tuple->GetValue(schema, col_idx_);
 38   }

关键设计：
1. 列索引绑定：col_idx_ 保存了该列在原始表中的位置，col_idx_：就像书的目录，告诉你"学生ID"信息存储在元组的哪个位置
2. 统一接口：所有表达式类型（列引用、运算、聚合等）都实现 Evaluate 方法
3. 模式感知：通过 schema 参数正确解析元组数据布局（schema 参数：相当于数据解码手册，说明：每个列的类型（INT/VARCHAR等），列的排列顺序，变长数据的存储位置）

// 当处理 SELECT student_id FROM students 时：
1. SeqScanExecutor::Next() 调用 col.GetExpr()->Evaluate()
2. ColumnValueExpression::Evaluate() 执行:
   - 查找 tuple 中第 col_idx_ 个位置的值
   - 假设 col_idx_ = 0 (student_id 是表的第一列)
3. tuple->GetValue() 根据 schema 信息正确解析二进制数据

```
可以看出，Column中保存了该列在表模式中的列号，Evaluate根据该列号从表元组中提取对应的列。

**全过程演示**
```
SELECT student_id, UPPER(name) AS cap_name 
FROM students 
WHERE age > 20;

处理步骤
1.定位原始数据：

通过 table_schema 知道：
student_id 在第0列（INT）
name 在第1列（VARCHAR）
age 在第2列（INT）

2.谓词评估：
// 检查 age > 20
if (tuple.GetValue(&table_schema, 2).GetAs<int>() > 20) ...

3.构建输出元组：
values = {
    tuple.GetValue(&table_schema, 0),  // student_id
    UpperExpression(tuple.GetValue(&table_schema, 1)) // UPPER(name)
};
return Tuple(values, out_schema);

4.输出验证：
out_schema 确保最终结果只有两列：
第0列：student_id (INT)
第1列：cap_name (VARCHAR)


SQL: SELECT student_id FROM students WHERE age > 20;

执行流程：
1. 扫描获取原始元组 (内存二进制数据)
2. 评估谓词：
   - 创建 ColumnValueExpression(age) → col_idx_=2
   - 调用 Evaluate() → 使用 table_schema 定位age值
3. 构建输出：
   - 创建 ColumnValueExpression(student_id) → col_idx_=0
   - 调用 Evaluate() → 使用 table_schema 提取ID值
4. 用提取的值构建新元组（按out_schema格式）
```

### InsertExecutor

这个构造函数是 Bustub 中插入操作执行器的初始化部分，在InsertExecutor中，其向特定的表中插入元组，元组的来源可能为其他计划节点或自定义的元组数组。其具体来源可通过IsRawInsert()提取。在构造函数中，提取其所要插入表的TableInfo，元组来源，以及与表中的所有索引：

```
InsertExecutor::InsertExecutor(
    ExecutorContext *exec_ctx,     // 执行器上下文
    const InsertPlanNode *plan,    // 插入计划节点
    std::unique_ptr<AbstractExecutor> &&child_executor)  // 子执行器（用于非raw插入）
    : AbstractExecutor(exec_ctx),  // 初始化基类
      plan_(plan),                 // 存储计划节点
      child_executor_(child_executor.release()) { // 接管子执行器所有权
  //获取表信息
  table_oid_t oid = plan->TableOid();  //从计划节点获取目标表的OID
  table_info_ = exec_ctx->GetCatalog()->GetTable(oid);  //通过目录服务获取表的元信息（TableInfo）
  is_raw_ = plan->IsRawInsert();

  //1. plan_ 包含的插入信息
  //目标表OID
  //插入模式（raw/non-raw）
  //如果是raw insert，包含值列表
  //输出模式（可能包含返回的插入计数等）

  //2. table_info_ 包含的表信息
  //表名
  //表结构（Schema）
  //表堆（TableHeap）的指针
  //其他元数据

  //判断是否是"原始值插入"（两种插入方式）：
  //- Raw Insert：直接插入值（如 INSERT INTO t VALUES (1), (2))
  //- 非Raw Insert：从子执行器获取数据（如 INSERT INTO t SELECT ...）

  if (is_raw_) {
    size_ = plan->RawValues().size();
    //RawValues()：获取计划节点中存储的原始值列表
    //size_：记录待插入的元组数量（用于进度跟踪）
  }
  indexes_ = exec_ctx->GetCatalog()->GetTableIndexes(table_info_->name_);
  //GetTableIndexes()：获取该表的所有索引信息，用途：后续插入数据时需要同步更新所有索引
    //3. indexes_ 包含的索引信息
    //索引元数据向量
    //每个索引包含：
    //索引OID
    //索引键模式
    //索引实现（B+树等）的指针
}

```
**场景1：直接值插入**

INSERT INTO students VALUES (1, 'Alice'), (2, 'Bob');

- is_raw_ = true
- size_ = 2
- child_executor_ = nullptr

**场景2：查询结果插入**

INSERT INTO good_students SELECT * FROM students WHERE score > 90;
- is_raw_ = false
- child_executor_ 指向一个查询执行器

在Init中，当其元组来源为其他计划节点时，执行对应计划节点的Init()方法：

```
void InsertExecutor::Init() {
  if (!is_raw_) {
    child_executor_->Init();
  }
}
//避免不必要的资源消耗（如 INSERT INTO ... VALUES 时不会初始化查询执行器）
//直接插入可以免掉（is_raw_ = true）

//对于复杂插入（如 INSERT ... SELECT ... WHERE）：
//InsertExecutor.Init()
//  → SeqScanExecutor.Init()
//      → FilterExecutor.Init()----->这块存疑
```
**child_executor_ 更准确的说法是数据源执行器：**

- 对于 INSERT...SELECT：它是整个 SELECT 查询的执行器根节点，not RAW, 可能包含多级执行器（如含 Filter、Projection 等）
- 对于 INSERT...VALUES：它为 nullptr, 数据直接从 plan_->RawValues() 获取

在Next()中，根据不同的元组来源实施不同的插入策略：

```
bool InsertExecutor::Next([[maybe_unused]] Tuple *tuple, RID *rid) {
  Transaction *txn = exec_ctx_->GetTransaction();
  //作用：获取当前事务上下文
  //用途：保证插入操作的原子性和隔离性
  //关键点：所有表数据和索引的修改都在同一事务中

  Tuple tmp_tuple;
  RID tmp_rid;

  //Raw Insert 处理路径（INSERT VALUES）
  if (is_raw_) {
    for (uint32_t idx = 0; idx < size_; idx++) {
      const std::vector<Value> &raw_value = plan_->RawValuesAt(idx);       // 1. 获取原始值
      tmp_tuple = Tuple(raw_value, &table_info_->schema_);                 // 2. 构造元组
      if (table_info_->table_->InsertTuple(tmp_tuple, &tmp_rid, txn)) {    // 3. 插入表数据
        for (auto indexinfo : indexes_) {                                  // 4. 更新所有索引
          indexinfo->index_->InsertEntry(
              tmp_tuple.KeyFromTuple(table_info_->schema_, indexinfo->key_schema_, indexinfo->index_->GetKeyAttrs()),
              tmp_rid, txn);   // 构造索引键
        }
      }
    }
    return false;
  }

    //值提取：从计划节点获取预定义的原始值列表
    //元组构造：按照表结构(table_info_->schema_)将值打包成元组
    //表插入：调用 TableHeap::InsertTuple 写入表数据
    //索引维护：为每个索引构造键并插入索引项

  //子查询 Insert 处理路径（INSERT SELECT）
  while (child_executor_->Next(&tmp_tuple, &tmp_rid)) {                  //迭代获取元组
    if (table_info_->table_->InsertTuple(tmp_tuple, &tmp_rid, txn)) {    // 1. 插入表数据
      for (auto indexinfo : indexes_) {                                  // 2. 更新所有索引
        indexinfo->index_->InsertEntry(
        tmp_tuple.KeyFromTuple(*child_executor_->GetOutputSchema(), indexinfo->key_schema_, indexinfo->index_->GetKeyAttrs()),

        // 注意使用子执行器的输出模式
        //table_info_->schema_,  ---->  源模式
        //indexinfo->key_schema_,  ---->  索引键模式
        //indexinfo->index_->GetKeyAttrs()  ---->  键包含的属性

        tmp_rid, txn);
      }
    }
  }
  return false;
}
```
***基本思路就是： 处理直接值插入 -> 处理子查询结果插入 -> 插入操作不输出元组***

需要注意，Insert节点不应向外输出任何元组，所以其总是返回false，即所有的插入操作均应当在一次Next中被执行完成。当来源为自定义的元组数组时，根据表模式构造对应的元组，并插入表中；当来源为其他计划节点时，通过子节点获取所有元组并插入表。在插入过程中，应当使用InsertEntry更新表中的所有索引，InsertEntry的参数应由KeyFromTuple方法构造。

**什么是InsertEntry？ 深度解析**

InsertEntry 是 Bustub 索引机制中的核心操作，负责将键值对插入到索引结构中。
  1. 本质功能
   - InsertEntry 是索引结构的成员方法，用于：
      ```
     // 基本调用形式
     index->InsertEntry(index_key, tuple_rid, transaction);
     ```
   - 输入：
     - index_key：索引键（通过 KeyFromTuple 生成）
     - tuple_rid：元组的位置标识（RID）
     - transaction：当前事务上下文
   - 作用：建立索引键到元组物理位置的映射  
     
  2. 键构造过程
  - KeyFromTuple 的工作细节：
```
Tuple key = tuple.KeyFromTuple(
    table_schema,    // 表结构
    index_schema,    // 索引定义的结构
    key_attrs        // 索引包含的列索引
);
```
  - 示例：
    - 表结构：(id INT, name VARCHAR, age INT)
    - 索引定义：CREATE INDEX idx_age ON students(age)
    - 生成的索引键：只包含age列的值 


### UpdateExecutor与DeleteExecutor

UpdateExecutor与DeleteExecutor用于从特定的表中更新、删除元组，其实现方法与InsertExecutor相似，但其元组来源仅为其他计划节点：

```
UpdateExecutor::UpdateExecutor(
    ExecutorContext *exec_ctx,                          // 执行上下文
    const UpdatePlanNode *plan,                         // 更新计划节点
    std::unique_ptr<AbstractExecutor> &&child_executor) // 数据源执行器
    : AbstractExecutor(exec_ctx),
      plan_(plan),
      child_executor_(child_executor.release()) {

  // 获取表元数据
  table_oid_t oid = plan->TableOid();
  auto catalog = exec_ctx->GetCatalog();
  table_info_ = catalog->GetTable(oid);

  // 获取表的所有索引
  indexes_ = catalog->GetTableIndexes(table_info_->name_);
}

void UpdateExecutor::Init() { child_executor_->Init(); }

bool UpdateExecutor::Next([[maybe_unused]] Tuple *tuple, RID *rid) {
  Tuple src_tuple;
  auto *txn = this->GetExecutorContext()->GetTransaction();
  while (child_executor_->Next(&src_tuple, rid)) {
    *tuple = this->GenerateUpdatedTuple(src_tuple);
    if (table_info_->table_->UpdateTuple(*tuple, *rid, txn)) {
      for (auto indexinfo : indexes_) {
        indexinfo->index_->DeleteEntry(tuple->KeyFromTuple(*child_executor_->GetOutputSchema(), indexinfo->key_schema_,
                                                           indexinfo->index_->GetKeyAttrs()),
                                       *rid, txn);
        indexinfo->index_->InsertEntry(tuple->KeyFromTuple(*child_executor_->GetOutputSchema(), indexinfo->key_schema_,
                                                           indexinfo->index_->GetKeyAttrs()),
                                       *rid, txn);
      }
    }
  }
  return false;
}
```
UpdateExecutor::Next中，利用GenerateUpdatedTuple方法将源元组更新为新元组，在更新索引时，删除表中与源元组对应的所有索引记录，并增加与新元组对应的索引记录。

```

DeleteExecutor::DeleteExecutor(ExecutorContext *exec_ctx, const DeletePlanNode *plan,
                               std::unique_ptr<AbstractExecutor> &&child_executor)
    : AbstractExecutor(exec_ctx), plan_(plan), child_executor_(child_executor.release()) {
  table_oid_t oid = plan->TableOid();
  auto catalog = exec_ctx->GetCatalog();
  table_info_ = catalog->GetTable(oid);
  indexes_ = catalog->GetTableIndexes(table_info_->name_);
}

void DeleteExecutor::Init() { child_executor_->Init(); }

bool DeleteExecutor::Next([[maybe_unused]] Tuple *tuple, RID *rid) {
  auto *txn = this->GetExecutorContext()->GetTransaction();
  while (child_executor_->Next(tuple, rid)) {
    if (table_info_->table_->MarkDelete(*rid, txn)) {
      for (auto indexinfo : indexes_) {
        indexinfo->index_->DeleteEntry(tuple->KeyFromTuple(*child_executor_->GetOutputSchema(), indexinfo->key_schema_,
                                                           indexinfo->index_->GetKeyAttrs()),
                                       *rid, txn);
      }
    }
  }
  return false;
}
```

### NestedLoopJoinExecutor
