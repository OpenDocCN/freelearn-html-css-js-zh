# 附录 E. 练习问题的答案

本附录列出了章节末尾练习的可能答案。可能的答案意味着它们不是唯一的答案，所以如果您的解决方案不同，不要担心。

与本书的其余部分一样，您应该在控制台中尝试并玩一下。

第一章和最后一章没有*练习*部分，所以让我们从第二章开始，*原始数据类型、数组、循环和条件*。

# 第二章，原始数据类型、数组、循环和条件

让我们尝试解决以下练习：

## 练习

1.  结果将如下：

```js
        > var a; typeof a; 
        "undefined" 

```

当您声明一个变量但不用值初始化它时，它会自动获得未定义的值。您还可以检查：

```js
        > a === undefined; 
        true 

```

`v`的值将是：

```js
        > var s = '1s'; s++; 
        NaN 

```

将数字`1`添加到字符串`'1s'`中，返回字符串`'1s1'`，这是*不是一个数字*，但`++`运算符应该返回一个数字；所以它返回特殊的`NaN`数字。

程序如下：

```js
        > !!"false"; 
        true 

```

问题的棘手部分在于`"false"`是一个字符串，所有字符串在转换为布尔值时都是`true`（除了空字符串`""`）。如果问题不是关于字符串`"false"`而是布尔值`false`，则双重否定!!将返回相同的布尔值：

```js
        > !!false; 
        false 

```

正如您所期望的，单个否定返回相反的值：

```js
        > !false; 
        true 
        > !true; 
        false 

```

您可以测试任何字符串，它都会转换为布尔值`true`，除了空字符串：

```js
        > !!"hello"; 
        true 
        > !!"0"; 
        true 
        > !!""; 
        false 

```

执行`undefined`后的输出如下：

```js
        > !!undefined; 
        false 

```

这里`undefined`是假值之一，它转换为`false`。您可以尝试任何其他假值，例如前面示例中的空字符串`""`，`NaN`或`0`。

```js
        > typeof -Infinity; 
        "number" 

```

数字类型包括所有数字、`NaN`、正数和负数`Infinity`。

执行以下操作后的输出是：

```js
        > 10 % "0"; 
        NaN 

```

字符串`"0"`被转换为数字`0`。除以`0`得到`Infinity`，没有余数。

执行以下操作后的输出是：

```js
        > undefined == null; 
        true 

```

与`==`运算符的比较不检查类型，但转换操作数；在这种情况下，两者都是假值。严格比较也检查类型：

```js
        > undefined === null; 
        false 

```

以下是代码行及其输出：

```js
        > false === ""; 
        false 

```

不同类型之间的严格比较（在本例中是布尔值和字符串）注定会失败，无论值是什么。

以下是代码行及其输出：

```js
        > typeof "2E+2"; 
        "string" 

```

引号中的任何内容都是字符串，尽管：

```js
        > 2E+2; 
        200 
        > typeof 2E+2; 
        "number" 

```

以下是代码行及其输出：

```js
        > a = 3e+3; a++; 
        3000 

```

`3e+3`是`3`加上三个零，意思是`3000`。然后`++`是后增量，意思是它返回旧值，然后增加它并将其分配给`a`。这就是为什么您在控制台中得到返回值`3000`，尽管`a`现在是`3001`：

```js
        > a; 
        3001 

```

1.  执行以下操作后的`v`的值是：

```js
        > var v = v || 10; 
        > v; 
        10 

```

如果`v`从未被声明过，则为`undefined`，因此这与以下内容相同：

```js
        > var v = undefined || 10; 
        > v; 
        10 

```

但是，如果`v`已经被定义并初始化为一个非假值，您将获得先前的值。

```js
        > var v = 100; 
        > var v = v || 10; 
        > v; 
        100 

```

第二次使用`var`不会“重置”变量。

如果`v`已经是一个假值（不是`100`），则检查`v || 10`将返回`10`。

```js
        > var v = 0; 
        > var v = v || 10; 
        > v; 
        10 

```

1.  要打印乘法表，请执行以下操作：

```js
        for (var i = 1; i <= 12; i++) { 
          for (var j = 1; j <= 12; j++) { 
            console.log(i + ' * ' + j + ' = ' + i * j); 
          } 
        } 

```

或：

```js
        var i = 1, j = 1; 
        while (i <= 12) { 
          while (j <= 12) { 
            console.log(i + ' * ' + j + ' = ' + i * j); 
            j++; 
          } 
          i++; 
          j = 1; 
        } 

```

# 第三章，函数

让我们做以下练习：

## 练习

1.  要将十六进制颜色转换为 RGB，请执行以下操作：

```js
        function getRGB(hex) { 
          return "rgb(" + 
            parseInt(hex[1] + hex[2], 16) + ", " + 
            parseInt(hex[3] + hex[4], 16) + ", " + 
            parseInt(hex[5] + hex[6], 16) + ")"; 
        } 
        Testing: 
        > getRGB("#00ff00"); 
               "rgb(0, 255, 0)" 
        > getRGB("#badfad"); 
               "rgb(186, 223, 173)" 

```

