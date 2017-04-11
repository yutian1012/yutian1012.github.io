---
title: webservice入门
tags: [webservice]
---

### 1. 什么是webservice

WebService是一种跨编程语言和跨操作系统平台的远程调用技术。

从表面上看，WebService就是一个应用程序向外界暴露出一个能通过Web进行调用的API，也就是说能用编程的方法通过 Web来调用这个应用程序。我们把调用这个WebService的应用程序叫做客户端，而把提供这个WebService的应用程序叫做服务端。从深层次 看，WebService是建立可互操作的分布式应用程序的新平台，是一个平台，是一套标准。它定义了应用程序如何在Web上实现互操作性，你可以用任何你喜欢的语言，在任何你喜欢的平台上写Web service，只要我们可以通过Web service标准对这些服务进行查询和访问。

### 2. webservice技术

XML+XSD,SOAP和WSDL就是构成WebService平台的三大技术

1）XML和XSD

XML是WebService平台中表示数据的格式。除了易于建立和易于分析外，XML主要的优点在于它既是平台无关的，又是厂商无关 的。无关性是比技术优越性更重要的。

Schema(XSD)定义了一套标准的数据类型，并给出了一种语言来扩展这套数据类型。WebService平台就是用XSD来作为其数据类型系统的。为xml提供数据类型标准。

2）SOAP

Simple Object Access Protocol(简单对象访问协议)

SOAP协议 = HTTP协议 + XML数据格式

WebService通过HTTP协议发送请求和接收结果时，发送的请求内容和结果内容都采用XML格式封装，并增加了一些特定的HTTP消息头，以说明HTTP消息的内容格式，这些特定的HTTP消息头和XML内容格式就是SOAP协议。

SOAP协议是基于HTTP协议的，SOAP也是基于XML和XSD的，XML是SOAP的数据编码方式。打个比 喻：HTTP就是普通公路，XML就是中间的绿色隔离带和两边的防护栏，SOAP就是普通公路经过加隔离带和防护栏改造过的高速公路。

3）WSDL

WebService Description Language – Web服务描述语言。

WebService客户端要调用一个WebService服务，首先要有知道这个服务的地址在哪，以及这个服务里有什么方法可以调用，所以，WebService务器端首先要通过一个WSDL文件来说明自己有啥服务可以对外调用，服务是什么（服务中有哪些方法，方法接受 的参数是什么，返回值是什么），服务的网络地址用哪个url地址表示，服务通过什么方式来调用。

WSDL(Web Services Description Language)是一个基于XML的语言，用于描述Web Service及其函数、参数和返回值。它是WebService客户端和服务器端都能理解的标准格式。

### 3. WebService开发

WebService开发可以分为服务器端开发和客户端开发两个方面

1）服务端开发

把系统的业务方法发布成WebService服务，供远程合作单位和个人调用。(借助一些WebService框架可以很轻松地把自己的业务对象发布成WebService服务，Java方面的典型WebService框架包括：axis，xfire，cxf等，JavaEE服务器通常也支持发布WebService服务，例如JBoss)。

2）客户端开发

调用别人发布的WebService服务，大多数人从事的开发都属于这个方面，例如，调用天气预报WebService服务。（使用厂 商的WSDL2Java之类的工具生成静态调用的代理类代码；使用厂商提供的客户端编程API类；使用SUN公司早期标准的jax-rpc开发包；使用SUN公司最新标准的jax-ws开发包）。

3）webservice调用原理

对客户端而言，为各类WebService客户端API传递wsdl文件的url地址，这些API就会创建出底层的代理类，调用这些代理，就可以访问到webservice服务。代理类把客户端的方法调用变成soap格式的请求数据再通过HTTP协议发出去，并把接收到的soap 数据变成返回值返回。

对服务端而言，各类WebService框架的本质就是一个Servlet，当远程调用客户端给它通过http协议发送过来soap格式的请求数据时，它分析这个数据，就知道要调用哪个java类的哪个方法，于是去查找或创建这个对象，并调用其方法，再把方法返回的结果包装成soap格式的数据，通过http响应消息回给客户端。

### 4. 适用场景

1）跨防火墙通信

如果应用程序有成千上万的用户，而且分布在世界各地，那么客户端和服务器之间的通信将是一个棘手的问题。传统的做法是，选择用浏览器作为客户端，写下一大堆jsp页面，通过web提供服务。这样做的结果是需要开发和维护jsp页面。将应用换成WebService，就可以通过soap更简单的实现通信。

2）应用程序集成

企业级的应用程序开发者都知道，企业里经常都要把用不同语言写成的、在不同平台上运行的各种程序集成起来，而这种集成将花费很大的开发力量。应用程序经常 需要从运行在IBM主机上的程序中获取数据；或者把数据发送到主机或UNIX应用程序中去。即使在同一个平台上，不同软件厂商生产的各种软件也常常需要集成起来。通过WebService，可以很容易的集成不同结构的应用程序。

3）B2B集成（business to business）

跨公司的商务交易集成通常叫做B2B集成。WebService是B2B集成成功的关键。通过WebService，公司可以把关键的商务应用"暴露"给指定的供应商和客户,而不管他们的系统在什么平台上运行，使用什么开发语言。这样就大大减少了花在B2B集成上的时间和成本。

例如，把电子下单系统和电子发票系统"暴露"出来，客户就可以以电子的方式发送订单，供应商则可以以电子的方式发送原料采购发票。

4）软件和数据重用

软件重用是一个很大的主题，重用的形式很多，重用的程度有大有小。最基本的形式是源代码模块或者类一级的重用，这种形式是二进制形式的组件重用。采用 WebService应用程序可以用标准的方法把功能和数据"暴露"出来，供其它应用程序使用，达到业务级重用。

### 5. 不适用场景

1）单机应用程序

运行在同一台服务器上的服务器软件也是这样。最好直接用COM或其它本地的API来进行应用程序间的调用。

2）局域网的同构应用程序

虽然这种情况也可以使用webservice来实现通信，不过最好还是直接通过TCP进行RPC调用，那样会有效得多。

### 6. 相关概念

1）WSDL(web service definition language)

WSDL是webservice定义语言，对应.wsdl文档。一个webservice会对应一个唯一的wsdl文档，定义了客户端与服务端发送请求和响应的数据格式和过程。

2）SOAP(simple object  access protocal)

SOAP是"简单对象访问协议"，是一种简单的、基于HTTP和XML的协议，用于在WEB上交换结构化的数据。

soap消息：请求消息和响应消息

3）SEI(WebService EndPoint Interface)

SEI是web service的终端接口，就是WebService服务器端用来处理请求的接口。

4）CXF(Celtix + XFire)

一个apache的用于开发webservice服务器端和客户端的框架。

参考：http://blog.csdn.net/h_o_w_e/article/details/50848573