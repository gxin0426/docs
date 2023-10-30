## 编译原理

#### 整体架构

```lua
+----------------+     +----------------+     +----------------+     +----------------+
|    词法分析器     | --> |   语法分析器   | --> |   语义分析器   | --> |   代码优化器   |
+----------------+     +----------------+     +----------------+     +----------------+
         |                      |                      |                      |
         |                      |                      |                      |
         v                      v                      v                      v
+----------------+     +----------------+     +----------------+     +----------------+
|   字符流输入   | --> |  词法单元序列  | --> |       语法树      | --> |      中间代码表示 |
+----------------+     +----------------+     +----------------+     +----------------+
                                                       |
                                                       |
                                                       v
                                             +----------------+
                                             | 目标代码生成器 |
                                             +----------------+
                                                       |
                                                       |
                                                       v
                                             +----------------+
                                             |   目标机器码   |
                                             +----------------+

```

在这个框架图中，编译器分为五个主要部分：

1. 字符流输入：编译器从输入文件中读取字符流。
2. 词法分析器：将字符流分解成词法单元序列。
3. 语法分析器：将词法单元序列转换成语法树。
4. 语义分析器：检查语法树是否符合语义规则，并生成中间代码表示。
5. 代码优化器：对中间代码进行优化，以提高代码执行效率。
6. 目标代码生成器：将优化后的中间代码转换为目标机器码。

这个框架图提供了一个概览，展示了编译器的主要组成部分和它们之间的关系。不同的编译器实现可能会有所不同，但通常都遵循类似的基本原理。

### 概念篇

#### LL(1)文法

两个限制条件

1. **First集合互斥**： 从任何非终结符号开始的任何两个产生式的首符号集合不能相交，即每个非终结符号在任何时候都只能展开成一种符号串，这个条件叫做“First集合互斥”

2. **Follow集合不相交**：对于任何一个非终结符号的任何一个产生式，如果它能推导出空串，那么这个非终结符号的所有产生式都不能再推导出非空符号串，即每个非终结符号在任何时候都不应该展开为一个空串，这个条件叫做“Follow集合不相交”

这两个限制条件保证了对于任何一个给定的符号串，从左到右只需要向前看一位（Look Ahead 1），就可以确定使用哪个产生式来推导出下一个符号。因此，LL(1)文法可以被用于自上而下的语法分析，包括递归下降分析等。

举例：

1. First集合互斥：

产生式：

```css
A → aB | aC
B → b
C → c
```

我们可以看出，A可以展开为aB或者aC，但是这**两个产生式的首符号集合都包含a**，因此这个文法不是LL(1)文法

为了满足First集合互斥的限制条件，我们可以将文法修改为：

```css
A → aD
D → B | C
B → b
C → c
```

这个文法满足First集合互斥的限制条件，因为对于非终结符号D，它的两个产生式的首符号集合不相交。

2. Follow集合不相交：

假设有以下的产生式：

```css
A → aB | ε
B → b | c
```

#### First集合概念

计算First集合的基本思路：如果一个符号串以非终结符号开头，那么这个符号串的First集合就包含该非终结符号的First集合的符号。如果一个符号串以终结符号开头，那么这个符号串的First集合就包含这个终结符号，如果一个符号串可以推导出空串，那么空串也是这个符号串的一个合法首符号，即空串也在First集合中。

文法：

```css
A → B C
B → a
C → b | ε
```

计算First集合：

```css
First(A) = {a}
First(B) = {a}
First(C) = {b, ε}
```

#### follow集合概念

计算follow集合的基本思路：首先将EOF(表示输入串结束)加入到文法开始符号的Follow集合中，





#### 怎么理解follow集，first集，select集

在语法分析中，follow集，first集，和select集都是用来帮助构建语法树的辅助集合。

1. First集

First集是指文法符号串中的第一个终结符号集合。例如，在一个产生式A -> BC | b中，First(A)就是{b, First(B)}，其中First(B)表示B能够产生的终结符号集合。

1. Follow集

Follow集是指文法符号串中跟在一个非终结符号之后的终结符号集合。例如，在一个产生式A -> BC中，Follow(B)就是跟在B后面的终结符号的集合。

1. Select集

Select集是指产生式右部的所有可能推导出的终结符号串的集合。例如，在一个产生式A -> BC | b中，Select(A -> BC)就是First(B)交Follow(A)，而Select(A -> b)就是{b}。

这些集合在构建语法树、进行语法分析时起到了重要作用，其中First集和Follow集主要用于构建预测分析表，而Select集则用于构建递归下降分析器。

#### 举一个例子说明一下这三个集合



假设有如下文法：

```css
S -> A B
A -> a | ε
B -> b | ε
```

其中，`ε`表示空串。

1. First集

- First(S) = {a}
- First(A) = {a, ε}
- First(B) = {b, ε}

1. Follow集

- Follow(S) = {$}（$表示输入的结束符号）
- Follow(A) = {b}
- Follow(B) = {$}

1. Select集

- Select(S -> AB) = First(A) = {a, ε}
- Select(A -> a) = {a}
- Select(A -> ε) = Follow(A) = {b}
- Select(B -> b) = {b}
- Select(B -> ε) = Follow(B) = {$}

这些集合在构建语法分析器时非常重要。例如，在构建预测分析表时，我们需要使用First集和Follow集来判断哪个产生式应该被用于推导输入串。在构建递归下降分析器时，我们需要使用Select集来确定每个非终结符号的递归下降函数应该推导哪些产生式。





## create a complier

### 源语言的语法

```rust
program     -> statement*
statement   -> exprStmt | printStmt
exprStmt    -> expr ';'
printStmt   -> "print" expr ';'
expr        -> literal | binaryExpr
binaryExpr  -> expr op expr
op          -> "+" | "-" | "*" | "/"
literal     -> NUMBER | STRING
```

以上语法使用类似于 `BNF`（巴克斯-诺尔范式）的语法来定义源语言的语法规则。

1. `program`：表示程序的入口，它由零个或多个 `statement` 组成。
2. `statement`：表示程序中的语句。它可以是表达式语句（`exprStmt`）或打印语句（`printStmt`）。
3. `exprStmt`：表示一个表达式语句，由一个表达式（`expr`）和一个分号（`;`）组成。
4. `printStmt`：表示一个打印语句，以 "print" 关键字开头，后面跟一个表达式（`expr`）和一个分号（`;`）。
5. `expr`：表示一个表达式，可以是字面量（`literal`）或二元表达式（`binaryExpr`）。
6. `binaryExpr`：表示一个二元表达式，由两个表达式和一个运算符（`op`）组成。
7. `op`：表示一个运算符，可以是加法（`+`）、减法（`-`）、乘法（`*`）或除法（`/`）。
8. `literal`：表示一个字面量，可以是数字（`NUMBER`）或字符串（`STRING`）。

简单来说，上述语法规则定义了一种类似于 C 语言的简单语言，它支持数字、字符串、四则运算和打印语句。
