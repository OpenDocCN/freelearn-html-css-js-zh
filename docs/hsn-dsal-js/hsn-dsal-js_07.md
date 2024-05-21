# 第七章：排序及其应用

排序是我们用来重新排列一组数字或对象以升序或降序排列的非常常见的算法。排序的更技术性的定义如下：

在计算机科学中，排序算法是一种将列表中的元素按照特定顺序排列的算法。

现在，假设您有一个包含*n*个项目的列表，并且您想对它们进行排序。您取出所有*n*个项目，并确定您可以将这些项目放置在所有可能的序列中，这种情况下总共有`n!`种可能。我们现在需要确定这些`n!`序列中哪些没有任何倒置对，以找出排序后的列表。倒置对被定义为列表中位置由`i，j`表示的一对元素，其中`i < j`，但值`x[i] > x[j]`。

当然，上述方法是繁琐的，需要一些繁重的计算。在本章中，我们将讨论以下主题：

+   排序算法的类型

+   为图书管理系统（如图书馆）创建 API

+   插入排序算法用于对书籍数据进行排序

+   归并排序算法用于对书籍数据进行排序

+   快速排序算法用于对书籍数据进行排序

+   不同排序算法的性能比较

让我们看一下上面列出的一些更优化的排序类型，可以在各种场景中使用。

# 排序算法的类型

我们都知道有不同类型的排序算法，大多数人在编程生涯中的某个时候都听说过这些不同类型的算法的名称。排序算法和数据结构之间的主要区别在于，无论使用哪种类型的算法，前者总是有相同的目标。这使得我们非常容易和重要地在各个方面比较不同的排序算法，大多数情况下都归结为速度和内存使用。在选择特定的排序算法之前，我们需要在手头的数据类型基础上做出这一决定。

考虑到以上情况，我们将比较和对比以下三种不同类型的算法：

+   插入排序

+   归并排序

+   快速排序

归并排序和快速排序是 v8 引擎在内部用于对数据进行排序的算法；当数据集大小太小（<10）时，使用归并排序，否则使用快速排序。另一方面，插入排序是一种更简单的算法。

然而，在我们深入讨论每种排序算法的实现之前，让我们快速看一下用例，然后设置相同的先决条件。

# 不同排序算法的用例

为了测试不同的排序算法，我们将创建一个小型的 express 服务器，其中将包含一个端点，用于获取按每本书的页数排序的所有书籍列表。在这个例子中，我们将从一个 JSON 文件开始，其中包含一个无序的书籍列表，这将作为我们的数据存储。

在生产应用中，排序应该推迟到数据库查询，并且不应作为应用逻辑的一部分来完成，以避免在处理筛选和分页请求等场景时出现痛苦和混乱。

# 创建一个 Express 服务器

我们设置项目的第一步是创建一个目录，我们想要在其中编写我们的应用程序；为此，在终端中运行以下命令：

```js
mkdir sorting
```

创建后，通过运行`cd`进入目录，然后运行 npm 初始化命令将其设置为 Node.js 项目：

```js
cd sorting
npm init
```

这将询问您一系列问题，您可以回答或留空以获得默认答案，两者都可以。项目初始化后，添加以下 npm 包，如前几章所做，以帮助我们设置 express 服务器：

```js
npm install express --save
```

添加后，我们现在准备创建我们的服务器。在项目的根目录中添加以下代码到一个新文件中，并将其命名为`index.js`：

```js
var express = require('express');
var app = express();

app.get('/', function (req, res) {
   res.status(200).send('OK!')
});

app.listen(3000, function () {
   console.log('Chat Application listening on port 3000!')
});
```

我们已经设置了一个返回`OK`的单个端点，并且我们的服务器正在端口`3000`上运行。让我们还在`package.json`文件的脚本中添加一个快捷方式来轻松启动应用程序：

```js
...
"scripts": {
  "start": "node index.js",
  "test": "echo \"Error: no test specified\" && exit 1" },
...
```

