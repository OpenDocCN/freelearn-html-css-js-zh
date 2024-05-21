# 第十二章。将物理和声音添加到您的场景

在这最后一章中，我们将看看 Physijs，这是另一个您可以用来扩展 Three.js 基本功能的库。Physijs 是一个允许您在 3D 场景中引入物理的库。通过物理，我们指的是您的对象受到重力的影响，它们可以相互碰撞，可以通过施加冲量移动，并且可以通过铰链和滑块约束其运动。这个库内部使用另一个著名的物理引擎**ammo.js**。除了物理，我们还将看看 Three.js 如何帮助您向场景添加空间声音。

在本章中，我们将讨论以下主题：

+   创建一个 Physijs 场景，其中您的对象受到重力的影响，并且可以相互碰撞。

+   展示如何改变场景中对象的摩擦和恢复（弹性）

+   解释 Physijs 支持的各种形状以及如何使用它们

+   展示如何通过组合简单形状来创建复合形状

+   展示高度场如何允许您模拟复杂的形状

+   通过应用点、铰链、滑块和锥扭转以及“自由度”约束来限制对象的移动

+   向场景添加声音源，其声音音量和方向基于它们与摄像机的距离。

我们要做的第一件事是创建一个可以与 Physijs 一起使用的 Three.js 场景。我们将在我们的第一个示例中这样做。

# 创建一个基本的 Three.js 场景

