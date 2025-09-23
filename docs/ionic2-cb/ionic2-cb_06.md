# 第六章：使用 Ionic Cloud 进行用户认证和推送通知

在本章中，我们将涵盖与用户认证、注册和接收推送通知消息相关的以下任务：

+   使用 Ionic Cloud 注册和验证用户

+   构建用于接收推送通知的 iOS 应用

+   构建用于接收推送通知的 Android 应用

# 简介

跟踪和吸引用户是您的应用增长的关键功能。这意味着您应该能够注册和验证用户。一旦用户开始使用应用，您还需要对用户进行细分，以便您可以定制他们的交互。然后，您可以发送推送通知来鼓励用户重新访问应用。

您的项目需要使用以下三个组件，如下所示：

+   **Ionic Cloud**：这是一个云服务，帮助您保存用户信息，并在您、苹果或谷歌与最终用户设备之间协调推送通知。

+   **Ionic Cloud Angular 模块**：这只是一个 Angular 模块，用于导入到您的本地项目中。它为您的代码提供了一些与 Ionic Cloud 交互的简单实用工具。否则，直接调用 Ionic Cloud API 会非常复杂。

+   **Cordova 推送通知和 InAppBrowser**：由于您的代码是用 JavaScript 编写的，您需要与设备的原生功能进行通信，例如推送通知。

# 使用 Ionic Cloud 注册和验证用户

Ionic Cloud 可以提供开箱即用的所有用户管理和认证功能。以下提供者是 Ionic Cloud 支持的：

+   电子邮件/密码

+   自定义认证

+   脸书

+   Google

+   Twitter

+   Instagram

+   领英

+   GitHub

根据应用的不同，您可能不需要使用所有这些认证方法。例如，对于专注于职业人士的应用，使用领英认证可能更有意义，以缩小符合应用用户画像的受众范围。如果您有自己的认证服务器，其中您维护自己的用户数据库，您仍然可以使用 Ionic Cloud 的自定义认证来创建自定义令牌。

本章将尽可能简化认证概念。您将学习如何完成以下事情：

+   注册新用户

+   登录和注销用户

+   使用自定义字段更改用户个人资料数据

观察以下应用的截图：

![使用 Ionic Cloud 注册和验证用户](img/image00269.jpeg)

## 准备工作

您可以通过浏览器运行此应用。无需使用物理设备测试用户认证。

## 如何操作...

观察以下说明：

1.  使用 `blank` 模板创建一个新的 `MySimpleAuth` 应用，如图所示，并转到 `MySimpleAuth` 文件夹：

    ```js
    $ ionic start MySimpleAuth blank --v2
    $ cd MySimpleAuth

    ```

1.  使用以下命令安装 Ionic Cloud Angular：

    ```js
    $ npm install @ionic/cloud-angular --save

    ```

1.  初始化您的 Ionic Cloud 设置，以便在您的账户中创建一个应用 ID，如下所示：

    ```js
    $ ionic io init

    ```

    ### 注意

    执行此命令行时，将提示你登录到你的 Ionic Cloud 账户。初始化过程会将 `app_id` 和 `api_key` 值保存到你的项目 `ionic.io.bundle.min.js` 中。这意味着在此之后你不能更改你的 Ionic Cloud 账户中的项目（否则你将不得不手动删除 ID 并重新初始化）。你的应用 ID 也记录在 `ionic.config.json` 文件中。

