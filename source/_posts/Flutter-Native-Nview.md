---
title: Flutter -- Native Nview
date: 2024-08-29 20:38:52
tags:
---

为什么要用原生视图。一种是不得不用，一种是非用不可。基本上就是废话了。但是，这样的废话也说明一个问题，Fluter费劲支持这个功能是有必要的。比如在参加显示地图、WebView之类的，还有原生支持了某些功能，而Flutter要支持还要等待些许时日的。还有就是某些功能在原生视图上都实现好了，直接拿来用也很正常。

对于原生视图的支持，本文分开Android和iOS两边来说。要实现Flutter显示原生视图，在细节上有很多的不同，但是大体上分三步：

*   在Flutter有一个专门用来包装原生视图的widget
*   在原生代码需要有一个实现了platform view的原生视图，和一个返回这个原生视图的工厂
*   在原生代码中注册这个原生视图工厂

而贯穿于这三部的就是叫做`viewType`的字符串。原生的`viewType`和Flutter的如果不同的话是会报错的。App直接Crash的那种！

### Android

Android原生视图在Flutter中显示是有两种显示模式，各有优缺点。主要是在性能上有些取舍，也有最低支持的Android版本的不同。这两种模式分别是：

*   Hybrid Composition

    *需要主要的是Flutter的某些**Transformation**在Android视图上不支持。*
*   Texture Layer（或者Texture Layer Hybrid Composition）

    *在这个模式之下，“快速滚动”会变得容易**卡顿**。文本放大器只有在`TextureView`上才可用*

#### 首先是**Hybrid Composition**：

还记得我们开始提到的现实原生视图的三步么，首先处理Flutter端。先新建一个widget来显示原生视图：

```dart
import 'package:flutter/foundation.dart';
import 'package:flutter/gestures.dart';
import 'package:flutter/material.dart';
import 'package:flutter/rendering.dart';
import 'package:flutter/services.dart';

class NativeExampleView extends StatelessWidget {
  const NativeExampleView({super.key});

  @override
  Widget build(BuildContext context) {
    const String viewType = '@views/native_example_view';
    const Map<String, dynamic> creationParams = <String, dynamic>{};

    return PlatformViewLink(
      viewType: viewType,
      surfaceFactory: (context, controller) {
        return AndroidViewSurface(
            controller: controller as AndroidViewController,
            hitTestBehavior: PlatformViewHitTestBehavior.opaque,
            gestureRecognizers: const <Factory<
                OneSequenceGestureRecognizer>>{});
      },
      onCreatePlatformView: (params) {
        return PlatformViewsService.initSurfaceAndroidView(
          id: params.id,
          viewType: viewType,
          layoutDirection: TextDirection.ltr,
          creationParams: creationParams,
          creationParamsCodec: const StandardMessageCodec(),
          onFocus: () {
            params.onFocusChanged(true);
          },
        )
          ..addOnPlatformViewCreatedListener(params.onPlatformViewCreated)
          ..create();
      },
    );
  }
}
```

在Android端，新建两个文件（推荐使用Android Studio）：

*   一个原生视图：`NativeView`，名字不要紧。重要的是有一个继承自`PlatformView`的原生视图。只有这个视图才能显示在Flutter上。
*   然后需要建一个返回原生视图的工厂。一个继承自`PlatformViewFactory`的类。它返回Android原生视图。

分别实现这两个kotlin类：

NativeView的实现：

```kotlin
package com.example.flutter_todo

import android.content.Context
import android.graphics.Color
import android.view.View
import android.widget.TextView
import io.flutter.plugin.platform.PlatformView

class NativeView(context: Context, id: Int, creationParams: Map<String?,Any?>?) :PlatformView {
    private val textView: TextView

    override fun getView(): View {
        return textView
    }

    override fun dispose() {}

    init {
        textView = TextView(context)
        textView.textSize=72f
        textView.setBackgroundColor(Color.rgb(255,255,255))
        textView.text="Rendered on a native Android view (id: $id)"
    }
}
```

原生视图工厂的实现：

