---
title: "Leetcode 刷题模版"
date: 2021-11-04T16:59:07+09:00
draft: true
author: Torres
tags: ["Leetcode", "Algorithm"]
categories: ["Algorithm"]
toc:
  enable: true
  auto: true
share:
  enable: true
comment:
  enable: true

---

## 树🌲

遍历树的数据结构：

```python
class TreeNode:
  def __init__(self, x):
    self.val = x
    self.left = None
    self.right = None

def traverse(root: TreeNode):
  # 前序遍历的位置
  traverse(root.left)
  # 中序遍历的位置
  traverse(root.right)
  # 后序遍历的位置
```

