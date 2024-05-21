# 前言

大多数数据分析师使用 Python 和 pandas 进行数据处理和操作，这得益于这些库提供的便利性和性能。然而，JavaScript 开发人员一直希望浏览器中也能实现**机器学习**（**ML**）。本书重点介绍了 Danfo.js 如何将数据处理、分析和 ML 工具带给 JavaScript 开发人员，以及如何充分利用这个库来开发数据驱动的应用程序。

本书从 JavaScript 概念和现代 JavaScript 的介绍开始。然后，您将使用 Danfo.js 和 Dnotebook 进行数据分析和转换，这是 JavaScript 的交互式计算环境。之后，本书涵盖了如何加载不同类型的数据集，并通过执行操作（如处理缺失值、合并数据集和字符串操作）来分析它们。您还将专注于数据绘图、可视化、数据聚合和组合操作，通过将 Danfo.js 与 Plotly 结合使用。随后，您将使用 Danfo.js 创建一个无代码数据分析和处理系统。然后，您将介绍基本的 ML 概念，以及如何使用 Tensorflow.js 和 Danfo.js 构建推荐系统。最后，您将使用 Danfo.js 构建由 Twitter 驱动的分析仪表板。

通过本书，您将能够在服务器端 Node.js 或浏览器中构建和嵌入数据分析、可视化和 ML 功能的 JavaScript 应用程序。

# 这本书是为谁准备的

本书适用于数据科学初学者、数据分析师和希望使用各种数据集探索数据分析和科学计算的 JavaScript 开发人员。如果您是数据分析师、数据科学家或 JavaScript 开发人员，并希望在 ML 工作流程中实现 Danfo.js，您也会发现本书很有用。对 JavaScript 编程语言、数据科学和 ML 的基本理解将有助于您理解本书涵盖的关键概念；然而，本书的第一章和附录中提供了 JavaScript 的入门部分。

# 本书涵盖的内容

第一章，现代 JavaScript 概述，讨论了 ECMA 6 语法和`import`语句、类方法、`extend`方法和构造函数的使用。它还深入解释了`Promise`方法的使用，`async`和`await`函数的使用，以及`fetch`方法。它还介绍了如何建立支持现代 JavaScript 语法的环境，以及适当的版本控制，以及如何编写单元测试。

第二章，Dnotebook-用于 JavaScript 的交互式计算环境，深入探讨了 Dnotebook。对于来自 Python 生态系统的读者来说，这类似于 Jupyter Notebook。我们讨论了如何使用 Dnotebook，如何创建和删除单元格，如何在其中编写 Markdown，以及如何保存和共享您的笔记本。

第三章，使用 Danfo.js 入门，介绍了 Danfo.js 以及如何创建数据框架和系列。它还介绍了一些数据分析和处理的基本方法。

第四章，数据分析、整理和转换，探讨了 Danfo.js 在实际数据集中的实际应用。在这里，您将学习如何加载不同类型的数据集，并通过执行操作（如处理缺失值、计算描述性统计、执行数学运算、合并数据集和字符串操作）来分析它们。

第五章，使用 Plotly.js 进行数据可视化，介绍了数据绘图和可视化。在这里，您将学习数据可视化和绘图的基础知识，以及如何使用 Plotly.js 进行基本绘图。

*第六章**，使用 Danfo.js 进行数据可视化*，介绍了使用 Danfo.js 进行数据绘图和可视化。在这里，您将学习如何在 DataFrame 或 series 上直接使用 Danfo.js 创建图表。您还将学习如何自定义 Danfo.js 图表。

*第七章**，数据聚合和分组操作*，介绍了分组操作以及如何使用 Danfo.js 执行这些操作，包括如何按一个或多个列进行分组，如何使用提供的分组-聚合函数，以及如何使用`.apply`创建自定义聚合函数。我们还展示了分组操作的内部工作原理。

*第八章**，创建无代码数据分析/处理系统*，展示了 Danfo.js 可以让我们做什么。在本章中，我们将创建一个无代码数据处理和分析环境，用户可以在其中上传他们的数据，然后进行艺术化的分析和处理。

*第九章**，机器学习基础*，以简单的术语介绍了机器学习。它还向您展示了如何在浏览器中借助一些 ML JavaScript 工具进行机器学习。

*第十章**，TensorFlow.js 简介*，介绍了 TensorFlow.js。它还展示了如何执行基本的数学运算以及如何创建、训练、保存和重新加载 ML 模型。本章还展示了如何有效地集成 Danfo.js 和 Tensorflow.js 来训练模型。

*第十一章**，使用 Danfo.js 和 TensorFlow.js 构建推荐系统*，向您展示了如何使用 TensorFlow.js 和 Danfo.js 构建电影推荐系统。它向您展示了如何在 Node.js 中训练模型以及如何将其与客户端集成。它还展示了 Danfo.js 如何使数据预处理变得简单。

*第十二章**，构建 Twitter 分析仪表盘*，您将在此构建一个使用 Danfo.js 作为前端和后端的 Twitter 分析仪表盘；目标是展示在数据分析应用中使用同一库的简易性，相比于例如在后端使用 Python 和在前端使用 JavaScript。

*第十三章**，附录：JavaScript 基本概念*，介绍了 JavaScript 编程语言。在这里，我们向初学者介绍了变量定义、函数创建以及在 JavaScript 中执行计算的不同方式。

# 充分利用本书

在本书中，您需要对 JavaScript 有基本的了解，并且了解 Next.js、React.js、TensorFlow.js 和 tailwindcss 等框架将是一个优势。

![](img/B17076_Preface_Table_RK.jpg)

**如果您使用的是本书的数字版本，我们建议您自己输入代码或从书的 GitHub 存储库中访问代码（链接在下一节中提供）。这样做将帮助您避免与复制和粘贴代码相关的任何潜在错误。**

# 下载示例代码文件

您可以从 GitHub 上下载本书的示例代码文件，链接为[`github.com/PacktPublishing/Building-Data-Driven-Applications-with-Danfo.js`](https://github.com/PacktPublishing/Building-Data-Driven-Applications-with-Danfo.js)。如果代码有更新，将在 GitHub 存储库中进行更新。

我们还提供了来自我们丰富书籍和视频目录的其他代码包，可在[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)上找到。快去看看吧！

# 下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的屏幕截图和图表的彩色图片。您可以在这里下载：[`static.packt-cdn.com/downloads/9781801070850_ColorImages.pdf`](https://static.packt-cdn.com/downloads/9781801070850_ColorImages.pdf)。

# 使用的约定

本书中使用了许多文本约定。

`文本中的代码`：表示文本中的代码词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 句柄。例如："在`financial_df` DataFrame 的情况下，当我们使用`read_csv`函数下载数据集时，索引是自动生成的。"

代码块设置如下：

```js
const df = new DataFrame({...})
df.plot("my_div_id").<chart type>
```

当我们希望引起您对代码块的特定部分的注意时，相关行或项目以粗体显示：

```js
…        
var config = {
            displayModeBar: true,
            modeBarButtonsToAdd: [
…
```

任何命令行输入或输出都以以下方式编写：

```js
npm install @tensorflow/tfjs
```

**粗体**：表示新术语、重要单词或屏幕上看到的单词。例如，菜单或对话框中的单词以**粗体**显示。例如："在 Microsoft Edge 中，打开浏览器窗口右上角的 Edge 菜单，然后选择**F12 开发人员工具**。"

提示或重要说明

像这样。
