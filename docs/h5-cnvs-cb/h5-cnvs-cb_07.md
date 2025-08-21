# 第七章：创建图表和图形

在本章中，我们将涵盖：

+   创建一个饼图

+   创建一个条形图

+   绘制方程

+   用线图绘制数据点

# 介绍

到目前为止，您可能已经注意到第一章到第四章涵盖了 HTML5 画布基础知识，第五章和第六章涵盖了高级主题，而第七章和第八章涵盖了真实的应用。毕竟，如果我们不能产生有用的东西，学习画布有什么好处呢？本章重点是通过创建一些真实的画布应用程序，如创建饼图、条形图、图表和线图。与前几章不同，本章只包含四个示例，因为每个示例都提供了一个完整、易于配置和可投入生产的产品。让我们开始吧！

# 创建一个饼图

饼图可能是最常见的数据可视化之一，因为它们可以快速让用户了解数据元素的相对权重。在这个示例中，我们将创建一个可配置的饼图类，它接受一个数据元素数组并生成一个饼图。此外，我们将以这样的方式构造饼图绘制方法，使得饼图和标签自动填满尽可能多的画布。

![创建饼图](img/1369_07_01.jpg)

## 如何做...

按照以下步骤创建一个可以自动定位和调整饼图和图例的饼图类：

1.  为'PieChart'类定义构造函数，用于绘制饼图：

```js
/*
 * PieChart constructor
 */
function PieChart(canvasId, data){
  // user defined properties
  this.canvas = document.getElementById(canvasId);
  this.data = data;

  // constants
  this.padding = 10;
  this.legendBorder = 2;
  this.pieBorder = 5;
  this.colorLabelSize = 20;
  this.borderColor = "#555";
  this.shadowColor = "#777";
  this.shadowBlur = 10;
  this.shadowX = 2;
  this.shadowY = 2;
  this.font = "16pt Calibri";

    // relationships
    this.context = this.canvas.getContext("2d");
    this.legendWidth = this.getLegendWidth();
    this.legendX = this.canvas.width - this.legendWidth;
    this.legendY = this.padding;
    this.pieAreaWidth = (this.canvas.width - this.legendWidth);
    this.pieAreaHeight = this.canvas.height;
    this.pieX = this.pieAreaWidth / 2;
    this.pieY = this.pieAreaHeight / 2;
    this.pieRadius = (Math.min(this.pieAreaWidth, this.pieAreaHeight) / 2) - (this.padding);

    // draw pie chart
    this.drawPieBorder();
    this.drawSlices();
    this.drawLegend();
}
```

1.  定义'getLegendWidth()'方法，通过考虑最长标签的文本长度来返回图例的宽度：

```js
/*
 * gets the legend width based on the size
 * of the label text
 */
PieChart.prototype.getLegendWidth = function(){
    /*
     * loop through all labels and determine which
     * label is the longest.  Use this information
     * to determine the label width
     */
    this.context.font = this.font;
    var labelWidth = 0;

    for (var n = 0; n < this.data.length; n++) {
        var label = this.data[n].label;
        labelWidth = Math.max(labelWidth, this.context.measureText(label).width);
    }

    return labelWidth + (this.padding * 2) + this.legendBorder + this.colorLabelSize;
};
```

1.  定义'drawPieBorder()'方法，绘制饼图周围的边框：

```js
PieChart.prototype.drawPieBorder = function(){
    var context = this.context;
    context.save();
    context.fillStyle = "white";
    context.shadowColor = this.shadowColor;
    context.shadowBlur = this.shadowBlur;
    context.shadowOffsetX = this.shadowX;
    context.shadowOffsetY = this.shadowY;
    context.beginPath();
    context.arc(this.pieX, this.pieY, this.pieRadius + this.pieBorder, 0, Math.PI * 2, false);
    context.fill();
    context.closePath();
    context.restore();
};
```

1.  定义'drawSlices()'方法，循环遍历数据并为每个数据元素绘制一个饼图切片：

```js
/*
 * draws the slices for the pie chart
 */
PieChart.prototype.drawSlices = function(){
    var context = this.context;
    context.save();
    var total = this.getTotalValue();
    var startAngle = 0;
    for (var n = 0; n < this.data.length; n++) {
        var slice = this.data[n];

        // draw slice
        var sliceAngle = 2 * Math.PI * slice.value / total;
        var endAngle = startAngle + sliceAngle;

        context.beginPath();
        context.moveTo(this.pieX, this.pieY);
        context.arc(this.pieX, this.pieY, this.pieRadius, startAngle, endAngle, false);
        context.fillStyle = slice.color;
        context.fill();
        context.closePath();
        startAngle = endAngle;
    }
  context.restore();
};
```

