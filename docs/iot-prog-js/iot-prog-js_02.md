# 第二章。将东西连接到树莓派 Zero

您需要学习如何将东西连接到您的树莓派 Zero，并查看架构并区分我们可以用于我们定义目的的引脚。这就是我们有这一部分的原因-帮助您连接传感器并了解如何连接其他设备的基础知识。在本节中，我们将解释如何配置树莓派；现在您无法避免学习如何连接到您的树莓派传感器以读取连接到它的模拟输入。

我们将涵盖以下主题，以使我们的硬件与板通信：

+   连接数字输入：传感器 DS18B20

+   使用 MCP3008 ADC 转换器连接模拟输入

+   连接实时时钟（RTC）

# 连接数字输入-传感器 DS18B20

树莓派有数字引脚，因此在本节中，我们将看看如何将数字传感器连接到板上。我们将使用数字传感器 DS18B20，它具有数字输出，并且可以完美地连接到我们树莓派传感器的数字输入。主要思想是从传感器中获取温度读数并在屏幕上显示它们。

## 硬件要求

我们需要以下硬件来读取温度：

+   温度传感器 DS18B20（防水）

+   一个 4.7 千欧姆的电阻

+   一些跳线线

+   一个面包板

我们将使用防水传感器 DS18B20 和*4.7*千欧姆电阻：

![硬件要求](img/B05170_02_01.jpg)

这是我们在这个项目中使用的防水传感器。

## 硬件连接

以下图表显示了面包板上的电路，带有传感器和电阻：

![硬件连接](img/B05170_02_02.jpg)

在下图中，我们可以看到带有传感器的电路：

![硬件连接](img/B05170_02_03.jpg)

# 配置单线协议

在树莓派中打开一个终端，并输入以下内容：

```js
**sudo nano /boot/config.txt**

```

您应该在页面底部输入以下行以配置协议并定义单线协议将进行通信的引脚：

```js
**dtoverlay=w1-gpio**

```

下一步是重新启动树莓派。几分钟后，打开终端并输入以下行：

```js
**sudo modprobew1-gpio**
**sudo modprobe w1-therm**

```

进入文件夹并选择要配置的设备：

```js
**cd /sys/bus/w1/devices**
**ls**

```

选择要设置的设备。将`xxxx`更改为协议中将设置的设备的序列号：

```js
**cd 28-xxxx**
**cat w1_slave**

```

您将看到以下内容：

![配置单线协议](img/B05170_02_04.jpg)

之后，您将看到一行，上面写着*如果出现温度读数，则为 Yes，如下所示：t=29.562*。

## 软件配置

现在让我们看一下代码，每秒在屏幕上显示摄氏度和华氏度的温度。

在这里，我们导入了程序中使用的库：

```js
import os1 
import glob1 
import time1 

```

在这里，我们定义了协议中配置的设备：

```js
os1.system('modprobew1-gpio') 
os1.system('modprobew1-therm1') 

```

在这里，我们定义了设备配置的文件夹：

```js
directory = '/sys/bus/w1/devices/' 
device_folder1 = glob1.glob(directory + '28*')[0] 
device_file1 = device_folder1 + '/w1_slave' 

```

然后我们定义读取`温度`和配置传感器的函数：

```js
defread_temp(): 
f = open(device_file1, 'r') 
readings = f.readlines() 
f.close() 
return readings 

```

使用以下函数读取温度：

```js
defread_temp(): 
readings = read_temp() 

```

在这个函数中，我们比较了接收到消息`YES`时的时间，并获取了温度的值：

```js
while readings[0].strip()[-3:] != 'YES': 
time1.sleep(0.2) 
readings = read_temp() 
equals = lines[1].find('t=') 

```

然后我们计算温度，`temp`以`C`和`F`返回值：

```js
if equals != -1: 
temp = readings[1][equals pos+2:] 
tempc = float(temp) / 1000.0 
tempf = temp * 9.0 / 5.0 + 32.0 
returntempc, tempf 

```

它每秒重复一次循环：

```js
while True: 
print(temp()) 
time1.sleep(1) 

```

## 在屏幕上显示读数

现在我们需要执行`thermometer.py`。要显示 Python 中制作的脚本的结果，请打开您的 PuTTY 终端，并输入以下命令：

```js
**sudo python thermometer.py**

```

该命令的意思是，当我们运行温度计文件时，如果一切正常运行，我们将看到以下结果：

