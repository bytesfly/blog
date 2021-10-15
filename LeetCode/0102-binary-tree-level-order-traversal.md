# [102. 二叉树的层序遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal)

## 题目描述

<!-- 这里写题目描述 -->

给你一个二叉树，请你返回其按`层序遍历`得到的节点值。 （即逐层地，从左到右访问所有节点）。

<strong>示例：</strong><br />
二叉树：`[3,9,20,null,null,15,7]`,

```bash
    3
   / \
  9  20
    /  \
   15   7
```

<p>返回其层序遍历结果：</p>

```bash
[
  [3],
  [9,20],
  [15,7]
]
```

## 分析

<!-- 这里可写通用的实现逻辑 -->

这题其实不难，把根节点放到一个`queue`中，遍历该`queue`时把当前节点的左右孩子放到另外一个`queue`中，再遍历新生成的`queue`。

在上面的思路下迭代与递归差别不大。

## 实现

<!-- tabs:start -->

### **Java**

<!-- 这里可写当前语言的特殊实现逻辑 -->

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
    public List<List<Integer>> levelOrder(TreeNode root) {
        List<List<Integer>> res = new ArrayList<>();
        if (root == null) {
            return res;
        }
        List<TreeNode> nodes = new ArrayList<>();
        nodes.add(root);
        while (!nodes.isEmpty()) {
            List<Integer> values = new ArrayList<>();
            List<TreeNode> nextNodes = new ArrayList<>();
            for (TreeNode node : nodes) {
                values.add(node.val);
                if (node.left != null) {
                    nextNodes.add(node.left);
                }
                if (node.right != null) {
                    nextNodes.add(node.right);
                }
            }
            res.add(values);
            nodes = nextNodes;
        }
        return res;
    }
}
```

### **Python3**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```python
# Definition for a binary tree node.
# class TreeNode:
#     def __init__(self, val=0, left=None, right=None):
#         self.val = val
#         self.left = left
#         self.right = right
class Solution:
    def levelOrder(self, root: TreeNode) -> List[List[int]]:
        res = []
        if not root:
            return res

        nodes = [root]
        while len(nodes) > 0:
            values = []
            next_nodes = []

            for node in nodes:
                values.append(node.val)
                if node.left:
                    next_nodes.append(node.left)
                if node.right:
                    next_nodes.append(node.right)

            res.append(values)
            nodes = next_nodes

        return res
```

<!-- tabs:end -->
