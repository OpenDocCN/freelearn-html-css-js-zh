# 第八章：Visualforce 开发的最佳实践

Visualforce 页面是 Salesforce 标准页面的替代品。当我们使用 Visualforce 页面时，不应存在延迟体验和意外行为。因此，我们必须遵循最佳实践，在 Visualforce 开发过程中提高用户体验和编码标准，以改善用户体验。在某些情况和组件中，我们可以应用一些最佳实践。本章将涵盖以下主题：

+   访问组件 ID

+   页面块组件

+   控制器和控制器扩展

+   提高 Visualforce 的性能

+   静态资源

+   渲染 PDF

+   使用组件面

让我们构建具有超级性能的 Visualforce 页面…

# 访问组件 ID

当我们在 JavaScript 中引用 Visualforce 组件时，ID 属性起着重要作用。每个 Visualforce 组件都有一个 ID 属性。为了在 JavaScript 中引用它，必须指定特定组件的 ID 属性，并且它用于将两个组件绑定在一起。当页面渲染时，这个 ID 属性是特定组件 DOM ID 的一部分。ID 属性也必须是唯一的。以下是一些用于访问组件 ID 的最佳实践：

+   使用`$Component`全局变量来简化访问。例如，当我们有一个`id="inputOne"`的输入字段在`id="blockOne"`的页面块内时，我们可以使用`$Component.blockOne.inputOne`表达式来访问输入字段。

+   如果要访问的组件是 Visualforce 组件层次结构中`$Component`变量的祖先或兄弟，则无需为该组件指定 ID。

# 页面块组件

`<apex:pageBlockSectionItem>`组件只能有两个子组件。根据客户需求和开发需求，`<apex:pageBlockSectionItem>`内部可以有超过两个子元素。使用`<apex:outputPanel>`，我们可以在`<apex:pageBlockSectionItem>`中添加超过两个元素，如下所示：

```js
<apex:pageBlock>
  <apex:pageBlockSection>
    <apex:pageBlockSectionItem>
      <apex:outputLabel value="LabelName"/> 
      <apex:outputPanel>
<!—We can add our extra child components here within the apex:outputPanel --> 
      </apex:outputPanel>
    </apex:pageBlockSectionItem>
  </apex:pageBlockSection>
</apex:pageBlock>
```

# 控制器和控制器扩展

当我们开发与 Visualforce 页面关联的控制器和控制器扩展时，我们需要遵守以下最佳实践：

+   通过使用`with sharing`关键字，我们可以在控制器中强制执行共享规则。然后代码将在用户模式下执行，而不是系统模式下。

+   我们不应依赖于在构造函数之前执行 setter 方法。

+   在自定义控制器或控制器扩展中创建自定义方法时，我们不应依赖于执行顺序或副作用。

+   不要在循环中使用 DML 操作。

+   在执行记录过滤时，按以下顺序添加过滤器：

    +   在 SOQL 中

    +   在 Apex 中

    +   在 Visualforce 中

+   如果可能，必须在 SOQL 中而不是 Apex 中进行计算。

# 提高 Visualforce 的性能

Visualforce 页面的性能是开发中需要考虑的关键因素，因为性能是影响最终用户对应用程序满意度的原因。以下是一些提高 Visualforce 性能的最佳实践：

+   每个 Visualforce 页面只使用一个 `<apex:form>` 标签，因为每个 `<apex:form>` 标签都会给页面添加一个视图状态。Visualforce 页面的视图状态大小限制为 135 KB。我们可以通过减少视图状态的大小来缩短 Visualforce 页面的加载时间。

+   尽可能地在自定义控制器中使用 transient 关键字。transient 实例变量不维护状态。如果特定实例仅在页面请求中使用，则它不应是视图状态的一部分。这将有助于减少视图状态的大小。

+   当使用 SOQL 查询引用特定对象的属性时，只使用 SOQL 查询中的相关数据。

+   在设计 Visualforce 页面时，不要让页面承载过多的功能和数据。过载的页面会增加视图状态、页面大小、堆大小，并可能导致视图状态达到限制器限制。

