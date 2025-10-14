---
title: Prometheus、Grafana 和 Alertmanager 快速上手
categories: Tech
tags: devops
date: 2025-10-14
---

Prometheus、Grafana 和 Alertmanager 是监控领域的三剑客，今天带你快速了解一下。

---

### 1. Prometheus 一般收集什么样的指标？

Prometheus 是一个时序数据库，其收集的指标（Metrics）是其监控系统的基石。这些指标通常可以分为以下几大类，遵循一些业界公认的方法论，如 Google SRE 的“四大黄金信号”和 RED、USE 方法。

#### a. 基础资源/节点指标 (Node Metrics)
这是最底层、最基础的监控，通常通过 `node_exporter` 收集。
* **CPU:** 使用率、负载 (Load Average)、上下文切换次数、中断次数。
* **内存 (Memory):** 总量、已用、可用、缓存 (Cache/Buffer)、交换空间 (Swap) 使用情况。
* **磁盘 (Disk):** 空间使用率、IOPS (每秒读写次数)、吞吐量 (读写速率)、I/O 等待时间。
* **网络 (Network):** 带宽使用率、TCP 连接数、丢包率、错误包率。

#### b. 容器与编排平台指标 (Container & Orchestration)
在使用 Docker 和 Kubernetes 的环境中，这些至关重要。
* **Kubernetes 集群:**
    * **节点状态:** `kube_node_status_condition` (节点是否 Ready)。
    * **Pod 状态:** `kube_pod_status_phase` (Pod 是否 Running, Pending, Failed)。
    * **容器状态:** `kube_pod_container_status_restarts_total` (容器重启次数，`CrashLoopBackOff` 的关键指标)。
    * **资源请求/限制:** `kube_pod_container_resource_requests` 和 `kube_pod_container_resource_limits` (CPU/内存的请求和限制)。
    * **Deployment/StatefulSet:** `kube_replicaset_spec_replicas` vs `kube_replicaset_status_ready_replicas` (期望副本数 vs 可用副本数)。
* **容器本身:**
    * 通过 `cAdvisor` 收集每个容器的 CPU、内存、网络、文件系统使用情况。

#### c. 应用/服务指标 (Application Metrics) - **核心**
这是最有价值的指标，直接反映了你的服务质量。推荐使用 **RED 方法**来定义：
* **速率 (Rate):** 每秒请求数 (QPS/RPS)。衡量服务的流量大小。
    * *PromQL 示例:* `sum(rate(http_requests_total[5m]))`
* **错误率 (Errors):** 每秒失败的请求数。通常是 HTTP 5xx 错误或 gRPC 错误码。
    * *PromQL 示例:* `sum(rate(http_requests_total{status=~"5.*"}[5m])) / sum(rate(http_requests_total[5m]))`
* **耗时 (Duration):** 请求的响应时间。通常使用百分位数（Percentiles）来衡量，如 P99, P95, P50。这比平均值更能反映用户体验。
    * *PromQL 示例:* `histogram_quantile(0.99, sum(rate(http_request_duration_seconds_bucket[5m])) by (le))`

#### d. 中间件与依赖服务指标 (Middleware & Dependencies)
* **数据库 (MySQL, PostgreSQL):** 连接数、慢查询数量、QPS、主从延迟、事务数。
* **缓存 (Redis):** 内存使用、命中率、连接数、阻塞的客户端数。
* **消息队列 (Kafka, RabbitMQ):** 生产者/消费者速率、**消费者延迟 (Consumer Lag)** - 这是一个核心指标、队列中的消息数。

#### e. 业务指标 (Business Metrics)
将技术监控与业务价值关联起来。
* 用户注册数、订单创建量、支付成功率、在线用户数等。

---

### 2. Grafana 一般配置什么样的报表？

Grafana 的核心思想是**分层和关联**，从宏观概览到微观细节。

#### a. 概览大盘 (Overview Dashboard)
* **目的:** "Single Pane of Glass"，一目了然地展示整个系统的健康状况。
* **内容:**
    * 核心业务指标（如：订单量、活跃用户）。
    * 核心服务的 RED 指标（QPS、错误率、P99 延迟）。
    * 集群资源概览（CPU/内存/磁盘总体使用率）。
    * 当前触发的重大告警列表。

#### b. 服务专属大盘 (Service-specific Dashboard)
* **目的:** 深入分析某个特定微服务。
* **内容:**
    * **黄金信号:** 该服务的 QPS、错误率、P99/P95 延迟的详细图表。
    * **资源使用:** 该服务所有 Pod/实例的 CPU、内存、网络使用情况。
    * **JVM/Go 运行时指标:** 如果适用，展示 GC 次数/耗时、堆内存使用、Goroutine 数量等。
    * **依赖关系:** 对数据库、缓存、其他服务的调用的延迟和错误率。
* **最佳实践:** 使用 Grafana 的**变量 (Variables)** 功能。你可以在仪表板顶部创建一个下拉菜单，动态选择不同的服务、环境 (production/staging) 或实例，一个模板化的仪表板就能监控所有服务。

#### c. 基础设置大盘 (Infrastructure Dashboard)
* **目的:** 监控底层硬件和平台。
* **内容:**
    * **Kubernetes 集群:** 节点健康状况、资源分配（Request/Limit vs Capacity）、Pod 状态分布。
    * **数据库/中间件:** 详细的性能指标，如 MySQL 的 InnoDB 状态、Kafka 的 Consumer Lag 详情等。
    * **物理/虚拟机:** 深入展示每个节点的 CPU、内存、磁盘 I/O 和网络流量。

