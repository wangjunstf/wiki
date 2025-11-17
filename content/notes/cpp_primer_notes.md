---
title: "C++ Primer Notes"
author: ["ubuntu"]
date: 2025-10-15T20:44:00+08:00
draft: false
---

## 第六章 函数 {#第六章-函数}


### 顶层const和底层const {#顶层const和底层const}

-   核心区别：

    -   指对象本身是常量
    -   修饰变量本身

    <!--listend-->

    -   底层const：
        -   指针或引用所指向的对象是常量
        -   修饰指针或引用指向的对象：表示不能通过指针或引用修改其指向的对象

<!--listend-->

-   示例：
    ```cpp
    int i = 0;

    // 顶层 const
    const int ci = 42;         // ci 的值不能改变
    int* const p1 = &i;        // p1 的指向不能改变（永远指向i）

    // 底层 const
    const int* p2 = &ci;       // 不能通过 p2 修改 ci 的值
    const int& r1 = ci;        // 不能通过 r1 修改 ci 的值

    // 既是顶层又是底层 const
    const int* const p3 = &ci; // p3 的指向和它所指对象的值都不能改变
    ```

<!--listend-->

-   顶层const不参与函数重载
    -   如下列函数重复定义了：
        ```cpp
        // 接受底层const
        int calc(const char *a, const char *b) {
          cout << "calc(const char* a, const char* b)" << endl;
          return 0;
        }

        // 顶层const不参与函数重载，所以跟上面的函数重复定义了。
        int calc(const char *const a, const char *const b) {
          cout << "calc(const char* a, const char* b)" << endl;
          return 0;
        }

        ```

    -   同样地下列函数也重复定义了：
        ```cpp
        int calc(char *a, char *b) {
          cout << "calc(char *a, char* b)" << endl;
          return 0;
        }

        // 顶层const不参与函数重载，所以跟上面的函数重复定义了。
        int calc(char *const a, char *const b) {
          cout << "calc(char *const a, char*const b)" << endl;
          return 0;
        }
        ```

    -   完整例子：
        ```cpp
        #include <iostream>

        using namespace std;

        int calc(const char *const a, const char *const b) {
          cout << "calc(const char* const a, const char* const b)" << endl;
          return 0;
        }

        int calc(char *const a, char *const b) { // 接受顶层
          cout << "calc(char *const a, char*const b)" << endl;
          return 0;
        }

        // 与 calc(char *const a, char *const b)
        // 构成重复定义，因为顶层const不参与函数重载
        // int calc(char *a, char *b) {
        //   cout << "calc(char *a, char* b)" << endl;
        //   return 0;
        // }

        // 与 calc(const char *const a, const char *const b)
        // 构成重复定义，因为顶层const不参与函数重载
        // int calc(const char *a, const char *b) { // 接受底层const
        //   cout << "calc(const char* a, const char* b)" << endl;
        //   return 0;
        // }

        int main() {
          char a;
          const char *const b = &a; // b 即是顶层const，也是底层const

          const char *c = &a; // 底层const

          char *const d = &a; // 顶层const

          const char *const e = &a;
          calc(c, c);
          calc(d, d);
          calc(e, e);
          return 0;
        }
        ```

        ```text
        calc(const char* const a, const char* const b)
        calc(char *const a, char*const b)
        calc(const char* const a, const char* const b)
        ```

<!--listend-->

-   顶层const和底层const的相互转换：
    -   底层不能直接转顶层:
        ```cpp
        char a;
        const char *c = &a; // 底层const
        // 底层const不能直接转顶层const，需要使用 const_cast
        //char *const f = c;
        char *const f = const_cast<char *>(c);
        ```

    -   顶层可以直接转底层：
        ```cpp
        char a;
        char *const d = &a; // 顶层const

        // 顶层const可以直接转底层const
        const char *g = d;
        ```

    -   顶层/底层可以直接转同时是顶层和底层
        ```cpp
        char a;
        const char *c = &a; // 底层const
        char *const d = &a; // 顶层const

        const char *const h = c;
        const char *const i = d;
        ```


### 函数重载 {#函数重载}

-   对于重载的函数：
    1.  形参数量不同。
    2.  形参类型不同。

<!--listend-->

-   `const 形参` 和重载：
    -   一个拥有顶层 const 的形参无法和另一个没有顶层 const 的形参区分开来。
        ```cpp
        // 以下两个函数无法重载区分
        Record lookup(Phone);
        Record lookup(const Phone);   // 顶层const

        // 以下两个函数无法重载区分
        Record lookup(Phone*);
        Record lookup(Phone* const);  // 顶层const
        ```

    -   如果形参是某种类型的指针或引用，则通过其指向的是常量对象还是非常量对象来实现函数重载，此时的 const 是底层的。以下是四个同名的重载函数：
        ```cpp
        Record lookup(Account &);             // 作用于 Account 的引用
        Record lookup(const Account &);       // 作用于常量引用
        Record lookup(Account *);             // 作用于指向 Account 的指针
        Record lookup(const Account *);       // 作用于指向常量的指针
        ```

-   `const_cast` 和重载
    -   我们可以利用 const_cast 来根据实参的类型来返回相对应的类型，例如：实参是const，返回const引用，当实参是普通对象，返回普通引用。
        ```c
        // 比较两个 string 对象的长度，返回较短的那个引用
        const string &shorterString(const string &s1, const string &s2){
          return s1.size() <= s2.size() ? s1 : s2;
        }

        string &shorterString(string &s1, string &s2){
          auto &r = shorterString(const_cast<const string&>(s1),const_cast<const string&>(s2));
          return const_cast<string&>(r);
        }
        ```

        -   当实参是 `const 对象` 时，调用第一个函数，并返回 `const 引用` 。
        -   当实参是 `普通对象` 时，调用第二个函数，并返回 `普通引用` 。


### 内联函数和constexpr函数 {#内联函数和constexpr函数}


#### 函数调用的代价 {#函数调用的代价}

在C++中一个普通的函数调用主要有以下成本：

| 阶段   | 说明         | 是否可优化    |
|------|------------|----------|
| 参数传递 | 将实参复制到调用者栈帧 | 编译器可能用寄存器传参 |
| 返回地址保存 | 将下一条指令地址压入栈中 | 硬件自动完成，无法省略 |
| 栈帧建立 | 为被调用函数创建新的栈帧 | 可优化为尾调用或寄存器分配 |
| 函数体执行 | 执行函数体   | 必须          |
| 栈帧销毁 | 恢复调用者栈帧、返回 | 尾调用优化时可省略 |

内联函数可以避免函数调用的开销

-   将函数指定为内联函数(inline)，就是在每个调用点上"内联地"展开。
-   如下所示：我们将square_inline声明为内联函数
    ```cpp
    inline int square_inline(int x) {
        return x * x;
    }

    int main() {
        int a = 5;
        int r1 = a * a;          // 直接执行表达式
        int r2 = square_inline(a); // 函数调用
    }
    ```

    -   这样在计算 r2 时就不会产生函数调用开销，直接就可以计算 `a*a` 的值。
    -   注意：内联说明只是向编译器发出一个请求，编译器可以忽略这个请求。


#### constexpr 函数 {#constexpr-函数}

-   constexpr 函数是指能用于常量表达式的函数。定义constexpr需要遵循以下约定：
    -   一个函数被声明为 `constexpr` ，意味着它的返回值必须能在编译期计算出来（当其参数是常量时）。
    -   函数的 `返回类型` 及所有 `形参类型` 都必须是 `字面值类型` 。
    -   函数体必须有且只有一条 `return语句` 。
    -   constexpr 函数被隐式地指定为内联函数。
    -   constexpr 函数可以包含其他语句，只要这学语句在运行时不执行任何操作，例如：空语句，类型别名，using 声明。

<!--listend-->

-   constexpt 代码示例：
    ```cpp
    constexpr int new_sz() {return 42;}
    constexpr int foo = new_sz();  // 正确：foo是一个常量表达式
    ```

<!--listend-->

-   constexpr 函数的返回值可以不是一个常量：
    ```cpp
    // 如果 arg 是常量表达式，则 scale(arg)也是常量表达式
    constexpr size_t scale(size_t cnt) { return new_se() * cnt; }

    int arr[scale(2)];      // 正确：scale(2)是常量表达式
    int i = 2;
    int a2[scale(i)];       // 错误：scale(i) 不是常量表达式
    ```

<!--listend-->

-   把内联函数和constexpr函数放到头文件内：
    -   和其他函数不一样，内联函数和 constexpr 函数可以定义多次，毕竟编译器需要获得函数定义，以便在函数调用出展开。
    -   对于给定的内联函数和 constexpr 函数，它的多个定义必须完全一致。
    -   基于以上两点：内联函数和 constexpr 函数通常定义在头文件中。


### 函数指针 {#函数指针}

