# [746. 使用最小花费爬楼梯](https://leetcode-cn.com/problems/min-cost-climbing-stairs)

## 题目描述

<!-- 这里写题目描述 -->

数组的每个下标作为一个阶梯，第`i`个阶梯对应着一个非负数的体力花费值`cost[i]`（下标从`0`开始）。</p>

<p>每当你爬上一个阶梯你都要花费对应的体力值，一旦支付了相应的体力值，你就可以选择向上爬一个阶梯或者爬两个阶梯。</p>

请你找出达到楼层顶部的最低花费。在开始时，你可以选择从下标为`0`或`1`的元素作为初始阶梯。

<p> </p>

<p><strong>示例 1：</strong></p>

```bash
输入：cost = [10, 15, 20]
输出：15
解释：最低花费是从 cost[1] 开始，然后走两步即可到阶梯顶，一共花费 15 。
```

<p><strong> 示例 2：</strong></p>

```bash
输入：cost = [1, 100, 1, 1, 1, 100, 1, 1, 100, 1]
输出：6
解释：最低花费方式是从 cost[0] 开始，逐个经过那些 1 ，跳过 cost[3] ，一共花费 6 。
```

<p> </p>

<p><strong>提示：</strong></p>

- `cost`的长度范围是`[2, 1000]`。
- `cost[i]`将会是一个整型数据，范围为`[0, 999]`。

## 分析

<!-- 这里可写通用的实现逻辑 -->

想上第 n 级台阶，可从第 n-1 级台阶爬一级上去，也可从第 n-2 级台阶爬两级上去。

所以到达第n级阶梯所需最小体力f(n)的递推关系为：  

f(n) = min ( f(n-1) + cost[n-1] , f(n-2) + cost[n-2] )

## 实现

<!-- tabs:start -->

### **Java**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```java
class Solution {
    public int minCostClimbingStairs(int[] cost) {
        // 到达第n-2个楼梯所需的最小体力花费
        int a = 0;
        // 到达第n-1个楼梯所需的最小体力花费
        int b = 0;

        // 题目要求到达楼梯顶，所以相当于是到达 (数组下标的最后位置+1)
        int last = cost.length;

        for (int n = 2; n <= last; n++) {
            // 到达第n个楼梯所需的最小体力花费
            int cur = Math.min(a + cost[n - 2], b + cost[n - 1]);
            a = b;
            b = cur;
        }

        return b;
    }
}
```

### **Python3**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```python
class Solution:
    def minCostClimbingStairs(self, cost: List[int]) -> int:
        # 到达第n-2, n-1级别所需的最小体力
        a, b = 0, 0

        # 题目要求到达楼梯顶，所以相当于是到达 (数组下标的最后位置+1)
        last = len(cost) + 1

        for n in range(2, last):
            a, b = b, min(a + cost[n - 2], b + cost[n - 1])

        return b
```

<!-- tabs:end -->