

# 第四章：组合组件

随着您的应用程序的增长，将所有逻辑都塞入单个组件变得不切实际。您需要将应用程序拆分为更小的、模块化的组件，并将它们组装成更复杂的应用程序。

在本章中，我们将探索各种有效组合组件的技术。我们将首先检查如何使用插槽向组件注入自定义内容。然后，我们将讨论如何在组件内条件性地渲染不同的 HTML 元素。我们还将深入研究递归组件，这对于显示嵌套或分层数据非常有用。

我们将通过实际示例引导您了解每个主题，确保您学习的技巧既实用又适用于现实世界的场景。到本章结束时，您将拥有一套更丰富的策略来构建您的 Svelte 应用程序中的组件。

在本章中，您将学习以下内容：

+   从父组件的角度操纵子组件的外观

+   通过插槽传递动态内容

+   渲染不同的 HTML 元素和组件类型

+   为递归数据创建递归组件

+   容器/表现模式

让我们从探索我们可以操纵和控制子组件内容的各种方式开始。

# 技术要求

本章中所有的代码都可以在[`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter04`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter04)找到。

# 操纵子组件的外观

当您组合多个组件时，您需要管理每个子组件的外观和行为。尽管子组件处理自己的显示和逻辑，但它仍然提供了一些控件来调整其外观和行为。从父组件的角度来看，您将想要协调这些子组件以实现所需的总体功能。

在本节中，我们将探索控制子组件外观的各种方法，从最常用到最不常用的方法。了解这些选项将使您拥有制作既灵活又有效的组件的工具。

控制子组件外观的选项列表包括以下内容：

+   **通过 props 控制**：这可能是在子组件的行为和外观上产生影响的最直接方式。通过从父组件传递 props 到子组件，您可以使子组件高度动态并对外部变化做出响应。

    为了演示父组件如何使用 props 在 Svelte 中控制内容，请考虑以下代码示例：

    ```js
    <script>
      import Child from './Child.svelte';
      let message = 'Hello from Parent';
    </script>
    <Child message={message} />
    Child component and passed the message prop to it. The child component, implemented in the following code snippet, then uses this message prop to display text within a <p> element:
    ```

    ```js
    <script>
      export let message;
    </script>
    <p>{message}</p>
    ```

    正如你所见，父组件可以通过修改`message`属性值的值来指定`Child`组件中显示的文本。控制属性是一种简单而有效的方式来操纵子组件的内容。如果你对属性感兴趣，我们在*第三章*中对此进行了广泛讨论。

+   `parent`组件：

    ```js
    <script>
      import Child from './Child.svelte';
      import { setContext } from 'svelte';
      setContext('message', 'Hello from parent');
    </script>
    <Child />
    ```

    在之前的代码中，我们使用`setContext`创建了一个名为`message`的上下文，其值为`'Hello from parent'`，然后导入并使用了`Child`组件，而没有向其发送任何属性。

    以下是为`Child`组件的代码：

    ```js
    <script>
      import { getContext } from 'svelte';
      const message = getContext('message');
    </script>
    <p>{message}</p>
    ```

    在这里，使用`getContext`读取`message`上下文值，然后在一个`<p>`元素中显示。如所示，父组件可以通过更改上下文值来影响子组件的`<p>`元素中的文本。

    若要深入了解 Svelte 的上下文功能，你可以参考*第八章*，我们将更详细地探讨这个主题。

+   **控制样式**：操纵子组件的外观并不仅仅是控制传递给它的数据。它还涉及到调整或修改其样式。

    你可以通过 CSS 自定义属性来修改组件样式。这种方法提供了更大的设计灵活性，并确保子组件能够适应父组件或更广泛应用程序中的各种上下文或主题。

    若要深入了解如何使用 CSS 自定义属性来更改组件样式，请参阅*第二章*。