-   函数指针的类型由以下两方面共同决定：
    -   返回类型
    -   形参类型
    -   例如：
        ```cpp
        // 比较两个 string 对象的长度
        bool lengthCompare(const string&, const string&);

        // pf 指向一个函数,该函数的参数是两个 const string 的引用，返回值是 bool 类型。
        bool (*pf)(const string&, const string&);
        ```

<!--listend-->

-   函数指针的使用
    -   和数组一样，把函数名作为一个值使用时，该函数自动转换成指针，例如以下两条语句等价：
        ```cpp
        pf = lengthCompare;      // pf指向名为 lengthCompare 的函数
        pf = &lengthCompare;     // 等价的赋值语句
        ```

    -   我们可以直接使用指向函数的指针调用该函数，无需提前解引用指针：
        ```cpp
        bool b1 = pf("hello","goodbye");
        bool b2 = (*pf)("hello","goodbye");
        bool b3 = lengthCompare("hello","goodbye");
        ```

    -   不同函数类型的指针间不存在转换规则，我们可以为函数指针赋予一个 `nullptr` ，表示不指向任何指针。

-   重载函数的指针：函数指针必须与重载函数中的某一个精确匹配。

-   函数指针形参：和数组类似，虽然不能定义函数类型的形参，可以定义函数指针类型的形参。
    -   下面的写法像是函数类型，但其实是指针类型：
        ```cpp
        void useBigger(const string &s1, const string &s2, bool pf(const string &, const string &));
        ```

    -   下面是等价的声明：
        ```cpp
        void useBigger(const string &s1, const string &s2, bool (*pf)(const string &, const string &));
        ```

<!--listend-->

-   函数指针别名
    -   使用 `类型别名` 和 `decltype` 可以简化函数指针的声明。
        ```cpp
        // Func 和 Func2 是函数类型
        typedef bool Func(const string&, const string&);
        typedef decltype(lengthCompare)Func2;            // 等价的类型

        // FuncP 和 FuncP2 是指向函数的指针
        typedef bool(*FuncP)(const string&, const string&);
        typedef decltype(lengthCompare) *FuncP2;
        ```

    -   类型别名使用：
        ```cpp
        void useFunction(Func f) { // f 是一个函数
          cout << boolalpha << f("hello", "hi") << endl;
        }

        FuncP p1 = lengthCompare; // ✅ 函数名自动转换为指针
        FuncP2 p2 = &lengthCompare; // ✅ 也可以显式取地址

        // 以下两条函数声明等价：
        void useBigger(const string&, const string&, Func);
        void useBigger(const string&, const string&, Funcp2);
        ```

<!--listend-->

-   返回函数指针的函数
    -   使用 using 声明函数类型和函数指针类型
        ```cpp
        using F = int(int*, int);       // F是函数类型，不是指针类型
        using PF = int(*)(int*, int);   // PF是指针类型
        ```

    -   声明返回函数指针的函数
        ```cpp
        PF f1(int);   // 正确：PF是函数指针类型
        F f1(int);    // 错误：F是函数类型，f1不能返回一个函数
        F* f1(int);   // 正确：显示地指定返回类型是指向函数的指针
        ```

    -   直接声明 `f1`
        ```cpp
        int (*f1(int)) (int*, int);
        ```

        -   按照由内向外的顺序阅读上述声明语句：
            -   首先 f1 有一个形参列表，所以f1是一个函数；
            -   f1 前面有一个 `*` ，所以f1返回一个指针；
            -   指针的类型也包含形参列表，因此指针指向函数，该函数的返回类型是int

    -   使用尾置返回类型：
        ```cpp
        auto f1(int) -> int (*)(int*, int);
        ```

<!--listend-->

-   auto 和 decltype 用于函数指针类型

    -   假如我们知道返回的函数是哪一个，就可以使用 decltype 简化书写函数指针返回类型的过程：
        ```cpp
        string::size_type sumLength(const string&, const string&);
        string::size_type largerLength(const string&, const string&);

        // 根据其形参的取值，getFcn 函数返回指向sumLength或largerLength的指针。
        decltype(sumLength) *getFcn(const string&);
        ```
        声明 getFcn，注意decltype作用于某个函数时，它返回函数类型而非指针类型。

    > [How to use decltype in c++]({{< relref "how_to_use_decltype_in_cpp.md" >}})


## 第七章 类 {#第七章-类}

类的基本思想是 `数据抽象(data abstraction)` 和 `封装(encapsulation)` 。数据抽象是一种依赖于 `接口(interface)` 和 `实现(implementation)` 分离的编程技术。

-   类的接口：用户能执行的操作。
-   类的实现：
    -   类的数据成员。
    -   负责接口实现的函数体。
    -   定义类所需要的各种私有函数。

-   封装实现了类的接口和实现分离。

-   编译器分两步处理类：
    -   首先编译成员声明。
    -   然后编译成员函数体。

<!--listend-->

-   内置的赋值运算符把它的 `左侧运算对象` 当成左值返回，因此我们定义多个对象的 `+`, `-` , `*`, `/` 运算时，最好返回引用类型。如下所示：
    ```cpp
    Sales_data& Sales_data::combine(const Sales_data &rhs){
      units_sole += rhs.units_sold;  // 把 rhs 的成员加到 this 对象的成员上。
      revenue += rhs.revenue;
      return *this;
    }
    ```

<!--listend-->

-   类的作者常常定义一些辅助函数。
    -   比如 add、read 和 print 等。这些函数从概念上来说属于类的接口的组成部分，但它们不属于类本身。
    -   如果函数在概念上属于类但不定义在类中，则它一般应与类声明（而非定义）在同一个头文件中。
    -   这样，用户使用接口的任何部分都只需引入一个头文件。

<!--listend-->

-   IO类属于不能被拷贝的类型，所以我们只能通过引用来传递它们。如下所示：
    ```cpp
    void print(std::ostream &os, Sales_data &sales) {
      os << sales.bookNo << " " << sales.units_sold << " " << sales.revenue;
    }
    ```
    像上述 print 函数不应该输出换行，这样可以确保由用户代码来决定是否换行。


### 构造函数 {#构造函数}

构造函数不能被声明为 const 的，当我们创建一个 const 对象时，直到 `构造函数` 完成初始化过程，对象才能取得 `“常量属性”` 。因此，构造函数在 const 对象的创造过程中可以向其写值。

-   合成的默认构造函数按以下规则初始化类内成员：
    -   如果存在类内初始值，则用它来初始化成员。
    -   否则，默认初始化该成员。

<!--listend-->

-   构造函数的初始值列表如果成员是const、引用或者属于某种未提供默认构造函数的类类型，我们必须通过构造函数初始值列表为这些成员提供初始值。
    ```cpp
    #include <iostream>

    using namespace std;

    class A {
    public:
      A(int _a) : a(_a), b(a+100) {}
      int get_a(){ return a;}
      int get_b() {return b;}
    private:
      int a;
      int b;
    };

    class B {
    public:
      B() : a(A(100)) {}

      void echo() {
        cout << a.get_a() << endl;
        cout << a.get_b() << endl;
      }

    private:
      A a;
    };
    int main() {
      cout << "Hello world" << endl;
      B b;
      b.echo();
      return 0;
    }
    ```


### 友元 {#友元}

类和非成员函数的声明不是必须在它们的友元声明之前。

```cpp
#include <iostream>

using namespace std;

class X {
public:
  friend void f();
  using ll = long long;
};

void f() { cout << "f()" << endl; }
int main() {
  f();
  X::ll l = 100; // 使用 X 中定义的类型别名
  cout << l << endl;
  return 0;
}
```


### 类内部的类型重定义 {#类内部的类型重定义}

```cpp
#include <iostream>

using namespace std;

typedef double Money;

int age = 100;

class Account {
public:
  //Money balance() {return bal;}
  int get_age(){ return age;}
private:
  typedef int Money;
  Money bal;
  int age = 10;

};

int main(){
  cout<< "H" << endl;
  Account acc;
  cout << acc.get_age() << endl;
}
```


### 全局作用域 {#全局作用域}

当类内定义了一个和类外的同名变量时，要指定使用类外定义的变量，可以使用全局作用域运算符 `::`


### 默认初始化和值初始化 {#默认初始化和值初始化}

```cpp
#include <iostream>

using namespace std;

class Foo{
public:
  Foo() { cout << "Foo()" << endl;};
  Foo(int _age):age(_age){}
private:
  int age;
};

Foo foo;
int main(){
 // Foo foo[10];
  static Foo f;
  cout << "hello world"  << endl;
  return 0;
}
```


### 隐式的类类型转换 {#隐式的类类型转换}

```cpp
#include <iostream>
#include <string>
#include <vector>

using namespace std;

class Foo{
public:
  int num;
  Foo(int n):num(n){
    cout << "Foo()" << endl;
  }

  void make_foo(const Foo& foo){
    cout << "make_foo num of Foo "<< foo.num<< endl;
  }
};

void test_vec(vector<int> vec){
  cout << "test_vec size " << vec.size() << endl;
}

int main(){
  Foo f(10);
  f.make_foo(125); // 调用make_foo时，形参125被转换为Foo类型，自动调用它接受一个int的构造函数。
  string str = "Hello world";
  test_vec(vector<int>(11));
  test_vec(static_cast<vector<int>>(125));
  return 0;
}
```

