# 第九章：编写自己的 Node 模块

在本章中，我们将涵盖：

+   创建一个测试驱动的模块 API

+   编写一个功能模块的模拟

+   从功能到原型的重构

+   扩展模块的 API

+   将模块部署到 npm

# 介绍

自从诞生以来，一个蓬勃发展的模块生态系统一直是 Node 的核心目标之一。该框架倾向于模块化。即使是核心功能（如 HTTP）也是通过模块系统提供的。

创建我们自己的模块几乎和使用核心和第三方模块一样容易。我们只需要了解模块系统的工作原理和一些最佳实践。

一个优秀的模块是执行特定功能的高标准，而优秀的代码是多个开发周期的结果。在本章中，我们将从头开始开发一个模块，从定义其应用程序编程接口（API）开始，逐步创建我们的模块。最后，我们将把它部署到 npm 以造福所有人。

# 创建一个测试驱动的模块 API

我们将通过松散地遵循测试驱动开发（TDD）模型（参见[`en.wikipedia.org/wiki/Test-driven_development`](http://en.wikipedia.org/wiki/Test-driven_development)了解更多信息）来创建我们的模块。JavaScript 是异步的，因此代码可以在多个时间流中执行。这有时可能会构成一个具有挑战性的心智难题。

测试套件在 JavaScript 开发中是一个特别强大的工具。当测试通过时，它提供了一个质量保证过程，并且能够激发模块用户的信心。

此外，我们可以预先定义我们的测试，作为一种在开始开发之前规划预期 API 的方式。

在这个示例中，我们将通过创建一个模块的测试套件来提取 MP3 文件的统计信息。

## 准备工作

让我们创建一个名为`mp3dat`的新文件夹，里面有一个名为`index.js`的文件。然后再创建两个子文件夹：`lib`和`test`，它们都包含`index.js`。

我们还需要 MP3 文件进行测试。为简单起见，我们的模块只支持关闭错误保护的 MPEG-1 Layer 3 文件。其他类型的 MP3 文件包括 MPEG-2 和 MPEG-2.5。MPEG-1（无错误保护）可能是最常见的类型，但我们的模块可以很容易地进行扩展。我们可以从[`www.paul.sladen.org/pronunciation/torvalds-says-linux.mp3`](http://www.paul.sladen.org/pronunciation/torvalds-says-linux.mp3)获取一个 MPEG-1 Layer 3 文件。让我们把这个文件放在我们的新`mp3dat/test`文件夹中，并将其命名为`test.mp3`。

本章的重点是创建一个完全功能的模块，不需要对 MP3 文件结构有先验知识。在本章中，关于 MP3 的细节可以安全地略过，而有关模块创建的信息则至关重要。然而，我们可以从[`en.wikipedia.org/wiki/MP3`](http://en.wikipedia.org/wiki/MP3)了解更多关于 MP3 文件及其结构的信息。

## 如何做...

让我们打开`test/index.js`并设置一些变量，如下所示：

```js
var assert = require('assert');
var mp3dat = require('../index.js');
var testFile = 'test/test.mp3';

```

`assert`是一个专门用于构建测试套件的核心 Node 模块。总体思路是我们断言某件事应该是真的（或假的），如果断言是正确的，测试就通过了。`mp3dat`变量需要我们的主要（当前为空白的）`index.js`文件，该文件将加载`lib/index.js`文件，其中包含实际的模块代码。

`testFile`变量从我们模块的根目录（`mp3dat`文件夹）的角度指向我们的`test.mp3`文件。这是因为我们从模块目录的根目录运行测试。

现在我们将决定我们的 API 并编写相应的测试。让我们模仿`fs.stat`方法来设计我们的模块。我们将使用`mp3dat.stat`方法获取有关 MP3 文件的数据，该方法将接受两个参数：文件路径和一次收集统计信息后要调用的回调函数。

`mp3dat.stat`回调将接受两个参数。第一个将是错误对象，如果没有错误，应将其设置为`null`，第二个将包含我们的`stats`对象。

`stats`对象将包含`duration, bitrate, filesize, timestamp`和`timesig`属性。`duration`属性将包含一个对象，其中包含`hours, minutes, seconds`和`milliseconds`键。

例如，我们的`test.mp3`文件应该返回类似以下的内容：

```js
{ duration: { hours: 0, minutes: 0, seconds: 5, milliseconds: 186 },
  bitrate: 128000,
  filesize: 82969,
  timestamp: 5186,
  timesig: '00:00:05' }

```

现在我们已经构想出我们的 API，我们可以将其映射到断言测试中，作为强制执行 API 在整个模块开发过程中的手段。

让我们从`mp3dat`和`mp3dat.stat`开始。

```js
assert(mp3dat, 'mp3dat failed to load');

assert(mp3dat.stat, 'there should be a stat method');

assert(mp3dat.stat instanceof Function, 'stat should be a Function');

```

要测试`mp3dat.stat`函数，我们实际上必须调用它，然后在其回调函数中执行进一步的测试。

```js
mp3dat.stat(testFile, function (err, stats) {

  assert.ifError(err);

  //expected properties
  assert(stats.duration, 'should be a truthy duration property');
  assert(stats.bitrate, 'should be a truthy bitrate property');
  assert(stats.filesize, 'should be a truthy filesize property');
  assert(stats.timestamp, 'should be a truthy timestamp property');
  assert(stats.timesig, 'should be a truthy timesig property');

```

现在我们已经确定了预期的`stats`属性，我们可以进一步指定这些属性应该是什么样子的，在回调函数中，我们写下以下代码：

```js
  //expected types
  assert.equal(typeof stats.duration, 'object', 'duration should be an object type');
  assert(stats.duration instanceof Object, 'durations should be an instance of Object');
  assert(!isNaN(stats.bitrate), 'bitrate should be a number');
  assert(!isNaN(stats.filesize), 'filesize should be a number');
  assert(!isNaN(stats.timestamp), 'timestamp should be a number');

  assert(stats.timesig.match(/^\d+:\d+:\d+$/), 'timesig should be in HH:MM:SS format');

  //expected duration properties
  assert.notStrictEqual(stats.duration.hours, undefined,  'should be a duration.hours property');
  assert.notStrictEqual(stats.duration.minutes, undefined, 'should be a duration.minutes property');
  assert.notStrictEqual(stats.duration.seconds, undefined, 'should be a duration.seconds property');
  assert.notStrictEqual(stats.duration.milliseconds, undefined, 'should be a duration.milliseconds property');

  //expected duration types
  assert(!isNaN(stats.duration.hours), 'duration.hours should be a number');
  assert(!isNaN(stats.duration.minutes), 'duration.minutes should be a number');
  assert(!isNaN(stats.duration.seconds), 'duration.seconds should be a number');
  assert(!isNaN(stats.duration.milliseconds), 'duration.milliseconds should be a number');

  //expected duration properties constraints
  assert(stats.duration.minutes < 60, 'duration.minutes should be no greater than 59');
  assert(stats.duration.seconds < 60, 'duration.seconds should be no greater than 59');
  assert(stats.duration.milliseconds < 1000, 'duration.seconds should be no greater than 999');

  console.log('All tests passed');  //if we've gotten this far we are done.
});

```

现在让我们运行我们的测试。从`mp3dat`文件夹中，我们说：

```js
node test 

```

这应该返回包含以下内容的文本：

```js
AssertionError: there should be a stat method

```

这完全正确，我们还没有编写`stat`方法。

## 它是如何工作的...

当运行测试时，`assert`模块将抛出`AssertionError`，以让开发人员知道他们的代码目前与他们对所需 API 的预定义断言不一致。

在我们的单元测试文件（`test/index.js`）中，我们主要使用简单的`assert`函数（`assert.ok`的别名）。`assert`要求传递给它的第一个参数为真值。否则，它会抛出`AssertionError`，其中第二个提供的参数用于错误消息（`assert.ok`的相反是`assert.fail`，它期望一个假值）。

我们的测试失败了：

```js
assert(mp3dat.stat, 'there should be a stat method');

```

这是因为`mp3dat.stat`是`undefined`（一个假值）。

`assert`的第一个参数可以是一个表达式。例如，我们使用`stats.duration.minutes < 60`来为`duration.minutes`属性设置约束，并在`timesig`上使用`match`方法来验证正确的时间模式 HH:MM:SS。

我们还使用`assert.equal`和`assert.notStrictEqual`。`assert.equal`是一个应用类型强制相等的测试（例如，等同于`==`），`assert.strictEqual`要求值和类型匹配，`assert.notEqual`和`assert.notStrictEqual`是相应的对立面。

我们使用`assert.notStrictEqual`来确保`duration`对象的子属性`(hours, minutes`等)的存在。

## 还有更多...

有许多测试框架提供额外的描述性语法、增强功能、异步测试能力等。让我们尝试一个。

### 使用 should.js 进行单元测试

第三方的`should`模块很好地放在核心`assert`模块之上，为我们的测试添加了一些语法糖，以简化和增强描述能力。让我们安装它。

```js
npm install should 

```

现在我们可以使用`should`来重写我们的测试，如下面的代码所示：

```js
var should = require('should');
var mp3dat = require('../index.js');
var testFile = 'test/test.mp3';

should.exist(mp3dat);
mp3dat.should.have.property('stat');
mp3dat.stat.should.be.an.instanceof(Function);

mp3dat.stat(testFile, function (err, stats) {
  should.ifError(err);

  //expected properties
  stats.should.have.property('duration');
  stats.should.have.property('bitrate');
  stats.should.have.property('filesize');    
  stats.should.have.property('timestamp');
  stats.should.have.property('timesig');  

  //expected types
  stats.duration.should.be.an.instanceof(Object);
  stats.bitrate.should.be.a('number');
  stats.filesize.should.be.a('number');
  stats.timestamp.should.be.a('number');  

  stats.timesig.should.match(/^\d+:\d+:\d+$/);

  //expected duration properties
  stats.duration.should.have.keys('hours', 'minutes', 'seconds', 'milliseconds');

  //expected duration types and constraints
  stats.duration.hours.should.be.a('number');
  stats.duration.minutes.should.be.a('number').and.be.below(60);
  stats.duration.seconds.should.be.a('number').and.be.below(60);
  stats.duration.milliseconds.should.be.a('number').and.be.below(1000);  

  console.log('All tests passed');

});

```

`should`允许我们编写更简洁和描述性的测试。它的语法是自然和不言自明的。我们可以在其 Github 页面上阅读各种`should`方法：[`www.github.com/visionmedia/should.js.`](https://www.github.com/visionmedia/should.js.)

## 另请参阅

+   *在本章讨论的编写功能模块原型*

+   *在本章讨论的模块 API 扩展*

+   *在本章讨论的将模块部署到 npm*

# 编写一个功能模块原型

现在我们已经编写了我们的测试（参见前面的配方），我们准备创建我们的模块（顺便说一句，从现在开始，我们将使用我们的单元测试的`should`版本，而不是`assert`）。

在这个配方中，我们将以简单的功能风格编写我们的模块，以证明概念。在下一个配方中，我们将把我们的代码重构成一个更常见的以可重用性和可扩展性为中心的模块化格式。

## 准备工作

让我们打开我们的主要`index.js`文件，并通过`module.exports`将其链接到`lib`目录。

```js
module.exports = require('./lib');

```

这使我们可以将模块代码的核心整齐地放在`lib`目录中。

## 如何做...

我们将打开`lib/index.js`，并开始引入`fs`模块，用于读取 MP3 文件，并设置一个`bitrates`映射，将十六进制表示的值与 MPEG-1 规范定义的比特率值进行交叉引用。

```js
var fs = require('fs');

//half-byte (4bit) hex values to interpreted bitrates (bps)
//only MPEG-1 bitrates supported
var bitrates = { 1 : 32000, 2 : 40000, 3 : 48000, 4 : 56000, 5 : 64000,
  6 : 80000, 7 : 96000, 8 : 112000, 9 : 128000, A : 160000, B : 192000,
  C : 224000, D : 256000, E : 320000 };

```

现在我们将定义两个函数，`findBitRate`用于定位和转换半字节比特率，`buildStats`用于将所有收集到的信息压缩成先前确定的最终`stats`对象。

```js
function buildStats(bitrate, size, cb) {
  var magnitudes = [ 'hours', 'minutes', 'seconds', 'milliseconds'],
    duration = {}, stats,
    hours = (size / (bitrate / 8) / 3600);

  (function timeProcessor(time, counter) {
      var timeArray = [], factor = (counter < 3) ? 60 : 1000 ;
      if (counter) {        
        timeArray = (factor * +('0.' + time)).toString().split('.');
      }

      if (counter < magnitudes.length - 1) {
        duration[magnitudes[counter]] = timeArray[0] || Math.floor(time);
        duration[magnitudes[counter]] = +duration[magnitudes[counter]];
        counter += 1;
        timeProcessor(timeArray[1] || time.toString().split('.')[1], counter);
        return;
      }
        //round off the final magnitude
        duration[magnitudes[counter]] = Math.round(timeArray.join('.'));
  }(hours, 0));

  stats = {
    duration: duration,
    bitrate: bitrate,
    filesize: size,
    timestamp: Math.round(hours * 3600000),
    timesig: ''
  };

  function pad(n){return n < 10 ? '0'+n : n}  
   magnitudes.forEach(function (mag, i) {
   if (i < 3) {
    stats.timesig += pad(duration[mag]) + ((i < 2) ? ':' : '');
   }
  });

  cb(null, stats);
}

```

`buildStats`接受`bitrate, size`和`cb`参数。它使用`bitrate`和`size`来计算音轨中的秒数，然后使用这些信息生成`stats`对象，并通过`cb`函数传递。

为了将`bitrate`传递给`buildStats`，让我们按照以下代码编写`findBitRate`函数：

```js
function findBitRate(f, cb) {
   fs.createReadStream(f)
    .on('data', function (data) {
      var i;
      for (i = 0; i < data.length; i += 2) {
        if (data.readUInt16LE(i) === 64511) {
          this.destroy();
          cb(null, bitrates[data.toString('hex', i + 2, i + 3)[0]]);
          break;
        };
    }
  }).on('end', function () {
    cb(new Error('could not find bitrate, is this definitely an MPEG-1 MP3?'));
  });   
}

```

最后，我们公开了一个`stat`方法，它利用我们的函数来生成`stats`对象：

```js
exports.stat = function (f, cb) {
  fs.stat(f, function (err, fstats) {
    findBitRate(f, function (err, bitrate) {
      if (err) { cb(err); return; }
      buildStats(bitrate, fstats.size, cb);    
    });
  });
}

```

现在让我们运行我们从上一个示例中的(`should`)测试：

```js
node test 

```

它应该输出以下内容：

```js
All tests passed

```

## 它是如何工作的...

`exports`对象是 Node 平台的核心部分。它是`require`的另一半。当我们需要一个模块时，添加到`exports`的任何属性都通过`require`暴露出来。因此，当我们这样做时：

```js
var mp3dat = require('mp3dat');

```

我们可以通过`mp3dat.stat`访问`exports.stat`，甚至可以通过`require('mp3dat').stat`访问（假设我们已经将`mp3dat`安装为一个模块，参见*将模块部署到 npm*）。

如果我们想为整个模块公开一个函数，我们使用`module.exports`，就像我们在本示例的*准备就绪*部分中设置的顶级`index.js`文件一样。

我们的`stat`方法首先调用`fs.stat`，并传递用户提供的文件名（f）。我们使用提供的`fstats`对象来检索文件的大小，然后将其传递给`buildStats`。也就是说，在我们调用`findBitRate`来检索 MP3 的`bitrate`之后，我们也将其传递给`buildStats`。

`buildStats`回调直接通过我们的`stat`方法的回调传递，用户回调的执行起源于`buildStats`。

`findBitRate`创建了用户提供文件（f）的`readStream`，并循环遍历每个发出的`data`块，每次两个字节，从而减少搜索时间。我们之所以能这样做，是因为我们正在寻找两个字节的同步字，它们总是在可以被二整除的位置。在十六进制中，同步字是`FFFB`，作为 16 字节的小端无符号整数是`64511`（这仅适用于没有错误保护的 MPEG-1 MP3 文件）。

MP3 同步字后面的四个比特（半字节）包含比特率值。因此，我们通过`Buffer.toString`方法将其传递，要求十六进制输出，然后与我们的`bitrates`对象映射进行匹配。在我们的`test.mp3`文件的情况下，半字节的十六进制值为`9`，表示每秒`128000`比特的比特率。

一旦我们找到比特率，我们执行回调并调用`this.destroy`，这会突然终止我们的`readStream`，防止`end`事件被触发。只有在没有发现比特率的情况下，`end`事件才会发生，此时我们通过回调发送错误。

`buildStats`接收`bitrate`并将其除以`8`得到每秒的字节数（8 位为 1 字节）。将 MP3 的总字节数除以每秒的字节数得到秒数。然后我们再除以 3,600 得到`hours`变量，然后将其传递到嵌入的`timeProcessor`函数中。`timeProcessor`简单地通过`magnitudes`数组（小时，分钟，秒，毫秒）进行递归，直到`seconds`被准确转换并分配到每个数量级，从而得到我们的`duration`对象。然后，我们使用计算出的持续时间（以任何形式）来构建我们的`timestamp`和`timesig`属性。

## 还有更多...

模块的使用示例可以成为最终用户的重要资源。让我们为我们的新模块编写一个示例。

### 编写模块使用示例

我们将在`mp3dat`文件夹中创建一个`examples`文件夹，并创建一个名为`basic.js`的文件（用于基本用法示例），将以下内容写入其中：

```js
var mp3dat = require('../index.js');

mp3dat.stat('../test/test.mp3', function (err, stats) {
  console.log(stats);
});

```

这应该导致控制台输出以下内容：

```js
{ duration: { hours: 0, minutes: 0, seconds: 5, milliseconds: 186 },
  bitrate: 128000,
  filesize: 82969,
  timestamp: 5186,
  timesig: '00:00:05' }

```

## 另请参阅

+   *在本章讨论的创建测试驱动模块 API*

+   *从功能到原型的重构*在本章中讨论

+   *将模块部署到 npm*在本章中讨论

# 从功能到原型的重构

在上一篇文章中创建的功能模拟可以帮助我们对概念有所了解，并且对于范围较小、简单的模块可能是完全足够的。

然而，原型模式（以及其他模式）通常被模块创建者使用，经常用于 Node 的核心模块，并且是原生 JavaScript 方法和对象的基础。

**原型继承**稍微更节省内存。原型上的方法在调用之前不会被实例化，并且它们会被重复使用，而不是在每次调用时重新创建。

另一方面，它可能比我们上一个配方的过程式风格稍慢，因为 JavaScript 引擎需要额外的开销来遍历原型链。尽管如此，将模块视为用户可以创建实例的实体，并以（例如，基于原型的方法）实现它们可能更合适。首先，这样可以更容易地通过克隆和原型修改进行编程扩展。这为最终用户提供了极大的灵活性，同时模块代码的核心完整性保持不变。

在这个配方中，我们将根据原型模式重写上一个任务中的代码。

## 准备工作

让我们开始编辑`mp3dat/lib`中的`index.js`。

## 如何做...

首先，我们需要创建一个构造函数（使用`new`调用的函数），我们将其命名为`Mp3dat`：

```js
var fs = require('fs');

function Mp3dat(f, size) {
  if (!(this instanceof Mp3dat)) {
    return new Mp3dat(f, size);
  }
  this.stats = {duration:{}};
}

```

我们还像上一个任务一样需要`fs`模块。

让我们向构造函数的原型添加一些对象和方法：

```js
Mp3dat.prototype._bitrates = { 1 : 32000, 2 : 40000, 3 : 48000, 4 : 56000, 5 : 64000, 6 : 80000, 7 : 96000, 8 : 112000, 9 : 128000, A : 160000, B : 192000, C : 224000, D : 256000, E : 320000 };

Mp3dat.prototype._magnitudes = [ 'hours', 'minutes', 'seconds', 'milliseconds'];

Mp3dat.prototype._pad = function (n) { return n < 10 ? '0' + n : n; }  

Mp3dat.prototype._timesig = function () {
  var ts = '', self = this;;
  self._magnitudes.forEach(function (mag, i) {
   if (i < 3) {
    ts += self._pad(self.stats.duration[mag]) + ((i < 2) ? ':' : '');
   }
  });
  return ts;
}

```

我们的新`Mp3dat`属性中的三个（`_magnitudes、_pad`和`_timesig`）在`buildStats`函数中以某种形式被包含。我们在它们的名称前加上下划线(_)来表示它们是私有的。这只是一个约定，JavaScript 实际上并没有将它们私有化。

现在我们将上一个配方的`findBitRate`函数转换如下：

```js
Mp3dat.prototype._findBitRate = function(cb) {
  var self = this;
   fs.createReadStream(self.f)
    .on('data', function (data) {
      var i = 0;
       for (i; i < data.length; i += 2) {
        if (data.readUInt16LE(i) === 64511) {
          self.bitrate = self._bitrates[data.toString('hex', i + 2, i + 3)[0]];
          this.destroy();
          cb(null);
          break;
        };
    }
  }).on('end', function () {
    cb(new Error('could not find bitrate, is this definitely an MPEG-1 MP3?'));
  });
}

```

唯一的区别是我们从对象（`self.f`）中加载文件名，而不是通过第一个参数，我们将`bitrate`加载到对象上，而不是通过`cb`的第二个参数发送它。

现在，为了将`buildStats`转换为原型模式，我们编写以下代码：

```js
Mp3dat.prototype._buildStats = function (cb) {
  var self = this,
  hours = (self.size / (self.bitrate / 8) / 3600);

  self._timeProcessor(hours, function (duration) {
    self.stats = {
      duration: duration,
      bitrate: self.bitrate,
      filesize: self.size,
      timestamp: Math.round(hours * 3600000),
      timesig: self._timesig(duration, self.magnitudes)
    };
    cb(null, self.stats);

  });
}

```

我们的`_buildStats`原型方法比上一个任务中的`buildStats`方法要小得多。我们不仅提取了它的内部`magnitudes`数组、`pad`实用函数和时间签名功能（将其包装成自己的`_timesig`方法），还将内部递归的`timeProcessor`函数外包到了一个原型方法中。

```js
Mp3dat.prototype._timeProcessor = function (time, counter, cb) {
  var self = this, timeArray = [], factor = (counter < 3) ? 60 : 1000,
    magnitudes = self._magnitudes, duration = self.stats.duration;

  if (counter instanceof Function) {
    cb = counter;
    counter = 0;
  }

  if (counter) {        
    timeArray = (factor * +('0.' + time)).toString().split('.');
  }
  if (counter < magnitudes.length - 1) {
    duration[magnitudes[counter]] = timeArray[0] || Math.floor(time);
    duration[magnitudes[counter]] = +duration[magnitudes[counter]];
    counter += 1;
    self._timeProcessor.call(self, timeArray[1] || time.toString().split('.')[1], counter, cb);
    return;
  }
    //round off the final magnitude (milliseconds)
    duration[magnitudes[counter]] = Math.round(timeArray.join('.'));
    cb(duration);
}

```

最后，我们编写`stat`方法（没有下划线前缀，因为它是供公共使用的），并导出`Mp3dat`对象。

```js
Mp3dat.prototype.stat = function (f, cb) {
  var self = this;
  fs.stat(f, function (err, fstats) {
    self.size = fstats.size;
    self.f = f;
    self._findBitRate(function (err, bitrate) {
      if (err) { cb(err); return; }
      self._buildStats(cb);
    });    
  });
}

module.exports = Mp3dat();

```

我们可以通过运行我们在第一个配方中构建的测试来确保一切都正确。在`mp3dat`文件夹的命令行中，我们说：

```js
node test 

```

应该输出：

```js
All tests passed

```

## 它是如何工作的...

在上一个配方中，我们有一个`exports.stat`函数，它调用`findBitRate`和`buildStats`函数来获取`stats`对象。在我们重构的模块中，我们将`stat`方法添加到原型上，并通过`module.exports`导出整个`Mp3dat`构造函数。

我们不必使用`new`将`Mp3dat`传递给`module.exports`。我们的函数在直接调用时生成新的实例，代码如下：

```js
  if (!(this instanceof Mp3dat)) {
    return new Mp3dat();
  }

```

这真的是一个保险策略。使用`new`初始化构造函数更有效（尽管在边缘上）。

我们重构的代码中的`stat`方法与先前任务中的`exports.stat`函数不同。它不是将文件名和指定 MP3 的大小作为参数传递给`findBitRate`和`buildStats`，而是通过`this`将它们分配给父对象（将其分配给`self`以避免`this`的新回调范围重新分配）。

然后调用`_findBitRate`和`_buildStats`方法，最终生成`stats`对象并将其传递回用户的回调。

在我们的`test.mp3`文件上运行`mp3dat.stats`之后，我们重构的`mp3dat`模块对象将包含以下内容：

```js
{ stats:
   { duration: { hours: 0, minutes: 0, seconds: 5, milliseconds: 186 },
     bitrate: 128000,
     filesize: 82969,
     timestamp: 5186,
     timesig: '00:00:05' },
  size: 82969,
  f: 'test/test.mp3',
  bitrate: 128000 }

```

然而，在前一个示例中，返回的对象将简单地如下所示：

```js
{ stat: [Function] }

```

功能风格揭示了 API。我们重构的代码允许用户以多种方式与信息进行交互（通过`stats`和`mp3dat`对象）。我们还可以扩展我们的模块，并在`stats`对象之外的其他时间填充`mp3dat`。

## 还有更多...

我们可以构建我们的模块，使其更容易使用。

### 将 stat 函数添加到初始化的 mp3dat 对象中

如果我们想直接将我们的`stat`函数暴露给`mp3dat`对象，从而允许我们直接查看 API（例如，使用`console.log`），我们可以通过删除`Mp3dat.prototype.stat`并修改`Mp3dat`来添加它如下：

```js
function Mp3dat() {
  var self = this;
  if (!(this instanceof Mp3dat)) {
    return new Mp3dat();
  }
  self.stat = function (f, cb) {
    fs.stat(f, function (err, fstats) {
      self.size = fstats.size;
      self.f = f;
      self._findBitRate(function (err, bitrate) {
        if (err) { cb(err); return; }
        self._buildStats(cb);
      });    
    });
  }  
  self.stats = {duration:{}};
}

```

然后我们的最终对象变成：

```js
{ stat: [Function],
  stats:
   { duration: { hours: 0, minutes: 0, seconds: 5, milliseconds: 186 },
     bitrate: 128000,
     filesize: 82969,
     timestamp: 5186,
     timesig: '00:00:05' },
  size: 82969,
  f: 'test/test.mp3',
  bitrate: 128000 }

```

或者，如果我们不关心将`stats`对象和其他`Mp3dat`属性推送到模块用户，我们可以将一切保持原样，只需更改以下代码：

```js
module.exports = Mp3dat()

```

到：

```js
exports.stat = function (f, cb) {
  var m = Mp3dat();
  return Mp3dat.prototype.stat.call(m, f, cb);
}

```

这使用`call`方法将`Mp3dat`范围应用于`stat`方法（允许我们依赖`stat`方法），并将返回一个具有以下内容的对象：

```js
{ stat: [Function] }

```

就像我们的模块的第一次写作一样，只是我们仍然保留了原型模式。这种第二种方法稍微更有效。

### 允许多个实例

我们的模块是一个单例，因为它返回已初始化的`Mp3dat`对象。这意味着无论我们需要多少次并将其分配给变量，模块用户始终将引用相同的对象，即使`Mp3dat`在父脚本加载的不同子模块中被需要。

这意味着如果我们尝试同时运行两个`mp3dat.stat`方法，将会发生糟糕的事情。在需要多次引用我们的模块的情况下，持有相同对象的两个变量最终可能会互相覆盖属性，导致不可预测（和令人沮丧）的代码。最有可能的结果是`readStreams`会发生冲突。

克服这一点的一种方法是修改以下内容：

```js
module.exports = Mp3dat()

```

到：

```js
module.exports = Mp3dat

```

然后使用以下代码加载两个实例：

```js
var Mp3dat = require('../index.js'),
	mp3dat = Mp3dat(),
      mp3dat2 = Mp3dat();

```

如果我们想要提供单例和多个实例，我们可以在构造函数的原型中添加一个`spawnInstance`方法：

```js
Mp3dat.prototype.spawnInstance = function () {
  return Mp3dat();
}

module.exports = Mp3dat();

```

然后我们可以做如下事情：

```js
var mp3dat = require('../index.js'),
   mp3dat2 = mp3dat.spawnInstance();

```

`mp3dat`和`mp3dat2`都将是单独的`Mp3dat`实例，而在以下情况下：

```js
var mp3dat = require('../index.js'),
   mp3dat2 = require('../index.js');

```

两者将是相同的实例。

## 另请参阅

+   *在本章中讨论的编写功能模块模拟*

+   *在本章中讨论的扩展模块的 API*

+   *在本章中讨论的将模块部署到 npm*

# 扩展模块的 API

我们可以以许多方式扩展我们的模块，例如，我们可以使其支持更多的 MP3 类型，但这只是例行工作。只需找出不同的同步字和不同类型的 MP3 的比特率，然后将它们添加到相关位置。

对于更有趣的冒险，我们可以扩展 API，为我们的模块用户创建更多选项。

由于我们使用流来读取我们的 MP3 文件，我们可以允许用户传入文件名或 MP3 数据流，提供简单（使用简单文件名）和灵活（使用流）两种方式。这样我们就可以启动下载流、STDIN 流，或者实际上任何 MP3 数据流。

## 准备工作

我们将从前一篇食谱的*允许多个实例*部分的*还有更多..*中继续使用我们的模块。

## 如何做...

首先，我们将为我们的新 API 添加一些更多的测试。在`tests/index.js`中，我们将从`mp3dat.stat`调用中提取回调函数到全局范围，并将其命名为`cb:`。

```js
function cb (err, stats) {
  should.ifError(err);

  //expected properties
  stats.should.have.property('duration');

  //...all the other unit tests here

  console.log('passed');

};

```

现在我们将调用`stat`以及一个我们将要编写并命名为`statStream`的方法：

```js
mp3dat.statStream({stream: fs.createReadStream(testFile),
  size: fs.statSync(testFile).size}, cb);

mp3dat2.stat(testFile, cb);

```

注意我们使用了两个`Mp3dat`实例（`mp3dat`和`mp3dat2`）。所以我们可以同时运行`stat`和`statStream`测试。由于我们正在创建一个`readStream`，我们需要在我们的`[tests/index.js]`文件的顶部引入`fs`。

```js
var should = require('should');
var fs = require('fs');
var mp3dat = require('../index.js'),
  mp3dat2 = mp3dat.spawnInstance();

```

我们还将为`statStream`方法添加一些顶层的`should`测试，如下所示：

```js
should.exist(mp3dat);
mp3dat.should.have.property('stat');
mp3dat.stat.should.be.an.instanceof(Function);
mp3dat.should.have.property('statStream');
mp3dat.statStream.should.be.an.instanceof(Function);

```

现在来满足我们测试的期望。

在`lib/index.js`中，我们向`Mp3dat`的原型添加了一个新的方法。它不再接受第一个参数的文件名，而是接受一个对象（我们将其称为`opts`），该对象必须包含`stream`和`size`属性：

```js
Mp3dat.prototype.statStream = function (opts, cb) {
  var self = this,
    errTxt = 'First arg must be options object with stream and size',
    validOpts = ({}).toString.call(opts) === '[object Object]'
      && opts.stream
      && opts.size
      && 'pause' in opts.stream
      && !isNaN(+opts.size);
   lib
  if (!validOpts) {
    cb(new Error(errTxt));
    return;
  }

  self.size = opts.size;
  self.f = opts.stream.path;

  self.stream = opts.stream;

  self._findBitRate(function (err, bitrate) {
    if (err) { cb(err); return; }
    self._buildStats(cb);
  });    

}

```

最后，对`_findBitRate`进行一些修改，我们就完成了。

```js
Mp3dat.prototype._findBitRate = function(cb) {
  var self = this,
    stream = self.stream || fs.createReadStream(self.f);
  stream
    .on('data', function (data) {
      var i = 0;
       for (i; i < data.length; i += 2) {
        if (data.readUInt16LE(i) === 64511) {
          self.bitrate = self._bitrates[data.toString('hex', i + 2, i + 3)[0]];
          this.destroy();
          cb(null);
          break;
        };
//rest of the _findBitRate function...

```

我们有条件地挂接到传入的流，或者我们从给定的文件名创建一个流。

让我们运行我们的测试（从`mp3dat`文件夹）：

```js
node tests 

```

结果应该是：

```js
passed
passed

```

一个用于`stat`，一个用于`statStream`。

## 它是如何工作的...

我们已经在使用流来检索我们的数据。我们只需通过修改`_findBitRate`来向用户公开这个接口，使其可以从文件名生成自己的流，或者如果流存在于父构造函数的属性中（`self.stream`），它只需将该流插入到已经存在的流程中。

然后，我们通过定义一个新的 API 方法`statStream`来使这个功能对模块用户可用。我们首先通过为其制定测试来概念化它，然后通过`Mp3dat.prototype`来定义它。

`statStream`方法类似于`stat`方法（实际上，我们可以将它们合并，参见*还有更多..*）。除了检查输入的有效性之外，它只是向`Mp3dat`实例添加了一个属性：`stream`属性，它取自`opts.stream`。为了方便起见，我们将`opts.stream.path`与`self.f`进行交叉引用（这取决于流的类型，这可能有或者没有）。这基本上是多余的，但对于用户的调试目的可能是有用的。

在`statStream`的顶部，我们有`validOpts`变量，其中有一系列由`&&`条件连接的表达式。这是一堆`if`语句的简写。如果这些表达式中的任何一个测试失败，`opts`对象就无效。一个有趣的表达式是`opts.stream`中的`'pause'`，它测试`opts.stream`是否绝对是一个流或者是从流继承而来的（所有流都有一个`pause`方法，`in`检查整个原型链中的属性）。在`validOpts`测试中另一个值得注意的表达式是`!isNaN(+opts.size)`，它检查`opts.size`是否是一个有效的数字。前面的`+`将其转换为`Number`类型，`!isNaN`检查它是否不是`"not a number"`（JavaScript 中没有`isNumber`，所以我们使用`!isNaN`）。

## 还有更多...

现在我们有了这个新方法。让我们写一些更多的例子。我们还将看到如何将`statStream`和`stat`合并在一起，并通过使其发出事件来进一步增强我们的模块。

### 制作标准输入流示例

为了演示与其他流的使用，我们可以编写一个使用`process.stdin`流的示例，如下所示：

```js
//to use try :
// cat ../test/test.mp3 | node stdin_stream.js 82969
// the argument (82969) is the size in bytes of the mp3

if (!process.argv[2]) {
  process.stderr.write('\nNeed mp3 size in bytes\n\n');
  process.exit();
}

var mp3dat = require('../');
process.stdin.resume();
mp3dat.statStream({stream : process.stdin, size: process.argv[2]}, function (err, stats) {
  if (err) { console.log(err); }
  console.log(stats);
});

```

我们在示例中包含了注释，以确保我们的用户了解如何使用它。我们在这里所做的就是接收`process.stdin`流和文件大小，然后将它们传递给我们的`statStream`方法。

### 制作 PUT 上传流示例

在第二章的*处理文件上传食谱*中，*探索 HTTP 对象*，我们在那个食谱的*还有更多..*部分创建了一个 PUT 上传实现。

我们将从该示例中获取`put_upload_form.html`文件，并在`mp3dat/examples`文件夹中创建一个名为`HTTP_PUT_stream.js`的新文件。

```js
var mp3dat = require('../../mp3dat');
var http = require('http');
var fs = require('fs');
var form = fs.readFileSync('put_upload_form.html');
http.createServer(function (req, res) {
  if (req.method === "PUT") {
    mp3dat.statStream({stream: req, size:req.headers['content-length']}, function (err, stats) {
      if (err) { console.log(err); return; }
      console.log(stats);
    });

  }
  if (req.method === "GET") {
    res.writeHead(200, {'Content-Type': 'text/html'});
    res.end(form);
  }
}).listen(8080);

```

在这里，我们创建了一个服务器，用于提供`put_upload_form.html`文件。HTML 文件允许我们指定要上传的文件（必须是有效的 MP3 文件），然后将其发送到服务器。

在我们的服务器中，我们将`req`（一个流）传递给`stream`属性和`req.headers['content-length']`，这样我们就可以得到 MP3 的大小（以浏览器通过`Content-Length`头部指定的字节为单位）。

然后我们通过在控制台上记录`stats`来完成（我们还可以通过以 JSON 形式将`stats`发送回浏览器来扩展此示例）。

### 合并 stat 和 statStream

`stat`和`statStream`之间有很多相似的代码。通过一些重构，我们可以将它们合并成一个方法，允许用户将包含文件名的字符串或包含流和大小属性的对象直接传递给`stat`方法。

首先，我们需要更新我们的测试和示例。在`test/index.js`中，我们应该删除以下代码：

```js
mp3dat.should.have.property('statStream');
mp3dat.statStream.should.be.an.instanceof(Function);

```

由于我们正在将`statStream`合并到`stat`中，我们对`stat`和`statStream`的两次调用应该变成：

```js
mp3dat.stat({stream: fs.createReadStream(testFile),
    size: fs.statSync(testFile).size}, cb);
mp3dat2.stat(testFile, cb);

```

`examples/stdin_stream.js`中的`statStream`行应该变成：

```js
mp3dat.stat({stream : process.stdin, size: process.argv[2]}

```

在`HTTP_PUT_stream.js`中应该是：

```js
mp3dat.stat({stream: req, size: req.headers['content-length']}

```

在`lib/index.js`中，我们删除`streamStat`方法，插入`_compile`方法：

```js
Mp3dat.prototype._compile =  function (err, fstatsOpts, cb) {
  var self = this;
  self.size = fstatsOpts.size;
  self.stream = fstatsOpts.stream;
    self._findBitRate(function (err, bitrate) {
    if (err) { cb(err); return; }
    self._buildStats(cb);
  });    
}

```

最后，我们修改我们的`Mp3dat.prototype.stat`方法如下：

```js
Mp3dat.prototype.stat = function (f, cb) {
  var self = this, isOptsObj = ({}).toString.call(f) === '[object Object]';

  if (isOptsObj) {
    var opts = f, validOpts = opts.stream && opts.size
      && 'pause' in opts.stream && !isNaN(+opts.size);
    errTxt = 'First arg must be options object with stream and size'

    if (!validOpts) { cb(new Error(errTxt)); return; }

    self.f = opts.stream.path;
    self._compile(null, opts, cb);
    return;
  }

  self.f = f;
  fs.stat(f, function (err, fstats) {
    self._compile.call(self, err, fstats, cb);
  });
}

```

实际生成`stats`的代码已放入`_compile`方法中。如果第一个参数是一个对象，我们假设是一个流，`stats`就承担了以前的`statStream`的角色，调用`_compile`并从函数中提前返回。如果不是，我们假设是一个文件名，并使用 JavaScript 的`call`方法在`fs.stat`回调中调用`_compile`，确保我们的`this`/`self`变量通过`_compile`方法传递。

### 集成 EventEmitter

在本书中，我们通常通过回调参数或通过侦听事件来从模块中接收数据。我们可以进一步扩展我们的`modules`接口，允许用户通过使 Node 的`EventEmitter`采用我们的`Mp3dat`构造函数来侦听事件。

我们需要引入`events`和`util`模块，然后通过将`Mp3dat`的`this`对象分配给它来将`Mp3dat`与`EventEmitter`连接起来，然后使用`util.inherits`给它`Mp3dat EventEmitter`的超级能力：

```js
var fs = require('fs'),
  EventEmitter = require('events').EventEmitter,
  util = require('util');

function Mp3dat() {
  if (!(this instanceof Mp3dat)) {
    return new Mp3dat();
  }  
  EventEmitter.call(this);
  this.stats = {duration:{}};
}

util.inherits(Mp3dat, EventEmitter);

```

现在我们只需遍历`Mp3dat`的现有方法，并在相关位置插入`emit`事件。一旦找到`bitrate`，我们就可以像下面这样`emit`它：

```js
Mp3dat.prototype._findBitRate = function(cb) {
//beginning of _findBitRate method
       for (i; i < data.length; i += 2) {
        if (data.readUInt16LE(i) === 64511) {
          self.bitrate = self._bitrates[data.toString('hex', i + 2, i + 3)[0]];
          this.destroy();
          self.emit('bitrate', self.bitrate);
          cb(null);
          break;
        };
 //rest of _findBitRate method

```

在我们回调错误的地方，我们也可以像下面的代码一样发出错误：

```js
//last part of _findBitRate method
  }).on('end', function () {
    var err = new Error('could not find bitrate, is this definately an MPEG-1 MP3?');
    self.emit('error', err);
    cb(err);
  });

```

然后是时间签名：

```js
Mp3dat.prototype._timesig = function () {
 //_timesig function code....
  self.emit('timesig', ts);
  return ts;
}

```

当然，`stats`对象：

```js
Mp3dat.prototype._buildStats = function (cb) {
//_buildStats code
  self._timeProcessor(hours, function (duration) {
   //_timeProcessor code
    self.emit('stats', self.stats);
    if (cb) { cb(null, self.stats); }    
  });
}

```

我们还在`_buildStats`中添加了`if (cb)`，因为如果用户选择侦听事件，则回调可能不再必要。

如果模块用户动态生成`Mp3dat`实例，他们可能希望有一种方法来连接到生成的实例事件：

```js
Mp3dat.prototype.spawnInstance = function () {
  var m = Mp3dat();
  this.emit('spawn', m);
  return m;
}

```

最后，为了允许链接，我们还可以从两个地方的`stat`函数返回`Mp3dat`实例。首先在`isOptsObj`块内如下：

```js
Mp3dat.prototype.stat = function (f, cb) {
//stat code
  if (isOptsObj) {
    //other code here
    self._compile(null, opts, cb);
    return self;
  }

```

然后在函数的最后，如下所示的代码：

```js
  //prior stat code
  self.f = f;
  fs.stat(f, function (err, fstats) {
    self._compile.call(self, err, fstats, cb);
  });
  return self;
}

```

这是因为我们根据检测到的输入（文件名或流）从函数中提前返回，所以我们必须从两个地方返回`self`。

现在我们可以为我们的新用户界面编写一个示例。让我们在`mp3dat/examples`中创建一个名为`event_emissions.js`的新文件。

```js
var mp3dat = require('../index');

mp3dat
  .stat('../test/test.mp3')
  .on('bitrate', function (bitrate) {
    console.log('Got bitrate:', bitrate);
  })
  .on('timesig', function (timesig) {
     console.log('Got timesig:', timesig);
  })
  .on('stats', function (stats) {
     console.log('Got stats:', stats);
     mp3dat.spawnInstance();
  })
  .on('error', function (err) {
     console.log('Error:', err);
  })
  .on('spawn', function (mp3dat2) {
    console.log('Second mp3dat', mp3dat2);
  });

```

## 另请参阅

+   *在本章中讨论创建一个测试驱动的模块 API*

+   *处理文件上传*在第二章中讨论，探索 HTTP 对象

+   *在本章中讨论将模块部署到 npm*

# 将模块部署到 npm

现在我们已经创建了一个模块，我们可以使用相同的集成工具与世界其他地方分享它：`npm`。

## 做好准备

在我们可以部署到 npm 之前，我们需要创建一个`package.json`文件，所以让我们为我们的模块做这个。在`mp3dat`中，我们将创建`package.json`并添加以下代码：

```js
{
  "author": "David Mark Clements <contact@davidmarkclements.com> (http://davidmarkclements.com)",
  "name": "mp3dat",
  "description": "A simple MP3 parser that returns stat infos in a similar style to fs.stat for MP3 files or streams. (MPEG-1 compatible only)",
  "version": "0.0.1",
  "homepage": "http://nodecookbook.com/mp3dat",
  "repository": {
    "type": "git",
    "url": "git://github.com/davidmarkclements/mp3dat.git"
  },
  "main": "./lib/index.js",
  "scripts": {
    "test": "node test"
  },
  "engines": {
    "node": "~0.6.13"
  },
  "dependencies": {},
  "devDependencies": {}
}

```

当然，我们可以插入我们自己的名称和包的名称。创建`package.json`文件的另一种方法是使用`npm init`，它通过命令行询问一系列问题，然后生成`package.json`文件。

我们可以在`package.json`中指定一个存储库。使用在线存储库（如 GitHub）来管理版本控制、共享代码并允许其他人在您的代码上工作是一个好主意。请参阅[`help.github.com/`](http://help.github.com/)开始使用。

`main`属性很重要。它定义了我们模块的入口点，在我们的情况下是`./lib/index.js`（尽管我们也可以指定`./index.js`来加载`./lib/index.js`）。通过将`scripts.test`定义为`node test`，我们现在可以运行`npm test`（或者一旦通过`npm`安装了`mp3dat`，就可以运行`npm mp3dat test`）来执行我们的单元测试。

我们将按照上一个步骤中的方式将我们的模块部署到 npm，其中`stat`和`statStream`都合并到了`stat`中，并且我们已经集成了`EventEmitter`。

## 如何做...

要部署到`npm`，我们必须拥有开发者账户。我们通过执行以下操作来实现这一点：

```js
npm adduser 

```

填写我们想要的用户名、密码和联系邮箱。就这样，我们现在注册了。

在继续发布我们的模块之前，我们会想要测试`npm`是否会在我们的系统上安装它。在`mp3dat`中，我们执行以下操作：

```js
sudo npm install -g 

```

然后，如果我们从命令行运行`node`，我们应该能够：

```js
require('mp3dat') 

```

如果没有收到错误消息。如果成功了，我们可以继续发布我们的模块！在`mp3dat`中，我们说以下内容：

```js
npm publish 

```

现在，如果我们转到一个完全不同的文件夹（比如我们的主文件夹）并输入以下内容：

```js
npm uninstall mp3dat
npm install mp3dat 

```

`npm`应该从其存储库中安装我们的包。

我们可以用以下命令来双重检查它是否存在：

```js
npm search mp3dat 

```

或者，如果这花费的时间太长，我们可以在浏览器中转到[`search.npmjs.org/`](http://search.npmjs.org/)。我们的模块可能会出现在主页上（其中包含最近发布的模块）。或者我们可以点击[`search.npmjs.org/#/mp3dat`](http://search.npmjs.org/#/mp3dat)直接转到我们模块的 npm 注册页面。

## 它是如何工作的...

`npm`是一个用 Node 编写的命令行脚本，为开发和发布模块提供了一些出色的工具。这些工具确实做到了它们所说的，`adduser`添加用户，`install`安装，`publish`发布。这真的非常优雅。

在服务器端，`npm`注册表由一个 CouchDB 数据库支持，该数据库保存了每个包的类似 JSON 的数据。甚至有一个我们可以连接到的 CouchDB `_changes`字段。在命令行上，我们可以这样做：

```js
curl http://isaacs.couchone.com/registry/_changes?feed=continuous&include_docs=true 

```

观察模块在实时中添加和修改。如果没有发生任何事情，我们可以打开另一个终端并输入以下命令：

```js
mp3dat unpublish
mp3dat publish 

```

这将导致 CouchDB 更改反馈更新。

## 还有更多...

`npm`有一些非常好的功能，让我们来看看其中的一些。

### npm link

`npm link`命令对于模块作者来说可能很有用。

在开发过程中，如果我们想要将`mp3dat`作为全局模块进行引用，例如`require('mp3dat')`，每次进行更改时，我们可以通过运行以下命令来更新全局包：

```js
sudo npm install . -g 

```

然而，当我们运行以下命令时，`npm link`提供了一个更简单的解决方案：

```js
sudo npm link 

```

在我们的`mp3dat`文件夹中，从我们的全局`node_modules`文件夹到我们的工作目录创建了一个符号链接。这会导致 Node 将`mp3dat`视为已安装的全局模块，但我们对开发副本所做的任何更改都将在全局范围内反映出来。当我们完成开发并希望在系统上冻结模块时，我们只需取消链接，如下所示：

```js
sudo npm unlink -g mp3dat 

```

### .npmignore 和 npm 版本

我们的`example`文件可能在 GitHub 上很方便，但我们可能会决定它们在`npm`中没有什么好处。我们可以使用`.npmignore`文件来阻止某些文件被发布到`npm`包中。让我们在`mp3dat`文件夹中创建`.npmignore`，并放入：

```js
examples/

```

现在，当我们重新发布到`npm`注册表时，我们的新包将不包括`examples`文件夹。在我们可以发布之前，我们要么取消发布，要么更改我们包的版本（或者我们可以使用`--force`参数）。让我们改变版本，然后再次发布：

```js
npm version 0.0.2 --message "added .npmignore"
npm publish 

```

更改版本还将改变我们的`package.json`文件到新的版本号。

## 参见

+   *编写一个功能模块的模拟*在本章讨论

+   *从功能到原型的重构*在本章讨论

+   *在本章讨论的扩展模块的 API*

+   *使用 Cradle 访问 CouchDB 更改流*在第四章中讨论，与数据库交互
