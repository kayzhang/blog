---
title: 《Expert C Programming》 第 4 章：数组和指针
subtitle: 全文摘录及小部分修改与备注
date: '2018-09-16'
slug: expert-c-progarmming-c4-arrays-and-pointers
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

# 第 4 章 令人震惊的事实：数组和指针并不相同

## 4.1 数组并非指针
C 编程新手最常听到的说法之一就是“数组和指针是相同的”。不幸的是，这是一种非常危险的说法，并不完全正确。ANSI C 标准 6.5.4.2 节建议：

> *Note the distinction between the declarations:*
> 
> `extern int *x;`  
> `extern int y[];`
> 
> *The first declares `x` to be a pointer to `int`; the second declares `y` to be an array of `int` of unspecified size (an incomplete type), the storage for which is defined elsewhere.*

## 4.2 我的代码为什么无法运行
下边的代码是无法运行的：

文件 1：

    int mango[100];

文件 2：

    extern int *mango;
    ...
    /* 一些引用 mango[i] 的代码 */
    
这里文件 1 定义了数组 `mango`，但文件 2 声明它为指针。这相当于把整数和浮点数混为一谈，类型不匹配。但是为什么人们会认为指针和数组始终是可以互换的呢？答案是对数组的引用总是可以写成对指针的引用，而且确实存在一种指针和数组的定义完全相同的上下文环境。不幸的是，由于这种上下文环境是数组的一种非常常见的用法，因此人们便自然而然地归纳并假定所有的情况下数组和指针都是相同的，于是便产生了诸如上述的“定义为数组但外部声明为指针”的错误。

## 4.3 什么是声明，什么是定义
为了搞清楚这个问题，我们首先重申**声明**和**定义**的概念。在 C 语言中，对象必须有且只有一个定义，但它可以有多个 `extern` 声明。注意，这里所说的对象跟 C++ 中对象并无关系，这里的对象只是跟链接器有关的“东西”，比如函数或者变量。

* 定义：只能出现在一个地方，确定对象的类型并分配内存，用于创建新的对象。例如：`int my_array[100];`
* 声明：可以多次出现，描述对象的类型，用于指代其他地方定义的对象（如在其他文件里）。例如：`extern int my_array[]；`

> **区分定义和声明**
> 
> 只要记住下面的内容即可分清定义和声明 ：
> 
> 声明相当于普通的声明：它所描述的并非其自身，而是描述在其他地方创建的对象。
> 
> 定义相当于特殊的声明：它为对象分配内存。

`extern` 对象声明告诉编译器对象的类型和名字，对象的内存分配则是在别处进行的。由于并未在声明中为数组分配内存，所以不需要提供关于数组长度的信息。对于多维数组，需要提供除最左边一维之外其他维的长度——用来为编译器提供足够的信息产生相应的索引代码。

### 4.3.1 数组和指针是如何访问的
本节讲述对数组的引用和对指针的引用有何不同之处。

#### l-value 和 r-value
在赋值表达式中，出现在赋值符左边的符号通常被叫做 l-value，出现在赋值符右边的符号通常被叫做 r-value。其实，编译器为每个变量分配一个地址（或 l-value），这个地址在编译时可知，而且该变量在运行时一直保存于这个地址。相反，存储于变量中的值（它的 r-value）只有在运行时才可知。如果需要用到变量中存储的值，编译器就发出指令从指定地址读入变量值并将它存于寄存器中。

#### 地址（l-value）和地址的内容（r-value）之间的区别
注意，“地址 `y`”和“地址 `y` 的内容”是不同的概念。这是一个相当微妙的地方，因为在大多数编程语言中我们用同一个符号来表示这两样东西，由编译器根据上下文环境判断它的具体含义。下边以一个简单的赋值为例，进行解释。

| x = y | x = y |
| :---: | :----: |
| 在这个上下文环境里，符号 `x` 的含义是 `x` 所代表的地址。 | 在这个上下文环境里，符号 `y` 的含义是 `y` 所代表的地址的内容。 |
| 这被称为 l-value | 这被称为 r-value |
| l-value 在编译时可知，表示存储结果的地方。 | r-value 直到运行时才知。如无特别声明，其表示 “`y` 的内容”。|
 
> C 语言引入了 **“Modifiable l-value”** 这一术语，它表示允许出现在赋值语句左边的 l-value。这个奇怪的术语是为了特殊处理数组名：数组名也用于确定对象在内存中的位置，也是 l-value，但它不能作为赋值的对象。**因此，数组名是个 l-value，但不是 modifiable l-value。标准规定赋值符必须用 modifiable l-value 作为它左边的一侧的操作数，即只能给可修改的东西赋值。**

