---
title: 类泛型的应用--抽取公用代码
tags: [generics]
---

```
public interface IPageParser<T extends PageModel> {
    
    public boolean isValid();
    
    public T parse(Page page) throws Exception;
    
    public List<T> parseList(Page page) throws Exception;
}
```

创建具体实现类时，传递泛型类后，对于的方法返回值也会变成传递的泛型类对象。