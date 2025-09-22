# 动画库

伺服机构是我们机器人项目创建运动的优秀工具，但为了创建真正移动的行走机器人，我们需要更多的控制。每个伺服机构都不同；例如，每个伺服机构以略不同的最大速度移动。如果你想让机器人行走，你需要时间控制，以及知道伺服机构何时完成其运动的能力。这时就出现了动画库；这个 Johnny-Five 内部强大的工具允许你微调伺服机构运动，以获得更深入的控制。

本章将涵盖以下主题：

+   动画运动

+   动画库的术语

+   动画对象的构建

+   逐步进行伺服机构动画

+   学习更多关于排队和播放动画片段

+   动画对象事件

# 技术要求

你需要从上一章中使用的两个伺服机构设置，这就是本章的硬件要求。

本章的代码可以在[`github.com/PacktPublishing/Hands-On-Robotics-with-JavaScript/tree/master/Chapter08`](https://github.com/PacktPublishing/Hands-On-Robotics-with-JavaScript/tree/master/Chapter08)找到。

# 动画运动

动画库使得使用伺服机构变得可能，否则这些事情可能从困难到完全不可能。然而，在我们探索动画库的“如何”之前，我们应该更彻底地解释“为什么”。

# 为什么我们需要动画库

当你迈步时，想想你的腿部运动。你通常不需要这样做，但当你这样做时，你的关节中会发生很多事情！你的臀部伸展你的腿，你的膝盖伸展你的腿而不通常锁定它。而且你的后腿也在做事情；你的臀部允许腿向后移动，你的脚踝在弯曲。这是一个巨大的简化，但它仍然非常复杂！

现在想象你的每个关节都是一个伺服机构，你需要编程来迈一步。你无法控制每个动作的时间，因为每个伺服机构会以你能达到的最快速度到达你告诉它的位置。你也不能知道一个关节何时完成运动，所以你必须硬编码时间并希望它能保持。

这种确切的问题正是动画库被制作出来要解决的。通过给你更多的控制，你就有更多的精确度。但我们的精确度指的是什么？

# 精确控制伺服机构运动

在移动伺服机构的背景下，真正的精确意味着能够控制所使用的伺服机构的时间、位置和速度。这种精确度在构建需要多个动作同步发生以避免物理碰撞的细致运动时至关重要。一个很好的例子是六足机器人：每个关节需要与其他腿部的关节同步移动，每条腿在迈步时需要精确地移动，以避免相互碰撞或使六足机器人失去平衡。

如果你使用硬编码来精确移动伺服电机，这将是一项艰巨的任务；想象一下，为了创建一个你希望持续一秒钟的动画，你需要设置 60 次`servo.to()`调用。或者，使用`servo.to()`硬编码每个动作，使用你放置在机器人腿部确切伺服电机的时间来计时，并且一切正常...直到伺服电机损坏（这是不可避免的），你必须更换它并重复整个校准过程。

Johnny-Five 中的动画库通过允许你将你的动作定义为更大设计的一部分，即动画本身，从而简化了此过程。它完成所有的数学运算并计算出所有的时间，以确保伺服电机在需要时处于正确的位置。

# 隐式使用动画库

有时候，你甚至不需要创建一个动画对象，就可以为你的伺服电机创建动画。在我们真正深入动画对象之前，让我们编写一些使用动画库隐式调用的代码。

# 使用`servo.to()`隐式创建动画

在本章的`project`文件夹中，创建一个名为`implicit-animations.js`的文件。我们将设置此文件以使用 REPL 来演示不显式创建动画对象创建的动画。从正常的样板开始：引入`johnny-five`和`raspi-io`，像往常一样设置你的板和`board.ready()`处理程序：

```js
const Raspi = require('raspi-io')
const five = require('johnny-five')

const board = new five.Board({
  io: new Raspi()
})

board.on('ready', () => {
})
```

然后，在`board.on('ready')`处理程序内部，构建一个`Servo`对象：

```js
  let servo = new five.Servo({
    controller: "PCA9685",
    pin: 0
  })
```

接下来，仍然在`board.on('ready')`处理程序内部，我们将创建相同的运动三次——每次使用不同的规格。其中两个将创建后台动画，一个则是默认运动，这样你可以看到差异。

如果你正在查看微型伺服电机叶片，可能会很难注意到伺服电机运动中的差异。为了使差异更容易看到，我在本章的伺服电机叶片上贴了一根冰棍棒。我还把它贴在桌子上，这样它就不会倒下，如下面的图所示：

![图片](img/fc252dec-e458-49c8-a00c-3ef579fee832.png)

第一个运动函数，仍然在`board.on('ready')`处理程序内部，将被命名为`normalFullSwing()`：

```js
function normalFullSwing() {
 servo.to(0)
 servo.to(180)
}
```

此函数将尽可能快地将伺服电机移动到`0`度，然后尽可能快地将伺服电机移动到`180`度。

让我们在下一个函数中为`servo.to()`添加一个参数，这将改变伺服电机到达所需位置所需的时间。我们将将其设置为接受一个时间参数，我们将在 REPL 中进行实验时传递它：

```js
function timedFullSwing(time) {
  servo.to(0)
  servo.to(180, time)
}
```

此函数将接受传递给它的毫秒数，从`0`度到`180`度。它仍然会尽可能快地开始移动到`0`度。

最后，我们将编写一个函数，该函数接受时间和步数参数，将伺服电机在给定的时间内从`0`度移动到`180`度，使用给定的步数。我们将此函数称为`timedFullSwingWithSteps()`：

```js
function timedFullSwingWithSteps(time, steps) {
 servo.to(0)
 servo.to(180, time, steps)
}
```

这个函数，和其他函数一样，仍然会尽可能快地先到达 `0`。

最后，我们将使用 `board.repl.inject()` 给自己提供访问这些函数的权限：

```js
board.repl.inject({
  normalFullSwing,
  timedFullSwing,
  timedFullSwingWithSteps
})
```

然后我们就准备开始了（或者说，就像它那样摇摆）！

# 玩转隐式动画

将文件夹加载到你的 Pi 上，使用你的 Pi SSH 会话进入该文件夹，并运行以下命令：

```js
npm init -y
npm i --save johnny-five raspi-io
```

现在我们已经设置好了，我们使用以下命令运行代码：

```js
sudo node implicit-animations.js
```

一旦我们看到 `Board Initialized`，我们就可以运行我们的函数并看到差异。这里有一些可以尝试的：

```js
normalFullSwing() // hm...
timedFullSwing(1000) // wait a tick...
timedFullSwingWithSteps(1000, 10) // why didn't it do it?
```

到现在为止，你可能已经注意到有些不对劲。最多，你可能看到一两个抽搐，但这显然不是按预期工作的！

这是因为一个非常重要的时间问题，那就是如果你不等待伺服器运动完成，它们将相互覆盖，导致不稳定的结果。

这也是动画库之所以重要的部分原因！它具有让你排队动画的能力，这意味着伺服器会在进行下一个动作之前让每个动作完成，从而避免你需要自己编写等待代码（特别是考虑到 JavaScript 的异步性质，这尤其令人讨厌）。

现在，我们将使用更多的隐式动画和一些 `setTimeout()` 调用来使这些函数正常工作。

# 玩转隐式动画，再来一次

为了修复你的 `normalFullSwing()` 函数，我们将设置 `servo.to(0)` 调用为 250 毫秒，然后在 255 毫秒后调用 `servo.to(180)`（只是为了确保它首先完全到达 `0`）：

```js
function normalFullSwing() {
 servo.to(0, 1000)
 setTimeout(() => { servo.to(180) }, 1010)
}
```

我们将对 `timedFullSwing()` 和 `timedFullSwingWithSteps()` 函数做同样的处理：

```js
function timedFullSwing(time) {
  servo.to(0, 1000)
  setTimeout(() => { servo.to(180, time) }, 1010)
}

function timedFullSwingWithSteps(time, steps) {
  servo.to(0, 1000)
  setTimeout(() => { servo.to(180, time, steps) }, 1010)
}
```

一旦你做了这些更改，重新加载你的 `project` 文件夹并运行它：

```js
sudo node implicit-animations.js
```

如果你直接使用书中的代码而不是跟随操作，`implicit-animations-fixed.js` 包含了超时设置，所以你应该运行那个文件而不是再次运行 `implicit-animations.js`。

现在代码按预期运行，让我们再对这些隐式动画玩玩：

```js
// Just a normal swing
normalFullSwing()
// Playing with the time parameter
timedFullSwing(1000)
timedFullSwing(750)
timedFullSwing(5000)
// Playing with the steps parameter
timedFullSwingWithSteps(3000, 2)
timedFullSwingWithSteps(1000, 10)
timedFullSwingWithSteps(1000, 100)
```

在心中记下改变时机和步骤对伺服器运动的影响，这对于本章的其余部分会很有帮助。

现在我们已经理解了一些动画库的底层效果，以及为什么在处理复杂的伺服器运动时它如此关键，让我们详细地研究一下动画库。我们将从术语开始，除非你拥有动画（如动画图像）的强大背景，你还需要学习一些词汇！

# 动画库的术语

动画库被命名为这种方式是非常有意的；动画对象的词汇非常接近动画图像的词汇。让我们看看我们将在这章中大量使用的几个术语。

+   **帧**：在这个上下文中，动画的帧是伺服系统在特定时间点的状态。正如你可以想象的那样，为复杂的伺服系统组（如肢体）编程每一帧伺服运动将是一场噩梦。幸运的是，技术站在我们这边，我们不需要编写每一帧。

+   **关键帧**：关键帧是动画中的一个点，它作为锚点，除非你手动绘制（或编程）动画的每一帧；你建立一组关键帧，以确定动画的主要运动点。例如，在我们之前所做的完整扫描中，一组好的关键帧可能如下所示：

    +   从任何角度开始

    +   处于`0`度

    +   在`180`度时结束

简单的动画拥有较少的关键帧，但你始终需要至少两个来创建一个动画。请注意，关键帧本身并没有任何时间概念；它们必须与提示点结合来创建动画。

+   **提示点**：提示点是在序列上下文中介于 0 和 1 之间的一个点，一组与等大小的关键帧集和整体持续时间配对的提示点创建了一个完整的动画。例如，当我们使用包含 0 秒、1 秒和 2 秒的提示点集与上述关键帧集配对时，你得到一个听起来像动画的东西：

    +   在 0%时从任何角度开始

    +   在 50%时处于`0`度

    +   在`180`度时结束于 100%

+   **持续时间**：持续时间是动画序列将运行的时间长度，当与关键帧和提示点配对时，完成动画。以上述 2000 毫秒的持续时间为例，你得到：

    +   在 0 毫秒时从任何角度开始

    +   在 1000 毫秒时处于`0`度

    +   在 2000 毫秒时结束于`180`度

+   **补间**：补间是软件在关键帧之间创建必要帧的想法。你建立关键帧，补间则确定这些帧之间的内容。每帧之间的时间（由我们的`timedFullSwing()`函数展示）和关键帧之间的步数（帧数）（由我们的`timedFullSwingWithSteps()`函数展示）使我们能够微调补间过程。

+   **缓动**：缓动是补间过程的一部分。没有缓动，所有的补间都是线性完成的，每个补间帧的运动量相同。如果你在构建试图行走的东西，这看起来一点也不平滑。存在几种补间形式；最常见的一种是缓入或缓出缓动，分别以缓慢开始并逐渐加速到快速结束，或者以快速开始并逐渐减速到缓慢结束。

现在我们已经讨论了动画的术语，我们可以开始用 Johnny-Five 编写我们的第一个（显式）动画对象了！

# 动画对象的构建

要构建一个动画对象，我们需要创建对象本身，创建一组关键帧和一组提示点，然后将这些关键帧和提示点作为动画排队在我们的伺服器上运行。

# 创建动画对象

在你的`project`文件夹中创建一个名为`my-first-animation.js`的新文件，并创建正常的样板：在 Johnny-Five 和 Raspi-IO 中`require`，创建你的`Board`对象，并创建`board.on('ready')`函数：

```js
const Raspi = require('raspi-io')
const five = require('johnny-five')

const board = new five.Board({
  io: new Raspi()
})

board.on('ready', () => {
})
```

然后，在`board.on('ready')`处理程序内部，在我们的 PWM 帽的`0`引脚和`1`引脚上构建我们的两个`Servo`对象：

```js
let servoOne = new five.Servo({
  controller: "PCA9685",
  pin: 0
})

let servoTwo = new five.Servo({
  controller: "PCA9685",
  pin: 1
})
```

并创建一个包含我们的伺服器的`Servos`对象：

```js
let servos = new five.Servos([servoOne, servoTwo])
```

现在我们有一组伺服器，我们可以创建一个动画对象：

```js
let myFirstAnimation = new five.Animation(servos)
```

现在我们有了动画对象，是时候规划我们的动画序列，设置关键帧和提示点，并将它们排队进行动画。

# 规划动画序列

让我们规划一个足够简单的动画来进行第一次尝试：让我们允许伺服器从任何地方开始，然后`servoOne`将移动到`0`度，而`servoTwo`将移动到`180`。然后，`servoOne`将扫过到`180`度，而`servoTwo`开始向`90`度移动，然后两个伺服器都将结束在`90`度。让我们让这些位置每两秒发生一次。因此，我们的关键帧看起来可能像这样：

1.  以任何度数开始`servoOne`，以任何度数开始`servoTwo`

1.  将`servoOne`移动到`0`度，将`servoTwo`移动到`180`度

1.  `servoOne`保持在它最后已知的位置，`servoTwo`正在向`90`度移动

1.  将`servoOne`移动到`180`度，`servoTwo`正在向`90`度移动

1.  将`servoOne`移动到`90`度，将`servoTwo`移动到`90`度完成

我们的提示点（以`0`-`1`表示）：`0`，`.25`，`.5`，`.75`，`1`。

现在我们已经规划好了我们的序列，我们可以开始编程我们的关键帧。

# 创建关键帧

我们需要为我们的伺服器组中的每个伺服器创建一个关键帧数组，对于每个提示点：两个包含五个关键帧的数组。

这听起来很简单，但我们如何让动画让伺服器从它们所处的任何位置开始？以及我们如何让`servoTwo`在移动到`90`度的过程中跨越两个提示点进行缓动？答案在于 Johnny-Five 如何解析 null 和 false 作为关键帧中的伺服器位置。

# 在关键帧中使用 null 和 false 作为位置

Johnny-Five 使用 null 和 false 允许我们创建复杂的段，在这些段中，我们可以在多个提示点之间进行缓动，或者使用伺服器的最后已知位置作为关键帧位置。

空对象的效果取决于它被使用的位置，如果它在第一个关键帧中使用，它将使用伺服器的位置作为动画开始时的关键帧位置。这正是我们开始动画序列所需要的，因为我们希望两个伺服器都从它们恰好所在的位置开始。如果空对象在不是第一个的关键帧中使用，那么在那个提示点处，关键帧将被基本忽略；如果你在一个关键帧中有 30 度，下一个关键帧中是空对象，第三个关键帧中是 120 度，伺服器将在两个提示点之间移动`90`度。我们将使用这一点来允许`servoTwo`在两个提示点之间从`180`度移动到`90`度。

当你使用`false`作为关键帧位置时，它将使用在最后一个关键帧中设置的位置。我们将使用这一点在`servoOne`的关键帧调用要求伺服器保持在最后已知位置时，而不是硬编码第二个 180 度的位置。

现在我们知道了空和`false`如何影响我们的关键帧定位，让我们为我们的计划动画序列编程我们的关键帧。

# 编程我们的关键帧

根据我们提供的信息，每个关键帧所需的值如下：

+   `servoOne null`，`servoTwo null`（从伺服器恰好所在的位置开始）

+   `servoOne 0`，`servoTwo 180`

+   `servoOne 180`，`servoTwo null`（`servoTwo`开始向`90`度移动）

+   `servoOne false`，`servoTwo null`（`servoOne`保持不动，`servoTwo`仍在向`90`度移动）

+   `servoOne 90`，`servoTwo 90`

每个位置都需要是一个对象，具有每个关键帧的`degrees`属性。让我们将其转换为 JavaScript，就在我们的动画对象构建下方：

```js
let keyframes = [
 [null, {degrees: 0}, {degrees: 180}, false, {degrees:90}], // servoOne
 [null, {degrees: 180}, null, null, {degrees: 90}] // servoTwo
]
```

现在我们已经编程了关键帧，让我们开始设置提示点。

# 设置提示点和持续时间

提示点，无论你有多少伺服器，都将始终是一个一维数组的时间，以匹配你传递到关键帧数组中的每个关键帧。

注意，尽管这个动画中的提示点均匀分布，但这完全是可选的，你的提示点可以彼此之间距离很远，而不会破坏任何东西。

在我们的关键帧对象下方，让我们设置我们的提示点数组：

```js
let cuePoints = [0, .25, .5, .75, 1]
```

我们希望整个动画持续 8 秒，所以添加：

```js
let duration = 8000
```

我们已经有了所有需要的数据，让我们制作一个动画！

# 将所有内容组合起来制作动画

为了运行我们的动画序列，我们必须使用`Animation.enqueue()`函数将其放入队列中。我们需要一起传递持续时间、关键帧和提示点。在你的`my-first-animation.js`中，在持续时间之后添加：

```js
myFirstAnimation.enqueue({
 duration: duration,
 keyFrames: keyframes,
 cuePoints: cuePoints
})
```

包含持续时间、`keyFrames`和`cuePoints`属性的对象在动画库中被称为`Segment`对象。

动画段将在排队后立即开始播放，所以我们准备好加载我们的项目并看到一些动画伺服器！

# 观看你的动画运行

将你的`project`文件夹加载到 Pi 上，在 Pi SSH 会话中导航到该文件夹，并运行：

```js
sudo node my-first-animation.js
```

你应该会看到动画以我们描述的方式播放，两个伺服电机也是如此。

这真的很强大，但当你想到一个六足步行机器人时，这些线性运动不会使步态看起来真实或美观。让我们在我们的动画序列中添加一些缓动，以创建一些更有机的外观运动。

# 缓动到伺服电机动画

除非你希望你的未来行走机器人非常坚定地处于“不祥谷”，否则你需要使用缓动来创建更流畅、更自然的动画片段运动。

# 缓动如何适应动画片段

伺服电机的`keyframes`中添加了缓动函数；因此，我们不仅说明了希望伺服电机到达的位置，还说明了它是如何到达那里的。例如，这些`keyframes`：

```js
let keyframes = [
  null,
  {degrees: 180, easing: 'inoutcirc'}
]
```

将从任何位置开始的伺服电机移动到`180`，开始时速度较慢，中间加速，然后再次减速。

缓动有许多不同的选项，它们在作为 Johnny-Five 依赖项包含的`ease-component` ([`www.npmjs.com/package/ease-component`](https://www.npmjs.com/package/ease-component)) `npm`模块中有文档说明。我们将从`incirc`、`outcirc`和`inoutcirc`开始使用。

# 为我们的第一个动画添加缓动

将`my-first-animation.js`的内容复制到一个名为`easing-animations.js`的新文件中。接下来，我们将修改`keyframes`数组以包含一些缓动：

```js
let keyframes = [
 [null, {degrees: 0}, {degrees: 180, easing: 'inOutCirc'}, false, {degrees:90, easing:'outCirc'}], // servoOne
 [null, {degrees: 180}, null, null, {degrees: 90, easing:'inCirc'}] // servoTwo
]
```

让我们也将动画片段的持续时间增加，以便我们真正看到缓动带来的差异：

```js
let duration = 16000
```

然后，将其加载到树莓派上，在您的树莓派 SSH 会话中导航到文件夹，并运行以下命令：

```js
sudo node easing-animations.js
```

真实地观察`inCirc`、`outCirc`和`inOutCirc`如何影响你的动画。

# 缓动整个动画片段

为了轻松地将动画片段中的所有关键帧设置为具有相同的缓动，你可以在排队你的片段时传递一个`easing`属性。例如：

```js
myFirstAnimation.enqueue({
  keyFrames: keyframes,
  duration: duration,
  cuePoints: cuePoints,
  easing: 'inOutCirc'
})
```

上述代码将覆盖关键帧，并且所有这些都将使用`inOutCirc`缓动。现在我们已经完全探索了缓动动画片段，让我们看看动画队列以及我们如何在排队和播放时影响我们的片段。

# 学习更多关于排队和播放动画片段

当我们排队一个动画片段时，我们传递给它持续时间、提示点和关键帧。但我们也可以传递其他影响动画片段播放的选项。我们还可以调用动画对象上的方法，这些方法会影响当前正在播放和排队中的动画片段。

在我们开始修改这些之前，将`easing-animations.js`的内容复制到一个名为`playing-with-the-queue.js`的新文件中。删除末尾对`myFirstAnimation.enqueue()`的调用；这次我们想要一点控制权。

# 循环动画片段

首先，让我们添加一个函数，以便我们正常排队动画：

```js
function playMyAnimation() {
  myFirstAnimation.enqueue({
    keyFrames: keyframes,
    duration: duration,
    cuePoints: cuePoints
  })
}
```

有时候你希望你正在排队的动画段在循环中运行。让我们在`board.on('ready')`处理程序中创建一个函数，该函数将循环我们的动画段：

```js
function loopMyAnimation() {
  myFirstAnimation.enqueue({
    keyFrames: keyframes,
    duration: duration,
    cuePoints: cuePoints,
    loop: true
  })
}
```

你还可以添加`loopBackTo`属性，并将其设置为提示点的索引；动画将从指定的提示点开始循环。

如果我们想让动画向前播放，然后回到起点并重复，我们应该如何编写一个函数来设置`metronomic`属性以实现这一点？

```js
function metronomeMyAnimation() {
  myFirstAnimation.enqueue({
    keyFrames: keyframes,
    duration: duration,
    cuePoints: cuePoints,
    metronomic: true
  })
}
```

现在我们知道了如何循环和节拍动画段，让我们来探索使用动画对象更改动画段速度的方法。

# 改变动画段的速度

你可以用一个数值乘数调用`Animation.speed()`来改变当前动画段的速度。例如，调用`Animation.speed(.5)`将减半速度，而`Animation.speed(2)`将加倍它。

让我们编写一些函数来将动画段的速度减半、加倍和归一化：

```js
function halfSpeed() {
 myFirstAnimation.speed(.5)
}

function doubleSpeed() {
 myFirstAnimation.speed(2)
}

function regularSpeed() {
 myFirstAnimation.speed(1)
}
```

将这些添加到循环和节拍器功能中。

现在我们知道了如何调整动画函数的速度以及如何循环它们，让我们来谈谈暂停、播放和停止动画。

# 播放、暂停和停止动画段

如果不进行干预，动画段将一直播放，直到队列中没有剩余的内容可以播放（这意味着如果有一个循环或节拍器段，它将停留在该段上）。

但你可以移动到下一个动画：

```js
Animation.next()
```

或者你可以暂停当前段：

```js
Animation.pause()
```

再次启动它：

```js
Animation.play()
```

或者停止当前段并清除整个队列：

```js
Animation.stop()
```

让我们使用这些，以及 REPL，来玩转我们的动画和我们新发现的能力来操纵它。

# 在 REPL 中将所有这些整合在一起

将以下内容添加到`playing-with-the-queue.js`中`board.on('ready')`处理程序的末尾：

```js
board.repl.inject({
  myFirstAnimation,
  playMyAnimation,
  loopMyAnimation,
  metronomeMyAnimation,
  halfSpeed,
  doubleSpeed,
  normalSpeed
})
```

然后，将你的`project`文件夹加载到你的 Pi 上，在 Pi SSH 会话中导航到`project`文件夹，并运行以下命令：

```js
sudo node playing-with-the-queue.js
```

一旦看到`Board Initialized`，尝试一些命令来实验动画的播放方式：

```js
loopMyAnimation()
myFirstAnimation.pause()
myFirstAnimation.play()
halfSpeed()
myFirstAnimation.stop()
metronomeMyAnimation()
doubleSpeed()
playMyAnimation()
myFirstAnimation.next()Summary
```

# 概述

在本章中，我们使用伺服电机深入研究了动画库。我们学习了动画库的关键术语，如何构建动画段，如何排队，以及如何在排队段或通过调用动画对象的方法来操作播放。

# 问题

1.  为什么对于多个伺服电机的复杂运动，动画是必要的？

1.  关键帧是什么？

1.  提示点是什么？

1.  动画段由哪三部分组成？

1.  缓动对动画关键帧和段有什么影响？

1.  动画对象中的哪个方法可以停止当前段并清除动画队列？

1.  调用`Animation.speed(.25)`会对当前动画做什么？
