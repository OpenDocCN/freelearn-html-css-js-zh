

# 第十四章：使用 Node.js 进行微服务登录

在使用微服务架构和 Node.js 的时候，启用和检查日志非常重要。

我们将从这个章节开始，了解使用 Node.js 进行微服务日志记录的核心概念。日志记录是微服务架构的一个关键方面，为分布式系统中的行为、性能和问题提供了有价值的见解。在 Node.js 微服务中，采用了各种日志技术和库来捕获相关信息和健壮的日志系统。

在微服务架构中，日志扮演着至关重要的角色。让我总结一下没有适当日志记录会发生什么：

+   **缺乏可见性**：没有日志记录，跟踪和理解单个微服务内部发生的事情变得具有挑战性。您将无法了解它们的行为、事件或事务。

+   **故障排除困难**：当问题出现时，故障排除变得繁琐。没有日志，您将没有关于错误、异常或堆栈跟踪的信息。在特定服务中识别故障点变成了一场猜谜游戏。

+   **缺乏全面视角**：每个微服务可能会生成自己的日志，但没有集中式日志系统，您将无法获得整个系统的全面视角。跨越多个服务的模式或趋势可能仍然隐藏。

记住——适当的日志记录确保更好的可见性、更快的故障排除以及更健壮的微服务架构！

到本章结束时，您将学会如何在 Node.js 微服务中更好地、更快地进行调试。

在本章中，我们将涵盖以下主要主题：

+   选择日志框架和定义日志级别

+   结构化日志、日志传输和存储

+   日志过滤、采样、错误处理和异常日志

+   上下文传播、日志监控和分析

在第一部分，我们将展示如何选择一个日志框架并定义日志级别。

# 选择日志框架并定义日志级别

**日志记录**是微服务的一个关键方面，有助于调试、性能监控和系统分析。通过选择合适的日志库并实施最佳实践，您可以为您的 Node.js 微服务构建一个健壮的日志系统。

## 选择日志库

日志库是一段软件，可以帮助您从 Node.js 应用程序中生成和管理日志数据。日志库可以提供各种功能，例如不同的日志级别、日志格式、日志传输和日志聚合。日志库还可以通过减少 `console.log` 的开销并提供更多信息和控制日志数据来提高应用程序的性能和功能。

选择合适的日志库是第一步。以下是一些流行的 Node.js 日志库：

+   `info`、`debug`、`warn`、`error`）。支持日志格式化和自定义。

+   **Bunyan**：强调结构化日志，这在微服务中特别有用。对具有高吞吐量要求的大型系统来说效率很高。支持日志轮转和多种日志级别。

+   **Pino**：专注于快速和轻量级日志。非常适合高性能应用程序。支持 JSON 日志和可定制的日志级别。

这些是最常用的 Node.js 日志库，它们将帮助开发者和系统工程师在调试时节省时间，并在创建系统时没有头痛。

对于这本书，让我们选择 *Winston* 日志库，这是一个广泛使用且功能丰富的 Node.js 应用程序日志库。Winston 允许你在不同的级别记录消息，并支持各种传输方式（例如，控制台、文件、数据库）。

让我们首先使用这里给出的命令安装 Winston：

```js
npm install winston
```

然后，创建一个文件（例如，`logger.js`）来配置 Winston 使用不同的日志级别：

```js
const winston = require('winston');
// Define log levels
const logLevels = {
  error: 0,
  warn: 1,
  info: 2,
  debug: 3,
};
// Define log level colors (optional)
const logColors = {
  error: 'red',
  warn: 'yellow',
  info: 'green',
  debug: 'blue',
};
// Configure Winston logger
const logger = winston.createLogger({
  levels: logLevels,
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.printf(({ level, message, timestamp }) => {
      return `${timestamp} [${level.toUpperCase()}]: ${message}`;
    })
  ),
  transports: [
    new winston.transports.Console({
      level: 'debug', // Log level for the console transport
      format: winston.format.combine(
        winston.format.colorize({ all: true }),
        winston.format.simple()
      ),
    }),
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
    new winston.transports.File({ filename: 'combined.log' }),
  ],
});
// Apply colors to log levels (optional)
winston.addColors(logColors);
module.exports = logger;
```

现在，你可以在你的应用程序中使用这个日志记录器，如下面的示例所示：

```js
const logger = require('./logger');
logger.error('This is an error message');
logger.warn('This is a warning message');
logger.info('This is an info message');
logger.debug('This is a debug message');
```

