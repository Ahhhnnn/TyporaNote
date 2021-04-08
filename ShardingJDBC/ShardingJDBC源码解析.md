# 源码解析

分析入手点

通过打出来的日志分析，发现会有一个专门打日志的类 **SQLLogger**

**通过打断点，找到对应shardingjdbc处理的对应逻辑**

![image-20210407214448365](/Users/hening/Library/Application Support/typora-user-images/image-20210407214448365.png)

## query

查询就是根据分库分表规则，根据逻辑表名，生成多个select sql语句，找出笛卡尔积，然后汇总返回。

比如分片规则为 **ds$->{0..1}.t_order_$->{0..1}**

那么 select  * from t_order 表最终会变为四条sql语句

```sql
select  * form ds0.t_order_0;
select  * form ds0.t_order_1;
select  * form ds1.t_order_0;
select  * form ds1.t_order_1;
```

源码入口在于使用shardingjdbc的 datasource去执行了真正的sql语句。如果与**mybatis**整合的话，那么在mybatis执行sql语句获取连接的时候，拿到的是shardingjdbc的**ShardingDataSource** 和 **ShardingConnection**



首先在项目启动的时候由sharding-jdbc-spring-boot-starter（**SpringBootConfiguration**）自动注入shardingdatasource

**期间会读取properties文件配置，获取数据源，解析分片配置，获取分片策略等**

![image-20210407215233416](/Users/hening/Library/Application Support/typora-user-images/image-20210407215233416.png)

当执行到真正的sql语句时，是有mybatis的**SimpleExecutor.doQuery** 方法执行的

![image-20210407215537155](/Users/hening/Library/Application Support/typora-user-images/image-20210407215537155.png)

然后获取Connection和PrepareStatement

![image-20210407215702660](/Users/hening/Library/Application Support/typora-user-images/image-20210407215702660.png)

之后由**ShardingJdbc接管connection，执行分片逻辑**

![image-20210407215848777](/Users/hening/Library/Application Support/typora-user-images/image-20210407215848777.png)

然后由sharding分片引擎进行处理（**BaseShardingEngine**），对sql进行**路由** 以及 **sql重写**

![image-20210407220112326](/Users/hening/Library/Application Support/typora-user-images/image-20210407220112326.png)

**sql重写**

![image-20210407220542439](/Users/hening/Library/Application Support/typora-user-images/image-20210407220542439.png)

![image-20210407220804556](/Users/hening/Library/Application Support/typora-user-images/image-20210407220804556.png)

之后将sql交由peraredstatment 执行，然后交由mybatis的结果处理器返回

## insert

跟query前面步骤差不多,同样是注入ShardingDatasource，然后mybatis执行的时候获取到shardingjdbc的connection（**这个时候拿到的connection并不是真正的连接，里面封装了datasourcemap，配置文件中的多个数据源，目前还不知道路由到具体哪个库中**）和statement，

然后执行update方法，由shardingjdbc接管，根据配置文件中的分片策略，如果是inline那么就会走入**InlineShardingStrategy**的**dosharding**方法进行分片，根据分片键，找到对应的**实际数据库和实际表**，接着调用**preparedStatementExecutor.execute()**

执行真正的sql语句。

**ShardingPreparedStatement.execute()**

![image-20210407232730502](/Users/hening/Library/Application Support/typora-user-images/image-20210407232730502.png)

**分片策略InlineShardingStrategy**

**通过dosharding方法，找出实际库（ds0 or ds1） 实际表（t_order_0 or t_order_1）**

![image-20210407232855438](/Users/hening/Library/Application Support/typora-user-images/image-20210407232855438.png)

如果是stander策略，那么需要自己实现 **PreciseShardingAlgorithm**接口（必须）或者**RangeShardingAlgorithm**接口

![image-20210407233327725](/Users/hening/Library/Application Support/typora-user-images/image-20210407233327725.png)

