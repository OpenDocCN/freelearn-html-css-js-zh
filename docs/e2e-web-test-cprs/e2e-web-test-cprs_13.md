# 第十章：：练习-导航和网络请求

在开始本章之前，重要的是要理解，我们在本书的第三部分的重点将放在练习和示例上，这将帮助您磨练测试技能并建立我们在本书之前可能无法涵盖的知识。在本节中，我们将采取实践方法，目标是尽可能多地进行示例和练习。在深入研究本章之前，重要的是您已经阅读了每一章，并且现在希望在我们学习 Cypress 如何用于测试时，建立在您获得的理论知识基础上。

在本章中，我们将专注于涵盖以下主题的练习和示例：

+   实现导航请求

+   实现网络请求

+   高级导航请求配置

一旦您完成了这些练习，您将有信心成为更好的测试人员，并在导航和网络请求领域进行更复杂的测试。

## 技术要求

为了开始，建议您从 GitHub 克隆包含源代码和本章中将编写的所有测试的存储库。

本章的 GitHub 存储库可以在[`github.com/PacktPublishing/End-to-End-Web-Testing-with-Cypress`](https://github.com/PacktPublishing/End-to-End-Web-Testing-with-Cypress)找到。

本章的源代码可以在`chapter-10`目录中找到。

在我们的 GitHub 存储库中，我们有一个财务测试应用程序，我们将在本章中的不同示例和练习中使用 Cypress 导航和 Cypress 请求。

重要说明：在 Windows 中运行命令

注意：默认的 Windows 命令提示符和 PowerShell 无法正确解析目录位置。

请遵循以下列出的 Windows 命令，这些命令仅适用于以`*windows`结尾的 Windows 操作系统。

为了确保测试应用程序在您的机器上运行，请从应用程序的根文件夹目录中在您的终端上运行以下命令。

`npm run cypress-init`命令将安装应用程序运行所需的依赖项，另一方面，`npm run cypress-app`命令将启动应用程序。可选地，您可以使用`npm run cypress-app-reset`命令重置应用程序状态。重置应用程序会删除任何不属于应用程序的已添加数据，将应用程序状态恢复到克隆存储库时的状态。我们可以在终端中运行这些命令，就像它们在这里显示的那样：

```js
$ cd cypress/chapter-10;
$ npm install -g yarn or sudo npm install -g yarn
$ npm run cypress-init; (for Linux or Mac OS)
$ npm run cypress-init-windows; (for Windows OS)
// run this command if it's the first time running the application
or
$ npm run cypress-app (for Linux or Mac OS)
$ npm run cypress-app-windows; (for Windows OS)
// run this command if you had already run the application previously
Optionally
$ npm run cypress-app-reset; (for Linux or Mac OS)
$ npm run cypress-app-reset-windows; (for Windows OS)
// run this command to reset the application state after running your tests
```

重要说明

在我们的`chapter-10`目录中有两个主要文件夹，一个文件夹包含我们将用于示例和测试练习的应用程序，而第二个文件夹包含我们的 Cypress 测试。为了正确运行我们的测试，我们必须同时运行我们的应用程序和 Cypress 测试，因为测试是在我们本地机器上运行的实时应用程序上运行的。还要注意，应用程序将要求我们使用端口*3000*用于前端应用程序和端口*3001*用于后端应用程序。

掌握上述命令将确保您能够运行应用程序，重置应用程序状态，甚至安装应用程序的依赖项。现在让我们开始导航请求。

# 实现导航请求

Cypress 导航涉及导航到应用程序的网页的行为。在本书中我们已经涵盖了很多测试，在测试之前，您可能还记得`cy.visit()`命令，其中包含我们要导航到或正在测试的页面的 URL。`cy.visit()`命令是导航命令的一个例子，它帮助我们在 Cypress 前端测试中进行导航请求。在本节中，我们将通过示例和练习介绍不同的 Cypress 导航命令。通过本节结束时，我们将更深入地了解 Cypress 导航命令，这将帮助我们在本书前几章已经具备的导航知识基础上构建更多的知识。

## cy.visit()

在 Cypress 中，我们使用`cy.visit()`导航到被测试应用程序的远程页面。通过使用这个命令，我们还可以传递配置信息给命令，并配置选项，如方法、URL、超时选项，甚至查询参数；我们将在本章后面深入探讨该命令的配置选项。

在我们的 GitHub 存储库中，在`chapter-10/cypress-realworld-app`目录中，我们有一个应用程序，我们将在示例和练习中使用。

重要提示

我们的金融应用程序位于`chapter-10/cypress-realworld-app`目录中，记录交易。通过该应用程序，我们可以通过请求或支付用户来创建交易，这些交易已经存在于应用程序中。我们可以看到已发生交易的通知，还可以查看联系人和已发生交易的日志。

该应用程序使用 JSON 数据库，因此在将所有数据加载到我们的应用程序时会有点慢。在我们的测试中，我们已经实现了一个“安全开关”，通过确保在`beforeEach`方法中，我们等待所有初始的 XHR（XMLHttpRequest）请求加载数据，以防止测试失败。在下面的代码块中查看有关`beforeEach`方法的更多信息。

在我们的第一个示例中，在`navigation.spec.js`中，如下所示的代码块，我们将使用`cy.visit()`命令导航到应用程序的通知页面：

```js
describe('Navigation Tests', () => {
    beforeEach(() => {
 	cy.loginUser();
	cy.server();
	cy.intercept('bankAccounts').as('bankAccounts');
     	cy.intercept('transactions/public').as('transactions')
     ;
     	cy.intercept('notifications').as('notifications');
     	cy.wait('@bankAccounts');
     	cy.wait('@transactions');
     	cy.wait('@notifications');
});
    afterEach(() => { cy.logoutUser()});
    it('Navigates to notifications page', () => {
        cy.visit('notifications', { timeout: 30000 });
        cy.url().should('contain', 'notifications');
    });
});
```

这个代码块说明了`cy.visit()`命令的用法，我们访问远程 URL 到通知路由（`http://localhost:3000/notifications`），然后验证我们访问的远程 URL 是否符合预期。在我们的导航命令中，我们还添加了超时选项，确保在失败导航测试之前，Cypress 将等待 30 秒的“页面加载”事件。

以下截图显示了我们的测试正在执行，Cypress 正在等待从后端接收的 XHR 请求加载所有必须从我们的 JSON 数据库中加载的数据：

![图 10.1 - XHR API 请求和响应](img/Figure_10.1_B15616.jpg)

图 10.1 - XHR API 请求和响应

在这个截图中，我们正在导航到`/signin`页面，然后等待所有资源加载完成后，我们使用 Cypress 的`cy.visit()`命令导航到`/notifications`页面，在测试应用程序预览的右侧可见。这进一步通过我们的测试断言进行验证，该断言验证访问的 URL 是否包含名称`notifications`。以下练习将帮助您更好地了解如何使用`cy.visit()`命令实现测试。

## 练习 1

使用 GitHub 存储库中提供的金融应用程序，位于`cypress-real-world-app`文件夹的根目录中，进行以下练习，测试您对`cy.visit()`命令的了解：

1.  登录到我们的测试应用程序，并使用`cy.visit()`命令导航到`http://localhost:3000/bankaccounts` URL。

1.  创建一个新的银行账户，然后检查在创建新的银行账户后应用程序是否重定向回`/bankaccounts` URL。

1.  登录应用程序，并使用`cy.visit()`命令尝试导航到`http://localhost:3000/signin`。

1.  成功登录测试用户后，验证 URL 重定向到仪表板而不是`/signin`页面。

练习的解决方案可以在`chapter-10/integration/navigation/navigation-exercise-solutions`目录中找到。

此练习将测试您理解`cy.visit()`命令的能力，确保作为 Cypress 用户，您可以有效地使用该命令导航到不同的 URL，并将参数和配置选项传递给命令。

## cy.go()

Cypress 的`cy.go()`导航命令使用户能够在测试应用程序中向前或向后导航。在使用`cy.go()`命令时，将'back'选项传递给命令将导致浏览器导航到浏览器历史记录的上一页，而'forward'选项将导致浏览器导航到页面的前进历史记录。我们还可以使用此命令通过传递数字选项作为参数来单击前进和后退按钮，其中'-1'选项将导航应用程序*后退*，而传递'1'将导致*前进*从浏览器历史记录中导航。

通过使用`cy.go()`，我们能够通过能够向后跳转到浏览器历史记录的上一页，以及向前跳转到浏览器历史记录的下一页，来操纵浏览器的导航行为。

重要提示

我们在`cy.visit()`命令中只使用`/bankaccounts`，因为我们已经在`cypress.json`文件中声明了`baseUrl`。`baseUrl`是 URL 的完整版本，我们在使用`cy.visit()`和`cy.intercept()`命令时不需要每次重复。您可以在开始本章时克隆的 GitHub 存储库中查看更多信息。

在以下代码块中，我们将使用我们的金融应用程序来验证我们可以在导航到`/bankaccounts`页面后返回到仪表板：

```js
describe('Navigation Tests', () => {
    it('cy.go(): Navigates front and backward', () => {
        cy.visit('bankaccounts');
        cy.url().should('contain', '/bankaccounts');
        cy.go('back');
        cy.url().should('eq', 'http://localhost:3000/');
    });
});
```

在此测试中，导航到`/bankaccounts` URL 后，我们然后使用 Cypress 内置的`cy.go('back')`命令导航回仪表板 URL，然后验证我们已成功导航回去。以下练习将更详细地介绍如何使用`cy.go()`命令。

## 练习 2

使用 GitHub 存储库中提供的金融应用程序，位于`chapter-10/cypress-real-world-app`目录中，执行以下练习以测试您对`cy.go()`命令的了解：

1.  登录后，在交易仪表板上，单击**Friends**选项卡，然后单击**Mine**选项卡。

1.  使用 Cypress 使用`cy.go()`命令返回到**Friends**选项卡。

1.  登录后，单击应用程序导航栏右上角的**New**按钮，并创建一个新交易。

1.  然后使用`cy.go()`Cypress 命令返回到仪表板页面，然后返回到新交易。

练习的解决方案可以在`chapter-10/integration/navigation/navigation-exercise-solutions`目录中找到。

此练习将帮助您建立使用`cy.go()`命令测试前进和后退导航的技能。它还将帮助您在测试应用程序时建立对导航的信心。

## cy.reload()

Cypress 的`cy.reload()`命令负责重新加载页面。该命令只有一组选项可以传递给它，即重新加载页面时清除缓存或重新加载页面时保留应用程序内存中的缓存。当将`true`的布尔值传递给`cy.reload()`方法时，Cypress 不会重新加载带有缓存的页面；相反，它会清除缓存并加载有关页面的新信息。省略布尔值会导致 Cypress 重新加载启用缓存的页面。在以下代码块中，我们在登录到我们的应用程序后重新加载仪表板；这将刷新我们仪表板页面的状态：

```js
it('cy.reload(): Navigates to notifications page', () => {
    cy.reload(true);
    });
```

在这个测试中，如果我们的浏览器中有任何缓存项目，Cypress 将重新加载页面并使缓存无效，以确保在执行我们的测试时创建页面的新状态和缓存。让我们看看下面的练习，了解更多关于使用`cy.reload()`命令的情景。

## 练习 3

使用 GitHub 存储库中提供的金融应用程序，位于`chapter-10/cypress-real-world-app`目录中，进行以下练习，以测试您对`cy.reload()`命令的了解：

1.  转到**账户**菜单项，那里有用户设置。

1.  在点击**保存**按钮之前，编辑您测试用户的名字和姓氏。

1.  重新加载页面并验证`cy.reload()`命令是否重置了尚未保存的所有设置。

练习的解决方案可以在`chapter-10/integration/navigation/navigation-exercise-solutions`目录中找到。

在这个练习中，我们已经学会了 reload 命令只会重置浏览器中暂时存储的项目。通过使用`cy.reload()`命令，我们了解了如何重置我们应用程序的缓存存储以及如何对其进行测试。

## 总结 - 实现导航请求

在本节中，我们通过评估示例和进行练习来学习了 Cypress 上的导航请求是如何工作的。我们还探讨了各种导航命令，如`cy.visit()`、`cy.go()`和`cy.reload()`，它们在执行 Cypress 中的导航请求时都扮演着重要角色。在下一节中，我们将深入研究如何使用练习和示例来实现网络请求。

# 实现网络请求

网络请求涉及处理向后端服务的 AJAX 和 XHR 请求。Cypress 使用其内置的`cy.request()`和`cy.intercept()`命令来处理这一点。在本节中，我们将采用实践方法，深入探讨如何使用示例和练习在 Cypress 中实现网络请求。在本书的*第九章*中，我们已经与网络请求进行了交互，本章将帮助您建立在您已经熟悉的理论知识和概念基础上。

## cy.request()

Cypress 的`cy.request()`命令负责向 API 端点发出 HTTP 请求。该命令可用于执行 API 请求并接收响应，而无需创建或导入外部库来处理我们的 API 请求和响应。我们的 Cypress 金融应用程序使用基于 JSON 数据库的后端 API。为了了解`cy.request()`命令的工作原理，我们将请求数据库并检查响应。以下代码块是一个请求，用于从我们的 API 中获取所有交易：

```js
it('cy.request(): fetch all transactions from our JSON database', () => {
        cy.request({
            url: 'http://localhost:3001/transactions',
            method: 'GET',
        }).then((response) => {
            expect(response.status).to.eq(200);
            expect(response.body.results).to.be.an
            ('array');
        })
    });
```

在上面的测试中，我们正在验证我们的后端是否以`200`状态代码和交易数据（数组）做出响应。我们将在下一个练习中了解更多关于`cy.request()`命令的内容。

## 练习 4

使用 GitHub 存储库中提供的金融应用程序，位于`chapter-10/cypress-real-world-app`目录中，进行以下练习，以测试您对`cy.server()`命令的了解。练习的解决方案可以在`chapter-10/integration/navigation/network-requests-excercise-solutions`目录中找到：

1.  登录后，使用您的浏览器，调查我们的`cypress-realworld`应用程序在我们首次登录时加载的 XHR 请求。

1.  根据观察，编写一个返回以下数据的测试：

应用程序中的联系人

应用程序中的通知

通过进行这个练习，您将更好地理解`cy.request()`命令，并增加对 Cypress 请求工作原理的了解。接下来，我们将看一下 Cypress 路由。

## cy.intercept()

`cy.intercept()`命令管理测试的网络层的 HTTP 请求的行为。通过该命令，我们可以了解是否进行了 XHR 请求，以及我们的请求的响应是否与我们的预期相匹配。我们甚至可以使用该命令来存根路由的响应。使用`cy.intercept()`，我们可以解析响应并确保我们实际上对于我们测试的应用程序有正确的响应。`cy.intercept()`命令使我们可以在所有阶段完全访问我们 Cypress 测试的所有 HTTP 请求。

重要提示

我们必须在测试中引用路由之前调用`cy.intercept()`，以便在测试中调用它们之前记录路由，并且从下面的测试中，我们可以观察到`beforeEach()`命令块中的行为。在接下来的测试中，我们在开始运行 Cypress 测试之前调用了`cy.intercept`命令。

在`network-request.spec.js`文件中找到的以下代码块中，我们正在验证在测试应用程序进行正确登录请求时，我们是否有用户信息的响应：

```js
describe('Netowork request routes', () => {
        beforeEach(() => {        
        cy.intercept('POST','login').as('userInformation');
        });

        it('cy.intercept(): verify login XHR is called when
        user logs in', () => {
            cy.login();
            cy.wait('@userInformation').its('
            response.statusCode').should('eq', 200)
        });
    });
```

在这个代码块中，我们正在验证应用程序是否向登录端点发出了`POST`请求，并且我们收到了成功的`200`状态，这是一个成功的登录。`cy.login()`命令导航到应用程序的登录页面。我们将在下一个练习中进一步使用`cy.intercept()`命令。

## 练习 5

使用 GitHub 存储库中提供的金融应用程序，位于`chapter-10/cypress-real-world-app`目录中，进行以下练习，以测试您对`cy.intercept()`命令的了解。练习的解决方案可以在`chapter-10/integration/navigation/network-requests-exercise-solutions`目录中找到：

1.  登录到测试应用程序并转到账户页面。

1.  使用 Cypress 的`cy.route()`命令来检查 Cypress 是否在更改用户信息时验证用户是否已登录。

是时候进行快速总结了。

## 总结-实施网络请求

在本节中，我们探讨了 Cypress 网络请求的工作原理，通过示例和练习来理解`cy.request()`和`cy.intercept()`在 Cypress 测试中的应用。通过示例和练习，我们还扩展了对如何使用诸如`cy.intercept()`之类的命令来操作和存根的知识。现在我们了解了网络请求，并且可以轻松地编写涉及 Cypress 网络请求的测试，在下一节中，我们将深入研究导航请求的高级配置。

# 高级导航请求配置

导航是正确运行测试的最重要的方面之一。通过使用`cy.visit()`、`cy.go()`甚至`cy.reload()`命令，我们能够知道在编写测试时应该采取什么样的快捷方式。导航命令还显著简化了测试工作流程。大多数前端测试都需要导航，因此掌握高级配置不仅会让您的生活更轻松，而且在编写测试时也会带来更顺畅的体验。在本节中，我们将主要关注 Cypress 的`cy.visit()`命令的高级命令配置，因为它是 Cypress 的主要导航命令。

## cy.visit() 配置选项

下表显示了`cy.visit()`命令的配置选项以及当没有传递选项给选项对象时加载的默认值：

![](img/B15616_10_Table_1a.jpg)![](img/B15616_10_Table_1b.jpg)

`cy.visit()`命令接受不同类型的参数，这决定了传递给它的配置和选项。以下是该命令接受的参数：

+   只使用 URL 进行配置：

```js
cy.visit(url)
e.g. cy.visit('https://test.com');
```

+   使用 URL 和选项作为对象的配置：

```js
cy.visit(url, options)
e.g. cy.visit('https://test.com', {timeout: 20000});
```

+   只使用选项作为对象进行配置：

```js
cy.visit(options);
e.g. cy.visit('{timeout: 30000}');
```

现在是总结时间！

## 总结-高级导航请求配置

在本节中，我们学习了如何使用不同的选项配置`cy.visit()`命令，以及该命令接受的不同类型的参数。我们还学习了 Cypress 在没有传递选项对象时为我们提供的不同默认选项，这使得使用`cy.visit()`命令的过程变得简单，因为我们只需要向命令提供我们需要在测试中覆盖的选项。

# 总结

在本章中，我们学习了 Cypress 如何执行导航，如何创建请求以及 Cypress 如何解释和返回它们以进行我们的测试执行过程。我们采用了实践方法来学习三个基本的 Cypress 导航命令，以及 Cypress 用于创建和解释请求的三个命令。这些练习为您提供了一个渠道，让您走出舒适区，并对 Cypress 的高级用法进行一些研究，以及如何将我们在本书中获得的逻辑和知识整合到编写有意义的测试中，从而为被测试的应用程序增加价值。最后，我们看了一下`cy.visit()`命令的高级配置选项。我相信在本章中，您学会了处理和实现测试中的导航和网络请求的技能，以及配置导航请求。

现在我们已经实际探索了使用 Cypress 进行导航和请求，接下来我们将在下一章中使用相同的方法来处理使用 Cypress 进行测试的存根和间谍。
