# InnoDB存储引擎

## InnoDB的体系架构

InnoDB存储引擎的体系架构如下图所示，多个InnoDB存储引擎内存块组成了一个大的内存池主要负责如下工作：

- 维护所有进程/线程需要访问的内部数据结构；
- 缓存磁盘上的数据，对磁盘文件修改之间在这里先缓存，提高读取速度；
- 重做日志（redo log）缓冲；
- 按照策略将缓存的内容刷新到磁盘文件中；

![image-20210304223802973](images/image-20210304223802973.png)

### 后台线程

#### Master Thread

Master Thread是一个非常核心的后台线程，主要负责将缓冲池中的数据异步刷新到磁盘，保证数据的一致性，包括脏页的刷新、合并插入缓冲（INSERT BUFFER）、UNDO页的回收等；（内容太多待整理……）

#### IO Thread

查看mysql的版本如下：

```sql
mysql> show variables like 'innodb_version';
+----------------+--------+
| Variable_name  | Value  |
+----------------+--------+
| innodb_version | 5.7.31 |
+----------------+--------+
```

IO线程分别是：write、read、insert、buffer和log IO thread。下列命令显示IO线程的数量。

```shell
show engine innodb status\G;

I/O thread 0 state: waiting for i/o request (insert buffer thread)
I/O thread 1 state: waiting for i/o request (log thread)
I/O thread 2 state: waiting for i/o request (read thread)
I/O thread 3 state: waiting for i/o request (read thread)
I/O thread 4 state: waiting for i/o request (read thread)
I/O thread 5 state: waiting for i/o request (read thread)
I/O thread 6 state: waiting for i/o request (write thread)
I/O thread 7 state: waiting for i/o request (write thread)
I/O thread 8 state: waiting for i/o request (write thread)
I/O thread 9 state: waiting for i/o request (write thread)
```

#### Purge Thread

事务被提交后，其所使用的 undo log可能不再需要，因此需要 PurgeThread来回收已经使用并分配的undo页。查看Purge Thread命令如下：

```
mysql> show variables like 'innodb_purge_threads';
+----------------------+-------+
| Variable_name        | Value |
+----------------------+-------+
| innodb_purge_threads | 4     |
+----------------------+-------+
```

#### Page Cleaner Thread

Page Cleaner Thread其作用是将之前版本中脏页的刷新操作都放人到单独的线程中来完成。其目的是为了减轻原 Master Thread的工作及对于用户査询线程的阻塞，进一步提高 InnoDB存储引擎的性能。

### 内存

#### 缓冲池

缓冲池是用来弥补磁盘速度较慢对数据库性能的影响。在数据库中进行读取页的操作，首先将从磁盘读到的页存放在缓冲池中，这个过程称为将页“FIX”在缓冲池中。下一次再读相同的页时，首先判断该页是否在缓冲池中。若在缓冲池中，称该页在缓冲池中被命中，直接读取该页。否则，读取磁盘上的页。

对于数据库中页的修改操作，则首先修改在缓冲池中的页，然后再以一定的频率刷新到磁盘上。这里需要注意的是，页从缓冲池刷新回磁盘的操作并不是在每次页发生更新时触发，而是通过一种称为 Checkpoint的机制刷新回磁盘。

查看缓冲池的大小命令如下：

```
mysql> show variables like 'innodb_buffer_pool_size';
+-------------------------+-----------+
| Variable_name           | Value     |
+-------------------------+-----------+
| innodb_buffer_pool_size | 134217728 |
+-------------------------+-----------+
```

缓冲池中缓存的数据页类型有：索引页、数据页、undo页、插入缓冲（insert buffer）、自适应哈希索引（adaptive hash index）、InnoDB存储的锁信息（lock info）、数据字典信息（data dictionary）等。不能简单地认为，缓冲池只是缓存索引页和数据页，它们只是占缓冲池很大的一部分而已。

缓冲池结构如下所示：

![image-20210304223736361](images/image-20210304223736361.png)

#### 缓冲池的管理

数据库中的缓冲池是通过LRU（Latest recent used，最近最少使用）算法来进行管理的。即最频繁使用的页在LRU列表的前端，而最少使用的页在LRU列表的尾端。当缓冲池不能存放新读取到的页时，将首先释放LRU列表中尾端的页。

而InnoDB存储引擎对传统的LRU进行了优化。InnoDB的存储引擎中，LRU列表中还加人了 midpoint位置。新读取到的页，虽然是最新访问的页，但并不是直接放入到LRU列表的首部，而是放入到LRU列表的 midpoint位置。在默认配置下，该位置在LRU列表长度的5/8处。midpoint位置可由参数indb_old blocks pct控制，如：

