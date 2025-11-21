# RabbitMQ 面试指南

评分说明：5=必须掌握，4=很重要，3=熟练更佳，2=了解即可，1=加分项  
结构：单条聚焦一个知识点；统一使用“是什么 / 为什么 / 怎么用 / 关键细节 / 面试提示”五段，便于面试中层层追问与结构化输出。

---

## 1. RabbitMQ 概述 [5]
- 是什么：开源 AMQP 0-9-1 协议实现的消息代理，核心提供可靠路由、中间缓冲、确认与交付保障；支持插件（管理、延迟、Shovel、Federation）、多协议（AMQP/ MQTT / STOMP / HTTP API）。
- 为什么：解耦生产与消费速率差异，削峰填谷，异步化提升吞吐与用户体验，隔离故障（消费者失败不影响生产者继续写入队列）。
- 怎么用：Producer 通过 TCP/AMQP 建立连接 → 创建 Channel → 声明交换机/队列/绑定 → basicPublish → Consumer 订阅队列（basicConsume）并 ack。建议：启动时幂等声明（重复声明不报错）。
- 关键细节：消息真正落入“队列”前必须经过交换机；队列与交换机均可声明为 durable；消息持久需 deliveryMode=2；协议级心跳维持连接。
- 面试提示：能清晰区分“交换机负责路由，队列负责存储与消费者分发”，说明与 Kafka（日志分区+顺序追加）模型差异。

## 2. 核心架构组件与关系 [5]
- 是什么：Connection（物理 TCP）→ Channel（逻辑会话，轻量多路复用）→ Exchange（路由）→ Binding（路由规则）→ Queue（缓冲与调度）→ Consumer（获取与 ack）。
- 为什么：Channel 降低频繁建立 TCP 的资源成本；分离路由与存储使模式扩展灵活。
- 怎么用：单连接多 Channel；为不同业务隔离 Channel；声明顺序推荐：exchange→queue→binding。
- 关键细节：exclusive queue（仅当前连接使用，连接断开自动删除）；auto-delete queue（最后一个消费者断开删除）；passive 声明用于存在性检测。
- 面试提示：说明“为何不为每个请求新建 Connection，而是复用并创建多个 Channel”。

## 3. Exchange 类型与路由策略 [5]
- 是什么：Direct（精确匹配 routing key）；Topic（* 单词占位/# 多层通配）；Fanout（广播忽略 routing key）；Headers（按 header 键值匹配）；Delayed（插件：基于 x-delay）。
- 为什么：满足不同业务：精确路由、模式匹配、广播分发、头部灵活条件、延迟投递。
- 怎么用：按场景选择：日志泛广播用 fanout；标签分类用 topic；用户 ID 精确用 direct；需要延迟使用 x-delayed-message 插件或 TTL+DLX。
- 关键细节：topic 中 # 可以匹配零或多段；避免过度使用高通配导致大量无关匹配；Headers 性能较低少用。
- 面试提示：给出“一个订单状态事件如何同时推送运营系统（全部）与某业务模块（特定 routing key）”的设计。

## 4. 队列属性与声明细节 [5]
- 是什么：durable（重启保留元数据+持久化消息）、exclusive（仅当前连接）、autoDelete（最后消费者断开删除）、arguments（x-message-ttl / x-max-length / DLX）。
- 为什么：控制生命周期、容量、过期策略、死信处理。
- 怎么用：设置 TTL 统一过期策略；x-max-length 避免无限积压；绑定 DLX 处理过期/拒绝/满队列消息。
- 关键细节：durable 保留队列但非保证队列里所有消息持久（需消息持久）；autoDelete 与临时订阅场景搭配。
- 面试提示：说明“为什么队列 durable + 消息非持久仍可能丢消息”。

## 5. 消息可靠投递与 Publisher Confirms [5]
- 是什么：Confirm 模式：Channel 进入 confirmSelect，Broker 每条或批量返回 ack/nack；Return 回调处理无法路由（mandatory=true）。
- 为什么：保障生产端不盲目丢失消息，失败可重试/降级。
- 怎么用：启用 confirm + 异步监听（减少阻塞）；mandatory 打开 + 回调记录无法路由告警；批量发送后根据序列号维护未确认映射。
- 关键细节：事务（txSelect/txCommit）性能差；推荐 Confirm；nack 可能无原因（内部失败），需重发或持久落盘重试队列。
- 面试提示：对比事务与 Confirm：说明性能差异与场景。

