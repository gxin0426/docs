## HIT OS 

### num01

```bash

```

### num02

### num03

### num04

### num05

### num06

### num07

### num08

### reference

```shell
#实验课
https://blog.csdn.net/weixin_43987915/category_10331305.html?spm=1001.2014.3001.5482
https://zhuanlan.zhihu.com/p/452389902
https://juejin.cn/post/6936136067812851742
https://github.com/hoverwinter/HIT-OSLab
https://blog.csdn.net/qq_42518941/category_11232052.html
```

### os terminology/terms

#### PCB (Process Control Block)

PCB是进程存在的唯一标志

PCB包含的信息:

- 进程描述信息： 进程标识符PID 用户标识符UID
- 进程控制和管理信息： cpu、磁盘、网络流量使用情况统计。进程当前状态：就绪ready/运行running/阻塞blocked
- 资源分配清单： 使用的文件，使用的内存区域，使用的IO设备
- 处理机相关信息：PSW PC等寄存器的值 (用于实现内存切换)

#### TCB(Thread Control Block)

- Thread Control Block，线程控制块 

#### CPL

CPL是当前执行的程序或任务的特权级 它被存储在CS和SS的第0位和第1位上。通常情况下，CPL代表代码所在的段的特权级。当程序转移到不同特权级的代码段时，处理器将改变CPL。只有0和3两个值，分别表示用户态和内核态

内核： level0  服务： level 1 level2 应用程序：level3

#### DPL

DPL是段描述符中的字段，用于表示访问该段所需的最低特权级别。它指示了段本身的特权级别。

#### RPL

RPL是在段选择子中的字段，用于指定对段的访问请求的特权级别。RPL指示了发出访问请求的代码的特权级别。

- 如果RPL大于DPL，则访问会被拒绝。
- 如果RPL小于或等于DPL，并且CPL（Current Privilege Level）小于或等于DPL，则访问被授权。

#### GDT

GDT存储在系统内存中的一块特定区域，其中包含一系列的描述符。每个描述符定义了一个内存段（segment）的起始地址、大小和访问权限等信息。

内存段的起始地址、大小、访问权限和其他属性 

描述符结构：

- 段限制（Segment Limit） 
- 基地址（Base Address）
- 段属性（Segment Attributes）：一个16位的值，用于指定段的属性和访问权限，包括段的类型、特权级别、是否可读写等

#### IDT

IDT（Interrupt Descriptor Table）是x86体系结构中用于存储中断和异常处理程序信息的表格。它由一系列中断描述符组成，每个中断描述符用于定义一个特定中断或异常的处理方式

1. 中断处理程序的注册：IDT中的每个中断描述符都对应着一个特定的中断或异常类型，其中包含了中断处理程序的入口点（代码的起始地址）和相关属性。通过将中断处理程序的入口点注册到IDT中的相应中断描述符中，系统可以建立中断处理程序与中断向量之间的映射关系。
2. 中断处理程序的调用：当发生一个中断或异常时，处理器会根据中断向量号在IDT中查找相应的中断描述符，从而获取对应的中断处理程序的入口点。处理器会暂停当前正在执行的任务，保存上下文，并跳转到中断处理程序的入口点开始执行相应的中断处理逻辑。
3. 中断处理的特权级别切换：执行中断处理程序时，处理器会进行特权级别的切换。通常情况下，中断处理程序在内核态（特权级0）执行，而被中断的任务在用户态执行（特权级3）。这种切换允许中断处理程序以更高的权限访问系统资源，并进行相关的操作。
4. 异常处理和系统调用：IDT不仅用于处理外部硬件中断，还用于处理异常事件，如除零错误、页故障等。此外，IDT还包括用于系统调用（system call）的中断向量，允许用户程序请求操作系统提供的服务。

总而言之，IDT在x86架构中提供了一种集中管理中断和异常处理程序的机制，使系统能够及时响应和处理各种中断事件，保证系统的稳定性和可靠性。

#### LDT

LDT的结构由一个LDT描述符和一系列段描述符组成 

LDT描述符是一个特殊的段描述符，用于指向LDT的位置和属性。每个段描述符定义了一个内存段的起始地址、大小、访问权限等信息。

