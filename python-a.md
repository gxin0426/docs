![](pythonA/fm.png)

### 第二章- 一切皆对象

#### 2.2 type class object 关系

<img src="pythonA/typevsclass.png" width="500" >

在Python中，`type`, `object`, 和 `class` 之间的关系是紧密且复杂的。以下是关于这三者之间关系的说明和代码示例：

1. **object**: 是所有Python类的基类。换句话说，所有Python类，无论是否显式声明，都是继承自 `object` 的。

2. **type**: 是一个内建的元类，用于创建其他类。简单来说，就像类是对象的蓝图，元类是类的蓝图。默认情况下，`type` 是所有类的元类。

3. **class**: 用户可以定义的自定义类型。类的实例是对象。但在Python中，类本身也是对象，它们是`type`的实例。

##### 代码示例：

```python
# 定义一个简单的类
class MyClass:
    pass

# MyClass 是 object 的子类
print(issubclass(MyClass, object))  # True

# MyClass 的类型是 type
print(isinstance(MyClass, type))  # True

# MyClass 的实例的类型是 MyClass
my_instance = MyClass()
print(isinstance(my_instance, MyClass))  # True

# 但 MyClass 的实例是 object 的实例
print(isinstance(my_instance, object))  # True
```

可以使用 `type` 函数动态地创建类，而不需要使用`class`关键字：

```python
# 使用type动态创建一个类
NewClass = type('NewClass', (object,), {'x': 5})

new_instance = NewClass()
print(new_instance.x)  # 输出 5
```

在上面的代码中，我们使用 `type` 动态地创建了一个新类 `NewClass`，它有一个属性 `x`。

总结：在Python中，所有的类都是`type`的实例，并且都继承自`object`。这种设计使得Python的对象模型特别灵活和统一。

#### 2.3 python中常见的内置类型

<img src="pythonA/internaltype.png" width="500" >

当然可以。根据你提供的图，以下是对Python中列出的每种类型的简要解释：

1. **对象的三个特征**

   - 身份(id)

     - 每个对象在Python中都有一个唯一的身份标识，通常我们可以认为它是对象在内存中的地址。

     - 可以使用内置函数 `id()` 获取对象的身份。

     - 身份在对象的生命周期中是恒定的。也就是说，一旦对象被创建，它的身份就不会改变，直到它被销毁。

       `x = [1, 2, 3]
       print(id(x))  # 输出该列表对象的身份`

   - 类型(type)

     - 对象的类型决定了该对象可以存储什么样的值，可以进行哪些操作，以及遵循哪些规则。 
     - 可以使用内置函数 `type()` 获取对象的类型。
     - 在Python中，类型本身也是对象。例如，`int`, `float`, `list`, `dict` 等都是类型对象。
     - 与身份一样，对象的类型在其生命周期中也是不变的。也就是说，你不能更改对象的类型，除非你重新创建该对象。

     `x = [1, 2, 3]
     print(type(x))  # 输出 <class 'list'>`

   - 值(value)

     - 对象的值是数据项表示的数据。例如，整数对象 `3` 的值就是数字 `3`，字符串对象 `"hello"` 的值就是文本 `hello`。
     - 对象的值可以是可变的或不可变的，这取决于对象的类型。例如，列表是可变的，这意味着你可以修改其内容；而字符串是不可变的，这意味着你不能修改字符串的内容。

     ```python
     x = [1, 2, 3]
     x[0] = 4  # 修改列表的值
     print(x)  # 输出 [4, 2, 3]
     y = "hello"
     y[0] = "H"  # 这会导致错误，因为字符串是不可变的`
     ```

2. **None（全局只有一个）**：

   - `None` 是Python中的一个特殊类型，代表“无”或“不存在”。
   - 它经常用于表示变量尚未被赋值，或函数没有返回值。
   - 在Python中，`None` 是唯一的一个`NoneType` 对象。

3. **数值**：
   - Python支持多种数值类型，如整数 (`int`), 浮点数 (`float`), 复数 (`complex`)  , bool。
   - 例如：`5` (整数), `3.14` (浮点数), `3 + 4j` (复数), True or False(bool)。

4. **迭代类型**

   1. list tuple string  dic set file-object generator 等等

   ```python
   def generator_func():
       yield 1
       yield 2
       yield 3
   for item in generator_func():
       print(item)
   ```

   任何定义了`__iter__()`方法或`__getitem__()`方法的对象都可以被认为是可迭代的。实际上，当你尝试在对象上进行迭代时，Python会首先查找`__iter__()`，如果没有找到，它会尝试`__getitem__()`，从索引0开始并递增，直到引发`IndexError` 。

5. **序列类型**

   1. Python中的序列类型是一组按特定顺序排列的元素集合。以下是您提供的序列类型，并配有简短的解释及代码演示：

      1. **list (列表)**:
         - 一个有序的元素集合，元素可以是任何数据类型，并且可以修改。
         
         ```python
         my_list = [1, 2, 3, 'a', 'b']
         print(my_list[3])  # 输出 'a'
         ```

      2. **bytes, bytearray, memoryview**:
         
         - `bytes`: 不可变的字节序列。
         - `bytearray`: 可变的字节序列。
         - `memoryview`: 内存查看对象，它暴露了底层数据的字节级视图。
         
         ```python
         b = bytes([65, 66, 67])
         print(b)  # 输出 b'ABC'
         
         ba = bytearray([65, 66, 67])
         print(ba)  # 输出 bytearray(b'ABC')
         
         mv = memoryview(ba)
         print(mv[1])  # 输出 66
         ```
         
      3. **range**:
         - 返回一个数字序列。
         
         ```python
         for i in range(3):
             print(i)  # 输出 0, 1, 2
         ```

      4. **tuple (元组)**:
         - 与列表类似，但元组是**不可变**的。
         
         ```python
         my_tuple = (1, 2, 3, 'a', 'b')
         print(my_tuple[3])  # 输出 'a'
         ```

      5. **str (字符串)**:
         
         - 一个字符序列。
         
         ```python
         my_str = "Hello, world!"
         print(my_str[7])  # 输出 'w'
         ```
         
      6. **array**:
         
         - 与列表类似，但`array`模块提供了一个更为紧凑的存储方式，它要求所有元素都是相同的数据类型。
         
         ```python
         from array import array
         #i表示数组中的元素是哪种数据类型
         my_array = array('i', [1, 2, 3, 4])
         print(my_array[2])  # 输出 3
         ```

      这些序列类型都支持索引、切片等常见操作，并有自己独特的特性和用途。

6. **字典 (dict)**：

   - 一个无序的键-值对集合。
   - 键必须是唯一的，例如：`{'a': 1, 'b': 2, 'c': 3}`。
   - 字典也是可变的。

7. **集合 (set)**：

   `set` 是 Python 中的一种数据结构，用于存储不重复元素的无序集合

   1. 使用花括号 `{}`:
   ```python
   my_set = {1, 2, 3, 4, 5}
   ```

   2. 使用 `set()` 函数:
   ```python
   my_list = [1, 2, 3, 4, 5, 5]
   my_set = set(my_list)  # 结果为 {1, 2, 3, 4, 5}
   ```
      - 创建一个空集合：
   ```python
   empty_set = set()
   ```

   注意: 不能直接使用空的 `{}` 来创建空集合，因为这实际上会创建一个空字典。

   基本用法：

   1. 添加元素:

      使用 `add` 方法添加单个元素:
   ```python
   my_set.add(6)
   ```
   ​	使用 `update` 方法添加多个元素 (可从其他序列添加):

   ```python
   my_set.update([6, 7, 8])
   ```

   2. 删除元素:
      
      使用 `remove` 方法删除指定元素 (如果元素不存在，会引发错误):
   ```python
   my_set.remove(5)
   ```
      - 使用 `discard` 方法删除指定元素 (如果元素不存在，不会引发错误):
   ```python
   my_set.discard(5)
   ```
      - 使用 `pop` 方法随机删除并返回一个元素:
   ```python
   elem = my_set.pop()
   ```
      - 使用 `clear` 方法清空集合:
   ```python
   my_set.clear()
   ```

   3. 集合运算:
      - 交集 (使用 `&` 运算符或 `intersection` 方法):
   ```python
   a = {1, 2, 3, 4}
   b = {3, 4, 5, 6}
   result = a & b  # 或者 result = a.intersection(b)
   ```
      - 并集 (使用 `|` 运算符或 `union` 方法):
   ```python
   result = a | b  # 或者 result = a.union(b)
   ```
      - 差集 (使用 `-` 运算符或 `difference` 方法):
   ```python
   result = a - b  # 或者 result = a.difference(b)
   ```
      - 对称差集 (使用 `^` 运算符或 `symmetric_difference` 方法):
   ```python
   result = a ^ b  # 或者 result = a.symmetric_difference(b)
   ```

   4. 其他方法和操作:
      
      检查一个元素是否在集合中:
   ```python
   if 3 in my_set:
       print("3 is in the set")
   ```
   ​	获取集合的长度:

   ```python
   length = len(my_set)
   ```
   ​	比较两个集合:

   ```python
   a = {1, 2, 3}
   b = {1, 2, 3, 4}
   is_subset = a.issubset(b)  # 检查 a 是否是 b 的子集
   ```

8. **frozenset**：

   - 与 `set` 相似，但是`frozenset`是不可变的。
   - 由于其不可变性，它可以作为字典的键，而普通的 `set` 不能。

9. **上下文管理器 (with)**：

   - 不是数据类型，而是一种控制资源，如文件或网络连接的分配和释放的方法。
   - 常用于文件操作，例如：`with open("file.txt", "r") as f: ...`

10. **生成器**：

   - 一种特殊的迭代器，允许你使用函数而不是循环来产生迭代的数据。
   - 使用 `yield` 关键字生成值，例如：
     ```python
     def generator_func():
         yield 1
         yield 2
         yield 3
     ```

11. **其他类型**

    1. **模块类型**:

       - 当你导入一个模块，Python创建了一个模块对象。
       - 你可以使用 `type(module_name)` 来查看。

    2. **类类型**:

       - 在Python中，你可以使用 `class` 关键字定义一个新的类型。
       - 示例: 
         ```python
         class MyClass:
             pass
         ```

    3. **实例类型**:

       - 当你从一个类创建一个对象时，这个对象是类的一个实例。
       - 示例:
         ```python
         obj = MyClass()
         ```

    4. **方法类型**:
       - 这是关联到对象的函数，可以访问和修改对象的数据。
       - 示例:
         ```python
         class MyClass:
             def my_method(self):
                 pass
         ```

    5. **代码类型**:
       - 这是Python中的代码对象，通常你不直接与它们交互。

    6. **object对象**:
       - 所有Python类都直接或间接地从 `object` 类派生。
       - 它是最基本的、最通用的类型。

    7. **type对象**:

       - `type` 是Python的元类，它本身是一个类，用于创建和管理其他类。

    8. **ellipsis对象**:

       - 通常表示为 `...`，在多种上下文中使用，例如在NumPy数组切片中表示连续的行或列。

    9. **notimplemented对象**:

       - 这是一个特殊的值，由二进制（特殊）方法返回，以表示它们不支持特定操作。


### 第三章-魔法函数

<img src="pythonA/magicfunction.png" width="500" >

Python 中的 "魔法方法"（Magic Methods），也被称为 "特殊方法"（Special Methods）或 "双下方法"（Dunder Methods，dunder 是 "double underscore" 的缩写），是指在 Python 类中定义的一些带有双下划线前后缀的方法，如 `__init__`、`__call__`、`__str__` 等。

魔法方法允许对象自定义某些内置操作，使得对象可以更加符合 Python 的数据模型，并且能够更自然地融入 Python 的语法和内置函数。

在 Python 中，魔法方法允许我们自定义对象的行为以模拟内置类型的行为，例如数字、字符串或列表。通过实现这些魔法方法，我们可以让自定义对象支持如加法、减法、乘法等操作，或者可以像字典或列表一样进行索引。

这里有一些具体的例子来解释这个概念：

##### 模拟数字类型

通过定义魔法方法如 `__add__`，我们可以让自定义对象支持加法操作

```python
class MyNumber:
    def __init__(self, value):
        self.value = value
    
    def __add__(self, other):
        return MyNumber(self.value + other.value)
```

使用这个类，我们可以进行如下操作：

```python
a = MyNumber(1)
b = MyNumber(2)
c = a + b  # 这里会调用 a.__add__(b)，得到一个新的 MyNumber 对象
```

##### 模拟容器类型

通过定义魔法方法如 `__getitem__` 和 `__setitem__`，我们可以让自定义对象像列表或字典一样进行索引。

```python
class MyList:
    def __init__(self):
        self.data = []
    
    def __getitem__(self, index):
        return self.data[index]
    
    def __setitem__(self, index, value):
        while len(self.data) <= index:
            self.data.append(None)
        self.data[index] = value
```

使用这个类，我们可以进行如下操作：

```python
my_list = MyList()
my_list[0] = 1  # 这里会调用 my_list.__setitem__(0, 1)
print(my_list[0])  # 这里会调用 my_list.__getitem__(0)
```

##### 与 Python 语法和内置函数的融合

魔法方法还可以让我们的自定义对象更自然地融入 Python 语法和内置函数。例如，通过定义 `__str__` 或 `__repr__` 方法，我们可以改变打印对象时的输出，或者使用 `str()` 和 `repr()` 函数获取对象的字符串表示。

```python
class MyObject:
    def __str__(self):
        return "This is MyObject"
