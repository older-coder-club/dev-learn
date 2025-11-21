# Java 语言与核心 API 面试指南（加强版）

评分说明：5=必须掌握，4=很重要，3=熟练更佳，2=了解即可，1=加分项  
补充说明：下面每个知识点均从“是什么 / 为什么 / 有什么用 / 关键细节 / 面试提示”多个角度展开，便于系统化回答面试官追问。

---

## 1. 集合框架与数据结构 [5]

### 1.1 集合体系与设计哲学
- 是什么：Java Collections Framework 通过接口层次（Collection、List、Set、Queue、Map）与具体实现类（ArrayList、LinkedList、HashSet、TreeSet、HashMap、LinkedHashMap、ConcurrentHashMap 等）提供统一抽象。
- 为什么：统一 API 降低认知成本，实现互换（面向接口编程），提供不同复杂度与特性（顺序、排序、并发、安全）。
- 有什么用：支撑绝大多数业务数据存储、缓存与聚合场景。
- 关键细节：可变/不可变视图（Collections.unmodifiableXXX、List.of）、结构性修改 modCount 影响 fail‑fast 迭代器；迭代中修改集合需用 Iterator.remove。
- 面试提示：能清晰选择结构 + 说明复杂度 + 并发场景替代选择。

### 1.2 ArrayList vs LinkedList
- 是什么：前者基于动态数组，后者双向链表。
- 为什么：不同数据访问模式（随机访问 vs 频繁头尾插入）。
- 用：ArrayList 适合随机访问与尾部追加；LinkedList 在大量中间插入的优势很有限（定位成本高）。
- 细节：扩容算法（一般 1.5 倍），扩容触发需要复制数组；LinkedList 节点对象内存额外开销，迭代稳定。
- 面试提示：LinkedList 并不适合高频随机插入（定位 O(n)），避免背诵“链表插入 O(1)”误导。

### 1.3 HashMap 内部结构与演进
- 是什么：数组 + 链表/红黑树（JDK 8+ 大桶转树）。
- 为什么：平均 O(1) 查找，树化优化高碰撞时退化。
- 用：通用键值缓存、频次统计、索引反查。
- 细节：扰动函数减少低质量 hash 聚集；负载因子默认 0.75；resize 需重新分配桶导致瞬时成本；链表长度 ≥8 且容量 ≥64 treeify。
- 面试提示：说明树化触发条件与为什么不是立即树化（空间与阈值折中）。

### 1.4 ConcurrentHashMap
- 是什么：高并发散列表，JDK8 改为分段理念弱化 → 节点级 CAS + 链/树。
- 为什么：提供近似 O(1) 并发访问且减少锁粒度。
- 用：并发缓存、计数、并发映射聚合。
- 细节：size 估算多步累加，compute/merge/putIfAbsent 原子复合语义，避免外部加锁。树化与链表逻辑类似普通 HashMap。
- 面试提示：描述并发安全来源（CAS + synchronized 局部锁）与为何 size() 不是严格 O(1)。

### 1.5 CopyOnWrite 家族
- 是什么：写时复制集合（CopyOnWriteArrayList/Set）。
- 为什么：读远多于写场景避免锁争用，迭代快照稳定。
- 用：系统配置快照、订阅者列表。
- 细节：写操作创建新数组，内存短暂翻倍；迭代不受并发写影响但不反映最新写。
- 面试提示：指出不适合大数据高频写；避免用于实时强一致。

### 1.6 阻塞队列与并发队列
- 是什么：生产者/消费者协作结构，BlockingQueue 系列（ArrayBlockingQueue、LinkedBlockingQueue、SynchronousQueue、DelayQueue）。
- 为什么：隔离生产与消费速率，应对背压。
- 用：任务调度、线程池工作队列、延迟执行。
- 细节：ArrayBlockingQueue 定长+单锁；LinkedBlockingQueue 链表+入出两锁；SynchronousQueue 无存储、直接移交；吞吐与延迟不同。
- 面试提示：区分使用场景与“为什么 SynchronousQueue 适合 cachedThreadPool”。

