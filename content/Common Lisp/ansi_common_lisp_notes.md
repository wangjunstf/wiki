---
title: "ANSI Common Lisp Notes"
author: ["ubuntu"]
date: 2025-11-04T20:05:00+08:00
draft: false
---

[ANSI Common Lisp 中文版](https://acl.readthedocs.io/en/latest/index.html)

对于没有输出的表达式，org 并不会捕获输出，而需要明确使用 print 或 format。为了方便输出特别定义宏 output（op）

```lisp
(defmacro op (&rest expressions)
  `(progn
     ,@(loop for expr in expressions
             collect `(format t "~A~%" ,expr))))

(op
 (+ 12 12 )
 (- 100 1))
```

```text
24
99
```


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


## 第二章：欢迎来到 Lisp {#第二章-欢迎来到-lisp}


### 2.1 形式（Form） {#2-dot-1-形式-form}

任何 Lisp 系统都含有一个交互式的前端，叫做顶层(toplevel)。你在顶层输入 Lisp 表达式，而系统会显示它们的值。

```lisp
(format t "Hello World")
```

```text
Hello World
```


### 2.2 求值（Evaluation） {#2-dot-2-求值-evaluation}

下面解释表达式是如何被求值的，以 (+ 2 4) 举例：

1.  首先从左至右对实参求值。在这个例子当中，实参对自身求值，所以实参的值分别是 2 跟 3 。

2.  实参的值传入以操作符命名的函数。在这个例子当中，将 2 跟 3 传给 + 函数，返回 5 。

对于函数调用来说，由左至右对实参求值，将它们的数值传入函数，来返回整个表达式的值。这称为 Common Lisp 的求值规则。

一个不遵守 Common Lisp 求值规则的操作符是 quote。quote 是一个特殊的操作符，它的规则就是什么也不做，原封不动地返回：

```lisp
(format t "~A~%" (quote (+ 3 5)))

;; ' 是 quote 的缩写
(format t "~A~%" '(+ 3 5))
```

```text
(+ 3 5)
(+ 3 5)
```

Lisp 提供 quote 作为一种保护表达式不被求值的方式。


### 2.3 数据（Data） {#2-dot-3-数据-data}

Lisp 支持常见的数据类型，例如整数和字符串。除此之外，Lisp 还具有一些特别的数据类型，例如符号(symbol)与列表(lists)。

符号是英语的单词 (words)。无论你怎么输入，通常会被转换为大写：

```lisp
(format t "~A~%" 'Artichoke)
```

```text
ARTICHOKE
```

列表是由被括号包住的零个或多个元素来表示。元素可以是任何类型，包含列表本身。使用列表必须要引用，不然 Lisp 会以为这是个函数调用：

```lisp
(format t "~A~%" '(my 3 "Sons"))
```

```text
(MY 3 Sons)
```

可以调用 list 来创建列表。由于 list 是函数，所以它的实参会被求值。

```lisp
(pprint (list 'my (+ 2 1) "Sons"))
```

```text

(MY 3 "Sons")
```

我们现在来到领悟 Lisp 最卓越特性的地方之一。Lisp 程序可以写出 Lisp 代码。 Lisp 程序员可以（并且经常）写出能为自己写程序的程序。

如果一个列表被引用了，则求值规则对列表自身来求值；如果没有被引用，则列表被视为是代码，依求值规则对列表求值后，返回它的值。

```lisp
(pprint (list '(+ 2 1) (+ 2 1)))
```

```text

((+ 2 1) 3)
```

这里第一个实参被引用了，所以产生一个列表。第二个实参没有被引用，视为函数调用，经求值后得到一个数字。

在 Common Lisp 里有两种方法来表示空列表。你可以用一对不包括任何东西的括号来表示，或用符号 nil 来表示空表。


### 2.4 列表操作（List Operations） {#2-dot-4-列表操作-list-operations}

用函数 cons 来构造列表。如果传入的第二个实参是列表，则返回由两个实参所构成的新列表，新列表为第一个实参加上第二个实参：

```lisp
(princ (cons 'a '(b c d)))
```

```text
(A B C D)
```

可以通过把新元素建立在空表之上，来构造一个新列表。上一节所看到的函数 list ，不过就是一个把几个元素加到 nil 上的快捷方式：

```lisp
(princ (cons 'a (cons 'b nil)))
(print (list 'a 'b))
```

```text
(A B)
(A B)
```

取出列表元素的基本函数是 car 和 cdr 。对列表取 car 返回第一个元素，而对列表取 cdr 返回第一个元素之后的所有元素：

```lisp
(princ (car '(a b c)))
(print (cdr '(a b c)))
```

```text
A
(B C)
```

你可以把 car 与 cdr 混合使用来取得列表中的任何元素。如果我们想要取得第三个元素，我们可以：

```lisp
(princ (car (cdr (cdr '(a b c d)))) )
```

```text
C
```

不过，你可以用更简单的 third 来做到同样的事情：

```lisp
(princ (third '(a b c d)))
```

```text
C
```


### 2.5 真与假（Truth） {#2-dot-5-真与假-truth}

在 Common Lisp 里，符号 t 是表示逻辑 真 的缺省值。与 nil 相同， t 也是对自身求值的。如果实参是一个列表，则函数 listp 返回 真 ：

```lisp
(princ (listp '(a b c)))
```

```text
T
```

函数的返回值将会被解释成逻辑 真 或逻辑 假 时，则称此函数为谓词（predicate）。在 Common Lisp 里，谓词的名字通常以 p 结尾。

逻辑 假 在 Common Lisp 里，用 nil ，即空表来表示。如果我们传给 listp 的实参不是列表，则返回 nil 。

```lisp
(princ (listp 27))
```

```text
NIL
```

由于 nil 在 Common Lisp 里扮演两个角色，如果实参是一个空表，则函数 null 返回 真 。

```lisp
(princ (null nil))
```

```text
T
```

而如果实参是逻辑 假 ，则函数 not 返回 真 ：

```lisp
(princ (not nil))
```

```text
T
```

在 Common Lisp 里，最简单的条件式是 if 。通常接受三个实参：一个 test 表达式，一个 then 表达式和一个 else 表达式。若 test 表达式求值为逻辑 真 ，则对 then 表达式求值，并返回这个值。若 test 表达式求值为逻辑 假 ，则对 else 表达式求值，并返回这个值：

```lisp
(princ (if (listp '(a b c))
      (+ 1 2)
      (+ 5 6)))
```

```text
3
```

```lisp
(princ (if (listp 27)
    (+ 1 2)
    (+ 5 6)))
```

```text
11
```

与 quote 相同， if 是特殊的操作符。不能用函数来实现，因为实参在函数调用时永远会被求值，而 if 的特点是，只有最后两个实参的其中一个会被求值。 if 的最后一个实参是选择性的。如果忽略它的话，缺省值是 nil ：

```lisp
(princ (if (listp 27)
   (+ 1 2)))
```

```text
NIL
```

虽然 t 是逻辑 真 的缺省表示法，任何非 nil 的东西，在逻辑的上下文里通通被视为 真 。

```lisp
(princ (if 27 1 2))
```

```text
1
```

逻辑操作符 and 和 or 与条件式类似。两者都接受任意数量的实参，但仅对能影响返回值的几个实参求值。如果所有的实参都为 真 （即非 nil ），那么 and 会返回最后一个实参的值：

```lisp
(princ (and t (+ 1 2)))
```

```text
3
```

如果其中一个实参为 假 ，那之后的所有实参都不会被求值。 or 也是如此，只要碰到一个为 真 的实参，就停止对之后所有的实参求值。

以上这两个操作符称为宏。宏和特殊的操作符一样，可以绕过一般的求值规则。

> 特殊操作符：quoto, if
>
> 宏：and, or
>
> 特殊操作符和宏的共同点有：可以绕过一般的求值规则。


### 2.6 函数（Functions） {#2-dot-6-函数-functions}

使用 defun 宏来定义新函数，通常接受三个实参：一个函数名、一组用列表表示的形参、以及一个或多个组成函数体的表达式。例如：这样定义 third：

```lisp
(defun our-third(x)
  (car (cdr (cdr x))))

(princ (our-third '(a b c d e f)))
```

```text
C
```

-   our-third 是函数名。
-   第二个实参，一个列表 (x) ,表示这个函数接受一个形参：x 。这样使用的占位符符号叫做变量。
-   剩余部分， (car (cdr (cdr x))) ，即所谓的函数主体。

> 在 Common Lisp中，符号是变量的名字，符号本身就是以对象的方式存在。
>
> [Common Lisp 符号|绑定|变量]({{< relref "common_lisp_符号_绑定_变量.md" >}})

例如：

```lisp
(defun sum-greater (x y z)
  (> (+ x y) z))

(princ (sum-greater 1 4 3))
```

```text
T
```

x,y,z 是三个符号，它们分别与 1,4,3绑定。

> Lisp 不对程序、过程以及函数作区别。函数做了所有的事情（事实上，函数是语言的主要部分）。


### 2.7 递归（Recursion） {#2-dot-7-递归-recursion}

函数可以调用任何函数，包括自己。自己调用自己的函数是递归的。

例如：

```lisp
(defun our-member (obj lst)
  (if (null lst)
      nil
      (if (eql (car lst) obj)
          lst
          (our-member obj (cdr lst)))))

(princ (our-member 'a '(a b c e f)))
(print (our-member 'e '(a b c e f)))
(print (our-member 'i '(a b c e f)))
```

```text
(A B C E F)
(E F)
NIL
```

-   our-member 是一个自定义函数，用于查找某个元素是否是位于给定列表中。

-   eql 是一个谓词，判断两个实参是否相等。

-   our-member 的执行过程：
    1.  首先检查 lst 列表是否为空列表。如果是空列表，那 obj 一定不是它的成员，结束。

    2.  否则，若 obj 是列表的第一个元素时，则它是列表的成员。

    3.  不然只有当 obj 是列表其余部分的元素时，它才是列表的成员。

<!--listend-->

```lisp
(defun our-member (obj lst)
  (if (null lst)
      nil
      (if (eql (car lst) obj)
          lst
          (our-member obj (cdr lst)))))
```


### 2.9 输入输出（Input and Output） {#2-dot-9-输入输出-input-and-output}

[Common Lisp 打印方法总结]({{< relref "lisp_打印方法总结.md" >}})

最普遍的 Common Lisp 输出函数是 format 。接受两个或两个以上的实参，第一个实参决定输出要打印到哪里，第二个实参是字符串模版，而剩余的实参，通常是要插入到字符串模版，用打印表示法（printed representation）所表示的对象。下面是一个典型的例子：

```lisp
(format t "~A plus ~A equals ~A. ~%" 2 3 (+ 2 3))
```

```text
2 plus 3 equals 5.
```

标准的输入函数是 read 。当没有实参时，会读取缺省的位置，通常是顶层。下面这个函数，提示使用者输入，并返回任何输入的东西：

```lisp
 ;; 读取一个S表达式（数字、符号、列表等）
 (let ((input (read)))
   (format t "你输入了: ~A" input))

 ;;
示例：输入 (1 2 3) 或 hello 或 42
```

记住 read 会一直永远等在这里，直到你输入了某些东西，并且（通常要）按下回车。因此，不打印明确的提示信息是很不明智的，程序会给人已经死机的印象，但其实它是在等待输入。

第二件关于 read 所需要知道的事是，它很强大： read 是一个完整的 Lisp 解析器（parser）。不仅是可以读入字符，然后当作字符串返回它们。它解析它所读入的东西，并返回产生出来的 Lisp 对象。在上述的例子，它返回一个数字。


### 2.10变量（Variables） {#2-dot-10变量-variables}

let 是一个最常用的 Common Lisp 的操作符之一，它让你引入新的局部变量（local variable）：

```lisp
(let ((x 1) (y 2))
  (+ x y))
```

一个 let 表达式有两个部分。第一个部分是一组创建新变量的指令，指令的形式为 (variable expression) 。每一个变量会被赋予相对应表达式的值。上述的例子中，我们创造了两个变量， x 和 y ，分别被赋予初始值 1 和 2 。这些变量只在 let 的函数体内有效。

一组变量与数值之后，是一个有表达式的函数体，表达式依序被求值。但这个例子里，只有一个表达式，调用 + 函数。最后一个表达式的求值结果作为 let 的返回值。以下是一个用 let 所写的，更有选择性的 askem 函数：

```lisp
(defun ask-number ()
  (format t "Please enter a number. ")
  (let ((val (read)))
    (if (numberp val)
        val
        (ask-number))))
```

let 创建局部变量，使用 defparameter 创建全局动态变量：

```lisp
(defparameter *glob* 99)
(princ *glob*)
```

```text
99
```

用 defconstant 来定义一个全局的常量：

```lisp
(defconstant limit (+ *glob* 1))
(princ limit)
```

```text
100
```

使用 boundp 函数检查是否为一个全局变量或常量：

```lisp
(princ (boundp '*glob*))
```

```text
T
```


### 2.11 赋值（Assignment） {#2-dot-11-赋值-assignment}

在 Common Lisp 里，最普遍的赋值操作符（assignment operator）是 setf 。可以用来给全局或局部变量赋值：

```lisp
(setf *glob* 98)

(let ((n 10))
  (setf n 2)
  (princ n))
```

```text
2
```

如果 setf 的第一个实参是符号（symbol），且符号不是某个局部变量的名字，则 setf 把这个符号设为全局变量：

```lisp
(setf x (list 'a 'b 'c))
(princ (boundp 'x))
```

```text
T
```

不仅可以给变量赋值。传入 setf 的第一个实参，还可以是表达式或变量名。在这种情况下，第二个实参的值被插入至第一个实参所引用的位置：

```lisp
(setf (car x) 'n)
(princ x)
```

```text
(N B C)
```

setf 的第一个实参几乎可以是任何引用到特定位置的表达式。可以给 setf 传入（偶数）个实参。一个这样的表达式

```lisp
(setf a 'b
    c 'd
    e 'f)
```


### 3.12 函数式编程 (Functional Programming) {#3-dot-12-函数式编程--functional-programming}

函数式编程意味着撰写利用返回值而工作的程序，而不是修改东西。它是 Lisp 的主流范式。大部分 Lisp 的内置函数被调用是为了取得返回值，而不是副作用。

举例来说，函数 remove 接受一个对象和一个列表，返回不含这个对象的新列表：

```lisp
(setf lst '(c a r a t))
(remove 'a lst)
```

上述代码并没有改变 lst 中的内容，如果要改变 lst 中的内容：

```lisp
(setf x (remove 'a x))
```

> 函数式编程最重要的优点之一是，它允许交互式测试（interactive testing）。在纯函数式的程序里，你可以测试每个你写的函数。如果它返回你预期的值，你可以有信心它是对的。这额外的信心，集结起来，会产生巨大的差别。当你改动了程序里的任何一个地方，会得到即时的改变。而这种即时的改变，使我们有一种新的编程风格。类比于电话与信件，让我们有一种新的通讯方式。


### 2.13 迭代（Iteration） {#2-dot-13-迭代-iteration}

当我们想重复做一些事情时，迭代比递归来得更自然。典型的例子是用迭代来产生某种表格。这个函数

```lisp
(defun show-squares (start end)
  (do ((i start (+ i 1)))
      ((> i end) 'done)
    (format t "~A ~A~%" i (* i i))))
```

列印从 start 到 end 之间的整数的平方：

```lisp
(show-squares 2 5)
```

```text
2 4
3 9
4 16
5 25
```

do 宏是 Common Lisp 里最基本的迭代操作符。和 let 类似， do 可以创建变量，而第一个实参是一组变量的规格说明列表。每个元素可以是以下的形式

(variable initial update)

其中 variable 是一个符号， initial 和 update 是表达式。最初每个变量会被赋予 initial 表达式的值；每一次迭代时，会被赋予 update 表达式的值。在 show-squares 函数里， do 只创建了一个变量 i 。第一次迭代时， i 被赋与 start 的值，在接下来的迭代里， i 的值每次增加 1 。

第二个传给 do 的实参可包含一个或多个表达式。第一个表达式用来测试迭代是否结束。在上面的例子中，测试表达式是 (&gt; i end) 。接下来在列表中的表达式会依序被求值，直到迭代结束。而最后一个值会被当作 do 的返回值来返回。所以 show-squares 总是返回 done 。

do 的剩余参数组成了循环的函数体。在每次迭代时，函数体会依序被求值。在每次迭代过程里，变量被更新，检查终止测试条件，接着（若测试失败）求值函数体。

作为对比，以下是递归版本的 show-squares ：

```lisp
(defun show-squares (i end)
  (if (> i end)
      'done
      (progn
        (format t "~A ~A~%" i (* i i))
        (show-squares (+ i 1) end))))

(princ (show-squares 1 3) )
```

```text
1 1
2 4
3 9
DONE
```

唯一的新东西是 progn 。 progn 接受任意数量的表达式，依序求值，并返回最后一个表达式的值。

为了处理某些特殊情况， Common Lisp 有更简单的迭代操作符。举例来说，要遍历列表的元素，你可能会使用 dolist 。以下函数返回列表的长度：

```lisp
(defun our-length (lst)
  (let ((len 0))
    (dolist (obj lst)
      (setf len (+ len 1)))
    len))
```

这里 dolist 接受这样形式的实参(variable expression)，跟着一个具有表达式的函数主体。函数主体会被求值，而变量相继与表达式所返回的列表元素绑定。因此上面的循环说，对于列表 lst 里的每一个 obj ，递增 len 。很显然这个函数的递归版本是：

```lisp
(defun our-length (lst)
  (if (null lst)
      0
      (+ (our-length (cdr lst)) 1)))
```


### 2.14 函数作为对象（Functions as Objects） {#2-dot-14-函数作为对象-functions-as-objects}

函数在 Lisp 里，和符号、字符串或列表一样，是稀松平常的对象。如果我们把函数的名字传给 function ，它会返回相关联的对象。和 quote 类似， function 是一个特殊操作符，所以我们无需引用（quote）它的实参：

```lisp
(princ (function +))
```

```text
#<FUNCTION +>
```

\#' 作为 function 的缩写：

```lisp
(princ #'+)
```

```text
#<FUNCTION +>
```

可以把函数当作实参传入。有个接受函数作为实参的函数是 apply 。apply 接受一个函数和实参列表，并返回把传入函数应用在实参列表的结果：

```lisp
(princ (apply #'+ '(1 2 3)))

(print (+ 1 2 3))
```

```text
6
6
```

apply 可以接受任意数量的实参，只要最后一个实参是列表即可：

```lisp
(princ (apply #'+ 1 2 '(3 4 5)))
```

```text
15
```

函数 funcall 做的是一样的事情，但不需要把实参包装成列表。

```lisp
(princ (funcall #'+ 1 2 3))
```

```text
6
```

> 什么是 lambda ？
>
> lambda 表达式里的 lambda 不是一个操作符。而只是个符号。 在早期的 Lisp 方言里， lambda 存在的原因是：由于函数在内部是用列表来表示， 因此辨别列表与函数的方法，就是检查第一个元素是否为 lambda 。
>
> 在 Common Lisp 里，你可以用列表来表达函数， 函数在内部会被表示成独特的函数对象。因此不再需要 lambda 了。 如果需要把函数记为
>
> ((x) (+ x 100))
>
> 而不是
>
> (lambda (x) (+ x 100))
>
> 但 Lisp 程序员习惯用符号 lambda ，来撰写函数， 因此 Common Lisp 为了传统，而保留了 lambda 。

defun 宏，创建一个函数并给函数命名。但函数不需要有名字，而且我们不需要 defun 来定义他们。和大多数的 Lisp 对象一样，我们可以直接引用函数。

要直接引用整数，我们使用一系列的数字；要直接引用一个函数，我们使用所谓的lambda 表达式。一个 lambda 表达式是一个列表，列表包含符号 lambda ，接着是形参列表，以及由零个或多个表达式所组成的函数体。

下面的 lambda 表达式，表示一个接受两个数字并返回两者之和的函数：

```lisp
(lambda (x y)
 (+ x y))
```

列表 (x y) 是形参列表，跟在它后面的是函数主体。

一个 lambda 表达式可以作为函数名。和普通的函数名称一样， lambda 表达式也可以是函数调用的第一个元素，

```lisp
((lambda (x) (+ x 100)) 1)
```

而通过在 lambda 表达式前面贴上 #' ，我们得到对应的函数，

```lisp
(princ (funcall #'(lambda (x) (+ x 100))
        1))
```

```text
101
```

lambda 表示法除上述用途以外，还允许我们使用匿名函数。


### 2.15 类型（Types） {#2-dot-15-类型-types}

Lisp 处理类型的方法非常灵活。在很多语言里，变量是有类型的，得声明变量的类型才能使用它。在 Common Lisp 里，数值才有类型，而变量没有。你可以想像每个对象，都贴有一个标明其类型的标签。这种方法叫做显式类型（manifest typing）。你不需要声明变量的类型，因为变量可以存放任何类型的对象。

虽然从来不需要声明类型，但出于效率的考量，你可能会想要声明变量的类型。类型声明在第 13.3 节时讨论。

Common Lisp 的内置类型，组成了一个类别的层级。对象总是不止属于一个类型。举例来说，数字 27 的类型，依普遍性的增加排序，依序是 fixnum 、 integer 、 rational 、 real 、 number 、 atom 和 t 类型。（数值类型将在第 9 章讨论。）类型 t 是所有类型的基类（supertype）。所以每个对象都属于 t 类型。

函数 typep 接受一个对象和一个类型，然后判定对象是否为该类型，是的话就返回真：

```lisp
(princ (typep 27 'integer))
```

```text
T
```


### 2.16 展望 (Looking Forward) {#2-dot-16-展望--looking-forward}

本章仅谈到 Lisp 的表面。然而，一种非比寻常的语言形象开始出现了。首先，这个语言用单一的语法，来表达所有的程序结构。语法基于列表，列表是一种 Lisp 对象。函数本身也是 Lisp 对象，函数能用列表来表示。而 Lisp 本身就是 Lisp 程序。几乎所有你定义的函数，与内置的 Lisp 函数没有任何区别。

如果你对这些概念还不太了解，不用担心。 Lisp 介绍了这么多新颖的概念，在你能驾驭它们之前，得花时间去熟悉它们。不过至少要了解一件事：在这些概念当中，有着优雅到令人吃惊的概念。

Richard Gabriel 曾经半开玩笑的说， C 是拿来写 Unix 的语言。我们也可以说， Lisp 是拿来写 Lisp 的语言。但这是两种不同的论述。一个可以用自己编写的语言和一种适合编写某些特定类型应用的语言，是有着本质上的不同。这开创了新的编程方法：你不但在语言之中编程，还把语言改善成适合程序的语言。如果你想了解 Lisp 编程的本质，理解这个概念是个好的开始。


### 总结 {#总结}

1.  p 是一种交互式语言。如果你在顶层输入一个表达式， Lisp 会显示它的值。

2.  p 程序由表达式组成。表达式可以是原子，或一个由操作符跟着零个或多个实参的列表。前序表示法代表操作符可以有任意数量的实参。

3.  mon Lisp 函数调用的求值规则： 依序对实参从左至右求值，接着把它们的值传入由操作符表示的函数。 quote 操作符有自己的求值规则，它完封不动地返回实参。

4.  一般的数据类型， Lisp 还有符号跟列表。由于 Lisp 程序是用列表来表示的，很轻松就能写出能编程的程序。

5.  基本的列表函数是 cons ，它创建一个列表； car ，它返回列表的第一个元素；以及 cdr ，它返回第一个元素之后的所有东西。

6.  Common Lisp 里， t 表示逻辑 真 ，而 nil 表示逻辑 假 。在逻辑的上下文里，任何非 nil 的东西都视为 真 。基本的条件式是 if 。 and 与 or 是相似的条件式。

7.  p 主要由函数所组成。可以用 defun 来定义新的函数。

8.  调用自己的函数是递归的。一个递归函数应该要被想成是过程，而不是机器。

9.  不是问题，因为程序员通过缩排来阅读与编写 Lisp 程序。

10. 的 I/O 函数是 read ，它包含了一个完整的 Lisp 语法分析器，以及 format ，它通过字符串模板来产生输出。

11. 以用 let 来创造新的局部变量，用 defparameter 来创造全局变量。

12. 操作符是 setf 。它的第一个实参可以是一个表达式。

13. 式编程代表避免产生副作用，也是 Lisp 的主导思维。

14. 的迭代操作符是 do 。

15. 是 Lisp 的对象。可以被当成实参传入，并且可以用 lambda 表达式来表示。

16. 在 Lisp 里，是数值才有类型，而不是变量。


### 习题 {#习题}

1.  描述下列表达式求值之后的结果：

    (a) (+ (- 5 1) (+ 3 7))
    ```lisp
    (princ (+ (- 5 1) (+ 3 7)))
    ```

    ```text
    14
    ```

    (b) (list 1 (+ 2 3))
    ```lisp
    (princ (list 1 (+ 2 3)))
    ```

    ```text
    (1 5)
    ```

    (c) (if (listp 1) (+ 1 2) (+ 3 4))
    ```lisp
    (princ (if (listp 1) (+ 1 2) (+ 3 4)))
    ```

    ```text
    7
    ```

    (d) (list (and (listp 3) t) (+ 1 2))
    ```lisp
    (princ (list (and (listp 3) t) (+ 1 2)))
    ```

    ```text
    (NIL 3)
    ```

2.  出 3 种不同表示 (a b c) 的 cons 表达式 。
    ```lisp
    (princ (list 'a 'b 'c))
    (print (cons 'a '(b c)))
    (print (cons 'a (cons 'b (cons 'c nil))))
    ```

    ```text
    (A B C)
    (A B C)
    (A B C)
    ```

3.  使用 car 与 cdr 来定义一个函数，返回一个列表的第四个元素。
    ```lisp
    (defun our-third (lst)
      (car (cdr (cdr (cdr lst)))))

    (princ (our-third '(1 2 3 4 5)))
    ```

    ```text
    4
    ```

4.  定义一个函数，接受两个实参，返回两者当中较大的那个。
    ```lisp
    (defun max-num (a b)
      (if (> a b)
        a
        b))

    (princ (max-num 3 5))
    ```

    ```text
    5
    ```

5.  这些函数做了什么？
    (a)
    ```lisp
    (defun enigma (x)
      (and (not (null x))
         (or (null (car x))
             (enigma (cdr x)))))
    ```

    该函数判断是否是列表，且含有空元素返回 true。
    ```lisp
    (princ (enigma '(1 2 3 4 nil 6)))
    (print (enigma nil))
    (print (enigma '(nil)))
    ```

    ```text
    T
    NIL
    T
    ```

    (b)
    ```lisp
    (defun mystery (x y)
      (if (null y)
        nil
        (if (eql (car y) x)
            0
            (let ((z (mystery x (cdr y))))
              (and z (+ z 1))))))
    ```

    ```lisp
    (princ (mystery 4 '(1 2 3 4 5 6)))
    (print (mystery 'a '(1 2 3 b d a)))
    ```

    ```text
    3
    5
    ```

    加入传入的参数是 4 和 '(1 2 3 4 5)，以下是整个递归过程，当参数是：4 '(4 5) 时返回0,之后依次加1

    4 '(1 2 3 4 5)

    4 '(2 3 4 5)

    4 '(3 4 5)

    4 '(4 5)

    作用：返回元素x 在列表 y 中的下标，下标从 0 开始。

6.  下列表达式， x 该是什么，才会得到相同的结果？

    (a) &gt; (car (x (cdr '(a (b c) d))))
       B
       答案：x 为 car
    ```lisp
    (princ (car (car (cdr '(a (b c) d)))))
    ```

    ```text
    B
    ```

    (b) &gt; (x 13 (/ 1 0))
        13

    答：x 为 and
    ```lisp
    (princ (or 13 (/ 1 0)))
    ```

    ```text
    13
    ```

    (c) &gt; (x #'list 1 nil)
        (1)

    答：x 为 cons
    ```lisp
    (princ (apply #'list 1 nil))
    ```

    ```text
    (1)
    ```

    > apply 可以接受任意数量的实参，只要最后一个实参是列表即可(包括空列表)
    > 最终传递给 list 函数的参数为：1
    >
    > funcall 做的是一样的事情，但不需要把实参包装成列表。最终传递给 list 函数的是：1 nil

<!--listend-->

1.  只使用本章所介绍的操作符，定义一个函数，它接受一个列表作为实参，如果有一个元素是列表时，就返回真。
    ```lisp
    (defun mylistp (lst)
      (if (null lst)
          nil
          (if (listp (car lst))
              t
              (mylistp (cdr lst)))))

    (princ (mylistp '(1 2 3)))
    (print (mylistp '(1 2 3 (1 2 3))))
    (print (mylistp '(1 2 3 nil)))
    ```

    ```text
    NIL
    T
    T
    ```

<!--listend-->

1.  给出函数的迭代与递归版本：
    -   接受一个正整数，并打印出数字数量的点。
        ```lisp
        (defun print-dot (n)
          (do ((i 1 (+ i 1)))
              ((> i n ) 'done)
            (format t "*")))

        (defun print-dot2 (n)
          (if (< n 1)
              nil
              (progn
                (format t "*")
                (print-dot2 (- n 1)))))

        (print-dot 5)
        (format t "~%")
        (print-dot2 8)
        ```

        ```text
        *****
        ********
        ```

<!--listend-->

-   接受一个列表，并返回 a 在列表里所出现的次数。
    ```lisp
    (defun char-count (a lst)
      (let ((i 0))
        (dolist (obj lst)
          (if (eql a obj)
              (setf i (+ i 1))))
        (format t "~A：~A出现了~A次~%" lst a i)))

    (let ((count 0))
      (defun count-tool (a lst)
        (if (null lst)
            nil
            (progn
              (if (eql a (car lst))
                  (incf count))
              (count-tool a (cdr lst)))))

      (defun get-count ()
        count))

    (defun char-count2 (a lst)
      (count-tool a lst)
      (format t "~A：~A出现了~A次~%" lst a (get-count)))

    (char-count 'a '(1 2 3 a a b 4 5 a))
    (format t "~%")
    (char-count2 'a '(a b c a a a a 1 2 3))
    ```

    ```text
    (1 2 3 A A B 4 5 A)：A出现了3次

    (A B C A A A A 1 2 3)：A出现了5次
    ```

<!--listend-->

1.  一位朋友想写一个函数，返回列表里所有非 nil 元素的和。他写了此函数的两个版本，但两个都不能工作。请解释每一个的错误在哪里，并给出正确的版本。
    (a)
    ```lisp
    (defun summit (lst)
      (remove nil lst)
      (apply #'+ lst))
    ```

    问题：remove 不会修改 lst 的值，只会返回一个新列表
    ```lisp
    (defun summit (lst)
      (setf lst (remove nil lst) )
      (apply #'+ lst))

    (princ (summit '(1 2 nil nil 2)))
    ```

    ```text
    5
    ```

    (b)
    ```lisp
    (defun summit (lst)
      (let ((x (car lst)))
        (if (null x)
            (summit (cdr lst))
            (+ x (summit (cdr lst))))))
    ```

    ```lisp
    (let ((i 0))
      (defun summit (lst)
        (let ((tem (car lst)))
          (if (numberp tem)
              (setf i (+ i tem)))
          (if (null (cdr lst))
              (return-from summit i)
              (summit (cdr lst)))))

      (defun get-i ()
        i)

      (defun set-i ()
        (setf i 0)))

    (set-i)
    (princ (summit '(1 2 3 nil nil 4 nil)))

    (set-i)
    (print (summit '(1 2 3)))

    (set-i)
    (print (summit '(nil)))

    (set-i)
    (print (summit '(nil 2 3 4 nil 1 nil)))



    ;; (defun make-summit ()
    ;;   (let ((i 0))
    ;;     (lambda (lst)
    ;;       (let ((tem (car lst)))
    ;; 	(if (numberp tem)
    ;; 	    (setf i (+ i tem)))
    ;; 	(if (null (cdr lst))
    ;; 	    (return-from summit i)
    ;; 	    (summit (cdr lst)))))))

    ;; (let ((my-summit (make-summit)))
    ;;   (format t "~A" (funcall my-summit '(1 2 3 4 1))))



    ```

    ```text
    10
    6
    0
    10
    ```


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

所以实际上你可以这么表示列表 (a b) ，

```latex
(a . (b . nil))
(a . (b))
(a b . nil)
(a b)
```


### 3.14 关联列表 (Assoc-lists) {#3-dot-14-关联列表--assoc-lists}

用 Cons 对象来表示映射 (mapping)也是很自然的。一个由 Cons 对象组成的列表称之为关联列表(assoc-listor alist)。这样的列表可以表示一个翻译的集合，举例来说：

```lisp
(setf trans '((+ . "add") (- . "subtract")))
(princ trans)
```

```text
((+ . add) (- . subtract))
```

关联列表很慢，但是在初期的程序中很方便。 Common Lisp 有一个内置的函数 assoc ，用来取出在关联列表中，与给定的键值有关联的 Cons 对：

```lisp
(princ (assoc '+ trans))
(print (assoc '* trans))          ; 如果 assoc 没有找到要找的东西时，返回 nil 。
```

```text
(+ . add)
NIL
```

以下是简化的 assoc 实现：

```lisp
(defun our-assoc (key alist)
  (and (consp alist)
       (let ((pair (car alist)))
         (if (eql key (car pair))
             pair
             (our-assoc key (cdr alist))))))

(princ (our-assoc '+ trans))
(print (our-assoc '* trans))
(print (our-assoc '- trans))
```

```text
(+ . add)
NIL
(- . "subtract")
```

和 member 一样，实际上的 assoc 接受关键字参数，包括 :test 和 :key 。 Common Lisp 也定义了一个 assoc-if 之于 assoc ，如同 member-if 之于 member 一样。


### 3.15 示例：最短路径 (Example: Shortest Path) {#3-dot-15-示例-最短路径--example-shortest-path}

以下是包含一个搜索网络中最短路径的程序。函数 shortest-path 接受一个起始节点，目的节点以及一个网络，并返回最短路径，如果有的话。

```lisp
;; 广度优先搜索
(defun shortest-path (start end net)
  (bfs end (list (list start)) net))

(defun bfs (end queue net)
  (format t  "~%log-bfs:[~A | ~A]" end queue)
  (if (null queue)
      nil
      (let ((path (car queue)))
        (let ((node (car path)))
          (if (eql node end)
              (reverse path)
              (bfs end
                   (append (cdr queue)
                           (new-paths path node net))
                   net))))))

(defun new-paths (path node net)
 (format t "~%log-new-paths:[~A | ~A]~%" path node)
  (mapcar #'(lambda (n)
              (cons n path))
          (cdr (assoc node net))))

(setf myMap '((a b c) (b c) (c d)))

(princ (shortest-path 'a 'd myMap))
```

```text

log-bfs:[D | ((A))]
log-new-paths:[(A) | A]

log-bfs:[D | ((B A) (C A))]
log-new-paths:[(B A) | B]

log-bfs:[D | ((C A) (C B A))]
log-new-paths:[(C A) | C]

log-bfs:[D | ((C B A) (D C A))]
log-new-paths:[(C B A) | C]

log-bfs:[D | ((D C A) (D C B A))](A C D)
```

在这个范例中，节点用符号表示，而网络用含以下元素形式的关联列表来表示：(node . neighbors)

所以由下图展示的最小网络 (minimal network)可以这样来表示：

{{< figure src="/images/最小网络.png" >}}

上图包含一个图，我们需要找到两点之间的最短路径。


### 3.16 垃圾 (Garbages) {#3-dot-16-垃圾--garbages}

自动内存管理(Automatic memory management)是 Lisp 最有价值的特色之一。 Lisp 系统维护着一段內存称之为堆(Heap)。系统持续追踪堆当中没有使用的内存，把这些内存发放给新产生的对象。举例来说，函数 cons ，返回一个新配置的 Cons 对象。从堆中配置内存有时候通称为 consing 。

如果内存永远没有释放， Lisp 会因为创建新对象把内存用完，而必须要关闭。所以系统必须周期性地通过搜索堆 (heap)，寻找不需要再使用的内存。不需要再使用的内存称之为垃圾 (garbage)，而清除垃圾的动作称为垃圾回收 (garbage collection或 GC)。

垃圾是从哪来的？让我们来创造一些垃圾：

```lisp
(setf lst (list 'a 'b 'c))
(setf lst nil)
```

因为我们没有任何方法再存取列表，它也有可能是不存在的。我们不再有任何方式可以存取的对象叫做垃圾。系统可以安全地重新使用这三个 Cons 核。

这种管理內存的方法，给程序员带来极大的便利性。你不用显式地配置 (allocate)或释放 (dellocate)內存。这也表示了你不需要处理因为这么做而可能产生的臭虫。內存泄漏 (Memory leaks)以及迷途指针 (dangling pointer)在 Lisp 中根本不可能发生。


### 总结 (Summary) {#总结--summary}

-   一个 Cons 是一个含两部分的数据结构。列表用链结在一起的 Cons 组成。

-   判断式 equal 比 eql 来得不严谨。基本上，如果传入参数印出来的值一样时，返回真。

-   所有 Lisp 对象表现得像指针。你永远不需要显式操作指针。

-   你可以使用 copy-list 复制列表，并使用 append 来连接它们的元素。

-   游程编码是一个餐厅中使用的简单压缩演算法。

-   Common Lisp 有由 car 与 cdr 定义的多种存取函数。

-   映射函数将函数应用至逐项的元素，或逐项的列表尾端。

-   嵌套列表的操作有时被考虑为树的操作。

-   要判断一个递归函数是否正确，你只需要考虑是否包含了所有情况。

-   列表可以用来表示集合。数个内置函数把列表当作集合。

-   关键字参数是选择性的，并不是由位置所识别，是用符号前面的特殊标签来识别。

-   列表是序列的子类型。 Common Lisp 有大量的序列函数。

-   一个不是正规列表的 Cons 称之为点状列表。

-   用 cons 对象作为元素的列表，可以拿来表示对应关系。这样的列表称为关联列表(assoc-lists)。

-   自动内存管理拯救你处理内存配置的烦恼，但制造过多的垃圾会使程序变慢。


## 第四章：特殊数据结构 {#第四章-特殊数据结构}


### 4.1 数组（Array） {#4-dot-1-数组-array}

调用 make-array 来构造一个数组，第一个实参为一个指定数组维度的列表。要构造一个 2 x 3 的数组，我们可以：

```lisp
(setf arr (make-array '(2 3) :initial-element nil))
(princ arr)
```

```text
#2A((NIL NIL NIL) (NIL NIL NIL))
```

Common Lisp 的数组至少可以达到七个维度，每个维度至少可以容纳 1023 个元素。

:initial-element 实参是选择性的。如果有提供这个实参，整个数组会用这个值作为初始值。若试着取出未初始化的数组内的元素，其结果为未定义（undefined）。

用 aref 取出数组内的元素。与 Common Lisp 的存取函数一样， aref 是零索引的（zero-indexed）：

```lisp
(princ (aref arr 0 0))
```

```text
NIL
```

要替换数组的某个元素，我们使用 setf 与 aref ：

```lisp
(setf (aref arr 0 0) 'b)
(princ (aref arr 0 0))
```

```text
B
```

要表示字面常量的数组（literal array），使用 #na 语法，其中 n 是数组的维度。举例来说，我们可以这样表示 arr 这个数组：

```lisp
(setf num #2a((b nil nil) (nil nil nil)))
(princ num)
```

```text
#2A((B NIL NIL) (NIL NIL NIL))
```

如果全局变量 **print-array** 为真，则数组会用以下形式来显示：

```lisp
(setf *print-array* nil)
(princ arr)
(setf *print-array* t)
(print arr)
```

```text
#<(SIMPLE-ARRAY T (2 3)) {1002FD499F}>
#2A((B NIL NIL) (NIL NIL NIL))
```

```lisp
(setf vec (make-array 4 :initial-element nil))
(princ vec)
```

```text
#(NIL NIL NIL NIL)
```

一维数组又称为向量（vector）。你可以通过调用 vector 来一步骤构造及填满向量，向量的元素可以是任何类型：

```lisp
(princ (vector "a" 'b 3))
```

```text
#(a B 3)
```

字面常量的数组可以表示成 #na ，字面常量的向量也可以用这种语法表达。

可以用 aref 来存取向量，但有一个更快的函数叫做 svref ，专门用来存取向量。

```lisp
;; 这些都是简单向量
(vector 1 2 3 4)
(make-array 5)
(make-array 5 :initial-element 0)

;; 这些不是简单向量（有特殊属性）
(make-array 5 :adjustable t)        ; 可调整大小
(make-array 5 :fill-pointer 0)      ; 有填充指针

;;(svref vec 0)
;; 创建和访问向量
;; 创建简单向量
(defparameter *my-vec* (vector 10 20 30 40 50))

;; 使用 svref 访问元素
(svref *my-vec* 0)    ; => 10
(svref *my-vec* 2)    ; => 30
(svref *my-vec* 4)    ; => 50

;; 遍历简单向量
(dotimes (i (length *my-vec*))
  (format t "~A " (svref *my-vec* i)))

```

```text
10 20 30 40 50
```

在 svref 内的 “sv” 代表“简单向量”（“simple vector”），所有的向量缺省是简单向量。


### 4.2 示例：二叉搜索 (Example: Binary Search) {#4-dot-2-示例-二叉搜索--example-binary-search}

也叫二分查找，需要搜索的序列有序。

```lisp
(defun bin-search (obj vec)
  (let ((len (length vec)))
    (and (not (zerop len))
         (finder obj vec 0 (- len 1)))))

(defun finder (obj vec start end)
  (format t "~A~%" (subseq vec start (+ end 1)))
  (let ((range (- end start)))
    (if (zerop range)
        (if (eql obj (aref vec start))
            obj
            nil)
        (let ((mid (+ start (round (/ range 2)))))
          (let ((obj2 (aref vec mid)))
            (if (< obj obj2)
                (finder obj vec start (- mid 1))
                (if (> obj obj2)
                    (finder obj vec (+ mid 1) end)
                    obj)))))))
(bin-search 3 #(0 1 2 3 4 5 6 7 8 9))
```

```text
#(0 1 2 3 4 5 6 7 8 9)
#(0 1 2 3)
#(3)
```

果要找的 range 缩小至一个元素，而如果这个元素是 obj 的话，则 finder 直接返回这个元素，反之返回 nil 。如果 range 大于 1 ，我们設置 middle ( round 返回离实参最近的整数) 為 obj2 。如果 obj 小于 obj2 ，则递归地往向量的左半部寻找。如果 obj 大于 obj2 ，则递归地往向量的右半部寻找。剩下的一个选择是 obj=obj2 ，在这个情况我们找到要找的元素，直接返回这个元素。


### 4.3 字符与字符串(Strings and Characters) {#4-dot-3-字符与字符串--strings-and-characters}

字符串是字符组成的向量。我们用一系列由双引号包住的字符，来表示一个字符串常量，而字符 c 用 #\c 表示。

每个字符都有一个相关的整数 ── 通常是 ASCII 码，但不一定是。在多数的 Lisp 实现里，函数 char-code 返回与字符相关的数字，而 code-char 返回与数字相关的字符。

字符比较函数 char&lt; （小于）， char&lt;= （小于等于)， char= （等于)， char&gt;= （大于等于) ， char&gt; （大于)，以及 char/= （不同)。

```lisp
(princ (sort "elbow" #'char<))
(print (sort "elbow" #'char>))
```

```text
below
"woleb"
```

由于字符串是字符向量，序列与数组的函数都可以用在字符串。你可以用 aref 来取出元素，举例来说，

```lisp
(princ (aref "abc" 1))
```

```text
b
```

但针对字符串可以使用更快的 char 函数：

```lisp
(princ (char "abc" 1))
```

```text
b
```

可以使用 setf 搭配 char （或 aref ）来替换字符串的元素：

```lisp
(let ((str (copy-seq "Merlin")))
 (setf (char str 3) #\k)
 (princ str))
```

```text
Merkin
```

如果你想要比较两个字符串，你可以使用通用的 equal 函数，但还有一个比较函数，是忽略字母大小写的 string-equal ：

```lisp
(princ (equal "fred" "fred"))
(print (equal "fred" "Fred"))
(print (string-equal "fred" "Fred"))
```

```text
T
NIL
T
```

Common Lisp 提供大量的操控、比较字符串的函数。

有许多方式可以创建字符串。最普遍的方式是使用 `format` 。将第一个参数设为 `nil` 来调用 format ，使它返回一个原本会印出来的字符串：

```lisp
(format t "~A or ~A" "truth" "dare")
```

```text
truth or dare
```

但若你只想把数个字符串连结起来，你可以使用 `concatenate` ，它接受一个特定类型的符号，加上一个或多个序列：

```lisp
(princ (concatenate 'string "not " "to worry"))
```

```text
not to worry
```


### 4.4 列序(Sequences) {#4-dot-4-列序--sequences}

在 Common Lisp 里，序列类型包含了列表与向量（因此也包含了字符串）。有些用在列表的函数，实际上是序列函数，包括 remove 、 length 、 subseq 、 reverse 、 sort 、 every 以及 some 。

```lisp
(princ (mirror? "abba"))
```

```text
T
```

取出序列元素的函数：给列表使用的 nth ， 给向量使用的 aref 及 svref

Common Lisp 也提供了通用的 elt ，对任何种类的序列都有效：

```lisp
(princ (elt '(a b c) 1))
```

```text
B
```

使用 elt ，我们可以写一个针对向量来说更有效率的 mirror? 版本：

```lisp
(defun mirror? (s)
  (let ((len (length s)))
    (and (evenp len)
         (do ((forward 0 (+ forward 1))
              (back (- len 1) (- back 1)))
             ((or (> forward back)
                  (not (eql (elt s forward)
                            (elt s back))))
              (> forward back))))))
(princ (mirror? "abc"))
(print (mirror? "abba"))
```

```text
NIL
T
```

这个版本也可用在列表，但这个实现更适合给向量使用。频繁的对列表调用 `elt` 的代价是昂贵的，因为列表仅允许顺序存取，而向量允许随机存取，从任何元素来存取每一个元素都是廉价的。

许多序列函数接受一个或多个由下表所列的标准关键字参数：

| 参数      | 用途       | 缺省值   |
|---------|----------|-------|
| :key      | 应用至每个元素的函数 | identity |
| :test     | 用来比较的函数 | eql      |
| :from-end | 若为真，反向工作 | nil      |
| :start    | 起始位置   | 0        |
| :end      | 若有给定，结束位置 | nil      |

接受所有关键字参数的函数之一是 position ，返回序列中一个元素的位置，未找到时返回 nil 。

```lisp
(princ (position #\a "fantasia"))
(print (position #\a "fantasia" :start 3 :end 5))
```

```text
1
4
```

使用 :from-end 关键字参数：

```lisp
(princ (position #\a "fantasia" :from-end t))
```

```text
7
```

:key 关键字参数是序列中每个元素在被考虑之前要应用的函数：

```lisp
(princ (position 'a '((c d) (a b)) :key #'car))
```

```text
1
```

:test 关键字参数接受需要两个实参的函数，并决定何谓成功匹配（默认 eql）。例如匹配列表时通常需要 equal ：

```lisp
(op
 (position '(a b) '((a b) (c d)))
 (position '(a b) '((a b) (c d)) :test #'equal))
```

```text
NIL
0
```

:test 可以是任何接受两个参数的函数。例如给定 &lt; ，我们可以询问第一个使第一个参数比它小的元素位置：

```lisp
(op
 (position 3 '(1 0 7 5) :test #'<))
```

```text
2
```

使用 subseq 与 position ，我们可以写出分开序列的函数。举例来说，这个函数

```lisp
(defun second-word (str)
  (let ((p1 (+ (position #\  str) 1)))
    (subseq str p1 (position #\  str :start p1))))

(op
 (second-word "Form follows function"))
```

```text
follows
```

要找到满足谓词的元素，其中谓词接受一个实参，我们使用 position-if 。它接受一个函数与序列，并返回第一个满足此函数的元素：

```lisp
(op
 (position-if #'oddp '(2 3 4 5)))
```

```text
1
```

position-if 接受除了 :test 之外的所有关键字参数。

有许多相似的函数，如给序列使用的 member 与 member-if 。分别是， find （接受全部关键字参数）与 find-if （接受除了 :test 之外的所有关键字参数）：

```lisp
(op (find #\a "cat")
    (find-if #'characterp "ham"))
```

```text
a
h
```

不同于 member 与 member-if ，它们仅返回要寻找的对象。

通常一个 find-if 的调用，如果解读为 find 搭配一个 :key 关键字参数的话，会显得更清楚。举例来说，表达式

```lisp
(setf lst '((complete success "Operation finished")
            (error failure "File not found")
            (complete success "Data saved")))

(op (find-if #'(lambda (x)
                 (eql (car x) 'complete))
             lst))

```

```text
(COMPLETE SUCCESS Operation finished)
```

上述代码等同于：

```lisp
(op (find 'complete lst :key #'car))
```

```text
(COMPLETE SUCCESS Operation finished)
```

函数 remove （22 页）以及 remove-if 通常都可以用在序列。它们跟 find 与 find-if 是一样的关系。另一个相关的函数是 remove-duplicates ，仅保留序列中每个元素的最后一次出现。

```lisp
(op (remove-duplicates "abracadabra"))
```

```text
cdbra
```

这个函数接受前表所列的所有关键字参数。

函数 reduce 用来把序列压缩成一个值。它至少接受两个参数，一个函数与序列。函数必须是接受两个实参的函数。在最简单的情况下，一开始函数用序列前两个元素作为实参来调用，之后接续的元素作为下次调用的第二个实参，而上次返回的值作为下次调用的第一个实参。最后调用最终返回的值作为 reduce 整个函数的返回值。也就是说像是这样的表达式：

(reduce #'fn '(a b c d))
等同于 (fn (fn (fn 'a 'b) 'c) 'd)

我们可以使用 reduce 来扩充只接受两个参数的函数。举例来说，要得到三个或多个列表的交集(intersection)，我们可以：

```lisp
(op (reduce #'intersection '((b r a d 's) (b a d) (c a t))))
```

```text
(A)
```


### 4.5 示例：解析日期 (Example: Parsing Dates) {#4-dot-5-示例-解析日期--example-parsing-dates}

作为序列操作的示例，本节演示了如何写程序来解析日期。我们将编写一个程序，可以接受像是 “16 Aug 1980” 的字符串，然后返回一个表示日、月、年的整数列表。

```lisp
(defun tokens (str test start)
  (let ((p1 (position-if test str :start start)))
    (if p1
        (let ((p2 (position-if #'(lambda (c)
                                   (not (funcall test c)))
                               str :start p1)))
          (cons (subseq str p1 p2)
                (if p2
                    (tokens str test p2)
                    nil)))
        nil)))

(defun constituent (c)
  (and (graphic-char-p c)
       (not (char= c #\ ))))
```

上述代码包含了某些在这个应用里所需的通用解析函数。第一个函数 tokens ，用来从字符串中取出语元 （token）。给定一个字符串及测试函数，满足测试函数的字符组成子字符串，子字符串再组成列表返回。举例来说，如果测试函数是对字母返回真的 alpha-char-p 函数，我们得到：

```lisp
(op (tokens "ab12 3cde.f" #'alpha-char-p 0))
```

```text
(ab cde f)
```

所有不满足此函数的字符被视为空白 ── 他们是语元的分隔符，但永远不是语元的一部分。

函数 constituent 被定义成用来作为 tokens 的实参。

在 Common Lisp 里，图形字符是我们可见的字符，加上空白字符。所以如果我们用 constituent 作为测试函数时，

```lisp
(op (tokens "ab12 3cde.f gh" #'constituent 0))
```

```text
(ab12 3cde.f gh)
```

则语元将会由空白区分出来。

```lisp
(defun parse-date (str)
  (let ((toks (tokens str #'constituent 0)))
    (list (parse-integer (first toks))
          (parse-month (second toks))
          (parse-integer (third toks)))))

(defparameter *month-names*
  #("jan" "feb" "mar" "apr" "may" "jun"
    "jul" "aug" "sep" "oct" "nov" "dec"))

(defun parse-month (str)
  (let ((p (position str *month-names*
                         :test #'string-equal)))
    (if p
        (+ p 1)
        nil)))
```

上述代码包含了特别为解析日期打造的函数。函数 parse-date 接受一个特别形式组成的日期，并返回代表这个日期的整数列表：

```lisp
(op (parse-date "16 Aug 1980"))
```

```text
(16 8 1980)
```

parse-date 使用 tokens 来解析日期字符串，接着调用 parse-month 及 parse-integer 来转译年、月、日。要找到月份，调用 parse-month ，由于使用的是 string-equal 来匹配月份的名字，所以输入可以不分大小写。要找到年和日，调用内置的 parse-integer ， parse-integer 接受一个字符串并返回对应的整数。

如果需要自己写程序来解析整数，也许可以这么写：

```lisp
(defun read-integer (str)
  (if (every #'digit-char-p str)
      (let ((accum 0))
        (dotimes (pos (length str))
          (setf accum (+ (* accum 10)
                         (digit-char-p (char str pos)))))
        accum)
    nil))
```

这个定义演示了在 Common Lisp 中，字符是如何转成数字的 ── 函数 digit-char-p 不仅测试字符是否为数字，同时返回了对应的整数。


### 4.6 结构 (Structures) {#4-dot-6-结构--structures}

结构可以想成是豪华版的向量。假设你要写一个程序来追踪长方体。你可能会想用三个向量元素来表示长方体：高度、宽度及深度。与其使用原本的 svref ，不如定义像是下面这样的抽象，程序会变得更容易阅读，

```lisp
(defun block-height (b) (svref b 0))
```

而结构可以想成是，这些函数通通都替你定义好了的向量。

要想定义结构，使用 defstruct 。在最简单的情况下，只要给出结构及字段的名字便可以了：

```lisp
(defstruct point
  x
  y)
```

这里定义了一个 point 结构，具有两个字段 x 与 y 。同时隐式地定义了 make-point 、 point-p 、 copy-point 、 point-x 及 point-y 函数。

通过使用宏，Lisp 程序可以写出 Lisp 程序。这是目前所见的明显例子之一。当你调用 defstruct 时，它自动生成了其它几个函数的定义。有了宏以后，你将可以自己来办到同样的事情（如果需要的话，你甚至可以自己写出 defstruct ）。

每一个 make-point 的调用，会返回一个新的 point 。可以通过给予对应的关键字参数，来指定单一字段的值：

```lisp
(setf p (make-point :x 0 :y 0))
```

存取 point 字段的函数不仅被定义成可取出数值，也可以搭配 setf 一起使用。

```lisp
(op (point-x p)
    (setf (point-y p) 2)
    p)
```

```text
0
2
#<0, 2>
```

定义结构也定义了以结构为名的类型。每个点（point）的类型层级会是，类型 point ，接着是类型 structure ，再来是类型 atom ，最后是 t 类型。所以使用 point-p 来测试某个东西是不是一个点时，也可以使用通用性的函数，像是 typep 来测试。

```lisp
(op (point-p p)
    (typep p 'point))
```

```text
T
T
```

我们可以在本来的定义中，附上一个列表，含有字段名及缺省表达式，来指定结构字段的缺省值。

```lisp
(defstruct polemic
  (type (progn
          (format t "What kind of polemic was it? ")
          (read)))
  (effect nil))
```

```latex
> (make-polemic)
What kind of polemic was it? scathing
#S(POLEMIC :TYPE SCATHING :EFFECT NIL)
```

如果 make-polemic 调用没有给字段指定初始值，则字段会被设成缺省表达式的值：

结构显示的方式也可以控制，以及结构自动产生的存取函数的字首。以下是做了前述两件事的 point 定义：

```lisp
(defstruct (point (:conc-name p)     ; 字段访问函数前缀为 p
                  (:print-function print-point))  ; 自定义打印函数
  (x 0)  ; x 坐标，默认值 0
  (y 0)) ; y 坐标，默认值 0

(defun print-point (p stream depth)
  (format stream "#<~A, ~A>" (px p) (py p)))
```

:conc-name 关键字参数指定了要放在字段前面的名字，并用这个名字来生成存取函数。预设是 point- ；现在变成只有 p 。不使用缺省的方式使代码的可读性些微降低了，只有在需要常常用到这些存取函数时，你才会想取个短点的名字。

:print-function 是在需要显示结构出来看时，指定用来打印结构的函数 ── 需要显示的情况比如，要在顶层显示时。这个函数需要接受三个实参：要被印出的结构，在哪里被印出，第三个参数通常可以忽略。 [2] 我们会在 7.1 节讨论流（stream）。现在来说，只要知道流可以作为参数传给 format 就好了。

函数 print-point 会用缩写的形式来显示点：

```lisp
;; 使用默认值创建点
(setq p1 (make-point))
;; 打印结果: #<0, 0>

;; 创建指定坐标的点
(setq p2 (make-point :x 10 :y 20))
;; 打印结果: #<10, 20>

;; 创建部分指定值的点
(setq p3 (make-point :x 5))
;; 打印结果: #<5, 0>

;; 访问字段（由于 :conc-name p，使用 px/py 而不是 point-x/point-y）
(op (px p2)
    (py p2))

;; 修改字段
(setf (px p2) 15)
(setf (py p2) 25)

(op p2)


```

```text
10
20
#<15, 25>
```


### 4.7 示例：二叉搜索树 (Example: Binary Search Tree) {#4-dot-7-示例-二叉搜索树--example-binary-search-tree}

由于 sort 本身系统就有了，极少需要在 Common Lisp 里编写排序程序。本节将演示如何解决一个与此相关的问题，这个问题尚未有现成的解决方案：维护一个已排序的对象集合。本节的代码会把对象存在二叉搜索树里（ binary search tree ）或称作 BST。当二叉搜索树平衡时，允许我们可以在与时间成 log n 比例的时间内，来寻找、添加或是删除元素，其中 n 是集合的大小。

{{< figure src="/images/二叉搜索树.png" >}}

二叉搜索树是一种二叉树，给定某个排序函数，比如 &lt; ，每个元素的左子树都 &lt; 该元素，而该元素 &lt; 其右子树。图 4.4 展示了根据 &lt; 排序的二叉树。

下列代码包含了二叉搜索树的插入与寻找的函数。基本的数据结构会是 node （节点），节点有三个部分：一个字段表示存在该节点的对象，以及各一个字段表示节点的左子树及右子树。可以把节点想成是有一个 car 和两个 cdr 的一个 cons 核（cons cell）。

```lisp
(defstruct (node (:print-function
                  (lambda (n s d)
                    (format s "#<~A>" (node-elt n)))))
  elt (l nil) (r nil))

(defun bst-insert (obj bst <)
  (if (null bst)
      (make-node :elt obj)  ; 空树时创建新节点
      (let ((elt (node-elt bst)))
        (if (equal obj elt)
            bst  ; 元素已存在，直接返回
            (if (funcall < obj elt)
                                        ; 插入左子树，重建路径上的所有节点
                (make-node :elt elt
                           :l (bst-insert obj (node-l bst) <)
                           :r (node-r bst))
                                        ; 插入右子树，重建路径上的所有节点
                (make-node :elt elt
                           :r (bst-insert obj (node-r bst) <)
                           :l (node-l bst)))))))


(defun bst-find (obj bst <)
  (if (null bst)
      nil
      (let ((elt (node-elt bst)))
        (if (equal obj elt)
            bst  ; 找到返回节点
            (if (funcall < obj elt)
                (bst-find obj (node-l bst) <)  ; 左子树查找
                (bst-find obj (node-r bst) <))))))  ; 右子树查找

(defun bst-min (bst)
  (and bst
       (or (bst-min (node-l bst)) bst)))  ; 最左节点

(defun bst-max (bst)
  (and bst
       (or (bst-max (node-r bst)) bst)))  ; 最右节点

```

基本操作：

```lisp
  ;; 创建空树
(setf tree nil)

;; 插入元素（需要比较函数）
(setf tree (bst-insert 5 tree #'<))
(setf tree (bst-insert 3 tree #'<))
(setf tree (bst-insert 7 tree #'<))
(setf tree (bst-insert 1 tree #'<))
(setf tree (bst-insert 9 tree #'<))

(op tree)
;; 树结构：
;;       5
;;      / \
;;     3   7
;;    /     \
;;   1       9
```

```text
#<5>
```

查找操作：

```lisp

(op ;; 查找元素
    (bst-find 3 tree #'<)  ; => #<3>  (找到节点)
    (bst-find 8 tree #'<)  ; => NIL   (未找到)

    ;; 查找极值
    (bst-min tree)  ; => #<1>  (最小值节点)
    (bst-max tree)  ; => #<9>  (最大值节点)
    )
```

```text
#<3>
NIL
#<1>
#<9>
```

字符串树示例：

```lisp
;; 使用字符串比较
(setf str-tree nil)
(setf str-tree (bst-insert "apple" str-tree #'string<))
(setf str-tree (bst-insert "banana" str-tree #'string<))
(setf str-tree (bst-insert "cherry" str-tree #'string<))

(op (bst-find "banana" str-tree #'string<)
    (bst-find "cherry" str-tree #'string<)
    (bst-min str-tree) )

```

```text
#<banana>
#<cherry>
#<apple>
```

删除操作：

```lisp
(defun bst-remove (obj bst <)
  (if (null bst)
      nil  ; 空树直接返回 nil
      (let ((elt (node-elt bst)))
        (if (eql obj elt)
            (percolate bst)  ; 找到要删除的节点
            (if (funcall < obj elt)
                                        ; 在左子树中删除，重建当前节点
                (make-node :elt elt
                           :l (bst-remove obj (node-l bst) <)
                           :r (node-r bst))
                                        ; 在右子树中删除，重建当前节点
                (make-node :elt elt
                           :r (bst-remove obj (node-r bst) <)
                           :l (node-l bst)))))))


(defun percolate (bst)
  (cond ((null (node-l bst))       ; 情况1：无左子树
         (if (null (node-r bst))
             nil                   ; 无子节点，直接删除
             (rperc bst)))         ; 只有右子树，用右子树替代
        ((null (node-r bst))        ; 情况2：无右子树
         (lperc bst))              ; 只有左子树，用左子树替代
        (t (if (zerop (random 2))  ; 情况3：有两个子树，随机选择
               (lperc bst)         ; 50%概率用左子树方式
               (rperc bst)))))     ; 50%概率用右子树方式

(defun rperc (bst)
  (make-node :elt (node-elt (node-r bst))  ; 用右子节点值替代当前节点
             :l (node-l bst)               ; 保持左子树不变
             :r (percolate (node-r bst)))) ; 递归处理右子树


(defun lperc (bst)
  (make-node :elt (node-elt (node-l bst))  ; 用左子节点值替代当前节点
             :l (percolate (node-l bst))   ; 递归处理左子树
             :r (node-r bst)))             ; 保持右子树不变

;; 更美观的版本，带树枝连接线
(defun print-tree-pretty (tree &optional (prefix "") (is-left t))
  "美观地打印二叉搜索树结构"
  (when tree
    ;; 打印右子树
    (print-tree-pretty (node-r tree)
                       (concatenate 'string prefix (if is-left "│   " "    "))
                       nil)

    ;; 打印当前节点
    (format t "~A" prefix)
    (format t (if is-left "└── " "┌── "))
    (format t "~A~%" (node-elt tree))

    ;; 打印左子树
    (print-tree-pretty (node-l tree)
                       (concatenate 'string prefix (if is-left "    " "│   "))
                       t)))

;; 增强版本：带树枝连接
(defun print-tree-with-branches (tree)
  "打印带树枝的树形结构"
  (labels ((get-height (node)
             (if (null node)
                 0
                 (1+ (max (get-height (node-l node))
                          (get-height (node-r node))))))

           (print-level (nodes level max-level)
             (when (and nodes (> max-level level))
               (let ((floor (- max-level level))
                     (first (zerop level))
                     (elements (length nodes)))

                 ;; 打印前导空格
                 (dotimes (i (expt 2 floor))
                   (format t "  "))

                 ;; 打印节点和连接线
                 (loop for node in nodes
                       for i from 0 do
                         (if node
                             (format t "~2D" (node-elt node))
                             (format t "  "))

                         (dotimes (i (- (expt 2 (+ floor 1)) 1))
                           (format t "  ")))

                 (format t "~%")

                 ;; 如果不是最后一级，打印连接线
                 (when (not first)
                   (dotimes (i (expt 2 floor))
                     (format t "  "))

                   (loop for node in nodes do
                     (if node
                         (format t "/\\  ")
                         (format t "    "))

                     (dotimes (i (- (expt 2 (+ floor 1)) 1))
                       (format t "  ")))

                   (format t "~%")))

               ;; 递归打印下一级
               (when (< level max-level)
                 (print-level (loop for node in nodes
                                    append (if node
                                               (list (node-l node) (node-r node))
                                               (list nil nil)))
                              (+ level 1)
                              max-level)))))

    (let ((height (get-height tree)))
      (print-level (list tree) 0 height))))
```

打印树形结构：

```lisp
(defun print-tree (tree &optional (level 0) (position :root) (parent-right nil))
  "打印二叉搜索树的树形结构"
  (when tree
    ;; 先递归打印右子树
    (print-tree (node-r tree) (+ level 1) :right (eq position :right))

    ;; 打印当前节点
    (let ((indent (* level 4)))
      ;; 打印连接线
      (when (> level 0)
        (dotimes (i (- indent 2))
          (format t " "))
        (case position
          (:left (format t "└── "))
          (:right (format t "┌── "))
          (t (format t ""))))

      ;; 打印节点值
      (if (> level 0)
          (format t "~A" (node-elt tree))
          (format t "~A" (node-elt tree)))

      (format t "~%"))

    ;; 递归打印左子树
    (print-tree (node-l tree) (+ level 1) :left (eq position :left))))
```

删除节点示例：

```lisp

;; 创建二叉搜索树
(setf tree nil)
(setf tree (bst-insert 5 tree #'<))
(setf tree (bst-insert 3 tree #'<))
(setf tree (bst-insert 7 tree #'<))
(setf tree (bst-insert 1 tree #'<))
(setf tree (bst-insert 4 tree #'<))
(setf tree (bst-insert 6 tree #'<))
(setf tree (bst-insert 9 tree #'<))

;; 树结构：
;;       5
;;      / \
;;     3   7
;;    / \  / \
;;   1  4 6   9
(print-tree tree)
```

```text
      ┌── 9
  ┌── 7
      └── 6
5
      ┌── 4
  └── 3
      └── 1
```

```lisp
;; 删除叶子节点 (1)
(setf new-tree (bst-remove 1 tree #'<))
(print-tree new-tree)

;; 新树结构：
;;       5
;;      / \
;;     3   7
;;      \  / \
;;      4 6   9
```

```text
      ┌── 9
  ┌── 7
      └── 6
5
      ┌── 4
  └── 3
```

```lisp
;; 删除有一个子节点的节点 (删除3后，4成为2的子节点)
(setf new-tree2 (bst-remove 3 new-tree #'<))
(print-tree new-tree2)
;; 新树结构：
;;       5
;;      / \
;;     4   7
;;         / \
;;        6   9

```

```text
      ┌── 9
  ┌── 7
      └── 6
5
  └── 4
```

```lisp
;; 删除有两个子节点的根节点 (5)
(setf new-tree3 (bst-remove 5 new-tree2 #'<))
;; 结果可能有两种（随机）：
;; 情况1（选择左子树方式）：       情况2（选择右子树方式）：
;;       4                         6
;;      / \                       / \
;;     NIL 7                     4   7
;;        / \                       / \
;;       6   9                     NIL 9

(print-tree new-tree3)
```

```text
      ┌── 9
  ┌── 7
      └── 6
4
```

算法特点

-   不可变数据结构：所有操作返回新树，原树不变

-   随机平衡：删除有两个子节点的节点时随机选择替代策略，有助于避免树的不平衡

-   路径复制：只复制受影响的路径，其他部分共享

-   递归渗透：通过 percolate函数递归处理子树提升


### 4.8 哈希表 (Hash Table) {#4-dot-8-哈希表--hash-table}

第三章演示过列表可以用来表示集合（sets）与映射（mappings）。但当列表的长度大幅上升时（或是 10 个元素），使用哈希表的速度比较快。你通过调用 make-hash-table 来构造一个哈希表，它不需要传入参数：

```lisp
(op (setf ht (make-hash-table)))
```

```text
#<HASH-TABLE :TEST EQL :COUNT 0 {1004DCCDC3}>
```

和函数一样，哈希表总是用 #&lt;...&gt; 的形式来显示。

一个哈希表，与一个关联列表类似，是一种表达对应关系的方式。要取出与给定键值有关的数值，我们调用 gethash 并传入一个键值与哈希表。预设情况下，如果没有与这个键值相关的数值， gethash 会返回 nil 。

```lisp
(op (gethash 'color ht))
(setf (gethash 'color ht) 'red)
(op (multiple-value-list (gethash 'color ht)))
```

```text
NIL
(RED T)
```

gethash 函数返回两个值，使用 multiple-value-list 将两个返回值转换为列表

[Common Lisp 返回多个值](https://yb.tencent.com/s/1D5CYfdGAoJk)

在这里我们首次看到 Common Lisp 最突出的特色之一：一个表达式竟然可以返回多个数值。函数 gethash 返回两个数值。第一个值是与键值有关的数值，第二个值说明了哈希表是否含有任何用此键值来储存的数值。由于第二个值是 nil ，我们知道第一个 nil 是缺省的返回值，而不是因为 nil 是与 color 有关的数值。

大部分的实现会在顶层显示一个函数调用的所有返回值，但仅期待一个返回值的代码，只会收到第一个返回值。 5.5 节会说明，代码如何接收多个返回值。

要把数值与键值作关联，使用 gethash 搭配 setf ：

```lisp
(setf (gethash 'color ht) 'red)
```

存在哈希表的对象或键值可以是任何类型。举例来说，如果我们要保留函数的某种讯息，我们可以使用哈希表，用函数作为键值，字符串作为词条（entry）：

```lisp
(setf bugs (make-hash-table))
(push "Doesn't take keyword arguments."
      (gethash #'our-member bugs))
(maphash #'(lambda (k v)
             (format t "~A = ~A~%" k v))
         bugs)
```

```text
#<FUNCTION OUR-MEMBER> = (Doesn't take keyword arguments.)
```

可以用哈希表来取代用列表表示集合。当集合变大时，哈希表的查询与移除会来得比较快。要新增一个成员到用哈希表所表示的集合，把 gethash 用 setf 设成 t ：

```lisp
(setf fruit (make-hash-table))
(setf (gethash 'apricot fruit) t)
(op (multiple-value-list (gethash 'apricot fruit)))
```

```text
(T T)
```

要从集合中移除一个对象，你可以调用 remhash ，它从一个哈希表中移除一个词条：

```lisp
(op (remhash 'apricot fruit))
```

```text
T
```

哈希表有一个迭代函数： maphash ，它接受两个实参，接受两个参数的函数以及哈希表。该函数会被每个键值对调用，没有特定的顺序：

```lisp
(setf (gethash 'shape ht) 'spherical
      (gethash 'size ht) 'giant)
(maphash #'(lambda (k v)
             (format t "~A = ~A~%" k v))
         ht)

```

```text
COLOR = RED
SHAPE = SPHERICAL
SIZE = GIANT
```

maphash 总是返回 nil ，但你可以通过传入一个会累积数值的函数，把哈希表的词条存在列表里。

哈希表可以容纳任何数量的元素，但当哈希表空间用完时，它们会被扩张。如果你想要确保一个哈希表，从特定数量的元素空间大小开始时，可以给 make-hash-table 一个选择性的 :size 关键字参数。做这件事情有两个理由：因为你知道哈希表会变得很大，你想要避免扩张它；或是因为你知道哈希表会是很小，你不想要浪费内存。 :size 参数不仅指定了哈希表的空间，也指定了元素的数量。平均来说，在被扩张前所能够容纳的数量。所以

```lisp
(make-hash-table :size 5)
```

会返回一个预期存放五个元素的哈希表。

和任何牵涉到查询的结构一样，哈希表一定有某种比较键值的概念。预设是使用 eql ，但你可以提供一个额外的关键字参数 :test 来告诉哈希表要使用 eq ， equal ，还是 equalp ：

```lisp
(setf writers (make-hash-table :test #'equal))
(setf (gethash '(ralph waldo emerson) writers) t)
(maphash #'(lambda (k v)
             (format t "~A = ~A~%" k v))
         writers)
```

```text
(RALPH WALDO EMERSON) = T
```

这是一个让哈希表变得有效率的取舍之一。有了列表，我们可以指定 member 为判断相等性的谓词。有了哈希表，我们可以预先决定，并在哈希表构造时指定它。

大多数 Lisp 编程的取舍（或是生活，就此而论）都有这种特质。起初你想要事情进行得流畅，甚至赔上效率的代价。之后当代码变得沉重时，你牺牲了弹性来换取速度。


## 总结(Summary) {#总结--summary}

1.  Common Lisp 支持至少 7 个维度的数组。一维数组称为向量。

2.  字符串是字符的向量。字符本身就是对象。

3.  序列包括了向量与列表。许多序列函数都接受标准的关键字参数。

4.  处理字符串的函数非常多，所以用 Lisp 来解析字符串是小菜一碟。

5.  调用 defstruct 定义了一个带有命名字段的结构。它是一个程序能写出程序的好例子。

6.  二叉搜索树见长于维护一个已排序的对象集合。

7.  哈希表提供了一个更有效率的方式来表示集合与映射 (mappings)。


## 习题 (Exercises) {#习题--exercises}

(1) 定义一个函数，接受一个平方数组（square array，一个相同维度的数组 (n n) )，并将它顺时针转 90 度。

```lisp
(defun quarter-turn (arr)
  (let ((a (aref arr 0 0))
        (b (aref arr 0 1))
        (c (aref arr 1 0))
        (d (aref arr 1 1))))
  #2a((c a) (d b)))

(op (quarter-turn #2A((a b) (c d))))
```

```text
#2A((C A) (D B))
```

(2) 使用 reduce 来定义以下函数：

(a) copy-list

```lisp
(defun my-copy-list-slow (lst)
  (reduce #'(lambda (acc item)
              (append acc (list item)))
          lst
          :initial-value nil))

(defun my-copy-list (lst)
  "更高效的实现"
  (reverse (reduce #'(lambda (acc item) (cons item acc))
                   lst
                   :initial-value nil)))

(op (my-copy-list-slow '(a b c)))
(op (my-copy-list '(a b c (d e h))))
```

```text
(A B C)
(A B C (D E H))
```

(b) reverse（针对列表）

```lisp
(defun my-reverse (lst)
  (reduce #'(lambda (acc item) (cons item acc))
          lst
          :initial-value nil))

(op (my-reverse '(a b c)))
```

```text
(C B A)
```

(3) 定义一个函数，接受一棵二叉搜索树，并返回由此树元素所组成的，一个由大至小排序的列表。

```lisp
(defun bst-inorder (bst)
  "中序遍历：右子树 -> 根节点 -> 左子树（对于BST会得到有序序列）"
  (when bst
    (append (bst-inorder (node-r bst))
            (list (node-elt bst))
            (bst-inorder (node-l bst)))))
;; 创建空树
(setf tree nil)
;; 插入元素（需要比较函数）
(setf tree (bst-insert 5 tree #'<))
(setf tree (bst-insert 3 tree #'<))
(setf tree (bst-insert 7 tree #'<))
(setf tree (bst-insert 1 tree #'<))
(setf tree (bst-insert 9 tree #'<))

(print-tree tree)
(op (bst-inorder tree))
```

```text
      ┌── 9
  ┌── 7
5
  └── 3
      └── 1
(9 7 5 3 1)
```

(5) 定义 bst-adjoin 。这个函数应与 bst-insert 接受相同的参数，但应该只在对象不等于任何树中对象时将其插入。同 bst-insert

(6) 任何哈希表的内容可以由关联列表（assoc-list）来描述，其中列表的元素是 (k . v) 的形式，对应到哈希表中的每一个键值对。定义一个函数：

(a) 接受一个关联列表，并返回一个对应的哈希表。

```lisp
(defun assoc2hash (trans)
  (let ((ht (make-hash-table)))
    (mapcar #'(lambda (x)
                (setf (gethash (car x) ht) (cdr x)))
            trans)
    ht))

(setf ht (assoc2hash '((+ . "add") (- . "sub") (nmae . "Jason"))))

(maphash #'(lambda (k v)
           (format t "~A = ~A~%" k v))
       ht)
```

```text
+ = add
- = sub
NMAE = Jason
```

(b) 接受一个哈希表，并返回一个对应的关联列表。

```lisp
(defun hash2assoc (ht)
  (let ((assoc nil))
    (maphash #'(lambda (k v)
                 ;;(format t "~A = ~A~%" k v)
                 (push (cons k v) assoc))
             ht)
    assoc))

(setf ass (hash2assoc ht))
(op ass)
```

```text
((NMAE . Jason) (- . sub) (+ . add))
```