```sql
mysql> show variables like 'innodb_old_blocks_pct';
+-----------------------+-------+
| Variable_name         | Value |
+-----------------------+-------+
| innodb_old_blocks_pct | 37    |
+-----------------------+-------+
```

参数 innodb_old_blocks_pct默认值为37，表示新读取的页插入到LRU列表尾端的37%的位置（差不多38的位置）。在 InnoDB存储引擎中，把midpoint之后的列表称为old列表，之前的列表称为new列表。可以简单地理解为new列表中的页都是最为活跃的热点数据。

InnoDB存储引擎还引入了另一个参数来进一步管理LRU列表，这个参数是 innodb_old_blocks_time，用于表示页读取到mid位置后需要等待多久才会被加入到LRU列表的热端。可以通过下面的方法尽可能使LRU列表中热点数据不被刷出。

```sql
mysql> set global innodb_old_blocks_time=1000;
```

LRU列表用来管理已经读取的页，但当数据库刚启动时，LRU列表是空的，即没有任何的页。这时页都存放在**Free List**中，当需要从缓冲池中分页时，首先从Free列表中查找是否有可用的空闲页，若有则将该页从Free列表中删除，放入到LRU列表中。

在LRU列表中的页被修改后，称该页为脏页（dirty page），即缓冲池中的页和磁盘上的页的数据产生了不一致。这时数据库会通过 CHECKPOINT机制将脏页刷新回磁盘，而 **Flush List**中的页即为脏页列表。

#### 重做日志缓冲（redo log buffer）

InnoDB存储引擎首先将重做日志信息先放人到重做日志缓冲区，然后按一定频率将其刷新到重做日志文件。重做日志缓冲一般不需要设置得很大，因为一般情况下每一秒钟会将重做日志缓冲刷新到日志文件，因此用户只需要保证每秒产生的事务量在这个缓冲大小之内即可。

```sql
mysql> show variables like 'innodb_log_buffer_size';
+------------------------+---------+
| Variable_name          | Value   |
+------------------------+---------+
| innodb_log_buffer_size | 1048576 |
+------------------------+---------+
```

下列三种情况下会将重做日志缓冲中的内容刷新到外部磁盘的重做日志文件中

- Master Thread每一秒将重做日志缓冲刷新到重做日志文件；
- 每个事务提交时会将重做日志缓冲刷新到重做日志文件；
- 当重做日志缓冲池剩余空间小于1/2时，重做日志缓冲刷新到重做日志文件；

#### 额外的内存池

在 InnoDB存储引擎中，对内存的管理是通过一种称为内存堆（heap）的方式进行的。

在对一些数据结构本身的内存进行分配时，需要从额外的内存池中进行申请，当该区域的内存不够时，会从缓冲池中进行申请。例如，分配了缓冲池（innodb_buffer_pool），但是每个缓冲池中的帧缓冲（frame buffer）还有对应的缓冲控制对象（buffer control block），这些对象记录了一些诸如LRU、锁、等待等信息，而这个对象的内存需要从额外内存池中申请。因此，在申请了很大的 InnoDB缓冲池时，也应考虑相增加额外内存池的容量。

## Checkpoint技术

如果从缓冲池将页的新数据刷新到磁盘宕机时，事务型数据库通常做法：采用 Write Ahead Log策略，即当事务提交时先写重做日志的缓存（redo log），再修改缓冲池中的页。因此，即使发生宕机的情况，也可以通过redo log来恢复数据，这就是事务ACID中的D（Durability持久性）。

当重做日志不断增加，难免会让数据恢复时间增加，而Checkpoint技术就是记录哪些redo log是需要进行恢复的。Checkpoint（检查点）技术的目的是解决以下几个问题：

- 缩短数据库的恢复时间；
- 缓冲池不够用时，将脏页刷新到磁盘
- 重做日志不可用时，刷新脏页。

当数据库发生宕机时，数据库不需要重做所有的日志，因为 Checkpoint之前的页都已经刷新回磁盘。故数据库只需对 Checkpoint后的重做日志进行恢复。

## InnoDB的关键特性

### 插入缓冲

