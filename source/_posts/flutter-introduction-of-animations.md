---
title: flutter introduction of animations
date: 2024-06-08 09:17:44
tags:
---

Flutter 是一个多用途的移动应用开发框架，为开发人员配备了强大的工具来创建高质量和高性能的应用程序。它的一个突出特点是对动画的强大支持，这可以使得应用程序的用户界面在视觉上更具吸引力和交互性。

### Flutter 都有哪些动画

就动画来说，Flutter 提供了一系列的工具来实现动画，包括补间动画、基于物理特性的动画，让动画可以模仿现实世界的动态，在你的应用程序中创造出更逼真和自然的运动。

这篇文章会宽泛略带深度的探讨 Flutter 的动画，探索各种各样的动画类型并展示如何在你的项目中实施这些动画。

#### 你需要了解

- Dart
- Flutter

不了解Dart没法理解实例代码，不了解Flutter不知道如何触发动画。所以，这两项是必须要了解的。

### 常用的 Flutter 动画

Flutter 的动画系统围绕着“动画对象”这一概念，这是一个随时间演变的值。这种演变由“动画控制器”控制，它定义了动画的持续时间、方向和其他参数。要设置一个动画，必须将这两个元素连接起来。

#### 常用的动画类

以下是在 Flutter 中创建动画时经常使用的一些关键类：

- Tween（补间）：补间类定义了一个动画要在其间进行插值的数值范围。例如，它可以指定动画的起始值和结束值。
- AnimationController（动画控制器）：这个类控制动画过程。它允许你启动、停止和重新启动动画，以及配置动画的持续时间和曲线。
- AnimatedBuilder（动画构建器）：只要动画发生变化，这个小部件就会自行重建。它在构建涉及多个小部件的复杂动画时特别有用。
- Curve（曲线）：曲线决定了动画的进展速度。Flutter 提供了如线性进度指示器和 Curves.easeInOut 这样的内置曲线，或者你也可以设计自己的自定义曲线。

##### Tween(补间动画）

这个动画是Flutter的基本动画。补间动画，顾名思义了就是在两个值之间插值，然后让一个widget动起来。如下图：
![img.png](img.png)

初始的状态是我们有一个背景色为蓝色的方形的widget，现在要最终让这个widget变为橘色。突然间的变色会让这个过程看起来很突兀，现在要突出丝滑的变色过程。显然，这个变色的过程如果需要开发者手动一帧一帧的设置颜色是不现实的。这个情况下，我们可以使用`ColorTween`，这个类会提供两个蓝色和橘色之间的色值，这样整个动画过程就可以实现了。

简而言之，一个补间动画会提供两个值之间的中间值，变化过程的中间值。比如：色值、整数值、位置等几乎所有属性。这个值不需要开发者提供，补间动画本身可以提供这些值，比如上文提到过的`ColorTween`，我们也可以自己定义`Tween<T>`。同时，动画控制器还可以控制动画是否重复：`controller.repeat()`。

##### 动画控制器

动画控制器，顾名思义，用来控制动画的触发、过程和停止的。

然而，动画控制器的主要用处驱动动画。也就是说它会计算动画动作的下一个值，让动画在定义的范围内动起来。每个Flutter动画都至少需要
两个基本组成：
1. 一个动画可以生成值的范围（Tween）
2. 一个动画控制器

总结，就是Flutter动画至少需要一个`Tween`和一个`AnimationController`。

**注意**：动画控制器用完了要销毁！
```dart
  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

现在你对Flutter动画有一些基本的了解了，现在可以新建一个自己的动画了。

```dart
import 'package:flutter/material.dart';

class TweenAnimationPage extends StatefulWidget {
  const TweenAnimationPage({super.key});

  @override
  State<StatefulWidget> createState() => _TweenAnimationPageState();
}

