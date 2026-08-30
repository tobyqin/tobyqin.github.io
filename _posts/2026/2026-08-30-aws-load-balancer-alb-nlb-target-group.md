---
title: 一文搞懂 AWS 负载均衡：ALB、NLB 与 Target Group 架构全解析
categories: [Tech]
tags: [devops, engineering]
date: 2026-08-30
---

在构建高可用、可扩展的后端架构时，负载均衡器（Load Balancer）是不可或缺的组件。但在 AWS 体系中，经常会遇到一串概念：ELB、ALB、NLB、Target Group。它们之间到底是什么关系？应该怎么选型？

## 1. 概念先理清：家族与成员

很多初学者容易混淆 ELB 与 ALB/NLB，其实它们是包含关系：

- **ELB（Elastic Load Balancing）**：AWS 负载均衡产品的家族总称（相当于"品牌"）。
- **ALB（Application Load Balancer）**：工作在 Layer 7（应用层），专注于 HTTP/HTTPS/gRPC 协议。
- **NLB（Network Load Balancer）**：工作在 Layer 4（传输层），专注于 TCP/UDP 高性能转发。

```
                    ┌── ALB (Application Load Balancer)  -> 7层：HTTP/HTTPS/路径路由
                    ├── NLB (Network Load Balancer)      -> 4层：TCP/UDP/极速/固定IP
ELB (产品家族) ──────┼── GLB (Gateway Load Balancer)      -> 3/4层：第三方防火墙/安全设备引流
                    └── CLB (Classic Load Balancer)      -> 传统初代(不建议新项目使用)
```

## 2. 核心架构关系：从请求到机器

一个完整的流量转发链条如下：

```
客户端请求 (Client Request)
        │
        ▼
[ Load Balancer: ALB / NLB ]  ──> 负责入口监听 (Listener 80/443) 与规则匹配 (Rules)
        │
        ▼
[ Target Group (目标组) ]     ──> 负责业务分组、健康检查 (Health Check) 与分流算法
        │
    ┌───┴──────────────┐
    (分流策略：轮询 Round Robin / 最少连接)
    ▼                  ▼
[ EC2 / Pod A ]    [ EC2 / Pod B ]  ──> 实际干活的业务实例 (同质副本)
```

**什么是 Target Group（目标组）？**

- **同质资源池**：一个 Target Group 内部通常挂载多台功能完全相同的机器或容器（用于高可用与水平弹性伸缩）。
- **内置分流与健康检查**：Target Group 会定期检查实例健康状态。当请求到达时，自动按轮询（Round Robin）或最少请求（Least Outstanding Requests）将流量均匀分发给健康的节点。

## 3. ALB vs NLB 核心区别

```
                ALB (Layer 7 拆包解析)                   NLB (Layer 4 盲转)
            ┌────────────────────────────┐          ┌────────────────────────────┐
            │ HTTP Request               │          │ TCP/UDP Packet             │
            │ ├─ Method: GET             │          │ ├─ Source IP: 1.2.3.4      │
            │ ├─ Path: /api/orders       │          │ ├─ Dest IP:   5.6.7.8      │
            │ └─ Header: User-Agent      │          │ └─ Port:      443          │
            └─────────────┬──────────────┘          └─────────────┬──────────────┘
                          │                                       │
                 [ 拆开分析，按路径分流 ]                 [ 不看内容，极速透传 ]
                          ▼                                       ▼
                    Target Group                            Target Group
```

| 维度 | ALB (应用型负载均衡) | NLB (网络型负载均衡) |
| --- | --- | --- |
| OSI 层级 | Layer 7（应用层） | Layer 4（传输层） |
| 支持协议 | HTTP, HTTPS, gRPC, WebSocket | TCP, UDP, TLS |
| 路由能力 | 内容感知路由（支持根据 URL 路径 `/api/*`、Header、Host 等分流） | 仅根据 IP + 端口 转发 |
| 公网 IP 特征 | 动态 IP（对外仅提供 DNS 域名） | 支持固定静态 IP（Elastic IP） |
| 性能与吞吐 | 毫秒级延迟，应对超大突发流量可能需要阶梯扩容 | 亚毫秒级（微秒级）延迟，支持千万级 QPS 且无需预热 |
| 客户端真实 IP | 写入 X-Forwarded-For HTTP 请求头 | 原生支持 源 IP 透传（Preserve Source IP） |
| 典型场景 | Web 网站、REST API、微服务容器路由、对接 WAF | 游戏网络包、IoT(MQTT)、高频交易、需要固定 IP 白名单的场景 |

## 4. 关键疑问：既然 ALB 能直连 Target Group，为什么还要 NLB？

很多开发者会问：既然 ALB 既懂 HTTP 又能按 URL 路由，还要 NLB 干嘛？

最关键的差异在于：**ALB 背后是动态扩缩容的计算集群，无法提供固定公网 IP**。

当你遇到以下场景时，必须使用 NLB：

- **对接严格的安全防火墙**：银行、政企客户的防火墙只允许配置固定 IP 白名单，不支持配置域名白名单。
- **非 HTTP 协议通信**：游戏客户端（UDP）、物联网设备通信（MQTT/TCP）、音视频推流等。
- **极端突发流量与超低延迟**：秒杀场景下几十倍的瞬时流量冲顶，NLB 无需预热即可直接承载。

## 5. 进阶玩法：NLB + ALB 组合架构

如果既需要固定公网 IP，又想要 ALB 的 7 层路径路由能力，业内标准的解决方案是串联部署：

```
客户端 (外部请求)
    │
    ▼
[ NLB ]  ──────> 对外提供固定的 Elastic IP (满足防火墙白名单，扛住外网突发流量)
    │ (4层直通)
    ▼
[ ALB ]  ──────> 拆解 HTTP 报文，做 /user、/order 路径路由及 WAF 安全过滤
    │
    ▼
[ Target Group ] ──> 业务 Pod / EC2 实例集群
```

## 6. 一句话选型指南

- **选 ALB**：标准 Web / API 网站、微服务容器调度、按网址/Header 分流、接入 WAF。
- **选 NLB**：TCP/UDP 自定义协议、物联网/游戏、追求微秒级延迟、下游客户强制要求固定公网 IP 白名单。
- **组合使用（NLB + ALB）**：既要固定 IP 白名单，又要 7 层路径高级路由。

> **关于作者：**
>
> Toby Qin, 从事软件测试开发、DevOps、研发效能相关工作，转载请注明原文出处。
>
> 欢迎关注我的博客 <https://tobyqin.cn>，你可以到我的公众号中去当吃瓜群众。
>
> ![Toby's Blog](https://image.tobyqin.cn/wechat-qrcode.jpg)
