---
title: 微服务改造-3 Kong路由配置实战及JWT校验
date: 2019-11-17 21:30:50
categories: 微服务改造
tags:
- 微服务
- 网关
---

[toc]

### 一、 背景

(一)网关用处

1. Open API: 将自身数据、能力作为开放平台对外开放. 这必然涉及到客户应用的接入、API权限的管理、调用次数管理等, 必然会有一个统一的入口进行管理
2. 微服务网关: 负载均衡,缓存,路由,访问控制,服务代理,监控,日志等
3. API服务管理平台: 由于不同系统间存在大量的API服务互相调用, 因此需要对系统间服务调用进行管理, 清晰的看到个系统调用关系, 对系统间调用进行监控等.
<!--more-->
(二)选型-Kong

Kong基于OpenResty(Nginx + lua), 具有高性能、配置简单、插件丰富、业务代码无入侵等特点,故使用了接口网关Kong.


(三)JWT拦截

基于等保与接口安全考虑, 要求所有服务私有API都要有安全校验.

JWT是目前最为流行的token生成规则,基于JSON开发标准, 适用于身份验证, SSO,鉴权等方案中.

Kong支持配置JWT插件, 并支持在不同级别Global,Service,Route上配置, 动态生效, 配置灵活, 因此我们开始了在Kong上配置JWT的实战方案调研.

### 二、需求与调研目的

1. 公有API无需进行JWT拦截
2. 私有API必须进行JWT拦截
3. 服务间内网互相调用, 所有API都无需拦截
4. 客户端使用公有API进行登录,同时获取到签发JWT, 访问私有API需携带token
5. 对代码无影响或影响较少(通过拦截器配置, 不能影响到方法级别)

目的：通过调研方案, 能够实现上述需求.

### 三、调研方案分析

#### (一) Project -> Service -> Route    不满足需求

```mermaid
Project->Kong.Service:一对一,Service.URL指向项目根路径
#Note over Project实体,Kong.Service: Service.URL配置为项目根路径
Kong.Service->Kong.Route:一对一,Route配置paths为/serviceName
Note right of Kong.Route: strip_path=true
```

<u>缺点: <b>代理了根路径,无法针对公有API和私有API分别配置</b></u>



####(二) Project -> Service -> Multi Route (by paths)    不满足需求

方案:  统计公有API和私有API, 并分别拆分出多条Route. 尝试使用`paths`进行区分.

```sequence
Project->Kong.Service:一对一,Service.URL指向项目根路径
#Note over Project实体,Kong.Service: Service.URL配置为项目根路径
Kong.Service->Kong.Route1:Route配置paths为/serviceName/${path_to_auth}
Kong.Service->Kong.Route2:Route配置paths为/serviceName/${path_to_noauth}
Note right of Kong.Route2: strip_path=true
```

<u>结论: **完全无法满足需求, 因为strip_path属性会将匹配的paths全部删除, 导致路由到项目的地址错误.**</u>

Eg:

``` yaml
#kong配置两条Route

route1: /weikefangtest/api/common         					   #-不配置JWT插件

route2: /weikefangtest/api/v1, /weikefangtest/api/v2   #-配置JWT插件

#----------------------------期望---------------------------------------------
访问:     https://open.ibeiwai.com/weikefangtest/api/common/login
期待路径: ${weikefang_domain}/api/common/login

#--------------------- 结果1: 当Route.Strip_path=true ------------------------
实际路径: ${weikefang_domain}/login


#--------------------- 结果1: 当Route.Strip_path=false ------------------------
实际路径: ${weikefang_domain}/weikefangtest/api/common/login
```

所以尝试寻找解决方案, 能够实现:

**部分匹配路径 或者 在请求时能够切除或者替换部分路径**

在官方文档、官方论坛、StackOverflow、QQ群多方查询下, 最终仍然无法实现上述需求.

* kong的配置项`strip_path`支持正则表达式的`paths`, 但也只能全部切除路径. 
* 找到request-transformer-advanced企业版插件, 有replace uri的功能
* 企业版没发现购买入口, 在kong官网申请企业版试用, 一周内未收到反馈

#### (三) Project -> Multi Service -> Route (by service)   不满足需求

``` sequence
Project->Kong.Service1:Service.URL指向接口需要鉴权的根路径
Project->Kong.Service2:URL指向不需要鉴权的根路径
#Note over Project实体,Kong.Service: Service.URL配置为项目根路径
Kong.Service1->Kong.Route1:Route配置paths为/${path_to_auth}
Kong.Service2->Kong.Route2:Route配置paths为/${path_to_noauth}
Note right of Kong.Route2: strip_path=false
```

结论: 当有多个项目时, 存在路径冲突. 

Eg:

```

```

#### (四)Project -> Service -> Multi Route (by headers) 满足需求

#### (五) Kong Project->Service->根Route  + Spring Interceptor 满足需求

###四、后端主动or前端主动

方案1.后端存储最后一次token，后端判断toekn,合法超时，刷新token推给前端

优点：前端改动小

缺点：后端实现复杂，需要而外存储。

存在问题：

1.由于前端会存在并发请求，当并发请求收到多个jwt token时，由于前端无序，会导致前端的jwt token和后端存储的jwt token不一致，导致不匹配

解决方案：

端会存在并发请求。当token失效时，遇到并发情况时，就搞个分布式锁让并行变成串行。

方案2.

前端解码token。拿到过期时间，和当前时间进行判断。如果快过期，主动调用获取新token.

缺点：前端每次请求需要解码判断

优点：后端压力小，不需要存储