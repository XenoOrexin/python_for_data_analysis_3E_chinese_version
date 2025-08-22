# 1 预备知识（Preliminaries）

__

《Python for Data Analysis 3rd Edition》（Python数据分析 第3版）的开放获取（Open Access）网络版现已上线，作为[纸质版与电子版](https://amzn.to/3DyLaJc)的配套资源。如果您发现任何勘误，[请在此处提交](https://oreilly.com/catalog/0636920519829/errata)。请注意，由Quarto生成的本站点某些格式会与O’Reilly出版的印刷版及电子书版本有所不同。

若您觉得本书在线版对您有帮助，请考虑[订购纸质版本](https://amzn.to/3DyLaJc)或[无DRM电子书](https://www.ebooks.com/en-us/book/210644288/python-for-data-analysis/wes-mckinney/?affId=WES398681F)以支持作者。本站内容禁止复制或转载。代码示例采用MIT许可证，可在GitHub或Gitee上获取。

## 1.1 本书主要内容是什么？

本书主要介绍使用 Python 进行数据操作、处理、清洗和加工的核心技术。我的目标是提供 Python 编程语言及其面向数据的库生态系统与工具的使用指南，帮助你成为一名高效的数据分析师。虽然书名包含"数据分析"，但本书重点在于 Python 编程、库和工具的使用，而非数据分析方法论本身。这正是数据分析所需的 Python 编程技能。

在我于 2012 年首次出版本书后不久，"数据科学"逐渐成为一个统称，涵盖从简单描述性统计到高级统计分析和机器学习的各种内容。与此同时，用于数据分析（或数据科学）的 Python 开源生态系统也显著扩展。现在已有许多其他书籍专门探讨这些更高级的方法论。我希望本书能为你奠定坚实基础，助你进一步学习更专业的领域知识。

__

注意 

有些人可能认为本书大部分内容属于"数据操作"而非"数据分析"。我们也会使用"数据规整"（wrangling）或"数据整理"（muning）来指代数据操作。

### 涉及哪些数据类型？

当我说"数据"时，具体指的是什么？本书主要关注**结构化数据**（structured data）——这个刻意模糊的术语涵盖了许多常见数据形式，例如：

*   表格或类电子表格数据：每列可能具有不同类型（字符串、数值、日期等）。这包括通常存储在关系数据库或制表符/逗号分隔文本文件中的大多数数据类型
*   多维数组（矩阵）
*   通过键列互相关联的多张数据表（SQL 用户所称的主键或外键）
*   均匀或非均匀间隔的时间序列

这绝非完整列表。尽管有时并不明显，但大部分数据集都可以转换为更适合分析和建模的结构化形式。即使不能直接转换，也可以从数据集中提取特征形成结构化数据。例如：新闻文章集合可以处理成词频表，进而用于情感分析。

对于电子表格软件（如全球使用最广泛的数据分析工具 Microsoft Excel）的大多数用户而言，这些数据类型应该都不陌生。

## 1.2 为何选择 Python 进行数据分析？

对许多人而言，Python 编程语言具有强大的吸引力。自 1991 年首次问世以来，Python 已成为最流行的解释型编程语言之一，与 Perl、Ruby 等语言齐名。大约从 2005 年开始，Python 和 Ruby 因拥有众多 Web 框架（如 Rails \(Ruby\) 和 Django \(Python\)）而特别受欢迎，被广泛用于网站构建。这类语言常被称为_脚本_语言，因为它们可用于快速编写小型程序或_脚本_来自动化其他任务。我不喜欢"脚本语言"这个术语，因为它带有一种暗示，似乎这些语言无法用于构建严肃的软件。由于各种历史和文化原因，在解释型语言中，Python 发展出了一个庞大而活跃的科学计算和数据分析社区。过去 20 年间，Python 已从一个前沿或"风险自负"的科学计算语言，转变为数据科学、机器学习以及学术界和工业界通用软件开发中最重要的语言之一。

在数据分析、交互式计算和数据可视化方面，Python 不可避免地会与其他广泛使用的开源和商业编程语言及工具（如 R、MATLAB、SAS、Stata 等）进行比较。近年来，Python 改进的开源库（如 pandas 和 scikit-learn）使其成为数据分析任务的热门选择。结合 Python 在通用软件工程方面的整体优势，它作为构建数据应用程序的主要语言是一个绝佳的选择。

### Python 作为粘合剂

Python 在科学计算领域的部分成功在于其易于集成 C、C++ 和 FORTRAN 代码。大多数现代计算环境共享一组类似的传统 FORTRAN 和 C 库，用于线性代数、优化、积分、快速傅里叶变换和其他此类算法。许多公司和国家实验室也是如此，他们使用 Python 将积累了数十年的传统软件粘合在一起。

许多程序由小部分代码（大部分时间花费在此）和大量不经常运行的"粘合代码"组成。在许多情况下，粘合代码的执行时间微不足道；最有效的努力是优化计算瓶颈，有时通过将代码迁移到 C 等低级语言来实现。

### 解决"两种语言"问题

在许多组织中，通常使用更专业的计算语言（如 SAS 或 R）来研究、原型设计和测试新想法，然后将这些想法移植到更大的生产系统中，例如用 Java、C\# 或 C++ 编写的系统。人们越来越多地发现，Python 不仅适用于研究和原型设计，还适用于构建生产系统。当一种语言就足够时，为什么要维护两个开发环境？我相信越来越多的公司会走这条路，因为让研究人员和软件工程师使用同一套编程工具通常会带来显著的组织效益。

过去十年中，出现了一些解决"两种语言"问题的新方法，例如 Julia 编程语言。在许多情况下，要充分发挥 Python 的潜力，_需要_使用 C 或 C++ 等低级语言进行编程，并为该代码创建 Python 绑定。也就是说，诸如 Numba 等库提供的"即时"（JIT）编译器技术，提供了一种在不离开 Python 编程环境的情况下，在许多计算算法中实现卓越性能的方法。

### 为何不选择 Python？

尽管 Python 是构建多种分析应用程序和通用系统的优秀环境，但在某些用途上 Python 可能不太适合。

由于 Python 是一种解释型编程语言，通常大多数 Python 代码的运行速度远慢于用 Java 或 C++ 等编译型语言编写的代码。鉴于_程序员的时间_通常比_CPU 时间_更宝贵，许多人乐于进行这种权衡。然而，在具有极低延迟或苛刻资源利用率要求的应用程序（例如高频交易系统）中，花费时间使用 C++ 等较低级（但生产力也较低）的语言进行编程以实现最大可能性能，可能是值得的。

Python 对于构建高度并发的多线程应用程序，尤其是具有许多 CPU 密集型线程的应用程序，可能具有挑战性。其原因在于 Python 具有所谓的_全局解释器锁_（GIL），这是一种防止解释器一次执行多个 Python 指令的机制。GIL 存在的技术原因超出了本书的范围。尽管在许多大数据处理应用中，可能需要计算机集群在合理的时间内处理数据集，但在某些情况下，单进程多线程系统仍然是可取的。

这并不是说 Python 无法执行真正的多线程并行代码。使用原生多线程（用 C 或 C++）的 Python C 扩展可以并行运行代码而不受 GIL 的影响，只要它们不需要频繁与 Python 对象交互。

## 1.3 必备 Python 库

对于不太熟悉 Python 数据生态系统及本书所使用库的读者，我将简要概述其中部分重要库。

### NumPy

[NumPy](https://numpy.org)（Numerical Python 的缩写）长期以来一直是 Python 数值计算的基石。它提供了 Python 中涉及数值数据的大多数科学应用所需的数据结构、算法和库连接机制。NumPy 包含以下核心功能：

*   快速高效的多维数组对象 _ndarray_
*   用于执行数组元素级计算或数组间数学运算的函数
*   用于将基于数组的数据集读写到磁盘的工具
*   线性代数运算、傅里叶变换和随机数生成
*   成熟的 C API，可用于 Python 扩展及原生 C/C++ 代码访问 NumPy 的数据结构和计算设施

除了 NumPy 为 Python 提升的快速数组处理能力外，它在数据分析中的一个主要用途是作为算法与库之间传递数据的容器。对于数值数据，NumPy 数组在存储和操作数据时比其他内置 Python 数据结构更高效。此外，用底层语言（如 C 或 FORTRAN）编写的库可以直接操作 NumPy 数组中存储的数据，而无需将数据复制到其他内存表示形式中。因此，许多 Python 数值计算工具要么将 NumPy 数组作为主要数据结构，要么以实现与 NumPy 的互操作性为目标。

### pandas

[pandas](https://pandas.pydata.org) 提供了高级数据结构和函数，旨在使结构化或表格化数据的处理变得直观灵活。自 2010 年问世以来，它助力 Python 成为强大高效的数据分析环境。本书将主要使用 pandas 中的两种核心对象：DataFrame（一种具有行标签和列标签的表格型、列向数据结构）和 Series（一维带标签数组对象）。

pandas 融合了 NumPy 的数组计算理念与电子表格及关系型数据库（如 SQL）的数据操作能力。它提供了便捷的索引功能，使您能够重塑、切片、分割、执行聚合操作并选择数据子集。由于数据操作、准备和清洗是数据分析中的重要技能，pandas 成为本书的重点内容之一。

背景方面，我于 2008 年初在量化投资管理公司 AQR Capital Management 任职期间开始构建 pandas。当时，我有一系列明确的需求，而现有工具无法很好地满足：

*   支持自动或显式数据对齐的带标签轴数据结构——这可避免因数据未对齐及处理来自不同源的不同索引数据而产生的常见错误
*   集成的时间序列功能
*   能够同时处理时间序列数据和非时间序列数据的统一数据结构
*   保留元数据的算术运算和归约操作
*   灵活的缺失数据处理机制
*   主流数据库（例如基于 SQL 的数据库）中的合并及其他关系操作

我希望能够在一个地方完成所有这些操作，最好使用适合通用软件开发的编程语言。Python 是满足这一需求的理想候选语言，但当时尚不存在提供此功能的集成化数据结构和工具集。由于最初是为解决金融和商业分析问题而构建，pandas 特别具备深度的时间序列功能以及非常适合处理业务流程生成的时间索引数据的工具。

2011 年和 2012 年的大部分时间里，我与前 AQR 同事 Adam Klein 和 Chang She 共同扩展了 pandas 的功能。2013 年，我逐渐淡出日常项目开发，此后 pandas 成为一个完全由社区拥有和维护的项目，全球贡献者数量远超两千人。

对于使用 R 语言进行统计计算的用户，DataFrame 的名称会很熟悉，因为该对象得名于 R 中类似的 `data.frame` 对象。与 Python 不同，数据框是 R 编程语言及其标准库的内置功能。因此，pandas 中的许多功能通常是 R 核心实现的一部分或由附加包提供。

pandas 名称本身源于计量经济学中的多维结构化数据集术语 _面板数据_（panel data），同时也是对短语 _Python 数据分析_（Python data analysis）的戏仿。

### matplotlib

[matplotlib](https://matplotlib.org) 是最流行的用于生成绘图及其他二维数据可视化的 Python 库。它最初由 John D. Hunter 创建，现由一个大型开发团队维护。该库专为创建适合出版物质量的图表而设计。虽然 Python 程序员可使用其他可视化库，但 matplotlib 仍被广泛使用，并能与生态系统中的其他部分良好集成。我认为将其作为默认可视化工具是稳妥的选择。

### IPython 与 Jupyter

[IPython 项目](https://ipython.org)始于 2001 年，是 Fernando Pérez 的副项目，旨在打造更好的交互式 Python 解释器。在随后的 20 年中，它已成为现代 Python 数据栈中最重要的工具之一。尽管 IPython 本身不提供任何计算或数据分析工具，但它专为交互式计算和软件开发工作而设计。它鼓励_执行-探索_（execute-explore）工作流，而非许多其他编程语言典型的_编辑-编译-运行_（edit-compile-run）工作流。它还提供对操作系统 shell 和文件系统的集成访问，这在许多情况下减少了在终端窗口和 Python 会话之间切换的需求。由于数据分析编码涉及大量探索、试错和迭代，IPython 能帮助您更高效地完成任务。

2014 年，Fernando 和 IPython 团队宣布了 [Jupyter 项目](https://jupyter.org)，这是一项更广泛的倡议，旨在设计语言无关的交互式计算工具。IPython web notebook 转变为 Jupyter notebook，目前支持超过 40 种编程语言。IPython 系统现在可作为_内核_（kernel，即编程语言模式）用于在 Jupyter 中使用 Python。

IPython 本身已成为更广泛的 Jupyter 开源项目的组成部分，后者为交互式和探索性计算提供了高效环境。其最古老且最简单的"模式"是作为增强的 Python shell，旨在加速 Python 代码的编写、测试和调试。您也可以通过 Jupyter notebook 使用 IPython 系统。

Jupyter notebook 系统还允许您使用 Markdown 和 HTML 编写内容，从而创建包含代码和文本的丰富文档。

我个人在 Python 工作中经常使用 IPython 和 Jupyter，无论是运行、调试还是测试代码。

在 [GitHub 上的随书材料](https://github.com/wesm/pydata-book)中，您将找到包含各章所有代码示例的 Jupyter notebook。如果您所在地区无法访问 GitHub，可以[尝试 Gitee 上的镜像](https://gitee.com/wesmckinn/pydata-book)。

### SciPy

[SciPy](https://scipy.org) 是一个软件包集合，解决了科学计算中的许多基础问题。其各模块包含以下工具：

`scipy.integrate`
    
数值积分例程和微分方程求解器

`scipy.linalg`
    
扩展 `numpy.linalg` 功能的线性代数例程和矩阵分解

`scipy.optimize`
    
函数优化器（最小化器）和求根算法

`scipy.signal`
    
信号处理工具

`scipy.sparse`
    
稀疏矩阵和稀疏线性系统求解器

`scipy.special`
    
SPECFUN 的封装器（SPECFUN 是 FORTRAN 库，实现了许多常见数学函数，如 `gamma` 函数）

`scipy.stats`
    
标准连续和离散概率分布（密度函数、采样器、连续分布函数）、各种统计检验及更多描述性统计量

NumPy 和 SciPy 共同构成了许多传统科学计算应用中相对完整成熟的计算基础。

### scikit-learn

自 2007 年项目启动以来，[scikit-learn](https://scikit-learn.org) 已成为 Python 程序员首选的通用机器学习工具包。截至本书撰写时，已有超过两千名不同贡献者向项目提交代码。其包含的子模块涵盖以下模型：

*   分类：支持向量机（SVM）、最近邻、随机森林、逻辑回归等
*   回归：Lasso、岭回归等
*   聚类：_k_ 均值（k-means）、谱聚类等
*   降维：主成分分析（PCA）、特征选择、矩阵分解等
*   模型选择：网格搜索、交叉验证、评估指标
*   预处理：特征提取、归一化

与 pandas、statsmodels 和 IPython 一样，scikit-learn 对于使 Python 成为高效的数据科学编程语言至关重要。虽然本书无法包含 scikit-learn 的全面指南，但我会简要介绍其部分模型及如何与书中其他工具配合使用。

### statsmodels

[statsmodels](https://statsmodels.org) 是一个统计分析包，其雏形源自斯坦福大学统计学教授 Jonathan Taylor 的工作，他实现了 R 编程语言中流行的多种回归分析模型。Skipper Seabold 和 Josef Perktold 于 2010 年正式创建了新的 statsmodels 项目，此后项目发展吸引了大量活跃用户和贡献者。Nathaniel Smith 开发了 Patsy 项目，为 statsmodels 提供了受 R 公式系统启发的公式或模型规范框架。

与 scikit-learn 相比，statsmodels 包含经典（主要是频率学派）统计和计量经济学算法。其子模块包括：

*   回归模型：线性回归、广义线性模型、稳健线性模型、线性混合效应模型等
*   方差分析（ANOVA）
*   时间序列分析：自回归（AR）、自回归移动平均（ARMA）、差分整合移动平均自回归（ARIMA）、向量自回归（VAR）等模型
*   非参数方法：核密度估计、核回归
*   统计模型结果可视化

statsmodels 更专注于统计推断，提供参数的不确定性估计和 _p_ 值。相比之下，scikit-learn 更侧重于预测。

与 scikit-learn 类似，我将简要介绍 statsmodels 及其与 NumPy 和 pandas 的配合使用方法。

### 其他包

2022 年，还有许多其他 Python 库可能在一本关于数据科学的书中讨论。这包括一些较新的项目，如 TensorFlow 或 PyTorch，这些项目在机器学习或人工智能工作中日益流行。鉴于已有其他书籍更专门地关注这些项目，我建议使用本书打好通用 Python 数据处理的基礎。此后，您将具备足够基础去学习更高级的资源，这些资源可能要求一定水平的专业知识。

## 1.4 安装与设置

由于每个人使用 Python 的应用场景不同，因此设置 Python 环境并获取必要的附加包并没有统一的解决方案。许多读者可能没有完全适合跟随本书学习的 Python 开发环境，因此这里我将针对各操作系统提供详细的设置说明。我将使用 Miniconda（conda 包管理器的极简安装版本）以及 [conda-forge](https://conda-forge.org)（一个基于 conda 的社区维护软件分发平台）。本书全程使用 Python 3.10，但如果您在未来阅读本书，也可以安装更新的 Python 版本。

如果由于某些原因，在您阅读时这些说明已过时，可以查看[本书的配套网站](https://wesmckinney.com/book)，我会尽力保持其中的安装说明最新。

### Windows 上的 Miniconda

在 Windows 上开始使用，请从 [_https://conda.io_](https://conda.io) 下载最新 Python 版本（当前为 3.9）的 Miniconda 安装程序。我建议遵循 conda 官网上提供的 Windows 安装说明，这些说明可能从本书出版到您阅读期间已发生变化。大多数用户需要 64 位版本，但如果无法在您的 Windows 机器上运行，可以安装 32 位版本。

当提示是为仅当前用户还是系统所有用户安装时，请选择最适合您的选项。仅为自己安装足以满足跟随本书学习的需求。安装程序还会询问是否要将 Miniconda 添加到系统的 PATH 环境变量中。如果选择添加（我通常这样做），那么这个 Miniconda 安装可能会覆盖您已安装的其他 Python 版本。如果不添加，则需要使用安装时创建的 Windows 开始菜单快捷方式来使用此 Miniconda。该开始菜单项可能名为“Anaconda3 (64-bit)”。

我将假设您没有将 Miniconda 添加到系统 PATH 中。要验证配置是否正确，请打开开始菜单中“Anaconda3 (64-bit)”下的“Anaconda Prompt (Miniconda3)”项。然后尝试通过输入 `python` 启动 Python 解释器。您应该看到类似这样的消息：
    
    
    (base) C:\Users\Wes>python
    Python 3.9 [MSC v.1916 64 bit (AMD64)] :: Anaconda, Inc. on win32
    Type "help", "copyright", "credits" or "license" for more information.
    >>>

要退出 Python shell，请输入 `exit()` 并按 Enter。

### GNU/Linux

Linux 的细节会因发行版的不同而略有差异，但这里我将针对 Debian、Ubuntu、CentOS 和 Fedora 等发行版提供详细说明。设置过程与 macOS 类似，除了 Miniconda 的安装方式不同。大多数读者需要下载默认的 64 位安装程序文件，该文件适用于 x86 架构（但未来可能有更多用户使用基于 aarch64 的 Linux 机器）。安装程序是一个必须在终端中执行的 shell 脚本。您将获得一个类似 _Miniconda3-latest-Linux-x86\_64.sh_ 的文件。要安装它，请使用 `bash` 执行此脚本：
    
    
    $ bash Miniconda3-latest-Linux-x86_64.sh

__

注意 

一些 Linux 发行版的包管理器（如 apt）提供了所有必需的 Python 包（尽管某些情况下版本较旧）。这里描述的设置使用 Miniconda，因为它既易于在不同发行版间复现，也便于将包升级到最新版本。

您可以选择 Miniconda 文件的安装位置。我建议将其安装在主目录的默认位置；例如，_/home/$USER/miniconda_（使用您的用户名，自然如此）。

安装程序会询问是否希望修改您的 shell 脚本以自动激活 Miniconda。出于方便考虑，我建议这样做（选择“是”）。

完成安装后，启动一个新的终端进程，并验证您正在使用新的 Miniconda 安装：
    
    
    (base) $ python
    Python 3.9 | (main) [GCC 10.3.0] on linux
    Type "help", "copyright", "credits" or "license" for more information.
    >>>

要退出 Python shell，请输入 `exit()` 并按 Enter，或按 Ctrl-D。

### macOS 上的 Miniconda

下载 macOS Miniconda 安装程序，对于 2020 年及之后发布的基于 Apple Silicon 的 macOS 计算机，文件名应类似 _Miniconda3-latest-MacOSX-arm64.sh_；对于 2020 年之前发布的基于 Intel 的 Mac，文件名应类似 _Miniconda3-latest-MacOSX-x86\_64.sh_。打开 macOS 中的终端应用程序，并通过使用 `bash` 执行安装程序（很可能在您的 `Downloads` 目录中）进行安装：
    
    
    $ bash $HOME/Downloads/Miniconda3-latest-MacOSX-arm64.sh

安装程序运行时，默认会自动在您的默认 shell 环境中的默认 shell 配置文件中配置 Miniconda。这很可能位于 _/Users/$USER/.zshrc_。我建议允许它这样做；如果您不希望安装程序修改默认 shell 环境，则需要查阅 Miniconda 文档以继续操作。

要验证一切是否正常工作，请尝试在系统 shell 中启动 Python（打开终端应用程序以获得命令提示符）：
    
    
    $ python
    Python 3.9 (main) [Clang 12.0.1 ] on darwin
    Type "help", "copyright", "credits" or "license" for more information.
    >>>

要退出 shell，请按 Ctrl-D 或输入 `exit()` 并按 Enter。

### 安装必要的包

现在我们已经在本系统上设置了 Miniconda，是时候安装本书将使用的主要包了。第一步是通过在 shell 中运行以下命令，将 conda-forge 配置为默认包频道：
    
    
    (base) $ conda config --add channels conda-forge
    (base) $ conda config --set channel_priority strict

现在，我们将使用 `conda create` 命令创建一个新的 conda“环境”，并使用 Python 3.10：
    
    
    (base) $ conda create -y -n pydata-book python=3.10

安装完成后，使用 `conda activate` 激活环境：
    
    
    (base) $ conda activate pydata-book
    (pydata-book) $

__

注意 

每次打开新终端时，都需要使用 `conda activate` 激活您的环境。您可以在终端中随时运行 `conda info` 来查看有关活动 conda 环境的信息。

现在，我们将使用 `conda install` 安装全书使用的基本包（及其依赖项）：
    
    
    (pydata-book) $ conda install -y pandas jupyter matplotlib

我们还会使用一些其他包，但这些可以在需要时再安装。有两种安装包的方式：使用 `conda install` 和使用 `pip install`。在使用 Miniconda 时，应始终优先使用 `conda install`，但有些包无法通过 conda 获取，因此如果 `conda install $package_name` 失败，请尝试 `pip install $package_name`。

__

注意 

如果您想立即安装本书其余部分使用的所有包，可以通过运行以下命令完成：
    
    
    conda install lxml beautifulsoup4 html5lib openpyxl \
                   requests sqlalchemy seaborn scipy statsmodels \
                   patsy scikit-learn pyarrow pytables numba

在 Windows 上，请将 Linux 和 macOS 使用的行继续符 `\` 替换为脱字符 `^`。

您可以使用 `conda` `update` 命令更新包：
    
    
    conda update package_name

pip 也支持使用 `--upgrade` 标志进行升级：
    
    
    pip install --upgrade package_name

在本书中，您将有机会多次尝试这些命令。

__

注意 

虽然您可以同时使用 conda 和 pip 安装包，但应避免使用 pip 更新最初通过 conda 安装的包（反之亦然），因为这样做可能导致环境问题。我建议尽可能坚持使用 conda，仅在 `conda install` 无法获取的包时再使用 pip。

### 集成开发环境和文本编辑器

当被问及我的标准开发环境时，我几乎总是说“IPython 加一个文本编辑器”。我通常编写一个程序，并在 IPython 或 Jupyter notebook 中迭代式测试和调试每个部分。能够交互式地处理数据并直观验证特定数据集操作是否正确执行也非常有用。像 pandas 和 NumPy 这样的库设计得在 shell 中使用时非常高效。

然而，在构建软件时，一些用户可能更喜欢使用功能更丰富的集成开发环境（IDE），而不是像 Emacs 或 Vim 这样开箱即用提供更简约环境的编辑器。以下是一些您可以探索的选项：

  * PyDev（免费），一个基于 Eclipse 平台构建的 IDE
  * JetBrains 的 PyCharm（商业用户需订阅，开源开发者免费）
  * Visual Studio 的 Python 工具（适用于 Windows 用户）
  * Spyder（免费），一个目前随 Anaconda 发行的 IDE
  * Komodo IDE（商业版）

由于 Python 的流行，大多数文本编辑器（如 VS Code 和 Sublime Text 2）都提供了出色的 Python 支持。

## 1.5 社区与会议

除了互联网搜索外，各种科学计算和数据相关的 Python 邮件列表通常很有帮助且能及时回应问题。值得关注的包括：

  * pydata：一个谷歌群组列表，用于讨论 Python 数据分析和 pandas 相关问题

  * pystatsmodels：用于 statsmodels 或 pandas 相关问题

  * scikit-learn 邮件列表（_scikit-learn@python.org_）及一般 Python 机器学习话题

  * numpy-discussion：NumPy 相关讨论

  * scipy-user：通用 SciPy 或科学计算 Python 问题

考虑到网址可能变更，此处特未提供具体链接。通过互联网搜索可轻松找到这些资源。

全球每年都会举办众多面向 Python 开发者的会议。若希望与志趣相投的 Python 开发者交流，建议尽可能尝试参与。许多会议为无法承担参会费用或差旅费的人员提供资金支持。以下是一些值得关注的会议：

  * PyCon 与 EuroPython：分别是北美和欧洲两大综合性 Python 会议

  * SciPy 与 EuroSciPy：分别聚焦北美和欧洲的科学计算领域

  * PyData：面向数据科学与数据分析应用场景的全球区域性系列会议

  * 国际与地区性 PyCon 会议（完整列表请参见 <https://pycon.org>）

## 1.6 本书使用指南

若您从未使用 Python 进行编程，建议先花时间阅读 [第 2 章：Python 语言基础、IPython 和 Jupyter Notebook](https://wesmckinney.com/book/python-basics) 和 [第 3 章：内置数据结构、函数与文件](https://wesmckinney.com/book/python-builtin)。这两章浓缩介绍了 Python 语言特性、IPython shell 以及 Jupyter 笔记本，是阅读后续章节的必备基础。若您已有 Python 经验，则可选择略读或跳过这些章节。

接下来，我将简要介绍 NumPy 的核心功能，更高级的 NumPy 用法将留到[附录 A：NumPy 高级应用](https://wesmckinney.com/book/advanced-numpy)中讲解。随后我会介绍 pandas 库，并在剩余章节中结合 pandas、NumPy 和 matplotlib（可视化库）专注于数据分析主题。本书内容采用渐进式结构，但章节间偶尔会出现少量交叉，极少数情况下会提前使用尚未介绍的概念。

虽然读者的工作目标可能各不相同，但所需完成的任务通常可分为以下几大类：

与外部世界交互
　　
读写各种文件格式和数据存储系统

数据准备 (Preparation)
　　
清理、加工 (munging)、合并、标准化、重塑、切片切块、转换数据以进行分析

数据转换 (Transformation)
　　
对数据集分组应用数学和统计运算以生成新数据集（例如，按分组变量聚合大型表格）

建模与计算 (Modeling and computation)
　　
将数据与统计模型、机器学习算法或其他计算工具相结合

结果呈现 (Presentation)
　　
创建交互式或静态图形可视化效果或文本摘要

### 代码示例

本书大部分代码示例均以 IPython shell 或 Jupyter 笔记本中的输入输出形式呈现：
```python
In [5]: 代码示例
Out[5]: 输出结果
```
当看到此类代码示例时，建议您在编程环境的 `In` 代码块中输入示例代码，并按 Enter 键（Jupyter 中按 Shift-Enter）执行。您应能看到与 `Out` 代码块中相似的输出结果。

为提升全书可读性和简洁性，我已调整 NumPy 和 pandas 的默认控制台输出设置。例如，数值数据可能会显示更高精度的数字。若需完全复现书中输出，可在运行代码示例前执行以下 Python 代码：
```python
import numpy as np
import pandas as pd
pd.options.display.max_columns = 20
pd.options.display.max_rows = 20
pd.options.display.max_colwidth = 80
np.set_printoptions(precision=4, suppress=True)
```

### 示例数据

各章节示例数据集托管于 [GitHub 仓库](https://github.com/wesm/pydata-book)（若无法访问 GitHub 可使用[ Gitee 镜像](https://gitee.com/wesmckinn/pydata-book)）。您可通过命令行使用 Git 版本控制系统下载，或从网站下载仓库的压缩包。若遇到问题，请访问[本书网站](https://wesmckinney.com/book)获取最新资料获取说明。

若下载包含示例数据的压缩包，需先将压缩包内容完整解压至目录，然后在终端中进入该目录再运行书中的代码示例：
```bash
$ pwd
/home/wesm/book-materials

$ ls
appa.ipynb  ch05.ipynb  ch09.ipynb  ch13.ipynb  README.md
ch02.ipynb  ch06.ipynb  ch10.ipynb  COPYING     requirements.txt
ch03.ipynb  ch07.ipynb  ch11.ipynb  datasets
ch04.ipynb  ch08.ipynb  ch12.ipynb  examples
```
我已尽力确保 GitHub 仓库包含复现示例所需的所有内容，但仍可能存在疏漏。如有发现，请发送邮件至 _book@wesmckinney.com_。最佳纠错方式是通过[ O'Reilly 网站的错误报告页面](https://oreil.ly/kmhmQ)反馈。

### 导入约定

Python 社区对常用模块采用了以下命名约定：
```python
import numpy as np
import matplotlib.pyplot as plt
import pandas as pd
import seaborn as sns
import statsmodels as sm
```
这意味着当您看到 `np.arange` 时，它引用的是 NumPy 中的 `arange` 函数。这是因为在 Python 软件开发中，从 NumPy 等大型包中直接导入全部内容（`from numpy import *`）被认为是不良实践。