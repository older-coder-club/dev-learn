# Spring 生态面试指南（按领域重组与分级编号）

评分说明：5=必须掌握，4=很重要，3=熟练更佳，2=了解即可，1=加分项  
重组说明：现采用层级编号：一级=领域（1..12），二级=具体知识点（如 1.1、2.3）。保留原“线性编号(1..60)”与“原始编号(重组前)”的映射，便于迁移与交叉引用。

---

## 1 容器与装配

### 1.1 BeanDefinition 与 IoC 容器结构 [5]
是什么：BeanDefinition 描述 Bean 的元信息（class、scope、lazy、依赖、初始化与销毁方法、属性等）；IoC 容器（BeanFactory/ApplicationContext）基于这些定义进行实例化与生命周期管理。  
为什么：将对象创建与依赖管理外置，支持配置与扩展点（后处理器、条件化装配）。  
怎么用：通过 @Configuration + @Bean / @ComponentScan / Import 注册；使用 BeanFactoryPostProcessor 修改定义；调试可查看 BeanDefinitionNames。  
关键细节：BeanDefinition 合并（子定义覆盖父定义）；FactoryBean 与普通 Bean 的区分（getBean("xxx") vs &xxx）；别名与 primary 冲突处理。  
常见陷阱：包扫描范围过大导致重复定义；误把 &name 当产品对象；@Primary 与 @Qualifier 冲突未消解。  
面试提示：说明 ApplicationContext 比 BeanFactory 多了哪些企业特性（国际化、事件、资源加载、自动装配）。  
案例：多实现注入使用 @Qualifier 指定；获取 FactoryBean 本体用 &beanName；限制扫描包避免重复。  
速记方式：定义管“元”，工厂管“生”，上下文管“用”。  

### 1.2 ApplicationContext Refresh 流程关键阶段 [5]
是什么：refresh() 包含：准备环境 → 加载 Bean 定义 → BeanFactory 后处理 → 注册 BeanPostProcessor → 实例化单例 → 初始化事件与监听器 → 完成。  
为什么：理解扩展钩子所在位置，便于注入自定义逻辑。  
怎么用：在 BeanFactoryPostProcessor 阶段修改定义；在 BeanPostProcessor 中增强实例；ContextRefreshedEvent 监听。  
关键细节：单例预实例化发生在注册完所有 BeanPostProcessor 后；父子容器 Bean 查找逻辑；早期单例暴露。  
常见陷阱：在实例化后才试图改定义无效；父子容器覆盖顺序误判；未注册自定义 BPP 导致增强缺失。  
面试提示：列出顺序并指出某阶段可干预点；解释为何某些增强必须是 BeanPostProcessor。  
案例：在 BeanFactoryPostProcessor 中动态修改数据源属性；在 BeanPostProcessor 中包装 AOP 代理。  
速记方式：备环境→载定义→改工厂→挂BPP→建单例→起事件。  

### 1.3 Bean 生命周期详解 [5]
是什么：加载定义 → 实例化（构造或工厂方法） → 属性填充 → Aware 回调 → BeanPostProcessor.before → 初始化（@PostConstruct / InitializingBean / init-method）→ BeanPostProcessor.after → 可用 → 销毁（@PreDestroy / DisposableBean / destroy-method）。  
为什么：在合适阶段插入校验、代理、资源初始化与释放。  
怎么用：实现 InitializingBean 做运行时检查；通过 BeanPostProcessor 包装代理（例如 AOP）；使用 destroy-method 释放连接。  
关键细节：顺序冲突处理（多个初始化回调都执行）；提前引用导致尚未初始化的 Bean 被其他 Bean 使用；原型作用域不托管销毁。  
常见陷阱：在 @PostConstruct 之后再改属性无效；原型 Bean 资源未释放；提前暴露导致未初始化对象被使用。  
面试提示：画出生命周期图并指出在哪些点“无法”再改变属性。  
案例：使用 destroy-method 关闭连接池；用 BeanPostProcessor 创建代理封装监控。  
速记方式：构造→填充→感知→前置→初始化→后置→使用→销毁。  

### 1.4 BeanPostProcessor 与 BeanFactoryPostProcessor 区别 [5]
是什么：前者处理 Bean“实例”级别，后者处理 BeanDefinition“元数据”级别。  
为什么：分离“定义变更”与“实例增强”，降低耦合与风险。  
怎么用：自定义属性占位符、YAML 解析属于工厂后处理；添加代理、校验逻辑属于 BeanPostProcessor。  
关键细节：执行时机：BeanFactoryPostProcessor 在单例实例化前；Ordered/PriorityOrdered 决定顺序；重复注册风险。  
常见陷阱：在 BeanPostProcessor 中修改 BeanDefinition 不生效；忽略顺序导致链路覆写；注册为普通 Bean 导致时机过晚。  
面试提示：回答“为什么修改类名不能在 BeanPostProcessor 中做”。  
案例：使用 PropertySourcesPlaceholderConfigurer 解析占位符；自定义 BPP 对敏感 Bean 做包装校验。  
速记方式：FactoryPP 改“定义”，BeanPP 改“实例”。  

