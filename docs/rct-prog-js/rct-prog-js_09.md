# 第九章。使用实时示例演示 JavaScript 中的函数式响应式编程第 II 部分 - 待办事项列表

在本章中，我们将演示一个待办事项列表。这个待办事项列表将说明一种略微晦涩的双向数据绑定。ReactJS 的长处是通过单向数据绑定，大多数问题都可以按照惯用的 ReactJS 方式解决，通常会遵循冯·诺伊曼模型的单向数据绑定（据称通常不需要双向数据绑定，而《AngularJS：坏处》等文章表明，双向绑定默认情况下带来了沉重的代价，特别是在扩展方面）。

如果我们以一种明显的方式构建待办事项列表，复选框将无响应。我们可以点击它们任意次数，但它们永远不会被选中，因为单向数据绑定使用 props 或者在我们的情况下，状态来确定它们是否被选中。我们将尝试双向数据绑定，这意味着复选框不仅是活动的，而且点击复选框还会更新状态。这意味着用户界面中显示的内容和作为状态的幕后内容是相互同步的。

待办事项列表作为一个独特的特性，提供的不仅仅是**已完成**或**未完成**的状态。它还有**重要**、**进行中**、**问题**等复选框。

本章将演示以下内容：

+   使用插件的要领

+   设置适当的初始状态

+   使`TEXTAREA`中的文本可编辑

+   执行一些繁重工作的`render()`函数

+   `render()`使用的内部函数

+   构建表格以显示

+   渲染我们的结果

+   在视觉上区分列

# 向我们的应用程序添加待办事项列表

在上一章中，您实现了一个`YouPick`占位符，用于创建自己的应用程序。在这里，我们将对其进行注释，以便仅显示我们的待办事项应用程序（我们可以，而且我们将在屏幕的不同部分安排事物，但现在我们一次只显示一件事物）。在 JSX 中，注释掉代码的方法是将其包装在 C 风格的多行注释中，然后将其包装在花括号中，因此`<YouPick />`变成了{/* <YouPick /> */}：

```js
  var Pragmatometer = React.createClass(
    {
    render: function()
      {
      return (
        <div className="Pragmatometer">
        <Calendar />
        <Todo />
        <Scratch />
        {/* <YouPick /> */}
        </div>
      );
    }
  }
);
```

## 在我们的项目中包括 ReactJS 插件

我们将打开`Todo`类并包括`React.addons.LinkedStateMixin`函数。请注意，我们在这里使用了一个插件，这意味着当我们在页面中包含 ReactJS 时，我们需要包含一个包含插件的构建。因此，请考虑这一行：

```js
//cdnjs.cloudflare.com/ajax/libs/react/0.13.3/react.min.js
```

我们包括以下内容：

```js
//cdnjs.cloudflare.com/ajax/libs/react/0.13.3/react-with-addons.min.js
```

`Todo`类的开头如下所示：

```js
  var Todo = React.createClass(
    {
      mixins: [React.addons.LinkedStateMixin],
```

## 设置适当的初始状态

初始状态为空；列表中没有待办事项，新的待办事项文本也为空：

```js
      getInitialState: function()
      {
        return {
          'items': [],
          'text': ''
        };
      },
```

请注意，一些初始状态设置可能涉及繁重的工作；在这里，它非常简单，但情况并非总是如此。

## 使文本可编辑

作为一个小的清理细节，当有人在框中输入时，我们希望行为明显。因此，我们为 TEXTAREA 定义了双向数据绑定，这样如果有人在 TEXTAREA 中输入，更改将被添加到状态中，并溢出回到 TEXTAREA 中：

```js
      onChange: function(event)
      {
        this.setState({text: event.target.value});
      },
```

如果有人在输入一些文本后点击**提交**按钮提交新的待办事项，我们将该项目添加到列表中：

```js
      handleSubmit: function(event)
      {
        event.preventDefault();
        var new_item = get_todo_item();
        new_item.description = this.state.text;
        this.state.items.push(new_item);
        var next_text = '';
        this.setState({text: next_text});
      },
```

## `render()`的繁重工作

`render()`函数稍微复杂，包含内部函数和基于双向数据绑定的响应式用户界面的大部分繁重工作。在其中，我们使用了`var that=this;`模式，这在大多数 ReactJS 代码中都是不存在的。在大多数 ReactJS 中，我们可以直接使用 this，它会自动工作；在这里，我们正在定义不像其他 ReactJS 函数那样直接构建的内部函数，并保留对 this 对象的引用：

```js
      render: function()
      {
        var that = this;
        var table_rows = [];
```

## 用于渲染的内部函数

