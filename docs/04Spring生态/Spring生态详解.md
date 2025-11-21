# Spring 生态面试指南（逐条细化版）

评分说明：5=必须掌握，4=很重要，3=熟练更佳，2=了解即可，1=加分项  
说明：将原条目拆分为更细粒度知识点；每条聚焦一个主题，按“是什么 / 为什么 / 怎么用 / 关键细节 / 面试提示”结构，便于层层追问与系统性回答。  

---

## 1. BeanDefinition 与 IoC 容器结构 [5]
- 是什么：BeanDefinition 描述 Bean 的元信息（class、scope、lazy、依赖、初始化与销毁方法、属性等）；IoC 容器（BeanFactory/ApplicationContext）基于这些定义进行实例化与生命周期管理。  
- 为什么：将对象创建与依赖管理外置，支持配置与扩展点（后处理器、条件化装配）。  
- 怎么用：通过 @Configuration + @Bean / @ComponentScan / Import 注册；使用 BeanFactoryPostProcessor 修改定义；调试可查看 BeanDefinitionNames。  
- 关键细节：BeanDefinition 合并（子定义覆盖父定义）；FactoryBean 与普通 Bean 的区分（getBean("xxx") vs &xxx）；别名与 primary 冲突处理。  
- 面试提示：说明 ApplicationContext 比 BeanFactory 多了哪些企业特性（国际化、事件、资源加载、自动装配）。  

## 2. ApplicationContext Refresh 流程关键阶段 [5]
- 是什么：refresh() 包含：准备环境 → 加载 Bean 定义 → BeanFactory 后处理 → 注册 BeanPostProcessor → 实例化单例 → 初始化事件与监听器 → 完成。  
- 为什么：理解扩展钩子所在位置，便于注入自定义逻辑。  
- 怎么用：在 BeanFactoryPostProcessor 阶段修改定义；在 BeanPostProcessor 中增强实例；ContextRefreshedEvent 监听。  
- 关键细节：单例预实例化发生在注册完所有 BeanPostProcessor 后；父子容器 Bean 查找逻辑；早期单例暴露。  
- 面试提示：列出顺序并指出某阶段可干预点；解释为何某些增强必须是 BeanPostProcessor。  

## 3. Bean 生命周期详解 [5]
- 是什么：加载定义 → 实例化（构造或工厂方法） → 属性填充 → Aware 回调 → BeanPostProcessor.before → 初始化（@PostConstruct / InitializingBean / init-method）→ BeanPostProcessor.after → 可用 → 销毁（@PreDestroy / DisposableBean / destroy-method）。  
- 为什么：在合适阶段插入校验、代理、资源初始化与释放。  
- 怎么用：实现 InitializingBean 做运行时检查；通过 BeanPostProcessor 包装代理（例如 AOP）；使用 destroy-method 释放连接。  
- 关键细节：顺序冲突处理（多个初始化回调都执行）；提前引用导致尚未初始化的 Bean 被其他 Bean 使用；原型作用域不托管销毁。  
- 面试提示：画出生命周期图并指出在哪些点“无法”再改变属性。  

## 4. BeanPostProcessor 与 BeanFactoryPostProcessor 区别 [5]
- 是什么：前者处理 Bean“实例”级别，后者处理 BeanDefinition“元数据”级别。  
- 为什么：分离“定义变更”与“实例增强”，降低耦合与风险。  
- 怎么用：自定义属性占位符、YAML 解析属于工厂后处理；添加代理、校验逻辑属于 BeanPostProcessor。  
- 关键细节：执行时机：BeanFactoryPostProcessor 在单例实例化前；Ordered/PriorityOrdered 决定顺序；重复注册风险。  
- 面试提示：回答“为什么修改类名不能在 BeanPostProcessor 中做”。  