#### Insert Buffer

 Insert Buffer针对非聚集索引（二级索引或辅助索引）的插入或更新操作，不是每一次直接插入到索引页中，而是先判断插入的非聚集索引页是否在缓冲池中，若在，则直接插入；若不在，则先放人到一个 Insert Buffer对象中，好似欺骗数据库这个非聚集的索引已经插到叶子节点，而实际并没有，只是存放在另一个位置。然后再以一定的频率和情况进行 Insert Buffer和辅助索引页子节点的 merge（合并）操作，这时通常能将多个插入合并到一个操作中（因为在一个索引页中），这就大大提高了对于非聚集索引插入的性能。

Insert Buffer的使用需要同时满足以下两个条件：

- 索引是辅助索引（secondary index）；
- 索引不是唯一（unique）的；

#### Change Buffer

Change Buffer，可将其视为 Insert Buffer的升级。
InnoDB存储引擎可以对INSERT、DELETE、UPDATE都进行缓冲，他们分别是：Insert Buffer、Delete Buffer、Purge buffer，和之前 Insert Buffer一样，Change Buffer适用的对象依然是非唯一的辅助索引对一条记录进行 UPDATE操作可能分为两个过程：

- 将记录标记为已删除；
- 真正将记录删除；

因此 Delete Buffer对应 UPDATE操作的第一个过程，即将记录标记为删除。Purg Buffer对应 UPDATE操作的第二个过程，即将记录真正的删除。

### Merge Insert Buffer

Insert/Change Buffer是一棵B+树。若需要实现插入记录的辅助索引页不在缓冲池中，那么需要将辅助索引记录首先插入到这棵B+树中。但是 Insert Buffer中的记录何时合并（merge）到真正的辅助索引中呢？
概括地说，Merge Insert Buffer的操作可能发生在以下几种情况下

- 辅助索引页被读取到缓冲池时；
- Insert Buffer Bitmap页追踪到该辅助索引页已无可用空间时；
- Master Thread主动刷新；

第一种情况：当辅助索引页被读取到缓冲池中时，例如这在执行正常的 SELECT查询操作，这时需要检查 Insert Buffer Bitmap页，然后确认该辅助索引页是否有记录存放于 Insert Buffer B+树中。若有，则将 Insert Buffer B+树中该页的记录插入到该辅助索引页中。

第二种情况：Insert Buffer Bitmap页用来迫踪每个辅助索引页的可用空间，并至少有1/32页的空间。若插入辅助索引记录时检测到插入记录后可用空间会小于1/32页，则会强制进行个合并操作，即强制读取辅助索引页，将 Insert Buffer B+树中该页的记录及待插入的记录插入到辅助索引页中。

第三种情况：在 Master Thread线程中每秒或每10秒会进行一次 Merge Insert Buffer的操作，不同之处在于每次进行mere操作的页的数量不同。

### 两次写（Double Write）

如果说 Insert Buffer带给 InnoDB存储引擎的是性能上的提升，那么 doublewrite（两次写）带给 InnoDB存储引擎的是数据页的可靠性。

当发生数据库宕机时，可能 InnoDB存储引擎正在写入某个页到表中，而这个页只写了一部分，比如16KB的页，只写了前4KB，之后就发生了宕机，这种情况被称为部分写失效（partial page write）。

有经验的DBA也许会想，如果发生写失效，可以通过重做日志进行恢复。这是个办法。但是必须清楚地认识到，重做日志中记录的是对页的物理操作，如偏移量800，写"aa'记录。如果这个页本身已经发生了损坏，再对其进行重做是没有意义的。

这就是说，在应用（apply）重做日志前，用户需要一个页的副本，当写入失效发生时，先通过页的副本来还原该页，再进行重做，这就是 doublewrite。在 InnoDB存储引擎中doublewrite的体系架构如图：

![image-20210305001042882](images/image-20210305001042882.png)

doublewrite由两部分组成，一部分是内存中的 doublewrite buffer，大小为2MB，另部分是物理磁盘上共享表空间中连续的128个页，即2个区（extent），大小同样为2MB。在对缓冲池的脏页进行刷新时，并不直接写磁盘，而是会通过 memcpy函数将脏页先复制到内存中的 doublewrite buffer，之后通过 doublewrite buffer再分两次，每次1MB顺序地写入共享表空间的物理磁盘上，待写入磁盘后将doublewrite buffer中的脏页数据写入实际的各个数据库表空间文件(离散写)；

如果操作系统在将页写入磁盘的过程中发生了崩溃，在恢复过程中，InnoDB存储引擎可以从共享表空向中的 doublewrite中找到该页的一个副本，将其复制到表空间文件，再应用重做日志。

### 自适应哈希索引