现在，要测试这些更改，请从根文件夹运行`npm start`，并在浏览器中打开`localhost:3000`。您应该在屏幕上看到一个`OK!`消息，如我们在`index.js`文件中定义的那样。

# 模拟图书馆书籍数据

现在，让我们创建我们的图书馆书籍的模拟数据，当用户请求书籍列表时，我们希望对其进行排序和返回。在本章中，我们将专注于按每本书的页数对图书馆书籍进行排序，因此我们只能简单地添加页数和书籍的 ID，如下面的代码所示：

```js
[
{"id":"dfa6cccd-d78b-4ea0-b447-abe7d6440180","pages":1133},
{"id":"0a2b0a9e-5b3d-4072-ad23-92afcc335c11","pages":708},
{"id":"e1a58d73-3bd2-4a3a-9f29-6cfb9f7a0007","pages":726},
{"id":"5edf9d36-9b5d-4d1f-9a5a-837ad9b73fe9","pages":1731},
...
]
```

我们想测试每个算法的性能，因此让我们添加 5000 本书，以确保我们有足够的数据来测试性能。此外，我们将在 300 到 2000 页之间随机添加这些页数，由于我们总共有 5000 本书，因此在不同的书籍中将会有明显的页数重复。

以下是一个示例脚本，您可以使用它来生成这些数据，如果您想使用此脚本，请确保安装了`uuid` npm 模块：

```js
npm install uuid --save
```

还要在项目的根目录创建一个名为`generator.js`的文件，并添加以下代码：

```js
const fs = require('fs');
const uuid = require('uuid');
const books = [];

for(var i = 0; i < 5000; i++) {
   books.push({
      "id": uuid.v4(),
      "pages": Math.floor(Math.random() * (2000 - 300 + 1) + 300)
   })
}

fs.writeFile('books.json', JSON.stringify(books), (err) => {});
```

现在，要运行它，请从根目录运行`node generator.js`命令，这将生成与前面代码中显示的记录类似的数据的`books.json`文件。

# 插入排序 API

现在，让我们创建一个端点，使用插入排序来根据页面计数对数据进行排序和返回。

# 什么是插入排序

插入排序，顾名思义，是一种排序类型，我们从输入数据集中逐个提取元素，然后确定元素应该放置的位置后，将它们插入到排序好的结果数据集中。

我们可以立即确定这种方法将需要额外的集合（与输入相同大小）来保存结果。因此，如果我们有一个包含 10 个元素的`Set`作为输入，我们将需要另一个大小也为 10 的`Set`作为输出。我们可以稍微改变这种方法，使我们的排序在内存中进行。在内存中执行操作意味着我们不会请求更多的内存（通过创建与输入相同大小的额外集合）。

# 伪代码

让我们快速勾画一下插入排序的伪代码：

```js
LOOP over all data excluding first entry (i = 1)

    INITIALIZE variable j = i - 1

    COPY data at index i

    WHILE all previous values are less than current

        COPY previous value to next

        DECREMENT j

    ADD current data to new position

RETURN sorted data
```

# 实现插入排序 API

根据前面描述的伪代码，实现插入排序非常容易。让我们首先创建一个名为`sort`的文件夹，然后创建一个名为`insertion.js`的文件，在其中我们将添加我们的插入类，如下面的代码所示：

```js
class Insertion {

   sort(data) {
      // loop over all the entries excluding the first record
  for (var i = 1; i< data.length; ++i) {

         // take each entry
  var current = data[i];

         // previous entry
  var j = i-1;

         // until beginning or until previous data is lesser than
         current
  while (j >= 0 && data[j].pages < current.pages) {

            // shift entries to right
  data[j + 1] = data[j];

            // decrement position for next iteration
  j = j - 1;
         }

         // push current data to new position
  data[j+1] = current;
      }

      // return all sorted data
  return data;
   }
}

module.exports = Insertion;
```