### 1.7 LongAdder vs AtomicLong
- 是什么：热点竞争下降计数替代方案；分桶累加。
- 为什么：减少单点 CAS 重试冲突，提高并发统计性能。
- 用：高并发下计数、监控指标累加。
- 细节：读取 sum 需聚合，多线程写性能高但瞬时读取非强一致。
- 面试提示：强调不适合需要严格读写原子场景（例如依赖旧值再计算）。

### 1.8 跳表与有序视图 (NavigableMap)
- 是什么：基于分层随机化结构（ConcurrentSkipListMap），支持排序与范围检索。
- 为什么：替代树结构提供并发友好的有序集合。
- 用：排行榜、时间窗口检索、按键前缀范围扫描。
- 细节：近似 O(log n)；并发插入/删除锁粒度小。
- 面试提示：对比 TreeMap（需要整体锁）并发劣势。

### 1.9 不可变集合与 Defensive Copy
- 是什么：不可变对象防止结构被外部修改；List.of()、Collections.unmodifiableXXX。
- 为什么：线程安全、避免意外修改、保证数据一致性。
- 用：配置、常量、快照。
- 细节：不可变 vs 只读视图区别（后者底层仍可能改变）；Defensive copy 避免暴露内部数组或可变对象。
- 面试提示：能举例“返回内部数组前先复制”原因。

### 1.10 常见陷阱与复杂度意识
- 是什么：对操作的时间/空间成本认知。
- 为什么：避免在热点使用非预期高成本结构。
- 用：提前 capacity 规划、避免在迭代中 remove 非迭代器方式。
- 面试提示：强调 ArrayList 扩容摊还 O(1)；HashMap 初始容量合理预估减少 resize。

---

## 2. 异常体系与最佳实践 [4]

### 2.1 Checked vs Unchecked 异常分层
- 是什么：Checked 需显式处理；Unchecked 表示编程错误或不可恢复。
- 为什么：限制异常泛滥，提高调用者注意力；避免吞掉关键错误。
- 用：业务可恢复异常（资源不可用重试） vs 逻辑错误（NullPointer）。
- 细节：过度使用 checked 造成调用链污染。
- 面试提示：讲述转换策略（底层 IO 异常包装成业务语义异常）。

### 2.2 异常传播与语义保留
- 是什么：包装异常时保留根因 cause。
- 为什么：便于排查根源，不丢栈信息。
- 用：在层边界转换为更领域化异常。
- 细节：不要重复日志→再抛出→再次日志造成噪音。
- 面试提示：说明“只在边界 logging”原则。

### 2.3 自定义异常规范
- 是什么：业务/系统/远程区分；名称清晰 (ResourceNotFoundException)。
- 为什么：提高可读性与处理分支简化。
- 用：上层 catch 分类处理（重试/降级/告警）。
- 细节：字段携带上下文（ID/状态）。
- 面试提示：避免万能 BusinessException。

### 2.4 性能与误用
- 是什么：频繁抛异常成本高（构造栈）。
- 为什么：异常用于“例外”非正常流控。
- 用：改用结果对象 / Optional 表达可预见状态。
- 细节：循环中抛异常=隐式慢路径。
- 面试提示：列举“解析失败抛异常 vs 预检查”取舍。

### 2.5 try-with-resources 与 Suppressed
- 是什么：自动关闭资源 + 避免遗漏 finally。
- 为什么：简化资源管理，减少泄漏。
- 用：IO/数据库/锁资源。
- 细节：多个资源关闭发生异常时使用 suppressed 保存附加异常。
- 面试提示：指出“手动 finally 可能吞掉原始异常”。

### 2.6 InterruptedException 处理
- 是什么：线程被请求中断信号。
- 为什么：协作式取消；忽略会导致不可预期等待。
- 用：恢复中断 (Thread.currentThread().interrupt()) 后交由上层处理。
- 面试提示：说明不应空 catch。

---

## 3. I/O 与 NIO / NIO.2 [4]

