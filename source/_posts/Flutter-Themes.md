---
title: Flutter -- Themes
date: 2024-06-28 00:42:52
tags:
---

Flutter开发者经常会遇到的问题之一就是如何处理好看并且各个界面都一致的Flutter主题。主题主要关系到设计的问题，这些主要使用了Material或者Cupertino设计元素，本文主要几种在Material Design 3.本文会详细介绍如何在Flutter应用中新建、定制和应用主题。

## 主题

### 了解Flutter的Material设计主题

Material Design 3是Google为开发者开发app、网站而提供的设计系统。在Flutter中这些设计一键值对的形式存在，这些键值对的组合最后体现在widget的外观上。存储这些外观键值对集合的叫做`ThemeData`。有必要对`ThemeData`做一些深入的了解。

### ThemeData

ThemeData类封装了Material Design主题的颜色、排版和形状属性。我们通常将其作为MaterialApp widget的参数，然后它将主题应用到所有子widget上。

### 新建一个主题

这次新建一个`ThemeData`，并给它的属性赋值，或者修改某些属性的值修改默认主题。然后把这个实例传递给`MaterialApp` widget里，看看app会发生什么变化。也可以在dartpad上测试，修改部分属性的值即可。确保`useMaterial`为`true`，这样Flutter就会使用Material Design的最新版本了，也就是V3.

```dart
ThemeData lightTheme = ThemeData(
  brightness: Brightness.light,
  useMaterial3: true,
  textTheme: const TextTheme(
    displayLarge: TextStyle(fontSize: 32, fontWeight: FontWeight.bold),
    bodyLarge: TextStyle(fontSize: 18, color: Colors.black87),
  ),
  appBarTheme: const AppBarTheme(
    color: Colors.blue,
    iconTheme: IconThemeData(color: Colors.white),
  ),
  colorScheme: ColorScheme.fromSeed(seedColor: Colors.blue),
);
```
### 应用ThemeData实例

在App中使用刚刚定义的主题：
```dart
MaterialApp(
  title: 'Custom Theme Demo',
  theme: lightTheme,
  home: MyHomePage(),
);
```

### 使用主题属性

用`Theme.of(context)`来使用主题的属性。这里有一个例子来展示如何使用主色调和文本主题：
```dart
Text(
  'Hello, Flutter!',
  style: Theme.of(context).textTheme.headlineMedium,
);
```

### 深色和浅色主题

在Flutter里可以分别定义浅色和深色的主题。你可以在`MaterialApp`里指定`dartTheme`深色主题。记得把`ColorScheme.brightness`的属性值设置为`Brightness.dark`。这样Flutter才知道是深色主题。

```dart
ThemeData darkTheme = ThemeData(
  textTheme: const TextTheme(
    displayLarge: TextStyle(fontSize: 32, fontWeight: FontWeight.bold),
    bodyLarge: TextStyle(fontSize: 18, color: Colors.white70),
  ),
  appBarTheme: const AppBarTheme(
    color: Colors.red,
    iconTheme: IconThemeData(color: Colors.white),
  ),
  colorScheme: ColorScheme.fromSeed(
    seedColor: Colors.red,
    brightness: Brightness.dark,
  ),
);
```

### 应用深色主题

在下面的例子中，Flutter会有两个主题，一个浅色和一个深色的。Flutter会根据设备的设置自动切换浅色和深色主题：
```dart
MaterialApp(
  title: 'Custom Theme Demo',
  theme: lightTheme,
  darkTheme: darkTheme,
  home: MyHomePage(),
);
```

Flutter也可以允许用户手动设定浅色和深色主题。如果手动指定了，Flutter的应用会忽略你的设备的设置转而使用指定的主题：
```diff
MaterialApp(
  title: 'Custom Theme Demo',
  theme: lightTheme,
  darkTheme: darkTheme,
+  themeMode: ThemeMode.light,
  home: MyHomePage(),
); 
```

## 颜色

Flutter主题对颜色尤为关注，这些都体现在了Flutter app的默认颜色中。

### 什么是颜色模式

Flutter app可以使用的颜色都定义在`ColorScheme`了。这些颜色都是连贯的一组颜色，并且是遵守了Material Design的视觉语言。这些颜色包含了主题色、次主题色、表面色、背景色，错误色等很多颜色。

### Material Design 3里的默认颜色

