# 第八章：从智能手机监控和控制您的设备

在之前的章节中，我们已经看到了从 Web 界面控制的项目。现在在本章中，我们将看到如何从 Android 的本机应用程序中控制您的 Arduino 和树莓派，使用平台来创建应用程序以进行控制和监视。

在本章中，我们将看到使用 Android 工具的不同项目和应用程序，将涵盖的主题如下：

+   使用 APP Inventor 从智能手机控制继电器

+   在 Android Studio 中读取 JSON 响应使用以太网盾

+   从 Android 应用程序控制直流电机

+   使用您的树莓派 Zero 从 Android 控制输出

+   通过蓝牙使用树莓派控制输出

# 使用 APP Inventor 从智能手机控制继电器

在本节中，我们将看到如何使用**APP Inventor**创建一个 Android 应用程序来控制连接到 Arduino 板的继电器。

## 硬件要求

项目所需的硬件如下：

+   继电器模块

+   Arduino UNO 板

+   以太网盾

+   一些电缆

## 软件要求

项目所需的软件如下：

+   软件 Arduino IDE

+   您需要激活 Gmail 帐户

# 创建我们的第一个应用程序

App Inventor for Android 是由 Google 最初提供的开源网络应用程序，现在由麻省理工学院（MIT）维护。它允许初学者为 Android 操作系统（OS）创建软件应用程序。它使用图形界面，非常类似于 Scratch 和 StarLogo TNG 用户界面，允许用户拖放可视对象以创建可以在 Android 设备上运行的应用程序。在创建 App Inventor 时，Google 利用了在教育计算领域的重要先前研究，以及 Google 在在线开发环境方面的工作。

您无需在计算机上安装任何软件来执行 APP Inventor；您只需要您的 Gmail 帐户来访问 APP Inventor 界面。

