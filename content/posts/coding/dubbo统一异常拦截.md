---
title: "Dubbo统一异常拦截"
date: 2017-10-09T10:02:55+08:00
draft: false
tags: [Dubbo]
slug: dubbo-exception-filter
---

# 问题
最近换了岗位，工作内容改成跟着架构师完善公司的技术架构。第一个任务就是做一个统一的异常拦截并返回相关错误码的方案。

公司采用dubbo做分布式框架，按dubbo的推荐，异常已经直接抛给调用方（consumer），只是公司现有的约定是使用错误码。问了几个前辈后才知道，原来dubbo在处理自定义异常的时候有些限制。

```java
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {
        try {
            Result result = invoker.invoke(invocation);
            if (result.hasException() && GenericService.class != invoker.getInterface()) {
                try {
                    Throwable exception = result.getException();

                    // 如果是checked异常，直接抛出
                    if (! (exception instanceof RuntimeException) && (exception instanceof Exception)) {
                        return result;
                    }
                    // 在方法签名上有声明，直接抛出
                    try {
                        Method method = invoker.getInterface().getMethod(invocation.getMethodName(), invocation.getParameterTypes());
                        Class<?>[] exceptionClassses = method.getExceptionTypes();
                        for (Class<?> exceptionClass : exceptionClassses) {
                            if (exception.getClass().equals(exceptionClass)) {
                                return result;
                            }
                        }
                    } catch (NoSuchMethodException e) {
                        return result;
                    }

                    // 未在方法签名上定义的异常，在服务器端打印ERROR日志
                    logger.error("Got unchecked and undeclared exception which called by " + RpcContext.getContext().getRemoteHost()
                            + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()
                            + ", exception: " + exception.getClass().getName() + ": " + exception.getMessage(), exception);

                    // 异常类和接口类在同一jar包里，直接抛出
                    String serviceFile = ReflectUtils.getCodeBase(invoker.getInterface());
                    String exceptionFile = ReflectUtils.getCodeBase(exception.getClass());
                    if (serviceFile == null || exceptionFile == null || serviceFile.equals(exceptionFile)){
                        return result;
                    }
                    // 是JDK自带的异常，直接抛出
                    String className = exception.getClass().getName();
                    if (className.startsWith("java.") || className.startsWith("javax.")) {
                        return result;
                    }
                    // 是Dubbo本身的异常，直接抛出
                    if (exception instanceof RpcException) {
                        return result;
                    }

                    // 否则，包装成RuntimeException抛给客户端
                    return new RpcResult(new RuntimeException(StringUtils.toString(exception)));
                } catch (Throwable e) {
                    logger.warn("Fail to ExceptionFilter when called by " + RpcContext.getContext().getRemoteHost()
                            + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()
                            + ", exception: " + e.getClass().getName() + ": " + e.getMessage(), e);
                    return result;
                }
            }
            return result;
        } catch (RuntimeException e) {
            logger.error("Got unchecked and undeclared exception which called by " + RpcContext.getContext().getRemoteHost()
                    + ". service: " + invoker.getInterface().getName() + ", method: " + invocation.getMethodName()
                    + ", exception: " + e.getClass().getName() + ": " + e.getMessage(), e);
            throw e;
        }
    }
```

这是dubbo的 `ExceptionFilter` 源码，是默认的异常处理。从代码中可以看到，异常对象需要满足一定的条件才会原样返回给前端。条件是：

1. 是checked异常
2. 或者方法签名上有申明的
3. 或者异常类和接口类在同一个jar包里
4. 或者JDK自带的异常
5. 或者Dubbo本身的异常
6. 或者继承了 `GenericService`

不是上述条件之一的话，就包装一层在返回。这样，对于自定义个的异常，上层就可能无法正确拿到。但是如果按dubbo的要求来，有出现一些问题，如果所有方法都声明异常或是抛出checked异常，肯定不符合实际要求，而且一旦出现未声明或不是checked的，上层就可能不知所措。如果异常类和接口类在同一个jar包，那每个工程最后打包都要有自己的异常类，这样就无法做到统一。对于继承 `GenericService` 类，这就让不用dubbo的工程也硬性依赖dubbo的内容。

架构师也给了一些参考，如通过Spring AOP，在所有Service接口打点。这样就在Dubbo前处理了异常。这种方法比较普遍，估计了解Spring AOP的都会实现。同样，这个配置也要求开发中在自己的工程中遵守一定的约定，并开启AOP和配置打点位置。过多的自主配置就会带来项目间的不一致性，所以还是先考虑其他方案。

# 方案

既然使用了Dubbo，我想这自然可以使用一些Dubbo的特性。翻了网上一些资料，最后决定通过自定义filter的方式来处理。参考Dubbo自己的 `ExceptionFilter`，我写了个大致的测试代码：

```java
public class TestExceptionFilter implements Filter {

    @Override
    public Result invoke(Invoker<?> invoker, Invocation invocation) throws RpcException {

        try {
            Result result = invoker.invoke(invocation);
            if (result.hasException()) {
                return new RpcResult("1");
            }
            return result;
        } catch (Exception e) {
            return new RpcResult("1");
        }
    }
}
```

这个代码这是拿来测试可行性的。实际证明这个方法可行。Consumer 那头拿到的就是 `1`。

网上还有说通过 `-exception` 方式去掉dubbo的默认异常处理。对于直接抛以上到上层的来说，确定有这个必要，那样的话我们自定义个的filter的就要写得很健壮，处理满足自己的需求外，还要包含dubbo默认异常处理的逻辑。不过我这边是返回错误码的，所以并不需要这么做。

# 关于执行顺序

虽然，我这种方式，对Filter的执行顺序没有啥要求，不过还是最好了解下，避免出现一些莫名其妙的问题。dubbo的Filter有自己的执行顺序，对于自定义的Filter的，不定义其顺序的话，一般都会在默认的Filter之后执行。dubbo的Filter使用了装饰模式，一层调一层，所以在 `Result result = invoker.invoke(invocation);` 这行代码前是正序执行，而这行代码后是逆序执行。如果我们喜欢自定义的Filter在Dubbo的默认处理之前执行，我们可以通过一下配置来实现。

```xml
<dubbo:provider filter="myException,default" />
```

不过异常处理其实是在拿到result之后，实际我们要的顺序是默认在前，自定义在后。这和dubbo的默认顺序一致，也就没啥好改的了。

# 总结

Dubbo 提供了很灵活的自定义方式。我们应该善勇这种特性，帮助我们简化工作。