如伪代码和实际实现中所讨论的，我们将取每个值并将其与之前的值进行比较，当您有 5000 个随机顺序的项目时，这听起来并不是一件好事；这是真的，插入排序只有在数据集几乎排序并且整个数据集中有一些倒置对时才是首选。

改进此功能的一种方法是改变我们确定要在排序列表中插入的位置的方式。我们可以不再将其与所有先前的值进行比较，而是执行二进制搜索来确定数据应该移动到排序列表中的位置。因此，通过稍微修改前面的代码，我们得到以下结果：

```js
class Insertion {

   sort(data) {
      // loop over all the entries
  for (var i = 1; i < data.length; ++i) {

         // take each entry
  var current = data[i];

         // previous entry
  var j = i - 1;

         // find location where selected sould be inseretd
  var index = this.binarySearch(data, current, 0, j);

         // shift all elements until new position
  while (j >= index) {
            // shift entries to right
  data[j + 1] = data[j];

            // decrement position for next iteration
  j = j - 1;
         }

         // push current data to new position
  data[j + 1] = current;
      }

      // return all sorted data
  return data;
   }

   binarySearch(data, current, lowPos, highPos) {
      // get middle position
  var midPos = Math.floor((lowPos + highPos) / 2);

      // if high < low return low position;
 // happens at the beginning of the data set  if (highPos <= lowPos) {

         // invert condition to reverse sorting
  return (current.pages < data[lowPos].pages) ? (lowPos + 1):
         lowPos;
      }

      // if equal, give next available position
  if(current.pages === data[midPos].pages) {
         return midPos + 1;
      }

      // if current page count is less than mid position page count,
 // reevaluate for left half of selected range // invert condition and exchange return statements to reverse
      sorting  if(current.pages > data[midPos].pages) {
         return this.binarySearch(data, current, lowPos, midPos - 1);
      }

      // evaluate for right half of selected range
  return this.binarySearch(data, current, midPos + 1, highPos);
   }
}

module.exports = Insertion;
```

一旦实现，我们现在需要定义在我们的数据集上使用此排序的路由。为此，首先我们将导入之前创建的 JSON 数据，然后在我们的端点中使用它，我们专门创建它来使用插入排序对数据进行排序：

```js
var express = require('express');
var app = express();
var data = require('./books.json');
var Insertion = require('./sort/insertion');

app.get('/', function (req, res) {
   res.status(200).send('OK!')
});

app.get('/insertion', function (req, res) {
 res.status(200).send(new Insertion().sort(data));
});

app.listen(3000, function () {
   console.log('Chat Application listening on port 3000!')
});
```

现在，我们可以重新启动服务器，并尝试在浏览器或 postman 中访问`localhost:3000/insertion`端点，如下截图所示，以查看包含排序数据的响应：

![](img/37231e17-b848-4451-b0a8-15d945bda797.png)

# 归并排序 API

现在，让我们创建一个端点，使用 Mergesort 对基于页面计数的数据进行排序并返回。

# 什么是 Mergesort

Mergesort 是一种分而治之的排序算法，首先将整个数据集划分为每个元素一个子集，然后重复地将这些子集连接和排序，直到得到一个排序好的集合。

这个算法同时使用了递归和分而治之的方法。让我们来看看这种实现的伪代码。

# 伪代码

根据我们迄今为止对 mergesort 的了解，我们可以得出实现的伪代码，如下所示：

```js
MERGE_SORT(array)
    INITIALIZE middle, left_half, right_half

    RETURN MERGE(MERGE_SORT(left_half), MERGE_SORT(right_half))

MERGE(left, right)

    INITIALIZE response

    WHILE left and right exist 

        IF left[0] < right[0]

            INSERT left[0] in result

        ELSE

            INSERT right[0] in result

    RETURN result concatenated with remainder of left and right
```

请注意，在前面的代码中，我们首先递归地将输入数据集划分，然后对数据集进行排序和合并。现在，让我们实现这个排序算法。

# 实现 Mergesort API