class _TweenAnimationPageState extends State<TweenAnimationPage>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;
  late Animation<double> _animation;
  late Animation _colorAnimation;

  @override
  void initState() {
    super.initState();

    _controller =
        AnimationController(duration: const Duration(seconds: 2), vsync: this);
    _animation = Tween<double>(begin: 0, end: 1).animate(_controller)
      ..addListener(() {
        setState(() {});
      });

    _colorAnimation =
        ColorTween(begin: Colors.white, end: Colors.orange).animate(_controller)
          ..addListener(() {
            setState(() {});
          });

    _controller.repeat(reverse: true);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("Tween Animation"),
      ),
      body: Center(
        child: Opacity(
            opacity: _animation.value,
            child: Text(
              "Tween Animation",
              style: TextStyle(color: _colorAnimation.value),
            )),
      ),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```
1. 这一段代码：`class _TweenAnimationPageState extends State<TweenAnimationPage>
   with SingleTickerProviderStateMixin {`用到了Dart的**mixin**。 使用这个mixin是因为需要用到动画控制器的
*vsync*。同时，这个mixin也提供了动画控制器计算动画插值的一些工具。
2. 初始化和配置动画控制器。代码如下：
```dart
  late AnimationController _controller;
  late Animation<double> _animation;
  late Animation _colorAnimation;

  @override
  void initState() {
    super.initState();

    _controller =
        AnimationController(duration: const Duration(seconds: 2), vsync: this);
    _animation = Tween<double>(begin: 0, end: 1).animate(_controller)
      ..addListener(() {
        setState(() {});
      });

    _colorAnimation =
        ColorTween(begin: Colors.white, end: Colors.orange).animate(_controller)
          ..addListener(() {
            setState(() {});
          });

    _controller.repeat(reverse: true);
  }
```

`initState()` 是构建一个widget的时候要运行的第一个方法。我们在这里初始化动画和动画控制器。

在控制器中，我们给`vsync`参数传入了`this`作为参数。这是`SingleTickerProviderStateMixin` mixin提供的
我们也把`duration`设置为2秒。

接下来，动画示例会用到`Tween`。第一个是`Tween<double>`，第二个是`ColorTween`。
1. `Tween<double>`, 这个tween会用到文本的不透明度属性。这个属性需要的是一个`double`值，所以用一个Tween的泛型
，类型参数是double。
2. `ColorTween`, 颜色这个就不是`double`值可以搞定的了，所以用专属的`ColorTween`来计算颜色值的变化。

现在来处理动画widget。

```dart
body: Center(
  child: Opacity(
      opacity: _animation.value, // 1
      child: Text(
        "Tween Animation",
        style: TextStyle(color: _colorAnimation.value), // 2
      )),
),
```
1. `Opacity`，的属性值需要根据动画提供的值改变，这里用`Tween<double>`的动画的值。
2. `TextStyle`，的`color`属性也需要跟着动画改变，这里用`ColorTween`的动画的值。

##### 让动画动起来

让动画起来，这就需要动画控制器了。

关键点就在`initState`方法的最后一行代码：`_controller.repeat(reverse: true);`。调用了动画控制器实例的`repeat`方法，并且参数为**true**，那么
这个动画就会一直循环播放。在app运行起来之后就可以看到动画在循环播放了。

但是，只是这样的话是不够的。我们都知道要让widget重绘需要用到`setState`，所以在初始化两个动画的时候可不要忘记
添加监听器。
```dart
_animation = Tween<double>(begin: 0, end: 1).animate(_controller)
      ..addListener(() {       // *
        setState(() {});
      });
```

给Tween实例添加监听器，监听器在监听到之后执行的就是`setState`。

##### 给动画加点料--Curve

为了让动画更好看，我们可以给动画加点变化的曲线 -- curve。

上面的动画是线性变化的，显得很直接。这样的话对于用户来说太直接，观赏性不高。要改变这个问题就需要引入动画曲线。
动画曲线可以改变动画的线性变化，让动画有时快一点，有时慢一点，这样看起来更好。

要添加动画曲线一样非常简单。增加一个`CurvedAnimation`，它在初始化的时候需要用到动画控制器。然后把这个初始化好的曲线动画放在Tween的`animate()`方法里作为参数就可以了。

代码如下：

```diff
    _controller =
        AnimationController(duration: const Duration(seconds: 2), vsync: this);

+    final curvedAnimation =
+        CurvedAnimation(parent: _controller, curve: Curves.bounceInOut);

-    _animation = Tween<double>(begin: 0, end: 1).animate(_controller)
+    _animation = Tween<double>(begin: 0, end: 1).animate(curvedAnimation)
      ..addListener(() {
        setState(() {});
      });

-    _colorAnimation = ColorTween(begin: Colors.white, end: Colors.orange).animate(_controller)
+    _colorAnimation = ColorTween(begin: Colors.white, end: Colors.orange).animate(curvedAnimation)
      ..addListener(() {
        setState(() {});
      });
```

再来看使用曲线动画的例子。这次是使用一个曲线动画来实现弹簧的效果，模拟一个物理动画：

```dart
import 'package:flutter/material.dart';

class SpringAnimation extends StatefulWidget {
  const SpringAnimation({Key? key});

  @override
  _SpringAnimationState createState() => _SpringAnimationState();
}

class _SpringAnimationState extends State<SpringAnimation> with SingleTickerProviderStateMixin {
  late final AnimationController _controller = AnimationController(
    vsync: this,
    duration: const Duration(seconds: 3),
  )..forward();

  late final Animation<double> _animation = Tween<double>(
    begin: 0,
    end: 400,
  ).animate(CurvedAnimation(
    parent: _controller,
    curve: Curves.elasticOut,
  ));

  void _startAnimation() {
    _controller.reset();
    _controller.forward();
  }

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: AnimatedBuilder(
            animation: _animation,
            builder: (context, child) {
              return Stack(
                children: [
                  Positioned(
                    left: 70,
                    top: _animation.value,
                    child: child!,
                  )
                ],
              );
            },
            child: GestureDetector(
              onTap: () {
                _startAnimation();
              },
              child: Container(
                height: 100,
                width: 250,
                decoration: BoxDecoration(
                  color: const Color(0xFF00EF3C),
                  borderRadius: BorderRadius.circular(10),
                ),
                child: const Center(
                    child: Text(
                  'SEMAPHORE',
                  style: TextStyle(fontSize: 40, fontWeight: FontWeight.bold),
                )),
              ),
            ),
          ),
        ),
      ),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
}
```

在这个例子中，`SpringAnimation` widget 使用一个`AnimationController`来驱动动画对象。一个补间动画（Tween)定义了动画的值范围，一个带有`Curves.elasticOut` 的`CurveAnimation`为动画赋予了一个类似弹性的缓动曲线。`AnimatedBuilder` widget 被用来根据动画的值来为一个容器 widget 的位置进行动画处理。`Positioned` widget 确保容器 widget 在屏幕上的位置与动画的值相关。当 widget 被构建时，`AnimationController`启动动画对象的值的动画，从而推动容器 widget 的位置动画。



### 基于物理特性的动画

Flutter 提供了模拟类用于创建具有初始状态和演化规则的基于物理的模拟。这个类使你能够创建各种各样基于物理的动画，包括那些基于弹簧动力学、摩擦力和重力的动画。

基于物理的动画还可以不沿着固定的时间线运行，这样动画的控制可以更加的灵活。在这一类的动画中可以用到的类有：

- `SpringSimulation`，模拟弹簧的动画。需要在参数中指定劲度系数、阻尼和初始状态。
```dart
import 'package:flutter/physics.dart';

