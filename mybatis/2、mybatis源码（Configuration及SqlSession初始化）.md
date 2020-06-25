

# Configuration初始化过程以及SqlSession的初始化

Mybatis将于数据库的交互封装成了一个会话，叫做SqlSession，而获取SqlSession一般通过SqlSessionFactory

```java
SqlSessionFactory.openSession();
```

```java
/**
 * 构建SqlSession的工厂.工厂模式
 *
 */
public interface SqlSessionFactory {

  //8个方法可以用来创建SqlSession实例
  SqlSession openSession();

  //自动提交
  SqlSession openSession(boolean autoCommit);
  //连接
  SqlSession openSession(Connection connection);
  //事务隔离级别
  SqlSession openSession(TransactionIsolationLevel level);

  //执行器的类型
  SqlSession openSession(ExecutorType execType);
  SqlSession openSession(ExecutorType execType, boolean autoCommit);
  SqlSession openSession(ExecutorType execType, TransactionIsolationLevel level);
  SqlSession openSession(ExecutorType execType, Connection connection);

  Configuration getConfiguration();

}
```



构建SqlSessionFactory有两种方式

### 从 XML 中构建 SqlSessionFactory

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

配置文件

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!--

       Copyright 2009-2012 the original author or authors.

       Licensed under the Apache License, Version 2.0 (the "License");
       you may not use this file except in compliance with the License.
       You may obtain a copy of the License at

          http://www.apache.org/licenses/LICENSE-2.0

       Unless required by applicable law or agreed to in writing, software
       distributed under the License is distributed on an "AS IS" BASIS,
       WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
       See the License for the specific language governing permissions and
       limitations under the License.

-->

<!DOCTYPE configuration
    PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
    "http://mybatis.org/dtd/mybatis-3-config.dtd">

<configuration>

  <properties resource="org/apache/ibatis/databases/blog/blog-derby.properties"/>

  <settings>
    <setting name="cacheEnabled" value="true"/>
    <setting name="lazyLoadingEnabled" value="false"/>
    <setting name="multipleResultSetsEnabled" value="true"/>
    <setting name="useColumnLabel" value="true"/>
    <setting name="useGeneratedKeys" value="false"/>
    <setting name="defaultExecutorType" value="SIMPLE"/>
    <setting name="defaultStatementTimeout" value="25"/>
  </settings>

  <typeAliases>
    <typeAlias alias="Author" type="org.apache.ibatis.domain.blog.Author"/>
    <typeAlias alias="Blog" type="org.apache.ibatis.domain.blog.Blog"/>
    <typeAlias alias="Comment" type="org.apache.ibatis.domain.blog.Comment"/>
    <typeAlias alias="Post" type="org.apache.ibatis.domain.blog.Post"/>
    <typeAlias alias="Section" type="org.apache.ibatis.domain.blog.Section"/>
    <typeAlias alias="Tag" type="org.apache.ibatis.domain.blog.Tag"/>
  </typeAliases>

  <typeHandlers>
    <typeHandler javaType="String" jdbcType="VARCHAR" handler="org.apache.ibatis.builder.ExampleTypeHandler"/>
  </typeHandlers>

  <objectFactory type="org.apache.ibatis.builder.ExampleObjectFactory">
    <property name="objectFactoryProperty" value="100"/>
  </objectFactory>

  <plugins>
    <plugin interceptor="org.apache.ibatis.builder.ExamplePlugin">
      <property name="pluginProperty" value="100"/>
    </plugin>
  </plugins>

  <environments default="development">
    <environment id="development">
      <transactionManager type="JDBC">
        <property name="" value=""/>
      </transactionManager>
      <dataSource type="UNPOOLED">
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>
        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>

  <mappers>
    <mapper resource="org/apache/ibatis/builder/AuthorMapper.xml"/>
    <mapper resource="org/apache/ibatis/builder/BlogMapper.xml"/>
    <mapper resource="org/apache/ibatis/builder/CachedAuthorMapper.xml"/>
    <mapper resource="org/apache/ibatis/builder/PostMapper.xml"/>
    <mapper resource="org/apache/ibatis/builder/NestedBlogMapper.xml"/>
  </mappers>

</configuration>

