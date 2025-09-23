# 附录 A. Apex 和 Visualforce 开发的安全提示

安全性是网络应用程序的重要组成部分。这一重要部分同样适用于 Force.com 应用程序。我们使用 Visualforce 标记和 Apex 创建自定义页面，这使得我们能够向客户提供完全定制的功能。当我们使用这些编程语言时，我们必须注意安全方面。

Force.com 平台提供一些内置的安全辅助功能，例如用户管理、配置文件管理、角色层次结构、**组织默认值**（**OWD**）、权限集、公共组、共享设置、字段可访问性、密码策略、会话设置、网络访问、登录访问策略、证书和密钥管理、单点登录设置、身份提供者、身份提供者、查看设置审计跟踪、过期所有密码、委托管理、远程站点设置和 HTML 文档和附件设置。但是，当我们使用 Apex 和 Visualforce 创建自定义页面时，我们必须小心，因为有许多方法可以绕过 Force.com 平台上的内置安全防御。可能存在一般安全漏洞以及 Apex 和 Visualforce 特定的漏洞。

# 安全扫描工具

在我们将应用程序添加到 AppExchange 之前，我们必须获得安全方面的认证。开发者必须意识到这些安全关注点。有一些工具可用于扫描我们的代码以检查安全和质量，例如，Force.com 安全源扫描器。

## Force.com 安全源扫描器

Force.com 安全源扫描器是 Force.com 平台上的基于云的代码分析工具。这是一个免费的工具，供 Force.com 开发者使用，代码扫描遵循先到先得的原则。文件大小、队列大小和代码的复杂性直接影响扫描结果的时间。为了扫描特定的 Salesforce.com 用户账户，我们必须拥有“编写 Apex”权限，并且特定的代码不能包含在包中。我们可以在[`security.force.com/security/tools/forcecom/scanner`](http://security.force.com/security/tools/forcecom/scanner)提交代码进行扫描。此工具扫描所有可能的代码流程，并检查 Apex 代码的安全漏洞和质量。Force.com 安全源扫描器可以检测以下安全漏洞。

+   跨站脚本

+   SOQL 注入

+   SOSL 注入

+   框架欺骗

+   访问控制问题

Force.com 安全源扫描器可以检测以下代码和设计问题：

+   循环中的 DML 语句

+   循环中的 SOQL/SOSL

+   未将 Apex 方法批量化

+   循环中的异步（`@future`）方法

+   硬编码 ID

+   硬编码`Trigger.new[0]`

+   硬编码`Trigger.old[0]`

+   引用静态资源

+   没有 WHERE 子句或没有 LIMIT 子句的查询

+   同一对象上的多个触发器

# 跨站脚本（XSS）

跨站脚本攻击会攻击存在恶意客户端脚本或 HTML 的 Web 应用程序。如果 Web 应用程序包含恶意脚本，攻击者就可以利用 Web 应用程序作为中间层，使受信任的用户成为攻击的受害者。当动态生成的 Web 页面显示无效、未经过滤和未编码的用户输入时，就会发生跨站脚本弱点，这允许攻击者将恶意脚本嵌入到生成的页面中。这可以被利用来执行脚本代码，就像它来自网站的服务器一样，发送到任何使用该网站的用户的计算机上。

Force.com 平台有几种方法来防止 XSS（跨站脚本）攻击，具体如下：

+   **Visualforce 页面中的未转义输出和公式**：可能存在依赖于用户输入的 Visualforce 页面，并且后续功能将使用该用户输入继续。有一些编码函数可以阻止 XSS 漏洞，具体如下：

    +   `HTMLENCODE`：此函数将文本和合并字段值编码为保留 HTML 字符。例如，大于号(>)编码为&gt;。

    +   `JSENCODE`：此函数通过插入转义字符来编码文本和合并字段值。

    +   `JSINHTMLENCODE`：此函数执行`HTMLENCODE`和`JSENCODE`函数的任务。

    +   `URLENCODE`：此函数通过替换 URL 中的非法字符来编码文本和合并字段值。例如，空格被替换为%20。

### 提示

所有标准 Visualforce 组件（以`<apex>`开头）都是防 XSS 的。它们有一个过滤器来阻止 XSS 攻击。可选地，我们可以通过将 escape 属性的值设置为 false 来禁用 Visualforce 组件上的转义。

# 跨站请求伪造（CSRF）

互联网无法，也无法充分验证一个格式良好、有效、一致请求是否是提交请求的用户有意提供的。实际上，当服务器收到请求时，它无法确定该请求是由有效用户还是攻击者发起的，这可能导致权限提升或数据盗窃攻击的潜在升级。

Force.com 平台在标准控制器中实现了反 CSRF（跨站请求伪造）机制。每个页面都有一个作为隐藏字段的随机字符。当我们加载下一页时，会检查其有效性，并在值与预期值匹配后执行命令。

以下代码在名为`AutoRun`的自定义方法中绕过了反 CSRF 控制。在 Force.com 平台中，此类场景没有内置的反 CSRF 控制。有一些解决方案可以在执行操作之前添加一个中间确认页面，并缩短组织的空闲会话超时时间：

```js
<apex:page controller="SecurityIssuesController" sidebar="false" action="{!AutoRun}"> 

public class SecurityIssuesController{
public Pagereference AutoRun(){
  Id id = ApexPages.currentPage().getParameters().get('id');
  Item__c singleItem = [select id, Name FROM Item__c WHERE id = :id];
  delete singleItem;	
  return null;
}
}
```

# SOQL 注入

最常见的注入攻击发生在用户的输入直接涉及查询或命令时。因此，攻击者可以传递不受信任的数据来执行特定的功能或命令。然后攻击者将获得访问未经授权数据的权限。

Apex 使用 SOQL 作为查询语言，其功能比 SQL 有限。但 SOQL 注入攻击与 SQL 注入攻击类似。Salesforce.com 用户愿意将敏感数据放入 Salesforce，因为 Salesforce.com 是一个安全平台。因此，当我们构建自定义页面和自定义控制器时，我们必须更加注意防止此类攻击。在 Force.com 中，SOQL 注入发生在动态 SOQL 查询中。

动态 SOQL 用于在 Apex 代码的运行时创建 SOQL 查询字符串，并允许我们构建更灵活的功能（例如，依赖于用户输入的搜索功能）。使用 `Database.query(queryString)` 方法，我们可以创建动态查询，返回单个 `sObject` 或 `sObjects` 列表。如果应用程序在 Apex 中使用用户的输入来构建动态 SOQL 而我们没有正确处理输入，则可以在 Apex 中实现 SOQL 注入。

Force.com 平台提供了一个名为 `escapeSingleQuotes` 的方法来防止 SOQL 注入。使用此方法，我们可以通过在用户输入字符串中的所有单引号添加转义字符（`\`）来处理用户的输入。基本上，此方法将所有单引号视为字符串的定界符，而不是数据库命令。

以下示例说明了 Apex 中 SOQL 注入的漏洞。此查询返回未交付的订单记录以及特定订单的客户的名称（`cusName`），该名称是根据用户输入找到的。

```js
String queryString = 'SELECT Id FROM Order__c WHERE (Delivered__c = false and Customer__r.Name like \'%' + cusName + '%')';
```

如果用户输入是 `chamil`，执行查询字符串将如下所示：

```js
queryString = SELECT Id FROM Order__c WHERE (Delivered__c = false and Customer__r.Name like '% chamil %')
```

这是一个干净的输入。但问题是用户可以输入恶意输入，例如，`chamil%'` 或 `Customer__r.Name like ''`。然后查询字符串将如下所示：