## 6. 消费确认与预取 (QoS) [5]
- 是什么：basicAck（确认成功）、basicNack/basicReject（可设置 requeue）、prefetchCount（每个消费者一次能未确认的最大消息数）。
- 为什么：防止未确认消息过多占用内存；加速公平分发。
- 怎么用：Channel.basicQos(prefetch); 手动 ack 在业务处理成功后调用；失败选择 requeue 或发送死信。
- 关键细节：autoAck=true 导致消息一旦发送即标记已消费（失败丢失）；prefetch 太大导致“长时间锁定大量消息”，其他消费者饥饿。
- 面试提示：解释“为什么设置 prefetch=1 可实现更公平分发”。

## 7. 死信队列 (DLX) 与重试策略 [5]
- 是什么：死信来源：消息被拒绝（requeue=false）、TTL 过期、队列满、长度溢出；通过 x-dead-letter-exchange 转发到指定交换机。
- 为什么：统一汇聚异常消息后做分析或延迟重试。
- 怎么用：原队列 arguments 配置 x-dead-letter-exchange + 可选 x-dead-letter-routing-key；消费者逻辑判断不可处理时 nack(false)。
- 关键细节：死信无限回流风险（错误绑定）；重试次数建议写入 header 或使用单独死信队列分层（retry-x）。
- 面试提示：给出“业务异常短暂不可用如何设置 3 次指数退避重试”方案。

## 8. 延迟消息实现方式对比 [4]
- 是什么：方式一：TTL+DLX+分组队列（多个延迟等级）；方式二：x-delayed-message 插件；方式三：业务侧调度扫描。
- 为什么：补偿任务执行、定时通知。
- 怎么用：TTL+DLX：消息设置 per-message TTL → 过期进入 DLX → 消费时做实际执行；插件方式直接设置 x-delay 属性。
- 关键细节：TTL 基于到期后出队可能存在延迟（过期是惰性）；插件更精确但需部署额外插件；大量延迟消息内存占用。
- 面试提示：说明“为何大量长 TTL 消息占用资源且不推荐统一超长队列”。

## 9. 顺序与幂等保证 [4]
- 是什么：RabbitMQ 不保证跨队列或多消费者全局顺序；单队列单消费者近似顺序；业务需幂等处理重复。
- 为什么：网络重试、nack 重投导致重复；多个消费者并行会乱序。
- 怎么用：关键场景限制一个消费者或引入分片队列（根据 key 路由到固定队列）；入库前做幂等检查。
- 关键细节：不要依赖到达顺序表示业务时间；使用消息携带业务时间戳。
- 面试提示：回答“如何处理库存扣减重复消息”→ 唯一业务键 + compare-and-update。

## 10. 高可用：镜像队列 vs Quorum 队列 [5]
- 是什么：镜像（HA policies：所有节点复制）传统方案；Quorum Queue（Raft 共识，多副本日志复制）。
- 为什么：镜像模式易在大量队列下造成同步压力；Quorum 更可控、崩溃恢复一致性强。
- 怎么用：策略选择：非关键临时用普通队列；核心交易使用 Quorum Queue；配置 x-quorum-initial-group-size。
- 关键细节：Quorum 队列仅支持少量副本（推荐 3~5）；镜像模式需要 policy: ha-mode=all/exactly；迁移需评估容量与磁盘。
- 面试提示：说明“为什么官方推荐新项目使用 Quorum 而非镜像”。

## 11. 集群、Federation 与 Shovel 对比 [4]
- 是什么：Cluster 同步元数据（队列位于单节点）；Federation 按需跨集群转发表消息；Shovel 持续复制从源到目标（跨网络）。
- 为什么：不同场景：同城集群扩展 vs 跨数据中心消息桥接。
- 怎么用：跨区域使用 Federation 或 Shovel；避免直接跨广域组建单集群。
- 关键细节：集群中队列“单点驻留”可能导致热点；解决靠分散创建或 Quorum 分布副本。
- 面试提示：能区分三个概念的适用场景与限制。

## 12. 性能调优关键点 [4]
- 是什么：连接与通道复用、批量发布（publisher confirms异步批）、合理 prefetch、消息大小控制、持久化权衡。
- 为什么：降低 CPU/IO，提升吞吐与稳定性。
- 怎么用：批量发送结合异步 confirm；prefetch 根据平均处理耗时与并行度估算；持久消息仅针对关键数据。
- 关键细节：小消息过多元数据开销高；过大消息建议外部存储引用；磁盘 fsync 影响持久化延迟。
- 面试提示：给出“为什么将 5KB 大消息拆成外部对象引用 + 元数据后 TPS 提升”的案例。

