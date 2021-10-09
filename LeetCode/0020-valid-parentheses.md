# [20. 有效的括号](https://leetcode-cn.com/problems/valid-parentheses)

## 题目描述

<!-- 这里写题目描述 -->

给定一个只包括 `'('，')'，'{'，'}'，'['，']'` 的字符串`s`，判断字符串是否有效。

<p>有效字符串需满足：</p>

1. 左括号必须用相同类型的右括号闭合。
2. 左括号必须以正确的顺序闭合。

<p> </p>

<p><strong>示例 1：</strong></p>

```bash
输入：s = "()"
输出：true
```

<p><strong>示例 2：</strong></p>

```bash
输入：s = "()[]{}"
输出：true
```

<p><strong>示例 3：</strong></p>

```bash
输入：s = "(]"
输出：false
```

<p><strong>示例 4：</strong></p>

```bash
输入：s = "([)]"
输出：false
```

<p><strong>示例 5：</strong></p>

```bash
输入：s = "{[]}"
输出：true
```

## 分析

<!-- 这里可写通用的实现逻辑 -->

`last-in-first-out`，正好可用`栈`(`LIFO`)这种常见的数据结构来实现。  

遍历字符串中的字符，遇到左括号则进栈，遇到右括号则弹出栈顶字符，如不匹配当前字符则直接返回，如匹配继续往下遍历。  

注意，遍历完所有字符后，空栈才能表示字符串是符合规则的。


## 实现

<!-- tabs:start -->

### **Java**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```java
class Solution {
    public boolean isValid(String s) {
        if (s == null || s.isEmpty()) {
            return false;
        }
        Map<Character, Character> charMap = new HashMap<>();
        charMap.put(')', '(');
        charMap.put('}', '{');
        charMap.put(']', '[');

        Set<Character> leftChars = new HashSet<>(charMap.values());
        LinkedList<Character> stack = new LinkedList<>();

        char[] chars = s.toCharArray();
        for (char ch : chars) {
            if (leftChars.contains(ch)) {
                stack.push(ch);
            } else if (stack.isEmpty() || !stack.pop().equals(charMap.get(ch))) {
                return false;
            }
        }

        return stack.isEmpty();
    }
}
```

### **Python3**

<!-- 这里可写当前语言的特殊实现逻辑 -->

```python
class Solution:
    def isValid(self, s: str) -> bool:
        if not s:
            return False

        d = {')': '(', '}': '{', ']': '['}
        left_chars = set(d.values())

        stack = []
        for char in s:
            if char in left_chars:
                stack.append(char)
            elif not stack or stack.pop() != d[char]:
                return False

        return not stack
```

<!-- tabs:end -->