+   **通过插槽传递动态内容**：Svelte 的插槽功能允许你将自定义内容插入子组件的特定区域。这提供了一种灵活的方式来更好地控制组件的内容，而无需修改其内部行为。

    在下一节中，我们将讨论插槽及其使用方法。

正如你所见，有各种方式来塑造子组件的外观和行为。当我们在一个父组件内组合不同的组件时，目标是使它们以协调的方式协同工作。你可以使用所讨论的方法的组合来实现这一点。

大多数这些方法将在单独的章节中介绍，在下一节中，我们将关注如何通过传递内容通过插槽来动态改变子组件的外观。

# 通过插槽传递动态内容

在构建复杂应用程序时，一种大小并不总是适合所有情况。在组件的模块化和定制灵活性之间取得平衡是至关重要的。

以一个通用的`Card`组件为例。有时你可能希望为特定用例包含特殊的标题、独特的列表或自定义页脚。几乎不可能预见所有需求，因此设计既模块化又可维护的组件，同时仍然允许定制，是至关重要的。

这正是 Svelte 槽位功能大放异彩的地方。`Card` 组件旨在包含所有可能的功能，目标是简单、干净的基座，可以通过组合进行增强。这种方法允许您根据需求的变化，拼凑出更复杂、定制的组件。

在 Svelte 组件中，一个 `<slot>` 元素是组件内的一个占位符，您可以在其中注入来自父组件的任何类型的内容。以下是您如何在 Svelte 组件中定义槽位的方法：

```js
<!-- filename: Card.svelte -->
<div class="card">
  <slot></slot>
</div>
```

在前面的代码片段中，我们在 `<div>` 元素内部定义了一个 `<slot>` 元素。任何来自父组件的动态内容都将插入到 `<div>` 元素中。

现在，要将父组件中的动态内容传递到子组件的槽位中，您需要将该内容放置在子组件的开闭标签之间：

```js
<script>
  import Card from './Card.svelte';
</script>
<Card>
  <h1>Special headline</h1>
</Card>
```

在此代码片段中，我们在 `<Card>` 组件的开闭标签之间传递了一个 `<h1>` 元素。这实际上用我们提供的 `<h1>` 元素替换了 `Card` 组件中的 `<slot>` 元素。

可能存在不需要从父组件将动态内容传递到子组件槽位的情况。对于这些情况，Svelte 允许您在 `<slot>` 元素内定义默认内容，当父组件没有提供自定义内容时提供回退选项。

## 提供默认槽位内容

要为您的 `<slot>` 元素提供默认内容，您可以将它们放置在 `<slot>` 元素内部，如下所示：

```js
<div class="card">
  <slot>
    <div>this is the default content</div>
  </slot>
</div>
```

在前面的代码片段中，我们定义了 `<div>`，其中包含 `<slot>` 元素内的文本 `This is the default content`。这作为槽位的默认内容。

当您在父组件中使用此 `Card` 组件且不提供槽位内容时，如以下代码片段所示，默认文本将自动出现：

```js
<script>
  import Card from './Card.svelte';
</script>
<Card />
```

通过利用槽位的默认内容，您可以创建更灵活的 Svelte 组件，当没有提供自定义内容时，提供合理的回退选项。

在我们迄今为止看到的示例中，我们被限制为仅将来自父组件的动态内容插入到一个位置。但如果我们想要多个动态内容插入点怎么办？让我们探索如何实现这一点。

## 有多个带有命名槽位的槽位

槽位非常出色，您也不限于单个槽位。您可以在组件中定义多个槽位并为它们命名，以针对特定的动态内容区域。

您可以使用 `<slot>` 元素上的 `name` 属性来为槽位命名，如下面的代码所示：

```js
<!-- filename: Card.svelte -->
<div class="card">
  <header>
    <slot name="header"></slot>
  </header>
  <footer>
    <slot name="footer"></slot>
  </footer>
</div>
```

在前面的代码片段中，我们定义了两个命名槽位，一个命名为 `"header"`，另一个命名为 `"footer"`。