> 使用 explicit可以抑制上述隐式类型转换，只需在相应的构造函数前加上 `explicit`
> 通过使用 `static_cast` 我们也可以绕过 `explicit` 的限制。


### 7.5.6 节练习 {#7-dot-5-dot-6-节练习}


#### 7.53 定义你自己的 Debug {#7-dot-53-定义你自己的-debug}

```cpp
#include <iostream>

using namespace std;

class Debug {
public:
  constexpr Debug(bool b = true) : hw(b), io(b), other(b) {}
  constexpr Debug(bool h, bool i, bool o) : hw(h), io(i), other(o) {}
  constexpr bool any() const {return hw || io || other;}

  void set_io(bool b) {io = b;}
  void set_hw(bool b) {hw = b;}
  void set_other(bool b) {other = b;}

private:
  bool hw;    // 硬件错误，而非IO错误
  bool io;    // IO 错误
  bool other; // 其他错误
};

int main() {
  constexpr Debug io_sub(false, true, false);          // 调试 IO
  if (io_sub.any())                                    // 等价于 if(true)
    cerr << "print appropriate error messages"<<endl;
  constexpr Debug prod(false);                         // 无调试
 // prod.set_io(true);
  if (prod.any())                                      // 等价于 if(false) 这段代码直接会被编译器优化调
    cerr << "print an error message" << endl;
  return 0;
}
```


### 类的静态成员 {#类的静态成员}

```cpp
#include <iostream>

using namespace std;

class Foo {
public:
  static int num;
  static constexpr int age = 27;
  static constexpr double rate = 6.5;
  static void setNum(int n) { num = n; }
};

int Foo::num = 1024;

int main() {
  cout << Foo::age<< endl;
  Foo f;
  cout << f.num << endl;
  cout << Foo::num << endl;

  return 0;
}
```

当需要把一个静态成员变量传递给一个非成员函数时，该静态成员变量必须提供类外定义。

```cpp
#include <iostream>

using namespace std;

class Account {
public:
  static constexpr int period = 200;
  static int age;
};

int Account::age = 12;

int foo(const int& n){
  return n+1;
}
int main(){
  cout << foo(Account::period) << endl;
  return 0;
}
```


### 术语表 {#术语表}

-   常量成员函数一个成员函数不能修改对象的普通数据成员（既不是 static 也不是 mutable）
    ```cpp
    #include <iostream>

    using namespace std;

    class Foo{
    public:
      void set(int n) const {
        age = n;
      }

      static int age;
      int num;
    };

    int Foo::age = 25;

    int main(){
      cout << "hello" << endl;
      Foo f;
      f.set(12);
      cout << f.age << endl;
    }
    ```


## 第八章 IO 库 {#第八章-io-库}

-   istream 输入流类型，提供输入操作
-   ostream 输出流类型，提供输出操作
-   cin  一个 istream 对象，从标准输入读取数据
-   cout 一个 ostream 对象，向标准输出写入数据
-   cerr 一个 ostream 对象，通常用于输出程序错误消息，写入到标准错误
-   `>>` 运算符 用来从一个 istream 对象读取输入数据
-   `<<` 运算符 用来向一个 ostream 对象写入输出数据
-   getline 函数 从一个给定的 istream 读取一行数据，存入一个给定的 string 对象中


### 8.1 IO 类 {#8-dot-1-io-类}

-   IO 库类型和头文件
    -   iostream
        -   istream 从流读取数据
        -   ostream 向流写入数据
        -   iostream 读写流
    -   fstream
        -   ifstream 从文件读取数据
        -   ofstream 向文件写入数据
        -   fstream 读写文件
    -   sstream
        -   istringstream 从 string 读取数据
        -   ostreamstream 向 string 写入数据
        -   stringstream 读写 string

> 在C++中，宽字符是用于表示扩展字符集（如Unicode）的字符类型，主要解决传统char类型无法充分表示国际字符的问题。
>
> ```cpp
> #include <iostream>
> #include <string>
>
> // 宽字符类型
> wchar_t wideChar = L'中';  // 注意前缀 L
>
> // 宽字符串
> const wchar_t* wideString = L"你好世界";
> std::wstring wideStr = L"宽字符串示例";
> ```
>
> 更多关于宽字符的概念：[宽字符]({{< relref "宽字符.md" >}})

-   宽字符为了支持使用宽字符


#### 8.1.1 IO 对象无拷贝或赋值 {#8-dot-1-dot-1-io-对象无拷贝或赋值}

-   由于不能拷贝 IO 对象，因此不能将形参或返回类型设置为流类型。
-   进行 IO 操作通常以引用方式传递和返回流。
-   读写一个 IO 对象会改变其状态，因此传递和返回的引用不能是 const 的。


#### 8.1.2 条件状态 {#8-dot-1-dot-2-条件状态}

IO 操作可能发生错误，一些错误可以恢复，而其他错误则发生在系统深处。下表列出了 IO 类所定义的一些函数和标志，可以辅助我们访问和操纵流的条件状态（condition state）。

> 继承关系：
>
> std::ios_base::iostate
>     ↑
> std::basic_ios&lt;char&gt;::iostate
>     ↑
> std::ifstream 继承自 std::basic_ios&lt;char&gt;

````cpp
#include <iostream>
#include <fstream>
#include <typeinfo>

int main() {
    std::ifstream file("data.txt");

    // 获取返回类型
    std::ifstream::iostate state = file.rdstate();
    //std::basic_ios<char>::iostate state = file.rdstate();
    //std::ios_base::iostate state = file.rdstate(); // 可以使用 auto 简化类型表示


    std::cout << "rdstate() 返回类型: " << typeid(state).name() << std::endl;
    std::cout << "当前状态值: " << state << std::endl;

    // 与状态常量比较
    if (state == std::ios_base::goodbit) {
        std::cout << "流状态正常" << std::endl;
    }

    // 使用位运算检查特定状态
    if (state & std::ios_base::eofbit) {
        std::cout << "包含 eofbit" << std::endl;
    }

    return 0;
}
````

<!--list-separator-->

-  IO 库条件状态

    1.  strm::iostate  strm 是一种 IO 类型，iostate 是一种机器相关类型，提供了表达条件状态的完整功能。
    2.  strm::badbit   用来指出流已经崩溃
    3.  strm::failbit  用来指出一个IO操作失败了
    4.  strm::eofbit   用来指出流到达了文件结束
    5.  strm::goodbit  用来指出流未处于错误状态，次值保证为零。

    <!--listend-->

    -   s.eof()     若流 s 的 eofbit 置位，则返回 true
    -   s.fail()    若流 s 的 failbit 或 badbit 置位，则返回 true
    -   s.bad()     若流 s 的 badbit 置位，则返回 true
    -   s.good()    若流 s 处于有效状态，则返回 true
    -   s.clear()   将流 s 中所有条件状态位复位，将流的状态设置为有效，返回 void。
    -   s.clear(flags)     根据给定的 flags 标志位，将流 s 中对应条件状态 `复位` 。flags 的类型位 strm::iostate。返回 void
    -   s.setstate(flags)  根据给定的 flags 标志位，将流 s 中对应条件状态 `置位` 。flags 的类型位 strm::iostate。返回 void
    -   s.rdstate()  返回流 s 的当前条件状态，返回置类型为 strm::iostate

    <!--listend-->

    -   一个 IO 错误例子：
        ````cpp
        #include <iostream>
        #include <sstream>
        using namespace std;

        void test_stream_state() {
            stringstream ss;
            int value;

            // 情况1: 正常读取
            ss.str("42");
            ss >> value;
            cout << "正常读取: " << boolalpha << static_cast<bool>(ss) << endl;

            // 情况2: 类型不匹配
            ss.clear();
            ss.str("abc");
            ss >> value;
            cout << "类型错误: " << static_cast<bool>(ss) << endl;

            // 情况3: 到达末尾
            ss.clear();
            ss.str("100");
            ss >> value;
            ss >> value;  // 第二次读取会失败
            cout << "到达末尾: " << static_cast<bool>(ss) << endl;

            // 情况4: 严重错误
            // 通常需要模拟硬件错误，这里不演示
        }

        int main() {
          test_stream_state();
          return 0;
        }

        ````
        如果我们在标准输入上键入字符'B'，度操作就会失败，代码中的输入运算符期待读取一个 int，但却得到一个字符B。

        类似的，当我们输入一个文件结束标识，cin 也会进入错误状态。

        确定一个流对象的状态的最简单的方法是将它作为一个条件来使用：
        ````cpp
        while(cin >> world)
          // 读操作成功
        ````
        上述代码的原理是：

        cin &gt;&gt; value返回 std::istream&amp;，而 std::istream类重载了 operator bool()或 operator void\*()（旧标准），使得流对象可以在布尔上下文中使用。

        -   如果流状态正常（good()返回 true），转换为 true
        -   如果流状态异常（fail()或 bad()），转换为 false
            ````cpp
            #include <iostream>

            using namespace std;

            int main() {
              int value;

              // 简介写法
              while (cin >> value) {
                // 处理 value
              }

              // 详细写法
              while (true) {
                cin >> value;
                if (!cin) { // 等价于 if(cin.fail())
                  break;
                }
                // 处理value
              }

              // 布尔转换的等价检查
              bool condition1 = static_cast<bool>(cin); // C++11 显式转换
              bool condition2 = !cin.fail();            // 功能等价
              bool condition3 = cin.good(); // 不完全等价，good()要求更严格

              cout << "bool转换: " << condition1 << endl;
              cout << "!fail(): " << condition2 << endl;
              cout << "good(): " << condition3 << endl;

              return 0;
            }
            ````

