# Dart

* 1.级联操作符

  Dart 中 **级联操作符** 可以返回对象从而继续执行其他方法：

  ```dart
  new Column(
        children: [
          new Container(),
        ]
          ..addAll(List.generate(3, (index) {
            return Container();
          }))
          ..addAll([
            new HorizontalLine(height: 5),
          ]),
      );
  ```

* 2.一次性执行匿名回调函数

  Dart支持一次性执行匿名回调函数，如：

  ```dart
  void test() {
    () {
      print('hi');
    }();
  }
  ```

  也可在组件中运用并接收参数，如：

  ```dart
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      body: new Text((String text) {
        return text;
      }('我是文字')),
  }
  ```

* 3.可选方法参数

  `Dart` 方法可以设置 **参数默认值** 和 **指定名称** 。
  
  比如： `getDetail(Sting userName, reposName, {branch = "master"}){}` 方法，这里 branch 不设置的话，默认是 “master” 。参数类型 可以指定或者不指定。调用效果： `getRepositoryDetailDao(“aaa", "bbbb", branch: "dev");` 。

* 4.Assert(断言)

  `assert` 只在检查模式有效，在开发过程中，`assert(unicorn == null);` 只有条件为真才正常，否则直接抛出异常，一般用在开发过程中，某些地方不应该出现什么状态的判断。

* 4.扩展

  扩展可以给指定类型的数值增加获取方法，如（ScreenUtil给num加的扩展）：

  ```dart
  extension SizeExtension on num {
    num get w => ScreenUtil().setWidth(this);
  
    num get h => ScreenUtil().setHeight(this);
  
    num get sp => ScreenUtil().setSp(this);
  
    num get ssp => ScreenUtil().setSp(this, allowFontScalingSelf: true);
  }
  ```

  比如我想获取一个ScreenUtil适配的50宽度值：

  ```dart
  Container(width: 50.w)
  ```

  等同于：

  ```dart
  Container(width: ScreenUtil().setWidth(50))
  ```

  从而简化了代码，提升了开发效率。

* 5.runZoned

  可以在自己的区域中运行指定代码（根据zoneSpecification使用Zone.fork创建的区域），如果代码出现错误将抛出全局错误在onError：

  ```dart
  runZoned(() {
      runApp(MyApp());
    }, onError: (Object obj, StackTrace stack) {
      print(obj);
      print(stack);
    });
  ```

  我们通用的错误信息统计平台如sentry，一般都是放在runZoned的onError内。







# 组件

* 1.ListView自动内边距

  ListView会在脚手架中未写AppBar的情况下自动添加padding内边距，内边距值是状态栏高度。

* 2.Draggable重绘问题

  Draggable的child和feedback是同一个组件是每次拖动都会重绘组件，如果不想被重绘，在组件的key给个GlobalKey即可。

* 3.拿到数组模型索引

  List.generate可以获得数组模型的索引index，length写数组模型的长度即可。

* 4.输入框内容被清空

  如果出现输入框内容输入的时候突然返回到桌面，回来的时候被清空了，可以尝试使用WidgetsBindingObserver监听不可见生命周期，输入框焦点自动取消，从而输入框会保存原有内容。（这个问题在早期flutter版本出现）