### 3.1 阻塞 vs 非阻塞 IO
- 是什么：阻塞等待数据 vs 立即返回状态由事件驱动。
- 为什么：非阻塞减少线程占用提高连接数扩展性。
- 用：高并发网络服务（Reactor 模型）。
- 细节：一个线程管理多 Channel；需要处理半包粘包协议。
- 面试提示：说明非阻塞仍需处理“读不到数据”循环。

### 3.2 Buffer 与 Channel
- 是什么：Buffer 管理数据状态(position/limit)；Channel 抽象传输通道。
- 为什么：集中控制、减少复制。
- 用：FileChannel/SocketChannel 同一 API。
- 细节：flip/clear/compact 操作易错；DirectBuffer 利用零拷贝但分配释放成本更高。
- 面试提示：解释“为什么频繁小 DirectBuffer 不合适”。

### 3.3 Selector 机制
- 是什么：注册兴趣 + 事件轮询。
- 为什么：极少线程管理大量连接。
- 用：高并发服务器。
- 细节：epoll 空轮询 bug（旧内核），wakeup 防止阻塞。
- 面试提示：讲述 select() 阻塞与唤醒策略。

### 3.4 零拷贝与高性能传输
- 是什么：避免用户态/内核态重复复制（mmap、transferTo）。
- 为什么：提升大文件/流媒体吞吐。
- 用：下载服务/大对象分发。
- 细节：mmap 需谨慎释放；FileChannel transferTo 可能受平台限制。
- 面试提示：比较“传统 read→write vs transferTo”。

### 3.5 Scatter/Gather 与协议解析
- 是什么：将数据分散/聚合至多 Buffer。
- 为什么：分离头/体处理，减少拼接复制。
- 用：自定义传输协议处理。
- 面试提示：说明性能与语义好处。

### 3.6 NIO.2 文件 API
- 是什么：Path、Files、WatchService 提升文件操作抽象。
- 用：原子移动、符号链接、目录监视。
- 面试提示：阐述 WatchService 在不同平台的局限。

### 3.7 AIO (Asynchronous IO)
- 是什么：操作系统直接回调通知结果。
- 为什么：进一步减少线程上下文切换。
- 用：少平台支持完全异步；实际多用 NIO + Reactor。
- 面试提示：说明 AIO 不等于自动高性能（取决内核支持）。

### 3.8 网络参数调优
- 是什么：SO_RCVBUF/SO_SNDBUF、TCP_NODELAY、SO_REUSEADDR。
- 为什么：影响吞吐、延迟与端口复用。
- 面试提示：解释 Nagle 关闭对小包延迟影响。

---

## 4. 函数式与 Stream API [4]

### 4.1 Lambda 与闭包
- 是什么：匿名函数表达逻辑；捕获有效 final 变量形成闭包。
- 为什么：简化集合操作，提升可读性。
- 用：map/filter/reduce 处理流水线。
- 细节：捕获外部对象可能延长生命周期。
- 面试提示：说明为何大对象引用在并行流中可能产生内存压力。

### 4.2 流式管道惰性
- 是什么：中间操作不执行直到终止操作。
- 为什么：构建管道优化执行（融合/短路）。
- 用：链式数据转换与聚合。
- 细节：有状态操作（sorted/distinct）需缓存导致内存膨胀。
- 面试提示：解释 distinct 在大数据集的代价。

### 4.3 并行流陷阱
- 是什么：使用 ForkJoinPool 默认共享线程。
- 为什么：阻塞或非线程安全操作可能降低性能/污染其他任务。
- 面试提示：给出适配场景（CPU 密集、避免同步结构）。

### 4.4 Collector 与自定义聚合
- 是什么：定义供应器、累加器、组合器实现可并行归约。
- 为什么：扩展 reduce 类型（分组、统计）。
- 用：groupingBy、mapping、partitioningBy 等。
- 细节：并行下组合器必须无副作用。
- 面试提示：说明使用 toMap 时处理键冲突的必要性。

