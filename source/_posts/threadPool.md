---
title: ThreadPoolExecutor源码解析
date: 2021-04-07 10:58:00
author: 陈城
top: true
cover: true
coverImg: /medias/banner/1-1.jpg
toc: false
mathjax: false
summary: 本文从线程池的核心参数入手，以线程创建，任务获取，任务结束，超时回收，线程退出为主线，以线程池的状态改变为节点,深入理解线程的内部工作流程，以及变型线程池的应用方式和场景
categories: Java
tags:
    - 源码
    - JUC
---

# ThreadPoolExecutor源码解析

