# 第九章：Elm 中的测试

在这一章中，我们将学习如何测试我们的 Elm 应用。

我们将涵盖的主题包括：

+   使用`elm-test`测试 Elm 应用

+   理解`elm-package.json`文件的结构和作用

+   理解`tests`文件夹的结构和功能

+   使用`describe`、`test`和`Expect`

+   使用`left pipe operator`编写更容易理解的测试

+   在我们的测试中使用`let-in`和`case-of`表达式

+   Elm 中的模糊测试

完成这一章后，您将：

+   对单元测试的工作原理有一个大致的了解

+   了解单元测试和模糊测试在 Elm 中的工作原理

+   了解您可以对 Elm 应用进行测试的不同方式

+   能够成功部署各种测试到您的 Elm 应用中

# Elm 测试简介

由于编译器在编译时捕捉所有错误，因此对于成功编译的 Elm 应用来说，零运行时异常是预期的结果。

既然如此，人们可能会问我们是否真的需要测试我们的 Elm 应用。答案是响亮的肯定，主要是因为编译器不会测试应用的*行为*。虽然 Elm 编译器的错误检查确实是一个非常好的工具，但它只会测试逻辑不一致性，这就是全部。

为了准备这一章，我们将从上一章的代码开始。只需复制整个名为`improved-weather-app-ch8`的文件夹，并将其粘贴到另一个位置。

与本书一起提供的代码文件在`chapter9`中包含了这个粘贴的文件夹，并且可以找到的复制粘贴的文件夹在`chapter9`文件夹内部已被重命名为`weather-app-with-tests-ch9`。

有时简单地复制粘贴一个现有的 Elm 项目到一个新文件夹，然后对其进行修改，而不是从头开始使用例如`create-elm-app`包来创建它，这是一种很有帮助的方法。

# 理解 Elm 中测试的工作原理

我们将通过将控制台指向`weather-app-with-tests-ch9`文件夹来开始使用 Elm 进行测试的工作。

一旦我们将终端指向正确的目录，我们则需要将`elm-test`作为一个 npm 包进行安装。Elm 社区中有许多工具，它们都有自己的 npm 版本，而`elm-test`只是其中之一。

因此，为了安装`elm-test`，让我们运行：

```js
npm install -g elm-test
```

就这样！由于我们传递了`-g`标志到命令中，`elm-test` npm 包现在可以在全局范围内使用。这意味着我们可以在任何文件夹中使用 elm-test，而不仅仅是当前所在的文件夹。

注意，安装 elm-test npm 包将需要一些时间，所以当它正在安装时，您可以随时休息。

一旦安装开始，将会有许多消息被记录到控制台。这些消息看起来类似于以下内容：

```js
C:\Users\PC\AppData\Roaming\npm\elm-test -> C:\Users\PC\AppData\Roaming\npm\node_modules\elm-test\bin\elm-test

> elm-test@0.18.12 install C:\Users\PC\AppData\Roaming\npm\node_modules\elm-test
> node install.js

Downloading binaries from https://dl.bintray.com/elmlang/elm-test/0.18.12/win32-x64.tar.gz
Successfully downloaded and processed https://dl.bintray.com/elmlang/elm-test/0.18.12/win32-x64.tar.gz
npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.1.2 (node_modules\elm-test\node_modules\fsevents):
npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.1.2: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})

+ elm-test@0.18.12
added 166 packages in 649.159s
```

注意上一段代码的最后一行，它读作：

```js
added 166 packages in 649.159s
```

这意味着安装花费了超过 10 分钟的时间，这是在等待安装完成时制作一杯好饮料的完美机会。一旦`elm-test`的安装完成，我们可以通过运行以下命令将测试文件夹添加到现有的 elm 应用中：

```js
elm-test init
```

一旦在控制台中运行前面的命令，控制台将通过记录以下消息来记录进度：

