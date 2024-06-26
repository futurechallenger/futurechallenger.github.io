---
title: Flutter -- Localization
date: 2024-06-26 14:29:11
tags:
---

首先，来概括一下在Flutter上使用本地化工具需要那些步骤：
1. 安装需要的依赖。
2. 配置`l10n`相关的文件，并添加需要支持的语言的翻译内容。配置完成之后可以自动生成相关文件。
3. 上一步生成的文件，在app中引入。并把l10n的代理和支持的语言（locale）传入`MaterialApp`或者`CupertinoApp`。

完成以上的三步基本就完成了Flutter app的本地化工作。之后，增加文本就增加翻译即可。



1. 安装`flutter_localization`和`intl`这两个包
```bash
flutter pub add flutter_localizations --sdk=flutter
flutter pub add intl:any
```

2. 引入`flutter_localization`，给`MaterialApp`或者`CupertinoApp`指定`localizationsDelegates`和`supportedLocales`。

```dart
import 'package:flutter_localizations/flutter_localizations.dart';
```

```dart
return const MaterialApp(
  title: 'Localizations Sample App',
  localizationsDelegates: [
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
  ],
  supportedLocales: [
    Locale('en'), // English
    Locale.fromSubtags(languageCode: 'zh'),
  ],
  home: MyHomePage(),
);
```

3. 添加需要翻译的内容
    a. 打开自动生成。
    ```yaml
    # 这项配置应该在你创建app的时候就加好了
    flutter:
        generate: true 
    ```

    b. 在根目录添加一个配置文件: *l10n.yaml*。(这一项也会在创建app的时候加好)
    ```yaml
    arb-dir: lib/l10n
    template-arb-file: app_en.arb
    output-localization-file: app_localizations.dart
    ```
    这里配置了：
    - 这里配置了arg文件存放的目录。arb文件（App Resource Bundle）里存放的是本地化的资源。
    - 英文本地化模板。
    - 在*app_localization.dart*文件生成一些相关内容。

    c. 在前一项配置的arb目录下添加英文模板：
    ```json
    {
        "helloWorld": "Hello World!",
        "@helloWorld": {
            "description": "The conventional newborn programmer greeting"
        }
    }
    ```

    d. 在同一个目录下，添加另外一个*app_zh.arb*：
    ```json
    {
        "helloWorld": "你好世界!"
    }
    ```

添加完成这些之后，运行命令：`flutter pub get`或者`flutter run`。自动生成代码的部分会自动运行。在项目的根目录下的*.dart_tool/flutter_gen/gen_l10n目录下会包含自动生成出来的文件。

也可以运行命令：`flutter gen-l10n`命令来生成这些文件。这样可以不必运行app。**注意**：如果配置完成之后还没有运行过app，先运行一次。否则可能会出错。

引入生成的文件，并在之前配置过的代码中添加这些配置：
```dart
import 'package:flutter_gen/gen_l10n/app_localizations.dart';
```

添加配置：
```dart
return const MaterialApp(
  title: 'Localizations Sample App',
  localizationsDelegates: [
    AppLocalizations.delegate, // 添加这个配置
    GlobalMaterialLocalizations.delegate,
    GlobalWidgetsLocalizations.delegate,
    GlobalCupertinoLocalizations.delegate,
  ],
  supportedLocales: [
    Locale('en'), // 英文
    Locale.fromSubtags(languageCode: 'zh'),
  ],
  home: MyHomePage(),
);
```

在`AppLocalizations`类中有自动生成的`localizationsDelegates`和`supportedLocales`。也就是说这些内容就无需手动配置了：
```dart
const MaterialApp(
  title: 'Localizations Sample App',
  localizationsDelegates: AppLocalizations.localizationsDelegates,
  supportedLocales: AppLocalizations.supportedLocales,
);
```

之后这样调用不同的语言文本内容：
```dart
appBar: AppBar(
  title: Text(AppLocalizations.of(context)!.helloWorld),
),
```

### 本地化字符串参数的处理


