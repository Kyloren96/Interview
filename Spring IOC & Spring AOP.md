# Spring IOC & Spring AOP

## IOC 是什么，什么是 Spring IOC 容器？⭐

IOC 是一种设计思想。 **IOC 容器是 Spring 用来实现 IOC 的载体， IOC 容器在某种程度上就是个Map（key，value）,key是 name 属性，value 是对应的对象。**容器创建 Bean 对象， 使用依赖注入来管理对象之间的相互依赖关系，配置它们并管理它们的完整生命周期，很大程度上简化应用的开发，降低了耦合度。

容器通过读取提供的配置,比如 XML，注解或 Java 代码来接收对象信息进行实例化，配置和组装。

Spring在创建容器时有一个点就是利用了模板方法设计模式设计了 refresh 方法，这个方法是模板方法，低级容器实现了 obtainFreshBeanFactory 的抽象方法，调用 refreshBeanFactory 加载了所有 BeanDefinition 和 Properties 到 **DefaultListableBeanFactory** 容器中。发送了注册事件后高级容器启动功能，比如接口回调，监听器，创建单例bean，发布事件等功能。

### IoC 的实现机制/初始化流程⭐

主要实现原理就是工厂模式加反射机制。

调用 refresh() 方法：

- 刷新准备，设置开始时间，状态， 初始化占位符等操作
- 获取内部的 BeanFactory，Spring 容器在启动的时候，先会保存所有注册进来的 Bean 的定义信息， 注册到 BeanFactory 中。
- 设置 BeanFactory 的类加载器和后置处理器，添加几个 BeanPostProcessor，手动注册默认的环境 bean
- 为子类提供后置处理 BeanFactory 的扩展能力，初始化上下文之前，可以复写 postProcessBeanFactory这个方法
- 执行 Context 中注册的 BeanFactory 后置处理器，对 SpringBoot 来说，这一步会进行 BeanDefintion 的解析
- 按优先级在 BeanFactory 注册 Bean 的后置处理器，这是在 Bean 初始化前后执行的方法
- 初始化国际化，事件广播器的模块，注册事件监听器
- 然后 **Spring容器就会创建这些非延迟加载的单例 Bean**
- 最后广播事件，ApplicationContext 初始化/刷新完成

具体源码实现分析请看我的另一篇文章 。

## Spring Bean

### 什么是 Spring Bean？

是基于用户提供的配置创建的，构成了应用程序主干的对象，是由 Spring IoC 容器实例化，装配和管理的。

**Bean 在代码层面上可以简单认为是 BeanDefinition 的实例**

### Bean 的作用域（scope）

- **Singleton** - 每个 Spring IoC 容器仅有一个单实例。
- **Prototype** - 每次请求都会产生一个新的实例。
- **Request** - 每次请求都会创建一个实例
- **Session** - 在一个会话周期内只有一个实例
- Global-session - 类似于标准的 HTTP Session 作用域，5.0版本后已不再使用
- **Appilcation** - 在一个 ServletContext 中只有一个实例
- **Websocket** - 在一个 Websocket 只有一个实例

仅当用户使用支持 Web 的 ApplicationContext 时，最后几个才可用。

### Bean 的生命周期⭐

- Bean容器/BeanFactory 通过对象的构造器或工厂方法先实例化 Bean；

- 再根据 Resource 中的信息再通过设定好的方法（典型的有setter，统称为BeanWrapper）对 Bean 设置属性值，得到 BeanDefintion 对象，然后 put 到 beanDefinitionMap 中，调用 getBean 的时候，从 beanDefinitionMap 里拿出 Class 对象进行注入（**使用了反射**），同时如果有依赖关系，将递归调用 getBean 方法，即依赖注入的过程。

- 检查 xxxAware 相关接口，比如 BeanNameAware，BeanClassLoaderAware，ApplicationContextAware（ BeanFactoryAware）等等，如果有就调用相应的 setxxx 方法把所需要的xxx传入到 Bean 中。

  **补充**：关于 Aware ，Aware 就是感知的意思， Aware 的目的是为了让Bean获得Spring容器的服务。 实现了这类接口的 bean 会存在“意识感”，从而让容器调用 setxxx 方法把所需要的 xxx 传到 Bean 中。

