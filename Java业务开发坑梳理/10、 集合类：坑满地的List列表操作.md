#  集合类：坑满地的List列表操作

## List的坑

1、**不能直接使用 Arrays.asList 来转换基本类型数组**

Arrays.asList 接受的是泛型T，传入基本类型会被当做一个整体，当做一个泛型

```java

int[] arr1 = {1, 2, 3};
List list1 = Arrays.stream(arr1).boxed().collect(Collectors.toList());
log.info("list:{} size:{} class:{}", list1, list1.size(), list1.get(0).getClass());


Integer[] arr2 = {1, 2, 3};
List list2 = Arrays.asList(arr2);
log.info("list:{} size:{} class:{}", list2, list2.size(), list2.get(0).getClass());
```

2、**Arrays.asList 返回的 List 不支持增删操作**

Arrays.asList 返回的 List 并不是我们期望的 java.util.ArrayList，而是 Arrays 的内部类 ArrayList。ArrayList 内部类继承自 AbstractList 类，并没有覆写父类的 add 方法，而父类中 add 方法的实现，就是抛出 UnsupportedOperationException



3、**对原始数组的修改会影响到我们获得的那个 List**

解决办法：重新new一个新的ArrayList 去接收

```java

String[] arr = {"1", "2", "3"};
List list = new ArrayList(Arrays.asList(arr));
arr[1] = "4";
try {
    list.add("5");
} catch (Exception ex) {
    ex.printStackTrace();
}
log.info("arr:{} list:{}", Arrays.toString(arr), list);
```



## 使用 List.subList 进行切片操作居然会导致 OOM？

List.subList 返回的同样不是我们期望的ArrayList 而是一个内部类SubList。

如果存在在for循环中一只调用subList 方法，那么原本的List会被subList 返回的List强引用导致得不到垃圾回收，可能会导致OOM问题。

```java

private static List<List<Integer>> data = new ArrayList<>();

private static void oom() {
    for (int i = 0; i < 1000; i++) {
        List<Integer> rawList = IntStream.rangeClosed(1, 100000).boxed().collect(Collectors.toList());
        data.add(rawList.subList(0, 1));
    }
}
```

因为在SubList中并没有一个单独的变量来保存原有的List，SubList 初始化的时候，并没有把原始 List 中的元素复制到独立的变量中保存。我们可以认为 SubList 是原始 List 的视图，并不是独立的 List。双方对元素的修改会相互影响，而且 SubList 强引用了原始的 List，所以大量保存这样的 SubList 会导致 OOM。

```java

List<Integer> list = IntStream.rangeClosed(1, 10).boxed().collect(Collectors.toList());
List<Integer> subList = list.subList(1, 4);
System.out.println(subList);
subList.remove(1);
System.out.println(list);
list.add(0);
try {
    subList.forEach(System.out::println);
} catch (Exception ex) {
    ex.printStackTrace();
}
```

由于SubList 被当做一个视图，在遍历SubList 的时候会跟原始的List的modCount进行比较，进行判等于，如果不等于则抛出ConcurrentModificationException异常



## 一定要让合适的数据结构做合适的事情

在进行大量数据的查找操作时，要选对合适的数据结构。

**要对大 List 进行单值搜索的话，可以考虑使用 HashMap，其中 Key 是要搜索的值，Value 是原始对象，会比使用 ArrayList 有非常明显的性能优势**

