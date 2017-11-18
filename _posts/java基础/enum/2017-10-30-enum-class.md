---
title: 反射获取枚举类内容
tags: [basic]
---

要求：给定一个枚举类，通过该枚举类生成相应的select/checkbox控件内容。

如果要把功能做通用，就需要通过传递一个枚举的Class对象，获取相应的枚举内容。针对
所有枚举类对象，都应该获取select相应的value和name的属性，用来渲染控件

方式一：自定义标签

1）定义接口

```
public interface EnumOption {
    String getName();
    String getValue();
}
```

2）定义枚举类，并实现接口

```
/**
 * 组织等级枚举
 */
public enum OrgTypeEnum implements EnumOption{
    BLOC("集团"),COMPANY("公司"),DEPARTMENT("组织"),GROUP("小组"),OTHER("其他组织");
    
    private OrgTypeEnum(String name){
        this.name=name;
    }
    
    private String name;
    public String getName() {
        return name;
    }
    @Override
    public String getValue() {
        return null;
    }

    public void setName(String name) {
        this.name = name;
    }
    
    public static void main(String[] args) {
        Class c=OrgTypeEnum.class;
        Object[] arr=c.getEnumConstants();
        for(int i=0;i<arr.length;i++){
            if(arr[i] instanceof EnumOption){
                System.out.println(((EnumOption)arr[i]).getName());
            }
        }
    }
}
```

3）定义TagSupport标签

```
public class EnumTag extends TagSupport{
    private Class<?> enumClass;
    ...
}
```

方式二：在jsp中使用c:forEach遍历显示

```
<c:forEach items="<%=OrgTypeEnum.values()%>" var="orgType">
    <option value="${orgType.name}" 
        <c:if test="${orgType.name== sysOrgType.name}">selected </c:if>>
        ${orgType.name}
    </option>
</c:forEach>
```