# 第七章：掌握定义类型艺术

在本章中，我们将看到如何从我们未直接工作但导入到 TypeScript 项目中的库中创建类型。主要区别在于，当我们消费项目外部的代码时，我们不会直接使用 TypeScript 代码，而是使用其定义。原因是那些库中提供的是 JavaScript，而不是 TypeScript 代码。我们将看到如何掌握为不提供它们的代码创建定义文件的艺术，使我们能够在强环境中继续工作。

本章涵盖以下内容：

+   如何使用第三方库定义文件

+   TypeScript 如何生成定义文件

+   如何手动为 JavaScript 项目添加定义文件

+   如何将类型合并到现有的定义文件中

+   如何为 JavaScript 项目创建定义文件

+   不需要强类型但想使用 JavaScript 库

+   需要使用另一个模块

+   如何将定义文件添加到现有模块的扩展中

# 如何使用第三方库定义文件

当 TypeScript 知道每个变量和函数的类型时，它运行得很好。然而，当使用用 JavaScript 编写的第三方库时，您没有定义文件。TypeScript 很智能，它通过利用标准文档 JSDoc 尽可能多地推断类型，但没有任何东西能比得上用 TypeScript 规则编写的签名。然而，有许多用 JavaScript 编写的有用库没有 TypeScript 的类型。定义文件填补了 JavaScript 和 TypeScript 之间的差距。对于第三方库，想法是使用定义文件。如果原始库是用 JavaScript 编写的，定义文件源可以来自手动编辑；如果用 TypeScript 编写，则可以由 TypeScript 自动生成。

要使用第三方库定义文件，您需要将文件放在我们的项目中。TypeScript 定义文件具有 .d.ts 扩展名。TypeScript 将在 `node_modules` 文件夹以及我们的项目中搜索定义文件。因为 TypeScript 使用 `node_modules`，这意味着它可以从 NPM 获取定义文件。TypeScript 拥有最活跃的 GitHub 仓库之一，该仓库有超过 4,200 个由社区支持的定义文件。它们都可以通过 NPM 下的 `@types` 访问。以下是如何获取 JQuery 定义文件的示例：

```js
npm install @types/jquery --save-dev
```

TypeScript 日益增长的流行度使得许多库直接将定义文件集成到它们的主 npm 包中。例如，Redux 在主`npm`包的根目录下有`index.d.ts`。这意味着您可能已经拥有了定义文件而没有注意到。库将定义文件直接带到 NPM 包中的原因是类型的版本始终与代码同步。这也使使用 JavaScript 并使用能够读取定义文件的代码编辑器的人受益。一些代码编辑器可以利用定义文件提供自动完成功能。

除了`node_modules`之外，TypeScript 读取`tsconfig.json`文件中的`types`和`typeroots`配置。有关更多详细信息，请参阅第一章，*TypeScript 入门*。

如果缺少第三方定义文件，您可以创建一个；创建一个类型，将主要导出设置为`any`，这将移除类型安全但能够访问任何内容。还有将新定义合并到现有第三方库中以提高其功能的选项。我们将在本章中介绍这一领域。

# TypeScript 如何生成定义文件

即使代码是用 TypeScript 编写的，当需要与世界分享时，也只发布 JavaScript 文件。这样做的原因是让所有人，包括 JavaScript 和 TypeScript 开发者，都能使用您的代码。最好以只提供类型能力的格式发布 TypeScript 的类型，而不是使用完整的 TypeScript 代码。另一方面，TypeScript 可以生成允许浏览器完美解释代码的 JavaScript 文件。TypeScript 生成两种类型的文件，即定义文件和 JavaScript 文件，这为开发者和浏览器打开了兼容性。虽然定义文件可以手动制作，这对于想要提供 TypeScript 支持的 JavaScript 库来说很方便，但自动生成它既快又少出错。也就是说，TypeScript 是生成定义的最佳选择，因为它存在于.ts 文件中。这就是为什么 TypeScript 有一个在编译时生成定义的编译选项，称为`declaration`，其路径可以通过另一个选项`declarationDir`来控制。这两个选项已经在第一章，*TypeScript 入门*中讨论过。以下是允许从 TypeScript 编译生成`definition`文件的行：

```js
"declaration": true
```

# 如何手动为 JavaScript 项目添加定义文件

许多项目是用 JavaScript 编写的，但仍然希望利用 TypeScript 的类型优势。或者，一些 JavaScript 项目生成 TypeScript 的定义文件，以便在代码编辑器中获得良好的支持。最后，一些位于 JavaScript 库主仓库之外的人为每个 TypeScript 用户开发手动定义文件，以便使用该库。

要从一个不属于你的项目中创建一个定义文件，你需要创建一个以你想要添加类型的模块命名的文件夹，并添加一个 index.d.ts 文件。然而，如果你拥有这个库，你可以将 `types` 或 `typing`（它们是同义词）设置为 `definition` 文件的路径和文件名。在下面的代码示例中，定义文件被设置为 `lib` 文件夹下的 `main.d.ts`。如果没有提供 `types` 或 `typing`，定义文件必须在包文件夹的根目录下命名为 `index.d.ts`。

