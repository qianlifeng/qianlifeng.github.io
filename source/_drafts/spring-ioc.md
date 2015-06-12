title: Spring IoC剖析
category: Java
date: 2015-06-12 11:12:17
---

<!-- more -->

Spring中接触的最多的就是关于Bean的配置与使用。在Spring中，所有由Spring IoC容器管理的**对象**都称为bean。所以，这个bean不是一种关系或者其他什么东西，就是一个Java对象。

#Bean的几种实例化方式
###构造函数
这种写法会直接调用ExampleBeanTwo的构造函数进行实例化，基本跟new操作差不多
```xml
<bean name="anotherExample" class="examples.ExampleBeanTwo"/>
```

###静态方法
这种写法用于使用指定的静态方法创建的实例
```xml
<bean id="clientService" class="examples.ClientService" factory-method="createInstance"/>
```

```java
public class ClientService {
    private static ClientService clientService = new ClientService();
    private ClientService() {}

    public static ClientService createInstance() {
        return clientService;
    }
}
```
