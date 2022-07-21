# [Unity项目开发过程中常见的问题，你遇到过吗？ ](https://www.cnblogs.com/murongxiaopifu/p/9833395.html)

最近看到有朋友问一个unity游戏开发团队，需要掌握哪些知识之类的问题。事实上Unity引擎是一个很灵活的引擎，根据团队开发游戏类型的不同，对人员的要求也有差异，所以不能一概而论。但是，一些在Unity项目开发过程中常常会遇到的问题还是可以总结一下的。

下面我就来聊聊实际工作中，一个项目组可能会遇到的问题吧。

## 0x01.项目前期规划时的问题

这里指的不是策划的需求或者游戏玩法的计划，而是作为一个Unity项目我们需要在一开始明确并制定好的规范和标准。作为一个Unity项目或软件项目，这部分是很重要的，因为项目早期的规划随着项目的开发时间越久，就越难以修改。



**对支持的最低机型不明确**
作为一个Unity项目，我们首先要明确我们所需要支持的最低设备标准。并且项目组要有这样的设备，以供开发和QA团队使用。
否则，对项目的优化将无从谈起。


**资源标准不明确**
开发过Unity项目的同学可能都有过类似的经历，即开发过程中的资源标准不明确。这常常也是早期在项目规划时没有重视资源标准导致的。
所以在项目的早期阶段，最好能够明确资源标准比如模型的vertex数量、纹理资源的尺寸格式等等。
也要对每帧开销中，脚本和渲染所花费的时间有一个目标和预期。


**没有合理的Asset流水线**
这里指的是资源应该按照一定的流程和标准从美术那里导入到Unity项目中。
很多项目最终出现性能问题，都是由于没有一个合理的Asset流水线。从而导致项目内的资源标准无法管理，很多冗余或不符合目标设备水平的资源构建进了最终的安装包里。
所以，作为项目组，大家一定要指定一套自动化的Asset流水线。为Asset的规格和标准制定明确的规范，在自动化脚本中进行设置。
例如texture是否开启read/write?texture的压缩格式？尺寸？非人形的model在导入时是否关闭了rigs？动画模型是否开启了Optimize Game Object选项？等等。


**没有合理的构建和QA流程**
也有很多项目的构建并非一番风顺，构建的版本也难以管理。策划或者QA常常是找负责某个功能的开发兵荒马乱的打一个包出来。
所以项目组可以思考一下下面的几个问题：
是否有专门的打包机？
一个新的功能是如何发布到最终的发布版本的？
是否有自动化的可持续集成设施（CI）？
QA要如何反馈Bug，Bug如何有效的管理？


**正式项目直接在Demo原型上进行开发**
这个也是一个常见的情况，有些项目组早期会有少数几个人开发一些玩法演示Demo，Demo被认可之后开始开发正式的项目。
此时会有一个问题，即在Demo的基础上直接开发正式项目。由于很多Demo只是为了演示玩法，所以代码中有很多为了尽快实现需求的特定Hack。
如果正式项目以此为基础，到后期维护会比较麻烦。



除了上面所提到的问题之外，还有一些别的需要重视的内容，例如制定统一的编码规范、确定采用的光照模式（RealTime？Mixed？Baked？） 等等。

 

## 0x02.项目开发过程中的问题

经过了项目早期的规划阶段，来到项目的开发阶段时项目组有可能会犯哪些错误呢？一些不好的实践有可能会拖慢项目的开发进度以及让项目组成员的焦躁感上升。


**不重视版本管理**
很多团队对版本管理不重视，或者团队内部对版本管理例如git的操作不熟练。
当然，关于git的最佳实践的资料有很多，建议项目组在内部进行培训和普及，让大家（程序美术策划etc）对版本管理的操作符合规范。
针对Unity项目，serialization 的格式建议设置为text serialization。
设置commit hook：

https://github.com/3pjgames/unity-git-hooks


**静态数据存储在Json或XML文件保存**
不少团队喜欢或习惯于使用Json文件或XML文件来保存一些静态数据，在游戏运行的时候加载使用。但是使用Json或XML文件保存数据会有以下的问题：

- 加载速度慢。
- Parse的时候会产生内存开销。

所以数据最好使用二进制来保存，在Unity内部也提供了ScriptableObject来帮助保存数据。


**项目中包含了没有用到的资源、插件或冗余的库**
这也是很多团队中常见的一个问题。一些废弃的资源没有及时处理，仍然留在项目中甚至被构建进入了最后的发布版本，从而造成不必要的开销。
另一个问题是冗余或多份同样功能的库，比如项目中使用的插件中有多款插件都使用到了Json解析库，那么就会造成冗余。

 

**只在Editor中测试性能**

这是一个很不好的开发习惯。因为在Editor中测试的是Editor的性能开销，而不是在真正的目标平台上的性能开销。所以在做Profile的时候，一定要在目标平台的设备上进行。否则只能得到让人误会的数据，例如在Editor中，GetComponent这个API会产生堆内存的分配，但是在真机上并不会产生堆内存的开销。

详情可以查看：

https://zhuanlan.zhihu.com/p/26763624

