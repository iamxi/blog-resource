---
title: "javabean字段文字拼接"
date: 2018-01-20T10:58:21+08:00
draft: false
tags: [Java]
slug: java-bean-field-assemble
---

# 问题
接了一个需求，从数据库拿出患者的记录，这些记录都是一些吸烟年份、饮酒年份、是否糖尿病等，不是数字就是布尔值，需要通过代码转化为普通人能理解的文字。

思前想后，总不能使用大量 `if-else` 判断来拼接字符串吧。可以采用模板，但是那样除了必要判空，代码和模板也将分离，不方便维护。而且有些字段存在依赖关系。

# 解决
刚好之前看了使用注解来替代枚举的方式。想着是否可以在每个需要拼接的字段上打上注解，通过判断注解的方式拼接。注解中也可以放入模板、依赖关系等配置。

```java
@Target({ElementType.FIELD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface StringAssemble {

    /**
     * 模板
     */
    String template();

    /**
     * 分类
     */
    String type() default "";

    /**
     * 上级依赖
     */
    String dependency() default "";

    /**
     * 相互依赖的内容之间的分割符号
     */
    String dependencySplit() default ",";
}
```

上面注解中，`templete` 主要是字段值需要填入的文字模板，`type` 用于不同的字段组合，`dependency` 主要是有依赖关系的字段之间的内容组合，最后一个是有依赖关系的字段字段拼接后用于隔离的标点。

实现代码如下：
```java
private static String stringAssemble(Object data, String type, String splitPunctuation) {
        final Map<String, String> infoMap = new HashMap<>();
        final Map<String, String> dependencyMap = new HashMap<>();
        try {
            Field[] fields = data.getClass().getDeclaredFields();
            for (Field field : fields) {
                if (field.isAnnotationPresent(StringAssemble.class)) {
                    StringAssemble stringAssemble = field.getAnnotation(StringAssemble.class);
                    if (type == null || stringAssemble.type().equals(type)) {
                        field.setAccessible(true);
                        Object o = field.get(data);
                        if (o != null) {
                            if (stringAssemble.dependency() != null && !"".equals(stringAssemble.dependency())) {
                                String info = stringAssemble.dependencySplit() + String.format(stringAssemble.template(), o.toString());
                                if (dependencyMap.containsKey(stringAssemble.dependency())) {
                                    info = dependencyMap.get(stringAssemble.dependency()) + info;
                                }
                                dependencyMap.put(stringAssemble.dependency(), info);
                            } else {
                                infoMap.put(field.getName(), String.format(stringAssemble.template(), o.toString()));
                            }
                        }
                    }
                }
            }

            dependencyMap.keySet().stream().filter(x -> infoMap.containsKey(x)).forEach(x -> infoMap.put(x, infoMap.get(x) + dependencyMap.get(x)));
        } catch (Exception e) {
            LOG.error("拼接字符串错误:Object:{}, e:{}", data, e);
            e.printStackTrace();
        }
```

这个代码实现按注解套用模板，处理一级依赖关系，自定义拼接使用的标点。需要待完善的字段组合时候没有顺序、无法处理两级或以上的依赖关系。

相比 `if-else`，这个代码能大大简化工作，只要这部分代码不出问题，能保证所有的拼接都正确，减少bug引入。当然这个仅能处理简单的字段转文字并拼接的需要。格式过于复制就无法使用。代码尚不完善，仅供参考。这个方案似乎也并完美，如果有更好的方案，欢迎留言。