final SpringDescription spring = SpringDescription(
  mass: 1,
  stiffness: 100,
  damping: 1,
);

final SpringSimulation simulation = SpringSimulation(spring, 0, 1, 0);
```
- `GravitySimulation`，模拟物体在重力作用下的运动。
```dart
import 'package:flutter/physics.dart';

final GravitySimulation simulation = GravitySimulation(
  0.1, // acceleration
  0,   // starting point
  100, // end point
  0,   // initial velocity
);
```
- `FrictionSimulation`，模拟物体在运动中受摩擦力影响下的效果。
```dart
import 'package:flutter/physics.dart';

final FrictionSimulation simulation = FrictionSimulation(
  0.1, // friction coefficient
  0,   // initial position
  1,   // initial velocity
);
```

让我们考虑一个在 Flutter 中基于物理的动画的例子：弹簧动画。可以使用弹簧模拟类来创建这个动画，模拟一个阻尼简谐振荡器。以下是你如何使用弹簧模拟来产生一个类似弹簧的动画：

```dart
import 'package:flutter/material.dart';
import 'package:flutter/physics.dart';

class PhysicsAnimationPage extends StatefulWidget {
  const PhysicsAnimationPage({super.key});

  @override
  State<PhysicsAnimationPage> createState() => _PhysicsAnimationPageState();
}

