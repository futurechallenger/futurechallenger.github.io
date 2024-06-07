---
title: Flutter -- Bloc状态管理 
date: 2024-06-07 21:43:21
tags:
---

在Flutter生态里有很多的状态管理工具。比如，你一开始就会熟悉的`setState`和`provider`这些。在其他的工具里，**Bloc**这个模式非常流行。所以，我们花点时间来学一学。

本文继续用todo app作为例子，尽量接近真实开发环境。大部分主要的操作需要请求后端服务器。状态管理工具当然就使用**Bloc**模式来实现。另外，还会实现Bloc的单元测试，以此保证Bloc的类都能按照预想的工作。

## Bloc是什么

在正式开始之前，如果你对Dart和Flutter有一定的了解。能理解Flutter的状态管理以及一些Flutter单元测试的知识就更好了。

Bloc的意思是*Business Logic Component*，这个模式用于分离逻辑和用户界面（UI）组件。这样可以让开发者更好的维护代码。

Bloc模式包含一下的部分：

- **Bloc**：这个是核心部分。它包含业务逻辑和app的状态管理。
- **事件**：Bloc会对于每个事件做出反应。事件可以是用户的操作（比如，按钮点击）也可以是外部对数据的更新。
- **状态**：状态是用来代表app可以存在的各种状态。Bloc发出更具不同的事件发出状态。UI则根据状态的变化做出反应，绘制出不同的界面。
- **用户界面**：用户界面根据状态的变化呈现不同的界面。当Bloc发出一个新的状态，界面就更具这个新的状态呈现一个新的几面。

基本上，Bloc模式会根据不同的事件产生出新的不同的状态。之后，用户界面观察状态的更新，更具新的状态绘制界面。为了更加深刻的说明这个问题，假设我们的todo app，可以点一个按钮表示完成，再点一次表示未完成。

- **事件**：
    * 事件完成按钮的toggle事件。未完成的点了就是完成，完成了的点了就是未完成。
- **状态**：
    * 初始状态：未完成
    * 已完成状态：从未完成转变为已完成
    * 未完成状态：从已完成变为未完成

实际代码与这个会有些许不同。已完成会分为：*已完成-成功*和*已完成-失败*。

## 调用远程API了，Bloc怎么管理状态

这里todo app又可以上场了。这个todo app调用远程API，并在app内使用Bloc管理状态。最后添加上单元测试（或者要尝试测试驱动开发也可以先写测试）。

这个todo app也是老朋友了，已经服务与Riverpod和GetX了。如果你看过这些文章再看这个会觉得简单。那么首先来添加所需要的依赖吧：
```yaml
dependencies:
  flutter:
    sdk: flutter

  flutter_bloc: ^8.1.5
  go_router: ^14.1.0
  json_annotation: ^4.9.0
  http: ^1.2.1
  equatable: ^2.0.5

dev_dependencies:
  flutter_test:
    sdk: flutter

  flutter_lints: ^3.0.0
  json_serializable: ^6.8.0
  build_runner: ^2.4.9
  bloc_test: ^9.1.7
  mocktail: ^1.0.3
```

执行命令：`flutter pub get`，这些依赖就安装好了。

### Service类

现在来写Service类。这些不用专门写了，直接从其他的todo app里拿过来就可以用。为了方便直接看本文的读者，这里来简单描述一下。有一个http请求的util类：
```dart
class HttpRequest {
  final String hostUrl = 'http://address_of_your_server:17788'; // 1
  var _client = http.Client(); // 2

  set client(c) => _client = c;
  get client => _client;

  Future<List<TodoItem>?> fetchTodoList({
    bool all = false,
    String completed = 'completed',
  }) async {
    // ...
  }

  Future<TodoItem?> fetchTodoById(int todoId) async {
    // ...
  }

  Future<TodoItem> addTodoItem(String todoTitle) async {
    // ...
  }

  Future<void> updateTodo(int id,
      {String? todoTitle, String? note, int? status, int? deleted}) async {
    // ...
  }

  Future<void> deleteTodo(int id) async {
    // ...
  }
}
```

1. *address_of_your_server*就是你的服务器地址，你的服务在本机的话可以写本机的ip地址，注意**不是localhost或者127.0.0.1**，是你的ip地址。要不然在移动端访问不到。
2. 需要使用`client`，否则单元测试不方便。更多可以参考关于Flutter测试部分的内容。

其他的方法就是对于todo的操作了。比如，获取todo列表、对todo的CRUD等。在其他的service class里可以使用。

**注意**：这些service类只是为了让整个的app尽量贴近实际开发而写的。读者在实际开发的时候根据实际情况决定如何操作。

