# *第九章*：使用 JavaScript 改进网站交互功能

在本章中，我们将学习如何使用不同的交互功能来改进我们的网站。这些功能将使用 JavaScript 创建，在大多数情况下，将使用基于 Bootstrap 5 的 JavaScript 组件。在前一章中，我们自定义了网站的*外观* – 在本章中，我们将自定义网站的*感觉*。

Bootstrap 5 有几个使用 JavaScript 的交互组件。其中一些组件通过使用数据属性自动初始化，例如，**手风琴**、**轮播图**、**下拉菜单**、**侧边栏**、**导航栏**、**模态框**和**标签页**组件，这些我们目前在网站上正在使用。其他组件需要使用 JavaScript 进行自定义初始化，例如弹出框、通知和工具提示组件。表单验证需要在正确的时间执行自定义 JavaScript，而旋转器组件应在浏览器处于加载状态时才可见。正如你所看到的，Bootstrap 5 中 JavaScript 以不同的方式用于不同的目的。

在本章中，我们将看到如何使用 JavaScript 来改进我们的网站，以下场景将使用交互功能：

+   向表单标签添加工具提示组件

+   向与产品相关的操作添加通知组件

+   在购物车表单中更改产品数量

+   添加表单验证

+   添加表单提交加载指示器

+   添加表单成功消息

+   创建程序化标签导航

+   更新结账流程中的进度状态

+   为产品画廊创建灯箱

所有这些场景都将影响我们网站各个部分或组件的*感觉*和*行为*，这就是为什么它们被包含在本章中的原因。

# 技术要求