- 此时检查是否存在有于 Bean 关联的任何 BeanPostProcessors， 执行 postProcessBeforeInitialization() 方法（前置处理器）。

- 如果 Bean 实现了InitializingBean接口（正在初始化的 Bean），执行 afterPropertiesSet() 方法。

- 检查是否配置了自定义的 init-method 方法，如果有就调用。

- 此时检查是否存在有于 Bean 关联的任何 BeanPostProcessors， 执行 postProcessAfterInitialization() 方法（后置处理器）。返回 wrapperBean（包装后的 Bean）。

- 这时就可以开始使用 Bean 了，当容器关闭时，会检查 Bean 是否实现了 DisposableBean 接口，如果有就调用 destory() 方法。

- 如果 Bean 配置文件中的定义包含 destroy-method 属性，执行指定的方法。

上面整个过程就是 Bean 的整个生命周期了。

### **Bean 单例和多例的情况：**

在实际情况中一般并不会实现很多扩展接口，我们知道，Bean 的基本类型分为 singleton（单例） 和 prototype（原型/多例） 两种，在容器创建过程中，单例 Bean 默认跟随容器一起实例化，而当我们指定 Bean节点的 lazy-init=”true” 时，只有在第一次获取 Bean 的时候才会初始化 Bean。当然，如果想让所有单例 Bean 都延迟加载，可以在根节点设置此属性。

当 scope="prototype" 时，容器也会延迟初始化 Bean，并不会立刻创建对象，而是在第一次请求该 bean 时才初始化（如调用 getBean 方法时）。和单例不同的情况是：在对象销毁时，容器不会帮我们调用任何方法。

Spring不能对一个 prototype bean 的整个生命周期负责：容器在初始化、配置、装饰或者是装配完一个 prototype 实例后，将它交给客户端，随后就对该prototype 实例不闻不问了。

也许你会问，那么怎么释放被 prototype 作用域 bean 占用的资源？

我们可以通过 Bean 的后置处理器， 该处理器持有要被清除的bean的引用。

Spring 被设计成**一个管理应用程序模块定义的容器以及工厂，而不是管理模块自身的容器**，让模块可以分别独立开发，实现了模块之间的解耦。

为什么要使用Spring：

1. Spring提供一个容器/工厂，统一管理模块的定义，根据需要创建。
2. 把模块的配置参数统一管理，模块不需要自行读取配置。
3. Spring提供依赖注入，把依赖的模块自动推送进来，不需要模块自己拉取。
4. 此外，Spring提供了对很多其他第三方框架的集成功能，减少了样板代码（boilerplate）。

## 常见扩展接口

BeanFactoryPostProcessor：处理所有bean前,对bean factory进行预处理 BeanDefinitionRegistryPostProcessor：可以添加自定义的bean BeanPostProcessor：支持在Bean初始化前、后对bean进行处理 ApplicationContextAware：可以获得ApplicationContext及其中的bean InitializingBean：在bean创建完成,所有属性注入完成后执行 DisposableBean：在bean销毁前执行 ApplicationListener：用来监听产生的应用事件

### Spring的后置处理器

1. BeanPostProcessor：Bean的后置处理器，主要在bean初始化前后工作。
2. InstantiationAwareBeanPostProcessor：继承于BeanPostProcessor，主要在实例化bean前后工作； AOP创建代理对象就是通过该接口实现。
3. BeanFactoryPostProcessor：Bean工厂的后置处理器，在bean定义(bean definitions)加载完成后，bean尚未初始化前执行。
4. BeanDefinitionRegistryPostProcessor：继承于BeanFactoryPostProcessor。其自定义的方法postProcessBeanDefinitionRegistry会在bean定义(bean definitions)将要加载，bean尚未初始化前真执行，即在BeanFactoryPostProcessor的postProcessBeanFactory方法前被调用。

### spring 自动装配 bean 有哪些方式？

Spring容器负责创建应用程序中的bean同时通过ID来协调这些对象之间的关系。作为开发人员，我们需要告诉Spring要创建哪些bean并且如何将其装配到一起。

spring中bean装配有两种方式：