#### 指针和数组的访问方式
1）对数组的访问方式

上边提到的一个关键信息是：每个符号的地址是在编译的时候就可知的。所以，如果编译器需要对一个地址（可能还需要加上偏移量）来执行某种操作，它就可以直接进行操作，并不需要增加指令首先取得具体的地址。

这就是为什么 `extern char a[]` 与 `extern char a[100]` 等价的原因。这两个声明都提示 **`a` 是一个数组**，也就是一个内存地址，数组内的字符可以从这个地址找到。编译器并不需要知道数组总共有多长，因为它只产生偏离起始地址的偏移地址。从数组提取一个字符，只要简单地从符号表显示的 `a` 的地址加上下标，需要的字符就位于这个地址中。

> `char a[9] = "abcdefgh";` ... ... ... ... `c = a[i];`
> 
> 编译器符号表具有一个符号 `a`，其地址为 9980
> 
> 运行时步骤 1：取 i 的值，将它与 9980 相加；  
> 运行时步骤 2：取地址（9980 + i）的内容。
> 
> ![数组的下表引用](https://i.gyazo.com/f14f94e38689c196a065354f1d1c26f3.png)
> <center>**图A 数组的下标引用**</center>

2）对指针的访问方式

但是，与此相反，对于指针而言，由于我们需要用到其 r-value，因此我们必须首先在运行时取得它的当前值（即 r-value），然后才能对它进行 dereference 操作（作为以后进行查找的步骤之一）。

与数组声明相反，如果声明 `extern char *p`，它将告诉编译器 **`p` 是一个指针**（通常为一个 4 或 8 字节的对象），它指向的对象是一个字符。为了取得这个字符，必须得到地址 `p` 的内容，把它作为字符的地址并从这个地址取得这个字符。指针的访问要灵活得多，但需要增加一次额外的提取操作。

> `char *p;` ... ... ... ... `c = *p;`
> 
> 编译器符号表具有一个符号 `p`，其地址为 4624
> 
> 运行时步骤 1：取地址 4624 的内容，就是 '5081'；  
> 运行时步骤 2：取地址 5081 的内容。
> 
> ![对指针的引用](https://i.gyazo.com/48186df7bb45724a27424efd43e72576.png)
> <center>**图B 对指针的引用**</center>

### 4.3.2 当你“声明为指针，但以数组方式引用”时会发生什么
#### 定义并声明为指针，但以数组方式引用
当一个外部数组的实际定义是一个指针，外部声明也为指针，但却以数组的方式对其引用时，会引起什么问题呢？此时，我们真正想要的（形式上为下标）是对内存的直接引用（如图A，对数组的引用），但是实际发生的却是对指针的不直接的内存引用（如图B，对指针的引用）。发生这种情况的原因是我们告诉编译器（通过声明而非通过定义）我们拥有的是一个指针，如图 C 所示。

> `char *p = "abcdefgh";` ... ... ... ... `c = p[i];`
> 
> 编译器符号表具有一个符号 `p`，其地址为 4624
> 
> 运行时步骤 1：取地址 4624 的内容，即 '5081'；  
> 运行时步骤 2：取得 `i` 的值，并将它与 5081 相加；  
> 运行时步骤 3：取地址 [5081 + i] 的内容。
> 
> ![对指针的下标引用](https://i.gyazo.com/a056178d2e20eecbea8a70db972a447c.png)
> <center>**图C 对指针的下标引用**</center>

#### 数组的下标引用 VS 指针的下标引用
对照图 C 的访问方式：

    char *p = "abcdedfg"; ... ... ... ... p[3]

和图 A 的访问方式：

    char a[] = "abcdefgh"; ... ... ... ... a[3]

在这两种情况下，都可以取得字符 'd'，但两者的途径非常不一样。

当书写了 `extern char *p;`，然后用 `p[3]` 来引用其中的元素时，其实质是图 A 和图 B 访问方式的组合。首先，进行图 B 所示的间接引用。然后，如图 A 所示用下标作为偏移量进行直接访问。

更为正式的说法是，编译器将会：

1. 取得符号表中 `p` 的地址，提取存储于此处的指针。
2. 把下标所表示的偏移量与指针的值相加，产生一个地址。
3. 访问上面这个地址，取得字符。

编译器已通过声明被告知 `p` 是一个指向字符的指针（**相反，数组声明告诉编译器 `a` 是一个字符序列。**）`p[i]` 表示“从 `p` 所指的地址开始，前进 `i` 步，每步都是一个字符（即每个元素的长度为一个字节）”。如果是其他类型的指针，其步长也各不相同。

#### 定义为数组，但声明为指针，并以数组方式引用
回到 4.2 中的错误程序，下面对其进行解释。

既然把 `p` 声明为指针，那么不管 `p` 原先是定义为指针还是数组，都会按照上面所示的三个步骤进行操作。但是，只有当 `p` 原先定义为指针时这个方法才是正确的。考虑一下 `p` 在这里被声明为 `extern char *p;`，而它原先的定义却是 `char p[10];` 这种情形。当用 `p[i]` 这种形式提取这个声明的内容时，实际上在第 1 步“取得符号表中 `p` 的地址，提取存储于此处的指针”中，实际上取得的是数组的第一个字符。但按照上面的方法，编译器却把它当成是一个指针，把 ACSII 字符解释为地址显然是牛头不对马嘴。如果此时程序当掉，你应该额手称庆。否则的话，它可能会污染程序地址空间的内容，并在将来出现莫名其妙的错误。

## 4.4 使声明与定义相匹配
指针的外部声明与数组定义不匹配的问题很容易修正，只要修改声明，使之与定义相匹配即可，如下所示：

文件 1：

    int mango[100];

文件 2：

    extern int mango[];
    ...
    /* 一些引用 mango[i] 的代码 */

注意：`mango` 的定义分配了 100 个 `int` 的空间。而指针定义：

    int *raisin;

则申请一个地址容纳该指针。指针的名字是 `raisin`，它可以指向任何一个 `int` 变量 (或 `int` 型数组)。指针变量 `raisin` 本身始终位于同一个地址，但它的内容在任何时候都可以不相同，指向不同地址的 `int` 变量。这些不同的 `int` 变量可以有不同的值。**`mango` 数组的地址并不能改变，在不同的时候它的内容可以不同，但它总是表示 100 个连续的内存空间。**

## 4.5 数组和指针的其他区别
对比数组和指针的另外一个方法就是对比两者的特点，见下表：

| 指针 | 数组 |
| :---: | :---:|
| 保存数据的地址 | 保存数据 |
| 间接访问数据，首先取得指针的内容，把它作为地址，然后从这个地址提取数据。如果指针有一个下标 [i]，就把指针的内容加 i 作为地址，从中提取数据。 | 直接访问数据，`a[i]` 只是简单地以 `a + i` 为地址取得数据 |
| 通常用于动态数据结构 | 通常用于存储固定数目且数据类型相同的元素 |
| 相关的函数为 `malloc()`，`free()` | 隐式分配和删除 |
| 通常指向匿名数据 | 自身即为数据名 |

#### 数组和指针字符串常量初始化的不同
数组和指针都可以在它们的定义中用字符串常量进行初始化。尽管看上去一样，底层的机制却不相同。

定义指针时，编译器并不为指针所指向的对象分配空间，它只是分配指针本身的空间，除非在定义时同时赋给指针一个字符串常量进行初始化。例如，下面的定义创建了一个字符串常量（为其分配了内存）：

    char *p = "breadfruit";
    // 注：右边的字符串常量（string literal）的类型为 specified character array，
    //（string literals are not modifiable）
    // 在此表达式中编译器自动将其转变为指向数组第一个元素，即字符 `'b'` 的指针。

注意，只有对字符串常量才是如此。不能指望为浮点数之类的常量分配空间，如：

    float *pip = 3.141; /* 错误！无法通过编译。 */

在 ASNI C 中，初始化指针时所创建的字符串常量被定义为只读。如果试图通过指针修改这个字符串的值，程序就会出现未定义的行为。在有些编译器中，字符串常量被存放在只允许读取的文本段中，以防止它被修改。

数组也可以用字符串常量进行初始化：

    char a[] = "gooseberry";
    // 注：在 literal string（类型为 specified character array）作为数组的初始化常量时，
    // 并未进行 Array to pointer conversion，仍为一个数组整体。

与指针相反，由字符串常量初始化的数组是可以修改的。其中的单个字符在以后可以改变。比如下面的语句：

    strncpy(a, "black", 5);

就将数组的值修改为“blackberry”。

第 9 章讨论指针和数组可以等同的情况，并讨论了为什么有时可以相等，其中的机理是怎样的。第 10 章描述了一些基于指针的使用数组的高级技巧。如果能坚持读完那一章，那么，关于数组方面的知识，仅仅是你忘掉的内容也可能比许多 C 程序员总共知道的内容还要多。

指针是 C 语言中最难正确理解和使用的部分之一，可能只有声明的语法比它更烦了。然而，它们也是 C 语言中最重要的部分之一。专业 C 程序员必须熟练掌握 `malloc()` 函数，并且学会用指针操纵匿名内存。