`table_rows`数组将保存待办事项。定义了这些之后，我们定义了我们的第一个内部匿名函数`handle_change()`。如果用户点击待办事项的复选框之一，我们提取 HTML ID，该 ID 告诉它的待办事项 ID，以及已切换的字段（即复选框标识符）：

```js
        var handle_change = function(event)
        {
          var address = event.target.id.split('.', 2);
          (that.state.items[parseInt(address[0])][address[1]] = !that.state.items[parseInt(address[0])][address[1]]);
        };
```

`display_item_details()`函数是用于构建显示的几个函数中最低级的一个。它构建了一个包含复选框的单个 TD：

```js
      var display_item_details = function(label, item)
          {
          var html_id = item.id + '.' + label;
        return ( <td className={label} title={label}>
            <input onChange={handle_change} 
              id={html_id} className={label} type="checkbox" checked={item[label]} />
          </td>
        );
      };
```

接下来，`display_item()`使用这些构建块来构建待办事项的显示。除了包括渲染的节点，也就是复选框，它还对框中的文本应用了 Markdown 格式：

```js
      var display_item = function(item)
      {
        var rendered_nodes = [];
        for(var index = 0; index < todo_item_names.length;
        index += 1) {
          rendered_nodes.push(
            display_item_details(todo_item_names[index], item)
          );
        }
        return ( <tr>
          {rendered_nodes}
          <td dangerouslySetInnerHTML={{__html:
          converter.makeHtml(item.description)}} />
        </tr>
        );
      };
```

## 构建结果表

对于每个项目，我们添加一个表格行：

```js
      table_rows.push(
      <tr>{this.state.items.map(display_item)}</tr>);
```

最后，返回一个包含到目前为止计算的各种值的 JSX 表达式，并将它们包装在一个功能齐全的表格中：

```js
      return (
        <form onSubmit={this.handleSubmit}>
          <table>
            <thead>
              <tr>
                <th>To do</th>
              </tr>
            </thead>
            <tbody>
              {table_rows}
            </tbody>
            <tfoot>
              <textarea onChange={this.onChange}
              value={this.state.text}></textarea><br />
              <button>{'Add activity'}</button>
            </tfoot>
          </table>
        </form>
      );
    }
  }
);
```

当选中应该隐藏和显示它们的复选框时，将数据行隐藏和显示是留给你作为练习的。

### 提示

关于表格的使用，这里有一个简短的备注：从主要使用表格转变为主要使用 CSS 进行格式化。然而，关于表格使用的确切规则并不是完全“根本不使用表格”或者“只有在确实必须使用表格时才使用”，而是“用于表格数据的表格”。这意味着像我们这里显示的网格。具有复选框网格的表格是表格在当前标记中完全适当的一个很好的例子。

## 呈现我们的结果

我们只有在告诉它时，结果才会呈现出来；这可以被视为一种繁琐，也可以被视为我们端的一种额外自由度。在结束闭包之前，我们写下了这个：

```js
var update = function()
{
  React.render(<Pragmatometer />,
  document.getElementById('main'));
};
update();
var update_interval = setInterval(update, 100);
```

## 视觉上区分列

目前，我们的未区分的复选框网格混在一起。我们可以做一些事情来区分它们。`index.html`中的 CSS 中的一系列颜色将它们区分开来：

```js
      td.Completed {
        border-left: 3px solid black;
        background-color: white;
      }
      td.Delete {
        background-color: gray;
      }
      td.Invisible
      {
        background-color: black;
      }
      td.Background
      {
        background-color: #604000;
      }
      td.You.Decide
      {
        background-color: blue;
      }
      td.In.Progress
      {
        background-color: #00ff00;
      }
      td.Important
      {
        background-color: yellow;
      }
      td.In.Question
      {
        background-color: darkorange;
      }
      td.Problems
      {
        background-color: red;
      }
```

# 摘要

在本章中，我们漫游了一个略微超过最小限度的待办事项列表。就功能而言，我们看到了一个以包括双向数据绑定的方式构建的交互式表单。ReactJS 通常建议，大多数情况下，你认为你需要双向数据绑定，但你真的最好只使用单向数据绑定。然而，该框架似乎并不打算作为一种约束，当 ReactJS 说，“你通常不应该做 X”时，是有办法做 X 的。对于`dangerouslySetInnerHTML`和双向数据绑定，你可以在特定点选择使用它，但已经尽一切努力选择更好的工程。`dangerouslySetInnerHTML`函数是一种非常有力的命名方式，ReactJS 团队明确表达的观点是冯·诺伊曼模型要求至少在大多数情况下使用单向数据绑定。然而，ReactJS 的哲学明确允许开发人员使用他们认为最好通常要避免的功能；最终的裁决权在你手中。

在我们下一章中，加入我们，我们将创建一个优雅处理重复预约的日历应用程序。
