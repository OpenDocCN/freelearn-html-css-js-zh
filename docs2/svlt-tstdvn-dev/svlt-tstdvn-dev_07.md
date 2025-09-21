

# 第七章：整理测试套件

你在处理测试套件时是否曾经感到沮丧？除非你积极维护它们，否则它们很容易变得杂乱无章。在本章中，我们将探讨一些保持测试套件整洁的方法。

你用于整理测试套件的技巧与你在应用程序代码中使用的技巧不同。应用程序代码需要构建抽象和封装细节，具有深层连接的对象。然而，测试受益于保持简洁，每个测试语句都有明确的效果。

另一种思考方式是，正常程序流程可以通过代码中的许多不同路径，但测试套件只有一条流程——它们是从上到下运行的脚本。其中缺少控制逻辑，如条件表达式和循环结构。

在 Playwright 测试中使用页面对象模型

本章涵盖了以下技巧：

+   在 Playwright 测试中使用页面对象模型

+   提取动作辅助函数

+   提取用于创建数据对象的工厂方法

到本章结束时，你将学会一系列策略来减少测试套件的大小。

# 技术要求

控制测试套件复杂性的主要机制是抽象函数，以隐藏细节。

# [代码示例](https://github.com/PacktPublishing/Svelte-with-Test-Driven-Development/tree/main/Chapter07/Start)

**页面对象模型**只是一个简单的 JavaScript 类，它将导航页面的机械动作（定位字段、点击按钮或填写文本字段）组合成描述应用程序中高级操作的方法（完成生日表单）。

在本节中，你将构建一个名为 `BirthdayListPage` 的页面对象模型，这将允许你更简单地重写现有的 Playwright 测试。

让我们从添加新的类开始：

1.  创建一个名为 `tests/BirthdayListPage.js` 的新文件，并给它以下内容。它创建了一个基本类，以及一个名为 `goto` 的单个方法，用于导航到 `/birthdays` 应用程序 URL：

    ```js
    export class BirthdayListPage {
      constructor(page) {
        this.page = page;
      }
      async goto() {
        await this.page.goto('/birthdays');
      }
    }
    ```

1.  我们已经在测试中使用了这个类。在 `tests/birthdays.test.js` 文件顶部添加以下导入：

    ```js
    import {
      BirthdayListPage
    } from './BirthdayListPage.js';
    ```

1.  现在，更新所有测试以使用此类，通过将直接调用 `page.goto` 替换为通过 `BirthdayListPage` 对象的间接调用。例如，考虑以下现有测试：

    ```js
    test('edits a birthday', async ({ page }) => {
      await page.goto('/birthdays');
      ...
    });
    ```

它应该修改为如下所示：

```js
test('edits a birthday', async ({ page }) => {
  const birthdayListPage = new BirthdayListPage(page);
  await birthdayListPage.goto();
  ...
});
```

1.  现在，我们将继续为每个单独的文件创建辅助函数。看看以下来自编辑测试的原始代码：

    ```js
    // add a birthday using the form
    await page.getByLabel('Name').fill('Ares');
    await page
      .getByLabel('Date of birth')
      .fill('1985-01-01');
    await page
      .getByRole('button', { name: 'Save' })
      .click();
    ```

从这里，我们可以提取名为 `nameField`、`dateOfBirthField` 和 `saveButton` 的辅助函数。现在，将它们添加到 `BirthdayListPage` 中，如下所示：

```js
export class BirthdayListPage {
  ...
  dateOfBirthField = () =>
    this.page.getByLabel('Date of birth');
nameField = () => this.page.getByLabel('Name');
  saveButton = () =>
    this.page.getByRole('button', { name: 'Save' });
}
```

1.  仍然在 `BirthdayLastPage` 中，你现在可以将这些辅助方法合并成一个执行整个操作的单一辅助方法，名为 `saveNameAndDateOfBirth`：

    ```js
    export class BirthdayListPage {
      ...
      saveNameAndDateOfBirth = async (name, dob) => {
        await this.nameField().fill(name);
        await this.dateOfBirthField().fill(dob);
        await this.saveButton().click();
    };
    ```

