## Flutter 架构

Flutter框架分三层
 Framework，Engine， Embedder

![fluttersystem](/imgs/fluttersystem.webp)

Framework使用dart语言实现，包括UI，文本，图片，按钮等Widgets，渲染，动画，手势等。此部分的核心代码是flutter仓库下的flutter package，以及sky_engine仓库下的 io, async, ui(dart:ui库提供了Flutter框架和引擎之间的接口)等package。

Engine使用C++实现，主要包括:Skia, Dart 和 Text。

- Skia是开源的二维图形库，提供了适用于多种软硬件平台的通用API。其已作为Google Chrome，Chrome OS，Android, Mozilla Firefox, Firefox OS等其他众多产品的图形引擎，支持平台还包括Windows, macOS, iOS，Android，Ubuntu等。
- Dart 部分主要包括:Dart Runtime，Garbage Collection(GC)，如果是Debug模式的话，还包括JIT(Just In Time)支持。Release和Profile模式下，是AOT(Ahead Of Time)编译成了原生的arm代码，并不存在JIT部分。
- Text 即文本渲染，其渲染层次如下:衍生自  Minikin的libtxt库(用于字体选择，分隔行)；HartBuzz用于字形选择和成型；Skia作为渲染/GPU后端，在Android和Fuchsia上使用FreeType渲染，在iOS上使用CoreGraphics来渲染字体。

Embedder是一个嵌入层，通过该层把Flutter嵌入到各个平台上去，Embedder的主要工作包括渲染Surface设置, 线程设置，以及插件等。平台(如iOS)只是提供一个画布，剩余的所有渲染相关的逻辑都在Flutter内部，这就使得它具有了很好的跨端一致性。

## Dart语言

Dart 也是一种VM语言，所以在每个运行flutter的app中都有一个dart的运行环境。编译模式支持AOT和JIT。
 Dart最开始是google设计出来替代javascript的，但是并没有凑效。后面Flutter选择了Dart, 才使Dart活跃起来。

Dart语言的特点：

- 单进程异步事件模型
- 强类型，可以类型推断
- 具有极高的运行效率和优秀的代码运行优化的VM，根据早前的基准测试，性能比肩 Java7 的JVM；
- 独特的隔离区( Isolate )，可以实现多线程
- 面向对象编程，一切数据类型均派生自 Object
- 运算符重载，泛型支持
- 强大的 Future 和 Stream 模型,可以简单实现高效的代码
- Minix 特性，可以更好的实现方法复用
- 全平台语言，可以很好的胜任移动和前后端的开发
- 在语法上，Dart 提供了很多便捷的操作

## Flutter线程管理

Flutter Engine自己不创建, 管理线程。Flutter Engine线程的创建和管理是由embedder负责的

Embeder提供四个Task Runner, 每个Task Runner负责不同的任务，Flutter Engine不在乎Task Runner具体跑在哪个线程，但是它需要线程配置在整一个生命周期里面保持稳定。也就是说一个Runner最好始终保持在同一线程运行

#### Platform Task Runner

是Flutter Engine的主Task Runner，运行Platform Task Runner的线程可以理解为是主线程。类似于Android Main Thread或者iOS的Main Thread。对于Flutter Engine来说Platform Runner所在的线程跟其它线程并没有实质上的区别。 可以同时启动多个Engine实例，每个Engine对应一个Platform Runner，每个Runner跑在各自的线程里。这也是Fuchsia（Google正在开发的操作引擎）里Content Handler的工作原理。一般情况下，一个Flutter应用启动的时候会创建一个Engine实例，Engine创建的时候会创建一个线程供Platform Runner使用。

跟Flutter Engine的所有交互（接口调用）必须发生在Platform Thread，试图在其它线程中调用Flutter Engine会导致无法预期的异常。这跟Android和IOS对于UI的操作都必须在主线程进行相类似。需要注意的是在Flutter Engine中有很多模块都是非线程安全的。一旦引擎正常启动运行起来，所有引擎API调用都将在Platform Thread里发生。

Platform Runner所在的Thread不仅仅处理与Engine交互，它还处理来自平台的消息。这样的处理比较方便的，因为几乎所有引擎的调用都只有在Platform Thread进行才能是安全的，Native Plugins不必要做额外的线程操作就可以保证操作能够在Platform Thread进行。如果Plugin自己启动了额外的线程，那么它需要负责将返回结果派发回Platform Thread以便Dart能够安全地处理。规则很简单，对于Flutter Engine的接口调用都需保证在Platform Thread进行。

