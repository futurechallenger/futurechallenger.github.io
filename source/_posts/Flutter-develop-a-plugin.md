---
title: Flutter -- develop a plugin
date: 2024-06-26 01:43:04
tags:
---

开发Flutter的Plugin。 代码在[这里](https://github.com/futurechallenger/calendar_plugin)。

新建一个plugin项目*calendar\_plugin*：

```shell
$ flutter create --template=plugin --platforms=android,ios calendar_plugin
```

平台指定为Android和iOS，稍后再加一个平台*macOS*来看看这个流程可以如何操作。后续可以的话再尝试添加windows和web。

默认的iOS使用的是swift，Android使用的是kotlin，如果需要换objc或者java可以使用`arguments -i objc -a java`换到你想要的语言。

生成的项目结构：

*   android: Android的原生代码 (Kotlin)。
*   example: 一个Flutter的实例项目，用来展示、测试你开发的plugin的。
*   ios: iOS本地代码(Swift)。
*   lib: Plugin的Dart代码。
*   test: 测试Plugin。
*   CHANGELOG.md: 一个markdown文件，说明发布版本中包含的修改的文档。
*   pubspec.yaml: 它包含了你的Plugin需要满足的环境等的信息。
*   README.md: 给使用这个Plugin的开发者看的帮助文档。

### 实现Dart代码

在生成的*lib*目录下生成了两个文件：

*   `calendar_plugin_method_channel.dart`
*   `calendar_plugin_platform_interface.dart`
*   `calendar_plugin.dart`

在interface文件里定义一个方法。在channel文件里实现这个方法，这个方法也是负责和原生代码通信的。最后在plugin里暴露这个方法，这个方法就会在其他项目里引入这个plugin的时候找到这个方法。

1.  添加给calendar增加事件的方法：

```dart
  Future<String?> addEventToCalendar(String eventName) {
    throw UnimplementedError();
  }
```

2.  实现这个方法：

```dart
  @override
  Future<String?> addEventToCalendar(String eventName) {
    return methodChannel.invokeMethod<void>('addEventToCalendar');
  }
```

3.  在plugin文件里定义这个方法，暴露给调用方。

```dart
  Future<String?> addEventToCalendar(String eventName) {
    return CalendarPluginPlatform.instance.addEventToCalendar(eventName);
  }
```

以上就是需要在dart这里处理的代码，是不是很简单。

现在添加它们的相关测试。

在*test*目录下的*calendar\_plugin\_test.dart*文件，其实已经添加好了。有一个示例测试，专门给一个返回系统版本的方法：`getPlatformVersion`生成好了：

```dart
  test('getPlatformVersion', () async {
    CalendarPlugin calendarPlugin = CalendarPlugin();
    MockCalendarPluginPlatform fakePlatform = MockCalendarPluginPlatform();
    CalendarPluginPlatform.instance = fakePlatform;

    expect(await calendarPlugin.getPlatformVersion(), '42');
  });
```

在正式开始前还得修改一下platform模拟类。

```dart
class MockCalendarPluginPlatform
    with MockPlatformInterfaceMixin
    implements CalendarPluginPlatform {
  @override
  Future<String?> getPlatformVersion() => Future.value('42');

  @override
  Future<String?> addEventToCalendar(String eventName) { // 1
    return Future.value(eventName);
  }
}
```

在这里添加`addEventToCalendar`模拟方法。

在下面增加一个测试给日历加事件的方法：

```dart
  test('addEventToCalendar', () async {
    CalendarPlugin calendarPlugin = CalendarPlugin();
    MockCalendarPluginPlatform fakePlatform = MockCalendarPluginPlatform();
    CalendarPluginPlatform.instance = fakePlatform;

    expect(
        await calendarPlugin.addEventToCalendar('hello world'), 'hello world');  // 2
  });
```

如果成功的给日历添加了事件，那么就返回日历的文字内容。测试就对比这返回的值就好。

### 实现iOS部分

首先需要配置你的example的Xcode项目，否则打开报错。执行这个命令：

```shell
cd hello/example; flutter build ios --no-codesign --config-only
```

之后使用xcode打开你的*example*项目。

在一个遥远的地方你可以找到需要编辑的插件的swift文件。

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/57f0e47d8923498896d0577b1de54890~tplv-k3u1fbpfcp-jj-mark:0:0:0:0:q75.image#?w=960&h=680&s=63927&e=png&b=dfdfde)

打开插件的swift文件*CalendarPlugin.swift*，你会看到已经生成好的文件：

```swift
import Flutter
import UIKit

public class CalendarPlugin: NSObject, FlutterPlugin {
  public static func register(with registrar: FlutterPluginRegistrar) {
    let channel = FlutterMethodChannel(name: "calendar_plugin", binaryMessenger: registrar.messenger())
    let instance = CalendarPlugin()
    registrar.addMethodCallDelegate(instance, channel: channel)
  }

  public func handle(_ call: FlutterMethodCall, result: @escaping FlutterResult) {
    switch call.method {
    case "getPlatformVersion":  // *
      result("iOS " + UIDevice.current.systemVersion)
        
    default:
      result(FlutterMethodNotImplemented)
    }
  }
}
```

就是在注释的\*这里，可以看到已经为获得系统版本号生成好了代码。类似的，在`handle`方法里添加新增的给calendar添加事件的代码：

```swift
case "addEventToCalendar":  // *
  return null;
```

在这个case分支里，就是需要实际在iOS里添加事件的代码。这些不必过多关注。需要关注的是如何处理异常情况。比如没有传入事件的title、note或者开始、结束日期等。

```dart
  case "addEventToCalendar":
    if call.arguments == nil {
      result(FlutterError(code: "Invalid parameters", message: "Invalid parameters", details: nil))
      return
    }
```

`code`和`message`都是字符串类型的，按照项目的代码规定写就可以。details是`Any?`类型的，按照项目需要想放什么都随意。

按照在前面写的测试，如果成功添加了一个事件则返回这个时间的title。

```dart
result(title)
```

### 实现Android部分

首先，需要对日历的写入权限

```xml
<uses-permission android:name="android.permission.WRITE_CALENDAR"/>
```

在`CalendarPlugin.kt`文件里有一个`onMethodCall`方法。还是老样子存在一个已经实现好的获取系统版本的方法。

```kotlin
  override fun onMethodCall(call: MethodCall, result: Result) {
    if (call.method == "getPlatformVersion") {
      result.success("Android ${android.os.Build.VERSION.RELEASE}")
    } else {
      result.notImplemented()
    }
  }
```

一样是在判断方法名，所以我们可以依葫芦画瓢加一个分支。

```kotlin
  if (call.method == "getPlatformVersion") {
    // 略
  }
```

### 测试开发好的插件

在\*example\`项目里调用刚刚开发好了的插件。

在这个项目的*lib*目录下只有一个*main.dart*文件负责调用插件。修改这个文件：

```dart
// 增加一个按钮和一个Text显示返回的事件title
  Center(
    child: Text('Added event $_eventName\n'), // 1
  ),
  ElevatedButton(
      onPressed: () {
        Permission.calendarFullAccess.request().then((status) { // 2
          if (status.isGranted) {
            debugPrint("calendar full access is granted");
            _calendarPlugin
                .addEventToCalendar("hello", "hello world")
                .then((value) {
              debugPrint("ret is $value");
              setState(() { // 3
                _eventName = value ?? '';
              });
            });
          } else {
            debugPrint("calendar full access is denied");
          }
        });
      },
      child: const Text("OK")
  ),
```

1.  一个`Text`显示返回的事件的title
2.  按钮，在点击事件中首先弹出请求用户的日历权限。这里使用的是[`permission_handler`](https://pub.dev/packages/permission_handler)。具体的安装和配置方式可以参考permission handler的官方文档。在很多时候Android和iOS都需要处理runtime权限，这个库非常有用。
3.  在成功的返回了事件title之后setState，这样这个title才会显示出来。

运行之后就会在日历中找到添加进去的事件。不过在android上需要先登录账户，所以没有成功添加。

### 在另外一个项目中使用Plugin

Plugin开发好了肯定不是在`example`目录跑一下就完成了。在正式的可以运行的项目里跑起来才对。

直接在*pubspec.yaml*里添加依赖：
```yaml
dependencies:
  # 略
  calendar_plugin:
    git:
      url: https://github.com/futurechallenger/calendar_plugin.git
      ref: main
```

之后执行：
```bash
flutter pub get
```

之后就可以在项目里使用这个Plugin了。

### 发布你的Plugin

开发出一款优秀的Plugin之后就可以给其他的开发者用了。这需要发布到pub.dev上。代码已经测试过，检查文档完整，详细的说明了如何使用这个Plugin之后就可以开始发布了。

首先使用dry-run模式分析一下Plugin看有没有什么其他问题：
```bash
flutter pub publish --dry-run
``` 

然后就可以发布到pub.dev上了：
```bash
flutter pub publish
```

### 最后

开发plugin是在处理Flutter开发中难免遇到的，如果没有开源的库就只能自己开发了。Plugin主要处理有一定的代码量的时候，如果只是比较简单的和native互操作的话可以有更加简单的处理方式。

Web和桌面app的开发中也会遇到互操作的问题。稍后会讲到。
