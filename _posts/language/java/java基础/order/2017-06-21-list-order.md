---
title: java集合排序与compareTo方法的思考
tags: [basic]
---

compareTo(T o)比较此对象与指定对象的顺序。如果该对象小于、等于或大于指定对象，则分别返回负整数、零或正整数。针对集合来讲，一般小于会放在集合前端。

```
public class OrderTest {
    public static void main(String[] args) {
        List<String> list=new ArrayList<String>();
        String hello="123";
        String hello2="124";
        list.add(hello2);
        list.add(hello);
        System.out.println(Arrays.toString(list.toArray()));
        //字符串的比较
        System.out.println(hello.compareTo(hello2));
        Collections.sort(list);
        //比较值小的会放在集合的前面
        System.out.println(Arrays.toString(list.toArray()));
    }
}
```