获取todo列表的service类：
```dart
class TodoListRepository {
  Future<List<TodoItem>?> fetchTodoList({
    bool all = false,
    String completed = 'completed',
  }) async {
    HttpRequest request = HttpRequest();
    return request.fetchTodoList(all: all, completed: completed);
  }
}
```

更新todo的状态的service类：
```dart
class TodoDetailRepository {
  Future<TodoItem?> toggleTodoStatus({required TodoItem todo}) async {
    HttpRequest request = HttpRequest();
    try {
      request.updateTodo(todo.id!, status: todo.status);
      return todo;
    } catch (err) {
      return null;
    }
  }
}
```

### 实体类

这里单独拿出来是非常有必要的。

记得在依赖里有一项是`equatable`不。这个是用来比较两个实例是否“存在变化”的。在**Bloc**中，它还会被用在状态的对比上。是否要重新绘制界面取决于状态是否有变化。`equatable`也可以用于对比两个状态是否发生了变化。

```dart
import 'package:equatable/equatable.dart';
import 'package:flutter/widgets.dart';
import 'package:json_annotation/json_annotation.dart';

part 'todo_model.g.dart'; // 1

@immutable
@JsonSerializable() // 2
class TodoItem extends Equatable { // 3
  const TodoItem({
    this.id,
    required this.content,
    this.note,
    this.deleted,
    this.status,
  });

  final int? id;
  final String? content;
  final String? note;
  final int? deleted;
  final int? status;

  static fromJson(Map<String, dynamic> json) => _$TodoItemFromJson (json); // 4
  Map<String, dynamic> toJson() => _$TodoItemToJson(this); // 4

  @override
  List<Object?> get props => [id, content, note, deleted, status]; // 5
}
```

1. 1、2、4都会在使用Json实体代码生成的时候用到。
2. 3、5会在`equatable`对比中用到。尤其**5**，是在对比中需要对比的属性，如果这些属性有一个不相等，那么两个对象就是不相等的。

### Bloc类

前面的内容都做好咱们就可以正式开始Bloc类的开发了。在开发中你会发现Bloc类连接了Repository层和UI层，Bloc类在中间正好有效的分割了业务逻辑和UI展示。

先来看看一个Bloc类是什么样子的：
```dart
class TodoListBloc extends Bloc<TodoListEvent, TodoListState> { // 1
  TodoListBloc() : super(const TodoListInitialState()) {
    on<TodoListRequested>(_onFetchTodoList); // 2
    on<TodoItemUpdated>(_onTodoItemUpdated);
  }
}
```

1. 一个Bloc类需要继承`Bloc<E, S>`。`E`是事件类型，`S`是状态类型。因此，需要先定义事件和状态。
2. on<E>()就是用来根据事件执行逻辑代码，之后发出（emit）对应的状态的方法。

#### 定义事件

事件的定义比较简单。在界面显示的时候开始加载todo列表。所以，只需要一个数据请求事件：`TodoListRequested`。

```dart
sealed class TodoListEvent extends Equatable {}

class TodoListRequested extends TodoListEvent {
  @override
  List<Object?> get props => [];
}
```

#### 定义状态

在定义状态的时候还需要考虑一个问题。在本里中，获取所有todo列表的时候是通过网络请求实现的。这需要一个过程：发出请求-->后台解析请求，并返回-->数据返回前端，并在UI显示。在这个过程还需要考虑出错的情况。

所以，在定义状态的时候需要有一个
- 初始状态
- 数据加载完成
- 数据请求出错

从*初始状态*到其他两个状态的过程在界面上就是**loading**。


```dart
@immutable
sealed class TodoListState extends Equatable {
  const TodoListState({required this.todoList});

  final List<TodoItem> todoList;
  @override
  List<Object?> get props => [todoList];
}

class TodoListInitialState extends TodoListState {
  const TodoListInitialState() : super(todoList: const <TodoItem>[]);
}

class TodoListLoadedState extends TodoListState {
  const TodoListLoadedState({required super.todoList});
}

class TodoListErrorState extends TodoListState {
  const TodoListErrorState() : super(todoList: const <TodoItem>[]);
}
```

如你所见，在状态的基类中`extends`了`Equatable`。就像前文所说的，这个是为了比较两个状态的时候更加高效。

在`TodoListLoadedState`中，构造函数有一个参数是todo列表。这是因为这个状态在emit的时候，需要带着后台返回的todo列表一起给UI展示用。

事件和状态都齐活了，那么就开始处理Bloc类了。

#### 定义Bloc类

Bloc类的作用就是根据传入的事件，做逻辑处理，之后更具处理的结果发出不同的状态。

