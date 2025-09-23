# 第十章：将一切整合在一起

在前面的章节中，我们探讨了在 ReasonML 中进行类型驱动开发的不同工具和技术。

在本章中，通过一个最终示例，我们将培养一种对何时使用每种类型驱动技术来解决问题的感觉。让我们看看我们至少可以部分地创建处理输入（在一个小的 JavaScript 应用中）的社交、生产力和商业应用代码。更精确地说，我们这里指的是由互联网或平台公司推出的成功应用，如 Gmail、Facebook、Twitter、Skype、Airbnb 或 Uber。

在本章中，我们将介绍以下主题：

+   从变体类型开始（版本 1）

+   使用更多模式匹配（版本 2）

+   转向多态变体类型（版本 3）

+   使用记录（版本 4）

+   使用模块进行代码结构（版本 5）

+   代码结构的另一种结构（版本 6）

+   改进：使用列表作为输出（版本 7）

+   另一项改进：使用可变记录（版本 8）

+   单元测试我们的代码（最终版本）

# 从变体类型开始（版本 1）

首先，我们需要一个 `type` 来表示公司，这些公司是互联网应用。基于这个 `type`，我们可以考虑编写函数，以帮助我们以类型安全的方式构建逻辑。让我们看看结果如何。

作为第一次尝试，我们从小处着手，定义一个我们感兴趣的互联网公司的变体类型。正如我们在前面的章节中看到的，我们将使用模式匹配来展示这些公司向用户提供的应用列表。

我们如下定义互联网公司类型：

```js
type internetCompany =
   | Facebook
   | Google
   | Twitter;
```

现在，我们定义一个显示应用的函数，如下所示：

```js
let *apps* = (company: internetCompany) : string => {
   switch (company) {
     | Facebook => "facebook, messenger, ads"
     | Google => "gmail, google+, maps, ads"
     | Twitter => "twitter"
   }
 };
```

以下代码将展示一些来自 Google 的应用：

```js
let *googleApps* = apps(Google);
Js.log(googleApps);
```

这里是整个代码的输出（由 `src/Ch10/Ch10_PlatformCompany_V1.re` 生成）：

```js
gmail, google+, maps, ads
```

这是一个良好的开端。

由于涉及不同的应用类别（社交、商业、通信、娱乐等），我们可以通过使用更多的模式匹配代码来区分应用列表，从而丰富我们的类型驱动逻辑。现在让我们这样做。

# 使用更多模式匹配（版本 2）

如果我们将变体的构造函数更改为传递一个字符串以表示每个可能的类别（例如 `Facebook(string)`，它可以给出 `Facebook("social")` 或 `Facebook("business")`），我们就可以做到这一点。额外的模式匹配可能如下所示：

```js
Facebook(str) => switch str {            
        | "social" => "facebook, messenger, instagram"
        | "business" => "facebook ads"
```

那么，让我们从定义互联网公司变体类型开始，如下所示：

```js
type internetCompany =
   | Facebook(string)
   | Google(string)
   | Twitter(string);
```

如我们计划的那样，我们的函数的模式匹配代码可以演变，如下所示：

```js
let apps = (company: internetCompany) : string => {
   switch (company) {
     | Facebook(str) => switch str {            
         | "social" => "facebook, messenger, instagram"
         | "business" => "facebook ads"
     }
     | Google(str) => switch str {            
         | "social" => "google+, gmail"
         | "business" => "google ads, google adsense, gmail for business"
     }
     | Twitter(str) => switch str {            
         | "social" => "twitter"
         | "business" => "twitter ads"
     }   
   }
 };
```

现在，以下代码将展示一些结果数据，这样我们就可以看到事情是否按预期进行：

```js
Js.log(Js.String.toUpperCase("facebook"))
Js.log("Business: " ++ apps(Facebook("business")));
Js.log("Social: " ++ apps(Facebook("social")));

Js.log(Js.String.toUpperCase("google"))
Js.log("Business: " ++ apps(Google("business")));
Js.log("Social: " ++ apps(Google("social")));
```

这里是执行此版本时我们得到的结果（`src/Ch10/Ch10_PlatformCompany_V2.bs.js`，由 `src/Ch10/Ch10_PlatformCompany_V2.re` 生成）：

