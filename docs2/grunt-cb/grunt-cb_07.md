# 第七章. 部署到最终用户

在本章中，我们将介绍以下菜谱：

+   部署到 Rackspace Cloud Files

+   部署到 AWS S3

+   通过 FTP 部署

+   通过 SFTP 部署

+   部署到 GitHub Pages

+   使 AWS CloudFront 分发无效

+   通过 SSH 运行命令

# 简介

一旦我们的网络应用程序构建完成，并且其资产已优化以实现最佳交付和消费，就是时候将其提供给我们的目标受众了。这主要涉及将构成应用程序的资产传输到某种形式的文件托管系统，该系统专门用于通过互联网交付静态内容。

将网络应用程序的资产部署到互联网的主要关注点是可用性、速度和服务集成。资产应始终可以从世界任何地方访问，尽可能快地交付，并且托管系统应允许我们轻松上传和管理我们的内容。

# 部署到 Rackspace Cloud Files

在本菜谱中，我们将使用 `cloudfiles (0.3.0)` 插件将文件上传到 **Rackspace Cloud Files** 容器。

Cloud Files 服务默认提供一项附加好处，即为它托管的所有内容提供 **内容分发网络** (**CDN**) 服务。

## 准备工作

在本例中，我们将使用我们在 第一章 中 *在项目中安装 Grunt* 菜单创建的基本项目结构。如果您还不熟悉其内容，请务必参考它。

以下示例还需要一个带有 API 密钥定义的 **Rackspace Cloud** 账户。我们还需要创建一个名为 `myapp` 的 Cloud Files 容器，并将其配置为托管一个 **静态网站**。

### 小贴士

请参考以下 URL 获取有关配置 Cloud Files 容器以托管静态网站的更多信息：

