# 第一章. 使用 Raspberry Pi Zero 入门

在为家庭安全系统和通过电子控制系统控制家用电器构建几个项目之前，在本章中，我们将进行初始配置并准备我们的 Raspberry Pi Zero 在网络中工作，以便您可以在本书中看到的所有项目中使用它。

在我们进行项目、构建网络与设备并将传感器连接到板子之前，了解 Raspberry Pi 的配置是很重要的。本章的主要目的是解释如何设置您的 Raspberry Pi Zero；我们将涵盖以下主题：

+   设置 Raspberry Pi Zero

+   准备 SD 卡

+   安装 Raspbian 操作系统

+   使用串行控制台电缆配置您的 Raspberry Pi Zero

+   远程访问网络

+   通过远程桌面访问

+   配置 Web 服务器

# 设置 Raspberry Pi Zero

Raspberry Pi 是一个专门用于项目的低成本板。在这里，我们将使用 Raspberry Pi Zero 板。查看以下链接：[`www.adafruit.com/products/2816`](https://www.adafruit.com/products/2816)。我用了这块板。

为了使 Raspberry Pi 工作，我们需要一个充当硬件和用户之间桥梁的操作系统。本书使用 Raspbian Jessy，可以从[`www.raspberrypi.org/downloads/`](https://www.raspberrypi.org/downloads/)下载。在此链接中，您将找到下载所有必要软件的信息，以便与您的 Raspberry Pi 一起使用部署 Raspbian。您需要至少 4GB 的微型 SD 卡。

我用来测试 Raspberry Pi Zero 的套件包括安装所有必要的东西和准备好板子所需的一切：

![设置 Raspberry Pi Zero](img/image_01_001-3.jpg)

## 准备 SD 卡

Raspberry Pi Zero 只能从 SD 卡启动，不能从外部驱动器或 USB 存储设备启动。对于本书，建议使用 4GB 的微型 SD 卡。

## 安装 Raspbian 操作系统

树莓派板上有许多可用的操作系统，其中大多数基于 Linux。然而，通常推荐的是 Raspbian，这是一个基于 Debian 的操作系统，专门为树莓派制作。

为了在您的 Pi 上安装 Raspbian 操作系统，请按照以下步骤：

1.  从官方 Raspberry Pi 网站下载最新的 Raspbian 镜像：[`www.raspberrypi.org/downloads/raspbian/`](https://www.raspberrypi.org/downloads/raspbian/)

1.  接下来，使用适配器将微型 SD 卡插入计算机。（适配器通常随 SD 卡一起提供。）

1.  然后从[`sourceforge.net/projects/win32diskimager/`](https://sourceforge.net/projects/win32diskimager/)下载 Win32DiskImager。

在下载文件夹后，您将看到以下文件，如截图所示：

![安装 Raspbian 操作系统](img/image_01_002-3.jpg)

1.  打开文件映像，选择您的微型 SD 卡路径，然后单击**写**按钮。

1.  几秒钟后，您的 SD 卡上安装了 Raspbian；将其插入 Raspberry Pi 并通过微型 USB 端口将 Raspberry Pi 板连接到电源源。

在下面的截图中，您可以看到安装的进度：

![安装 Raspbian 操作系统](img/image_01_003-3.jpg)

## 使用串行控制台电缆调试您的 Raspberry Pi Zero

在这一部分，我们将看看如何使用 TTL 串行转换器从计算机与 Raspberry Pi Zero 进行通信。我们可以通过连接到计算机的 USB 端口的串行控制电缆进行调试。我们使用串行电缆与板子通信，因为如果我们想要从计算机向板子发送命令，就必须使用这根电缆进行通信。您可以在[`www.adafruit.com/products/954`](https://www.adafruit.com/products/954)找到这根电缆：

![使用串行控制电缆调试您的 Raspberry Pi Zero](img/B05170_01_04.jpg)

重要的是要考虑到这根电缆使用 3.3 伏特，但我们不在乎，因为我们使用的是 Adafruit 的电缆。它经过测试可以在这个电压级别下工作。

您需要按照以下步骤安装和与您的 Raspberry Pi Zero 通信：

1.  您的计算机上需要有一个空闲的 USB 端口。

1.  我们需要安装串行控制电缆的驱动程序，以便系统可以识别硬件。我们建议您从[`www.adafruit.com/images/product-files/954/PL2303_Prolific_DriverInstaller_v1_12_0.zip`](https://www.adafruit.com/images/product-files/954/PL2303_Prolific_DriverInstaller_v1_12_0.zip)下载驱动程序。

1.  我们使用一个名为 PuTTY 的接口（控制台软件），在 Windows 计算机上运行；这样我们就可以与我们的 Raspberry Pi 板进行通信。这个软件可以从[`www.putty.org/`](http://www.putty.org/)下载和安装。

1.  对于连接，我们需要将红色电缆连接到**5**伏特，黑色电缆连接到地面，白色电缆连接到**TXD**引脚，绿色电缆连接到 Raspberry Pi Zero 的 RXD 引脚。

1.  电缆的另一端连接插头到 USB 端口。

这是连接的图像；这是硬件配置：

![使用串行控制电缆调试您的 Raspberry Pi Zero](img/B05170_01_05.jpg)

## 测试和访问串行 COM 接口

驱动程序安装完成后，我们已经安装了端口 COM：

### 提示

这个配置是为了 Windows 安装；如果你有不同的操作系统，你需要执行不同的步骤。

**如何获得设备管理器屏幕**：在您的 Windows PC 上，点击**开始**图标，转到控制面板，选择系统，然后点击**设备管理器**。

在下面的截图中，您可以看到 USB 串行端口的设备管理器：

![测试和访问串行 COM 接口](img/B05170_01_06.jpg)

1.  在 PuTTY 中打开终端，并选择串行通信为`COM3`，**速度**为`115200`，**奇偶校验**为**无**，**流控制**为**无**；点击**打开**：![测试和访问串行 COM 接口](img/B05170_01_07.jpg)

1.  当出现空白屏幕时，按下键盘上的*Enter*：![测试和访问串行 COM 接口](img/B05170_01_08.jpg)

1.  这将启动与您的 Pi 板的连接，并要求您输入用户名和密码；您将看到一个屏幕，类似于以下截图，带有认证登录：![测试和访问串行 COM 接口](img/B05170_01_09.jpg)

1.  Raspberry Pi Zero 的默认用户名是`pi`，密码是`raspberry`：![测试和访问串行 COM 接口](img/B05170_01_10.jpg)

# 连接到家庭网络并远程访问

我们的 Raspberry Pi 将在一个真实的网络中工作，因此它需要设置为与将一起使用的所有设备一起工作。因此，我们需要配置我们的家庭网络。我们将向您展示如何使用以太网适配器和可以用于 Raspberry Pi Zero 的 Wi-Fi 插头。

## 使用以太网适配器连接

如果您想将我们的树莓派 Zero 连接到本地网络，您需要使用来自 Adafruit 的 USB OTG 主机电缆-MicroB OTG 公对母。您可以在这里找到它：[`www.adafruit.com/products/1099`](https://www.adafruit.com/products/1099)。我们正在使用的板没有以太网连接器，因此有必要使用它与外部设备进行通信。

在下面的图像中，我们可以看到以太网适配器连接到树莓派 Zero：

![使用以太网适配器连接](img/B05170_01_11.jpg)

这是您可以使用的连接器，用于连接您的以太网适配器并与网络建立链接：

![使用以太网适配器连接](img/B05170_01_12.jpg)

现在我们需要按照以下步骤来配置以太网连接适配器：

1.  将适配器连接到转换器；我使用了**TRENDnet NETAdapter**，但您也可以使用来自 Adafruit 的以太网集线器和 Micro USB OTG 连接器的 USB 集线器。您可以在这里找到它：[`www.adafruit.com/products/2992m`](https://www.adafruit.com/products/2992m)。这是一个集线器，可以连接到以太网电缆或 USB 设备。

1.  验证路由器配置，两个 LED 都开始闪烁后，您可以在配置中看到 IP 地址。 DHCP 服务器将 IP 地址分配给树莓派。

这是您在主机名**raspberrypi**上看到的路由器配置：

![使用以太网适配器连接](img/B05170_01_13.jpg)

## 通过 SSH 访问树莓派 Zero

因为我们知道了树莓派的 IP 地址，我们将使用 PuTTY 终端访问它，如下面的屏幕截图所示。您需要输入 IP 地址，默认端口是`22`；点击**打开**按钮：

![通过 SSH 访问树莓派 Zero](img/B05170_01_14.jpg)

之后，我们将看到如下的登录屏幕：

![通过 SSH 访问树莓派 Zero](img/B05170_01_15.jpg)

使用以下命令：

```js
**sudo ifconfig -a**

```

现在我们可以看到以太网控制器适配器的配置信息。**Eth0**是以太网适配器：

![通过 SSH 访问树莓派 Zero](img/B05170_01_16.jpg)

## 连接到 Wi-Fi 网络

在本节中，我们将向您展示如何配置您的 Wi-Fi 网络连接，以便您的树莓派 Zero 可以与您的 Wi-Fi 网络进行交互。首先，我们需要使用 USB OTG 电缆将微型 Wi-Fi（802.11b/g/n）Wi-Fi dongle 连接到树莓派：

![连接到 Wi-Fi 网络](img/B05170_01_17.jpg)

# 如何安装无线工具

使用以下命令配置无线网络：

```js
**sudo apt-get install wireless-tools**

```

在下面的屏幕截图中，我们可以看到`ifconfig`命令的结果：

![如何安装无线工具](img/B05170_01_18.jpg)

执行命令后，我们将看到安装`wireless-tools`的结果：

![如何安装无线工具](img/B05170_01_19.jpg)

## 配置 IP 地址和无线网络

为了进行网络配置，我们需要为我们的设备分配一个 IP 地址，以便参与网络。

输入以下命令：

```js
**sudo nano etc/network/interfaces**

```

![配置 IP 地址和无线网络](img/B05170_01_20.jpg)

在名为`interface`的配置文件中，我们解释了需要向文件添加什么内容，以便我们可以将树莓派 Zero 连接到 Wi-Fi 网络进行**Wlan0**连接。

我们启动文件配置；这意味着文件的开始：

```js
auto lo 

```

我们为本地主机配置以太网设备`loopback`并启动 DHCP 服务器：

```js
iface lo inet loopback 
iface eth0 inet dhcp 

```

允许配置`wlan0`以进行 Wi-Fi 连接：

```js
allow-hotplug wlan0 
auto wlan0
```

我们启动 Wi-Fi 连接的 DHCP 服务器，并输入您的`ssid`和密码的名称。我们需要输入您的 Wi-Fi 网络的`ssid`和`password`参数：

```js
iface wlan0 inet dhcp 
        wpa-ssid "ssid" 
        wpa-psk "password" 

```

# 测试通信

我们需要测试设备是否响应其他主机。现在，如果一切配置正确，我们可以在 Wi-Fi 连接中看到以下 IP 地址：

![测试通信](img/B05170_01_21.jpg)

我们可以在路由器配置中看到分配给无线网络的当前 IP 地址：

![测试通信](img/B05170_01_22.jpg)

## 从计算机 ping

将计算机连接到与 Raspberry Pi 相同的网络：

![从计算机 ping](img/B05170_01_23.jpg)

您需要 ping Raspberry Pi 的 IP 地址。在我们对 Raspberry Pi 无线连接的 IP 地址进行 ping 后，我们看到结果：

![从计算机 ping](img/B05170_01_24.jpg)

# 更新软件包存储库

这将通过从官方 Raspberry Pi 存储库下载所有最新软件包来升级您的 Pi 板，因此这是确保您的板连接到互联网的绝佳方式。然后，从您的计算机上键入以下内容：

```js
**sudo apt-get update**

```

以下屏幕截图显示了 Raspberry Pi 收集软件包数据的过程：

![更新软件包存储库](img/B05170_01_25.jpg)

安装完成后，我们有以下结果：

![更新软件包存储库](img/B05170_01_26.jpg)

# 远程桌面

在这一部分，我们需要具有 Raspbian 操作系统的**RDP**软件包。为此，首先需要执行以下命令：

```js
**sudo apt-get install xrdp** 

```

此命令执行并安装 RDP 进程并更新软件包：

![使用 Windows 远程桌面](img/B05170_01_27.jpg)

## 使用 Windows 远程桌面

在本章结束时，您希望能够使用远程桌面从自己的计算机访问板；您需要输入您的 Raspberry Pi 的 IP 地址并单击**连接**按钮：

![使用 Windows 远程桌面](img/B05170_01_28.jpg)

在我们输入 Raspberry Pi Zero 的 IP 地址后，我们将看到以下屏幕；需要输入您的用户名和密码：

![使用 Windows 远程桌面](img/B05170_01_29.jpg)

您需要您的 Raspberry Pi 的登录信息，用户名和密码：

![使用 Windows 远程桌面](img/B05170_01_30.jpg)

这是操作系统的主窗口；您已经正确地通过远程桌面访问了您的 Raspberry Pi：

![使用 Windows 远程桌面](img/B05170_01_31.jpg)

# 配置 Web 服务器

有几个 Web 服务器可用，我们可以在 Raspberry Pi 上安装。我们将安装`lighttpd`网络服务器。此外，我们需要安装 PHP 支持，这将帮助我们在 Raspberry Pi 上运行网站并拥有动态网页。

要安装和配置，请通过 PuTTY 的终端控制台登录到 Raspberry Pi：

1.  更新软件包安装程序：

```js
 **sudo apt-get update**

```

1.  安装`lighttpd`网络服务器：

```js
 **sudo apt-get install lighttpd**

```

安装后，它将自动作为后台服务启动；每次 Raspberry Pi 启动时都会这样做：

1.  为了设置我们的 PHP 5 界面以使用 PHP 5 进行编程，我们需要使用以下命令安装`PHP5`模块支持；这对于拥有我们的服务器并且可以执行 PHP 文件的必要性是必要的，这样我们就可以制作我们的网站：

```js
 **sudo apt-get install php5-cgi**

```

1.  现在我们需要在我们的 Web 服务器上启用`PHP FastCGI`模块：

```js
 **sudo lighty-enable-mod fastcgi-php**

```

1.  最后一步，我们需要使用以下命令重新启动服务器：

```js
 **sudo /etc/init.d/lighttpd**

```

在下面的屏幕截图中，我们展示了配置 Web 服务器和 PHP 5 界面时将出现的页面内容。Web 服务器在位置`/var/www`安装了一个测试占位页面。在浏览器中输入您的 Raspberry Pi 的 IP 地址，例如`http://192.168.1.105/`，然后出现以下屏幕，打开配置服务器的活动页面：

![配置 Web 服务器](img/B05170_01_32.jpg)

# 测试 PHP 安装

在这一点上，我们需要用 PHP 测试我们的网站。这可以通过编写一个简单的 PHP 脚本页面来完成。如果 PHP 安装正确，它将返回有关其环境和配置的信息。

1.  转到下一个文件夹，那里是根文档：

```js
 **cd /var/www/html** 

```

1.  创建一个名为`phpinfo.php`的文件。

我们使用`nano`这个词，这样我们就可以以特权进入系统文件并执行以下命令：

```js
 **sudo nano phpinfo.php**

```

1.  创建文件后，按照以下截图，按下*CTRL-X*，然后保存文件：![测试 PHP 安装](img/B05170_01_33.jpg)

1.  在浏览器中输入您的树莓派的 IP 地址，例如`http://192.168.1.105/phpinfo.php`，您应该会看到以下屏幕：![测试 PHP 安装](img/B05170_01_34.jpg)

# 总结

在本书的第一章中，我们看了如何配置树莓派 Zero 板，以便在后面的章节中使用。我们看了一下树莓派需要什么组件，以及如何安装 Raspbian，这样我们就可以在板上运行软件。

我们还安装了一个 Web 服务器，这将在本书的一些项目中使用。在下一章中，我们将深入探讨如何将设备连接到您的树莓派和 Arduino 板上。我们还将看看使用 GPIO 可以连接到树莓派的各种东西。
