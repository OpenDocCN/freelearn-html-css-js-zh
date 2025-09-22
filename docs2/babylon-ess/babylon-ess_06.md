# 第六章：在 Babylon.js 中管理音频

在上一章中，你通过添加碰撞检查和物理模拟来为场景添加动态效果。另一个重要功能是处理场景中的声音，并最终使场景更加生动。本章不仅解释了使用 Babylon.js 进行声音管理以创建配乐声音，还涵盖了空间化声音（3D）。在本章中，我们将涵盖以下主题：

+   播放 2D 声音

+   播放 3D 声音

# 播放 2D 声音

Babylon.js 框架提供了一个基于 WebAudio 的音频引擎。它允许你使用 Babylon.js 团队为你开发的工具轻松添加 2D 和 3D 声音。

## 创建 2D 声音

Babylon.js 框架提供了一个 `BABYLON.Sound` 类。此类允许你为场景创建和管理 2D 和 3D 声音。要添加声音，你需要做的只是创建一个新的 `BABYLON.Sound` 对象，如下所示：

```js
var sound = new BABYLON.Sound("sound_name", "sound_file", scene); 

```

现在，你可以访问 `.play`、`.pause` 和 `.stop` 等方法。

事实上，声音是异步加载的，因此你无法在创建新的声音对象后立即调用 `sound.play()`。这就是为什么 `BABYLON.Sound` 构造函数在场景之后提供了一个 `readyToPlayCallback` 参数，以便处理加载过程。要加载后播放声音，只需设置 `readyToPlayCallback` 参数，如下所示：

```js
var sound = new BABYLON.Sound("sound_name", "sound_file", scene, () => { 
   sound.play(); 
}); 

```

幸运的是，开发者考虑到了这种行为，并提供了一个名为 `options` 的最后一个参数。此参数允许你自动设置默认行为，而不是在准备播放回调中管理它们。`options` 参数是可选的，其外观类似于以下内容：

```js
var sound = new BABYLON.Sound("sound_name", "sound_file", scene, () => { 
}, { 
   loop: true, // [Boolean] plays sounds in loop mode 
   autoPlay: true, // [Boolean] plays the sound after loading 
   volume: 1.0, // Between [Number [0, 1]] volume of sound 
   playbackRate: 1.0 // [Number 1.0] playing speed 
   spatialSound: false, [Boolean] is a 3D sound 
   maxDistance: 100, [Boolean] maximum distance of 3D sound 
}); 

```

`autoPlay` 参数会在加载后自动播放声音，你不需要在准备播放回调中管理声音。

Babylon.js 框架仍然保持了简单性，因为你只需在 `BABYLON.Sound` 类上调用一个 `new` 语句，就为场景添加了一个音频轨道。

当然，你可以在单个场景中播放多个声音，如下面的代码所示：

```js
var options = { 
   loop: true, 
   autoPlay: true 
};  
var sound1 = new BABYLON.Sound("sound1", "sound1.mp3", scene, null, options); 
var sound2 = new BABYLON.Sound("sound2", "sound2.mp3", scene, null, options); 

```

## 管理二维声音

有几个属性可以用来在 2D 中操作声音，例如音量、声音是否正在播放等。

要设置声音的音量，只需调用声音的 `.setVolume` 函数。新的音量设置在 [0, 1] 区间内，如下面的代码所示：

```js
sound.setVolume(0.5); 

```

你还可以获取音量，如下面的代码所示：

```js
var currentVolume = sound.getVolume(); 

```

你可以随时检查声音的当前状态，如下所示：

```js
    sound.isPlaying; // true if the sound is playing
    sound.isPaused // true if the sound was paused

```

你也可以随时设置声音的状态，如下所示：

```js
    sound.play(); // Plays the sound
    sound.pause(); // Pauses the sound
    sound.stop(); // Stops the sound

```

# 播放 3D 声音

在前面的主题中，你已经在场景中添加并播放了 2D 声音。这些 2D 声音可以很容易地用作游戏的配乐。为了给场景增加动态效果，就像物理和碰撞一样，你可以配置声音在场景中空间化。空间化声音，也称为 3D 声音，提供了玩家与声音之间的距离和方向感。换句话说，玩家离声音越远，声音的衰减就越大。此外，如果声音位置相对在玩家的右边，右扬声器产生的声音比左扬声器多，反之亦然。

