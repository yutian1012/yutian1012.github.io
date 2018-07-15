---
title: 应用绑定服务器的mac地址
tags: [auth]
---

### 获取本地的mac地址

```
import java.net.InetAddress;
import java.net.NetworkInterface;

public class MacAddressUtil {
    public static String[] getMacAddress()throws Exception{
        String[] addr=new String[2];
        InetAddress address=InetAddress.getLocalHost();
        //主机名
        addr[0]=address.getHostName();
        
        byte[] mac=NetworkInterface.getByInetAddress(address).getHardwareAddress();
        //输出字符串
        StringBuffer sb = new StringBuffer("");

        for(int i=0; i<mac.length; i++) {
            //字节转换为整数
            int temp = mac[i] & 0xff;
            String str = Integer.toHexString(temp);
            if(str.length()==1) {//格式化显示的位数
                sb.append("0"+str);
            }else {
                sb.append(str);
            }
            sb.append("-");
        }
        addr[1]=sb.substring(0,sb.length()-1).toUpperCase();
        return addr;
    }
    /**
     * 获取服务器端mac地址和计算机名
     * 使用下划线分隔两个值
     * @return
     */
    public static String getMachineCode(){
        String code="";
        try {
            String[] temp=getMacAddress();
            if(null!=temp[0]&&null!=temp[1]){
                code=temp[0]+"_"+temp[1];
            }
        } catch (Exception e) {
        }
        return code;
    }
}
```

### 获取本地所有的mac地址

```
import java.net.InetAddress;
import java.net.NetworkInterface;
import java.net.UnknownHostException;
import java.util.ArrayList;
import java.util.Enumeration;
import java.util.List;

public class Mac {  
    public static void main(String[] args) {  
        System.out.println(getHostName());
        System.out.println(getAllMac());
    }  
    /**
     * 获取所有的mac地址（计算机中有多张网卡就能有多个mac地址）
     * @return
     */
    public static List<String> getAllMac(){
        List<String> macList=new ArrayList<>();
        try {  
            Enumeration<NetworkInterface> enumeration = NetworkInterface.getNetworkInterfaces();  
            while (enumeration.hasMoreElements()) {  
                StringBuffer stringBuffer = new StringBuffer();  
                NetworkInterface networkInterface = enumeration.nextElement();  
                
                if (networkInterface != null) {
                    if(!networkInterface.isUp()&&networkInterface.isPointToPoint()){
                        continue;
                    }
                    byte[] bytes = networkInterface.getHardwareAddress();  
                    if (bytes != null) {  
                        for (int i = 0; i < bytes.length; i++) {  
                            if (i != 0) {  
                                stringBuffer.append("-");  
                            }  
                            int tmp = bytes[i] & 0xff; // 字节转换为整数  
                            String str = Integer.toHexString(tmp);  
                            if (str.length() == 1) {  
                                stringBuffer.append("0" + str);  
                            } else {  
                                stringBuffer.append(str);  
                            }  
                        }  
                        String mac=stringBuffer.toString().toUpperCase();
                        if(null!=mac&&!"".equals(mac)){
                            macList.add(mac);
                        }
                    }  
                }  
            }  
        } catch (Exception e) {  
            e.printStackTrace();  
        }  
        return macList;
    }
    /**
     * 获取计算机主机名
     * @return
     */
    public static String getHostName(){
        InetAddress address;
        try {
            address = InetAddress.getLocalHost();
            if(null!=address){
                return address.getHostName(); 
            }
        } catch (UnknownHostException e) {
            e.printStackTrace();
        }
        return null;
    }
}  
```