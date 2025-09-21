# 第七章：在 Elm 中制作天气应用

欢迎来到第七章，*在 Elm 中制作天气应用*。在本章中，我们将制作我们的 Elm 天气应用。这个应用的目的在于学习如何从 JSON 中获取信息并将其用于我们的应用中。

我们将涵盖的主题包括：

+   使用`Result`处理错误

+   使用`Maybe`处理可选值和空值

+   使用解码器解码 JSON 字符串

+   使用 HTTP 包帮助获取远程数据

+   与第三方 API 合作

完成这一章后，你将能够：

+   以 JSON 格式从互联网获取信息

+   让你的 Elm 应用使用解码器消费 JSON 数据

+   理解所有部件如何组合在一起以构建一个功能性的应用

首先，我们将探讨从第三方 API 获取天气数据。

# 从第三方 API 获取天气数据

为了使我们的天气应用成为可能，我们需要从某处获取天气数据。幸运的是，网上有大量的天气相关 API，这使得这项任务变得容易得多。

为了有一个工作的应用，我们需要连接到第三方数据提供商。在我们的案例中，我们将使用的数据提供商是 Open Weather Map。

Open Weather Map 的 API 信息可在[`openweathermap.org/api`](https://openweathermap.org/api)找到。

要访问 API，你需要获取 APPID。如何操作的说明可在[`openweathermap.org/appid`](http://openweathermap.org/appid)找到。基本上，获取它的所有你需要做的就是创建一个新账户。

一旦你创建了账户，你将在 API keys 标签下找到生成的 API 密钥。例如，要获取芝加哥的数据，只需访问以下 URL：`http://api.openweathermap.org/data/2.5/weather?q=chicago&APPID=abcdef1234567890`

显然，前面的 URL 使用了错误的 APPID。将其替换为你的自己的，你应该就可以正常使用了。使用正确的 APPID 在前面 URL 中，你的浏览器将显示一个如下的 JSON 字符串：

```js
{"coord":{"lon":-87.62,"lat":41.88},"weather":[{"id":802,"main":"Clouds","description":"scattered clouds","icon":"03n"}],"base":"stations","main":{"temp":268.9,"pressure":1025,"humidity":86,"temp_min":267.15,"temp_max":270.15},"visibility":16093,"wind":{"speed":2.1,"deg":260},"clouds":{"all":40},"dt":1518941700,"sys":{"type":1,"id":1030,"message":0.0033,"country":"US","sunrise":1518957687,"sunset":1518996470},"id":4887398,"name":"Chicago","cod":200}
```

我们将如何处理这个字符串将在本章后面讨论。

# 我们将要构建什么？

我们将构建一个非常简单的天气应用，它将与外部世界通信以获取 JSON 格式的天气数据。

一旦我们收到天气数据，我们将使用解码器将 JSON 字符串解码成我们的应用模型能够理解和处理的值。让我们直接进入正题。

# 构建我们的天气应用

我们将首先用之前章节中已经使用过的裸骨 Elm 代码替换`Main.elm`中的代码：

```js
module Main exposing (..)

import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)

-- Model
init =
    {}

-- Update
type Msg
    = Nothing

update msg model =
    model

-- View
view model =
    div [] [ text "Everything will go here" ]

-- Main
main =
    beginnerProgram
        { model = init
        , view = view
        , update = update
        }
```

让我们立即通过将`beginnerProgram`替换为`Html.Program`来改进我们的应用，因为，正如前一章所解释的，`beginnerProgram`没有处理副作用的方法：

```js
module Main exposing (..)
import Html exposing (..)

-- Model
type alias Model =
    {}

init : ( Model, Cmd Msg )
init =
    ( Model, Cmd.none )

-- Update
type Msg
    = Noop

update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        Noop ->
            ( model, Cmd.none )

-- View
view : Model -> Html Msg
view model =
    div [] [ text "Everything will go here, this time using Html.program" ]

-- Subscriptions
subscriptions : Model -> Sub Msg
subscriptions model =
    Sub.none

-- Main
main : Program Never Model Msg
main =
    Html.program
        { init = init
        , view = view
        , update = update
        , subscriptions = subscriptions
        }
```

让我们回顾一下前面的代码。

我们首先将应用程序模型描述为一个名为`Model`的类型别名，并将其设置为空记录。在类型别名之后，我们引入`init`函数，它包含我们应用的初始状态和要运行的初始命令。通过`Cmd.none`，我们基本上是在告诉 Elm 运行时不应该在这个时候运行任何命令。所以，尽管我们此刻不会请求运行任何命令，我们不能简单地省略它们，我们必须明确地声明我们目前不会运行任何命令。

注意`init`函数的变化：现在的`init`函数接受以下两个元素的元组：`( Model, Cmd.none )`。

接下来，在`update`函数中，我们使用一个联合类型`Msg`，它只能是`NoOp`值。基本上，每当我们要断言我们的应用不应该对收到的消息做任何事情时，我们都可以使用`NoOp`作为值。

更新函数使用`case of`表达式。我们向`update`函数传递两个参数：`msg`和`model`，这些参数用于更新`model`并返回一个命令。在这里，由于我们收到的是`NoOp`消息，唯一发生的事情就是：我们只返回`model`本身。通过在返回的两个元素元组中使用`Cmd.none`，我们告诉 Elm 运行时不应该运行任何命令。

在`subscriptions`函数中，我们所做的一切就是：我们告诉 Elm 运行时我们不想订阅任何事件。因此，我们将其赋值为`Sub.none`。最后，`main`函数被赋值为`Html.program`，所有上述内容都汇集在一起。

在下一节中，我们将安装 HTTP 包。

# 安装 HTTP 包

要安装 HTTP 包，我们需要将我们的控制台指向`weather app`文件夹的位置。接下来，在控制台中运行以下命令：

```js
elm package install elm-lang/http 1.0.0
```

这就是控制台将返回的内容：

```js
To install elm-lang/http I would like to add the following
dependency to elm-package.json:

 "elm-lang/http": "1.0.0 <= v < 2.0.0"

May I add that to elm-package.json for you? [Y/n]
```

在确认`y`后，控制台将输出以下内容：

```js
Some new packages are needed. Here is the upgrade plan.

 Install:
 elm-lang/http 1.0.0

Do you approve of this plan? [Y/n]
```

通过输入`y`批准计划将在控制台产生以下消息：

```js
Starting downloads...

 ● elm-lang/http 1.0.0

Packages configured successfully!
```

接下来，让我们打开`elm-package.json`并验证它看起来如下：

```js
{
    "version": "1.0.0",
    "summary": "helpful summary of your project, less than 80 characters",
    "repository": "https://github.com/user/project.git",
    "license": "BSD3",
    "source-directories": [
        "src"
    ],
    "exposed-modules": [],
    "dependencies": {
        "elm-lang/core": "5.1.1 <= v < 6.0.0",
        "elm-lang/html": "2.0.0 <= v < 3.0.0",
        "elm-lang/http": "1.0.0 <= v < 2.0.0"
    },
    "elm-version": "0.18.0 <= v < 0.19.0"
}
```

通过这种方式，我们已经成功地将 HTTP 包添加到我们的项目中。

HTTP 包的官方文档可在[`package.elm-lang.org/packages/elm-lang/http/1.0.0/Http`](http://package.elm-lang.org/packages/elm-lang/http/1.0.0/Http)找到。

现在我们已经添加了所有必要的包，我们可以继续构建我们的应用。

# 添加所有导入

让我们从在`Main.elm`的顶部添加所有需要的导入开始：

```js
module Main exposing (..)

import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)
import Http
import Json.Decode as Decode
```

在您使用前面的更新保存代码后，代码检查器将抛出四个相同的警告。每个警告中唯一的区别是引用的模块。例如：

```js
Module `Html.Attributes` is unused.
Best to remove it. Don't save code quality for later!
```

编译器鼓励我们在第一个可能的机会改进我们的代码质量真是太好了，但就目前而言，忽略这些警告是安全的。我们很快就会添加到我们的应用中的代码来解决这些问题。

# 概念化模型

由于我们将在我们的应用程序中更改很多东西，在这个时候，我们可以通过简单地临时移除类型注解来防止几个令人烦恼的错误。为此，只需从以下函数中移除类型注解：`init`、`update`、`view`、`subscriptions`和`main`。

现在我们来看看我们的更新后的模型：

```js
type alias Model =
    { temperatureInfo : TemperatureInfo
    , city : String
    }

type alias TemperatureInfo =
    { name : String
    , windSpeed : Float
    , temperature : Float
    , pressure : Float
    , humidity : Float
    }
```

我们模型的基本构建块应该是自解释的。然后我们将前面的代码添加到`Main.elm`中，并保存文件。我们的应用程序编译得很好，一切正常。现在让我们更新`init`函数：

```js
init =
    ( Model (TemperatureInfo "Did not load" 0 0 0 0) ""
    , Cmd.none
    )
```

我们在这里所做的是，简单地将模型中引入的新变化反映到我们的`init`函数中。在下一节中，我们将设置`Msg`和`update`函数。

# 设置 Msg 联合类型

我们的`Msg`联合类型相当简单：

```js
type Msg
    = GetTemp
    | CityInput String
    | NewTemp (Result Http.Error String)
```

`update`函数可以接收三种可能的消息：`GetTemp`、`CityInput`和`NewTemp`。其中最特别的是第三个：

```js
NewTemp (Result Http.Error String)
```

那里有一些我们之前没见过的代码。`NewTemp`值包含一个`Result`。这个结果可以是一个`Http.Error`，这意味着某些原因导致我们的 HTTP 请求失败。可能的原因有很多：URL 不存在，或者服务器没有响应，等等。还有可能发生的情况是：请求可以成功。

在那种情况下，我们将得到成功 HTTP 请求的结果`String`。

# Result 和 Maybe

这是个好时机，让我们更详细地讨论一下`Result`和`Maybe`。为了处理错误，Elm 也有`Tasks`，但我们现在不会讨论它们。

# 使用 Result

`Result`的官方文档可以在以下网址找到：[`package.elm-lang.org/packages/elm-lang/core/latest/Result`](http://package.elm-lang.org/packages/elm-lang/core/latest/Result)。每当我们的代码执行可能失败的操作时，我们都会使用`Result`。使用`Result`可以让我们更好地处理 Elm 中的错误。`Result`的定义表明它是一个联合类型，有两个类型构造函数：`Ok`和`Err`。看看下面的代码片段：

```js
type Result error value
    = Ok value
    | Err error
```

如果一个操作成功，`Result`就是`Ok`。否则，`Result`就是`Err`。

要测试驱动`Result`，我们只需要运行一个可能失败的操作。REPL 是进行此类测试的完美场所。因此，让我们将浏览器指向[`elmrepl.cuberoot.in/`](http://elmrepl.cuberoot.in/)并运行以下代码：

```js
import Date
Date.fromString "2018-02-18"
```

在 REPL 中运行前面的代码，将返回以下内容：

```js
Ok <Sun Feb 18 2018 00:00:00 GMT+0000 (UTC)> : Result.Result String Date.Date
```

太棒了，我们从可能失败的操作中得到了一个`Ok`结果。为了保险起见，我们再试一次：

```js
Date.fromString "0"
```

你可能会认为前面的代码会失败，但实际上它是完全有效的：

```js
Ok <Sat Jan 01 2000 00:00:00 GMT+0000 (UTC)> : Result.Result String Date.Date
```

太好了，我们仍然得到了一个`Ok`。这次，让我们使用一串字母，以确保 REPL 无法处理它：

```js
Date.fromString "abc"
```

事实上，REPL 不知道如何处理这种输入。然而，而不是抛出一个丑陋的异常，我们得到了一个优雅的异常：

```js
Err "Unable to parse 'abc' as a date. Dates must be in the ISO 8601 format."
 : Result.Result String Date.Date
```

为了使观点更加明确，在前面例子中，我们运行了一个可能失败的表达式，即我们运行了 `Date.fromString` 函数。`Date.fromString` 函数接受一个类型为 `String` 的参数，并返回一个 `Date`。在前面代码片段中，我们给它了一个值为 `abc` 的字符串，它返回了一个 `Err` 作为 `Result`。

`Result` 有它自己的模块，这就是为什么类型定义读作 `Result.Result`。

接下来，我们将探讨如何使用 `Maybe`。

# 与 Maybe 一起工作

也是一个联合类型，`Maybe` 是 Elm 中处理空值的一种方式，或者说，处理可选值，即可能存在，也可能不存在。官方文档可在 [`package.elm-lang.org/packages/elm-lang/core/latest/Maybe`](http://package.elm-lang.org/packages/elm-lang/core/latest/Maybe) 找到，它给出了 `Maybe` 联合类型的定义如下：

```js
type Maybe a
  = Just a
  | Nothing
```

如我们在书中前面讨论的，在 Elm 中使用字母 `a` 是一种约定，它表示在该位置使用的值可以是任何东西。因此，前面定义中的 `a` 可以是 `String`，`Int` 或任何其他值。

因此，在 Elm 中，`Maybe` 只能是两种情况之一——`Just` 任何东西，或者 `Nothing`。更具体地说，`Maybe` 可以是以下之一：

+   `Just` 一个 `String`，或者 `Just` 一个 `Int`，或者 `Just` 一个 `Float`...，

+   `Nothing`

现在，让我们转向 Elm REPL [`elmrepl.cuberoot.in/`](http://elmrepl.cuberoot.in/) 并看看 `Maybe` 类型在实际中的应用：

```js
testingMaybe = Just 1
```

REPL 将返回以下内容：

```js
Just 1 : Maybe.Maybe number
```

如我们所见，`Just 1` 是一个 `Maybe` 数字。让我们再做一个例子：

```js
testingMaybe = Just 1.1
```

执行上一行将得到以下结果：

```js
Just 1.1 : Maybe.Maybe Float
```

最后，让我们看看 `Nothing` 的实际应用：

```js
testingMaybe = Nothing
```

REPL 回复如下：

```js
Nothing : Maybe.Maybe a
```

由于 `Nothing` 没有值，我们回到了 `Maybe a`。Elm 保证要求我们在这里有一个 `a`，即使 `Nothing` 是 `Nothing`，因为它代表值的缺失，它不需要为其值指定类型。

要解构 `Maybe`，我们可以使用 `case-of` 表达式。让我们用一个例子来看看 Ellie：

```js
module Main exposing (main)

import Html exposing (Html, text)

-- A person, but maybe we do not know their age.
type alias Person =
    { name : String
    , age : Maybe String
    }
tom = { name = "Tom", age = Just "42" }
sue = { name = "Sue", age = Nothing }

main =
    text <| 
    case tom.age of 
        Nothing ->
            "No age for this person"
        Just val ->
            val
```

在前面的例子中，我们使用的是 `Maybe` 官方文档中提供的稍作修改的人的例子。前面代码中的区别在于，我们不是使用 `Int`，而是使用 `String` 作为记录中的 `age` 条目。这样，我们可以避免使我们的代码比必需的更复杂，并且我们仍然从 `case-of` 表达式的所有分支中返回 `String`。

应用程序在编译时将打印出 `42`。现在，让我们修改应用程序，以便尝试打印 Sue 的年龄。需要做的唯一区别如下：

```js
case sue.age of
```

这次，在编译时，应用程序将匹配 `main` 中的 `case-of` 表达式的 `Nothing` 分支，这将导致屏幕上打印出 `"No age for this person"` 消息。

# 结果和带有默认值的 Maybe

`Result`和`Maybe`非常相似。如果我们的操作成功，我们通过`Result`得到`Ok a`。当处理`Maybe`时，如果存在值，我们得到`Just a`。

如果发生错误，对于`Result`我们得到`Err error`。在`Maybe`中没有值的情况下，我们得到`Nothing`。我们已经在本书前面的章节中（在第四章[5c3a6b83-d672-49a1-9da9-355bb415b8c4.xhtml]，*在 Elm 中准备一个单位转换网站*）查看过使用`Result.withDefault`。让我们通过在 Elm REPL 中运行以下代码来快速回顾：

```js
Result.withDefault 0 (Ok 1)
```

REPL 响应如下：

```js
1 : number
```

现在我们尝试使用`Strings`：

```js
Result.withDefault "This is default." (Ok "This is OK")
```

REPL 返回以下内容：

```js
"This is OK" : String
```

与`Result`类似，我们也可以在`Maybe`上使用`withDefault`。在 REPL 中运行以下代码：

```js
Maybe.withDefault 0 Nothing
```

REPL 返回以下内容：

```js
0 : number
```

让我们再来几个。首先，给它一个默认的`String`：

```js
Maybe.withDefault "abc" Nothing
```

REPL 返回以下内容：

```js
"abc" : String
```

那么一个默认的`Record`呢？

```js
Maybe.withDefault {} Nothing
```

在 REPL 中运行前面的代码将产生以下内容：

```js
{} : {}
```

在下一节中，我们将处理我们的`update`函数。

# 更新`update`函数

我们的`update`函数将更加复杂。首先，我们需要在我们的`update`函数中涵盖所有前面的消息。我们将通过添加一个`case-of`表达式来完成：

```js
update msg model =
    case msg of
        GetTemp ->
            ( model, getTemperature model.city )

        NewTemp (Ok json) ->
            let
                newTemperatureInfo =
                    decodeTemperatureInfo json
            in
            ( { model | temperatureInfo = newTemperatureInfo }, Cmd.none )

        NewTemp (Err _) ->
            ( model, Cmd.none )

        CityInput city ->
            ( { model | city = city }, Cmd.none )
```

在前面的代码片段中，我们可以看到`update`函数的四种可能情况。如果我们的`update`函数收到`GetTemp`消息，它将返回一个包含`model`和`getTemperature`值的二元组。

如果`update`函数收到`NewTemp (Ok json)`消息，即如果我们从远程服务器成功接收 JSON 字符串，那么我们使用`let-in`表达式返回相同的`model`，并更新`newTemperatureInfo`。

第三个要匹配的模式是`NewTemp (Err _)`。这个模式将在我们收到远程服务器的错误时匹配，在这种情况下，我们将只返回现有的模型，即`(model, Cmd.none)`。

在`update`函数中我们可以进行模式匹配的最后一个可能的消息是`CityInput`消息。如果我们收到一个`CityInput`消息，我们将取传递给它的`city`字符串，并返回现有的模型，加上新的`city`字符串。

然而，如果我们现在运行我们的应用程序，编译器将抛出两个错误：

+   找不到变量`getTemperature`

+   找不到变量`decodeTemperatureInfo`

显然，将复杂性抽象到单独的变量中是很好的，可以使我们更容易地推理我们的应用程序，但现在我们需要实际实现`getTemperature`和`decodeTemperatureInfo`，以便使我们的`update`函数工作。但在我们能够这样做之前，我们需要更详细地查看解码器和编码器。

# 解码器和编码器

为了从 JSON 解析数据，Elm 使用解码器。为了做相反的操作，Elm 使用编码器。使用解码器和编码器，我们可以将动态类型 JSON 数据转换为静态类型 Elm 数据结构，反之亦然。

首先，让我们看看 Elm 语言核心部分可用的 `Json.Decode` 包。`Json.Decode` 的官方文档可在：[`package.elm-lang.org/packages/elm-lang/core/5.1.1/Json-Decode`](http://package.elm-lang.org/packages/elm-lang/core/5.1.1/Json-Decode) 找到。官方文档定义解码器如下：

一个知道如何解码 JSON 值的值。

# 解码原始值

让我们通过 Elm REPL 的帮助来查看解码原始值的一些示例。首先，让我们解码一个简单的 `Int`：

```js
import Json.Decode exposing (..)
decodeString int "100"
```

REPL 返回以下内容：

```js
Ok 100 : Result.Result String Int
```

因此，我们得到了一个 `Result`，这是表示一个可能失败的操作的标志。让我们通过向前面的表达式提供一个 `Bool` 而不是 `Int` 来看看它如何失败：

```js
decodeString int "true"
```

运行前面的代码会导致 REPL 返回以下内容：

```js
Err "Expecting an Int but instead got: true" : Result.Result String Int
```

接下来，让我们解码一个 JSON `float` 到 Elm 的 `Float`：

```js
decodeString float "1.234"
```

REPL 返回以下内容：

```js
Ok 1.234 : Result.Result String Float
```

那么将 JSON `string` 解码为 Elm 的 `String` 呢？

```js
decodeString string "abcd"
```

这次，REPL 响应如下：

```js
Err "Given an invalid JSON: Unexpected token a" : Result.Result String String
```

为什么会出现这个错误？因为我们还需要提供转义引号，以便使其工作：

```js
decodeString string "\"abcd\""
```

现在 REPL 不再抱怨了：

```js
Ok "abcd" : Result.Result String String
```

最后，让我们看看如何将 JSON 布尔值解码为 Elm 的 `Bool`：

```js
decodeString bool "false"
```

REPL 返回以下内容：

```js
Ok False : Result.Result String Bool
```

让我们检查每个解码器的签名：

```js
import Json.Decode exposing (..)
int
<decoder> : Json.Decode.Decoder Int
float
<decoder> : Json.Decode.Decoder Float
string
<decoder> : Json.Decode.Decoder String
bool
<decoder> : Json.Decode.Decoder Bool
```

如前代码片段所示，`int` 是 `Int` 的解码器，`float` 是 `Float` 的解码器，`string` 是 `String` 的解码器，而 `bool` 是 `Bool` 的解码器。现在我们了解了如何从 JSON 解码原始值到 Elm，让我们看看如何解码一个更复杂的 JSON 字符串，即本章开头从 API 得到的那个。

# 解码 API 返回的 JSON 字符串

首先，让我们看看我们从 API 得到的 JSON 字符串：

```js
{  
   "coord":{  
      "lon":-87.62,
      "lat":41.88
   },
   "weather":[  
      {  
         "id":802,
         "main":"Clouds",
         "description":"scattered clouds",
         "icon":"03n"
      }
   ],
   "base":"stations",
   "main":{  
      "temp":268.9,
      "pressure":1025,
      "humidity":86,
      "temp_min":267.15,
      "temp_max":270.15
   },
   "visibility":16093,
   "wind":{  
      "speed":2.1,
      "deg":260
   },
   "clouds":{  
      "all":40
   },
   "dt":1518941700,
   "sys":{  
      "type":1,
      "id":1030,
      "message":0.0033,
      "country":"US",
      "sunrise":1518957687,
      "sunset":1518996470
   },
   "id":4887398,
   "name":"Chicago",
   "cod":200
}
```

显然，我们已经格式化 JSON 字符串，使其更容易阅读。首先要注意的是；我们不必解码所有前面的代码。我们可以选择只解码我们需要的部分数据。所以，让我们假设我们唯一感兴趣的数据是倒数第二个键值对，即：`''name'':''Chicago''`。

我们如何从前面那块 JSON 中解码出来？我们将创建一个小 Ellie 应用来查看如何实现这一点：

```js
import Html exposing (..)
import Json.Decode exposing (..)

main = text (toString (decodeString weatherDataDecoder (json)))

type alias WeatherData =
 { name : String }

json = """
{ 
 "coord":{ 
 "lon":-87.62,
 "lat":41.88
 },
 "weather":[ 
 { 
 "id":802,
 "main":"Clouds",
 "description":"scattered clouds",
 "icon":"03n"
 }
 ],
 "base":"stations",
 "main":{ 
 "temp":268.9,
 "pressure":1025,
 "humidity":86,
 "temp_min":267.15,
 "temp_max":270.15
 },
 "visibility":16093,
 "wind":{ 
 "speed":2.1,
 "deg":260
 },
 "clouds":{ 
 "all":40
 },
 "dt":1518941700,
 "sys":{ 
 "type":1,
 "id":1030,
 "message":0.0033,
 "country":"US",
 "sunrise":1518957687,
 "sunset":1518996470
 },
 "id":4887398,
 "name":"Chicago",
 "cod":200
}
"""

weatherDataDecoder =
 Json.Decode.map
 WeatherData
 (field "name" string)
```

前面的这个小应用成功解码了提供的 JSON 字符串。正如我们所见，我们只解码了 JSON 对象中的 `name` 键，其值根据前面提供的 JSON 对象是 `Chicago`。在 Ellie 编辑器中编译应用的结果显示在屏幕上：

```js
Ok { name = "Chicago" }
```

让我们看看我们如何改进前面的应用。这次，我们不仅想从我们的 JSON 对象中返回 `name` 键，还想返回 `id` 键：

```js
import Html exposing (..)
import Json.Decode exposing (..)

main = text (toString (decodeString weatherDataDecoder (json)))

type alias WeatherData =
  { id : Int 
  , name : String 
  }

json = """
{ 
   "coord":{ 
      "lon":-87.62,
      "lat":41.88
   },
   "weather":[ 
      { 
         "id":802,
         "main":"Clouds",
         "description":"scattered clouds",
         "icon":"03n"
      }
   ],
   "base":"stations",
   "main":{ 
      "temp":268.9,
      "pressure":1025,
      "humidity":86,
      "temp_min":267.15,
      "temp_max":270.15
   },
   "visibility":16093,
   "wind":{ 
      "speed":2.1,
      "deg":260
   },
   "clouds":{ 
      "all":40
   },
   "dt":1518941700,
   "sys":{ 
      "type":1,
      "id":1030,
      "message":0.0033,
      "country":"US",
      "sunrise":1518957687,
      "sunset":1518996470
   },
   "id":4887398,
   "name":"Chicago",
   "cod":200
}
"""

weatherDataDecoder =
  Json.Decode.map2
    WeatherData
    (field "id" int)
    (field "name" string)
```

如果我们现在编译应用，我们会得到一个与上次略有不同的结果：

```js
Ok { id = 4887398, name = "Chicago" }
```

在先前的应用程序中，有一些细微的变化，这使得我们可以提取两个值，而不是像我们之前的小型 JSON 解码应用程序版本中那样只提取一个值。让我们看看这些变化：

```js
type alias WeatherData =
  { id : Int 
  , name : String 
  }
```

在`WeatherData`类型别名中，我们在`Record`中添加了一个`id`类型为`Int`：

```js
weatherDataDecoder =
  Json.Decode.map2
    WeatherData
    (field "id" int)
    (field "name" string)
```

在解码器中，我们使用`map2`函数而不是`map`函数，并添加另一个带有`id`键的 JSON 字段，以及预期的 JSON 整数值。类似于我们所看到的，如果我们想要从我们的 JSON 对象中映射另一个值，我们需要使用`map3`函数。

# 解码嵌套对象

假设我们想要从返回的 JSON 中获取国家信息。我们需要对我们的解码器进行简单的更改：而不是使用`field`，我们将使用`at`。虽然`field`使用一个字符串，但`at`使用一个字符串列表，这允许我们进入嵌套对象的内部结构。让我们更新我们的 Ellie 应用程序的`weatherDataDecoder`，使其看起来如下：

```js
weatherDataDecoder =
  Json.Decode.map3
    WeatherData
    (field "id" int)
    (field "name" string)
    (at ["sys", "country"] string)
```

让我们也更新`WeatherData`类型别名：

```js
type alias WeatherData =
  { id : Int 
  , name : String 
  , country: String
  }
```

注意，在先前的代码片段中，我们保持了我们扁平的`Record`结构，并且它并不反映我们从其中获取数据的 JSON 对象。这是完全可以接受的。在编译后，应用程序将以下结果打印到屏幕上：

```js
Ok { id = 4887398, name = "Chicago", country = "US" }
```

现在我们已经了解了解码器的工作原理，并在解码器上练习了一段时间后，理解下一节中我们将看到的代码应该会容易得多。

# 添加 getTemperature 和 decodeTemperatureInfo

我们的`getTemperature`是一个简单的`let-in`表达式，它将使用`Http.send`和`Http.getString`发送一个新的请求，以便从自定义 URL 获取天气数据。从自定义 URL 获取数据的字符串，取决于`city`变量，即我们从应用程序文本输入字段获取的用户输入值：

```js
getTemperature city =
    let
        url =
            "http://api.openweathermap.org/data/2.5/weather?q=" ++ city ++ "&APPID=b30c7031a64e301cb64ceaa346e24a83"
    in
    Http.send NewTemp (Http.getString url)
```

`decodeTemperatureInfo`也是一个`let-in`表达式，虽然乍一看可能有点令人畏惧，但实际上它只是尝试解码 JSON 数据的一系列重复操作：

```js
decodeTemperatureInfo : String -> TemperatureInfo
decodeTemperatureInfo json =
    let
        name =
            Result.withDefault "Error decoding data!" (Decode.decodeString (Decode.field "name" Decode.string) json)

        windSpeed =
            Result.withDefault 0 (Decode.decodeString (Decode.at [ "wind", "speed" ] Decode.float) json)

        temperature =
            Result.withDefault 0 (Decode.decodeString (Decode.at [ "main", "temp" ] Decode.float) json)

        pressure =
            Result.withDefault 0 (Decode.decodeString (Decode.at [ "main", "pressure" ] Decode.float) json)

        humidity =
            Result.withDefault 0 (Decode.decodeString (Decode.at [ "main", "humidity" ] Decode.float) json)
    in
    TemperatureInfo name windSpeed temperature pressure humidity
```

基本上，在先前的代码中发生的事情是——我们正在逐步解码从远程服务器接收到的 JSON 字符串。在这个阶段，保存并运行我们的应用程序，我们会看到与前完全相同的窗口：所有内容都将在这里，这次使用 Html.program。

这是一个好兆头。这意味着我们的应用程序编译成功。这也意味着在`view`和`main`函数中还有更多的工作要做。

# 更新视图

我们的`view`函数是用来给我们模型的可视表示的。为了使我们的应用程序工作，`view`函数需要一个输入字段，提交按钮，以及一些在联系远程服务器并解码 JSON 字符串时需要填充的值：

```js
view model =
    div []
        [ input [ placeholder "City", onInput CityInput ] []
        , br [] []
        , button [ onClick GetTemp ] [ text "Get temperature" ]
        , br [] []
        , div [] [ text "Name: ", text model.temperatureInfo.name ]
        , div [] [ text "Temp: ", text (toString model.temperatureInfo.temperature) ]
        , div [] [ text "Wind: ", text (toString model.temperatureInfo.windSpeed) ]
        , div [] [ text "Pressure: ", text (toString model.temperatureInfo.pressure) ]
        , div [] [ text "Humidity: ", text (toString model.temperatureInfo.humidity) ]
        ]
```

让我们看看前面的代码是如何工作的。我们首先使用包裹`div`。接下来，我们有`input`函数，它将在文本输入字段中显示占位符单词`City`。我们还有一个消息，即这个`input`函数将发出一个`CityInput`类型的消息。

接下来，我们有`br`函数，然后是`button`函数，它将在`onClick`时发出`GetTemp`消息。下一个重要的函数是`div`函数，它将包含字符串`''Name: ''`和`model.temperatureInfo.name`的值。同样，我们接着使用其他`div`函数，这些函数根据当前`model`中包含的值拼接字符串。

保存所有内容并运行你的应用。结果将是一个完全工作的天气应用，它从远程服务器获取 JSON 字符串并正确显示结果。要使应用获取数据，只需在输入字段中输入一个主要城市名称，然后点击按钮。

在结束本节之前，让我们看看完整的天气应用代码：

```js
module Main exposing (..)

import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)
import Http
import Json.Decode as Decode

main =
    Html.program
        { init = init
        , view = view
        , update = update
        , subscriptions = subscriptions
        }

type alias Model =
    { temperatureInfo : TemperatureInfo
    , city : String
    }

type alias TemperatureInfo =
    { name : String
    , windSpeed : Float
    , temperature : Float
    , pressure : Float
    , humidity : Float
    }

init : ( Model, Cmd Msg )
init =
    ( Model (TemperatureInfo "Did not load" 0 0 0 0) ""
    , Cmd.none
    )

type Msg
    = GetTemp
    | CityInput String
    | NewTemp (Result Http.Error String)

update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        GetTemp ->
            ( model, getTemperature model.city )

        NewTemp (Ok json) ->
            let
                newTemperatureInfo =
                    decodeTemperatureInfo json
            in
                ( { model | temperatureInfo = newTemperatureInfo }, Cmd.none )

        NewTemp (Err _) ->
            ( model, Cmd.none )

        CityInput city ->
            ( { model | city = city }, Cmd.none )

view : Model -> Html Msg
view model =
    div []
        [ input [ placeholder "City", onInput CityInput ] []
        , br [] []
        , button [ onClick GetTemp ] [ text "Get temperature" ]
        , br [] []
        , div [] [ text "Name: ", text (model.temperatureInfo.name) ]
        , div [] [ text "Temp: ", text (toString model.temperatureInfo.temperature) ]
        , div [] [ text "Wind: ", text (toString model.temperatureInfo.windSpeed) ]
        , div [] [ text "Pressure: ", text (toString model.temperatureInfo.pressure) ]
        , div [] [ text "Humidity: ", text (toString model.temperatureInfo.humidity) ]
        ]

subscriptions : Model -> Sub Msg
subscriptions model =
    Sub.none

getTemperature : String -> Cmd Msg
getTemperature city =
    let
        url =
            "http://api.openweathermap.org/data/2.5/weather?q=" ++ city ++ "&APPID=b30c7031a64e301cb64ceaa346e24a83"
    in
        Http.send NewTemp (Http.getString url)

decodeTemperatureInfo : String -> TemperatureInfo
decodeTemperatureInfo json =
    let
        name =
            Result.withDefault "Error decoding data!" (Decode.decodeString (Decode.field "name" Decode.string) json)

        windSpeed =
            Result.withDefault 0 (Decode.decodeString (Decode.at [ "wind", "speed" ] Decode.float) json)

        temperature =
            Result.withDefault 0 (Decode.decodeString (Decode.at [ "main", "temp" ] Decode.float) json)

        pressure =
            Result.withDefault 0 (Decode.decodeString (Decode.at [ "main", "pressure" ] Decode.float) json)

        humidity =
            Result.withDefault 0 (Decode.decodeString (Decode.at [ "main", "humidity" ] Decode.float) json)
    in
        TemperatureInfo name windSpeed temperature pressure humidity
```

# 摘要

在本章中，我们介绍了很多新概念，并加强了一些我们在前几章中学到的概念。我们探讨了使用`Result`和`Maybe`来处理可能失败的操作以及处理可能缺失的数据。我们探讨了使用解码器和映射。

我们还探讨了使用`Http`包获取远程 JSON 数据。现在我们已经设置了天气应用的基础，我们将在下一章讨论改进它的方法。
