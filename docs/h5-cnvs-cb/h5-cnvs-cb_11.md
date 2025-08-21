# 附录 B. 画布安全

为了保护网站上图像、视频和画布的像素数据，HTML5 画布规范中有防护措施，防止其他域的脚本访问这些媒体，操纵它们，然后创建新的图像、视频或画布。

在画布上绘制任何内容之前，画布标签的 origin-clean 标志设置为 true。这基本上意味着画布是“干净的”。如果您在托管代码运行的同一域上的画布上绘制图像，则 origin-clean 标志保持为 true。但是，如果您在托管在另一个域上的画布上绘制图像，则 origin-clean 标志将设置为 false，画布现在是“脏的”。

根据 HTML5 画布规范，一旦发生以下任何操作，画布就被视为脏：

+   - 调用元素的 2D 上下文的`drawImage()`方法时，使用的`HTMLImageElement`或`HTMLVideoElement`的原点与拥有画布元素的文档对象不同。

+   - 调用元素的 2D 上下文的`drawImage()`方法时，使用的`HTMLCanvasElement`的 origin-clean 标志为 false。

+   - 元素的 2D 上下文的`fillStyle`属性设置为从`HTMLImageElement`或`HTMLVideoElement`创建的`CanvasPattern`对象，当时该模式的原点与拥有画布元素的文档对象不同。

+   - 元素的 2D 上下文的`fillStyle`属性设置为从`HTMLCanvasElement`创建的`CanvasPattern`对象，当时该模式的 origin-clean 标志为 false。

+   元素的 2D 上下文的`strokeStyle`属性设置为从`HTMLImageElement`或`HTMLVideoElement`创建的`CanvasPattern`对象，当时该模式的原点与拥有画布元素的文档对象不同。

+   - 元素的 2D 上下文的`strokeStyle`属性设置为从`HTMLCanvasElement`创建的`CanvasPattern`对象，当时该模式的 origin-clean 标志为 false。

+   - 调用元素的 2D 上下文的`fillText()`或`strokeText()`方法，并考虑使用原点与拥有画布元素的文档对象不同的字体。（甚至不必使用字体；重要的是字体是否被用于绘制任何字形。）

- 此外，如果您在本地计算机上执行以下任何操作（而不是在 Web 服务器上），则 origin-clean 标志将自动设置为 false，因为资源将被视为来自不同的来源。

接下来，根据规范，如果在脏画布上发生以下任何操作，将抛出`SECURITY_ERR`异常：

+   - 调用`toDataURL()`方法

+   - 调用`getImageData()`方法

+   - 使用`measureText()`方法时，使用的字体的原点与文档对象不同

尽管画布安全规范是出于良好意图创建的，但它可能会给我们带来更多麻烦。举个例子，假设您想创建一个绘图应用程序，该应用程序可以连接到 Flickr API，从公共域中获取图像以添加到您的绘图中。如果您希望您的应用程序能够使用`toDataURL()`方法将该绘图保存为图像，或者如果您希望您的应用程序使用`getImageData()`方法具有复杂的像素操作算法，那么您将遇到一些麻烦。在脏画布上执行这些操作将抛出 JavaScript 错误，并阻止您的应用程序正常工作。

解决这个问题的一种方法是创建一个代理，从另一个域获取图像，然后传递回客户端，使其看起来像图像来自您的域。如果您曾经使用过跨域 AJAX 应用程序，您会感到非常熟悉。