```

当执行**SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);**

的时候，会复用SqlSessionFactoryBuilder的Build方式构建SqlSessionFactory 

**这里使用了Xpath去解析mybatis.xml配置文件。**

```java
public SqlSessionFactory build(Reader reader) {
    return build(reader, null, null);
  }

//第4种方法是最常用的，它使用了一个参照了XML文档或更特定的SqlMapConfig.xml文件的Reader实例。
  //可选的参数是environment和properties。Environment决定加载哪种环境(开发环境/生产环境)，包括数据源和事务管理器。
  //如果使用properties，那么就会加载那些properties（属性配置文件），那些属性可以用${propName}语法形式多次用在配置文件中。和Spring很像，一个思想？
  public SqlSessionFactory build(Reader reader, String environment, Properties properties) {
    try {
        //委托XMLConfigBuilder来解析xml文件，并构建
      XMLConfigBuilder parser = new XMLConfigBuilder(reader, environment, properties);
      	//方法见下面
        return build(parser.parse());
    } catch (Exception e) {
        //这里是捕获异常，包装成自己的异常并抛出的idiom？，最后还要reset ErrorContext
      throw ExceptionFactory.wrapException("Error building SqlSession.", e);
    } finally {
      ErrorContext.instance().reset();
      try {
        reader.close();
      } catch (IOException e) {
        // Intentionally ignore. Prefer previous error.
      }
    }
  }
//基于Configuration创建一个SqlSessionFactory
//最后一个build方法使用了一个Configuration作为参数,并返回DefaultSqlSessionFactory
  public SqlSessionFactory build(Configuration config) {
    return new DefaultSqlSessionFactory(config);
  }