在这个例子中，我们定义了四个日志级别：`error`、`warn`、`info` 和 `debug`。级别与日益增加的严重性相关联。配置还包括颜色化，以便在控制台中更好地可见。你可以根据具体需求自定义日志级别、颜色和传输方式。

*图 14.1* 展示了日志库：

![图 14.1：日志库（图片由 Freepik 上的 johnstocker 提供）](img/B14980_14_01.jpg)

图 14.1：日志库（图片由 Freepik 上的 johnstocker 提供）

总结来说，虽然 `console.log` 简单直接，但 Winston 提供了更强大的功能，包括可定制的日志级别、结构化格式化和将日志重定向到各种目的地的能力。

*最佳实践*：不要使用 `console.log`，而应使用合适的日志库（如 Winston 或 Bunyan）。这些库允许你控制日志级别、格式化消息并将日志定向到适当的目的地（文件、数据库等）。在生产环境中留下 `console.log` 可能会无意中暴露敏感信息。想象一下意外记录用户凭据或 API 密钥！记住——生产日志很重要。使它们有意义、安全且高效！

现在，让我们继续讨论日志级别。

## 日志级别

**日志级别**是分类 Node.js 中日志消息严重性和重要性的方式。日志级别可以帮助你更有效地过滤、优先排序和管理你的日志数据。日志级别还可能根据你如何配置你的日志框架而影响应用程序的性能和功能。

这里是关于日志级别的几个主要建议：

+   适当地利用不同的日志级别（`info`、`debug`、`warn`、`error`）。

+   根据部署环境或配置动态调整日志级别。

日志级别对于系统非常重要，需要正确调整以实现更快的调试和性能所需的各项要求。

此外，请记住，日志级别可以根据您的环境或配置动态调整，允许您在不同场景中控制日志的详细程度。

在理解了这些概念之后，我们现在转向结构化日志记录、日志传输和存储。

# 结构化日志记录、日志传输和存储

结构化日志记录、日志传输和存储是相关概念，可以帮助您更有效地管理和分析应用程序日志。在本节中，我们将深入探讨结构化日志记录。

## 结构化日志记录

**结构化日志记录**是一种日志记录方法，其中日志消息格式化为一系列键值对或 JSON 对象。这种格式使日志更易于机器读取，并允许更轻松地进行解析、过滤和分析。当与适当的日志传输和存储机制结合使用时，结构化日志记录成为监控和故障排除在微服务架构中的强大工具。

下面是结构化日志记录的一些好处：

+   **机器可读性**：结构化日志易于机器解析，便于自动化日志分析。

+   **上下文信息**：键值对允许在日志消息中包含上下文信息，有助于故障排除。

+   **一致性**：一致的日志格式便于创建日志分析工具，并确保在不同微服务之间的一致性。

现在，让我们看看使用 Winston（Node.js）实现的结构化日志记录：

```js
const winston = require('winston');
const logger = winston.createLogger({
  level: 'info',
  format: winston.format.simple(),
  transports: [
    new winston.transports.Console(),
    new winston.transports.File({ filename: 'error.log', level: 'error' }),
  ],
});
logger.info('This is an info message');
logger.error('This is an error message');
```

在前面的例子中，日志消息包括结构化数据，作为键值对。

什么是敏感数据？敏感数据是指必须保护免受未经授权访问的私人信息。虽然具体内容可能因您的环境而异，但以下是一些常见的敏感数据类型：

+   **个人身份信息（PII）**：这包括全名、地址、电子邮件地址、驾照号码和电话号码等数据。

+   **财务数据**：信用卡信息和其他财务细节属于这一类别。

+   **医疗数据**：医疗历史、记录和任何与健康相关的信息。

+   **密码**：在日志中存储密码是一个重大的安全风险。

+   **IP 地址**：尽管不一定总是严格敏感，但泄露 IP 地址可能会产生隐私影响。

请记住，数据敏感性取决于您的业务环境。即使是看似无害的细节（如邮政编码），如果其泄露可能会损害您的业务或侵蚀客户信任，也应谨慎处理。

下面是一些避免敏感数据日志记录的最佳实践：

+   **排除敏感数据**：最简单的方法是完全避免记录敏感数据。只记录必要的信息。

+   **使用结构化日志记录**：以结构化的方式（例如，JSON）格式化日志，使其更易于管理和搜索。

+   `INFO`、`DEBUG`、`ERROR`）并选择性地记录。

