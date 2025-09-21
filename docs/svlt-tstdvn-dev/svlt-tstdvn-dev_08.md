

# 第八章：创建匹配器以简化测试

本章介绍了另一种简化测试的方法：构建自定义匹配器。大多数时候，坚持使用内置匹配器是有意义的。例如，`toEqual` 匹配器与 `expect.objectContaining` 和 `expect.arrayContaining` 函数的强大组合使得构建表达式的期望变得容易。

但有时构建一个可以将多个不同的检查合并到一个单独的检查中的匹配器是有意义的。这不仅缩短了测试，还可以使它们更容易阅读。

在 *第五章* *验证表单数据* 中，每个表单验证规则都通过一个包含四个测试的 `describe` 上下文进行测试，如下所示：

```js
describe('when the date of birth is in the wrong format')
  it('does not save the birthday', ...)
  it('returns a 422', ...)
  it('returns a useful message', ...)
  it('returns the other data back including the incorrect
      value', ...)
});
```

由于所有验证规则都具有相同的格式，因此似乎是一个很好的候选者来抽象一些共享代码。我们将创建的匹配器将把这三个测试中的三个合并到一个自定义匹配器中——`toBeUnprocessableEntity` 匹配器——这样就可以用一个测试来替换它们：

```js
it('returns a complete error response', async () => {
  const result = await performFormAction(
    createBirthday('Hercules', 'unknown')
  );
  expect(result).toBeUnprocessableEntity({
    error:
      'Please provide a date of birth in the YYYY-MM-DD
       format.',
    name: 'Hercules',
    dob: 'unknown'
  });
});
```

最后还有一个重要的一点：自定义匹配器需要它自己的单元测试集。这样你就可以确保匹配器做了正确的事情：当它应该通过时通过，当它应该失败时失败。就像你想要确保你的应用程序代码做正确的事情一样。

测试测试代码

我的一般规则是这样的：如果你的代码包含任何类型的控制结构或分支逻辑，例如 `if` 语句或循环结构，那么它需要测试。

本章涵盖了以下内容：

+   测试期望的通过或失败

+   在失败消息中提供额外信息

+   实现否定匹配器

+   更新现有测试以使用匹配器

到本章结束时，你将了解如何构建一个匹配器以及如何为其编写测试。

# 技术要求