```

判断是否已经解析过，获取到配置文件的**configuration**节点，在mybatis配置文件中，**configuration**节点就是配置内容的根节点。

```java
//解析配置
  public Configuration parse() {
    //如果已经解析过了，报错
    if (parsed) {
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
    
    //根节点是configuration
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
  }
```

**调用parseConfiguration方法解析配置文件中的各个节点信息**。其中包括

- properties（属性）
- setting(设置)
- typeAliases 类型别名
- plugin 插件
- objectFactory 对象工厂
- bjectWrapperFactory 对象包装工厂
- environments 环境
  - environments 环境变量
    - transactionManager 	事务管理器
    - dataSource 数据源
- databaseIdProvider 数据库厂商标识
- typeHandler 类型处理器
- mapper 映射器

```java
//解析配置
private void parseConfiguration(XNode root) {
  try {
    //分步骤解析
    //issue #117 read properties first
    //1.properties
    propertiesElement(root.evalNode("properties"));
    //2.类型别名
    typeAliasesElement(root.evalNode("typeAliases"));
    //3.插件
    pluginElement(root.evalNode("plugins"));
    //4.对象工厂
    objectFactoryElement(root.evalNode("objectFactory"));
    //5.对象包装工厂
    objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
    //6.设置
    settingsElement(root.evalNode("settings"));
    // read it after objectFactory and objectWrapperFactory issue #631
    //7.环境
    environmentsElement(root.evalNode("environments"));
    //8.databaseIdProvider
    databaseIdProviderElement(root.evalNode("databaseIdProvider"));
    //9.类型处理器
    typeHandlerElement(root.evalNode("typeHandlers"));
    //10.映射器
    mapperElement(root.evalNode("mappers"));
  } catch (Exception e) {
    throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
  }
}
```



**environmentsElement(root.evalNode("environments"));**

这里是初始化一个**Environment**，**Environment**中设置**事务管理器（transactionManager）**和**数据源（dataSource）**向最后向**configuration**设置。

```java
private void environmentsElement(XNode context) throws Exception {
  if (context != null) {
    if (environment == null) {
      environment = context.getStringAttribute("default");
    }
    for (XNode child : context.getChildren()) {
      String id = child.getStringAttribute("id");
//循环比较id是否就是指定的environment
      if (isSpecifiedEnvironment(id)) {
        //7.1事务管理器
        TransactionFactory txFactory = transactionManagerElement(child.evalNode("transactionManager"));
        //7.2数据源
        DataSourceFactory dsFactory = dataSourceElement(child.evalNode("dataSource"));
        DataSource dataSource = dsFactory.getDataSource();
        Environment.Builder environmentBuilder = new Environment.Builder(id)
            .transactionFactory(txFactory)
            .dataSource(dataSource);
        .setEnvironment(environmentBuilder.build());
      }
    }
  }
}

private TransactionFactory transactionManagerElement(XNode context) throws Exception {
    if (context != null) {
      String type = context.getStringAttribute("type");
      Properties props = context.getChildrenAsProperties();
		//根据type="JDBC"解析返回适当的TransactionFactory
      TransactionFactory factory = (TransactionFactory) resolveClass(type).newInstance();
      factory.setProperties(props);
      return factory;
    }
    throw new BuilderException("Environment declaration requires a TransactionFactory.");
  }

private DataSourceFactory dataSourceElement(XNode context) throws Exception {
    if (context != null) {
      String type = context.getStringAttribute("type");
      Properties props = context.getChildrenAsProperties();
		//根据type="POOLED"解析返回适当的DataSourceFactory
      DataSourceFactory factory = (DataSourceFactory) resolveClass(type).newInstance();
      factory.setProperties(props);
      return factory;
    }
    throw new BuilderException("Environment declaration requires a DataSourceFactory.");
  }
```



**构建映射器，将所有的Mapper添加到configuration 中**

**构建映射器，实际就是将映射器的名称和映射器的实例添加到一个Map中**

 //将已经添加的映射都放入HashMap
  private final Map<Class<?>, MapperProxyFactory<?>> **knownMappers** = new HashMap<Class<?>, MapperProxyFactory<?>>();

构建映射器分为以下四种方式：每一种解析方式都是获取到对应的Mapper实例，每一个Mapper实质上都是一个接口，但是mybatis会通过**MapperProxyFactory**，生成一个代理对象

```java
 knownMappers.put(type, new MapperProxyFactory<T>(type));
```



```
10.映射器
10.1使用类路径
<mappers>
  <mapper resource="org/mybatis/builder/AuthorMapper.xml"/>
  <mapper resource="org/mybatis/builder/BlogMapper.xml"/>
  <mapper resource="org/mybatis/builder/PostMapper.xml"/>
</mappers>

10.2使用绝对url路径
<mappers>
  <mapper url="file:///var/mappers/AuthorMapper.xml"/>
  <mapper url="file:///var/mappers/BlogMapper.xml"/>
  <mapper url="file:///var/mappers/PostMapper.xml"/>
</mappers>

10.3使用java类名
<mappers>
  <mapper class="org.mybatis.builder.AuthorMapper"/>
  <mapper class="org.mybatis.builder.BlogMapper"/>
  <mapper class="org.mybatis.builder.PostMapper"/>
</mappers>

10.4自动扫描包下所有映射器
<mappers>
  <package name="org.mybatis.builder"/>
</mappers>
```

对应的解析代码如下

```java
private void mapperElement(XNode parent) throws Exception {
    if (parent != null) {
      for (XNode child : parent.getChildren()) {
        if ("package".equals(child.getName())) {
          //10.4自动扫描包下所有映射器
          String mapperPackage = child.getStringAttribute("name");
          configuration.addMappers(mapperPackage);
        } else {
          String resource = child.getStringAttribute("resource");
          String url = child.getStringAttribute("url");
          String mapperClass = child.getStringAttribute("class");
          if (resource != null && url == null && mapperClass == null) {
            //10.1使用类路径
            ErrorContext.instance().resource(resource);
            InputStream inputStream = Resources.getResourceAsStream(resource);
            //映射器比较复杂，调用XMLMapperBuilder
            //注意在for循环里每个mapper都重新new一个XMLMapperBuilder，来解析
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, resource, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url != null && mapperClass == null) {
            //10.2使用绝对url路径
            ErrorContext.instance().resource(url);
            InputStream inputStream = Resources.getUrlAsStream(url);
            //映射器比较复杂，调用XMLMapperBuilder
            XMLMapperBuilder mapperParser = new XMLMapperBuilder(inputStream, configuration, url, configuration.getSqlFragments());
            mapperParser.parse();
          } else if (resource == null && url == null && mapperClass != null) {
            //10.3使用java类名
            Class<?> mapperInterface = Resources.classForName(mapperClass);
            //直接把这个映射加入配置
            configuration.addMapper(mapperInterface);
          } else {
            throw new BuilderException("A mapper element may only specify a url, resource or class, but not more than one.");
          }
        }
      }
    }
  }
```