### 1.5 三级缓存与循环依赖解决机制 [5]
是什么：singletonFactories（ObjectFactory 生成早期代理）、earlySingletonObjects（早期暴露实例）、singletonObjects（最终单例）。  
为什么：在属性注入阶段允许引用彼此但避免未完成初始化的重复创建。  
怎么用：避免构造器循环（无法通过此机制解决）；改用 Setter 或 @Lazy；合理拆分职责。  
关键细节：AOP 早期暴露代理；原型作用域不支持自动循环解决；构造器注入环最终报异常。  
常见陷阱：构造器注入形成环无法打破；原型循环依赖不被容器处理；过早暴露半初始化对象。  
面试提示：描述三级缓存数据流并说明“为什么构造器环失败”。  
案例：将构造器依赖改为 setter 或 @Lazy；提取接口打断依赖方向。  
速记方式：工厂缓存→早期缓存→单例缓存，构造环必失败。  

### 1.6 作用域与线程上下文 [4]
是什么：singleton / prototype / request / session / application / websocket / 自定义（如 ThreadLocalScope）。  
为什么：匹配对象生命周期与资源使用模式。  
怎么用：@Scope + proxyMode (ScopedProxy) 解决在单例中注入 request scoped Bean 问题。  
关键细节：prototype 不自动销毁；会话作用域需容器支持；代理模式避免过早创建。  
常见陷阱：将 request 范围对象直接注入单例而未启用代理；prototype 资源泄漏；ThreadLocal 未清理。  
面试提示：说明一个 request Bean 注入 Controller 的代理工作原理。  
案例：使用 ScopedProxyMode.TARGET_CLASS 注入 RequestScoped 服务；自定义 ThreadLocalScope。  
速记方式：作用域匹配生命周期，跨域用代理。  

### 1.7 条件化装配与 Profile [4]
是什么：@ConditionalX 系列（OnClass/OnMissingBean/OnProperty/OnExpression）、@Profile 控制环境差异。  
为什么：减少硬编码分支与复杂环境 if-else。  
怎么用：配置不同数据源；根据属性启用可选模块；Profile 与 Conditional 可组合。  
关键细节：条件评估在 BeanDefinition 阶段；顺序影响最终结果；@ConditionalOnMissingBean 常与用户自定义冲突防止覆盖。  
常见陷阱：条件顺序错误导致默认 Bean 抢先；Profile 混用引发意外装配；条件过宽误匹配。  
面试提示：给出“用户自定义覆盖默认自动装配”示例。  
案例：@ConditionalOnMissingBean + @Primary 让用户实现优先；基于 @Profile 切换不同数据源。  
速记方式：条件定装配，Profile 切环境。  

### 1.8 属性绑定与 @ConfigurationProperties [5]
是什么：将外部化配置（properties/yaml/环境变量）绑定到 POJO；支持松散命名与校验。  
为什么：集中配置管理与类型安全；避免散落 @Value。  
怎么用：@ConfigurationProperties + @EnableConfigurationProperties 或扫描；引入 @Validated 强制校验。  
关键细节：与 @Value 区别（批量 vs 零散）；嵌套对象需配置前缀；ConstructorBinding 在只读场景。  
常见陷阱：松散绑定命名不一致；缺少 @Validated 导致空值上线；@Value 与绑定类混用造成维护困难。  
面试提示：讲“从散乱 @Value 到集中配置类”的演进。  
案例：使用 @ConfigurationProperties + @Validated + 构造绑定实现只读安全配置。  
速记方式：外部化→绑定→校验→可测试。  

### 1.9 FactoryBean 与普通 Bean 区分 [4]
是什么：特殊 Bean：getObject() 返回实际产品对象；容器内注册名称 vs &name 获取工厂本身。  
为什么：支持复杂初始化（代理、动态创建）屏蔽调用方细节。  
怎么用：MyBatis-SqlSessionFactoryBean 等场景；自定义 FactoryBean 生成远程代理。  
关键细节：作用域与生命周期遵循产品对象行为；循环依赖处理差别。  
常见陷阱：误注入工厂本体；多实例作用域引发不可预期共享；与 AOP 代理叠加导致类型判断偏差。  
面试提示：问“如何拿到 FactoryBean 本体”回答 & + beanName。  
案例：自定义 FactoryBean 创建远程代理对象，容器注入为产品对象。  
速记方式：name→产品，&name→工厂。  

### 1.10 ConversionService 与类型转换体系 [4]
是什么：统一类型转换与格式化框架；Converter / GenericConverter / Formatter。  
为什么：降低控制器层参数与属性注入转换样板。  
怎么用：注册自定义 Converter；使用 @DateTimeFormat；全局配置 FormattingConversionService。  
关键细节：优先级：自定义 > 默认；Formatter 面向打印与解析；转换失败抛异常进入 BindingResult。  
常见陷阱：遗漏注册全局转换导致参数解析失败；混用 Formatter 与 Converter；忽略 BindingResult 错误处理。  
面试提示：说明为什么推荐集中注册转换。  
案例：实现 String→Enum Converter 并通过 WebMvcConfigurer 全局注册。  
速记方式：先转后绑，错进 BindingResult。  