例如，如果声音从你的右边发出，只有右边的扬声器应该播放它，而且你离声音越远，声音的音量应该越低。

## 创建 3D 声音

你可以想象，对于 2D 声音，你可以使用相同的`BABYLON.Sound`构造函数创建空间化声音。只需更改`options`参数，因为必须将`spatialSound`设置为`true`，如下所示：

```js
    var sound = new BABYLON.Sound("sound", "sound_file", scene, () => {
       sound.play();
    }, { spatialSound: true });

```

一旦创建了空间化声音，你可以使用`BABYLON.Vector3`在场景的世界中设置其 3D 位置，如下所示：

```js
    sound.setPosition(new BABYLON.Vector3(0, 0, 50)); // For example

```

## 管理三维声音

与 2D 声音相比，你可以使用 3D 声音自定义更多属性。空间化声音提供了配置衰减和声像模型的属性。

例如，默认的距离模型（衰减）设置为线性。存在其他两种模型，如`exponential`和`inverse`，如下所示：

```js
    sound.distanceModel = "exponential";

```

关于雾效（第四章;
```

关于`maxDistance`属性，如果你使用指数模型，可以设置`rolloffFactor`属性，如下面的代码所示：

```js
sound.updateOptions({
rolloffFactor: 2
});
```

当然，你可以同时更新多个值。只需将`.updateOptions`参数配置为相应的值。以下是一个示例：

```js
sound.updateOptions({
maxDistance: 10,
rolloffFactor: 2
});
```

对于空间化声音，最后一个有用的功能是直接将声音附加到网格上。然后，不需要手动更新声音的位置来将其设置为网格的位置，如下所示：

```js
sound.attachToMesh(myMesh);
```

现在，当调用`scene.render()`时，声音和网格共享相同的位置，并由场景一起更新。

## 创建方向性空间化声音

你之前创建的空间化声音是全方向的。这意味着如果你在扬声器后面，你会听到和站在扬声器前面一样响亮的声音。这在现实生活中是不会发生的。Babylon.js 音频引擎提供了一种创建易于配置的方向性声音的方法。

### 注意

注意，方向性空间化声音仅在声音附加到网格上时才起作用。

让我们从以下声音参考开始：

```js
var sound = new BABYLON.Sound("sound", "sound_file", scene, () => {
}, { loop: true, autoplay: true });
```

你可以通过调用仅三个函数来配置它为方向性。

首先，声音的方向由一个锥体表示。只需设置方向锥，如下所示：

```js
sound.setDirectionalCone(90, 180, 0.1); // Values are in degree
```

有三个参数，如下所示：

+   内锥体的大小（以度为单位）应该比外锥体小

+   外锥体的大小（以度为单位）应该比内锥体大

+   当玩家位于外锥体外时，空间化声音的音量

为了获得完美的方向性声音，内锥体和外锥体的大小应该相等。

一旦设置了方向锥，只需根据网格旋转设置声音的方向。该参数是局部于网格的。然后，如果你旋转扬声器，例如，声音将始终跟随扬声器的旋转，这取决于参数。考虑以下示例：

```js
sound.setLocalDirectionToMesh(new BABYLON.Vector3(0, 0, 1)); // Always speak ahead (Z is the forward axis)
```

最后，不要忘记按照以下方式将声音附加到网格上：

```js
sound.attachToMesh(myMesh); // myMesh should be the speaker in the example
```

# 摘要

这章快速演示了当为开发者提供强大工具时，在 3D 引擎中使用声音（2D 和 3D）可以变得非常简单。示例文件创建了一个作为配乐播放的 2D 声音和一个位于盒子位置处的 3D 声音。不要犹豫，尝试调整距离模型，并使用你的耳机检查效果。

在下一章中，我们将尝试使用 Babylon.js 的`ActionManager`类自动化一些事情。这个类在触发器被激活时对对象执行动作很有用。例如，如果玩家左键单击盒子，它将播放名为`my_sound.wav`的声音。这也是介绍**动作构建器**的时候，它是 Babylon.js 3ds Max 导出器的一部分。动作构建器允许艺术家（和开发者）在不编写任何代码的情况下为他们的对象创建动作。
