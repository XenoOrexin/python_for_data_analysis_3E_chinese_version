# 10 数据聚合与分组操作

__

《利用 Python 进行数据分析（第三版）》的开放获取网络版本现已作为[印刷版和数字版](https://amzn.to/3DyLaJc)的配套资源发布。如果您发现任何错误，[请在此处报告](https://oreilly.com/catalog/0636920519829/errata)。请注意，由 Quarto 生成的此站点部分格式会与 O’Reilly 出版的印刷版和电子书版本有所不同。

如果您觉得此在线版本有帮助，请考虑[订购纸质版](https://amzn.to/3DyLaJc)或[无DRM限制的电子书](https://www.ebooks.com/en-us/book/210644288/python-for-data-analysis/wes-mckinney/?affId=WES398681F)以支持作者。本网站内容不可复制或转载。代码示例遵循 MIT 许可协议，可在 GitHub 或 Gitee 上获取。

对数据集进行分类并对每个分组应用函数（无论是聚合(aggregation)还是转换(transformation)操作），可能是数据分析工作流中的关键环节。在完成数据集的加载、合并和预处理后，您可能需要计算分组统计量或生成_数据透视表(pivot tables)_ 用于报告或可视化。pandas 提供了一个功能丰富的 `groupby` 接口，使您能够以直观的方式对数据集进行切片、分割和汇总。

关系型数据库和 SQL（即“结构化查询语言”）之所以广受欢迎，原因之一在于其能够轻松实现数据连接、过滤、转换和聚合。然而，像 SQL 这样的查询语言在执行分组操作时存在一定局限性。通过 Python 和 pandas 强大的表达能力，我们可以将复杂的分组操作实现为自定义 Python 函数，直接处理每个分组的数据。本章您将学习如何：

* 使用一个或多个键（以函数、数组或 DataFrame 列名的形式）将 pandas 对象分割成若干部分
* 计算分组摘要统计量，例如计数、均值、标准差或用户自定义函数
* 应用组内转换或其他操作，如标准化、线性回归、排名或子集筛选
* 计算数据透视表与交叉表
* 执行分位数分析及其他统计分组分析

__

注意 

时间序列数据的基于时间的聚合（`groupby` 的一种特殊应用场景）在本书中称为_重采样(resampling)_，将在[第 11 章：时间序列](https://wesmckinney.com/book/time-series)中单独讨论。

与其余章节一样，我们首先导入 NumPy 和 pandas：
    
    
    In [12]: import numpy as np
    
    In [13]: import pandas as pd __

## 10.1 如何理解分组操作（Group Operations）

R 语言许多流行包的作者 Hadley Wickham 创造了术语"拆分-应用-合并"（_split-apply-combine_）来描述分组操作。在此过程的第一阶段，pandas 对象（无论是 Series、DataFrame 还是其他对象）中包含的数据会根据您提供的一个或多个_键_（keys）被_拆分_成组。拆分是在对象的特定轴上执行的。例如，DataFrame 可以按其行（`axis="index"`）或其列（`axis="columns"`）进行分组。完成此操作后，将一个函数_应用_到每个组，产生一个新值。最后，所有这些函数应用的结果被_合并_成一个结果对象。结果对象的形式通常取决于对数据执行的操作。参见图 10.1 以了解一个简单分组聚合的示意图。

每个分组键可以采取多种形式，并且键不必全是同一类型：

*   与待分组轴长度相同的值列表或数组
*   指示 DataFrame 中列名的值
*   给出待分组轴上的值与组名之间对应关系的字典或 Series
*   要在轴索引或索引中的单个标签上调用的函数

![](images/pda3_1001.png)

图 10.1：分组聚合示意图

请注意，后三种方法是生成用于拆分对象的数值数组的快捷方式。如果这一切看起来有些抽象，请不要担心。在本章中，我将提供所有这些方法的许多示例。首先，这里有一个小的表格数据集作为 DataFrame：

```python
In [14]: df = pd.DataFrame({"key1" : ["a", "a", None, "b", "b", "a", None],
       ....:                    "key2" : pd.Series([1, 2, 1, 2, 1, None, 1],
       ....:                                       dtype="Int64"),
       ....:                    "data1" : np.random.standard_normal(7),
       ....:                    "data2" : np.random.standard_normal(7)})

In [15]: df
Out[15]: 
       key1  key2     data1     data2
0     a     1 -0.204708  0.281746
1     a     2  0.478943  0.769023
2  None     1 -0.519439  1.246435
3     b     2 -0.555730  1.007189
4     b     1  1.965781 -1.296221
5     a  <NA>  1.393406  0.274992
6  None     1  0.092908  0.228913
```

假设您想使用 `key1` 中的标签计算 `data1` 列的平均值。有多种方法可以做到这一点。一种是访问 `data1` 并使用 `key1` 处的列（一个 Series）调用 `groupby`：

```python
In [16]: grouped = df["data1"].groupby(df["key1"])

In [17]: grouped
Out[17]: <pandas.core.groupby.generic.SeriesGroupBy object at 0x17b7913f0>
```

这个 `grouped` 变量现在是一个特殊的 _"GroupBy"_ 对象。它实际上还没有计算任何东西，只是计算了关于组键 `df["key1"]` 的一些中间数据。这个对象的理念是，它拥有将所有需要的信息，以便随后对每个组应用某些操作。例如，要计算组均值，我们可以调用 GroupBy 的 `mean` 方法：

```python
In [18]: grouped.mean()
Out[18]: 
key1
a    0.555881
b    0.705025
Name: data1, dtype: float64
```

稍后在“数据聚合”部分，我将详细解释调用 `.mean()` 时会发生什么。这里重要的是，数据（一个 Series）已通过按组键拆分数据进行了聚合，产生了一个新的 Series，该 Series 现在由 `key1` 列中的唯一值索引。结果索引具有名称 `"key1"`，因为 DataFrame 列 `df["key1"]` 也是如此。

如果我们传递多个数组作为列表，我们会得到不同的结果：

```python
In [19]: means = df["data1"].groupby([df["key1"], df["key2"]]).mean()

In [20]: means
Out[20]: 
key1  key2
a     1      -0.204708
      2       0.478943
b     1       1.965781
      2      -0.555730
Name: data1, dtype: float64
```

这里我们使用两个键对数据进行分组，结果的 Series 现在具有一个由观测到的唯一键对组成的分层索引：

```python
In [21]: means.unstack()
Out[21]: 
key2         1         2
key1                    
a    -0.204708  0.478943
b     1.965781 -0.555730
```

在这个例子中，组键都是 Series，尽管它们可以是任何长度合适的数组：

```python
In [22]: states = np.array(["OH", "CA", "CA", "OH", "OH", "CA", "OH"])

In [23]: years = [2005, 2005, 2006, 2005, 2006, 2005, 2006]

In [24]: df["data1"].groupby([states, years]).mean()
Out[24]: 
CA  2005    0.936175
    2006   -0.519439
OH  2005   -0.380219
    2006    1.029344
Name: data1, dtype: float64
```

通常，分组信息与您要处理的数据位于同一个 DataFrame 中。在这种情况下，您可以将列名（无论是字符串、数字还是其他 Python 对象）作为组键传递：

```python
In [25]: df.groupby("key1").mean()
Out[25]: 
          key2     data1     data2
key1                          
a      1.5  0.555881  0.441920
b      1.5  0.705025 -0.144516

In [26]: df.groupby("key2").mean(numeric_only=True)
Out[26]: 
             data1     data2
key2                    
1     0.333636  0.115218
2    -0.038393  0.888106

In [27]: df.groupby(["key1", "key2"]).mean()
Out[27]: 
                  data1     data2
key1 key2                    
a    1    -0.204708  0.281746
     2     0.478943  0.769023
b    1     1.965781 -1.296221
     2    -0.555730  1.007189
```

您可能会注意到，在第二种情况下，必须传递 `numeric_only=True`，因为 `key1` 列不是数值型的，因此无法用 `mean()` 进行聚合。

无论使用 `groupby` 的目标是什么，一个通常有用的 GroupBy 方法是 `size`，它返回一个包含组大小的 Series：

```python
In [28]: df.groupby(["key1", "key2"]).size()
Out[28]: 
key1  key2
a     1       1
      2       1
b     1       1
      2       1
dtype: int64
```

请注意，组键中的任何缺失值默认都会从结果中排除。可以通过向 `groupby` 传递 `dropna=False` 来禁用此行为：

```python
In [29]: df.groupby("key1", dropna=False).size()
Out[29]: 
key1
a      3
b      2
NaN    2
dtype: int64

In [30]: df.groupby(["key1", "key2"], dropna=False).size()
Out[30]: 
key1  key2
a     1       1
      2       1
      <NA>    1
b     1       1
      2       1
NaN   1       2
dtype: int64
```

一个在精神上与 `size` 相似的分组函数是 `count`，它计算每个组中非空值的数量：

```python
In [31]: df.groupby("key1").count()
Out[31]: 
          key2  data1  data2
key1                    
a        2      3      3
b        2      2      2
```

### 遍历分组

`groupby` 返回的对象支持迭代，生成一个包含组名和数据块的二元组序列。考虑以下代码：

```python
In [32]: for name, group in df.groupby("key1"):
       ....:     print(name)
       ....:     print(group)
       ....:
a
  key1  key2     data1     data2
0    a     1 -0.204708  0.281746
1    a     2  0.478943  0.769023
5    a  <NA>  1.393406  0.274992
b
  key1  key2     data1     data2
3    b     2 -0.555730  1.007189
4    b     1  1.965781 -1.296221
```

在多个键的情况下，元组中的第一个元素将是键值的元组：

```python
In [33]: for (k1, k2), group in df.groupby(["key1", "key2"]):
       ....:     print((k1, k2))
       ....:     print(group)
       ....:
('a', 1)
  key1  key2     data1     data2
0    a     1 -0.204708  0.281746
('a', 2)
  key1  key2     data1     data2
1    a     2  0.478943  0.769023
('b', 1)
  key1  key2     data1     data2
4    b     1  1.965781 -1.296221
('b', 2)
  key1  key2    data1     data2
3    b     2 -0.55573  1.007189
```

当然，您可以选择对数据块进行任何操作。您可能会发现一个有用的技巧是使用一行代码计算数据块的字典：

```python
In [34]: pieces = {name: group for name, group in df.groupby("key1")}

In [35]: pieces["b"]
Out[35]: 
  key1  key2     data1     data2
3    b     2 -0.555730  1.007189
4    b     1  1.965781 -1.296221
```

默认情况下，`groupby` 在 `axis="index"` 上进行分组，但您可以在任何其他轴上进行分组。例如，我们可以根据示例 `df` 中的列是否以 `"key"` 或 `"data"` 开头来对它们进行分组：

```python
In [36]: grouped = df.groupby({"key1": "key", "key2": "key",
       ....:                   "data1": "data", "data2": "data"}, axis="columns")
```

我们可以这样打印出分组：

```python
In [37]: for group_key, group_values in grouped:
       ....:     print(group_key)
       ....:     print(group_values)
       ....:
data
      data1     data2
0 -0.204708  0.281746
1  0.478943  0.769023
2 -0.519439  1.246435
3 -0.555730  1.007189
4  1.965781 -1.296221
5  1.393406  0.274992
6  0.092908  0.228913
key
   key1  key2
0     a     1
1     a     2
2  None     1
3     b     2
4     b     1
5     a  <NA>
6  None     1
```

### 选择一列或列的子集

使用列名或列名数组对从 DataFrame 创建的 GroupBy 对象进行索引，具有用于聚合的列子集选择的效果。这意味着：

```python
df.groupby("key1")["data1"]
df.groupby("key1")[["data2"]]
```

是以下操作的便捷方式：

```python
df["data1"].groupby(df["key1"])
df[["data2"]].groupby(df["key1"])
```

特别是对于大型数据集，可能只希望聚合少数几列。例如，在前面的数据集中，要仅计算 `data2` 列的平均值并将结果作为 DataFrame，我们可以这样写：

```python
In [38]: df.groupby(["key1", "key2"])[["data2"]].mean()
Out[38]: 
              data2
key1 key2          
a    1     0.281746
     2     0.769023
b    1    -1.296221
     2     1.007189
```

如果传递的是列表或数组，则此索引操作返回的对象是一个分组的 DataFrame；如果仅将单个列名作为标量传递，则返回一个分组的 Series：

```python
In [39]: s_grouped = df.groupby(["key1", "key2"])["data2"]

In [40]: s_grouped
Out[40]: <pandas.core.groupby.generic.SeriesGroupBy object at 0x17b8356c0>

In [41]: s_grouped.mean()
Out[41]: 
key1  key2
a     1       0.281746
      2       0.769023
b     1      -1.296221
      2       1.007189
Name: data2, dtype: float64
```

### 使用字典和 Series 进行分组

分组信息可能以数组以外的形式存在。让我们考虑另一个示例 DataFrame：

```python
In [42]: people = pd.DataFrame(np.random.standard_normal((5, 5)),
       ....:                   columns=["a", "b", "c", "d", "e"],
       ....:                   index=["Joe", "Steve", "Wanda", "Jill", "Trey"])

In [43]: people.iloc[2:3, [1, 2]] = np.nan # 添加一些 NA 值

In [44]: people
Out[44]: 
              a         b         c         d         e
Joe    1.352917  0.886429 -2.001637 -0.371843  1.669025
Steve -0.438570 -0.539741  0.476985  3.248944 -1.021228
Wanda -0.577087       NaN       NaN  0.523772  0.000940
Jill   1.343810 -0.713544 -0.831154 -2.370232 -1.860761
Trey  -0.860757  0.560145 -1.265934  0.119827 -1.063512
```

现在，假设我有一个列的组对应关系，并希望按组对列求和：

```python
In [45]: mapping = {"a": "red", "b": "red", "c": "blue",
       ....:        "d": "blue", "e": "red", "f" : "orange"}
```

现在，您可以从这个字典构造一个数组传递给 `groupby`，但我们可以直接传递字典（我包含了键 `"f"` 以突出显示未使用的分组键是可以的）：

```python
In [46]: by_column = people.groupby(mapping, axis="columns")

In [47]: by_column.sum()
Out[47]: 
           blue       red
Joe   -2.373480  3.908371
Steve  3.725929 -1.999539
Wanda  0.523772 -0.576147
Jill  -3.201385 -1.230495
Trey  -1.146107 -1.364125
```

同样的功能也适用于 Series，它可以被视为一个固定大小的映射：

```python
In [48]: map_series = pd.Series(mapping)

In [49]: map_series
Out[49]: 
a       red
b       red
c      blue
d      blue
e       red
f    orange
dtype: object

In [50]: people.groupby(map_series, axis="columns").count()
Out[50]: 
       blue  red
Joe       2    3
Steve     2    3
Wanda     1    2
Jill      2    3
Trey      2    3
```

### 使用函数进行分组

与字典或 Series 相比，使用 Python 函数是定义组映射的更通用方法。任何作为组键传递的函数将在每个索引值上调用一次（如果使用 `axis="columns"`，则在每个列值上调用一次），返回值将用作组名。更具体地说，考虑上一节中的示例 DataFrame，它包含人们的名字作为索引值。假设您想按名字长度分组。虽然您可以计算字符串长度的数组，但直接传递 `len` 函数更简单：

```python
In [51]: people.groupby(len).sum()
Out[51]: 
          a         b         c         d         e
3  1.352917  0.886429 -2.001637 -0.371843  1.669025
4  0.483052 -0.153399 -2.097088 -2.250405 -2.924273
5 -1.015657 -0.539741  0.476985  3.772716 -1.020287
```

将函数与数组、字典或 Series 混合使用不是问题，因为所有内容都会在内部转换为数组：

```python
In [52]: key_list = ["one", "one", "one", "two", "two"]

In [53]: people.groupby([len, key_list]).min()
Out[53]: 
              a         b         c         d         e
3 one  1.352917  0.886429 -2.001637 -0.371843  1.669025
4 two -0.860757 -0.713544 -1.265934 -2.370232 -1.860761
5 one -0.577087 -0.539741  0.476985  0.523772 -1.021228
```

### 按索引层级分组

对于分层索引的数据集，最后一个便利之处是能够使用轴索引的其中一个层级进行聚合。让我们看一个例子：

```python
In [54]: columns = pd.MultiIndex.from_arrays([["US", "US", "US", "JP", "JP"],
       ....:                                 [1, 3, 5, 1, 3]],
       ....:                                 names=["cty", "tenor"])

In [55]: hier_df = pd.DataFrame(np.random.standard_normal((4, 5)), columns=columns)

In [56]: hier_df
Out[56]: 
cty          US                            JP          
tenor         1         3         5         1         3
0      0.332883 -2.359419 -0.199543 -1.541996 -0.970736
1     -1.307030  0.286350  0.377984 -0.753887  0.331286
2      1.349742  0.069877  0.246674 -0.011862  1.004812
3      1.327195 -0.919262 -1.549106  0.022185  0.758363
```

要按层级分组，请使用 `level` 关键字传递层级编号或名称：

```python
In [57]: hier_df.groupby(level="cty", axis="columns").count()
Out[57]: 
cty  JP  US
0     2   3
1     2   3
2     2   3
3     2   3
```

## 10.2 数据聚合

_聚合_(aggregations) 是指从数组生成标量值的任何数据转换过程。之前的示例已经使用了多种聚合操作，包括 `mean`、`count`、`min` 和 `sum`。你可能想知道在 GroupBy 对象上调用 `mean()` 时会发生什么。许多常见的聚合操作（如表 10.1 所列）都拥有优化实现。不过，你并不局限于这组方法。

表 10.1：优化的 `groupby` 方法

| 函数名          | 描述                                                                 |
|-----------------|----------------------------------------------------------------------|
| `any, all`      | 如果任何（一个或多个值）或所有非 NA 值为"真"则返回 `True`                |
| `count`         | 非 NA 值的数量                                                       |
| `cummin, cummax`| 非 NA 值的累积最小值和最大值                                          |
| `cumsum`        | 非 NA 值的累积和                                                     |
| `cumprod`       | 非 NA 值的累积积                                                     |
| `first, last`   | 第一个和最后一个非 NA 值                                              |
| `mean`          | 非 NA 值的平均值                                                     |
| `median`        | 非 NA 值的算术中位数                                                 |
| `min, max`      | 非 NA 值的最小值和最大值                                              |
| `nth`           | 获取按排序顺序排在第 `n` 位的值                                       |
| `ohlc`          | 为时间序列类数据计算四个"开盘-最高-最低-收盘"统计量                    |
| `prod`          | 非 NA 值的乘积                                                       |
| `quantile`      | 计算样本分位数                                                       |
| `rank`          | 非 NA 值的序数排名（类似于调用 `Series.rank`）                         |
| `size`          | 计算分组大小，以 Series 形式返回结果                                  |
| `sum`           | 非 NA 值的总和                                                       |
| `std, var`      | 样本标准差和方差                                                     |

你可以使用自己设计的聚合函数，也可以调用分组对象上已定义的任何方法。例如，`nsmallest` Series 方法可以从数据中选取指定数量的最小值。虽然 `nsmallest` 没有为 GroupBy 显式实现，但我们仍可通过非优化方式使用它。在内部，GroupBy 会对 Series 进行切片，对每个切片调用 `piece.nsmallest(n)`，然后将这些结果组装成最终结果对象：

```python
In [58]: df
Out[58]: 
       key1  key2     data1     data2
    0     a     1 -0.204708  0.281746
    1     a     2  0.478943  0.769023
    2  None     1 -0.519439  1.246435
    3     b     2 -0.555730  1.007189
    4     b     1  1.965781 -1.296221
    5     a  <NA>  1.393406  0.274992
    6  None     1  0.092908  0.228913

In [59]: grouped = df.groupby("key1")

In [60]: grouped["data1"].nsmallest(2)
Out[60]: 
key1   
a     0   -0.204708
      1    0.478943
b     3   -0.555730
      4    1.965781
Name: data1, dtype: float64
```

要使用自定义聚合函数，只需将能处理数组的任意函数传递给 `aggregate` 方法或其简称 `agg`：

```python
In [61]: def peak_to_peak(arr):
   ....:     return arr.max() - arr.min()

In [62]: grouped.agg(peak_to_peak)
Out[62]: 
      key2     data1     data2
key1                          
a        1  1.598113  0.494031
b        1  2.521511  2.303410
```

你可能会注意到，某些方法（如 `describe`）同样有效，尽管严格来说它们并非聚合操作：

```python
In [63]: grouped.describe()
Out[63]: 
      key2                                           data1            ...   
     count mean       std  min   25%  50%   75%  max count      mean  ...   
key1                                                                  ...   
a      2.0  1.5  0.707107  1.0  1.25  1.5  1.75  2.0   3.0  0.555881  ... \
b      2.0  1.5  0.707107  1.0  1.25  1.5  1.75  2.0   2.0  0.705025  ...   
                         data2                                           
           75%       max count      mean       std       min       25%   
key1                                                                     
a     0.936175  1.393406   3.0  0.441920  0.283299  0.274992  0.278369 \
b     1.335403  1.965781   2.0 -0.144516  1.628757 -1.296221 -0.720368   
                                        
           50%       75%       max  
key1                                
a     0.281746  0.525384  0.769023  
b    -0.144516  0.431337  1.007189  
[2 rows x 24 columns]
```

我将在"应用：通用的拆分-应用-合并"中详细解释这里发生的过程。

注意

自定义聚合函数通常比表 10.1 中的优化函数慢得多，这是因为在构建中间分组数据块时存在额外开销（函数调用、数据重排）。

### 按列应用多个函数

让我们回到上一章使用的小费数据集。通过 `pandas.read_csv` 加载后，我们添加一个小费百分比列：

```python
In [64]: tips = pd.read_csv("examples/tips.csv")

In [65]: tips.head()
Out[65]: 
   total_bill   tip smoker  day    time  size
0       16.99  1.01     No  Sun  Dinner     2
1       10.34  1.66     No  Sun  Dinner     3
2       21.01  3.50     No  Sun  Dinner     3
3       23.68  3.31     No  Sun  Dinner     2
4       24.59  3.61     No  Sun  Dinner     4
```

现在添加一个 `tip_pct` 列，表示小费占总账单的百分比：

```python
In [66]: tips["tip_pct"] = tips["tip"] / tips["total_bill"]

In [67]: tips.head()
Out[67]: 
   total_bill   tip smoker  day    time  size   tip_pct
0       16.99  1.01     No  Sun  Dinner     2  0.059447
1       10.34  1.66     No  Sun  Dinner     3  0.160542
2       21.01  3.50     No  Sun  Dinner     3  0.166587
3       23.68  3.31     No  Sun  Dinner     2  0.139780
4       24.59  3.61     No  Sun  Dinner     4  0.146808
```

如你所见，对 Series 或 DataFrame 的所有列进行聚合时，只需使用 `aggregate`（或 `agg`）并传入所需函数，或调用如 `mean` 或 `std` 之类的方法。但有时你可能希望根据不同列应用不同的函数，或同时应用多个函数。幸运的是，这是可以实现的，我将通过几个示例来说明。首先，按 `day` 和 `smoker` 对 `tips` 进行分组：

```python
In [68]: grouped = tips.groupby(["day", "smoker"])
```

注意，对于表 10.1 中的描述性统计函数，你可以将函数名作为字符串传递：

```python
In [69]: grouped_pct = grouped["tip_pct"]

In [70]: grouped_pct.agg("mean")
Out[70]: 
day   smoker
Fri   No        0.151650
      Yes       0.174783
Sat   No        0.158048
      Yes       0.147906
Sun   No        0.160113
      Yes       0.187250
Thur  No        0.160298
      Yes       0.163863
Name: tip_pct, dtype: float64
```

如果传入函数或函数名的列表，将返回一个 DataFrame，其列名来自这些函数：

```python
In [71]: grouped_pct.agg(["mean", "std", peak_to_peak])
Out[71]: 
                 mean       std  peak_to_peak
day  smoker                                  
Fri  No      0.151650  0.028123      0.067349
     Yes     0.174783  0.051293      0.159925
Sat  No      0.158048  0.039767      0.235193
     Yes     0.147906  0.061375      0.290095
Sun  No      0.160113  0.042347      0.193226
     Yes     0.187250  0.154134      0.644685
Thur No      0.160298  0.038774      0.193350
     Yes     0.163863  0.039389      0.151240
```

这里我们向 `agg` 传递了一个聚合函数列表，以便在数据组上独立求值。

你并不一定要接受 GroupBy 自动生成的列名；特别是 `lambda` 函数会使用 `"<lambda>"` 这样的名称，难以识别（可通过查看函数的 `__name__` 属性确认）。因此，如果传递一个 `(名称, 函数)` 元组列表，每个元组的第一个元素将用作 DataFrame 的列名（你可以将二元元组列表视为有序映射）：

```python
In [72]: grouped_pct.agg([("average", "mean"), ("stdev", np.std)])
Out[72]: 
                 average     stdev
day  smoker                    
Fri  No      0.151650  0.028123
     Yes     0.174783  0.051293
Sat  No      0.158048  0.039767
     Yes     0.147906  0.061375
Sun  No      0.160113  0.042347
     Yes     0.187250  0.154134
Thur No      0.160298  0.038774
     Yes     0.163863  0.039389
```

对于 DataFrame，你有更多选择，可以指定应用于所有列的函数列表，或为不同列指定不同函数。首先，假设我们想对 `tip_pct` 和 `total_bill` 列计算相同的三个统计量：

```python
In [73]: functions = ["count", "mean", "max"]

In [74]: result = grouped[["tip_pct", "total_bill"]].agg(functions)

In [75]: result
Out[75]: 
               tip_pct                     total_bill                  
                 count      mean       max      count       mean    max
day  smoker                                                         
Fri  No           4  0.151650  0.187735          4  18.420000  22.75
     Yes         15  0.174783  0.263480         15  16.813333  40.17
Sat  No          45  0.158048  0.291990         45  19.661778  48.33
     Yes         42  0.147906  0.325733         42  21.276667  50.81
Sun  No          57  0.160113  0.252672         57  20.506667  48.17
     Yes         19  0.187250  0.710345         19  24.120000  45.35
Thur No          45  0.160298  0.266312         45  17.113111  41.19
     Yes         17  0.163863  0.241255         17  19.190588  43.11
```

如你所见，结果 DataFrame 拥有分层列索引，与分别聚合每列后再使用 `concat` 函数并以列名作为 `keys` 参数拼接的结果相同：

```python
In [76]: result["tip_pct"]
Out[76]: 
                count      mean       max
day  smoker                           
Fri  No          4  0.151650  0.187735
     Yes        15  0.174783  0.263480
Sat  No         45  0.158048  0.291990
     Yes        42  0.147906  0.325733
Sun  No         57  0.160113  0.252672
     Yes        19  0.187250  0.710345
Thur No         45  0.160298  0.266312
     Yes        17  0.163863  0.241255
```

如前所述，也可以传递包含自定义名称的元组列表：

```python
In [77]: ftuples = [("Average", "mean"), ("Variance", np.var)]

In [78]: grouped[["tip_pct", "total_bill"]].agg(ftuples)
Out[78]: 
                 tip_pct           total_bill            
                 Average  Variance    Average    Variance
day  smoker                                           
Fri  No      0.151650  0.000791  18.420000   25.596333
     Yes     0.174783  0.002631  16.813333   82.562438
Sat  No      0.158048  0.001581  19.661778   79.908965
     Yes     0.147906  0.003767  21.276667  101.387535
Sun  No      0.160113  0.001793  20.506667   66.099980
     Yes     0.187250  0.023757  24.120000  109.046044
Thur No      0.160298  0.001503  17.113111   59.625081
     Yes     0.163863  0.001551  19.190588   69.808518
```

现在，假设你想对一列或多列应用不同的函数。为此，可以向 `agg` 传递一个字典，其中包含从列名到前述任何函数规范的映射：

```python
In [79]: grouped.agg({"tip" : np.max, "size" : "sum"})
Out[79]: 
                tip  size
day  smoker             
Fri  No       3.50     9
     Yes      4.73    31
Sat  No       9.00   115
     Yes     10.00   104
Sun  No       6.00   167
     Yes      6.50    49
Thur No       6.70   112
     Yes      5.00    40

In [80]: grouped.agg({"tip_pct" : ["min", "max", "mean", "std"],
   ....:              "size" : "sum"})
Out[80]: 
                 tip_pct                               size
                     min       max      mean       std  sum
day  smoker                                             
Fri  No      0.120385  0.187735  0.151650  0.028123    9
     Yes     0.103555  0.263480  0.174783  0.051293   31
Sat  No      0.056797  0.291990  0.158048  0.039767  115
     Yes     0.035638  0.325733  0.147906  0.061375  104
Sun  No      0.059447  0.252672  0.160113  0.042347  167
     Yes     0.065660  0.710345  0.187250  0.154134   49
Thur No      0.072961  0.266312  0.160298  0.038774  112
     Yes     0.090014  0.241255  0.163863  0.039389   40
```

只有当对至少一列应用了多个函数时，DataFrame 才会拥有分层列索引。

### 返回无行索引的聚合数据

到目前为止的所有示例中，聚合数据返回时都带有一个索引（可能是分层的），由唯一的分组键组合构成。由于这并不总是需要的，在大多数情况下可以通过向 `groupby` 传递 `as_index=False` 来禁用此行为：

```python
In [81]: grouped = tips.groupby(["day", "smoker"], as_index=False)

In [82]: grouped.mean(numeric_only=True)
Out[82]: 
    day smoker  total_bill       tip      size   tip_pct
0   Fri     No   18.420000  2.812500  2.250000  0.151650
1   Fri    Yes   16.813333  2.714000  2.066667  0.174783
2   Sat     No   19.661778  3.102889  2.555556  0.158048
3   Sat    Yes   21.276667  2.875476  2.476190  0.147906
4   Sun     No   20.506667  3.167895  2.929825  0.160113
5   Sun    Yes   24.120000  3.516842  2.578947  0.187250
6  Thur     No   17.113111  2.673778  2.488889  0.160298
7  Thur    Yes   19.190588  3.030000  2.352941  0.163863
```

当然，始终可以通过对结果调用 `reset_index` 来获得此格式的数据。使用 `as_index=False` 参数可以避免一些不必要的计算。

## 10.3 Apply：通用的拆分-应用-合并（General split-apply-combine）

最通用的 GroupBy 方法是 `apply`，这也是本节的主题。`apply` 将被操作对象拆分成多个片段，在每个片段上调用传入的函数，然后尝试将这些片段拼接起来。

回到之前的小费数据集，假设您希望按组选出 `tip_pct` 值最大的前 5 个。首先，编写一个函数，用于选择指定列中具有最大值的行：

```python
In [83]: def top(df, n=5, column="tip_pct"):
   ....:     return df.sort_values(column, ascending=False)[:n]

In [84]: top(tips, n=6)
Out[84]: 
     total_bill   tip smoker  day    time  size   tip_pct
172        7.25  5.15    Yes  Sun  Dinner     2  0.710345
178        9.60  4.00    Yes  Sun  Dinner     2  0.416667
67         3.07  1.00    Yes  Sat  Dinner     1  0.325733
232       11.61  3.39     No  Sat  Dinner     2  0.291990
183       23.17  6.50    Yes  Sun  Dinner     4  0.280535
109       14.31  4.00    Yes  Sat  Dinner     2  0.279525
```

现在，如果我们按 `smoker` 分组，并对此分组对象调用 `apply` 和该函数，会得到以下结果：

```python
In [85]: tips.groupby("smoker").apply(top)
Out[85]: 
            total_bill   tip smoker   day    time  size   tip_pct
smoker                                                           
No     232       11.61  3.39     No   Sat  Dinner     2  0.291990
       149        7.51  2.00     No  Thur   Lunch     2  0.266312
       51        10.29  2.60     No   Sun  Dinner     2  0.252672
       185       20.69  5.00     No   Sun  Dinner     5  0.241663
       88        24.71  5.85     No  Thur   Lunch     2  0.236746
Yes    172        7.25  5.15    Yes   Sun  Dinner     2  0.710345
       178        9.60  4.00    Yes   Sun  Dinner     2  0.416667
       67         3.07  1.00    Yes   Sat  Dinner     1  0.325733
       183       23.17  6.50    Yes   Sun  Dinner     4  0.280535
       109       14.31  4.00    Yes   Sat  Dinner     2  0.279525
```

这里发生了什么？首先，`tips` DataFrame 根据 `smoker` 的值被拆分成多个组。然后，`top` 函数在每个分组上被调用，每个函数调用的结果使用 `pandas.concat` 拼接在一起，并用组名标记每个片段。因此，结果具有一个分层索引，其内层包含来自原始 DataFrame 的索引值。

如果您传递给 `apply` 的函数接受其他参数或关键字，可以在函数之后传递这些参数：

```python
In [86]: tips.groupby(["smoker", "day"]).apply(top, n=1, column="total_bill")
Out[86]: 
                 total_bill    tip smoker   day    time  size   tip_pct
smoker day                                                             
No     Fri  94        22.75   3.25     No   Fri  Dinner     2  0.142857
       Sat  212       48.33   9.00     No   Sat  Dinner     4  0.186220
       Sun  156       48.17   5.00     No   Sun  Dinner     6  0.103799
       Thur 142       41.19   5.00     No  Thur   Lunch     5  0.121389
Yes    Fri  95        40.17   4.73    Yes   Fri  Dinner     4  0.117750
       Sat  170       50.81  10.00    Yes   Sat  Dinner     3  0.196812
       Sun  182       45.35   3.50    Yes   Sun  Dinner     3  0.077178
       Thur 197       43.11   5.00    Yes  Thur   Lunch     4  0.115982
```

除了这些基本用法机制外，充分利用 `apply` 可能需要一些创造力。传入函数内部发生的一切取决于您；它必须返回一个 pandas 对象或标量值。本章的其余部分将主要包含示例，向您展示如何使用 `groupby` 解决各种问题。

例如，您可能记得我之前在 GroupBy 对象上调用了 `describe`：

```python
In [87]: result = tips.groupby("smoker")["tip_pct"].describe()

In [88]: result
Out[88]: 
        count      mean       std       min       25%       50%       75%   
smoker                                                                      
No      151.0  0.159328  0.039910  0.056797  0.136906  0.155625  0.185014  \
Yes      93.0  0.163196  0.085119  0.035638  0.106771  0.153846  0.195059   
             max  
smoker            
No      0.291990  
Yes     0.710345  

In [89]: result.unstack("smoker")
Out[89]: 
       smoker
count  No        151.000000
       Yes        93.000000
mean   No          0.159328
       Yes         0.163196
std    No          0.039910
       Yes         0.085119
min    No          0.056797
       Yes         0.035638
25%    No          0.136906
       Yes         0.106771
50%    No          0.155625
       Yes         0.153846
75%    No          0.185014
       Yes         0.195059
max    No          0.291990
       Yes         0.710345
dtype: float64
```

在 GroupBy 内部，当您调用像 `describe` 这样的方法时，它实际上只是以下代码的快捷方式：

```python
def f(group):
    return group.describe()

grouped.apply(f)
```

### 抑制组键（Suppressing the Group Keys）

在前面的示例中，您看到结果对象具有一个由组键和原始对象每个片段的索引组成的分层索引。您可以通过向 `groupby` 传递 `group_keys=False` 来禁用此功能：

```python
In [90]: tips.groupby("smoker", group_keys=False).apply(top)
Out[90]: 
     total_bill   tip smoker   day    time  size   tip_pct
232       11.61  3.39     No   Sat  Dinner     2  0.291990
149        7.51  2.00     No  Thur   Lunch     2  0.266312
51        10.29  2.60     No   Sun  Dinner     2  0.252672
185       20.69  5.00     No   Sun  Dinner     5  0.241663
88        24.71  5.85     No  Thur   Lunch     2  0.236746
172        7.25  5.15    Yes   Sun  Dinner     2  0.710345
178        9.60  4.00    Yes   Sun  Dinner     2  0.416667
67         3.07  1.00    Yes   Sat  Dinner     1  0.325733
183       23.17  6.50    Yes   Sun  Dinner     4  0.280535
109       14.31  4.00    Yes   Sat  Dinner     2  0.279525
```

### 分位数和分桶分析（Quantile and Bucket Analysis）

您可能还记得[第 8 章：数据整理：连接、合并和重塑](https://wesmckinney.com/book/data-wrangling)，pandas 有一些工具，特别是 `pandas.cut` 和 `pandas.qcut`，用于将数据按照您选择的分箱或样本分位数切片到不同的桶中。将这些函数与 `groupby` 结合使用，可以方便地对数据集进行分桶或分位数分析。考虑一个简单的随机数据集，并使用 `pandas.cut` 进行等长分桶分类：

```python
In [91]: frame = pd.DataFrame({"data1": np.random.standard_normal(1000),
   ....:                       "data2": np.random.standard_normal(1000)})

In [92]: frame.head()
Out[92]: 
      data1     data2
0 -0.660524 -0.612905
1  0.862580  0.316447
2 -0.010032  0.838295
3  0.050009 -1.034423
4  0.670216  0.434304

In [93]: quartiles = pd.cut(frame["data1"], 4)

In [94]: quartiles.head(10)
Out[94]: 
0     (-1.23, 0.489]
1     (0.489, 2.208]
2     (-1.23, 0.489]
3     (-1.23, 0.489]
4     (0.489, 2.208]
5     (0.489, 2.208]
6     (-1.23, 0.489]
7     (-1.23, 0.489]
8    (-2.956, -1.23]
9     (-1.23, 0.489]
Name: data1, dtype: category
Categories (4, interval[float64, right]): [(-2.956, -1.23] < (-1.23, 0.489] < (0.
489, 2.208] <
                                           (2.208, 3.928]]
```

`cut` 返回的 `Categorical` 对象可以直接传递给 `groupby`。因此，我们可以为四分位数计算一组分组统计量，如下所示：

```python
In [95]: def get_stats(group):
   ....:     return pd.DataFrame(
   ....:         {"min": group.min(), "max": group.max(),
   ....:         "count": group.count(), "mean": group.mean()}
   ....:     )

In [96]: grouped = frame.groupby(quartiles)

In [97]: grouped.apply(get_stats)
Out[97]: 
                            min       max  count      mean
data1                                                     
(-2.956, -1.23] data1 -2.949343 -1.230179     94 -1.658818
                data2 -3.399312  1.670835     94 -0.033333
(-1.23, 0.489]  data1 -1.228918  0.488675    598 -0.329524
                data2 -2.989741  3.260383    598 -0.002622
(0.489, 2.208]  data1  0.489965  2.200997    298  1.065727
                data2 -3.745356  2.954439    298  0.078249
(2.208, 3.928]  data1  2.212303  3.927528     10  2.644253
                data2 -1.929776  1.765640     10  0.024750
```

请记住，同样的结果可以更简单地用以下方式计算：

```python
In [98]: grouped.agg(["min", "max", "count", "mean"])
Out[98]: 
                    data1                               data2                   
                      min       max count      mean       min       max count   
data1                                                                           
(-2.956, -1.23] -2.949343 -1.230179    94 -1.658818 -3.399312  1.670835    94  \
(-1.23, 0.489]  -1.228918  0.488675   598 -0.329524 -2.989741  3.260383   598   
(0.489, 2.208]   0.489965  2.200997   298  1.065727 -3.745356  2.954439   298   
(2.208, 3.928]   2.212303  3.927528    10  2.644253 -1.929776  1.765640    10   
                           
                       mean  
data1                      
(-2.956, -1.23] -0.033333  
(-1.23, 0.489]  -0.002622  
(0.489, 2.208]   0.078249  
(2.208, 3.928]   0.024750
```

这些是等长分桶；要基于样本分位数计算等大小的分桶，请使用 `pandas.qcut`。我们可以传递 `4` 作为分桶数量以计算样本四分位数，并传递 `labels=False` 以仅获取四分位数索引而不是区间：

```python
In [99]: quartiles_samp = pd.qcut(frame["data1"], 4, labels=False)

In [100]: quartiles_samp.head()
Out[100]: 
0    1
1    3
2    2
3    2
4    3
Name: data1, dtype: int64

In [101]: grouped = frame.groupby(quartiles_samp)

In [102]: grouped.apply(get_stats)
Out[102]: 
                  min       max  count      mean
data1                                           
0     data1 -2.949343 -0.685484    250 -1.212173
      data2 -3.399312  2.628441    250 -0.027045
1     data1 -0.683066 -0.030280    250 -0.368334
      data2 -2.630247  3.260383    250 -0.027845
2     data1 -0.027734  0.618965    250  0.295812
      data2 -3.056990  2.458842    250  0.014450
3     data1  0.623587  3.927528    250  1.248875
      data2 -3.745356  2.954439    250  0.115899
```

### 示例：使用组特定值填充缺失值（Example: Filling Missing Values with Group-Specific Values）

在清理缺失数据时，有时您会使用 `dropna` 删除数据观测值，但在其他情况下，您可能希望使用固定值或从数据派生的值来填充空值（NA）。`fillna` 是合适的工具；例如，这里我用均值填充空值：

```python
In [103]: s = pd.Series(np.random.standard_normal(6))

In [104]: s[::2] = np.nan

In [105]: s
Out[105]: 
0         NaN
1    0.227290
2         NaN
3   -2.153545
4         NaN
5   -0.375842
dtype: float64

In [106]: s.fillna(s.mean())
Out[106]: 
0   -0.767366
1    0.227290
2   -0.767366
3   -2.153545
4   -0.767366
5   -0.375842
dtype: float64
```

假设您需要填充值随组变化。一种方法是对数据进行分组，并使用 `apply` 和一个在每个数据块上调用 `fillna` 的函数。以下是一些关于美国州的示例数据，分为东部和西部区域：

```python
In [107]: states = ["Ohio", "New York", "Vermont", "Florida",
   .....:           "Oregon", "Nevada", "California", "Idaho"]

In [108]: group_key = ["East", "East", "East", "East",
   .....:              "West", "West", "West", "West"]

In [109]: data = pd.Series(np.random.standard_normal(8), index=states)

In [110]: data
Out[110]: 
Ohio          0.329939
New York      0.981994
Vermont       1.105913
Florida      -1.613716
Oregon        1.561587
Nevada        0.406510
California    0.359244
Idaho        -0.614436
dtype: float64
```

让我们将数据中的一些值设置为缺失：

```python
In [111]: data[["Vermont", "Nevada", "Idaho"]] = np.nan

In [112]: data
Out[112]: 
Ohio          0.329939
New York      0.981994
Vermont            NaN
Florida      -1.613716
Oregon        1.561587
Nevada             NaN
California    0.359244
Idaho              NaN
dtype: float64

In [113]: data.groupby(group_key).size()
Out[113]: 
East    4
West    4
dtype: int64

In [114]: data.groupby(group_key).count()
Out[114]: 
East    3
West    2
dtype: int64

In [115]: data.groupby(group_key).mean()
Out[115]: 
East   -0.100594
West    0.960416
dtype: float64
```

我们可以使用组均值填充 NA 值，如下所示：

```python
In [116]: def fill_mean(group):
   .....:     return group.fillna(group.mean())

In [117]: data.groupby(group_key).apply(fill_mean)
Out[117]: 
East  Ohio          0.329939
      New York      0.981994
      Vermont      -0.100594
      Florida      -1.613716
West  Oregon        1.561587
      Nevada        0.960416
      California    0.359244
      Idaho         0.960416
dtype: float64
```

在另一种情况下，您可能在代码中预定义了随组变化的填充值。由于组在内部设置了 `name` 属性，我们可以使用它：

```python
In [118]: fill_values = {"East": 0.5, "West": -1}

In [119]: def fill_func(group):
   .....:     return group.fillna(fill_values[group.name])

In [120]: data.groupby(group_key).apply(fill_func)
Out[120]: 
East  Ohio          0.329939
      New York      0.981994
      Vermont       0.500000
      Florida      -1.613716
West  Oregon        1.561587
      Nevada       -1.000000
      California    0.359244
      Idaho        -1.000000
dtype: float64
```

### 示例：随机抽样和排列（Example: Random Sampling and Permutation）

假设您想从大型数据集中随机抽样（有放回或无放回）以进行蒙特卡洛模拟或其他应用。有多种方法可以执行“抽取”；这里我们使用 Series 的 `sample` 方法。

为了演示，以下是构建一副英式风格扑克牌的方法：

```python
suits = ["H", "S", "C", "D"]  # 红心、黑桃、梅花、方块
card_val = (list(range(1, 11)) + [10] * 3) * 4
base_names = ["A"] + list(range(2, 11)) + ["J", "K", "Q"]
cards = []
for suit in suits:
    cards.extend(str(num) + suit for num in base_names)

deck = pd.Series(card_val, index=cards)
```

现在我们有一个长度为 52 的 Series，其索引包含卡片名称，值是在二十一点和其他游戏中使用的值（为简单起见，我将王牌 `"A"` 设为 1）：

```python
In [122]: deck.head(13)
Out[122]: 
AH      1
2H      2
3H      3
4H      4
5H      5
6H      6
7H      7
8H      8
9H      9
10H    10
JH     10
KH     10
QH     10
dtype: int64
```

现在，根据我之前所说的，从牌堆中抽取五张牌可以写成：

```python
In [123]: def draw(deck, n=5):
   .....:     return deck.sample(n)

In [124]: draw(deck)
Out[124]: 
4D     4
QH    10
8S     8
7D     7
9C     9
dtype: int64
```

假设您想从每种花色中抽取两张随机牌。由于花色是每张牌名的最后一个字符，我们可以基于此进行分组并使用 `apply`：

```python
In [125]: def get_suit(card):
   .....:     # 最后一个字母是花色
   .....:     return card[-1]

In [126]: deck.groupby(get_suit).apply(draw, n=2)
Out[126]: 
C  6C     6
   KC    10
D  7D     7
   3D     3
H  7H     7
   9H     9
S  2S     2
   QS    10
dtype: int64
```

或者，我们可以传递 `group_keys=False` 以删除外部花色索引，仅保留选中的牌：

```python
In [127]: deck.groupby(get_suit, group_keys=False).apply(draw, n=2)
Out[127]: 
AC      1
3C      3
5D      5
4D      4
10H    10
7H      7
QS     10
7S      7
dtype: int64
```

### 示例：组加权平均和相关性（Example: Group Weighted Average and Correlation）

在 `groupby` 的拆分-应用-合并范式下，DataFrame 中列之间或两个 Series 之间的操作是可能的，例如组加权平均。例如，取这个包含组键、值和一些权重的数据集：

```python
In [128]: df = pd.DataFrame({"category": ["a", "a", "a", "a",
   .....:                                 "b", "b", "b", "b"],
   .....:                    "data": np.random.standard_normal(8),
   .....:                    "weights": np.random.uniform(size=8)})

In [129]: df
Out[129]: 
  category      data   weights
0        a -1.691656  0.955905
1        a  0.511622  0.012745
2        a -0.401675  0.137009
3        a  0.968578  0.763037
4        b -1.818215  0.492472
5        b  0.279963  0.832908
6        b -0.200819  0.658331
7        b -0.217221  0.612009
```

按 `category` 的加权平均将是：

```python
In [130]: grouped = df.groupby("category")

In [131]: def get_wavg(group):
   .....:     return np.average(group["data"], weights=group["weights"])

In [132]: grouped.apply(get_wavg)
Out[132]: 
category
a   -0.495807
b   -0.357273
dtype: float64
```

另一个例子，考虑一个最初从 Yahoo! Finance 获取的金融数据集，包含几只股票和标普 500 指数（`SPX` 符号）的每日收盘价：

```python
In [133]: close_px = pd.read_csv("examples/stock_px.csv", parse_dates=True,
   .....:                        index_col=0)

In [134]: close_px.info()
<class 'pandas.core.frame.DataFrame'>
DatetimeIndex: 2214 entries, 2003-01-02 to 2011-10-14
Data columns (total 4 columns):
 #   Column  Non-Null Count  Dtype  
---  ------  --------------  -----  
 0   AAPL    2214 non-null   float64
 1   MSFT    2214 non-null   float64
 2   XOM     2214 non-null   float64
 3   SPX     2214 non-null   float64
dtypes: float64(4)
memory usage: 86.5 KB

In [135]: close_px.tail(4)
Out[135]: 
              AAPL   MSFT    XOM      SPX
2011-10-11  400.29  27.00  76.27  1195.54
2011-10-12  402.19  26.96  77.16  1207.25
2011-10-13  408.43  27.18  76.37  1203.66
2011-10-14  422.00  27.27  78.11  1224.58
```

DataFrame 的 `info()` 方法是获取 DataFrame 内容概览的便捷方式。

一个感兴趣的任务可能是计算一个由每日收益率（通过百分比变化计算）与 `SPX` 的年度相关性组成的 DataFrame。实现此目的的一种方法是，首先创建一个函数，计算每列与 `"SPX"` 列的成对相关性：

```python
In [136]: def spx_corr(group):
   .....:     return group.corrwith(group["SPX"])
```

接下来，我们使用 `pct_change` 计算 `close_px` 的百分比变化：

```python
In [137]: rets = close_px.pct_change().dropna()
```

最后，我们按年对这些百分比变化进行分组，可以通过一个单行函数从每个行标签中提取 `datetime` 标签的 `year` 属性：

```python
In [138]: def get_year(x):
   .....:     return x.year

In [139]: by_year = rets.groupby(get_year)

In [140]: by_year.apply(spx_corr)
Out[140]: 
          AAPL      MSFT       XOM  SPX
2003  0.541124  0.745174  0.661265  1.0
2004  0.374283  0.588531  0.557742  1.0
2005  0.467540  0.562374  0.631010  1.0
2006  0.428267  0.406126  0.518514  1.0
2007  0.508118  0.658770  0.786264  1.0
2008  0.681434  0.804626  0.828303  1.0
2009  0.707103  0.654902  0.797921  1.0
2010  0.710105  0.730118  0.839057  1.0
2011  0.691931  0.800996  0.859975  1.0
```

您也可以计算列间的相关性。这里我们计算苹果和微软的年度相关性：

```python
In [141]: def corr_aapl_msft(group):
   .....:     return group["AAPL"].corr(group["MSFT"])

In [142]: by_year.apply(corr_aapl_msft)
Out[142]: 
2003    0.480868
2004    0.259024
2005    0.300093
2006    0.161735
2007    0.417738
2008    0.611901
2009    0.432738
2010    0.571946
2011    0.581987
dtype: float64
```

### 示例：组内线性回归（Example: Group-Wise Linear Regression）

与前面的示例相同，您可以使用 `groupby` 执行更复杂的组内统计分析，只要函数返回 pandas 对象或标量值。例如，我可以定义以下 `regress` 函数（使用 `statsmodels` 计量经济学库），该函数对每个数据块执行普通最小二乘（OLS）回归：

```python
import statsmodels.api as sm
def regress(data, yvar=None, xvars=None):
    Y = data[yvar]
    X = data[xvars]
    X["intercept"] = 1.
    result = sm.OLS(Y, X).fit()
    return result.params
```

如果您还没有安装 `statsmodels`，可以使用 conda 安装：

```bash
conda install statsmodels
```

现在，要运行 `AAPL` 对 `SPX` 收益的年度线性回归，执行：

```python
In [144]: by_year.apply(regress, yvar="AAPL", xvars=["SPX"])
Out[144]: 
           SPX  intercept
2003  1.195406   0.000710
2004  1.363463   0.004201
2005  1.766415   0.003246
2006  1.645496   0.000080
2007  1.198761   0.003438
2008  0.968016  -0.001110
2009  0.879103   0.002954
2010  1.052608   0.001261
2011  0.806605   0.001514
```

## 10.4 分组转换与"未包装的"分组操作（Unwrapped GroupBys）

在 Apply：通用拆分-应用-合并 章节中，我们探讨了分组操作中用于执行转换的 `apply` 方法。实际上还存在另一个内置方法 `transform`，其功能与 `apply` 类似，但对函数类型有更多限制：

  * 可生成标量值，并将其广播（broadcast）至分组形状
  * 可生成与输入分组形状相同的对象
  * 禁止对输入数据进行原地修改

通过一个简单示例进行说明：
    
    
    In [145]: df = pd.DataFrame({'key': ['a', 'b', 'c'] * 4,
       .....:                    'value': np.arange(12.)})
    
    In [146]: df
    Out[146]: 
       key  value
    0    a    0.0
    1    b    1.0
    2    c    2.0
    3    a    3.0
    4    b    4.0
    5    c    5.0
    6    a    6.0
    7    b    7.0
    8    c    8.0
    9    a    9.0
    10   b   10.0
    11   c   11.0 __

按key分组计算均值：
    
    
    In [147]: g = df.groupby('key')['value']
    
    In [148]: g.mean()
    Out[148]: 
    key
    a    4.5
    b    5.5
    c    6.5
    Name: value, dtype: float64 __

若需要生成与 `df['value']` 形状相同但值为按`'key'`分组均值的Series，可以向`transform`传递计算单组均值的函数：
    
    
    In [149]: def get_mean(group):
       .....:     return group.mean()
    
    In [150]: g.transform(get_mean)
    Out[150]: 
    0     4.5
    1     5.5
    2     6.5
    3     4.5
    4     5.5
    5     6.5
    6     4.5
    7     5.5
    8     6.5
    9     4.5
    10    5.5
    11    6.5
    Name: value, dtype: float64 __

对于内置聚合函数，可以像GroupBy的`agg`方法那样传递字符串别名：
    
    
    In [151]: g.transform('mean')
    Out[151]: 
    0     4.5
    1     5.5
    2     6.5
    3     4.5
    4     5.5
    5     6.5
    6     4.5
    7     5.5
    8     6.5
    9     4.5
    10    5.5
    11    6.5
    Name: value, dtype: float64 __

与`apply`类似，`transform`也支持返回Series的函数，但结果必须与输入尺寸相同。例如使用辅助函数将每组乘以2：
    
    
    In [152]: def times_two(group):
       .....:     return group * 2
    
    In [153]: g.transform(times_two)
    Out[153]: 
    0      0.0
    1      2.0
    2      4.0
    3      6.0
    4      8.0
    5     10.0
    6     12.0
    7     14.0
    8     16.0
    9     18.0
    10    20.0
    11    22.0
    Name: value, dtype: float64 __

更复杂的例子：计算每组内的降序排名：
    
    
    In [154]: def get_ranks(group):
       .....:     return group.rank(ascending=False)
    
    In [155]: g.transform(get_ranks)
    Out[155]: 
    0     4.0
    1     4.0
    2     4.0
    3     3.0
    4     3.0
    5     3.0
    6     2.0
    7     2.0
    8     2.0
    9     1.0
    10    1.0
    11    1.0
    Name: value, dtype: float64 __

考虑由简单聚合组成的组转换函数：
    
    
    In [156]: def normalize(x):
       .....:     return (x - x.mean()) / x.std()__

此案例中使用`transform`或`apply`可获得等效结果：
    
    
    In [157]: g.transform(normalize)
    Out[157]: 
    0    -1.161895
    1    -1.161895
    2    -1.161895
    3    -0.387298
    4    -0.387298
    5    -0.387298
    6     0.387298
    7     0.387298
    8     0.387298
    9     1.161895
    10    1.161895
    11    1.161895
    Name: value, dtype: float64
    
    In [158]: g.apply(normalize)
    Out[158]: 
    key    
    a    0    -1.161895
         3    -0.387298
         6     0.387298
         9     1.161895
    b    1    -1.161895
         4    -0.387298
         7     0.387298
         10    1.161895
    c    2    -1.161895
         5    -0.387298
         8     0.387298
         11    1.161895
    Name: value, dtype: float64 __

内置聚合函数（如`'mean'`或`'sum'`）通常比通用`apply`函数快得多，且与`transform`结合使用时存在"快速路径"。这使我们能够执行所谓的_未包装_分组操作（unwrapped group operation）：
    
    
    In [159]: g.transform('mean')
    Out[159]: 
    0     4.5
    1     5.5
    2     6.5
    3     4.5
    4     5.5
    5     6.5
    6     4.5
    7     5.5
    8     6.5
    9     4.5
    10    5.5
    11    6.5
    Name: value, dtype: float64
    
    In [160]: normalized = (df['value'] - g.transform('mean')) / g.transform('std')
    
    In [161]: normalized
    Out[161]: 
    0    -1.161895
    1    -1.161895
    2    -1.161895
    3    -0.387298
    4    -0.387298
    5    -0.387298
    6     0.387298
    7     0.387298
    8     0.387298
    9     1.161895
    10    1.161895
    11    1.161895
    Name: value, dtype: float64 __

此处的核心在于：我们在多个分组操作的输出之间进行算术运算，而非编写函数传递给`groupby(...).apply`。这就是"未包装"（unwrapped）的含义。

虽然未包装的分组操作可能涉及多个分组聚合，但向量化操作（vectorized operations）的整体优势通常能抵消这种开销。

## 10.5 数据透视表与交叉表

_数据透视表_（pivot table）是一种常见于电子表格程序和其他数据分析软件的数据汇总工具。它通过一个或多个键对数据表进行聚合，将数据排列成矩形形式，其中部分分组键沿行排列，部分沿列排列。Python 中借助 pandas 实现的数据透视表，是通过本章介绍的 `groupby` 功能结合利用分层索引的reshape操作完成的。DataFrame 有一个 `pivot_table` 方法，此外还有一个顶层的 `pandas.pivot_table` 函数。除了为 `groupby` 提供便捷接口外，`pivot_table` 还可以添加部分总计（也称为_边际值_（margins））。

回到小费数据集，假设您想计算一个按 `day` 和 `smoker` 排列的分组均值表（默认的 `pivot_table` 聚合类型）：
    
    
    In [162]: tips.head()
    Out[162]: 
       total_bill   tip smoker  day    time  size   tip_pct
    0       16.99  1.01     No  Sun  Dinner     2  0.059447
    1       10.34  1.66     No  Sun  Dinner     3  0.160542
    2       21.01  3.50     No  Sun  Dinner     3  0.166587
    3       23.68  3.31     No  Sun  Dinner     2  0.139780
    4       24.59  3.61     No  Sun  Dinner     4  0.146808
    
    In [163]: tips.pivot_table(index=["day", "smoker"],
       .....:                  values=["size", "tip", "tip_pct", "total_bill"])
    Out[163]: 
                     size       tip   tip_pct  total_bill
    day  smoker                                          
    Fri  No      2.250000  2.812500  0.151650   18.420000
         Yes     2.066667  2.714000  0.174783   16.813333
    Sat  No      2.555556  3.102889  0.158048   19.661778
         Yes     2.476190  2.875476  0.147906   21.276667
    Sun  No      2.929825  3.167895  0.160113   20.506667
         Yes     2.578947  3.516842  0.187250   24.120000
    Thur No      2.488889  2.673778  0.160298   17.113111
         Yes     2.352941  3.030000  0.163863   19.190588 __

这也可以直接使用 `groupby` 生成，即 `tips.groupby(["day", "smoker"]).mean()`。现在，假设我们只想计算 `tip_pct` 和 `size` 的平均值，并额外按 `time` 分组。我将把 `smoker` 放在表的列中，`time` 和 `day` 放在行中：
    
    
    In [164]: tips.pivot_table(index=["time", "day"], columns="smoker",
       .....:                  values=["tip_pct", "size"])
    Out[164]: 
                     size             tip_pct          
    smoker             No       Yes        No       Yes
    time   day                                         
    Dinner Fri   2.000000  2.222222  0.139622  0.165347
           Sat   2.555556  2.476190  0.158048  0.147906
           Sun   2.929825  2.578947  0.160113  0.187250
           Thur  2.000000       NaN  0.159744       NaN
    Lunch  Fri   3.000000  1.833333  0.187735  0.188937
           Thur  2.500000  2.352941  0.160311  0.163863 __

我们可以通过传递 `margins=True` 来扩充此表以包含部分总计。这样会添加 `All` 行和列标签，对应的值是在单个层级内所有数据的组统计信息：
    
    
    In [165]: tips.pivot_table(index=["time", "day"], columns="smoker",
       .....:                  values=["tip_pct", "size"], margins=True)
    Out[165]: 
                     size                       tip_pct                    
    smoker             No       Yes       All        No       Yes       All
    time   day                                                             
    Dinner Fri   2.000000  2.222222  2.166667  0.139622  0.165347  0.158916
           Sat   2.555556  2.476190  2.517241  0.158048  0.147906  0.153152
           Sun   2.929825  2.578947  2.842105  0.160113  0.187250  0.166897
           Thur  2.000000       NaN  2.000000  0.159744       NaN  0.159744
    Lunch  Fri   3.000000  1.833333  2.000000  0.187735  0.188937  0.188765
           Thur  2.500000  2.352941  2.459016  0.160311  0.163863  0.161301
    All          2.668874  2.408602  2.569672  0.159328  0.163196  0.160803 __

这里的 `All` 值是不考虑吸烟者与非吸烟者（`All` 列）或行上任何两个分组层级（`All` 行）的均值。

要使用 `mean` 以外的聚合函数，请将其传递给 `aggfunc` 关键字参数。例如，`"count"` 或 `len` 将给出组大小的交叉表（计数或频率）（尽管 `"count"` 会从数据组内的计数中排除空值，而 `len` 不会）：
    
    
    In [166]: tips.pivot_table(index=["time", "smoker"], columns="day",
       .....:                  values="tip_pct", aggfunc=len, margins=True)
    Out[166]: 
    day             Fri   Sat   Sun  Thur  All
    time   smoker                             
    Dinner No       3.0  45.0  57.0   1.0  106
           Yes      9.0  42.0  19.0   NaN   70
    Lunch  No       1.0   NaN   NaN  44.0   45
           Yes      6.0   NaN   NaN  17.0   23
    All            19.0  87.0  76.0  62.0  244 __

如果某些组合为空（或其他 NA），您可能希望传递一个 `fill_value`：
    
    
    In [167]: tips.pivot_table(index=["time", "size", "smoker"], columns="day",
       .....:                  values="tip_pct", fill_value=0)
    Out[167]: 
    day                      Fri       Sat       Sun      Thur
    time   size smoker                                        
    Dinner 1    No      0.000000  0.137931  0.000000  0.000000
                Yes     0.000000  0.325733  0.000000  0.000000
           2    No      0.139622  0.162705  0.168859  0.159744
                Yes     0.171297  0.148668  0.207893  0.000000
           3    No      0.000000  0.154661  0.152663  0.000000
    ...                      ...       ...       ...       ...
    Lunch  3    Yes     0.000000  0.000000  0.000000  0.204952
           4    No      0.000000  0.000000  0.000000  0.138919
                Yes     0.000000  0.000000  0.000000  0.155410
           5    No      0.000000  0.000000  0.000000  0.121389
           6    No      0.000000  0.000000  0.000000  0.173706
    [21 rows x 4 columns]__

有关 `pivot_table` 选项的摘要，请参见表 10.2。

表 10.2: `pivot_table` 选项 参数 | 描述  
---|---  
`values` | 要聚合的列名或多个列名；默认聚合所有数值列  
`index` | 用于在结果数据透视表的行上进行分组的列名或其他分组键  
`columns` | 用于在结果数据透视表的列上进行分组的列名或其他分组键  
`aggfunc` | 聚合函数或函数列表（默认为 `"mean"`）；可以是 `groupby` 上下文中有效的任何函数  
`fill_value` | 替换结果表中的缺失值  
`dropna` | 如果为 `True`，则不包含所有条目均为 `NA` 的列  
`margins` | 添加行/列小计和总计（默认为 `False`）  
`margins_name` | 传递 `margins=True` 时用于边际行/列标签的名称；默认为 `"All"`  
`observed` | 对于分类分组键，如果为 `True`，则仅显示键中观察到的类别值，而不是所有类别  
  
### 交叉表：Crosstab

_交叉表_（cross-tabulation，简称 _crosstab_）是数据透视表的一种特殊情况，用于计算组频率。以下是一个示例：
    
    
    In [168]: from io import StringIO
    
    In [169]: data = """Sample  Nationality  Handedness
       .....: 1   USA  Right-handed
       .....: 2   Japan    Left-handed
       .....: 3   USA  Right-handed
       .....: 4   Japan    Right-handed
       .....: 5   Japan    Left-handed
       .....: 6   Japan    Right-handed
       .....: 7   USA  Right-handed
       .....: 8   USA  Left-handed
       .....: 9   Japan    Right-handed
       .....: 10  USA  Right-handed"""
       .....:
    
    In [170]: data = pd.read_table(StringIO(data), sep="\s+")__
    
    
    In [171]: data
    Out[171]: 
       Sample Nationality    Handedness
    0       1         USA  Right-handed
    1       2       Japan   Left-handed
    2       3         USA  Right-handed
    3       4       Japan  Right-handed
    4       5       Japan   Left-handed
    5       6       Japan  Right-handed
    6       7         USA  Right-handed
    7       8         USA   Left-handed
    8       9       Japan  Right-handed
    9      10         USA  Right-handed __

作为某项调查分析的一部分，我们可能希望按国籍和用手习惯汇总这些数据。您可以使用 `pivot_table` 来实现，但 `pandas.crosstab` 函数可能更方便：
    
    
    In [172]: pd.crosstab(data["Nationality"], data["Handedness"], margins=True)
    Out[172]: 
    Handedness   Left-handed  Right-handed  All
    Nationality                                
    Japan                  2             3    5
    USA                    1             4    5
    All                    3             7   10 __

`crosstab` 的前两个参数可以分别是数组、Series 或数组列表。如小费数据所示：
    
    
    In [173]: pd.crosstab([tips["time"], tips["day"]], tips["smoker"], margins=True)
    Out[173]: 
    smoker        No  Yes  All
    time   day                
    Dinner Fri     3    9   12
           Sat    45   42   87
           Sun    57   19   76
           Thur    1    0    1
    Lunch  Fri     1    6    7
           Thur   44   17   61
    All          151   93  244 __

## 10.6 结论

掌握 pandas 的数据分组工具有助于数据清洗、建模或统计分析工作。在第十三章：数据分析示例中，我们将看到更多在真实数据上使用 `groupby` 的实际案例。

在下一章中，我们将把注意力转向时间序列数据。