1.  定义'getTotalValue()'方法，用于获取数据值的总和：

```js
/*
 * gets the total value of the labels by looping through
 * the data and adding up each value
 */
PieChart.prototype.getTotalValue = function(){
    var data = this.data;
    var total = 0;

    for (var n = 0; n < data.length; n++) {
        total += data[n].value;
    }

    return total;
};
```

1.  定义'drawLegend()'方法，绘制图例：

```js
/*
 * draws the legend
 */
PieChart.prototype.drawLegend = function(){
    var context = this.context;
    context.save();
    var labelX = this.legendX;
    var labelY = this.legendY;

    context.strokeStyle = "black";
    context.lineWidth = this.legendBorder;
    context.font = this.font;
    context.textBaseline = "middle";

    for (var n = 0; n < this.data.length; n++) {
        var slice = this.data[n];

        // draw legend label
        context.beginPath();
        context.rect(labelX, labelY, this.colorLabelSize, this.colorLabelSize);
        context.closePath();
        context.fillStyle = slice.color;
        context.fill();
        context.stroke();
        context.fillStyle = "black";
        context.fillText(slice.label, labelX + this.colorLabelSize + this.padding, labelY + this.colorLabelSize / 2);

        labelY += this.colorLabelSize + this.padding;
    }
  context.restore();
};
```

1.  页面加载时，构建数据并实例化一个'PieChart'对象：

```js
window.onload = function(){
    var data = [{
        label: "Eating",
        value: 2,
        color: "red"
    }, {
        label: "Working",
        value: 8,
        color: "blue"
    }, {
        label: "Sleeping",
        value: 8,
        color: "green"
    }, {
        label: "Errands",
        value: 2,
        color: "yellow"
    }, {
        label: "Entertainment",
        value: 4,
        color: "violet"
    }];

    new PieChart("myCanvas", data);
};
```

1.  在 HTML 文档的 body 中嵌入 canvas 标签：

```js
<canvas id="myCanvas" width="600" height="300" style="border:1px solid black;">
</canvas>
```

## 另请参阅...

+   *在第一章中绘制一个弧线*

+   *在第一章中使用文本*

+   *在第二章中绘制一个矩形*

## 它是如何工作的...

在深入了解代码如何工作之前，让我们先退一步，思考一下'PieChart'对象应该做什么。作为开发人员，我们需要传入画布 ID，以便对象知道在哪里绘制，还需要一个数据元素数组，以便知道要绘制什么。

使用'drawSlices()'和'drawPieBorder()'方法呈现'PieChart'元素。'drawSlices()'方法执行以下步骤：

1.  循环遍历数据元素。

1.  通过将 2π乘以总值的值分数来计算每个数据值的角度。

1.  使用'arc()'方法为每个切片绘制弧线。

1.  用数据元素颜色填充每个切片。

饼图呈现后，我们可以使用'drawLegend()'方法绘制图例。此方法执行以下步骤：

1.  循环遍历数据元素。

1.  为每个元素使用'rect()'绘制一个框。

1.  使用'stroke()'和'fill()'为每个框描边和填充数据元素颜色。

1.  使用'fillText()'为每个元素写入相应的标签。

页面加载后，我们可以创建一个数据元素数组，用于标识我们的日常活动以及每个活动对应的小时数，然后通过传入数据数组实例化一个新的 PieChart 对象。

### 提示

在这个示例中，我们通过硬编码数据元素数组来创建了人工数据。然而，在现实生活中，我们更可能通过 JSON 或 XML 等方式提供数据。

# 创建条形图

在饼图之后，条形图是另一个用于可视化数据的流行工具。在这个示例中，我们将创建一个可配置的 Bar Chart 类，该类接受一个数据元素数组并创建一个简单的条形图。我们将重用上一个示例中的数据结构来进行比较。与饼图类不同，条形图绘制方法还会自动缩放图表以填充画布。

![创建条形图](img/1369_07_02.jpg)

## 如何做...

按照以下步骤创建一个 Bar Chart 类，该类可以自动定位和调整一个数据数组的条形图的大小：