## 5. 三级缓存与循环依赖解决机制 [5]
- 是什么：singletonFactories（ObjectFactory 生成早期代理）、earlySingletonObjects（早期暴露实例）、singletonObjects（最终单例）。  
- 为什么：在属性注入阶段允许引用彼此但避免未完成初始化的重复创建。  
- 怎么用：避免构造器循环（无法通过此机制解决）；改用 Setter 或 @Lazy；合理拆分职责。  
- 关键细节：AOP 早期暴露代理；原型作用域不支持自动循环解决；构造器注入环最终报异常。  
- 面试提示：描述三级缓存数据流并说明“为什么构造器环失败”。  

## 6. 作用域与线程上下文 [4]
- 是什么：singleton / prototype / request / session / application / websocket / 自定义（如 ThreadLocalScope）。  
- 为什么：匹配对象生命周期与资源使用模式。  
- 怎么用：@Scope + proxyMode (ScopedProxy) 解决在单例中注入 request scoped Bean 问题。  
- 关键细节：prototype 不自动销毁；会话作用域需容器支持；代理模式避免过早创建。  
- 面试提示：说明一个 request Bean 注入 Controller 的代理工作原理。  

## 7. 条件化装配与 Profile [4]
- 是什么：@ConditionalX 系列（OnClass/OnMissingBean/OnProperty/OnExpression）、@Profile 控制环境差异。  
- 为什么：减少硬编码分支与复杂环境 if-else。  
- 怎么用：配置不同数据源；根据属性启用可选模块；Profile 与 Conditional 可组合。  
- 关键细节：条件评估在 BeanDefinition 阶段；顺序影响最终结果；@ConditionalOnMissingBean 常与用户自定义冲突防止覆盖。  
- 面试提示：给出“用户自定义覆盖默认自动装配”示例。  

## 8. 属性绑定与 @ConfigurationProperties [5]
- 是什么：将外部化配置（properties/yaml/环境变量）绑定到 POJO；支持松散命名与校验。  
- 为什么：集中配置管理与类型安全；避免散落 @Value。  
- 怎么用：@ConfigurationProperties + @EnableConfigurationProperties 或扫描；引入 @Validated 强制校验。  
- 关键细节：与 @Value 区别（批量 vs 零散）；嵌套对象需配置前缀；ConstructorBinding 在只读场景。  
- 面试提示：讲“从散乱 @Value 到集中配置类”的演进。  

## 9. FactoryBean 与普通 Bean 区分 [4]
- 是什么：特殊 Bean：getObject() 返回实际产品对象；容器内注册名称 vs &name 获取工厂本身。  
- 为什么：支持复杂初始化（代理、动态创建）屏蔽调用方细节。  
- 怎么用：MyBatis-SqlSessionFactoryBean 等场景；自定义 FactoryBean 生成远程代理。  
- 关键细节：作用域与生命周期遵循产品对象行为；循环依赖处理差别。  
- 面试提示：问“如何拿到 FactoryBean 本体”回答 & + beanName。  

## 10. ConversionService 与类型转换体系 [4]
- 是什么：统一类型转换与格式化框架；Converter / GenericConverter / Formatter。  
- 为什么：降低控制器层参数与属性注入转换样板。  
- 怎么用：注册自定义 Converter；使用 @DateTimeFormat；全局配置 FormattingConversionService。  
- 关键细节：优先级：自定义 > 默认；Formatter 面向打印与解析；转换失败抛异常进入 BindingResult。  
- 面试提示：说明为什么推荐集中注册转换。  

## 11. 校验与 JSR-303 集成 [4]
- 是什么：Bean Validation：@NotNull/@Size 等注解 + Validator；Spring 接入参数与对象校验。  
- 为什么：将约束与模型共存保持一致性。  
- 怎么用：方法参数加 @Valid/@Validated；BindingResult 捕获错误；分组校验。  
- 关键细节：嵌套对象需级联 @Valid；默认仅对控制器参数自动触发。  
- 面试提示：说明 @Validated 与 @Valid 在分组场景的区别。  