阻塞Platform Thread不会直接导致Flutter应用的卡顿（跟iOS android主线程不同）。尽管如此，平台对Platform Thread还是有强制执行限制。所以建议复杂计算逻辑操作不要放在Platform Thread而是放在其它线程（不包括我们现在讨论的这个四个线程）。其他线程处理完毕后将结果转发回Platform Thread。长时间卡住Platform Thread应用有可能会被系统Watchdot强行杀死。

#### UI Task Runner

Flutter Engine用于执行Dart root isolate代码。Root isolate比较特殊，它绑定了不少Flutter需要的函数方法。Root isolate运行应用的main code。引擎启动的时候为其增加了必要的绑定，使其具备调度提交渲染帧的能力。

1. 对于每一帧，引擎要做的事情有：
2. Root isolate通知Flutter Engine有帧需要渲染。
3. Flutter Engine通知平台，需要在下一个vsync的时候得到通知。
4. 平台等待下一个vsync
5. 对创建的对象和Widgets进行Layout并生成一个Layer Tree，Layer Tree马上被提交给Flutter Engine。当前阶段没有进行任何光栅化，这个步骤仅是生成了对需要绘制内容的描述。
6. 创建或者更新Tree，这个Tree包含了用于屏幕上显示Widgets的语义信息。这个东西主要用于平台相关的辅助Accessibility元素的配置和渲染。

除了渲染相关逻辑之外Root Isolate还是处理来自Native Plugins的消息响应，Timers，MicroTasks和异步IO。
 Root Isolate负责创建管理的Layer Tree最终决定什么内容要绘制到屏幕上。因此这个线程的过载会直接导致卡顿掉帧。
 如果确实有无法避免的繁重计算，建议将其放到独立的Isolate去执行，比如使用compute关键字或者放到非Root Isolate，这样可以避免应用UI卡顿。但是需要注意的是非Root Isolate缺少Flutter引擎需要的一些函数绑定，你无法在这个Isolate直接与Flutter Engine交互。所以只在需要大量计算的时候采用独立Isolate。

#### GPU Task Runner

用于执行设备GPU的相关调用。UI Task Runner创建的Layer Tree信息是平台不相关，也就是说Layer Tree提供了绘制所需要的信息，具体如何实现绘制取决于具体平台和方式，可以是OpenGL，Vulkan，软件绘制或者其他Skia配置的绘图实现。GPU Task Runner中的模块负责将Layer Tree提供的信息转化为实际的GPU指令。GPU Task Runner同时也负责配置管理每一帧绘制所需要的GPU资源，这包括平台Framebuffer的创建，Surface生命周期管理，保证Texture和Buffers在绘制的时候是可用的。

基于Layer Tree的处理时长和GPU帧显示到屏幕的耗时，GPU Task Runner可能会延迟下一帧在UI Task Runner的调度。一般来说UI Runner和GPU Runner跑在不同的线程。存在这种可能，UI Runner在已经准备好了下一帧的情况下，GPU Runner却还正在向GPU提交上一帧。这种延迟调度机制确保不让UI Runner分配过多的任务给GPU Runner。

GPU Runner可以导致UI Runner的帧调度的延迟，GPU Runner的过载会导致Flutter应用的卡顿。一般来说用户没有机会向GPU Runner直接提交任务，因为平台和Dart代码都无法跑进GPU Runner。但是Embeder还是可以向GPU Runner提交任务的。因此建议为每一个Engine实例都新建一个专用的GPU Runner线程。

#### IO Task Runner

主要功能是从图片存储（比如磁盘）中读取压缩的图片格式，将图片数据进行处理为GPU Runner的渲染做好准备。在Texture的准备过程中，IO Runner首先要读取压缩的图片二进制数据（比如PNG，JPEG），将其解压转换成GPU能够处理的格式然后将数据上传到GPU。这些复杂操作如果跑在GPU线程的话会导致Flutter应用UI卡顿。但是只有GPU Runner能够访问GPU，所以IO Runner模块在引擎启动的时候配置了一个特殊的Context，这个Context跟GPU Runner使用的Context在同一个ShareGroup。事实上图片数据的读取和解压是可以放到一个线程池里面去做的，但是这个Context的访问只能在特定线程才能保证安全。这也是为什么需要有一个专门的Runner来处理IO任务的原因。获取诸如ui.Image这样的资源只有通过async call，当这个调用发生的时候Flutter Framework告诉IO Runner进行刚刚提到的那些图片异步操作。这样GPU Runner可以使用IO Runner准备好的图片数据而不用进行额外的操作。