要将动态内容针对这些命名槽位，您需要在父组件中使用 `<svelte:fragment>` 元素：

```js
<script>
  import Card from './Card.svelte';
</script>
<Card>
  <svelte:fragment slot="header">
    <h1>Special headline</h1>
  </svelte:fragment>
  <svelte:fragment slot="footer">
    Custom footer
  </svelte:fragment>
</Card>
```

在这里，我们向 `Card` 组件传递了两个 `<svelte:fragment>` 元素。第一个具有槽位属性设置为 `"header"`，第二个槽位属性设置为 `"footer"`。

在`slot`属性中指定的名称将与`Card`组件内的命名插槽相对应。在`<svelte:fragment slot="header">`内的内容替换了`Card`组件中的`<slot name="header">`。同样，在`<svelte:fragment slot="footer">`内的内容取代了组件中的`<slot name="footer">`。

有时，你可能希望父组件的动态内容了解子组件中的数据。为了处理这种情况，Svelte 提供了一个称为插槽属性的功能，允许你从子组件将数据传递到插槽中的内容。让我们看看它是如何工作的。

## 通过插槽属性传递数据

你可以使用插槽属性，如下所示，从子组件将数据传递回父组件中的动态内容：

```js
<!-- filename: Card.svelte -->
<div class="card">
  <header>
    <slot name="header" width={30} height={50}></slot>
  </header>
</div>
```

在代码示例中，我们除了传递了用于命名插槽的`name`属性外，还向`<slot>`元素传递了两个额外的属性。这些额外的属性，`width`和`height`，作为插槽属性；它们的值可以在创建动态内容时在父组件中访问。

这里有一个示例，说明你如何在使用`<svelte:fragment>`时使用这些插槽属性值：

```js
<script>
  import Card from './Card.svelte';
</script>
<Card>
  <svelte:fragment slot="header" let:width let:height>
    <h1>Dimension: {width} x {height}</h1>
  </svelte:fragment>
</Card>
```

在前面的代码片段中，你可以看到我们在`<svelte:fragment>`元素内部使用了`let:width`和`let:height`。这些被称为`let:`绑定。

`let:`绑定使我们能够访问由子`Card`组件提供的`width`和`height`插槽属性。然后，这些宽度和高度的值在`<h1>`元素中使用，以显示尺寸。这样，我们可以根据来自子组件的数据动态调整父组件中的内容。

现在我们已经涵盖了如何通过插槽传递动态内容、如何定义多个命名的插槽以及如何将子组件中的数据传递给父组件中的动态插槽内容，你现在已经具备了创建更灵活和可重用 Svelte 组件的能力。

无渲染组件

使用插槽组合组件的许多常见模式之一是无渲染组件模式。这种技术允许你构建纯函数组件，将展示细节留给父组件。我们专门在*第十一章*中深入探讨了这一迷人的主题。

现在我们已经探讨了如何通过插槽传递动态内容，接下来让我们深入了解 Svelte 的另一个令人兴奋的特性，它允许你动态渲染不同的元素或组件类型。

# 渲染不同的 HTML 元素或组件类型

在任何动态应用程序中，都会有一个时刻你需要比静态组件或元素提供的更多灵活性。假设你在运行时才知道需要渲染的元素或组件的类型呢？

让我们假设您正在构建一个表单生成器，表单字段的类型——无论是 `<Input>`、`<Checkbox>` 还是 `<Select>`——由动态数据决定。您如何在这些组件之间无缝切换，尤其是当它们共享同一组属性时？

一个简单的方法是使用 Svelte 的 `{#if}` 块有条件地渲染所需的组件。以下是一个示例代码片段：

