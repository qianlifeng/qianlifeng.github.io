title: Spring IoC剖析
category: Java
date: 2015-06-12 11:12:17
---

自从用了Spring之后，除了一些数据模型类，基本上已经很少自己new对象了。Spring接管了这些对象的创建工作。今天就来聊聊Spring的根基IOC。
<!-- more -->

先看两个祖先级的对象`BeanFactory`(since 2001)和`BeanDefinition`(since 2004)。`BeanFactory`是所有bean工厂的基类，而`BeanDefinition`是用于描述Bean的信息，实例化的时候需要用到。  

