# 第十四章：测试

自动化测试是软件开发的关键部分。尽管它不能（也不打算）取代手动测试和其他质量保证方法。当正确使用时，自动化测试是一个非常有价值的工具，可以避免回归、错误或不正确的功能。

软件开发是一门棘手的学科：尽管许多开发人员试图隔离软件的不同部分，但往往不可避免地，一些拼图的部分会对其他部分产生影响，无论是有意还是无意。

使用自动化测试的主要目标之一是检测新代码可能破坏先前工作功能的错误类型。这些测试被称为*回归测试*，当作为合并或部署过程的一部分触发时，它们是最有意义的。这意味着，如果自动化测试失败，合并或部署将被中断，从而避免向主代码库或生产环境引入新错误。

自动化测试还可以实现一种被称为*测试驱动开发（TDD）*的开发工作流程。在遵循 TDD 方法时，自动化测试是事先编写的，作为反映需求的非常具体的案例。新测试编写完成后，开发人员运行*所有测试*；新测试应该失败，因为尚未编写新代码。在这一点上，新代码必须被编写，以便新测试通过，同时不破坏旧测试。

如果正确执行测试驱动开发方法，可以提高对代码质量和需求符合的信心。它们还可以使重构甚至完整的代码迁移变得不那么冒险。

在本书中，我们将涵盖两种主要类型的自动化测试：单元测试和端到端测试。

# 单元测试

顾名思义，每个单元测试覆盖一个特定的功能。处理单元测试时最重要的原则是：

+   **隔离；**每个组件必须在没有任何其他相关组件的情况下进行测试；它不能受到副作用的影响，同样，它也不能产生任何副作用。

+   **可预测性；**每个测试必须产生相同的结果，只要输入不改变。

在许多情况下，遵守这两个原则意味着*模拟*（即模拟组件依赖的功能）。

## 工具

与 Angular 不同，Nest.js 没有用于运行测试的“官方”工具集；这意味着我们可以自由设置我们自己的工具，用于在 Nest.js 项目中运行自动化测试。

JavaScript 生态系统中有多个专注于编写和运行自动化单元测试的工具。典型的解决方案涉及使用多个不同的包进行设置，因为这些包在范围上受限（一个用于测试运行，第二个用于断言，第三个用于模拟，甚至可能还有一个用于代码覆盖报告）。