+   **集中式日志记录**：使用集中式系统安全地收集和存储日志。

+   **屏蔽敏感数据**：如果您必须记录某些数据（例如，用于调试），请屏蔽或删除敏感部分（例如，将信用卡号替换为星号）。

记住——负责任地处理敏感数据对于安全和合规至关重要。

学习了这些概念后，我们可以继续学习日志传输和存储。

## 日志传输和存储

结构化日志记录，当与适当的**日志传输和存储**机制结合使用时，可以增强微服务的可观察性和可管理性。日志传输和存储是将您的日志数据从应用程序移动到日志管理系统的过程。

这里是最常见的机制：

+   **控制台传输**：日志通常最初输出到控制台，用于开发和调试目的。控制台传输设置快速简单，如下所示：

    ```js
    new winston.transports.Console()
    ```

+   **文件传输**：日志可以存储在文件中以供后续分析。文件传输适合本地存储日志，可以设置如下：

    ```js
    new winston.transports.File({ filename: 'error.log', level: 'error' })
    ```

+   **基于云的存储**：对于基于云的存储，请考虑像 Amazon CloudWatch、Google Cloud Logging 或 Azure Monitor 这样的服务。这些服务提供可扩展的、可搜索的、集中的日志存储。以下是设置它们的方法：

    ```js
    const { CloudWatchLogTransport } = require('winston-aws-cloudwatch');
    ```

    ```js
    new CloudWatchLogTransport({
    ```

    ```js
      logGroupName: 'your-log-group-name',
    ```

    ```js
      logStreamName: 'your-log-stream-name',
    ```

    ```js
      level: 'info',
    ```

    ```js
      formatLog: (info) => `${info.timestamp} ${info.message}`,
    ```

    ```js
    })
    ```

+   **ELK Stack（Elasticsearch、Logstash、Kibana）**：ELK Stack 是一个流行的开源日志存储和分析解决方案。它允许您在 Elasticsearch 中索引日志，使用 Logstash 进行处理，并使用 Kibana 进行可视化。

+   **集中式日志记录解决方案**：像 Splunk、Sumo Logic 或 Datadog 这样的服务提供集中式日志记录解决方案。**集中式日志记录解决方案**是收集、存储和分析来自多个来源（如 Node.js 应用程序、服务器、网络或其他服务）的日志数据的系统。集中式日志记录解决方案可以帮助您通过提供对日志数据的统一和全面视图来监控、故障排除和优化您的 Node.js 应用程序。集中式日志记录解决方案还可以通过更快地检测和解决问题、减少日志噪音以及通过高级功能（如搜索、分析和警报）提高日志质量来帮助您提高 Node.js 应用程序的安全性、性能和可靠性。

总结来说，传输和存储的选择取决于诸如可扩展性、分析需求以及您应用程序的整体架构等因素。

现在，我们可以继续到下一节，我们将讨论日志过滤、采样、错误处理和异常日志记录。

# 日志过滤、采样、错误处理和异常日志记录

在微服务架构中，有效的**日志过滤**、**采样**、**错误处理**和**异常日志记录**对于高效管理日志和深入了解系统行为至关重要。

这里是如何处理这些方面的方法：

+   **日志过滤**：日志过滤涉及根据特定标准选择性地捕获和存储日志条目。这对于管理日志量并关注相关信息至关重要。

    下面是使用 Winston（Node.js）的实现：

    ```js
    const winston = require('winston');
    ```

    ```js
    const logger = winston.createLogger({
    ```

    ```js
      format: winston.format.simple(),
    ```

    ```js
      transports: [
    ```

    ```js
        new winston.transports.Console(),
    ```

    ```js
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
    ```

    ```js
        new winston.transports.File({ filename: 'combined.log' }),
    ```

    ```js
      ],
    ```

    ```js
      // Filtering to include only error logs in a specific file
    ```

    ```js
      exceptionHandlers: [
    ```

    ```js
        new winston.transports.File({ filename: 'exceptions.log' }),
    ```

    ```js
      ],
    ```

    ```js
    });
    ```

    ```js
    logger.info('This will be logged');
    ```

    ```js
    logger.error('This will be logged as an error');
    ```

    ```js
    // Manually throw an exception to trigger the exception handler
    ```

    ```js
    try {
    ```

    ```js
      throw new Error('This is a manually triggered exception');
    ```

    ```js
    } catch (error) {
    ```

    ```js
      logger.error('Caught an exception:', error);
    ```

    ```js
    }
    ```

    在本例中，日志根据严重级别进行过滤，异常则单独处理。

