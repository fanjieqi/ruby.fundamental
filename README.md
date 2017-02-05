# 编程基础 - Ruby示例

这是一个有着程序示例的知识库，且附带描述当下的编程规则与编程模式。

#### 内容:
  - [线程](#线程)
    - [绿色线程](#绿色线程)
    - [全局解释器锁](#全局解释器锁)
    - [互斥锁](#互斥锁)
    - [纤程](#纤程)
    - [Rails的线程安全](#rails的线程安全)
  - [SOLID原则](#solid原则)
    - [单一职责原则](#单一职责原则)
    - [开放封闭原则](#开放封闭原则)
    - [里氏替换原则](#里氏替换原则)
    - [接口隔离原则](#接口隔离原则)
    - [依赖反转原则](#依赖反转原则)
  - [设计模式](#设计模式)
    - [创建型模式](#创建型模式)
      - [抽象工厂模式](#抽象工厂模式)
      - [生成器模式](#生成器模式)
      - [工厂模式](#工厂模式)
      - [原型模式](#原型模式)
      - [单例模式](#单例模式)
    - [结构型模式](#结构型模式)
      - [适配器模式](#适配器模式)
      - [组合模式](#组合模式)
      - [修饰模式](#修饰模式)
      - [外观模式](#外观模式)
      - [享元模式](#享元模式)
      - [代理模式](#代理模式)
    - [行为型模式](#行为型模式)
      - [责任链模式](#责任链模式)
      - [命令模式](#命令模式)
      - [解释器模式](#解释器模式)
      - [迭代器模式](#迭代器模式)
      - [中介者模式](#中介者模式)
      - [备忘录模式](#备忘录模式)
      - [观察者模式](#观察者模式)
      - [状态模式](#状态模式)
      - [策略模式](#策略模式)
      - [模板方法模式](#模板方法模式)
      - [访问者模式](#访问者模式)
  - [数据结构](#数据结构)
    - [数据结构的基本公理](#数据结构的基本公理)
    - [大O记号](#大o记号)  
    - [实现](#实现)
      - [栈](#栈)
      - [队列](#队列)
      - [双端队列](#双端队列)
      - [单向链表](#单向链表)
      - [双向链表](#双向链表)
      - [有序列表](#有序列表)
      - [哈希表](#哈希表)
      - [二叉树](#二叉树)
      - [二叉查找树](#二叉查找树)
      - [B树](#B树)
      - [二叉堆](#二叉堆)
  - [算法](#算法)
    - [排序](#排序)
      - [冒泡排序](#冒泡排序)
      - [插入排序](#插入排序)
      - [选择排序](#选择排序)
      - [希尔排序](#希尔排序)
      - [堆排序](#堆排序)
      - [归并排序](#归并排序)
      - [快速排序](#快速排序)
    - [搜索](#搜索)
      - [二分查找](#二分查找)
      - [KMP查找](#kmp查找)

## 线程

需注意并行性和并发性：两者主要区别在于操作内存是使用进程还是线程。进一步说，进程会复制一份内存，而线程会分享内存。 这使得进程的产生比线程产生得慢，并且导致了进程一旦运行会消耗更多的资源。总的来说，线程相比进程引发的费用少。这个线程API是一个Ruby的API。我已经暗示了不同的Ruby实现有着不同的潜在线程行为。

#### 绿色线程

Ruby 1.9 用本地线程替换掉了绿色线程。然而，全局解释器锁依然妨碍了并行性。话虽如此，并发已通过更好的调度改进来获得了提升。 新的调度器通过将“上下文切换决策”实质上地弄为单独的本地线程（称为定时器线程），使其更加有效率。

#### 全局解释器锁

MRI有一个围绕着Ruby代码执行的全局解释器锁（GIL）。意味着在多线程上下文中，在任意时刻只有一个线程可以执行Ruby代码。

因此如果在一个8核的机器上你有8个线程在忙碌着，在任何给定的时间只有一个线程核一个核在工作。竞争冒险会毁坏数据，GIL的存在是为了保护Ruby内部不受其影响。虽说GIL有很多警告和优化，但是这才是要旨。

这对于MRI有着很多非常重要的影响，最大的影响在于GIL阻止了Ruby代码在MRI上并行运行。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/threads/gil.rb)

#### 互斥锁

互斥锁给多线程能同步访问代码中的重要部分提供了一种机制。换句话说，互斥锁给多线程的混乱世界带了秩序和保证。

Mutex是mutual exclusion（相互排斥）的缩写。如果你把你的代码用互斥锁括起来，就能保证没有两个线程能在同一时刻进入那段代码。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/threads/mutex.rb)

#### 纤程

在Ruby中，纤程是实现轻量协作并发的基元。基本上，纤程是一套创造可暂停、恢复的代码块的手段。主要的差异在于纤程从不被强占，调度器只能由程序员完成而不是VM。与其他无堆栈轻量级并发模型不同，每个纤程都有一个小小的4KB堆栈。这使得纤程能够在纤程块内被深度嵌套的函数调用暂停。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/threads/fibers.rb)

#### Rails的线程安全

Rails的线程安全问题在于没有一种简单的方式能绝对确定一个应用程序作为一个整体是线程安全的。

* 全局变量是全局的，这意味着在线程种他们是分享的。到目前为止，如果你不能确信是否要使用全局变量，这有着其它原因来说明不要去触碰它。如果你真的需要在一个程序应用中分享一些全局内容，不管怎样，你最好还是用常量来伺候（往下看）。

* 类变量，对于讨论线程的目的，类变量和全局变量没多大区别。类变量一样是在线程中进行分享。问题不在于使用类变量，而是在于改变他们。如果你不准备改变类变量，大多数情况下常量是一个更好的选择。

* 实例变量，在Ruby中，或许你已经了解到你应该总是使用实例变量而不是类变量。好吧，或许你应该如此，但是和类变量一样，他们在线程程序中也是有问题的。


* 记忆化函数本身不是一个线程安全的问题，然而，由于一些原因它很容易成为一个问题：
  
  - 它时常习惯于在类变量或者实例变量中存储数据（看前面）
  - ||=操作符实际上是两个操作，所以在其中可能存在上下文切换，造成线程间的竞争冒险。
  - 很容易忽略记忆化函数也能造成线程安全问题，并且告诉人们只需要避免类变量和实例变量。然后，问题比那个复杂。
  - 在这个线程安全问题上，Evan Phoenix在Rails代码库种挤入了一个非常棘手的竞争冒险错误，这个错误是在记忆化函数调用上一层出现的。所以即便你只使用实例变量，你也会在记忆化函数中以竞争冒险结束。

  确保记忆化在你的例子中函数有意义、起作用。在很多例子中，不管怎样，Rails实际上缓存了结果，所以你不是用记忆化方法存了一整个资源。不要记忆类变量或者实例变量。如果你需要记忆类一级别的东西，用线程中的局部变量 （Thread.current[:baz]）即可。注意，虽然如此，那也是某种意义上的全局变量。所以虽然这是线程安全的，但仍旧不是良好的编程实践。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/threads/rails.rb)

#### 代码及文献资源:

* [http://www.jstorimer.com/blogs/workingwithcode/8085491-nobody-understands-the-gil](http://www.jstorimer.com/blogs/workingwithcode/8085491-nobody-understands-the-gil)

* [https://en.wikipedia.org/wiki/Global_interpreter_lock](https://en.wikipedia.org/wiki/Global_interpreter_lock)

* [http://www.csinaction.com/2014/10/10/multithreading-in-the-mri-ruby-interpreter/](http://www.csinaction.com/2014/10/10/multithreading-in-the-mri-ruby-interpreter/)

## SOLID原则

在计算机编程领域，SOLID原则是一个由Michael Feathers介绍引入、由Robert C. Martin在21世纪早期命名的“五个首要原则”的记忆术首字母缩写词，其代表了五个关于面向对象编程与设计的五个基本原则，包括单一职责、开闭原则、里氏替换、接口隔离以及依赖反转。这些原则的目的在于，当他们一起应用时，将会使得程序员更容易创造一个容易维护和随时间扩展的系统。SOLID的原则是可以应用于在编写软件过程中去除代码异味的指导原则，通过程序员重构软件的源代码，直到它是清晰可读的和可扩展的。它是软件敏捷、自适应开发的全部战略中的一部分。

#### 单一职责原则

一个类应该只有一个职责。

每个类应该只有一个职责，并且这一职责应该完全封装起来。
它的所有服务应该与这一责任紧密相连，这包括了高凝聚。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/solid/single_responsibility.rb)

#### 开放封闭原则

软件实体应该为了扩展而开放，但是为了修改而关闭。
也就是说，这样的实体可以允许其行为被扩展而不修改其源代码。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/solid/open_close.rb)

#### 里氏替换原则
Objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.

It states that, in a computer program, if S is a subtype of T, then objects of type T may be replaced with objects of type S (i.e., objects of type S may substitute objects of type T) without altering any of the desirable properties of that program (correctness, task performed, etc.)

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/solid/liskov_substitution.rb)

#### 接口隔离原则

多个客户端特定接口比单一通用接口更好。

任何客户都不应该被迫依赖于它不使用的方法。
ISP将非常大的接口分割成更小和更具体的接口，使得客户端将只需要知道他们感兴趣的方法。 这种缩小的接口也称为角色接口。
ISP旨在保持系统解耦，从而更容易重构，更改和重新部署。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/solid/interface_segregation.rb)

####依赖反转原则

应该依赖于抽象，而非依赖于具体的结构。

指的是将软件模块去耦的特定形式。 当遵循该原理时，从高级策略设置模块到低级，依赖性模块建立的常规依赖关系被反转（即颠倒），从而使高级模块独立于低级模块实现细节。

####代码及文献资源:

* [https://gist.github.com/khusnetdinov/9d8f50fdcaab197871b31578f2e14d5d](https://gist.github.com/khusnetdinov/9d8f50fdcaab197871b31578f2e14d5d) 

* [https://robots.thoughtbot.com/back-to-basics-solid](https://robots.thoughtbot.com/back-to-basics-solid) 

* [https://subvisual.co/blog/posts/19-solid-principles-in-ruby](https://subvisual.co/blog/posts/19-solid-principles-in-ruby) 

* [http://blog.siyelo.com/solid-principles-in-ruby/](http://blog.siyelo.com/solid-principles-in-ruby/)

## 设计模式

### 创建型模式

在软件工程中，创建型设计模式是处理对象创建机制的设计模式，试图以适应情势的方式创建对象。 对象创建的基本形式可能导致设计问题或增加设计的复杂性。 创建型设计模式通过某种方式控制这个对象创建来解决这个问题。 创建型设计模式由两个主导思想组成。 一个是封装关于系统使用哪个具体类的知识。 另一个是隐藏这些具体类的实例是如何创建和组合的。

[维基百科](https://en.wikipedia.org/wiki/Creational_pattern)

#### 抽象工厂模式

抽象工厂模式提供了一种方式来封装一组具有共同主题的个体工厂，而不指定他们的具体类。 在正常使用中，客户端软件创建抽象工厂的具体实现，然后使用工厂的通用接口创建作为主题一部分的具体对象。 客户端不知道（或关心）具体对象从哪个内部工厂获得，因为它仅使用其产品的通用接口。 此模式将一组对象的实现细节与其一般用法分离，并依赖对象组合，因为对象创建是在工厂接口中公开的方法中实现的。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/creational/abstract_factory.rb) | [维基百科](https://en.wikipedia.org/wiki/Abstract_factory_pattern)

#### 生成器模式

生成器模式是一种对象创建型软件设计模式。 不像抽象工厂模式和工厂方法模式的意图是启用多态性，生成器模式的意图是找到一个解决方案的伸缩构造函数反模式。 当对象构造函数参数组合的增加导致构造函数的指数级列表时，出现了伸缩构造函数反模式。 生成器模式使用另一个对象，即生成器，而不是使用大量构造函数，而是逐步接收每个初始化参数，然后一次返回生成的构造对象。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/creational/builder.rb) | [维基百科](https://en.wikipedia.org/wiki/Builder_pattern)

#### 工厂模式

在基于类的编程中，工厂方法模式是一种创建型模式，它使用工厂方法来处理创建对象的问题，而无需指定将要创建对象的具体类。 这是通过调用工厂方法（在接口中指定并由子类实现，或在基类中实现，并随意地由派生类覆盖而不是通过调用构造函数）创建对象来完成的。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/creational/factory.rb) | [维基百科](https://en.wikipedia.org/wiki/Factory_method_pattern)

#### 原型模式

原型模式是软件开发中的创建设计模式。 当要创建的对象类型由原型实例确定时使用，该原型实例被克隆以生成新对象。 此模式用于：

* 在客户端应用程序中消除对象创建者的子类，类似于抽象工厂模式的作法。
* 当给定应用程序过于昂贵时，避免以标准方式创建新对象（例如，使用“new”关键字）的固有成本。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/creational/prototype.rb) | [维基百科](https://en.wikipedia.org/wiki/Prototype_pattern)

#### 单例模式

确保一个类只有一个实例，并提供一个访问它的全局指针。 当需要一个对象来协调系统上的操作时，这是很有用的。 该概念有时被推广到仅有一个对象能更有效操作，或者对象实例化限制到一定数量的系统。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/creational/singleton.rb) | [维基百科](https://en.wikipedia.org/wiki/Singleton_pattern)

#### 其它模式:

* [惰性初始模式](https://en.wikipedia.org/wiki/Lazy_initialization)

* [多例模式](https://en.wikipedia.org/wiki/Multiton_pattern)

* [对象池](https://en.wikipedia.org/wiki/Object_pool_pattern)

* [RAII](https://en.wikipedia.org/wiki/Resource_Acquisition_Is_Initialization)

* [效用模式](https://en.wikipedia.org/wiki/Utility_pattern)

### 结构型模式

在软件工程中，结构设计模式通过确定一种简单方式来认识实体之间的关系，从而简化设计的设计模式。

[维基百科](https://en.wikipedia.org/wiki/Structural_pattern)

#### 适配器模式

在软件工程中，适配器模式是一种允许现有类的接口用作另一个接口的软件设计模式。 它通常用于使现有类与其他类一起工作而不用不修改其源代码。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/structural/adapter.rb) | [维基百科](https://en.wikipedia.org/wiki/Adapter_pattern)

#### 组合模式

组合模式是一种用于表示具有分层树结构的对象的结构模式。 它允许单个叶节点和由许多节点组成的分支的统一处理。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/structural/composite.rb) | [维基百科](https://en.wikipedia.org/wiki/Composite_pattern)

#### 修饰模式

在面向对象编程中，修饰模式（也称为Wrapper，适配器模式的替代命名）是一种设计模式，允许将静态行为或动态行为添加到单个对象，而不影响同一个类的其他对象。修饰模式通常对遵守单一责任原则很有用，因为它允许功能被划分到具有唯一关注领域的类之中。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/structural/decorator.rb) | [维基百科](https://en.wikipedia.org/wiki/Decorator_pattern)

#### 外观模式

当系统具有大量相互依赖的类或其源代码不可用以至于非常复杂或难以理解时，通常使用外观设计模式。 这种模式隐藏了较大系统的复杂性，并为客户端提供了一个更简单的接口。它通常有一个包含客户端所需的一组成员的单个包装类。 这些成员代表外观客户端访问系统并隐藏实现细节。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/structural/facade.rb) | [维基百科](https://en.wikipedia.org/wiki/Facade_pattern)

#### 享元模式

在计算机编程中，享元模式是一种软件设计模式。享元模式是一个对象，通过与其他类似对象共享尽可能多的数据来最小化内存使用；它是一种大量使用对象的方法，用于当简单且重复的指代将使用巨量内存时。 通常，对象状态的某些部分是可以共享的，并且通常的做法是将它们保存在外部数据结构中，并在使用时将它们临时传递给享元对象。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/structural/flyweight.rb) | [维基百科](https://en.wikipedia.org/wiki/Flyweight_pattern)

#### 代理模式

代理，最普遍的形式是一个类，作为其他东西的接口。 代理可以做任何东西的接口：网络连接，存储器中的大对象，文件或一些昂贵或不可能复制的其他资源。 简而言之，代理是一个包装器或代理对象，客户端调用它来访问幕后的实际服务对象。 代理的使用可以简单地转发到实际对象，或者可以提供附加逻辑。 在代理中，可以提供额外的功能，例如当实际对象进行资源密集型操作时提供高速缓存，或者在调用实际对象的操作之前检查前提条件。 对于客户端，代理对象的用法类似于使用实际对象，因为两者都实现相同的接口。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/structural/proxy.rb) | [维基百科](https://en.wikipedia.org/wiki/Proxy_pattern)

#### 其它模式:

* [注释回调](http://c2.com/cgi/wiki?AnnotatedCallback)

* [桥接模式](https://en.wikipedia.org/wiki/Bridge_pattern)

* [数据总线模式](http://c2.com/cgi/wiki?DataBusPattern)

* [角色对象模式](http://c2.com/cgi/wiki?RoleObjectPattern)

### 行为型模式 

在软件工程中，行为型模式是识别对象之间的公共通信模式并实现这些模式的设计模式。 通过这样做，这些模式增加了执行该通信的灵活性。

[维基百科](https://en.wikipedia.org/wiki/Behavioral_pattern)

#### 责任链模式

在面向对象设计中，责任链模式是由命令对象源和一系列处理对象组成的设计模式。 每个处理对象包含的逻辑，定义了其可以处理的命令对象的类型；其余的被传递到链中的下一个处理对象。 还存在一个将新的处理对象添加到该链的末端的机制。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/chain_of_responsibility.rb) | [维基百科](https://en.wikipedia.org/wiki/Chain-of-responsibility_pattern)

#### 命令模式

将请求封装为对象，从而允许具有不同请求的客户端的参数化，以及请求的排队或日志记录。 它还允许支持可撤销操作。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/command.rb) | [维基百科](https://en.wikipedia.org/wiki/Command_pattern)

#### 解释器模式

在计算机编程中，解释器模式是说明如何评估语言中的句子的设计模式。 基本思想是在专门的计算机语言中为每个符号（终端或非终端）设置一个类。 语言中的句子的语法树是复合模式的实例，并且用于为客户端评估（解释）句子。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/interpreter.rb) | [维基百科](https://en.wikipedia.org/wiki/Interpreter_pattern)

#### 迭代器模式

迭代器设计模式提供对容器中的元素的顺序访问，而不暴露容器如何具体地表示元素。 迭代器可以被认为是一个可移动的指针，允许访问封装在容器中的元素。

*外部迭代器：迭代逻辑包含在一个单独的类中。 迭代类可以被概括为处理多个对象类型，只要它们允许索引。 虽然它需要附加类来做实际的迭代，但它们允许更大的灵活性，因为你可以控制迭代及迭代顺序。

*内部迭代器：所有迭代逻辑发生在聚合对象内部。 使用代码块将你的逻辑传递给聚合，然后为每个元素调用代码块。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/iterator.rb) | [维基百科](https://en.wikipedia.org/wiki/Iterator_pattern)

#### 中介者模式

通常一个程序是由大量的类组成的，所以逻辑和计算分布在这些类中。 然而，随着在程序中开发更多的类，特别是在维护或重构期间，这些类之间的通信问题可能变得更复杂。这使得程序更难以阅读和维护。 此外，可能变得难以改变程序，因为任何改变可能影响其他几个类中的代码。 使用中介器模式，对象之间的通信用中介器对象封装。 对象彼此间不再直接通信，而是通过中介器进行通信。 这减少了通信对象之间的依赖性，从而降低了耦合。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/mediator.rb) | [维基百科](https://en.wikipedia.org/wiki/Mediator_pattern)

#### 备忘录模式

备忘录模式由三个对象实现：发起者，守护者备忘录。 发起者是具有内部状态的一些对象。 守护者将操作发起者，但希望能够撤消该更改。 守护者首先向发起人请求备忘录。 然后它将要做任何操作（或操作序列）。 要回滚到操作之前的状态，它将备忘录对象返回给发起者。 备忘录对象本身是一个不透明的对象（守护者不能，或不应该改变）。当使用这种模式时，请注意 - 如果发起者可能改变其他对象或资源，备忘录模式对单个对象进行操作。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/momento.rb) | [维基百科](https://en.wikipedia.org/wiki/Memento_pattern)

#### 观察者模式

观察者模式是一种软件设计模式，其中称为主题的对象维护其依赖项的列表，称为观察者，并且通常通过调用它们的方法来自动通知任何状态改变。 它主要用于实现分布式事件处理系统。 观察者模式也是熟悉的模型-视图-控制器（MVC）架构模式中的关键部分。观察者模式在许多编程库和系统中实现，包括几乎所有的GUI工具包。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/observer.rb) | [维基百科](https://en.wikipedia.org/wiki/Observer_pattern)

#### 状态模式

状态模式是以面向对象的方式实现状态机的行为型模式。 利用状态模式，通过将每个单独状态实现为状态模式接口的派生类，并通过调用由模式的超类定义的方法来实现状态转换，从而实现了状态机。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/state.rb) | [维基百科](https://en.wikipedia.org/wiki/State_pattern)

#### 策略模式

在计算机编程中，策略模式（也称为政策模式）是使得能够在运行时选择算法的软件设计模式。战略模式

*定义一系列算法，
*封装每个算法
*使得算法在该系列中可互换。

策略模式允许算法变化独立于使用它的客户端。Gamma等人所著的一本非常有影响力的书“设计模式”普及了使用模式来描述软件设计的概念，策略模式是其中包含的模式之一。

例如，对输入数据执行验证的类可以使用策略模式来基于数据的类型、数据的源、用户选择或其他辨别因素来选择验证算法。直到运行时间，这些因素在每种情况下才是可知的，并且可能需要进行彻底不同的验证。与验证对象分开封装的验证策略，可以由系统的不同区域（或甚至不同系统）中的其他验证对象使用，而无需代码重复。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/strategy.rb) | [维基百科](https://en.wikipedia.org/wiki/Strategy_pattern)

#### 模板方法模式

在面向对象编程中，首先创建一个提供算法设计的基本步骤的类。 这些步骤使用抽象方法实现。 之后，子类改变了抽象方法来实现真正的操作。 因此通用算法保存在一个位置，但具体步骤可以由子类修改。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/template_method.rb) | [维基百科](https://en.wikipedia.org/wiki/Template_method_pattern)

#### 访问者模式

在面向对象的编程和软件工程中，访问者设计模式是将算法与其操作的对象结构分离的一种方式。这种分离的实际结果是在不修改这些结构的情况下向现有对象结构添加新操作的能力。它是遵循开放封闭原则的一种方式。

实质上，访问者模式允许向一系列类添加新的虚拟函数而不修改类本身，反而是创建一个访问者类来实现所有虚拟函数适当的特殊化。访问者将实例引用作为输入，并通过双调度实现目标。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/patterns/behavioral/visitor.rb) | [维基百科](https://en.wikipedia.org/wiki/Visitor_pattern)

#### 其它模式:

* [访问者模式](http://c2.com/cgi/wiki?HierarchicalVisitorPattern)

#### 代码及文献资源:

* [https://ru.wikipedia.org/wiki/Design_Patterns](https://ru.wikipedia.org/wiki/Design_Patterns)

* [https://en.wikipedia.org/wiki/Creational_pattern](https://en.wikipedia.org/wiki/Creational_pattern)

* [https://en.wikipedia.org/wiki/Structural_pattern](https://en.wikipedia.org/wiki/Structural_pattern)

* [https://en.wikipedia.org/wiki/Behavioral_pattern](https://en.wikipedia.org/wiki/Behavioral_pattern)

* [http://c2.com/cgi/wiki?CategoryPattern](http://c2.com/cgi/wiki?CategoryPattern)

* [https://gist.github.com/martindemello/7231bf0f407ca428b509](https://gist.github.com/martindemello/7231bf0f407ca428b509)

## 数据结构 

在计算机科学中，大O符号用于通过算法如何响应输入大小的变化来对其进行分类，例如算法的处理时间随着问题规模变得非常大而变化。 在分析数论中，它用于估计“错误提交”，用一个大的有限自变量取代一个算术函数的渐进大小。 一个着名的例子是估计质数定理中的余项的问题。

### 数据结构的基本公理

公共语言的运行时间性能由我们现在假设的一组公理给出。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/axioms.rb)

#### 代码及文献资源:

* [Axioms](http://www.brpreiss.com/books/opus8/html/page37.html)

### 大O记号

* [Conventions for Writing Big Oh Expressions](http://www.brpreiss.com/books/opus8/html/page66.html)

* [Rules For Big Oh Analysis of Running Time](http://www.brpreiss.com/books/opus8/html/page72.html)

* [Reality Check](http://www.brpreiss.com/books/opus8/html/page76.html)

* [Example: Ruby method with Big O explanation](http://www.brpreiss.com/books/opus8/html/page71.html)

* [Example: Example-Prefix Sums](http://www.brpreiss.com/books/opus8/html/page73.html)

* [Example: Bucket Sort](http://www.brpreiss.com/books/opus8/html/page75.html)

### 实现 

> NOTE: 所有数据结构都是作为学习目的的示例。 对于在项目中使用，请参考其他资源。 大多数例子取自Bruno R. Preiss的[网站](http://www.brpreiss.com/)。

#### 栈

堆栈是队列的兄弟。 它模仿了现实生活中的堆栈（例如纸）。 它是“先进后出”FILO（first-in-last-out），因此当从堆栈中检索项目时，它们按照添加顺序的反向返回。 并且，Ruby数组提供了一个完美的容器。 与队列一样，它也可以用链表实现。

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[栈](http://en.wikipedia.org/wiki/Stack_(abstract_data_type)) | `Θ(n)` | `Θ(n)` | `Θ(1)` | `Θ(1)` | `O(n)` | `O(n)` | `O(1)` | `O(1)` |

[栈(数组) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/stack_as_array.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page131.html)

[栈(链表) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/stack_as_linked_list.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page137.html)

#### 队列

队列是一个简单的基于容器的结构，模拟了现实生活中的队列（例如在银行排队等待）。 它是先进先出FIFO（first-in-first-out），意味着当您从队列中检索项目时，它们按照它们的添加顺序返回。 Ruby数组提供了使Queue实现简单易用的方法，但是让它们适当地命名并包含在一个方便的类中是值得去看看它们是如何实现的，并且因为其他结构将继承这个方法。队列可以使用链表来完成替代实现。

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[队列](http://en.wikipedia.org/wiki/Queue_(abstract_data_type)) | `Θ(n)` | `Θ(n)` | `Θ(1)` | `Θ(1)` | `O(n)` | `O(n)` | `O(1)` | `O(1)` |

[队列(数组) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/queue_as_array.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page147.html)

[队列(链表) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/queue_as_linked_list.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page151.html)

#### 双端队列

双端队列是一个允许在两端添加和删除项目的队列。

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[双端队列](https://en.wikipedia.org/wiki/Double-ended_queue) | `Θ(n)` | `Θ(n)` | `Θ(1)` | `Θ(1)` | `O(n)` | `O(n)` | `O(1)` | `O(1)` |

[双端队列(数组) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/deque_as_array.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page158.html)

[双端队列(链表) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/deque_as_linked_list.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page161.html)

#### 单向链表

单向链表的节点包含数据字段以及“下一个”域，其指向链表中的下一个节点。可以对单向链表执行的操作包括插入，删除和遍历。

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[单向链表](http://en.wikipedia.org/wiki/Singly_linked_list#Singly_linked_lists) | `Θ(n)` | `Θ(n)` | `Θ(1)` | `Θ(1)` | `O(n)` | `O(n)` | `O(1)` | `O(1)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/singly_linked_list.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page96.html#SECTION004300000000000000000) | [维基百科](https://en.wikipedia.org/wiki/Linked_list#Singly_linked_list) 

#### 双向链表

在双向链表中，每个链表节点包含两个引用 - 一个指向其后继，一个指向其前任。

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[双向链表](http://en.wikipedia.org/wiki/Doubly_linked_list) | `Θ(n)` | `Θ(n)` | `Θ(1)` | `Θ(1)` | `O(n)` | `O(n)` | `O(1)` | `O(1)` | 

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/double_linked_list.rb)

#### 有序列表
 
有序列表是一种项目顺序非常重要的列表。但是，有序列表中的项目不一定排序。因此，可以改变项目的顺序，并且仍然具有有效的有序列表。

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
有序列表(数组) |`Θ(1)` | `Θ(n)` | `Θ(1)` | `Θ(1)` | `O(1)` | `O(n)` | `O(1)` | `O(1)` |
有序列表(链表) |`Θ(n)` | `Θ(n)` | `Θ(1)` | `Θ(1)` | `O(n)` | `O(n)` | `O(1)` | `O(1)` |

[有序列表(数组) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/ordered_list_as_array.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page169.html)

[有序列表(链表) - 示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/ordered_as_linked_list.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page178.html)

#### 哈希表

哈希表是一种可搜索的容器。因此，它提供了用于插入对象、查找对象、删除对象的方法。

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[哈希表](http://en.wikipedia.org/wiki/Hash_table) | `Θ(n)` | `Θ(1)` | `Θ(1)` | `Θ(1)` | `Θ(n)` | `O(n)` | `O(n)` | `O(n)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/hash_table.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page222.html)

#### 二叉树

二叉树是每个节点最多可以有两个子节点的树。孩子们被指定为左节点和右节点。

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[二叉树](https://en.wikipedia.org/wiki/Binary_tree) | `Θ(log(n))` | `Θ(log(n))` | `Θ(log(n))` | `Θ(log(n))` | `O(n)` | `O(n)` | `O(n)` | `O(n)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/binary_tree.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page286.html)

#### 二叉查找树

在计算机科学中，二叉查找树（BST）（有时称为有序二叉树或排序二叉树）是特定类型的容器：在内存中存储“项目”（例如数字，名称等）的数据结构。 它们允许快速查找，添加和删除项目，并且可以用于实现项目的动态集合或者允许通过查找其键来查找项目（例如，通过名称找到人的电话号码）。

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[二叉查找树](http://en.wikipedia.org/wiki/Binary_search_tree) | `Θ(log(n))` | `Θ(log(n))` | `Θ(log(n))` | `Θ(log(n))` | `O(n)` | `O(n)` | `O(n)` | `O(n)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/binary_search_tree.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page300.html)

#### B树

在计算机科学中，B树是自平衡树数据结构，其保持数据有序并允许在对数时间内的搜索，顺序存取，插入和删除。 B树是二叉查找树的概括，因为一个节点可以有多于两个子节点（与自平衡二叉搜索树不同，B树是针对读取和写入大块数据的系统进行优化的。 B树是提供给外存的数据结构的一个很好的例子，它常用于数据库和文件系统。

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[B树](http://en.wikipedia.org/wiki/B_tree) | `Θ(log(n))` | `Θ(log(n))` | `Θ(log(n))` | `O(log(n))` | `O(log(n))` | `O(log(n))` | `O(log(n))` | `O(log(n))` | 

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/b_tree.rb) | [实现步骤](http://www.brpreiss.com/books/opus8/html/page339.html)

#### 二叉堆

二叉堆是使用数组实现的堆有序完整二叉树。 在堆中，最小的键在根处找到，并且由于根总是在数组的第一个位置找到，因此找到最小的键是二进制堆中的一个微不足道的操作。

| 数据结构 | 访问平均 | 查找平均 | 插入平均 | 删除平均 | 访问最坏 | 查找最坏 | 插入最坏 | 删除最坏 |
|-----------|---------------:|---------------:|------------------:|-----------------:|-------------:|-------------:|----------------:|---------------:|
[二叉堆](https://en.wikipedia.org/wiki/Binary_heap) | `Θ(n)` | `Θ(n)` | `Θ(1)` | `O(log(n))` | `O(n)` | `O(n))` | `O(log(n))` | `O(log(n))` | 

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/structures/binary_heap.rb) | [实现步骤] (http://www.brpreiss.com/books/opus8/html/page355.html)

#### 代码及文献资源:

* [https://github.com/blahah/datastructures](https://github.com/blahah/datastructures)

* [http://www.brpreiss.com](http://www.brpreiss.com)

* [https://en.wikipedia.org/wiki/List_of_data_structures](https://en.wikipedia.org/wiki/List_of_data_structures)

* [https://www.cs.cmu.edu/~adamchik/15-121/lectures/Trees/trees.html](https://www.cs.cmu.edu/~adamchik/15-121/lectures/Trees/trees.html)

* [http://bigocheatsheet.com/](http://bigocheatsheet.com/)

* [https://gist.github.com/TSiege/cbb0507082bb18ff7e4b](https://gist.github.com/TSiege/cbb0507082bb18ff7e4b)

## 算法

### 排序

排序算法是以一定顺序排列列表元素的算法。 最常用的是数字顺序和字典顺序。 高效排序对于优化其他算法（例如搜索和合并算法）的使用是重要的，这些算法要求输入数据在排序列表中; 它也常用于规范化数据和产生人类可读的输出。

#### 冒泡排序

冒泡排序和插入排序有许多相同的属性，但有稍高的开销。 在几乎有序的数据的情况下，冒泡排序需要O（n）时间，但是需要至少遍历2遍数据（而插入排序需要差不多1遍）。

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Ω(n)` | `Θ(n^2)` | `O(n^2)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/bubble.rb) | [维基百科](https://en.wikipedia.org/wiki/Bubble_sort)

#### 插入排序

虽然它是具有O（n2）最坏情况时间的基本排序算法之一，但是当数据几乎被排序（因为它是自适应的）时或者当问题规模小时（因为它低开销），插入排序是可以选择的算法。
由于这些原因，并且因为它也是稳定的，所以插入排序通常用于更高开销的分治算法的递归基础（当问题规模较小时），例如合并排序或快速排序。

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Ω(n)` | `Θ(n^2)` | `O(n^2)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/insertion.rb) | [维基百科](https://en.wikipedia.org/wiki/Insertion_sort)

#### 选择排序

从这里提供的比较，可能会得出结论，不应该使用选择排序。 它不适合任何排列方式的数据（注意上面的四个图表），它的运行时总是二次的。
但是，选择排序具有最小化交换数量的属性。 在交换项目的成本高的应用中，可能选择使用选择排序。

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Ω(n^2)` | `Θ(n^2)` | `O(n^2)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/selection.rb) | [维基百科](https://en.wikipedia.org/wiki/Selection_sort)

#### 希尔排序

希尔排序的最糟糕的时间复杂度取决于增量序列。 对于这里使用的增量1 4 13 40 121 ...，时间复杂度是O（n3 / 2）。 对于其他增量，已知时间复杂度为O（n4 / 3）和甚至O（n·lg2（n））。 时间复杂度的上限和最佳增量序列都是未知的。
因为希尔排序基于插入排序，所以shell排序继承了插入排序的自适应属性。自适应属性不那么显着，因为希尔排序需要遍历一遍每次递增的数据的，但它是重要的。 对于上述增量序列，有log3（n）个增量，因此几乎排序的数据的时间复杂度为O（n·log3（n））。
由于其低开销，相对简单的实现，自适应属性和子二次时间复杂度，对于某些应用程序，当需要排序的数据不是很大的时候，希尔排序可能是O（n·lg（n））排序算法的可行替代 。

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Ω(n log(n))` | `Θ(n(log(n))^2)` | `O(n(log(n))^2)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/shell.rb) | [维基百科](https://en.wikipedia.org/wiki/Shellsort)

#### 堆排序

堆排序易于实现，执行O（n·lg（n））就地排序，但不稳定。
第一个循环，Θ（n）堆化阶段，将数组放入堆。 第二个循环，O（n·lg（n））排序阶段，重复提取最大值和修复堆顺序。
为了清楚起见，递归地写下沉功能。 因此，如图所示，代码需要用于递归调用栈的Θ（lg（n））空间。 然而，下沉中的尾递归很容易转换为迭代，这产生了O（1）空间束缚。
两个阶段略微自适应，虽然不是以任何特别有用的方式。 在几乎有序的情况下，堆化阶段销毁原始顺序。 在相反的情况下，堆化阶段是尽可能快的，因为数组以堆顺序开始，但在排序阶段是一贯的。 在少数独特的关键字案例中，有一些加速，但不如希尔排序或3路快速排序。

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Ω(n log(n))` | `Θ(n log(n))` | `O(n log(n))` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/heap.rb) | [维基百科](https://en.wikipedia.org/wiki/Heapsort)

#### 归并排序

归并排序是十分可预测的。 它使得每个元素进行0.5lg（n）到lg（n）次比较，以及每个元素交换lg（n）到1.5lg（n）次。 对已经排序的数据实现最小比较和交换; 平均来说，对于随机数据实现最多比较和交换。 如果不担心使用Θ（n）额外空间，那么归并排序是一个很好的选择：它很容易实现，它是唯一稳定的O（n·lg（n））排序算法。 注意，当排序链表时，归并排序只需要Θ（lg（n））额外空间（用于递归）。
归并排序是针对各种情况都可选择的算法：需要稳定性时，排序链表时，以及当随机访问比顺序访问贵得多时（例如，磁带上的外部排序）。
对于算法的最后一步，存在线性时间就地归并算法，但是它们既昂贵又复杂。 当Θ（n）额外空间不可用时，这个复杂度还是合理的。

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Ω(n log(n))` | `Θ(n log(n))` | `O(n log(n))` | 

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/merge.rb) | [维基百科](https://en.wikipedia.org/wiki/Merge_sort)

#### 快速排序

当细心实现时，快速排序是健壮的并且具有低开销。当不需要稳定的排序时，快速排序是一个很好的通用排序 - 虽然应该总是使用3路分区版本。
上面显示的双路分区代码是为了清楚而不是最佳性能而编写的;它局部表现差，并且，当关键字很少花费O（n2）时间。Robert Sedgewick和Jon Bentley的论文Quicksort is Optimal给出了更有效和鲁棒的双向分割方法。当存在许多值等于中枢值时，鲁棒分区产生平衡递归，为所有输入产生O（n·lg（n））时间和O（lg（n））空间的概率保证。
对于递归执行的两个子排序，快速排序在递归不平衡的最坏情况下需要O（n）额外的空间用于递归堆栈。这是极不可能发生的，但它可以通过递归排序较小的子数组避免;第二个子数组排序是一个尾递归调用，可以通过迭代来完成。通过这种优化，算法在最坏的情况下使用O（lg（n））额外空间。

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Ω(n log(n))` | `Θ(n log(n))` | `O(n^2)` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/quick.rb) | [维基百科](https://en.wikipedia.org/wiki/Quicksort)

#### 其它排序算法

* [http://rosettacode.org/wiki/Category:Sorting_Algorithms](http://rosettacode.org/wiki/Category:Sorting_Algorithms)

* [https://www.toptal.com/developers/sorting-algorithms](https://www.toptal.com/developers/sorting-algorithms)

#### Additional reading

* [https://www.igvita.com/2009/03/26/ruby-algorithms-sorting-trie-heaps/](https://www.igvita.com/2009/03/26/ruby-algorithms-sorting-trie-heaps/)

### 搜索

#### 二分查找

在计算机科学中，二分搜索，也称为半间隔搜索或对数搜索，是一种搜索算法，其在排序的数组中找到目标值的位置。 它将目标值与数组的中间元素进行比较; 如果它们不相等，则目标值不可能呆在的一半被排序，并且搜索在剩余的一半继续，直到它成功。

| 最好 | 平均 | 最坏 |
|-----:|--------:|------:|
| `Θ(1)` | `Θ(n log(n))` | `O(log(n))` |

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/search/binary.rb) | [维基百科](https://en.wikipedia.org/wiki/Knuth%E2%80%93Morris%E2%80%93Pratt_algorithm)

#### KMP查找

在计算机科学中，Knuth-Morris-Pratt字符串搜索算法（或KMP算法）通过采用以下观察来搜索主“文本串”S内的“字”W的出现：当出现不匹配时，该字本身体现足够信息以确定下一个匹配可以开始的位置，从而绕过对先前匹配的字符的重新检查。

[示例](https://github.com/fanjieqi/ruby.fundamental/blob/master/algorithms/sort/knuth-morris-pratt.rb) | [维基百科](https://en.wikipedia.org/wiki/Binary_search_algorithm)

#### 其它算法

* [Dijkstra算法](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm)

* [Kruskal算法](https://en.wikipedia.org/wiki/Kruskal%27s_algorithm)

* [最长上升子序列](https://en.wikipedia.org/wiki/Longest_increasing_subsequence)

* [电话号码对应英语单词](http://www.mobilefish.com/services/phonenumber_words/phonenumber_words.php)

#### 代码及文献资源:

* [https://github.com/sagivo/algorithms](https://github.com/sagivo/algorithms)

* [https://github.com/kanwei/algorithms](https://github.com/kanwei/algorithms)



