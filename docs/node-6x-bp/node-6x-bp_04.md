# 第四章：不要拍照，制作它-摄影师的应用程序

在本章中，我们将讨论 Node.js 社区在全球范围内广泛讨论的一个话题-使用 Node.js 架构和云进行图像处理。

正如我们在上一章中看到的，我们有两种方式来存储图像和文件，一种是在服务器上使用硬盘，另一种是直接存储到云中。在第三章中，*构建多媒体应用程序*，我们使用了直接上传到服务器的方法，将图像存储在硬盘上。

在本章中，我们将使用云中的服务器来存储和处理我们相册应用程序中的图像。

我们将使用一个名为 Cloudinary 的服务来存储和处理图像。

在本章中，我们将涵盖以下主题：

+   如何使用 generator-express 设置 MVC 应用程序

+   如何安装 cloudinary npm 模块

+   实现 Materialize.css 框架

+   如何上传图像到云并在 MongoDB 上保存元数据

+   如何使用点文件中的全局变量

+   设置 Cloudinary 帐户并创建文件夹

+   如何使用 Cloudinary API 上传图像

+   如何使用 Cloudinary API 中的 URL 参数呈现模板

# 我们正在构建的内容

在本章结束时，我们将创建以下示例应用程序，一个强大而可扩展的相册：

![我们正在构建的内容](img/image_04_001.jpg)

相册应用程序的主屏幕

# 创建基线应用程序

在本章中，我们将使用稍微修改过的`express-generator`的版本，这是我们在之前章节中使用的。

这个生成器叫做`generator-express`；它在`express generator`的基础上进行了大量的改进。

以下是我们安装的步骤：

1.  打开终端/ shell 并输入：

```js
 npm install -g generator-express 

```

1.  创建一个名为`chapter04`的文件夹。

1.  在`chapter04`文件夹中打开您的终端/ shell，并输入以下命令：

```js
 yo express 

```

现在，按照以下顺序填写问题：

+   选择`N`，我们已经在*步骤 2*中创建了项目文件夹

+   选择`MVC`作为应用程序类型

+   选择`Swig`作为模板引擎

+   选择`None`作为 CSS 预处理器

+   选择`None`作为数据库（在本章后面，我们将手动设置数据库）

+   选择`Gulp`进行 LiveReload 和其他内容

### 提示

不要担心`Gulp`，如果你从未听说过它。在本书的后面，我们将看到并解释一些构建工具。

在生成器的最后，我们有以下目录结构：

![创建基线应用程序](img/image_04_002.jpg)

应用程序文件夹结构

## 更改应用程序结构

与我们在第一章中使用的示例不同，*使用 MVC 设计模式构建类似 Twitter 的应用程序*，我们不会对当前的结构进行重大更改；我们只会更改`views`文件夹。

作为示例应用程序，将有一本图片书；我们将在`views`文件夹中添加一个名为 book 的文件夹：

1.  在`app/views`文件夹中创建一个名为`book`的文件夹。

1.  现在我们将为 Cloudinary 服务创建一个配置文件。在本章后面，我们将讨论有关 Cloudinary 的所有细节；现在，只需创建一个新文件。

1.  在根文件夹中创建一个名为`.env`的文件。

现在，我们有必要的基础来继续前进。

## 添加处理图像和 Cloudinary 云服务的 Node 模块

现在我们将在`package.json`文件中为我们的应用程序添加必要的模块。

1.  将以下突出显示的代码行添加到`package.json`文件中：

```js
      { 
         "name": "chapter-04", 
         "description": "Don't take a photograph, make it - An app for
           photographers", 
         "license": "MIT", 
         "author": { 
           "name": "Fernando Monteiro", 
           "url": "https://github.com/newaeonweb/node-6-blueprints" 
         }, 
         "repository": { 
           "type": "git", 
           "url": "https://github.com/newaeonweb/node-6-blueprints.git" 
         }, 
         "keywords": [ 
           "MVC", 
           "Express Application", 
           "Expressjs", 
           "Expressjs cloud upload", 
           "Expressjs cloudinary upload" 
         ], 
         "version": "0.0.1", 
         "private": true, 
        "scripts": { 
          "start": "gulp" 
        }, 
        "dependencies": { 
          "body-parser": "¹.13.3", 
          "cloudinary": "¹.3.1", 
          "compression": "¹.5.2", 
          "connect-multiparty": "².0.0", 
          "cookie-parser": "¹.3.3", 
          "dotenv": "².0.0", 
          "express": "⁴.13.3", 
          "glob": "⁶.0.4", 
          "jugglingdb": "².0.0-rc3", 
          "jugglingdb-mongodb": "⁰.1.1", 
          "method-override": "².3.0", 
          "morgan": "¹.6.1", 
          "serve-favicon": "².3.0", 
          "swig": "¹.4.2" 
        }, 
        "devDependencies": { 
          "gulp": "³.9.0", 
          "gulp-nodemon": "².0.2", 
          "gulp-livereload": "³.8.0", 
          "gulp-plumber": "¹.0.0" 
        } 
      } 

```

