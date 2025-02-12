# MySQL索引篇——妈妈再也不用担心我不会索引了

>索引问题，在面试中是肯定会出现的，记一道知乎服务端面试题
>
>“如果有这样一个查询 `select * from table where a=1 group by b order by c;` 如果每个字段都有一个单列索引，索引会生效吗？如果是复合索引，能说下几种情况吗？

## 一、回顾索引基础

- MYSQL 官方对索引的定义为：索引（Index）是帮助 MySQL 高效获取数据的数据结构，所以说**索引的本质是：数据结构**

- 索引的目的在于提高查询效率，可以类比字典、 火车站的车次表、图书的目录等 。

- 可以简单的理解为“排好序的快速查找数据结构”，数据本身之外，<font color=#FF0000>**数据库还维护者一个满足特定查找算法的数据结构**</font>，这些数据结构以某种方式引用（指向）数据，这样就可以在这些数据结构上实现高级查找算法。这种数据结构，就是索引。下图是一种可能的索引方式示例。

  ![](https://tva1.sinaimg.cn/large/007S8ZIlly1ghphshtu04j30gt08xmxn.jpg)

  左边的数据表，一共有两列七条记录，最左边的是数据记录的物理地址

  为了加快 Col2 的查找，可以维护一个右边所示的二叉查找树，每个节点分别包含索引键值，和一个指向对应数据记录物理地址的指针，这样就可以运用二叉查找在一定的复杂度内获取到对应的数据，从而快速检索出符合条件的记录。

- 索引本身也很大，不可能全部存储在内存中，**一般以索引文件的形式存储在磁盘上**

- 平常说的索引，没有特别指明的话，就是 B+ 树（多路搜索树，不一定是二叉树）结构组织的索引。其中聚集索引，次要索引，覆盖索引，联合索引，前缀索引，唯一索引默认都是使用 B+ 树索引，统称索引。此外还有哈希索引等。

  

### 优势

- 索引大大减少了服务器需要扫描的数据量（提高数据检索效率）
- 索引可以帮助服务器避免排序和临时表（降低数据排序的成本，降低CPU的消耗）
- 索引可以将随机 I/O 变为顺序 I/O（降低数据库IO成本）



### 劣势

- 索引也是一张表，保存了主键和索引字段，并指向实体表的记录，所以也需要占用内存
- 虽然索引大大提高了查询速度，同时却会降低更新表的速度，如对表进行 INSERT、UPDATE 和 DELETE。
  因为更新表时，MySQL 不仅要保存数据，还要保存一下索引文件每次更新添加了索引列的字段，
  都会调整因为更新所带来的键值变化后的索引信息



### MySQL 索引分类

##### 数据结构角度

- B+树索引
- Hash索引
- Full-Text全文索引
- R-Tree索引

##### 从物理存储角度

- 聚集索引（clustered index）

- 非聚集索引（non-clustered index），也叫辅助索引（secondary index）

  聚集索引和非聚集索引都是 B+ 树结构

##### 从逻辑角度

- 主键索引：主键索引是一种特殊的唯一索引，不允许有空值
- 普通索引或者单列索引：每个索引只包含单个列，一个表可以有多个单列索引
- 多列索引（复合索引、联合索引）：复合索引指多个字段上创建的索引，只有在查询条件中使用了创建索引时的第一个字段，索引才会被使用。使用复合索引时遵循最左前缀集合
- 唯一索引或者非唯一索引
- 空间索引：空间索引是对空间数据类型的字段建立的索引，MYSQL 中的空间数据类型有4种，分别是GEOMETRY、POINT、LINESTRING、POLYGON。 MYSQL 使用 SPATIAL 关键字进行扩展，使得能够用于创建正规索引类型的语法创建空间索引。创建空间索引的列，必须将其声明为 NOT NULL，空间索引只能在存储引擎为 MYISAM 的表中创建



### 基本语法

**创建**：

- 创建索引：

  ```mysql
  CREATE [UNIQUE] INDEX indexName ON mytable(username(length));
  ```

  如果是 CHAR，VARCHAR 类型，length 可以小于字段实际长度；如果是 BLOB 和 TEXT 类型，必须指定 length。

- 修改表结构(添加索引)：

  ```mysql
  ALTER table tableName ADD [UNIQUE] INDEX indexName(columnName);
  ```

**删除**：

```mysql
DROP INDEX [indexName] ON mytable;
```

**查看**：

```mysql
SHOW INDEX FROM table_name\G         --可以通过添加 \G 来格式化输出信息
```

**修改**：

- ```mysql
  ALTER TABLE tbl_name ADD PRIMARY KEY (column_list):
  ```

  该语句添加一个主键，这意味着索引值必须是唯一的，且不能为NULL。

- ```mysql
  ALTER TABLE tbl_name ADD UNIQUE index_name (column_list)
  ```

   这条语句创建索引的值必须是唯一的（除了NULL外，NULL可能会出现多次）。

- ```mysql
  ALTER TABLE tbl_name ADD INDEX index_name (column_list)
  ```

   添加普通索引，索引值可出现多次。

- ```mysql
  ALTER TABLE tbl_name ADD FULLTEXT index_name (column_list)
  ```

  该语句指定了索引为 FULLTEXT ，用于全文索引。





> 为什么 MySQL 索引中用 B+tree，不用 B-tree 或者其他树，为什么不用 Hash 索引
>
> 聚簇索引/非聚簇索引，MySQL 索引底层实现，叶子结点存放的是数据还是指向数据的内存地址，使用索引需要注意的几个地方？
>
> 使用索引查询一定能提高查询的性能吗？为什么?

## 二、MySQL索引结构

介绍索引之前，先了解下[磁盘IO与预读](https://tech.meituan.com/2014/06/30/mysql-index.html "MySQL索引原理及慢查询优化")

> 磁盘读取数据靠的是机械运动，每次读取数据花费的时间可以分为寻道时间、旋转延迟、传输时间三个部分，寻道时间指的是磁臂移动到指定磁道所需要的时间，主流磁盘一般在 5ms 以下；旋转延迟就是我们经常听说的磁盘转速，比如一个磁盘 7200 转，表示每分钟能转 7200 次，也就是说 1 秒钟能转 120 次，旋转延迟就是 `1/120/2 = 4.17ms`；传输时间指的是从磁盘读出或将数据写入磁盘的时间，一般在零点几毫秒，相对于前两个时间可以忽略不计。那么访问一次磁盘的时间，即一次磁盘 IO 的时间约等于 `5+4.17 = 9ms` 左右，听起来还挺不错的，但要知道一台 500 -MIPS的机器每秒可以执行 5 亿条指令，因为指令依靠的是电的性质，换句话说执行一次 IO 的时间可以执行 40 万条指令，数据库动辄十万百万乃至千万级数据，每次 9 毫秒的时间，显然是个灾难。下图是计算机硬件延迟的对比图，供大家参考：![](https://awps-assets.meituan.net/mit-x/blog-images-bundle-2014/7f46a0a4.png)
>
> 考虑到磁盘 IO 是非常高昂的操作，计算机操作系统做了一些优化，当一次 IO 时，不光把当前磁盘地址的数据，而是把相邻的数据也都读取到内存缓冲区内，因为局部预读性原理告诉我们，当计算机访问一个地址的数据的时候，与其相邻的数据也会很快被访问到。每一次 IO 读取的数据我们称之为**一页(page)**。具体一页有多大数据跟操作系统有关，一般为 4k 或 8k，也就是我们读取一页内的数据时候，实际上才发生了一次 IO，这个理论对于索引的数据结构设计非常有帮助。 
>



那是不应该有一种数据结构，可以在每次查找数据时把磁盘 IO 次数控制在一个很小的数量级， B+ 树就这样应用而生。

索引有很多种类型，可以为不同的场景提供更好的性能。

>  **首先要明白索引（index）是在存储引擎（storage engine）层面实现的，而不是 server 层面**。不是所有的存储引擎都支持所有的索引类型。即使多个存储引擎支持某一索引类型，它们的实现和行为也可能有所差别。 



上文中我们知道 MySQL 从数据结构角度分可以分为：B+树索引、Hash索引、Full-Text全文索引、R-Tree索引等，下边就一一扯扯这样索引

### B+Tree索引

MyISAM 和 InnoDB 存储引擎，都使用 B+Tree 的数据结构，它相对与 B-Tree结构，所有的数据都存放在叶子节点上，且把叶子节点通过指针连接到一起，形成了一条数据链表，以加快相邻数据的检索效率。

**先了解下 B-Tree 和 B+Tree 的区别**

#### B- Tree

B-Tree 是为磁盘等外存储设备设计的一种平衡查找树。

系统从磁盘读取数据到内存时是以磁盘块（block）为基本单位的，位于同一个磁盘块中的数据会被一次性读取出来，而不是需要什么取什么。

InnoDB 存储引擎中有页（Page）的概念，页是其磁盘管理的最小单位。InnoDB 存储引擎中默认每个页的大小为16KB，可通过参数 `innodb_page_size` 将页的大小设置为 4K、8K、16K，在 MySQL 中可通过如下命令查看页的大小：`show variables like 'innodb_page_size';`

而系统一个磁盘块的存储空间往往没有这么大，因此 InnoDB 每次申请磁盘空间时都会是若干地址连续磁盘块来达到页的大小 16KB。InnoDB 在把磁盘数据读入到磁盘时会以页为基本单位，在查询数据时如果一个页中的每条数据都能有助于定位数据记录的位置，这将会减少磁盘 I/O 次数，提高查询效率。

B-Tree 结构的数据可以让系统高效的找到数据所在的磁盘块。为了描述 B-Tree，首先定义一条记录为一个二元组[key, data] ，key 为记录的键值，对应表中的主键值，data 为一行记录中除主键外的数据。对于不同的记录，key值互不相同。

一棵 m 阶的 B-Tree 有如下特性： 

1. 每个节点最多有 m 个孩子
2. 除了根节点和叶子节点外，其它每个节点至少有 Ceil(m/2) 个孩子。
3. 若根节点不是叶子节点，则至少有 2 个孩子 
4. **所有叶子节点都在同一层，且不包含其它关键字信息** 
5. 每个非终端节点包含 n 个关键字信息（P0,P1,…Pn, k1,…kn） 
6. 关键字的个数 n 满足：ceil(m/2)-1 <= n <= m-1 
7. ki(i=1,…n) 为关键字，且关键字升序排序
8. Pi(i=1,…n) 为指向子树根节点的指针。P(i-1) 指向的子树的所有节点关键字均小于 ki，但都大于 k(i-1)

B-Tree 中的每个节点根据实际情况可以包含大量的关键字信息和分支，如下图所示为一个 3 阶的 B-Tree：

![来源：网络（各种盗版，不知道哪个是原创了）](https://tva1.sinaimg.cn/large/007S8ZIlly1gg1de1fj9qj30ou08aaas.jpg)

每个节点占用一个盘块的磁盘空间，一个节点上有两个升序排序的关键字和三个指向子树根节点的指针，指针存储的是子节点所在磁盘块的地址。两个关键词划分成的三个范围域对应三个指针指向的子树的数据的范围域。以根节点为例，关键字为 17 和 35，P1 指针指向的子树的数据范围为小于 17，P2 指针指向的子树的数据范围为17~35，P3 指针指向的子树的数据范围为大于 35。

模拟查找关键字 29 的过程：

1. 根据根节点找到磁盘块1，读入内存。【磁盘I/O操作第1次】
2. 比较关键字 29 在区间（17,35），找到磁盘块1的指针P2。
3. 根据 P2 指针找到磁盘块 3，读入内存。【磁盘I/O操作第2次】
4. 比较关键字 29 在区间（26,30），找到磁盘块3的指针P2。
5. 根据 P2 指针找到磁盘块 8，读入内存。【磁盘I/O操作第3次】
6. 在磁盘块 8 中的关键字列表中找到关键字 29。

分析上面过程，发现需要 3 次磁盘 I/O 操作，和 3 次内存查找操作。由于内存中的关键字是一个有序表结构，可以利用二分法查找提高效率。而 3 次磁盘 I/O 操作是影响整个 B-Tree 查找效率的决定因素。

#### B+Tree

B+Tree 是在 B-Tree 基础上的一种优化，使其更适合实现外存储索引结构，InnoDB 存储引擎就是用 B+Tree 实现其索引结构。

从上一节中的 B-Tree 结构图中可以看到每个节点中不仅包含数据的 key 值，还有 data 值。而每一个页的存储空间是有限的，如果 data 数据较大时将会导致每个节点（即一个页）能存储的 key 的数量很小，当存储的数据量很大时同样会导致 B-Tree 的深度较大，增大查询时的磁盘 I/O 次数，进而影响查询效率。在 B+Tree 中，**所有数据记录节点都是按照键值大小顺序存放在同一层的叶子节点上**，而非叶子节点上只存储 key 值信息，这样可以大大加大每个节点存储的 key 值数量，降低 B+Tree 的高度。

B+Tree 相对于 B-Tree 有几点不同：

1. 非叶子节点只存储键值信息；
2. 所有叶子节点之间都有一个链指针；
3. 数据记录都存放在叶子节点中

将上一节中的 B-Tree 优化，由于 B+Tree 的非叶子节点只存储键值信息，假设每个磁盘块能存储 4 个键值及指针信息，则变成 B+Tree 后其结构如下图所示： 
![](https://tva1.sinaimg.cn/large/007S8ZIlly1gf3t57jvq1j30sc0aj0tj.jpg)

通常在 B+Tree 上有两个头指针，一个指向根节点，另一个指向关键字最小的叶子节点，而且所有叶子节点（即数据节点）之间是一种链式环结构。因此可以对 B+Tree 进行两种查找运算：一种是对于主键的范围查找和分页查找，另一种是从根节点开始，进行随机查找。

可能上面例子中只有 22 条数据记录，看不出 B+Tree 的优点，下面做一个推算：

InnoDB 存储引擎中页的大小为 16KB，一般表的主键类型为 INT（占用4个字节）或 BIGINT（占用8个字节），指针类型也一般为 4 或 8 个字节，也就是说一个页（B+Tree中的一个节点）中大概存储 `16KB/(8B+8B)=1K` 个键值（因为是估值，为方便计算，这里的 K 取值为 $10^3$）。也就是说一个深度为 3 的 B+Tree 索引可以维护 `10^3 * 10^3 * 10^3 = 10亿` 条记录。

实际情况中每个节点可能不能填充满，因此在数据库中，B+Tree 的高度一般都在 2-4 层。MySQL 的 InnoDB 存储引擎在设计时是将根节点常驻内存的，也就是说查找某一键值的行记录时最多只需要 1~3 次磁盘 I/O 操作。

**B+Tree性质**

1. 通过上面的分析，我们知道 IO 次数取决于 B+ 数的高度 h，假设当前数据表的数据为 N，每个磁盘块的数据项的数量是 m，则有 `h=㏒(m+1)N`，当数据量 N 一定的情况下，m 越大，h 越小；而 `m = 磁盘块的大小 / 数据项的大小`，磁盘块的大小也就是一个数据页的大小，是固定的，如果数据项占的空间越小，数据项的数量越多，树的高度越低。这就是为什么每个数据项，即索引字段要尽量的小，比如 int 占4字节，要比 bigint 8 字节少一半。这也是为什么 B+ 树要求把真实的数据放到叶子节点而不是内层节点，一旦放到内层节点，磁盘块的数据项会大幅度下降，导致树增高。当数据项等于 1 时将会退化成线性表。
2. 当 B+ 树的数据项是复合的数据结构，比如(name,age,sex)的时候，b+ 数是按照从左到右的顺序来建立搜索树的，比如当(张三,20,F)这样的数据来检索的时候，B+ 树会优先比较 name 来确定下一步的所搜方向，如果name 相同再依次比较 age 和 sex，最后得到检索的数据；但当 (20,F) 这样的没有 name 的数据来的时候，B+ 树就不知道下一步该查哪个节点，因为建立搜索树的时候 name 就是第一个比较因子，必须要先根据name 来搜索才能知道下一步去哪里查询。比如当 (张三,F) 这样的数据来检索时，B+ 树可以用 name 来指定搜索方向，但下一个字段 age 的缺失，所以只能把名字等于张三的数据都找到，然后再匹配性别是 F 的数据了， 这个是非常重要的性质，即**索引的最左匹配特性**。

### Hash索引

- 主要就是通过 Hash 算法（常见的 Hash 算法有直接定址法、平方取中法、折叠法、除数取余法、随机数法），将数据库字段数据转换成定长的 Hash 值，与这条数据的行指针一并存入 Hash 表的对应位置；如果发生 Hash 碰撞，则在对应 Hash 键下以链表形式存储。

  检索算法：在检索查询时，就再次对待查关键字再次执行相同的 Hash 算法，得到 Hash 值，到对应 Hash 表对应位置取出数据即可，如果发生 Hash 碰撞，则需要在取值时进行筛选。使用 Hash 索引的数据库并不多， 目前有 Memory 引擎和 NDB 引擎支持 Hash 索引。

- Hash 索引的弊端

  一般来说，索引的检索效率非常高，可以一次定位，不像 B-Tree 索引需要进行从根节点到叶节点的多次 IO 操作。有利必有弊，Hash 算法在索引的应用也有很多弊端。

  1. Hash 索引只支持等值比较查询，包括 =、IN() 等，不支持任何范围查询，如 where price > 100。因为数据在经过 Hash 算法后，其大小关系就可能发生变化。
  2. Hash 索引不能被排序。同样是因为数据经过 Hash 算法后，大小关系就可能发生变化，排序是没有意义的。
  3. Hash 索引不能避免表数据的扫描。因为发生 Hash 碰撞时，仅仅比较 Hash 值是不够的，需要比较实际的值以判定是否符合要求。
  4. Hash 索引在发生大量 Hash 值相同的情况时性能不一定比 B-Tree 索引高。因为碰撞情况会导致多次的表数据的扫描，造成整体性能的低下，可以通过采用合适的 Hash 算法一定程度解决这个问题。
  5. Hash 索引不能使用部分索引键查询。因为当使用组合索引情况时，是把多个数据库列数据合并后再计算Hash 值，所以对单独列数据计算 Hash 值是没有意义的。

### full-text全文索引

-  全文索引也是 MyISAM 的一种特殊索引类型，主要用于全文索引，InnoDB 从 MYSQL5.6 版本提供对全文索引的支持。 

-  它用于替代效率较低的 LIKE 模糊匹配操作，而且可以通过多字段组合的全文索引一次性全模糊匹配多个字段。 
-  同样使用 B-Tree 存放索引数据，但使用的是特定的算法，将字段数据分割后再进行索引（一般每4个字节一次分割），索引文件存储的是分割前的索引字符串集合，与分割后的索引信息，对应 BTree 结构的节点存储的是分割后的词信息以及它在分割前的索引字符串集合中的位置。 

### R-Tree空间索引

空间索引是 MyISAM 的一种特殊索引类型，主要用于地理空间数据类型 。



> 为什么 MySQL 索引要用 B+ 树不是 B 树？

用 B+ 树不用 B 树考虑的是 IO 对性能的影响，B 树的每个节点都存储数据，而B+树只有叶子节点才存储数据，所以查找相同数据量的情况下，B树的高度更高，IO更频繁。数据库索引是存储在磁盘上的，当数据量大时，就不能把整个索引全部加载到内存了，只能逐一加载每一个磁盘页（对应索引树的节点）。其中在MySQL底层对B+树进行进一步优化：在叶子节点中是双向链表，且在链表的头结点和尾节点也是循环指向的。



> 面试官：为何不采用Hash方式？

因为Hash索引底层是哈希表，哈希表是一种以key-value存储数据的结构，所以多个数据在存储关系上是完全没有任何顺序关系的，所以，对于区间查询是无法直接通过索引查询的，就需要全表扫描。所以，哈希索引只适用于等值查询的场景。而B+ Tree是一种多路平衡查询树，所以他的节点是天然有序的（左子节点小于父节点、父节点小于右子节点），所以对于范围查询的时候不需要做全表扫描。

哈希索引不支持多列联合索引的最左匹配规则，如果有大量重复键值得情况下，哈希索引的效率会很低，因为存在哈希碰撞问题。



## 三、MyISAM 和 InnoDB 索引原理

### MyISAM 主键索引与辅助索引的结构

MyISAM 引擎的索引文件和数据文件是分离的。**MyISAM 引擎索引结构的叶子节点的数据域，存放的并不是实际的数据记录，而是数据记录的地址**。索引文件与数据文件分离，这样的索引称为"**非聚簇索引**"。MyISAM 的主索引与辅助索引区别并不大，只是主键索引不能有重复的关键字。

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gewoy5bddkj31bp0u04lv.jpg)

在 MyISAM 中，索引（含叶子节点）存放在单独的 `.myi` 文件中，叶子节点存放的是数据的物理地址偏移量（通过偏移量访问就是随机访问，速度很快）。

主索引是指主键索引，键值不可能重复；辅助索引则是普通索引，键值可能重复。

通过索引查找数据的流程：先从索引文件中查找到索引节点，从中拿到数据的文件指针，再到数据文件中通过文件指针定位了具体的数据。辅助索引类似。

### InnoDB主键索引与辅助索引的结构

**InnoDB 引擎索引结构的叶子节点的数据域，存放的就是实际的数据记录**（对于主索引，此处会存放表中所有的数据记录；对于辅助索引此处会引用主键，检索的时候通过主键到主键索引中找到对应数据行），或者说，**InnoDB的数据文件本身就是主键索引文件**，这样的索引被称为"“**聚簇索引**”，一个表只能有一个聚簇索引。

#### 主键索引：

我们知道 InnoDB 索引是聚集索引，它的索引和数据是存入同一个 `.idb` 文件中的，因此它的索引结构是在同一个树节点中同时存放索引和数据，如下图中最底层的叶子节点有三行数据，对应于数据表中的 id、stu_id、name数据项。

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gewoy2lhr5j320d0u016k.jpg)

在 Innodb 中，索引分叶子节点和非叶子节点，非叶子节点就像新华字典的目录，单独存放在索引段中，叶子节点则是顺序排列的，在数据段中。InnoDB 的数据文件可以按照表来切分（只需要开启`innodb_file_per_table)`，切分后存放在`xxx.ibd`中，默认不切分，存放在 `xxx.ibdata`中。

#### 辅助（非主键）索引：

这次我们以示例中学生表中的 name 列建立辅助索引，它的索引结构跟主键索引的结构有很大差别，在最底层的叶子结点有两行数据，第一行的字符串是辅助索引，按照 ASCII 码进行排序，第二行的整数是主键的值。

这就意味着，对 name 列进行条件搜索，需要两个步骤：

1. 在辅助索引上检索 name，到达其叶子节点获取对应的主键；
2. 使用主键在主索引上再进行对应的检索操作

这也就是所谓的“**回表查询**”

![](https://tva1.sinaimg.cn/large/007S8ZIlly1gewsc7l623j320r0u0gwt.jpg)

**InnoDB 索引结构需要注意的点**

1. 数据文件本身就是索引文件
2. 表数据文件本身就是按 B+Tree 组织的一个索引结构文件
3. 聚集索引中叶节点包含了完整的数据记录
4. InnoDB 表必须要有主键，并且推荐使用整型自增主键

正如我们上面介绍 InnoDB 存储结构，索引与数据是共同存储的，不管是主键索引还是辅助索引，在查找时都是通过先查找到索引节点才能拿到相对应的数据，如果我们在设计表结构时没有显式指定索引列的话，MySQL 会从表中选择数据不重复的列建立索引，如果没有符合的列，则 MySQL 自动为 InnoDB 表生成一个隐含字段作为主键，并且这个字段长度为 6 个字节，类型为整型。



## 三、索引策略

### 哪些情况需要创建索引

1. 主键自动建立唯一索引

2. 频繁作为查询条件的字段

3. 查询中与其他表关联的字段，外键关系建立索引

4. 单键/组合索引的选择问题，who?高并发下倾向创建组合索引

5. 查询中排序的字段，排序字段通过索引访问大幅提高排序速度

6. 查询中统计或分组字段

   

### 哪些情况不要创建索引

1. 表记录太少
2. 经常增删改的表
3. 数据重复且分布均匀的表字段，只应该为最经常查询和最经常排序的数据列建立索引（如果某个数据类包含太多的重复数据，建立索引没有太大意义）
4. 频繁更新的字段不适合创建索引（会加重IO负担）
5. where 条件里用不到的字段不创建索引



### [高效索引](高性能的索引策略 "《高性能MySQL》")

#### 独立的列

**如果查询中的列不是独立的，MySQL 就不会使用索引**。“独立的列”是指索引不能是表达式的一部分，也不能是函数的参数。

比如：

```mysql
EXPLAIN SELECT * FROM user_info where id = 2;
```

在 user_info 表中，id 是主键，有主键索引，索引 exmplain 出来结果就是：

![](https://static01.imgkr.com/temp/035c42a7c91b48499029dd1f3c5641cb.png)

可见这次查询使用了PRIMARY KEY来优化查询，如果变成这样：

```mysql
EXPLAIN SELECT * FROM user_info where id + 1 = 2;
```

结果就是：

![](https://static01.imgkr.com/temp/4e6c1f2fe25646bb98d799a2d2d2e52c.png)



#### 前缀索引

前缀索引说白了就是对文本的前几个字符（具体是几个字符在建立索引时指定）建立索引，这样建立起来的索引更小，所以查询更快。

```mysql
ALTER TABLE table_name ADD KEY(column_name(prefix_length));
```

对于内容很长的列，比如 blob, text 或者很长的 varchar 列，必须使用前缀索引，MySQL 不允许索引这些列的完整长度。

所以问题就在于要选择合适长度的前缀，即 prefix_length。前缀太短，选择性太低，前缀太长，索引占用空间太大。

为了决定前缀的合适长度，需要找到最常见的值得列表，然后和最常见的前缀列进行比较。

前缀索引是一种能使索引更小、更快的有效办法，但另一方面也有缺点：MySQL 无法使用前缀索引做 ORDER BY 和 GROUP BY，也无法使用前缀索引做覆盖扫描。

> 一个常见的场景是针对很长的十六进制唯一 ID 使用前缀索引。



#### 多列索引（复合索引、联合索引）

**在多个列上建立独立的单列索引大部分情况下并不能提高 MySQL 的查询性能**。对于下面的查询 where 条件，这两个单列索引都是不好的选择：

```mysql
select film_id, actor_id from table1 where actor_id=1 or film_id=1;
```

MySQL 5.0 版本之前，MySQL会对这个查询使用全表扫描，除非改写成两个查询 UNION 的方式。

MySQL 5.0 和后续版本引入了一种叫做“**索引合并**”的策略，查询能够同时使用这两个单列索引进行扫描，并将结果合并。这种算法有三个变种：OR 条件的联合（union），AND 条件的相交（intersection），组合前两种情况的联合及相交。索引合并策略有时候是一种优化的结果，但实际上更多时候说明了表上的索引建得很糟糕：

1. 当出现服务器对多个索引做相交操作时（多个AND条件），通常意味着需要一个包含所有相关列的多列索引，而不是多个独立的单列索引。

2. 当出现服务器对多个索引做联合操作时（多个OR条件），通常需要耗费大量的 CPU 和内存资源在算法的缓存、排序和合并操作上。特别是当其中有些索引的选择性不高，需要合并扫描返回的大量数据的时候。

3. 如果在 explain 中看到有索引合并，应该好好检查一下查询和表的结构，看是不是已经是最优的。



组合索引(concatenated index)：由多个列构成的索引，如 `create index idx_emp on emp(col1, col2, col3, ……)`，则我们称 idx_emp 索引为组合索引。

在组合索引中有一个重要的概念：引导列(leading column)，在上面的例子中，col1 列为**<mark>引导列</mark>**。当我们进行查询时可以使用 ”where col1 = ? ”，也可以使用 ”where col1 = ? and col2 = ?”，这样的限制条件都会使用索引，但是”where col2 = ? ”查询就不会使用该索引。**所以限制条件中包含先导列时，该限制条件才会使用该组合索引。**



#### 覆盖索引

**覆盖索引**（Covering Index），或者叫索引覆盖， 也就是平时所说的**<mark>不需要回表操作</mark>** 

- 就是 select 的数据列只用从索引中就能够取得，不必读取数据行，MySQL 可以利用索引返回 select 列表中的字段，而不必根据索引再次读取数据文件，换句话说**查询列要被所建的索引覆盖**。

- 索引是高效找到行的一个方法，但是一般数据库也能使用索引找到一个列的数据，因此它不必读取整个行。毕竟索引叶子节点存储了它们索引的数据，当能通过读取索引就可以得到想要的数据，那就不需要读取行了。一个索引包含（覆盖）满足查询结果的数据就叫做覆盖索引。

- **判断标准** 

  使用 explain，可以通过输出的 extra 列来判断，对于一个索引覆盖查询，显示为 **using index**，MySQL 查询优化器在执行查询前会决定是否有索引覆盖查询 



#### 使用索引扫描来做排序

MySQL 有两种方式可以生成有序的结果，通过排序操作或者按照索引顺序扫描，如果 explain 的 type 列的值为index，则说明 MySQL 使用了索引扫描来做排序（不要和 extra 列的 Using index 搞混了，那个是使用了覆盖索引查询）。扫描索引本身是很快的，因为只需要从一条索引记录移动到紧接着的下一条记录，但如果索引不能覆盖查询所需的全部列，那就不得不扫描一条索引记录就回表查询一次对应的整行，这基本上都是随机 IO，因此按索引顺序读取数据的速度通常要比顺序地全表扫描慢，尤其是在IO 密集型的工作负载时。

**MySQL 可以使用同一个索引既满足排序，又用于查找行，因此，如果可能，设计索引时应该尽可能地同时满足这两种任务，这样是最好的**。只有当索引的列顺序和 order by 子句的顺序完全一致，并且所有列的排序方向（倒序或升序，创建索引时可以指定ASC或DESC）都一样时，MySQL 才能使用索引来对结果做排序，如果查询需要关联多张表，则只有当 order by 子句引用的字段全部为第一个表时，才能使用索引做排序，order by 子句和查找型查询的限制是一样的，需要满足索引的最左前缀的要求，否则 MySQL 都需要执行排序操作，而无法使用索引排序。



#### 压缩（前缀压缩）索引

MyISAM 使用前缀压缩来减少索引的大小，从而让更多的索引可以放入内存中，这在某些情况下能极大地提高性能。

默认只压缩字符串，但通过参数设置也可以对整数做压缩。

MyISAM 压缩每个索引块的方法是，先完全保存索引块中的第一个值，然后将其他值和第一个值进行比较得到相同前缀的字节数和剩余的不同后缀部分，把这部分存储起来即可。

例如，索引块中的第一个值是“perform“，第二个值是”performance“，那么第二个值的前缀压缩后存储的是类似”7,ance“这样的形式。MyISAM 对**行指针**也采用类似的前缀压缩方式。 

压缩块使用更少的空间，代价是某些操作可能更慢。因为每个值的压缩前缀都依赖前面的值，所以 MyISAM 查找时无法在索引块使用二分查找而只能从头开始扫描。正序的扫描速度还不错，但是如果是倒序扫描——例如ORDER BY DESC——就不是很好了。所有在块中查找某一行的操作平均都需要扫描半个索引块。 
测试表明，对于 CPU 密集型应用，因为扫描需要随机查找，压缩索引使得 MyISAM 在索引查找上要慢好几倍。压缩索引的倒序扫描就更慢了。压缩索引需要在 CPU 内存资源与磁盘之间做权衡。压缩索引可能只需要十分之一大小的磁盘空间，如果是 I/O 密集型应用，对某些查询带来的好处会比成本多很多。 

可以在 CREATE TABLE 语句中指定 PACK_KEYS 参数来控制索引压缩的方式。



#### 重复索引和冗余索引

MySQL 允许在相同列上创建多个索引，无论是有意的还是无意的。MySQL 需要单独维护重复的索引，并且优化器在优化查询的时候也需要逐个地进行考虑，这会影响性能。
重复索引是指在相同的列上按照相同的顺序创建的相同类型的索引。应该避免这样创建重复索引，发现以后也应该立即移除。

冗余索引和重复索引有一些不同。如果创建了索引(A,B)，再创建索引(A)就是冗余索引，因为这只是前一个索引的前缀索引。因此索引(A,B)也可以当做索引(A)来使用（这种冗余只是对B-Tree索引来说的）。但是如果再创建索引(B,A)，则不是冗余索引，索引(B)也不是，因为B不是索引(A,B)的最左前缀。另外，其他不同类型的索引（例如哈希索引或者全文索引）也不会是B-Tree索引的冗余索引，而无论覆盖的索引列是什么。



#### 未使用的索引

除了冗余索引和重复索引，可能还会有一些服务器永远不使用的索引，这样的索引完全是累赘，建议考虑删除，有两个工具可以帮助定位未使用的索引：

1. 在 percona server 或者 mariadb 中先打开 userstat=ON 服务器变量，默认是关闭的，然后让服务器运行一段时间，再通过查询` information_schema.index_statistics` 就能查到每个索引的使用频率。

2. 使用 percona toolkit 中的 pt-index-usage 工具，该工具可以读取查询日志，并对日志中的每个查询进行explain 操作，然后打印出关于索引和查询的报告，这个工具不仅可以找出哪些索引是未使用的，还可以了解查询的执行计划，如：在某些情况下有些类似的查询的执行方式不一样，这可以帮助定位到那些偶尔服务器质量差的查询，该工具也可以将结果写入到 MySQL 的表中，方便查询结果。



## 四、索引优化

### 导致SQL执行慢的原因

1. 硬件问题。如网络速度慢，内存不足，I/O吞吐量小，磁盘空间满了等。 

2. 没有索引或者索引失效。（一般在互联网公司，DBA会在半夜把表锁了，重新建立一遍索引，因为当你删除某个数据的时候，索引的树结构就不完整了。所以互联网公司的数据做的是假删除.一是为了做数据分析,二是为了不破坏索引 ）

3. 数据过多（分库分表）

4. 服务器调优及各个参数设置（调整my.cnf）



### [建索引的几大原则](https://tech.meituan.com/2014/06/30/mysql-index.html "MySQL索引原理及慢查询优化")

1. 最左前缀匹配原则，非常重要的原则，MySQL 会一直向右匹配直到遇到范围查询(>、<、between、like)就停止匹配，比如a = 1 and b = 2 and c > 3 and d = 4 如果建立(a,b,c,d)顺序的索引，d是用不到索引的，如果建立(a,b,d,c)的索引则都可以用到，a,b,d 的顺序可以任意调整。

2. =和in可以乱序，比如a = 1 and b = 2 and c = 3 建立(a,b,c)索引可以任意顺序，MySQL 的查询优化器会帮你优化成索引可以识别的形式。

3. 尽量选择区分度高的列作为索引，区分度的公式是count(distinct col)/count(*)，表示字段不重复的比例，比例越大我们扫描的记录数越少，唯一键的区分度是1，而一些状态、性别字段可能在大数据面前区分度就是0，那可能有人会问，这个比例有什么经验值吗？使用场景不同，这个值也很难确定，一般需要join的字段我们都要求是0.1以上，即平均1条扫描10条记录。

4. 索引列不能参与计算，保持列“干净”，比如 `from_unixtime(create_time) = ’2014-05-29’` 就不能使用到索引，原因很简单，b+ 树中存的都是数据表中的字段值，但进行检索时，需要把所有元素都应用函数才能比较，显然成本太大。所以语句应该写成 `create_time = unix_timestamp(’2014-05-29’)`。

5. 尽量的扩展索引，不要新建索引。比如表中已经有 a 的索引，现在要加(a,b)的索引，那么只需要修改原来的索引即可。



### [优化要注意的一些事](https://www.cnblogs.com/frankdeng/p/8990181.html "优化要注意的一些事")

1. 索引其实就是一种归类方式，当某一个字段属性都不能归类，建立索引后是没什么效果的，或归类就二种（0和1），且各自都数据对半分，建立索引后的效果也不怎么强。

2. 主键的索引是不一样的，要区别理解。

3. 当时间存储为时间戳保存的可以建立前缀索引。

4. 在什么字段上建立索引，需要根据查询条件而定，不要一上来就建立索引，浪费内存还有可能用不到。

5. 大字段（blob）不要建立索引，查询也不会走索引。

6. 常用建立索引的地方：
   - 主键的聚集索引
   - 外键索引
   - 类别只有0和1就不要建索引了，没有意义，对性能没有提升，还影响写入性能
   - 用模糊其实是可以走前缀索引

7. 唯一索引一定要小心使用，它带有唯一约束，由于前期需求不明等情况下，可能造成我们对于唯一列的误判。

8. 由于我们建立索引并想让索引能达到最高性能，这个时候我们应当充分考虑该列是否适合建立索引，可以根据列的区分度来判断，区分度太低的情况下可以不考虑建立索引，区分度越高效率越高。

9. 写入比较频繁的时候，不能开启MySQL的查询缓存，因为在每一次写入的时候不光要写入磁盘还的更新缓存中的数据。
10. 二次SQL查询区别不大的时候，不能按照二次执行的时间来判断优化结果，没准第一次查询后又保存缓存数据，导致第二次查询速度比第二次快，很多时候我们看到的都是假象。
11. Explain 执行计划只能解释SELECT操作。
12. 使用UNION ALL 替换OR多条件查询并集。
13. 对查询进行优化，应尽量避免全表扫描，首先应考虑在 where 及 order by 涉及的列上建立索引。
14. 应尽量避免在 where 子句中对字段进行 null 值判断，否则将导致引擎放弃使用索引而进行全表扫描，如：`select id from t where num is null` 可以在 num上设置默认值 0，确保表中 num 列没有 null 值，然后这样查询：`select id from t where num=0`
15. 应尽量避免在 where 子句中使用!=或<>操作符，否则引擎将放弃使用索引而进行全表扫描。
16. 应尽量避免在 where 子句中使用or 来连接条件，否则将导致引擎放弃使用索引而进行全表扫描，如：`select id from t where num=10 or num=20` 可以这样查询：`select id from t where num=10 union all select id from t where num=20`
17. in 和 not in 也要慎用，否则会导致全表扫描，如：`select id from t where num in(1,2,3)`  对于连续的数值，能用 between 就不要用 in 了：`select id from t where num between 1 and 3`
18. 下面的查询也将导致全表扫描：`select id from t where name like '李%'` 若要提高效率，可以考虑全文检索。
19. 如果在 where 子句中使用参数，也会导致全表扫描。因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然 而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描：`select id from t where num=@num` 可以改为强制查询使用索引：`select id from t with(index(索引名)) where num=@num`
20. 应尽量避免在 where 子句中对字段进行表达式操作，这将导致引擎放弃使用索引而进行全表扫描。如：`select id from t where num/2=100` 应改为: `select id from t where num=100*2`
21. 应尽量避免在where子句中对字段进行函数操作，这将导致引擎放弃使用索引而进行全表扫描。如：`select id from t where substring(name,1,3)='abc' `，name 以 abc 开头的 id 应改为: `select id from t where name like 'abc%'`
22. 不要在 where 子句中的“=”左边进行函数、算术运算或其他表达式运算，否则系统将可能无法正确使用索引。
23. 在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致。
24. 不要写一些没有意义的查询，如需要生成一个空表结构：`select col1,col2 into #t from t where 1=0` 这类代码不会返回任何结果集，但是会消耗系统资源的，应改成这样： create table #t(...)
25. 很多时候用 exists 代替 in 是一个好的选择：`select num from a where num in(select num from b)` 用下面的语句替换：` select num from a where exists(select 1 from b where num=a.num)`
26. 并不是所有索引对查询都有效，SQL是根据表中数据来进行查询优化的，当索引列有大量数据重复时，SQL查询可能不会去利用索引，如一表中有字段sex，male、female几乎各一半，那么即使在sex上建了索引也对查询效率起不了作用。
27. 索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有 必要。
28. 应尽可能的避免更新 clustered 索引数据列，因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered 索引数据列，那么需要考虑是否应将该索引建为 clustered 索引。
29. 尽量使用数字型字段，若只含数值信息的字段尽量不要设计为字符型，这会降低查询和连接的性能，并会增加存储开销。这是因为引擎在处理查询和连接时会逐个比较字符串中每一个字符，而对于数字型而言只需要比较一次就够了。
30. 尽可能的使用 varchar/nvarchar 代替 char/nchar ，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。
31. 任何地方都不要使用 `select * from t` ，用具体的字段列表代替“*”，不要返回用不到的任何字段。

