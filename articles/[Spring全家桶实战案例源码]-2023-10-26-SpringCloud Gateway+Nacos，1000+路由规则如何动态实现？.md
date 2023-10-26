#  SpringCloud Gateway+Nacos，1000+路由规则如何动态实现？

原创 Springboot实战案例锦集 [ Spring全家桶实战案例源码 ](javascript:void\(0\);)

**Spring全家桶实战案例源码** ![]()

微信号 xgwcy007

功能介绍 spring, springboot, springcloud 案例开发详解

____

___发表于_

收录于合集

环境：SpringBoot2.7.12 + SpringCloud 2021.0.7 + Spring Cloud Alibaba 2021.0.4.0

* * *

 **1.  简介**

 **什么是Gateway？**

Spring Cloud Gateway是Spring Cloud生态系统中的一部分，是Spring Framework 5，Spring Boot
2和Project
Reactor等技术构建的网关服务器。它主要用于为微服务应用程序提供路由、负载均衡、安全性、限流、降级等功能，并具有高性能、高吞吐量和低延迟的优势。

Spring Cloud
Gateway的主要特点包括：基于异步非阻塞的Reactor框架实现的响应式编程模型，具有高性能、高吞吐量和低延迟的优势。它支持多种路由策略，包括基于路径、请求参数、请求头、主机等的路由。同时，Spring
Cloud Gateway还支持多种过滤器，包括预置的全局过滤器和自定义的局部过滤器，用于实现请求转发、请求修改、请求日志、请求验证、请求缓存等功能。

此外，Spring Cloud Gateway还支持集成Spring Cloud
Security，可以实现身份认证、授权和安全限制等功能。同时，它还支持动态路由配置和动态过滤器配置，可以在运行时动态添加、修改和删除路由和过滤器。

 **什么是Nacos？**  

Nacos
是阿里巴巴推出的一款开源的动态服务发现、配置管理和服务管理平台。它是为了帮助用户发现、配置和管理微服务而设计的，并且提供了简单易用的特性集，以帮助用户快速实现动态服务发现、服务配置、服务元数据及流量管理。

Nacos 致力于构建以“服务”为中心的现代应用架构，例如微服务范式和云原生范式。作为一个服务基础设施，Nacos
支持多种核心特性，包括服务发现、服务健康监测、动态配置服务、动态 DNS 服务以及服务及其元数据管理。

 **2\. 环境配置**

 **依赖管理**

  *   *   *   *   *   *   *   *   *   *   * 

    
    
     <properties>  <java.version>17</java.version>  <spring-cloud.version>2021.0.7</spring-cloud.version>  <spring-cloud-alibaba.version>2021.0.4.0</spring-cloud-alibaba.version></properties><dependencies>  <dependency>    <groupId>org.springframework.cloud</groupId>    <artifactId>spring-cloud-starter-gateway</artifactId>  </dependency></dependencies>

如果你不是基于服务发现机制，那么加入上面的starter-gateway依赖后，就可以在配置文件中配置静态路由了。

 **3.  静态路由**

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    spring:  cloud:    gateway:      metrics:        enabled: true      enabled: true      discovery:        locator:          enabled: false          lowerCaseServiceId: true      default-filters:      - StripPrefix=1       routes:      - id: rt-001        uri: http://www.baidu.com        predicates:        - Path=/api-x/**

浏览器输入：http://localhost:8188/api-x/ 可以直接跳转到baidu。

多数情况下，在项目中通过上面在appliation.yml文件中配置路由即可，但如果有新增修改这时服务必须重新启动才能生效。那有没有什么办法我们可以动态修改或者新增路由信息呢？有两种实现方式：

  1.  **通过actuator动态管理网关**

  2.  **结合Nacos动态配置 +  事件机制**

  

第一种方式其实比较简单，我们只需引入相应的依赖结合Endpoint就能实现。这里我们对这种方式不做介绍，有需要的可以参考官方文档。

本篇主要讲如何通关Nacos来实现路由的动态管理  

 **4.  动态路由**

首先还需要将Nacos相关的依赖引入。

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    <dependency>  <groupId>org.springframework.cloud</groupId>  <artifactId>spring-cloud-loadbalancer</artifactId></dependency><dependency>  <groupId>org.springframework.cloud</groupId>  <artifactId>spring-cloud-starter-circuitbreaker-reactor-resilience4j</artifactId></dependency><dependency>  <groupId>com.alibaba.cloud</groupId>  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId></dependency><dependency>  <groupId>com.alibaba.cloud</groupId>  <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId></dependency>

配置文件添加bootstrap.yml文件  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    spring:  application:    name: cloud-gateway---spring:  cloud:    nacos:      server-addr: localhost:8848      username: nacos      password: nacos      discovery:        enabled: true            group: cloudApp         config:        file-extension: yaml        group: cloudApp

上面配置了服务的注册和配置管理。

路由定义信息：  

  *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    {    // 路由ID    "id": "",    // 路由谓词    "predicates": [],    // 路由过滤器    "filters": [],    // 路由uri    "uri": "",    // 元数据信息    "metadata": {},    // 顺序    "order": 0}

接下来通过nacos控制台新建DataId= **shared-routes.json** 配置：

![]()

配置内容是json数组，这里我们就可以任意的配置路由信息。  

有个上面的配置信息，如何才能让上面的配置信息在gateway中生效呢？  

执行要在项目中添加如下Bean，该Bean专门用来监听上面的配置内容：  

  *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   *   * 

    
    
    @Componentpublic class DynamicRouteComponent {  @Resource  private InMemoryRouteDefinitionRepository routeDefinitionRepository ;  @Resource  private ApplicationEventMulticaster multicaster ;    @PostConstruct  public void init() throws Exception {    ConfigService cs = this.ncm.getConfigService() ;    cs.addListener("shared-routes.json", "CAPP", new Listener() {      @Override      public void receiveConfigInfo(String configInfo) {        routeDefinitionRepository.getRouteDefinitions().doOnNext(def -> {          routeDefinitionRepository.delete(Mono.just(def.getId())).subscribe() ;        }).subscribe() ;        ObjectMapper mapper = new ObjectMapper() ;        TypeReference<List<RouteDefinition>> ref = new TypeReference<>() {} ;        try {          List<RouteDefinition> routeDefinitions = mapper.readValue(configInfo, ref) ;          routeDefinitions.forEach(routeDefintion -> {            routeDefinitionRepository.save(Mono.just(routeDefintion)).subscribe() ;          }) ;          multicaster.multicastEvent(new RefreshRoutesEvent(routeDefinitions)) ;        }      }      @Override      public Executor getExecutor() {        return new ThreadPoolExecutor(1, 1, 60, TimeUnit.SECONDS, new LinkedBlockingQueue<>()) ;      }    });  }}

上面的发布事件其实可以没有，我们的服务在启动后nacos会每隔30s进行路由的定时刷新操作。不过自己添加上事件发布能够立即刷新路由，即刻生效。

完毕！！！  

 **求关注+转发**  

![]()

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