```

使用这个类，我们可以进行如下操作：

```python
obj = MyObject()
print(obj)  # 输出：This is MyObject
```

通过实现魔法方法，我们的自定义对象能够更符合 Python 的数据模型，使得对象的使用更加直观和 Pythonic，也就是符合 Python 的编程习惯。

##### 以下是一些魔法方法的作用示例：

###### 1. `__init__`
用于在创建对象后对其进行初始化。这个方法会在类的对象被实例化时自动调用。你可以在 `__init__` 方法中设置对象属性的初始值或执行任何其他必要的设置

1. 默认参数：如果 `__init__` 方法中的参数有默认值，那么在实例化对象时可以省略这些参数。如果省略，将使用默认值。

   ```python
   class Example:
       def __init__(self, a, b=2):
           self.a = a
           self.b = b
   obj = Example(1)  # 这里只提供了一个参数，b 使用默认值 2
   ```
   
2. 可变数量的参数：`__init__` 方法可以接受可变数量的参数，如 `*args` 和 `**kwargs`，这允许在实例化时提供不同数量的参数。

   ```python
   class Example:
       def __init__(self, a, *args, **kwargs):
           self.a = a
           self.args = args
           self.kwargs = kwargs
   
   obj = Example(1, 2, 3, x=4, y=5)  # 这里提供了多个参数
   ```

3. 没有参数：如果 `__init__` 方法没有定义任何参数（除了 `self` 外），那么在实例化时不需要提供任何参数。

   ```python
   class Example:
       def __init__(self):
           self.a = 1
   obj = Example()  # 这里没有提供参数
   ```

总之，在实例化类的对象时，应根据 `__init__` 方法的定义提供适当的参数。如果没有提供必要的参数且没有默认值，Python 将引发类型错误（TypeError），指出缺少必要的位置参数。

**番外：**

`*args` 和 `**kwargs` 是 Python 中的两种特殊参数传递方法，它们允许你在函数或方法中传递可变数量的非关键字参数（`*args`）和关键字参数（`**kwargs`）

**`*args`**

- `*args` 允许你传递可变数量的非关键字参数到函数中。
- 在函数内部，`args` 是一个包含所有传递的参数的元组。

```python
def function_with_args(*args):
    for arg in args:
        print(arg)

# 调用函数，传递任意数量的参数
function_with_args(1, 2, 3, 4)  # 输出：1 2 3 4
```

`**kwargs`

- `**kwargs` 允许你传递可变数量的关键字参数到函数中。
- 在函数内部，`kwargs` 是一个包含所有关键字参数的**字典**。

```python
def function_with_kwargs(**kwargs):
    for key, value in kwargs.items():
        print(key, value)

# 调用函数，传递任意数量的关键字参数
function_with_kwargs(a=1, b=2, c=3)  # 输出：a 1 b 2 c 3
```

- 结合使用 `*args` 和 `**kwargs`

你还可以在同一个函数中同时使用`*args` 和 `**kwargs`，但 `*args` 必须出现在 `**kwargs` 之前。

```python
def function_with_args_and_kwargs(*args, **kwargs):
    print("Args:")
    for arg in args:
        print(arg)
    
    print("Kwargs:")
    for key, value in kwargs.items():
        print(key, value)

# 调用函数，传递非关键字参数和关键字参数
function_with_args_and_kwargs(1, 2, a=3, b=4)
# 输出：
# Args:
# 1
# 2
# Kwargs:
# a 3
# b 4
```

- 在函数调用中使用 `*` 和 `**`

你也可以在函数调用时使用 `*` 和 `**`，用来解包元组和字典，将它们作为参数传递给函数。

```python
def function(a, b, c):
    print(a, b, c)

# 使用 * 解包元组
args = (1, 2, 3)
function(*args)  # 输出：1 2 3

# 使用 ** 解包字典
kwargs = {'a': 1, 'b': 2, 'c': 3}
function(**kwargs)  # 输出：1 2 3
```

- 小结

  - `*args` 用于传递可变数量的非关键字参数。

  - `**kwargs` 用于传递可变数量的关键字参数。

  - 它们可以在函数定义和函数调用中使用，实现参数的动态传递和解包。

###### 2. `__str__` 和 `__repr__`

`__str__` 和 `__repr__` 是 Python 中的两个魔法方法，都是用于定义对象的字符串表示形式，但它们之间有一些区别，并且各自适用于不同的场合。

`__str__`

- `__str__` 方法应返回一个字符串，这个字符串是对用户友好的，简单明了的对象描述。
- 当你使用 `print()` 函数或 `str()` 函数对一个对象进行转换时，Python 会自动调用该对象的 `__str__` 方法。

```python
class MyClass:
    def __str__(self):
        return "This is a MyClass object"

obj = MyClass()
print(obj)  # 输出：This is a MyClass object
```

`__repr__`

- `__repr__` 方法应返回一个字符串，这个字符串尽可能包含对象的详细信息，甚至可以用来重新创建对象（当然，这不是强制的）。
- 当你在控制台中直接输入对象名，或使用 `repr()` 函数进行转换时，Python 会自动调用该对象的 `__repr__` 方法。

示例：

```python
class MyClass:
    def __repr__(self):
        return 'MyClass()'

obj = MyClass()
print(repr(obj))  # 输出：MyClass()
```

当两者同时存在时

如果一个对象同时定义了 `__str__` 和 `__repr__`，则 `print()` 和 `str()` 会优先使用 `__str__`，而在控制台中直接输入对象名或使用 `repr()` 会使用 `__repr__`。

```python
class MyClass:
    def __str__(self):
        return "This is a MyClass object"
    
    def __repr__(self):
        return 'MyClass()'

obj = MyClass()
print(obj)  # 输出：This is a MyClass object
print(repr(obj))  # 输出：MyClass()
```

当只定义了 `__repr__` 时

如果只定义了 `__repr__` 而没有定义 `__str__`，那么 `__repr__` 也会在调用 `print()` 和 `str()` 时被使用。

```python
class MyClass:
    def __repr__(self):
        return 'MyClass()'

obj = MyClass()
print(obj)  # 输出：MyClass()
```

总结

- `__str__` 更注重用户可读性，用于输出简单明了的对象信息。
- `__repr__` 更注重详细性和精确性，甚至可以用来重新创建对象。

###### 3. `__call__`
`__call__` 是一个特殊方法，用于使一个对象变得可调用。换句话说，它允许一个对象的实例被像函数一样调用。当你调用一个对象的实例就像调用一个函数一样时，Python 会自动执行该对象的 `__call__` 方法。

```python
class Greeter:
    def __init__(self, greeting="Hello"):
        self.greeting = greeting
    
    def __call__(self, name):
        return f"{self.greeting}, {name}!"

# 创建一个 Greeter 对象
greeter = Greeter()

# 调用 greeter 实例，传递 "World" 作为参数
print(greeter("World"))  # 输出：Hello, World!

# 你也可以改变问候语
greeter_custom = Greeter("Hi there")
print(greeter_custom("Python"))  # 输出：Hi there, Python!
```

- `Greeter` 类有一个 `__call__` 方法，它接受一个 `name` 参数。
- 当我们像调用函数一样调用 `Greeter` 类的实例 `greeter("World")` 时，`__call__` 方法被执行。
- `__call__` 方法返回一个字符串，该字符串包含一个问候语和传递给它的 `name`。

`__call__` 方法增加了对象的灵活性，使其可以以多种方式被使用和调用。通过 `__call__` 方法，你可以创建像函数一样表现的对象，同时还保留了更多 OOP（面向对象编程）的特性，如状态保持和方法。

###### 4. `__getitem__` 和 `__setitem__`

用于自定义获取和设置元素的行为，使对象可以像列表或字典一样进行索引。
```python
class Example:
    def __getitem__(self, key):
        return self.data[key]
    def __setitem__(self, key, value):
        self.data[key] = value
```

###### 5. `__eq__` 和其他比较方法
`__eq__` 是一个 Python 的魔法方法，用于自定义对象的等值比较逻辑。当你使用等号 `==` 来比较两个对象是否相等时，Python 会调用这个方法。

`__eq__` 方法应该接受两个参数：第一个是 `self`（代表实例自身），第二个是被比较的对象。方法应该返回一个布尔值，表示两个对象是否相等。

```python
class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
        
    def __eq__(self, other):
        if isinstance(other, Point):
            return self.x == other.x and self.y == other.y
        return False

# 创建两个 Point 对象
p1 = Point(1, 2)
p2 = Point(1, 2)
p3 = Point(3, 4)

# 使用 == 操作符比较两个 Point 对象
print(p1 == p2)  # 输出：True
print(p1 == p3)  # 输出：False
```

- `Point` 类有一个 `__eq__` 方法，用于比较两个 `Point` 对象是否有相同的 `x` 和 `y` 值。
- 当我们使用 `==` 操作符比较两个 `Point` 对象时，`__eq__` 方法被调用，并返回一个布尔值。

通过自定义 `__eq__` 方法，你可以根据自己的需求定义对象的等值比较规则，以支持 `==` 操作符的自定义行为。这使得你的自定义对象能够更自然地融入 Python 的语法和数据模型中。

###### 6. `__add__` 和其他数学运算方法
`__add__` 是Python中的一个魔法方法，用于重载加法操作符（`+`）。通过定义该方法，你可以为自定义类的对象自定义加法操作。

当使用 `+` 运算符将两个对象相加时，Python会自动调用 `__add__` 方法，并传递参与加法操作的两个对象作为参数。

```python
class ComplexNumber:
    def __init__(self, real, imag):
        self.real = real
        self.imag = imag
        
    def __add__(self, other):
        if isinstance(other, ComplexNumber):
            return ComplexNumber(self.real + other.real, self.imag + other.imag)
        return NotImplemented
    
    def __str__(self):
        return f"({self.real} + {self.imag}i)"

# 创建两个 ComplexNumber 对象
num1 = ComplexNumber(1, 2)
num2 = ComplexNumber(3, 4)

# 使用 + 运算符将两个 ComplexNumber 对象相加
result = num1 + num2

# 输出结果
print(result)  # 输出：(4 + 6i)
```

在这个示例中：

- `ComplexNumber` 类有一个 `__add__` 方法，用于定义两个 `ComplexNumber` 对象的加法操作。方法返回一个新的 `ComplexNumber` 对象，其实部和虚部分别为两个被加对象的实部和虚部之和。
- 当我们使用 `+` 运算符将两个 `ComplexNumber` 对象相加时，`__add__` 方法被调用，并返回一个新的 `ComplexNumber` 对象作为结果。

###### 7. `__enter__` 和 `__exit__`
`__enter__` 和 `__exit__` 是 Python 的魔法方法，主要用于实现上下文管理器。上下文管理器是一种 Python 对象，它定义了在 with 语句中需要设置的运行时上下文，并且由于某些代码块的执行而进行清理。这通常用于管理如文件、网络连接和锁这样的资源。

`__enter__(self)`

当执行 with 语句时，会首先调用上下文管理器对象的 `__enter__` 方法。`__enter__` 方法的返回值会被赋值给在 as 关键字后面指定的变量。

示例：

```python
class ManagedFile:
    def __init__(self, filename):
        self.filename = filename

    def __enter__(self):
        self.file = open(self.filename, 'w')
        return self.file

```

`__exit__(self, exc_type, exc_val, exc_tb)`

当 with 语句中的代码块被执行完毕后，`__exit__` 方法会被调用，即使在代码块中发生了异常也是如此。`__exit__` 方法可以接受三个参数，它们分别是异常类型、异常值和追踪信息。

示例：

```python
class ManagedFile:
    # ... (其他代码)

    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
```

你可以在 with 语句中使用定义了 `__enter__` 和 `__exit__` 方法的类的实例。

```python
with ManagedFile('hello.txt') as file:
    file.write('Hello, world!')
    file.write('Bye, world!')
```

- `ManagedFile` 类的 `__enter__` 方法打开一个文件并返回它。
- `file.write(...)` 代码在文件中写入一些内容。
- `__exit__` 方法关闭文件，即使写入文件时发生了异常也会执行此操作。

通过实现 `__enter__` 和 `__exit__` 方法，你可以定义自己的上下文管理器，进而优雅地管理资源和处理异常。

###### Pythonic

"Pythonic" 是一个广泛用于描述代码的术语，表示代码不仅仅是用Python语言编写的，而且是遵循Python的哲学和最佳实践编写的。一个Pythonic的代码通常会更易于阅读、理解和维护。这种代码通常也会更加简洁和优雅，并且充分利用了Python的特性和功能。

Python有一些核心的设计哲学，如在PEP 20中描述的"Zen of Python"（Python之禅）。其中包括了一些指导原则，如：

- 美胜于丑陋（Beautiful is better than ugly）
- 显式优于隐式（Explicit is better than implicit）
- 简单胜于复杂（Simple is better than complex）
- 复杂胜于复杂化（Complex is better than complicated）
- 可读性计数（Readability counts）

编写Pythonic代码通常包括以下几个方面：

1. **遵循PEP 8编码风格**：PEP 8是Python编程的样式指南，包括命名约定、代码布局和缩进等。

2. **使用Python的数据结构和函数**：充分利用Python的内置类型和函数，如列表推导式、生成器、装饰器等。

3. **编写自文档化的代码**：代码应该易于理解，注释应该是用来解释代码的“为什么”而不是“什么”。

4. **利用Python的特殊方法**：如上面提到的魔法方法，使得自定义对象能够更自然地融入Python的语言特性和数据模型。

5. **优先考虑代码的可读性和可维护性**：代码应该是易于他人阅读和维护的。

总之，Pythonic意味着编写的代码不仅符合Python的语法规则，而且符合Python的文化和习惯，使得代码更加清晰、简洁和优雅。

### 第四章-类与对象

<img src="pythonA/classobj.png" width="500" >

##### 1. 抽象基类(abc模块)

抽象基类（Abstract Base Classes，简称 ABCs）是Python中的一个概念，允许我们定义抽象方法和抽象类。一**个抽象类是不能被实例化的类，而一个抽象方法是在抽象类中必须由任何子类实现的方法**。`abc`模块为我们提供了工具来定义抽象基类和抽象方法。

**作用：**

1. 定义接口：ABCs允许我们定义一个类应该有哪些方法，而不需要实现它们。这为代码提供了明确的规范。
  
2. 强制实现：继承自抽象基类的子类必须实现所有的抽象方法，否则它们也将成为抽象类，并且不能被实例化。

**示例**:

1. 定义一个抽象基类表示所有的多边形，并要求子类实现计算面积的方法：

```python
from abc import ABC, abstractmethod