- 隐式的bean发现机制和自动装配
- 在java代码或者XML中进行显示配置

### ApplicationContext和BeanFactory的区别⭐

```
public interface ApplicationContext extends EnvironmentCapable, ListableBeanFactory, HierarchicalBeanFactory,
    MessageSource, ApplicationEventPublisher, ResourcePatternResolver {} 
```

- BeanFactory 粗暴简单，可以理解为就是个 HashMap，Key 是 BeanName，Value 是 Bean 实例。通常只提供实例化对象 和获取这两个功能。BeanFactory在启动的时候不会去实例化Bean，中有从容器中拿Bean的时候才会去实例化。
- ApplicationContext（**应用上下文**） 可以称之为 “高级容器”。因为他比 BeanFactory 多了更多的功能，继承了多个接口，因此具备了更多的功能。 如国际化，访问资源，载入多个（有继承关系）上下文 ，消息发送、响应机制，AOP等。

## 循环依赖

### 循环依赖是什么？⭐

bean A依赖于另一个bean B时，bean B依赖于bean A：

当Spring上下文加载所有bean时，它会尝试按照它们完全工作所需的顺序创建bean。例如，如果我们没有循环依赖，如下例所示：

豆A→豆B→豆C.

Spring将创建bean C，然后创建bean B（并将bean注入其中），然后创建bean A（并将bean B注入其中）。

但是，当具有循环依赖时，Spring无法决定应该首先创建哪个bean，因为它们彼此依赖。

以setter方式构成的循环依赖，spring可以帮你解决，以构造器方式构成的循环依赖，spring无法搞定。

setter 注入和构造器注入的区别就在于创建bean的过程中，setter注入可以先用无参数构造方法返回bean实例，再注入依赖的属性，使用到了 Spring 的三级缓存，而constructor方式**无法先返回bean的实例，必须先实例化它所依赖的属性，这样一来就会造成死循环所以会失败。**

我们使用比较多的在属性上@Autowired的方式，在spring内部的处理中，与setter方法不太一样，但用此种方式循环依赖也可以解决，原理同上，只要能先实例化出来，提前暴露出来，就可以解决循环依赖的问题。

### Spring 如何解决循环依赖？⭐

Spring使用了三级缓存解决了循环依赖的问题。在populateBean()给属性赋值阶段里面Spring会解析你的属性，并且赋值，当发现，A对象里面依赖了B，此时又会走getBean方法，但这个时候，你去缓存中是可以拿的到的。因为我们在对createBeanInstance对象创建完成以后已经放入了缓存当中，所以创建B的时候发现依赖A，直接就从缓存中去拿，此时B创建完，A也创建完。至此Bean的创建完成，最后将创建好的Bean放入单例缓存池中。

上面已经大概的解释了一下，现在来详细说说：

**我们知道，Bean 的创建最为核心三个方法解释如下：**

- `createBeanInstance`：例化，其实也就是调用对象的**构造方法**实例化对象
- `populateBean`：填充属性，这一步主要是对bean的依赖属性进行注入(`@Autowired`)
- `initializeBean`：回到一些形如`initMethod`、`InitializingBean`等方法

从对单例Bean的初始化可以看出，循环依赖主要发生在**第二步（populateBean）**，也就是field属性注入的处理。

**现在再来了解一下三级缓存：**

1. `singletonObjects`：第一级，单例缓存池。用于存放完全初始化好的 bean，**从该缓存中取出的 bean 可以直接使用**
2. `earlySingletonObjects`：第二级。提前曝光的单例对象的cache，存放原始的 bean 对象（尚未填充属性的 bean）
3. `singletonFactories`：第三级，单例对象工厂缓存 。单例对象工厂的cache，存放 bean 工厂对象

**了解完缓存就可以开始了解单例 Bean 的创建过程：**

1. 先从一级缓存singletonObjects中去获取。（如果获取到就直接return）
2. 如果获取不到或者对象正在创建中（isSingletonCurrentlyInCreation()），那就再从二级缓存earlySingletonObjects中获取。（如果获取到就直接return）
3. 如果还是获取不到，且允许singletonFactories（allowEarlyReference=true）通过getObject()获取。就从三级缓存singletonFactory.getObject()获取。**（如果获取到了就把这个 bean 从 singletonFactories 中移除，并且放进 earlySingletonObjects。其实也就是从三级缓存移动（是剪切）到了二级缓存）**

