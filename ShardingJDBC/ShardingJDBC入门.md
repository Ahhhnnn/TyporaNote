# ShardingJDBC入门

官网地址：https://shardingsphere.apache.org/document/legacy/4.x/document/cn/manual/sharding-jdbc/configuration/config-spring-boot/#%E8%AF%BB%E5%86%99%E5%88%86%E7%A6%BB-1

**注意选择版本**

## 基本概念

### ShardingJDBC执行流程

![image-20210327233926796](/Users/hening/Library/Application Support/typora-user-images/image-20210327233926796.png)

## 主库

添加、更新以及删除数据操作所使用的数据库，目前仅支持单主库。

## 从库

查询数据操作所使用的数据库，可支持多从库。

## 主从同步

将主库的数据异步的同步到从库的操作。由于主从同步的异步性，从库与主库的数据会短时间内不一致。

## 负载均衡策略

通过负载均衡策略将查询请求疏导至不同从库。

## 逻辑表

水平拆分的数据库（表）的相同逻辑和数据结构表的总称。例：订单数据根据主键尾数拆分为10张表，分别是`t_order_0`到`t_order_9`，他们的逻辑表名为`t_order`。

## 真实表

在分片的数据库中真实存在的物理表。即上个示例中的`t_order_0`到`t_order_9`。

## 数据节点

数据分片的最小单元。由数据源名称和数据表组成，例：`ds_0.t_order_0`。



## 分库分表定义

分为数据源分片策略和数据表分片策略

根据分片键及分片算法决定数据落到那个数据源以及那一个数据表

![image-20210327234415233](/Users/hening/Library/Application Support/typora-user-images/image-20210327234415233.png)

## 读写分离实践操作

新建SpringBoot项目，整合mysql mybatis shardingjdbc druid

![image-20210331221719367](/Users/hening/Library/Application Support/typora-user-images/image-20210331221719367.png)

```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
            <version>2.1.4</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.16</version>
        </dependency>
        <dependency>
            <groupId>org.apache.shardingsphere</groupId>
            <artifactId>sharding-jdbc-spring-boot-starter</artifactId>
            <version>4.0.0-RC1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.shardingsphere</groupId>
            <artifactId>sharding-core-common</artifactId>
            <version>4.0.0-RC1</version>
        </dependency>
        <!-- hutool 工具类 -->
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-setting</artifactId>
            <version>5.2.4</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <scope>runtime</scope>
        </dependency>
```

配置文件

```properties
server.port=8080

# 配置真实数据源
spring.shardingsphere.datasource.names=ds0,ds1
spring.main.allow-bean-definition-overriding=true

# 配置第 1 个数据源
spring.shardingsphere.datasource.ds0.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.ds0.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.ds0.url=jdbc:mysql://47.100.214.147:3307/20210326?useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&serverTimezone=Asia/Shanghai
spring.shardingsphere.datasource.ds0.username=root
spring.shardingsphere.datasource.ds0.password=123456

# 配置第 2 个数据源
spring.shardingsphere.datasource.ds1.type=com.alibaba.druid.pool.DruidDataSource
spring.shardingsphere.datasource.ds1.driver-class-name=com.mysql.cj.jdbc.Driver
spring.shardingsphere.datasource.ds1.url=jdbc:mysql://47.100.214.147:3308/20210326?useUnicode=true&characterEncoding=utf-8&allowMultiQueries=true&serverTimezone=Asia/Shanghai
spring.shardingsphere.datasource.ds1.username=root
spring.shardingsphere.datasource.ds1.password=123456
# 打印sql
spring.shardingsphere.props.sql.show=true
# 默认数据源
spring.shardingsphere.sharding.default-data-source-name=ds0
# 读的随机算法
spring.shardingsphere.masterslave.load-balance-algorithm-type=round_robin
# 数据源名称 随意取
spring.shardingsphere.masterslave.name=ms
# 主库
spring.shardingsphere.masterslave.master-data-source-name=ds0
# 从库
spring.shardingsphere.masterslave.slave-data-source-names=ds1

mybatis.mapper-locations=classpath:mapper/*.xml
mybatis.type-aliases-package=com.he.sharding.entity
```

