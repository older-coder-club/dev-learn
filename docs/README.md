# 资深 Java 工程师面试知识点清单（含重要度评分）

评分说明：5=必须掌握，4=很重要，3=熟练更佳，2=了解即可，1=加分项

## 1. Java 语言与核心 API
- 集合框架（HashMap/ConcurrentHashMap/CopyOnWrite 等实现与复杂度）[5]
- 异常体系与最佳实践（checked/unchecked、异常包装）[4]
- I/O 与 NIO/NIO.2（Channel、Buffer、Selector、零拷贝）[4]
- 函数式与流式编程（Lambda、Stream、并行流陷阱）[4]
- 序列化机制与 JSON 生态（Jackson/Gson；兼容性与安全）[3]
- 时间日期 API（java.time 时区与不变性）[3]

## 2. 并发与多线程
- JMM 与可见性/有序性（happens-before、volatile）[5]
- 锁与同步（synchronized、ReentrantLock、AQS、读写锁、StampedLock）[5]
- 线程池（ThreadPoolExecutor 参数、队列/拒绝策略、调优与监控）[5]
- 原子类与 CAS、自旋/退避、伪共享与缓存行对齐 [4]
- 并发容器（ConcurrentHashMap、BlockingQueue、LongAdder）[4]
- 任务编排（CompletableFuture、ForkJoinPool）[4]

## 3. JVM 与性能
- 内存结构与对象布局（堆/栈/元空间、TLAB、逃逸分析）[5]
- GC 算法与收集器（G1/ZGC/Shenandoah 原理、调优套路）[5]
- 类加载与模块化（双亲委派、ClassLoader、SPI）[4]
- JIT/逃逸/内联与性能分析方法论 [4]
- 诊断工具（jcmd/jmap/jstack/jfr、Arthas、async-profiler）[5]

## 4. Spring 生态
- IoC/DI 与 Bean 生命周期、循环依赖处理 [5]
- AOP 与代理（JDK/CGLIB、切点/通知、事务切面）[4]
- Spring Boot 自动装配（条件装配、Starter 机制、配置优先级）[5]
- Spring MVC/WebFlux 请求处理链与数据绑定/校验 [4]
- 声明式事务（传播/隔离级别、失效场景与最佳实践）[5]
- Actuator 可观测性、配置管理（Profiles/ConfigProps）[4]

## 5. 数据访问与数据库
- SQL 与查询优化（执行计划、索引命中、反模式识别）[5]
- MySQL 基础（B+ 树索引、MVCC、隔离级别、锁/间隙锁）[5]
- 连接池与事务边界（HikariCP、超时/泄露检测）[4]
- ORM（JPA/Hibernate：缓存、延迟加载、N+1、映射陷阱）[4]
- MyBatis（动态 SQL、插件、缓存）[4]
- 分库分表与读写分离、数据一致性策略 [4]

## 6. 缓存与中间件
- Redis 结构与场景（String/Hash/Set/SortedSet/Bitmap/HyperLogLog）[5]
- 持久化与高可用（RDB/AOF、哨兵、Cluster、槽迁移）[4]
- 缓存策略（穿透/击穿/雪崩、双写一致性、热点 Key）[5]
- 消息队列（Kafka：分区/副本/ISR/位移、Exactly-once/幂等）[5]
- Elasticsearch（倒排索引、分片/副本、查询/聚合基础）[3]

## 7. 分布式与微服务
- CAP/一致性模型（强/最终一致、事务外一致性设计）[5]
- 服务注册发现与配置中心（Eureka/Nacos/Consul、优缺点）[4]
- 网关与负载均衡（限流/熔断/降级、重试与超时）[5]
- 分布式事务（TCC/Saga/Outbox、二阶段消息、幂等键）[4]
- ID 生成与时钟问题（Snowflake、时钟回拨）[4]
- 可观测性（三件套：日志/指标/链路；OpenTelemetry）[5]
- API 设计与版本治理、向后兼容与灰度策略 [4]

## 8. 系统设计与高可用
- 高并发架构（读写分离、异步化、削峰填谷、背压）[5]
- 缓存+数据库一致性、幂等性与重试策略 [5]
- 高可用与故障域（副本、选主、仲裁、隔离区）[4]
- 数据分片与路由、一致性哈希 [4]
- 事件驱动/CQRS/事件溯源（取舍与落地）[3]

