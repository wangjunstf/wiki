---
title: "第二章：欢迎来到 Lisp"
author: ["ubuntu"]
date: 2025-11-13T18:27:00+08:00
draft: false
---

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
