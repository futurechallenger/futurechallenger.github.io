---
title: Flutter -- Localization
date: 2024-06-26 14:29:11
tags:
---

本文会分两部分，首先我们来学习Flutter本身给我们提供的本地化的工具如何使用。然后我们来看看社区提供了那些优化了的工具。

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
    Locale('es'), // Spanish
  ],
  home: MyHomePage(),
);
```

