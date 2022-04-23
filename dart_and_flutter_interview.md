# Flutter和Dart知识点 
------
## Dart中的基本概念  
- 所有变量都是对象，也就是类的实例。数字，函数，null，都是对象，继承自Object类  
- Dart是强类型语言，支持类型推断，也可以使用dynamic类型
- Dart支持泛型
- Dart支持顶层函数和类成员变量
- Dart没有public,privite,protected关键字，使用下划线“_”开头标识只在库内可见。
- Dart内存回收策略：创建对象的时候只在堆上移动指针，内存增长始终是线形的，省去了查找可用内存的过程。
Dart中类似线程的概念教Isolate，每个Isolate之间无法共享内存，这种策略让Dart实现无锁的快速分配。

## Flutter相关  

### Flutter FrameWork  
- Framework最底层是Foundation,定义最基础的给其他层使用的工具类和方法。  
- Painting层封装了Flutter引擎的绘制接口  
- Animation层是动画相关的类，提供了类似Android ValueAnimator的功能  
- Gestures层提供了收拾识别相关功能，GestureBinding类是Flutter中处理收拾的抽象服务类，继承自BindingBase。Binding类在Flutter中充当着类似于Android中SystemService系列（ActivityManager,PackageManager）功能，每个Binding类都提供一个服务的单例对象，App最顶层的Binding会包含所有相关的Binding抽象类，如果使用Flutter提供的控件开发，则需要使用WidgetFlutterBinding,如果不适用Flutter提供的空间，则直接调用Render层，则需要使用RenderingFlutterBinding.  
- Rendering层，Flutter Widget树实际显示的时候会转换成对应的RenderObject树来实现布局和绘制。一般只会在调试布局或使用自定义控件实现某些特定效果才考虑RenderObject树的细节，RenderBinding是渲染树和Flutter Engine的胶水层，负责管理帧绘制，窗口尺寸，渲染参数变化的监听
- Widgets层提供了大量的空间，包括文本，图片，容器，输入框，动画等，Flutter中一切皆控件。Element是用来分离控件树和渲染树的中间层，Widget用来描述对应的Element属性，控件重建可能会复用用一个element，RenderObjectElement真正负责布局，绘制和碰撞测试的RenderObject.StateFulWidget和StateLessWidget并不会直接影响RenderObject的创建，它们只负责创建对应的RenderObjectWidget。如果Widget的属性发生了变化，因为控件的属性是只读的，所以意味着重新创建了新的控件树，但其树上每个节点没有变化时，element树和render树可以完全复用原来的对象，因为element和renderobject的属性可变。但是如果Widget的类型发生变化， 那element树和render树对应的节点也需要重新创建。

