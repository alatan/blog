---  
title: "volatile详解"  
date: 2018-04-04
weight: 70  
draft: false  
keywords: [""]  
description: "volatile详解"  
tags: ["Java并发编程"]
categories: ["Java并发编程"]  
author: "默哥"  

lightgallery: true

toc:
  auto: false
---

## volatile简介
volatile可以保证多线程场景下变量的可见性和有序性。如果某变量用volatile修饰，则可以确保所有线程看到变量的值是一致的。
* 可见性：保证此变量的修改对所有线程的可见性。
* 有序性：禁止指令重排序优化，编译器和处理器在进行指令优化时，不能把在volatile变量操作(读/写)后面的语句放到其前面执行，也不能将volatile变量操作前面的语句放在其后执行。遵循了JMM的happens-before规则
## volatile实现原理剖析
volatile实现内存可见性原理：内存屏障（Memory Barrier）
内存屏障（Memory Barrier）是一种CPU指令，用于控制特定条件下的重排序和内存可见性问题。Java编译器也会根据内存屏障的规则禁止重排序
* 写操作时，通过在写操作指令后加入一条store屏障指令，让本地内存中变量的值能够刷新到主内存中
* 读操作时，通过在读操作前加入一条load屏障指令，及时读取到变量在主内存的值

## volatile缺陷：原子性Bug
原子性的问题：虽然volatile可以保证可见性，但是不能满足原子性
## volatile适合使用场景
变量真正独立于其他变量和自己以前的值，在单独使用的时候，适合用volatile
* 对变量的写入操作不依赖其当前值：例如++和--运算符的场景则不行
* 该变量没有包含在具有其他变量的不变式中

## synchronized和volatile比较
* volatile不需要加锁，比synchronized更轻便，不会阻塞线程
* synchronized既能保证可见性，又能保证原子性，而volatile只能保证可见性，无法保证原子性
* 与synchronized相比volatile是一种非常简单的同步机制