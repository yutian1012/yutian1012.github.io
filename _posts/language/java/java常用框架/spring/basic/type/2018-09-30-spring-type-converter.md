---
title: spring类型转换
tags: [spring]
---

TypeConverter是对外提供服务的接口，其内部主要由两部分组成，分别是类型转换服务和类型转换器。

注：TypeConverter类型转换时依赖PropertyEditor来实现的。

1）TypeConverter接口定义

TypeConverter提供了类型转换的接口，提供了字段、方法参数的转换接口。spring默认使用SimpleTypeConverter实现。

```
<T> T convertIfNecessary(Object value, Class<T> requiredType) throws TypeMismatchException;

<T> T convertIfNecessary(Object value, Class<T> requiredType, MethodParameter methodParam)
        throws TypeMismatchException;

<T> T convertIfNecessary(Object value, Class<T> requiredType, Field field)
        throws TypeMismatchException;
```

注：将给定对象值转换成requiredType类型，针对string类型的对象，将使用PropertyEditor类中的setAsText()方法转换String类型到任何类型。

2）SimpleTypeConverter类

查看类的继承体系

```
public class SimpleTypeConverter extends TypeConverterSupport

public abstract class TypeConverterSupport extends PropertyEditorRegistrySupport implements TypeConverter {
```

注：SimpleTypeConverter继承了TypeConverterSupport（抽象类），从而间接继承了PropertyEditorRegistrySupport。

3）PropertyEditorRegistrySupport类

PropertyEditorRegistrySupport中变量defaultEditors（Map<Class<?>, PropertyEditor>）内置了多种类型的PropertyEditor转换器。

```
private void createDefaultEditors() {
    this.defaultEditors = new HashMap<Class<?>, PropertyEditor>(64);

    // Simple editors, without parameterization capabilities.
    // The JDK does not contain a default editor for any of these target types.
    this.defaultEditors.put(Charset.class, new CharsetEditor());
    this.defaultEditors.put(Class.class, new ClassEditor());
    this.defaultEditors.put(Class[].class, new ClassArrayEditor());
    this.defaultEditors.put(Currency.class, new CurrencyEditor());
    ...
    if (zoneIdClass != null) {
        this.defaultEditors.put(zoneIdClass, new ZoneIdEditor());
    }

    // Default instances of collection editors.集合类型的处理器
    // Can be overridden by registering custom instances of those as custom editors.
    this.defaultEditors.put(Collection.class, new CustomCollectionEditor(Collection.class));
    ...

    // Default editors for primitive arrays.原始类型数组的处理器
    this.defaultEditors.put(byte[].class, new ByteArrayPropertyEditor());
    this.defaultEditors.put(char[].class, new CharArrayPropertyEditor());
    ...
    
    //原始类型及其包装类型的处理器
    this.defaultEditors.put(byte.class, new CustomNumberEditor(Byte.class, false));
    this.defaultEditors.put(Byte.class, new CustomNumberEditor(Byte.class, true));
    this.defaultEditors.put(short.class, new CustomNumberEditor(Short.class, false));
    this.defaultEditors.put(Short.class, new CustomNumberEditor(Short.class, true));
    ...

    // Only register config value editors if explicitly requested.
    // 替换相应的数组的处理器
    if (this.configValueEditorsActive) {
        StringArrayPropertyEditor sae = new StringArrayPropertyEditor();
        this.defaultEditors.put(String[].class, sae);
        this.defaultEditors.put(short[].class, sae);
        this.defaultEditors.put(int[].class, sae);
        this.defaultEditors.put(long[].class, sae);
    }
}
```