### 1.11 校验与 JSR-303 集成 [4]
是什么：Bean Validation：@NotNull/@Size 等注解 + Validator；Spring 接入参数与对象校验。  
为什么：将约束与模型共存保持一致性。  
怎么用：方法参数加 @Valid/@Validated；BindingResult 捕获错误；分组校验。  
关键细节：嵌套对象需级联 @Valid；默认仅对控制器参数自动触发。  
常见陷阱：未开启方法级校验；漏加级联 @Valid；分组没指定导致规则未生效。  
面试提示：说明 @Validated 与 @Valid 在分组场景的区别。  
案例：控制器参数使用 @Validated(GroupA) + BindingResult 捕获并返回统一错误。  
速记方式：注解在对象，错误进 BindingResult。  

### 1.12 Starter 设计原则 [3]
是什么：约定封装自动装配 + 配置属性 + 依赖收敛。  
为什么：减少使用方重复配置。  
怎么用：AutoConfiguration + spring.factories/imports；提供默认 Bean 可被覆盖。  
关键细节：避免业务逻辑；拆分 API 与 Impl；暴露配置元数据。  
常见陷阱：强耦合业务逻辑；未提供覆盖点；未生成配置元数据导致提示缺失。  
面试提示：列出“如何允许用户覆盖你的 Bean”。  
案例：AutoConfiguration + @ConditionalOnMissingBean + Properties 元数据提供 IDE 提示。  
速记方式：封装默认，留可扩，文档清。  

### 1.13 循环依赖重构策略 [4]
是什么：通过拆分服务、引入接口、事件驱动、延迟注入解决构造环。  
为什么：降低耦合并改善可测试性。  
怎么用：分离职责；@Lazy 延迟依赖；使用 setter 注入。  
关键细节：复杂环多源于跨聚合领域服务；事件化需防止分散逻辑。  
常见陷阱：@Lazy 只缓解不治本；事件泛滥难以追踪；接口过度拆分反增复杂度。  
面试提示：给一个真实重构案例。  
案例：拆出领域服务B，A 通过发布领域事件驱动 B，去除直接依赖。  
速记方式：拆模块、断方向、用事件。  

### 1.14 Profiles 与环境隔离 [3]
是什么：不同 profile 激活不同配置/Bean；application-{profile}.yaml。  
为什么：区分开发、测试、生产。  
怎么用：spring.profiles.active；复合激活；动态切换谨慎。  
关键细节：多 profile 覆盖顺序；敏感配置隔离。  
常见陷阱：激活顺序误判覆盖生产配置；测试与生产资源混用；密钥进入镜像。  
面试提示：说明误加载生产资源风险。  
案例：application-prod.yml 独立且默认不加载，CI 校验主动拒绝 prod 资源在非 prod 使用。  
速记方式：多环境隔离，显式激活。  

## 2 AOP 与事务

### 2.1 Spring AOP 代理模型与织入点 [5]
是什么：基于代理（JDK 动态或 CGLIB）在运行时增强方法调用；非编译时织入。  
为什么：横切关注点（事务、日志、安全）与业务解耦。  
怎么用：@EnableAspectJAutoProxy + @Aspect；定义切点表达式 + 通知。  
关键细节：final 方法不可增强；自调用绕过代理；顺序通过 @Order 控制；目标类多接口时默认 JDK。  
常见陷阱：在同类内部调用导致 AOP 不生效；final/private 方法无法被代理；错误选择 JDK/CGLIB 影响代理行为。  
面试提示：回答“为什么事务在同类内部调用失效”。  
案例：将内部调用抽到同包服务，通过代理调用生效；使用 CGLIB 代理需要类非 final。  
速记方式：代理拦方法，自调不走代理。  

### 2.2 切点表达式类型与组合 [4]
是什么：execution、within、this、target、args、@annotation、@within、@target 等。  
为什么：灵活选择增强范围，避免误匹配。  
怎么用：组合 && / || / !；限定包与方法签名。  
关键细节：this/target 区分代理类型与实际目标；@annotation 捕获具体方法注解。  
常见陷阱：execution 表达式过宽误伤；包名通配写错不生效；args 与 @annotation 搭配时顺序、位置错误。  
面试提示：举一个“仅增强标注 @Transactional 方法”的表达式。  
案例：@annotation(org.springframework.transaction.annotation.Transactional) 仅拦截事务方法进行日志埋点。  
速记方式：点到为止，窄而准。  

### 2.3 @Transactional 原理与代理增强 [5]
是什么：基于代理拦截公共方法，在调用前后由事务管理器获取/提交/回滚资源。  
为什么：屏蔽底层 JDBC/JPA 事务处理样板。  
怎么用：标注在类或方法；配置 PlatformTransactionManager；设置传播与隔离。  
关键细节：仅 public 方法有效；自调用失效；异常被捕获不回滚；非 RuntimeException 需 rollbackFor。  
常见陷阱：捕获异常后未重新抛出；方法非 public；不同线程执行未传播事务上下文。  
面试提示：列出至少五个失效场景。  
案例：在 Service 层抛出自定义异常并配置 rollbackFor，控制器层统一处理。  
速记方式：公有+代理+异常外抛才回滚。  