```assembly
section .data
ldt_descriptor:
    dw ldt_end - ldt_start - 1  ; LDT大小
    dd ldt_start                ; LDT起始地址

ldt_start:
    segment_descriptor_1:
        dw 0x0000  ; 段限长（段大小）
        dw 0x0000  ; 段基址（低16位）
        db 0x00    ; 段基址（中8位）
        db 0x9A    ; 段访问权限（示例为代码段，可执行）
        db 0xCF    ; 段限长和标志（高4位为段大小的高4位，低4位为标志）
        db 0x00    ; 段基址（高8位）

ldt_end:

section .text
; 使用LDT选择子访问内存
mov ax, 0x1000      ; 偏移地址
mov dx, ldt_descriptor
lldt dx            ; 加载LDT描述符到LDTR寄存器
mov al, [bx+ax]    ; 访问内存，将结果存入寄存器AL
```

上述代码定义了一个LDT，其中包含一个段描述符`segment_descriptor_1`。LDT描述符`ldt_descriptor`指向LDT的起始地址和大小。然后，通过将LDT描述符加载到LDTR（LDT Register）寄存器中，我们可以使用LDT选择子来选择并访问特定的内存段。

#### MMU

https://www.modb.pro/db/167342

MMU（Memory Management Unit）是操作系统和计算机体系结构中的一部分，用于管理和控制内存的访问和映射。它是硬件或硬件与软件结合的组件，负责实现虚拟内存系统和内存保护机制

#### TSS

TSS（Task State Segment）是操作系统中的一个数据结构，用于存储任务的上下文信息。它在x86体系结构中起着重要的作用，特别是在任务切换和多任务处理中

#### ISR

ISR（Interrupt Service Routine）是操作系统中的一段特殊的代码，用于处理中断事件的发生。中断是计算机系统中的一种事件，可以是来自硬件设备的信号（硬件中断）或由软件触发的异常情况（软件中断）

CS:IP

CS : 代码段寄存器；IP : 指令指针寄存器

https://zhuanlan.zhihu.com/p/258863021

CS:EIP

AS86

#### FCB

<img src=osimage\FCB.png width=80% height=60% align="left" />

### algorithm

LRU

Clock

LFU

### language

#### GC

1. 引用计数（Reference Counting）：这是一种简单的垃圾回收方法，它通过跟踪每个对象的引用数来确定何时释放对象。当引用计数为零时，表示没有任何引用指向对象，因此可以安全地回收内存。然而，引用计数无法处理循环引用的情况，这会导致内存泄漏。
2. 标记-清除（Mark and Sweep）：这是一种常见的垃圾回收算法。它通过标记不再可达的对象，然后清除这些对象并回收它们所占用的内存。垃圾收集器从根对象开始，通过遍历对象图形来标记可达对象，然后清除未标记的对象。
3. 复制（Copying）：这是一种适用于内存紧缺的情况的垃圾回收算法。它将内存分为两个区域：一个存活对象区域和一个空闲区域。垃圾收集器通过将存活对象复制到空闲区域，并且不复制垃圾对象，从而实现内存回收。最后，清空存活对象区域并将两个区域交换，使空闲区域成为新的存活对象区域。
4. 标记-压缩（Mark and Compact）：这是一种结合了标记和清除以及压缩技术的垃圾回收算法。它首先标记不再可达的对象，然后将存活对象压缩到内存的一端，以便为它们创建一个连续的块。最后，清除未标记的对象并更新对象的引用。
5. 分代（Generational）：这是一种基于对象的存活时间的垃圾回收方法。它假设大部分对象的生命周期较短，因此将内存分为几个代（generations）。新创建的对象分配在较新的代中，而存活更长时间的对象则逐渐晋升到较旧的代。通过这种方式，可以针对不同的代使用不同的回收策略，提高垃圾回收效率。

#### LINQ

LINQ（Language Integrated Query）是一种编程模型，最初由微软引入，用于在编程语言中集成查询功能。它最常见的应用是在.NET平台上，特别是C#语言中。

LINQ的目标是提供一种统一的查询语法，用于处理和操作各种数据源，包括集合、数据库、XML等。它允许开发人员使用类似于SQL的查询语句来查询和操作数据，而无需编写繁琐的循环和条件语句。

使用LINQ，开发人员可以通过一组统一的查询操作符（称为LINQ操作符）来表达查询逻辑。这些操作符包括过滤、排序、投影、连接等，使查询变得简洁且易于理解。