class Polygon(ABC):

    @abstractmethod
    def area(self):
        pass

class Rectangle(Polygon):
    def __init__(self, width, height):
        self.width = width
        self.height = height

    def area(self):
        return self.width * self.height

class Triangle(Polygon):
    def __init__(self, base, height):
        self.base = base
        self.height = height

    def area(self):
        return 0.5 * self.base * self.height

# 正常工作，因为子类实现了area()方法
rectangle = Rectangle(10, 5)
print(rectangle.area())

triangle = Triangle(10, 5)
print(triangle.area())

# 将导致错误，因为Polygon是抽象类，不能被实例化
# polygon = Polygon()
```

2. 定义一个抽象基类，表示所有的电子设备，并要求子类实现开启和关闭的方法：

```python
from abc import ABC, abstractmethod

class ElectronicDevice(ABC):

    @abstractmethod
    def turn_on(self):
        pass

    @abstractmethod
    def turn_off(self):
        pass

class Television(ElectronicDevice):

    def turn_on(self):
        print("Television is now on!")

    def turn_off(self):
        print("Television is now off!")

tv = Television()
tv.turn_on()
tv.turn_off()

# 将导致错误，因为ElectronicDevice是抽象类，不能被实例化
# device = ElectronicDevice()
```

以上示例中，如果你试图不实现`area()`或`turn_on()`和`turn_off()`方法，Python将会在运行时给出错误，因为这些方法是被标记为`@abstractmethod`的。这就确保了所有的子类都正确地实现了所需的方法。

##### 2. isinstance 用法

1. `isinstance`使用场景：

   `isinstance` 是一个内置函数，用于检查一个对象是否是指定类的实例，或是否是一个类的子类的实例。以下是一些常见的使用场景：

   - 类型检查：在函数或方法中，确保参数是正确的类型。
     ```python
     def add_numbers(a, b):
         if not isinstance(a, (int, float)) or not isinstance(b, (int, float)):
             raise ValueError("Inputs must be numbers")
         return a + b
     ```

   - 处理不同的数据类型：基于输入的数据类型执行不同的代码。
     
     ```python
     def process_data(data):
         if isinstance(data, list):
             # Handle list
         elif isinstance(data, dict):
             # Handle dictionary
     ```
     
   - 检查是否是子类的实例：与特定类或其子类的实例进行交互。
     
     ```python
     if isinstance(obj, BaseClass):
         # obj is an instance of BaseClass or a subclass of BaseClass
     ```

2. `is` 与 `==` 的区别：

   - `is`：检查两个引用是否指向内存中的同一个对象。也就是说，它比较的是对象的身份。
     
     ```python
     a = [1, 2, 3]
     b = a
     print(a is b)  # True，因为a和b指向同一个对象
     ```
     
   - `==`：检查两个对象的内容是否相等，基于它们的 `__eq__` 方法进行比较（如果定义了的话）。
     
     ```python
     a = [1, 2, 3]
     b = [1, 2, 3]
     print(a == b)  # True，因为a和b的内容相同
     print(a is b)  # False，因为a和b指向不同的对象
     ```
   
   因此，`is` 是检查两个变量是否引用同一对象，而 `==` 是检查两个对象的内容是否相等。在大多数情况下，我们更关心的是内容是否相等，所以更常使用 `==`。但在检查变量是否为 `None` 时，通常使用 `is`：
   ```python
   if variable is None:
       # Do something
   ```

##### 3. 类变量的作用

类变量（有时也被称为类属性或静态变量）是附加到类本身的变量，而不是类的具体实例。这意味着类变量在所有实例之间是共享的，而不是每个实例都有自己的拷贝。下面是类变量的一些特点和用途：

1. 共享值：因为类变量在所有实例之间共享，所以当一个实例修改类变量时，其他所有实例看到的值也会被修改。

2. 存储常量：类变量经常被用来定义一个类的常量设置。

3. 计数器功能：可以使用类变量来计算创建了多少个类的实例。

4. 默认值：类变量可以用作实例变量的默认值。

```python
class Car:
    # 1. 共享值：类变量，记录所有汽车的默认颜色
    default_color = "White"
    
    # 2. 存储常量：汽车的最大速度
    MAX_SPEED = 200
    
    # 3. 计数器功能：记录已经创建的汽车数量
    num_of_cars = 0
    
    def __init__(self, brand, speed):
        self.brand = brand
        # 4. 默认值：如果创建实例时没有指定颜色，将使用default_color作为默认值
        self.color = Car.default_color
        self.speed = speed if speed <= Car.MAX_SPEED else Car.MAX_SPEED
        Car.num_of_cars += 1

    # 方法：修改类变量default_color
    @classmethod
    def set_default_color(cls, new_color):
        cls.default_color = new_color


# 创建两辆车
car1 = Car("Toyota", 180)
car2 = Car("BMW", 210)

# 输出默认颜色和汽车数量
print(car1.color)  # 输出: White
print(Car.num_of_cars)  # 输出: 2

# 修改默认颜色
Car.set_default_color("Blue")

# 创建第三辆车
car3 = Car("Mercedes", 190)

# 查看第三辆车的颜色（已修改为新的默认颜色）
print(car3.color)  # 输出: Blue

# 查看三辆车的速度（注意：尽管BMW的速度被设置为210，但由于MAX_SPEED是200，所以速度被限制在200以内）
print(car1.speed)  # 输出: 180
print(car2.speed)  # 输出: 200
print(car3.speed)  # 输出: 190
```

##### 4. 实例方法，类方法，静态方法

实例方法、类方法和静态方法是 Python 中定义在类内部的三种方法，它们有不同的应用场景和调用方式。

1. 实例方法：
    
    - 定义：实例方法的第一个参数是`self`，它表示类的实例。
    - 调用：需要通过类的实例来调用。

    ```python
    class Person:
        def __init__(self, name):
            self.name = name
        
        # 实例方法
        def greet(self):
            print(f"Hello, my name is {self.name}.")
    
    p = Person("Alice")
    p.greet()  # 输出: Hello, my name is Alice.
    ```

2. 类方法：
    - 定义：类方法的第一个参数是`cls`，它表示类本身。使用`@classmethod`装饰器定义。
    - 调用：可以通过类或其实例来调用。
    
    ```python
    class Counter:
        count = 0
        
        @classmethod
        def increment(cls):
            cls.count += 1
        
        @classmethod
        def get_count(cls):
            return cls.count
    
    Counter.increment()
    print(Counter.get_count())  # 输出: 1
    ```
    
3. 静态方法：
    
    - 定义：静态方法没有特殊的第一个参数（如`self`或`cls`）。使用`@staticmethod`装饰器定义。
    - 调用：可以通过类或其实例来调用。
    - 场景：当方法不需要访问实例或类的属性，但仍与类相关并需要在类内部定义时。
    
    ```python
    class MathHelper:
        
        @staticmethod
        def add(x, y):
            return x + y
        
        @staticmethod
        def subtract(x, y):
            return x - y
    
    result = MathHelper.add(5, 3)
    print(result)  # 输出: 8
    ```

总结：
- 实例方法用于操作与特定实例相关的数据。
- 类方法用于操作与整个类相关的数据。
- 静态方法不与特定实例或类数据相关，但仍然是类的逻辑组成部分。

##### 5. 自省机制和动态特性

1.Python对象的动态性质

在许多编程语言中，对象的结构（属性和方法）在定义时是固定的，不能在运行时修改。但是Python的对象模型是动态的，这意味着您可以在运行时为对象添加、修改或删除属性和方法。

```python
class Person:
    def __init__(self, name):
        self.name = name

# 创建一个实例
p = Person("Alice")

# 动态添加一个新属性
p.age = 25

# 动态添加一个新方法
def say_hello(self):
    print(f"Hello, I'm {self.name} and I'm {self.age} years old.")
    
#`types.MethodType(say_hello, p)`：这将`say_hello`函数转换为`p`的实例方法。也就是说，我们现在可以像调用对象的任何其他方法一样来调用`say_hello`。
#`p.say_hello = ...`：这行代码将转换后的方法赋值给`p`的`say_hello`属性，从而为`p`对象添加了一个新的方法
import types
p.say_hello = types.MethodType(say_hello, p)
p.say_hello()  # 输出: Hello, I'm Alice and I'm 25 years old.

# 动态删除属性
del p.age
```

这行代码使用`del`语句删除了`p`对象的`age`属性。在这之后，试图访问`p.age`会引发一个`AttributeError`，因为该属性不再存在。 这段代码展示了Python的动态性，它允许我们在运行时为对象添加、修改或删除属性和方法。

2.自省机制

自省是指对象能够知道自己属性和方法的能力。Python中的自省机制允许您在运行时查询关于对象状态的信息。

Python提供了多种自省工具，如：
- `type()`: 返回对象的类型。
- `id()`: 返回对象的唯一标识符。
- `getattr()`, `setattr()`, `hasattr()`, `delattr()`: 获取、设置、检查和删除对象的属性。
- `isinstance()`: 检查对象是否是给定类型的实例。
- `dir()`: 返回对象的所有属性和方法的列表。

```python
class Sample:
    def __init__(self, value):
        self.value = value

    def show(self):
        print(self.value)

s = Sample(10)
print(dir(s))
```

这种能够在运行时查询和修改对象属性的能力使Python成为一种动态语言，为开发者提供了极大的灵活性。



##### 6. super用法

在Python 3中，`super()`函数常用于调用基类（父类）的方法。它的主要用途是在继承的情境下，尤其是多重继承的情境。以下是`super()`的常见应用场景：

1. 单继承 - 重写方法:
在单继承中，当子类需要重写父类的某个方法，但还希望在子类中保持父类该方法的某些功能时，通常会用到`super()`。

```python
class Par:
    def greet(self):
        print('hello i am your father')
    
class Clild(Par):
    def greet(self):
        super().greet()
        print('you are my son')
        
c = Clild()
c.greet()
```

2. 多继承 - MRO (Method Resolution Order):
在多重继承的场景中，`super()`用于确保所有基类的方法都得以调用，并遵循MRO。

```python
class Base:
    def greet(self):
        print("Hello from Base")

class A(Base):
    def greet(self):
        print("Hello from A")
        super().greet()

class B(Base):
    def greet(self):
        print("Hello from B")
        super().greet()

class Child(A, B):
    def greet(self):
        print("Hello from Child")
        super().greet()

c = Child()
c.greet()  # 输出顺序是：Child -> A -> B -> Base
```

在上面的代码中，`Child`类从`A`和`B`类继承，因此当调用`c.greet()`时，方法解析顺序确保了`A`的`greet`、`B`的`greet`和`Base`的`greet`都会被按照正确的顺序调用。

3. 使用super()初始化父类: 
  当子类需要继承父类的属性时，常使用`super()`在子类的`__init__`方法中初始化父类。

  ```python
  class Parent:
      def __init__(self, name, age):
          self.name = name
          self.age = age
  
  class Child(Parent):
      def __init__(self, name, age, grade):
          super().__init__(name, age)  # 使用super()初始化父类的属性
          self.grade = grade
  
  c = Child("Tom", 10, "Fifth")
  print(c.name)  # 输出: Tom
  print(c.age)  # 输出: 10
  print(c.grade)  # 输出: Fifth
  ```

  在上述代码中，`Child`类是`Parent`类的子类，并添加了额外的`grade`属性。我们使用`super().__init__(name, age)`来确保`Parent`类的`__init__`方法被正确调用，从而初始化`name`和`age`属性。 这种使用`super()`在子类中初始化父类的模式，确保子类不仅继承了父类的所有方法，而且还继承了父类的所有属性，从而提供了一个稳健的继承体系。

##### 7.contextlib用法

`contextlib`模块提供了用于与`with`语句一起工作的实用工具，使开发者能够更轻松地编写资源管理代码。

以下是一个常用的`contextlib`模块的功能：`contextmanager`装饰器。这个装饰器可以将生成器转换为上下文管理器，从而简化上下文管理器的创建。

```python
import time
from contextlib import contextmanager

@contextmanager
def measure_time():
    start_time = time.time()
    yield
    end_time = time.time()
    print(f"Elapsed time: {end_time - start_time:.2f} seconds")

