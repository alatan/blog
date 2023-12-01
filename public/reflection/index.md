# 框架的灵魂-反射

  
## 什么是反射
*简而言之，通过反射，我们可以在运行时获得程序中每一个类型的成员和成员的信息。
程序中一般的对象的类型都是在编译期就确定下来的，而 Java 反射机制可以动态地创建对象并调用其属性，这样的对象的类型在编译期是未知的。
所以我们可以通过反射机制直接创建对象，即使这个对象的类型在编译期是未知的。*

**反射的核心是 JVM 在运行时才动态加载类或调用方法/访问属性，它不需要事先（写代码的时候或编译期）知道运行对象是谁。**
### Java 反射主要提供以下功能
* 在运行时构造任意一个类的对象。
* 在运行时调用任意一个对象的方法。
* 在运行时判断任意一个对象所属的类。
* 在运行时判断任意一个类所具有的成员变量和方法（通过反射甚至可以调用private方法）。

## 反射的主要用途
**反射最重要的用途就是开发各种通用框架。**

很多框架（比如 Spring）都是配置化的（比如通过 XML 文件配置 Bean），为了保证框架的通用性，它们可能需要根据配置文件加载不同的对象或类，调用不同的方法，这个时候就必须用到反射，运行时动态加载需要加载的对象。

举一个例子，在运用 Struts 2 框架的开发中我们一般会在 struts.xml 里去配置 Action，比如：
```xml
    <action name="login" class="org.ScZyhSoft.test.action.SimpleLoginAction" method="execute">
        <result>/shop/shop-index.jsp</result>
        <result name="error">login.jsp</result>
    </action>
```
配置文件与 Action 建立了一种映射关系，当 View 层发出请求时，请求会被 StrutsPrepareAndExecuteFilter 拦截，然后 StrutsPrepareAndExecuteFilter 会去动态地创建 Action 实例。比如我们请求 login.action，那么 StrutsPrepareAndExecuteFilter就会去解析struts.xml文件，检索action中name为login的Action，并根据class属性创建SimpleLoginAction实例，并用invoke方法来调用execute方法，这个过程离不开反射。

对与框架开发人员来说，反射虽小但作用非常大，它是各种容器实现的核心。而对于一般的开发者来说，不深入框架开发则用反射用的就会少一点，不过了解一下框架的底层机制有助于丰富自己的编程思想，也是很有益的。

**像Java中的一大利器注解的实现也用到了反射。**

为什么你使用 Spring 的时候 ，一个@Component注解就声明了一个类为 Spring Bean 呢？为什么你通过一个 @Value注解就读取到配置文件中的值呢？究竟是怎么起作用的呢？

这些都是因为你可以基于反射分析类，然后获取到类/属性/方法/方法的参数上的注解。你获取到注解之后，就可以做进一步的处理。

## 反射的基本运用
### 获得Class对象
使用Class类的forName 静态方法。

```java
    Class appleClass = Class.forName("base.reflection.Apple");
```
直接获取

```java
    Class appleClass = Apple.class;
```

获取某一个对象的class

```java
    Apple apple = new Apple();
    Class appleClass = apple.getClass();
```
通过类加载器ClassLoader.loadClass()传入类路径获取

```java
    Class appleClass = ClassLoader.getSystemClassLoader().loadClass("base.reflection.Apple");
```
### 构造方法
```java
    Constructor[] declaredConstructors = appleClass.getDeclaredConstructors();
    Constructor[] constructors = appleClass.getConstructors();
    //通过无参构造来获取该类对象 newInstance()
    Apple apple= (Apple)appleClass.newInstance();
    //通过有参构造来获取该类对象 newInstance
    Constructor constructor = appleClass.getConstructor(String.class,int.class,int.class);
    Apple apple=(Apple)constructor.newInstance("红色",10,5);
```
### 属性
```java
      //getDeclaredFields所有已声明的成员变量，但不能得到其父类的成员变量
      Field[] declaredFields = appleClass.getDeclaredFields();
      //getFields访问公有的成员变量
      Field[] fields = appleClass.getFields();
```
### 方法
```java
    //getDeclaredMethods 方法返回类或接口声明的所有方法，包括公共、保护、默认（包）访问和私有方法，但不包括继承的方法。
    Method[] declaredMethods = appleClass.getDeclaredMethods();
    //getMethods方法返回某个类的所有公用（public）方法，包括其继承类的公用方法。
    Method[] methods = appleClass.getMethods();
```
### 调用方法
```java
    Constructor constructor = appleClass.getConstructor(String.class,int.class,int.class);
    Apple apple = (Apple)constructor.newInstance("红色",10,5);
    //获取toString方法并调用
    Method method = appleClass.getDeclaredMethod("toString");
    String str = (String)method.invoke(apple);
    System.out.println(str);
```
### 利用反射创建数组
```java
    Class<?> cls = Class.forName("java.lang.String");
    Object array = Array.newInstance(cls,5);
    //往数组里添加内容
    Array.set(array,0,"hello");
    Array.set(array,1,"Java");
    Array.set(array,2,"fuck");
    Array.set(array,3,"Scala");
    Array.set(array,4,"Clojure");
    //获取某一项的内容
    System.out.println(Array.get(array,3));
```
