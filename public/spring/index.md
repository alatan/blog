# Spring总结

> Spring 是一种轻量级开发框架，旨在提高开发人员的开发效率以及系统的可维护性。

**我们一般说 Spring 框架指的都是 Spring Framework，它是很多模块的集合，使用这些模块可以很方便地协助我们进行开发。这些模块是：核心容器、数据访问/集成、Web、AOP（面向切面编程）、工具、消息和测试模块。比如：Core Container 中的 Core 组件是Spring 所有组件的核心，Beans 组件和 Context 组件是实现IOC和依赖注入的基础，AOP组件用来实现面向切面编程。**

Spring 官网列出的 Spring 的 6 个特征:
* 核心技术 ：依赖注入(DI)，AOP，事件(events)，资源，i18n，验证，数据绑定，类型转换，SpEL。
* 测试 ：模拟对象，TestContext框架，Spring MVC 测试，WebTestClient。
* 数据访问 ：事务，DAO支持，JDBC，ORM，编组XML。
* Web支持 : Spring MVC和Spring WebFlux Web框架。
* 集成 ：远程处理，JMS，JCA，JMX，电子邮件，任务，调度，缓存。
* 语言 ：Kotlin，Groovy，动态语言。

![](/images/spring/spring-model.png "Spring模块")
## IOC
### IOC容器初始化过程
BeanFactory和ApplicationContext是Spring中两种很重要的容器，前者提供了最基本的依赖注入的支持，后者在继承前者的基础上进行了功能的拓展，增加了事件传播，资源访问，国际化的支持等功能。同时两者的生命周期也稍微有些不同。

Spring IOC容器初始化过程分为Resource定位，载入解析，注册。**IOC容器初始化过程中不包含Bean的依赖注入。Bean的依赖注入一般会发生在第一次通过getBean向容器索取Bean的时候。**

![](/images/spring/iocInit.png "IOC容器初始化过程")

#### 关键步骤
* IOC容器初始化入口是在构造方法中调用refresh开始的。
* 通过ResourceLoader来完成资源文件位置的定位，DefaultResourceLoader是默认的实现，同时上下文本身就给除了ResourceLoader的实现。
* 创建的IOC容器是DefaultListableBeanFactory。
* IOC对Bean的管理和依赖注入功能的实现是通过对其持有的BeanDefinition进行相关操作来完成的。
* 通过BeanDefinitionReader来完成定义信息的解析和Bean信息的注册。
* XmlBeanDefinitionReader是BeanDefinitionReader的实现了，通过它来解析xml配置中的bean定义。
* 实际的处理过程是委托给BeanDefinitionParserDelegate来完成的。得到Bean的定义信息，这些信息在Spring中使用BeanDefinition对象来表示。
* BeanDefinition的注册是由BeanDefinitionRegistry实现的registerBeanDefiition方法进行的。内部使用ConcurrentHashMap来保存BeanDefiition。

#### Spring解决循环依赖的过程总结
Spring在初始化Bean的时候，会先初始化当前Bean所依赖的Bean，如果两个Bean互相依赖，就产生了循环依赖，Spring针对循环依赖的办法是：提前曝光加上三个缓存singletonObjects、earlySingletonObjects、singletonFactories。

假设当前Bean是A，A依赖的Bean是B，B又依赖A。

提前曝光的意思就是，当前Bean A实例化完，还没有初始化完就先把当前Bean曝光出去，在B初始化需要依赖A的时候，就先拿到提前曝光的A，这样就可以继续将B初始化完成，然后返回A继续进行初始化。

循环依赖解决只针对单例Bean。
#### 总结
1. Spring启动。
2. 加载配置文件，xml、JavaConfig、注解、其他形式等等，将描述我们自己定义的和Spring内置的定义的Bean加载进来。
3. 加载完配置文件后将配置文件转化成统一的Resource来处理。
4. 使用Resource解析将我们定义的一些配置都转化成Spring内部的标识形式：BeanDefinition。
5. 在低级的容器BeanFactory中，到这里就可以宣告Spring容器初始化完成了，Bean的初始化是在我们使用Bean的时候触发的；在高级的容器ApplicationContext中，会自动触发那些1. lazy-init=false的单例Bean，让Bean以及依赖的Bean进行初始化的流程，初始化完成Bean之后高级容器也初始化完成了。
6. 在我们的应用中使用Bean。
7. Spring容器关闭，销毁各个Bean。

### SpringBean生命周期
![](/images/spring/beanLife.png "SpringBean生命周期")

