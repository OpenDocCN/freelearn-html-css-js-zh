# 为天气应用添加更多功能

欢迎来到 第八章，*为天气应用添加更多功能*。本章的目标是通过添加更多功能来改进我们的简单天气应用。

我们将涵盖的主题包括：

+   将 `elm-mdl` 添加到我们的应用中

+   将温度显示从开尔文转换为摄氏度

+   使用 Round 模块

完成本章内容后，你将：

+   能够使用 Elm 的 Material Design 库，`elm-mdl`

+   理解 Round 模块的工作原理

# 导入 Material 和 Round 模块

首先，让我们创建一个新的文件夹，命名为 `chapter8`，将控制台指向这个新创建的文件夹，并运行以下命令：

```js
create-elm-app improved-weather-app-ch8
```

一旦应用准备就绪，只需将我们在 第七章 的末尾所拥有的所有代码，即 *在 Elm 中制作天气应用*，复制到这个新的 `Main.elm` 文件中。这将是我们起点（而不是像在上一章中那样从样板应用开始）。

然而，我们需要解决一个轻微的问题。目前，我们的新应用将无法编译。相反，我们将收到以下警告：

```js
Failed to compile
./src/Main.elm

I cannot find module 'Http'.

Module 'Main' is trying to import it.

Potential problems could be:
 * Misspelled the module name
 * Need to add a source directory or new dependency to elm-package.json
```

问题源于我们没有更新 Elm 包。让我们快速解决这个问题，运行以下命令：

```js
elm package install elm-lang/http 1.0.0
```

在我们批准升级后，控制台将显示以下消息：

```js
Packages configured successfully!
```

现在，我们可以开始为我们 Elm 应用提供服务，并在更新应用时监视变化。要开始提供服务，让我们将控制台指向 `improved-weather-app-ch8` 文件夹，并运行以下命令：

```js
elm-app start 
```