## 12. Spring AOP 代理模型与织入点 [5]
- 是什么：基于代理（JDK 动态或 CGLIB）在运行时增强方法调用；非编译时织入。  
- 为什么：横切关注点（事务、日志、安全）与业务解耦。  
- 怎么用：@EnableAspectJAutoProxy + @Aspect；定义切点表达式 + 通知。  
- 关键细节：final 方法不可增强；自调用绕过代理；顺序通过 @Order 控制；目标类多接口时默认 JDK。  
- 面试提示：回答“为什么事务在同类内部调用失效”。  

## 13. 切点表达式类型与组合 [4]
- 是什么：execution、within、this、target、args、@annotation、@within、@target 等。  
- 为什么：灵活选择增强范围，避免误匹配。  
- 怎么用：组合 && / || / !；限定包与方法签名。  
- 关键细节：this/target 区分代理类型与实际目标；@annotation 捕获具体方法注解。  
- 面试提示：举一个“仅增强标注 @Transactional 方法”的表达式。  

## 14. @Transactional 原理与代理增强 [5]
- 是什么：基于代理拦截公共方法，在调用前后由事务管理器获取/提交/回滚资源。  
- 为什么：屏蔽底层 JDBC/JPA 事务处理样板。  
- 怎么用：标注在类或方法；配置 PlatformTransactionManager；设置传播与隔离。  
- 关键细节：仅 public 方法有效；自调用失效；异常被捕获不回滚；非 RuntimeException 需 rollbackFor。  
- 面试提示：列出至少五个失效场景。  

## 15. 事务传播行为详解 [5]
- 是什么：REQUIRED / REQUIRES_NEW / SUPPORTS / MANDATORY / NOT_SUPPORTED / NEVER / NESTED。  
- 为什么：控制嵌套调用事务边界与资源管理。  
- 怎么用：子方法需独立回滚用 REQUIRES_NEW；多层调用共享事务用 REQUIRED。  
- 关键细节：NESTED 依赖保存点（DataSourceTransactionManager）；REQUIRES_NEW 暂停外部事务；NOT_SUPPORTED 挂起处理。  
- 面试提示：给出外层成功内层失败需独立回滚场景选择。  

## 16. 隔离级别与数据库实现差异 [4]
- 是什么：READ_UNCOMMITTED/READ_COMMITTED/REPEATABLE_READ/SERIALIZABLE 以及幻读、不可重复读、脏读概念。  
- 为什么：限制并发读写异常影响正确性。  
- 怎么用：@Transactional(isolation=...)；根据数据库（MySQL InnoDB 默认 Repeatable Read）调整。  
- 关键细节：部分数据库不支持全部级别（如 MySQL 没有真正 READ_UNCOMMITTED 仅意义差别）；Serializable 性能低。  
- 面试提示：说明幻读在 MVCC 下的处理方式。  

## 17. readOnly 与事务优化 [3]
- 是什么：向底层标识事务仅执行查询；可能跳过脏检查。  
- 为什么：减少不必要 flush，提高性能。  
- 怎么用：@Transactional(readOnly=true)；与 ORM flush 模式结合。  
- 关键细节：不保证绝对写阻止；驱动是否优化取决于实现。  
- 面试提示：说明“readOnly 不等于只读数据库”。  

## 18. Actuator 基础与端点扩展 [4]
- 是什么：提供运行时健康、配置、指标、线程/堆转储等管理端点。  
- 为什么：提升可观测性与运维效率。  
- 怎么用：management.endpoints.web.exposure.include 控制暴露；自定义 @Endpoint 和 @ReadOperation。  
- 关键细节：敏感端点需鉴权；heapdump/threaddump 影响资源。  
- 面试提示：给出保护 /env 的安全措施。  

## 19. Micrometer 指标设计与标签控制 [4]
- 是什么：统一指标 API：Counter/Gauge/Timer/DistributionSummary/LongTaskTimer。  
- 为什么：跨监控系统（Prometheus、Influx、OTel）可移植。  
- 怎么用：@Timed 或手动 registry.timer().record；自定义 MeterBinder。  
- 关键细节：标签爆炸（高基数）导致存储压力；Gauge 要注意对象生命周期。  
- 面试提示：说明“高基数标签危害”。  