1.  定义`BarChart`构造函数，绘制图表：

```js
/*
 * BarChart constructor
 */
function BarChart(config){
    // user defined properties
    this.canvas = document.getElementById(config.canvasId);
    this.data = config.data;
    this.color = config.color;
    this.barWidth = config.barWidth;
    this.gridLineIncrement = config.gridLineIncrement;
    /*
     * adjust max value to highest possible value divisible
     * by the grid line increment value and less than
     * the requested max value
     */
    this.maxValue = config.maxValue - Math.floor(config.maxValue % this.gridLineIncrement);
    this.minValue = config.minValue;

    // constants
    this.font = "12pt Calibri";
    this.axisColor = "#555";
    this.gridColor = "#aaa";
    this.padding = 10;

    // relationships
    this.context = this.canvas.getContext("2d");
    this.range = this.maxValue - this.minValue;
    this.numGridLines = this.numGridLines = Math.round(this.range / this.gridLineIncrement);
    this.longestValueWidth = this.getLongestValueWidth();
    this.x = this.padding + this.longestValueWidth;
    this.y = this.padding * 2;
    this.width = this.canvas.width - (this.longestValueWidth + this.padding * 2);
    this.height = this.canvas.height - (this.getLabelAreaHeight() + this.padding * 4);

    // draw bar chart
    this.drawGridlines();
    this.drawYAxis();
    this.drawXAxis();
    this.drawBars();
    this.drawYVAlues();
    this.drawXLabels();
}
```

1.  定义`getLabelAreaHeight()`方法，确定标签区域的高度（x 轴下方的标签）：

```js
/*
 * gets the label height by finding the max label width and
 * using trig to figure out the projected height since
 * the text will be rotated by 45 degrees
 */
BarChart.prototype.getLabelAreaHeight = function(){
    this.context.font = this.font;
    var maxLabelWidth = 0;

    /*
     * loop through all labels and determine which
     * label is the longest.  Use this information
     * to determine the label width
     */
    for (var n = 0; n < this.data.length; n++) {
        var label = this.data[n].label;
        maxLabelWidth = Math.max(maxLabelWidth, this.context.measureText(label).width);
    }

    /*
     * return y component of the labelWidth which
     * is at a 45 degree angle:
     *
     * a² + b² = c²
     * a = b
     * c = labelWidth
     * a = height component of right triangle
     * solve for a
     */
    return Math.round(maxLabelWidth / Math.sqrt(2));
};
```

1.  定义`getLongestValueWidth()`方法，返回最长值文本宽度：

```js
BarChart.prototype.getLongestValueWidth = function(){
    this.context.font = this.font;
    var longestValueWidth = 0;
    for (var n = 0; n <= this.numGridLines; n++) {
        var value = this.maxValue - (n * this.gridLineIncrement);
        longestValueWidth = Math.max(longestValueWidth, this.context.measureText(value).width);

    }
    return longestValueWidth;
};
```

1.  定义`drawXLabels()`方法，绘制 x 轴标签：

```js
BarChart.prototype.drawXLabels = function(){
    var context = this.context;
    context.save();
    var data = this.data;
    var barSpacing = this.width / data.length;

    for (var n = 0; n < data.length; n++) {
        var label = data[n].label;
        context.save();
        context.translate(this.x + ((n + 1 / 2) * barSpacing), this.y + this.height + 10);
        context.rotate(-1 * Math.PI / 4); // rotate 45 degrees
        context.font = this.font;
        context.fillStyle = "black";
        context.textAlign = "right";
        context.textBaseline = "middle";
        context.fillText(label, 0, 0);
        context.restore();
    }
    context.restore();
};
```

1.  定义`drawYValues()`方法，绘制 y 轴值：

```js
BarChart.prototype.drawYVAlues = function(){
    var context = this.context;
    context.save();
    context.font = this.font;
    context.fillStyle = "black";
    context.textAlign = "right";
    context.textBaseline = "middle";

    for (var n = 0; n <= this.numGridLines; n++) {
        var value = this.maxValue - (n * this.gridLineIncrement);
        var thisY = (n * this.height / this.numGridLines) + this.y;
        context.fillText(value, this.x - 5, thisY);
    }

    context.restore();
};
```

1.  定义`drawBars()`方法，该方法循环遍历所有数据元素，并为每个数据元素绘制一个条形图：

