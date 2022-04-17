# 短域名系统设计



## 系统功能需求

1. 接受长域名信息，返回短域名信息
2. 接受短域名信息，返回长域名信息
3. 短域名最大长度为8
4. 使用JVM内存



## 非功能性需求

1. 可用性达到99.9%
2. 可快速的扩展来提高系统吞吐量
3. 接口平均响应时间在200ms以内



## 假设

1. 长域名的最大长度，以兼容各大浏览器支持的URL长度来决定，取IE的长度限制2083个字符，短域名长度占8个字符，一条记录的最大长度为2091个字符，以JVM堆内存4GB来计算，按长连接平均长度为最大长度的一半来估算，可保存410W+组记录。
2. 同样的长域名，在1个小时内，请求时返回相同的短域名。
3. 读多写少，短域名转长域名的请求量约为长域名转短域名请求量的10倍。
4. 短域名的生成随机，不可预测。



## 设计思路

#### 系统功能需求设计

1. 短域名生成算法：采用发号器算法，10进制数转换为62进制。
2. 拆分为web服务和SOA服务，web服务接收请求，向SOA服务请求号段并存储长短域名映射记录。
3. web服务部署多个节点，初始时从发号器批量拿取10000个号，保存在10个发号器worker中，请求时随机从10个worker中选择一个进行发号；并使用缓存框架caffeine实现本地缓存，保存1000个最近写入的短域名到长域名的映射。
4. SOA服务部署单点服务，使用内存缓存框架caffeine存储长短域名映射

#### 非功能性需求设计

1. web服务与SOA服务采用Deployment部署运行在Kubernetes环境上，必要时可快速扩容增加web服务副本数。
2. 通过Kubernetes的存活探针保证系统可用性，在服务不可用时自动重启服务，保证服务的可用性。
3. 开启spring boot服务的actuator，集成Prometheus监控, 增加相应监控指标告警。



# 系统架构

#### 逻辑架构

![](images\1. 逻辑架构.png)

#### 物理架构

![](images\2. 物理架构.png)



#### 业务流程

1. ###### 长域名转短域名

   ![](images\3.1.长域名转短域名.png)

2. ###### 短域名查长域名

   ![](images\3.2.短域名查长域名.png)