1.  原始测试中 *步骤 4* 的代码部分，包括解释性注释，现在可以通过一个函数调用完成。注释不再必要，因为方法名基本上表达了相同的意思。现在按照以下方式更新测试：

    ```js
    test('edits a birthday', async ({ page }) => {
      const birthdayListPage = new BirthdayListPage(page);
      await birthdayListPage.goto();
      await birthdayListPage.saveNameAndDateOfBirth(
        'Ares',
        '1985-01-01'
      );
      ...
    );
    ```

这样就完成了本节的测试。接下来是下一部分，看起来如下：

```js
// find the Edit button for that person
await page
  .getByRole('listitem')
  .filter({ hasText: 'Ares' })
  .getByRole('button', { name: 'Edit' })
  .click();
```

对于本节，我们可以重复之前的步骤来提取一个内部辅助方法以定位所需的字段，然后提取一个外部辅助方法来执行操作。首先在 `BirthdayListPage` 页面对象模型中添加一个新的辅助方法，命名为 `entryFor`，用于查找某人的姓名条目：

```js
entryFor = (name) =>
  this.page
    .getByRole('listitem')
    .filter({ hasText: name });
```

1.  现在，使用 `entryFor` 方法构建另一个辅助方法，`beginEditingFor`，它将点击该生日页面的 **编辑** 按钮：

    ```js
    beginEditingFor = (name) =>
      this.entryFor(name)
        .getByRole('button', { name: 'Edit' })
        .click();
    ```

1.  是时候移除 *步骤 6* 中显示的原始代码，通过在页面对象模型中调用这个新辅助方法来执行单一调用。按照以下代码块中的更新进行操作，再次删除注释并将六行代码缩减为一行：

    ```js
    test('edits a birthday', async ({ page }) => {
      const birthdayListPage = new BirthdayListPage(page);
      await birthdayListPage.goto();
      await birthdayListPage.saveNameAndDateOfBirth(
        'Ares',
        '1985-01-01'
      );
      await birthdayListPage.beginEditingFor('Ares');
      ...
    });
    ```

1.  测试中还有一个最终的操作可以更新，即开始编辑后修改表单值。我们不需要为此操作创建任何新的辅助方法。我们只需要重用 `saveNameAndDateOfBirth` 辅助方法。现在按照以下方式执行此操作：

    ```js
    test('edits a birthday', async ({ page }) => {
      ...
      await birthdayListPage.beginEditingFor('Ares');
      await birthdayListPage.saveNameAndDateOfBirth(
        'Ares',
        '1995-01-01'
      );
      ...
    });
    ```

1.  最后要做的更改是更改期望。它们可以更新为使用 `entryFor` 辅助方法。由于这是本节中的最后一个更改，列表显示了完整的测试。按照以下方式更改期望：

    ```js
    test('edits a birthday', async ({ page }) => {
      const birthdayListPage = new BirthdayListPage(page);
      await birthdayListPage.goto();
      await birthdayListPage.saveNameAndDateOfBirth(
        'Ares',
        '1985-01-01'
      );
      await birthdayListPage.beginEditingFor('Ares');
      await birthdayListPage.saveNameAndDateOfBirth(
        'Ares',
        '1995-01-01'
      );
      await expect(
        birthdayListPage.entryFor('Ares')
      ).not.toContainText('1985-01-01');
      await expect(
        birthdayListPage.entryFor('Ares')
      ).toContainText('1995-01-01');
    });
    ```

引入 `BirthdayListPage` 页面对象模型的好处是显而易见的：测试更易读，测试数据更突出（两个出生日期的变化现在更明显），并且任何未来的更改都将更快完成，因为测试更短。

继续进行剩余的测试

仓库中的其他 Playwright 测试也可以使用完全相同的辅助方法重写。这些更改在本书中没有展示，但在配套仓库中可用。

在本节中，你看到了如何创建 Playwright 页面对象模型。接下来，我们将对 Vitest 单元测试做类似的事情。

# 提取操作辅助方法

本节介绍了如何使用辅助方法简化测试的 *执行* 阶段。

理解 Arrange-Act-Assert 模式

*Arrange-Act-Assert* 模式是描述单元测试编写顺序的标准方式。

它们从 *安排* 阶段开始，这是测试结构准备工作的阶段。任何输入数据都会被构建，并且会调用任何准备方法，使系统进入正确的状态。