<!--list-separator-->

-  查询流的状态

    -   badbit 表示系统级错误，该错误意味着流无法再使用。
    -   failbit 表示一般读写错误，可以被恢复。
    -   若到达文件结束位置：eofbit 和 failbit 都会被置位。
    -   goodbit 的值为0,表示流未发生错误。

<!--list-separator-->

-  管理条件状态

    -   流对象的 rdstate 成员返回一个 iostate 值，对应流的当前状态。
    -   setstate 操作将给定条件位置位，表示发生了对应错误。
    -   clear 不接受参数的版本清除所有错误标志位，再次调用 good 会返回 true。

    使用示例：

    ````cpp
    auto old_sate = cin.rdstate();   // 记住 cin 的当前状态
    cin.clear();                     // 使 cin 有效
    process_input(cin);              // 使用 cin
    cin.setstate(old_state);         // 将 cin 置为原有状态
    ````

    使用 clean 复位单一的条件状态位：

    ````cpp
    // 复位 failbit 和  badbit，保持其他标志位不变
    cin.clear(cin.rdstate() & ~cin.failbit & ~cin.badbit);
    ````


#### 8.1.2 节练习 {#8-dot-1-dot-2-节练习}

-   8.1
    ````cpp
    #include <iostream>
    #include <istream>

    std::istream &read(std::istream &cin) {
      char c;
      while (cin >> c && !cin.eof()) {
        std::cout << c;
      }

      cin.clear();
      return cin;
    }

    int main() {
      read(std::cin);
      std::cout << "程序结束" << std::endl;
      return 0;
    }
    ````


#### 8.1.3 管理缓冲区 {#8-dot-1-dot-3-管理缓冲区}

每个输出流都管理一个缓冲区，用来保存程序读写的数据。

手动刷新缓冲区的方式：

````cpp
#include <iostream>

using namespace std;

int main(){
  cout << "hi" << endl;    // 输出 hi 和一个换行，然后刷新缓冲区
  cout << "hi" << flush;   // 输出 hi,然后刷新缓冲区，不附加任何额外的字符
  cout << "hi" << ends;    // 输出 hi 和一个空字符，然后刷新缓冲区，空字符也就是 '\0'
  return 0;
}
````

除了上述三种方式，还可以使用 unitbuf 来刷新缓冲区

````cpp
cout << unitbuf;    // 所有输出操作后都会立即刷新缓冲区
// 任何输出操作立即刷新，无缓冲

cout << nounitbuf;  // 回到正常的缓冲方式
````

> 如果程序异常终止，输出缓冲区是不会被刷新的。

关联输入流和输出流：当一个输出流被关联到一个输出流时，任何试图从输入流读取数据的操作都会先刷新关联的输出流。标准库将 cout 和 cin 关联在一起，每次使用 cin都会导致 cout 的缓冲区被刷新。

我们可以使用 tie 将一个流关联到另一个ie流。

-   tie 不带参数，返回指向输出流的指针，入股对象未关联到流，则返回空指针。
-   tie 接受一个指向 ostream 的指针，将自己关联到此 ostream。即 x.tie(&amp;o) 将流 x 关联到输出流 O。


### 8.2 文件输入输出 {#8-dot-2-文件输入输出}

-   头文件 fstream 定义了三个类型来支持文件IO
    -   ifstream 从一个给定的文件读取数据。
    -   ofstream 向一个给定文件写入数据。
    -   fstream 可以同时读和写一个文件。

<!--listend-->

-   fstream 特有的操作有：

    | 项目                  | 解释                        |
    |---------------------|---------------------------|
    | fstream fstrm         | 创建一个未绑定的文件流      |
    | fstream fstrm(s)      | 创建一个iefstream，并打开名为 s 的文件 |
    | fstream fstrm(s,mode) | 同前，但按照指定 mode 打开文件 |
    | fstrm.open(s)         | 打开名为 s 的文件，并将文件与 fstrm 绑定 |
    | fstrm.close()         | 关闭与 fstrm 绑定的文件，返回 void |
    | fstrm.is_open()       | 返回一个 bool 值，支持文件是否成功打开且还未关闭 |


#### 8.2.1 使用文件流对象 {#8-dot-2-dot-1-使用文件流对象}

<!--list-separator-->

-  用 fstream 代替 iostream&amp;

    ````cpp
    std::istream &read(std::istream &is, Sales_data &sales) {
      double price = 0.0;
      if (is >> sales.bookNo >> sales.units_sold >> price) {
        sales.revenue = sales.units_sold * price;
      } else {
        sales = Sales_data();
      }

      return is;
    }

    std::ostream &print(std::ostream &os, Sales_data &sales) {
      os << sales.bookNo << " " << sales.units_sold << " " << sales.revenue
         << std::endl;
      return os;
    }
    ````

    函数 read 接受一个 istream 对象的引用，我们既可以向其传递 std::cin 也可以向其传递 ifstream 对象。

    ````cpp
    Sales_data total;
    read(std::cin, total);         // 从标准输入读取数据

    ifstream input("input.txt");
    read(input,total);             // 从给定文件流读取数据
    ````

    函数 print 接受一个 ostream 对象的引用作为参数，我们既可以向其 std::cout,也可以向其传递 ofstream。

    ````cpp
    Sales_data total;
    print(std::cout, total);

    ofstream output("output.txt");
    print(output, total);
    ````

    > 类似上述 read 和 print 函数，基类引用或指针可以指向派生类对象，通过这种方式，我们可以通过一个接口实现不同的操作，这种技术就是“多态”。

<!--list-separator-->

-  成员函数 open 和 close

    我们可以先调用一个空文件流对象，随后再调用 open 来将它与文件关联起来：

    ````cpp
    ifstream in(ifile);       // 构造一个 ifstream 对象并与 ifile 关联
    ofstream out;             // 同理，但未与任何文件关联
    out.open(ifile+".copy")   // 打开指定文件
    ````

    如果调用 open 失败， failbit 会被置位，所以进行 open 是否成功的检测通常是一个好习惯: if(out)

    为了将文件流关联到另一个文件，必须首先关闭已经关联的文件：

    ````cpp
    in.close();                 // 关闭文件
    in.open(ifile + ".bak");    // 打开另一个文件

    // 如果 open 成功，则 open 会设置流的状态，使得 good() 为 true。
    ````

<!--list-separator-->

-  自动构造和析构

    每一个打开的文件，系统都会为其分配一个文件描述符，而文件描述符的数量是有限的，及时关闭不使用的文件是一个好习惯。

    如果在程序中，特别是循环中，持续创建文件流并关联文件，但不关闭，容易造成资源泄漏。

    为了防止潜在的资源泄漏问题，我们可以使用类来管理文件流，通过在构造函数中创建文件流对象，在析构函数中自动关闭，能减少这类资源泄漏问题。

<!--list-separator-->

