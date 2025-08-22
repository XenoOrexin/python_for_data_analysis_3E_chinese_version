# 附录B：深入IPython系统

```python
# 代码示例：IPython魔法命令
%timeit [x*x for x in range(1000)]
```

![IPython架构图](https://example.com/ipython_arch.png)

IPython系统提供交互式计算环境，支持[并行计算](https://ipython.org/parallel)和交互式可视化。通过`%run`命令可直接执行Python脚本，而`$ pip install ipython`可安装最新版本。

数学公式示例：
$$E = mc^2$$
和行内公式 $F = ma$

主要组件包括：
- IPython内核（IPython kernel）
- 交互式shell
- 并行计算架构

保持代码块不变：
```bash
#!/bin/bash
echo "Hello IPython"
```

术语对照：
- 魔法命令（Magic commands）
- 单元执行（Cell execution）
- 历史管理（History management）

__

《利用 Python 进行数据分析 第三版》的开放获取（Open Access）网络版本现已作为[印刷版与数字版](https://amzn.to/3DyLaJc)的配套资源发布。如果您发现书中存在错误，[请在此处提交勘误](https://oreilly.com/catalog/0636920519829/errata)。请注意，由于本网站通过 Quarto 生成，部分呈现形式会与 O’Reilly 出版的印刷版及电子书版本有所不同。

如果您觉得本书在线版对您有帮助，请考虑[订购纸质版本](https://amzn.to/3DyLaJc)或[无DRM限制的电子书](https://www.ebooks.com/en-us/book/210644288/python-for-data-analysis/wes-mckinney/?affId=WES398681F)以支持作者。网站内容不可复制或转载。代码示例遵循 MIT 许可协议，可在 GitHub 或 Gitee 上获取。

在[第二章：Python语言基础、IPython和Jupyter笔记本](https://wesmckinney.com/book/python-basics)中，我们探讨了使用IPython shell和Jupyter笔记本的基础知识。在本附录中，我们将深入介绍IPython系统的更多高级功能，这些功能既可在控制台中使用，也适用于Jupyter环境。

## B.1 终端键盘快捷键

IPython 拥有许多键盘快捷键，可用于在提示符间导航（对于 Emacs 文本编辑器或 Unix bash shell 的用户来说会感到熟悉）以及与 shell 的命令历史记录交互。表 B.1 总结了一些最常用的快捷键。关于其中一些快捷键（例如光标移动）的图示，请参见图 B.1。

表 B.1：标准 IPython 键盘快捷键

| 键盘快捷键              | 描述                                                                 |
| :---------------------- | :------------------------------------------------------------------- |
| Ctrl-P 或上箭头         | 在命令历史记录中向后搜索以当前输入文本开头的命令                     |
| Ctrl-N 或下箭头         | 在命令历史记录中向前搜索以当前输入文本开头的命令                     |
| Ctrl-R                  | Readline 风格的反向历史搜索（部分匹配）                                |
| Ctrl-Shift-V            | 从剪贴板粘贴文本                                                     |
| Ctrl-C                  | 中断当前正在执行的代码                                               |
| Ctrl-A                  | 将光标移动到行首                                                     |
| Ctrl-E                  | 将光标移动到行尾                                                     |
| Ctrl-K                  | 删除从光标到行尾的文本                                               |
| Ctrl-U                  | 丢弃当前行的所有文本                                                 |
| Ctrl-F                  | 将光标向前移动一个字符                                               |
| Ctrl-B                  | 将光标向后移动一个字符                                               |
| Ctrl-L                  | 清屏                                                                 |

![](images/pda3_b001.png)

图 B.1：IPython shell 中部分键盘快捷键的图示

请注意，Jupyter notebooks 拥有另一套主要用于导航和编辑的键盘快捷键。由于这些快捷键的发展比 IPython 中的更为快速，建议您探索 Jupyter notebook 菜单中的集成帮助系统。

## B.2 关于魔术命令 (Magic Commands)

IPython 中的特殊命令（并非 Python 内置）被称为*魔术*命令 (magic commands)。这些命令旨在简化常见任务，并让你能轻松控制 IPython 系统的行为。任何以百分号 `%` 为前缀的命令都是魔术命令。例如，你可以使用 `%timeit` 魔术函数来检查任何 Python 语句（如矩阵乘法）的执行时间：

```python
In [20]: a = np.random.standard_normal((100, 100))

In [20]: %timeit np.dot(a, a)
92.5 µs ± 3.43 µs per loop (mean ± std. dev. of 7 runs, 10000 loops each)
```

魔术命令可以看作是在 IPython 系统中运行的命令行程序。其中许多命令还有额外的“命令行”选项，所有这些选项（正如你可能期望的那样）都可以使用 `?` 来查看：

```python
In [21]: %debug?
Docstring:
::

%debug [--breakpoint FILE:LINE] [statement [statement ...]]

Activate the interactive debugger.

This magic command support two ways of activating debugger.
One is to activate debugger before executing code.  This way, you
can set a break point, to step through the code from the point.
You can use this mode by giving statements to execute and optionally
a breakpoint.

The other one is to activate debugger in post-mortem mode.  You can
activate this mode simply running %debug without any argument.
If an exception has just occurred, this lets you inspect its stack
frames interactively.  Note that this will always work only on the last
traceback that occurred, so you must call this quickly after an
exception that you wish to inspect has fired, because if another one
occurs, it clobbers the previous one.

If you want IPython to automatically do this on every exception, see
the %pdb magic for more details.

.. versionchanged:: 7.3
When running code, user variables are no longer expanded,
the magic line is always left unmodified.

positional arguments:
  statement             Code to run in debugger. You can omit this in cell 
                        magic mode.

optional arguments:
  --breakpoint <FILE:LINE>, -b <FILE:LINE>
                        Set break point at LINE in FILE.
```

默认情况下，只要没有定义与同名魔术函数相同的变量，就可以不使用百分号来使用魔术函数。此功能称为*自动魔术* (automagic)，可以通过 `%automagic` 启用或禁用。

一些魔术函数的行为类似于 Python 函数，它们的输出可以分配给变量：

```python
In [22]: %pwd
Out[22]: '/home/wesm/code/pydata-book'

In [23]: foo = %pwd

In [24]: foo
Out[24]: '/home/wesm/code/pydata-book'
```

由于 IPython 的文档可以从系统内部访问，我鼓励你通过使用 `%quickref` 或 `%magic` 来探索所有可用的特殊命令。这些信息会显示在控制台的分页器 (pager) 中，因此你需要按 `q` 键退出分页器。表 B.2 重点介绍了在 IPython 中进行交互式计算和 Python 开发时最常用的一些关键命令。

**表 B.2：一些常用的 IPython 魔术命令**

| 命令                      | 描述                                                                 |
| :------------------------ | :------------------------------------------------------------------- |
| `%quickref`               | 显示 IPython 快速参考卡                                                |
| `%magic`                  | 显示所有可用魔术命令的详细文档                                           |
| `%debug`                  | 在最后一个异常回溯 (traceback) 的底部进入交互式调试器                     |
| `%hist`                   | 打印命令输入（以及可选的输出）历史记录                                     |
| `%pdb`                    | 在任何异常后自动进入调试器                                               |
| `%paste`                  | 执行从剪贴板获取的预格式化 Python 代码                                    |
| `%cpaste`                 | 打开一个特殊提示符，用于手动粘贴要执行的 Python 代码                        |
| `%reset`                  | 删除在交互式命名空间中定义的所有变量/名称                                  |
| `%page` <OBJECT>          | 漂亮打印 (pretty-print) 对象并通过分页器显示                             |
| `%run` <script.py>        | 在 IPython 内部运行 Python 脚本                                        |
| `%prun` <statement>       | 使用 `cProfile` 执行 <statement> 并报告分析器 (profiler) 输出          |
| `%time` <statement>       | 报告单个语句的执行时间                                                   |
| `%timeit` <statement>     | 多次运行一条语句以计算整体平均执行时间；适用于执行时间非常短的代码计时             |
| `%who, %who_ls, %whos`    | 显示在交互式命名空间中定义的变量，提供不同级别的信息/详细程度                      |
| `%xdel` <variable>        | 删除变量并尝试清除 IPython 内部对该对象的任何引用                              |

### %run 命令

你可以使用 `%run` 命令在 IPython 会话环境中将任何文件作为 Python 程序运行。假设你在 *script.py* 中存储了以下简单脚本：

```python
def f(x, y, z):
    return (x + y) / z

a = 5
b = 6
c = 7.5

result = f(a, b, c)
```

你可以通过将文件名传递给 `%run` 来执行它：

```python
In [14]: %run script.py
```

该脚本在一个*空命名空间*（没有导入或其他已定义的变量）中运行，因此其行为应与使用 `python script.py` 在命令行上运行程序相同。然后，该文件中定义的所有变量（导入、函数和全局变量）（直到发生异常，如果有）都将在 IPython shell 中可访问：

```python
In [15]: c
Out [15]: 7.5

In [16]: result
Out[16]: 1.4666666666666666
```

如果 Python 脚本需要命令行参数（可在 `sys.argv` 中找到），则可以在文件路径之后传递这些参数，就像在命令行上运行一样。

> **注意**
> 如果你希望让脚本访问已在交互式 IPython 命名空间中定义的变量，请使用 `%run -i` 而不是普通的 `%run`。

在 Jupyter notebook 中，你也可以使用相关的 `%load` 魔术函数，它将脚本导入到一个代码单元格中：

```python
In [16]: %load script.py

    def f(x, y, z):
        return (x + y) / z

    a = 5
    b = 6
    c = 7.5

    result = f(a, b, c)
```

#### 中断正在运行的代码

在任何代码运行时（无论是通过 `%run` 运行的脚本还是长时间运行的命令），按 Ctrl-C 都会引发 `KeyboardInterrupt`。这将导致几乎所有 Python 程序立即停止，除非在某些特殊情况下。

> **警告**
> 当一段 Python 代码调用了一些编译后的扩展模块时，按 Ctrl-C 并不总是能立即停止程序执行。在这种情况下，你必须等待控制权返回给 Python 解释器，或者在更严重的情况下，强制终止操作系统中的 Python 进程（例如在 Windows 上使用任务管理器，或在 Linux 上使用 `kill` 命令）。

### 从剪贴板执行代码

如果你正在使用 Jupyter notebook，可以将代码复制并粘贴到任何代码单元格中并执行。在 IPython shell 中也可以运行剪贴板中的代码。假设你在其他应用程序中有以下代码：

```python
x = 5
y = 7
if x > 5:
    x += 1
    y = 8
```

最可靠的方法是使用 `%paste` 和 `%cpaste` 魔术函数（注意，这些在 Jupyter 中不起作用，因为你可以直接复制粘贴到 Jupyter 代码单元格中）。`%paste` 获取剪贴板中的任何文本，并在 shell 中将其作为单个代码块执行：

```python
In [17]: %paste
x = 5
y = 7
if x > 5:
    x += 1

    y = 8
## -- End pasted text --
```

`%cpaste` 类似，但它会提供一个特殊的提示符用于粘贴代码：

```python
In [18]: %cpaste
Pasting code; enter '--' alone on the line to stop or use Ctrl-D.
:x = 5
:y = 7
:if x > 5:
:    x += 1
:
:    y = 8
:--
```

使用 `%cpaste` 块，你可以在执行前自由粘贴任意多的代码。你可能会决定在使用 `%cpaste` 执行代码前先查看粘贴的代码。如果不小心粘贴了错误的代码，可以通过按 Ctrl-C 退出 `%cpaste` 提示符。

## B.3 使用命令历史（Command History）

IPython 维护了一个小型磁盘数据库，其中包含您执行的每条命令的文本。这用于多种目的：

- 通过最少的输入来搜索、补全和执行先前执行的命令
- 在不同会话之间持久化命令历史
- 将输入/输出历史记录到文件中

这些功能在 shell 中比在 notebook 中更有用，因为 notebook 在设计上会记录每个代码单元格的输入和输出。

### 搜索和重用命令历史

IPython shell 允许您搜索并执行之前的代码或其他命令。这非常有用，因为您可能经常发现自己重复执行相同的命令，例如 `%run` 命令或其他代码片段。假设您运行了：

```python
In[7]: %run first/second/third/data_script.py
```

然后探索了脚本的结果（假设它成功运行），却发现计算有误。在找出问题并修改了 _data_script.py_ 之后，您可以输入 `%run` 命令的几个字母，然后按下 Ctrl-P 组合键或上箭头键。这将搜索命令历史，查找与您键入的字母匹配的第一个先前命令。多次按下 Ctrl-P 或上箭头键将继续在历史记录中搜索。如果您错过了要执行的命令，别担心。您可以通过按下 Ctrl-N 或下箭头键在命令历史中向前移动。这样操作几次后，您可能会不假思索地按下这些键！

使用 Ctrl-R 可以提供与 Unix 风格 shell（例如 bash shell）中使用的 `readline` 相同的部分增量搜索功能。在 Windows 上，IPython 模拟了 `readline` 功能。要使用此功能，请按下 Ctrl-R，然后键入要搜索的输入行中包含的几个字符：

```python
In [1]: a_command = foo(x, y, z)

(reverse-i-search)`com': a_command = foo(x, y, z)
```

按下 Ctrl-R 将循环遍历历史记录中的每一行，匹配您键入的字符。

### 输入和输出变量

忘记将函数调用的结果分配给变量可能会非常令人烦恼。IPython 会话在特殊变量中存储了对输入命令和输出 Python 对象的引用。前两个输出分别存储在 `_`（一个下划线）和 `__`（两个下划线）变量中：

```python
In [18]: 'input1'
Out[18]: 'input1'

In [19]: 'input2'
Out[19]: 'input2'

In [20]: __
Out[20]: 'input1'

In [21]: 'input3'
Out[21]: 'input3'

In [22]: _
Out[22]: 'input3'__
```

输入变量存储在名为 `_iX` 的变量中，其中 `X` 是输入行号。

每个输入变量都有一个对应的输出变量 `_X`。因此，例如在输入行 27 之后，会出现两个新变量：`_27`（用于输出）和 `_i27`（用于输入）：

```python
In [26]: foo = 'bar'

In [27]: foo
Out[27]: 'bar'

In [28]: _i27
Out[28]: u'foo'

In [29]: _27
Out[29]: 'bar'__
```

由于输入变量是字符串，可以使用 Python 的 `eval` 关键字再次执行它们：

```python
In [30]: eval(_i27)
Out[30]: 'bar'__
```

这里，`_i27` 指的是在 `In [27]` 中输入的代码。

有几个魔术函数（magic functions）允许您处理输入和输出历史。`%hist` 可以打印全部或部分输入历史，带或不带行号。`%reset` 可以清除交互式命名空间，以及可选的输入和输出缓存。魔术函数 `%xdel` 可以从 IPython 机制中移除对特定对象的所有引用。有关这些魔术函数的更多详细信息，请参阅文档。

__

警告

在处理非常大的数据集时，请注意，IPython 的输入和输出历史可能会导致其中引用的对象无法被垃圾回收（释放内存），即使您使用 `del` 关键字从交互式命名空间中删除了变量。在这种情况下，谨慎使用 `%xdel` 和 `%reset` 可以帮助您避免内存问题。

## B.4 与操作系统交互

IPython 的另一项特性是允许你访问文件系统和操作系统外壳（shell）。这意味着，除其他功能外，你可以在不退出 IPython 的情况下执行大多数标准命令行操作，就像在 Windows 或 Unix（Linux、macOS）外壳中一样。这包括执行 shell 命令、更改目录以及将命令结果存储到 Python 对象（列表或字符串）中。此外，还提供了命令别名和目录书签功能。

有关调用 shell 命令的魔术命令（magic functions）和语法摘要，请参见表 B.3。我将在接下来的几节中简要介绍这些功能。

表 B.3：IPython 系统相关命令  
命令 | 描述  
---|---  
`!cmd` | 在系统 shell 中执行 `cmd`  
`output = !cmd args` | 运行 `cmd` 并将标准输出（stdout）存储到 `output` 中  
`%alias alias_name cmd` | 为系统（shell）命令定义别名  
`%bookmark` | 使用 IPython 的目录书签系统  
`%cd` <directory> | 将系统工作目录更改为指定目录  
`%pwd` | 返回当前系统工作目录  
`%pushd` <directory> | 将当前目录压入堆栈并切换到目标目录  
`%popd` | 切换到堆栈顶部弹出的目录  
`%dirs` | 返回包含当前目录堆栈的列表  
`%dhist` | 打印已访问目录的历史记录  
`%env` | 以字典形式返回系统环境变量  
`%matplotlib` | 配置 matplotlib 集成选项  

### Shell 命令和别名

在 IPython 中，以感叹号 `!`（也称为 bang）开头的一行表示要求 IPython 在系统 shell 中执行感叹号之后的所有内容。这意味着你可以删除文件（根据操作系统使用 `rm` 或 `del`）、更改目录或执行任何其他进程。

通过将转义 `!` 的表达式赋值给变量，可以将 shell 命令的控制台输出存储到变量中。例如，在我通过以太网连接互联网的基于 Linux 的机器上，我可以将 IP 地址获取为 Python 变量：
    
    
    In [1]: ip_info = !ifconfig wlan0 | grep "inet "
    
    In [2]: ip_info[0].strip()
    Out[2]: 'inet addr:10.0.0.11  Bcast:10.0.0.255  Mask:255.255.255.0'

返回的 Python 对象 `ip_info` 实际上是一个自定义列表类型，包含控制台输出的各种版本。

在使用 `!` 时，IPython 还可以替换当前环境中定义的 Python 值。为此，请在变量名前加上美元符号 `$`：
    
    
    In [3]: foo = 'test*'
    
    In [4]: !ls $foo
    test4.py  test.py  test.xml

魔术命令 `%alias` 可以为 shell 命令定义自定义快捷方式。例如：
    
    
    In [1]: %alias ll ls -l
    
    In [2]: ll /usr
    total 332
    drwxr-xr-x   2 root root  69632 2012-01-29 20:36 bin/
    drwxr-xr-x   2 root root   4096 2010-08-23 12:05 games/
    drwxr-xr-x 123 root root  20480 2011-12-26 18:08 include/
    drwxr-xr-x 265 root root 126976 2012-01-29 20:36 lib/
    drwxr-xr-x  44 root root  69632 2011-12-26 18:08 lib32/
    lrwxrwxrwx   1 root root      3 2010-08-23 16:02 lib64 -> lib/
    drwxr-xr-x  15 root root   4096 2011-10-13 19:03 local/
    drwxr-xr-x   2 root root  12288 2012-01-12 09:32 sbin/
    drwxr-xr-x 387 root root  12288 2011-11-04 22:53 share/
    drwxrwsr-x  24 root src    4096 2011-07-17 18:38 src/

你可以像在命令行中一样，通过用分号分隔多个命令来执行它们：
    
    
    In [558]: %alias test_alias (cd examples; ls; cd ..)
    
    In [559]: test_alias
    macrodata.csv  spx.csv  tips.csv

你会注意到，IPython 在会话关闭后会立即“忘记”你以交互方式定义的任何别名。要创建永久别名，你需要使用配置系统。

### 目录书签系统

IPython 提供了一个目录书签系统，使你可以为常用目录保存别名，从而轻松跳转。例如，假设你想创建一个指向本书补充材料的书签：
    
    
    In [6]: %bookmark py4da /home/wesm/code/pydata-book

完成此操作后，当你使用 `%cd` 魔术命令时，可以使用已定义的任何书签：
    
    
    In [7]: cd py4da
    (bookmark:py4da) -> /home/wesm/code/pydata-book
    /home/wesm/code/pydata-book

如果书签名称与当前工作目录中的目录名称冲突，可以使用 `-b` 标志覆盖并使用书签位置。使用 `%bookmark` 时加上 `-l` 选项可以列出所有书签：
    
    
    In [8]: %bookmark -l
    Current bookmarks:
    py4da -> /home/wesm/code/pydata-book-source

与别名不同，书签会在 IPython 会话之间自动持久化。

## B.5 软件开发工具 (Software Development Tools)

除了为交互式计算和数据探索提供舒适环境外，IPython 也可作为通用 Python 软件开发的实用伙伴。在数据分析应用中，首要的是拥有*正确*的代码。幸运的是，IPython 紧密集成并增强了 Python 内置的 `pdb` 调试器。其次，你希望代码*快速*运行。为此，IPython 提供了便捷的集成代码计时和分析工具。这里我将详细概述这些工具。

### 交互式调试器 (Interactive Debugger)

IPython 的调试器通过标签补全、语法高亮以及异常回溯中每一行的上下文信息，增强了 `pdb`。调试代码的最佳时机之一是在错误刚发生后。在异常发生后立即输入 `%debug` 命令，会调用“事后”调试器，并将你置于引发异常的堆栈帧中：

```python
In [2]: run examples/ipython_bug.py
---------------------------------------------------------------------------
AssertionError                            Traceback (most recent call last)
/home/wesm/code/pydata-book/examples/ipython_bug.py in <module>()
     13     throws_an_exception()
     14
---> 15 calling_things()

/home/wesm/code/pydata-book/examples/ipython_bug.py in calling_things()
     11 def calling_things():
     12     works_fine()
---> 13     throws_an_exception()
     14
     15 calling_things()

/home/wesm/code/pydata-book/examples/ipython_bug.py in throws_an_exception()
     7     a = 5
     8     b = 6
----> 9     assert(a + b == 10)
     10
     11 def calling_things()

AssertionError:

In [3]: %debug
> /home/wesm/code/pydata-book/examples/ipython_bug.py(9)throws_an_exception()
     8     b = 6
----> 9     assert(a + b == 10)
     10

ipdb>
```

进入调试器后，你可以执行任意的 Python 代码，并探索每个堆栈帧内的所有对象和数据（这些数据由解释器“保持存活”）。默认情况下，你处于错误发生的最低层级。通过输入 `u`（向上）和 `d`（向下），你可以在堆栈跟踪的不同层级间切换：

```python
ipdb> u
> /home/wesm/code/pydata-book/examples/ipython_bug.py(13)calling_things()
     12     works_fine()
---> 13     throws_an_exception()
     14
```

执行 `%pdb` 命令可以让 IPython 在任何异常后自动调用调试器，许多用户会发现这种模式非常有用。

在开发代码时使用调试器也很有帮助，尤其是当你需要设置断点或逐步执行函数或脚本以检查其在每一步的行为时。有几种方法可以实现这一点。第一种是使用带 `-d` 标志的 `%run` 命令，它会在执行传入脚本中的任何代码之前调用调试器。你必须立即输入 `s`（步进）以进入脚本：

```python
In [5]: run -d examples/ipython_bug.py
Breakpoint 1 at /home/wesm/code/pydata-book/examples/ipython_bug.py:1
NOTE: Enter 'c' at the ipdb>  prompt to start your script.
> <string>(1)<module>()

ipdb> s
--Call--
> /home/wesm/code/pydata-book/examples/ipython_bug.py(1)<module>()
1---> 1 def works_fine():
     2     a = 5
     3     b = 6
```

此后，如何逐步执行文件就取决于你了。例如，在上述异常中，我们可以在调用 `works_fine` 函数之前设置一个断点，然后通过输入 `c`（继续）来运行脚本直到断点处：

```python
ipdb> b 12
ipdb> c
> /home/wesm/code/pydata-book/examples/ipython_bug.py(12)calling_things()
     11 def calling_things():
2--> 12     works_fine()
     13     throws_an_exception()
```

此时，你可以 `step` 进入 `works_fine()`，或者通过输入 `n`（下一步）来执行 `works_fine()` 并前进到下一行：

```python
ipdb> n
> /home/wesm/code/pydata-book/examples/ipython_bug.py(13)calling_things()
2    12     works_fine()
---> 13     throws_an_exception()
     14
```

然后，我们可以步进到 `throws_an_exception` 中，前进到发生错误的行，并查看该作用域内的变量。注意，调试器命令优先于变量名；在这种情况下，请在变量前加上 `!` 来检查其内容：

```python
ipdb> s
--Call--
> /home/wesm/code/pydata-book/examples/ipython_bug.py(6)throws_an_exception()
     5
----> 6 def throws_an_exception():
     7     a = 5

ipdb> n
> /home/wesm/code/pydata-book/examples/ipython_bug.py(7)throws_an_exception()
     6 def throws_an_exception():
----> 7     a = 5
     8     b = 6

ipdb> n
> /home/wesm/code/pydata-book/examples/ipython_bug.py(8)throws_an_exception()
     7     a = 5
----> 8     b = 6
     9     assert(a + b == 10)

ipdb> n
> /home/wesm/code/pydata-book/examples/ipython_bug.py(9)throws_an_exception()
     8     b = 6
----> 9     assert(a + b == 10)
     10

ipdb> !a
5
ipdb> !b
6
```

根据我的经验，熟练使用交互式调试器需要时间和练习。表 B.4 提供了调试器命令的完整目录。如果你习惯使用 IDE，可能会觉得终端驱动的调试器起初有些难以适应，但随着时间的推移会有所改善。一些 Python IDE 具有出色的 GUI 调试器，因此大多数用户都能找到适合自己的工具。

**表 B.4: Python 调试器命令**

| 命令 | 操作 |
| :--- | :--- |
| `h(elp)` | 显示命令列表 |
| `help` <command> | 显示 <command> 的文档 |
| `c(ontinue)` | 恢复程序执行 |
| `q(uit)` | 退出调试器，不再执行任何代码 |
| `b(reak)` <number> | 在当前文件的 <number> 行设置断点 |
| `b` <path/to/file.py:number> | 在指定文件的 <number> 行设置断点 |
| `s(tep)` | 步进*到*函数调用中 |
| `n(ext)` | 执行当前行并前进到当前层级的下一行 |
| `u(p)`/`d(own)` | 在函数调用堆栈中向上/向下移动 |
| `a(rgs)` | 显示当前函数的参数 |
| `debug` <statement> | 在新的（递归）调试器中调用语句 <statement> |
| `l(ist)` <statement> | 显示当前堆栈层级的当前位置和上下文 |
| `w(here)` | 打印当前位置的完整堆栈跟踪及上下文 |

#### 使用调试器的其他方法

还有其他几种有用的调用调试器的方法。第一种是使用一个特殊的 `set_trace` 函数（以 `pdb.set_trace` 命名），它基本上是一个“简易断点”。以下是两个小技巧，你可能希望将其放在某处供日常使用（像我一样，可以将其添加到 IPython 配置文件中）：

```python
from IPython.core.debugger import Pdb

def set_trace():
    Pdb(.set_trace(sys._getframe().f_back)

def debug(f, *args, **kwargs):
    pdb = Pdb()
    return pdb.runcall(f, *args, **kwargs)
```

第一个函数 `set_trace` 提供了一种方便的方法，可以在代码中的任意位置设置断点。你可以在代码的任何部分使用 `set_trace`，以便临时停止并更仔细地检查（例如，在异常发生之前）：

```python
In [7]: run examples/ipython_bug.py
> /home/wesm/code/pydata-book/examples/ipython_bug.py(16)calling_things()
     15     set_trace()
---> 16     throws_an_exception()
     17
```

输入 `c`（继续）将使代码正常恢复运行，不会造成任何损害。

我们刚才看到的 `debug` 函数使你可以轻松地在任意函数调用上调用交互式调试器。假设我们编写了如下函数，并希望逐步跟踪其逻辑：

```python
def f(x, y, z=1):
    tmp = x + y
    return tmp / z
```

通常使用 `f` 的方式是 `f(1, 2, z=3)`。要逐步进入 `f`，请将 `f` 作为第一个参数传递给 `debug`，后面跟着要传递给 `f` 的位置参数和关键字参数：

```python
In [6]: debug(f, 1, 2, z=3)
> <ipython-input>(2)f()
     1 def f(x, y, z):
----> 2     tmp = x + y
     3     return tmp / z

ipdb>
```

这两个技巧多年来为我节省了大量时间。

最后，调试器可以与 `%run` 结合使用。通过使用 `%run -d` 运行脚本，你将直接进入调试器，准备设置任何断点并启动脚本：

```python
In [1]: %run -d examples/ipython_bug.py
Breakpoint 1 at /home/wesm/code/pydata-book/examples/ipython_bug.py:1
NOTE: Enter 'c' at the ipdb>  prompt to start your script.
> <string>(1)<module>()

ipdb>
```

添加 `-b` 和行号可以启动调试器并已设置好断点：

```python
In [2]: %run -d -b2 examples/ipython_bug.py
Breakpoint 1 at /home/wesm/code/pydata-book/examples/ipython_bug.py:2
NOTE: Enter 'c' at the ipdb>  prompt to start your script.
> <string>(1)<module>()

ipdb> c
> /home/wesm/code/pydata-book/examples/ipython_bug.py(2)works_fine()
     1 def works_fine():
1---> 2     a = 5
     3     b = 6

ipdb>
```

### 代码计时：%time 和 %timeit

对于大规模或长时间运行的数据分析应用，你可能希望测量各个组件或单个语句或函数调用的执行时间。你可能需要一份报告，显示在复杂过程中哪些函数占用了最多时间。幸运的是，IPython 使你在开发和测试代码时可以方便地获取这些信息。

使用内置的 `time` 模块及其函数 `time.clock` 和 `time.time` 手动计时代码通常既繁琐又重复，因为你必须编写相同的乏味样板代码：

```python
import time
start = time.time()
for i in range(iterations):
    # some code to run here
elapsed_per = (time.time() - start) / iterations
```

由于这是一个非常常见的操作，IPython 有两个魔术函数 `%time` 和 `%timeit` 来自动化这个过程。

`%time` 运行一条语句一次，报告总执行时间。假设我们有一个巨大的字符串列表，并且我们希望比较选择所有以特定前缀开头的字符串的不同方法。以下是一个包含 600,000 个字符串的列表，以及两种选择以 `'foo'` 开头的字符串的相同方法：

```python
# a very large list of strings
In [11]: strings = ['foo', 'foobar', 'baz', 'qux',
       ....:            'python', 'Guido Van Rossum'] * 100000

In [12]: method1 = [x for x in strings if x.startswith('foo')]

In [13]: method2 = [x for x in strings if x[:3] == 'foo']
```

看起来它们在性能上应该差不多，对吧？我们可以使用 `%time` 来确认：

```python
In [14]: %time method1 = [x for x in strings if x.startswith('foo')]
CPU times: user 49.6 ms, sys: 676 us, total: 50.3 ms
Wall time: 50.1 ms

In [15]: %time method2 = [x for x in strings if x[:3] == 'foo']
CPU times: user 40.3 ms, sys: 603 us, total: 40.9 ms
Wall time: 40.6 ms
```

`Wall time`（“挂钟时间”的缩写）是主要关注的数字。从这些计时中，我们可以推断出存在一些性能差异，但这不是一个非常精确的测量。如果你自己多次尝试 `%time` 这些语句，你会发现结果有些变化。为了获得更精确的测量，请使用 `%timeit` 魔术函数。给定任意语句，它采用启发式方法多次运行该语句以产生更准确的平均运行时间（这些结果在你的系统上可能有所不同）：

```python
In [563]: %timeit [x for x in strings if x.startswith('foo')]
10 loops, best of 3: 159 ms per loop

In [564]: %timeit [x for x in strings if x[:3] == 'foo']
10 loops, best of 3: 59.3 ms per loop
```

这个看似无害的例子说明，值得去了解 Python 标准库、NumPy、pandas 以及本书中使用的其他库的性能特性。在更大规模的数据分析应用中，那些毫秒会开始累积起来！

`%timeit` 对于分析执行时间非常短的语句和函数特别有用，即使是微秒（百万分之一秒）或纳秒（十亿分之一秒）级别。这些看似微不足道的时间，但当然，一个 20 微秒的函数调用 100 万次要比一个 5 微秒的函数多花 15 秒。在前面的例子中，我们可以非常直接地比较两种字符串操作以了解它们的性能特征：

```python
In [565]: x = 'foobar'

In [566]: y = 'foo'

In [567]: %timeit x.startswith(y)
1000000 loops, best of 3: 267 ns per loop

In [568]: %timeit x[:3] == y
10000000 loops, best of 3: 147 ns per loop
```

### 基本性能分析：%prun 和 %run -p

分析代码与计时代码密切相关，只是它关注的是确定时间*花费在哪里*。主要的 Python 分析工具是 `cProfile` 模块，它并非特定于 IPython。`cProfile` 在执行程序或任意代码块时，会跟踪每个函数花费的时间。

使用 `cProfile` 的一种常见方式是在命令行上，运行整个程序并输出每个函数的聚合时间。假设我们有一个脚本，在循环中执行一些线性代数操作（计算一系列 100 × 100 矩阵的最大绝对特征值）：

```python
import numpy as np
from numpy.linalg import eigvals

def run_experiment(niter=100):
    K = 100
    results = []
    for _ in range(niter):
        mat = np.random.standard_normal((K, K))
        max_eigenvalue = np.abs(eigvals(mat)).max()
        results.append(max_eigenvalue)
    return results
some_results = run_experiment()
print('Largest one we saw: {0}'.format(np.max(some_results)))
```

你可以在命令行中使用以下命令通过 `cProfile` 运行此脚本：

```bash
python -m cProfile cprof_example.py
```

如果你尝试这样做，你会发现输出是按函数名排序的。这使得有点难以了解大部分时间花费在哪里，因此使用 `-s` 标志指定*排序顺序*很有用：

```bash
$ python -m cProfile -s cumulative cprof_example.py
Largest one we saw: 11.923204422
    15116 function calls (14927 primitive calls) in 0.720 seconds

Ordered by: cumulative time

ncalls  tottime  percall  cumtime  percall filename:lineno(function)
     1    0.001    0.001    0.721    0.721 cprof_example.py:1(<module>)
   100    0.003    0.000    0.586    0.006 linalg.py:702(eigvals)
   200    0.572    0.003    0.572    0.003 {numpy.linalg.lapack_lite.dgeev}
     1    0.002    0.002    0.075    0.075 __init__.py:106(<module>)
   100    0.059    0.001    0.059    0.001 {method 'randn')
     1    0.000    0.000    0.044    0.044 add_newdocs.py:9(<module>)
     2    0.001    0.001    0.037    0.019 __init__.py:1(<module>)
     2    0.003    0.002    0.030    0.015 __init__.py:2(<module>)
     1    0.000    0.000    0.030    0.030 type_check.py:3(<module>)
     1    0.001    0.001    0.021    0.021 __init__.py:15(<module>)
     1    0.013    0.013    0.013    0.013 numeric.py:1(<module>)
     1    0.000    0.000    0.009    0.009 __init__.py:6(<module>)
     1    0.001    0.001    0.008    0.008 __init__.py:45(<module>)
   262    0.005    0.000    0.007    0.000 function_base.py:3178(add_newdoc)
   100    0.003    0.000    0.005    0.000 linalg.py:162(_assertFinite)
   ...
```

只显示了输出的前 15 行。最容易阅读的方式是扫描 `cumtime` 列，查看在每个函数*内部*花费了多少总时间。请注意，如果一个函数调用了其他函数，*时钟不会停止运行*。`cProfile` 记录每个函数调用的开始和结束时间，并以此产生计时。

除了命令行用法外，`cProfile` 也可以以编程方式使用，来分析任意代码块，而无需运行新进程。IPython 使用 `%prun` 命令和 `%run` 的 `-p` 选项提供了一个方便的接口。`%prun` 采用与 `cProfile` 相同的“命令行选项”，但将分析任意 Python 语句而不是整个 _.py_ 文件：

```python
In [4]: %prun -l 7 -s cumulative run_experiment()
         4203 function calls in 0.643 seconds

Ordered by: cumulative time
List reduced from 32 to 7 due to restriction <7>

ncalls  tottime  percall  cumtime  percall filename:lineno(function)
     1    0.000    0.000    0.643    0.643 <string>:1(<module>)
     1    0.001    0.001    0.643    0.643 cprof_example.py:4(run_experiment)
   100    0.003    0.000    0.583    0.006 linalg.py:702(eigvals)
   200    0.569    0.003    0.569    0.003 {numpy.linalg.lapack_lite.dgeev}
   100    0.058    0.001    0.058    0.001 {method 'randn'}
   100    0.003    0.000    0.005    0.000 linalg.py:162(_assertFinite)
   200    0.002    0.000    0.002    0.000 {method 'all' of 'numpy.ndarray'}
```

类似地，调用 `%run -p -s cumulative cprof_example.py` 与命令行方法具有相同的效果，只是你无需离开 IPython。

在 Jupyter notebook 中，你可以使用 `%%prun` 魔术（两个 `%` 符号）来分析整个代码块。这会弹出一个带有分析输出的单独窗口。这对于快速回答诸如“为什么那个代码块运行了那么久？”之类的问题非常有用。

还有其他工具可用，可以帮助你在使用 IPython 或 Jupyter 时更轻松地理解分析结果。其中之一是 [SnakeViz](https://github.com/jiffyclub/snakeviz/)，它使用 D3.js 生成分析结果的交互式可视化。

### 逐行分析函数性能

在某些情况下，从 `%prun`（或其他基于 `cProfile` 的分析方法）获得的信息可能无法完整说明函数的执行时间，或者可能过于复杂，导致按函数名聚合的结果难以解释。对于这种情况，有一个名为 `line_profiler` 的小型库（可通过 PyPI 或包管理工具获得）。它包含一个 IPython 扩展，启用了一个新的魔术函数 `%lprun`，可以计算一个或多个函数的逐行分析。你可以通过修改 IPython 配置（参见 IPython 文档或本附录后面的配置部分）来包含以下行，以启用此扩展：

```python
# A list of dotted module names of IPython extensions to load.
c.InteractiveShellApp.extensions = ['line_profiler']
```

你也可以运行命令：

```python
%load_ext line_profiler
```

`line_profiler` 可以以编程方式使用（参见完整文档），但在 IPython 中以交互方式使用时可能最强大。假设你有一个模块 `prof_mod`，包含以下执行一些 NumPy 数组操作的代码（如果你想重现此示例，请将此代码放入新文件 _prof_mod.py_ 中）：

```python
from numpy.random import randn

def add_and_sum(x, y):
    added = x + y
    summed = added.sum(axis=1)
    return summed

def call_function():
    x = randn(1000, 1000)
    y = randn(1000, 1000)
    return add_and_sum(x, y)
```

如果我们想了解 `add_and_sum` 函数的性能，`%prun` 给我们以下结果：

```python
In [569]: %run prof_mod

In [570]: x = randn(3000, 3000)

In [571]: y = randn(3000, 3000)

In [572]: %prun add_and_sum(x, y)
         4 function calls in 0.049 seconds
   Ordered by: internal time
   ncalls  tottime  percall  cumtime  percall filename:lineno(function)
        1    0.036    0.036    0.046    0.046 prof_mod.py:3(add_and_sum)
        1    0.009    0.009    0.009    0.009 {method 'sum' of 'numpy.ndarray'}
        1    0.003    0.003    0.049    0.049 <string>:1(<module>)
```

这并不特别有启发性。启用 `line_profiler` IPython 扩展后，可以使用新命令 `%lprun`。用法上的唯一区别是我们必须指示 `%lprun` 我们要分析哪个函数或哪些函数。一般语法是：

```python
%lprun -f func1 -f func2 statement_to_profile
```

在这种情况下，我们想要分析 `add_and_sum`，所以我们运行：

```python
In [573]: %lprun -f add_and_sum add_and_sum(x, y)
Timer unit: 1e-06 s
File: prof_mod.py
Function: add_and_sum at line 3
Total time: 0.045936 s
Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     3                                           def add_and_sum(x, y):
     4         1        36510  36510.0     79.5      added = x + y
     5         1         9425   9425.0     20.5      summed = added.sum(axis=1)
     6         1            1      1.0      0.0      return summed
```

这可能更容易解释。在这种情况下，我们分析了语句中使用的同一个函数。查看前面的模块代码，我们可以调用 `call_function` 并同时分析它和 `add_and_sum`，从而获得代码性能的完整图景：

```python
In [574]: %lprun -f add_and_sum -f call_function call_function()
Timer unit: 1e-06 s
File: prof_mod.py
Function: add_and_sum at line 3
Total time: 0.005526 s
Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     3                                           def add_and_sum(x, y):
     4         1         4375   4375.0     79.2      added = x + y
     5         1         1149   1149.0     20.8      summed = added.sum(axis=1)
     6         1            2      2.0      0.0      return summed
File: prof_mod.py
Function: call_function at line 8
Total time: 0.121016 s
Line #      Hits         Time  Per Hit   % Time  Line Contents
==============================================================
     8                                           def call_function():
     9         1        57169  57169.0     47.2      x = randn(1000, 1000)
    10         1        58304  58304.0     48.2      y = randn(1000, 1000)
    11         1         5543   5543.0      4.6      return add_and_sum(x, y)
```

根据一般经验，我倾向于使用 `%prun`（`cProfile`）进行“宏观”分析，使用 `%lprun`（`line_profiler`）进行“微观”分析。值得对这两种工具都有很好的理解。

> **注意**
> 
> 你必须使用 `%lprun` 显式指定要分析的函数名称的原因是，“跟踪”每行执行时间的开销很大。跟踪不感兴趣的函数可能会显著改变分析结果。

## B.6 使用 IPython 进行高效代码开发的技巧

以方便交互式开发、调试并最终交互式_使用_的方式编写代码，对许多用户而言可能是一种范式转变。这既涉及代码重载等需要适应的操作细节，也关乎代码风格方面的考量。

因此，实施本节所述的大部分策略更像是一门艺术而非科学，需要您亲自进行一些实验，以找到一种适合自身的高效 Python 代码编写方式。最终目标是构建一种便于迭代使用、并能尽可能轻松地探索程序或函数运行结果的代码结构。我发现，专为 IPython 设计的软件比仅作为独立命令行应用程序运行的代码更易于使用。当出现问题时，尤其是在诊断您或他人在数月或数年前编写的代码错误时，这一点变得尤为重要。

### 重载模块依赖项

在 Python 中，当输入 `import some_lib` 时，`some_lib` 中的代码会被执行，其中定义的所有变量、函数和导入都会存储在新创建的 `some_lib` 模块命名空间中。下次使用 `import some_lib` 时，您将获得对现有模块命名空间的引用。在交互式 IPython 代码开发中，潜在困难在于：当您 `%run` 一个依赖于其他模块的脚本时，可能已对那些模块进行了修改。假设 _test\_script.py_ 中有以下代码：

```python
import some_lib

x = 5
y = [1, 2, 3, 4]
result = some_lib.get_answer(x, y)
```

如果您执行 `%run test_script.py` 后修改了 _some\_lib.py_，那么下次执行 `%run test_script.py` 时，由于 Python 的“一次性加载”模块机制，您仍然会得到 _some\_lib.py_ 的_旧版本_。这种行为与其他一些数据分析环境（如 MATLAB）不同，后者会自动传播代码更改¹。为了解决这个问题，您有几个选择。第一种方法是使用标准库中 `importlib` 模块的 `reload` 函数：

```python
import some_lib
import importlib

importlib.reload(some_lib)
```

这会在每次运行 _test\_script.py_ 时尝试为您提供 _some\_lib.py_ 的新副本（但在某些情况下可能无法实现）。显然，如果依赖关系更深，到处插入 `reload` 的使用可能会有些棘手。针对此问题，IPython 提供了一个特殊的 `dreload` 函数（_非_魔法函数），用于模块的“深度”（递归）重载。如果在运行 _some\_lib.py_ 后使用 `dreload(some_lib)`，它将尝试重载 `some_lib` 及其所有依赖项。遗憾的是，这并非在所有情况下都有效，但当其生效时，相比重启 IPython 更为便捷。

### 代码设计技巧

这方面没有简单的诀窍，但以下是我在工作中发现有效的一些高层原则。

#### 保持相关对象和数据处于活动状态

看到为命令行编写的程序具有类似以下结构并不罕见：

```python
from my_functions import g

def f(x, y):
    return g(x + y)

def main():
    x = 6
    y = 7.5
    result = x + y

if __name__ == '__main__':
    main()
```

您是否看出在 IPython 中运行此程序可能遇到的问题？程序完成后，在 `main` 函数中定义的任何结果或对象都无法在 IPython shell 中访问。更好的方式是让 `main` 中的代码直接在模块的全局命名空间中执行（或者放在 `if __name__ == '__main__':` 块中，如果您还希望模块可被导入）。这样，当您 `%run` 代码时，就能查看在 `main` 中定义的所有变量。这相当于在 Jupyter notebook 的单元格中定义顶层变量。

#### 扁平化优于嵌套化

深度嵌套的代码让我想到洋葱的多层结构。在测试或调试函数时，需要剥开多少层洋葱才能触及目标代码？“扁平化优于嵌套化”是 Python 之禅的一部分，这一理念同样适用于交互式代码开发。尽可能使函数和类解耦和模块化，有助于更轻松地进行测试（如果您编写单元测试）、调试和交互式使用。

#### 克服对长文件的恐惧

如果您有 Java（或类似语言）背景，可能被告知要保持文件简短。在许多语言中，这是合理的建议；过长通常是一种不良的“代码异味”，表明可能需要重构或重组。然而，在使用 IPython 开发代码时，处理 10 个小型但互相关联的文件（例如每个少于 100 行）通常比处理 2 或 3 个较长的文件更令人头疼。更少的文件意味着需要重载的模块更少，编辑时在文件间的切换也更少。我发现维护较大的模块（每个模块具有较高的_内部_内聚性，即代码全部与解决同类问题相关）更加实用且符合 Python 风格。当然，在迭代找到解决方案后，有时将大文件重构为小文件是有意义的。

显然，我不支持将这一观点极端化，比如将所有代码放入一个庞大的文件中。为大型代码库找到合理且直观的模块和包结构通常需要一些功夫，但在团队协作中尤为重要。每个模块应内部内聚，并且应尽可能明显地找到负责各功能区域的函数和类。

## B.7 IPython 高级功能

要充分运用 IPython 系统，可能需要您以略微不同的方式编写代码，或深入探索其配置选项。

### 配置集与配置文件

IPython 和 Jupyter 环境的大多数外观特性（颜色、提示符、行间距等）和行为均可通过一套全面的配置系统进行自定义。以下是一些可通过配置实现的功能：

  * 更改配色方案
  * 修改输入输出提示符的外观，或消除 `Out` 输出后与下一个 `In` 提示符之间的空行
  * 执行任意 Python 语句列表（例如，您经常使用的导入语句，或任何希望每次启动 IPython 时自动执行的操作）
  * 启用常驻 IPython 扩展（例如 `line_profiler` 中的 `%lprun` 魔术命令）
  * 启用 Jupyter 扩展功能
  * 自定义魔术命令或系统别名

IPython shell 的配置通过特殊的 _ipython\_config.py_ 文件指定，该文件通常位于用户主目录的 _.ipython/_ 目录下。配置基于特定 _配置集（profile）_ 生效。当您正常启动 IPython 时，默认会加载存储在 _profile\_default_ 目录中的 _默认配置集（default profile）_ 。例如，在我的 Linux 系统上，默认 IPython 配置文件的完整路径为：
    
    
    /home/wesm/.ipython/profile_default/ipython_config.py

要在您的系统上初始化此文件，请在终端中运行：
    
    
    ipython profile create default

此处不再赘述该文件的完整细节。幸运的是，文件中的注释说明了每个配置选项的用途，因此我将留给读者自行调整和定制。另一个实用功能是支持 _多配置集（multiple profiles）_ 。假设您需要为特定应用或项目量身定制另一套 IPython 配置，创建新配置集需输入以下命令：
    
    
    ipython profile create secret_project

完成此操作后，编辑新创建的 _profile\_secret\_project_ 目录中的配置文件，然后按如下方式启动 IPython：
    
    
    $ ipython --profile=secret_project
    Python 3.8.0 | packaged by conda-forge | (default, Nov 22 2019, 19:11:19)
    Type 'copyright', 'credits' or 'license' for more information
    IPython 7.22.0 -- An enhanced Interactive Python. Type '?' for help.
    
    IPython profile: secret_project

一如既往，IPython 在线文档是关于配置集和配置功能的绝佳资源。

Jupyter 的配置方式略有不同，因为其笔记本（notebooks）可支持除 Python 之外的其他语言。要创建类似的 Jupyter 配置文件，请运行：
    
    
    jupyter notebook --generate-config

这会将默认配置文件写入您主目录下的 _.jupyter/jupyter\_notebook\_config.py_ 目录。根据需求编辑该文件后，您可以将其重命名为其他文件，例如：
    
    
    $ mv ~/.jupyter/jupyter_notebook_config.py ~/.jupyter/my_custom_config.py

启动 Jupyter 时，随后可添加 `--config` 参数：
    
    
    jupyter notebook --config=~/.jupyter/my_custom_config.py

## B.8 结论

在学习本书中的代码示例并提升 Python 编程技能的过程中，我鼓励你持续学习 IPython 和 Jupyter 生态系统（IPython and Jupyter ecosystems）。由于这些项目的设计初衷是提升用户效率，你可能会发现一些工具能够让你更轻松地完成工作，而不仅仅是单纯使用 Python 语言及其计算库（computational libraries）。

你还可以在 [nbviewer 网站](https://nbviewer.jupyter.org)上找到大量有趣的 Jupyter 笔记本。

* * *

  1. 由于模块或包可能在特定程序中的多个不同位置被导入，Python 会在首次导入时缓存模块的代码，而不是每次导入时都执行模块中的代码。否则，模块化（modularity）和良好的代码组织可能会导致应用程序效率低下。↩︎