现在，让我们创建我们的 Mergesort 类，以及之前创建的 Insertionsort 类，并将其命名为`merge.js`：

```js
class Merge {

   sort(data) {
      // when divided to single elements
  if(data.length === 1) {
         return data;
      }

      // get middle index
  const middle = Math.floor(data.length / 2);

      // left half
  const left = data.slice(0, middle);

      // right half
  const right = data.slice(middle);

      // sort and merge
  return this.merge(this.sort(left), this.sort(right));
   }

   merge(left, right) {
      // initialize result
  const result = [];

      // while data
  while(left.length && right.length) {

         // sort and add to result
 // change to invert sorting  if(left[0].pages > right[0].pages) {
            result.push(left.shift());
         } else {
            result.push(right.shift());
         }
      }

      // concat remaining elements with result
  return result.concat(left, right);
   }
}

module.exports = Merge;
```

一旦我们有了这个类，我们现在可以添加一个新的端点来使用这个类：

```js
var express = require('express');
var app = express();
var data = require('./books.json');
var Insertion = require('./sort/insertion');
var Merge = require('./sort/merge');

app.get('/', function (req, res) {
   res.status(200).send('OK!')
});

app.get('/insertion', function (req, res) {
   res.status(200).send(new Insertion().sort(data));
});

app.get('/merge', function (req, res) {
 res.status(200).send(new Merge().sort(data));
});

app.listen(3000, function () {
   console.log('Chat Application listening on port 3000!')
});
```

现在重新启动服务器并测试所做的更改：

![](img/7933b749-866e-43ec-bdde-05cfb564cf67.png)

# Quicksort API

与 Mergesort 类似，Quicksort 也是一种分而治之的算法。在本节中，我们将创建一个端点，使用这个算法对数据集进行排序并返回。

# 什么是 Quicksort

Quicksort 根据预先选择的枢轴值将集合分为两个较小的低值和高值子集，然后递归地对这些较小的子集进行排序。

选择枢轴值可以通过几种方式完成，这是算法中最重要的方面。一种方法是简单地从集合中选择第一个、最后一个或中间值。然后，还有自定义的分区方案，如 Lomuto 或 Hoare（我们将在本章后面使用），可以用来实现相同的效果。我们将在本节中探讨其中一些实现。

让我们来看看这个实现的伪代码。

# 伪代码

根据我们迄今为止讨论的内容，quicksort 的伪代码非常明显：

```js
QUICKSORT(Set, lo, high)

    GET pivot

    GENERATE Left, Right partitions

    QUICKSORT(SET, lo, Left - 1)

    QUICKSORT(SET, Right + 1, high)
```

正如您在前面的代码中所注意到的，一旦我们抽象出获取枢轴的逻辑，算法就不是很复杂。

# 实现 Quicksort API

首先，让我们创建 Quicksort 类，它将根据传递的集合中的第一个元素作为枢轴来对元素进行排序。让我们在`sort`文件夹下创建一个名为`quick.js`的文件：

```js
class Quick {

   simpleSort(data) {

      // if only one element exists
  if(data.length < 2) {
         return data;
      }

      // first data point is the pivot
  const pivot = data[0];

      // initialize low and high values
  const low = [];
      const high = [];

      // compare against pivot and add to
 // low or high values  for(var i = 1; i < data.length; i++) {

         // interchange condition to reverse sorting
  if(data[i].pages > pivot.pages) {
            low.push(data[i]);
         } else {
            high.push(data[i]);
         }
      }

      // recursively sort and concat the
 // low values, pivot and high values  return this.simpleSort(low)
         .concat(pivot, this.simpleSort(high));
   }

}

module.exports = Quick;
```

这很直接了当，现在，让我们快速添加一个端点来访问这个算法，对我们的书进行排序并将它们返回给请求的用户：

