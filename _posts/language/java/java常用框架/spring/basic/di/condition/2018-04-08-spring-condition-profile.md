---
title: spring中Profile借助了Condition
tags: [spring]
---

从Spring4开始，@Profile注解进行了重构，使其基于@Conditional和Condition实现。

1）查看@Profile注解定义

```
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE, ElementType.METHOD})
@Documented
@Conditional(ProfileCondition.class)
public @interface Profile {

    /**
     * The set of profiles for which the annotated component should be registered.
     */
    String[] value();

}
```

注：@Profile本身也使用了@Conditional注解，并且引用ProfileCondition作为Condition实现。

2）ProfileCondition

ProfileCondition实现了Condition接口，并且在做出决策的过程中，考虑到了ConditionContext和AnnotatedTypeMetadata中的多个因素。

```
class ProfileCondition implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        if (context.getEnvironment() != null) {
            MultiValueMap<String, Object> attrs = metadata.getAllAnnotationAttributes(Profile.class.getName());
            if (attrs != null) {
                for (Object value : attrs.get("value")) {
                    if (context.getEnvironment().acceptsProfiles(((String[]) value))) {
                        return true;
                    }
                }
                return false;
            }
        }
        return true;
    }

}
```

ProfileCondition通过AnnotatedTypeMetadata得到了用于@Profile注解的所有属性。它会明确地检查@Profile注解的value属性，该属性包含了bean的profile名称。然后，它根据通过ConditionContext得到的Environment来检查该profile是否处于激活状态。