```js
<script>
  import Input from './Input.svelte';
  import Checkbox from './Checkbox.svelte';
  import Select from './Select.svelte';
  let type = "input"; // Could be "checkbox" or "select"
</script>
{#if type === "input"}
  <Input value={value} onChange={onChange} />
{:else if type === "checkbox"}
  <Checkbox value={value} onChange={onChange} />
{:else}
  <Select value={value} onChange={onChange} />
{/if}
```

在代码片段中，我们使用 `{#if}` 块来选择三种不同的组件类型，并将相同的属性值传递给每个组件。如果您发现自己需要管理更多组件类型，这可能导致一系列冗长且难以维护的条件块。

有没有更有效的方法来处理这个问题？

Svelte 提供了两个专用元素，`<svelte:element>` 和 `<svelte:component>`，正是为了这个目的。`<svelte:element>` 元素允许您根据变量数据动态创建不同的 HTML 元素类型，而 `<svelte:component>` 元素允许您动态渲染不同的 Svelte 组件类型。

这里是如何使用 `<svelte:component>` 重新编写上一个示例的：

```js
<script>
  import Input from './Input.svelte';
  import Checkbox from './Checkbox.svelte';
  import Select from './Select.svelte';
  let type = "input"; // Could be "checkbox" or "select"
  let DynamicComponent;
  if (type === "input") {
    DynamicComponent = Input;
  } else if (type === "checkbox") {
    DynamicComponent = Checkbox;
  } else {
    DynamicComponent = Select;
  }
</script>
<svelte:component
  this={DynamicComponent}
  value={value}
  onChange={onChange}
DynamicComponent variable is used to hold the type of component that will be rendered. This component type is then passed to the this attribute within the <svelte:component> element. The <svelte:component> element also accepts other props such as value and onChange.
With the preceding code, what happens is that `<svelte:component>` renders the designated component stored in the `this` attribute, simultaneously forwarding any props passed to `<svelte:component>` to this dynamically rendered component. For example, if the value of `DynamicComponent` is the `Select` component, then the preceding code is effectively the same as this code:

```

<Select value={value} onChange={onChange} />

```js

 By using `<svelte:component>`, we simplify the code and make it more maintainable. It’s also easier to extend; adding another form element type would only require an additional condition and assignment.
Now that we’ve explored the use of `<svelte:component>`, let’s look at `<svelte:element>`, which follows a similar pattern. The following is a sample code snippet that demonstrates the usage of `<svelte:element>`:

```

<script>

let type = 'button'; // 可能是 'div', 'p', 'a' 等。

</script>

<svelte:element this={type}>

点击我

</svelte:element>

