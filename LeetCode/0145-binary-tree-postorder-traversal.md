# [145. 二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal)

## 题目描述

<!-- 这里写题目描述 -->

给定一个二叉树，返回它的`后序`遍历。

<p><strong>示例:</strong></p>

```bash
输入: [1,null,2,3]  
   1
    \
     2
    /
   3 

输出: [3,2,1]
```

<p><strong>进阶:</strong>&nbsp;递归算法很简单，你可以通过迭代算法完成吗？</p>

## 分析

<!-- 这里可写通用的实现逻辑 -->

**递归思路：**

先递归访问左孩子，然后递归访问右孩子，再记录根节点的值。

<br />

**迭代思路：**

仔细观察后序遍历的元素进出栈规律编写代码，细节见下面代码中的注释。

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
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        visit(res, root);
        return res;
    }

    public void visit(List<Integer> res, TreeNode root) {
        if (root != null) {
            visit(res, root.left);
            visit(res, root.right);
            res.add(root.val);
        }
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
    public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();

        TreeNode cur = root;
        // 记录上次输出的节点
        TreeNode visited = null;

        LinkedList<TreeNode> stack = new LinkedList<>();

        while (cur != null || !stack.isEmpty()) {
            if (cur != null) {
                // 左边有孩子，则一直往左遍历
                stack.push(cur);
                cur = cur.left;
            } else {
                // 查看栈顶节点
                TreeNode node = stack.peek();
                if (node.right != null && node.right != visited) {
                    // 如果栈顶节点的右孩子不为null且没有输出过，则继续往右走一个节点
                    cur = node.right;
                } else {
                    // 如果栈顶节点的右孩子为null或者已经输出过，则直接弹出栈顶元素并且输出，同时对该节点做标记
                    node = stack.pop();
                    res.add(node.val);
                    visited = node;
                }
            }
        }

        return res;
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
    def postorderTraversal(self, root: TreeNode) -> List[int]:
        res = []

        def visit(root: TreeNode):
            if root:
                visit(root.left)
                visit(root.right)
                res.append(root.val)

        visit(root)
        return res
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
    def postorderTraversal(self, root: TreeNode) -> List[int]:
        res = []

        cur = root
        # 记录上次输出的节点
        visited = None
        stack = []

        while cur or stack:
            if cur:
                # 左边有孩子，则一直往左遍历
                stack.append(cur)
                cur = cur.left
            else:
                # 查看栈顶节点
                node = stack[-1]
                if node.right and node.right != visited:
                    # 如果栈顶节点的右孩子不为None且没有输出过，则继续往右走一个节点
                    cur = node.right
                else:
                    # 如果栈顶节点的右孩子为None或者已经输出过，则直接弹出栈顶元素并且输出，同时对该节点做标记
                    node = stack.pop()
                    res.append(node.val)
                    visited = node

        return res
```

<!-- tabs:end -->

