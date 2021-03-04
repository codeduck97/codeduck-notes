# InnoDB存储引擎

## InnoDB的体系架构

### 后台线程

#### Master THread

#### IO Thread

```
mysql> show variables like 'innodb_version'\g;
+----------------+--------+
| Variable_name  | Value  |
+----------------+--------+
| innodb_version | 5.7.31 |
+----------------+--------+
```



```shell
show engine innodb status\g;

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

```
mysql> show variables like 'innodb_purge_threads'\g;
+----------------------+-------+
| Variable_name        | Value |
+----------------------+-------+
| innodb_purge_threads | 4     |
+----------------------+-------+
1 row in set (0.01 sec)
```



#### Page Cleaner Thread

```
mysql> show variables like 'innodb_buffer_pool_size'\g;
+-------------------------+-----------+
| Variable_name           | Value     |
+-------------------------+-----------+
| innodb_buffer_pool_size | 134217728 |
+-------------------------+-----------+
1 row in set (0.00 sec)
```



### 内存

#### 缓冲池

## Checkpoint技术

如果从缓冲池将页的新数据刷新到磁盘宕机时，事务型数据库普通的做法：采用 Write Ahead Log策略，即当食物提交时先写重做日志的缓存（redo log），再修改缓冲池中的页。因此，即使发生宕机的情况，也可以通过redo log来恢复数据，这就是事务ACID中的D（Durability持久性）。

InnoDB引擎会每秒将重做日志缓存中的内容刷新到重做日志文件（刷新到磁盘）！！！即使当前某个事务还没有提交。

## Master Thread工作方式

## InnoDB的关键特性

- 插入缓冲（Insert Buffer）；
- 两次写（Double Write）；
- 自适应哈希索引；
- 异步IO；
- 刷新邻接表；

文件

表

索引

锁

事务