使用 `index.d.ts` 是最佳实践，因为 TypeScript 在进行模块解析时已经优化了搜索 `index.d.ts`，以及具有与模块名称相同的文件名（后跟 `.d.ts`）：

```js
{
"name": "your-library",
"main": "./lib/main.js",
"types": "./lib/main.d.ts"
}
```

与 `library` 一样，所有依赖项都必须指定。这次，所有定义文件库都必须在 `package.json` 中提及。重要的是要注意，我们不是在提及 `dev` 依赖项中的定义文件，因为我们希望所有类型都由我们的 `definition` 文件库的消费者下载和安装。

# 如何将类型合并到现有的定义文件中

类型可以写在不同地方，并合并成 TypeScript 可以依赖的单一定义集。原则是，你可能能够通过自己的定义扩展现有定义。合并能力在你有可以增强的 JavaScript 代码（通过插件或扩展）时非常有用。例如，Redux 库在其仓库和 NPM 包中都有自己的定义文件。名为 `Redux-thunk` 的库也有自己的定义文件，它为 Redux 添加了一个新的 `dispatch` 函数签名，覆盖了 `redux` 中定义的签名。定义文件依赖于合并类型来将其自己的 `dispatch` 定义添加到 `redux` 模块中：

```js
declare module "redux" {
   export interface Dispatch<S> {
   <R, E>(asyncAction: ThunkAction<R, S, E>): R;
 }
}
```

合并类型需要了解 TypeScript 允许的方式。第一条规则是，所有命名空间可以在一个或多个文件中定义多次。这意味着你可以在多个命名空间作用域内定义代码，而 TypeScript 会将其视为同一个命名空间中的代码。

命名空间的内容只有在标记为导出元素时才会共享：

```js
namespace Merge {
 export interface I1 { m1: string; }
}

namespace Merge {
 export interface I2 { m2: string; }
}
```

这可以写在一个命名空间中：

```js
namespace Merge {
   export interface I1 { m1: string; }
   export interface I2 { m2: string; }
}
```

同样，接口也可以合并：

```js
interface Mergeable {
 m1: string;
}

interface Mergeable {
 m2: string;
}
const mergeInterface: Mergeable = { m1: "", m2: "" }
```

然而，`type` 并不作为一个接口，并且不允许合并。

一个类可以通过具有相同名称的接口来增强其定义。这意味着你可以定义一个与具体类（在 JavaScript 中）具有相同名称的接口，并能够定义一个强类型定义。这也意味着如果需要，你可以在定义中提供类的扩展成员：

```js
export interface Album { m1: string; m2: number; }
export class Album {
public m2: number = 12;
}
const a = new Album();
a.m1; // Not implemented but compile.
a.m2;
```

可以使用命名空间来定义一个函数变量。在 JavaScript 中，可以通过使用函数名和点符号来将变量赋给一个函数。要定义此函数的类型，需要指定不仅参数名和返回类型，还要指定变量。这可以通过使用函数名定义一个命名空间来实现：

```js
function functionInJavaScript(param: string): string {
  return functionInJavaScript.variableOfFunction + param;
}
namespace functionInJavaScript {
  export let variableOfFunction = "";
}
```

在`全局`作用域中可以声明一个`接口`函数：

```js
declare global {
   interface Array<T> {
       toObservable(): Observable<T>;
   }
}
```

# 为 JavaScript 项目创建定义文件

当前的开源世界降低了示例的门槛。TypeScript 拥有最活跃的仓库之一，这是所有那些在主仓库中没有定义文件的第三方库的类型。快速查看几个库会显示出在编写定义文件方面的碎片化。这是由于大量不同的库结构。JavaScript 有全局的、模块化的、UMD、插件和全局修改的。

# 全局结构库的定义文件

全局库的典范是带有流行美元符号的 JQuery。全局库将它们的函数和变量添加到 window 作用域。这可以通过使用`window`或通过定义一个变量来显式完成。它不使用任何导入、导出或`require`函数。

要为全局结构库创建一个定义文件，你可以使用许多 TypeScript 关键字来定义一个类型。在函数的情况下，你可以使用`declare`前缀的`function`，并像在接口中那样编写函数签名，但不包含函数体：

```js
declare function myGlobalFunction(p1: string): string;
```

关键字`declare`的存在是为了表明函数存在于其他地方：

```js
Let var1:string = myGlobalFunction(“test”);
```

如果你全局作用域中声明的是一个类型而不是函数，你可以使用一个接口。关键字`declare`被省略了：

```js
interface myGlobalType{
   name: string;
}
```

全局接口允许声明一个全局类型的变量，无需在类型前添加任何前缀：

```js
Let var1: myGlobalType={name:”test”};
```

全局接口允许在一系列连贯元素中指定一个类型。它通常代表一个函数作用域：

```js
declare namespace myScope{
  let var1: number;
  class MyClass{
  }
}

let x: number = myScope.var1;
let y: myScope.MyClass = new myScope.MyClass();
```

命名空间可以包括一个接口来定义一个对象，一个类型来定义特定类型的变量，以及一个`function`来定义函数：

