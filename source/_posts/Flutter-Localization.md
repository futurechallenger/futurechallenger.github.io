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

2. 添加配置文件和英文模板以及需要翻译的语言。
    a. 打开自动生成。
    ```yaml
    # 这项配置应该在你创建app的时候就加好了
    flutter:
        generate: true 
    ```

    b. 在根目录添加一个配置文件: *l10n.yaml*。(这一项也会在创建app的时候加好，内容与下面的不一定相同)
    ```yaml
    arb-dir: lib/l10n
    template-arb-file: app_en.arb
    output-localization-file: app_localizations.dart
    ```
    这里配置了：
    - 这里配置了arb文件存放的目录。arb文件（App Resource Bundle）里存放的是本地化的资源。
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

3. 引入生成的文件，并在之前配置过的代码中添加这些配置：
```dart
import 'package:flutter_gen/gen_l10n/app_localizations.dart';
```

添加配置：
```diff
return const MaterialApp(
  title: 'Localizations Sample App',
-  localizationsDelegates: [
-    AppLocalizations.delegate, // 添加这个配置
-    GlobalMaterialLocalizations.delegate,
-    GlobalWidgetsLocalizations.delegate,
-    GlobalCupertinoLocalizations.delegate,
-  ],
+  localizationsDelegates: AppLocalizations.localizationsDelegates,
-  supportedLocales: [
-    Locale('en'), // 英文
-    Locale.fromSubtags(languageCode: 'zh'),
-  ],
+  supportedLocales: AppLocalizations.supportedLocales,
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

### 字符串模板的处理


### 只显示一个语言

#### 占位符

比如，现在要显示todo的完成状态：*蹲腿已完成*，或者*欺骗餐进行中*。那么需要翻译的字符串就可以成：`{todoTitle} is {status}`。
在英文的模板里可以写成这样：
```json
"todoStatus": "{todoTitle} is {status}",
"@todoStatus": {
  "description": "Todo status",
  "placeholders": {
    "todoTitle": {
      "type": "String",
      "example": "Leg day"
    },
    "status": {
      "type": "String",
      "example": "completed"
    }
  }
}
```

当然同时需要修改其他语言的模板内容：
```json
{
    "@@locale": "zh",
    "appTitle": "提醒事项",
    "todo": "提醒事项",
    "settings": "设置",
    "todoStatus": "{todoTitle}的状态是{status}"
}
```

所有对于占位符的配置都在英文模板里实现就好，其他语言的只要有占位符出现就可以。

### 在翻译串中使用Select

在上一个例子中表达状态的时候使用了占位符，`"todoStatus": "{todoTitle}的状态是{status}"`。但是`status`在服务端返回的是数字。直接扔数字进去肯定不行，在代码中把数字转换成对应的字符串也不行，难道要在代码完成翻译？那还要flutter的l10n工具干啥呢。 所以，这就需要用到翻译的`Select`语法。

```json
{
  "todoStatus": "{todoTitle} is {status, select, 0{in progress} 1{completed} other{in progress}}",
  "@todoStatus": {
    "description": "Todo status",
    "placeholders": {
      "todoTitle": {
        "type": "String",
        "example": "Leg day"
      },
      "status": {
        "type": "String"
      }
    }
  },
}
```

对应的修改中文的arb文件：
```json
{
  "todoStatus": "{todoTitle}的状态是{status, select, 0{进行中} 1{完成} other{进行中}}",
  "@todoStatus": {
    "description": "todo完成状态",
    "placeholders": {
      "todoTitle": {
        "type": "String"
      },
      "status": {
        "type": "String"
      }
    }
  }
}
```

在文本中显示的代码是这样的：
```dart
Text(
  AppLocalizations.of(context)!.todoStatus(
                      _.rxTodoItem().content,
                      '${_.rxTodoItem().status ?? 0}',
                    )
)
```

**注意**：在翻译串的`Select`语法中**other**是必须的，类型必须是`String`。


### 处理英文的复数

英文有复数，那就得处理一下。。。

```json
{
  "todoItems": "{count, plural, =0{no todo items} =1{1 todo item} other{{count} items}}",
  "@todoItems": {
    "description": "A plural message",
    "placeholders": {
      "count": {
        "type": "num",
        "format": "compact"
      }
    }
  }
}
```

注意这个`other`，使用了占位符中的占位符。

```json
{
  "todoItems": "{count, plural, =0{没有提醒事项} =1{一个提醒事项} other{{count} 个提醒事项}}",
  "@todoItems": {
    "description": "一个复数消息",
    "placeholders": {
      "count": {
        "type": "num",
        "format": "compact"
      }
    }
  }
}
```

在文本中显示的代码：
```dart
Text(
  AppLocalizations.of(context)!.todoItems(10) // 可以随意修改，hot reload会让结果很快显示出来
)
```

有心的同学这里可能已经注意到了，*一个注意事项*只是后面`other{{count}个提醒事项}`的特殊情况。这里为了和英文模板保持就先留着了。

### 转义语法

首先需要在配置文件中开启这个功能：
```yaml
# 在项目更目录下的l10n.yaml文件中添加如下的配置开启这个功能

use-escaping: true
```

在需要显示特殊符号的翻译串中使用转义语法，这里用英文举例：
```json
{
  "helloWorld": "Hello! '{Isn''t}' this a wonderful day?"
}
```
显示出来的结果是：
```
"Hello! {Isn't} this a wonderful day?"
```

## 完

Flutter主要的本地化内容就这些了。关于货币和日期的部分可以查看[官方文档](https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization#messages-with-numbers-and-currencies)。