---
layout: hexo
title: java题库
date: 2021-12-01 16:50:24
tags: java
cover: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fi2.hdslb.com%2Fbfs%2Farchive%2F540e03f9ba1527258a27e6a0f31aacc59c6a5ec6.jpg&refer=http%3A%2F%2Fi2.hdslb.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1654421952&t=e363fdea62ccee37819c1f9151cd51ea
category: 笔记
top_img: https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fi2.hdslb.com%2Fbfs%2Farchive%2F540e03f9ba1527258a27e6a0f31aacc59c6a5ec6.jpg&refer=http%3A%2F%2Fi2.hdslb.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=auto?sec=1654421952&t=e363fdea62ccee37819c1f9151cd51ea
---
#### 1、HashMap的put方法
HashMap是数组、链表、红黑树实现的。红黑树又称自平衡二叉查找树。
> HashMap的put的具体流程如下：  
> （1）HashMap通过put方法传进来的key通过哈希算法得到HashCode，HashCode通过与**与操作**去和数组长度-1运算先得出数组下标。
> （2）如果数组下标为止元素为空则将key和value封装为Node（Entry）对象放入该位置。 
> （3）如果数组下标元素不为空，先判断当前位置Node的类型，是红黑树还是链表Node。  
>   如果是是红黑树则将key和value封装成红黑树节点，添加到红黑树中去。 在添加到红黑树的过程中判断红黑树是否存在当前的key，如果存在则更新value。
>   如果Node对象是链表节点的话则将key和value封装成一个链表Node通过尾插法添加到链表最后的位置。（JDK1.7是头插法）尾插法会遍历链表，如果存在当前的key则更新value。
> 插入链表后当前链表元素大于等于8个，会将链表转成红黑树。  
> jdk1.7是插入前判断是否需要扩容，jdk1.8是红黑树Node或者链表Node插入后判断是否需要扩容，需要就扩容，不需要就结束当前的put方法。
#### 2、说一下ThreadLocal
> ThreadLocal是java中线程本地存储机制，可以利用该机制将某个数据、某个对象，某个值缓存在某个线程的内部，
该线程可以在任意时刻、任意方法中获取缓存的数据。
> ThreadLocal是通过ThreadLocalMap实现的，每个Thread对象中都存在一个ThreadLocalMap，Map的Key是Thread对象，
> Map的Value为要缓存的值。  
> 使用场景：  
> 两个线程如果给同一个对象赋不同的值，获取对象值可能为线程1赋得值，也可能为线程2赋得值。会有线程不安全的问题。  
> 使用ThreadLocal会将对象值缓存到该线程中，每个线程拿到的值固定为该线程赋予的。最简单的使用场景，往里存用户信息，用户对象User是全局的，每个线程拿到的用户信息都不一样，就往里存，然后可以根据线程拿到不同用户的信息
#### 3、说一下JVM中哪些是共享区域
> 首先jvm分为三个主要部分：运行时数据区域（内存区域）、类装载系统、字节码执行引擎。
> 运行时数据区域分为：方法区、栈、堆、程序计数器、本地方法栈；其中栈又叫做线程栈，每一个线程开始运行，JVM虚拟机就会给此线程分配一个线程栈，程序计数器同理栈。本地方法栈同理栈，只是运行方法为java虚拟机缺省的方法。由此可知栈、程序计数器、本地方法栈为非共享区域，
> 方法区（元空间）存储的是类信息和常量、静态变量、类信息，这些数据每个线程都可以去调用是共享的。堆存放的是对象同理元空间，也是共享的每个线程都可以去访问。
#### 4、说一下gc root
> java是有垃圾回收机制的，jvm会主动进行垃圾回收，区分哪些为"垃圾"，那些为"非垃圾"就是通过对象是否被引用判断，gc root作为是否被引用进行查找的根，gc root特征是只会引用，而不会被引用。举例有：栈中的本地变量、方法区中的静态变量、本地方法栈中的变量，正在运行的线程等可以叫gc root。
#### 5、说一下Minor GC、full gc 和 Major gc的区别
> Minor gc 从年轻代空间（包括 Eden 和 Survivor 区域）回收内存被称为 Minor gc。  
> Major gc 是清理老年代。会stop the world停止应用线程或者叫用户线程。
> full gc是对青年代、老年代、元空间的全局垃圾回收，触发full gc的时候会stop the world停止应用线程或者叫用户线程。
> Minor gc使用复制算法不会STW，Major gc和full gc使用标记整理算法会STW。
#### 6、正在运行的项目如何排查jvm问题
> 可以使用jmap命令查看jvm各个区域的使用情况。     
> 可以使用jstack命令查看jvm各个线程的情况。 
> 可以使用jstat命令查看垃圾回收情况。 
> 一般使用的jvm监控程序也是调用这些命令。  
> 一般jvm问题就是减少full gc。 
> 系统发生OOM的时候会生成dump文件，通过工具分析dump文件查看异常的对象、线程定位到具体代码。  
> jvm 调优需要分析，定位问题，不同场景使用不同的解决方法，目的是解决cpu占用高、频繁full gc、内存溢出等。
#### 7、如何查看线程死锁
> 通过工具或者直接用jstack命令
> mysql线程死锁可用通过sql或查看表
#### 8、线程之间是如何进行通讯的
> 一个进程内两个线程进行通讯和两个进程（两台机器）的线程如和进行通讯。  
> 第一种可以通过共享内存的方式进行通讯，第二种通过网络的方式进行通讯。  
#### 9、介绍一下Spring
> 简单说Spring是一个生态，可以构建java应用所需要的一切基础设施，通常提到Spring是指SpringFramWork.  
> Spring 是一个轻量级开源的容器框架。  
> Spring 是为了解决企业级应用开发业务逻辑层和其他层对象和对象之间耦合问题。  
> Spring 是一个IOC和AOP的容器框架。AOP：面向切面编程；IOC：控制反转；容器：包含并管理应用对象的生命周期。  
> （1）启动Spring的时候就是创建一个Spring容器，就是创建一个SpringApplicationContext的对象。  
> （2）首先Spring会扫面，得到所有的BeanDefinition对象，存在一个map里。
> （3）筛选非懒加载的单例BeanDefinition对象进行创建bean，对于多例的bean不需要在启动的过程中去创建。对于多例的bean会在每次获取的时候通过BeanDefinition去创建。
> （4）利用BeanDefinition创建bean就是bean的创建生命周期，创建bean期间包括合并BeanDefinition、推断构造方法、实例化、属性填充、初始化前、初始化后等步骤。其中AOP就是初始化后这一步骤中。
> （5）单例Bean创建完成后，Spring会发布一个容器启动事件。  
> （6）String启动结束。  
> （7）在源码中会更加复杂，比如1源码中会提供一些模板方法，让子类来实现。比如源码中还涉及到了一些BeanFactoryPostProcessor和BeanPostProcessor的注册，Spring的扫描就是通过BeanFactoryPostProcessor来实现的，依赖注入就是通过BeanPostProcessor来实现的。  
> （8）在Spring启动中还会处理@import等注解。
#### 10、Spring的优点和缺点
> 基于Spring的特性可以总结一些Spring的优点，比如：  
> （一）Spring是通过IOC集中管理对象的，对象和对象之间的**耦合性降低了**，对象蛋例、多例模式、懒加载、这些通过配置就可以实现，不用在编写代码，**方便了对象的维护**。    
> （二）基于AOP可以再不修改业务代码的情况对业务增强（比如业务监控、日志、安全）**减少重复代码提高了开发效率，维护也方便**。  
> （三）Spring可以通过声明@Transactional灵活的进行事务管理提高开发的效率。  
> （四）Spring有优秀的扩展能力，只需要简单配置就可以集成大量的第三方框架。  
> （五）Spring封装了一些java EE API的功能性方法，比如：JDBC、远程调用、邮件发送等，还是为了简化开发提高开发效率。  
> （六） Spring源码是提高java水平很好的范例。   
> 个人认为在应用层Spring是没有缺点的，如果必须说出缺点的话就是就是深入了解底层的Spring比较困难，上层使用越简单，底层封装越复杂，Spring源码有超过百万行。  
#### 11、什么是IOC容器，有什么作用？
> IOC通俗的讲就是控制反转，当引入了IOC就是将创建对象的控制权交给了Spring的IOC容器，不使用IOC的话对象是开发人员通过new对象去创建的，现在是由IOC创建对象，使用对象的话通过DI依赖注入获取， 
#### 12、Spring IOC的实现机制是什么？
> 通过简单工厂+反射机制实现的。  
#### 13、Spring的IOC和DI有什么区别？
> IOC控制反转是一种思想或者说是思路。DI是实现IOC的重要的一个环节。  
#### 14、紧耦合与松耦合的区别，如何编写松耦合的代码？
> 在java中紧耦合是指类与类之间高度依赖。（缺点：代码复用性低、维护成本高、连带性修改一个地方影响n个地方）。  
> 编写松耦合代码可以遵循java的五大原则来实现：  
> （1）单一性原则（将事物抽象为类）
> （2）接口分离原则（模块间通过接口隔离）
> （3）依赖倒置原则（反转依赖关系，上层不去依赖下层，都去依赖抽象；简单说就是层与层之间不能设计成垂直的，要有层次，是平行分层的）
>  符合以上三个原则可以降低耦合度，下面两个原则为扩展内容。
> （1）开闭原则（类、模块、方法要可扩展，不可修改）  
> （2）替换原则（提取相同的公共属性做一个基类，保证所以子类可以替换它的基类）
#### 15、 BeanFactory作用
> （一） BeanFactory是Spring的顶层接口。  
> （二） BeanFactory是Bean工厂主要是生产Bean。 
> （三） BeanFactory实现了简单工厂模式，通过getBean传入一个标识来生产bean。   
> （四） BeanFactory有非常多的实现类，这些工厂有不同的职责，其中最强大的工厂是DefaultListableBeanFactory，Spring底层就是用这个工厂生产Bean的。  
> （五） BeanFactory也是容器，应为它管理着Bean的生命周期。  
#### 16、BeanDefinition的作用
> BeanDefinition是存储Bean的定义信息的，比如比如是否多例、是否懒加载、是否抽象、是否自动装配等，它决定了Bean的生产方式。  
#### 17、BeanFactory和Applicationcontext的区别。
> （一） Applicationcontext是Spring的上下文，是Spring的容器。Applicationcontext实现了BeanFactory，Applicationcontext不生产Bean，是通知BeanFactory生产
> Bean，Applicationcontext和BeanFactory都可以作为Spring容器，都管理着Bean的生命周期。Applicationcontext做的事情更多，比如：自动将配置好的Bean注册、加载环境变量、支持多语言、这册对外扩展的接口等。  
#### 18、Spring ioc容器的加载过程。
> 当创建一个Spring容器或者加载一个Spring上下文的时候就开始了IOC的加载，IOC的加载就是创建Bean的过程。   
> （一） 实例化一个Applicationcontext的对象。  
> （二） 调用Bean工厂后置处理器完成扫描。  
> （三） 循环解析扫描出来的类的信息。   
> （四） 实例化一个BeanDefinition对象存储解析出来的类信息。    
> （五） 把实例化beanDefinition对象put到beanDefinitionMap中缓存起来，方便以后实例化Bean。  
> （六） 再次调用其他Bean工厂的后置处理器。  
> （七） 验证Bean定义是否懒加载、是否是单俐、是否抽象，验证通过后会在IOC加载时生产。
> （八） 在生产的时候会先去容器看一下是否有生产好的bean，有则返回，没有则生产。
> （九）通过反射拿到一个没有赋值属性的对象。  
> （十） 判断是否注入。
> （十一）判断Bean类型，回调Awaro接口。  
> （十二）调用生命周期回调方法。  
>  (十三）如果需要代理则完成代理（AOP)。
> （十四）put到单俐池，生产Bean完成，存在Spring容器中。
#### 19、Spring IOC扩展点有什么作用？
> IOC加载的时候提供的扩展接口或者说扩展方法。
#### 20、什么是SpringBean、javaBean、对象
> SpringBean是交给Spring管理的对象。javabean是符合规范的Java对象，比如类是公开的、属性是私有的、提供get和set方法等。  
#### 21、配置Bean的几种方法？
>  (一) xml方式。
>  (二) 注解方式需要配置扫面包。
>  (三) javaConfig@Bean方式自己实例化对象。
>  (四) @Input。
#### 22、Spring有几种作用域
>  (一) 单俐。
>  (二) 多例。
>  (三) Web应用的话有session、request、application作用域
#### 23、单俐Bean的优势或者说单俐设计模式
>  (一) 减少性能消耗、节省内存、提高内存利用率。
>  (二) 减少JVM的回收。
>  (三) 获取对象速度快，除第一次生成外都是从缓存获取。
#### 24、Spring是线程安全的吗？如何处理线程安全问题，  
> 单俐的Bean，如果类中声明了成员变量，并且有读写操作，就会线程不安全。  
> (一) 成员变量声明在方法中。  
> (二) 设置单俐Bean为多例Bean。
> (三) 同步锁，并行变船行影响吞吐量。 
> (四) 成员变量声明在方法内。  
#### 25、实例化Bean有几种方式？
> (一) 构造器通过反射方式。  
> (二) 静态工厂方式，指定静态方法实例对象。  
> (三) 实例工厂方式（@Bean）。 
> (四) FactoryBean，实现FactoryBean的接口。
#### 26、Bean的装配是什么？
> 就是SpirngBean之间互相依赖起来。 
#### 27、自动注入有什么限制吗？
> (一) 要声明set方法
> (二) 可以用constructor、property覆盖自动注入。  
> (三) 不能自动注入简单属性比如：基本数据类型、字符串等。
> (四) 自动注入不如显式装配精确。
#### 28、== 与 equals 的区别？
> (一) == 符号是比较两个对象的内存地址，equals方法是对比值是否一致。
> (二) 从逻辑上编写代码都应该使用equals方法，比如Integer前127位使用的是同一个内存空间，使用==比较返回true，超过128位==比较返回false。
#### 29、两个对象hashcode值相同，是否equals也一定为true？