```js
BarChart.prototype.drawBars = function(){
    var context = this.context;
    context.save();
    var data = this.data;
    var barSpacing = this.width / data.length;
    var unitHeight = this.height / this.range;

    for (var n = 0; n < data.length; n++) {
        var bar = data[n];
        var barHeight = (data[n].value - this.minValue) * unitHeight;

        /*
         * if bar height is less than zero, this means that its
         * value is less than the min value.  Since we don't want to draw
         * bars below the x-axis, only draw bars whose height is greater
         * than zero
         */
        if (barHeight > 0) {
            context.save();
            context.translate(Math.round(this.x + ((n + 1 / 2) * barSpacing)), Math.round(this.y + this.height));
            /*
             * for convenience, we can draw the bars upside down
             * starting at the x-axis and then flip
             * them back into the correct orientation using
             * scale(1, -1).  This is a great example of how
             * transformations can help reduce computations
             */
            context.scale(1, -1);

            context.beginPath();
            context.rect(-this.barWidth / 2, 0, this.barWidth, barHeight);
            context.fillStyle = this.color;
            context.fill();
            context.restore();
        }
    }
    context.restore();
};
```

1.  定义`drawGridlines()`方法，该方法在条形图上绘制水平网格线：

```js
BarChart.prototype.drawGridlines = function(){
    var context = this.context;
    context.save();
    context.strokeStyle = this.gridColor;
    context.lineWidth = 2;

    // draw y axis grid lines
    for (var n = 0; n < this.numGridLines; n++) {
        var y = (n * this.height / this.numGridLines) + this.y;
        context.beginPath();
        context.moveTo(this.x, y);
        context.lineTo(this.x + this.width, y);
        context.stroke();
    }
    context.restore();
};
```

1.  定义`drawXAxis()`方法，绘制 x 轴：

```js
BarChart.prototype.drawXAxis = function(){
    var context = this.context;
    context.save();
    context.beginPath();
    context.moveTo(this.x, this.y + this.height);
    context.lineTo(this.x + this.width, this.y + this.height);
    context.strokeStyle = this.axisColor;
    context.lineWidth = 2;
    context.stroke();
    context.restore();
};
```

1.  定义`drawYAxis()`方法，绘制 y 轴：

```js
BarChart.prototype.drawYAxis = function(){
    var context = this.context;
    context.save();
    context.beginPath();
    context.moveTo(this.x, this.y);
    context.lineTo(this.x, this.height + this.y);
    context.strokeStyle = this.axisColor;
    context.lineWidth = 2;
    context.stroke();
    context.restore();
};
```

1.  页面加载时，构建数据并实例化一个新的`BarChart`对象：

```js
window.onload = function(){
    var data = [{
        label: "Eating",
        value: 2
    }, {
        label: "Working",
        value: 8
    }, {
        label: "Sleeping",
        value: 8
    }, {
        label: "Errands",
        value: 2
    }, {
        label: "Entertainment",
        value: 4
    }];

    new BarChart({
        canvasId: "myCanvas",
        data: data,
        color: "blue",
        barWidth: 50,
        minValue: 0,
        maxValue: 10,
        gridLineIncrement: 2
    });
};
```

1.  将画布嵌入到 HTML 文档的 body 中：

```js
<canvas id="myCanvas" width="600" height="300" style="border:1px solid black;">
</canvas>
```

## 它是如何工作的...

与饼图相比，条形图需要更多的配置才能真正通用。对于我们的`BarChart`类的实现，我们需要传入画布 id、数据元素数组、条形颜色、条形宽度、网格线增量（网格线之间的单位数）、最大值和最小值。`BarChart`构造函数使用六种方法来渲染条形图——`drawGridlines()`、`drawYAxis()`、`drawXAxis()`、`drawBars()`、`drawYValues()`和`drawXLabels()`。

`BarChart`类的关键是`drawBars()`方法，该方法遍历所有数据元素，然后为每个数据元素绘制一个矩形。绘制每个条形图的最简单方法是首先垂直反转上下文（使得 y 的正值向上而不是向下），将光标定位在 x 轴上，然后向下绘制一个高度等于数据元素值的矩形。由于上下文在垂直方向上被反转，条形图实际上会向上升起。

## 参见...

+   *在第一章中使用文本*

+   *在第二章中绘制矩形*

+   *在第四章中翻译画布上下文*

+   *在第四章中旋转画布上下文*

+   *在第四章中创建镜像变换*

# 绘制方程