`ColorScheme`的主要目的就是定义了一组默认的颜色。这样你创建的app的各种widget就会使用这些默认的颜色。在Material Design 3中，也一样依赖于这些定义在`ColorScheme`的各种颜色作为默认颜色。

在定义`ThemeData`的时候，可以使用`ColorScheme.fromSeed`来定义自定义的一些颜色。当然，也可以手动的指定每一个颜色的色值。下面的一个例子演示了使用`ColorScheme.fromSeed`定义颜色：
```dart
ThemeData(
  useMaterial3: true,
  colorScheme: ColorScheme.fromSeed(
    seedColor: Colors.blue,
  ),
)
```
### 了解Widget如何获得默认颜色

Widget从`ColorScheme`中获取它们的默认颜色。每个widget都有一组特定的属性，定义了它使用的各种颜色。例如，`TextButton`从`ColorScheme.primary`获取前景文本颜色。在这个例子中，按钮的文本显示为绿色。

```dart
import 'package:flutter/material.dart';

void main() => runApp(
      MaterialApp(
        theme: ThemeData(
          brightness: Brightness.light,
          useMaterial3: true,
          colorScheme: ColorScheme.fromSeed(
            seedColor: Colors.blue,
          ).copyWith(
            primary: Colors.green,
          ),
        ),
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          body: TextButton(
            onPressed: () {},
            child: const Text('test'),
          ),
        ),
      ),
    );
```

不过，并没有一个通用的方法来确定widget用的是`ColorScheme`的那个属性。因为他们的默认颜色用的是不同的颜色。因为不同的widget是使用的可能是不同的`ColorScheme`属性作为他们的默认值。每个不同个例需要分别对待，要有深入的了解只能是翻看Flutter的widget的代码或者查阅文档。

只不过文档也并没有那么详细，所以要找到答案最好的办法还是翻看widget的代码。不管你用的是vs code还是Android Studio只需要cmd+click（windows上ctrl+click）就可以查看一个widget的代码了。

按钮有一个方法可以返回`ButtonStyle`，这个方法是：`defaultStyleOf`。下面的代码就是按钮的源码：
```dart
@override
ButtonStyle defaultStyleOf(BuildContext context) {
final ThemeData theme = Theme.of(context);
final ColorScheme colorScheme = theme.colorScheme;

return Theme.of(context).useMaterial3
    ? _TextButtonDefaultsM3(context)
    : styleFrom(
    // [This code is irrelevant for Material Design 3]
    );
}
```

下面一个例子是`_TextButtonDefaultM3`的源码。多数的widget都有类似的后缀：`DefaultM3`。这些代码表明了这个widget在可用的时候是如何从`ColorScheme`获取前景色的。

```dart
class _TextButtonDefaultsM3 extends ButtonStyle {
  _TextButtonDefaultsM3(this.context)
   : super(
       animationDuration: kThemeChangeDuration,
       enableFeedback: true,
       alignment: Alignment.center,
     );

  final BuildContext context;
  late final ColorScheme _colors = Theme.of(context).colorScheme;

  // ...

  @override
  MaterialStateProperty<Color?>? get foregroundColor =>
    MaterialStateProperty.resolveWith((Set<MaterialState> states) {
      if (states.contains(MaterialState.disabled)) {
        return _colors.onSurface.withOpacity(0.38);
      }
      return _colors.primary;
    });

  // ...
}
```

看完上面的代码就可以体会到，如果用好了`ColorScheme`就自动遵循了Material Design 3的设计语言，并且获得了一致的颜色系统。当然，你也可以在某几个widget中覆盖了默认颜色。但是，这样一来你的app的颜色就不那么一致了。

### 覆盖默认颜色

在Material Design 3中，`ElevatedButton`，`OutlinedButton`和`TextButton`都从`ColorScheme`中获得了默认色值。要修改他们的颜色，可以在某个button widget的定义中修改它的颜色值。

比如你可以覆盖`ElevatedButton`的背景色。这个颜色需要转化为`MaterialStateProperty<Color?>`类型。这是因为按钮的背景色可以根据按钮的不同状态改变。下面这个例子让按钮的背景色无论在什么状态下都是红色的。

```dart
ThemeData theme = ThemeData(
  elevatedButtonTheme: ElevatedButtonThemeData(
      style: ButtonStyle(
    backgroundColor: MaterialStateProperty.resolveWith<Color?>(
      (Set<MaterialState> states) => Colors.red,
    ),
  )),
  useMaterial3: true,
);
```

