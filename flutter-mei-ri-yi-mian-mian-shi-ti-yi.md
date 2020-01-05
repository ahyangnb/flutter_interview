---
description: 屏幕适配，isolate通信和原理，热重载原理和过程，Flutter性能，跨平台区别，组件渲染。
---

# Flutter每日一面（面试题一）



## **1.Flutter屏幕算法面试题基础\(一\)：**

假如蓝湖设计图给你一张轮播图，宽度是 x 高度是 y（x px \* y px），左右间隔是t，如何使用屏幕算法适配全机型屏幕宽和高？

其实这种形式是可以直接用`AspectRatio`写宽高比，但是面试官要求手动算出来，可能这里答案不止一个，如果大家有更好的可以在下方评论出来。

**答案：**

> 宽度：整宽 - t  _2（左右的）。 高度：\(整宽 - t_  2 \) \* y / x。

## **2.Flutter屏幕算法面试题基础\(二\)：**

假如蓝湖设计图给你一个未知数据数量有规则的列表视图，要求一行显示5个，每个间隔为10（含上下），最外边距`margin`左右都为20，高度为50，多出的数据继续往下排并向左对齐，适配任何机型，你会使用什么组件？怎么做适配？

**答案：**

> 使用组件：Wrap spacing和runSpacing都设置为10间隔， 然后Item的高度设置为50，宽度算法为： \( 整宽 - （外边的margin + 内边的space） \) / 5

## **3 isolate是怎么进行通信和实例化的？**

* **答案：**

  > isolate线程之间的通信主要通过port来进行，这个port消息传递过程是异步的。通过dart源码可以看出，实例化一个isolate的过程包括： 1.实例化isolate结构体。 2.在堆中分配线程内存。 3.配置port等过程。

代码示例： 下面是一个isolate的例子，例子中新建了一个isolate，并且绑定了一个方法网络请求和数据解析处理，并通过port将处理好的数据返回给调用方。

```dart
loadData() async {
  // 通过spawn新建一个isolate，并绑定静态方法
  ReceivePort receivePort = ReceivePort();
  await Isolate.spawn(dataLoader, receivePort.sendPort);

  // 获取新的isolate监听port
  SendPort sendPort = await receivePort.first;
  //调用sendReceive自定义方法
  List dataList = await sendReceive(sendPort,
      'http://www.flutterj.com');
  print('dataList $dataList');
}

// isolate绑定方法
static dataLoader(SendPort sendPort) async {
  // 创建监听port，并将sendPort传给外界来调用
  ReceivePort receivePort = ReceivePort();
  sendPort.send(receivePort.sendPort);
// 监听外界调用
  await for (var msg in receivePort) {
    String requestURL = msg[0];
    SendPort callbackPort = msg[1];

    Client client = Client();
    Response response = await client.get(requestURL);
    List dataList = json.decode(response.body);
// 回调返回值给调用者
    callbackPort.send(dataList);
  }
}

// 创建自己的监听port，并且向新的isolate发送消息
Future sendReceive(SendPort sendPort, String url) {
  ReceivePort receivePort = ReceivePort();
  sendPort.send([url, receivePort.sendPort]);
// 接收到返回值， 返回给调用者
  return receivePort.first;
}
```

## **4.Flutter是怎么实现热重载的，原理和过程是怎么样的？**

* **答案：**

  Flutter热重载是基于`State`的，也就是我们在代码中经常出现的`setState`方法，通过这个来修改后，会执行相应的build方法，这就是热重载的基本过程。

实现源码在下面路径中，包含文件`run_cold.dart`和`run_hot.dart`两个文件，前者负责冷启动，后者负责热重载。

```dart
~/flutter/packages/flutter_tools/lib/src/run_hot.dart
```

#### 热重载实现过程：

代码在run\_hot.dart文件中，HotRunner负责具体代码执行。 当Flutter热重载时，调用restart函数，函数内部会传入一个fullRestart的bool类型变量。 热重载分为全量和非全量，fullRestart参数金牛是表示是否为全量。 以非全量热重载为例，函数的fullRestart会传入false，根据传入false参数，下面是哪部核心代码。

```dart
Future<OperationResult> restart(
    {bool fullRestart = false,
    bool pauseAfterRestart = false,
    String reason}) async {
  if (fullRestart) {
    // .....
  } else {
    final bool reloadOnTopOfSnapshot = _runningFromSnapshot;
    final String progressPrefix =
        reloadOnTopOfSnapshot ? 'Initializing' : 'Performing';
    final Status status = logger.startProgress('$progressPrefix hot reload...',
        progressId: 'hot.reload');
    OperationResult result;
    try {
      result = await _reloadSources(pause: pauseAfterRestart, reason: reason);
    } finally {
      status.cancel();
    }
  }
}
```

调用Restart函数后，内部会调用\_reloadSources函数，去执行内部逻辑。

执行流程图： ![image](https://github.com/ahyangnb/flutter_interview/blob/master/img/reload.png?raw=true)

图解：

在\_reloadSources函数内部，会调用\_updateDevFs函数，函数内部会扫描修改的文件，并将修改的文件进行对比，随后将被改动代码生成一个kernel files文件。 然后通过HTTP Server将生成的kernel files文件发送给Dart VM虚拟机，虚拟机拿到kernel文件后会调用\_reloadSources函数进行资源重载，将kernel文件注入正在运行的Dart VM中，当资源重载完成后，会调用RPC接口触发Widgets的重绘。

## **5.为什么说Flutter性能好？说下和其他跨平台的本质区别！**

* **答案：**

  > 与用于构建移动应用程序的其他大多数框架不同，Flutter是重写了一整套包括底层渲染逻辑和上层开发语言的完整解决方案。这样不仅可以保证视图渲染在Android和iOS上的高度一致性，在代码执行效率和渲染性能上也可以媲美原生App的体验。这，就是Flutter和其他跨平台方案的本质区别。

## **6.Flutter是怎么完成组件渲染的?**

* **答案：**

  在计算机系统中，图像的显示需要CPU、GPU和显示器一起配合完成CPU负责图像数据计算，GPU负责图像数据渲染，而显示器则负责最终图像显示。CPU把计算好的、需要显示的内容交给GPU，由GPU完成渲染后放入帧缓冲区，随后视频控制器根据垂直同步信号以每秒60次的速度，从帧缓冲区读取帧数据交由显示器完成图像显示。操作系统在呈现图像时遵循了这种机制。

而Flutter作为跨平台开发框架也采用了这种底层方案，UI线程使用Dart语言来构建视图结构数据，这些数据会在GPU线程进行图层合成，随后交给图像渲染引擎Skia加工成GPU数据，而这些数据会通过OpenGL最终提供给GPU渲染。

可以看到Flutter用了计算机最基本的图像渲染技术，摒弃其他一些通道和过程，用最直接的方式完成了图形显示，自然性能也就得到了保障。

#### [下一篇：Flutter每日一面（面试题二）](https://github.com/ahyangnb/flutter_interview/issues/2)

#### [Flutter每日一面目录大全](https://github.com/ahyangnb/flutter_interview)

