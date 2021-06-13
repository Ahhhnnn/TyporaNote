# 使用Docker安装oracle-12c

## 安装

```shell
docker search oracle | grep 12c
docker pull truevoly/oracle-12c
```

![image-20210404161558974](/Users/hening/Library/Application Support/typora-user-images/image-20210404161558974.png)



## 启动oracle12c

```shell
docker run -d -p 8080:8080 -p 1521:1521  truevoly/oracle-12c
// 查看日志
docker logs -f 容器id
```

![image-20210404161633873](/Users/hening/Library/Application Support/typora-user-images/image-20210404161633873.png)



## 进入容器

```shell
docker exec -it 3b8f3cdf1009 /bin/bash
$ORACLE_HOME/bin/sqlplus / as sysdba
select status from v$instance;
```

![image-20210404163641377](/Users/hening/Library/Application Support/typora-user-images/image-20210404163641377.png)



## 创建用户

### 创建表空间

```sql
 /*创建表空间*/
  create tablespace TABLESPACE_QCJ
  /*表空间物理文件名称*/
  datafile 'TABLESPACE_QCJ.dbf' 
  -- 这种方式指定表空间物理文件位置
  -- datafile 'F:\APP\QIUCHANGJIN\ORADATA\ORCL\TABLESPACE_QCJ.dbf' 
  -- 大小 500M，每次 5M 自动增大，最大不限制
  size 500M autoextend on next 5M maxsize unlimited;

create tablespace TABLESPACE_HE datafile 'TABLESPACE_HE.dbf' size 50M autoextend on next 5M maxsize unlimited;
```

### 新建用户

```sql
-- 新建用户`TEST`并选择刚创建的表空间 `BKJ`
CREATE USER HENING IDENTIFIED BY  123456 ACCOUNT UNLOCK DEFAULT TABLESPACE TABLESPACE_HE;
```

### 授权

```sql
-- connect,resource,dba权限赋予 test用户
GRANT CONNECT,RESOURCE,DBA TO HENING;

-- 多权限授权
GRANT CREATE USER,DROP USER,ALTER USER ,CREATE ANY VIEW ,
DROP ANY VIEW,EXP_FULL_DATABASE,IMP_FULL_DATABASE,
DBA,CONNECT,RESOURCE,CREATE SESSION TO TEST;
```

![image-20210404164439665](/Users/hening/Library/Application Support/typora-user-images/image-20210404164439665.png)

### 常用命令

```sql
1. Oracle的服务名(ServiceName)查询
SQL> show parameter service_name;

2. Oracle的SID查询命令：
SQL> select instance_name from v$instance;

3. 查看Oracle版本
SQL> select version from v$instance

/*查询所有表空间物理位置*/
select name from v$datafile;
/*查询当前用户的表空间*/
select username,default_tablespace from user_users;
/*修改用户的默认表空间*/
alter user 用户名 default tablespace 新表空间; 
/*查询所有的表空间*/
select * from user_tablespaces; 

```

## 连接Oracle

![image-20210404175002317](/Users/hening/Library/Application Support/typora-user-images/image-20210404175002317.png)

![image-20210404175026027](/Users/hening/Library/Application Support/typora-user-images/image-20210404175026027.png)