class _PhysicsAnimationPageState extends State<PhysicsAnimationPage>
    with SingleTickerProviderStateMixin {
  late AnimationController _controller;

  @override
  void initState() {
    super.initState();

    _controller = AnimationController(vsync: this, upperBound: 500)
      ..addListener(() {
        setState(() {});
      });

    final simulation = SpringSimulation(
        const SpringDescription(mass: 1, stiffness: 10, damping: 1),
        0,
        300,
        10);

    _controller.animateWith(simulation);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
        appBar: AppBar(
          title: const Text("Physics Animation"),
        ),
        body: Stack(children: [
          Positioned(
            left: 50,
            top: _controller.value,
            height: 40,
            width: 40,
            child: Container(
              color: Colors.blue,
            ),
          ),
        ]));
  }

  @override
  void dispose() {
    _controller.dispose();

    super.dispose();
  }
}
```

结果是一个模拟弹簧的动画，在屏幕上产生一个弹跳效果。从本质上讲，基于物理的动画为你提供了一个强大的机制，用于在你的应用程序中创造逼真和流畅的运动，而 Flutter 提供了一系列的工具和类来促进它们的实现。

### 英雄动画

Flutter 中的 Hero widget是充当不同屏幕之间跳转实现动画的工具。例如，从todo列表页点击某个todo之后跳转到todo的详情页的时候，可以用Hero widget来实现。当然不只是todo文本还包括图像、甚至容器等各种元素。

Hero widget的一个关键点是在起始屏幕和目标屏幕上都需要相同的*标签（tag）*。这个标签在识别正在进行过渡动画至关重要。

下面来看看如何实现从todo列表点击某个todo之后跳转到todo详情页的过渡动画。

```dart
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context) {
    final homeController = Get.find<HomeController>();

    return Scaffold(
      appBar: AppBar(
        // 略
      ),
      body: Column(
        children: [
          // 略
          Expanded(
            flex: 1,
            child: GetBuilder<HomeController>(
                init: Get.find<HomeController>(),
                builder: (controller) => ListView.separated(
                      restorationId: 'sampleItemListView',
                      itemCount: controller.todoList.length,
                      itemBuilder: (BuildContext context, int index) {
                        final item = controller.todoList[index];
                        return Hero( // 1
                          tag: item.content,
                          child: Material(
                            type: MaterialType.transparency,
                            child: ListRow(
                                content: item.content,
                                navigateTo: () async {
                                  final result = await Navigator.of(context)
                                      .push(MaterialPageRoute(
                                    builder: (builder) {
                                      return HeroAnimationPage( // 2
                                        todoItem: item,
                                      );
                                    },
                                  ));
                                  // 略
                                }),
                          ),
                        );
                      },
                      separatorBuilder: (context, index) => const Divider(),
                    )),
          ),
          // 略
        ],
      ),
    );
  }

  // 略
}
```

解释如下：
1. `Hero`: Hero动画需要两个Hero widget，一个是源（source）Hero，一个是目标（destination）Hero。这里的是源Hero。
2. `HeroAnimationPage`：这是导航的路由目标页。在这个页面里包含了两个`Hero`的目标Hero。

#### Hero动画目标页

```dart
import 'package:flutter/material.dart';
import 'package:flutter/scheduler.dart';
import 'package:flutter_todo/src/models/todo_model.dart';

class HeroAnimationPage extends StatefulWidget {
  const HeroAnimationPage({super.key, required this.todoItem});

  final TodoItem todoItem;

  @override
  State<StatefulWidget> createState() => _HeroAnimationPageState();
}

class _HeroAnimationPageState extends State<HeroAnimationPage>
    with SingleTickerProviderStateMixin {
  @override
  Widget build(BuildContext context) {
    timeDilation = 3.0; // 1 

    final content = widget.todoItem.content;
    return Scaffold(
      appBar: AppBar(
        // 略
      ),
      body: Hero( // 2
        tag: content,
        child: Center(
          child: Text(
            content,
            style: Theme.of(context).textTheme.headlineSmall,
          ),
        ),
      ),
    );
  }
}
```

解析如下：
1. `tmeDilation`：用于调整动画的执行时间，这里延长动画时间可以更轻蹙的看到动画执行的过程
2. `Hero`：这里的`Hero`是目标Hero widget。Hero动画需要两个`Hero`，一个是源Hero，一个是目标Hero。

到这里，你对Hero动画已经有大概的了解了。要实现一个Hero动画需要
1. 两个**tag**一样的Hero widget。
2. 导航连接这里两个Hero widget所在的页面。
3. 导航的`push`到下一个页面或者从目标页面`pop`回退到上一个页面的时候出发Hero动画。
4. 在动画开始之后，flutter会自动给这个动画加一个`ReactTween`动画，显示在定义好的路由之上。也就是理由的两个页面之上。
·

### 隐性动画

隐式动画是在Flutter中生成响应widget属性变化的简单动画的非常方便的工具。隐式动画无需深入研究复杂的动画控制器和补间，使您能够为小部件的属性设置动画，而无需操心动画的复杂细节。隐式动画有对于单一属性的widget，也有多个属性同时改变的widget。先来看一个简单的修改`opacity`值的隐式动画widget：`AnimatedOpacity`。

```dart
import 'package:flutter/material.dart';

class ImplicitAnimationPage extends StatefulWidget {
  const ImplicitAnimationPage({super.key});