```kotlin
package com.example.flutter_todo

import android.content.Context
import io.flutter.plugin.common.StandardMessageCodec
import io.flutter.plugin.platform.PlatformView
import io.flutter.plugin.platform.PlatformViewFactory

class NativeViewFactory : PlatformViewFactory(StandardMessageCodec.INSTANCE) {
    override fun create(context: Context, viewId: Int, args: Any?): PlatformView {
        val creationParams = args as Map<String?, Any?>?
        return NativeView(context, viewId, creationParams)
    }
}
```

最后在`MainActivity`里注册这个原生视图工厂。

```kotlin
package com.example.flutter_todo

import io.flutter.embedding.android.FlutterActivity
import io.flutter.embedding.engine.FlutterEngine
import java.lang.annotation.Native

class MainActivity: FlutterActivity() {
    override fun configureFlutterEngine(flutterEngine: FlutterEngine) {
        super.configureFlutterEngine(flutterEngine)

        flutterEngine
            .platformViewsController
            .registry
            .registerViewFactory("@views/native_example_view",
                NativeViewFactory())
    }
}
```

这个实例非常简单。需要注意的就是`viewType`在Flutter和Android上必须要一致！

运行效果：

<img src="https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/53520b7586614883b09019f0794ff376~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5bCP57qi5pif6Zeq5ZWK6Zeq:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNTAxMDMzMDMxNzE2NzI4In0%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1725541737&x-orig-sign=d2U5foFxQ2g8e%2BwO1ddkMy5fsHs%3D" width="400" alt="Android运行效果">

#### Texture Layer

使用**Texture Layer**这个模式只需要修改Flutter部分的代码，如下：

```dart
import 'package:flutter/material.dart';
import 'package:flutter/services.dart';

class NativeTextureLayerView extends StatelessWidget {
  const NativeTextureLayerView({super.key});

  @override
  Widget build(BuildContext context) {
    const String viewType = "@views/native_example_view";
    final Map<String, dynamic> creationParams = <String, dynamic>{};

    return AndroidView(
        viewType: viewType,
        layoutDirection: TextDirection.ltr,
        creationParams: creationParams,
        creationParamsCodec: const StandardMessageCodec());
  }
}
```

至少看起来是比使用**Hybrid Composition**模式代码上少了一些。

这个模式和上一个模式使用的原生视图代码是一样的。把`NativeTextureLayerView`放在页面里展示出来就可以了。效果如图：

<img src="https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/a8848fc43bcf41888e8d7216193fe8b1~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5bCP57qi5pif6Zeq5ZWK6Zeq:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNTAxMDMzMDMxNzE2NzI4In0%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1725541737&x-orig-sign=i3YW%2BYQ2Dupwf4CRktjUn0B%2B%2F20%3D" width="400" alt="Texture Layer运行效果">

这一套技术的两个模式都hold不住`SurfaceView`，尽量不要用。

### iOS

这里说句废话，编辑然后运行iOS代码需要**Xcode**。

iOS没有两个个模式需要处理。只有一个**Hybrid Composition**。

在修改中如果遇到：*没有Flutter这个module*，的报错的话可以**暂时**先忽略。直接在VS Code或者其他编辑Flutter代码的编辑器或者IDE运行就可以。

#### 在Flutter需要处理的

这里只需要稍微修改一下之前在处理Android的时候使用的Flutter代码即可。根据平台判断，显示Android或者iOS。

```dart
  const String viewType = '@views/native_example_view';
  const Map<String, dynamic> creationParams = <String, dynamic>{};
  
  return const UiKitView(
      viewType: viewType,
      layoutDirection: TextDirection.ltr,
      creationParams: creationParams,
      creationParamsCodec: StandardMessageCodec());
```

和Android一样，接着来处理原生视图和原生视图的Factory。

原生视图的部分：

```swift
import Foundation

class FLNariveView: NSObject, FlutterPlatformView {
  private var _view: UIView
  
  init(
    frame: CGRect,
    viewIdentitifer viewId: Int64,
    arguments args: Any?,
    binaryMessenger messenger: FlutterBinaryMessenger?
  ){
    _view = UIView()
    super.init()
    
    createNativeView(view: _view)
  }
  
  func view() -> UIView {
    return _view
  }
  
  func createNativeView(view _view: UIView) {
    _view.backgroundColor = UIColor.blue
    let nativeLabel = UILabel()
    nativeLabel.text = "Native text from iOS"
    nativeLabel.textColor = UIColor.white
    nativeLabel.textAlignment = .center
    nativeLabel.frame = CGRect(x:0, y:0, width: 180, height: 48.0)
    _view.addSubview(nativeLabel)
  }
}
```