# 使用示例
with measure_time():
    # 一些耗时的操作
    result = sum(i for i in range(10000000))
    print(result)
```

当进入`with`块时，`measure_time`上下文管理器的前半部分（`yield`之前的部分）会执行。当退出`with`块时，`yield`之后的部分会执行，从而完成资源的清理或其他必要操作。在这个例子中，我们使用上下文管理器来测量代码块的执行时间，从而无需显式地调用开始和结束的时间戳。

### 第五章-自定义序列类

##### 5.1 序列类型的分类

1. **容器序列**：这些序列可以存储不同类型的数据。

   - `list`：一个可变的、有序的数据集合。
     
     ```python
     lst = [1, 2, "hello", 3.14]
     lst.append("world")
     print(lst)
     ```
     
   - `tuple`：一个不可变的、有序的数据集合。
     
     ```python
     t = (1, 2, "hello")
     print(t[2])
     ```
     
   - `deque`：双端队列，可以在队列的两端进行插入和删除操作。
     
     ```python
     from collections import deque
     dq = deque([1, 2, 3])
     dq.appendleft(0)
     dq.append(4)
     print(dq)
     ```

2. **扁平序列**：这些序列只能容纳单一类型的数据。

   - `str`：字符串，用于存储文本信息。
     
     ```python
     s = "Hello, world!"
     print(s.lower())
     ```
     
   - `bytes`：一个不可变的字节序列。
     ```python
     b = b"hello"
     print(b[0])
     ```

   - `bytearray`：一个可变的字节序列。
     
     ```python
     ba = bytearray(b"hello")
     ba[0] = 72
     print(ba)
     ```
     
   - `array`：数组，存储同一类型的数据。
     
     ```python
     import array
     arr = array.array('i', [1, 2, 3])
     arr.append(4)
     print(arr)
     ```

3. **可变序列**：这些序列的内容可以修改。

   请参考上面的`list`、`deque`和`bytearray`的示例。

4. **不可变**：这些序列的内容一旦定义就不能更改。

   请参考上面的`tuple`、`str`和`bytes`的示例。

这四种分类代表了Python中数据结构的不同特点。以下是每个分类的特性以及它们之间的主要区别：

1. **容器序列** (`list`, `tuple`, `deque`)
   - 可以存储不同类型的元素。例如，它们可以同时存储整数、字符串、其他对象等。
   - 存储的是元素的引用（或指针），而不是直接存储元素本身。

2. **扁平序列** (`str`, `bytes`, `bytearray`, `array.array`)
   - 只能存储单一类型的元素。例如，`str`只能存储字符，`bytes`只能存储字节。
   - 存储的是值本身，而不是引用。因此，它们在内存中是连续的，并且通常更加紧凑。

3. **可变序列** (`list`, `deque`, `bytearray`, `array`)
   - 元素或内容可以在创建后进行修改。例如，您可以向列表中添加、删除或更改元素。
   - 适合那些需要动态调整或变化的数据结构。

4. **不可变序列** (`tuple`, `str`, `bytes`)
   - 一旦定义，其内容就不能更改。
   - 这些不可变的特性使它们在某些情况下更加安全和高效（例如作为字典的键）。

简而言之，主要的区别在于：
- 容器与扁平：是根据存储的元素类型（多种或单一）以及存储方式（引用还是值本身）来区分的。
- 可变与不可变：是根据元素是否可以在创建后更改来区分的。

##### 5.2 序列的abc继承关系

在 Python 中，`collections.abc` 模块提供了一系列抽象基类 (ABCs, Abstract Base Classes) 来为容器类型和其他与容器相关的类型定义形式化的接口。对于序列，`collections.abc` 模块提供了多个 ABCs，如 `Iterable`, `Container`, `Sized`, `Sequence` 等，这些可以帮助定义和判断对象是否满足某种序列的接口。

以下是序列相关的 ABCs 和它们之间的继承关系：

1. `Iterable`: 所有迭代对象的基本接口，定义了 `__iter__()` 方法。

2. `Container`: 定义了 `__contains__()` 方法，用于判断元素是否在容器中。

3. `Sized`: 定义了 `__len__()` 方法，返回容器中的元素数量。

4. `Sequence`: 继承自 `Iterable`, `Container`, 和 `Sized`。它定义了序列的核心接口，如索引 (`__getitem__()`), `count()`, 和 `index()`。所有满足序列接口的对象应该直接或间接地继承自这个 ABC。

5. `MutableSequence`: 继承自 `Sequence`。除了所有 `Sequence` 的方法外，还增加了修改序列的方法，如 `__setitem__()`, `__delitem__()`, `insert()`, `append()`, `reverse()` 等。这表示一个可变的序列。

6. `Reversible`: 定义了 `__reversed__()` 方法，允许对象以反向顺序进行迭代。

例如，`list` 是一个 `MutableSequence`，因为你可以修改它（添加、删除、修改元素）。而 `str` 和 `tuple` 则是 `Sequence`，因为它们是不可变的。

要注意的是，不需要直接继承这些 ABCs 来被认为是序列或迭代器。只要对象正确实现了相应的方法（鸭子类型），`isinstance()` 检查就会认为它是对应的类型。例如，只要对象实现了 `__iter__()` 方法，它就被认为是一个 `Iterable`。

在实际的编码过程中，这些 ABCs 对于确定一个给定的类是否满足某个接口，以及为自定义的容器类型提供标准接口非常有用。

##### 5.3 列外的+, +=和extend的区别

序列中的 `+`, `+=` 和 `extend()` 方法都与序列的连接和扩展有关，但它们的工作方式和使用场景有所不同。以下是它们之间的主要区别：

1. `+` (连接)
    
    - `+` 用于连接两个相同类型的序列，例如两个列表或两个字符串。
    
    - 它不会修改原始序列，而是返回一个新的序列。
    
      ```python
      list1 = [1, 2, 3]
      list2 = [4, 5, 6]
      result = list1 + list2
      print(result)  # 输出：[1, 2, 3, 4, 5, 6]
      ```
    
2. `+=` (原地增加)
    
    - 对于可变序列，如列表，`+=` 会原地修改序列。
    
    - 它等效于使用 `extend()` 方法对列表进行扩展。
    
      ```python
      list1 = [1, 2, 3]
      list1 += [4, 5, 6]
      print(list1)  # 输出：[1, 2, 3, 4, 5, 6]
      ```
    
3. `extend()` (扩展)
    
    - `extend()` 是列表的一个方法，用于在列表的末尾添加另一个序列的元素。
    
    - 它会修改原始列表。
    
    - 与 `+=` 不同，`extend()` 只适用于列表，而 `+=` 可用于所有支持原地增加的序列。
    
      ```python
      list1 = [1, 2, 3]
      list1.extend([4, 5, 6])
      print(list1)  # 输出：[1, 2, 3, 4, 5, 6]
      ```

小结:

- `+` 用于连接，返回一个新的序列。
- `+=` 和 `extend()` 用于原地修改可变序列。
- `+=` 可用于所有支持原地增加的序列，而 `extend()` 仅适用于列表。

另外，使用 `+` 连接大量序列可能会导致效率问题，因为每次连接都会创建一个新的序列。在这种情况下，使用 `+=` 或 `extend()` 可能更为高效，因为它们原地修改序列，不需要创建新的对象。

##### 5.4 实现可切片的对象

记住一个公式： [start: end: step]

##### 5.5 bisect管理可排序序列

`bisect` 是 Python 标准库中的一个模块，用于处理已排序的序列。它提供了二分查找算法来快速找到一个元素在排序序列中的位置或应该插入的位置。这对于大量数据的搜索和插入操作非常有用，因为它可以显著提高效率。

1. `bisect_left(a, x, lo=0, hi=len(a))`:

    - 返回 `x` 在 `a` 中应该插入的位置，以保持排序顺序。如果 `x` 已经存在于 `a` 中，返回 `x` 左边的位置。

      ```python
      import bisect
      numbers = [1, 3, 4, 4, 6, 8]
      position = bisect.bisect_left(numbers, 4)
      print(position)  # 输出：2
      ```

2. `bisect_right(a, x, lo=0, hi=len(a))` 或 `bisect(a, x, lo=0, hi=len(a))`:

    - 返回 `x` 在 `a` 中应该插入的位置，以保持排序顺序。如果 `x` 已经存在于 `a` 中，返回 `x` 右边的位置。

      ```python
      import bisect
      numbers = [1, 3, 4, 4, 6, 8]
      position = bisect.bisect_right(numbers, 4)
      print(position)  # 输出：4
      ```

3. `insort_left(a, x, lo=0, hi=len(a))`:
    
    - 将 `x` 插入到 `a` 中，使其保持排序顺序。如果 `x` 已经存在于 `a` 中，将其插入在 `x` 左边的位置。
    
      ```python
      import bisect
      numbers = [1, 3, 4, 4, 6, 8]
      bisect.insort_left(numbers, 5)
      print(numbers)  # 输出：[1, 3, 4, 4, 5, 6, 8]
      ```
    
4. `insort_right(a, x, lo=0, hi=len(a))` 或 `insort(a, x, lo=0, hi=len(a))`:

    - 将 `x` 插入到 `a` 中，使其保持排序顺序。如果 `x` 已经存在于 `a` 中，将其插入在 `x` 右边的位置。

      ```python
      import bisect
      numbers = [1, 3, 4, 4, 6, 8]
      bisect.insort_right(numbers, 5)
      print(numbers)  # 输出：[1, 3, 4, 4, 5, 6, 8]
      ```

这些函数都接受可选的 `lo` 和 `hi` 参数，用于指定在序列中进行搜索的子范围。

##### 5.6 array和deque

`array` 和 `deque` 都是 Python 的内置模块，用于存储数据。但它们的特性和日常用途有所不同。

1. array:
   `array` 模块提供了一个 `array()` 对象，这是一个紧凑的、可变的、序列数据结构，它存储的是基于数组的元素（如整数或浮点数），而不是普通的 Python 列表中的元素。

   日常开发中的用法:
   - 当需要一个只包含数字的列表，并且想要更加节省存储空间时。

   ```python
   #使用 `array` 可以帮助节省存储空间，特别是与普通的 Python 列表相比
   from array import array
   import sys
   
   # 创建一个普通的 Python 列表
   normal_list = [i for i in range(100000)]
   print("Size of normal list:", sys.getsizeof(normal_list))  # 输出普通列表的大小
   
   # 创建一个整数数组
   int_array = array('i', [i for i in range(100000)])
   print("Size of array:", sys.getsizeof(int_array))  # 输出整数数组的大小，你会发现 `array` 的大小通常小于普通的 Python 列表
   ```

   - 与文件 I/O 一起使用，例如，直接从二进制文件中读取或写入数字数组。

   ```python
   from array import array
   
   # 创建一个整数数组
   numbers = array('i', [1, 2, 3, 4, 5])
   
   # 将数组写入二进制文件
   with open("data.bin", "wb") as file:
       numbers.tofile(file)
   
   # 从二进制文件中读取数组
   read_numbers = array('i')  # 创建一个空的整数数组
   with open("data.bin", "rb") as file:
       read_numbers.fromfile(file, len(numbers))
   
   print(read_numbers)  # 输出：array('i', [1, 2, 3, 4, 5])
   ```

   `list` 和 `array` 是 Python 中的两种不同的数据结构，它们之间有几个主要的区别：

   1. 数据类型的均一性:
       - `list`: 可以包含不同类型的元素，例如整数、字符串、对象等。
       - `array`: 只能包含单一类型的元素，例如只有整数或只有浮点数。

   2. 存储效率和性能:
       - `list`: 由于它是一种通用数据结构，它需要存储额外的信息以支持混合类型的元素，这导致它占用更多的内存。
       - `array`: 由于它存储的是均一的数据类型，它可以紧凑地存储数据，从而减少内存使用。此外，由于不需要额外的类型检查，某些操作可能更快。

   3. 灵活性:
       - `list`: 更为灵活，可以轻松地增加、删除和修改元素。
       - `array`: 在某些操作上可能不如 `list` 灵活，特别是当涉及到改变数组大小或类型时。

   4. 用途:
       - `list`: 适用于标准的序列操作，如插入、删除、迭代等。
       - `array`: 适用于更为特定的任务，如数值计算、文件 I/O 操作或其他需要均一数据类型的场合。

   为什么 `list` 效率低，占用空间大：

   - **动态类型**：Python 中的 `list` 是动态类型的，这意味着每个元素都是一个对象，包含了类型信息、引用计数等。而 `array` 只存储相同类型的数据，不需要为每个元素存储这些额外的信息。
     
   - **内存分配**：为了支持快速增加元素，`list` 在背后可能会预分配额外的内存。这意味着 `list` 的实际内存消耗可能大于它当前所包含的元素所需的内存。
     
   - **指针/引用存储**：`list` 存储的实际上是指向对象的指针，而不是直接存储值，这导致了额外的内存开销。而 `array` 直接存储值。

2. deque:
   `deque` 是 "double-ended queue" 的缩写，代表双端队列。它允许你在队列的两端都进行添加和删除操作。

   日常开发中的用法:

   - 快速地在头部和尾部进行添加和删除操作

   ```python
   from collections import deque
   
   # 创建一个双端队列
   dq = deque([1, 2, 3, 4, 5])
   
   # 在头部和尾部添加元素
   dq.appendleft(0)
   dq.append(6)
   
   # 在头部和尾部删除元素
   dq.popleft()  # 删除并返回 0
   dq.pop()      # 删除并返回 6
   ```

​	当你需要一个可以快速地在头部和尾部进行添加和删除操作的数据结构时，`deque`（双端队列）是一个非常好的选择。它是从 `collections` 模块中导入的。

实现队列和栈的数据结构

队列:

```python
queue = deque()

