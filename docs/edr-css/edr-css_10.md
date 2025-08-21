# 附录 1. CSS 选择器性能

2014 年初，我与一些其他开发人员进行了一场*辩论*（我在那里用了引号），讨论了担心 CSS 选择器速度的相关性或无关性。

每当交换关于 CSS 选择器相对速度的理论/证据时，开发人员经常引用*Steve Souders*（[`stevesouders.com/`](http://stevesouders.com/)）2009 年关于 CSS 选择器的工作。它被用来验证诸如*属性选择器速度慢*或*伪选择器速度慢*等说法。

在过去的几年里，我觉得这些事情根本不值得担心。多年来我一直在重复的一句话是：

> *对于 CSS，架构在大括号外；性能在大括号内*

但是，除了参考*Nicole Sullivan 在 Performance Calendar 上的后续帖子*（[`calendar.perfplanet.com/2011/css-selector-performance-has-changed-for-the-better/`](http://calendar.perfplanet.com/2011/css-selector-performance-has-changed-for-the-better/)）来支持我对所使用的选择器并不重要的信念外，我从未真正测试过这个理论。

为了解决这个问题，我尝试自己进行一些测试，以解决这个争论。至少，我相信这会促使更有知识/证据的人提供进一步的数据。

# 测试选择器速度

Steve Souders 之前的测试使用了 JavaScript 的`new Date()`。然而，现在，现代浏览器（iOS/Safari 在测试时是一个明显的例外）支持*导航定时 API*（[`www.w3.org/TR/navigation-timing/`](https://www.w3.org/TR/navigation-timing/)），这为我们提供了更准确的测量。对于测试，我实现了这样的方法：

```css
<script>
    ;(function TimeThisMother() {
        window.onload = function(){
            setTimeout(function(){
            var t = performance.timing;
                alert("Speed of selection is: " + (t.loadEventEnd - t.responseEnd) + " milliseconds");
            }, 0);
        };
    })();
</script>
```

这让我们可以将测试的时间限制在所有资产都已接收（`responseEnd`）和页面呈现（`loadEventEnd`）之间。

因此，我设置了一个非常简单的测试。20 个不同的页面，所有页面都有相同的巨大 DOM，由 1000 个相同的标记块组成：

```css
<div class="tagDiv wrap1">
  <div class="tagDiv layer1" data-div="layer1">
    <div class="tagDiv layer2">
      <ul class="tagUl">
        <li class="tagLi"><b class="tagB"><a href="/" class="tagA link" data-select="link">Select</a></b></li>
      </ul>
    </div>
  </div>
</div>
```

测试了 20 种不同的 CSS 选择方法来将最内部的节点着色为红色。每个页面只在应用于选择块内最内部节点的规则上有所不同。以下是测试的不同选择器和该选择器的测试页面链接：

1.  数据属性：[`benfrain.com/selector-test/01.html`](https://benfrain.com/selector-test/01.html)

1.  数据属性（带修饰）：[`benfrain.com/selector-test/02.html`](https://benfrain.com/selector-test/02.html)

1.  数据属性（未经修饰但有值）：[`benfrain.com/selector-test/03.html`](https://benfrain.com/selector-test/03.html)

1.  数据属性（带值）：[`benfrain.com/selector-test/04.html`](https://benfrain.com/selector-test/04.html)

1.  多个数据属性（带值）：[`benfrain.com/selector-test/05.html`](https://benfrain.com/selector-test/05.html)

1.  单独伪选择器（例如`:after`）：[`benfrain.com/selector-test/06.html`](https://benfrain.com/selector-test/06.html)

1.  组合类（例如`class1.class2`）：[`benfrain.com/selector-test/07.html`](https://benfrain.com/selector-test/07.html)

1.  多个类：[`benfrain.com/selector-test/08.html`](https://benfrain.com/selector-test/08.html)

1.  多个类与子选择器：[`benfrain.com/selector-test/09.html`](https://benfrain.com/selector-test/09.html)

1.  部分属性匹配（例如`[class<sup>ˆ=</sup>“wrap”]`）：[`benfrain.com/selector-test/10.html`](https://benfrain.com/selector-test/10.html)

1.  nth-child 选择器：[`benfrain.com/selector-test/11.html`](https://benfrain.com/selector-test/11.html)

1.  紧接着另一个 nth-child 选择器的 nth-child 选择器：[`benfrain.com/selector-test/12.html`](https://benfrain.com/selector-test/12.html)

1.  疯狂选择（所有选择都有资格，每个类都使用，例如`div.wrapper``> div.tagDiv > div.tagDiv.layer2 > ul.tagUL > li.tagLi > b.tagB > a.TagA.link`）：[`benfrain.com/selector-test/13.html`](https://benfrain.com/selector-test/13.html)

1.  轻微疯狂选择（例如`.tagLi .tagB a.TagA.link`）：[`benfrain.com/selector-test/14.html`](https://benfrain.com/selector-test/14.html)

1.  通用选择器：[`benfrain.com/selector-test/15.html`](https://benfrain.com/selector-test/15.html)

1.  单一元素：[`benfrain.com/selector-test/16.html`](https://benfrain.com/selector-test/16.html)

1.  元素双：[`benfrain.com/selector-test/17.html`](https://benfrain.com/selector-test/17.html)

1.  元素三倍：[`benfrain.com/selector-test/18.html`](https://benfrain.com/selector-test/18.html)

1.  元素三倍带伪：[`benfrain.com/selector-test/19.html`](https://benfrain.com/selector-test/19.html)

1.  单一类：[`benfrain.com/selector-test/20.html`](https://benfrain.com/selector-test/20.html)

每个浏览器上的测试运行了 5 次，并且结果是在 5 个结果之间平均的。测试的浏览器：

+   Chrome 34.0.1838.2 dev

+   Firefox 29.0a2 Aurora

+   Opera 19.0.1326.63

+   Internet Explorer 9.0.8112.16421

+   Android 4.2（7 英寸平板电脑）

使用了 Internet Explorer 的以前版本（而不是我可以使用的最新 Internet Explorer）来揭示*非常绿色*浏览器的表现。所有其他测试过的浏览器都定期更新，所以我想确保现代定期更新的浏览器处理 CSS 选择器的方式与稍旧的浏览器有没有明显的差异。

### 注意

想要自己尝试相同的测试吗？去这个 GitHub 链接获取文件：[`github.com/benfrain/css-performance-tests`](https://github.com/benfrain/css-performance-tests)。只需在您选择的浏览器中打开每个页面（记住浏览器必须支持网络定时 API 以警报响应）。还要注意，当我进行测试时，我丢弃了前几个结果，因为它们在某些浏览器中往往异常高。

### 提示

在考虑结果时，不要将一个浏览器与另一个浏览器进行比较。这不是测试的目的。目的纯粹是为了尝试和评估每个浏览器上使用的不同选择器的选择速度之间的比较差异。例如，选择器 3 是否比任何浏览器上的选择器 7 更快？因此，当查看表格时，最好看列而不是行。

以下是结果。所有时间以毫秒为单位：

| **测试** | **Chrome 34** | **Firefox 29** | **Opera 19** | **IE 19** | **Android 4** |
| --- | --- | --- | --- | --- | --- |
| 1 | 56.8 | 125.4 | 63.6 | 152.6 | 1455.2 |
| 2 | 55.4 | 128.4 | 61.4 | 141 | 1404.6 |
| 3 | 55 | 125.6 | 61.8 | 152.4 | 1363.4 |
| 4 | 54.8 | 129 | 63.2 | 147.4 | 1421.2 |
| 5 | 55.4 | 124.4 | 63.2 | 147.4 | 1411.2 |
| 6 | 60.6 | 138 | 58.4 | 162 | 1500.4 |
| 7 | 51.2 | 126.6 | 56.8 | 147.8 | 1453.8 |
| 8 | 48.8 | 127.4 | 56.2 | 150.2 | 1398.8 |
| 9 | 48.8 | 127.4 | 55.8 | 154.6 | 1348.4 |
| 10 | 52.2 | 129.4 | 58 | 172 | 1420.2 |
| 11 | 49 | 127.4 | 56.6 | 148.4 | 1352 |
| 12 | 50.6 | 127.2 | 58.4 | 146.2 | 1377.6 |
| 13 | 64.6 | 129.2 | 72.4 | 152.8 | 1461.2 |
| 14 | 50.2 | 129.8 | 54.8 | 154.6 | 1381.2 |
| 15 | 50 | 126.2 | 56.8 | 154.8 | 1351.6 |
| 16 | 49.2 | 127.6 | 56 | 149.2 | 1379.2 |
| 17 | 50.4 | 132.4 | 55 | 157.6 | 1386 |
| 18 | 49.2 | 128.8 | 58.6 | 154.2 | 1380.6 |
| 19 | 48.6 | 132.4 | 54.8 | 148.4 | 1349.6 |
| 20 | 50.4 | 128 | 55 | 149.8 | 1393.8 |
| 最大差异 | 16 | 13.6 | 17.6 | 31 | 152 |
| 最低 | 13 | 6 | 13 | 10 | 6 |

## 最快选择器和最慢选择器之间的差异

**最大差异**行显示了最快和最慢选择器之间的毫秒差异。在桌面浏览器中，IE9 以**31**毫秒的最大差异脱颖而出。其他浏览器的差异都在这个数字的一半左右。然而，有趣的是。

## 最慢的选择器

我注意到，最慢的选择器类型在不同的浏览器中有所不同。Opera 和 Chrome 都发现*insanity*选择器（测试 13）最难匹配（这里 Opera 和 Chrome 的相似性可能并不令人惊讶，因为它们共享*blink*引擎），而 Firefox 和 Android 4.2 设备（Tesco hudl 7 英寸平板电脑）都难以匹配单个伪选择器（*测试 6*），Internet Explorer 9 的软肋是部分属性选择器（*测试 10*）。

# 良好的 CSS 架构实践

我们可以肯定的是，使用基于类的选择器的扁平层次结构，就像 ECSS 一样，提供的选择器与其他选择器一样快。 

## 这意味着什么？

对我来说，这证实了我的信念，即担心所使用的选择器类型是绝对愚蠢的。对选择器引擎进行猜测是毫无意义的，因为选择器引擎处理选择器的方式显然是不同的。而且，即使在像这样的庞大 DOM 上，最快和最慢的选择器之间的差异也不是很大。正如我们在英格兰北部所说，“有更重要的事情要做”。

自从记录了我的原始结果以来，WebKit 工程师本杰明·普兰联系我，指出了他对所使用方法的担忧。他的评论非常有趣，他提到的一些信息如下所述：

> *通过选择通过加载来衡量性能，你正在衡量比 CSS 大得多的东西，CSS 性能只是加载页面的一小部分。*

如果以`[class^="wrap"]`的时间配置文件为例（在旧的 WebKit 上进行，以便与 Chrome 有些相似），我看到：

+   ~10%的时间用于光栅化。

+   ~21%的时间用于第一次布局。

+   ~48%的时间用于解析器和 DOM 树的创建

+   ~8%用于样式解析

+   ~5%用于收集样式-这是我们应该测试的内容，也是最耗时的内容。（剩下的时间分布在许多小函数中）。

通过上面的测试，我们可以说我们有一个基线为 100 毫秒的最快选择器。其中，5 毫秒将用于收集样式。如果第二个选择器慢 3 倍，总共将显示为 110 毫秒。测试应该报告 300%的差异，但实际上只显示了 10%。

在这一点上，我回答说，虽然我理解本杰明指出的问题，但我的测试只是为了说明，相同的页面，在其他所有条件相同的情况下，无论使用哪种选择器，渲染基本上都是相同的。本杰明花时间回复并提供了更多细节：

> *我完全同意提前优化选择器是没有用的，但原因完全不同：*
> 
> *仅仅通过检查选择器就几乎不可能预测给定选择器的最终性能影响。在引擎中，选择器被重新排序、拆分、收集和编译。要知道给定选择器的最终性能，你必须知道选择器被收集到了哪个桶中，它是如何编译的，最后 DOM 树是什么样子的。*
> 
> *各种引擎之间都非常不同，使整个过程变得更不可预测。*
> 
> *我反对网页开发人员优化选择器的第二个论点是，他们可能会让情况变得更糟。关于选择器的错误信息比正确的跨浏览器信息要多。有人做正确的事情的机会是相当低的。*
> 
> 在实践中，人们发现 CSS 的性能问题，并开始逐条删除规则，直到问题消失。我认为这是正确的做法，这样做很容易，并且会导致正确的结果。

## 因果关系

在这一点上，我感到 CSS 选择器的使用几乎是无关紧要的。然而，我想知道我们还能从测试中得出什么。

如果页面上的 DOM 元素数量减半，正如你所期望的，完成任何测试的速度也相应下降。但在现实世界中，减少 DOM 的大部分并不总是可能的。这让我想知道 CSS 中未使用的样式数量对结果的影响。

# 样式膨胀会产生什么影响？

*另一个测试*（[`benfrain.com/selector-test/2-01.html`](https://benfrain.com/selector-test/2-01.html)）：我拿了一张与 DOM 树完全无关的庞大样式表。大约有 3000 行 CSS。所有这些无关的样式都是在最后一个选择我们内部的`a.link`节点并将其变红的规则之前插入的。我对每个浏览器进行了 5 次运行的结果平均值。

*然后删除了一半的规则并重复了测试*（[`benfrain.com/selector-test/2-02.html`](https://benfrain.com/selector-test/2-02.html)）以进行比较。以下是结果：

| **测试** | **Chrome 34** | **Firefox 29** | **Opera 19** | **IE 19** | **Android 4** |
| --- | --- | --- | --- | --- | --- |
| 完全膨胀 | 64.4 | 237.6 | 74.2 | 436.8 | 1714.6 |
| 一半膨胀 | 51.6 | 142.8 | 65.4 | 358.6 | 1412.4 |

## 规则减肥

这提供了一些有趣的数据。例如，Firefox 在完成这个测试时比其最慢的选择器测试（测试 6）慢了 1.7 倍。Android 4.3 比其最慢的选择器测试（测试 6）慢了 1.2 倍。Internet Explorer 比其最慢的选择器慢了 2.5 倍！

你可以看到，当删除了一半的样式（大约 1500 行）后，Firefox 的速度大大下降。Android 设备在那时也降到了其最慢选择器的速度。

## 删除未使用的样式

这种恐怖的场景对你来说是不是很熟悉？巨大的 CSS 文件包含各种选择器（通常包含甚至不起作用的选择器），大量更具体的选择器，七层或更深的选择器，不适用的供应商前缀，到处都是 ID 选择器，文件大小为 50-80 KB（有时更大）。

如果你正在处理一个有着庞大 CSS 文件的代码库，而且没有人确切知道所有这些样式实际上是用来做什么的，我的建议是在选择器之前查看 CSS 优化。希望到这一点你会相信 ECSS 方法在这方面可能会有所帮助。

但这并不一定会帮助 CSS 的实际性能。

# 括号内的性能

*最终测试*（[`benfrain.com/selector-test/3-01.html`](https://benfrain.com/selector-test/3-01.html)）我进行的是对页面应用一堆*昂贵*的属性和值。考虑这条规则：

```css

.link {
    background-color: red;
    border-radius: 5px;
    padding: 3px;
    box-shadow: 0 5px 5px #000;
    -webkit-transform: rotate(10deg);
    -moz-transform: rotate(10deg);
    -ms-transform: rotate(10deg);
    transform: rotate(10deg);
    display: block;
}

```

应用了这条规则后，以下是结果：

| **测试** | **Chrome 34** | **Firefox 29** | **Opera 19** | **IE 19** | **Android 4** |
| --- | --- | --- | --- | --- | --- |
| 昂贵的样式 | 65.2 | 151.4 | 65.2 | 259.2 | 1923 |

在这里，所有浏览器至少都达到了其最慢选择器的速度（IE 比其最慢的选择器测试（10）慢了 1.5 倍，Android 设备比最慢的选择器测试（测试 6）慢了 1.3 倍），但这还不是全部。试着滚动那个页面！这种样式的重绘可能会让浏览器崩溃（或者浏览器的等价物）。

我们放在大括号内的属性才是真正影响性能的。可以想象，滚动一个需要无休止昂贵的重绘和布局更改的页面会给设备带来压力。高分辨率屏幕？这会更糟，因为 CPU/GPU 会努力在 16 毫秒内将所有内容重新绘制到屏幕上。

在昂贵的样式测试中，在我测试的 15 英寸 Retina MacBook Pro 上，Chrome 连续绘制模式中显示的绘制时间从未低于 280 毫秒（请记住，我们的目标是低于 16 毫秒）。为了让你有所了解，第一个选择器测试页面从未超过 2.5 毫秒。这不是打错字。这些属性导致绘制时间增加了 112 倍。天啊，这些属性真是昂贵啊！确实是罗宾。确实是。

## 什么属性是昂贵的？

*昂贵*的属性/值配对是我们可以相当确信会使浏览器在重新绘制屏幕时感到吃力的（例如在滚动时）。

我们如何知道什么样式是*昂贵*的？幸运的是，我们可以运用常识来得出一个相当好的想法，知道什么会让浏览器负担。任何需要浏览器在绘制到页面之前进行操作/计算的东西都会更加昂贵。例如，盒阴影，边框半径，透明度（因为浏览器必须计算下面显示的内容），变换和性能杀手，如 CSS 滤镜-如果性能是你的优先考虑因素，那么任何类似的东西都是你的大敌。

### 注意

Juriy kangax Zaytsev 在 2012 年做了一篇`非常棒的博客文章，也涵盖了 CSS 性能`（[`perfectionkills.com/profiling-css-for-fun-and-profit-optimization-notes/`](http://perfectionkills.com/profiling-css-for-fun-and-profit-optimization-notes/)）。他使用各种开发者工具来衡量性能。他特别出色地展示了各种属性对性能的影响。如果你对这种事情感兴趣，那么这篇文章绝对值得一读。

# 总结

从这些测试中得出的一些要点：

+   在现代浏览器中纠结于所使用的选择器是徒劳的；大多数选择方法现在都非常快，真的不值得花太多时间在上面。此外，不同浏览器对最慢的选择器也存在差异。最后查看这里以加快你的 CSS 速度。

+   过多的未使用样式可能会在性能上造成更多的损失，而不是你选择的任何选择器，所以第二要整理那里。在页面上有 3000 行未使用或多余的样式并不罕见。虽然将所有样式都捆绑到一个大的`styles.css`中很常见，但如果站点/网络应用的不同区域可以添加不同的（额外的）样式表（依赖图样式），那可能是更好的选择。

+   如果你的 CSS 随着时间被多位不同的作者添加，可以使用*UnCSS*等工具（[`github.com/giakki/uncss`](https://github.com/giakki/uncss)）来自动删除样式；手动进行这个过程并不有趣！

+   高性能 CSS 的竞争不在于所使用的选择器，而在于对属性和值的慎重使用。

+   快速将某物绘制到屏幕上显然很重要，但用户与页面交互时页面的感觉也很重要。首先寻找昂贵的属性和值对（Chrome 连续重绘模式在这里是你的朋友），它们可能会带来最大的收益。