```js
FACEBOOK
Business: facebook ads
Social: facebook, messenger, instagram
GOOGLE
Business: google ads, google adsense, gmail for business
Social: google+, gmail
```

注意，我们的类型只考虑了互联网公司，但我们的逻辑也应该适用于其他使用技术（Web 移动、数据库、AI）以可扩展的方式构建和提供服务的现代公司。我们可以称它们为**平台公司**。因此，我们也可以为平台公司添加一个类型。

我们甚至可以认为一些互联网公司（至少是那些大公司）也是平台公司或者已经推出了平台业务。因此，我们可以从*普通变体类型*切换到*多态变体类型*，以利用它们的类型重用能力。

让我们从这些新想法开始第三个版本。

# 切换到多态变体类型（版本 3）

在这个新版本中，我们定义了我们构建逻辑的两个类型，在第二个类型中重用第一个类型，如下所示：

```js
type internetCompany = [ `Facebook(string) | `Google(string) | `Twitter(string) ];
type platformCompany = [ internetCompany | `Amazon(string) | `Uber(string) ]
```

基于此，我们使我们的函数如下进化（使用`platformCompany`允许所有变体都能接受`company`参数）：

```js
let *apps* = (company: platformCompany) : string => {
  switch (company) {
    | `Facebook(str) => switch str {            
      | "social" => "facebook, messenger, instagram"
      | "business" => "facebook ads"
    }
    | `Google(str) => switch str {            
      | "social" => "google+, gmail"
      | "business" => "google ads, google adsense, gmail for business"
    }
    | `Twitter(str) => switch str {            
      | "social" => "twitter"
      | "business" => "twitter ads"
    }   
    | `Amazon(str) => switch str {            
      | "social" => ""
      | "business" => "amazon, AWS"
    } 
    | `Uber(str) => switch str {            
      | "social" => ""
      | "business" => "uber"
    } 
  }
};
```

现在，让我们添加通常的快速数据显示代码，如下所示：

```js
Js.log(Js.String.toUpperCase("facebook"))
Js.log("Business: " ++ apps(`Facebook("business")));
Js.log("Social: " ++ apps(`Facebook("social")));
Js.log("")

Js.log(Js.String.toUpperCase("google"))
Js.log("Business: " ++ apps(`Google("business")));
Js.log("Social: " ++ apps(`Google("social")));
Js.log("")

Js.log(Js.String.toUpperCase("uber"))
Js.log("Business: " ++ apps(`Uber("business")));
```

这看起来是一个很好的改进，但让我们看看它是否有效。基于`src/Ch10/Ch10_PlatformCompany_V3.re`生成的 JavaScript 代码的测试给出了以下输出：

```js
FACEBOOK
Business: facebook ads
Social: facebook, messenger, instagram

GOOGLE
Business: google ads, google adsense, gmail for business
Social: google+, gmail

