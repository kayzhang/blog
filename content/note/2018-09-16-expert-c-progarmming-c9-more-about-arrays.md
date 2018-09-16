---
title: 《Expert C Programming》 第 9 章：再论数组
subtitle: 全文摘录及小部分修改与备注
date: '2018-09-16'
slug: expert-c-progarmming-c4-more-about-arrays
categories:
  - C
  - Expert C Programming
  - Arrays
  - Pointers
tags:
  - arrays
  - pointers
toc: true
---

# 再论数组

## 9.1 什么时候数组与指针相同
第 4 章着重强调了数组和指针并不一致的绝大多数情形。本章的开始部分就是讲述可以把它们看做是相同的情形。在实际应用中，数组和指针可以互换的情形要比两者不可互换的情形更为常见。让我们分别考虑“声明”和“使用”（使用它们传统的直含义）这两种情况。

声明本身还可以分成 3 种情况：

* 外部数组（external array）的声明；
* 数组的定义（记住，定义是声明的一种特殊情况，它分配内存空间，并可能提供一个初始值）；
* 函数参数的声明。

所有作为函数参数的数组名总是可以通过编译器转换为指针。在其他所有情况下（最有趣的情况就是“在一个文件中定义为数组，在另一个文件中声明为指针”，第 4 章已有所描述），数组的声明得到的是一个数组，指针的声明得到的是一个指针，两者不能混淆。但在使用数组（在语句或表达式中引用）时，数组总是可以写成指针的形式，两者可以互换。下图对这些情形作了总结。

![数组与指针](https://i.gyazo.com/810a2a15ba8d11d9197a4be96e2d4477.png)

然而，数组和指针在编译器处理时是不同的，在运行时的表示形式也是不一样的，并可能产生不同的代码。**对编译器而言，一个数组就是一个地址，一个指针就是一个地址的地址**，你应该根据情况做出选择。

## 9.2 为什么会发生混淆
为什么人们会错误地认为数组和指针是可以完全互换地呢？这是因为他们阅读了标准的参考文献！

*The C Programming Language*，第二版：

> As formal parameters in a function definition
> 
> char s[];  
> and  
> char *s;
> 
> are equivalent; ...

人们在阅读后一句话时，很容易忘掉前面还有一句“作为函数定义的形式参数时”，尤其是整句话的重点在于“数组小标表达式总是可以改写成带偏移量的指针表达式”。

在作为函数的参数时，数组和指针是完全可以互换的，如下所示：

```C
char my_array[10];
char *my_ptr;
...
i = strlen(my_array);
j = strlen(my_ptr);
```

还有许多类似下面的语句：

```C
printf("%s %s", my_ptr, my_array);
```

它清楚地展示了数组和指针的可互换性。人们很容易忽视这只是发生在一种特定的上下文环境中，也就是它们作为一个函数调用的参数使用。更遭的是，你可以如下编写：

```C
char a[10];
...
printf("array at location %x holds string %s", a, a);
```
在同一条语句中，既把数组名作为一个地址（指针），又把它作为一个字符数组。这条语句之所以可行是因为 `printf` 是一个函数，所以数组实际上是作为指针来传递的。我们也习惯了在 `main` 函数的参数中看到 `char **argv` 或 `char *argv[]` 这样的形式，它们也是可以互换的。同样，这个之所以成立是因为 `agrv` 是一个函数的参数，但它仍然诱使程序员错误地总结出“C语言在地址运算方法上是一致和规则的”。若在脑子里已经存在这样一个概念，再加上平时常常可以见到数组下标表达式被写成指针的形式，久而久之，便很容易把数组和指针混淆。

下边这个表格非常重要！！！

> **软件信条**
> 
> 什么时候数组和指针是相同的：
> 
> **规则 1.** **表达式**中的数组名（用于区别声明中的数组名）被编译器当作一个指向该数组第一个元素的指针（具体释义见 ANSI C 标准第 6.2.2.1 节）。
> 
> **规则 2.** 数组的下标引用总是与指针的偏移量引用相同（具体释义见 ANSI C 标准第 6.3.2.1 节）。
> 
> **规则 3.** 在函数参数的声明中，数组名被编译器当作指向该数组第一个元素的指针（具体释义见 ANSI C 标准第 6.7.1 节）。

注：规则 1 存在几个极为少见的例外，就是把数组作为一个整体来考虑。在下列情况下，对数组的引用不能用指向该数组第一个元素的指针来代替：

> * 数组作为 `sizeof` 的操作数 -- 显然此时需要的是整个数组的大小，而不是指针所指向的第一个元素的大小；
* 使用 `&` 操作符取数组的地址；
* 数组是一个另一个数组的字符串（或宽字符串）常量（string literal）初始值。  
如：`char str[] = "abc";`，其中 string literal `"abc"`的类型为 not modifiable specified character array type，但是并未转换为指向第一个元素的指针，而是作为一个数组整体用于初始化数组 `str`。

具体的 C 参考文档中是这么描述规则 1 的：

> Array to pointer conversion
> 
> Any lvalue expression of array type, when used in any context other than
> 
> * as the operand of the address-of operator
* as the operand of sizeof
* as the string literal used for array initialization
* as the operand of _Alignof
(since C11)
> 
> undergoes an implicit conversion to the pointer to its first element. The result is not an lvalue.

### 9.2.1 规则 1：“表达式中的数组名”就是指针
上面的规则 1 和规则 2 合在一起理解，就是对数组下标的引用总是可以写成“一个指向数组的起始地址的指针加上偏移量”。例如，假如我们声明：

```C
int a[10], *p, i = 2;
```

就可以通过以下任何一种方式来访问 `a[i]`：

```C
// 方式一
p = a;
p[i];

// 方式二
p = a;
*(p + i);

// 方式三
p = a + i;
*p
```

事实上，可以采用的方法更多。对数组的引用如 `a[i]` 在编译时总是被编译器改写成 `*(a + i)` 的形式。