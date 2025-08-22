# 序言

__

《Python for Data Analysis 第三版》的开放获取网络版现已作为[印刷版和数字版](https://amzn.to/3DyLaJc)的配套资源上线。如果您发现任何勘误，[请在此处报告](https://oreilly.com/catalog/0636920519829/errata)。请注意，由Quarto生成的本站某些格式会与O'Reilly出版的印刷版和电子书版本有所不同。

如果您觉得本书在线版对您有帮助，请考虑[订购纸质版本](https://amzn.to/3DyLaJc)或[无DRM限制的电子书](https://www.ebooks.com/en-us/book/210644288/python-for-data-analysis/wes-mckinney/?affId=WES398681F)以支持作者。本站内容不可复制或转载。代码示例采用MIT许可证，可在GitHub或Gitee上获取。

本书第一版于2012年出版，当时正值Python开源数据分析库（特别是pandas）处于早期快速发展阶段。2016至2017年编写第二版时，我不仅需要将内容更新至Python 3.6（第一版基于Python 2.7），还需要适配pandas在过去五年中的重大变化。如今到2022年，Python语言的变化相对较小（当前版本为Python 3.10，3.11版将于2022年底发布），但pandas仍在持续演进。

在第三版中，我的目标是使内容与当前版本的Python、NumPy、pandas及其他项目保持同步，同时对近些年新出现的Python项目持相对保守的讨论态度。鉴于本书已成为许多大学课程和在职专业人士的重要参考资料，我将尽量避免涉及可能在一两年内过时的主题，以确保2023年及之后出版的纸质版本仍能顺畅使用。

第三版的新特点是开放获取的在线版本，托管于我的个人网站<https://wesmckinney.com/book>，作为印刷版和数字版用户的辅助资源。我将努力保持该网站内容的及时更新，因此如果您在使用纸质书时遇到问题，建议查阅网站获取最新内容变更。

## 本书使用的排版约定

本书采用以下排版约定：

_斜体_
    

表示新术语、URL、电子邮件地址、文件名及文件扩展名。

`等宽字体`
    

用于程序清单，以及在段落中引用程序元素，例如变量或函数名称、数据库、数据类型、环境变量、语句和关键字。

`等宽粗体`
    

表示需要用户逐字输入的命令或其他文本。

<等宽斜体>
    

表示需要用用户提供的值或由上下文确定的值替换的文本。

__

提示 

该元素表示提示或建议。

__

注意 

该元素表示一般性说明。

__

警告 

该元素表示警告或需特别注意的事项。

## 使用代码示例 (Using Code Examples)

你可以在本书的 GitHub 代码仓库 <https://github.com/wesm/pydata-book> 中找到各章节的数据文件及相关资料。该仓库已镜像至 Gitee（供无法访问 GitHub 的用户使用），地址为 <https://gitee.com/wesmckinn/pydata-book>。

本书旨在帮助你高效完成工作。一般而言，书中提供的示例代码可直接用于你的程序和文档中，无需联系我们获取授权，除非需要大量复制代码内容。例如：编写程序时使用本书中的多处代码片段无需授权；但销售或分发 O'Reilly 书籍中的示例代码则需获得许可；引用本书内容并附示例代码回答问题无需授权；而将大量示例代码整合到产品文档中则需获得许可。

我们鼓励但不强制要求署名。署名信息通常包含书名、作者、出版社和 ISBN。例如："《利用 Python 进行数据分析》(Python for Data Analysis) ，Wes McKinney 著，O'Reilly 出版。Copyright 2022 Wes McKinney，ISBN 978-1-098-10403-0"。

如果你认为代码使用方式超出合理使用范围或上述授权许可，欢迎通过 [permissions@oreilly.com](mailto:permissions@oreilly.com) 与我们联系。

## O'Reilly 在线学习平台

__

注意

40 多年来，[O'Reilly Media](http://oreilly.com) 始终致力于通过技术与商业培训、知识共享及深度洞察帮助企业获得成功。

我们由专家和革新者组成的独特网络通过书籍、文章及在线学习平台分享知识与专业见解。O'Reilly 在线学习平台为您提供实时培训课程、深度学习路径、交互式编程环境的按需访问权限，以及来自 O'Reilly 和 200 多家其他出版商的海量文本与视频资源。了解更多信息，请访问 [_http://oreilly.com_](http://oreilly.com)。

## 如何联系我们

关于本书的评论和问题，请致函以下出版商：

O’Reilly Media, Inc.

1005 Gravenstein Highway North

Sebastopol, CA 95472

800-998-9938（美国或加拿大境内）

707-829-0515（国际或本地）

707-829-0104（传真）

本书设有相关网页，其中列出了勘误、示例及其他附加信息。您可通过以下网址访问该页面：<https://oreil.ly/python-data-analysis-3e>。

如需对本书提出评论或技术性问题，请发送邮件至 [bookquestions@oreilly.com](mailto:bookquestions@oreilly.com)。

了解我们的书籍和课程的最新动态，请访问 <http://oreilly.com>。

在 LinkedIn 上关注我们：<https://linkedin.com/company/oreilly-media>。

在 Twitter 上关注我们：<http://twitter.com/oreillymedia>。

在 YouTube 上观看我们的内容：<http://youtube.com/oreillymedia>。

## 致谢（Acknowledgments）

这项工作是世界各地许多人多年来的富有成果的讨论、合作与帮助的结晶。我想对他们中的一些人表示感谢。

## 纪念 John D. Hunter（1968–2012）

我们亲爱的朋友与同事 John D. Hunter 在于结肠癌抗争后，于 2012 年 8 月 28 日不幸离世。彼时我刚刚完成本书第一版的最终手稿。

John 对 Python 科学计算及数据社区的贡献与影响难以言表。除在 2000 年代初（当时 Python 还远未如此流行）开发了 matplotlib 之外，他还帮助塑造了关键一代开源开发者的文化，这些人已成为 Python 生态系统中不可或缺的支柱——而我们如今却常视之为理所当然。

我很幸运在 2010 年 1 月，即刚发布 pandas 0.1 版本后不久，就在我的开源生涯早期与 John 建立了联系。他的激励与指导帮助我在最艰难的时期也能持续推进关于 pandas 及 Python 成为一流数据分析语言的愿景。

John 与 Fernando Pérez 和 Brian Granger（IPython、Jupyter 及 Python 社区中许多其他项目的先驱）关系密切。我们四人曾希望共同撰写一本书，但最终我成了最有空闲时间的那一位。我确信他会为过去九年来我们作为个人和作为一个社区所取得的成就感到自豪。

## 第三版致谢（2022）

自我开始撰写本书第一版已逾十载，而我最初踏上Python程序员之旅更是超过十五年。其间万象更新！Python已从相对小众的数据分析语言，演进为最流行且应用最广泛的语言，驱动着数据科学、机器学习与人工智能领域的大部分（即便不是绝大多数！）工作。

自2013年起，我虽未再积极参与pandas开源项目，但其全球开发者社区持续蓬勃发展，成为以社区为中心的开源软件开发的典范。许多处理表格数据的"下一代"Python项目都直接以pandas的用户界面为设计蓝本，这证明该项目对Python数据科学生态系统的未来发展方向具有持久影响力。

我期望本书能继续为渴望学习Python数据处理的学生和个人提供宝贵资源。

特别感谢O'Reilly允许我在个人网站（https://wesmckinney.com/book）发布本书的"开放获取（open access）"版本，期待此举能惠及更多读者，助力拓展数据分析领域的发展机遇。J.J. Allaire在此过程中功不可没，他协助我将本书从Docbook XML格式迁移至[Quarto](https://quarto.org)——一个卓越的适用于印刷与网页的科学技术出版新系统。

衷心感谢我的技术审校者Paul Barry、Jean-Christophe Leyder、Abdullah Karasan和William Jamir，他们细致的反馈极大提升了内容的可读性、清晰度与易理解性。

## 第二版致谢（2017）

自我于2012年7月完成本书第一版手稿以来，已过去近整整五年。世事变迁：Python社区规模急剧扩大，围绕它的开源软件生态也日益繁荣。

若非pandas核心开发者们的不懈努力，本书新版将无缘面世。他们推动项目及其用户社区成长为Python数据科学生态系统的基石之一。这些贡献者包括但不限于：Tom Augspurger、Joris van den Bossche、Chris Bartak、Phillip Cloud、gfyoung、Andy Hayden、Masaaki Horikoshi、Stephan Hoyer、Adam Klein、Wouter Overmeire、Jeff Reback、Chang She、Skipper Seabold、Jeff Tratner以及y-p。

关于第二版的实际撰写工作，我要感谢O'Reilly团队在写作过程中给予的耐心帮助，包括Marie Beaugureau、Ben Lorica和Colleen Toporek。本次同样有幸获得顶尖技术审校人员的支持，感谢Tom Augspurger、Paul Barry、Hugh Brown、Jonathan Coe和Andreas Müller作出的贡献。

本书第一版已被翻译成多种外语版本，包括中文、法文、德文、日文、韩文和俄文。翻译全部内容并将其推广给更广泛的读者群体是一项艰巨且常被忽视的工作。感谢你们帮助世界上更多人学习编程与数据分析工具的使用。

此外，我很幸运在过去几年中持续获得Cloudera和Two Sigma Investments对开源开发工作的支持。相对于用户基数规模，开源软件项目获得的资源支持日益稀缺，企业为关键开源项目提供开发支持显得愈发重要——这本身就是值得坚持的正确之事。

## 初版致谢（2012年）

若没有众多人士的支持，本书的完成将是难以实现的。

在O’Reilly团队中，我特别感谢编辑Meghan Blanchette和Julie Steele，她们在整个过程中给予我指导。Mike Loukides也在提案阶段与我合作，助力本书的诞生。

众多技术审校者提供了宝贵的意见。尤其要感谢Martin Blais和Hugh Brown，他们对全书示例、表述清晰度及结构体系提出了极富建设性的改进建议。James Long、Drew Conway、Fernando Pérez、Brian Granger、Thomas Kluyver、Adam Klein、Josh Klein、Chang She和Stéfan van der Walt分别审阅了一个或多个章节，从多角度提供了精准反馈。

数据科学圈的朋友和同事们为示例和数据集提供了诸多创意，包括：Mike Dewar、Jeff Hammerbacher、James Johndrow、Kristian Lum、Adam Klein、Hilary Mason、Chang She和Ashley Williams。

当然，我还要感谢开源科学Python社区的众多领袖，他们为我的开发工作奠定了基石，并在写作过程中给予鼓励：IPython核心团队（Fernando Pérez、Brian Granger、Min Ragan-Kelly、Thomas Kluyver等）、John Hunter、Skipper Seabold、Travis Oliphant、Peter Wang、Eric Jones、Robert Kern、Josef Perktold、Francesc Alted、Chris Fonnesbeck，以及众多未能一一提及的贡献者。此外还有多人给予了重要支持、创意和鼓励：Drew Conway、Sean Taylor、Giuseppe Paleologo、Jared Lander、David Epstein、John Krowas、Joshua Bloom、Den Pilsworth、John Myles-White等。

我还要感谢成长期遇到的诸多良师益友。首先是AQR的前同事们，这些年来一直为我在pandas领域的工作加油助威：Alex Reyfman、Michael Wong、Tim Sargen、Oktay Kurbanov、Matthew Tschantz、Roni Israelov、Michael Katz、Ari Levine、Chris Uga、Prasad Ramanan、Ted Square和Hoon Kim。最后要感谢我的学术导师Haynes Miller（MIT）和Mike West（Duke）。

2014年，Phillip Cloud和Joris van den Bossche为更新本书代码示例及修正因pandas变更导致的谬误提供了重要帮助。

在个人层面，Casey在写作过程中提供了至关重要的日常支持，容忍我在原本已超负荷的日程基础上赶稿时的情绪起伏。最后，我的父母Bill和Kim教导我永远追随梦想，永不停歇追求的脚步。