```js
var express = require('express');
var app = express();
var data = require('./books.json');
var Insertion = require('./sort/insertion');
var Merge = require('./sort/merge');
var Quick = require('./sort/quick');

....

app.get('/quick', function (req, res) {
 res.status(200).send(new Quick().simpleSort(data));
});

app.listen(3000, function () {
   console.log('Chat Application listening on port 3000!')
});
```

此外，现在重新启动服务器以访问新创建的端点。我们可以看到这里的方法并不理想，因为它需要额外的内存来包含低值和高值，与枢轴相比。

因此，我们可以使用之前讨论过的 Lomuto 或 Hoare 分区方案来在内存中执行此操作，并减少内存成本。

# Lomuto 分区方案

Lomuto 分区方案与我们之前实现的简单排序函数非常相似。不同之处在于，一旦我们选择最后一个元素作为枢轴，我们需要通过在内存中对元素进行排序和交换来不断调整其位置，如下面的代码所示：

```js
partitionLomuto(data, low, high) {

   // Take pivot as the high value
  var pivot = high;

   // initialize loop pointer variable
  var i = low;

   // loop over all values except the last (pivot)
  for(var j = low; j < high - 1; j++) {

      // if value greater than pivot
  if (data[j].pages >= data[pivot].pages) {

         // swap data
  this.swap(data, i , j);

         // increment pointer
  i++;
      }
   }

   // final swap to place pivot at correct
 // position by swapping  this.swap(data, i, j);

   // return pivot position
  return i;
}
```

例如，让我们考虑以下数据：

```js
[{pages: 20}, {pages: 10}, {pages: 1}, {pages: 5}, {pages: 3}]
```

当我们使用这个数据集调用我们的 partition 时，我们的枢轴首先是最后一个元素`3`（表示`pages: 3`），低值为 0（所以是我们的指针），高值为 4（最后一个元素的索引）。

现在，在第一次迭代中，我们看到第`j`个元素的值大于枢轴，所以我们将第`j`个值与低当前指针位置交换；由于它们两者相同，交换时什么也不会发生，但我们会增加指针。因此，数据集保持不变：

```js
20, 10, 1, 5, 3
pointer: 1
```

在下一次迭代中，同样的事情发生了：

```js
20, 10, 1, 5, 3
pointer: 2
```

在第三次迭代中，值较小，所以什么也不会发生，循环继续：

```js
20, 10, 1, 5, 3
pointer: 2
```

在第四次迭代中，值（`5`）大于枢轴值，所以值交换并且指针增加：

```js
20, 10, 5, 1, 3
pointer: 3
```

现在，控制权从`for`循环中退出，我们最终通过最后一次交换将数据放在正确的位置，得到以下结果：

```js
20, 10, 5, 3, 1 
```

之后，我们可以返回指针的位置，这只是枢轴的新位置。在这个例子中，数据在第一次迭代中就已经排序，但可能会有情况，也会有情况，其中不是这样，因此我们递归地重复这个过程，对枢轴位置左右的子集进行排序。

# 霍尔分区方案

另一方面，霍尔分区方案从数据集的中间获取一个枢轴值，然后开始解析从低端和高端确定枢轴的实际位置；与 Lomuto 方案相比，这会导致更少的操作次数：

```js
partitionHoare(data, low, high) {
   // determine mid point
  var pivot = Math.floor((low + high) / 2 );

   // while both ends do not converge
  while(low <= high) {

      // increment low index until condition matches
  while(data[low].pages > data[pivot].pages) {
         low++;
      }

      // decrement high index until condition matches
  while(data[high] && (data[high].pages < data[pivot].pages)) {
         high--;
      }

      // if not converged, swap and increment/decrement indices
  if (low <= high) {
         this.swap(data, low, high);
         low++;
         high--;
      }
   }

   // return the smaller value
  return low;
}
```

现在，我们可以将所有这些放入我们的`Quick`类中，并更新我们的 API 以使用新创建的方法，如下面的代码所示：