InnoDB存储引擎会监控对表上各索引页的查询。如果观察到建立哈希索引可以带来速度提升，则建立哈希索引，称之为自适应哈希索引（Adaptive Hash Index，AHI）。

AHI是通过缓冲池的B+树页构遣而来，因此建立的速度很快，而且不需要对整张表构建哈希索引。InnoDB存储引擎会自动根据访问的频率和模式来自动地为某些热点页建立哈希索引。

### 异步IO

与AIO对应的是 Sync IO，即每进行一次Io操作，需要等待此次操作结束才能继续接下来的操作。但是如果用户发出的是一条索引扫描的查询，那么这条SQL查询语句可能需要扫描多个索引页，也就是需要进行多次的IO操作。在每扫描一个页并等待其完成后再进行下一次的扫描，这是没有必要的。用户可以在发出一个IO请求后立即再发出另一个IO请求，当全部IO请求发送完毕后，等待所有IO操作的完成，这就是AIO。

在 InnoDB存储引擎中，read ahead方式的读取都是通过AIO完成，脏页的刷新，即磁盘的写入操作则全部由AIO完成。222

### 刷新邻接表

InnoDB存储引擎还提供了 Flush Neighbor Page（刷新邻接页）的特性。

其工作原理为：当刷新一个脏页时，InnoDe存储引擎会检测该页所在区（extent）的所有页，如果是脏页，那么一起进行刷新。这样做的好处显而易见，通过AIO可以将多个1O写入操作合并为一个IO操作，故该工作机制在传统机械磁盘下有着显著的优势。

但是需要考虑到下面两个问题：

- 是不是可能将不怎么脏的页进行了写人，而该页之后又会很快变成脏页？

- 固态硬盘有着较高的IOPS，是否还需要这个特性？

为此，InnoDB存储引擎从1.2.x版本开始提供了参数 innodb_flush_neighbors，用来控制是否启用该特性。对于传统机械硬盘建议启用该特性，而对于固态硬盘有着超高IOPS性能的磁盘，则建议将该参数设置为0，即关闭此特性。

# 文件

## 参数文件

参数文件主要包含数据库启动时会加载参数文件的配置信息来实现数据库的初始化工作，例如 InnoDB 缓冲池大小：innodb_buffer_pool_size = 1G。参数的信息通过Key/Value的形式存储在文件中，参数的类型可分为：

- 静态参数：在整个数据库运行时的生命周期内都不允许改变；
- 动态参数：可以通过命令行动态的修改参数；

## 日志文件

日志文件记录了影响 MySQL数据库的各种类型活动。MySQL数据库中常见的日志文件有：

- 错误日志（error log）；
- 二进制日志（binlog）；
- 慢查询日志（slow query log）；
- 查询日志（log）；

### 错误日志

**错误日志文件对 MySQL的启动、运行、关闭过程进行了记录。**该文件不仅记录了所有的错误信息，也记录一些警告信息或正确的信息。可通过下列命令查看错误日志所在位置：

```mysql
# 查看mysql数据存储位置
mysql> show variables like 'datadir';
+---------------+---------------------------------------------+
| Variable_name | Value                                       |
+---------------+---------------------------------------------+
| datadir       | C:\ProgramData\MySQL\MySQL Server 5.7\Data\ |
+---------------+---------------------------------------------+

# 查看错误日志存储的相对位置
mysql> show variables like 'log_error';
+---------------+----------------------+
| Variable_name | Value                |
+---------------+----------------------+
| log_error     | .\JASON-YOGA-14S.err |
+---------------+----------------------+
```

上述信息可见，错误日志的名称（通常为主机的名称.err）与存放的相对路径地址，查看具体内容如下：

![image-20210305214524050](images/image-20210305214524050.png)

### 慢查询日志

**慢查询日志（slow log）**可帮助DBA定位可能存在问题的SQL语句，从而进行SQL语句层面的优化。例如，可以在 MySQL启动时设一个阈值，将运行时间超过该值的所有SQL语句都记录到慢查询日志文件中。该阈值可以通过参数 `long_query_time`来设置，默认值为10，代表10秒。

```sql
mysql> show variables like 'long_query_time';
+-----------------+-----------+
| Variable_name   | Value     |
+-----------------+-----------+
| long_query_time | 10.000000 |
+-----------------+-----------+
```

`log_queries_not_using_indexes`变量的作用是：如果没有SQL语句没有使用索引，则MySQL会将该条SQL存储到慢查询日志文件。使用时需要手动开启（在mysql配置文件中添加参数）。

