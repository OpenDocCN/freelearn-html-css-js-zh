

# 第十四章：时间旅行者的困境——基于状态的端到端用户旅程

本章讨论了不同但常见的一种测试自动化类型——逐页导航通过调查，以达到一个共同终点。这里的挑战是，用户在途中做出的决定可能会改变下游显示的页面顺序——如果它们甚至出现的话。在这种情况下，我们需要一个灵活的用户旅程，可以走许多分支到达一个共同终点，并且能够报告是否有路径以错误页面结束。尝试为端到端自动化创建一个不断扩展的页面路径是不高效的。`if`/`then`分支或`select`/`case`选项的数量将是无限的且复杂。为了解决这个问题，我们将探讨通过 URL 识别每一页的方法，并查看解耦页面路径的方法，允许测试以任何顺序处理顺序页面，同时保持最小维护。想象这种方法就像是一个球在大型弹珠游戏中的多个柱子之间随机弹跳。

本章涵盖了以下主要主题：

+   按章节划分

+   快乐路径

+   使用`getPageName()`从当前 URL 中提取页面名称

+   页面处理循环

+   常见退出点

# 技术要求

所有测试示例都可以在这个 GitHub 仓库中找到：[`github.com/PacktPublishing/Enhanced-Test-Automation-with-WebdriverIO`](https://github.com/PacktPublishing/Enhanced-Test-Automation-with-WebdriverIO)。

# 分而治之！

在这种方法中，我们将考虑一种非确定性的导航方式，从起点到终点。我们不会基于一个页面跟随另一个页面的传统假设。相反，我们的脚本将反复通过每个潜在的页面，并且只有当当前页面匹配页面类之一时才会进行交互。然后，根据高级流程要求，我们可以做出不同的选择，甚至输入不同的和独特的数据。我们甚至可能会在途中发现一个错误！

在这个小型示例中，我们将自动化 Candymapper 网站的万圣节派对功能。这为顾客提供了一组有限的选项来计划主题派对。派对客人可以使用相同的网站参加或避免这些令人毛骨悚然的活动。以下路径包括客户和客人：

+   客户可以选择举办僵尸主题派对

+   客户可以选择举办幽灵主题派对

+   客人可以选择参加 Zombieton 派对

+   参加幽灵镇派对

+   惊恐的客人可以退出到**主持人或参加者**选择页面

至少这些路径中的一个会在[www.Candymapper.com](http://www.Candymapper.com)生产网站上出错。这次旅程在新网站的发布中得到了修正，即[www.CandymapperR2.com](http://www.CandymapperR2.com)，以及页面顺序和为参加派对的客人提供的新的带朋友选项。

在这四条路径中的前四条中，我们将结束在共同的**派对倒计时**计时器页面。在最后一条路径中，我们将回到**万圣节派对**举办者或参加者页面。我们将遇到的页面包括以下操作：

+   **主页**：清除弹出窗口，并在页面标题中点击**万圣节派对**。

+   **万圣节派对页面**：选择举办或参加派对

+   **举办派对页面**：地点地址和“**了解更多**”按钮（仅限宾客路径）

+   **派对地点**：一个可以选择 Zombieton 或 Ghostville 的页面

+   **参加派对页面**：有两个地点选择和一个**害怕**返回按钮

+   **派对地点选择页面**：Zombieton 或 Ghostville：

    +   （仅限开发者）宾客名单

    +   一个带有“**提醒我**”按钮的电子邮件字段（仅限开发者）

+   **派对倒计时页面**：最终目的地

此测试的核心是`PartyPath()`驱动程序，它接受一个可以解析以在多个里程碑处进行决策点更改的单个字符串。这个函数是一个巨大的循环，反复遍历每个已知的页面。对于每个页面，我们将创建一个对象模型和一个`build()`方法。此方法仅在当前页面与类页面类型匹配时执行。如果所有操作都成功，则返回`true`。如果当前页面不是此页面，则是一个`null`函数并返回`false`。此方法将引用一个预先配置了快乐路径数据的测试数据模型，可以从`Switchboard`对象中覆盖。这将允许从开始到结束修改页面流程中的选择。

# 简化动态旅程的复杂性

在现实世界的应用中，我们可能会在多个具有不同数据要求的求职申请页面中导航。一些决策点可能包括添加当前地址、军事状态、前雇主联系信息和个人推荐人。申请流程可能包括为 21 岁以下的人或需要证明专业驾驶执照以驾驶卡车的驾驶员提供额外的页面。对于单身、已婚、离婚或丧偶且有子女的人，多个路径可能会要求提供越来越多的信息。

挑战在于没有人能预测用户旅程中的下一页，最终将引导到最终成功的页面目的地。然而，每个里程碑都可以用高级术语来描述，这些术语表明在每个决策点应该做什么。

如果我们有一个术语字典，可以按几乎任何顺序传递给测试，描述申请人在现实世界工作中申请的类型？

+   `快乐路径`：单身、平民、21 岁以上、无子女、无工作历史或推荐人

+   `已婚子女 cdl`：此路径包括配偶、子女和 CDL 驾驶执照信息

+   `军事未成年 cu dd`：使用公司已知的缩写词**信用合作社**和**直接存款**

对于万圣节派对的例子，我们将遵循以下路径：

+   `快乐路径`：举办一个以僵尸为主题的派对，倒计时计时器作为最终目的地

+   `Attend`/ `""Attend zombie"":` 以僵尸主题的决策点参加派对，而不是默认的幽灵主题

+   `Scared`: 开始参加派对的路径，但返回到主选择“主机”或“参加”选择页面

这种方法的秘密是一个路径生成器，`partyPath()`。这是一个循环，它反复执行页面对象模型中每个页面的 `build()` 方法，直到达到最终页面。每个页面都会确定当前页面是否是处理页面。然后，使用测试数据与该页面的值进行交互。每个字段和列表填充方法将在它们各自的对象上执行。如果没有为特定输入字段提供数据，则执行交互。

退出无限循环将有四个原因：

+   达到了最终的派对倒计时页面（`快乐路径`）

+   遇到了一个未知页面

+   在两次循环尝试后，页面保持不变

+   恐惧之旅会遇到“主机”或“参加”页面

最终，`build()` 方法将始终尝试显式地移动到下一页。在大多数情况下，这将是一个 `Page` 类，所有其他页面都从中扩展。在其他情况下，移动到下一页将是通过简单地输入最终所需的字段来实现，其中不会涉及前面章节中提到的任何 `clickIfExists()` 功能。目标始终是尝试到达过程的末尾，而不是寻找错误。

在某些情况下，页面可能不会继续前进，期望在做出选择后从用户那里获取更多信息。虽然这可能在同一路径内处理，但它可能是第二次遍历同一页面上检测到的路径。因此，如果在两次循环尝试后导航没有前进，测试将以用户路径不完整退出，并在 Allure 报告中详细说明问题。

我们需要添加的第一个组件是确定我们当前所在的页面。虽然这因项目而异，但我们经常发现可以在其 URL 中找到一个唯一的标识符。让我们从提取页面 URL 的最后一段中的唯一标识符开始：

+   [糖果地图者 2 的万圣节派对](https://candymapperr2.com/halloween-party)

+   [糖果地图者 2 的派对位置 1](https://candymapperr2.com/party-location-1)

+   [糖果地图者 2 的派对时间](https://candymapperr2.com/party-time)

要捕获框架导航到的 URL，`getPageName()` 函数将给出 URL 末尾的描述。以下函数将分别从上述 URL 返回 `halloween-party`、`party-location-1` 和 `party-time`。

```js
/**
 * Gets last segment of current URL after splitting by "/".
 * @returns {Promise<string>} The last URL segment.
 */
export async function getPageName(): Promise<string> {
  const currentURL = await browser.getUrl();
  const urlSegments = currentURL.split('/');
  return urlSegments[urlSegments.length - 1];
}
```

这将在主 `partyPath()` 循环中执行，并存储在自动化交换板对象的 `page` 键中。请注意，我们只使用 URL 的结尾部分，因为这允许我们在不同的环境中具有类似的功能：

+   [`candymapper.com/party-time`](https://candymapper.com/party-time)

+   [`candymapperr2.com/party-time`](https://candymapperr2.com/party-time)

每个页面类都包含可能出现在页面上的 `pages` 对象模型：

```js
import Page from './page.tjs';
import * as helpers from "../../helpers/helpers.tjs";
/**
 * sub page with selectors for a specific page
 */
class HalloweenPartyPage extends Page {
    /**
     * define selectors using getter methods
     */
    public get hostParty () {
        return $(`//a[contains(normalize-space(),'ost')]`);
    }
    public get attendParty () {
        return $(`//a[contains(normalize-space(),'ttend')]`);
    }
```

注意，通用的定位器使用 `normalize-space`，并且如果首字母大小写发生变化，还会跳过第一个字母。这使得它们更加健壮，不太可能因为文本从发布到发布的变化而变得过时。

接下来，我们有一个对所有页面都通用的 `build()` 方法。这个方法首先会检查 URL 字符串中的标识符是否与页面匹配，如果不匹配则立即返回 `false`：

```js
public async build () {
   // Is this the page to process?
   if (await ASB.get("page") !== "attend-a-party") {
      return false // Not the right page
   }
   if (ASB.get("hostorattend") === `attend`){
     return await helpers.clickAdv(
         await this.attendParty);
   }
   return helpers.clickAdv(await this.hostParty);
}
export default new HalloweenPartyPage();
```

此路径示例默认将举办派对。在 switchboard 中的 `HostOrAttend` 里程碑可以被切换，使其成为一个参加派对的用户旅程。点击后，出现的下一页将根据我们点击的按钮以及运行此测试的环境而有所不同。因此，我们在哪里解耦用户旅程中预期的页面至关重要，因为这可能并不总是相同的。然而，我们需要从某个地方开始，那就是 Happy Path。

我们将要引入的最后一个特性是一个 `beforeEach()` 函数，这将在编写测试时显著减少我们的代码。由于我们有多个测试用例，并且每次都编写 ```jsloginPage.open(``)```。这可以通过将函数放在 `beforeEach()` 函数中来实现。这个重构执行的是完全相同的过程，但代码行数更少：

```js
describe("Ch14: State-drive Automation - Host a Party (Default in Ghostville)",  () => {
  it("should loop around until the final page is found", async () => {
    await LoginPage.open(``);
    await stateDrivenUtils.partyPath("Host");
  });
});
describe("Ch14: State-drive Automation - Host a Party (Default in Ghostville)",  () => {
  it("should loop around until the final page is found", async () => {
    await LoginPage.open(``);
   await stateDrivenUtils.partyPath("Host");
  });
});
```

在这个重构版本中，我们将 `await ```jsLoginPage.open(``)```` 移入 `beforeEach(async ()` 函数中。现在，这段代码将在每个测试用例之前执行。

```js
beforeEach(async () => {
  await LoginPage.open(``);
});
describe("Ch14: State-drive Automation - Host a Party (Default in Ghostville)",  () => {
  it("should loop around until the final page is found", async () => {
    await stateDrivenUtils.partyPath("Host");
  });
});
describe("Ch14: State-drive Automation - Host a Party (Default in Ghostville)",  () => {
  it("should loop around until the final page is found", async () => {
    await stateDrivenUtils.partyPath("Host");
  });
});
```

请记住，这是因为我们不是在构建一个从一个页面跳转到另一个特定页面的测试。我们只是在填充当前活动的页面，尝试在循环中移动到下一个页面。让我们看看我们的状态驱动流程驱动程序 `pathyPath(testData)` – 它将导航到我们网站的里程碑决策点。

# Happy Path

如果 `testData` 参数为空，则将执行默认的 Happy Path。这配置在 `shared-data` 文件夹中的 `userData.json` 文件中。这两个里程碑包括 `HostOrAttend` 和 `location`：

```js
}
  "journeyData": {
    "_hostOrAttend_comment": "'host' (default Happy Path), 'attend' or 'scared' ",
    "hostOrAttend: "host",
    "_location_comment": "'zombieton' (default Happy Path) or 'ghostville' ",
    "location": "zombieton",
  }
}
```

经验法则 - 记录 JSON 文件

任何值得尊敬的开发者都不会在没有一些手头文档的情况下设计数据文件。然而，JSON 文件不允许我们像在 JavaScript 文件中使用双斜杠（`//`）那样添加注释。解决方案是添加一个以下划线开头并以单词“`comment`”结尾的匹配键。现在，每个键都可以被记录下来，以确保团队中的其他人可以理解设计背后的意图。

此文件首先被提取到 Switchboard 对象中。然后，任何覆盖的值都会从 `testData` 参数或 `JOURNEY` 系统变量中解析出来：

```js
  public parseTestData(testData: string = '') {
    if (process.env.JOURNEY !== undefined) {
      testData = process.env.JOURNEY;
    }
```

在这一点上，我们必须为未来做计划。在我们的最后一章中，我们将使用`JOURNEY`系统变量从 Jenkins 驱动这些测试用例。如果这个变量被填充，它将覆盖`testData`参数。

当我们构建这个驱动程序时，我们应该有一个已知公司术语的列表，这些术语可以用来调整里程碑决策点。

在我们的示例中，我们有`host`和`attend`作为路径动词，以及`zombie`、`ghost`和`scared`作为决策点：

```js
  if (testData != "") {
      testData = " " + testData.toLowerCase(); // Add space to make sure we match whole words and convert to lowercase once
// Overriding default values
      if (testData.includes(" host")) {
        ASB.set("hostOrAttend", "host");
      }
      if (testData.includes(" attend")) {
        ASB.set("hostOrAttend", "attend");
      }
      if (testData.includes(" zombie")) {
        ASB.set("location", "zombieton");
      }
      if (testData.includes(" ghost")) {
        ASB.set("location", "ghostville");
      }
      if (testData.includes(" scared")) {
        ASB.set("location", "scared");
      }
```

接下来，默认数据值会被解析到 ASB 交换板中：

```js
    parseToASB("path/to/userdata.json")
```

然后，`testData`字符串中的任何覆盖`key:value`数据都会被解析到 ASB 对象中：

```js
    parseToASB(testData)
```

在测试数据字符串内部，我们将允许测试为我们的交换板中的键分配特定的值。例如，我们可能想要将邮编设置为`12345`。在这种情况下，我们需要一个正则表达式来解析与等号相连的值。带有和不带有空格的字符串可以是`be address=""123 main""`以及`zip=12345`：

```js
export function parseToASB(testData: string) {
  const regex = /(\w+)=("([^"]*)"|\b\w+\b)/g;
  let match;
  while ((match = regex.exec(testData)) !== null) {
    let key = match[1];
    let value = match[2];
    // Remove quotes if present
    if (value.startsWith('"') && value.endsWith('"')) {
      value = value.slice(1, -1);
    }
    let keyLower = key.toLowerCase();
    let oldValue = ASB.get(keyLower);
    // Always save value as a string
    ASB.set(keyLower, value);
    if (oldValue !== undefined) {
      console.log(`ASB(«${keyLower}") updated from "${oldValue}" to "${ASB.get(keyLower)}"`);
    } else {
      console.log(`ASB(«${keyLower}") set to "${ASB.get(keyLower)}"`);
    }
  }
}
import fs from 'fs';
import xml2js from 'xml2js';
export async function parseXMLFileToASB(filePath: string) {
  const data = fs.readFileSync(filePath);
  const result = await xml2js.parseStringPromise(data);
  for (let key in result) {
    let newValue = result[key];
    let oldValue = ASB.get(key.toLowerCase());
    // Always save value as a string
    ASB.set(key.toLowerCase(), newValue);
    if (oldValue !== newValue) {
      console.log(`ASB(«${key.toLowerCase()}") updated from "${oldValue}" to "${newValue}"`);
    }
  }
}
```

虽然`testData`字符串参数告诉我们每个里程碑点将发生什么变化，但这个信息首先被转移到 ASB 交换板对象中，所有页面在执行时都会引用这个对象。数据文件将包含填充到交换板中的默认值，然后`testData`被解析以自定义这些值。为了准备我们的最后一章，这个函数还会从`JOURNEY`系统变量中获取数据，该数据将由 Jenkins 发送：

```js
import { ASB } from "../../helpers/globalObjects";
import candymapperPage from "../pageObjects/candymapper.page";
import halloweenAttendPartyPage from "../pageObjects/halloweenAttendParty.page";
import halloweenPartyPage from "../pageObjects/halloweenParty.page";
import halloweenPartyLocationPage from "../pageObjects/halloweenPartyLocation.page";
import halloweenPartyThemePage from "../pageObjects/halloweenPartyTheme.page";
import halloweenPartyTimerPage from "../pageObjects/halloweenPartyTimer.page";
import * as helpers from "../../helpers/helpers";
import AllureReporter from "@wdio/allure-reporter";
import Page from "../pageObjects/page";
class StateDrivenUtils extends Page {
```

## 主要驱动循环

`partyPath()`方法实际上是一个实用工具，而不是一个测试，因此我们将它存储在项目根目录下的`Utilities`文件夹中：

```js
export async function partyPath (testData) {
    let complete: Boolean = false;
    let lastPage: string = ""
    let retry = 2
   this.parseTestData(testData);  // Parse the known milestone verbs                                   // to the switchboard
    helpers.parseToASB(testData);   // Parse the key=value data to set                                     // the switchboard
```

下一步是导航万圣节派对的起始路径。这需要处理每个页面的`is a`循环，重复寻找四个出口点之一：

```js
    while (complete === false) { // Loop until final page
      //Get Page Name
        let pageName = await browser.getUrl();
        pageName = extractPathFromUrl(pageName)
        ASB.set("page", pageName);
```

这将当前页面的名称放入交换板中。这个对象将在每个页面的`build()`方法中被读取。接下来，我们将使用逻辑`OR`调用每个已知的页面，以确定是否有任何页面被成功识别：

```js
   // Pass through every known page
knownPage =
        await halloweenLocationPage.build() ||
        await halloweenAttendPartyPage.build() ||
   await halloweenHostPartyPage.build() ||
   await halloweenPartyPage.build() ||
// Add new pages along the journey here.
```

注意，这些页面不需要按照任何特定的顺序排列。实际上，我们的设计将第一个处理页面列表放在最后，其他页面则大致按照相反的顺序排列。这样做通常可以确保每次循环只解析一个页面。每次循环执行多个页面也是可以接受的。

经验法则——重构

在这个小型示例中，我们只使用了六个页面。现实世界的项目可能有 50 到 100 个网页需要导航。始终建议将代码缩减为更小、更易于管理的单元。在这种情况下，页面列表越长，就越应该将其重构为它们自己的函数。

现在我们添加四个出口点中的第一个。注意，在前面的页面列表中，有一个缺失——最后的倒计时页面。这是我们的旅程结束的成功页面。此时，其他任何东西都不再重要：

```js
// Exit Point #1: Success reached the timer page
   if (await halloweenPartyTimerPage.build()) {
      knownPage = true; // Skip Exit point 2
      console.log("Success: Reached the timer page")
      complete = true; // Exit the loop
 break;
   } // End of all paths except unknown page
```

如果这个页面被处理，我们报告旅程成功的结束。它还设置了`knownPage`标志为`true`，以便在函数结束时进行额外的报告或清理。然后，我们退出。

我们在这里的工作已经完成，除非有什么未知的东西潜伏在那里？

```js
// Exit Point #2: Unknown page encountered
   if (knownPage === false) {
     //None of the build methods returned true
     AllureReporter.addAttachment(`Unknown Page detected: ${pageName}`, "", "text/plain");
    console.log(`Unknown Page detected: ${pageName} - Exiting Journey`);
    expect(pageName).toBe("a known page");
    break;
}
```

在这种情况下，没有页面返回`true`。我们输出未知页面的名称。我们还使用一个巧妙的`expect`将测试设置为失败状态，同时进一步记录手头的问题，即新页面的 URL。在我们的例子中，当访客因为害怕参加派对而期望返回时，生产网站`Candymapper.com`将生成一个`Error 404`页面。这个页面从未被期望过，但如果我们能够截屏它。

接下来，我们需要处理一种情况，即页面并没有从上一个页面前进：

```js
// Exit Point #3: Page did not change
if (lastPage === pageName) {
  retry--; // Give two additional attempts
  if (retry === 0) {
    console.log(`Page did not change: ${lastPage} - Exiting Journey`);
    expect("Page did not move on from").toBe(lastPage);
  } else {
     // Page moved on, reset retry for next page
     retry = 2;
  }
}
```

作为超级英雄，我们总是对第二次机会持开放态度。在这种情况下，一个页面本身可能有两种或三种状态。一个销售点页面在购物车为空时可能添加一个产品，而在购物车有一个项目时添加不同的商品，然后继续前进。可能需要调用这样的方法两次，以提供灵活性。

最后，可能有一个期望以替代页面作为其最终目的地的旅程。这种情况很少见，但作为一个选项提供。这条路径是我们返回到过程中的一个先前步骤。这之所以有效，是因为页面在循环的第一步中被识别和处理，在到达这个点之前移动到下一个页面：

```js
// Exit Point #4: We were scared and went back - Halloween Party Home page reached - only works in dev, prod has an intentional Error 404 issue.
if (ASB.get("page") === "halloween-party") {
   console.log("Halloween Party Home page reached")
   complete = true;
}
```

这就是所有的退出点吗？我们确信还有其他方式旅程必须提前结束。我们可以想到另一个例子——一个无限循环的旅程。这需要跟踪所有访问过的 URL 数组。当页面第三次被访问时，它会被触发。我们已经给了你所有实现这一点的技巧和窍门。现在，是时候自己解决这个问题了。

在再次循环之前，我们的最后一步是将当前页面名称设置为最后一个页面名称：

```js
      lastPage = ASB.get("page"); // Save the last page name
    }
```

这确保了在确定我们是否卡在同一个页面上时，退出点#3 能够正确工作。

接下来，使用测试数据文件对象来确定哪些字段被填充。如果测试数据对象不包含特定字段的资料，我们会在结果中添加一个警告，但测试会继续。

最后，执行前进到下一个页面的方法。请注意，我们将使用`ClickIfExits()`方法来确保我们的引擎即使在未来的测试环境中元素不存在的情况下也能保持稳健。

在循环中，我们检查页面是否通过递增计数器（从 2 开始）前进。如果页面在第一次循环中没有前进，我们减少循环计数器。如果循环计数器达到 0，我们未能成功前进，并退出。如果连续的循环导致新的页面 URL，计数器将重置以执行两个额外的循环。

一些页面可能有代码可以解决由不完整数据生成的问题。这可能在页面`build`方法的第一次执行中未被检测到；因此，在放弃测试用户旅程之前，必须至少执行`build`方法两次。

完成每个页面的这个过程后，我们可以开始添加里程碑，基于从`JOURNEY`用户变量传递的值。在这个例子中，我们将有一个默认计划僵尸派对的路径，但将通过在`JOURNEY`字符串中传递`attend`来切换到用户参加派对。这将反过来注入到测试数据对象中，并由适当的页面读取。

如果我们遇到一个新页面，测试将停止。我们将向 Allure 报告页面的 URL，并提供结果的屏幕截图。我们的报告自动告诉我们导致那个新端点的路径。

## 一切都在细节中

同样，我们可以在我们的页面报告中通过屏幕截图检测 URL 中的字符串`error`，或者我们可以这样捕获屏幕上的所有文本 – `const allText =` `await $('body').getText();`。

捕获屏幕文本的另一种方式是将*Ctrl-A / Ctrl-V*发送到浏览器，并将剪贴板发送到 Allure 报告：

```js
import { clipboard } from 'electron';
const operatingSystem = process.platform;
let selectAllKeys: string[];
let copyKeys: string[];
if (operatingSystem === 'darwin') {
  selectAllKeys = ['Command', 'a'];
  copyKeys = ['Command', 'c'];
} else {
  selectAllKeys = ['Control', 'a'];
  copyKeys = ['Control', 'c'];
}
await browser.keys(selectAllKeys);
await browser.keys(copyKeys);
const allText = clipboard.readText();
```

这使得我们只捕获浏览器可见文本的可能性更大。

这个循环还提供了与任何已知页面交互的机会，无论它在路径中的顺序如何。考虑在 candymapperR2 网站上举办派对的这个路径：

1.  举办派对

1.  派对场地地址确认

1.  派对主题

1.  派对倒计时

考虑参加派对的路径：

1.  参加派对

1.  选择派对场地或返回

1.  派对场地地址

1.  派对倒计时

注意，这两条路径遇到了相同的**派对场地地址**页面，但来自不同的页面。此外，这两条路径与生产 Candymapper 网站不同。因此，我们可以在两个具有不同路径但目标相同的环境中运行相同的测试用例。

我们最终的路径是设置我们的最终预期页面。在这个例子中，路径返回到`Plan`或`Attend`页面，指示用户旅程；尽管它没有结束在共同端点，但仍然是一个成功。

# 改变决策点

我们通过单个环境变量传递自定义路径的字符串，如下所示：

```js
Set JOURNEY=""; yarn ch15
```

这样，我们可以创建多个测试路径。在这个例子中，用户没有参加派对，而是点击了**我害怕**按钮：

```js
Set JOURNEY="attend scared"; yarn ch15
```

在这个例子中，用户选择了以僵尸为主题的举办派对的路径：

```js
Set JOURNEY="Host ZOMBIE"; yarn ch15
```

单个环境值可以修改 Happy Path 基线中的多个决策点。虽然默认情况下空字符串将创建 Happy Path，但最佳实践是分配字符串，以便路径在结果中显示，解析不区分大小写。

```js
let journey: string = " " + (process.env.JOURNEY || "Host").toLowerCase();
if (journey===" ")) {
    journey = " host"; //Default Happy Path
}
```

注意，在旅程变量之前有一个前置空格。这样做的原因是为了降低与路径中类似字符串匹配的可能性。我们有一个`host`命令和一个`ghost`派对。编写这一行代码可能会从`ghost`字符串中获取`host`命令：

```js
Set JOURNEY="Attend Ghost"; yarn ch15
if (journey.includes(" host").toLowerCase()) {
// Host path being taken in error.
}
```

这已经解决了，因为每个被执行的命令总是有一个前置空格。现在，不正确路径匹配的机会更少了。以我们的示例为例，我们将更进一步，将`Host`作为默认的 Happy Path，将`Attend`作为路径的偏差：

```js
if (journey.includes(" attend").toLowerCase()) {
// Attend a party path, case insensitively
Helpers.click(attend)
} else {
    // The JOURNEY variable contains "host" Happy Path
Helpers.click(host)
}
```

这就完成了在第一页上选择决策点的第一种方法。函数退出并循环。下一页可能是下一个，但不必按顺序排列。它可能位于循环列表的更下方。所有后续的页面都不会匹配 URL，并且`build()`方法将简单地立即返回一个空函数。下一页可能在循环的更早位置。它会在这一页之前作为空函数执行，现在随着循环从顶部再次开始执行。

# 洗，冲，重复

同样的原则适用于修改页面类中的数据类型。例如，自定义的`date`令牌可以像这样传递：

```js
Set JOURNEY="host <today+7>"; yarn ch15
const match = journey.match(/(<.+>)/);
const dateToken = match ? match[0] : "";
Helpers.setValueIfExists(dateField, dateToken);
```

现在，我们已经从旅程值中提取了一个日期令牌。如果存在，令牌将传递到`dateField`，并设置为下周的日期。如果没有令牌，日期将设置为空字符串，方法将立即返回，因为没有要做的事情。

到目前为止，我们已经涵盖了非确定性引擎的所有过程。新页面将在结果中报告，并且必须添加以扩展路径覆盖率。如果一个页面在特定路径中从未遇到，它不会停止测试。如果遇到错误，它将被报告。如果路径卡住，它也将被报告。数据和路径可以自定义。这些构建方法可以从其他测试用例中调用。

# 为什么不用 API 调用生成这些工件呢？

使用 API 调用进行任务确实意味着更快、更稳定的测试，因为它可以直接与应用程序通信。它可以在开发过程的早期实现。这可能导致问题的早期发现，使开发过程更加灵活和敏捷。

相反，使用自动化的 GUI 可以提供对用户体验的更深入理解，因为它可以模拟用户可能采取的确切路径，包括与 API 测试可能忽略的视觉元素的交互。这种方法可能更直观，并且可以涵盖更广泛的分析，包括外观和布局，这对于用户满意度至关重要。

实际上，没有理由支持一种方法而不是另一种方法。GUI 方法确认系统对于用户来说是正确工作的。API 方法以更快的速度生成工件。我们可以通过在旅程解析代码中添加另一个关键字值来实现方法之间的差异：

```js
Set JOURNEY="military references api"; yarn job-app-engine
if (journey.toLowerCase().includes(" api)) {
// API path
} else {
// GUI path
}
```

在这个示例中，默认的 GUI 方法在相同的`build()`方法中被 API 路径覆盖。

# 摘要

在本章中，我们深入探讨了一种复杂的自动化测试方法，这种方法让人联想到一个超级英雄在为动态且复杂的任务制定策略，这些任务基于所做选择有多种可能的结局。我们的方法涉及一个循环，它不断地导航到每一页，仅在识别到已知资源时才进行交互。旅程是非确定性的，由高级目标塑造，偶尔会遇到意外的障碍，如死胡同或逻辑循环。

这种方法很灵活，可以根据各种变量调整用户路径，并且可以适应不同的环境，突显其鲁棒性和多功能性。它还有助于为手动测试人员生成数据，与 Jenkins 等工具配合使用时，可以显著减少创建测试数据记录所需的时间和精力，从而提高整个测试过程的效率。

在最后一章中，这种方法将具有额外的优势，使手动测试人员更加高效。最糟糕的想法是假设自动化取代了“昂贵”的手动测试人员。我们应该始终努力增强他们的工作。当手动测试人员只需要验证结果时，他们可以减少创建复杂测试工件所需的大量设置时间。但我们真的希望他们安装编码工具并教他们如何运行这样的脚本吗？这就是我们使用我们最后的神奇物品，让 CI/CD 工具提供一种轻松完成繁重工作的方法的地方。也许像一件神秘的感知悬浮斗篷？