```js
queryString = SELECT Id FROM Order__c WHERE (Delivered__c = false and Customer__r.Name like '%chamil%') or (Name like '%')
```

在这种情况下，查询的结果将不会返回选择性的订单，而是将数据库中的所有订单都发送出来。这是 SOQL 注入的影响。有一种方法可以保护免受此类 SOQL 注入攻击。我们可以使用一个字符串变量来分配用户输入，并将该特定变量添加到动态查询中。以下是为前述漏洞提供的固定代码片段：

```js
String userInput = '%' + cusName  + '%'; 
String queryString = 'SELECT Id FROM Order__c WHERE (Delivered__c = false and Customer__r.Name like: userInput)';
```

# 数据访问控制

Force.com 平台允许我们配置对象权限（读取、创建、编辑和删除）并创建数据共享规则。我们可以使用这些功能来实现安全控制。标准控制器遵循这些安全设置。但是，自定义控制器和控制器扩展可以在执行过程中访问所有数据。这是默认行为，但我们可以使用 `with sharing` 关键字从 Apex 类中控制数据访问。该关键字的使用方法如下：

```js
public with sharing class ExampleController {
  public void methodOne()
{
      List<Item__c> = [Select Id, Name FROM Item__c WHERE Id IN: itemIds];
}
}
```

`with sharing`关键字强制 Apex 类考虑已登录用户的权限共享安全设置。

# 摘要

我们学习了保护应用程序的重要性。我们熟悉了可能存在的漏洞、解决这些漏洞的方法以及安全扫描工具。

每个开始都有一个结束，因此我们已到达本书的结尾。我们涵盖了帮助你提高 Visualforce 开发知识的最重要主题。此外，你可以通过在 Twitter 上使用`#askforce`和[`blogs.developerforce.com`](https://blogs.developerforce.com)来使用 Force.com 资源，例如 Force.com 讨论板（你可以在技术问题上获得帮助）。

愿力量与你同在！