```sql
mysql> show variables like 'log_queries_not_using_indexes';
+-------------------------------+-------+
| Variable_name                 | Value |
+-------------------------------+-------+
| log_queries_not_using_indexes | OFF   |
+-------------------------------+-------+
```

查看慢日志文件名称及所在位置：

```sql
mysql> show variables like 'slow_query_log_file'\G;
+---------------------+-------------------------+
| Variable_name       | Value                   |
+---------------------+-------------------------+
| slow_query_log_file | JASON-YOGA-14S-slow.log |
+---------------------+-------------------------+
```

### 查询日志

**查询日志记录了所有对 MySQL数据库请求的信息，无论这些请求是否得到了正确的执行。**默认文件名为：主机名.log。如查看一个查询日志是否开启以及查询日志的文件名称：

```sql
mysql> show variables like '%general_log%';
+------------------+--------------------+
| Variable_name    | Value              |
+------------------+--------------------+
| general_log      | OFF                |
| general_log_file | JASON-YOGA-14S.log |
+------------------+--------------------+
```

`general_log=OFF`表示没有开启记录查询日志的功能，需手动开启，而JASON-YOGA-14S.log为查询日志的名称。启动查询日志需要添加mysql配置文件信息。

### 二进制日志

**二进制日志（binary log）记录了对 MySQL数据库执行更改的所有操作**，**但是不包括 SELECT和SHOW这类操作**，因为这类操作对数据本身并没有修改。**如果用户想记录 SELECT和SHOW操作，那只能使用查询日志，而不是二进制日志。**二进制日志主要作用如下：

- 恢复（recovery）：某些数据的恢复需要二进制日志，例如，在一个数据库全备文件恢复后，用户可以通过二进制日志进行 point-in-time的恢复。
- 复制（replication）：其原理与恢复类似，通过复制和执行二进制日志使一台远程的MySL数据库（一般称为 slave或 standby）与一台 MySQL数据库（一般称为 master或 primary）进行实时同步。
- 审计（audit）：用户可以通过二进制日志中的信息来进行审计，判断是否有对数据库进行注入的攻击。

通过修改mysql的配置文件添加配置参数`log-bin=name`来启动二进制日志，若不指定name，则默认为主机名，

```mysql
# 未启动
mysql> show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | OFF   |
+---------------+-------+

# 已启动，启动后日志默认存放路径为mysql data的根路径
mysql> show variables like 'log_bin';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| log_bin       | ON    |
+---------------+-------+
```

## 表结构定义文件

因为 MySQL插件式存储引擎的体系结构的关系，**MySQL数据的存储是根据表进行的**，不论表采用何种存储引擎，只要创建一张表（或视图）MySQL就在磁盘生一个以`.frm`为后缀名的文件，**这个文件记录了该表的表结构定义。**

## InnoDB存储引擎文件

### 表空间文件

**InnoDB采用将存储的数据按表空间（tablespace）进行存放的设计。**在默认配置下会有一个初始大小为10MB，名为 `ibdata1`的文件。该文件就是默认的表空间文件（tablespace file），InnoDB存储引擎的表的数据都会记录到该共享表空间中。

除了默认的表空间文件外，用户还可以为每个InnoDB表配置生成独立表空间，独立表的命名规则：.ibd，这些单独的表空间文件仅存储该表的数据、索引、和插入缓冲BITMAP等信息，其余信息还是存放在默认表空间中。

```mysql
# 默认表空间的配置方式: 修改mysql配置文件
# 下列配置参数为 用两个文件夹组成表空间，文件大小为2000MB，用完了2000MB后可以自动增长
innodb_data_file_path = /xx/ibdata1:2000M;/yy/ibdata2:2000M:autoextend

# 开启独立表空间
mysql> set @@global.innodb_file_per_table=on; 
```

### 重做日志文件

在默认情况下，在 InnoDB存储引擎的数据目录下会有两个名为 `ib_logfile0`和`ib_logfile1`的文件。这两个文件就是重做日志文件（redo log file）。

每个 InnoDB存储引擎至少有1个重做日志文件组（group），每个文件组下至少有2个重做日志文件，如默认的 ib_logfile0和 ib_logfile1。在日志组中每个重做日志文件的大小一致，并以循环写的方式运行。

InnoDB存储引擎先写重做日志文件1，当达到文件的最后时会切换至重做日志文件2，再当重做日志文件2也被写满时，会再切换到重做日志文件1中。

当实例或介质失败（media failure）时，重做日志文件就能派上用场。例如，数据库由于所在主机掉电导致实例失败，InnoDB存储引擎会使用重做日志恢复到掉电前的时刻，以此来保证数据的完整性。

