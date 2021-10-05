# [206. 反转链表](https://leetcode-cn.com/problems/reverse-linked-list)

## 题目描述

<!-- 这里写题目描述 -->

<p>反转一个单链表。</p>

<p><strong>示例:</strong></p>

```bash
输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL
```

<p><strong>进阶:</strong><br>
你可以迭代或递归地反转链表。你能否用两种方法解决这道题？</p>

## 分析

<!-- 这里可写通用的实现逻辑 -->

**迭代思路：**  

把第一个`Node`置为`null`，然后在迭代过程中总是把后一个`Node`放到链表头部。

<br />

**递归思路：**  

第一种方式：类似于上面的迭代思路，把第一个`Node`置为`null`，相邻两个`Node`反转后，继续取后一个`Node`参与反转。

第二种方式：先反转`head.next`，然后再把`head`放到链表尾部。反转`head.next`正好是一个递归的过程。


## 实现

<!-- tabs:start -->

### **Java**

<!-- 这里可写当前语言的特殊实现逻辑 -->

**迭代版本：**
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        ListNode pre = null;
        ListNode cur = head;
        while (cur != null) {
            ListNode next = cur.next;
            cur.next = pre;
            pre = cur;
            cur = next;
        }
        return pre;
    }
}
```

**递归版本(一)：**
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        return reverse(null, head);
    }

    public ListNode reverse(ListNode pre, ListNode cur) {
        if (cur == null) {
            return pre;
        }
        ListNode next = cur.next;
        cur.next = pre;
        return reverse(cur, next);
    }
}
```


**递归版本(二)：**
```java
/**
 * Definition for singly-linked list.
 * public class ListNode {
 *     int val;
 *     ListNode next;
 *     ListNode() {}
 *     ListNode(int val) { this.val = val; }
 *     ListNode(int val, ListNode next) { this.val = val; this.next = next; }
 * }
 */
class Solution {
    public ListNode reverseList(ListNode head) {
        if (head == null || head.next == null) {
            return head;
        }

        ListNode res = reverseList(head.next);
        head.next.next = head;
        head.next = null;

        return res;
    }
}
```

### **Python3**

<!-- 这里可写当前语言的特殊实现逻辑 -->

**迭代版本：**
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseList(self, head: ListNode) -> ListNode:
        pre, cur = None, head
        while cur:
            cur.next, pre, cur = pre, cur, cur.next
        return pre
```

**递归版本(一)：**
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseList(self, head: ListNode) -> ListNode:
        
        def reverse(pre: ListNode, cur: ListNode) -> ListNode:
            if not cur:
                return pre
            next_node = cur.next
            cur.next = pre
            return reverse(cur, next_node)

        return reverse(None, head)
```

**递归版本(二)：**
```python
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, val=0, next=None):
#         self.val = val
#         self.next = next
class Solution:
    def reverseList(self, head: ListNode) -> ListNode:
        if not head or not head.next:
            return head
        res = self.reverseList(head.next)
        head.next.next, head.next = head, None
        return res
```

<!-- tabs:end -->