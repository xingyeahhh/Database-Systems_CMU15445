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
**结构**

<img src="https://github.com/user-attachments/assets/30dc7a14-aa59-4882-9e32-054b79b0cc5c" 
     alt="image" 
     style="width:90%; max-width:600px;">

<img src="https://github.com/user-attachments/assets/692bbe79-e24f-4bdc-9dd3-0ae2be41a8a9" 
     alt="image" 
     style="width:50%; max-width:600px;">