---

### 3. 告警在哪个平台配置？

告警规则本身在 **Prometheus** 中定义，而告警的处理（去重、分组、路由、抑制）则由 **Alertmanager** 负责。

* **Prometheus:**
    * 在 Prometheus 的配置文件中定义 `alerting rules` (告警规则)。
    * 规则基于 PromQL 表达式。当表达式在一定时间内（`for` 关键字）持续为真时，Prometheus 会将告警状态从 `Pending` 变为 `Firing`。
    * Prometheus 将 `Firing` 状态的告警发送给 Alertmanager。

* **Alertmanager:**
    * 是独立于 Prometheus 的组件。
    * **核心职责:**
        1.  **分组 (Grouping):** 将相似的告警（如：同一个集群的多个节点磁盘满了）合并为一条通知。
        2.  **抑制 (Inhibition):** 当某个严重告警（如：数据库挂了）触发时，抑制由它引发的其他次要告警（如：所有依赖该数据库的服务 API 错误率飙升）。
        3.  **静默 (Silences):** 在计划内维护期间，可以手动屏蔽某些告警。
        4.  **路由 (Routing):** 根据告警的标签 (labels)，将不同严重性或来源的告警发送到不同的渠道（如：邮件、Slack、钉钉、PagerDuty、微信）。

**流程:** PromQL 表达式为真 -> Prometheus 触发告警 -> Alertmanager -> 分组/抑制 -> 路由到正确的人/渠道。

---

### 4. 一般会配置什么告警？

好的告警应该是**可操作的 (Actionable)**，并且**紧急的 (Urgent)**。

#### a. 资源类告警
* `HostHighCpuLoad`: 节点 CPU 负载持续 5 分钟超过 80%。
* `HostHighMemoryUsage`: 节点内存使用率持续 10 分钟超过 85%。
* `HostDiskWillFillIn4Hours`: 按照当前磁盘写入速率，4 小时内磁盘将满。（**预测性告警**）
* `HostDiskSpaceLow`: 磁盘空间低于 15%。

#### b. Kubernetes 类告警
* `KubePodCrashLooping`: Pod 在频繁重启。
* `KubeDeploymentReplicasMismatch`: Deployment 的可用副本数远低于期望值。
* `KubeNodeNotReady`: 集群中有节点处于 `NotReady` 状态。
* `KubeAPIDown`: API Server 无法访问。

#### c. 服务质量类告警 (基于 SLO)
这类告警是最重要的，直接关系到用户体验。
* `HighErrorRate`: 服务的 5xx 错误率在过去 5 分钟内高于 1%。
* `HighLatency`: 服务的 P99 响应时间在过去 10 分钟内持续高于 500ms。
* `ServiceDown`: Prometheus 无法抓取到任何一个服务的实例 (`up == 0`)。

#### d. 业务/应用逻辑类告警
* `KafkaConsumerGroupLagTooHigh`: Kafka 消费组延迟持续增长超过阈值，表示消息处理不过来。
* `RedisOutOfMemory`: Redis 实例即将耗尽内存。
* `SSLCertificateWillExpireSoon`: SSL 证书将在 14 天内过期。

---

### 5. 有什么最佳实践和原则？

1.  **告警于症候，而非原因 (Alert on Symptoms, Not Causes):**
    * **坏实践:** CPU 使用率 90% 就告警。
    * **好实践:** 用户请求的 P99 延迟过高或错误率过高时告警。
    * **原因:** 高 CPU 并不一定意味着问题，也许它正在高效地处理批处理任务。只有当它导致了用户可见的“症状”（如慢、错误）时，才需要人去介入。

2.  **定义服务等级目标 (SLO/SLI):**
    * **SLI (服务等级指标):** 你测量的东西，如请求延迟、错误率。
    * **SLO (服务等级目标):** 你承诺的目标，如“99.9% 的请求在 200ms 内完成”。
    * 你的告警应该围绕“SLO 预算即将耗尽”来设置，这使得告警与业务目标直接挂钩。

3.  **告警分级 (Tiered Alerting):**
    * **紧急 (Urgent / Pager):** 需要立即处理，会半夜打电话叫醒工程师。例如：主站完全不可用。
    * **重要 (Important / Ticket):** 需要在工作时间内处理。例如：某个非核心服务的延迟升高。
    * **信息 (Info / Logging):** 仅作记录，无需人工干预。

4.  **减少告警噪音:**
    * 一个问题只发一个告警。善用 Alertmanager 的分组和抑制功能。
    * 每个告警都必须是可操作的，并附带排查手册 (Runbook) 的链接。
    * 定期回顾和优化告警规则，删除那些频繁误报或无人处理的告警。

5.  **仪表盘讲述故事 (Dashboards Tell a Story):**
    * 好的仪表盘应该能引导你从“哪里出问题了？”（概览大盘）到“是什么问题？”（服务大盘）再到“为什么会出问题？”（基础设施或依赖大盘）。
    * 充分利用 Grafana 的图表注解和链接功能，将告警和相关的仪表盘关联起来。

6.  **监控即代码 (Monitoring as Code):**
    * 将 Prometheus 的规则、Alertmanager 的配置、Grafana 的仪表盘（通过 JSON model）全部存储在 Git 中进行版本控制。使用 CI/CD 流程来部署变更，确保监控系统的健壮性和可追溯性。
