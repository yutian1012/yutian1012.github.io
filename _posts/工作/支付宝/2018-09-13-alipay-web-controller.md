---
title: 支付宝对接访问支付页面
tags: [alipay]
---

1）访问支付页面

下面配置的信息使用的是沙箱环境下的信息，可根据具体情况替换数据

```
@RequestMapping("alipay")
@Controller
public class AlipayController {
    
    public final static String URL="https://openapi.alipaydev.com/gateway.do";
    public final static String FORMAT="json";
    public final static String CHARSET="UTF-8";
    public final static String SIGN_TYPE="RSA2";
    public final static String APPID="xxx";
    public final static String APP_PRIVATE_KEY="xxx";
    public final static String ALIPAY_PUBLIC_KEY="xxx";
    
    @RequestMapping("/page")
    public void page(HttpServletRequest request,HttpServletResponse response) throws IOException {
        AlipayClient alipayClient = new DefaultAlipayClient(URL, APPID, APP_PRIVATE_KEY, FORMAT, CHARSET, ALIPAY_PUBLIC_KEY, SIGN_TYPE); //获得初始化的AlipayClient
        AlipayTradePagePayRequest alipayRequest = new AlipayTradePagePayRequest();//创建API对应的request
        alipayRequest.setReturnUrl("http://domain.com/CallBack/return_url.jsp");
        alipayRequest.setNotifyUrl("http://domain.com/CallBack/notify_url.jsp");//在公共参数中设置回跳和通知地址
        alipayRequest.setBizContent("{" +
            "    \"out_trade_no\":\"20150320010101001\"," +
            "    \"product_code\":\"FAST_INSTANT_TRADE_PAY\"," +
            "    \"total_amount\":88.88," +
            "    \"subject\":\"Iphone6 16G\"," +
            "    \"body\":\"Iphone6 16G\"," +
            "    \"passback_params\":\"merchantBizType%3d3C%26merchantBizNo%3d2016010101111\"," +
            "    \"extend_params\":{" +
            "    \"sys_service_provider_id\":\"2088511833207846\"" +
            "    }"+
            "  }");//填充业务参数
        String form="";
        try {
            form = alipayClient.pageExecute(alipayRequest).getBody(); //调用SDK生成表单
        } catch (AlipayApiException e) {
            e.printStackTrace();
        }
        response.setContentType("text/html;charset=" + CHARSET);
        response.getWriter().write(form);//直接将完整的表单html输出到页面
        response.getWriter().flush();
        response.getWriter().close();
    }
}
```