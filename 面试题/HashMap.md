

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



