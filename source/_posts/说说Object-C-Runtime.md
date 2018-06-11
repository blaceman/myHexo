---
title: 说说Object-C Runtime
date: 2016-12-15 12:49:58
tags:
---
##### 今天照着文档来说说runtime,看到一半看不下去了(怪自己英文不好)...先写下来慢慢理解,然后在看下半部分~
首先还是来首歌

<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="https://music.163.com/outchain/player?type=2&id= 234252&auto=1&height=66"></iframe>

还是建议大家看[官方文档](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008048-CH1-SW1)

不想看文档的朋友,推荐一篇比较好的[博客](http://yulingtianxia.com/blog/2014/11/05/objective-c-runtime/)

### 介绍
Object-C 语言延迟了许多从编译链接到运行时的许多决策,尽可能让这门语言动态化.这就意味着这门语言仅需要编译器是不够的,还需要运行系统来执行编译代码.运行时系统作为Object-C语言的操作系统,使它工作.

文档大概说了NSObject class和Object-C程序如何runtime系统进行交互.尤其是检查动态加载新类,和消息转发给其它对象的范式.文档还提供一些有关于运行时你的程序怎么寻找一些有关对象的information

读这篇文章,你该明白Object-C runtime 系统怎样工作和可以如何利用它.一般情况下,就算你不了解它,也不影响你编写的应用程序.


##### 文档目录
这篇文章有以下章节

- Runtime Versions and Platforms(运行时版本与平台)
- Interacting with the Runtime(与运行时交互)
- Messaging(消息)
- Dynamic Method Resolution(动态方法解析)
- Message Forwarding(消息转发)
- Type Encodings(类型编码)
- Declared Properties(声明属性)

##### 也可以参考
[Objective-C Runtime Reference](https://developer.apple.com/reference/objectivec/1657527-objective_c_runtime)描述运行时支持库的数据结构和函数.你的程序可以使用这些和Object-C runtime系统进行交互.例如,你可以增加一些类和方法或者获得加载类的所有类定义列表

[Programming with Objective-C](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/Introduction/Introduction.html#//apple_ref/doc/uid/TP40011210)描述了Object-C语言

[Objective-C Release Notes](https://developer.apple.com/library/content/releasenotes/Cocoa/RN-ObjectiveC/index.html#//apple_ref/doc/uid/TP40004309)描述了Object-C 运行时在最近OS X版本的一些变化


### Runtime Versions and Platforms(运行时版本与平台)

在不同的平台中有不同的Object-C runtime 版本.

#####  Legacy and Modern Versions(过时与现在版本)

在Object-C中有两个runtime版本,分别是"modern"(现在)和"legacy"(过时).modern版本是从Object-C 2.0开始到现在.Objective-C 1运行时参考中描述了运行时的legacy版本的编程接口;现在的Object-C运行时参考中描述了运行时的modern版本的编程接口.

modern运行时最显著的新特征是实例变量是"non-fragile"(非脆)

- 在legacy runtime,如果你在类中改变了实例变量的布局,你必须重新编译继承于它的类
- 在modern runtime,则不需要

##### Platforms(平台)

iPhone 应用程序和在OS X v10.5和以后的64-bit 使用modern 运行时版本.其它则使用legacy运行时版本.


### Interacting with the Runtime(与运行时交互)

Object-C编程与运行时系统交互主要是以下三个方面:1.通过Object源代码,通过NSObject类方法和直接通过runttime函数

##### Objective-C Source Code(Objective-c 源代码)

大多数情况下,runtime系统通过自动的在幕后工作.你只需要写和编译Object-C源代码即可.

当你编译包含Object类和方法代码时,编译器会创建实现这门语言的动态特性的数据结构和函数调用.数据结构会捕获关于类、类别定义和协议声明的信息;包括在Object-C编程语言中定义类和协议中类和协议对象,也包括方法选择器(method selector)实例变量模板,和其它从源代码提取的信息.运行时最主要的是消息发送,像消息的描述.它由源代码表达式发送.


##### NSObject Methods(NSObject 方法)

在Cocoa中大部分对象是NSObject子类,所以大部分对象是继承它定义的方法(NSProxy例外),因此建立了每个实例和每个对象固有的行为.然而，在少数情况下，NSObject类只定义了一个模板，用于如何做一些事情; 它不提供所有必要的代码.

例如,NSObject 类定义了一个description 返回字符串描述类内容实例方法.这主要用于调试 -这个CDB print-object命令打印字符串从这个方法返回.这个方法的NSObject's实现,不知道这个类包含什么内容,所以他返回一个对象名字和地址的字符串.Object子类可以可以实现这个方法来返回更多的细节.例如，Foundation类NSArray返回它包含的对象的描述列表.

一些NSObject方法只是查询运行时系统的信息,这些方法允许对象执行自省.例如一些类方法,要求识别其类;`isKindOfClass:`和`isMemberOfClass:`测试对象在继承层次结构中的位置;`respondsToSelector:`指示对象是否可以接受特定消息;`conformsToProtocol:`指示对象是否宣称去实现在特定的协议中被定义的方法和`methodForSelector:`提供一个方法实现的地址.等能够自省的方法.

##### Runtime Functions(运行时函数)

运行时系统是一个动态共享库，其公共接口由位于目录/ usr / include / objc中的头文件中的一组函数和数据结构组成。 许多这些函数允许你使用纯C来重新使用编译器在编写Objective-C代码时所做的工作。 其他通过NSObject类的方法导出的功能的基础。 这些功能使得可以开发到运行时系统的其他接口，并产生增强开发环境的工具; 当在Objective-C中编程时不需要它们。 但是，编写Objective-C程序时，一些运行时函数有时会很有用。 所有这些函数都记录在[Objective-C Runtime Reference.](https://developer.apple.com/reference/objectivec/1657527-objective_c_runtime)。


### Messaging(消息)

本章节描述的是如何将信息表达式转化为obj_msgSend函数调用,以及怎样按名称查找方法.怎样利用object_msgSend和怎样绕过动态绑定.

##### The objc_msgSend Function(objc_msgSend函数)


在Objective-C中，消息在运行时之前不绑定到方法实现.编译器转换消息表达式

```
[receiver message]
```

转成换objc_msgSend消息函数,这个函数有接收者和被提到的方法名称-就是selector函数,作为主要的两个参数

```
objec_msgSend(receiver,selector)
```

在消息传递中,参数也是通过也是传递给objc_msgSend

```
objc_msgSend(receiver, selector, arg1, arg2, ...)
```

消息传递函数为动态绑定做任何事情

- 首先找到selector的引用过程(方法的实现),相同的方法可以被不同的类实现,其精确查找依赖于receiver的类
- 然后它调用该过程，将其传递给接收对象（指向其数据的指针）以及为该方法指定的任何参数
- 最后，它传递过程的返回值作为它自己的返回值


注意：编译器生成对消息传递函数的调用。 你不应该直接在你写的代码中调用它。


消息转发的关键在于编译器为每个类和对象构建结构。 每个类结构包括这两个基本元素：

- 指向超类的指针。
- 类调度表。 此表具有将方法选择器与其标识的方法的类特定地址相关联的条目。 setOrigin ::方法的选择器与（实现的过程）setOrigin ::的地址相关联，显示方法的选择器与显示地址相关联，依此类推。

创建新对象时，将分配其内存，并初始化其实例变量。 第一个对象的变量是一个指向它的类结构的指针。 这个指针，称为isa，给对象访问它的类，并通过类，到它继承的所有类。


![message](messaging1.gif)

当消息发送給对象,这个消息转发函数会随着对象的isa指针到调度表寻找的方法选择器(selector)类结构.如果没有发现selector,objc_msgSend会随着指针到超类调度表寻找selector,直到NSObject.一旦找到selector,就进入表调用这个方法和传递它给接收对象的数据结构

这就是方法runtime方法实现的方式,或者用术语来说,方法动态绑定.

为了加快消息转发的过程,当它们被调用,运行时系统缓存了selector和方法的地址.而且为每个类分开做缓存.它包含继承的的方法和在类中定义的方法.在查找总调度表之前,首先查找缓存调度表.如果是在缓存找到selector,消息转发仅比函数调用略微耗时.


##### Using Hidden Argument(使用隐藏参数)

在消息转发中,当objc_msgSend找到方法实现的过程,它会调用这个过程和传递它所有的参数,它也传递两个隐藏的参数:

- 接收对象
- 方法选择器

这些参数给有关引用这两个信息表达式的每一个方法实现详细的信息.它们称之为"hidden"(隐藏)是因为在方法定义的源代码没有声明.它们在代码被编译的时候被插入到实现中.

尽管这些参数没有明确的声明,源代码仍然可以引用它们(仅能引用接受对象的实例变量).方法引用接收对象称为self,和用用它自己的selector称为_cmd.在下面就例子中,_cmd引用了strange方法,self引用收到strange消息的对象

```
- strange
{
    id  target = getTheReceiver();
    SEL method = getTheMethod();
 
    if ( target == self || method == _cmd )
        return nil;
    return [target performSelector:method];
}
```

self是更有用的参数,事实上,接收对象的实例变量是可以获得方法的定义.

##### Getting a Method Address(获得方法地址)

规避动态绑定的唯一方法是获取方法的地址，并直接调用它，就像它是一个函数。 这在极少情况下可能是适当的，当特定方法将被连续执行多次，并且希望避免每次执行该方法时消息的开销。

使用NSObject类`methodForSelector`中定义的方法，可以要求指向实现方法的过程的指针，然后使用指针调用过程. `methodForSelector：`返回的指针必须小心地转换为正确的函数类型。 返回和参数类型都应包含在转换中。

下面的示例显示了如何调用实现setFilled：方法的过程：

```
void (*setter)(id, SEL, BOOL);
int i;
 
setter = (void (*)(id, SEL, BOOL))[target
    methodForSelector:@selector(setFilled:)];
for ( i = 0 ; i < 1000 ; i++ )
    setter(targetList[i], @selector(setFilled:), YES);
```


传递给过程的前两个参数是接收对象（self）和方法选择器（_cmd）。 这些参数隐藏在方法语法中，但是当方法作为函数调用时必须显式。

使用methodForSelector：规避动态绑定可以节省消息传递所需的大部分时间。 然而，仅当特定消息被重复多次时，节省将是显着的，如在上面所示的for循环中。

注意，methodForSelector：由Cocoa运行时系统提供; 它不是Objective-C语言本身的一个特性。

### Dynamic Method Resolution(动态派发)
本章介绍如何动态提供方法的实现。

##### Dynamic Method Resolution (动态派发)

某些情况下，您可能需要动态提供方法的实现。 例如，Objective-C声明的属性特性,包括@dynamic指令：

```
@dynamic propertyName;
```

它告诉编译器与属性相关联的方法将被动态提供。

您可以实现方法`resolveInstanceMethod：`和`resolveClassMethod：`分别为实例和类方法动态提供给定选择器的实现。

Objective-C方法只是一个C函数，至少需要两个参数 - self和_cmd。 您可以使用函数class_addMethod将函数添加到类作为方法。 因此，给定以下函数：

```
void dynamicMethodIMP(id self, SEL _cmd) {
    // implementation ....
}
```

你可以使用`resolveInstanceMethod`动态地将它添加到一个类作为一个方法（称为resolveThisMethodDynamically）：像这样：

```
@implementation MyClass
+ (BOOL)resolveInstanceMethod:(SEL)aSEL
{
    if (aSEL == @selector(resolveThisMethodDynamically)) {
          class_addMethod([self class], aSEL, (IMP) dynamicMethodIMP, "v@:");
          return YES;
    }
    return [super resolveInstanceMethod:aSEL];
}
@end
```

转发方法（如消息转发描述）和动态方法解析是，在很大程度上，正交。 类有机会在转发机制开始之前动态地解析方法。如果调用了`responsesToSelector：`或`instancesRespondToSelector：`，动态方法解析器被给予为选择器提供IMP的机会。 如果实现`resolveInstanceMethod：`但希望特定的选择器实际上通过转发机制转发，则对这些选择器返回NO。

##### Dynamic Loading(动态加载)

Objective-C程序可以在运行时加载和链接新的类和类别。新代码被合并到程序中，并且与开始时加载的类和类别完全相同。

动态加载可以用来做很多不同的事情。例如，系统首选项应用程序中的各个模块都是动态加载的。

在Cocoa环境中，通常使用动态加载来允许自定义应用程序。其他人可以编写程序在运行时加载的模块，就像Interface Builder加载自定义调色板和OS X系统首选项应用程序加载自定义首选项模块一样。可加载模块扩展了应用程序可以做什么。他们以你允许，但不能预期或定义自己的方式贡献它。你提供的框架，但其他人提供的代码。

虽然有一个运行时函数在Mach-O文件（objc_loadModules，在objc / objc-load.h中定义）中执行Objective-C模块的动态加载，Cocoa的NSBundle类为动态加载提供了一个非常方便的界面 - 定向和集成相关服务。有关NSBundle类及其使用的信息，请参阅Foundation框架参考中的NSBundle类规范。有关Mach-O文件的信息，请参见OS X ABI Mach-O文件格式参考


### Messag Forwarding(消息转发)

向不处理该消息的对象发送消息是一个错误。 然而，在报错之前，运行时系统给接收对象第二次处理消息的机会。

##### Forwarding(转发)

如果您向不处理该消息的对象发送消息，则在发布错误之前，运行时将对象发送一个`forwardInvocation：`消息，并将`NSInvocation`对象作为其唯一参数，NSInvocation对象封装原始消息和传递的参数。

您可以实现`forwardInvocation`方法以给予消息默认响应，或以某种其他方式避免错误。顾名思义，`forwardInvocation：`通常用于将消息转发到另一个对象。

想要看看转发的范围和意图，想象以下场景：假设，首先，你设计一个对象，可以响应一个消息，称为`negotiate`，并且你想要另一对象的响应。你可以通过传递一个`negotiate`消息到你实现的`negotiate`方法的主体中的其他对象轻松地实现这一点。

再进一步，假设您希望对象对协商消息的响应正是在另一个类中实现的响应。实现这一点的一种方法是让你的类从另一个类继承该方法。但是，可能不可能这样安排。可能有很好的理由为什么你的类和实现协商的类在继承层次结构的不同分支。

即使你的类不能继承negotiate方法，你仍然可以通过实现一个版本的方法，只是将消息传递给另一个类的实例“借”它：


```
- (id)negotiate
{
    if ( [someOtherObject respondsTo:@selector(negotiate)] )
        return [someOtherObject negotiate];
    return self;
}
```

这种做事的方式可能会有点麻烦，特别是如果有一些消息，你想让你的对象传递给另一个对象。你必须实现一个方法来覆盖你想从另一个类借用的每个方法。此外，它是不可能处理的情况下，你不知道，在你写代码的时候，你可能要转发的完整的消息集。该集合可能取决于运行时的事件，并且可能随着新方法和类在将来实现而改变。

`forwardInvocation：`message提供的第二次机会为这个问题提供了一个不太特别的解决方案，动态解决的而不是静态的。它的工作原理是这样的：当对象不能响应消息，因为它没有匹配消息中的选择器的方法，运行时系统通过发送一个`forwardInvocation：`消息通知对象。每个对象从NSObject类继承`forwardInvocation：`方法。然而，NSObject的版本的方法只是调用doesNotRecognizeSelector：。通过重写NSObject的版本并实现自己的，你可以利用`forwardInvocation：`message提供的将消息转发给其他对象的机会。

要转发消息，所有forwardInvocation：方法需要做的是：

- 确定消息应该到哪里，和
- 发送它的原始参数。


消息可以使用`invokeWithTarget：`方法发送：

```
- (void)forwardInvocation:(NSInvocation *)anInvocation
{
    if ([someOtherObject respondsToSelector:
            [anInvocation selector]])
        [anInvocation invokeWithTarget:someOtherObject];
    else
        [super forwardInvocation:anInvocation];
}
```

被转发消息的返回值被返回到原来的Sender(发送发),所有类型的返回值都可以传递给发送方，包括ids，double和双精度浮点数。


`forwardInvocation：`方法可以充当未识别消息的分发中心，将它们分解到不同的接收方。 或者它可以是转移站，将所有消息发送到相同的目的地。 它可以将一个消息转换为另一个消息，或者简单地“吞下”一些消息，所以没有响应和没有错误。 `forwardInvocation：`方法还可以将多个消息合并为单个响应。 `forwardInvocation：`做的事情给实现者。 然而，它提供用于在转发链中链接对象的机会为程序设计打开了可能性。

注意：`forwardInvocation：`方法仅在它们不调用名义接收器中的现有方法时才处理消息。 例如，如果您希望对象将negotiate消息转发给另一个对象，则它不能具有自己的negotiate方法。 如果有，消息将永远不会到达`forwardInvocation：`。
有关转发和调用的更多信息，请参阅Foundation框架参考中的NSInvocation类规范。

##### Forwarding and Multiple Inheritance(转发和多继承)

转发模拟继承，可用于将多重继承的一些影响借给Objective-C程序，通过转发消息来响应消息的对象似乎借用或“继承”另一个类中定义的方法实现。

![forwarding](forwarding.gif)

在此图中，Warrior类的实例将negotiate消息转发到Diplomat类的实例。Warrior就像像Diplomat调用negotiate。Warrior似乎对negotiate的信息作出反应，并且为了所有实际目做出回应（它就像真的是一个Diplomat(外交官)，正在做的工作）。

转发消息的对象因此从继承层次结构的两个分支“继承”方法 - 它自己的分支和响应消息的对象的分支。在上面的例子中，Warrior类继承自Diplomat以及它自己的超类。

转发提供了您通常希望从多重继承的大多数功能。但是，这两者之间有一个重要的区别：多重继承将单个对象中的不同能力组合在一起。它倾向于大，多面的对象。另一方面，转发为不同的对象分配单独的职责。它将问题分解为较小的对象，但以对消息发送方透明的方式关联这些对象


##### Surrogate Objects(代替对象)

转发不仅模仿多重继承，还使得开发表示或“覆盖”更实质对象的轻量级对象成为可能。

在Objective-C编程语言的“远程消息”中讨论的代理是这样的surrogate(代理)。代理负责将消息转发到远程接收器的管理细节，确保在连接上复制和检索参数值，等等。但它不试图做许多其他;它不重复远程对象的功能，而只是给远程对象一个本地地址，它可以在另一个应用程序中接收消息。

其他类型的替代对象也是可能的。假设，例如，你有一个对象操纵大量的数据 - 或许它创建一个复杂的图像或读取磁盘上的文件的内容。将此对象设置为向上可能是耗时的，所以你更喜欢做它懒惰 - 当它真的需要或当系统资源暂时空闲时。同时，您至少需要一个此对象的占位符，以便应用程序中的其他对象正常工作。

在这种情况下，你最初可以创建，不是完整的对象，是一个轻量级的代理。这个对象可以自己做一些事情，例如回答有关数据的问题，但大多数情况下，它只是为较大的对象保存一个地方，当时间到来时，转发消息给它。当代理的`forwardInvocation：`方法首先接收到目的地为另一个对象的消息时，它将确保该对象存在，并且如果它不存在则将创建它。对于较大对象的所有消息通过代理，因此，就程序的其余部分而言，代理和较大对象将是相同的。

##### Forwarding and Inheritance(转发和继承)

虽然转发模仿继承，NSObject类从不混淆两者。 像`responsesToSelector：`和`isKindOfClass`这样的方法：只看继承层次结构，从不看转发链。 例如，如果一个Warrior对象被询问是否响应negotiate消息，

```
if ( [aWarrior respondsToSelector:@selector(negotiate)] )
    ...
```

答案是否定的，即使它可以无误地接收negotiate消息，并在某种意义上通过将它们转发给Diplomat来对它们作出响应。

在许多情况下，NO是正确的答案。 但它可能不是。 如果使用转发来设置代理对象或扩展类的功能，则转发机制应该像继承一样透明。 如果你想让对象的行为好像真的继承了他们转发消息的对象的行为，你需要重新实现`responsesToSelector：`和`isKindOfClass：`方法来包含你的转发算法：

```
- (BOOL)respondsToSelector:(SEL)aSelector
{
    if ( [super respondsToSelector:aSelector] )
        return YES;
    else {
        /* Here, test whether the aSelector message can     *
         * be forwarded to another object and whether that  *
         * object can respond to it. Return YES if it can.  */
    }
    return NO;
}
```

除了`responsesToSelector：`和`isKindOfClass：`，`instancesRespondToSelector：`方法也应该镜像转发算法。 如果使用协议，那么也应该将`conformsToProtocol：`方法添加到列表中。 类似地，如果一个对象转发它接收的任何远程消息，它应该有一个版本的`methodSignatureForSelector：`它可以返回最终响应转发消息的方法的准确描述; 例如，如果对象能够将消息转发到其代理，则可以实现`methodSignatureForSelector：`如下所示

```
- (NSMethodSignature*)methodSignatureForSelector:(SEL)selector
{
    NSMethodSignature* signature = [super methodSignatureForSelector:selector];
    if (!signature) {
       signature = [surrogate methodSignatureForSelector:selector];
    }
    return signature;
}
```

你可能会考虑把转发算法放在私人代码中，并由`forwardInvocation：included`，调用它。


注意：这是一种高级技术，仅适用于没有其他解决方案的情况。 它不是作为继承的替代。 如果你必须使用这种技术，请确保你完全理解类的行为做转发和你转发的类。


本节中提到的方法在Foundation框架参考中的NSObject类规范中进行了描述。 有关invokeWithTarget：的信息，请参阅Foundation框架参考中的[NSInvocation](https://developer.apple.com/reference/foundation/nsinvocation)类规范。


### Declared Properties(声明属性)

当编译器遇到属性声明（请参阅Objective-C编程语言中的声明属性）时，它会生成与封闭类，类别或协议相关联的描述性元数据。 您可以使用支持按类名或协议的名称查找属性的@encode字符串获取属性的类型，以及将属性属性的列表复制为C字符串数组的函数来访问此元数据。 每个类和协议都有一个声明的属性列表。

##### Property Type and Functions(属性类型和函数)

Property结构定义了属性描述符的不透明句柄

```
typedef struct objc_property *Property;
```
您可以使用函数`class_copyPropertyList`和`protocol_copyPropertyList`来检索与类（包括加载的类别）和协议相关联的属性的数组：

```
objc_property_t *class_copyPropertyList(Class cls, unsigned int *outCount)
objc_property_t *protocol_copyPropertyList(Protocol *proto, unsigned int *outCount)
```

例如，给定以下类声明

```
@interface Lender : NSObject {
    float alone;
}
@property float alone;
@end
```
您可以使用以下方式获取属性列表：

```
id LenderClass = objc_getClass("Lender");
unsigned int outCount;
objc_property_t *properties = class_copyPropertyList(LenderClass, &outCount);
```
您可以使用property_getName函数来发现属性的名称：

```
const char *property_getName(objc_property_t property)
```

您可以使用函数class_getProperty和protocol_getProperty来获取对类和协议中给定名称的属性的引用：

```
objc_property_t class_getProperty(Class cls, const char *name)
objc_property_t protocol_getProperty(Protocol *proto, const char *name, BOOL isRequiredProperty, BOOL isInstanceProperty)
```

您可以使用property_getAttributes函数来发现属性的名称和@encode类型字符串。

```
const char *property_getAttributes(objc_property_t property)
```

将这些组合在一起，您可以使用以下代码打印与类关联的所有属性的列表：

```
id LenderClass = objc_getClass("Lender");
unsigned int outCount, i;
objc_property_t *properties = class_copyPropertyList(LenderClass, &outCount);
for (i = 0; i < outCount; i++) {
    objc_property_t property = properties[i];
    fprintf(stdout, "%s %s\n", property_getName(property), property_getAttributes(property));
}
```

##### Property Type String(属性类型字符串)

您可以使用property_getAttributes函数来发现名称，属性的@encode类型字符串和属性的其他属性。

该字符串以一个T开头，后面跟着@encode类型和一个逗号，最后是一个V，后面跟着背景实例变量的名字。 在这些属性之间，属性由以下描述符指定，用逗号分隔


