## 20. 健康检查与 Liveness/Readiness [4]
- 是什么：健康端点返回组件状态；区分存活（进程是否可继续）与就绪（是否可接流量）。  
- 为什么：Kubernetes 滚动升级与故障隔离。  
- 怎么用：实现 HealthIndicator；添加 readiness group；慢依赖延迟就绪。  
- 关键细节：不要将外部波动直接标红导致频繁重启；分级展示。  
- 面试提示：解释“数据库短暂抖动不一定要下线实例”。  

## 21. Spring MVC 调用链核心组件 [5]
- 是什么：DispatcherServlet → HandlerMapping → HandlerAdapter → HandlerMethod → 参数解析 → 返回值处理 → 异常解析。  
- 为什么：清晰分层方便扩展与插拔。  
- 怎么用：自定义 HandlerMethodArgumentResolver 解析复杂参数；@ControllerAdvice 管理异常。  
- 关键细节：消息转换器顺序影响内容协商；异步请求 DeferredResult/WebAsyncManager。  
- 面试提示：描述一次请求经过的主要扩展点。  

## 22. 参数绑定与类型转换流程 [4]
- 是什么：WebDataBinder 将请求参数（query/form/json）绑定到方法参数对象。  
- 为什么：减少手工解析与校验样板。  
- 怎么用：自定义 Converter 或 Formatter；使用 @InitBinder 局部定制。  
- 关键细节：BindingResult 捕获错误；忽略字段策略；日期转换失败示例。  
- 面试提示：说明为什么推荐用 DTO 而非直接实体。  

## 23. 异常处理与统一响应 [4]
- 是什么：@ControllerAdvice + @ExceptionHandler 统一响应格式；ResponseStatusException 支持 HTTP 状态。  
- 为什么：提升一致性与可维护性。  
- 怎么用：定义全局异常映射；日志与 TraceId 集成。  
- 关键细节：异常链处理；HandlerExceptionResolver 顺序。  
- 面试提示：解释“不应在 Controller 吞掉异常”。  

## 24. Filter vs Interceptor vs AOP 区别 [4]
- 是什么：Filter 属于 Servlet 层；Interceptor 属于 Spring MVC 层；AOP 基于代理与方法级增强。  
- 为什么：针对不同关注点选择最合适层。  
- 怎么用：安全/跨域用 Filter；登录态/统计用 Interceptor；事务/日志结构化用 AOP。  
- 关键细节：执行顺序：Filter→DispatcherServlet→Interceptor→Controller→AOP（方法内部）；异步请求特殊。  
- 面试提示：列出一个“链路追踪”实现分层组合示例。  

## 25. WebFlux 响应式模型 [4]
- 是什么：基于 Reactor (Mono/Flux) 的非阻塞、事件驱动处理；默认 Netty 事件循环。  
- 为什么：高并发 IO 等待场景提升资源使用效率。  
- 怎么用：Controller 返回 Mono/Flux；使用 WebClient 异步调用；背压机制保护下游。  
- 关键细节：阻塞调用需 Schedulers.boundedElastic；避免 block()；上下文传播使用 Context。  
- 面试提示：说明不适合 CPU 密集与少量并发典型场景。  

## 26. 序列化与 HttpMessageConverter [4]
- 是什么：请求/响应对象与 HTTP Body 的转换；基于内容协商确定媒体类型。  
- 为什么：支持多格式（JSON/XML/Protobuf）。  
- 怎么用：扩展 MappingJackson2HttpMessageConverter；自定义媒体类型。  
- 关键细节：Accept/Content-Type 处理；@JsonView 局部视图；性能优化（禁用默认多余特性）。  
- 面试提示：回答“为何大小写敏感问题导致绑定失败”。  

## 27. Spring Data JPA 实体生命周期与缓存 [4]
- 是什么：Transient → Managed(persistent) → Detached → Removed；一级缓存（Persistence Context）。  
- 为什么：理解脏检查与延迟加载时机。  
- 怎么用：事务边界包裹查询和修改；使用 fetch join 减少 N+1。  
- 关键细节：修改在事务提交前 flush；Detach 后变更不再自动同步；Lazy 初始化异常。  
- 面试提示：解释 N+1 与解决方案。  