+   为了减少 Visualforce 页面的加载时间：

    +   不要在 getter 方法中使用 SOQL 查询。

    +   经常使用或全局数据必须进行缓存。

    +   通过使用 `standardSetControllers` 的内置分页或限制 SOQL 查询中的数据来减少页面中显示的记录数。

+   我们可以通过使用 `<apex:actionPoller>` 组件来减少多个并发请求的延迟，从而增加调用 Apex 控制器的间隔时间。

+   使用 SOQL 的 `OFFSET` 实现特定子集结果的分页。

+   如果一个组织中有大量只读数据，那么我们必须使用自定义对象或自定义设置来存储这些数据。

+   使用 `<apex:repeat>` 在大型集合上迭代。

+   不要在 Visualforce 页面中硬编码选择列表值，而是使用控制器将它们添加到 `selectOption` 列表中。

+   使用延迟加载方法，根据必要性减少或延迟数据的加载。在延迟加载中，基本功能将首先加载，其他功能将延迟到用户操作时再加载。为了实现延迟加载，我们可以：

    +   使用 `rerender` 属性执行部分页面加载。

    +   使用 JavaScript remoting 调用控制器中的操作。

+   在 Visualforce 页面中使用 CSS 时，我们必须小心。Visualforce 的性能直接受到 Visualforce 优化的影响。以下是一些提高 CSS 性能的技巧：

    +   使用单独的 CSS 文件，并在 Visualforce 页面中引用它们（而不是在页面本身中编写 CSS 代码）。这将减小文件大小。

    +   使用单个 CSS 文件而不是多个 CSS 文件。这将减少 HTTP 请求的数量。

    +   通过静态资源引用 CSS 文件，因为它具有内置的缓存机制。

    +   当创建完全自定义 CSS 的页面（不使用 Salesforce CSS）时，不要忘记将 `<apex:page>` 标签的 `showHeaders` 和 `standardStylesheets` 属性设置为 `false`。

+   当在 Visualforce 页面中使用 JavaScript 时，我们必须优化它们以提高 Visualforce 的性能。以下是一些提高 JavaScript 性能的提示：

    +   使用单独的 JavaScript 文件并在 Visualforce 页面中引用它们（而不是在页面本身中编写 JavaScript）。这将减少利用浏览器缓存的单个页面的大小。

    +   使用单个 JavaScript 文件而不是使用多个 JavaScript 文件。这将减少 HTTP 请求的数量并移除重复的函数。

    +   使用仅包含所需功能的自定义版本的 JavaScript 库。这将减少文件大小。

    +   如果可能，使用 `<script>` 标签在 Visualforce 页面中包含 JavaScript，并将其放置在 `</apex:page>` 关闭标签之前。这将避免在 Visualforce 页面的任何其他内容之前加载 JavaScript。

    +   删除不必要的注释和空白，以减少文件大小并加快下载速度。

+   使用 `escapeSingleQuotes` 方法来避免 SOQL 和 SOSL 注入攻击。

+   图片在 Visualforce 页面中经常扮演重要角色。因此，我们必须使用以下提示来优化 Visualforce 页面中图片的使用：

    +   使用 CSS Sprites 而不是单个图片。使用 Sprites，我们可以将（相似大小的）图片合并成一个文件。这将减少页面中使用的图片数量并减少 HTTP 请求的数量。

    +   如果可能，尽量减少图片的使用并鼓励使用 CSS。

    +   使用静态资源来引用 Visualforce 页面中的图片。

### 提示

由于视图状态数据被加密，无法使用 Firebug 等工具查看视图状态。

# 静态资源

静态资源具有内置的缓存功能，并使用 Salesforce 内置的内容分发网络。使用静态资源引用 CSS 文件、图片和 JavaScript 的以下是一些优点：

+   使用静态资源通过 `<apex:page>` 标签的动作属性显示另一个静态资源的内容。通过这样做，我们可以从 Visualforce 页面重定向到静态资源。假设我们有一个名为 `helpPdf` 的 PDF 作为静态资源，我们将其用作 `<apex:page>` 标签的动作属性，如下所示：

    ```js
    <apex:page sidebar="false" showHeader="false" standardStylesheets="false" action="{!URLFOR($Resource.helpPdf)}">
    </apex:page>
    ```