要进入 APP Inventor，只需访问：[`appinventor.mit.edu/explore/`](http://appinventor.mit.edu/explore/)。

转到创建应用程序开始设计应用程序。

首先，我们需要一个 Gmail 帐户；我们需要创建如下图所示的文件：

![创建我们的第一个应用程序](img/B05170_08_01.jpg)

转到菜单**项目**和**开始新项目**：

![创建我们的第一个应用程序](img/B05170_08_02.jpg)

写下项目的名称：

![创建我们的第一个应用程序](img/B05170_08_03.jpg)

在下图中，我们将项目的名称写为**aREST**：

![创建我们的第一个应用程序](img/B05170_08_04.jpg)

按下**确定**，我们将看到项目已创建：

![创建我们的第一个应用程序](img/B05170_08_05.jpg)

## 设计界面

现在是时候看看如何创建应用程序的界面了，创建项目后，我们点击项目名称，然后会看到以下屏幕：

![设计界面](img/B05170_08_06.jpg)

在左侧的用户界面中（您可以看到所有对象），要将对象移动到主屏幕，只需拖动**Web Viewer**和**Button**，如下图所示：

![设计界面](img/B05170_08_07.jpg)

在上一张屏幕截图中，我们可以看到我们将用来控制 Arduino 板的应用程序界面。

## 使用 APP Inventor 与 Arduino 以太网盾通信

现在我们将看到如何通过以太网网络与 Arduino 通信应用程序。

在**Web Viewer**控件的属性中，我们将看到主页 URL：

![使用 APP Inventor 与 Arduino 以太网盾通信](img/B05170_08_08a.jpg)

在这两个控件中，我们有我们的 Arduino 以太网盾的 URL，我们将使用`RESTful`服务发出请求，并将从应用程序发送以下请求：

+   `http://192.168.1.110/digital/7/1`

+   `http://192.168.1.110/digital/7/0`

## APP Inventor 的代码

原始版本的块编辑器在单独的 Java 进程中运行，使用`Open Blocks Java`库创建可视块编程语言和编程。

当我们点击按钮时，我们有 APP 发明者的代码，为了做到这一点，你只需要做以下事情：

+   转到显示**块**的屏幕界面

+   每个按钮拖动`当...执行`模块

+   在刚刚拖动的模块内部，放置`Call...WebViewer.GoToUrl`模块

+   在模块的 URL 中，放置`WebViewer.HomeUrl`模块

关闭应用程序：

+   拖动`当...按钮点击时执行`模块

+   并在模块内部放置关闭应用程序模块

![APP Inventor 的代码](img/B05170_08_09.jpg)

当我们打开 Web 浏览器时，我们将得到以下结果：

![APP Inventor 的代码](img/B05170_08_10.jpg)

以下截图显示了应用程序在手机上运行的情况：

![APP Inventor 的代码](img/B05170_08_11.jpg)

以下图像显示了连接的最终结果：

![APP Inventor 的代码](img/B05170_08_12.jpg)

# 使用以太网盾在 Android Studio 中读取 JSON 响应

在本节中，我们将看到如何从 Arduino 板读取响应并在 Android Studio 中读取。

在我们继续下一部分之前，我们需要做以下事情：

+   安装 Android Studio 的 IDE，可以从以下网址获取：[`developer.android.com/studio/index.html?hl=es-419`](https://developer.android.com/studio/index.html?hl=es-419)

+   获取 Android Studio 的最新 SDK

然后我们将在 Android Studio 中创建一个项目，如下截图所示：

![使用以太网盾在 Android Studio 中读取 JSON 响应](img/B05170_08_13.jpg)

然后选择我们想要使用的 API 版本并单击**下一步**按钮：

![使用以太网盾在 Android Studio 中读取 JSON 响应](img/B05170_08_14.jpg)

然后选择**空白活动**并单击**下一步**按钮：

![使用以太网盾在 Android Studio 中读取 JSON 响应](img/B05170_08_15.jpg)

输入您的 Activity 和 Layout 的名称，然后单击**完成**按钮：

![使用以太网盾在 Android Studio 中读取 JSON 响应](img/B05170_08_16.jpg)

# Android 应用程序

在本节中，我们将看到 Android 应用程序。在您的文件夹中，打开关于 Android Studio 的项目文件。

这里是界面代码生成的 XML 代码：

```js
FrameLayout  

    android:id="@+id/container" 
    android:layout_width="match_parent" 
    android:layout_height="match_parent" 
    tools:context=".MainActivity"> 
    tools:ignore="MergeRootFrame"> 

    <WebView 
        android:id="@+id/activity_main_webview" 
        android:layout_width="match_parent" 
        android:layout_height="match_parent" /> 
</FrameLayout> 

```

## Java 类

当我们创建项目时，一些类会自动生成，我们将在以下行中看到：

1.  类的名称：

```js
        import android.webkit.WebView; 

```

1.  主类：

```js
        public class MonitoringTemperatureHumidity extends
          ActionBarActivity { 

            private WebView mWebView; 

```

在 Android 应用程序的这一部分中，我们请求值：

```js
mWebView.loadUrl("http://192.168.1.110/temperature");
mWebView.loadUrl("http://192.168.1.110/humidity");
super.onCreate(savedInstanceState);
setContentView(R.layout.activity_monitoring_temperature_humidity);
```

我们定义将包含在主活动中的对象，在这种情况下是`mWebView`控件，它在应用程序的主活动中定义：

```js
    mWebView = (WebView)  findViewById(R.id.activity_main_webview);
    mWebView.loadUrl("http://192.168.1.110/humidity");
}
```

## 应用程序的权限

为了给予应用程序执行网络权限的权限，我们需要在 Android 清单文件中添加以下行：

```js
<uses-permission android:name="android.permission.INTERNET"/>
```

当应用程序在设备上调试和安装后，我们将在屏幕上看到以下结果，显示`温度`的值：

![应用程序的权限](img/B05170_08_17.jpg)

`湿度`的值：

![应用程序的权限](img/B05170_08_18.jpg)

# 使用 Android 应用程序控制直流电机

在本节中，我们将有一个应用程序来将我们的智能手机与手机的蓝牙连接起来，它被称为**Amarino**，你可以从以下网址获取：[`www.amarino-toolkit.net/index.php/home.html`](http://www.amarino-toolkit.net/index.php/home.html)。我们还将看到如何从 Android 应用程序控制直流电机，让我们深入研究一下！

## 硬件要求

在下图中，我们看到以下电路（L293D）用于控制电机的速度和转向：

![硬件要求](img/B05170_08_19.jpg)

在下图中，我们有电路连接到 Arduino 板的最终连接：

![硬件要求](img/B05170_08_20.jpg)

最终界面显示在以下截图中：

![硬件要求](img/B05170_08_21.jpg)

最终结果显示在以下图像中，带有连接：

![硬件要求](img/B05170_08_22.jpg)

# 使用 Raspberry Pi Zero 从 Android 控制输出

在本节中，我们将看到如何使用在`Node.js`服务器中运行的`control.js`脚本来控制连接到 Raspberry Pi 的输出。

我们需要使用 Android 应用程序控制 LED 输出的请求如下：

1.  `http://192.168.1.111:8099/ledon`

1.  `http://192.168.1.111:8099/ledoff`![使用 Raspberry Pi Zero 从 Android 控制输出](img/B05170_08_23.jpg)

在 APP Inventor 中创建的界面将类似于以下截图：

![使用 Raspberry Pi Zero 从 Android 控制输出](img/B05170_08_24.jpg)

最终电路连接将如以下截图所示：

![使用 Raspberry Pi Zero 从 Android 控制输出](img/B05170_08_25.jpg)

# 通过蓝牙使用 Raspberry Pi 控制输出

一旦您尝试与使用蓝牙模块连接到 Raspberry Pi 的串行端口的其他电子设备进行通信，情况就会有所不同。

这些模块非常便宜，实际模块是坐落在我的模型的插板上的绿色板。纯 HC-05 只能在*3.3V*电平上工作，而不能在*5V-TTL*电平上工作。因此，人们需要电平转换器（再次）。

在本节中，我们将 Raspberry Pi Zero 与蓝牙模块进行通信，并连接 Raspberry Pi 的**TX**和**RX**引脚。

首先，我们需要配置系统文件，进行一些更改以激活 Raspberry Pi Zero TX 和 RX 的通信：

![通过蓝牙使用 Raspberry Pi 控制输出](img/B05170_08_26.jpg)

## 从 Android 应用程序控制灯光

我们需要下载蓝牙终端，如下截图所示：

![从 Android 应用程序控制灯光](img/B05170_08_27.jpg)

以下截图显示了发送数字 1、2、3、4、5 和 6 的结果：

![从 Android 应用程序控制灯光](img/B05170_08_28.jpg)

以下图像显示了项目的最终部分以及与 HC05 模块和 Raspberry Pi Zero 的连接：

![从 Android 应用程序控制灯光](img/B05170_08_29.jpg)

# 总结

在本章中，您学会了如何使用 Android Studio 和 APP inventor 通过蓝牙和以太网通信从智能手机控制 Arduino 和 Raspberry Pi Zero。我们还研究了几个项目，如控制电机、控制继电器模块以及读取湿度和温度。对于未来的项目，您现在可以在应用程序的任何区域控制和监视任何您想要的东西。

在下一章中，我们将整合前几章的所有内容，并将所有知识应用到一起。
