## SpringCloud 微服务

### 1. SpringCloud 简介

SpringCloud是基于SpringBoot的一整套实现微服务的框架。它利用SpringBootde的开发便利性简化分布式系统的开发，它提供了微服务开发所需的配置管理、服务发现、断路器、智能路由、微代理、控制总线、全局锁、决策竞选、分布式会话和集群状态管理等组件。SpringCloud并不重复造轮子，而是将市面上开发得比较好的模块集成进去，进行封装，从而减少了各模块的开发成本。换句话说：Spring Cloud 提供了构建分布式系统所需的“全家桶”。

在SpringCloud框架中，主要使用六大组件来构建基础的微服务框架。六大组件分别是服务发现注册组件、配置中心组件、认证中心组件、网关路由组件、服务消费负载均衡组件和断路器组件。

#### 1.1 服务发现注册组件
  
  Netflix开发的服务发现框架Eureka，基于REST的服务，主要用于AWS云中定位服务，以实现中间层服务器的负载平衡和故障转移。
  
  Eureka提供服务注册和发现。Eureka包含服务端和客户端。
  
  服务应用配置Eureka客户端，启动后Eureka服务端会自动发现并注册服务。在应用启动后，将会向Eureka服务端发送心跳，默认周期为30秒，如果Eureka服务端在多个心跳周期内没有接收到某个应用的心跳，则认为服务宕机，注销该实例，Eureka服务端将会从服务注册表中把这个服务节点移除。
  
  Eureka客户端还有一个内置的负载均衡器，可以进行基本的循环负载均衡。
  
  Eureka服务端做集群部署时，服务端之间通过复制的方式完成数据的同步，同时还提供客户端缓存机制，即多有服务端的挂掉，客户端依然可以利用缓存中的信息消费其他服务。

  Netflix在2018年对Eureka 2.x版本开源不再进行维护，所以在第3节中介绍由阿里开源的Nacos，它实现与Eureka同样的功能。

  ![](https://raw.githubusercontent.com/Chenxicheng/Document/master/study/spring%20cloud%20%E5%BE%AE%E6%9C%8D%E5%8A%A1/img/Eureka.png?raw=true)

#### 1.2 配置中心组件
  在微服务中，由于服务数量，为方便多个服务的配置文件的统一管理，实时更新，所以需要配置中心组件来进行管理。
  
  配置中心组件的核心功能，提供服务端和客户端支持、集中管理各环境的配置文件、配置文件修改之后快速的生效、可以进行版本管理、支持大的并发查询、支持各种语言。

  Spring提供SpringCloudConfig组件实现以上核心功能。它包含了客户端和服务端。服务端提供配置文件的存储，集中管理配置文件，动态化的配置更新，以接口的形式将配置文件的内容提供出去；客户端服务应用通过接口获取自己配置信息，初始化应用，当配置发生更新是，服务应用不需要重启即可感知到配置的变化并应用新的配置中。

  SpringCloudConfig服务端存储配置文件分为Git、SVN和本地存储，默认为Git。

  在阿里Nacos中集成了配置中心组件，实现与SpringCloudConfig同样功能，但更易操作和实时更新。
  ![](https://github.com/Chenxicheng/Document/blob/master/study/spring%20cloud%20%E5%BE%AE%E6%9C%8D%E5%8A%A1/img/springcloudconfig.png?raw=true)

#### 1.3 认证中心组件
  认证中心组件提供身份管理认证服务，用于构建安全的应用程序和服务。

  Spring提供spring security oauth2安全认证框架。

  oauth2 认证授权流程：
  名词解释：
  - Resource Owner：资源所有者。即用户。
  - Client：客户端（第三方应用）。
  - Authorization server：授权（认证）服务器。即服务提供商专门用来处理认证的服务器。
  - Resource server：资源服务器，即服务提供商存放用户生成的资源的服务器。它与认证服务器，可以是同一台服务器，也可以是不同的服务器。
  - Access Token：访问令牌。使用合法的访问令牌获取受保护的资源。

  ![oauth2](https://github.com/Chenxicheng/Document/blob/master/study/spring%20cloud%20%E5%BE%AE%E6%9C%8D%E5%8A%A1/img/oauth2.png?raw=true)

  流程说明：
  - （A）客户端向资源所有者请求授权。授权请求可以直接对资源所有者(如图所示)进行，或者通过授权服务器作为中介进行间接访问（首选方案）。
  - （B）资源所有者允许授权，并返回凭证（如code）。
  - （C）客户端通过授权服务器进行身份验证，并提供授权凭证（如code），请求访问令牌（access token）。
  - （D）授权服务器对客户端进行身份验证，并验证授权凭证，如果有效，则发出访问令牌。
  - （E）客户端向资源服务器请求受保护的资源，并通过提供访问令牌来进行身份验证。
  - （F）资源服务器验证访问令牌，如果正确则返回受保护资源。

  oauth2根据使用场景不同，分成了4种模式

  - 授权码模式（authorization code）
  - 简化模式（implicit）- 
  - 密码模式（resource owner password credentials）
  - 客户端模式（client credentials）

  在微服务认证授权中通常使用密码模式和客户端模式。授权码模式使用到了回调地址，是最为复杂的方式，通常网站中经常出现的微博，qq第三方登录，都会采用这个形式。简化模式不常用。

#### 1.4 网关路由组件

  网关路由组件在微服务架构中提供一种简单有效的统一的 API 路由管理方式。常见的功能有路由转发、权限校验、限流控制和监控等作用。

  Spring提供Spring Cloud Gateway网关路由组件，它取代Zuul网关。其不仅提供统一的路由方式，并且基于 Filter 链的方式提供网关基本的功能。

  Spring Cloud Gateway 基本特征：
  - 动态路由
  - Predicates（使用它来匹配来自 HTTP 请求的任何内容，例如 headers 或参数） 和 Filters （使用它修改请求和响应） 作用于特定路由
  - 集成 Hystrix 断路器
  - 限流
  - 路径重写

  Spring Cloud Gateway 工作流程：
  - 客户端向网关发出请求。
  - 如果 Gateway Handler Mapping 中找到与请求相匹配的路由，将其发送到 Gateway Web Handler。
  - Handler 再通过指定的过滤器链来将请求路由发送到实际的服务执行业务逻辑，然后返回结果。 
  过滤器之间用虚线分开是因为过滤器可能会在发送代理请求之前（“pre”）或之后（“post”）执行业务逻辑。
  ![](https://github.com/Chenxicheng/Document/blob/master/study/spring%20cloud%20%E5%BE%AE%E6%9C%8D%E5%8A%A1/img/spring-cloud-gateway.png?raw=true)

#### 1.5 服务消费负载均衡组件
  服务消费负载均衡组件在微服务中帮助消费方服务自动请求消费提供方服务，并基于负载均衡算法，请求其中一个提供者服务。

  Netfilx提供Feign框架实现服务消费负载均衡。Feign是一个声明式的Web Service客户端，它的目的就是让Web Service调用更加简单。Feign提供HTTP请求的模板，通过编写简单的接口和插入注解，就可以定义好HTTP请求的参数、格式、地址等信息。

  Feign特性：
  - 可插拔的注解支持，包括Feign注解和JAX-RS注解;
  - 支持可插拔的HTTP编码器和解码器;
  - 支持Hystrix断路器和它的Fallback;
  - 支持Ribbon负载均衡;
  - 支持HTTP请求和响应的压缩。

  Feign整合Ribbon负载均衡器。Ribbon是Netflix发布的开源项目，主要功能是提供客户端的服务负载均衡算法，将中间层服务连接在一起。Ribbon客户端组件提供一系列完善的配置项如连接超时，重试等。简单的说，就是在配置文件中列出Load Balancer（简称LB）后面所有的机器，Ribbon会自动的帮助你基于某种规则（如简单轮询，随即连接等）去连接这些机器。Ribbon使服务调用端具备负载均衡能力，通过和服务注册组件的紧密结合，不用在服务集群内再架设负载均衡服务，很大程度上简化服务集群内的架构。
![](https://github.com/Chenxicheng/Document/blob/master/study/spring%20cloud%20%E5%BE%AE%E6%9C%8D%E5%8A%A1/img/rabooin.png?raw=true)

#### 1.6 断路器组件
  在微服务中，服务之间形成调用链路，链路中的任何一个服务提供者都可能面临着相应超时、宕机等不可用的情况，在高并发的情况下，这种情况会随着并发量的上升恶化，形成“雪崩效应”，而断路器正是用来解决这一个问题的组件。

  Netfilx提供的hystrix断路器组件。hystrix断路器具备服务降级、服务熔断、线程隔离、请求缓存、请求合并以及服务监控等强大功能。

  断路器的三个重要参数：快照时间窗、请求总数下限、错误百分比下限。这个参数的作用分别是：
  - 快照时间窗：断路器确定是否打开需要统计一些请求和错误数据，而统计的时间范围就是快照时间窗，默认为最近的10秒。
  - 请求总数下限：在快照时间窗内，必须满足请求总数下限才有资格根据熔断。默认为20，意味着在10秒内，如果该hystrix命令的调用此时不足20次，即时所有的请求都超时或其他原因失败，断路器都不会打开。
  - 错误百分比下限：当请求总数在快照时间窗内超过了下限，比如发生了30次调用，如果在这30次调用中，有16次发生了超时异常，也就是超过50%的错误百分比，在默认设定50%下限情况下，这时候就会将断路器打开。

  断路器实现基本原理：
  - 正常情况下，断路器关闭，服务消费者正常请求微服务
  - 连接请求超时，在断路器未打开之前，每个请求都会在当hystrix超时之后返回fallback，每个请求时间延迟就是近似hystrix的超时时间，如果设置为5秒，那么每个请求就都要延迟5秒才会返回
  - 当熔断器在10秒内发现请求总数超过20，并且错误百分比超过50%，这个时候熔断器打开
  - 打开之后，再有请求调用的时候，将不会调用主逻辑，而是直接调用降级逻辑，这个时候就不会等待5秒之后才返回fallback。通过断路器，实现了自动地发现错误并将降级逻辑切换为主逻辑，减少响应延迟的效果
  - 在断路器打开之后，处理逻辑并没有结束，降级逻辑已经被成了主逻辑，hystrix也实现自动恢复功能，实现主逻辑恢复
  - 当断路器打开，对主逻辑进行熔断之后，hystrix会启动一个休眠时间窗，在这个时间窗内，降级逻辑是临时的成为主逻辑，当休眠时间窗到期，断路器将进入半开状态，释放一次请求到原来的主逻辑上，如果此次请求正常返回，那么断路器将继续闭合，主逻辑恢复，如果这次请求依然有问题，断路器继续进入打开状态，休眠时间窗重新计时
  
  通过上面的一系列机制，hystrix的断路器实现对依赖资源故障的端口、对降级策略的自动切换以及对主逻辑的自动恢复机制。这使得微服务在依赖外部服务或资源的时候得到了非常好的保护，同时对于一些具备降级逻辑的业务需求可以实现自动化的切换与恢复，相比于设置开关由监控和运维来进行切换的传统实现方式显得更为智能和高效。
![](https://github.com/Chenxicheng/Document/blob/master/study/spring%20cloud%20%E5%BE%AE%E6%9C%8D%E5%8A%A1/img/hystrix-work-flow.png?raw=true)

### 2. [Nacos](https://nacos.io/zh-cn/docs/what-is-nacos.html) 

Nacos 实现发现、配置和管理微服务。它提供一组简单易用的特性集，快速实现动态服务发现、服务配置、服务元数据及流量管理。
它实现Eureka服务发现注册组件和SpringConfig配置中心组件的功能。它是构建以“服务”为中心的现代应用架构 的服务基础设施

#### 2.1 Nacos特性
- 服务发现和服务健康监测

  Nacos 支持基于 DNS 和基于 RPC 的服务发现。服务提供者使用 原生SDK、OpenAPI、或一个独立的Agent TODO注册 Service 后，服务消费者可以使用DNS TODO 或HTTP&API查找和发现服务。

  Nacos 提供对服务的实时的健康检查，阻止向不健康的主机或服务实例发送请求。Nacos 支持传输层 (PING 或 TCP)和应用层 (如 HTTP、MySQL、用户自定义）的健康检查。 对于复杂的云环境和网络拓扑环境中（如 VPC、边缘网络等）服务的健康检查，Nacos 提供了 agent 上报模式和服务端主动检测2种健康检查模式。Nacos 还提供了统一的健康检查仪表盘，帮助您根据健康状态管理服务的可用性及流量。

- 动态配置服务

  动态配置服务可以让您以中心化、外部化和动态化的方式管理所有环境的应用配置和服务配置。

  动态配置消除了配置变更时重新部署应用和服务的需要，让配置管理变得更加高效和敏捷。

  配置中心化管理让实现无状态服务变得更简单，让服务按需弹性扩展变得更容易。

  Nacos 提供了一个简洁易用的UI帮助您管理所有的服务和应用的配置。Nacos 还提供包括配置版本跟踪、金丝雀发布、一键回滚配置以及客户端配置更新状态跟踪在内的一系列开箱即用的配置管理特性，帮助您更安全地在生产环境中管理配置变更和降低配置变更带来的风险。

- 动态 DNS 服务

  动态 DNS 服务支持权重路由，让您更容易地实现中间层负载均衡、更灵活的路由策略、流量控制以及数据中心内网的简单DNS解析服务。动态DNS服务还能让您更容易地实现以 DNS 协议为基础的服务发现，以帮助您消除耦合到厂商私有服务发现 API 上的风险。

  Nacos 提供了一些简单的 DNS APIs TODO 帮助您管理服务的关联域名和可用的 IP:PORT 列表.

- 服务及其元数据管理

  Nacos 能让您从微服务平台建设的视角管理数据中心的所有服务及元数据，包括管理服务的描述、生命周期、服务的静态依赖分析、服务的健康状态、服务的流量管理、路由及安全策略、服务的 SLA 以及最首要的 metrics 统计数据。

#### 2.2 Nacos架构
![]()

- 服务 (Service)

  服务是指一个或一组软件功能（例如特定信息的检索或一组操作的执行），其目的是不同的客户端可以为不同的目的重用（例如通过跨进程的网络调用）。Nacos 支持主流的服务生态，如 Kubernetes Service、gRPC|Dubbo RPC Service 或者 Spring Cloud RESTful Service.

- 服务注册中心 (Service Registry)

  服务注册中心，它是服务，其实例及元数据的数据库。服务实例在启动时注册到服务注册表，并在关闭时注销。服务和路由器的客户端查询服务注册表以查找服务的可用实例。服务注册中心可能会调用服务实例的健康检查 API 来验证它是否能够处理请求。

- 服务元数据 (Service Metadata)

  服务元数据是指包括服务端点(endpoints)、服务标签、服务版本号、服务实例权重、路由规则、安全策略等描述服务的数据

- 服务提供方 (Service Provider)

  是指提供可复用和可调用服务的应用方

- 服务消费方 (Service Consumer)

  是指会发起对某个服务调用的应用方

- 配置 (Configuration)

  在系统开发过程中通常会将一些需要变更的参数、变量等从代码中分离出来独立管理，以独立的配置文件的形式存在。目的是让静态的系统工件或者交付物（如 WAR，JAR 包等）更好地和实际的物理运行环境进行适配。配置管理一般包含在系统部署的过程中，由系统管理员或者运维人员完成这个步骤。配置变更是调整系统运行时的行为的有效手段之一。

- 配置管理 (Configuration Management)

  在数据中心中，系统中所有配置的编辑、存储、分发、变更管理、历史版本管理、变更审计等所有与配置相关的活动统称为配置管理。

- 名字服务 (Naming Service)

  提供分布式系统中所有对象(Object)、实体(Entity)的“名字”到关联的元数据之间的映射管理服务，例如 ServiceName -> Endpoints Info, Distributed Lock Name -> Lock Owner/Status Info, DNS Domain Name -> IP List, 服务发现和 DNS 就是名字服务的2大场景。

- 配置服务 (Configuration Service)

  在服务或者应用运行过程中，提供动态配置或者元数据以及配置管理的服务提供者。

- 实例（Instance）
  提供一个或多个服务的具有可访问网络地址（IP:Port）的进程。

- 权重（Weight）
  实例级别的配置。权重为浮点数。权重越大，分配给该实例的流量越大。

- 健康检查（Health Check）
  以指定方式检查服务下挂载的实例 (Instance) 的健康度，从而确认该实例 (Instance) 是否能提供服务。根据检查结果，实例 (Instance) 会被判断为健康或不健康。对服务发起解析请求时，不健康的实例 (Instance) 不会返回给客户端。

#### 2.3 Nacos安装与UI界面使用
- 下载Nacos压缩包
  从 [最新稳定版本](https://github.com/alibaba/nacos/releases) 下载 nacos-server-$version.zip 包，并解压。
  在 bin 目录下启动或关闭Nacos服务。
- 启动服务器
  Linux/Unix/Mac
  启动命令(standalone代表着单机模式运行，非集群模式):

  `sh startup.sh -m standalone`

  Windows
  启动命令：

  `cmd startup.cmd`

  或者双击startup.cmd运行文件。
- 关闭服务器
  Linux/Unix/Mac
  `sh shutdown.sh`  

  Windows
  `cmd shutdown.cmd`
  或者双击shutdown.cmd运行文件。
- 使用Nacos
  - 启动Nacos服务成功后，默认端口8848，访问连接 `http://127.0.0.1:8848/nacos`，默认用户名`nacos`及密码`nacos`。
  - Nacos管理平台分为配置管理和服务管理两功能模块。服务管理模块主要是显示已经发现注册到Nacos上的服务及服务的健康状态。配置管理模块主要是对于服务的配置文件进行统一管理，可添加修改删除各种格式的服务配置文件，可进行配置文件历史版本的回退。

#### 2.4 Nacos在SpringCloud中使用

  ##### 2.4.1 服务注册Nacos
  在SpringCloud的Nacos使用中，客户端确保版本一直。在Maven依赖中，SpringBoot 2.0.x 版本，SpringCloud Finchley.SR2 版本，Nacos 0.2.x 版本。

  在Springboot应用中pom.xml引入nacos依赖
  ```
    <!-- nacos服务注册发现依赖 -->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
      <version>0.2.1.RELEASE</version>
    </dependency>
    <!-- springcloud依赖 -->
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-dependencies</artifactId>
      <version>Finchley.SR1</version>
    </dependency>
  ```
  在Springboot中配置文件的格式分为 `.properties` 和 `.yml`，通常使用 `.yml` 格式，文件名称改为 `bootstrap.yml` , 因为在springboot加载配置文件中`bootstrap.yml` 为主文件，而另外的 `application.yml` 为次文件，加载顺序是 `bootstrap.yml > application.yml`。

  在Springboot的配置文件`bootstrap.yml`中添加Nacos注册地址。
  ```
    spring:
      application:
        name: app-name # 应用名称
      cloud:
        nacos:
          discovery:
            server-addr: localhost:8848 # 注册nacos地址 （此处是默认地址）
  ```

  启动应用后，在Nacos管理界面服务管理的服务列表中查看该服务时候注册上，若注册成功，健康状态为true，否则为false。

  ##### 2.4.2 Nacos配置中心

  在Springboot应用中引入nacos config依赖

  ```
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
      <version>0.2.1.RELEASE</version>
    </dependency>
  ```
  在配置文件`bootstrap.yml`中配置Nacos config 的地址等其他信息。

  ```
    spring:
    application:
      name: app-name # 应用名称
    #配置中心
    cloud:
      nacos:
        config:
          server-addr: 192.168.30.35:8848 # config 地址， 与Nacos注册地址一致
          file-extension: yaml # 配置文件扩展名
          shared-dataids: xx-application.yaml # 公共配置文件 
    profiles:
      active: dev # 配置文件环境 （此处dev表示开发环境）
  ```
  应用启动前，在nacos管理平台的配置管理的配置列表中添加配置信息。

  在`bootstrap.yml`需要配置 spring.application.name ，是因为它是构成 Nacos 配置管理 dataId 字段的一部分。在 Nacos Spring Cloud 中，dataId 的完整格式如下：

  `${prefix}-${spring.profile.active}.${file-extension}`
  - prefix 默认为 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix来配置。
  - spring.profile.active 即为当前环境对应的 profile。 注意：当 spring.profile.active 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 `${prefix}.${file-extension}`
  - file-exetension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml 类型。
  
  可以在配置管理中的配置文件里配置Nacos server的注册地址。
  
  通过 Spring Cloud 原生注解 @RefreshScope 实现配置自动更新：

  ```
    @RestController
    @RequestMapping("/config")
    @RefreshScope
    public class ConfigController {

        @Value("${useLocalCache:false}")
        private boolean useLocalCache;

        @RequestMapping("/get")
        public boolean get() {
            return useLocalCache;
        }
    }
  ```


### 3. Sentinel 简介