iOS原生视图的Factory：

```swift
import Foundation
import Flutter

class FLNativeViewFactory: NSObject, FlutterPlatformViewFactory {
  private var messenger: FlutterBinaryMessenger
  
  init(messenger: FlutterBinaryMessenger) {
    self.messenger = messenger
    super.init()
  }
  
  func create(withFrame frame: CGRect,
              viewIdentifier viewId: Int64,
              arguments args: Any?) -> FlutterPlatformView {
    return FLNariveView(frame: frame, viewIdentitifer: viewId, arguments: args, binaryMessenger: messenger)
  }
}
```

注册原生视图

```swift
import UIKit
import Flutter

@main
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    GeneratedPluginRegistrant.register(with: self)
    
    guard let pluginRegistrar = self.registrar(forPlugin: "plugin-name") else { return false }
    
    let factory = FLNativeViewFactory(messenger: pluginRegistrar.messenger())
    pluginRegistrar.register(
      factory,
      withId: "@views/native_example_view")
    
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

运行效果：

<img src="https://p0-xtjj-private.juejin.cn/tos-cn-i-73owjymdk6/ca130da628e34790acb14f78700ee2da~tplv-73owjymdk6-jj-mark-v1:0:0:0:0:5o6Y6YeR5oqA5pyv56S-5Yy6IEAg5bCP57qi5pif6Zeq5ZWK6Zeq:q75.awebp?policy=eyJ2bSI6MywidWlkIjoiNTAxMDMzMDMxNzE2NzI4In0%3D&rk3s=f64ab15b&x-orig-authkey=f32326d3454f2ac7e96d3d06cdbb035152127018&x-orig-expires=1725541737&x-orig-sign=IJx8LgHHf89iI%2FMAoffnLv9jWLk%3D" alt="iOS运行原生视图" width="300">

### Android和iOS一起处理

Flutter显示原生视图可以支持Android、iOS和macOS。其他的平台还支持不到。

在Android的*Hybrid Composition*的代码里加上处理iOS的代码：

```dart
import 'dart:io' show Platform;

// 其他的import略。。

class NativeExampleView extends StatelessWidget {
  const NativeExampleView({super.key});

  @override
  Widget build(BuildContext context) {
    const String viewType = '@views/native_example_view';
    const Map<String, dynamic> creationParams = <String, dynamic>{};

    if (Platform.isAndroid) {
      return PlatformViewLink(
        viewType: viewType,
        surfaceFactory: (context, controller) {
          return AndroidViewSurface(
              controller: controller as AndroidViewController,
              hitTestBehavior: PlatformViewHitTestBehavior.opaque,
              gestureRecognizers: const <Factory<
                  OneSequenceGestureRecognizer>>{});
        },
        onCreatePlatformView: (params) {
          return PlatformViewsService.initSurfaceAndroidView(
            id: params.id,
            viewType: viewType,
            layoutDirection: TextDirection.ltr,
            creationParams: creationParams,
            creationParamsCodec: const StandardMessageCodec(),
            onFocus: () {
              params.onFocusChanged(true);
            },
          )
            ..addOnPlatformViewCreatedListener(params.onPlatformViewCreated)
            ..create();
        },
      );
    } else if (Platform.isIOS) {
      return const UiKitView(
          viewType: viewType,
          layoutDirection: TextDirection.ltr,
          creationParams: creationParams,
          creationParamsCodec: StandardMessageCodec());
    } else {
      return const Text("Only works on Android or iOS");
    }
  }
}

```

总之，使用了platform view来显示原生视图都会造成app性能上的一定的损失。所以酌情考虑使用原生视图。如果一定要使用的话，遇到性能问题或者加载缓慢可以在加载之前先播放一个动画。

使用原生视图的功能还可以在**macOS**实现，具体可以参考[官方文档](https://docs.flutter.dev/platform-integration/macos/platform-views)。
