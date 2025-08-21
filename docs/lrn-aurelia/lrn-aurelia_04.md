# 第四章：表单及其验证方式

在本章中，我们将了解数据绑定如何适用于用户输入元素，如`input`、`select`和`textarea`。我们还将了解当处理比简单 GET 请求更复杂的场景时，Fetch 客户端如何工作，例如带有 JSON 主体的 POST 或 PUT 请求，或者向服务器上传文件的需求。

此外，我们还将了解如何使用`aurelia-validation`插件验证表单。

最后，我们将讨论使用`aurelia-dialog`插件创建复杂表单的各种策略，从内联列表编辑到使用模态窗口编辑。

# 绑定表单输入

Aurelia 支持所有官方 HTML5 用户输入元素的双向绑定。其中一些相当简单易用，比如`text input`，我们已经在前面的章节中用许多示例探索过。其他的，如单选按钮或复选框，则不太直接。让我们逐一了解它们。

以下部分中的代码片段摘自`chapter-4/samples/binding-forms`。

## 选择元素

对于`select`元素，我们通常绑定到它的`value`属性，并且经常使用`repeat.for`来渲染它的`option`元素：

```js
<template> 
  <select value.bind="selectedCountry"> 
    <option>Select your country</option> 
    <option repeat.for="country of countries"  
            value.bind="country">${country}</option> 
  </select> 
</template> 

```

当然，`select`元素的`value`属性默认绑定双向，所以选中的`option`元素的`value`将分配给绑定到`select`的`value`属性的表达式。在此示例中，`selectedCountry`属性将被分配选中的`country`值。

`option`元素的`value`属性只期望字符串值。在前一个示例中，`countries`属性是一个字符串数组，因此每个`option`的`value`绑定到一个字符串。要渲染绑定到任何其他类型值的`option`——例如一个对象——必须使用特殊的`model`属性：

```js
<template> 
  <select value.bind="selectedCulture"> 
    <option>Select your culture</option> 
    <option repeat.for="culture of cultures"  
            model.bind="culture">${culture.name}</option> 
  </select> 
</template> 

```

在此，`selectedCulture`属性将被赋值为选中的`culture`对象，因为`cultures`属性是一个对象数组。

或者，如果你需要选择一个键属性，比如一个 ID，而不是整个数组项，你仍然可以使用`option`元素的`value`属性，前提是键属性是一个字符串值：

```js
<template> 
  <select value.bind="selectedCultureIsoCode"> 
    <option>Select your culture</option> 
    <option repeat.for="culture of cultures"  
            value.bind="culture.isoCode">${culture.name}</option> 
  </select> 
</template> 

```

在此示例中，选中的`option`的`value`绑定到相应项的`isoCode`属性，这是一个字符串，因此选中项的此属性将被分配给`selectedCultureIsoCode`。

当然，在渲染过程中，绑定到`select`属性的`value`表达式的值将被求值，如果任何`option`具有匹配的`value`或`model`属性，这个`option`将被渲染为选中状态。

### 多选

当`select`元素具有`multiple`属性时，绑定到其`value`属性的表达式预期是一个数组：

```js
<template> 
  <select value.bind="selectedCountries" multiple> 
    <option repeat.for="country of countries"  
            value.bind="country">${country}</option> 
  </select> 
</template> 

```

在此，选中的选项的值将被添加到`selectedCountries`数组中。

当用户选择一个项目时，选中的值总是添加到选择数组的末尾。

当然，当将非字符串值的数组渲染到多选列表时，也适用于相同的规则；数组的每个项目应绑定到其`option`的`model`属性上：

```js
<template> 
  <select value.bind="selectedCultures" multiple> 
    <option repeat.for="culture of cultures"  
            model.bind="culture">${culture.name}</option> 
  </select> 
</template> 

```

在这里，所选的`culture`对象将被添加到`selectedCultures`数组中。

使用键字符串属性的替代方案，在多选中同样适用：

```js
<template> 
  <select value.bind="selectedCulturesIsoCodes" multiple> 
    <option repeat.for="culture of cultures"  
            value.bind="culture.isoCode">${culture.name}</option> 
  </select> 
</template> 

```

在这个示例中，所选`culture`对象的`isoCode`属性将被添加到`selectedCulturesIsoCodes`数组中，这是一个字符串数组。

### 匹配器

当使用`model`属性时，可能会发生这种情况：分配给`select`的`value`属性的对象具有相同的身份，但与分配给`option`的`model`属性的对象不是同一个实例。在这种情况下，Aurelia 将无法渲染正确的`option`作为选中项。

`matcher`属性正是为这种场景设计的：

```js
<template> 
  <select value.bind="selectedCulture" matcher.bind="matchCulture"> 
    <option>Select your culture</option> 
    <option repeat.for="culture of cultures"  
            model.bind="culture">${culture.name}</option> 
  </select> 
</template> 

```

在这里，当尝试找出哪个`option`应该被选中时，`select`元素会将等价比较委托给`matchCulture`函数，该函数应大致如下所示：

```js
export class ViewModel { 
  matchCulture = (culture1, culture2) => culture1.isoCode === culture2.isoCode; 
} 

```

在这里，这个函数期望有两个文化对象，它们可能具有相同的身份，代表相同的文化。如果这两个对象具有相同的身份，将返回`true`，否则返回`false`。

## 输入元素

绑定到`input`元素在大多数情况下是很简单的，但实际上取决于`type`属性。例如，对于`text`输入，`value`属性默认是双向绑定的，可以用来获取用户的输入：

```js
<template> 
  <input type="text" value.bind="title"> 
</template> 

```

在这里，`title`属性的初始值将在`input`中显示，用户对`input`值的任何更改也将应用到`title`属性上。类似地，对`title`属性的任何更改也将应用到`input`的`value`上。

对于大多数其他类型的`input`，使用方式相同：`color`、`date`、`email`、`number`、`password`、`tel`或`url`等。然而，也有一些特殊情况，如下所述。

### 文件选择器

当`input`元素的`type`属性为`file`时，它暴露其`files`属性作为一个属性。它默认使用双向绑定：

```js
<template> 
  <input type="file" accepts="image/*" files.bind="images"> 
</template> 

```

在这个示例中，`input`元素的`files`属性被绑定到视图模型的`images`属性上。当用户选择一个文件时，`images`被赋予一个包含所选文件的`FileList`对象。如果`input`元素具有`multiple`属性，用户可以选择多个文件，结果的`FileList`对象将包含用户选择的多个`File`对象。

`FileList`和`File`类是 HTML5 文件 API 的一部分，可以与 Fetch API 一起使用，将用户选择的文件发送到服务器。在本书稍后的章节中，我们将看到在构建联系人应用程序的照片编辑组件时的一个示例。

