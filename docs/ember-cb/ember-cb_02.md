# 第二章. Ember.Object 模型

在本章中，我们将介绍以下食谱：

+   与类和实例一起工作

+   与计算属性一起工作

+   在 Ember.js 中使用 Ember 观察者

+   与绑定一起工作

+   使用混合

+   使用数组与可枚举一起

# 简介

**Ember.Object** 是几乎所有其他 Ember 对象的基类。路由、模型、视图和组件都继承自 Ember.Object。它无处不在，因此了解它的工作方式和如何在我们的应用程序中使用它非常重要。

标准的 JavaScript 对象在 Ember 中不常用。Ember 的对象模型建立在 JavaScript 对象之上，并添加了重要的功能，如观察者、混合、计算属性和初始化器。许多这些功能都与新的 **ECMAScript** 标准保持一致。

# 与类和实例一起工作

创建和扩展类是 Ember 对象模型的主要功能之一。在本食谱中，我们将探讨创建和扩展对象的工作方式。

## 如何做到...

1.  让我们首先使用 `extend()` 创建一个非常简单的 `Ember` 类：

    ```js
    const Light = Ember.Object.extend({
      isOn: false
    });
    ```

    这定义了一个新的名为 `Light` 的类，它有一个名为 `isOn` 的属性。`Light` 类从 Ember 对象继承属性和行为，例如初始化器、混合和计算属性。

    ### 小贴士

    **Ember Twiddle 小贴士**

    在任何时候，你可能需要测试 Ember 代码的小片段。一个简单的方法是使用一个名为 **Ember Twiddle** 的网站。从这个网站，你可以创建一个 Ember 应用程序并在浏览器中运行它，就像使用 Ember CLI 一样。你甚至可以保存和分享它。它有类似 JSFiddle 的工具，但仅限于 Ember。在 [`ember-twiddle.com`](http://ember-twiddle.com) 上查看。

1.  一旦你定义了一个类，你需要能够创建它的一个实例。你可以使用 `create()` 方法来做这件事。我们将创建一个 `Light` 的实例：

    ```js
    constbulb = Light.create();
    ```

### 访问灯泡实例内的属性

1.  我们可以使用 `set` 和 `get` 访问器方法访问 `bulb` 对象的属性。让我们先获取 `Light` 类的 `isOn` 属性：

    ```js
    console.log(bulb.get('isOn'));
    ```

    上述代码将从 `bulb` 实例获取 `isOn` 属性。

1.  要更改 `isOn` 属性，我们可以使用 `set` 访问器方法：

    ```js
    bulb.set('isOn', true)
    ```

    现在 `isOn` 属性将被设置为 `true` 而不是 `false`。

### 初始化 Ember 对象

`init` 方法在创建新实例时被调用。这是一个放置任何可能需要为新实例编写的代码的好地方。

1.  在我们的例子中，我们将添加一个警报消息，显示 `isOn` 属性的默认设置：

    ```js
    const Light = Ember.Object.extend({
      init(){
        alert('The isON property is defaulted to ' + this.get('isOn'));
      },
      isOn: false
    });
    ```

1.  一旦执行了 `Light.create` 这行代码，实例将被创建，并且屏幕上会弹出**“isON 属性默认为 false”**的消息。

    ### 小贴士

    **子类**

    注意，你可以在 Ember 中创建你对象的子类。你可以重写方法并使用`_super()`关键字方法访问父类。这是通过创建一个新的对象来完成的，该对象使用父类的 Ember `extend`方法。

    另一个重要的事情是，如果你正在子类化一个框架类，例如`Ember.Component`，并且你重写了`init`方法，你需要确保调用`this._super()`。如果不这样做，组件可能无法正常工作。

### 重新打开类

在任何时候，你都可以重新打开一个类并在其中定义新的属性或方法。为此，请使用`reopen`方法。

在我们之前的示例中，我们有一个`isON`属性。让我们重新打开同一个类并添加一个`color`属性：

1.  要添加`color`属性，我们需要使用`reopen()`方法：

    ```js
        Light.reopen({
          color: 'yellow'
        });
    ```

1.  如果需要，你可以使用`reopenClass`添加静态方法或属性：

    ```js
    Light.reopenClass({
      wattage: 80
    });
    ```

1.  你现在可以访问静态属性`Light.wattage`。

## 它是如何工作的...

在前面的示例中，我们使用`extend`创建了 Ember 对象。这告诉 Ember 创建一个新的`Ember`类。`extend`方法在 Ember.js 框架中使用继承。`Light`对象继承了 Ember 对象的所有方法和绑定。

`create`方法也继承自 Ember 对象类，并返回该类的新实例。`bulb`对象是我们创建的 Ember 对象的新实例。

## 更多...

要使用前面的示例，我们可以创建自己的模块并将其导入到我们的项目中。

1.  要这样做，在`app`文件夹中创建一个名为`MyObject.js`的新文件：

    ```js
    // app/myObject.js
    import Ember from 'ember';
    export default function() {
        const Light = Ember.Object.extend({
          init(){
            alert('The isON property is defaulted to ' +this.get('isOn'));
          },
          isOn: false
        });

        Light.reopen({
          color: 'yellow'
        });

        Light.reopenClass({
          wattage: 80
        });

        const bulb = Light.create();

        console.log(bulb.get('color'));
        console.log(Light.wattage);
    }
    ```

    这是一个我们可以现在导入到我们的 Ember 应用程序中任何文件的模块。

1.  在`app`文件夹中，编辑`app.js`文件。你需要在文件顶部添加以下行：

    ```js
    // app/app.js
    import myObject from './myObject';
    ```

1.  在导出之前，添加以下行：

    ```js
    myObject();
    ```

    这将执行我们在`myObject.js`文件中创建的`myObject`函数。在运行`ember server`之后，你会看到`isOn`属性默认显示为`false`的弹出消息。

# 使用计算属性

在这个菜谱中，我们将查看计算属性以及它们如何用于显示数据，即使这些数据在应用程序运行时发生变化。

## 如何做到这一点...

让我们创建一个新的 Ember.Object 并给它添加一个计算属性：

1.  让我们首先创建一个新的`description`计算属性。这个属性将反映`isOn`和`color`属性的状态：

    ```js
    const Light = Ember.Object.extend({
      isOn: false,
      color: 'yellow',

      description: Ember.computed('isOn','color',function() {
        return 'The ' + this.get('color') + ' light is set to ' + this.get('isOn');
      })

    });
    ```

1.  我们现在可以创建一个新的`Light`对象并获取计算属性`description`：

    ```js
    const bulb = Light.create();
    bulb.get('description'); //The yellow light is set to false
    ```

    前面的示例创建了一个依赖于`isOn`和`color`属性的计算属性。当调用`description`函数时，它返回一个描述灯的状态的字符串。

    计算属性将观察变化，并在它们发生时动态更新。

1.  要看到这个功能的效果，我们可以修改前面的示例并将`isOn`属性设置为`false`。使用以下代码来完成此操作：

    ```js
    bulb.set('isOn', true);
    bulb.get('description') //The yellow light is set to true
    ```

    描述已经自动更新，现在将显示`The yellow light is set to true`。

### 连接 Light 对象

Ember 提供了一个很好的功能，允许计算属性存在于其他计算属性中。在先前的例子中，我们创建了一个 `description` 属性，该属性输出了有关 `Light` 对象的一些基本信息。

1.  让我们添加另一个属性，该属性提供完整的描述：

    ```js
    const Light = Ember.Object.extend({
      isOn: false,
      color: 'yellow',
      age: null,

      description: Ember.computed('isOn','color',function() {
        return 'The ' + this.get('color') + ' light is set to ' + this.get('isOn');
      }),

      fullDescription: Ember.computed('description','age',function() {
        return this.get('description') + ' and the age is ' + this.get('age')
      }),

    });
    ```

1.  `fullDescription` 函数返回一个字符串，该字符串将描述的输出与显示 `age` 的新字符串连接起来：

    ```js
    const bulb = Light.create({age:22});
    bulb.get('fullDescription'); //The yellow light is set to false and the age is 22
    ```

    在这个例子中，在 `Light` 对象实例化期间，我们将 `age` 设置为 `22`。如果需要，我们可以覆盖任何属性。

### 别名

`Ember.computed.alias` 方法允许我们创建一个属性，该属性是另一个属性或对象的别名。

1.  对 `get` 或 `set` 的任何调用都将表现得像是更改了原始属性：

    ```js
    const Light = Ember.Object.extend({
      isOn: false,
      color: 'yellow',
      age: null,
      description: Ember.computed('isOn','color',function() {
        return 'The ' + this.get('color') + ' light is set to ' + this.get('isOn');
      }),
      fullDescription: Ember.computed('description','age',function() {
        return this.get('description') + ' and the age is ' + this.get('age')
      }),
     aliasDescription: Ember.computed.alias('fullDescription')
    });

    const bulb = Light.create({age: 22});
    bulb.get('aliasDescription');//The yellow light is set to false and the age is 22.
    ```

1.  `aliasDescription` 别名将显示与 `fullDescription` 相同的文本，因为它只是这个对象的别名。如果我们稍后对 `Light` 对象中的任何属性进行了更改，别名也会观察这些更改并正确计算。我们将在 *使用绑定* 的配方中进一步讨论这一点。

## 它是如何工作的...

计算属性建立在观察者模式之上。每当观察显示状态变化时，它都会重新计算输出。如果没有变化发生，则结果将被缓存。

换句话说，计算属性是当它们的依赖值中的任何一个发生变化时都会更新的函数。您可以使用它们的方式与您使用静态属性的方式几乎相同。它们在 Ember 及其代码库中很常见且很有用。

请记住，计算属性只有在它位于模板或正在使用的函数中时才会更新。如果函数或模板没有被调用，则不会发生任何事情。这将有助于提高性能。

# 在 Ember.js 中使用 Ember 观察者

观察者是 Ember 对象模型的基础。在下一个配方中，我们将使用我们的灯示例，添加一个观察者，并查看它是如何运行的。

## 如何做到这一点...

1.  首先，我们将添加一个名为 `isOnChanged` 的新观察者。它只会在 `isOn` 属性更改时触发：

    ```js
    const Light = Ember.Object.extend({
      isOn: false,
      color: 'yellow',
      age: null,
      description: Ember.computed('isOn','color',function() {
        return 'The ' + this.get('color') + 'light is set to ' + this.get('isOn')
      }),
      fullDescription: Ember.computed ('description','age',function() {
        return this.get('description') + ' and the age is ' + this.get('age')
      }),
      desc: Ember.computed.alias('description'),
     isOnChanged: Ember.observer('isOn',function() {
     console.log('isOn value changed')
     })
    });

    const bulb = Light.create({age: 22});

    bulb.set('isOn',true); //console logs isOn value changed
    ```

    `Ember.observer` `isOnChanged` 监视 `isOn` 属性。如果此属性发生任何更改，`isOnChanged` 将被调用。

    ### 注意

    **计算属性与观察者**

    初看之下，观察者可能看起来与计算属性相同。实际上，它们非常不同。计算属性可以使用 `get` 和 `set` 方法，并且可以在模板中使用。另一方面，观察者只是监视属性变化，不能在模板中使用或像属性一样访问。它们也不返回任何值。因此，请注意不要过度使用观察者。在许多情况下，计算属性是更合适的解决方案。

1.  此外，如果需要，您可以为观察者添加多个属性。只需使用以下代码：

    ```js
    Light.reopen({  
    isAnythingChanged: Ember.observer('isOn','color',function() {
        console.log('isOn or color value changed')
      })
    });

    const bulb = Light.create({age: 22});
    bulb.set('isOn',true); // console logs isOn or color value changed
    bulb.set('color','blue'); // console logs isOn or color value changed
    ```

    当`isOn`或`color`属性变化时，`isAnything`观察者会被调用。观察者会触发两次，因为每个属性都发生了变化。

### 与 Light 对象和观察者的同步问题

观察者很容易变得不同步。例如，如果它观察的属性发生变化，它将按预期被调用。调用后，它可能会操作一个尚未更新的属性。由于所有事情都同时发生，这可能导致同步问题。

1.  以下示例展示了这种行为：

    ```js
      Light.reopen({
        checkIsOn: Ember.observer('isOn', function() {
          console.log(this.get('fullDescription'));
        })
      });

    const bulb = Light.create({age: 22});
    bulb.set('isOn', true);
    ```

    当`isOn`发生变化时，不清楚计算属性`fullDescription`是否已经更新。由于观察者同步工作，很难判断已经触发和改变的内容。这可能导致意外的行为。

1.  为了解决这个问题，最好使用`Ember.run.once`方法。这是 Ember `run`循环的一部分，是 Ember 管理代码执行的方式。重新打开`Light`对象，你会看到以下内容：

    ```js
    Light.reopen({
        checkIsOn: Ember.observer('isOn','color', function() {
          Ember.run.once(this,'checkChanged');
        }),
        checkChanged: Ember.observer('description',function() {
          console.log(this.get('description'));
        })
    });
    const bulb = Light.create({age: 22});
    bulb.set('isOn', true);
    bulb.set('color', 'blue'); 
    ```

    `checkIsOn`观察者使用`Ember.run.once`调用`checkChanged`观察者。此方法在每个`run`循环中只运行一次。通常，`checkChanged`会被触发两次；然而，由于它是通过`Ember.run.once`调用的，它只输出一次。

## 它是如何工作的...

Ember 观察者是来自`Ember.Observable`类的混入。它们通过监控属性变化来工作。当任何变化发生时，它们会被触发。请注意，这些与计算属性不同，不能在模板中使用，也不能与获取器或设置器一起使用。

# 与绑定一起工作

大多数框架都包含某种绑定实现。Ember 也不例外，它有可以与任何对象一起使用的绑定。以下食谱解释了如何使用它们以及单向和双向绑定。

## 如何做到这一点...

在这个例子中，有一个教师和学生 Ember 对象。每个都有自己的属性集，并且它们都有 homeroom。我们可以通过为教师对象设置别名来共享 homeroom。

1.  让我们从创建一个教师和学生`Ember.Object`开始：

    ```js
    const Teacher = Ember.Object.extend({
      homeroom: '',
      age: '',
      gradeTeaching: ''
    });

    const Student = Ember.Object.extend({
      homeroom: Ember.computed.alias('teacher.homeroom'),
      age: '',
      grade: '',
      teacher: null
    });
    ```

    学生`homeroom`是`Ember.computed.alias`，它将`homeroom`属性绑定到`teacher.homeroom`。

1.  接下来，我们将实例化`teacher`和`student`对象：

    ```js
    const avery = Teacher.create({
      age: '27',
      homeroom: '1075',
      gradeTeaching: 'sophmore'
    });

    const joey = student.create({
      age: '16',
      grade: 'sophmore',
      teacher: avery
    });
    ```

    `joey`对象将`homeroom`属性设置为`avery`，这是我们刚刚创建的`teacher`对象。

1.  我们现在可以使用`console.log`来输出我们的发现：

    ```js
    console.log(joey.get('age')); //16
    console.log(avery.get('homeroom')); //1075
    avery.set('homeroom','2423');
    console.log(joey.get('homeroom')); //2423
    ```

    如你所见，每当`avery`对象更改其`homeroom`时，学生`joey`的`homeroom`也会改变。这是因为 joey 的 homeroom 是教师`avery`的别名。

1.  你并不总是需要访问引用其他对象的属性。你可以绑定到任何东西：

    ```js
    const myName = Ember.Object.extend({
      name: 'Erik Hanchett',
      otherName: Ember.computed.alias('name')
    });

    const erik = myName.create();

    console.log(erik.get('name')); //Erik Hanchett
    console.log(erik.get('otherName')); //Erik Hanchett
    ```

    别名指向`name`；因此，当打印到控制台时，它显示`Erik Hanchett`。

    ### 注意

    Ember 有一个名为 `Ember.Binding` 的类。这是一个具有与 `Ember.computed.alias` 和 `Ember.computed.oneWay` 非常相似的行为和功能的公共类。你应该使用 `Ember.computed.alias`，而不是 `Ember.Binding`。计算别名是 Ember 中绑定的首选方法。`Ember.Binding` 仍然存在，并可能在某个时候被弃用。

### 单向绑定

Ember 默认使用一种称为双向绑定的机制。这意味着当 UI 中的属性发生变化时，这些变化会更新回控制器或组件。另一方面，单向绑定只在一个方向上传播变化。

例如，假设我们有一个具有 `firstName`、`lastName` 和 `nickName` 属性的 `User` 对象。我们可以使用 `Ember.computed.oneWay` 为 `firstName` 属性创建一个单向绑定。

让我们看看当我们尝试修改它时会发生什么。创建一个新的用户对象并具有这些属性。实例化对象并尝试更改属性：

```js
const User = Ember.Object.extend({
  firstName: null,
  lastName: null,
  nickName: Ember.computed.oneWay('firstName')
});

const user = User.create({
  firstName: 'Erik',
  lastName:  'Hanchett'
});

console.log(user.get('nickName'));              // 'Erik'
user.set('nickName', 'Bravo'); // 'Bravo'
console.log(user.get('firstName'));             // 'Erik'
```

你可以看到，即使用户已经更新，`nickName` 也没有改变。你可以将单向绑定视为使用 `Ember.computed.alias`。然而，它只允许你获取值，而不能设置值。使用 `Ember.computed.oneWay` 时，上游属性不会发生变化。

## 它是如何工作的...

Ember 绑定被用于 Ember 框架的许多部分。它们是从 `Ember.computed` 命名空间派生出来的。在这个命名空间中是计算别名方法。计算别名通过创建双向绑定来指定另一个对象的路径。

绑定对象不会立即更新。Ember 会等待所有应用程序代码运行完毕后再同步所有更改。这防止了在值更新时同步绑定产生不必要的开销。

单向绑定通过只单向传播信息来工作。信息不会在上游属性中更新。

# 使用混入

混入是 Ember 中重用和共享代码的绝佳方式。以下配方将介绍如何在代码中使用它们的一些基本操作。

## 如何做到这一点...

在这个配方中，我们将创建一个通用的混入对象。

1.  创建一个具有一些属性和函数的 Ember 混入对象：

    ```js
    const common = Ember.Mixin.create({
        property1: 'This is a mixin property',
        edit: function() {
          console.log('Starting to edit');
          this.set('isEditing', true);
        },
        isEditing: false
    });
    ```

    这个混入可以被添加到任何对象中。为了简单起见，这个混入所做的只是显示一些文本，并在调用 `edit` 函数时将 `isEditing` 属性设置为 `true`。

1.  让我们看看当我们把这个对象添加到另一个对象中时它看起来是什么样子：

    ```js
    const obj = Ember.Object.extend(common,{
      objprop: 'This is an Ember object property'
    });

    const object = obj.create();
    ```

    `Ember.Object` 中存在的 `extend` 方法允许有一个或多个可选的 `Ember.Mixin` 类型的参数。在这个例子中，我们将通用混入添加到新的 `Ember.Object` 对象中。然后我们使用 `create` 实例化这个 Ember 对象。

1.  剩下的就是输出内容。使用 `console.log` 来显示每个属性：

    ```js
    console.log(object.get('objprop'));  //This is an Ember object property
    console.log(object.get('property1'));  //This is a mixin property
    console.log(object.get('isEditing'));  //false
    object.edit();  //Starting to edit
    console.log(object.get('isEditing')); //true
    ```

    这就是输出将看起来像什么。如您所见，我们可以访问 mixin 的任何属性或方法，就像 mixin 被包含在 Ember 对象本身中一样。这是在应用程序中重用代码的一种方便方式。

1.  让我们创建另一个 mixin：

    ```js
    const secondMixin = Ember.Mixin.create({
      secondProperty: 'This is the second mixin property'
    });
    ```

1.  现在让我们看看如果我们将其添加到 Ember 对象中会是什么样子：

    ```js
    const obj = Ember.Object.extend(common,secondMixin,{
      objprop: 'This is an Ember object Property'
    });
    ```

1.  现在，我们可以在我们的对象中访问公共的`secondMixin`。我们可以使用`console.log`来输出`secondProperty`：

    ```js
    console.log(object.get('secondProperty'));//This is the   second mixin propety
    ```

### 使用 Ember CLI 的 mixin

Mixins 与 Ember CLI 配合得非常好。首先，使用 mixin 生成器创建一个。

1.  确保你处于应用程序目录中，然后输入以下命令：

    ```js
    $ ember generate mixin common
    installing mixin
     create app/mixins/common.js
    installing mixin-test
     create tests/unit/mixins/common-test.js

    ```

    `generator`命令会创建一个`app/mixins`文件夹和`common.js`文件。`common.js`文件是我们将放置 mixin 代码的地方。

1.  我们将使用前面的例子中的 mixin 并将其添加到这个文件中：

    ```js
    // app/mixins/common.js
    import Ember from 'ember';

    export default Ember.Mixin.create({
        property1: 'This is a mixin property',
        edit: function() {
          console.log('Starting to edit');
          this.set('isEditing', true);
        },
        isEditing: false
    });
    ```

    这个 mixin 与前面的例子完全相同；然而，现在它在一个我们可以导入到任何地方的模块中，包括组件或控制器。

    现在，我们将将其导入到`app`文件夹目录中的`app.js`文件中。

1.  首先，我们需要在文件顶部添加`import`语句：

    ```js
    import common from './mixins/common';
    ```

    这允许我们在`app.js`文件的任何地方使用公共 mixin。

1.  我们将在`app/app.js`文件的底部添加以下代码：

    ```js
    // app/app.js

    const obj = Ember.Object.extend(common,{
      objprop: 'This is an Ember object property'
    });

    const object = obj.create();

    console.log(object.get('objprop'));//This is an Ember object property
    console.log(object.get('property1'));//This is a mixin property
    console.log(object.get('isEditing'));//false
    object.edit();  //Starting to edit
    console.log(object.get('isEditing'));  //true
    ```

    如您所见，公共 mixin 中的所有属性和方法都可用于该对象。

1.  如果我们将公共 mixin 添加到组件中，它可能看起来像以下代码。将此代码添加到`common-example.js`文件中：

    ```js
    // app/components/common-example.js
    import Ember from 'ember';
    import common from '../mixins/common';

    export default Ember.Component.extend(common,{
        compprop: 'This is a component property',
        actions: {
          pressed: function(){
            this.edit();
          }
        }
    });
    ```

    和往常一样，我们必须首先将 mixin 导入到我们的组件中。路径始终相对于你所在的目录，因此，我们必须使用`../mixins/common`来找到它。

    在组件中，我添加了一个简单的动作`pressed`，它触发 mixin 的`edit`方法。如果动作被触发，我们将在控制台看到`Starting to edit message`。更多组件的例子请参考第六章，*Ember 组件*。

## 它是如何工作的...

`Ember.Mixin`类允许创建可以将其属性和方法添加到其他类的 mixin。它们不能被实例化，但可以被添加或*混合*。

计算机科学中的 mixin 是一个类，它通过组合而不是继承将其行为借用到借用类中。它鼓励代码重用，并避免了多重继承可能引起的歧义。

# 使用数组中的可枚举

当处理数组时，`Ember.Enumerable`方法非常重要。在这些菜谱中，我们将查看一些常见用例。

## 准备工作

要了解如何使用可枚举的，我们首先需要看看标准的 JavaScript 数组方法和它们使用可观察枚举的等效方法：

| 标准方法 | 可观察的等效方法 |
| --- | --- |
| `unshift` | `unshiftObject` |
| `shift` | `shiftObject` |
| `reverse` | `reverseObjects` |
| `push` | `pushObject` |
| `pop` | `popObject` |

我们将在示例中使用一些这些方法，所以请记住它们的标准和可观察的等效方法。

`Ember.Enumerable`类有几个我们可以在我们的 Ember 应用程序中使用的方法。以下是更常见的方法列表以及它们的作用：

| 枚举方法 | 定义 |
| --- | --- |
| `forEach` | 这遍历枚举，对每个项调用传递的函数 |
| `firstObject` | 这返回集合中的第一个对象 |
| `lastObject` | 这返回集合中的最后一个对象 |
| `map()` | 这会将枚举中的所有项映射到另一个值，类似于 JavaScript 1.6 中的 map |
| `mapBy()` | 与 map 类似，这返回枚举中所有项的命名属性的值 |
| `filter` | 这返回一个数组，包含枚举中传递的函数返回 true 的所有项 |
| `find` | 这返回方法返回的第一个数组项 |
| `findby` | 这返回第一个具有与传递的值匹配的属性的项 |
| `every` | 这仅在传递的函数对枚举中的每个项返回 true 时才返回 true |
| `any` | 这仅在传递的函数对枚举中的任何项返回 true 时才返回 true |

许多这些方法与它们的 JavaScript 对应方法类似。如果您知道如何使用 JavaScript 方法，您应该能够使用 Ember 的等效方法。

## 如何做到这一点...

`Ember.Enumerables`将 Ember 对象的全部良好特性添加到枚举中。我们将查看几个示例，说明如何做到这一点。所有这些食谱的内容都在`app.js`文件中的`chapter2/example6`文件夹中。

### 使用数组中的 forEach

枚举的一个非常常见的用例是使用`forEach`遍历数组。

1.  创建一个学生数组：

    ```js
    const students = ['Erik', 'Jim', 'Shelly', 'Kate'];
    ```

1.  使用`forEach`枚举遍历数组：

    ```js
    students.forEach(function(item, index) {
      console.log(`Student #${index+1}: ${item}`);
    });
    ```

    控制台输出将显示数组中的每个学生的姓名：

    ```js
    Student #1: Erik
    Student #2: Jim
    Student #3: Shelly
    Student #4: Kate

    ```

    ### 提示

    **模板字面量**

    Ember 与最新的 ECMAScript 2015 兼容。一个很酷的新特性被称为模板字面量或模板字符串。模板字面量是可以在多行中扩展并包含插值表达式的字符串字面量。您可以通过在字符串中包围变量来进行字符串插值，如下所示`${}`。每个变量将按前一个`forEach`示例中所示的方式显示在字符串中。

### 使用数组中的 map

`map`方法接受一个数组，映射每个项，并返回一个新的修改后的数组。假设我们想将学生姓名全部转换为大写。我们可以使用`map`来实现这一点。

1.  创建一个`students`列表：

    ```js
    const students = ['Erik', 'Jim', 'Shelly', 'Kate'];
    ```

    第一个字母需要大写；然而，我们希望所有的字母都大写。

1.  使用`map`将每个项目转换为大写：

    ```js
    const upperCaseStudent= students.map(function(item) {
      return item.toUpperCase();
    });
    ```

    数组中的每个项目都已转换为大写。新的`upperCaseStudent`数组包含所有新值。

1.  使用`forEach`枚举遍历数组中的每个项目并显示其内容：

    ```js
    upperCaseStudent.forEach(function(item, index) {
      console.log(`student #${index+1}: ${item}`);
    });
    ```

    输出显示了新 `upperCaseStudent` 数组中的每个名称：

    ```js
    student #1: ERIK
    student #2: JIM
    student #3: SHELLY
    student #4: KATE

    ```

### 使用 `mapBy` 和一个数组

如果你的数组由对象组成，可以使用 `mapBy` 可枚举来提取每个对象的命名属性并返回一个新数组。

1.  让我们创建一个教师和学生对象：

    ```js
    const student = Ember.Object.extend({
      name: 'Erik Hanchett'
    });

    const teacher = Ember.Object.extend({
      name: 'John P. Smith'

    });
    ```

    每个对象都有一个名为 `name` 的属性：

1.  接下来，我们将实例化每个对象。

    ```js
    const t= teacher.create();
    const s = student.create();
    const people = [s, t];
    ```

    将每个对象放入 `people` 数组中：

1.  我们可以使用 `mapBy` 来创建一个新数组。

    ```js
    console.log(people.mapBy('name'));//['Erik Hanchett', 'John P.   Smith']
    ```

    这个新数组返回了两个对象 `name` 属性的值。

### 在数组中查找第一个和最后一个对象

如果需要，我们有一个简单的方法来获取数组中的第一个和最后一个对象。

1.  我们首先创建一个学生数组：

    ```js
    const students = ['Erik', 'Jim', 'Shelly', 'Kate', 'Jenny', 'Susan'];
    ```

    这个数组有六个不同的学生。

1.  让我们获取数组中的最后一个对象：

    ```js
    console.log(students.get('lastObject')); //Susan
    ```

    这将显示数组中的最后一个对象 `Susan`。

1.  现在让我们检索数组中的第一个对象：

    ```js
    console.log(students.get('firstObject')); //Erik
    ```

    这将显示数组中的第一个项目 `Erik`。

1.  我们也可以将对象推送到数组中：

    ```js
    students.pushObject('Jeff');
    ```

1.  学生 `Jeff` 现在已经被添加到列表中：

    ```js
    console.log(students.get('lastObject')); //Jeff
    ```

### 使用过滤器进行有趣的操作

一个非常常见的做法是取一个数组并返回一个过滤后的项目列表。

1.  首先，创建一个数字数组：

    ```js
    const array = [1,2,5,10,25,23];
    ```

1.  将 `array` 和 `filter` 结合起来，只返回那些大于 `10` 的数字：

    ```js
    const newArray =array.filter(function(item, index, self) {
      return item > 10;
    })
    ```

1.  使用 `console.log` 来显示新数组：

    ```js
    console.log(newArray); //[25,23]
    ```

    这个新数组中只包含大于 `10` 的数字。

### 使用 `filterBy` 和一组对象

使用 `filterBy`，你可以通过某些属性过滤一组对象。

1.  创建一个新的 `student` 对象，它有一个 `name` 和 `grade`：

    ```js
    const student = Ember.Object.extend({
      grade: null,
      name: null
    });
    ```

1.  将学生添加到新数组中：

    ```js
    const listOfStudents = [
      student.create({grade: 'senior', name: 'Jen Smith'}),
      student.create({grade: 'sophmore', name: 'Ben Shine'}),
      student.create({grade: 'senior', name: 'Ann Cyrus'})
    ];
    ```

1.  使用 `filterBy` 来显示高年级学生：

    ```js
    const newStudent = listOfStudents.filterBy('grade','senior');
    ```

    这将返回一个包含所有高年级学生的数组。

1.  我们可以使用 `forEach` 来双重检查输出：

    ```js
    newStudent.forEach(function(item,index){
      console.log(item.get('name'));
    });
    Jen Smith
    Ann Cyrus
    ```

### 使用 `find` 来获取第一个匹配项

`find` 可枚举与 `filter` 的工作方式非常相似，但它会在找到第一个匹配项后停止。

1.  创建一个数字数组：

    ```js
    const array = [1,2,5,10,25,23];
    ```

1.  使用 `array.find` 来检索列表中第一个大于 `10` 的数字：

    ```js
    const newArray =array.find(function(item, index){
      return item > 10;
    }); 
    ```

1.  然后，我们将检查新数组的输出：

    ```js
    console.log(newArray); //25
    ```

    答案是 `25`，因为它是列表中第一个大于 `10` 的数字。

### 使用 `findBy` 和集合

`findBy` 可枚举与 `filterBy` 的工作方式非常相似，但它会在找到第一个匹配项后停止。

1.  创建一个新的 `student` 对象：

    ```js
    const student = Ember.Object.extend({
      grade: null,
      name: null
    });
    ```

1.  接下来，创建一个学生数组：

    ```js
    const listOfStudents = [
      student.create({grade: 'senior', name: 'Jen Smith'}),
      student.create({grade: 'sophmore', name: 'Ben Shine'}),
      student.create({grade: 'senior', name: 'Ann Cyrus'})
    ];
    ```

1.  使用 `findBy` 来匹配只有 `grade` 为 `senior` 的属性：

    ```js
    const newStudent = listOfStudents.findBy('grade','senior');
    ```

1.  这将返回第一个高年级学生：

    ```js
    console.log(newStudent.get('name')); //Jen Smith
    ```

    `Jen Smith` 是第一个符合这个标准的学生，因此它被返回到 `newStudent` 数组中。

### 使用 `every` 可枚举来学习

`every` 可枚举只有在每个项目都符合某个特定条件时才会返回 `true`。

1.  开始创建一个数字数组：

    ```js
    const array = [11,25,23,30];
    ```

1.  使用 `every` 可枚举来检查数组中的每个项目是否大于 `10`：

    ```js
    console.log(array.every(function(item, index, self) {
      return item > 10;
    })); //returns true
    ```

    这返回 `true`，因为数组中的每个项目都大于 `10`。

### 使用 `any` 来找到至少一个匹配项

`any` 可枚举将在至少有一个项目符合某个条件时返回 `true`。

1.  再次创建一个数字列表：

    ```js
    const array = [1,2,5,10,25,23];
    ```

1.  使用 `any` 可枚举来检查这个数组中的这些数字是否超过 `10`：

    ```js
    console.log(array.any(function(item, index, self) {
      return item > 10;
    })); //returns true
    ```

    这将返回 `true`，因为至少有一个数字超过了 `10`。

## 它是如何工作的...

`Ember.Enumerable` 混合是 Ember 对 JavaScript 1.8 之前定义的数组 API 的实现。它在页面加载时自动应用，因此任何方法都是可用的。为了使 Ember 能够观察可枚举的变化，你必须使用 `Ember.Enumerable`。

可枚举 API 尽可能遵循 ECMAScript 规范，以最小化与其他库的不兼容性。在可用的情况下，它使用原生的浏览器实现。