**重做日志文件和二进制文件的区别：**

- 二进制日志会记录所有与 MySQL数据库有关的日志记录，包括 InnoDB、MyISAM、Heap等其他存储引擎的日志。而 **InnodB存储引擎的重做日志**只记录有关该存储引擎本身的**事务日志**。
- 记录的内容不同。无论用户将二进制日志文件记录的都是关于一个事务的具体操作内容，即该日志是逻辑日志。而InDB存储引擎的重做日志文件记录的是关于每个页（Page）的更改的物理情况。
- 写入的时间不同。二进制日志文件仅在事务提交前进行提交，即只写磁盘一次，不论这时该事务多大。而在事务进行的过程中，却不断有重做日志条目（redo entry）被写入到重做日志文件中。

## 套接字文件

在UNIX系统下本地连接 MySQL可以采用UNX域套接字方式，这种方式需要一个套接字（socket）文件。套接字文件可由参数 socket控制。一般在/tmp目录下，名为 mysql.sock。

## PID文件

当 MySQL实例启动时，会将自已的进程ID写人一个文件中，该文件即为pid文件。该文件可由参数 pid_file控制，默认位于数据库目录下，文件名为主机名pid

# 表

## 索引组织表

在 InnoDB存储引擎中，表都是根据主键顺序组织存放的，这种存储方式的表称为索引组织表（index organized table）。**在 InnoDB存储引擎表中，每张表都有个主键（Primary Key）**，**如果在创建表时没有显式地定义主键，则 InnoDB存储引擎会按如下方式选择或创建主键：**

- 首先判断表中是否有非空的唯一索引（Unique NOT NULL），如果有，则该列即为主键。

- 如果不符合上述条件，InnoDB存储引擎自动创建一个6字节大小的指针。

当表中有多个非空唯一索引时，InnoDB存储引擎将选择建表时第一个定义的非空唯一索引为主键。

## InnoDB逻辑存储结构

从 InnoDB存储引擎的逻辑存储结构看，**所有数据都被逻辑地存放在表空间（tablespace）**。

表空间（table space）由段（segment）、区（extent）、页（page）组成。页在一些文档中有时也称为块（block），InnoDB存储引擎的逻辑存储结构图如下所示：

![image-20210306193119017](images/image-20210306193119017.png)

上图引自https://www.cnblogs.com/ilifeilong/p/7347920.html

### 表空间

**表空间**可以看做是 InnoDB存储引擎逻辑结构的最高层，**默认情况下InnoDB存储引擎有一个共享表空间 ibdata1，即所有数据都存放在这个表空间内。**

如果用户启用了参数 `innodb_file_per_table`，**则每张表内的数据可以单独放到一个表空间（xx.ibd文件）内**。需要注意的是每个单独的表空间内存放的只是数据、索引和插人缓冲 Bitmap页，其他类的数据，如回滚（undo）信息，插人缓冲索引页、系统事务信息，二次写缓冲（Double write buffer）等还是存放在原来的共享表空间内。

### 段

**表空间是由各个段组成的，常见的段有数据段、索引段、回滚段等。**

InnoDB存储引擎表是索引组织的（index organized），因此数据即索引，索引即数据。那么**数据段即为B+树的叶子节点**（上图中的Leaf node segment），**索引段即为B+树的非索引节点**（图41的Non-leaf node segment）。回滚段较为特殊。在InnoDB存储引擎中，对段的管理都是由引擎自身所完成。

### 区

区是由连续页组成的空间，在任何情况下**每个区的大小都为1MB**。为了保证区中页的连续性，InnoDB存储引擎一次从磁盘申请4~5个区。在默认情况下，InnoDB存储引擎**页的大小为16KB**，即一个区中一共有64个连续的页。

### 页

**页是 InnoDB磁盘管理的最小单位。**在 InnoDe存储引擎中，默认每个页的大小为16KB。而从InnoDB1.2.x版本开始，可以通过参数 innodb_page_size将页的大小设置为4K、8kK、16K。若设置完成，则所有表中页的大小都为 innodb_page_size，不可以对其再次进行修改。除非通过 mysqldump导入和导出操作来产生新的库。

在 InnodB存储引擎中，常见的页类型有：

- 数据页（B-tree Node）；
- undo页（undo Log Page）；
- 系统页（System Page）；
- 事务数据页（Transaction system Page）；
- 插入缓冲位图页（Insert Buffer Bitmap）；
- 插入缓冲空闲列表页（Insert Buffer Free List）；
- 未压缩的二进制大对象页（Uncompressed BLOB Page）；
- 压缩的二进制大对象页（compressed BLOB Page）；

