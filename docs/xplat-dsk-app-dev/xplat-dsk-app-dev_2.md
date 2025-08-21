# 第二章：使用 NW.js 创建文件资源管理器-增强和交付

好了，我们有一个可以用于浏览文件系统并使用默认关联程序打开文件的文件资源管理器的工作版本。现在我们将扩展它以进行其他文件操作，比如删除和复制粘贴。这些选项将保留在动态构建的上下文菜单中。我们还将考虑 NW.js 在不同应用程序之间使用系统剪贴板传输数据的能力。我们将使应用程序响应命令行选项。我们还将提供对多种语言和区域设置的支持。我们将通过将其编译成本机代码来保护源代码。我们将考虑打包和分发。最后，我们将建立一个简单的发布服务器，并使文件资源管理器自动更新。

# 国际化和本地化

**国际化**，通常缩写为**i18n**，意味着一种特定的软件设计，能够适应目标本地市场的要求。换句话说，如果我们想将我们的应用程序分发到美国以外的市场，我们需要关注翻译、日期时间、数字、地址等的格式化。

# 按国家格式化日期

国际化是一个横切关注点。当您更改区域设置时，通常会影响多个模块。因此，我建议使用我们在处理`DirService`时已经检查过的观察者模式：

`./js/Service/I18n.js`

```js
const EventEmitter = require( "events" ); 

class I18nService extends EventEmitter { 
  constructor(){ 

   super(); 
   this.locale = "en-US"; 
  } 
  notify(){ 
   this.emit( "update" ); 
  } 
} 

exports.I18nService = I18nService;

```

正如您所看到的，我们可以通过为`locale`属性设置新值来更改`locale`属性。一旦我们调用`notify`方法，所有订阅的模块立即做出响应。

然而，`locale`是一个公共属性，因此我们无法控制其访问和变异。我们可以使用重载来修复它：

`./js/Service/I18n.js`

```js
//... 
  constructor(){ 
   super(); 
   this._locale = "en-US"; 
  } 
  get locale(){ 

  return this._locale; 
  } 
  set locale( locale ){ 
   // validate locale... 
   this._locale = 

locale; 
  } 
  //...

```

现在，如果我们访问`I18n`实例的`locale`属性，它将通过 getter（`get locale`）传递。当设置它的值时，它将通过 setter（`set locale`）传递。因此，我们可以添加额外的功能，比如在属性访问和变异时进行验证和记录。

请记住，我们在 HTML 中有一个用于选择语言的组合框。为什么不给它一个视图呢？

`./js/View/LangSelector.js`：

```js
class LangSelectorView { 
  constructor( boundingEl, i18n ){ 
   boundingEl.addEventListener( "change", 

this.onChanged.bind( this ), false ); 
   this.i18n = i18n; 
  } 
   onChanged( e ){ 
   const selectEl 

= e.target; 
   this.i18n.locale = selectEl.value; 
   this.i18n.notify(); 
  } 
} 

exports.LangSelectorView = LangSelectorView;

```

在上述代码中，我们监听组合框的更改事件。

当事件发生时，我们使用传入的`I18n`实例更改`locale`属性，并调用`notify`通知订阅者：

`./js/app.js`

```js
const i18nService = new I18nService(), 
      { LangSelectorView } = require( "./js/View/LangSelector" ); 

new LangSelectorView( document.querySelector( "[data-bind=langSelector]" ), i18nService );

```

好了，我们可以更改区域设置并触发事件。那么消费模块呢？

在`FileList`视图中，我们有`formatTime`静态方法，用于格式化传递的`timeString`以进行打印。我们可以使其根据当前选择的`locale`进行格式化：

`./js/View/FileList.js`：

```js
constructor( boundingEl, dirService, i18nService ){ 
    //... 
    this.i18n = i18nService; 
    // 

Subscribe on i18nService updates 
          i18nService.on( "update", () => this.update( dirService.getFileList() ) 

); 
  } 
  static formatTime( timeString, locale ){ 
   const date = new Date( Date.parse( timeString ) ), 

         options = { 
         year: "numeric", month: "numeric", day: "numeric", 
         hour: 

"numeric", minute: "numeric", second: "numeric", 
         hour12: false 
         }; 
   return 

date.toLocaleString( locale, options ); 
  } 
 update( collection ) { 
        //... 

this.el.insertAdjacentHTML( "beforeend", `<li class="file-list__li" data-file="${fInfo.fileName}"> 

<span class="file-list__li__name">${fInfo.fileName}</span> 
         <span class="file-

list__li__size">${filesize(fInfo.stats.size)}</span> 
         <span class="file-list__li__time">

${FileListView.formatTime( fInfo.stats.mtime, this.i18n.locale )}</span> 
   </li>` ); 
        //... 

  } 
//...

```

