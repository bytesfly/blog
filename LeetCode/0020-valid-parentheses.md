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