### 4.5 Spliterator
- 是什么：可拆分迭代器用于并行切分。
- 为什么：改善数据分片粒度。
- 用：自定义数据源并行化。
- 面试提示：说明 characteristics 对并行效率影响。

### 4.6 Optional 使用规范
- 是什么：显式表示可能缺失值的容器。
- 为什么：避免 NullPointer 与语义不明确。
- 用：链式 map/flatMap/filter。
- 面试提示：强调不在字段、集合元素中使用 Optional（序列化与性能问题）。

---

## 5. 序列化与 JSON 生态 [3]

### 5.1 Java 原生序列化问题
- 是什么：ObjectOutputStream/ObjectInputStream 二进制形式。
- 为什么不推荐：性能低、版本脆弱、存在安全反序列化攻击风险。
- 面试提示：列举 gadget 攻击与 serialVersionUID 兼容难题。

### 5.2 Externalizable vs Serializable
- 是什么：自定义序列化过程 vs 默认反射。
- 为什么：优化性能与控制字段结构。
- 面试提示：强调需手动处理所有字段、防止遗漏。

### 5.3 JSON 框架对比
- Jackson：丰富特性与 streaming。
- Gson：反射多，性能较低。
- Fastjson：性能快但历史安全事件多。
- 面试提示：选择时基于安全与生态成熟度。

### 5.4 多态与类型信息
- 是什么：@JsonTypeInfo 或手动类型字段。
- 为什么：反序列化需要恢复实例具体类型。
- 面试提示：说明默认启用全类型曝光风险。

### 5.5 兼容与演进
- 是什么：字段新增/删除/默认值变化。
- 为什么：保持跨服务/版本向后兼容。
- 用：使用 null/缺省值策略；避免重命名破坏兼容。
- 面试提示：谈及 schema version 与稳定性。

### 5.6 性能优化
- ObjectMapper 复用、关闭默认特性（FAIL_ON_UNKNOWN_PROPERTIES）、使用 Afterburner。
- 面试提示：说明避免频繁创建 ObjectMapper。

### 5.7 安全防护
- 白名单类、限制深度、大小、禁用默认多态。
- 面试提示：举例“远程输入反序列化风险”。

---

## 6. 日期与时间 API (java.time) [3]

### 6.1 核心类型语义
- Instant：时间点（UTC）。
- LocalDateTime：无时区人类时间，不能直接比较跨时区事件。
- ZonedDateTime：包含时区规则与 DST。
- OffsetDateTime：固定偏移。
- 面试提示：分析 DST 对定时任务影响。

### 6.2 不变性与线程安全
- 为什么：避免共享修改造成竞态。
- 用：组合操作返回新对象。
- 面试提示：Date/Calendar 不可安全共享。

### 6.3 格式与解析
- DateTimeFormatter：线程安全；Locale 影响大小写与语言。
- 面试提示：解释 SimpleDateFormat 线程不安全问题。

### 6.4 时区转换策略
- 统一存储 Instant/UTC → 展示层转换。
- 面试提示：说明数据库保存时间戳的好处。

### 6.5 Clock 抽象
- 为什么：测试可控时间逻辑。
- 面试提示：用固定 Clock 避免测试不稳定。

---

## 7. 字符串与编码处理 [4]

### 7.1 编码与内存
- UTF-16 内部表示；JDK9 Compact Strings 根据是否 Latin1 优化。
- 面试提示：说明 String 不可变性与常量池提升复用。

### 7.2 拼接与性能
- 编译期将 “a”+ x + “b” 转成 StringBuilder；循环中应显式使用 StringBuilder。
- 面试提示：解释 + 在热路径内的性能退化。

### 7.3 Locale 与正规化
- 大小写转换在土耳其语等特殊语言下差异。
- 面试提示：避免对国际化用户出现错误比较。

### 7.4 intern
- 是什么：将字符串放入字符串常量池。
- 为什么：减少重复内存（有限条件）。
- 面试提示：滥用 intern 可能造成常量池膨胀。

