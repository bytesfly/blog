# [136. 只出现一次的数字](https://leetcode-cn.com/problems/single-number)

## 题目描述

<!-- 这里写题目描述 -->

<p>给定一个<strong>非空</strong>整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。</p>

<p><strong>说明：</strong></p>

<p>你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？</p>

<p><strong>示例 1:</strong></p>

```bash
输入: [2,2,1]
输出: 1
```

<p><strong>示例 2:</strong></p>

```bash
输入: [4,1,2,1,2]
输出: 4
```

## 分析

<!-- 这里可写通用的实现逻辑 -->

用`异或运算`求解。

```bash
num1 ^ num2 ^ num3 = num1 ^ num3 ^ num2

num ^ num = 0

num ^ 0 = num
```
即：
- `异或运算`满足交换律
- 两个相同的数`异或`之后等于`0`
- 一个数与0进行`异或`等于其本身

所以，对该数组所有元素进行异或运算，结果就是那个只出现一次的数字。

## 实现

<!-- tabs:start -->

### **Java**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```java
class Solution {
    public int singleNumber(int[] nums) {
        int res = 0;
        for (int num : nums) {
            res ^= num;
        }
        return res;
    }
}
```

### **Python3**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```python
class Solution:
    def singleNumber(self, nums: List[int]) -> int:
        res = 0
        for num in nums:
            res ^= num
        return res
```

<!-- tabs:end -->