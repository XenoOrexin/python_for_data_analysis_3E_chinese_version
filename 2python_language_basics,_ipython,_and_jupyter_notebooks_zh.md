# 2Python 语言基础、IPython 和 Jupyter Notebooks

__

《Python 数据分析 第 3 版》的开放获取网络版本现已上线，作为[印刷版和数字版](https://amzn.to/3DyLaJc)的配套资源。如果您发现任何错误，[请在此处报告](https://oreilly.com/catalog/0636920519829/errata)。请注意，由于本网站由 Quarto 生成，其某些方面会与 O'Reilly 的印刷版和电子书版本的格式有所不同。

如果您觉得本书在线版有用，请考虑[订购纸质版](https://amzn.to/3DyLaJc)或[无 DRM 限制的电子书](https://www.ebooks.com/en-us/book/210644288/python-for-data-analysis/wes-mckinney/?affId=WES398681F)以支持作者。本网站内容不可复制或转载。代码示例采用 MIT 许可证，可在 GitHub 或 Gitee 上获取。

当我在 2011 年和 2012 年撰写本书第一版时，可用于学习 Python 数据分析的资源较少。这部分是一个“先有鸡还是先有蛋”的问题；许多我们现在认为理所当然的库（如 pandas、scikit-learn 和 statsmodels）在当时还相对不成熟。如今到了 2022 年，关于数据科学、数据分析和机器学习的文献日益丰富，补充了之前面向计算科学家、物理学家和其他研究领域专业人士的通用科学计算著作。此外，还有大量优秀的书籍涉及 Python 编程语言本身的学习以及如何成为高效的软件工程师。

由于本书旨在作为使用 Python 处理数据的入门教材，我认为从数据操作的角度，独立概述 Python 内置数据结构和库的一些最重要特性是非常有价值的。因此，在本章及[第 3 章：内置数据结构、函数和文件](https://wesmckinney.com/book/python-builtin)中，我将仅提供足够的基础知识，以便您能理解本书后续内容。

本书大部分内容聚焦于基于表的分析和数据准备工具，适用于能在个人计算机上处理的数据集。为了使用这些工具，您有时需要进行一些数据整理，将杂乱的数据转换为更规整的表格（或_结构化_）形式。幸运的是，Python 是完成这项工作的理想语言。您对 Python 语言及其内置数据类型的掌握越熟练，就越容易为分析准备新的数据集。

本书中的一些工具最好通过实时的 IPython 或 Jupyter 会话来探索。一旦您学会了如何启动 IPython 和 Jupyter，我建议您跟着示例操作，以便进行实验和尝试不同的内容。与任何基于键盘的控制台环境一样，熟悉常用命令也是学习过程的一部分。

__

注意 

本章未涉及一些 Python 入门概念（如类和面向对象编程），但这些概念可能对您入门 Python 数据分析有所帮助。

为深化 Python 语言知识，我建议您结合[官方 Python 教程](http://docs.python.org)以及任何一本优秀的通用 Python 编程书籍来补充本章内容。以下推荐书籍可供入门：

*   《Python Cookbook 第 3 版》David Beazley 和 Brian K. Jones 著（O'Reilly 出版）
*   《流畅的 Python》Luciano Ramalho 著（O'Reilly 出版）
*   《Effective Python 第 2 版》Brett Slatkin 著（Addison-Wesley 出版）

## 2.1 Python 解释器（Python Interpreter）

Python 是一门_解释型_语言。Python 解释器通过逐行执行语句的方式来运行程序。可以通过在命令行中输入 `python` 命令来启动标准的交互式 Python 解释器：

```bash
$ python
Python 3.10.4 | packaged by conda-forge | (main, Mar 24 2022, 17:38:57)
[GCC 10.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> a = 5
>>> print(a)
5
```

你所看到的 `>>>` 是_提示符_（prompt），可以在其后输入代码表达式。要退出 Python 解释器，可以输入 `exit()` 或按 Ctrl-D（仅适用于 Linux 和 macOS 系统）。

运行 Python 程序非常简单，只需以 _.py_ 文件作为第一个参数调用 `python` 命令。假设我们已经创建了包含以下内容的 _hello\_world.py_ 文件：

```python
print("Hello world")
```

可以通过执行以下命令来运行该文件（_hello\_world.py_ 文件必须位于当前终端工作目录中）：

```bash
$ python hello_world.py
Hello world
```

虽然有些 Python 程序员会完全以这种方式执行所有代码，但从事数据分析或科学计算的人员通常会使用 IPython（一个增强的 Python 解释器）或 Jupyter notebooks（最初基于 IPython 项目开发的基于 Web 的代码笔记本）。本章将介绍 IPython 和 Jupyter 的使用方法，并在[附录 A：NumPy 高级应用](https://wesmckinney.com/book/advanced-numpy)中深入探讨 IPython 的功能。当你使用 `%run` 命令时，IPython 会在同一进程中执行指定文件中的代码，并在执行完成后支持交互式结果探索：

```bash
$ ipython
Python 3.10.4 | packaged by conda-forge | (main, Mar 24 2022, 17:38:57)
Type 'copyright', 'credits' or 'license' for more information
IPython 7.31.1 -- An enhanced Interactive Python. Type '?' for help.

In [1]: %run hello_world.py
Hello world

In [2]:
```

默认的 IPython 提示符采用带编号的 `In [2]:` 样式，这与标准解释器的 `>>>` 提示符有所不同。

## 2.2 IPython 基础

在本节中，我将引导您启动并运行 IPython shell 和 Jupyter notebook，并介绍一些基本概念。

### 运行 IPython Shell

您可以在命令行中启动 IPython shell，就像启动常规的 Python 解释器一样，只不过使用 `ipython` 命令：

```
$ ipython
Python 3.10.4 | packaged by conda-forge | (main, Mar 24 2022, 17:38:57)
Type 'copyright', 'credits' or 'license' for more information
IPython 7.31.1 -- An enhanced Interactive Python. Type '?' for help.

In [1]: a = 5

In [2]: a
Out[2]: 5
```

您可以通过输入任意 Python 语句并按 Return（或 Enter）键来执行它们。当您在 IPython 中仅输入一个变量时，它会呈现该对象的字符串表示形式：

```
In [5]: import numpy as np

In [6]: data = [np.random.standard_normal() for i in range(7)]

In [7]: data
Out[7]: 
[-0.20470765948471295,
 0.47894333805754824,
 -0.5194387150567381,
 -0.55573030434749,
 1.9657805725027142,
 1.3934058329729904,
 0.09290787674371767]
```

前两行是 Python 代码语句；第二条语句创建了一个名为 `data` 的变量，该变量引用一个新创建的列表。最后一行在控制台中打印了 `data` 的值。

许多类型的 Python 对象都被格式化为更易读的形式，或者称为美观打印（pretty-printed），这与使用 `print` 进行普通打印不同。如果您在标准 Python 解释器中打印上述 `data` 变量，可读性会差很多：

```
>>> import numpy as np
>>> data = [np.random.standard_normal() for i in range(7)]
>>> print(data)
>>> data
[-0.5767699931966723, -0.1010317773535111, -1.7841005313329152,
-1.524392126408841, 0.22191374220117385, -1.9835710588082562,
-1.6081963964963528]
```

IPython 还提供了执行任意代码块（通过一种稍显高级的复制粘贴方法）和整个 Python 脚本的功能。您还可以使用 Jupyter notebook 来处理更大的代码块，我们很快就会看到。

### 运行 Jupyter Notebook

Jupyter 项目的主要组件之一是 notebook，这是一种用于代码、文本（包括 Markdown）、数据可视化和其他输出的交互式文档类型。Jupyter notebook 与内核（kernels）交互，内核是针对不同编程语言的 Jupyter 交互式计算协议的具体实现。Python Jupyter 内核使用 IPython 系统作为其底层行为。

要启动 Jupyter，请在终端中运行 `jupyter notebook` 命令：

```
$ jupyter notebook
[I 15:20:52.739 NotebookApp] Serving notebooks from local directory:
/home/wesm/code/pydata-book
[I 15:20:52.739 NotebookApp] 0 active kernels
[I 15:20:52.739 NotebookApp] The Jupyter Notebook is running at:
http://localhost:8888/?token=0a77b52fefe52ab83e3c35dff8de121e4bb443a63f2d...
[I 15:20:52.740 NotebookApp] Use Control-C to stop this server and shut down
all kernels (twice to skip confirmation).
Created new window in existing browser session.
    To access the notebook, open this file in a browser:
        file:///home/wesm/.local/share/jupyter/runtime/nbserver-185259-open.html
    Or copy and paste one of these URLs:
        http://localhost:8888/?token=0a77b52fefe52ab83e3c35dff8de121e4...
     or http://127.0.0.1:8888/?token=0a77b52fefe52ab83e3c35dff8de121e4...
```

在许多平台上，Jupyter 会自动在您的默认网页浏览器中打开（除非您使用 `--no-browser` 选项启动它）。否则，您可以导航到启动 notebook 时打印的 HTTP 地址，例如 `http://localhost:8888/?token=0a77b52fefe52ab83e3c35dff8de121e4bb443a63f2d3055`。图 2.1 显示了在 Google Chrome 中的样子。

注意

许多人将 Jupyter 用作本地计算环境，但它也可以部署在服务器上并通过远程访问。这里不会详细介绍这些内容，但我鼓励您根据需要在互联网上探索这个话题。

![](images/pda3_0201.png)

图 2.1：Jupyter notebook 登录页面

要创建一个新的 notebook，请单击 New 按钮并选择 "Python 3" 选项。您应该会看到类似图 2.2 的内容。如果这是您第一次使用，请尝试单击空的代码“单元格”并输入一行 Python 代码。然后按 Shift-Enter 执行它。

![](images/pda3_0202.png)

图 2.2：Jupyter 新建 notebook 视图

当您保存 notebook 时（参见 notebook 文件菜单中的 "Save and Checkpoint"），它会创建一个扩展名为 _.ipynb_ 的文件。这是一种自包含的文件格式，包含 notebook 中当前的所有内容（包括任何已评估的代码输出）。其他 Jupyter 用户可以加载和编辑这些文件。

要重命名一个已打开的 notebook，请单击页面顶部的 notebook 标题，输入新标题，完成后按 Enter 键。

要加载现有的 notebook，请将文件放在启动 notebook 进程的同一目录（或其子文件夹）中，然后在登录页面上单击名称。您可以使用我 GitHub 上 _wesm/pydata-book_ 代码库中的 notebook 进行尝试。参见图 2.3。

当您想要关闭一个 notebook 时，单击文件菜单并选择 "Close and Halt"。如果您只是关闭浏览器标签页，与 notebook 关联的 Python 进程将继续在后台运行。

虽然 Jupyter notebook 可能感觉与 IPython shell 是不同的体验，但本章中几乎所有的命令和工具都可以在任一环境中使用。

![](images/pda3_0203.png)

图 2.3：现有 notebook 的 Jupyter 示例视图

### Tab 键补全

从表面上看，IPython shell 看起来像是标准终端 Python 解释器（使用 `python` 调用）的一个外观不同的版本。与标准 Python shell 相比，一个主要的改进是 Tab 键补全（tab completion），这在许多 IDE 或其他交互式计算分析环境中都有。在 shell 中输入表达式时，按 Tab 键将在命名空间中搜索与您已键入字符匹配的任何变量（对象、函数等），并在方便的下拉菜单中显示结果：

```
In [1]: an_apple = 27

In [2]: an_example = 42

In [3]: an<Tab>
an_apple   an_example  any
```

在这个例子中，请注意 IPython 显示了我定义的两个变量，以及内置函数 `any`。此外，您也可以在输入句点后补全任何对象的方法和属性：

```
In [3]: b = [1, 2, 3]

In [4]: b.<Tab>
append()  count()   insert()  reverse()
clear()   extend()  pop()     sort()
copy()    index()   remove()
```

模块也是如此：

```
In [1]: import datetime

In [2]: datetime.<Tab>
date          MAXYEAR       timedelta
datetime      MINYEAR       timezone
datetime_CAPI time          tzinfo
```

注意

请注意，IPython 默认隐藏以下划线开头的方法和属性，例如魔术方法和内部“私有”方法及属性，以避免显示混乱（并迷惑新手用户！）。这些也可以通过 Tab 键补全，但您必须先键入一个下划线才能看到它们。如果您希望总是在 Tab 补全中看到此类方法，可以在 IPython 配置中更改此设置。请参阅 [IPython 文档](https://ipython.readthedocs.io)了解如何操作。

Tab 键补全在许多上下文之外也有效，除了搜索交互式命名空间和补全对象或模块属性。当键入任何看起来像文件路径的内容（甚至在 Python 字符串中）时，按 Tab 键将补全您计算机文件系统中与您已键入内容匹配的任何内容。

结合 `%run` 命令（参见[附录 B.2.1：%run 命令](https://wesmckinney.com/book/ipython#ipython_basics_magic_run)），此功能可以为您节省许多击键。

Tab 键补全节省时间的另一个领域是补全函数关键字参数（包括 `=` 符号！）。参见图 2.4。

![](images/pda3_0204.png)

图 2.4：在 Jupyter notebook 中自动补全函数关键字

我们稍后会更仔细地看看函数。

### 自省

在变量前或后使用问号（`?`）将显示有关该对象的一些常规信息：

```
In [1]: b = [1, 2, 3]

In [2]: b?
Type:        list
String form: [1, 2, 3]
Length:      3
Docstring:
Built-in mutable sequence.

If no argument is given, the constructor creates a new empty list.
The argument must be an iterable if specified.

In [3]: print?
Docstring:
print(value, ..., sep=' ', end='\n', file=sys.stdout, flush=False)

Prints the values to a stream, or to sys.stdout by default.
Optional keyword arguments:
file:  a file-like object (stream); defaults to the current sys.stdout.
sep:   string inserted between values, default a space.
end:   string appended after the last value, default a newline.
flush: whether to forcibly flush the stream.
Type:      builtin_function_or_method
```

这被称为对象自省（object introspection）。如果对象是函数或实例方法，并且定义了文档字符串（docstring），也会显示出来。假设我们编写了以下函数（您可以在 IPython 或 Jupyter 中重现）：

```python
def add_numbers(a, b):
    """
    Add two numbers together

    Returns
    -------
    the_sum : type of arguments
    """
    return a + b
```

然后使用 `?` 会显示文档字符串：

```
In [6]: add_numbers?
Signature: add_numbers(a, b)
Docstring:
Add two numbers together
Returns
-------
the_sum : type of arguments
File:      <ipython-input-9-6a548a216e27>
Type:      function
```

`?` 还有一个最终用途，即以类似于标准 Unix 或 Windows 命令行的方式搜索 IPython 命名空间。许多字符与通配符（`*`）结合使用将显示所有与通配符表达式匹配的名称。例如，我们可以获取顶级 NumPy 命名空间中包含 `load` 的所有函数的列表：

```
In [9]: import numpy as np

In [10]: np.*load*?
np.__loader__
np.load
np.loads
np.loadtxt
```

## 2.3 Python 语言基础

在本节中，我将概述 Python 编程的基本概念和语言机制。下一章将详细讲解 Python 数据结构、函数和其他内置工具。

### 语言语义

Python 语言设计以其对可读性、简洁性和明确性的重视而著称。有人甚至将其比作“可执行的伪代码”。

#### 缩进，而非大括号

Python 使用空白（制表符或空格）来组织代码结构，而不是像 R、C++、Java 和 Perl 等许多其他语言那样使用大括号。来看一个排序算法中的 `for` 循环示例：

```python
for x in array:
    if x < pivot:
        less.append(x)
    else:
        greater.append(x)
```

冒号表示一个缩进代码块的开始，之后所有代码必须缩进相同的量，直到代码块结束。

无论喜欢与否，有意义的空白符对 Python 程序员来说都是必须面对的现实。虽然一开始可能显得陌生，但希望您会逐渐习惯它。

> 注意
>
> 我强烈建议使用**四个空格**作为默认缩进，并用四个空格替换制表符。许多文本编辑器都有自动将制表位替换为空格的设置（请这样做！）。IPython 和 Jupyter 笔记本会在冒号后的新行自动插入四个空格，并将制表符替换为四个空格。

正如您现在所看到的，Python 语句也不需要以分号结尾。但是，分号可用于在同一行上分隔多个语句：

```python
a = 5; b = 6; c = 7
```

在 Python 中，通常不鼓励在一行上放置多个语句，因为这可能会降低代码的可读性。

#### 万物皆对象

Python 语言的一个重要特性是其**对象模型（object model）**的一致性。每个数字、字符串、数据结构、函数、类、模块等等，在 Python 解释器中都存在于自己的“盒子”中，这被称为 **Python 对象（Python object）**。每个对象都有一个关联的**类型（type）**（例如，**整数（integer）**、**字符串（string）** 或**函数（function）**）和内部数据。实际上，这使语言非常灵活，因为即使是函数也可以像任何其他对象一样被对待。

#### 注释

Python 解释器会忽略任何以井号（#）开头的文本。这通常用于向代码添加注释。有时您可能还想排除某些代码块而不删除它们。一种解决方案是**注释掉（comment out）** 代码：

```python
results = []
for line in file_handle:
    # 暂时保留空行
    # if len(line) == 0:
    #   continue
    results.append(line.replace("foo", "bar"))
```

注释也可以出现在已执行代码的行后。虽然一些程序员更喜欢将注释放在特定代码行之前，但这种方式有时也很有用：

```python
print("Reached this line")  # 简单状态报告
```

#### 函数和对象方法调用

您可以使用括号调用函数并传递零个或多个参数，可以选择将返回值赋给变量：

```python
result = f(x, y, z)
g()
```

Python 中几乎每个对象都有附加的函数，称为**方法（methods）**，这些方法可以访问对象的内部内容。您可以使用以下语法调用它们：

```python
obj.some_method(x, y, z)
```

函数可以接受**位置（positional）** 参数和**关键字（keyword）** 参数：

```python
result = f(a, b, c, d=5, e="foo")
```

我们将在后面更详细地讨论这一点。

#### 变量和参数传递

在 Python 中为变量（或**名称（name）**）赋值时，您是在创建对等号右侧所示对象的**引用（reference）**。实际上，考虑一个整数列表：

```python
In [8]: a = [1, 2, 3]
```

假设我们将 `a` 赋给一个新变量 `b`：

```python
In [9]: b = a

In [10]: b
Out[10]: [1, 2, 3]
```

在某些语言中，对 `b` 的赋值会导致数据 `[1, 2, 3]` 被复制。在 Python 中，`a` 和 `b` 现在实际上引用的是同一个对象，即原始列表 `[1, 2, 3]`（参见图 2.5 的示意图）。您可以通过向 `a` 追加一个元素然后检查 `b` 来证明这一点：

```python
In [11]: a.append(4)

In [12]: b
Out[12]: [1, 2, 3, 4]
```

![](images/pda3_0205.png)

图 2.5：同一对象的两个引用

在 Python 中理解引用的语义，以及数据何时、如何以及为何被复制，尤其是在处理较大数据集时，尤为重要。

> 注意
>
> 赋值也被称为**绑定（binding）**，因为我们将一个名称绑定到一个对象。已被赋值的变量名有时可能被称为**绑定变量（bound variables）**。

当您将对象作为参数传递给函数时，会创建新的局部变量引用原始对象，而无需任何复制。如果您在函数内部将一个新对象绑定到一个变量，这不会覆盖函数外部“作用域”（“父作用域”）中同名的变量。因此，可以改变可变参数的内部。假设我们有以下函数：

```python
In [13]: def append_element(some_list, element):
   ....:     some_list.append(element)
```

那么我们有：

```python
In [14]: data = [1, 2, 3]

In [15]: append_element(data, 4)

In [16]: data
Out[16]: [1, 2, 3, 4]
```

#### 动态引用，强类型

Python 中的变量没有固有的类型与之关联；变量只需通过赋值就可以引用不同类型的对象。以下代码没有问题：

```python
In [17]: a = 5

In [18]: type(a)
Out[18]: int

In [19]: a = "foo"

In [20]: type(a)
Out[20]: str
```

变量是特定命名空间中对象的名称；类型信息存储在对象本身中。一些观察者可能会草率地得出结论，认为 Python 不是“类型化语言”。这是不正确的；考虑这个例子：

```python
In [21]: "5" + 5
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-21-7fe5aa79f268> in <module>
----> 1 "5" + 5
TypeError: can only concatenate str (not "int") to str
```

在某些语言中，字符串 `'5'` 可能会被隐式转换（或**强制转换（cast）**）为整数，从而产生 10。在其他语言中，整数 `5` 可能被强制转换为字符串，产生连接后的字符串 `'55'`。在 Python 中，不允许此类隐式强制转换。在这方面，我们说 Python 是一种**强类型（strongly typed）** 语言，这意味着每个对象都有一个特定的类型（或**类（class）**），并且隐式转换只会在某些允许的情况下发生，例如：

```python
In [22]: a = 4.5

In [23]: b = 2

# 字符串格式化，稍后会介绍
In [24]: print(f"a is {type(a)}, b is {type(b)}")
a is <class 'float'>, b is <class 'int'>

In [25]: a / b
Out[25]: 2.25
```

这里，即使 `b` 是一个整数，它也会在除法操作中被隐式转换为浮点数。

了解对象的类型很重要，并且能够编写可以处理许多不同种类输入的函数也很有用。您可以使用 `isinstance` 函数检查对象是否是特定类型的实例：

```python
In [26]: a = 5

In [27]: isinstance(a, int)
Out[27]: True
```

如果您想检查对象的类型是否存在于元组中，`isinstance` 可以接受一个类型元组：

```python
In [28]: a = 5; b = 4.5

In [29]: isinstance(a, (int, float))
Out[29]: True

In [30]: isinstance(b, (int, float))
Out[30]: True
```

#### 属性和方法

Python 中的对象通常既有**属性（attributes）**（存储在对象“内部”的其他 Python 对象）也有**方法（methods）**（与对象关联的函数，可以访问对象的内部数据）。两者都通过语法 <obj.attribute_name> 访问：

```python
In [1]: a = "foo"

In [2]: a.<Press Tab>
capitalize() index()        isspace()      removesuffix()  startswith()
casefold()   isprintable()  istitle()      replace()       strip()
center()     isalnum()      isupper()      rfind()         swapcase()
count()      isalpha()      join()         rindex()        title()
encode()     isascii()      ljust()        rjust()         translate()
endswith()   isdecimal()    lower()        rpartition()
expandtabs() isdigit()      lstrip()       rsplit()
find()       isidentifier() maketrans()    rstrip()
format()     islower()      partition()    split()
format_map() isnumeric()    removeprefix() splitlines()
```

也可以通过 `getattr` 函数按名称访问属性和方法：

```python
In [32]: getattr(a, "split")
Out[32]: <function str.split(sep=None, maxsplit=-1)>
```

虽然我们在本书中不会广泛使用 `getattr` 及相关函数 `hasattr` 和 `setattr`，但它们可以非常有效地用于编写通用、可重用的代码。

#### 鸭子类型（Duck typing）

通常您可能不关心对象的类型，而只关心它是否具有某些方法或行为。这有时被称为**鸭子类型（duck typing）**，源于谚语“如果它走路像鸭子，叫声像鸭子，那么它就是鸭子”。例如，如果对象实现了**迭代器协议（iterator protocol）**，您可以验证它是可迭代的。对于许多对象，这意味着它有一个 `__iter__` “魔术方法”，不过另一种更好检查方法是尝试使用 `iter` 函数：

```python
In [33]: def isiterable(obj):
   ....:     try:
   ....:         iter(obj)
   ....:         return True
   ....:     except TypeError: # not iterable
   ....:         return False
```

对于字符串以及大多数 Python 集合类型，此函数将返回 `True`：

```python
In [34]: isiterable("a string")
Out[34]: True

In [35]: isiterable([1, 2, 3])
Out[35]: True

In [36]: isiterable(5)
Out[36]: False
```

#### 导入

在 Python 中，**模块（module）** 就是一个带有 _.py_ 扩展名的文件，其中包含 Python 代码。假设我们有以下模块：

```python
# some_module.py
PI = 3.14159

def f(x):
    return x + 2

def g(a, b):
    return a + b
```

如果我们想从同一目录中的另一个文件访问 _some_module.py_ 中定义的变量和函数，我们可以这样做：

```python
import some_module
result = some_module.f(5)
pi = some_module.PI
```

或者：

```python
from some_module import g, PI
result = g(5, PI)
```

通过使用 `as` 关键字，您可以为导入项指定不同的变量名：

```python
import some_module as sm
from some_module import PI as pi, g as gf

r1 = sm.f(pi)
r2 = gf(6, pi)
```

#### 二元运算符和比较

大多数二元数学运算和比较使用其他编程语言中熟悉的数学语法：

```python
In [37]: 5 - 7
Out[37]: -2

In [38]: 12 + 21.5
Out[38]: 33.5

In [39]: 5 <= 2
Out[39]: False
```

所有可用的二元运算符请参见表 2.1。

表 2.1：二元运算符
操作 | 描述  
---|---  
`a + b` | 将 `a` 和 `b` 相加  
`a - b` | 从 `a` 中减去 `b`  
`a * b` | 将 `a` 乘以 `b`  
`a / b` | 将 `a` 除以 `b`  
`a // b` | 将 `a` 除以 `b` 取整，丢弃任何小数余数  
`a ** b` | 将 `a` 的 `b` 次幂  
`a & b` | 如果 `a` 和 `b` 都是 `True`，则为 `True`；对于整数，取按位 `AND`  
`a | b` | 如果 `a` 或 `b` 是 `True`，则为 `True`；对于整数，取按位 `OR`  
`a ^ b` | 对于布尔值，如果 `a` 或 `b` 是 `True`，但不同时为 `True`，则为 `True`；对于整数，取按位 `EXCLUSIVE-OR`  
`a == b` | 如果 `a` 等于 `b`，则为 `True`  
`a != b` | 如果 `a` 不等于 `b`，则为 `True`  
`a < b`, a <= b | 如果 `a` 小于（小于或等于）`b`，则为 `True`  
`a > b, a >= b` | 如果 `a` 大于（大于或等于）`b`，则为 `True`  
`a is b` | 如果 `a` 和 `b` 引用同一个 Python 对象，则为 `True`  
`a is not b` | 如果 `a` 和 `b` 引用不同的 Python 对象，则为 `True`  

要检查两个变量是否引用同一个对象，请使用 `is` 关键字。使用 `is not` 检查两个对象是否不同：

```python
In [40]: a = [1, 2, 3]

In [41]: b = a

In [42]: c = list(a)

In [43]: a is b
Out[43]: True

In [44]: a is not c
Out[44]: True
```

由于 `list` 函数总是创建一个新的 Python 列表（即副本），我们可以确定 `c` 与 `a` 是不同的。与 `is` 比较不同于 `==` 运算符，因为在这种情况下我们有：

```python
In [45]: a == c
Out[45]: True
```

`is` 和 `is not` 的一个常见用法是检查变量是否为 `None`，因为只有一个 `None` 实例：

```python
In [46]: a = None

In [47]: a is None
Out[47]: True
```

#### 可变和不可变对象

Python 中的许多对象，例如列表、字典、NumPy 数组和大多数用户定义的类型（类），都是**可变的（mutable）**。这意味着它们包含的对象或值可以被修改：

```python
In [48]: a_list = ["foo", 2, [4, 5]]

In [49]: a_list[2] = (3, 4)

In [50]: a_list
Out[50]: ['foo', 2, (3, 4)]
```

其他对象，如字符串和元组，是**不可变的（immutable）**，这意味着它们的内部数据无法更改：

```python
In [51]: a_tuple = (3, 5, (4, 5))

In [52]: a_tuple[1] = "four"
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-52-cd2a018a7529> in <module>
----> 1 a_tuple[1] = "four"
TypeError: 'tuple' object does not support item assignment
```

请记住，仅仅因为您**可以**改变一个对象并不意味着您总是**应该**这样做。此类操作称为**副作用（side effects）**。例如，在编写函数时，任何副作用都应在函数的文档或注释中明确告知用户。如果可能，我建议尽量避免副作用并**倾向于不可变性（favor immutability）**，即使可能涉及可变对象。

### 标量类型（Scalar Types）

Python 有一小组内置类型用于处理数值数据、字符串、布尔值（`True` 或 `False`）以及日期和时间。这些“单值”类型有时被称为**标量类型（scalar types）**，我们在本书中将其称为**标量（scalars）**。主要标量类型列表请参见表 2.2。日期和时间处理将单独讨论，因为它们是由标准库中的 `datetime` 模块提供的。

表 2.2：标准 Python 标量类型
类型 | 描述  
---|---  
`None` | Python 的“空”值（只存在一个 `None` 对象实例）  
`str` | 字符串类型；保存 Unicode 字符串  
`bytes` | 原始二进制数据  
`float` | 双精度浮点数（注意没有单独的 `double` 类型）  
`bool` | 布尔值 `True` 或 `False`  
`int` | 任意精度整数  

#### 数值类型

Python 中主要的数字类型是 `int` 和 `float`。`int` 可以存储任意大的数字：

```python
In [53]: ival = 17239871

In [54]: ival ** 6
Out[54]: 26254519291092456596965462913230729701102721
```

浮点数用 Python 的 `float` 类型表示。在底层，每个值都是双精度的。它们也可以用科学记数法表示：

```python
In [55]: fval = 7.243

In [56]: fval2 = 6.78e-5
```

整数除法如果不产生整数，将总是产生浮点数：

```python
In [57]: 3 / 2
Out[57]: 1.5
```

要获得 C 风格的整数除法（如果结果不是整数则丢弃小数部分），请使用地板除运算符 `//`：

```python
In [58]: 3 // 2
Out[58]: 1
```

#### 字符串

许多人使用 Python 是因为其内置的字符串处理能力。您可以使用单引号 `'` 或双引号 `"`（通常更倾向于双引号）来编写**字符串字面量（string literals）**：

```python
a = 'one way of writing a string'
b = "another way"
```

Python 的字符串类型是 `str`。

对于带有换行符的多行字符串，您可以使用三引号，可以是 `'''` 或 `"""`：

```python
c = """
This is a longer string that
spans multiple lines
"""
```

可能会让您惊讶的是，这个字符串 `c` 实际上包含四行文本；`"""` 之后和 `lines` 之后的换行符都包含在字符串中。我们可以使用 `c` 的 `count` 方法来计算换行符的数量：

```python
In [60]: c.count("\n")
Out[60]: 3
```

Python 字符串是不可变的；您不能修改字符串：

```python
In [61]: a = "this is a string"

In [62]: a[10] = "f"
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-62-3b2d95f10db4> in <module>
----> 1 a[10] = "f"
TypeError: 'str' object does not support item assignment
```

要解释此错误消息，请从下往上阅读。我们尝试将位置 10 的字符（“项”）替换为字母 `"f"`，但这对于字符串对象是不允许的。如果需要修改字符串，我们必须使用创建新字符串的函数或方法，例如字符串的 `replace` 方法：

```python
In [63]: b = a.replace("string", "longer string")

In [64]: b
Out[64]: 'this is a longer string'
```

在此操作之后，变量 `a` 未被修改：

```python
In [65]: a
Out[65]: 'this is a string'
```

许多 Python 对象可以使用 `str` 函数转换为字符串：

```python
In [66]: a = 5.6

In [67]: s = str(a)

In [68]: print(s)
5.6
```

字符串是 Unicode 字符的序列，因此可以像其他序列（如列表和元组）一样对待：

```python
In [69]: s = "python"

In [70]: list(s)
Out[70]: ['p', 'y', 't', 'h', 'o', 'n']

In [71]: s[:3]
Out[71]: 'pyt'
```

语法 `s[:3]` 称为**切片（slicing）**，并为许多 Python 序列实现。这将在后面更详细地解释，因为它在本书中广泛使用。

反斜杠字符 `\` 是一个**转义字符（escape character）**，这意味着它用于指定特殊字符，如换行符 `\n` 或 Unicode 字符。要编写带有反斜杠的字符串字面量，您需要对它们进行转义：

```python
In [72]: s = "12\\34"

In [73]: print(s)
12\34
```

如果您有一个包含许多反斜杠且没有特殊字符的字符串，您可能会觉得这有点烦人。幸运的是，您可以在字符串的前导引号前加上 `r`，这意味着字符应按原样解释：

```python
In [74]: s = r"this\has\no\special\characters"

In [75]: s
Out[75]: 'this\\has\\no\\special\\characters'
```

`r` 代表**原始（raw）**。

将两个字符串相加会将它们连接起来并产生一个新字符串：

```python
In [76]: a = "this is the first half "

In [77]: b = "and this is the second half"

In [78]: a + b
Out[78]: 'this is the first half and this is the second half'
```

字符串模板或格式化是另一个重要主题。随着 Python 3 的出现，这样做的方法数量有所增加，这里我将简要介绍其中一种主要接口的机制。字符串对象有一个 `format` 方法，可用于将格式化的参数替换到字符串中，从而产生一个新字符串：

```python
In [79]: template = "{0:.2f} {1:s} are worth US${2:d}"
```

在这个字符串中：

* `{0:.2f}` 表示将第一个参数格式化为带两位小数的浮点数。
* `{1:s}` 表示将第二个参数格式化为字符串。
* `{2:d}` 表示将第三个参数格式化为精确整数。

要为这些格式参数替换参数，我们将一个参数序列传递给 `format` 方法：

```python
In [80]: template.format(88.46, "Argentine Pesos", 1)
Out[80]: '88.46 Argentine Pesos are worth US$1'
```

Python 3.6 引入了一项称为 **f-strings**（**格式化字符串字面量（formatted string literals）** 的缩写）的新功能，它可以使创建格式化字符串更加方便。要创建 f-string，请在字符串字面量前立即写入字符 `f`。在字符串中，将 Python 表达式括在大括号中，以将表达式的值替换到格式化字符串中：

```python
In [81]: amount = 10

In [82]: rate = 88.46

In [83]: currency = "Pesos"

In [84]: result = f"{amount} {currency} is worth US${amount / rate}"
```

可以使用与上述字符串模板相同的语法在每个表达式后添加格式说明符：

```python
In [85]: f"{amount} {currency} is worth US${amount / rate:.2f}"
Out[85]: '10 Pesos is worth US$0.11'
```

字符串格式化是一个深入的话题；有多种方法和无数选项和调整可用于控制值在结果字符串中的格式化方式。要了解更多信息，请查阅 [官方 Python 文档](https://docs.python.org/3/library/string.html)。

#### 字节和 Unicode

在现代 Python（即 Python 3.0 及以上版本）中，Unicode 已成为一等字符串类型，以便更一致地处理 ASCII 和非 ASCII 文本。在旧版本的 Python 中，字符串都是字节，没有任何显式的 Unicode 编码。如果您知道字符编码，可以转换为 Unicode。这是一个带有非 ASCII 字符的 Unicode 字符串示例：

```python
In [86]: val = "español"

In [87]: val
Out[87]: 'español'
```

我们可以使用 `encode` 方法将此 Unicode 字符串转换为其 UTF-8 字节表示形式：

```python
In [88]: val_utf8 = val.encode("utf-8")

In [89]: val_utf8
Out[89]: b'espa\xc3\xb1ol'

In [90]: type(val_utf8)
Out[90]: bytes
```

假设您知道 `bytes` 对象的 Unicode 编码，您可以使用 `decode` 方法返回：

```python
In [91]: val_utf8.decode("utf-8")
Out[91]: 'español'
```

虽然现在更倾向于对任何编码使用 UTF-8，但由于历史原因，您可能会遇到许多不同编码的数据：

```python
In [92]: val.encode("latin1")
Out[92]: b'espa\xf1ol'

In [93]: val.encode("utf-16")
Out[93]: b'\xff\xfee\x00s\x00p\x00a\x00\xf1\x00o\x00l\x00'

In [94]: val.encode("utf-16le")
Out[94]: b'e\x00s\x00p\x00a\x00\xf1\x00o\x00l\x00'
```

最常见的情况是在处理文件时遇到 `bytes` 对象，在这种情况下可能不希望将所有数据隐式解码为 Unicode 字符串。

#### 布尔值

Python 中的两个布尔值写作 `True` 和 `False`。比较和其他条件表达式求值为 `True` 或 `False`。布尔值使用 `and` 和 `or` 关键字组合：

```python
In [95]: True and True
Out[95]: True

In [96]: False or True
Out[96]: True
```

当转换为数字时，`False` 变为 `0`，`True` 变为 `1`：

```python
In [97]: int(False)
Out[97]: 0

In [98]: int(True)
Out[98]: 1
```

关键字 `not` 将布尔值从 `True` 翻转为 `False`，反之亦然：

```python
In [99]: a = True

In [100]: b = False

In [101]: not a
Out[101]: False

In [102]: not b
Out[102]: True
```

#### 类型转换

`str`、`bool`、`int` 和 `float` 类型也是函数，可用于将值强制转换为这些类型：

```python
In [103]: s = "3.14159"

In [104]: fval = float(s)

In [105]: type(fval)
Out[105]: float

In [106]: int(fval)
Out[106]: 3

In [107]: bool(fval)
Out[107]: True

In [108]: bool(0)
Out[108]: False
```

请注意，大多数非零值在转换为 `bool` 时会变为 `True`。

#### None

`None` 是 Python 的空值类型：

```python
In [109]: a = None

In [110]: a is None
Out[110]: True

In [111]: b = 5

In [112]: b is not None
Out[112]: True
```

`None` 也是函数参数的常见默认值：

```python
def add_and_maybe_multiply(a, b, c=None):
    result = a + b

    if c is not None:
        result = result * c

    return result
```

#### 日期和时间

内置的 Python `datetime` 模块提供了 `datetime`、`date` 和 `time` 类型。`datetime` 类型结合了 `date` 和 `time` 中存储的信息，是最常用的：

```python
In [113]: from datetime import datetime, date, time

In [114]: dt = datetime(2011, 10, 29, 20, 30, 21)

In [115]: dt.day
Out[115]: 29

In [116]: dt.minute
Out[116]: 30
```

给定一个 `datetime` 实例，您可以通过调用同名的方法来提取等效的 `date` 和 `time` 对象：

```python
In [117]: dt.date()
Out[117]: datetime.date(2011, 10, 29)

In [118]: dt.time()
Out[118]: datetime.time(20, 30, 21)
```

`strftime` 方法将 `datetime` 格式化为字符串：

```python
In [119]: dt.strftime("%Y-%m-%d %H:%M")
Out[119]: '2011-10-29 20:30'
```

字符串可以使用 `strptime` 函数转换（解析）为 `datetime` 对象：

```python
In [120]: datetime.strptime("20091031", "%Y%m%d")
Out[120]: datetime.datetime(2009, 10, 31, 0, 0)
```

有关格式规范的完整列表，请参见 [表 11.2](https://wesmckinney.com/book/time-series#tbl-table_datetime_formatting)。

当您聚合或以其他方式分组时间序列数据时，有时替换一系列 `datetime` 的时间字段会很有用——例如，将 `minute` 和 `second` 字段替换为零：

```python
In [121]: dt_hour = dt.replace(minute=0, second=0)

In [122]: dt_hour
Out[122]: datetime.datetime(2011, 10, 29, 20, 0)
```

由于 `datetime.datetime` 是不可变类型，因此此类方法总是产生新对象。所以在前面的例子中，`dt` 不会被 `replace` 修改：

```python
In [123]: dt
Out[123]: datetime.datetime(2011, 10, 29, 20, 30, 21)
```

两个 `datetime` 对象的差产生一个 `datetime.timedelta` 类型：

```python
In [124]: dt2 = datetime(2011, 11, 15, 22, 30)

In [125]: delta = dt2 - dt

In [126]: delta
Out[126]: datetime.timedelta(days=17, seconds=7179)

In [127]: type(delta)
Out[127]: datetime.timedelta
```

输出 `timedelta(17, 7179)` 表示 `timedelta` 编码了 17 天和 7,179 秒的偏移量。

将 `timedelta` 添加到 `datetime` 会产生一个新的偏移后的 `datetime`：

```python
In [128]: dt
Out[128]: datetime.datetime(2011, 10, 29, 20, 30, 21)

In [129]: dt + delta
Out[129]: datetime.datetime(2011, 11, 15, 22, 30)
```

### 控制流

Python 有几个内置关键字，用于条件逻辑、循环和其他编程语言中常见的标准**控制流（control flow）** 概念。

#### if、elif 和 else

`if` 语句是最著名的控制流语句类型之一。它检查一个条件，如果为 `True`，则评估后续代码块中的代码：

```python
x = -5
if x < 0:
    print("It's negative")
```

`if` 语句后面可以选择跟随一个或多个 `elif` 块和一个包罗万象的 `else` 块，如果所有条件都为 `False`：

```python
if x < 0:
    print("It's negative")
elif x == 0:
    print("Equal to zero")
elif 0 < x < 5:
    print("Positive but smaller than 5")
else:
    print("Positive and larger than or equal to 5")
```

如果任何条件为 `True`，则不会到达后续的 `elif` 或 `else` 块。对于使用 `and` 或 `or` 的复合条件，条件从左到右求值并且会短路：

```python
In [130]: a = 5; b = 7

In [131]: c = 8; d = 4

In [132]: if a < b or c > d:
   .....:     print("Made it")
Made it
```

在这个例子中，比较 `c > d` 从未被求值，因为第一个比较是 `True`。

也可以链式比较：

```python
In [133]: 4 > 3 > 2 > 1
Out[133]: True
```

#### for 循环

`for` 循环用于迭代集合（如列表或元组）或迭代器。`for` 循环的标准语法是：

```python
for value in collection:
    # 用 value 做一些事情
```

您可以使用 `continue` 关键字将 `for` 循环推进到下一次迭代，跳过块的其余部分。考虑这段代码，它对列表中的整数求和并跳过 `None` 值：

```python
sequence = [1, 2, None, 4, None, 5]
total = 0
for value in sequence:
    if value is None:
        continue
    total += value
```

可以使用 `break` 关键字完全退出 `for` 循环。这段代码对列表中的元素求和，直到遇到 5：

```python
sequence = [1, 2, 0, 4, 6, 5, 2, 1]
total_until_5 = 0
for value in sequence:
    if value == 5:
        break
    total_until_5 += value
```

`break` 关键字只终止最内层的 `for` 循环；任何外部的 `for` 循环将继续运行：

```python
In [134]: for i in range(4):
   .....:     for j in range(4):
   .....:         if j > i:
   .....:             break
   .....:         print((i, j))
   .....:
(0, 0)
(1, 0)
(1, 1)
(2, 0)
(2, 1)
(2, 2)
(3, 0)
(3, 1)
(3, 2)
(3, 3)
```

正如我们将更详细看到的，如果集合或迭代器中的元素是序列（例如元组或列表），它们可以在 `for` 循环语句中方便地**解包（unpacked）** 到变量中：

```python
for a, b, c in iterator:
    # 做一些事情
```

#### while 循环

`while` 循环指定一个条件和一段代码，该代码将一直执行，直到条件求值为 `False` 或使用 `break` 显式结束循环：

```python
x = 256
total = 0
while x > 0:
    if total > 500:
        break
    total += x
    x = x // 2
```

#### pass

`pass` 是 Python 中的“无操作”（或“什么都不做”）语句。它可用于不采取任何操作的块中（或作为尚未实现的代码的占位符）；之所以需要它，仅仅是因为 Python 使用空白来分隔块：

```python
if x < 0:
    print("negative!")
elif x == 0:
    # TODO: 在这里放一些聪明的东西
    pass
else:
    print("positive!")
```

#### range

`range` 函数生成一个均匀间隔的整数序列：

```python
In [135]: range(10)
Out[135]: range(0, 10)

In [136]: list(range(10))
Out[136]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
```

可以给出起始值、结束值和步长（可以是负数）：

```python
In [137]: list(range(0, 20, 2))
Out[137]: [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

In [138]: list(range(5, 0, -1))
Out[138]: [5, 4, 3, 2, 1]
```

如您所见，`range` 生成整数直到但不包括端点。`range` 的一个常见用法是通过索引迭代序列：

```python
In [139]: seq = [1, 2, 3, 4]

In [140]: for i in range(len(seq)):
   .....:     print(f"element {i}: {seq[i]}")
element 0: 1
element 1: 2
element 2: 3
element 3: 4
```

虽然您可以使用 `list` 等函数将 `range` 生成的所有整数存储在其他数据结构中，但通常默认的迭代器形式就是您想要的。这段代码对 0 到 99,999 之间所有是 3 或 5 的倍数的数字求和：

```python
In [141]: total = 0

In [142]: for i in range(100_000):
   .....:     # % 是取模运算符
   .....:     if i % 3 == 0 or i % 5 == 0:
   .....:         total += i

In [143]: print(total)
2333316668
```

虽然生成的范围可以任意大，但任何给定时间的内存使用量可能非常小。

## 2.4 结论

本章简要介绍了 Python 语言的一些基础概念以及 IPython 和 Jupyter 编程环境。在下一章中，我将详细讨论许多内置数据类型、函数和输入输出工具，这些内容将在本书后续章节中持续使用。