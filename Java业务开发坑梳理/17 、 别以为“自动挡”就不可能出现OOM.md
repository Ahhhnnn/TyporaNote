# 17 | 别以为“自动挡”就不可能出现OOM

## 建议配置

```log
-XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=. -XX:+PrintGCDateStamps -XX:+PrintGCDetails -Xloggc:gc.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=10 -XX:GCLogFileSize=100M
```

- -XX:-HeapDumpOnOutOfMemoryError : OOM导出堆内存
- -XX:HeapDumpPath ：堆内存导出路径
- -XX:+PrintGCDateStamps ：GC日志信息
- -XX:+PrintGCDetails  ：GC详细日志
-  -Xloggc:gc.log ：日志名称
- -XX:+UseGCLogFileRotation ： 循环记录GC日志，当10个日志都记录到100M后，将新的日志记录到第一个日志中
- -XX:NumberOfGCLogFiles=10 ： GC日志数量
-  -XX:GCLogFileSize=100M GC文件大小

```
-XX:+PrintGCDetails 
-XX:+PrintGCDateStamps 
-Xloggc:/home/GCEASY/gc-%t.log
```

- %t 可以给日志文件名称加上时间戳 格式是YYYY-MM-DD_HH-MM-SS，避免日志文件覆盖问题



## 太多份相同的对象导致 OOM

对于快速检索的需求，最好使用 Map 来实现，会比直接从 List 搜索快得多。

值得注意的是，我们虽然清楚数据总量，但却忽略了每一份数据在内存中可能有多份。我之前还遇到一个案例，一个后台程序需要从数据库加载大量信息用于数据导出，这些数据在数据库中占用 100M 内存，但是 1GB 的 JVM 堆却无法完成导出操作。我来和你分析下原因吧。100M 的数据加载到程序内存中，变为 Java 的数据结构就已经占用了 200M 堆内存；这些数据经过 JDBC、MyBatis 等框架其实是加载了 2 份，然后领域模型、DTO 再进行转换可能又加载了 2 次；最终，占用的内存达到了 200M*4=800M。

**所以，在进行容量评估时，我们不能认为一份数据在程序内存中也是一份，有可能会存在多份**

## 使用 WeakHashMap 不等于不会 OOM

对于上一节实现快速检索的案例，为了防止缓存中堆积大量数据导致 OOM，一些同学可能会想到使用 WeakHashMap 作为缓存容器。WeakHashMap 的特点是 Key 在哈希表内部是弱引用的，当没有强引用指向这个 Key 之后，Entry 会被 GC，即使我们无限往 WeakHashMap 加入数据，只要 Key 不再使用，也就不会 OOM

说到了强引用和弱引用，我先和你回顾下 Java 中引用类型和垃圾回收的关系：垃圾回收器不会回收有强引用的对象；在内存充足时，垃圾回收器不会回收具有软引用的对象；垃圾回收器只要扫描到了具有弱引用的对象就会回收，WeakHashMap 就是利用了这个特点。

**注意：**

如果WeakHashMap  value强引用了key，那么也不会进行回收

WeakHashMap 使用的Entry和HashMap不同

```java

private static class Entry<K,V> extends WeakReference<Object> ...
/**
 * Creates new entry.
 */
Entry(Object key, V value,
      ReferenceQueue<Object> queue,
      int hash, Entry<K,V> next) {
    super(key, queue);
    this.value = value;
    this.hash  = hash;
    this.next  = next;
}
```

Entry 对象继承了 WeakReference，Entry 的构造函数调用了 super (key,queue)，这是父类的构造函数。其中，key 是我们执行 put 方法时的 key；**queue 是一个 ReferenceQueue。如果你了解 Java 的引用就会知道，被 GC 的对象会被丢进这个 queue 里面。**

**key是弱引用，那value是什么时候回收的呢？**