```js
class Quick {

   simpleSort(data) {
        ...
   }

   // sort class, default the values of high, low and sort
   sort(data, low = 0, high = data.length - 1, sort = 'hoare') {
      // get the pivot   var pivot =  (sort === 'hoare') ? this.partitionHoare(data, low,
      high)
                  : this.partitionLomuto(data, low, high);

      // sort values lesser than pivot position recursively
  if(low < pivot - 1) {
         this.sort(data, low, pivot - 1);
      }

      // sort values greater than pivot position recursively
  if(high > pivot) {
         this.sort(data, pivot, high);
      }

      // return sorted data
  return data;
   }

   // Hoare Partition Scheme
  partitionHoare(data, low, high) {
        ...
   }

   // Lomuto Partition Scheme
  partitionLomuto(data, low, high) {
        ...
   }

   // swap data at two indices
  swap(data, i, j) {
      var temp = data[i];
      data[i] = data[j];
      data[j] = temp;
   }

}

module.exports = Quick;
```

当我们更新 API 调用签名时，在我们的`index.js`文件中得到以下结果：

```js
app.get('/quick', function (req, res) {
 res.status(200).send(new Quick().sort(data));
});
```

重新启动服务器并访问端点后，我们会得到以下结果：

![](img/19f51739-0563-46d2-bbe3-db4a92dce4ce.png)

从前面的截图中可以看出，对于给定的数据集，快速排序比归并排序稍微快一些。

# 性能比较

现在我们列出并实现了一些排序算法，让我们快速看一下它们的性能。在我们实现这些算法时，我们简要讨论了一些性能增强；我们将尝试量化这种性能增强。

为此，我们将首先安装名为`benchmark`的节点模块，以创建我们的测试套件：

```js
npm install benchmark --save
```

安装了基准框架后，我们可以将我们的测试添加到项目根目录下的名为`benchmark.js`的文件中，该文件将运行前面部分描述的不同排序算法：

```js
var Benchmark = require('benchmark');
var suite = new Benchmark.Suite();
var Insertion = require('./sort/insertion');
var Merge = require('./sort/merge');
var Quick = require('./sort/quick');
var data = require('./books.json');

suite
  .add('Binary Insertionsort', function(){
      new Insertion().sort(data);
   })
   .add('Mergesort', function(){
      new Merge().sort(data);
   })
   .add('Quicksort -> Simple', function(){
      new Quick().simpleSort(data);
   })
   .add('Quicksort -> Lomuto', function(){
      new Quick().sort(data, undefined, undefined, 'lomuto');
   })
   .add('Quicksort -> Hoare', function(){
      new Quick().sort(data);
   })
   .on('cycle', function(e) {
      console.log(`${e.target}`);
   })
   .on('complete', function() {
      console.log(`Fastest is ${this.filter('fastest').map('name')}`);
   })
   .run({ 'async': true });
```

现在，让我们更新`package.json`文件的脚本标签以更新和运行测试：

```js
...

"scripts": {
  "start": "node index.js",
  "test": "node benchmark.js" },

...
```

要查看更改，请从项目的根目录运行`npm run test`命令，我们将在终端中看到类似的东西：

```js
Binary Insertionsort x 1,366 ops/sec ±1.54% (81 runs sampled)
Mergesort x 199 ops/sec ±1.34% (78 runs sampled)
Quicksort -> Simple x 2.33 ops/sec ±7.88% (10 runs sampled)
Quicksort -> Lomuto x 2,685 ops/sec ±0.66% (86 runs sampled)
Quicksort -> Hoare x 2,932 ops/sec ±0.67% (88 runs sampled)
Fastest is Quicksort -> Hoare
```

# 总结

排序是我们经常使用的东西。了解排序算法的工作原理以及根据数据集类型如何使用这些算法是很重要的。我们对基本方法进行了一些关键的改变，以确保我们优化了我们的算法，并最终得出了一些统计数据，以了解这些算法在相互比较时的效率如何。当然，有人可能会想到是否有必要进行性能测试来检查一个算法是否比另一个更好。我们将在接下来的章节中讨论这个问题。