这种解决方案的一个问题是，像`hex[0]`这样的字符串数组访问不在 ECMAScript 3 中，尽管许多浏览器长期支持它，现在在 ES5 中也有描述。

但是，在本书的这一部分，尚未讨论对象和方法。否则，符合 ES3 的解决方案将是使用字符串方法之一，例如`charAt()`、`substring()`或`slice()`。您还可以使用数组来避免太多的字符串连接：

```js
    function getRGB2(hex) { 
      var result = []; 
      result.push(parseInt(hex.slice(1, 3), 16)); 
      result.push(parseInt(hex.slice(3, 5), 16)); 
      result.push(parseInt(hex.slice(5), 16)); 
      return "rgb(" + result.join(", ") + ")"; 
    } 

```

**奖励练习**：重写前面的函数，使用循环，这样您就不必三次输入`parseInt()`，而只需一次。

1.  结果如下：

```js
        > parseInt(1e1); 
        10 
        Here, you're parsing something that is already an integer: 
        > parseInt(10); 
        10 
        > 1e1; 
        10 

```

在这里，字符串的解析放弃了第一个非整数值。`parseInt()`不理解指数文字，它期望整数表示法：

```js
        > parseInt('1e1'); 
        1 

```

这是解析字符串`'1e1'`，同时期望它是十进制表示法，包括指数：

```js
        > parseFloat('1e1'); 
        10 

```

以下是代码行及其输出：

```js
        > isFinite(0 / 10); 
        true 

```

因为`0/10`是`0`，而`0`是有限的。

以下是代码行及其输出：

```js
        > isFinite(20 / 0); 
        false 

```

因为除以`0`是`Infinity`：

```js
        > 20 / 0; 
        Infinity 

```

以下是代码行及其输出：

```js
        > isNaN(parseInt(NaN)); 
        true 

```

解析特殊的`NaN`值是`NaN`。

1.  以下是结果：

```js
        var a = 1; 
        function f() { 
          function n() { 
            alert(a); 
          } 
          var a = 2; 
          n(); 
        } 
        f(); 

```

这段代码警报`2`，即使`n()`在赋值`a = 2`之前被定义。在函数`n()`内部，你看到的是在相同作用域中的变量`a`，并且在调用`f()`（因此`n()`）时访问它的最新值。由于变量提升，`f()`的行为就像是：

```js
        function f() { 
          var a; 
          function n() { 
            alert(a); 
          } 
          a = 2; 
          n(); 
        } 

```

更有趣的是，考虑这段代码：

```js
        var a = 1; 
        function f() { 
          function n() { 
            alert(a); 
          } 
          n(); 
          var a = 2; 
          n(); 
        } 
        f(); 

```

它警报`undefined`，然后是`2`。你可能期望第一个警报显示`1`，但由于变量提升，`a`的声明（而不是初始化）被移动到函数的顶部。就像`f()`是：

```js
        var a = 1; 
        function f() { 
          var a; // a is now undefined 
          function n() { 
            alert(a); 
          } 
          n(); // alert undefined 
          a = 2; 
          n(); // alert 2 
        } 
        f(); 

```

本地的`a`“遮蔽”了全局的`a`，即使它在底部。

1.  为什么所有这些警报都是“Boo!”

以下是示例 1 的结果：

```js
        var f = alert; 
        eval('f("Boo!")'); 

```

以下是示例 2 的结果。你可以将一个函数分配给另一个变量。所以`f()`指向`alert()`。评估这个字符串就像这样：

```js
        > f("Boo"); 

```

在我们执行`eval()`之后，以下是输出：

```js
        var e; 
        var f = alert; 
        eval('e=f')('Boo!'); 

```

以下是示例 3 的输出。`eval()`返回评估的结果。在这种情况下，它是一个赋值`e = f`，也返回`e`的新值。就像下面这样：

```js
        > var a = 1; 
        > var b; 
        > var c = (b = a); 
        > c; 
        1 

```

所以`eval('e=f')`给你一个指向`alert()`的指针，它立即执行带有`"Boo!"`的`alert()`。

立即（自调用）匿名函数返回对函数`alert()`的指针，然后立即用参数`"Boo!"`调用它：

```js
        (function(){ 
          return alert; 
        })()('Boo!'); 

```

# 第四章，对象

让我们解决以下练习：

## 练习

1.  这里发生了什么？`this`是什么，`o`是什么？

```js
        function F() { 
          function C() { 
            return this; 
          } 
          return C(); 
        } 
        var o = new F(); 

```

在这里，`this === window`，因为`C()`是在没有`new`的情况下调用的。

还有`o === window`，因为`new F()`返回`C()`返回的对象，即`this`，而`this`是`window`。

你可以将对`C()`的调用变成构造函数调用：

```js
        function F() { 
          function C() { 
            return this; 
          } 
          return new C(); 
        } 
        var o = new F(); 

```

在这里，`this`是由`C()`构造函数创建的对象。`o`也是：

```js
        > o.constructor.name; 
        "C" 

```

在 ES5 的严格模式下会更有趣。在严格模式下，非构造函数调用会导致`this`为`undefined`，而不是全局对象。在`F()`或`C()`构造函数的主体内部使用`"use strict"`，`this`在`C()`中将是`undefined`。因此，`return C()`不能返回非对象的`undefined`（因为所有构造函数调用都返回某种对象），并返回`F`实例的`this`（在闭包范围内）。试试看：