注意：在主题中定义按钮的`buttonStyle`基本是不会有用的：
```dart
import 'package:flutter/material.dart';

void main() => runApp(
      MaterialApp(
        theme: ThemeData(
          buttonTheme: const ButtonThemeData(buttonColor: Colors.red),
          useMaterial3: true,
        ),
        debugShowCheckedModeBanner: false,
        home: Scaffold(
          body: ElevatedButton(
            onPressed: () {},
            child: const Text('test'),
          ),
        ),
      ),
    );
```



## 文本

### 使用TextStyle修改文本

Material Design 3的另外一个很重要的部分就是排版。同样的也可以在`ThemeData`中定制app的排版。这包括定制字体大小、粗细和颜色。`ThemeData`的`textTheme`属性包含一个`TextTheme`对象，它包含了一组预定义的`TextStyle`属性。这里写属性代表了不同的文本样式，比如标题、正文等。只需要修改`TextStyle`属性的值就可以定制你想要的排版样式了。

```dart
ThemeData lightTheme = ThemeData(
  textTheme: const TextTheme(
    displayLarge: TextStyle(
        fontSize: 96, fontWeight: FontWeight.w300, color: Colors.black),
    displayMedium: TextStyle(
        fontSize: 60, fontWeight: FontWeight.w400, color: Colors.black),
    displaySmall: TextStyle(
        fontSize: 48, fontWeight: FontWeight.w400, color: Colors.black),
    headlineMedium: TextStyle(
        fontSize: 34, fontWeight: FontWeight.w400, color: Colors.black),
    headlineSmall: TextStyle(
        fontSize: 24, fontWeight: FontWeight.w400, color: Colors.black),
    titleLarge: TextStyle(
        fontSize: 20, fontWeight: FontWeight.w500, color: Colors.black),
    bodyLarge: TextStyle(
        fontSize: 16, fontWeight: FontWeight.w400, color: Colors.black87),
    bodyMedium: TextStyle(
        fontSize: 14, fontWeight: FontWeight.w400, color: Colors.black87),
    bodySmall: TextStyle(
        fontSize: 12, fontWeight: FontWeight.w400, color: Colors.black54),
    labelLarge: TextStyle(
        fontSize: 14, fontWeight: FontWeight.w500, color: Colors.white),
  ),
);
```

要使用定义好的这些排版值只需要读取`Theme.of(context).textTheme`属性的值。文本widget的`TextStyle`的默认值是`bodyMedium`。下面这个例子怎么定制文本样式和Text widget如何使用默认样式，和命名样式。

```dart
import 'package:flutter/material.dart';

void main() => runApp(
    MaterialApp(
    theme: ThemeData(
        fontFamily: 'Roboto',
        useMaterial3: true,
        textTheme: const TextTheme(
        bodyMedium: TextStyle(
            color: Colors.red,
            fontSize: 30,
            fontWeight: FontWeight.bold,
        ),
        bodySmall: TextStyle(
            color: Colors.purple,
            fontSize: 10,
        ),
        ),
    ),
    debugShowCheckedModeBanner: false,
    home: Builder(
        builder: (context) => Scaffold(
        body: Center(
            child: Column(
            mainAxisAlignment: MainAxisAlignment.center,
            children: [
                const Text('Default Text Style'),
                Text('Body Small',
                    style: Theme.of(context).textTheme.bodySmall),
            ],
            ),
        ),
        ),
    ),
    ),
);
```

## 形状

### 主题形状

在主题这个级别定义形状，可以让你的app所有用到这个形状的地方都报纸一一致。当然都定义在主题这个级别了肯定就是为了一致才这么做。形状用好了可以很好的增加用户界面的视觉层次和结构，给用户提供更好的用户体验。

在主题中定义形状，请看下面的实例代码：
```dart
ThemeData(
  useMaterial3: true,
  cardTheme: CardTheme(
    shape: RoundedRectangleBorder(
      borderRadius: BorderRadius.circular(4.0),
    ),
  ),
)
```

这个主题定义了widget的默认形状。
```dart
Card(
  shape: Theme.of(context).cardTheme.shape,
  child: const SizedBox(
    width: 100,
    height: 100,
  ),
),
```

## 完结

想要使用Flutter开发出完美的app，不深入的了解它的主题肯定是不行的。本文可以做为你深入了解主题的一个大概的概括。之后可以通过查看官方文档或者代码深入的了解Flutter主题是如何运作的。