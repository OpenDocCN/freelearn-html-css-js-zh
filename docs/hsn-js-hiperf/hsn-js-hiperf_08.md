# 第八章：数据格式 - 查看除 JSON 之外的不同数据类型

我们几乎已经完成了关于服务器端 JavaScript 的讨论。一个话题似乎鲜为人知，但在与其他系统进行接口或使事情更快时经常出现的话题是以不同格式传输数据。其中最常见的，如果不是最常见的格式就是 JSON。JSON 是非常容易与之进行接口的数据格式之一，特别是在 JavaScript 中。

在 JavaScript 中，我们不必担心不匹配类的 JSON 对象。如果我们使用的是像 Java（或者正在使用 TypeScript 的人）这样的强类型语言，我们将不得不担心以下事项：

+   创建一个模仿 JSON 对象格式的类。

+   创建一个基于嵌套对象数量的嵌套映射结构。

+   根据我们收到的 JSON 创建即时类。

这些都不一定难，但当我们与使用这些语言编写的系统进行接口时，它可能会增加速度和复杂性。使用其他数据格式时，我们可能会获得一些主要的速度优势；不仅可能会获得更小的数据传输量，而且其他语言也能更容易地解析对象。当我们转向基于模式的数据格式时，甚至会获得更多的好处，比如版本控制，这可以使向后兼容更容易。

考虑到所有这些，让我们继续看一下 JSON，并了解一些利弊，以及我们在使用它时得到的损失。除此之外，我们将看一下一个新的自定义格式，我们将为我们的服务创建一个更小的数据传输格式。之后，我们将看一下无模式数据格式，比如 JSON，最后，我们将看一下基于模式的格式。

这一章可能比其他章节都要轻一些，但在开发企业应用程序或与其进行接口时，这是一个非常有用的章节。

本章涵盖的主题如下：

+   使用 JSON

+   JSON 编码

+   JSON 解码

+   查看数据格式

在 TypeScript 中，如果我们愿意，我们可以只使用`any`类型，但这在某种程度上会削弱 TypeScript 的目的。虽然本书不会涉及 TypeScript，但知道它存在，并且很容易看出开发人员在开发后端应用程序时可能会遇到它。

# 技术要求

完成本章需要以下工具：

+   一个编辑器或 IDE，最好是 VS Code

+   支持 Node.js 的操作系统