为 Physijs 设置一个 Three.js 场景非常简单，只需要几个步骤。我们需要做的第一件事是包含正确的 JavaScript 文件，您可以从 GitHub 存储库[`chandlerprall.github.io/Physijs/`](http://chandlerprall.github.io/Physijs/)获取。像这样将 Physijs 库添加到您的 HTML 页面中：

```js
<script type="text/javascript" src="../libs/physi.js"></script>
```

模拟场景需要相当多的处理器。如果我们在渲染线程上运行所有模拟计算（因为 JavaScript 的本质是单线程），它将严重影响场景的帧速率。为了补偿这一点，Physijs 在后台线程中进行计算。这个后台线程是通过大多数现代浏览器实现的“web workers”规范提供的。使用这个规范，您可以在单独的线程中运行 CPU 密集型任务，从而不影响渲染。有关 web workers 的更多信息可以在[`www.w3.org/TR/workers/`](http://www.w3.org/TR/workers/)找到。

对于 Physijs，这意味着我们必须配置包含这个工作任务的 JavaScript 文件，并告诉 Physijs 它在哪里可以找到需要模拟我们场景的 ammo.js 文件。我们需要包含 ammo.js 文件的原因是，Physijs 是一个围绕 ammo.js 的包装器，使其易于使用。Ammo.js（您可以在[`github.com/kripken/ammo.js/`](https://github.com/kripken/ammo.js/)找到）是实现物理引擎的库；Physijs 只是为这个物理库提供了一个易于使用的接口。由于 Physijs 只是一个包装器，我们也可以将其他物理引擎与 Physijs 一起使用。在 Physijs 存储库上，您还可以找到一个使用不同物理引擎 Cannon.js 的分支。

要配置 Physijs，我们必须设置以下两个属性：

```js
Physijs.scripts.worker = '../libs/physijs_worker.js';
Physijs.scripts.ammo = '../libs/ammo.js';
```

第一个属性指向我们要执行的工作任务，第二个属性指向内部使用的 ammo.js 库。我们需要执行的下一步是创建一个场景。Physijs 提供了一个围绕 Three.js 普通场景的包装器，因此在您的代码中，您可以这样做来创建一个场景：

```js
var scene = new Physijs.Scene();
scene.setGravity(new THREE.Vector3(0, -10, 0));
```

这将创建一个新的场景，应用了物理，并设置了重力。在这种情况下，我们将* y *轴上的重力设置为`-10`。换句话说，物体直下落。您可以为各个轴设置或在运行时更改重力为任何您认为合适的值，场景将相应地做出响应。

在场景中开始模拟物理之前，我们需要添加一些对象。为此，我们可以使用 Three.js 指定对象的常规方式，但我们必须将它们包装在特定的 Physijs 对象中，以便它们可以被 Physijs 库管理，如下面的代码片段所示：

```js
var stoneGeom = new THREE.BoxGeometry(0.6,6,2);
var stone = new Physijs.BoxMesh(stoneGeom, new THREE.MeshPhongMaterial({color: 0xff0000}));
scene.add(stone);
```

在这个例子中，我们创建一个简单的`THREE.BoxGeometry`对象。我们不是创建`THREE.Mesh`，而是创建`Physijs.BoxMesh`，这告诉 Physijs 在模拟物理和检测碰撞时将几何形状视为盒子。Physijs 提供了许多不同形状的网格供您使用。有关可用形状的更多信息可以在本章后面找到。

现在`THREE.BoxMesh`已经添加到场景中，我们已经具备了第一个 Physijs 场景的所有要素。剩下的就是告诉 Physijs 模拟物理并更新场景中对象的位置和旋转。我们可以通过在我们刚刚创建的场景上调用 simulate 方法来实现这一点。因此，为此，我们将基本的渲染循环更改为以下内容：

```js
render = function() {
  requestAnimationFrame(render);
  renderer.render(scene, camera);
  scene.simulate();
}
```

通过调用`scene.simulate()`这一最后一步，我们就完成了 Physijs 场景的基本设置。不过，如果我们运行这个例子，我们不会看到太多。我们只会看到屏幕中央的一个立方体，当场景渲染时开始下落。因此，让我们看一个更复杂的例子，模拟多米诺骨牌倒下的情况。

对于这个例子，我们将创建以下场景：

![创建基本的 Three.js 场景](img/2215OS_12_01.jpg)

如果您在浏览器中打开`01-basic-scene.html`示例，您会看到一组多米诺石，当场景加载时开始倒下。第一个会推倒第二个，依此类推。这个场景的完整物理效果由 Physijs 管理。我们启动此动画的唯一操作是推倒第一个多米诺。实际上，创建这个场景非常简单，只需要几个步骤，如下所示：

1.  定义一个 Physijs 场景。

1.  定义容纳石头的地面区域。

1.  放置石头。

1.  把第一块石头推倒。

让我们跳过这一步，因为我们已经知道如何做，直接进入第二步，定义包含所有石头的沙盒。这个沙盒由几个箱子组成。以下是完成此操作所需的代码：

```js
function createGround() {
  var ground_material = Physijs.createMaterial(new THREE.MeshPhongMaterial({ map: THREE.ImageUtils.loadTexture( '../assets/textures/general/wood-2.jpg' )}),0.9,0.3);

  var ground = new Physijs.BoxMesh(new THREE.BoxGeometry(60, 1, 60), ground_material, 0);

  var borderLeft = new Physijs.BoxMesh(new THREE.BoxGeometry (2, 3, 60), ground_material, 0);
  borderLeft.position.x=-31;
  borderLeft.position.y=2;
  ground.add(borderLeft);

  var borderRight = new Physijs.BoxMesh(new THREE. BoxGeometry (2, 3, 60), ground_material, 0);
  borderRight.position.x=31;
  borderRight.position.y=2;
  ground.add(borderRight);

  var borderBottom = new Physijs.BoxMesh(new THREE. BoxGeometry (64, 3, 2), ground_material, 0);
  borderBottom.position.z=30;
  borderBottom.position.y=2;
  ground.add(borderBottom);

  var borderTop = new Physijs.BoxMesh(new THREE.BoxGeometry (64, 3, 2), ground_material, 0);
  borderTop.position.z=-30;
  borderTop.position.y=2;
  ground.add(borderTop);

  scene.add(ground);
}
```

这段代码并不复杂。首先，我们创建一个简单的盒子作为地面平面，然后我们添加一些边界以防止物体从这个地面平面上掉下来。我们将这些边界添加到地面对象上，以创建一个复合对象。这是 Physijs 将作为单个对象处理的对象。在这段代码中还有一些其他新内容，我们将在接下来的章节中更深入地解释。第一个是我们创建的`ground_material`。我们使用`Physijs.createMaterial`函数来创建这种材料。这个函数包装了一个标准的 Three.js 材料，但允许我们设置材料的`摩擦`和`弹性`（弹性）属性。关于这一点的更多信息可以在下一节中找到。另一个新方面是我们添加到`Physijs.BoxMesh`构造函数的最后一个参数。对于我们在本节中创建的所有`BoxMesh`对象，我们将`0`作为最后一个参数添加。通过这个参数，我们设置了对象的重量。我们这样做是为了防止地面受到场景中的重力影响，以免它下落。

现在我们有了地面，我们可以放置多米诺骨牌。为此，我们创建简单的`Three.BoxGeometry`实例，将它们包裹在`BoxMesh`中，并将它们放置在地面网格的特定位置上，如下所示：

```js
var stoneGeom = new THREE.BoxGeometry(0.6,6,2);
var stone = new Physijs.BoxMesh(stoneGeom, Physijs.createMaterial(new THREE.MeshPhongMaterial(color: scale(Math.random()).hex(),transparent:true, opacity:0.8})));
stone.position.copy(point);
stone.lookAt(scene.position);
stone.__dirtyRotation = true;
stone.position.y=3.5;
scene.add(stone);
```

我们不展示计算每个多米诺骨牌位置的代码（请参阅示例源代码中的`getPoints()`函数）；这段代码只是展示了多米诺骨牌的位置。您可以在这里看到，我们再次创建了`BoxMesh`，它包装了`THREE.BoxGeometry`。为了确保多米诺骨牌正确对齐，我们使用`lookAt`函数来设置它们正确的旋转。如果我们不这样做，它们将面向同一个方向，不会倒下。我们必须确保在手动更新 Physijs 包装对象的旋转（或位置）之后，告诉 Physijs 发生了变化，以便 Physijs 可以更新场景中所有对象的内部表示。对于旋转，我们可以使用内部的`__dirtyRotation`属性，对于位置，我们将`__dirtyPosition`设置为`true`。

现在剩下要做的就是推第一个多米诺骨牌。我们只需将*x*轴上的旋转设置为 0.2，稍微倾斜它。场景中的重力会完成剩下的工作，完全推倒第一个多米诺骨牌。以下是我们如何推第一个多米诺骨牌：

```js
stones[0].rotation.x=0.2;
stones[0].__dirtyRotation = true;
```

这完成了第一个示例，其中已经展示了 Physijs 的许多功能。如果您想要调整重力，可以通过右上角的菜单进行更改。当您按下**resetScene**按钮时，重力的更改将被应用：

![创建基本的 Three.js 场景](img/2215OS_12_02.jpg)

在下一节中，我们将更仔细地看一下 Physijs 材质属性如何影响对象。

# 材质属性

让我们从示例的解释开始。当您打开`02-material-properties.html`示例时，您会看到一个空盒子，与之前的示例有些相似。这个盒子围绕其*x*轴上下旋转。在右上角的菜单中，您有几个滑块，可以用来改变 Physijs 的一些材质属性。这些属性适用于您可以使用**addCubes**和**addSpheres**按钮添加的立方体和球体。当您按下**addSpheres**按钮时，场景中将添加五个球体，当您按下**addCubes**按钮时，将添加五个立方体。以下是一个演示摩擦和恢复的示例：

![材质属性](img/2215OS_12_03.jpg)

这个示例允许您玩弄在创建 Physijs 材质时可以设置的`restitution`（弹性）和`friction`属性。例如，如果您将**cubeFriction**设置为`1`并添加一些立方体，您会发现，即使地面在移动，立方体几乎不会移动。如果您将**cubeFriction**设置为**0**，您会注意到立方体在地面停止水平时立即滑动。以下截图显示了高摩擦力允许立方体抵抗重力：

![材质属性](img/2215OS_12_04.jpg)

您可以在此示例中设置的另一个属性是`restitution`属性。`restitution`属性定义了对象具有的能量在碰撞时有多少被恢复。换句话说，高恢复会创建一个有弹性的对象，低恢复会导致对象在碰撞时立即停止。

### 提示

当您使用物理引擎时，通常不必担心检测碰撞。引擎会处理这个问题。然而，有时候在两个对象发生碰撞时得到通知是非常有用的。例如，您可能想要创建一个声音效果，或者在创建游戏时扣除一条生命。

使用 Physijs，您可以向 Physijs 网格添加事件侦听器，如下面的代码所示：

```js
mesh.addEventListener( 'collision', function( other_object, relative_velocity, relative_rotation, contact_normal ) {
});
```

这样，当此网格与 Physijs 处理的其他网格发生碰撞时，您将得到通知。

一个很好的演示方法是使用球体，将恢复设置为`1`，然后点击**addSpheres**按钮几次。这将创建一些到处弹跳的球体。

在我们继续下一节之前，让我们先看一下这个示例中使用的一点代码：

```js
sphere = new Physijs.SphereMesh(new THREE.SphereGeometry( 2, 20 ), Physijs.createMaterial(new THREE.MeshPhongMaterial({color: colorSphere, opacity: 0.8, transparent: true}), controls.sphereFriction, controls.sphereRestitution));
box.position.set(Math.random() * 50 -25, 20 + Math.random() * 5, Math.random() * 50 -25);
scene.add( sphere );
```

这是当我们向场景添加球体时执行的代码。这一次，我们使用了不同的 Physijs 网格：`Physijs.SphereMesh`。我们创建了`THREE.SphereGeometry`，而提供的最佳匹配逻辑上是`Physijs.SphereMesh`（在下一节中会详细介绍）。当我们创建`Physijs.SphereMesh`时，我们传入我们的几何图形，并使用`Physijs.createMaterial`来创建一个 Physijs 特定的材质。我们这样做是为了可以为这个对象设置`摩擦`和`弹性`。

到目前为止，我们已经看到了`BoxMesh`和`SphereMesh`。在下一节中，我们将解释并展示 Physijs 提供的不同类型的网格，你可以用它们来包装你的几何图形。

# 基本支持的形状

Physijs 提供了一些形状，你可以用它们来包装你的几何图形。在本节中，我们将向你介绍所有可用的 Physijs 网格，并通过一个示例演示这些网格。记住，你只需要用这些网格之一替换`THREE.Mesh`构造函数即可使用这些网格。

以下表格提供了 Physijs 中可用的网格的概述：

| 名称 | 描述 |
| --- | --- |
| --- | --- |
| `Physijs.PlaneMesh` | 这个网格可以用来创建一个零厚度的平面。你也可以使用`BoxMesh`和`THREE.BoxGeometry`以及低高度一起使用。 |
| `Physijs.BoxMesh` | 如果你有类似立方体的几何图形，使用这个网格。例如，这是`THREE.BoxGeometry`的一个很好的匹配。 |
| `Physijs.SphereMesh` | 对于球形，使用这个几何图形。这个几何图形是`THREE.SphereGeometry`的一个很好的匹配。 |
| `Physijs.CylinderMesh` | 使用`THREE.Cylinder`，你可以创建各种类似圆柱体的形状。根据圆柱体的形状，Physijs 提供了多个网格。`Physijs.CylinderMesh`应该用于具有相同顶部半径和底部半径的普通圆柱体。 |
| `Physijs.ConeMesh` | 如果你将顶部半径指定为`0`，并使用底部半径的正值，你可以使用`THREE.Cylinder`来创建一个圆锥。如果你想对这样的对象应用物理效果，Physijs 中最合适的选择是`ConeMesh`。 |
| `Physijs.CapsuleMesh` | 胶囊体就像`THREE.Cylinder`，但顶部和底部都是圆形的。我们稍后将向你展示如何在 Three.js 中创建一个胶囊体。 |
| `Physijs.ConvexMesh` | `hysijs.ConvexMesh`是一个粗糙的形状，你可以用它来创建更复杂的对象。它创建一个凸面（就像`THREE.ConvexGeometry`）来近似复杂对象的形状。 |
| `Physijs.ConcaveMesh` | 虽然`ConvexMesh`是一个粗糙的形状，`ConcaveMesh`是你复杂几何图形的更详细的表示。请注意，使用`ConcaveMesh`会有很高的性能惩罚。通常，最好是要么创建具有自己特定 Physijs 网格的单独几何图形，要么将它们组合在一起（就像我们在前面的示例中所做的那样）。 |
| `Physijs.HeightfieldMesh` | 这个网格是一个非常专业化的网格。使用这个网格，你可以从`THREE.PlaneGeometry`创建一个高度场。查看`03-shapes.html`示例以了解这个网格。 |

我们将快速浏览这些形状，使用`03-shapes.html`作为参考。我们不会进一步解释`Physijs.ConcaveMesh`，因为它的使用非常有限。

在我们看示例之前，让我们先快速看一下`Physijs.PlaneMesh`。这个网格基于`THREE.PlaneGeometry`创建一个简单的平面，如下所示：

```js
var plane = new Physijs.PlaneMesh(new THREE.PlaneGeometry(5,5,10,10), material);

scene.add( plane );
```

在这个函数中，您可以看到我们只是传入一个简单的`THREE.PlaneGeometry`来创建这个网格。如果您将其添加到场景中，您会注意到一些奇怪的事情。您刚刚创建的网格不会对重力产生反应。原因是`Physijs.PlaneMesh`的固定重量为`0`，因此它不会对重力产生反应，也不会被其他对象的碰撞移动。除了这个网格，所有其他网格都会对重力和碰撞产生反应，就像您所期望的那样。以下截图显示了一个高度场，您可以在其中放置各种支持的形状：

![基本支持的形状](img/2215OS_12_05.jpg)

前面的图像显示了`03-shapes.html`示例。在这个示例中，我们创建了一个随机高度场（稍后会详细介绍），并在右上角有一个菜单，您可以使用它来放置各种形状的对象。如果您尝试这个示例，您会看到不同的形状如何对高度图和与其他对象的碰撞产生不同的响应。

让我们来看看一些这些形状的构造：

```js
new Physijs.SphereMesh(new THREE.SphereGeometry(3,20),mat);
new Physijs.BoxMesh(new THREE.BoxGeometry(4,2,6),mat);
new Physijs.CylinderMesh(new THREE.CylinderGeometry(2,2,6),mat);
new Physijs.ConeMesh(new THREE.CylinderGeometry(0,3,7,20,10),mat);
```

这里没有什么特别的；我们创建一个几何体，并使用 Physijs 中最匹配的网格来创建我们添加到场景中的对象。然而，如果我们想要使用`Physijs.CapsuleMesh`呢？Three.js 中没有类似胶囊的几何体，所以我们必须自己创建一个。以下是为此目的编写的代码：

```js
var merged = new THREE.Geometry();
var cyl = new THREE.CylinderGeometry(2, 2, 6);
var top = new THREE.SphereGeometry(2);
var bot = new THREE.SphereGeometry(2);

var matrix = new THREE.Matrix4();
matrix.makeTranslation(0, 3, 0);
top.applyMatrix(matrix);

var matrix = new THREE.Matrix4();
matrix.makeTranslation(0, -3, 0);
bot.applyMatrix(matrix);

// merge to create a capsule
merged.merge(top);
merged.merge(bot);
merged.merge(cyl);

// create a physijs capsule mesh
var capsule = new Physijs.CapsuleMesh(merged, getMaterial());
```

`Physijs.CapsuleMesh`看起来像一个圆柱体，但顶部和底部是圆角的。我们可以通过创建一个圆柱体（`cyl`）和两个球体（`top`和`bot`），然后使用`merge()`函数将它们合并在一起来轻松地在 Three.js 中重新创建这个形状。以下截图显示了一些胶囊体沿着高度图滚动：

![基本支持的形状](img/2215OS_12_06.jpg)

在我们查看高度图之前，让我们来看看您可以添加到这个示例中的最后一个形状，`Physijs.ConvexMesh`。凸包是包裹几何体所有顶点的最小形状。结果形状将只有小于 180 度的角。您可以将此网格用于复杂的形状，如环面结，如下所示：

```js
var convex = new Physijs.ConvexMesh(new THREE.TorusKnotGeometry(0.5,0.3,64,8,2,3,10), material);
```

在这种情况下，对于物理模拟和碰撞，将使用环面结的凸包。这是一种非常好的方法，可以应用物理效果并检测复杂对象的碰撞，同时最小化性能影响。

Physijs 中讨论的最后一个网格是`Physijs.HeightMap`。以下截图显示了使用 Physijs 创建的高度图：

![基本支持的形状](img/2215OS_12_07.jpg)

使用高度图，您可以非常容易地创建一个包含凸起和浅滩的地形。使用`Physijs.Heightmap`，我们确保所有对象对这个地形的高度差作出正确的响应。让我们来看看完成这个任务所需的代码：

```js
var date = new Date();
var pn = new Perlin('rnd' + date.getTime());

function createHeightMap(pn) {

  var ground_material = Physijs.createMaterial(
    new THREE.MeshLambertMaterial({
      map: THREE.ImageUtils.loadTexture('../assets/textures/ground/grasslight-big.jpg')
    }),
    0.3, // high friction
    0.8 // low restitution
  );

  var ground_geometry = new THREE.PlaneGeometry(120, 100, 100, 100);
  for (var i = 0; i < ground_geometry.vertices.length; i++) {
    var vertex = ground_geometry.vertices[i];
    var value = pn.noise(vertex.x / 10, vertex.y / 10, 0);
    vertex.z = value * 10;
  }
  ground_geometry.computeFaceNormals();
  ground_geometry.computeVertexNormals();

  var ground = new Physijs.HeightfieldMesh(
    ground_geometry,
    ground_material,
    0, // mass
    100,
    100
  );
  ground.rotation.x = Math.PI / -2;
  ground.rotation.y = 0.4;
  ground.receiveShadow = true;

  return ground;
}
```

在这段代码片段中，我们采取了几个步骤来创建您在示例中看到的高度图。首先，我们创建了 Physijs 材质和一个简单的`PlaneGeometry`对象。为了从`PlaneGeometry`创建一个崎岖的地形，我们遍历这个几何体的每个顶点，并随机设置`z`属性。为此，我们使用 Perlin 噪声生成器来创建一个凸起地图，就像我们在第十章的*使用画布作为凸起地图*部分中使用的那样，*加载和使用纹理*。我们需要调用`computeFaceNormals`和`computeVertexNormals`来确保纹理、光照和阴影被正确渲染。在这一点上，我们有了包含正确高度信息的`PlaneGeometry`。使用`PlaneGeometry`，我们可以创建`Physijs.HeightFieldMesh`。构造函数的最后两个参数取`PlaneGeometry`的水平和垂直段数，并应与用于构造`PlaneGeometry`的最后两个属性匹配。最后，我们将`HeightFieldMesh`旋转到我们想要的位置，并将其添加到场景中。所有其他 Physijs 对象现在将正确地与这个高度图交互。

# 使用约束来限制对象的移动

到目前为止，我们已经看到了一些基本的物理现象。我们已经看到了各种形状如何对重力、摩擦和恢复力作出反应，以及它们如何影响碰撞。Physijs 还提供了高级构造，允许您限制对象的运动。在 Physijs 中，这些对象被称为约束。以下表格概述了 Physijs 中可用的约束：

| 约束 | 描述 |
| --- | --- |
| `PointConstraint` | 这允许您将一个对象的位置固定到另一个对象的位置。如果一个对象移动，另一个对象也会随之移动，保持它们之间的距离和方向不变。 |
| `HingeConstraint` | `HingeConstraint`允许您限制物体的运动，就像它是在铰链上一样，比如门。 |
| `SliderConstraint` | 这个约束，正如其名称所示，允许您限制物体在一个单一轴上的运动，比如滑动门。 |
| `ConeTwistConstraint` | 使用这个约束，您可以限制一个对象相对于另一个对象的旋转和运动。这个约束的功能类似于球和插座连接，比如您的手臂在肩膀插座中的移动方式。 |
| `DOFConstraint` | `DOFConstraint`允许您指定围绕任意三个轴的运动限制，并允许您设置允许的最小和最大角度。这是可用的约束中最通用的一个。 |

理解这些约束的最简单方法是看到它们在实际中的作用并与它们一起玩耍。为此，我们提供了一个使用所有这些约束的例子，`04-physijs-constraints.js`。以下截图显示了这个例子：

![使用约束来限制物体的移动](img/2215OS_12_08.jpg)

基于这个例子，我们将为您介绍这五个约束中的四个。对于`DOFConstraint`，我们创建了一个单独的例子。我们首先看的是`PointConstraint`。

## 使用 PointConstraint 来限制两个点之间的移动

如果您打开这个例子，您会看到两个红色的球体。这两个球体使用`PointConstraint`连接在一起。在左上角的菜单中，您可以移动绿色的滑块。一旦其中一个滑块碰到一个红色的球体，您会看到它们两个以相同的方式移动，并且它们保持相同的距离，同时仍然遵守重量、重力、摩擦和其他物理方面的规则。

在这个例子中，`PointConstraint`是这样创建的：

```js
function createPointToPoint() {
  var obj1 = new THREE.SphereGeometry(2);
  var obj2 = new THREE.SphereGeometry(2);

  var objectOne = new Physijs.SphereMesh(obj1, Physijs.createMaterial(new THREE.MeshPhongMaterial({color: 0xff4444, transparent: true, opacity:0.7}),0,0));

  objectOne.position.x = -10;
  objectOne.position.y = 2;
  objectOne.position.z = -18;

  scene.add(objectOne);

  var objectTwo = new Physijs.SphereMesh(obj2,Physijs.createMaterial(new THREE.MeshPhongMaterial({color: 0xff4444, transparent: true, opacity:0.7}),0,0));

  objectTwo.position.x = -20;
  objectTwo.position.y = 2;
  objectTwo.position.z = -5;

  scene.add(objectTwo);

  var constraint = new Physijs.PointConstraint(objectOne, objectTwo, objectTwo.position);
  scene.addConstraint(constraint);
}
```

在这里，您可以看到我们使用了 Physijs 特定的网格（在这种情况下是`SphereMesh`）来创建对象，并将它们添加到场景中。我们使用`Physijs.PointConstraint`构造函数来创建约束。这个约束需要三个参数：

+   前两个参数定义了您想要连接在一起的对象。在这种情况下，我们将两个球体连接在一起。

+   第三个参数定义了约束绑定的位置。例如，如果您将第一个对象绑定到一个非常大的对象，您可以将这个位置设置为该对象的右侧。通常，如果您只想连接两个对象在一起，一个很好的选择就是将它设置为第二个对象的位置。

如果您不想将一个对象固定到另一个对象，而是固定到场景中的一个静态位置，您可以省略第二个参数。在这种情况下，第一个对象保持与您指定的位置相同的距离，当然也遵守重力和其他物理方面的规则。

一旦约束被创建，我们可以通过使用`addConstraint`函数将其添加到场景中来启用它。当您开始尝试使用约束时，您可能会遇到一些奇怪的问题。为了使调试更容易，您可以向`addConstraint`函数传递`true`。如果这样做，约束点和方向将显示在场景中。这可以帮助您正确获取约束的旋转和位置。

## 使用 HingeConstraint 创建类似门的约束

`HingeConstraint`，顾名思义，允许您创建一个行为类似铰链的对象。它围绕特定轴旋转，将运动限制在指定的角度内。在我们的示例中，`HingeConstraint`显示为场景中心的两个白色弹簧板。这些弹簧板受约束于小的棕色立方体，并可以围绕它们旋转。如果您想要玩弄这些铰链，您可以通过在**铰链**菜单中勾选`enableMotor`框来启用它们。这将加速弹簧板到**general**菜单中指定的速度。负速度将使铰链向下移动，正速度将使其向上移动。以下屏幕截图显示了铰链处于上升位置和下降位置：

![使用 HingeConstraint 创建类似门的约束](img/2215OS_12_09.jpg)

让我们更仔细地看看我们是如何创建其中一个弹簧板的：

```js
var constraint = new Physijs.HingeConstraint(flipperLeft, flipperLeftPivot, flipperLeftPivot.position, new THREE.Vector3(0,1,0));
scene.addConstraint(constraint);
constraint.setLimits(-2.2, -0.6, 0.1, 0);
```

此约束接受四个参数。让我们更详细地看看每个参数：

| 参数 | 描述 |
| --- | --- |
| `mesh_a` | 传递到函数中的第一个对象是要约束的对象。在本例中，第一个对象是作为弹簧板的白色立方体。这是受约束的对象在其运动中受到约束的对象。 |
| `mesh_b` | 第二个对象定义了`mesh_a`约束到哪个对象。在本例中，`mesh_a`受约束于小的棕色立方体。如果我们移动此网格，`mesh_a`将跟随它移动，仍然保持`HingeConstraint`在原位。您会发现所有约束都有这个选项。例如，如果您创建了一个四处移动的汽车，并希望创建一个打开门的约束，您可以使用此选项。如果省略了第二个参数，铰链将受到场景的约束（永远无法移动）。 |
| `position` | 这是应用约束的点。在本例中，它是`mesh_a`围绕其旋转的铰链点。如果您指定了`mesh_b`，则此铰链点将随着`mesh_b`的位置和旋转而移动。 |
| `axis` | 这是铰链应该围绕其旋转的轴。在本例中，我们将铰链水平设置为(0,1,0)。 |

向场景添加`HingeConstraint`的方式与我们在`PointConstraint`中看到的方式相同。您使用`addConstraint`方法，指定要添加的约束，并可选择添加`true`以显示约束的确切位置和方向，以进行调试。然而，对于`HingeConstraint`，我们还需要定义允许的运动范围。我们使用`setLimits`函数来实现这一点。

此函数接受以下四个参数：

| 参数 | 描述 |
| --- | --- |
| `low` | 运动的最小角度，以弧度表示。 |
| `high` | 运动的最大角度，以弧度表示。 |
| `bias_factor` | 此属性定义约束在位置错误后纠正自身的速率。例如，当铰链被不同对象推出其约束时，它将移动到正确的位置。此值越高，它纠正位置的速度就越快。最好将其保持在`0.5`以下。 |
| `relaxation_factor` | 这定义了约束改变速度的速率。如果设置为较高的值，当达到运动的最小或最大角度时，对象将会弹跳。 |

您可以在运行时更改这些属性。如果您使用这些属性添加`HingeConstraint`，您将看不到太多移动。只有当网格被另一个对象击中或基于重力时，网格才会移动。然而，许多其他约束也可以通过内部电机移动。当您在我们的示例中检查**铰链**子菜单中的`enableMotor`框时，您会看到这一点。以下代码用于启用此电机：

```js
constraint.enableAngularMotor( controls.velocity, controls.acceleration );
```

这将使用提供的加速度将网格（在我们的例子中是弹簧板）加速到指定的速度。如果我们想让弹簧板向另一个方向移动，我们只需指定一个负速度。如果我们没有任何限制，这将导致我们的弹簧板只要电机持续运行就会旋转。要禁用电机，我们只需调用以下代码：

```js
flipperLeftConstraint.disableMotor();
```

现在网格会根据摩擦、碰撞、重力和其他物理方面减速。

## 使用 SliderConstraint 限制运动到单一轴

下一个约束是`SliderConstraint`。使用这个约束，你可以限制物体沿着任意一个轴的运动。在`04-constraints.html`示例中，绿色的滑块可以从**滑块**子菜单中控制。以下截图显示了这个例子：

![使用 SliderConstraint 限制运动到单一轴](img/2215OS_12_10.jpg)

使用**SlidersLeft**按钮，滑块将移动到左侧（它们的下限），使用**SlidersRight**按钮，它们将移动到右侧（它们的上限）。从代码中创建这些约束非常容易：

```js
var constraint = new Physijs.SliderConstraint(sliderMesh, new THREE.Vector3(0, 2, 0), new THREE.Vector3(0, 1, 0));

scene.addConstraint(constraint);
constraint.setLimits(-10, 10, 0, 0);
constraint.setRestitution(0.1, 0.1);
```

从代码中可以看出，这个约束需要三个参数（如果你想将一个对象约束到另一个对象，则需要四个参数）。以下表格解释了这个约束的参数：

| 参数 | 描述 |
| --- | --- |
| `mesh_a` | 传入函数的第一个对象是要约束的对象。在这个例子中，第一个对象是作为滑块的绿色立方体。这是将在其运动中受到约束的对象。 |
| `mesh_b` | 这是第二个对象，定义了`mesh_a`被约束到哪个对象。这是一个可选参数，在这个例子中被省略了。如果省略，网格将被约束到场景。如果指定了，当这个网格移动或其方向改变时，滑块将随之移动。 |
| `position` | 这是约束应用的点。当你将`mesh_a`约束到`mesh_b`时，这一点尤为重要。 |

| `axis` | 这是`mesh_a`将要滑动的轴。请注意，如果指定了`mesh_b`，这是相对于`mesh_b`的方向。在当前版本的 Physijs 中，使用线性电机和线性限制时，这个轴似乎有一个奇怪的偏移。如果你想沿着这个方向滑动，以下内容适用于这个版本：

+   *x*轴：`new` `THREE.Vector3(0,1,0)`

+   *y*轴：`new` `THREE.Vector3(0,0,Math.PI/2)`

+   *z*轴：`new` `THREE.Vector3(Math.PI/2,0,0)`

|

在创建约束并使用`scene.addConstraint`将其添加到场景后，你可以设置`constraint.setLimits(-10, 10, 0, 0)`来指定这个约束的限制，以确定滑块可以滑动多远。你可以在`SliderConstraint`上设置以下限制：

| 参数 | 描述 |
| --- | --- |
| `linear_lower` | 这是物体的线性下限 |
| `linear_upper` | 这是物体的线性上限 |
| `angular_lower` | 这是物体的角度下限 |
| `angular_higher` | 这是物体的角度上限 |

最后，当你击中这些限制时，你可以设置恢复（弹跳）的大小。你可以使用`constraint.setRestitution(res_linear, res_angular)`来设置，第一个参数设置线性限制时的弹跳量，第二个参数设置角限制时的弹跳量。

现在，完整的约束已经配置好了，我们可以等待碰撞发生，使物体滑动，或者使用电机。对于`SlideConstraint`，我们有两个选择：我们可以使用角度电机沿着我们指定的轴加速，遵守我们设置的角度限制，或者使用线性电机沿着我们指定的轴加速，遵守我们设置的线性限制。在这个例子中，我们使用了线性电机。要使用角度电机，请看后面在本章中解释的`DOFConstraint`。

## 使用`ConeTwistConstraint`创建类似球和插座的约束

使用`ConeTwistConstraint`，可以创建一个受限于一组角度的约束。我们可以指定从一个对象到另一个对象的*x*、*y*和*z*轴的最小和最大角度。以下截图显示了`ConeTwistConstraint`允许您以一定角度围绕参考点移动对象：

![使用 ConeTwistConstraint 创建类似球和插座的约束](img/2215OS_12_11.jpg)

要理解`ConeTwistConstraint`最简单的方法是看一下创建它所需的代码。实现这一点所需的代码如下：

```js
var baseMesh = new THREE.SphereGeometry(1);
var armMesh = new THREE.BoxGeometry(2, 12, 3);

var objectOne = new Physijs.BoxMesh(baseMesh,Physijs.createMaterial(new THREE.MeshPhongMaterial({color: 0x4444ff, transparent: true, opacity:0.7}), 0, 0), 0);
objectOne.position.z = 0;
objectOne.position.x = 20;
objectOne.position.y = 15.5;
objectOne.castShadow = true;
scene.add(objectOne);

var objectTwo = new Physijs.SphereMesh(armMesh,Physijs.createMaterial(new THREE.MeshPhongMaterial({color: 0x4444ff, transparent: true, opacity:0.7}), 0, 0), 10);
objectTwo.position.z = 0;
objectTwo.position.x = 20;
objectTwo.position.y = 7.5;
scene.add(objectTwo);
objectTwo.castShadow = true;

var constraint = new Physijs.ConeTwistConstraint(objectOne, objectTwo, objectOne.position);

scene.addConstraint(constraint);

constraint.setLimit(0.5*Math.PI, 0.5*Math.PI, 0.5*Math.PI);
constraint.setMaxMotorImpulse(1);
constraint.setMotorTarget(new THREE.Vector3(0, 0, 0));
```

在这段 JavaScript 代码中，你可能已经认识到了我们之前讨论过的许多概念。我们首先创建了两个对象，并用约束将它们连接起来：`objectOne`（一个球体）和`objectTwo`（一个立方体）。我们将这些对象定位，使得`objectTwo`悬挂在`objectOne`下方。现在我们可以创建`ConeTwistConstraint`。如果你已经看过其他约束的话，那么这个约束所需的参数并不陌生。第一个参数是要约束的对象，第二个参数是第一个对象要被约束到的对象，最后一个参数是约束被构建的位置（在这种情况下，是`objectOne`围绕其旋转的点）。在将约束添加到场景后，我们可以使用`setLimit`函数设置其限制。这个函数接受三个弧度值，用来指定每个轴的最大角度。

就像大多数其他约束一样，我们可以使用约束提供的马达来移动`objectOne`。对于`ConeTwistConstraint`，我们设置了`MaxMotorImpulse`（马达可以施加的力量），并设置了马达应该将`objectOne`移动到的目标角度。在这个例子中，我们将其移动到其直接在球体下方的静止位置。您可以通过设置这个目标值来玩弄这个例子，如下面的截图所示：

![使用 ConeTwistConstraint 创建类似球和插座的约束](img/2215OS_12_12.jpg)

我们将要看的最后一个约束也是最通用的——`DOFConstraint`。

## 使用 DOFConstraint 创建详细控制

`DOFConstraint`，也称为自由度约束，允许您精确控制对象的线性和角运动。我们将通过创建一个示例来展示如何使用这个约束，您可以在这个示例中驾驶一个简单的类似汽车的形状。这个形状由一个作为车身的矩形和四个作为车轮的球体组成。让我们从创建车轮开始：

```js
function createWheel(position) {
  var wheel_material = Physijs.createMaterial(
   new THREE.MeshLambertMaterial({
     color: 0x444444,
     opacity: 0.9,
     transparent: true
    }),
    1.0, // high friction
    0.5 // medium restitution
  );

  var wheel_geometry = new THREE.CylinderGeometry(4, 4, 2, 10);
  var wheel = new Physijs.CylinderMesh(
    wheel_geometry,
    wheel_material,
    100
  );

  wheel.rotation.x = Math.PI / 2;
  wheel.castShadow = true;
  wheel.position = position;
  return wheel;
}
```

在这段代码中，我们只是创建了一个简单的`CylinderGeometry`和`CylinderMesh`对象，它们可以作为我们汽车的车轮。以下截图展示了前面代码的结果：

![使用 DOFConstraint 创建详细控制](img/2215OS_12_13.jpg)

接下来，我们需要创建汽车的车身并将所有东西添加到场景中：

```js
var car = {};
var car_material = Physijs.createMaterial(new THREE.MeshLambertMaterial({
    color: 0xff4444,
    opacity: 0.9,  transparent: true
  }),   0.5, 0.5 
);

var geom = new THREE.BoxGeometry(15, 4, 4);
var body = new Physijs.BoxMesh(geom, car_material, 500);
body.position.set(5, 5, 5);
body.castShadow = true;
scene.add(body);

var fr = createWheel(new THREE.Vector3(0, 4, 10));
var fl = createWheel(new THREE.Vector3(0, 4, 0));
var rr = createWheel(new THREE.Vector3(10, 4, 10));
var rl = createWheel(new THREE.Vector3(10, 4, 0));

scene.add(fr);
scene.add(fl);
scene.add(rr);
scene.add(rl);
```

到目前为止，我们只是创建了组成汽车的各个部件。为了将所有东西联系在一起，我们将创建约束。每个车轮都将被约束到`body`上。约束的创建如下：

```js
var frConstraint = new Physijs.DOFConstraint(fr,body, new THREE.Vector3(0,4,8));
scene.addConstraint(frConstraint);
var flConstraint = new Physijs.DOFConstraint (fl,body, new THREE.Vector3(0,4,2));
scene.addConstraint(flConstraint);
var rrConstraint = new Physijs.DOFConstraint (rr,body, new THREE.Vector3(10,4,8));
scene.addConstraint(rrConstraint);
var rlConstraint = new Physijs.DOFConstraint (rl,body, new THREE.Vector3(10,4,2));
scene.addConstraint(rlConstraint);
```

每个车轮（第一个参数）都有自己的约束，并且它附加到汽车的位置（第二个参数）是用最后一个参数指定的。如果我们按照这个配置运行，我们会看到四个车轮支撑着汽车的车身。为了让汽车移动，我们还需要做两件事：我们需要为车轮设置约束（它们可以沿着哪个轴移动），并且我们需要配置正确的马达。首先，我们为两个前轮设置约束；对于这些前轮，我们希望它们只能沿着*z*轴旋转，以便它们可以驱动汽车，并且不允许它们沿着其他轴移动。

实现这一点所需的代码如下：

```js
frConstraint.setAngularLowerLimit({ x: 0, y: 0, z: 0 });
frConstraint.setAngularUpperLimit({ x: 0, y: 0, z: 0 });
flConstraint.setAngularLowerLimit({ x: 0, y: 0, z: 0 });
flConstraint.setAngularUpperLimit({ x: 0, y: 0, z: 0 });
```

乍一看，这可能看起来很奇怪。通过将下限和上限设置为相同的值，我们确保在指定的方向上不能进行旋转。这也意味着车轮不能围绕其*z*轴旋转。我们之所以这样指定它，是因为当您为特定轴启用马达时，这些限制将被忽略。因此，在这一点上设置*z*轴上的限制对我们的前轮没有任何影响。

我们将使用后轮来转向，并确保它们不会倒下，我们需要固定*x*轴。使用以下代码，我们固定*x*轴（将上限和下限设置为`0`），固定*y*轴，以便这些车轮最初已经转向，并且禁用*z*轴上的任何限制：

```js
rrConstraint.setAngularLowerLimit({ x: 0, y: 0.5, z: 0.1 });
rrConstraint.setAngularUpperLimit({ x: 0, y: 0.5, z: 0 });
rlConstraint.setAngularLowerLimit({ x: 0, y: 0.5, z: 0.1 });
rlConstraint.setAngularUpperLimit({ x: 0, y: 0.5, z: 0 });
```

如您所见，要禁用限制，我们必须将特定轴的下限设置得比上限高。这将允许围绕该轴的自由旋转。如果我们不为*z*轴设置这个，这两个车轮将只是被拖着走。在这种情况下，它们将因与地面的摩擦而与其他车轮一起转动。

剩下的就是为前轮设置马达，可以这样做：

```js
flConstraint.configureAngularMotor(2, 0.1, 0, -2, 1500);
frConstraint.conAngularMotor(2, 0.1, 0, -2, 1500);
```

由于我们可以为三个轴创建一个马达，我们需要指定马达工作的轴：0 是*x*轴，1 是*y*轴，2 是*z*轴。第二个和第三个参数定义了马达的角度限制。在这里，我们再次将下限（`0.1`）设置得比上限（`0`）高，以允许自由旋转。第三个参数指定我们要达到的速度，最后一个参数指定这个马达可以施加的力。如果这最后一个参数太小，汽车就不会移动；如果太大，后轮就会离开地面。

使用以下代码启用它们：

```js
flConstraint.enableAngularMotor(2);
frConstraint.enableAngularMotor(2);
```

如果您打开`05-dof-constraint.html`示例，您可以玩弄各种约束和马达，并驾驶汽车四处走动。以下截图显示了这个例子：

![使用 DOFConstraint 创建详细控制](img/2215OS_12_14.jpg)

在下一节中，我们将看看这本书中我们将讨论的最后一个主题，那就是如何将声音添加到您的 Three.js 场景中。

## 将声源添加到您的场景中

到目前为止，我们已经讨论了很多内容，可以创建美丽的场景、游戏和其他 3D 可视化。然而，我们还没有展示如何将声音添加到您的 Three.js 场景中。在本节中，我们将看看两个 Three.js 对象，它们允许您向场景中添加声音源。这是特别有趣的，因为这些声源会响应摄像机的位置：

+   声源与摄像机之间的距离决定了声源的音量。

+   摄像机左侧和右侧的位置决定了左侧扬声器和右侧扬声器的声音音量。

最好的解释方法是看到它的实际效果。在浏览器中打开`06-audio.html`示例，您会看到三个带有动物图片的立方体。以下截图显示了这个例子：

![向您的场景添加声源](img/2215OS_12_15.jpg)

这个例子使用了我们在第九章中看到的第一人称控制，*动画和移动摄像机*，所以您可以使用箭头键与鼠标结合来在场景中移动。您会发现，您越接近特定的立方体，该特定动物的声音就会越大。如果您将摄像机位置放在狗和奶牛之间，您将会从右侧听到奶牛的声音，从左侧听到狗的声音。

### 提示

在这个例子中，我们使用了一个特定的辅助工具`THREE.GridHelper`，从 Three.js 中创建了立方体下面的网格：

```js
var helper = new THREE.GridHelper( 500, 10 );
helper.color1.setHex( 0x444444 );
helper.color2.setHex( 0x444444 );
scene.add( helper );
```

创建网格时，您需要指定网格的大小（在本例中为 500）和单个网格元素的大小（我们在这里使用了 10）。如果您愿意，还可以通过指定`color1`和`color2`属性来设置水平线的颜色。

只需少量代码即可完成此操作。我们需要做的第一件事是定义`THREE.AudioListener`并将其添加到`THREE.PerspectiveCamera`中，如下所示：

```js
var listener1 = new THREE.AudioListener();
camera.add( listener1 );
```

接下来，我们需要创建`THREE.Mesh`并将`THREE.Audio`对象添加到该网格中，如下所示：

```js
var cube = new THREE.BoxGeometry(40, 40, 40);

var material_1 = new THREE.MeshBasicMaterial({
  color: 0xffffff,
  map: THREE.ImageUtils.loadTexture("../assets/textures/animals/cow.png")
});

var mesh1 = new THREE.Mesh(cube, material_1);
mesh1.position.set(0, 20, 100);

var sound1 = new THREE.Audio(listener1);
sound1.load('../assets/audio/cow.ogg');
sound1.setRefDistance(20);
sound1.setLoop(true);
sound1.setRolloffFactor(2);

mesh1.add(sound1);
```

从这段代码片段中可以看出，我们首先创建了一个标准的`THREE.Mesh`实例。接下来，我们创建了一个`THREE.Audio`对象，将其连接到之前创建的`THREE.AudioListener`对象上。最后，我们将`THREE.Audio`对象添加到我们创建的网格中，完成了整个过程。

有一些属性可以在`THREE.Audio`对象上设置以配置其行为：

+   `load`：这允许我们加载要播放的音频文件。

+   `setRefDistance`：这确定了从对象到声音减小的距离。

+   `setLoop`：默认情况下，声音只播放一次。通过将此属性设置为`true`，声音将循环播放。

+   `setRolloffFactor`：这确定了音量随着远离声源而减小的速度。

在内部，Three.js 使用 Web Audio API（[`webaudio.github.io/web-audio-api/`](http://webaudio.github.io/web-audio-api/)）来播放声音并确定正确的音量。并非所有浏览器都支持此规范。目前最好的支持来自 Chrome 和 Firefox。

# 总结

在本章的最后，我们探讨了如何通过添加物理功能来扩展 Three.js 的基本 3D 功能。为此，我们使用了 Physijs 库，它允许您添加重力、碰撞、约束等等。我们还展示了如何在场景中添加定位声音使用`THREE.Audio`和`THREE.AudioListener`对象。通过这些主题，我们已经完成了关于 Three.js 的这本书。在这些章节中，我们涵盖了许多不同的主题，并探索了 Three.js 所提供的几乎所有内容。在前几章中，我们解释了 Three.js 背后的核心概念和思想；之后，我们研究了可用的灯光以及材质如何影响对象的渲染方式。在掌握基础知识之后，我们探索了 Three.js 提供的各种几何图形以及如何组合几何图形来创建新的图形。

在书的第二部分，我们研究了一些更高级的主题。您学会了如何创建粒子系统，如何从外部来源加载模型，以及如何创建动画。最后，在最后几章中，我们研究了您可以在皮肤中使用的高级纹理以及在场景渲染后可以应用的后处理效果。我们以这一章关于物理的书结束，除了解释如何将物理添加到 Three.js 场景中，还展示了围绕 Three.js 的活跃社区项目，您可以使用这些项目为已经很棒的库添加更多功能。

希望您喜欢阅读本书并且和我写作一样喜欢玩弄示例！
