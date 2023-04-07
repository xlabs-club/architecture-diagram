# Service Mesh

基于 Envoy + Java Agent 的智能路由服务实现方案介绍。

## 整体架构

智能路由服务从逻辑上分为数据平面和控制平面，主要包含以下组件。

- Nacos：服务注册中心，配置中心。
- XDS Server：对接服务注册中心、配置中心，实现 XDS 协议将配置下发到 Envoy。
- Envoy + WASM Plugin：通过 Envoy 代理流量，自定义 WASM 插件实现按照企业、用户路由到不同服务，实现自定义负载均衡。同时承担熔断、分布式限流等功能。
- Java Agent：增强 Java 应用 Http Client，拦截 OkHttp、Apache Http Client、RestTemplate、OpenFeign 等客户端调用，自动将服务重定向到 Envoy，实现服务发现和灾备切换。

```mermaid
C4Context
      title 基于 Envoy + Java Agent 的智能路由服务

      Enterprise_Boundary(dp, "Data Plane") {
        Container(appA, "Application A", "Java,Agent", "Agent 拦截客户端重定向到 Envoy")
        Container(envoy, "Envoy Proxy", "Envoy,WASM", "代理所有入口流量<br>基于企业、服务负载均衡")
        Container(appB, "Application B", "Java,Agent", "应用注册到服务中心")
        
        Rel(appA, envoy, "request by name", "http")
        Rel(envoy, appB, "http proxy & lb", "http")
        UpdateRelStyle(appA, envoy, $offsetX="-40",$offsetY="-40")
        UpdateRelStyle(envoy, appB, $offsetX="-40",$offsetY="-40")
      }

      Enterprise_Boundary(cp, "Control Plane") {
         System(registry, "服务注册", "服务注册，服务元数据<br>配置管理，双向刷新")
         Container(xdsServer, "控制面板", "Java,Grpc", "对接服务注册中心配置中心<br> 实现 XDS 协议将配置下发到 Envoy")
         System(config, "配置管理",  "服务注册，服务元数据<br>配置管理，双向刷新")

         Rel_L(xdsServer, registry, "实时获取服务实例")
         Rel_R(xdsServer, config, "实时配置刷新")
         UpdateRelStyle(xdsServer, registry, $offsetX="-40",$offsetY="-20")
         UpdateRelStyle(xdsServer, config, $offsetX="-40",$offsetY="-20")
      
      }
      
      Rel_U(xdsServer,envoy,"通过 XDS 实现配置动态下发","grpc")
      Rel_D(appA, registry, "服务注册，配置刷新")
      Rel_D(appB, config, "服务注册，配置刷新")
      
      UpdateLayoutConfig($c4ShapeInRow="3", $c4BoundaryInRow="1")
      
```