```js
Starting downloads...

 ● debois/elm-dom 1.2.3
 ● eeue56/elm-html-query 3.0.0
 ● eeue56/elm-html-in-elm 5.2.0
 ● eeue56/elm-lazy 1.0.0
 ● eeue56/elm-lazy-list 1.0.0
 ● eeue56/elm-html-test 5.1.3
 ● eeue56/elm-shrink 1.0.0
 ● elm-community/elm-test 4.2.0
 ● elm-lang/core 5.1.1
 ● debois/elm-mdl 8.1.0
 ● elm-lang/dom 1.1.1
 ● elm-lang/html 2.0.0
 ● elm-lang/http 1.0.0
 ● elm-lang/mouse 1.0.1
 ● elm-lang/virtual-dom 2.0.4
 ● elm-lang/window 1.0.1
 ● mgold/elm-random-pcg 5.0.2
 ● myrho/elm-round 1.0.2

Packages configured successfully!
```

让我们运行`dir`命令来查看我们项目的结构，从其根目录开始：

```js
dir
```

此命令将返回：

```js
elm-package.json  elm-stuff  public  README.md  src  tests
```

现在，让我们通过运行`cd tests`命令切换到`tests`文件夹。接下来，让我们再次运行`dir`命令来检查`tests`文件夹的结构。我们得到以下结果：

```js
elm-package.json  elm-stuff  Example.elm  Tests.elm
```

列表中的第一个文件，`elm-package.json`，列出了我们测试所需的所有依赖项。以下是位于`tests`文件夹内的`elm-package.json`文件的内容：

```js
{
    "version": "1.0.0",
    "summary": "Test Suites",
    "repository": "https://github.com/user/project.git",
    "license": "BSD3",
    "source-directories": [
        "..\\src",
        "."
    ],
    "exposed-modules": [],
    "dependencies": {
        "debois/elm-mdl": "8.1.0 <= v < 9.0.0",
        "eeue56/elm-html-test": "5.1.3 <= v < 6.0.0",
        "elm-community/elm-test": "4.0.0 <= v < 5.0.0",
        "elm-lang/core": "5.0.0 <= v < 6.0.0",
        "elm-lang/html": "2.0.0 <= v < 3.0.0",
        "elm-lang/http": "1.0.0 <= v < 2.0.0",
        "myrho/elm-round": "1.0.2 <= v < 2.0.0"
    },
    "elm-version": "0.18.0 <= v < 0.19.0"
}
```

以下代码中的行用于我们包的语义版本控制：

```js
    "version": "1.0.0",
    "summary": "Test Suites",
    "repository": "https://github.com/user/project.git",
    "license": "BSD3",   
```

这些行仅在您打算在 Elm 包页面上发布包时才相关。源目录列出了我们的源文件所在的文件夹。在我们的`tests`文件夹的情况下，它需要访问自身，表示为`.`，并且需要访问其父文件夹的`src`文件夹，表示为`..\\src`：

```js
    "source-directories": [
        "..\\src",
        "."
    ],
```

暴露的模块列出了您希望公开的所有模块。这仅在发布包时使用，否则不需要指定。

依赖项列表列出了我们的测试所依赖的所有包。将我们的天气应用文件夹根目录下的依赖项列表与我们的测试所使用的依赖项列表进行比较是很有趣的。为了比较这两个列表，让我们首先列出天气应用文件夹中的依赖项：

```js
    "dependencies": {
        "debois/elm-mdl": "8.1.0 <= v < 9.0.0",
        "elm-lang/core": "5.0.0 <= v < 6.0.0",
        "elm-lang/html": "2.0.0 <= v < 3.0.0",
        "elm-lang/http": "1.0.0 <= v < 2.0.0",
        "myrho/elm-round": "1.0.2 <= v < 2.0.0"
    },
```

接下来，让我们列出`test`文件夹内的依赖项：

```js
    "dependencies": {
        "debois/elm-mdl": "8.1.0 <= v < 9.0.0",
        "eeue56/elm-html-test": "5.1.3 <= v < 6.0.0",
        "elm-community/elm-test": "4.0.0 <= v < 5.0.0",
        "elm-lang/core": "5.0.0 <= v < 6.0.0",
        "elm-lang/html": "2.0.0 <= v < 3.0.0",
        "elm-lang/http": "1.0.0 <= v < 2.0.0",
        "myrho/elm-round": "1.0.2 <= v < 2.0.0"
    },
```

