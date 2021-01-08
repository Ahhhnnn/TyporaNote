# Mysql为什么会突然抖一下？

## Sql语句为什么变慢了

因为在Sql语句在更新过程中，使用WAL 机制。InnoDB 在处理更新语句的时候，只做了写日志这一个磁盘操作。这个日志叫作 redo log（重做日志）。写进内存后直接返回成功，后续在刷进磁盘中（Flush）。

**当内存数据页跟磁盘数据页内容不一致的时候，我们称这个内存页为“脏页”。内存数据写入到磁盘后，内存和磁盘上的数据页的内容就一致了，称为“干净页”。**



所以很明显平时很快的sql，其实就是在写内存和日志，而 MySQL 偶尔“抖”一下的那个瞬间，可能就是在刷脏页（flush）。

## 什么情况导致Mysql进行Flush

### redo log写满了

InnoDB 的 redo log 写满了。这时候系统会停止所有更新操作，把 checkpoint 往前推进，redo log 留出空间可以继续写。如图所示：

![image-20210107222842351](/Users/hening/Library/Application Support/typora-user-images/image-20210107222842351.png)

**把 checkpoint 位置从 CP 推进到 CP’，就需要将两个点之间的日志（浅绿色部分），对应的所有脏页都 flush 到磁盘上。之后，图中从 write pos 到 CP’之间就是可以再写入的 redo log 的区域**



### 内存不足

当需要新的内存页，而内存不够用的时候，就要淘汰一些数据页，空出内存给别的数据页使用。如果淘汰的是“脏页”，就要先将脏页写到磁盘。

为什么不直接清除掉内存中的数据，在下次请求数据的时候在通过redo log应用进行刷页呢?

因为如果清除掉了内存，那么redolog成为了唯一记录真实数据的地方，但是redolog是被清除的，那么就会影响redolog的清除与推进。



### Mysql系统空闲的时候

mysql认为系统空闲的时候，就会后台开启进程进行flush



### Mysql关闭的时候

MySQL 正常关闭的情况。这时候，MySQL 会把内存的脏页都 flush 到磁盘上，这样下次 MySQL 启动的时候，就可以直接从磁盘上读数据，启动速度会很快。



## Flush控制策略

第三种和第四种情况不考虑，因为一个是系统空闲和关闭的时候，对性能的要求并不关心。

**第一种和第二种会因为 日志写满更新操作全部阻塞 和 等待淘汰脏业延长响应时间，从而影响系统性能**



### 刷盘速度

正确地告诉 InnoDB 所在主机的 IO 能力，这样 InnoDB 才能知道需要全力刷脏页的时候，可以刷多快。

通过参数**innodb_io_capacity**控制，一般设置为磁盘的 IOPS。

测试磁盘的IOPS，**fio命令**

```
fio -filename=$filename -direct=1 -iodepth 1 -thread -rw=randrw -ioengine=psync -bs=16k -size=500M -numjobs=10 -runtime=10 -group_reporting -name=mytest 
```



### 脏页比例

参数 innodb_max_dirty_pages_pct 是脏页比例上限，默认值是 75%，注意观察mysql脏页比例，不要超过75%。

脏页比例是通过 Innodb_buffer_pool_pages_dirty/Innodb_buffer_pool_pages_total 得到的

```sql
mysql> select VARIABLE_VALUE into @a from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_dirty';
select VARIABLE_VALUE into @b from global_status where VARIABLE_NAME = 'Innodb_buffer_pool_pages_total';
select @a/@b;
```



### 脏页连坐

参数innodb_flush_neighbors 参数就是用来控制连坐行为的，值为 1 的时候会有上述的“连坐”机制，值为 0 时表示不找邻居，自己刷自己的。

**连坐机制：如果需要flush一个脏页时，这个数据页旁边的数据页刚好是脏页，就会把这个“邻居”也带着一起刷掉；而且这个把“邻居”拖下水的逻辑还可以继续蔓延，也就是对于每个邻居数据页，如果跟它相邻的数据页也还是脏页的话，也会被放到一起刷。**