然后，我们有 *Act* 阶段，它调用正在检查的操作。最后，测试以 *Assert* 阶段结束，该阶段可以是一个或多个期望（或 *断言*），以验证操作是否按预期执行。

这三个阶段各自受益于不同的策略来消除重复。

*Act* 阶段是受益于消除重复最少的阶段。这是因为您将要编写的绝大多数单元测试——以及本书中的所有单元测试——都有一个由单个方法调用触发的操作。很少会遇到需要超过单个指令的操作来观察的动作。

由于这个原因，我倾向于确保方法调用直接在单元测试中调用，而不是与任何 *Arrange* 阶段的语句混合。话虽如此，有些情况下构建一个 *Act* 辅助函数是有帮助的。通过 `actions.default` 导入调用 SvelteKit 表单操作的调用就是这些情况之一。

有几个原因。首先，`actions.default` 的名称在单元测试套件的上下文中不够描述性。其次，表单操作的参数并不简单——它使用表单 API 的 `request` 对象来包装表单数据，然后将其打包成一个类似 SvelteKit `RequestEvent` 的对象。这需要在每个测试中完成。我们关心的是表单数据中的值，而不是围绕它的管道。

在 `src/routes/birthdays/page.server.test.js` 文件中，您将看到以下模式被重复多次：

```js
const request = createFormDataRequest({
 // ... the form data object ...
});
await actions.default({ request });
```

您可以将请求语句视为设置的一部分。每次调用表单操作时都需要它。没有进行表单数据操作就无法调用表单操作。

因此，让我们创建一个包装此行为的辅助函数。我们将称之为 `performFormAction`，这样在测试中就可以清楚地了解正在发生什么。

让我们开始：

1.  在 `src/routes/birthdays/page.server.test.js` 中，将以下函数定义添加到 `/birthdays - default` `action` 上下文的顶部：

    ```js
    const performFormAction = (formData) =>
      actions.default({
        request: createFormDataRequest(formData)
      });
    ```

1.  在每个测试中，寻找前面代码片段中描述的模式——调用 `createFormDataRequest` 然后导入的 `actions.default` 函数——并将每个实例替换为对 `performFormAction` 的调用。以下是从测试中的一个示例：

    ```js
    it('adds a new birthday into the list', async () => {
      await performFormAction({
        name: 'Zeus',
        dob: '2009-02-02'
      });
      expect(birthdayStore.getAll()).toContainEqual(
        expect.objectContaining({
          name: 'Zeus',
          dob: '2009-02-02'
        });
      });
    });
    ```

确保您在测试套件中对所有这些更改重新运行所有测试。在下一节中，我们将继续查看简化测试套件的 *Arrange* 部分。

# 提取用于创建数据对象的工厂方法

是时候使用名为 `createBirthday` 的工厂方法简化测试的 *Arrange* 阶段了。

上一节提到，每个 *安排-行动-断言* 阶段都需要不同的处理以简化。*安排* 阶段的一个关键方法是使用工厂。您已经在 *第四章*，*保存表单数据* 中创建了一个这样的工厂。那就是您在上一节中使用的 `createFormDataRequest` 方法。

使用测试工厂来隐藏无关数据

工厂方法帮助您以尽可能少的代码生成支持对象。它们这样做的一种方式是为对象属性设置默认值，这样您就不需要指定它们。然后您可以自由地在每个单独的测试中覆盖这些默认值。

隐藏必要但无关的数据是保持单元测试简洁和清晰的关键方法。

我们的生日对象结构非常简单，只有三个字段——`name`、`dob` 和 `id`。在这三个中，`name` 和 `dob` 频繁设置，而 `id` 字段很少设置。此外，每个单独的字段都有独特的数据形状——名称看起来与日期非常不同，与 **通用唯一** **标识符**（**UUID**）也非常不同。

考虑到这一点，即将到来的 `createBirthday` 辅助函数需要 `name` 和 `dob` 两个参数，但将 `id` 作为可以有时指定的额外字段。这些 `name` 和 `dob` 值作为位置参数给出——这意味着它们通过位置而不是通过名称来识别——因为很明显哪个是哪个。这节省了页面上的空间。

这可能看起来并不重要，但当你编写可维护的软件时，每个单词都必须证明其价值。