-  8.2.1 节练习

    练习 8.4

    ````cpp
    #include <fstream>
    #include <iostream>
    #include <string>
    #include <vector>

    using namespace std;

    int main(){
      ifstream ifile("input.txt");
      vector<string> vec;
      if(ifile.is_open()){
        string line;
        while(getline(ifile,line)){
          vec.push_back(line);
        }
      }

      for(auto& x: vec){
        cout << x << endl;
      }
      return 0;
    }

    ````

    练习 8.5

    ````cpp
    #include <fstream>
    #include <iostream>
    #include <string>
    #include <vector>

    using namespace std;

    int main(){
      ifstream ifile("input.txt");
      vector<string> vec;
      if(ifile.is_open()){
        string line;
        while(ifile>> line){
          vec.push_back(line);
        }
      }

      for(auto& x: vec){
        cout << x << endl;
      }
      return 0;
    }
    ````

    练习 8.6

    ````cpp
    #include <fstream>
    #include <iostream>
    #include <istream>
    #include <string>

    struct Sales_data {
      std::string bookNo;
      unsigned units_sold = 0;
      double revenue = 0.0;

      // 成员函数
      std::string isbn() { return this->bookNo; }
      Sales_data &combine(const Sales_data &rhs) {
        units_sold += rhs.units_sold;
        revenue += rhs.revenue;
        return *this;
      }
    };

    std::istream &read(std::istream &is, Sales_data &sales) {
      double price = 0.0;
      if (is >> sales.bookNo >> sales.units_sold >> price) {
        sales.revenue = sales.units_sold * price;
      } else {
        sales = Sales_data();
      }

      return is;
    }

    std::ostream &print(std::ostream &os, Sales_data &sales) {
      os << sales.bookNo << " " << sales.units_sold << " " << sales.revenue
         << std::endl;
      return os;
    }

    Sales_data add(const Sales_data &lhs, const Sales_data &rhs) {
      Sales_data sum = lhs;
      sum.combine(rhs);
      return sum;
    }

    int main() {
      Sales_data total;
      std::string fileName("8.6.txt");
      std::ifstream ifile(fileName, std::fstream::in);
      if (!ifile.is_open()) {
        throw std::runtime_error("无法打开文件: " + fileName);
      }

      if (read(ifile, total)) {
        Sales_data trans;
        while (read(ifile, trans)) {
          if (total.isbn() == trans.bookNo)
            total.combine(trans);
          else {
            print(std::cout, total);
            total = trans;
          }
        }
        print(std::cout, total);
      } else {
        std::cerr << "No data?!" << std::endl;
        return -1;
      }
      return 0;
    }
    ````

    ````text
    AAA 2 2
    AAA 2 1
    BBB 1 2
    ````


#### 8.2.2 文件模式 {#8-dot-2-dot-2-文件模式}

每个流都有一个关联的文件模式，用来指出如何使用该文件。

| 文件模式 | 解释           |
|------|--------------|
| in     | 以读方式打开   |
| out    | 以写方式打开   |
| app    | 每次写操作均定位到文件末尾 |
| ate    | 打开文件后立即定位到文件末尾 |
| trunc  | 截断文件       |
| binary | 以二进制方式进行IO |

-   文件模式有以下限制：
    -   out 模式只适用于 ofstream 或 fstream
    -   in 模式只适用于于 ifstream 或 fstream
    -   只有设定了 out 才能设定 trunc 模式
    -   没有设定 trunc 的情况下才能设定 app 模式。
    -   默认情况下以 out 模式打开会被截断，为了保留以 out 模式打开的文件内容，必须同时指定 app 模式、或者指定 in 模式，即打开文件同时进行读写操作。
        -   out + app
        -   out + in

    -   ate 和 binary 模式可以用于任何类型的文件流对象，且可以与其它任何文件模式组合使用。

<!--list-separator-->

-  以 out 模式打开文件会丢弃已有数据

    ````cpp
    #include <fstream>
    #include <iostream>

    using namespace std;
    int main(){
      // 下面三条语句会导致文件 file1 被截断（清空）
      ofstream out("file1");
      ofstream out2("file1",ofstream::out);
      ofstream out3("file1",ofstream::out | ofstream::trunc);

      // 为了保留文件内容，必须显式指定 app 模式 或 in 模式
      ofstream app("file2", ofstream::app);
      ofstream app2("file2", ofstream::out | ofstream::app);
      ofstream app3("file2", ofstream::out | ofstream::in);
      return 0;
    }
    ````

<!--list-separator-->

-  每次调用 open 时都会确定文件模式

    对于一个给定的流，每次调用 open 函数时都可以改变文件模式。

    ````cpp
    ofstream out;
    out.open("file");   // 模式隐含设置为 out： 输出和截断。
    out.close();        // 关闭 out，以便我们将其用于其他文件。
    out.open("file",ofstream::app);   // 模式为输出和追加
    out.close();
    ````

<!--list-separator-->

-  8.2.2 节练习

    8.7

    ````cpp
    #include <fstream>
    #include <iostream>
    #include <istream>
    #include <string>

    struct Sales_data {
      std::string bookNo;
      unsigned units_sold = 0;
      double revenue = 0.0;

      // 成员函数
      std::string isbn() { return this->bookNo; }
      Sales_data &combine(const Sales_data &rhs) {
        units_sold += rhs.units_sold;
        revenue += rhs.revenue;
        return *this;
      }
    };

    std::istream &read(std::istream &is, Sales_data &sales) {
      double price = 0.0;
      if (is >> sales.bookNo >> sales.units_sold >> price) {
        sales.revenue = sales.units_sold * price;
      } else {
        sales = Sales_data();
      }

      return is;
    }

    std::ostream &print(std::ostream &os, Sales_data &sales) {
      os << sales.bookNo << " " << sales.units_sold << " " << sales.revenue
         << std::endl;
      return os;
    }

    Sales_data add(const Sales_data &lhs, const Sales_data &rhs) {
      Sales_data sum = lhs;
      sum.combine(rhs);
      return sum;
    }

    int main() {
      Sales_data total;
      std::string infile("8.6.txt");
      std::string outfile("8.6.output.txt");
      std::ifstream ifile(infile, std::fstream::in);
      std::ofstream ofile(outfile,std::fstream::out | std::fstream::app);
      if (!ifile.is_open()) {
        throw std::runtime_error("无法打开文件: " + infile);
      }

      if (!ofile.is_open()) {
        throw std::runtime_error("无法打开文件: " + outfile);
      }


      if (read(ifile, total)) {
        Sales_data trans;
        while (read(ifile, trans)) {
          if (total.isbn() == trans.bookNo)
            total.combine(trans);
          else {
            print(ofile, total);
            total = trans;
          }
        }
        print(ofile, total);
      } else {
        std::cerr << "No data?!" << std::endl;
        return -1;
      }
      return 0;
    }
    ````

    8.8
    已验证。


### 8.3 string 流 {#8-dot-3-string-流}

-   sstream 头文件定义了三个类型来支持内存IO，这些类型可以将 string 作为一个流来使用。sstream 继承自 iostream，支持 iostream 中定义的操作。例如 &lt;&lt; 和 &gt;&gt;。
    -   istringstream 从 string 读取数据
    -   ostringstream 向 string 写入数据
    -   stringstream 即可以读也可以写

-   stringstream 特有的操作:

    | 项目             | 说明                                                       |
    |----------------|----------------------------------------------------------|
    | sstream strm;    | strm 是一个未绑定的 stringstream 对象。sstream 是头文件 sstream 中定义的一个类型 |
    | sstream strm(s); | strm 是一个 sstream 对象，保存 string s 的一个拷贝。此构造函数是 explicit |
    | strm.str();      | 返回 strm 所保存的 string 的拷贝                           |
    | strm.str(s);     | 将 string s 拷贝到 strm 中。返回 void                      |