**解决循环依赖的关键就在这个三级缓存，下面给出 A,B 循环依赖的情况 Spring 是如何操作的：**

1. 使用context.getBean(A.class)，为了获取容器内的单例A，若A不存在，就会走A这个Bean的创建流程，显然初次获取A是不存在的，所以开始创建 A
2. 开始实例化A（**createBeanInstance**，注意此处仅仅是实例化），并将它放进缓存（此时A已经实例化完成，已经可以被引用了）
3. 开始准备初始化A（populateBean）：解析 A 的依赖发现依赖注入了 B（此时需要去容器内获取B）
4. 此时开始实例化 B，到了依赖注入B时，会通过getBean(B)去容器内找B。
5. 实例化B，并将其放入缓存。（此时B也能够被引用了）
6. 开始准备初始化B，发现依赖注入A（此时需要去容器内获取A）
7. **此处重要**：初始化B时会调用getBean(A)去容器内找到A，上面我们已经说过了此时候因为A已经实例化完成了并且放进了缓存里，所以这个时候去看缓存里是已经存在A的引用了的，所以getBean(A)能够正常返回
8. **B初始化成功**，return（注意此处return相当于是返回最上面的 getBean(B) 这句代码，回到了初始化A的流程中）。
9. 因为B实例已经成功返回了，因此最终**A也初始化成功**
10. 到此，B持有的已经是初始化完成的A，A持有的也是初始化完成的B

## AOP

### 解释一下什么是 AOP？

AOP（Aspect-Oriented Programming，面向切面编程），可以说是 OOP 的补充和完善。OOP 定义了从上到下的关系，但并不适合定义从左到右的关系，例如权限认证、日志、事务处理。这些导致了大量代码的重复，而不利于各个模块的重用。而AOP技术将那些与业务无关，却为业务模块所共同调用的逻辑或责任封装起来，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。

### AOP 的原理（重要）⭐

开启了 AOP 功能后，容器在注册 Bean 的后置处理器的时候，就会注册一个相关的后置处理器（AspectJAutoProxyCreator），在创建单实例 Bean 的时候，这个后置处理器就会拦截业务逻辑组件和切面组件的创建过程，怎么拦截呢？就是等组件创建完后，判断是否是通知方法，如果是就把切面的通知方法包装成增强器（Advisor），给业务逻辑组件（目标类）创建一个代理对象，代理的方式由 Spring 来判断返回这个代理对象，容器创建完成后这个代理对象执行目标方法的时候，执行拦截器链依次执行通知方法。

### JDK 动态代理和 CGLIB 的区别⭐

**静态代理与动态代理** 静态代理，是编译时增强，AOP 框架会在编译阶段生成 AOP 代理类，在程序运行前代理类的.class文件就已经存在了。常见的实现：JDK静态代理，AspectJ 。 动态代理，是运行时增强，它不修改代理类的字节码，而是在程序运行时，运用反射机制，在内存中临时为方法生成一个AOP对象，这个AOP对象包含了目标对象的全部方法，并且在特定的切点做了增强处理，并回调原对象的方法。

JDK 动态代理基于接口，所以只有接口中的方法会被增强，而 CGLIB 基于类继承，需要注意就是如果方法使用了 final 修饰，或者是 private 方法，是不能被增强的。

### 实现原理

JDK动态代理：基于反射，利用反射机制生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理。 CGLIB动态代理：基于操作字节码，通过加载代理对象的类字节码，为代理对象创建一个子类，并在子类中拦截父类方法并织入方法增强逻辑。底层是依靠ASM（开源的java字节码编辑类库）操作字节码实现的。

### springAOP 项目中的实际应用

只需要事务处理那些操作数据库的方法，所以使用了自定义注解

建议从项目中的事务注解讲

如果项目有实际应用的AOP自定义注解更好,

没有也不要慌把spring自带的事务注解讲明白也可以

面试官主要考察有没有AOP的思想,

实际项目中AOP应用太广泛了 日志,权限,事务等等都可以利用AOP去完成统一设计.







