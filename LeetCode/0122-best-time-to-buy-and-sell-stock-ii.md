# [122. 买卖股票的最佳时机 II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii)

## 题目描述

<!-- 这里写题目描述 -->

给定一个数组`prices`，其中`prices[i]`是一支给定股票第`i`天的价格。

<p>设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。</p>

`注意`：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

<p> </p>

<p><strong>示例 1:</strong></p>

```bash
输入: prices = [7,1,5,3,6,4]
输出: 7
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6-3 = 3 。
```

<p><strong>示例 2:</strong></p>

```bash
输入: prices = [1,2,3,4,5]
输出: 4
解释: 在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。
```

<p><strong>示例 3:</strong></p>

```bash
输入: prices = [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
```

## 分析

<!-- 这里可写通用的实现逻辑 -->

始终保证低价时买入，高价时卖出即可。

## 实现

<!-- tabs:start -->

### **Java**

<!-- 这里可写当前语言的特殊实现逻辑 -->

始终保证低价时买入，高价时卖出，我的实现代码如下：
```java
class Solution {
    public int maxProfit(int[] prices) {
        int res = 0;
        int buyPrice = -1;
        int sellPrice = -1;

        for (int i = 0, len = prices.length; i < len; i++) {
            int curPrice = prices[i];
            if (buyPrice == -1) {
                buyPrice = curPrice;
            } else if (sellPrice == -1 && curPrice <= buyPrice) {
                buyPrice = curPrice;
            } else if (curPrice > sellPrice) {
                sellPrice = curPrice;
            } else if (curPrice < sellPrice) {
                res += (sellPrice - buyPrice);

                buyPrice = curPrice;
                sellPrice = -1;
            }
        }
        if (sellPrice > buyPrice) {
            res += (sellPrice - buyPrice);
        }
        return res;
    }
}
```

其实本质上就是**只要今天比昨天大，就卖出**，更简洁的实现如下：
```java
class Solution {
    public int maxProfit(int[] prices) {
        int res = 0;
        for (int i = 1, len = prices.length; i < len; i++) {
            if (prices[i] > prices[i - 1]) {
                res += (prices[i] - prices[i - 1]);
            }
        }
        return res;
    }
}
```

### **Python3**

<!-- 这里可写当前语言的特殊实现逻辑 -->

始终保证低价时买入，高价时卖出，我的实现代码如下：
```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        res, buy_price, sell_price = 0, -1, -1

        for price in prices:
            if buy_price == -1:
                buy_price = price
            elif sell_price == -1 and price <= buy_price:
                buy_price = price
            elif price > sell_price:
                sell_price = price
            elif price < sell_price:
                res += (sell_price - buy_price)

                buy_price = price
                sell_price = -1

        if sell_price > buy_price:
            res += (sell_price - buy_price)

        return res
```

其实本质上就是**只要今天比昨天大，就卖出**，更简洁的实现如下：
```python
class Solution:
    def maxProfit(self, prices: List[int]) -> int:
        res = 0

        for i in range(1, len(prices)):
            if prices[i] > prices[i - 1]:
                res += (prices[i] - prices[i - 1])

        return res
```

<!-- tabs:end -->