在这个示例中，我们将创建一个可配置的 Graph 类，该类绘制带有刻度线和值的 x 和 y 轴，然后我们将构建一个名为`drawEquation()`的方法，该方法允许我们绘制 f(x)函数。我们将实例化一个 Graph 对象，然后绘制正弦波、抛物线方程和线性方程。

![绘制方程](img/1369_07_03.jpg)

## 如何做...

按照以下步骤创建一个可以绘制带有值的 x 和 y 轴，并且还可以绘制多个 f(x)方程的 Graph 类：

1.  定义`Graph`类的构造函数，绘制 x 和 y 轴：

```js
function Graph(config){
    // user defined properties
    this.canvas = document.getElementById(config.canvasId);
    this.minX = config.minX;
    this.minY = config.minY;
    this.maxX = config.maxX;
    this.maxY = config.maxY;
    this.unitsPerTick = config.unitsPerTick;
    // constants
    this.axisColor = "#aaa";
    this.font = "8pt Calibri";
    this.tickSize = 20;

    // relationships
    this.context = this.canvas.getContext("2d");
    this.rangeX = this.maxX - this.minX;
    this.rangeY = this.maxY - this.minY;
    this.unitX = this.canvas.width / this.rangeX;
    this.unitY = this.canvas.height / this.rangeY;
    this.centerY = Math.round(Math.abs(this.minY / this.rangeY) * this.canvas.height);
    this.centerX = Math.round(Math.abs(this.minX / this.rangeX) * this.canvas.width);
    this.iteration = (this.maxX - this.minX) / 1000;
    this.scaleX = this.canvas.width / this.rangeX;
    this.scaleY = this.canvas.height / this.rangeY;

    // draw x and y axis 
    this.drawXAxis();
    this.drawYAxis();
}
```

1.  定义`drawXAxis()`方法，绘制 x 轴：

```js
Graph.prototype.drawXAxis = function(){
    var context = this.context;
    context.save();
    context.beginPath();
    context.moveTo(0, this.centerY);
    context.lineTo(this.canvas.width, this.centerY);
    context.strokeStyle = this.axisColor;
    context.lineWidth = 2;
    context.stroke();

    // draw tick marks
    var xPosIncrement = this.unitsPerTick * this.unitX;
    var xPos, unit;
    context.font = this.font;
    context.textAlign = "center";
    context.textBaseline = "top";

    // draw left tick marks
    xPos = this.centerX - xPosIncrement;
    unit = -1 * this.unitsPerTick;
    while (xPos > 0) {
        context.moveTo(xPos, this.centerY - this.tickSize / 2);
        context.lineTo(xPos, this.centerY + this.tickSize / 2);
        context.stroke();
        context.fillText(unit, xPos, this.centerY + this.tickSize / 2 + 3);
        unit -= this.unitsPerTick;
        xPos = Math.round(xPos - xPosIncrement);
    }
    // draw right tick marks
    xPos = this.centerX + xPosIncrement;
    unit = this.unitsPerTick;
    while (xPos < this.canvas.width) {
        context.moveTo(xPos, this.centerY - this.tickSize / 2);
        context.lineTo(xPos, this.centerY + this.tickSize / 2);
        context.stroke();
        context.fillText(unit, xPos, this.centerY + this.tickSize / 2 + 3);
        unit += this.unitsPerTick;
        xPos = Math.round(xPos + xPosIncrement);
    }
    context.restore();
};
```

1.  定义`drawYAxis()`方法，绘制 y 轴：

```js
Graph.prototype.drawYAxis = function(){
    var context = this.context;
    context.save();
    context.beginPath();
    context.moveTo(this.centerX, 0);
    context.lineTo(this.centerX, this.canvas.height);
    context.strokeStyle = this.axisColor;
    context.lineWidth = 2;
    context.stroke();

    // draw tick marks  
    var yPosIncrement = this.unitsPerTick * this.unitY;
    var yPos, unit;
    context.font = this.font;
    context.textAlign = "right";
    context.textBaseline = "middle";

    // draw top tick marks
    yPos = this.centerY - yPosIncrement;
    unit = this.unitsPerTick;
    while (yPos > 0) {
        context.moveTo(this.centerX - this.tickSize / 2, yPos);
        context.lineTo(this.centerX + this.tickSize / 2, yPos);
        context.stroke();
        context.fillText(unit, this.centerX - this.tickSize / 2 - 3, yPos);
        unit += this.unitsPerTick;
        yPos = Math.round(yPos - yPosIncrement);
    }

    // draw bottom tick marks
    yPos = this.centerY + yPosIncrement;
    unit = -1 * this.unitsPerTick;
    while (yPos < this.canvas.height) {
        context.moveTo(this.centerX - this.tickSize / 2, yPos);
        context.lineTo(this.centerX + this.tickSize / 2, yPos);
        context.stroke();
        context.fillText(unit, this.centerX - this.tickSize / 2 - 3, yPos);
        unit -= this.unitsPerTick;
        yPos = Math.round(yPos + yPosIncrement);
    }
    context.restore();
};
```

