# 第九章：智能监视

在第八章 *树莓派图像流*中，我们学习了如何将树莓派相机模块连接到树莓派 3，拍摄照片或视频，然后实时上传/流式传输。在本章中，我们将把这种逻辑提升到下一个级别。当检测到入侵时，我们将拍照，然后将该图像发送到亚马逊 Rekognition 平台并与一组图像进行比较。

在本章中，我们将涵盖以下内容：

+   理解 AWS Rekognition

+   使用授权人脸填充 AWS Rekognition 集合

+   在入侵时从树莓派 3 拍照并将其与种子人脸进行比较

# AWS Rekognition

以下引用来自 Amazon Rekognition ([`aws.amazon.com/rekognition/`](https://aws.amazon.com/rekognition/))：

“Amazon Rekognition 是一项使您的应用程序轻松添加图像分析的服务。使用 Rekognition，您可以检测对象、场景、人脸；识别名人；并识别图像中的不当内容。您还可以搜索和比较人脸。Rekognition 的 API 使您能够快速将基于深度学习的复杂视觉搜索和图像分类添加到您的应用程序中。”

在本章中，我们将利用 AWS Rekognition 功能，帮助我们基于人脸识别而不是人脸检测来设置有条件的监视。

假设您已经在家门口使用树莓派设置了摄像头，并对其进行了编程，以持续拍摄入侵者的照片并将其发送给您。在这种设置中，您将收到每个来到您家门口的人的照片，例如您的家人、邻居等。但是，如果只有在入侵者是陌生人时才通知您呢？现在，这就是我所说的智能监视。

在第八章 *树莓派图像流*中，我们建立了一个设置，当检测到入侵时捕获图像，然后实时发送电子邮件并更新应用程序。

在本章中，我们将使用一组受信任的人脸填充 AWS Rekognition。然后，当摄像头捕获图像时，在检测到入侵时，我们将其发送到 AWS Rekognition 进行人脸识别。如果图像与受信任的图像之一匹配，则不会发生任何事情；否则，将发送电子邮件通知。

要了解更多关于 AWS Rekognition 及其工作原理的信息，请查看*宣布亚马逊 Rekognition-基于深度学习的图像分析*([`www.youtube.com/watch?v=b6gN9jCmq3w`](https://www.youtube.com/watch?v=b6gN9jCmq3w))。

# 设置智能监视

现在我们已经了解了我们要做什么，我们将开始设置树莓派。

我们将设置摄像头和运动检测器，就像我们在第八章 *树莓派图像流*中所做的那样。接下来，我们将添加所需的逻辑，以在检测到运动时捕获图像，然后将其发送进行处理。

在执行此操作之前，我们需要使用授权人脸填充 Rekognition 集合。

此脚本可以作为 API 的一部分，作为 API 引擎，并且使用 Web 仪表板，我们可以上传和填充图像。但为了保持简单，我们将从计算机上运行这个独立的脚本。

# 设置 AWS 凭据

在开始开发之前，我们需要在本地机器上安装 AWS CLI 和 AWS 凭据。

首先，我们需要安装 AWS CLI。转到[`aws.amazon.com/cli`](https://aws.amazon.com/cli)并按照页面上的说明进行操作。要从命令提示符中测试安装，请运行：

```js
aws --version
```

您应该看到类似以下的内容：

```js
aws-cli/1.7.38 Python/2.7.9 Darwin/16.1.0
```

设置完成后，我们需要配置 AWS 凭据，这样只要我们使用这台机器，就不需要在代码中输入任何凭据。

运行以下内容：

```js
aws configure
```

您将看到四个问题；用适当的信息填写它们：

![](img/00119.jpeg)

如果在配置 AWS 凭据时遇到问题，请参考[`docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration`](http://docs.aws.amazon.com/cli/latest/userguide/cli-chap-getting-started.html#cli-quick-configuration)。

另一个选择是在代码本身中添加`accessKeyId`和`secretAccessKey`。但我们仍然需要`accessKeyId`和`secretAccessKey`来继续。

配置完成后，我们将开始与 AWS Rekognition 进行接口。

# 播种授权面孔

创建一个名为`chapter9`的文件夹，在这个文件夹内创建一个名为`rekogniton_seed`的文件夹。在这个文件夹内，创建一个名为`seed.js`的文件。

按照以下方式更新`seed.js`：

```js
var config = { 
    collectionName: 'AIOWJS-FACES', 
    region: 'eu-west-1', 
// If the credentials are set using `aws configure`, below two properties are not needed.    
    accessKeyId: 'YOUR-ACCESSKEYID',  
    secretAccessKey: YOUR-SECRETACCESSKEY' 
}; 

var AWS = require('aws-sdk'); 
var fs = require('fs-extra'); 
var path = require('path'); 
var klawSync = require('klaw-sync') 

AWS.config.region = config.region; 

var rekognition = new AWS.Rekognition({ 
    region: config.region, 
 // accessKeyId: config.accessKeyId, // uncomment as applicable 
 // secretAccessKey: config.secretAccessKey // uncomment as applicable 
}); 

function createCollection() { 
    rekognition.createCollection({ 
        'CollectionId': config.collectionName 
    }, (err, data) => { 
        if (err) { 
            console.log(err, err.stack); // an error occurred 
        } else { 
            console.log(data); // successful response 
        } 
    }); 
} 

function indexFaces() { 
    var paths = klawSync('./faces', { 
        nodir: true, 
        ignore: ['*.json'] 
    }); 

    paths.forEach((file) => { 
        var p = path.parse(file.path); 
        var name = p.name.replace(/\W/g, ''); 
        var bitmap = fs.readFileSync(file.path); 

        rekognition.indexFaces({ 
            'CollectionId': config.collectionName, 
            'DetectionAttributes': ['ALL'], 
            'ExternalImageId': name, 
            'Image': { 
                'Bytes': bitmap 
            } 
        }, (err, data) => { 
            if (err) { 
                console.log(err, err.stack); // an error occurred 
            } else { 
                console.log(data.FaceRecords); // successful response 
                fs.writeJson(file.path + '.json', data, (err) => { 
                    if (err) return console.error(err) 
                }); 
            } 
        }); 
    }); 
} 

createCollection(); 
indexFaces(); 
```

有关其他评论，请参阅源代码：[`github.com/PacktPublishing/Practical-Internet-of-Things-with-JavaScript`](https://github.com/PacktPublishing/Practical-Internet-of-Things-with-JavaScript)。

从前面的代码片段中，我们看到我们正在在`eu-west-1`地区创建一个名为`AIOWJS-FACES`的新集合。您可以在代码中使用`accessKeyId`和`secretAccessKey`，也可以使用 AWS CLI 配置中的这两个。如果您正在使用 AWS CLI 配置中的密钥和密钥，您可以在初始化`rekognition`的新实例时将这两行注释掉。

我们调用`createCollection()`来创建一个新的集合，这只需要运行一次。

您可以随时播种数据，但集合创建应该只发生一次。

创建集合后，我们将从名为`faces`的文件夹中索引一些图像，现在我们将创建。在`rekogniton_seed`文件夹的根目录下创建一个名为`faces`的文件夹。在此文件夹中，上传带有面孔的清晰图像。图像的质量和清晰度越好，被识别的机会就越大。

我在`faces`文件夹中放了几张我的照片。在我们开始播种之前，我们需要安装所需的依赖项：

1.  在`rekogniton_seed`文件夹内打开命令提示符/终端并运行：

```js
npm init --yes
```

1.  接下来，运行：

```js
npm install aws-sdk fs-extra klaw-sync --save
```

1.  安装完成后，通过运行创建集合并通过运行种子面孔：

```js
node seed.js
```

1.  对于每个上传的图像，我们应该看到类似以下的输出：

```js
[ { Face:  
     { FaceId: '2d7ac2b3-fa84-5a16-ad8c-7fa670b8ec8c', 
       BoundingBox: [Object], 
       ImageId: '61a299b6-3004-576d-b966-31fb6780f1c7', 
       ExternalImageId: 'photo', 
       Confidence: 99.96211242675781 }, 
    FaceDetail:  
     { BoundingBox: [Object], 
       AgeRange: [Object], 
       Smile: [Object], 
       Eyeglasses: [Object], 
       Sunglasses: [Object], 
       Gender: [Object], 
       Beard: [Object], 
       Mustache: [Object], 
       EyesOpen: [Object], 
       MouthOpen: [Object], 
       Emotions: [Object], 
       Landmarks: [Object], 
       Pose: [Object], 
       Quality: [Object], 
       Confidence: 99.96211242675781 } } ] 
```

这个对象将包含 Rekognition 分析的图像的信息。

一旦播种完成，您可以查看`faces`文件夹中的`*.json`文件。这些 JSON 文件将包含有关图像的更多信息。

# 测试播种

现在播种完成了，让我们验证播种。这一步是完全可选的；如果您愿意，可以跳过这一步。

在`chapter9`文件夹的根目录下创建一个名为`rekogniton_seed_test`的新文件夹。然后在`rekogniton_seed_test`的根目录下创建一个名为`faces`的文件夹，并将要测试的图像放入该文件夹。在我的情况下，图片是我在不同地点的照片。

接下来，创建一个名为`seed_test.js`的文件并更新它，如下所示：

```js
var config = { 
    collectionName: 'AIOWJS-FACES', 
    region: 'eu-west-1', 
    accessKeyId: 'ACCESSKEYID',  
    secretAccessKey: SECRETACCESSKEY' 
}; 

var AWS = require('aws-sdk'); 
var fs = require('fs-extra'); 
var path = require('path'); 
var klawSync = require('klaw-sync') 

AWS.config.region = config.region; 

var rekognition = new AWS.Rekognition({ 
    region: config.region, 
    // accessKeyId: config.accessKeyId, // uncomment as applicable 
    // secretAccessKey: config.secretAccessKey // uncomment as applicable 
}); 

// Once you've created your collection you can run this to test it out. 
function FaceSearchTest(imagePath) { 
    var bitmap = fs.readFileSync(imagePath); 

    rekognition.searchFacesByImage({ 
        "CollectionId": config.collectionName, 
        "FaceMatchThreshold": 80, 
        "Image": { 
            "Bytes": bitmap, 
        }, 
        "MaxFaces": 1 
    }, (err, data) => { 
        if (err) { 
            console.error(err, err.stack); // an error occurred 
        } else { 
            // console.log(data); // successful response 
            console.log(data.FaceMatches.length > 0 ? data.FaceMatches[0].Face : data); 
        } 
    }); 
} 

FaceSearchTest(__dirname + '/faces/arvind_2.jpg'); 
```

在前面的代码中，我们从`faces`文件夹中提取图像并提交给识别，然后打印适当的响应。

完成后，我们将安装所需的依赖项：

1.  在`rekogniton_seed_test`文件夹内打开命令提示符/终端并运行：

```js
npm init --yes
```

1.  然后运行：

```js
npm install aws-sdk fs-extra path --save
```

1.  现在，我们已经准备好运行这个例子了。从`rekogniton_seed_test`文件夹内运行：

```js
node seed_test.js
```

1.  我们应该看到类似以下的东西：

```js
{ FaceId: '2d7ac2b3-fa84-5a16-ad8c-7fa670b8ec8c', 
  BoundingBox:  
   { Width: 0.4594019949436188, 
     Height: 0.4594019949436188, 
     Left: 0.3076919913291931, 
     Top: 0.2820509970188141 }, 
  ImageId: '61a299b6-3004-576d-b966-31fb6780f1c7', 
  ExternalImageId: 'photo', 
  Confidence: 99.96209716796875 } 
```

从前面的响应中有几件事情需要注意：

+   `FaceId`：这是当前面部匹配的 ID

+   `ImageId`：这是当前面部匹配的图像

有了这个，我们甚至可以从我们已经索引/播种的图像中标记用户。

您可以通过放置一个与我们的种子数据不匹配的图像并更新上述代码的最后一行来进行负面测试：

```js
FaceSearchTest(__dirname + '/faces/no_arvind.jpg');

We should see something like the following:

{ SearchedFaceBoundingBox:

{ Width: 0.5322222113609314,

Height: 0.5333333611488342,

Left: 0.2777777910232544,

Top: 0.12444444745779037 },

SearchedFaceConfidence: 99.76634979248047,

FaceMatches: [] }
```

如你所见，没有找到匹配项。

一旦我们捕获了图像，我们将在树莓派中使用上述方法。

# 部署到树莓派

现在我们已经创建了一个 Rekognition 集合，并对其进行了测试（可选步骤），我们现在将开始设置树莓派代码。

我们将使用`chapter8`文件夹中的所有其他代码片段，并且只修改`chapter9`文件夹中的树莓派客户端。

将整个代码从`chapter8`文件夹复制到`chapter9`文件夹中。然后，打开`pi-client`文件夹，无论是在您的桌面上还是在树莓派本身上，并进行以下更新：

```js
var config = require('./config.js'); 
var mqtt = require('mqtt'); 
var GetMac = require('getmac'); 
var Raspistill = require('node-raspistill').Raspistill; 
var crypto = require("crypto"); 
var Gpio = require('onoff').Gpio; 
var exec = require('child_process').exec; 

var AWS = require('aws-sdk'); 

var pir = new Gpio(17, 'in', 'both'); 
var raspistill = new Raspistill({ 
    noFileSave: true, 
    encoding: 'bmp', 
    width: 640, 
    height: 480 
}); 

// Rekognition config 
var config = { 
    collectionName: 'AIOWJS-FACES', 
    region: 'eu-west-1', 
    accessKeyId: 'ACCESSKEYID',  
    secretAccessKey: 'SECRETACCESSKEY' 
}; 

AWS.config.region = config.region; 

var rekognition = new AWS.Rekognition({ 
    region: config.region, 
    accessKeyId: config.accessKeyId, 
    secretAccessKey: config.secretAccessKey 
}); 

var client = mqtt.connect({ 
    port: config.mqtt.port, 
    protocol: 'mqtts', 
    host: config.mqtt.host, 
    clientId: config.mqtt.clientId, 
    reconnectPeriod: 1000, 
    username: config.mqtt.clientId, 
    password: config.mqtt.clientId, 
    keepalive: 300, 
    rejectUnauthorized: false 
}); 

client.on('connect', function() { 
    client.subscribe('rpi'); 
    GetMac.getMac(function(err, mac) { 
        if (err) throw err; 
        macAddress = mac; 
        client.publish('api-engine', mac); 
        // startStreaming(); 
    }); 

}); 

client.on('message', function(topic, message) { 
    message = message.toString(); 
    if (topic === 'rpi') { 
        console.log('API Engine Response >> ', message); 
    } else { 
        console.log('Unknown topic', topic); 
    } 
}); 

var processing = false; 

// keep watching for motion 
pir.watch(function(err, value) { 
    if (err) exit(); 
    if (value == 1 && !processing) { 
        raspistill.takePhoto() 
            .then((photo) => { 
                console.log('took photo'); 
                checkForMatch(photo, function(err, authorizedFace) { 
                    if (err) { 
                        console.error(err); 
                    } else { 
                        if (authorizedFace) { 
                            console.log('User Authorized'); 
                        } else { 
                            // unauthorized user,  
                            // send an email! 
                            require('./mailer').sendEmail(photo, function(err, info) { 
                                if (err) { 
                                    console.error(err); 
                                } else { 
                                    console.log('Email Send Success', info); 
                                } 
                            }); 
                        } 
                    } 
                }); 
            }) 
            .catch((error) => { 
                console.error('something bad happened', error); 
            }); 
    } 
}); 

function checkForMatch(image, cb) { 
    rekognition.searchFacesByImage({ 
        'CollectionId': config.collectionName, 
        'FaceMatchThreshold': 80, 
        'Image': { 
            'Bytes': image, 
        }, 
        'MaxFaces': 1 
    }, (err, data) => { 
        if (err) { 
            console.error(err, err.stack); // an error occurred 
            cb(err, null); 
        } else { 
            // console.log(data); // successful response 
            console.log(data.FaceMatches.length > 0 ? data.FaceMatches[0].Face : data); 
            cb(null, data.FaceMatches.length >= 1); 
        } 
    }); 
} 

function exit() { 
    pir.unexport(); 
    process.exit(); 
} 
```

在上述代码中，我们已经配置了向 AWS Rekognition 发出请求所需的配置，然后运行`checkForMatch()`，它将获取原始照片并检查匹配项。如果找到任何匹配项，我们将不会收到电子邮件；如果没有找到匹配项，我们将收到电子邮件。

接下来，我们将安装所需的依赖项。

运行以下命令：

```js
npm install getmac mqtt node-raspistill aws-sdk --save
```

安装完成后，启动代理、`api-engine`和 Web 仪表板。然后运行以下命令：

```js
node index.js
```

触发运动以捕获图像。如果捕获的图像与我们索引的人脸之一匹配，我们将不会收到电子邮件；如果匹配，我们将收到电子邮件。

简单吧？这是一个非常强大的设置，我们已经建立了一个可以在家庭或办公室提供监控的系统，可以轻松识别简单的误报。

这个例子可以进一步扩展，以便使用基于云的呼叫服务（如 Twilio）发送推送通知或呼叫邻居。

# 总结

在本章中，我们看到了如何使用树莓派和 AWS Rekognition 平台建立智能监控系统。

我们首先了解了 AWS Rekognition 平台，然后使用我们的图像对集合进行了索引/种子化。接下来，我们更新了树莓派代码，以便在检测到运动时拍摄照片，然后将该图像发送到 AWS Rekognition 以确定当前照片中的人脸是否与任何索引图像匹配。如果匹配，我们忽略该图像；如果不匹配，我们将发送一封带有该图像的电子邮件。

通过这样，我们完成了*使用 JavaScript 进行实际物联网*。我希望您已经学会了几种利用 JavaScript 和树莓派构建简单而强大的物联网解决方案的方法。