如我们所见，`tests`文件夹包括了我们的天气应用所使用的所有依赖项，还有一些额外的依赖项。由于我们已经知道天气应用`src`文件夹内的包的功能，让我们专注于那些特定于我们的测试文件夹的依赖项：

```js
        "eeue56/elm-html-test": "5.1.3 <= v < 6.0.0",
        "elm-community/elm-test": "4.0.0 <= v < 5.0.0",
```

如我们所见，有两个包是专门用于我们的`tests`文件夹的。在查看每个包的功能之前，让我们简要讨论包命名约定，以及我们之前列出的依赖项的版本。

Elm 包命名的方式很简单。包名中斜杠之前的部分是与此包关联的 GitHub 账户的名称。换句话说，它是包作者的 GitHub 用户名。包名中斜杠之后的部分应该尽可能具有描述性。因此，而不是为每个包强制唯一的命名，Elm 的理念是为包提供一个尽可能具有描述性的名称，这样它的潜在用户只需看一眼其名称就能辨别出包的用途。因此，完全有可能存在多个 `elm-html-test` 包，而名称的唯一性是由包作者的用户名决定的。

我们对 `eeue56/elm-html-test` 依赖项所需版本的范围在 5.1.3 和 6.0.0 之间，这意味着最低可接受的版本是 5.1.3，最高可接受的版本是 6.0.0。

