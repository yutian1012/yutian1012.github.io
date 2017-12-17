---
title: Netty protobuf序列化过程分析
tags: [architecture]
---

protobuf类如何将int序列化的

```
public void writeRawVarint32(int value) throws IOException{
    while(true){
        //判断数值是否在前7位中，如果在，则value使用一个字节就能够表示
        if((value & ~0x7F)==0){
            writeRawByte(value);
            return;
        }else{
            //获取低7位值，或上0x80，即将最高位置成1
            writeRawByte((value & 0x7F) | 0x80 );
            //将value向右移动
            value >>>= 7;
        }
    }
}
```

注：protobuf并不是把一个字节的8位都当成数据位，而是将最高位当成标识当前变量值是否还有数据位。