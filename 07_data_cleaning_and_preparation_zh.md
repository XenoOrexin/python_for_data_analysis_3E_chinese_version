# 7数据清洗与准备（Data Cleaning and Preparation）

__

《Python for Data Analysis 3rd Edition》（Python 数据分析 第三版）的开放获取网络版现已上线，作为[印刷版与数字版](https://amzn.to/3DyLaJc)的配套资源。如果您发现任何错误，[请在此处提交勘误](https://oreilly.com/catalog/0636920519829/errata)。请注意，由 Quarto 生成的本站点某些格式会与 O’Reilly 出版的印刷版及电子书版本有所不同。

如果您觉得本书在线版对您有帮助，请考虑[订购纸质版本](https://amzn.to/3DyLaJc)或[无DRM限制的电子书](https://www.ebooks.com/en-us/book/210644288/python-for-data-analysis/wes-mckinney/?affId=WES398681F)以支持作者。本站内容不可复制或转载。代码示例采用 MIT 许可证，可在 GitHub 或 Gitee 上获取。

在进行数据分析和建模的过程中，数据准备会花费大量时间：包括加载、清洗、转换和重新排列。这类任务通常占分析师 80% 甚至更多的时间。有时文件或数据库中的数据存储格式并不适用于特定任务。许多研究者选择使用通用编程语言（如 Python、Perl、R 或 Java）或 Unix 文本处理工具（如 sed 或 awk）对数据进行临时处理（ad hoc processing），将数据从一种形式转换为另一种形式。幸运的是，pandas 库与 Python 内置语言特性相结合，提供了一套高阶、灵活且高效的工具，帮助您将数据 Manipulate（处理）成所需的格式。

如果您发现某种数据处理方式在本书或 pandas 库的其他地方都未涉及，欢迎在 Python 邮件列表或 pandas 的 GitHub 站点上分享您的用例。实际上，pandas 的设计和实现很大程度上是由实际应用需求驱动的。

本章将讨论处理缺失数据、重复数据、字符串操作以及其他一些分析性数据转换的工具。下一章将重点介绍以不同方式组合和重新排列数据集。

## 7.1 缺失值处理 (Handling Missing Data)

缺失值在许多数据分析应用中十分常见。pandas 的设计目标之一就是让缺失值处理尽可能轻松。例如，pandas 对象的所有描述性统计默认都会排除缺失数据。

pandas 对象中表示缺失数据的方式虽然并不完美，但足以满足大多数实际应用需求。对于 `float64` 数据类型的数值，pandas 使用浮点数值 `NaN`（Not a Number，非数值）来表示缺失数据。

我们将其称为 _哨兵值_ (sentinel value)：当它出现时，就表示存在缺失（或称为 _空值_，null）：
    
    
    In [14]: float_data = pd.Series([1.2, -3.5, np.nan, 0])
    
    In [15]: float_data
    Out[15]: 
    0    1.2
    1   -3.5
    2    NaN
    3    0.0
    dtype: float64 __

`isna` 方法会返回一个布尔 Series，其中 `True` 表示该位置的值为空：
    
    
    In [16]: float_data.isna()
    Out[16]: 
    0    False
    1    False
    2     True
    3    False
    dtype: bool __

在 pandas 中，我们借鉴了 R 语言的惯例，使用 NA（not available 的缩写）来指代缺失数据。在统计学应用中，NA 数据可能表示不存在的数据，也可能表示存在但未被观测到的数据（例如因数据收集问题导致）。在清理数据进行分析时，对缺失数据本身进行分析通常很重要，这有助于识别数据收集问题或由缺失数据导致的潜在数据偏差。

Python 内置的 `None` 值也会被视为 NA：
    
    
    In [17]: string_data = pd.Series(["aardvark", np.nan, None, "avocado"])
    
    In [18]: string_data
    Out[18]: 
    0    aardvark
    1         NaN
    2        None
    3     avocado
    dtype: object
    
    In [19]: string_data.isna()
    Out[19]: 
    0    False
    1     True
    2     True
    3    False
    dtype: bool
    
    In [20]: float_data = pd.Series([1, 2, None], dtype='float64')
    
    In [21]: float_data
    Out[21]: 
    0    1.0
    1    2.0
    2    NaN
    dtype: float64
    
    In [22]: float_data.isna()
    Out[22]: 
    0    False
    1    False
    2     True
    dtype: bool __

pandas 项目致力于实现跨数据类型的统一缺失值处理方式。像 `pandas.isna` 这样的函数抽象掉了许多繁琐的细节。表 7.1 列出了一些与缺失值处理相关的函数。

**表 7.1：NA 处理对象方法**

方法 | 描述  
---|---  
`dropna` | 根据每个标签对应的值是否存在缺失数据来筛选轴标签，可通过不同阈值控制对缺失数据的容忍度。  
`fillna` | 使用某个值或插值方法（如 `"ffill"` 或 `"bfill"`）填充缺失数据。  
`isna` | 返回布尔值，指示哪些值是缺失值/NA。  
`notna` | `isna` 的否定形式，对非 NA 值返回 `True`，对 NA 值返回 `False`。  
  
### 过滤缺失值 (Filtering Out Missing Data)

有多种方法可以过滤缺失值。虽然你始终可以使用 `pandas.isna` 和布尔索引手动操作，但 `dropna` 会更方便。在 Series 上调用时，它会返回仅包含非空数据及其索引值的 Series：
    
    
    In [23]: data = pd.Series([1, np.nan, 3.5, np.nan, 7])
    
    In [24]: data.dropna()
    Out[24]: 
    0    1.0
    2    3.5
    4    7.0
    dtype: float64 __

这与以下操作结果相同：
    
    
    In [25]: data[data.notna()]
    Out[25]: 
    0    1.0
    2    3.5
    4    7.0
    dtype: float64 __

对于 DataFrame 对象，有不同的缺失值移除方式。你可能希望删除全部为 NA 的行或列，或者仅删除包含任何 NA 的行或列。`dropna` 默认会删除包含任何缺失值的行：
    
    
    In [26]: data = pd.DataFrame([[1., 6.5, 3.], [1., np.nan, np.nan],
       ....:                      [np.nan, np.nan, np.nan], [np.nan, 6.5, 3.]])
    
    In [27]: data
    Out[27]: 
         0    1    2
    0  1.0  6.5  3.0
    1  1.0  NaN  NaN
    2  NaN  NaN  NaN
    3  NaN  6.5  3.0
    
    In [28]: data.dropna()
    Out[28]: 
         0    1    2
    0  1.0  6.5  3.0 __

传入 `how="all"` 将仅删除全部为 NA 的行：
    
    
    In [29]: data.dropna(how="all")
    Out [29]: 
         0    1    2
    0  1.0  6.5  3.0
    1  1.0  NaN  NaN
    3  NaN  6.5  3.0 __

请注意，这些函数默认返回新对象，不会修改原始对象的内容。

要以相同方式删除列，需传入 `axis="columns"`：
    
    
    In [30]: data[4] = np.nan
    
    In [31]: data
    Out[31]: 
         0    1    2   4
    0  1.0  6.5  3.0 NaN
    1  1.0  NaN  NaN NaN
    2  NaN  NaN  NaN NaN
    3  NaN  6.5  3.0 NaN
    
    In [32]: data.dropna(axis="columns", how="all")
    Out[32]: 
         0    1    2
    0  1.0  6.5  3.0
    1  1.0  NaN  NaN
    2  NaN  NaN  NaN
    3  NaN  6.5  3.0 __

如果你希望仅保留包含最多指定数量缺失观测值的行，可以通过 `thresh` 参数实现：
    
    
    In [33]: df = pd.DataFrame(np.random.standard_normal((7, 3)))
    
    In [34]: df.iloc[:4, 1] = np.nan
    
    In [35]: df.iloc[:2, 2] = np.nan
    
    In [36]: df
    Out[36]: 
              0         1         2
    0 -0.204708       NaN       NaN
    1 -0.555730       NaN       NaN
    2  0.092908       NaN  0.769023
    3  1.246435       NaN -1.296221
    4  0.274992  0.228913  1.352917
    5  0.886429 -2.001637 -0.371843
    6  1.669025 -0.438570 -0.539741
    
    In [37]: df.dropna()
    Out[37]: 
              0         1         2
    4  0.274992  0.228913  1.352917
    5  0.886429 -2.001637 -0.371843
    6  1.669025 -0.438570 -0.539741
    
    In [38]: df.dropna(thresh=2)
    Out[38]: 
              0         1         2
    2  0.092908       NaN  0.769023
    3  1.246435       NaN -1.296221
    4  0.274992  0.228913  1.352917
    5  0.886429 -2.001637 -0.371843
    6  1.669025 -0.438570 -0.539741 __

### 填充缺失值 (Filling In Missing Data)

相较于直接过滤掉缺失值（可能会同时丢弃其他数据），你可能更希望通过多种方式填补这些“空洞”。在大多数情况下，`fillna` 是主要的处理工具。使用常量调用 `fillna` 会用该值替换所有缺失值：
    
    
    In [39]: df.fillna(0)
    Out[39]: 
              0         1         2
    0 -0.204708  0.000000  0.000000
    1 -0.555730  0.000000  0.000000
    2  0.092908  0.000000  0.769023
    3  1.246435  0.000000 -1.296221
    4  0.274992  0.228913  1.352917
    5  0.886429 -2.001637 -0.371843
    6  1.669025 -0.438570 -0.539741 __

如果传入字典，可以为每列指定不同的填充值：
    
    
    In [40]: df.fillna({1: 0.5, 2: 0})
    Out[40]: 
              0         1         2
    0 -0.204708  0.500000  0.000000
    1 -0.555730  0.500000  0.000000
    2  0.092908  0.500000  0.769023
    3  1.246435  0.500000 -1.296221
    4  0.274992  0.228913  1.352917
    5  0.886429 -2.001637 -0.371843
    6  1.669025 -0.438570 -0.539741 __

重索引时可用的插值方法（参见[表 5.3](https://wesmckinney.com/book/pandas-basics#tbl-table_reindex_function)）也可用于 `fillna`：
    
    
    In [41]: df = pd.DataFrame(np.random.standard_normal((6, 3)))
    
    In [42]: df.iloc[2:, 1] = np.nan
    
    In [43]: df.iloc[4:, 2] = np.nan
    
    In [44]: df
    Out[44]: 
              0         1         2
    0  0.476985  3.248944 -1.021228
    1 -0.577087  0.124121  0.302614
    2  0.523772       NaN  1.343810
    3 -0.713544       NaN -2.370232
    4 -1.860761       NaN       NaN
    5 -1.265934       NaN       NaN
    
    In [45]: df.fillna(method="ffill")
    Out[45]: 
              0         1         2
    0  0.476985  3.248944 -1.021228
    1 -0.577087  0.124121  0.302614
    2  0.523772  0.124121  1.343810
    3 -0.713544  0.124121 -2.370232
    4 -1.860761  0.124121 -2.370232
    5 -1.265934  0.124121 -2.370232
    
    In [46]: df.fillna(method="ffill", limit=2)
    Out[46]: 
              0         1         2
    0  0.476985  3.248944 -1.021228
    1 -0.577087  0.124121  0.302614
    2  0.523772  0.124121  1.343810
    3 -0.713544  0.124121 -2.370232
    4 -1.860761       NaN -2.370232
    5 -1.265934       NaN -2.370232 __

使用 `fillna` 还可以实现许多其他功能，例如使用中位数或均值统计量进行简单的数据插补：
    
    
    In [47]: data = pd.Series([1., np.nan, 3.5, np.nan, 7])
    
    In [48]: data.fillna(data.mean())
    Out[48]: 
    0    1.000000
    1    3.833333
    2    3.500000
    3    3.833333
    4    7.000000
    dtype: float64 __

表 7.2 列出了 `fillna` 的函数参数参考。

**表 7.2：`fillna` 函数参数**

参数 | 描述  
---|---  
`value` | 用于填充缺失值的标量值或类字典对象  
`method` | 插值方法：可选 `"bfill"`（向后填充）或 `"ffill"`（向前填充）；默认为 `None`  
`axis` | 填充操作的轴（`"index"` 或 `"columns"`）；默认为 `axis="index"`  
`limit` | 对于向前和向后填充，指定连续填充的最大周期数

## 7.2 数据转换

本章至此主要讨论了缺失值处理。过滤、清理和其他转换是另一类重要操作。

### 移除重复数据

DataFrame 中出现重复行可能由多种原因导致。以下是一个示例：

```python
In [49]: data = pd.DataFrame({"k1": ["one", "two"] * 3 + ["two"],
   ....:                      "k2": [1, 1, 2, 3, 3, 4, 4]})

In [50]: data
Out[50]: 
    k1  k2
0  one   1
1  two   1
2  one   2
3  two   3
4  one   3
5  two   4
6  two   4
```

DataFrame 的 `duplicated` 方法返回一个布尔型 Series，表示每行是否为重复行（即该行的列值是否与之前某行的值完全相同）：

```python
In [51]: data.duplicated()
Out[51]: 
0    False
1    False
2    False
3    False
4    False
5    False
6     True
dtype: bool
```

相应地，`drop_duplicates` 方法返回一个 DataFrame，其中过滤掉了 `duplicated` 数组为 `False` 的行：

```python
In [52]: data.drop_duplicates()
Out[52]: 
    k1  k2
0  one   1
1  two   1
2  one   2
3  two   3
4  one   3
5  two   4
```

默认情况下，这两种方法会考虑所有列； Alternatively，您可以指定任意列的子集来检测重复项。假设我们有一个附加的值列，并希望仅基于 `"k1"` 列过滤重复项：

```python
In [53]: data["v1"] = range(7)

In [54]: data
Out[54]: 
    k1  k2  v1
0  one   1   0
1  two   1   1
2  one   2   2
3  two   3   3
4  one   3   4
5  two   4   5
6  two   4   6

In [55]: data.drop_duplicates(subset=["k1"])
Out[55]: 
    k1  k2  v1
0  one   1   0
1  two   1   1
```

`duplicated` 和 `drop_duplicates` 默认保留首次出现的值组合。传入 `keep="last"` 将返回最后一次出现的组合：

```python
In [56]: data.drop_duplicates(["k1", "k2"], keep="last")
Out[56]: 
    k1  k2 v1
0  one   1  0
1  two   1  1
2  one   2  2
3  two   3  3
4  one   3  4
6  two   4  6
```

### 使用函数或映射转换数据

对于许多数据集，您可能希望基于数组、Series 或 DataFrame 列中的值执行某些转换。考虑以下关于各种肉类的假设数据：

```python
In [57]: data = pd.DataFrame({"food": ["bacon", "pulled pork", "bacon",
   ....:                               "pastrami", "corned beef", "bacon",
   ....:                               "pastrami", "honey ham", "nova lox"],
   ....:                      "ounces": [4, 3, 12, 6, 7.5, 8, 3, 5, 6]})

In [58]: data
Out[58]: 
          food  ounces
0        bacon     4.0
1  pulled pork     3.0
2        bacon    12.0
3     pastrami     6.0
4  corned beef     7.5
5        bacon     8.0
6     pastrami     3.0
7    honey ham     5.0
8     nova lox     6.0
```

假设您想添加一列，指示每种食物来源的动物类型。让我们写下每种肉类到动物种类的映射：

```python
meat_to_animal = {
  "bacon": "pig",
  "pulled pork": "pig",
  "pastrami": "cow",
  "corned beef": "cow",
  "honey ham": "pig",
  "nova lox": "salmon"
}
```

Series 的 `map` 方法（在 [5.2.5 节：函数应用与映射](https://wesmckinney.com/book/pandas-basics#pandas_apply) 中也讨论过）接受一个函数或类似字典的对象进行值转换：

```python
In [60]: data["animal"] = data["food"].map(meat_to_animal)

In [61]: data
Out[61]: 
          food  ounces  animal
0        bacon     4.0     pig
1  pulled pork     3.0     pig
2        bacon    12.0     pig
3     pastrami     6.0     cow
4  corned beef     7.5     cow
5        bacon     8.0     pig
6     pastrami     3.0     cow
7    honey ham     5.0     pig
8     nova lox     6.0  salmon
```

我们也可以传递一个完成所有工作的函数：

```python
In [62]: def get_animal(x):
   ....:     return meat_to_animal[x]

In [63]: data["food"].map(get_animal)
Out[63]: 
0       pig
1       pig
2       pig
3       cow
4       cow
5       pig
6       cow
7       pig
8    salmon
Name: food, dtype: object
```

使用 `map` 是执行逐元素转换和其他数据清理相关操作的便捷方式。

### 替换值

使用 `fillna` 方法填充缺失数据是更通用值替换的特殊情况。如您所见，`map` 可用于修改对象中的值子集，但 `replace` 提供了更简单灵活的方式。考虑以下 Series：

```python
In [64]: data = pd.Series([1., -999., 2., -999., -1000., 3.])

In [65]: data
Out[65]: 
0       1.0
1    -999.0
2       2.0
3    -999.0
4   -1000.0
5       3.0
dtype: float64
```

`-999` 值可能是缺失数据的标记值（sentinel values）。要将这些替换为 pandas 可理解的 NA 值，可以使用 `replace`，生成新 Series：

```python
In [66]: data.replace(-999, np.nan)
Out[66]: 
0       1.0
1       NaN
2       2.0
3       NaN
4   -1000.0
5       3.0
dtype: float64
```

如果想一次替换多个值，可以传递一个列表和替换值：

```python
In [67]: data.replace([-999, -1000], np.nan)
Out[67]: 
0    1.0
1    NaN
2    2.0
3    NaN
4    NaN
5    3.0
dtype: float64
```

要为每个值使用不同的替换，可以传递一个替换值列表：

```python
In [68]: data.replace([-999, -1000], [np.nan, 0])
Out[68]: 
0    1.0
1    NaN
2    2.0
3    NaN
4    0.0
5    3.0
dtype: float64
```

参数也可以是字典：

```python
In [69]: data.replace({-999: np.nan, -1000: 0})
Out[69]: 
0    1.0
1    NaN
2    2.0
3    NaN
4    0.0
5    3.0
dtype: float64
```

注意

`data.replace` 方法与 `data.str.replace` 不同，后者执行逐元素字符串替换。我们将在本章后面讨论 Series 的字符串方法。

### 重命名轴索引

与 Series 中的值类似，轴标签可以通过函数或某种形式的映射进行转换，以生成新的、不同标签的对象。您也可以就地修改轴而不创建新的数据结构。以下是一个简单示例：

```python
In [70]: data = pd.DataFrame(np.arange(12).reshape((3, 4)),
   ....:                     index=["Ohio", "Colorado", "New York"],
   ....:                     columns=["one", "two", "three", "four"])
```

与 Series 类似，轴索引有一个 `map` 方法：

```python
In [71]: def transform(x):
   ....:     return x[:4].upper()

In [72]: data.index.map(transform)
Out[72]: Index(['OHIO', 'COLO', 'NEW '], dtype='object')
```

您可以赋值给 `index` 属性，从而就地修改 DataFrame：

```python
In [73]: data.index = data.index.map(transform)

In [74]: data
Out[74]: 
      one  two  three  four
OHIO    0    1      2     3
COLO    4    5      6     7
NEW     8    9     10    11
```

如果想创建数据集的转换版本而不修改原始数据，`rename` 方法很有用：

```python
In [75]: data.rename(index=str.title, columns=str.upper)
Out[75]: 
      ONE  TWO  THREE  FOUR
Ohio    0    1      2     3
Colo    4    5      6     7
New     8    9     10    11
```

值得注意的是，`rename` 可以与类似字典的对象结合使用，为轴标签的子集提供新值：

```python
In [76]: data.rename(index={"OHIO": "INDIANA"},
   ....:             columns={"three": "peekaboo"})
Out[76]: 
         one  two  peekaboo  four
INDIANA    0    1         2     3
COLO       4    5         6     7
NEW        8    9        10    11
```

`rename` 避免了手动复制 DataFrame 并为其 `index` 和 `columns` 属性分配新值的繁琐工作。

### 离散化与分箱

连续数据通常被离散化或分成“分箱”（bins）以进行分析。假设您有一个研究人群的年龄数据，并希望将他们分组到离散的年龄桶中：

```python
In [77]: ages = [20, 22, 25, 27, 21, 23, 37, 31, 61, 45, 41, 32]
```

让我们将其分为 18-25、26-35、36-60 和 61 岁及以上的分箱。为此，您必须使用 `pandas.cut`：

```python
In [78]: bins = [18, 25, 35, 60, 100]

In [79]: age_categories = pd.cut(ages, bins)

In [80]: age_categories
Out[80]: 
[(18, 25], (18, 25], (18, 25], (25, 35], (18, 25], ..., (25, 35], (60, 100], (35, 60], (35, 60], (25, 35]]
Length: 12
Categories (4, interval[int64, right]): [(18, 25] < (25, 35] < (35, 60] < (60, 100]]
```

pandas 返回的对象是一个特殊的 Categorical 对象。输出描述了由 `pandas.cut` 计算的分箱。每个分箱由一个特殊的（pandas 独有的）区间值类型标识，包含每个分箱的下限和上限：

```python
In [81]: age_categories.codes
Out[81]: array([0, 0, 0, 1, 0, 0, 2, 1, 3, 2, 2, 1], dtype=int8)

In [82]: age_categories.categories
Out[82]: IntervalIndex([(18, 25], (25, 35], (35, 60], (60, 100]], dtype='interval[int64, right]')

In [83]: age_categories.categories[0]
Out[83]: Interval(18, 25, closed='right')

In [84]: pd.value_counts(age_categories)
Out[84]: 
(18, 25]     5
(25, 35]     3
(35, 60]     3
(60, 100]    1
Name: count, dtype: int64
```

注意，`pd.value_counts(categories)` 是 `pandas.cut` 结果的分箱计数。

在区间的字符串表示中，圆括号表示该侧为_开区间_（不包含），而方括号表示_闭区间_（包含）。您可以通过传递 `right=False` 来更改闭合侧：

```python
In [85]: pd.cut(ages, bins, right=False)
Out[85]: 
[[18, 25), [18, 25), [25, 35), [25, 35), [18, 25), ..., [25, 35), [60, 100), [35, 60), [35, 60), [25, 35)]
Length: 12
Categories (4, interval[int64, left]): [[18, 25) < [25, 35) < [35, 60) < [60, 100)]
```

您可以通过向 `labels` 选项传递列表或数组来覆盖默认的基于区间的分箱标签：

```python
In [86]: group_names = ["Youth", "YoungAdult", "MiddleAged", "Senior"]

In [87]: pd.cut(ages, bins, labels=group_names)
Out[87]: 
['Youth', 'Youth', 'Youth', 'YoungAdult', 'Youth', ..., 'YoungAdult', 'Senior', 'MiddleAged', 'MiddleAged', 'YoungAdult']
Length: 12
Categories (4, object): ['Youth' < 'YoungAdult' < 'MiddleAged' < 'Senior']
```

如果向 `pandas.cut` 传递分箱数量的整数而不是显式的分箱边缘，它将根据数据的最小值和最大值计算等长分箱。考虑将一些均匀分布的数据切成四份的情况：

```python
In [88]: data = np.random.uniform(size=20)

In [89]: pd.cut(data, 4, precision=2)
Out[89]: 
[(0.34, 0.55], (0.34, 0.55], (0.76, 0.97], (0.76, 0.97], (0.34, 0.55], ..., (0.34, 0.55], (0.34, 0.55], (0.55, 0.76], (0.34, 0.55], (0.12, 0.34]]
Length: 20
Categories (4, interval[float64, right]): [(0.12, 0.34] < (0.34, 0.55] < (0.55, 0.76] < (0.76, 0.97]]
```

`precision=2` 选项将小数精度限制为两位。

一个密切相关的函数 `pandas.qcut` 基于样本分位数进行分箱。根据数据分布，使用 `pandas.cut` 通常不会使每个分箱具有相同数量的数据点。由于 `pandas.qcut` 使用样本分位数，您将获得大致相等大小的分箱：

```python
In [90]: data = np.random.standard_normal(1000)

In [91]: quartiles = pd.qcut(data, 4, precision=2)

In [92]: quartiles
Out[92]: 
[(-0.026, 0.62], (0.62, 3.93], (-0.68, -0.026], (0.62, 3.93], (-0.026, 0.62], ..., (-0.68, -0.026], (-0.68, -0.026], (-2.96, -0.68], (0.62, 3.93], (-0.68, -0.026]]
Length: 1000
Categories (4, interval[float64, right]): [(-2.96, -0.68] < (-0.68, -0.026] < (-0.026, 0.62] < (0.62, 3.93]]

In [93]: pd.value_counts(quartiles)
Out[93]: 
(-2.96, -0.68]     250
(-0.68, -0.026]    250
(-0.026, 0.62]     250
(0.62, 3.93]       250
Name: count, dtype: int64
```

与 `pandas.cut` 类似，您可以传递自己的分位数（0 到 1 之间的数字，包含）：

```python
In [94]: pd.qcut(data, [0, 0.1, 0.5, 0.9, 1.]).value_counts()
Out[94]: 
(-2.9499999999999997, -1.187]    100
(-1.187, -0.0265]                400
(-0.0265, 1.286]                 400
(1.286, 3.928]                   100
Name: count, dtype: int64
```

我们将在本章后面的聚合和分组操作讨论中再次介绍 `pandas.cut` 和 `pandas.qcut`，因为这些离散化函数对于分位数和分组分析特别有用。

### 检测和过滤异常值

过滤或转换异常值主要是应用数组操作的问题。考虑一个包含一些正态分布数据的 DataFrame：

```python
In [95]: data = pd.DataFrame(np.random.standard_normal((1000, 4)))

In [96]: data.describe()
Out[96]: 
                 0            1            2            3
count  1000.000000  1000.000000  1000.000000  1000.000000
mean      0.049091     0.026112    -0.002544    -0.051827
std       0.996947     1.007458     0.995232     0.998311
min      -3.645860    -3.184377    -3.745356    -3.428254
25%      -0.599807    -0.612162    -0.687373    -0.747478
50%       0.047101    -0.013609    -0.022158    -0.088274
75%       0.756646     0.695298     0.699046     0.623331
max       2.653656     3.525865     2.735527     3.366626
```

假设您想找到某一列中绝对值超过 3 的值：

```python
In [97]: col = data[2]

In [98]: col[col.abs() > 3]
Out[98]: 
41    -3.399312
136   -3.745356
Name: 2, dtype: float64
```

要选择所有具有超过 3 或 –3 值的行，可以在布尔型 DataFrame 上使用 `any` 方法：

```python
In [99]: data[(data.abs() > 3).any(axis="columns")]
Out[99]: 
            0         1         2         3
41   0.457246 -0.025907 -3.399312 -0.974657
60   1.951312  3.260383  0.963301  1.201206
136  0.508391 -0.196713 -3.745356 -1.520113
235 -0.242459 -3.056990  1.918403 -0.578828
258  0.682841  0.326045  0.425384 -3.428254
322  1.179227 -3.184377  1.369891 -1.074833
544 -3.548824  1.553205 -2.186301  1.277104
635 -0.578093  0.193299  1.397822  3.366626
782 -0.207434  3.525865  0.283070  0.544635
803 -3.645860  0.255475 -0.549574 -1.907459
```

`data.abs() > 3` 周围的括号是必要的，以便在比较操作的结果上调用 `any` 方法。

可以根据这些条件设置值。以下是限制区间 –3 到 3 之外值的代码：

```python
In [100]: data[data.abs() > 3] = np.sign(data) * 3

In [101]: data.describe()
Out[101]: 
                 0            1            2            3
count  1000.000000  1000.000000  1000.000000  1000.000000
mean      0.050286     0.025567    -0.001399    -0.051765
std       0.992920     1.004214     0.991414     0.995761
min      -3.000000    -3.000000    -3.000000    -3.000000
25%      -0.599807    -0.612162    -0.687373    -0.747478
50%       0.047101    -0.013609    -0.022158    -0.088274
75%       0.756646     0.695298     0.699046     0.623331
max       2.653656     3.000000     2.735527     3.000000
```

语句 `np.sign(data)` 根据 `data` 中的值是正还是负产生 1 和 –1 值：

```python
In [102]: np.sign(data).head()
Out[102]: 
     0    1    2    3
0 -1.0  1.0 -1.0  1.0
1  1.0 -1.0  1.0 -1.0
2  1.0  1.0  1.0 -1.0
3 -1.0 -1.0  1.0 -1.0
4 -1.0  1.0 -1.0 -1.0
```

### 排列与随机抽样

使用 `numpy.random.permutation` 函数可以排列（随机重新排序）Series 或 DataFrame 中的行。使用要排列的轴长度调用 `permutation` 会产生一个表示新顺序的整数数组：

```python
In [103]: df = pd.DataFrame(np.arange(5 * 7).reshape((5, 7)))

In [104]: df
Out[104]: 
    0   1   2   3   4   5   6
0   0   1   2   3   4   5   6
1   7   8   9  10  11  12  13
2  14  15  16  17  18  19  20
3  21  22  23  24  25  26  27
4  28  29  30  31  32  33  34

In [105]: sampler = np.random.permutation(5)

In [106]: sampler
Out[106]: array([3, 1, 4, 2, 0])
```

然后可以在基于 `iloc` 的索引或等效的 `take` 函数中使用该数组：

```python
In [107]: df.take(sampler)
Out[107]: 
    0   1   2   3   4   5   6
3  21  22  23  24  25  26  27
1  7   8   9  10  11  12  13
4  28  29  30  31  32  33  34
2  14  15  16  17  18  19  20
0   0   1   2   3   4   5   6

In [108]: df.iloc[sampler]
Out[108]: 
    0   1   2   3   4   5   6
3  21  22  23  24  25  26  27
1  7   8   9  10  11  12  13
4  28  29  30  31  32  33  34
2  14  15  16  17  18  19  20
0   0   1   2   3   4   5   6
```

通过使用 `axis="columns"` 调用 `take`，我们也可以选择列的排列：

```python
In [109]: column_sampler = np.random.permutation(7)

In [110]: column_sampler
Out[110]: array([4, 6, 3, 2, 1, 0, 5])

In [111]: df.take(column_sampler, axis="columns")
Out[111]: 
    4   6   3   2   1   0   5
0   4   6   3   2   1   0   5
1  11  13  10   9   8   7  12
2  18  20  17  16  15  14  19
3  25  27  24  23  22  21  26
4  32  34  31  30  29  28  33
```

要选择无放回的随机子集（同一行不能出现两次），可以在 Series 和 DataFrame 上使用 `sample` 方法：

```python
In [112]: df.sample(n=3)
Out[112]: 
    0   1   2   3   4   5   6
2  14  15  16  17  18  19  20
4  28  29  30  31  32  33  34
0   0   1   2   3   4   5   6
```

要生成有放回的样本（允许重复选择），向 `sample` 传递 `replace=True`：

```python
In [113]: choices = pd.Series([5, 7, -1, 6, 4])

In [114]: choices.sample(n=10, replace=True)
Out[114]: 
2   -1
0    5
3    6
1    7
4    4
0    5
4    4
0    5
4    4
4    4
dtype: int64
```

### 计算虚拟变量/指示变量

另一种用于统计建模或机器学习应用的转换是将分类变量转换为_虚拟_（dummy）或_指示_（indicator）矩阵。如果 DataFrame 中的一列有 `k` 个不同的值，您将派生一个包含 `k` 列全部为 1 和 0 的矩阵或 DataFrame。pandas 有一个 `pandas.get_dummies` 函数可以完成此操作，尽管您也可以自己设计一个。考虑以下示例 DataFrame：

```python
In [115]: df = pd.DataFrame({"key": ["b", "b", "a", "c", "a", "b"],
   .....:                    "data1": range(6)})

In [116]: df
Out[116]: 
  key  data1
0   b      0
1   b      1
2   a      2
3   c      3
4   a      4
5   b      5

In [117]: pd.get_dummies(df["key"], dtype=float)
Out[117]: 
     a    b    c
0  0.0  1.0  0.0
1  0.0  1.0  0.0
2  1.0  0.0  0.0
3  0.0  0.0  1.0
4  1.0  0.0  0.0
5  0.0  1.0  0.0
```

这里我传递了 `dtype=float` 以将输出类型从布尔值（pandas 较新版本的默认值）更改为浮点数。

在某些情况下，您可能希望在指示器 DataFrame 的列中添加前缀，然后可以与其他数据合并。`pandas.get_dummies` 有一个 prefix 参数用于此目的：

```python
In [118]: dummies = pd.get_dummies(df["key"], prefix="key", dtype=float)

In [119]: df_with_dummy = df[["data1"]].join(dummies)

In [120]: df_with_dummy
Out[120]: 
   data1  key_a  key_b  key_c
0      0    0.0    1.0    0.0
1      1    0.0    1.0    0.0
2      2    1.0    0.0    0.0
3      3    0.0    0.0    1.0
4      4    1.0    0.0    0.0
5      5    0.0    1.0    0.0
```

`DataFrame.join` 方法将在下一章详细解释。

如果 DataFrame 中的一行属于多个类别，我们必须使用不同的方法来创建虚拟变量。让我们看看 MovieLens 1M 数据集，该数据集在 Ch 13: 数据分析示例中有更详细的研究：

```python
In [121]: mnames = ["movie_id", "title", "genres"]

In [122]: movies = pd.read_table("datasets/movielens/movies.dat", sep="::",
   .....:                        header=None, names=mnames, engine="python")

In [123]: movies[:10]
Out[123]: 
   movie_id                               title                        genres
0         1                    Toy Story (1995)   Animation|Children's|Comedy
1         2                      Jumanji (1995)  Adventure|Children's|Fantasy
2         3             Grumpier Old Men (1995)                Comedy|Romance
3         4            Waiting to Exhale (1995)                  Comedy|Drama
4         5  Father of the Bride Part II (1995)                        Comedy
5         6                         Heat (1995)         Action|Crime|Thriller
6         7                      Sabrina (1995)                Comedy|Romance
7         8                 Tom and Huck (1995)          Adventure|Children's
8         9                 Sudden Death (1995)                        Action
9        10                    GoldenEye (1995)     Action|Adventure|Thriller
```

pandas 实现了一个特殊的 Series 方法 `str.get_dummies`（以 `str.` 开头的方法将在后面的字符串操作中详细讨论），该方法处理以分隔字符串编码的多组成员情况：

```python
In [124]: dummies = movies["genres"].str.get_dummies("|")

In [125]: dummies.iloc[:10, :6]
Out[125]: 
   Action  Adventure  Animation  Children's  Comedy  Crime
0       0          0          1           1       1      0
1       0          1          0           1       0      0
2       0          0          0           0       1      0
3       0          0          0           0       1      0
4       0          0          0           0       1      0
5       1          0          0           0       0      1
6       0          0          0           0       1      0
7       0          1          0           1       0      0
8       1          0          0           0       0      0
9       1          1          0           0       0      0
```

然后，如前所述，您可以将此与 `movies` 结合，同时使用 `add_prefix` 方法向 `dummies` DataFrame 的列名添加 `"Genre_"`：

```python
In [126]: movies_windic = movies.join(dummies.add_prefix("Genre_"))

In [127]: movies_windic.iloc[0]
Out[127]: 
movie_id                                       1
title                           Toy Story (1995)
genres               Animation|Children's|Comedy
Genre_Action                                   0
Genre_Adventure                                0
Genre_Animation                                1
Genre_Children's                               1
Genre_Comedy                                   1
Genre_Crime                                    0
Genre_Documentary                              0
Genre_Drama                                    0
Genre_Fantasy                                  0
Genre_Film-Noir                                0
Genre_Horror                                   0
Genre_Musical                                  0
Genre_Mystery                                  0
Genre_Romance                                  0
Genre_Sci-Fi                                   0
Genre_Thriller                                 0
Genre_War                                      0
Genre_Western                                  0
Name: 0, dtype: object
```

注意

对于更大的数据，这种构建多成员指示变量的方法不是特别快。最好编写一个直接写入 NumPy 数组的低级函数，然后将结果包装在 DataFrame 中。

统计应用中的一个有用配方是将 `pandas.get_dummies` 与像 `pandas.cut` 这样的离散化函数结合：

```python
In [128]: np.random.seed(12345) # 使示例可重复

In [129]: values = np.random.uniform(size=10)

In [130]: values
Out[130]: 
array([0.9296, 0.3164, 0.1839, 0.2046, 0.5677, 0.5955, 0.9645, 0.6532,
       0.7489, 0.6536])

In [131]: bins = [0, 0.2, 0.4, 0.6, 0.8, 1]

In [132]: pd.get_dummies(pd.cut(values, bins))
Out[132]: 
   (0.0, 0.2]  (0.2, 0.4]  (0.4, 0.6]  (0.6, 0.8]  (0.8, 1.0]
0       False       False       False       False        True
1       False        True       False       False       False
2        True       False       False       False       False
3       False        True       False       False       False
4       False       False        True       False       False
5       False       False        True       False       False
6       False       False       False       False        True
7       False       False       False        True       False
8       False       False       False        True       False
9       False       False       False        True       False
```

我们将在后面的创建建模用虚拟变量中再次讨论 `pandas.get_dummies`。

## 7.3 扩展数据类型（Extension Data Types）

__

注意 

这是一个较新且更高级的主题，许多 pandas 用户不需要深入了解，但为了内容的完整性，我在此进行介绍，因为在后续章节中我将在多处引用和使用扩展数据类型。

pandas 最初构建于 NumPy 的功能之上，NumPy 是一个主要用于处理数值数据的数组计算库。许多 pandas 的概念（例如缺失数据）都是利用 NumPy 中已有的功能实现的，同时试图最大化使用 NumPy 和 pandas 的库之间的兼容性。

基于 NumPy 构建导致了一些缺陷，例如：

*   对于某些数值数据类型（如整数和布尔值）的缺失数据处理不完整。因此，当此类数据中出现缺失数据时，pandas 会将数据类型转换为 `float64` 并使用 `np.nan` 表示空值。这给许多 pandas 算法带来了微妙的问题，产生了复合效应。
*   包含大量字符串数据的数据集在计算上成本高昂且占用大量内存。
*   某些数据类型（如时间间隔、时间差和带时区的时间戳）如果不使用计算成本高昂的 Python 对象数组，就无法高效支持。

最近，pandas 开发了一个_扩展类型_系统，允许添加新的数据类型，即使它们不受 NumPy 原生支持。这些新数据类型可以与来自 NumPy 数组的数据一起被视为一等公民。

让我们看一个示例，其中我们创建了一个包含缺失值的整数 Series：
    
    
    In [133]: s = pd.Series([1, 2, 3, None])
    
    In [134]: s
    Out[134]: 
    0    1.0
    1    2.0
    2    3.0
    3    NaN
    dtype: float64
    
    In [135]: s.dtype
    Out[135]: dtype('float64')__

主要是为了向后兼容，Series 使用了传统行为，即使用 `float64` 数据类型和 `np.nan` 表示缺失值。我们可以改用 `pandas.Int64Dtype` 创建这个 Series：
    
    
    In [136]: s = pd.Series([1, 2, 3, None], dtype=pd.Int64Dtype())
    
    In [137]: s
    Out[137]: 
    0       1
    1       2
    2       3
    3    <NA>
    dtype: Int64
    
    In [138]: s.isna()
    Out[138]: 
    0    False
    1    False
    2    False
    3     True
    dtype: bool
    
    In [139]: s.dtype
    Out[139]: Int64Dtype()__

输出中的 `<NA>` 表示扩展类型数组中的值缺失。这里使用了特殊的 `pandas.NA` 标记值：
    
    
    In [140]: s[3]
    Out[140]: <NA>
    
    In [141]: s[3] is pd.NA
    Out[141]: True __

我们也可以使用简写 `"Int64"` 而不是 `pd.Int64Dtype()` 来指定类型。首字母大写是必要的，否则它将是一个基于 NumPy 的非扩展类型：
    
    
    In [142]: s = pd.Series([1, 2, 3, None], dtype="Int64")__

pandas 还有一个专门用于字符串数据的扩展类型，它不使用 NumPy 对象数组（它需要 pyarrow 库，您可能需要单独安装）：
    
    
    In [143]: s = pd.Series(['one', 'two', None, 'three'], dtype=pd.StringDtype())
    
    In [144]: s
    Out[144]: 
    0      one
    1      two
    2     <NA>
    3    three
    dtype: string __

这些字符串数组通常使用更少的内存，并且在对大型数据集进行操作时，计算效率通常更高。

另一个重要的扩展类型是 `Categorical`，我们将在分类数据中更详细地讨论。截至撰写本文时，可用的扩展类型的相对完整列表见表 7.3。

扩展类型可以传递给 Series 的 `astype` 方法，使您可以轻松地将其作为数据清理过程的一部分进行转换：
    
    
    In [145]: df = pd.DataFrame({"A": [1, 2, None, 4],
       .....:                    "B": ["one", "two", "three", None],
       .....:                    "C": [False, None, False, True]})
    
    In [146]: df
    Out[146]: 
         A      B      C
    0  1.0    one  False
    1  2.0    two   None
    2  NaN  three  False
    3  4.0   None   True
    
    In [147]: df["A"] = df["A"].astype("Int64")
    
    In [148]: df["B"] = df["B"].astype("string")
    
    In [149]: df["C"] = df["C"].astype("boolean")
    
    In [150]: df
    Out[150]: 
          A      B      C
    0     1    one  False
    1     2    two   <NA>
    2  <NA>  three  False
    3     4   <NA>   True __

表 7.3：pandas 扩展数据类型

扩展类型 | 描述  
---|---  
`BooleanDtype` | 可空布尔数据，作为字符串传递时使用 `"boolean"`  
`CategoricalDtype` | 分类数据类型，作为字符串传递时使用 `"category"`  
`DatetimeTZDtype` | 带时区的日期时间  
`Float32Dtype` | 32 位可空浮点数，作为字符串传递时使用 `"Float32"`  
`Float64Dtype` | 64 位可空浮点数，作为字符串传递时使用 `"Float64"`  
`Int8Dtype` | 8 位可空有符号整数，作为字符串传递时使用 `"Int8"`  
`Int16Dtype` | 16 位可空有符号整数，作为字符串传递时使用 `"Int16"`  
`Int32Dtype` | 32 位可空有符号整数，作为字符串传递时使用 `"Int32"`  
`Int64Dtype` | 64 位可空有符号整数，作为字符串传递时使用 `"Int64"`  
`UInt8Dtype` | 8 位可空无符号整数，作为字符串传递时使用 `"UInt8"`  
`UInt16Dtype` | 16 位可空无符号整数，作为字符串传递时使用 `"UInt16"`  
`UInt32Dtype` | 32 位可空无符号整数，作为字符串传递时使用 `"UInt32"`  
`UInt64Dtype` | 64 位可空无符号整数，作为字符串传递时使用 `"UInt64"`

## 7.4 字符串操作 (String Manipulation)

Python 长期以来一直是一种流行的原始数据操作语言，部分原因在于其在字符串和文本处理方面的易用性。借助字符串对象的内置方法，大多数文本操作变得非常简单。对于更复杂的模式匹配和文本操作，则可能需要使用正则表达式。pandas 在此基础上更进一步，使您能够简洁地将字符串和正则表达式应用于整个数据数组，同时还处理了缺失数据带来的麻烦。

### Python 内置字符串对象方法

在许多字符串处理和脚本应用中，内置的字符串方法就足够了。例如，一个逗号分隔的字符串可以使用 `split` 方法拆分成若干部分：

```python
In [151]: val = "a,b,  guido"

In [152]: val.split(",")
Out[152]: ['a', 'b', '  guido']
```

`split` 方法通常与 `strip` 方法结合使用，以修剪空白符（包括换行符）：

```python
In [153]: pieces = [x.strip() for x in val.split(",")]

In [154]: pieces
Out[154]: ['a', 'b', 'guido']
```

这些子字符串可以使用加法操作符和双冒号分隔符连接起来：

```python
In [155]: first, second, third = pieces

In [156]: first + "::" + second + "::" + third
Out[156]: 'a::b::guido'
```

但这并非一个实用的通用方法。一种更快且更符合 Python 风格的方法是向字符串 `"::"` 的 `join` 方法传递一个列表或元组：

```python
In [157]: "::".join(pieces)
Out[157]: 'a::b::guido'
```

其他方法涉及定位子字符串。使用 Python 的 `in` 关键字是检测子字符串的最佳方式，不过 `index` 和 `find` 方法也可以使用：

```python
In [158]: "guido" in val
Out[158]: True

In [159]: val.index(",")
Out[159]: 1

In [160]: val.find(":")
Out[160]: -1
```

请注意，`find` 和 `index` 的区别在于，如果未找到字符串，`index` 会引发异常（而 `find` 返回 -1）：

```python
In [161]: val.index(":")
---------------------------------------------------------------------------
ValueError                                Traceback (most recent call last)
<ipython-input-161-bea4c4c30248> in <module>
----> 1 val.index(":")
ValueError: substring not found
```

相关地，`count` 方法返回特定子字符串出现的次数：

```python
In [162]: val.count(",")
Out[162]: 2
```

`replace` 方法会将一个模式的出现替换为另一个。通常也通过传递空字符串来删除模式：

```python
In [163]: val.replace(",", "::")
Out[163]: 'a::b::  guido'

In [164]: val.replace(",", "")
Out[164]: 'ab  guido'
```

表 7.4 列出了一些 Python 字符串方法。

正则表达式也可以用于许多此类操作，稍后您会看到。

**表 7.4：Python 内置字符串方法**

| 方法 | 描述 |
|------|------|
| `count` | 返回字符串中非重叠子字符串的出现次数 |
| `endswith` | 如果字符串以指定后缀结尾，则返回 `True` |
| `startswith` | 如果字符串以指定前缀开头，则返回 `True` |
| `join` | 使用字符串作为分隔符来连接其他字符串序列 |
| `index` | 如果在字符串中找到传入的子字符串，则返回其首次出现的起始索引；否则，如果未找到，则引发 `ValueError` |
| `find` | 返回字符串中子字符串 _首次_ 出现的第一个字符的位置；类似于 `index`，但未找到时返回 -1 |
| `rfind` | 返回字符串中子字符串 _最后一次_ 出现的第一个字符的位置；未找到时返回 -1 |
| `replace` | 将字符串的出现替换为另一个字符串 |
| `strip, rstrip, lstrip` | 分别修剪两侧、右侧或左侧的空白符（包括换行符） |
| `split` | 使用传入的分隔符将字符串拆分为子字符串列表 |
| `lower` | 将字母字符转换为小写 |
| `upper` | 将字母字符转换为大写 |
| `casefold` | 将字符转换为小写，并将任何区域特定的可变字符组合转换为通用的可比较形式 |
| `ljust, rjust` | 分别左对齐或右对齐；用空格（或其他填充字符）填充字符串的对侧，以返回具有最小宽度的字符串 |

### 正则表达式 (Regular Expressions)

_正则表达式_ 提供了一种灵活的方式来搜索或匹配文本中的（通常更复杂的）字符串模式。单个表达式（通常称为 _regex_）是根据正则表达式语言形成的字符串。Python 内置的 `re` 模块负责将正则表达式应用于字符串；我将在此给出一些使用示例。

注意

编写正则表达式的技巧本身可以单独成章，因此不在本书讨论范围之内。互联网和其他书籍中有许多优秀的教程和参考资料。

`re` 模块的函数分为三类：模式匹配、替换和分割。自然这些功能都是相关的；正则表达式描述了要在文本中定位的模式，然后可以用于多种目的。让我们看一个简单的例子：假设我们想用可变数量的空白字符（制表符、空格和换行符）分割一个字符串。

描述一个或多个空白字符的正则表达式是 `\s+`：

```python
In [165]: import re

In [166]: text = "foo    bar\t baz  \tqux"

In [167]: re.split(r"\s+", text)
Out[167]: ['foo', 'bar', 'baz', 'qux']
```

当您调用 `re.split(r"\s+", text)` 时，正则表达式首先被 _编译_，然后其 `split` 方法在传入的文本上被调用。您可以使用 `re.compile` 自己编译正则表达式，形成一个可重用的正则表达式对象：

```python
In [168]: regex = re.compile(r"\s+")

In [169]: regex.split(text)
Out[169]: ['foo', 'bar', 'baz', 'qux']
```

相反，如果您想要获取所有匹配正则表达式的模式列表，可以使用 `findall` 方法：

```python
In [170]: regex.findall(text)
Out[170]: ['    ', '\t ', '  \t']
```

注意

为了避免在正则表达式中因 `\` 产生不必要的转义，请使用 _原始_ 字符串字面量，如 `r"C:\x"`，而不是等价的 `"C:\\x"`。

如果您打算将同一个表达式应用于多个字符串，强烈建议使用 `re.compile` 创建正则表达式对象；这样做将节省 CPU 周期。

`match` 和 `search` 与 `findall` 密切相关。虽然 `findall` 返回字符串中的所有匹配项，但 `search` 仅返回第一个匹配项。更严格地说，`match` _仅_ 匹配字符串的开头。作为一个更复杂的例子，我们考虑一段文本和一个能够识别大多数电子邮件地址的正则表达式：

```python
text = """Dave dave@google.com
Steve steve@gmail.com
Rob rob@gmail.com
Ryan ryan@yahoo.com"""
pattern = r"[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,4}"

# re.IGNORECASE 使正则表达式不区分大小写
regex = re.compile(pattern, flags=re.IGNORECASE)
```

在文本上使用 `findall` 会生成一个电子邮件地址列表：

```python
In [172]: regex.findall(text)
Out[172]: 
['dave@google.com',
 'steve@gmail.com',
 'rob@gmail.com',
 'ryan@yahoo.com']
```

`search` 为文本中的第一个电子邮件地址返回一个特殊的匹配对象。对于前面的正则表达式，匹配对象只能告诉我们模式在字符串中的起始和结束位置：

```python
In [173]: m = regex.search(text)

In [174]: m
Out[174]: <re.Match object; span=(5, 20), match='dave@google.com'>

In [175]: text[m.start():m.end()]
Out[175]: 'dave@google.com'
```

`regex.match` 返回 `None`，因为它仅在模式出现在字符串开头时匹配：

```python
In [176]: print(regex.match(text))
None
```

相关地，`sub` 将返回一个新字符串，其中所有模式的出现都被新字符串替换：

```python
In [177]: print(regex.sub("REDACTED", text))
Dave REDACTED
Steve REDACTED
Rob REDACTED
Ryan REDACTED
```

假设您想要查找电子邮件地址，并同时将每个地址分割为其三个组成部分：用户名、域名和域后缀。为此，请将模式中要分割的部分用括号括起来：

```python
In [178]: pattern = r"([A-Z0-9._%+-]+)@([A-Z0-9.-]+)\.([A-Z]{2,4})"

In [179]: regex = re.compile(pattern, flags=re.IGNORECASE)
```

由此修改后的正则表达式产生的匹配对象通过其 `groups` 方法返回模式组件的元组：

```python
In [180]: m = regex.match("wesm@bright.net")

In [181]: m.groups()
Out[181]: ('wesm', 'bright', 'net')
```

当模式有分组时，`findall` 返回一个元组列表：

```python
In [182]: regex.findall(text)
Out[182]: 
[('dave', 'google', 'com'),
 ('steve', 'gmail', 'com'),
 ('rob', 'gmail', 'com'),
 ('ryan', 'yahoo', 'com')]
```

`sub` 还可以使用特殊符号（如 `\1` 和 `\2`）访问每个匹配中的分组。符号 `\1` 对应第一个匹配的组，`\2` 对应第二个，依此类推：

```python
In [183]: print(regex.sub(r"Username: \1, Domain: \2, Suffix: \3", text))
Dave Username: dave, Domain: google, Suffix: com
Steve Username: steve, Domain: gmail, Suffix: com
Rob Username: rob, Domain: gmail, Suffix: com
Ryan Username: ryan, Domain: yahoo, Suffix: com
```

Python 中的正则表达式还有更多内容，其中大部分超出了本书的范围。表 7.5 提供了一个简要总结。

**表 7.5：正则表达式方法**

| 方法 | 描述 |
|------|------|
| `findall` | 将字符串中所有非重叠的匹配模式作为列表返回 |
| `finditer` | 类似于 `findall`，但返回一个迭代器 |
| `match` | 在字符串开头匹配模式，并可选地将模式组件分割成组；如果模式匹配，则返回一个匹配对象，否则返回 `None` |
| `search` | 扫描字符串以查找匹配模式，如果找到则返回一个匹配对象；与 `match` 不同，匹配可以出现在字符串中的任何位置，而不仅限于开头 |
| `split` | 在每次出现模式的地方将字符串分割成片段 |
| `sub, subn` | 将字符串中所有（`sub`）或前 `n` 次（`subn`）出现的模式替换为替换表达式；在替换字符串中使用符号 `\1, \2, ...` 来引用匹配的分组元素 |

### pandas 中的字符串函数

为分析清理混乱的数据集通常需要大量的字符串操作。更复杂的是，包含字符串的列有时会有缺失数据：

```python
In [184]: data = {"Dave": "dave@google.com", "Steve": "steve@gmail.com",
       .....:         "Rob": "rob@gmail.com", "Wes": np.nan}

In [185]: data = pd.Series(data)

In [186]: data
Out[186]: 
Dave     dave@google.com
Steve    steve@gmail.com
Rob        rob@gmail.com
Wes                  NaN
dtype: object

In [187]: data.isna()
Out[187]: 
Dave     False
Steve    False
Rob      False
Wes       True
dtype: bool
```

可以使用 `data.map` 将字符串和正则表达式方法（传递 `lambda` 或其他函数）应用于每个值，但它会在 NA（空）值上失败。为了处理这个问题，Series 具有面向数组的字符串操作方法，这些方法会跳过并传播 NA 值。这些方法通过 Series 的 `str` 属性访问；例如，我们可以使用 `str.contains` 检查每个电子邮件地址是否包含 `"gmail"`：

```python
In [188]: data.str.contains("gmail")
Out[188]: 
Dave     False
Steve     True
Rob       True
Wes        NaN
dtype: object
```

请注意，此操作的结果具有 `object` 数据类型。pandas 具有 _扩展类型_，可以对字符串、整数和布尔数据进行特殊处理，这些类型在处理缺失数据时直到最近还存在一些粗糙的边缘：

```python
In [189]: data_as_string_ext = data.astype('string')

In [190]: data_as_string_ext
Out[190]: 
Dave     dave@google.com
Steve    steve@gmail.com
Rob        rob@gmail.com
Wes                 <NA>
dtype: string

In [191]: data_as_string_ext.str.contains("gmail")
Out[191]: 
Dave     False
Steve     True
Rob       True
Wes       <NA>
dtype: boolean
```

扩展类型在“扩展数据类型”一节中有更详细的讨论。

也可以使用正则表达式，以及任何 `re` 选项，如 `IGNORECASE`：

```python
In [192]: pattern = r"([A-Z0-9._%+-]+)@([A-Z0-9.-]+)\.([A-Z]{2,4})"

In [193]: data.str.findall(pattern, flags=re.IGNORECASE)
Out[193]: 
Dave     [(dave, google, com)]
Steve    [(steve, gmail, com)]
Rob        [(rob, gmail, com)]
Wes                        NaN
dtype: object
```

有几种方法可以进行向量化元素检索。可以使用 `str.get` 或索引到 `str` 属性：

```python
In [194]: matches = data.str.findall(pattern, flags=re.IGNORECASE).str[0]

In [195]: matches
Out[195]: 
Dave     (dave, google, com)
Steve    (steve, gmail, com)
Rob        (rob, gmail, com)
Wes                      NaN
dtype: object

In [196]: matches.str.get(1)
Out[196]: 
Dave     google
Steve     gmail
Rob       gmail
Wes         NaN
dtype: object
```

您可以使用类似的语法对字符串进行切片：

```python
In [197]: data.str[:5]
Out[197]: 
Dave     dave@
Steve    steve
Rob      rob@g
Wes        NaN
dtype: object
```

`str.extract` 方法会将正则表达式的捕获组作为 DataFrame 返回：

```python
In [198]: data.str.extract(pattern, flags=re.IGNORECASE)
Out[198]: 
               0       1    2
Dave    dave  google  com
Steve  steve   gmail  com
Rob      rob   gmail  com
Wes      NaN     NaN  NaN
```

更多 pandas 字符串方法请参见表 7.6。

**表 7.6：Series 字符串方法的部分列表**

| 方法 | 描述 |
|------|------|
| `cat` | 使用可选分隔符逐元素连接字符串 |
| `contains` | 如果每个字符串包含模式/正则表达式，则返回布尔数组 |
| `count` | 计算模式的出现次数 |
| `extract` | 使用带分组的正则表达式从字符串 Series 中提取一个或多个字符串；结果将是一个每列对应一个组的 DataFrame |
| `endswith` | 等价于每个元素的 `x.endswith(pattern)` |
| `startswith` | 等价于每个元素的 `x.startswith(pattern)` |
| `findall` | 计算每个字符串中所有模式/正则表达式出现的列表 |
| `get` | 索引到每个元素（检索第 _i_ 个元素） |
| `isalnum` | 等价于内置的 `str.alnum` |
| `isalpha` | 等价于内置的 `str.isalpha` |
| `isdecimal` | 等价于内置的 `str.isdecimal` |
| `isdigit` | 等价于内置的 `str.isdigit` |
| `islower` | 等价于内置的 `str.islower` |
| `isnumeric` | 等价于内置的 `str.isnumeric` |
| `isupper` | 等价于内置的 `str.isupper` |
| `join` | 使用传入的分隔符连接 Series 中每个元素的字符串 |
| `len` | 计算每个字符串的长度 |
| `lower, upper` | 转换大小写；等价于每个元素的 `x.lower()` 或 `x.upper()` |
| `match` | 在每个元素上使用 `re.match` 和传入的正则表达式，根据是否匹配返回 `True` 或 `False` |
| `pad` | 向字符串的左侧、右侧或两侧添加空白符 |
| `center` | 等价于 `pad(side="both")` |
| `repeat` | 重复值（例如，`s.str.repeat(3)` 等价于每个字符串的 `x * 3`） |
| `replace` | 将模式/正则表达式的出现替换为其他字符串 |
| `slice` | 对 Series 中的每个字符串进行切片 |
| `split` | 根据分隔符或正则表达式分割字符串 |
| `strip` | 修剪两侧的空白符（包括换行符） |
| `rstrip` | 修剪右侧的空白符 |
| `lstrip` | 修剪左侧的空白符 |

## 7.5 分类数据 (Categorical Data)

本节将介绍 pandas 的 `Categorical` 类型。我将展示如何通过使用它，在某些 pandas 操作中获得更好的性能和内存使用效率。我还将介绍一些工具，这些工具有助于在统计和机器学习应用中使用分类数据。

### 背景与动机

通常，表格中的某一列可能包含一个较小 distinct 值集合的重复实例。我们已经见过 `unique` 和 `value_counts` 这类函数，它们使我们能够分别从数组中提取 distinct 值并计算其频率：

```python
In [199]: values = pd.Series(['apple', 'orange', 'apple',
   .....:                     'apple'] * 2)

In [200]: values
Out[200]: 
0     apple
1    orange
2     apple
3     apple
4     apple
5    orange
6     apple
7     apple
dtype: object

In [201]: pd.unique(values)
Out[201]: array(['apple', 'orange'], dtype=object)

In [202]: pd.value_counts(values)
Out[202]: 
apple     6
orange    2
Name: count, dtype: int64
```

许多数据系统（用于数据仓库、统计计算或其他用途）已经开发出特殊的方法来高效地表示具有重复值的数据，以实现更高效的存储和计算。在数据仓库中，一个最佳实践是使用所谓的*维度表*（dimension tables），其中包含 distinct 值，并将主要观测值存储为引用维度表的整数键：

```python
In [203]: values = pd.Series([0, 1, 0, 0] * 2)

In [204]: dim = pd.Series(['apple', 'orange'])

In [205]: values
Out[205]: 
0    0
1    1
2    0
3    0
4    0
5    1
6    0
7    0
dtype: int64

In [206]: dim
Out[206]: 
0     apple
1    orange
dtype: object
```

我们可以使用 `take` 方法来恢复原始的字符串 Series：

```python
In [207]: dim.take(values)
Out[207]: 
0     apple
1    orange
0     apple
0     apple
0     apple
1    orange
0     apple
0     apple
dtype: object
```

这种用整数表示的方式称为*分类*（categorical）或*字典编码*（dictionary-encoded）表示法。distinct 值的数组可以称为数据的*类别*（categories）、*字典*（dictionary）或*层级*（levels）。在本书中，我们将使用术语*分类*（categorical）和*类别*（categories）。引用这些类别的整数值称为*类别编码*（category codes）或简称*编码*（codes）。

在进行数据分析时，分类表示法可以带来显著的性能提升。您还可以对类别进行转换，同时保持编码不变。一些可以以相对较低成本进行的转换示例包括：

*   重命名类别
*   在不更改现有类别的顺序或位置的情况下追加新类别

### pandas 中的分类扩展类型

pandas 有一个特殊的 `Categorical` 扩展类型，用于保存使用基于整数的分类表示法或*编码*（encoding）的数据。这是一种对于具有大量相似值出现的数据的流行数据压缩技术，并且可以提供显著更快的性能和更低的内存使用，特别是对于字符串数据。

让我们考虑之前的示例 Series：

```python
In [208]: fruits = ['apple', 'orange', 'apple', 'apple'] * 2

In [209]: N = len(fruits)

In [210]: rng = np.random.default_rng(seed=12345)

In [211]: df = pd.DataFrame({'fruit': fruits,
   .....:                    'basket_id': np.arange(N),
   .....:                    'count': rng.integers(3, 15, size=N),
   .....:                    'weight': rng.uniform(0, 4, size=N)},
   .....:                   columns=['basket_id', 'fruit', 'count', 'weight'])

In [212]: df
Out[212]: 
   basket_id   fruit  count    weight
0          0   apple     11  1.564438
1          1  orange      5  1.331256
2          2   apple     12  2.393235
3          3   apple      6  0.746937
4          4   apple      5  2.691024
5          5  orange     12  3.767211
6          6   apple     10  0.992983
7          7   apple     11  3.795525
```

这里，`df['fruit']` 是一个 Python 字符串对象数组。我们可以通过调用以下方法将其转换为分类类型：

```python
In [213]: fruit_cat = df['fruit'].astype('category')

In [214]: fruit_cat
Out[214]: 
0     apple
1    orange
2     apple
3     apple
4     apple
5    orange
6     apple
7     apple
Name: fruit, dtype: category
Categories (2, object): ['apple', 'orange']
```

现在，`fruit_cat` 的值是一个 `pandas.Categorical` 实例，您可以通过 `.array` 属性访问它：

```python
In [215]: c = fruit_cat.array

In [216]: type(c)
Out[216]: pandas.core.arrays.categorical.Categorical
```

`Categorical` 对象具有 `categories` 和 `codes` 属性：

```python
In [217]: c.categories
Out[217]: Index(['apple', 'orange'], dtype='object')

In [218]: c.codes
Out[218]: array([0, 1, 0, 0, 0, 1, 0, 0], dtype=int8)
```

这些可以通过 `cat` 访问器更轻松地访问，这将在“分类方法”部分 soon 解释。

获取编码和类别之间映射的一个有用技巧是：

```python
In [219]: dict(enumerate(c.categories))
Out[219]: {0: 'apple', 1: 'orange'}
```

您可以通过分配转换结果将 DataFrame 列转换为分类类型：

```python
In [220]: df['fruit'] = df['fruit'].astype('category')

In [221]: df["fruit"]
Out[221]: 
0     apple
1    orange
2     apple
3     apple
4     apple
5    orange
6     apple
7     apple
Name: fruit, dtype: category
Categories (2, object): ['apple', 'orange']
```

您也可以直接从其他类型的 Python 序列创建 `pandas.Categorical`：

```python
In [222]: my_categories = pd.Categorical(['foo', 'bar', 'baz', 'foo', 'bar'])

In [223]: my_categories
Out[223]: 
['foo', 'bar', 'baz', 'foo', 'bar']
Categories (3, object): ['bar', 'baz', 'foo']
```

如果您从其他来源获得了分类编码数据，可以使用替代的 `from_codes` 构造函数：

```python
In [224]: categories = ['foo', 'bar', 'baz']

In [225]: codes = [0, 1, 2, 0, 0, 1]

In [226]: my_cats_2 = pd.Categorical.from_codes(codes, categories)

In [227]: my_cats_2
Out[227]: 
['foo', 'bar', 'baz', 'foo', 'foo', 'bar']
Categories (3, object): ['foo', 'bar', 'baz']
```

除非明确指定，否则分类转换假定类别没有特定的顺序。因此，`categories` 数组的顺序可能因输入数据的顺序而异。当使用 `from_codes` 或任何其他构造函数时，您可以指示类别具有有意义的顺序：

```python
In [228]: ordered_cat = pd.Categorical.from_codes(codes, categories,
   .....:                                         ordered=True)

In [229]: ordered_cat
Out[229]: 
['foo', 'bar', 'baz', 'foo', 'foo', 'bar']
Categories (3, object): ['foo' < 'bar' < 'baz']
```

输出 `[foo < bar < baz]` 表示 `'foo'` 在排序中位于 `'bar'` 之前，依此类推。可以使用 `as_ordered` 将无序的分类实例变为有序：

```python
In [230]: my_cats_2.as_ordered()
Out[230]: 
['foo', 'bar', 'baz', 'foo', 'foo', 'bar']
Categories (3, object): ['foo' < 'bar' < 'baz']
```

最后一点需要注意的是，分类数据不必是字符串，尽管我只展示了字符串示例。分类数组可以由任何不可变的值类型组成。

### 使用分类数据进行计算

在 pandas 中使用 `Categorical` 与使用非编码版本（如字符串数组）相比，行为方式大体相同。pandas 的某些部分，如 `groupby` 函数，在处理分类数据时表现更好。还有一些函数可以利用 `ordered` 标志。

让我们考虑一些随机数值数据，并使用 `pandas.qcut` 分箱函数。这会返回 `pandas.Categorical`；我们在本书前面使用过 `pandas.cut`，但忽略了分类工作方式的细节：

```python
In [231]: rng = np.random.default_rng(seed=12345)

In [232]: draws = rng.standard_normal(1000)

In [233]: draws[:5]
Out[233]: array([-1.4238,  1.2637, -0.8707, -0.2592, -0.0753])
```

让我们计算这些数据的四分位数分箱并提取一些统计信息：

```python
In [234]: bins = pd.qcut(draws, 4)

In [235]: bins
Out[235]: 
[(-3.121, -0.675], (0.687, 3.211], (-3.121, -0.675], (-0.675, 0.0134], (-0.675, 0
.0134], ..., (0.0134, 0.687], (0.0134, 0.687], (-0.675, 0.0134], (0.0134, 0.687],
 (-0.675, 0.0134]]
Length: 1000
Categories (4, interval[float64, right]): [(-3.121, -0.675] < (-0.675, 0.0134] < 
(0.0134, 0.687] <
                                              (0.687, 3.211]]
```

虽然有用，但精确的样本四分位数对于生成报告可能不如四分位数名称有用。我们可以使用 `qcut` 的 `labels` 参数来实现这一点：

```python
In [236]: bins = pd.qcut(draws, 4, labels=['Q1', 'Q2', 'Q3', 'Q4'])

In [237]: bins
Out[237]: 
['Q1', 'Q4', 'Q1', 'Q2', 'Q2', ..., 'Q3', 'Q3', 'Q2', 'Q3', 'Q2']
Length: 1000
Categories (4, object): ['Q1' < 'Q2' < 'Q3' < 'Q4']

In [238]: bins.codes[:10]
Out[238]: array([0, 3, 0, 1, 1, 0, 0, 2, 2, 0], dtype=int8)
```

带标签的 `bins` 分类数据不包含数据中箱边界的信息，因此我们可以使用 `groupby` 来提取一些汇总统计信息：

```python
In [239]: bins = pd.Series(bins, name='quartile')

In [240]: results = (pd.Series(draws)
   .....:            .groupby(bins)
   .....:            .agg(['count', 'min', 'max'])
   .....:            .reset_index())

In [241]: results
Out[241]: 
  quartile  count       min       max
0       Q1    250 -3.119609 -0.678494
1       Q2    250 -0.673305  0.008009
2       Q3    250  0.018753  0.686183
3       Q4    250  0.688282  3.211418
```

结果中的 `'quartile'` 列保留了来自 `bins` 的原始分类信息，包括排序：

```python
In [242]: results['quartile']
Out[242]: 
0    Q1
1    Q2
2    Q3
3    Q4
Name: quartile, dtype: category
Categories (4, object): ['Q1' < 'Q2' < 'Q3' < 'Q4']
```

#### 使用分类数据获得更好的性能

在本节开头，我说过分类类型可以提高性能和内存使用效率，所以让我们看一些例子。考虑一个包含 1000 万个元素且只有少量 distinct 类别的 Series：

```python
In [243]: N = 10_000_000

In [244]: labels = pd.Series(['foo', 'bar', 'baz', 'qux'] * (N // 4))
```

现在我们将 `labels` 转换为分类类型：

```python
In [245]: categories = labels.astype('category')
```

现在我们注意到 `labels` 使用的内存明显多于 `categories`：

```python
In [246]: labels.memory_usage(deep=True)
Out[246]: 600000128

In [247]: categories.memory_usage(deep=True)
Out[247]: 10000540
```

当然，转换为分类类型并非没有成本，但它是一次性成本：

```python
In [248]: %time _ = labels.astype('category')
CPU times: user 279 ms, sys: 6.06 ms, total: 285 ms
Wall time: 285 ms
```

使用分类数据，GroupBy 操作可以显著更快，因为底层算法使用基于整数的编码数组，而不是字符串数组。这里我们比较 `value_counts()` 的性能，它在内部使用了 GroupBy 机制：

```python
In [249]: %timeit labels.value_counts()
331 ms +- 5.39 ms per loop (mean +- std. dev. of 7 runs, 1 loop each)

In [250]: %timeit categories.value_counts()
15.6 ms +- 152 us per loop (mean +- std. dev. of 7 runs, 100 loops each)
```

### 分类方法

包含分类数据的 Series 有几个特殊的方法，类似于 `Series.str` 专用字符串方法。这也提供了对类别和编码的便捷访问。考虑以下 Series：

```python
In [251]: s = pd.Series(['a', 'b', 'c', 'd'] * 2)

In [252]: cat_s = s.astype('category')

In [253]: cat_s
Out[253]: 
0    a
1    b
2    c
3    d
4    a
5    b
6    c
7    d
dtype: category
Categories (4, object): ['a', 'b', 'c', 'd']
```

特殊的*访问器*（accessor）属性 `cat` 提供了对分类方法的访问：

```python
In [254]: cat_s.cat.codes
Out[254]: 
0    0
1    1
2    2
3    3
4    0
5    1
6    2
7    3
dtype: int8

In [255]: cat_s.cat.categories
Out[255]: Index(['a', 'b', 'c', 'd'], dtype='object')
```

假设我们知道此数据的实际类别集超出了数据中观察到的四个值。我们可以使用 `set_categories` 方法来更改它们：

```python
In [256]: actual_categories = ['a', 'b', 'c', 'd', 'e']

In [257]: cat_s2 = cat_s.cat.set_categories(actual_categories)

In [258]: cat_s2
Out[258]: 
0    a
1    b
2    c
3    d
4    a
5    b
6    c
7    d
dtype: category
Categories (5, object): ['a', 'b', 'c', 'd', 'e']
```

虽然数据看起来没有变化，但新的类别将反映在使用它们的操作中。例如，`value_counts` 会遵循存在的类别：

```python
In [259]: cat_s.value_counts()
Out[259]: 
a    2
b    2
c    2
d    2
Name: count, dtype: int64

In [260]: cat_s2.value_counts()
Out[260]: 
a    2
b    2
c    2
d    2
e    0
Name: count, dtype: int64
```

在大型数据集中，分类数据通常用作节省内存和提高性能的便捷工具。在过滤大型 DataFrame 或 Series 之后，许多类别可能不会出现在数据中。为了帮助解决这个问题，我们可以使用 `remove_unused_categories` 方法来修剪未观察到的类别：

```python
In [261]: cat_s3 = cat_s[cat_s.isin(['a', 'b'])]

In [262]: cat_s3
Out[262]: 
0    a
1    b
4    a
5    b
dtype: category
Categories (4, object): ['a', 'b', 'c', 'd']

In [263]: cat_s3.cat.remove_unused_categories()
Out[263]: 
0    a
1    b
4    a
5    b
dtype: category
Categories (2, object): ['a', 'b']
```

可用分类方法的列表请参见表 7.7。

**表 7.7: pandas 中 Series 的分类方法**

方法 | 描述
---|---
`add_categories` | 在现有类别的末尾追加新的（未使用的）类别
`as_ordered` | 使类别有序
`as_unordered` | 使类别无序
`remove_categories` | 移除类别，将任何移除的值设置为 null
`remove_unused_categories` | 移除数据中未出现的任何类别值
`rename_categories` | 用指定的新类别名称集替换类别；不能更改类别数量
`reorder_categories` | 行为类似于 `rename_categories`，但还可以将结果更改为具有有序类别
`set_categories` | 用指定的新类别集替换类别；可以添加或删除类别

#### 为建模创建虚拟变量 (dummy variables)

当使用统计或机器学习工具时，您经常需要将分类数据转换为*虚拟变量*（dummy variables），也称为*独热编码*（one-hot encoding）。这涉及创建一个 DataFrame，其中每个 distinct 类别都有一列；这些列包含给定类别出现时为 1，否则为 0。

考虑之前的例子：

```python
In [264]: cat_s = pd.Series(['a', 'b', 'c', 'd'] * 2, dtype='category')
```

如本章前面提到的，`pandas.get_dummies` 函数将这一维分类数据转换为包含虚拟变量的 DataFrame：

```python
In [265]: pd.get_dummies(cat_s, dtype=float)
Out[265]: 
         a    b    c    d
0  1.0  0.0  0.0  0.0
1  0.0  1.0  0.0  0.0
2  0.0  0.0  1.0  0.0
3  0.0  0.0  0.0  1.0
4  1.0  0.0  0.0  0.0
5  0.0  1.0  0.0  0.0
6  0.0  0.0  1.0  0.0
7  0.0  0.0  0.0  1.0
```

## 7.6 结论

高效的数据准备能显著提升生产力——它让你将更多时间投入数据分析而非数据预处理。本章我们探讨了多种工具，但这里的覆盖范围远非全面。下一章中，我们将深入探索pandas的连接（joining）与分组（grouping）功能。