只需几个模块，我们就可以构建一个非常强大和可扩展的应用程序。让我们描述每一个：

| 模块名称 | 描述 | 更多信息 |
| --- | --- | --- |
| `cloudinary` | 用于存储和管道图像和视频文件的云服务 | [`www.npmjs.com/package/cloudinary`](https://www.npmjs.com/package/cloudinary) |
| `connect-multiparty` | 用于接受多部分表单上传的中间件 | [`www.npmjs.com/package/connect-multiparty`](https://www.npmjs.com/package/connect-multiparty) |
| `dotenv` | 加载环境变量 | [`www.npmjs.com/package/dotenv`](https://www.npmjs.com/package/dotenv) |
| `jugglingdb` | 跨数据库 ORM | [`www.npmjs.com/package/jugglingdb`](https://www.npmjs.com/package/jugglingdb) |
| `jugglingdb-mongodb` | MongoDB 连接器 | [`www.npmjs.com/package/jugglingdb-mongodb`](https://www.npmjs.com/package/jugglingdb-mongodb) |

# 创建书籍控制器

我们将遵循生成器建议的相同生成器代码模式；使用此生成器的优势之一是我们已经可以使用 MVC 模式。

### 提示

请记住，您可以从 Packpub 网站或直接从 GitHub 书库下载示例文件。

1.  在`controllers`文件夹中创建一个名为`books.js`的文件。

1.  将以下代码添加到`book.js`文件中：

```js
      var express = require('express'), 
          router = express.Router(), 
          schema = require('../models/book'), 
          Picture = schema.models.Picture, 
          cloudinary = require('cloudinary').v2, 
          fs = require('fs'),     
          multipart = require('connect-multiparty'), 
          multipartMiddleware = multipart(); 

      module.exports = function (app) { 
          app.use('/', router); 
      }; 
      // Get pictures list 
      router.get('/books', function (req, res, next) { 
          Picture.all().then(function (photos) { 
              console.log(photos); 
          res.render('book/books', { 
              title: 'PhotoBook', 
              photos: photos, 
              cloudinary: cloudinary 
          }) 
          }); 
       }); 
       // Get form upload 
       router.get('/books/add', function (req, res, next) { 
           res.render('book/add-photo', { 
           title: 'Upload Picture' 
        }); 
      }); 
      // Post to 
      router.post('/books/add', multipartMiddleware, function (req, res,
       next)
      {
          // Checking the file received 
          console.log(req.files); 
          // create a new instance using Picture Model 
          var photo = new Picture(req.body); 
          // Get temp file path 
          var imageFile = req.files.image.path; 
          // Upload file to Cloudinary 
          cloudinary.uploader.upload(imageFile, { 
              tags: 'photobook', 
              folder: req.body.category + '/', 
              public_id: req.files.image.originalFilename 
              // eager: { 
              //   width: 280, height: 200, crop: "fill", gravity: "face" 
             // } 
          }) 
            .then(function (image) { 
              console.log('Picture uploaded to Cloudinary'); 
              // Check the image Json file 
              console.dir(image); 
              // Added image informations to picture model 
              photo.image = image; 
              // Save photo with image metadata 
              return photo.save(); 
            }) 
            .then(function (photo) { 
                console.log('Successfully saved'); 
                // Remove image from local folder 
                var filePath = req.files.image.path; 
                fs.unlinkSync(filePath); 
            }) 
            .finally(function () { 
                // Show the result with image file 
                res.render('book/posted-photo', { 
                    title: 'Upload with Success', 
                    photo: photo, 
                    upload: photo.image 
                }); 
            }); 
      }); 

```

让我们解释一些关于前面代码示例的重要点：

+   为了在视图中使用 Cloudinary API，我们需要将`cloudinary`变量传递给我们的视图：

```js
      res.render('book/books', { 
                title: 'PhotoBook', 
                photos: photos, 
                cloudinary: cloudinary 
      }) 

```

+   在使用`multipartMiddleware`时，为了最佳实践，我们需要清理上传到云中的每个文件：

```js

      .then(function (photo) { 
            console.log('Successfully saved'); 
            // Remove image from local folder 
            var filePath = req.files.image.path; 
            fs.unlinkSync(filePath); 
      }) 

```

稍后我们将讨论更多关于 Cloudinary API 的内容。

### 提示

请注意，当您使用多部分连接时，默认情况下会将图像加载到硬盘上的文件夹中，因此您应该始终删除应用程序中加载的所有文件。

## 创建书籍模型文件

为此应用程序创建模型的过程与我们在前几章中看到的非常相似；几乎每个模块的**ORM/ODM**都有非常相似的操作。

让我们看看如何为书籍对象创建模型：

1.  在`app/models`文件夹中创建一个名为`book.js`的文件，并放入以下代码：

```js
      var Schema = require('jugglingdb').Schema; 
      // Pay attention, we are using MongoDB for this example. 
      var schema = new Schema('mongodb', {url: 'mongodb://localhost
      /photobookapp'}); 

      // Setup Books Schema 
      var Picture = schema.define('Picture', { 
        title : { type: String, length: 255 }, 
        description: {type: Schema.Text}, 
        category: {type: String, length: 255 }, 
        image : { type: JSON} 
      }); 

      module.exports = schema; 

```

### 提示

请注意，我们使用 MongoDB 来存储书籍模型。还记得在启动应用程序之前必须使本地 MongoDB 运行起来。

# 向应用程序添加 CSS 框架

在本书的所有章节中，我们将始终使用最新的技术，就像在前几章中使用新的 Bootstrap（Alpha Release）一样。

特别是在本章中，我们将使用一种称为`Material Design`的设计模式。

### 提示

您可以在[`www.google.com/design/spec/material-design/introduction.html`](https://www.google.com/design/spec/material-design/introduction.html)上阅读更多关于设计材料的信息。

为此，我们将使用一个名为`Materialize.css`的简单`CSS`框架。

### 提示

您可以在此链接找到有关 Materialize 的更多信息：[`materializecss.com/`](http://materializecss.com/)。

1.  用以下代码替换`app/views/layout.swig`文件中的所有内容：

```js
      <!doctype html> 
      <html lang="en"> 
      <head> 
         <meta charset="UTF-8"> 
         <meta name="viewport" content="width=device-width"> 
         <title>{{ title }}</title> 
         <!--Let browser know website is optimized for mobile--> 
         <meta name="viewport" content="width=device-width, initial-
         scale=1.0"/>
         <!-- Import Google Material font and icons -->
         <link href="https://fonts.googleapis.com/icon?family=
         Material+Icons" rel="stylesheet"> 
          <!-- Compiled and minified CSS --> 
          <link rel="stylesheet" href="https://cdnjs.cloudflare.com/
          ajax/libs/materialize/0.97.6/css/materialize.min.css"> 
         <link rel="stylesheet" href="/css/style.css"> 
     </head> 
     <body> 
          <nav class="orange darken-4" role="navigation"> 
              <div class="nav-wrapper container"><a id="logo-container"
              href="/" class="brand-logo">Logo</a> 
                  <ul class="right hide-on-med-and-down"> 
                      <li><a href="/books">Books</a></li> 
                      <li><a href="/books/add">Add Picture</a></li> 
                  </ul> 

                  <ul id="nav-mobile" class="side-nav" style="transform:
                   translateX(-100%);"> 
                      <li><a href="/books">Books</a></li> 
                      <li><a href="/books/add">Add Picture</a></li> 
                  </ul> 
                  <a href="#" data-activates="nav-mobile" class="button-
                  collapse">
                  <i class="material-icons">menu</i></a> 
              </div> 
          </nav> 
          {% block content %}{% endblock %} 
          <!-- Footer --> 
          <footer class="page-footer orange darken-4"> 
              <div class="container"> 
                  <div class="row"> 
                     <div class="col l6 s12"> 
                          <h5 class="white-text">Some Text Example</h5> 
                          <p class="grey-text text-lighten-4">Lorem ipsum
                          dolor sit amet, consectetur adipiscing elit,
                          sed do eiusmod tempor incididunt ut labore et
                          dolore magnaaliqua. Ut enim ad minim veniam, quis
                          nostrud exercitation ullamco laboris nisi ut 
                          aliquip ex ea commodo consequat. Duis aute irure 
                          dolor in reprehenderit in voluptate velit esse 
                          cillum dolore eu fugiat nulla pariatur.</p> 
                      </div> 
                      <div class="col l3 s12"> 
                          <h5 class="white-text">Sample Links</h5> 
                          <ul> 
                            <li><a class="white-text" href="#!">Link 1
                              </a></li> 
                            <li><a class="white-text" href="#!">Link 2
                               </a></li> 
                            <li><a class="white-text" href="#!">Link 3
                               </a></li>
                            <li><a class="white-text" href="#!">Link 4
                               </a></li> 
                          </ul> 
                      </div> 
                    <div class="col l3 s12"> 
                        <h5 class="white-text">Sample Links</h5> 
                         <ul> 
                            <li><a class="white-text" href="#!">Link 1
                              </a></li> 
                            <li><a class="white-text" href="#!">Link 2
                              </a></li> 
                            <li><a class="white-text" href="#!">Link 3
                              </a></li>
                            <li><a class="white-text" href="#!">Link 4
                              </a></li>
                          </ul> 
                    </div> 
                </div> 
            </div> 
              <div class="footer-copyright"> 
                <div class="container"> 
                  MVC Express App for: <a class="white-text text-darken-2"
                  href="#">Node.js 6 Blueprints Book</a> 
                </div> 
              </div> 
          </footer> 
          <!-- Place scripts at the bottom page--> 
          {% if ENV_DEVELOPMENT %} 
          <script src="img/livereload.js"></script> 
          {% endif %} 

          <!--Import jQuery before materialize.js--> 
          <script type="text/javascript"
            src="img/jquery-2.1.1.min.js"></script> 
          <!-- Compiled and minified JavaScript --> 
          <script src="https://cdnjs.cloudflare.com/
          ajax/libs/materialize/0.97.6/js/materialize.min.js"></script> 
          <!-- Init Rsponsive Sidenav Menu  --> 
          <script> 
          (function($){ 
              $(function(){ 
                $('.button-collapse').sideNav(); 
                $('.materialboxed').materialbox(); 
              }); 
          })(jQuery); 
          </script> 
      </body> 
      </html> 

```

### 提示

为了避免 CSS 冲突，请清理您的`public/css/style.css`文件。

# 重构视图文件夹

现在我们将对`app/views`文件夹进行一些小改动并添加一些文件：

1.  首先，让我们编辑`app/views/index.js`。用以下代码替换原始代码：

```js
      {% extends 'layout.swig' %} 

      {% block content %} 

      <div class="section no-pad-bot" id="index-banner"> 
        <div class="container"> 
              <br><br> 
              <h1 class="header center orange-text">{{ title }}</h1> 
              <div class="row center">
                <h5 class="header col s12 light">Welcome to 
                    {{ title }}
                </h5> 
              </div> 
              <div class="row center"> 
                  <a href="books/add" id="download-button" class="btn-large
                   waves-effect waves-light orange">Create Your Photo
                   Book
                  </a> 
              </div> 
              <br><br> 
          </div> 
      </div> 
      <div class="container"> 
          <div class="section"> 

              <!--   Icon Section   --> 
              <div class="row"> 
                <div class="col s12 m4"> 
                  <div class="icon-block"> 
                    <h5 class="center">Animals</h5> 
                    <img src="img/animals"/> 
                    <p class="light">Lorem ipsum dolor sit amet,
                      consectetur adipiscing elit, sed do eiusmod
                      tempor incididunt ut laboreet dolore magna-
                      aliqua.</p> 
                  </div> 
                </div> 

                  <div class="col s12 m4"> 
                    <div class="icon-block"> 
                      <h5 class="center">Cities</h5> 
                      <img src="img/city"/> 
                      <p class="light">Lorem ipsum dolor sit amet,
                        consectetur adipiscing elit, sed do eiusmod
                        tempor incididunt ut laboreet dolore magna-
                        aliqua.</p> 
                    </div> 
                  </div> 

                  <div class="col s12 m4"> 
                    <div class="icon-block"> 
                      <h5 class="center">Nature</h5> 
                      <img src="img/nature"/> 
                      <p class="light">Lorem ipsum dolor sit amet,
                        consectetur adipiscing elit, sed do eiusmod
                         tempor incididunt ut laboreet dolore magna-
                          aliqua..</p> 
                    </div> 
                  </div> 
              </div> 

          </div> 
          <br><br> 

          <div class="section"> 

          </div> 
      </div> 

      {% endblock %} 

```

1.  创建一个名为`add-photo.swig`的文件，并添加以下代码：

```js
      {% extends '../layout.swig' %} 

      {% block content %} 
      <div class="section no-pad-bot" id="index-banner"> 
          <div class="container"> 
              <br>
              <br> 
              <h1 class="header center orange-text">{{ title }}</h1> 
              <div class="row center"> 
                  <h5 class="header col s12 light">Welcome to 
                    {{ title }}</h5> 
              </div> 

              <div class="photo"> 
                <h2>{{ photo.title }}</h2> 
                {% if photo.image.url %} 
                <img src="img/{{ photo.image.url }}" height='200' width='100%'>
                </img> 
                <a href="{{ photo.image.url }}" target="_blank"> 
                  {{ cloudinary.image(photo.image.public_id, {width: 150,
                   height: 150, quality:80,crop:'fill',format:'png',
                   class:'thumbnail inline'})  }} 
                </a> 
                {% endif %} 
              </div> 

              <div class="card"> 
                  <div class="card-content orange-text"> 
                   <form action="/books/add" enctype="multipart/form-data"
                     method="POST"> 
                          <div class="input-field"> 
                             <input id="title" name="title" type="text"
                               value="{{ photo.title }}" class="validate"> 
                              <label for="title">Image Title</label> 
                          </div> 
                          <div class="file-field input-field"> 
                              <div class="btn orange"> 
                                  <span>Choose File</span> 
                                  <input type="file" name="image"> 
                                <input id="photo_image_cache"
                                 name="image_cache" type="hidden" /> 
                              </div> 
                              <div class="file-path-wrapper"> 
                               <input class="file-path validate"
                                 type="text"> 
                              </div> 
                          </div> 
                          <div class="input-field col s12"> 
                              <select class="browser-default" id="category"
                               name="category"> 
                                <option value="" disabled selected>Choose a
                                  category</option> 
                                <option value="animals">Animals</option> 
                                <option value="cities">Cities</option> 
                               <option value="nature">Nature</option> 
                               </select> 
                          </div> 
                          <div class="input-field"> 
                              <input id="description" name="description"
                               type="text" value="{{ photo.description }}"
                                class="validate"> 
                              <label for="description">Image Text
                                 Description</label> 
                          </div> 
                          <br> 
                          <br> 
                          <div class="row center"> 
                              <button class="btn orange waves-effect waves
                               light" type="submit" name="action">
                                  Submit
                               </button> 
                          </div> 
                      </form> 
                  </div> 
              </div> 
              <br> 
              <br> 
              <br> 
          </div> 
      </div> 

      {% endblock %} 

```

1.  然后创建一个名为`books.swig`的文件，并添加以下代码：

```js
      {% extends '../layout.swig' %} 

      {% block content %} 
      <div class="section no-pad-bot" id="index-banner"> 
          <div class="container"> 
              <br><br> 
              <h1 class="header center orange-text">{{ title }}</h1> 
              <div class="row center"> 
               < h5 class="header col s12 light">Welcome to {{ title }}
               </h5> 
              </div> 
              <br><br> 

              {% if photos.length == 0 %} 
                <div class="row center"> 
                    <div class="card-panel orange lighten-2">No photos yet,
                      click add picture to upload</div> 
                </div> 
                {% endif %} 

              <div class="row"> 
              {% for item in photos %} 
                  <div class="col s12 m4"> 
                      <div class="icon-block"> 
                          <h5 class="center">{{ item.title }}</h5> 
                            {{ cloudinary.image(item.image.public_id, { 
                              width:280, height: 200, quality:80, 
                               crop:'fill',format:'png', effect:
                               'brightness:20', radius: 5, class:
                                'materialboxed' }) | safe }} 
                          {# 
                            Swig comment tag 
                            <img class="materialboxed" src="
                            {{ item.image.url }}" height='200' 
                             width='100%' alt="{{ item.title }}"
                             data-caption="{{item.description}}"></img> 
                          #} 
                          <p class="light">{{ item.description }}</p> 
                      </div> 
                  </div> 
              {% endfor %} 
              </div> 
          </div> 
      </div> 
      {% endblock %} 

```

1.  创建一个名为`posted-photo.swig`的文件，并添加以下代码：

```js
      {% extends '../layout.swig' %} 

      {% block content %} 
      <div class="section no-pad-bot" id="index-banner"> 
          <div class="container"> 
              <br><br> 
              <h1 class="header center orange-text">{{ title }}</h1> 
              <div class="row center"> 
                  <h5 class="header col s12 light">Welcome to 
                    {{ title }}
                  </h5> 
              </div> 
              <div class="photo col s12 m12"> 
                <h2>{{ photo.title }}</h2> 
                {% if photo.image.url %} 
                <img src="img/{{ photo.image.url }}" width='100%'></img> 
                <a href="{{ photo.image.url }}" target="_blank"> 
                  {{ cloudinary.image(photo.image.public_id, {width: 150,
                    height: 150, quality: 80,crop:'fill',format:'png', 
                    class:'thumbnail inline'})  }} 
                </a> 
                {% endif %} 
              </div> 
              <br> 
              <br> 
             <br> 
          </div> 
       </div> 

      {% endblock %} 

```

# 创建.env.js 文件

在这一点上，我们将创建`env.js`文件；此文件验证了 Cloudinary 服务的配置。在`config`文件夹中创建一个名为`env.js`的文件，并放入以下代码：

```js
// get Env variable / cloudinary 
module.exports = function(app, configEnv) { 

    var dotenv = require('dotenv'); 
    dotenv.load(); 
    var cloudinary = require('cloudinary').v2; 
    // Log some messages on Terminal 
    if ( typeof(process.env.CLOUDINARY_URL) == 'undefined' ){ 
      console.log('Cloudinary config file is not defined'); 
      console.log('Setup CLOUDINARY_URL or use dotenv mdule file') 
    } else { 
      console.log('Cloudinary config, successfully used:'); 
      console.log(cloudinary.config()) 
    } 
} 

```

现在我们有一个完全配置的应用程序；但是，我们仍然需要在 Cloudinary 服务上创建一个帐户。

## 创建和配置 Cloudinary 账户

Cloudinary 是用于存储和处理图像和视频文件的云服务；您可以在[`cloudinary.com`](http://cloudinary.com)找到有关 Cloudinary 服务的更多信息：

1.  转到[`cloudinary.com/users/register/free`](https://cloudinary.com/users/register/free)并注册一个免费帐户。

### 提示

在注册表单的末尾，您可以为您的云设置一个名称。我们选择了`n6b`（Node.js 6 蓝图）；选择您自己的名称。

1.  从您的帐户中复制数据（`环境变量`）并将其直接放到仪表板面板上，如下面的屏幕截图所示：![创建和配置 Cloudinary 帐户](img/image_04_003.jpg)

Cloudinary 仪表板面板

1.  现在在`.env.js`文件中使用您自己的凭据更新以下代码：

```js
      PORT=9000
      CLOUDINARY_URL=cloudinary://82499XXXXXXXXX:dXXXXXXXXXXX@n6b
```

# Cloudinary 的工作原理

除了我们在 Cloudinary 上存储文件之外，我们还可以使用强大的 API 来操作和转换图像，应用效果，调整大小，以及在我们的机器上不使用任何软件做更多的事情。

让我们回到`books.js`控制器，查看我们使用了什么。我们从 promises 函数中提取了额外的代码，以便专注于突出显示的代码行：

```js
      cloudinary.uploader.upload(imageFile, 
            { 
              tags: 'photobook', 
              folder: req.body.category + '/', 
              public_id: req.files.image.originalFilename 
              // eager: { 
              //   width: 280, height: 200, crop: "fill", gravity: "face" 
             // } 
            }) 
              .then(function (image) { 
               ... 
             }) 
             .then(function (photo) { 
                ... 
              }) 
              .finally(function () { 
                ... 
              }); 

```

在这里，我们设置了一个文件夹，`folder: req.body.category`，用于存储我们的图像，并覆盖了默认的`public_id: req.files.image.originalFilename`，以使用图像名称。这是一个很好的做法，因为 API 为我们提供了一个带有随机字符串的`public_id`——没有错，但非常有用。例如，查看这样的链接：`http://res.cloudinary.com/demo/image/upload/mydog.jpg`

而不是这个：[`res.cloudinary.com/demo/image/upload/8jsb1xofxdqamu2rzwt9q.jpg`](http://res.cloudinary.com/demo/image/upload/8jsb1xofxdqamu2rzwt9q.jpg)。

注释的`eager`属性使我们能够转换图像并生成一个带有所有急切选项的新图像。在这种情况下，我们可以保存一个宽度为*280px*，高度为*200px*的转换图像，裁剪填充内容，如果图片中有一些脸，缩略图将居中显示在脸上。这是一个非常有用的功能，可以保存图像配置文件。

您可以在上传方法上使用任何转换组合；以下是 API 返回的`JSON`的示例：

```js
      { title: 'Sample01',
          description: 'Example with Transformation',
          image:
           { public_id: 'cpl6ghhoiqphyagwvbaj',
             version: 1461269043,
             signature: '59cbbf3be205d72fbf7bbea77de8e7391d333363',
             width: 845,
             height: 535,
             format: 'bmp',
             resource_type: 'image',
             created_at: '2016-04-21T20:04:03Z',
             tags: [ 'photobook' ],
             bytes: 1356814,
             type: 'upload',
             etag: '639c51691528139ae4f1ef00bc995464',
             url: 'http://res.cloudinary.com/n6b/image/upload/v146126904
                   /cpl6ghhoiqphyagwvbaj.bmp',
             secure_url: 'https://res.cloudinary.com/n6b/image/upload
               /v1461269043/cpl6ghhoiqphyagwvbaj.bmp',
               coordinates: { faces: [ [ 40, 215, 116, 158 ] ] },  original_filename: 'YhCmSuFxm0amW5TFX9FqXt3F',  eager:[ { transformation: 'c_thumb,g_face,h_100,w_150', 
			   width: 150, height: 100,url: 'http://res.cloudinary.com
               /n6b/image/upload/c_thumb,g_face,h_100,w_150/v1461269043
               /cpl6ghhoiqphyagwvbaj.bmp', secure_url:
                'https://res.cloudinary.com/n6b/image/upload
                 /c_thumb,g_face,h_100,w_150/v1461269043
                 /cpl6ghhoiqphyagwvbaj.bmp' } ] }

```

注意带有 URL 转换的突出显示的代码：

`c_thumb,g_face,h_100,w_150`

### 提示

您可以在以下链接找到有关 Cloudinary 上传 API 的更多信息：[`cloudinary.com/documentation/node_image_upload`](http://cloudinary.com/documentation/node_image_upload)。

# 运行应用程序

现在是时候执行应用程序并上传一些照片了：

1.  打开您的终端/Shell 并输入以下命令：

```js
 npm start

```

1.  转到`http://localhost:3000/books`，您将看到以下屏幕：![运行应用程序](img/image_04_004.jpg)

书籍屏幕

## 上传和显示图像

现在让我们插入一些图像，并检查我们应用程序的行为：

1.  转到`http://localhost:3000/books/add`，并填写表单：![上传和显示图像](img/image_04_005.jpg)

上传表单

添加以下信息：

标题：**图像示例 02**

文件：选择 sample02.jpg 文件。

类别：**城市**

描述：**Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua.**

1.  让我们检查一下我们的 MongoDB，看看在我们继续之前发生了什么。

1.  打开您的`RoboMongo`并选择第一个对象：![上传和显示图像](img/image_04_006.jpg)

来自 MongoDB（RoboMongo）的屏幕截图

### 提示

请注意，您必须从左侧面板菜单中选择正确的数据库。

1.  当我们上传一张图片时，API 会返回一个包含与该图片相关的所有信息的 JSON。我们将这个 JSON 存储为我们的书籍模型的图像属性，存储在 MongoDB 中，正如我们在之前的图片中所看到的。

1.  通过*Sample02*至*Sample06*重复*步骤 1*。

## 检查 MongoDb 图片集合

让我们在 MongoDB 上看一下图片集合：

1.  打开 RoboMongo 并从左侧面板中选择正确的数据库。

1.  打开`collections`文件夹，双击**图片集合**。

1.  在面板的右上方，单击`以表格模式查看结果`图标。

现在您可以在**RoboMongo**界面上看到以下屏幕截图：

![检查 MongoDb 图片集合](img/image_04_007.jpg)

来自图片集合的屏幕截图

# 在 Cloudinary 仪表板中创建文件夹

如前所述，我们设置了文件夹（`folder: req.body.category`）。在这种情况下，文件夹名称将是类别名称。为了更好地在云中组织我们的图像，就像我们以编程方式做的那样，我们需要直接在 Cloudinary 仪表板中创建它们：

1.  登录到您的 Cloudinary 帐户。

1.  转到[`cloudinary.com/console/media_library`](https://cloudinary.com/console/media_library)：![在 Cloudinary 仪表板中创建文件夹](img/image_04_008.jpg)

创建文件夹截图

### 注意

不要担心 Cloudinary 仪表板上的其他图像；它们是每个帐户中的默认图像。如果您愿意，可以删除它们。

1.  点击右侧的输入字段（文件夹名称）并创建一个名为`animals`的文件夹。

1.  点击右侧的输入字段（文件夹名称）并创建一个名为`cities`的文件夹。

1.  点击右侧的输入字段（文件夹名称）并创建一个名为`nature`的文件夹。

您可以在图像顶部看到所有创建的类别，如下面的截图所示：

![在 Cloudinary 仪表板中创建文件夹](img/image_04_009.jpg)

类别截图

现在当您选择一个类别时，您只会看到属于该类别的图像，例如`animals`，如下图所示：

![在 Cloudinary 仪表板中创建文件夹](img/image_04_010.jpg)

动物文件夹的截图

这是一种更有效的组织所有照片的方式，您可以创建几个相册，例如：

```js
 my-vacations/germany/berlin 
      road-trip/2015/route-66

```

# URL 转换渲染

作为 Cloudinary API 的一部分，我们可以通过使用 URL 参数设置来操纵图像，就像我们在书籍页面上所做的那样：

1.  转到`http://localhost:3000/books`。

1.  打开您的 Web 检查器并检查第一张图像的呈现代码；您将看到以下代码：

```js
      <img src="http://res.cloudinary.com/n6b/image/upload
       /c_fill,e_brightness:20,h_200,q_80,r_5,w_280/v1/cities/sample01.jpg
       .png" class="materialboxed initialized" height="200" width="280"> 

```

API 创建`img`标签，并应用`app/views/books.swig`中定义的对象属性作为 URL 参数，如下面的代码所示：

```js
      {{ cloudinary.image(item.image.public_id, { width: 280, height: 200,
       quality: 80,crop: 'fill',format:'png', effect: 'brightness:20',
       radius: 5, class:'materialboxed' }) | safe }} 

```

| **对象属性** | **URL 参数** |
| --- | --- |
| width: *280* | `w_280` |
| height: *200* | `h_200` |
| crop: *fill* | `c_fill` |
| quality: *80* | `q_80` |
| effect:brightness:*20* | `e_brightness:20` |
| 半径：5 | `r_5` |

花括号和安全过滤器`{{... | safe}}`是`Swig`模板引擎的标记，用于在视图上安全地呈现变量。

此外，我们还可以直接使用`img`标签，如下面的代码所示：

```js
 <img class="materialboxed" src="img/{{ item.image.url }}" height='200'
       width='100%' alt="{{ item.title }}" 
       data-caption="{{item.description}}">
      </img>

```

## 添加原始图像的直接链接

我们还可以使用 API 生成原始图像链接，而不应用任何转换：

1.  打开`app/views/books.swig`并添加以下突出显示的代码：

```js
      <div class="icon-block"> 
      <h5 class="center">{{ item.title }}</h5> 
        {{ cloudinary.image(item.image.public_id, { width: 280, height:
          200, quality: 80,crop: 'fill',format:'png', effect:
          'brightness:20', radius:5,class:'materialboxed' }) | safe }} 
       {# 
          Swig comment tag 
          <img class="materialboxed" src="img/{{ item.image.url }}"
           height='200' width='100%' alt="{{ item.title }}"
           data-caption="{{item.description}}">
          </img> 
      #} 
      <p class="light">{{ item.description }}</p> 
      <a href="{{ cloudinary.url(item.image.url) }}" target="_blank"> 
      Link to original image 
      </a> 
      </div> 

```

1.  现在当我们点击`链接到原始图像`时，我们可以在另一个浏览器窗口中看到完整的图像：![添加原始图像的直接链接](img/image_04_011.jpg)

带有原始图像链接的书籍页面截图

重要的是要注意，我们还使用了`Materialize.css`框架中的简单`colorbox`，因此当我们将鼠标悬停在图像上时，我们可以看到一个图标，显示全尺寸的图像。

# 总结

我们已经到达了另一章的结尾。通过这一章，我们完成了一系列四章，讨论了使用 Node.js 进行软件开发的 MVC 模式。

在本章中，我们看到了如何构建一个使用云服务上传和操作图像的应用程序，还展示了如何应用效果，如亮度和边框半径。此外，我们还看到了如何使用简单的界面框架**Materialize.css**构建简单的图像库。

我们探索了一种不同的使用 ORM 模块的方式，并直接以 JSON 格式将所有有关图像的信息保存在 MongoDB 中。

在下一章中，我们将看到如何使用 Node 和 Firebase 云服务构建 Web 应用程序。
