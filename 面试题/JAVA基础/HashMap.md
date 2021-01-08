

### 静态内部类 Node

实现了Map.Entry<K,V>，重写了hashcode方法，为key的hashcode 与 value的hashcode 进行异或操作

```java
static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```



### put操作

put操作实际上是将key - value存到了一个Node中，node的hash属性存的是 key的hash值。

首先判断table（即一个Node数组）是否为空，如果为空，先进行开辟空间操作，执行resize()方法

```java
//该表在首次使用时初始化，并根据需要调整大小。分配时，长度始终是2的幂
// transient关键字：将不需要序列化的属性前添加关键字transient，序列化对象的时候，这个属性就不会被序列化
// 1、transient关键字修饰的属性，生命周期只会在内存中，不会持久化到内存
// 2、 被transient关键字修饰过得变量，如果是通过实现Exteranlizable接口，重写writeExternal和readExternal方法，那么可以进行序列化
// 3、被static修饰的静态变量，天生不会被序列化。因为static修饰的变量存在方法区。序列化只能虚化列化堆内存
transient Node<K,V>[] table;
```

resize 方法根据设置的初始化大小（默认16，扩容因子0.75）初始化map。

记录下map下次的扩容点（newThr，当前map的大小>newThr时，再次执行resize方法进行扩容）

将 hash值、key、value赋值给Node，设置到Node数组中即完成了put操作。

# 面试题

##  数据结构

HashMap的实现框架都是**哈希表 + 链表**的组合方式

![image-20200816125502080](assets/image-20200816125502080.png)



## put方法

put方法的处理过程，可拆分为四部分

**part1：特殊key值处理，key为null；**

-  HashMap中，是允许key、value都为null的，且key为null只存一份，多次存储会将旧value值覆盖
-  key为null的存储位置，都统一放在下标为**0**的bucket，即：table[0]位置的链表
- 如果是第一次对key=null做put操作，将会在table[0]的位置新增一个Entry结点，使用头插法做链表插入

**part2：计算table中目标bucket的下标；**



**part3：指定目标bucket，遍历Entry结点链表，若找到key相同的Entry结点，则做替换；**

**part4：若未找到目标Entry结点，则新增一个Entry结点。**

### 回答

![put方法](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/put%E6%96%B9%E6%B3%95.png)

首先put方法会调用putVal方法，传入hash值，key，value参数。putVal(hash(key), key, value, false, true)。

判断table（Node数组）是否为空或者长度为0，如果是则进行扩容操作，调用resize方法。根据hash值计算位于哪个数组下标（i = (n - 1) & hash）。如果对应位置为null，说明没有原始，则直接new 一个Node节点，放入对应的数组下标中。如果对应位置有值，判断key是否相等，如果相等则直接进行覆盖。如果是树节点，那么调用putTreeVal方法，使用红黑树的插入方式。如果不是说明是一个链表，则进行链表的遍历，一直遍历到末尾，在遍历过程中，如果找到相同的key那么就直接break并记录找到的元素，如果遍历到末尾还没有找到，那么就插入到末尾，此时需要判断长度是否大于阈值，如果大于那么就需要转换为红黑树，采用红黑树的插入方式。最后如果在插入过程中，找到了已经存在的key，那么就需要进行覆盖，用新的value替换掉旧的value，并返回旧的value值。如果没有找到旧的值，说明需要插入一个新的值，此时需要判断是否超过了扩容的阈值（threshold），如果超过就调用resize方法进行扩容操作。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // table未初始化或者长度为0，进行扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // (n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 桶中已经存在元素
    else {
        Node<K,V> e; K k;
        // 比较桶中第一个元素(数组中的结点)的hash值相等，key相等
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                // 将第一个元素赋值给e，用e来记录
                e = p;
        // hash值不相等，即key不相等；为红黑树结点
        else if (p instanceof TreeNode)
            // 放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 为链表结点
        else {
            // 在链表最末插入结点
            for (int binCount = 0; ; ++binCount) {
                // 到达链表的尾部
                if ((e = p.next) == null) {
                    // 在尾部插入新结点
                    p.next = newNode(hash, key, value, null);
                    // 结点数量达到阈值，转化为红黑树
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    // 跳出循环
                    break;
                }
                // 判断链表中结点的key值与插入的元素的key值是否相等
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 相等，跳出循环
                    break;
                // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        // 表示在桶中找到key值、hash值与插入元素相等的结点
        if (e != null) { 
            // 记录e的value
            V oldValue = e.value;
            // onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                //用新值替换旧值
                e.value = value;
            // 访问后回调
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    // 结构性修改
    ++modCount;
    // 实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
} 
```

对比 JDK1.7的put操作

- ①如果定位到的数组位置没有元素 就直接插入。
- ②如果定位到的数组位置有元素，遍历以这个元素为头结点的链表，依次和插入的key比较，如果key相同就直接覆盖，不同就采用头插法插入元素。

```java
public V put(K key, V value)
    if (table == EMPTY_TABLE) { 
    inflateTable(threshold); 
}  
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) { // 先遍历
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue; 
        }
    }

    modCount++;
    addEntry(hash, key, value, i);  // 再插入
    return null;
}
```






## 扩容操作

1、扩容后大小是扩容前的2倍

2、数据搬迁，从旧table迁到扩容后的新table。 为避免碰撞过多，先决策是否需要对每个Entry链表结点重新hash，然后根据hash值计算得到bucket下标，然后使用头插法做结点迁移



### 回答

进行扩容，会伴随着一次重新hash分配，并且会遍历hash表中所有的元素，是非常耗时的。

首先将原有的node数组table，保留为oldTab，计算原有table的长度（有可能是第一次resize，原有长度为0），如果oldCap长度大于0，说明不是第一次扩容了，判断是否大于最大值（MAXIMUM_CAPACITY = 1 << 30），如果大于那么不进行扩容操作，设置阈值为MAX_VALUE = 0x7fffffff，直接返回oldTab。如果没有大于最大值，那么就扩容为原来的两倍（采用右移操作）并且扩容后也需要小于最大值（newThr = oldThr << 1; // double threshold）。如果是第一次扩容，那么设置数组大小为默认大小（16），设置扩容的阈值为默认大小*默认扩容因子（0.75） = 12。如果前面计算的newThr（扩容阈值）为0，那么进行重新计算，结果指为当前数组大小*扩容因子，并判断指是否大于最大值，如果不大于则采用计算的值，如果大于，则使用Integer.MAX_VALUE。最后将newThr赋值给全局变量扩容阈值（threshold）。

数组大小及扩容阈值计算完成后，就初始化一个新的Node数组。判断oldTab是否为空（是否是第一次resize），如果不是那么就需要将原来的值移动到新的Node数组中（把每个bucket都移动到新的buckets中）。for循环遍历老的Node数组，取出每一个元素，判断当前元素next是否为空，如果为空说明没有hash冲突，直接根据元素的hash值计算出在新的数组中的下标位置，然后插入即可。如果next不为空，那么判断当前元素的结构，如果是红黑树结构，那么采用红黑树的方法移动，如果是链表方式，那么采用链表的方式移动。

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 超过最大值就不再扩充了，就只好随你碰撞去吧
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else { 
        // signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else { 
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
                    if (hiTail != null) {
                        hiTail.next = null;
                        newTab[j + oldCap] = hiHead;
                    }
                }
            }
        }
    }
    return newTab;
}
```





