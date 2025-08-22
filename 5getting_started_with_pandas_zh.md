# 5 pandas 入门

__

《利用 Python 进行数据分析 第三版》的开放获取网络版本现已作为[印刷版和数字版](https://amzn.to/3DyLaJc)的配套资源发布。如果您发现任何错误，[请在此处报告](https://oreilly.com/catalog/0636920519829/errata)。请注意，由 Quarto 生成的此站点的某些方面会与 O’Reilly 的印刷版和电子书版本的格式有所不同。

如果您觉得此在线版本有用，请考虑[订购纸质版](https://amzn.to/3DyLaJc)或[无 DRM 限制的电子书](https://www.ebooks.com/en-us/book/210644288/python-for-data-analysis/wes-mckinney/?affId=WES398681F)以支持作者。本网站内容不可复制或转载。代码示例遵循 MIT 许可证，可在 GitHub 或 Gitee 上找到。

pandas 将是本书后续大部分内容中关注的主要工具。它包含数据结构和数据操作工具，旨在使数据清洗和分析在 Python 中快速便捷。pandas 通常与数值计算工具（如 NumPy 和 SciPy）、分析库（如 statsmodels 和 scikit-learn）以及数据可视化库（如 matplotlib）结合使用。pandas 大量采用了 NumPy 惯用的基于数组的计算风格，尤其是基于数组的函数以及倾向于不使用 `for` 循环进行数据处理。

尽管 pandas 借鉴了许多 NumPy 的编码习惯，但最大的区别在于 pandas 专为处理表格型或异构数据而设计。相比之下，NumPy 最适合处理同类型的数值数组数据。

自 2010 年成为开源项目以来，pandas 已经发展成为一个相当庞大的库，适用于广泛的现实世界用例。开发者社区已增长到超过 2,500 名不同的贡献者，他们在使用 pandas 解决日常数据问题的同时，也帮助构建了这个项目。活跃的 pandas 开发者和用户社区是其成功的关键部分。

__

注意 

许多人不知道，自 2013 年以来，我就不再积极参与 pandas 的日常开发工作；从那时起，它完全成为一个由社区管理的项目。请务必向核心开发团队及所有贡献者的辛勤工作表示感谢！

在本书的其余部分，我将使用以下导入约定来引入 NumPy 和 pandas：
    
    
    In [1]: import numpy as np
    
    In [2]: import pandas as pd __

因此，每当您在代码中看到 `pd.`，它指的就是 pandas。您可能还会发现，由于 Series 和 DataFrame 使用非常频繁，将它们导入到本地命名空间会更方便：
    
    
    In [3]: from pandas import Series, DataFrame __

## 5.1 pandas 数据结构简介

要开始使用 pandas，你需要熟悉其两个核心数据结构：_Series_ 和 _DataFrame_。虽然它们并非适用于所有问题的通用解决方案，但为各种数据任务提供了坚实的基础。

### Series

Series 是一种一维数组状对象，包含一个值序列（类型与 NumPy 类型相似）和一个关联的数据标签数组，称为其 _索引_。最简单的 Series 仅由一个数据数组构成：
    
    
    In [14]: obj = pd.Series([4, 7, -5, 3])
    
    In [15]: obj
    Out[15]: 
    0    4
    1    7
    2   -5
    3    3
    dtype: int64 __

在交互式显示中，Series 的字符串表示形式左侧显示索引，右侧显示值。由于我们没有为数据指定索引，因此会创建一个默认索引，由整数 `0` 到 `N - 1`（其中 `N` 是数据长度）组成。你可以分别通过 Series 的 `array` 和 `index` 属性获取其数组表示和索引对象：
    
    
    In [16]: obj.array
    Out[16]: 
    <PandasArray>
    [4, 7, -5, 3]
    Length: 4, dtype: int64
    
    In [17]: obj.index
    Out[17]: RangeIndex(start=0, stop=4, step=1)__

`.array` 属性的结果是一个 `PandasArray`，它通常包装了一个 NumPy 数组，但也可以包含特殊的扩展数组类型，这将在 [第 7.3 章：扩展数据类型](https://wesmckinney.com/book/data-cleaning#pandas-ext-types) 中详细讨论。

通常，你会希望创建一个带有索引的 Series，以便用标签标识每个数据点：
    
    
    In [18]: obj2 = pd.Series([4, 7, -5, 3], index=["d", "b", "a", "c"])
    
    In [19]: obj2
    Out[19]: 
    d    4
    b    7
    a   -5
    c    3
    dtype: int64
    
    In [20]: obj2.index
    Out[20]: Index(['d', 'b', 'a', 'c'], dtype='object')__

与 NumPy 数组相比，你可以在选择单个值或一组值时使用索引中的标签：
    
    
    In [21]: obj2["a"]
    Out[21]: -5
    
    In [22]: obj2["d"] = 6
    
    In [23]: obj2[["c", "a", "d"]]
    Out[23]: 
    c    3
    a   -5
    d    6
    dtype: int64 __

这里 `["c", "a", "d"]` 被解释为索引列表，即使其中包含字符串而非整数。

使用 NumPy 函数或类似 NumPy 的操作（例如使用布尔数组进行过滤、标量乘法或应用数学函数）将保留索引与值的关联：
    
    
    In [24]: obj2[obj2 > 0]
    Out[24]: 
    d    6
    b    7
    c    3
    dtype: int64
    
    In [25]: obj2 * 2
    Out[25]: 
    d    12
    b    14
    a   -10
    c     6
    dtype: int64
    
    In [26]: import numpy as np
    
    In [27]: np.exp(obj2)
    Out[27]: 
    d     403.428793
    b    1096.633158
    a       0.006738
    c      20.085537
    dtype: float64 __

另一种理解 Series 的方式是将其视为固定长度的有序字典，因为它是一个从索引值到数据值的映射。它可以在许多使用字典的场景中发挥作用：
    
    
    In [28]: "b" in obj2
    Out[28]: True
    
    In [29]: "e" in obj2
    Out[29]: False __

如果你有存储在 Python 字典中的数据，可以通过传递该字典来创建 Series：
    
    
    In [30]: sdata = {"Ohio": 35000, "Texas": 71000, "Oregon": 16000, "Utah": 5000}
    
    In [31]: obj3 = pd.Series(sdata)
    
    In [32]: obj3
    Out[32]: 
    Ohio      35000
    Texas     71000
    Oregon    16000
    Utah       5000
    dtype: int64 __

Series 可以通过其 `to_dict` 方法转换回字典：
    
    
    In [33]: obj3.to_dict()
    Out[33]: {'Ohio': 35000, 'Texas': 71000, 'Oregon': 16000, 'Utah': 5000}__

当你仅传递字典时，结果 Series 中的索引将遵循字典 `keys` 方法的键顺序，这取决于键的插入顺序。你可以通过传递一个包含字典键的索引来覆盖此行为，以指定它们在结果 Series 中的出现顺序：
    
    
    In [34]: states = ["California", "Ohio", "Oregon", "Texas"]
    
    In [35]: obj4 = pd.Series(sdata, index=states)
    
    In [36]: obj4
    Out[36]: 
    California        NaN
    Ohio          35000.0
    Oregon        16000.0
    Texas         71000.0
    dtype: float64 __

此处，`sdata` 中的三个值被放置在了适当的位置，但由于未找到 `"California"` 的值，因此显示为 `NaN`（非数字），在 pandas 中用于标记缺失或 _NA_ 值。由于 `"Utah"` 未包含在 `states` 中，因此被排除在结果对象之外。

我将互换使用“缺失”、“NA”或“空值”这些术语来指代缺失数据。pandas 中的 `isna` 和 `notna` 函数应用于检测缺失数据：
    
    
    In [37]: pd.isna(obj4)
    Out[37]: 
    California     True
    Ohio          False
    Oregon        False
    Texas         False
    dtype: bool
    
    In [38]: pd.notna(obj4)
    Out[38]: 
    California    False
    Ohio           True
    Oregon         True
    Texas          True
    dtype: bool __

Series 也有这些作为实例方法：
    
    
    In [39]: obj4.isna()
    Out[39]: 
    California     True
    Ohio          False
    Oregon        False
    Texas         False
    dtype: bool __

我将在 [第 7 章：数据清洗与准备](https://wesmckinney.com/book/data-cleaning) 中更详细地讨论处理缺失数据。

Series 的一个有用特性是，在算术运算中它会自动按索引标签对齐，这在许多应用中非常实用：
    
    
    In [40]: obj3
    Out[40]: 
    Ohio      35000
    Texas     71000
    Oregon    16000
    Utah       5000
    dtype: int64
    
    In [41]: obj4
    Out[41]: 
    California        NaN
    Ohio          35000.0
    Oregon        16000.0
    Texas         71000.0
    dtype: float64
    
    In [42]: obj3 + obj4
    Out[42]: 
    California         NaN
    Ohio           70000.0
    Oregon         32000.0
    Texas         142000.0
    Utah               NaN
    dtype: float64 __

数据对齐特性将在后面更详细地讨论。如果你有数据库经验，可以将其视为类似于连接操作。

Series 对象本身及其索引都有一个 `name` 属性，该属性与 pandas 的其他功能区域集成：
    
    
    In [43]: obj4.name = "population"
    
    In [44]: obj4.index.name = "state"
    
    In [45]: obj4
    Out[45]: 
    state
    California        NaN
    Ohio          35000.0
    Oregon        16000.0
    Texas         71000.0
    Name: population, dtype: float64 __

Series 的索引可以通过赋值就地更改：
    
    
    In [46]: obj
    Out[46]: 
    0    4
    1    7
    2   -5
    3    3
    dtype: int64
    
    In [47]: obj.index = ["Bob", "Steve", "Jeff", "Ryan"]
    
    In [48]: obj
    Out[48]: 
    Bob      4
    Steve    7
    Jeff    -5
    Ryan     3
    dtype: int64 __

### DataFrame

DataFrame 表示一个矩形数据表，包含一个有序的、命名的列集合，每列可以是不同的值类型（数值、字符串、布尔值等）。DataFrame 具有行索引和列索引；可以将其视为共享相同索引的 Series 字典。

__

注意 

虽然 DataFrame 在物理上是二维的，但你可以使用分层索引（hierarchical indexing）以表格格式表示更高维度的数据，这一主题我们将在 [第 8 章：数据整理：连接、合并和重塑](https://wesmckinney.com/book/data-wrangling) 中讨论，它也是 pandas 中一些更高级数据处理功能的组成部分。

构建 DataFrame 的方法有很多种，但最常见的一种是从等长列表或 NumPy 数组的字典中创建：
    
    
    data = {"state": ["Ohio", "Ohio", "Ohio", "Nevada", "Nevada", "Nevada"],
            "year": [2000, 2001, 2002, 2001, 2002, 2003],
            "pop": [1.5, 1.7, 3.6, 2.4, 2.9, 3.2]}
    frame = pd.DataFrame(data)__

结果 DataFrame 会自动分配索引，与 Series 类似，并且列会根据 `data` 中键的顺序（取决于它们在字典中的插入顺序）放置：
    
    
    In [50]: frame
    Out[50]: 
        state  year  pop
    0    Ohio  2000  1.5
    1    Ohio  2001  1.7
    2    Ohio  2002  3.6
    3  Nevada  2001  2.4
    4  Nevada  2002  2.9
    5  Nevada  2003  3.2 __

__

注意 

如果你使用 Jupyter notebook，pandas DataFrame 对象将显示为更易于浏览器查看的 HTML 表格。示例请参见图 5.1。

![](images/pda3_0501.png)

图 5.1：pandas DataFrame 对象在 Jupyter 中的显示效果

对于大型 DataFrame，`head` 方法仅选择前五行：
    
    
    In [51]: frame.head()
    Out[51]: 
        state  year  pop
    0    Ohio  2000  1.5
    1    Ohio  2001  1.7
    2    Ohio  2002  3.6
    3  Nevada  2001  2.4
    4  Nevada  2002  2.9 __

类似地，`tail` 返回最后五行：
    
    
    In [52]: frame.tail()
    Out[52]: 
        state  year  pop
    1    Ohio  2001  1.7
    2    Ohio  2002  3.6
    3  Nevada  2001  2.4
    4  Nevada  2002  2.9
    5  Nevada  2003  3.2 __

如果你指定了列的顺序，DataFrame 的列将按该顺序排列：
    
    
    In [53]: pd.DataFrame(data, columns=["year", "state", "pop"])
    Out[53]: 
       year   state  pop
    0  2000    Ohio  1.5
    1  2001    Ohio  1.7
    2  2002    Ohio  3.6
    3  2001  Nevada  2.4
    4  2002  Nevada  2.9
    5  2003  Nevada  3.2 __

如果你传递的列不在字典中，它将在结果中显示为缺失值：
    
    
    In [54]: frame2 = pd.DataFrame(data, columns=["year", "state", "pop", "debt"])
    
    In [55]: frame2
    Out[55]: 
       year   state  pop debt
    0  2000    Ohio  1.5  NaN
    1  2001    Ohio  1.7  NaN
    2  2002    Ohio  3.6  NaN
    3  2001  Nevada  2.4  NaN
    4  2002  Nevada  2.9  NaN
    5  2003  Nevada  3.2  NaN
    
    In [56]: frame2.columns
    Out[56]: Index(['year', 'state', 'pop', 'debt'], dtype='object')__

DataFrame 中的列可以通过类似字典的表示法或点属性表示法检索为 Series：
    
    
    In [57]: frame2["state"]
    Out[57]: 
    0      Ohio
    1      Ohio
    2      Ohio
    3    Nevada
    4    Nevada
    5    Nevada
    Name: state, dtype: object
    
    In [58]: frame2.year
    Out[58]: 
    0    2000
    1    2001
    2    2002
    3    2001
    4    2002
    5    2003
    Name: year, dtype: int64 __

__

注意 

类属性访问（例如 `frame2.year`）和 IPython 中的列名选项卡补全功能是为了方便而提供的。

`frame2[column]` 适用于任何列名，但 `frame2.column` 仅当列名是有效的 Python 变量名且不与 DataFrame 中的任何方法名冲突时才有效。例如，如果列名包含空格或下划线以外的符号，则无法使用点属性方法访问。

请注意，返回的 Series 具有与 DataFrame 相同的索引，并且它们的 `name` 属性已适当设置。

行也可以通过特殊属性 `iloc` 和 `loc` 按位置或名称检索（更多内容请参见“使用 loc 和 iloc 在 DataFrame 上进行选择”）：
    
    
    In [59]: frame2.loc[1]
    Out[59]: 
    year     2001
    state    Ohio
    pop       1.7
    debt      NaN
    Name: 1, dtype: object
    
    In [60]: frame2.iloc[2]
    Out[60]: 
    year     2002
    state    Ohio
    pop       3.6
    debt      NaN
    Name: 2, dtype: object __

可以通过赋值修改列。例如，可以为空的 `debt` 列分配标量值或数组值：
    
    
    In [61]: frame2["debt"] = 16.5
    
    In [62]: frame2
    Out[62]: 
       year   state  pop  debt
    0  2000    Ohio  1.5  16.5
    1  2001    Ohio  1.7  16.5
    2  2002    Ohio  3.6  16.5
    3  2001  Nevada  2.4  16.5
    4  2002  Nevada  2.9  16.5
    5  2003  Nevada  3.2  16.5
    
    In [63]: frame2["debt"] = np.arange(6.)
    
    In [64]: frame2
    Out[64]: 
       year   state  pop  debt
    0  2000    Ohio  1.5   0.0
    1  2001    Ohio  1.7   1.0
    2  2002    Ohio  3.6   2.0
    3  2001  Nevada  2.4   3.0
    4  2002  Nevada  2.9   4.0
    5  2003  Nevada  3.2   5.0 __

当你将列表或数组分配给列时，值的长度必须与 DataFrame 的长度匹配。如果你分配一个 Series，其标签将精确重新对齐到 DataFrame 的索引，并在任何不存在的索引值处插入缺失值：
    
    
    In [65]: val = pd.Series([-1.2, -1.5, -1.7], index=[2, 4, 5])
    
    In [66]: frame2["debt"] = val
    
    In [67]: frame2
    Out[67]: 
       year   state  pop  debt
    0  2000    Ohio  1.5   NaN
    1  2001    Ohio  1.7   NaN
    2  2002    Ohio  3.6  -1.2
    3  2001  Nevada  2.4   NaN
    4  2002  Nevada  2.9  -1.5
    5  2003  Nevada  3.2  -1.7 __

分配不存在的列将创建新列。

`del` 关键字可以像字典一样删除列。例如，我首先添加一个新的布尔值列，其中 `state` 列等于 `"Ohio"`：
    
    
    In [68]: frame2["eastern"] = frame2["state"] == "Ohio"
    
    In [69]: frame2
    Out[69]: 
       year   state  pop  debt  eastern
    0  2000    Ohio  1.5   NaN     True
    1  2001    Ohio  1.7   NaN     True
    2  2002    Ohio  3.6  -1.2     True
    3  2001  Nevada  2.4   NaN    False
    4  2002  Nevada  2.9  -1.5    False
    5  2003  Nevada  3.2  -1.7    False __

__

注意 

不能使用 `frame2.eastern` 点属性表示法创建新列。

然后可以使用 `del` 方法删除此列：
    
    
    In [70]: del frame2["eastern"]
    
    In [71]: frame2.columns
    Out[71]: Index(['year', 'state', 'pop', 'debt'], dtype='object')__

__

注意 

从 DataFrame 索引返回的列是底层数据的 _视图_，而不是副本。因此，对 Series 的任何就地修改都会反映在 DataFrame 中。可以使用 Series 的 `copy` 方法显式复制列。

另一种常见的数据形式是字典的嵌套字典：
    
    
    In [72]: populations = {"Ohio": {2000: 1.5, 2001: 1.7, 2002: 3.6},
       ....:                "Nevada": {2001: 2.4, 2002: 2.9}}__

如果将嵌套字典传递给 DataFrame，pandas 会将外部字典键解释为列，内部键解释为行索引：
    
    
    In [73]: frame3 = pd.DataFrame(populations)
    
    In [74]: frame3
    Out[74]: 
          Ohio  Nevada
    2000   1.5     NaN
    2001   1.7     2.4
    2002   3.6     2.9 __

你可以使用与 NumPy 数组类似的语法转置 DataFrame（交换行和列）：
    
    
    In [75]: frame3.T
    Out[75]: 
            2000  2001  2002
    Ohio     1.5   1.7   3.6
    Nevada   NaN   2.4   2.9 __

__

警告 

请注意，如果列并非全部具有相同的数据类型，转置会丢弃列数据类型，因此转置后再转置回来可能会丢失先前的类型信息。在这种情况下，列会变为纯 Python 对象的数组。

内部字典中的键被组合以形成结果中的索引。如果指定了显式索引，则情况并非如此：
    
    
    In [76]: pd.DataFrame(populations, index=[2001, 2002, 2003])
    Out[76]: 
          Ohio  Nevada
    2001   1.7     2.4
    2002   3.6     2.9
    2003   NaN     NaN __

Series 字典的处理方式大致相同：
    
    
    In [77]: pdata = {"Ohio": frame3["Ohio"][:-1],
       ....:          "Nevada": frame3["Nevada"][:2]}
    
    In [78]: pd.DataFrame(pdata)
    Out[78]: 
          Ohio  Nevada
    2000   1.5     NaN
    2001   1.7     2.4 __

有关可以传递给 DataFrame 构造函数的许多内容的列表，请参见表 5.1。

表 5.1：DataFrame 构造函数的可能数据输入类型 类型 | 说明  
---|---  
二维 ndarray | 数据矩阵，传递可选的行和列标签  
数组、列表或元组的字典 | 每个序列成为 DataFrame 中的一列；所有序列必须长度相同  
NumPy 结构化/记录数组 | 视为“数组字典”的情况  
Series 字典 | 每个值成为一列；如果没有传递显式索引，则每个 Series 的索引被联合起来形成结果的行索引  
字典的字典 | 每个内部字典成为一列；键被联合起来形成行索引，如同“Series 字典”的情况  
字典或 Series 的列表 | 每个项目成为 DataFrame 中的一行；字典键或 Series 索引的联合成为 DataFrame 的列标签  
列表或元组的列表 | 视为“二维 ndarray”的情况  
另一个 DataFrame | 使用 DataFrame 的索引，除非传递了不同的索引  
NumPy 掩码数组 | 类似于“二维 ndarray”的情况，但掩码值在 DataFrame 结果中缺失  
  
如果 DataFrame 的 `index` 和 `columns` 设置了 `name` 属性，这些也会显示：
    
    
    In [79]: frame3.index.name = "year"
    
    In [80]: frame3.columns.name = "state"
    
    In [81]: frame3
    Out[81]: 
    state  Ohio  Nevada
    year               
    2000    1.5     NaN
    2001    1.7     2.4
    2002    3.6     2.9 __

与 Series 不同，DataFrame 没有 `name` 属性。DataFrame 的 `to_numpy` 方法将 DataFrame 中包含的数据作为二维 ndarray 返回：
    
    
    In [82]: frame3.to_numpy()
    Out[82]: 
    array([[1.5, nan],
           [1.7, 2.4],
           [3.6, 2.9]])__

如果 DataFrame 的列是不同的数据类型，则返回数组的数据类型将被选择以容纳所有列：
    
    
    In [83]: frame2.to_numpy()
    Out[83]: 
    array([[2000, 'Ohio', 1.5, nan],
           [2001, 'Ohio', 1.7, nan],
           [2002, 'Ohio', 3.6, -1.2],
           [2001, 'Nevada', 2.4, nan],
           [2002, 'Nevada', 2.9, -1.5],
           [2003, 'Nevada', 3.2, -1.7]], dtype=object)__

### 索引对象

pandas 的索引对象负责保存轴标签（包括 DataFrame 的列名）和其他元数据（如轴名称）。在构造 Series 或 DataFrame 时使用的任何数组或其他标签序列都会在内部转换为索引：
    
    
    In [84]: obj = pd.Series(np.arange(3), index=["a", "b", "c"])
    
    In [85]: index = obj.index
    
    In [86]: index
    Out[86]: Index(['a', 'b', 'c'], dtype='object')
    
    In [87]: index[1:]
    Out[87]: Index(['b', 'c'], dtype='object')__

索引对象是不可变的，因此用户无法修改：
    
    
    index[1] = "d"  # TypeError __

不可变性使得在数据结构之间共享索引对象更加安全：
    
    
    In [88]: labels = pd.Index(np.arange(3))
    
    In [89]: labels
    Out[89]: Index([0, 1, 2], dtype='int64')
    
    In [90]: obj2 = pd.Series([1.5, -2.5, 0], index=labels)
    
    In [91]: obj2
    Out[91]: 
    0    1.5
    1   -2.5
    2    0.0
    dtype: float64
    
    In [92]: obj2.index is labels
    Out[92]: True __

__

注意 

有些用户不会经常利用索引提供的功能，但由于某些操作会产生包含索引数据的结果，因此理解它们的工作原理非常重要。

除了类似数组之外，索引还表现得像固定大小的集合：
    
    
    In [93]: frame3
    Out[93]: 
    state  Ohio  Nevada
    year               
    2000    1.5     NaN
    2001    1.7     2.4
    2002    3.6     2.9
    
    In [94]: frame3.columns
    Out[94]: Index(['Ohio', 'Nevada'], dtype='object', name='state')
    
    In [95]: "Ohio" in frame3.columns
    Out[95]: True
    
    In [96]: 2003 in frame3.index
    Out[96]: False __

与 Python 集合不同，pandas 索引可以包含重复的标签：
    
    
    In [97]: pd.Index(["foo", "foo", "bar", "bar"])
    Out[97]: Index(['foo', 'foo', 'bar', 'bar'], dtype='object')__

使用重复标签进行选择将选择该标签的所有出现。

每个索引都有许多用于集合逻辑的方法和属性，可以回答关于其包含数据的其他常见问题。一些有用的方法总结在表 5.2 中。

表 5.2：一些索引方法和属性 方法/属性 | 描述  
---|---  
`append()` | 与其他索引对象连接，生成新索引  
`difference()` | 计算集合差集作为索引  
`intersection()` | 计算集合交集  
`union()` | 计算集合并集  
`isin()` | 计算布尔数组，指示每个值是否包含在传递的集合中  
`delete()` | 计算删除索引 `i` 处元素的新索引  
`drop()` | 通过删除传递的值计算新索引  
`insert()` | 通过在索引 `i` 处插入元素计算新索引  
`is_monotonic` | 如果每个元素大于或等于前一个元素，则返回 `True`  
`is_unique` | 如果索引没有重复值，则返回 `True`  
`unique()` | 计算索引中唯一值的数组

## 5.2 核心功能 (Essential Functionality)

本节将引导您了解与 Series 或 DataFrame 中所含数据交互的基本机制。在后续章节中，我们将更深入地探讨使用 pandas 进行数据分析和操作的主题。本书并非旨在作为 pandas 库的详尽文档；相反，我们将重点让您熟悉常用的功能，而将不太常见（即更晦涩）的内容留给您通过阅读在线 pandas 文档来进一步学习。

### 重新索引 (Reindexing)

pandas 对象的一个重要方法是 `reindex`，其含义是创建一个新对象，其中的数据经过重新排列以与新索引对齐。来看一个例子：

```python
In [98]: obj = pd.Series([4.5, 7.2, -5.3, 3.6], index=["d", "b", "a", "c"])

In [99]: obj
Out[99]: 
d    4.5
b    7.2
a   -5.3
c    3.6
dtype: float64
```

在此 Series 上调用 `reindex` 会根据新索引重新排列数据，如果任何索引值原本不存在，则会引入缺失值：

```python
In [100]: obj2 = obj.reindex(["a", "b", "c", "d", "e"])

In [101]: obj2
Out[101]: 
a   -5.3
b    7.2
c    3.6
d    4.5
e    NaN
dtype: float64
```

对于像时间序列这样的有序数据，重新索引时可能需要进行一些插值或填充值。`method` 选项允许我们这样做，例如使用 `ffill` 方法进行前向填充：

```python
In [102]: obj3 = pd.Series(["blue", "purple", "yellow"], index=[0, 2, 4])

In [103]: obj3
Out[103]: 
0      blue
2    purple
4    yellow
dtype: object

In [104]: obj3.reindex(np.arange(6), method="ffill")
Out[104]: 
0      blue
1      blue
2    purple
3    purple
4    yellow
5    yellow
dtype: object
```

对于 DataFrame，`reindex` 可以更改（行）索引、列或两者。当仅传递一个序列时，它会在结果中重新索引行：

```python
In [105]: frame = pd.DataFrame(np.arange(9).reshape((3, 3)),
   .....:                      index=["a", "c", "d"],
   .....:                      columns=["Ohio", "Texas", "California"])

In [106]: frame
Out[106]: 
   Ohio  Texas  California
a     0      1           2
c     3      4           5
d     6      7           8

In [107]: frame2 = frame.reindex(index=["a", "b", "c", "d"])

In [108]: frame2
Out[108]: 
   Ohio  Texas  California
a   0.0    1.0         2.0
b   NaN    NaN         NaN
c   3.0    4.0         5.0
d   6.0    7.0         8.0
```

可以使用 `columns` 关键字重新索引列：

```python
In [109]: states = ["Texas", "Utah", "California"]

In [110]: frame.reindex(columns=states)
Out[110]: 
   Texas  Utah  California
a      1   NaN           2
c      4   NaN           5
d      7   NaN           8
```

因为 `"Ohio"` 不在 `states` 中，所以该列的数据会从结果中删除。

重新索引特定轴的另一种方法是将新轴标签作为位置参数传递，然后使用 `axis` 关键字指定要重新索引的轴：

```python
In [111]: frame.reindex(states, axis="columns")
Out[111]: 
   Texas  Utah  California
a      1   NaN           2
c      4   NaN           5
d      7   NaN           8
```

有关 `reindex` 参数的更多信息，请参见表 5.3。

表 5.3: `reindex` 函数参数
参数 | 描述  
---|---  
`labels` | 用作索引的新序列。可以是 Index 实例或任何其他类似序列的 Python 数据结构。Index 将完全按原样使用，无需任何复制。  
`index` | 使用传递的序列作为新的索引标签。  
`columns` | 使用传递的序列作为新的列标签。  
`axis` | 要重新索引的轴，无论是 `"index"`（行）还是 `"columns"`。默认为 `"index"`。您也可以使用 `reindex(index=new_labels)` 或 `reindex(columns=new_labels)`。  
`method` | 插值（填充）方法；`"ffill"` 向前填充，而 `"bfill"` 向后填充。  
`fill_value` | 通过重新索引引入缺失数据时使用的替代值。当您希望结果中缺失的标签具有空值时，请使用 `fill_value="missing"`（默认行为）。  
`limit` | 向前或向后填充时，要填充的最大间隙大小（以元素数量计）。  
`tolerance` | 向前或向后填充时，为不精确匹配填充的最大间隙大小（以绝对数值距离计）。  
`level` | 在 MultiIndex 的级别上匹配简单索引；否则选择子集。  
`copy` | 如果为 `True`，即使新索引与旧索引等效，也始终复制底层数据；如果为 `False`，当索引等效时不复制数据。  

正如我们稍后在“使用 loc 和 iloc 在 DataFrame 上进行选择”中探讨的那样，您也可以使用 `loc` 操作符进行重新索引，许多用户更喜欢始终以这种方式操作。这仅当所有新索引标签都已存在于 DataFrame 中时才有效（而 `reindex` 会为新标签插入缺失数据）：

```python
In [112]: frame.loc[["a", "d", "c"], ["California", "Texas"]]
Out[112]: 
   California  Texas
a           2      1
d           8      7
c           5      4
```

### 从轴中删除条目 (Dropping Entries from an Axis)

如果您已经有一个不包含某些条目的索引数组或列表，那么从轴中删除一个或多个条目很简单，因为您可以使用 `reindex` 方法或基于 `.loc` 的索引。由于这可能需要进行一些处理和集合逻辑，`drop` 方法将返回一个新对象，其中已从轴中删除指定的值：

```python
In [113]: obj = pd.Series(np.arange(5.), index=["a", "b", "c", "d", "e"])

In [114]: obj
Out[114]: 
a    0.0
b    1.0
c    2.0
d    3.0
e    4.0
dtype: float64

In [115]: new_obj = obj.drop("c")

In [116]: new_obj
Out[116]: 
a    0.0
b    1.0
d    3.0
e    4.0
dtype: float64

In [117]: obj.drop(["d", "c"])
Out[117]: 
a    0.0
b    1.0
e    4.0
dtype: float64
```

对于 DataFrame，可以从任一轴删除索引值。为了说明这一点，我们首先创建一个示例 DataFrame：

```python
In [118]: data = pd.DataFrame(np.arange(16).reshape((4, 4)),
   .....:                     index=["Ohio", "Colorado", "Utah", "New York"],
   .....:                     columns=["one", "two", "three", "four"])

In [119]: data
Out[119]: 
          one  two  three  four
Ohio        0    1      2     3
Colorado    4    5      6     7
Utah        8    9     10    11
New York   12   13     14    15
```

使用标签序列调用 `drop` 将从行标签（轴 0）中删除值：

```python
In [120]: data.drop(index=["Colorado", "Ohio"])
Out[120]: 
          one  two  three  four
Utah        8    9     10    11
New York   12   13     14    15
```

要删除列中的标签，请改用 `columns` 关键字：

```python
In [121]: data.drop(columns=["two"])
Out[121]: 
          one  three  four
Ohio        0      2     3
Colorado    4      6     7
Utah        8     10    11
New York   12     14    15
```

您还可以通过传递 `axis=1`（类似于 NumPy）或 `axis="columns"` 从列中删除值：

```python
In [122]: data.drop("two", axis=1)
Out[122]: 
          one  three  four
Ohio        0      2     3
Colorado    4      6     7
Utah        8     10    11
New York   12     14    15

In [123]: data.drop(["two", "four"], axis="columns")
Out[123]: 
          one  three
Ohio        0      2
Colorado    4      6
Utah        8     10
New York   12     14
```

### 索引、选择和过滤 (Indexing, Selection, and Filtering)

Series 索引（`obj[...]`）类似于 NumPy 数组索引，不同之处在于您可以使用 Series 的索引值，而不仅仅是整数。以下是一些示例：

```python
In [124]: obj = pd.Series(np.arange(4.), index=["a", "b", "c", "d"])

In [125]: obj
Out[125]: 
a    0.0
b    1.0
c    2.0
d    3.0
dtype: float64

In [126]: obj["b"]
Out[126]: 1.0

In [127]: obj[1]
Out[127]: 1.0

In [128]: obj[2:4]
Out[128]: 
c    2.0
d    3.0
dtype: float64

In [129]: obj[["b", "a", "d"]]
Out[129]: 
b    1.0
a    0.0
d    3.0
dtype: float64

In [130]: obj[[1, 3]]
Out[130]: 
b    1.0
d    3.0
dtype: float64

In [131]: obj[obj < 2]
Out[131]: 
a    0.0
b    1.0
dtype: float64
```

虽然您可以通过标签以这种方式选择数据，但选择索引值的首选方法是使用特殊的 `loc` 操作符：

```python
In [132]: obj.loc[["b", "a", "d"]]
Out[132]: 
b    1.0
a    0.0
d    3.0
dtype: float64
```

首选 `loc` 的原因在于使用 `[]` 进行索引时对整数的不同处理。如果索引包含整数，常规的基于 `[]` 的索引会将整数视为标签，因此行为会根据索引的数据类型而有所不同。例如：

```python
In [133]: obj1 = pd.Series([1, 2, 3], index=[2, 0, 1])

In [134]: obj2 = pd.Series([1, 2, 3], index=["a", "b", "c"])

In [135]: obj1
Out[135]: 
2    1
0    2
1    3
dtype: int64

In [136]: obj2
Out[136]: 
a    1
b    2
c    3
dtype: int64

In [137]: obj1[[0, 1, 2]]
Out[137]: 
0    2
1    3
2    1
dtype: int64

In [138]: obj2[[0, 1, 2]]
Out[138]: 
a    1
b    2
c    3
dtype: int64
```

当使用 `loc` 时，如果索引不包含整数，表达式 `obj.loc[[0, 1, 2]]` 将失败：

```python
In [134]: obj2.loc[[0, 1]]
---------------------------------------------------------------------------
KeyError                                  Traceback (most recent call last)
/tmp/ipykernel_804589/4185657903.py in <module>
----> 1 obj2.loc[[0, 1]]

^ 长异常信息已省略 ^

KeyError: "None of [Int64Index([0, 1], dtype='int64')] are in the [index]"
```

由于 `loc` 操作符专门使用标签进行索引，因此还有一个 `iloc` 操作符专门使用整数进行索引，无论索引是否包含整数，都能保持一致地工作：

```python
In [139]: obj1.iloc[[0, 1, 2]]
Out[139]: 
2    1
0    2
1    3
dtype: int64

In [140]: obj2.iloc[[0, 1, 2]]
Out[140]: 
a    1
b    2
c    3
dtype: int64
```

注意

您也可以使用标签进行切片，但其工作方式与普通的 Python 切片不同，因为端点包含在内：

```python
In [141]: obj2.loc["b":"c"]
Out[141]: 
b    2
c    3
dtype: int64
```

使用这些方法赋值会修改 Series 的相应部分：

```python
In [142]: obj2.loc["b":"c"] = 5

In [143]: obj2
Out[143]: 
a    1
b    5
c    5
dtype: int64
```

注意

新手常犯的一个错误是尝试像调用函数一样调用 `loc` 或 `iloc`，而不是使用方括号对它们进行“索引”。方括号符号用于启用切片操作，并允许在 DataFrame 对象上对多个轴进行索引。

对 DataFrame 进行索引可以检索一个或多个列，使用单个值或序列：

```python
In [144]: data = pd.DataFrame(np.arange(16).reshape((4, 4)),
   .....:                     index=["Ohio", "Colorado", "Utah", "New York"],
   .....:                     columns=["one", "two", "three", "four"])

In [145]: data
Out[145]: 
          one  two  three  four
Ohio        0    1      2     3
Colorado    4    5      6     7
Utah        8    9     10    11
New York   12   13     14    15

In [146]: data["two"]
Out[146]: 
Ohio         1
Colorado     5
Utah         9
New York    13
Name: two, dtype: int64

In [147]: data[["three", "one"]]
Out[147]: 
          three  one
Ohio          2    0
Colorado      6    4
Utah         10    8
New York     14   12
```

像这样的索引有一些特殊情况。第一种是使用布尔数组进行切片或选择数据：

```python
In [148]: data[:2]
Out[148]: 
          one  two  three  four
Ohio        0    1      2     3
Colorado    4    5      6     7

In [149]: data[data["three"] > 5]
Out[149]: 
          one  two  three  four
Colorado    4    5      6     7
Utah        8    9     10    11
New York   12   13     14    15
```

行选择语法 `data[:2]` 是为了方便而提供的。向 `[]` 操作符传递单个元素或列表会选择列。

另一种情况是使用布尔 DataFrame 进行索引，例如由标量比较产生的布尔 DataFrame。考虑一个通过与标量值比较产生的全为布尔值的 DataFrame：

```python
In [150]: data < 5
Out[150]: 
            one    two  three   four
Ohio       True   True   True   True
Colorado   True  False  False  False
Utah      False  False  False  False
New York  False  False  False  False
```

我们可以使用此 DataFrame 将每个值为 `True` 的位置赋值为 0，如下所示：

```python
In [151]: data[data < 5] = 0

In [152]: data
Out[152]: 
          one  two  three  four
Ohio        0    0      0     0
Colorado    0    5      6     7
Utah        8    9     10    11
New York   12   13     14    15
```

#### 使用 loc 和 iloc 在 DataFrame 上进行选择 (Selection on DataFrame with loc and iloc)

与 Series 类似，DataFrame 也有特殊的属性 `loc` 和 `iloc`，分别用于基于标签和基于整数的索引。由于 DataFrame 是二维的，您可以使用轴标签（`loc`）或整数（`iloc`）通过类似 NumPy 的表示法来选择行和列的子集。

作为第一个示例，让我们通过标签选择单行：

```python
In [153]: data
Out[153]: 
          one  two  three  four
Ohio        0    0      0     0
Colorado    0    5      6     7
Utah        8    9     10    11
New York   12   13     14    15

In [154]: data.loc["Colorado"]
Out[154]: 
one      0
two      5
three    6
four     7
Name: Colorado, dtype: int64
```

选择单行的结果是一个 Series，其索引包含 DataFrame 的列标签。要选择多行并创建新的 DataFrame，请传递一个标签序列：

```python
In [155]: data.loc[["Colorado", "New York"]]
Out[155]: 
          one  two  three  four
Colorado    0    5      6     7
New York   12   13     14    15
```

您可以在 `loc` 中通过逗号分隔选择来组合行和列选择：

```python
In [156]: data.loc["Colorado", ["two", "three"]]
Out[156]: 
two      5
three    6
Name: Colorado, dtype: int64
```

然后，我们使用 `iloc` 用整数进行一些类似的选择：

```python
In [157]: data.iloc[2]
Out[157]: 
one       8
two       9
three    10
four     11
Name: Utah, dtype: int64

In [158]: data.iloc[[2, 1]]
Out[158]: 
          one  two  three  four
Utah        8    9     10    11
Colorado    0    5      6     7

In [159]: data.iloc[2, [3, 0, 1]]
Out[159]: 
four    11
one      8
two      9
Name: Utah, dtype: int64

In [160]: data.iloc[[1, 2], [3, 0, 1]]
Out[160]: 
          four  one  two
Colorado     7    0    5
Utah        11    8    9
```

除了单个标签或标签列表外，这两个索引函数还可以与切片一起使用：

```python
In [161]: data.loc[:"Utah", "two"]
Out[161]: 
Ohio        0
Colorado    5
Utah        9
Name: two, dtype: int64

In [162]: data.iloc[:, :3][data.three > 5]
Out[162]: 
          one  two  three
Colorado    0    5      6
Utah        8    9     10
New York   12   13     14
```

布尔数组可以与 `loc` 一起使用，但不能与 `iloc` 一起使用：

```python
In [163]: data.loc[data.three >= 2]
Out[163]: 
          one  two  three  four
Colorado    0    5      6     7
Utah        8    9     10    11
New York   12   13     14    15
```

有许多方法可以选择和重新排列 pandas 对象中包含的数据。对于 DataFrame，表 5.4 提供了其中许多方法的简要总结。正如您稍后将看到的，处理分层索引还有许多其他选项。

表 5.4: DataFrame 的索引选项
类型 | 说明  
---|---  
`df[column]` | 从 DataFrame 中选择单列或列序列；特殊情况便利方法：布尔数组（过滤行）、切片（切片行）或布尔 DataFrame（根据某些条件设置值）  
`df.loc[rows]` | 通过标签从 DataFrame 中选择单行或行子集  
`df.loc[:, cols]` | 通过标签选择单列或列子集  
`df.loc[rows, cols]` | 通过标签选择行和列  
`df.iloc[rows]` | 通过整数位置从 DataFrame 中选择单行或行子集  
`df.iloc[:, cols]` | 通过整数位置选择单列或列子集  
`df.iloc[rows, cols]` | 通过整数位置选择行和列  
`df.at[row, col]` | 通过行和列标签选择单个标量值  
`df.iat[row, col]` | 通过行和列位置（整数）选择单个标量值  
`reindex` 方法 | 通过标签选择行或列  

#### 整数索引的陷阱 (Integer indexing pitfalls)

使用由整数索引的 pandas 对象可能是新用户的一个绊脚石，因为它们的工作方式与内置的 Python 数据结构（如列表和元组）不同。例如，您可能不会期望以下代码生成错误：

```python
In [164]: ser = pd.Series(np.arange(3.))

In [165]: ser
Out[165]: 
0    0.0
1    1.0
2    2.0
dtype: float64

In [166]: ser[-1]
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
~/miniforge-x86/envs/book-env/lib/python3.10/site-packages/pandas/core/indexes/range.py in get_loc(self, key)
    344             try:
--> 345                 return self._range.index(new_key)
    346             except ValueError as err:
ValueError: -1 is not in range
The above exception was the direct cause of the following exception:
KeyError                                  Traceback (most recent call last)
<ipython-input-166-44969a759c20> in <module>
----> 1 ser[-1]
~/miniforge-x86/envs/book-env/lib/python3.10/site-packages/pandas/core/series.py in __getitem__(self, key)
   1010 
   1011         elif key_is_scalar:
-> 1012             return self._get_value(key)
   1013 
   1014         if is_hashable(key):
~/miniforge-x86/envs/book-env/lib/python3.10/site-packages/pandas/core/series.py in _get_value(self, label, takeable)
   1119 
   1120         # Similar to Index.get_value, but we do not fall back to positional
-> 1121         loc = self.index.get_loc(label)
   1122 
   1123         if is_integer(loc):
~/miniforge-x86/envs/book-env/lib/python3.10/site-packages/pandas/core/indexes/range.py in get_loc(self, key)
    345                 return self._range.index(new_key)
    346             except ValueError as err:
--> 347                 raise KeyError(key) from err
    348         self._check_indexing_error(key)
    349         raise KeyError(key)
KeyError: -1
```

在这种情况下，pandas 可以“回退”到整数索引，但通常很难做到这一点，而不会在用户代码中引入难以察觉的错误。这里我们有一个包含 `0`、`1` 和 `2` 的索引，但 pandas 不想猜测用户想要什么（基于标签的索引还是基于位置的索引）：

```python
In [167]: ser
Out[167]: 
0    0.0
1    1.0
2    2.0
dtype: float64
```

另一方面，对于非整数索引，没有这种歧义：

```python
In [168]: ser2 = pd.Series(np.arange(3.), index=["a", "b", "c"])

In [169]: ser2[-1]
Out[169]: 2.0
```

如果您有一个包含整数的轴索引，数据选择将始终是面向标签的。正如我上面所说，如果您使用 `loc`（用于标签）或 `iloc`（用于整数），您将得到 exactly 您想要的内容：

```python
In [170]: ser.iloc[-1]
Out[170]: 2.0
```

另一方面，使用整数切片始终是面向整数的：

```python
In [171]: ser[:2]
Out[171]: 
0    0.0
1    1.0
dtype: float64
```

由于这些陷阱，最好始终优先使用 `loc` 和 `iloc` 进行索引以避免歧义。

#### 链式索引的陷阱 (Pitfalls with chained indexing)

在上一节中，我们研究了如何使用 `loc` 和 `iloc` 在 DataFrame 上进行灵活的选择。这些索引属性也可用于就地修改 DataFrame 对象，但这样做需要一些小心。

例如，在上面的示例 DataFrame 中，我们可以通过标签或整数位置对列或行进行赋值：

```python
In [172]: data.loc[:, "one"] = 1

In [173]: data
Out[173]: 
          one  two  three  four
Ohio        1    0      0     0
Colorado    1    5      6     7
Utah        1    9     10    11
New York    1   13     14    15

In [174]: data.iloc[2] = 5

In [175]: data
Out[175]: 
          one  two  three  four
Ohio        1    0      0     0
Colorado    1    5      6     7
Utah        5    5      5     5
New York    1   13     14    15

In [176]: data.loc[data["four"] > 5] = 3

In [177]: data
Out[177]: 
          one  two  three  four
Ohio        1    0      0     0
Colorado    3    3      3     3
Utah        5    5      5     5
New York    3    3      3     3
```

pandas 新用户常见的一个问题是在赋值时链式选择，如下所示：

```python
In [177]: data.loc[data.three == 5]["three"] = 6
<ipython-input-11-0ed1cf2155d5>:1: SettingWithCopyWarning:
A value is trying to be set on a copy of a slice from a DataFrame.
Try using .loc[row_indexer,col_indexer] = value instead
```

根据数据内容，这可能会打印一个特殊的 `SettingWithCopyWarning`，警告您正在尝试修改一个临时值（`data.loc[data.three == 5]` 的非空结果）而不是原始 DataFrame `data`，这可能才是您的本意。这里，`data` 未被修改：

```python
In [179]: data
Out[179]: 
          one  two  three  four
Ohio        1    0      0     0
Colorado    3    3      3     3
Utah        5    5      5     5
New York    3    3      3     3
```

在这些场景中，修复方法是将链式赋值重写为使用单个 `loc` 操作：

```python
In [180]: data.loc[data.three == 5, "three"] = 6

In [181]: data
Out[181]: 
          one  two  three  four
Ohio        1    0      0     0
Colorado    3    3      3     3
Utah        5    5      6     5
New York    3    3      3     3
```

一个好的经验法则是避免在赋值时使用链式索引。还有其他情况 pandas 会生成 `SettingWithCopyWarning`，这些情况与链式索引有关。我建议您参考在线 pandas 文档中的此主题。

### 算术和数据对齐 (Arithmetic and Data Alignment)

pandas 可以大大简化处理具有不同索引的对象。例如，当您添加对象时，如果任何索引对不相同，则结果中的相应索引将是索引对的并集。让我们看一个例子：

```python
In [182]: s1 = pd.Series([7.3, -2.5, 3.4, 1.5], index=["a", "c", "d", "e"])

In [183]: s2 = pd.Series([-2.1, 3.6, -1.5, 4, 3.1],
   .....:                index=["a", "c", "e", "f", "g"])

In [184]: s1
Out[184]: 
a    7.3
c   -2.5
d    3.4
e    1.5
dtype: float64

In [185]: s2
Out[185]: 
a   -2.1
c    3.6
e   -1.5
f    4.0
g    3.1
dtype: float64
```

添加这些得到：

```python
In [186]: s1 + s2
Out[186]: 
a    5.2
c    1.1
d    NaN
e    0.0
f    NaN
g    NaN
dtype: float64
```

内部数据对齐会在不重叠的标签位置引入缺失值。缺失值将在进一步的算术计算中传播。

对于 DataFrame，对齐会在行和列上同时执行：

```python
In [187]: df1 = pd.DataFrame(np.arange(9.).reshape((3, 3)), columns=list("bcd"),
   .....:                    index=["Ohio", "Texas", "Colorado"])

In [188]: df2 = pd.DataFrame(np.arange(12.).reshape((4, 3)), columns=list("bde"),
   .....:                    index=["Utah", "Ohio", "Texas", "Oregon"])

In [189]: df1
Out[189]: 
            b    c    d
Ohio      0.0  1.0  2.0
Texas     3.0  4.0  5.0
Colorado  6.0  7.0  8.0

In [190]: df2
Out[190]: 
          b     d     e
Utah    0.0   1.0   2.0
Ohio    3.0   4.0   5.0
Texas   6.0   7.0   8.0
Oregon  9.0  10.0  11.0
```

添加这些返回一个 DataFrame，其索引和列是每个 DataFrame 中索引和列的并集：

```python
In [191]: df1 + df2
Out[191]: 
            b   c     d   e
Colorado  NaN NaN   NaN NaN
Ohio      3.0 NaN   6.0 NaN
Oregon    NaN NaN   NaN NaN
Texas     9.0 NaN  12.0 NaN
Utah      NaN NaN   NaN NaN
```

由于 `"c"` 和 `"e"` 列未在两个 DataFrame 对象中找到，因此它们在结果中显示为缺失。对于两个对象中不常见的行标签也是如此。

如果您添加没有共同列或行标签的 DataFrame 对象，结果将包含所有空值：

```python
In [192]: df1 = pd.DataFrame({"A": [1, 2]})

In [193]: df2 = pd.DataFrame({"B": [3, 4]})

In [194]: df1
Out[194]: 
   A
0  1
1  2

In [195]: df2
Out[195]: 
   B
0  3
1  4

In [196]: df1 + df2
Out[196]: 
    A   B
0 NaN NaN
1 NaN NaN
```

#### 带有填充值的算术方法 (Arithmetic methods with fill values)

在不同索引的对象之间的算术运算中，您可能希望用一个特殊值（如 0）来填充，当在一个对象中找到轴标签但在另一个对象中找不到时。以下是一个示例，我们通过将 `np.nan` 分配给它来将特定值设置为 NA（空）：

```python
In [197]: df1 = pd.DataFrame(np.arange(12.).reshape((3, 4)),
   .....:                    columns=list("abcd"))

In [198]: df2 = pd.DataFrame(np.arange(20.).reshape((4, 5)),
   .....:                    columns=list("abcde"))

In [199]: df2.loc[1, "b"] = np.nan

In [200]: df1
Out[200]: 
     a    b     c     d
0  0.0  1.0   2.0   3.0
1  4.0  5.0   6.0   7.0
2  8.0  9.0  10.0  11.0

In [201]: df2
Out[201]: 
      a     b     c     d     e
0   0.0   1.0   2.0   3.0   4.0
1   5.0   NaN   7.0   8.0   9.0
2  10.0  11.0  12.0  13.0  14.0
3  15.0  16.0  17.0  18.0  19.0
```

添加这些会在不重叠的位置产生缺失值：

```python
In [202]: df1 + df2
Out[202]: 
      a     b     c     d   e
0   0.0   2.0   4.0   6.0 NaN
1   9.0   NaN  13.0  15.0 NaN
2  18.0  20.0  22.0  24.0 NaN
3   NaN   NaN   NaN   NaN NaN
```

在 `df1` 上使用 `add` 方法，我传递 `df2` 和一个 `fill_value` 参数，该参数用传递的值替换操作中任何缺失的值：

```python
In [203]: df1.add(df2, fill_value=0)
Out[203]: 
      a     b     c     d     e
0   0.0   2.0   4.0   6.0   4.0
1   9.0   5.0  13.0  15.0   9.0
2  18.0  20.0  22.0  24.0  14.0
3  15.0  16.0  17.0  18.0  19.0
```

有关 Series 和 DataFrame 的算术方法列表，请参见表 5.5。每个都有一个以字母 `r` 开头的对应方法，其参数是反转的。因此这两个语句是等价的：

```python
In [204]: 1 / df1
Out[204]: 
          a         b         c         d
0    inf  1.000000  0.500000  0.333333
1  0.250  0.200000  0.166667  0.142857
2  0.125  0.111111  0.100000  0.090909

In [205]: df1.rdiv(1)
Out[205]: 
          a         b         c         d
0    inf  1.000000  0.500000  0.333333
1  0.250  0.200000  0.166667  0.142857
2  0.125  0.111111  0.100000  0.090909
```

相关地，当重新索引 Series 或 DataFrame 时，您也可以指定不同的填充值：

```python
In [206]: df1.reindex(columns=df2.columns, fill_value=0)
Out[206]: 
     a    b     c     d  e
0  0.0  1.0   2.0   3.0  0
1  4.0  5.0   6.0   7.0  0
2  8.0  9.0  10.0  11.0  0
```

表 5.5: 灵活的算术方法
方法 | 描述  
---|---  
`add, radd` | 加法方法（+）  
`sub, rsub` | 减法方法（-）  
`div, rdiv` | 除法方法（/）  
`floordiv, rfloordiv` | 地板除法方法（//）  
`mul, rmul` | 乘法方法（*）  
`pow, rpow` | 求幂方法（**）  

#### DataFrame 和 Series 之间的操作 (Operations between DataFrame and Series)

与不同维度的 NumPy 数组一样，DataFrame 和 Series 之间的算术运算也有定义。首先，作为一个激励示例，考虑一个二维数组与其一行之间的差异：

```python
In [207]: arr = np.arange(12.).reshape((3, 4))

In [208]: arr
Out[208]: 
array([[ 0.,  1.,  2.,  3.],
       [ 4.,  5.,  6.,  7.],
       [ 8.,  9., 10., 11.]])

In [209]: arr[0]
Out[209]: array([0., 1., 2., 3.])

In [210]: arr - arr[0]
Out[210]: 
array([[0., 0., 0., 0.],
       [4., 4., 4., 4.],
       [8., 8., 8., 8.]])
```

当我们将 `arr[0]` 从 `arr` 中减去时，减法对每一行执行一次。这被称为_广播_，并在[附录 A：高级 NumPy](https://wesmckinney.com/book/advanced-numpy) 中有关一般 NumPy 数组的部分有更详细的解释。DataFrame 和 Series 之间的操作类似：

```python
In [211]: frame = pd.DataFrame(np.arange(12.).reshape((4, 3)),
   .....:                      columns=list("bde"),
   .....:                      index=["Utah", "Ohio", "Texas", "Oregon"])

In [212]: series = frame.iloc[0]

In [213]: frame
Out[213]: 
          b     d     e
Utah    0.0   1.0   2.0
Ohio    3.0   4.0   5.0
Texas   6.0   7.0   8.0
Oregon  9.0  10.0  11.0

In [214]: series
Out[214]: 
b    0.0
d    1.0
e    2.0
Name: Utah, dtype: float64
```

默认情况下，DataFrame 和 Series 之间的算术运算将 Series 的索引与 DataFrame 的列匹配，并向下广播行：

```python
In [215]: frame - series
Out[215]: 
          b    d    e
Utah    0.0  0.0  0.0
Ohio    3.0  3.0  3.0
Texas   6.0  6.0  6.0
Oregon  9.0  9.0  9.0
```

如果在 DataFrame 的列或 Series 的索引中找不到索引值，则对象将被重新索引以形成并集：

```python
In [216]: series2 = pd.Series(np.arange(3), index=["b", "e", "f"])

In [217]: series2
Out[217]: 
b    0
e    1
f    2
dtype: int64

In [218]: frame + series2
Out[218]: 
          b   d     e   f
Utah    0.0 NaN   3.0 NaN
Ohio    3.0 NaN   6.0 NaN
Texas   6.0 NaN   9.0 NaN
Oregon  9.0 NaN  12.0 NaN
```

如果您想改为在列上广播，匹配行，则必须使用算术方法之一并指定在索引上匹配。例如：

```python
In [219]: series3 = frame["d"]

In [220]: frame
Out[220]: 
          b     d     e
Utah    0.0   1.0   2.0
Ohio    3.0   4.0   5.0
Texas   6.0   7.0   8.0
Oregon  9.0  10.0  11.0

In [221]: series3
Out[221]: 
Utah       1.0
Ohio       4.0
Texas      7.0
Oregon    10.0
Name: d, dtype: float64

In [222]: frame.sub(series3, axis="index")
Out[222]: 
          b    d    e
Utah   -1.0  0.0  1.0
Ohio   -1.0  0.0  1.0
Texas  -1.0  0.0  1.0
Oregon -1.0  0.0  1.0
```

您传递的轴是_要匹配的轴_。在这种情况下，我们的意思是在 DataFrame 的行索引（`axis="index"`）上匹配，并在列上广播。

### 函数应用和映射 (Function Application and Mapping)

NumPy 的 ufuncs（元素级数组方法）也可与 pandas 对象一起使用：

```python
In [223]: frame = pd.DataFrame(np.random.standard_normal((4, 3)),
   .....:                      columns=list("bde"),
   .....:                      index=["Utah", "Ohio", "Texas", "Oregon"])

In [224]: frame
Out[224]: 
               b         d         e
Utah   -0.204708  0.478943 -0.519439
Ohio   -0.555730  1.965781  1.393406
Texas   0.092908  0.281746  0.769023
Oregon  1.246435  1.007189 -1.296221

In [225]: np.abs(frame)
Out[225]: 
               b         d         e
Utah    0.204708  0.478943  0.519439
Ohio    0.555730  1.965781  1.393406
Texas   0.092908  0.281746  0.769023
Oregon  1.246435  1.007189  1.296221
```

另一个常见操作是将一维数组上的函数应用于每列或每行。DataFrame 的 `apply` 方法正是这样做的：

```python
In [226]: def f1(x):
   .....:     return x.max() - x.min()

In [227]: frame.apply(f1)
Out[227]: 
b    1.802165
d    1.684034
e    2.689627
dtype: float64
```

这里，计算 Series 最大值和最小值之间差异的函数 `f`，在 `frame` 的每一列上被调用一次。结果是一个 Series，其索引是 `frame` 的列。

如果您向 `apply` 传递 `axis="columns"`，该函数将在每行上调用一次。考虑这一点的一个有用方式是“跨列应用”：

```python
In [228]: frame.apply(f1, axis="columns")
Out[228]: 
Utah      0.998382
Ohio      2.521511
Texas     0.676115
Oregon    2.542656
dtype: float64
```

许多最常见的数组统计信息（如 `sum` 和 `mean`）都是 DataFrame 的方法，因此不需要使用 `apply`。

传递给 `apply` 的函数不必返回标量值；它也可以返回具有多个值的 Series：

```python
In [229]: def f2(x):
   .....:     return pd.Series([x.min(), x.max()], index=["min", "max"])

In [230]: frame.apply(f2)
Out[230]: 
            b         d         e
min -0.555730  0.281746 -1.296221
max  1.246435  1.965781  1.393406
```

也可以使用元素级的 Python 函数。假设您想根据 `frame` 中的每个浮点值计算一个格式化的字符串。您可以使用 `applymap` 来实现：

```python
In [231]: def my_format(x):
   .....:     return f"{x:.2f}"

In [232]: frame.applymap(my_format)
Out[232]: 
            b     d      e
Utah    -0.20  0.48  -0.52
Ohio    -0.56  1.97   1.39
Texas    0.09  0.28   0.77
Oregon   1.25  1.01  -1.30
```

名称 `applymap` 的原因在于 Series 有一个 `map` 方法用于应用元素级函数：

```python
In [233]: frame["e"].map(my_format)
Out[233]: 
Utah      -0.52
Ohio       1.39
Texas      0.77
Oregon    -1.30
Name: e, dtype: object
```

### 排序和排名 (Sorting and Ranking)

按某些条件对数据集进行排序是另一个重要的内置操作。要按行或列标签字典顺序排序，请使用 `sort_index` 方法，该方法返回一个新的排序对象：

```python
In [234]: obj = pd.Series(np.arange(4), index=["d", "a", "b", "c"])

In [235]: obj
Out[235]: 
d    0
a    1
b    2
c    3
dtype: int64

In [236]: obj.sort_index()
Out[236]: 
a    1
b    2
c    3
d    0
dtype: int64
```

对于 DataFrame，您可以在任一轴上按索引排序：

```python
In [237]: frame = pd.DataFrame(np.arange(8).reshape((2, 4)),
   .....:                      index=["three", "one"],
   .....:                      columns=["d", "a", "b", "c"])

In [238]: frame
Out[238]: 
        d  a  b  c
three  0  1  2  3
one    4  5  6  7

In [239]: frame.sort_index()
Out[239]: 
        d  a  b  c
one    4  5  6  7
three  0  1  2  3

In [240]: frame.sort_index(axis="columns")
Out[240]: 
        a  b  c  d
three  1  2  3  0
one    5  6  7  4
```

默认情况下，数据按升序排序，但也可以按降序排序：

```python
In [241]: frame.sort_index(axis="columns", ascending=False)
Out[241]: 
        d  c  b  a
three  0  3  2  1
one    4  7  6  5
```

要按 Series 的值排序，请使用其 `sort_values` 方法：

```python
In [242]: obj = pd.Series([4, 7, -3, 2])

In [243]: obj.sort_values()
Out[243]: 
2   -3
3    2
0    4
1    7
dtype: int64
```

默认情况下，任何缺失值都会排序到 Series 的末尾：

```python
In [244]: obj = pd.Series([4, np.nan, 7, np.nan, -3, 2])

In [245]: obj.sort_values()
Out[245]: 
4   -3.0
5    2.0
0    4.0
2    7.0
1    NaN
3    NaN
dtype: float64
```

可以使用 `na_position` 选项将缺失值排序到开头：

```python
In [246]: obj.sort_values(na_position="first")
Out[246]: 
1    NaN
3    NaN
4   -3.0
5    2.0
0    4.0
2    7.0
dtype: float64
```

在对 DataFrame 排序时，您可以使用一列或多列中的数据作为排序键。为此，将一个或多个列名传递给 `sort_values`：

```python
In [247]: frame = pd.DataFrame({"b": [4, 7, -3, 2], "a": [0, 1, 0, 1]})

In [248]: frame
Out[248]: 
   b  a
0  4  0
1  7  1
2 -3  0
3  2  1

In [249]: frame.sort_values("b")
Out[249]: 
   b  a
2 -3  0
3  2  1
0  4  0
1  7  1
```

要按多列排序，传递一个名称列表：

```python
In [250]: frame.sort_values(["a", "b"])
Out[250]: 
   b  a
2 -3  0
0  4  0
3  2  1
1  7  1
```

_排名_ 将从数组中的有效数据点数量中分配从 1 开始的排名，从最低值开始。Series 和 DataFrame 的 `rank` 方法就是用于此的；默认情况下，`rank` 通过为每个组分配平均排名来打破平局：

```python
In [251]: obj = pd.Series([7, -5, 7, 4, 2, 0, 4])

In [252]: obj.rank()
Out[252]: 
0    6.5
1    1.0
2    6.5
3    4.5
4    3.0
5    2.0
6    4.5
dtype: float64
```

排名也可以根据它们在数据中观察到的顺序进行分配：

```python
In [253]: obj.rank(method="first")
Out[253]: 
0    6.0
1    1.0
2    7.0
3    4.0
4    3.0
5    2.0
6    5.0
dtype: float64
```

在这里，条目 0 和 2 没有使用平均排名 6.5，而是被设置为 6 和 7，因为标签 0 在数据中先于标签 2。

您也可以按降序排名：

```python
In [254]: obj.rank(ascending=False)
Out[254]: 
0    1.5
1    7.0
2    1.5
3    3.5
4    5.0
5    6.0
6    3.5
dtype: float64
```

有关可用的平局打破方法列表，请参见表 5.6。

DataFrame 可以跨行或列计算排名：

```python
In [255]: frame = pd.DataFrame({"b": [4.3, 7, -3, 2], "a": [0, 1, 0, 1],
   .....:                       "c": [-2, 5, 8, -2.5]})

In [256]: frame
Out[256]: 
     b  a    c
0  4.3  0 -2.0
1  7.0  1  5.0
2 -3.0  0  8.0
3  2.0  1 -2.5

In [257]: frame.rank(axis="columns")
Out[257]: 
     b    a    c
0  3.0  2.0  1.0
1  3.0  1.0  2.0
2  1.0  2.0  3.0
3  3.0  2.0  1.0
```

表 5.6: 使用 rank 的平局打破方法
方法 | 描述  
---|---  
`"average"` | 默认：为平等组中的每个条目分配平均排名  
`"min"` | 为整个组使用最小排名  
`"max"` | 为整个组使用最大排名  
`"first"` | 按照值在数据中出现的顺序分配排名  
`"dense"` | 类似于 `method="min"`，但排名始终在组之间增加 1，而不是组中相等元素的数量  

### 具有重复标签的轴索引 (Axis Indexes with Duplicate Labels)

到目前为止，我们看到的几乎所有示例都具有唯一的轴标签（索引值）。虽然许多 pandas 函数（如 `reindex`）要求标签是唯一的，但这并不是强制性的。让我们考虑一个具有重复索引的小型 Series：

```python
In [258]: obj = pd.Series(np.arange(5), index=["a", "a", "b", "b", "c"])

In [259]: obj
Out[259]: 
a    0
a    1
b    2
b    3
c    4
dtype: int64
```

索引的 `is_unique` 属性可以告诉您其标签是否唯一：

```python
In [260]: obj.index.is_unique
Out[260]: False
```

数据选择是处理重复项时表现不同的主要方面之一。对具有多个条目的标签进行索引会返回一个 Series，而单个条目则返回一个标量值：

```python
In [261]: obj["a"]
Out[261]: 
a    0
a    1
dtype: int64

In [262]: obj["c"]
Out[262]: 4
```

这可能会使您的代码更加复杂，因为索引的输出类型可能会根据标签是否重复而变化。

相同的逻辑扩展到在 DataFrame 中索引行（或列）：

```python
In [263]: df = pd.DataFrame(np.random.standard_normal((5, 3)),
   .....:                   index=["a", "a", "b", "b", "c"])

In [264]: df
Out[264]: 
          0         1         2
a  0.274992  0.228913  1.352917
a  0.886429 -2.001637 -0.371843
b  1.669025 -0.438570 -0.539741
b  0.476985  3.248944 -1.021228
c -0.577087  0.124121  0.302614

In [265]: df.loc["b"]
Out[265]: 
          0         1         2
b  1.669025 -0.438570 -0.539741
b  0.476985  3.248944 -1.021228

In [266]: df.loc["c"]
Out[266]: 
0   -0.577087
1    0.124121
2    0.302614
Name: c, dtype: float64
```

## 5.3 汇总与计算描述性统计

pandas 对象配备了一组常用的数学和统计方法。其中大多数属于*归约*（reductions）或*汇总统计*（summary statistics）类别，这些方法从 Series 中提取单个值（如总和或均值），或从 DataFrame 的行或列中提取一系列值。与 NumPy 数组中的类似方法相比，它们内置了对缺失数据的处理功能。考虑以下小型 DataFrame：

```python
In [267]: df = pd.DataFrame([[1.4, np.nan], [7.1, -4.5],
   .....:                    [np.nan, np.nan], [0.75, -1.3]],
   .....:                   index=["a", "b", "c", "d"],
   .....:                   columns=["one", "two"])

In [268]: df
Out[268]: 
    one  two
a  1.40  NaN
b  7.10 -4.5
c   NaN  NaN
d  0.75 -1.3
```

调用 DataFrame 的 `sum` 方法会返回一个包含列总和的 Series：

```python
In [269]: df.sum()
Out[269]: 
one    9.25
two   -5.80
dtype: float64
```

传入 `axis="columns"` 或 `axis=1` 则会按行进行求和：

```python
In [270]: df.sum(axis="columns")
Out[270]: 
a    1.40
b    2.60
c    0.00
d   -0.55
dtype: float64
```

当整行或整列均为 NA 值时，求和结果为 0；而如果存在任何非 NA 值，则结果为 NA。可以通过 `skipna` 选项禁用此行为，此时行或列中的任何 NA 值都会使对应结果变为 NA：

```python
In [271]: df.sum(axis="index", skipna=False)
Out[271]: 
one   NaN
two   NaN
dtype: float64

In [272]: df.sum(axis="columns", skipna=False)
Out[272]: 
a     NaN
b    2.60
c     NaN
d   -0.55
dtype: float64
```

某些聚合操作（如 `mean`）要求至少有一个非 NA 值才能产生有效结果，因此这里：

```python
In [273]: df.mean(axis="columns")
Out[273]: 
a    1.400
b    1.300
c      NaN
d   -0.275
dtype: float64
```

常见归约方法的选项列表请参见表 5.7。

**表 5.7：归约方法的选项**

| 方法      | 描述                                                                 |
|-----------|----------------------------------------------------------------------|
| `axis`    | 归约操作的轴；DataFrame 行用 "index"，列用 "columns"                   |
| `skipna`  | 排除缺失值；默认为 `True`                                            |
| `level`   | 如果轴是分层索引（MultiIndex），则按层级分组归约                         |

某些方法（如 `idxmin` 和 `idxmax`）返回间接统计信息，例如达到最小值或最大值的索引值：

```python
In [274]: df.idxmax()
Out[274]: 
one    b
two    d
dtype: object
```

其他方法属于*累积*操作：

```python
In [275]: df.cumsum()
Out[275]: 
    one  two
a  1.40  NaN
b  8.50 -4.5
c   NaN  NaN
d  9.25 -5.8
```

还有一些方法既不是归约也不是累积操作。`describe` 就是其中之一，它可以一次性生成多个汇总统计量：

```python
In [276]: df.describe()
Out[276]: 
            one       two
count  3.000000  2.000000
mean   3.083333 -2.900000
std    3.493685  2.262742
min    0.750000 -4.500000
25%    1.075000 -3.700000
50%    1.400000 -2.900000
75%    4.250000 -2.100000
max    7.100000 -1.300000
```

对于非数值数据，`describe` 会生成替代的汇总统计量：

```python
In [277]: obj = pd.Series(["a", "a", "b", "c"] * 4)

In [278]: obj.describe()
Out[278]: 
count     16
unique     3
top        a
freq       8
dtype: object
```

完整的汇总统计及相关方法列表请参见表 5.8。

**表 5.8：描述性与汇总统计方法**

| 方法            | 描述                                                                 |
|-----------------|----------------------------------------------------------------------|
| `count`         | 非 NA 值的数量                                                       |
| `describe`      | 计算一组汇总统计量                                                    |
| `min, max`      | 计算最小值和最大值                                                    |
| `argmin, argmax`| 分别计算最小或最大值所在的索引位置（整数）；不适用于 DataFrame 对象      |
| `idxmin, idxmax`| 分别计算最小或最大值所在的索引标签                                    |
| `quantile`      | 计算从 0 到 1 的样本分位数（默认：0.5）                               |
| `sum`           | 值的总和                                                             |
| `mean`          | 值的平均值                                                           |
| `median`        | 值的算术中位数（50% 分位数）                                          |
| `mad`           | 值的平均绝对偏差                                                      |
| `prod`          | 所有值的乘积                                                         |
| `var`           | 值的样本方差                                                         |
| `std`           | 值的样本标准差                                                       |
| `skew`          | 值的样本偏度（三阶矩）                                                |
| `kurt`          | 值的样本峰度（四阶矩）                                                |
| `cumsum`        | 值的累计和                                                           |
| `cummin, cummax`| 分别计算值的累计最小值或最大值                                         |
| `cumprod`       | 值的累计乘积                                                         |
| `diff`          | 计算一阶算术差分（对时间序列有用）                                     |
| `pct_change`    | 计算百分比变化                                                        |

### 相关性（Correlation）与协方差（Covariance）

一些汇总统计量（如相关性和协方差）是通过成对参数计算得出的。考虑一些股票价格和成交量的 DataFrame，这些数据最初从 Yahoo! Finance 获取，并保存在二进制 Python pickle 文件中，你可以在本书的附带数据集中找到：

```python
In [279]: price = pd.read_pickle("examples/yahoo_price.pkl")

In [280]: volume = pd.read_pickle("examples/yahoo_volume.pkl")
```

现在计算价格的百分比变化，这是一个时间序列操作，将在[第 11 章：时间序列](https://wesmckinney.com/book/time-series)中进一步探讨：

```python
In [281]: returns = price.pct_change()

In [282]: returns.tail()
Out[282]: 
                AAPL      GOOG       IBM      MSFT
Date                                              
2016-10-17 -0.000680  0.001837  0.002072 -0.003483
2016-10-18 -0.000681  0.019616 -0.026168  0.007690
2016-10-19 -0.002979  0.007846  0.003583 -0.002255
2016-10-20 -0.000512 -0.005652  0.001719 -0.004867
2016-10-21 -0.003930  0.003011 -0.012474  0.042096
```

Series 的 `corr` 方法计算两个 Series 中重叠、非 NA 且按索引对齐的值的相关性。类似地，`cov` 计算协方差：

```python
In [283]: returns["MSFT"].corr(returns["IBM"])
Out[283]: 0.49976361144151166

In [284]: returns["MSFT"].cov(returns["IBM"])
Out[284]: 8.870655479703549e-05
```

另一方面，DataFrame 的 `corr` 和 `cov` 方法分别返回完整的相关性或协方差矩阵作为 DataFrame：

```python
In [285]: returns.corr()
Out[285]: 
          AAPL      GOOG       IBM      MSFT
AAPL  1.000000  0.407919  0.386817  0.389695
GOOG  0.407919  1.000000  0.405099  0.465919
IBM   0.386817  0.405099  1.000000  0.499764
MSFT  0.389695  0.465919  0.499764  1.000000

In [286]: returns.cov()
Out[286]: 
          AAPL      GOOG       IBM      MSFT
AAPL  0.000277  0.000107  0.000078  0.000095
GOOG  0.000107  0.000251  0.000078  0.000108
IBM   0.000078  0.000078  0.000146  0.000089
MSFT  0.000095  0.000108  0.000089  0.000215
```

使用 DataFrame 的 `corrwith` 方法，可以计算 DataFrame 的列或行与另一个 Series 或 DataFrame 之间的成对相关性。传入 Series 会返回一个 Series，其中包含每列计算出的相关性值：

```python
In [287]: returns.corrwith(returns["IBM"])
Out[287]: 
AAPL    0.386817
GOOG    0.405099
IBM     1.000000
MSFT    0.499764
dtype: float64
```

传入 DataFrame 则会计算匹配列名的相关性。此处计算百分比变化与成交量的相关性：

```python
In [288]: returns.corrwith(volume)
Out[288]: 
AAPL   -0.075565
GOOG   -0.007067
IBM    -0.204849
MSFT   -0.092950
dtype: float64
```

传入 `axis="columns"` 会改为逐行操作。在所有情况下，数据点在计算相关性之前会按标签对齐。

### 唯一值、值计数与成员资格

另一类相关方法用于提取一维 Series 中包含的值的信息。以下例说明：

```python
In [289]: obj = pd.Series(["c", "a", "d", "a", "a", "b", "b", "c", "c"])
```

第一个函数是 `unique`，它返回 Series 中唯一值的数组：

```python
In [290]: uniques = obj.unique()

In [291]: uniques
Out[291]: array(['c', 'a', 'd', 'b'], dtype=object)
```

唯一值不一定按首次出现的顺序返回，也不按排序顺序，但如果需要可以事后排序（`uniques.sort()`）。相关地，`value_counts` 计算一个包含值频次的 Series：

```python
In [292]: obj.value_counts()
Out[292]: 
c    3
a    3
b    2
d    1
Name: count, dtype: int64
```

该 Series 按值降序排序以便使用。`value_counts` 也可作为顶级 pandas 方法使用，适用于 NumPy 数组或其他 Python 序列：

```python
In [293]: pd.value_counts(obj.to_numpy(), sort=False)
Out[293]: 
c    3
a    3
d    1
b    2
Name: count, dtype: int64
```

`isin` 执行向量化的集合成员资格检查，可用于将数据集过滤为 Series 或 DataFrame 列中的值子集：

```python
In [294]: obj
Out[294]: 
0    c
1    a
2    d
3    a
4    a
5    b
6    b
7    c
8    c
dtype: object

In [295]: mask = obj.isin(["b", "c"])

In [296]: mask
Out[296]: 
0     True
1    False
2    False
3    False
4    False
5     True
6     True
7     True
8     True
dtype: bool

In [297]: obj[mask]
Out[297]: 
0    c
5    b
6    b
7    c
8    c
dtype: object
```

与 `isin` 相关的是 `Index.get_indexer` 方法，它可以从一个可能包含重复值的数组到另一个唯一值数组生成索引数组：

```python
In [298]: to_match = pd.Series(["c", "a", "b", "b", "c", "a"])

In [299]: unique_vals = pd.Series(["c", "b", "a"])

In [300]: indices = pd.Index(unique_vals).get_indexer(to_match)

In [301]: indices
Out[301]: array([0, 2, 1, 1, 0, 2])
```

这些方法的参考信息请参见表 5.9。

**表 5.9：唯一值、值计数与集合成员方法**

| 方法            | 描述                                                                 |
|-----------------|----------------------------------------------------------------------|
| `isin`          | 计算布尔数组，指示每个 Series 或 DataFrame 值是否包含在传入的值序列中     |
| `get_indexer`   | 计算数组中每个值到另一个唯一值数组的整数索引；有助于数据对齐和连接类操作   |
| `unique`        | 计算 Series 中的唯一值数组，按观察顺序返回                              |
| `value_counts`  | 返回以唯一值为索引、频次为值的 Series，按计数降序排列                    |

在某些情况下，可能需要计算 DataFrame 中多个相关列的直方图。例如：

```python
In [302]: data = pd.DataFrame({"Qu1": [1, 3, 4, 3, 4],
   .....:                      "Qu2": [2, 3, 1, 2, 3],
   .....:                      "Qu3": [1, 5, 2, 4, 4]})

In [303]: data
Out[303]: 
   Qu1  Qu2  Qu3
0    1    2    1
1    3    3    5
2    4    1    2
3    3    2    4
4    4    3    4
```

可以计算单列的值计数，如下所示：

```python
In [304]: data["Qu1"].value_counts().sort_index()
Out[304]: 
Qu1
1    1
3    2
4    2
Name: count, dtype: int64
```

要为所有列计算此值，可将 `pandas.value_counts` 传递给 DataFrame 的 `apply` 方法：

```python
In [305]: result = data.apply(pd.value_counts).fillna(0)

In [306]: result
Out[306]: 
   Qu1  Qu2  Qu3
1  1.0  1.0  1.0
2  0.0  2.0  1.0
3  2.0  2.0  0.0
4  2.0  0.0  2.0
5  0.0  0.0  1.0
```

此处，结果中的行标签是所有列中出现的不同值。值是这些值在每列中的相应计数。

还有一个 `DataFrame.value_counts` 方法，但它将 DataFrame 的每一行视为一个元组来计算每个不同行的出现次数：

```python
In [307]: data = pd.DataFrame({"a": [1, 1, 1, 2, 2], "b": [0, 0, 1, 0, 0]})

In [308]: data
Out[308]: 
   a  b
0  1  0
1  1  0
2  1  1
3  2  0
4  2  0

In [309]: data.value_counts()
Out[309]: 
a  b
1  0    2
2  0    2
1  1    1
Name: count, dtype: int64
```

在这种情况下，结果有一个索引，将不同行表示为分层索引，我们将在[第 8 章：数据整理：连接、合并与重塑](https://wesmckinney.com/book/data-wrangling)中详细探讨这一主题。

## 5.4 结论

在下一章中，我们将讨论使用 pandas 读取（或_加载_）和写入数据集的工具。之后，我们将深入探讨使用 pandas 进行数据清洗、整理、分析和可视化的工具。