## 28. JPA 乐观锁与并发冲突 [4]
- 是什么：@Version 字段标识版本，更新时检查；冲突抛 OptimisticLockException。  
- 为什么：避免过度行锁造成性能损失。  
- 怎么用：在聚合根加 @Version；冲突重试逻辑。  
- 关键细节：批量更新绕开版本检查风险；合并逻辑需谨慎。  
- 面试提示：区分悲观锁与乐观锁适用场景。  

## 29. MyBatis 动态 SQL 与可维护性 [3]
- 是什么：XML 标签 <if><choose><foreach> 构造条件；支持注解方式但复杂时推荐 XML。  
- 为什么：构建灵活查询避免字符串拼接。  
- 怎么用：合理拆分片段 <sql>；基础片段复用。  
- 关键细节：动态拼接空条件风险；参数 #{}/ ${} 安全差异（SQL 注入）。  
- 面试提示：强调“不要使用 ${} 连接用户输入”。  

## 30. MyBatis 插件拦截链 [3]
- 是什么：四大对象：Executor/StatementHandler/ParameterHandler/ResultSetHandler 可被拦截。  
- 为什么：实现分页、审计、加密解密等横切逻辑。  
- 怎么用：实现 Interceptor + @Intercepts + @Signature。  
- 关键细节：拦截过度影响性能；顺序与多插件兼容。  
- 面试提示：提供一个分页插件原理简述。  

## 31. Spring Cache 抽象 [4]
- 是什么：@Cacheable/@CacheEvict/@CachePut/@Caching；统一访问不同缓存实现。  
- 为什么：减少重复查询与计算；提升响应速度。  
- 怎么用：选择 key 生成策略；TTL 与失效同步外部缓存配置。  
- 关键细节：自调用失效；空值缓存策略；并发击穿预防（同步条件）。  
- 面试提示：解释 Cacheable + unless 条件。  

## 32. 事件机制与 @EventListener [3]
- 是什么：发布/订阅模型，ApplicationEvent 与监听器解耦。  
- 为什么：降低模块耦合、异步处理非关键路径。  
- 怎么用：ApplicationEventPublisher 发布；@EventListener 支持条件与异步(@Async)。  
- 关键细节：事务事件 AFTER_COMMIT；事件风暴风险；顺序可用 @Order。  
- 面试提示：指出“过度事件化导致逻辑分散”问题。  

## 33. @Async 异步与上下文传播 [4]
- 是什么：通过代理将方法提交线程池执行；返回 Future/CompletableFuture。  
- 为什么：释放调用线程用于并发处理。  
- 怎么用：配置 TaskExecutor；链路追踪需包装执行器。  
- 关键细节：不支持内部调用；异常在 Future 中；上下文 (Security/MDC) 丢失需装饰。  
- 面试提示：说明与消息队列异步方式区别。  

## 34. @Scheduled 定时任务与集群一致性 [3]
- 是什么：基于调度线程池按照 cron/fixedRate/fixedDelay 执行。  
- 为什么：周期任务（清理、统计）。  
- 怎么用：配置线程池大小；避免长任务阻塞。  
- 关键细节：集群重复执行需分布式锁或调度中心（Quartz/XXL）。  
- 面试提示：谈“幂等设计”与断点续执行。  

## 35. Profiles 与环境隔离 [3]
- 是什么：不同 profile 激活不同配置/Bean；application-{profile}.yaml。  
- 为什么：区分开发、测试、生产。  
- 怎么用：spring.profiles.active；复合激活；动态切换谨慎。  
- 关键细节：多 profile 覆盖顺序；敏感配置隔离。  
- 面试提示：说明误加载生产资源风险。  

