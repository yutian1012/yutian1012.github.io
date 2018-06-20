---
title: CXF拦截IP地址
tags: [webservice]
---

webservice是soap协议，简单对象访问协议是交换数据的一种协议规范，是一种轻量的、简单的、基于XML。

注：可理解为基于http和xml创建的数据交换协议，使用xml传输数据的载体，使用http作为传输的网络协议。

1）创建拦截器类

该拦截器是需要配置在webservice服务提供端的，用来过滤掉客户端访问的ip地址。

```
package org.ipph.web.webservice.cxf.security;

import java.net.InetAddress;
import java.net.UnknownHostException;
import java.util.ArrayList;
import java.util.List;

import javax.servlet.http.HttpServletRequest;

import org.apache.cxf.interceptor.Fault;
import org.apache.cxf.message.Message;
import org.apache.cxf.phase.AbstractPhaseInterceptor;
import org.apache.cxf.phase.Phase;
import org.apache.cxf.transport.http.AbstractHTTPDestination;

public class AddressInInterceptor extends AbstractPhaseInterceptor<Message>{
    //指定拦截阶段
    public AddressInInterceptor() {
        super(Phase.RECEIVE);
    }

    @Override
    public void handleMessage(Message message) throws Fault {
        HttpServletRequest request = (HttpServletRequest) message.get(AbstractHTTPDestination.HTTP_REQUEST);

        String ipAddr=request.getRemoteAddr();
       
        if(!accept(ipAddr)) 
            throw new Fault(new IllegalAccessException("ip地址" + ipAddr + " 拒绝访问"));
    }
    /**
     * 判断此IP地址是否为可以接收到IP地址
     * @param ipAddr
     * @return
     */
    protected boolean accept(String ipAddr){        
        try {
            long requestIP = conveIp2Long(ipAddr);
            List<AcceptIp> list = new ArrayList<AcceptIp>();//从配置文件或数据库中加载数据
            if (null!=list&&list.size()>0) {
                for (AcceptIp b : list) {
                    long start = conveIp2Long(b.getStartIp());
                    long end = conveIp2Long(b.getEndIp());
                    if (requestIP >= start && requestIP <= end)
                        return true;
                }
            }
        } catch (UnknownHostException e) {
            e.printStackTrace();
            return false;
        }
        return false;
    }
    
    /**
     * 转化IP地址为long类型。
     * @param ipAddr    IP地址。
     * @return IP地址的long表现形式。
     * @throws UnknownHostException 当IP地址格式错误是会抛出此异常。
     */
    public static long conveIp2Long(String ipAddr) throws UnknownHostException{
        //从IP串转成IP地址对象
        InetAddress ia=InetAddress.getByName(ipAddr);
        //从IP地址对象获取到IP的四个字节
        byte[] b=ia.getAddress();
        //每字节分加转位并转化为long然后相加
        long iaddr=
                (b[0]<<24)  &  0x00000000ffffffffL 
                |(b[1]<<16) &  0x00000000ffffffffL 
                |(b[2]<<8)  &  0x00000000ffffffffL 
                |(b[3]<<0)  &  0x00000000ffffffffL;  
        return iaddr;
    }
}
```

注：如果是拦截器获取的IP地址值不在允许的IP范围内，则抛出Fault异常。

2）创建AcceptIp实体类

该类对应页面设置允许IP访问的实体类，用户设置完成后保存到数据库中，供拦截器加载

```
public class AcceptIp {
    protected Long acceptId;//主键
    // 标题
    protected String title;
    // 开始地址
    protected String startIp;
    // 结束地址
    protected String endIp; 
    // 备注
    protected String remark;
    public Long getAcceptId() {
        return acceptId;
    }
    public void setAcceptId(Long acceptId) {
        this.acceptId = acceptId;
    }
    public String getTitle() {
        return title;
    }
    public void setTitle(String title) {
        this.title = title;
    }
    public String getStartIp() {
        return startIp;
    }
    public void setStartIp(String startIp) {
        this.startIp = startIp;
    }
    public String getEndIp() {
        return endIp;
    }
    public void setEndIp(String endIp) {
        this.endIp = endIp;
    }
    public String getRemark() {
        return remark;
    }
    public void setRemark(String remark) {
        this.remark = remark;
    }
}
```