# 入队操作
queue.append("first")
queue.append("second")
queue.append("third")

# 出队操作
print(queue.popleft())  # 输出: first
```

栈:

```python
stack = deque()

# 压栈操作
stack.append("bottom")
stack.append("middle")
stack.append("top")

# 弹栈操作
print(stack.pop())  # 输出: top
```

-  实现滑动窗口或者轮询等算法:

  滑动窗口:

  ```python
  from collections import deque
  
  def moving_average(sequence, n=3):
      # 初始化一个迭代器，用于遍历输入的序列
      it = iter(sequence)
      
      # 初始化滑动窗口。使用生成器表达式计算序列的前n个元素的和，并将其除以n得到平均值。
      # 这是我们的初始滑动窗口的值。
      total = sum(next(it) for _ in range(n))
      
      # 创建一个滑动窗口deque，并将前n个元素添加进去
      d = deque(sequence[i] for i in range(n))
      
      # 产出初始窗口的平均值
      yield total / n
      
      # 使用迭代器it遍历序列的剩余部分
      for elem in it:
          # 将新元素添加到deque的右侧
          d.append(elem)
          
          # 将deque的左侧元素（即离开窗口的元素）从总和中减去
          total -= d.popleft()
          
          # 把新元素加到总和中
          total += elem
          
          # 产出新窗口的平均值
          yield total / n
  
  # 输入序列
  sequence = [1, 2, 3, 4, 5, 6, 7, 8, 9]
  # 调用函数并打印结果
  print(list(moving_average(sequence)))
  ```

  轮询算法:

  ```python
  #这是一个简单的轮询调度器的例子
  tasks = deque(["task1", "task2", "task3", "task4"])
  
  while tasks:
      task = tasks.popleft()
      # ...处理任务...
      print(f"Handling {task}")
      # ...假设每次我们处理完一个任务后，我们再次将其放回队列末尾...
      tasks.append(task)
  ```

  **注意**：上面的轮询示例会产生无限循环，因为我们始终将任务放回队列末尾。在实际应用中，你可能需要某种退出条件或计数器来确定何时停止轮询。

结论:

- 如果你需要一个简单、紧凑且只包含基本数据类型的数据结构，那么 `array` 可能是一个好选择。
- 如果你需要一个可以从两端快速添加和删除元素的动态数据结构，那么 `deque` 是更好的选择。

##### 5.7 列表推导式、生成器表达式、字典推导式

推导式（comprehensions）是Python中的一个强大功能，它们提供了一种简洁的方式来创建列表、字典或集合。与此同时，生成器表达式允许你创建一个生成器，这是一种更为内存友好的数据结构。

1. 列表推导式 (List Comprehensions):
   
   列表推导式为创建列表提供了一种简明的方法。

   ```python
   # 使用列表推导式从0到9获取每个数的平方
   squares = [x**2 for x in range(10)]
   print(squares)  # 输出: [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
   ```

   还可以在推导式中添加条件判断：

   ```python
   # 使用列表推导式从0到9获取每个偶数的平方
   even_squares = [x**2 for x in range(10) if x % 2 == 0]
   print(even_squares)  # 输出: [0, 4, 16, 36, 64]
   ```

2. 生成器表达式 (Generator Expressions):
   
   生成器表达式与列表推导式语法相似，但使用圆括号而不是方括号。它返回一个生成器对象，该对象可以逐个产生值，从而节省内存。

   ```python
   # 使用生成器表达式从0到9获取每个数的平方
   gen_squares = (x**2 for x in range(10))
   
   for square in gen_squares:
       print(square)
   ```

3. 字典推导式 (Dictionary Comprehensions):
   
   字典推导式可以用来直接生成字典

   ```python
   # 创建一个字典，其中键是0到9之间的数字，值是其平方
   square_dict = {x: x**2 for x in range(10)}
   print(square_dict)  # 输出: {0: 0, 1: 1, 2: 4, 3: 9, ...}
   ```

   可以像列表推导式一样添加条件判断：

   ```python
   # 创建一个字典，其中键是0到9之间的偶数，值是其平方
   even_square_dict = {x: x**2 for x in range(10) if x % 2 == 0}
   print(even_square_dict)  # 输出: {0: 0, 2: 4, 4: 16, 6: 36, 8: 64}
   ```

日常开发中的使用:

- 列表推导式通常用于根据现有列表创建新列表，例如进行数据转换或过滤。
  
- 生成器表达式非常适合处理大数据流或可能无法一次性放入内存的数据集，因为它是延迟计算的并且每次只产生一个值。
  
- 字典推导式用于从现有数据结构创建字典，例如，根据两个相关的列表创建一个映射关系。

### 第六章-SET and DICT

<img src="pythonA/dict-set.png" width="500" >

##### 6.1 dict与abc继承关系

`dict`（字典）在Python中是一个内置的数据类型，用于存储键值对。在Python的`collections.abc`模块中，有许多抽象基类 (ABCs, Abstract Base Classes) 用于定义容器类应该具有的接口和行为。这些抽象基类为容器类型定义了一组共同的方法，并且可以用来检查某个类是否满足特定的接口。

关于`dict`和`collections.abc`之间的关系，以下是一些要点：

1. `collections.abc.Mapping`:
   - `dict` 是 `collections.abc.Mapping` 的一个具体实现。
   - `Mapping` 定义了键值对容器应该有的基本方法，如 `__getitem__`, `__iter__`, `__len__`, `keys`, `items`, `values` 等。
   - 所有满足这些基本方法的键值对容器都可以被认为是 `Mapping`，包括内置的 `dict`。

2. `collections.abc.MutableMapping`:
   - `dict` 还实现了 `collections.abc.MutableMapping` 接口。
   - `MutableMapping` 继承自 `Mapping`，并添加了一些修改映射内容的方法，如 `__setitem__`, `__delitem__`, `pop`, `popitem`, `clear` 等。
   - `dict` 作为一个可变的映射，提供了这些方法的具体实现。

3. 其他ABCs:
   - 还有一些其他的ABCs在`collections.abc`中定义，但`dict`主要与`Mapping`和`MutableMapping`相关。

在实践中，你可以使用 `isinstance` 函数来检查一个对象是否是某个ABC的实例。例如，检查一个对象是否是一个映射（不仅仅是`dict`，还可以是其他实现了映射接口的对象）：

```python
from collections.abc import Mapping

d = {}
print(isinstance(d, Mapping))  # 输出: True
```

总之，`dict` 是 `collections.abc` 中 `Mapping` 和 `MutableMapping` 抽象基类的一个具体实现，并提供了这些基类定义的所有方法的具体实现。

##### 6.2 `dict`的常见用法:

- `setdefault`: 这是一个`dict`的方法，用于获取一个键的值，如果该键不存在于字典中，则返回一个默认值，并同时设置这个键到字典中。

- `defaultdict`是Python的`collections`模块中提供的一个子类，它继承自Python的内建`dict` 类。其主要特点是它可以为字典提供一个默认值，这样在访问不存在的键时，它会自动为这个键生成一个默认值，而不是抛出 `KeyError`异常。

  下面是`defaultdict`的一些基本用法：

  1. 创建defaultdict

  首先，你需要从`collections`模块导入`defaultdict`。

  ```python
  from collections import defaultdict
  ```

  当你创建一个`defaultdict`时，你需要提供一个返回默认值的函数。常见的用法是使用内建的`list`、`int`或`set`函数。

  2. 使用`list`作为默认值

  这在你需要一个列表来存储某个键的多个值时很有用。

  ```python
  dd_list = defaultdict(list)
  dd_list["a"].append(1)
  dd_list["a"].append(2)
  dd_list["b"].append(4)
  print(dd_list)  # defaultdict(<class 'list'>, {'a': [1, 2], 'b': [4]})
  ```

  3. 使用`int`作为默认值

  这在你需要计数或累加某个键的值时很有用。

  ```python
  dd_int = defaultdict(int)
  words = ["apple", "banana", "apple", "orange", "banana"]
  for word in words:
      dd_int[word] += 1
  print(dd_int)  # defaultdict(<class 'int'>, {'apple': 2, 'banana': 2, 'orange': 1})
  ```

  4. 使用`set` 作为默认值

  这在你需要存储某个键的唯一值时很有用。

  ```python
  dd_set = defaultdict(set)
  dd_set["a"].add(1)
  dd_set["a"].add(2)
  dd_set["a"].add(1)
  print(dd_set)  # defaultdict(<class 'set'>, {'a': {1, 2}})
  ```

  ### 注意事项

  - 默认值的生成是通过传递给`defaultdict`的函数进行的，该函数不接受任何参数并返回期望的默认值。
  - 当你尝试访问不存在的键时，该函数会被调用并为该键生成一个默认值。这也意味着每次访问不存在的键时，字典的大小都会增加。

  `defaultdict`在处理复杂数据结构或需要默认值的场景时特别有用，例如在图算法或文本处理中。

- `_missing_`方法: 这是`dict`的一个魔术方法，当使用`defaultdict`或自定义的字典类时，这个方法可以定义对不存在的键的处理方式。

`dict`（字典）是Python中的一个核心数据类型，它用于存储键-值对。字典的键是唯一的，而值可以是任意类型。以下是`dict`在日常开发中的一些常见用法：

1. **基本操作**：
   
   - 创建字典：
     ```python
     my_dict = {"a": 1, "b": 2, "c": 3}
     ```
   - 获取值：
     ```python
     value = my_dict["a"]
     ```
   - 设置值：
     ```python
     my_dict["d"] = 4
     ```
   
2. **使用`get`方法安全地获取值**：避免直接访问字典键导致的KeyError。
   
   ```python
   value = my_dict.get("e", default_value)
   ```
   
3. **遍历字典**：
   - 遍历键：
     ```python
     for key in my_dict:
         print(key)
     ```
   - 遍历键-值对：
     ```python
     for key, value in my_dict.items():
         print(key, value)
     ```

4. **使用字典推导式**：轻松创建或转换字典。
   
   ```python
   squared = {k: v*v for k, v in my_dict.items()}
   ```
   
5. **删除键-值对**：
   ```python
   del my_dict["a"]
   ```

6. **合并两个字典**：
   ```python
   dict1 = {"a": 1, "b": 2}
   dict2 = {"b": 3, "c": 4}
   merged_dict = {**dict1, **dict2}  # {'a': 1, 'b': 3, 'c': 4}
   ```

7. **使用`setdefault`**：如果键不存在于字典中，设置其默认值。
   ```python
   my_dict.setdefault("e", 5)
   ```

8. **字典作为缓存（Memoization）**：使用字典存储计算结果，避免重复计算。
   ```python
   def fibonacci(n, memo={}):
       if n in memo:
           return memo[n]
       if n <= 1:
           return n
       result = fibonacci(n-1, memo) + fibonacci(n-2, memo)
       memo[n] = result
       return result
   ```

9. **组织数据**：将相关数据组织为字典，例如存储用户信息：
   
   ```python
   user = {"id": 123, "name": "Alice", "email": "alice@example.com"}
   ```
   
10. **默认字典（`defaultdict`）**：自动为不存在的键提供默认值。
   ```python
   from collections import defaultdict
   count = defaultdict(int)
   for letter in "banana":
       count[letter] += 1
   ```

11. **使用`Counter`计数**：快速统计元素的出现次数。
   ```python
   from collections import Counter
   letter_count = Counter("banana")
   ```

##### 6.3 `dict`的变体:

- **`Counter`**: 这是`collections`模块中的一个类，用于计算元素的数量。它是`dict`的一个子类，其中键是元素，值是元素的数量。
- 不要在字典的键和集合中使用可变类型：这是一个常见的建议，因为字典的键和集合的元素必须是不可变的。
- `update`方法: 这是`dict`的一个方法，用于将一个字典或键值对的可迭代对象合并到当前字典中。

##### 6.4 `set`和`frozenset`:

`set`和`frozenset`是Python中用来表示无序且不包含重复元素的集合类型。它们在很多应用场景中都非常有用。

###### set
1. **去重**：由于集合不允许重复的元素，所以它常被用来从列表或其他序列类型中删除重复的元素。
   
   ```python
   nums = [1, 2, 2, 3, 4, 4, 4]
   unique_nums = set(nums)
   ```
   
2. **集合运算**：集合支持多种集合运算，如并集、交集、差集等。
   
   ```python
   a = {1, 2, 3}
   b = {3, 4, 5}
   union_set = a | b
   intersection_set = a & b
   difference_set = a - b
   ```
   
3. **成员检查**：集合的成员检查操作比列表更快，特别是在数据量大的情况下。
   
   ```python
   if value in my_set:
       # Do something
   ```
   
4. **不可变数据类型的容器**：由于集合本身是可变的，它不能被用作其他集合的元素或字典的键。但是，你可以将其他不可变类型放入集合中。

5. **数据筛选**：结合条件表达式，使用集合可以轻松地筛选数据。
   
   ```python
   filtered_data = {x for x in data if x.condition()}
   ```

###### frozenset
1. **字典的键**：由于`frozenset`是不可变的，它可以被用作字典的键。
   
   ```python
   my_dict = {frozenset([1, 2, 3]): "value"}
   ```
   
2. **集合中的集合**：如果你需要一个集合，它的元素也是集合，那么这些元素集合必须是`frozenset`，因为`set`是可变的，不能被放入其他集合中。
   
   ```python
   set_of_sets = {frozenset([1, 2]), frozenset([3, 4])}
   ```
   
3. **不变的集合数据**：在需要确保集合数据不被修改的场景中，`frozenset`是一个很好的选择。

4. **集合运算**：与`set`相同，`frozenset`也支持集合运算，但任何修改操作（如`add`或`remove`）都是不允许的。

在日常开发中，`set`可能会更常见，因为它是可变的，更加灵活。但在需要不可变集合或要用集合作为字典键等特殊情况下，`frozenset`会非常有用。

### 第七章-对象引用，可变和垃圾回收

##### 7.1 python变量的本质

在Python中，变量可以被理解为一个指向对象的引用或标签。不同于一些其他的编程语言，Python中的变量不直接存储值，而是存储对一个对象的引用。

以下是Python变量的几个关键理解点：

1. 变量是引用，不是容器

当你在Python中创建一个变量并为其分配一个值，你实际上是在创建一个对象，并让变量指向该对象。例如：

```python
a = [1, 2, 3]
```

这里，一个列表对象`[1, 2, 3]`被创建，变量`a`是指向这个列表对象的引用。

2. 变量的赋值与对象的身份

当你对一个变量进行赋值操作，例如`b = a`，你并没有复制对象，而只是创建了一个新的引用`b`，它指向与`a`相同的对象。

```python
a = [1, 2, 3]
b = a
```

此时，`a`和`b`都指向同一个列表对象。

3. 可变对象 vs. 不可变对象

- **可变对象**：这些对象的内容（或称为状态）可以在创建后被改变。常见的可变对象有：列表、字典、集合等。
  
  ```python
  lst = [1, 2, 3]
  lst.append(4)  # lst 现在是 [1, 2, 3, 4]
  ```

- **不可变对象**：这些对象的内容在创建后不能被改变。常见的不可变对象有：整数、浮点数、字符串、元组等。

  ```python
  s = "hello"
  # s[0] = 'H'  # 这会抛出一个错误，因为字符串是不可变的
  ```

4. 对象的引用计数

Python使用引用计数作为垃圾回收的一部分。当一个对象的引用计数降为0时，该对象的内存就可以被释放。

```python
x = [1, 2, 3]
y = x  # x 和 y 指向同一个对象
del x  # 删除 x 的引用
# 此时对象 [1, 2, 3] 的引用计数不为 0，因为 y 仍然指向它
```

5. 变量和对象的生命周期

当一个变量的引用被重新赋值，原来的对象（如果没有其他引用指向它）的引用计数就可能变为0，从而成为垃圾回收的目标。

```python
x = [1, 2, 3]
x = "hello"
# 列表 [1, 2, 3] 现在没有任何引用指向它，它可以被垃圾回收
```

##### 7.2 == 与 is区别

`==` 和 `is` 在Python中都是比较操作符，但它们用于比较的目的和方式是不同的。以下是它们之间的主要区别：

1. `==`：值比较

- `==` 用于比较两个对象的值是否相等。
- 当两个对象的内容或数据相等时，即使它们在内存中是不同的对象，`==` 仍然会返回 `True`。

示例：
```python
list1 = [1, 2, 3]
list2 = [1, 2, 3]
print(list1 == list2)  # 输出 True，因为两个列表的内容相同
print(list1 is list2)  # False
```

2. `is`：身份比较

- `is` 用于比较两个对象是否是同一个对象（即，它们是否指向内存中的同一个位置）。
- 当两个变量指向同一个对象时，`is` 返回 `True`。

示例：
```python
list1 = [1, 2, 3]
list2 = list1  # list2 和 list1 指向同一个对象
print(list1 is list2)  # 输出 True
```

但在这种情况下：
```python
list1 = [1, 2, 3]
list2 = [1, 2, 3]
print(list1 is list2)  # 输出 False，因为 list1 和 list2 指向内存中的两个不同位置
```

特别注意：

对于一些小的整数和字符串，Python为了优化性能会缓存这些对象。因此，使用 `is` 来比较这些对象可能会产生意外的结果。

示例：
```python
x = 10
y = 10
print(x is y)  # 输出 True，因为 10 是一个小整数，被缓存