以下是一个使用LINQ的示例，假设有一个包含一组学生对象的集合，每个学生对象都有姓名和年龄属性：

```csharp
var students = new List<Student>
{
    new Student { Name = "Alice", Age = 20 },
    new Student { Name = "Bob", Age = 22 },
    new Student { Name = "Charlie", Age = 18 }
};

var result = from student in students
             where student.Age >= 20
             orderby student.Name
             select student.Name;

foreach (var name in result)
{
    Console.WriteLine(name);
}
```

上述代码使用LINQ查询语法，从学生集合中选择年龄大于等于20的学生，并按姓名进行排序。最后，只选择学生的姓名属性。通过这个简单的查询，我们可以获得满足条件的学生姓名列表。

LINQ不仅限于处理集合数据，还可以扩展到其他数据源，如数据库和XML文档。它提供了一种通用的查询机制，使得在不同的数据源上执行类似的查询操作变得简单和一致。

#### Promise

Promise（承诺）是一种用于处理异步操作的编程概念，存在于一些编程语言中，如JavaScript、TypeScript等。Promise用于管理和处理那些需要时间来完成的操作，例如从网络请求数据、读取文件等。

Promise表示一个可能尚未完成的操作，它有三种状态：

1. Pending（进行中）：Promise的初始状态，表示操作尚未完成。

2. Fulfilled（已完成）：操作成功完成，并返回结果。一旦Promise进入Fulfilled状态，它就会保持在这个状态，并且不会再改变。

3. Rejected（已拒绝）：操作失败或出现错误。一旦Promise进入Rejected状态，它也会保持在这个状态，并且不会再改变。

Promise对象提供了一组方法，用于处理操作的完成状态，这些方法包括：

- `then()`: 用于指定当Promise进入Fulfilled状态时要执行的回调函数。
- `catch()`: 用于指定当Promise进入Rejected状态时要执行的回调函数。
- `finally()`: 无论Promise进入Fulfilled还是Rejected状态，都会执行的回调函数。

使用Promise，开发人员可以通过链式调用这些方法来处理异步操作的结果。这样可以更好地组织和控制异步代码，并避免回调地狱（callback hell）的问题。

以下是一个简单的JavaScript Promise的示例，模拟一个异步操作的延迟：

```javascript
function getData() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const data = 'Hello, world!';
      // 模拟异步操作成功
      resolve(data);
      // 模拟异步操作失败
      // reject(new Error('Failed to get data'));
    }, 2000);
  });
}

getData()
  .then((result) => {
    console.log(result); // 输出: Hello, world!
  })
  .catch((error) => {
    console.error(error); // 输出: Error: Failed to get data
  });
```

上述代码创建了一个Promise对象，表示一个异步操作，延迟2秒后返回数据。在调用`then()`方法时，传递了一个回调函数，用于在操作成功时处理结果。在调用`catch()`方法时，传递了一个回调函数，用于在操作失败时处理错误。

通过Promise，我们可以更清晰地编写和处理异步代码，并使用链式调用来组织多个异步操作的顺序和逻辑。

#### Lambda

Lambda表达式是一种匿名函数的表示形式，存在于一些编程语言中，如Python、Java、C#等。Lambda表达式允许开发人员定义和传递简洁的函数代码块，而无需显式命名函数。

Lambda表达式通常用于函数式编程和处理集合数据。它可以作为参数传递给其他函数或方法，也可以在需要函数作为返回值的情况下使用。

Lambda表达式的语法通常包含以下部分：

1. 参数列表：定义函数的输入参数，可以是零个或多个参数。
2. 箭头符号（或其他分隔符）：用于分隔参数列表和函数体。
3. 函数体：包含要执行的代码块。

下面是一些编程语言中Lambda表达式的示例：

**Python**：

```python
# 使用Lambda表达式定义一个匿名函数并传递给map()函数
numbers = [1, 2, 3, 4, 5]
squared_numbers = list(map(lambda x: x**2, numbers))
print(squared_numbers)  # 输出: [1, 4, 9, 16, 25]
```

**Java**：

```java
// 使用Lambda表达式定义一个匿名函数并传递给Thread类的构造函数
Thread thread = new Thread(() -> {
    System.out.println("Hello, world!");
});
thread.start();
```

**C#**：