### 行

InnoDB存储引擎是面向行的（row-oriented），也就说数据是按行进行存放的，即页中保存表中一行一行的数据。每个页存放的行记录也是有硬性定义的，最多允许存放16KB/2-200行的记录，即7992行记录。除了面向行存储的数据库还有column-oriented的数据库。

## InnoDB数据页结构

**页是 InnoDB存储引擎管理数据库的最小磁盘单位。**B-ree Node类型的页存放的即是表中行的实际数据。



![image-20210306194154679](images/image-20210306194154679.png)

InnodB数据页由以下7个部分组成：

- File header：表示页的一些通用信息，占固定的38字节。
- Page Header：表示数据页专有的一些信息，占固定的56个字节。
-  Infimum和 Supremum Records：两个虚拟的伪记录，分别表示页中的最小和最大记录，占固定的26个字节。
-  User Records：存储用户数据（即插入数据库的数据），大小不固定。
- Free Space：页中尚未使用的部分，大小不确定。
- Page Directory：页中某些记录的相对位置，也就是各个槽在页面中的地址偏移量，大小不固定，插入的记录越多，这个部分占用的空间越多。
-  File trailer：用于检验页是否完整的部分，占用固定的8个字节。

其中 File header、Page Header、File trailer的大小是固定的，分别为38、56、8字节，这些空间用来标记该页的一些信息，如 Checksum，数据页所在B+树索引的层数等。User Records、Free Space、Page Directory这些部分为实际的行记录存储空间，因此大小是动态的。

InnoDB会把页中的记录划分为若干个组，每个组的最后一个记录的地址偏移量作为一个槽，存放在Page Directory中，所以在一个页中根据主键查找记录是非常快的，分为两步：

- 通过二分法确定该记录所在的槽。

- 通过记录的next_record属性遍历该槽所在的组中的各个记录。

每个数据页的File Header部分都有上一个和下一个页的编号，因此所有的数据页会组成一个双链表。

为了保证从内存中同步到磁盘的页的完整性，在页的首部和尾部都会存储页中数据的校验和和页面最后修改时对应的LSN值，如果首部和尾部的校验和和LSN值校验不成功的话，就说明同步过程出现了问题。

上述总结 引自https://blog.csdn.net/qq_43561507/article/details/108975856

# 索引与算法

## InnoDB索引概述

InnoDB存储引擎支持以下几种常见的索引：

- B+树索引
- 全文索引
- 哈希索

InnoDB存储引擎支持的哈希索引是自适应的，InnoDB存储引擎会根据表的使用情况自动为表生成哈希索引，不能人为干预是否在一张表中生成哈希索引。

B+树索引是目前关系型数据库系统中查找最为有效的索引。B+树索引的构造类似于二叉树，根据键值（Key value）快速找到数据。

**常见的误区是：**B+树索引并不能找到一个给定键值的具体行。B+树索引能找到的只是被查找数据行所在的页。然后数据库通过把页读入到内存，再在内存中进行查找具体行，最后得到要查找的数据。

## B+树索引

**B树与B+树的区别：**

- B 树的所有节点既存放键(key) 也存放数据(data)；**B+树叶子节点存放 key 和 data，其他内节点只存放 key。**
- **B 树的叶子节点都是独立的，叶子节点之间无法顺序访问；**B+树所有的叶子节点数据构成了一个有序链表，可以进行顺序访问（双向链表）；
- B 树的检索的过程相当于对范围内的每个节点的关键字做二分查找，可能还没有到达叶子节点，检索就结束了。而 B+树的检索效率就很稳定，任何查找都是从根节点到叶子节点的过程；

下图为B+数索引的数据结构：（引自https://blog.csdn.net/weixin_43155866/article/details/108791358）

![image-20210306203810163](images/image-20210306203810163.png)

**数据库中的B+树索引可以分为：**

- 聚集索引（clustered inex）：主键索引（Primary Key）；
- 辅助索引（secondary index）（亦或称为：非聚集索引/二级索引）：普通索引、唯一索引、组合索引

B+树索引详述文章：https://www.jianshu.com/p/486a514b0ded

### 聚集索引

**聚集索引（clustered index）就是以每张表的主键构造一棵B+树，同时叶子节点中存放整张表的行记录数据，也将聚集索引的叶子节点称为数据页（数据结构如上图所示）。**聚集索引的这个特性决定了索引组织表中数据也是索引的一部分。同B+树数据结构一样，每个数据页都通过一个双向链表来进行链接。此外聚集索引的存储并不是物理连续的，而是逻辑连续的，这样可以提高维护效率以及提高存储空间利用率。下图摘自网络（@Draveness）