```js
        function F() { 
          "use strict"; 
          this.name = "I am F()"; 
          function C() { 
            console.log(this); // undefined 
            return this; 
          } 
          return C(); 
        } 

```

测试：

```js
        > var o = new F(); 
        > o.name; 
        "I am F()" 

```

1.  使用`new`调用这个构造函数会发生什么？

```js
        function C() { 
          this.a = 1; 
          return false; 
        } 
        And testing: 
        > typeof new C(); 
        "object" 
        > new C().a; 
         1 

```

`new C()`是一个对象，不是布尔值，因为构造函数调用总是产生一个对象。这是你得到的`this`对象，除非你在构造函数中返回其他对象。返回非对象是行不通的，你仍然得到`this`。

1.  这是做什么？

```js
        > var c = [1, 2, [1, 2]]; 
        > c.sort(); 
        > c; 
         [1, Array[2], 2] 

```

这是因为`sort()`比较字符串。`[1, 2].toString()`是`"1,2"`，所以它在`"1"`之后和`"2"`之前。

使用`join()`也是一样的：

```js
        > c.join('--'); 
        > c; 
        "1--1,2--2" 

```

1.  假设`String()`不存在，并创建模仿`String()`的`MyString()`。将输入的原始字符串视为数组（ES5 中正式支持数组访问）。

这是一个只有练习要求的方法的示例实现。随意继续使用其他方法。参考附录 C，*内置对象*，获取完整列表。

```js
        function MyString(input) { 
          var index = 0; 

          // cast to string 
          this._value = '' + input; 

          // set all numeric properties for array access 
          while (input[index] !== undefined) { 
            this[index] = input[index]; 
            index++; 
          } 

          // remember the length 
          this.length = index; 
        } 

        MyString.prototype = { 
          constructor: MyString, 
          valueOf: function valueOf() { 
            return this._value; 
          }, 
          toString: function toString() { 
            return this.valueOf(); 
          }, 
          charAt: function charAt(index) { 
            return this[parseInt(index, 10) || 0]; 
          }, 
          concat: function concat() { 
            var prim = this.valueOf(); 
            for (var i = 0, len = arguments.length; i < len; i++) { 
              prim += arguments[i]; 
            } 
            return prim; 
          }, 
          slice: function slice(from, to) { 
            var result = '', 
                original = this.valueOf(); 
            if (from === undefined) { 
              return original; 
            } 
            if (from > this.length) { 
              return result; 
            } 
            if (from < 0) { 
              from = this.length - from; 
            } 
            if (to === undefined || to > this.length) { 
              to = this.length; 
            } 
            if (to < 0) { 
              to = this.length + to; 
            } 
            // end of validation, actual slicing loop now 
            for (var i = from; i < to; i++) { 
              result += original[i]; 
            } 
            return result; 
          }, 
          split: function split(re) { 
            var index = 0, 
               result = [], 
                original = this.valueOf(), 
                match, 
                pattern = '', 
                modifiers = 'g'; 

            if (re instanceof RegExp) { 
              // split with regexp but always set "g" 
              pattern = re.source; 
              modifiers += re.multiline  ? 'm' : ''; 
              modifiers += re.ignoreCase ? 'i' : ''; 
            } else { 
              // not a regexp, probably a string, we'll convert it 
              pattern = re; 
            } 
            re = RegExp(pattern, modifiers); 

            while (match = re.exec(original)) { 
              result.push(this.slice(index, match.index)); 
              index = match.index + new MyString(match[0]).length; 
            } 
            result.push(this.slice(index)); 
            return result; 
           } 
        }; 

```

测试：

```js
         > var s = new MyString('hello'); 
        > s.length; 
         5 
        > s[0]; 
        "h" 
         > s.toString(); 
         "hello" 
        > s.valueOf(); 
         "hello" 
        > s.charAt(1); 
         "e" 
        > s.charAt('2'); 
        "l" 
        > s.charAt('e'); 
        "h" 
        > s.concat(' world!'); 
        "hello world!" 
        > s.slice(1, 3); 
        "el" 
        > s.slice(0, -1); 
        "hell" 
        > s.split('e'); 
         ["h", "llo"] 
        > s.split('l'); 
         ["he", "", "o"] 

```

随意使用正则表达式进行拆分。

1.  用`reverse()`方法更新`MyString()`：

```js
        > MyString.prototype.reverse = function reverse() { 
            return this.valueOf().split("").reverse().join(""); 
          }; 
        > new MyString("pudding").reverse(); 
         "gniddup" 

```

1.  想象`Array()`消失了，世界需要你实现`MyArray()`。以下是一些方法，让你开始：

