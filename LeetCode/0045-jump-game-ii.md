# [45. 跳跃游戏 II](https://leetcode-cn.com/problems/jump-game-ii)

## 题目描述

<!-- 这里写题目描述 -->

<p>给定一个非负整数数组，你最初位于数组的第一个位置。</p>

<p>数组中的每个元素代表你在该位置可以跳跃的最大长度。</p>

<p>你的目标是使用最少的跳跃次数到达数组的最后一个位置。</p>

<p><strong>示例:</strong></p>

```bash
输入: [2,3,1,1,4]
输出: 2
解释: 跳到最后一个位置的最小跳跃数是 2。
     从下标为 0 跳到下标为 1 的位置，跳 1 步，然后跳 3 步到达数组的最后一个位置。
```

<p><strong>说明:</strong></p>

<p>假设你总是可以到达数组的最后一个位置。</p>

## 分析

<!-- 这里可写通用的实现逻辑 -->

贪心算法，详细可参考: [代码随想录](https://programmercarl.com/0045.%E8%B7%B3%E8%B7%83%E6%B8%B8%E6%88%8FII.html)

## 实现

<!-- tabs:start -->

### **Java**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```java
class Solution {
    public int jump(int[] nums) {
        if (nums == null || nums.length <= 1) {
            return 0;
        }
        // 跳跃次数
        int count = 0;
        // 当前的覆盖最大区域
        int curDistance = 0;
        // 最大的覆盖区域
        int maxDistance = 0;

        for (int i = 0; i < nums.length; i++) {
            // 在可覆盖区域内更新最大的覆盖区域
            maxDistance = Math.max(maxDistance, i + nums[i]);
            // 说明当前一步，再跳一步就到达了末尾
            if (maxDistance >= nums.length - 1) {
                count++;
                break;
            }
            // 走到当前覆盖的最大区域时，更新下一步可达的最大区域
            if (i == curDistance) {
                curDistance = maxDistance;
                count++;
            }
        }

        return count;
    }
}
```

这里顺便也保留自己用动态规划思路的实现(虽然不够巧妙，但也通过了)：
```java
class Solution {
    public int jump(int[] nums) {
        if (nums.length == 1) {
            return 0;
        }
        // 记录对应位置到达最后一个下标需要的最小跳跃数
        // 初始值都为0，表示不知道能否到达最后一个下标
        int[] history = new int[nums.length];
        return jump(nums, 0, history);
    }

    public int jump(int[] nums, int startIdx, int[] history) {
        int count = history[startIdx];
        if (count == -1) {
            return -1;
        }
        int maxStep = nums[startIdx];
        if ((startIdx + maxStep) >= (nums.length - 1)) {
            // 从当前位置可以直接到达最后一个下标
            count++;
            return count;
        }
        Integer minCount = null;
        for (int i = maxStep; i >= 1; i--) {
            // 先看是否已经记录过
            int jumpCount = history[startIdx + i];
            if (jumpCount == 0) {
                // 尝试往前跳跃长度i
                jumpCount = jump(nums, startIdx + i, history);
                history[startIdx + i] = jumpCount;
            }

            if (jumpCount > 0 && (minCount == null || jumpCount < minCount)) {
                minCount = jumpCount;
            }
        }
        if (minCount == null) {
            return -1;
        }
        return count + 1 + minCount;
    }
}
```

### **Python3**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```python
class Solution:
    def jump(self, nums: List[int]) -> int:
        if not nums:
            return 0

        length = len(nums)

        if length <= 1:
            return 0

        # 跳跃次数
        count = 0
        # 当前的覆盖最大区域
        cur_distance = 0
        # 最大的覆盖区域
        max_distance = 0

        for i in range(length):
            # 在可覆盖区域内更新最大的覆盖区域
            max_distance = max(max_distance, i + nums[i])
            # 说明当前一步，再跳一步就到达了末尾
            if max_distance >= length - 1:
                count += 1
                break
            # 走到当前覆盖的最大区域时，更新下一步可达的最大区域
            if i == cur_distance:
                cur_distance = max_distance
                count += 1

        return count
```

<!-- tabs:end -->