![image-20210308233804422](images/image-20210308233804422.png)

聚集索引的一个好处是，它对于主键的排序查找和范围查找速度非常快。叶子节点的数据就是用户所要查询的数据。如用户需要查询一张注册用户的表，查询最后注册的10位用户，由于B+树索引是双向链表的，用户可以快速找到最后一个数据页，并取出10条记录。

### 辅助索引

**辅助索引（Secondary Index，也称非聚集索引），叶子节点并不包含行记录的全部数据。**叶子节点除了包含键值以外，**每个叶子节点中的索引行中还包含了一个书签（bookmark）**。该书签用来告诉 InnodB存储引擎哪里可以找到与索引相对应的行数据。InnoDB存储引擎的辅助索引的书签就是相应行数据的聚集索引键。

下图（摘自网络@Draveness）显示了InnoDB存储引擎中辅助索引与聚集索引的关系：其辅助索引为 first_name 与 age 的组合索引，叶子节点存储辅助索引信息与主键索引的主键，然后通过主键索引来查到具体的数据行信息，因此如果辅助索引树和聚集索引树的高度一致，辅助索引遍历3次，则聚集索引同样遍历3次才可以获取最终数据页。

![image-20210308235207014](images/image-20210308235207014.png)

#### 组合索引

如上图所示的组合索引数据结构，组合索引是将一张表中的多个列组合起来进行索引，在建立索引时可以选择字段的长度进行建立，如：

```mysql
CREATE TABLE t3
(
    id  INT NOT NULL,
    username VARCHAR(30) 　NOT NULL,
    age  INT NOT　 NULL,
    info VARCHAR(255),
    INDEX MultiIdx(id, username, info(50)) # 选则info的前50个字符建立索引
);
```

组合索引在索引时，须遵守最左前缀原则，即：

```mysql
# 该查询语句走索引id和username的组合索引
SELECT * FROM t3 WHERE id=1 AND username='zs';

# 该查询语句由于info无法匹配左前缀username字段，因此只对id进行索引
SELECT * FROM t3 WHERE id=1 AND info='man';

# 由于最左前缀id不存在，因此不走索引MultiIdx组合索引
SELECT * FROM t3 WHERE username='zs' AND info='man';

# 即使字段之间乱序组合，只要存在最左前缀，InnoDB优化器就会对索引语句进行优化
SELECT * FROM t3 WHERE username='zs' AND id=1 AND info='man'; 
```

#### 覆盖索引

InnoDB存储引擎支持覆盖索引（covering index，或称索引覆盖），即从辅助索引中就可以得到查询的记录，而不需要查询聚集索引中的记录。使用覆盖索引的一个好处是辅助索引不包含整行记录的所有信息，故其大小要远小于聚集索引，因此可以减少大量的IO操作。

## 全文索引

全文检索（Full-Text Search）是将存储于数据库中的整本书或整篇文章中的任意内容信息查找出来的技术。它可以根据需要获得全文中有关章、节、段、句、词等信息，也可以进行各种统计和分析。

## Hash算法

**InnoDB存储引擎使用哈希算法来对字典进行查找，其冲突机制采用链表方式，哈希函数采用除法散列方式。**

对于缓冲池页的哈希表来说，在缓冲池中的Page页都有令个chain指针，它指向相同哈希函数值的页。而对于除法散列，m的取值为略大于2倍的缓冲池页数量的质数。例如：当前参数 innodb_buffer_pool_size的大小为10M，则共有640个16KB的页。对于缓冲池页内存的哈希表来说，需要分配640×2=1280个槽，但是由于1280不是质数，需要取比1280略大的一个质数，应该是1399，所以在启动时会分配1399个槽的哈希表，用来哈希查询所在缓冲池中的页。

那么 InnoDB存储引擎的缓冲池对于其中的页是怎么进行查找的呢？上面只是给出了一般的算法，怎么将要查找的页转换成自然数呢？

其实也很简单，InnoDB存储引擎的表空间都有一个 space_id，用户所要查询的应该是某个表空间的某个连续16KB的页，即偏移量 offset。InnoDB存储引擎将 space_id左移20位，然后加上这个 space_id和offset，即关键字K=space_id<<20+-space_id+ offset，然后通过除法散列到各个槽中去。

# 锁

# 事务