用户操作，无论是Dart Code还是Native Plugins都是没有办法直接访问IO Runner。尽管Embeder可以将一些一般复杂任务调度到IO Runner，这不会直接导致Flutter应用卡顿，但是可能会导致图片和其它一些资源加载的延迟间接影响性能。所以建议为IO Runner创建一个专用的线程

> android & iOS平台上面每一个Engine实例启动的时候会为UI，GPU，IO Runner各自创建一个新的线程。所有Engine实例共享同一个Platform Runner线程

## isolate



![isolated](/imgs/flutterisolated.webp)

An isolated Dart execution context

isolate是Dart对actor并发模式的实现。运行中的Dart程序由一个或多个actor组成，actor也就是Dart概念里面的isolate。isolate是隔离的，每个isolate有自己的内存和单线程运行的实体. isolate之间不互相共享内存，且独立GC。
 isolate中的代码是顺序执行的，且是单线程，所以不存在资源竞争和变量状态同步的问题，也就不需要锁。Dart中的并发都是多个isolate并行实现的

由于isolate不共享内存，所以isolate之间不能直接互相通信，只能通过Port进行通信，而且是异步的

## Flutter Engine Runners与Dart Isolate

Dart的Isolate是Dart虚拟机自己管理的，Flutter Engine无法直接访问。Root Isolate通过Dart的C++调用能力把UI渲染相关的任务提交到UI Runner执行, 这样就可以跟Flutter Engine相关模块进行交互，Flutter UI相关的任务也被提交到UI Runner也可以相应的给Isolate一些事件通知，UI Runner同时也处理来自App方面Native Plugin的任务。 Dart isolate跟Flutter Runner是相互独立的，它们通过任务调度机制相互协作。

## Dart内存管理

Dart VM将内存管理分为新生代(New Generation)和老年代(Old Generation)

- 新生代：初次分配的对象都位于新生代中，该区域主要是存放内存较小并且生命周期较短的对象，比如局部变量。新生代会频繁执行内存回收(GC)，回收采用“复制-清除”算法，将内存分为两块，运行时每次只使用其中的一块，另一块备用。当发生GC时，将当前使用的内存块中存活的对象拷贝到备用内存块中，然后清除当前使用内存块，最后，交换两块内存的角色。
- 老年代: 在新生代的GC中“幸存”下来的对象，它们会被转移到老年代中。老年代存放生命力周期较长，内存较大的对象。老年代的GC回收采用“标记-清除”算法，分成标记和清除两个阶段。在标记阶段会触发停顿，多线程并发的完成对垃圾对象的标记，降低标记阶段耗时。在清理阶段，由GC线程负责清理回收对象，和应用线程同时执行，不影响应用运行。

## Flutter中的image所占的内存

Android将中内存分java内存或native内存，通常在代码中的申请的内存都在这两个范围内

java内存是指java或kotlin分配的内存对象
 native内存是指由C/C++中分配的内存，也包括一些android原生系统占用的内存，如图像资源和其他图形等

Flutter中的image占用的不用这两种内存，而是Graphics内存，Graphics内存内存是指图形缓冲区队列向屏幕显示像素所使用的内存，图形缓冲区是指GL表面，GL纹理等。Graphics内存是与CPU共享的内存，而不是GPU专用的内存

## Flutter运行模式

Flutter常见的种运行模式：Debug，Release和Profile

Release和Profile模式比较类似，不用之处在于Profile模式的服务扩展的支持，支持跟踪，以及最小化使用跟踪信息需要的依赖。Profile并不支持模拟器，原因在于模拟器上的诊断并不代表真实的性能。所有重点截介绍
 Debug和Release的差异

- Debug模式：使用JIT编译，支持模拟器和设备。打开了断言支持，包括所有的调试信息，服务扩展和Observatory等调试辅助。此模式为快速开发和运行做了优化，但并未对执行速度，包大小和部署做优化。
   所以能实现秒级别的hot reload
- Release模式：使用AOT编译，只支持真机，不支持模拟器。关闭了所有断言，尽可能多地去掉了调试信息，关闭了所有调试工具。为快速启动，快速执行，包大小做了优化。禁止了所有调试辅助手段，服务扩展。

## Flutter Platform Channel

Platform Channel用来实现flutter和Native之间的通讯，实现方式类似远程通讯。

Flutter定义了三种Channel：