UBER
Business: uber
```

很好！我们还能添加什么？

虽然这个实现很棒，但你很快就能看出它缺乏处理更复杂数据结构的能力。基本上，我们希望用一个包含所有必要信息的*app*来表示，例如*名称*和*URL*（至少对于 Web 应用）。为此，Reason 有一个方便的工具我们可以使用：*记录*。

让我们看看使用记录处理应用的下个版本会怎样。

# 使用记录（版本 4）

与我们的第一个变体类型相比，没有变化，但我们将添加`webapp`记录类型定义，如下所示：

```js
type internetCompany = [ `Facebook(string) | `Google(string) | `Twitter(string) ];
type platformCompany = [ internetCompany | `Amazon(string) | `Uber(string) ];
type webapp = {
  *name*: string,
  *url*: string,
};
```

然后，我们可以使用该记录类型输入一些数据给程序的其余部分，如下所示：

```js
/* some data */
let *facebook*: webapp = {
  *name*: "facebook",
  *url*: "https://facebook.com",
}
let *facebookads*: webapp = {
  *name*: "facebook ads",
  *url*: "https://www.facebook.com/business",
}
let *messenger*: webapp = {
  *name*: "messenger",
  *url*: "https://www.facebook.com/messenger",
}
let *instagram*: webapp = {
  *name*: "instagram",
  *url*: "https://www.instagram.com",
}
```

注意，我们只为最小测试定义了这些值中的几个。真正的生产就绪代码应该包括所有需要的记录值。

最小的`apps`函数将如下所示：

```js
let *apps* = (company: platformCompany) : string => {
  switch (company) {
    | `Facebook(str) => switch str {            
        | "social" => facebook.name ++ ", " ++ messenger.name ++ ", " ++ instagram.name
        | "business" => facebookads.name
    }
  }
};
```

让我们添加一些输出显示代码，如下所示：

```js
Js.log(Js.String.toUpperCase("facebook"));
Js.log("Business: " ++ apps(`Facebook("business")));
Js.log("Social: " ++ apps(`Facebook("social")));
```

我们得到了这个最小测试用例的以下输出（来自`src/Ch10/Ch10_PlatformCompany_V4.re`文件的代码）：

```js
FACEBOOK
Business: facebook ads
Social: facebook, messenger, instagram
```

在下一个版本中，我们还将使用每个应用的 URL 并在输出中显示它。

我们还可以通过将一些类型和函数打包到模块中来改进全局代码结构。

# 使用模块进行代码结构（版本 5）

我们可以做的第一个改进是创建一个模块来包含 Web 应用的记录以及一个返回它们字符串表示的函数。让我们称这个模块为`WebApp`。它的定义如下：

```js
module *WebApp* = {
  type t = {
    *name*: string,
    *url*: string,
  };

  let *toString* = (app: t) => app.name ++ " (" ++ app.url ++ ")" ;
}
```

然后，就像上一个版本一样，我们有我们的示例 Web 应用值。唯一改变的是，类型注解使用`WebApp.t`完成。这部分代码如下：

```js
let *facebook*: WebApp.t = {
  *name*: "facebook",
  *url*: "https://facebook.com",
}
let *facebookads*: WebApp.t = {
  *name*: "facebook ads",
  *url*: "https://www.facebook.com/business",
}
let *messenger*: WebApp.t = {
  *name*: "messenger",
  *url*: "https://www.facebook.com/messenger",
}
let *instagram*: WebApp.t = {
  *name*: "instagram",
  *url*: "https://www.instagram.com",
}
```

然后，我们创建一个名为 `Platform` 的模块，用于其余的逻辑。它可能包含公司类型的定义和 `apps` 函数。为了简化，让我们选择一个包含所有公司的唯一多态变体类型。在模块内部，我们可以称它为 `t`。

我们创建模块如下：

```js
module *Platform* = {  
  type t = [ `Facebook(string) 
             | `Google(string) 
             | `Twitter(string) 
             | `Amazon(string) 
             | `Uber(string) 
           ];

  let *apps* = (company: t) : string => {
    switch (company) {
      | `Facebook(str) => switch str {            
          | "social" => WebApp.toString(facebook) ++ ", " ++ WebApp.toString(messenger) ++ ", " ++ WebApp.toString(instagram)
          | "business" => WebApp.toString(facebookads)
      }
    }
  };
}
```

我们可以添加类似的代码来展示一些可能的输出，以及代码执行（由 `src/Ch10/Ch10_PlatformCompany_V5.re` 文件生成的 JS 代码）给出的输出类似于以下内容：

```js
FACEBOOK
Business: facebook ads (https://www.facebook.com/business)
Social: facebook (https://facebook.com), messenger (https://www.facebook.com/messenger), instagram (https://www.instagram.com)
```

我们的结果令人鼓舞。可能会有不同的可能性，所以让我们继续通过尝试替代的代码结构并进行后续改进来实验。

# 替代代码结构（版本 6）

到目前为止，实际上可以在我们的代码文件中使用更少的模块来编写全面的代码。让我们保留平台模块，但将我们在 *使用模块进行代码结构（版本 5）* 中拥有的 `WebApp` 模块的内容移出，从而消除该模块。同时，我们将调整一些名称和定义。

当我们有更少的模块时，我们可以通过引入一个接口文件（`src/Ch10/Ch10_PlatformCompany_V6.rei`）来改进基于类型的代码，该文件用于存储 `.re` 文件（`src/Ch10/Ch10_PlatformCompany_V6.re`）的类型信息，如下所示：

```js
type webapp;
type pfcompany;

let *appToString*: webapp => string;
```

接下来，我们定义 `webapp` 和 `pfcompany` 类型，并相应地调整 `appToString` 函数（在 `src/Ch10/Ch10_PlatformCompany_V6.re` 中），如下所示：

```js
/* Basic types and functions we need (see .rei file) */

type webapp = {
    *name*: string,
    *url*: string,
};

type pfcompany = [ `Facebook(string) 
                    | `Google(string) 
                    | `Twitter(string) 
                    | `Amazon(string) 
                    | `Uber(string) 
                    ];

let *appToString* = (app: webapp) => app.name ++ " (" ++ app.url ++ ")" ;
```

然后，在执行 *let 绑定* 以获取 web 应用输入数据的部分没有算法上的变化，因此为了可读性，我们不会在这里重复那段代码。您可以在 `src/Ch10/Ch10_PlatformCompany_V6.re` 文件中看到完整的记录值集合，您会注意到我们添加了一些 Google 应用的输入，如下所示：

```js
/* Data */

/* ... 
   Extract from src/Ch10/Ch10_PlatformCompany_V6.re */

let *google*: webapp = {
  *name*: "google",
  *url*: "https://google.com",
}
let *gmail*: webapp = {
  *name*: "gmail",
  *url*: "https://google.com/gmail",
}
let *googleads*: webapp = {
  *name*: "google ads",
  *url*: "https://ads.google.com",
}
let googleplus: webapp = {
  *name*: "google+",
  *url*: "https://plus.google.com",
}
```

接下来，我们改进 `Platform` 模块代码：

+   我们有 `Platform` 模块，它前面是它的签名 `PlatformType`。

+   我们现在使用 `pfcompany` 作为 `company` 值的类型。

+   我们在 `apps` 函数的模式匹配代码中添加了 Google（及其应用）的情况。

平台模块代码的改进部分如下：

```js
/* Platform module, signature followed by implementation */

module type PlatformType = {
  let apps: pfcompany => string;
};

module *Platform*: PlatformType = {  
  let *apps* = (company: pfcompany) : string => {
    switch (company) {
      | `Facebook(str) => switch str {            
          | "social" => appToString(facebook) ++ ", " ++ appToString(messenger) ++ ", " ++ appToString(instagram)
          | "business" => appToString(facebookads)
      }

      | `Google(str) => switch str {            
          | "social" => appToString(googleplus)
          | "business" => appToString(googleads)
      }
    }
  };
}
```

为了使这个实现版本的测试变得容易，我们添加了常用的输入和输出打印代码，如下所示：

```js
Js.log("Facebook")
Js.log("Business: " ++ Platform.apps(`Facebook("business")));
Js.log("Social: " ++ Platform.apps(`Facebook("social")));
print_newline();
Js.log("Google")
Js.log("Business: " ++ Platform.apps(`Google("business")));
Js.log("Social: " ++ Platform.apps(`Google("social")));
```

总结一下，我们通过使用 `.rei` 文件（有助于代码文档）改进了类型声明，为平台添加了一个模块签名（`PlatformType`），并通过添加 Google 情况来扩展了输入的覆盖范围。

鼓励读者添加输入数据（用于变体类型中的其他公司，如 Twitter、Amazon 和 Uber）。

# 一种改进——使用列表作为输出（版本 7）

您注意到，在我们的输出中，我们只是使用字符串（通过连接）。从一开始，我们就可以返回每个公司真实的应用列表。没问题，让我们现在更改代码来实现这一点。

改变仅限于`Platform`模块。在签名中，对于`apps`函数，我们将输出类型从`string`更改为`list(string)`。在函数的模式匹配部分，我们相应地更改实现，例如，通过返回`[appToString(facebook), appToString(messenger), appToString(instagram)]`列表来表示 Facebook 应用。

新版本的主要部分如下：

```js
/* Platform module, signature followed by implementation */

 module type PlatformType = {
   let *apps*: pfcompany => list(string);
 };

 module *Platform*: PlatformType = {  
   let *apps* = (company: pfcompany) : list(string) => {
     switch (company) {
       | `Facebook(str) => switch str {            
           | "social" => [appToString(facebook), appToString(messenger), appToString(instagram)]
           | "business" => [appToString(facebookads),]
       }

       | `Google(str) => switch str {            
           | "social" => [appToString(googleplus),]
           | "business" => [appToString(googleads),]
       }
     }
   };
 }
```

由于我们现在输出列表，所以使用`Array.of_list`来打印它们是个好主意，因为它是一个既好又快的解决方案。我们用以下代码更改代码的最后部分：

```js
 Js.log("Facebook")
 print_string("Business: ")
 Js.log(Array.of_list(Platform.apps(`Facebook("business"))));
 print_string("Social: ")
 Js.log(Array.of_list(Platform.apps(`Facebook("social"))));
 print_newline();

 Js.log("Google")
 print_string("Business: ")
 Js.log(Array.of_list(Platform.apps(`Google("business"))));
 print_string("Social: ")
 Js.log(Array.of_list(Platform.apps(`Google("social"))));
```

使用我的输入数据执行生成的代码，得到以下输出：

```js
Facebook
Business: [ 'facebook ads (https://www.facebook.com/business)' ]
Social: [ 'facebook (https://facebook.com)',
  'messenger (https://www.facebook.com/messenger)',
  'instagram (https://www.instagram.com)' ]

Google
Business: [ 'google ads (https://ads.google.com)' ]
Social: [ 'google+ (https://plus.google.com)' ]
```

太棒了！这是一个有趣的改进。让我们继续添加。

# 另一个改进——使用可变记录（版本 8）

现在，我们可以为`webapp`类型使用可变记录，这样我们就可以用它来处理不断更新的有趣应用数据。这样一个数据点是创建的账户数量。另一个可能是对应移动应用的下载次数。

在这个例子中，让我们看看如何通过向记录中添加*账户数量*参数来改进我们的实现。这是通过使用`mutable numberOfAccounts: int`作为记录定义中该参数的条目来完成的。

因此，现在只有这一个更改，但让我们回顾一下`webapp`类型、`pfcompany`类型和`appToString`函数的定义，以便更好地阅读，如下所示：

```js
type webapp = {
    *name*: string,
    *url*: string,
    mutable *numberOfAccounts*: int,
};

type pfcompany = [ `Facebook(string) 
                    | `Google(string) 
                    | `Twitter(string) 
                    | `Amazon(string) 
                    | `Uber(string) 
                    ];

let *appToString* = (app: webapp) => app.name ++ " (" ++ app.url ++ ")" ;
```

之后，让我们添加一个函数，每次有新的注册时，都会在记录中对应网页应用的`numberofAccounts`值上增加。这个函数可能看起来如下所示：

```js
let *newSignUp* = (app: webapp) : unit => {
  app.*numberOfAccounts* = app.*numberOfAccounts* + 1;
  ()
};
```

然后，像之前一样，我们有输入数据部分。但现在我们有了新的参数`newSignUp`，我们必须不断更新它。为了简化问题，让我们假设所有这些应用目前都有相同的账户数量，我们选择 10,000 作为一个任意数。因此，现在的记录定义如下：

```js
/* Data */

let *facebook*: webapp = {
 * name*: "facebook",
 * url*: "https://facebook.com",
  *numberOfAccounts*: 10000,
}
let *facebookads*: webapp = {
  *name*: "facebook ads",
  *url*: "https://www.facebook.com/business",
  *numberOfAccounts*: 10000,
}
let *messenger*: webapp = {
  *name*: "messenger",
  *url*: "https://www.facebook.com/messenger",
  *numberOfAccounts*: 10000,
}
let *instagram*: webapp = {
  *name*: "instagram",
  *url*: "https://www.instagram.com",
  *numberOfAccounts*: 10000,
}
let *google*: webapp = {
  *name*: "google",
  *url*: "https://google.com",
  *numberOfAccounts*: 10000,
}
let *gmail*: webapp = {
  *name*: "gmail",
  *url*: "https://google.com/gmail",
  *numberOfAccounts*: 10000,
}
let *googleads*: webapp = {
  *name*: "google ads",
  *url*: "https://ads.google.com",
  *numberOfAccounts*: 10000,
}
let *googleplus*: webapp = {
  *name*: "google+",
  *url*: "https://plus.google.com",
  *numberOfAccounts*: 10000,
}
```

`Platform`模块部分没有变化，所以让我们转到下一部分，也就是测试代码的部分：

```js
Js.log("Facebook")
print_string("Business: ")
Js.log(Array.of_list(Platform.apps(`Facebook("business"))));
print_string("Social: ")
Js.log(Array.of_list(Platform.apps(`Facebook("social"))));
print_newline();

Js.log("New sign-up on Instagram")
*newSignUp*(instagram);
Js.log("New sign-up on Instagram")
*newSignUp*(instagram);
Js.log(instagram.numberOfAccounts)
```

从`src/Ch10/Ch10_PlatformCompany_V5.re`文件编译的代码执行给出以下输出：

```js
Facebook
Business: [ 'facebook ads (https://www.facebook.com/business)' ]
Social: [ 'facebook (https://facebook.com)',
  'messenger (https://www.facebook.com/messenger)',
  'instagram (https://www.instagram.com)' ]

New sign-up on Instagram
New sign-up on Instagram
10002
```

因此，我们只选取了一个有趣的用例，其中可以使用可变记录，并看到了添加该功能是多么简单。

# 单元测试我们的代码（最终版本）

现在是时候给我们的代码添加测试了！为了完整演示，让我们创建一个新的包，我们将在这个包中设置使用`Jest`框架编写单元测试所必需的设置。

在 Facebook 使用的另一种网络技术，`Jest`是一个用于编写 JavaScript 代码测试的框架，它还与编译到 JavaScript 的语言（如 TypeScript 或 ReasonML）一起工作。对于 Reason，我们还需要安装`bs-jest`包，它为 BuckleScript 提供了 Jest 的绑定。

# 创建我们的最终包并设置测试

为了快速启动工作，我们创建了一个名为 `Ch10-final` 的文件夹，其中包含以下文件结构：

```js
bsconfig.json
package.json
src
__tests__
```

我们通过复制 ReasonML 或 BuckleScript 代码生成器生成的我们正在使用的 `package.json` 文件，并对其进行调整来获取 `package.json` 文件。第一个版本如下：

```js
{
  "name": "Ch10-final",
  "version": "0.1.0",
  "scripts": {
    "build": "bsb -make-world",
    "start": "bsb -make-world -w",
    "clean": "bsb -clean-world"
  },
  "keywords": [
    "BuckleScript"
  ],
  "author": "",
  "license": "MIT",
  "devDependencies": {
    "bs-platform": "⁴.0.7",
  },
  "dependencies": {}
}
```

然后，我们可以使用 `npm` 将 `Jest` 和 `bs-jest`（确切地说是 `glennsl/bs-jest` 中引用的）添加到我们的包中作为开发依赖项。

要安装 `bs-jest`，请运行 `npm install @glennsl/bs-jest --save-dev` 命令。

要安装 `Jest`，请运行 `npm install jest --save-dev` 命令。

使用这些安装，所需的文件被安装到常规的 `node_modules` 子目录中，并且我们的 `package.json` 文件更改以引用已安装的两个依赖项的版本。`package.json` 文件中的更新 `devDependencies` 显示了两个新增项，如下所示：

```js
"devDependencies": {
    "@glennsl/bs-jest": "⁰.4.5",
    "bs-platform": "⁴.0.7",
    "jest": "²³.6.0"
  },
```

`bsconfig.json` 文件也是从我们之前使用的代码设置中复制（并调整）的（这允许 `bsb -w` 命令工作，并且我们的 `.re` 文件可以即时编译）。我们将 `sources` 列表调整为引用 `src` 目录的常规代码和 `__tests__` 目录的 *测试代码*（注意 `"type": "dev"` 部分），如下所示：

```js
"sources": [        
    {
      "dir" : "src",
    },  
    {
      "dir": "__tests__",
      "type": "dev",
    },
  ],
```

我们最后需要添加的是将 `@glennsl/bs-jest` 添加到 `bs-dev-dependencies` 参数中。我们将在下一分钟看到原因。

我们 `ch10final` 包的 `bsconfig.json` 文件如下：

```js
{
  "name": "ch10final",
  "version": "0.1.0",
  "sources": [        
    {
      "dir" : "src",
    },  
    {
      "dir": "__tests__",
      "type": "dev",
    },
  ],
  "package-specs": {
    "module": "commonjs",
    "in-source": true
  },
  "suffix": ".bs.js",
  "bs-dependencies": [
  ],
  "bs-dev-dependencies": ["@glennsl/bs-jest"],
  "warnings": {
    "error" : "+101"
  },
  "namespace": true,
  "refmt": 3
}
```

在 `src` 子目录中，我们可以添加我们的最终 Reason 代码文件（`src/Ch10-final/src/Ch10_PlatformCompany.re`）。我们的 ReasonML 代码与上一个版本（`src/Ch10/Ch10_PlatformCompany_V8.re` 文件）完全相同，除了我们通过删除最后部分（打印一些输出的代码片段）来简化它。

现在，让我们编写测试。

# 编写我们的第一个测试

要使用 `Jest` 测试代码，我们必须首先打开模块：

```js
open Jest;
```

然后，我们定义一个 `describe` 函数来封装测试套件。我们需要打开 `Expect` 模块，它是 `Jest` 的一部分，它提供了 `expect` 函数以及其他一些我们需要检查某些值是否满足特定条件的地方。我们还打开模块文件，其中包含我们的实现代码。到目前为止，我们的测试函数包含以下内容：

```js
describe("Platform", () => {
  open Expect;
  open Ch10_PlatformCompany;

});
```

现在，我们可以添加第一个测试来验证 `Platform` 模块中 `apps` 函数返回的数据。测试套件代码如下：

```js
describe("Platform", () => {
  open Expect;
  open Ch10_PlatformCompany;

  test("list facebook business app", () => {
    let *facebook_biz* = Platform.apps(`Facebook("business"));
    expect(facebook_biz) |> toEqual([ "facebook ads (https://www.facebook.com/business)" ]);
  });
});
```

我们不要停下来，添加第二个测试来验证在调用 `newSignUp` 函数后，账户数量是否是上一个数量加 `1`。

完整的测试套件代码如下（在 `src\Ch10-final\__tests__\Platform_test.re` 文件中）：

```js
open Jest;

describe("Platform", () => {
  open Expect;
  open Ch10_PlatformCompany;

  test("list facebook business app", () => {
    let *facebook_biz* = Platform.apps(`Facebook("business"));
    expect(facebook_biz) |> toEqual([ "facebook ads (https://www.facebook.com/business)" ]);
  });

  test("instagram number of accounts", () => {
    let *nb* = instagram.numberOfAccounts;
    *newSignUp*(instagram);
    expect(instagram.numberOfAccounts) |> toEqual(nb + 1);
  });
});
```

# 运行测试

在我们执行测试之前，我们需要使用`bsb -make-world`命令来构建代码。这个命令会找到测试代码并编译它。如果一切顺利，这个过程会将生成的文件复制到包结构的`lib`部分（位于`src\Ch10-final\lib\bs\__tests__`）。我们现在可以开始运行测试了。

要运行测试，我们使用`Jest`命令。在我的情况下，在 Windows 上运行，可执行文件实际上位于`node_modules\.bin\jest`，但根据你的情况，你只需输入`jest`即可。

当我们执行`Jest`测试运行器命令时，我们会得到以下输出：

```js
 PASS  __tests__/Platform_test.bs.js
  Platform
    √ list facebook business app (16ms)
    √ instagram number of accounts (2ms)

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        8.433s
Ran all test suites.
```

到目前为止，我们已经有一个良好的结构，包括一个测试套件，我们可以在此基础上进行扩展。我们可以扩展代码的输入部分，以考虑所有*平台公司*，并添加更多功能。我们还可以在过程中添加更多测试。这留作读者的练习。

# 摘要

在本章中，我们使用 Reason 的核心特性构建了一些类型安全的代码，这些代码也相对容易维护和扩展。我们可以进一步使用高级技术，如函子，来创建更通用的代码，但这对于这个小示例来说并不是必要的。

这是我们最后一章。我们通过类型驱动的过程解决了编码问题。在这个过程中，我们提高了对 ReasonML 的特性和技术，特别是变体类型、函数、模块和记录的理解。我们还探讨了如何使用`Jest`框架直接测试 ReasonML 代码。

希望这本书作为 ML 语言世界的入门之书是有用的，并且它将帮助你进一步学习 ReasonML 的技术和工具，如果你是一个具有额外技能的 Web 开发者，甚至可能包括 React。