## 36. Spring Security 过滤器链与认证流程 [5]
- 是什么：FilterChainProxy 管理顺序过滤器；UsernamePasswordAuthenticationFilter 等执行认证；最终 SecurityContext 存储用户信息。  
- 为什么：统一认证授权扩展点。  
- 怎么用：HttpSecurity DSL 配置；自定义 AuthenticationProvider；添加自定义 Filter。  
- 关键细节：顺序影响结果；OncePerRequestFilter 避免重复；异常翻译。  
- 面试提示：描述一次表单登录到成功响应的过滤器经过。  

## 37. 授权与方法级安全 [4]
- 是什么：基于角色/权限表达式控制访问；@PreAuthorize/@PostAuthorize。  
- 为什么：精细化权限控制；业务规则内聚。  
- 怎么用：SpEL 表达式访问参数与认证对象；全局开启方法安全。  
- 关键细节：延迟加载权限数据；表达式过度复杂减可维护性。  
- 面试提示：强调“方法安全不替代 URL 安全”。  

## 38. OAuth2 / JWT 与无状态安全 [4]
- 是什么：OAuth2 授权流程（授权码、客户端凭证、密码模式已不推荐、刷新令牌）；JWT 自包含令牌。  
- 为什么：支持分布式与扩展性认证。  
- 怎么用：Resource Server 校验 Token；JWT 加签与过期；RefreshToken 刷新。  
- 关键细节：JWT 不可撤销；需短 TTL + 黑名单策略；Scopes 设计。  
- 面试提示：比较 Session 与 JWT 优劣。  

## 39. 服务发现与注册 (Spring Cloud) [3]
- 是什么：Eureka/Consul/Nacos 管理实例状态；客户端从注册表拉取。  
- 为什么：动态扩缩容与故障剔除。  
- 怎么用：心跳续约；健康检查；配置超时与重试策略。  
- 关键细节：Eureka 自我保护；Consul KV 与健康分离。  
- 面试提示：说明“自我保护”意义。  

## 40. 配置中心与动态刷新 [3]
- 是什么：集中管理配置（Spring Cloud Config / Nacos），支持更新分发。  
- 为什么：减少重新部署成本；统一安全控制。  
- 怎么用：@RefreshScope Bean；总线广播刷新；灰度发布。  
- 关键细节：刷新造成 Bean 重新实例；连接池类不宜频繁刷新。  
- 面试提示：指出“刷新造成短暂性能波动”原因。  

## 41. Gateway 路由与过滤链 [3]
- 是什么：基于 Netty 与响应式，路由请求到后端服务并应用过滤器（前/后）。  
- 为什么：集中处理跨领域逻辑（鉴权、限流、监控、熔断）。  
- 怎么用：定义 RoutePredicateFactory；GlobalFilter 插入链。  
- 关键细节：限流基于令牌桶或 Redis；背压模型适配下游。  
- 面试提示：说明“与传统 Servlet 网关差异”。  

## 42. 声明式客户端 Feign [3]
- 是什么：接口 + 注解生成 HTTP 客户端；整合负载与熔断。  
- 为什么：减少样板代码与增强可读性。  
- 怎么用：开启 @EnableFeignClients；配置超时与重试；日志级别。  
- 关键细节：序列化开销；集中异常处理；线程池隔离（熔断器）。  
- 面试提示：对比 WebClient 与 Feign 场景。  

## 43. Resilience4j 容错组件 [3]
- 是什么：限流、熔断、重试、隔离、缓存、回退策略。  
- 为什么：提高整体鲁棒性与防雪崩。  
- 怎么用：@Retry/@CircuitBreaker 注解；配置阈值与滑动窗口。  
- 关键细节：状态转换半开 -> 闭合；统计窗口设置影响灵敏度。  
- 面试提示：解释“过度敏感导致抖动”。  

## 44. Starter 设计原则 [3]
- 是什么：约定封装自动装配 + 配置属性 + 依赖收敛。  
- 为什么：减少使用方重复配置。  
- 怎么用：AutoConfiguration + spring.factories/imports；提供默认 Bean 可被覆盖。  
- 关键细节：避免业务逻辑；拆分 API 与 Impl；暴露配置元数据。  
- 面试提示：列出“如何允许用户覆盖你的 Bean”。  