Mozilla 开发者网络有关于文件 API 的详尽文档。关于`FileList`类的详细信息可以在[`developer.mozilla.org/en-US/docs/Web/API/FileList`](https://developer.mozilla.org/en-US/docs/Web/API/FileList)找到。

### 单选按钮

与`select`元素的`option`类似，单选按钮可以使用`value`或`model`属性来指定按钮选中时的值。`value`属性只期望字符串值，所以对于任何其他类型的值，必须使用`model`属性。

此外，单选按钮可以绑定它们的`checked`属性，该属性默认是双向的，到一个表达式，当选中时将被分配按钮的`value`或`model`。

```js
<template> 
  <label repeat.for="country of countries"> 
    <input type="radio" name="countries" value.bind="country"  
           checked.bind="selectedCountry"> 
    ${country} 
  </label> 
</template> 

```

在这里，一组单选按钮使用名为`countries`的字符串数组进行渲染。选中的单选按钮的`country`，绑定到`value`属性，将被分配给`selectedCountry`属性。

与`option`元素一样，当绑定到不是字符串的值时，应使用`model`属性而不是`value`：

```js
<template> 
  <label repeat.for="culture of cultures"> 
    <input type="radio" name="cultures" model.bind="culture"  
              checked.bind="selectedCulture"> 
    ${culture.name} 
  </label> 
</template> 

```

在这里，一组单选按钮使用一个`culture`对象的数组进行渲染。选中的单选按钮的`culture`，绑定到`model`属性，将被分配给`selectedCulture`属性。

与`select`元素类似，使用`model`属性的单选按钮也可以使用`matcher`属性来自定义等价比较逻辑。

所有之前的示例都使用了`repeat.for`绑定到数组来渲染动态的单选按钮列表。如果你需要渲染一个静态的单选按钮列表，并且期望的输出是一个布尔值，例如呢？在这种情况下，不需要在数组上迭代：

```js
<template> 
  <h4>Do you speak more than one language?</h4> 
  <label> 
    <input type="radio" name="isMultilingual" model.bind="null"  
           checked.bind="isMultilingual">  
    That's none of your business 
  </label> 
  <label> 
    <input type="radio" name="isMultilingual" model.bind="true"  
           checked.bind="isMultilingual"> 
    Yes 
  </label> 
  <label> 
    <input type="radio" name="isMultilingual" model.bind="false"  
           checked.bind="isMultilingual"> 
    No 
  </label> 
</template> 

```

在这个例子中，渲染了一个静态的单选按钮列表，每个按钮都使用它们的`model`属性绑定到不同的标量值。它们的`checked`属性绑定到`isMultilingual`属性，这将根据选择哪个按钮而被分配为`null`、`true`或`false`。

当然，在渲染过程中，如果绑定到按钮组`checked`属性的表达式有一个值与按钮的`value`或`model`属性匹配，这个按钮将被渲染为选中状态。

### 复选框

复选框列表在其典型用法上与带`multiple`属性的`select`元素相似。每个`input`元素都有`value`或`model`属性。此外，预期`checked`属性将被绑定到数组，到这个数组中将会添加所有选中的`input`的`value`或`model`：

```js
<template> 
  <label repeat.for="country of countries"> 
    <input type="checkbox" value.bind="country"  
           checked.bind="selectedCountries"> 
    ${country} 
  </label> 
</template> 

```

在这里，一组复选框使用名为`countries`的字符串数组进行渲染。选中的复选框的`country`，绑定到`value`属性，将被添加到`selectedCountries`数组。

与`option`元素或单选按钮一样，`value`属性只期望字符串值。当绑定到任何其他类型的值时，应使用`model`属性：

```js
<template> 
  <label> 
    <input type="checkbox" model.bind="culture"  
           checked.bind="selectedCultures"> 
    ${culture.name} 
  </label> 
</template> 

```

在此，一组复选框使用`culture`对象的数组进行渲染。选中的复选框的`culture`，通过`model`属性绑定，将被添加到`selectedCultures`数组中。

与`select`元素和单选按钮类似，使用`model`属性的复选框也可以使用`matcher`属性来自定义等价比较逻辑。

当然，如果渲染对象数组时，选中的值是某种字符串 ID，仍然可以使用`value`属性：

```js
<template> 
  <label> 
    <input type="checkbox" value.bind="culture.isoCode"  
           checked.bind="selectedCulturesIsoCodes"> 
    ${culture.name} 
  </label> 
</template> 

```

在此，一组复选框使用`culture`对象的数组进行渲染。选中的复选框的`culture`的`isoCode`属性，绑定到`value`属性，将被添加到`selectedCulturesIsoCodes`字符串数组中。

当然，在渲染过程中，如果绑定到`checked`属性的数组包含绑定到`value`或`model`属性的值，此复选框将被渲染为选中状态。

Alternatively，复选框可以绑定到不同的布尔表达式，而不是一个单一的数组。这可以通过省略任何`value`或`model`属性来实现：

```js
<template> 
  <label> 
    <input type="checkbox" checked.bind="speaksFrench">French 
  </label> 
  <label> 
    <input type="checkbox" checked.bind="speaksEnglish">English 
  </label> 
  <label> 
    <input type="checkbox" checked.bind="speaksGerman">German 
  </label> 
</template> 

```

在此示例中，每个`checkbox`绑定到不同的属性，这将根据复选框是否被选中分配`true`或`false`。

## `textarea`

绑定到`textarea`元素与绑定到`text``input`元素相同：

```js
<template> 
  <textarea value.bind="text"></textarea> 
</template> 

```

在此，`text`属性的初始值将在`textarea`内显示，由于`textarea`的`value`属性的绑定是默认的双向的，用户对`textarea`内容的所有修改都将反映在`text`属性上。

## 禁用元素

禁用`input`、`select`、`textarea`或`button`元素只需绑定到其`disabled`属性即可：

```js
<template> 
  <input type="text" disabled.bind="isSending"> 
  <button disabled.bind="isSending">Send</button> 
</template> 

```

当`isSending`为`true`时，`input`和`button`元素都将被禁用。

## 设置元素只读

同样，使`input`或`textarea`元素只读只需将其`readonly`属性绑定即可：

```js
<template> 
  <input type="text" readonly.bind="!canEdit"> 
</template> 

```

在此，当`canEdit`为`false`时，`input`将变为只读。

# 向我们的应用程序添加表单

既然我们知道如何处理用户输入元素，我们可以在我们的联系人管理应用程序中添加表单以创建和编辑联系人。

## 添加新路由

我们需要添加三个新路由：一个用于创建新联系人，另一个用于编辑现有联系人，最后一个用于上传联系人的照片。让我们在根组件中添加它们：

`src/app.js`文件将如下所示：

```js
export class App { 
  configureRouter(config, router) { 
    this.router = router; 
    config.title = 'Learning Aurelia'; 
    config.map([ 
      { route: '', redirect: 'contacts' }, 
      { route: 'contacts', name: 'contacts', moduleId:        'contact-list',  
        nav: true, title: 'Contacts' }, 
      { route: 'contacts/new', name: 'contact-creation',  
        moduleId: 'contact-edition', title: 'New contact' }, 
      { route: 'contacts/:id', name: 'contact-details',  
        moduleId: 'contact-details' }, 
      { route: 'contacts/:id/edit', name: 'contact-edition',  
        moduleId: 'contact-edition' }, 
      { route: 'contacts/:id/photo', name: 'contact-photo', 
        moduleId: 'contact-photo' }, 
    ]); 
    config.mapUnknownRoutes('not-found'); 
  } 
} 

```

在前面的代码片段中，三个新路由被突出显示。

在这里，定位很重要。`contact-creation`路径在`contact-details`路径之前，这是因为它们的`route`属性。在尝试查找 URL 更改时的匹配路径时，路由器会按照它们被定义的顺序深入路由定义。由于`contact-details`的模式匹配任何以`contacts/`开头，后跟第二个部分作为参数的解释，因此路径`contacts/new`符合此模式，所以如果`contact-creation`路径定义在后面，它将无法到达，而`contact-details`路径将使用等于`new`的`id`参数到达。

依赖于路由顺序的更好替代方法是将模式更改，以避免可能的冲突。例如，我们可以将`contact-details`的模式更改为类似于`contacts/:id/details`。在这种情况下，路由的顺序将不再重要。

您可能已经注意到两个新路径具有相同的`moduleId`。这是因为我们将为创建新联系人和编辑现有联系人使用相同的组件。

## 添加新路径的链接

下一步将是添加刚刚添加的路由的链接。我们首先在`contact-list`组件中添加一个到`contact-creation`路径的链接：

`src/contact-list.html`

```js
 <template> 
  <section class="container"> 
    <h1>Contacts</h1> 

    <div class="row"> 
      <div class="col-sm-1"> 
        <a route-href="route: contact-creation" class= "btn btn-primary"> 
          <i class="fa fa-plus-square-o"></i> New 
        </a> 
      </div> 
      <div class="col-sm-2"> 
        <!-- Search box omitted for brevity --> 
      </div> 
    </div> 
    <!--  Contact list omitted for brevity --> 
  </section> 
</template> 

```

在这里，我们添加了一个`a`元素，并利用`route-href`属性渲染`contact-creation`路径的 URL。

我们还需要添加到`contact-photo`和`contact-edition`路径的链接。我们将在`contact-details`组件中完成这个任务：

`src/contact-details.html`

```js
 <template> 
  <section class="container"> 
    <div class="row"> 
      <div class="col-sm-2"> 
        <a route-href="route: contact-photo; params.bind:
          { id: contact.id }"  
           > 
          <img src.bind="contact.photoUrl" class= "img-responsive" alt="Photo"> 
        </a> 
      </div> 
      <div class="col-sm-10"> 
        <template if.bind="contact.isPerson"> 
          <h1>${contact.fullName}</h1> 
          <h2>${contact.company}</h2> 
        </template> 
        <template if.bind="!contact.isPerson"> 
          <h1>${contact.company}</h1> 
        </template> 
        <a class="btn btn-default" route-href="route:
          contact-edition;  
          params.bind: { id: contact.id }"> 
          <i class="fa fa-pencil-square-o"></i> Modify 
        </a> 
      </div> 
    </div> 
    <!-- Rest of template omitted for brevity --> 
  </section> 
</template> 

```

在这里，我们首先重构显示`fullName`和`company`（如果联系人是人）的模板，通过添加一个外部的`div`并将`col-sm-10`CSS 类从标题移动到这个`div`。

接下来，我们将显示联系人性照的`img`元素包裹在一个导航到`contact-photo`路径的锚点中，使用联系人的`id`作为参数。

最后，我们添加另一个指向`contact-edition`路径的锚点，使用联系人的`id`作为参数。

## 更新模型

为了重用代码，我们将坚持使用`Contact`类，并在我们的表单组件中使用它。我们还将为电话号码、电子邮件地址、地址和社会资料创建类，这样我们的`contact-edition`组件就无需知道创建这些对象空实例的详细信息。

我们需要添加创建我们模型空实例的能力，并将其所有属性初始化为适当的默认值。因此，我们将为我们的模型类添加所有属性的默认值。

最后，我们需要更新`Contact`的`fromObject`工厂方法，以便所有列表项都正确映射到我们的模型类实例。

`src/models.js`

```js
export class PhoneNumber { 
  static fromObject(src) { 
    return Object.assign(new PhoneNumber(), src); 
  } 

  type = 'Home'; 
  number = ''; 
} 

export class EmailAddress { 
  static fromObject(src) { 
    return Object.assign(new EmailAddress(), src); 
  } 

  type = 'Home'; 
  address = ''; 
} 

export class Address { 
  static fromObject(src) { 
    return Object.assign(new Address(), src); 
  } 

  type = 'Home'; 
  number = ''; 
  street = ''; 
  postalCode = ''; 
  city = ''; 
  state = ''; 
  country = ''; 
} 

export class SocialProfile { 
  static fromObject(src) { 
    return Object.assign(new SocialProfile(), src); 
  } 

  type = 'GitHub'; 
  username = ''; 
} 

export class Contact { 
  static fromObject(src) { 
    const contact = Object.assign(new Contact(), src); 
    contact.phoneNumbers = contact.phoneNumbers 
      .map(PhoneNumber.fromObject); 
    contact.emailAddresses = contact.emailAddresses 
      .map(EmailAddress.fromObject); 
    contact.addresses = contact.addresses 
      .map(Address.fromObject); 
    contact.socialProfiles = contact.socialProfiles 
      .map(SocialProfile.fromObject); 
    return contact; 
  } 

  firstName = ''; 
  lastName = ''; 
  company = ''; 
  birthday = ''; 
  phoneNumbers = []; 
  emailAddresses = []; 
  addresses = []; 
  socialProfiles = []; 
  note = ''; 

  // Omitted snippet... 
} 

```

在这里，我们首先添加了`PhoneNumber`、`EmailAddress`、`Address`和`SocialProfile`类的类。每个类都有一个静态的`fromObject`工厂方法，其属性都使用默认值正确初始化。

接下来，我们添加了一个`Contact`对象属性，其初始值为默认值，并更改了其`fromObject`工厂方法，以便列表项能够正确映射到它们相应的类中。

## 创建表单组件

现在我们可以创建我们新的`contact-edition`组件了。如早前所提，这个组件将用于创建和编辑。它能够通过检查在其`activate`回调方法中是否接收了一个`id`参数来检测它是用于创建新的联系人还是编辑现有的联系人。确实，`contact-creation`路由的模式定义了无参数，所以当我们的表单组件通过这个路由被激活时，它不会接收任何`id`参数。另一方面，由于`contact-edition`路由的模式确实定义了一个`id`参数，所以当我们的表单组件通过这个路由被激活时，它会接收到这个参数。

我们可以这样做，因为在我们联系管理应用程序的范围内，创建和编辑过程几乎是一致的。然而，在许多情况下，最好是为创建和编辑分别设计单独的组件。

### 激活视图模型

让我们首先从视图模型和`activate`回调方法开始：

`src/contact-edition.js`

```js
import {inject} from 'aurelia-framework'; 
import {ContactGateway} from './contact-gateway'; 
import {Contact} from './models'; 

@inject(ContactGateway) 
export class ContactEdition { 
  constructor(contactGateway) { 
    this.contactGateway = contactGateway; 
  } 

  activate(params, config) { 
    this.isNew = params.id === undefined; 
    if (this.isNew) { 
      this.contact = new Contact(); 
    } 
    else { 
      return this.contactGateway.getById(params.id).then(contact => { 
        this.contact = contact; 
        config.navModel.setTitle(contact.fullName); 
      }); 
    } 
  } 
} 

```

在这里，我们首先向我们的视图模型注入`ContactGateway`类的实例。然后，在`activate`回调方法中，我们首先定义了一个`isNew`属性，该属性基于是否存在`id`参数。这个属性将用于我们的组件，使其知道它是被用来创建一个新的联系人还是编辑一个现有的联系人。

接下来，基于这个`isNew`属性，我们初始化组件。如果我们正在创建一个新的联系人，那么我们只需创建一个`contact`属性并将其分配给一个新的、空的`Contact`实例；否则，我们使用`ContactGateway`根据`id`参数检索适当的联系人，当`Promise`解决时，将`Contact`实例分配给`contact`属性，并将文档标题设置为联系人的`fullName`属性。

一旦激活周期完成，视图模型将有一个适当初始化为`Contact`对象的`contact`属性和一个指示联系人是新的还是现有的`isNew`属性。

### 构建表单布局

接下来，让我们构建一个用于显示表单的模板。由于这个模板相当大，我将把它分成几部分，这样你就可以逐步构建它并在每个步骤进行测试（如果需要的话）。

模板由一个头部组成，后面是一个`form`元素，它将包含模板的其余部分：

`src/contact-edition.html`

```js
 <template> 
  <section class="container"> 
    <h1 if.bind="isNew">New contact</h1> 
    <h1 if.bind="!isNew">Contact #${contact.id}</h1> 

    <form class="form-horizontal"> 
      <!-- The rest of the template goes in here --> 
    </form> 
  </section> 
</template> 

```

在头部，我们使用`isNew`属性来显示是告诉用户他正在创建一个新的联系人还是显示正在编辑的联系人`id`的动态标题。

### 编辑标量属性

接下来，我们将向`form`元素中添加块，其中包含输入元素，以编辑联系人的`firstName`、`lastName`、`company`、`birthday`和`note`，如前一个代码片段中定义的那样：

```js
<div class="form-group"> 
  <label class="col-sm-3 control-label">First name</label> 
  <div class="col-sm-9"> 
    <input type="text" class="form-control" value.bind="contact.firstName"> 
  </div> 
</div> 

<div class="form-group"> 
  <label class="col-sm-3 control-label">Last name</label> 
  <div class="col-sm-9"> 
    <input type="text" class="form-control" value.bind="contact.lastName"> 
  </div> 
</div> 

<div class="form-group"> 
  <label class="col-sm-3 control-label">Company</label> 
  <div class="col-sm-9"> 
    <input type="text" class="form-control" value.bind="contact.company"> 
  </div> 
</div> 

<div class="form-group"> 
  <label class="col-sm-3 control-label">Birthday</label> 
  <div class="col-sm-9"> 
    <input type="date" class="form-control" value.bind="contact.birthday"> 
  </div> 
</div> 

<div class="form-group"> 
  <label class="col-sm-3 control-label">Note</label> 
  <div class="col-sm-9"> 
    <textarea class="form-control" value.bind="contact.note"></textarea> 
  </div> 
</div> 

```

在这里，我们仅为每个属性定义一个`form-group`以进行编辑。前三个属性各自绑定到一个`text input`元素。此外，`birthday`属性绑定到一个`date`输入，使其更容易编辑日期——当然，仅限支持它的浏览器，而`note`属性则绑定到一个`textarea`元素。

### 编辑电话号码

在此之后，我们需要为列表添加编辑器。由于每个列表中包含的数据并不复杂，我们将使用内联编辑器，这样用户就可以在最少的点击次数内直接编辑任何项目的任何字段。

我们将在本章后面讨论更复杂的编辑模型，使用对话框。

让我们从电话号码开始：

```js
<hr> 
<div class="form-group" repeat.for="phoneNumber of contact.phoneNumbers"> 
  <div class="col-sm-2 col-sm-offset-1"> 
    <select value.bind="phoneNumber.type" class="form-control"> 
      <option value="Home">Home</option> 
      <option value="Office">Office</option> 
      <option value="Mobile">Mobile</option> 
      <option value="Other">Other</option> 
    </select> 
  </div> 
  <div class="col-sm-8"> 
    <input type="tel" class="form-control" placeholder="Phone number"  
           value.bind="phoneNumber.number"> 
  </div> 
  <div class="col-sm-1"> 
    <button type="button" class="btn btn-danger"  
            click.delegate="contact.phoneNumbers.splice($index, 1)"> 
      <i class="fa fa-times"></i>  
    </button> 
  </div> 
</div> 
<div class="form-group"> 
  <div class="col-sm-9 col-sm-offset-3"> 
    <button type="button" class="btn btn-default" click.delegate="contact.addPhoneNumber()"> 
      <i class="fa fa-plus-square-o"></i> Add a phone number 
    </button> 
  </div> 
</div> 

```

这个电话号码列表编辑器可以分解为几个部分，其中最重要的是突出显示的。首先，为联系的`phoneNumbers`数组中的每个`phoneNumber`重复一个`form-group`。

对于每个`phoneNumber`，我们定义一个`select`元素，其`value`绑定到`phoneNumber`的`type`属性，以及一个`tel`输入，其`value`绑定到`phoneNumber`的`number`属性。此外，我们定义了一个`button`，当点击时，使用当前的`$index`（正如您可能记得的前一章中提到的，这是通过`repeat`属性添加到绑定上下文的），从`contact`的`phoneNumbers`数组中拼接出电话号码。

最后，在电话号码列表之后，我们定义了一个`button`，其`click`事件调用`contact`中的`addPhoneNumber`方法。

### 添加缺失的方法

我们在上一个模板中添加的按钮之一调用了一个尚未定义的方法。让我们把这个方法添加到`Contact`类中：

`src/models.js`

```js
//Snippet... 
export class Contact { 
  //Snippet... 
  addPhoneNumber() { 
    this.phoneNumbers.push(new PhoneNumber()); 
  } 
} 

```

此代码片段中的第一个方法用于向列表中添加一个空电话号码，简单地在`phoneNumbers`数组中推入一个新的`PhoneNumber`实例。

### 编辑其他列表

其他列表的模板，如电子邮件地址、地址和社会资料，都非常相似。只有正在编辑的字段会改变，但主要概念——重复的表单组、每个条目都有一个删除按钮和一个在列表末尾的添加按钮——是相同的。

让我们从`emailAddresses`开始：

```js
<hr> 
<div class="form-group" repeat.for="emailAddress of contact.emailAddresses"> 
  <div class="col-sm-2 col-sm-offset-1"> 
    <select value.bind="emailAddress.type" class="form-control"> 
      <option value="Home">Home</option> 
      <option value="Office">Office</option> 
      <option value="Other">Other</option> 
    </select> 
  </div> 
  <div class="col-sm-8"> 
    <input type="email" class="form-control" placeholder="Email address"  
           value.bind="emailAddress.address"> 
  </div> 
  <div class="col-sm-1"> 
    <button type="button" class="btn btn-danger"  
            click.delegate="contact.emailAddresses.splice($index, 1)"> 
      <i class="fa fa-times"></i>  
    </button> 
  </div> 
</div> 
<div class="form-group"> 
  <div class="col-sm-9 col-sm-offset-3"> 
    <button type="button" class="btn btn-primary"  
            click.delegate="contact.addEmailAddress()"> 
      <i class="fa fa-plus-square-o"></i> Add an email address 
    </button> 
  </div> 
</div> 

```

这个模板与电话号码的模板非常相似。主要区别在于可用的类型并不完全相同，而且`input`的`type`是`email`。

如您所想象，地址的编辑器会更大一些：

```js
<hr> 
<div class="form-group" repeat.for="address of contact.addresses"> 
  <div class="col-sm-2 col-sm-offset-1"> 
    <select value.bind="address.type" class="form-control"> 
      <option value="Home">Home</option> 
      <option value="Office">Office</option> 
      <option value="Other">Other</option> 
    </select> 
  </div> 
  <div class="col-sm-8"> 
    <div class="row"> 
      <div class="col-sm-4"> 
        <input type="text" class="form-control" placeholder="Number"  
               value.bind="address.number"> 
      </div> 
      <div class="col-sm-8"> 
        <input type="text" class="form-control" placeholder="Street"  
               value.bind="address.street"> 
      </div> 
    </div> 
    <div class="row"> 
      <div class="col-sm-4"> 
        <input type="text" class="form-control" placeholder="Postal code"  
               value.bind="address.postalCode"> 
      </div> 
      <div class="col-sm-8"> 
        <input type="text" class="form-control" placeholder="City"  
               value.bind="address.city"> 
      </div> 
    </div> 
    <div class="row"> 
      <div class="col-sm-4"> 
        <input type="text" class="form-control" placeholder="State"  
               value.bind="address.state"> 
      </div> 
      <div class="col-sm-8"> 
        <input type="text" class="form-control" placeholder="Country"  
               value.bind="address.country"> 
      </div> 
    </div> 
  </div> 
  <div class="col-sm-1"> 
    <button type="button" class="btn btn-danger"  
            click.delegate="contact.addresses.splice($index, 1)"> 
      <i class="fa fa-times"></i>  
    </button> 
  </div> 
</div> 
<div class="form-group"> 
  <div class="col-sm-9 col-sm-offset-3"> 
    <button type="button" class="btn btn-primary"  
            click.delegate="contact.addAddress()"> 
      <i class="fa fa-plus-square-o"></i> Add an address 
    </button> 
  </div> 
</div> 

```

在这里，左侧包含六个不同的输入，允许我们编辑地址的各种文本属性。

至此，您可能已经对社交资料的模板有一个大致的了解：

```js
<hr> 
<div class="form-group" repeat.for="profile of contact.socialProfiles"> 
  <div class="col-sm-2 col-sm-offset-1"> 
    <select value.bind="profile.type" class="form-control"> 
      <option value="GitHub">GitHub</option> 
      <option value="Twitter">Twitter</option> 
    </select> 
  </div> 
  <div class="col-sm-8"> 
    <input type="text" class="form-control" placeholder="Username"  
           value.bind="profile.username"> 
  </div> 
  <div class="col-sm-1"> 
    <button type="button" class="btn btn-danger"  
            click.delegate="contact.socialProfiles.splice($index, 1)"> 
      <i class="fa fa-times"></i>  
    </button> 
  </div> 
</div> 
<div class="form-group"> 
  <div class="col-sm-9 col-sm-offset-3"> 
    <button type="button" class="btn btn-primary"  
            click.delegate="contact.addSocialProfile()"> 
      <i class="fa fa-plus-square-o"></i> Add a social profile 
    </button> 
  </div> 
</div> 

```

当然，每个列表添加项目的方法都需要添加到`Contact`类中：

`src/models.js`

```js
//Omitted snippet... 
export class Contact { 
  //Omitted snippet... 
  addEmailAddress() { 
    this.emailAddresses.push(new EmailAddress()); 
  } 

  addAddress() { 
    this. addresses.push(new Address()); 
  } 

  addSocialProfile() { 
    this.socialProfiles.push(new SocialProfile()); 
  } 
} 

```

正如你所看到的，这些方法与我们之前为电话号码编写的那些几乎完全相同。此外，每个列表的模板片段也基本上彼此相同。所有这种冗余都呼吁进行重构。我们将在第五章，*制作可复用的组件*中看到，如何将常见行为和模板片段提取到一个组件中，我们将重新使用它来管理每个列表。

### 保存和取消

我们表单（至少在视觉上）完整的最后一件缺失的事情是在包含`form`元素的末尾添加一个保存和取消按钮：

```js
//Omitted snippet... 
<form class="form-horizontal" submit.delegate="save()"> 
  //Omitted snippet... 
  <div class="form-group"> 
      <div class="col-sm-9 col-sm-offset-3"> 
        <button type="submit" class="btn btn-success">Save</button> 
        <a if.bind="isNew" class="btn btn-danger"  
           route-href="route: contacts">Cancel</a> 
        <a if.bind="!isNew" class="btn btn-danger"  
           route-href="route: contact-details;  
           params.bind: { id: contact.id }">Cancel</a> 
      </div> 
    </div> 
</form> 

```

首先，我们将一个对`save`方法的调用绑定到`form`元素的`submit`事件，然后我们添加了一个包含一个名为`Save`的`submit`按钮的最后一个`form-group`。

接下来，我们添加了两个`Cancel`链接：一个在创建新联系人时显示，用于导航回到联系人列表；另一个在编辑现有联系人时显示，用于导航回到联系人的详细信息。

我们还需要将`save`方法添加到视图模型中。这个方法最终将委派给`ContactGateway`，但为了测试我们到目前为止所做的一切是否工作，让我们只写一个方法版本：

```js
save() { 
  alert(JSON.stringify(this.contact)); 
} 

```

至此，你应该能够运行应用程序并尝试创建或编辑一个联系人。点击**保存**按钮时，你应该会看到一个显示联系人的警报，该联系人作为 JSON 序列化格式。

## 使用 fetch 发送数据

我们现在可以向`ContactGateway`类添加创建和更新联系人的方法：

`src/contact-gateway.js`

```js
//Omitted snippet... 
import {HttpClient, json} from 'aurelia-fetch-client'; 
//Omitted snippet... 
export class ContactGateway { 
  //Omitted snippet... 
  create(contact) { 
    return this.httpClient.fetch('contacts',  
      { method: 'POST', body: json(contact) }); 
  } 

  update(id, contact) { 
    return this.httpClient.fetch(`contacts/${id}`,  
      { method: 'PUT', body: json(contact) }); 
  } 
} 

```

首先要做的第一件事是`import`从`fetch-client`中`json`函数。这个函数接受任何 JS 值作为参数，并返回一个包含接收参数序列化为 JSON 的`Blob`对象。

接下来，我们添加了一个`create`方法，它接受一个`contact`作为参数，并调用 HTTP 客户端的`fetch`方法，传递要调用的相对 URL，然后是一个配置对象。这个对象包含将分配给底层`Request`对象的属性。在这里，我们指定一个`method`属性，告诉客户端执行一个`POST`请求，我们指示请求的`body`将是序列化为 JSON 的`contact`。最后，`fetch`方法返回一个`Promise`，这是我们新`create`方法返回的，所以调用者可以在请求完成后做出反应。

`update`方法非常相似。第一个区别是参数：首先期望联系人的`id`，然后是`contact`对象本身。其次，`fetch`调用略有不同；它发送一个到不同 URL 的请求，使用`PUT`方法，但其主体相同。

一个 Fetch`Request`的`body`预期是一个`Blob`、一个`BufferSource`、一个`FormData`、一个`URLSearchParams`或一个`USVString`对象。关于这方面的文档可以在 Mozilla 开发者网络上找到，网址为[`developer.mozilla.org/en-US/docs/Web/API/Request/Request`](https://developer.mozilla.org/en-US/docs/Web/API/Request/Request)。

为了测试我们新方法是否有效，让我们将`contact-edition`组件的视图模型中的模拟`save`方法替换为真实的方法：

```js
//Omitted snippet... 
import {Router} from 'aurelia-router'; 

@inject(ContactGateway, Router) 
export class ContactEdition { 
  constructor(contactGateway, router) { 
    this.contactGateway = contactGateway; 
    this.router = router; 
  } 

  // Omitted snippet... 
  save() { 
    if (this.isNew) { 
      this.contactGateway.create(this.contact)  
        .then(() => this.router.navigateToRoute('contacts')); 
    } 
    else { 
      this.contactGateway.update(this.contact.id, this.contact)  
        .then(() => this.router.navigateToRoute('contact-details',  
                    { id: this.contact.id })); 
    } 
  } 
} 

```

在这里，我们首先导入`Router`，并在视图模型中注入它的一个实例。接下来，我们改变`save`方法的主体：如果组件正在创建一个新的联系人，我们首先调用`ContactGateway`的`create`方法，将`contact`对象传递给它，然后在`Promise`解决时返回至`contacts`路由；否则，当组件正在编辑一个现有的联系人时，我们首先调用`ContactGateway`的`update`方法，将联系人的`id`和`contact`对象传递给它，然后在`Promise`解决时返回至该联系人的详情路由。

此时，你应该能够创建或更新一个联系人。然而，一些创建或更新的请求可能会返回状态码为 400 的响应，表示“坏的请求”。不必惊慌；因为 HTTP 端点会执行一些验证，而我们的表单目前不会，所以这种情况是预料之中的，例如，如果你留下了一些字段是空的。我们将在本章后面为我们的表单添加验证，这将防止这类错误的发生。

## 上传联系人的照片

既然我们能够创建和编辑联系人，现在让我们添加一个组件来上传其照片。这个组件将被命名为`contact-photo`，并通过我们已经在本章早些时候添加的具有相同名称的路由来激活。

这个组件将使用一个`file input`元素让用户从他的文件系统中选择一个图片文件，并将利用 HTML5 文件 API 以及 Fetch 客户端将选定的图片文件发送到我们的 HTTP 端点。

### 构建模板

这个组件的模板简单地重用了我们已经在前面讲解过的几个概念：

`src/contact-photo.html`

```js
 <template> 
  <section class="container"> 
    <h1>${contact.fullName}</h1> 

    <form class="form-horizontal" submit.delegate="save()"> 
      <div class="form-group"> 
        <label class="col-sm-3 control-label" for="photo">Photo</label> 
        <div class="col-sm-9"> 
          <input type="file" id="photo" accept="image/*"  
                 files.bind="photo"> 
        </div> 
      </div> 

      <div class="form-group"> 
        <div class="col-sm-9 col-sm-offset-3"> 
          <button type="submit" class="btn btn-success">Save</button> 
          <a class="btn btn-danger" route-href="route: contact-details;  
             params.bind: { id: contact.id }">Cancel</a> 
        </div> 
      </div> 
    </form> 
  </section> 
</template> 

```

在这里，我们首先将联系人的`fullName`作为页面标题显示出来。然后，在一个`form`元素中，其`submit`事件会触发一个`save`方法，我们添加了一个`file input`和两个按钮，用于**保存**或**取消**上传照片。`file input`有一个`accept`属性，迫使浏览器的文件选择对话框只显示图片文件，并且它的`files`属性被绑定到`photo`属性。

### 创建视图模型

视图模型与`contact-edition`视图模型非常相似，至少在比较导入、构造函数和`activate`方法时是这样的：

`src/contact-photo.js`

```js
import {inject} from 'aurelia-framework'; 
import {Router} from 'aurelia-router'; 
import {ContactGateway} from './contact-gateway'; 

@inject(ContactGateway, Router) 
export class ContactPhoto { 

  constructor(contactGateway, router) { 
    this.contactGateway = contactGateway; 
    this.router = router; 
  } 

  activate(params, config) { 
    return this.contactGateway.getById(params.id).then(contact => { 
      this.contact = contact; 
      config.navModel.setTitle(this.contact.fullName); 
    }); 
  } 
  save() { 
    if (this.photo && this.photo.length > 0) { 
      this.contactGateway.updatePhoto( 
        this.contact.id,  
        this.photo.item(0) 
      ).then(() => { 
        this.router.navigateToRoute( 
          'contact-details',  
          { id: this.contact.id }); 
      }); 
    } 
  } 
} 

```

这个视图模型期望在其构造函数中注入`ContactGateway`的一个实例和`Router`的一个实例。在其`activate`方法中，它然后使用其`id`参数加载一个`Contact`实例，并使用`contact`的`fullName`初始化文档标题。这与`contact-edition`视图模型非常相似。

`save`方法有一点不同。它首先检查是否已经选择了文件；如果没有，现在什么也不做。否则，它调用`ContactGateway`的`updatePhoto`方法，将联系人的`id`和选定的文件传递给它，并在`Promise`解决时返回到联系人的详细信息。

### 使用 fetch 上传文件

使我们的照片上传功能正常工作的最后一步是在`ContactGateway`类中的`uploadPhoto`方法：

`src/contact-gateway.js`

```js
//Omitted snippet... 
export class ContactGateway { 
  //Omitted snippet... 
  updatePhoto(id, file) { 
    return this.httpClient.fetch(`contacts/${id}/photo`, {  
      method: 'PUT', 
      headers: { 'Content-Type': file.type }, 
      body: file 
    }); 
  } 
} 

```

我们 HTTP 后端的`contacts/{id}/photo`端点期望一个 PUT 请求，带有正确的`Content-Type`头和图像二进制作为其主体。这正是这里`fetch`调用的作用：它使用`file`参数，这被期望是一个`File`类的实例，并使用它的`type`属性设置`Content-Type`头，然后将`file`本身作为请求体发送。

如早先所述，`File`类是 HTML5 文件 API 的一部分。Mozilla 开发者网络提供了关于这个 API 的详尽文档。关于`File`类的细节可以在[`developer.mozilla.org/en-US/docs/Web/API/File`](https://developer.mozilla.org/en-US/docs/Web/API/File)找到。

像往常一样，`updatePhoto`方法返回由 HTTP 请求解决的`Promise`，所以调用者可以在操作完成时采取行动。

至此，你应该能够运行应用程序并通过上传新图像文件来更新联系人的照片。

## 删除联系人

至此，我们的应用程序允许我们创建、读取和更新联系人。显然，**创建、读取、更新、删除**（**CRUD**）这四个字母中有一个缺失了：我们还不能删除一个联系人。让我们快速实现这个功能。

首先，让我们在联系人的`details`组件中添加一个**删除**按钮：

`src/contact-details.html`

```js
 <template> 
  <section class="container"> 
    <div class="row"> 
      <div class="col-sm-2"> 
        <!-- Omitted snippet... --> 
      </div> 
      <div class="col-sm-10"> 
        <!-- Omitted snippet... --> 
        <a class="btn btn-default" route-href="route: contact-edition;  
          params.bind: { id: contact.id }"> 
          <i class="fa fa-pencil-square-o"></i> Modify 
        </a> 
        <button class="btn btn-danger" click.delegate="tryDelete()"> 
          <i class="fa fa-trash-o"></i> Delete 
        </button> 
      </div> 
    </div> 
    <!-- Rest of template omitted for brevity --> 
  </section> 
</template> 

```

新的**删除**按钮将在点击时调用`tryDelete`方法：

`src/contact-details.js`

```js
//Omitted snippet... 
export class ContactDetails { 
  //Omitted snippet... 
  tryDelete() { 
    if (confirm('Do you want to delete this contact?')) { 
      this.contactGateway.delete(this.contact.id) 
        .then(() => { this.router.navigateToRoute('contacts'); }); 
    } 
  } 
} 

```

`tryDelete`方法首先要求用户进行**确认**删除，然后使用联系人的`id`调用网关的`delete`方法。当返回的`Promise`解决时，它返回到联系人列表。

最后，`ContactGateway`类的`delete`方法只是执行一个到后端适当路径的 Fetch 调用，使用`DELETE`HTTP 方法：

`src/contact-gateway.js`

```js
//Omitted snippet... 
export class ContactGateway { 
  //Omitted snippet... 
  delete(id) { 
    return this.httpClient.fetch(`contacts/${id}`, { method: 'DELETE' }); 
  } 
} 

```

至此，如果你点击一个联系人的**删除**按钮并批准确认对话框，你应该会被重定向到联系人列表，并且联系人应该消失了。

# 验证

如果你尝试保存一个生日无效、电话号码为空、地址、电子邮件或社交资料用户名为空的联系人，而你的浏览器的调试控制台是打开的，你会看到 HTTP 端点用 400 Bad Request 响应拒绝这个请求。这是因为后端在对创建或更新的联系人执行一些验证。

拥有一个执行某种验证的远程服务是很常见的；相反的，实际上被认为是糟糕的架构，因为远程服务不应该信任其客户端的有效数据。然而，为了提供更佳的用户体验，通常也会看到客户端应用程序也执行验证。

Aurelia 提供了`aurelia-validation`库，该库为验证提供者定义了一个接口，以及将验证插入组件的各种机制。它还提供了这个接口的默认实现，提供了一个简单而强大的验证机制。

让我们看看我们如何使用这些库来验证我们的联系人表单。

这一节只是对`aurelia-validation`提供的最常见特性的概述。实际上，这个库比这里描述的要灵活得多，功能也更强大，所以我在阅读这本书后邀请你进一步挖掘它。

## 安装库

要安装这个库，你只需要在项目的目录下运行以下命令：

```js
> npm install aurelia-validation --save

```

接下来，我们需要使这个库在应用程序的包中可用。在`aurelia_project/aurelia.json`中，在`build`下的`bundles`中，在名为`vendor-bundle.js`的包的`dependencies`数组中，添加以下条目：

```js
{ 
  "name": "aurelia-validation", 
  "path": "../node_modules/aurelia-validation/dist/amd", 
  "main": "aurelia-validation" 
}, 

```

这个配置项将告诉 Aurelia 的打包器将新安装的库包含在供应商包中。

## 配置

`aurelia-validation`库在使用前需要一些配置。此外，作为一个 Aurelia 插件，它需要在我们的应用程序启动时加载。

我们可以在我们主要的`configure`函数里完成这一切。然而，这种情况真的是一个很好的 Aurelia 特性的候选。如果你记得的话，特性类似于插件，只不过它们是在应用程序本身内定义的。通过引入一个`validation`特性，我们可以隔离验证的配置，这会给我们一个可以放置额外服务和自定义验证规则的中央位置。

让我们先创建我们的`validation`特性：

`src/validation/index.js`

```js
export function configure(config) { 
  config 
    .plugin('aurelia-validation'); 
} 

```

我们新特性的`configure`函数只是加载了`aurelia-validation`插件。

接下来，我们需要在我们主要的`configure`函数中加载这个特性：

```js
src/main.js 
//Omitted snippet... 
export function configure(aurelia) { 
  aurelia.use 
    .standardConfiguration() 
    .feature('validation') 
    .feature('resources'); 
  //Omitted snippet... 
} 

```

在这里，我们只是链接了引导式 API 的`feature`方法的额外调用，以加载我们的`validation`特性。

## 验证联系人表单

既然一切配置都正确，那就让我们在我们的`contact-edition`表单中添加验证吧。

### 设置模板

为了告诉验证机制需要验证什么，所有用于获取待验证用户输入的双向绑定都必须用`validate`绑定行为装饰，这由`aurelia-validation`提供：

`src/contact-edition.html`

```js
 <template> 
  <!-- Omitted snippet... -->   
  <input type="text" class="form-control"  
         value.bind="contact.firstName & validate"> 
  <!-- Omitted snippet... --> 
  <input type="text" class="form-control"  
         value.bind="contact.birthday & validate"> 
  <!-- Omitted snippet... --> 
  <textarea class="form-control"  
            value.bind="contact.note & validate"></textarea> 
  <!-- Omitted snippet... --> 
  <select value.bind="phoneNumber.type & validate" class="form-control"> 
  <!-- Omitted snippet... --> 
  <input type="tel" class="form-control" placeholder="Phone number"  
         value.bind="phoneNumber.number & validate"> 
  <!-- Omitted snippet... --> 
</template> 

```

在这里，我们在每个双向绑定中添加了`validate`绑定行为。代码片段没有展示`contact-edition`表单的所有绑定；我留给读者一个练习，即在模板中所有`input`、`textarea`和`select`元素的`value`属性上添加`validate`。本书的示例应用程序可以作为参考。

`validate`绑定行为有两个任务。首先，它将绑定指令注册到`ValidationController`，该控制器为给定组件组织验证，所以验证机制知道指令绑定的属性，并在需要时对其进行验证。其次，它可以连接到绑定指令，所以绑定到元素的属性可以在元素的目标值变化时立即验证。

### 使用 ValidationController

`ValidationController`在验证过程中扮演着指挥者的角色。它跟踪一组需要验证的绑定，提供方法手动触发验证，并记录当前的验证错误。

为了利用`ValidationController`，我们首先必须在组件中注入一个实例：

`src/contact-edition.js`

```js
import {inject, NewInstance} from 'aurelia-framework'; 
import {ValidationController} from 'aurelia-validation'; @inject(ContactGateway, NewInstance.of(ValidationController), Router) 
export class ContactEdition { 

  constructor(contactGateway, validationController, router) { 
    this.contactGateway = contactGateway; 
    this.validationController = validationController; 
    this.router = router; 
  } 
  //Omitted snippet... 
} 

```

在这里，我们向视图模型中注入了一个全新的`ValidationController`实例。使用`NewInstance`解析器很重要，因为默认情况下，DI 容器认为所有服务都是应用程序单例，而我们确实希望每个组件都有一个独特的实例，以便在验证时它们可以被孤立考虑。

接下来，我们只需要确保在保存任何联系人之前表单是有效的：

`src/contact-edition.js`

```js
//Omitted snippet... 
export class ContactEdition { 
  //Omitted snippet... 
  save() { 
    this.validationController.validate().then(errors => { 
 if (errors.length > 0) { 
 return; 
 } 
      //Omitted call to create or update... 
    } 
  } 
} 

```

在这里，我们将调用网关的`create`或`update`方法的代码封装起来，以便在验证（完成且没有错误时）执行：

`validate`方法返回一个`Promise`，该`Promise`用验证错误数组解决。这意味着验证规则可以是异步的。例如，自定义规则可以执行 HTTP 调用到后端以检查数据唯一性或执行进一步的数据验证，`validate`方法的返回`Promise`将在 HTTP 调用完成时解决。

如果异步规则的`Promise`被拒绝，例如 HTTP 调用失败，`validate`返回的`Promise`也将被拒绝，所以当使用此类异步、远程验证规则时，确保在这个层次上处理拒绝，这样用户就知道发生了什么。

### 添加验证规则

此时，验证已经准备就绪，但不会做任何事情，因为我们还没有在模型上定义任何验证规则。让我们从`Contact`类开始：

`src/models.js`

```js
import {ValidationRules} from 'aurelia-validation'; 
// Omitted snippet... 

export class Contact { 
  // Omitted snippet... 

  constructor() { 
 ValidationRules 
 .ensure('firstName').maxLength(100) 
 .ensure('lastName').maxLength(100) 
 .ensure('company').maxLength(100) 
 .ensure('birthday') 
 .satisfies((value, obj) => value === null || value === undefined 
 || value === '' || !isNaN(Date.parse(value))) 
 .withMessage('${$displayName} must be a valid date.') 
 .ensure('note').maxLength(2000) 
 .on(this); 
 } 

  //Omitted snippet... 
} 

```

在这里，我们使用`aurelia-validation`的流式 API，为`Contact`的某些属性添加验证规则：`firstName`、`lastName`和`company`属性的长度不能超过 100 个字符，`note`属性的长度不能超过 2000 个字符。

此外，我们使用`satisfies`方法为`birthday`属性定义内联的自定义规则。这个规则确保`birthday`只有在它是空值或可以解析为有效`Date`对象的字符串时才是有效的。我们还使用`withMessage`方法指定当我们的自定义规则被违反时应显示的错误消息模板。

消息模板使用与 Aurelia 的模板引擎相同的字符串插值语法，并且可以使用一个名为`$displayName`的上下文变量，它包含正在验证的属性的显示名称。

### 注意

自定义验证规则应始终接受空值。这是为了保持关注点的分离；`required`规则已经负责拒绝空值，所以你的自定义规则应只关注其自己的特定验证逻辑。这样，开发者可以根据他们想做什么，选择性地使用你的自定义规则，或不与`required`一起使用。

最后，`on`方法将刚刚构建的规则集附加到`Contact`实例的元数据中。这样，当验证`Contact`对象的属性时，验证过程可以检索应适用的验证规则。

我们还需要为`Contact`中代表列表项的所有类添加验证规则：

`src/models.js`

```js
//Omitted snippet... 

export class PhoneNumber { 
  //Omitted snippet... 

  constructor() { 
 ValidationRules 
 .ensure('type').required().maxLength(25) 
 .ensure('number').required().maxLength(25) 
 .on(this); 
 } 

  //Omitted snippet... 
} 

export class EmailAddress { 
  //Omitted snippet...   

  constructor() { 
 ValidationRules 
 .ensure('type').required().maxLength(25) 
 .ensure('address').required().maxLength(250).email() 
 .on(this); 
 } 

  //Omitted snippet...   
} 

export class Address { 
  //Omitted snippet... 

  constructor() { 
 ValidationRules 
 .ensure('type').required().maxLength(25) 
 .ensure('number').required()maxLength(100) 
 .ensure('street').required().maxLength(100) 
 .ensure('postalCode').required().maxLength(25) 
 .ensure('city').required().maxLength(100) 
 .ensure('state').maxLength(100) 
 .ensure('country').required().maxLength(100) 
 .on(this); 
 } 

  //Omitted snippet... 
} 

export class SocialProfile { 
  //Omitted snippet...   

  constructor() { 
 ValidationRules 
 .ensure('type').required().maxLength(25) 
 .ensure('username').required().maxLength(100) 
 .on(this); 
 } 

  //Omitted snippet...   
} 

```

在这里，我们将每个属性设置为`required`，并为它们指定最大长度。此外，我们确保`EmailAddress`类的`address`属性是一个有效的电子邮件地址。

## 渲染验证错误

此时，如果我们的表单无效，`save`方法不会向后端发送任何 HTTP 请求，这是正确的行为。然而，它仍然不显示任何错误消息。让我们看看如何向用户显示验证错误。

### 错误属性

控制器有一个`errors`属性，其中包含当前的验证错误。这个属性可以用来，例如，渲染一个验证摘要：

`src/contact-edition.html`

```js
<template>   
  <!-- Omitted snippet... --> 
  <form class="form-horizontal" submit.delegate="save()"> 
    <ul class="col-sm-9 col-sm-offset-3 list-group text-danger" 
        if.bind="validationController.errors"> 
      <li repeat.for="error of validationController.errors"  
          class="list-group-item"> 
        ${error.message} 
      </li> 
    </ul> 
    <!-- Omitted snippet... --> 
  </form> 
</template> 

```

在这个例子中，我们添加了一个无序列表，它将在验证控制器有错误时渲染。在这个列表内部，我们为每个`error`重复一个列表项。在每个列表项中，我们渲染错误的`message`。

### 验证错误属性

使用`validation-errors`自定义属性，也可以检索到不是所有的验证错误，而是只检索来自更窄范围的错误。

当添加到给定元素时，此属性会收集其宿主元素下所有验证过的绑定指令的验证错误，并使用双向绑定将这些错误分配给它所绑定的属性。

例如，让我们从上一个示例中移除验证摘要，并使用`validation-errors`属性为表单中的特定字段渲染错误：

`src/contact-edition.html`

```js
<template> 
  <!-- Omitted snippet... --> 
  <div validation-errors.bind="birthdayErrors"  
       class="form-group ${birthdayErrors.length ? 'has-error' : ''}"> 
    <label class="col-sm-3 control-label">Birthday</label> 
    <div class="col-sm-9"> 
      <input type="text" class="form-control"  
             value.bind="contact.birthday & validate"> 
      <span class="help-block" repeat.for="errorInfo of birthdayErrors"> 
 ${errorInfo.error.message} 
 <span> 
    </div> 
  </div> 
  <!-- Omitted snippet... --> 
</template> 

```

在这里，我们在包含`birthday`属性的`form-group div`中添加了`validation-errors`属性，我们将其绑定到新的`birthdayErrors`属性。如果`birthday`有任何错误，我们还向`form-group div`添加了`has-error` CSS 类。最后，我们添加了一个`help-block span`，它针对`birthdayErrors`数组中的每个错误重复出现，并显示错误的`message`。

### 创建自定义 ValidationRenderer

`validation-errors`属性允许我们在模板中显示特定区域的错误。然而，如果我们必须为表单中的每个属性添加此代码，这将很快变得繁琐且无效。幸运的是，`aurelia-validation`提供了一个机制，可以在一个名为验证渲染器的专用服务中提取此逻辑。

验证渲染器是一个实现`render`方法的类。这个方法以其第一个参数接收到一个验证渲染指令对象。这个指令对象包含了关于应显示哪些错误和哪些应移除的信息。它基本上是前一次和当前验证状态之间的差异，因此渲染器知道它必须对 DOM 中显示的错误消息应用哪些更改。

在撰写本文时，Aurelia 中还没有可用的验证渲染器。很可能一些社区插件很快就会提供针对主要 CSS 框架的渲染器。与此同时，让我们自己实现这个功能：

`src/validation/bootstrap-form-validation-renderer.js`

```js
export class BootstrapFormValidationRenderer { 

  render(instruction) { 
    for (let { error, elements } of instruction.unrender) { 
      for (let element of elements) { 
        this.remove(element, error); 
      } 
    } 

    for (let { error, elements } of instruction.render) { 
      for (let element of elements) { 
        this.add(element, error); 
      } 
    } 
  } 
} 

```

在这里，我们导出一个名为`BootstrapFormValidationRenderer`的类，其中包含一个`render`方法。这个方法简单地遍历`instruction`的错误来进行`unrender`，然后遍历每个错误`elements`，并调用一个`remove`方法（我们马上就会写）。接下来，它遍历`instruction`的错误来进行`render`，然后遍历每个错误`elements`，并调用一个`add`方法。

接下来，我们需要告诉我们的类如何显示验证错误，通过编写我们的验证渲染器类中的`add`方法：

```js
add(element, error) { 
  const formGroup = element.closest('.form-group'); 
  if (!formGroup) { 
    return; 
  } 

  formGroup.classList.add('has-error'); 

  const message = document.createElement('span'); 
  message.className = 'help-block validation-message'; 
  message.textContent = error.message; 
  message.id = `bs-validation-message-${error.id}`; 
  element.parentNode.insertBefore(message, element.nextSibling); 
} 

```

在这里，我们检索到离承载绑定指令触发错误的元素的`form-group` CSS 类最近的元素，并向其添加`has-error` CSS 类。接下来，我们创建一个`help-block span`，它将包含错误的`message`。我们还设置其`id`属性使用错误的`id`，这样在需要删除时可以轻松找到它。最后，我们将这个消息元素插入 DOM，紧随触发错误的元素之后。

为了完成我们的渲染器，让我们编写一个将删除先前渲染的验证错误的方法：

```js
remove(element, error) { 
  const formGroup = element.closest('.form-group'); 
  if (!formGroup) { 
    return; 
  } 

  const message = formGroup.querySelector( 
    `#bs-validation-message-${error.id}`); 
  if (message) { 
    element.parentNode.removeChild(message); 
    if (formGroup.querySelectorAll('.help-block.validation-message').length  
        === 0) {     
      formGroup.classList.remove('has-error'); 
    } 
  } 
} 

```

在这里，我们首先获取到离触发错误的绑定说明的宿主元素最近的具有`form-group`类的元素。然后我们使用错误的`id`获取消息元素，并将其从 DOM 中移除。最后，如果`form-group`不再包含任何错误消息，我们将其`has-error`类移除。

我们的验证渲染器现在必须通过依赖注入容器向应用程序提供。逻辑上，我们会在我们`validation`特性的`configure`函数中进行此操作：

`src/validation/index.js`

```js
//Omitted snippet... 
import {BootstrapFormValidationRenderer} 
 from './bootstrap-form-validation-renderer'; 

export function configure(config) { 
  config.plugin('aurelia-validation'); 
  config.container.registerHandler( 
 'bootstrap-form', 
 container => container.get(BootstrapFormValidationRenderer)); 
} 

```

在这里，我们以`bootstrap-form`的名称注册我们的验证渲染器。我们可以在我们的`contact-edition`表单中使用这个名称，告诉验证控制器应该使用这个渲染器来显示`form`的验证错误：

`src/contact-edition.html`

```js
<template> 
  <!-- Omitted snippet... --> 
  <form class="form-horizontal" submit.delegate="save()" 
        validation-renderer="bootstrap-form"> 
    <!-- Omitted snippet... --> 
  </form> 
  <!-- Omitted snippet... --> 
</template> 

```

`validation-renderer`属性将根据提供的值解析我们的`BootstrapFormValidationRenderer`实例，并将其注册到当前的验证控制器。然后控制器会在验证状态发生更改时通知渲染器，以便可以渲染新的错误并移除已解决的错误。

### 注意

使用字符串键注册渲染器使得可以注册多个不同名称的验证渲染器，因此不同的渲染器可以在不同的表单中使用。

## 更改验证触发器

默认情况下，当元素失去焦点时验证属性，但是可以通过设置控制器的`validateTrigger`属性来更改这种行为：

`src/contact-edition.js`

```js
import {ValidationController, validateTrigger} from 'aurelia-validation'; 
// Omitted snippet... 
export class ContactEdition { 
  constructor(contactGateway, validationController, router) { 
    validationController.validateTrigger = validateTrigger.change; 
    // Omitted snippet... 
  } 
} 

```

在这里，我们首先导入`validateTrigger`枚举，并告诉`ValidationController`当它们绑定的元素的值发生变化时应该重新验证属性。

除了`change`，`validateTrigger`枚举还有另外三个值：

+   `blur`：当绑定说明的宿主元素失去焦点时验证属性。这是默认值。

+   `changeOrBlur`：当绑定说明发生变化时或当宿主元素失去焦点时验证属性。它基本上结合了`change`和`blur`两种行为。

+   `manual`：完全禁用自动验证。在这种情况下，只有调用控制器的`validate`方法，如我们在`save`方法中所做的那样，才能触发验证，并且它一次性对所有注册的绑定进行验证。

当然，即使`validateTrigger`是`blur`、`change`或`blurOrChange`，显式调用`validate`方法总是执行验证。

## 创建自定义 ValidationRules

`aurelia-validation`库可以轻松添加自定义验证规则。为了说明这一点，我们首先将应用于`Contact`的`birthday`属性的规则移动到一个可重用的`date`验证规则中。然后，我们还将向我们的联系人照片上传组件添加验证，这需要一些自定义规则来验证文件。

### 验证日期

让我们先创建一个文件，该文件将声明并注册我们各种自定义规则：

`src/validation/rules.js`

```js
import {ValidationRules} from 'aurelia-validation'; 

ValidationRules.customRule( 
  'date',  
  (value, obj) => value === null || value === undefined || value === ''  
                  || !isNaN(Date.parse(value)),  
  '${$displayName} must be a valid date.' 
); 

```

这个文件没有导出任何内容。它只是导入了`ValidationRules`类，并使用其`customRule`静态方法注册了一个新的`date`规则，该规则重用了我们在`Contact`类中之前定义的准则和消息。

接下来，我们需要在某个地方导入这个文件，以便注册规则并将其提供给应用程序。最好在`validation`功能的`configure`函数中执行此操作：

`src/validation/index.js`

```js
import './rules'; 
//Omitted snippet... 

```

通过导入`rules`文件，`date`自定义规则被注册，因此一旦通过 Aurelia 导入`validation`功能，它就可以使用。

最后，我们现在可以更改`Contact`的`birthday`属性的`ValidationRules`，使其使用这个规则：

`src/models.js`

```js
//Omitted snippet... 
export class Contact { 
  //Omitted snippet... 

  constructor() { 
    ValidationRules 
      .ensure('firstName').maxLength(100) 
      .ensure('lastName').maxLength(100) 
      .ensure('company').maxLength(100) 
 .ensure('birthday').satisfiesRule('date') 
      .ensure('note').maxLength(2000) 
      .on(this); 
  } 

  //Omitted snippet... 
} 
//Omitted snippet... 

```

在这里，我们简单地移除了对`birthday`属性的`satisfies`调用，并将其替换为对`satisfiesRule`的调用，该调用期望规则名称作为其第一个参数。

### 验证文件是否被选择

在这一点上，如果未选择任何文件，联系人照片上传组件在用户点击**保存**按钮时不会做任何事情。我们在验证方面可以做的第一件事是确保已选择文件。因此，我们将创建一个名为`notEmpty`的新规则，以确保验证的值有一个`length`属性大于零：

`src/validation/rules.js`

```js
//Omitted snippet... 
ValidationRules.customRule( 
  'notEmpty', 
  (value, obj) => value && value.length && value.length > 0, 
  '${$displayName} must contain at least one item.' 
); 

```

在这里，我们使用`ValidationRules`类的`customRule`静态方法全局注册我们的验证规则。此方法期望以下参数：

+   规则的名称。它必须是唯一的。

+   条件函数，它接收值和（如果有）父对象。如果规则得到满足，它预期返回`true`，如果规则被违反，则返回`false`。它还可以返回一个`Promise`，其解析结果为`boolean`。

+   错误消息模板。

这个规则能够与具有`length`属性的任何值一起工作。例如，它可以用于数组或`FileList`实例。

### 验证文件大小

接下来，我们将创建一个验证规则，以确保`FileList`实例中的所有文件重量小于最大尺寸：

`src/validation/rules.js`

```js
//Omitted snippet... 
ValidationRules.customRule( 
  'maxFileSize', 
  (value, obj, maximum) => !(value instanceof FileList) 
    || value.length === 0 
    || Array.from(value).every(file => file.size <= maximum), 
  '${$displayName} must be smaller than ${$config.maximum} bytes.', 
  maximum => ({ maximum }) 
); 

```

在这里，我们首先定义一个新的`maxFileSize`验证规则，确保`FileList`中的每个文件的大小不超过给定的`maximum`。该规则仅在值为`FileList`实例且`FileList`不为空时适用。

此规则期望一个`maximum`参数。使用此类规则时，传递给`satisfiesRule`流畅方法的任何参数都将传递给底层的条件函数，以便它使用它来评估条件。然而，为了对消息模板可用，规则参数必须在单个对象中聚合。因此，`customRule`可以传递第四个参数，预期是一个函数，它将规则参数聚合到单个对象中。此对象随后作为`$config`对消息模板可用。

这就是我们在`maxFileSize`规则中所看到的；它期望以一个名为`maximum`的参数被调用，这是以字节为单位的最大文件大小。当向属性添加规则时，此参数预期传递给`satisfiesRule`方法：

```js
ValidationRules.ensure('photo').satisfiesRule('maxFileSize', 1024); 

```

此参数随后传递给条件函数，以便可以验证`FileList`实例中所有文件的大小。它还传递给聚合函数，该函数返回一个包含`maximum`属性的对象。此对象随后作为`$config`对消息模板可用，因此模板可以在错误消息中显示`maximum`。

在这里，我们的自定义规则只有一个参数，但一个规则可以有尽可能多的参数。它们都将以相同的顺序传递给条件函数和聚合函数，顺序与传递给`satisfiesRule`的顺序相同。

### 验证文件扩展名

最后，让我们创建一个规则，以确保`FileList`实例中的所有文件扩展名都在特定的一组值中：

`src/validation/rules.js`

```js
//Omitted snippet... 
function hasOneOfExtensions(file, extensions) { 
  const fileName = file.name.toLowerCase(); 
  return extensions.some(ext => fileName.endsWith(ext)); 
} 

function allHaveOneOfExtensions(files, extensions) { 
  extensions = extensions.map(ext => ext.toLowerCase()); 
  return Array.from(files) 
    .every(file => hasOneOfExtensions(file, extensions)); 
} 

ValidationRules.customRule( 
  'fileExtension', 
  (value, obj, extensions) => !(value instanceof FileList) 
    || value.length === 0 
    || allHaveOneOfExtensions(value, extensions), 
  '${$displayName} must have one of the following extensions: '  
    + '${$config.extensions.join(', ')}.', 
  extensions => ({ extensions }) 
); 

```

此规则名为`fileExtension`，期望一个文件扩展名数组作为参数，并确保`FileList`中的所有文件名以扩展名之一结尾。与`maxFileSize`一样，仅当验证的值是一个不为空的`FileList`实例时，它才适用。

### 验证联系照片选择器

既然我们已经定义了验证联系照片组件所需的所有规则，让我们像对`contact-edition`组件一样设置视图模型：

1.  在`ContactPhoto`视图模型中注入`ValidationController`的`NewInstance`

1.  在`save`中显式调用`validate`方法，如果有任何验证错误，则省略调用`updatePhoto`

1.  在`contact-photo.html`模板中的`form`元素添加`validation-renderer="bootstrap-form"`属性

1.  将`validate`绑定行为添加到`file input`上`files`属性的绑定

这些任务与我们对`contact-edition`组件已经完成的任务相同，我将留给读者作为练习。

接下来，我们需要向视图模型的`photo`属性添加验证规则：

`src/contact-photo.js`

```js
import {ValidationController, ValidationRules} from 'aurelia-validation'; 
//Omitted snippet... 
export class ContactPhoto { 
  //Omitted snippet... 

  constructor(contactGateway, router, validationController) { 
    //Omitted snippet... 
    ValidationRules 
      .ensure('photo') 
        .satisfiesRule('notEmpty') 
          .withMessage('${$displayName} must contain 1 file.') 
        .satisfiesRule('maxFileSize', 1024 * 1024 * 2) 
        .satisfiesRule('fileExtension', ['.jpg', '.png']) 
      .on(this); 
  } 

  //Omitted snippet... 
} 

```

在这里，我们告诉验证控制器`photo`必须至少包含一个文件，这个文件必须是 JPEG 或 PNG，并且最大不超过 2 MB。我们还使用`withMessage`方法定制当没有选择文件时显示的消息。

如果您测试这个，它应该能正常工作。然而，验证在`file input`失去焦点时触发，使得可用性有些奇怪。为了在用户关闭浏览器的文件选择对话框时立即验证表单，从而立即显示可能的错误消息，让我们将验证控制器的`validateTrigger`更改为`change`：

`src/contact-photo.js`

```js
import {ValidationController, ValidationRules, validateTrigger}  
  from 'aurelia-validation'; 
//Omitted snippet... 
export class ContactPhoto { 
  //Omitted snippet... 

  constructor(contactGateway, router, validationController) { 
    validationController.validateTrigger = validateTrigger.change; 
    //Omitted snippet... 
  } 

  //Omitted snippet... 
} 

```

如果您在做出此更改后进行测试，您应该发现可用性得到了很大改善，因为文件在用户关闭文件选择对话框时就会进行验证。

# 编辑复杂结构

在前几节中，我们创建了一个表单，用于编辑项目列表（如电话号码、电子邮件地址、地址和社会资料等），这种策略称为内联编辑。表单包括每个列表项的输入元素。这种策略将用户编辑或添加新列表项所需的点击次数降到最低，因为用户可以直接在表单中编辑所有列表项的所有字段。

然而，当表单需要管理更复杂项目的列表时，一个解决方案是只显示列表中最具相关性的信息作为只读，并使用模态对话框创建或编辑项目。对话框为单个项目显示复杂表单提供了更多的空间。

`aurelia-dialog`插件暴露了一个对话框功能，我们可以利用它来创建模态编辑器。为了说明这一点，我们将克隆我们的联系人管理应用程序，并更改`contact-edition`组件，使其使用对话框编辑而不是列表项的内联编辑。

以下代码片段是`chapter-4/samples/list-edition-models`的摘录。

## 安装对话框插件

要安装`aurelia-dialog`插件，只需在项目目录中打开一个控制台，并运行以下命令：

```js
> npm install aurelia-dialog --save 

```

安装完成后，我们还需要将插件添加到供应商包配置中。为此，请打开`aurelia_project/aurelia.json`，然后在`build`下的`bundles`中，在名为`vendor-bundle.js`的包的`dependencies`数组中添加以下条目：

```js
{ 
  "name": "aurelia-dialog", 
  "path": "../node_modules/aurelia-dialog/dist/amd", 
  "main": "aurelia-dialog" 
}, 

```

最后，我们需要在我们的主`configure`函数中加载插件：

`src/main.js`

```js
//Omitted snippet... 
export function configure(aurelia) { 
  aurelia.use 
    .standardConfiguration() 
    .plugin('aurelia-dialog') 
    .feature('validation') 
    .feature('resources'); 
  //Omitted snippet... 
} 

```

此时，`aurelia-dialog`暴露的服务和组件已准备好在我们的应用程序中使用。

## 创建编辑对话框

对话框插件使用组合来将组件作为对话框显示。这意味着下一步是创建将用于编辑新或现有项目的组件。

由于无论编辑的项目类型如何，对话框编辑的背后的行为都将相同，我们将创建一个单一的视图模型，我们将在电话号码、电子邮件地址、地址和社会资料等项目中重复使用：

`src/dialogs/edition-dialog.js`

```js
import {inject, NewInstance} from 'aurelia-framework'; 
import {DialogController} from 'aurelia-dialog'; 
import {ValidationController} from 'aurelia-validation'; 

@inject(DialogController, NewInstance.of(ValidationController)) 
export class EditionDialog { 

  constructor(dialogController, validationController) { 
    this.dialogController = dialogController; 
    this.validationController = validationController; 
  } 

  activate(model) { 
    this.model = model; 
  } 

  ok() { 
    this.validationController.validate().then(errors => { 
      if (errors.length === 0) { 
        this.dialogController.ok(this.model) 
      } 
    }); 
  } 

  cancel() { 
    this.dialogController.cancel(); 
  } 
} 

```

在这里，我们创建了一个组件，在该组件中注入了`DialogController`和`ValidationController`类的`NewInstance`。接下来，我们定义了一个接收`model`的`activate`方法，该`model`将是需要编辑的项目 - 电话号码、电子邮件地址、地址或社会资料。我们还定义了一个`ok`方法，该方法验证表单，如果没有错误，则使用更新后的`model`作为对话框的输出调用`DialogController`的`ok`方法。最后，我们定义了一个`cancel`方法，它简单地将调用委托给`DialogController`的`cancel`方法。

当`DialogController`被注入到一个作为对话框显示的组件中时，它被用来控制显示组件的对话框。它的`ok`和`cancel`方法可以用来关闭对话框，并向调用者返回一个响应。这个响应随后可以被调用者用来确定对话框是否被取消以及检索其输出（如果有）。

尽管我们将为所有项目类型重用相同的视图模型类，但每个项目类型的模板必须是不同的。让我们从电话号码的对话框编辑开始：

`src/dialogs/phone-number-dialog.html`

```js
<template> 
  <ai-dialog> 
    <form class="form-horizontal" validation-renderer="bootstrap-form"  
          submit.delegate="ok()"> 
      <ai-dialog-body> 
        <h2>Phone number</h2> 
        <div class="form-group"> 
          <div class="col-sm-2"> 
            <label for="type">Type</label> 
          </div> 
          <div class="col-sm-10"> 
            <select id="type" value.bind="model.type & validate"  
                    attach-focus="true" class="form-control"> 
              <option value="Home">Home</option> 
              <option value="Office">Office</option> 
              <option value="Mobile">Mobile</option> 
              <option value="Other">Other</option> 
            </select> 
          </div> 
        </div> 
        <div class="form-group"> 
          <div class="col-sm-2"> 
            <label for="number">Number</label> 
          </div> 
          <div class="col-sm-10"> 
            <input id="number" type="tel" class="form-control"  
                   placeholder="Phone number"  
                   value.bind="model.number & validate"> 
          </div> 
        </div> 
      </ai-dialog-body>
<ai-dialog-footer> 
        <button type="submit" class="btn btn-primary">Ok</button> 
        <button class="btn btn-danger"  
                click.trigger="cancel()">Cancel</button> 
      </ai-dialog-footer> 
    </form> 
  </ai-dialog> 
</template> 

```

这里值得注意的是`ai-dialog`、`ai-dialog-body`和`ai-dialog-footer`元素，它们是 Aurelia 对话框的容器。此外，`select`元素上的`attach-focus="true"`属性确保当对话框显示时这个元素获得焦点。最后，`form`的`submit`事件委托给`ok`方法，而点击取消按钮则委托给`cancel`方法。

模板的其余部分应该很熟悉。用户输入元素绑定到`model`的属性，这些绑定被`validate`绑定行为装饰，以便属性得到适当验证。

我们还需要为其他项目类型创建模板：`src/dialogs/email-address-dialog.html`、`src/dialogs/address-dialog.html`和`src/dialogs/social-profile-dialog.html`。此时，这些模板应该很容易创建。我将留给读者一个练习来编写它们；`list-edition-models`示例可以作为参考。

## 使用编辑对话框

利用我们新的视图模型和模板的最后一步是改变`contact-edition`组件的行为：

`src/contact-edition.js`

```js
import {DialogService} from 'aurelia-dialog'; 
import {Contact, PhoneNumber, EmailAddress, Address, SocialProfile}  
  from './models'; 
//Omitted snippet... 
@inject(ContactGateway, NewInstance.of(ValidationController), Router,  
        DialogService) 
export class ContactEdition { 
  constructor(contactGateway, validationController, router, dialogService) { 
    this.contactGateway = contactGateway; 
    this.validationController = validationController; 
    this.router = router; 
    this.dialogService = dialogService; 
  } 
   //Omitted snippet... 
  _openEditDialog(view, model) { 
    return new Promise((resolve, reject) => { 
      this.dialogService.open({  
        viewModel: 'dialogs/edition-dialog', 
        view: `dialogs/${view}-dialog.html`,  
        model: model 
      }).then(response => { 
        if (response.wasCancelled) { 
          reject(); 
        } else { 
          resolve(response.output); 
        } 
      }); 
    }); 
  } 

  editPhoneNumber(phoneNumber) { 
    this._openEditDialog('phone-number',  
                         PhoneNumber.fromObject(phoneNumber)) 
      .then(result => { Object.assign(phoneNumber, result); }); 
  } 

  addPhoneNumber() { 
    this._openEditDialog('phone-number', new PhoneNumber()) 
      .then(result => { this.contact.phoneNumbers.push(result); }); 
  } 

  //Omitted snippet... 
} 

```

在这里，我们通过在构造函数中注入`DialogService`来向我们的`ContactEdition`视图模型添加一个新的依赖。接下来，我们定义了一个`_openEditDialog`方法，它定义了打开编辑对话框的通用行为。

此方法调用`DialogService`的`open`方法来打开一个对话框，使用`edition-dialog`视图模型和给定项目类型的模板，组合成一个单一组件。还传递了一个`model`，它将在`edition-dialog`的`activate`方法中注入。如果你阅读了第三章*显示数据*中的组合部分，这应该会很熟悉。

此外，该方法返回一个 `Promise`，当用户点击 **确定** 时解析，但当用户点击 **取消** 时拒绝。这样，在使用这个方法时，只有当用户通过点击 **确定** 来确认其修改时，结果的 `Promise` 才会被解析，否则会被拒绝。

`editPhoneNumber` 方法使用 `_openEditDialog` 方法来显示电话号码编辑对话框。要编辑的 `phoneNumber` 的副本作为 `model` 传递，因为如果我们传递原始 `phoneNumber` 对象，即使用户取消其修改，它也会被修改。当用户确认其修改时，`Promise` 解析，这时更新后的 `model` 属性会被回赋给原始的 `phoneNumber`。

同样地，`addPhoneNumber` 方法使用了 `_openEditDialog` 方法，但传递了一个新的 `PhoneNumber` 实例作为模型。另外，当 `Promise` 解析时，新的电话号码会被添加到 `contact` 的 `phoneNumbers` 数组中。

最后，模板必须更改，以便电话号码列表以只读方式显示，并为每个电话号码添加一个新的 **编辑** 按钮：

`src/contact-edition.html`

```js
<template> 
  <!-- Omitted snippet... --> 
  <hr> 
  <div class="form-group" repeat.for="phoneNumber of contact.phoneNumbers"> 
    <div class="col-sm-2 col-sm-offset-1">${phoneNumber.type}</div> 
    <div class="col-sm-7">${phoneNumber.number}</div> 
    <div class="col-sm-1"> 
      <button type="button" class="btn btn-danger"  
              click.delegate="editPhoneNumber(phoneNumber)"> 
        <i class="fa fa-pencil"></i> Edit 
      </button> 
    </div> 
    <div class="col-sm-1"> 
      <button type="button" class="btn btn-danger"  
              click.delegate="contact.phoneNumbers.splice($index, 1)"> 
        <i class="fa fa-times"></i>  
      </button> 
    </div> 
  </div> 
  <div class="form-group"> 
    <div class="col-sm-9 col-sm-offset-3"> 
      <button type="button" class="btn btn-primary"  
              click.delegate="addPhoneNumber()"> 
        <i class="fa fa-plus-square-o"></i> Add a phone number 
      </button> 
    </div> 
  </div> 
  <!-- Omitted snippet... --> 
</template> 

```

在这里，我们移除了 `select` 和 `input` 元素，并用字符串插值指令来显示 `phoneNumber` 的 `type` 和 `number` 属性。我们还添加了一个 **编辑** 按钮，当点击时，调用新的 `editPhoneNumber` 方法。最后，我们更改了 **添加** 按钮，使其调用新的 `addPhoneNumber` 方法。

当然，对于 `contact-edition` 组件的视图模型和模板，以及其他项目类型的更改也必须应用相同的更改。然而，对于电子邮件地址、地址和社会资料的内联编辑策略的更改，现在对您来说应该是很直接的；我将把这个留给读者作为一个练习。

# 总结

使用 Aurelia 创建表单很简单，主要是利用双向绑定。验证表单也很容易，得益于验证插件。此外，验证插件的抽象层允许我们使用我们想要的验证库，尽管插件提供的默认实现已经相当强大。

在下一章中，Aurelia 的力量将真正开始变得清晰。通过利用我们迄今为止看到的内容，并添加自定义属性、自定义元素和内容投射到混合中，我们将能够创建极其强大、可重用和可扩展的组件，将它们组合成模块化和可测试的应用程序。当然，在覆盖这些主题的同时，我们将对我们的联系人管理应用程序进行大量重构，以提取组件和可重用行为，同时添加在没有自定义元素和属性时无法实现的特性。