下面是一个示例，注意 `id` 的指定方式不同，因为很少使用：

```js
createBirthday('Zeus Ex', '2007-02-02', { id: storedId() })
```

让我们开始：

1.  创建一个名为 `src/factories/birthday.js` 的新文件，并给它以下内容：

    ```js
    export const createBirthday = (
      name,
      dob,
      extra = {}
    ) => ({ name, dob, ...extra });
    ```

1.  打开 `src/routes/birthdays/Birthday.test.js` 文件并导入新的辅助函数：

    ```js
    import {
      createBirthday
    } from src/factories/birthday.js';
    ```

1.  接下来，找到所有使用 `exampleBirthday` 对象的 `render` 调用。它们看起来像这样：

    ```js
    render(Birthday, {
      ...exampleBirthday,
      name: 'Hercules'
    });
    ```

将它们更新为使用新的辅助函数，如下所示：

```js
render(
  Birthday,
  createBirthday('Hercules', '1996-03-03')
);
```

1.  然后，在 `src/routes/birthdays/page.server.test.js` 中添加 `createBirthday` 导入：

    ```js
    import {
      createBirthday
    } from 'src/factories/birthday.js';
    ```

1.  找到上一节中更新的所有对 `performFormAction` 的调用。它们看起来像这样：

    ```js
    await performFormAction({
      name: 'Zeus',
      dob: '2009-02-02'
    });
    ```

将它们更新为使用 `createBirthday` 辅助函数，如下所示：

```js
await performFormAction(
  createBirthday('Zeus', '2009-02-02')
);
```

有几个测试中，前面的更改并不直接。在 *保存每个新生日唯一的 ID* 测试中，您可以将创建的生日保存到 `request` 对象中，然后将其传递给 `performFormAction` 两次，如下面的代码所示：

```js
it('saves unique ids onto each new birthday', async () => {
  const request = createBirthday(
    'Zeus',
    '2009-02-02'
  );
  await performFormAction(request);
  await performFormAction(request);
  expect(birthdayStore.getAll()[0].id).not.toEqual(
    birthdayStore.getAll()[1].id
  );
});
```

*更新具有相同 ID 的条目* 测试需要一个特定的 `id` 在第二次调用中传递。注意工厂方法是如何构建的，以便需要命名不常见的信息，例如 `id` 字段：

```js
it('updates an entry that shares the same id', async () => {
  await performFormAction(
    createBirthday('Zeus', '2009-02-02')
  );
  await performFormAction(
    createBirthday('Zeus Ex', '2007-02-02', { id:
      storedId() })
  );
  expect(birthdayStore.getAll()).toHaveLength(1);
  expect(birthdayStore.getAll()).toContainEqual({
    id,
    name: 'Zeus Ex',
    dob: '2007-02-02'
  });
});
```

1.  对于最终的测试套件，在 `src/routes/birthdays/page.test.js` 中，可以将 `birthdays` 数组更新为使用两个 `createBirthday` 调用，如下所示：

    ```js
    const birthdays = [
      createBirthday('Hercules', '1994-02-02', {
        id: '123'
      }),
      createBirthday('Athena', '1989-01-01', {
        id: '234'
      })
    ];
    ```

1.  最后，更新这里显示的测试，使其直接在 `render` 调用中使用 `createBirthday` 助手，用于 `birthdays` 属性值和 `form` 属性值：

    ```js
    it('displays all the birthdays passed to it', () => {
      render(Page, {
        data: {
          birthdays: [
            createBirthday('Hercules', '1994-02-02', {
            id: '123'
            })
          ]
        },
        form: {
          ...createBirthday('Hercules', 'bad dob', {
            id: '123'
            }),
          error: 'An error'
        }
      });
    });
    ```

使用 `createBirthday` 的过程到此结束。请确保重新运行你的测试，以确保一切仍然正常。

你现在已经学会了如何使用测试工厂方法来简化并使你的单元测试更加清晰。

# 摘要

本章向你展示了三种缩短测试套件的技术：Playwright 端到端测试的页面对象模型、Vitest 单元测试的动作助手和工厂方法。保持测试套件清晰和有意义是保持它们易于维护的关键。

在下一章中，我们将探讨一种更复杂的方式来减少单元测试代码——编写你自己的自定义匹配器。
