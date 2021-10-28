# [121. 买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock)

## 题目描述

<!-- 这里写题目描述 -->

给定一个数组`prices`，它的第`i`个元素`prices[i]`表示一支给定股票第`i`天的价格。

你只能选择`某一天`买入这只股票，并选择在`未来的某一个不同的日子`卖出该股票。设计一个算法来计算你所能获取的最大利润。

返回你可以从这笔交易中获取的最大利润。如果你不能获取任何利润，返回`0`。

<p> </p>

<p><strong>示例 1：</strong></p>

```bash
输入：[7,1,5,3,6,4]
输出：5
解释：在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。
```

<p><strong>示例 2：</strong></p>

```bash
输入：prices = [7,6,4,3,1]
输出：0
解释：在这种情况下, 没有交易完成, 所以最大利润为 0。
```

<p> </p>

<p><strong>注意：</strong></p>

- 1 <= `prices.length` <= 105
- 0 <= `prices[i]` <= 104

## 分析

<!-- 这里可写通用的实现逻辑 -->

遍历数组，遍历过程中不断更新最大的利润以及最小的价格。

## 实现

<!-- tabs:start -->

### **Java**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```java
class Solution {
    public int maxProfit(int[] prices) {
        if (prices == null || prices.length <= 1) {
            return 0;
        }
        int minPrice = prices[0];
        int maxProfit = 0;

        for (int i = 1, len = prices.length; i < len; i++) {
            maxProfit = Math.max(maxProfit, prices[i] - minPrice);
            minPrice = Math.min(minPrice, prices[i]);
        }

        return maxProfit;
    }
}
```

### **Python3**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        if not prices or len(prices) <= 1:
            return 0

        min_price, max_profit = prices[0], 0

        for i in range(1, len(prices)):
            max_profit = max(max_profit, prices[i] - min_price)
            min_price = min(min_price, prices[i])
        
        return max_profit
```

<!-- tabs:end -->