### 2.4 事务传播行为详解 [5]
是什么：REQUIRED / REQUIRES_NEW / SUPPORTS / MANDATORY / NOT_SUPPORTED / NEVER / NESTED。  
为什么：控制嵌套调用事务边界与资源管理。  
怎么用：子方法需独立回滚用 REQUIRES_NEW；多层调用共享事务用 REQUIRED。  
关键细节：NESTED 依赖保存点；REQUIRES_NEW 暂停外部事务；NOT_SUPPORTED 挂起处理。  
常见陷阱：误用 REQUIRED 导致整体回滚；REQUIRES_NEW 事务过多导致资源耗尽；NESTED 未被底层实现支持。  
面试提示：给出外层成功内层失败需独立回滚场景选择。  
案例：审核通过主流程 REQUIRED，短信通知使用 REQUIRES_NEW，失败不影响主事务。  
速记方式：要独立就 NEW，要局部就 NESTED。  

### 2.5 隔离级别与数据库实现差异 [4]
是什么：READ_UNCOMMITTED/READ_COMMITTED/REPEATABLE_READ/SERIALIZABLE 及幻读/不可重复读/脏读概念。  
为什么：限制并发读写异常影响正确性。  
怎么用：@Transactional(isolation=...)；依据数据库默认级别调整。  
关键细节：部分数据库不支持全部级别；Serializable 性能低。  
常见陷阱：在 MySQL RR 下误以为无幻读；错误选择级别造成性能劣化；忽视索引导致锁范围扩大。  
面试提示：说明幻读在 MVCC 下的处理方式。  
案例：在高并发写少读多场景用 RC；金融强一致短事务用 Serializable 并限流。  
速记方式：级别越高越慢，按需选。  

### 2.6 readOnly 与事务优化 [3]
是什么：向底层标识事务仅执行查询；可能跳过脏检查。  
为什么：减少不必要 flush。  
怎么用：@Transactional(readOnly=true)；与 ORM flush 模式结合。  
关键细节：不保证绝对写阻止；驱动优化取决实现。  
常见陷阱：以为设置 readOnly 就禁止写；忽略 ORM flush 行为；二级缓存产生脏读误判。  
面试提示：“readOnly 不等于只读数据库”。  
案例：查询场景统一标注 readOnly=true 并下调连接池校验。  
速记方式：只“倾向”只读，不“强制”只写。  

### 2.7 常见事务失效与排查清单 [5]
是什么：自调用、private、final 类、异常吞掉、未启代理、不同线程、非 public。  
为什么：避免错误假设导致数据不一致。  
怎么用：重构调用链；显式抛异常；启用@EnableTransactionManagement。  
关键细节：异步线程需显式事务；测试 @Transactional 懒加载错位。  
常见陷阱：异常被吞；不同类同名方法被代理错过；代理链顺序导致 @Transactional 被覆盖。  
面试提示：清单逐项解释。  
案例：将自调用重构为外部 Bean；异常不捕获直接抛出，由全局异常器处理。  
速记方式：别自调、抛异常、跨线程显式管。  

### 2.8 分布式事务与补偿思路 [3]
是什么：跨服务一致性：2PC、TCC、消息最终一致。  
为什么：保障多资源操作可靠。  
怎么用：Outbox + 事件；TCC 预留/确认/取消。  
关键细节：幂等与重试；局部失败补偿。  
常见陷阱：幂等键设计不当；补偿缺少反向操作；消息乱序未处理。  
面试提示：为何避免通用 2PC。  
案例：Outbox + 事件驱动实现订单与库存最终一致。  
速记方式：先本地，再消息，靠幂等补偿。  

## 3 Web MVC 与 WebFlux

### 3.1 Spring MVC 调用链核心组件 [5]
是什么：DispatcherServlet → HandlerMapping → HandlerAdapter → HandlerMethod → 参数解析 → 返回值处理 → 异常解析。  
为什么：清晰分层方便扩展。  
怎么用：自定义 ArgumentResolver；@ControllerAdvice 管异常。  
关键细节：消息转换器顺序；异步 DeferredResult/WebAsyncManager。  
常见陷阱：异常未被 @ControllerAdvice 捕捉；返回值处理器不生效；字符集/内容协商误配。  
面试提示：一次请求主要扩展点。  
案例：自定义 HandlerMethodArgumentResolver 绑定登录用户。  
速记方式：找 Dispatcher，挂 Mapping/Adapter/Resolver。  

### 3.2 参数绑定与类型转换流程 [4]
是什么：WebDataBinder 绑定请求参数到对象。  
为什么：减少解析与校验样板。  
怎么用：自定义 Converter/Formatter；@InitBinder 局部定制。  
关键细节：BindingResult；日期转换失败；忽略字段策略。  
常见陷阱：直接绑定实体导致越权；未校验导致 NPE；字符串修剪缺失引发误差。  
面试提示：DTO 优于直接实体。  
案例：创建 UserCreateDTO + @Valid + BindingResult 返回错误码。  
速记方式：DTO 承接，Binder 绑定。  

### 3.3 异常处理与统一响应 [4]
是什么：@ControllerAdvice + @ExceptionHandler 统一响应；ResponseStatusException 设置状态。  
为什么：提升一致性。  
怎么用：全局异常映射；日志与 TraceId。  
关键细节：异常链；Resolver 顺序。  
常见陷阱：在控制器捕获并吞掉异常；多 Resolver 顺序混乱；状态码与业务码混淆。  
面试提示：不要在 Controller 吞异常。  
案例：全局异常映射到统一响应体并带 TraceId。  
速记方式：Advice 兜底，异常上浮。  