1.  定义`drawEquation()`方法，该方法接受一个函数 f(x)，然后通过循环从`minX`到`maxX`的增量 x 值来绘制方程：

```js
Graph.prototype.drawEquation = function(equation, color, thickness){
    var context = this.context;
    context.save();
    context.save();
    this.transformContext();

    context.beginPath();
    context.moveTo(this.minX, equation(this.minX));

    for (var x = this.minX + this.iteration; x <= this.maxX; x += this.iteration) {
        context.lineTo(x, equation(x));
    }

    context.restore();
    context.lineJoin = "round";
    context.lineWidth = thickness;
    context.strokeStyle = color;
    context.stroke();
    context.restore();
};
```

1.  定义`transformContext()`方法，将上下文平移到图形中心，拉伸图形以适应画布，然后反转 y 轴：

```js
Graph.prototype.transformContext = function(){
    var context = this.context;

    // move context to center of canvas
    this.context.translate(this.centerX, this.centerY);

    /*
     * stretch grid to fit the canvas window, and
     * invert the y scale so that increments
     * as you move upwards
     */
    context.scale(this.scaleX, -this.scaleY);
};
```

1.  当页面加载时，实例化一个新的`Graph`对象，然后使用`drawEquation()`方法绘制三个方程：

```js
window.onload = function(){
    var myGraph = new Graph({
        canvasId: "myCanvas",
        minX: -10,
        minY: -10,
        maxX: 10,
        maxY: 10,
        unitsPerTick: 1
    });

    myGraph.drawEquation(function(x){
        return 5 * Math.sin(x);
    }, "green", 3);

    myGraph.drawEquation(function(x){
        return x * x;
    }, "blue", 3);

    myGraph.drawEquation(function(x){
        return 1 * x;
    }, "red", 3);
};
```

1.  将画布嵌入 HTML 文档的 body 中：

```js
<canvas id="myCanvas" width="600" height="300" style="border:1px solid black;">
</canvas>
```

## 工作原理...

我们的`Graph`类只需要六个参数，`canvasId`、`minX`、`minY`、`maxX`、`maxY`和`unitsPerTick`。实例化后，它使用`drawXAxis()`方法绘制 x 轴，使用`drawYAxis()`方法绘制 y 轴。

`Graph`对象的真正亮点是`drawEquation()`方法，它接受一个方程 f(x)、一个线颜色和一个线粗细。虽然该方法相对较短（大约 20 行代码），但实际上非常强大。它的工作原理如下：

1.  首先调用`transformContext()`方法，将画布上下文定位，缩放上下文以适应画布，并使用`scale()`方法通过将 y 分量乘以-1 来反转 y 轴。这样做是为了使绘图过程更简单，因为增加的 y 值将向上而不是向下（请记住，默认情况下，向下移动时 y 值增加）。

1.  一旦画布上下文准备好，使用`equation`函数确定 x 等于`minX`时的 y 值，即 f(minX)。

1.  使用`moveTo()`移动绘图光标。

1.  通过`for`循环，轻微增加 x 值，并使用方程 f(x)确定每次迭代的相应 y 值。

1.  用`lineTo()`从最后一个点画线到当前点。

1.  继续循环，直到 x 等于`maxX`。

由于每次迭代绘制的线条非常小，人眼看不见，从而产生平滑曲线的错觉。

当页面加载时，我们可以实例化一个新的`Graph`对象，然后通过调用`drawEquation()`方法绘制绿色正弦波、蓝色抛物线方程和红色线性方程。

## 另请参阅...

+   *在第一章中绘制一条线*

+   *在第一章中处理文本*

+   *在第四章中转换画布上下文*

+   *在第四章中缩放画布上下文*

+   *在第四章中创建镜像变换*

# 用线图绘制数据点