要了解这两个包的功能，我们可以参考它们的官方文档，文档可在 [`package.elm-lang.org/packages/elm-community/elm-test/latest`](http://package.elm-lang.org/packages/elm-community/elm-test/latest) 和 [`package.elm-lang.org/packages/eeue56/elm-html-test/latest`](http://package.elm-lang.org/packages/eeue56/elm-html-test/latest) 找到。

`elm-community/elm-test` 包允许我们编写单元测试和模糊测试。`eeue56/elm-html-test` 允许我们通过指定我们期望的 HTML 值来测试视图。

继续检查测试文件夹的内容，`tests/elm-stuff` 文件夹包含下载的包以及我们依赖的确切版本列表，如 `exact-dependencies.json` 文件中所示。接下来，`Example.elm` 文件包含了我们运行第一个测试所需的所有设置。这是 `Example.elm` 文件包含的代码：

```js
module Example exposing (..)

import Expect exposing (Expectation)
import Fuzz exposing (Fuzzer, int, list, string)
import Test exposing (..)

suite : Test
suite =
    todo "Implement our first test. See http://package.elm-lang.org/packages/elm-community/elm-test/latest for how to do this!"
```

此文件中的代码遵循与其他 Elm 代码相同的规则。在第一行，我们暴露了 `Example` 模块。随后我们导入 `Expect`、`Fuzz` 和 `Test`，这些都是测试正常工作所必需的。

熟悉一些软件测试的术语会有所帮助，因为它将为我们编写测试提供更多上下文。测试被分组为测试套件。测试套件包含测试用例。测试用例是最小的测试单元。因此，适当地，我们在 `Example.elm` 中通过指定一个 `suite` 函数开始我们的测试，该函数将包含我们的测试。正如我们所看到的，`suite` 函数的类型是 `Test`。有关 `Test` 的官方文档可以在 [`package.elm-lang.org/packages/elm-community/elm-test/latest/Test#Test`](http://package.elm-lang.org/packages/elm-community/elm-test/latest/Test#Test) 找到。

如官方文档所述，一个 `Test` 至少会产生一个 `Expectation`。让我们看看我们测试中还将使用的一些其他函数。

# 描述函数

我们的测试被分组在一个`List`中。为了描述这个测试`List`的作用，我们使用`describe`函数。`describe`函数的签名可以在以下链接中找到[`package.elm-lang.org/packages/elm-community/elm-test/latest/Test#describe`](http://package.elm-lang.org/packages/elm-community/elm-test/latest/Test#describe)。

如我们从提供的 URL 中看到的那样，`describe`函数接受一个`String`和一个`Tests`的`List`，并返回一个`Test`。换句话说，它遵循以下结构：

```js
describe "An arbitrary description of our test" 
  [ test ...
  , test ...
  , test ... 
  ]
```

注意，这并不是实际的 Elm 代码，而是一种类似于 Elm 的伪代码，作为理解 Elm 中测试的中间步骤来编写的。

# 测试和 Expect 函数

`test`函数用于编写实际的单元测试。`test`函数只能有一个`Expectation`，并返回一个`Test`。

`Expectation`是一个类型别名。它可以有两种：`pass`或`fail`。`Expect`有几个函数，例如`Expect.equal`、`Expect.notEqual`、`Expect.lessThan`等等。有关`Expect`函数的完整列表，请参阅官方文档[`package.elm-lang.org/packages/elm-community/elm-test/latest/Expect`](http://package.elm-lang.org/packages/elm-community/elm-test/latest/Expect)。

现在我们已经介绍了一些概念并描述了我们将要使用的函数，是时候在`Example.elm`中编写我们的第一个单元测试了。

# 在 Elm 中编写单元测试

让我们更新`suite`函数的代码，使其看起来像这样：

```js
suite : Test
suite =
  describe "Zero is equal to zero"
    [ test "Zero is equal to one minus one" <|
        \_ -> Expect.equal 0 (1 - 1)
    , test "Zero is equal to two minus two" <|
        \_ -> Expect.equal 0 (2 - 2)
    ]
```

现在让我们从控制台运行这个测试，将控制台指向我们项目的根目录（在`test`文件夹之上的一级），并输入以下命令：

```js
elm-test tests/Example.elm
```

控制台将记录以下消息：

```js
$ elm-test tests/Example.elm
Success! Compiled 0 modules.
Successfully generated /dev/null
Success! Compiled 1 module.
Successfully generated C:\Users\PC\Desktop\improved-weather-app\elm-stuff\generated-code\elm-community\elm-test\elmTestOutput.js

elm-test 0.18.12
----------------

Running 2 tests. To reproduce these results, run: elm-test --fuzz 100 --seed 800244194 tests/Example.elm

TEST RUN PASSED

Duration: 517 ms
Passed: 2
Failed: 0
```

让我们看看我们刚刚测试的代码中的一个单个测试用例：

```js
test "Zero is equal to one minus one" <|
        \_ -> Expect.equal 0 (1 - 1)
```

让我们从`<|`管道操作符开始。它所做的就是评估其右侧的表达式，并将其作为参数传递给左侧的函数。换句话说，如果不使用`<|`操作符，我们可以这样编写完全相同的文本函数：

```js
test "Zero is equal to one minus one"
        ( \_ -> Expect.equal 0 (1 - 1) )
```

接下来，让我们看看括号内部，并描述那里发生了什么。括号内的代码是`test`函数的第二个参数：

```js
( \_ -> Expect.equal 0 (1 - 1) )
```

第二个参数是一个忽略其参数的匿名函数，这由以下代码表示：`_`。在本书早期，我们讨论了 Elm 中每个函数都是柯里化的，以及如何通过部分应用，我们可以得出结论，Elm 中的每个函数都可以被设计成只接受一个参数。使用`_`，这个参数就被忽略了。

匿名函数中的箭头（`->`）与常规函数中的等号（`=`）相同。`Expect.equal`将如之前所述，返回一个`pass`或`fail`。

`Expect.equal` 接受两个参数：第一个是期望值，第二个是要测试的表达式，该表达式要么与期望值相同（因此返回一个 `pass`），要么不同（因此返回一个 `fail`）。

由于零确实等于一减一，所以我们得到了一个 `pass`。

回到使用 `<|` 管道操作符，我们可以这样编写这个匿名函数：

```js
( \_ -> Expect.equal 0 <| 1 - 1 )
```

因此，如果我们想的话，可以没有任何括号地重写整个 `suite` 函数，如下所示：

```js
suite : Test
suite =
  describe "Zero is equal to zero"
    [ test "Zero is equal to one minus one" <|
        \_ -> Expect.equal 0 <| 1 - 1
    , test "Zero is equal to two minus two" <|
        \_ -> Expect.equal 0 <| 2 - 2
    ]
```

一切仍然会按预期工作，如果我们再次运行测试，我们的测试仍然会通过。

现在，让我们将注意力转向 `tests` 文件夹中的另一个测试文件，名为 `Tests.elm`。`Tests.elm` 文件有以下代码：

```js
module Tests exposing (..)

import Test exposing (..)
import Expect

-- Check out http://package.elm-lang.org/packages/elm-community/elm-test/latest to learn more about testing in Elm!

all : Test
all =
    describe "A Test Suite"
        [ test "Addition" <|
            \_ ->
                Expect.equal 10 (3 + 7)
        , test "String.left" <|
            \_ ->
                Expect.equal "a" (String.left 1 "abcdefg")
        , test "This test should fail" <|
            \_ ->
                Expect.fail "failed as expected!"
        ]
```

让我们在控制台中运行测试，只需从我们的应用程序文件夹的根目录运行 `elm-test` 命令：

```js
elm-test
```

以下信息将被记录到控制台：

```js
Success! Compiled 1 module.
Successfully generated /dev/null
Success! Compiled 1 module.
Successfully generated C:\Users\PC\Desktop\improved-weather-app\elm-stuff\generated-code\elm-community\elm-test\elmTestOutput.js

elm-test 0.18.12
----------------

Running 5 tests. To reproduce these results, run: elm-test --fuzz 100 --seed 1660804947

> Tests
> A Test Suite
> This test should fail

    failed as expected!

TEST RUN FAILED

Duration: 696 ms
Passed: 4
Failed: 1
```

这个测试失败了，但正如控制台输出所示，这是预期的。正如在 `Tests.elm` 文件的代码中可以看到的，当我们需要编写一个失败的测试时，我们只需简单地使用 `Expect.fail` 函数。

在接下来的章节中，我们将继续使用 `Tests.elm` 文件，一个有用的命令是我们可以使用 `elm-test --watch`。如果你熟悉可以在控制台中运行的许多不同的实用工具，那么 `--watch` 标志会监视我们代码中的更改，并重新运行附加了 `--watch` 标志的命令。

这意味着我们只需要运行一次 `elm-test --watch` 命令，每次我们保存 `Tests.elm` 文件时，它都会再次运行我们的测试套件。

# 在我们的测试中使用 let-in 表达式

让我们在我们的 `Example.elm` 文件中添加一个 let-in 表达式。为了做到这一点，我们将使用另一个 `describe` 函数来指定另一个测试套件，然后我们将用另一个 `describe` 函数包裹所有的测试。最外层的 `describe` 函数将包含所有的其他 `describe` 函数，而这些函数反过来将包含所有的 `test` 函数的 `Lists`：

```js
module Example exposing (..)

import Expect exposing (Expectation)
-- import Fuzz exposing (Fuzzer, int, list, string)
import Test exposing (..)

suite : Test
suite =
  describe "A Test Suite"
    [ describe "Testing addition"
      [ test "Addition" <|
        \_ ->
          Expect.equal 10 (3 + 7)
      ]
    , describe "Using let-in expression in a test suite"
      [ test "Multiplication" <|
        \_ ->
          let
            x = 2
            y = 4
            xy = x * y
          in
            xy
              |> \_ -> Expect.equal 8 (2 * 4)
      ]
    ]
```

使用 `elm-test tests/Example.elm` 运行此 `Example.elm` 文件将在控制台产生以下输出：

```js
Success! Compiled 1 module.
Successfully generated /dev/null
Success! Compiled 1 module.
Successfully generated C:\Users\PC\Desktop\improved-weather-app\elm-stuff\generated-code\elm-community\elm-test\elmTestOutput.js

elm-test 0.18.12
----------------

Running 2 tests. To reproduce these results, run: elm-test --fuzz 100 --seed 983607346 tests/Example.elm

TEST RUN PASSED

Duration: 334 ms
Passed: 2
Failed: 0
```

当然，这个例子很简单，但看到一个非常简单的实现，你可以从中构建出来，是有帮助的。

# 在我们的测试中解码 JSON

在我们的测试中解码 JSON 是简单的。

首先，让我们在我们的 `tests` 文件夹中添加一个新的文件。我们将把这个新文件命名为 `DecoderTests.elm`。

接下来，让我们将以下代码添加到 `DecoderTests.elm`：

```js
module DecoderTests exposing (..)

import Expect exposing (Expectation)
import Fuzz exposing (Fuzzer, int, list, string)
import Main exposing (..)
import Test exposing (..)

suite : Test
suite =
    describe "Decoder test"
        [ test "Test decoding valid string" <|
            \_ ->
                let
                    jsonInput =
                        "{\"main\":{\"temp\":303.15,\"pressure\":30,\"humidity\":20},\"wind\":{\"speed\":15.3},\"name\":\"Rome\"}"

                    expectedResult =
                        TemperatureInfo "Rome" 15.3 30.0 30.0 20.0
                in
                Expect.equal expectedResult (decodeTemperatureInfo jsonInput)

        {--
        , test "Test decoding invalid string" <|
            \_ ->
                let
                    jsonInput =
                        "Scrabbled json message"

                    expectedResult =
                        TemperatureInfo "Error decoding data!" 0 0 0 0
                in
                Expect.equal expectedResult (decodeTemperatureInfo jsonInput)
        , fuzz string "Fuzz test decoding invalid string" <|
            \randomlyGeneratedString ->
                let
                    expectedResult =
                        TemperatureInfo "Error decoding data!" 0 0 0 0
                in
                Expect.equal expectedResult (decodeTemperatureInfo randomlyGeneratedString)
        -}
        ]
```

现在，我们可以运行我们的测试：

```js
elm-test tests/DecoderTests.elm
```

控制台将记录以下信息：

```js
Success! Compiled 2 modules.
Successfully generated /dev/null
Success! Compiled 3 modules.
Successfully generated C:\Users\PC\Desktop\elm-web-development\chapter9\weather-app-with-tests-ch9\elm-stuff\generated-code\elm-community\elm-test\elmTestOutput.js

elm-test 0.18.12
----------------

Running 1 test. To reproduce these results, run: elm-test --fuzz 100 --seed 1485733894 tests/DecoderTests.elm

TEST RUN PASSED

Duration: 663 ms
Passed: 1
Failed: 0
```

我们编写的测试此 JSON 的代码总共有三个测试。第一个测试编写得不会失败。其他两个测试编写得会失败，但它们被注释掉了。你可以随意取消注释其他两个测试，并在控制台中查看结果。

为了能够正确解析 JSON，我们需要在 JSON 字符串中转义双引号。这就是为什么 `jsonInput` 变量看起来是这样的：

```js
jsonInput =
"{\"main\":{\"temp\":303.15,\"pressure\":30,\"humidity\":20},\"wind\":\"speed\":15.3},\"name\":\"Rome\"}"
```

另一种方法是，将整个 JSON 字符串放在多行引号的开头和结尾处，正如本书前面提到的，这些引号是三个双引号，如下所示：`"""`。我们将在下一节构建一些自定义期望时看到这个方法的示例。

# 在解码 JSON 时构建自定义期望

要构建自定义期望，我们可以参考前面提到的 `pass` 和 `fail` 函数。`pass` 函数总是通过，而 `fail` 函数会失败并显示一条消息。要生成自定义消息，可以使用 `onFail` 函数。

让我们通过在 `tests` 文件夹内创建一个新文件来查看一个示例。我们将这个新文件命名为 `CustomExpectations.elm` 并添加以下代码：

```js
module CustomExpectations exposing (..)

import Expect exposing (Expectation)
import Json.Decode exposing (decodeString, field, int, list, map2, string)
import Test exposing (..)

type alias Player =
    { name : String
    , language : String
    }

playerDecoder =
    map2 Player
        (field "name" string)
        (field "language" string)

suite : Test
suite =
    describe "A Test Suite"
        [ describe "A custom expectation with a custom decoder"
            [ test "Decoding JSON strings" <|
                \_ ->
                    let
                        json =
                            """
                            {
                            "name" : "John Doe",
                            "language" : "English"
                            }
                            """
                    in
                    case decodeString playerDecoder json of
                        Ok json ->
                            Expect.pass

                        Err err ->
                            Expect.fail err
            ]
        ]
```

如果我们使用命令 `elm-test tests/CustomExpectations.elm` 运行此测试，我们会得到以下输出：

```js
Success! Compiled 0 modules.
Successfully generated /dev/null
Success! Compiled 1 module.
Successfully generated C:\Users\PC\Desktop\elm-web-development\chapter9\weather-app-with-tests-ch9\elm-stuff\generated-code\elm-community\elm-test\elmTestOutput.js

elm-test 0.18.12
----------------

Running 1 test. To reproduce these results, run: elm-test --fuzz 100 --seed 619873355 tests/CustomExpectations.elm

TEST RUN PASSED

Duration: 468 ms
Passed: 1
Failed: 0
```

在代码中，我们在全局作用域中设置类型别名 `Player`，以便在需要时在我们的测试中访问它。`Person` 类型别名是一个具有 `name` 字段的记录，该字段是 `String` 类型，还有一个 `language` 字段，也是 `String` 类型。

我们使用 `describe` 函数来描述我们的测试套件，然后描述第一个 `List` 测试，目前实际上只有一个测试。

让我们看看 `playerDecoder` 函数：

```js
playerDecoder =
    map2 Player
        (field "name" string)
        (field "age" string)
```

如我们在第六章中学习的，*深入探索 Elm*，`string` 是一个具有以下签名的解码器：

```js
decoder : Json.Decode.Decoder String
```

因此，`string` 是 `Strings` 的解码器，意味着它将 JSON `strings` 转换为 Elm `Strings`，这个过程将 JSON `strings` 解析为 Elm `Strings` 称为 *解码*。

表达式 `field "name" string` 也是一个 `Strings` 的解码器，整个 `playerDecoder` 是一个由 `field "name" string` 解码器和 `field "age" string` 解码器组成的复杂解码器。

在匿名函数内部，我们提供 `let-in` 表达式。在 `let-in` 表达式的 `let` 部分，我们定义我们的 `json` 字符串。

最后，在 `let-in` 表达式的 `in` 部分，我们评估一个 `case-of` 表达式，并借助 `pass` 和 `fail` 函数，我们编写我们的自定义期望。

接下来，我们将探讨模糊测试。

# 在 Elm 中编写模糊测试

模糊测试允许我们通过在测试中随机化一些输入值，从而运行任意数量的值组合，将我们的单元测试提升到新的水平。

为了了解模糊测试是如何工作的，让我们比较单元测试和模糊测试。

让我们从本章前面已经使用过的常规单元测试开始。为了刷新我们对这个测试的记忆，让我们将其添加为 `Example.elm` 的内容，然后运行实际的测试：

```js
module Example exposing (..)

-- import Fuzz exposing (Fuzzer, int, list, string)

import Expect exposing (Expectation)
import Test exposing (..)

suite : Test
suite =
    describe "Zero is equal to zero"
        [ test "Zero is equal to one minus one" <|
            \_ -> Expect.equal 0 (1 - 1)
        , test "Zero is equal to two minus two" <|
            \_ -> Expect.equal 0 (2 - 2)
        ]
```

注意，在这段代码中，Fuzz 导入被注释掉了，因为编译器会给出关于此导入未使用的警告。

如果我们运行这个测试，它将在控制台输出一个成功的测试消息，显示两个测试通过，零个失败。

尽管这个例子非常简单，但它将帮助我们比较 Elm 中的单元测试和模糊测试。我们列出两个测试的事实本身就暗示了模糊测试的有用性。

如果我们想确保 `Expect.equal` 对于任何减去自身的数字都会返回零，我们可能需要编写多个测试来确认这种行为。

然而，编写模糊测试将是一个更好的解决方案。让我们看看更新后的代码：

```js
module Example exposing (..)

-- import Fuzz exposing (Fuzzer, int, list, string)

import Expect exposing (Expectation)
import Test exposing (..)

suite : Test
suite =
    describe "Zero is equal to zero"
        [ test "Zero is equal to one minus one" <|
            \_ -> Expect.equal 0 (1 - 1)
        , test "Zero is equal to two minus two" <|
            \_ -> Expect.equal 0 (2 - 2)
        ]
```

现在，让我们将这个单元测试重写为模糊测试的形式。

我们将取消注释 Fuzz 导入，并在我们的测试中随机化一些输入，如下所示：

```js
module Example exposing (..)

import Expect exposing (Expectation)
import Fuzz exposing (Fuzzer, int, list, string)
import Test exposing (..)

suite : Test
suite =
    describe "Zero is equal to zero"
        [ fuzz int "Zero is equal to random number minus itself" <|
            \randomNumber -> Expect.equal 0 (randomNumber - randomNumber)
        ]
```

这个测试将运行 100 次。这使得我们比编写普通的单元测试更有信心地得出通过测试的结论。

通过查看我们的模糊测试语法的结构，我们可以看到模糊函数接受三个参数：

+   一个 `fuzzer`

+   一个用于描述模糊测试的 `String`

+   一个匿名函数（与单元测试不同，它接受一个实际参数）

因此，这个模糊测试的模糊器用于生成 `int` 值。`int` 模糊器生成随机的 `Ints`。用于描述模糊测试的 `String` 是直接的。我们使用的是读取为“零等于随机数减去自身”的 `String`。最后，我们的匿名函数 `\` 接收我们命名为 `randomNumber` 的参数。由于我们在实际的 `Expect.equal` 断言中使用 `randomNumber`，将此参数传递给匿名函数是完美的。

# 与多个模糊器一起工作

类似于我们可以使用 `map2`、`map3`、`map4` 等等，根据我们从 JSON 值解码到 Elm 值的字段数量，我们还有以下函数可供使用：`fuzz2`、`fuzz3`、`fuzz4` 和 `fuzz5`。

让我们更新我们的 `Example.elm` 测试，使其使用 `fuzz2` 函数。为了简化，我们将测试数字加法的交换律。这是数学中的规则，即如果我们交换相加的数字的顺序，结果将相同：

```js
module Example exposing (..)

import Expect exposing (Expectation)
import Fuzz exposing (Fuzzer, int, list, string)
import Test exposing (..)

suite : Test
suite =
    describe "Zero is equal to zero"
      [ fuzz2 int int "Swiching positions on two numbers being added will still yield the same result" <|
          \randomNumber1 randomNumber2 -> 
            Expect.equal (randomNumber1 + randomNumber2) (randomNumber2 + randomNumber1)
      ]
```

所有 100 个测试都如预期通过。重要的是要注意，控制台输出将只列出通过的一个测试，因为实际上就是这样。确实只有一个测试在运行，尽管有 100 种不同的值组合。

# 摘要

在本章中，我们学习了 Elm 中单元测试和模糊测试的基础。我们了解了 Elm 测试的结构和语法：

+   我们了解了 `describe`、`Expect.equals` 和 `test`

+   我们讨论了如何在 Elm 测试中使用 `\_` 和 `<|`

+   我们编写了将 JSON 字符串解码为 Elm 值的测试

+   我们研究了单元测试和模糊测试之间的区别

+   我们编写了接受多个参数的模糊测试

然而，本章所涵盖的内容仅仅是开始。TDD（测试驱动开发）是一种构建软件的方法，它在整个应用程序的生命周期中拥抱测试；实际上，正如其名称所暗示的，它是一种测试驱动整个开发过程的方法。通过本课程所涵盖的主题，你应该能够开始在 Elm 开发中例行实施测试。

在下一章中，我们将探讨 Elm 应用程序中的身份验证。