1. 手动或者自动的触发获取一个Bean，使用BeanFactory的时候需要我们代码自己获取Bean，ApplicationContext则是在IOC启动的时候自动初始化一个Bean。
2. IOC会根据BeanDefinition来实例化这个Bean，如果这个Bean还有依赖其他的Bean则会先初始化依赖的Bean，这里又涉及到了循环依赖的解决。实例化Bean的时候根据工厂方法、构造方法或者简单初始化等选择具体的实例来进行实例化，最终都是使用反射进行实例化。
3. Bean实例化完成，也就是一个对象实例化完成后，会继续填充这个Bean的各个属性，也是使用反射机制将属性设置到Bean中去。
4. 填充完属性后，会调用各种Aware方法，将需要的组件设置到当前Bean中。BeanFactory这种低级容器需要我们手动注册Aware接口，而ApplicationContext这种高级容器在IOC启动的时候就自动给我们注册了Aware等接口。
5. 接下来如果Bean实现了PostProcessor一系列的接口，会先调用其中的postProcessBeforeInitialization方法。BeanFactory这种低级容器需要我们手动注册PostProcessor接口，而ApplicationContext这种高级容器在IOC启动的时候就自动给我们注册了PostProcessor等接口。
6. 如果Bean实现了InitializingBean接口，则会调用对应的afterPropertiesSet方法。
7. 如果Bean设置了init-method属性，则会调用init-method指定的方法。
8. 接下来如果Bean实现了PostProcessor一系列的接口，会先调用其中的postProcessAfterInitialization方法。BeanFactory这种低级容器需要我们手动注册PostProcessor接口，而 ApplicationContext这种高级容器在IOC启动的时候就自动给我们注册了PostProcessor等接口。
9. 到这里Bean就可以使用了。
10. 容器关闭的时候需要销毁Bean。
11. 如果Bean实现了DisposableBean，则调用destroy方法。
12. 如果Bean配置了destroy-method属性，则调用指定的destroy-method方法。

## AOP
Spring AOP流程大致上可以分为三个阶段：标签解析和AutoProxyCreator的注册、AOP代理的创建、代理的使用。

### 标签解析和AutoProxyCreator的注册
在Spring的扩展点中，最早期的扩展点是NamespaceHandler，这个阶段是在解析成BeanDefinition的阶段。Spring在这里完成自定义标签的解析工作，比如aop、tx等等。AOP功能在这里注册了自己的NamespaceHandler以及BeanDefinitionParser，用来将AOP标签转换成BeanDefinition交给IOC容器管理。

### 关于AutoProxyCreator的理解
同时在这里也会注册一个AutoProxyCreator，这个组件是用来在后面Bean的初始化过程中生成代理的。这个AutoProxtCreator实现了一个接口是：SmartInstantiationAwareBeanPostProcessor，看起来很眼熟，SmartInstantiationAwareBeanPostProcessor这个接口的父接口是：InstantiationAwareBeanPostProcessor，而InstantiationAwareBeanPostProcessor的父接口是BeanPostProcessor，到这里我们可能就大概明白了。

我们知道实现了BeanPostProcessor接口的类会在Bean初始化过程中的填充属性之后这一步被调用，调用的方法是postProcessBeforeInitialization和postProcessAfterInitialization。但是Spring干嘛还要衍生出那么多子接口呢？通过接口的名字我们可以看到不同，那些接口名字都含有一个关键词：Instantiation实例化，并不是初始化Initialization，也就是说这些接口中的方法调用是要在Bean实例化的时候进行处理。在Bean的生命周期中，我们知道Bean的实例化是Bean初始化步骤中最早的一步，所以对于Instantiation等方法的处理会比Initialization要早。

试想一下，我们自己写这些逻辑的时候，会在什么时候去创建AOP代理？第一个时间点：在Bean实例化之前，我就通过创建代理的逻辑直接返回一个代理好的实例，就不用继续走Bean初始化的后面的步骤了；第二个时间点：在Bean初始化之后，也就是走完了所有的Bean初始化过程后生成了一个完整的Bean，我再进行代理的创建。Spring就是这么处理的，要么我就不用Spring创建Bean，我直接返回一个代理，要么我就等Spring创建完成一个Bean再返回一个代理。Spring还有会另外一个触发点创建代理：getEarlyBeanReference，用来在解决循环依赖时提前曝光的Bean的代理生成，暂时不做说明。

### AOP代理创建
明确了代理创建的时间点，就可以继续看AOP代理的创建过程了。
1. 筛选出所有适合当前Bean的通知器，也就是所有的Advisor、Advise、Interceptor。
2. 选择使用JDK还是CGLIB来进行创建代理。
3. 使用具体的代理实现来创建代理。

### 代理的使用
1. 获取当前调用方法的拦截器链，包含了所有将要执行的advice。
2. 如果没有任何拦截器，直接执行目标方法。
3. 如果有拦截器存在，则将拦截器和目标方法封装成一个MethodInvocation，递归调用proceed方法进行调用。
4. 上面的处理中还有对目标对象的自我方法调用实施增强的处理，比如平时遇到的问题：在同一个类中一个方法调用另外一个带事务注解的方法，事务不会生效；在同一个类中一个方法调用另外一个带缓存注解的方法，缓存不会生效。

以上就是大概的流程，总结一下就是：**AOP实现使用的是动态代理和拦截器链。**

### 动态代理实现步骤  
1. 实现InvocationHandler接口创建自己的调用处理器 
1. 给Proxy类提供ClassLoader和代理接口类型数组创建动态代理类 
1. 以调用处理器类型为参数，利用反射机制得到动态代理类的构造函数 
1. 以调用处理器对象为参数，利用动态代理类的构造函数创建动态代理类对象 

## 学习资料
* [AOP介绍](https://mp.weixin.qq.com/s?__biz=MzAxOTc0NzExNg==&mid=2665513187&idx=1&sn=f603eee3e798e79ce010c9d58cd2ecf3)