### 3.4 Filter vs Interceptor vs AOP 区别 [4]
是什么：Filter=Servlet 层；Interceptor=MVC 层；AOP=代理方法级。  
为什么：针对不同关注点选层。  
怎么用：安全/跨域 Filter；登录统计 Interceptor；事务/日志 AOP。  
关键细节：执行顺序；异步差异。  
常见陷阱：把鉴权放在 Interceptor 导致静态资源漏管；在 Filter 中做业务逻辑；AOP 想处理非 Bean。  
面试提示：链路追踪分层组合。  
案例：CORS 与安全在 Filter；鉴权在 Interceptor；审计在 AOP。  
速记方式：Filter 先，Interceptor 中，AOP 最后。  

### 3.5 WebFlux 响应式模型 [4]
是什么：Reactor Mono/Flux 非阻塞；默认 Netty。  
为什么：高并发 IO 等待。  
怎么用：返回 Mono/Flux；WebClient 异步；背压保护。  
关键细节：阻塞需 boundedElastic；避免 block()；上下文 Context。  
常见陷阱：在 event-loop 中阻塞；忽略 backpressure；上下文丢失。  
面试提示：不适合 CPU 密集少并发。  
案例：外部调用使用 WebClient 并切换到 boundedElastic 适配阻塞驱动。  
速记方式：少阻塞，守背压，用 Context。  

### 3.6 序列化 与 HttpMessageConverter [4]
是什么：请求/响应对象与 HTTP Body 转换；内容协商。  
为什么：支持多格式。  
怎么用：扩展 Jackson 转换器；自定义媒体类型。  
关键细节：Accept/Content-Type；@JsonView；性能优化。  
常见陷阱：忽略字符集导致中文乱码；忽略日期格式；大对象序列化性能差。  
面试提示：大小写敏感导致绑定失败。  
案例：注册 Kotlin/JavaTime 模块优化时间序列化；统一 ObjectMapper。  
速记方式：媒体类型先谈好，转换器再上场。  

## 4 数据访问与缓存

### 4.1 Spring Data JPA 实体生命周期与缓存 [4]
是什么：Transient→Managed→Detached→Removed；一级缓存。  
为什么：理解脏检查与延迟加载。  
怎么用：事务包裹查询修改；fetch join 减少 N+1。  
关键细节：提交前 flush；Detach 不同步；Lazy 初始化异常。  
常见陷阱：事务外访问懒加载抛 LazyInitializationException；错误的缓存意识；N+1 未被察觉。  
面试提示：N+1 解决。  
案例：使用 fetch join 或 @EntityGraph 预取，事务内访问懒加载属性。  
速记方式：瞬态→托管→脱管→删除，懒属性要在事务内。  

### 4.2 JPA 乐观锁与并发冲突 [4]
是什么：@Version 更新检查；冲突抛异常。  
为什么：避免过度行锁。  
怎么用：聚合根加 @Version；冲突重试。  
关键细节：批量更新绕过版本；合并谨慎。  
常见陷阱：版本字段未更新；批量 JPQL 更新跳过版本检查；高冲突场景仍用乐观锁。  
面试提示：悲观 vs 乐观锁。  
案例：订单更新冲突捕获 OptimisticLockException 后重试 N 次上限。  
速记方式：读多写少用乐观，冲突重试要限次。  

### 4.3 MyBatis 动态 SQL 与可维护性 [3]
是什么：XML 标签构造条件。  
为什么：灵活查询避免拼接。  
怎么用：<sql> 片段复用。  
关键细节：${} 注入风险；#{} 安全。  
常见陷阱：条件拼接遗漏导致全表扫描；<where> 未处理前导 AND；${} 引入注入风险。  
面试提示：禁止用户输入拼接 ${}。  
案例：使用 <where><if> 组合生成安全条件与动态排序白名单。  
速记方式：#{} 安全，${} 慎用，<where> 自动去前缀。  

### 4.4 MyBatis 插件拦截链 [3]
是什么：Executor/StatementHandler/ParameterHandler/ResultSetHandler 拦截。  
为什么：分页审计加密等。  
怎么用：Interceptor + @Signature。  
关键细节：过度拦截性能；顺序兼容。  
常见陷阱：多插件顺序冲突；拦截 ResultSetHandler 影响全局性能；线程安全问题。  
面试提示：分页插件原理。  
案例：分页插件置于 SQL 重写前，审计插件置于其后，明确顺序。  
速记方式：四点位拦截，少而精、顺序定。  

### 4.5 Spring Cache 抽象 [4]
是什么：@Cacheable/@Evict/@Put 统一缓存访问。  
为什么：减少重复查询。  
怎么用：key 策略；TTL 对齐外部缓存。  
关键细节：自调用失效；空值缓存；击穿预防。  
常见陷阱：key 生成不稳定；未缓存空值导致穿透；@CacheEvict 条件不当导致脏数据。  
面试提示：Cacheable + unless。  
案例：@Cacheable(key="#id", unless="#result==null") + @CacheEvict(on update)。  
速记方式：读走 Cacheable，写用 Evict/Put，key 要稳定。  

## 5 异步、事件与调度