```dart
class TodoListBloc extends Bloc<TodoListEvent, TodoListState> {
  TodoListBloc() : super(const TodoListInitialState()) {
    on<TodoListRequested>(_onFetchTodoList);
  }

  Future<void> _onFetchTodoList(
      TodoListEvent event, Emitter<TodoListState> emit) async {
    emit(const TodoListInitialState()); // 1
    try {
      final repository = TodoListRepository(); // 2
      final result = await repository.fetchTodoList(all: true);
      emit(TodoListLoadedState(todoList: result ?? [])); // 3
    } catch (error) {
      emit(const TodoListErrorState()); // 4
    }
  }
}
```

1. 首先发出一个初始状态。
2. Repository的实例在这里为了简单是直接初始化在需要使用的地方，其实也可以使用*flutter_bloc*提供的依赖注入工具。可以参考更新todo状态的部分的代码。
3. 如果正确获得数据则发出`TodoListLoadedState`，里面带着todo列表。
4. 如果不能正确处理请求，则发出`TodoListErrorState`。

在初始状态发出，正确获取数据或者错误获取数据的状态发出之前是数据加载中。在UI显示loading的界面元素让用户知道。

### 在UI中使用Bloc类

```dart
class HomePage extends StatelessWidget {
  const HomePage({super.key});

  @override
  build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        // 略
      ),
      body: BlocBuilder<TodoListBloc, TodoListState>(builder: (context, state) {
        switch (state) { // 2
          case TodoListInitialState():
            // Loading...
          case TodoListLoadedState(): // 3
            return ListView.separated(
              restorationId: 'sampleItemListView',
              itemCount: state.todoList.length,
              itemBuilder: (BuildContext context, int index) {
                final item = state.todoList[index];
                content: item.content ?? '',

                // 其他略
              },
              separatorBuilder: (context, index) => const Divider(),
            );
          case TodoListErrorState():
            // Error, 略...
        }
      }),
    );
  }
}
```

1. 使用`BlocBuilder`来根据Bloc发出来的状态来绘制界面。
2. 使用`switch`语句。根据`state`的类型分别执行不同的分支。
3. 在`TodoListLoadedState`状态里就可以使用状态带回来的todo列表。

现在里运行代码查看todo列表就差一点点了，不是运行api后台。

#### 在app中配置Bloc

不配置的话，Bloc是没发使用的。比如前文中所述`BlocBuilder`就用不了。下面来看看如何配置：

```dart
void main() {
  runApp(
    MultiRepositoryProvider( // 1
        providers: [
          RepositoryProvider(
              create: (BuildContext context) => TodoDetailRepository()) // 2
        ],
        child: MultiBlocProvider(providers: [ // 3
          BlocProvider<TodoListBloc>(
              create: (BuildContext context) =>
                  TodoListBloc()..add(TodoListRequested())), // 4
          // BlocProvider<TodoDetailCubit>(
          //     create: (_) => TodoDetailCubit(const TodoDetailState())),
        ], child: const MyApp())),
  );
}
```

1. `MultiRepositoryProvider`这是flutter_bloc提供的Repository的依赖注入工具。
2. `RepositoryProvider`配置了repository的初始化方式。如：
```dart
RepositoryProvider(create: (BuildContext context) => TodoDetailRepository()) 
  ```
3. `MultiBlocProvider`和repository provider类似的，这是一个多个bloc的provider。
4. `BlocProvider`配置bloc的初始化方式。
```dart 
BlocProvider<TodoListBloc>(
              create: (BuildContext context) =>
                  TodoListBloc()..add(TodoListRequested()))
```