然而，我们将使用[ Jest ](https://facebook.github.io/jest/)，这是来自 Facebook 的“一体化”，“零配置”测试解决方案，大大减少了运行自动化测试所需的配置工作量。它还官方支持 TypeScript，因此非常适合 Nest.js 项目！

## 准备

正如您所期望的，Jest 被分发为一个 npm 包。让我们在我们的项目中安装它。从命令行或终端运行以下命令：

```js
npm install --save-dev jest ts-jest @types/jest

```

我们正在安装三个不同的 npm 包作为开发依赖项：Jest 本身；`ts-jest`，它允许我们在 TypeScript 代码中使用 Jest；以及 Jest 的类型定义，对于我们的 IDE 体验做出了宝贵的贡献！

还记得我们提到 Jest 是一个“零配置”测试解决方案吗？这是他们主页上宣称的。不幸的是，这并不完全正确：在我们能够运行测试之前，我们仍然需要定义一些配置。在我们的情况下，这主要是因为我们使用了 TypeScript。另一方面，我们需要编写的配置实际上并不多，所以我们可以将其编写为一个普通的 JSON 对象。

所以，让我们在项目的根文件夹中创建一个名为 `nest.json` 的新的 JSON 文件。

**`/nest.json`**

```js
{
  "moduleFileExtensions": ["js", "ts", "json"],
  "transform": {
    "^.+\\.ts": "<rootDir>/node_modules/ts-jest/preprocessor.js"
  },
  "testRegex": "/src/.*\\.(test|spec).ts",
  "collectCoverageFrom": [
    "src/**/*.ts",
    "!**/node_modules/**",
    "!**/vendor/**"
  ],
  "coverageReporters": ["json", "lcov", "text"]
}

```

这个小的 JSON 文件设置了以下配置：

1.  将 `.js`、`.ts` 和 `.json` 文件作为我们应用程序的模块（即代码）进行了配置。你可能会认为我们不需要 `.js` 文件，但事实上，由于 Jest 自身的一些依赖关系，我们的代码没有这个扩展名就无法运行。

1.  告诉 Jest 使用 `ts-jest` 包处理扩展名为 `.ts` 的文件（这个包在之前已经从命令行安装过）。

1.  指定我们的测试文件将位于 `/src` 文件夹中，并且将具有 `.test.ts` 或 `.spec.ts` 文件扩展名。

1.  指示 Jest 从 `/src` 文件夹中的任何 `.ts` 文件生成代码覆盖报告，同时忽略 `node_modules` 和 `vendor` 文件夹中的内容。此外，生成的覆盖报告格式为 `JSON` 和 `LCOV`。

最后，在我们开始编写测试之前的最后一步是向你的 `package.json` 文件中添加一些新的脚本：

```js
{
  ...
  "scripts": {
    ...
    "test": "jest --config=jest.json",
    "test:watch": "jest --watch --config=jest.json",
    ...
  }
}

```

这三个新的脚本将分别：运行一次测试，以观察模式运行测试（它们将在每次文件保存后运行），以及运行测试并生成代码覆盖报告（将输出到一个 `coverage` 文件夹中）。

**注意：** Jest 将其配置作为 `package.json` 文件中的 `jest` 属性接收。如果你决定以这种方式做事情，你将需要在你的 npm 脚本中省略 `--config=jest.json` 参数。

我们的测试环境已经准备好了。如果我们现在在项目文件夹中运行 `npm test`，我们很可能会看到以下内容：

```js
No tests found
In /nest-book-example
  54 files checked.
  testMatch:  - 54 matches
  testPathIgnorePatterns: /node_modules/ - 54 matches
  testRegex: /src/.*\.(test|spec).ts - 0 matches
Pattern:  - 0 matches
npm ERR! Test failed.  See above for more details.

```

测试失败了！好吧，其实并没有失败；我们只是还没有写任何测试！现在让我们来写一些测试。

## 编写我们的第一个测试

如果你已经阅读了本书的更多章节，你可能还记得我们的博客条目以及我们为它们编写的代码。让我们回顾一下 `EntryController`。根据章节的不同，代码看起来可能是这样的：

**`/src/modules/entry/entry.controller.ts`**

```js
import { Controller, Get, Post, Param } from '@nestjs/common';

import { EntriesService } from './entry.service';

@Controller('entries')
export class EntriesController {
  constructor(private readonly entriesSrv: EntriesService) {}

  @Get()
  findAll() {
    return this.entriesSrv.findAll();
  }
  ...
}

```

请注意，这个控制器是 `EntriesService` 的一个依赖项。由于我们提到每个组件都必须在*隔离*中进行测试，我们需要模拟它可能具有的任何依赖项；在这种情况下，是 `EntriesService`。

让我们为控制器的 `findAll()` 方法编写一个单元测试。我们将使用一个名为 `@nestjs/testing` 的特殊 Nest.js 包，它将允许我们为测试专门包装我们的服务在一个 Nest.js 模块中。

此外，遵循约定并将测试文件命名为 `entry.controller.spec.ts`，并将其放在 `entry.controller.ts` 文件旁边，这样当我们触发测试运行时 Jest 就能正确地检测到它。

**`/src/modules/entry/entry.controller.spec.ts`**

```js
import { Test } from '@nestjs/testing';
import { EntriesController } from './entry.controller';
import { EntriesService } from './entry.service';

describe('EntriesController', () => {
  let entriesController: EntriesController;
  let entriesSrv: EntriesService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [EntriesController],
    })
      .overrideComponent(EntriesService)
      .useValue({ findAll: () => null })
      .compile();

    entriesSrv = module.get<EntriesService>(EntriesService);
    entriesController = module.get<EntriesController>(EntriesController);
  });
});

```

现在让我们仔细看一下测试代码实现了什么。

首先，我们在 `describe('EntriesController', () => {` 上声明了一个测试套件。我们还声明了一些变量，`entriesController` 和 `entriesSrv`，分别用来保存被测试的控制器本身以及控制器所依赖的服务。

接下来是 `beforeEach` 方法。该方法中的代码将在每个测试运行之前执行。在这段代码中，我们为每个测试实例化了一个 Nest.js 模块。请注意，这是一种特殊类型的模块，因为我们使用了来自 `@nestjs/testing` 包的 `Test` 类的 `.createTestingModule()` 方法。因此，让我们把这个模块看作是一个“模拟模块”，它只用于测试目的。

现在是有趣的部分：我们在测试模块中将`EntriesController`作为控制器包含进来。然后我们继续使用：

```js
.overrideComponent(EntriesService)
.useValue({ findAll: () => null })

```

这替换了原始的`EntryService`，它是我们测试的控制器的一个依赖项。这是服务的模拟版本，甚至不是一个类，因为我们不需要它是一个类，而是一个没有参数并返回 null 的`findAll`方法的对象。

您可以将上述两行代码的结果视为一个空的、愚蠢的服务，它只重复我们以后需要使用的方法，而没有任何实现内部。

最后，`.compile()`方法是实际实例化模块的方法，因此它绑定到`module`常量。

一旦模块正确实例化，我们就可以将先前的`entriesController`和`entriesSrv`变量绑定到模块内控制器和服务的实例上。这是通过调用`module.get`方法实现的。

一旦所有这些初始设置都完成了，我们就可以开始编写一些实际的测试了。让我们实现一个检查我们的控制器中的`findAll()`方法是否正确返回条目数组的测试，即使我们只有一个条目：

```js
import { Test } from '@nestjs/testing';
import { EntriesController } from './entry.controller';
import { EntriesService } from './entry.service';

describe('EntriesController', () => {
  let entriesController: EntriesController;
  let entriesSrv: EntriesService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [EntriesController],
    })
      .overrideComponent(EntriesService)
      .useValue({ findAll: () => null })
      .compile();

    entriesSrv = module.get<EntriesService>(EntriesService);
    entriesController = module.get<EntriesController>(EntriesController);
  });

  describe('findAll', () => {
    it('should return an array of entries', async () => {
      expect(Array.isArray(await entriesController.findAll())).toBe(true);
    });
  });
});

```

`describe('findAll', () => {`行是开始实际测试套件的行。我们期望`entriesController.findAll()`的解析值是一个数组。这基本上是我们最初编写代码的方式，所以应该可以工作，对吧？让我们用`npm test`运行测试并检查测试输出。

```js
FAIL  src/modules/entry/entry.controller.spec.ts
  EntriesController
    findAll
      ✕ should return an array of entries (4ms)

  ● EntriesController › findAll › should return an array of entries

    expect(received).toBe(expected) // Object.is equality

    Expected value to be:
      true
    Received:
      false

      30 |       ];
      31 |       // jest.spyOn(entriesSrv, 'findAll').mockImplementation(() => result);
    > 32 |       expect(Array.isArray(await entriesController.findAll())).toBe(true);
      33 |     });
      34 |
      35 |     // it('should return the entries retrieved from the service', async () => {

      at src/modules/entry/entry.controller.spec.ts:32:64
      at fulfilled (src/modules/entry/entry.controller.spec.ts:3:50)

Test Suites: 1 failed, 1 total
Tests:       1 failed, 1 total
Snapshots:   0 total
Time:        1.112s, estimated 2s
Ran all test suites related to changed files.

```

它失败了... 好吧，当然失败了！记得`beforeEach()`方法吗？

```js
...
.overrideComponent(EntriesService)
.useValue({ findAll: () => null })
.compile();
...

```

我们告诉 Nest.js 将服务中的原始`findAll()`方法替换为另一个只返回`null`的方法。我们需要告诉 Jest 用返回数组的东西来模拟该方法，以便检查当`EntriesService`返回一个数组时，控制器实际上也将该结果作为数组返回。

```js
...
describe('findAll', () => {
  it('should return an array of entries', async () => {
    jest.spyOn(entriesSrv, 'findAll').mockImplementationOnce(() => [{}]);
    expect(Array.isArray(await entriesController.findAll())).toBe(true);
  });
});
...

```

为了模拟服务中的`findAll()`方法，我们使用了两个 Jest 方法。`spyOn()`接受一个对象和一个方法作为参数，并开始监视该方法的执行（换句话说，设置一个*spy*）。`mockImplementationOnce()`，顾名思义，当下一次调用该方法时改变方法的实现（在这种情况下，我们将其更改为返回一个空对象的数组）。

让我们尝试再次用`npm test`运行测试：

```js
 PASS  src/modules/entry/entry.controller.spec.ts
  EntriesController
    findAll
      ✓ should return an array of entries (3ms)

Test Suites: 1 passed, 1 total
Tests:       1 passed, 1 total
Snapshots:   0 total
Time:        1.134s, estimated 2s
Ran all test suites related to changed files.

```

测试现在通过了，因此您可以确信控制器上的`findAll()`方法将始终表现自如，并返回一个数组，以便依赖于该输出为数组的其他代码组件不会自己破坏。

如果这个测试在将来的某个时刻开始失败，那将意味着我们在代码库中引入了一个回归。自动化测试的一个很大的好处是，在为时已晚之前，我们将收到有关此回归的通知。

## 测试相等性

直到这一点，我们可以确定`EntriesController.findAll()`返回一个数组。我们无法确定它不是一个空对象数组，或者一个布尔值数组，或者只是一个空数组。换句话说，我们可以将该方法重写为`findAll() { return []; }`，测试仍然会通过。

因此，让我们改进我们的测试，以检查该方法是否真的返回了来自服务的输出，而不会搞乱事情。

```js
import { Test } from '@nestjs/testing';
import { EntriesController } from './entry.controller';
import { EntriesService } from './entry.service';

describe('EntriesController', () => {
  let entriesController: EntriesController;
  let entriesSrv: EntriesService;

  beforeEach(async () => {
    const module = await Test.createTestingModule({
      controllers: [EntriesController],
    })
      .overrideComponent(EntriesService)
      .useValue({ findAll: () => null })
      .compile();

    entriesSrv = module.get<EntriesService>(EntriesService);
    entriesController = module.get<EntriesController>(EntriesController);
  });

  describe('findAll', () => {
    it('should return an array of entries', async () => {
      jest.spyOn(entriesSrv, 'findAll').mockImplementationOnce(() => [{}]);
      expect(Array.isArray(await entriesController.findAll())).toBe(true);
    });

    it('should return the entries retrieved from the service', async () => {
      const result = [
        {
          uuid: '1234567abcdefg',
          title: 'Test title',
          body:
            'This is the test body and will serve to check whether the controller is properly doing its job or not.',
        },
      ];
      jest.spyOn(entriesSrv, 'findAll').mockImplementationOnce(() => result);

      expect(await entriesController.findAll()).toEqual(result);
    });
  });
});

```

我们保留了大部分测试文件之前的内容，尽管我们添加了一个新的测试，最后一个测试，在其中：

+   我们设置了一个包含一个*非空*对象（`result`常量）的数组。

+   我们再次模拟服务的`findAll()`方法的实现，以返回该`result`。

+   我们检查控制器在调用时是否确实像原始的那样返回`result`对象。请注意，我们使用了 Jest 的`.toEqual()`方法，它与`.toBe()`不同，它对两个对象的所有属性进行深度相等比较。

这是我们再次运行`npm test`时得到的结果：

```js
 PASS  src/modules/entry/entry.controller.spec.ts
  EntriesController
    findAll
      ✓ should return an array of entries (2ms)
      ✓ should return the entries retrieved from the service (1ms)

Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        0.935s, estimated 2s
Ran all test suites related to changed files.

```

我们的两个测试都通过了。我们已经取得了相当大的成就。现在我们有了一个坚实的基础，将测试扩展到尽可能多的测试用例将是一项容易的任务。

当然，我们只为一个控制器编写了一个测试。但测试服务和我们的 Nest.js 应用程序的其余部分的工作方式是相同的。

## 在测试中覆盖我们的代码

代码自动化中的一个关键方面是代码覆盖报告。因为，你怎么知道你的测试实际上覆盖了尽可能多的测试用例？嗯，答案就是检查代码覆盖率。

如果您希望对您的测试作为回归检测系统有真正的信心，确保它们尽可能多地覆盖功能。让我们想象一下，我们有一个有五个方法的类，我们只为其中两个编写了测试。我们大约覆盖了五分之二的代码，这意味着我们对另外三分之二没有任何了解，也不知道随着代码库的不断增长它们是否仍然有效。

代码覆盖引擎分析我们的代码和测试，并检查测试套件中运行的测试覆盖的行数、语句和分支的数量，返回一个百分比值。

如前几节所述，Jest 已经默认包含代码覆盖报告，您只需要通过向`jest`命令传递`--coverage`参数来激活它。

让我们在`package.json`文件中添加一个脚本，当执行时将生成覆盖报告：

```js
{
  ...
  "scripts": {
    ...
    "test:coverage":"jest --config=jest.json --coverage --coverageDirectory=coverage",
    ...
  }
}

```

在之前编写的控制器上运行`npm run test:coverage`，您将看到以下输出：

```js
 PASS  src/modules/entry/entry.controller.spec.ts
  EntriesController
    findAll
      ✓ should return an array of entries (9ms)
      ✓ should return the entries retrieved from the service (2ms)

---------------------|----------|----------|----------|----------|-------------------|
File                 |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
---------------------|----------|----------|----------|----------|-------------------|
All files            |      100 |    66.67 |      100 |      100 |                   |
 entry.controller.ts |      100 |    66.67 |      100 |      100 |                 6 |
---------------------|----------|----------|----------|----------|-------------------|
Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        4.62s
Ran all test suites.

```

为了更好地了解本书中的控制台输出，我们将把控制台输出转换成一个合适的表格。

| 文件 | % 语句 | % 分支 | % 函数 | % 行 | 未覆盖的行号 |
| --- | --- | --- | --- | --- | --- |
| 所有文件 | 100 | 66.67 | 100 | 100 |  |
| entry.controller.ts | 100 | 66.67 | 100 | 100 | 6 |

我们可以很容易地看到，我们在测试中覆盖了 100%的代码行。这是有道理的，因为我们为控制器中唯一的方法编写了两个测试。

### 覆盖率低的失败测试

现在想象一下，我们在一个复杂的项目中与几个开发人员同时在同一个基础上工作。还要想象我们的工作流程包括一个持续集成/持续交付流水线，运行在像 Travis CI、CircleCI 甚至 Jenkins 之类的东西上。我们的流水线可能包括一个在合并或部署之前运行我们的自动化测试的步骤，这样如果测试失败，流水线就会中断。

在这个想象中的项目中工作的所有虚构开发人员将会添加（以及重构和删除，但这些情况并不适用于这个例子）新功能（即*新代码*），但他们可能会忘记对该代码进行适当的测试。那么会发生什么？项目的覆盖百分比值会下降。

为了确保我们仍然可以依赖我们的测试作为回归检测机制，我们需要确保覆盖率永远不会*太低*。什么是太低？这实际上取决于多个因素：项目和其使用的技术栈、团队等。然而，通常一个很好的经验法则是在每个编码过程迭代中不要让覆盖率值下降。

无论如何，Jest 允许您为测试指定覆盖率阈值：如果值低于该阈值，测试将返回失败*即使它们都通过了*。这样，我们的 CI/CD 流水线将拒绝合并或部署我们的代码。

覆盖率阈值必须包含在 Jest 配置对象中；在我们的情况下，它位于项目根文件夹中的`jest.json`文件中。

```js
{
  ...
  "coverageThreshold": {
    "global": {
      "branches": 80,
      "functions": 80,
      "lines": 80,
      "statements": 80
    }
  }
}

```

传递给对象的每个属性的数字都是百分比值；如果低于这个值，测试将失败。

为了演示，让我们以以上设置运行我们的控制器测试。`npm run test:coverage`返回如下结果：

```js
 PASS  src/modules/entry/entry.controller.spec.ts
  EntriesController
    findAll
      ✓ should return an array of entries (9ms)
      ✓ should return the entries retrieved from the service (1ms)

---------------------|----------|----------|----------|----------|-------------------|
File                 |  % Stmts | % Branch |  % Funcs |  % Lines | Uncovered Line #s |
---------------------|----------|----------|----------|----------|-------------------|
All files            |      100 |    66.67 |      100 |      100 |                   |
 entry.controller.ts |      100 |    66.67 |      100 |      100 |                 6 |
---------------------|----------|----------|----------|----------|-------------------|
Jest: "global" coverage threshold for branches (80%) not met: 66.67%
Test Suites: 1 passed, 1 total
Tests:       2 passed, 2 total
Snapshots:   0 total
Time:        2.282s, estimated 4s
Ran all test suites.
npm ERR! code ELIFECYCLE
npm ERR! errno 1
npm ERR! nest-book-example@1.0.0 test:coverage: `jest --config=jest.json --coverage --coverageDirectory=coverage`
npm ERR! Exit status 1
npm ERR!
npm ERR! Failed at the nest-book-example@1.0.0 test:coverage script.
npm ERR! This is probably not a problem with npm. There is likely additional logging output above.

```

正如你所看到的，测试通过了，但是进程以状态 1 失败并返回错误。此外，Jest 报告说`"全局"分支覆盖率阈值（80%）未达到：66.67%`。我们已成功将不可接受的代码覆盖率远离了我们的主分支或生产环境。

接下来的步骤可能是实现一些端到端测试，以及我们的单元测试，以改进我们的系统。

# 端到端测试

尽管单元测试根据定义是孤立和独立的，端到端（或 E2E）测试在某种程度上具有相反的功能：它们旨在检查系统作为一个整体的健康状况，并尝试包括尽可能多的解决方案组件。因此，在 E2E 测试中，我们将专注于测试完整的模块，而不是孤立的组件或控制器。

## 准备工作

幸运的是，我们可以像对单元测试一样使用 Jest 进行 E2E 测试。我们只需要安装`supertest` npm 包来执行 API 请求并断言它们的结果。通过在控制台中运行`npm install --save-dev supertest`来安装它。

另外，我们将在项目的根目录下创建一个名为`e2e`的文件夹。这个文件夹将保存所有的 E2E 测试文件，以及它们的配置文件。

这将带我们到下一步：在`e2e`文件夹内创建一个名为`jest-e2e.json`的新文件，内容如下：

```js
{
  "moduleFileExtensions": ["js", "ts", "json"],
  "transform": {
    "^.+\\.tsx?$": "<rootDir>/node_modules/ts-jest/preprocessor.js"
  },
  "testRegex": "/e2e/.*\\.(e2e-test|e2e-spec).ts|tsx|js)$",
  "coverageReporters": ["json", "lcov", "text"]
}

```

正如你所看到的，新的 E2E 配置对象与单元测试的对象非常相似；主要区别在于`testRegex`属性，它现在指向`/e2e/`文件夹中具有`.e2e-test`或`e2e.spec`文件扩展名的文件。

准备的最后一步将是在我们的`package.json`文件中包含一个 npm 脚本来运行端到端测试：

```js
{
  ...
  "scripts": {
    ...
    "e2e": "jest --config=e2e/jest-e2e.json --forceExit"
  }
  ...
}

```

## 编写端到端测试

使用 Jest 和 Nest.js 编写端到端测试的方式也与我们用于单元测试的方式非常相似：我们使用`@nestjs/testing`包创建一个测试模块，我们覆盖`EntriesService`的实现以避免需要数据库，然后我们准备运行我们的测试。

让我们来编写测试的代码。在`e2e`文件夹内创建一个名为`entries`的新文件夹，然后在其中创建一个名为`entries.e2e-spec.ts`的新文件，内容如下：

```js
import { INestApplication } from '@nestjs/common';
import { Test } from '@nestjs/testing';
import * as request from 'supertest';

import { EntriesModule } from '../../src/modules/entry/entry.module';
import { EntriesService } from '../../src/modules/entry/entry.service';

describe('Entries', () => {
  let app: INestApplication;
  const mockEntriesService = { findAll: () => ['test'] };

  beforeAll(async () => {
    const module = await Test.createTestingModule({
      imports: [EntriesModule],
    })
      .overrideComponent(EntriesService)
      .useValue(mockEntriesService)
      .compile();

    app = module.createNestApplication();
    await app.init();
  });

  it(`/GET entries`, () => {
    return request(app.getHttpServer())
      .get('/entries')
      .expect(200)
      .expect({
        data: mockEntriesService.findAll(),
      });
  });

  afterAll(async () => {
    await app.close();
  });
});

```

让我们回顾一下代码的功能：

1.  `beforeAll`方法创建一个新的测试模块，在其中导入`EntriesModule`（我们将要测试的模块），并用非常简单的`mockEntriesService`常量覆盖`EntriesService`的实现。一旦完成，它使用`.createNestApplication()`方法创建一个实际运行的应用程序来进行请求，然后等待其初始化。

1.  `'/GET entries'`测试使用 supertest 执行对`/entries`端点的 GET 请求，然后断言该请求的响应状态码是否为`200`，并且接收到的响应体是否与`mockEntriesService`常量的值匹配。如果测试通过，这意味着我们的 API 正确地响应了收到的请求。

1.  `afterAll`方法在所有测试运行完毕时结束了我们创建的 Nest.js 应用程序。这很重要，以避免在下次运行测试时产生副作用。

# 总结

在本章中，我们探讨了向我们的项目添加自动化测试的重要性以及它带来的好处。

另外，我们开始使用 Jest 测试框架，并学习了如何配置它，以便与 TypeScript 和 Nest.js 无缝使用。

最后，我们回顾了 Nest.js 为我们提供的测试工具，并学习了如何编写测试，包括单元测试和端到端测试，以及如何检查我们的测试覆盖了多少代码的百分比。

在下一章中，我们将介绍使用 Angular Universal 进行服务器端渲染。