如果你曾经上过科学课，你可能熟悉根据实验数据生成线图。线图可能是在传达数据趋势时最有用的数据可视化之一。在这个示例中，我们将创建一个可配置的 Line Chart 类，它接受一个数据元素数组，并绘制每个点，同时用线段连接这些点。

![用线图绘制数据点](img/1369_07_04.jpg)

## 如何做...

按照以下步骤创建一个 Line Chart 类，可以自动定位和调整线图的大小，从一个数据数组中：

1.  定义`LineChart`类的构造函数，绘制 x 轴和 y 轴：

```js
function LineChart(config){
    // user defined properties
    this.canvas = document.getElementById(config.canvasId);
    this.minX = config.minX;
    this.minY = config.minY;
    this.maxX = config.maxX;
    this.maxY = config.maxY;
    this.unitsPerTickX = config.unitsPerTickX;
    this.unitsPerTickY = config.unitsPerTickY;

    // constants
    this.padding = 10;
    this.tickSize = 10;
    this.axisColor = "#555";
    this.pointRadius = 5;
    this.font = "12pt Calibri";
    /*
     * measureText does not provide a text height
     * metric, so we'll have to hardcode a text height
     * value
     */
    this.fontHeight = 12;
    // relationships  
    this.context = this.canvas.getContext("2d");
    this.rangeX = this.maxX - this.minY;
    this.rangeY = this.maxY - this.minY;
    this.numXTicks = Math.round(this.rangeX / this.unitsPerTickX);
    this.numYTicks = Math.round(this.rangeY / this.unitsPerTickY);
    this.x = this.getLongestValueWidth() + this.padding * 2;
    this.y = this.padding * 2;
    this.width = this.canvas.width - this.x - this.padding * 2;
    this.height = this.canvas.height - this.y - this.padding - this.fontHeight;
    this.scaleX = this.width / this.rangeX;
    this.scaleY = this.height / this.rangeY;

    // draw x y axis and tick marks
    this.drawXAxis();
    this.drawYAxis();
}
```

1.  定义`getLongestValueWidth()`方法，返回最长值文本的像素长度：

```js
LineChart.prototype.getLongestValueWidth = function(){
    this.context.font = this.font;
    var longestValueWidth = 0;
    for (var n = 0; n <= this.numYTicks; n++) {
        var value = this.maxY - (n * this.unitsPerTickY);
        longestValueWidth = Math.max(longestValueWidth, this.context.measureText(value).width);
    }
    return longestValueWidth;
};
```

1.  定义`drawXAxis()`方法，绘制 x 轴和标签：

```js
LineChart.prototype.drawXAxis = function(){
    var context = this.context;
    context.save();
    context.beginPath();
    context.moveTo(this.x, this.y + this.height);
    context.lineTo(this.x + this.width, this.y + this.height);
    context.strokeStyle = this.axisColor;
    context.lineWidth = 2;
    context.stroke();

    // draw tick marks
    for (var n = 0; n < this.numXTicks; n++) {
        context.beginPath();
        context.moveTo((n + 1) * this.width / this.numXTicks + this.x, this.y + this.height);
        context.lineTo((n + 1) * this.width / this.numXTicks + this.x, this.y + this.height - this.tickSize);
        context.stroke();
    }

    // draw labels
    context.font = this.font;
    context.fillStyle = "black";
    context.textAlign = "center";
    context.textBaseline = "middle";

    for (var n = 0; n < this.numXTicks; n++) {
        var label = Math.round((n + 1) * this.maxX / this.numXTicks);
        context.save();
        context.translate((n + 1) * this.width / this.numXTicks + this.x, this.y + this.height + this.padding);
        context.fillText(label, 0, 0);
        context.restore();
    }
    context.restore();
};
```

1.  定义`drawYAxis()`方法，绘制 y 轴和值：

```js
LineChart.prototype.drawYAxis = function(){
    var context = this.context;
    context.save();
    context.save();
    context.beginPath();
    context.moveTo(this.x, this.y);
    context.lineTo(this.x, this.y + this.height);
    context.strokeStyle = this.axisColor;
    context.lineWidth = 2;
    context.stroke();
    context.restore();
    // draw tick marks
    for (var n = 0; n < this.numYTicks; n++) {
        context.beginPath();
        context.moveTo(this.x, n * this.height / this.numYTicks + this.y);
        context.lineTo(this.x + this.tickSize, n * this.height / this.numYTicks + this.y);
        context.stroke();
    }

    // draw values
    context.font = this.font;
    context.fillStyle = "black";
    context.textAlign = "right";
    context.textBaseline = "middle";

    for (var n = 0; n < this.numYTicks; n++) {
        var value = Math.round(this.maxY - n * this.maxY / this.numYTicks);
        context.save();
        context.translate(this.x - this.padding, n * this.height / this.numYTicks + this.y);
        context.fillText(value, 0, 0);
        context.restore();
    }
    context.restore();
};
```