```js
        function MyArray(length) { 
          // single numeric argument means length 
          if (typeof length === 'number' && 
              arguments[1] === undefined) { 
            this.length = length; 
            return this; 
          } 

          // usual case 
           this.length = arguments.length; 
          for (var i = 0, len = arguments.length; i < len; i++) { 
            this[i] = arguments[i]; 
          } 
          return this; 

          // later in the book you'll learn how to support 
          // a non-constructor invocation too 
        } 

        MyArray.prototype = { 
          constructor: MyArray, 
          join: function join(glue) { 
            var result = ''; 
            if (glue === undefined) { 
              glue = ','; 
            } 
            for (var i = 0; i < this.length - 1; i++) { 
              result += this[i] === undefined ? '' : this[i]; 
              result += glue; 
            } 
            result += this[i] === undefined ? '' : this[i]; 
            return result; 
          }, 
          toString: function toString() { 
            return this.join(); 
          }, 
          push: function push() { 
            for (var i = 0, len = arguments.length; i < len; i++) { 
              this[this.length + i] = arguments[i]; 
            } 
            this.length += arguments.length; 
            return this.length; 
          }, 
          pop: function pop() { 
            var poppd = this[this.length - 1]; 
            delete this[this.length - 1]; 
            this.length--; 
            return poppd; 
          } 
        }; 

```

测试：

```js
        > var a = new MyArray(1, 2, 3, "test"); 
        > a.toString(); 
        "1,2,3,test" 
        > a.length; 
         4 
        > a[a.length - 1]; 
        "test" 
        > a.push('boo'); 
         5 
        > a.toString(); 
        "1,2,3,test,boo" 
        > a.pop(); 
        "boo" 
        > a.toString(); 
        "1,2,3,test" 
        > a.join(','); 
        "1,2,3,test" 
        > a.join(' isn't '); 
        "1 isn't 2 isn't 3 isn't test" 

```

如果你觉得这个练习有趣，不要停在`join()`上；尽可能多地使用方法。

1.  创建一个`MyMath`对象，它还具有`rand()`，`min([])`，`max([])`。

这里的重点是`Math`不是一个构造函数，而是一个具有一些“静态”属性和方法的对象。以下是一些方法，供您开始使用。

我们还可以使用立即函数来保留一些私有实用函数。您也可以采用上面的`MyString`方法，在那里`this._value`可以真正是私有的。

```js
        var MyMath = (function () { 

         function isArray(ar) { 
            return 
              Object.prototype.toString.call(ar) === 
                '[object Array]'; 
         } 

          function sort(numbers) { 
            // not using numbers.sort() directly because 
            // `arguments` is not an array and doesn't have sort() 
            return Array.prototype.sort.call(numbers, function (a, b) { 
              if (a === b) { 
                return 0; 
              } 
              return  1 * (a > b) - 0.5; // returns 0.5 or -0.5 
           }); 
          } 

          return { 
            PI:   3.141592653589793, 
            E:    2.718281828459045, 
            LN10: 2.302585092994046, 
            LN2:  0.6931471805599453, 
            // ... more constants 
            max: function max() { 
              // allow unlimited number of arguments 
              // or an array of numbers as first argument 
              var numbers = arguments; 
              if (isArray(numbers[0])) { 
                numbers = numbers[0]; 
              } 
              // we can be lazy:  
              // let Array sort the numbers and pick the last 
              return sort(numbers)[numbers.length - 1]; 
            }, 
            min: function min() { 
              // different approach to handling arguments: 
              // call the same function again 
              if (isArray(numbers)) { 
                return this.min.apply(this, numbers[0]); 
              } 

              // Different approach to picking the min: 
              // sorting the array is an overkill, it's too much  
              // work since we don't worry about sorting but only  
              // about the smallest number. 
              // So let's loop: 
              var min = numbers[0]; 
              for (var i = 1; i < numbers.length; i++) { 
                if (min > numbers[i]) { 
                  min = numbers[i]; 
                } 
             } 
              return min; 
            }, 
            rand: function rand(min, max, inclusive) { 
              if (inclusive) { 
                return Math.round(Math.random() * (max - min) + min); 
                // test boundaries for random number 
                // between 10 and 100 *inclusive*: 
                // Math.round(0.000000 * 90 + 10); // 10 
                // Math.round(0.000001 * 90 + 10); // 10 
                // Math.round(0.999999 * 90 + 10); // 100 

              } 
              return Math.floor(Math.random() * (max - min - 1) + min + 1); 
              // test boundaries for random number 
              // between 10 and 100 *non-inclusive*: 
              // Math.floor(0.000000 * (89) + (11)); // 11 
              // Math.floor(0.000001 * (89) + (11)); // 11 
              // Math.floor(0.999999 * (89) + (11)); // 99 
            } 
          }; 
        })(); 

```

在您完成本书并了解 ES5 之后，您可以尝试使用`defineProperty()`来更严格地控制和更接近内置对象的复制。

# 第五章，原型

让我们尝试解决以下练习：

## 练习

1.  创建一个名为`shape`的对象，该对象具有`type`属性和`getType()`方法：

```js
        var shape = { 
          type: 'shape', 
          getType: function () { 
            return this.type; 
          } 
        }; 

```

1.  以下是`Triangle()`构造函数的程序：

```js
        function Triangle(a, b, c) { 
          this.a = a; 
          this.b = b; 
          this.c = c; 
        } 

        Triangle.prototype = shape; 
        Triangle.prototype.constructor = Triangle; 
        Triangle.prototype.type = 'triangle'; 

```

