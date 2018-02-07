---
title: JAXB日期和数字格式化输出
tags: [xml]
---

参考：http://www.cnblogs.com/yjmyzz/p/xstream-jaxb-format-date-and-number.html

Jaxb是java中用于对象xml序列化/反序列化的经典开源项目，利用它们将对象转换成xml时，经常会遇到日期(Date)、数字按指定格式输出的需求。

注：jaxb处理要先创建一个Adapter，然后要处理类的属性成员Date字段的get方法上使用创建的Adapter。

### 1. 日期的格式化操作

1）新建Adapter类

```
import java.text.DateFormat;
import java.text.SimpleDateFormat;
import java.util.Date;

import javax.xml.bind.annotation.adapters.XmlAdapter;

public class JaxbDateAdapter extends XmlAdapter<String, Date> {
    static final String STANDARM_DATE_FORMAT = "yyyy-MM-dd";
    private DateFormat format = new SimpleDateFormat(STANDARM_DATE_FORMAT);
    
    @Override
    public Date unmarshal(String v) throws Exception {
        if (v == null) {
            return null;
        }
        return format.parse(v);
    }

    @Override
    public String marshal(Date v) throws Exception {
        return format.format(v);
    }
}
```

2）将Adapter应用于字段上

```
@XmlJavaTypeAdapter(JaxbDateAdapter.class)
public Date getBirthDay() {
    return birthDay;
}
```