---  
title: "Java七武器-反射"  
date: 2020-02-03
weight: 70  
markup: mmark  
draft: false  
keywords: [""]  
description: "反射让一切有了可能"  
tags: ["Java基础"]  
categories: ["Java基础"]  
author: "默哥"  
---  
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

    <action name="login" class="org.ScZyhSoft.test.action.SimpleLoginAction" method="execute">
        <result>/shop/shop-index.jsp</result>
        <result name="error">login.jsp</result>
    </action>

配置文件与 Action 建立了一种映射关系，当 View 层发出请求时，请求会被 StrutsPrepareAndExecuteFilter 拦截，然后 StrutsPrepareAndExecuteFilter 会去动态地创建 Action 实例。比如我们请求 login.action，那么 StrutsPrepareAndExecuteFilter就会去解析struts.xml文件，检索action中name为login的Action，并根据class属性创建SimpleLoginAction实例，并用invoke方法来调用execute方法，这个过程离不开反射。

对与框架开发人员来说，反射虽小但作用非常大，它是各种容器实现的核心。而对于一般的开发者来说，不深入框架开发则用反射用的就会少一点，不过了解一下框架的底层机制有助于丰富自己的编程思想，也是很有益的。

## 反射的基本运用
### 获得Class对象
使用Class类的forName 静态方法。

    Class appleClass = Class.forName("base.reflection.Apple");
直接获取

    Class appleClass = Apple.class;
获取某一个对象的class

    Apple apple = new Apple();
    Class appleClass = apple.getClass();