+   **日志采样**：日志采样涉及捕获日志的子集，而不是记录每个事件。当处理高流量系统时，这很有用，可以避免日志存储超载。

    下面是使用 Winston 进行日志采样的实现：

    ```js
    const winston = require('winston');
    ```

    ```js
    const logger = winston.createLogger({
    ```

    ```js
      format: winston.format.simple(),
    ```

    ```js
      transports: [
    ```

    ```js
        new winston.transports.Console(),
    ```

    ```js
        new winston.transports.File({ filename: 'sampled.log' }),
    ```

    ```js
      ],
    ```

    ```js
    });
    ```

    ```js
    // Custom sampling function to log only 10% of the messages
    ```

    ```js
    const samplingFunction = (info) => Math.random() < 0.1 ? info : false;
    ```

    ```js
    logger.add(
    ```

    ```js
      new winston.transports.File({
    ```

    ```js
        filename: 'sampled.log',
    ```

    ```js
        format: winston.format.combine(
    ```

    ```js
          winston.format(info => samplingFunction(info))(),
    ```

    ```js
          winston.format.simple()
    ```

    ```js
        ),
    ```

    ```js
      })
    ```

    ```js
    );
    ```

    ```js
    for (let i = 0; i < 100; i++) {
    ```

    ```js
      logger.info(`Log message ${i}`);
    ```

    ```js
    }
    ```

    在本例中，只有大约 10%的日志消息被写入`sampled.log`文件。

+   **错误处理和异常记录**：在微服务架构中，错误处理对于识别和解决问题至关重要。记录带有详细信息的异常有助于调试。

    下面是使用 Winston 进行错误处理和异常记录的实现：

    ```js
    const winston = require('winston');
    ```

    ```js
    const logger = winston.createLogger({
    ```

    ```js
      format: winston.format.simple(),
    ```

    ```js
      transports: [
    ```

    ```js
        new winston.transports.Console(),
    ```

    ```js
        new winston.transports.File({ filename: 'error.log', level: 'error' }),
    ```

    ```js
      ],
    ```

    ```js
      exceptionHandlers: [
    ```

    ```js
        new winston.transports.File({ filename: 'exceptions.log' }),
    ```

    ```js
      ],
    ```

    ```js
    });
    ```

    ```js
    // Example of logging an exception
    ```

    ```js
    try {
    ```

    ```js
      // Some code that might throw an exception
    ```

    ```js
      throw new Error('This is an exception');
    ```

    ```js
    } catch (error) {
    ```

    ```js
      logger.error('Caught an exception:', error);
    ```

    ```js
    }
    ```

    在本例中，异常被捕获，并在`exceptions.log`文件中单独记录详细信息。

总结来说，使用过滤根据特定标准（如严重级别或自定义条件）包括或排除日志。在高流量系统中，实现日志采样以捕获日志子集，以避免日志存储超载。正确处理代码中的错误，并记录有关异常的详细信息以帮助调试和故障排除。为了清晰起见，将异常日志与常规日志分开。

根据您的特定应用程序和环境定制日志过滤和采样对于在捕获有价值信息和高效管理日志量之间取得适当的平衡至关重要。

在下一节中，我们将学习关于上下文传播、日志监控和分析的内容。

# 上下文传播、监控和分析日志

上下文传播、监控和分析日志是管理分布式系统中的微服务的关键方面。在本节中，我们将更深入地探讨上下文传播。

## 上下文传播

在微服务中，由于请求可以跨越多个服务，传播上下文信息对于跟踪和理解请求流至关重要。上下文信息通常以头或令牌的形式存在，允许您在不同微服务之间关联日志。

下面是一个上下文传播的例子。在一个使用 Express.js 的 Node.js 环境中，您可以使用中间件来传播上下文信息：

```js
// Middleware to add context to requests
app.use((req, res, next) => {
  // Add a unique request ID to the request
  req.requestId = generateRequestId();
  // Log the start of the request
  logger.info(`[${new Date()}] Start processing request ${req.requestId}`);
  next();
});
// Middleware for logging requests
app.use((req, res, next) => {
  // Log relevant information with the request context
  logger.info(`[${new Date()}] ${req.method} ${req.url} - Request ID: ${req.requestId}`);
  next();
});
// Other middleware and routes
// Error handling middleware
app.use((err, req, res, next) => {
  // Log errors with the request context
  logger.error(`[${new Date()}] Error processing request ${req.requestId}: ${err.message}`, err);
  res.status(500).send('Something went wrong!');
});
```

