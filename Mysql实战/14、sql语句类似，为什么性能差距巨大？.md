# Sql语句类似，为什么性能差距巨大

## 条件字段函数操作

对左边的条件列进行了函数操作

```sql

mysql> select count(*) from tradelog where month(t_modified)=7;
```





## 隐示类型转换

在 MySQL 中，字符串和数字做比较的话，是将字符串转换成数字。

```sql
mysql> select * from tradelog where tradeid=110717;
```

比如这句tradeid是字符串类型，在和数字作比较时，会将每一列转换为数字，导致无法走索引

```sql
mysql> select * from tradelog where  CAST(tradid AS signed int) = 110717;
```



## 隐示字符集编码转换

utf8mb4 是 utf8 的超集。类似地，在程序设计语言里面，做自动类型转换的时候，为了避免数据在转换过程中由于截断导致数据错误，也都是“按数据长度增加的方向”进行转换的

**所以会将utf8 隐示转换为utf8mb4**