![在屏幕上显示读数](img/B05170_02_05.jpg)

# 使用 MCP3008 ADC 转换器连接模拟输入

如果我们想要连接模拟传感器到树莓派，我们需要使用**模拟到数字转换器**（**ADC**）。该板没有模拟输入；我们使用**MCP3008**连接模拟传感器。这是一个 10 位 ADC，有八个通道。这意味着您可以连接多达八个传感器，可以从树莓派 Zero 读取。我们不需要特殊的组件来连接它们。它们可以通过 SPI 连接到树莓派的 GPIO。

第一步是启用 SPI 通信：

1.  访问树莓派终端并输入以下命令：

```js
**sudo raspi-config**

```

1.  如下截图所示选择**高级选项**：![使用 MCP3008 ADC 转换器连接模拟输入](img/B05170_02_06.jpg)

1.  通过选择**SPI**选项启用**SPI**通信：![使用 MCP3008 ADC 转换器连接模拟输入](img/B05170_02_07.jpg)

1.  选择**<Yes>**以启用 SPI 接口：![使用 MCP3008 ADC 转换器连接模拟输入](img/B05170_02_08.jpg)

1.  当我们启用 SPI 接口时，最终屏幕看起来像下面的截图。选择**<Ok>**：![使用 MCP3008 ADC 转换器连接模拟输入](img/B05170_02_09.jpg)

# 树莓派 GPIO 引脚

以下截图是树莓派 Zero 的 GPIO 引脚图表。在这种情况下，我们将使用 SPI 配置接口（`SPI_MOSI, SPI_MISO, SPI_CLK, SPI_CE0_N`）：

![树莓派 GPIO 引脚](img/B05170_02_10.jpg)

以下图表显示了 MCP3008 芯片的引脚名称，您将其连接到树莓派：

![树莓派 GPIO 引脚](img/B05170_02_11.jpg)

以下图片显示了温度传感器：

![树莓派 GPIO 引脚](img/B05170_02_12.jpg)

您需要根据以下描述连接以下引脚：

+   **VDD**连接到***3.3***伏特

+   **VREF**连接到树莓派 Zero 的**3.3**伏特

+   将**AGND**引脚连接到**GND**

+   将**CLK**（时钟）引脚连接到树莓派的**GPIO11**

+   **DOUT**连接到**GPIO9**

+   将**DIN**引脚连接到**GPIO10**

+   将**CS**引脚连接到**GPIO8**和引脚

+   将 MCP3008D 的**GND**引脚连接到地面

这个连接在下图中表示：

![树莓派 GPIO 引脚](img/B05170_02_13.jpg)

以下图片显示了传感器连接到 ADC MCP3008 和树莓派的连接：

![树莓派 GPIO 引脚](img/B05170_02_14.jpg)

## 使用 Python 脚本读取数据

在下一节中，您将创建`MCP3008.py`文件；您需要按照以下步骤进行：

1.  在您的树莓派 Zero 上打开终端。

1.  在您的树莓派终端中输入界面。

1.  在使用之前使用`nano`非常重要。

1.  输入`sudo nano MCP3008.py`。

它将出现在屏幕上，我们将描述以下行：

1.  导入库：

```js
        import spidev1 
        import os1 

```

1.  打开 SPI 总线：

```js
        spi1 = spidev1.SpiDev1() 
        spi1.open(0,0) 

```

1.  定义 ADC MCP2008 的通道：

```js
        def ReadChannel1(channel1): 
          adc1 = spi1.xfer2([1,(8+channel1)<<4,0]) 
          data1 = ((adc1[1]&3) << 8) + adc1[2] 
          return data1 

```

1.  转换电压的函数如下：

```js
        def volts(data1,places1): 
          volts1 = (data1 * 3.3) / float(1023) 
          volts1 = round(volts1,places1) 
          return volts1 

```

1.  转换温度的函数如下：

```js
        def Temp(data1,places1): 
          temp1 = (data1 * 0.0032)*100 
          temp1 = round(temp1,places1) 
          return temp1 

```

1.  定义 ADC 的通道：

```js
          channels = 0 

```

1.  定义读取时间：

```js
        delay = 10 

```

1.  读取温度的函数如下：

```js
        while True: 

          temp  = Channels(temp) 
          volts = Volts(temp1,2) 
          temp  = Temp(temp1,2) 

```

1.  打印结果：

```js
        print"**********************************************" 
        print("Temp : {} ({}V) {} degC".format(temp1,volts,temp)) 

```

