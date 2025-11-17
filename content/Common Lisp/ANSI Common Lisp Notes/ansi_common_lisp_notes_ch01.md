---
title: "第一章：简介"
author: ["ubuntu"]
date: 2025-11-13T18:27:00+08:00
draft: false
---

## 第一章：简介 {#第一章-简介}

[链接](https://acl.readthedocs.io/en/latest/zhCN/ch1-cn.html)


### 1.1 新的工具 {#1-dot-1-新的工具}

写一个函数，输入一个数 n ，返回把 n 与传入参数 (argument)相加的函数。

```lisp
; Lisp
(defun addn (n)
  #'(lambda (x)
      (+ x n)))
```

[Lisp 词法闭包]({{< relref "lisp_词法闭包.md" >}})

有了宏、闭包以及运行期类型，Lisp 凌驾在面向对象程序设计之上。


### 1.2 新的技术 {#1-dot-2-新的技术}

Lisp 带来的新特性来说 ── 自动内存管理 (automatic memory management)，显式类型 (manifest typing)，闭包 (closures)等

Lisp 被设计成可扩展的：让你定义自己的操作符。