* 5.刷新页面的指定组件

  如果只想刷新某个页面的指定组件可使用GlobalKey包裹此组件的State类，调用此组件的时候传过去，想刷新的时候就可以在外面调用这个GlobalKey的setState或其他刷新方法了。

  详情：[http://book.flutterj.com/chapter2/partial_refresh.html](http://book.flutterj.com/chapter2/partial_refresh.html)

* 6.拿特殊滑动组件内控制器

  特殊滑动组件如NestedScrollView是有内外控制器的，普通的赋值controller只能拿到外部的滑动值，想要拿到内控制可以在NestedScrollView的body中类在上下文获取，调用上下文的ancestorWidgetOfExactType
  
  ```dart
  
  class DataBody extends StatefulWidget {
    @override
    _DataBodyState createState() => _DataBodyState();
  }
  
  class _DataBodyState extends State<DataBody> {
    ScrollController pageScrollController;
  
    Type typeOf<T>() => T;
  
    @override
    void initState() {
      super.initState();
  
      PrimaryScrollController primaryScrollController =
          context.ancestorWidgetOfExactType(typeOf<PrimaryScrollController>());
  
      pageScrollController = primaryScrollController.controller;
    }
  }
  ```
  
  不过这个在1.12已被废弃，新版flutter使用findAncestorWidgetOfExactType代替。
  
  







# 功能

* 1.无需上下文路由

  自己定义个NavigatorState的全局key然后赋值到MaterialApp的navigatorKey就能使用NavigatorState的所有功能并无需上下文。

  详情：[http://book.flutterj.com/chapter2/no_context_route.html](http://book.flutterj.com/chapter2/no_context_route.html)

* 2.Future执行完成错误

  如果出现Future执行完成错误可尝试使用Completer的future，如：

  ```dart
  Future<Null> _refreshData() {
    final Completer<Null> completer = new Completer<Null>();
  
    new Future.delayed(new Duration(seconds: 2), () {
      completer.complete(null);
    });
  
    return completer.future;
  }
  
  ```
  
* 3.调用方法出现null错误

    dispose销毁某对象时在调用`.dispose`前加个`?`问号，可以避免前面的为空导致调用不到dispose方法报错。

* 4.页面状态保存

  Flutter 中可以通过 `mixins AutomaticKeepAliveClientMixin` ，然后重写 `wantKeepAlive` 保持住页面，记得在被保持住的页面 `build` 中调用 `super.build` 。

*  5.手势识别范围

  使用GestureDetector范围只有child的设定颜色或使用区域才可被触发，behavior为HitTestBehavior.translucent时整个child会被触发，InkWell组件默认整个child会被触发。

* 6.Flutter 手势事件主要是通过竞技判断的：

    主要有 `hitTest` 把所有需要处理的控件对应的 `RenderObject` ， 从  `child` 到 `parent` 全部组合成列表，从最里面一直添加到最外层。

    然后从队列头的 child 开始 for 循环执行 `handleEvent` 方法，执行 `handleEvent` 的过程不会被拦截打断。

    一般情况下 Down 事件不会决出胜利者，大部分时候是在 MOVE 或者 UP 的时候才会决出胜利者。

    **竞技场关闭时只有一个的就直接胜出响应，没有胜利者就拿排在队列第一个强制胜利响应。**

    同时还有 `didExceedDeadline` 处理按住时的 Down 事件额外处理，同时手势处理一般在 `GestureRecognizer` 的子类进行。





# 插件开发

* 1.android获取context和activity

  可以在插件的Plugin类实现个ActivityAware，然后在onAttachedToActivity和onReattachedToActivityForConfigChanges都可以拿到ActivityPluginBinding的binding，binding调用getActivity就是了。
  
  ```java
  
  public class DemoPlugin implements FlutterPlugin, MethodCallHandler, ActivityAware {
  
      static Context ctx;
      static Activity activity;
  
      @Override
      public void onDetachedFromEngine(@NonNull FlutterPluginBinding binding) {
          ctx = binding.getApplicationContext();
      }
  
      @Override
      public void onAttachedToActivity(ActivityPluginBinding binding) {
          activity = binding.getActivity();
          ctx = binding.getActivity();
      }
  
      @Override
      public void onDetachedFromActivityForConfigChanges() {
  
      }
  
      @Override
      public void onReattachedToActivityForConfigChanges(ActivityPluginBinding binding) {
          ctx = binding.getActivity();
          activity = binding.getActivity();
      }
  }
  ```
  
* 2.PlatformView
  
  Flutter 中通过 `PlatformView` 可以嵌套原生 `View` 到 `Flutter` UI 中，这里面其实是使用了 `Presentation` + `VirtualDisplay` + `Surface` 等实现的，大致原理就是：
  
  使用了类似副屏显示的技术，`VirtualDisplay` 类代表一个虚拟显示器，调用 `DisplayManager` 的 `createVirtualDisplay()` 方法，将虚拟显示器的内容渲染在一个 `Surface` 控件上，然后将 `Surface` 的 id 通知给 Dart，让 engine 绘制时，在内存中找到对应的 `Surface` 画面内存数据，然后绘制出来。**实时控件截图渲染显示技术。**
  
  

* 3.Platform Channel

  Flutter 中可以通过 `Platform Channel` 让 Dart 代码和原生代码通信的：

  > - `BasicMessageChannel` ：用于传递字符串和半结构化的信息。
  > - `MethodChannel` ：用于传递方法调用（method invocation）。
  > - `EventChanne` l: 用于数据流（event streams）的通信。

  **同时 `Platform Channel` 并非是线程安全的** ，更多详细可查阅闲鱼技术的 [《深入理解Flutter Platform Channel》](https://www.jianshu.com/p/39575a90e820)

基础数据类型映射如下：

![image-20200918115845140](/Users/q1/Library/Application Support/typora-user-images/image-20200918115845140.png)











# 介绍

* 1.运行模式

  **Flutter 的 Debug 下是 JIT 模式，release下是AOT模式。**





