```csharp
// 使用Lambda表达式定义一个匿名函数并传递给LINQ查询
int[] numbers = { 1, 2, 3, 4, 5 };
var squaredNumbers = numbers.Select(x => x * x);
foreach (var number in squaredNumbers)
{
    Console.WriteLine(number);  // 输出: 1 4 9 16 25
}
```

Lambda表达式的优点在于它简化了函数定义和传递过程，尤其是对于简单的函数逻辑。它允许开发人员在需要函数的地方直接使用匿名函数，避免了编写额外的命名函数的麻烦。此外，Lambda表达式还提高了代码的可读性和简洁性，使代码更易于理解和维护。

### CPU原理

#### 布尔逻辑和布尔门

布尔代数true or false：NOT AND OR

 NOT   gate

<img src=osimage\notgate2.png width=40% height=30% align="left" />

AND gate

<img src=osimage\andgate.png width=40% height=40% align='left'>

OR gate

<img src=osimage\orgate.png width=40% height=40% align='left'>

XOR

<img src=osimage\xor1.png width=40% height=40% align='left'>

<img src=osimage\xor2.png width=40% height=40% align='left'>

#### ALU

半加器

<img src=osimage\halfadd.png width=40% height=40% align='left'>

<img src=osimage\halfadd2.png width=40% height=40% align='left'>

全加器

<img src=osimage\fulladd.png width=40% height=40% align='left'>

<img src=osimage\fulladd2.png width=40% height=40% align='left'>

8位加法器

<img src=osimage\add8.png width=60% height=60% align='left'>

alu

<img src=osimage\alu1.png width=60% height=60% align='left'>

<img src=osimage\alu2.png width=60% height=60% align='left'>

寄存器和内存

锁存器

<img src=osimage\lock1.png width=60% height=60% align='left'>

<img src=osimage\lock2.png width=60% height=60% align='left'>

<img src=osimage\lock3.png width=60% height=60% align='left'>

<img src=osimage\lock4.png width=60% height=60% align='left'>

set 和 reset 都是0 电路会输出最后放入的内容 这时 存住了1位信息

<img src=osimage\lock5.png width=60% height=60% align='left'>

寄存器

多路复用器

<img src=osimage\multiplexer.png width=60% height=60% align='left'>

内存

<img src=osimage\memory.png width=60% height=60% align='left'>

<img src=osimage\memory2.png width=60% height=60% align='left'>

<img src=osimage\memory3.png width=60% height=60% align='left'>

ram register alu

 程序由一个个操作组成 这些操作叫做指令 instruction 

如果是数学指令 比如加/减 cpu会让ALU进行数学运算

也可能是内存指令，CPU会和内存通信，然后读/写值

<img src=osimage\instructionset.png width=60% height=60% align='left'>

 4-BIT OPCODE 操作码

ADDRESS OR REGISTER 地址或寄存器

**instruction address register 指令地址寄存器 ： 存当前指令的内存地址**

**instruction register 指令寄存器： 存当前指令**

<img src=osimage\microarch.png width=60% height=60% align='left'>

cpu第一阶段叫“取指令阶段”

<img src=osimage\instructionset2.png width=60% height=60% align='left'>

指令存到指令寄存后 进入解码阶段decode

前4位0010是LOAD A指令 后4位 1110是RAM地址 转成十进制是14

<img src=osimage\instructionset4.png width=60% height=60% align='left'>

为了识别LOAD A指令 需要一个电路 检查操作码是部署0010

检查后打开RAM‘允许读取线’，把地址14传过去

用‘检查是否LOAD A指令的电路’启用寄存器A的‘允许写入线’

写入以后  指令地址寄存器 instruction address register + 1  

<img src=osimage\instructionset3.png width=60% height=60% align='left'>

新一层抽象

<img src=osimage\instructionset5.png width=60% height=60% align='left'>

<img src=osimage\instructionset6.png width=60% height=60% align='left'>

时钟速度 赫兹

<img src=osimage\instructionset7.png width=60% height=60% align='left'>

cpu “取指令 解码 执行” 的速度叫做 时钟速度 单位赫兹

1赫兹代表一秒1个周期

<img src=osimage\instructionset8.png width=60% height=60% align='left'>

<img src=osimage\cpu.png width=60% height=60% align='left'>

ALU control-UNIT RAM clock



立即值

<img src=osimage\pipeline.png width=60% height=60% align='left'>
