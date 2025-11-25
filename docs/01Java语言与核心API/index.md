# Java 语言与核心 API 面试指南（加强版）

评分说明：5=必须掌握，4=很重要，3=熟练更佳，2=了解即可，1=加分项  
补充说明：下面每个知识点均从“是什么 / 为什么 / 有什么用（怎么用） / 关键细节 / 面试提示”多个角度展开，便于系统化回答追问。面试回答建议：先主线→再细节→再风险与优化→最后真实案例。

---
## 面试知识点难度与重要度评分总览

| 序号 | 主题                               | 重要度(1-5) | 难度(1-5) | 备注(核心掌握点) |
| ---- |----------------------------------| ----------- | --------- | ---------------- |
| 1 | 集合框架总览与选型  | 5 | 3 | 结构/复杂度/选型矩阵 |
| 2 | ArrayList vs LnkedList           | 5 | 2 | 时空局部性与链表误区 |
| 3 | HashMap 结构与树化                    | 5 | 3 | 负载因子/resize/树化条件 |
| 4 | ConcurrentHashMap 并发机制           | 5 | 4 | CAS + 局部锁 + 原子操作 |
| 5 | CopyOnWrite 容器                   | 4 | 3 | 读多写少与内存放大 |
| 6 | 阻塞/并发队列选型                        | 4 | 3 | 各队列语义与背压 |
| 7 | ConcurrentSkipListMap 跳表         | 4 | 3 | 有序并发与范围查询 |
| 8 | 不可变集合与 Defensive Copy            | 4 | 2 | 线程安全/封装保护 |
| 9 | 异常分层与传播                          | 4 | 2 | Checked vs Unchecked / 边界单次日志 |
| 10 | try-with-resources & suppressed  | 4 | 2 | 自动关闭与异常保留 |
| 11 | InterruptedException 与中断语义       | 4 | 3 | 恢复中断/协作取消 |
| 12 | 阻塞 vs 非阻塞 NIO                    | 4 | 4 | Selector/半包处理 |
| 13 | Buffer/Channel 生命周期              | 4 | 3 | flip/clear/compact 差异 |
| 14 | Selector 工作机制                    | 4 | 4 | 事件分发与 OP_WRITE 控制 |
| 15 | 零拷贝传输与 mmap                      | 4 | 3 | sendfile/transferTo 场景 |
| 16 | 背压与流控策略                          | 4 | 3 | 写缓冲水位/限速策略 |
| 17 | Stream 惰性与短路                     | 4 | 2 | 中间/终止操作区分 |
| 18 | 并行流适用条件与陷阱                       | 4 | 3 | 数据量/阻塞/拆分/共享状态 |
| 19 | Collector 五要素                    | 4 | 3 | supplier/accumulator/combiner |
| 20 | Spliterator 拆分特征                 | 3 | 4 | trySplit/特征标志 |
| 21 | Optional 规范                      | 3 | 2 | 返回值/避免字段/惰性 |
| 22 | 字符串不可变与内存布局                      | 4 | 2 | byte[]+coder/哈希缓存 |
| 23 | 字符串拼接与性能                         | 4 | 2 | 编译器优化/+ vs StringBuilder |
| 24 | 编码与 Charset 管理                   | 4 | 2 | UTF-8 显式/避免默认编码 |
| 25 | Unicode 代码点处理                    | 4 | 3 | surrogate/length vs codePointCount |
| 26 | Locale 与正规化                      | 3 | 3 | Locale.ROOT/Normalization |
| 27 | 泛型擦除与桥接方法                        | 5 | 4 | 上界/运行时无实参类型 |
| 28 | PECS 原则                          | 5 | 3 | Producer extends / Consumer super |
| 29 | TypeToken 捕获泛型                   | 4 | 3 | 匿名子类获取 ParameterizedType |
| 30 | 堆污染与 @SafeVarargs                | 4 | 3 | 泛型可变参安全声明 |
| 31 | Raw Type 风险                      | 4 | 2 | unchecked 警告与类型安全 |
| 32 | 反射性能优化 (缓存/MethodHandle)         | 4 | 4 | 元数据缓存/替换反射 |
| 33 | ClassLoader 双亲委派与泄漏              | 4 | 4 | 上下文加载器/SPI/泄漏案例 |
| 34 | 注解模型与元注解                         | 4 | 2 | Retention/Target/Inherited |
| 35 | APT 编译期处理                        | 3 | 3 | Processor/多轮/生成源码 |
| 36 | 动态代理技术栈                          | 4 | 3 | JDK Proxy/CGLIB/ByteBuddy |
| 37 | 模块封装与 --add-opens                | 3 | 3 | 强封装/最小开放 |
| 38 | 虚拟线程 (Loom)                      | 3 | 3 | 阻塞挂起/线程数量扩展 |
| 39 | Structured Concurrency           | 3 | 3 | 作用域取消/聚合异常 |
| 40 | Scoped Values                    | 3 | 3 | 上下文只读替代 ThreadLocal |
| 41 | Foreign Function & Memory API    | 3 | 4 | MemorySegment/Linker/Arena |
| 42 | Record                           | 3 | 2 | 不可变值对象语法糖 |
| 43 | Sealed Classes                   | 3 | 3 | permits/穷举校验 |
| 44 | Pattern Matching instanceof/switch | 3 | 3 | 守护条件/穷举安全 |
| 45 | Switch Expressions               | 3 | 2 | 箭头/yield/无贯穿 |
| 46 | Text Blocks                      | 3 | 1 | """ 多行字符串 |
| 47 | BigDecimal 构造与 scale             | 3 | 2 | valueOf/equals vs compareTo |
| 48 | RoundingMode 与财务规则               | 3 | 2 | HALF_EVEN/除法需模式 |
| 49 | BigDecimal 性能与 long “分”          | 3 | 2 | 分级存储/减少对象 |
| 50 | 不可变对象与 Defensive Copy            | 4 | 2 | final + 深不可变 |
| 51 | equals/hashCode 合约               | 4 | 2 | 自反/传递/一致/哈希稳定 |
| 52 | 资源管理 try-with-resources          | 3 | 2 | 逆序关闭/suppressed |
| 53 | Cleaner 替代 finalize              | 3 | 2 | 兜底/非实时 |
| 54 | 对象头与锁升级                          | 3 | 4 | 偏向→轻量→重量级 |
| 55 | 对齐/伪共享/@Contended                | 3 | 3 | 缓存行隔离 |
| 56 | 装箱与缓存区间                          | 3 | 2 | [-128,127] 缓存/避免热点装箱 |
| 57 | API 命名与参数校验                      | 2 | 2 | 语义明确/早失败 |
| 58 | JSON 序列化选型与安全                    | 3 | 3 | ObjectMapper 复用/禁默认多态 |
| 59 | 多态反序列化白名单                        | 3 | 3 | @JsonTypeInfo 安全控制 |
| 60 | 序列化兼容与演进策略                       | 3 | 2 | 默认值/双阶段删除 |

---

## 常见面试问题与解答
1. 问：HashMap 为什么需要树化链表？  
   思路：碰撞链过长退化 O(n) → JDK8 红黑树化；触发条件（链≥8 且容量≥64）；折中空间与常态性能。

2. 问：HashMap 扩容发生在何时？  
   思路：size > capacity * loadFactor；新容量=旧容量*2；迁移成本集中，建议预估初始容量。

3. 问：ConcurrentHashMap 为何不再分段？  
   思路：JDK8 使用节点级 CAS + 局部 synchronized + 红黑树；提高并发度，减少锁数量与伪共享。

4. 问：ConcurrentHashMap 中 size() 为什么不精确实时？  
   思路：并发更新中统计需遍历或累加近似；可用 mappingCount 或基于阈值决策而非精确值。

5. 问：ConcurrentHashMap 用什么方式实现复合原子操作？  
   思路：compute / merge / putIfAbsent；强调不要外层 synchronized。

6. 问：ArrayList 与 LinkedList 性能差异来源？  
   思路：ArrayList 连续内存 + O(1) 随机访问；LinkedList 节点分散 + 定位 O(n)；缓存局部性优劣。

7. 问：为什么实际开发很少使用 LinkedList？  
   思路：随机访问与中间插入需遍历；额外对象指针与 GC 压力；场景极少。

8. 问：CopyOnWriteArrayList 适用场景？  
   思路：读多写少，如监听器、配置、订阅；迭代稳定；写放大与内存双倍。

9. 问：BlockingQueue 常见实现区别？  
   思路：ArrayBlockingQueue（定长低延迟）、LinkedBlockingQueue（链表通用）、SynchronousQueue（直接交接）、DelayQueue（定时）、PriorityBlockingQueue（优先级）。

10. 问：SynchronousQueue 为什么适合 cachedThreadPool？  
    思路：零容量直接交接任务 → 快速创建/销毁线程满足突发。

11. 问：ConcurrentSkipListMap 相比 TreeMap 并发优势？  
    思路：跳表局部指针修改，无全局锁或旋转；近似 O(log n) 有序操作 + 范围查询。

12. 问：fail‑fast 迭代器原理？  
    思路：modCount 修改版本检测；迭代过程检测到结构变化抛 ConcurrentModificationException。

13. 问：不可变集合与 unmodifiable 有何区别？  
    思路：List.of 真不可变；Collections.unmodifiableX 仅包装底层集合（底层仍可变）。

14. 问：如何安全返回内部集合？  
    思路：防御性复制或返回不可变副本；避免内部结构泄漏。

15. 问：为什么 String 设计为不可变？  
    思路：安全（类加载、URL）、并发共享、hash 缓存稳定、作为 Map key 保障一致性。

16. 问：JDK9 Compact Strings 带来什么收益？  
    思路：value 变 byte[] + coder；LATIN1 单字节节省内存与提升缓存命中。

17. 问：substring 早期共享父数组问题？  
    思路：JDK7 之前共享 char[] 导致大对象泄漏；现在复制子数组消除风险。

18. 问：intern 使用策略？  
    思路：少量高度重复常量（状态、枚举）；避免海量动态值导致堆膨胀。

19. 问：循环中字符串拼接为何低效？  
    思路：String 不可变，每次 + 生成新对象与复制；应使用单一预容量 StringBuilder。

20. 问：编码未显式指定风险？  
    思路：平台默认编码差异（UTF-8 vs GBK） → 解析错误、签名不一致；必须 StandardCharsets.UTF_8。

21. 问：length() 与真实字符数差异？  
    思路：UTF-16 代码单元计数；代理对（emoji）占两个 char；用 codePointCount。

22. 问：Locale.ROOT 的使用场景？  
    思路：逻辑规范化（lowerCase/upperCase）稳定行为，避免地区特殊规则（土耳其语 I）。

23. 问：Optional 何时使用？  
    思路：返回值表达缺失；不用于字段、集合元素；节省空判断分支。

24. 问：Optional.orElse 与 orElseGet 区别？  
    思路：orElse 总执行参数表达式；orElseGet 惰性执行 Supplier。

25. 问：泛型类型擦除是啥及为什么？  
    思路：编译期检查+插入强转；运行时擦除成上界；与旧 JVM 兼容，简化运行时。

26. 问：为什么不能 instanceof List<String>?  
    思路：运行时无泛型实参（擦除）；只能判断原始类型 List。

27. 问：PECS 原则含义与示例？  
    思路：Producer extends（只读） / Consumer super（写入）；copy(src, dst)。

28. 问：? extends 集合不能 add 的原因？  
    思路：元素具体类型不确定（可能是子类）；写入破坏类型安全 → 禁止除 null。

29. 问：桥接方法 (Bridge Method) 作用？  
    思路：保持子类返回更具体类型后的多态一致；编译器生成 synthetic 方法。

30. 问：堆污染定义与防范？  
    思路：泛型类型不一致引用写入错误元素导致后续 ClassCastException；避免参数化数组/谨慎泛型 varargs；@SafeVarargs 声明内部安全。

31. 问：Raw Type 危害？  
    思路：跳过编译期类型检查；出现 unchecked 警告；运行时 ClassCastException。

32. 问：如何在运行时获取 List<String> 的元素类型？  
    思路：使用 TypeToken / 捕获 ParameterizedType；无法直接从实例泛型推断。

33. 问：为什么泛型对性能影响可忽略？  
    思路：仅编译期存在；开销在装箱与转换而非语法；关注原始类型使用。

34. 问：反射性能低的根本原因？  
    思路：访问检查、方法查找、无法 JIT 内联；频繁调用导致热点退化。

35. 问：如何优化反射访问？  
    思路：启动期扫描缓存 Method/Field；转 MethodHandle/VarHandle；避免重复 setAccessible。

36. 问：MethodHandle 优势？  
    思路：JIT 可内联、类型签名严格、组合器灵活；性能优于传统反射调用。

37. 问：ClassLoader 双亲委派意义？  
    思路：防止核心类被篡改；类型一致性；安全隔离；向上委派再本地加载。

38. 问：上下文 ClassLoader 用途？  
    思路：SPI（ServiceLoader）、框架在回调线程加载应用类；区别系统加载器。

39. 问：ClassLoader 泄漏常见场景？  
    思路：容器热部署静态缓存/ThreadLocal 持引用；导致类无法卸载 → Metaspace 增长。

40. 问：注解元注解有哪些及作用？  
    思路：@Retention（生命周期）、@Target（适用位置）、@Documented、@Inherited、@Repeatable。

41. 问：@Inherited 限制？  
    思路：仅类层次；不作用于方法、字段、接口。

42. 问：编译期注解处理优势？  
    思路：减少运行时扫描与反射；提前失败；生成高效代码（MapStruct/AutoService）。

43. 问：动态代理 JDK Proxy 与 CGLIB 区别？  
    思路：JDK Proxy 基于接口；CGLIB 继承子类（无法代理 final）；性能与限制差异。

44. 问：事务自调用失效原因？  
    思路：内部直接调用方法未经过代理层切面；需通过代理或拆分 Bean。

45. 问：为什么升级到 Java 17 后反射失败？  
    思路：模块强封装；未添加 --add-opens；需最小开放策略。

46. 问：Metaspace OOM 排查思路？  
    思路：检测大量动态生成类（代理/增强）与未卸载 ClassLoader；使用 jcmd/jfr/native_memory。

47. 问：异常只在边界记录的原因？  
    思路：防日志风暴与堆栈重复；保持一次上下文清晰；内部层抛出不重复 log。

48. 问：为什么不使用异常做控制流？  
    思路：构造与栈捕获昂贵；影响热点编译；可预测失败用返回值（Optional / Result）。

49. 问：InterruptedException 正确处理方式？  
    思路：catch 后恢复中断标志或向上抛；不要吞掉；循环检测 Thread.interrupted()。

50. 问：try-with-resources 相比 finally 优势？  
    思路：自动逆序关闭 + suppressed 异常保留；减少样板与遗漏风险。

51. 问：BigDecimal 构造为什么避免 new BigDecimal(double)?  
    思路：二进制浮点转换误差；推荐字符串或 valueOf(double) 使用精确十进制表示。

52. 问：BigDecimal equals 与 compareTo 差异？  
    思路：equals 比较 value+scale；compareTo 忽略 scale；集合 key 要统一 scale。

53. 问：金钱为什么推荐用 long “分” 存储？  
    思路：减少 BigDecimal 分配与算术开销；仅边界转换；热路径性能提升。

54. 问：RoundingMode.HALF_EVEN 为什么用于金融？  
    思路：减少统计累计偏差；长期聚合更公平。

55. 问：不可变对象带来的并发优势？  
    思路：无写入竞争；无需同步；状态推断简单；哈希稳定。

56. 问：Defensive Copy 的典型应用？  
    思路：构造保存外部可变参数副本；返回集合时复制或不可变包装；防止外部修改内部状态。

57. 问：equals/hashCode 常见错误？  
    思路：未同时重写；使用可变字段构成哈希；违反自反/传递；导致集合逻辑错误。

58. 问：偏向锁 / 轻量级锁 / 重量级锁升级路径？  
    思路：无竞争偏向 → CAS 轻量 → 竞争加重锁（操作系统监视器）；依据撤销与膨胀条件。

59. 问：伪共享 (False Sharing) 现象与解决？  
    思路：多线程写同缓存行导致频繁失效；使用分片结构或 @Contended 分离。

60. 问：装箱导致的性能问题表现？  
    思路：频繁分配与 GC；自动拆箱潜在 NPE；== 与 equals 语义不一致（缓存区间）。

61. 问：Stream 中 distinct/sorted 为何称有状态操作？  
    思路：需要维护全部或部分元素集合/排序缓冲；并行额外开销与同步。

62. 问：并行流不当使用导致性能下降的原因？  
    思路：数据量小拆分开销、阻塞 IO 占线程、共享可变状态竞争、顺序语义维护。

63. 问：Collector 的 combiner 作用？  
    思路：并行分段结果合并；顺序流可能调用；需保证线程安全或限制特征。

64. 问：toMap 为什么常抛 IllegalStateException？  
    思路：键重复且无合并函数；需要提供 (k, v1, v2) 解决冲突。

65. 问：Spliterator trySplit 质量影响什么？  
    思路：并行任务负载均衡；过倾斜导致某线程拖尾；特征标识影响优化。

66. 问：Optional 不建议做成员字段的原因？  
    思路：增加层级与序列化冗余；集合嵌套复杂度高；内部可用 null 简化。

67. 问：虚拟线程 (Virtual Thread) 与平台线程区别？  
    思路：调度在 JVM；阻塞 I/O 挂起不占 OS 线程；适合大量阻塞任务；CPU 密集收益有限。

68. 问：Structured Concurrency 解决什么问题？  
    思路：子任务作用域生命周期显式；统一取消与异常聚合；避免孤儿线程。

69. 问：Scoped Values 与 ThreadLocal 对比？  
    思路：只读作用域绑定；生命周期清晰不需 remove；虚拟线程友好；避免泄漏。

70. 问：Foreign Function & Memory API 相比 JNI 优势？  
    思路：更少样板、内存安全检查、性能更好、无手写头文件；结构化 Arena 管理生命周期。

71. 问：Pattern Matching switch 如何改进可维护性？  
    思路：穷举校验 sealed 层次；减少 instanceof + 强转样板；守护条件增加表达力。

72. 问：Record 与普通类 + Lombok 区别？  
    思路：语言级不可变值对象语义；自动生成核心方法；语义清晰无注解处理器开销。

73. 问：为什么不在热路径大量使用反射访问字段？  
    思路：无法被内联优化 + 访问检查 + 查找开销；转换为 MethodHandle / 直接调用。

74. 问：类路径扫描导致启动慢的缓解方式？  
    思路：限定基础包、预构建索引（AOT/Jandex）、延迟加载非关键组件、缓存元数据。

75. 问：半包/粘包问题解决方法？  
    思路：协议设计：长度字段、分隔符、定长头；累积缓冲读取直到满足完整帧。

76. 问：零拷贝 (sendfile/transferTo) 适用场景？  
    思路：大文件/日志/静态资源分发；减少用户态缓冲复制与上下文切换；TLS 时可能退化。

77. 问：DirectByteBuffer 优劣权衡？  
    思路：减少一次复制（直接与内核交互）；分配慢 & 回收依赖 Cleaner；需池化与上限控制。

78. 问：为什么 OP_WRITE 不应常驻 Selector？  
    思路：写就绪几乎一直真；常驻导致空轮询浪费 CPU；仅在有待写数据时注册。

79. 问：背压设计的关键指标？  
    思路：待写队列长度、水位回调、延迟增长、内存占用；策略：限速/拒绝/降级。

80. 问：单例中使用双重检查锁定需要注意什么？  
    思路：volatile 保证可见性与指令重排安全；解释初始化步骤拆分风险。

81. 问：为什么在高并发计数中推荐 LongAdder？  
    思路：分段热点分散减少竞争；最终求和聚合；比 AtomicLong 在高争用下吞吐高。

82. 问：对象逃逸分析对性能优化的意义？  
    思路：JIT 判断对象不逃逸 → 栈上分配 / 标量替换；减少 GC 压力；反射与复杂调用降低可分析性。

83. 问：final 语义与内存屏障影响？  
    思路：构造完成前写入 final 字段，对其他线程可见性保证（安全发布）；禁止后续更改。

84. 问：为什么避免在 equals/hashCode 中使用可变集合字段？  
    思路：哈希桶定位依赖 immutability；修改后找不到元素 → 逻辑错误。

85. 问：当看到大量 ClassLoader 未卸载如何处理？  
    思路：定位持有引用（ThreadLocal/静态缓存）；清除并触发 GC；使用分析工具（jmap/class histogram）。

86. 问：JIT 内联限制与反射关系？  
    思路：反射调用目标不稳定、类型信息晚绑定 → 难内联；MethodHandle + invokeExact 可暴露稳定调用点。

87. 问：为什么 Web 容器热部署易出现内存泄漏？  
    思路：旧 ClassLoader 仍被静态结构/线程持有；类与元数据无法卸载；Metaspace 增长。

88. 问：枚举相比常量类优势？  
    思路：类型安全、可扩展方法、switch 支持、序列化稳定（名称绑定）。

89. 问：Stream 与 for 循环性能差异评估方式？  
    思路：JMH 基准；区分操作类型（简单映射 vs 有状态排序）；并行/顺序对比 p95。

90. 问：为什么推荐统一使用 UTC 存储时间？  
    思路：避免时区/DST 变化影响排序与比较；显示层转换用户时区；减少歧义。

91. 问：SimpleDateFormat 为什么不推荐？  
    思路：线程不安全；需创建或同步；java.time DateTimeFormatter 替代。

92. 问：HttpClient (Java 11) 相比旧 HttpURLConnection 优势？  
    思路：异步 API、WebSocket、易用与可靠；支持 HTTP/2；减少样板。

93. 问：var 局部类型推断潜在风险？  
    思路：可读性下降（不清楚类型）；复杂泛型模糊；建议在显而易见场景使用。

94. 问：为什么在热路径避免频繁创建临时集合？  
    思路：分配与 GC 额外成本；可重用缓冲或使用原始数组；减少逃逸与内存抖动。

95. 问：如何判断并行流收益是否值得？  
    思路：基准与度量：顺序 vs 并行耗时 / CPU 利用率 / 分配情况；满足四条件。

96. 问：ServiceLoader 的使用注意事项？  
    思路：依赖 META-INF/services；上下文 ClassLoader 加载；懒加载迭代；避免启动阶段阻塞扫描超大类路径。

97. 问：为什么不要滥用 ThreadLocal？  
    思路：生命周期难控；内存泄漏（线程池长期存在）；虚拟线程频繁创建增加复制/设置成本；Scoped Values 替代。

98. 问：JSON 反序列化安全防护要点？  
    思路：禁止默认多态、限制深度/集合大小、白名单类型、输入校验、升级漏洞版本。

99. 问：对象头中 Mark Word 的作用？  
    思路：存锁状态、哈希、GC 年龄；锁升级路径管理；调试同步与性能行为。

100. 问：为什么 finalize 已弃用？  
     思路：不可预测执行时机、性能差、资源延迟释放、可能复活对象；使用 try-with-resources + Cleaner 兜底。 

---

## 学习与提升建议
| 阶段 | 目标 | 核心行动 | 产出评估 |
| ---- | ---- | -------- | -------- |
| 基础夯实 | 熟悉语义与复杂度 | 精读官方文档 + JDK 源码核心类(ArrayList/HashMap) | 写出各集合插入/查找/扩容流程图 |
| 进阶理解 | 掌握并发与泛型机制 | 阅读 ConcurrentHashMap、ForkJoinPool、TypeSystem 源码 | 手写简化版 CHM + 正确说明树化与扩容条件 |
| 性能感知 | 建立量化思维 | 使用 JMH/async-profiler/JFR 做基准与火焰图 | 报告：String 拼接/Stream vs for/装箱成本对比 |
| 生态运用 | 框架底层认知 | 观察 Spring / Netty 启动日志 + 断点查看反射/代理路径 | 列表：框架中反射/动态代理使用点与替代方案 |
| 并发深化 | 简化高并发设计 | 编写虚拟线程版本 IO Demo + 背压队列实现 | 对比线程池 vs 虚拟线程吞吐与峰值内存曲线 |
| 可靠性与安全 | 异常/序列化/编码边界 | 制定统一异常层次 + ObjectMapper 安全配置 | 静态检查脚本：扫描裸 getBytes()/默认编码调用 |
| 内存与对象模型 | 了解布局与锁 | 用 JOL 打印对象结构；JFR 采样锁事件 | 分析文档：锁升级触发 + 伪共享改造收益 |
| 语言演进追踪 | 跟进新特性价值 | 阅读相关 JEP（Records/Sealed/Loom/Panama） | 摘要：每特性适用场景 + 当前团队引入决策 |
| 深度实践 | 综合应用 | 设计“高并发日志聚合服务”实现：NIO + 背压 + 虚拟线程 + Record DTO | 指标：p99 延迟、吞吐、CPU、GC 次数、代码行数 |
| 复盘与输出 | 内化与分享 | 内部分享 PPT + 编写最佳实践清单 | 文档被团队引用次数/Review 采纳率 |


(完)