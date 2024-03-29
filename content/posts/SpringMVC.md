---  
title: "SpringMVC总结"
description: "SpringMVC总结"
keywords: ["Spring"]
date: 2019-04-02
author: "默哥"
weight: 70
draft: false

categories: ["Spring"]
tags: ["Spring", "大纲"]  
toc: 
    auto: false
---

## SpringMVC 执行流程
1. 用户请求发送到前端控制器DispatcherServlet。
2. 前端控制器DispatcherServlet接收到请求后，DispatcherServlet会使用HandlerMapping来处理，HandlerMapping会查找到具体进行处理请求的Handler对象。
3. HandlerMapping找到对应的Handler之后，并不是返回一个Handler原始对象，而是一个Handler执行链，在这个执行链中包括了拦截器和处理请求的Handler。HandlerMapping返回一个执行链给DispatcherServlet。
4. DispatcherServlet接收到执行链之后，会调用Handler适配器去执行Handler。
5. Handler适配器执行完成Handler（也就是我们写的Controller）之后会得到一个ModelAndView，并返回给DispatcherServlet。
6. DispatcherServlet接收到Handler适配器返回的ModelAndView之后，会根据其中的视图名调用视图解析器。
7. 视图解析器根据逻辑视图名解析成一个真正的View视图，并返回给DispatcherServlet。
8. DispatcherServlet接收到视图之后，会根据上面的ModelAndView中的model来进行视图中数据的填充，也就是所谓的视图渲染。
9. 渲染完成之后，DispatcherServlet就可以将结果返回给用户了。