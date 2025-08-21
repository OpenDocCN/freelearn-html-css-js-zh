# 附录 A. 检测 Canvas 支持

# Canvas 回退内容

由于所有浏览器都不支持 canvas，因此最好提供回退内容，以便用户知道如果他们选择的浏览器不支持 canvas，则某些功能无法正常工作。处理不支持 canvas 的浏览器最简单和最直接的技术是在 canvas 标签内添加回退内容。通常，这个内容将是文本或图像，告诉用户他们过时的浏览器不支持 canvas，并建议下载一个在本年代开发的浏览器。使用支持 canvas 的浏览器的用户将看不到内部内容：

```js
<canvas id="myCanvas" width="578" height="250">
            Yikes!  Your browser doesn't support canvas.  Try using 
Google Chrome or Firefox instead.
</canvas>
```

Canvas 回退内容并不总是最好的解决方案。例如，如果浏览器不支持 canvas，您可能希望警告错误消息，将用户重定向到不同的 URL，甚至使用应用程序的 Flash 版本作为回退。检测浏览器是否支持 canvas 的最简单方法是创建一个虚拟 canvas 元素，然后检查我们是否可以执行 getContext 方法：

```js
function isCanvasSupported(){
            return !!document.createElement('canvas').getContext;
        }
```

当页面加载时，我们可以调用 isCanvasSupported()函数来确定浏览器是否支持 canvas，然后适当处理结果。

这个函数使用了我最喜欢的 JavaScript 技巧之一，双非技巧（!!），它确定了 getContext 方法是否成功执行。双非的第一个非将数据类型强制转换为布尔值。由于强制转换数据类型的行为产生了我们不想要的相反结果，我们可以添加第二个非（!!）来翻转结果。双非技巧是检查一段代码是否成功执行的一种非常方便的方式，在我看来，它比用 try/catch 块包装一行代码要优雅得多。

## 检测可用的 WebGL 上下文

如果您的 canvas 应用程序利用了 WebGL，您可能还想知道浏览器支持哪些上下文，以便您可以成功初始化一个 WebGL 应用程序。

在撰写本文时，有五个主要的上下文：

+   2D

+   webgl

+   实验性的 WebGL

+   moz-webgl

+   webkit-3d

包括 Google Chrome，Firefox，Safari，Opera 和 IE9 在内的所有主要浏览器都支持 2D 上下文。然而，当涉及到 WebGL 支持时，情况完全不同。在撰写本文时，Google Chrome 和 Safari 支持实验性的 WebGL 和 webkit-3d 上下文，Firefox 支持实验性的 WebGL 和 moz-webgl 上下文，IE9 不支持任何形式的 WebGL。

要自己看到这一点，您可以创建一个名为 getCanvasSupport()的函数，该函数循环遍历所有可能的上下文，并使用双非技巧来确定哪些上下文是可用的：

```js
function getCanvasSupport(){
    // initialize return object
    var returnObj = {
        canvas: false,
        webgl: false,
        context_2d: false,
        context_webgl: false,
        context_experimental_webgl: false,
        context_moz_webgl: false,
        context_webkit_3d: false
    };
    // check if canvas is supported
    if (!!document.createElement('canvas').getContext) {
        returnObj.canvas = true;
    }

    // check if WebGL rendering context is supported
    if (window.WebGLRenderingContext) {
        returnObj.webgl = true;
    }

    // check specific contexts
    var contextMapping = {
        context_2d: "2d",
        context_webgl: "webgl",
        context_experimental_webgl: "experimental-webgl",
        context_moz_webgl: "moz-webgl",
        context_webkit_3d: "webkit-3d"
    };

    for (var key in contextMapping) {
        try {
            if (!!document.createElement('canvas').getContext(contextMapping[key])) {
                returnObj[key] = true;
            }
        } 
        catch (e) {
        }
    }

    return returnObj;
}

function showSupport(obj){
    var str = "";

    str += "-- General Support --<br>";
    str += "canvas: " + (obj.canvas ? "YES" : "NO") + "<br>";
    str += "webgl: " + (obj.webgl ? "YES" : "NO") + "<br>";

    str += "<br>-- Successfully Initialized Contexts --<br>";
    str += "2d: " + (obj.context_2d ? "YES" : "NO") + "<br>";
    str += "webgl: " + (obj.context_webgl ? "YES" : "NO") + "<br>";
    str += "experimental-webgl: " + (obj.context_experimental_webgl ? "YES" : "NO") + "<br>";
    str += "moz-webgl: " + (obj.context_moz_webgl ? "YES" : "NO") + "<br>";
    str += "webkit-3d: " + (obj.context_webkit_3d ? "YES" : "NO") + "<br>";

    document.write(str);
}

window.onload = function(){
    showSupport(getCanvasSupport());
};
```
