---
description: Dart 语言的特性，Dart 语言重要概念，mixin extends implement 之间关系，mixins的条件，mixin 指定异常类型。
---

# Flutter每日一面（面试题三）



* **1.Dart 语言的特性是怎么样的？**

  **答案：**

* Productive（生产力高，Dart的语法清晰明了，工具简单但功能强大）
* Fast（执行速度快，Dart提供提前优化编译，以在移动设备和Web上获得可预测的高性能和快速启动。）
* Portable（易于移植，Dart可编译成ARM和X86代码，这样Dart移动应用程序可以在iOS、Android和其他地方运行）
* Approachable（容易上手，充分吸收了高级语言特性，如果你已经知道C++，C语言，或者Java，你可以在短短几天内用Dart来开发）
* Reactive（响应式编程）
* **2.Dart 语言有哪些重要的概念？**

  **答案：**

* 在Dart中，一切都是对象，所有的对象都是继承自Object
* Dart是强类型语言，但可以用var或 dynamic来声明一个变量，Dart会自动推断其数据类型,dynamic类似c\#
* 没有赋初值的变量都会有默认值null
* Dart支持顶层方法，如main方法，可以在方法内部创建方法
* Dart支持顶层变量，也支持类变量或对象变量
* Dart没有public protected private等关键字，如果某个变量以下划线（\_）开头，代表这个变量在库中是私有的
* **3.说下mixin extends implement 之间的关系?**

  **答案：**

  > 继承（关键字 extends）、混入 mixins （关键字 with）、接口实现（关键字 implements）。这三者可以同时存在，前后顺序是`extends -> mixins -> implements`。

> Flutter中的继承是单继承，子类重写超类的方法要用`@Override`，子类调用超类的方法要用super。
>
> 在Flutter中，Mixins是一种在多个类层次结构中复用类代码的方法。mixins的对象是类，mixins绝不是继承，也不是接口，而是一种全新的特性，可以mixins多个类，mixins的使用需要满足一定条件。

* **4.使用mixins的条件是什么？**

  **答案：**

  随着Dart版本一直在变，这里讲的是Dart2.1中使用mixins的条件：

* mixins类只能继承自object
* mixins类不能有构造函数
* 一个类可以mixins多个mixins类
* 可以mixins多个类，不破坏Flutter的单继承
* **5.mixin 怎么指定异常类型？**

  **答案：**

on关键字可用于指定异常类型。 on只能用于被mixins标记的类，例如mixins X on A，意思是要mixins X的话，得先接口实现或者继承A。这里A可以是类，也可以是接口，但是在mixins的时候用法有区别.

on 一个类：

```dart
class A {
  void a(){
    print("a");
  }
}


mixin X on A{
  void x(){
    print("x");
  }
}


class mixinsX extends A with X{
}
```

on 的是一个接口： 得首先实现这个接口，然后再用mix

```dart
class A {
  void a(){
    print("a");
  }
}

mixin X on A{
  void x(){
    print("x");
  }
}

class implA implements A{
  @override
  void a() {}
}

class mixinsX2 extends implA with X{
}
```

#### [下一篇：Flutter每日一面（面试题四）](https://github.com/ahyangnb/flutter_interview/issues/4)

#### [Flutter每日一面目录大全](https://github.com/ahyangnb/flutter_interview)