```java

public V get(Object key) {
    Object k = maskNull(key);
    int h = hash(k);
    Entry<K,V>[] tab = getTable();
    int index = indexFor(h, tab.length);
    Entry<K,V> e = tab[index];
    while (e != null) {
        if (e.hash == h && eq(k, e.get()))
            return e.value;
        e = e.next;
    }
    return null;
}

private Entry<K,V>[] getTable() {
    expungeStaleEntries();
    return table;
}

/**
 * Expunges stale entries from the table.
 */
private void expungeStaleEntries() {
    for (Object x; (x = queue.poll()) != null; ) {
        synchronized (queue) {
            @SuppressWarnings("unchecked")
                Entry<K,V> e = (Entry<K,V>) x;
            int i = indexFor(e.hash, table.length);

            Entry<K,V> prev = table[i];
            Entry<K,V> p = prev;
            while (p != null) {
                Entry<K,V> next = p.next;
                if (p == e) {
                    if (prev == e)
                        table[i] = next;
                    else
                        prev.next = next;
                    // Must not null out e.next;
                    // stale entries may be in use by a HashIterator
                    e.value = null; // Help GC
                    size--;
                    break;
                }
                prev = p;
    ``            p = next;
            }
        }
    }
}
```

从源码中可以看到，每次调用 get、put、size 等方法时，都会从 queue 里拿出所有已经被 GC 掉的 key 并删除对应的 Entry 对象。我们再来回顾下这个逻辑：

- put 一个对象进 Map 时，它的 key 会被封装成弱引用对象；
- 发生 GC 时，弱引用的 key 被发现并放入 queue；
- 调用 get 等方法时，扫描 queue 删除 key，以及包含 key 和 value 的 Entry 对象。



WeakHashMap 的 Key 虽然是弱引用，但是其 Value 却持有 Key 中对象的强引用，Value 被 Entry 引用，Entry 被 WeakHashMap 引用，最终导致 Key 无法回收

解决方法：

1、value使用新的对象，不强引用key值

2、使用WeakReference来包装一次 value对象

```java

private Map<User, WeakReference<UserProfile>> cache2 = new WeakHashMap<>();

@GetMapping("right")
public void right() {
    String userName = "zhuye";
    //间隔1秒定时输出缓存中的条目数
    Executors.newSingleThreadScheduledExecutor().scheduleAtFixedRate(
            () -> log.info("cache size:{}", cache2.size()), 1, 1, TimeUnit.SECONDS);
    LongStream.rangeClosed(1, 2000000).forEach(i -> {
        User user = new User(userName + i);
        //这次，我们使用弱引用来包装UserProfile
        cache2.put(user, new WeakReference(new UserProfile(user, "location" + i)));
    });
}
```



## 参数不合理导致

一定要根据实际需求来修改参数配置，可以考虑预留 2 到 5 倍的量。容量类的参数背后往往代表了资源，设置超大的参数就有可能占用不必要的资源，在并发量大的时候因为资源大量分配导致 OOM。

如tomcat 的 工作线程的请求头参数可以进行配置大小,每次来一个请求，tomcat创建一个工作线程，都会分配最大的内存给工作线程，导致在高并发下出现OOM。

```properties
server.max-http-header-size=10000000
```



## 重点回顾

一是，我们的程序确实需要超出 JVM 配置的内存上限的内存。不管是程序实现的不合理，还是因为各种框架对数据的重复处理、加工和转换，相同的数据在内存中不一定只占用一份空间。针对内存量使用超大的业务逻辑，比如缓存逻辑、文件上传下载和导出逻辑，我们在做容量评估时，可能还需要实际做一下 Dump，而不是进行简单的假设。

二是，出现内存泄露，其实就是我们认为没有用的对象最终会被 GC，但却没有。GC 并不会回收强引用对象，我们可能经常在程序中定义一些容器作为缓存，但如果容器中的数据无限增长，要特别小心最终会导致 OOM。使用 WeakHashMap 是解决这个问题的好办法，但值得注意的是，如果强引用的 Value 有引用 Key，也无法回收 Entry。

三是，不合理的资源需求配置，在业务量小的时候可能不会出现问题，但业务量一大可能很快就会撑爆内存。比如，随意配置 Tomcat 的 max-http-header-size 参数，会导致一个请求使用过多的内存，请求量大的时候出现 OOM。在进行参数配置的时候，我们要认识到，很多限制类参数限制的是背后资源的使用，资源始终是有限的，需要根据实际需求来合理设置参数。





