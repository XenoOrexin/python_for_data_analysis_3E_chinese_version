# 第六章 数据加载、存储与文件格式

__

这本《利用 Python 进行数据分析 第三版》（_Python for Data Analysis 3rd Edition_）的开放获取网络版本现已作为[印刷版和数字版](https://amzn.to/3DyLaJc)的配套资源提供。如果您发现任何错误，[请在此处报告](https://oreilly.com/catalog/0636920519829/errata)。请注意，此网站由 Quarto 生成，某些方面会与 O’Reilly 的印刷版和电子书版本的格式有所不同。

如果您觉得此书的在线版有用，请考虑[订购纸质版](https://amzn.to/3DyLaJc)或[无 DRM 限制的电子书](https://www.ebooks.com/en-us/book/210644288/python-for-data-analysis/wes-mckinney/?affId=WES398681F)以支持作者。本网站内容不可复制或转载。代码示例遵循 MIT 许可证，可在 GitHub 或 Gitee 上获取。

读取数据并使其可访问（通常称为 _数据加载，data loading_）是使用本书中大多数工具的必要第一步。术语 _解析（parsing）_ 有时也用于描述加载文本数据并将其解释为表格和不同的数据类型。我将重点介绍使用 pandas 进行数据输入和输出，尽管其他库中有许多工具可以帮助读写各种格式的数据。

输入和输出通常分为几大类：读取文本文件和其他更高效的磁盘格式、从数据库加载数据，以及与网络源（如 Web API）交互。

```markdown
## 6.1 文本格式数据的读取与写入

pandas 提供了一系列函数用于将表格数据读取为 DataFrame 对象。表 6.1 总结了其中一些函数；`pandas.read_csv` 是本书中最常用的函数之一。我们将在后续的“二进制数据格式”章节中介绍二进制数据格式。

表 6.1: pandas 中的文本和二进制数据加载函数 函数 | 描述  
---|---  
`read_csv` | 从文件、URL 或文件类对象加载分隔数据；默认使用逗号作为分隔符  
`read_fwf` | 读取固定宽度列格式的数据（即无分隔符）  
`read_clipboard` | `read_csv` 的变体，从剪贴板读取数据；适用于从网页转换表格  
`read_excel` | 从 Excel XLS 或 XLSX 文件读取表格数据  
`read_hdf` | 读取由 pandas 写入的 HDF5 文件  
`read_html` | 读取给定 HTML 文档中的所有表格  
`read_json` | 从 JSON（JavaScript 对象表示法）字符串表示、文件、URL 或文件类对象读取数据  
`read_feather` | 读取 Feather 二进制文件格式  
`read_orc` | 读取 Apache ORC 二进制文件格式  
`read_parquet` | 读取 Apache Parquet 二进制文件格式  
`read_pickle` | 读取使用 Python pickle 格式存储的 pandas 对象  
`read_sas` | 读取以 SAS 系统自定义存储格式存储的 SAS 数据集  
`read_spss` | 读取由 SPSS 创建的数据文件  
`read_sql` | 读取 SQL 查询结果（使用 SQLAlchemy）  
`read_sql_table` | 读取整个 SQL 表（使用 SQLAlchemy）；等效于使用 `read_sql` 查询该表中的所有内容  
`read_stata` | 从 Stata 文件格式读取数据集  
`read_xml` | 从 XML 文件读取数据表  
  
我将概述这些函数的工作原理，它们旨在将文本数据转换为 DataFrame。这些函数的可选参数可分为几类：

索引 (Indexing)
    

可以将一列或多列作为返回的 DataFrame 的索引，以及是否从文件、提供的参数中获取列名，或者根本不获取。

类型推断和数据转换 (Type inference and data conversion)
    

包括用户定义的值转换和自定义缺失值标记列表。

日期和时间解析 (Date and time parsing)
    

包括组合功能，例如将分布在多个列中的日期和时间信息合并到结果中的单个列。

迭代 (Iterating)
    

支持迭代处理超大型文件的分块。

不规整数据问题 (Unclean data issues)
    

包括跳过行或页脚、注释，或其他小问题，如带有逗号千位分隔符的数值数据。

由于现实世界中的数据可能非常混乱，一些数据加载函数（尤其是 `pandas.read_csv`）随着时间的推移积累了大量的可选参数。面对众多不同的参数（`pandas.read_csv` 约有 50 个）感到不知所措是正常的。pandas 在线文档中有许多关于每个参数如何工作的示例，因此，如果你在读取特定文件时遇到困难，可能会找到足够相似的示例来帮助你找到正确的参数。

其中一些函数执行*类型推断*，因为列数据类型不是数据格式的一部分。这意味着你不必指定哪些列是数值型、整数型、布尔型或字符串型。其他数据格式（如 HDF5、ORC 和 Parquet）在格式中嵌入了数据类型信息。

处理日期和其他自定义类型可能需要额外的工作。

让我们从一个小的逗号分隔值（CSV）文本文件开始：
    
    
    In [10]: !cat examples/ex1.csv
    a,b,c,d,message
    1,2,3,4,hello
    5,6,7,8,world
    9,10,11,12,foo __

__

注意 

这里我使用了 Unix 的 `cat` shell 命令将文件的原始内容打印到屏幕上。如果你在 Windows 上，可以在 Windows 终端（或命令行）中使用 `type` 代替 `cat` 来实现相同的效果。

由于这是逗号分隔的，我们可以使用 `pandas.read_csv` 将其读入 DataFrame：
    
    
    In [11]: df = pd.read_csv("examples/ex1.csv")
    
    In [12]: df
    Out[12]: 
       a   b   c   d message
    0  1   2   3   4   hello
    1  5   6   7   8   world
    2  9  10  11  12     foo __

文件并不总是有标题行。考虑这个文件：
    
    
    In [13]: !cat examples/ex2.csv
    1,2,3,4,hello
    5,6,7,8,world
    9,10,11,12,foo __

要读取此文件，你有几个选项。你可以允许 pandas 分配默认列名，也可以自己指定名称：
    
    
    In [14]: pd.read_csv("examples/ex2.csv", header=None)
    Out[14]: 
       0   1   2   3      4
    0  1   2   3   4  hello
    1  5   6   7   8  world
    2  9  10  11  12    foo
    
    In [15]: pd.read_csv("examples/ex2.csv", names=["a", "b", "c", "d", "message"])
    Out[15]: 
       a   b   c   d message
    0  1   2   3   4   hello
    1  5   6   7   8   world
    2  9  10  11  12     foo __

假设你希望 `message` 列作为返回的 DataFrame 的索引。你可以使用 `index_col` 参数指定要使用索引为 4 的列或名为 `"message"` 的列：
    
    
    In [16]: names = ["a", "b", "c", "d", "message"]
    
    In [17]: pd.read_csv("examples/ex2.csv", names=names, index_col="message")
    Out[17]: 
             a   b   c   d
    message               
    hello    1   2   3   4
    world    5   6   7   8
    foo      9  10  11  12 __

如果你想从多个列形成分层索引（在 [第 8.1 章：分层索引](https://wesmckinney.com/book/data-wrangling#pandas_hierarchical) 中讨论），可以传递列号或列名的列表：
    
    
    In [18]: !cat examples/csv_mindex.csv
    key1,key2,value1,value2
    one,a,1,2
    one,b,3,4
    one,c,5,6
    one,d,7,8
    two,a,9,10
    two,b,11,12
    two,c,13,14
    two,d,15,16
    
    In [19]: parsed = pd.read_csv("examples/csv_mindex.csv",
       ....:                      index_col=["key1", "key2"])
    
    In [20]: parsed
    Out[20]: 
               value1  value2
    key1 key2                
    one  a          1       2
         b          3       4
         c          5       6
         d          7       8
    two  a          9      10
         b         11      12
         c         13      14
         d         15      16 __

在某些情况下，表格可能没有固定的分隔符，而是使用空格或其他模式来分隔字段。考虑这样一个文本文件：
    
    
    In [21]: !cat examples/ex3.txt
    A         B         C
    aaa -0.264438 -1.026059 -0.619500
    bbb  0.927272  0.302904 -0.032399
    ccc -0.264273 -0.386314 -0.217601
    ddd -0.871858 -0.348382  1.100491 __

虽然你可以手动进行一些处理，但这里的字段由可变数量的空格分隔。在这种情况下，你可以将正则表达式作为分隔符传递给 `pandas.read_csv`。这可以通过正则表达式 `\s+` 来表示，因此我们有：
    
    
    In [22]: result = pd.read_csv("examples/ex3.txt", sep="\s+")
    
    In [23]: result
    Out[23]: 
                A         B         C
    aaa -0.264438 -1.026059 -0.619500
    bbb  0.927272  0.302904 -0.032399
    ccc -0.264273 -0.386314 -0.217601
    ddd -0.871858 -0.348382  1.100491 __

由于列名比数据行少一个，`pandas.read_csv` 在这种特殊情况下推断第一列应为 DataFrame 的索引。

文件解析函数有许多附加参数，可帮助你处理各种异常文件格式（部分列表见表 6.2）。例如，你可以使用 `skiprows` 跳过文件的第一行、第三行和第四行：
    
    
    In [24]: !cat examples/ex4.csv
    # hey!
    a,b,c,d,message
    # just wanted to make things more difficult for you
    # who reads CSV files with computers, anyway?
    1,2,3,4,hello
    5,6,7,8,world
    9,10,11,12,foo
    
    In [25]: pd.read_csv("examples/ex4.csv", skiprows=[0, 2, 3])
    Out[25]: 
       a   b   c   d message
    0  1   2   3   4   hello
    1  5   6   7   8   world
    2  9  10  11  12     foo __

处理缺失值是文件读取过程中一个重要且常常需要细致处理的部分。缺失数据通常要么不存在（空字符串），要么由某个*标记*（占位符）值表示。默认情况下，pandas 使用一组常见的标记，例如 `NA` 和 `NULL`：
    
    
    In [26]: !cat examples/ex5.csv
    something,a,b,c,d,message
    one,1,2,3,4,NA
    two,5,6,,8,world
    three,9,10,11,12,foo
    In [27]: result = pd.read_csv("examples/ex5.csv")
    
    In [28]: result
    Out[28]: 
      something  a   b     c   d message
    0       one  1   2   3.0   4     NaN
    1       two  5   6   NaN   8   world
    2     three  9  10  11.0  12     foo __

回想一下，pandas 将缺失值输出为 `NaN`，因此我们在 `result` 中有两个空值或缺失值：
    
    
    In [29]: pd.isna(result)
    Out[29]: 
       something      a      b      c      d  message
    0      False  False  False  False  False     True
    1      False  False  False   True  False    False
    2      False  False  False  False  False    False __

`na_values` 选项接受一个字符串序列，用于添加到默认的被识别为缺失值的字符串列表中：
    
    
    In [30]: result = pd.read_csv("examples/ex5.csv", na_values=["NULL"])
    
    In [31]: result
    Out[31]: 
      something  a   b     c   d message
    0       one  1   2   3.0   4     NaN
    1       two  5   6   NaN   8   world
    2     three  9  10  11.0  12     foo __

`pandas.read_csv` 有一个默认的 NA 值表示列表，但这些默认值可以通过 `keep_default_na` 选项禁用：
    
    
    In [32]: result2 = pd.read_csv("examples/ex5.csv", keep_default_na=False)
    
    In [33]: result2
    Out[33]: 
      something  a   b   c   d message
    0       one  1   2   3   4      NA
    1       two  5   6       8   world
    2     three  9  10  11  12     foo
    
    In [34]: result2.isna()
    Out[34]: 
       something      a      b      c      d  message
    0      False  False  False  False  False    False
    1      False  False  False  False  False    False
    2      False  False  False  False  False    False
    
    In [35]: result3 = pd.read_csv("examples/ex5.csv", keep_default_na=False,
       ....:                       na_values=["NA"])
    
    In [36]: result3
    Out[36]: 
      something  a   b   c   d message
    0       one  1   2   3   4     NaN
    1       two  5   6       8   world
    2     three  9  10  11  12     foo
    
    In [37]: result3.isna()
    Out[37]: 
       something      a      b      c      d  message
    0      False  False  False  False  False     True
    1      False  False  False  False  False    False
    2      False  False  False  False  False    False __

可以为每列在字典中指定不同的 NA 标记：
    
    
    In [38]: sentinels = {"message": ["foo", "NA"], "something": ["two"]}
    
    In [39]: pd.read_csv("examples/ex5.csv", na_values=sentinels,
       ....:             keep_default_na=False)
    Out[39]: 
      something  a   b   c   d message
    0       one  1   2   3   4     NaN
    1       NaN  5   6       8   world
    2     three  9  10  11  12     NaN __

表 6.2 列出了 `pandas.read_csv` 中一些常用的选项。

表 6.2: 一些 `pandas.read_csv` 函数参数 参数 | 描述  
---|---  
`path` | 表示文件系统位置、URL 或文件类对象的字符串。  
`sep` 或 `delimiter` | 用于分割每行字段的字符序列或正则表达式。  
`header` | 用作列名的行号；默认为 0（第一行），但如果无标题行，应为 `None`。  
`index_col` | 用作结果中行索引的列号或名称；可以是单个名称/数字或用于分层索引的列表。  
`names` | 结果的列名列表。  
`skiprows` | 要忽略的文件开头行数或要跳过的行号列表（从 0 开始）。  
`na_values` | 要替换为 NA 的值序列。除非传递 `keep_default_na=False`，否则它们会被添加到默认列表中。  
`keep_default_na` | 是否使用默认的 NA 值列表（默认为 `True`）。  
`comment` | 用于从行尾拆分注释的字符。  
`parse_dates` | 尝试将数据解析为 `datetime`；默认为 `False`。如果为 `True`，将尝试解析所有列。否则，可以指定要解析的列号或名称列表。如果列表元素是元组或列表，则将多个列合并并解析为日期（例如，如果日期/时间跨两列）。  
`keep_date_col` | 如果合并列以解析日期，则保留合并的列；默认为 `False`。  
`converters` | 包含列号或名称映射到函数的字典（例如，`{"foo": f}` 会将函数 `f` 应用于 `"foo"` 列中的所有值）。  
`dayfirst` | 在解析可能歧义的日期时，将其视为国际格式（例如，7/6/2012 -> 2012年6月7日）；默认为 `False`。  
`date_parser` | 用于解析日期的函数。  
`nrows` | 从文件开头读取的行数（不计算标题）。  
`iterator` | 返回一个用于逐段读取文件的 `TextFileReader` 对象。此对象也可与 `with` 语句一起使用。  
`chunksize` | 用于迭代的文件块大小。  
`skip_footer` | 忽略文件末尾的行数。  
`verbose` | 打印各种解析信息，例如文件转换每个阶段所花费的时间和内存使用信息。  
`encoding` | 文本编码（例如，`"utf-8"` 表示 UTF-8 编码文本）。如果为 `None`，则默认为 `"utf-8"`。  
`squeeze` | 如果解析的数据仅包含一列，则返回 Series。  
`thousands` | 千位分隔符（例如，`","` 或 `"."`）；默认为 `None`。  
`decimal` | 数字中的小数分隔符（例如，`"."` 或 `","`）；默认为 `"."`。  
`engine` | 要使用的 CSV 解析和转换引擎；可以是 `"c"`、`"python"` 或 `"pyarrow"` 之一。默认是 `"c"`，不过较新的 `"pyarrow"` 引擎可以更快地解析某些文件。`"python"` 引擎较慢，但支持其他引擎不支持的一些功能。  
  
### 分块读取文本文件

在处理非常大的文件或确定正确参数集以正确处理大文件时，你可能希望仅读取文件的一小部分或迭代处理文件的较小块。

在查看大文件之前，我们先将 pandas 的显示设置调整得更紧凑：
    
    
    In [40]: pd.options.display.max_rows = 10 __

现在我们得到：
    
    
    In [41]: result = pd.read_csv("examples/ex6.csv")
    
    In [42]: result
    Out[42]: 
               one       two     three      four key
    0     0.467976 -0.038649 -0.295344 -1.824726   L
    1    -0.358893  1.404453  0.704965 -0.200638   B
    2    -0.501840  0.659254 -0.421691 -0.057688   G
    3     0.204886  1.074134  1.388361 -0.982404   R
    4     0.354628 -0.133116  0.283763 -0.837063   Q
    ...        ...       ...       ...       ...  ..
    9995  2.311896 -0.417070 -1.409599 -0.515821   L
    9996 -0.479893 -0.650419  0.745152 -0.646038   E
    9997  0.523331  0.787112  0.486066  1.093156   K
    9998 -0.362559  0.598894 -1.843201  0.887292   G
    9999 -0.096376 -1.012999 -0.657431 -0.573315   0
    [10000 rows x 5 columns]__

省略号 `...` 表示 DataFrame 中间的行已被省略。

如果你只想读取少量行（避免读取整个文件），可以使用 `nrows` 指定：
    
    
    In [43]: pd.read_csv("examples/ex6.csv", nrows=5)
    Out[43]: 
            one       two     three      four key
    0  0.467976 -0.038649 -0.295344 -1.824726   L
    1 -0.358893  1.404453  0.704965 -0.200638   B
    2 -0.501840  0.659254 -0.421691 -0.057688   G
    3  0.204886  1.074134  1.388361 -0.982404   R
    4  0.354628 -0.133116  0.283763 -0.837063   Q __

要分块读取文件，请将 `chunksize` 指定为行数：
    
    
    In [44]: chunker = pd.read_csv("examples/ex6.csv", chunksize=1000)
    
    In [45]: type(chunker)
    Out[45]: pandas.io.parsers.readers.TextFileReader __

`pandas.read_csv` 返回的 `TextFileReader` 对象允许你根据 `chunksize` 迭代文件的部分内容。例如，我们可以迭代 `ex6.csv`，聚合 `"key"` 列中的值计数，如下所示：
    
    
    chunker = pd.read_csv("examples/ex6.csv", chunksize=1000)
    
    tot = pd.Series([], dtype='int64')
    for piece in chunker:
        tot = tot.add(piece["key"].value_counts(), fill_value=0)
    
    tot = tot.sort_values(ascending=False)__

我们得到：
    
    
    In [47]: tot[:10]
    Out[47]: 
    key
    E    368.0
    X    364.0
    L    346.0
    O    343.0
    Q    340.0
    M    338.0
    J    337.0
    F    335.0
    K    334.0
    H    330.0
    dtype: float64 __

`TextFileReader` 还配备了 `get_chunk` 方法，允许你读取任意大小的块。

### 将数据写入文本格式

数据也可以导出为分隔格式。让我们考虑之前读取的一个 CSV 文件：
    
    
    In [48]: data = pd.read_csv("examples/ex5.csv")
    
    In [49]: data
    Out[49]: 
      something  a   b     c   d message
    0       one  1   2   3.0   4     NaN
    1       two  5   6   NaN   8   world
    2     three  9  10  11.0  12     foo __

使用 DataFrame 的 `to_csv` 方法，我们可以将数据写入逗号分隔文件：
    
    
    In [50]: data.to_csv("examples/out.csv")
    
    In [51]: !cat examples/out.csv
    ,something,a,b,c,d,message
    0,one,1,2,3.0,4,
    1,two,5,6,,8,world
    2,three,9,10,11.0,12,foo __

当然，也可以使用其他分隔符（写入 `sys.stdout` 以便将文本结果打印到控制台而不是文件）：
    
    
    In [52]: import sys
    
    In [53]: data.to_csv(sys.stdout, sep="|")
    |something|a|b|c|d|message
    0|one|1|2|3.0|4|
    1|two|5|6||8|world
    2|three|9|10|11.0|12|foo __

缺失值在输出中显示为空字符串。你可能希望用其他标记值来表示它们：
    
    
    In [54]: data.to_csv(sys.stdout, na_rep="NULL")
    ,something,a,b,c,d,message
    0,one,1,2,3.0,4,NULL
    1,two,5,6,NULL,8,world
    2,three,9,10,11.0,12,foo __

如果没有指定其他选项，则会写入行和列标签。这两者都可以禁用：
    
    
    In [55]: data.to_csv(sys.stdout, index=False, header=False)
    one,1,2,3.0,4,
    two,5,6,,8,world
    three,9,10,11.0,12,foo __

你也可以只写入列的子集，并按照你选择的顺序：
    
    
    In [56]: data.to_csv(sys.stdout, index=False, columns=["a", "b", "c"])
    a,b,c
    1,2,3.0
    5,6,
    9,10,11.0 __

### 处理其他分隔格式

使用 `pandas.read_csv` 之类的函数可以从磁盘加载大多数形式的表格数据。然而，在某些情况下，可能需要进行一些手动处理。收到一个或多个格式错误的行导致 `pandas.read_csv` 出错的情况并不少见。为了说明基本工具，考虑一个小的 CSV 文件：
    
    
    In [57]: !cat examples/ex7.csv
    "a","b","c"
    "1","2","3"
    "1","2","3"__

对于任何具有单字符分隔符的文件，你可以使用 Python 内置的 `csv` 模块。要使用它，将任何打开的文件或文件类对象传递给 `csv.reader`：
    
    
    In [58]: import csv
    
    In [59]: f = open("examples/ex7.csv")
    
    In [60]: reader = csv.reader(f)__

像文件一样遍历阅读器会产生移除了任何引号字符的值列表：
    
    
    In [61]: for line in reader:
       ....:     print(line)
    ['a', 'b', 'c']
    ['1', '2', '3']
    ['1', '2', '3']
    
    In [62]: f.close()__

从那里开始，需要你进行必要的整理以将数据置于所需的形式。让我们一步一步来。首先，将文件读入行列表：
    
    
    In [63]: with open("examples/ex7.csv") as f:
       ....:     lines = list(csv.reader(f))__

然后我们将行拆分为标题行和数据行：
    
    
    In [64]: header, values = lines[0], lines[1:]__

然后我们可以使用字典推导式和表达式 `zip(*values)`（注意，这在大型文件上会使用大量内存）来创建数据列的字典，该表达式将行转置为列：
    
    
    In [65]: data_dict = {h: v for h, v in zip(header, zip(*values))}
    
    In [66]: data_dict
    Out[66]: {'a': ('1', '1'), 'b': ('2', '2'), 'c': ('3', '3')}__

CSV 文件有多种不同的风格。要定义具有不同分隔符、字符串引用约定或行终止符的新格式，我们可以定义一个简单的 `csv.Dialect` 子类：
    
    
    class my_dialect(csv.Dialect):
        lineterminator = "\n"
        delimiter = ";"
        quotechar = '"'
        quoting = csv.QUOTE_MINIMAL __
    
    
    reader = csv.reader(f, dialect=my_dialect)__

我们也可以将单独的 CSV 方言参数作为关键字提供给 `csv.reader`，而无需定义子类：
    
    
    reader = csv.reader(f, delimiter="|")__

可能的选项（`csv.Dialect` 的属性）及其作用可以在表 6.3 中找到。

表 6.3: CSV `dialect` 选项 参数 | 描述  
---|---  
`delimiter` | 用于分隔字段的单字符字符串；默认为 `","`。  
`lineterminator` | 用于写入的行终止符；默认为 `"\r\n"`。读取器会忽略此选项并识别跨平台行终止符。  
`quotechar` | 用于带有特殊字符（如分隔符）的字段的引号字符；默认为 `'"'`。  
`quoting` | 引用约定。选项包括 `csv.QUOTE_ALL`（引用所有字段）、`csv.QUOTE_MINIMAL`（仅引用带有特殊字符如分隔符的字段）、`csv.QUOTE_NONNUMERIC` 和 `csv.QUOTE_NONE`（不引用）。详见 Python 文档。默认为 `QUOTE_MINIMAL`。  
`skipinitialspace` | 忽略每个分隔符后的空格；默认为 `False`。  
`doublequote` | 如何处理字段内的引号字符；如果为 `True`，则将其加倍（详见在线文档以获取完整细节和行为）。  
`escapechar` | 如果 `quoting` 设置为 `csv.QUOTE_NONE`，则用于转义分隔符的字符串；默认禁用。  
  
__

注意 

对于具有更复杂或固定的多字符分隔符的文件，你将无法使用 `csv` 模块。在这些情况下，你必须使用字符串的 `split` 方法或正则表达式方法 `re.split` 进行行拆分和其他清理。幸运的是，如果你传递必要的选项，`pandas.read_csv` 几乎能够完成你需要的所有操作，因此你很少需要手动解析文件。

要手动*写入*分隔文件，你可以使用 `csv.writer`。它接受一个可写的打开文件对象，以及与 `csv.reader` 相同的方言和格式选项：
    
    
    with open("mydata.csv", "w") as f:
        writer = csv.writer(f, dialect=my_dialect)
        writer.writerow(("one", "two", "three"))
        writer.writerow(("1", "2", "3"))
        writer.writerow(("4", "5", "6"))
        writer.writerow(("7", "8", "9"))__

### JSON 数据

JSON（JavaScript 对象表示法的缩写）已成为通过 HTTP 请求在 Web 浏览器和其他应用程序之间发送数据的标准格式之一。它是一种比 CSV 这样的表格文本形式自由得多的数据格式。下面是一个示例：
    
    
    obj = """
    {"name": "Wes",
     "cities_lived": ["Akron", "Nashville", "New York", "San Francisco"],
     "pet": null,
     "siblings": [{"name": "Scott", "age": 34, "hobbies": ["guitars", "soccer"]},
                  {"name": "Katie", "age": 42, "hobbies": ["diving", "art"]}]
    }
    """__

JSON 几乎是有效的 Python 代码，除了它的空值 `null` 和其他一些细微差别（例如不允许在列表末尾出现尾随逗号）。基本类型是对象（字典）、数组（列表）、字符串、数字、布尔值和空值。对象中的所有键必须是字符串。有多个 Python 库用于读写 JSON 数据。我将在这里使用 `json`，因为它内置于 Python 标准库中。要将 JSON 字符串转换为 Python 形式，请使用 `json.loads`：
    
    
    In [68]: import json
    
    In [69]: result = json.loads(obj)
    
    In [70]: result
    Out[70]: 
    {'name': 'Wes',
     'cities_lived': ['Akron', 'Nashville', 'New York', 'San Francisco'],
     'pet': None,
     'siblings': [{'name': 'Scott',
       'age': 34,
       'hobbies': ['guitars', 'soccer']},
      {'name': 'Katie', 'age': 42, 'hobbies': ['diving', 'art']}]}__

另一方面，`json.dumps` 将 Python 对象转换回 JSON：
    
    
    In [71]: asjson = json.dumps(result)
    
    In [72]: asjson
    Out[72]: '{"name": "Wes", "cities_lived": ["Akron", "Nashville", "New York", "San
     Francisco"], "pet": null, "siblings": [{"name": "Scott", "age": 34, "hobbies": [
    "guitars", "soccer"]}, {"name": "Katie", "age": 42, "hobbies": ["diving", "art"]}
    ]}'__

如何将 JSON 对象或对象列表转换为 DataFrame 或其他数据结构进行分析取决于你。方便的是，你可以将字典列表（之前是 JSON 对象）传递给 DataFrame 构造函数并选择数据字段的子集：
    
    
    In [73]: siblings = pd.DataFrame(result["siblings"], columns=["name", "age"])
    
    In [74]: siblings
    Out[74]: 
        name  age
    0  Scott   34
    1  Katie   42 __

`pandas.read_json` 可以自动将特定排列的 JSON 数据集转换为 Series 或 DataFrame。例如：
    
    
    In [75]: !cat examples/example.json
    [{"a": 1, "b": 2, "c": 3},
     {"a": 4, "b": 5, "c": 6},
     {"a": 7, "b": 8, "c": 9}]__

`pandas.read_json` 的默认选项假定 JSON 数组中的每个对象都是表中的一行：
    
    
    In [76]: data = pd.read_json("examples/example.json")
    
    In [77]: data
    Out[77]: 
       a  b  c
    0  1  2  3
    1  4  5  6
    2  7  8  9 __

有关读取和操作 JSON 数据（包括嵌套记录）的扩展示例，请参见第 13 章：数据分析示例中的 USDA 食品数据库示例。

如果你需要将数据从 pandas 导出到 JSON，一种方法是使用 Series 和 DataFrame 上的 `to_json` 方法：
    
    
    In [78]: data.to_json(sys.stdout)
    {"a":{"0":1,"1":4,"2":7},"b":{"0":2,"1":5,"2":8},"c":{"0":3,"1":6,"2":9}}
    In [79]: data.to_json(sys.stdout, orient="records")
    [{"a":1,"b":2,"c":3},{"a":4,"b":5,"c":6},{"a":7,"b":8,"c":9}]__

### XML 和 HTML：网络抓取

Python 有许多用于读写无处不在的 HTML 和 XML 格式数据的库。例如 lxml、Beautiful Soup 和 html5lib。虽然 lxml 通常速度更快，但其他库可以更好地处理格式错误的 HTML 或 XML 文件。

pandas 有一个内置函数 `pandas.read_html`，它使用所有这些库自动将 HTML 文件中的表格解析为 DataFrame 对象。为了展示这是如何工作的，我从美国 FDIC 下载了一个 HTML 文件（用于 pandas 文档），显示银行倒闭情况。1 首先，你必须安装 `read_html` 使用的一些附加库：
    
    
    conda install lxml beautifulsoup4 html5lib

如果你不使用 conda，`pip install lxml` 也应该有效。

`pandas.read_html` 函数有许多选项，但默认情况下它会搜索并尝试解析 `<table>` 标签中包含的所有表格数据。结果是一个 DataFrame 对象列表：
    
    
    In [80]: tables = pd.read_html("examples/fdic_failed_bank_list.html")
    
    In [81]: len(tables)
    Out[81]: 1
    
    In [82]: failures = tables[0]
    
    In [83]: failures.head()
    Out[83]: 
                          Bank Name             City  ST   CERT   
    0                   Allied Bank         Mulberry  AR     91  \
    1  The Woodbury Banking Company         Woodbury  GA  11297   
    2        First CornerStone Bank  King of Prussia  PA  35312   
    3            Trust Company Bank          Memphis  TN   9956   
    4    North Milwaukee State Bank        Milwaukee  WI  20364   
                     Acquiring Institution        Closing Date       Updated Date  
    0                         Today's Bank  September 23, 2016  November 17, 2016  
    1                          United Bank     August 19, 2016  November 17, 2016  
    2  First-Citizens Bank & Trust Company         May 6, 2016  September 6, 2016  
    3           The Bank of Fayette County      April 29, 2016  September 6, 2016  
    4  First-Citizens Bank & Trust Company      March 11, 2016      June 16, 2016  __

由于 `failures` 有许多列，pandas 插入了一个换行符 `\`。

正如你将在后续章节中学到的，从这里我们可以继续进行一些数据清理和分析，例如按年计算银行倒闭数量：
    
    
    In [84]: close_timestamps = pd.to_datetime(failures["Closing Date"])
    
    In [85]: close_timestamps.dt.year.value_counts()
    Out[85]: 
    Closing Date
    2010    157
    2009    140
    2011     92
    2012     51
    2008     25
           ... 
    2004      4
    2001      4
    2007      3
    2003      3
    2000      2
    Name: count, Length: 15, dtype: int64 __

#### 使用 lxml.objectify 解析 XML

XML 是另一种常见的结构化数据格式，支持带有元数据的分层、嵌套数据。你目前正在阅读的这本书实际上是从一系列大型 XML 文档创建的。

之前，我展示了 `pandas.read_html` 函数，它在底层使用 lxml 或 Beautiful Soup 来从 HTML 解析数据。XML 和 HTML 在结构上相似，但 XML 更通用。这里，我将展示一个如何使用 lxml 从更通用的 XML 格式解析数据的示例。

多年来，纽约大都会运输署（MTA）以 XML 格式发布了许多关于其巴士和火车服务的数据系列。这里我们将查看性能数据，这些数据包含在一组 XML 文件中。每项火车或巴士服务都有一个不同的文件（例如 Metro-North Railroad 的 _Performance\_MNR.xml_），其中包含按月数据，是一系列 XML 记录，如下所示：
    
    
    <INDICATOR>
      <INDICATOR_SEQ>373889</INDICATOR_SEQ>
      <PARENT_SEQ></PARENT_SEQ>
      <AGENCY_NAME>Metro-North Railroad</AGENCY_NAME>
      <INDICATOR_NAME>Escalator Availability</INDICATOR_NAME>
      <DESCRIPTION>Percent of the time that escalators are operational
      systemwide. The availability rate is based on physical observations performed
      the morning of regular business days only. This is a new indicator the agency
      began reporting in 2009.</DESCRIPTION>
      <PERIOD_YEAR>2011</PERIOD_YEAR>
      <PERIOD_MONTH>12</PERIOD_MONTH>
      <CATEGORY>Service Indicators</CATEGORY>
      <FREQUENCY>M</FREQUENCY>
      <DESIRED_CHANGE>U</DESIRED_CHANGE>
      <INDICATOR_UNIT>%</INDICATOR_UNIT>
      <DECIMAL_PLACES>1</DECIMAL_PLACES>
      <YTD_TARGET>97.00</YTD_TARGET>
      <YTD_ACTUAL></YTD_ACTUAL>
      <MONTHLY_TARGET>97.00</MONTHLY_TARGET>
      <MONTHLY_ACTUAL></MONTHLY_ACTUAL>
    </INDICATOR>__

使用 `lxml.objectify`，我们解析文件并通过 `getroot` 获取 XML 文件根节点的引用：
    
    
    In [86]: from lxml import objectify
    
    In [87]: path = "datasets/mta_perf/Performance_MNR.xml"
    
    In [88]: with open(path) as f:
       ....:     parsed = objectify.parse(f)
    
    In [89]: root = parsed.getroot()__

`root.INDICATOR` 返回一个生成器，产生每个 `<INDICATOR>` XML 元素。对于每条记录，我们可以通过运行以下代码来填充标签名称（如 `YTD_ACTUAL`）到数据值的字典（排除一些标签）：
    
    
    data = []
    
    skip_fields = ["PARENT_SEQ", "INDICATOR_SEQ",
                   "DESIRED_CHANGE", "DECIMAL_PLACES"]
    
    for elt in root.INDICATOR:
        el_data = {}
        for child in elt.getchildren():
            if child.tag in skip_fields:
                continue
            el_data[child.tag] = child.pyval
        data.append(el_data)__

最后，将这个字典列表转换为 DataFrame：
    
    
    In [91]: perf = pd.DataFrame(data)
    
    In [92]: perf.head()
    Out[92]: 
                AGENCY_NAME                        INDICATOR_NAME   
    0  Metro-North Railroad  On-Time Performance (West of Hudson)  \
    1  Metro-North Railroad  On-Time Performance (West of Hudson)   
    2  Metro-North Railroad  On-Time Performance (West of Hudson)   
    3  Metro-North Railroad  On-Time Performance (West of Hudson)   
    4  Metro-North Railroad  On-Time Performance (West of Hudson)   
                                                                      DESCRIPTION   
    0  Percent of commuter trains that arrive at their destinations within 5 m...  \
    1  Percent of commuter trains that arrive at their destinations within 5 m...   
    2  Percent of commuter trains that arrive at their destinations within 5 m...   
    3  Percent of commuter trains that arrive at their destinations within 5 m...   
    4  Percent of commuter trains that arrive at their destinations within 5 m...   
       PERIOD_YEAR  PERIOD_MONTH            CATEGORY FREQUENCY INDICATOR_UNIT   
    0         2008             1  Service Indicators         M              %  \
    1         2008             2  Service Indicators         M              %   
    2         2008             3  Service Indicators         M              %   
    3         2008             4  Service Indicators         M              %   
    4         2008             5  Service Indicators         M              %   
      YTD_TARGET YTD_ACTUAL MONTHLY_TARGET MONTHLY_ACTUAL  
    0       95.0       96.9           95.0           96.9  
    1       95.0       96.0           95.0           95.0  
    2       95.0       96.3           95.0           96.9  
    3       95.0       96.8           95.0           98.3  
    4       95.0       96.6           95.0           95.8  __

pandas 的 `pandas.read_xml` 函数将此过程变为一行表达式：
    
    
    In [93]: perf2 = pd.read_xml(path)
    
    In [94]: perf2.head()
    Out[94]: 
       INDICATOR_SEQ  PARENT_SEQ           AGENCY_NAME   
    0          28445         NaN  Metro-North Railroad  \
    1          28445         NaN  Metro-North Railroad   
    2          28445         NaN  Metro-North Railroad   
    3          28445         NaN  Metro-North Railroad   
    4          28445         NaN  Metro-North Railroad   
                             INDICATOR_NAME   
    0  On-Time Performance (West of Hudson)  \
    1  On-Time Performance (West of Hudson)   
    2  On-Time Performance (West of Hudson)   
    3  On-Time Performance (West of Hudson)   
    4  On-Time Performance (West of Hudson)   
                                                                      DESCRIPTION   
    0  Percent of commuter trains that arrive at their destinations within 5 m...  \
    1  Percent of commuter trains that arrive at their destinations within 5 m...   
    2  Percent of commuter trains that arrive at their destinations within 5 m...   
    3  Percent of commuter trains that arrive at their destinations within 5 m...   
    4  Percent of commuter trains that arrive at their destinations within 5 m...   
       PERIOD_YEAR  PERIOD_MONTH            CATEGORY FREQUENCY DESIRED_CHANGE   
    0         2008             1  Service Indicators         M              U  \
    1         2008             2  Service Indicators         M              U   
    2         2008             3  Service Indicators         M              U   
    3         2008             4  Service Indicators         M              U   
    4         2008             5  Service Indicators         M              U   
      INDICATOR_UNIT  DECIMAL_PLACES YTD_TARGET YTD_ACTUAL MONTHLY_TARGET   
    0              %               1      95.00      96.90          95.00  \
    1              %               1      95.00      96.00          95.00   
    2              %               1      95.00      96.30          95.00   
    3              %               1      95.00      96.80          95.00   
    4              %               1      95.00      96.60          95.00   
      MONTHLY_ACTUAL  
    0          96.90  
    1          95.00  
    2          96.90  
    3          98.30  
    4          95.80  __

对于更复杂的 XML 文档，请参考 `pandas.read_xml` 的文档字符串，其中描述了如何进行选择和过滤以提取感兴趣的具体表格。
```

## 6.2 二进制数据格式

以二进制格式存储（或称为 _序列化_，serialize）数据的一种简单方法是使用 Python 内置的 `pickle` 模块。pandas 对象都拥有一个 `to_pickle` 方法，可以将数据以 pickle 格式写入磁盘：

```python
In [95]: frame = pd.read_csv("examples/ex1.csv")

In [96]: frame
Out[96]: 
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo

In [97]: frame.to_pickle("examples/frame_pickle")
```

Pickle 文件通常只能在 Python 中被读取。你可以使用内置的 `pickle` 直接读取存储在文件中的任何“pickled”对象，或者更方便地使用 `pandas.read_pickle`：

```python
In [98]: pd.read_pickle("examples/frame_pickle")
Out[98]: 
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo
```

注意
`pickle` 仅推荐作为短期存储格式。问题在于很难保证格式的长期稳定性；今天被 pickle 的对象可能无法在未来的库版本中被反序列化。pandas 已尽力保持向后兼容性，但在未来的某个时间点可能仍需“破坏” pickle 格式。

pandas 内置支持多种其他开源二进制数据格式，例如 HDF5、ORC 和 Apache Parquet。例如，如果你安装了 `pyarrow` 包（`conda install pyarrow`），那么你可以使用 `pandas.read_parquet` 读取 Parquet 文件：

```python
In [100]: fec = pd.read_parquet('datasets/fec/fec.parquet')
```

我将在“使用 HDF5 格式”一节中提供一些 HDF5 示例。我鼓励你探索不同的文件格式，以了解它们的速度以及它们对你的分析工作的适用程度。

### 读取 Microsoft Excel 文件

pandas 还支持使用 `pandas.ExcelFile` 类或 `pandas.read_excel` 函数读取存储在 Excel 2003（及更高版本）文件中的表格数据。在内部，这些工具分别使用附加包 `xlrd` 和 `openpyxl` 来读取旧式的 XLS 和较新的 XLSX 文件。这些包必须使用 pip 或 conda 单独从 pandas 安装：

```bash
conda install openpyxl xlrd
```

要使用 `pandas.ExcelFile`，请通过传递一个 `xls` 或 `xlsx` 文件的路径来创建一个实例：

```python
In [101]: xlsx = pd.ExcelFile("examples/ex1.xlsx")
```

该对象可以显示文件中可用的工作表名称列表：

```python
In [102]: xlsx.sheet_names
Out[102]: ['Sheet1']
```

然后，可以使用 `parse` 将工作表中存储的数据读入 DataFrame：

```python
In [103]: xlsx.parse(sheet_name="Sheet1")
Out[103]: 
   Unnamed: 0  a   b   c   d message
0           0  1   2   3   4   hello
1           1  5   6   7   8   world
2           2  9  10  11  12     foo
```

这个 Excel 表有一个索引列，因此我们可以使用 `index_col` 参数来指明这一点：

```python
In [104]: xlsx.parse(sheet_name="Sheet1", index_col=0)
Out[104]: 
   a   b   c   d message
0  1   2   3   4   hello
1  5   6   7   8   world
2  9  10  11  12     foo
```

如果你正在读取文件中的多个工作表，那么创建 `pandas.ExcelFile` 会更快，但你也可以简单地将文件名传递给 `pandas.read_excel`：

```python
In [105]: frame = pd.read_excel("examples/ex1.xlsx", sheet_name="Sheet1")

In [106]: frame
Out[106]: 
   Unnamed: 0  a   b   c   d message
0           0  1   2   3   4   hello
1           1  5   6   7   8   world
2           2  9  10  11  12     foo
```

要将 pandas 数据写入 Excel 格式，你必须首先创建一个 `ExcelWriter`，然后使用 pandas 对象的 `to_excel` 方法将数据写入其中：

```python
In [107]: writer = pd.ExcelWriter("examples/ex2.xlsx")

In [108]: frame.to_excel(writer, "Sheet1")

In [109]: writer.close()
```

你也可以将文件路径传递给 `to_excel`，从而避免使用 `ExcelWriter`：

```python
In [110]: frame.to_excel("examples/ex2.xlsx")
```

### 使用 HDF5 格式

HDF5 是一种备受推崇的文件格式，旨在存储大量科学数组数据。它作为一个 C 库提供，并且有许多其他语言的接口可用，包括 Java、Julia、MATLAB 和 Python。HDF5 中的“HDF”代表 _分层数据格式_（hierarchical data format）。每个 HDF5 文件可以存储多个数据集和支持的元数据。与更简单的格式相比，HDF5 支持使用多种压缩模式进行即时压缩，从而能够更高效地存储具有重复模式的数据。对于处理无法放入内存的数据集，HDF5 可能是一个不错的选择，因为你可以高效地读取和写入更大数组的小部分。

要开始使用 HDF5 和 pandas，你必须首先通过使用 conda 安装 `tables` 包来安装 PyTables：

```bash
conda install pytables
```

注意
请注意，PyTables 包在 PyPI 中被称为 "tables"，因此如果你使用 pip 安装，你将需要运行 `pip install tables`。

虽然可以使用 PyTables 或 h5py 库直接访问 HDF5 文件，但 pandas 提供了一个简化存储 Series 和 DataFrame 对象的高级接口。`HDFStore` 类的工作方式类似于字典，并处理底层细节：

```python
In [113]: frame = pd.DataFrame({"a": np.random.standard_normal(100)})

In [114]: store = pd.HDFStore("examples/mydata.h5")

In [115]: store["obj1"] = frame

In [116]: store["obj1_col"] = frame["a"]

In [117]: store
Out[117]: 
<class 'pandas.io.pytables.HDFStore'>
File path: examples/mydata.h5
```

然后可以使用相同的类似字典的 API 检索 HDF5 文件中包含的对象：

```python
In [118]: store["obj1"]
Out[118]: 
           a
0  -0.204708
1   0.478943
2  -0.519439
3  -0.555730
4   1.965781
..       ...
95  0.795253
96  0.118110
97 -0.748532
98  0.584970
99  0.152677
[100 rows x 1 columns]
```

`HDFStore` 支持两种存储模式，`"fixed"` 和 `"table"`（默认为 `"fixed"`）。后者通常较慢，但它支持使用特殊语法进行查询操作：

```python
In [119]: store.put("obj2", frame, format="table")

In [120]: store.select("obj2", where=["index >= 10 and index <= 15"])
Out[120]: 
           a
10  1.007189
11 -1.296221
12  0.274992
13  0.228913
14  1.352917
15  0.886429

In [121]: store.close()
```

`put` 是 `store["obj2"] = frame` 方法的显式版本，但允许我们设置其他选项，如存储格式。

`pandas.read_hdf` 函数为你提供了使用这些工具的快捷方式：

```python
In [122]: frame.to_hdf("examples/mydata.h5", "obj3", format="table")

In [123]: pd.read_hdf("examples/mydata.h5", "obj3", where=["index < 5"])
Out[123]: 
          a
0 -0.204708
1  0.478943
2 -0.519439
3 -0.555730
4  1.965781
```

如果你愿意，可以删除你创建的 HDF5 文件，如下所示：

```python
In [124]: import os

In [125]: os.remove("examples/mydata.h5")
```

注意
如果你正在处理存储在远程服务器（如 Amazon S3 或 HDFS）上的数据，使用为分布式存储设计的其他二进制格式（如 [Apache Parquet](http://parquet.apache.org)）可能更合适。

如果你在本地处理大量数据，我鼓励你探索 PyTables 和 h5py，看看它们如何满足你的需求。由于许多数据分析问题是 I/O 受限（而不是 CPU 受限），使用像 HDF5 这样的工具可以极大地加速你的应用程序。

注意
HDF5 _不是_ 数据库。它最适合一次写入、多次读取的数据集。虽然可以随时向文件中添加数据，但如果多个写入器同时这样做，文件可能会损坏。

## 6.3 与 Web API 交互

许多网站提供公共 API，通过 JSON 或其他格式提供数据流。有多种方法可以从 Python 访问这些 API；我推荐的一种方法是使用 [`requests` 包](http://docs.python-requests.org)，可以通过 pip 或 conda 安装：

```bash
conda install requests
```

要获取 GitHub 上 pandas 项目的最近 30 个议题（GitHub issues），我们可以使用附加库 `requests` 发起一个 `GET` HTTP 请求：

```python
In [126]: import requests

In [127]: url = "https://api.github.com/repos/pandas-dev/pandas/issues"

In [128]: resp = requests.get(url)

In [129]: resp.raise_for_status()

In [130]: resp
Out[130]: <Response [200]>
```

在使用 `requests.get` 后调用 `raise_for_status` 来检查 HTTP 错误是一个良好的实践。

响应对象的 `json` 方法将返回一个 Python 对象，其中包含解析后的 JSON 数据，形式为字典或列表（取决于返回的 JSON 类型）：

```python
In [131]: data = resp.json()

In [132]: data[0]["title"]
Out[132]: 'BUG: DataFrame.pivot mutates empty index.name attribute with typing._LiteralGenericAlias'
```

由于检索的结果基于实时数据，你运行此代码时看到的内容几乎肯定会有所不同。

`data` 中的每个元素都是一个字典，包含 GitHub 议题页面上找到的所有数据（除了评论）。我们可以直接将 `data` 传递给 `pandas.DataFrame`，并提取感兴趣的字段：

```python
In [133]: issues = pd.DataFrame(data, columns=["number", "title",
       .....:                                  "labels", "state"])

In [134]: issues
Out[134]: 
        number   
0    52629  \
1    52628   
2    52626   
3    52625   
4    52624   
..     ...   
25   52579   
26   52577   
27   52576   
28   52571   
29   52570   
                                                                             title   
0   BUG: DataFrame.pivot mutates empty index.name attribute with typing._Li...  \
1                                 DEPR: unused keywords in DTI/TDI construtors   
2                         ENH: Infer best datetime format from a random sample   
3            BUG: ArrowExtensionArray logical_op not working in all directions   
4              ENH: pandas.core.groupby.SeriesGroupBy.apply allow raw argument   
..                                                                         ...   
25                                     BUG: Axial inconsistency of pandas.diff   
26                  BUG: describe not respecting ArrowDtype in include/exclude   
27                  BUG: describe does not distinguish between Int64 and int64   
28  BUG: `pandas.DataFrame.replace` silently fails to replace category type...   
29     BUG: DataFrame.describe include/exclude do not work for arrow datatypes   
                                                                            labels   
0   [{'id': 76811, 'node_id': 'MDU6TGFiZWw3NjgxMQ==', 'url': 'https://api.g...  \
1                                                                           []   
2                                                                           []   
3   [{'id': 76811, 'node_id': 'MDU6TGFiZWw3NjgxMQ==', 'url': 'https://api.g...   
4   [{'id': 76812, 'node_id': 'MDU6TGFiZWw3NjgxMg==', 'url': 'https://api.g...   
..                                                                         ...   
25  [{'id': 76811, 'node_id': 'MDU6TGFiZWw3NjgxMQ==', 'url': 'https://api.g...   
26  [{'id': 3303158446, 'node_id': 'MDU6TGFiZWwzMzAzMTU4NDQ2', 'url': 'http...   
27  [{'id': 76811, 'node_id': 'MDU6TGFiZWw3NjgxMQ==', 'url': 'https://api.g...   
28  [{'id': 76811, 'node_id': 'MDU6TGFiZWw3NjgxMQ==', 'url': 'https://api.g...   
29  [{'id': 76811, 'node_id': 'MDU6TGFiZWw3NjgxMQ==', 'url': 'https://api.g...   
   state
0   open
1   open
2   open
3   open
4   open
..   ...
25  open
26  open
27  open
28  open
29  open
[30 rows x 4 columns]
```

花些功夫，你可以为常见的 Web API 创建一些更高级的接口，这些接口返回 DataFrame 对象，以便进行更方便的分析。

## 6.4 与数据库交互

在商业环境中，大量数据可能并不存储在文本或 Excel 文件中。基于 SQL 的关系型数据库（如 SQL Server、PostgreSQL 和 MySQL）被广泛使用，同时许多替代型数据库也相当流行。数据库的选择通常取决于应用程序的性能、数据完整性和可扩展性需求。

pandas 提供了一些函数来简化将 SQL 查询结果加载到 DataFrame 中的过程。作为示例，我将使用 Python 内置的 `sqlite3` 驱动程序创建一个 SQLite3 数据库：
    
    
    In [135]: import sqlite3
    
    In [136]: query = """
       .....: CREATE TABLE test
       .....: (a VARCHAR(20), b VARCHAR(20),
       .....:  c REAL,        d INTEGER
       .....: );"""
    
    In [137]: con = sqlite3.connect("mydata.sqlite")
    
    In [138]: con.execute(query)
    Out[138]: <sqlite3.Cursor at 0x188e40ac0>
    
    In [139]: con.commit()__

接着插入几行数据：
    
    
    In [140]: data = [("Atlanta", "Georgia", 1.25, 6),
       .....:         ("Tallahassee", "Florida", 2.6, 3),
       .....:         ("Sacramento", "California", 1.7, 5)]
    
    In [141]: stmt = "INSERT INTO test VALUES(?, ?, ?, ?)"
    
    In [142]: con.executemany(stmt, data)
    Out[142]: <sqlite3.Cursor at 0x188ed02c0>
    
    In [143]: con.commit()__

大多数 Python SQL 驱动程序在从表中查询数据时会返回元组列表：
    
    
    In [144]: cursor = con.execute("SELECT * FROM test")
    
    In [145]: rows = cursor.fetchall()
    
    In [146]: rows
    Out[146]: 
    [('Atlanta', 'Georgia', 1.25, 6),
     ('Tallahassee', 'Florida', 2.6, 3),
     ('Sacramento', 'California', 1.7, 5)]__

您可以将元组列表传递给 DataFrame 构造函数，但还需要列名信息，这些信息存储在游标的 `description` 属性中。需要注意的是，对于 SQLite3，游标的 `description` 仅提供列名（其他字段是 Python 数据库 API 规范的一部分，但值为 `None`），但对于某些其他数据库驱动程序，会提供更多列信息：
    
    
    In [147]: cursor.description
    Out[147]: 
    (('a', None, None, None, None, None, None),
     ('b', None, None, None, None, None, None),
     ('c', None, None, None, None, None, None),
     ('d', None, None, None, None, None, None))
    
    In [148]: pd.DataFrame(rows, columns=[x[0] for x in cursor.description])
    Out[148]: 
                 a           b     c  d
    0      Atlanta     Georgia  1.25  6
    1  Tallahassee     Florida  2.60  3
    2   Sacramento  California  1.70  5 __

这种繁琐操作显然不希望每次查询数据库时都重复进行。[SQLAlchemy 项目](http://www.sqlalchemy.org/)是一个流行的 Python SQL 工具包，它抽象了 SQL 数据库之间的许多常见差异。pandas 提供了一个 `read_sql` 函数，使您能够轻松地从通用的 SQLAlchemy 连接中读取数据。您可以使用 conda 安装 SQLAlchemy：
    
    
    conda install sqlalchemy

现在，我们将使用 SQLAlchemy 连接到同一个 SQLite 数据库，并读取之前创建的表中的数据：
    
    
    In [149]: import sqlalchemy as sqla
    
    In [150]: db = sqla.create_engine("sqlite:///mydata.sqlite")
    
    In [151]: pd.read_sql("SELECT * FROM test", db)
    Out[151]: 
                 a           b     c  d
    0      Atlanta     Georgia  1.25  6
    1  Tallahassee     Florida  2.60  3
    2   Sacramento  California  1.70  5 __

## 6.5 结论

获取数据通常是数据分析流程的第一步。本章我们探讨了一系列实用工具，旨在帮助您快速入门。在后续章节中，我们将深入探讨数据整理（data wrangling）、数据可视化、时间序列分析及其他主题。

* * *

  1. 完整列表请参阅 <https://www.fdic.gov/bank/individual/failed/banklist.html>。↩︎