## 45. 启动性能优化与懒加载 [4]
- 是什么：减少启动扫描与实例化耗时；延迟创建非关键 Bean。  
- 为什么：提升冷启动效率，支持弹性扩缩。  
- 怎么用：剔除无关 starter；spring.main.lazy-initialization=true（谨慎）；分析启动报告。  
- 关键细节：懒加载可能在首请求引入延迟；Bean 数量与反射扫描占比。  
- 面试提示：区分“启动时间”与“首请求延迟”。  

## 46. 常见事务失效与排查清单 [5]
- 是什么：自调用、private 方法、final 类、异常吞掉、未开启代理、不同线程、非公共访问。  
- 为什么：避免错误假设导致数据不一致。  
- 怎么用：重构调用链；显式抛异常；使用代理暴露；检查配置@EnableTransactionManagement。  
- 关键细节：异步线程需显式管理事务；测试中使用 @Transactional 导致懒加载错位。  
- 面试提示：列出清单并逐项解释。  

## 47. 循环依赖重构策略 [4]
- 是什么：通过拆分服务、引入接口、事件驱动、延迟注入解决构造环。  
- 为什么：降低耦合并改善可测试性。  
- 怎么用：分离职责；@Lazy 延迟依赖；使用 setter 注入。  
- 关键细节：复杂环多源于跨聚合领域服务；事件化需防止分散逻辑。  
- 面试提示：给一个真实重构案例。  

## 48. 线程安全与并发在 Spring 中的注意点 [4]
- 是什么：单例 Bean 多线程访问；线程池执行异步任务；请求作用域隔离。  
- 为什么：防止共享可变状态导致数据竞争。  
- 怎么用：无状态设计；使用 ThreadLocal 注意清理；配置独立 TaskExecutor。  
- 关键细节：Controller 单例；原型注入线程安全误解。  
- 面试提示：回答“为什么不要在单例保存可变集合无需同步”。  

## 49. Test Slice 与分层测试策略 [3]
- 是什么：@WebMvcTest / @DataJpaTest / @RestClientTest 限制上下文加载；@SpringBootTest 全量。  
- 为什么：减少测试执行时间与隔离问题。  
- 怎么用：根据层选择切片；对外部依赖用 Testcontainers；Mock Beans。  
- 关键细节：切片默认不加载 Security；引入额外配置需显式导入。  
- 面试提示：区分单测、集成测试与端到端测试。  

## 50. Testcontainers 与环境一致性 [3]
- 是什么：以容器方式拉起真实依赖（DB、MQ、Cache）。  
- 为什么：避免“测试通过/生产失败”差异。  
- 怎么用：动态端口绑定；抽象公共基类初始化。  
- 关键细节：启动成本；缓存镜像优化。  
- 面试提示：说明比内存数据库更可靠的原因。  

## 51. 模块拆分与巨大上下文控制 [3]
- 是什么：按领域/层分模块减少单上下文 Bean 冲突与启动时间。  
- 为什么：提高边界清晰度与可维护性。  
- 怎么用：多模块 Maven/Gradle；合并核心共享模块。  
- 关键细节：循环依赖跨模块更难发现；公共模块膨胀风险。  
- 面试提示：给出“从单体到多模块”分阶段策略。  

## 52. 常见性能陷阱（扫描/反射/序列化） [4]
- 是什么：过多组件扫描、频繁反射创建、JSON 序列化深层对象、N+1 查询。  
- 为什么：增加启动、CPU 与 IO 成本。  
- 怎么用：限制扫描路径；缓存反射元数据；序列化白名单；fetch join。  
- 关键细节：Jackson 循环引用处理；Hibernate batch fetch。  
- 面试提示：描述一次性能剖析 → 发现 N+1 → fetch join 修复案例。  