+   要预览示例，你需要一个代码编辑器和浏览器。所有代码示例的源代码可以在以下位置找到：[`github.com/PacktPublishing/The-Missing-Bootstrap-5-Guide`](https://github.com/PacktPublishing/The-Missing-Bootstrap-5-Guide)

+   要将 Sass 编译为 CSS，你需要以下之一：

    +   **Node.js**，如果你更喜欢使用终端（Mac）或命令提示符（Windows）的**命令行界面**（CLI）

    +   **Scout-App**，如果你更喜欢使用**图形用户界面**（GUI）

    +   **Visual Studio Code**，如果你更喜欢使用 Visual Studio Code 市场中的扩展

所有这些方法都在*第二章* *使用和编译 Sass*中进行了解释。

# 关于代码示例

在我们接下来更详细地查看即将到来的各个示例之前，这里有一些关于代码示例的注意事项，提前了解这些内容会很有帮助。我鼓励你使用自己的编辑器查看本章中每个代码示例提到的各种文件，以便你可以亲自看到这些变化。此外，我也鼓励你通过浏览器查看项目，以便你可以体验所做的改进。本章是关于交互式功能的，所以要看到它带来的真正差异，必须在浏览器中查看。

## 简单和干净的 JavaScript 代码

网站的定制 JavaScript 代码是用简单和干净的 JavaScript 编写的。重点是使其易于理解，而不一定是最佳实践。所有代码都在 `bootstrap.bundle.min.js` Bootstrap 5 JavaScript 文件之后放置在同一个 `script.js` 文件中。

## 页面 ID

页面 ID 已添加到 `<body>` 标签上的 `script.js`，通常根据代码应执行的页面或页面组进行分组，并且它们也以本章中展示的大致相同顺序出现。

`<body>` 标签上的更新代码看起来如下：

part-2/chapter-9/website/index.xhtml 行 13

```js
<body id="homePage">
```

part-2/chapter-9/website/shop.xhtml 行 13

```js
<body id="shopPage">
```

part-2/chapter-9/website/product.xhtml 行 13

```js
<body id="productPage">
```

part-2/chapter-9/website/contact.xhtml 行 13

```js
<body id="contactPage">
```

part-2/chapter-9/website/wishlist.xhtml 行 13

```js
<body id="wishlistPage">
```

part-2/chapter-9/website/cart.xhtml 行 13

```js
<body id="cartPage">
```

以下是使用这些 ID 的条件 `if` 语句的示例说明：

part-2/chapter-9/website/js/script.js

```js
// Variables for individual pages
```

```js
const homePage = document.querySelector('#homePage'); // index.xhtml
```

```js
// Used like this for one page only
```

```js
if (homePage) {
```

```js
  [code added here]
```

```js
}
```

```js
// Used like this for multiple pages
```

```js
if (homePage || shopPage || productPage) {
```

```js
  [code added here]
```

```js
}
```

由于我希望页面变量的条件`if`语句尽可能简单，所以代码示例中不会展示这些语句。实际上，源代码中的`if`语句可以忽略。它们对于理解如何实现各种交互式功能不是必需的，但“只是”用于处理代码的不同部分执行时，以避免“破坏”代码。

## 部分和完整代码

在本章中，一些代码示例可能不会看起来像是完整和完成的源代码。这是因为，对于一些交互式功能，作为一个起点，我会向你展示所需的最少代码。然后，在之后，当我们添加更多功能时，我会将新代码添加到现有的代码示例中。选择这种方法是为了逐步引导你了解代码的工作原理。

当一个代码示例更新为另一个功能的新代码时，新代码将被突出显示。以下是一个说明这意味着什么的示例：

script.js

```js
this is the minimum required code
```

```js
  for an interactive feature
```

```js
  this is new code
```

```js
    for another feature that we add to
```

```js
  the existing code example
```

```js
that we can think of as partial code
```

## 原始和更新代码

一些代码示例展示了原始代码（来自*第八章**，*使用 Bootstrap 5 变量、实用 API 和 Sass 自定义网站*）和更新的新代码。代码将以相同的代码块并排展示，更新的代码将突出显示，如下所示：

script.js

```js
// Original code
```

```js
<label for="securityCode">Lorem ipsum</label>
```

```js
// Updated code
```

```js
<label class="form-label d-flex justify-content-between" for="securityCode">Lorem ipsum</label>
```

## 行号

为了使源代码的导航更简单，大多数代码示例都添加了行号，以及文件路径。这对于 JavaScript 和 Sass 代码示例尤其如此，但也包括一些 HTML 代码示例。

# 将工具提示组件添加到表单标签

现在我们将在**购物车**页面上的结账流程中的表单标签处添加一个工具提示组件，为用户提供一些在鼠标悬停（或在触摸设备上轻触）时触发的有用上下文信息。我们将把这个工具提示组件添加到位于表单中**支付选项**标签页的信用卡安全码输入旁边的图标上。

我们需要做的第一件事是添加适当的标记。在标签文本之后，我们将添加一个`.bi-question-circle`图标，并用`.text-info`着色，然后在这个图标中放入我们工具提示组件所需的属性。我们的标记将按以下方式更新：

part-2/chapter-9/website/cart.xhtml

```js
// Original code
```

```js
<label class="form-label d-flex justify-content-between" for="securityCode">Security code:</label>
```

```js
// Updated code
```

```js
<label class="form-label d-flex justify-content-between" for="securityCode">Security code: <i class="bi-question-circle text-info" data-bs-toggle="tooltip" title="Lorem ipsum dolor sit amet, consectetur adipiscing elit. Duis convallis velit quis sapien sollicitudin ultrices."></i></label>
```

工具提示组件不是通过使用`data-bs-toggle="tooltip"`自动初始化的，这是我们可能期望的。相反，出于性能原因，工具提示组件必须由我们自己初始化。因此，我们将添加以下 JavaScript，它将使用`Tooltip()`构造函数初始化页面上的所有工具提示组件：

part-2/chapter-9/website/js/script.js 行 3-7

```js
// Initialize all tooltips
```

```js
const tooltipTriggerElements = document.querySelectorAll('[data-bs-toggle="tooltip"]');
```

```js
for (let i = 0; i < tooltipTriggerElements.length; i++) {
```

```js
  new bootstrap.Tooltip(tooltipTriggerElements[i])
```

```js
}
```

通过这些更改，你将能够在浏览器中看到新的图标和工具提示组件的实际效果。目前，我们网站上只有一个工具提示，所以我们本可以选择另一种实现方式，但我喜欢这种方式，因为它可以容纳我们可能稍后选择添加的任何额外工具提示。

# 将 toast 组件添加到与产品相关的操作

现在，我们将添加由不同的产品相关操作触发的 toast 组件，例如，当在**首页**、**商店**、**购物车**、**产品**和**愿望单**页面的产品卡片底部点击操作按钮时。

在创建单个 toast 组件之前，我们将向之前提到的每个页面添加一个 toast 容器，位于打开的`<body>`标签之后和`<header>`标签之前。我们将使用以下代码：

```js
<div class="toast-container position-fixed end-0 mt-3 me-3"></div>
```

现在，我们遇到了一个问题，即 toast 容器不会沿着我们布局的 z 轴正确定位。页面上的其他组件将被放置在顶部，而这不是我们想要的。这是 navbar、按钮组和下拉组件的情况。我们将通过在我们的自定义样式表中使用以下 Bootstrap 5 `z-index`变量之一来解决这个问题：

bootstrap/scss/_variables.scss

```js
1068 $zindex-dropdown:                   1000 !default;
```

```js
1069 $zindex-sticky:                     1020 !default;
```

```js
1070 $zindex-fixed:                      1030 !default;
```

```js
1071 $zindex-offcanvas-backdrop:         1040 !default;
```

```js
1072 $zindex-offcanvas:                  1045 !default;
```

```js
1073 $zindex-modal-backdrop:             1050 !default;
```

```js
1074 $zindex-modal:                      1055 !default;
```

```js
1075 $zindex-popover:                    1070 !default;
```

```js
1076 $zindex-tooltip:                    1080 !default;
```

从前面的列表中，我们可以看到`$zindex-fixed`变量将是一个很好的选择。这是因为我们的 toast 容器已经使用了`.position-fixed`类，我们希望它在我们的页眉中的货币下拉菜单上方显示，并在 offcanvas 背景下方。然后，我们可以在我们的自定义样式表中添加以下 Sass 代码：

part-2/chapter-9/website/scss/_custom-styles.scss 行 12-15

```js
// Add z-index to get the right position along the z-axis
```

```js
.toast-container {
```

```js
  z-index: $zindex-fixed;
```

```js
}
```

我们将在以下场景中使用吐司组件：

+   将产品添加到购物车

+   将产品添加到心愿单

+   将所有产品添加到购物车

+   从心愿单中移除产品

+   从购物车中移除产品

我们将逐一查看所有这些内容。

## 将产品添加到购物车

当用户点击位于**首页**、**商店**、**产品**和**心愿单**页面产品卡片页脚中的**添加到购物车**按钮时，我们希望显示一个吐司组件。

首先，我们将向所有上述页面上的吐司容器中添加以下标记：

part-2/chapter-9/website/index.xhtml

```js
<div class="toast js-cartToast" role="status" aria-live="polite" aria-atomic="true">
```

```js
  <div class="toast-header fw-bold">Added to cart</div>
```

```js
  <div class="toast-body">Product Name was added to the 
```

```js
    cart.</div>
```

```js
</div>
```

注意在前面代码中，我们已经将`.js-cartToast`类添加到 Bootstrap 5 代码中，用于默认的吐司组件。我们将使用它作为初始化吐司组件时使用的 JavaScript 的钩子。

在我们这样做之前，我们还需要更新我们的`.js-addToCart`类，作为我们的 JavaScript 钩子：

part-2/chapter-9/website/index.xhtml

```js
<button type="button" class="btn btn-primary me-2 js-addToCart">
```

```js
  <i class="bi-cart-plus"></i><span class=
```

```js
    "visually-hidden">Add to cart</span>
```

```js
</button>
```

现在，当点击以下 JavaScript 代码中的任何**添加到购物车**按钮时，我们可以使用`Toast()`构造函数和`show()`方法来显示吐司组件：

part-2/chapter-9/website/js/script.js 行 20-27

```js
// Handle button click and show toast when adding product 
```

```js
// to cart
```

```js
const addToCartButtons = document.querySelectorAll('.js-addToCart');
```

```js
const addToCartToast = document.querySelector('.js-cartToast');
```

```js
for (let i = 0; i < addToCartButtons.length; i++) {
```

```js
  addToCartButtons[i].addEventListener('click', function() {
```

```js
    new bootstrap.Toast(addToCartToast).show();
```

```js
  })
```

```js
}
```

使用 JavaScript 方法与交互式 Bootstrap 5 组件

在本章的交互式功能中，`show()`方法将被多次使用。我们将在*第十一章*“使用具有高级 JavaScript 功能的 Bootstrap 5”中学习如何使用 JavaScript 方法与交互式 Bootstrap 5 组件一起使用。

## 将产品添加到心愿单

当用户点击位于**首页**、**商店**和**产品**页面产品卡片页脚中的**添加到心愿单**下拉菜单中的按钮时，我们希望显示一个吐司组件。

首先，我们将向所有上述页面上的吐司容器中添加以下标记：

part-2/chapter-9/website/index.xhtml

```js
<div class="toast js-wishlistToast" role="status" aria-live="polite" aria-atomic="true">
```

```js
  <div class="toast-header fw-bold">Added to wishlist</div>
```

```js
  <div class="toast-body">Product Name was added to the 
```

```js
    wishlist.</div>
```

```js
</div>
```

注意在前面代码中，我们已经将`.js-wishlistToast`类添加到 Bootstrap 5 代码中，用于默认的吐司组件。我们将使用它作为初始化吐司组件时使用的 JavaScript 的钩子。

在我们这样做之前，我们还需要更新`.js-addToWishlist`类中的按钮，作为我们的 JavaScript 钩子：

part-2/chapter-9/website/index.xhtml

```js
<div class="dropdown">
```

```js
  <button type="button" class="btn btn-secondary" 
```

```js
    id="addToWishlistDropdown1" data-bs-toggle="dropdown" 
```

```js
    aria-expanded="false"><i class="bi-heart"></i><span 
```

```js
    class="visually-hidden">Add to wishlist</span></button>
```

```js
  <div class="dropdown-menu" aria-
```

```js
    labelledby="addToWishlistDropdown1">
```

```js
    <h6 class="dropdown-header">Select wishlist</h6>
```

```js
    <button type="button" class="dropdown-item js-
```

```js
      addToWishlist">Wishlist #1</button>
```

```js
    <button type="button" class="dropdown-item js-
```

```js
      addToWishlist">Wishlist #2</button>
```

```js
    <button type="button" class="dropdown-item js-
```

```js
      addToWishlist">Wishlist #3</button>
```

```js
  </div>
```

```js
</div>
```

现在，当点击任何**添加到心愿单**下拉菜单中的按钮时，我们可以使用以下 JavaScript 代码来初始化吐司组件：

part-2/chapter-9/website/js/script.js 行 34-41

```js
// Handle button click and show toast when adding product 
```

```js
// to wishlist
```

```js
const addToWishlistButtons = document.querySelectorAll('.js-addToWishlist');
```

```js
const addToWishlistToast = document.querySelector('.js-wishlistToast');
```

```js
for (let i = 0; i < addToWishlistButtons.length; i++) {
```

```js
  addToWishlistButtons[i].addEventListener('click', 
```

```js
    function() {
```

```js
    new bootstrap.Toast(addToWishlistToast).show();
```

```js
  })
```

```js
}
```

## 将所有产品添加到购物车

当用户点击**心愿单**页面中的`<aside>`元素时，我们希望显示一个吐司组件。

首先，我们将向该页面的吐司容器中添加以下标记：

part-2/chapter-9/website/wishlist.xhtml

```js
<div class="toast js-addAllToast" role="status" aria-live="polite" aria-atomic="true">
```

```js
  <div class="toast-header fw-bold">All added to cart</div>
```

```js
  <div class="toast-body">All products from wishlist were 
```

```js
    added to the cart.</div>
```

```js
</div>
```

注意在前面代码中，我们已经为 Bootstrap 5 代码添加了`.js-addAllToast`类以创建默认的吐司组件。我们将使用这个类作为初始化吐司组件的 JavaScript 钩子。

在此之前，我们还需要更新`.js-addAllToCart`类作为 JavaScript 的钩子：

part-2/chapter-9/website/wishlist.xhtml

```js
<button type="button" class="btn btn-primary mt-3 js-addAllToCart">Add all products to cart</button>
```

现在，当点击**将所有产品添加到购物车**按钮时，我们可以使用以下 JavaScript 代码初始化吐司组件：

part-2/chapter-9/website/js/script.js 行 48-53

```js
// Handle button click and show toast when adding all 
```

```js
// products to cart
```

```js
const addAllToCartButton = document.querySelector('.js-addAllToCart');
```

```js
const addAllToCartToast = document.querySelector('.js-addAllToast');
```

```js
addAllToCartButton.addEventListener('click', function() {
```

```js
  new bootstrap.Toast(addAllToCartToast).show();
```

```js
})
```

## 从愿望单中移除产品

我们希望在用户点击位于**愿望单**页面产品卡片页脚的**从愿望单移除**按钮时显示一个吐司组件。同时，我们还想从页面上移除产品卡片以及相关的 DOM 标记。

首先，我们将添加以下标记到该页面的吐司容器中：

part-2/chapter-9/website/wishlist.xhtml

```js
<div class="toast js-wishlistToast" role="status" aria-live="polite" aria-atomic="true">
```

```js
  <div class="toast-header fw-bold">Removed from 
```

```js
    wishlist</div>
```

```js
  <div class="toast-body">Product Name was removed from the 
```

```js
    wishlist.</div>
```

```js
</div>
```

注意在前面代码中，我们已经为 Bootstrap 5 代码添加了`.js-wishlistToast`类以创建默认的吐司组件。我们将使用这个类作为初始化吐司组件的 JavaScript 钩子。

在此之前，我们还需要更新我们的`.js-removeFromWishlist`类作为 JavaScript 的钩子：

part-2/chapter-9/website/wishlist.xhtml

```js
<button type="button" class="btn btn-danger js-removeFromWishlist">
```

```js
  <i class="bi-x-circle"></i><span class="visually-
```

```js
    hidden">Remove from wishlist</span>
```

```js
</button>
```

现在，当点击任何**从愿望单移除**按钮时，我们可以初始化吐司组件，并使用以下 JavaScript 代码从 DOM 中移除元素：

part-2/chapter-9/website/js/script.js 行 55-63

```js
// Handle button click and show toast + remove element from 
```

```js
// DOM when removing product from wishlist
```

```js
const removeFromWishlistButtons = document.querySelectorAll('.js-removeFromWishlist');
```

```js
const removeFromWishlistToast = document.querySelector('.js-wishlistToast');
```

```js
for (let i = 0; i < removeFromWishlistButtons.length; i++) {
```

```js
  removeFromWishlistButtons[i].addEventListener('click', 
```

```js
    function() {
```

```js
    new bootstrap.Toast(removeFromWishlistToast).show();
```

```js
    removeFromWishlistButtons[i].closest(
```

```js
      '.card').parentElement.remove();
```

```js
  })
```

```js
}
```

## 从购物车中移除产品

在这里，我们希望在用户点击**购物车**页面**购物车**选项卡中的产品概览中的移除按钮时显示一个吐司组件。同时，我们还想从页面上移除产品以及相关的 DOM 标记。

首先，我们将添加以下标记到该页面的吐司容器中：

part-2/chapter-9/website/cart.xhtml

```js
<div class="toast js-cartToast" role="status" aria-live="polite" aria-atomic="true">
```

```js
  <div class="toast-header fw-bold">Removed from cart</div>
```

```js
  <div class="toast-body">Product Name was removed from the 
```

```js
    cart.</div>
```

```js
</div>
```

注意在前面代码中，我们已经为 Bootstrap 5 代码添加了`.js-cartToast`类以创建默认的吐司组件。我们将使用这个类作为初始化吐司组件的 JavaScript 钩子。

在此之前，我们还需要通过添加`.js-removeFromCart`类作为 JavaScript 的钩子来更新我们的移除按钮：

part-2/chapter-9/website/cart.xhtml

```js
<button type="button" class="btn btn-danger js-removeFromCart"><i class="bi-x-circle"></i></button>
```

现在，当任何移除按钮被点击时，我们可以初始化吐司组件，并使用以下 JavaScript 代码从 DOM 中移除元素：

part-2/chapter-9/website/js/script.js 行 71-78

```js
// Handle button click and show toast + remove element from DOM when removing product from cart
```

```js
const removeFromCartButtons = document.querySelectorAll('.js-removeFromCart');
```

```js
const removeFromCartToast = document.querySelector('.js-cartToast');
```

```js
for (let i = 0; i < removeFromCartButtons.length; i++) {
```

```js
  removeFromCartButtons[i].addEventListener('click', 
```

```js
    function() {
```

```js
    new bootstrap.Toast(removeFromCartToast).show();
```

```js
    removeFromCartButtons[i].parentElement.
```

```js
      parentElement.remove();
```

```js
  })
```

```js
}
```

现在，我们已经完成了向不同产品相关操作添加吐司组件的工作。接下来，我们将继续添加一个功能，用于更改购物车中单个产品的数量。

# 修改购物车中的产品数量

在购物车标签页中`.js-productQuantity`类的产品概览中，针对每个产品的输入组组件使用它作为我们的 JavaScript 的钩子。我们的输入组组件的标记将按以下方式更新：

part-2/chapter-9/website/cart.xhtml

```js
<div class="input-group me-3 js-productQuantity">
```

```js
  <button type="button" class="btn btn-secondary"><i 
```

```js
    class="bi-dash-circle"></i></button>
```

```js
  <input type="text" class="form-control text-center" 
```

```js
    value="1" aria-label="Product quantity">
```

```js
  <button type="button" class="btn btn-secondary"><i 
```

```js
    class="bi-plus-circle"></i></button>
```

```js
</div>
```

现在我们可以使用以下 JavaScript 来减少数量（如果它大于`1`，因此它永远不会降到`0`或以下）或增加它：

part-2/chapter-9/website/js/script.js 行 81-94

```js
// Handle button click on input group buttons and adjust 
```

```js
// value of input
```

```js
const productQuantityInputGroup = document.querySelectorAll('.js-productQuantity');
```

```js
for (let i = 0; i < productQuantityInputGroup.length; i++) {
```

```js
  let productQuantityInput = 
```

```js
    productQuantityInputGroup[i].querySelector(
```

```js
    '.form-control');
```

```js
  const productQuantitySubtractButton = 
```

```js
    productQuantityInputGroup[i].querySelectorAll(
```

```js
    '.btn')[0];
```

```js
  const productQuantityAddButton = 
```

```js
    productQuantityInputGroup[i].querySelectorAll(
```

```js
    '.btn')[1];
```

```js
  productQuantitySubtractButton.addEventListener('click', 
```

```js
    function() {
```

```js
    if (productQuantityInput.value > 1) {
```

```js
      productQuantityInput.value--;
```

```js
    }
```

```js
  })
```

```js
  productQuantityAddButton.addEventListener('click', 
```

```js
    function() {
```

```js
    productQuantityInput.value++;
```

```js
  })
```

```js
}
```

在添加此功能后，我们将继续为所有表单添加表单验证。

# 添加表单验证

Bootstrap 5 将表单验证作为其功能之一，因此我们想利用这一点来处理一些表单：主页上的新闻通讯表单，联系页面上的联系表单，以及购物车页面上的**发货详情**和**支付选项**标签页中的两个表单。

我们需要做的第一件事是为所有表单元素添加一个`id`属性，这样我们就可以从我们的 JavaScript 代码中定位它们。此外，我们还需要为表单元素添加`novalidate`属性，该属性用于禁用浏览器默认的反馈工具提示，同时仍然可以通过 JavaScript 访问表单验证。因此，我们需要对我们的标记进行以下更新：

part-2/chapter-9/website/index.xhtml 行 306

```js
<form class="text-white" id="newsletterForm" novalidate>
```

part-2/chapter-9/website/contact.xhtml 行 185

```js
<form id="contactForm" novalidate>
```

part-2/chapter-9/website/cart.xhtml 行 172

```js
<form id="shippingForm" novalidate>
```

part-2/chapter-9/website/cart.xhtml 行 336

```js
<form id="paymentForm" novalidate>
```

现在我们需要为有效和无效状态添加反馈信息。这将添加到具有所需属性的每个表单控件之后，并且我们希望在其中有反馈信息。我们不想在新闻通讯表单中的复选框添加此类反馈信息。这是因为有效和无效状态的样式足以让用户理解这种表单控件。

这是主页上新闻通讯表单的第一个表单字段的标记更新，以便容纳反馈信息：

part-2/chapter-9/website/index.xhtml

```js
<div class="mb-3">
```

```js
  <label for="firstName" class="visually-hidden">First 
```

```js
    name:</label>
```

```js
  <input type="text" class="form-control" 
```

```js
    placeholder="Firstname" id="firstName" required>
```

```js
  <p class="valid-feedback">Valid feedback text</p>
```

```js
  <p class="invalid-feedback">Invalid feedback text</p>
```

```js
</div>
```

在将这些反馈信息添加到所有我们希望在前面提到的所有表单字段中后，我们现在可以实施 JavaScript 中的表单验证。

首先，我们将表单存储在一个变量中，并监听表单的`submit`事件，该事件在点击提交按钮时触发。如果表单无效，我们将阻止表单提交；在此条件之外，将`.was-validated`类添加到表单中，以显示有效和无效状态的正确样式和反馈信息。

这是新闻通讯表单表单验证的 JavaScript 代码：

part-2/chapter-9/website/js/script.js 行 104-123

```js
// Newsletter form
```

```js
const newsletterForm = document.querySelector('#newsletterForm');
```

```js
newsletterForm.addEventListener('submit', function (event) {
```

```js
  if (!newsletterForm.checkValidity()) {
```

```js
    event.preventDefault();
```

```js
    event.stopPropagation();
```

```js
  }
```

```js
  newsletterForm.classList.add('was-validated');
```

```js
}, false)
```

在尝试提交无效表单后，每当用户更新表单中的任何表单字段时，都会再次运行验证。只有第一次验证不会运行，直到用户尝试提交表单。

# 添加表单提交加载指示器

我们希望为通讯录表单、联系表单和支付表单的表单提交添加加载指示器，这样用户可以看到浏览器正在等待服务器的响应。对于我们的网站，我们不会将任何表单提交到服务器或等待任何响应，因此我们将模拟这种行为。

我们的加载指示器将简单地是 Bootstrap 5 的旋转组件（更准确地说，是边框旋转变体），它将替换提交按钮的文本。

我们需要做的第一件事是为所有想要添加加载指示器的提交按钮添加一个`id`属性。然后，我们可以使用这些`id`属性在 JavaScript 代码中定位提交按钮。

这就是我们的提交按钮更新后的标记看起来像什么：

part-2/chapter-9/website/index.xhtml

```js
<button type="submit" class="btn btn-primary" id="newsletterButton">Subscribe</button>
```

part-2/chapter-9/website/contact.xhtml

```js
<button type="submit" class="btn btn-primary" id="contactButton">Submit</button>
```

part-2/chapter-9/website/cart.xhtml

```js
<button type="submit" class="btn btn-primary" id="paymentButton">Pay</button>
```

我们将使用以下代码将旋转组件的标记添加到我们的 JavaScript 中的一个变量中：

part-2/chapter-9/website/js/script.js 行 98-100

```js
// Markup for spinner component used on various forms
```

```js
const spinnerMarkup = '<span class="spinner-border spinner-border-sm me-2" role="status" aria-hidden="true"></span>Please wait...';
```

现在这个变量可以用来在我们需要时将旋转组件插入到我们的提交按钮中。

我们将在处理表单验证的脚本中添加加载指示器的代码。如果表单有效，首先，我们将阻止表单提交（以便我们可以模拟加载行为）。然后，我们将提交按钮的文本替换为旋转组件，并禁用提交按钮。之后，我们将使用`setTimeout()`方法模拟`2000`毫秒的加载时间，之后提交按钮内的旋转组件将被一些新的文本替换。

使用加载指示器的表单提交更新代码，再次以通讯录表单为例，现在看起来是这样的：

part-2/chapter-9/website/js/script.js 行 104-123

```js
// Newsletter form
```

```js
const newsletterForm = document.querySelector('#newsletterForm');
```

```js
newsletterForm.addEventListener('submit', function (event) {
```

```js
  if (!newsletterForm.checkValidity()) {
```

```js
    event.preventDefault();
```

```js
    event.stopPropagation();
```

```js
  } else {
```

```js
    event.preventDefault();
```

```js
const newsletterButton = 
```

```js
      document.querySelector('#newsletterButton');
```

```js
    newsletterButton.innerHTML = spinnerMarkup;
```

```js
    newsletterButton.disabled = true;
```

```js
    setTimeout(function() {
```

```js
      const newsletterConfirmationText = 'Subscribed';
```

```js
newsletterButton.innerHTML = 
```

```js
        newsletterConfirmationText;
```

```js
    }, 2000)
```

```js
  }
```

```js
  newsletterForm.classList.add('was-validated');
```

```js
}, false)
```

现在我们已经添加了表单提交加载指示器，但我们还有一个表单的交互特性。这是表单成功消息，我们将在下一个步骤中创建它。

# 添加表单成功消息

在上一个示例中，在表单提交完成后加载结束，提交按钮的文本被替换。现在我们希望在加载完成后，在我们的表单下方也添加表单成功消息。

我们将使用带有`$success`颜色的警告组件来显示我们的成功消息，并将其放置在折叠组件内，以便可以通过垂直过渡显示。为了实现这一点，首先，我们将添加以下标记到我们的表单中，在提交和重置按钮之后以及`</form>`标签之前，再次以通讯录表单为例：

part-2/chapter-9/website/index.xhtml

```js
<div class="collapse mt-3" id="newsletterConfirmation">
```

```js
  <div class="alert alert-success">
```

```js
    <h3 class="alert-heading h4">Subscribed!</h3>
```

```js
    <p class="mb-0">Lorem ipsum dolor sit amet, consectetur 
```

```js
      adipiscing elit. Donec mattis posuere consequat. 
```

```js
      Nulla fermentum sodales augue, vitae ornare eros 
```

```js
      ornare quis. Donec lectus est, congue eu risus quis, 
```

```js
      tempus sagittis nunc.</p>
```

```js
  </div>
```

```js
</div>
```

在我们的 JavaScript 中，我们现在将更新`setTimeout()`方法内部使用的代码。我们将从上一个代码块中存储的折叠组件放入一个变量中，并使用该变量通过`Collapse()`构造函数和`show()`方法来显示折叠组件。然后，将使用垂直过渡显示警报组件。以新闻通讯表单为例，更新后的 JavaScript 代码如下所示：

part-2/chapter-9/website/js/script.js 行 104-123

```js
// Newsletter form
```

```js
const newsletterForm = document.querySelector('#newsletterForm');
```

```js
newsletterForm.addEventListener('submit', function (event) {
```

```js
  if (!newsletterForm.checkValidity()) {
```

```js
    event.preventDefault();
```

```js
    event.stopPropagation();
```

```js
  } else {
```

```js
    event.preventDefault();
```

```js
    const newsletterButton = 
```

```js
      document.querySelector('#newsletterButton');
```

```js
    newsletterButton.innerHTML = spinnerMarkup;
```

```js
    newsletterButton.disabled = true;
```

```js
    setTimeout(function() {
```

```js
      const newsletterConfirmationText = 'Subscribed';
```

```js
      newsletterButton.innerHTML = 
```

```js
        newsletterConfirmationText;
```

```js
      const newsletterConfirmationElement = 
```

```js
        document.querySelector('#newsletterConfirmation');
```

```js
      new bootstrap.Collapse(
```

```js
        newsletterConfirmationElement).show();
```

```js
    }, 2000)
```

```js
  }
```

```js
  newsletterForm.classList.add('was-validated');
```

```js
}, false)
```

最后两个代码块以新闻通讯表单为例。联系表单和支付表单的代码类似。

# 创建程序化标签导航

目前，用户可以使用**购物车**页面上的标签导航来在三个标签之间导航。我们希望禁用用于标签导航的按钮，而是使用程序化标签导航。这意味着当用户通过每个表单中的提交按钮（分别标记为**下一步**、**下一步**和**支付**）通过结账流程时，活动标签面板将自动更改。

首先，我们需要更新我们的标记以禁用用于标签导航的按钮。对于每个按钮，我们将添加`.disabled`类以获得正确的样式，并移除`data-bs-toggle="tab"`属性以禁用点击时更改标签的功能。更新后的标记如下所示：

part-2/chapter-9/website/cart.xhtml

```js
<ul class="nav nav-pills nav-justified mb-5" role="tablist">
```

```js
  <li class="nav-item" role="presentation">
```

```js
    <button type="button" id="cartTabs-1" class="nav-link 
```

```js
      active disabled" data-bs-toggle="tab" data-bs-
```

```js
      target="#cartTabs-pane-1" role="tab" aria-
```

```js
      controls="cartTabs-pane-1" aria-selected="true">1\. 
```

```js
      Shopping Cart</button>
```

```js
  </li>
```

```js
  <li class="nav-item" role="presentation">
```

```js
    <button type="button" id="cartTabs-2" class="nav-link
```

```js
      disabled" data-bs-toggle="tab" data-bs-
```

```js
      target="#cartTabs-pane-2" role="tab" aria-
```

```js
      controls="cartTabs-pane-2" aria-selected="false">2\. 
```

```js
      Shipping Details</button>
```

```js
  </li>
```

```js
  <li class="nav-item" role="presentation">
```

```js
    <button type="button" id="cartTabs-3" class="nav-link
```

```js
      disabled" data-bs-toggle="tab" data-bs-
```

```js
      target="#cartTabs-pane-3" role="tab" aria-
```

```js
      controls="cartTabs-pane-3" aria-selected="false">3\. 
```

```js
      Payment Options</button>
```

```js
  </li>
```

```js
</ul>
```

在添加用于程序化更改标签的 JavaScript 之前，我们将创建一个辅助函数，用于通过平滑过渡将页面滚动到顶部。该辅助函数由以下代码组成：

part-2/chapter-9/website/js/script.js 行 154-160

```js
// Scroll effect for forms
```

```js
function scrollToTop() {
```

```js
  scroll({
```

```js
    top: 0,
```

```js
    behavior: "smooth"
```

```js
  });
```

```js
}
```

现在我们将看看如何创建用于程序化标签导航的 JavaScript。我们希望从**购物车**标签面板导航到**运输详情**标签面板，然后再从**运输详情**标签面板导航到**支付选项**标签面板。由于每个标签都需要单独激活，并且我们还需要为**购物车**和**运输详情**标签面板中的每个表单提供不同的行为，我们将逐一处理它们。

通过结账流程存储表单值

当我们切换到标签面板时，我们不会在结账流程中存储每个表单的输入值。这必须以使其适用于真实项目的方式完成。

## 购物车标签面板

在我们的 JavaScript 中，我们将等待表单的`submit`事件，然后阻止默认的表单提交。之后，我们将调用我们的辅助函数`scrollToTop()`，并添加一个延迟为`600`毫秒的`setTimeout()`方法。这样做是为了让浏览器有足够的时间在执行以下代码之前将页面滚动到顶部。

在`setTimeout()`方法内部，我们将存储第二个标签的按钮并将其存储在一个变量中，然后使用该变量通过`Tab()`构造函数和`show()`方法显示相应的标签页。标签按钮通过`data-bs-target`属性与相关的标签页相关联：

part-2/chapter-9/website/js/script.js 行 165-177

```js
// Shopping Cart form
```

```js
const shoppingCartForm = document.querySelector('#shoppingCartForm');
```

```js
shoppingCartForm.addEventListener('submit', function (event) {
```

```js
  event.preventDefault();
```

```js
  // Go to cart tab 2
```

```js
  scrollToTop();
```

```js
  setTimeout(function() {
```

```js
    const cartTabs2Element = 
```

```js
      document.querySelector('#cartTabs-2');
```

```js
    new bootstrap.Tab(cartTabs2Element).show();
```

```js
  }, 600)
```

```js
}, false)
```

## 发货详情标签页

与条件语句的`else`部分相比，这个表单的不同之处如下：

part-2/chapter-9/website/js/script.js 行 179-198

```js
// Shipping Details form
```

```js
const shippingForm = document.querySelector('#shippingForm');
```

```js
shippingForm.addEventListener('submit', function (event) {
```

```js
  if (!shippingForm.checkValidity()) {
```

```js
    event.preventDefault();
```

```js
    event.stopPropagation();
```

```js
  } else {
```

```js
    event.preventDefault();
```

```js
    // Go to cart tab 3
```

```js
    scrollToTop();
```

```js
    setTimeout(function() {
```

```js
const cartTabs3Element = 
```

```js
        document.querySelector('#cartTabs-3');
```

```js
      new bootstrap.Tab(cartTabs3Element).show();
```

```js
    }, 600)
```

```js
  }
```

```js
  shippingForm.classList.add('was-validated');
```

```js
}, false)
```

现在我们已经创建了程序化标签导航，可以继续添加一个功能来更新结账流程中的进度状态。

# 在结账流程中更新进度状态

我们想要改进的最后一个功能是在**购物车**页面上的结账流程，即当标签页切换时更新进度组件。在**购物车**标签页中，初始进度应为 0%，当切换到**发货详情**标签页时，进度应更新为 33%。当**发货详情**标签页上的表单提交并切换到**支付选项**标签页时，进度应更新为 67%。最后，当**支付选项**标签页上的表单提交并显示成功消息后，进度应更新为 100%。

要更新进度组件，我们需要更改`.progress-bar`元素（进度组件的子元素）的`width`属性和`aria-valuenow`属性。`aria-valuenow`属性的值将直接通过 JavaScript 更新，而`width`属性的值可以通过内联样式直接更新或通过类间接更新。我们将探讨这两种方法。

## 使用内联样式更新进度

对于这种方法，首先，我们将使用内联样式更新我们的标记`width`属性。我们将将其设置为初始值`0`，这在我们要从一个值过渡到另一个值时是必要的。如果事先不将`width`设置为`0`，则`.progress-bar`元素的宽度将从一个值“跳”到另一个值。以下是更新的代码：

part-2/chapter-9/website/cart.xhtml

```js
<div class="progress mb-4" id="cartProgress">
```

```js
  <div class="progress-bar progress-bar-striped progress-
```

```js
    bar-animated" style="width: 0;" role="progressbar" 
```

```js
    aria-valuenow="00" aria-valuemin="0" aria-
```

```js
    valuemax="100"></div>
```

```js
</div>
```

在我们的 JavaScript 中，首先，我们将进度组件的`.progress-bar`元素存储在一个变量中，如下所示：

part-2/chapter-9/website/js/script.js 行 162-163

```js
// Progress component which is animated by the tabs 
```

```js
// navigation below
```

```js
const cartProgressBar = document.querySelector('#cartProgress .progress-bar');
```

然后，我们将在购物页面上每个表单的`setTimeout()`方法末尾添加两行 JavaScript 代码。这将更新内联样式和`aria-valuenow`属性的值。更新的代码如下，以**购物车**表单为例：

part-2/chapter-9/website/js/script.js 行 165-177

```js
// Shopping Cart form
```

```js
const shoppingCartForm = document.querySelector('#shoppingCartForm');
```

```js
shoppingCartForm.addEventListener('submit', function (event) {
```

```js
  event.preventDefault();
```

```js
  // Go to cart tab 2
```

```js
  scrollToTop();
```

```js
  setTimeout(function() {
```

```js
    const cartTabs2Element = 
```

```js
      document.querySelector('#cartTabs-2');
```

```js
    new bootstrap.Tab(cartTabs2Element).show();
```

```js
    cartProgressBar.style.width = "33%";
```

```js
    cartProgressBar.ariaValueNow = 33;
```

```js
  }, 600)
```

```js
}, false)
```

我们不会使用内联样式的方法。这是因为我们想要利用宽度实用类`.w-33`和`.w-67`，这些是我们使用实用 API 在自定义网站样式时生成的。我们不会使用内联样式。

## 使用类更新进度

为了使这种方法有效，我们还需要明确指定`.progress-element`的宽度，以便在添加宽度实用类时能够获得过渡效果。我们可以在我们的标记中保留内联样式，但由于我们想要避免使用内联样式，因此我们将宽度指定在我们的自定义样式表中，如下所示：

part-2/chapter-9/website/scss/_custom-styles.scss 行 17-20

```js
// Set the width of the .progress-bar so we can use the 
```

```js
// transition effect when adding width utility classes
```

```js
.progress-bar {
```

```js
  width: 0;
```

```js
}
```

现在，在我们的 JavaScript 中，我们将添加和移除宽度实用类`.w-33`、`.w-67`和`.w-100`，以实现之前相同的效果。更新`aria-valuenow`属性的代码将保持不变，并且代码仍然放置在`setTimeout()`方法的末尾。它将以以下三种不同的形式出现：

part-2/chapter-9/website/js/script.js 行 174-175, 行 192-194, 行 216-218

```js
// Shopping Cart form
```

```js
cartProgressBar.classList.add("w-33");
```

```js
cartProgressBar.ariaValueNow = 33;
```

```js
// Shipping Details form
```

```js
cartProgressBar.classList.remove("w-33");
```

```js
cartProgressBar.classList.add("w-67");
```

```js
cartProgressBar.ariaValueNow = 67;
```

```js
// Payment Options form
```

```js
cartProgressBar.classList.remove("w-67");
```

```js
cartProgressBar.classList.add("w-100");
```

```js
cartProgressBar.ariaValueNow = 100;
```

我们想要做的最后一件事是在进度组件达到 100%时停止动画。为了做到这一点，我们只需在从 67%到 100%的过渡完成后从`.progress-bar`元素中移除`.progress-bar-animated`类即可。在前一章中，当我们使用 Bootstrap 5 变量自定义网站时，我们使用`_default-variable-overrides.scss`文件中的`$progress-bar-animation-timing`变量将`.progress-bar`元素的动画持续时间设置为 0.5 秒。因此，我们将移除`.progress-bar-animated`类的代码放在一个延迟为`500`毫秒的`setTimeout()`方法中，如下所示：

part-2/chapter-9/website/js/script.js 行 219-221

```js
setTimeout(function() {
```

```js
  cartProgressBar.classList.remove("progress-bar-
```

```js
    animated");
```

```js
}, 500)
```

现在我们已经完成了为购物车页面添加交互功能的工作，我们只剩下一个交互功能需要创建。

# 为产品画廊创建灯箱

我们想要为网站添加的最后一个交互功能是在**产品**页面上的产品画廊中添加一个灯箱。我们将简单地使用模态组件，当点击其中一个缩略图图像时，显示一个大型图像。

首先，我们将在`product.xhtml`文件的底部插入模态的最小代码，在`</footer>`标签之后和两个`<script>`标签之前。我们将使用`.modal-xl`和`.modal-dialog-centered`修饰类使模态框变大并垂直居中。由于我们不希望在模态框中添加任何填充或其他 UI 元素，因此我们不会有任何模态头部、模态主体或模态页脚。我们只想显示我们的大图像以及围绕它的模态背景。仍然可以通过按*Escape*键或点击背景的任何位置来关闭模态框。`<img>`标签上的类将被用作 JavaScript 的钩子：

part-2/chapter-9/website/product.xhtml 行 408-414

```js
<div class="modal fade" id="productGallery" tabindex="-1" aria-hidden="true">
```

```js
  <div class="modal-dialog modal-xl modal-dialog-centered">
```

```js
    <div class="modal-content">
```

```js
      <img class="js-productGalleryImage">
```

```js
    </div>
```

```js
  </div>
```

```js
</div>
```

现在，我们需要更新围绕缩略图的链接的标记。将使用一个类作为我们的 JavaScript 的钩子，两个数据属性将用于初始化模态组件，一个数据属性将用于存储我们想在模态中显示的大图的 URL。第一个缩略图图像的更新代码将如下所示：

part-2/chapter-9/website/product.xhtml 行 85

```js
<a href="#" class="js-productGalleryThumbnail" data-bs-toggle="modal" data-bs-target="#productGallery" data-src="img/1600x900.png"><img src="img/185x104.png" class="img-thumbnail" alt="Product image"></a>
```

在我们的 JavaScript 中，首先，我们将使用我们刚刚添加的类作为钩子，将产品画廊中的所有图像缩略图存储在一个变量中，并使用我们之前提供的类作为钩子将模态中的空图像存储在一个变量中。然后，我们将遍历图像缩略图并监听`click`事件。在点击事件中，我们将`<img>`标签的空`src`属性设置为图像缩略图上定义的`data-src`属性值。由于此代码将在模态触发时同时执行，因此现在将在模态内部显示大图像。下次点击图像缩略图时，`src`属性将被替换为`data-src`属性的新值。此 JavaScript 代码如下所示：

part-2/chapter-9/website/js/script.js 行 231-238

```js
// Handle button click and open product gallery with 
```

```js
// specific "src" value
```

```js
const productGalleryThumbnails = document.querySelectorAll('.js-productGalleryThumbnail');
```

```js
const productGalleryImage = document.querySelector('.js-productGalleryImage');
```

```js
for (let i = 0; i < productGalleryThumbnails.length; i++) {
```

```js
  productGalleryThumbnails[i].addEventListener('click', 
```

```js
    function() {
```

```js
    productGalleryImage.src = 
```

```js
      productGalleryThumbnails[i].dataset.src;
```

```js
  })
```

```js
}
```

现在我们已经为产品画廊创建了自己的灯箱，并且已经完成了向我们的网站添加交互式功能的工作。

# 摘要

在本章中，我们学习了如何使用 JavaScript 通过交互式功能来改进我们的网站。其中一些功能基于需要自定义 JavaScript 才能工作的 Bootstrap 5 组件，而其他功能则是在不使用 Bootstrap 5 组件的情况下创建的。

在浏览本章的所有场景时，我们看到了如何通过添加这些新的交互式功能使网站更具吸引力和互动性。

本章总结了本书的第二部分，这部分内容主要围绕如何仅使用 Bootstrap 5 创建一个网站，然后通过 Sass 和 JavaScript 对其进行定制和改进。在第三部分，我们将揭示更多使用 Sass 和 CSS 功能与 Bootstrap 5 一起工作的方法，并学习如何优化 Bootstrap 5 代码和文件的使用。
