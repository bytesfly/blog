# [144. 二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal)

## 题目描述

<!-- 这里写题目描述 -->

给你二叉树的根节点`root`，返回它节点值的`前序`遍历。

<p> </p>

<p><strong>示例 1：</strong></p>
<img alt="" src="https://cdn.jsdelivr.net/gh/doocs/leetcode@main/solution/0100-0199/0144.Binary%20Tree%20Preorder%20Traversal/images/inorder_1.jpg" style="width: 202px; height: 324px;" />

```bash
输入：root = [1,null,2,3]
输出：[1,2,3]
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
<img alt="" src="https://cdn.jsdelivr.net/gh/doocs/leetcode@main/solution/0100-0199/0144.Binary%20Tree%20Preorder%20Traversal/images/inorder_5.jpg" style="width: 202px; height: 202px;" />

```bash
输入：root = [1,2]
输出：[1,2]
```

<p><strong>示例 5：</strong></p>
<img alt="" src="https://cdn.jsdelivr.net/gh/doocs/leetcode@main/solution/0100-0199/0144.Binary%20Tree%20Preorder%20Traversal/images/inorder_4.jpg" style="width: 202px; height: 202px;" />

```bash
输入：root = [1,null,2]
输出：[1,2]
```

<p> </p>

<p><strong>提示：</strong></p>

- 树中节点数目在范围`[0, 100]`内
- -100 <= `Node.val` <= 100


<p><strong>进阶：</strong>递归算法很简单，你可以通过迭代算法完成吗？</p>

## 分析

<!-- 这里可写通用的实现逻辑 -->

**递归思路：**

先立刻记录根节点的值，然后递归访问左孩子，递归访问右孩子。

<br />

**迭代思路：**

可以借助栈来实现，初始化一个只有根节点的栈。只要栈不为空，反复执行以下操作：  
【弹出栈顶元素，立刻记录该栈顶元素的值；如果该栈顶元素的右孩子不为空，则入栈；如果该栈顶元素的左孩子不为空，则入栈】


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
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        visit(res, root);
        return res;
    }

    public void visit(List<Integer> res, TreeNode root) {
        if (root == null) {
            return;
        }

        res.add(root.val);

        if (root.left != null) {
            visit(res, root.left);
        }

        if (root.right != null) {
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
    public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        if (root == null) {
            return res;
        }

        LinkedList<TreeNode> stack = new LinkedList<>();
        stack.push(root);

        while (!stack.isEmpty()) {
            TreeNode cur = stack.pop();
            res.add(cur.val);

            if (cur.right != null) {
                stack.push(cur.right);
            }
            if (cur.left != null) {
                stack.push(cur.left);
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
    def preorderTraversal(self, root: TreeNode) -> List[int]:
        res = []
        if not root:
            return res

        def visit(root: TreeNode):
            res.append(root.val)
            if root.left:
                visit(root.left)
            if root.right:
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
    def preorderTraversal(self, root: TreeNode) -> List[int]:
        res = []
        if not root:
            return res

        stack = [root]
        while stack:
            cur = stack.pop()
            res.append(cur.val)
            if cur.right:
                stack.append(cur.right)
            if cur.left:
                stack.append(cur.left)
        
        return res
```

<!-- tabs:end -->