## Get方法

- 先是通过key的hash值计算目标bucket的下标
- 然后遍历对应bucket上的链表，逐个对比，得到结果

```java
public V get(Object key) {
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key);


    return null == entry ? null : entry.getValue();
}


/**
 * Returns the entry associated with the specified key in the
 * HashMap.  Returns null if the HashMap contains no mapping
 * for the key.
 */
final Entry<K,V> getEntry(Object key) {
	// 得到hash值
    int hash = (key == null) ? 0 : hash(key);
    // 获取Entry下标
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

### 回答

结合代码回答。

根据传入的key，调用hash方法得到hash值，然后在Node数组中根据hash值计算出数组的下标，取出第一个元素，如果第一个元素的hash值、key和传入的相等，那么直接返回找到的第一个节点值。如果没有找到，判断是否桶中不止一个节点（first.next() != null），

判断当前节点是什么类型，如果是红黑树，那么采用红黑树的getTreeNode方法获取元素，如果是链表，那么一直遍历链表，判断key是否相同，然后找到的元素。如果都没有找到，那么返回null。

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 数组元素相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 桶中不止一个节点
        if ((e = first.next) != null) {
            // 在树中get
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 在链表中get
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```





## Map迭代

### Iterator迭代器

逐个获取哈希table中的每个bucket中的每个Entry结点

### Map.EntrySet 同时得到 Key  Value



### foreach



### 遍历KeySet

若只需要使用key则使用 Map.KeySet，如果需要使用key-value则不建议使用这个



## fail-fast策略

快速失效策略，当遇到可能会诱导失败的条件时立即上报错误，快速失效系统往往被设计在立即终止正常操作过程，而不是尝试去继续一个可能会存在错误的过程。就是尽可能早的发现问题，立即终止当前执行过程，由更高层级的系统来做处理。

如果需要在for循环删除Map中删除一个元素，直接删除会报错，正确做法是使用iterator.remove



## JDK1.8的实现

结构：哈希表+链表+红黑树

![image-20200816131652418](assets/image-20200816131652418.png)



链表结点的定义也由**`Entry`**类转为了**`Node`**类

### put方法

![image-20200816133600726](assets/image-20200816133600726.png)

### 扩容操作

**场景1**：哈希table为null或长度为0；
**场景2**：Map中存储的k-v对数量超过了阈值**`threshold`**；
**场景3**：链表中的长度超过了`TREEIFY_THRESHOLD`，但表长度却小于`MIN_TREEIFY_CAPACITY`。



### Get方法

1、根据key计算hash ，定位存储在hash表的位置，如果不存在，直接返回null

2、如果存在，并且是树结构，走红黑树查找

3、如果是链表，遍历链表，返回value



## HashMap、ConcurrentHashMap、HashTable的区别

HashMap不是同步的、线程不安全的和不适合应用于多线程并发环境下，而`ConcurrentHashMap`是线程安全的集合容器，特别是在多线程和并发环境中，通常作为`Map`的主要实现