+   在[`github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter08`](https://github.com/PacktPublishing/Hands-On-High-Performance-Web-Development-with-JavaScript/tree/master/Chapter08)找到的代码。

# 使用 JSON

如前所述，JSON 提供了一个易于使用和操作的接口，用于在服务之间发送和接收消息。对于不了解的人来说，JSON 代表**JavaScript 对象表示法**，这也是它与 JavaScript 接口得很好的原因之一。它模仿了 JavaScript 对象的许多行为，除了一些基本类型（例如函数）。这也使得它非常容易解析。我们可以使用内置的`JSON.parse`函数将 JSON 的字符串化版本转换为对象，或者使用`JSON.stringify`将我们的对象转换为其在网络上传输的格式。

那么在使用 JSON 时有哪些缺点呢？首先，当通过网络发送数据时，格式可能会变得非常冗长。考虑一个具有以下格式的对象数组：

```js
{
    "name" : "Bob",
    "birth" : "01/02/1993",
    "address" : {
        "zipcode" : 11111,
        "street" : "avenue of av.",
        "streetnumber" : 123,
        "state" : "CA",
        "country" : "US"
    },
    "contact" : {
        "primary" : "111-222-3333",
        "secondary" : "444-555-6666",
        "email" : "bob@example.com"
    }
}
```

对于那些曾经处理过联系表单或客户信息的人来说，这可能是一个常见的情景。现在，虽然我们应该为网站涉及某种分页，但我们仍然可能一次获取`100`甚至`500`个这样的数据。这可能很容易导致巨大的传输成本。我们可以使用以下代码来模拟这种情况：

```js
const send = new Array(100);
send.fill(json);
console.log('size of over the wire buffer is: ',
 Buffer.from(JSON.stringify(send)).byteLength);
```

通过使用这种方法，我们可以得到对于我们发送的`100`条数据进行字符串化后的缓冲区的字节长度。我们将看到它大约为 22 KB 的数据。如果我们将这个数字增加到`500`，我们可以推断出它将大约为 110 KB 的数据。虽然这可能看起来不像是很多数据，但我们可能会看到这种类型的数据被发送到智能手机上，我们希望限制我们传输的数据量，以免耗尽电池。

我们尚未深入讨论手机和我们的应用程序，尤其是在前端，但这是我们需要越来越意识到的事情，因为我们正在变得越来越像一个远程商业世界。许多用户，即使没有应用程序的移动版本，仍然会尝试使用它。一个个人的轶事是利用为桌面应用程序设计的电子邮件服务，因为移动版本的应用程序中缺少一些功能。我们始终需要意识到我们正在传输的数据量，但移动设备已经使这个想法成为主要目标。

解决这个问题的一种方法是利用某种压缩/解压缩格式。一个相当知名的格式是`gzip`。这种格式非常快速，没有数据质量损失（一些压缩格式有这个问题，比如 JPEG），并且在网页中非常普遍。

让我们继续使用 Node.js 中的`zlib`模块来`gzip`这些数据。以下代码展示了`zlib`中一个易于使用的`gzip`方法，并展示了原始版本和 gzip 版本之间的大小差异：

```js
gzipSync(Buffer.from(JSON.stringify(send))).byteLength
```

现在我们将看到经过 gzip 压缩的版本只有 301 字节，对于 500 长度的数组，我们看到大约 645 字节的 gzip 版本。这是相当节省的！然而，这里有几点需要记住。首先，我们在数组中的每个项目中使用完全相同的对象。压缩算法是基于模式的，因此一遍又一遍地看到完全相同的对象给了我们对原始形式到压缩形式的错误感觉。这并不意味着这不是未压缩与压缩数据之间大小差异的指示，但在测试各种格式时需要牢记这一点。根据各种网站，我们将看到原始数据的 4-10 倍的压缩比（这意味着如果原始数据为 1 MB，我们将看到压缩大小从 250 KB 到 100 KB 不等）。

我们可以创建一个自己的格式，以更紧凑的方式表示数据，而不是使用 JSON。首先，我们将只支持三种项目类型：整数、浮点数和字符串。其次，我们将在消息头中存储所有的键。

模式最好可以描述为传入数据的定义。这意味着我们将知道如何解释传入的数据，而不必寻找特殊的编码符号来告诉我们负载的结束（尽管我们的格式将使用一个结束信号）。

我们的模式将看起来像以下内容：

1.  我们将为消息的头部和主体使用包装字节。头部将用`0x10`字节表示，主体将用`0x11`字节表示。

1.  我们将支持以下类型，它们的转换看起来类似于以下内容：

+   整数：`0x01`后跟一个 32 位整数

+   浮点数：`0x02`后跟一个 32 位整数

+   字符串：`0x03`后跟字符串的长度，后跟数据

这应该足够让我们理解数据格式以及它们可能与仅对 JSON 进行编码和解码有所不同的工作方式。在接下来的两个部分中，我们将看到如何使用流来实现编码器和解码器。

# 实现编码器

我们将使用转换流来实现编码器和解码器。这将为我们提供最大的灵活性，因为我们实际上正在实现流，并且它已经具有我们需要的许多行为，因为我们在技术上正在转换数据。首先，我们需要一些通用的辅助方法，用于编码和解码我们特定数据类型的方法，并将所有这些方法放在一个`helpers.js`辅助文件中。编码函数将如下所示：

```js
export const encodeString = function(str) {
    const buf = Buffer.from(str);
    const len = Buffer.alloc(4);
    len.writeUInt32BE(buf.byteLength);
    return Buffer.concat([Buffer.from([0x03]), len, buf]);
}
export const encodeNumber = function(num) {
    const type = Math.round(num) === num ? 0x01 : 0x02;
    const buf = Buffer.alloc(4);
    buf.writeInt32BE(num);
    return Buffer.concat([Buffer.from([type]), buf]); 
}
```

编码字符串接受字符串并输出将保存解码器工作信息的缓冲区。首先，我们将字符串更改为`Buffer`格式。接下来，我们创建一个缓冲区来保存字符串的长度。然后，我们利用`writeUInt32BE`方法存储缓冲区的长度。

对于那些不了解字节/位转换的人来说，8 位信息（位要么是 1 要么是 0-我们可以提供的最低形式的数据）组成 1 个字节。我们要写入的 32 位整数由 4 个字节组成（32/8）。该方法的 U 部分表示它是无符号的。无符号表示我们只想要正数（在我们的情况下长度只能是 0 或正数）。有了这些信息，我们就可以看到为什么我们为这个操作分配了 4 个字节，以及为什么我们要使用这个特定的方法。有关缓冲区的写入/读取部分的更多信息，请访问[`nodejs.org/api/buffer.html`](https://nodejs.org/api/buffer.html)，因为它深入解释了我们可以访问的缓冲区操作。我们只会解释我们将要使用的操作。

一旦我们将字符串转换为缓冲区格式并获得字符串的长度，我们将写出一个缓冲区，其中`type`作为第一个字节，在我们的情况下是`0x03`字节；字符串的长度，这样我们就知道传入缓冲区的字符串有多长；最后，字符串本身。这个方法应该是两个辅助方法中最复杂的一个，但从解码的角度来看，它应该是有意义的。当我们读取缓冲区时，我们不知道字符串的长度。因此，我们需要在此类型的前缀中有一些信息，以知道实际读取多少。在我们的情况下，`0x03`告诉我们类型是字符串，根据我们之前建立的数据类型协议，我们知道接下来的 4 个字节将是字符串的长度。最后，我们可以使用这些信息来在缓冲区中向前读取，以获取字符串并将其解码回字符串。

`encodeNumber`方法更容易理解。首先，我们检查数字的四舍五入是否等于自身。如果是，那么我们知道我们正在处理一个整数，否则，我们将其视为浮点数。对于不了解的人来说，在大多数情况下，在 JavaScript 中知道这些信息并不太重要（尽管 V8 引擎在知道它正在处理整数时会使用某些优化），但如果我们想要将这种数据格式与其他语言一起使用，那么差异就很重要了。

接下来，我们分配了 4 个字节，因为我们只打算写出 32 位有符号整数。有符号意味着它们将支持正数和负数（再次，我们不会深入探讨两者之间的巨大差异，但对于那些好奇的人来说，如果我们使用有符号整数，我们实际上限制了我们可以在其中存储的最大值，因为我们必须利用其中一个位告诉我们这个数字是正数还是负数）。然后，我们写出最终的缓冲区，其中包括我们的类型，然后是缓冲区格式中的数字。

现在，使用`helper.js`文件中的辅助方法和以下常量进行如下操作：

```js
export const CONSTANTS = {
    object : 0x04,
    number : 0x01,
    floating : 0x02,
    string : 0x03,
    header : 0x10,
    body : 0x11
}
```

我们可以创建我们的`encoder.js`文件：

1.  导入必要的依赖项，并创建我们的`SimpleSchemaWriter`类的框架：

```js
import { Transform } from 'stream';
import { encodeString, encodeNumber } from './helper.js';

export default class SimpleSchemaWriter extends Transform {
}
```

1.  创建构造函数，并确保始终打开`objectMode`：

```js
// inside our SimpleSchemaWriter class
constructor(opts={}) {
    opts.writableObjectMode = true;
    super(opts);
}
```

1.  添加一个私有的`#encode`辅助函数，它将为我们进行底层数据检查和转换：

```js
// inside of our SimpleSchemaWriter class
#encode = function(data) {
    return typeof data === 'string' ?
            encodeString(data) :
            typeof data === 'number' ?
            encodeNumber(data) :
            null;
}
```

1.  编写我们`Transform`流的主要`_transform`函数。该流的详细信息将在下文中解释：

```js
_transform(chunk, encoding, callback) {
    const buf = [];
    buf.push(Buffer.from([0x10]));
    for(const key of Object.keys(chunk)) { 
        const item = this.#encode(key);
        if(item === null) {
            return callback(new Error("Unable to parse!"))
        }
        buf.push(item);
    }
    buf.push(Buffer.from([0x10])); 
    buf.push(Buffer.from([0x11]));
    for(const val of Object.values(chunk)) { 
        const item = this.#encode(val);
        if(item === null) {
            return callback(new Error("Unable to parse!"))
        }
        buf.push(item);
    }
    buf.push(Buffer.from([0x11]));
    this.push(Buffer.concat(buf)); 
    callback();
}
```

总的来说，`transform`函数应该与我们之前实现的`_transform`方法很相似，但有一些例外：

1.  我们编码的第一部分是包装我们的标头（对象的键）。这意味着我们需要写出我们的标头分隔符，即`0x10`字节。

1.  我们将遍历对象的所有键。然后，我们将利用`private`方法`encode`。这个方法将检查键的数据类型，并利用我们之前讨论过的辅助方法之一返回编码。如果它得到一个它不理解的类型，它将返回`null`。然后我们将返回一个`Error`，因为我们的数据协议不理解这种类型。

1.  一旦我们遍历完所有的键，我们将再次写出`0x10`字节，表示我们已经完成了标头，并写出`0x11`字节告诉解码器我们要开始消息的主体部分。（我们可以在这里使用`helpers.js`文件中的常量，而且我们可能应该这样做，但这应该有助于理解底层协议。解码器将利用这些常量来展示更好的编程实践。）

1.  现在我们将遍历对象的值，并将它们通过与标头相同的编码系统运行，并在不理解数据类型时返回一个`Error`。

1.  一旦我们完成了主体部分，我们将再次推送`0x11`字节，表示我们已经完成了主体部分。这将是解码器停止转换此对象并发送出它一直在转换的信号。然后我们将所有这些数据推送到我们`Transform`流的`Readable`部分，并使用回调来表示我们已准备好处理更多数据。

我们的编码方案的整体结构存在一些问题（我们不应该使用单个字节作为包装器，因为它们很容易被我们的编码器和解码器误解），我们应该支持更多的数据类型，但这应该对如何为更常用的数据格式构建编码器有一个很好的理解。

现在，我们无法测试这一点，除了它能正确输出编码外，但一旦我们的解码器运行起来，我们就能测试是否两边得到相同的对象。现在让我们来看看这个系统的解码器。

# 实现解码器

解码器的状态比编码器要复杂得多，这通常是数据格式的特点。当处理原始字节时，尝试从中解析信息通常比以原始格式写出数据更困难。

让我们来看看我们将用来解码支持的数据类型的辅助方法：

```js
import { CONSTANTS } from './helper.js';

export const decodeString = function(buf) {
    if(buf[0] !== CONSTANTS.string) {
        return false;
    }
    const len = buf.readUInt32BE(1);
    return buf.slice(5, 5 + len).toString('utf8');
}
export const decodeNumber = function(buf) {
    return buf.readInt32BE(1);
}
```

`decodeString`方法展示了我们如何处理格式不正确的数据的错误，而`decodeNumber`方法则没有展示这一点。对于`decodeString`方法，我们需要从缓冲区中获取字符串的长度，我们知道这是传入的缓冲区的第二个字节。基于此，我们知道可以通过从缓冲区的第五个字节开始（第一个字节告诉我们这是一个字符串；接下来的四个字节是字符串的长度）获取字符串，并且获取直到达到字符串的长度。然后我们通过`toString`方法运行这个缓冲区。

`decodeNumber`非常简单，因为我们只需要读取告诉我们它是一个数字的第一个字节后面的 4 个字节（再次，我们应该在这里进行检查，但我们保持简单）。这展示了我们需要解码支持的数据类型的两个主要辅助方法。接下来，我们将看一下实际的解码器。它将看起来像下面这样。

如前所述，解码过程有点复杂。这是由于许多原因，如下所述：

+   我们直接处理字节，所以我们需要做相当多的处理。

+   我们正在处理头部和主体部分。如果我们创建了一个非基于模式的系统，我们可能可以编写一个解码器，其状态不像这个解码器中那么多。

+   同样，由于我们直接处理缓冲区，所有数据可能不会一次全部到达，因此我们需要处理这种情况。编码器不必担心这一点，因为我们正在以对象模式操作可写流。

考虑到这一点，让我们来看一下解码流程：

1.  我们将使用与以前的`Transform`流相同类型的设置来设置我们的解码流。我们将设置一些私有变量来跟踪我们在解码器中的状态：

```js
import { Transform } from 'stream'
import { decodeString, decodeNumber, CONSTANTS } from './helper.js'

export default class SimpleSchemaReader extends Transform {
    #obj = {}
    #inHeaders = false
    #inBody = false
    #keys = []
    #currKey = 0
}
```

1.  接下来，我们将在解码过程中使用一个索引。我们不能简单地一次读取一个字节，因为解码过程以不同的速度运行（当我们读取一个数字时，我们要读取 5 个字节；当我们读取一个字符串时，至少要读取 6 个字节）。因此，使用`while`循环会更好：

```js
#decode = function(chunk, index, type='headers') { 
        const item = chunk[index] === CONSTANTS.string ?
            decodeString(chunk.slice(index)) :
            decodeNumber(chunk.slice(index, index + 5));

        if( type === 'headers' ) {
            this.#obj[item] = null;
        } else {
            this.#obj[this.#keys[this.#currKey]] = item;
        }
        return chunk[index] === CONSTANTS.string ?
            index + item.length + 5 :
            index + 5;
    }
    constructor(opts={}) {
        opts.readableObjectMode = true;
        super(opts);
    }
    _transform(chunk, encoding, callback) {
        let index = 0; //1
        while(index <= chunk.byteLength ) {
        }
    }
```

1.  现在，我们要检查当前字节，看它是头部还是主体的分隔标记。这将让我们知道我们是在处理对象键还是对象值。如果我们检测到`headers`标志，我们将设置`#inHeaders`布尔值，表示我们在头部。如果我们在主体中，我们还有更多工作要做：

```js
// in the while loop
const byte = chunk[index];
if( byte === CONSTANTS.header ) { 
    this.#inHeaders = !this.#inHeaders
    index += 1;
    continue;
} else if( byte === CONSTANTS.body ) { 
    this.#inBody = !this.#inBody
    if(!this.#inBody ) { 
        this.push(this.#obj);
        this.#obj = {};
        this.#keys = [];
        this.#currKey = 0;
        return callback();
    } else {
        this.#keys = Object.keys(this.#obj); 
    }
    index += 1;
    continue;
}
if( this.#inHeaders ) { 
    index = this.#decode(chunk, index);
} else if( this.#inBody ) {
    index = this.#decode(chunk, index, 'body');
    this.#currKey += 1;
} else {
    callback(new Error("Unknown state!"));
}
```

1.  接下来，接下来的段落将解释获取每个 JSON 对象的头部和值的过程。

首先，我们将把我们的主体布尔值更改为当前状态的相反值。接下来，如果我们从主体内部到主体外部，这意味着我们已经完成了这个对象。因此，我们可以推出我们当前正在处理的对象，并重置所有内部状态变量（临时对象`#obj`，我们从头部获取的临时`#keys`集合，以及`#currKey`，用于在主体中工作时知道我们正在处理哪个键）。一旦我们完成这些操作，我们就可以运行回调（我们在这里返回，所以我们不会运行更多的主体）。如果我们不这样做，我们将继续循环，并处于一个糟糕的状态。

否则，我们已经浏览了有效负载的头部，并已经到达了每个对象的值。我们将把我们的私有`#keys`变量设置为对象的键（因为在这一点上，头部应该已经从头部获取了所有的键）。我们现在可以开始看到解码过程。

如果我们在头部，我们将运行我们的私有`#decode`方法，并且不使用第三个参数，因为默认情况下是以头部运行该方法。否则，我们将像在主体中一样运行它，并传递第三个参数以说明我们在主体中。此外，如果我们在主体中，我们将增加我们的`#currKey`变量。

最后，我们可以看一下解码过程的核心，`#decode`方法。我们根据缓冲区中的第一个字节获取项目，这将告诉我们应该运行哪个解码辅助方法。然后，如果我们在头部模式下运行此方法，我们将为我们的临时对象设置一个新键，并将其值设置为 null，因为一旦我们到达主体，它将被填充。如果我们在主体模式下，我们将设置与我们正在循环的`#keys`数组中的`#currKey`索引对应的键的值，一旦我们进入主体，我们就会开始循环。

有了这个代码解释，正在发生的基本过程可以总结为几个基本步骤：

1.  我们需要浏览头部并将对象的键设置为这些值。我们暂时将这些键的值设置为 null，因为它们将在以后填充。

1.  一旦我们离开头部部分并进入主体部分，我们可以从临时对象中获取所有键，并且我们在那时进行的解码运行应该对应于数组中当前键索引处的键。

1.  一旦我们离开主体部分，我们将重置所有临时变量的状态，并发送相应的对象，因为我们已经完成了解码过程。

这可能看起来令人困惑，但我们所做的就是将头部与相同索引处的主体元素对齐。如果我们想要将键和值的数组放在一起，这将类似于以下代码：

```js
const keys = ['item1', 'item2', 'item3'];
const values = [1, 'what', 2.2];
const tempObj = {};
for(let i = 0; i < keys.length; i++) {
    tempObj[keys[i]] = null;
}
for(let i = 0; i < values.length; i++) {
    tempObj[keys[i]] = values[i];
}
```

这段代码几乎与之前的缓冲区完全相同，只是我们必须使用原始字节而不是更高级的项目，如字符串、数组和对象。

解码器和编码器都完成后，我们现在可以通过我们的编码器和解码器运行一个对象，看看我们是否得到相同的值。让我们运行以下测试代码：

```js
import encoder from './encoder.js'
import decoder from './decoder.js'
import json from './test.json'

const enc = new encoder();
const dec = new decoder();
enc.pipe(dec);
dec.on('data', (obj) => {
    console.log(obj);
});
enc.write(json);
```

我们将使用以下测试对象：

```js
{
    "item1" : "item",
    "item2" : 12,
    "item3" : 3.3
}
```

我们将看到，当我们将数据通过编码器传输到解码器时，我们将得到相同的对象。现在，我们已经创建了自己的编码和解码方案，但它与 JSON 相比在传输大小上如何？使用这个负载，我们实际上增加了大小！如果我们考虑一下，这是有道理的。我们必须添加所有特殊的编码项（除了数据之外的所有信息，如`0x10`和`0x11`字节），但现在我们开始向我们的列表中添加更多的大型数字项。我们将看到，我们开始击败基本的`JSON.stringify`和`JSON.parse`。

```js
{
    "item1" : "item",
    "item2" : 120000000,
    "item3" : 3.3,
    "item4" : 120000000,
    "item5" : 120000000,
    "item6" : 120000000
}
```

这是因为字符串化的数字被转换成了字符串版本的数字，所以当我们得到大于 5 个字节的数字时，我们开始节省字节（1 个字节用于数据类型，4 个字节用于 32 位数字编码）。对于字符串，我们永远不会节省，因为我们总是添加额外的 5 个字节的信息（1 个字节用于数据类型，4 个字节用于字符串的长度）。

在大多数编码和解码方案中，情况都是如此。它们处理数据的方式取决于传递的数据类型。在我们的情况下，如果我们通过网络发送大量的高度数值化的数据，我们的方案可能效果更好，但如果我们传输字符串，我们将无法从这种编码和解码方案中获益。在我们看一些在野外广泛使用的数据格式时，请记住这一点。

记住，这种编码和解码方案并不是用于实际环境的，因为它充满了问题。然而，它展示了构建数据格式的基本主题。虽然大多数人永远不需要构建数据格式，但了解构建数据格式时发生的情况以及数据格式可能需要根据其主要处理的数据类型专门化其编码和解码方案是很好的。

# 数据格式的一瞥

现在我们已经看过了我们自己的数据格式，让我们继续看看一些目前流行的数据格式。这不是对这些数据格式的详尽了解，而是对数据格式和我们可能在野外发现的内容的介绍。

我们将要查看的第一种数据格式是无模式格式。如前所述，基于模式的格式要么提前发送数据的模式，要么将模式与数据本身一起发送。这通常允许数据以更紧凑的形式传入，同时确保双方同意数据接收方式。另一种形式是无模式，我们通过规范发送数据的新形式，但解码所有信息都是通过规范完成的。

JSON 就是其中一种格式。当我们发送 JSON 时，我们必须对其进行编码，然后在另一端对其进行解码。另一种无模式数据格式是 XML。这两种格式对于 Web 开发人员来说应该非常熟悉，因为我们广泛使用 JSON，并且在组装前端（HTML）时使用一种 XML 形式。

另一种流行的格式是`MessagePack`（[`msgpack.org/index.html`](https://msgpack.org/index.html)）。`MessagePack`是一种以比 JSON 更小的有效载荷而闻名的格式。`MessagePack`的另一个优点是有许多语言为其编写了原生库。我们将看一下 Node.js 版本，但请注意，这可以在前端（浏览器）和服务器上都可以使用。所以让我们开始吧：

1.  我们将使用以下命令通过`npm install`安装`what-the-pack`扩展。

```js
> npm install what-the-pack
```

1.  完成后，我们可以开始使用这个库。通过以下代码，我们可以看到在网络上传输这种数据格式是多么容易。

```js
import MessagePack from 'what-the-pack';
import json from '../schema/test.json';

const { encode, decode } = MessagePack.initialize(2**22);
const encoded = encode(json);
const decoded = decode(encoded);
console.log(encoded.byteLength, Buffer.from(JSON.stringify(decoded)).byteLength);
console.log(encoded, decoded);
```

我们在这里看到的是对`what-the-pack`页面上示例的略微修改版本（[`www.npmjs.com/package/what-the-pack`](https://www.npmjs.com/package/what-the-pack)）。我们导入了该包，然后初始化了该库。该库的一个不同之处在于，我们需要为编码和解码过程初始化一个缓冲区。这就是`initialize`方法中的`2**22`所做的。我们正在初始化一个大小为 2 的 22 次方字节的缓冲区。这样，它可以轻松地切割缓冲区并复制它，而不需要昂贵的基于数组的操作。敏锐的观察者还会注意到的另一件事是，该库不是基于流的。他们很可能这样做是为了在浏览器和 Node.js 之间保持兼容。除了这些小问题，整个库的工作方式与我们想象的一样。

第一个控制台日志向我们展示了编码后的缓冲区比 JSON 版本少了 5 个字节。虽然这确实表明该库给我们提供了更紧凑的形式，但应该注意到，有些情况下`MessagePack`可能不比相应的 JSON 更小。它也可能比内置的`JSON.stringify`和`JSON.parse`方法运行得更慢。记住，一切都是一种权衡。

有很多无模式数据格式，每种格式都有自己的技巧，试图使编码/解码时间更快，使过程中的数据更小。然而，当我们处理企业系统时，我们很可能会看到使用基于模式的数据格式。

有几种定义模式的方法，但在我们的情况下，我们将使用 proto 文件格式。

1.  让我们继续创建一个**proto**文件，以模拟我们之前的`test.json`文件。模式可能看起来像以下内容：

```js
package exampleProtobuf;
syntax = "proto3";

message TestData {
    string item1 = 1;
    int32  item2 = 2;
    float  item3 = 3;
}
```

我们在这里声明的是，这条名为`TestData`的消息将存储在名为`exampleProtobuf`的包中。该包主要用于将类似的项目分组（这在诸如 Java 和 C#等语言中被广泛利用）。语法告诉我们的编码器和解码器，我们将使用的协议是`proto3`。协议还有其他版本，而这个版本是最新的稳定版本。

然后，我们声明一个名为`TestData`的新消息，其中包含三个条目。一个将被称为`item1`，类型为`string`，一个将是称为`item2`的整数，最后一个将是称为`item3`的浮点数。我们还为它们分配了 ID，因为这样可以更容易进行索引和自引用类型（也因为这对于`protobuf`来说是强制性的）。我们不会详细介绍这样做的具体作用，但请注意它可以帮助编码和解码过程。

1.  接下来，我们可以编写一些代码，可以使用它在我们的代码中创建一个`TestData`对象，可以专门处理这些消息。这将看起来像下面这样：

```js
protobuf.load('test.proto', function(err, root) {
    if( err ) throw err;
    const TestTypeProto = 
     root.lookupType("exampleProtobuf.TestData");
    if( TestTypeProto.verify(json) ) {
        throw Error("Invalid type!");
    }
    const message2 = TestTypeProto.create(json);
    const buf2 = TestTypeProto.encode(message2).finish();
    const final2 = TestTypeProto.decode(buf2);
    console.log(buf2.byteLength, 
     Buffer.from(JSON.stringify(final2)).byteLength);
    console.log(buf2, final2);
});
```

请注意，这与我们之前看到的大多数代码类似，除了一些验证和创建过程。首先，库需要读取我们拥有的原型文件，并确保它确实是正确的。接下来，我们根据我们给它的命名空间和名称创建对象。现在，我们验证我们的有效负载并从中创建消息。然后，我们通过特定于此数据类型的编码器运行它。最后，我们解码消息并测试以确保我们得到了与输入相同的数据。

从这个例子中应该注意到两件事。首先，数据大小非常小！这是基于模式/protobuf 的优势之一，超过了无模式数据格式。由于我们提前知道类型应该是什么，我们不需要将该信息编码到消息本身中。其次，我们将看到浮点数并没有返回为 3.3。这是由于精度错误，这是我们应该警惕的事情。

1.  现在，如果我们不想像这样读取原型文件，我们可以在代码中构建消息，就像下面这样：

```js
const TestType = new protobuf.Type("TestType");
TestType.add(new protobuf.Field("item1", 1, "string"));
TestType.add(new protobuf.Field("item2", 2, "int32"));
TestType.add(new protobuf.Field("item3", 3, "float"));
```

这应该类似于我们在原型文件中创建的消息，但我们将逐行查看以显示它与`protobuf`对象相同。在这种情况下，我们首先创建一个名为`TestType`的新类型（而不是`TestData`）。接下来，我们添加三个字段，每个字段都有自己的标签、索引号和存储在其中的数据类型。如果我们通过相同类型的验证、创建、编码、解码过程运行它，我们将得到与之前相同的结果。

虽然这并不是对不同数据格式的全面概述，但它应该有助于识别何时使用无模式（当我们不知道数据可能是什么样子时）以及何时使用模式（当在未知系统之间通信或我们需要减少有效负载大小时）。

# 总结

虽然我们大多数起始应用程序将使用 JSON 在不同服务器之间传递数据，甚至在我们应用程序的不同部分之间传递数据，但应该注意到我们可能不想使用它的地方。通过利用其他数据格式，我们可以确保尽可能地提高应用程序的速度。

我们已经看到了构建自己的数据格式可能涉及的内容，然后我们看了一下当前流行的其他格式。这应该是我们构建高性能 Node.js 服务器应用程序所需的最后一部分信息。虽然我们将使用一些数据格式的库，但我们也应该注意到，我们实际上只使用了 Node.js 自带的原始库。

接下来，我们将看一个实际的静态服务器示例，该服务器缓存信息。从这里开始，我们将利用之前的所有概念来创建一个高可用和高速的静态服务器。