s1 = "hello"
s2 = "hello"
print(s1 is s2)  # 输出 True，因为 "hello" 字符串被缓存
```

总结：
- 使用 `==` 当你想要比较两个对象的值。
- 使用 `is` 当你想要检查两个引用是否指向同一个对象。

##### 7.3 del 和 垃圾回收

当我们在Python中创建对象（如整数、字符串、列表、字典、类的实例等），它们会在内存中占据一定的空间。为了管理这些内存空间，Python提供了自动的内存管理机制，其中包括垃圾回收机制来自动回收不再使用的内存。

1. `del` 语句：

- `del` 是Python提供的一个语句，用于删除对象的引用。
- 当你使用 `del` 删除一个变量引用后，该变量不再存在，尝试访问它会导致NameError。
- 但是，仅仅删除一个对象的引用并不意味着该对象的内存会被立即释放。实际上，只有当一个对象的引用计数降为0时，它的内存才有可能被回收。

```python
x = [1, 2, 3]
del x  # 删除 x 引用
# print(x)  # 这将抛出 NameError，因为 x 已经被删除了
```

2. 垃圾回收：

引用计数:

- Python内部使用引用计数机制来跟踪每个对象被引用的次数。
- 当一个对象的引用计数降为0时，意味着这个对象不再被任何变量或其他对象引用，因此它的内存可以被释放。

循环引用问题:

- 但是，有些时候可能会出现循环引用的情况，即对象A引用对象B，对象B引用对象A。这种情况下，即使没有其他引用它们，它们的引用计数也不会是0。
- 为了解决这个问题，Python提供了一个垃圾回收器来检测这种循环引用并释放相关内存。

```python
# 循环引用示例
a = []
b = [a]
a.append(b)
# 在这里，a 和 b 形成了循环引用
```

`gc` 模块:

- Python的 `gc` (Garbage Collector) 模块提供了与垃圾回收机制交互的功能。
- 默认情况下，垃圾回收是启用的，但你可以通过 `gc` 模块来手动控制它，例如手动运行垃圾回收或禁用它。

总结:

- 使用 `del` 可以删除对象的引用，但并不保证立即释放对象的内存。
- 当一个对象的所有引用都被删除时，该对象的内存可能会被自动回收。
- Python的垃圾回收机制确保了循环引用的对象也可以被正确地回收。

### 第八章-元编程

##### 8.1 property动态属性

`property` 是一个Python内置的装饰器，用于**将方法转换为属性**，从而允许用户以属性的方式访问对象的数据，同时可以提供数据的读取、设置、删除操作的自定义实现。

```python
class MyClass:
    def __init__(self, value):
        self._value = value

    @property
    def value(self):
        return self._value

    @value.setter
    def value(self, new_value):
        self._value = new_value
```

`property`是一个常用的Python内置装饰器，用于将方法转化为属性。通过`property`，开发者可以创建只读属性，或者在设置属性值时添加额外的逻辑，例如数据验证。它也是描述符的一种简单形式。

1. 基础用法 - 创建只读属性

```python
class Circle:
    def __init__(self, radius):
        self._radius = radius

    @property
    def radius(self):
        return self._radius

    @property
    def diameter(self):
        return self._radius * 2

circle = Circle(5)
print(circle.radius)  # 输出: 5
print(circle.diameter)  # 输出: 10
#在这个示例中，我们使用`property`为`Circle`类创建了一个只读的`diameter`属性。
```
2. 数据验证

```python
class Student:
    def __init__(self, name, age):
        self._name = name
        self.age = age  # 使用setter设置

    @property
    def age(self):
        return self._age

    @age.setter
    def age(self, value):
        if not (0 <= value <= 150):
            raise ValueError("Invalid age")
        self._age = value

student = Student("Alice", 20)
student.age = 25  # 正确
# student.age = 200  # 抛出ValueError: Invalid age
#在这个示例中，我们使用`property`来验证`age`属性的值是否在合理的范围内。 
```
3. 数据转换

```python
class Temperature:
    def __init__(self, celsius):
        self.celsius = celsius

    @property
    def fahrenheit(self):
        return self.celsius * 9/5 + 32

    @fahrenheit.setter
    def fahrenheit(self, value):
        self.celsius = (value - 32) * 5/9

temp = Temperature(0)
print(temp.fahrenheit)  # 输出: 32.0
temp.fahrenheit = 32
print(temp.celsius)  # 输出: 0.0
```
在这个示例中，我们使用`property`来在摄氏度和华氏度之间进行转换。

##### 8.2 `__getattr__` 与 `__getattribute__` 属性

`__getattr__` 和 `__getattribute__` 是特殊的魔法方法，用于定制属性的获取行为。其中 `__getattr__` 只有在属性找不到时调用，而 `__getattribute__` 则在每次属性访问时都会被调用。

1. `__getattribute__`：
    
    - 这是一个非常强大的魔法方法，因为**每次访问对象的任何属性时都会调用它**，无论该属性是否存在。
    - 由于它的这种行为，需要特别小心地使用它，以避免无限递归（因为在该方法内部访问任何属性都会再次触发它）。
    
    示例：
    ```python
    class MyClass:
        def __init__(self, data):
            self.data = data
    
        def __getattribute__(self, name):
            print(f"Accessing attribute {name}")
            return super().__getattribute__(name)
    
    obj = MyClass(5)
    print(obj.data)  # 输出 "Accessing attribute data" 然后输出 5
    ```
    
2. `__getattr__`：
    - 仅当所请求的属性在对象上不存在时，才会调用这个方法。
    - 它通常用于为未知的属性访问提供默认值或在动态地创建属性时进行一些处理。

    示例：
    ```python
    class MyClass:
        def __getattr__(self, name):
            print(f"Attribute {name} not found, providing a default value.")
            return 10
    
    obj = MyClass()
    print(obj.existing_attr)  # 输出 "Attribute existing_attr not found, providing a default value." 然后输出 10
    ```

以下是一个结合了 `__getattribute__` 和 `__getattr__` 的例子，展示了如何使用这两个方法来记录对不存在的属性的访问：

```python
class MyClass:
    def __init__(self, data):
        self.data = data

    def __getattribute__(self, name):
        print(f"Trying to access attribute {name}")
        return super().__getattribute__(name)

    def __getattr__(self, name):
        print(f"Attribute {name} not found.")
        return "Default value"

obj = MyClass(5)
print(obj.data)          # 输出 "Trying to access attribute data" 然后输出 5
print(obj.nonexistent)   # 输出 "Trying to access attribute nonexistent"，然后输出 "Attribute nonexistent not found."，最后输出 "Default value"
```

这些魔法方法为 Python 对象的属性访问提供了巨大的灵活性和控制能力，使开发者能够为不同的需求定制特定的行为。

##### 8.3 属性描述符和属性查找过程

属性描述符是Python中的一个高级特性，允许开发者自定义属性访问的行为。具体来说，描述符是一个实现了`__get__`、`__set__` 和/或 `__delete__` 方法的对象。这些特殊的方法允许开发者在获取、设置或删除属性时执行自定义的操作。描述符通常作为类的类属性使用，并且它们对于类的所有实例都是共享的。

描述符可以分为两类：

1. **数据描述符**：实现了`__set__`或`__delete__`方法的描述符。当一个描述符是数据描述符时，它的`__set__`方法会优先于实例字典中的同名属性。

2. **非数据描述符**：只实现了`__get__`方法的描述符。如果实例字典中有与描述符同名的属性，那么实例字典中的属性会覆盖非数据描述符。

一个简单的描述符示例：

```python
class Descriptor:
    def __get__(self, instance, owner):
        return "This is a descriptor!"

    def __set__(self, instance, value):
        print(f"Setting value: {value}")

class MyClass:
    attribute = Descriptor()

