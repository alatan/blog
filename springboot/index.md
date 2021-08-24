# SpringBoot总结

> 使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置，简化Spring应用的初始搭建以及开发过程。简单理解，就是SpringBoot其实不是什么新的框架，它默认配置了很多框架的使用方式，就像Maven整合了所有的Jar包，Spring Boot整合了所有的框架。

## SpringBoot 的启动过程
开始源码分析，先从 SpringBoot 的启动类的 run() 方法开始看，以下是调用链：SpringApplication.run() -> run(new Class[]{primarySource}, args) -> new SpringApplication(primarySources)).run(args)。

一直在run，终于到重点了，我们直接看 new SpringApplication(primarySources)).run(args) 这个方法。
```java
public static ConfigurableApplicationContext run(Class<?>[] primarySources,	String[] args) {
    return new SpringApplication(primarySources).run(args);
}
```
上面的方法主要包括两大步骤：
1. 创建 SpringApplication 对象。
2. 运行 run() 方法。

### 创建 SpringApplication 对象
```java
public SpringApplication(ResourceLoader resourceLoader, Class... primarySources) { 
    this.sources = new LinkedHashSet(); 
    this.bannerMode = Mode.CONSOLE; 
    this.logStartupInfo = true; 
    this.addCommandLineProperties = true; 
    this.addConversionService = true; 
    this.headless = true; 
    this.registerShutdownHook = true; 
    this.additionalProfiles = new HashSet(); 
    this.isCustomEnvironment = false; 
    this.resourceLoader = resourceLoader; 
    Assert.notNull(primarySources, "PrimarySources must not be null"); 
    // 保存主配置类（这里是一个数组，说明可以有多个主配置类） 
    this.primarySources = new LinkedHashSet(Arrays.asList(primarySources)); 
    // 判断当前是否是一个 Web 应用 
    this.webApplicationType = WebApplicationType.deduceFromClasspath(); 
    // 从类路径下找到 META/INF/Spring.factories 配置的所有 ApplicationContextInitializer，然后保存起来 
    this.setInitializers(this.getSpringFactoriesInstances(ApplicationContextInitializer.class)); 
    // 从类路径下找到 META/INF/Spring.factories 配置的所有 ApplicationListener，然后保存起来 
    this.setListeners(this.getSpringFactoriesInstances(ApplicationListener.class)); 
    // 从多个配置类中找到有 main 方法的主配置类（只有一个） 
    this.mainApplicationClass = this.deduceMainApplicationClass(); 
}
```

### 运行 run() 方法
```java
public ConfigurableApplicationContext run(String... args) { 
    // 创建计时器 
    StopWatch stopWatch = new StopWatch(); 
    stopWatch.start(); 
    // 声明 IOC 容器 
    ConfigurableApplicationContext context = null; 
    Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList(); 
    this.configureHeadlessProperty(); 
    // 从类路径下找到 META/INF/Spring.factories 获取 SpringApplicationRunListeners 
    SpringApplicationRunListeners listeners = this.getRunListeners(args); 
    // 回调所有 SpringApplicationRunListeners 的 starting() 方法 
    listeners.starting(); 
    Collection exceptionReporters; 
    try { 
        // 封装命令行参数 
        ApplicationArguments applicationArguments = new DefaultApplicationArguments(args); 
        // 准备环境，包括创建环境，创建环境完成后回调 SpringApplicationRunListeners#environmentPrepared()方法，表示环境准备完成 
        ConfigurableEnvironment environment = this.prepareEnvironment(listeners, applicationArguments); 
        this.configureIgnoreBeanInfo(environment); 
        // 打印 Banner 
        Banner printedBanner = this.printBanner(environment); 
        // 创建 IOC 容器（决定创建 web 的 IOC 容器还是普通的 IOC 容器） 
        context = this.createApplicationContext(); 
        exceptionReporters = this.getSpringFactoriesInstances(SpringBootExceptionReporter.class, new Class[]{ConfigurableApplicationContext.class}, context); 
        /*
        * 准备上下文环境，将 environment 保存到 IOC 容器中，并且调用 applyInitializers() 方法
        * applyInitializers() 方法回调之前保存的所有的 ApplicationContextInitializer 的 initialize() 方法
        * 然后回调所有的 SpringApplicationRunListener#contextPrepared() 方法 
        * 最后回调所有的 SpringApplicationRunListener#contextLoaded() 方法 
        */
        this.prepareContext(context, environment, listeners, applicationArguments, printedBanner); 
        // 刷新容器，IOC 容器初始化（如果是 Web 应用还会创建嵌入式的 Tomcat），扫描、创建、加载所有组件的地方 
        this.refreshContext(context); 
        // 从 IOC 容器中获取所有的 ApplicationRunner 和 CommandLineRunner 进行回调 
        this.afterRefresh(context, applicationArguments); 
        stopWatch.stop(); 
        if (this.logStartupInfo) { 
        (new StartupInfoLogger(this.mainApplicationClass)).logStarted(this.getApplicationLog(), stopWatch); 
        } 
        // 调用 所有 SpringApplicationRunListeners#started()方法 
        listeners.started(context); 
        this.callRunners(context, applicationArguments); 
    } catch (Throwable var10) { 
        this.handleRunFailure(context, var10, exceptionReporters, listeners); 
        throw new IllegalStateException(var10); 
    } 

    try { 
        listeners.running(context); 
        return context; 
    } catch (Throwable var9) { 
        this.handleRunFailure(context, var9, exceptionReporters, (SpringApplicationRunListeners)null); 
        throw new IllegalStateException(var9); 
    } 
} 
```