+   `URLFOR` 函数在这里起着重要作用。没有 `URLFOR` 函数，重定向将无法正常工作。这不仅仅限于 PDF；我们可以使用任何静态资源进行重定向。

    ```js
    <apex:page sidebar="false" showHeader="false" standardStylesheets="false" action="{!URLFOR($Resource.helpStaticResource, 'index.htm')}">
    </apex:page>
    ```

# 渲染 PDF

当我们在 Visualforce 页面中使用组件并且页面被渲染为 PDF 时，这些组件并不总是工作。我们不得使用依赖于 JavaScript 动作和 Salesforce 标准样式的组件。

+   以下组件在 PDF 渲染中是安全的：

    +   `<apex:composition>`（只要页面包含安全的 PDF 组件）

    +   `<apex:facet>`

    +   `<apex:dataList>`

    +   `<apex:define>`

    +   `<apex:include>`（只要页面包含安全的 PDF 组件）

    +   `<apex:insert>`

    +   `<apex:image>`

    +   `<apex:repeat>`

    +   `<apex:outputLabel>`

    +   `<apex:outputLink>`

    +   `<apex:outputPanel>`

    +   `<apex:outputText>`

    +   `<apex:page>`

    +   `<apex:panelGrid>`

    +   `<apex:panelGroup>`

    +   `<apex:param>`

    +   `<apex:stylesheet>`（只要 URL 没有直接引用 Salesforce 样式表）

    +   `<apex:variable>`

+   以下组件在渲染 PDF 时可以谨慎使用（其他组件在 PDF 渲染中不安全）：

    +   `<apex:attribute>`

    +   `<apex:column>`

    +   `<apex:component>`

    +   `<apex:componentBody>`

    +   `<apex:dataTable>`

# 使用组件面

`<apex:facet>`组件用于指定 Visualforce 页面区域的内容，并提供有关父组件中数据的信息。例如，我们可以在`<apex:dataTable>`的页眉或页脚中使用面组件。我们可以通过使用`<apex:facet>`组件来覆盖 Visualforce 组件的默认面。以下是一个关于面组件的优缺点示例：

+   `<apex:facet>`组件不能直接在 Apex 中使用；它必须是另一个 Visualforce 组件的子组件。我们可以在动态组件中使用它。

+   面仅允许在开始和结束标签内有一个子组件。

+   以下是一个使用`<apex:facet>`组件与`<apex:dataTable>`组件一起使用的示例：

    ```js
    <apex:page standardController="Item__c">
      <apex:pageBlock>
        <apex:dataTable value="{!item}" var="i">
          <apex:facet name="caption"><h1>This is {!item.Item_Name__c}</h1></apex:facet>      
          <apex:column>
          <apex:facet name="header">Name</apex:facet>
          <apex:outputText value="{!i.Item_Name__c}"/>
          </apex:column>
          <apex:column>
          <apex:facet  name="header">Unit Price</apex:facet>
            <apex:outputText value="{!i.Unit_Price__c }"/>
          </apex:column>
        </apex:dataTable>
      </apex:pageBlock>
    </apex:page>
    ```

+   我们可以使用面组件与`<apex:actionSatus>`一起使用。它用于扩展显示状态指示器。以下是一个示例：

    ```js
    <apex:page>
      <apex:form >
        <apex:commandButton value="Facet with action Status" status="Status" rerender="rerenderBlock"/>
        <apex:pageBlock id="rerenderBlock">
        </apex:pageBlock>
        <apex:actionStatus id="Status">
          <apex:facet name="start">
            <img src="img/{!$Resource.LoadingAnimation}"/> 
          </apex:facet>
        </apex:actionStatus>
      </apex:form>
    </apex:page>
    ```

# 摘要

本章旨在解释 Visualforce 开发的最佳实践。在本章中，我们熟悉了遵循的最佳实践，以避免意外行为，减少访问组件 ID、页面块组件、控制器和控制器扩展、提高 Visualforce 性能、静态资源、渲染 PDF 和使用组件面的延迟体验。我们看到了提高用户体验和编码标准的方法。
