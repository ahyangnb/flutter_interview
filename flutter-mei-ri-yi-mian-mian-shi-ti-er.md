---
description: >-
  Flutter绘制流程，Widget 和 element 和 RenderObject
  之间的关系，特殊方法的执行顺序，Future和Isolate区别，Stream订阅模式，await for的使用，Widget、State、Context
  的核心概念，key和Navigator。
---

# Flutter每日一面（面试题二）



* **1.Flutter绘制流程是怎么样的？**

  **答案：**

  > Flutter只关心向 GPU提供视图数据，GPU的 VSync信号同步到 UI线程，UI线程使用 Dart来构建抽象的视图结构，这份数据结构在 GPU线程进行图层合成，视图数据提供给 Skia引擎渲染为 GPU数据，这些数据通过 OpenGL或者 Vulkan提供给 GPU。

所以 Flutter并不关心显示器、视频控制器以及 GPU具体工作，它只关心 GPU发出的 VSync信号，尽可能快地在两个 VSync信号之间计算并合成视图数据，并且把数据提供给 GPU。

![image](https://github.com/ahyangnb/flutter_interview/blob/master/img/huizhi.jpg?raw=true)

* **2.说下Widget 和 element 和 RenderObject 之间的关系？**

  **答案：**

* Widget是用户界面的一部分,并且是不可变的。
* Element是在树中特定位置Widget的实例。
* RenderObject是渲染树中的一个对象，它的层次结构是渲染库的核心。

Widget会被inflate（填充）到Element，并由Element管理底层渲染树。Widget并不会直接管理状态及渲染,而是通过State这个对象来管理状态。Flutter创建Element的可见树，相对于Widget来说，是可变的，通常界面开发中，我们不用直接操作Element,而是由框架层实现内部逻辑。就如一个UI视图树中，可能包含有多个TextWidget\(Widget被使用多次\)，但是放在内部视图树的视角，这些TextWidget都是填充到一个个独立的Element中。Element会持有renderObject和widget的实例。记住，Widget 只是一个配置，RenderObject 负责管理布局、绘制等操作。

在第一次创建 Widget 的时候，会对应创建一个 Element， 然后将该元素插入树中。如果之后 Widget 发生了变化，则将其与旧的 Widget 进行比较，并且相应地更新 Element。重要的是，Element 不会被重建，只是更新而已。

* **3.Flutter main future mirotask 的执行顺序是怎么样的?**

  **答案：**

  > 普通代码都是同步执行的，结束后会开始检查microtask中是否有任务，若有则执行，执行完继续检查microtask，直到microtask列队为空。最后会去执行event队列（future）。

* **4.Future和Isolate有什么区别？**

  **答案：**

  > future是异步编程，调用本身立即返回，并在稍后的某个时候执行完成时再获得返回结果。在普通代码中可以使用await 等待一个异步调用结束。

> isolate是并发编程，Dartm有并发时的共享状态，所有Dart代码都在isolate中运行，包括最初的main\(\)。每个isolate都有它自己的堆内存，意味着其中所有内存数据，包括全局数据，都仅对该isolate可见，它们之间的通信只能通过传递消息的机制完成，消息则通过端口\(port\)收发。isolate只是一个概念，具体取决于如何实现，比如在Dart VM中一个isolate可能会是一个线程，在Web中可能会是一个Web Worker。

* **5.Stream 与 Future是什么关系？**

  **答案：**

  > Stream 和 Future 是 Dart 异步处理的核心 API。Future 表示稍后获得的一个数据，所有异步的操作的返回值都用 Future 来表示。但是 Future 只能表示一次异步获得的数据。而 Stream 表示多次异步获得的数据。比如界面上的按钮可能会被用户点击多次，所以按钮上的点击事件（onClick）就是一个 Stream 。简单地说，Future将返回一个值，而Stream将返回多次值。Dart 中统一使用 Stream 处理异步事件流。Stream 和一般的集合类似，都是一组数据，只不过一个是异步推送，一个是同步拉取。

* **6.Stream 有哪两种订阅模式？分别是怎么调用的？**

  **答案：**

  Stream有两种订阅模式：单订阅\(single\) 和 多订阅（broadcast）。单订阅就是只能有一个订阅者，而广播是可以有多个订阅者。这就有点类似于消息服务（Message Service）的处理模式。单订阅类似于点对点，在订阅者出现之前会持有数据，在订阅者出现之后就才转交给它。而广播类似于发布订阅模式，可以同时有多个订阅者，当有数据时就会传递给所有的订阅者，而不管当前是否已有订阅者存在。

Stream 默认处于单订阅模式，所以同一个 stream 上的 listen 和其它大多数方法只能调用一次，调用第二次就会报错。但 Stream 可以通过 transform\(\) 方法（返回另一个 Stream）进行连续调用。通过 Stream.asBroadcastStream\(\) 可以将一个单订阅模式的 Stream 转换成一个多订阅模式的 Stream，isBroadcast 属性可以判断当前 Stream 所处的模式。

* **7.await for 如何使用?**

  **答案：**

  > await for是不断获取stream流中的数据，然后执行循环体中的操作。它一般用在直到stream什么时候完成，并且必须等待传递完成之后才能使用，不然就会一直阻塞。

```dart
Stream<String> stream = new Stream<String>.fromIterable(['不开心', '面试', '没', '过']);
main() async{
  print('上午被开水烫了脚');
  await for(String s in stream){
    print(s);
  }
  print('晚上还没吃饭');
}
```

* **8.Flutter中的Widget、State、Context 的核心概念？是为了解决什么问题？**

  **答案：**

* Widget: 在Flutter中，几乎所有东西都是Widget。将一个Widget想象为一个可视化的组件（或与应用可视化方面交互的组件），当你需要构建与布局直接或间接相关的任何内容时，你正在使用Widget。
* Widget树: Widget以树结构进行组织。包含其他Widget的widget被称为父Widget\(或widget容器\)。包含在父widget中的widget被称为子Widget。
* Context: 仅仅是已创建的所有Widget树结构中的某个Widget的位置引用。简而言之，将context作为widget树的一部分，其中context所对应的widget被添加到此树中。一个context只从属于一个widget，它和widget一样是链接在一起的，并且会形成一个context树。
* State: 定义了StatefulWidget实例的行为，它包含了用于”交互/干预“Widget信息的行为和布局。应用于State的任何更改都会强制重建Widget。

> 这些状态的引入，主要是为了解决多个部件之间的交互和部件自身状态的维护。

* **9.Widget分别有哪两种状态类？**

  **答案：**

  > StatelessWidget: 一旦创建就不关心任何变化，在下次构建之前都不会改变。它们除了依赖于自身的配置信息（在父节点构建时提供）外不再依赖于任何其他信息。比如典型的Text、Row、Column、Container等，都是StatelessWidget。它的生命周期相当简单：初始化、通过build\(\)渲染。

> StatefulWidget: 在生命周期内，该类Widget所持有的数据可能会发生变化，这样的数据被称为State，这些拥有动态内部数据的Widget被称为StatefulWidget。比如复选框、Button等。State会与Context相关联，并且此关联是永久性的，State对象将永远不会改变其Context，即使可以在树结构周围移动，也仍将与该context相关联。当state与context关联时，state被视为已挂载。StatefulWidget由两部分组成，在初始化时必须要在createState\(\)时初始化一个与之相关的State对象。

* **10.Widget 唯一标识Key有那几种？**

  **答案：**

  > 在flutter中，每个widget都是被唯一标识的。这个唯一标识在build或rendering阶段由框架定义。该标识对应于可选的Key参数，如果省略，Flutter将会自动生成一个。

> 在flutter中，主要有4种类型的Key：GlobalKey（确保生成的Key在整个应用中唯一，是很昂贵的，允许element在树周围移动或变更父节点而不会丢失状态）、LocalKey、UniqueKey、ObjectKey。

* **11.Navigator是什么，为什么Navigator可以实现无需上下文路由导航？**

  **答案：**

  > Navigator是在Flutter中负责管理维护页面堆栈的导航器。MaterialApp在需要的时候，会自动为我们创建Navigator。Navigator.of\(context\)，会使用context来向上遍历Element树，找到MaterialApp提供的NavigatorState再调用其push/pop方法完成导航操作。

> 所以如果在MaterialApp的navigatorKey属性内设置好一个Key就可以直接使用这个Key来进行路由导航，无需上下文。

#### [下一篇：Flutter每日一面（面试题三）](https://github.com/ahyangnb/flutter_interview/issues/3)

#### [Flutter每日一面目录大全](https://github.com/ahyangnb/flutter_interview)

