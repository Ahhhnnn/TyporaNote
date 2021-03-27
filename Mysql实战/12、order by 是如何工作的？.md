# Order by 是如何工作的？

业务中经常有需要排序的场景，假如有一个表如下

```sql
CREATE TABLE `t` (
  `id` int(11) NOT NULL,
  `city` varchar(16) NOT NULL,
  `name` varchar(16) NOT NULL,
  `age` int(11) NOT NULL,
  `addr` varchar(128) DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `city` (`city`)
) ENGINE=InnoDB;
```

查询语句如下

```sql
select city,name,age from t where city='杭州' order by name limit 1000  ;
```



## 执行流程

### 全字段排序

![img](https://static001.geekbang.org/resource/image/82/03/826579b63225def812330ef6c344a303.png)

Extra 这个字段中的“**Using filesor**t”表示的就是需要排序，MySQL 会给每个线程分配一块内存用于排序，称为 **sort_buffer**。

![img](https://static001.geekbang.org/resource/image/53/3e/5334cca9118be14bde95ec94b02f0a3e.png)

- 初始化 sort_buffer，确定放入 name、city、age 这三个字段；
- 从索引 city 找到第一个满足 city='杭州’条件的主键 id，也就是图中的 ID_X；
- 到主键 id 索引取出整行，取 name、city、age 三个字段的值，存入 sort_buffer 中；
- 从索引 city 取下一个记录的主键 id；
- 重复步骤 3、4 直到 city 的值不满足查询条件为止，对应的主键 id 也就是图中的 ID_Y；
- 对 sort_buffer 中的数据按照字段 name 做快速排序；
- 按照排序结果取前 1000 行返回给客户端。



全字段排序示意图

![img](https://static001.geekbang.org/resource/image/6c/72/6c821828cddf46670f9d56e126e3e772.jpg)

**排序这个动作，可能在内存中完成，也可能需要使用外部排序，这取决于排序所需的内存和参数 sort_buffer_size**

sort_buffer_size，就是 MySQL 为排序开辟的内存（sort_buffer）的大小。如果要排序的数据量小于 sort_buffer_size，排序就在内存中完成。但如果排序数据量太大，内存放不下，则不得不利用磁盘临时文件辅助排序。

外部排序一般使用归并排序算法：MySQL 将需要排序的数据分成 12 份，每一份单独排序后存在这些临时文件中。然后把这 12 个有序文件再合并成一个有序的大文件

### rowid 排序

全字段排序存在的问题：

如果查询要返回的字段很多的话，那么 sort_buffer 里面要放的字段数太多，这样内存里能够同时放下的行数很少，要分成很多个临时文件，排序的性能会很差。

只向sort_buffer中存入需要排序的字段和id值

- 初始化 sort_buffer，确定放入两个字段，即 name 和 id；
- 从索引 city 找到第一个满足 city='杭州’条件的主键 id，也就是图中的 ID_X；到主键 id 索引取出整行，取 name、id 这两个字段，存入 sort_buffer 中；
- 从索引 city 取下一个记录的主键 id；
- 重复步骤 3、4 直到不满足 city='杭州’条件为止，也就是图中的 ID_Y；对 sort_buffer 中的数据按照字段 name 进行排序；
- 遍历排序结果，取前 1000 行，并按照 id 的值回到原表中取出 city、name 和 age 三个字段返回给客户端。

rowid排序执行流程图

![img](https://static001.geekbang.org/resource/image/dc/6d/dc92b67721171206a302eb679c83e86d.jpg)

**Rowid排序会多一次步骤7的回表操作**



## 全字段排序 VS rowid 排序

如果 MySQL 实在是担心排序内存太小，会影响排序效率，才会采用 rowid 排序算法，这样排序过程中一次可以排序更多行，但是需要再回到原表去取数据。

如果 MySQL 认为内存足够大，会优先选择全字段排序，把需要的字段都放到 sort_buffer 中，这样排序后就会直接从内存里面返回查询结果了，不用再回到原表去取数据。

**这也就体现了 MySQL 的一个设计思想：如果内存够，就要多利用内存，尽量减少磁盘访问。**



## 其他优化方案

### 联合索引

对name字段加索引，构建联合索引，保证数据有序，不需要进行额外的排序操作。

联合索引示意图：

![img](https://static001.geekbang.org/resource/image/f9/bf/f980201372b676893647fb17fac4e2bf.png)

### 覆盖索引

对city，name，age构建索引，实现覆盖索引，减少回表操作

**执行流程：直接从二级索引中拿到需要的值直接返回即可**

![img](https://static001.geekbang.org/resource/image/df/d6/df4b8e445a59c53df1f2e0f115f02cd6.jpg)



### 备注

Explain 的Extra 字段解释：

【Using filesort】 本次查询语句中有order by，且排序依照的字段不在本次使用的索引中，不能自然有序。需要进行额外的排序工作。

【Using index】 使用了覆盖索引——即本次查询所需的所有信息字段都可以从利用的索引上取得。无需回表，额外去主索引上去数据。 The column information is retrieved from the table using only information in the index tree without having to do an additional seek to read the actual row. This strategy can be used when the query uses only columns that are part of a single index. 

【Using index condition】 使用了索引下推技术ICP。（虽然本次查询所需的数据，不能从利用的索引上完全取得，还是需要回表去主索引获取。但在回表前，充分利用索引中的字段，根据where条件进行过滤。提前排除了不符合查询条件的列。这样就减少了回表的次数，提高了效率。） Tables are read by accessing index tuples and testing them first to determine whether to read full table rows. In this way, index information is used to defer (“push down”) reading full table rows unless it is necessary. See Section 8.2.1.5, “Index Condition Pushdown Optimization”. 

【Using where】 表示本次查询要进行筛选过滤。

【Using temporary】表示的是需要使用临时表