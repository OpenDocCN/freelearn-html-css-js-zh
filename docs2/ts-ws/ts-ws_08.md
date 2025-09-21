# 第八章：7. 装饰器

概述

本章首先确立了装饰器的动机，然后描述了 TypeScript 中可用的各种装饰器类型。我们将探讨装饰器的使用方法和如何根据特定需求进行定制。我们还将涵盖编写自己的装饰器。到本章结束时，你将能够使用装饰器来改变代码的行为，并使用装饰器工厂来定制正在使用的装饰器。你还将学习如何创建自己的装饰器，以便在代码中使用或供他人使用。

# 简介

在前面的章节中，你看到了如何创建类型和类，以及如何使用接口、继承和组合将它们组合成一个合适的类层次结构。

使用 TypeScript 类型系统，你可以创建一些非常优雅的应用程序领域模型。然而，模型并非孤立存在；它们是更大图景的一部分——它们是应用程序的一部分。类需要意识到它们生活在一个更大的世界中，与许多其他系统部分并行运行，并且关注点超出了特定类的范围。

为了处理上述场景，向类添加行为或修改类并不总是容易。这正是装饰器大显身手的地方。装饰器是一些特殊的声明，可以添加到类声明、方法和参数中。

在本章中，我们将学习如何使用一种称为**装饰器**的技术，透明地给类添加复杂且常见的功能，而不会让你的应用程序逻辑因为额外的代码而变得混乱。