本章的代码可以在网上找到，地址为 [`github.com/PacktPublishing/Svelte-with-Test-Driven-Development/tree/main/Chapter08/Start`](https://github.com/PacktPublishing/Svelte-with-Test-Driven-Development/tree/main/Chapter08/Start)。

# 测试期望的通过或失败

在本节中，你将构建 `toBeUnprocessableEntity` 匹配器的基本功能，确保它能够正确地通过或失败你的测试。但首先，我们将查看匹配器的结构，然后是单元测试匹配器的方法。

## 理解匹配器结构

让我们看看匹配器的基本结构以及我们如何测试它：

```js
export function toTestSomething(received, expected) {
  const pass = ...
  const message = () => "..."
  return {
   pass,
   message
  };
}
```

`received` 参数值是传递给 `expect` 调用的对象。`expected` 值是传递给匹配器的值。因此，在引言中的示例中，结果是接收到的对象，包含 `error`、`name` 和 `dob` 属性的对象是期望的对象。

`return` 值有两个重要的属性：`pass` 和 `message`。如果匹配器通过了检查，`pass` 布尔值应该是 `true`，否则为 `false`。然而，对于否定匹配器，情况正好相反：`pass` 的 `true` 值意味着期望失败。

`message` 属性是一个返回字符串的函数。这个字符串是测试运行器在测试失败时显示的内容。字符串的内容应该足够让开发者定位到发生的错误。这个属性本身被定义为函数，以便它可以延迟评估：如果测试通过，运行此代码就没有意义。

与本书中的其他代码示例不同，匹配器函数将使用标准的 `function` 关键字定义。这意味着它能够访问 `this` 绑定变量。

Vitest 为 `this` 提供了一些有用的实用函数，匹配器可以使用。我们将在本章中使用其中几个：`this.equals` 和 `this.utils.diff`。另一个有用的属性是 `this.isNot`，如果匹配器以否定形式调用，则该属性为 `true`。

## 测试匹配器

测试匹配器有几种方法。一种方法是与任何其他函数一样测试函数返回值。这种方法的问题是需要设置 `this` 变量，而这并不简单。

另一种方法，也是我们将在本章中使用的方法，是使用 `toThrowError` 匹配器来包装要测试的匹配器，如下所示：

```js
expect(() =>
  expect(response).toBeUnprocessableEntity()
).toThrowError(
  'Expected 422 status code but got 500'
);
```

`toThrowError` 匹配器接受一个函数作为参数来期望；然后在 `try` 块中执行该函数。捕获的 `Error` 对象然后会检查其 `message` 值是否与期望值匹配。

为了使这种方法有效，我们需要确保 `toBeUnprocessableEntity` 已注册到 Vitest 测试运行器。我们可以通过在测试套件开始时运行一次的 `beforeAll` 函数来实现这一点。

拥有所有这些知识，我们就可以开始编写匹配器了。

## 编写 `toBeUnprocessableEntity` 匹配器

让我们开始编写测试套件：

1.  创建一个名为 `src/matchers/toBeUnprocessableEntity.test.js` 的新文件，并从以下导入开始。它们包括所有我们将要使用的 Vitest 导入。我们还将导入 `fail` SvelteKit 函数，我们将在测试中使用它：

    ```js
    import {
      describe,
      it,
      expect,
      beforeAll
    } from 'vitest';
    import { fail } from '@sveltejs/kit';
    import {
      toBeUnprocessableEntity
    } from './toBeUnprocessableEntity.js';
    ```

1.  接下来，在开始处创建一个新的 `describe` 块和一个 `beforeAll` 块。这确保了新匹配器在我们运行测试之前被注册。这只需要做一次：

    ```js
    describe('toBeUnprocessableEntity', () => {
      beforeAll(() => {
        expect.extend({ toBeUnprocessableEntity });
      });
    });
    ```

1.  我们的第一项测试将导致期望失败。添加以下测试代码，它使用 `500` 错误代码而不是 `422` 创建一个失败原因，然后使用 `toThrowError` 匹配器来检查期望是否失败：

    ```js
    it('throws if the status is not 422', () => {
      const response = fail(500);
      expect(() =>
        expect(response).toBeUnprocessableEntity()
      ).toThrowError();
    });
    ```

1.  要使该测试通过，我们只需要构建一个返回`pass`值为`false`的匹配器。创建一个名为`src/matchers/toBeUnprocessableEntity.js`的新文件，并包含以下内容。如前所述，这使用`function`关键字语法，以便我们可以访问附加到`this`变量的 Vitest 匹配器实用函数：

    ```js
    export function toBeUnprocessableEntity(
      received, expected
    ) {
      return { pass: false };
    }
    ```

1.  在第一个测试通过后，添加第二个。这个测试检查的是相反的情况——即如果响应有`422`错误代码，匹配器不会抛出异常：

    ```js
    it('does not throw if the status is 422', () => {
      const response = fail(422);
      expect(() =>
        expect(response).toBeUnprocessableEntity()
      ).not.toThrowError();
    });
    ```

1.  要使测试通过，将原始代码包裹在条件中，使其成为一个守卫子句，如果该条件不满足，则返回`pass`值为`true`，如下所示：

    ```js
    export function toBeUnprocessableEntity(
      received, expected
    ) {
      if (received.status !== 422) {
        return { pass: false };
      }
      return { pass: true };
    }
    ```

1.  如果发生失败，我们希望测试运行器显示有关期望失败原因的有用消息。为此，你可以向`toThrowError`匹配器传递一个字符串值，该值定义了错误消息。这就是 Vitest 测试运行器将在屏幕上显示的内容。添加以下测试：

    ```js
    it('returns a message that includes the actual error code', () => {
      const response = fail(500);
      expect(() =>
        expect(response).toBeUnprocessableEntity()
      ).toThrowError(
        'Expected 422 status code but got 500'
      );
    });
    ```

1.  使其通过涉及返回`message`属性。该属性的值是一个仅在测试失败时调用的函数。这是一种懒加载的形式，允许测试运行器避免进行不必要的操作。更新守卫子句以包含`message`属性，如下所示：

    ```js
    if (received.status !== 422) {
      return {
        pass: false,
        message: () =>
          `Expected 422 status code but got
            ${received.status}`
      };
    }
    ```

1.  接下来，我们还需要检查`response`体。在我们的应用程序代码中，任何`422`结果都会返回一个包含原始表单值的`error`属性。我们希望匹配器在实际响应与预期值不匹配时使测试失败：

    ```js
    it('throws error if the provided object does not match', () => {
      const response = fail(422, { a: 'b' });
      expect(() =>
        expect(response).toBeUnprocessableEntity({
          c: 'd'
        })
      ).toThrowError();
    });
    ```

1.  要使测试通过，我们只需要添加一个非常简单的第二个守卫子句。如果向匹配器传递了一个参数，那么测试就会失败。这个实现甚至离正确实现还差得远，但它足以使测试通过。我们还需要通过更多测试来验证。但就目前而言，你可以从以下代码开始：

    ```js
    export function toBeUnprocessableEntity(
      received,
      expected
    ) {
      if (received.status !== 422) {
        ...
      }
      if (expected) {
        return {
          pass: false
        };
      }
      ...
    }
    ```

1.  下一个测试非常相似，但这次，两个响应体*确实*匹配。这种情况不应该导致失败错误：

    ```js
    it('does not throw error if the provided object does match', () => {
      const response = fail(422, { a: 'b' });
      expect(() =>
        expect(response).toBeUnprocessableEntity({
          a: 'b'
        })
      ).not.toThrowError();
    });
    ```

1.  将第二个守卫子句更新为使用`this.equals`函数对`received.data`值和`expected`参数进行深度相等性检查。这足以使测试通过：

    ```js
    export function toBeUnprocessableEntity(
      received,
      expected
    ) {
      if (received.status !== 422) {
        ...
      }
      if (!this.equals(received.data, expected)) {
        return {
          pass: false
        };
      }
      ...
    };
    ```

1.  最终测试是一个检查部分对象是否匹配的测试：

    ```js
    it('does not throw error if the provide object is a partial match', () => {
      const response = fail(422, { a: 'b', c: 'd' });
      expect(() =>
        expect(response).toBeUnprocessableEntity({
          a: 'b'
        })
      ).not.toThrowError();
    });
    ```

1.  要使该测试通过，我们将使用`expect.objectContaining`约束函数，该函数可以传递到`this.equals`的调用中。首先，在测试文件顶部导入`expect`对象：

    ```js
    import { expect } from 'vitest';
    ```

1.  然后，更新守卫类，将`expected`值包裹在`expect.objectContaining`的调用中，如下所示：

    ```js
    if (
      !this.equals(
        received.data,
        expect.objectContaining(expected)
      )
    ) {
      ...
    }
    ```

1.  最后，如果你现在运行测试，你会发现第一个测试失败了，因为`expected`的值是`undefined`，而`expect.objectContaining`不喜欢这个值。要修复这个问题，为`expected`参数设置一个默认值，如下所示：

    ```js
    export function toBeUnprocessableEntity(
      received,
      expected = {}
    ) {
      ...
    }
    ```

你现在已经看到了如何测试驱动匹配器函数。下一节将重点介绍在发生失败时显示的错误消息的改进。

# 在错误消息中提供额外信息

本节改进了在测试失败时向开发者展示的详细信息。额外信息的目的是帮助定位应用程序代码中的问题，以便开发者不会对出了什么问题感到困惑。

让我们开始：

1.  添加下一个测试，该测试检查当响应体不匹配时是否显示基本消息：

    ```js
    it('returns a message if the provided object does not match', () => {
      const response = fail(422, { a: 'b' });
      expect(() =>
        expect(response).toBeUnprocessableEntity({
          c: 'd'
        })
      ).toThrowError(/Response body was not equal/);
    });
    ```

1.  要实现这个跳过，请将`message`属性添加到第二个保护子句的`return`值。我们将在下一个测试中进一步说明这一点：

    ```js
    if (!this.equals(...)) {
      return {
        pass: false,
        message: () => 'Response body was not equal'
      };
    }
    ```

1.  Vitest 包含一个内置的`diff`辅助对象，它将打印出彩色的差异。颜色是通过 ANSI 颜色代码添加到文本字符串中的，终端将解析并使用这些代码来切换颜色。这些代码的存在意味着在`toThrowError`匹配器中检查文本内容并不简单。以下测试展示了以更简单的方式检查相同内容的实用方法，通过检查`c`和`a`属性是否出现在输出中：

    ```js
    it('includes a diff if the provided object does not match', () => {
      const response = fail(422, { a: 'b' });
      expect(() =>
        expect(response).toBeUnprocessableEntity({
          c: 'd'
        })
      ).toThrowError('c:');
      expect(() =>
        expect(response).toBeUnprocessableEntity({
          c: 'd'
        })
      ).toThrowError('a:');
    });
    ```

1.  要实现这个跳过，我们将把差异追加到我们正在打印的`message`的末尾。首先，从 Node.js 的`os`模块中导入`EOL`常量，它提供了当前平台的行结束符：

    ```js
    import { EOL } from 'os';
    ```

1.  在匹配器代码中，更新第二个保护子句的`message`属性，使用`this.utils.diff`函数来打印`expected`和`received.data`对象的差异：

    ```js
    return {
      pass: false,
      message: () =>
        `Response body was not equal:` + EOL +
        this.utils.diff(expected, received.data)
    };
    ```

这样就完成了详细错误信息的显示。我们将在下一节中完成我们的匹配器，确保它在被否定使用时工作良好。

# 实现否定匹配器

否定匹配器是一项棘手的工作，主要是因为否定匹配器可能有令人困惑的含义。例如，以下期望意味着什么？

```js
expect(result).not.toBeUnprocessableEntity({
  error: 'An unknown ID was provided.'
});
```

据推测，如果响应是`422`且响应体与提供的对象匹配，它应该失败。但是，如果响应是`500`或`200`响应，它也应该失败吗？如果这是预期的，那么写这个就足够了吗？

```js
expect(result).not.toBeUnprocessableEntity();
```

我发现，在编写针对特定领域思想的匹配器时，最好避免使用否定匹配器，或者至少限制其使用。然而，为了展示如何实现，让我们继续使用匹配器。

当我们否定匹配器时，Vitest 测试运行器会在匹配器返回`pass`值为`true`时失败测试。我们恰好有一个场景会发生这种情况，因为所有保护子句都返回`pass`值为`false`。因此，所有这些剩余的测试只是在这种情况下检查`message`属性。

让我们从创建一个嵌套的`describe`块开始：

1.  添加一个名为`not`的嵌套`describe`块，并添加第一个测试：

    ```js
    describe('not', () => {
      it('returns a message if the status is 422 with the
      same body', () => {
        const response = fail(422, { a: 'b' });
        expect(() =>
          expect(response).not.toBeUnprocessableEntity({
            a: 'b'
          })
        ).toThrowError(
          /Expected non-422 status code but got 422/
        );
      });
    });
    ```

1.  要实现这个跳过，请转到匹配器的底部，并将基本的`message`属性值添加到函数中的最后一个`return`值，如下所示：

    ```js
    return {
      pass: true,
      message: () => 'Expected non-422 status code but got 422'
    };
    ```

1.  我们可以通过确保实际`response`体在消息中返回来改进这一点。添加下一个测试：

    ```js
    it('includes with the received response body in the message', () => {
      const response = fail(422, { a: 'b' });
      expect(() =>
        expect(response).not.toBeUnprocessableEntity({
          a: 'b'
        })
      ).toThrowError(/"a": "b"/);
    });
    ```

1.  为了使其通过，您可以使用`this.utils.stringify`实用函数，它为您做了所有艰苦的工作：

    ```js
    return {
      pass: true,
      message: () =>
        `Expected non-422 status code but got 422 with
    body:` + EOL +
        this.utils.stringify(received.data)
    };
    ```

1.  最后，我们需要注意没有传递预期对象的情况。当这种情况发生时，实际的身体对于开发者来说并不相关，因为通过从期望中省略它，他们已经表示他们对此不感兴趣：

    ```js
    it('returns a negated message for a non-422 status with no body', () => {
      const response = fail(422);
      expect(() =>
        expect(response).not.toBeUnprocessableEntity()
      ).toThrowError(
        'Expected non-422 status code but got 422'
      );
    });
    ```

1.  为了使其通过，添加一个第三条守卫子句，如下所示：

    ```js
    if (!received.data) {
      return {
        pass: true,
        message: () =>
          'Expected non-422 status code but got 422'
      };
    }
    ```

你现在已经测试驱动了一个完整的匹配器，它具有有用的错误消息和对否定形式的支持。接下来，是时候在我们的现有测试套件中使用它了。

# 更新现有测试以使用匹配器

在本节的最后，我们将使用我们刚刚构建的匹配器来简化表单验证错误测试套件。

让我们开始吧：

1.  首先，通过在`src/vitest/registerMatchers.js`文件中添加一个`import`语句和调用`expect.extend`来为我们的测试运行注册匹配器：

    ```js
    ...
    import {
      toBeUnprocessableEntity
    } from './src/matchers/toBeUnprocessableEntity.js';
    ...
    expect.extend({ toBeUnprocessableEntity });
    ```

1.  然后，在`src/routes/birthdays/page.server.test.js`中，找到具有描述`当名称未提供时`的嵌套`describe`块。它包含四个测试。保留第一个测试，并用以下测试替换最后三个测试：

    ```js
    describe('when the name is not provided', () => {
      ...
      it('does not save the birthday', ...);
      it('returns a complete error response', () => {
        expect(result).toBeUnprocessableEntity({
          error: 'Please provide a name.',
      dob: '2009-02-02'
        });
      });
    });
    ```

1.  然后，对具有描述`当出生日期格式错误时`的嵌套`describe`块执行相同的操作，用显示的测试替换最后三个测试：

    ```js
    describe('when the date of birth is in the wrong format', () => {
      ...
      it('does not save the birthday', ...);
      it('returns a complete error response', () => {
        expect(result).toBeUnprocessableEntity({
          error:
      'Please provide a date of birth in the YYYY-
              MM-DD format.',
          name: 'Hercules',
          dob: 'unknown'
        });
      });
    });
    ```

1.  用`当 id 是` `未知`上下文完全做同样的事情：

    ```js
    describe('when the id is unknown', () => {
      ...
      it('does not save the birthday', ...);
      it('returns a complete error message', () => {
        expect(result).toBeUnprocessableEntity({
      error: 'An unknown ID was provided.'
        });
      });
    });
    ```

1.  接下来，有一些特定的测试是为了确保返回`id`。更新它们的期望，如下所示：

    ```js
    it('returns the id when an empty name is provided', async () => {
      ...
      expect(result).toBeUnprocessableEntity({
        id: storedId()
      });
    });
    ...
    it('returns the id when an empty date of birth is provided', async () => {
      ...
      expect(result).toBeUnprocessableEntity({
        id: storedId()
      });
    });
    ```

这样就完成了更改。请确保运行所有测试并检查是否一切通过。退后一步，看看您的测试变得多么清晰和简单。

# 摘要

本章向您展示了如何构建一个自定义匹配器以简化您的测试期望。它还讨论了测试驱动匹配器代码的重要性。

您的单元测试文件是您软件的规范。这些文件必须清晰简洁，以便规范清晰。有时，编写自定义匹配器可以帮助您实现这种清晰性。

为什么我们要测试驱动匹配器的实现？因为几乎所有的匹配器都有分支逻辑——有时会通过，有时会失败——你想要确保在正确的时间使用正确的分支。

在下一章中，我们将切换回重构我们的应用程序代码，目的是提高其可测试性。
