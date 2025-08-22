# 8 数据整理（Data Wrangling）：连接、合并和重塑

__

《利用 Python 进行数据分析 第三版》的开放获取网络版现已上线，作为[印刷版与电子版](https://amzn.to/3DyLaJc)的配套资源。如果您发现任何勘误，[请在此处提交](https://oreilly.com/catalog/0636920519829/errata)。请注意，由 Quarto 生成的本站点某些格式会与 O’Reilly 出版的印刷版及电子书版本有所不同。

如果您觉得本书在线版对您有帮助，请考虑[订购纸质版](https://amzn.to/3DyLaJc)或[无DRM限制的电子书](https://www.ebooks.com/en-us/book/210644288/python-for-data-analysis/wes-mckinney/?affId=WES398681F)以支持作者。本站内容禁止复制或转载。代码示例遵循 MIT 许可协议，可在 GitHub 或 Gitee 上获取。

在许多应用场景中，数据可能分散在多个文件或数据库中，或以不便于分析的形式排列。本章重点介绍有助于组合、连接和重新排列数据的工具。

首先，我将介绍 pandas 中的_分层索引_（hierarchical indexing）概念，该概念在这些操作中被广泛使用。随后我将深入讲解具体的数据操作方法。您可以在第 13 章：数据分析示例中查看这些工具的各种应用场景。

## 8.1 分层索引（Hierarchical Indexing）

分层索引（Hierarchical indexing）是 pandas 的一个重要特性，它允许你在一个轴（axis）上拥有多个（两个或更多）索引层级（levels）。另一种理解方式是，它提供了一种用低维形式处理高维数据的方法。让我们从一个简单示例开始：创建一个索引为列表（或数组）的列表的 Series：

```python
In [11]: data = pd.Series(np.random.uniform(size=9),
   ....:                  index=[["a", "a", "a", "b", "b", "c", "c", "d", "d"],
   ....:                         [1, 2, 3, 1, 3, 1, 2, 2, 3]])

In [12]: data
Out[12]: 
a  1    0.929616
   2    0.316376
   3    0.183919
b  1    0.204560
   3    0.567725
c  1    0.595545
   2    0.964515
d  2    0.653177
   3    0.748907
dtype: float64
```

你看到的是一个以 `MultiIndex` 作为索引的 Series 的美化视图。索引显示中的“空白”表示“使用正上方的标签”：

```python
In [13]: data.index
Out[13]: 
MultiIndex([('a', 1),
            ('a', 2),
            ('a', 3),
            ('b', 1),
            ('b', 3),
            ('c', 1),
            ('c', 2),
            ('d', 2),
            ('d', 3)],
           )
```

对于具有分层索引的对象，可以进行所谓的部分（partial）索引，使你能够简洁地选择数据的子集：

```python
In [14]: data["b"]
Out[14]: 
1    0.204560
3    0.567725
dtype: float64

In [15]: data["b":"c"]
Out[15]: 
b  1    0.204560
   3    0.567725
c  1    0.595545
   2    0.964515
dtype: float64

In [16]: data.loc[["b", "d"]]
Out[16]: 
b  1    0.204560
   3    0.567725
d  2    0.653177
   3    0.748907
dtype: float64
```

甚至可以从“内部”层级进行选择。这里，我从第二个索引层级中选择所有值为 `2` 的数据：

```python
In [17]: data.loc[:, 2]
Out[17]: 
a    0.316376
c    0.964515
d    0.653177
dtype: float64
```

分层索引在数据重塑和基于分组的操作（如数据透视表）中扮演着重要角色。例如，你可以使用 `unstack` 方法将此数据重新排列成 DataFrame：

```python
In [18]: data.unstack()
Out[18]: 
          1         2         3
a  0.929616  0.316376  0.183919
b  0.204560       NaN  0.567725
c  0.595545  0.964515       NaN
d       NaN  0.653177  0.748907
```

`unstack` 的逆操作是 `stack`：

```python
In [19]: data.unstack().stack()
Out[19]: 
a  1    0.929616
   2    0.316376
   3    0.183919
b  1    0.204560
   3    0.567725
c  1    0.595545
   2    0.964515
d  2    0.653177
   3    0.748907
dtype: float64
```

`stack` 和 `unstack` 将在后面的“数据重塑与透视”章节中详细探讨。

对于 DataFrame，任何一个轴都可以具有分层索引：

```python
In [20]: frame = pd.DataFrame(np.arange(12).reshape((4, 3)),
   ....:                      index=[["a", "a", "b", "b"], [1, 2, 1, 2]],
   ....:                      columns=[["Ohio", "Ohio", "Colorado"],
   ....:                               ["Green", "Red", "Green"]])

In [21]: frame
Out[21]: 
        Ohio     Colorado
       Green Red    Green
a 1     0   1        2
  2     3   4        5
b 1     6   7        8
  2     9  10       11
```

层级可以具有名称（作为字符串或任何 Python 对象）。如果有名称，它们将显示在控制台输出中：

```python
In [22]: frame.index.names = ["key1", "key2"]

In [23]: frame.columns.names = ["state", "color"]

In [24]: frame
Out[24]: 
state      Ohio     Colorado
color     Green Red    Green
key1 key2                   
a    1        0   1        2
     2        3   4        5
b    1        6   7        8
     2        9  10       11
```

这些名称会覆盖 `name` 属性，该属性仅用于单级索引。

> 注意 
> 请注意，索引名称 `"state"` 和 `"color"` 不是行标签（即 `frame.index` 的值）的一部分。

你可以通过访问索引的 `nlevels` 属性来查看它有多少个层级：

```python
In [25]: frame.index.nlevels
Out[25]: 2
```

通过部分列索引，你可以类似地选择列组：

```python
In [26]: frame["Ohio"]
Out[26]: 
color      Green  Red
key1 key2            
a    1         0    1
     2         3    4
b    1         6    7
     2         9   10
```

`MultiIndex` 可以单独创建并重复使用；前面带有层级名称的 DataFrame 的列也可以这样创建：

```python
pd.MultiIndex.from_arrays([["Ohio", "Ohio", "Colorado"],
                          ["Green", "Red", "Green"]],
                          names=["state", "color"])
```

### 重排序和排序层级

有时你可能需要重新排列轴上的层级顺序，或根据特定层级中的值对数据进行排序。`swaplevel` 方法接受两个层级编号或名称，并返回一个层级交换后的新对象（但数据本身未被修改）：

```python
In [27]: frame.swaplevel("key1", "key2")
Out[27]: 
state      Ohio     Colorado
color     Green Red    Green
key2 key1                   
1    a        0   1        2
2    a        3   4        5
1    b        6   7        8
2    b        9  10       11
```

默认情况下，`sort_index` 会使用所有索引层级对数据进行字典序排序，但你可以通过传递 `level` 参数选择仅使用单个层级或层级子集进行排序。例如：

```python
In [28]: frame.sort_index(level=1)
Out[28]: 
state      Ohio     Colorado
color     Green Red    Green
key1 key2                   
a    1        0   1        2
b    1        6   7        8
a    2        3   4        5
b    2        9  10       11

In [29]: frame.swaplevel(0, 1).sort_index(level=0)
Out[29]: 
state      Ohio     Colorado
color     Green Red    Green
key2 key1                   
1    a        0   1        2
     b        6   7        8
2    a        3   4        5
     b        9  10       11
```

> 注意 
> 如果分层索引对象的索引从最外层开始按字典序排序，即调用 `sort_index(level=0)` 或 `sort_index()` 的结果，那么数据选择的性能会好得多。

### 按层级汇总统计

DataFrame 和 Series 上的许多描述性和汇总统计信息都有一个 `level` 选项，你可以在其中指定要在特定轴上聚合的层级。考虑上面的 DataFrame；我们可以按行或列的层级进行聚合，如下所示：

```python
In [30]: frame.groupby(level="key2").sum()
Out[30]: 
state  Ohio     Colorado
color Green Red    Green
key2                    
1         6   8       10
2        12  14       16

In [31]: frame.groupby(level="color", axis="columns").sum()
Out[31]: 
color      Green  Red
key1 key2            
a    1         2    1
     2         8    4
b    1        14    7
     2        20   10
```

我们将在[第 10 章：数据聚合与分组操作](https://wesmckinney.com/book/data-aggregation)中更详细地讨论 `groupby`。

### 使用 DataFrame 的列进行索引

通常希望使用 DataFrame 中的一个或多个列作为行索引；或者，你可能希望将行索引移动到 DataFrame 的列中。以下是一个示例 DataFrame：

```python
In [32]: frame = pd.DataFrame({"a": range(7), "b": range(7, 0, -1),
   ....:                       "c": ["one", "one", "one", "two", "two",
   ....:                             "two", "two"],
   ....:                       "d": [0, 1, 2, 0, 1, 2, 3]})

In [33]: frame
Out[33]: 
   a  b    c  d
0  0  7  one  0
1  1  6  one  1
2  2  5  one  2
3  3  4  two  0
4  4  3  two  1
5  5  2  two  2
6  6  1  two  3
```

DataFrame 的 `set_index` 函数将使用一个或多个列作为索引来创建新的 DataFrame：

```python
In [34]: frame2 = frame.set_index(["c", "d"])

In [35]: frame2
Out[35]: 
         a  b
c   d      
one 0  0  7
    1  1  6
    2  2  5
two 0  3  4
    1  4  3
    2  5  2
    3  6  1
```

默认情况下，这些列会从 DataFrame 中移除，但你可以通过向 `set_index` 传递 `drop=False` 来保留它们：

```python
In [36]: frame.set_index(["c", "d"], drop=False)
Out[36]: 
         a  b    c  d
c   d              
one 0  0  7  one  0
    1  1  6  one  1
    2  2  5  one  2
two 0  3  4  two  0
    1  4  3  two  1
    2  5  2  two  2
    3  6  1  two  3
```

另一方面，`reset_index` 的作用与 `set_index` 相反；分层索引的层级会被移动到列中：

```python
In [37]: frame2.reset_index()
Out[37]: 
     c  d  a  b
0  one  0  0  7
1  one  1  1  6
2  one  2  2  5
3  two  0  3  4
4  two  1  4  3
5  two  2  5  2
6  two  3  6  1
```

## 8.2 数据集合并与连接

pandas对象中包含的数据可以通过多种方式组合：

`pandas.merge`
    
基于一个或多个键连接DataFrame中的行。对于SQL或其他关系型数据库的用户来说，这会很熟悉，因为它实现了数据库的_连接（join）_操作。

`pandas.concat`
    
沿轴方向拼接或"堆叠"对象。

`combine_first`
    
拼接重叠数据，用一个对象中的值填充另一个对象中的缺失值。

我将逐一介绍这些方法，并给出一些示例。在本书后续的示例中，它们将会被用到。

### 数据库风格的DataFrame连接

_合并（merge）_或_连接（join）_操作通过使用一个或多个_键（keys）_链接行来组合数据集。这些操作在关系型数据库（例如基于SQL的数据库）中特别重要。pandas中的`pandas.merge`函数是在数据上使用这些算法的主要入口点。

让我们从一个简单的例子开始：
    
    
    In [38]: df1 = pd.DataFrame({"key": ["b", "b", "a", "c", "a", "a", "b"],
       ....:                     "data1": pd.Series(range(7), dtype="Int64")})
    
    In [39]: df2 = pd.DataFrame({"key": ["a", "b", "d"],
       ....:                     "data2": pd.Series(range(3), dtype="Int64")})
    
    In [40]: df1
    Out[40]: 
      key  data1
    0   b      0
    1   b      1
    2   a      2
    3   c      3
    4   a      4
    5   a      5
    6   b      6
    
    In [41]: df2
    Out[41]: 
      key  data2
    0   a      0
    1   b      1
    2   d      2 __

这里我使用了pandas的`Int64`扩展类型来表示可空整数，这在[第7.3节：扩展数据类型](https://wesmckinney.com/book/data-cleaning#pandas-ext-types)中讨论过。

这是一个_多对一（many-to-one）_连接的例子；`df1`中的数据有多个标记为`a`和`b`的行，而`df2`在`key`列中每个值只有一行。对这些对象调用`pandas.merge`，我们得到：
    
    
    In [42]: pd.merge(df1, df2)
    Out[42]: 
      key  data1  data2
    0   b      0      1
    1   b      1      1
    2   b      6      1
    3   a      2      0
    4   a      4      0
    5   a      5      0 __

注意我没有指定要连接哪一列。如果未指定该信息，`pandas.merge`会使用重叠的列名作为键。但显式指定是一个好习惯：
    
    
    In [43]: pd.merge(df1, df2, on="key")
    Out[43]: 
      key  data1  data2
    0   b      0      1
    1   b      1      1
    2   b      6      1
    3   a      2      0
    4   a      4      0
    5   a      5      0 __

通常，`pandas.merge`操作中列输出的顺序是不确定的。

如果每个对象中的列名不同，你可以分别指定它们：
    
    
    In [44]: df3 = pd.DataFrame({"lkey": ["b", "b", "a", "c", "a", "a", "b"],
       ....:                     "data1": pd.Series(range(7), dtype="Int64")})
    
    In [45]: df4 = pd.DataFrame({"rkey": ["a", "b", "d"],
       ....:                     "data2": pd.Series(range(3), dtype="Int64")})
    
    In [46]: pd.merge(df3, df4, left_on="lkey", right_on="rkey")
    Out[46]: 
      lkey  data1 rkey  data2
    0    b      0    b      1
    1    b      1    b      1
    2    b      6    b      1
    3    a      2    a      0
    4    a      4    a      0
    5    a      5    a      0 __

你可能会注意到结果中缺少`"c"`和`"d"`值及相关数据。默认情况下，`pandas.merge`执行`"inner"`连接；结果中的键是交集，或两个表中找到的公共集合。其他可能的选项是`"left"`、`"right"`和`"outer"`。外连接（outer join）取键的并集，结合了左连接和右连接的效果：
    
    
    In [47]: pd.merge(df1, df2, how="outer")
    Out[47]: 
      key  data1  data2
    0   b      0      1
    1   b      1      1
    2   b      6      1
    3   a      2      0
    4   a      4      0
    5   a      5      0
    6   c      3   <NA>
    7   d   <NA>      2
    
    In [48]: pd.merge(df3, df4, left_on="lkey", right_on="rkey", how="outer")
    Out[48]: 
      lkey  data1 rkey  data2
    0    b      0    b      1
    1    b      1    b      1
    2    b      6    b      1
    3    a      2    a      0
    4    a      4    a      0
    5    a      5    a      0
    6    c      3  NaN   <NA>
    7  NaN   <NA>    d      2 __

在外连接中，来自左或右DataFrame对象中未在另一个DataFrame的键上匹配的行，将在另一个DataFrame的列中显示为NA值。

有关`how`选项的总结，请参见表8.1。

表8.1：使用`how`参数的不同连接类型 选项 | 行为  
---|---  
`how="inner"` | 仅使用两个表中都观察到的键组合  
`how="left"` | 使用左表中找到的所有键组合  
`how="right"` | 使用右表中找到的所有键组合  
`how="outer"` | 使用两个表中观察到的所有键组合  
  
_多对多（many-to-many）_合并形成匹配键的笛卡尔积。以下是一个例子：
    
    
    In [49]: df1 = pd.DataFrame({"key": ["b", "b", "a", "c", "a", "b"],
       ....:                     "data1": pd.Series(range(6), dtype="Int64")})
    
    In [50]: df2 = pd.DataFrame({"key": ["a", "b", "a", "b", "d"],
       ....:                     "data2": pd.Series(range(5), dtype="Int64")})
    
    In [51]: df1
    Out[51]: 
      key  data1
    0   b      0
    1   b      1
    2   a      2
    3   c      3
    4   a      4
    5   b      5
    
    In [52]: df2
    Out[52]: 
      key  data2
    0   a      0
    1   b      1
    2   a      2
    3   b      3
    4   d      4
    
    In [53]: pd.merge(df1, df2, on="key", how="left")
    Out[53]: 
       key  data1  data2
    0    b      0      1
    1    b      0      3
    2    b      1      1
    3    b      1      3
    4    a      2      0
    5    a      2      2
    6    c      3   <NA>
    7    a      4      0
    8    a      4      2
    9    b      5      1
    10   b      5      3 __

由于左DataFrame中有三个`"b"`行，右DataFrame中有两个，因此结果中有六个`"b"`行。传递给`how`关键字参数的连接方法仅影响结果中出现的不同键值：
    
    
    In [54]: pd.merge(df1, df2, how="inner")
    Out[54]: 
      key  data1  data2
    0   b      0      1
    1   b      0      3
    2   b      1      1
    3   b      1      3
    4   b      5      1
    5   b      5      3
    6   a      2      0
    7   a      2      2
    8   a      4      0
    9   a      4      2 __

要使用多个键进行合并，传递列名列表：
    
    
    In [55]: left = pd.DataFrame({"key1": ["foo", "foo", "bar"],
       ....:                      "key2": ["one", "two", "one"],
       ....:                      "lval": pd.Series([1, 2, 3], dtype='Int64')})
    
    In [56]: right = pd.DataFrame({"key1": ["foo", "foo", "bar", "bar"],
       ....:                       "key2": ["one", "one", "one", "two"],
       ....:                       "rval": pd.Series([4, 5, 6, 7], dtype='Int64')})
    
    In [57]: pd.merge(left, right, on=["key1", "key2"], how="outer")
    Out[57]: 
      key1 key2  lval  rval
    0  foo  one     1     4
    1  foo  one     1     5
    2  foo  two     2  <NA>
    3  bar  one     3     6
    4  bar  two  <NA>     7 __

要根据合并方法的选择确定哪些键组合将出现在结果中，可以将多个键视为形成一个元组数组，用作单个连接键。

__

注意 

当你在列上连接列时，传递的DataFrame对象的索引将被丢弃。如果需要保留索引值，可以使用`reset_index`将索引追加到列中。

合并操作中最后一个要考虑的问题是重叠列名的处理。例如：
    
    
    In [58]: pd.merge(left, right, on="key1")
    Out[58]: 
      key1 key2_x  lval key2_y  rval
    0  foo    one     1    one     4
    1  foo    one     1    one     5
    2  foo    two     2    one     4
    3  foo    two     2    one     5
    4  bar    one     3    one     6
    5  bar    one     3    two     7 __

虽然你可以手动处理重叠（参见[第7.2.4节：重命名轴索引](https://wesmckinney.com/book/data-cleaning#prep_renaming)以了解重命名轴标签），但`pandas.merge`有一个`suffixes`选项，用于指定要附加到左和右DataFrame对象中重叠名称的字符串：
    
    
    In [59]: pd.merge(left, right, on="key1", suffixes=("_left", "_right"))
    Out[59]: 
      key1 key2_left  lval key2_right  rval
    0  foo       one     1        one     4
    1  foo       one     1        one     5
    2  foo       two     2        one     4
    3  foo       two     2        one     5
    4  bar       one     3        one     6
    5  bar       one     3        two     7 __

有关`pandas.merge`的参数参考，请参见表8.2。下一节将介绍使用DataFrame的行索引进行连接。

表8.2：`pandas.merge`函数参数 参数 | 描述  
---|---  
`left` | 要在左侧合并的DataFrame。  
`right` | 要在右侧合并的DataFrame。  
`how` | 要应用的连接类型：`"inner"`、`"outer"`、`"left"`或`"right"`之一；默认为`"inner"`。  
`on` | 要连接的列名。必须在两个DataFrame对象中都找到。如果未指定且未给出其他连接键，则将使用`left`和`right`中列名的交集作为连接键。  
`left_on` | `left` DataFrame中用作连接键的列。可以是单个列名或列名列表。  
`right_on` | 类似于`left_on`，但用于`right` DataFrame。  
`left_index` | 使用`left`中的行索引作为其连接键（如果是`MultiIndex`，则为键）。  
`right_index` | 类似于`left_index`。  
`sort` | 按连接键对合并数据进行字典排序；默认为`False`。  
`suffixes` | 在重叠情况下要附加到列名的字符串元组；默认为`("_x", "_y")`（例如，如果两个DataFrame对象中都有`"data"`，则在结果中显示为`"data_x"`和`"data_y"`）。  
`copy` | 如果为`False`，在某些特殊情况下避免将数据复制到结果数据结构中；默认情况下总是复制。  
`validate` | 验证合并是否为指定类型，无论是一对一、一对多还是多对多。有关选项的完整详细信息，请参阅文档字符串。  
`indicator` | 添加一个特殊的列`_merge`，指示每行的来源；基于每行中连接数据的来源，值将为`"left_only"`、`"right_only"`或`"both"`。  
  
### 在索引上合并

在某些情况下，DataFrame中的合并键（keys）将在其索引（行标签）中找到。在这种情况下，你可以传递`left_index=True`或`right_index=True`（或两者）来指示索引应用作合并键：
    
    
    In [60]: left1 = pd.DataFrame({"key": ["a", "b", "a", "a", "b", "c"],
       ....:                       "value": pd.Series(range(6), dtype="Int64")})
    
    In [61]: right1 = pd.DataFrame({"group_val": [3.5, 7]}, index=["a", "b"])
    
    In [62]: left1
    Out[62]: 
      key  value
    0   a      0
    1   b      1
    2   a      2
    3   a      3
    4   b      4
    5   c      5
    
    In [63]: right1
    Out[63]: 
       group_val
    a        3.5
    b        7.0
    
    In [64]: pd.merge(left1, right1, left_on="key", right_index=True)
    Out[64]: 
      key  value  group_val
    0   a      0        3.5
    2   a      2        3.5
    3   a      3        3.5
    1   b      1        7.0
    4   b      4        7.0 __

__

注意 

如果你仔细看这里，你会看到`left1`的索引值被保留了，而在上面的其他例子中，输入DataFrame对象的索引被丢弃了。因为`right1`的索引是唯一的，所以这种"多对一"合并（使用默认的`how="inner"`方法）可以保留`left1`中与输出行对应的索引值。

由于默认的合并方法是取连接键的交集，你可以通过外连接形成它们的并集：
    
    
    In [65]: pd.merge(left1, right1, left_on="key", right_index=True, how="outer")
    Out[65]: 
      key  value  group_val
    0   a      0        3.5
    2   a      2        3.5
    3   a      3        3.5
    1   b      1        7.0
    4   b      4        7.0
    5   c      5        NaN __

对于分层索引的数据，情况更复杂，因为在索引上连接相当于多键合并：
    
    
    In [66]: lefth = pd.DataFrame({"key1": ["Ohio", "Ohio", "Ohio",
       ....:                                "Nevada", "Nevada"],
       ....:                       "key2": [2000, 2001, 2002, 2001, 2002],
       ....:                       "data": pd.Series(range(5), dtype="Int64")})
    
    In [67]: righth_index = pd.MultiIndex.from_arrays(
       ....:     [
       ....:         ["Nevada", "Nevada", "Ohio", "Ohio", "Ohio", "Ohio"],
       ....:         [2001, 2000, 2000, 2000, 2001, 2002]
       ....:     ]
       ....: )
    
    In [68]: righth = pd.DataFrame({"event1": pd.Series([0, 2, 4, 6, 8, 10], dtype="I
    nt64",
       ....:                                            index=righth_index),
       ....:                        "event2": pd.Series([1, 3, 5, 7, 9, 11], dtype="I
    nt64",
       ....:                                            index=righth_index)})
    
    In [69]: lefth
    Out[69]: 
         key1  key2  data
    0    Ohio  2000     0
    1    Ohio  2001     1
    2    Ohio  2002     2
    3  Nevada  2001     3
    4  Nevada  2002     4
    
    In [70]: righth
    Out[70]: 
                 event1  event2
    Nevada 2001       0       1
           2000       2       3
    Ohio   2000       4       5
           2000       6       7
           2001       8       9
           2002      10      11 __

在这种情况下，你必须指示多个列作为列表进行合并（注意使用`how="outer"`处理重复索引值）：
    
    
    In [71]: pd.merge(lefth, righth, left_on=["key1", "key2"], right_index=True)
    Out[71]: 
         key1  key2  data  event1  event2
    0    Ohio  2000     0       4       5
    0    Ohio  2000     0       6       7
    1    Ohio  2001     1       8       9
    2    Ohio  2002     2      10      11
    3  Nevada  2001     3       0       1
    
    In [72]: pd.merge(lefth, righth, left_on=["key1", "key2"],
       ....:          right_index=True, how="outer")
    Out[72]: 
         key1  key2  data  event1  event2
    0    Ohio  2000     0       4       5
    0    Ohio  2000     0       6       7
    1    Ohio  2001     1       8       9
    2    Ohio  2002     2      10      11
    3  Nevada  2001     3       0       1
    4  Nevada  2002     4    <NA>    <NA>
    4  Nevada  2000  <NA>       2       3 __

使用合并两侧的索引也是可能的：
    
    
    In [73]: left2 = pd.DataFrame([[1., 2.], [3., 4.], [5., 6.]],
       ....:                      index=["a", "c", "e"],
       ....:                      columns=["Ohio", "Nevada"]).astype("Int64")
    
    In [74]: right2 = pd.DataFrame([[7., 8.], [9., 10.], [11., 12.], [13, 14]],
       ....:                       index=["b", "c", "d", "e"],
       ....:                       columns=["Missouri", "Alabama"]).astype("Int64")
    
    In [75]: left2
    Out[75]: 
       Ohio  Nevada
    a     1       2
    c     3       4
    e     5       6
    
    In [76]: right2
    Out[76]: 
       Missouri  Alabama
    b         7        8
    c         9       10
    d        11       12
    e        13       14
    
    In [77]: pd.merge(left2, right2, how="outer", left_index=True, right_index=True)
    Out[77]: 
       Ohio  Nevada  Missouri  Alabama
    a     1       2      <NA>     <NA>
    b  <NA>    <NA>         7        8
    c     3       4         9       10
    d  <NA>    <NA>        11       12
    e     5       6        13       14 __

DataFrame有一个`join`实例方法，用于简化按索引合并。它也可以用于组合具有相同或相似索引但非重叠列的许多DataFrame对象。在前面的例子中，我们可以这样写：
    
    
    In [78]: left2.join(right2, how="outer")
    Out[78]: 
       Ohio  Nevada  Missouri  Alabama
    a     1       2      <NA>     <NA>
    b  <NA>    <NA>         7        8
    c     3       4         9       10
    d  <NA>    <NA>        11       12
    e     5       6        13       14 __

与`pandas.merge`相比，DataFrame的`join`方法默认在连接键上执行左连接。它还支持将传递的DataFrame的索引连接到调用DataFrame的其中一个列上：
    
    
    In [79]: left1.join(right1, on="key")
    Out[79]: 
      key  value  group_val
    0   a      0        3.5
    1   b      1        7.0
    2   a      2        3.5
    3   a      3        3.5
    4   b      4        7.0
    5   c      5        NaN __

你可以将此方法视为将数据连接"到"调用`join`方法的对象中。

最后，对于简单的索引对索引合并，你可以将DataFrame列表传递给`join`，作为使用下一节描述的更通用的`pandas.concat`函数的替代方法：
    
    
    In [80]: another = pd.DataFrame([[7., 8.], [9., 10.], [11., 12.], [16., 17.]],
       ....:                        index=["a", "c", "e", "f"],
       ....:                        columns=["New York", "Oregon"])
    
    In [81]: another
    Out[81]: 
       New York  Oregon
    a       7.0     8.0
    c       9.0    10.0
    e      11.0    12.0
    f      16.0    17.0
    
    In [82]: left2.join([right2, another])
    Out[82]: 
       Ohio  Nevada  Missouri  Alabama  New York  Oregon
    a     1       2      <NA>     <NA>       7.0     8.0
    c     3       4         9       10       9.0    10.0
    e     5       6        13       14      11.0    12.0
    
    In [83]: left2.join([right2, another], how="outer")
    Out[83]: 
       Ohio  Nevada  Missouri  Alabama  New York  Oregon
    a     1       2      <NA>     <NA>       7.0     8.0
    c     3       4         9       10       9.0    10.0
    e     5       6        13       14      11.0    12.0
    b  <NA>    <NA>         7        8       NaN     NaN
    d  <NA>    <NA>        11       12       NaN     NaN
    f  <NA>    <NA>      <NA>     <NA>      16.0    17.0 __

### 沿轴连接

另一种数据组合操作被称为_连接（concatenation）_或_堆叠（stacking）_。NumPy的`concatenate`函数可以用NumPy数组做到这一点：
    
    
    In [84]: arr = np.arange(12).reshape((3, 4))
    
    In [85]: arr
    Out[85]: 
    array([[ 0,  1,  2,  3],
           [ 4,  5,  6,  7],
           [ 8,  9, 10, 11]])
    
    In [86]: np.concatenate([arr, arr], axis=1)
    Out[86]: 
    array([[ 0,  1,  2,  3,  0,  1,  2,  3],
           [ 4,  5,  6,  7,  4,  5,  6, 7],
           [ 8,  9, 10, 11,  8,  9, 10, 11]])__

在pandas对象（如Series和DataFrame）的上下文中，拥有带标签的轴使您能够进一步推广数组连接。特别是，您有许多额外的考虑：

  * 如果对象在其他轴上的索引方式不同，我们是应该组合这些轴上的不同元素，还是只使用共有的值？

  * 连接的数据块是否需要可在结果对象中识别？

  * "连接轴"是否包含需要保留的数据？在许多情况下，DataFrame中的默认整数标签在连接期间最好被丢弃。

pandas中的`concat`函数提供了一致的方式来处理每个这些问题。我将给出一些示例来说明它是如何工作的。假设我们有三个没有索引重叠的Series：
    
    
    In [87]: s1 = pd.Series([0, 1], index=["a", "b"], dtype="Int64")
    
    In [88]: s2 = pd.Series([2, 3, 4], index=["c", "d", "e"], dtype="Int64")
    
    In [89]: s3 = pd.Series([5, 6], index=["f", "g"], dtype="Int64")__

在列表中调用`pandas.concat`这些对象会将值和索引粘合在一起：
    
    
    In [90]: s1
    Out[90]: 
    a    0
    b    1
    dtype: Int64
    
    In [91]: s2
    Out[91]: 
    c    2
    d    3
    e    4
    dtype: Int64
    
    In [92]: s3
    Out[92]: 
    f    5
    g    6
    dtype: Int64
    
    In [93]: pd.concat([s1, s2, s3])
    Out[93]: 
    a    0
    b    1
    c    2
    d    3
    e    4
    f    5
    g    6
    dtype: Int64 __

默认情况下，`pandas.concat`沿`axis="index"`工作，产生另一个Series。如果您传递`axis="columns"`，结果将是一个DataFrame：
    
    
    In [94]: pd.concat([s1, s2, s3], axis="columns")
    Out[94]: 
          0     1     2
    a     0  <NA>  <NA>
    b     1  <NA>  <NA>
    c  <NA>     2  <NA>
    d  <NA>     3  <NA>
    e  <NA>     4  <NA>
    f  <NA>  <NA>     5
    g  <NA>  <NA>     6 __

在这种情况下，其他轴上没有重叠，正如您所看到的，这是索引的并集（`"outer"`连接）。您可以通过传递`join="inner"`来取交集：
    
    
    In [95]: s4 = pd.concat([s1, s3])
    
    In [96]: s4
    Out[96]: 
    a    0
    b    1
    f    5
    g    6
    dtype: Int64
    
    In [97]: pd.concat([s1, s4], axis="columns")
    Out[97]: 
          0  1
    a     0  0
    b     1  1
    f  <NA>  5
    g  <NA>  6
    
    In [98]: pd.concat([s1, s4], axis="columns", join="inner")
    Out[98]: 
       0  1
    a  0  0
    b  1  1 __

在最后一个例子中，由于`join="inner"`选项，`"f"`和`"g"`标签消失了。

一个潜在的问题是连接的部分在结果中无法识别。假设您想在连接轴上创建一个分层索引。为此，使用`keys`参数：
    
    
    In [99]: result = pd.concat([s1, s1, s3], keys=["one", "two", "three"])
    
    In [100]: result
    Out[100]: 
    one    a    0
           b    1
    two    a    0
           b    1
    three  f    5
           g    6
    dtype: Int64
    
    In [101]: result.unstack()
    Out[101]: 
              a     b     f     g
    one       0     1  <NA>  <NA>
    two       0     1  <NA>  <NA>
    three  <NA>  <NA>     5     6 __

在沿`axis="columns"`组合Series的情况下，`keys`成为DataFrame的列标题：
    
    
    In [102]: pd.concat([s1, s2, s3], axis="columns", keys=["one", "two", "three"])
    Out[102]: 
        one   two  three
    a     0  <NA>   <NA>
    b     1  <NA>   <NA>
    c  <NA>     2   <NA>
    d  <NA>     3   <NA>
    e  <NA>     4   <NA>
    f  <NA>  <NA>      5
    g  <NA>  <NA>      6 __

相同的逻辑扩展到DataFrame对象：
    
    
    In [103]: df1 = pd.DataFrame(np.arange(6).reshape(3, 2), index=["a", "b", "c"],
       .....:                    columns=["one", "two"])
    
    In [104]: df2 = pd.DataFrame(5 + np.arange(4).reshape(2, 2), index=["a", "c"],
       .....:                    columns=["three", "four"])
    
    In [105]: df1
    Out[105]: 
       one  two
    a    0    1
    b    2    3
    c    4    5
    
    In [106]: df2
    Out[106]: 
       three  four
    a      5     6
    c      7     8
    
    In [107]: pd.concat([df1, df2], axis="columns", keys=["level1", "level2"])
    Out[107]: 
      level1     level2     
         one two  three four
    a      0   1    5.0  6.0
    b      2   3    NaN  NaN
    c      4   5    7.0  8.0 __

这里`keys`参数用于创建一个分层索引，其中第一级可用于识别每个连接的DataFrame对象。

如果您传递一个对象字典而不是列表，字典的键将用于`keys`选项：
    
    
    In [108]: pd.concat({"level1": df1, "level2": df2}, axis="columns")
    Out[108]: 
      level1     level2     
         one two  three four
    a      0   1    5.0  6.0
    b      2   3    NaN  NaN
    c      4   5    7.0  8.0 __

还有其他参数控制分层索引的创建（参见表8.3）。例如，我们可以使用`names`参数为创建的轴级别命名：
    
    
    In [109]: pd.concat([df1, df2], axis="columns", keys=["level1", "level2"],
       .....:           names=["upper", "lower"])
    Out[109]: 
    upper level1     level2     
    lower    one twothree four
    a          0   1    5.0  6.0
    b          2   3    NaN  NaN
    c          4   5    7.0  8.0 __

最后一个考虑是关于行索引不包含任何相关数据的DataFrames：
    
    
    In [110]: df1 = pd.DataFrame(np.random.standard_normal((3, 4)),
       .....:                    columns=["a", "b", "c", "d"])
    
    In [111]: df2 = pd.DataFrame(np.random.standard_normal((2, 3)),
       .....:                    columns=["b", "d", "a"])
    
    In [112]: df1
    Out[112]: 
              a         b         c         d
    0  1.248804  0.774191 -0.319657 -0.624964
    1  1.078814  0.544647  0.855588  1.343268
    2 -0.267175  1.793095 -0.652929 -1.886837
    
    In [113]: df2
    Out[113]: 
              b         d         a
    0  1.059626  0.644448 -0.007799
    1 -0.449204  2.448963  0.667226 __

在这种情况下，您可以传递`ignore_index=True`，这会丢弃每个DataFrame的索引，并仅连接列中的数据，分配一个新的默认索引：
    
    
    In [114]: pd.concat([df1, df2], ignore_index=True)
    Out[114]: 
              a         b         c         d
    0  1.248804  0.774191 -0.319657 -0.624964
    1  1.078814  0.544647  0.855588  1.343268
    2 -0.267175  1.793095 -0.652929 -1.886837
    3 -0.007799  1.059626       NaN  0.644448
    4  0.667226 -0.449204       NaN  2.448963 __

表8.3描述了`pandas.concat`函数参数。

表8.3：`pandas.concat`函数参数 参数 | 描述  
---|---  
`objs` | 要连接的pandas对象的列表或字典；这是唯一必需的参数  
`axis` | 要连接的轴；默认为沿行连接（`axis="index"`）  
`join` | `"inner"`或`"outer"`（默认为`"outer"`）；是取其他轴上索引的交集（inner）还是并集（outer）  
`keys` | 与要连接的对象关联的值，沿连接轴形成分层索引；可以是任意值的列表或数组、元组数组，或数组列表（如果在`levels`中传递了多级数组）  
`levels` | 如果传递了键，则用作分层索引级别的特定索引  
`names` | 如果传递了`keys`和/或`levels`，则为创建的分层级别的名称  
`verify_integrity` | 检查连接对象中的新轴是否有重复项，如果有则引发异常；默认（`False`）允许重复项  
`ignore_index` | 不保留沿连接`axis`的索引，而是生成一个新的`range(total_length)`索引  
  
### 组合重叠数据

还有另一种数据组合情况，不能表示为合并或连接操作。您可能有两个索引完全或部分重叠的数据集。作为一个激励示例，考虑NumPy的`where`函数，它执行数组导向的if-else表达式等效：
    
    
    In [115]: a = pd.Series([np.nan, 2.5, 0.0, 3.5, 4.5, np.nan],
       .....:               index=["f", "e", "d", "c", "b", "a"])
    
    In [116]: b = pd.Series([0., np.nan, 2., np.nan, np.nan, 5.],
       .....:               index=["a", "b", "c", "d", "e", "f"])
    
    In [117]: a
    Out[117]: 
    f    NaN
    e    2.5
    d    0.0
    c    3.5
    b    4.5
    a    NaN
    dtype: float64
    
    In [118]: b
    Out[118]: 
    a    0.0
    b    NaN
    c    2.0
    d    NaN
    e    NaN
    f    5.0
    dtype: float64
    
    In [119]: np.where(pd.isna(a), b, a)
    Out[119]: array([0. , 2.5, 0. , 3.5, 4.5, 5. ])__

在这里，只要`a`中的值为空，就选择`b`中的值，否则选择`a`中的非空值。使用`numpy.where`不检查索引标签是否对齐（甚至不要求对象长度相同），因此如果您想按索引对齐值，请使用Series的`combine_first`方法：
    
    
    In [120]: a.combine_first(b)
    Out[120]: 
    a    0.0
    b    4.5
    c    3.5
    d    0.0
    e    2.5
    f    5.0
    dtype: float64 __

对于DataFrames，`combine_first`逐列执行相同的操作，因此您可以将其视为用传递的对象中的数据"修补"调用对象中的缺失数据：
    
    
    In [121]: df1 = pd.DataFrame({"a": [1., np.nan, 5., np.nan],
       .....:                     "b": [np.nan, 2., np.nan, 6.],
       .....:                     "c": range(2, 18, 4)})
    
    In [122]: df2 = pd.DataFrame({"a": [5., 4., np.nan, 3., 7.],
       .....:                     "b": [np.nan, 3., 4., 6., 8.]})
    
    In [123]: df1
    Out[123]: 
         a    b   c
    0  1.0  NaN   2
    1  NaN  2.0   6
    2  5.0  NaN  10
    3  NaN  6.0  14
    
    In [124]: df2
    Out[124]: 
         a    b
    0  5.0  NaN
    1  4.0  3.0
    2  NaN  4.0
    3  3.0  6.0
    4  7.0  8.0
    
    In [125]: df1.combine_first(df2)
    Out[125]: 
         a    b     c
    0  1.0  NaN   2.0
    1  4.0  2.0   6.0
    2  5.0  4.0  10.0
    3  3.0  6.0  14.0
    4  7.0  8.0   NaN __

`combine_first`与DataFrame对象的输出将包含所有列名的并集。

## 8.3 重塑与透视（Reshaping and Pivoting）

重新排列表格数据的基础操作有多种，这些操作被称为*重塑（reshape）*或*透视（pivot）*操作。

### 使用分层索引进行重塑

分层索引为在 DataFrame 中重新排列数据提供了一致的方法。主要有两种操作：

`stack`
    
此操作将数据从列“旋转”或透视到行。

`unstack`
    
此操作将数据从行透视到列。

我将通过一系列示例来说明这些操作。考虑一个小的 DataFrame，其行和列索引为字符串数组：
    
    
    In [126]: data = pd.DataFrame(np.arange(6).reshape((2, 3)),
       .....:                     index=pd.Index(["Ohio", "Colorado"], name="state"),
       .....:                     columns=pd.Index(["one", "two", "three"],
       .....:                     name="number"))
    
    In [127]: data
    Out[127]: 
    number    one  two  three
    state                    
    Ohio        0    1      2
    Colorado    3    4      5 __

对此数据使用 `stack` 方法会将列透视到行，生成一个 Series：
    
    
    In [128]: result = data.stack()
    
    In [129]: result
    Out[129]: 
    state     number
    Ohio      one       0
              two       1
              three     2
    Colorado  one       3
              two       4
              three     5
    dtype: int64 __

对于具有分层索引的 Series，可以使用 `unstack` 将数据重新排列回 DataFrame：
    
    
    In [130]: result.unstack()
    Out[130]: 
    number    one  two  three
    state                    
    Ohio        0    1      2
    Colorado    3    4      5 __

默认情况下，最内层会被取消堆叠（`stack` 也是如此）。可以通过传递层级编号或名称来取消堆叠不同的层级：
    
    
    In [131]: result.unstack(level=0)
    Out[131]: 
    state   Ohio  Colorado
    number                
    one        0         3
    two        1         4
    three      2         5
    
    In [132]: result.unstack(level="state")
    Out[132]: 
    state   Ohio  Colorado
    number                
    one        0         3
    two        1         4
    three      2         5 __

如果在每个子组中未找到层级中的所有值，取消堆叠可能会引入缺失数据：
    
    
    In [133]: s1 = pd.Series([0, 1, 2, 3], index=["a", "b", "c", "d"], dtype="Int64")
    
    In [134]: s2 = pd.Series([4, 5, 6], index=["c", "d", "e"], dtype="Int64")
    
    In [135]: data2 = pd.concat([s1, s2], keys=["one", "two"])
    
    In [136]: data2
    Out[136]: 
    one  a    0
         b    1
         c    2
         d    3
    two  c    4
         d    5
         e    6
    dtype: Int64 __

默认情况下，堆叠会过滤掉缺失数据，因此该操作更易于反转：
    
    
    In [137]: data2.unstack()
    Out[137]: 
            a     b  c  d     e
    one     0     1  2  3  <NA>
    two  <NA>  <NA>  4  5     6
    
    In [138]: data2.unstack().stack()
    Out[138]: 
    one  a    0
         b    1
         c    2
         d    3
    two  c    4
         d    5
         e    6
    dtype: Int64
    
    In [139]: data2.unstack().stack(dropna=False)
    Out[139]: 
    one  a       0
         b       1
         c       2
         d       3
         e    <NA>
    two  a    <NA>
         b    <NA>
         c       4
         d       5
         e       6
    dtype: Int64 __

当你在 DataFrame 中取消堆叠时，被取消堆叠的层级将成为结果中的最低层级：
    
    
    In [140]: df = pd.DataFrame({"left": result, "right": result + 5},
       .....:                   columns=pd.Index(["left", "right"], name="side"))
    
    In [141]: df
    Out[141]: 
    side             left  right
    state    number             
    Ohio     one        0      5
             two        1      6
             three      2      7
    Colorado one        3      8
             two        4      9
             three      5     10
    
    In [142]: df.unstack(level="state")
    Out[142]: 
    side   left          right         
    state  Ohio Colorado  Ohio Colorado
    number                             
    one       0        3     5        8
    two       1        4     6        9
    three     2        5     7       10 __

与 `unstack` 类似，调用 `stack` 时我们可以指定要堆叠的轴名称：
    
    
    In [143]: df.unstack(level="state").stack(level="side")
    Out[143]: 
    state         Colorado  Ohio
    number side                 
    one    left          3     0
           right         8     5
    two    left          4     1
           right         9     6
    three  left          5     2
           right        10     7 __

### 从“长格式”透视到“宽格式”

在数据库和 CSV 文件中存储多个时间序列的常见方式是所谓的*长格式（long）*或*堆叠格式（stacked）*。在此格式中，单个值由表中的单行表示，而不是每行多个值。

让我们加载一些示例数据，并进行少量时间序列整理和其他数据清理：
    
    
    In [144]: data = pd.read_csv("examples/macrodata.csv")
    
    In [145]: data = data.loc[:, ["year", "quarter", "realgdp", "infl", "unemp"]]
    
    In [146]: data.head()
    Out[146]: 
       year  quarter   realgdp  infl  unemp
    0  1959        1  2710.349  0.00    5.8
    1  1959        2  2778.801  2.34    5.1
    2  1959        3  2775.488  2.74    5.3
    3  1959        4  2785.204  0.27    5.6
    4  1960        1  2847.699  2.31    5.2 __

首先，我使用 `pandas.PeriodIndex`（表示时间间隔而不是时间点，在[第 11 章：时间序列](https://wesmckinney.com/book/time-series)中有更详细的讨论）来合并 `year` 和 `quarter` 列，将索引设置为每季度末的 `datetime` 值：
    
    
    In [147]: periods = pd.PeriodIndex(year=data.pop("year"),
       .....:                          quarter=data.pop("quarter"),
       .....:                          name="date")
    
    In [148]: periods
    Out[148]: 
    PeriodIndex(['1959Q1', '1959Q2', '1959Q3', '1959Q4', '1960Q1', '1960Q2',
                 '1960Q3', '1960Q4', '1961Q1', '1961Q2',
                 ...
                 '2007Q2', '2007Q3', '2007Q4', '2008Q1', '2008Q2', '2008Q3',
                 '2008Q4', '2009Q1', '2009Q2', '2009Q3'],
                dtype='period[Q-DEC]', name='date', length=203)
    
    In [149]: data.index = periods.to_timestamp("D")
    
    In [150]: data.head()
    Out[150]: 
                 realgdp  infl  unemp
    date                             
    1959-01-01  2710.349  0.00    5.8
    1959-04-01  2778.801  2.34    5.1
    1959-07-01  2775.488  2.74    5.3
    1959-10-01  2785.204  0.27    5.6
    1960-01-01  2847.699  2.31    5.2 __

这里我使用了 DataFrame 的 `pop` 方法，该方法返回一列，同时从 DataFrame 中删除该列。

然后，我选择一个列的子集，并为 `columns` 索引指定名称 `"item"`：
    
    
    In [151]: data = data.reindex(columns=["realgdp", "infl", "unemp"])
    
    In [152]: data.columns.name = "item"
    
    In [153]: data.head()
    Out[153]: 
    item         realgdp  infl  unemp
    date                             
    1959-01-01  2710.349  0.00    5.8
    1959-04-01  2778.801  2.34    5.1
    1959-07-01  2775.488  2.74    5.3
    1959-10-01  2785.204  0.27    5.6
    1960-01-01  2847.699  2.31    5.2 __

最后，我使用 `stack` 进行重塑，使用 `reset_index` 将新的索引层级转换为列，并最终将包含数据值的列命名为 `"value"`：
    
    
    In [154]: long_data = (data.stack()
       .....:              .reset_index()
       .....:              .rename(columns={0: "value"}))__

现在，`ldata` 看起来像这样：
    
    
    In [155]: long_data[:10]
    Out[155]: 
            date     item     value
    0 1959-01-01  realgdp  2710.349
    1 1959-01-01     infl     0.000
    2 1959-01-01    unemp     5.800
    3 1959-04-01  realgdp  2778.801
    4 1959-04-01     infl     2.340
    5 1959-04-01    unemp     5.100
    6 1959-07-01  realgdp  2775.488
    7 1959-07-01     infl     2.740
    8 1959-07-01    unemp     5.300
    9 1959-10-01  realgdp  2785.204 __

在这种用于多个时间序列的所谓*长格式*中，表中的每一行代表一个单独的观察值。

数据经常以这种方式存储在关系型 SQL 数据库中，因为固定的模式（列名和数据类型）允许 `item` 列中的不同值数量随着数据添加到表中而变化。在前面的例子中，`date` 和 `item` 通常是主键（用关系数据库的术语来说），既提供了关系完整性，又便于连接。在某些情况下，以这种格式处理数据可能更困难；你可能更希望有一个 DataFrame，其中包含每个不同的 `item` 值的一列，并以 `date` 列中的时间戳作为索引。DataFrame 的 `pivot` 方法正是执行这种转换：
    
    
    In [156]: pivoted = long_data.pivot(index="date", columns="item",
       .....:                           values="value")
    
    In [157]: pivoted.head()
    Out[157]: 
    item        infl   realgdp  unemp
    date                             
    1959-01-01  0.00  2710.349    5.8
    1959-04-01  2.34  2778.801    5.1
    1959-07-01  2.74  2775.488    5.3
    1959-10-01  0.27  2785.204    5.6
    1960-01-01  2.31  2847.699    5.2 __

前两个传递的值分别用作行和列索引的列，最后是一个可选的值列来填充 DataFrame。假设你有两个值列想要同时重塑：
    
    
    In [159]: long_data["value2"] = np.random.standard_normal(len(long_data))
    
    In [160]: long_data[:10]
    Out[160]: 
            date     item     value    value2
    0 1959-01-01  realgdp  2710.349  0.802926
    1 1959-01-01     infl     0.000  0.575721
    2 1959-01-01    unemp     5.800  1.381918
    3 1959-04-01  realgdp  2778.801  0.000992
    4 1959-04-01     infl     2.340 -0.143492
    5 1959-04-01    unemp     5.100 -0.206282
    6 1959-07-01  realgdp  2775.488 -0.222392
    7 1959-07-01     infl     2.740 -1.682403
    8 1959-07-01    unemp     5.300  1.811659
    9 1959-10-01  realgdp  2785.204 -0.351305 __

通过省略最后一个参数，你可以获得一个具有分层列的 DataFrame：
    
    
    In [161]: pivoted = long_data.pivot(index="date", columns="item")
    
    In [162]: pivoted.head()
    Out[162]: 
               value                    value2                    
    item        infl   realgdp unemp      infl   realgdp     unemp
    date                                                          
    1959-01-01  0.00  2710.349   5.8  0.575721  0.802926  1.381918
    1959-04-01  2.34  2778.801   5.1 -0.143492  0.000992 -0.206282
    1959-07-01  2.74  2775.488   5.3 -1.682403 -0.222392  1.811659
    1959-10-01  0.27  2785.204   5.6  0.128317 -0.351305 -1.313554
    1960-01-01  2.31  2847.699   5.2 -0.615939  0.498327  0.174072
    
    In [163]: pivoted["value"].head()
    Out[163]: 
    item        infl   realgdp  unemp
    date                             
    1959-01-01  0.00  2710.349    5.8
    1959-04-01  2.34  2778.801    5.1
    1959-07-01  2.74  2775.488    5.3
    1959-10-01  0.27  2785.204    5.6
    1960-01-01  2.31  2847.699    5.2 __

注意，`pivot` 等同于使用 `set_index` 创建分层索引，然后调用 `unstack`：
    
    
    In [164]: unstacked = long_data.set_index(["date", "item"]).unstack(level="item")
    
    In [165]: unstacked.head()
    Out[165]: 
               value                    value2                    
    item        infl   realgdp unemp      infl   realgdp     unemp
    date                                                          
    1959-01-01  0.00  2710.349   5.8  0.575721  0.802926  1.381918
    1959-04-01  2.34  2778.801   5.1 -0.143492  0.000992 -0.206282
    1959-07-01  2.74  2775.488   5.3 -1.682403 -0.222392  1.811659
    1959-10-01  0.27  2785.204   5.6  0.128317 -0.351305 -1.313554
    1960-01-01  2.31  2847.699   5.2 -0.615939  0.498327  0.174072 __

### 从“宽格式”透视到“长格式”

DataFrame 的 `pivot` 的逆操作是 `pandas.melt`。它不是将一列转换为新 DataFrame 中的多列，而是将多列合并为一列，生成一个比输入更长的 DataFrame。让我们看一个例子：
    
    
    In [167]: df = pd.DataFrame({"key": ["foo", "bar", "baz"],
       .....:                    "A": [1, 2, 3],
       .....:                    "B": [4, 5, 6],
       .....:                    "C": [7, 8, 9]})
    
    In [168]: df
    Out[168]: 
       key  A  B  C
    0  foo  1  4  7
    1  bar  2  5  8
    2  baz  3  6  9 __

`"key"` 列可能是一个分组指示符，其他列是数据值。使用 `pandas.melt` 时，我们必须指定哪些列（如果有）是分组指示符。这里我们使用 `"key"` 作为唯一的分组指示符：
    
    
    In [169]: melted = pd.melt(df, id_vars="key")
    
    In [170]: melted
    Out[170]: 
       key variable  value
    0  foo        A      1
    1  bar        A      2
    2  baz        A      3
    3  foo        B      4
    4  bar        B      5
    5  baz        B      6
    6  foo        C      7
    7  bar        C      8
    8  baz        C      9 __

使用 `pivot`，我们可以重塑回原始布局：
    
    
    In [171]: reshaped = melted.pivot(index="key", columns="variable",
       .....:                         values="value")
    
    In [172]: reshaped
    Out[172]: 
    variable  A  B  C
    key              
    bar       2  5  8
    baz       3  6  9
    foo       1  4  7 __

由于 `pivot` 的结果从用作行标签的列创建索引，我们可能希望使用 `reset_index` 将数据移回列中：
    
    
    In [173]: reshaped.reset_index()
    Out[173]: 
    variable  key  A  B  C
    0         bar  2  5  8
    1         baz  3  6  9
    2         foo  1  4  7 __

你也可以指定要用作 `value` 列的列的子集：
    
    
    In [174]: pd.melt(df, id_vars="key", value_vars=["A", "B"])
    Out[174]: 
       key variable  value
    0  foo        A      1
    1  bar        A      2
    2  baz        A      3
    3  foo        B      4
    4  bar        B      5
    5  baz        B      6 __

`pandas.melt` 也可以在没有分组标识符的情况下使用：
    
    
    In [175]: pd.melt(df, value_vars=["A", "B", "C"])
    Out[175]: 
      variable  value
    0        A      1
    1        A      2
    2        A      3
    3        B      4
    4        B      5
    5        B      6
    6        C      7
    7        C      8
    8        C      9
    
    In [176]: pd.melt(df, value_vars=["key", "A", "B"])
    Out[176]: 
      variable value
    0      key   foo
    1      key   bar
    2      key   baz
    3        A     1
    4        A     2
    5        A     3
    6        B     4
    7        B     5
    8        B     6 __

## 8.4 总结

现在你已经掌握了数据导入、清洗和重组方面的 pandas 基础操作，接下来我们将转向使用 matplotlib 进行数据可视化。在本书后续讨论更高级的分析方法时，我们还会继续探索 pandas 的其他功能领域。