装饰器是 TypeScript 中可用且广泛使用的功能之一，但在 JavaScript 中不可用。JavaScript 中有一个装饰器的提案([`github.com/tc39/proposal-decorators`](https://github.com/tc39/proposal-decorators))，但它还不是标准的一部分。你将在 TypeScript 中使用的装饰器与该提案非常相似，功能上几乎一样。

TypeScript 方法有其优点和缺点。一个优点是，一旦装饰器成为 JavaScript 的标准功能，你就可以无缝地将你的装饰技能转移到 JavaScript 上，TypeScript 编译器（`tsc`）生成的代码将更加符合 JavaScript 习惯用法。坏处是，直到它成为标准功能，提案可能会改变。这就是为什么默认情况下，编译器中关闭了装饰器的使用，并且为了使用它们，你需要传递一个标志，无论是作为命令行选项还是作为 `tsconfig.json` 的一部分。然而，在你深入了解如何实现之前，你首先需要理解反射的概念，这将在下一节中探讨。

# 反射

装饰代码的概念与一个称为**反射**的概念紧密相连。简而言之，反射是指某段代码能够检查和反思自身的能力——在某种意义上，进行自我审视。这意味着一段代码可以访问其内部定义的变量、函数和类。大多数语言都为我们提供了一些反射 API，使我们能够将代码本身视为数据，由于 TypeScript 建立在 JavaScript 之上，因此它继承了 JavaScript 的反射能力。

JavaScript 没有广泛的反射 API，但有一个提议（[`tc39.es/ecma262/#sec-reflection`](https://tc39.es/ecma262/#sec-reflection)）要将适当的元数据（关于数据的数据）支持添加到语言中。

## 设置编译器选项

TypeScript 的装饰器使用了上述提议的功能，为了使用它们，你必须相应地启用 TypeScript 编译器（`tsc`）。如前言所述，有两种方法可以做到这一点。你可以在调用`tsc`时在命令行上添加必要的标志，或者你可以在`tsconfig.json`文件中配置必要的选项。

有两个与装饰器相关的标志。第一个是`experimentalDecorators`，这是使用装饰器所必需的。如果你有一个使用装饰器的文件，并且尝试在不指定它的情况下编译，你会得到以下错误：

```js
tsc --target es2015 .\decorator-example.ts
decorator-example.ts:18:5 – error TS1219: 
  Experimental support for decorators is a feature 
  that is subject to change in a future release. 
  Set the 'experimentalDecorators' option in your 'tsconfig' or 
  'jsconfig' to remove this warning.
```

如果你指定了标志，你可以成功编译：

```js
tsc --experimentalDecorators --target es2015 
  .\decorator-example.ts
```

为了避免每次都指定标志，请在`tsconfig.json`文件中添加以下标志：

```js
{
  "compilerOptions": {
    "target": "ES2015",
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
  }
}
```

注意

在你开始执行示例、练习和活动之前，我们建议你确保前面的编译器选项已经在你的`tsconfig.json`文件中启用。或者，你可以使用这里提供的文件：[`packt.link/hoeVy`](https://packt.link/hoeVy)。

# 装饰器的重要性

因此，现在你已经准备好开始装饰了。但为什么你想这样做呢？让我们通过一个简单的例子来模拟你将在以后遇到的真实场景。假设你正在构建一个简单的类，该类将封装篮球比赛的得分：

```js
Example_Basketball.ts
1  interface Team {
2      score: number;
3      name: string;
4  }
5  
6  class BasketBallGame {
7      private team1: Team;
8      private team2: Team;
9  
10     constructor(teamName1: string, teamName2: string) {
11         this.team1 = { score: 0, name: teamName1 };
12         this.team2 = { score: 0, name: teamName2 };
13     }
14  
15     getScore() {
16         return `${this.team1.score}:${this.team2.score}`;
17     }
18 }
19 
20 const game = new BasketBallGame("LA Lakers", "Boston Celtics");
Link to the preceding example: https://packt.link/ORdNl.
```

我们的类有两个团队，每个团队都有一个名称和数值得分。你在类构造函数中初始化你的团队，并且有一个提供当前得分的方法。然而，你没有更新得分的方法。让我们添加一个：

```js
updateScore(byPoints: number, updateTeam1: boolean) {
    if (updateTeam1) {
        this.team1.score += byPoints;
    } else {
        this.team2.score += byPoints;
    }
}
```

此方法接受要添加的点数和一个布尔值。如果布尔值为`true`，你正在更新第一支球队的得分，如果为`false`，你正在更新第二支球队的得分。你可以尝试运行你的类，如下所示：

```js
const game = new BasketBallGame("LA Lakers", "Boston Celtics");
game.updateScore(3, true);
game.updateScore(2, false);
game.updateScore(2, true);
game.updateScore(2, false);
game.updateScore(2, false);
game.updateScore(2, true);
game.updateScore(2, false);
console.log(game.getScore());
```

这段代码将向我们展示湖人队在对阵凯尔特人的比赛中以`7:8`失利（如果有人想知道，这是 2010 年总决赛的第 7 场比赛）。

## 横切关注点问题

到目前为止，一切顺利，你的类在自身功能方面完全可用。然而，由于你的类将存在于整个应用程序中，你还有其他一些担忧。其中之一就是授权——是否任何人都能够更新分数？当然不是，因为常见的用例是只有一个被允许更新分数的人，而可能有成千上万的人只是观看分数的变化。

让我们使用一个假设的函数`isAuthorized`来添加这个担忧，该函数将检查当前用户是否实际上被授权更改分数。你将调用这个函数，如果它返回`true`，我们将继续执行方法的常规逻辑。如果返回`false`，那么我们只需发出适当的消息。代码将如下所示：

```js
updateScore(byPoints: number, updateTeam1: boolean) {
    if (isAuthorized()) {
        if (updateTeam1) {
            this.team1.score += byPoints;
        } else {
            this.team2.score += byPoints;
        }
    } else {
        console.log("You're not authorized to change the score");
    }
}
```

再次强调，这将很好地工作，尽管它将你的方法代码行数从五行增加到九行，并增加了一些复杂性。坦白说，增加的行与计分无关，但它们必须被添加以支持授权。

那么，这就结束了？当然不是。即使你知道某人被授权，这也并不意味着你的操作员可以随时更新分数。审计员需要详细的信息，包括何时以及使用什么参数调用`updateScore`方法。没问题，我们可以使用一个假设的函数`audit`来添加这些信息。你还需要添加一些验证，以确保`byPoints`参数是一个合法的值（在篮球中，你只能有 1 分、2 分或 3 分的加成）。你还可以添加一些代码来记录方法执行的性能，以便追踪执行所需的时间。因此，你那清晰、简洁的五行方法将变成一个 17 行的怪物：

```js
updateScore(byPoints: number, updateTeam1: boolean) {
    audit("updateScore", byPoints, updateTeam1);
    const start = Date.now();
    if (isAuthorized()) {
        if (validatePoints(byPoints)) {
            if (updateTeam1) {
                this.team1.score += byPoints;
            } else {
                this.team2.score += byPoints;
            }
        } else {
            console.log(`Invalid point value ${byPoints}`);
        }
    } else {
        console.log("You're not authorized to change the score");
    }
    const end = Date.now();
    logDuration("updateScore", start, end);
}
```

在所有这些复杂性中，你仍然有你简单、清晰的逻辑，如果布尔值为`true`，则更新湖人队的分数，如果为`false`，则更新凯尔特人队的分数。

这里重要的是，增加的复杂性并非来自你的特定商业模式——篮球比赛本身仍然有效。所有增加的功能都源于类所在的系统。篮球比赛本身不需要授权、性能指标或审计。但记分牌应用程序确实需要所有这些以及更多。

注意，所有添加的逻辑都已经封装在方法（`audit`、`isAuthorized`、`logDuration`）中，而实际执行上述所有操作的代码位于你的方法之外。你插入到函数中的代码只做了最基本的工作——然而，它仍然使你的代码变得复杂。

此外，授权、性能指标和审计将在你的应用程序的许多地方需要，而在这些地方，代码对授权、测量或审计的实际代码的工作并不起作用。

### 解决方案

让我们更仔细地看看上一节中的一个关注点，即性能指标，也就是持续时间测量。这对应用程序非常重要，要将它添加到任何特定的方法中，你需要在方法的开头和结尾添加几行代码：

```js
const start = Date.now();
// actual code of the method
const end = Date.now();
logDuration("updateScore", start, end);
```

我们需要将其添加到你需要测量的每个方法中。这是一段非常重复的代码，每次你编写它时，你都在打开做错的可能性。此外，如果你需要更改它，即向`logDuration`方法添加参数，你将需要更改数百甚至数千个调用点。

为了避免这种风险，你可以做的是将方法的实际代码包裹在另一个函数中，该函数仍然会调用它。这个函数可能看起来像这样：

```js
function wrapWithDuration(method: Function) {
    const result = {
        [method.name]: function (this: any, ...args: any[]) {
            const start = Date.now();
            const result = method.apply(this, args);
            const end = Date.now();
            logDuration(method.name, start, end);
            return result;
        },
    };
    return result[method.name];
}
```

`wrapWithDuration`函数（其细节你现在可以忽略）将接受一个方法并返回一个具有以下功能的函数：

+   相同的`this`引用

+   相同的方法名

+   相同的签名（参数和返回类型）

+   原始方法所具有的所有行为

+   扩展行为，因为它将测量实际方法的持续时间

由于它实际上会调用原始方法，从外部看，新函数与原始函数完全无法区分。你在保持所有已有内容的同时添加了一些行为。现在，你可以用新的改进版本替换原始方法。

采用这种方法，你将得到的是：原始方法不会知道或关心应用程序的横切关注点，而是专注于自己的业务逻辑——应用程序可以在运行时用具有所有必要业务逻辑以及所有所需添加的函数“升级”该方法。

这种透明的“升级”通常被称为**装饰**，而执行装饰的方法被称为**装饰器方法**。

这里所展示的只是装饰可以采取的一种形式。解决方案的数量可以与开发者的数量一样多，而且它们都不会简单直接。应该制定一些标准，TypeScript 设计团队决定采用提议的 JavaScript 语法。

本章的其余部分将使用该语法，你可以忽略这里给出的解决方案。

# 装饰器和装饰器工厂

如我们所见，装饰器只是特殊的包装函数，它们为你的常规方法、类和属性添加行为。它们特殊之处在于在 TypeScript 中的使用方式。TypeScript 支持以下装饰器类型：

+   **类装饰器**：这些装饰器附加到类声明上。

+   **方法装饰器**：这些装饰器附加到方法声明上。

+   **访问器装饰器**：这些装饰器附加到属性的访问器声明上。

+   **属性装饰器**：这些装饰器附加到属性本身。

+   **参数装饰器**：这些装饰器附加到方法声明中的单个参数上。

因此，你有五个不同的地方可以使用装饰器，这意味着有五种不同类型的特殊函数可以用来装饰你的代码。所有这些都在下面的示例中展示：

```js
@ClassDecorator
class SampleClass {
    @PropertyDecorator
    public sampleProperty:number = 0;
    private _sampleField: number = 0;
    @AccessorDecorator
    public get sampleField() { return this._sampleField; }
    @MethodDecorator
    public sampleMethod(@ParameterDecorator paramName: string) {}
}
```

样本装饰器是按照以下方式定义的函数：

```js
function ClassDecorator (constructor: Function) {}
function AccessorDecorator (target: any, propertyName: string, descriptor: PropertyDescriptor) {}
function MethodDecorator (target: any, propertyName: string, descriptor: PropertyDescriptor) {}
function PropertyDecorator (target: any, propertyName: string) {}
function ParameterDecorator (target: any, propertyName: string, parameterIndex: number) {}
```

## 装饰器语法

向项目添加装饰器的语法是使用特殊的符号 `@` 后跟装饰器的名称。装饰器放置在它装饰的代码之前，所以在前面的示例中，你执行了以下装饰：

+   `@ClassDecorator` 紧接在 `SampleClass` 类之前，是一个类装饰器。

+   `@PropertyDecorator` 紧接在 `public sampleProperty` 之前，是一个属性装饰器。

+   `@AccessorDecorator` 紧接在 `public get sampleField()` 之前，是一个 `get` 访问器装饰器。

+   `@MethodDecorator` 紧接在 `public sampleMethod()` 之前，是一个方法装饰器。

+   `@ParameterDecorator` 紧接在 `paramName: string` 之前，是一个参数装饰器。

虽然装饰器本身是常规函数，但按照惯例，它们的名称使用 `PascalCase` 而不是 `lowerCamelCase`。

注意

想了解更多关于 `PascalCase` 和 `lowerCamelCase` 的信息，请访问 [`techterms.com/definition/camelcase`](https://techterms.com/definition/camelcase) 和 [`techterms.com/definition/pascalcase`](https://techterms.com/definition/pascalcase)。

## 装饰器工厂

你可以看到，在前面的示例中，你没有为样本装饰器集合指定任何参数，然而装饰器函数接受一个到三个参数。这些参数由 TypeScript 本身处理，并在代码运行时自动提供。这意味着你无法直接配置装饰器，例如，通过传递额外的参数。

幸运的是，你可以使用一个称为 `@` 符号的构造来指定装饰器，它将评估随后的表达式。因此，你不需要提供符合特殊装饰器要求的函数名称，而是可以提供一个将评估为该函数的表达式。换句话说，装饰器工厂仅仅是返回装饰器函数的高阶函数。

例如，让我们创建一个简单的函数，它将接受一个消息作为参数并将消息记录到控制台。该函数的返回值，其输入参数不符合类装饰器签名，将是一个函数，其输入参数符合类装饰器签名。这个生成的函数也将简单地记录消息到控制台。考虑以下代码：

```js
Example_Decorator_Factory.ts
1 function ClassDecoratorFactory(message: string) {
2     console.log(`${message} inside factory`);
3     return function (constructor: Function) {
4       console.log(`${message} inside decorator`);
5     };
6 }
Link to the preceding example: https://packt.link/M2Ixp.
```

从本质上讲，`ClassDecoratorFactory`函数本身不是一个装饰器，但它的返回值是。这意味着你不能直接将`ClassDecoratorFactory`用作装饰器，但如果你调用它，例如`ClassDecoratorFactory("Hi")`，那么这个值确实是一个装饰器。你可以使用它用这种语法装饰几个类。以下示例将帮助你更好地理解这一点：

```js
@ClassDecoratorFactory("Hi")
class DecoratedOne {}
@ClassDecoratorFactory("Hello")
class DecoratedTwo {}
```

在这里，你不再使用之前的`@ClassDecorator`这样的表达式，而是使用`@ClassDecoratorFactory("hi")`或`@ClassDecoratorFactory("hello")`。由于`ClassDecoratorFactory`函数的执行结果是一个类装饰器，这是可行的，并且装饰器成功地装饰了代码。当你运行代码时，你会看到以下输出：

```js
Hi inside factory
Hi inside decorator
Hello inside factory
Hello inside decorator
```

注意，你将使用的大多数装饰器实际上都是装饰器工厂，因为装饰时添加参数非常有用。大多数资料甚至一些文档都不会区分这两个术语。

# 类装饰器

类装饰器是一个应用于整个类的装饰器函数。它可以用来观察、更改或完全替换类定义。当一个类装饰器被调用时，它接收一个参数——调用类的构造函数。

## 属性注入

属性注入是类装饰器常用的场景之一。例如，假设你正在构建一个将模拟学校的系统。你将有一个名为`Teacher`的类，它将具有属性并模拟教师的行为。这个类的构造函数将接受两个参数，即教师的`id`和教师的`name`。这个类将看起来是这样的：

```js
class Teacher {
    constructor (public id: number, public name: string) {}
    // other teacher specific code
}
```

假设我们构建了系统并且它正在运行。一切都很顺利，但过了一段时间，就需要更新它。

我们希望使用令牌实现一个访问控制系统。由于新系统与教学过程无关，最好在不更改班级本身代码的情况下添加它，这样你可以使用一个装饰器来完成这个任务，并且你的装饰器可以给`Teacher`类的原型注入一个额外的布尔属性。`Teacher`类可以按照以下方式更改：

```js
Example_PropertyInjection.ts
1 @Token
2 class Teacher {
3     // old teacher specific code
4 }
```

可以用以下方式定义`Token`装饰器：

```js
5 function Token (constructor: Function) {
6     constructor.prototype.token = true;
7 }
```

现在，考虑以下代码，它创建了类的实例并打印一条消息：

```js
8 const teacher = new Teacher(1, "John Smith");
9 console.log("Does the teacher have a token? ",teacher["token"]);
```

运行所有这些代码将在控制台给出以下结果：

```js
Does the teacher have a token? true
Link to the preceding example: https://packt.link/asjvA.
```

在注入场景中，您使用提供的 `constructor` 参数，但您的函数不返回任何内容。在这种情况下，类将继续像以前一样工作。通常，我们将使用构造函数的原型来向对象添加字段和属性。

注意

在本章的所有练习和活动中，在执行代码文件之前，您需要使用 `npm i` 在目标目录中安装所有依赖项。然后，您可以在目标目录中运行 `npx ts-node 'filename'` 来执行文件。

## 练习 7.01：创建简单的类装饰器工厂

在这个练习中，你将创建一个简单的 `Token` 装饰器工厂。从 `Teacher` 类的代码开始，我们将创建一个名为 `Student` 的类，该类需要使用 `Token` 装饰器进行装饰。我们将扩展装饰器以接受一个参数，并使用创建的装饰器工厂装饰这两个类。

以下步骤将帮助您找到解决方案：

注意

在开始之前，请确保您已设置正确的编译器选项，如 *设置编译器选项* 部分所述。此练习的代码文件也可以从 [`packt.link/UpdO9`](https://packt.link/UpdO9) 下载。此存储库包含两个文件：`school-token.start.ts` 和 `school-token.end.ts`。前者包含此练习的 *步骤 6* 之前的代码，后者包含练习的最终代码。

1.  打开 Visual Studio Code，在新的目录（`Exercise01`）中创建一个新文件，并将其保存为 `school-token.ts`。

1.  在 `school-token.ts` 中输入以下代码：

    ```js
    @Token
    class Teacher {
        constructor (public id: number, public name: string) {}
        // teacher specific code
    }
    function Token (constructor: Function) {
        constructor.prototype.token = true;
    }
    /////////////////////////
    const teacher = new Teacher(1, "John Smith");
    console.log("Does the teacher have a token? ",teacher["token"]);
    ```

1.  执行代码，并注意它在控制台输出 `true`。

1.  在文件末尾添加一个 `Student` 类：

    ```js
    class Student {
        constructor (public id: number, public name: string) {}
        // student specific code
    }
    ```

1.  添加代码以创建一个学生并尝试打印其 `token` 属性：

    ```js
    const student = new Student(101, "John Bender");
    console.log("Does the student have a token? ",student["token"]);
    ```

1.  执行代码，并注意它在控制台输出 `true` 和 `undefined`。

1.  将 `Token` 装饰器添加到 `Student` 类中：

    ```js
    @Token
    class Student {//… 
    ```

1.  执行代码，并注意它在控制台输出 `true` 两次。

1.  将 `Token` 函数更改为一个接受布尔参数的工厂函数：

    ```js
    function Token(hasToken: boolean) {
        return function (constructor: Function) {
            constructor.prototype.token = hasToken;
        }
    }
    ```

1.  修改 `Teacher` 类的 `Token` 装饰器以包含一个 `true` 布尔参数：

    ```js
    @Token(true)
    class Teacher {//…
    ```

1.  修改 `Student` 类的 `Token` 装饰器以包含一个 `false` 布尔参数：

    ```js
    @Token(false)
    class Student {//…
    ```

1.  通过在控制台运行 `npx ts-node` `school-token.ts` 来执行代码，并注意它按如下所示输出 `true` 和 `false`：

    ```js
    Does the teacher have a token?  true
    Does the student have a token?  false
    ```

在这个练习中，你看到了如何添加一个类装饰器，该装饰器向装饰过的类添加一个属性。然后，你将装饰器更改为使用工厂，并为两个装饰过的类添加了两个不同的参数。最后，你验证了注入的属性确实存在于装饰过的类中，并且通过原型链具有你指定的值。

## 构造函数扩展

使用属性注入，你能够通过它们的原型向你装饰的对象添加行为和数据。这是可以的，但有时你可能想直接向构造对象本身添加数据。你可以通过继承来实现这一点，但你也可以用装饰器包装继承。

如果你从装饰器返回一个函数，该函数将用作类的替换构造函数。虽然这给了你完全改变类的超级能力，但这种方法的主要目标是让你能够通过添加一些新的行为或数据来增强类。让我们使用自动继承来向类添加属性。一个将 `token` 属性添加到构造对象本身而不是原型的装饰器看起来像这样：

```js
type Constructable = {new(...args: any[]):{}};
function Token(hasToken: boolean) {
    return function <T extends Constructable>(constructor: T) {
        return class extends constructor {
            token: boolean = hasToken;
        }
    }
}
```

首次看到这样做时，语法可能看起来有点奇怪，因为你正在使用一个泛型参数来确保从你的装饰器返回的类仍然与传递给构造函数的构造函数兼容。除了语法之外，需要记住的重要部分是代码 `token: boolean = hasToken;` 将在常规构造函数之外执行。

## 练习 7.02：使用构造函数扩展装饰器

在这个练习中，你将创建一个用于 `Token` 装饰器的构造函数扩展装饰器工厂。从 `Teacher` 类代码开始，我们将添加一个名为 `Token` 的工厂，通过添加一个 `token` 布尔属性来增强类。我们将创建一个提供的类对象，并验证该对象确实有自己的 `token` 属性。以下步骤将帮助你找到解决方案：

注意

在开始之前，请确保你已经设置了如 *设置编译器选项* 部分所述的正确编译器选项。此练习的代码文件也可以从 [`packt.link/DhVfC`](https://packt.link/DhVfC) 下载。此存储库包含两个文件：`school-token.start.ts` 和 `school-token.end.ts`。前者包含此练习的 *步骤 3* 之前的代码，后者包含练习的最终代码。

1.  打开 Visual Studio Code，在新的目录（`Exercise02`）中创建一个新文件，并将其保存为 `school-token.ts`。

1.  在 `school-token.ts` 文件中输入以下代码：

    ```js
    class Teacher {
        constructor (public id: number, public name: string) {}
        // teacher specific code
    }
    /////////////////////////
    const teacher = new Teacher(1, "John Smith");
    console.log("Do you have a token:", teacher["token"]);
    console.log("Do you have a token property: ", teacher.hasOwnProperty("token"));
    ```

1.  执行代码，并注意它将输出 `undefined` 和 `false` 到控制台：

    ```js
    Do we have a token: undefined
    Do we have a token property:  false
    ```

1.  在文件末尾添加一个 `Token` 函数：

    ```js
    type Constructable = {new(...args: any[]):{}};
    function Token(hasToken: boolean) {
        return function <T extends Constructable>(constructor: T) {
            return class extends constructor {
                token: boolean = hasToken;
            }
        }
    }
    ```

1.  使用 `Token` 装饰器工厂装饰 `Teacher` 类：

    ```js
    @Token(true)
    class Teacher {
    ```

1.  执行代码，并注意它将输出 `true` 两次到控制台：

    ```js
    Do we have a token: true
    Do we have a token property:  true
    ```

在这个练习中，你看到了如何更改提供的类构造函数以在实例化对象时运行自定义代码。你使用了这一点来在构造对象本身上注入属性，然后验证注入的属性存在于装饰类的对象上，并且它们的值是你指定的。

## 构造函数包装

对于类装饰器来说，另一个常见的场景是在创建类的实例时只需运行一些代码，例如，在创建类的实例时添加一些日志记录。你不需要或想要以任何方式更改类的行为，但你确实希望能够以某种方式依赖这个过程。这意味着你需要在类构造器运行时执行一些代码——你不需要更改现有的构造器。

在这种情况下，解决方案是让装饰器函数返回一个新的构造器，该构造器执行装饰器本身所需的新代码以及原始构造器。例如，如果你想每次实例化装饰过的类时都向控制台写入一些文本，你可以使用这个装饰器：

```js
Example_ConstructorWrapping.ts
1  type Constructable = { new (...args: any[]): {} };
2  
3  function WrapConstructor(message: string) {
4      return function <T extends Constructable>(constructor: T) {
5          const wrappedConstructor: any = function (...args: any[]) {
6              console.log(`Decorating ${message}`);
7              const result = new constructor(...args);
8              console.log(`Decorated ${message}`);
9              return result;
10         };
11         wrappedConstructor.prototype = constructor.prototype;
12         return wrappedConstructor;
13    };
14  }
Link to the preceding example: https://packt.link/kgAme.
```

这个装饰器工厂将使用提供的信息生成一个装饰器。由于你返回了一个新的构造器，你必须使用一个泛型参数来确保从你的装饰器返回的构造器仍然与传递给参数的原始构造器兼容。你可以在`wrappedConstructor`函数内部创建一个新的函数，在其中你可以调用自定义代码（`Decorating`和`Decorated`消息），并通过在原始构造器上调用`new`来实际创建对象，传递原始参数。

你应该在这里注意以下内容：你可以在对象创建前后添加自定义代码。在前面的例子中，`Decorating`消息将在对象创建之前打印到控制台，而`Decorated`消息将在创建完成后打印到控制台。

另一个非常重要的事情是，这种包装会破坏原始对象的原型链。如果你装饰的对象使用了通过原型链可用的任何属性或方法，它们将丢失，改变装饰类的行为。由于这与你想通过构造器包装实现的目标正好相反，你需要重置链。这是通过将新创建的包装函数的`prototype`属性设置为原始构造器的原型来完成的。

因此，让我们在客户端类上使用装饰器，如下所示：

```js
@WrapConstructor("decorator")
class Teacher {
    constructor(public id: number, public name: string) {
        console.log("Constructing a teacher class instance");
    }
}
```

接下来，你可以创建一个`Teacher`类的对象：

```js
const teacher = new Teacher(1, "John");
```

当你运行文件时，你将在控制台看到以下内容：

```js
Decorating decorator
Constructing a teacher class instance
Decorated decorator
```

## 练习 7.03：为类创建一个日志装饰器

在这个练习中，你将创建一个用于`LogClass`装饰器的构造器包装装饰器工厂。从`Teacher`类的代码开始，你将添加一个名为`LogClass`的装饰器工厂，它将使用一些日志代码包装类构造器。你将创建提供的类的对象，并验证日志方法确实被调用。以下步骤将帮助你找到解决方案：

注意

在你开始之前，请确保你已经设置了如 *设置编译器选项* 部分所述的正确编译器选项。此练习的代码文件也可以从 [`packt.link/vBLMg`](https://packt.link/vBLMg) 下载。

1.  打开 Visual Studio Code，在新的目录（`Exercise03`）中创建一个新文件，并将其保存为 `teacher-logging.ts`。

1.  在 `teacher-logging.ts` 中输入以下代码：

    ```js
    class Teacher {
        constructor(public id: number, public name: string) {
            console.log("Constructing a teacher");
        }
    }
    /////////////////////////
    const teacher = new Teacher(1, "John Smith");
    ```

1.  执行代码，注意它会将 `Constructing a teacher` 输出到控制台。

1.  接下来，创建装饰器。首先，你需要添加 `Constructable` 类型定义：

    ```js
    type Constructable = { new (...args: any[]): {} };
    ```

1.  现在，添加你的装饰器工厂的定义：

    ```js
    function LogClass(message: string) {
        return function <T extends Constructable>(constructor: T) {
            return constructor;
        };
    }
    ```

    在前面的代码中，构造函数接受一个字符串参数并返回一个装饰器函数。装饰器函数本身最初只会返回被装饰类的原始、未更改的构造函数。

1.  使用带有适当消息参数的 `LogClass` 装饰器装饰 `Teacher` 类：

    ```js
    @LogClass("Teacher decorator")
    class Teacher {
        constructor(public id: number, public name: string) {
            console.log("Constructing a teacher");
        }
    }
    ```

1.  执行代码，注意行为没有任何变化。

1.  现在，向你的应用程序添加一个日志对象：

    ```js
    const logger = {
        info: (message: string) => {
            console.log(`[INFO]: ${message}`);
        },
    };
    ```

    在实际的生产级代码实现中，你可能需要将日志记录到数据库、文件、第三方服务等。在上一个步骤中，你只是将日志记录到控制台。

1.  接下来，使用 `logger` 对象向你的装饰器添加一个包装构造函数：

    ```js
        return function <T extends Constructable>(constructor: T) {
            const loggingConstructor: any = function(...args: any[]){
                logger.info(message);
                return new constructor(...args);
            }
            loggingConstructor.prototype = constructor.prototype;
            return loggingConstructor;
        };
    ```

1.  执行代码并验证你是否在控制台收到日志消息：

    ```js
    [INFO]: Teacher decorator
    Constructing a teacher
    ```

1.  构造几个更多的对象并验证每次创建对象时构造函数都会运行：

    ```js
    for (let index = 0; index < 10; index++) {
        const teacher = new Teacher(index +1, "LouAnne Johnson");
    }
    ```

    当你执行文件时，你会看到以下输出：

    ```js
    [INFO]: Teacher decorator
    Constructing a teacher
    [INFO]: Teacher decorator
    Constructing a teacher
    [INFO]: Teacher decorator
    Constructing a teacher
    [INFO]: Teacher decorator
    Constructing a teacher
    [INFO]: Teacher decorator
    Constructing a teacher
    [INFO]: Teacher decorator
    Constructing a teacher
    [INFO]: Teacher decorator
    Constructing a teacher
    ```

在这个练习中，你看到了如何包装提供的类构造函数，以便它可以运行自定义代码，但又不改变对象的构造。通过包装，你向一个没有任何日志功能的类添加了日志功能。你构造了这个类的对象，并验证了日志功能是可操作的。

# 方法和访问器装饰器

方法装饰器是一个应用于类单个方法的装饰器函数。在方法装饰器中，你可以观察、修改或完全用装饰器提供的定义替换方法定义。当方法装饰器被调用时，它接收三个参数：`target`、`propertyKey` 和 `descriptor`：

+   `target`: 由于方法可以是实例方法（在类的实例上定义）和静态方法（在类本身上定义），`target` 可以是两件不同的事情。对于实例方法，它是类的原型。对于静态方法，它是类的构造函数。通常，你将此参数键入为 `any`。

+   `propertyKey`: 这是你要装饰的方法的名称。

+   `descriptor`: 这是你要装饰的方法的属性描述符。`PropertyDescriptor` 接口定义如下：

    ```js
    interface PropertyDescriptor {
        configurable?: boolean;
        enumerable?: boolean;
        value?: any;
        writable?: boolean;
        get?(): any;
        set?(v: any): void;
    }
    ```

此接口定义了对象属性的值，以及属性的属性（属性是否可配置、可枚举和可写）。我们还将使用此接口的强类型版本，`TypedPropertyDescriptor`，其定义如下所示：

```js
interface TypedPropertyDescriptor<T> {
    enumerable?: boolean;
    configurable?: boolean;
    writable?: boolean;
    value?: T;
    get?: () => T;
    set?: (value: T) => void;
}
```

注意，在 JavaScript 和随后的 TypeScript 中，属性访问器只是管理属性访问的特殊方法。适用于装饰方法的一切也适用于装饰访问器。任何访问器特定的内容将单独介绍。

如果你在一个方法上设置了一个装饰器，我们将得到该方法的`PropertyDescriptor`实例，并且描述符的`value`属性将给我们访问其主体的权限。如果你在一个访问器上设置了一个装饰器，我们将得到相应属性的`PropertyDescriptor`实例，其`get`和`set`属性分别设置为获取器和设置器访问器。这意味着如果你正在装饰属性访问器，你不需要分别装饰获取器和设置器，因为对其中一个的任何装饰都是对另一个的装饰。实际上，如果你这样做，TypeScript 将发出以下错误：

```js
TS1207: Decorators cannot be applied to multiple get/set accessors of the same name.
```

方法装饰器不需要返回值，因为大多数情况下，你可以通过修改属性描述符来完成所需操作。然而，如果你返回一个值，那么这个值将替换最初提供的属性描述符。

## 实例函数上的装饰器

如前所述，任何接受`target`、`propertyKey`和`descriptor`参数的函数都可以用来装饰方法和属性访问器。所以，让我们有一个将简单地记录`target`、`propertyKey`和`descriptor`参数到控制台的功能：

```js
Example_Decorators_Instance_Functions.ts
1 function DecorateMethod(target: any, propertyName: string,
2 descriptor: PropertyDescriptor) {
3     console.log("Target is:", target);
4     console.log("Property name is:", propertyName);
5     console.log("Descriptor is:", descriptor);
6 }
Link to the preceding example: https://packt.link/gle5U.
```

你可以使用这个函数来装饰类的成员方法。这是一个极其简单的装饰器，但你可以用它来调查方法装饰器的使用情况。

让我们从简单的类开始：

```js
class Teacher {
    constructor (public name: string){}
    private _title: string = "";
    public get title() { 
        return this._title;
    }

    public set title(value: string) {
        this._title = value;
    }
    public teach() {
        console.log(`${this.name} is teaching`)
    }
}
```

这个类有一个构造函数，一个名为`teach`的方法，以及一个具有定义的获取器和设置器的`title`属性。访问器只是将控制权传递给`_title`私有字段。你可以使用以下代码将装饰器添加到`teach`方法上：

```js
    @DecorateMethod
    public teach() {
        // ....
```

当你运行你的代码（不需要实例化类）时，你将在控制台得到以下输出：

```js
    Target is: {}
    Property name is: teach
    Descriptor is: {
        value: [Function: teach],
        writable: true,
        enumerable: false,
        configurable: true
    }
```

考虑以下代码片段，其中你将装饰器应用于设置器或获取器（任何一个都可以正常工作，但不能同时使用）：

```js
    @DecorateMethod
    public get title() { 
        // ....
```

或者：

```js
    @DecorateMethod
    public set title(value: string) {
        // ....
```

当你使用上述任何一种建议运行代码时，你将得到以下输出：

```js
    Target is: {}
    Property name is: title
    Descriptor is: {
        get: [Function: get title],
        set: [Function: set title],
        enumerable: false,
        configurable: true
    }
```

注意，你无法在构造函数本身上添加方法装饰器，因为这会导致错误：

```js
 TS1206: Decorators are not valid here.
```

如果你需要更改构造函数的行为，你应该使用类装饰器。

## 练习 7.04：创建一个标记函数可枚举的装饰器

在这个练习中，你将创建一个装饰器，它将能够改变它所装饰的方法和访问器的`enumerable`状态。你将使用这个装饰器来设置你将要编写的类中一些函数的`enumerable`状态，最后，你将验证当你枚举对象实例的属性时，你也会得到修改后的方法。

注意

在你开始之前，请确保你已经设置了正确的编译器选项，如*设置编译器选项*部分所述。这个练习的代码文件也可以从[`packt.link/1nAff`](https://packt.link/1nAff)下载。这个存储库包含两个文件：`teacher-enumerating.start.ts`和`teacher-enumerating.end.ts`。前者包含这个练习的*步骤 5*之前的代码，后者包含练习的最终代码。

1.  打开 Visual Studio Code，在新的目录（`Exercise04`）中创建一个新文件，并将其保存为`teacher-enumerating.ts`。

1.  在`teacher-enumerating.ts`中输入以下代码：

    ```js
    class Teacher {
        constructor (public name: string){}
        private _title: string = "";
        public get title() { 
            return this._title;
        }

        public set title(value: string) {
            this._title = value;
        }
        public teach() {
            console.log(`${this.name} is teaching`)
        }
    }
    ```

1.  编写代码以实例化这个类的对象：

    ```js
    const teacher = new Teacher("John Smith");
    ```

1.  编写代码以枚举创建的对象中的所有键：

    ```js
    for (const key in teacher) {
        console.log(key);
    }
    ```

1.  执行文件并验证在控制台上显示的唯一键是`name`和`_title`。

1.  添加一个接受布尔参数的装饰器工厂，该工厂将生成一个设置提供的参数的`enumerable`状态的函数装饰器：

    ```js
    function Enumerable(value: boolean) {
        return function (target: any, propertyName: string, descriptor: PropertyDescriptor) {
            descriptor.enumerable = value;
        }
    };
    ```

1.  使用装饰器装饰`title`获取器或设置器访问器以及`teach`方法：

    ```js
        @Enumerable(true)
        public get title() { 
            return this._title;
        }

        public set title(value: string) {
            this._title = value;
        }
        @Enumerable(true)
        public teach() {
            console.log(`${this.name} is teaching`)
        }
    ```

1.  重新运行代码并验证`title`和`teach`属性是否被枚举：

    ```js
    name
    _title
    title
    teach
    ```

在这个练习中，你看到了如何添加一个方法装饰器工厂并将其应用于实例方法或实例属性访问器。你学习了如何使属性可枚举，并使用这些知识来设置类中函数的`enumerable`状态。最后，你枚举了一个类的所有属性。

## 静态函数上的装饰器

就像实例方法一样，装饰器也可以用于静态方法。你可以这样向你的`Teacher`类添加一个静态方法：

```js
Example_Decorator_StaticFunctions.ts
1 class Teacher {
2     //.....
3 
4     public static showUsage() {
5        console.log("This is the Teacher class")
6    }
7    //.....
Link to the preceding example https://packt.link/Ckuct.
```

我们同样允许在静态方法上使用方法装饰器。因此，你可以使用以下代码添加`DecorateMethod`装饰器：

```js
    @DecorateMethod
    public static showUsage() {
        //......
```

当你运行代码时，你将得到类似以下的输出：

```js
Target is: [Function: Teacher]
Property name is: showUsage
Descriptor is: {
  value: [Function: showUsage],
  writable: true,
  enumerable: false,
  configurable: true
}
```

与实例方法的主要区别是`target`参数。实例方法和访问器是在类原型上生成的，因此，当使用方法/访问器装饰器时，你会收到类原型作为`target`参数。静态方法和访问器是在类变量本身上生成的，因此，当使用方法/访问器装饰器时，你会收到类变量作为构造函数的化身作为`target`参数。

注意，这正是你作为类装饰器参数得到的确切相同的对象。你甚至可以用几乎相同的方式使用它。然而，在方法装饰器中，焦点应该放在我们实际装饰的属性上。在非类装饰器内部操作构造函数被认为是一种不好的做法。

## 方法包装装饰器

方法装饰器的最常见用法是将其用于包装原始方法，添加一些自定义的横切代码。例如，添加一些通用的错误处理或添加自动日志记录功能。

为了做到这一点，你需要更改被调用的函数。你可以通过使用方法属性描述符的 `value` 属性，以及使用属性访问器描述符的 `get` 和 `set` 属性来实现这一点。

## 练习 7.05：为方法创建日志装饰器

在这个练习中，你将创建一个装饰器，每次调用被装饰的方法或访问器时都会记录日志。你将使用这个装饰器向 `Teacher` 类添加日志记录，并验证每次使用被装饰的方法和属性访问器时，都会得到适当的日志条目：

注意

在你开始之前，请确保你已经设置了正确的编译器选项，如 *设置编译器选项* 部分所述。此练习的代码文件也可以从 [`packt.link/rmEZi`](https://packt.link/rmEZi) 下载。

1.  打开 Visual Studio Code，在新的目录（`Exercise05`）中创建一个新文件，并将其保存为 `teacher-logging.ts`。

1.  在 `teacher-logging.ts` 文件中输入以下代码：

    ```js
    class Teacher {
        constructor (public name: string){}
        private _title: string = "";
        public get title() { 
            return this._title;
        }

        public set title(value: string) {
            this._title = value;
        }
        public teach() {
            console.log(`${this.name} is teaching`)
        }
    }
    /////////////////
    const teacher = new Teacher("John Smith");
    teacher.teach(); // we're invoking the teach method
    teacher.title = "Mr." // we're invoking the title setter
    console.log(`${teacher.title} ${teacher.name}`); // we're invoking the title getter
    ```

1.  执行代码，注意它将输出 `John Smith is teaching` 和 `Mr. John Smith` 到控制台。

1.  创建一个方法装饰器工厂，可以包装任何方法、获取器或设置器，并添加一个日志语句。它将接受一个字符串参数并返回一个装饰器函数。最初，你不会对属性描述符进行任何更改：

    ```js
    function LogMethod(message: string) {
        return function (target: any, propertyName: string, descriptor: PropertyDescriptor) {
        };
    }
    ```

1.  使用带有适当消息参数的 `LogMethod` 装饰器装饰 `teach` 方法和 `title` 获取器：

    ```js
        @LogMethod("Title property")
        public get title() { 
        //...    
        @LogMethod("Teach method")
        public teach() {
        //...    
    ```

1.  执行代码，注意行为没有发生变化。

1.  现在，向你的应用程序添加一个 `logger` 对象：

    ```js
    const logger = {
        info: (message: string) => {
            console.log(`[INFO]: ${message}`);
        },
    };
    ```

    在实际的生产级实现中，你可能需要将日志记录到数据库、文件、第三方服务等等。在上一个步骤中，你只是将日志记录到控制台。

1.  向装饰器工厂添加代码，用于包装属性描述符，包括 `value`、`get` 和 `set` 属性（如果存在）：

    ```js
    function LogMethod(message: string) {
        return function (target: any, propertyName: string, descriptor: PropertyDescriptor) {
            if (descriptor.value) {
                const original = descriptor.value;
                descriptor.value = function (...args: any[]) {
                    logger.info(`${message}: Method ${propertyName} invoked`);
                    // we're passing in the original arguments to the method
                    return original.apply(this, args);
                }
            }
            if (descriptor.get) {
                const original = descriptor.get;
                descriptor.get = function () {
                    logger.info(`${message}: Getter for ${propertyName} invoked`);
                    // getter accessors do not take parameters
                    return original.apply(this, []);
                }
            }
            if (descriptor.set) {
                const original = descriptor.set;
                descriptor.set = function (value: any) {
                    logger.info(`${message}: Setter for ${propertyName} invoked`);
                    // setter accessors take a single parameter, i.e. the value to be set
                    return original.apply(this, [value]);
                }
            }
        }
    }
    ```

1.  执行代码并验证，当你调用方法以及使用 `title` 属性时，都会在控制台得到日志消息。

    ```js
    [INFO]: Teach method: Method teach invoked
    John Smith is teaching
    [INFO]: Title property: Setter for title invoked
    [INFO]: Title property: Getter for title invoked
    Mr. John Smith
    ```

在这个练习中，你看到了如何包装提供的方法和属性访问器类的定义，以便在每次调用时运行自定义代码，而不改变函数本身的行为。你使用了这一点来为没有任何日志记录功能的函数添加日志记录功能。你构建了这个类的对象并验证了日志功能的运行情况。

# 活动第 7.01 部分：创建用于调用计数的装饰器

作为网站后端服务的开发者，你被要求创建一个解决方案，使运营部门能够清楚地审计服务的操作行为。为此，应用程序需要统计所有类的实例化和方法调用。

在这个活动中，你将创建类和方法装饰器，这些装饰器可以用来统计类的实例化和方法调用次数。你将创建一个包含有关个人信息的数据的类，并使用装饰器来统计创建了多少个这样的对象以及每个方法被调用了多少次。在你构建了几个对象并使用它们的属性之后，查看计数器的值。

本活动的目的是展示类和方法装饰器的用途，以便解决应用程序的横切关注点，而不改变给定类的功能。你应该有关于你的对象生命周期的详细统计信息，而不增加任何业务逻辑的复杂性。

以下步骤将帮助你解决问题：

注意

在开始之前，请确保你已经设置了如 *设置编译器选项* 部分所述的正确编译器选项。此活动的代码文件也可以从 [`packt.link/UK49t`](https://packt.link/UK49t) 下载。

1.  创建一个名为 `Person` 的类，具有名为 `firstName`、`lastName` 和 `birthday` 的公共属性。

1.  添加一个构造函数，通过构造函数参数初始化属性。

1.  添加一个名为 `_title` 的私有字段，并通过名为 `title` 的属性作为 getter 和 setter 公开它。

1.  添加一个名为 `getFullName` 的方法，它将返回一个人的全名。

1.  添加一个名为 `getAge` 的方法，它将返回人的当前年龄（通过从当前年份减去生日）。

1.  创建一个全局对象 `count` 并将其初始化为空对象。这将作为你的状态变量，用于存储每个实例化和调用的计数。

1.  创建一个名为 `CountClass` 的构造函数包装装饰器工厂，它将接受一个名为 `counterName` 的字符串参数。我们将使用该参数作为 `count` 对象的键。

1.  在包装代码内部，通过 `counterName` 参数定义的 `count` 对象的属性增加 1。

1.  不要忘记设置包装构造函数的原型链。

1.  创建一个名为 `CountMethod` 的方法包装装饰器工厂，它将接受一个名为 `counterName` 的字符串参数。

1.  添加检查以确定`descriptor`参数是否具有`value`、`get`和`set`属性。您需要涵盖两种情况，即此装饰器用作访问器和方法装饰器时的情况。

1.  在每个相应的分支中，添加包装方法的代码。

1.  在包装代码内部，将`counterName`参数中定义的`count`对象的属性增加 1。

1.  使用`CountClass`装饰器装饰类，并带有`person`参数。

1.  使用`CountMethod`装饰器分别装饰`getFullName`、`getAge`和`title`属性获取器，参数分别为`person-full-name`、`person-age`和`person-title`。请注意，您只需要装饰一个属性访问器。

1.  在类外部编写代码，以实例化三个`person`对象。

1.  编写代码，调用对象的`getFullName`和`getAge`方法。

1.  编写代码，检查`title`属性是否为空，如果为空则设置它。

1.  编写代码，将`count`对象记录到控制台，以查看您的装饰器是否运行正确。

预期输出如下：

```js
{
    person: 3,
    "person-full-name": 3,
    "person-age": 3,
    "person-title": 6
}
```

此活动展示了使用装饰器扩展和增强类功能而不污染代码的能力。您能够在不更改任何底层业务逻辑的情况下，向对象中注入自定义代码执行。

注意

活动的解决方案可以通过此链接找到。

# 在装饰器中使用元数据

到目前为止，您一直在装饰类和方法。这些基本上是执行中的代码片段，您已经能够更改和增强执行中的代码。但您的代码不仅包括“活跃”的代码，还包括其他定义——特别是，您的类有字段，您的方 法有参数。在上一节的活动之前，您能够检测到何时访问`title`属性，因为您有一个获取值的方法和一个设置值的方法——所以您将代码附加到已经存在的“活跃”代码上。但您如何装饰程序的“被动”部分呢？您不能附加在“被动”代码执行时运行的代码，因为坦白地说，在`public firstName: string`中没有什么可以执行的。这是一个简单的定义。

您不能为您的“被动代码”附加任何执行代码，但您可以使用装饰器向有关装饰的“被动”代码片段的某个全局对象添加一些数据。在 *活动 7.01：为调用计数创建装饰器* 中，您定义了一个全局 `count` 对象，并在装饰器中使用它来跟踪执行。这种方法是可行的，但它需要创建一个全局变量，这在大多数情况下都是不好的。如果您能够在方法和类本身上定义某种属性，那就更干净利落了。但另一方面，您不希望添加太多与业务逻辑代码并存的属性——意外错误的概率太高。您需要能够以某种方式向您的类和方法添加元数据。

幸运的是，这是一个常见问题，并且有一个提议要为 JavaScript 添加适当的元数据支持。在此期间，有一个名为 **reflect-metadata** 的 polyfill 库可以用来。

备注

有关 `reflect-metadata` 库的更多信息，请访问 [`www.npmjs.com/package/reflect-metadata`](https://www.npmjs.com/package/reflect-metadata)。

这个库本质上所做的，是向您的类附加一个特殊属性，为我们提供了一个存储、检索和处理有关类元数据的地方。

在 TypeScript 中，为了使用此功能，您必须指定一个额外的编译器标志，无论是通过命令行还是通过 `tsconfig.json`。这就是 `emitDecoratorMetadata` 标志，需要将其设置为 `true` 以便与元数据方法一起使用。

## Reflect 对象

`reflect-metadata` 库的 API 很简单，您主要可以关注以下方法：

+   `Reflect.defineMetadata`: 在类或方法上定义元数据

+   `Reflect.hasMetadata`: 返回一个布尔值，指示是否存在某个特定的元数据

+   `Reflect.getMetadata`: 如果存在，返回实际的元数据

考虑以下代码：

```js
class Teacher {
    constructor (public name: string){}
    private _title: string = "";
    public get title() { 
        return this._title;
    }

    public set title(value: string) {
        this._title = value;
    }
    public teach() {
        console.log(`${this.name} is teaching`)
    }
}
```

这里有一个名为 `Teacher` 的类，它有一个简单的私有字段 `_title`，该字段为名为 `title` 的属性提供了 `get` 和 `set` 访问器方法，还有一个名为 `teach` 的方法，该方法将向控制台记录教师实际上正在教学。

您可以在 `Teacher` 类上定义一个名为 `call-count` 的元数据键，并通过执行以下 `defineMetadata` 调用来将其值设置为 `0`：

```js
Reflect.defineMetadata("call-count", 0, Teacher);
```

如果您想在 `teach` 方法上而不是在 `Teacher` 类本身上添加一个名为 `call-count` 的元数据键，您可以使用以下 `defineMetadata` 调用来实现：

```js
Reflect.defineMetadata("call-count", 10, Teacher, "teach");
```

这将在 `Teacher` 类的 `teach` 属性上定义一个名为 `call-count` 的元数据键，并将其值设置为 `10`。您可以使用以下命令检索这些值：

```js
Reflect.getMetadata("call-count", Teacher); // will return 0
Reflect.getMetadata("call-count", Teacher, "teach"); // will return 10
```

从本质上讲，您可以使用以下代码创建一个方法，该方法将方法调用注册为：

```js
function increaseCallCount(target: any, propertyKey: string) {
    if (Reflect.hasMetadata("call-count", target)) {
        const value = Reflect.getMetadata("call-count", target, propertyKey);
        Reflect.defineMetadata("call-count", value+1, target, propertyKey)
    } else {
        Reflect.defineMetadata("call-count", 1, target, propertyKey)
    }
}
```

此代码将首先调用`hasMetadata`方法，以检查你是否已经为`call-count`元数据定义了一个值。如果是`true`，则`hasMetadata`方法将调用`getMetadata`以获取当前值，然后调用`defineMetadata`以重新定义元数据属性，其值为增加的（`value+1`）。如果你没有这样的元数据属性，则`defineMetadata`方法将使用值为 1 来定义它。

当使用`increaseCallCount(Teacher, "teach");`调用时，它将成功增加`Teacher`类中`teach`方法的调用次数。添加到类中的元数据将不会以任何方式阻碍类已有的行为，因此正在执行的任何代码都不会受到影响。

## 练习 7.06：通过装饰器添加方法元数据

在这个练习中，我们将创建一个简单的类并为其方法添加一些元数据。完成此操作后，你将编写一个函数，给定一个类，将显示其可用的描述：

注意

在你开始之前，请确保你已经设置了如*设置编译器选项*部分中提到的正确编译器选项。此练习的代码文件也可以从[`packt.link/JG4F8`](https://packt.link/JG4F8)下载。

1.  打开 Visual Studio Code，在新的目录（`Exercise06`）中创建一个新文件，并将其保存为`calculator-metadata.ts`。

1.  在`calculator-metadata.ts`中输入以下代码：

    ```js
    class Calculator {
        constructor (public first: number, public second: number) {}
        public add() {
            return this.first + this.second;
        }
        public subtract() {
            return this.first – this.second;
        }
        public multiply() {
            return this.first / this.second;
        }
        public divide() {
            return this.first / this.second;
        }
    }
    ```

1.  接下来，为类及其一些方法添加元数据描述：

    ```js
    Reflect.defineMetadata("description", "A class that offers common operations over two numbers", Calculator);
    Reflect.defineMetadata("description", "Returns the result of adding two numbers", Calculator, "add");
    Reflect.defineMetadata("description", "Returns the result of subtracting two numbers", Calculator, "subtract");
    Reflect.defineMetadata("description", "Returns the result of dividing two numbers", Calculator, "divide");
    ```

1.  定义一个函数，当给定一个类时，将对其反思并提取并显示该类的`description`元数据：

    ```js
    function showDescriptions (target: any) {
        if (Reflect.hasMetadata("description", target)) {
            const classDescription = Reflect.getMetadata("description", target);
            console.log(`${target.name}: ${classDescription}`);
        }
    }
    ```

1.  使用`showDescriptions(Calculator);`调用该函数，并验证它将显示以下输出：

    ```js
    Calculator: A class that offers common operations over two numbers
    ```

    为了获取类的所有方法列表，我们必须使用`Object.getOwnPropertyNames`函数。此外，由于方法实际上是在类的原型上定义的，因此获取类所有方法名称的正确行是`const methodNames = Object.getOwnPropertyNames(target.prototype);`。

1.  接下来，遍历返回的数组并检查每个方法是否有描述。`showDescription`函数现在将具有以下格式：

    ```js
    function showDescriptions (target: any) {
        if (Reflect.hasMetadata("description", target)) {
            const classDescription = Reflect.getMetadata("description", target);
            console.log(`${target.name}: ${classDescription}`);
            const methodNames = Object.getOwnPropertyNames(target.prototype);
            for (const methodName of methodNames) {
                if (Reflect.hasMetadata("description", target, methodName)) {
                    const description = Reflect.getMetadata("description", target, methodName);
                    console.log(`  ${methodName}: ${description}`);
                }
            }
        }
    }
    ```

1.  再次调用该函数并验证它将显示以下输出：

    ```js
    Calculator: A class that offers common operations over two numbers
      add: Returns the result of adding two numbers
      subtract: Returns the result of subtracting two numbers
      divide: Returns the result of dividing two numbers
    ```

注意，你没有为`multiply`方法显示任何内容，因为你没有为其添加任何元数据。

在这个练习中，你学习了如何向类和方法添加元数据，以及如何检查其存在性，如果存在，则检索它。你还成功获取了给定类的所有方法的列表。

# 属性装饰器

属性装饰器是一个应用于类单个属性的装饰器函数。与方法或类装饰器不同，你不能修改或替换属性定义，但你确实可以观察它。

注意

由于你在装饰器中接收构造函数，这并不完全正确。你可以更改类的代码，但这非常不推荐。

当调用属性装饰器时，它接收两个参数：`target`和`propertyKey`：

+   `target`: 由于属性可以是实例属性（在类的实例上定义）和静态属性（在类本身上定义），因此`target`可以是两件不同的事情。对于实例属性，它是类的原型。对于静态属性，它是类的构造函数。通常，你会将此参数类型指定为`any`。

+   `propertyKey`: 这是你要装饰的属性的名称。

与方法装饰器不同，你不会收到属性描述符参数，因为显然没有可用的参数。另外，因为你没有返回任何可以替换的代码，所以属性装饰器的返回值被忽略。

例如，你可以定义一个简单的属性装饰器工厂，它只是将消息记录到控制台以通知属性实际上已被装饰：

```js
Example_PropertyDecorators.ts
1 function DecorateProperty(message: string) {
2     return function (target: any, propertyKey: string) {
3        console.log(`Decorated 
4 ${target.constructor.name}.${propertyKey} with '${message}'`);
5     }
6 }
Link to the preceding example: https://packt.link/HkkNi.
```

考虑以下类定义：

```js
class Teacher {
    public id: number;
    public name: string;
    constructor(id: number, name: string) {
        this.id = id;
        this.name = name;
    }
}
```

你可以使用以下代码注释`id`和`name`属性：

```js
    @DecorateProperty("ID")
    public id: number;
    @DecorateProperty("NAME")
    public name: string;
```

如果你现在执行代码（我们不需要调用任何东西；TypeScript 引擎将调用它），你将获得以下输出：

```js
Decorated Teacher.id with 'ID'
Decorated Teacher.name with 'NAME'
```

注意，你没有创建任何教师类的对象，也没有调用任何方法。装饰器在定义类时执行。由于属性装饰器是被动型的，通常你会使用它们将某种数据输入到某种机制中，该机制将使用这些数据。常见的方法之一是将被动装饰器与一个或多个主动装饰器（即类和方法的装饰器）结合使用。

注意

例如，在 Angular 中，这是常见的情况，其中被动的`@Input`和`@Output`装饰器与主动的`@Component`装饰器结合使用。

另一个常见的用例是有一个额外的机制来获取装饰器提供的数据并使用它。例如，装饰器可以记录一些元数据，然后有另一个函数读取并使用这些元数据。

## 练习 7.07：创建和使用属性装饰器

在这个练习中，你将创建一个简单的属性装饰器工厂，将为每个属性提供一个描述。完成此操作后，你将编写一个函数，该函数给定一个类将显示其可用的描述：

注意

在开始之前，请确保你已经设置了如*设置编译器选项*部分所述的正确编译器选项。此练习的代码文件也可以从[`packt.link/1WU6d`](https://packt.link/1WU6d)下载。

1.  打开 Visual Studio Code，在新的目录（`Exercise07`）中创建一个新文件，并将其保存为`teacher-properties.ts`。

1.  在`teacher-properties.ts`文件中输入以下代码：

    ```js
    class Teacher {
        public id: number;
        public name: string;
        constructor(id: number, name: string) {
            this.id = id;
            this.name = name;
        }
    }
    ```

1.  添加一个装饰器工厂，它接受一个字符串参数并生成一个属性装饰器，该装饰器将为给定属性添加一个元数据`description`字段：

    ```js
    function Description(message: string) {
        return function (target: any, propertyKey: string) {
            Reflect.defineMetadata("description", message, target, propertyKey)
        }
    }
    ```

1.  接下来，使用以下描述注释`Teacher`类的属性：

    ```js
        @Description("This is the id of the teacher")
        public id: number;
        @Description("This is the name of the teacher")
        public name: string;
    ```

1.  定义一个函数，当给定一个对象时，将反思该对象并提取并显示该对象的`description`元数据：

    ```js
    function showDescriptions (target: any) {
        for (const key in target) {
            if (Reflect.hasMetadata("description", target, key)) {
                const description = Reflect.getMetadata("description", target, key);
                console.log(`  ${key}: ${description}`);
            }
        }
    }
    ```

1.  创建一个`Teacher`类的对象：

    ```js
    const teacher = new Teacher(1, "John Smith");
    ```

1.  将该对象传递给`showDescriptions`函数：

    ```js
    showDescriptions(teacher);
    ```

1.  执行代码并验证描述是否已显示：

    ```js
      id: This is the id of the teacher
      name: This is the name of the teacher
    ```

在这个练习中，你学习了如何使用属性装饰器为属性添加元数据，以及如何使用属性装饰器为你的类添加快速的基本文档。

# 参数装饰器

参数装饰器是一个应用于函数调用单个参数的装饰器函数。就像属性装饰器一样，参数装饰器是被动型的，也就是说，它们只能用来观察值，但不能注入和执行代码。参数装饰器的返回值同样被忽略。因此，参数装饰器几乎总是与其他主动装饰器一起使用。

当调用参数装饰器时，它接收三个参数：`target`、`propertyKey`和`parameterIndex`：

+   `target`：这个参数的行为与相应的方法装饰器相同。如果参数在类的构造函数上，则有一个例外，但这将在稍后解释。

+   `propertyKey`：这是你要装饰的方法的名称（类的构造函数例外将在稍后解释）。

+   `parameterIndex`: 这是在函数参数列表中参数的序号索引（第一个参数从零开始）。

因此，让我们有一个函数，它将简单地记录`target`、`propertyKey`和`parameterIndex`参数到控制台：

```js
Example_ParameterDecorators.ts
1 function DecorateParam(target: any, propertyName: string,
2 parameterIndex: number) {
3    console.log("Target is:", target);
4    console.log("Property name is:", propertyName);
5    console.log("Index is:", parameterIndex);
6 }
Link to the preceding example: https://packt.link/5vuL2.
```

你可以使用这个函数来装饰函数的参数，并调查参数装饰器的使用情况。让我们从一个简单的类开始：

```js
class Teacher {
    public id: number;
    public name: string;
    constructor(id: number, name: string) {
        this.id = id;
        this.name = name;
    }
    public getFullName(title: string, suffix: string) {
        return `${title} ${this.name}, ${suffix}`
    }
}
```

该类有一个接受两个参数`id`和`name`的构造函数，以及一个名为`getFullName`的方法，该方法接受两个参数`title`和`suffix`。假设你将装饰器添加到`getFullName`方法的第一个参数上，如下所示：

```js
     public getFullName(@DecorateParam title: string, suffix: string) {
        // ....
```

如果你运行你的代码（不需要实例化类），你将在控制台上得到以下输出：

```js
Target is: Teacher {}
Property name is: getFullName
Index is: 0
```

我们还可以将参数装饰器应用于构造函数本身的参数。比如说，你装饰第二个构造函数参数，如下所示：

```js
    constructor(id: number, @DecorateParam name: string) {
        // ....
```

当你运行代码时，你会得到以下输出：

```js
Target is: [Function: Teacher]
Property name is: undefined
Index is: 1
```

注意，在这种情况下，目标不是类的原型，而是类构造函数本身。另外，当装饰构造函数参数时，属性的名称是`undefined`。

## 练习 7.08：创建和使用参数装饰器

在这个练习中，你将创建一个参数装饰器，它将指示某个参数是必需的；也就是说，它不应该有空的值。你还将创建一个用于方法的验证装饰器，以便验证实际上可以发生。我们将创建一个使用装饰器的类，你将尝试使用有效和无效的值调用该方法：

注意

在开始之前，请确保你已经设置了如*设置编译器选项*部分所述的正确编译器选项。此练习的代码文件也可以从[`packt.link/Hf3fv`](https://packt.link/Hf3fv)下载。

1.  打开 Visual Studio Code，在新的目录（`Exercise08`）中创建一个新文件，并将其保存为`teacher-parameters.ts`。

1.  在`teacher-parameters.ts`中输入以下代码：

    ```js
    class Teacher {
        public id: number;
        public name: string;
        constructor(id: number, name: string) {
            this.id = id;
            this.name = name;
        }
        public getFullName(title: string, suffix: string) {
            return `${title} ${this.name}, ${suffix}`
        }
    }
    ```

1.  创建一个名为`Required`的参数装饰器，它将给定的属性的索引添加到类的`required`元数据字段中：

    ```js
    function Required(target: any, propertyKey: string, parameterIndex: number) {
        if (Reflect.hasMetadata("required", target, propertyKey)) {
            const existing = Reflect.getMetadata("required", target, propertyKey) as number[];
            Reflect.defineMetadata("required", existing.concat(parameterIndex), target, propertyKey);
        } else {
            Reflect.defineMetadata("required", [parameterIndex], target, propertyKey)
        }
    }
    ```

    在这里，如果元数据已经存在，这意味着还有一个必需的参数。如果是这样，你加载它并将你的`parameterIndex`连接起来。如果没有先前的元数据，你使用包含你的`parameterIndex`的数组定义它。

1.  接下来，创建一个方法装饰器，它将包装原始方法并在调用原始方法之前检查所有必需参数：

    ```js
    function Validate(target: any, propertyKey:string, descriptor: PropertyDescriptor) {
        const original = descriptor.value;
        descriptor.value = function (...args: any[]) {
            // validate parameters
            if (Reflect.hasMetadata("required", target, propertyKey)) {
                const requiredParams = Reflect.getMetadata("required", target, propertyKey) as number[];
                for (const required of requiredParams) {
                    if (!args[required]) {
                        throw Error(`The parameter at position ${required} is required`)
                    }
                }
            }
            return original.apply(this, args);
        }
    }
    ```

    如果你的任何必需参数有一个假值，那么你的装饰器将不会执行原始方法，而是抛出一个错误。

1.  然后，使用`Required`装饰器注释`getFullName`方法的`title`参数，并使用`Validate`装饰器注释该方法本身：

    ```js
        @Validate
        public getFullName(@Required title: string, suffix: string) {
            // ....
    ```

1.  创建`Teacher`类的对象：

    ```js
    const teacher = new Teacher(1, "John Smith");
    ```

1.  尝试使用空字符串作为第一个参数调用`getFullName`方法：

    ```js
    try {
         console.log(teacher.getFullName("", "Esq"));
    } catch (e) {
         console.log(e.message);
    }
    ```

1.  执行代码并验证是否显示错误消息：

    ```js
    The parameter at position 0 is required
    ```

在这个练习中，你了解了如何创建参数装饰器以及如何使用它们来添加元数据。你还协调了将相同的元数据用于另一个装饰器，并构建了一个基本的验证系统。

# 在单个目标上应用多个装饰器

经常需要在单个目标上应用多个装饰器。由于装饰器可以（并且确实）更改实际执行的代码，因此了解不同装饰器如何协同工作是很重要的。

基本上，装饰器是函数，你使用它们来组合你的目标。这意味着本质上，装饰器将自下而上地应用和执行，最接近目标的目标装饰器先执行并提供结果给下一个装饰器，依此类推。这类似于函数式组合；也就是说，当我们试图计算`f(g(x))`时，首先调用`g`函数，然后调用`f`函数。

当使用装饰器工厂时，有一个小陷阱。组合规则仅适用于装饰器本身——而装饰器工厂本身不是装饰器。它们是需要执行以返回装饰器的函数。这意味着它们按照源代码顺序执行，即自上而下。想象一下，你有两个装饰器工厂：

```js
Example_MultipleDecorators.ts
1 function First () {
2    console.log("Generating first decorator")
3    return function (constructor: Function) {
4        console.log("Applying first decorator")
5    }
6 }
Link to the preceding example https://packt.link/jMhDj.
```

第二个装饰器工厂：

```js
7  function Second () {
8      console.log("Generating second decorator")
9      return function (constructor: Function) {
10         console.log("Applying second decorator")
11     }
12 }
```

现在想象它们被应用于单个目标：

```js
13 @First()
14 @Second()
15 class Target {}
```

生成过程将在第二个装饰器之前生成第一个装饰器，但在应用过程中，第二个装饰器将被应用，然后才是第一个：

```js
Generating first decorator
Generating second decorator
Applying second decorator
Applying first decorator
```

## 活动第 7.02 部分：使用装饰器应用横切关注点

在这个活动中，我们将完整地回到篮球游戏示例 (`Example_Basketball.ts`)。您需要以可维护的方式向 `Example_Basketball.ts` 文件添加所有必要的横切关注点，例如身份验证、性能指标、审计和验证。

您可以使用已经在 `Example_Basketball.ts.` 中存在的代码开始活动。首先，盘点一下文件中已经存在的元素：

+   描述团队的接口。

+   游戏本身的类。您有一个构造函数，根据团队名称创建团队对象。您还有一个 `getScore` 函数，用于显示得分，以及一个简单的 `updateScore` 方法，该方法更新游戏的得分，并接受得分团队和得分值作为参数。

现在您需要在不更改类本身代码的情况下，仅通过使用装饰器添加之前提到的横切关注点。

在 `Example_Basketball.ts` 中更早的时候，您必须完全将计分业务逻辑包含在处理其他所有事情（如授权、审计、指标等）所需的代码中。现在应用所有需要的装饰器技能，以便应用程序能够正常运行，同时代码库仍然清晰简洁。

注意

此活动的代码文件也可以从 [`packt.link/7KfCx`](https://packt.link/7KfCx) 下载。

以下步骤将帮助您解决问题：

1.  为 `BasketBallGame` 类创建代码。

1.  创建一个名为 `Authenticate` 的类装饰器工厂，它将接受一个 `permission` 参数并返回一个具有构造函数包装的类装饰器。类装饰器应加载 `permissions` 元数据属性（字符串数组），然后检查传递的参数是否为数组的元素。如果传递的参数不是数组的元素，类装饰器应抛出错误；如果存在，则继续进行类创建。

1.  定义一个名为 `permissions` 的 `BasketballGame` 类元数据属性，其值为 `["canUpdateScore"]`。

1.  使用参数值为 `canUpdateScore` 的类装饰器工厂对 `BasketballGame` 类进行应用。

1.  创建一个名为 `MeasureDuration` 的方法装饰器，它将使用方法包装在方法体执行之前启动计时器，并在完成后停止计时。它应计算持续时间并将其推送到名为 `durations` 的方法元数据属性中。

1.  在 `updateScore` 方法上应用 `MeasureDuration` 方法装饰器。

1.  创建一个名为 `Audit` 的方法装饰器工厂，它将接受一个 `message` 参数并返回一个方法装饰器。该方法装饰器应使用方法包装来获取方法的参数和返回值。在原始方法成功执行后，它应在控制台显示审计日志。

1.  将 `Audit` 方法装饰器工厂应用于 `updateScore` 方法，参数值为 `Updated score`。

1.  创建一个名为 `OneTwoThree` 的参数装饰器，它将装饰的参数添加到 `one-two-three` 元数据属性中。

1.  创建一个名为 `Validate` 的方法装饰器，它将使用方法包装来加载 `one-two-three` 元数据属性的值，并对所有标记的参数检查它们的值。如果值为 `1`、`2` 或 `3`，则应继续执行原始方法。如果不是，则应停止执行并显示错误。

1.  将 `OneTwoThree` 装饰器应用于 `updateScore` 的 `byPoints` 参数，并将 `Validate` 装饰器应用于 `updateScore` 方法：

    ```js
    Create a game object, and update its score a few times. The console should reflect the applications of all decoratorsas shown:
    [AUDIT] Updated score (updateScore) called with arguments:
    [AUDIT] [ 3, true ]
    [AUDIT] and returned result:
    [AUDIT] undefined
    //…
    [AUDIT] Updated score (updateScore) called with arguments:
    [AUDIT] [ 2, true ]
    [AUDIT] and returned result:
    [AUDIT] undefined
    [AUDIT] Updated score (updateScore) called with arguments:
    [AUDIT] [ 2, false ]
    [AUDIT] and returned result:
    [AUDIT] undefined
    7:8
    ```

    注意

    为了便于展示，这里只展示了预期输出的部分。本活动的解决方案可以通过这个链接找到。

在这个活动中，你利用装饰来快速高效地实现复杂的横切关注点。当你成功完成活动后，你将根据应用需求实现多种类型的装饰器，从而在不牺牲清晰度和可读性的情况下扩展代码的功能。

# 摘要

在本章中，你了解了一种称为 **装饰** 的技术，它在 TypeScript 中是原生支持的。本章首先确立了使用装饰器的动机，然后探讨了 TypeScript 中的多种装饰器类型（类、方法、访问器、属性和参数装饰器），并考察了每种的可能性。你学习了如何使用类装饰器交换或更改类的完整构造函数，如何使用方法装饰器包装单个方法或属性访问器，以及如何使用属性和参数装饰器丰富可用的元数据。

本章还讨论了主动装饰器和被动装饰器之间的区别，这归结为代码和定义之间的区别。你实现了每种装饰器类型的几个常见变体，并展示了不同类型的装饰器如何很好地相互补充。本章应有助于你轻松管理来自第三方库（如 Angular）以及你自己创建的装饰器工厂中装饰器的使用和创建。在下一章中，我们将开始探索 TypeScript 中的依赖注入。