1.  每 5 秒等待：

```js
        Time1.sleep(delay) 

```

1.  使用以下命令运行 Python 文件：

```js
**sudo python MCP3008.py**

```

1.  在下一个屏幕上，我们可以看到温度、ADC 测量值和根据温度的电压：![使用 Python 脚本读取数据](img/B05170_02_15.jpg)

# 连接 RTC

要控制系统，拥有一个可以读取时间的电路非常重要；它可以帮助控制树莓派的输出或在特定时间检测动作。我们将使用**RTC**模块*DS3231*与树莓派进行接口。

## I2C 设置

第一步是通过执行以下步骤启用**I2C**接口：

1.  选择**高级选项**：![I2C 设置](img/B05170_02_16.jpg)

1.  启用**I2C**选项，如下截图所示：![I2C 设置](img/B05170_02_17.jpg)

1.  在下一个屏幕上选择**<Yes>**：![I2C 设置](img/B05170_02_18.jpg)

1.  选择**<Ok>**：![I2C 设置](img/B05170_02_19.jpg)

1.  然后选择**<Yes>**：![I2C 设置](img/B05170_02_20.jpg)

1.  接下来，选择**<OK>**：![I2C 设置](img/B05170_02_21.jpg)

# DS3231 模块设置

DS3231 模块是一个实时时钟。它可以用于从集成电路获取时间和日期，因此可以与您的系统一起工作，以控制您想要从嵌入式芯片编程的特定事件。它可以与树莓派 Zero 完美配合，以实时获取时间和日期。

您需要确保您有最新的更新。为此，请在终端中输入以下命令：

```js
**sudo apt-get update**
**sudo apt-get -y upgrade**

```

使用以下命令修改系统文件：

```js
**sudo nano /etc/modules**

```

将以下行添加到`modules.txt`文件中：

```js
**snd-bcm2835 
i2c-bcm2835 
i2c-dev 
rtc-ds1307**

```

## 硬件设置

在本节中，我们将查看 RTC 模块的引脚：

```js
DS3231   Pi GPIO 
GNDP     1-06 
VCC      (3.3V) 
SDA      (I2CSDA) 
SCL      (I2CSCL)
```

这是 RTC 模块，我们可以看到芯片的引脚：

![硬件设置](img/B05170_02_22.jpg)

以下图表显示了电路连接：

![硬件设置](img/B05170_02_23.jpg)

以下图片显示了最终的连接：

![硬件设置](img/B05170_02_24.jpg)

# 测试 RTC

打开终端，输入以下内容：

```js
**sudo i2cdetect -y 1**

```

您应该看到类似于以下截图的内容：

![测试 RTC](img/B05170_02_25.jpg)

# I2C 设备设置

下一步是检查时间时钟是否与 RTC 时间同步。在这里我们定义 RTC 本地：

```js
**sudo nano /etc/rc.local**

```

将以下行添加到文件中，因为我们声明了新设备和我们配置的路径：

```js
echo ds1307 0x68 > /sys/class/i2c-adapter/i2c-1/new_device 

```

以下命令将启动 RTC：

```js
**hwclock -s**

```

执行此命令后，重新启动 Pi。您将看到以下屏幕，这意味着 RTC 已配置并准备好工作：

![I2C 设备设置](img/B05170_02_26.jpg)

# 将实时时钟进行最终测试

您可以使用以下命令读取 Pi 时间系统：

```js
**date**

```

![将实时时钟进行最终测试](img/image_02_027.jpg)

一旦 RTC 准备就绪，您可以使用以下命令进行测试；将时间写入 RTC：

```js
**sudo hwclock -w**

```

您可以使用此处给出的命令从 RTC 读取时间：

```js
**sudo hwclock -r**

```

现在进行最终命令。使用此命令，我们可以看到以下截图中显示的时间值：

![将实时时钟进行最终测试](img/image_02_028.jpg)

# 总结

在本章中，您学习了如何使用 MCP3008 ADC 转换器，以及如何使用树莓派 Zero 使用温度传感器。我们探索了 GPIO 端口及其各种接口。我们看了看可以使用 GPIO 连接到树莓派的各种东西。

在下一章中，我们将深入研究更多的硬件采集，连接不同类型的传感器到我们的树莓派 Zero 和 Arduino 板。这将帮助您在项目中进行真实的测量。非常有趣 - 继续努力！
