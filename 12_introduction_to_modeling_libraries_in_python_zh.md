# 12 Python中的建模库简介

__

《利用 Python 进行数据分析 第三版》的开放获取网络版现作为[印刷版与电子版](https://amzn.to/3DyLaJc)的配套资源提供。如果您发现任何勘误，[请在此处提交](https://oreilly.com/catalog/0636920519829/errata)。请注意，由 Quarto 生成的此站点部分样式会与 O'Reilly 出版的印刷版及电子书版本有所不同。

若您觉得此在线版本有帮助，请考虑[订购纸质版](https://amzn.to/3DyLaJc)或[无DRM限制的电子书](https://www.ebooks.com/en-us/book/210644288/python-for-data-analysis/wes-mckinney/?affId=WES398681F)以支持作者。本站内容不可复制或转载。代码示例遵循 MIT 许可协议，可在 GitHub 或 Gitee 上获取。

本书着重为使用 Python 进行数据分析提供编程基础。由于数据分析师和科学家经常需要花费大量时间进行数据整理（data wrangling）和预处理，本书的结构体现了掌握这些技术的重要性。

选择何种库进行模型开发取决于具体应用场景。许多统计问题可以通过普通最小二乘回归等简单技术解决，而其他问题可能需要更先进的机器学习方法。幸运的是，Python 已成为实现分析方法的首选语言之一，因此在读完本书后您还可以探索更多工具。

本章将回顾 pandas 的一些特性，这些特性在数据整理（data wrangling）与模型拟合、评分之间交叉工作时可能很有帮助。随后将简要介绍两个流行的建模工具包：[statsmodels](http://statsmodels.org) 和 [scikit-learn](http://scikit-learn.org)。由于这两个项目的规模都足以单独成书，本章不会追求全面性，而是引导读者查阅项目的在线文档以及其他关于数据科学、统计学和机器学习的 Python 相关书籍。

## 12.1 pandas 与模型代码的接口交互

模型开发的常见工作流程是：先使用 pandas 进行数据加载和清洗，再切换到建模库来构建模型本身。模型开发过程中一个重要环节在机器学习中被称为_特征工程（feature engineering）_。这可以指代任何从原始数据集中提取信息的数据转换或分析操作，这些信息可能在建模场景中有用。本书中探讨的数据聚合和 GroupBy 工具经常用于特征工程场景。

虽然"优秀"特征工程的细节超出了本书范围，但我将展示一些方法，使 pandas 数据操作与建模之间的切换尽可能无缝。

pandas 与其他分析库的交互点通常是 NumPy 数组。要将 DataFrame 转换为 NumPy 数组，可使用 `to_numpy` 方法：

```python
In [12]: data = pd.DataFrame({
   ....:     'x0': [1, 2, 3, 4, 5],
   ....:     'x1': [0.01, -0.01, 0.25, -4.1, 0.],
   ....:     'y': [-1.5, 0., 3.6, 1.3, -2.]})

In [13]: data
Out[13]: 
   x0    x1    y
0   1  0.01 -1.5
1   2 -0.01  0.0
2   3  0.25  3.6
3   4 -4.10  1.3
4   5  0.00 -2.0

In [14]: data.columns
Out[14]: Index(['x0', 'x1', 'y'], dtype='object')

In [15]: data.to_numpy()
Out[15]: 
array([[ 1.  ,  0.01, -1.5 ],
       [ 2.  , -0.01,  0.  ],
       [ 3.  ,  0.25,  3.6 ],
       [ 4.  , -4.1 ,  1.3 ],
       [ 5.  ,  0.  , -2.  ]])
```

如前面章节所述，若要转换回 DataFrame，可以传递一个二维 ndarray 并可选指定列名：

```python
In [16]: df2 = pd.DataFrame(data.to_numpy(), columns=['one', 'two', 'three'])

In [17]: df2
Out[17]: 
   one   two  three
0  1.0  0.01   -1.5
1  2.0 -0.01    0.0
2  3.0  0.25    3.6
3  4.0 -4.10    1.3
4  5.0  0.00   -2.0
```

`to_numpy` 方法适用于数据为同构类型（例如全为数值类型）的情况。如果数据是异构的，结果将是一个 Python 对象的 ndarray：

```python
In [18]: df3 = data.copy()

In [19]: df3['strings'] = ['a', 'b', 'c', 'd', 'e']

In [20]: df3
Out[20]: 
   x0    x1    y strings
0   1  0.01 -1.5       a
1   2 -0.01  0.0       b
2   3  0.25  3.6       c
3   4 -4.10  1.3       d
4   5  0.00 -2.0       e

In [21]: df3.to_numpy()
Out[21]: 
array([[1, 0.01, -1.5, 'a'],
       [2, -0.01, 0.0, 'b'],
       [3, 0.25, 3.6, 'c'],
       [4, -4.1, 1.3, 'd'],
       [5, 0.0, -2.0, 'e']], dtype=object)
```

对于某些模型，可能只需要使用列的子集。建议结合使用 `loc` 索引和 `to_numpy`：

```python
In [22]: model_cols = ['x0', 'x1']

In [23]: data.loc[:, model_cols].to_numpy()
Out[23]: 
array([[ 1.  ,  0.01],
       [ 2.  , -0.01],
       [ 3.  ,  0.25],
       [ 4.  , -4.1 ],
       [ 5.  ,  0.  ]])
```

有些库原生支持 pandas，会自动完成部分工作：例如将 DataFrame 转换为 NumPy 数组，并将模型参数名称附加到输出表或 Series 的列上。其他情况下，则需要手动执行这种"元数据管理"。

在[第 7.5 章：分类数据](https://wesmckinney.com/book/data-cleaning#pandas-categorical)中，我们介绍了 pandas 的 `Categorical` 类型和 `pandas.get_dummies` 函数。假设示例数据集中有一个非数值列：

```python
In [24]: data['category'] = pd.Categorical(['a', 'b', 'a', 'a', 'b'],
   ....:                                   categories=['a', 'b'])

In [25]: data
Out[25]: 
   x0    x1    y category
0   1  0.01 -1.5        a
1   2 -0.01  0.0        b
2   3  0.25  3.6        a
3   4 -4.10  1.3        a
4   5  0.00 -2.0        b
```

若要用虚拟变量（dummy variables）替换 `'category'` 列，可以创建虚拟变量，删除 `'category'` 列，然后连接结果：

```python
In [26]: dummies = pd.get_dummies(data.category, prefix='category',
   ....:                          dtype=float)

In [27]: data_with_dummies = data.drop('category', axis=1).join(dummies)

In [28]: data_with_dummies
Out[28]: 
   x0    x1    y  category_a  category_b
0   1  0.01 -1.5         1.0         0.0
1   2 -0.01  0.0         0.0         1.0
2   3  0.25  3.6         1.0         0.0
3   4 -4.10  1.3         1.0         0.0
4   5  0.00 -2.0         0.0         1.0
```

使用虚拟变量拟合某些统计模型时存在一些细节问题。当数据不仅仅包含简单数值列时，使用 Patsy（下一节的主题）可能更简单且更不易出错。

## 12.2 使用 Patsy 创建模型描述

[Patsy](https://patsy.readthedocs.io/) 是一个 Python 库，用于通过基于字符串的"公式语法"描述统计模型（特别是线性模型）。其语法灵感来源于（但并不完全相同）R 和 S 统计编程语言中使用的公式语法。当你安装 statsmodels 时，它会自动安装：
    
    
    conda install statsmodels

Patsy 在 statsmodels 中得到了良好支持，可用于指定线性模型，因此我将重点介绍一些主要功能以帮助你快速上手。Patsy 的 _公式（formulas）_ 是一种特殊的字符串语法，形式如下：
    
    
    y ~ x0 + x1

语法 `a + b` 并非表示将 `a` 与 `b` 相加，而是指这些是为此模型创建的 _设计矩阵（design matrix）_ 中的 _项（terms）_。`patsy.dmatrices` 函数接收一个公式字符串和一个数据集（可以是 DataFrame 或数组字典），并为线性模型生成设计矩阵：
    
    
    In [29]: data = pd.DataFrame({
       ....:     'x0': [1, 2, 3, 4, 5],
       ....:     'x1': [0.01, -0.01, 0.25, -4.1, 0.],
       ....:     'y': [-1.5, 0., 3.6, 1.3, -2.]})
    
    In [30]: data
    Out[30]: 
       x0    x1    y
    0   1  0.01 -1.5
    1   2 -0.01  0.0
    2   3  0.25  3.6
    3   4 -4.10  1.3
    4   5  0.00 -2.0
    
    In [31]: import patsy
    
    In [32]: y, X = patsy.dmatrices('y ~ x0 + x1', data)__

现在我们得到：
    
    
    In [33]: y
    Out[33]: 
    DesignMatrix with shape (5, 1)
         y
      -1.5
       0.0
       3.6
       1.3
      -2.0
      Terms:
        'y' (column 0)
    
    In [34]: X
    Out[34]: 
    DesignMatrix with shape (5, 3)
      Intercept  x0     x1
              1   1   0.01
              1   2  -0.01
              1   3   0.25
              1   4  -4.10
              1   5   0.00
      Terms:
        'Intercept' (column 0)
        'x0' (column 1)
        'x1' (column 2)__

这些 Patsy 的 `DesignMatrix` 实例是带有额外元数据的 NumPy ndarrays：
    
    
    In [35]: np.asarray(y)
    Out[35]: 
    array([[-1.5],
           [ 0. ],
           [ 3.6],
           [ 1.3],
           [-2. ]])
    
    In [36]: np.asarray(X)
    Out[36]: 
    array([[ 1.  ,  1.  ,  0.01],
           [ 1.  ,  2.  , -0.01],
           [ 1.  ,  3.  ,  0.25],
           [ 1.  ,  4.  , -4.1 ],
           [ 1.  ,  5.  ,  0.  ]])__

你可能会好奇 `Intercept` 项是从哪里来的。这是普通最小二乘（OLS）回归等线性模型的惯例。你可以通过在模型中添加 `+ 0` 项来抑制截距项：
    
    
    In [37]: patsy.dmatrices('y ~ x0 + x1 + 0', data)[1]
    Out[37]: 
    DesignMatrix with shape (5, 2)
      x0     x1
       1   0.01
       2  -0.01
       3   0.25
       4  -4.10
       5   0.00
      Terms:
        'x0' (column 0)
        'x1' (column 1)__

Patsy 对象可以直接传递给算法，如 `numpy.linalg.lstsq`，它执行普通最小二乘回归：
    
    
    In [38]: coef, resid, _, _ = np.linalg.lstsq(X, y, rcond=None)__

模型元数据保留在 `design_info` 属性中，因此你可以将模型列名重新附加到拟合系数上，例如得到一个 Series：
    
    
    In [39]: coef
    Out[39]: 
    array([[ 0.3129],
           [-0.0791],
           [-0.2655]])
    
    In [40]: coef = pd.Series(coef.squeeze(), index=X.design_info.column_names)
    
    In [41]: coef
    Out[41]: 
    Intercept    0.312910
    x0          -0.079106
    x1          -0.265464
    dtype: float64 __

### Patsy 公式中的数据转换

你可以在 Patsy 公式中混入 Python 代码；在评估公式时，库将尝试在封闭作用域中找到你使用的函数：
    
    
    In [42]: y, X = patsy.dmatrices('y ~ x0 + np.log(np.abs(x1) + 1)', data)
    
    In [43]: X
    Out[43]: 
    DesignMatrix with shape (5, 3)
      Intercept  x0  np.log(np.abs(x1) + 1)
              1   1                 0.00995
              1   2                 0.00995
              1   3                 0.22314
              1   4                 1.62924
              1   5                 0.00000
      Terms:
        'Intercept' (column 0)
        'x0' (column 1)
        'np.log(np.abs(x1) + 1)' (column 2)__

一些常用的变量转换包括 _标准化（standardizing）_（均值为 0，方差为 1）和 _中心化（centering）_（减去均值）。Patsy 为此提供了内置函数：
    
    
    In [44]: y, X = patsy.dmatrices('y ~ standardize(x0) + center(x1)', data)
    
    In [45]: X
    Out[45]: 
    DesignMatrix with shape (5, 3)
      Intercept  standardize(x0)  center(x1)
              1         -1.41421        0.78
              1         -0.70711        0.76
              1          0.00000        1.02
              1          0.70711       -3.33
              1          1.41421        0.77
      Terms:
        'Intercept' (column 0)
        'standardize(x0)' (column 1)
        'center(x1)' (column 2)__

作为建模过程的一部分，你可能在一个数据集上拟合模型，然后基于另一个数据集评估模型。这可能是 _保留（hold-out）_ 部分或后来观测到的新数据。当应用诸如中心化和标准化等转换时，在使用模型基于新数据进行预测时应格外小心。这些被称为 _有状态（stateful）_ 转换，因为在转换新数据集时，你必须使用原始数据集的统计量（如均值或标准差）。

`patsy.build_design_matrices` 函数可以使用原始 _样本内（in-sample）_ 数据集保存的信息，将转换应用于新的 _样本外（out-of-sample）_ 数据：
    
    
    In [46]: new_data = pd.DataFrame({
       ....:     'x0': [6, 7, 8, 9],
       ....:     'x1': [3.1, -0.5, 0, 2.3],
       ....:     'y': [1, 2, 3, 4]})
    
    In [47]: new_X = patsy.build_design_matrices([X.design_info], new_data)
    
    In [48]: new_X
    Out[48]: 
    [DesignMatrix with shape (4, 3)
       Intercept  standardize(x0)  center(x1)
               1          2.12132        3.87
               1          2.82843        0.27
               1          3.53553        0.77
               1          4.24264        3.07
       Terms:
         'Intercept' (column 0)
         'standardize(x0)' (column 1)
         'center(x1)' (column 2)]__

由于在 Patsy 公式中加号（`+`）并不表示加法运算，当你想要按名称添加数据集中的列时，必须将它们包装在特殊的 `I` 函数中：
    
    
    In [49]: y, X = patsy.dmatrices('y ~ I(x0 + x1)', data)
    
    In [50]: X
    Out[50]: 
    DesignMatrix with shape (5, 2)
      Intercept  I(x0 + x1)
              1        1.01
              1        1.99
              1        3.25
              1       -0.10
              1        5.00
      Terms:
        'Intercept' (column 0)
        'I(x0 + x1)' (column 1)__

Patsy 在 `patsy.builtins` 模块中还有其他几个内置转换。更多内容请参阅在线文档。

分类数据有一类特殊的转换，我将在接下来进行解释。

### 分类数据与 Patsy

非数值数据可以通过多种不同的方式转换为模型设计矩阵。对此主题的完整讨论超出了本书的范围，最好结合统计学课程进行学习。

当你在 Patsy 公式中使用非数值项时，默认情况下它们会被转换为虚拟变量（dummy variables）。如果存在截距项，其中一个水平将被排除以避免共线性：
    
    
    In [51]: data = pd.DataFrame({
       ....:     'key1': ['a', 'a', 'b', 'b', 'a', 'b', 'a', 'b'],
       ....:     'key2': [0, 1, 0, 1, 0, 1, 0, 0],
       ....:     'v1': [1, 2, 3, 4, 5, 6, 7, 8],
       ....:     'v2': [-1, 0, 2.5, -0.5, 4.0, -1.2, 0.2, -1.7]
       ....: })
    
    In [52]: y, X = patsy.dmatrices('v2 ~ key1', data)
    
    In [53]: X
    Out[53]: 
    DesignMatrix with shape (8, 2)
      Intercept  key1[T.b]
              1          0
              1          0
              1          1
              1          1
              1          0
              1          1
              1          0
              1          1
      Terms:
        'Intercept' (column 0)
        'key1' (column 1)__

如果你从模型中省略截距，那么每个类别值的列都将包含在模型设计矩阵中：
    
    
    In [54]: y, X = patsy.dmatrices('v2 ~ key1 + 0', data)
    
    In [55]: X
    Out[55]: 
    DesignMatrix with shape (8, 2)
      key1[a]  key1[b]
            1        0
            1        0
            0        1
            0        1
            1        0
            0        1
            1        0
            0        1
      Terms:
        'key1' (columns 0:2)__

数值列可以通过 `C` 函数被解释为分类数据：
    
    
    In [56]: y, X = patsy.dmatrices('v2 ~ C(key2)', data)
    
    In [57]: X
    Out[57]: 
    DesignMatrix with shape (8, 2)
      Intercept  C(key2)[T.1]
              1             0
              1             1
              1             0
              1             1
              1             0
              1             1
              1             0
              1             0
      Terms:
        'Intercept' (column 0)
        'C(key2)' (column 1)__

当在模型中使用多个分类项时，情况可能会更复杂，因为你可以包含形式为 `key1:key2` 的交互项，例如可用于方差分析（ANOVA）模型：
    
    
    In [58]: data['key2'] = data['key2'].map({0: 'zero', 1: 'one'})
    
    In [59]: data
    Out[59]: 
      key1  key2  v1   v2
    0    a  zero   1 -1.0
    1    a   one   2  0.0
    2    b  zero   3  2.5
    3    b   one   4 -0.5
    4    a  zero   5  4.0
    5    b   one   6 -1.2
    6    a  zero   7  0.2
    7    b  zero   8 -1.7
    
    In [60]: y, X = patsy.dmatrices('v2 ~ key1 + key2', data)
    
    In [61]: X
    Out[61]: 
    DesignMatrix with shape (8, 3)
      Intercept  key1[T.b]  key2[T.zero]
              1          0             1
              1          0             0
              1          1             1
              1          1             0
              1          0             1
              1          1             0
              1          0             1
              1          1             1
      Terms:
        'Intercept' (column 0)
        'key1' (column 1)
        'key2' (column 2)
    
    In [62]: y, X = patsy.dmatrices('v2 ~ key1 + key2 + key1:key2', data)
    
    In [63]: X
    Out[63]: 
    DesignMatrix with shape (8, 4)
      Intercept  key1[T.b]  key2[T.zero]  key1[T.b]:key2[T.zero]
              1          0             1                       0
              1          0             0                       0
              1          1             1                       1
              1          1             0                       0
              1          0             1                       0
              1          1             0                       0
              1          0             1                       0
              1          1             1                       1
      Terms:
        'Intercept' (column 0)
        'key1' (column 1)
        'key2' (column 2)
        'key1:key2' (column 3)__

Patsy 提供了其他转换分类数据的方法，包括对具有特定顺序的项进行转换。更多内容请参阅在线文档。

## 12.3 statsmodels 简介

[statsmodels](http://www.statsmodels.org) 是一个用于拟合多种统计模型、执行统计检验以及进行数据探索和可视化的 Python 库。statsmodels 包含更多"经典"的频率派统计方法，而贝叶斯方法和机器学习模型则存在于其他库中。

statsmodels 中包含的一些模型类型包括：

* 线性模型、广义线性模型和稳健线性模型
* 线性混合效应模型
* 方差分析（ANOVA）方法
* 时间序列过程和状态空间模型
* 广义矩估计方法

在接下来的几页中，我们将使用 statsmodels 中的一些基本工具，并探索如何将建模接口与 Patsy 公式和 pandas DataFrame 对象结合使用。如果您之前没有在讨论 Patsy 时安装 statsmodels，现在可以通过以下方式安装：

```bash
conda install statsmodels
```

### 估计线性模型

statsmodels 中有几种线性回归模型，从较为基础的（例如普通最小二乘法）到更复杂的（例如迭代重加权最小二乘法）。

statsmodels 中的线性模型有两个不同的主要接口：基于数组的和基于公式的。可以通过以下 API 模块导入来访问它们：

```python
import statsmodels.api as sm
import statsmodels.formula.api as smf
```

为了展示如何使用这些接口，我们使用一些随机数据生成一个线性模型。在 Jupyter 单元格中运行以下代码：

```python
# 为使示例可重现
rng = np.random.default_rng(seed=12345)

def dnorm(mean, variance, size=1):
    if isinstance(size, int):
        size = size,
    return mean + np.sqrt(variance) * rng.standard_normal(*size)

N = 100
X = np.c_[dnorm(0, 0.4, size=N),
          dnorm(0, 0.6, size=N),
          dnorm(0, 0.2, size=N)]
eps = dnorm(0, 0.1, size=N)
beta = [0.1, 0.3, 0.5]

y = np.dot(X, beta) + eps
```

这里，我写下了具有已知参数 `beta` 的"真实"模型。在这种情况下，`dnorm` 是一个辅助函数，用于生成具有特定均值和方差的正态分布数据。所以现在我们得到：

```python
In [66]: X[:5]
Out[66]: 
array([[-0.9005, -0.1894, -1.0279],
       [ 0.7993, -1.546 , -0.3274],
       [-0.5507, -0.1203,  0.3294],
       [-0.1639,  0.824 ,  0.2083],
       [-0.0477, -0.2131, -0.0482]])

In [67]: y[:5]
Out[67]: array([-0.5995, -0.5885,  0.1856, -0.0075, -0.0154])
```

线性模型通常包含一个截距项，正如我们之前在 Patsy 中看到的那样。`sm.add_constant` 函数可以向现有矩阵添加一个截距列：

```python
In [68]: X_model = sm.add_constant(X)

In [69]: X_model[:5]
Out[69]: 
array([[ 1.    , -0.9005, -0.1894, -1.0279],
       [ 1.    ,  0.7993, -1.546 , -0.3274],
       [ 1.    , -0.5507, -0.1203,  0.3294],
       [ 1.    , -0.1639,  0.824 ,  0.2083],
       [ 1.    , -0.0477, -0.2131, -0.0482]])
```

`sm.OLS` 类可以拟合一个普通最小二乘线性回归：

```python
In [70]: model = sm.OLS(y, X)
```

模型的 `fit` 方法返回一个回归结果对象，其中包含估计的模型参数和其他诊断信息：

```python
In [71]: results = model.fit()

In [72]: results.params
Out[72]: array([0.0668, 0.268 , 0.4505])
```

`results` 上的 `summary` 方法可以打印一个包含模型详细诊断输出的模型摘要：

```python
In [73]: print(results.summary())
OLS Regression Results                                
=================================================================================
======
Dep. Variable:                      y   R-squared (uncentered):                  
 0.469
Model:                            OLS   Adj. R-squared (uncentered):             
 0.452
Method:                 Least Squares   F-statistic:                             
 28.51
Date:                Wed, 12 Apr 2023   Prob (F-statistic):                    2.
66e-13
Time:                        13:09:20   Log-Likelihood:                         -
25.611
No. Observations:                 100   AIC:                                     
57.22
Df Residuals:                      97   BIC:                                     
65.04
Df Model:                           3                                            
      
Covariance Type:            nonrobust                                            
      
==============================================================================
               coef    std err          t      P>|t|      [0.025      0.975]
------------------------------------------------------------------------------
x1             0.0668      0.054      1.243      0.217      -0.040       0.174
x2             0.2680      0.042      6.313      0.000       0.184       0.352
x3             0.4505      0.068      6.605      0.000       0.315       0.586
==============================================================================
Omnibus:                        0.435   Durbin-Watson:                   1.869
Prob(Omnibus):                  0.805   Jarque-Bera (JB):                0.301
Skew:                           0.134   Prob(JB):                        0.860
Kurtosis:                       2.995   Cond. No.                         1.64
==============================================================================
Notes:
[1] R² is computed without centering (uncentered) since the model does not contai
n a constant.
[2] Standard Errors assume that the covariance matrix of the errors is correctly 
specified.
```

这里的参数名称被赋予了通用名称 `x1`、`x2` 等。假设所有模型参数都在一个 DataFrame 中：

```python
In [74]: data = pd.DataFrame(X, columns=['col0', 'col1', 'col2'])

In [75]: data['y'] = y

In [76]: data[:5]
Out[76]: 
       col0      col1      col2         y
0 -0.900506 -0.189430 -1.027870 -0.599527
1  0.799252 -1.545984 -0.327397 -0.588454
2 -0.550655 -0.120254  0.329359  0.185634
3 -0.163916  0.824040  0.208275 -0.007477
4 -0.047651 -0.213147 -0.048244 -0.015374
```

现在我们可以使用 statsmodels 的公式 API 和 Patsy 公式字符串：

```python
In [77]: results = smf.ols('y ~ col0 + col1 + col2', data=data).fit()

In [78]: results.params
Out[78]: 
Intercept   -0.020799
col0         0.065813
col1         0.268970
col2         0.449419
dtype: float64

In [79]: results.tvalues
Out[79]: 
Intercept   -0.652501
col0         1.219768
col1         6.312369
col2         6.567428
dtype: float64
```

注意 statsmodels 如何将结果作为带有 DataFrame 列名的 Series 返回。在使用公式和 pandas 对象时，我们也不需要使`add_constant`。

给定新的样本外数据，您可以根据估计的模型参数计算预测值：

```python
In [80]: results.predict(data[:5])
Out[80]: 
0   -0.592959
1   -0.531160
2    0.058636
3    0.283658
4   -0.102947
dtype: float64
```

statsmodels 中还有许多用于线性模型结果的分析、诊断和可视化的附加工具，您可以探索。除了普通最小二乘法之外，还有其他类型的线性模型。

### 估计时间序列过程

statsmodels 中的另一类模型用于时间序列分析。其中包括自回归过程、卡尔曼滤波和其他状态空间模型，以及多元自回归模型。

让我们模拟一些具有自回归结构和噪声的时间序列数据。在 Jupyter 中运行以下代码：

```python
init_x = 4

values = [init_x, init_x]
N = 1000

b0 = 0.8
b1 = -0.4
noise = dnorm(0, 0.1, N)
for i in range(N):
    new_x = values[-1] * b0 + values[-2] * b1 + noise[i]
    values.append(new_x)
```

该数据具有 AR(2) 结构（两个滞后项），参数为 `0.8` 和 `–0.4`。当您拟合 AR 模型时，您可能不知道要包含多少个滞后项，因此您可以用较大的滞后数来拟合模型：

```python
In [82]: from statsmodels.tsa.ar_model import AutoReg

In [83]: MAXLAGS = 5

In [84]: model = AutoReg(values, MAXLAGS)

In [85]: results = model.fit()
```

结果中的估计参数首先是截距，然后是前两个滞后的估计：

```python
In [86]: results.params
Out[86]: array([ 0.0235,  0.8097, -0.4287, -0.0334,  0.0427, -0.0567])
```

这些模型的更深入细节以及如何解释它们的结果超出了本书的范围，但 statsmodels 文档中还有更多内容可供探索。

## 12.4 scikit-learn 简介

[scikit-learn](http://scikit-learn.org) 是最广泛使用且备受信赖的通用 Python 机器学习工具包之一。它包含了大量标准的监督式和非监督式机器学习方法，并提供了模型选择与评估、数据转换、数据加载以及模型持久化等工具。这些模型可用于分类、聚类、预测及其他常见任务。你可以通过 conda 安装 scikit-learn，如下所示：

```bash
conda install scikit-learn
```

目前有大量优秀的在线和印刷资源可供学习机器学习知识，以及了解如何应用 scikit-learn 等库解决实际问题。在本节中，我将简要介绍 scikit-learn 的 API 风格。

近年来，scikit-learn 与 pandas 的集成已有显著改进，等到你阅读本文时，可能还会有更大提升。建议你查阅最新的项目文档。

本章示例采用了来自 Kaggle 竞赛的[一个经典数据集](https://www.kaggle.com/c/titanic)，该数据集记录了 1912 年泰坦尼克号上乘客的生存率。我们使用 pandas 加载训练和测试数据集：

```python
In [87]: train = pd.read_csv('datasets/titanic/train.csv')

In [88]: test = pd.read_csv('datasets/titanic/test.csv')

In [89]: train.head(4)
Out[89]: 
   PassengerId  Survived  Pclass   
0            1         0       3  \
1            2         1       1   
2            3         1       3   
3            4         1       1   
                                                  Name     Sex   Age  SibSp   
0                            Braund, Mr. Owen Harris    male  22.0      1  \
1  Cumings, Mrs. John Bradley (Florence Briggs Thayer)  female  38.0      1   
2                             Heikkinen, Miss. Laina  female  26.0      0   
3         Futrelle, Mrs. Jacques Heath (Lily May Peel)  female  35.0      1   
   Parch            Ticket     Fare Cabin Embarked  
0      0         A/5 21171   7.2500   NaN        S  
1      0          PC 17599  71.2833   C85        C  
2      0  STON/O2. 3101282   7.9250   NaN        S  
3      0            113803  53.1000  C123        S  
```

statsmodels 和 scikit-learn 等库通常无法处理缺失数据，因此我们需要检查各列是否存在缺失值：

```python
In [90]: train.isna().sum()
Out[90]: 
PassengerId      0
Survived         0
Pclass           0
Name             0
Sex              0
Age            177
SibSp            0
Parch            0
Ticket           0
Fare             0
Cabin          687
Embarked         2
dtype: int64

In [91]: test.isna().sum()
Out[91]: 
PassengerId      0
Pclass           0
Name             0
Sex              0
Age             86
SibSp            0
Parch            0
Ticket           0
Fare             1
Cabin          327
Embarked         0
dtype: int64
```

在此类统计和机器学习示例中，典型任务是根据数据特征预测乘客是否幸存。模型会在训练数据集上拟合，然后在样本外测试数据集上进行评估。

虽然我想使用 `Age` 作为预测特征，但它存在缺失数据。处理缺失数据插补的方法有很多，这里采用简单方式：使用训练数据集的中位数填充两个表中的空值：

```python
In [92]: impute_value = train['Age'].median()

In [93]: train['Age'] = train['Age'].fillna(impute_value)

In [94]: test['Age'] = test['Age'].fillna(impute_value)
```

现在需要指定模型。我添加了 `IsFemale` 列作为 `'Sex'` 列的编码版本：

```python
In [95]: train['IsFemale'] = (train['Sex'] == 'female').astype(int)

In [96]: test['IsFemale'] = (test['Sex'] == 'female').astype(int)
```

接着确定模型变量并创建 NumPy 数组：

```python
In [97]: predictors = ['Pclass', 'IsFemale', 'Age']

In [98]: X_train = train[predictors].to_numpy()

In [99]: X_test = test[predictors].to_numpy()

In [100]: y_train = train['Survived'].to_numpy()

In [101]: X_train[:5]
Out[101]: 
array([[ 3.,  0., 22.],
       [ 1.,  1., 38.],
       [ 3.,  1., 26.],
       [ 1.,  1., 35.],
       [ 3.,  0., 35.]])

In [102]: y_train[:5]
Out[102]: array([0, 1, 1, 1, 0])
```

需要说明的是，这并不代表这是一个优质模型，也不表示这些特征经过恰当处理。我们使用 scikit-learn 的 `LogisticRegression` 模型创建一个模型实例：

```python
In [103]: from sklearn.linear_model import LogisticRegression

In [104]: model = LogisticRegression()
```

通过模型的 `fit` 方法将模型拟合到训练数据：

```python
In [105]: model.fit(X_train, y_train)
Out[105]: LogisticRegression()
```

现在可以使用 `model.predict` 对测试数据集进行预测：

```python
In [106]: y_predict = model.predict(X_test)

In [107]: y_predict[:10]
Out[107]: array([0, 0, 0, 0, 1, 0, 1, 0, 1, 0])
```

如果获得测试数据集的真实值，可以计算准确率或其他误差指标：

```python
(y_true == y_predict).mean()
```

实践中，模型训练通常包含更多复杂层次。许多模型具有可调参数，而像交叉验证（cross-validation）这样的技术可用于参数调优，以避免对训练数据的过拟合。这通常能提升新数据上的预测性能或鲁棒性。

交叉验证通过拆分训练数据来模拟样本外预测。基于均方误差等模型精度评分，可以对模型参数执行网格搜索。逻辑回归等模型的内置交叉验证功能可通过估计器类实现。例如，`LogisticRegressionCV` 类可与参数配合使用，该参数指示对模型正则化参数 `C` 进行网格搜索的精细程度：

```python
In [108]: from sklearn.linear_model import LogisticRegressionCV

In [109]: model_cv = LogisticRegressionCV(Cs=10)

In [110]: model_cv.fit(X_train, y_train)
Out[110]: LogisticRegressionCV()
```

若要手动进行交叉验证，可使用处理数据拆分过程的辅助函数 `cross_val_score`。例如，要对训练数据进行四次非重叠拆分来交叉验证模型，可以执行：

```python
In [111]: from sklearn.model_selection import cross_val_score

In [112]: model = LogisticRegression(C=10)

In [113]: scores = cross_val_score(model, X_train, y_train, cv=4)

In [114]: scores
Out[114]: array([0.7758, 0.7982, 0.7758, 0.7883])
```

默认评分标准取决于模型，但也可以选择显式评分函数。交叉验证模型需要更长的训练时间，但通常能获得更好的模型性能。

## 12.5 结论

虽然我只是浅尝辄止地介绍了一些 Python 建模库，但现有越来越多针对各类统计和机器学习（或采用 Python 实现，或提供 Python 用户界面）的框架。

本书特别侧重于数据规整（data wrangling）方面，但还有许多其他专门探讨建模和数据科学工具的书籍。其中一些优秀著作包括：

*   Andreas Müller 和 Sarah Guido 所著的《Python 机器学习入门》（O'Reilly 出版）
*   Jake VanderPlas 所著的《Python 数据科学手册》（O'Reilly 出版）
*   Joel Grus 所著的《数据科学入门：使用 Python 实践第一原理》（O'Reilly 出版）
*   Sebastian Raschka 和 Vahid Mirjalili 所著的《Python 机器学习》（Packt Publishing 出版）
*   Aurélien Géron 所著的《Scikit-Learn、Keras 和 TensorFlow 机器学习实战》（O'Reilly 出版）

尽管书籍是宝贵的学习资源，但当底层开源软件发生变化时，它们有时可能会变得过时。熟悉各种统计或机器学习框架的文档，以及时了解最新特性和 API，是一个很好的做法。