[`docs.rackspace.com/files/api/v1/cf-devguide/content/Create_Static_Website-dle4000.html`](http://docs.rackspace.com/files/api/v1/cf-devguide/content/Create_Static_Website-dle4000.html)

## 如何操作...

以下步骤将引导我们创建一个简单的 HTML 文档和一个将其上传到 Cloud Files 容器的任务：

1.  我们将按照 第一章 中 *安装插件* 菜单提供的说明，安装包含 `cloudfiles` 插件的包，*使用 Grunt 入门*。

1.  接下来，我们将在项目目录中创建一个简单的 HTML 文档，名为 `index.html`，并为其提供以下内容：

    ```js
    <html>
      <head>
        <title>MyApp</title>
      </head>
      <body>
        <h1>This is MyApp.</h1>
      </body>
    </html>
    ```

1.  现在，我们将向我们的配置中添加以下 `cloudfiles` 任务，这将指示我们希望将 `index.html` 文件上传到 `myapp` Cloud Files 容器：

    ```js
    cloudfiles: {
      myapp: {
        user: '[username]',
        key: '[api key]',
        region: 'LON',
        upload: [{
          container: 'myapp',
          src: 'index.html',
        }]
      }
    }
    ```

    ### 小贴士

    `user` 和 `key` 选项填充了占位符值，以表明您应该使用您自己的 Rackspace Cloud 账户的用户名和 API 密钥。

    你可能希望将`user`和`key`选项存储在本地文件中，而不是共享仓库中。请参考第一章中关于如何从外部文件导入配置的示例，即*使用 Grunt 入门*配方。同时，请确保排除包含访问凭证的项目代码库中的文件。

    `region`选项用于指示托管 Cloud Files 容器的地理位置区域。所需的区域在创建容器时指定。在我们的示例中，我们在`LON`区域创建了容器。

1.  我们现在可以通过使用`grunt cloudfiles`命令来运行任务，它应该会产生类似于以下内容的输出：

    ```js
    Running "cloudfiles:myapp" (cloudfiles) task
    Uploading into myapp
    Syncing files to container: myapp
    Uploading index.html to myapp (NEW)

    ```

1.  我们现在可以确保文件已上传到我们的容器中，并且可以通过互联网访问。为此，我们需要从其设置中确定容器的目标域名，并在浏览器中导航到它。这应该看起来像以下这样：![如何操作...](img/image00274.jpeg)

## 还有更多...

`cloudfiles`任务为我们提供了几个有用的选项，可以与基本上传功能一起使用。我们将查看如何将目录内容上传到目标目录。

### 上传目录内容

如果我们想上传目录内容，我们可以在`src`选项中使用 Grunt 支持的常规通配符模式。然而，我们可能还希望使用`stripcomponents`选项从目标文件的目录名称中删除前导路径。

以下示例将`www`目录的内容上传到`myapp`容器，并在确定目标文件名时从目标文件中删除第一个路径名：

```js
cloudfiles: {
  myapp: {
    user: 'juriejan',
    key: 'c980b9327b823b96dd83b51cdc5cf7dd',
    region: 'LON',
    upload: [{
      container: 'myapp',
      src: 'www/**/*',
 stripcomponents: 1
    }]
  }
}
```

### 上传到目标目录

如果我们想将文件上传到目标容器上的特定目标目录，我们可以使用`dest`选项来指示目标目录，如下例所示：

```js
cloudfiles: {
  myapp: {
    user: 'juriejan',
    key: 'c980b9327b823b96dd83b51cdc5cf7dd',
    region: 'LON',
    upload: [{
      container: 'myapp',
      src: 'index.html',
      dest: '/htmlfiles/'
    }]
  }
}
```

# 部署到 AWS S3

在此配方中，我们将使用`aws-s3 (0.12.3)`插件将文件上传到**AWS S3 存储桶**。

此服务默认不提供 CDN 设置，但可以轻松与**AWS CloudFront**集成，以加快托管文件的分发速度。

## 准备工作

在此示例中，我们将使用我们在第一章中*在项目中安装 Grunt*配方中创建的基本项目结构。如果您还不熟悉其内容，请务必参考它。

除了标准的项目设置外，此配方还需要设置一个具有**AWS 访问密钥**的**AWS 用户**。

### 小贴士

有关如何获取您的 AWS 安全凭证的详细信息，请参阅以下 URL：

[`docs.aws.amazon.com/general/latest/gr/getting-aws-sec-creds.html`](http://docs.aws.amazon.com/general/latest/gr/getting-aws-sec-creds.html)

还需要创建一个以 `[name].myapp` 格式命名的存储桶，其中 `[name]` 是您希望使用的任何唯一名称。上述用户还应通过 **AWS IAM** 接口获得对创建的存储桶的完全访问权限。

### 小贴士

有关如何授予用户账户对 S3 存储桶的访问权限的信息，请参阅以下 URL：

[`blogs.aws.amazon.com/security/post/Tx3VRSWZ6B3SHAV/Writing-IAM-Policies-How-to-grant-access-to-an-Amazon-S3-bucket`](http://blogs.aws.amazon.com/security/post/Tx3VRSWZ6B3SHAV/Writing-IAM-Policies-How-to-grant-access-to-an-Amazon-S3-bucket)

为了使存储桶按预期工作，它还应配置为 **静态网站托管**，并且 `index.html` 应定义为它的 **索引文档**。

### 小贴士

有关如何配置 AWS S3 存储桶以进行静态网站托管的信息，请参阅以下 URL：

[`docs.aws.amazon.com/AmazonS3/latest/dev/HowDoIWebsiteConfiguration.html`](http://docs.aws.amazon.com/AmazonS3/latest/dev/HowDoIWebsiteConfiguration.html)

## 如何操作...

以下步骤将引导我们创建一个示例 HTML 文档并配置一个任务，将文档上传到 AWS S3 存储桶：

1.  我们将按照 第一章 中 *安装插件* 菜单提供的说明，安装包含 `aws-s3` 插件的包，该菜单位于 *使用 Grunt 入门*。

1.  接下来，我们将在项目目录中创建一个名为 `index.html` 的简单 HTML 文档，并为其提供以下内容：

    ```js
    <html>
      <head>
        <title>MyApp</title>
      </head>
      <body>
        <h1>This is MyApp.</h1>
      </body>
    </html>
    ```

1.  现在，我们将添加以下 `aws_s3` 任务到我们的配置中，该任务表示我们希望将 `index.html` 文件上传到 `gruntbook.myapp` 存储桶：

    ```js
    aws_s3: {
      myapp: {
        options: {
          accessKeyId: '[access key id]',
          secretAccessKey: '[secret access key]',
          region: 'eu-west-1',
          bucket: 'gruntbook.myapp',
        },
        files: [{
          src: 'index.html',
          dest: '/'
        }]
      }
    }
    ```

    ### 小贴士

    `accessKeyId` 和 `secretAccessKey` 选项填充了占位符值，以表明您应使用自己的 AWS 访问凭证。

    `accessKeyId` 选项应设置为为您的用户生成的 **访问密钥 ID**，该密钥 ID 是通过 **AWS 身份和访问管理**（**IAM**）控制台生成的。

    `secretAccessKey` 选项应设置为为在 `accessKeyId` 选项中指定的特定访问密钥 ID 生成的 **秘密访问密钥**。请注意，秘密访问密钥仅在创建访问密钥 ID 时显示，因此如果您第一次没有保存它，您将无法在 IAM 控制台中找到它。

    您可能希望将 `accessKeyId` 和 `secretAccessKey` 选项存储在本地文件中，而不是共享存储库中。请参阅 第一章 的 *导入外部数据* 菜谱，*开始使用 Grunt*，了解如何从外部文件导入配置的示例。此外，请确保将包含访问凭证的文件从您的项目代码存储库中排除。

    `region` 选项用于指示托管存储桶的地理位置。所需的区域在创建存储桶时指定。在我们的示例中，我们在 `eu-west-1` 区域创建了一个容器。

    注意，此任务的文件配置支持所有标准的 Grunt 选项。您可以在以下网址了解更多信息：

    [`gruntjs.com/configuring-tasks#files`](http://gruntjs.com/configuring-tasks#files)

1.  我们现在可以通过使用 `grunt aws_s3` 命令来运行任务，这应该会产生类似于以下输出的结果：

    ```js
    Running "aws_s3:myapp" (aws_s3) task
    Uploading to https://s3-eu-west-1.amazonaws.com/gruntbook.myapp/
    .
    List: (1 objects):
    - index.html -> https://s3-eu-west-1.amazonaws.com/gruntbook.myapp/index.html

    ```

1.  现在，我们可以确保文件已上传到我们的容器中，并且可以通过互联网访问。为此，我们需要从其属性部分确定存储桶的端点，并在浏览器中导航到它。这应该看起来像以下这样：![如何操作...](img/image00275.jpeg)

## 还有更多...

`aws_s3` 任务为我们提供了一些有用的选项，可以与它的上传功能一起使用。我们将探讨如何指定上传文件的访问权限并启用并发操作。

### 指定上传文件的访问权限

每个上传到 AWS S3 存储桶的文件都有一组访问权限，指示谁可以访问它。如果我们想指定一组特定的权限，可以通过使用 `access` 选项来实现。以下示例将 `access` 选项设置为 `private`，表示所有上传的文件只能由上传过程中使用的用户账户访问：

```js
aws_s3: {
  myapp: {
    options: {
      accessKeyId: 'AKIAJG27ICB3NTY3SGCQ',
      secretAccessKey: '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',
      region: 'eu-west-1',
      bucket: 'gruntbook.myapp',
      access: 'private'
    },
    files: [{
      src: 'index.html',
      dest: '/'
    }]
  }
}
```

### 小贴士

访问选项的可能值列表可以在以下 AWS `putObject` 操作的文档中找到（在 ACL 参数下查看）：

[`docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#putObject-property`](http://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/S3.html#putObject-property)

### 启用并发上传

`aws_s3` 任务的默认行为是依次完成上传。如果我们想并行执行上传，可以通过使用 `uploadConcurrency` 选项来实现。在以下示例中，我们表明我们希望同时上传的最大文件数为 `3`：

```js
aws_s3: {
  myapp: {
    options: {
      accessKeyId: 'AKIAJG27ICB3NTY3SGCQ',
      secretAccessKey: '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',
      region: 'eu-west-1',
      bucket: 'gruntbook.myapp',
      uploadConcurrency: 3
    },
    files: [{
      expand: true,
      cwd: 'www',
      src: '**/*',
      dest: '/'
    }]
  }
}
```

### 小贴士

之前的示例还演示了递归上传整个目录内容的配置。

# 通过 FTP 部署

在本菜谱中，我们将使用 `ftp-push (0.3.2)` 插件将文件上传到托管服务器，使用 **文件传输协议** (**FTP**)。

FTP 自互联网早期以来就存在，并且仍然被广泛使用。正如其名称所暗示的，它提供了一种在互联网上传输文件的方式，因此，自其诞生以来，它一直是部署资源到 Web 服务器的基石。

## 准备工作

在本例中，我们将使用我们在第一章中创建的基本项目结构，即在项目中安装 Grunt 配方中创建的结构。如果您还不熟悉其内容，请务必参考它。

除了标准的项目设置外，以下配方还需要在目标 FTP 启用服务器上现有的用户账户。他们的凭证通常由托管服务提供商或负责维护相关服务器的系统管理员提供。

## 如何操作...

以下步骤将引导我们创建一个简单的 HTML 文档，并配置一个任务，使用 FTP 将其上传到服务器：

1.  我们将按照第一章中安装插件配方中提供的说明，安装包含`ftp-push`插件的包。

1.  接下来，我们将在项目目录中创建一个简单的 HTML 文档，命名为`index.html`，并为其提供以下内容：

    ```js
    <html>
      <head>
        <title>MyApp</title>
      </head>
      <body>
        <h1>This is MyApp.</h1>
      </body>
    </html>
    ```

1.  现在，我们将添加以下`ftp_push`任务到我们的配置中，并指出我们希望它将`index.html`文件上传到我们的托管服务器：

    ```js
    ftp_push: {
      myapp: {
        options: {
          host: '[host]',
          username: '[username]',
          password: '[password]',
          dest: '/www'
        },
        files: [{
          src: 'index.html',
        }]
      }
    }
    ```

    ### 小贴士

    `username`、`password`和`host`选项填写了占位符值，以表明您应该使用自己的特定访问凭证和托管服务器地址。

    您可能希望将`username`和`password`选项存储在本地文件中，而不是共享存储库中。请参考第一章中导入外部数据配方中的示例，了解如何从外部文件导入配置。同时，请注意，您应该从项目的代码存储库中排除包含访问凭证的文件。

    `host`选项用于指定您想要上传文件的互联网地址。如果您不确定这个值应该是多少，此类信息通常可以由托管提供商或负责维护相关服务器的系统管理员提供。

    `dest`选项是必需的，用于指示我们将上传的文件的目标目录。在我们的例子中，我们将指示文件应该上传到托管服务器上的`www`目录。

    注意，此任务的文件配置支持所有标准的 Grunt 选项。您可以在以下 URL 中了解更多信息：

    [`gruntjs.com/configuring-tasks#files`](http://gruntjs.com/configuring-tasks#files)

1.  我们现在可以通过使用 `grunt ftp_push` 命令来运行此任务，它应该产生类似于以下内容的输出：

    ```js
    Running "ftp_push:myapp" (ftp_push) task
    >> [username] successfully authenticated!
    >> /www/index.html transferred successfully.
    >> FTP connection closed!

    ```

    ### 小贴士

    注意，`ftp_push` 任务不能自动创建目标目录，因此在运行此任务之前，FTP 服务器的根目录上需要存在 `www` 目录。

1.  我们的服务器上现在应该已经有了 `index.html` 文件，并且如果配置正确，应该可以通过互联网访问。将域名配置为指向以这种方式托管站点的操作超出了本书的范围。

# 通过 SFTP 部署

在本菜谱中，我们将使用由 `ssh (0.12.2)` 插件提供的 `sftp` 任务，通过 **SSH 文件传输协议**（**SFTP**）将文件上传到托管服务器。

SFTP 提供了与之前菜谱中讨论的常规 FTP 相同的功能，但通过利用 **安全壳**（**SSH**）网络协议的附加安全层提供了额外的安全优势。

## 准备工作

在本例中，我们将使用我们在 第一章 中创建的基本项目结构，即 *在项目中安装 Grunt* 菜谱，*使用 Grunt 开始*。如果您还不熟悉其内容，请务必参考它。

除了标准项目设置外，以下菜谱还需要在目标启用了 SSH 的服务器上现有的用户账户。这些凭证通常由托管服务提供商或负责维护目标服务器的系统管理员提供。

## 如何操作...

以下步骤将引导我们创建一个简单的 HTML 文档，并配置一个任务，使用 SFTP 将其上传到服务器：

1.  我们将按照 `Installing a plugin` 菜谱中提供的说明安装包含 `ssh` 插件的包。

1.  接下来，我们将在项目目录中创建一个名为 `index.html` 的简单 HTML 文档，并为其提供以下内容：

    ```js
    <html>
      <head>
        <title>MyApp</title>
      </head>
      <body>
        <h1>This is MyApp.</h1>
      </body>
    </html>
    ```

1.  现在，我们将添加以下 `sftp` 任务到我们的配置中，这表明我们希望将 `index.html` 文件上传到支持 SFTP 的特定主机：

    ```js
    sftp: {
      myapp: {
        options: {
          host: '[host]',
          username: '[username]',
          password: '[password]',
          path: 'www/'
        },
        files: [{
          src: 'index.html',
        }]
      }
    }
    ```

    ### 小贴士

    `username`、`password` 和 `host` 选项填充了占位符值，以表明您应该使用自己的特定访问凭证和托管服务器地址。

    您可能希望将 `username` 和 `password` 选项存储在本地文件中，而不是共享仓库中。请参考 第一章 中 *导入外部数据* 菜谱的示例，了解如何从外部文件导入配置。同时，请记住，您应该从项目的代码仓库中排除包含访问凭证的文件。

    `host` 选项用于指定您想要上传文件的服务器的互联网地址。如果您不确定这个值应该是多少，此类信息通常可以由托管提供商或负责维护相关服务器的系统管理员提供。

    `path` 选项是必需的，用于指定我们将上传的文件的目标目录。在我们的例子中，我们将指示文件应该上传到托管服务器上的 `www` 目录。

    注意，此任务的文件配置支持所有标准 Grunt 选项。您可以在以下网址了解更多信息：

    [`gruntjs.com/configuring-tasks#files`](http://gruntjs.com/configuring-tasks#files)

1.  我们现在可以通过使用 `grunt sftp` 命令来运行任务，它应该产生类似于以下内容的输出：

    ```js
    Running "sftp:myapp" (sftp) task
    Copied 1 files

    ```

    ### 小贴士

    注意，`sftp` 任务不能自动创建目标目录，因此在运行此任务之前，`www` 目录需要存在于 FTP 服务器的根目录。

1.  我们的 `index.html` 文件现在应该出现在我们的托管服务器上，如果配置正确，应该可以通过互联网访问。配置域名指向以这种方式托管站点超出了本书的范围。

## 更多...

`sftp` 任务为我们提供了几个有用的选项，可以与它的上传功能一起使用。我们将探讨如何使用私钥和密码短语以及使用 SSH 代理。

### 使用私钥和密码短语

如果我们想使用私钥和密码短语来访问我们的主机服务器，我们可以通过使用 `privateKey` 和 `passphrase` 选项来实现。在下面的例子中，我们将使用 `grunt.file.load` 函数从常规位置加载我们的私钥，并提供用于锁定它的密码短语。

```js
sftp: {
  myapp: {
    options: {
      host: '[host]',
      username: '[username]',
      privateKey: grunt.file.read('[path to home]/.ssh/id_rsa'),
 passphrase: '[passphrase]',
      path: 'www/'
    },
    files: [{
      src: 'index.html',
    }]
  }
}
```

### 使用 SSH 代理

如果我们经常使用 SSH 访问我们的服务器，我们可能最好在会话期间第一次解锁后使用 **SSH 代理** 来存储我们的未加密私钥。这将允许我们访问所有使用我们的 **公钥** 的服务，而无需在整个用户会话期间再次输入密码短语。

### 小贴士

大多数类 Unix（包括 OS X）操作系统应该默认安装并运行 SSH 代理，在这种情况下，以下示例应该无需任何初始步骤即可工作。Windows 用户将需要手动安装和配置 SSH 代理。

以下示例通过使用 `agent` 选项指定套接字文件路径来使用 SSH 代理：

```js
sftp: {
  myapp: {
    options: {
      host: '[host]',
      username: '[username]',
      agent: process.env.SSH_AUTH_SOCK,
      path: 'www/'
    },
    files: [{
      src: 'index.html',
    }]
  }
}
```

### 小贴士

大多数操作系统捆绑的 SSH 代理程序遵循使用 `SSH_AUTH_SOCK` 环境变量提供其运行套接字的约定。在我们的例子中，我们使用标准的 Node.js `process.env` 对象来检索此环境变量的值。

# 部署到 GitHub Pages

在这个配方中，我们将使用`gh-pages (0.10.0)`插件将我们的站点发布到**GitHub Pages**服务。

GitHub Pages 服务为 GitHub 用户提供了一种简单的方式来托管与他们自己、他们的组织或他们的项目相关的静态站点。该服务的核心是提供基于**Git**代码仓库托管的**GitHub**标准服务。

## 准备工作

在这个例子中，我们将使用我们在第一章中创建的基本项目结构，即*在项目中安装 Grunt*配方中的*使用 Grunt 入门*。如果您还不熟悉其内容，请务必参考。

除了标准的项目设置外，以下配方还需要一个具有适当公钥（SSH）关联的 GitHub 用户账户。

### 小贴士

请参考以下 URL 以获取有关为您的 GitHub 账户创建和设置 SSH 密钥的帮助：

[`help.github.com/articles/generating-ssh-keys/`](https://help.github.com/articles/generating-ssh-keys/)

我们还需要在 GitHub 上创建一个名为`myapp`的仓库，克隆它，并将其用作我们的项目文件夹。

### 小贴士

您可以在以下 URL 中了解更多关于如何在 GitHub 上创建和克隆仓库的信息：

[`help.github.com/articles/creating-a-new-repository/`](https://help.github.com/articles/creating-a-new-repository/)

[`help.github.com/articles/cloning-a-repository/`](https://help.github.com/articles/cloning-a-repository/)

## 如何操作...

以下步骤将指导我们创建一个简单的 HTML 文档，并配置一个将其发布到 GitHub Pages 站点上的任务：

1.  我们将按照第一章中*安装插件*配方提供的说明来安装包含`gh-pages`插件的包。

1.  接下来，我们将在`www`目录下创建一个简单的 HTML 文档，名为`index.html`，并为其提供以下内容：

    ```js
    <html>
      <head>
        <title>MyApp Project Site</title>
      </head>
      <body>
        <h1>Welcome!</h1>
        <h2>This is the MyApp project site.</h1>
      </body>
    </html>
    ```

1.  现在，我们将添加以下`gh-pages`任务到我们的配置中，并指出我们希望将`www`目录的内容上传到我们的项目仓库：

    ```js
    'gh-pages': {
      myapp: {
        options: {
          base: 'www'
        },
        src: '**/*'
      }
    }
    ```

    ### 小贴士

    `base`选项用于提供包含我们站点的目录。在我们的例子中，我们将使用`www`目录，因为这是我们创建示例 HTML 文件的地方。

    `src`选项用于提供应传输到目标仓库的基于基本目录的文件。在我们的例子中，我们将其设置为`**/*`，表示我们希望将基本目录中的所有文件传输。

1.  我们现在可以通过使用`grunt gh-pages`命令来运行任务，它应该产生类似于以下内容的输出：

    ```js
    Running "gh-pages:myapp" (gh-pages) task
    Cloning git@github.com:[name]/myapp.git into .grunt/grunt-gh-pages/gh-pages/myapp
    Cleaning
    Fetching origin
    Checking out origin/gh-pages
    Removing files
    Copying files
    Adding all
    Committing
    Pushing

    ```

1.  现在，我们可以通过导航到 GitHub 自动分配给我们的项目的`[name].github.io/myapp`域名来查看我们的项目站点。这样做应该看起来像以下这样：![如何操作...](img/image00276.jpeg)

# 使 AWS CloudFront 分发失效

在此配方中，我们使用`invalidate-cloudfront (0.1.6)`插件来使 AWS CloudFront 分发失效。

AWS CloudFront 服务为我们提供了一个简单的方法，通过全球边缘位置来分发我们的网站和应用程序的文件。这导致我们的目标受众无论在世界任何地方都能获得更快的响应时间。

将我们的文件托管在 CDN 上的副作用是，每当它们在源处更新时，更新可能需要一段时间才能反映在各个边缘位置，这可能会阻止我们的观众获取关键更新。然而，AWS CloudFront 确实允许我们通过使分发失效来指示我们希望刷新存储在 CDN 上的内容。

## 准备工作

在本例中，我们将使用我们在第一章“使用 Grunt 入门”配方中创建的基本项目结构。如果您还不熟悉其内容，请务必参考。

除了标准的项目设置外，此配方还需要设置一个具有 AWS 访问密钥的 AWS 用户。

### 小贴士

有关如何获取您的 AWS 访问凭证的详细信息，请参阅以下 URL：

[`docs.aws.amazon.com/general/latest/gr/getting-aws-sec-creds.html`](http://docs.aws.amazon.com/general/latest/gr/getting-aws-sec-creds.html)

还需要创建一个 AWS CloudFront 分发，以便我们可以在配方中使用它。在我们的示例中，我们将使其分发我们在本章前面“部署到 AWS S3”配方中创建的存储桶中的文件。

### 小贴士

有关如何创建 AWS CloudFront 分发的更多信息，请参阅以下 URL：

[`docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-web-creating.html`](http://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/distribution-web-creating.html)

## 如何操作...

以下步骤将指导我们创建和配置一个使特定 AWS CloudFront 分发失效的任务：

1.  我们将按照第一章“使用 Grunt 入门”配方中提供的说明，安装包含`invalidate-cloudfront`插件的包。

1.  现在，我们将添加以下`invalidate_cloudfront`任务到我们的配置中，并指示我们希望刷新在`www`目录中可以找到的每个文件：

    ```js
    invalidate_cloudfront: {
      myapp: {
        options: {
          key: '[key]',
          secret: '[secret]',
          distribution: '[distribution id]'
        },
        files: [{
          expand: true,
          cwd: 'www',
          src: '**/*'
        }]
      }
    }
    ```

    ### 小贴士

    `key`和`secret`选项填充了占位符值，以表明您应该使用自己的特定访问凭证。

    `key`选项应设置为为您的用户生成的**访问密钥 ID**，使用 AWS IAM 控制台。

    `secret`选项应设置为为在`key`选项中指定的特定访问密钥 ID 生成的秘密访问密钥。请注意，秘密访问密钥仅在创建访问密钥 ID 时显示，因此如果您第一次没有保存它，您将无法在 IAM 控制台中找到它。

    您可能希望将`key`和`secret`选项存储在本地文件中，而不是共享存储库中。请参考第一章中“导入外部数据”配方，了解如何从外部文件导入配置的示例。同时，请注意，您应该从项目的代码存储库中排除包含安全凭证的文件。

1.  我们现在可以通过使用`grunt invalidate_cloudfront`命令来运行任务，该命令应该产生类似于以下内容的输出：

    ```js
    Running "invalidate_cloudfront:myapp" (invalidate_cloudfront) task
    Invalidating 1 files: /index.html

    0 Completed and 0 In Progress invalidations on: [dist id]

    Creating invalidation for 1 files
    Invalidation created at https://cloudfront.amazonaws.com/2014-10-21/distribution/[dist id]/invalidation/[invalidation id]

    ```

1.  目标 CloudFront 分发现在应该正在被使无效。如果对底层内容进行了任何更改，它应该在 CloudFront CDN 的所有边缘位置上反映出来，这通常需要几分钟。

# 通过 SSH 运行命令

在本配方中，我们将使用`sshexec`任务，该任务由`ssh (0.12.2)`插件提供，通过 SSH 网络协议在远程服务器上运行命令。

在部署过程中，在远程服务器上运行命令可能成为必需，当我们运行命令时，我们希望确保它是安全进行的。SSH 协议是运行远程服务器命令的事实标准，部分原因是它通过加密通过网络发送的所有数据来提供改进的安全性。

## 准备工作

在本例中，我们将使用我们在第一章中“在项目中安装 Grunt”配方中创建的基本项目结构，即*使用 Grunt 入门*。如果您还不熟悉其内容，请务必参考。

除了标准项目设置外，以下配方还需要在目标启用了 SSH 的服务器上现有的用户账户。这些凭据通常由托管服务或维护目标服务器的管理员提供。

## 如何操作...

以下步骤将指导我们创建和配置一个任务，该任务将运行一个在远程服务器上打印日期和时间的命令：

1.  我们将按照第一章中“安装插件”配方中提供的说明来安装包含`ssh`插件的包。

1.  现在，我们将以下 `sshexec` 任务添加到我们的配置中，并设置它通过运行 `date` 命令在远程服务器上打印日期和时间：

    ```js
    sshexec: {
      date: {
        options: {
          host: '[host]',
          username: '[username]',
          password: '[password]'
        },
        command: 'date'
      }
    }
    ```

    ### 小贴士

    `username`、`password` 和 `host` 选项填充了占位符值，以表明您应该使用您自己的特定访问凭证和托管服务器。

    您可能希望将 `username` 和 `password` 选项存储在本地文件中，而不是共享存储库中。有关如何从外部文件导入配置的示例，请参阅第一章中的 *导入外部数据* 菜谱，*使用 Grunt 入门*。同时，请记住，您应该从项目的代码存储库中排除包含访问凭证的文件。

    使用 `host` 选项来指定您想要上传文件的服务器的 Internet 地址。如果您不确定这个值应该是多少，此类信息通常可以由托管提供商或负责维护相关服务器的系统管理员提供。

1.  我们现在可以通过使用 `grunt sshexec` 命令来运行此任务，它应该产生类似于以下内容的输出：

    ```js
    Running "sshexec:myapp" (sshexec) task
    Thu Jan 1 12:00:00 GMT 2015

    ```

1.  如果我们在运行命令后看到返回的日期和时间，那么我们已经成功在远程服务器上运行了命令。

## 还有更多...

`sshexec` 任务为我们提供了几个有用的选项，这些选项可以与其上传功能一起使用。我们将探讨如何使用私钥和密码短语以及使用 SSH 代理。

### 使用私钥和密码短语

如果我们想使用私钥和密码短语来访问我们的主机服务器，我们可以通过使用 `privateKey` 和 `passphrase` 选项来实现。在以下示例中，我们将使用 `grunt.file.load` 函数从常规位置加载我们的私钥，并提供用于锁定它的密码短语。

```js
sshexec: {
  date: {
    options: {
      host: '[host]',
      username: '[username]',
      privateKey: grunt.file.read('[path to home]/.ssh/id_rsa'),
 passphrase: '[passphrase]'
    },
    command: 'date'
  }
}
```

### 使用 SSH 代理

如果我们经常使用 SSH 访问我们的服务器，我们最好在会话期间第一次解锁后使用 SSH 代理来存储我们的未加密私钥。这将允许我们访问所有使用我们的公钥的服务，而无需在整个用户会话期间再次输入密码短语。

### 小贴士

大多数 *nix（这包括 OS X）操作系统应该默认安装并运行 SSH 代理，在这种情况下，以下示例应该无需任何初始步骤即可工作。Windows 用户将需要手动安装和配置 SSH 代理。

以下示例通过指定带有 `agent` 选项的套接字来使用 SSH 代理：

```js
sshexec: {
  date: {
    options: {
      host: '[host]',
      username: '[username]',
      agent: process.env.SSH_AUTH_SOCK
    },
    command: 'date'
  }
}
```

### 小贴士

大多数操作系统捆绑的 SSH 代理程序遵循使用 `SSH_AUTH_SOCK` 环境变量提供其运行套接字的约定。在我们的示例中，我们使用了标准的 Node.js `process.env` 对象来检索此环境变量的值。
