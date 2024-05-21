# 第八章：Reason 中的单元测试

在像 Reason 这样的类型化语言中进行测试是一个颇具争议的话题。有些人认为一个良好的测试套件会减少对类型系统的需求。另一方面，有些人更看重类型系统而不是他们的测试套件。这些意见上的差异可能导致一些激烈的辩论。

当然，类型和测试并不是互斥的。我们可以同时拥有类型和测试。也许 Reason 核心团队成员之一郑楼说得最好。

测试。这很容易，对吧？类型会减少一类测试的数量，但并不是所有测试。这是一个人们不够重视的讨论。他们总是把测试和类型对立起来。关键是：如果你有类型，并且*添加*了测试，你的测试将能够用更少的精力表达更多。你不再需要断言无效的输入。你可以断言更重要的东西。如果你想要，测试可以存在；你只是用它们表达了更多。

- 郑楼

您可以在以下 URL 上观看郑楼在 2017 年 React Conf 的演讲：

[`youtu.be/_0T5OSSzxms`](https://youtu.be/_0T5OSSzxms)

在本章中，我们将通过`bs-jest` BuckleScript 绑定来设置流行的 JavaScript 测试框架 Jest。我们将进行以下操作：

+   学习如何使用`es6`和`commonjs`模块格式设置`bs-jest`

+   对 Reason 函数进行单元测试

+   看看编写测试如何帮助我们改进我们的代码

要跟着做，克隆这本书的 GitHub 存储库，并从`Chapter08/app-start`开始使用以下代码：

```js
git clone https://github.com/PacktPublishing/ReasonML-Quick-Start-Guide.git
cd ReasonML-Quick-Start-Guide
cd Chapter08/app-start
npm install
```

# 使用 Jest 进行测试

Jest，也是由 Facebook 创建的，可以说是最受欢迎的 JavaScript 测试框架之一。如果你熟悉 React，你可能也熟悉 Jest。因此，我们将跳过正式介绍，开始在 Reason 中使用 Jest。

# 安装

就像任何其他包一样，我们从**Reason Package Index**（或简称为**Redex**）开始。

Reason 包索引：

[`redex.github.io/`](https://redex.github.io/)

输入`jest`会显示到 Jest 的`bs-jest`绑定。按照`bs-jest`的安装说明，我们首先用 npm 安装`bs-jest`：

```js
npm install --save-dev @glennsl/bs-jest
```

然后我们通过在`bsconfig.json`中包含它来让 BuckleScript 知道这个开发依赖项。请注意，键是`"bs-dev-dependencies"`而不是`"bs-dependencies"`：

```js
"bs-dev-dependencies": ["@glennsl/bs-jest"]
```

由于`bs-jest`将`jest`列为依赖项，npm 也会安装`jest`，因此我们不需要将`jest`作为应用程序的直接依赖项。

现在让我们创建一个`__tests__`目录，作为`src`目录的同级目录：

```js
cd Chapter08/app-start
mkdir __tests__
```

并告诉 BuckleScript 查找这个目录：

```js
/* bsconfig.json */
...
"sources": [
  {
    "dir": "src",
    "subdirs": true
  },
  {
    "dir": "__tests__",
    "type": "dev"
  }
],
...
```

最后，我们将更新`package.json`中的`test`脚本以使用 Jest：

```js
/* package.json */
"test": "jest"
```

# 我们的第一个测试

让我们在`__tests__/First_test.re`中创建我们的第一个测试，暂时使用一些简单的内容：

```js
/* __tests__/First_test.re */
open Jest;

describe("Expect", () =>
  Expect.(test("toBe", () =>
            expect(1 + 2) |> toBe(3)
          ))
);
```

现在运行`npm test`会出现以下错误：

```js
 FAIL lib/es6/__tests__/First_test.bs.js
  ● Test suite failed to run

    Jest encountered an unexpected token

    This usually means that you are trying to import a file which Jest
    cannot parse, e.g. it's not plain JavaScript.

    By default, if Jest sees a Babel config, it will use that to transform
    your files, ignoring "node_modules".

    Here's what you can do:
     • To have some of your "node_modules" files transformed, you can
       specify a custom "transformIgnorePatterns" in your config.
     • If you need a custom transformation specify a "transform" option in
       your config.
     • If you simply want to mock your non-JS modules (e.g. binary assets)
       you can stub them out with the "moduleNameMapper" config option.

    You'll find more details and examples of these config options in the
    docs:
    https://jestjs.io/docs/en/configuration.html

    Details:

    .../lib/es6/__tests__/First_test.bs.js:3
    import * as Jest from "@glennsl/bs-jest/lib/es6/src/jest.js";
           ^

    SyntaxError: Unexpected token *

      at ScriptTransformer._transformAndBuildScript (node_modules/jest-
      runtime/build/script_transformer.js:403:17)

Test Suites: 1 failed, 1 total
Tests: 0 total
Snapshots: 0 total
Time: 1.43s
Ran all test suites.
npm ERR! Test failed. See above for more details.
```

问题在于 Jest 无法直接理解 ES 模块格式。记住，我们已经通过以下配置（参见第二章，*设置开发环境*）配置了 BuckleScript 使用 ES 模块：

```js
/* bsconfig.json */
...
"package-specs": [
  {
    "module": "es6"
  }
],
...
```

解决这个问题的一种方法是将 BuckleScript 配置为使用`"commonjs"`模块格式：

```js
/* bsconfig.json */
...
"package-specs": [
  {
    "module": "commonjs"
  }
],
...
```

然后我们还需要更新 webpack 的`entry`字段：

```js
/* webpack.config.js */
...
entry: "./lib/js/src/Index.bs.js", /* changed es6 to js */
...
```

现在，运行`npm test`会得到一个通过的测试：

```js
 PASS lib/js/__tests__/First_test.bs.js
  Expect
    ✓ toBe (4ms)

Test Suites: 1 passed, 1 total
Tests: 1 passed, 1 total
Snapshots: 0 total
Time: 1.322s
Ran all test suites.
```

或者，如果我们想继续使用 ES 模块格式，我们需要确保 Jest 首先通过 Babel 运行`*test.bs.js`文件。为此，我们需要按照以下步骤进行：

1.  安装`babel-jest`和`babel-preset-env`：

```js
npm install babel-core@6.26.3 babel-jest@23.6.0 babel-preset-env@1.7.0
```

1.  在`.babelrc`中添加相应的 Babel 配置：

```js
/* .babelrc */
{
  "presets": ["env"]
}
```

1.  确保 Jest 通过 Babel 运行`node_modules`中的某些第三方依赖。出于性能原因，Jest 默认不会通过 Babel 运行`node_modules`中的任何内容。我们可以通过在`package.json`中提供自定义的 Jest 配置来覆盖这种行为。在这里，我们将告诉 Jest 只忽略不匹配`/node_modules/glennsl*`、`/node_modules/bs-platform*`等的第三方依赖：

```js
/* package.json */
...
"jest": {
 "transformIgnorePatterns": [
 "/node_modules/(?!@glennsl|bs-platform|bs-css|reason-react)"
 ]
}
```

现在，使用 ES 模块格式运行`npm test`可以正常工作：

```js
 PASS lib/es6/__tests__/First_test.bs.js
  Expect
    ✓ toBe (7ms)

Test Suites: 1 passed, 1 total
Tests: 1 passed, 1 total
Snapshots: 0 total
Time: 1.041s
Ran all test suites.
```

# 测试业务逻辑

让我们编写一个测试，验证我们能够通过`id`获取正确的顾客。在`Customer.re`中，有一个名为`getCustomer`的函数，接受一个`customers`数组，并通过调用`getId`来获取`id`。`getId`函数接受一个在`getCustomer`范围之外存在的`pathname`：

```js
let getCustomer = customers => {
  let id = getId(pathname);
  customers |> Js.Array.find(customer => customer.CustomerType.id == id);
};
```

我们立即注意到这不是理想的情况。如果`getCustomer`接受一个`customers`数组*和*一个`id`，并专注于通过他们的`id`获取顾客，那将会更好。否则，仅仅为`getCustomer`编写测试会更加困难。

因此，我们重构`getCustomer`，也接受一个`id`：

```js
let getCustomerById = (customers, id) => {
 customers |> Js.Array.find(customer => customer.CustomerType.id == id);
};
```

现在，我们可以更容易地编写测试。遵循编译器错误，确保你已经用`getCustomerById`替换了`getCustomer`。对于`id`参数，传入`getId(pathname)`。

让我们将我们的测试重命名为`__tests__/Customers_test.re`，并包括以下测试：

```js
open Jest;

describe("Customer", () =>
  Expect.(
    test("can create a customer", () => {
      let customers: array(CustomerType.t) = [|
        {
          id: 1,
          name: "Irita Camsey",
          address: {
            street: "69 Ryan Parkway",
            city: "Kansas City",
            state: "MO",
            zip: "00494",
          },
          phone: "8169271752",
          email: "icamsey0@over-blog.com",
        },
        {
          id: 2,
          name: "Luise Grayson",
          address: {
            street: "2756 Gale Trail",
            city: "Jacksonville",
            state: "FL",
            zip: "23566",
          },
          phone: "9044985243",
          email: "lgrayson1@netlog.com",
        },
        {
          id: 3,
          name: "Derick Whitelaw",
          address: {
            street: "45 Southridge Par",
            city: "Lexington",
            state: "KY",
            zip: "08037",
          },
          phone: "4079634850",
          email: "dwhitelaw2@fema.gov",
        },
      |];
      let customer: CustomerType.t =
        Customer.getCustomerById(customers, 2) |> Belt.Option.getExn;
      expect((customer.id, customer.name)) |> toEqual((2, "Luise 
       Grayson"));
    })
  )
);
```

使用现有代码运行这个测试（通过`npm test`）会导致以下错误：

```js
 FAIL lib/es6/__tests__/Customers_test.bs.js
  ● Test suite failed to run

    Error: No message was provided

Test Suites: 1 failed, 1 total
Tests: 0 total
Snapshots: 0 total
Time: 1.711s
Ran all test suites.
```

错误的原因是`Customers.re`在顶层调用了`localStorage`。

```js
/* Customer.re */
let customers = DataBsJson.(parse(getItem("customers"))); /* this is the problem */
```

由于 Jest 在 Node.js 中运行，我们无法访问浏览器 API。为了解决这个问题，我们可以将这个调用包装在一个函数中：

```js
/* Customer.re */
let getCustomers = () => DataBsJson.(parse(getItem("customers")));
```

我们可以在`initialState`中调用这个`getCustomers`函数。这将使我们能够在 Jest 中避免对`localStorage`的调用。

让我们更新`Customer.re`，将顾客数组移到状态中：

```js
/* Customer.re */
...
type state = {
  mode,
  customer: CustomerType.t,
  customers: array(CustomerType.t),
};

...

let getCustomers = () => DataBsJson.(parse(getItem("customers")));

let getCustomerById = (customers, id) => {
 customers |> Js.Array.find(customer => customer.CustomerType.id == id);
};

...

initialState: () => {
  let mode = Js.String.includes("create", pathname) ? Create : Update;
  let customers = getCustomers();
  {
    mode,
    customer:
      switch (mode) {
      | Create => getDefault(customers)
      | Update =>
        Belt.Option.getWithDefault(
          getCustomerById(customers, getId(pathname)),
          getDefault(customers),
        )
      },
    customers,
  };
},

...

/* within the reducer */
ReasonReact.UpdateWithSideEffects(
  {
    ...state,
    customer: {
      id: state.customer.id,
      name: getInputValue("input[name=name]"),
      address: {
        street: getInputValue("input[name=street]"),
        city: getInputValue("input[name=city]"),
        state: getInputValue("input[name=state]"),
        zip: getInputValue("input[name=zip]"),
      },
      phone: getInputValue("input[name=phone]"),
      email: getInputValue("input[name=email]"),
    },
  },
  self => {
    let customers =
      switch (self.state.mode) {
      | Create =>
        Belt.Array.concat(state.customers, [|self.state.customer|])
      | Update =>
        Belt.Array.setExn(
          state.customers,
          Js.Array.findIndex(
            customer =>
              customer.CustomerType.id == self.state.customer.id,
            state.customers,
          ),
          self.state.customer,
        );
        state.customers;
      };

    let json = customers->DataBsJson.toJson;
    DataBsJson.setItem("customers", json);
  },
);
```

在这些更改之后，我们的测试成功了：

```js
 PASS lib/es6/__tests__/Customers_test.bs.js
  Customer
    ✓ can create a customer (5ms)

Test Suites: 1 passed, 1 total
Tests: 1 passed, 1 total
Snapshots: 0 total
Time: 1.179s
Ran all test suites.
```

# 反思

在本章中，我们学习了如何使用 CommonJS 和 ES 模块格式设置`bs-jest`的基础知识。我们还了解到单元测试可以帮助我们编写更好的代码，因为大部分情况下，易于测试的代码也更好。我们将`getCustomer`重构为`getCustomerById`，并将顾客数组移到该组件的状态中。

由于我们在 Reason 中编写了单元测试，编译器也会检查我们的测试。例如，如果`Customer_test.re`使用`getCustomer`，而我们在`Customer.re`中将`getCustomerById`更改为`getCustomer`，我们会得到一个编译时错误：

```js
We've found a bug for you!
/__tests__/Customers_test.re 45:9-28

43  |];
44  let customer: CustomerType.t =
45  Customer.getCustomer(customers, 2) |> Belt.Option.getExn;
46  expect((customer.id, customer.name)) |> toEqual((2, "Luise Grayson")
      );
47  })

The value getCustomer can't be found in Customer

Hint: Did you mean getCustomers?
```

这意味着我们也无法编写某些单元测试。例如，如果我们想要测试第五章中的*Effective ML*代码，我们在那里使用类型系统来保证发票不会被打折两次，测试甚至无法编译。多么美好。

# 总结

由于 Reason 的影响如此广泛，有许多不同的学习方法。本书侧重于从前端开发人员的角度学习 Reason。我们学习了我们已经熟悉的技能和概念（如使用 ReactJS 构建 Web 应用程序），并探讨了如何在 Reason 中做同样的事情。在这个过程中，我们了解了 Reason 的类型系统、工具链和生态系统。

我相信 Reason 的未来是光明的。我们学到的许多技能都可以直接转移到针对本机平台。Reason 的前端故事目前比其本机故事更加完善，但已经可以编译到 Web 和本机。而且从现在开始，它只会变得更好。从我开始使用 Reason 的时候，已经有了巨大的改进，我很期待看到未来会有什么。

希望这本书能引起您对 Reason、OCaml 和 ML 语言家族的兴趣。Reason 的类型系统经过数十年的工程技术。因此，这本书没有涵盖的内容很多，我自己也在不断学习。然而，您现在应该已经建立了坚实的基础，可以继续学习。我鼓励您通过在 Discord 频道上提问、撰写博客文章、指导他人、在聚会中分享您的学习经历等公开学习。

非常感谢您能走到这一步，并在 Discord 频道上见到您！

Reason Discord 频道：

[`discord.gg/reasonml`](https://discord.gg/reasonml)
