# [94. 二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal)

## 题目描述

<!-- 这里写题目描述 -->

给定一个二叉树的根节点`root`，返回它的`中序`遍历。

<p> </p>

<p><strong>示例 1：</strong></p>
<img alt="" src="https://cdn.jsdelivr.net/gh/doocs/leetcode@main/solution/0000-0099/0094.Binary%20Tree%20Inorder%20Traversal/images/inorder_1.jpg" style="width: 202px; height: 324px;" />

```bash
输入：root = [1,null,2,3]
输出：[1,3,2]
```

<p><strong>示例 2：</strong></p>

```bash
输入：root = []
输出：[]
```

<p><strong>示例 3：</strong></p>

```bash
输入：root = [1]
输出：[1]
```

<p><strong>示例 4：</strong></p>
<img alt="" src="https://cdn.jsdelivr.net/gh/doocs/leetcode@main/solution/0000-0099/0094.Binary%20Tree%20Inorder%20Traversal/images/inorder_5.jpg" style="width: 202px; height: 202px;" />

```bash
输入：root = [1,2]
输出：[2,1]
```

<p><strong>示例 5：</strong></p>
<img alt="" src="https://cdn.jsdelivr.net/gh/doocs/leetcode@main/solution/0000-0099/0094.Binary%20Tree%20Inorder%20Traversal/images/inorder_4.jpg" style="width: 202px; height: 202px;" />

```bash
输入：root = [1,null,2]
输出：[1,2]
```

<p> </p>

<p><strong>提示：</strong></p>

- 树中节点数目在范围`[0, 100]`内
- -100 <= `Node.val` <= 100

<p> </p>

<p><strong>进阶:</strong> 递归算法很简单，你可以通过迭代算法完成吗？</p>

## 分析

<!-- 这里可写通用的实现逻辑 -->

**递归思路：**

先递归访问左孩子，然后记录该节点的值，再递归访问右孩子。

<br />

**迭代思路：**

中序遍列，始终牢记【左，根，右】的顺序，优先左孩子，然后根节点，最后右孩子，遍历左右孩子时同样遵循【左，根，右】的顺序。

1. 只要有左孩子，就需要一直往左遍历，所以需要用`栈`来记录遇到的节点，直到左孩子为空为止；
2. 弹出栈顶元素，并输出该节点的值；
3. 当第2步中的栈顶节点的右孩子为空，则重复步骤2和3，否则重复步骤1,2,3。

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
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        visit(res, root);
        return res;
    }

    public void visit(List<Integer> res, TreeNode root) {
        if (root != null) {
            visit(res, root.left);
            res.add(root.val);
            visit(res, root.right);
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
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        if (root == null) {
            return res;
        }
        Queue<TreeNode> queue = new LinkedList<>();
        queue.offer(root);

        LinkedList<TreeNode> stack = new LinkedList<>();

        while (!queue.isEmpty() || !stack.isEmpty()) {

            if (!queue.isEmpty()) {
                TreeNode cur = queue.poll();
                stack.push(cur);

                while (cur.left != null) {
                    cur = cur.left;
                    stack.push(cur);
                }
            }

            if (!stack.isEmpty()) {
                TreeNode cur = stack.pop();
                res.add(cur.val);

                if (cur.right != null) {
                    queue.offer(cur.right);
                }
            }
        }

        return res;
    }
}
```

迭代过程中，你可以发现上面代码的那个`queue`中其实始终最多只会有一个元素，所以可以再简单优化一下：
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
    public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        if (root == null) {
            return res;
        }
        LinkedList<TreeNode> stack = new LinkedList<>();
        TreeNode cur = root;

        while (cur != null || !stack.isEmpty()) {
            if (cur != null) {
                stack.push(cur);
                cur = cur.left;
            } else {
                cur = stack.pop();
                res.add(cur.val);
                cur = cur.right;
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
    def inorderTraversal(self, root: TreeNode) -> List[int]:
        res = []

        def visit(root: TreeNode):
            if root:
                visit(root.left)
                res.append(root.val)
                visit(root.right)

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
    def inorderTraversal(self, root: TreeNode) -> List[int]:
        res = []

        cur = root
        stack = []

        while cur or stack:
            if cur:
                stack.append(cur)
                cur = cur.left
            else:
                cur = stack.pop()
                res.append(cur.val)
                cur = cur.right

        return res
```

<!-- tabs:end -->