[sstream继承链](https://yuanbao.tencent.com/bot/app/share/chat/DoxqHSnts9nJ)


#### 8.3.1 使用 istringstream {#8-dot-3-dot-1-使用-istringstream}

使用场景：当我们需要对整行文本进行处理，其中一些工作是处理行内的单个单词时，通常可以使用 istringstream。

假定有一个文件，列出了一些人和他们的电话号码，有些人只有一个电话号，有些人有多个电话号，如下所示：

````latex
Jason 123456 125874
Jack 145225 85545 1211212 1212121
Tim 44545
````

我们可以定义一个简单的类来描述输入数据：

<a id="code-snippet--inline-PersonInfo"></a>
````cpp
//  成员默认公有
struct PersonInfo {
  std::string name;
  std::vector<std::string> phones;
};
````

我们需要读取数据文件,并创建一个 PersonInfo 的 vector。vector 中每个元素对应文件中的一条记录。我们在一个循环中处理输入数据，每个循环需要分别提取人名和若干个电话号码。

````cpp
#include <iostream>
#include <sstream>
#include <string>
#include <vector>
#include <fstream>

using namespace std;
//  成员默认公有
struct PersonInfo {
  std::string name;
  std::vector<std::string> phones;
};
int main() {
  string line,word;             // 分别保存输入的一行和单词
  vector<PersonInfo> people;    // 保存来自输入的所有记录

  ifstream infile("personInfo.txt",std::fstream::in);
  if(!infile.is_open()) { throw std::runtime_error("无法打开文件: personInfo.txt"); }
  // 逐行从文件读取数据，直到遇到文件尾，或其他错误
  while(getline(infile, line)){
    PersonInfo info;              // 创建一个保存此记录数据的对象
    istringstream record(line);   // 将每一行和一个流对象绑定
    record >> info.name;          // 读取名字
    while(record >> word){        // 读取电话号码
      info.phones.push_back(word);
    }

    people.push_back(info);
  }

  for(auto &p: people){
    cout << p.name << " ";
    for(auto &x:p.phones){
      cout << x << " ";
    }
    cout << endl;
  }
  return 0;
}
````

````text
Jason 123456 125874
Jack 145225 85545 1211212 1212121
Tim 44545
````


#### 8.3.1 节练习 {#8-dot-3-dot-1-节练习}

<!--list-separator-->

-  练习 8.9

    ````cpp
    #include <iostream>
    #include <istream>
    #include <sstream>

    using namespace std;

    std::istream &read(std::istream &cin) {
      char c;
      while (cin.get(c)) {
        std::cout << c;
      }

      cin.clear();
      return cin;
    }

    int main() {
      istringstream istrm("Hello Jason");
      read(istrm);
      return 0;
    }
    ````

    ````text
    Hello Jason
    ````

<!--list-separator-->

-  练习 8.10

    假设有以下文件 8.10.txt

    ````latex
    hello world ni
    jason Jason
    jack Jack
    ````

    ````cpp
    #include <fstream>
    #include <iostream>
    #include <sstream>
    #include <stdexcept>
    #include <string>
    #include <vector>

    using namespace std;

    int main() {
      ifstream infile("8.10.txt", ifstream::in);
      if (!infile.is_open()) {
        throw runtime_error("文件打开失败：8.10.txt");
      }

      vector<string> vec;
      string line;
      string word;
      while (getline(infile, line)) {
        vec.push_back(line);
      }

      for (auto &x : vec) {
        istringstream istrm(x);
        while (istrm >> word) {
          cout << word << " ";
        }
      }
      return 0;
    }
    ````

<!--list-separator-->

-  练习 8.11

    ````cpp
    #include <iostream>
    #include <sstream>
    #include <string>
    #include <vector>
    #include <fstream>

    using namespace std;
    //  成员默认公有
    struct PersonInfo {
      std::string name;
      std::vector<std::string> phones;
    };
    int main() {
      string line,word;             // 分别保存输入的一行和单词
      vector<PersonInfo> people;    // 保存来自输入的所有记录

      ifstream infile("personInfo.txt",std::fstream::in);
      if(!infile.is_open()) { throw std::runtime_error("无法打开文件: personInfo.txt"); }
      // 逐行从文件读取数据，直到遇到文件尾，或其他错误

      istringstream record;
      while(getline(infile, line)){
        PersonInfo info;              // 创建一个保存此记录数据的对象
        record.str(line);             // 将每一行和一个流对象绑定

        record >> info.name;          // 读取名字
        while(record >> word){        // 读取电话号码
          info.phones.push_back(word);
        }

        people.push_back(info);
        record.clear();                   // 清除错误状态
      }

      for(auto &p: people){
        cout << p.name << " ";
        for(auto &x:p.phones){
          cout << x << " ";
        }
        cout << endl;
      }
      return 0;
    }
    ````


#### 8.3.2 使用 ostringstream {#8-dot-3-dot-2-使用-ostringstream}

````cpp
#include <sstream>
#include <iostream>

using namespace std;

int main(){
  ostringstream ostrm;
  ostrm << "Hello World";
  ostrm <<" Jason";
  cout << ostrm.str() << endl;
  return 0;
}
````


## 第九章 顺序容器 {#第九章-顺序容器}


### 9.1 顺序容器概述 {#9-dot-1-顺序容器概述}

不同的容器在以下两方面的性能有折中：

-   向容器添加或从容器中删除元素的代价。

-   非顺序访问容器中元素的代价。

| 容器         | 说明                                       |
|------------|------------------------------------------|
| vector       | 可变大小数组。支持快速随机访问。在尾部之外插入或删除元素可能很慢。 |
| deque        | 双端队列。支持快速随机访问。在头部位置插入/删除速度很快 |
| list         | 双向链表。只支持双向数据访问。在 list 中任何位置进行插入/删除操作都很快 |
| forward_list | 单向链表。只支持单向顺序访问。在链表的任何位置进行插入/删除操作速度都很快 |
| array        | 固定大小数组。支持快速随机访问。不能添加或删除元素 |
| string       | 与 vector 相似的容器，但专门用于保存字符。随机访问快。在尾部插入/删除速度快 |

容器选择的基本原则：

-   有很多小元素且对空间占用敏感，则不应该使用 list 或 forward_list
-   程序需要随机访问元素，应该使用 vector 或 deque
-   程序需要在头尾插入或删除元素，但不会在中间位置插入或删除操作，则使用 deque
-   如果程序在读取时才需要在中间插入元素，然后需要随机访问，则：
    -   输入阶段使用 list。
    -   输入完成后将内容拷贝到 vector 中。

9.1 节练习

-   练习 9.1
    (a) 使用 list。
    (b) 使用 deque。
    (c) 使用 deque。


### 9.2 容器库概览 {#9-dot-2-容器库概览}

容器中几乎可以保存任何类型，但某些容器的构造函数要求保存的类型有默认构造函数。例如：

````cpp
// 假定 noDefault 是一个没有默认构造函数的类型,它的一个构造函数接受一个int和一个std::string
noDefault init(42, "initial");     // 创建一个初始化对象
vector<noDefault> v1(10, init);    // 正确：提供了元素初始化器。
// 方法2：直接使用构造函数参数（C++11及以上）
vector<noDefault> v2(3, noDefault(100, "temp"));

vector<noDefault> v2(10);          // 错误：必须提供了一个元素初始化器
````


#### 容器操作 {#容器操作}

-   类型别名

    | 类型别名        | 说明                                    |
    |-------------|---------------------------------------|
    | iterator        | 此容器类型的迭代器类型                  |
    | const_iterator  | 可以读取元素，但不能修改元素的迭代器类型 |
    | size_type       | 无符号整数类型。                        |
    | difference_type | 带符号整数类型，足够保存两个迭代器之间的距离 |
    | value_type      | 元素类型                                |
    | reference       | 元素的左值类型；与 value_type&amp; 含义相同 |
    | const_reference | 元素的 const 左值类型（即 const value_type&amp;） |

<!--listend-->

-   构造函数

    | 构造函数      | 说明                                  |
    |-----------|-------------------------------------|
    | C c;          | 默认构造函数，构造空容器              |
    | C c1(c2);     | 构造C2的拷贝C1                        |
    | C c(b,e);     | 构造 c，将迭代器b和e指定的范围内的元素拷贝到c (array 不支持) |
    | C c{a,b,c...} | 列表初始化 c                          |

    [C++ 列表初始化]({{< relref "cpp_列表初始化.md" >}})

<!--listend-->

-   赋值与 swap

    | 表达式       | 说明                        |
    |-----------|---------------------------|
    | c1=c2        | 将C1中的元素替换为C2中的元素 |
    | c1={a,b,c..} | 将c1中的元素替换为列表中的元素（不适用于array） |
    | a.swap(b)    | 交换 a 和 b 的元素          |
    | swap(a,b)    | 与 a.swap(b) 等价           |

<!--listend-->

-   大小

    | 表达式       | 说明                        |
    |-----------|---------------------------|
    | c.size()     | c中元素的数目（不支持 forward_list） |
    | c.max_size() | c可保存的最大元素数目       |
    | c.empty()    | 若c中存储了元素，返回 false,否则返回 true |

<!--listend-->

-   添加/删除元素（不适用于array）

    | 表达式           | 说明                |
    |---------------|-------------------|
    | c.insert(args)   | 将 args 中的元素拷贝进c |
    | c.emplace(inits) | 使用 inits 构造c中的一个元素 |
    | c.erase(args)    | 删除 args 指定的元素 |
    | c.clear()        | 删除 c 中的所有元素，返回 void |

<!--listend-->

-   关系运算符

    | 运算符                   | 说明             |
    |-----------------------|----------------|
    | `=, !`                   | 所有容器都支持相等（不等）运算符 |
    | &lt;, &lt;=, &gt;, &gt;= | 关系运算符（无序关联容器不支持） |

<!--listend-->

-   迭代器

    | 表达式               | 说明                    |
    |-------------------|-----------------------|
    | c.begin(), c.end()   | 返回指向 c 的首元素和尾元素之后位置的迭代器 |
    | c.cbegin(), c.cend() | 返回 const_iterator     |

<!--listend-->

-   反向容器的额外成员

    | 表达式                 | 说明                      |
    |---------------------|-------------------------|
    | reverse_iterator       | 按逆序寻址的迭代器        |
    | const_reverse_iterator | 不能修改元素的逆序迭代器  |
    | c.rbegin(), c.rend()   | 返回指向 c 的尾元素和首元素之前位置的迭代器 |
    | c.crbegin(), c.crend() | 返回 const_reverse_iterator |

9.2 节练习练习 9.2

````cpp
#include <list>
#include <deque>
std::list<std::deque<int>> l;
````


#### 9.2.1 迭代器 {#9-dot-2-dot-1-迭代器}

一个 `迭代器范围` 由一对迭代器表示，这两个迭代器分别是代称为 begin 和 end。begin 表示首元素所在位置，end 表示容器最后一个元素之后的位置。

这种元素范围被称为左闭合区间： [begin, end)

-   `迭代器范围` 的构成要求（程序员维持这个约定）：
    -   它们指向同一个容器中的元素或最后一个元素之后的位置。

    -   我们可以通过反复递增 begin 来到达 end。换句话来说，end 不在 begin 之前。

-   通过迭代器可以获得的信息：
    -   如果 begin 与 end 相等，则容器为空

    -   如果 begin 与 end 不等，则容器至少包含一个元素，且 begin 指向该范围中的第一个元素。

    -   我们可以对 begin 递增若干次，使得 begin  == end

<!--listend-->

-   迭代器的一般使用
    ````cpp
    while(begin != end){
      *begin == val;    // 正确：范围非空，因此 begin 指向一个元素
      ++begin;          // 移动迭代器，获取下一个元素
     }
    ````


#### 9.2.1 节练习 {#9-dot-2-dot-1-节练习}

-   练习 9.3
    参见本节内容。

<!--listend-->

-   练习 9.4
    ````cpp
    #include <iostream>
    #include <vector>

    using namespace std;

    bool find_value(vector<int>::iterator begin, vector<int>::iterator end, int value){
      while(begin!=end){
        if(*begin == value)
          return true;

        ++begin;
      }
      return false;
    }
    int main(){
      vector<int> vec{1,2,3,4,5,6};
      cout << (find_value(vec.begin(), vec.end(), 2)?"true":"false") << endl;
      return 0;
    }
    ````

<!--listend-->

-   练习 9.5
    ````cpp
    #include <iostream>
    #include <vector>

    using namespace std;

    vector<int>::iterator find_value(vector<int>::iterator begin, vector<int>::iterator end, int value){
      auto it = end;
      while(begin!=end){
        if(*begin == value)
          return begin;

        ++begin;
      }
      return it;
    }
    int main(){
      vector<int> vec{1,2,3,4,5,6};

      auto it = find_value(vec.begin(), vec.end(),5);
      if(it!=vec.end())
        cout<< *it << endl;
      return 0;
    }

    ````

<!--listend-->

-   练习 9.6
    当容器为空时，两个迭代器都指向尾元素之后的位置。


#### 9.2.2 容器类型成员 {#9-dot-2-dot-2-容器类型成员}

每个容器都定义了多个类型，例如：size_type, iterator 和 const_iterator。

除此之外，大多数容器（std::forward_list 除外）还提供反向迭代器，对一个反向迭代器执行++操作，会得到上一个元素。

除了上述两种，就剩下类型类型别名了，通过类型别名，我们可以在不了解容器中元素类型的情况下使用它。

-   value_type 是元素类型别名

-   reference 和 const_reference 是元素的引用别名

如何使用类型别名：

````cpp
// iter 是通过 list<string>定义的一个迭代器类型：
list<string>::iterator iter;

// count 是通过 vector<int>定义的一个 difference_type 类型
vector<int>::difference_type count;
````

9.2.2 节练习

-   练习 9.7
    迭代器类型，std::vector&lt;int&gt;::const_iterator

<!--listend-->

-   练习 9.8
    读取：std::list&lt;std::string&gt;::const_iterator
    写入：std::list&lt;std::string&gt;::iterator


#### 9.2.3 begin 和 end 成员 {#9-dot-2-dot-3-begin-和-end-成员}

begin 和 end 有多个版本：带 r 开头的版本返回反向迭代器，带 c 开头的版本返回 const 迭代器。 以 cr 开头的版本返回 const 反向迭代器。

不以c开头的函数都是重载函数，每一个类别有两个实现，例如 begin 有两个版本，其它 rbegin、end 和 rend 也是一样的都有两个版本。这两个版本一个是 const 成员，返回容器的 const_iterator 类型，另一个是非常量成员，返回容器的 iterator 类型。

````cpp
#include <list>
#include <string>

using namespace std;

int main(){
  list<string> vec{"Hello","World","Jason"};

  // 显式指定类型
  list<string>::iterator it1 = vec.begin();
  list<string>::const_iterator it2 = vec.begin();
  //list<string>::iterator it1 = vec.cbegin(); // 类型不批评，错误
  auto it3 = vec.cbegin();
  return 0;
}
````

9.2.3 节练习

-   练习 9.9
    begin 没有const 修饰，cbegin 成员函数有 const 修饰
    ````cpp
    iterator begin() _GLIBCXX_NOEXCEPT {
      return iterator(this->_M_impl._M_node._M_next);
    }

    const_iterator cbegin() const noexcept {
      return const_iterator(this->_M_impl._M_node._M_next);
    }
    ````


#### 9.2.4 容器定义和初始化 {#9-dot-2-dot-4-容器定义和初始化}

除 array 之外，其它容器的默认构造函数都会创建一个指定类型的空容器，且都可以接受指定容器大小和元素初始值的参数。

| 容器构造方式     | 说明                                                               |
|------------|------------------------------------------------------------------|
| C c;             | 默认构造函数。如果 C 是一个 array，则 C 中元素按照默认方式初始化；否则 C 为空 |
| C c1(c2)         | c1初始化为 c2 的拷贝，两者必须是相同类型，对于 array 类型，两者还必须具有相同的大小 |
| C c{a,b,c,d};    | c 初始化为初始化列表中元素的拷贝。对于 array，列表中元素数目必须小于或等于 array 的大小，任何遗漏的元素都进行值初始化 |
| C c = {a,b,c,d}; | 同上                                                               |
| C c(b,e);        | c 初始化为迭代器 b 和 e 指定范围中元素的拷贝，array 不适用。       |
| C seq(n);        | seq 包含 n 个元素，这些元素都进行了值初始化；构造函数是 explicit。只适用于顺序容器，且不能是 string。 |
| C seq(n,t);      | seq 包含 n 个初始化为值 t 的元素。只适用于顺序容器。               |

<!--list-separator-->

-  将一个容器初始化为另一个容器的拷贝

    将一个容器初始化为另一个容器的拷贝的方法有两种：可以直接拷贝整个容器，或者（array 除外）拷贝由一个迭代器对指定的元素范围。

    当传递迭代器参数来拷贝一个范围时，不需要容器类型是相同的，容器中的元素类型也可以不同，只要能将要拷贝的元素转换为要初始化的容器的元素类型即可。

    ````cpp
    list<string> authors = {"Milton", "Shakespeare", "Austen"};
    vector<const char*> articles = {"a","an","the"};

    list<string> list2(authors);      // 正确：类型匹配
    deque<string> authList(authors);  // 错误：容器类型不匹配
    vector<string> words(articles);   // 错误：容器类型必须匹配

    former_list<string> words (articles.begin(), articles.end());  // 正确：可以将 const char* 元素转换为 string

    // 拷贝元素，直到 it 指向的位置前一个元素。
    deque<string> authList (authors.begin(), it);
    ````

<!--list-separator-->

-  列表初始化

    对于除 array 之外的容器类型，初始化列表还隐含地指定了容器的大小：容器将包含与初始值一样多的元素。

    ````cpp
    #include <iostream>
    #include <string>
    #include <vector>
    #include <list>

    using namespace std;

    int main(){
      list<string> authors = {"Milton", "Shakespeare", "Austen"};
      vector<const char*> articles = {"a", "an", "the"};
      vector<const char*> articles2{"a", "an", "the"};    // 与上一行代码等价

      vector<string> vec;
      vec.push_back("H");
      vec.push_back("H");
      vec.push_back("H");
      vec.push_back("H");
      vec.push_back("H");

      cout << authors.size() <<" " << endl;
      cout << articles.size() << " " <<articles.capacity() << endl;
      cout << articles2.size() << " " << articles2.capacity() <<endl;

      cout << vec.size() << " " << vec.capacity() <<endl;
      return 0;
    }
    ````

    ````text
    3
    3 3
    3 3
    5 8
    ````

<!--list-separator-->

-  标准库 array 具有固定大小

    和内置数组一样，标准库 array 的大小也是类型的一部分。

    ````cpp
    array<int, 42> num1;
    array<string, 10> arr;
    ````

    为了使用 array 类型，我们必须同时指定元素类型和大小：

    ````cpp
    array<int, 10>::size_type i;   // array 类型包括元素类型和大小
    array<int>::size_type j;       // 错误：array<int>不是一个类型
    ````

    如果元素是类类型，那么该类必须有一个默认构造函数，以使值初始化能够进行。

    ````cpp
    int digs[10] = {0, 1,2,3,4,5,6,7,8,9};
    //int cpy[10] = digs;                    // 错误：内置数组不支持拷贝和赋值
    std::array<int, 10> digits = {0,1,2,3,4,5,6,7,8,9};
    std::array<int, 10> copy = digits;   // 正确：只要数组类型匹配即可合法,除此之外，两者元素数量也必须一样。
    ````

    -   9.2.4 节练习
        -   练习9.11
            ````cpp
            #include <iostream>
            #include <vector>
            #include <string>
            using namespace std;

            int main(){
              vector<string> vec;                                     // 空容器
              vector<string> vec2{"Hello", "World"};                  // 含有两个元素 {"Hello", "World"}
              vector<string> vec3 = vec2;                             // 含有两个元素 {"Hello", "World"}
              vector<string> vec4(vec2.begin(), vec2.end());          // 含有两个元素 {"Hello", "World"}
              vector<string> vec5(100);                               // 含有100个空string
              vector<string> vec6(100,string("Hello"));               // 含有100个"Hello"

              cout << vec.size() << " " << vec.capacity() << endl;
              cout << vec2.size() << " " << vec2.capacity() << endl;
              cout << vec3.size() << " " << vec3.capacity() << endl;
              cout << vec4.size() << " " << vec4.capacity() << endl;
              cout << vec5.size() << " " << vec5.capacity() << endl;
              cout << vec6.size() << " " << vec6.capacity() << endl;
              return 0;
            }
            ````

            ````text
            0 0
            2 2
            2 2
            2 2
            100 100
            100 100
            ````

        -   练习9.12
            使用构造函数：容器类型和元素类型都必须一致使用迭代器范围：容器类型一致，元素类型可以不同，只需能相互转换即可。

    <!--listend-->

    -   练习 9.13
        ````cpp
        #include <iostream>
        #include <list>
        #include <vector>

        using namespace std;

        int main(){
          list<int> num_link = {1,2,3,4,5,6};
          vector<int> ivec = {1,2,3,4,5,6,7,8,9};
          vector<double> dvec(num_link.begin(), num_link.end());
          vector<double> dvec2(ivec.begin(),ivec.end());

          return 0;
        }
        ````


#### 9.2.5 赋值和swap {#9-dot-2-dot-5-赋值和swap}

赋值运算符将其左边容器中的全部元素替换为右边容器中的元素的拷贝。

````cpp
c1 = c2;        // 将 c1 的内容替换为 c2 中元素的拷贝
c1 = {a,b,c};   // 赋值后，c1 大小为3
````

标准库 array 类型也允许赋值，但赋值号左右两边的运算对象必须具有相同的类型：相同的类型、相同的大小

````cpp
array<int, 10> a1 = {0,1,2,3,4,5,6,7,8,9};
array<int, 10> a2 = {0};  // 所有10个元素均为 0
a1 = a2;   // 替换 a1 中的元素
a2 = {0};  // 错误：不能将一个花括号列表赋予数组。
````

<!--list-separator-->

-  容器赋值运算

    | 表达式          | 说明                                                        |
    |--------------|-----------------------------------------------------------|
    | c1=c2           | 将 c1 中的元素替换为 c2 中的拷贝。c1 和 c2 必须具有相同的类型。 |
    | c = {a,b,c...}  | 将 c1 中的元素替换为初始化列表中的拷贝（array 不适用）      |
    | swap(c1,c2)     | 交换 c1 和 c2 中的元素。c1 和 c2 必须具有相同的类型。swap 通常比从 c2 向 c1 拷贝元素快得多 |
    | c1.swap(c2)     | 同上                                                        |
    | seq.assign(b,e) | 将 seq 中的元素替换为迭代器 b 和 e 所表示的范围中的元素。迭代器 b 和 e 不能指向 seq 中的元素 |
    | seq.assign(n,t) | 将 seq 中的元素替换为 n 个值为 t 的元素。                   |

    警告：赋值相关运算符会导致指向左边容器内部的迭代器、引用和指针失效。而 swap 操作不会导致它们的 迭代器、引用和指针失效（array和string除外）

<!--list-separator-->

-  使用 assign（仅顺序容器）

    顺序容器（除了 array ）使用 assign ，我们可以从一个不同但相容的类型赋值，或者从容器的一个子序列赋值。例如：我们可以使用 assign 实现将一个vector 中的一段 char\* 值赋予一个 list 中的 string：

    ````cpp
    list<string> names;
    vector<const char*> oldstyle;
    names = oldstyle;  // 错误：容器类型不匹配

    // 正确：可以将 const char* 转换为 string
    names.assign(oldstyle.cbegin(), oldstyle.cend());
    ````

    assign 第二个版本接受一个整型值（数量）和一个元素值（元素类型）。

    ````cpp
    // 等价于 slist1.clear();
    // 后跟 slist1.insert(slist1.begin(),10,"Hiya!");

    list<string> slist1(1);    // 一个元素，为空string
    slist1.assign(10,"Hiya!"); // 10个元素，每个都是 "Hiya!"
    ````

<!--list-separator-->

-  使用 swap

    swap 交换两个相同类型容器的内容。除了 array 外，swap 不对任何元素进行拷贝、删除或插入操作，因此可以保证在常数时间内完成。

    ````cpp
    #include <iostream>
    #include <string>
    #include <vector>

    using namespace std;

    int main() {
      vector<string> svec1 = {"a","b", "c", "d"};
      vector<string> svec2 = {"e", "f"};

      auto it1 = svec1.begin();
      auto it2 = svec2.begin();
      swap(svec1, svec2);

      cout << "it1 " << *it1 << endl;
      cout << "it2 " << *it2 << endl;


      return 0;
    }
    ````

    -   swap 操作注意事项：
        -   原本的迭代器、引用、指针都保持有效。
        -   end() 迭代器也会跟随交换，需要重新获取
        -   容量(capacity)也会交换，内存分配策略随之交换
        -   这是一种高效的O(1)操作，只交换内部指针，不复制元素

    <!--listend-->

    -   9.2.5 节练习
        -   练习 9.14
            ````cpp
            #include <iostream>
            #include <list>
            #include <string>
            #include <vector>
            using namespace std;

            int main(){
              list<const char*> list_str = {"Hello","World"};
              vector<string> vec_str(list_str.cbegin(), list_str.cend());
              return 0;
            }
            ````


#### 9.2.6 容器大小操作 {#9-dot-2-dot-6-容器大小操作}

除了一个例外，每个容器都有三个与大小相关的操作：

-   成员函数 size：返回容器中元素的数目

-   成员函数 empty：当 size 为 0 时返回 true，否则返回 false

-   max_size 返回一个大于或等于该类型容器能容纳的最大元素数的值

-   forward_list 支持 max_size 和 empty，但不支持 size。


#### 9.2.7 关系运算符 {#9-dot-2-dot-7-关系运算符}

每个容器都支持相等运算符（== 和 !=）；除了无序关联容器外的所有容器都支持关系运算符（&gt;、&gt;=、&lt;、&lt;=）。关系运算符要求左边两边的运算对象必须是相同的类型。

-   两个容器相等的条件：两个容器具有相同的大小且所有元素都两两对应相等。

-   如果两个容器大小不同，如果较小容器中每个元素都等于较大容器中的对应元素，则较小容器小于较大容器：
    ````cpp
    vector<int> vec1 = {1,2,3,4,5};
    vector<int> vec2 = {1,2,3};

    // vec2 < vec1
    ````

<!--listend-->

-   如果两个容器都不是另一个容器的前缀子序列，则比较的结果取决于第一个不相等的元素的比较结果。


#### 容器的关系运算符使用元素的关系运算符完成比较 {#容器的关系运算符使用元素的关系运算符完成比较}

只有当其元素类型也定义了相应的比较运算符时，我们才可以使用关系运算符来比较两个容器。

-   容器的相等运算符实际使用元素的 == 运算符实现。
-   而其他关系运算符是使用元素的  &lt; 运算符。

<!--listend-->

-   9.2.7 节练习
    -   练习 9.15
        ````cpp
        #include <iostream>
        #include <vector>

        using namespace std;

        int main(){
          vector<int> vec1 = {1,2,4,5,3};
          vector<int> vec2 = {1,2,5,7,3};

          cout << "vec1 < vec2: " << (vec1<vec2?"true":"false") << endl;
          return 0;
        }
        ````

<!--listend-->

-   练习 9.16
    ````cpp
    #include <iostream>
    #include <list>
    #include <vector>

    using namespace std;

    bool compare(list<int> lst, vector<int> vec) {
      auto it_lst = lst.cbegin();
      auto it_vec = vec.cbegin();

      while (it_lst != lst.cend() && it_vec != vec.cend()) {
        if (*it_lst != *it_vec) {
          if(*it_lst > *it_vec)
            return true;
          else
            return false;
        }
        ++it_lst;
        ++it_vec;
      }

      if(it_lst != lst.cend()){
        return true;
      }else{
        return false;
      }
    }

    int main() {
      vector<int> vec1 = {1, 2, 4, 5, 3};
      list<int> vec2 = {1, 2, 4,5,3,5};

      cout << "vec2 > vec1: " << (compare(vec2,vec1)?"true":"false") << endl;
      return 0;
    }
    ````

<!--listend-->

-   练习 9.17
    容器元素必须定义比较运算符


### 9.3 顺序容器操作 {#9-dot-3-顺序容器操作}

顺序容器和关联容器的不同之处在于：元素如何存储、访问、添加以及删除。


#### 9.3.1 向顺序容器添加元素 {#9-dot-3-dot-1-向顺序容器添加元素}

除了 array 外，所有标准容器都支持动态添加或删除元素来改变容器的大小。

<!--list-separator-->

-  向顺序容器添加元素的操作

    {{< figure src="/images/向顺序容器添加元素.png" >}}