- BasicMessageChannel：用于传递字符串和半结构化的信息
- MethodChannel：用于传递方法调用（method invocation）
- EventChannel: 用于数据流（event streams）的通信

这三种channel的工作原理都一致，都用三个基本的属性：

- name:  String类型，代表Channel的名字，也是其唯一标识符
- Messager：BinaryMessenger类型，代表消息信使，是消息的发送与接收的工具
- codec: MessageCodec类型或MethodCodec类型，代表消息的编解码器

BinaryMessenger是Native端与Flutter端通信的工具，其通信使用的消息格式为二进制格式数据。初始化一个Channel，并向该Channel注册处理消息的Handler时，实际上会生成一个与之对应的BinaryMessageHandler，并以channel name为key，注册到BinaryMessenger中。当Flutter端发送消息到BinaryMessenger时，BinaryMessenger会根据其入参channel找到对应的BinaryMessageHandler，并交由其处理。

BinaryMessenger只和BinaryMessageHandler通讯。而Channel和BinaryMessageHandler则是一一对应的。由于Channel从BinaryMessageHandler接收到的消息是二进制格式数据，无法直接使用，故Channel会将该二进制消息通过Codec（消息编解码器）解码为能识别的消息并传递给Handler进行处理。

当Handler处理完消息之后，会通过回调函数返回result，并将result通过编解码器编码为二进制格式数据，通过BinaryMessenger发送回Flutter端。

Codec：息编解码器，主要用来将二进制格式的数据转化为Handler能够识别的数据，Flutter定义了两种Codec：MessageCodec和MethodCodec

MessageCodec用于二进制格式数据与基础数据之间的编码和解码。有多重实现如：BinaryCodec， StringCodec， JSONMessageCodec等

MethodCodec用于二进制数据与方法调用(MethodCall)和返回结果之间的编解码。MethodChannel和EventChannel所使用的编解码器均为MethodCodec。

MethodCodec用于MethodCall对象的编解码，一个MethodCall对象代表一次从Flutter端发起的方法调用。MethodCall有2个成员变量：String类型的method代表需要调用的方法名称，通用类型(Android中为Object，iOS中为id)的arguments代表需要调用的方法入参。

由于处理的是方法调用，MethodCodec多了对调用结果的处理。当方法调用成功时，使用encodeSuccessEnvelope将result编码为二进制数据，而当方法调用失败时，则使用encodeErrorEnvelope将error的code、message、detail编码为二进制数据。

MethodCodec有两种实现：JSONMethodCodec和StandardMethodCodec

由于Platform Channel运行在flutter App的UI Task Runner, 对应的native实现运行在Platform Task Runner，而Platform Task Runner运行在主线程，所以在native实现是不能进行耗时的操作，且Platform Task Runner是非线程安全的，所以要保证回调函数在主线程中执行

Platform Channel支持大数据传递，传递大内存数据块时，使用BasicMessageChannel以及BinaryCodec。而整个数据传递的过程中，唯一可能出现数据拷贝的位置为native二进制数据转化为Dart语言二进制数据。若二进制数据大于阈值时（目前阈值为1000byte）则不会拷贝数据，直接转化，否则拷贝一份再转化。



## flutter中的widget

万物皆widget

目前主流的思想，都希望将各个ui控件接耦，慢慢演变出组件化的思想。