1.  要添加`getPerimeter()`方法，请使用以下代码：

```js
        Triangle.prototype.getPerimeter = function () { 
          return this.a + this.b + this.c; 
        }; 

```

1.  测试以下代码：

```js
        > var t = new Triangle(1, 2, 3); 
        > t.constructor === Triangle; 
        true 
        > shape.isPrototypeOf(t); 
        true 
        > t.getPerimeter(); 
        6 
        > t.getType(); 
        "triangle" 

```

1.  循环遍历`t`，只显示自有属性和方法：

```js
        for (var i in t) { 
          if (t.hasOwnProperty(i)) { 
            console.log(i, '=', t[i]); 
         } 
        } 

```

1.  使用以下代码片段随机化数组元素：

```js
        Array.prototype.shuffle = function () { 
          return this.sort(function () { 
            return Math.random() - 0.5; 
          }); 
        }; 

```

测试：

```js
        > [1, 2, 3, 4, 5, 6, 7, 8, 9].shuffle(); 
         [4, 2, 3, 1, 5, 6, 8, 9, 7] 
        > [1, 2, 3, 4, 5, 6, 7, 8, 9].shuffle(); 
         [2, 7, 1, 3, 4, 5, 8, 9, 6] 
        > [1, 2, 3, 4, 5, 6, 7, 8, 9].shuffle(); 
         [4, 2, 1, 3, 5, 6, 8, 9, 7] 

```

# 第六章，继承

让我们解决以下练习：

## 练习

1.  通过混合到原型中进行多重继承，例如：

```js
        var my = objectMulti(obj, another_obj, a_third, { 
          additional: "properties" 
        }); 
        A possible solution: 
        function objectMulti() { 
          var Constr, i, prop, mixme; 

        // constructor that sets own properties 
        var Constr = function (props) { 
          for (var prop in props) { 
            this[prop] = props[prop]; 
          } 
        }; 

       // mix into the prototype 
       for (var i = 0; i < arguments.length - 1; i++) { 
         var mixme = arguments[i]; 
         for (var prop in mixme) { 
           Constr.prototype[prop] = mixme[prop]; 
         } 
       } 

      return new Constr(arguments[arguments.length - 1]);
   } 

```

测试：

```js
        > var obj_a = {a: 1}; 
        > var obj_b = {a: 2, b: 2}; 
        > var obj_c = {c: 3}; 
        > var my = objectMulti(obj_a, obj_b, obj_c, {hello: "world"}); 
        > my.a; 
         2 

```

属性`a`是`2`，因为`obj_b`覆盖了与`obj_a`相同名称的属性（最后一个获胜）：

```js
        > my.b; 
        2 
        > my.c; 
        3 
        > my.hello; 
        "world" 
        > my.hasOwnProperty('a'); 
        false 
        > my.hasOwnProperty('hello'); 
        true 

```