### 5.1 事件机制与 @EventListener [3]
是什么：发布/订阅模型。  
为什么：降低耦合异步处理。  
怎么用：publisher 发布；@EventListener 支持条件与 @Async。  
关键细节：事务 AFTER_COMMIT；事件风暴风险。  
常见陷阱：在回滚事务中发布事件；同步事件阻塞主流程；监听器无序导致竞态。  
面试提示：过度事件化分散逻辑。  
案例：领域事件在 AFTER_COMMIT 发送，异步监听落库/推送。  
速记方式：发布-监听-（可异步），与事务对齐。  

### 5.2 @Async 异步与上下文传播 [4]
是什么：代理提交线程池，返回 Future/CompletableFuture。  
为什么：释放调用线程。  
怎么用：配置 TaskExecutor；包装传播追踪上下文。  
关键细节：内部调用失效；异常在 Future；上下文需装饰。  
常见陷阱：线程池饱和未监控；MDC/安全上下文丢失；绕过代理自调用。  
面试提示：与消息队列异步区别。  
案例：包装 TaskDecorator 传递 MDC，统一捕获异常记录指标。  
速记方式：代理发起+池可观测+上下文要传。  

### 5.3 @Scheduled 定时任务与集群一致性 [3]
是什么：线程池按 cron/fixedRate/fixedDelay 调度。  
为什么：周期任务。  
怎么用：配置线程池大小；避免长任务阻塞。  
关键细节：集群重复执行需分布式锁或调度中心。  
常见陷阱：长任务阻塞调度线程；时钟漂移导致错过触发；非幂等任务反复执行。  
面试提示：幂等与断点续执行。  
案例：使用 ShedLock/Redis 锁避免集群重入，长任务分片。  
速记方式：小而快，需加锁，能续跑。  

### 5.4 线程安全与并发注意点 [4]
是什么：单例 Bean 多线程访问；线程池异步；请求作用域隔离。  
为什么：防止数据竞争。  
怎么用：无状态设计；ThreadLocal 清理；独立 TaskExecutor。  
关键细节：Controller 单例；原型线程安全误解。  
常见陷阱：单例持有可变全局状态；ThreadLocal 未清理泄漏；非线程安全集合在并发写。  
面试提示：单例中可变集合风险。  
案例：服务无状态化，必要状态放请求作用域或用并发集合。  
速记方式：单例无状态，状态限定域。  

### 5.5 错误处理与重试策略（异步任务） [3]
是什么：失败重试、指数退避、死信队列。  
为什么：提高可靠性。  
怎么用：@Retry + Recovery；死信处理。  
关键细节：避免无限重试；幂等设计。  
常见陷阱：固定间隔放大雪崩；无熔断叠加压力；重试未记录导致排障难。  
面试提示：支付重试场景。  
案例：指数退避 + 最大次数 + 死信队列；结合幂等键。  
速记方式：短频少，退一步，留痕迹。  

## 6 安全

### 6.1 Spring Security 过滤器链与认证流程 [5]
是什么：FilterChainProxy 管理过滤器；认证后 SecurityContext 保存。  
为什么：统一认证授权扩展。  
怎么用：HttpSecurity DSL；自定义 AuthenticationProvider；添加 Filter。  
关键细节：顺序；OncePerRequestFilter；异常翻译。  
常见陷阱：放行规则过宽；异常未被 ExceptionTranslationFilter 处理；跨域与安全顺序不当。  
面试提示：表单登录过滤器链。  
案例：自定义 SecurityFilterChain 精确排列过滤器并最小授权。  
速记方式：链先行，序最要，错一环全失效。  

### 6.2 授权与方法级安全 [4]
是什么：角色/权限表达式；@PreAuthorize/@PostAuthorize。  
为什么：精细权限控制。  
怎么用：SpEL 访问参数与认证对象；开启方法安全。  
关键细节：权限延迟加载；表达式复杂度。  
常见陷阱：表达式空指针；权限缓存未失效；仅方法安全无 URL 安全。  
面试提示：方法安全不替代 URL 安全。  
案例：@PreAuthorize("#id == authentication.name or hasRole('ADMIN')") 做细粒度控制。  
速记方式：URL 把门，方法再验。  

### 6.3 OAuth2 / JWT 与无状态安全 [4]
是什么：OAuth2 授权流程；JWT 自包含令牌。  
为什么：分布式扩展认证。  
怎么用：Resource Server 校验；JWT 加签与过期；RefreshToken。  
关键细节：不可撤销；短 TTL + 黑名单；Scopes 设计。  
常见陷阱：长 TTL 泄露风险；密钥管理不当；无滚动刷新机制。  
面试提示：Session vs JWT。  
案例：短期 Access Token + 刷新令牌 + 黑名单退出。  
速记方式：短签名、勤轮换、严校验。  

### 6.4 安全加固与敏感信息管理 [3]
是什么：配置脱敏、密钥管理、日志防泄漏。  
为什么：防止信息暴露。  
怎么用：集中加密；过滤器屏蔽参数。  
关键细节：secret 不放镜像层；热加载策略。  
常见陷阱：日志打印敏感信息；错误回显；配置文件入库未加密。  
面试提示：敏感配置泄漏防范。  
案例：统一日志脱敏过滤；KMS/密管服务集中管理密钥。  
速记方式：少输出、多加密、零平铺。  

