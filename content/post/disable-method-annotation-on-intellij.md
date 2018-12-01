---
title: 禁用Intellij的注解自动折行
date: 2015-08-15 18:53:17
tags: [安全]
---

在 Intellij 中，自动格式化代码的时候，方法的注解默认会被格式化到单独的一行中，看着非常不爽，遂改之

原来默认格式化后的结果如下，个人以为非常难看

```Java
    @RequestMapping(value = "/test")
    public
    @ResponseBody
    String index() {
        return "qlf";
    }
```

修改之后的格式化效果：

```Java
    @RequestMapping(value = "/test")
    public @ResponseBody String index() {
        return "qlf";
    }
```

具体的修改位置：

**File** -> **Settings** -> **Editor** -> **Code Style** -> **Java** -> **Wrapping and Braces** -> **Method Annotations** 改为 **Do not warp** 即可。