  @override
  State<StatefulWidget> createState() => _ImplicitAnimationPageState();
}

class _ImplicitAnimationPageState extends State<ImplicitAnimationPage> {
  double opacity = 0; // 1

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("Implicit Animation"),
        actions: [
          TextButton(
              onPressed: () {
                setState(() { // 2
                  opacity = 1;
                });
              },
              child: const Text("Start"))
        ],
      ),
      body: Container(
        color: Colors.yellow,
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            Align(
              alignment: Alignment.center,
              child: AnimatedOpacity(  // 3
                duration: const Duration(seconds: 2), // 4
                opacity: opacity,
                child: Container(
                    color: Colors.red,
                    child: const Text(
                      "Implicit Animation",
                      style: TextStyle(fontSize: 30),
                    )),
              ),
            )
          ],
        ),
      ),
    );
  }
}
```

解释如下：
1. 这个需要在一个`StatefulWidget`里使用，动画组件会监听属性值状态的变化。
2. `setState`给`opacity`赋新的值开始动画。
3. 使用`AnimatedOpacity`组件来执行动画。还有其他的类似的使用`Animated`为前缀的动画组件。
4. 动画组件需要`duration`属性的值来决定动画的执行时间。

`AnimatedOpacity`是对于一个属性实现动画的动画组件。对于多个属性同时动画的组件叫做`AnimatedContainer`。 类似于一个标准容器，但它具有额外的动画能力。它对诸如大小、颜色和形状等属性的变化进行动画处理。让我们来探讨一个示例，说明使用 AnimatedContainer 小部件在按下按钮时对颜色变化进行动画处理。


#### 隐性动画示例

```dart
import 'dart:math';

import 'package:flutter/material.dart';
import 'package:flutter/widgets.dart';

double randomBorderRadius() {
  return Random().nextDouble() * 64;
}

double randomMargin() {
  return Random().nextDouble() * 64;
}

Color randomColor() {
  return Color(0xFFFFFFFF & Random().nextInt(0xFFFFFFFF));
}

class ImplicitAnimationPage extends StatefulWidget {
  const ImplicitAnimationPage({super.key});

  @override
  State<StatefulWidget> createState() => _ImplicitAnimationPageState();
}

class _ImplicitAnimationPageState extends State<ImplicitAnimationPage> {
  double opacity = 0;
  //
  Color color = Colors.white;
  double radius = 0;
  double margin = 0;

  @override
  void initState() {
    super.initState();

    color = randomColor();
    radius = randomBorderRadius();
    margin = randomMargin();
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text("Implicit Animation"),
        actions: [
          TextButton(
              onPressed: () {
                setState(() {
                  opacity = 1;
                });
              },
              child: const Text("Start 1")),
          TextButton(
              onPressed: () {
                setState(() {
                  color = randomColor();
                  radius = randomBorderRadius();
                  margin = randomMargin();
                });
              },
              child: const Text("Start 2"))
        ],
      ),
      body: Container(
        color: Colors.yellow,
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          crossAxisAlignment: CrossAxisAlignment.stretch,
          children: [
            Align(
              alignment: Alignment.center,
              child: AnimatedOpacity(
                duration: const Duration(seconds: 2),
                opacity: opacity,
                child: AnimatedContainer( // 1
                    duration: const Duration(milliseconds: 500),
                    margin: EdgeInsets.all(margin),
                    decoration: BoxDecoration(
                        color: color,
                        borderRadius: BorderRadius.circular(radius)),
                    child: const Text(
                      "Implicit Animation",
                      style: TextStyle(fontSize: 30),
                    )),
              ),
            )
          ],
        ),
      ),
    );
  }
}
```

在这个例子中，`AnimatedContainer`会有圆角、背景色和`margin`三个属性的变化。在这里增加了三个随机值生成的方法，对于上述三个值每次点击按钮之后给`setState`三个随机的值。每次点击按钮都会看到不同的变化。

隐式动画是一种将动画引入到你的 Flutter 应用程序的用户友好型方法，无需动画控制器和补间的复杂性。

### 总结一下

动画在移动应用开发中起着关键作用，通过为应用注入活力和吸引力来增强用户体验。本文深入探讨了 Flutter 的动画能力，从其动画系统和基本类到基于物理的动画、自定义动画以及英雄动画过渡等动画技术。我们还讨论了隐式动画。

通过利用这些资源，你可以进一步深入高级 Flutter 动画的世界，打造更加符合用户体验的动态的用户界面，使你的移动应用脱颖而出。