### 7.5 编码显式控制
- InputStreamReader/OutputStreamWriter 必须指定 Charset，避免依赖默认。
- 面试提示：避免跨平台出现乱码。

---

## 8. 泛型与类型擦除 [5]

### 8.1 擦除机制
- 是什么：编译期检查，运行期类型信息擦除为上界/Object。
- 为什么：保持向后兼容（老 JVM）。
- 面试提示：说明不能获取泛型运行时具体类型 (T.class)。

### 8.2 通配符与 PECS
- ? extends 读协变；? super 写逆变。
- 为什么：保证类型安全，限制操作方向。
- 面试提示：举例 List<? super Number> 添加 Integer 可行但读取需 Object。

### 8.3 上界与多重约束
- <T extends InterfaceA & InterfaceB>，顺序：类在前接口在后。
- 面试提示：合理限定能力集合。

### 8.4 数组与泛型
- 数组协变不安全 vs 泛型不协变，编译期防止类型错误。
- 面试提示：说明为何不推荐使用泛型数组 (Type erasure + 运行期不安全)。

### 8.5 桥接方法
- 编译器生成以保证多态签名一致。
- 面试提示：反射看到的额外方法不困惑。

### 8.6 设计原则
- 返回类型使用协变，参数使用逆变提高灵活性。
- 面试提示：能重构 API 降低泛型复杂度。

---

## 9. 反射与注解机制 [4]

### 9.1 ClassLoader
- 是什么：类加载器层次结构定义类型唯一性。
- 为什么：隔离模块，多版本共存。
- 面试提示：说明上下文加载器在 SPI 的作用。

### 9.2 反射性能与缓存
- setAccessible 成本；Method/Field 缓存避免重复查找。
- 面试提示：指出大量反射在热路径需替换为直接调用或 MethodHandle。

### 9.3 MethodHandle 与 invokedynamic
- 更贴近底层、JIT 优化空间更大。
- 面试提示：说明现代框架为什么开始使用。

### 9.4 注解保留策略
- SOURCE/CLASS/RUNTIME；运行期处理需 RUNTIME。
- 面试提示：标注自定义注解需明确元注解 @Retention/@Target。

### 9.5 扫描与启动成本
- 大量包扫描减慢启动；索引化（Spring 的预编元数据）。
- 面试提示：说明减少反射增强（AOT/Native Image）趋势。

### 9.6 动态代理
- JDK 动态代理基于接口；CGLIB 基于子类；final 方法不可拦截。
- 面试提示：解释 @Transactional 自调用失效原因（未通过代理）。

---

## 10. Optional 最佳实践 [3]

### 10.1 语义定位
- 表示“可能为空”而非业务实体；避免滥用。
- 面试提示：说明不用于字段（序列化成本与嵌套难读）。

### 10.2 链式操作
- map/flatMap/filter/orElseThrow 提升可读性。
- 面试提示：强调 orElse vs orElseGet 的惰性差异。

### 10.3 性能与替代
- 在热点循环中频繁构建 Optional 有额外对象开销。
- 面试提示：必要时直接 null + 注释语义。

---

## 11. 新语言特性 (Record / Sealed / Pattern Matching) [3]

### 11.1 Record
- 是什么：面向数据载体的简化语法（自动生成构造器/equals/hashCode/toString）。
- 为什么：减少样板代码，提高不可变建模。
- 面试提示：说明可定义 compact constructor 做校验。

### 11.2 Sealed
- 是什么：限制继承范围，提高模式匹配穷尽性。
- 为什么：增强 API 控制与安全。
- 面试提示：列举 sealed + permits 使用。

### 11.3 Pattern Matching
- instanceof 与 switch 模式语法（类型解构）。
- 面试提示：趋势：减少样板类型转换代码。

---

## 12. 数值与精度 (BigDecimal) [3]

### 12.1 精度与构造
- BigDecimal(double) 产生二进制小数误差；推荐使用字符串或 valueOf。
- 面试提示：能展示错误示例。