### 插入数据

走的主库 ds0

![image-20210331221505192](/Users/hening/Library/Application Support/typora-user-images/image-20210331221505192.png)

### 查询数据

走的从库ds1

![image-20210331221555121](/Users/hening/Library/Application Support/typora-user-images/image-20210331221555121.png)



## 数据分片实践操作

### 仅分表（同一数据库）

创建数据库

```sql
CREATE TABLE `t_order_0` (
  `id` bigint(11) NOT NULL,
  `user_id` int(11) DEFAULT NULL,
  `order_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
CREATE TABLE `t_order_1` (
  `id` bigint(11) NOT NULL,
  `user_id` int(11) DEFAULT NULL,
  `order_id` int(11) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COLLATE=utf8_bin;
```



![image-20210401223450369](/Users/hening/Library/Application Support/typora-user-images/image-20210401223450369.png)

配置文件

```properties
spring.shardingsphere.sharding.tables.t_order.actual-data-nodes=ds0.t_order_$->{0..1}
spring.shardingsphere.sharding.tables.t_order.table-strategy.inline.sharding-column=order_id
spring.shardingsphere.sharding.tables.t_order.table-strategy.inline.algorithm-expression=t_order_$->{order_id % 2}
```

编写mapper 进行插入order，期间变换order_id，观察入库情况

![image-20210401223625557](/Users/hening/Library/Application Support/typora-user-images/image-20210401223625557.png)

**发现order_id 为1 （1%2=1）和 2 （2%2=0）的数据分别落到了 t_order_1 和 t_order_0 中**



## 分库分表

创建数据库，数据表

**两个数据库中数据表必须一致(之前做了主从同步，现在自动就有了环境)**

![image-20210401223836834](/Users/hening/Library/Application Support/typora-user-images/image-20210401223836834.png)

配置文件

```properties

```

编写mapper插入数据

**根据路由规则order_id =5 %2 =1 进入t_order_1  user_id = 1001 %2 =1 进入ds0 库** （其他库和表中没有这条数据，说明分库分表成功）

![image-20210401224516908](/Users/hening/Library/Application Support/typora-user-images/image-20210401224516908.png)

![image-20210401224618415](/Users/hening/Library/Application Support/typora-user-images/image-20210401224618415.png)



查询数据

**在mapper中直接编写sql，注意写逻辑表（注意配置文件中表名配置正确！！！）**

查询时会根据配置文件中的（ds$->{0..1}.t_order_$->{0..1}） 表达式，**求出数据的笛卡尔积**

![image-20210401230447875](/Users/hening/Library/Application Support/typora-user-images/image-20210401230447875.png)



## 分片算法



## 分片策略

常用的两种

### 行表达式分片策略

对应InlineShardingStrategy。使用Groovy的表达式，提供对SQL语句中的=和IN的分片操作支持，只支持单分片键。对于简单的分片算法，可以通过简单的配置使用，从而避免繁琐的Java代码开发，如: `t_user_$->{u_id % 8}` 表示t_user表根据u_id模8，而分成8张表，表名称为`t_user_0`到`t_user_7`。

### 标准分片策略

对应StandardShardingStrategy。提供对SQL语句中的=, >, <, >=, <=, IN和BETWEEN AND的分片操作支持。StandardShardingStrategy只支持单分片键，提供PreciseShardingAlgorithm和RangeShardingAlgorithm两个分片算法。PreciseShardingAlgorithm是必选的，用于处理=和IN的分片。RangeShardingAlgorithm是可选的，用于处理BETWEEN AND, >, <, >=, <=分片，如果不配置RangeShardingAlgorithm，SQL中的BETWEEN AND将按照全库路由处理。

