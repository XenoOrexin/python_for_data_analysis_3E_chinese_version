# 3内置数据结构、函数和文件

__

《Python 数据分析 第3版》的这份开放获取网络版本现作为[印刷版与数字版](https://amzn.to/3DyLaJc)的配套资源提供。如果您发现任何勘误，[请在此处提交报告](https://oreilly.com/catalog/0636920519829/errata)。请注意，由 Quarto 生成的本站点某些呈现形式会与 O'Reilly 出版的印刷版及电子书版本有所不同。

如果您觉得本书在线版对您有帮助，请考虑[订购纸质版本](https://amzn.to/3DyLaJc)或[无DRM限制的电子书](https://www.ebooks.com/en-us/book/210644288/python-for-data-analysis/wes-mckinney/?affId=WES398681F)以支持作者。本站内容禁止复制或转载。代码示例遵循 MIT 许可证，可在 GitHub 或 Gitee 上获取。

本章将探讨 Python 语言内置的核心功能，这些功能在全书中将无处不在。虽然如 pandas 和 NumPy 等扩展库为处理大型数据集添加了高级计算功能，但它们的设计初衷是与 Python 内置的数据操作工具协同使用。

我们将从 Python 的核心数据结构开始：元组(tuples)、列表(lists)、字典(dictionaries)和集合(sets)。接着讨论如何创建可复用的 Python 函数。最后我们将深入了解 Python 文件对象的运作机制以及与本地硬盘的交互方式。

## 3.1 数据结构和序列

Python 的数据结构简单而强大。精通它们的使用是成为熟练 Python 程序员的关键部分。我们从元组（tuple）、列表（list）和字典（dictionary）开始，这些是最常用的 _序列_（sequence）类型。

### 元组（Tuple）

_元组_（tuple）是一个固定长度、不可变的 Python 对象序列。一旦赋值，就无法更改。创建元组最简单的方法是用括号括起来的逗号分隔的值序列：
    
    
    In [2]: tup = (4, 5, 6)
    
    In [3]: tup
    Out[3]: (4, 5, 6)__

在许多情况下，括号可以省略，因此这里我们也可以写成：
    
    
    In [4]: tup = 4, 5, 6
    
    In [5]: tup
    Out[5]: (4, 5, 6)__

你可以通过调用 `tuple` 将任何序列或迭代器转换为元组：
    
    
    In [6]: tuple([4, 0, 2])
    Out[6]: (4, 0, 2)
    
    In [7]: tup = tuple('string')
    
    In [8]: tup
    Out[8]: ('s', 't', 'r', 'i', 'n', 'g')__

与大多数其他序列类型一样，可以使用方括号 `[]` 访问元素。与 C、C++、Java 和许多其他语言一样，Python 中的序列索引从 0 开始：
    
    
    In [9]: tup[0]
    Out[9]: 's'__

当在更复杂的表达式中定义元组时，通常需要将值括在括号中，如以下创建元组元组的示例所示：
    
    
    In [10]: nested_tup = (4, 5, 6), (7, 8)
    
    In [11]: nested_tup
    Out[11]: ((4, 5, 6), (7, 8))
    
    In [12]: nested_tup[0]
    Out[12]: (4, 5, 6)
    
    In [13]: nested_tup[1]
    Out[13]: (7, 8)__

虽然存储在元组中的对象本身可能是可变的，但一旦创建了元组，就无法修改每个槽中存储的对象：
    
    
    In [14]: tup = tuple(['foo', [1, 2], True])
    
    In [15]: tup[2] = False
    ---------------------------------------------------------------------------
    TypeError                                 Traceback (most recent call last)
    <ipython-input-15-b89d0c4ae599> in <module>
    ----> 1 tup[2] = False
    TypeError: 'tuple' object does not support item assignment __

如果元组内的对象是可变的（例如列表），你可以就地修改它：
    
    
    In [16]: tup[1].append(3)
    
    In [17]: tup
    Out[17]: ('foo', [1, 2, 3], True)__

你可以使用 `+` 运算符连接元组以生成更长的元组：
    
    
    In [18]: (4, None, 'foo') + (6, 0) + ('bar',)
    Out[18]: (4, None, 'foo', 6, 0, 'bar')__

与列表一样，将元组乘以整数会产生连接该元组的多个副本的效果：
    
    
    In [19]: ('foo', 'bar') * 4
    Out[19]: ('foo', 'bar', 'foo', 'bar', 'foo', 'bar', 'foo', 'bar')__

请注意，对象本身不会被复制，只有对它们的引用会被复制。

#### 解包元组（Unpacking tuples）

如果你尝试为类似元组的变量表达式 _赋值_（assign），Python 会尝试 _解包_（unpack）等号右侧的值：
    
    
    In [20]: tup = (4, 5, 6)
    
    In [21]: a, b, c = tup
    
    In [22]: b
    Out[22]: 5 __

即使是包含嵌套元组的序列也可以解包：
    
    
    In [23]: tup = 4, 5, (6, 7)
    
    In [24]: a, b, (c, d) = tup
    
    In [25]: d
    Out[25]: 7 __

使用此功能，你可以轻松交换变量名称，这在许多语言中可能看起来像：
    
    
    tmp = a
    a = b
    b = tmp __

但是，在 Python 中，交换可以这样完成：
    
    
    In [26]: a, b = 1, 2
    
    In [27]: a
    Out[27]: 1
    
    In [28]: b
    Out[28]: 2
    
    In [29]: b, a = a, b
    
    In [30]: a
    Out[30]: 2
    
    In [31]: b
    Out[31]: 1 __

变量解包的一个常见用途是遍历元组或列表的序列：
    
    
    In [32]: seq = [(1, 2, 3), (4, 5, 6), (7, 8, 9)]
    
    In [33]: for a, b, c in seq:
       ....:     print(f'a={a}, b={b}, c={c}')
    a=1, b=2, c=3
    a=4, b=5, c=6
    a=7, b=8, c=9 __

另一个常见用途是从函数返回多个值。我将在后面更详细地介绍这一点。

在某些情况下，你可能想从元组的开头“提取”（pluck）几个元素。有一个特殊的语法可以做到这一点，即 `*rest`，它也用于函数签名中以捕获任意长的位置参数列表：
    
    
    In [34]: values = 1, 2, 3, 4, 5
    
    In [35]: a, b, *rest = values
    
    In [36]: a
    Out[36]: 1
    
    In [37]: b
    Out[37]: 2
    
    In [38]: rest
    Out[38]: [3, 4, 5]__

这个 `rest` 部分有时是你想丢弃的东西；`rest` 名称没有什么特别之处。按照惯例，许多 Python 程序员会使用下划线 \(`_`\) 来表示不需要的变量：
    
    
    In [39]: a, b, *_ = values __

#### 元组方法（Tuple methods）

由于元组的大小和内容无法修改，因此其实例方法非常少。一个特别有用的方法（也可用于列表）是 `count`，它计算某个值出现的次数：
    
    
    In [40]: a = (1, 2, 2, 2, 3, 4, 2)
    
    In [41]: a.count(2)
    Out[41]: 4 __

### 列表（List）

与元组相比，列表是可变长度的，并且其内容可以就地修改。列表是可变的（mutable）。你可以使用方括号 `[]` 或使用 `list` 类型函数来定义它们：
    
    
    In [42]: a_list = [2, 3, 7, None]
    
    In [43]: tup = ("foo", "bar", "baz")
    
    In [44]: b_list = list(tup)
    
    In [45]: b_list
    Out[45]: ['foo', 'bar', 'baz']
    
    In [46]: b_list[1] = "peekaboo"
    
    In [47]: b_list
    Out[47]: ['foo', 'peekaboo', 'baz']__

列表和元组在语义上相似（尽管元组无法修改），并且可以在许多函数中互换使用。

`list` 内置函数在数据处理中经常用于实现迭代器或生成器表达式：
    
    
    In [48]: gen = range(10)
    
    In [49]: gen
    Out[49]: range(0, 10)
    
    In [50]: list(gen)
    Out[50]: [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]__

#### 添加和删除元素（Adding and removing elements）

可以使用 `append` 方法将元素追加到列表的末尾：
    
    
    In [51]: b_list.append("dwarf")
    
    In [52]: b_list
    Out[52]: ['foo', 'peekaboo', 'baz', 'dwarf']__

使用 `insert` 可以在列表中的特定位置插入元素：
    
    
    In [53]: b_list.insert(1, "red")
    
    In [54]: b_list
    Out[54]: ['foo', 'red', 'peekaboo', 'baz', 'dwarf']__

插入索引必须介于 0 和列表长度之间（包括 0 和列表长度）。

__

警告（Warning）

与 `append` 相比，`insert` 的计算成本较高，因为必须内部移动后续元素的引用以便为新元素腾出空间。如果你需要在序列的开头和结尾都插入元素，你可能希望探索 `collections.deque`，这是一个双端队列，为此目的进行了优化，并可在 Python 标准库中找到。

`insert` 的逆操作是 `pop`，它删除并返回特定索引处的元素：
    
    
    In [55]: b_list.pop(2)
    Out[55]: 'peekaboo'
    
    In [56]: b_list
    Out[56]: ['foo', 'red', 'baz', 'dwarf']__

可以使用 `remove` 按值删除元素，它会定位第一个此类值并将其从列表中删除：
    
    
    In [57]: b_list.append("foo")
    
    In [58]: b_list
    Out[58]: ['foo', 'red', 'baz', 'dwarf', 'foo']
    
    In [59]: b_list.remove("foo")
    
    In [60]: b_list
    Out[60]: ['red', 'baz', 'dwarf', 'foo']__

如果性能不是问题，通过使用 `append` 和 `remove`，你可以将 Python 列表用作类似集合的数据结构（尽管 Python 有实际的集合对象，稍后讨论）。

使用 `in` 关键字检查列表是否包含某个值：
    
    
    In [61]: "dwarf" in b_list
    Out[61]: True __

关键字 `not` 可用于否定 `in`：
    
    
    In [62]: "dwarf" not in b_list
    Out[62]: False __

检查列表是否包含某个值比使用字典和集合（稍后介绍）要慢得多，因为 Python 会对列表的值进行线性扫描，而它可以（基于哈希表）在恒定时间内检查其他值。

#### 连接和组合列表（Concatenating and combining lists）

与元组类似，使用 `+` 将两个列表相加会将它们连接起来：
    
    
    In [63]: [4, None, "foo"] + [7, 8, (2, 3)]
    Out[63]: [4, None, 'foo', 7, 8, (2, 3)]__

如果你已经定义了一个列表，可以使用 `extend` 方法向其追加多个元素：
    
    
    In [64]: x = [4, None, "foo"]
    
    In [65]: x.extend([7, 8, (2, 3)])
    
    In [66]: x
    Out[66]: [4, None, 'foo', 7, 8, (2, 3)]__

请注意，通过加法进行列表连接是一项相对昂贵的操作，因为必须创建一个新列表并复制对象。使用 `extend` 将元素追加到现有列表通常更可取，尤其是在构建大型列表时。因此：
    
    
    everything = []
    for chunk in list_of_lists:
        everything.extend(chunk)__

比连接替代方案更快：
    
    
    everything = []
    for chunk in list_of_lists:
        everything = everything + chunk __

#### 排序（Sorting）

你可以通过调用列表的 `sort` 函数就地排序（无需创建新对象）：
    
    
    In [67]: a = [7, 2, 5, 1, 3]
    
    In [68]: a.sort()
    
    In [69]: a
    Out[69]: [1, 2, 3, 5, 7]__

`sort` 有一些选项，偶尔会派上用场。一个是能够传递一个次要的 _排序键_（sort key）——即一个产生用于排序对象的值。例如，我们可以按字符串的长度对字符串集合进行排序：
    
    
    In [70]: b = ["saw", "small", "He", "foxes", "six"]
    
    In [71]: b.sort(key=len)
    
    In [72]: b
    Out[72]: ['He', 'saw', 'six', 'small', 'foxes']__

很快，我们将看到 `sorted` 函数，它可以生成一个通用序列的排序副本。

#### 切片（Slicing）

你可以使用切片符号选择大多数序列类型的部分，其基本形式由传递给索引运算符 `[]` 的 `start:stop` 组成：
    
    
    In [73]: seq = [7, 2, 3, 7, 5, 6, 0, 1]
    
    In [74]: seq[1:5]
    Out[74]: [2, 3, 7, 5]__

切片也可以用序列赋值：
    
    
    In [75]: seq[3:5] = [6, 3]
    
    In [76]: seq
    Out[76]: [7, 2, 3, 6, 3, 6, 0, 1]__

虽然包含 `start` 索引处的元素，但 _不包含_ `stop` 索引处的元素，因此结果中的元素数量为 `stop - start`。

可以省略 `start` 或 `stop`，在这种情况下，它们分别默认为序列的开头和结尾：
    
    
    In [77]: seq[:5]
    Out[77]: [7, 2, 3, 6, 3]
    
    In [78]: seq[3:]
    Out[78]: [6, 3, 6, 0, 1]__

负索引相对于末尾对序列进行切片：
    
    
    In [79]: seq[-4:]
    Out[79]: [3, 6, 0, 1]
    
    In [80]: seq[-6:-2]
    Out[80]: [3, 6, 3, 6]__

切片语义需要一些时间来适应，尤其是如果你来自 R 或 MATLAB。有关使用正整数和负整数进行切片的说明，请参见图 3.1。在图中，索引显示在“bin edges”处，以帮助显示使用正整数或负整数的切片选择开始和停止的位置。

![](images/pda3_0301.png)

图 3.1: Python 切片约定说明

也可以在第二个冒号后使用 `step`，例如，每隔一个元素取一个：
    
    
    In [81]: seq[::2]
    Out[81]: [7, 3, 3, 0]__

一个聪明的用法是传递 `-1`，这具有反转列表或元组的有用效果：
    
    
    In [82]: seq[::-1]
    Out[82]: [1, 0, 6, 3, 6, 3, 2, 7]__

### 字典（Dictionary）

字典或 `dict` 可能是最重要的 Python 内置数据结构。在其他编程语言中，字典有时称为 _哈希映射_（hash maps）或 _关联数组_（associative arrays）。字典存储 _键值对_（key-value）的集合，其中 _键_（key）和 _值_（value）是 Python 对象。每个键与一个值关联，以便可以方便地检索、插入、修改或删除给定特定键的值。创建字典的一种方法是使用花括号 `{}` 和冒号分隔键和值：
    
    
    In [83]: empty_dict = {}
    
    In [84]: d1 = {"a": "some value", "b": [1, 2, 3, 4]}
    
    In [85]: d1
    Out[85]: {'a': 'some value', 'b': [1, 2, 3, 4]}__

你可以使用与访问列表或元组元素相同的语法来访问、插入或设置元素：
    
    
    In [86]: d1[7] = "an integer"
    
    In [87]: d1
    Out[87]: {'a': 'some value', 'b': [1, 2, 3, 4], 7: 'an integer'}
    
    In [88]: d1["b"]
    Out[88]: [1, 2, 3, 4]__

你可以使用与检查列表或元组是否包含值相同的语法来检查字典是否包含键：
    
    
    In [89]: "b" in d1
    Out[89]: True __

你可以使用 `del` 关键字或 `pop` 方法（同时返回值并删除键）删除值：
    
    
    In [90]: d1[5] = "some value"
    
    In [91]: d1
    Out[91]: 
    {'a': 'some value',
     'b': [1, 2, 3, 4],
     7: 'an integer',
     5: 'some value'}
    
    In [92]: d1["dummy"] = "another value"
    
    In [93]: d1
    Out[93]: 
    {'a': 'some value',
     'b': [1, 2, 3, 4],
     7: 'an integer',
     5: 'some value',
     'dummy': 'another value'}
    
    In [94]: del d1[5]
    
    In [95]: d1
    Out[95]: 
    {'a': 'some value',
     'b': [1, 2, 3, 4],
     7: 'an integer',
     'dummy': 'another value'}
    
    In [96]: ret = d1.pop("dummy")
    
    In [97]: ret
    Out[97]: 'another value'
    
    In [98]: d1
    Out[98]: {'a': 'some value', 'b': [1, 2, 3, 4], 7: 'an integer'}__

`keys` 和 `values` 方法分别提供字典键和值的迭代器。键的顺序取决于它们的插入顺序，这些函数以相同的相应顺序输出键和值：
    
    
    In [99]: list(d1.keys())
    Out[99]: ['a', 'b', 7]
    
    In [100]: list(d1.values())
    Out[100]: ['some value', [1, 2, 3, 4], 'an integer']__

如果需要同时迭代键和值，可以使用 `items` 方法以 2 元组的形式迭代键和值：
    
    
    In [101]: list(d1.items())
    Out[101]: [('a', 'some value'), ('b', [1, 2, 3, 4]), (7, 'an integer')]__

你可以使用 `update` 方法将一个字典合并到另一个字典中：
    
    
    In [102]: d1.update({"b": "foo", "c": 12})
    
    In [103]: d1
    Out[103]: {'a': 'some value', 'b': 'foo', 7: 'an integer', 'c': 12}__

`update` 方法会就地更改字典，因此传递给 `update` 的数据中任何现有键的旧值将被丢弃。

#### 从序列创建字典（Creating dictionaries from sequences）

偶尔最终得到两个序列，并希望将它们按元素配对成字典，这很常见。作为初步尝试，你可能会编写如下代码：
    
    
    mapping = {}
    for key, value in zip(key_list, value_list):
        mapping[key] = value __

由于字典本质上是一个 2 元组的集合，`dict` 函数接受一个 2 元组列表：
    
    
    In [104]: tuples = zip(range(5), reversed(range(5)))
    
    In [105]: tuples
    Out[105]: <zip at 0x17d604d00>
    
    In [106]: mapping = dict(tuples)
    
    In [107]: mapping
    Out[107]: {0: 4, 1: 3, 2: 2, 3: 1, 4: 0}__

稍后我们将讨论 _字典推导式_（dictionary comprehensions），这是另一种构造字典的方法。

#### 默认值（Default values）

通常会有这样的逻辑：
    
    
    if key in some_dict:
        value = some_dict[key]
    else:
        value = default_value __

因此，字典方法 `get` 和 `pop` 可以接受一个要返回的默认值，因此上面的 `if-else` 块可以简单地写成：
    
    
    value = some_dict.get(key, default_value)__

默认情况下，如果键不存在，`get` 将返回 `None`，而 `pop` 将引发异常。对于 _设置_ 值，字典中的值可能是另一种集合，例如列表。例如，你可以想象按单词的首字母将单词列表分类为列表的字典：
    
    
    In [108]: words = ["apple", "bat", "bar", "atom", "book"]
    
    In [109]: by_letter = {}
    
    In [110]: for word in words:
       .....:     letter = word[0]
       .....:     if letter not in by_letter:
       .....:         by_letter[letter] = [word]
       .....:     else:
       .....:         by_letter[letter].append(word)
       .....:
    
    In [111]: by_letter
    Out[111]: {'a': ['apple', 'atom'], 'b': ['bat', 'bar', 'book']}__

`setdefault` 字典方法可用于简化此工作流程。前面的 `for` 循环可以重写为：
    
    
    In [112]: by_letter = {}
    
    In [113]: for word in words:
       .....:     letter = word[0]
       .....:     by_letter.setdefault(letter, []).append(word)
       .....:
    
    In [114]: by_letter
    Out[114]: {'a': ['apple', 'atom'], 'b': ['bat', 'bar', 'book']}__

内置的 `collections` 模块有一个有用的类 `defaultdict`，这使得这更加容易。要创建一个，你传递一个类型或函数来为字典中的每个槽生成默认值：
    
    
    In [115]: from collections import defaultdict
    
    In [116]: by_letter = defaultdict(list)
    
    In [117]: for word in words:
       .....:     by_letter[word[0]].append(word)__

#### 有效的字典键类型（Valid dictionary key types）

虽然字典的值可以是任何 Python 对象，但键通常必须是不可变对象，如标量类型（int、float、string）或元组（元组中的所有对象也必须不可变）。这里的术语是 _可哈希性_（hashability）。你可以使用 `hash` 函数检查对象是否可哈希（可用作字典中的键）：
    
    
    In [118]: hash("string")
    Out[118]: 4022908869268713487
    
    In [119]: hash((1, 2, (2, 3)))
    Out[119]: -9209053662355515447
    
    In [120]: hash((1, 2, [2, 3])) # 失败，因为列表是可变的
    ---------------------------------------------------------------------------
    TypeError                                 Traceback (most recent call last)
    <ipython-input-120-473c35a62c0b> in <module>
    ----> 1 hash((1, 2, [2, 3])) # 失败，因为列表是可变的
    TypeError: unhashable type: 'list'__

使用 `hash` 函数时看到的哈希值通常取决于你使用的 Python 版本。

要将列表用作键，一种选择是将其转换为元组，只要其元素也可以被哈希，元组就可以被哈希：
    
    
    In [121]: d = {}
    
    In [122]: d[tuple([1, 2, 3])] = 5
    
    In [123]: d
    Out[123]: {(1, 2, 3): 5}__

### 集合（Set）

_集合_（set）是唯一元素的无序集合。可以通过两种方式创建集合：通过 `set` 函数或通过带有花括号的 _集合字面量_（set literal）：
    
    
    In [124]: set([2, 2, 2, 1, 3, 3])
    Out[124]: {1, 2, 3}
    
    In [125]: {2, 2, 2, 1, 3, 3}
    Out[125]: {1, 2, 3}__

集合支持数学 _集合操作_（set operations），如并集、交集、差集和对称差集。考虑以下两个示例集合：
    
    
    In [126]: a = {1, 2, 3, 4, 5}
    
    In [127]: b = {3, 4, 5, 6, 7, 8}__

这两个集合的并集是出现在任一集合中的不同元素的集合。这可以使用 `union` 方法或 `|` 二元运算符计算：
    
    
    In [128]: a.union(b)
    Out[128]: {1, 2, 3, 4, 5, 6, 7, 8}
    
    In [129]: a | b
    Out[129]: {1, 2, 3, 4, 5, 6, 7, 8}__

交集包含出现在两个集合中的元素。可以使用 `&` 运算符或 `intersection` 方法：
    
    
    In [130]: a.intersection(b)
    Out[130]: {3, 4, 5}
    
    In [131]: a & b
    Out[131]: {3, 4, 5}__

有关常用集合方法的列表，请参见表 3.1。

表 3.1: Python 集合操作 函数 | 替代语法 | 描述  
---|---|---  
`a.add(x)` | 无 | 将元素 `x` 添加到集合 `a`  
`a.clear()` | 无 | 将集合 `a` 重置为空状态，丢弃其所有元素  
`a.remove(x)` | 无 | 从集合 `a` 中移除元素 `x`  
`a.pop()` | 无 | 从集合 `a` 中移除任意元素，如果集合为空则引发 `KeyError`  
`a.union(b)` | `a | b` | `a` 和 `b` 中所有唯一元素  
`a.update(b)` | `a |= b` | 将 `a` 的内容设置为 `a` 和 `b` 中元素的并集  
`a.intersection(b)` | `a & b` | `a` 和 `b` 中 _都_ 存在的元素  
`a.intersection_update(b)` | `a &= b` | 将 `a` 的内容设置为 `a` 和 `b` 中元素的交集  
`a.difference(b)` | `a - b` | `a` 中不在 `b` 中的元素  
`a.difference_update(b)` | `a -= b` | 将 `a` 设置为 `a` 中不在 `b` 中的元素  
`a.symmetric_difference(b)` | `a ^ b` | 在 `a` 或 `b` 中但 _不同时在两者中_ 的所有元素  
`a.symmetric_difference_update(b)` | `a ^= b` | 将 `a` 设置为包含在 `a` 或 `b` 中但 _不同时在两者中_ 的元素  
`a.issubset(b)` | `<=` | 如果 `a` 的所有元素都包含在 `b` 中，则为 `True`  
`a.issuperset(b)` | `>=` | 如果 `b` 的所有元素都包含在 `a` 中，则为 `True`  
`a.isdisjoint(b)` | 无 | 如果 `a` 和 `b` 没有共同元素，则为 `True`  
  
__

注意（Note）

如果你将不是集合的输入传递给 `union` 和 `intersection` 等方法，Python 将在执行操作之前将输入转换为集合。使用二元运算符时，两个对象必须已经是集合。

所有逻辑集合操作都有就地对应项，使你能够用操作的结果替换操作左侧集合的内容。对于非常大的集合，这可能更高效：
    
    
    In [132]: c = a.copy()
    
    In [133]: c |= b
    
    In [134]: c
    Out[134]: {1, 2, 3, 4, 5, 6, 7, 8}
    
    In [135]: d = a.copy()
    
    In [136]: d &= b
    
    In [137]: d
    Out[137]: {3, 4, 5}__

与字典键一样，集合元素通常必须是不可变的，并且必须是 _可哈希的_（hashable）（这意味着对值调用 `hash` 不会引发异常）。为了在集合中存储类似列表的元素（或其他可变序列），你可以将它们转换为元组：
    
    
    In [138]: my_data = [1, 2, 3, 4]
    
    In [139]: my_set = {tuple(my_data)}
    
    In [140]: my_set
    Out[140]: {(1, 2, 3, 4)}__

你还可以检查一个集合是否是另一个集合的子集（被包含在）或超集（包含所有元素）：
    
    
    In [141]: a_set = {1, 2, 3, 4, 5}
    
    In [142]: {1, 2, 3}.issubset(a_set)
    Out[142]: True
    
    In [143]: a_set.issuperset({1, 2, 3})
    Out[143]: True __

集合相等当且仅当它们的内容相等：
    
    
    In [144]: {1, 2, 3} == {3, 2, 1}
    Out[144]: True __

### 内置序列函数（Built-In Sequence Functions）

Python 有一些有用的序列函数，你应该熟悉并在任何机会中使用。

#### enumerate（枚举）

在迭代序列时，通常希望跟踪当前项的索引。一种自己动手的方法可能如下所示：
    
    
    index = 0
    for value in collection:
       # 用 value 做点什么
       index += 1 __

由于这很常见，Python 有一个内置函数 `enumerate`，它返回一个 `(i, value)` 元组的序列：
    
    
    for index, value in enumerate(collection):
       # 用 value 做点什么 __

#### sorted（排序）

`sorted` 函数从任何序列的元素返回一个新的排序列表：
    
    
    In [145]: sorted([7, 1, 2, 6, 0, 3, 2])
    Out[145]: [0, 1, 2, 2, 3, 6, 7]
    
    In [146]: sorted("horse race")
    Out[146]: [' ', 'a', 'c', 'e', 'e', 'h', 'o', 'r', 'r', 's']__

`sorted` 函数接受与列表上的 `sort` 方法相同的参数。

#### zip（拉链）

`zip` 将多个列表、元组或其他序列的元素“配对”起来，创建一个元组列表：
    
    
    In [147]: seq1 = ["foo", "bar", "baz"]
    
    In [148]: seq2 = ["one", "two", "three"]
    
    In [149]: zipped = zip(seq1, seq2)
    
    In [150]: list(zipped)
    Out[150]: [('foo', 'one'), ('bar', 'two'), ('baz', 'three')]__

`zip` 可以接受任意数量的序列，它产生的元素数量由 _最短_ 序列决定：
    
    
    In [151]: seq3 = [False, True]
    
    In [152]: list(zip(seq1, seq2, seq3))
    Out[152]: [('foo', 'one', False), ('bar', 'two', True)]__

`zip` 的一个常见用途是同时迭代多个序列，也可能与 `enumerate` 结合使用：
    
    
    In [153]: for index, (a, b) in enumerate(zip(seq1, seq2)):
       .....:     print(f"{index}: {a}, {b}")
       .....:
    0: foo, one
    1: bar, two
    2: baz, three __

#### reversed（反转）

`reversed` 以相反的顺序迭代序列的元素：
    
    
    In [154]: list(reversed(range(10)))
    Out[154]: [9, 8, 7, 6, 5, 4, 3, 2, 1, 0]__

请记住，`reversed` 是一个生成器（稍后将更详细地讨论），因此它不会具体化反转序列，直到被具体化（例如，使用 `list` 或 `for` 循环）。

### 列表、集合和字典推导式（List, Set, and Dictionary Comprehensions）

_列表推导式_（List comprehensions）是一种方便且广泛使用的 Python 语言特性。它们允许你通过过滤集合的元素，将通过过滤的元素转换为一个简洁的表达式来简洁地形成一个新列表。它们采用基本形式：
    
    
    [expr for value in collection if condition]__

这等效于以下 `for` 循环：
    
    
    result = []
    for value in collection:
        if condition:
            result.append(expr)__

过滤条件可以省略，只留下表达式。例如，给定一个字符串列表，我们可以过滤掉长度小于或等于 `2` 的字符串，并将它们转换为大写，如下所示：
    
    
    In [155]: strings = ["a", "as", "bat", "car", "dove", "python"]
    
    In [156]: [x.upper() for x in strings if len(x) > 2]
    Out[156]: ['BAT', 'CAR', 'DOVE', 'PYTHON']__

集合和字典推导式是自然的扩展，以类似惯用的方式生成集合和字典，而不是列表。

字典推导式如下所示：
    
    
    dict_comp = {key-expr: value-expr for value in collection
                 if condition}__

集合推导式看起来类似于等效的列表推导式，只是用花括号代替了方括号：
    
    
    set_comp = {expr for value in collection if condition}__

与列表推导式一样，集合和字典推导式主要是为了方便，但它们同样可以使代码更易于编写和阅读。考虑之前的字符串列表。假设我们想要一个集合，仅包含集合中字符串的长度；我们可以使用集合推导式轻松计算：
    
    
    In [157]: unique_lengths = {len(x) for x in strings}
    
    In [158]: unique_lengths
    Out[158]: {1, 2, 3, 4, 6}__

我们也可以使用稍后介绍的 `map` 函数更函数式地表达这一点：
    
    
    In [159]: set(map(len, strings))
    Out[159]: {1, 2, 3, 4, 6}__

作为一个简单的字典推导式示例，我们可以为这些字符串在列表中的位置创建一个查找映射：
    
    
    In [160]: loc_mapping = {value: index for index, value in enumerate(strings)}
    
    In [161]: loc_mapping
    Out[161]: {'a': 0, 'as': 1, 'bat': 2, 'car': 3, 'dove': 4, 'python': 5}__

#### 嵌套列表推导式（Nested list comprehensions）

假设我们有一个列表的列表，包含一些英文和西班牙文名称：
    
    
    In [162]: all_data = [["John", "Emily", "Michael", "Mary", "Steven"],
       .....:             ["Maria", "Juan", "Javier", "Natalia", "Pilar"]]__

假设我们想要一个包含所有名称中包含两个或更多 `a` 的单个列表。我们当然可以用一个简单的 `for` 循环来做到这一点：
    
    
    In [163]: names_of_interest = []
    
    In [164]: for names in all_data:
       .....:     enough_as = [name for name in names if name.count("a") >= 2]
       .....:     names_of_interest.extend(enough_as)
       .....:
    
    In [165]: names_of_interest
    Out[165]: ['Maria', 'Natalia']__

你实际上可以将整个操作包装在一个 _嵌套列表推导式_ 中，看起来像：
    
    
    In [166]: result = [name for names in all_data for name in names
       .....:           if name.count("a") >= 2]
    
    In [167]: result
    Out[167]: ['Maria', 'Natalia']__

起初，嵌套列表推导式有点难以理解。列表推导式的 `for` 部分根据嵌套顺序排列，并且任何过滤条件都像以前一样放在最后。这是另一个例子，我们将整数元组列表“展平”为一个简单的整数列表：
    
    
    In [168]: some_tuples = [(1, 2, 3), (4, 5, 6), (7, 8, 9)]
    
    In [169]: flattened = [x for tup in some_tuples for x in tup]
    
    In [170]: flattened
    Out[170]: [1, 2, 3, 4, 5, 6, 7, 8, 9]__

请记住，如果你编写嵌套 `for` 循环而不是列表推导式，`for` 表达式的顺序将是相同的：
    
    
    flattened = []
    
    for tup in some_tuples:
        for x in tup:
            flattened.append(x)__

你可以有任意多级别的嵌套，但如果你有超过两到三个级别的嵌套，你可能应该开始质疑这是否从代码可读性的角度来看是有意义的。区分刚才显示的语法和列表推导式内部的列表推导式非常重要，后者也是完全有效的：
    
    
    In [172]: [[x for x in tup] for tup in some_tuples]
    Out[172]: [[1, 2, 3], [4, 5, 6], [7, 8, 9]]__

这产生了一个列表的列表，而不是所有内部元素的扁平列表。

```markdown
## 3.2 函数

_函数_（Functions）是 Python 中代码组织和重用的主要且最重要的方法。根据经验，如果你预期需要多次重复相同或非常相似的代码，那么可能值得编写一个可重用的函数。函数还可以通过为一组 Python 语句命名来帮助提高代码的可读性。

函数使用 `def` 关键字声明。函数包含一个代码块，可选择使用 `return` 关键字：

```python
In [173]: def my_function(x, y):
   .....:     return x + y
```

当执行到带有 `return` 的行时，`return` 后面的值或表达式将被发送到调用函数的上下文中，例如：

```python
In [174]: my_function(1, 2)
Out[174]: 3

In [175]: result = my_function(1, 2)

In [176]: result
Out[176]: 3
```

拥有多个 `return` 语句没有问题。如果 Python 在未遇到 `return` 语句的情况下到达函数末尾，则会自动返回 `None`。例如：

```python
In [177]: def function_without_return(x):
   .....:     print(x)

In [178]: result = function_without_return("hello!")
hello!

In [179]: print(result)
None
```

每个函数可以有_位置_参数（positional arguments）和_关键字_参数（keyword arguments）。关键字参数最常用于指定默认值或可选参数。这里我们将定义一个带有可选参数 `z` 的函数，其默认值为 `1.5`：

```python
def my_function2(x, y, z=1.5):
    if z > 1:
        return z * (x + y)
    else:
        return z / (x + y)
```

虽然关键字参数是可选的，但在调用函数时必须指定所有位置参数。

你可以选择提供或不提供关键字来向 `z` 参数传递值，但鼓励使用关键字：

```python
In [181]: my_function2(5, 6, z=0.7)
Out[181]: 0.06363636363636363

In [182]: my_function2(3.14, 7, 3.5)
Out[182]: 35.49

In [183]: my_function2(10, 20)
Out[183]: 45.0
```

函数参数的主要限制是关键字参数_必须_跟在位置参数（如果有）之后。你可以按任意顺序指定关键字参数。这使你无需记住函数参数的指定顺序，只需记住它们的名称即可。

### 命名空间、作用域和局部函数

函数可以访问函数内部创建的变量，也可以访问函数外部更高（甚至_全局_）作用域中的变量。描述 Python 中变量作用域的另一个更具描述性的名称是_命名空间_（namespace）。默认情况下，在函数内赋值的任何变量都会被分配到局部命名空间。局部命名空间在调用函数时创建，并立即由函数的参数填充。函数完成后，局部命名空间会被销毁（有一些例外情况不在本章讨论范围内）。考虑以下函数：

```python
def func():
    a = []
    for i in range(5):
        a.append(i)
```

当调用 `func()` 时，会创建空列表 `a`，追加五个元素，然后在函数退出时销毁 `a`。假设我们如下声明 `a`：

```python
In [184]: a = []

In [185]: def func():
   .....:     for i in range(5):
   .....:         a.append(i)
```

每次调用 `func` 都会修改列表 `a`：

```python
In [186]: func()

In [187]: a
Out[187]: [0, 1, 2, 3, 4]

In [188]: func()

In [189]: a
Out[189]: [0, 1, 2, 3, 4, 0, 1, 2, 3, 4]
```

在函数作用域外赋值变量是可能的，但这些变量必须使用 `global` 或 `nonlocal` 关键字显式声明：

```python
In [190]: a = None

In [191]: def bind_a_variable():
   .....:     global a
   .....:     a = []
   .....: bind_a_variable()
   .....:

In [192]: print(a)
[]
```

`nonlocal` 允许函数修改在非全局的更高作用域中定义的变量。由于它的使用有些深奥（我在本书中从未使用过），我建议你参考 Python 文档以了解更多信息。

__

注意 

我通常不鼓励使用 `global` 关键字。通常，全局变量用于在系统中存储某种状态。如果你发现自己大量使用它们，可能表明需要面向对象编程（使用类）。

### 返回多个值

当我从 Java 和 C++ 转向 Python 编程时，我最喜欢的特性之一是能够以简单的语法从函数返回多个值。下面是一个例子：

```python
def f():
    a = 5
    b = 6
    c = 7
    return a, b, c

a, b, c = f()
```

在数据分析和其他科学应用中，你可能会经常这样做。这里发生的情况是，函数实际上只返回_一个_对象，即一个元组，然后该元组被解包到结果变量中。在前面的例子中，我们也可以这样做：

```python
return_value = f()
```

在这种情况下，`return_value` 将是一个包含三个返回变量的三元组。对于像之前那样返回多个值，一个可能吸引人的替代方法是返回一个字典：

```python
def f():
    a = 5
    b = 6
    c = 7
    return {"a" : a, "b" : b, "c" : c}
```

这种替代技术可能有用，具体取决于你想要做什么。

### 函数是对象

由于 Python 函数是对象，因此可以轻松表达许多在其他语言中难以实现的构造。假设我们正在进行一些数据清理，需要对以下字符串列表应用一系列转换：

```python
In [193]: states = ["   Alabama ", "Georgia!", "Georgia", "georgia", "FlOrIda",
   .....:           "south   carolina##", "West virginia?"]
```

任何处理过用户提交的调查数据的人都见过这样混乱的结果。需要做很多事情才能使这个字符串列表统一并准备好进行分析：去除空格、删除标点符号和标准化正确的大小写。一种方法是使用内置的字符串方法以及用于正则表达式的 `re` 标准库模块：

```python
import re

def clean_strings(strings):
    result = []
    for value in strings:
        value = value.strip()
        value = re.sub("[!#?]", "", value)
        value = value.title()
        result.append(value)
    return result
```

结果如下所示：

```python
In [195]: clean_strings(states)
Out[195]: 
['Alabama',
 'Georgia',
 'Georgia',
 'Georgia',
 'Florida',
 'South   Carolina',
 'West Virginia']
```

你可能觉得有用的另一种方法是列出要应用于特定字符串集的操作：

```python
def remove_punctuation(value):
    return re.sub("[!#?]", "", value)

clean_ops = [str.strip, remove_punctuation, str.title]

def clean_strings(strings, ops):
    result = []
    for value in strings:
        for func in ops:
            value = func(value)
        result.append(value)
    return result
```

然后我们得到：

```python
In [197]: clean_strings(states, clean_ops)
Out[197]: 
['Alabama',
 'Georgia',
 'Georgia',
 'Georgia',
 'Florida',
 'South   Carolina',
 'West Virginia']
```

像这样更_函数式_的模式使您能够在高层轻松修改字符串的转换方式。`clean_strings` 函数现在也更可重用和通用。

您可以将函数用作其他函数的参数，例如内置的 `map` 函数，它将函数应用于某种序列：

```python
In [198]: for x in map(remove_punctuation, states):
   .....:     print(x)
   Alabama 
   Georgia
   Georgia
   georgia
   FlOrIda
   south   carolina
   West virginia
```

`map` 可以作为列表推导式的替代方案，但没有任何过滤器。

### 匿名（Lambda）函数

Python 支持所谓的_匿名_或_lambda_函数，这是一种编写由单个语句组成的函数的方法，该语句的结果即为返回值。它们使用 `lambda` 关键字定义，该关键字除了“我们正在声明一个匿名函数”之外没有其他含义：

```python
In [199]: def short_function(x):
   .....:     return x * 2

In [200]: equiv_anon = lambda x: x * 2
```

在本书的其余部分，我通常将它们称为 lambda 函数。它们在数据分析中特别方便，因为正如你将看到的，在很多情况下，数据转换函数会将函数作为参数。与编写完整的函数声明甚至将 lambda 函数分配给局部变量相比，传递 lambda 函数通常输入更少（且更清晰）。考虑这个例子：

```python
In [201]: def apply_to_list(some_list, f):
   .....:     return [f(x) for x in some_list]

In [202]: ints = [4, 0, 1, 5, 6]

In [203]: apply_to_list(ints, lambda x: x * 2)
Out[203]: [8, 0, 2, 10, 12]
```

你也可以写成 `[x * 2 for x in ints]`，但在这里我们能够简洁地将自定义操作符传递给 `apply_to_list` 函数。

再举一个例子，假设你想根据每个字符串中不同字母的数量对字符串集合进行排序：

```python
In [204]: strings = ["foo", "card", "bar", "aaaa", "abab"]
```

这里我们可以将一个 lambda 函数传递给列表的 `sort` 方法：

```python
In [205]: strings.sort(key=lambda x: len(set(x)))

In [206]: strings
Out[206]: ['aaaa', 'foo', 'abab', 'bar', 'card']
```

### 生成器

Python 中的许多对象都支持迭代，例如遍历列表中的对象或文件中的行。这是通过_迭代器协议_（iterator protocol）实现的，这是一种使对象可迭代的通用方法。例如，遍历字典会产生字典键：

```python
In [207]: some_dict = {"a": 1, "b": 2, "c": 3}

In [208]: for key in some_dict:
   .....:     print(key)
a
b
c
```

当你编写 `for key in some_dict` 时，Python 解释器首先尝试从 `some_dict` 创建一个迭代器：

```python
In [209]: dict_iterator = iter(some_dict)

In [210]: dict_iterator
Out[210]: <dict_keyiterator at 0x17d60e020>
```

迭代器是任何在像 `for` 循环这样的上下文中使用时会将对象产生给 Python 解释器的对象。大多数期望列表或类似列表对象的方法也会接受任何可迭代对象。这包括内置方法如 `min`、`max` 和 `sum`，以及类型构造函数如 `list` 和 `tuple`：

```python
In [211]: list(dict_iterator)
Out[211]: ['a', 'b', 'c']
```

_生成器_（Generator）是一种方便的方法，类似于编写普通函数，来构造新的可迭代对象。普通函数执行并一次返回单个结果，而生成器可以通过每次使用生成器时暂停和恢复执行来返回多个值的序列。要创建生成器，请在函数中使用 `yield` 关键字而不是 `return`：

```python
def squares(n=10):
    print(f"Generating squares from 1 to {n ** 2}")
    for i in range(1, n + 1):
        yield i ** 2
```

当你实际调用生成器时，不会立即执行任何代码：

```python
In [213]: gen = squares()

In [214]: gen
Out[214]: <generator object squares at 0x17d5fea40>
```

直到你向生成器请求元素时，它才会开始执行其代码：

```python
In [215]: for x in gen:
   .....:     print(x, end=" ")
Generating squares from 1 to 100
1 4 9 16 25 36 49 64 81 100
```

__

注意 

由于生成器一次产生一个元素，而不是一次生成整个列表，因此它可以帮助你的程序使用更少的内存。

#### 生成器表达式

另一种创建生成器的方法是使用_生成器表达式_（generator expression）。这是列表、字典和集合推导式的生成器类似物。要创建一个生成器表达式，请将原本是列表推导式的内容括在圆括号中而不是方括号中：

```python
In [216]: gen = (x ** 2 for x in range(100))

In [217]: gen
Out[217]: <generator object <genexpr> at 0x17d5feff0>
```

这等效于以下更冗长的生成器：

```python
def _make_gen():
    for x in range(100):
        yield x ** 2
gen = _make_gen()
```

在某些情况下，生成器表达式可以用作函数参数代替列表推导式：

```python
In [218]: sum(x ** 2 for x in range(100))
Out[218]: 328350

In [219]: dict((i, i ** 2) for i in range(5))
Out[219]: {0: 0, 1: 1, 2: 4, 3: 9, 4: 16}
```

根据推导式表达式产生的元素数量，生成器版本有时可能明显更快。

#### itertools 模块

标准库 `itertools` 模块包含许多常见数据算法的生成器集合。例如，`groupby` 接受任何序列和一个函数，根据函数的返回值对序列中的连续元素进行分组。这是一个例子：

```python
In [220]: import itertools

In [221]: def first_letter(x):
   .....:     return x[0]

In [222]: names = ["Alan", "Adam", "Wes", "Will", "Albert", "Steven"]

In [223]: for letter, names in itertools.groupby(names, first_letter):
   .....:     print(letter, list(names)) # names is a generator
A ['Alan', 'Adam']
W ['Wes', 'Will']
A ['Albert']
S ['Steven']
```

表 3.2 列出了一些我经常发现有用的其他 `itertools` 函数。你可能想查看 [Python 官方文档](https://docs.python.org/3/library/itertools.html) 以了解更多关于这个有用的内置实用程序模块的信息。

表 3.2：一些有用的 `itertools` 函数
函数 | 描述  
---|---  
`chain(*iterables)` | 通过将迭代器链接在一起来生成序列。第一个迭代器的元素用完后，返回下一个迭代器的元素，依此类推。  
`combinations(iterable, k)` | 生成迭代器中所有可能的 `k` 元组元素序列，忽略顺序且无放回（另请参阅配套函数 `combinations_with_replacement`）。  
`permutations(iterable, k)` | 生成迭代器中所有可能的 `k` 元组元素序列，尊重顺序。  
`groupby(iterable[, keyfunc])` | 为每个唯一键生成 `(key, sub-iterator)`。  
`product(*iterables, repeat=1)` | 生成输入可迭代对象的笛卡尔积作为元组，类似于嵌套的 `for` 循环。  

### 错误和异常处理

优雅地处理 Python 错误或_异常_（exceptions）是构建健壮程序的重要组成部分。在数据分析应用程序中，许多函数仅适用于特定类型的输入。例如，Python 的 `float` 函数能够将字符串转换为浮点数，但在输入不当时会失败并出现 `ValueError`：

```python
In [224]: float("1.2345")
Out[224]: 1.2345

In [225]: float("something")
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-225-5ccfe07933f4> in <module>
----> 1 float("something")
ValueError: could not convert string to float: 'something'
```

假设我们想要一个能够优雅失败的 `float` 版本，返回输入参数。我们可以通过编写一个函数来实现，该函数将调用 `float` 的代码封装在 `try`/`except` 块中（在 IPython 中执行此代码）：

```python
def attempt_float(x):
    try:
        return float(x)
    except:
        return x
```

仅当 `float(x)` 引发异常时，才会执行 `except` 块中的代码：

```python
In [227]: attempt_float("1.2345")
Out[227]: 1.2345

In [228]: attempt_float("something")
Out[228]: 'something'
```

你可能会注意到 `float` 除了 `ValueError` 之外还会引发其他异常：

```python
In [229]: float((1, 2))
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-229-82f777b0e564> in <module>
----> 1 float((1, 2))
TypeError: float() argument must be a string or a real number, not 'tuple'
```

你可能只想抑制 `ValueError`，因为 `TypeError`（输入不是字符串或数值）可能表示程序中存在合法的错误。为此，请在 `except` 后写上异常类型：

```python
def attempt_float(x):
    try:
        return float(x)
    except ValueError:
        return x
```

然后我们得到：

```python
In [231]: attempt_float((1, 2))
---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-231-8b0026e9e6b7> in <module>
----> 1 attempt_float((1, 2))
<ipython-input-230-6209ddecd2b5> in attempt_float(x)
      1 def attempt_float(x):
      2     try:
----> 3         return float(x)
      4     except ValueError:
      5         return x
TypeError: float() argument must be a string or a real number, not 'tuple'
```

你可以通过编写异常类型的元组来捕获多种异常类型（需要括号）：

```python
def attempt_float(x):
    try:
        return float(x)
    except (TypeError, ValueError):
        return x
```

在某些情况下，你可能不想抑制异常，但希望无论 `try` 块中的代码是否成功都执行某些代码。为此，请使用 `finally`：

```python
f = open(path, mode="w")

try:
    write_to_file(f)
finally:
    f.close()
```

在这里，文件对象 `f` 将_始终_被关闭。类似地，你可以使用 `else` 来执行仅在 `try:` 块成功时执行的代码：

```python
f = open(path, mode="w")

try:
    write_to_file(f)
except:
    print("Failed")
else:
    print("Succeeded")
finally:
    f.close()
```

#### IPython 中的异常

如果在 `%run`-ning 脚本或执行任何语句时引发异常，IPython 默认会打印完整的调用堆栈跟踪（traceback），并在堆栈中每个点周围提供几行上下文：

```python
In [10]: %run examples/ipython_bug.py
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
```

拥有额外的上下文本身相对于标准 Python 解释器（不提供任何额外上下文）是一个巨大的优势。你可以使用 `%xmode` 魔术命令控制显示的上下文量，从 `Plain`（与标准 Python 解释器相同）到 `Verbose`（内联函数参数值等）。正如你将在 [附录 B：关于 IPython 系统的更多信息](https://wesmckinney.com/book/ipython) 中看到的，你可以在错误发生后_进入堆栈_（使用 `%debug` 或 `%pdb` 魔术）进行交互式事后调试。
```

## 3.3 文件与操作系统

本书大部分内容使用如 `pandas.read_csv` 之类的高级工具，将数据文件从磁盘读取到 Python 数据结构中。然而，理解 Python 中文件操作的基础知识非常重要。幸运的是，这相对简单明了，这也是 Python 在文本和文件处理方面如此受欢迎的原因之一。

要打开文件进行读取或写入，可使用内置的 `open` 函数，并指定相对或绝对文件路径以及可选的编码方式：
    
    
    In [233]: path = "examples/segismundo.txt"
    
    In [234]: f = open(path, encoding="utf-8")__

此处传递 `encoding="utf-8"` 是最佳实践，因为读取文件的默认 Unicode 编码因平台而异。

默认情况下，文件以只读模式 `"r"` 打开。随后，我们可以像处理列表一样处理文件对象 `f`，并逐行迭代：
    
    
    for line in f:
        print(line)__

从文件中取出的行保留行尾（EOL）标记，因此常见做法是使用以下代码获取不含 EOL 的文件行列表：
    
    
    In [235]: lines = [x.rstrip() for x in open(path, encoding="utf-8")]
    
    In [236]: lines
    Out[236]: 
    ['Sueña el rico en su riqueza,',
     'que más cuidados le ofrece;',
     '',
     'sueña el pobre que padece',
     'su miseria y su pobreza;',
     '',
     'sueña el que a medrar empieza,',
     'sueña el que afana y pretende,',
     'sueña el que agravia y ofende,',
     '',
     'y en el mundo, en conclusión,',
     'todos sueñan lo que son,',
     'aunque ninguno lo entiende.',
     '']__

使用 `open` 创建文件对象时，建议在完成后关闭文件。关闭文件会将其资源释放回操作系统：
    
    
    In [237]: f.close()__

一种简化清理打开文件操作的方法是使用 `with` 语句：
    
    
    In [238]: with open(path, encoding="utf-8") as f:
       .....:     lines = [x.rstrip() for x in f]__

这将在退出 `with` 代码块时自动关闭文件 `f`。未能确保文件关闭在许多小型程序或脚本中不会导致问题，但在需要处理大量文件的程序中可能会引发问题。

如果我们输入 `f = open(path, "w")`，将会在 *examples/segismundo.txt* 处创建一个*新文件*（注意！），覆盖该位置的所有文件。还有一种 `"x"` 文件模式，它会创建一个可写文件，但如果文件路径已存在则会失败。所有有效的文件读写模式见表 3.3。

表 3.3：Python 文件模式 模式 | 描述  
---|---  
`r` | 只读模式  
`w` | 只写模式；创建新文件（清除同名文件的任何数据）  
`x` | 只写模式；创建新文件，但如果文件路径已存在则失败  
`a` | 追加到现有文件（如果文件不存在则创建）  
`r+` | 读写模式  
`b` | 添加到模式中以处理二进制文件（例如 `"rb"` 或 `"wb"`）  
`t` | 文本模式（自动将字节解码为 Unicode）；如果未指定，此为默认模式  
  
对于可读文件，一些最常用的方法是 `read`、`seek` 和 `tell`。`read` 从文件中返回一定数量的字符。“字符”的构成由文件编码决定，或者如果文件以二进制模式打开，则仅为原始字节：
    
    
    In [239]: f1 = open(path)
    
    In [240]: f1.read(10)
    Out[240]: 'Sueña el r'
    
    In [241]: f2 = open(path, mode="rb")  # 二进制模式
    
    In [242]: f2.read(10)
    Out[242]: b'Sue\xc3\xb1a el '__

`read` 方法根据读取的字节数推进文件对象的位置。`tell` 提供当前位置：
    
    
    In [243]: f1.tell()
    Out[243]: 11
    
    In [244]: f2.tell()
    Out[244]: 10 __

尽管我们从以文本模式打开的文件 `f1` 中读取了 10 个字符，但位置是 11，因为使用默认编码解码 10 个字符需要这么多字节。可以在 `sys` 模块中检查默认编码：
    
    
    In [245]: import sys
    
    In [246]: sys.getdefaultencoding()
    Out[246]: 'utf-8'__

为了在不同平台上获得一致的行为，最好在打开文件时传递编码（例如广泛使用的 `encoding="utf-8"`）。

`seek` 将文件位置更改为文件中指定的字节：
    
    
    In [247]: f1.seek(3)
    Out[247]: 3
    
    In [248]: f1.read(1)
    Out[248]: 'ñ'
    
    In [249]: f1.tell()
    Out[249]: 5 __

最后，记得关闭文件：
    
    
    In [250]: f1.close()
    
    In [251]: f2.close()__

要将文本写入文件，可以使用文件的 `write` 或 `writelines` 方法。例如，我们可以创建一个不含空行的 *examples/segismundo.txt* 版本：
    
    
    In [252]: path
    Out[252]: 'examples/segismundo.txt'
    
    In [253]: with open("tmp.txt", mode="w") as handle:
       .....:     handle.writelines(x for x in open(path) if len(x) > 1)
    
    In [254]: with open("tmp.txt") as f:
       .....:     lines = f.readlines()
    
    In [255]: lines
    Out[255]: 
    ['Sueña el rico en su riqueza,\n',
     'que más cuidados le ofrece;\n',
     'sueña el pobre que padece\n',
     'su miseria y su pobreza;\n',
     'sueña el que a medrar empieza,\n',
     'sueña el que afana y pretende,\n',
     'sueña el que agravia y ofende,\n',
     'y en el mundo, en conclusión,\n',
     'todos sueñan lo que son,\n',
     'aunque ninguno lo entiende.\n']__

许多最常用的文件方法见表 3.4。

表 3.4：重要的 Python 文件方法或属性 方法/属性 | 描述  
---|---  
`read([size])` | 根据文件模式从文件中返回字节或字符串数据，可选参数 `size` 指示要读取的字节数或字符串字符数  
`readable()` | 如果文件支持 `read` 操作，则返回 `True`  
`readlines([size])` | 返回文件中的行列表，可选参数 `size`  
`write(string)` | 将传递的字符串写入文件  
`writable()` | 如果文件支持 `write` 操作，则返回 `True`  
`writelines(strings)` | 将传递的字符串序列写入文件  
`close()` | 关闭文件对象  
`flush()` | 将内部 I/O 缓冲区刷新到磁盘  
`seek(pos)` | 移动到指定的文件位置（整数）  
`seekable()` | 如果文件对象支持寻址（从而支持随机访问），则返回 `True`（某些类文件对象不支持）  
`tell()` | 以整数形式返回当前文件位置  
`closed` | 如果文件已关闭，则为 `True`  
`encoding` | 用于将文件中的字节解释为 Unicode 的编码（通常为 UTF-8）  
  
### 文件的字节与 Unicode

Python 文件的默认行为是*文本模式*，这意味着您打算使用 Python 字符串（即 Unicode）。这与*二进制模式*形成对比，您可以通过在文件模式后附加 `b` 来获得二进制模式。回顾上一节的文件（包含采用 UTF-8 编码的非 ASCII 字符），我们有：
    
    
    In [258]: with open(path) as f:
       .....:     chars = f.read(10)
    
    In [259]: chars
    Out[259]: 'Sueña el r'
    
    In [260]: len(chars)
    Out[260]: 10 __

UTF-8 是一种可变长度的 Unicode 编码，因此当我从文件中请求一定数量的字符时，Python 会从文件中读取足够的字节（可能少至 10 字节或多达 40 字节）来解码这些字符。如果我改为以 `"rb"` 模式打开文件，`read` 会请求确切的字节数：
    
    
    In [261]: with open(path, mode="rb") as f:
       .....:     data = f.read(10)
    
    In [262]: data
    Out[262]: b'Sue\xc3\xb1a el '__

根据文本编码，您可能能够自行将字节解码为 `str` 对象，但前提是每个编码的 Unicode 字符都是完整的：
    
    
    In [263]: data.decode("utf-8")
    Out[263]: 'Sueña el '
    
    In [264]: data[:4].decode("utf-8")
    ---------------------------------------------------------------------------
    UnicodeDecodeError                        Traceback (most recent call last)
    <ipython-input-264-846a5c2fed34> in <module>
    ----> 1 data[:4].decode("utf-8")
    UnicodeDecodeError: 'utf-8' codec can't decode byte 0xc3 in position 3: unexpecte
    d end of data __

文本模式结合 `open` 的 `encoding` 选项，提供了一种便捷的方法来在不同 Unicode 编码之间进行转换：
    
    
    In [265]: sink_path = "sink.txt"
    
    In [266]: with open(path) as source:
       .....:     with open(sink_path, "x", encoding="iso-8859-1") as sink:
       .....:         sink.write(source.read())
    
    In [267]: with open(sink_path, encoding="iso-8859-1") as f:
       .....:     print(f.read(10))
    Sueña el r __

在以非二进制模式打开文件时使用 `seek` 要小心。如果文件位置落在定义 Unicode 字符的字节中间，则后续读取将导致错误：
    
    
    In [269]: f = open(path, encoding='utf-8')
    
    In [270]: f.read(5)
    Out[270]: 'Sueña'
    
    In [271]: f.seek(4)
    Out[271]: 4
    
    In [272]: f.read(1)
    ---------------------------------------------------------------------------
    UnicodeDecodeError                        Traceback (most recent call last)
    <ipython-input-272-5a354f952aa4> in <module>
    ----> 1 f.read(1)
    ~/miniforge-x86/envs/book-env/lib/python3.10/codecs.py in decode(self, input, fin
    al)
        320         # decode input (taking the buffer into account)
        321         data = self.buffer + input
    --> 322         (result, consumed) = self._buffer_decode(data, self.errors, final
    )
        323         # keep undecoded input until the next call
        324         self.buffer = data[consumed:]
    UnicodeDecodeError: 'utf-8' codec can't decode byte 0xb1 in position 0: invalid s
    tart byte
    
    In [273]: f.close()__

如果您经常处理非 ASCII 文本数据的数据分析，掌握 Python 的 Unicode 功能将非常有价值。更多信息请参阅 [Python 在线文档](https://docs.python.org)。

## 3.4 结论

既然您已经掌握了 Python 环境和语言的一些基础知识，现在是时候继续学习 NumPy 和 Python 中面向数组的计算（array-oriented computing）了。