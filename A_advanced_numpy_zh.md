# AAdvanced NumPy

__

《利用Python进行数据分析 第三版》（_Python for Data Analysis 3rd Edition_）的开放获取（Open Access）网络版现已上线，作为[纸质版与电子版](https://amzn.to/3DyLaJc)的配套资源。如果您发现任何勘误，[请在此处提交](https://oreilly.com/catalog/0636920519829/errata)。请注意，由于本网站由Quarto生成，部分呈现形式会与O'Reilly出版的印刷版及电子书版本有所不同。

若您觉得本书在线版对您有帮助，请考虑[订购纸质版本](https://amzn.to/3DyLaJc)或[无DRM限制的电子书](https://www.ebooks.com/en-us/book/210644288/python-for-data-analysis/wes-mckinney/?affId=WES398681F)以支持作者。本站内容不得复制或转载。代码示例遵循MIT许可协议，可在GitHub或Gitee上获取。

在本附录中，我将深入探讨用于数组计算的NumPy库。这将包括有关ndarray类型的更多内部细节，以及更高级的数组操作和算法。

本附录包含多个主题，无需按顺序阅读。在各章节中，我将使用`numpy.random`模块中的默认随机数生成器生成随机数据用于示例：
    
    
    In [11]: rng = np.random.default_rng(seed=12345)__

## A.1 ndarray 对象内部结构

NumPy 的 ndarray 提供了一种将均匀类型数据块（无论是连续还是跨步的）解释为多维数组对象的方式。数据类型（或称 _dtype_）决定了数据如何被解释为浮点数、整数、布尔值或我们一直在研究的任何其他类型。

ndarray 之所以灵活，部分原因在于每个数组对象都是数据块上的一个 _跨步_ 视图。例如，你可能会想知道数组视图 `arr[::2, ::-1]` 为何不复制任何数据。原因在于 ndarray 不仅仅是一块内存和一个数据类型；它还具有 _步长_ 信息，使数组能够以不同的步幅在内存中移动。更准确地说，ndarray 内部由以下部分组成：

  * 一个指向数据的 _指针_ ——即 RAM 或内存映射文件中的数据块

  * 描述数组中固定大小值单元的 _数据类型_ 或 dtype

  * 表示数组 _形状_ 的元组

  * 一个 _步长_ 元组——整数，指示沿维度前进一个元素需要“步进”的字节数

图 A.1 展示了 ndarray 内部结构的简单示意图。

![](images/pda3_a001.png)

图 A.1：NumPy ndarray 对象

例如，一个 10 × 5 的数组的形状为 `(10, 5)`：
    
    
    In [12]: np.ones((10, 5)).shape
    Out[12]: (10, 5)__

一个典型的（C 顺序）3 × 4 × 5 的 `float64`（8 字节）值数组的步长为 `(160, 40, 8)`（了解步长很有用，因为通常来说，特定轴上的步长越大，沿该轴执行计算的成本就越高）：
    
    
    In [13]: np.ones((3, 4, 5), dtype=np.float64).strides
    Out[13]: (160, 40, 8)__

虽然典型的 NumPy 用户很少会对数组步长感兴趣，但构建“零复制”数组视图时需要它们。步长甚至可以是负数，这使得数组能够在内存中“向后”移动（例如，在像 `obj[::-1]` 或 `obj[:, ::-1]` 这样的切片中就会出现这种情况）。

### NumPy 数据类型层次结构

偶尔，你可能需要编写代码来检查数组是否包含整数、浮点数、字符串或 Python 对象。由于浮点数有多种类型（从 `float16` 到 `float128`），检查数据类型是否属于某个类型列表会非常冗长。幸运的是，数据类型有超类，例如 `np.integer` 和 `np.floating`，它们可以与 `np.issubdtype` 函数一起使用：
    
    
    In [14]: ints = np.ones(10, dtype=np.uint16)
    
    In [15]: floats = np.ones(10, dtype=np.float32)
    
    In [16]: np.issubdtype(ints.dtype, np.integer)
    Out[16]: True
    
    In [17]: np.issubdtype(floats.dtype, np.floating)
    Out[17]: True __

你可以通过调用类型的 `mro` 方法来查看特定数据类型的所有父类：
    
    
    In [18]: np.float64.mro()
    Out[18]: 
    [numpy.float64,
     numpy.floating,
     numpy.inexact,
     numpy.number,
     numpy.generic,
     float,
     object]__

因此，我们还有：
    
    
    In [19]: np.issubdtype(ints.dtype, np.number)
    Out[19]: True __

大多数 NumPy 用户永远不需要了解这一点，但这偶尔会很有用。图 A.2 展示了数据类型层次结构及父子类关系的图表。

![](images/pda3_a002.png)

图 A.2：NumPy 数据类型类层次结构

## A.2 高级数组操作

除了花式索引、切片和布尔子集选择外，还有许多操作数组的方法。虽然数据分析应用中的大部分繁重工作由 pandas 中的高级函数处理，但有时你可能需要编写现有库中未提供的数据算法。

### 数组重塑

在许多情况下，你可以在不复制任何数据的情况下将数组从一种形状转换为另一种形状。为此，只需向 `reshape` 数组实例方法传递一个表示新形状的元组。例如，假设我们有一个一维数值数组，希望将其重新排列成矩阵（如图 A.3 所示）：
    
    
    In [20]: arr = np.arange(8)
    
    In [21]: arr
    Out[21]: array([0, 1, 2, 3, 4, 5, 6, 7])
    
    In [22]: arr.reshape((4, 2))
    Out[22]: 
    array([[0, 1],
           [2, 3],
           [4, 5],
           [6, 7]])__

![](images/pda3_a003.png)

图 A.3：按 C（行优先）或 FORTRAN（列优先）顺序进行重塑

多维数组也可以重塑：
    
    
    In [23]: arr.reshape((4, 2)).reshape((2, 4))
    Out[23]: 
    array([[0, 1, 2, 3],
           [4, 5, 6, 7]])__

传递的形状维度中可以有一个为 –1，该维度的值将根据数据推断得出：
    
    
    In [24]: arr = np.arange(15)
    
    In [25]: arr.reshape((5, -1))
    Out[25]: 
    array([[ 0,  1,  2],
           [ 3,  4,  5],
           [ 6,  7,  8],
           [ 9, 10, 11],
           [12, 13, 14]])__

由于数组的 `shape` 属性是一个元组，它也可以传递给 `reshape`：
    
    
    In [26]: other_arr = np.ones((3, 5))
    
    In [27]: other_arr.shape
    Out[27]: (3, 5)
    
    In [28]: arr.reshape(other_arr.shape)
    Out[28]: 
    array([[ 0,  1,  2,  3,  4],
           [ 5,  6,  7,  8,  9],
           [10, 11, 12, 13, 14]])__

从一维到更高维度的 `reshape` 逆操作通常称为 _扁平化_（flattening）或 _展平_（raveling）：
    
    
    In [29]: arr = np.arange(15).reshape((5, 3))
    
    In [30]: arr
    Out[30]: 
    array([[ 0,  1,  2],
           [ 3,  4,  5],
           [ 6,  7,  8],
           [ 9, 10, 11],
           [12, 13, 14]])
    
    In [31]: arr.ravel()
    Out[31]: array([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14])__

如果结果中的值在原始数组中是连续的，`ravel` 不会生成底层值的副本。

`flatten` 方法的行为类似于 `ravel`，但它总是返回数据的副本：
    
    
    In [32]: arr.flatten()
    Out[32]: array([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14])__

数据可以按不同的顺序重塑或展平。这对 NumPy 新手来说是一个稍显微妙的话题，因此是下一个子主题。

### C 顺序与 FORTRAN 顺序

NumPy 能够适应内存中数据的多种不同布局。默认情况下，NumPy 数组以 _行优先_（row major）顺序创建。在空间上，这意味着如果你有一个二维数据数组，数组中每一行的项存储在相邻的内存位置。行优先顺序的替代方案是 _列优先_（column major）顺序，这意味着数据每一列中的值存储在相邻的内存位置。

由于历史原因，行优先和列优先顺序也分别称为 C 顺序和 FORTRAN 顺序。在 FORTRAN 77 语言中，矩阵都是列优先的。

像 `reshape` 和 `ravel` 这样的函数接受一个 `order` 参数，指示使用数组中数据的顺序。在大多数情况下，这通常设置为 `'C'` 或 `'F'`（还有较少使用的选项 `'A'` 和 `'K'`；请参阅 NumPy 文档，并参考图 A.3 以了解这些选项的图示）：
    
    
    In [33]: arr = np.arange(12).reshape((3, 4))
    
    In [34]: arr
    Out[34]: 
    array([[ 0,  1,  2,  3],
           [ 4,  5,  6,  7],
           [ 8,  9, 10, 11]])
    
    In [35]: arr.ravel()
    Out[35]: array([ 0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11])
    
    In [36]: arr.ravel('F')
    Out[36]: array([ 0,  4,  8,  1,  5,  9,  2,  6, 10,  3,  7, 11])__

重塑超过二维的数组可能有点令人费解（参见图 A.3）。C 顺序和 FORTRAN 顺序之间的关键区别在于遍历维度的方式：

C/行优先顺序
    

_先_ 遍历较高维度（例如，在推进轴 0 之前先处理轴 1）。

FORTRAN/列优先顺序
    

_后_ 遍历较高维度（例如，在推进轴 1 之前先处理轴 0）。

### 连接和拆分数组

`numpy.concatenate` 接受一个数组序列（元组、列表等）并按输入轴顺序连接它们：
    
    
    In [37]: arr1 = np.array([[1, 2, 3], [4, 5, 6]])
    
    In [38]: arr2 = np.array([[7, 8, 9], [10, 11, 12]])
    
    In [39]: np.concatenate([arr1, arr2], axis=0)
    Out[39]: 
    array([[ 1,  2,  3],
           [ 4,  5,  6],
           [ 7,  8,  9],
           [10, 11, 12]])
    
    In [40]: np.concatenate([arr1, arr2], axis=1)
    Out[40]: 
    array([[ 1,  2,  3,  7,  8,  9],
           [ 4,  5,  6, 10, 11, 12]])__

有一些便捷函数，如 `vstack` 和 `hstack`，用于常见类型的连接。前面的操作可以表示为：
    
    
    In [41]: np.vstack((arr1, arr2))
    Out[41]: 
    array([[ 1,  2,  3],
           [ 4,  5,  6],
           [ 7,  8,  9],
           [10, 11, 12]])
    
    In [42]: np.hstack((arr1, arr2))
    Out[42]: 
    array([[ 1,  2,  3,  7,  8,  9],
           [ 4,  5,  6, 10, 11, 12]])__

另一方面，`split` 将数组沿某个轴切片为多个数组：
    
    
    In [43]: arr = rng.standard_normal((5, 2))
    
    In [44]: arr
    Out[44]: 
    array([[-1.4238,  1.2637],
           [-0.8707, -0.2592],
           [-0.0753, -0.7409],
           [-1.3678,  0.6489],
           [ 0.3611, -1.9529]])
    
    In [45]: first, second, third = np.split(arr, [1, 3])
    
    In [46]: first
    Out[46]: array([[-1.4238,  1.2637]])
    
    In [47]: second
    Out[47]: 
    array([[-0.8707, -0.2592],
           [-0.0753, -0.7409]])
    
    In [48]: third
    Out[48]: 
    array([[-1.3678,  0.6489],
           [ 0.3611, -1.9529]])__

传递给 `np.split` 的值 `[1, 3]` 表示将数组分割成片段的索引。

有关所有相关连接和拆分函数的列表，请参见表 A.1，其中一些函数仅作为通用 `concatenate` 的便捷方式提供。

表 A.1：数组连接函数 | 函数 | 描述  
---|---  
`concatenate` | 最通用的函数，沿一个轴连接数组集合  
`vstack, row_stack` | 按行堆叠数组（沿轴 0）  
`hstack` | 按列堆叠数组（沿轴 1）  
`column_stack` | 类似于 `hstack`，但先将一维数组转换为二维列向量  
`dstack` | 按“深度”堆叠数组（沿轴 2）  
`split` | 沿特定轴在指定位置拆分数组  
`hsplit`/`vsplit` | 分别在轴 0 和 1 上拆分的便捷函数  
  
#### 堆叠辅助函数：r\_ 和 c\_

NumPy 命名空间中有两个特殊对象 `r_` 和 `c_`，它们使堆叠数组更简洁：
    
    
    In [49]: arr = np.arange(6)
    
    In [50]: arr1 = arr.reshape((3, 2))
    
    In [51]: arr2 = rng.standard_normal((3, 2))
    
    In [52]: np.r_[arr1, arr2]
    Out[52]: 
    array([[ 0.    ,  1.    ],
           [ 2.    ,  3.    ],
           [ 4.    ,  5.    ],
           [ 2.3474,  0.9685],
           [-0.7594,  0.9022],
           [-0.467 , -0.0607]])
    
    In [53]: np.c_[np.r_[arr1, arr2], arr]
    Out[53]: 
    array([[ 0.    ,  1.    ,  0.    ],
           [ 2.    ,  3.    ,  1.    ],
           [ 4.    ,  5.    ,  2.    ],
           [ 2.3474,  0.9685,  3.    ],
           [-0.7594,  0.9022,  4.    ],
           [-0.467 , -0.0607,  5.    ]])__

此外，它们还可以将切片转换为数组：
    
    
    In [54]: np.c_[1:6, -10:-5]
    Out[54]: 
    array([[  1, -10],
           [  2,  -9],
           [  3,  -8],
           [  4,  -7],
           [  5,  -6]])__

有关 `c_` 和 `r_` 的更多功能，请参阅文档字符串。

### 重复元素：tile 和 repeat

`repeat` 和 `tile` 函数是两个有用的工具，用于重复或复制数组以产生更大的数组。`repeat` 将数组中的每个元素重复若干次，产生一个更大的数组：
    
    
    In [55]: arr = np.arange(3)
    
    In [56]: arr
    Out[56]: array([0, 1, 2])
    
    In [57]: arr.repeat(3)
    Out[57]: array([0, 0, 0, 1, 1, 1, 2, 2, 2])__

__

注意 

在 NumPy 中，复制或重复数组的需求可能比其他数组编程框架（如 MATLAB）少。其中一个原因是 _广播_（broadcasting）通常能更好地满足这一需求，这将是下一节的主题。

默认情况下，如果传递一个整数，每个元素将重复该次数。如果传递一个整数数组，每个元素可以重复不同的次数：
    
    
    In [58]: arr.repeat([2, 3, 4])
    Out[58]: array([0, 0, 1, 1, 1, 2, 2, 2, 2])__

多维数组可以沿特定轴重复其元素：
    
    
    In [59]: arr = rng.standard_normal((2, 2))
    
    In [60]: arr
    Out[60]: 
    array([[ 0.7888, -1.2567],
           [ 0.5759,  1.399 ]])
    
    In [61]: arr.repeat(2, axis=0)
    Out[61]: 
    array([[ 0.7888, -1.2567],
           [ 0.7888, -1.2567],
           [ 0.5759,  1.399 ],
           [ 0.5759,  1.399 ]])__

注意，如果没有传递轴，数组将首先被展平，这可能不是你想要的结果。类似地，在重复多维数组时，可以传递一个整数数组，以不同的次数重复给定的切片：
    
    
    In [62]: arr.repeat([2, 3], axis=0)
    Out[62]: 
    array([[ 0.7888, -1.2567],
           [ 0.7888, -1.2567],
           [ 0.5759,  1.399 ],
           [ 0.5759,  1.399 ],
           [ 0.5759,  1.399 ]])
    
    In [63]: arr.repeat([2, 3], axis=1)
    Out[63]: 
    array([[ 0.7888,  0.7888, -1.2567, -1.2567, -1.2567],
           [ 0.5759,  0.5759,  1.399 ,  1.399 ,  1.399 ]])__

另一方面，`tile` 是沿轴堆叠数组副本的快捷方式。直观上，你可以将其类比为“铺瓷砖”：
    
    
    In [64]: arr
    Out[64]: 
    array([[ 0.7888, -1.2567],
           [ 0.5759,  1.399 ]])
    
    In [65]: np.tile(arr, 2)
    Out[65]: 
    array([[ 0.7888, -1.2567,  0.7888, -1.2567],
           [ 0.5759,  1.399 ,  0.5759,  1.399 ]])__

第二个参数是瓷砖的数量；使用标量时，平铺是逐行进行的，而不是逐列进行。`tile` 的第二个参数可以是一个元组，指示“平铺”的布局：
    
    
    In [66]: arr
    Out[66]: 
    array([[ 0.7888, -1.2567],
           [ 0.5759,  1.399 ]])
    
    In [67]: np.tile(arr, (2, 1))
    Out[67]: 
    array([[ 0.7888, -1.2567],
           [ 0.5759,  1.399 ],
           [ 0.7888, -1.2567],
           [ 0.5759,  1.399 ]])
    
    In [68]: np.tile(arr, (3, 2))
    Out[68]: 
    array([[ 0.7888, -1.2567,  0.7888, -1.2567],
           [ 0.5759,  1.399 ,  0.5759,  1.399 ],
           [ 0.7888, -1.2567,  0.7888, -1.2567],
           [ 0.5759,  1.399 ,  0.5759,  1.399 ],
           [ 0.7888, -1.2567,  0.7888, -1.2567],
           [ 0.5759,  1.399 ,  0.5759,  1.399 ]])__

### 花式索引等效方法：take 和 put

你可能还记得 [第 4 章：NumPy 基础：数组和向量化计算](https://wesmckinney.com/book/numpy-basics)，获取和设置数组子集的一种方法是使用整数数组进行 _花式_ 索引：
    
    
    In [69]: arr = np.arange(10) * 100
    
    In [70]: inds = [7, 1, 2, 6]
    
    In [71]: arr[inds]
    Out[71]: array([700, 100, 200, 600])__

有一些替代的 ndarray 方法，仅在单个轴上选择时有用：
    
    
    In [72]: arr.take(inds)
    Out[72]: array([700, 100, 200, 600])
    
    In [73]: arr.put(inds, 42)
    
    In [74]: arr
    Out[74]: array([  0,  42,  42, 300, 400, 500,  42,  42, 800, 900])
    
    In [75]: arr.put(inds, [40, 41, 42, 43])
    
    In [76]: arr
    Out[76]: array([  0,  41,  42, 300, 400, 500,  43,  40, 800, 900])__

要在其他轴上使用 `take`，可以传递 `axis` 关键字：
    
    
    In [77]: inds = [2, 0, 2, 1]
    
    In [78]: arr = rng.standard_normal((2, 4))
    
    In [79]: arr
    Out[79]: 
    array([[ 1.3223, -0.2997,  0.9029, -1.6216],
           [-0.1582,  0.4495, -1.3436, -0.0817]])
    
    In [80]: arr.take(inds, axis=1)
    Out[80]: 
    array([[ 0.9029,  1.3223,  0.9029, -0.2997],
           [-1.3436, -0.1582, -1.3436,  0.4495]])__

`put` 不接受 `axis` 参数，而是索引到数组的展平（一维，C 顺序）版本中。因此，当需要使用索引数组在其他轴上设置元素时，最好使用基于 `[]` 的索引。

## A.3 广播机制

_广播_（Broadcasting）规定了不同形状数组之间的运算方式。它虽然是一个强大功能，但即使用户经验丰富也容易产生困惑。最简单的广播示例是将标量值与数组结合时：
    
    
    In [81]: arr = np.arange(5)
    
    In [82]: arr
    Out[82]: array([0, 1, 2, 3, 4])
    
    In [83]: arr * 4
    Out[83]: array([ 0,  4,  8, 12, 16])

这里我们说标量值 4 在乘法运算中被_广播_到了所有其他元素上。

例如，我们可以通过减去列均值来对数组的每一列进行去均值处理。此时只需减去包含每列均值的数组：
    
    
    In [84]: arr = rng.standard_normal((4, 3))
    
    In [85]: arr.mean(0)
    Out[85]: array([0.1206, 0.243 , 0.1444])
    
    In [86]: demeaned = arr - arr.mean(0)
    
    In [87]: demeaned
    Out[87]: 
    array([[ 1.6042,  2.3751,  0.633 ],
           [ 0.7081, -1.202 , -1.3538],
           [-1.5329,  0.2985,  0.6076],
           [-0.7793, -1.4717,  0.1132]])
    
    In [88]: demeaned.mean(0)
    Out[88]: array([ 0., -0.,  0.])

图 A.4 展示了这个运算过程。对行进行去均值作为广播操作需要更谨慎处理。幸运的是，只要遵循规则，就可以在数组的任何维度上广播可能较低维度的值（例如从二维数组的每列中减去行均值）。

这就引出了广播规则：

两个数组可进行广播的条件是：对于每个_末尾维度_（即从尾部开始计算），轴长度匹配或任一长度为 1。广播会在缺失或长度为 1 的维度上进行。

![](images/pda3_a004.png)

图 A.4：使用一维数组在轴 0 上进行广播

即使作为经验丰富的 NumPy 用户，我在思考广播规则时也经常需要停下来画示意图。考虑最后一个例子，假设我们希望对每行减去均值。由于 `arr.mean(0)` 的长度为 3，它可跨轴 0 进行广播，因为 `arr` 的末尾维度是 3 且匹配。根据规则，要在轴 1 上进行减法（即从每行减去行均值），较小数组的形状必须为 `(4, 1)`：
    
    
    In [89]: arr
    Out[89]: 
    array([[ 1.7247,  2.6182,  0.7774],
           [ 0.8286, -0.959 , -1.2094],
           [-1.4123,  0.5415,  0.7519],
           [-0.6588, -1.2287,  0.2576]])
    
    In [90]: row_means = arr.mean(1)
    
    In [91]: row_means.shape
    Out[91]: (4,)
    
    In [92]: row_means.reshape((4, 1))
    Out[92]: 
    array([[ 1.7068],
           [-0.4466],
           [-0.0396],
           [-0.5433]])
    
    In [93]: demeaned = arr - row_means.reshape((4, 1))
    
    In [94]: demeaned.mean(1)
    Out[94]: array([-0.,  0.,  0.,  0.])

图 A.5 展示了这个运算过程。

![](images/pda3_a005.png)

图 A.5：在二维数组的轴 1 上进行广播

图 A.6 展示了另一个示例，这次是在轴 0 上将二维数组与三维数组相加。

![](images/pda3_a006.png)

图 A.6：在三维数组的轴 0 上进行广播

### 在其他轴上的广播

高维数组的广播可能更令人费解，但实际上只需遵循规则即可。如果不遵守规则，就会出现如下错误：
    
    
    In [95]: arr - arr.mean(1)
    ---------------------------------------------------------------------------
    ValueError                                Traceback (most recent call last)
    <ipython-input-95-8b8ada26fac0> in <module>
    ----> 1 arr - arr.mean(1)
    ValueError: operands could not be broadcast together with shapes (4,3) (4,)

通常需要在轴 0 以外的轴上使用低维数组进行算术运算。根据广播规则，较小数组的"广播维度"必须为 1。在展示的行去均值示例中，这意味着将行的形状从 `(4,)` 重塑为 `(4, 1)`：
    
    
    In [96]: arr - arr.mean(1).reshape((4, 1))
    Out[96]: 
    array([[ 0.018 ,  0.9114, -0.9294],
           [ 1.2752, -0.5124, -0.7628],
           [-1.3727,  0.5811,  0.7915],
           [-0.1155, -0.6854,  0.8009]])

对于三维情况，在三个维度中的任何一个上进行广播都只是将数据重塑为形状兼容的问题。图 A.7 清晰地展示了在三维数组的每个轴上进行广播所需的形状。

![](images/pda3_a007.png)

图 A.7：用于在三维数组上进行广播的兼容二维数组形状

因此，常见问题是为了广播目的需要专门添加长度为 1 的新轴。使用 `reshape` 是一种选择，但插入轴需要构造指示新形状的元组，这通常很繁琐。因此，NumPy 数组提供了一种通过索引插入新轴的特殊语法。我们使用特殊的 `np.newaxis` 属性与"完整"切片来插入新轴：
    
    
    In [97]: arr = np.zeros((4, 4))
    
    In [98]: arr_3d = arr[:, np.newaxis, :]
    
    In [99]: arr_3d.shape
    Out[99]: (4, 1, 4)
    
    In [100]: arr_1d = rng.standard_normal(3)
    
    In [101]: arr_1d[:, np.newaxis]
    Out[101]: 
    array([[ 0.3129],
           [-0.1308],
           [ 1.27  ]])
    
    In [102]: arr_1d[np.newaxis, :]
    Out[102]: array([[ 0.3129, -0.1308,  1.27  ]])

因此，如果我们有一个三维数组并希望对轴 2 进行去均值处理，就需要这样写：
    
    
    In [103]: arr = rng.standard_normal((3, 4, 5))
    
    In [104]: depth_means = arr.mean(2)
    
    In [105]: depth_means
    Out[105]: 
    array([[ 0.0431,  0.2747, -0.1885, -0.2014],
           [-0.5732, -0.5467,  0.1183, -0.6301],
           [ 0.0972,  0.5954,  0.0331, -0.6002]])
    
    In [106]: depth_means.shape
    Out[106]: (3, 4)
    
    In [107]: demeaned = arr - depth_means[:, :, np.newaxis]
    
    In [108]: demeaned.mean(2)
    Out[108]: 
    array([[ 0., -0.,  0., -0.],
           [ 0., -0., -0., -0.],
           [ 0.,  0.,  0.,  0.]])

您可能想知道是否有办法在不牺牲性能的情况下通用地对轴进行去均值处理。确实有，但需要一些索引技巧：
    
    
    def demean_axis(arr, axis=0):
        means = arr.mean(axis)
    
        # 这将 [:, :, np.newaxis] 等操作推广到 N 维
        indexer = [slice(None)] * arr.ndim
        indexer[axis] = np.newaxis
        return arr - means[indexer]

### 通过广播设置数组值

控制算术运算的相同广播规则也适用于通过数组索引设置值。在简单情况下，我们可以这样做：
    
    
    In [109]: arr = np.zeros((4, 3))
    
    In [110]: arr[:] = 5
    
    In [111]: arr
    Out[111]: 
    array([[5., 5., 5.],
           [5., 5., 5.],
           [5., 5., 5.],
           [5., 5., 5.]])

但是，如果我们有一个一维数组的值想要设置到数组的列中，只要形状兼容就可以实现：
    
    
    In [112]: col = np.array([1.28, -0.42, 0.44, 1.6])
    
    In [113]: arr[:] = col[:, np.newaxis]
    
    In [114]: arr
    Out[114]: 
    array([[ 1.28,  1.28,  1.28],
           [-0.42, -0.42, -0.42],
           [ 0.44,  0.44,  0.44],
           [ 1.6 ,  1.6 ,  1.6 ]])
    
    In [115]: arr[:2] = [[-1.37], [0.509]]
    
    In [116]: arr
    Out[116]: 
    array([[-1.37 , -1.37 , -1.37 ],
           [ 0.509,  0.509,  0.509],
           [ 0.44 ,  0.44 ,  0.44 ],
           [ 1.6  ,  1.6  ,  1.6  ]])

## A.4 高级 ufunc 用法

虽然许多 NumPy 用户只会使用通用函数（ufunc）提供的快速逐元素操作，但一些附加功能偶尔能帮助您编写更简洁的代码，而无需显式循环。

### ufunc 实例方法

NumPy 的每个二元 ufunc 都有特殊方法，用于执行某些特殊类型的向量化操作。这些方法总结在表 A.2 中，但我会给出一些具体示例来说明它们的工作原理。

`reduce` 接受单个数组，并通过执行一系列二元操作来聚合其值（可选沿某个轴进行）。例如，对数组元素求和的另一种方法是使用 `np.add.reduce`：
    
    
    In [117]: arr = np.arange(10)
    
    In [118]: np.add.reduce(arr)
    Out[118]: 45
    
    In [119]: arr.sum()
    Out[119]: 45 __

起始值（例如，对于 `add` 是 0）取决于 ufunc。如果传递了轴，则沿该轴执行归约（reduce）。这使您能够以简洁的方式回答某些类型的问题。作为一个更实际的例子，我们可以使用 `np.logical_and` 来检查数组中每行的值是否已排序：
    
    
    In [120]: my_rng = np.random.default_rng(12346)  # 为了可重现性
    
    In [121]: arr = my_rng.standard_normal((5, 5))
    
    In [122]: arr
    Out[122]: 
    array([[-0.9039,  0.1571,  0.8976, -0.7622, -0.1763],
           [ 0.053 , -1.6284, -0.1775,  1.9636,  1.7813],
           [-0.8797, -1.6985, -1.8189,  0.119 , -0.4441],
           [ 0.7691, -0.0343,  0.3925,  0.7589, -0.0705],
           [ 1.0498,  1.0297, -0.4201,  0.7863,  0.9612]])
    
    In [123]: arr[::2].sort(1) # 对部分行进行排序
    
    In [124]: arr[:, :-1] < arr[:, 1:]
    Out[124]: 
    array([[ True,  True,  True,  True],
           [False,  True,  True, False],
           [ True,  True,  True,  True],
           [False,  True,  True, False],
           [ True,  True,  True,  True]])
    
    In [125]: np.logical_and.reduce(arr[:, :-1] < arr[:, 1:], axis=1)
    Out[125]: array([ True, False,  True, False,  True])__

注意，`logical_and.reduce` 等价于 `all` 方法。

`accumulate` ufunc 方法与 `reduce` 相关，就像 `cumsum` 与 `sum` 相关一样。它生成一个相同大小的数组，其中包含中间的“累积”值：
    
    
    In [126]: arr = np.arange(15).reshape((3, 5))
    
    In [127]: np.add.accumulate(arr, axis=1)
    Out[127]: 
    array([[ 0,  1,  3,  6, 10],
           [ 5, 11, 18, 26, 35],
           [10, 21, 33, 46, 60]])__

`outer` 对两个数组执行逐对叉积（cross product）：
    
    
    In [128]: arr = np.arange(3).repeat([1, 2, 2])
    
    In [129]: arr
    Out[129]: array([0, 1, 1, 2, 2])
    
    In [130]: np.multiply.outer(arr, np.arange(5))
    Out[130]: 
    array([[0, 0, 0, 0, 0],
           [0, 1, 2, 3, 4],
           [0, 1, 2, 3, 4],
           [0, 2, 4, 6, 8],
           [0, 2, 4, 6, 8]])__

`outer` 的输出将具有一个维度，该维度是输入维度的串联：
    
    
    In [131]: x, y = rng.standard_normal((3, 4)), rng.standard_normal(5)
    
    In [132]: result = np.subtract.outer(x, y)
    
    In [133]: result.shape
    Out[133]: (3, 4, 5)__

最后一个方法 `reduceat` 执行“局部归约”（local reduce），本质上是一种数组“分组”操作，其中数组的切片被聚合在一起。它接受一个“分箱边界”（bin edges）序列，指示如何分割和聚合值：
    
    
    In [134]: arr = np.arange(10)
    
    In [135]: np.add.reduceat(arr, [0, 5, 8])
    Out[135]: array([10, 18, 17])__

结果是在 `arr[0:5]`、`arr[5:8]` 和 `arr[8:]` 上执行的归约（此处为求和）。与其他方法一样，您可以传递 `axis` 参数：
    
    
    In [136]: arr = np.multiply.outer(np.arange(4), np.arange(5))
    
    In [137]: arr
    Out[137]: 
    array([[ 0,  0,  0,  0,  0],
           [ 0,  1,  2,  3,  4],
           [ 0,  2,  4,  6,  8],
           [ 0,  3,  6,  9, 12]])
    
    In [138]: np.add.reduceat(arr, [0, 2, 4], axis=1)
    Out[138]: 
    array([[ 0,  0,  0],
           [ 1,  5,  4],
           [ 2, 10,  8],
           [ 3, 15, 12]])__

有关 ufunc 方法的部分列表，请参见表 A.2。

表 A.2: ufunc 方法

方法 | 描述  
---|---  
`accumulate(x)` | 聚合值，保留所有部分聚合结果。  
`at(x, indices, b=None)` | 在指定索引处对 `x` 执行就地操作。参数 `b` 是需要两个数组输入的 ufunc 的第二个输入。  
`reduce(x)` | 通过连续应用操作来聚合值。  
`reduceat(x, bins)` | “局部”归约或“分组”；归约连续的数据切片以产生聚合数组。  
`outer(x, y)` | 对 `x` 和 `y` 中的所有元素对应用操作；结果数组的形状为 `x.shape + y.shape`。  

### 在 Python 中编写新的 ufunc

有多种方法可以创建您自己的 NumPy ufunc。最通用的是使用 NumPy C API，但这超出了本书的范围。在本节中，我们将介绍纯 Python ufunc。

`numpy.frompyfunc` 接受一个 Python 函数以及输入和输出数量的规范。例如，一个简单的逐元素相加函数可以指定为：
    
    
    In [139]: def add_elements(x, y):
       .....:     return x + y
    
    In [140]: add_them = np.frompyfunc(add_elements, 2, 1)
    
    In [141]: add_them(np.arange(8), np.arange(8))
    Out[141]: array([0, 2, 4, 6, 8, 10, 12, 14], dtype=object)__

使用 `frompyfunc` 创建的函数总是返回 Python 对象的数组，这可能不太方便。幸运的是，有一个替代的（但功能稍弱的）函数 `numpy.vectorize`，允许您指定输出类型：
    
    
    In [142]: add_them = np.vectorize(add_elements, otypes=[np.float64])
    
    In [143]: add_them(np.arange(8), np.arange(8))
    Out[143]: array([ 0.,  2.,  4.,  6.,  8., 10., 12., 14.])__

这些函数提供了创建类 ufunc 函数的方法，但它们非常慢，因为需要调用 Python 函数来计算每个元素，这比 NumPy 基于 C 的 ufunc 循环要慢得多：
    
    
    In [144]: arr = rng.standard_normal(10000)
    
    In [145]: %timeit add_them(arr, arr)
    1.18 ms +- 14.8 us per loop (mean +- std. dev. of 7 runs, 1000 loops each)
    
    In [146]: %timeit np.add(arr, arr)
    2.8 us +- 64.1 ns per loop (mean +- std. dev. of 7 runs, 100000 loops each)__

在本附录的后面部分，我们将展示如何使用 [Numba 库](http://numba.pydata.org)在 Python 中创建快速的 ufunc。

## A.5 结构化数组与记录数组

你可能已经注意到，到目前为止 ndarray 是一种*同构*（homogeneous）数据容器；也就是说，它表示一个内存块，其中每个元素占用的字节数相同，由数据类型决定。从表面上看，这似乎不允许你表示异构或表格数据。*结构化*数组（structured array）是一种 ndarray，其中每个元素可以被视为表示 C 语言中的一个*结构体*（struct）（因此得名“结构化”），或者类似 SQL 表中具有多个命名字段的一行：

```python
In [147]: dtype = [('x', np.float64), ('y', np.int32)]

In [148]: sarr = np.array([(1.5, 6), (np.pi, -2)], dtype=dtype)

In [149]: sarr
Out[149]: array([(1.5   ,  6), (3.1416, -2)], dtype=[('x', '<f8'), ('y', '<i4')])
```

有几种方法可以指定结构化数据类型（参见 NumPy 在线文档）。一种典型的方式是使用元组列表，每个元组形式为 `(字段名, 字段数据类型)`。现在，数组的元素是类似元组的对象，其元素可以像字典一样访问：

```python
In [150]: sarr[0]
Out[150]: (1.5, 6)

In [151]: sarr[0]['y']
Out[151]: 6
```

字段名称存储在 `dtype.names` 属性中。当你访问结构化数组上的字段时，会返回数据的一个跨步视图（strided view），因此不会复制任何数据：

```python
In [152]: sarr['x']
Out[152]: array([1.5   , 3.1416])
```

### 嵌套数据类型与多维字段

在指定结构化数据类型时，你还可以额外传递一个形状（作为整数或元组）：

```python
In [153]: dtype = [('x', np.int64, 3), ('y', np.int32)]

In [154]: arr = np.zeros(4, dtype=dtype)

In [155]: arr
Out[155]: 
array([([0, 0, 0], 0), ([0, 0, 0], 0), ([0, 0, 0], 0), ([0, 0, 0], 0)],
      dtype=[('x', '<i8', (3,)), ('y', '<i4')])
```

在这种情况下，`x` 字段现在指向每个记录中一个长度为 3 的数组：

```python
In [156]: arr[0]['x']
Out[156]: array([0, 0, 0])
```

方便的是，访问 `arr['x']` 会返回一个二维数组，而不是之前示例中的一维数组：

```python
In [157]: arr['x']
Out[157]: 
array([[0, 0, 0],
       [0, 0, 0],
       [0, 0, 0],
       [0, 0, 0]])
```

这使你能将更复杂的嵌套结构表示为数组中的单个内存块。你还可以嵌套数据类型以创建更复杂的结构。以下是一个示例：

```python
In [158]: dtype = [('x', [('a', 'f8'), ('b', 'f4')]), ('y', np.int32)]

In [159]: data = np.array([((1, 2), 5), ((3, 4), 6)], dtype=dtype)

In [160]: data['x']
Out[160]: array([(1., 2.), (3., 4.)], dtype=[('a', '<f8'), ('b', '<f4')])

In [161]: data['y']
Out[161]: array([5, 6], dtype=int32)

In [162]: data['x']['a']
Out[162]: array([1., 3.])
```

pandas 的 DataFrame 并不以相同方式支持此功能，尽管它类似于分层索引。

### 为何使用结构化数组？

与 pandas DataFrame 相比，NumPy 结构化数组是一种更底层的工具。它们提供了一种将内存块解释为具有嵌套列的表格结构的方法。由于数组中的每个元素在内存中表示为固定字节数，结构化数组提供了一种高效的方式，用于将数据写入磁盘或从磁盘读取（包括内存映射）、通过网络传输数据以及其他类似用途。结构化数组中每个值的内存布局基于 C 编程语言中结构体数据类型的二进制表示。

结构化数组的另一个常见用途是将数据文件写入固定长度的记录字节流，这是在 C 和 C++ 代码中序列化数据的常用方法，有时在工业界的遗留系统中可以找到。只要文件的格式已知（每个记录的大小以及每个元素的顺序、字节大小和数据类型），就可以使用 `np.fromfile` 将数据读入内存。像这样的专业用途超出了本书的范围，但值得了解这些可能性。

## A.6 关于排序的更多内容

与 Python 内置的列表类似，ndarray 的 `sort` 实例方法是一种**原地**（in-place）排序，意味着数组内容会被重新排列而不产生新数组：

```python
In [163]: arr = rng.standard_normal(6)

In [164]: arr.sort()

In [165]: arr
Out[165]: array([-1.1553, -0.9319, -0.5218, -0.4745, -0.1649,  0.03  ])
```

当对数组进行原地排序时，请记住：如果该数组是另一个 ndarray 的视图，则原始数组会被修改：

```python
In [166]: arr = rng.standard_normal((3, 5))

In [167]: arr
Out[167]: 
array([[-1.1956,  0.4691, -0.3598,  1.0359,  0.2267],
       [-0.7448, -0.5931, -1.055 , -0.0683,  0.458 ],
       [-0.07  ,  0.1462, -0.9944,  1.1436,  0.5026]])

In [168]: arr[:, 0].sort()  # 对第一列值进行原地排序

In [169]: arr
Out[169]: 
array([[-1.1956,  0.4691, -0.3598,  1.0359,  0.2267],
       [-0.7448, -0.5931, -1.055 , -0.0683,  0.458 ],
       [-0.07  ,  0.1462, -0.9944,  1.1436,  0.5026]])
```

另一方面，`numpy.sort` 会创建数组的一个新的已排序副本。除此之外，它接受与 ndarray 的 `sort` 方法相同的参数（例如 `kind`）：

```python
In [170]: arr = rng.standard_normal(5)

In [171]: arr
Out[171]: array([ 0.8981, -1.1704, -0.2686, -0.796 ,  1.4522])

In [172]: np.sort(arr)
Out[172]: array([-1.1704, -0.796 , -0.2686,  0.8981,  1.4522])

In [173]: arr
Out[173]: array([ 0.8981, -1.1704, -0.2686, -0.796 ,  1.4522])
```

所有这些排序方法都接受一个 `axis` 参数，用于沿指定轴独立排序数据片段：

```python
In [174]: arr = rng.standard_normal((3, 5))

In [175]: arr
Out[175]: 
array([[-0.2535,  2.1183,  0.3634, -0.6245,  1.1279],
       [ 1.6164, -0.2287, -0.6201, -0.1143, -1.2067],
       [-1.0872, -2.1518, -0.6287, -1.3199,  0.083 ]])

In [176]: arr.sort(axis=1)

In [177]: arr
Out[177]: 
array([[-0.6245, -0.2535,  0.3634,  1.1279,  2.1183],
       [-1.2067, -0.6201, -0.2287, -0.1143,  1.6164],
       [-2.1518, -1.3199, -1.0872, -0.6287,  0.083 ]])
```

你可能会注意到，这些排序方法都没有提供降序排序的选项。这在实践中是一个问题，因为数组切片会产生视图，因此不会产生副本或需要任何计算工作。许多 Python 用户熟悉这样一个“技巧”：对于列表 `values`，`values[::-1]` 会返回一个逆序排列的列表。对于 ndarray 也是如此：

```python
In [178]: arr[:, ::-1]
Out[178]: 
array([[ 2.1183,  1.1279,  0.3634, -0.2535, -0.6245],
       [ 1.6164, -0.1143, -0.2287, -0.6201, -1.2067],
       [ 0.083 , -0.6287, -1.0872, -1.3199, -2.1518]])
```

### 间接排序：argsort 和 lexsort

在数据分析中，你可能需要根据一个或多个键对数据集重新排序。例如，一个包含学生信息的数据表可能需要先按姓氏排序，再按名字排序。这是一个**间接**排序的例子，如果你阅读过与 pandas 相关的章节，你已经见过许多更高级的例子。给定一个或多个键（一个值数组或多个值数组），你希望获得一个整数**索引**数组（我通俗地称之为**索引器**），它告诉你如何重新排序数据以使其处于排序顺序。实现此目的的两个方法是 `argsort` 和 `numpy.lexsort`。例如：

```python
In [179]: values = np.array([5, 0, 1, 3, 2])

In [180]: indexer = values.argsort()

In [181]: indexer
Out[181]: array([1, 2, 4, 3, 0])

In [182]: values[indexer]
Out[182]: array([0, 1, 2, 3, 5])
```

作为一个更复杂的例子，以下代码通过第一行对二维数组进行重新排序：

```python
In [183]: arr = rng.standard_normal((3, 5))

In [184]: arr[0] = values

In [185]: arr
Out[185]: 
array([[ 5.    ,  0.    ,  1.    ,  3.    ,  2.    ],
       [-0.7503, -2.1268, -1.391 , -0.4922,  0.4505],
       [ 0.8926, -1.0479,  0.9553,  0.2936,  0.5379]])

In [186]: arr[:, arr[0].argsort()]
Out[186]: 
array([[ 0.    ,  1.    ,  2.    ,  3.    ,  5.    ],
       [-2.1268, -1.391 ,  0.4505, -0.4922, -0.7503],
       [-1.0479,  0.9553,  0.5379,  0.2936,  0.8926]])
```

`lexsort` 与 `argsort` 类似，但它对多个键数组执行间接的**字典序**排序。假设我们想对一些由名字和姓氏标识的数据进行排序：

```python
In [187]: first_name = np.array(['Bob', 'Jane', 'Steve', 'Bill', 'Barbara'])

In [188]: last_name = np.array(['Jones', 'Arnold', 'Arnold', 'Jones', 'Walters'])

In [189]: sorter = np.lexsort((first_name, last_name))

In [190]: sorter
Out[190]: array([1, 2, 3, 0, 4])

In [191]: list(zip(last_name[sorter], first_name[sorter]))
Out[191]: 
[('Arnold', 'Jane'),
 ('Arnold', 'Steve'),
 ('Jones', 'Bill'),
 ('Jones', 'Bob'),
 ('Walters', 'Barbara')]
```

第一次使用 `lexsort` 时可能会有点困惑，因为用于排序数据的键的顺序是从传递的**最后一个**数组开始的。在这里，`last_name` 在 `first_name` 之前被使用。

### 替代排序算法

**稳定**排序算法会保留相等元素的相对位置。这在间接排序中尤其重要，因为相对顺序是有意义的：

```python
In [192]: values = np.array(['2:first', '2:second', '1:first', '1:second',
       .....:                    '1:third'])

In [193]: key = np.array([2, 2, 1, 1, 1])

In [194]: indexer = key.argsort(kind='mergesort')

In [195]: indexer
Out[195]: array([2, 3, 4, 0, 1])

In [196]: values.take(indexer)
Out[196]: 
array(['1:first', '1:second', '1:third', '2:first', '2:second'],
      dtype='<U8')
```

唯一可用的稳定排序是 **mergesort**，它保证了 `O(n log n)` 的性能，但其平均性能比默认的 quicksort 方法差。有关可用方法及其相对性能（和性能保证）的总结，请参见表 A.3。大多数用户不需要考虑这一点，但知道它的存在是有用的。

表 A.3：数组排序方法
种类 | 速度 | 稳定 | 工作空间 | 最坏情况
---|---|---|---|---
`'quicksort'` | 1 | 否 | 0 | `O(n^2)`
`'mergesort'` | 2 | 是 | `n / 2` | `O(n log n)`
`'heapsort'` | 3 | 否 | 0 | `O(n log n)`

### 部分排序数组

排序的目标之一可能是确定数组中的最大或最小元素。NumPy 有快速的方法 `numpy.partition` 和 `np.argpartition`，用于围绕第 `k` 个最小元素对数组进行分区：

```python
In [197]: rng = np.random.default_rng(12345)

In [198]: arr = rng.standard_normal(20)

In [199]: arr
Out[199]: 
array([-1.4238,  1.2637, -0.8707, -0.2592, -0.0753, -0.7409, -1.3678,
        0.6489,  0.3611, -1.9529,  2.3474,  0.9685, -0.7594,  0.9022,
       -0.467 , -0.0607,  0.7888, -1.2567,  0.5759,  1.399 ])

In [200]: np.partition(arr, 3)
Out[200]: 
array([-1.9529, -1.4238, -1.3678, -1.2567, -0.8707, -0.7594, -0.7409,
       -0.0607,  0.3611, -0.0753, -0.2592, -0.467 ,  0.5759,  0.9022,
        0.9685,  0.6489,  0.7888,  1.2637,  1.399 ,  2.3474])
```

调用 `partition(arr, 3)` 后，结果中的前三个元素是三个最小值，没有特定顺序。`numpy.argpartition` 类似于 `numpy.argsort`，返回将数据重新排列为等效顺序的索引：

```python
In [201]: indices = np.argpartition(arr, 3)

In [202]: indices
Out[202]: 
array([ 9,  0,  6, 17,  2, 12,  5, 15,  8,  4,  3, 14, 18, 13, 11,  7, 16,
        1, 19, 10])

In [203]: arr.take(indices)
Out[203]: 
array([-1.9529, -1.4238, -1.3678, -1.2567, -0.8707, -0.7594, -0.7409,
       -0.0607,  0.3611, -0.0753, -0.2592, -0.467 ,  0.5759,  0.9022,
        0.9685,  0.6489,  0.7888,  1.2637,  1.399 ,  2.3474])
```

### numpy.searchsorted：在排序数组中查找元素

`searchsorted` 是一个数组方法，对已排序的数组执行二分查找，返回在数组中需要插入值以保持排序性的位置：

```python
In [204]: arr = np.array([0, 1, 7, 12, 15])

In [205]: arr.searchsorted(9)
Out[205]: 3
```

你也可以传递一个值数组以获取索引数组：

```python
In [206]: arr.searchsorted([0, 8, 11, 16])
Out[206]: array([0, 3, 3, 5])
```

你可能已经注意到，`searchsorted` 对 `0` 元素返回了 `0`。这是因为默认行为是返回一组相等值左侧的索引：

```python
In [207]: arr = np.array([0, 0, 0, 1, 1, 1, 1])

In [208]: arr.searchsorted([0, 1])
Out[208]: array([0, 3])

In [209]: arr.searchsorted([0, 1], side='right')
Out[209]: array([3, 7])
```

作为 `searchsorted` 的另一个应用，假设我们有一个值在 0 到 10,000 之间的数组，以及一个单独的“桶边缘”数组，我们想用它来分箱数据：

```python
In [210]: data = np.floor(rng.uniform(0, 10000, size=50))

In [211]: bins = np.array([0, 100, 1000, 5000, 10000])

In [212]: data
Out[212]: 
array([ 815., 1598., 3401., 4651., 2664., 8157., 1932., 1294.,  916.,
       5985., 8547., 6016., 9319., 7247., 8605., 9293., 5461., 9376.,
       4949., 2737., 4517., 6650., 3308., 9034., 2570., 3398., 2588.,
       3554.,   50., 6286., 2823.,  680., 6168., 1763., 3043., 4408.,
       1502., 2179., 4743., 4763., 2552., 2975., 2790., 2605., 4827.,
       2119., 4956., 2462., 8384., 1801.])
```

然后，为了获取每个数据点所属的区间标签（其中 1 表示桶 `[0, 100)`），我们可以简单地使用 `searchsorted`：

```python
In [213]: labels = bins.searchsorted(data)

In [214]: labels
Out[214]: 
array([2, 3, 3, 3, 3, 4, 3, 3, 2, 4, 4, 4, 4, 4, 4, 4, 4, 4, 3, 3, 3, 4,
       3, 4, 3, 3, 3, 3, 1, 4, 3, 2, 4, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3, 3,
       3, 3, 3, 3, 4, 3])
```

结合 pandas 的 `groupby`，这可以用于分箱数据：

```python
In [215]: pd.Series(data).groupby(labels).mean()
Out[215]: 
1      50.000000
2     803.666667
3    3079.741935
4    7635.200000
dtype: float64
```

## A.7 使用 Numba 编写快速的 NumPy 函数

[Numba](http://numba.pydata.org) 是一个开源项目，它使用 CPU、GPU 或其他硬件为类 NumPy 数据创建快速函数。它利用 [LLVM 项目](http://llvm.org/) 将 Python 代码转换为编译后的机器码。

为了介绍 Numba，让我们考虑一个纯 Python 函数，它使用 `for` 循环计算表达式 `(x - y).mean()`：

```python
import numpy as np

def mean_distance(x, y):
    nx = len(x)
    result = 0.0
    count = 0
    for i in range(nx):
        result += x[i] - y[i]
        count += 1
    return result / count
```

这个函数运行缓慢：

```python
In [209]: x = rng.standard_normal(10_000_000)

In [210]: y = rng.standard_normal(10_000_000)

In [211]: %timeit mean_distance(x, y)
1 loop, best of 3: 2 s per loop

In [212]: %timeit (x - y).mean()
100 loops, best of 3: 14.7 ms per loop
```

NumPy 版本的速度快了 100 多倍。我们可以使用 `numba.jit` 函数将这个函数转换为编译后的 Numba 函数：

```python
In [213]: import numba as nb

In [214]: numba_mean_distance = nb.jit(mean_distance)
```

我们也可以将其写作装饰器形式：

```python
@nb.jit
def numba_mean_distance(x, y):
    nx = len(x)
    result = 0.0
    count = 0
    for i in range(nx):
        result += x[i] - y[i]
        count += 1
    return result / count
```

得到的函数实际上比向量化的 NumPy 版本更快：

```python
In [215]: %timeit numba_mean_distance(x, y)
100 loops, best of 3: 10.3 ms per loop
```

Numba 无法编译所有纯 Python 代码，但它支持 Python 的一个重要子集，这对于编写数值算法最为有用。

Numba 是一个功能深厚的库，支持不同类型的硬件、编译模式和用户扩展。它还能够编译 NumPy Python API 的大部分子集，而无需显式的 `for` 循环。Numba 能够识别可以编译为机器码的结构，同时对其不知道如何编译的函数替换为调用 CPython API。Numba 的 `jit` 函数选项 `nopython=True` 将允许的代码限制为可以编译为 LLVM 而无需任何 Python C API 调用的 Python 代码。`jit(nopython=True)` 有一个更短的别名 `numba.njit`。

在前面的例子中，我们可以这样写：

```python
from numba import float64, njit

@njit(float64(float64[:], float64[:]))
def mean_distance(x, y):
    return (x - y).mean()
```

我鼓励你通过阅读 [Numba 的在线文档](http://numba.pydata.org/) 来了解更多信息。下一节将展示创建自定义 NumPy ufunc 对象的示例。

### 使用 Numba 创建自定义的 numpy.ufunc 对象

`numba.vectorize` 函数创建编译后的 NumPy ufunc（通用函数），其行为类似于内置的 ufunc。让我们考虑 `numpy.add` 的一个 Python 实现：

```python
from numba import vectorize

@vectorize
def nb_add(x, y):
    return x + y
```

现在我们得到：

```python
In [13]: x = np.arange(10)

In [14]: nb_add(x, x)
Out[14]: array([  0.,   2.,   4.,   6.,   8.,  10.,  12.,  14.,  16.,  18.])

In [15]: nb_add.accumulate(x, 0)
Out[15]: array([  0.,   1.,   3.,   6.,  10.,  15.,  21.,  28.,  36.,  45.])
```

## A.8 高级数组输入和输出

在[第4章：NumPy基础：数组与向量化计算](https://wesmckinney.com/book/numpy-basics)中，我们熟悉了使用 `np.save` 和 `np.load` 将数组以二进制格式存储到磁盘。对于更复杂的应用场景，还有一些额外的选项可供考虑。特别是内存映射（memory maps），它具备一项额外优势：能够对无法放入RAM（内存）的数据集执行某些操作。

### 内存映射文件

_内存映射_文件是一种与磁盘上的二进制数据交互的方法，操作时仿佛数据存储在内存数组中。NumPy实现了一个类似于ndarray的`memmap`对象，使得能够对大型文件的小片段进行读写，而无需将整个数组读入内存。此外，`memmap`拥有与内存数组相同的方法，因此可以在许多原本期望使用ndarray的算法中替代它。

要创建新的内存映射，请使用函数`np.memmap`并传入文件路径、数据类型、形状和文件模式：

```python
In [217]: mmap = np.memmap('mymmap', dtype='float64', mode='w+',
       .....:                  shape=(10000, 10000))

In [218]: mmap
Out[218]: 
memmap([[0., 0., 0., ..., 0., 0., 0.],
        [0., 0., 0., ..., 0., 0., 0.],
        [0., 0., 0., ..., 0., 0., 0.],
        ...,
        [0., 0., 0., ..., 0., 0., 0.],
        [0., 0., 0., ..., 0., 0., 0.],
        [0., 0., 0., ..., 0., 0., 0.]])
```

对`memmap`进行切片操作会返回磁盘上数据的视图：

```python
In [219]: section = mmap[:5]
```

如果向这些切片赋值数据，数据会缓存在内存中，这意味着如果其他应用程序读取该文件，更改不会立即反映到磁盘文件中。任何修改都可以通过调用`flush`方法同步到磁盘：

```python
In [220]: section[:] = rng.standard_normal((5, 10000))

In [221]: mmap.flush()

In [222]: mmap
Out[222]: 
memmap([[-0.9074, -1.0954,  0.0071, ...,  0.2753, -1.1641,  0.8521],
        [-0.0103, -0.0646, -1.0615, ..., -1.1003,  0.2505,  0.5832],
        [ 0.4583,  1.2992,  1.7137, ...,  0.8691, -0.7889, -0.2431],
        ...,
        [ 0.    ,  0.    ,  0.    , ...,  0.    ,  0.    ,  0.    ],
        [ 0.    ,  0.    ,  0.    , ...,  0.    ,  0.    ,  0.    ],
        [ 0.    ,  0.    ,  0.    , ...,  0.    ,  0.    ,  0.    ]])

In [223]: del mmap
```

每当内存映射超出作用域并被垃圾回收时，任何更改也会被刷新到磁盘。_打开现有的内存映射_时，仍然必须指定数据类型和形状，因为文件仅仅是一块二进制数据，不包含任何数据类型信息、形状或步幅（strides）：

```python
In [224]: mmap = np.memmap('mymmap', dtype='float64', shape=(10000, 10000))

In [225]: mmap
Out[225]: 
memmap([[-0.9074, -1.0954,  0.0071, ...,  0.2753, -1.1641,  0.8521],
        [-0.0103, -0.0646, -1.0615, ..., -1.1003,  0.2505,  0.5832],
        [ 0.4583,  1.2992,  1.7137, ...,  0.8691, -0.7889, -0.2431],
        ...,
        [ 0.    ,  0.    ,  0.    , ...,  0.    ,  0.    ,  0.    ],
        [ 0.    ,  0.    ,  0.    , ...,  0.    ,  0.    ,  0.    ],
        [ 0.    ,  0.    ,  0.    , ...,  0.    ,  0.    ,  0.    ]])
```

内存映射也适用于结构化或嵌套数据类型，如“结构化和记录数组”一节所述。

如果您在计算机上运行此示例，可能需要删除上面创建的大文件：

```python
In [226]: %xdel mmap

In [227]: !rm mymmap
```

### HDF5及其他数组存储选项

PyTables和h5py是两个Python项目，它们提供了对NumPy友好的接口，用于以高效且可压缩的HDF5格式（HDF代表_分层数据格式_）存储数组数据。您可以安全地将数百GB甚至TB的数据存储在HDF5格式中。要了解更多关于在Python中使用HDF5的信息，我建议阅读[pandas在线文档](http://pandas.pydata.org)。

## A.9 性能优化技巧

将数据处理代码适配为使用 NumPy 通常能显著提升运行速度，因为数组操作通常可以替代相对而言极其缓慢的纯 Python 循环。以下是一些有助于从该库中获取最佳性能的技巧：

*   将 Python 循环和条件逻辑转换为数组操作和布尔数组操作。
*   尽可能使用广播（broadcasting）机制。
*   使用数组视图（切片）来避免复制数据。
*   利用 ufuncs（通用函数）及其方法。

如果你在充分使用 NumPy 提供的能力后，仍无法获得所需的性能，可以考虑用 C、FORTRAN 或 Cython 编写代码。我在自己的工作中经常使用 [Cython](http://cython.org)，以此获得类似 C 语言的性能，同时通常所需的开发时间要少得多。

### 连续内存的重要性

虽然这个话题的完整内容有点超出本书的范围，但在某些应用中，数组的内存布局会显著影响计算速度。这部分原因与 CPU 的缓存层次结构有关的性能差异：访问连续内存块的操作（例如，对按 C 顺序存储的数组的行求和）通常是最快的，因为内存子系统会将适当的内存块缓冲到低延迟的 L1 或 L2 CPU 缓存中。此外，NumPy 的 C 代码库中的某些代码路径已针对连续内存的情况进行了优化，这样可以避免通用的跨步内存访问。

说一个数组的内存布局是*连续的*，意味着元素在内存中存储的顺序与它们在数组中出现的顺序一致，遵循 FORTRAN（列优先）或 C（行优先）顺序。默认情况下，NumPy 数组创建为 C 连续或简称为连续。一个列优先数组，例如 C 连续数组的转置，因此被称为 FORTRAN 连续。可以通过 ndarray 的 `flags` 属性显式检查这些属性：

```python
In [228]: arr_c = np.ones((1000, 10000), order='C')

In [229]: arr_f = np.ones((1000, 10000), order='F')

In [230]: arr_c.flags
Out[230]: 
  C_CONTIGUOUS : True
  F_CONTIGUOUS : False
  OWNDATA : True
  WRITEABLE : True
  ALIGNED : True
  WRITEBACKIFCOPY : False

In [231]: arr_f.flags
Out[231]: 
  C_CONTIGUOUS : False
  F_CONTIGUOUS : True
  OWNDATA : True
  WRITEABLE : True
  ALIGNED : True
  WRITEBACKIFCOPY : False

In [232]: arr_f.flags.f_contiguous
Out[232]: True
```

在这个例子中，对这些数组的行进行求和，理论上 `arr_c` 应该比 `arr_f` 更快，因为它的行在内存中是连续的。这里，我在 IPython 中使用 `%timeit` 进行检查（结果可能因你的机器而异）：

```python
In [233]: %timeit arr_c.sum(1)
199 µs ± 1.18 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)

In [234]: %timeit arr_f.sum(1)
371 µs ± 6.77 µs per loop (mean ± std. dev. of 7 runs, 1000 loops each)
```

当你希望从 NumPy 中榨取更多性能时，这通常是一个值得投入一些精力的地方。如果你有一个数组不具有所需的内存顺序，可以使用 `copy` 并传入 `'C'` 或 `'F'`：

```python
In [235]: arr_f.copy('C').flags
Out[235]: 
  C_CONTIGUOUS : True
  F_CONTIGUOUS : False
  OWNDATA : True
  WRITEABLE : True
  ALIGNED : True
  WRITEBACKIFCOPY : False
```

在对数组创建视图时，请记住，结果不保证是连续的：

```python
In [236]: arr_c[:50].flags.contiguous
Out[236]: True

In [237]: arr_c[:, :50].flags
Out[237]: 
  C_CONTIGUOUS : False
  F_CONTIGUOUS : False
  OWNDATA : False
  WRITEABLE : True
  ALIGNED : True
  WRITEBACKIFCOPY : False
```

* * *

1.  一些数据类型的名称中带有下划线后缀。这是为了避免 NumPy 特定类型与 Python 内置类型之间的变量名冲突。↩︎