Flutter控件主要分为两大类，StatelessWidget和StatefulWidget，StatelessWidget用来展示静态的文本或者图片，如果控件需要根据外部数据或者用户操作来改变的话，就需要使用StatefulWidget。State的概念也是来源于Facebook的流行Web框架[React](http://facebook.github.io/react/)，React风格的框架中使用控件树和各自的状态来构建界面，当某个控件的状态发生变化时由框架负责对比前后状态差异并且采取最小代价来更新渲染结果。

widget效果见这两篇文章 [widget 应用一](https://blog.csdn.net/qq_38366777/article/details/83089782)  [widget 应用二](https://blog.csdn.net/qq_38366777/article/details/83108977)

接下来主要看下渲染树和flutter引擎绑定在一起，看一下binding.dart、object.dart，分析下flutter是如何渲染的

```
abstract class RendererBinding extends BindingBase with ServicesBinding, SchedulerBinding, HitTestable { ... }
abstract class RenderObject extends AbstractNode with DiagnosticableTreeMixin implements HitTestTarget {
abstract class RenderBox extends RenderObject { ... }
class RenderParagraph extends RenderBox { ... }
class RenderImage extends RenderBox { ... }
class RenderFlex extends RenderBox with ContainerRenderObjectMixin<RenderBox, FlexParentData>,
                                        RenderBoxContainerDefaultsMixin<RenderBox, FlexParentData>,
                                        DebugOverflowIndicatorMixin { ... }
```

 

创建[RenderView]对象作为[RenderObject]呈现树的根，并对其进行初始化，以便在引擎下次准备显示一个框架时将其呈现。

创建绑定是自动调用

每个框架由以下几个阶段组成:

1. The animation phase:

2. Microtasks: 

3. The layout phase: 布局阶段:

布局阶段:系统中所有脏的[RenderObject]都被放置(见[RenderObject.performLayout])。[RenderObject.markNeedsLayout]

有关为布局标记脏对象的详细信息

4. The compositing bits phase: 合成部分阶段

5. The paint phase:绘制阶段:

系统中所有的渲染对象都是重新绘制(见[RenderObject.paint])。这将生成[层]树

6. The compositing phase:

合成阶段:将图层树转换为[场景]和发送到GPU。

7. The semantics phase:语义阶段

8. The finalization phase:结束阶段

看一下代码当中的具体执行 

```
  @protected
  void drawFrame() {
    assert(renderView != null);
    pipelineOwner.flushLayout(); //1
    pipelineOwner.flushCompositingBits();//2
    pipelineOwner.flushPaint();//3
    renderView.compositeFrame(); // this sends the bits to the GPU
    pipelineOwner.flushSemantics(); // this also sends the semantics to the OS. //4
  }
```

1。更新任何需要计算它们的渲染对象布局。在这个阶段，每个渲染的大小和位置计算对象。渲染对象可能会弄脏他们的画或这一阶段的合成状态。

2。更新有dirty的渲染对象合成部分。在这个阶段，每个呈现对象都知道是否它的任何一个孩子都需要合成。此信息在期间使用

绘画阶段选择如何实现视觉效果等剪裁。如果呈现对象有一个合成的子对象，它需要使用一个

[层]来创建剪辑，以便该剪辑应用于合成子元素(将被绘制到它自己的[图层]中)。

3.访问任何需要绘制的渲染对象。在这阶段，渲染对象有机会记录绘制命令[图片图层]，并构建其他合成的[图层]。

4.将编译呈现对象的语义。这个语义信息被使用辅助技术，以改善渲染树的可访问性。

 

![img](/imgs/fluttergpu.png)

 

 

![img](/imgs/jianzhuchicun.png)

渲染对象树中的每个对象都会在布局过程中接受父对象的`Constraints`参数，决定自己的大小，然后父对象就可以按照自己的逻辑决定各个子对象的位置，完成布局过程。子对象不存储自己在容器中的位置，所以在它的位置发生改变时并不需要重新布局或者绘制。子对象的位置信息存储在它自己的`parentData`字段中，但是该字段由它的父对象负责维护，自身并不关心该字段的内容。同时也因为这种简单的布局逻辑，Flutter可以在某些节点设置布局边界（Relayout boundary），即当边界内的任何对象发生重新布局时，不会影响边界外的对象，反之亦然：

![img](/imgs/layoutbianjie.webp)

布局完成后，渲染对象树中的每个节点都有了明确的尺寸和位置，Flutter会把所有对象绘制到不同的图层上：

![img](/imgs/chicunweizhi.png)

因为绘制节点时也是深度遍历，可以看到第二个节点在绘制它的背景和前景不得不绘制在不同的图层上，因为第四个节点切换了图层（因为“4”节点是一个需要独占一个图层的内容，比如视频），而第六个节点也一起绘制到了红色图层。这样会导致第二个节点的前景（也就是“5”）部分需要重绘时，和它在逻辑上毫不相干但是处于同一图层的第六个节点也必须重绘。为了避免这种情况，Flutter提供了另外一个“重绘边界”的概念：

在进入和走出重绘边界时，Flutter会强制切换新的图层，这样就可以避免边界内外的互相影响。典型的应用场景就是ScrollView，当滚动内容重绘时，一般情况下其他内容是不需要重绘的。虽然重绘边界可以在任何节点手动设置，但是一般不需要我们来实现，Flutter提供的控件默认会在需要设置的地方自动设置

## widget总结

在Flutter界面渲染过程分为三个阶段：布局、绘制、合成，布局和绘制在Flutter框架中完成，合成则交由引擎负责。

![img](/imgs/widgetzongjie.png)