[ ](https://zhuanlan.zhihu.com/p/26763624)

当然，更可怕的是真正的性能瓶颈被隐藏了，这样只会让Profile变成一件浪费时间而又没有收益的事情。

 

**开发者不了解Profiler工具**

在开发Unity项目的过程中，常用的Profiler工具主要包括以下几种：

Unity提供的：

- Unity自带的Profiler：

[https://unity3d.com/cn/learn/tutorials/topics/performance-optimizationunity3d.com](https://link.zhihu.com/?target=https%3A//unity3d.com/cn/learn/tutorials/topics/performance-optimization)

- Unity内置的Frame Debugger
- Memory Profiler：

[Unity-Technologies / MemoryProfiler - Bitbucketbitbucket.org](https://link.zhihu.com/?target=https%3A//bitbucket.org/Unity-Technologies/memoryprofiler)

移动平台相关的：

- Xcode Instruments
- Android Studio
- Mali Graphics Debugger
- Snapdragon Profiler
- Renderdoc

 

**在项目后期才进行性能分析和优化**

在开发的过程中就应该关注项目的性能问题，在日常的工作中就应该重视性能分析，而不是等到了项目后期甚至是deadline前才进行。

一方面是能做到对项目的性能问题心中有数，不会在后期手忙脚乱。

另一方面能够及时发现问题，修正工作流程，不至于“开发债务”越积累越多。

 

## 0x03.写代码时的问题 

**开发者不了解Unity脚本的生命周期**

有一些开发者由于不了解Unity脚本的生命周期，而出现一些程序上的错误和问题。

例如引擎什么时候会调用Awake、OnEnable、Update等等API？协程在什么时候会被更新？FixedUpdate又是怎么执行的？

可以参考Unity的文档：

https://docs.unity3d.com/Manual/ExecutionOrder.html

 

FixedUpdate的执行可以查看：

https://zhuanlan.zhihu.com/p/30335370[ ](https://zhuanlan.zhihu.com/p/30335370)

 

**过于重度的使用MonoBehavior以及Update**

有些开发者喜欢在制作Demo甚至时正式开发项目的时候大量的依赖MonoBehavior以及Update方法。在Unity中MonoBehavior脚本的Update方法会被引擎记录在一个List中，在运行时引擎会遍历这个List，并调用其中的Update方法以实现脚本逻辑的更新。但是，原生代码到托管代码的调用总是会有性能上的开销的，因此场景中的MonoBehavior以及Update过多会影响游戏的性能。

相关文档可以参考：

https://blogs.unity3d.com/cn/2015/12/23/1k-update-calls/[ ](https://link.zhihu.com/?target=https%3A//blogs.unity3d.com/cn/2015/12/23/1k-update-calls/)

 

**没有对需要频繁访问的数据进行缓存**

这也是一个初学Unity的开发者有可能会犯的错误。要对需要频繁访问的数据进行缓存，以避免不必要的性能开销。

例如访问Camera.main，其背后的实现是FindObjectWithTag，因此如果没有对这个值进行缓存，那么每次都会调用

FindObjectWithTag，这是一个开销很大的操作。

另一个常见的情况就是GetComponent，对它的返回值也要进行缓存，以避免不必要的开销。

 

**频繁实例化时不使用缓存池**

这一点其实并不是什么新的观点，或者是什么高阶的知识。但是仍然有很多开发者可能忙于业务功能的实现，而没有对此给予相应的重视，造成了性能上的瓶颈。

实例化操作是一个比较耗时的操作，所以切记当需要频繁实例化时，创建一个池，并对其中的对象进行复用。

 

**不了解会造成堆内存分配的API**

C#的GC操作是造成游戏卡顿的一个常见原因。因此，对脚本堆内存分配的优化是很重要的。这里可以使用Unity的Profiler来检测堆内存分配，除了重视堆内存分配很多的帧之外，对数量不起眼但是每帧都会有堆内存分配的方法也要重视。

Unity开发中，常见的会带来堆内存分配的API主要有LINQ、String相关的操作以及返回Array的Unity的API。

例如Physics.RaycastAll，这个API每次调用都会返回一个Array的拷贝，因此更好的选择是使用RaycastNonAlloc来代替。

还例如很多开发者喜欢在检测用户输入时使用Input.touches，对这种property的访问也会带来一次Array的拷贝，所以更好的选择是Input.GetTouch。

 

 

## 0x04.图形以及UI开发中遇到的问题

**项目的Overdraw太多了**

这一点主要是开发移动平台项目的团队需要注意的问题，Overdraw是导致图形性能瓶颈的最常见的原因之一。所以要避免渲染不必要的半透明物体，也可以使用更复杂的mesh来勾出半透明的区域。

 

**制作UI时遇到的种种问题**

UI系统也是导致很多项目出现性能问题的原因，下面Unity官方公众号总结出的《Unity UI性能优化技巧》是十分有参考价值的。

[https://mp.weixin.qq.com/s?__biz=MzU5MjQ1NTEwOA==&mid=2247495094&idx=1&sn=4a948884855c5a6f26f73ff7845c8caf&chksm=fe1dd91dc96a500baf2970fed3b13009e897997472f6596325102dff0cad616defd7a2470d52&token=2019102974&lang=zh_CN#rd](https://link.zhihu.com/?target=https%3A//mp.weixin.qq.com/s%3F__biz%3DMzU5MjQ1NTEwOA%3D%3D%26mid%3D2247495094%26idx%3D1%26sn%3D4a948884855c5a6f26f73ff7845c8caf%26chksm%3Dfe1dd91dc96a500baf2970fed3b13009e897997472f6596325102dff0cad616defd7a2470d52%26token%3D2019102974%26lang%3Dzh_CN%23rd)

 

## 0x05.提醒

**不基于数据来优化**

今年Unite Berlin上，Ian做的一个Presentation的图片可以很好的说明这一点。

![img](https://img2018.cnblogs.com/blog/686199/201810/686199-20181022215945990-177367042.jpg)

除了数据，不要相信任何人，更不要迷信自己的经验。通过之前提到过的各种性能测试工具来获取目标平台上的真正数据，根据数据分析真正的性能瓶颈。并进行调整、对比来确认问题真正的得到了修改。

 