### 12.2 舍入与 scale
- RoundingMode（HALF_UP、HALF_EVEN）选型与财务规范。
- 面试提示：equals 与 compareTo 不同：scale 影响 equals。

### 12.3 性能
- 频繁高精度运算在热点路径需优化（如预乘常量）。
- 面试提示：指出 BigDecimal 在集合 key 使用慎重（equals/scale）。

---

## 13. 对象与不可变设计 [4]

### 13.1 不可变好处
- 线程安全、缓存 key 稳定、失败回滚更简单。
- 面试提示：举例 value object。

### 13.2 Defensive Copy
- 构造时复制输入，访问时返回副本防止外部修改。
- 面试提示：说明数组/Date 常见风险。

### 13.3 equals/hashCode 合约
- 必须一致性：equals 为真 → hashCode 相同。
- 面试提示：影响 HashMap/HashSet 行为（丢失元素）。

---

## 14. 设计与 API 可用性 [2]

### 14.1 语义清晰
- 方法命名表达意图；减少布尔陷阱 (flag1, flag2)。
- 面试提示：避免重载模糊，比如不同语义同参数个数。

### 14.2 参数验证
- 前置失败快速反馈；IllegalArgumentException / NullPointerException 区分。
- 面试提示：能说明 precondition 的价值。

---

## 15. 资源管理与 AutoCloseable [3]

### 15.1 try-with-resources
- 自动关闭顺序与 suppressed 异常辅助调试。
- 面试提示：对比手动 finally 容易遗漏。

### 15.2 Cleaner 取代 finalize
- finalize 不确定性与性能差；Cleaner 可控回收。
- 面试提示：表明 finalize 已废弃用途。

---

## 16. 内存与对象模型基础 [3]

### 16.1 对象头
- Mark Word（锁状态/哈希码/年龄）+ Klass Pointer。
- 面试提示：解释锁升级状态（偏向→轻量→重量）。

### 16.2 对齐与填充
- 8 字节对齐，避免伪共享在复杂结构中。
- 面试提示：指出小对象过多导致 GC 压力。

### 16.3 装箱与缓存
- Integer 缓存 [-128,127]；频繁装箱创建垃圾。
- 面试提示：强调热点使用原始类型或 LongAdder。

---

## 常见面试抽象性问题示例（加强解释）
1. ConcurrentHashMap 扩容为什么无需全表锁？桶级迁移 + 标记节点 + CAS 控制并发协作。  
2. parallelStream 退化场景？数据集小、阻塞 IO、共享可变状态、复杂有状态操作、线程池竞争。  
3. 原生序列化不推荐原因？性能差、版本脆弱（字段兼容难）、安全风险（反序列化链执行）。  
4. ZonedDateTime vs OffsetDateTime？Zoned 包含时区规则含 DST；Offset 仅固定偏移，不适合需要 DST 计算的业务。  
5. ? super vs ? extends？Producer Extends / Consumer Super；确保读写方向安全。  

---

## 学习与提升建议
- 源码阅读：HashMap、ConcurrentHashMap、Collections、Stream、Record。
- 微基准：JMH 验证集合扩容、Stream vs for 循环、装箱成本。
- 异常治理：建立库化处理（统一转换与日志策略）。
- 时间 API 迁移：替换旧 Date/Calendar + 统一时区策略 + 测试验证。
- 安全序列化：统一 ObjectMapper 配置（禁用多态、确定字段全集）。

---

## 重要度汇总
- 集合框架 [5]  
- 泛型与类型擦除 [5]  
- 异常体系 [4]  
- I/O / NIO [4]  
- 函数式与 Stream [4]  
- 字符串与编码 [4]  
- 反射与注解 [4]  
- 不可变对象与设计 [4]  
- 序列化与 JSON [3]  
- 日期时间 API [3]  
- Optional [3]  
- BigDecimal 与数值 [3]  
- 资源管理 (AutoCloseable) [3]  
- 内存与对象模型基础 [3]  
- 新语言特性 (Record/Sealed/Pattern Matching) [3]  
- API 可用性补充 [2]  

(完)