```js
declare namespace myScope{
 interface MyObject{
    x: number;
 }

 type data = string;
 function myFunction():void{};
}
```

对象、变量和函数的使用总是使用命名空间名称，因为它通过变量名在全局范围内暴露：

```js
let s: myScope.MyObject = { x: 5 };
let x: myScope.data = “test”;
myScope.myFunction();
```

在本节中，我们看到了如何定义一个全局库，它可以有一个全局函数或变量，也可以有一个全局变量，它可以包含一个对象、一个函数的原始类型。

# 模块库的定义文件

库的 `definition` 文件类似但也有一些不同。如果你需要提供定义文件，建议使用以下规则命名一个 `index.d.ts` 文件。首先，有一个可选的导出声明，如果库支持 UMD，则需要这个声明。这发生在库导出一个变量时。以下代码示例中暴露的变量是 `myScope`，整个模块都驻留在其中：

```js
export as namespace myScope;
```

下一步是将每个函数直接添加到定义文件中。没有必要将函数包含在命名空间中。对于对象也是同样的：

```js
export function myFunction(): void;
export interface MyObject{
 x: number;
}
export let data: string;
```

函数、接口和变量在真实代码中的使用方式如下：

```js
import {myFunction, MyObject, data} from “myScope”;
myFunction();
x:MyObject = {x:1};
console.log(data);
```

剩余的功能是在你的模块中有一个对象：

```js
export namespace myProperty{
   export function myFunction2(): void;
}
```

这里是命名空间的实际使用方式，它像函数、对象和数据一样，有两种格式。第一种是明确调用模块中要检索的元素，第二种是使用星号，它将整个内容定义导入到别名中：

```js
Import {myProperty} from “myScope”;
myProperty.myFunction2();
//or
Import * from my from “myScope”
my.myProperty.myFunction2();
```

然而，在某些情况下，这不会起作用。这取决于 JavaScript 模块的编写方式。以下代码适用于现代模块创建。它使用带有模块的声明语句。模块名称必须是引号中的库名称。在模块内部，你可以使用 `export =` 定义你的 `CommonJs/Amd` 导出，然后跟随着你想要默认导出的内容。在以下代码中，类 `MessageFormat` 是默认导出。你也可以不进行 `CommonJs/Amd` 导出，并导出每一个类型。你也可以导出一个包含许多类型的命名空间：

```js
declare module "modulenamehere" {
  type Msg = (params: {}) => string;
  type SrcMessage = string | SrcObject;
  interface SrcObject {
  m1: SrcMessage;
}

class MessageFormat {
  constructor(message: string);
  constructor();
  compile: (messages: SrcMessage, locale?: string) => Msg;
}

export = MessageFormat ; // CommonJs/AMD export syntax
}

// Usage:

import MessageFormat from "modulenamehere";
const mf = new MessageFormat("en");
const thing = mf.compile("blarb");
```

# 没有定义文件的 JavaScript 库

如果你需要使用没有定义文件的第三方库，你可以从一个单行声明开始。你需要创建一个具有模块名称的文件，并添加这个单行：

```js
declare module "*";
```

这不会给你任何智能提示、自动完成，但文件可以在你的 TypeScript 文件中无任何问题地导入。你可以从这种单行方法开始，然后逐渐过渡到一个更详细的定义。

# 使用定义文件中的另一个模块

你可能需要从你的定义文件中消费另一个模块。原因可能是为了从另一个库中获取类型。为了能够使用另一个库的类型，你可以使用 `import *` 并从你想要引用的类型中用 `as` 分配一个别名到你的 `definition` 文件中：

```js
declare module "react-summernote" {
  import * as React from "react";
  let ReactSummernote: React.ComponentClass<any>;
  export default ReactSummernote;
}
```

# 将定义文件添加到现有模块的扩展中

想法是使用 `declare` 和模块名称来扩展。一个系统可以有多个相同模块的声明，允许添加导出的 `type`、`function`、`interface` 或 `class`。以下代码还展示了另一个模块的使用，该模块被模块的扩展所使用：

```js
import * as extendMe from "moduleToExtend";
import * as other from "anotherModule";

declare module "moduleToExtend" {
export function theNewMethod(x: extendMe.aTypeInsideModuleToExtend): other.anotherTypeFromAnotherModule;
export interface ExistingInterfaceFromModuleToExtend {
newMember: string;
}

export interface NewTypeForModuleToExtend {
size: number;
}
}
```

# 摘要

在本章中，我们介绍了如何使用定义文件。我们解释了如何使用定义文件的多方面内容，以及许多细节，以便根据 JavaScript 代码的编写方式来促进定义的创建。

在这本书中，我们总结了开始使用 TypeScript 所必需的一切。这本快速入门指南的目标是为一场美味的餐宴做好准备，你可以用 TypeScript 来准备这场餐宴。书中讨论了如何从基本概念（原始类型）开始编码，到更高级的概念（泛型）。希望你能像我一样，对使用接近 JavaScript 但更强大（在可维护性方面）且更容易阅读的强类型语言感到高兴。感谢类型和 TypeScript，Web 开发变得更加安全、高效和愉快。
