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

```


### **Python3**

<!-- 这里可写当前语言的特殊实现逻辑 -->


**递归版本：**
```python

```

**迭代版本：**
```python

```


<!-- tabs:end -->