obj = MyClass()
print(obj.attribute)  # 输出: This is a descriptor!
obj.attribute = "Hello"  # 输出: Setting value: Hello
```

在上述示例中，`Descriptor` 是一个描述符类，它定义了`__get__`和`__set__`方法。当我们尝试访问`MyClass`的实例`obj`的`attribute`属性时，会调用`Descriptor`的`__get__`方法。当我们尝试设置`obj`的`attribute`属性时，会调用`Descriptor`的`__set__`方法。

描述符在很多Python内置功能中都有应用，例如`@staticmethod`、`@classmethod`和`@property`装饰器背后的实现都是基于描述符的。

属性描述符在开发中有多种应用场景。以下是几个常见的使用描述符的示例场景及代码例子：

1. **属性验证**：
   当你想要确保为某个属性设置的值满足特定条件时，可以使用描述符。

   ```python
   class PositiveValue:
       def __get__(self, instance, owner):
           return instance.__dict__['value']
       
       def __set__(self, instance, value):
           if value <= 0:
               raise ValueError("Value must be positive!")
           instance.__dict__['value'] = value
   
   class MyClass:
       value = PositiveValue()
   
   obj = MyClass()
   obj.value = 10   # 正确
   # obj.value = -5  # 将抛出 ValueError
   ```

2. **属性访问日志记录**：
   如果你希望记录每次属性被访问或更改的时间、旧值和新值，可以使用描述符。

   ```python
   import datetime
   
   class LoggedAttribute:
       def __get__(self, instance, owner):
           return instance.__dict__['attribute']
   
       def __set__(self, instance, value):
           print(f"{datetime.datetime.now()}: Updated 'attribute' from {instance.__dict__.get('attribute')} to {value}")
           instance.__dict__['attribute'] = value
   
   class MyClass:
       attribute = LoggedAttribute()
   
   obj = MyClass()
   obj.attribute = "Test"   # 输出时间和属性值的更新信息
   ```

3. **懒加载**：
   如果一个属性的计算成本很高，你可能希望它只在首次访问时被计算，并缓存其结果。

   ```python
   class LazyProperty:
       def __init__(self, function):
           self.function = function
           self.name = function.__name__
   
       def __get__(self, instance, owner):
           if self.name not in instance.__dict__:
               instance.__dict__[self.name] = self.function(instance)
           return instance.__dict__[self.name]
   
   class MyClass:
       @LazyProperty
       def expensive_calculation(self):
           print("Calculating...")
           return 42
   
   obj = MyClass()
   print(obj.expensive_calculation)  # 输出 "Calculating..." 然后输出 42
   print(obj.expensive_calculation)  # 只输出 42
   ```

4. **计算属性**：
   使用描述符可以创建基于其他属性的值动态计算的属性。

   ```python
   class Celsius:
       def __get__(self, instance, owner):
           return (instance.fahrenheit - 32) * 5.0/9.0
   
       def __set__(self, instance, value):
           instance.fahrenheit = value * 9.0/5.0 + 32
   
   class Temperature:
       celsius = Celsius()
   
       def __init__(self, fahrenheit=32):
           self.fahrenheit = fahrenheit
   
   temp = Temperature(212)
   print(temp.celsius)  # 输出 100.0
   temp.celsius = 0
   print(temp.fahrenheit)  # 输出 32.0
   ```

**属性查找过程:**

```python
如果user是某个类的实例，那么user.age（以及等价的getattr(user,’age’)）
首先调用__getattribute__。如果类定义了__getattr__方法，
那么在__getattribute__抛出 AttributeError 的时候就会调用到__getattr__，
而对于描述符(__get__）的调用，则是发生在__getattribute__内部的。
user = User(), 那么user.age 顺序如下：
（1）如果“age”是出现在User或其基类的__dict__中， 且age是data descriptor， 那么调用其__get__方法, 否则
（2）如果“age”出现在user的__dict__中， 那么直接返回 obj.__dict__[‘age’]， 否则
（3）如果“age”出现在User或其基类的__dict__中
（3.1）如果age是non-data descriptor，那么调用其__get__方法， 否则
（3.2）返回 __dict__[‘age’]
（4）如果User有__getattr__方法，调用__getattr__方法，否则
（5）抛出AttributeError
```

##### 8.4 `__new__` 与 `__init__` 的区别

`__new__` 和 `__init__` 都是 Python 中用于对象创建和初始化的魔法方法，但它们有明确的区别和用途。`__new__` 是一个静态方法，负责创建并返回一个对象实例，而 `__init__` 用于初始化这个对象的状态。

1. `__new__`:
    
    - `__new__` 是一个静态方法，它在 `__init__` 之前被调用。
    - 它负责创建并返回一个新的实例。如果没有重新定义 `__new__`，默认情况下它会创建一个该类的实例并返回。
    - 这是少数几个显式需要返回值的魔法方法之一（它必须返回一个实例）。
    - 它常用于控制不可变对象（如字符串、元组或自定义的不可变对象）的创建或单例模式的实现。
    
    ```python
    #在这段代码中，super() 函数的作用是临时提供 MyClass 的父类（在这个例子中，这个父类是内置的 object 类，因为 MyClass 没有明确地继承其他类）。
    
    #当在 MyClass 中调用 super().__new__(cls) 时，它实际上是在调用其父类（即 object 类）的 __new__ 方法来创建一个新的实例。然后，这个新创建的实例被返回，可以用作 MyClass 的实例。
    
    #简而言之，super() 在这里确保了 MyClass 的实例是通过其父类（object）的 __new__ 方法来创建的。这也是为什么当你实例化 MyClass 时，你会看到输出 "Creating an instance"。
    
    class MyClass:
        def __new__(cls, *args, **kwargs):
            print("Creating an instance")
            instance = super().__new__(cls)
            return instance
    
    obj = MyClass()  # 输出 "Creating an instance"
    ```
    
     `__new__` 在两种场景下的示例：
    
    1. 不可变对象的创建:
    
        假设我们想要创建一个自定义的不可变整数类，我们可以使用 `__new__` 来确保一旦对象被创建就不能再修改它。
    
        ```python
        class ImmutableInt:
            def __new__(cls, value):
                instance = super().__new__(cls)
                instance._value = value
                return instance
        
            @property
            def value(self):
                return self._value
        
        num = ImmutableInt(5)
        print(num.value)  # 输出: 5
        
        # num.value = 10  # AttributeError，因为没有提供 value 的 setter 方法
        ```
    
    2. 单例模式的实现:
    
        单例模式确保一个类只有一个实例，并提供一个全局点来访问这个实例。我们可以通过 `__new__` 来实现这一点，确保每次都返回相同的实例。
    
        ```python
        class Singleton:
            _instance = None
        
            def __new__(cls, *args, **kwargs):
                if not cls._instance:
                    cls._instance = super().__new__(cls)
                return cls._instance
        
        obj1 = Singleton()
        obj2 = Singleton()
        
        print(obj1 is obj2)  # 输出: True，说明 obj1 和 obj2 是相同的实例
        ```
    
        在上述单例模式示例中，我们首先检查 `_instance` 是否已经存在，如果不存在，则调用超类的 `__new__` 方法来创建一个实例。如果已经存在，则直接返回这个实例。这确保了 `Singleton` 类始终只有一个实例。
    
2. `__init__`:
    
    - `__init__` 用于初始化已创建的实例。
    - 与 `__new__` 不同，`__init__` 并不需要返回任何值。它只是修改对象的状态。
    - 在对象创建后立即被调用。
    
    ```python
    class MyClass:
        def __init__(self, data):
            print("Initializing an instance")
            self.data = data
    
    obj = MyClass(5)  # 输出 "Initializing an instance"
    ```

结合使用 `__new__` 和 `__init__` 的示例：
```python
class MyClass:
    def __new__(cls, *args, **kwargs):
        print("Creating an instance")
        instance = super().__new__(cls)
        return instance

    def __init__(self, data):
        print("Initializing an instance")
        self.data = data

obj = MyClass(5)
# 输出:
# Creating an instance
# Initializing an instance
```

总结：
- `__new__` 主要负责创建对象。
- `__init__` 主要负责初始化对象。
- 在日常的类定义中，我们主要使用 `__init__` 来定义对象的初始化行为。但在某些特殊场景下，例如单例模式、元编程或自定义不可变对象时，可能需要重写 `__new__`。

##### 8.5 自定义元类

在最基本的层面上，元类是创建类的类。在Python中，类也是对象。这意味着，就像所有对象（例如字符串、列表、字典等）都是由类创建的一样，类也是由某些东西创建的。那“某物”就是元类。

让我们逐步了解元类，并使用代码示例来演示其功能：

1. 类是对象的蓝图:
```python
class MyClass:
    pass

obj = MyClass()  # obj是由MyClass这个“蓝图”创建的实例
```

2. 但类也是对象 (type 最基本的元类) :
这意味着类（就像`MyClass`）本身也是由某个“蓝图”或类创建的。这个“蓝图”就是`type`。
```python
print(isinstance(MyClass, type))  # 输出: True
```

​	在Python中，`type`是最原始的元类。它实际上可以做两件事：

​		1. 当作为一个函数调用时（例如`type(obj)`），它返回`obj`的类型。

​		2. 当作为一个元类调用时，它可以创建并返回一个新的类。

```python
# 使用type创建一个新的类
MyClass = type('MyClass', (), {})
obj = MyClass()
print(isinstance(obj, MyClass))  # True
```

在上面的代码中，我们使用`type`直接创建了一个新的类。

在`type`用于创建类时，这三个参数的含义是：

1. **类的名称** (`'myclass'`): 这是一个字符串，表示要创建的类的名称。

2. **基类的元组** (`()`): 这是一个元组，其中包含了此类继承的所有基类。如果没有指定基类，则默认为继承自`object`。

3. **属性字典** (`{}`): 这是一个字典，其中包含了类的属性名和值。这可以是方法、类变量等。

因此，在这个例子中:
```python
myclass = type('myclass', (), {})
```
你创建了一个名为`myclass`的新类，它没有继承任何基类（除了隐式的`object`基类）并且没有任何属性或方法。

为了更好地理解，让我们看一个稍微复杂的例子：

```python
def my_method(self):
    return "Hello from my_method"

MyClass = type('MyClass', (object,), {'x': 10, 'my_method': my_method})

obj = MyClass()
print(obj.x)           # 输出: 10
print(obj.my_method()) # 输出: Hello from my_method
```

在这个例子中，我们创建了一个类`MyClass`，它继承了`object`并有一个类变量`x`和一个方法`my_method`。

3. 自定义元类:
  虽然`type`是最原始的元类，但我们可以定义自己的元类来更改类的创建方式。要定义自己的元类，只需继承`type`并重写其方法，例如`__new__`和`__init__`。

```python
class Meta(type):
    def __new__(cls, name, bases, dct):
        dct['class_created_by'] = "Meta class"
        return super(Meta, cls).__new__(cls, name, bases, dct)

class MyClass(metaclass=Meta):
    pass

obj = MyClass()
print(obj.class_created_by)  # 输出: Meta class
```
在上面的代码中，我们创建了一个名为`Meta`的元类，它确保任何使用它作为元类的类都会自动获得一个`class_created_by`属性。

4. 为什么使用元类?

​	元类通常用于框架和库中，其中开发者可能需要在多个类上提供一致的行为或结构，而无需手动为每个类编	写重复的代码。例如：

- **ORM (Object-Relational Mapping) 框架**：元类可以确保模型类的属性与数据库表中的列相匹配。
- **代码验证和强制模式**：元类可以在类创建时检查类定义，确保它们符合某些约定或模式。

5. 元类的工作原理

​	当Python解释器遇到类定义时，它使用三个参数（**类名、基类的元组和类属性的字典**）调用元类。元类负责	使用这些参数构造并返回新的类对象。

​	在没有明确指定元类的情况下，默认的元类是`type`。但是，通过在类定义中使用`metaclass`关键字，我们可	以指定使用自定义的元类。

##### 8.6 元类实现简单的orm

完整代码：

```python
import numbers
import pymysql

class Field:
    pass
    
class IntField(Field):
    # 数据描述符
    def __init__(self, db_column, min_value=None, max_value=None):
        self._value = None
        self.min_value = min_value
        self.max_value = max_value
        self.db_column = db_column
        if min_value is not None:
            if not isinstance(min_value, numbers.Integral):
                raise ValueError("min_value must be int")
            elif min_value < 0:
                raise ValueError("min_value must be positive int")
        if max_value is not None:
            if not isinstance(max_value, numbers.Integral):
                raise ValueError("max_value must be int")
            elif max_value < 0:
                raise ValueError("max_value must be positive int")
        if min_value is not None and max_value is not None:
            if min_value > max_value:
                raise ValueError("min_value must be smaller than max_value")

                
    def __get__(self, instance, owner):
        return self._value

    def __set__(self, instance, value):
        if not isinstance(value, numbers.Integral):
            raise ValueError("int value need")
        if value < self.min_value or value > self.max_value:
            raise ValueError("value must between min_value and max_value")
        self._value = value
        
        


class CharField(Field):
    def __init__(self, db_column, max_length=None):
        self._value = None
        self.db_column = db_column
        if max_length is None:
            raise ValueError("you must spcify max_lenth for charfiled")
        self.max_length = max_length

    def __get__(self, instance, owner):
        return self._value

    def __set__(self, instance, value):
        if not isinstance(value, str):
            raise ValueError("string value need")
        if len(value) > self.max_length:
            raise ValueError("value len excess len of max_length")
        self._value = value

        
        
#重点
class ModelMetaClass(type):
    def __new__(cls, name, bases, attrs, **kwargs):
        if name == "BaseModel":
            return super().__new__(cls, name, bases, attrs, **kwargs)
        fields = {}
        for key, value in attrs.items():
            if isinstance(value, Field):
                fields[key] = value
        attrs_meta = attrs.get("Meta", None)
        _meta = {}
        db_table = name.lower()
        if attrs_meta is not None:
            table = getattr(attrs_meta, "db_table", None)
            if table is not None:
                db_table = table
        _meta["db_table"] = db_table
        attrs["_meta"] = _meta
        attrs["fields"] = fields
        del attrs["Meta"]
        return super().__new__(cls, name, bases, attrs, **kwargs)


class BaseModel(metaclass=ModelMetaClass):
    
    db = None  # 设置数据库连接

    @classmethod
    def set_database(cls, db):
        cls.db = db
        
        
    def __init__(self, *args, **kwargs):
        for key, value in kwargs.items():
            setattr(self, key, value)
        return super().__init__()

    def save(self):
        fields = []
        values = []
        for key, value in self.fields.items():
            db_column = value.db_column
            if db_column is None:
                db_column = key.lower()
            fields.append(db_column)
            value = getattr(self, key)
            values.append(str(value))
        
        sql = f"INSERT INTO {self._meta['db_table']} ({','.join(fields)}) VALUES ({','.join(['%s'] * len(values))})"
        self.db.execute(sql, values)
        pass

    
class MySQLDatabase:
    def __init__(self, host, port, user, password, db_name):
        self.conn = pymysql.connect(
            host=host,
            port=port,
            user=user,
            password=password,
            db=db_name,
            charset='utf8'
        )
        self.cursor = self.conn.cursor()

    def execute(self, sql, params=None):
        self.cursor.execute(sql, params)
        self.conn.commit()

    def close(self):
        self.cursor.close()
        self.conn.close()

    
class User(BaseModel):
    name = CharField(db_column="name", max_length=100)
    age = IntField(db_column="age", min_value=1, max_value=100)

    class Meta:
        db_table = "user"

database = MySQLDatabase(host='10.244.1.101', port=3306, user='root', password='admin', db_name='testorm')

# 设置模型使用的数据库连接
BaseModel.set_database(database)        
user = User(name="hahahha", age=28)
user.save()   
database.close()
```

### 第九章-迭代器和生成器

##### 9.1 python的迭代协议
Python的迭代协议是一个定义对象如何成为可迭代对象和如何产生迭代器的协议。该协议基于两个主要的方法：`__iter__()` 和 `__next__()`。

1. __iter__() 方法:
    - 它在你尝试迭代一个对象时被调用（例如在一个 `for` 循环中）。
    - 它应该返回一个实现了 `__next__()` 方法的迭代器对象。
    - 如果类已经实现了 `__next__()` 方法，`__iter__()` 可以简单地返回 `self`。

2. __next__() 方法:
    - 当迭代器被用于产生下一个值时，这个方法会被调用。
    - 当没有更多的项可供返回时，它应该抛出一个 `StopIteration` 异常来终止迭代。

一个简单的例子是如何使用这两个方法在一个类中实现迭代协议：

```python
class SimpleCounter:
    def __init__(self, limit):
        self.limit = limit
        self.value = 0

    def __iter__(self):
        return self

    def __next__(self):
        if self.value < self.limit:
            self.value += 1
            return self.value - 1
        else:
            raise StopIteration

counter = SimpleCounter(3)
for val in counter:
    print(val)
```

输出：

```
0
1
2
```

以上，`SimpleCounter` 类定义了一个简单的计数器，它从0计数到（但不包括）指定的限制。当你在 `for` 循环中使用它时，它会使用迭代协议来产生值。

##### 9.2 什么是迭代器和可迭代对象
可迭代对象（Iterable）需要实现 `__iter__` 方法，**该方法返回一个迭代器**。

迭代器（Iterator）必须实现 `__iter__` 和 `__next__` 方法。其中，`__iter__` 方法应返回迭代器本身（即 `return self`），而 `__next__` 方法则负责返回序列的下一个元素，或当没有更多元素时抛出 `StopIteration` 异常。

所以，为了支持迭代，一个对象只需要实现 `__iter__` 方法。但要成为一个完全的迭代器，还需要实现 `__next__` 方法。

```python
#可迭代对象
class MyIterable:
    def __init__(self, start, end):
        self.start = start
        self.end = end

    def __iter__(self):
        # 返回一个新的迭代器对象
        return MyIterator(self.start, self.end)

# 迭代器
class MyIterator:
    def __init__(self, start, end):
        self.current = start
        self.end = end

    def __iter__(self):
        # 迭代器对象应返回自身
        return self

    def __next__(self):
        # 提供下一个值或者当所有值都被迭代完毕时抛出StopIteration
        if self.current < self.end:
            value = self.current
            self.current += 1
            return value
        else:
            raise StopIteration

```

**iter和getitem**

`__iter__` 和 `__getitem__` 都是Python中的魔术方法，它们与对象的迭代操作有关，但有一些关键的区别：

1. `__iter__`:

- 目的: 定义对象的迭代器协议。
- 实现: 当对象实现了 `__iter__` 方法，并且该方法返回一个具有 `__next__` 方法的对象时，该对象就被视为迭代器。
- 用法: 主要用于 `for` 循环中。每次循环时，都会调用迭代器的 `__next__` 方法来获取下一个值，直到 `StopIteration` 异常被抛出。
- 示例:

    ```python
    class MyIterable:
        def __init__(self, data):
            self.data = data
    
        def __iter__(self):
            self.index = 0
            return self
    
        def __next__(self):
            if self.index < len(self.data):
                result = self.data[self.index]
                self.index += 1
                return result
            raise StopIteration
    ```

2. `__getitem__`:

- 目的: 使对象支持下标操作（索引和切片）。
- 实现: 如果对象实现了 `__getitem__` 方法，那么你可以使用下标操作符 `[]` 来访问、切片或修改对象的内容。
- 用法: 主要用于支持索引、切片等操作。例如，字符串、列表和元组都实现了 `__getitem__`。
- 附加点: 如果一个类没有实现 `__iter__`，但实现了 `__getitem__`，那么它仍然可以在 `for` 循环中使用，因为Python会尝试从索引0开始，使用 `__getitem__` 来获取元素，直到抛出 `IndexError`。
- 示例:

    ```python
    class MyCollection:
        def __init__(self, data):
            self.data = data
    
        def __getitem__(self, index):
            return self.data[index]
    ```

区别总结:

1. 目的: `__iter__` 是为了实现迭代器协议，使对象可迭代；而 `__getitem__` 使对象支持下标操作。
2. 返回值: `__iter__` 应返回一个迭代器（通常是自身或其他对象），这个迭代器有 `__next__` 方法；而 `__getitem__` 返回指定索引的值。
3. 异常: 在迭代结束时，迭代器抛出 `StopIteration` 异常；如果使用 `__getitem__` 进行迭代，那么在迭代结束时会抛出 `IndexError`。

##### 9.3 生成器的使用
1. 什么是生成器？

生成器是一种特殊的迭代器。与普通函数不同，生成器允许你在函数中使用 `yield` 语句，这使得函数在每次调用时可以“暂停”并在下次调用时“恢复”。

2. 创建生成器

创建生成器的最简单方式是定义一个包含 `yield` 语句的函数：

```python
def simple_generator():
    yield 1
    yield 2
    yield 3
```

使用生成器函数：

```python
gen = simple_generator()
print(next(gen))  # 输出: 1
print(next(gen))  # 输出: 2
print(next(gen))  # 输出: 3
```

3. 为什么使用生成器？

生成器是“惰性的”，它们只在请求时生成值，而不是像列表那样预先存储所有值。这样可以节省内存，特别是在处理大数据流或无限系列时。

4. 生成器表达式

除了使用 `yield`，你还可以使用生成器表达式创建生成器：

```python
gen = (x**2 for x in range(5))
```

这与列表推导式非常相似，只是使用了**圆括号**而不是方括号。

5. 使用生成器实现迭代器

生成器可以非常简洁地实现迭代器协议：

```python
def countdown(n):
    while n > 0:
        yield n
        n -= 1
```

6. 使用 `yield from`

在Python 3.3及以上版本中，可以使用 `yield from` 来从另一个**生成器**或**可迭代对象**中产生值：

```python
def generator_chain():
    yield from range(3)
    yield from range(4, 7)
```

7. 发送值到生成器

使用生成器的 `send` 方法，可以向生成器发送值，这些值将作为 `yield` 表达式的结果：

```python
def repeater():
    while True:
        value = (yield)
        print(value)
        
gen = repeater()
next(gen)  # 必须首先调用一次，以启动生成器
gen.send('Hello')  # 输出: Hello
```

8. 生成器的异常处理

可以在生成器中使用 `try`/`except` 进行异常处理。如果在生成器外部使用 `throw` 方法抛出异常，它将在 `yield` 表达式位置被捕获：

```python
def handle_exception():
    while True:
        try:
            value = (yield)
            print(value)
        except ValueError:
            print("Caught a ValueError!")
            
gen = handle_exception()
next(gen)
gen.throw(ValueError)  # 输出: Caught a ValueError!
```

9. 使用 `close` 结束生成器

可以使用 `close` 方法来关闭生成器：

```python
gen = simple_generator()
print(next(gen))  # 输出: 1
gen.close()
```

关闭后再调用 `next` 会抛出 `StopIteration` 异常。

10. 总结

生成器为Python提供了一个简单、内存高效的方式来处理数据流或长系列。它们是**迭代器的一个子集**，**并允许你轻松地创建自己的迭代器而不必实现完整的迭代器协议**。

##### 9.4 生成器原理
当我们谈论堆和栈时，我们通常是在讨论两种主要的内存管理机制。生成器确实提供了一个很好的例子来展示这两者的工作方式。让我为你解释。

1. 栈内存：

这是程序执行时使用的内存区域，它遵循先进后出（LIFO）原则。当一个函数被调用时，一个新的栈帧就会被推到栈上，当函数完成时，这个栈帧就会被弹出。每个栈帧都包含了函数的局部变量、参数和返回地址。

2. 堆内存：

堆是程序中用于分配动态内存的区域。对象在这里被创建，并在不再需要时由垃圾收集器回收。

---

考虑我们的生成器：

```python
def simple_gen():
    yield 1
    a = 2
    yield a
    a = 3
    yield a
```

调用过程与内存状态：

1. 生成器对象的创建

当你调用`simple_gen()`时，它返回一个生成器对象，但函数本身并没有真正执行。这个生成器对象存储在堆上。

```ABAP
Stack:               Heap:
                    
                    simple_gen generator object
```

2. 请求第一个值

当你调用`next()`时，`simple_gen`开始执行直到第一个`yield`。

```ABAP
Stack:               Heap:
simple_gen frame    simple_gen generator object
(yielding 1)         
```

3. 请求第二个值

函数继续执行，创建局部变量`a`，并继续到第二个`yield`。

```ABAP
Stack:               Heap:
simple_gen frame    simple_gen generator object
(yielding 2)        a = 2
```

4. 请求第三个值

函数继续执行，修改局部变量`a`的值，并到达第三个`yield`。

```ABAP
Stack:               Heap:
simple_gen frame    simple_gen generator object
(yielding 3)        a = 3
```

5. 生成器耗尽

再次调用`next()`，没有更多的`yield`，`StopIteration`异常被抛出，栈帧被弹出。

```ABAP
Stack:               Heap:
                    simple_gen generator object
                    a = 3
```

此时，生成器对象仍然存在于堆上，但是它没有活跃的栈帧在栈上。生成器对象会在不再被引用时由垃圾收集器回收。

这就是为什么生成器可以暂停并恢复执行的原因：它们在堆上维护了一个状态，并且在需要时可以重新推入一个栈帧到调用栈上。

##### 9.6 生成器实现大文件读取
```python
#读取大文件
def readfiles(f, label):
    buf = ''
    while True:
        while label in buf:
            pos = buf.index(label)
            yield buf[:pos]
            buf = buf[pos + len(label):]
        chunk = f.read(4096 * 10)
        if not chunk:
            yield buf
            break
        buf += chunk

with open('test.txt') as f:
    for line in readfiles(f, "{|}"):
        print(line)
```

### 第十章- socket编程

|   OSI层    |             功能             |            TCP/IP            |
| :--------: | :--------------------------: | :--------------------------: |
|   应用层   | 文件传输、电子邮件、文件服务 | HTTP, FTP, SMTP, DNS, Telnet |
|   传输层   |       提供端对端的接口       |           TCP, UDP           |
|   网络层   |       为数据包选择路由       |           IP, ICMP           |
| 数据链路层 | 传输有地址的帧，错误检测功能 |             ARP              |
|   物理层   |           物理媒体           |         1000BASE-SX          |

##### 实现一个socket对话功能

```python
#server

import socket

def server_program():
    host = '127.0.0.1'
    port = 65432

    server_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server_socket.bind((host, port))
    server_socket.listen(1)
    print("服务器正在监听...")

    conn, address = server_socket.accept()
    print(f"连接来自 {address}")

    while True:
        data = conn.recv(1024).decode()
        if not data:
            break
        print(f"从客户端接收到的消息: {data}")

        message = "你好，客户端!"
        conn.send(message.encode())

    conn.close()

if __name__ == "__main__":
    server_program()
```

```python
#client
import socket

def client_program():
    host = '127.0.0.1'
    port = 65432

    client_socket = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    client_socket.connect((host, port))

    message = input(" -> ")
    while message.lower().strip() != 'bye':
        client_socket.send(message.encode())
        data = client_socket.recv(1024).decode()
        print(f"从服务器接收到的消息: {data}")
        message = input(" -> ")

    client_socket.close()

if __name__ == "__main__":
    client_program()
```

1. Socket 类型:
   - `SOCK_STREAM`: TCP socket
   - `SOCK_DGRAM`: UDP socket

2. Socket 地址家族:
   - `AF_INET`: IPv4
   - `AF_INET6`: IPv6

3. 错误处理: 使用 `socket.error` 处理 socket 相关的异常。

4. 非阻塞 Sockets: 使用 `setblocking(0)` 或 `settimeout(value)` 设置非阻塞模式或超时。

5. Socket 选项: 使用 `setsockopt()` 和 `getsockopt()` 方法设置和获取 socket 选项，例如 `SO_REUSEADDR`。

6. 获取本机信息: 使用 `gethostname()`, `gethostbyname()` 和其他相关方法。

7. 高级功能: 包括但不限于 SSL 封装、多路复用（使用 `select` 或 `selectors` 模块）等。