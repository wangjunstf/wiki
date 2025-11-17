---
title: "第三章：列表"
author: ["ubuntu"]
date: 2025-11-13T15:15:00+08:00
draft: false
---

## 第三章：列表 {#第三章-列表}


### 3.1 构造（Conses） {#3-dot-1-构造-conses}

基本的 List 操作函数有：cons, car 以及 cdr。

-   cons：把两个对象结合成一个有两部分的对象，称之为 Cons 对象。

-   概念上来说，一个 Cons 是一对指针；第一个是 car ，第二个是 cdr。

Cons 对象提供了一个方便的表示法，来表示任何类型的对象。一个 Cons 对象里的一对指针，可以指向任何类型的对象，包括 Cons 对象本身。它利用到我们之后可以用 cons 来构造列表的可能性。

```lisp
(setf x (cons 'a nil))
(princ x)
```

```text
(A)
```

<a id="figure--fig:Figure-3.1.list"></a>

{{< figure src="/images/ansi_common_lisp_notes_3_1_list.png" >}}

产生的列表由一个 Cons 所组成，如图 [1](#figure--fig:Figure-3.1.list)，这种表达 Cons 的方式叫做箱子表示法 (box notation)，因为每一个 Cons 是用一个箱子表示，内含一个 car 和 cdr 的指针。当我们调用 car 与 cdr 时，我们得到指针指向的地方：

```lisp
(princ (car x))
(print (cdr x))
```

```text
A
NIL
```

当我们构造一个多元素的列表时，我们得到一串 Cons (a chain of conses):

```lisp
(setf y (list 'a 'b 'c))
(princ y)
```

```text
(A B C)
```

产生的结构如下图 [1](#figure--fig:Figure-3.1.threelist), 现在当我们想得到列表的 cdr 时，它是一个两个元素的列表。

<a id="figure--fig:Figure-3.1.threelist"></a>

{{< figure src="/images/ansi_common_lisp_notes_3_1_threelist.png" >}}

```lisp
(princ (cdr y))
```

```text
(B C)
```

在一个有多个元素的列表中， car 指针让你取得元素，而 cdr 让你取得列表内其余的东西。

一个列表可以有任何类型的对象作为元素，包括另一个列表：

```lisp
(setf z (list 'a (list 'b 'c) 'd))
(princ z)
```

```text
(A (B C) D)
```

{{< figure src="/images/ansi_common_lisp_notes_3_1_嵌套列表.png" >}}

对于上图中的列表：

```lisp
(princ (car (cdr z)))
```

```text
(B C)
```

前两个我们构造的列表都有三个元素；只不过 z 列表的第二个元素也刚好是一个列表。像这样的列表称为嵌套列表，而像 y 这样的列表称之为平坦列表 (flatlist)。

如果参数是一个 Cons 对象，函数 consp 返回真。所以我们可以这样定义 listp :

```lisp
(defun our-listp (x)
  (or (null x) (consp x)))
```

因为所有不是 Cons 对象的东西，就是一个原子 (atom)，判断式 atom 可以这样定义：

```lisp
(defun our-atom (x) (not (consp x)))
(princ (our-atom 1))
(print (our-atom nil))
(print (our-atom '(1 2 3)))
(print (our-atom '(nil)))
```

```text
T
T
NIL
NIL
```


### 3.2 等式（Equality） {#3-dot-2-等式-equality}

每一次你调用 cons 时， Lisp 会配置一块新的内存给两个指针。所以如果我们用同样的参数调用 cons 两次，我们得到两个数值看起来一样，但实际上是两个不同的对象：

```lisp
(princ (eql (cons 'a nil) (cons 'a nil)))
```

```text
NIL
```

equal 可以判断两个列表是否有相同的元素。eql 只有在它的参数是相同对象时才返回真。

```lisp
(setf x (cons 'a nil))
(princ (equal x (cons 'a nil)))
```

```text
T
```

本质上 equal 若它的参数打印出的值相同时，返回真：

以下是简化的 equal 实现，仅对列表有效：

```lisp
(defun our-equal (x y)
  (or (eql x y)
      (and (consp x)
           (consp y)
           (our-equal (car x) (car y))
           (our-equal (cdr x) (cdr y)))))

(princ (our-equal '(a b c)  '(a b c)))
(print (our-equal '(a b c (a b c))  '(a b c (a b c))))
(print (our-equal '(a b c (a b c))  '(a b c (a b))))
```

```text
T
T
NIL
```


### 3.3 为什么 Lisp 没有指针 (Why Lisp Has No Pointers) {#3-dot-3-为什么-lisp-没有指针--why-lisp-has-no-pointers}

一个理解 Lisp 的秘密之一是意识到变量是有值的，就像列表有元素一样。如同 Cons 对象有指针指向他们的元素，变量有指针指向他们的值。

你可能在别的语言中使用过显式指针 (explicitly pointer)。在 Lisp，你永远不用这么做，因为语言帮你处理好指针了。我们已经在列表看过这是怎么实现的。同样的事情发生在变量身上。举例来说，假设我们想要把两个变量设成同样的列表：

```lisp
(setf x '(a b c))
(princ x)
(setf y x)
(print y)
```

```text
(A B C)
(A B C)
```

{{< figure src="/images/ansi_common_lisp_notes_3_1_XY指向同一个列表.png" >}}

当我们给 y 赋一个相同的值时， Lisp 复制的是指针，而不是列表。无论何时，你将某个变量的值赋给另个变量时，两个变量的值将会是 eql 的：

```lisp
(princ (eql x y))
```

```text
T
```

Lisp 没有指针的原因是因为每一个值，其实概念上来说都是一个指针。当你赋一个值给变量或将这个值存在数据结构中，其实被储存的是指向这个值的指针。当你要取得变量的值，或是存在数据结构中的内容时， Lisp 返回指向这个值的指针。但这都在台面下发生。你可以不加思索地把值放在结构里，或放“在”变量里。

为了效率的原因， Lisp 有时会选择一个折衷的表示法，而不是指针。举例来说，因为一个小整数所需的内存空间，少于一个指针所需的空间，一个 Lisp 实现可能会直接处理这个小整数，而不是用指针来处理。但基本要点是，程序员预设可以把任何东西放在任何地方。除非你声明你不愿这么做，不然你能够在任何的数据结构，存放任何类型的对象，包括结构本身。


### 3.4 建立列表（Building Lists） {#3-dot-4-建立列表-building-lists}

函数 copy-list 接受一个列表，然后返回此列表的复本。新的列表会有同样的元素，但是装在新的 Cons 对象里：

{{< figure src="/images/ansi_common_lisp_notes_3_1_列表复制.png" >}}

```lisp
(setf x '(a b c)
      y (copy-list x))

(princ x)
(print y)
```

```text
(A B C)
(A B C)
```

下面是一个 copy-list 的简化实现：

```lisp
(defun our-copy-list (lst)
  (if (atom lst)
      lst
      (cons (car lst) (our-copy-list (cdr lst)))))
```

这个定义暗示着 x 与 (copy-list x) 会永远 equal ，并永远不 eql ，除非 x 是 NIL 。

最后，函数 append 返回任何数目的列表串接 (concatenation)：如果最后一个参数不是列表，它会直接附加在最后，而不是被当作列表展开：

```lisp
(princ (append '(a b) 'e))
(print (append '(1 2) '(3 4)))
(print (append '(x y) '(nil) '(z h) 'i))
```

```text
(A B . E)
(1 2 3 4)
(X Y NIL Z H . I)
```

对于上述第一条输出 (A B . E)
它的内部结构是 (cons A (cons B E))

对于上述第二条输出 (1 2 3 4)
它的内部结构是 (cons 1 (cons 2 (cons 3 (cons 4 nil))))

```lisp
(princ (cons 'A (cons 'B 'E)))
(print (cons 1 (cons 2 (cons 3 (cons 4 nil)))))
```

```text
(A B . E)
(1 2 3 4)
```


### 3.5 示例：压缩(Example: Compression) {#3-dot-5-示例-压缩--example-compression}

作为一个例子，这节将演示如何实现简单形式的列表压缩。这个算法有一个令人印象深刻的名字，游程编码(run-length encoding)。

```lisp
(defun compress (x)
  (if (consp x)
      (compr (car x) 1 (cdr x))
      x))

(defun compr (elt n lst)
  (if (null lst)
      (list (n-elts elt n))
      (let ((next (car lst)))
        (if (eql next elt)
            (compr elt (+ n 1) (cdr lst))
            (cons (n-elts elt n)
                  (compr next 1 (cdr lst)))))))

(defun n-elts (elt n)
  (if (> n 1)
      (list n elt)
      elt))

```

函数 compress 接受一个由原子组成的列表，然后返回一个压缩的列表：

```lisp
(princ (compress '(1 1 1 0 1 0 0 0 0 1)))
```

```text
((3 1) 0 1 (4 0) 1)
```

当相同的元素连续出现好几次，这个连续出现的序列 (sequence)被一个列表取代，列表指明出现的次数及出现的元素。

主要的工作是由递归函数 compr 所完成。这个函数接受三个参数： elt ， 上一个我们看过的元素； n ，连续出现的次数；以及 lst ，我们还没检查过的部分列表。如果没有东西需要检查了，我们调用 n-elts 来取得 n elts 的表示法。如果 lst 的第一个元素还是 elt ，我们增加出现的次数 n 并继续下去。否则我们得到，到目前为止的一个压缩的列表，然后 cons 这个列表在 compr 处理完剩下的列表所返回的东西之上。

```lisp
(defun uncompress (lst)
  (if (null lst)
      nil
      (let ((elt (car lst))
            (rest (uncompress (cdr lst))))
        (if (consp elt)
            (append (apply #'list-of elt)
                    rest)
            (cons elt rest)))))

(defun list-of (n elt)
  (if (zerop n)
      nil
      (cons elt (list-of (- n 1) elt))))
```

复原一个压缩的列表，我们调用 uncompress

```lisp
(princ (uncompress '((3 1) 0 1 (4 0) 1)))
```

```text
(1 1 1 0 1 0 0 0 0 1)
```

list-of 函数逐字复制原子并调用 list-of ，展开成列表。

```lisp
(princ (list-of 3 'ho))
(print (apply #'list-of '(3 *)))
```

```text
(HO HO HO)
(* * *)
```

我们其实不需要自己写 list-of 。内置的 make-list 可以办到一样的事情 ── 但它使用了我们还没介绍到的关键字参数 (keyword argument)。


### 3.6 存取（Access） {#3-dot-6-存取-access}

Common Lisp 有额外的存取函数，它们是用 car 跟 cdr 所定义的。要找到列表特定位置的元素，我们可以调用 nth ，

```lisp
(princ (nth 0 '(a b c)))     ;; 找到第0个元素
(print (nth 1 '(a b c)))     ;; 找到第1个素元
(print (nth 2 '(a b c)))     ;; 找到第2个元素
```

```text
A
B
C
```

```lisp
(princ (nthcdr 2 '(a b c)))
```

```text
(C)
```

nth 与 nthcdr 都是零索引的 (zero-indexed); 即元素从 0 开始编号，而不是从 1 开始。在 Common Lisp 里，无论何时你使用一个数字来参照一个数据结构中的元素时，都是从 0 开始编号的。

两个函数几乎做一样的事; nth 等同于取 nthcdr 的 car 。

nthcdr 的简化实现：

```lisp
(defun our-nthcdr (n lst)
  (if (zerop n)
      lst
      (our-nthcdr (- n 1) (cdr lst))))

(princ (our-nthcdr 2 '(a b c d)))
```

```text
(C D)
```

函数 zerop 仅在参数为零时，才返回真。

函数 last 返回列表的最后一个 Cons 对象：

```lisp
(princ (last '(a b c)))
```

```text
(C)
```

Common Lisp 定义了函数 first 直到 tenth 可以取得列表对应的元素。这些函数不是 零索引的 (zero-indexed)：

(second x) 等同于 (nth 1 x) 。

```lisp
(princ (first '(a b c d)))
(print (second '(a b c d)))
(print (third '(a b c d)))
```

```text
A
B
C
```

此外， Common Lisp 定义了像是 caddr 这样的函数，它是 cdr 的 cdr 的 car 的缩写 ( car of cdr of cdr )。所有这样形式的函数 cxr ，其中 x 是一个字符串，最多四个 a 或 d ，在 Common Lisp 里都被定义好了。使用 cadr 可能会有异常 (exception)产生，在所有人都可能会读的代码里使用这样的函数，可能不是个好主意。

```lisp
(princ (caddr '(a b c d e)))
```

```text
C
```


### 3.7 映射函数（Mapping Functions） {#3-dot-7-映射函数-mapping-functions}

Common Lisp 提供了数个函数来对一个列表的元素做函数调用。最常使用的是 mapcar ，接受一个函数以及一个或多个列表，并返回把函数应用至每个列表的元素的结果，直到有的列表没有元素为止：

```lisp
(princ (mapcar #'(lambda (x) (+ x 10))
               '(1 2 3)))

(print (mapcar #'list
               '(a b c)
               '(1 2 3 4)))

```

```text
(11 12 13)
((A 1) (B 2) (C 3))
```

相关的 maplist 接受同样的参数，将列表的渐进的下一个 cdr 传入函数。

```lisp
(princ (maplist #'(lambda (x) x)
           '(a b c)))
```

```text
((A B C) (B C) (C))
```


### 3.8 树（Trees） {#3-dot-8-树-trees}

Cons 对象可以想成是二叉树， car 代表左子树，而 cdr 代表右子树。举例来说，列表

(a (b c) d) 也是一棵由下图所代表的树。 （如果你逆时针旋转 45 度，你会发现跟图 [1](#figure--fig:Figure-3.1.threelist) 一模一样）

{{< figure src="/images/ansi_common_lisp_notes_3_1_二叉树.png" >}}

Common Lisp 有几个内置的操作树的函数。举例来说， copy-tree 接受一个树，并返回一份副本。它的简化定义如下：

```lisp
(defun our-copy-tree (tr)
  (if (atom tr)
      tr
      (cons (our-copy-tree (car tr))
            (our-copy-tree (cdr tr)))))
```

与前文3.4节提到的 copy-list 比较一下； copy-tree 复制每一个 Cons 对象的 car 与 cdr ，而 copy-list 仅复制 cdr 。

没有内部节点的二叉树没有太大的用处。 Common Lisp 包含了操作树的函数，不只是因为我们需要树这个结构，而是因为我们需要一种方法，来操作列表及所有内部的列表。举例来说，假设我们有一个这样的列表：

```lisp
(and (integerp x) (zerop (mod x 2)))
```

而我们想要把各处的 x 都换成 y 。调用 substitute 是不行的，它只能替换序列 (sequence)中的元素：

```lisp
(princ (substitute 'y 'x '(and (integerp x) (zerop (mod x 2)))))
```

```text
(AND (INTEGERP X) (ZEROP (MOD X 2)))
```

这个调用是无效的，因为列表有三个元素，没有一个元素是 x 。我们在这所需要的是 subst ，它替换树之中的元素。

```lisp
(princ (subst 'y 'x '(and (integerp x) (zerop (mod x 2)))))
```

```text
(AND (INTEGERP Y) (ZEROP (MOD Y 2)))
```

如果我们定义一个 subst 的版本，它看起来跟 copy-tree 很相似：

```lisp
(defun our-subst (new old tree)
  (if (eql tree old)
      new
      (if (atom tree)
          tree
          (cons (our-subst new old (car tree))
                (our-subst new old (cdr tree))))))

(princ (our-subst 'y 'x '(and (integerp x) (zerop (mod x 2))) ))
```

```text
(AND (INTEGERP Y) (ZEROP (MOD Y 2)))
```

操作树的函数通常有这种形式， car 与 cdr 同时做递归。这种函数被称之为是 双重递归 (doubly recursive)。


### 3.9 理解递归 (Understanding Recursion) {#3-dot-9-理解递归--understanding-recursion}

要知道一个递归函数是否做它该做的事，你只需要问，它包含了所有的情况吗？举例来说，下面是一个寻找列表长度的递归函数：

```lisp
  (defun len (lst)
    (if (null lst)
        0
        (+ (len (cdr lst)) 1)))


(princ (len '(1 2 3 4 5)))
```

```text
5
```

我们可以借由检查两件事情，来确信这个函数是正确的：

1.  对长度为 0 的列表是有效的。

2.  给定它对于长度为 n 的列表是有效的，它对长度是 n+1 的列表也是有效的。

我们的定义显然地满足第一点：如果列表( lst ) 是空的( nil )，函数直接返回 0 。现在假定我们的函数对长度为 n 的列表是有效的。我们给它一个 n+1 长度的列表。这个定义说明了，函数会返回列表的 cdr 的长度再加上 1 。 cdr 是一个长度为 n 的列表。我们经由假定可知它的长度是 n 。所以整个列表的长度是 n+1 。

第一个情况（长度零的列表）称之为基本用例( base case )。当一个递归函数不像你想的那样工作时，通常是处理基本用例就错了。下面这个不正确的 member 定义，是一个常见的错误，整个忽略了基本用例：

```lisp
(defun our-member (obj lst)
  (if (eql (car lst) obj)
      lst
      (our-member obj (cdr lst))))
```

我们需要初始一个 null 测试，确保在到达列表底部时，没有找到目标时要停止递归。如果我们要找的对象没有在列表里，这个版本的 member 会陷入无穷循环。

正确版本：

```lisp
(defun our-member (obj lst)
  (if (null lst)
      nil
      (if (eql (car lst) obj)
          lst
          (our-member obj (cdr lst)))))

(princ (our-member 3 '(1 2 3 4 5 6 7)))
```

```text
(3 4 5 6 7)
```


### 3.10 集合 (Sets) {#3-dot-10-集合--sets}

列表是表示小集合的好方法。列表中的每个元素都代表了一个集合的成员：

```lisp
(princ (member 'b '(a b c)))
```

```text
(B C)
```

当 member 要返回“真”时，与其仅仅返回 t ，它返回由寻找对象所开始的那部分。逻辑上来说，一个 Cons 扮演的角色和 t 一样，而经由这么做，函数返回了更多资讯。

-   一般情况下， member 使用 eql 来比较对象。

-   多数的 Common Lisp 函数接受一个或多个关键字参数。

-   一个 member 函数所接受的关键字参数是 :test 参数。

如果我们想找到一个给定的对象与列表中的成员是否相等( equal )：

```lisp
(princ (member '(a) '((a) (z)) :test #'equal))
```

```text
((A) (Z))
```

另一个 member 接受的关键字参数是 :key 参数。借由提供这个参数，你可以在作比较之前，指定一个函数运用在每一个元素：

```lisp
(princ (member 'a '((a b) (c d)) :key #'car))
```

```text
((A B) (C D))
```

在这个例子里，我们询问是否有一个元素的 car 是 a 。

如果我们想要使用两个关键字参数，我们可以使用其中一个顺序。下面这两个调用是等价的：

```lisp
(princ (member 2 '((1) (2)) :key #'car :test #'equal))
(print (member 2 '((1) (2)) :test #'equal :key #'car))
```

```text
((2))
((2))
```

两者都询问是否有一个元素的 car 等于( equal ) 2。

如果我们想要找到一个元素满足任意的判断式像是── oddp ，奇数返回真──我们可以使用相关的 member-if ：

```lisp
(princ (member-if #'oddp '(2 3 4)))
```

```text
(3 4)
```

我们可以想像一个限制性的版本 member-if 是这样写成的：

```lisp
(defun our-member-if (fn lst)
  (and (consp lst)
       (if (funcall fn (car lst))
           lst
           (our-member-if fn (cdr lst)))))
```

函数 adjoin 像是条件式的 cons 。它接受一个对象及一个列表，如果对象还不是列表的成员，才构造对象至列表上。

```lisp
(princ (adjoin 'b '(a b c)))
(print (adjoin 'z '(a b c)))
```

```text
(A B C)
(Z A B C)
```

通常的情况下它接受与 member 函数同样的关键字参数。

集合论中的并集 (union)、交集 (intersection)以及补集 (complement)的实现，是由函数 union 、 intersection 以及 set-difference 。

这些函数期望两个（正好 2 个）列表（一样接受与 member 函数同样的关键字参数）。

```lisp
(princ (union '(a b c) '(c b s)))
(print (intersection '(a b c) '(b b c)))
(print (set-difference '(a b c d e) '(b e)))
```

```text
(A C B S)
(C B)
(D C A)
```

因为集合中没有顺序的概念，这些函数不需要保留原本元素在列表被找到的顺序。举例来说，调用 set-difference 也有可能返回 (d c a) 。


### 3.11 序列（Sequences） {#3-dot-11-序列-sequences}

在 Common Lisp 里，序列( sequences )包括了列表与向量 (vectors)。

函数 length 返回序列中元素的数目。

```lisp
(princ (length '(a b c)))
```

```text
3
```

要复制序列的一部分，我们使用 subseq 。第二个（需要的）参数是第一个开始引用进来的元素位置，第三个（选择性）参数是第一个不引用进来的元素位置。

```lisp
(princ (subseq '(a b c d) 1 2))
(print (subseq '(a b c d) 1))
```

```text
(B)
(B C D)
```

如果省略了第三个参数，子序列会从第二个参数给定的位置引用到序列尾端。

函数 reverse 返回与其参数相同元素的一个序列，但顺序颠倒。

```lisp
(princ (reverse '(a b c)))
```

```text
(C B A)
```

一个回文 (palindrome) 是一个正读反读都一样的序列 —— 举例来说， (abba) 。如果一个回文有偶数个元素，那么后半段会是前半段的镜射 (mirror)。使用 length 、 subseq 以及 reverse ，我们可以定义一个函数

```lisp
(defun mirror? (s)
  (let ((len (length s)))
    (and (evenp len)
         (let ((mid (/ len 2)))
           (equal (subseq s 0 mid)
                  (reverse (subseq s mid)))))))

(princ (mirror? '(a b b a)))
```

```text
T
```

Common Lisp 有一个内置的排序函数叫做 sort 。它接受一个序列及一个比较两个参数的函数，返回一个有同样元素的序列，根据比较函数来排序：

```lisp
(princ (sort '(0 2 1 3 8) #'>))
```

```text
(8 3 2 1 0)
```

你要小心使用 sort ，因为它是破坏性的(destructive)。考虑到效率的因素， sort 被允许修改传入的序列。所以如果你不想你本来的序列被改动，传入一个副本。

使用 sort 及 nth ，我们可以写一个函数，接受一个整数 n ，返回列表中第 n 大的元素：

```lisp
(defun nthmost (n lst)
  (nth (- n 1)
       (sort (copy-list lst) #'>)))

(princ (nthmost 2 '(0 2 1 3 8)))
```

```text
3
```

我们把整数减一因为 nth 是零索引的，但如果 nthmost 是这样的话，会变得很不直观。

函数 every 和 some 接受一个判断式及一个或多个序列。当我们仅输入一个序列时，它们测试序列元素是否满足判断式：

```lisp
(princ (every #'oddp '(1 3 5)))
(print (some #'evenp '(1 2 3)))
```

```text
T
T
```

如果它们输入多于一个序列时，判断式必须接受与序列一样多的元素作为参数，而参数从所有序列中一次提取一个：

```lisp
(princ (every #'> '(1 3 5) '(0 2 4)))
```

```text
T
```


### 3.12 栈（Stacks） {#3-dot-12-栈-stacks}

用 Cons 对象来表示的列表，很自然地我们可以拿来实现下推栈 (pushdown stack)。这太常见了，以致于 Common Lisp 提供了两个宏给堆使用： (push x y) 把 x 放入列表 y 的前端。而 (pop x) 则是将列表 x 的第一个元素移除，并返回这个元素。

两个函数都是由 setf 定义的。如果参数是常数或变量，很简单就可以翻译出对应的函数调用。

表达式

(push obj lst)

等同于

(setf lst (cons obj lst))

而表达式

(pop lst)

等同于

```lisp
(let ((x (car lst)))
  (setf lst (cdr lst))
  x)
```

```lisp
(setf x '(b))
(princ (push 'a x))
(print x)
(print (setf y x))
(print (pop x))
(print x)
(print y)
```

```text
(A B)
(A B)
(A B)
A
(B)
(A B)
```

以上，全都遵循上述由 setf 所给出的相等式。下图展示了这些表达式被求值后的结构。

{{< figure src="/images/push-pop.png" >}}

你可以使用 push 来定义一个给列表使用的互动版 reverse 。

```lisp
(defun our-reverse (lst)
  (let ((acc nil))
    (dolist (elt lst)
      (push elt acc))
    acc))

(print (our-reverse '(a b c)))
```

```text

(C B A)
```

在这个版本，我们从一个空列表开始，然后把 lst 的每一个元素放入空表里。等我们完成时，lst 最后一个元素会在最前端。

pushnew 宏是 push 的变种，使用了 adjoin 而不是 cons ：

```lisp
(let ((x '(a b)))
  (pushnew 'c x)
  (pushnew 'a x)
  (princ x))
```

```text
(C A B)
```

在这里， c 被放入列表，但是 a 没有，因为它已经是列表的一个成员了。


### 3.13 点状列表 (Dotted Lists) {#3-dot-13-点状列表--dotted-lists}

调用 list 所构造的列表，这种列表精确地说称之为正规列表(properlist )。一个正规列表可以是 NIL 或是 cdr 是正规列表的 Cons 对象。也就是说，我们可以定义一个只对正规列表返回真的判断式：

```lisp
(defun proper-list? (x)
  (or (null x)
      (and (consp x)
           (proper-list? (cdr x)))))
```

然而， cons 不仅是构造列表。无论何时你需要一个具有两个字段 (field)的列表，你可以使用一个 Cons 对象。你能够使用 car 来参照第一个字段，用 cdr 来参照第二个字段。

```lisp
(setf pair (cons 'a 'b))
(princ pair)
```

```text
(A . B)
```

因为这个 Cons 对象不是一个正规列表，它用点状表示法来显示。在点状表示法，每个 Cons 对象的 car 与 cdr 由一个句点隔开来表示。这个 Cons 对象的结构展示下图：

{{< figure src="/images/一个成对的cons对象.png" >}}

一个非正规列表的 Cons 对象称之为点状列表 (dotted list)。这不是个好名字，因为非正规列表的 Cons 对象通常不是用来表示列表： (a . b) 只是一个有两部分的数据结构。

你也可以用点状表示法表示正规列表，但当 Lisp 显示一个正规列表时，它会使用普通的列表表示法：

```lisp
(princ '(a . (b . (c . nil))))
```

```text
(A B C)
```

还有一个过渡形式 (intermediate form)的表示法，介于列表表示法及纯点状表示法之间，对于 cdr 是点状列表的 Cons 对象：

```lisp
(princ (cons 'a (cons 'b (cons 'c 'd))))
```

```text
(A B C . D)
```

{{< figure src="/images/一个点状列表.png" >}}

这样的 Cons 对象看起来像正规列表，除了最后一个 cdr 前面有一个句点。这个列表的结构展示上图 ; 注意它跟图 [1](#figure--fig:Figure-3.1.threelist) 是多么的相似。
