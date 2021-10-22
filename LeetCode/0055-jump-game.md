# [55. 跳跃游戏](https://leetcode-cn.com/problems/jump-game)

## 题目描述

<!-- 这里写题目描述 -->

给定一个非负整数数组`nums`，你最初位于数组的`第一个下标`。

<p>数组中的每个元素代表你在该位置可以跳跃的最大长度。</p>

<p>判断你是否能够到达最后一个下标。</p>

<p> </p>

<p><strong>示例 1：</strong></p>

```bash
输入：nums = [2,3,1,1,4]
输出：true
解释：可以先跳 1 步，从下标 0 到达下标 1, 然后再从下标 1 跳 3 步到达最后一个下标。
```

<p><strong>示例 2：</strong></p>

```bash
输入：nums = [3,2,1,0,4]
输出：false
解释：无论怎样，总会到达下标为 3 的位置。但该下标的最大跳跃长度是 0 ， 所以永远不可能到达最后一个下标。
```

## 分析

<!-- 这里可写通用的实现逻辑 -->

从头到尾遍历数组，如果到不了数组的某个位置，那么也就到不了最后一个下标，是不是这样？

## 实现

<!-- tabs:start -->

### **Java**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```java
class Solution {
    public boolean canJump(int[] nums) {
        // 当前能到达的最大位置
        int curMax = 0;
        for (int i = 0, len = nums.length; i < len; i++) {
            if (curMax < i) {
                return false;
            }
            curMax = Math.max(curMax, i + nums[i]);
        }
        return true;
    }
}
```

这里顺便也保留自己第一次做的实现(虽然不够简洁，但也通过了)：
```java
class Solution {
    public boolean canJump(int[] nums) {
        // 记录对应位置能否到达最后一个下标，true表示不能到达最后一个下标
        // 初始值都为false，表示不知道能否到达最后一个下标
        boolean[] history = new boolean[nums.length];
        return tryJump(nums, 0, history);
    }

    public boolean tryJump(int[] nums, int startIdx, boolean[] history) {
        if (history[startIdx]) {
            // 如果前面已经确定从startIdx位置不能到达最后一个下标，直接返回
            return false;
        }
        int maxStep = nums[startIdx];
        if ((startIdx + maxStep) >= (nums.length - 1)) {
            // 从当前位置可以直接到达最后一个下标
            return true;
        }
        for (int i = maxStep; i >= 1; i--) {
            // 尝试往前跳跃长度i
            boolean canJump = tryJump(nums, startIdx + i, history);
            if (canJump) {
                return true;
            } else {
                // 记录从(startIdx + i)位置不能到达最后一个下标
                history[startIdx + i] = true;
            }
        }
        return false;
    }
}
```

### **Python3**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```python
class Solution:
    def canJump(self, nums: List[int]) -> bool:
        # 当前能到达的最大位置
        cur_max = 0
        for i in range(len(nums)):
            if cur_max < i:
                return False
            cur_max = max(cur_max, i + nums[i])
        return True
```

<!-- tabs:end -->