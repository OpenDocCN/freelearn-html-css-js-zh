# 第八章。创建自定义任务

在本章中，我们将涵盖以下配方：

+   创建别名任务

+   创建基本任务

+   访问项目配置

+   检查所需配置

+   检查其他任务的执行是否成功

+   在任务中运行非阻塞代码

+   任务失败

+   使用命令行参数

+   将任务入队以运行

+   创建多任务

+   在任务中使用选项

+   在任务中使用文件

# 简介

要真正开始欣赏 Grunt 的强大和灵活性，我们必须深入研究创建我们自己的任务。

在我们可用的各种插件中，每个插件提供的任务都可以以多种方式配置，应该能够满足我们的大部分需求。然而，如果我们遇到无法使用现有任务和配置组合解决的问题，我们可以轻松地创建一个新的任务来填补这个空白。

我们也应该将创建任务视为创建我们自己的插件的第一步。本章中讨论的创建任务的方法将为我们提供一个简单的实验室，我们可以在这里构建最终可以直接转移到插件中的任务。

### 小贴士

以下两个 URL 提供了有关创建任务和 Grunt 提供的构建它们的工具的大量信息。如果您认真考虑创建任务，强烈建议您查看这些链接：

[`gruntjs.com/creating-tasks`](http://gruntjs.com/creating-tasks)

[`gruntjs.com/api/inside-tasks`](http://gruntjs.com/api/inside-tasks)

# 创建别名任务

我们为项目配置的任务往往只执行一个功能，没有更多。随着项目的增长，我们可能会开始识别出我们倾向于按特定顺序一起运行的多个任务组。在这个时候，别名任务可以对我们非常有帮助，因为它允许我们将一组任务组合成一个新任务名下。

## 准备工作

在这个例子中，我们将使用我们在第一章中“在项目中安装 Grunt”配方中创建的基本项目结构。如果您还不熟悉其内容，请务必参考。

这个配方还包含一个示例，它使用了第一章中“在项目中安装 Grunt”配方中的*设置基本 Web 服务器*和*监视文件变化*配方，以及第三章中“模板引擎”配方中的*渲染 Jade 模板*配方，在一个设置中。如果您想深入了解这些方面，请务必参考这些内容。

## 如何操作...

以下步骤将引导我们构建一个基本的网站开发设置，该设置会持续从模板渲染 HTML 文件，使用基本网络服务器提供服务，并在开始之前通过删除所有先前渲染的模板来确保干净的环境：

1.  我们将首先按照 第一章 的 *安装插件* 菜谱中提供的说明安装包含 `contrib-clean`、`contrib-connect`、`contrib-jade` 和 `contrib-watch` 插件的包。

1.  接下来，我们将在 `templates` 目录中创建一个简单的 Jade 模板，名为 `index.jade`，并为其提供以下内容：

    ```js
    doctype html
    html
      head
        title Sample Site
      body
        h1 Welcome to my Sample Site!
    ```

1.  现在，我们可以添加任务来监视、渲染、提供服务和清理我们的配置：

    ```js
    clean: {
      all: ['www']
    },
    jade: {
      www: {
        expand: true,
        cwd: 'templates',
        src: '**/*.jade',
        dest: 'www',
        ext: '.html'
      }
    },
    connect: {
      dev: {
        options: {
          base: 'www'
        }
      }
    },
    watch: {
      www: {
        files: 'templates/**/*.jade',
        tasks: ['jade:www']
      }
    }
    ```

1.  在所有任务就绪之后，我们现在可以创建一个名为 `run` 的别名任务，它将它们按适当的顺序全部关联起来。这是通过将以下内容添加到我们的 Grunt 配置文件中的主函数末尾来完成的：

    ```js
    grunt.registerTask('run', [
      'clean:all',
      'jade:www',
      'connect:dev',
      'watch:www'
    ]);
    ```

1.  现在，我们可以通过使用 `grunt run` 命令来运行别名任务，就像运行常规任务一样，这应该会产生类似于以下内容的输出：

    ```js
    Running "clean:all" (clean) task
    >> 0 paths cleaned.

    Running "jade:www" (jade) task
    >> 1 file created.

    Running "connect:dev" (connect) task
    Started connect web server on http://localhost:8000

    Running "watch" task
    Waiting…

    ```

1.  如我们从输出中看到的那样，我们配置的任务都按照 `run` 任务中指定的顺序运行，我们现在正在提供渲染后的 `index.jade` 模板。为了测试设置，我们可以在浏览器中导航到 `http://localhost:8000`，这将显示渲染后的模板。

## 还有更多...

可能会有这样的情况，我们有一个特定的任务，我们希望比其他任务运行得多得多，以至于我们可能会把它视为默认任务。在这种情况下，我们可以使用 `default` 任务别名。当没有与 `grunt` 命令一起指定的特定任务时，将调用此别名。

我们可以通过更改别名任务声明为以下内容来修改我们的主菜谱以使用 `default` 任务：

```js
grunt.registerTask('default', [
  'clean:all',
  'jade:www',
  'connect:dev',
  'watch:www'
]);
```

如果我们现在在我们的项目中运行 `grunt` 命令而不跟随着任务名称，它应该以与上一节中相同的方式运行所有任务。

# 创建基本任务

尽管 Grunt 用户有大量的插件可用，但仍然可能发生我们想要创建自己的任务的情况。Grunt 提供了很好的支持，因为它提供了一套使创建新任务变得相当简单的实用工具。

## 准备工作

在本例中，我们将使用我们在 第一章 的 *在项目中安装 Grunt* 菜谱中创建的基本项目结构，*开始使用 Grunt*。如果您还不熟悉其内容，请务必参考它。

## 如何做到这一点...

以下步骤将引导我们创建一个简单的任务，该任务会打印当前系统日期和时间。

1.  首先，我们将使用名称 `datetime` 注册我们的任务，并为其提供描述和一个空函数作为占位符。以下代码显示了带有任务注册代码高亮的整个 Grunt 文件：

    ```js
    module.exports = function (grunt) {
      grunt.initConfig({});
      grunt.registerTask('default', []);
     grunt.registerTask(
     'datetime',
     'Prints out the current date and time.',
     function () {
     }
     );
    };
    ```

1.  在任务注册后，我们现在可以提供代码，该代码将实际打印当前日期和时间。以下代码显示了带有新添加代码高亮的任务注册：

    ```js
    grunt.registerTask(
      'datetime',
      'Prints out the current date and time.',
      function () {
        var date = new Date();
     grunt.log.writeln(date.toString());
      }
    );
    ```

    ### 小贴士

    这段代码使用了标准的 JavaScript `Date` 对象和 Grunt 的 `grunt.log` 工具，该工具专门为任务提供，用于记录所有类型的消息。

    您可以在以下网址了解更多关于 `grunt.log` 工具的信息：

    [`gruntjs.com/api/grunt.log`](http://gruntjs.com/api/grunt.log)

1.  现在，我们可以通过运行 `grunt datetime` 命令来尝试运行新创建的任务，它应该产生类似于以下内容的输出：

    ```js
    Running "datetime" task
    Thu Jan 1 2015 12:00:00 GMT

    ```

1.  如果任务的输出显示了当前系统日期和时间，那么我们就成功创建并执行了我们的简单自定义任务。

# 访问项目配置

为了使任务在项目中正常工作，它通常需要一种方式来获取有关它的特定细节。Grunt 项目的标准做法是在 `grunt.initConfig` 函数提供的配置对象中提供项目特定的细节。

## 准备工作

在这个例子中，我们将使用我们在 第一章 的 *在项目中安装 Grunt* 配方中创建的基本项目结构，*使用 Grunt 入门*。如果您还不熟悉其内容，请务必参考。

如果这个配方中的任何步骤看起来难以理解，请务必查看本章前面提供的 *创建基本任务* 配方。

## 如何操作...

以下步骤将引导我们创建一个打印出由项目配置确定的消息的任务。

1.  首先，我们将使用 `grunt.initConfig` 函数添加一些示例项目配置。以下代码显示了带有配置高亮的整个 Grunt 文件内容：

    ```js
    module.exports = function (grunt) {
      grunt.initConfig({
        location: 'Earth',
     datetime: '<%= new Date().toString() %>',
     project: {
     name: 'Life'
     }
      });
      grunt.registerTask('default', []);
    };
    ```

    ### 小贴士

    注意在 `datetime` 配置中使用模板字符串。以下网址提供了有关在配置值中使用模板字符串的更多信息：

    [`gruntjs.com/configuring-tasks#templates`](http://gruntjs.com/configuring-tasks#templates)

1.  接下来，我们将使用名称 `describe` 注册我们的任务，并为其提供描述和一个空函数作为占位符：

    ```js
    grunt.registerTask(
      'describe',
      'Describes the situation.',
      function () {
      }
    )
    ```

1.  在任务注册后，我们可以用以下代码填充空函数，该代码检索一些项目配置并在打印我们的消息时使用它：

    ```js
    var projectName = grunt.config('project.name');
    var location = grunt.config('location');
    var datetime = grunt.config('datetime');
    grunt.log.write(
      projectName + ' on ' + location + ' at ' + datetime
    );
    ```

    ### 小贴士

    在这里，我们使用 `grunt.config` 函数从 `grunt.initConfig` 函数提供的配置对象中获取数据。

    注意，在从配置对象中的对象检索属性时可以使用点符号，如 `projectName` 变量所示。

    使用 `datetime` 变量演示的另一个概念是，使用 `grunt.config` 函数检索的值是渲染过的，这意味着任何包含模板的字符串都将被解析和渲染。如果出于某种原因我们想要获取未渲染的值，我们可以通过使用 `grunt.config.getRaw` 函数来实现。

1.  现在，我们可以通过运行 `grunt describe` 命令来尝试我们的自定义任务，应该会产生类似于以下输出的结果：

    ```js
    Running "describe" task
    Life on Earth at Thu Jan 1 2015 12:00:00 GMT
    ```

1.  如果任务的输出显示了一个包含我们在项目配置中提供的值的消息，那么我们已经成功创建并执行了一个利用项目配置的任务。

# 检查所需配置

对于任务来说，通常需要一组最小配置才能正常工作是很常见的。Grunt 框架专门为此提供了 `grunt.config.requires` 函数，如果找不到指定的配置，则会失败任务。

## 准备工作

在这个例子中，我们将使用在 第一章 中 *在项目中安装 Grunt* 食谱中创建的基本项目结构，*开始使用 Grunt*。如果你还不熟悉其内容，请务必参考。

如果这个食谱中的任何步骤看起来难以理解，请务必查看本章前面提供的 *创建基本任务* 食谱。

## 如何操作...

以下步骤将引导我们创建两个任务：第一个将检查我们提供的所需配置，第二个将检查我们未提供的配置。

1.  首先，我们将使用 `grunt.initConfig` 函数添加一些示例项目配置。以下代码显示了包含配置高亮的整个 Grunt 文件的内容：

    ```js
    module.exports = function (grunt) {
      grunt.initConfig({
        integral: 'check',
     complex: {
     important: 'present'
     }
      });
    };
    ```

1.  接下来，我们将在 `grunt.initConfig` 调用之后添加以下代码，以注册一个名为 `complete` 的任务，该任务检查我们提供的配置：

    ```js
    grunt.registerTask(
      'complete',
      'Complete Configuration.',
      function () {
        grunt.config.requires('integral');
        grunt.config.requires('complex.important');
        grunt.log.write('Complete: Success!');
      }
    );
    ```

1.  然后，我们可以通过添加以下代码来注册一个名为 `incomplete` 的任务，该任务检查我们未提供的配置：

    ```js
    grunt.registerTask(
      'incomplete',
      'Incomplete Configuration.',
      function () {
        grunt.config.requires('missing');
        grunt.log.write('Incomplete: Success!');
      }
    );
    ```

1.  在添加了我们的任务之后，我们首先将运行一个成功的任务，使用 `grunt complete` 命令，应该会产生类似于以下输出的结果：

    ```js
    Running "complete" task
    Complete: Success!

    ```

1.  然后，我们可以使用 `grunt incomplete` 命令运行应该失败的任务，应该会产生类似于以下输出的结果：

    ```js
    Running "incomplete" task
    Verifying property missing exists in config...ERROR
    >> Unable to process task.

    Aborted due to warnings.

    ```

1.  如我们从最后一个任务的输出中看到的那样，它由于找不到 `missing` 配置属性而失败。

# 检查其他任务的执行成功

由于任务相互依赖会损害其可重用性，因此任务之间相互依赖的情况并不常见。如果任务紧密耦合到无法独立运行，应强烈考虑将它们合并为一个单独的任务。

话虽如此，总有例外，Grunt 通过`grunt.task.requires`函数提供了对此的处理。当调用此函数时，它会检查指定名称的任务是否已成功执行。如果它发现情况并非如此，它将使当前任务失败。

## 准备工作

在本例中，我们将使用我们在第一章中“在项目中安装 Grunt”配方中创建的基本项目结构，即“使用 Grunt 入门”。如果你还不熟悉其内容，请务必参考。

如果本配方中的任何步骤看起来难以理解，请务必查看本章前面提供的“创建基本任务”配方。

## 如何操作...

以下步骤将引导我们创建两个任务。第一个任务将打印一条简单的消息，第二个任务将检查第一个任务是否已成功执行。

1.  首先，我们将注册一个名为`independent`的任务，该任务可以在没有运行任何任务的情况下运行，并在完成时打印一条消息。这是通过将以下代码添加到我们的 Grunt 文件中的主函数中实现的：

    ```js
    grunt.registerTask(
      'independent',
      'Independent task.',
      function () {
        grunt.log.write('Independent: Success!')
      }
    );
    ```

1.  接下来，我们将注册一个名为`dependent`的任务，如果`independent`任务没有成功运行，它将会失败。这是通过将以下代码添加到我们的 Grunt 文件中的主函数中实现的：

    ```js
    grunt.registerTask(
      'dependent',
      'Dependent task.',
      function () {
        grunt.task.requires('independent');
        grunt.log.write('Dependent: Success!')
      }
    );
    ```

1.  为了测试这个设置，我们可以首先使用`grunt dependent`命令运行`dependent`任务，这应该会产生类似于以下内容的输出：

    ```js
    Running "dependent" task
    Warning: Required task "independent" must be run first. Use --force to continue.

    Aborted due to warnings.

    ```

1.  从输出中我们可以看到，这个任务失败了，因为在尝试运行“依赖”任务之前，“独立”任务并没有成功执行。

1.  最后，我们可以使用`grunt independent dependent`命令连续运行我们创建的两个任务，这应该会产生类似于以下内容的输出：

    ```js
    Running "independent" task
    Independent: Success!
    Running "dependent" task
    Dependent: Success!
    Done, without errors.

    ```

1.  这次我们可以看到，这两个任务都运行得很好，没有遇到任何问题，并打印了它们各自的消息。

# 在任务中运行非阻塞代码

在 Node.js 领域可用的大多数库通常都是按照非阻塞的方式实现的。这意味着调用库提供的函数后，通常会继续执行被调用之后的下一行代码，即使它还没有完成其预期的目的。

这些类型的函数通常会使用事件驱动方法或回调函数来指示它们的进度。在这两种情况下，我们都需要一个在任务完成后可以调用的函数，为此，Grunt 为我们提供了`this.async`实用工具。

## 准备工作

在本例中，我们将使用我们在第一章中“在项目中安装 Grunt”配方中创建的基本项目结构，即“使用 Grunt 入门”。如果你还不熟悉其内容，请务必参考。

如果本菜谱中的任何步骤看起来难以理解，请务必查看本章前面提供的 *创建基本任务* 菜谱。

## 如何操作...

以下步骤将指导我们创建一个任务，该任务每秒打印当前系统时间 5 秒，然后退出。

1.  首先，我们将通过在 Grunt 文件中的主函数末尾添加以下代码来注册一个名为 `clock` 的任务，包括一个描述和一个空的占位符函数：

    ```js
    grunt.registerTask(
      'clock',
      'Prints the time every second for 5 seconds.',
      function () {
      }
    );
    ```

1.  接下来，我们将在我们的占位符函数内部定义一些变量，这些变量将用于我们的时间打印代码：

    ```js
    var done = this.async();
    var count = 5;
    var interval = null;
    ```

    ### 小贴士

    `done` 变量是本菜谱的重点，因为它展示了 `this.async` 函数的使用。它被分配给调用 `this.async` 函数返回的函数。这个返回的函数可以被调用以指示任务是否已完成。

    `count` 变量将用于跟踪系统时间打印的次数。每次打印日期和时间时，它都会递减，并且当它达到 `0` 的值时，我们将停止打印时间。

    `interval` 变量用于存储在任务函数末尾启动的 **计时器间隔** 的 ID。我们需要这个 ID 来在 `count` 变量达到 `0` 的值时停止计时器间隔的运行。

1.  在所有必要的变量都已就绪后，我们现在可以定义一个函数，该函数将打印当前系统日期和时间，并在运行完毕后停止计时器。以下代码应该在之前的变量定义之后立即添加：

    ```js
    printTime = function () {
      grunt.log.writeln(new Date());
      count -= 1;
      if (count <= 0) {
        clearInterval(interval);
        done();
      }
    };
    ```

    ### 小贴士

    在这里，我们使用 `grunt.log` 工具通过提供一个新创建的内置 JavaScript `Date` 对象来打印当前系统日期和时间，该对象默认将始终包含当前日期和时间。

    我们还递减了计数变量，并检查它是否小于或等于 `zero`。如果我们发现它小于或等于零，我们将使用内置的 `clearInterval` 函数来停止计时器间隔的运行。我们将在下一步将计时器间隔 ID 分配给 `interval` 变量。

    一旦我们看到计数器达到 `0` 并且我们已经停止了时间间隔，我们可以使用 `this.async` 函数返回给我们的 `done` 函数来指示任务已运行完成。

1.  最后，我们将在 `printTime` 函数定义之后立即添加以下代码来启动一个每 `1000` 毫秒运行一次之前步骤中定义的函数的计时器间隔：

    ```js
    interval = setInterval(printTime, 1000);
    ```

1.  在我们的任务中包含所有必要的代码后，我们可以通过运行 `grunt clock` 命令来尝试它，它应该产生类似于以下内容的输出：

    ```js
    Running "clock" task
    Thu Apr 1 2015 12:00:00 GMT
    Thu Apr 1 2015 12:00:01 GMT
    Thu Apr 1 2015 12:00:02 GMT
    Thu Apr 1 2015 12:00:03 GMT
    Thu Apr 1 2015 12:00:04 GMT

    Done, without errors.

    ```

1.  如任务输出所示，它已打印当前系统日期和时间五次，然后停止。

# 任务失败

任务的成功或失败可以用来指示任务是否无问题完成，或者是否在执行其预期功能之前遇到了问题。默认情况下，Grunt 会假设任务已成功，但它也提供了一个简单的方法来指示任务是否失败以及传递其失败的原因。

## 准备工作

在本例中，我们将使用我们在第一章中“在项目中安装 Grunt”配方中创建的基本项目结构，即*开始使用 Grunt*。如果您还不熟悉其内容，请务必参考。

如果本配方中的任何步骤看起来难以理解，请务必查看本章前面提供的*访问项目配置*配方。

## 如何实现...

以下步骤将引导我们创建一个任务，其成功基于项目配置属性的值：

1.  我们将首先通过在 Grunt 文件中的主函数末尾添加以下代码来注册一个名为`check`的任务，并带有描述和空占位符函数：

    ```js
    grunt.registerTask(
      'check',
      'Fails if indicated in configuration.',
      function () {
      }
    );
    ```

1.  任务注册后，我们现在可以填充空函数，以检索项目配置属性并使用它来决定是否使任务失败：

    ```js
    var fail = grunt.config('shouldFail');
    if (fail === true) {
      return new Error('Error Message.');
    } else {
      grunt.log.writeln('Success!');
    }
    ```

    ### 小贴士

    注意，我们返回一个包含消息的新创建的`Error`对象，以指示任务已失败。如果我们只想使任务失败而不提供任何消息，我们只需返回`false`值即可。

1.  在我们的任务准备就绪后，我们可以添加所需的配置属性以指示它不应失败。以下代码显示了带有新配置高亮的整个`grunt.initConfig`调用：

    ```js
    grunt.initConfig({
      shouldFail: false
    });
    ```

1.  现在，我们可以运行任务以查看它是否按预期成功。这可以通过使用`grunt check`命令来完成，它应该产生类似于以下内容的输出：

    ```js
    Running "check" task
    Success!

    Done, without errors.

    ```

1.  现在我们已经看到任务成功执行，我们可以更改配置以指示我们希望它失败：

    ```js
    grunt.initConfig({
      shouldFail: true
    });
    ```

1.  如果我们现在再次使用`grunt check`命令运行任务，它应该产生类似于以下内容的输出：

    ```js
    Running "check" task
    Warning: Task "check" failed. Use --force to continue.

    Aborted due to warnings.

    ```

1.  如我们从输出中看到的那样，当将`shouldFail`配置属性设置为`true`时，任务如预期那样失败了。

## 更多内容...

由于任务有多种类型和结构，因此我们有多种方式来指示它们的失败。现在我们将查看如何使非阻塞任务失败以及如何在失败时立即终止任务。

### 失败的非阻塞任务

如果我们使用了`this.async`函数来指示任务是否以非阻塞方式运行，那么任务函数的返回值就变得没有意义了。

如果我们希望使一个非阻塞任务失败，我们可以通过调用`this.async`方法返回的函数的第一个参数来实现。使用此参数提供`false`或一个`Error`对象将指示任务已失败。

在以下示例中，我们将任务从主配方中修改为以非阻塞方式运行，并适当失败：

```js
grunt.registerTask(
  'check',
  'Fails if indicated in configuration.',
  function () {
 var done = this.async();
    var fail = grunt.config('shouldFail');
    if (fail === true) {
 done(new Error('Error Message.'));
    } else {
      grunt.log.writeln('Success!');
    }
  }
);
```

### 在失败时立即终止任务

有时，我们可能希望遇到失败时立即终止整个 Grunt 进程。这种类型的行为通常与通常所说的致命错误相关联。

以下示例使用了`grunt.fail.fatal`实用工具来指示发生了致命错误，并且 Grunt 进程应该立即退出，而无需任何进一步的延迟：

```js
grunt.registerTask(
  'check',
  'Fails if indicated in configuration.',
  function () {
    var fail = grunt.config('shouldFail');
    if (fail === true) {
      grunt.fail.fatal('Error message.');
    } else {
      grunt.log.writeln('Success!');
    }
  }
);
```

# 使用命令行参数

如果一个任务的配置方面需要经常更改，将其添加到项目配置中可能效率不高。这就是使用命令行参数可以派上用场的地方，因为它们为我们提供了一种轻松地为任务提供选项的方法，而无需更改我们的配置。

## 准备工作

在这个例子中，我们将使用在第一章中创建的基本项目结构，即*在项目中安装 Grunt*配方中的内容。如果您还不熟悉其内容，请务必参考。

如果这个配方中的任何步骤看起来难以理解，请务必查看本章前面提供的*创建基本任务*配方。

## 如何操作...

以下步骤将指导我们创建一个任务，该任务使用命令行参数接收两个字符串，并使用它们来打印一条消息：

1.  我们将首先通过在 Grunt 文件中的主函数末尾添加以下代码来注册一个名为`welcome`的任务，并添加一个描述和一个空的占位符函数：

    ```js
    grunt.registerTask(
      'welcome',
      'Displays a welcome message.',
      function () {
      }
    );
    ```

1.  在任务注册设置完成后，我们现在可以填充空函数，以接收命令行参数并使用它们来打印消息：

    ```js
    var message = 'Welcome to ' + city + ', ' + name + '!';
    grunt.log.writeln(message);
    ```

1.  现在，我们可以使用一些示例参数来运行任务。这是通过使用`grunt welcome:Aaron:London`命令来完成的，它应该产生类似于以下内容的输出：

    ```js
    Running "welcome:Aaron:London" (welcome) task
    Welcome to London, Aaron!

    ```

    ### 小贴士

    注意，如果您的任务使用了目标，第一个参数将始终被认为是目标的名称，并且不会传递给任务函数。

1.  从输出中我们可以看到，任务已成功接收我们提供的参数，并使用它们来显示欢迎消息。

# 将任务排队运行

有时候，我们可能希望在任务内部排队运行现有任务。这种做法并不推荐，因为设计良好的任务通常应该专注于特定的功能，并与其他任务保持松散耦合。以这种方式设计的任务通常在更广泛的场景中更健壮且更有用。

## 准备工作

在这个例子中，我们将使用我们在 第一章 中 *在项目中安装 Grunt* 菜谱中创建的基本项目结构，该菜谱在 *使用 Grunt 入门* 中。如果你还不熟悉其内容，请务必参考。

如果这个菜谱中的任何步骤看起来难以理解，请务必查看本章前面提供的 *创建基本任务* 菜谱。

## 如何操作...

以下步骤将引导我们创建两个简单的任务，其中一个任务打印一个字符串，而另一个任务在内部将其入队：

1.  我们将首先注册一个名为 `first` 的任务，该任务会打印出一个字符串。这可以通过将以下代码添加到我们的 Grunt 文件中的主函数末尾来完成：

    ```js
    grunt.registerTask('first', function() {
      grunt.log.write('First Task');
    });
    ```

1.  接下来，我们将注册另一个任务，该任务将紧跟在我们刚刚添加的任务之后。这个任务也会打印一个字符串，然后继续将 `first` 任务入队以运行：

    ```js
    grunt.registerTask('second', function() {
      grunt.log.write('Second Task');
      grunt.task.run('first');
    });
    ```

1.  现在我们已经创建了两个任务，我们可以通过运行 `grunt second` 命令来尝试它们，这应该会产生类似于以下输出的结果：

    ```js
    Running "second" task
    Second Task
    Running "first" task
    First Task

    ```

1.  如我们从输出中看到的那样，我们运行的第二个任务将第一个任务入队，该任务也运行了。

# 创建多任务

任务配置为我们使用在项目中的任务的功能调整提供了最常见的方式。通过配置，我们可以设置一个任务以满足我们的特定需求，并且它还允许我们在单个项目中以不同的方式使用同一个任务。应用于同一任务的不同的配置被称为 **任务目标**。

## 准备工作

在这个例子中，我们将使用我们在 第一章 中 *在项目中安装 Grunt* 菜谱中创建的基本项目结构，该菜谱在 *使用 Grunt 入门* 中。如果你还不熟悉其内容，请务必参考。

如果这个菜谱中的任何步骤看起来难以理解，请务必查看本章前面提供的 *创建基本任务* 菜谱。

## 如何操作...

以下步骤将引导我们创建一个简单的任务，该任务打印目标配置的名称和提供给它的数据：

1.  我们将首先使用 `registerMultiTask` 函数在 Grunt 文件的主函数末尾添加以下代码来注册一个名为 `display` 的多任务。

    ```js
    grunt.registerMultiTask(
      'display',
      'Displays target name and related configuration',
      function() {
      }
    );
    ```

1.  在任务注册后，我们可以用以下代码填充空函数，该代码打印包含指定目标名称和提供的数据的消息：

    ```js
    grunt.log.write(
      'Displaying ' + this.target + ' with ' + this.data
    );
    ```

    ### 小贴士

    注意，只有在使用 `registerMultiTask` 注册的任务函数的范围内，`this.target` 和 `this.data` 才会被填充为目标名称和配置数据。

1.  现在我们已经注册了一个功能正常的任务，我们可以为我们的 `grunt.initConfig` 调用设置一些示例配置，我们将用它来演示任务的行为：

    ```js
    grunt.initConfig({
      display: {
     foo: 'these configurations',
     bar: 'those configurations'
     }
    });
    ```

1.  在我们的任务创建和配置后，我们可以通过运行`grunt display`命令来尝试它，该命令应该产生类似于以下输出的结果：

    ```js
    Running "display:foo" (display) task
    Displaying foo with these configurations
    Running "display:bar" (display) task
    Displaying bar with those configurations

    ```

1.  现在，我们也可以尝试通过运行`grunt display:foo`命令来指示我们只想使用`foo`配置目标，该命令应该产生类似于以下输出的结果：

    ```js
    Running "display:foo" (display) task
    Displaying foo with these configurations

    ```

# 在任务中使用选项

调整任务功能的最常见方式是在项目配置中提供选项。选项可以通过在任务或目标级别提供一个对象到`options`属性来指定。如果同时在任务和目标级别提供了选项，则目标级别的选项将具有优先权。

## 准备工作

在本例中，我们将使用我们在第一章中创建的基本项目结构，即“在项目中安装 Grunt”配方中的结构，来工作。如果你还不熟悉其内容，请务必参考它。

如果这个配方中的任何步骤看起来难以理解，请务必查看本章前面提供的“创建多任务”配方。

## 如何操作...

以下步骤将引导我们创建一个多任务，该任务打印出在两个不同任务的两个可用级别提供的选项值：

1.  我们将首先使用`registerMultiTask`函数在 Grunt 文件中的主函数末尾添加以下代码来注册一个名为`display`的多任务：

    ```js
    grunt.registerMultiTask(
      'display',
      "Displays the value of the 'foo' option",
      function() {
      }
    );
    ```

1.  在任务注册后，我们可以用以下代码填充空函数，该代码打印包含`foo`选项值的消息：

    ```js
    var options = this.options();
    grunt.log.write("The value of foo is '" + options.foo + "'.");
    ```

    ### 小贴士

    注意，在任务函数的作用域中提供的`this.options`方法将自动合并任务和目标选项，如果目标中提供了任务选项，则将覆盖任务选项。

    想要了解更多关于`this.options`方法的信息，请访问以下网址：

    [`gruntjs.com/api/inside-tasks#this.options`](http://gruntjs.com/api/inside-tasks#this.options)

1.  现在我们已经注册并启用了一个任务，我们可以设置一些样本配置，我们将使用它来演示任务的行为：

    ```js
    grunt.initConfig({
      display: {
     options: {
     foo: 'initial'
     },
     first: {},
     second: {
     options: {
     foo: 'override'
     }
     }
     }
    });
    ```

1.  在我们的任务创建并提供了样本配置后，我们可以通过运行`grunt display:first`命令来尝试使用`first`目标，该命令应该提供类似于以下输出的结果：

    ```js
    Running "display:first" (display) task
    The value of foo is 'initial'.

    ```

1.  现在我们已经看到了`first`目标的行为，我们可以通过运行`grunt display:second`命令来尝试使用`second`目标，该命令应该产生类似于以下输出的结果：

    ```js
    Running "display:second" (display) task
    The value of foo is 'override'.
    ```

# 在任务中使用文件

任务通常需要访问一组特定的文件，无论是使用它们包含的数据，还是以某种方式修改它。Grunt 通过`src`、`dest`和`files`属性提供了一种统一的方式来指定文件集，这些属性可以在任务或目标级别使用。

## 准备工作

在本例中，我们将使用我们在第一章 Chapter 1 中创建的 *在项目中安装 Grunt* 配方中的基本项目结构，即 *开始使用 Grunt*。如果您还不熟悉其内容，请务必参考。

如果这个配方中的任何步骤看起来难以理解，请务必查看本章前面提供的 *创建多任务* 配方。

## 如何操作...

以下步骤将引导我们创建一些样本数据文件和一个简单的打印其内容的任务：

1.  首先，我们在名为 `data` 的新目录中创建两个名为 `one.dat` 和 `two.dat` 的样本文件，并给它们以下内容：

    ```js
    one.dat - Initial sample data.
    two.dat - Some more sample data.
    ```

1.  接下来，我们将使用 `registerMultiTask` 函数注册一个名为 `display` 的多任务，通过将以下代码添加到我们的 Grunt 文件中的主函数末尾：

    ```js
    grunt.registerMultiTask(
      'display',
      "Displays the contents of the specified files.",
      function() {
      }
    );
    ```

1.  注册任务后，我们现在可以填充空函数，以下代码将遍历指定的文件并打印其内容：

    ```js
    this.files.forEach(function(file) {
      file.src.forEach(function(filepath) {
        var content = grunt.file.read(filepath);
        grunt.log.write(content);
      });
    });
    ```

    ### 小贴士

    注意，`this.files` 属性仅在多任务函数的作用域内提供。它提供了一个标准化列表，包含在任务和目标级别提供的所有文件配置。

    关于 `this.files` 属性的更多信息，请访问以下网址：

    [`gruntjs.com/api/inside-tasks#this.files`](http://gruntjs.com/api/inside-tasks#this.files)

1.  为了演示我们的任务，我们现在可以在 `grunt.initConfig` 调用中添加一些配置，这将针对 `data` 目录中包含的所有文件：

    ```js
    grunt.initConfig({
      display: {
     sample: {
     src: 'data/*'
     }
     }
    });
    ```

1.  一切准备就绪后，我们现在可以通过运行 `grunt display` 命令来尝试任务，它应该产生类似于以下内容的输出：

    ```js
    Running "display:sample" (display) task
    Initial sample data.
    Some more sample data.

    ```

## 更多内容...

除了从文件中读取之外，我们可能还会达到想要写入文件的程度。以下步骤将修改之前的配方，以便将指定文件的内容写入目标文件而不是打印到控制台：

1.  首先，我们将修改之前注册的 `display` 任务的函数中的代码，改为以下内容：

    ```js
    this.files.forEach(function(file) {
      var content = file.src.map(function(filepath) {
     return grunt.file.read(filepath);
     }).join('');
     grunt.file.write(file.dest, content);
    });
    ```

1.  接下来，我们将修改项目配置，以指示输出应写入的目标文件：

    ```js
    grunt.initConfig({
      display: {
        sample: {
          src: 'data/*',
          dest: 'output.dat'
        }
      }
    });
    ```

1.  现在，我们可以通过再次运行 `grunt display` 命令来尝试我们的修改后的任务，它应该产生类似于以下内容的输出：

    ```js
    Running "display:sample" (display) task

    ```

1.  应该现在在我们的项目文件夹中有一个名为 `output.dat` 的文件，其内容如下：

    ```js
    Initial sample data.
    Some more sample data.
    ```