## 53. 常见排错思路（启动失败/Bean 覆盖/端点异常） [4]
- 是什么：收集日志 → 定位阶段 → 验证配置 → 最小化复现。  
- 为什么：提升定位速度避免盲目尝试。  
- 怎么用：--debug / ConditionEvaluationReport / 环境差异定位。  
- 关键细节：Bean 覆盖日志 WARN；端口冲突异常；Profile 混用。  
- 面试提示：展示一个 5 步排错流程。  

## 54. 可观测性：追踪 + 日志 + 指标整合 [4]
- 是什么：统一 TraceId/SpanId + 结构化日志 + 指标关联合并。  
- 为什么：减少跨系统排查成本。  
- 怎么用：MDC 注入追踪上下文；日志平台检索关联；指标异常关联 Trace 采样。  
- 关键细节：异步丢失上下文需包装；高采样率耗资源。  
- 面试提示：说明如何“从一个高延迟指标跳到具体调用链”。  

## 55. 链路追踪上下游传播原理 [3]
- 是什么：HTTP Header（B3、W3C traceparent）携带上下文；组件提取与继续传播。  
- 为什么：多服务调用串联定位延迟与故障。  
- 怎么用：注册拦截器/Filter 注入与提取；异步线程包装 Runnable/Callable。  
- 关键细节：跨线程丢失；采样策略。  
- 面试提示：区分日志中的 TraceId 与 SpanId 含义。  

## 56. 限流与熔断在 Spring 组合策略 [3]
- 是什么：限流控制瞬时并发/速率；熔断快速失败避免雪崩。  
- 为什么：保护下游与自身稳定。  
- 怎么用：Gateway + Redis 令牌桶；Resilience4j 熔断与重试。  
- 关键细节：错误率窗口统计；过度重试放大压力。  
- 面试提示：分析一个“缓存失效 + 下游抖动”综合场景。  

## 57. 分布式事务与补偿思路 [3]
- 是什么：跨服务数据一致性：2PC、TCC、消息最终一致。  
- 为什么：保证业务逻辑在多资源操作中可靠。  
- 怎么用：事件驱动 + Outbox；TCC 预留/确认/取消。  
- 关键细节：幂等与重试；局部失败补偿。  
- 面试提示：解释为什么避免通用 2PC。  

## 58. 错误处理与重试策略在异步任务中 [3]
- 是什么：任务失败重试、指数退避、死信队列。  
- 为什么：提高可靠性并缓解瞬时故障。  
- 怎么用：@Retry + Recovery；消息框架死信处理。  
- 关键细节：避免无限重试占用资源；幂等设计。  
- 面试提示：说明一个支付重试场景处理。  

## 59. 安全加固与敏感信息管理 [3]
- 是什么：配置脱敏、密钥管理（Vault/KMS）、避免日志泄漏。  
- 为什么：防止信息暴露风险。  
- 怎么用：集中配置加密；过滤器屏蔽参数。  
- 关键细节：避免将 secret 放在镜像层；热加载策略。  
- 面试提示：给出一次“敏感配置泄漏”防范措施。  

## 60. 重要度汇总（简版）
- 核心 IoC/生命周期/循环依赖/事务传播隔离/AOP/ MVC/WebFlux/指标与安全：[5]
- 自动装配/配置属性/Actuator/性能调优/安全过滤链/缓存/诊断工具/并发注意：[4]
- MyBatis/事件/异步/Scheduling/Profiles/Cloud 组件/Starter/测试切片/追踪/模块化/限流熔断：[3]
- 分布式事务/Native 集成/安全加固 等：[2]

---

## 常见面试提示汇总（示例思路短句）
- 生命周期：能列阶段并说明增强点。  
- 事务失效：自调用、异常处理、访问级别。  
- AOP：代理类型选择与自调用绕过。  
- MVC：请求链与扩展点。  
- WebFlux：适用 IO 场景与阻塞适配。  
- 缓存：击穿/穿透/雪崩策略。  
- 指标：避免高基数标签。  
- 安全：过滤器链顺序与 JWT 风险。  
- 性能：N+1、过度反射、启动分析。  
- 诊断：ConditionEvaluationReport 使用。  

(完)