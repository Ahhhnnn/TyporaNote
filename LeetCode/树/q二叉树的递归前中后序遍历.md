## 二叉树前中后序递归遍历



```java
public class Solution {

    public static void main(String[] args) {
        TreeNode root = new TreeNode(1);
        root.left = new TreeNode(2);
        root.right = new TreeNode(3);
        root.right.left = new TreeNode(4);
        root.right.right = new TreeNode(5);
        List<Integer> result = new ArrayList<Integer>();
        //helperOrder(result,root);
        //helperIn(result,root);
        helperPre(result,root);
        System.out.println(result);
    }

    // 递归前序遍历
    private static void helperOrder(List<Integer> result, TreeNode root){
        if(root == null){
            return;
        }
        result.add(root.val);
        helperOrder(result,root.left);
        helperOrder(result,root.right);
    }

    // 递归中序遍历
    private static void helperIn(List<Integer> result, TreeNode root){
        if(root == null){
            return;
        }
        helperIn(result,root.left);
        result.add(root.val);
        helperIn(result,root.right);
    }

    // 递归后序遍历
    private static void helperPre(List<Integer> result, TreeNode root){
        if(root == null){
            return;
        }
        helperPre(result,root.left);
        helperPre(result,root.right);
        result.add(root.val);
    }



    //Definition for a binary tree node.
    public static class TreeNode {
        int val;
        TreeNode left;
        TreeNode right;
        TreeNode(int x) { val = x; }
    }
}
```

