# 空值处理：分不清楚的null和恼人的空指针

## NullPointerException 是 Java 代码中最常见的异常，我将其最可能出现的场景归为以下 5 种

- 参数值是 Integer 等包装类型，使用时因为自动拆箱出现了空指针异常；
- 字符串比较出现空指针异常；
- 诸如 ConcurrentHashMap 这样的容器不支持 Key 和 Value 为 null，强行 put null 的 Key 或 Value 会出现空指针异常；
- A 对象包含了 B，在通过 A 对象的字段获得 B 之后，没有对字段判空就级联调用 B 的方法出现空指针异常；
- 方法或远程服务返回的 List 不是空而是 null，没有进行判空就直接调用 List 的方法出现空指针异常。

## 模拟NullPointerException

```java

private List<String> wrongMethod(FooService fooService, Integer i, String s, String t) {
    log.info("result {} {} {} {}", i + 1, s.equals("OK"), s.equals(t),
            new ConcurrentHashMap<String, String>().put(null, null));
    if (fooService.getBarService().bar().equals("OK"))
        log.info("OK");
    return null;
}

@GetMapping("wrong")
public int wrong(@RequestParam(value = "test", defaultValue = "1111") String test) {
    return wrongMethod(test.charAt(0) == '1' ? null : new FooService(),
            test.charAt(1) == '1' ? null : 1,
            test.charAt(2) == '1' ? null : "OK",
            test.charAt(3) == '1' ? null : "OK").size();
}

class FooService {
    @Getter
    private BarService barService;

}

class BarService {
    String bar() {
        return "OK";
    }
}
```

解决方法：

- 对于 Integer 的判空，可以使用 Optional.ofNullable 来构造一个 Optional，然后使用 orElse(0) 把 null 替换为默认值再进行 +1 操作。
- 对于 String 和字面量的比较，可以把字面量放在前面，比如"OK".equals(s)，这样即使 s 是 null 也不会出现空指针异常；
- 而对于两个可能为 null 的字符串变量的 equals 比较，可以使用 Objects.equals，它会做判空处理（需要a对象重写equals方法）。
- 对于 ConcurrentHashMap，既然其 Key 和 Value 都不支持 null，修复方式就是不要把 null 存进去。HashMap 的 Key 和 Value 可以存入 null，而 ConcurrentHashMap 看似是 HashMap 的线程安全版本，却不支持 null 值的 Key 和 Value，这是容易产生误区的一个地方。
- 对于类似 fooService.getBarService().bar().equals(“OK”) 的级联调用，需要判空的地方有很多，包括 fooService、getBarService() 方法的返回值，以及 bar 方法返回的字符串。如果使用 if-else 来判空的话可能需要好几行代码，但使用 Optional 的话一行代码就够了。对于 rightMethod 返回的 List，由于不能确认其是否为 null，所以在调用 size 方法获得列表大小之前，同样可以使用 Optional.ofNullable 包装一下返回值，然后通过.orElse(Collections.emptyList()) 实现在 List 为 null 的时候获得一个空的 List，最后再调用 size 方法。



```java

private List<String> rightMethod(FooService fooService, Integer i, String s, String t) {
    log.info("result {} {} {} {}", Optional.ofNullable(i).orElse(0) + 1, "OK".equals(s), Objects.equals(s, t), new HashMap<String, String>().put(null, null));
    Optional.ofNullable(fooService)
            .map(FooService::getBarService)
            .filter(barService -> "OK".equals(barService.bar()))
            .ifPresent(result -> log.info("OK"));
    return new ArrayList<>();
}

@GetMapping("right")
public int right(@RequestParam(value = "test", defaultValue = "1111") String test) {
    return Optional.ofNullable(rightMethod(test.charAt(0) == '1' ? null : new FooService(),
            test.charAt(1) == '1' ? null : 1,
            test.charAt(2) == '1' ? null : "OK",
            test.charAt(3) == '1' ? null : "OK"))
            .orElse(Collections.emptyList()).size();
}
```



## POJO 中属性的 null 到底代表了什么？

需要明确以下几点

- DTO 中字段的 null 到底意味着什么？是客户端没有传给我们这个信息吗？
- DTO 中的字段要设置默认值么？
- 如果数据库实体中的字段有 null，那么通过数据访问框架保存数据是否会覆盖数据库中的既有数据？



**总结一句话**

**在写业务代码时注意判断POJO中的Null，如果操作数据库的框架有这个功能就更好。**

**如Mybatis ，写xml使用动态sql，使用if标签判断是否为空。**

**在Mybatis中拦截判断POJO类的null字段，过滤掉null的字段，即不操作为null的字段。**

## 小心 MySQL 中有关 NULL 的三个坑

前面提到，数据库表字段允许存 NULL 除了会让我们困惑外，还容易有坑。这里我会结合 NULL 字段，和你着重说明 sum 函数、count 函数，以及 NULL 值条件可能踩的坑

通过 sum 函数统计一个只有 NULL 值的列的总和，比如 SUM(score)；select 记录数量，count 使用一个允许 NULL 的字段，比如 COUNT(score)；使用 =NULL 条件查询字段值为 NULL 的记录，比如 score=null 条件

```java

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    @Query(nativeQuery=true,value = "SELECT SUM(score) FROM `user`")
    Long wrong1();
    @Query(nativeQuery = true, value = "SELECT COUNT(score) FROM `user`")
    Long wrong2();
    @Query(nativeQuery = true, value = "SELECT * FROM `user` WHERE score=null")
    List<User> wrong3();
}
```

结果：

```
	
[11:38:50.137] [http-nio-45678-exec-1] [INFO ] [t.c.nullvalue.demo3.DbNullController:26  ] - result: null 0 [] 
```

原因：

- MySQL 中 sum 函数没统计到任何记录时，会返回 null 而不是 0，可以使用 IFNULL 函数把 null 转换为 0
- MySQL 中 count 字段不统计 null 值，COUNT(*) 才是统计所有记录数量的正确方式。（区分是否要统计包含Null的行数）
- MySQL 中使用诸如 =、<、> 这样的算数比较操作符比较 NULL 的结果总是 NULL，这种比较就显得没有任何意义，需要使用 IS NULL、IS NOT NULL 或 ISNULL() 函数来比较。

```java

@Query(nativeQuery = true, value = "SELECT IFNULL(SUM(score),0) FROM `user`")
Long right1();
@Query(nativeQuery = true, value = "SELECT COUNT(*) FROM `user`")
Long right2();
@Query(nativeQuery = true, value = "SELECT * FROM `user` WHERE score IS NULL")
List<User> right3();
```