对于repository和bloc的provider的使用，统一方法是`context.read<T>()`。比如上例中的repository provider可以这样使用`context.read<TodoDetailRepository>()`。而bloc provider的使用方法是:`context.read<TodoListBloc>()`。更多详细的内容可以参考[文档](https://bloclibrary.dev/flutter-bloc-concepts/#blocprovider)。

### 简化版的Bloc -- Cubit

Cubit是一个简化版的Bloc。基本的原理是完全一直的，只是事件不需要定义，是Cubit类的方法。

在todo app中，用户在todo列表中点击了一个todo会跳转到todo的详情也，并在该页面显示todo的title。这一块儿的工能使用Cubit来实现。

既然不用定义事件，那么就以定义状态开始：
```dart
final class TodoDetailState extends Equatable {
  const TodoDetailState({this.todoItem});

  final TodoItem? todoItem;

  @override
  List<Object?> get props => [todoItem];
}
```

状态会以不同的`todoItem`做对比。

然后就可以定义`Cubit`了。
```dart
class TodoDetailCubit extends Cubit<TodoDetailState> {    // 1
  TodoDetailCubit({required TodoItem todo})
      : super(TodoDetailState(todoItem: todo));

  void setTodo(TodoItem todo) => emit(TodoDetailState(todoItem: todo));   // 2
}
```

1. 定义Cubit类只需要一个类型参数，就是开始定义的状态类型。
2. 定义cubit的方法`setTodo`，这个就是当做事件用的。或者，更加准确的合作是当做bloc类的`on`方法用。只是这个`on`方法已经关联好了一个事件。

看看在UI中是怎么使用的。在go_router的路由配置中，跳转到详情也的时候拿出传过来的参数，并且配置依赖注入中cubit类的初始化方式。

```dart
GoRoute(
  path: "/edit",
  builder: (context, state) {
    final todo = state.extra as TodoItem;
    return MultiBlocProvider(
      providers: [
        BlocProvider(
          create: (context) => TodoDetailCubit(todo: todo),
        ),
        BlocProvider(
            create: (context) => TodoToggleBloc(
                todoDetailRepository:
                    // RepositoryProvider.of<TodoDetailRepository>(
                    //     context),
                    context.read<TodoDetailRepository>(),
                todo: todo))
      ],
      child: const EditPage(),
    );
  }),
```

在UI中使用：
```dart
final todo =
        context.select((TodoDetailCubit cubit) => cubit.state.todoItem);
```
在`EditPage`的`build`方法中使用`context.select`方法获取状态中的todo，并把它的title显示在页面中。

### 对于bloc的测试

在项目目录下的*test*目录下新建一个文件叫做*bloc_test.dart*。我们就在这里开始写对于Bloc的测试。

首先需要Mock一下我们要用的repository类：
```dart
class MockTodoListRepository extends Mock implements TodoListRepository {}
```

这个操作之后直接使用Mock之后的类来初始化出来给Bloc类使用，这样Bloc在运行中调用的就是Mock过的方法了。你可以操总这个方法的行为，比如在调用的时候抛出一个异常。

```dart
  late MockTodoListRepository mockRepository;

  setUp(() {
    mockRepository = MockTodoListRepository();
  });
```
使用`setUp`方法，在每个测试开始运行之前保证`mockRepository`使用`MockTodoListRepository`初始化。

正式开始bloc_test之前，先使用常用的单元测试来验证Bloc在初始化之后的状态是否为初始状态：`TodoListInitialState`。
```dart
    test('load todo list init state', () {
      expect(TodoListBloc(todoListRepository: mockRepository).state,
          const TodoListInitialState());
    });
```

开胃菜之后正式开始*bloc_test*。

*bloc_test*和一般的测试的不同就在于开头。*bloc_test*写作`blocTest`。还有两个类型参数。一个是Bloc类的类型，一个是这个Bloc类发出的状态的类型。看起来是这样的：`blocTest<TodoListBloc, TodoListState>()`。

来看一个真是的例子：
```dart
    blocTest<TodoListBloc, TodoListState>(
      'emits [TodoListInitialState, TodoListErrorState] when http request throws an error',
      setUp: () => when(mockRepository.fetchTodoList).thenThrow(Exception()), // 1
      build: () => TodoListBloc(todoListRepository: mockRepository), // 2
      act: (bloc) => bloc.add(TodoListRequested()), // 3
      expect: () => <TodoListState>[  // 4
        const TodoListInitialState(),
        const TodoListErrorState()
      ],
      verify: (_) => verify(() =>  // 5
              mockRepository.fetchTodoList(all: true, completed: 'completed'))
          .called(1),
    );
```
本例是用来测试在获取todo列表的时候遇到异常是Bloc类是否可以正常的工作，即：可以发出一个初始状态的state和一个遇到异常时候的状态。并且验证获取todo列表的方法被执行了一次。

在每个测试里都会包含一个**描述**，这就不用多说了。文本内容有开发者自己定，或者根据团队的开发准则定。其他的解析如下：
1. `setUp`：在开始测试之前需要执行的操作，这里配置在执行到方法`fetchTodoList`的时候抛出异常。
2. `build`：初始化Bloc类：`TodoListBloc`，并在初始化的时候使用mock的repository作为参数。
3. `act`：正式开始执行bloc，给bloc添加一个事件。
4. `expect`：在bloc收到事件之后开始执行bloc的内部逻辑。这时候正式开始检测执行的是否符合预期：先发出一个初始状态，再发出一个错误状态。所以expect的是一个状态数组。
5. `verify`：在测试运行完成之后，检测`fetchTodoList`方法执行了一次。