## 7 云原生与微服务

### 7.1 服务发现与注册 [3]
是什么：Eureka/Consul/Nacos 管实例；客户端拉取。  
为什么：动态扩缩容与故障剔除。  
怎么用：心跳续约；健康检查；超时重试。  
关键细节：Eureka 自我保护；Consul KV 与健康分离。  
常见陷阱：自我保护被误关导致雪崩；健康检查间隔过长；注册信息与真实状态不一致。  
面试提示：自我保护意义。  
案例：生产开启自我保护并配合短心跳与快速剔除策略。  
速记方式：注册-续约-剔除，保护抖动期。  

### 7.2 配置中心与动态刷新 [3]
是什么：集中管理配置并动态分发。  
为什么：减少重新部署成本。  
怎么用：@RefreshScope；总线广播；灰度发布。  
关键细节：刷新重新实例；连接池不宜频繁刷新。  
常见陷阱：全局 @RefreshScope 导致大面积重建；敏感配置明文下发；灰度范围过宽。  
面试提示：刷新性能波动原因。  
案例：仅对少量 Bean 使用 @RefreshScope，并将连接池配置排除在动态刷新之外。  
速记方式：集中管，少热更，慎刷新。  

### 7.3 Gateway 路由与过滤链 [3]
是什么：响应式路由与过滤器链。  
为什么：集中跨领域逻辑。  
怎么用：RoutePredicateFactory；GlobalFilter。  
关键细节：限流令牌桶/Redis；背压适配。  
常见陷阱：全局过滤器顺序不当；丢失背压导致 OOM；限流键设计不合理。  
面试提示：与 Servlet 网关差异。  
案例：基于 RedisRateLimiter 实现按用户限流并配合超时降级。  
速记方式：先路由，后过滤，守背压。  

### 7.4 声明式客户端 Feign [3]
是什么：接口+注解生成 HTTP 客户端。  
为什么：减少样板。  
怎么用：@EnableFeignClients；配置超时重试。  
关键细节：序列化开销；集中异常；隔离策略。  
常见陷阱：默认重试放大问题；超时设置过长；日志打印敏感头。  
面试提示：WebClient vs Feign。  
案例：配置连接/读取超时与重试策略，使用 ErrorDecoder 统一异常。  
速记方式：声明客户端，先限时，再容错。  

## 8 弹性与治理

### 8.1 Resilience4j 容错组件 [3]
是什么：限流、熔断、重试、隔离、缓存、降级。  
为什么：提高鲁棒性防雪崩。  
怎么用：@Retry/@CircuitBreaker；配置阈值窗口。  
关键细节：半开状态转换；窗口灵敏度。  
常见陷阱：阈值过紧频繁抖动；回退策略缺失；指标未上报无法调参。  
面试提示：过度敏感导致抖动。  
案例：熔断半开使用探针请求验证后再关闭熔断。  
速记方式：限-熔-退 要配套，调参靠指标。  

### 8.2 限流与熔断组合策略 [3]
是什么：限流控制速率；熔断快速失败。  
为什么：保护下游与自身稳定。  
怎么用：Gateway+Redis；Resilience4j 熔断重试。  
关键细节：错误率统计窗口；过度重试放大压力。  
常见陷阱：全局限流误杀关键接口；熔断恢复阈值太低；重试风暴。  
面试提示：缓存失效+下游抖动分析。  
案例：令牌桶限流 + 熔断 + 退避重试组合降低雪崩。  
速记方式：先限流，后熔断，再重试。  

## 9 可观测性与运维

### 9.1 Actuator 基础与端点扩展 [4]
是什么：健康、配置、指标、线程/堆转储端点。  
为什么：提升可观测性运维效率。  
怎么用：exposure.include；自定义 @Endpoint。  
关键细节：敏感端点鉴权；heapdump 资源影响。  
常见陷阱：将 /env、/beans 暴露公网；高频采集影响性能；生产触发 heapdump。  
面试提示：保护 /env 措施。  
案例：仅对内网开放管理端点并启用角色鉴权与采样限速。  
速记方式：少暴露、要鉴权、控成本。  

### 9.2 Micrometer 指标设计与标签控制 [4]
是什么：统一指标 API。  
为什么：跨监控系统可移植。  
怎么用：@Timed 或 timer.record；MeterBinder。  
关键细节：高基数标签；Gauge 生命周期。  
常见陷阱：将 userId 作为标签；Gauge 绑定临时对象；聚合维度过细。  
面试提示：高基数危害。  
案例：标签白名单与采样，删除高基数明细仅保留汇总。  
速记方式：少标签、稳对象、粗粒度。  

### 9.3 健康检查与 Liveness/Readiness [4]
是什么：区分存活与就绪。  
为什么：K8s 升级与故障隔离。  
怎么用：HealthIndicator；readiness 分组。  
关键细节：外部短暂波动不必下线；分级展示。  
常见陷阱：混用 liveness/readiness；短抖动即摘除；健康检查耗时长阻塞探针。  
面试提示：数据库抖动处理。  
案例：数据库短暂抖动只影响 readiness，不影响 liveness；探针限时。  
速记方式：活着≠就绪，分开看。  