## 9. 安全
- 认证鉴权（OAuth2/OIDC、JWT、会话安全）[4]
- Web 安全（OWASP Top 10、CORS/CSRF/XSS/SQL 注入）[4]
- 传输与存储安全（TLS/mTLS、密钥与机密管理）[3]
- 审计与合规（最小权限、审计日志、隐私/脱敏）[3]

## 10. 测试与质量保障
- 单元测试（JUnit5、Mockito、参数化/覆盖率度量）[5]
- 集成/端到端测试（Testcontainers、WireMock）[4]
- 合约测试与兼容性回归（PACT）[3]
- 性能/基准测试（JMH、JMeter、容量规划）[4]
- 静态检查与规范（SpotBugs/Checkstyle/PMD、Error Prone）[3]
- CI/CD（流水线、质量门禁、增量回归）[4]

## 11. 运行与运维（DevOps/SRE）
- 容器化（Docker 多阶段构建、镜像减肥与安全）[4]
- Kubernetes（Deployment/Service/Ingress、HPA/资源配额）[4]
- 配置与密钥、服务网格（Istio 基础、流量治理）[3]
- 监控栈（Prometheus/Grafana/ELK、告警与SLO）[4]
- Linux/网络排障（文件描述符、TCP 基础、连接调优）[3]

## 12. 算法与数据结构
- 复杂度分析、常用结构（数组/链表/堆/哈希）[3]
- 树与图（遍历、并查集、最短路基础）[3]
- 工程常用（布隆过滤器、LRU/LFU、跳表、一致性哈希）[4]

## 13. 工程实践与软技能
- 设计原则与模式（SOLID、策略/模板/工厂/观察者）[4]
- 架构文档与 ADR、代码评审要点 [3]
- 故障演练与复盘（5 Whys、无责文化、变更管理）[3]
- 需求澄清与技术取舍（成本/收益/风险沟通）[3]

## 14. 计算机基础（体系结构 / 操作系统 / Linux）
- 计算机体系结构概念（冯·诺依曼、CPU 指令流水线、缓存层级 L1/L2/L3、内存对齐）[4]
- CPU 缓存一致性与内存屏障（MESI、乱序执行、Java 内存模型关系）[5]
- 进程与线程模型（上下文切换成本、调度策略 CFS、用户态 vs 内核态）[5]
- 虚拟内存与分页（页表/多级页表、TLB、缺页异常、内存映射文件）[4]
- Linux 文件系统与 I/O（页缓存、direct I/O、零拷贝 sendfile/splice）[4]
- 系统调用与中断（syscall 入口开销、阻塞 vs 非阻塞、epoll 原理）[4]
- 进程间通信（管道/共享内存/消息队列/Unix Domain Socket）[3]
- 调优与排障基础（top/iostat/vmstat/strace/lsof、负载指标解释）[4]
- 容量与资源限制（ulimit、cgroups CPU/Memory、命名空间隔离）[4]
- 常见性能问题定位（CPU 单核拉满、IO wait、内存泄漏、僵尸进程）[4]

## 15. 计算机网络
- 分层模型（OSI vs TCP/IP、各层职责与常见协议）[5]
- TCP 连接管理与可靠性（握手/挥手、序列号、重传、拥塞控制 CUBIC、Nagle/Delayed ACK）[5]
- UDP 特性与使用场景（无连接、丢包容忍、DNS/视频）[4]
- HTTP/1.1 与性能痛点（队头阻塞、长连接、Pipeline 限制）[4]
- HTTP/2/HTTP/3 演进（多路复用、HPACK/QPACK、QUIC 特性）[4]
- TLS 握手与证书验证（对称/非对称、会话复用、前向安全）[4]
- 代理与负载均衡（反向代理、七层 vs 四层、哈希/一致性哈希）[4]
- 网络安全基础（DDoS 流量清洗、防火墙、WAF、常见端口扫描）[3]
- DNS 与域名解析（缓存层次、TTL、权威与递归、劫持问题）[3]
- NAT 与内网穿透（端口映射、连接追踪、对长连接影响）[3]
- 常见排障方法（抓包 tcpdump/Wireshark、netstat/ss、连接状态 TIME_WAIT/CLOSE_WAIT 分析）[4]