```js

 In the preceding code snippet, the `type` variable holds the type of HTML element we want to render – in this example, it’s `button`. The `<svelte:element>` tag dynamically creates an HTML element of the type specified by `type`. So, if `type` is `'button'`, this will render as a `<button>` element, containing the text `"``Click Me"`.
This approach is particularly useful when you want to switch the type of an HTML element, based on some condition, without having to rewrite the entire block of code. All you need to do is change the value of `type`.
To summarize, `<svelte:element>` and `<svelte:component>` offer a more efficient and maintainable way to handle dynamic rendering needs. They provide a robust alternative to multiple `{#if}` blocks, making your Svelte applications more flexible and easier to manage.
Sometimes, when designing Svelte components for data visualization, we encounter recursive data structures. In such cases, we need to build components that can recursively render themselves. Let’s delve into how we can accomplish this next.
Creating recursive components for recursive data
A **recursive data structure** is a data structure that is defined in terms of itself, meaning that it can be composed of smaller instances of the same type of structure. Recursive data is everywhere – think of a comment section where replies can have their own sub-replies, or a filesystem with nested folders. Creating a component to display them in a frontend application can be challenging.
Imagine we have a variable called `folder`, which is an array containing either files or additional folders. In this example, the `folder` variable could look like this:

```

const folder = [

{ type: 'file', name: 'a.js' },

{ type: 'file', name: 'b.js' },

{ type: 'folder', name: 'c', children: [

{ type: 'file', name: 'd.js' },

]},

];

```js

 Currently, our `folder` variable is two levels deep. To represent this structure in a Svelte component, you might think to use nested `{#``each}` blocks:

```

<script>

export let folder;

</script>

<ul>

{#each folder as item}

<li>

{#if item.type === 'file'}

<div>文件: {item.name}</div>

{:else}

<div>文件夹: {item.name}</div>

<ul>

{#each item.children as child}

<li>

{#if child.type === 'file'}

<div>文件: {child.name}</div>

{:else}

...

{/if}

</li>

{/each}

</ul>

{/if}

</li>

{/each}

</ul>

```js

 In the preceding code snippet, we used an `{#each}` block to iterate over the items in the `folder` variable, rendering either a file or a folder based on `item.type`. If `item.type` is a folder, we use another nested `{#each}` block to go through its contents.
However, here’s the catch – the inner folder could also contain files or additional folders, making it recursive. As a result, we end up with repeated code for the inner and outer `{#each}` blocks. This works fine for a folder structure that’s only two levels deep, but what if it’s more complex? How can we avoid duplicating the same code for each level of nesting?
Svelte offers an elegant solution to handle such recursive structures with the `<svelte:self>` element. The `<svelte:self>` element allows a component to embed itself within its own markup, thus making it easier to handle recursive data.
Here’s the updated code using the `<``svelte:self>` element:

```

<script>

export let folder;

</script>

<ul>

{/each folder as item}

<li>

{#if item.type === 'file'}

<div>文件: {item.name}</div>

{:else}

<div>文件夹: {item.name}</div>

<svelte:self folder={item.children} />

{/if}

</li>

{/each}

</ul>

```js

 In the updated code snippet, we replaced the nested `{#each}` block with `<svelte:self folder={item.children} />`. This effectively re-renders the current component, passing `item.children` as the `folder` prop.
The beauty of this is that it eliminates the need to initiate a new `<ul>` element and duplicate `{#each}` blocks for each nesting level. Instead, the component simply reuses itself, making it capable of handling nested structures of any depth.
When comparing this with the previous code snippet, you can immediately see the advantage – it’s much cleaner and avoids repetitive code, making it easier to read and maintain, and it scales naturally to handle any level of nested folders.
Now that we’ve discussed how to tackle recursive data with `<svelte:self>`, let’s look at a practical example in the following section, where we will create a Svelte-based JSON tree viewer.
Example – a JSON tree viewer
In this section, we will walk you through building a JSON tree viewer component in Svelte. The JSON tree viewer component helps you visualize JSON data in a tree-like format. Along the way, we’ll make use of some of the advanced Svelte features we’ve covered in this chapter, including `<svelte:self>`, `<svelte:component>`, and slots.
Before we start, let’s think about what our JSON Tree Viewer should look like and how it should behave. Essentially, a JSON object is made up of key-value pairs, where the values can be either primitives, arrays, or other nested objects. Our goal is to display these key-value pairs in a way that clearly represents the hierarchical structure of the JSON object.
So, let’s create a `JsonTree` component for our JSON tree viewer:

```

<!-- filename: JsonTree.svelte -->

<script>

export let data;

</script>

<ul>

{#each Object.entries(data) as [key, value]}

<li>{key}: {value}<li>

{/each}

</ul>

```js

 In the preceding code snippet, we defined a `JsonTree` component that accepts data as a prop. Inside this component, we utilize Svelte’s `{#each}` block to generate a list of `<li>` elements. Each element displays a key-value pair from the `data` object.
However, these values can vary. They could be primitive types or nested objects. Take the following example:

```

data = {

name: 'John Doe',

address: {

street: '123 Main St'

}

}

```js

 In the preceding `data` object, the `name` key has a primitive string value, while the value for the `address` key is an object. For nested objects like this, we will need to use our `JsonTree` component recursively to render that nested object. This is where `<svelte:self>` comes into play.
Here’s the updated code:

```

<!-- filename: JsonTree.svelte -->

<script>

export let data;

</script>

<ul>

{#each Object.entries(data) as [key, value]}

<li>

{key}:

{#if typeof value === 'object'}

<svelte:self data={value} />

{:else}

{value}

{/if}

<li>

{/each}

</ul>

```js

 In this updated code snippet, we introduced an `{#if}` block that renders differently based on whether the value is a primitive type or a nested object. For nested objects, we recursively render another `JsonTree` component using `<svelte:self>`. This allows us to elegantly handle JSON objects of any depth.
Now, in our current implementation, we haven’t made distinctions between different types of primitive values. Suppose you’d like to render various primitive types using specialized Svelte components. In that case, you could dynamically switch between different component types using `<svelte:component>`.
Let’s assume you have three separate Svelte components – `StringValue` for strings, `NumberValue` for numbers, and `JsonValue` for others (such as Booleans, `null`, and `undefined`). Here is how you can update the code to use `<svelte:component>`:

```

<script>

import StringValue from './StringValue.svelte';

import NumberValue from './NumberValue.svelte';

import JsonValue from './JsonValue.svelte';

export let data;

function getComponent(type) {

if (type === 'string') return StringValue;

if (type === 'number') return NumberValue;

return JsonValue;

}

</script>

<ul>

{#each Object.entries(data) as [key, value]}

<li>

{key}:

{#if typeof value === 'object'}

<svelte:self data={value} />

{:else}

<svelte:component this={getComponent(typeof value)} value={value} />

{/if}

</li>

{/each}

</ul>

```js

 In the updated code snippet, we created a utility function named `getComponent` to determine which component to render, based on the type of the primitive value. We then used `<svelte:component this={getComponent(typeof value)} />` to dynamically load the appropriate component for each primitive type, allowing our `JsonTree` component to switch to different components for different types of data.
Lastly, to make our JSON tree viewer more versatile, we can introduce named slots to customize the appearance of keys and values. By doing so, users can easily tailor the look of these elements according to their needs, while keeping our current design as the default fallback. Let’s update the code to add the two named slots:

```

<ul>

{#each Object.entries(data) as [key, value]}

<li>

<slot name="obj-key" key={key}>{key}:<slot>

{#if typeof value === 'object'}

<svelte:self data={value} />

{:else}

<slot name="obj-value" value={value}>

<svelte:component this={getComponent(typeof value)} value={value} />

</slot>

{/if}

</li>

{/each}

</ul>

```js

 In the updated code, we added two named slots – one to customize keys, called `"obj-key"`, and another to customize values, called `"obj-value"`. These slots receive the current `key` and `value` as slot props, enabling you to tailor their appearance in the parent component.
For instance, if you wish to change how keys and values are displayed, here’s how you can write in your parent component:

```

<JsonTree data={{a: 1, b: 2}}>

<svelte:fragment slot="obj-key" let:key>

<em>{key}</em>

</svelte:fragment>

<svelte:fragment slot="obj-value" let:value>

<u>{value}</u>

</svelte:fragment>

</JsonTree>

```js

 Here, we use Svelte’s `<svelte:fragment>` special element to target the named slots, `"obj-key"` and `"obj-value"`, in our `JsonTree` component. The `let:key` and `let:value` syntax allows us to access the current key and value being rendered, respectively.
In the `<svelte:fragment>` element, we wrapped the key in an `<em>` tag and the value in a `<u>` tag, offering custom styling for these elements. This customization overrides the default rendering provided by the `JsonTree` component. If you don’t specify any custom content, the component will fall back to its default rendering, offering both robustness and flexibility.
By combining `<svelte:self>` for recursion, `<svelte:component>` for dynamic behavior, and slots for customization, we have created a flexible and powerful JSON tree viewer. This not only demonstrates Svelte’s capability to handle complex UI patterns but also serves as a practical example of component composition in Svelte.
You can find the complete code for this exercise here: [`github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter04/06-exercise-svelte-json-tree`](https://github.com/PacktPublishing/Real-World-Svelte/tree/main/Chapter04/06-exercise-svelte-json-tree%0D).
Before we wrap up this chapter, let us take a moment to discuss a helpful common pattern to organize our components, the Container/Presentational pattern.
The Container/Presentational pattern
As your application scales in complexity, you may find it beneficial to adopt specific design patterns or guidelines to structure components. One such approach is the **Container/Presentational pattern**, which divides a component into two categories, the Container component and the Presentational component:

*   *Container components* focus on functionality. They handle data fetching, state management, and user interactions. While they usually don’t render **Document Object Model** (**DOM**) elements themselves, they wrap around Presentational components and supply them with data and actions.
*   *Presentational components* are all about the user interface. They get their data and event-handling functions exclusively through props, making them highly reusable and straightforward to test.

A common scenario where you’ll see this pattern in action is when using a UI component library. In this case, the library’s components serve as the presentational elements, focusing solely on how things look. Meanwhile, the components you create that utilize these library elements act as Container components, managing data and handling interactions such as clicks, typing, and dragging.
The Component/Presentational pattern is useful in a few aspects:

*   **Simplicity**: Separating behavior from appearance makes your codebase easier to understand and maintain
*   **Reusability**: Since Presentational components are agnostic about data sources or user actions, you can reuse them in different parts of your application
*   **Collaboration**: With this division, designers can work on the Presentational components while developers focus on the Container components, streamlining development

As you come to appreciate the benefits of the Component/Presentational pattern, there are specific situations where I think you should consider using it, such as the following:

*   **When your application starts to grow**: Managing everything in a single component can become confusing as complexity increases
*   **When you find yourself repeating the same UI patterns**: Creating reusable Presentational components can save time in the long run
*   **When your team scales**: As your development team grows, having a standardized way of building components can reduce learning curves and prevent code conflicts

While the Container/Presentational pattern offers a structured approach to organizing your components, it’s not always the best fit – especially for smaller applications, where it might be overkill. Hopefully, with the insights provided, you’ll be better equipped to make informed decisions.
Summary
In this chapter, we delved into various strategies for component composition in Svelte, each offering its own set of advantages and applicable scenarios. Mastering these techniques will equip you to build Svelte applications that are not only more dynamic but also easier to maintain and scale.
We kicked off by discussing multiple ways to influence a child component. These ranged from controlling props or using Svelte’s context to customizing styles via CSS custom properties, and even dynamically passing content through slots.
Then, we turned our attention to some of Svelte’s special elements. We explored `<svelte:element>` and `<svelte:component>` to dynamically render various HTML elements and Svelte components. We also learned about `<svelte:self>`, which allows a component to reference itself, thereby facilitating the creation of recursive UI structures. We then applied these newfound skills to build a JSON tree viewer as an illustrative example.
Finally, we touched upon a popular design pattern – the Container/Presentational pattern. We examined its advantages and considered scenarios where it would be beneficial to employ this approach.
Armed with these advanced techniques, you are now better prepared to tackle complex challenges in your Svelte projects. As we conclude our four-chapter exploration of writing Svelte components, we’ll shift our focus in the upcoming chapter to another fundamental feature of Svelte – Svelte actions.

```

# 第二部分：动作

在本部分，我们将开始学习 Svelte 动作。在三个章节中，我们将深入研究 Svelte 动作的三个不同用例。从使用动作创建自定义事件开始，我们将逐步过渡到通过动作将第三方 JavaScript 库与 Svelte 整合。最后，我们将揭示利用 Svelte 动作的力量来渐进式增强我们应用程序的技术。

本部分包含以下章节：

+   *第五章*, *使用动作创建自定义事件*

+   *第六章*, *使用动作整合库*

+   *第七章*, *使用 Svelte 动作进行渐进式增强*