1.  登录到你的 [`apps.ionic.io`](https://apps.ionic.io) 的 Ionic Cloud。

1.  识别你初始化的新应用并复制应用 ID（即 `7588ce26`），如下截图所示：![如何做...](img/image00270.jpeg)

1.  使用以下代码打开并编辑 `./src/app/app.module.ts`：

    ```js
    import { NgModule } from '@angular/core';
    import { IonicApp, IonicModule } from 'ionic-angular';
    import { CloudSettings, CloudModule } from '@ionic/cloud-angular';
    import { MyApp } from './app.component';
    import { HomePage } from '../pages/home/home';

    const cloudSettings: CloudSettings = {
      'core': {
        'app_id': '288db316'
      }
    };

    @NgModule({
      declarations: [
        MyApp,
        HomePage
      ],
      imports: [
        IonicModule.forRoot(MyApp),
        CloudModule.forRoot(cloudSettings)
      ],
      bootstrap: [IonicApp],
      entryComponents: [
        MyApp,
        HomePage
      ],
      providers: []
    })
    export class AppModule {} 
    ```

    ### 注意

    你需要确保 `app_id` 在你的情况下具有正确的值。

1.  使用以下代码编辑并替换 ./src/pages/home/home.html：

    ```js
    <ion-header>
      <ion-navbar>
        <ion-title>
          User Authentication
        </ion-title>
      </ion-navbar>
    </ion-header>

    <ion-content padding>

      <div *ngIf="!auth.isAuthenticated()">

        <h4>
          Register or Login
        </h4>

        <ion-list>

          <ion-item>
            <ion-label fixed>Email</ion-label>
            <ion-input type="text" [(ngModel)]="email"></ion-input>
          </ion-item>

          <ion-item>
            <ion-label fixed>Password</ion-label>
            <ion-input type="password" [(ngModel)]="password"></ion-input>
          </ion-item>

        </ion-list>

        <ion-grid>
          <ion-row>
            <ion-col width-50>
              <button ion-button block (click)="register()">Register</button>
            </ion-col>
            <ion-col width-50>
              <button ion-button block (click)="login()">Login</button>
            </ion-col>
          </ion-row>
        </ion-grid>

      </div>

      <div *ngIf="auth.isAuthenticated()">

        <h4>
          User Profile
        </h4>

        <ion-list *ngIf="auth.isAuthenticated()">

          <ion-item>
            <ion-label fixed>Name</ion-label>
            <ion-input class="right" type="text" [(ngModel)]="name"></ion-input>
          </ion-item>

          <ion-item>
            <ion-label>Birthday</ion-label>
            <ion-datetime displayFormat="MM/DD/YYYY" [(ngModel)]="birthday"></ion-datetime>
          </ion-item>

        </ion-list>

        <ion-grid>
          <ion-row>
            <ion-col width-50>
              <button ion-button color="secondary" block (click)="save()">Save</button>
            </ion-col>
            <ion-col width-50>
              <button ion-button color="dark" block (click)="logout()">Logout</button>
            </ion-col>
          </ion-row>
        </ion-grid>

      </div>

    </ion-content>
    ```

    ### 注意

    这些只是基本的 `登录` 和 `注销` 模板。所有内容都在一个页面上以保持简单。

1.  使用以下代码打开并编辑 `./src/pages/home/home.ts`：

    ```js
    import { Component } from '@angular/core';
    import { NavController, LoadingController, ToastController } from 'ionic-angular';
    import { Auth, User, UserDetails, IDetailedError } from '@ionic/cloud-angular';

    @Component({
      templateUrl: 'home.html'
    })
    export class HomePage {
      public email: string = "";
      public password: string = "";
      public name: string = "";
      public birthday: string = "";

      constructor(public navCtrl: NavController, public auth: Auth, public user: User, public loadingCtrl: LoadingController, public toastCtrl: ToastController) {
        this.initProfile();
      }

      private initProfile() {
        if (this.auth.isAuthenticated()) {
          this.name = this.user.get('name', '');
          this.birthday = this.user.get('birthday', '');
        }
      }

      register() {
        let details: UserDetails = {'email': this.email, 'password': this.password};

        let loader = this.loadingCtrl.create({
          content: "Registering user..."
        });
        loader.present();    

        this.auth.signup(details).then(() => {
          console.log('User is now registered');
          console.log(this.user);
          loader.dismiss();
          return this.auth.login('basic', {'email': this.email, 'password': this.password});

        }, (err: IDetailedError<string[]>) => {
          loader.dismiss();

          for (let e of err.details) {
            if (e === 'conflict_email') {
              alert('Email already exists.');
            } else {
              alert('Error creating user.');
            }
          }
        });
      }

      login() {
        let details: UserDetails = {'email': this.email, 'password': this.password};

        let loader = this.loadingCtrl.create({
          content: "Logging in user..."
        });
        loader.present();    

        this.auth.login('basic', details).then((data) => {
          console.log('Finished login');
          this.initProfile();

          loader.dismiss();

          console.log(data);
          console.log(this.user);
        }, (err) => {
          loader.dismiss();
          alert('Login Error');
        } );    
      }

      save() {
        let toast = this.toastCtrl.create({
          message: 'User profile was saved successfully',
          position: 'bottom',
          duration: 3000
        });
        toast.present();

        this.user.set('name', this.name);
        this.user.set('birthday', this.birthday);
        this.user.save();
      }

      logout() {
        this.auth.logout();
        this.email = '';
        this.password = '';
        this.name = '';
        this.birthday = '';
      }

    }
    ```

    ### 注意

    上述代码提供了四种方法来注册、保存数据、登录和注销用户。

1.  最后，在 `./src/pages/home/home.scss` 中添加一个小的 `input` 字段对齐，如下所示：

    ```js
    .home-page {
      ion-input.right > input {
        text-align: right;
      }
    }
    ```

1.  前往你的终端并运行应用：

    ```js
    $ ionic serve

    ```

## 如何工作...

简而言之，Ionic Cloud 作为你的应用的后端服务器。它允许你在其数据库中创建新的用户记录。通过 `user` 类，你可以与 Ionic Cloud 认证系统交互。第一步是注册用户。这需要一个 **电子邮件** 和 **密码**，如下截图所示：

![如何工作...](img/image00271.jpeg)

当你点击 **注册** 时，以下代码将被执行：

```js
this.auth.signup(details).then(() => {
  console.log('User is now registered');
  console.log(this.user);
  loader.dismiss();
  return this.auth.login('basic', {'email': this.email, 'password': this.password});
}
```

`details` 对象包含表单中的电子邮件和密码。一旦成功完成，它将通过 `this.auth.login` 自动登录用户。

注意，这里的这段代码只是为了显示加载屏幕，防止用户多次点击注册按钮：

```js
let loader = this.loadingCtrl.create({
  content: "Registering user..."
});
loader.present();  
```

此示例应用仅包含 **姓名** 和 **生日** 作为自定义用户数据，如下截图所示：

![如何工作...](img/image00272.jpeg)

要保存 **用户资料**，你调用 `save()` 方法，如下代码所示：

```js
save() {
  let toast = this.toastCtrl.create({
    message: 'User profile was saved successfully',
    position: 'bottom',
    duration: 3000
  });
  toast.present();

  this.user.set('name', this.name);
  this.user.set('birthday', this.birthday);
  this.user.save();
}
```

如果你查看控制台日志，用户令牌和自定义数据也都可以看到，如下截图所示：

![如何工作...](img/image00273.jpeg)

你也可以在 Ionic Cloud 门户中查看 **UserData**。登录到你的账户并导航到应用的 **认证** 菜单。

点击你创建的用户旁边的 **查看** 按钮，如下截图所示：

![如何工作...](img/image00274.jpeg)

选择 **自定义数据** 选项卡，你可以看到为用户存储的相同信息，如下截图所示：

![如何工作...](img/image00275.jpeg)

如需了解更多关于用户认证的信息，你可能需要参考官方的 Ionic 文档，网址为 [`docs.ionic.io/services/users/`](http://docs.ionic.io/services/users/)。

# 构建一个用于接收推送通知的 iOS 应用

推送通知是吸引用户频繁参与的重要功能，尤其是在用户没有使用应用时。许多人下载了应用，但只打开了几次。如果你向他们发送推送通知消息，这将鼓励他们打开应用参与新的活动。如果你必须从头开始构建一切，实现推送通知非常复杂。然而，Ionic 通过利用 Cordova 推送通知插件和 Ionic Cloud 作为提供商，使这个过程变得非常简单。推送通知提供商是一个可以与 **Apple 推送通知服务** (**APNs**) 或 Google 的 **Firebase 云消息** (**FCM**) 通信的服务器。你可以使用现有的开源软件设置自己的提供商服务器，但你必须单独维护此服务器，并跟上 APN API 的潜在变化。

在本节中，你将学习以下内容：

+   设置 Ionic Cloud 以支持 iOS 推送通知

+   配置 iOS 应用、证书（应用和推送）以及配置文件

+   编写代码以接收推送通知

以下是在接收了几条通知消息后应用的截图：

![构建一个用于接收推送通知的 iOS 应用](img/image00276.jpeg)

## 准备工作

为了测试通知消息，需要有一个可用的物理 iOS 设备。

你还必须注册 **Apple 开发者计划** (**ADP**)，以便访问 [`developer.apple.com`](https://developer.apple.com) 和 [`itunesconnect.apple.com`](https://itunesconnect.apple.com)，因为这些网站将需要经过批准的账户。

此外，以下说明使用了这些组件的具体版本：

+   Mac OSX El Capitan 10.11.4

+   Xcode 7.3.1

+   Ionic CLI 2.1.8

+   Cordova 6.4.0

+   Node 6.8.1

+   NPM 3.10.8

## 如何操作...

仔细阅读以下说明：

1.  使用 `blank` 模板创建一个新的 `MyiOSPush` 应用，如图所示，然后进入 `MyiOSPush` 文件夹：

    ```js
    $ ionic start MyiOSPush blank --v2
    $ cd MyiOSPush

    ```

1.  安装 Ionic `cloud-angular` 客户端，这是一个用于与 `push` 对象交互的库，安装方法如下：

    ```js
    $ npm install @ionic/cloud-angular --save

    ```

    ### 注意

    你需要拥有 Node 版本 4.x 或更高版本以及 NPM 版本 3.x 或更高版本。

1.  按照图示初始化你的 Ionic Cloud 设置，以便在你的账户中创建一个应用 ID：

    ```js
    $ ionic io init

    ```

    ### 注意

    在此命令行中，你将被提示登录你的 Ionic Cloud 账户。初始化过程将保存 `app_id` 和 `api_key` 值到你的项目的 `ionic.io.bundle.min.js` 文件中。这意味着在此之后你不能更改你的 Ionic Cloud 账户中的项目（否则你将不得不手动删除 ID 并重新初始化）。你的应用 ID 也记录在 `ionic.config.json` 文件中。

1.  您需要安装 Cordova 推送通知插件，并提供一些值作为`SENDER_ID`。由于您仅使用 iOS 推送通知，您可以在这里提供一个假值，暂时如下所示：![如何操作...](img/image00290.jpeg)

    ```js
    $ cordova plugin add phonegap-plugin-push --variable SENDER_ID=12341234 --save

    ```

1.  在项目文件夹中打开您的`./ionic.config.json`文件，并复制`app_id`值。在这种情况下，值是`00f293c4`，如下所示：

    ```js
    {
      "name": "MyiOSPush",
      "app_id": "00f293c4",
      "v2": true,
      "typescript": true
    }
    ```

1.  使用以下内容打开并编辑`./src/app/app.module.ts`：

    ```js
    import { NgModule } from '@angular/core';
    import { IonicApp, IonicModule } from 'ionic-angular';
    import { CloudSettings, CloudModule } from '@ionic/cloud-angular';
    import { MyApp } from './app.component';	
    import { HomePage } from '../pages/home/home';

    const cloudSettings: CloudSettings = {
      'core': {
        'app_id': '00f293c4'
      },
      'push': {
        'sender_id': 'SENDER_ID',
        'pluginConfig': {
          'ios': {
            'badge': true,
            'sound': true
          },
          'android': {
            'iconColor': '#343434'
          }
        }
      }
    };

    @NgModule({
      declarations: [
        MyApp,
        HomePage
      ],
      imports: [
        IonicModule.forRoot(MyApp),
        CloudModule.forRoot(cloudSettings)
      ],
      bootstrap: [IonicApp],
      entryComponents: [
        MyApp,
        HomePage
      ],
      providers: []
    })
    export class AppModule {}
    ```

    ### 注意

    您必须将`'app_id': '00f293c4'`替换为您自己的 App ID。

1.  访问 Apple 开发者网站[`developer.apple.com`](https://developer.apple.com)，并使用您的凭据登录。

1.  点击**证书、标识符和配置文件**，如图所示：![如何操作...](img/image00277.jpeg)

1.  选择您要针对的正确设备平台。在这种情况下，它将是**iOS、tvOS、watchOS**，如图所示：![如何操作...](img/image00278.jpeg)

1.  导航到**标识符** > **App ID**以创建一个 App ID，如图所示：![如何操作...](img/image00279.jpeg)

1.  点击屏幕右上角的加号(**+**)按钮，如图所示：![如何操作...](img/image00280.jpeg)

1.  填写表格以注册您的**App ID**。**名称**字段可以是任何内容，并且它与 Ionic Cloud 的 App ID 不同。您可以为您的项目（即`MyiOSPush`）提供名称以保持简单，如图所示：![如何操作...](img/image00281.jpeg)

1.  在这里需要正确操作的重要部分是**捆绑 ID**，因为它必须与`./config.xml`文件或 Xcode 中的捆绑标识符匹配，如图所示：![如何操作...](img/image00282.jpeg)

1.  要启用推送通知，您需要在以下页面检查**推送通知**服务：![如何操作...](img/image00283.jpeg)

1.  选择**注册**，如图所示：![如何操作...](img/image00284.jpeg)

1.  选择**完成**以完成创建**App ID**的步骤，如图所示：![如何操作...](img/image00285.jpeg)

1.  要开始证书创建，您需要在 Mac OSX 本地使用**钥匙串访问**生成证书签名请求文件。导航到**钥匙串访问**左上角的菜单，并导航到**证书助手** > **从证书颁发机构请求证书…**，如图所示：![如何操作...](img/image00286.jpeg)

1.  输入您的**用户电子邮件地址**和**通用名称**。将**CA 电子邮件地址**字段留空，并勾选**保存到磁盘**，如图所示：![如何操作...](img/image00287.jpeg)

1.  保存您的`CertificateSigningRequest.certSigningRequest`文件，如下所示：![如何操作...](img/image00288.jpeg)

1.  导航到 Apple 开发者网站，并导航到**证书** > **所有**，如图所示：![如何操作...](img/image00289.jpeg)

1.  点击右上角的加号按钮开始创建证书，如下所示：![如何操作...](img/image00290.jpeg)

1.  您只需按照网站上的步骤填写必要的信息。在这个例子中，您将选择**开发**版本而不是**生产**版本，如图所示：![如何操作...](img/image00291.jpeg)

1.  按照以下方式点击**继续**按钮以继续：![如何操作...](img/image00292.jpeg)

1.  按如图所示点击**选择文件…**按钮，上传您之前保存的签名请求文件：![如何操作...](img/image00293.jpeg)

1.  点击如图所示的**继续**按钮以继续：![如何操作...](img/image00294.jpeg)

1.  点击**下载**按钮以下载您的 iOS 开发证书文件：![如何操作...](img/image00295.jpeg)

1.  点击您下载的`.cer`文件，如图所示，以便将其导入到**钥匙串访问**：![如何操作...](img/image00296.jpeg)

1.  在**钥匙串访问**中找到您的证书，因为您需要将其导出为`.p12`文件格式。Ionic Cloud 稍后需要此文件以生成推送令牌并向应用发送推送通知。观察以下截图：![如何操作...](img/image00297.jpeg)

1.  右键单击并从下拉菜单中选择**导出**：![如何操作...](img/image00298.jpeg)

1.  按如图所示保存您的`Certificates.p12`文件，以便您可以在以后将其导入到 Ionic Cloud：![如何操作...](img/image00299.jpeg)

1.  如以下截图所示，为此文件提供密码以保护它：![如何操作...](img/image00300.jpeg)

    ### 注意

    在此步骤中，您必须提供密码，尽管在**钥匙串访问**中它是可选的。原因是 Ionic Cloud 无法在没有密码的情况下导入`.p12`文件。

1.  如果您需要将应用推送到特定设备，您必须注册该设备。转到**设备** > **所有**：![如何操作...](img/image00301.jpeg)

1.  点击加号按钮：![如何操作...](img/image00302.jpeg)

1.  提供设备的**UDID**并保存以注册设备。观察以下截图：![如何操作...](img/image00303.jpeg)

1.  您需要配置文件。导航到**配置文件** > **所有**：![如何操作...](img/image00304.jpeg)

1.  点击加号按钮：![如何操作...](img/image00305.jpeg)

1.  选择**iOS 应用开发**作为您的配置文件，因为此示例仅针对开发版本：![如何操作...](img/image00306.jpeg)

1.  点击**继续**按钮：![如何操作...](img/image00307.jpeg)

1.  在下拉菜单中选择正确的**App ID**并保存以完成您的配置文件创建：![如何操作...](img/image00308.jpeg)

1.  点击**继续**按钮：![如何操作...](img/image00309.jpeg)

1.  选择您之前创建的 iOS 开发证书，如图所示：![如何操作...](img/image00310.jpeg)

1.  如以下截图所示，选择至少一个您想要安装应用进行测试的设备：![如何操作...](img/image00311.jpeg)

1.  为你的配置文件提供**配置文件名称**，如图所示：![如何操作...](img/image00312.jpeg)

1.  点击**下载**按钮以下载配置文件文件（即`MyiOSPush_Provisioning_Profile.mobileprovision`）：![如何操作...](img/image00313.jpeg)

1.  点击你刚刚下载的`MyiOSPush_Provisioning_Profile.mobileprovision`，以便将其导入到 Xcode 中：![如何操作...](img/image00314.jpeg)

    ### 注意

    这一步非常重要，因为如果你没有将其导入到 Xcode 中，你的应用将无法成功构建。如果你的应用因为无效的配置文件而无法构建，最好在开发者控制台中检查配置文件状态。

1.  要启用**推送通知**功能，你必须请求一个**推送证书**，它与应用证书不同。选择你之前创建的**App ID**（即`MyiOSPush`）：![如何操作...](img/image00315.jpeg)

1.  点击页面底部的**编辑**按钮：![如何操作...](img/image00316.jpeg)

    ### 注意

    **推送通知**必须显示**可配置**状态。否则，你的应用将无法使用**推送通知**。

1.  点击**推送通知** > **开发 SSL 证书**部分下的**创建证书...**按钮：![如何操作...](img/image00317.jpeg)

1.  你将被带到新页面以创建你的 CSR 文件。点击**继续**按钮：![如何操作...](img/image00318.jpeg)

1.  点击底部的**选择文件...**按钮：![如何操作...](img/image00319.jpeg)

1.  找到你之前创建的`CertificateSigningRequest.certSigningRequest`文件：![如何操作...](img/image00320.jpeg)

    ### 注意

    你必须上传与应用证书相同的`.certSigningRequest`文件。否则，你的应用将无法接收推送通知，并且很难调试。

1.  点击**继续**按钮：![如何操作...](img/image00321.jpeg)

1.  点击**下载**按钮以下载证书文件。你可以将其命名为`aps_certificate.cer`以避免覆盖早期的`.cer`文件：![如何操作...](img/image00322.jpeg)

1.  下载完`.cer`文件后，你需要点击它以将其导入到**钥匙串访问**：![如何操作...](img/image00323.jpeg)

1.  在**钥匙串访问**中找到新的推送服务证书并选择它，如图所示：![如何操作...](img/image00324.jpeg)

1.  右键单击证书并选择**导出**：![如何操作...](img/image00325.jpeg)

1.  给它一个新的名称以避免覆盖应用证书。这个过程基本上是将`.cer`文件转换为`.p12`文件以供 Ionic Cloud 使用：![如何操作...](img/image00326.jpeg)

1.  为这个`.p12`文件提供密码以保护它：![如何操作...](img/image00327.jpeg)

    ### 注意

    `.p12`文件的密码是必需的，因为 Ionic Cloud 不会导入没有密码的 APN `.p12`文件。

1.  完成在 Apple 开发者网站上的设置后，您需要将配置文件和两个证书（应用和推送）上传到 Ionic Cloud。导航到[`apps.ionic.io`](https://apps.ionic.io)并使用您的凭据登录。

1.  选择为此项目生成的应用（即`MyiOSPush`）：![如何操作...](img/image00328.jpeg)

1.  导航到**设置** > **证书**：![如何操作...](img/image00329.jpeg)

1.  选择**新建安全配置文件**：![如何操作...](img/image00330.jpeg)

1.  提供一个**配置文件名称**并点击**创建**按钮以保存配置文件：![如何操作...](img/image00331.jpeg)

1.  选择您刚刚创建的`MyPushProfile`旁边的**编辑**以编辑设置：![如何操作...](img/image00332.jpeg)

1.  您需要上传三个文件：一个配置文件，一个用于应用本身的应用开发证书，以及一个用于推送通知的 APN 证书。让我们从应用要求开始。在**构建凭据**下的**选择文件**处点击，上传两个文件：`Certificates.p12`（第 30 步）和`MyiOSPush_Provisioning_Profile.mobileprovision`（第 44 步）。确保您提供与之前保护`.p12`文件相同的密码：![如何操作...](img/image00333.jpeg)

1.  在**推送通知服务**下点击**选择文件**并上传`push_Certificates.p12`文件（第 58 步）。确保您提供保护此文件的相同密码：![如何操作...](img/image00334.jpeg)

    ### 备注

    您不应混淆两个`.p12`文件，因为一个是您的应用，另一个是您的推送通知功能。

1.  点击**保存**按钮以保存您的安全配置文件。这完成了您的推送通知的 Ionic Cloud 设置。

1.  您需要修改主页代码以接收通知消息。打开并编辑`./src/pages/home/home.html`，然后粘贴以下代码：

    ```js
    <ion-header>
      <ion-navbar>
        <ion-title>
          Push Notification
        </ion-title>
      </ion-navbar>
    </ion-header>

    <ion-content padding>

      <code class="center">{{ pushToken }}</code>
      <button ion-button block [disabled]="clicked" [hidden]="pushToken" (click)="registerPush()">
        <span [hidden]="clicked">Register Push</span>
        <span [hidden]="!clicked">Registering...</span>
      </button>

      <h2 class="big-square" *ngIf="!pushToken">
        You have no message
      </h2>

      <h3 class="sub-title" *ngIf="pushToken">
        Your messages
      </h3>

      <ion-card *ngFor="let msg of messages">
        <ion-card-header>
          {{ msg.title }}
        </ion-card-header>
        <ion-card-content>
          {{ msg.text }}
        </ion-card-content>
      </ion-card>  
    </ion-content>
    ```

1.  将同一文件夹中的`home.ts`文件的内容替换为以下代码：

    ```js
    import { Component, ApplicationRef } from '@angular/core';
    import { NavController } from 'ionic-angular';
    import { Push, PushToken } from '@ionic/cloud-angular';

    @Component({
      templateUrl: 'home.html'
    })
    export class HomePage {
      public push: any;
      public pushToken: string;
      public messages = [];
      public clicked: Boolean = false;
      constructor(public navCtrl: NavController, push: Push, private applicationRef: ApplicationRef) {
        this.push = push;
      }

      private processPush(msg, that) {
        console.log('Push notification message received');
        console.log(msg);
        this.messages.push({
          title: msg.title,
          text: msg.text
        })
        this.applicationRef.tick();
      }

      registerPush() {
        this.clicked = true;

        this.push.register().then((t: PushToken) => {
          return this.push.saveToken(t);
        }).then((t: PushToken) => {

          this.push.rx.notification().subscribe(msg => this.processPush(msg, this));

          console.log('Token saved:', t.token);
          this.pushToken = t.token;
        }, (err) => {
          alert('Token error');
          console.log(err);
        });

      }  
    }
    ```

1.  将`/home`文件夹中的`home.scss`替换为以下代码：

    ```js
    .home-page {
      .center {
        text-align: center;
      }

      h2.big-square {
        text-align: center;
        padding: 50px;
        color: #D91E18;
        background: #F9BF3B;
      }

      h3.sub-title {
        text-align: center;
        padding: 10px;
        color: #446CB3;
        background: #E4F1FE;
      }

      ion-card ion-card-header {
        padding: 10px 16px;
        background: #F9690E;
        color: white;
      }

      ion-card ion-card-header + ion-card-content, ion-card .item + ion-card-content {
        padding-top: 16px;
      }
    }
    ```

1.  通过 USB 连接将您的物理 iPhone 连接到 Mac。

1.  确保您位于应用文件夹中，并针对 iOS 平台进行构建，如下所示：

    ```js
    $ ionic run ios --device

    ```

    ### 备注

    由于 Ionic 2 存在一个现有错误，您可能需要在末尾包含`--device`参数。

1.  操作系统将提示允许使用 iOS 开发者证书进行 codesign 签名。您必须接受此提示以允许访问，以便构建应用并将其上传到您的设备：![如何操作...](img/image00335.jpeg)

1.  验证应用是否在设备上成功运行。初始屏幕应如图所示：![如何操作...](img/image00336.jpeg)

到目前为止，您已完成了推送通知的设置和编码。下一步是验证您是否通过应用接收通知。以下是说明：

1.  在移动应用中点击 **注册推送** 按钮以向 Ionic Cloud 提供者服务器注册推送通知并获取令牌。点击 **确定** 接受接收推送通知的权限，如下所示：![如何操作...](img/image00337.jpeg)

1.  导航到 **Ionic Cloud** > 您的应用 (**MyiOSPush**) > **推送**：![如何操作...](img/image00338.jpeg)

1.  点击 **创建您的第一个推送** 按钮：![如何操作...](img/image00339.jpeg)

1.  填写推送通知表单以创建您的第一个推送消息：![如何操作...](img/image00340.jpeg)

1.  在右侧屏幕上验证以确保推送消息显示如预期：![如何操作...](img/image00341.jpeg)

1.  将段保留为 **所有用户**，以便任何人都可以接收推送：![如何操作...](img/image00342.jpeg)

1.  选择您之前创建的安全配置文件（即 `mypushprofile`）。然后，点击 **发送此推送** 按钮：![如何操作...](img/image00343.jpeg)

1.  验证从 Ionic Cloud 发送的推送通知是否成功。它应该有 **已发送** 状态，如下所示：![如何操作...](img/image00344.jpeg)

1.  在您的移动设备上的应用中，验证推送消息是否已按图示出现：![如何操作...](img/image00345.jpeg)

您已成功完成验证步骤。

## 它是如何工作的...

为了理解整个流程是如何工作的，让我们总结一下您所做的工作，如下所示：

+   创建了一个 Ionic 项目并将其初始化以创建一个 Ionic Cloud 项目：

    +   您本地的 Ionic 项目必须与 Ionic Cloud 项目同步，以便进行推送通知和其他管理，例如用户身份验证

+   通过以下步骤设置您的苹果开发者账户：

    +   创建了一个应用 ID

    +   创建了应用证书（在本地通过 **Keychain Access** 创建签名请求之后）

    +   创建了一个配置文件

    +   创建了一个推送证书

+   设置您的 Ionic Cloud 账户：

    +   创建了一个安全配置文件。

    +   从苹果开发者那里导入三个文件：一个应用证书、一个配置文件和一个推送证书。这些文件是必需的，以便 Ionic Cloud 可以成为受信任的提供者，与苹果的 APN 服务器通信以触发推送通知。

+   在您的应用中编写了接收通知的代码：

    +   您需要安装 Ionic Cloud Angular 库和 Cordova 推送通知插件。基本思想是使 `push` 对象可供应用使用。此 `push` 对象已配置为使用您的 Ionic Cloud 推送提供者。

现在，让我们专注于编码部分本身，以了解它是如何工作的。

您需要更改 `app.module.ts` 中的应用启动方式。这需要导入 `provideCloud` 和 `CloudSettings` 提供者，如下所示：

```js
import { provideCloud, CloudSettings } from '@ionic/cloud-angular';
```

除了将 `app_id` 设置为与 Ionic Cloud 中的项目 `app_id` 匹配之外，您还需要指定具有您想要为 iOS 和 Android 设置的参数的 `push` 对象，如下所示：

```js
const cloudSettings: CloudSettings = {
  'core': {
    'app_id': '00f293c4'
  },
  'push': {
    'sender_id': 'SENDER_ID',
    'pluginConfig': {
      'ios': {
        'badge': true,
        'sound': true
      },
      'android': {
        'iconColor': '#343434'
      }
    }
  }
};
```

然后，在 `NgModule` 内部，您需要插入以下行，以便 Ionic 知道它还需要初始化 Ionic Cloud：

```js
CloudModule.forRoot(cloudSettings)
```

在您的 `home.html` 模板中，有一个按钮可以通过调用 `registerpush()` 触发推送通知的注册：

```js
<code class="center">{{ pushToken }}</code>
<button ion-button block [disabled]="clicked" [hidden]="pushToken" (click)="registerPush()">
  <span [hidden]="clicked">Register Push</span>
  <span [hidden]="!clicked">Registering...</span>
</button>
```

此注册过程必须由用户手动干预，因为用户将需要在下一步接受权限。不建议在用户立即打开应用时就要求他们接受推送通知请求。主要原因是因为他们不熟悉您的应用，不知道会期待什么（即，他们是否会后来被通知轰炸）。

消息将通过 `messages` 对象显示，如下面的代码所示：

```js
<ion-card *ngFor="let msg of messages">
  <ion-card-header>
    {{ msg.title }}
  </ion-card-header>
  <ion-card-content>
    {{ msg.text }}
  </ion-card-content>
</ion-card>  
```

在这里，每个 `message` 项目都有 `title` 和 `text` 字段。

在 `home.ts` 文件中，有两个关键的导入项，您必须注意：`Push` 和 `PushToken` 是注册和接收推送通知所必需的。`ApplicationRef` 将在稍后讨论，因为您需要手动触发 Angular 模板的重新渲染，如下所示：

```js
import { Component, ApplicationRef } from '@angular/core';
import { NavController } from 'ionic-angular';
import { Push, PushToken } from '@ionic/cloud-angular';
The registerPush() method is the key to acquire the PushToken from Ionic Cloud, as shown:
registerPush() {
  this.clicked = true;

  this.push.register().then((t: PushToken) => {
    return this.push.saveToken(t);
  }).then((t: PushToken) => {

    this.push.rx.notification().subscribe(msg => this.processPush(msg, this));

    console.log('Token saved:', t.token);
    this.pushToken = t.token;
  }, (err) => {
    alert('Token error');
    console.log(err);
  });
}  
```

您只需要调用 `this.push.register()` 函数。这将返回一个 `PushToken` 对象，您可以在以下控制台日志的屏幕截图中看到：

![如何工作...](img/image00346.jpeg)

要接收通知，您需要使用以下代码进行订阅：

```js
this.push.rx.notification().subscribe()
```

这将在每次有新的通知消息时调用 `processPush()`，如下所示：

```js
private processPush(msg, that) {
  console.log('Push notification message received');
  console.log(msg);
  this.messages.push({
    title: msg.title,
    text: msg.text
  })
  this.applicationRef.tick();
}
```

当用户收到推送消息时，此函数将向 `messages` 数组中添加。如果您不调用 `this.applicationRef.tick()`，由于此过程在 Angular 生命周期之外，UI 将不会更新。如果您查看控制台日志，`PushMessage` 将如下所示，其中包含 `text` 和 `title` 字段：

![如何工作...](img/image00347.jpeg)

如果用户没有打开应用，您会看到通知出现在通知区域。

## 还有更多...

Ionic 有自己的 iOS 设置说明页面，如下所示：

+   [`docs.ionic.io/setup.html`](http://docs.ionic.io/setup.html)

+   [`docs.ionic.io/services/profiles/#ios-setup`](http://docs.ionic.io/services/profiles/#ios-setup)

+   [`docs.ionic.io/services/push/`](http://docs.ionic.io/services/push/)

Cordova 推送通知插件可以直接在 [`github.com/phonegap/phonegap-plugin-push`](https://github.com/phonegap/phonegap-plugin-push) 找到。

关于 Apple 推送通知服务的更多信息，您可以访问官方文档 [`developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html`](https://developer.apple.com/library/ios/documentation/NetworkingInternet/Conceptual/RemoteNotificationsPG/Chapters/ApplePushService.html)。

# 构建一个用于接收推送通知的 Android 应用

Google 推送通知与 iOS 的工作方式相同。但是，您将通过 Firebase Cloud Messaging 服务器进行操作，这是 Google Cloud Messaging（**GCM**）的新替代品。然而，Ionic Cloud 抽象了此过程，因此您无需使用不同的 API 进行编码。您将使用与 iOS 应用相同的 `push` 对象。

### 注意

有关 FCM 和 GCM 之间差异的更多信息，请访问[`firebase.google.com/support/faq`](https://firebase.google.com/support/faq)中的常见问题解答。

在本节中，您将学习如何完成以下事项：

+   设置 Android 推送通知的 Ionic Cloud

+   配置 Firebase 项目以使用推送 API

+   编写 Android 接收推送通知的代码

您将使用与您的 iOS 推送通知示例相同的代码库。主要区别在于在您的 Firebase 和 Ionic Cloud 账户中设置的过程。

## 准备工作

您可以使用 Android 模拟器测试 Android 推送通知。因此，无需物理 Android 设备。

您还必须注册 Firebase 以访问[`console.firebase.google.com`](https://console.firebase.google.com)。

此外，以下说明使用了这些组件的特定版本：

+   Mac OSX El Capitan 10.11.4

+   Ionic CLI 2.1.8

+   Cordova 6.4.0

+   Node 6.8.1

+   NPM 3.10.8

+   安装了至少 Android 5.1（Lollipop）的 Android Studio 2.x。

    观察以下内容

    ![准备就绪](img/image00348.jpeg)

+   Android SDK 工具、构建工具、平台工具和英特尔**硬件加速执行管理器**（**HAXM**）([`software.intel.com/en-us/android/articles/installation-instructions-for-intel-hardware-accelerated-execution-manager-windows`](https://software.intel.com/en-us/android/articles/installation-instructions-for-intel-hardware-accelerated-execution-manager-windows))。

    观察以下截图：

    ![准备就绪](img/image00349.jpeg)

+   至少已创建一个**Android 虚拟设备**（**AVD**）（使用 `$ android avd` 命令行打开 AVD 管理器）。观察以下截图：![准备就绪](img/image00350.jpeg)

## 如何操作...

这里是说明：

1.  使用 `blank` 模板创建一个新的 `MyAndroidPush` 应用，如下所示，并转到 `MyAndroidPush` 文件夹：

    ```js
    $ ionic start MyAndroidPush blank --v2
    $ cd MyAndroidPush

    ```

1.  安装 Ionic Cloud Angular 客户端库，以与 `push` 对象交互，如图所示：

    ```js
    $ npm install @ionic/cloud-angular --save

    ```

    ### 注意

    您需要安装 Node 版本 4.x 或更高版本以及 NPM 版本 3.x 或更高版本。

1.  初始化您的 Ionic Cloud 设置，以便在您的账户中创建一个应用 ID，如下所示：

    ```js
    $ ionic io init

    ```

    ### 注意

    在此命令行中，您将被提示登录到您的 Ionic Cloud 账户。

1.  您需要 Firebase 项目编号和 Firebase 服务器 ID 才能正确设置您的 `app.ts` 文件。首先，让我们登录到 Firebase 控制台[`console.firebase.google.com`](https://console.firebase.google.com)。

1.  点击**创建新项目**按钮，并填写一个项目名称（即`MyAndroidPush`）：![如何操作...](img/image00351.jpeg)

1.  导航到左侧导航菜单中的**增长** > **通知**：![如何操作...](img/image00352.jpeg)

1.  选择 Android 图标：![如何操作...](img/image00353.jpeg)

    ### 注意

    Firebase 云消息服务也支持 iOS 应用。因此，你可能可以在 iOS 和 Android 项目中都使用 FCM。

1.  在表单中提供**包名**。你可以从你的应用项目在`./config.xml`中的**包名**复制并粘贴：![如何操作...](img/image00354.jpeg)

1.  选择**继续**并保存 JSON 文件到某个位置。你不需要这个 JSON 文件用于 Ionic 项目：![如何操作...](img/image00355.jpeg)

1.  点击**完成**按钮以完成设置通知服务：![如何操作...](img/image00356.jpeg)

1.  现在，你需要**服务器密钥**和**发送者 ID**。导航到左上角的齿轮图标，并选择**项目设置**菜单项：![如何操作...](img/image00357.jpeg)

1.  选择**云消息**选项卡：![如何操作...](img/image00358.jpeg)

1.  复制**服务器密钥**和**发送者 ID**（如果使用 Google Cloud Messaging，则与**项目 ID**相同）：![如何操作...](img/image00359.jpeg)

1.  返回到终端并确保你处于 Ionic 项目文件夹中。

1.  你需要安装 Cordova 推送通知插件，并提供与之前命令行中的`SENDER_ID`值相同的**发送者 ID**，如下所示：

    ```js
    $ cordova plugin add phonegap-plugin-push --variable SENDER_ID=749277510481 --save

    ```

1.  在项目文件夹中打开你的`./ionic.config.json`文件，并从以下代码中复制`app_id`值（在这个例子中，值是`e546b9f6`）：

    ```js
    {
      "name": "MyAndroidPush",
      "app_id": "e546b9f6",
      "v2": true,
      "typescript": true
    }
    ```

1.  使用以下内容打开并编辑`./src/app/app.module.ts`：

    ```js
    import { NgModule } from '@angular/core';
    import { IonicApp, IonicModule } from 'ionic-angular';
    import { MyApp } from './app.component';

    import { CloudSettings, CloudModule } from '@ionic/cloud-angular';

    import { HomePage } from '../pages/home/home';

    const cloudSettings: CloudSettings = {
      'core': {
        'app_id': 'e546b9f6'
      },
      'push': {
        'sender_id': '749277510481',
        'pluginConfig': {
          'ios': {
            'badge': true,
            'sound': true
          },
          'android': {
            'iconColor': '#343434'
          }
        }
      }
    };

    @NgModule({
      declarations: [
        MyApp,
        HomePage
      ],
      imports: [
        IonicModule.forRoot(MyApp),
        CloudModule.forRoot(cloudSettings)
      ],
      bootstrap: [IonicApp],
      entryComponents: [
        MyApp,
        HomePage
      ],
      providers: []
    })
    export class AppModule {}
    ```

    ### 注意

    你必须将`'app_id': '00f293c4'`替换为你自己的应用 ID。此外，你还需要在这里再次提供**发送者 ID**。

1.  你的主页代码与 iOS 推送示例非常相似。打开并编辑`./src/pages/home/home.html`，并粘贴以下代码：

    ```js
    <ion-header>
      <ion-navbar>
        <ion-title>
          Push Notification
        </ion-title>
      </ion-navbar>
    </ion-header>

    <ion-content padding>

      <code class="center">{{ pushToken }}</code>
      <button ion-button block [disabled]="clicked" [hidden]="pushToken" (click)="registerPush()">
        <span [hidden]="clicked">Register Push</span>
        <span [hidden]="!clicked">Registering...</span>
      </button>

      <h2 class="big-square" *ngIf="!pushToken">
        You have no message
      </h2>

      <h3 class="sub-title" *ngIf="pushToken">
        Your messages
      </h3>

      <ion-card *ngFor="let msg of messages">
        <ion-card-header>
          {{ msg.title }}
        </ion-card-header>
        <ion-card-content>
          {{ msg.text }}
        </ion-card-content>
      </ion-card>  
    </ion-content>
    ```

1.  将同一文件夹中的`home.ts`文件的内容替换为以下代码：

    ```js
    import { Component, ApplicationRef } from '@angular/core';
    import { NavController } from 'ionic-angular';
    import { Push, PushToken } from '@ionic/cloud-angular';

    @Component({
      templateUrl: home.html'
    })
    export class HomePage {
      public push: any;
      public pushToken: string;
      public messages = [];
      public clicked: Boolean = false;
      constructor(public navCtrl: NavController, push: Push, private applicationRef: ApplicationRef) {
        this.push = push;
      }

      private processPush(msg, that) {
        console.log('Push notification message received');
        console.log(msg);
        this.messages.push({
          title: msg.title,
          text: msg.text
        })
        this.applicationRef.tick();
      }

      registerPush() {
        this.clicked = true;

        this.push.register().then((t: PushToken) => {
          return this.push.saveToken(t);
        }).then((t: PushToken) => {

          this.push.rx.notification().subscribe(msg => this.processPush(msg, this));

          console.log('Token saved:', t.token);
          this.pushToken = t.token;
        }, (err) => {
          alert('Token error');
          console.log(err);
        });

      }  
    }
    ```

1.  将`home.scss`替换为以下代码，也在`/home`文件夹中：

    ```js
    .home-page {
      .center {
        text-align: center;
      }

      h2.big-square {
        text-align: center;
        padding: 50px;
        color: #D91E18;
        background: #F9BF3B;
      }

      h3.sub-title {
        text-align: center;
        padding: 10px;
        color: #446CB3;
        background: #E4F1FE;
      }

      ion-card ion-card-header {
        padding: 10px 16px;
        background: #F9690E;
        color: white;
      }

      ion-card ion-card-header + ion-card-content, ion-card .item + ion-card-content {
        padding-top: 16px;
      }
    }
    ```

1.  如果你第一次创建此应用，你必须有一个`keystore`文件。此文件用于识别你的应用以进行推送和发布。如果你丢失它，你以后无法更新你的应用。要创建`keystore`，请输入以下命令行，并确保它与 SDK 的`keytool`版本相同（即检查你的`PATH`环境变量）：

    ```js
    $ keytool -genkey -v -keystore MyAndroidPushKey.keystore -alias MyAndroidPushKey -keyalg RSA -keysize 2048 -validity 10000

    ```

1.  一旦你在命令行中填写了信息，请将此文件的副本保存在安全的地方，因为你以后需要它。观察以下代码：

    ```js
    Enter keystore password:
    Re-enter new password:
    What is your first and last name?
     [Unknown]:  Hoc Phan
    What is the name of your organizational unit?
     [Unknown]:
    What is the name of your organization?
     [Unknown]:
    What is the name of your City or Locality?
     [Unknown]:
    What is the name of your State or Province?
     [Unknown]:
    What is the two-letter country code for this unit?
     [Unknown]:
    Is CN=Hoc Phan, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown correct?
     [no]:  yes

    Generating 2,048 bit RSA key pair and self-signed certificate (SHA256withRSA) with a validity of 10,000 days
     for: CN=Hoc Phan, OU=Unknown, O=Unknown, L=Unknown, ST=Unknown, C=Unknown
    Enter key password for <MyAndroidPushKey>
     (RETURN if same as keystore password):
    [Storing MyAndroidPushKey.keystore]

    ```

    ### 注意

    你必须为此步骤提供密码。否则，Ionic Cloud 不会让你上传`keystore`文件。

1.  现在，登录到你的 Ionic Cloud [`apps.ionic.io`](https://apps.ionic.io)以配置安全配置文件。

1.  导航到你的项目（即 `MyAndroidPush`）并选择**设置**菜单：![如何操作...](img/image00360.jpeg)

1.  选择**证书**，如下所示：![如何操作...](img/image00361.jpeg)

1.  创建一个新的**安全配置文件**并提供一个名称（例如，`MyAndroidPush`）：![如何操作...](img/image00362.jpeg)

1.  选择**编辑**，如图所示：![如何操作...](img/image00363.jpeg)

1.  上传你的 `keystore` 文件（例如，`MyAndroidPushKe``y.keystore`），别名（例如，`MyAndroidPushKey`），以及步骤 22 中的密码：![如何操作...](img/image00364.jpeg)

1.  将步骤 13 中的 **Server key** 粘贴到 **GCM API Key** 输入框中：![如何操作...](img/image00365.jpeg)

1.  选择**保存**以完成创建安全配置文件。

1.  返回终端。

1.  确保你处于应用文件夹中，并针对 Android 平台进行构建，如下所示：

    ```js
    $ ionic emulate android

    ```

验证 Android 推送通知的过程与 iOS 非常相似。以下是说明：

1.  导航到 **Ionic Cloud** > 你的应用（**MyAndroidPush**）> **推送**：![如何操作...](img/image00338.jpeg)

1.  点击**创建第一个推送**按钮：![如何操作...](img/image00339.jpeg)

1.  从创建第一个推送消息填写推送通知表单：![如何操作...](img/image00366.jpeg)

1.  将段设置为**所有用户**，以便任何人都可以接收推送：![如何操作...](img/image00342.jpeg)

1.  选择你之前创建的安全配置文件（即 `MyAndroidPush`）。然后，点击**发送此推送**按钮。观察以下截图：![如何操作...](img/image00343.jpeg)

1.  验证从 Ionic Cloud 发送的推送通知是否成功。它应该有**已发送**状态，如图所示：![如何操作...](img/image00367.jpeg)

1.  在你的模拟器设备上的应用中，验证推送消息是否已出现，如图所示：![如何操作...](img/image00368.jpeg)

你已成功完成验证步骤。

## 它是如何工作的...

从高层次来看，这是幕后通信过程：

+   你的设备向谷歌的 Firebase 服务器发送推送通知的注册请求。你的设备的应用程序必须具备以下内容：

    +   Firebase **项目 ID**

    +   Firebase **发送者 ID**

+   谷歌将回复注册 ID（它与 Ionic `push` 对象的推送令牌相同）。

+   你的应用将向 Ionic Cloud（或任何推送提供者服务器）发送注册 ID。此过程在幕后进行，因为 Ionic Cloud Angular 调用 Ionic Cloud API 来执行。然而，你的 Ionic Cloud 必须有一个包含以下列出的四条信息的安全配置文件：

    +   Firebase 的服务器密钥

    +   密钥库

    +   密钥库别名

    +   密钥库密码

+   Ionic Cloud 将保存此注册 ID 以请求谷歌稍后发送推送通知。您可以通过 Ionic Cloud UI 触发通知。

这与 Ionic Cloud 与苹果的合作方式非常相似。所有内容都简化为与 Ionic Cloud Angular 模块中的`push`对象交互。

要接收控制台输出，您可以导航到谷歌的 URL [chrome://inspect/#devices](http://chrome://inspect/#devices)。这将提供可用的模拟器列表以进行调试。点击**检查**链接以打开谷歌开发者工具：

![如何工作...](img/image00369.jpeg)

您应该能够看到以下相同的屏幕：

![如何工作...](img/image00370.jpeg)

应用程序输出一个令牌（即注册 ID），每个推送通知都将是一个`PushMessage`对象。您可以打开并检查其属性。

总结来说，Android 推送通知之所以工作得很好，是因为 Ionic Cloud、Ionic Angular 模块和 Firebase 云消息传递之间的即插即用集成。

## 还有更多...

Ionic 有自己的 Android 设置说明页面，如下所示：

+   [`docs.ionic.io/setup.html`](http://docs.ionic.io/setup.html)

+   [`docs.ionic.io/services/profiles/#android-app-keystore`](http://docs.ionic.io/services/profiles/#android-app-keystore)

+   [`docs.ionic.io/services/push/`](http://docs.ionic.io/services/push/)

关于 Firebase 通知服务的更多信息，您可以访问官方文档[`firebase.google.com/docs/cloud-messaging/`](https://firebase.google.com/docs/cloud-messaging/)。
