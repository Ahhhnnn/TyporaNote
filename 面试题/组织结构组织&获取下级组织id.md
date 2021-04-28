## 通过id pid组装组织结构为树结构

```java
import java.util.List;

/**
 * @author hening
 */
public class Node {
    private String id;
    private String pId;
  	private List<Node> children;
}
```

递归法

```java
public class OrgTest {

    public static void main(String[] args) {
        List<Node> list = new ArrayList<Node>();
        // 父节点
        Node node1 = new Node("1","");
        Node node2 = new Node("2","");
        Node node3 = new Node("3","1");
        Node node4 = new Node("4","1");
        Node node5 = new Node("5","4");
        Node node6 = new Node("6","5");
        Node node7 = new Node("7","2");
        Node node8 = new Node("8","7");
        list.add(node1);
        list.add(node2);
        list.add(node3);
        list.add(node4);
        list.add(node5);
        list.add(node6);
        list.add(node7);
        list.add(node8);
        List<Node> list1 = buildTree(list);
        System.out.println(list1);
    }

    public static  List<Node> buildTree(Collection<Node> nodes) {
        List<Node> tree = new ArrayList<Node>();
        for (Node node : nodes) {
            // 找到所有顶级节点
            if (node.getpId() == null || "".equals(node.getpId())) {
                // 添加二级节点
                tree.add(buildChildren(node, nodes));
            }
        }
        return tree;
    }

    /**
     * 递归构建子节点
     *
     * @param current 当前节点
     * @param nodes   所有节点
     * @return 当前节点及子节点信息
     */
    private static  Node buildChildren(Node current, Collection<Node> nodes) {
        for (Node node : nodes) {
            if (current.getId().equals(node.getpId())) {
                if (null == current.getChildren()) {
                    current.setChildren(new ArrayList<Node>());
                }
                current.getChildren().add(buildChildren(node, nodes));
            }
        }
        return current;
    }
}
```



## 通过id 获取所有下级组织

map  + 递归

```java
public class OrgMapTest {

    public static void main(String[] args) {
        List<Node> list = new ArrayList<Node>();
        // 父节点
        Node node1 = new Node("1","");
        Node node2 = new Node("2","1");
        Node node3 = new Node("3","1");
        Node node4 = new Node("4","1");
        Node node5 = new Node("5","4");
        Node node6 = new Node("6","5");
        Node node7 = new Node("7","2");
        Node node8 = new Node("8","7");
        list.add(node1);
        list.add(node2);
        list.add(node3);
        list.add(node4);
        list.add(node5);
        list.add(node6);
        list.add(node7);
        list.add(node8);
        StopWatch stopWatch = new StopWatch();
        stopWatch.start("组装map");
        Map<String, List<String>> pidMap = new HashMap<>();
        for (Node item : list) {
            pidMap.computeIfAbsent(item.getpId(), k -> new ArrayList<>());
            pidMap.get(item.getpId()).add(item.getId());
        }
        System.out.println(pidMap);
        stopWatch.stop();
        stopWatch.start("递归获取下级");
        // 获取指定组织的所有下级
        Set<String> child = new HashSet<>();
        getChild("1", pidMap, child);
        System.out.println(child);
        stopWatch.stop();
        System.out.println(stopWatch.prettyPrint());
        System.out.println(stopWatch.getTotalTimeSeconds());
    }

    public static Set<String> getChild (String id, Map<String, List<String>> map,Set<String> child){
        List<String> strings = map.get(id);
        if (CollectionUtils.isEmpty(strings)) {
            return Collections.emptySet();
        }
        child.addAll(strings);
        for (String string : strings) {
            child.addAll(getChild(string, map,child));
        }
        return child;
    
```