我们将通过使用 `elm-mdl` 模块开始对我们应用进行改进。该模块的预览效果可在 [`debois.github.io/elm-mdl/`](https://debois.github.io/elm-mdl/) 查看，官方文档可在 [`github.com/debois/elm-mdl`](https://github.com/debois/elm-mdl) 找到。

首先，让我们导入我们将要使用的所有依赖项。打开 `Main.elm`，找到代码的开始部分，其中包含导入：

```js
module Main exposing (..)

import Html exposing (..)
import Html.Attributes exposing (..)
import Html.Events exposing (..)
import Http
import Json.Decode as Decode
```

在导入的底部，我们将导入 `elm-mdl` 模块依赖项，然后，在最底部，导入 `Round` 包：

```js
import Material
import Material.Button as Button
import Material.Card as Card
import Material.Color as Color
import Material.Elevation as Elevation
import Material.Options as Options
import Material.Scheme
import Material.Textfield as Textfield
import Material.Typography as Typography
import Round exposing (..)
```

官方的 `elm-mdl` 包也列在 Elm 包网站上，网址为 [`package.elm-lang.org/packages/debois/elm-mdl/latest`](http://package.elm-lang.org/packages/debois/elm-mdl/latest)，而 `Round` 包位于 [`package.elm-lang.org/packages/myrho/elm-round/latest`](http://package.elm-lang.org/packages/myrho/elm-round/latest)。

不幸的是，在这个阶段我们的应用再次无法编译。原因是类似的：我们需要安装缺失的包。我们可以通过查看编译器的错误消息来验证这一点：

```js
Failed to compile
./src/Main.elm

I cannot find module 'Material'.

Module 'Main' is trying to import it.

Potential problems could be:
 * Misspelled the module name
 * Need to add a source directory or new dependency to elm-package.json
```

为了解决这个问题，让我们在控制台运行以下命令：

```js
elm package install debois/elm-mdl
```

接下来，让我们再次尝试运行我们的应用，使用 `elm-app start`。

再次，我们的应用无法编译；这次，我们缺少 `Round` 模块。让我们在控制台中使用以下命令添加它：

```js
elm package install myrho/elm-round
```

在成功安装运行我们的应用所需的依赖项后，我们现在可以通过再次运行 `elm-app start` 来看到它成功编译并被浏览器提供服务。目前，应用看起来与 第七章 的末尾，*在 Elm 中制作天气应用*完全相同。

# 将 elm-mdl 添加到我们的 Model 中

虽然我们的主要功能将保持不变——这意味着它仍然会像前一章一样使用 `Html.program`，但我们不得不对我们的 `Model` 类型别名进行修改。

在 第七章 中，*在 Elm 中制作天气应用*，我们的 `Model` 看起来如下：

```js
type alias Model =
    { temperatureInfo : TemperatureInfo
    , city : String
    }
```

我们需要做的只是向 `Model` 的记录中添加另一个条目，如下所示：

```js
type alias Model =
    { temperatureInfo : TemperatureInfo
    , city : String
    , mdl : Material.Model
    }
```

这个新条目 `mdl : Material.Model` 是包含显示 `mdl` 元素所需所有数据的类型。我们的 `TemperatureInfo` 类型别名将保持不变。

# 更新 init 函数

我们需要更新 `init` 函数，以便考虑到 `Material.model` 的初始值。我们在 第七章 的 *在 Elm 中制作天气应用* 中得到的 `init` 函数如下所示：

```js
init : ( Model, Cmd Msg )
init =
    ( Model (TemperatureInfo "Did not load" 0 0 0 0) ""
    , Cmd.none
    )
```

更新很简单，它将反映我们对 `Model` 类型别名所做的更改：

```js
init : ( Model, Cmd Msg )
init =
    ( Model (TemperatureInfo "Did not load" 0 0 0 0) "" Material.model
    , Cmd.none
    )
```

通过这次更新，我们已经整理出了我们的天气应用所需的初始值。

# 更新 Msg 联合类型和 update 函数

为了反映我们通过引入 `elm-mdl` 包所做出的改进，我们需要向我们的 `Msg` 联合类型添加另一个类型构造函数。`Msg` 联合类型将看起来与 第七章 中完全相同，*在 Elm 中制作天气应用*，除了新的 `Mdl` 类型构造函数，我们将简单地将其附加到联合类型的末尾，如下所示：

```js
type Msg
    = GetTemp
    | CityInput String
    | NewTemp (Result Http.Error String)
    | Mdl (Material.Msg Msg)
```

`Mdl` 类型构造函数是 `mdl` 将生成的消息。

让我们现在改进我们的 `update` 函数，它在 第七章 的末尾，*在 Elm 中制作天气应用*看起来是这样的：

```js
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
```

如前述代码片段所示，我们为 `GetTemp`、`NewTemp` 和 `CityInput` 进行了模式匹配，这在 第七章 中有解释，*在 Elm 中制作天气应用*。我们现在需要修改 `update` 函数以进行对新的 `Msg` 类型构造函数 `Mdl` 的模式匹配。

实际上，这意味着 `update` 函数将保持完全不变，只是增加了一个简单的修改：为我们的 `case-of` 表达式添加 `Mdl` 分支：

```js
update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        Mdl msg_ ->
            Material.update Mdl msg_ model

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

在下一节中，我们将添加对 `view` 函数的更新。

# 更新 view 函数

第七章《在 Elm 中制作天气应用》结尾的`view`函数看起来如下：

```js
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
```

由于在前面的代码中我们基本上没有使用样式，因此更新后的`view`函数在第一眼看起来可能有些令人畏惧：

```js
view : Model -> Html Msg
view model =
    div []
        [ Textfield.render Mdl
            [ 0 ]
            model.mdl
            [ Textfield.label "City"
            , Textfield.floatingLabel
            , Textfield.text_
            , Textfield.value model.city
            , Options.onInput CityInput
            ]
            []
        , Button.render Mdl
            [ 1 ]
            model.mdl
            [ Button.raised
            , Button.colored
            , Button.ripple
            , Options.onClick GetTemp
            ]
            [ text "Get temperature" ]
        , br [] []
        , Card.view
            [ Options.css "width" "256px"
            , Options.css "margin" "20px"
            , Elevation.e8
            , Elevation.transition 250
            , Color.background (Color.color Color.LightBlue Color.S50)
            ]
            [ Card.title
                [ Options.css "flex-direction" "column" ]
                [ Card.head [] [ text model.city ]
                , Options.div
                    [ Options.css "padding" "2rem 2rem 0 2rem" ]
                    [ Options.span
                        [ Typography.display4
                        , Typography.contrast 0.87
                        , Color.text Color.primary
                        ]
                        [ text (Round.round 0 model.temperatureInfo.temperature ++ "°") ]
                    ]
                ]
            , Card.actions []
                [ div [] [ text "Wind: ", text (toString model.temperatureInfo.windSpeed) ]
                , div [] [ text "Pressure: ", text (toString model.temperatureInfo.pressure) ]
                , div [] [ text "Humidity: ", text (toString model.temperatureInfo.humidity) ]
                ]
            ]
        ]
        |> Material.Scheme.top
```

在浏览器中检查您应用程序的外观，因为到目前为止，我们已经成功地将`elm-mdl`应用到它上面。

幸运的是，一旦你理解了它是如何工作的，前面的代码就相当直接了。让我们一步一步地来看。由于第七章，《在 Elm 中制作天气应用》，没有使用任何样式，我们使用的函数从一开始就很容易理解。我们只有一个`input`，一个`button`，以及四个包含文本节点的`div`。由于我们没有使用任何样式，我们使用了`br`函数来在视觉上分隔`input`、`button`和四个`div`。

第八章《向天气应用添加更多功能》中的改进包括添加`mdl`特定的元素，包括`mdl`特定的`Textfield`、`Button`和`Card`。让我们更仔细地看看`Textfield`。

# 理解`Textfield`函数

我们`Textfield`的代码如下：

```js
Textfield.render Mdl
            [ 0 ]
            model.mdl
            [ Textfield.label "City"
            , Textfield.floatingLabel
            , Textfield.text_
            , Textfield.value model.city
            , Options.onInput CityInput
            ]
            []
```

我们从`Textfield.render Mdl`开始，这意味着它的消息类型是`Mdl`，它只是简单地渲染`Textfield`组件。我们使用的每个`mdl`组件都需要有一个唯一的 ID，只需用一个数字标记。在先前的例子中，我们使用零为`Textfield`分配了一个唯一的 ID：`[ 0 ]`。

接下来，我们指定这个更新将应用到模型的哪个部分，即`model.mdl`。在第二个列表中，我们提供了我们希望`Textfield`具有的具体外观。

我们`Textfield`组件有关外观的所有选项都可以在以下 URL 找到，[`package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Textfield`](http://package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Textfield)。

要了解提供给我们的`Textfield`的第二列表中外观如何变化，我们可以简单地关闭列表中的某些成员，如下所示：

```js
Textfield.render Mdl
            [ 0 ]
            model.mdl
            [ --Textfield.label "City"
              --, Textfield.floatingLabel
              --, Textfield.text_
              Textfield.value model.city
            , Options.onInput CityInput
            ]
            []
```

通过注释掉`Textfield.label`、`Textfield.floatingLabel`和`Textfield.text_`，我们实际上关闭了它们，只留下了我们应用程序仍然可以工作的功能所需的最小部分。如果我们注释掉`Textfield.value model.city`或`Options.onInput CityInput`中的任何一个，我们的应用程序将停止工作，所以在先前的案例中，我们必须保留这两个，而我们可以注释掉其余的。

我们的应用程序会停止工作的原因是，我们会破坏更新函数期望从视图接收的模型。

如本节前面所述，我们可以参考官方文档中 `Textfield` 的 *选项*、*外观* 和 *HTML 属性* 部分，以获取不同的结果和样式。例如，我们可以将 `Textfield` 的 `maxlength` 设置为五个字符。在这种情况下，我们的 `Textfield` 需要看起来如下所示：

```js
Textfield.render Mdl
            [ 0 ]
            model.mdl
            [ Textfield.maxlength 5
            , Textfield.text_
            , Textfield.value model.city
            , Options.onInput CityInput
            ]
            []
```

现在，我们将能够输入一个最多五个字符的城市名称。例如，我们仍然可以查找巴黎的天气信息，但我们将无法查找伦敦，因为我们无法输入该城市名称的所有六个字符。

接下来，我们将通过 `Button` 函数。

# 理解 `Button` 函数

查看位于 `Textfield` 后面的 `Button`，我们可以看到以下代码：

```js
Button.render Mdl
            [ 1 ]
            model.mdl
            [ Button.raised
            , Button.colored
            , Button.ripple
            , Options.onClick GetTemp
            ]
            [ text "Get temperature" ]
```

在理解了 `Textfield` 的工作原理之后，我们可以几乎一眼就看出 `view` 函数前面的部分：我们的 `Button` 使用 `Mdl` 消息进行渲染。我们给 `Button` 分配一个 `id` 为 `1`，并使用 `Button.raised`、`Button.colored` 和 `Button.ripple` 的外观选项。我们通过使用 elm-mdl 的 `Options.onClick` 发送 `GetTemp` 消息来完成它。还有一个第三个列表，内容如下：

```js
[ text "Get temperature" ]
```

如果我们本来就不打算使用它，为什么在 `Textfield` 中使用一个空列表？原因当然是满足 Elm 的静态类型系统。例如，如果我们从 `Textfield` 函数中删除最后一个空列表，编译器将抛出一个错误。接下来，我们将查看 `Card` 函数。

# 理解 `Card` 函数

作为组件，`Card` 相对复杂，因为它由几个子部分组成，因为它的作用是以连贯的方式显示相关信息。`Card` 函数的官方文档可在以下网址找到，[`package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Card`](http://package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Card)。

# 理解 `Card.view` 的用法

让我们看看使用 `Card` 函数的前几行：

```js
        , br [] []
        , Card.view
            [ Options.css "width" "256px"
            , Options.css "margin" "20px"
            , Elevation.e8
            , Elevation.transition 250
            , Color.background (Color.color Color.LightBlue Color.S50)
            ]
```

首先要注意的是，这里的 `br` 函数是完全多余的，因此我们可以简单地将其删除，而不会对天气应用的函数性或布局产生任何影响。我们接下来的一行以 `Card.view` 开始。

正如官方文档所述，`Card.view` 函数用于构建卡片。虽然 `Textfield` 和 `Button` 是使用 `render` 函数构建的，但在这里我们使用的是 `view`。我们使用 `Options.css` 添加额外的样式，后面跟着两个字符串，前者设置 CSS 属性，后者设置属性的值。

`Material.Elevation`用于给我们的`mdl`组件添加阴影效果。有关可用的`Elevations`的完整描述，请参阅官方文档，该文档位于[`package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Elevation`](http://package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Elevation)。例如，要使用`Elevation`获取最大可能的阴影，我们可以将读取`Elevation.e8`的行更改为新值`Elevation.e24`。与阴影类似，我们可以设置我们的过渡所需的时间，以毫秒为单位。我们使用`Elevation.transition 250`将过渡设置为四分之一秒。

下一行设置了卡片的颜色：

```js
, Color.background (Color.color Color.LightBlue Color.S50)
```

处理`Material.Color`的`elm-mdl`包的官方文档位于[`package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Color`](http://package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Color)。

# 渲染`Card.title`内容块

接下来，我们渲染一个`Card.title`，如下所示：

```js
            [ Card.title
                [ Options.css "flex-direction" "column" ]
                [ Card.head [] [ text model.city ]
                , Options.div
                    [ Options.css "padding" "2rem 2rem 0 2rem" ]
                    [ Options.span
                        [ Typography.display4
                        , Typography.contrast 0.87
                        , Color.text Color.primary
                        ]
                        [ text (Round.round 0 model.temperatureInfo.temperature ++ "°") ]
                    ]
                ]
```

在逐行分析前面的代码之前，重要的是要注意`elm-mdl`中的每个`Card`都由内容块组成。内容块可以是`Card.title`、`Card.media`、`Card.text`和`Card.actions`。

`Card.title`内容块的类型签名如下：

```js
title : List (Style a) -> List (Html a) -> Block a
```

这意味着每个标题内容块都包含两个`列表`并返回一个`Block`。

所有其他内容块（`media`、`text`和`actions`）具有相同的类型签名，除了内容块函数的名称之外。

由于我们知道`title`内容块需要有两个`列表`，并且第一个`列表`需要指定样式，因此现在很容易理解这一行代码的作用：

```js
[ Options.css "flex-direction" "column" ]
```

上一行代码只是指定了要在我们的`Card.title`上使用的样式。如果我们想的话，例如，我们可以在这里添加另一个样式：

```js
  [ Options.css "flex-direction" "column"
  , Options.css "padding-left" "100px"
  ]
```

显然，由于前面的更改，我们的卡片将获得 100 像素的左内边距。`Card.title`内容块内部的第二个`列表`指定了`Html`：

```js
[ Card.head [] [ text model.city ]
                , Options.div
                    [ Options.css "padding" "2rem 2rem 0 2rem" ]
                    [ Options.span
                        [ Typography.display4
                        , Typography.contrast 0.87
                        , Color.text Color.primary
                        ]
                        [ text (Round.round 0 model.temperatureInfo.temperature ++ "°") ]
                    ]
                ]
```

在第一行，我们可以看到使用了`Card.head`。`Card.head`的官方文档位于[`package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Card#head`](http://package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Card#head)。`Card.head`函数的行为正如我们所期望的那样——它接受两个`列表`并返回一个`Html a`。第一个`列表`允许我们指定`Style`，第二个`列表`允许我们指定`Html`。

查看`Material.Options`的官方文档，该文档位于[`package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Options`](http://package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Options)，我们可以导航到`Elements`部分，它以`div`开始。正如官方文档所述，`div`是一个：

"为将 elm-mdl 样式应用于 div 元素的超常见情况提供的便利函数。"

在 `Options.div` 中，我们首先在第一个 `List` 中指定样式，然后是第二个 `List` 中的 `Html` 和 `Options.span`。按照相同的模式，`Options.span` 本身包含两个 `List`。在第一个 `List` 中，我们通过调用 `Material.Typography` 和 `Material.Color` 来指定要使用的样式——总共三种不同的样式。然后，在第二个 `List` 中，如预期的那样，我们渲染一个 `text` 节点。

`text` 节点内的内容，括号中的内容可能看起来有点复杂，所以让我们来分析一下。首先，我们可以看到我们在调用 `Round.round`。让我们参考官方文档来获取更多关于这个包的信息。为此，将您的浏览器指向以下 URL，[`package.elm-lang.org/packages/myrho/elm-round/latest/Round`](http://package.elm-lang.org/packages/myrho/elm-round/latest/Round)。

如 `Round` 包的官方页面所述，它允许我们将 `Float` 转换为 `String`，并且额外的好处是可以设置小数点后数字的位数。它还允许我们指定如何对 `Float` 中的其余数字进行四舍五入。

在官方页面上我们看到的第一例子是：

```js
x = 3.141592653589793

round 2 x -- "3.14"
```

上述代码显示 `round` 接收一个 `Int` 和一个 `Float`，并返回一个 `String`。您可以在以下链接中找到确切的类型签名，[`package.elm-lang.org/packages/myrho/elm-round/latest/Round#round`](http://package.elm-lang.org/packages/myrho/elm-round/latest/Round#round)。

在我们的情况下，我们有以下代码：

```js
Round.round 0 model.temperatureInfo.temperature ++ "°"
```

查看类型别名 `temperatureInfo`，我们可以看到 `temperatureInfo.temperature` 是一个 `Float`。因此，我们确实是在接收一个 `Int` 和一个 `Float`，然后返回一个 `String`。这里有一个需要注意的地方，就是拼接的温度单位符号，虽然它是一个 `String`，但它会与一个 `Float` 拼接，这都要归功于 `Round.round` 函数。

第一个参数的值是零，这意味着我们不想在显示温度时有任何小数点。我们可以将其更改为，例如，小数点后一位数字，只需简单地替换零，如下所示：

```js
Round.round 1 model.temperatureInfo.temperature ++ "°"
```

接下来，我们将查看 `Card.actions` 代码。

# 理解 `Card.actions` 代码

最后，是时候讨论我们的 `Card` 组件中的 `Card.actions` 部分：

```js
, Card.actions []
    [ div [] [ text "Wind: ", text (toString model.temperatureInfo.windSpeed) ]
    , div [] [ text "Pressure: ", text (toString model.temperatureInfo.pressure) ]
    , div [] [ text "Humidity: ", text (toString model.temperatureInfo.humidity) ]
    ]
```

`Material.Card` 的官方文档指定 `Card.actions` 允许我们生成一个 `actions` 块。正如我们之前所看到的，`actions` 块是 `mdl` 组件的四个可能内容块之一。

材料设计语言的官方文档——即 Google 文档，而不是 Elm 包文档——指出，卡片操作应用作与我们的卡片进行交互的方式。本质上，它是对动作按钮的调用。例如，如果我们在一个网站上以 MDL 卡的形式显示博客文章摘录的列表，卡片操作可以是包含 *阅读更多* 文本节点的按钮。

然而，在我们的例子中，我们并没有真正与卡片组件下温度提供的信息进行交互。换句话说，我们并没有让用户点击温度下方列出的*风速*、*气压*和*湿度*信息的意图。

因此，我们可以安全地将读取为`, Card.actions []`的行更改为以下代码：`, Card.text []`。我们可以保留所有其他代码不变，应用仍然可以工作。唯一的变化将与我们的天气应用的编译 HTML 结构有关。为了看到变化，我们需要检查`dev`工具中的代码，这可以通过在大多数主要浏览器中按*F12*按钮来访问。

如果在我们的 Elm 应用中使用`Card.action`，浏览器中的结果`div`将具有以下 CSS 类属性：`mdl-card__actions`。如果我们改为在视图中使用`Card.text`，浏览器中运行的应用的结果`div`将具有 CSS 类属性`mdl-card__supporting-text`。

我们通过在底部管道`Material.Scheme.top`来完成`view`函数的代码。这将向我们的应用添加`mdl` CSS。要了解更多信息，请参阅官方文档[`package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Scheme#top`](http://package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Scheme#top)。

# 添加颜色方案

我们可以通过利用颜色方案几乎毫不费力地改变我们的 mdl 样式应用的外观。为了了解如何做到这一点，我们首先需要参考 elm-mdl 包的官方文档。更具体地说，我们对`topWithScheme`函数感兴趣，这是一个在`elm-mdl`包中提供的函数。此函数的官方文档可在[`package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Scheme#topWithScheme`](http://package.elm-lang.org/packages/debois/elm-mdl/8.1.0/Material-Scheme#topWithScheme)找到。如果您访问所引用的 URL，您将看到我们使它工作所需做的所有事情就是将主色和强调色作为参数提供给`topWithScheme`函数。

要更好地了解 mdl 中的颜色方案如何工作，请参阅颜色方案定制器[https://getmdl.io/customize/index.html]。

实际上，这意味着我们可以通过更改一行代码来更新`view`函数中的颜色方案。到目前为止，我们的`view`函数的前几行看起来如下：

```js
view : Model -> Html Msg
view model =
    div []
        [ Textfield.render Mdl
```

要更新视图以使用`topWithScheme`函数，我们只需添加以下内容：

```js
view : Model -> Html Msg
view model =
    div []
         Material.Scheme.topWithScheme Color.Orange Color.Red <|
            Textfield.render Mdl
```

如我们所见，主色调现在是橙色，强调色是红色。到目前为止，我们几乎完成了天气应用的更新。对`subscriptions`函数或`getTemperature`函数都没有需要做的更改。

唯一需要更新的函数是 `decodeTemperatureInfo` 函数，其中我们需要进行一些小的调整，我们将在下一节中进行这些调整。

# 更新 decodeTemperatureInfo

我们在 [第七章 中停止了，*在 Elm 中制作天气应用*，其中 `decodeTemperatureInfo` 的以下代码：

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

在 `decodeTemperatureInfo` 函数中需要更新的唯一事项是将温度从开尔文转换为摄氏度。幸运的是，这种转换非常直接：为了转换为摄氏度，我们只需从基于开尔文的值中减去 `273.15`。换句话说，如果当前温度是 `293.15` 开尔文，转换为摄氏度的过程如下所示：

```js
293.15 - 273.15 = 20
```

因此，`293.15` 开尔文等于 `20` 摄氏度。这使得更新我们的 `decodeTemperatureInfo` 函数变得相当简单。我们只需用以下代码替换现有的代码：

```js
temperature =
     Result.withDefault 0 (Decode.decodeString (Decode.at [ "main", "temp" ] Decode.float) json) - 273.15
```

我们将保持其余代码与我们的应用前一个版本中的代码相同。

在本节中，我们更新了温度，以便在浏览器中显示摄氏度到开尔文的转换。如果你在浏览器中查看应用，你将能够看到这个变化。

# 摘要

在本章中，我们学习了如何使用神奇的 `elm-mdl` 包来改进现有应用的样式。我们探讨了如何使用 `Result` 和 `Maybe` 来处理可能失败的操作以及处理可能缺失的数据。我们还研究了使用解码器以及如何对它们进行映射。此外，我们还探讨了如何使用 HTTP 包来获取远程 JSON 数据。

在下一章中，我们将探讨如何为我们的 Elm 应用编写测试。
