---
title: "设计模式学习-生成器（Builder）"
date: 2013-07-30T15:56:00+08:00
draft: false
description: 将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示，java的StringBuilder就是一个Builder模式的例子。
tags: ["Design Patterns", 设计模式]
slug: design-patterns-builder
---

用来构建复杂的实例，java的StringBuilder就是一个Builder模式的例子。

### 意图：

将一个复杂对象的构建与它的表示分离，使得同样的构建过程可以创建不同的表示

### 结构：
![Builder](design-patterns-builder.jpg)

### 代码示例：

Builder类
```java
public abstract class Builder {  
    protected String product;  
      
    public Builder() {  
        product = new String("");  
    }  
  
    public void buildPart(String part) {  
        //默认空实现  
    }  
      
    public String getResult() {  
        return product;  
    }  
}
```

ConcreteBuilder类
```java
public class ConcreteBuilder extends Builder {  
  
    @Override  
    public void buildPart(String part) {  
        product += part;  
    }  
}
```

Director类
```java
public class Director {  
    private Builder builder;  
      
    public Director(Builder builder) {  
        this.builder = builder;  
    }  
      
    public void Construct() {  
        for (int i = 0; i < 10; i++) {  
            builder.buildPart("aaa");  
        }  
    }  
      
    public String getResult() {  
        return builder.getResult();  
    }  
}
```

调用
```java
public class Client {  
  
    public static void main(String[] args) {  
        Director dir = new Director(new ConcreteBuilder());  
        dir.Construct();  
        System.out.println(dir.getResult());  
    }  
}
```

### 优点：
<ul>
<li>使你可以改变一个产品的内部表示；</li>
<li>它将构造代码和表示代码分开；</li>
<li>使你可对构造过程进行更精细的控制；</li>
</ul>

### 适用：

当创建复杂对象的算法应该独立于该对象的组成部分以及它们的装配方式时。
当构造过程必须允许被构造的对象有不同的表示时。

### 相关模式：

Abstract Factory和Builder比较相似。主要的区别是Builder模式着重于一步步构造一个复杂对象。而 Abstract Factory着重于多个系列的产品对象（简单的或是复杂的）。Builder在最后的一步返回产品，而对于 Abstract Factory来说，产品是立即返回的。

Composite通常是用Builder生成的
