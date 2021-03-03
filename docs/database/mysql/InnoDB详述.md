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

# InnoDB关键特性

文件

表

索引

锁

事务