## 13. 典型监控指标与告警 [4]
- 是什么：Ready（就绪未消费）、Unacked（已投递未确认）、Connections、Channels、Publish Rate、Confirm Latency、Disk Free、Memory、Queue Depth。
- 为什么：早期发现积压、消费者阻塞、资源耗尽。
- 怎么用：阈值：Unacked 长时间增长 → 消费端处理慢；Ready 激增 → 消费能力不足或路由错误。
- 关键细节：镜像/Quorum 队列磁盘压力需监控；管理 API 定期采样。
- 面试提示：描述一次“Unacked 激增”定位（消费者线程阻塞或未 ack）流程。

## 14. 常见故障与排查路径 [4]
- 是什么：消息积压、连接耗尽、频繁连接重建、队列单节点热点、重复消费。
- 为什么：配置不当或资源不足。
- 怎么用：查看 management UI → metrics → logs；确认 prefetch、ack 模式；检查网络与心跳超时。
- 关键细节：心跳（默认 60s）过长无法快速感知断链；积压过大持久化慢造成磁盘抖动。
- 面试提示：讲出“五步排查：症状→指标→队列详情→消费者状态→日志”。

## 15. 安全与权限控制 [3]
- 是什么：Virtual Host（隔离命名空间+权限），用户/标签（administrator/monitoring），权限：configure/write/read。
- 为什么：限制误操作与越权访问。
- 怎么用：按系统划分 vhost；仅授予所需队列与交换机访问；TLS 加密通信。
- 关键细节：不要用同一个超级账号运行所有业务；禁用不必要管理权限。
- 面试提示：说明“虚拟主机在多租户中的作用”。

## 16. Java 客户端实践细节 [5]
- 是什么：com.rabbitmq.client API：ConnectionFactory→Connection→Channel→发布与消费。
- 为什么：直接控制 QoS、Confirm、Return 回调实现可靠模式。
- 怎么用：启用：channel.confirmSelect(); channel.basicPublish(exchange, key, props, body); 添加 ConfirmListener；basicConsume(queue, false, DeliverCallback, CancelCallback)。
- 关键细节：Channel 线程不安全；每线程独立或用池；异常后 Channel 不可重用需重建。
- 面试提示：指出“为什么一个 Channel 不能被多线程并发写入”与解决策略。

## 17. 与 Kafka/ActiveMQ 对比要点 [3]
- 是什么：RabbitMQ 多路由模式、多队列驻留；Kafka append-only 日志+分区顺序；ActiveMQ 传统 JMS 风格。
- 为什么：路线差异影响场景：复杂路由选择 RabbitMQ；海量顺序日志与流处理选择 Kafka。
- 怎么用：评估消息量级、顺序需求、路由复杂度、消费模型（扇出 vs 流处理）。
- 关键细节：RabbitMQ 不擅长超高吞吐海量存储；Kafka 不擅长细粒度路由。
- 面试提示：能基于业务需求给出定性对比而非泛泛。

## 18. 端到端一致性与补偿 [4]
- 是什么：生产确认 + 幂等消费 + 重试/死信 + 业务侧补偿表。
- 为什么：保障消息不丢失且重复不破坏业务状态。
- 怎么用：消息携带唯一业务 ID；消费方记录已处理 ID；失败进入 DLX 重试或人工补偿。
- 关键细节：避免使用重新入队无限循环；补偿需要后台巡检任务。
- 面试提示：举例“订单创建消息重复消费如何保持幂等”。

## 19. 连接管理与流控 [3]
- 是什么：Broker 对过载客户端发送流控（basic.flow 已废弃；现代通过 TCP/资源限制）；应用端需感知异常。
- 为什么：防止过度写入耗尽内存。
- 怎么用：监控 publish confirm 延迟突增；合理批次。
- 关键细节：连接数过多内存元数据膨胀；建议复用。
- 面试提示：说明“高并发短连接”带来的隐患。

## 20. 面试速答清单（记忆锚点）[辅助]
1. 确认与事务差异：Confirm 异步高吞吐；事务重且阻塞。  
2. 死信来源：nack(requeue=false)/TTL/队列满/长度溢出。  
3. 顺序策略：单队列单消费者；按 key 分片。  
4. 延迟实现：TTL+DLX vs 插件 vs 应用调度。  
5. 幂等：唯一业务键 + 去重存储。  
6. Quorum 优势：Raft 共识保证一致性与恢复。  
7. mandatory 用途：发现未路由消息。  
8. prefetch 调整：防止某消费者抢占过多消息。  

(完)