### 9.4 常见排错思路（启动失败/Bean 覆盖） [4]
是什么：日志→阶段→配置→最小化复现。  
为什么：提升定位速度。  
怎么用：--debug / ConditionEvaluationReport。  
关键细节：Bean 覆盖 WARN；端口冲突；Profile 混用。  
常见陷阱：直接改配置未留痕；跳步骤定位；忽视自动装配条件报告。  
面试提示：5 步排错流程。  
案例：--debug + ConditionEvaluationReport 定位未生效的自动装配。  
速记方式：日志→条件→配置→复现→修复。  

### 9.5 可观测性：追踪 + 日志 + 指标整合 [4]
是什么：统一 TraceId/SpanId + 结构化日志 + 指标关联。  
为什么：减少跨系统排查成本。  
怎么用：MDC 注入；异常指标关联 Trace 采样。  
关键细节：异步上下文丢失；高采样率资源消耗。  
常见陷阱：未打通 TraceId 到日志与指标；全量采样导致压力；跨线程丢上下文。  
面试提示：高延迟指标定位调用链。  
案例：MDC 注入 TraceId，按错误率动态提高采样。  
速记方式：一号多用（追踪/日志/指标）。  

### 9.6 链路追踪上下游传播原理 [3]
是什么：Header (B3/W3C traceparent) 传递上下文。  
为什么：串联多服务调用。  
怎么用：拦截器/Filter 注入提取；异步包装。  
关键细节：跨线程丢失；采样策略。  
常见陷阱：线程池切换未传播上下文；上下游协议不一致；采样不统一。  
面试提示：TraceId vs SpanId。  
案例：配置 W3C Propagator 并包装线程池任务传递上下文。  
速记方式：Header 传上下文，线程也要传。  

## 10 测试与工程实践

### 10.1 Test Slice 与分层测试策略 [3]
是什么：按层局部上下文测试；全量用@SpringBootTest。  
为什么：减少执行时间。  
怎么用：切片 + Mock Beans；外部依赖 Testcontainers。  
关键细节：切片默认不加载 Security；需显式导入额外配置。  
常见陷阱：切片误加载全上下文变慢；Mock 不完整导致误报；测试互相污染。  
面试提示：单测/集成/端到端区分。  
案例：@WebMvcTest(controllers=X) 搭配 MockBeans 隔离外部依赖。  
速记方式：能切就切，尽量少装。  

### 10.2 Testcontainers 与环境一致性 [3]
是什么：容器拉起真实依赖。  
为什么：消除环境差异。  
怎么用：动态端口；公共基类初始化。  
关键细节：启动成本；镜像缓存。  
常见陷阱：CI 网络慢导致超时；端口冲突；版本漂移。  
面试提示：为何优于内存数据库。  
案例：复用容器、预拉镜像、固定版本标签并在 CI 缓存。  
速记方式：近真实，控成本，稳版本。  

### 10.3 模块拆分与巨大上下文控制 [3]
是什么：按领域/层拆分减少冲突启动时间。  
为什么：提高边界清晰度。  
怎么用：多模块构建；共享抽象。  
关键细节：跨模块循环依赖；公共模块膨胀。  
常见陷阱：把实现泄漏到公共模块；模块边界不清；测试碎片化。  
面试提示：分阶段演进策略。  
案例：按领域拆模块，公共模块仅放接口与 DTO，通过契约测试保障边界。  
速记方式：按域拆，轻公共，靠契约。  

## 11 性能与优化

### 11.1 启动性能优化与懒加载 [4]
是什么：减少启动扫描与实例化；延迟创建非关键 Bean。  
为什么：提升冷启动。  
怎么用：剔除无关 starter；lazy-initialization；分析启动报告。  
关键细节：懒加载首请求延迟；反射扫描占比。  
常见陷阱：引入过多 starter；组件扫描过宽；未禁用无用自动装配。  
面试提示：启动时间 vs 首请求延迟。  
案例：限定扫描包、移除无关 starter、开启 lazy-initialization 并对热点预热。  
速记方式：少 starter、窄扫描、可懒可热。  

### 11.2 常见性能陷阱（扫描/反射/序列化/N+1） [4]
是什么：过度扫描、频繁反射、深层 JSON、N+1。  
为什么：增加启动、CPU、IO 成本。  
怎么用：限制扫描路径；缓存反射；白名单序列化；fetch join。  
关键细节：Jackson 循环引用；Hibernate batch fetch。  
常见陷阱：N+1 查询；频繁反射与深层序列化；批处理缺失。  
面试提示：性能剖析修复 N+1。  
案例：使用 fetch join/批量处理/对象图优化降低序列化成本。  
速记方式：扫少、批多、浅序列。  



## 附录B 常见面试提示短句（更新）
- 生命周期：列阶段与增强点限制  
- 事务失效：自调用/访问级别/线程边界/异常处理  
- AOP：代理类型与自调用绕过  
- MVC：请求链扩展点与协商  
- WebFlux：IO 高并发适用与阻塞适配  
- 缓存：击穿/穿透/雪崩控制  
- 指标：高基数标签风险  
- 安全：过滤器链顺序、JWT 不可撤销策略  
- 性能：N+1 诊断与修复、启动优化  
- 诊断：ConditionEvaluationReport 使用路径  

(完)