1.  定义`drawLine()`方法，循环遍历数据点并绘制连接每个数据点的线段：

```js
LineChart.prototype.drawLine = function(data, color, width){
    var context = this.context;
    context.save();
    this.transformContext();
    context.lineWidth = width;
    context.strokeStyle = color;
    context.fillStyle = color;
    context.beginPath();
    context.moveTo(data[0].x * this.scaleX, data[0].y * this.scaleY);

    for (var n = 0; n < data.length; n++) {
        var point = data[n];

        // draw segment
        context.lineTo(point.x * this.scaleX, point.y * this.scaleY);
        context.stroke();
        context.closePath();
        context.beginPath();
        context.arc(point.x * this.scaleX, point.y * this.scaleY, this.pointRadius, 0, 2 * Math.PI, false);
        context.fill();
        context.closePath();

        // position for next segment
        context.beginPath();
        context.moveTo(point.x * this.scaleX, point.y * this.scaleY);
    }
    context.restore();
};
```

1.  定义`transformContext()`方法，平移上下文，然后垂直反转上下文：

```js
LineChart.prototype.transformContext = function(){
    var context = this.context;

    // move context to center of canvas
    this.context.translate(this.x, this.y + this.height);

    // invert the y scale so that that increments
    // as you move upwards
    context.scale(1, -1);
};
```

1.  页面加载时，实例化一个`LineChart`对象，为蓝线创建一个数据集，使用`drawLine()`绘制线条，为红线定义另一个数据集，然后绘制红线：

```js
window.onload = function(){
    var myLineChart = new LineChart({
        canvasId: "myCanvas",
        minX: 0,
        minY: 0,
        maxX: 140,
        maxY: 100,
        unitsPerTickX: 10,
        unitsPerTickY: 10
    });
    var data = [{
        x: 0,
        y: 0
    }, {
        x: 20,
        y: 10
    }, {
        x: 40,
        y: 15
    }, {
        x: 60,
        y: 40
    }, {
        x: 80,
        y: 60
    }, {
        x: 100,
        y: 50
    }, {
        x: 120,
        y: 85
    }, {
        x: 140,
        y: 100
    }];

    myLineChart.drawLine(data, "blue", 3);

    var data = [{
        x: 20,
        y: 85
    }, {
        x: 40,
        y: 75
    }, {
        x: 60,
        y: 75
    }, {
        x: 80,
        y: 45
    }, {
        x: 100,
        y: 65
    }, {
        x: 120,
        y: 40
    }, {
        x: 140,
        y: 35
    }];

    myLineChart.drawLine(data, "red", 3);
};
```

1.  将画布嵌入到 HTML 文档的主体中：

```js
<canvas id="myCanvas" width="600" height="300" style="border:1px solid black;">
</canvas>
```

## 工作原理...

首先，我们需要使用七个属性配置`LineChart`对象，包括`canvasId`、`minX`、`minY`、`maxX`、`maxY`、`unitsPerTickX`和`unitsPerTickY`。当`LineChart`对象被实例化时，我们将渲染 x 轴和 y 轴。

大部分有趣的事情发生在`drawLine()`方法中，它需要一个数据元素数组、线条颜色和线条粗细。它的工作原理如下：

1.  使用`transformContext()`来平移、缩放和反转上下文。

1.  使用`moveTo()`方法将绘图光标定位在数据数组中的第一个数据点处。

1.  循环遍历所有数据元素，从上一个点到当前点画一条线，然后使用`arc()`方法在当前位置画一个小圆圈。

页面加载后，我们可以实例化`LineChart`对象，为蓝线创建一个数据点数组，使用`drawLine()`方法画线，为红线创建另一个数据点数组，然后画红线。

## 另请参阅...

+   *在第一章中画一条线*

+   *在第一章中处理文本*

+   *在第二章中画一个圆*

+   *在第四章中翻译画布上下文*
