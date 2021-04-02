---
title: synchronized源码解析
date: 2021-03-22 09:25:00
author: 陈城
top: true
cover: true
coverImg: /medias/banner/6.jpg
toc: false
mathjax: false
summary: 锁无非就是不同线程对同一个共有资源的竞争占有，本文从jvm源码的角度来解析synchronized是如何在线程竞争的情况下操作锁对象的markWord。偏向锁，轻量锁，重量锁的加锁解锁及其升级过程，以及JDK1.8如何优化synchronized，synchronized的优缺点，和实现AQS框架锁的共同点和区别
categories: Java
tags:
    - Java
    - JUC
---

# synchronized源码解析

