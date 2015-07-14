title: Spring AOP小试
date: 2015-07-14 20:47:17
tags: [Java]
---

第一次使用Spring的AOP拦截，感觉还是很强大的嘛^Q^
<!-- more -->

直接来一个最简单的例子，首先实现一个同步方法的切面。要求是所有注解了`@Synchronized`的方法都必须同步执行（加锁）。  

首先声明注解：

```Java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

/**
 * 顺序执行方法，防止高并发下的执行顺序问题
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Synchronized {
}
```

接着声明切面

```Java
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

@Component
@Aspect
@Order(1)
public class SynchronizedAspect {

    public final static Object locker = new Object();

    @Around("@annotation(cn.fcgyl.oms.aspect.Synchronized)()")
    public Object doSynchronize(ProceedingJoinPoint pjp) throws Throwable {
        synchronized (locker) {
            Object retVal = pjp.proceed();
            return retVal;
        }
    }
}
```

1. 定义的切面必须使用`@Aspect`注解
2. 注册这个切面bean到Spring，我这里是使用`@Component`自动装配的
3. 定义拦截方法（本例中的`doSynchronize`），同时标注拦截类型。Spring AOP只支持拦截方法，所以这里的几种拦截类型大概是`@Before`,`@After`,`@Around`
4. 这里的`@Order`指定了该注解的优先级，越小的数字代表了越高的优先级

最后启用切面，在spring配置文件中配置如下信息：
```Xml
<!-- 开启AOP切面 -->
<aop:aspectj-autoproxy/>
```

大功告成。
