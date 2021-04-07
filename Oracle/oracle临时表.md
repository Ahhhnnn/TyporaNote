# 临时表

因为项目上有部分地方用到了临时表，学习一下。

## 基本概念

临时表就是用来暂时保存临时数据（亦或叫中间数据）的一个数据库对象，它和普通表有些类似，然而又有很大区别。它只能存储在临时表空间，而非用户的表空间。ORACLE临时表是会话或事务级别的，只对当前会话或事务可见。每个会话只能查看和修改自己的数据。



## 临时表分类

会话级的临时表和事务级的临时表

**1）ON COMMIT DELETE ROWS**  事务

它是临时表的默认参数，表示临时表中的数据仅在事物过程（Transaction）中有效，当事物提交（COMMIT）后，临时表的暂时段将被自动截断（TRUNCATE），但是临时表的结构 以及元数据还存储在用户的数据字典中。如果临时表完成它的使命后，最好删除临时表，否则数据库会残留很多临时表的表结构和元数据。

**2）ON COMMIT PRESERVE ROWS** 会话

它表示临时表的内容可以跨事物而存在，不过，当该会话结束时，临时表的暂时段将随着会话的结束而被丢弃，临时表中的数据自然也就随之丢弃。但是临时表的结构以及元数据还存储在用户的数据字典中。如果临时表完成它的使命后，最好删除临时表，否则数据库会残留很多临时表的表结构和元数据。

### 会话级别

会话级的临时表的数据和你当前会话有关系，当前SESSION不退出的情况下，临时表中的数据就还存在，临时表的数据只有当你退出当前SESSION的时候才被截断（TRUNCATE TABLE）

**建表语句**

```sql
CREATE GLOBAL TEMPORARY TABLE TMP_TEST

(

　　　　ID NUMBER ,

　　　　NAME VARCHAR2(32)

) ON COMMIT PRESERVE ROWS;

或

CREATE GLOBAL TEMPORARY TABLE TMP_TEST ON COMMIT PRESERVE ROWS

AS

SELECT * FROM TEST;
```

**实例**

```sql
SQL> CREATE GLOBAL TEMPORARY TABLE TMP_TEST
(
 　　　　ID NUMBER ,
 　　　　NAME VARCHAR2(32)
 ) ON COMMIT PRESERVE ROWS;

Table created

SQL> INSERT INTO TMP_TEST

 　　SELECT 1, 'kerry' FROM DUAL;

1 row inserted

SQL> SELECT * FROM TMP_TEST;

ID 　　　　　　　　　　NAME

---------- ----------------------

1 　　　　　　　　　　kerry

SQL> COMMIT;

Commit complete
// Commit以及Rollback 不会截断会话级别的临时表数据

SQL> SELECT * FROM TMP_TEST;

ID 　　　　　　　　　　  NAME

---------- ------------------------
```

### 事务级别

```sql
CREATE GLOBAL TEMPORARY TABLE TMP_TEST

(

　　　　ID NUMBER ,

　　　　NAME VARCHAR2(32)

) ON COMMIT DELETE ROWS;

或

CREATE GLOBAL TEMPORARY TABLE TMP_TEST ON COMMIT DELETE AS SELECT * FROM TEST;

SQL> CREATE GLOBAL TEMPORARY TABLE TMP_TEST

 (

 　　　　ID NUMBER ,

 　　　　NAME VARCHAR2(32)

 ) ON COMMIT DELETE ROWS;

Table created

SQL> INSERT INTO TMP_TEST

　　 SELECT 1, 'kerry' FROM DUAL;

1 row inserted

SQL> SELECT * FROM TMP_TEST;

ID 　　　　　　　　　　NAME

---------- ---------------------

1 kerry

SQL> COMMIT;

Commit complete

SQL> SELECT * FROM TMP_TEST;

ID 　　　　　　　　　　NAME

---------- -----------------------
```



```sql
// 查看当前用户的表
select 
 table_name  
from 
user_tables;
```



## 临时表与永久表的区别

- 临时表是存储在临时表空间里面的
- 临时表的DML操作速度比较快