### 总结
run() 阶段主要就是回调本节开头提到过的4个监听器中的方法与加载项目中组件到 IOC 容器中，而所有需要回调的监听器都是从类路径下的 META/INF/Spring.factories 中获取，从而达到启动前后的各种定制操作。

## SpringBoot 自动配置的原理
### @SpringBootApplication
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = {@Filter(type = FilterType.CUSTOM,classes = {TypeExcludeFilter.class}
), @Filter(type = FilterType.CUSTOM, classes = {AutoConfigurationExcludeFilter.class})})
public @interface SpringBootApplication {
```

1. @SpringBootConfiguration：我们点进去以后可以发现底层是Configuration注解，说白了就是支持JavaConfig的方式来进行配置(使用Configuration配置类等同于XML文件)。
2. @ComponentScan：就是扫描注解，默认是扫描当前类下的package。将@Controller/@Service/@Component/@Repository等注解加载到IOC容器中。
3. @EnableAutoConfiguration ：开启自动配置功能

### @EnableAutoConfiguration 详解
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@AutoConfigurationPackage
@Import({AutoConfigurationImportSelector.class})
public @interface EnableAutoConfiguration {
```

1. @AutoConfigurationPackage：自动配置包
2. @Import：给IOC容器导入组件

#### AutoConfigurationPackage 详解
> 从字面意思理解就是自动配置包。点进去可以看到就是一个 @Import 注解：@Import({Registrar.class})，导入了一个 Registrar 的组件。

```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@Import({Registrar.class})
public @interface AutoConfigurationPackage {
}
```

我们可以发现，依靠的还是 @Import注解，再点进去查看，我们发现重要的就是以下的代码：
```java
public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
    AutoConfigurationPackages.register(registry, (new AutoConfigurationPackages.PackageImport(metadata)).getPackageName());
}
```
@AutoConfigurationPackage 注解就是将主配置类（@SpringBootConfiguration标注的类）的所在包及下面所有子包里面的所有组件扫描到Spring容器中。所以说，默认情况下主配置类包及子包以外的组件，Spring 容器是扫描不到的。

在默认的情况下就是将：主配置类(@SpringBootApplication)的所在包及其子包里边的组件扫描到Spring容器中，看完这句话，会不会觉得，这不就是ComponentScan的功能吗？这俩不就重复了吗？

比如说，你用了Spring Data JPA，可能会在实体类上写@Entity注解。这个@Entity注解由 @AutoConfigurationPackage 扫描并加载，而我们平时开发用的@Controller/@Service/@Component/@Repository这些注解是由ComponentScan来扫描并加载的。简单理解：**这二者扫描的对象是不一样的。**



##### AutoConfigurationImportSelector
![](/images/spring/springboot/AutoConfigurationImportSelector.png "AutoConfigurationImportSelector")

调用链：在 getAutoConfigurationEntry() -> getCandidateConfigurations() -> loadFactoryNames(), 在这里 loadFactoryNames() 方法传入了 EnableAutoConfiguration.class 这个参数。

loadFactoryNames() 中关键的三步：
1. 从当前项目的类路径中获取所有 META-INF/spring.factories 这个文件下的信息。
2. 将上面获取到的信息封装成一个 Map 返回。
3. 从返回的 Map 中通过刚才传入的 EnableAutoConfiguration.class 参数，获取该 key 下的所有值。

![](/images/spring/springboot/loadFactoryNames.jpg "loadFactoryNames")

一般每导入一个第三方的依赖，除了本身的jar包以外，还会有一个 xxx-spring-boot-autoConfigure，这个就是第三方依赖自己编写的自动配置类

![](/images/spring/springboot/spring.factories.png "spring.factories")

可以看到 EnableAutoConfiguration 下面有很多类，这些就是我们项目进行自动配置的类。
1. 将类路径下 META-INF/spring.factories 里面配置的所有 EnableAutoConfiguration 的值加入到 Spring 容器中。
2. Spring会继续的处理这些自动配置类，也就是处理这些配置类中的@Configuration、@Import这些注解，继续递归处理这些注解，最后把相关的Bean都注册到容器中。

#### HttpEncodingAutoConfiguration
> 通过上面方式，所有的自动配置类就被导进主配置类中了。以 HttpEncodingAutoConfiguration为例来看一个自动配置类是怎么工作的。
```java
@Configuration 
@EnableConfigurationProperties({HttpProperties.class}) 
@ConditionalOnWebApplication( 
type = Type.SERVLET 
) 
@ConditionalOnClass({CharacterEncodingFilter.class}) 
@ConditionalOnProperty( 
prefix = "spring.http.encoding", 
value = {"enabled"}, 
matchIfMissing = true 
) 
public class HttpEncodingAutoConfiguration { 
```
* @Configuration：标记为配置类。
* @ConditionalOnWebApplication：web应用下才生效。
* @ConditionalOnClass：指定的类（依赖）存在才生效。
* @ConditionalOnProperty：主配置文件中存在指定的属性才生效。
* @EnableConfigurationProperties({HttpProperties.class})：启动指定类的ConfigurationProperties功能；将配置文件中对应的值和 HttpProperties 绑定起来；并把 HttpProperties 加入到 IOC 容器中。

因为 @EnableConfigurationProperties({HttpProperties.class}) 把配置文件中的配置项与当前 HttpProperties 类绑定上了。然后在 HttpEncodingAutoConfiguration 中又引用了 HttpProperties ，所以最后就能在 HttpEncodingAutoConfiguration 中使用配置文件中的值了。最终通过 @Bean 和一些条件判断往容器中添加组件，实现自动配置。（当然该Bean中属性值是从 HttpProperties 中获取）

##### HttpProperties
HttpProperties 通过 @ConfigurationProperties 注解将配置文件与自身属性绑定。

所有在配置文件中能配置的属性都是在 xxxProperties 类中封装着；配置文件能配置什么就可以参照某个功能对应的这个属性类。
```java
@ConfigurationProperties( 
prefix = "spring.http" 
)// 从配置文件中获取指定的值和bean的属性进行绑定 
public class HttpProperties { 
```

### 总结
@SpringBootApplication 上有三个注解： @SpringBootConfiguration ，@EnableAutoConfiguration ，@ComponentScan ，@EnableAutoConfiguration 是关键(启用自动配置)，内部实际上就去加载 META-INF/spring.factories 文件的信息，然后筛选出以 EnableAutoConfiguration 为key的数据，加载到IOC容器中，实现自动配置功能。

## 参考文章
*. [Spring Boot 总结](https://juejin.cn/post/6844903849178873870 "Spring Boot 总结")
