# [101. 对称二叉树](https://leetcode-cn.com/problems/symmetric-tree)

## 题目描述

<!-- 这里写题目描述 -->

<p>给定一个二叉树，检查它是否是镜像对称的。</p>

<p>&nbsp;</p>

例如，二叉树 `[1,2,2,3,4,4,3]`是对称的。

```bash
    1
   / \
  2   2
 / \ / \
3  4 4  3
```

<p>&nbsp;</p>

但是下面这个`[1,2,2,null,3,null,3]`则不是镜像对称的:

```bash
    1
   / \
  2   2
   \   \
   3    3
```

<p>&nbsp;</p>

<p><strong>进阶：</strong></p>

<p>你可以运用递归和迭代两种方法解决这个问题吗？</p>

## 分析

<!-- 这里可写通用的实现逻辑 -->

**递归思路：**

当左子树与右子树对称时，这棵树镜像对称。  
那么怎么知道左子树与右子树是否镜像对称呢？  
左树的左孩子与右树的右孩子对称，左树的右孩子与右树的左孩子对称，那么这个左树和右树就对称。

<br />

**迭代思路：**

层序遍历，然后检查每一层是不是回文数组。



## 实现

<!-- tabs:start -->

### **Java**

<!-- 这里可写当前语言的特殊实现逻辑 -->

**递归版本：**
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean isSymmetric(TreeNode root) {
        if (root == null) {
            return true;
        }
        return compare(root.left, root.right);
    }

    public boolean compare(TreeNode left, TreeNode right) {
        if (left == null && right == null) {
            return true;
        }

        if (left != null && right != null
                && left.val == right.val
                && compare(left.left, right.right)
                && compare(left.right, right.left)) {
            return true;
        }

        return false;
    }
}
```

**迭代版本：**
```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    public boolean isSymmetric(TreeNode root) {
        List<TreeNode> nodes = Arrays.asList(root);
        while (!nodes.isEmpty()) {
            int size = nodes.size();
            List<Integer> values = new ArrayList<>(size);
            List<TreeNode> nextNodes = new ArrayList<>(size * 2);

            for (TreeNode node : nodes) {
                if (node == null) {
                    values.add(null);
                } else {
                    values.add(node.val);

                    nextNodes.add(node.left);
                    nextNodes.add(node.right);
                }
            }

            if (!isSymmetric(values)) {
                return false;
            }

            nodes = nextNodes;
        }
        return true;
    }

    public boolean isSymmetric(List<Integer> values) {
        for (int i = 0, j = values.size() - 1; i < j; i++, j--) {
            Integer left = values.get(i);
            Integer right = values.get(j);

            if (left == null && right != null) {
                return false;
            } else if (left != null && right == null) {
                return false;
            } else if (left != null && right != null && !left.equals(right)) {
                return false;
            }
        }
        return true;
    }
}
```


### **Python3**

<!-- 这里可写当前语言的特殊实现逻辑 -->


**递归版本：**
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def isSymmetric(self, root: TreeNode) -> bool:
        
        def compare(left: TreeNode, right: TreeNode) -> bool:
            if not left and not right:
                return True
            if not left or not right or left.val != right.val:
                return False
            return compare(left.left, right.right) and compare(left.right, right.left)

        if not root:
            return True
        return compare(root.left, root.right)
```

**迭代版本：**
```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def isSymmetric(self, root: TreeNode) -> bool:
        nodes = [root]

        while nodes:
            values = []
            next_nodes = []

            for node in nodes:
                if node:
                    values.append(node.val)

                    next_nodes.append(node.left)
                    next_nodes.append(node.right)
                else:
                    values.append(None)

            if values != values[::-1]:
                return False

            nodes = next_nodes

        return True
```


<!-- tabs:end -->