在本例中，请求中添加了一个唯一的请求 ID，并用于记录请求的开始以及发生的任何错误。在 Node.js 中的上下文传播是跨异步边界（如回调、承诺或事件发射器）传输上下文信息（如跟踪 ID）的过程。上下文传播使得分布式跟踪成为可能，这允许您监控和分析您的 Node.js 应用程序在多个服务和进程中的性能和行为。

在下一节中，我们将讨论监控。

## 监控

**监控**涉及积极观察微服务的行为和性能，以确保它们满足**服务级别目标**（**SLOs**），并主动识别和解决问题。

以下是一些监控工具：

+   **Prometheus**：一个开源的监控和警报工具包，旨在实现可靠性和可伸缩性。

+   **Grafana**：与 Prometheus 协作良好，用于可视化和分析指标。

+   **Datadog、New Relic 或 AppDynamics**：提供全面监控能力的商业解决方案，包括性能指标、错误率和分布式跟踪。

这些只是可以帮助您监控 Node.js 应用程序的一些工具。您还可以使用其他工具或方法，例如内置的 Node.js 调试器、`console` 模块或 `node:async_hooks` 模块。工具的选择取决于您的具体需求和目标。您还可以结合不同的工具，以获得更全面的应用程序性能和行为视图。

在下一节中，我们将讨论日志分析。

## 日志分析

**日志分析**涉及从日志中提取有价值的信息，以了解微服务的行为、解决问题和识别优化区域。

这里有一些日志分析工具：

+   **ELK Stack**：Elasticsearch 用于索引日志，Logstash 用于日志处理，Kibana 用于可视化。

+   **Splunk**：一个商业日志分析平台，允许您搜索、监控和分析机器生成数据。

+   **Graylog**：一个具有搜索和分析功能的开源日志管理平台。

总结来说，跨微服务传播上下文信息，例如请求 ID，以关联日志和跟踪请求流。使用 Prometheus、Grafana、Datadog 或其他工具积极监控微服务，以确保它们满足性能和可靠性目标。使用 ELK Stack、Splunk 或 Graylog 等日志分析工具从日志中提取有意义的见解，并促进故障排除和优化。

通过有效地实施上下文传播、监控和日志分析，您可以增强微服务的可观察性，使维护、故障排除和优化整个系统变得更加容易。

# 概述

在本章中，我们学习了关于微服务以及如何使用几个原则和工具在 Node.js 中监控微服务的大量知识。

总结起来，使用 Node.js 在微服务中进行日志记录是确保可观察性、故障排除和维护分布式系统健康的关键方面。以下是关键点的总结：

+   **日志库**：在 Node.js 中使用如 Winston 这样的日志库进行结构化和灵活的日志记录。

+   使用 `error`、`warn`、`info`、`debug` 等级别对日志进行分类和优先级排序。

+   **结构化日志**：通过将日志格式化为键值对或 JSON 对象来实现结构化日志，以增强机器可读性。

+   **上下文传播**：在微服务之间传播上下文信息（例如，请求 ID），以关联日志并追踪请求的流程。

+   **错误处理**：实现错误处理和异常日志记录，以捕获有关错误的详细信息，有助于调试。

+   **日志过滤和采样**：根据严重程度等标准应用日志过滤，以选择性地捕获日志。考虑日志采样以捕获日志子集，尤其是在高流量系统中。

+   **日志传输和存储**：根据您的应用程序需求和架构选择合适的日志传输方式（例如，控制台、文件、基于云的存储）。

+   **监控**：使用 Prometheus、Grafana、Datadog 或商业解决方案等工具积极监控微服务，以确保性能和可靠性。

+   **日志分析**：利用日志分析工具，如 ELK Stack、Splunk 或 Graylog，从日志中提取有价值的见解，以用于故障排除和优化。

+   **集中式日志**：考虑集中式日志解决方案以更好地聚合、搜索和分析日志。有效的日志实践有助于微服务的整体可观察性，便于快速识别和解决问题，优化性能，并提高整个系统的可靠性。

在下一章中，我们将学习如何解释微服务中的监控数据。

# 测验时间

+   Node.js 有哪些流行的日志库？

+   结构化日志是什么？

+   日志过滤、采样、错误处理和异常日志记录是什么？