1.  在[`www.phpied.com/files/canvas/`](http://www.phpied.com/files/canvas/)上使用画布示例进行练习。

使用以下代码片段绘制几个三角形：

```js
        new Triangle( 
          new Point(100, 155), 
          new Point(30, 50), 
          new Point(220, 00)).draw(); 

        new Triangle( 
          new Point(10, 15),   
          new Point(300, 50), 
          new Point(20, 400)).draw(); 

```

使用以下代码片段绘制几个正方形：

```js
        new Square(new Point(150, 150), 300).draw(); 
        new Square(new Point(222, 222), 222).draw(); 

```

使用以下代码片段绘制几个矩形：

```js
        new Rectangle(new Point(100, 10), 200, 400).draw(); 
        new Rectangle(new Point(400, 200), 200, 100).draw(); 

```

1.  要添加菱形、风筝、五边形、梯形和圆（重新实现`draw()`），请使用以下代码：

```js
        function Kite(center, diag_a, diag_b, height) { 
          this.points = [ 
            new Point(center.x - diag_a / 2, center.y), 
            new Point(center.x, center.y + (diag_b - height)), 
            new Point(center.x + diag_a / 2, center.y), 
            new Point(center.x, center.y - height) 
          ]; 
          this.getArea = function () { 
            return diag_a * diag_b / 2; 
          }; 
        } 

        function Rhombus(center, diag_a, diag_b) { 
          Kite.call(this, center, diag_a, diag_b, diag_b / 2); 
        } 

        function Trapezoid(p1, side_a, p2, side_b) { 
          this.points = [p1, p2, new Point(p2.x + side_b, p2.y),
          new Point(p1.x + side_a, p1.y) 
          ]; 

          this.getArea = function () { 
            var height = p2.y - p1.y; 
            return height * (side_a + side_b) / 2; 
          }; 
        } 

        // regular pentagon, all edges have the same length 
        function Pentagon(center, edge) { 
          var r = edge / (2 * Math.sin(Math.PI / 5)), 
              x = center.x, 
              y = center.y; 

          this.points = [new Point(x + r, y),
        new Point(x + r * Math.cos(2 * Math.PI / 5), y - r * 
         Math.sin(2 * Math.PI / 5)), 
        new Point(x - r * Math.cos(    Math.PI / 5), y - r * 
         Math.sin(    Math.PI / 5)), 
        new Point(x - r * Math.cos(    Math.PI / 5), y + r * 
         Math.sin(    Math.PI / 5)), 
        new Point(x + r * Math.cos(2 * Math.PI / 5), y + r * 
         Math.sin(2 * Math.PI / 5)) 
          ]; 

          this.getArea = function () { 
            return 1.72 * edge * edge; 
          }; 
        } 

        function Circle(center, radius) { 
          this.getArea = function () { 
            return Math.pow(radius, 2) * Math.PI; 
          }; 

          this.getPerimeter = function () { 
            return 2 * radius * Math.PI; 
          };   

          this.draw = function () { 
            var ctx = this.context; 
            ctx.beginPath(); 
            ctx.arc(center.x, center.y, radius, 0, 2 * Math.PI); 
            ctx.stroke(); 
          }; 
        } 

        (function () { 
          var s = new Shape(); 
          Kite.prototype = s; 
          Rhombus.prototype = s; 
          Trapezoid.prototype = s; 
          Pentagon.prototype = s; 
          Circle.prototype = s; 
        }()); 

```

测试：

```js
        new Kite(new Point(300, 300), 200, 300, 100).draw(); 
        new Rhombus(new Point(200, 200), 350, 200).draw(); 
        new Trapezoid( 
          new Point(100, 100), 100,  
          new Point(50, 250), 400).draw(); 
        new Pentagon(new Point(400, 400), 100).draw(); 
        new Circle(new Point(500, 300), 270).draw(); 

```

![练习](img/image_69_001.jpg)

测试新形状的结果

1.  想出另一种继承的方法。使用`uber`让子类可以访问其父类。还要让父类意识到它们的子类。

请记住，并非所有子类都继承`Shape`；例如，`Rhombus`继承`Kite`，`Square`继承`Rectangle`。您最终会得到类似这样的东西：

```js
        // inherit(Child, Parent) 
        inherit(Rectangle, Shape); 
        inherit(Square, Rectangle); 

```

在本章和上一个练习中的继承模式中，所有子类都共享相同的原型，例如：

```js
        var s = new Shape(); 
        Kite.prototype = s; 
        Rhombus.prototype = s; 

```

虽然这很方便，但这也意味着没有人可以触及原型，因为这会影响其他人的原型。缺点是所有自定义方法都需要自有属性，例如`this.getArea`。

最好将方法共享在实例之间，并在原型中定义，而不是为每个对象重新创建它们。以下示例将自定义的`getArea()`方法移动到原型中。

在继承函数中，您会看到子类只继承父类的原型。因此，诸如`this.lines`之类的自有属性将不会被设置。因此，您需要让每个子类构造函数调用其`uber`以获取自有属性，例如：

```js
        Child.prototype.uber.call(this, args...) 

```

另一个很好的功能是将已添加到子类的原型属性传递给子类。这允许子类首先继承，然后添加更多自定义或者反过来，这也更方便一些。

```js
        function inherit(Child, Parent) { 
          // remember prototype 
          var extensions = Child.prototype; 

          // inheritance with an intermediate F() 
          var F = function () {}; 
           F.prototype = Parent.prototype; 
          Child.prototype = new F(); 
          // reset constructor 
          Child.prototype.constructor = Child; 
          // remember parent 
          Child.prototype.uber = Parent; 

          // keep track of who inherits the Parent 
          if (!Parent.children) { 
            Parent.children = []; 
          } 
          Parent.children.push(Child); 

          // carry over stuff previsouly added to the prototype 
          // because the prototype is now overwritten completely 
          for (var i in extensions) { 
            if (extensions.hasOwnProperty(i)) { 
              Child.prototype[i] = extensions[i]; 
            } 
          } 
        } 

```

`Shape()`，`Line()`和`Point()`的一切都保持不变。变化只发生在子类中：

```js
        function Triangle(a, b, c) { 
          Triangle.prototype.uber.call(this); 
          this.points = [a, b, c]; 
        } 

        Triangle.prototype.getArea = function () { 
          var p = this.getPerimeter(), s = p / 2; 
          return Math.sqrt(s * (s - this.lines[0].length) * 
        (s - this.lines[1].length) * (s - this.lines[2].length)); 
        }; 

        function Rectangle(p, side_a, side_b) { 
          // calling parent Shape() 
          Rectangle.prototype.uber.call(this); 

          this.points = [ p, 
            new Point(p.x + side_a, p.y), 
            new Point(p.x + side_a, p.y + side_b), 
            new Point(p.x, p.y + side_b) 
          ]; 
        } 

       Rectangle.prototype.getArea = function () { 
           // Previsouly we had access to side_a and side_b  
           // inside the constructor closure. No more. 
          // option 1: add own properties this.side_a and this.side_b 
          // option 2: use what we already have: 
          var lines = this.getLines(); 
          return lines[0].length * lines[1].length; 
        }; 

        function Square(p, side) { 
          this.uber.call(this, p, side, side); 
          // this call is shorter than Square.prototype.uber.call() 
          // but may backfire in case you inherit  
          // from Square and call uber 
          // try it :-) 
        } 

```

继承：

```js
        inherit(Triangle, Shape); 
        inherit(Rectangle, Shape); 
        inherit(Square, Rectangle); 

```

测试：

```js
        > var sq = new Square(new Point(0, 0), 100); 
        > sq.draw(); 
        > sq.getArea(); 
        10000 

```

测试`instanceof`是否正确：

```js
        > sq.constructor === Square; 
        true 
        > sq instanceof Square; 
        true 
        > sq instanceof Rectangle; 
        true 
        > sq instanceof Shape; 
        true 

```

`children`数组：

```js
        > Shape.children[1] === Rectangle; 
        true 
        > Rectangle.children[0] === Triangle; 
        false 
        > Rectangle.children[0] === Square; 
        true 
        > Square.children; 
        undefined 

```

并且`uber`看起来也不错：

```js
        > sq.uber === Rectangle; 
        true 

```

调用`isPrototypeOf()`也会返回预期的结果：

```js
        Shape.prototype.isPrototypeOf(sq); 
        true 
        Rectangle.prototype.isPrototypeOf(sq); 
        true 
        Triangle.prototype.isPrototypeOf(sq); 
        false 

```

完整的代码可在[`www.phpied.com/files/canvas/index2.html`](http://www.phpied.com/files/canvas/index2.html)上找到，还有来自上一个练习的额外的`Kite()`，`Circle()`等。

# 第七章，浏览器环境

让我们练习以下练习：

## 练习

1.  标题时钟程序如下：

```js
        setInterval(function () { 
          document.title = new Date().toTimeString(); 
        }, 1000); 

```

1.  要动画调整弹出窗口的大小从 200 x 200 到 400 x 400，请使用以下代码：

```js
        var w = window.open( 
            'http://phpied.com', 'my', 
             'width = 200, height = 200'); 

        var i = setInterval((function () { 
          var size = 200; 
          return function () { 
            size += 5; 
            w.resizeTo(size, size); 
            if (size === 400) { 
              clearInterval(i); 
            } 
          }; 
        }()), 100); 

```

每 100 毫秒（1/10 秒），弹出窗口的大小增加五个像素。您保留对间隔`i`的引用，以便在完成后清除它。变量`size`跟踪弹出窗口的大小（为什么不在闭包内保持它私有）。

1.  地震程序如下：

```js
       var i = setInterval((function () { 
          var start = +new Date(); // Date.now() in ES5 
          return function () { 
            w.moveTo( 
              Math.round(Math.random() * 100), 
              Math.round(Math.random() * 100)); 
            if (new Date() - start > 5000) { 
              clearInterval(i); 
            } 
          }; 
         }()), 20); 

```

尝试所有这些，但使用`requestAnimationFrame()`而不是`setInterval()`。

1.  带有回调的不同的`walkDOM()`如下：

```js
        function walkDOM(n, cb) { 
          cb(n); 
          var i, 
              children = n.childNodes, 
              len = children.length, 
              child; 
          for (i = 0; i < len; i++) { 
          child = n.childNodes[i]; 
            if (child.hasChildNodes()) { 
              walkDOM(child, cb); 
            } 
          } 
        } 

```

测试：

```js
        > walkDOM(document.documentElement,
        console.dir.bind(console)); 
       html 
       head 
       title 
       body 
       h1 
       ... 

```

1.  要删除内容并清理函数，请使用以下代码：

```js
        // helper 
        function isFunction(f) { 
          return Object.prototype.toString.call(f) === 
            "[object Function]"; 
        } 

        function removeDom(node) { 
          var i, len, attr; 

          // first drill down inspecting the children 
          // and only after that remove the current node 
          while (node.firstChild) { 
            removeDom(node.firstChild); 
          } 

          // not all nodes have attributes, e.g. text nodes don't 
          len = node.attributes ? node.attributes.length : 0; 

          // cleanup loop 
          // e.g. node === <body>,  
          // node.attributes[0].name === "onload" 
          // node.onload === function()... 
          // node.onload is not enumerable so we can't use  
          // a for-in loop and have to go the attributes route 
          for (i = 0; i < len; i++) { 
            attr = node[node.attributes[i].name]; 
            if (isFunction(attr)) { 
              // console.log(node, attr); 
              attr = null; 
            } 
          } 

          node.parentNode.removeChild(node); 
        } 

```

测试：

```js
        > removeDom(document.body); 

```

1.  要动态包含脚本，请使用以下代码：

```js
        function include(url) { 
          var s = document.createElement('script'); 
          s.src = url; 
          document.getElementsByTagName('head')[0].
          appendChild(s); 
        } 

```

测试：

```js
        > include("http://www.phpied.com/files/jinc/1.js"); 
        > include("http://www.phpied.com/files/jinc/2.js"); 

```

1.  **事件**：事件实用程序如下：

```js
        var myevent = (function () { 

          // wrap some private stuff in a closure 
          var add, remove, toStr = Object.prototype.toString; 

          // helper 
          function toArray(a) { 
            // already an array 
            if (toStr.call(a) === '[object Array]') { 
              return a; 
           } 

            // duck-typing HTML collections, arguments etc 
            var result, i, len; 
            if ('length' in a) { 
              for (result = [], i = 0, len = a.length; i < len; i++)
              { 
                result[i] = a[i]; 
              } 
              return result; 
           } 

            // primitives and non-array-like objects 
            // become the first and single array element 
            return [a]; 
          } 

          // define add() and remove() depending 
          // on the browser's capabilities 
          if (document.addEventListener) { 
            add = function (node, ev, cb) { 
              node.addEventListener(ev, cb, false); 
            }; 
            remove = function (node, ev, cb) { 
              node.removeEventListener(ev, cb, false); 
            }; 
          } else if (document.attachEvent) { 
            add = function (node, ev, cb) { 
              node.attachEvent('on' + ev, cb); 
            }; 
            remove = function (node, ev, cb) { 
              node.detachEvent('on' + ev, cb); 
            }; 
          } else { 
            add = function (node, ev, cb) { 
              node['on' + ev] = cb; 
            }; 
            remove = function (node, ev) { 
              node['on' + ev] = null; 
            }; 
          } 

          // public API 
          return { 

            addListener: function (element, event_name, callback) { 
              // element could also be an array of elements 
              element = toArray(element); 
              for (var i = 0; i < element.length; i++) { 
                add(element[i], event_name, callback); 
              } 
            }, 

           removeListener: function (element, event_name, callback) { 
              // same as add(), only practicing a different loop 
              var i = 0, els = toArray(element), len = els.length; 
             for (; i < len; i++) { 
                remove(els[i], event_name, callback); 
              } 
           }, 

            getEvent: function (event) { 
              return event || window.event; 
            }, 

            getTarget: function (event) { 
              var e = this.getEvent(event); 
              return e.target || e.srcElement; 
            }, 

            stopPropagation: function (event) { 
              var e = this.getEvent(event); 
              if (e.stopPropagation) { 
                e.stopPropagation(); 
              } else { 
                e.cancelBubble = true; 
              } 
            }, 

            preventDefault: function (event) { 
              var e = this.getEvent(event); 
              if (e.preventDefault) { 
                e.preventDefault(); 
              } else { 
                e.returnValue = false; 
              } 
            } 

          }; 
        }()); 

```

**测试**：转到任何带有链接的页面，执行以下操作，然后单击任何链接：

```js
        function myCallback(e) { 
          e = myevent.getEvent(e); 
          alert(myevent.getTarget(e).href); 
          myevent.stopPropagation(e); 
          myevent.preventDefault(e); 
        } 
        myevent.addListener(document.links, 'click', myCallback); 

```

1.  使用以下代码使用键盘移动`div`：

```js
        // add a div to the bottom of the page 
        var div = document.createElement('div'); 
        div.style.cssText = 'width: 100px; height:
         100px; background: red; position: absolute;'; 
        document.body.appendChild(div); 

        // remember coordinates 
        var x = div.offsetLeft; 
        var y = div.offsetTop; 

        myevent.addListener(document.body, 
         'keydown', function (e) { 
         // prevent scrolling 
          myevent.preventDefault(e); 

          switch (e.keyCode) { 
            case 37: // left 
              x--; 
              break; 
            case 38: // up 
              y--; 
              break; 
            case 39: // right 
              x++; 
              break; 
            case 40: // down 
              y++; 
              break; 
            default: 
              // not interested 
          } 

          // move 
          div.style.left = x + 'px'; 
          div.style.top  = y + 'px'; 

        }); 

```

1.  你自己的 Ajax 实用程序：

```js
        var ajax = { 
          getXHR: function () { 
            var ids = ['MSXML2.XMLHTTP.3.0', 
             'MSXML2.XMLHTTP', 'Microsoft.XMLHTTP']; 
            var xhr; 
            if (typeof XMLHttpRequest === 'function') { 
              xhr = new XMLHttpRequest(); 
            } else { 
              // IE: try to find an ActiveX object to use 
              for (var i = 0; i < ids.length; i++) { 
                try { 
                  xhr = new ActiveXObject(ids[i]); 
                  break; 
                } catch (e) {} 
              } 
            } 
            return xhr; 

          }, 
          request: function (url, method, cb, post_body) { 
            var xhr = this.getXHR(); 
            xhr.onreadystatechange = (function (myxhr) { 
              return function () { 
                if (myxhr.readyState === 4 && myxhr.status === 200) { 
                  cb(myxhr); 
                } 
              }; 
            }(xhr)); 
            xhr.open(method.toUpperCase(), url, true); 
            xhr.send(post_body || ''); 
          } 
        }; 

```

在测试时，请记住相同的来源限制适用，因此您必须在相同的域上。您可以转到[`www.phpied.com/files/jinc/`](http://www.phpied.com/files/jinc/)，这是一个目录列表，然后在控制台中进行测试：

```js
        function myCallback(xhr) { 
          alert(xhr.responseText); 
        } 
        ajax.request('1.css', 'get', myCallback); 
        ajax.request('1.css', 'post', myCallback,
         'first=John&last=Smith'); 

```

两者的结果是相同的，但是如果您查看 Web 检查器的**网络**选项卡，您会发现第二个确实是带有主体的`POST`请求。