在构造函数中，我们订阅`I18n`更新事件，并在区域设置更改时更新文件列表。`formatTime`静态方法将传递的字符串转换为`Date`对象，并使用`Date.prototype.toLocaleString()`方法根据给定的区域设置格式化日期时间。这个方法属于所谓的**ECMAScript 国际化 API**（[`norbertlindenberg.com/2012/12/ecmascript-internationalization-api/index.html`](http://norbertlindenberg.com/2012/12/ecmascript-internationalization-api/index.html)）。这个 API 描述了内置对象--`String`、`Date`和`Number`--的方法，旨在格式化和比较本地化数据。然而，它真正做的是使用`toLocaleString`为英语（美国）区域设置（`en-US`）格式化`Date`实例，并返回日期，如下：

```js
3/17/2017, 13:42:23

```

然而，如果我们将德国区域设置（`de-DE`）传递给该方法，我们会得到完全不同的结果：

```js
17.3.2017, 13:42:23

```

为了付诸实践，我们给组合框设置了一个标识符。`./index.html`文件包含以下代码：

```js
.. 
<select class="footer__select" data-bind="langSelector"> 
..

```

当然，我们必须创建一个`I18n`服务的实例，并将其传递给`LangSelectorView`和`FileListView`：

`./js/app.js`

```js
// ... 
const { I18nService } = require( "./js/Service/I18n" ), 
   { LangSelectorView } = require( 

"./js/View/LangSelector" ), 
   i18nService = new I18nService(); 

new LangSelectorView( 

document.querySelector( "[data-bind=langSelector]" ), i18nService ); 
// ... 
new FileListView( 

document.querySelector( "[data-bind=fileList]" ), dirService, i18nService );

```

现在我们将启动应用程序。是的！当我们在组合框中更改语言时，文件修改日期会相应调整：

![](img/97388f9b-0432-45bf-9777-dd650d980998.png)

# 多语言支持

本地化日期和数字是一件好事，但为多种语言提供翻译将更加令人兴奋。我们的应用程序中有许多术语，即文件列表的列标题和窗口操作按钮上的工具提示（通过`title`属性）。我们需要的是一个字典。通常，它包含了映射到语言代码或区域设置的令牌翻译对的集合。因此，当您从翻译服务请求一个术语时，它可以与当前使用的语言/区域设置相匹配的翻译相关联。

在这里，我建议将字典作为一个静态模块，可以通过所需的函数加载：

`./js/Data/dictionary.js`

```js
exports.dictionary = { 
  "en-US": { 
    NAME: "Name", 
    SIZE: "Size", 
    MODIFIED: 

"Modified", 
    MINIMIZE_WIN: "Minimize window", 
    RESTORE_WIN: "Restore window", 
    MAXIMIZE_WIN: 

"Maximize window", 
    CLOSE_WIN: "Close window" 
  }, 
  "de-DE": { 
    NAME: "Dateiname", 

SIZE: "Grösse", 
    MODIFIED: "Geändert am", 
    MINIMIZE_WIN: "Fenster minimieren", 

RESTORE_WIN: "Fenster wiederherstellen", 
    MAXIMIZE_WIN: "Fenster maximieren", 
    CLOSE_WIN: "Fenster 

schliessen" 
  } 
};

```

因此，我们有两个翻译的术语。我们将字典作为依赖项注入到我们的`I18n`服务中：

`./js/Service/I18n.js`

```js
//... 
constructor( dictionary ){ 
    super(); 
    this.dictionary = dictionary; 

this._locale = "en-US"; 
 } 

translate( token, defaultValue ) { 
    const dictionary = 

this.dictionary[ this._locale ]; 
    return dictionary[ token ] || defaultValue; 
} 
//...

```

我们还添加了一个新方法`translate`，它接受两个参数：`token`和`default`翻译。第一个参数可以是字典中的键之一，比如`NAME`。第二个参数是在字典中请求的 token 尚不存在时的默认值。因此，我们至少可以得到一个有意义的文本，至少是英文。

让我们看看如何使用这个新方法：

`./js/View/FileList.js`

```js
//... 
update( collection ) { 
    this.el.innerHTML = `<li class="file-list__li file-list__head"> 
        <span class="file-list__li__name">${this.i18n.translate( "NAME", "Name" )}</span> 

<span class="file-list__li__size">${this.i18n.translate( "SIZE", "Size" )}</span> 
        <span 

class="file-list__li__time">${this.i18n.translate( "MODIFIED", "Modified" )}</span> 
      </li>`; 
//...

```

我们用`I18n`实例的`translate`方法来更改`FileList`视图中的硬编码列标题，这意味着每次视图更新时，它都会接收到实际的翻译。我们也不要忘记`TitleBarActions`视图，那里有窗口操作按钮：

`./js/View/TitleBarActions.js`

```js
constructor( boundingEl, i18nService ){ 
  this.i18n = i18nService; 
  //... 
  // Subscribe on 

i18nService updates 
  i18nService.on( "update", () => this.translate() ); 
} 

translate(){ 

 this.unmaximizeEl.title = this.i18n.translate( "RESTORE_WIN", "Restore window" ); 
  this.maximizeEl.title = 

this.i18n.translate( "MAXIMIZE_WIN", "Maximize window" ); 
  this.minimizeEl.title = this.i18n.translate( 

"MINIMIZE_WIN", "Minimize window" ); 
  this.closeEl.title = this.i18n.translate( "CLOSE_WIN", "Close window" ); 
}

```

在这里，我们添加了`translate`方法，它会使用实际的翻译更新按钮标题属性。我们订阅`i18n`更新事件，以便在用户更改`locale`时调用该方法：

![](img/08faa557-defe-45b5-be0d-1d505d9f6bd3.png)

# 上下文菜单

好吧，通过我们的应用程序，我们已经可以浏览文件系统并打开文件，但是人们可能希望文件资源管理器有更多功能。我们可以添加一些与文件相关的操作，比如删除和复制/粘贴。通常，这些任务可以通过上下文菜单来完成，这给了我们一个很好的机会来研究如何在`NW.js`中实现。通过环境集成 API，我们可以创建系统菜单的实例（[`docs.nwjs.io/en/latest/References/Menu/`](http://docs.nwjs.io/en/latest/References/Menu/)）。然后，我们组合表示菜单项的对象，并将它们附加到菜单实例上（[`docs.nwjs.io/en/latest/References/MenuItem/`](http://docs.nwjs.io/en/latest/References/MenuItem/)）。这个`menu`可以在任意位置显示：

```js
const menu = new nw.Menu(), 
      menutItem = new nw.MenuItem({ 
        label: "Say hello", 

click: () => console.log( "hello!" ) 
      }); 

menu.append( menu ); 
menu.popup( 10, 10 );

```

然而，我们的任务更具体。我们必须在鼠标右键单击时在光标位置显示菜单，为了实现这一点，我们通过订阅`contextmenu` DOM 事件来实现：

```js
document.addEventListener( "contextmenu", ( e ) => { 
   console.log( `Show menu in position ${e.x}, ${e.y}` 
);   
});

```

现在，每当我们在应用程序窗口内右键单击时，菜单就会显示出来。这并不完全是我们想要的，是吗？我们只需要在光标停留在特定区域时才显示菜单，例如当它悬停在文件名上时。这意味着我们必须测试目标元素是否符合我们的条件：

```js
document.addEventListener( "contextmenu", ( e ) => { 
   const el = e.target; 
   if ( el instanceof 

HTMLElement && el.parentNode.dataset.file ) { 
     console.log( `Show menu in position ${e.x}, ${e.y}` );   

   } 
});

```

在这里，我们忽略事件，直到光标悬停在文件表行的任何单元格上，因为每一行都是由`FileList`视图生成的列表项，并且为数据文件属性提供了一个值。

这段话基本上解释了如何构建系统菜单以及如何将其附加到文件列表上。然而，在开始创建一个能够创建菜单的模块之前，我们需要一个处理文件操作的服务：

`./js/Service/File.js`

```js
const fs = require( "fs" ), 
      path = require( "path" ), 
      // Copy file helper 
      cp = ( 

from, toDir, done ) => { 
        const basename = path.basename( from ), 
              to = path.join( 

toDir, basename ), 
              write = fs.createWriteStream( to ) ; 

        fs.createReadStream( from 

) 
          .pipe( write ); 

        write 
          .on( "finish",  done ); 
      }; 

class FileService { 

  constructor( dirService ){ 
    this.dir = dirService; 

this.copiedFile = null; 
  } 

  remove( file ){ 
    fs.unlinkSync( this.dir.getFile( file ) ); 
    this.dir.notify(); 
  } 

  paste(){ 
    const file = this.copiedFile; 
    if ( 

fs.lstatSync( file ).isFile() ){ 
      cp( file, this.dir.getDir(), () => this.dir.notify() ); 
    } 

} 

  copy( file ){ 
    this.copiedFile = this.dir.getFile( file ); 
  }  

  open( file 

){ 
    nw.Shell.openItem( this.dir.getFile( file ) ); 
  } 

  showInFolder( file ){ 

nw.Shell.showItemInFolder( this.dir.getFile( file ) ); 
  } 
}; 

exports.FileService = 

FileService;

```

这里发生了什么？`FileService`接收`DirService`的实例作为构造函数参数。它使用该实例通过名称获取文件的完整路径（`this.dir.getFile(file)`）。它还利用实例的`notify`方法请求所有订阅`DirService`的视图更新。`showInFolder`方法调用`nw.Shell`的相应方法，在系统文件管理器中显示文件的父文件夹。正如你所料，`remove`方法删除文件。至于复制/粘贴，我们做了以下技巧。当用户点击复制时，我们将目标文件路径存储在`copiedFile`属性中。因此，当用户下次点击粘贴时，我们可以使用它将该文件复制到可能已更改的当前位置。`open`方法显然使用默认关联程序打开文件。这就是我们在`FileList`视图中直接做的。实际上，这个操作属于`FileService`。因此，我们调整视图以使用该服务：

`./js/View/FileList.js`

```js
constructor( boundingEl, dirService, i18nService, fileService ){ 
   this.file = fileService; 
   //... 
} 
bindUi(){ 
  //... 
  this.file.open( el.dataset.file ); 
  //... 
}

```

现在，我们有一个模块来处理所选文件的上下文菜单。该模块将订阅`contextmenu`DOM 事件，并在用户右键单击文件时构建菜单。此菜单将包含在文件夹中显示项目、复制、粘贴和删除。复制和粘贴与其他项目分隔开，并且在我们存储了复制文件之前，粘贴将被禁用：

`./js/View/ContextMenu.js`

```js
class ConextMenuView { 
  constructor( fileService, i18nService ){ 
    this.file = fileService; 

this.i18n = i18nService; 
    this.attach(); 
  } 

  getItems( fileName ){ 
    const file = 

this.file, 
          isCopied = Boolean( file.copiedFile ); 

    return [ 
      { 

label: this.i18n.translate( "SHOW_FILE_IN_FOLDER", "Show Item in the 
                                                          Folder" ), 
        enabled: Boolean( fileName ), 

        click: () => file.showInFolder( fileName ) 
      }, 
      { 
        type: "separator" 

      }, 
      { 
        label: this.i18n.translate( "COPY", "Copy" ), 
        enabled: Boolean( 

              fileName ), 
        click: () => file.copy( fileName ) 
      }, 
      { 
        label: 

this.i18n.translate( "PASTE", "Paste" ), 
        enabled: isCopied, 
        click: () => file.paste() 

     }, 
      { 
        type: "separator" 
      }, 
      { 
        label: 

this.i18n.translate( "DELETE", "Delete" ), 
        enabled: Boolean( fileName ), 
        click: () => 

file.remove( fileName ) 
      } 
    ]; 
  } 

  render( fileName ){ 
    const menu = new 

nw.Menu(); 
    this.getItems( fileName ).forEach(( item ) => menu.append( new  
                                            nw.MenuItem( item ))); 

return menu; 
  } 

  attach(){ 
    document.addEventListener( "contextmenu", ( e ) => { 

  const el = e.target; 
      if ( !( el instanceof HTMLElement ) ) { 
        return; 
      } 

      if ( el.classList.contains( "file-list" ) ) { 
        e.preventDefault(); 
        this.render() 

        .popup( e.x, e.y ); 
      } 
      // If a child of an element matching [data-file] 
      if ( 

el.parentNode.dataset.file ) { 
        e.preventDefault(); 
        this.render( el.parentNode.dataset.file ) 

          .popup( e.x, e.y ); 
      } 

    }); 
  } 
} 

exports.ConextMenuView = ConextMenuView;

```

因此，在`ConextMenuView`构造函数中，我们接收`FileService`和`I18nService`的实例。在构造过程中，我们还调用`attach`方法，该方法订阅`contextmenu`DOM 事件，创建菜单，并在鼠标光标的位置显示它。除非光标悬停在文件上或停留在文件列表组件的空白区域中，否则事件将被忽略。当用户右键单击文件列表时，菜单仍然会出现，但除了粘贴（如果之前复制了文件）之外，所有项目都会被禁用。`render`方法创建菜单的实例，并使用`getItems`方法创建的`nw.MenuItems`填充它。该方法创建表示菜单项的数组。数组的元素是对象文字。`label`属性接受项目标题的翻译。`enabled`属性根据我们的情况定义项目的状态（我们是否持有复制的文件）。最后，`click`属性期望点击事件的处理程序。

现在我们需要在主模块中启用我们的新组件：

`./js/app.js`

```js
const { FileService } = require( "./js/Service/File" ), 
      { ConextMenuView } = require( 

"./js/View/ConextMenu" ), 
      fileService = new FileService( dirService ); 

new FileListView( 

document.querySelector( "[data-bind=fileList]" ), dirService, i18nService, fileService ); 
new ConextMenuView( 

fileService, i18nService );

```

现在，让我们运行应用程序，在文件上右键单击，哇！我们有上下文菜单和新文件操作：

![](img/8d5747ae-172d-4c7f-9b2c-7d0ad5c95d3d.png)

# 系统剪贴板

通常，复制/粘贴功能涉及系统剪贴板。`NW.js`提供了一个 API 来控制它（[`docs.nwjs.io/en/latest/References/Clipboard/`](http://docs.nwjs.io/en/latest/References/Clipboard/)）。不幸的是，它相当有限；我们无法在应用程序之间传输任意文件，这可能是您对文件管理器的期望。然而，对我们来说仍然有一些事情是可用的。

# 传输文本

为了检查使用剪贴板传输文本，我们修改了`FileService`的`copy`方法：

```js
copy( file ){ 
    this.copiedFile = this.dir.getFile( file ); 
    const clipboard = nw.Clipboard.get(); 

    clipboard.set( this.copiedFile, "text" ); 
}

```

它是做什么的？一旦我们获得文件的完整路径，我们创建一个`nw.Clipboard`的实例，并将文件路径保存为文本。因此，现在在文件资源管理器中复制文件后，我们可以切换到外部程序（例如文本编辑器）并从剪贴板中粘贴复制的路径：

![](img/d76eb224-6217-4723-af99-7e6005321ec7.png)

# 传输图形

看起来不太方便，是吗？如果我们能复制/粘贴一个文件会更有趣。不幸的是，`NW.js`在文件交换方面并没有给我们太多选择。然而，我们可以在`NW.js`应用程序和外部程序之间传输 PNG 和 JPEG 图像：

`./js/Service/File.js`

```js
//... 
  copyImage( file, type ){ 
    const clip = nw.Clipboard.get(), 
          // load file content 

as Base64 
          data = fs.readFileSync( file ).toString( "base64" ), 
          // image as HTML 

    html = `<img src="img/, "" ) )}">`; 

    // write both options 

(raw image and HTML) to the clipboard 
    clip.set([ 
      { type, data: data, raw: true }, 
      { type: 

"html", data: html } 
    ]); 
  } 

  copy( file ){ 
    this.copiedFile = this.dir.getFile( 

file ); 
    const ext = path.parse( this.copiedFile ).ext.substr( 1 ); 
    switch ( ext ){ 
      case 

"jpg": 
      case "jpeg": 
        return this.copyImage( this.copiedFile, "jpeg" ); 
      case "png": 
        return this.copyImage( this.copiedFile, "png" ); 
    } 
  } 
//...

```

我们用`copyImage`私有方法扩展了我们的`FileService`。它读取给定的文件，将其内容转换为 Base64，并将结果代码传递给剪贴板实例。此外，它创建了一个包含 Base64 编码图像的图像标签的 HTML，其中包含数据**统一资源标识符**（**URI**）。现在，在文件资源管理器中复制图像（PNG 或 JPEG）后，我们可以将其粘贴到外部程序中，例如图形编辑器或文本处理器。

# 接收文本和图形

我们已经学会了如何将文本和图形从我们的`NW.js`应用程序传递到外部程序，但是我们如何从外部接收数据呢？正如您可以猜到的那样，它可以通过`nw.Clipboard`的`get`方法访问。文本可以按如下方式检索：

```js
 const clip = nw.Clipboard.get(); 
console.log( clip.get( "text" ) );

```

当图形放在剪贴板上时，我们只能在 NW.js 中获取 Base64 编码的内容或 HTML。为了看到它的实际效果，我们向`FileService`添加了一些方法：

`./js/Service/File.js`

```js
//... 
  hasImageInClipboard(){ 
    const clip = nw.Clipboard.get(); 
    return 

clip.readAvailableTypes().indexOf( "png" ) !== -1; 
  } 

  pasteFromClipboard(){ 
    const clip = 

nw.Clipboard.get(); 
    if ( this.hasImageInClipboard() ) { 
      const base64 = clip.get( "png", true ), 
            binary = Buffer.from( base64, "base64" ), 
            filename = Date.now() + "--img.png"; 

fs.writeFileSync( this.dir.getFile( filename ), binary ); 
      this.dir.notify(); 
    } 
  } 
//...

```

`hasImageInClipboard`方法检查剪贴板是否保留任何图形。`pasteFromClipboard`方法将剪贴板中的图形内容作为 Base64 编码的 PNG 获取；它将内容转换为二进制代码，将其写入文件，并请求`DirService`订阅者更新它。

要使用这些方法，我们需要编辑`ContextMenu`视图：

`./js/View/ContextMenu.js`

```js
getItems( fileName ){ 
    const file = this.file, 
          isCopied = Boolean( file.copiedFile ); 
       return [ 
     //... 
      { 
        label: this.i18n.translate( "PASTE_FROM_CLIPBOARD", "Paste 

image from clipboard" ), 
        enabled: file.hasImageInClipboard(), 
        click: () => 

file.pasteFromClipboard() 
      }, 
      //... 
    ]; 
  }

```

我们向菜单添加一个新项目`从剪贴板粘贴图像`，仅当剪贴板中有一些图形时才启用。

# 系统托盘中的菜单

我们的应用程序可用的三个平台都有所谓的系统通知区域，也称为系统托盘。这是用户界面的一部分（在 Windows 的右下角和其他平台的右上角），即使在桌面上没有应用程序图标，也可以在其中找到应用程序图标。使用`NW.js` API（[`docs.nwjs.io/en/latest/References/Tray/`](http://docs.nwjs.io/en/latest/References/Tray/)），我们可以为我们的应用程序提供一个图标和托盘中的下拉菜单，但我们还没有任何图标。因此，我已经创建了带有文本`Fe`的`icon.png`图像，并将其保存在大小为 32x32px 的应用程序根目录中。它在 Linux，Windows 和 macOS 上都受支持。但是，在 Linux 中，我们可以使用更高的分辨率，因此我将 48x48px 版本放在了旁边。

我们的应用程序在托盘中将由`TrayService`表示：

`./js/View/Tray.js`

```js
const appWindow = nw.Window.get(); 

class TrayView { 

  constructor( title ){ 

this.tray = null; 
    this.title = title; 
    this.removeOnExit(); 
    this.render(); 
  } 

  render(){ 
    const icon = ( process.platform === "linux" ? "icon-48x48.png" : "icon-32x32.png" ); 

    this.tray = new nw.Tray({ 
      title: this.title, 
      icon, 
      iconsAreTemplates: false 
    }); 

    const menu = new nw.Menu(); 
    menu.append( new nw.MenuItem({ 
      label: "Exit", 

      click: () => appWindow.close() 
    })); 
    this.tray.menu = menu; 
  } 

removeOnExit(){ 
    appWindow.on( "close", () => { 
      this.tray.remove(); 
      appWindow.hide(); 

// Pretend to be closed already 
      appWindow.close( true ); 
    }); 
    // do not spawn Tray instances 

on page reload
    window.addEventListener( "beforeunload", () => this.tray.remove(), false );
  } 

} 

exports.TrayView = TrayView;

```

它是做什么的？该类将托盘的标题作为构造函数参数，并在实例化期间调用`removeOnExit`和 render 方法。第一个订阅窗口的`close`事件，并确保在关闭应用程序时删除托盘。方法 render 创建`nw.Tray`实例。通过构造函数参数，我们传递了包含标题的配置对象，该标题是图标的相对路径。我们为 Linux 分配了`icon-48x48.png`图标，为其他平台分配了`icon-32x32.png`图标。默认情况下，macOS 尝试将图像调整为菜单主题，这需要图标由透明背景上的清晰颜色组成。如果您的图标不符合这些限制，您最好将其添加到配置对象属性`iconsAreTemplates`中，该属性设置为`false`。

在 Ubuntu 16.x 中启动我们的文件资源管理器时，由于白名单策略，它不会出现在系统托盘中。您可以通过在终端中运行`sudo apt-get install libappindicator1`来解决这个问题。

`nw.Tray`接受`nw.Menu`实例。因此，我们以与上下文菜单相同的方式填充菜单。现在我们只需在主模块中初始化`Tray`视图并运行应用程序：

`./js/app.js`

```js
const { TrayView } = require( "./js/View/Tray" ); 
new TrayView( "File Explorer" );

```

如果现在运行应用程序，我们可以在系统托盘中看到应用程序图标和菜单：

![](img/f193457b-315c-42e0-8b29-adbed34fdbef.png)

是的，唯一的菜单项退出看起来有点孤单。

让我们扩展`Tray`视图：

`./js/View/Tray.js`

```js
class TrayView { 

  constructor( title ){ 
    this.tray = null; 
    this.title = title; 
    // subscribe to window events 
    appWindow.on("maximize", () => this.render( false )); 

appWindow.on("minimize", () => this.render( false )); 
    appWindow.on("restore", () => this.render( true )); 

    this.removeOnExit(); 
    this.render( true ); 
  } 

  getItems( reset ){ 

  return [ 
      { 
        label: "Minimize", 
        enabled: reset, 
        click: () => 

appWindow.minimize() 
      }, 
      { 
        label: "Maximize", 
        enabled: reset, 

   click: () => appWindow.maximize() 
      }, 
      { 
        label: "Restore", 
        enabled: 

!reset, 
        click: () => appWindow.restore() 
      }, 
      { 
        type: "separator" 
      }, 
      { 
        label: "Exit", 
        click: () => appWindow.close() 
      } 

  ]; 
  } 

  render( reset ){ 
    if ( this.tray ) { 
      this.tray.remove(); 
    } 

    const icon = ( process.platform === "darwin" ? "macicon.png" : "icon.png" ); 

    this.tray = 

new nw.Tray({ 
      title: this.title, 
      icon, 
      iconsAreTemplates: true 
    }); 

    const menu = new nw.Menu(); 
    this.getItems( reset ).forEach(( item ) => menu.append( new nw.MenuItem( 

item ))); 

    this.tray.menu = menu; 
  } 

  removeOnExit(){ 
    appWindow.on( 

"close", () => { 
      this.tray.remove(); 
      appWindow.hide(); // Pretend to be closed already 

  appWindow.close( true ); 
    }); 
  } 

} 

exports.TrayView = TrayView;

```

现在，`render`方法接收一个布尔值作为参数，定义应用程序窗口是否处于初始模式；该标志传递给新的`getItems`方法，该方法生成菜单项元数据数组。如果标志为 true，则所有菜单项都可用，除了还原。有意义的是在最小化或最大化后将窗口恢复到初始模式。显然，当标志为`false`时，`Minimize`和`Maximize`项将被禁用，但我们如何知道窗口的当前模式？在构造时，我们订阅窗口事件最小化、最大化和还原。当事件发生时，我们使用相应的标志调用`render`。由于我们现在可以从`TitleBarActions`和`Tray`视图中更改窗口模式，因此`TitleBarActions`的`toggle`方法不再是窗口模式的可靠来源。相反，我们更倾向于重构模块，依赖窗口事件，就像我们在`Tray`视图中所做的那样：

`./js/View/TitleBarActions.js`

```js
const appWindow = nw.Window.get(); 

class TitleBarActionsView { 

  constructor( 

boundingEl, i18nService ){ 
    this.i18n = i18nService; 
    this.unmaximizeEl = boundingEl.querySelector( 

"[data-bind=unmaximize]" ); 
    this.maximizeEl = boundingEl.querySelector( "[data-bind=maximize]" ); 

this.minimizeEl = boundingEl.querySelector( "[data-bind=minimize]" ); 
    this.closeEl = boundingEl.querySelector( 

"[data-bind=close]" ); 
    this.bindUi(); 
    // Subscribe on i18nService updates 
    i18nService.on( 

"update", () => this.translate() ); 

    // subscribe to window events 
    appWindow.on("maximize", () 

=> this.toggleButtons( false ) ); 
    appWindow.on("minimize", () => this.toggleButtons( false ) ); 

appWindow.on("restore", () => this.toggleButtons( true ) ); 
  } 

  translate(){ 

this.unmaximizeEl.title = this.i18n.translate( "RESTORE_WIN", "Restore window" ); 
    this.maximizeEl.title = 

this.i18n.translate( "MAXIMIZE_WIN", "Maximize window" ); 
    this.minimizeEl.title = this.i18n.translate( 

"MINIMIZE_WIN", "Minimize window" ); 
    this.closeEl.title = this.i18n.translate( "CLOSE_WIN", "Close window" ); 
  } 

  bindUi(){ 
    this.closeEl.addEventListener( "click", this.onClose.bind( this ), false ); 
    this.minimizeEl.addEventListener( "click", this.onMinimize.bind( this ), false ); 

this.maximizeEl.addEventListener( "click", this.onMaximize.bind( this ), false ); 

this.unmaximizeEl.addEventListener( "click", this.onRestore.bind( this ), false ); 
  } 

toggleButtons( reset ){ 
    this.maximizeEl.classList.toggle( "is-hidden", !reset ); 

this.unmaximizeEl.classList.toggle( "is-hidden", reset ); 
    this.minimizeEl.classList.toggle( "is-hidden", !reset 

); 
  } 

  onRestore( e ) { 
    e.preventDefault(); 
    appWindow.restore(); 
  } 

  onMaximize( e ) { 
    e.preventDefault(); 
    appWindow.maximize(); 
  } 

onMinimize( e ) { 
    e.preventDefault(); 
    appWindow.minimize(); 
  } 

  onClose( e ) { 

    e.preventDefault(); 
    appWindow.close(); 
  } 
} 

exports.TitleBarActionsView = 

TitleBarActionsView;

```

这次当我们运行应用程序时，我们可以在系统托盘应用程序菜单中找到窗口操作：

![](img/18a14be2-3418-4b01-989c-90b4f25adabd.png)

# 命令行选项

其他文件管理器通常接受命令行选项。例如，您可以在启动 Windows 资源管理器时指定一个文件夹。它还响应各种开关。比如，您可以给它开关`/e`，资源管理器将以展开模式打开文件夹。

`NW.js`将命令行选项显示为`nw.App.argv`中的字符串数组。因此，我们可以更改主模块中`DirService`初始化的代码：

`./js/app.js`

```js
const dirService = new DirService( nw.App.argv[ 0 ] );

```

现在，我们可以直接从命令行中打开指定的文件夹：

```js
npm start ~/Sandbox

```

在基于 UNIX 的系统中，波浪线表示用户主目录。在 Windows 中的等效表示如下：

```js
npm start %USERPROFILE%Sandbox

```

我们还能做什么？仅作为展示，我建议实现`--minimize`和`--maximize`选项，分别在启动时切换应用程序窗口模式：`./js/app.js`

```js
const argv = require( "minimist" )( nw.App.argv ), 
         dirService = new DirService( argv._[ 0 ] ); 
 if ( argv.maximize ){ 
  nw.Window.get().maximize(); 
} 
if ( argv.minimize ){ 
  nw.Window.get().minimize(); 
}

```

当我们可以使用外部模块 minimist（[`www.npmjs.com/package/minimist`](https://www.npmjs.com/package/minimist)）时，手动解析`nw.App.argv`数组就没有意义了。它导出一个函数，该函数将所有不是选项或与选项相关联的参数收集到`_`（下划线）属性中。我们期望该类型的唯一参数是启动目录。它还在命令行上提供`maximize`和`minimize`属性时将它们设置为 true。

应该注意，NPM 不会将选项委托给运行脚本，因此我们应该直接调用`NW.js`可执行文件：

```js
nw . ~/Sandbox/ --minimize

```

或

```js
nw . ~/Sandbox/ --maximize

```

# 本机外观和感觉

现在，人们可以找到许多具有半透明背景或圆角的本机桌面应用程序。我们能否用`NW.js`实现这样的花哨外观？当然可以！首先，我们应该编辑我们的应用程序清单文件：

`./package.json`

```js
... 
"window": { 
    "frame": false, 
    "transparent": true, 
    ... 
  }, 
...

```

通过将 frame 字段设置为`false`，我们指示`NW.js`不显示窗口框架，而是显示其内容。幸运的是，我们已经实现了自定义窗口控件，因为默认的窗口控件将不再可用。通过透明字段，我们去除了应用程序窗口的不透明度。要看它的实际效果，我们编辑 CSS 定义模块：

`./assets/css/Base/definitions.css`

```js
:root { 
  --titlebar-bg-color: rgba(45, 45, 45, 0.7); 
  --titlebar-fg-color: #dcdcdc; 
  --dirlist-

bg-color: rgba(222, 222, 222, 0.9); 
  --dirlist-fg-color: #636363; 
  --filelist-bg-color: rgba(249, 249, 249, 

0.9); 
  --filelist-fg-color: #333341; 
  --dirlist-w: 250px; 
  --titlebar-h: 40px; 
  --footer-h: 

40px; 
  --footer-bg-color: rgba(222, 222, 222, 0.9); 
  --separator-color: #2d2d2d; 
  --border-radius: 

1em; 
}

```

通过 RGBA 颜色函数，我们将标题栏的不透明度设置为 70%，其他背景颜色设置为 90%。我们还引入了一个新变量`--border-radius`，我们将在`titlebar`和`footer`组件中使用它来使顶部和底部的角变圆：

`./assets/css/Component/titlebar.css`

```js
.titlebar { 
  border-radius: var(--border-radius) var(--border-radius) 0 0; 
}

```

`./assets/css/Component/footer.css`

```js
.footer { 
  border-radius: 0 0 var(--border-radius) var(--border-radius); 
}

```

现在我们可以启动应用程序并享受我们更新的花哨外观。

在 Linux 上，我们需要使用`nw . --enable-transparent-visuals --disable-gpu`命令行选项来触发透明度。

# 源代码保护

与原生应用程序不同，我们的源代码没有编译，因此对所有人都是开放的。如果你考虑商业用途，这可能不适合你。你至少可以混淆源代码，例如使用 Jscrambler（[`jscrambler.com/en/`](https://jscrambler.com/en/)）。另一方面，我们可以将我们的源代码编译成本地代码，并用`NW.js`加载它，而不是 JavaScript。为此，我们需要将 JavaScript 与应用程序捆绑分离。让我们创建`app`文件夹，并将除了`js`之外的所有内容移动到那里。`js`文件夹将被移动到一个新创建的目录`src`中：

```js
    .
    ├── app
    │   

└── assets
    │       └── css

│           ├── Base
    │           └── 

Component
    └── src
        └── 

js
            ├── Data
            ├── 

Service
            └── View

```

我们的 JavaScript 模块现在已经超出了项目范围，当需要时我们无法访问它们。然而，这些仍然是 Node.js 模块（[`nodejs.org/api/modules.html`](https://nodejs.org/api/modules.html)），符合 CommonJS 模块定义标准。因此，我们可以使用捆绑工具将它们合并成一个单一文件，然后将其编译成本地代码。我建议使用 Webpack（[`webpack.github.io/`](https://webpack.github.io/)），这似乎是目前最流行的捆绑工具。因此，我们将其放在根目录 webpack 配置文件中，内容如下：

`webpack.config.js`

```js
const { join } = require( "path" ), 
      webpack = require( "webpack" ); 

module.exports = { 

 entry: join( __dirname, "src/js/app.js" ), 
  target: "node-webkit", 
  output: { 
      path: join( 

__dirname, "/src/build" ), 
      filename:  "bundle.js" 
  } 
};

```

通过这样做，我们指示 Webpack 将所有必需的模块转译，从`src/js/app.js`开始，转译成一个单一的`src/build/bundle.js`文件。然而，与`NW.js`不同，Webpack 期望从托管文件（而不是项目根目录）相对于所需的依赖项；因此，我们必须从主模块的文件路径中删除`js/`：

`./src/js/app.js`

```js
// require( "./js/View/LangSelector" ) becomes 
require( "./View/LangSelector" )

```

为了转换 CommonJS 模块并将派生文件编译成本地代码，我们需要在清单的脚本字段中添加一些任务：

`package.json`

```js
//... 
"scripts": { 
    "build:js": "webpack", 
    "protect:js": "node_modules/nw/nwjs/nwjc 

src/build/bundle.js app/app.bin", 
    "build": "npm run build:js && npm run protect:js", 
    //... 
  }, 
//...

```

在第一个任务中，我们让 webpack 将我们的 JavaScript 源代码构建成一个单一文件。第二个任务使用`NW.js`编译器对其进行编译。最后一个任务同时完成了这两个任务。

在 HTML 文件中，我们用以下代码替换调用主模块的代码：

`app/index.html`

```js
<script> 
      nw.Window.get().evalNWBin( null, "./app.bin" ); 
</script>

```

现在我们可以运行应用程序，并观察实现的功能是否仍然符合我们的要求。

# 打包

好吧，我们已经完成了我们的应用程序，现在是时候考虑分发了。正如你所理解的，要求我们的用户安装`Node.js`并从命令行输入`npm start`并不友好。用户会期望一个可以像其他软件一样简单启动的软件包。因此，我们必须将我们的应用程序与`NW.js`捆绑在每个目标平台上。在这里，`nwjs-builder`派上了用场（[`github.com/evshiron/nwjs-builder`](https://github.com/evshiron/nwjs-builder)）。

因此，我们安装了`npm i -D nwjs-builder`工具，并在清单中添加了一个任务：

`./package.json`

```js
//... 
"scripts": { 
    "package": "nwb nwbuild -v 0.21.3-sdk ./app -o ./dist  -p linux64, win32,osx64", 

    //...   
  },
 //...

```

在这里，我们一次指定了三个目标平台（`-p linux64, win32,osx64`），因此，在运行此任务（`npm run package`）后，我们在`dist`目录中得到特定于平台的子文件夹，其中包含以我们应用程序命名的其他可执行文件：

```js
 dist 
├── file-explorer-linux-x64 
│   └── file-explorer 

├── file-explorer-osx-x64 
│   └── file-explorer.app 
└── file-explorer-win-x64 
    └── file-explorer.exe

```

`Nwjs-builder`接受各种选项。例如，我们可以要求它将软件包输出为 ZIP 存档：

```js
nwb nwbuild -v 0.21.3-sdk ./app -o ./dist --output-format=ZIP

```

或者，我们可以在构建过程后运行包并使用给定的选项：

```js
nwb nwbuild -v 0.21.3-sdk ./app -o ./dist -r  -- --enable-transparent-visuals --disable-gpu

```

# 自动更新

在持续部署的时代，新版本发布得相当频繁。作为开发人员，我们必须确保用户可以透明地接收更新，而不必经过下载/安装的流程。对于传统的 Web 应用程序，这是理所当然的。用户访问页面，最新版本就会加载。对于桌面应用程序，我们需要传递更新。不幸的是，`NW.js`并没有提供任何内置设施来处理自动更新，但我们可以欺骗它；让我们看看如何做。

首先，我们需要一个简单的发布服务器。让我们给它一个文件夹（例如`server`）并在那里创建清单文件：

`./server/package.json`

```js
{ 
  "name": "release-server", 
  "version": "1.0.0", 
  "packages": { 
    "linux64": { 
     "url": "http://localhost:8080/releases/file-explorer-linux-  
      x64.zip", 
      "size": 98451101 
    } 
  }, 
  "scripts": { 
    "start": "http-server ." 
  } 
}

```

该文件包含一个`packages`自定义字段，描述可用的应用程序发布。这个简化的实现只接受每个平台的最新发布。发布版本必须在清单版本字段中设置。每个包对象的条目包含可下载的 URL 和包大小（以字节为单位）。

为了为`release`文件夹中的清单和包提供 HTTP 请求服务，我们将使用 HTTP 服务器（[`www.npmjs.com/package/http-server`](https://www.npmjs.com/package/http-server)）。因此，我们安装该软件包并启动 HTTP 服务器：

```js
npm i -S http-server
npm start

```

现在，我们将回到我们的客户端并修改应用程序清单文件：

`./client/package.json`

```js
{ 
  "name": "file-explorer", 
   manifestUrl": "http://127.0.0.1:8080/package.json", 
  "scripts": { 

    "package": "nwb nwbuild -v 0.21.3-sdk . -o ../server/releases --output-format=ZIP", 
    "postversion": "npm 

run package" 
  }, 
//... 
}

```

在这里，我们添加了一个自定义字段`manifestUrl`，其中包含指向服务器清单的 URL。启动服务器后，清单将在`http://127.0.0.1:8080/package.json`上可用。我们指示`nwjs-builder`使用 ZIP 打包应用程序包并将它们放在`../server/release`中。最终，我们设置了`postversion`钩子；因此，当提升软件包版本（例如`npm version patch`）时，NPM 将自动构建并发送一个发布包到服务器，每次都是如此。

从客户端，我们可以读取服务器清单并将其与应用程序进行比较。如果服务器有更新版本，我们会下载与我们平台匹配的发布包，并将其解压缩到临时目录。现在我们需要做的就是用下载的版本替换正在运行的应用程序版本。但是，该文件夹在应用程序运行时被锁定，因此我们关闭正在运行的应用程序并启动下载的应用程序（作为一个独立的进程）。它备份旧版本并将下载的软件包复制到初始位置。所有这些都可以很容易地使用`nw-autoupdater`（`https://github.com/dsheiko/nw-autoupdater`）完成，因此我们安装`npm i -D nw-autoupdater`软件包并创建一个新的服务来处理自动更新流程：

`./client/js/Service/Autoupdate.js`

```js
const AutoUpdater = require( "nw-autoupdater" ), 
      updater = new AutoUpdater( nw.App.manifest ); 

async function start( el ){ 
  try { 
    // Update copy is running to replace app with the update 
    if 

( updater.isSwapRequest() ) { 
      el.innerHTML = `Swapping...`; 
      await updater.swap(); 

el.innerHTML = `Restarting...`; 
      await updater.restart(); 
      return; 
    } 

    // 

Download/unpack update if any available 
    const rManifest = await updater.readRemoteManifest(); 
    const 

needsUpdate = await updater.checkNewVersion( rManifest ); 
    if ( !needsUpdate ) { 
      return; 
    } 

    if ( !confirm( "New release is available. Do you want to upgrade?" ) ) { 
      return; 
    } 

    // Subscribe for progress events 
    updater.on( "download", ( downloadSize, totalSize ) => { 

 const procent = Math.floor( downloadSize / totalSize * 100 ); 
      el.innerHTML = `Downloading - ${procent}%`; 
    }); 
    updater.on( "install", ( installFiles, totalFiles ) => { 
      const procent = Math.floor( 

installFiles / totalFiles * 100 ); 
      el.innerHTML = `Installing - ${procent}%`; 
    }); 

const updateFile = await updater.download( rManifest ); 
    await updater.unpack( updateFile ); 

await updater.restartToSwap(); 
  } catch ( e ) { 
    console.error( e ); 
  } 
} 

exports.start = start;

```

在这里，我们应用了 ES2016 的 async/await 语法。通过在函数前加上`async`，我们声明它是异步的。之后，我们可以在任何 Promise（`https://mzl.la/1jLTOHB`）前使用 await 来接收其解析值。如果 Promise 被拒绝，异常将在 try/catch 语句中捕获。

这段代码到底是做什么的？正如我们商定的那样，它比较本地和远程清单版本。

如果发布服务器有更新版本，它会使用 JavaScript 的 confirm 函数通知用户。如果用户同意升级，它会下载最新版本并解压缩。在下载和解压缩过程中，更新程序对象会发出相应的消息；因此，我们可以订阅并表示进度。准备就绪后，服务将重新启动应用程序进行交换；因此，现在它用下载的版本替换了过时的版本并再次重新启动。在此过程中，服务通过在传入的 HTML 元素（el）中写入来向用户报告。按照设计，它期望元素代表标题栏中的路径容器。

因此，我们现在可以在主模块中启用服务：

`./client/js/app.js`

```js
const { start } = require( "./js/Service/Autoupdate" ), 
// start autoupdate 
setTimeout(() => { 

start( document.querySelector( "[data-bind=path]" ) ); 
}, 500 );

```

好了，我们如何测试它？我们跳转到客户端文件夹并构建一个分发包：

```js
npm run package

```

据说它会落在服务器/releases。我们解压到任意位置，例如`~/sandbox/`：

```js
unzip ../server/releases/file-explorer-linux-x64.zip -d ~/sandbox/

```

在这里，我们将找到可执行文件（对于 Linux，它将是`file-explorer`）并运行它。文件资源管理器将像往常一样工作，因为发布服务器没有更新版本，所以我们回到客户端文件夹并创建一个：

```js
npm version patch 

```

现在我们切换到服务器文件夹并编辑清单的版本以匹配刚生成的版本（1.0.1）。

然后，我们重新启动捆绑的应用程序（例如，`~/sandbox/file-explorer`）并观察提示：

![](img/4a1a69d8-fab1-4d91-a411-0956ebc63fe4.png)

点击“确定”后，我们可以在标题栏中看到下载和安装的进度：

![](img/c530a27a-0b49-47aa-9f6b-7b431e4749fa.png)

然后，应用程序重新启动并报告交换。完成后，它再次重新启动，现在已更新。

# 总结

在本章的开始，我们的文件资源管理器只能浏览文件系统并打开文件。我们扩展了它以显示文件夹中的文件，并复制/粘贴和删除文件。我们利用了`NW.js` API 来为文件提供动态构建的上下文菜单。我们学会了在应用程序之间使用系统剪贴板交换文本和图像。我们使我们的文件资源管理器支持各种命令行选项。我们提供了国际化和本地化的支持，并通过在本机代码中进行编译来保护源代码。我们经历了打包过程并为分发做好了准备。最后，我们建立了一个发布服务器，并为文件资源管理器扩展了一个自动更新的服务。
