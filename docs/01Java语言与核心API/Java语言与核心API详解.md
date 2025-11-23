# Java 语言与核心 API 面试指南（加强版）

评分说明：5=必须掌握，4=很重要，3=熟练更佳，2=了解即可，1=加分项  
补充说明：下面每个知识点均从“是什么 / 为什么 / 有什么用（怎么用） / 关键细节 / 面试提示”多个角度展开，便于系统化回答追问。面试回答建议：先主线→再细节→再风险与优化→最后真实案例。

---

## 1. 集合框架与数据结构 [5]

### 1.1 集合体系与设计哲学
![集合类](../../assets/images/集合类.png)
- 是什么：Java Collections Framework 通过接口层次（Collection、List、Set、Queue、Map）定义抽象，常见实现（ArrayList、LinkedList、HashSet、TreeSet、PriorityQueue、Deque、HashMap、LinkedHashMap、ConcurrentHashMap 等）按语义与复杂度分层。  
- 为什么：
  - 统一 API 降低学习成本与替换成本；
  - 不同实现在访问模式、顺序保证、并发特性与内存占用上各有取舍。  
- 怎么用：
  - 根据访问模式选择：随机访问→ArrayList；
  - 频繁按键查找→HashMap；
  - 需要排序视图→TreeMap / TreeSet；
  - 需要并发读写→ConcurrentHashMap；
  - 频繁队头/队尾→Deque。  
- 关键细节：
  - fail‑fast 迭代器基于 modCount；
  - 不可变与只读视图区别（List.of 返回真正不可变，Collections.unmodifiableXXX 只是包装）；
  - 迭代中修改必须用 Iterator.remove；
  - 返回集合时进行 defensive copy。  
- 面试提示：快速建立选择矩阵并说明“为什么不是 LinkedList”：强调定位成本 O(n) 与对象额外指针开销；举出一次因错误结构导致性能劣化的改造。

### 1.2 ArrayList vs LinkedList
- 是什么：
  - ArrayList 基于动态数组（连续内存）
  - LinkedList 双向链表（节点分散）。  
- 为什么：
  - 时间局部性与 CPU 缓存友好使 ArrayList 在绝大多数场景占优；
  - 链表优势仅在频繁从已定位的节点附近插入删除且数据量极大时体现。  
- 怎么用：
  - 通用首选 ArrayList；
  - 当需要队列/栈操作考虑 ArrayDeque；
  - 避免使用 LinkedList 做随机插入。  
- 关键细节：
  - ArrayList 扩容系数 1.5（JDK 8）；
  - 扩容需要复制旧数组；LinkedList 每节点对象额外两指针，GC 压力高；
  - 链表迭代无法利用预取。  
- 面试提示：
  - 指出“链表中间插入 O(1) 是误导——前提是已拿到节点引用，查找仍 O(n)”。

### 1.3 HashMap 内部结构与演进
![HahsMap结构](../../assets/images/hashMap.png)
- 是什么：
  - 桶数组 + 链表/红黑树（JDK8：链表长度达到阈值且容量足够时树化）。  
- 为什么：
  - 降低平均查找成本至近似 O(1)；
  - 树化避免高碰撞退化 O(n)。  
- 怎么用：
  - 预估元素数量 (n/负载因子) 设置初始容量减少 resize；
  - 高并发只读可用普通 HashMap + 安全发布；
  - 修改并发需 ConcurrentHashMap。  
- 关键细节：
  - 默认负载因子 0.75；
  - resize 成本集中；树化阈值：
  - 链表长度≥8 且容量≥64；
  - 扰动函数减少低质量 hash 聚集。  
- 面试提示：回答树化条件 + 为什么不一开始用树：空间与维护成本折中 → 展示对演进理解。

### 1.4 ConcurrentHashMap
- 是什么：
  - 并发安全哈希表，JDK8 取消分段锁，采用节点 CAS + 局部 synchronized + 红黑树。  
- 为什么：
  - 减少全表锁争用，提高并发吞吐。  
- 怎么用：
  - 使用 compute / merge / putIfAbsent 原子复合操作代替外部加锁；
  - 频繁统计用 LongAdder 搭配 map 或内置 mappingCount。  
- 关键细节：
  - size() 非严格实时；
  - 树化逻辑与 HashMap 类似；
  - 扩容采用协助迁移（转移任务分配给多个线程）。  
- 面试提示：说明“安全性来自局部锁 + CAS”并指出为何不要在其外层再加 synchronized（扩大临界区）。

### 1.5 CopyOnWrite 家族
- 是什么：写操作复制底层数组（CopyOnWriteArrayList/Set），读操作直接使用旧快照。  
- 为什么：读远多于写时避免锁竞争与迭代不稳定。  
- 怎么用：配置列表、观察者/订阅者集合、系统级白名单。  
- 关键细节：
  - 写成本高 + 短暂双倍内存；
  - 迭代不反映最新写入；
  - 不适合大集合高频更新；
  - 可能导致旧对象滞留增加 GC。  
- 面试提示：给“订阅列表用 CopyOnWrite 减少锁争用”同时强调数据量与写频率限制。

### 1.6 阻塞队列与并发队列
- 是什么：
  - BlockingQueue 提供容量、阻塞与协调；
  - 常见实现：ArrayBlockingQueue（定长）、LinkedBlockingQueue（链表）、SynchronousQueue（零容量）、DelayQueue、PriorityBlockingQueue。  
- 为什么：解耦生产消费速率实现背压与缓冲。  
- 怎么用：
  - 线程池：LinkedBlockingQueue 通用；
  - 高吞吐低延迟：ArrayBlockingQueue；
  - 任务瞬时移交：SynchronousQueue；
  - 延迟调度：DelayQueue。  
- 关键细节：
  - 链表队列入/出两锁减竞争；
  - SynchronousQueue 造成线程直接交接适配动态扩容池；
  - 容量过大隐藏拥塞。  
- 面试提示：区分几个队列特性并说明为何 cachedThreadPool 使用 SynchronousQueue。

### 1.7 跳表与有序视图 (ConcurrentSkipListMap)
![跳表数据结构](../../assets/images/跳表数据结构.png)
- 是什么：多级索引结构，通过随机层级提供近似 O(log n) 有序操作并发友好。  
- 为什么：
  - TreeMap 全局锁限制并发；
  - 跳表允许局部修改。  
- 怎么用：排行榜（按分值有序）、时间窗口逻辑（按时间戳范围扫描）、前缀范围检索。  
- 关键细节：空间换时间；极端随机可能退化但概率极低；删除/插入需维护多层指针。  
- 面试提示：展示选择理由：并发 + 有序 + 可范围扫描。

### 1.8 不可变集合与 Defensive Copy
- 是什么：
  - 不可变集合内容创建后不可更改；
  - 防御性复制避免外部修改内部结构。  
- 为什么：线程安全、减少状态复杂度、避免共享意外修改。  
- 怎么用：
  - 配置对象使用 List.of / Map.of；
  - 返回内部集合前复制；
  - 领域对象设计为不可变。  
- 关键细节：
  - unmodifiableXXX 仅包装底层可变集合；
  - 序列化时不可变视图可能暴露底层引用问题。  
- 面试提示：说明“防御性复制与不可变”在并发下的好处。

### 1.9 常见陷阱与复杂度意识
- 是什么：集合操作的时间/空间复杂度与隐藏成本（扩容、装箱、迭代器失效）。  
- 为什么：热点路径错误结构导致性能灾难。  
- 怎么用：
  - 预估 capacity；
  - 避免在遍历时使用 remove 而不通过迭代器；
  - 频繁包含判断使用 HashSet 而非 List。  
- 关键细节：
  - ArrayList 扩容摊还 O(1)；
  - HashMap resize 为集中高成本；
  - LinkedList 频繁 GC；
  - 装箱集合性能退化。  
- 面试提示：能迅速说明每种结构的典型复杂度并举性能调优案例。

---

## 2. 异常体系与最佳实践 [4]
![异常体系](../../assets/images/Java异常.png)
### 2.1 Checked vs Unchecked 异常分层
- 是什么：
  - Checked 需显式声明或捕获；
  - Unchecked (RuntimeException) 不强制。  
- 为什么：分层提高 API 语义清晰度与调用者处理意愿。  
- 怎么用：
  - 可恢复/资源类异常使用 Checked；
  - 编程错误/前置条件失败使用 Unchecked。  
- 关键细节：
  - 过度 Checked 污染调用链；
  - 合理包装底层异常表达领域语义。  
- 面试提示：说明什么时候把 IOException 包装成 DomainSpecificException。

### 2.2 异常传播与语义保留
- 是什么：重新抛出或包装时保留原始 cause 和上下文。  
- 为什么：排障依赖完整堆栈链。  
- 怎么用：catch → log 一次（必要时）→ wrap 并保留 cause；禁止重复多层 log。  
- 关键细节：日志风暴影响可读性；添加业务标识帮助定位（订单号/用户 ID）。  
- 面试提示：强调“只在边界 log 一次”原则。

### 2.3 性能与误用
- 是什么：异常构造与栈捕获成本高。  
- 为什么：异常应表示异常路径而非普通分支。  
- 怎么用：
  - 使用 Optional / Result 对可预见失败；
  - 循环解析前先校验格式。  
- 关键细节：
  - 大批量解析时抛异常退化性能；
  - 异常频发会被热点编译器视为慢路径。  
- 面试提示：对比“靠异常做控制流”与预检查的延迟差异。

### 2.4 try-with-resources 与 Suppressed
- 是什么：
  - 自动关闭资源避免遗漏；
  - Suppressed 记录关闭时的次要异常。  
- 为什么：减少资源泄漏与被主异常覆盖的次异常丢失。  
- 怎么用：
  - 多个资源并列声明；
  - 在日志中输出 getSuppressed()。  
- 关键细节：
  - 不要混用 try-with-resources + 手动 finally 冗余；
  - 关闭顺序逆序。  
- 面试提示：说明 suppressed 的存在场景。

### 2.5 InterruptedException 处理
- 是什么：线程中断信号用于协作取消。  
- 为什么：忽略中断导致阻塞无法及时停止。  
- 怎么用：
  - catch 后恢复中断（Thread.currentThread().interrupt()）或向上抛出；
  - 在循环中检测 Thread.interrupted()。  
- 关键细节：重置中断标志 vs 清除标志；阻塞方法抛出后标志清除。  
- 面试提示：说明“空 catch”风险与正确恢复方式。

---

## 3. I/O 与 NIO / NIO.2 [4]

### 3.1 阻塞 vs 非阻塞 IO
- 是什么：
  - 阻塞等待数据到达或发送完成；
  - 非阻塞轮询状态 + 事件通知。  
- 为什么：
  - 阻塞模式一连接一线程扩展性差；
  - 非阻塞提高资源利用率。  
- 怎么用：
  - Reactor（选择器监听）+ 业务线程池分离；
  - 结合 Netty。  
- 关键细节：
  - 处理半包/粘包；
  - 非阻塞需要循环读取直到返回 0/-1；
  - 避免忙轮询。  
- 面试提示：描述非阻塞在 10 万连接示例下的线程数优势。

### 3.2 Buffer 与 Channel
- 是什么：
  - Buffer 维护 position/limit/capacity；
  - Channel 抽象双向数据通道。  
- 为什么：
  - 显式状态控制减少复制；
  - 跨多种 IO 类型统一接口。  
- 怎么用：
  - 读取后 flip；写入前确保 position/limit；
  - 使用 DirectByteBuffer 加速网络传输。  
- 关键细节：
  - compact 与 clear 差异；
  - DirectBuffer 分配慢 + 需清楚生命周期；错误 flip 导致空读。  
- 面试提示：解释使用模式并指出常见 Bug（忘记 flip）。

### 3.3 Selector 机制
- 是什么：注册 Channel interestOps，统一轮询 selectionKeys。  
- 为什么：少量线程即可管理大量连接。  
- 怎么用：
  - 边缘触发下循环读到 EAGAIN；
  - 迁移任务避免长时间阻塞 selector 线程。  
- 关键细节：
  - 旧 JDK epoll 空轮询 bug；
  - wakeup 用于跨线程唤醒阻塞 select。  
- 面试提示：说明“为什么 selector 线程不可做复杂业务”。

### 3.4 零拷贝与高性能传输
[参考资料](https://segmentfault.com/a/1190000044068914)
![零拷贝](../../assets/images/零拷贝.webp)
- 是什么：mmap / sendfile / transferTo 让内核缓冲区直接与网卡交互减少用户态复制。  
- 为什么：大文件传输吞吐与 CPU 占用优化显著。  
- 怎么用：FileChannel.transferTo；Kafka 使用 zero-copy；只读索引 mmap。  
- 关键细节：mmap SIGBUS 风险；sendfile 与 TLS 兼容问题；PageCache 命中率。  
- 面试提示：比较传统 read+write 与零拷贝的路径差异。

### 3.5 Scatter/Gather 与协议解析
- 是什么：分散读取到多个 Buffer（头部/体分离）、聚集写入合并多段。  
- 为什么：减少拼接复制与解析复杂度。  
- 怎么用：ByteBuffer[] 数组传递；自定义协议：头（长度/类型）+体分离。  
- 关键细节：需要维护每个 buffer position；适合定长或明确头部场景。  
- 面试提示：说明它如何减少额外数组拼接。

### 3.6 NIO.2 文件 API
- 是什么：
  - Path 取代 File；Files 提供原子操作；
  - WatchService 文件系统事件。  
- 为什么：增强可移植性、错误处理与现代特性（符号链接）。  
- 怎么用：Files.move 原子替换、WatchService 监听配置更新。  
- 关键细节：WatchService 不保证所有事件（平台差异）；需防抖处理。  
- 面试提示：谈监听文件变更的局限。

### 3.7 AIO (Asynchronous IO)
- 是什么：内核完成后回调通知；减少阻塞线程。  
- 为什么：理论上更高并发但实现依赖平台内核支持。  
- 怎么用：
  - Windows 与部分 Linux 场景；
  - Java AIO 在网络不如 Netty 广泛。  
- 关键细节：线程调度与回调上下文；仍需处理背压。  
- 面试提示：说明 AIO 不一定比 NIO 更好（成熟度与生态）。

---

## 4. 函数式与 Stream API [4]

### 4.1 Lambda 与闭包
- 是什么：以简洁语法表达行为单元；捕获外部“有效 final”变量形成闭包。  
- 为什么：减少样板，提升可读性与抽象级别。  
- 怎么用：集合变换 map/filter；策略替换；简化匿名内部类。  
- 关键细节：捕获外部对象延长生命周期；避免在热路径频繁装箱/分配。  
- 面试提示：指出合理使用与避免复杂多层嵌套。

### 4.2 流式管道惰性
- 是什么：中间操作构建描述，终止操作触发整体执行。  
- 为什么：可进行短路优化（anyMatch）、融合、延迟开销。  
- 怎么用：链式 map/filter/collect；短路匹配大型集合减少遍历。  
- 关键细节：有状态操作（sorted/distinct）需缓存中间数据；多次终止操作需重新创建流；避免滥用复杂链影响调试。  
- 面试提示：解释“distinct 大数据集内存膨胀”原因。

### 4.3 并行流陷阱
- 是什么：使用公共 ForkJoinPool 并行拆分任务。  
- 为什么：不当使用导致线程池污染、阻塞传播、性能退化。  
- 怎么用：仅在纯 CPU 密集、数据量较大且操作无共享状态时启用；或自定义 ForkJoinPool。  
- 关键细节：阻塞操作会占用池线程；拆分不均匀导致负载倾斜；装箱/同步结构影响收益。  
- 面试提示：列举“并行流在 IO 密集场景变慢”例子。

### 4.4 Collector 与自定义聚合
- 是什么：收集器定义供应器、累加器、组合器，支持并行安全聚合。  
- 为什么：扩展 reduce 类能力实现复杂分组与统计。  
- 怎么用：groupingBy、partitioningBy、mapping、reducing；自定义 Collector.of(...)。  
- 关键细节：组合器需无副作用；toMap 需提供合并函数避免键冲突异常。  
- 面试提示：说明自定义 collector 并行下注意事项。

### 4.5 Spliterator
- 是什么：可拆分迭代器提供特征 (SIZED/SORTED/CONCURRENT) 影响并行策略。  
- 为什么：提升并行划分效率；避免不均衡工作。  
- 怎么用：实现 trySplit 控制粒度；用于非集合数据源。  
- 关键细节：特征标志影响优化（ORDERED 保留顺序成本）。  
- 面试提示：说明为什么不正确 trySplit 导致性能差。

### 4.6 Optional 使用规范
- 是什么：表示可能为空的值的显式容器。  
- 为什么：提升语义与减少 NullPointer。  
- 怎么用：链式 map/flatMap/filter/orElseThrow；避免 get()。  
- 关键细节：不用于集合元素与字段（序列化成本与冗余）；orElse 与 orElseGet 区别（前者总执行参数）。  
- 面试提示：强调合理用法与限制。

---

## 5. 序列化与 JSON 生态 [3]

### 5.1 Java 原生序列化问题
- 是什么：默认对象图到二进制格式机制。  
- 为什么：性能低、不可演进、存在反序列化攻击。  
- 怎么用：避免在分布式或缓存中使用；改用 JSON / ProtoBuf / Kryo。  
- 关键细节：serialVersionUID 不匹配抛异常；不安全 Gadget 链。  
- 面试提示：列举安全风险与替代方案。

### 5.2 Externalizable vs Serializable
- 是什么：Externalizable 自定义读写，Serializable 自动。  
- 为什么：可精确控制字段与版本兼容。  
- 怎么用：writeExternal/readExternal 显式序列化；保持顺序一致。  
- 关键细节：忘记字段导致数据丢失；需处理异常；性能优于默认反射路径。  
- 面试提示：说明选择 Externalizable 的时机。

### 5.3 JSON 框架对比
- 是什么：Jackson（特性丰富与 streaming）、Gson（简单但反射多）、Fastjson（性能快历史安全事件）。  
- 为什么：选型需兼顾性能、生态、稳定与安全。  
- 怎么用：统一 ObjectMapper 配置（禁 Unknown Properties、指定日期格式）。  
- 关键细节：复用 ObjectMapper；启用 Afterburner；禁用默认多态。  
- 面试提示：说明为什么不频繁创建 ObjectMapper。

### 5.4 多态与类型信息
- 是什么：反序列化恢复具体类型的机制（@JsonTypeInfo 或手动 type 字段）。  
- 为什么：需要抽象集合中存储不同子类实例。  
- 怎么用：显式白名单子类型；安全封装。  
- 关键细节：启用默认类型信息有 RCE 风险；保持向后兼容字段。  
- 面试提示：强调“默认多态风险 + 白名单策略”。

### 5.5 兼容与演进
- 是什么：字段增加/删除/默认值策略保证老客户端可解析。  
- 为什么：分布式系统版本不一致常态。  
- 怎么用：添加新字段设默认值；避免重命名；使用 wrapper 版本字段。  
- 关键细节：向后兼容优先；删除字段需双阶段（弃用→移除）。  
- 面试提示：描述一次安全演进过程。

### 5.6 性能优化
- 是什么：降低序列化/反序列化 CPU 与内存。  
- 为什么：高 QPS 场景 JSON 解析成为热点。  
- 怎么用：开启 streaming API；关闭不必要特性；预创建常用序列化器。  
- 关键细节：深层嵌套对象增加栈与复制；Large String 高分配。  
- 面试提示：说明优化前后延迟对比。

### 5.7 安全防护
- 是什么：限制类、深度、集合大小；输入校验。  
- 为什么：防止内存膨胀或 gadget 攻击。  
- 怎么用：配置最大深度/长度；关闭默认类型推断；只解析预期字段。  
- 关键细节：保持安全基线审计；CVE 及时升级。  
- 面试提示：举反序列化漏洞预防措施。

---

## 6. 日期与时间 API (java.time) [3]

### 6.1 核心类型语义
- 是什么：Instant（UTC 时间点）、LocalDateTime（无时区）、ZonedDateTime（含时区与 DST）、OffsetDateTime（固定偏移）。  
- 为什么：避免旧 API (Date/Calendar) 线程不安全与表达混乱。  
- 怎么用：存储统一使用 Instant/UTC；展示转换为用户时区；区分业务本地日历 vs 全局时间点。  
- 关键细节：DST 造成小时重复/跳过；LocalDateTime 不适合跨时区比较。  
- 面试提示：解释“定时任务在 DST 跳变日如何处理”。

### 6.2 不变性与线程安全
- 是什么：所有 java.time 类型不可变。  
- 为什么：并发安全且简化共享。  
- 怎么用：链式操作返回新对象；在函数中直接传递引用。  
- 关键细节：避免在循环中创建无意义新对象；偏向使用 Duration/Period 描述差值。  
- 面试提示：对比旧 SimpleDateFormat 线程不安全。

### 6.3 格式与解析
- 是什么：DateTimeFormatter 提供线程安全格式化与解析。  
- 为什么：避免 SimpleDateFormat 需要同步。  
- 怎么用：预定义格式常量；设置 Locale 与 Zone。  
- 关键细节：大小写模式与 Locale 差异；解析失败抛异常。  
- 面试提示：说明默认 Locale 与国际化影响。

### 6.4 时区转换策略
- 是什么：统一内部 UTC 存储，界面展示根据用户偏好转换。  
- 为什么：减少跨时区计算混乱与 DST 问题。  
- 怎么用：Instant → ZonedDateTime.ofInstant(…, zone)；数据库存时间戳 + 时区。  
- 关键细节：避免存 LocalDateTime 表示绝对时间；时区数据库更新。  
- 面试提示：强调“UTC 存储 + 展示转换”模式。

### 6.5 Clock 抽象
- 是什么：Clock 提供可替换时间源。  
- 为什么：测试可控制当前时间；避免依赖系统时间。  
- 怎么用：业务层注入 Clock；测试注入 fixed/offset。  
- 关键细节：避免在测试中使用 System.currentTimeMillis 导致不稳定；nanoTime 用于间隔度量。  
- 面试提示：举利用固定 Clock 测试过期逻辑示例。

---

## 7. 字符串与编码处理 [4]

### 7.1 String不可变性
- 是什么：
- 为什么：
  - final修饰class定义
  - final修饰value属性，内容不可变
  - 不提供修改value内容的方法
- 怎么用：
- 关键细节：
- 面试提示：

### 7.2 编码与内存
- 是什么：JDK9 Compact Strings：Latin1（单字节）或 UTF‑16（双字节）内部优化。  
- 为什么：降低内存占用与 GC 压力。  
- 怎么用：无需手动干预但避免频繁创建临时字符串。  
- 关键细节：String 不可变 + 常量池；concat 产生新对象。  
- 面试提示：解释不可变性对安全与缓存好处。

### 7.3 拼接与性能
- 是什么：编译器将常量拼接优化为单次；循环中多个 + 退化。  
- 为什么：频繁拼接产生大量中间对象。  
- 怎么用：热点循环使用 StringBuilder 或预分配 char[]；日志使用占位符。  
- 关键细节：StringBuilder 默认容量 16；扩容机制复制开销。  
- 面试提示：给“拼接从 + 改到 StringBuilder 减少 GC”案例。

### 7.4 intern
- 是什么：将字符串放入全局常量池以复用。  
- 为什么：减少重复常量对象占用内存。  
- 怎么用：少量高频重复值适当 intern；避免动态海量值。  
- 关键细节：常量池容量受限；滥用增加查找成本。  
- 面试提示：说明 intern 不是万能优化。

### 7.5 编码显式控制
- 是什么：IO 转换时显式 charset 避免系统默认。  
- 为什么：默认编码跨平台不一致导致乱码。  
- 怎么用：使用 StandardCharsets.UTF_8；明确读取文件时编码。  
- 关键细节：避免 new String(bytes) 无 charset；网络协议需统一编码约定。  
- 面试提示：指出“默认编码风险”案例。

### 7.6 字符串常量池
- 是什么：
- 为什么：
- 怎么用：
- 关键细节
- 面试提示：
---

## 8. 泛型与类型擦除 [5]

### 8.1 擦除机制
- 是什么：编译期检查类型安全，运行时擦除到原始类型（Object 或上界）。  
- 为什么：保持与无泛型旧字节码兼容。  
- 怎么用：通过 Class<T> 传递类型；使用 TypeToken 捕获泛型参数。  
- 关键细节：无法直接获取 T 的 Class；桥接方法维持多态签名。  
- 面试提示：举“List<String> 与 List<Integer 在运行时类型相同”展示擦除。

### 8.2 通配符与 PECS
- 是什么：Producer Extends / Consumer Super 原则约束读写方向。  
- 为什么：保持类型安全与灵活性。  
- 怎么用：读取使用 ? extends T；写入使用 ? super T。  
- 关键细节：? super 写时可添加 T 或其子类，读出只能 Object；extends 只能安全读。  
- 面试提示：快速构造示例 List<? super Number> 添加 Integer。

### 8.3 上界与多重约束
- 是什么：<T extends ClassA & InterfaceB & InterfaceC> 控制能力集合。  
- 为什么：确保泛型对象具备所需方法。  
- 怎么用：在算法/框架层对泛型做能力限制。  
- 关键细节：类在最前；不支持多个类。  
- 面试提示：说明结合 Comparable 限制排序能力。

### 8.4 数组与泛型协变差异
- 是什么：数组协变（Sub[] 可赋给 Super[]）+ 运行时类型检查；泛型不协变（需通配符）。  
- 为什么：数组协变存在潜在运行时异常；泛型通过编译期约束提高安全。  
- 怎么用：避免使用泛型数组；改用 List。  
- 关键细节：new T[] 不允许；需使用 @SuppressWarnings 谨慎处理。  
- 面试提示：说明“泛型数组危险”原因。

### 8.5 桥接方法
- 是什么：编译器生成合成方法维持重写与泛型签名一致。  
- 为什么：保证运行时多态调用正确。  
- 怎么用：了解反射看到额外方法不困惑。  
- 关键细节：Method.isSynthetic 区分。  
- 面试提示：简述桥接方法在 Covariant Return 中作用。

### 8.6 设计原则
- 是什么：泛型设计目标：类型安全 + 简洁 + 可读。  
- 为什么：过度嵌套影响维护（Map<String, List<Map<K,V>>>）。  
- 怎么用：抽离值对象；封装泛型参数；减少多层嵌套。  
- 关键细节：通配符 vs 具体类型平衡；提供 builder 简化调用。  
- 面试提示：展示一次简化复杂泛型 API 案例。

---

## 9. 反射与注解机制 [4]

### 9.1 ClassLoader
是什么：类加载器层级树（Bootstrap → Extension → Application → 自定义），决定类型唯一性。  
为什么：模块隔离、热替换、插件系统。  
怎么用：自定义 URLClassLoader / Layer；使用 Thread Context ClassLoader 进行 SPI 加载。  
关键细节：同名类在不同 ClassLoader 下视为不同类型；内存泄漏（ClassLoader 未释放）。  
面试提示：说明“上下文 classloader 在服务发现中的作用”。

### 9.2 反射性能与缓存
是什么：运行时动态获取字段/方法/构造并调用。  
为什么：灵活但有性能与安全成本。  
怎么用：启动阶段缓存 Method/Field 并关闭 setAccessible；使用 MethodHandle 提升性能。  
关键细节：过度反射影响 JIT 内联；AccessibleObject 安全限制（Java 17 强封装）。  
面试提示：描述从反射迁移到 MethodHandle 的优化。

### 9.3 MethodHandle 与 invokedynamic
是什么：更底层的、可被 JIT 优化的调用机制；invokedynamic 支撑 Lambda/字符串拼接等。  
为什么：减少反射开销并提高可优化空间。  
怎么用：使用 lookup.findVirtual/findStatic；缓存句柄。  
关键细节：调用链短；参数签名严格；适合框架底层。  
面试提示：说明框架采用它的性能理由。

### 9.4 注解保留策略
是什么：@Retention(SOURCE/CLASS/RUNTIME) 控制生命周期；@Target 约束使用位置。  
为什么：避免不必要运行时保留；减少扫描范围。  
怎么用：框架扩展使用 RUNTIME；编译检查使用 SOURCE。  
关键细节：重复注解与元注解组合；自定义处理器。  
面试提示：列出一个自定义注解完整元注解集合。

### 9.5 扫描与启动成本
是什么：启动期反射/类路径扫描耗时高。  
为什么：影响冷启动和资源使用。  
怎么用：缩小扫描包范围；预生成索引（Spring AOT/Quarkus）；延迟初始化。  
关键细节：Native Image 限制反射需配置；大量反射影响内存。  
面试提示：给“预索引启动时间减少”数据。

### 9.6 动态代理
是什么：JDK Proxy 基于接口，CGLIB 基于子类（不能拦截 final 方法）。  
为什么：实现横切（事务、缓存、日志）。  
怎么用：确保调用通过代理（自调用失效）；选择库时考虑 final 类限制。  
关键细节：事务失效多源于自调用绕过代理；CGLIB 生成类数量控制。  
面试提示：说明 @Transactional 自调用问题与解决（切分方法或注入自身代理）。

---

## 10. Optional 最佳实践 [3]

### 10.1 语义定位
是什么：表示“可能缺失”的返回值。  
为什么：替代返回 null 的潜在风险，提高表达力。  
怎么用：服务层返回 Optional；链式处理。  
关键细节：不作为字段、序列化成本高；不用于集合元素。  
面试提示：强调其使用边界与正确场景。

### 10.2 链式操作
是什么：map/flatMap/filter 对值操作组合。  
为什么：避免嵌套 if null。  
怎么用：opt.filter(...).map(...).orElse(...); orElseGet 惰性。  
关键细节：避免 Optional.of(null) 导致 NPE；使用 ofNullable。  
面试提示：区分 orElse vs orElseGet 计算时机。

### 10.3 性能与替代
是什么：包装对象带开销。  
为什么：热点循环中大量创建影响 GC。  
怎么用：内部逻辑可用注释 + null；对外 API 用 Optional。  
关键细节：统一规范避免混乱；不要链式过度复杂。  
面试提示：说明性能权衡。

---

## 11. 新语言特性 (Record / Sealed / Pattern Matching) [3]

### 11.1 Record
是什么：数据载体语法糖（自动 equals/hashCode/toString/accessor）。  
为什么：减少样板代码并提供不可变建模。  
怎么用：record User(id, name)；使用 compact constructor 校验。  
关键细节：字段 final；可定义自定义方法；序列化兼容关注。  
面试提示：说明使用场景：DTO/配置对象。

### 11.2 Sealed
是什么：限制允许的子类增强穷举性与安全控制。  
为什么：模式匹配方便全面处理（未来 switch）。  
怎么用：sealed interface Shape permits Circle, Rect。  
关键细节：子类需 final / sealed / non-sealed；跨模块限制。  
面试提示：说明对 API 控制的价值。

### 11.3 Pattern Matching
是什么：instanceof 后自动类型绑定与 switch 模式匹配（JDK 演进）。  
为什么：减少显式强制转换与样板。  
怎么用：if (obj instanceof User u) { ... }；switch(pattern)。  
关键细节：match 顺序；预览特性版本变化。  
面试提示：谈语言可读性改进。

---

## 12. 数值与精度 (BigDecimal) [3]

### 12.1 精度与构造
是什么：BigDecimal 提供任意精度十进制。  
为什么：避免二进制浮点误差在金额场景。  
怎么用：BigDecimal.valueOf(double) 或字符串构造；避免 new BigDecimal(double)。  
关键细节：scale 与 value 区分；equals 考虑 scale。  
面试提示：展示误差例子 (new BigDecimal(0.1))。

### 12.2 舍入与 scale
是什么：RoundingMode 控制除法/格式化行为。  
为什么：财务计算规范要求；不同模式对结果影响显著。  
怎么用：setScale(scale, RoundingMode.HALF_UP)；统一常量。  
关键细节：compareTo 忽略 scale；HALF_EVEN 减少统计偏差。  
面试提示：说明选择 HALF_EVEN 的原因。

### 12.3 性能
是什么：高精度运算创建大量对象。  
为什么：热点路径需优化减少分配。  
怎么用：缓存常量；提前乘除；使用原始 long 表示分转换。  
关键细节：作为 Map key 性能差；注意 equals/scale。  
面试提示：给转化为“分”存储提升性能案例。

---

## 13. 对象与不可变设计 [4]

### 13.1 不可变好处
是什么：状态固定对象。  
为什么：降低并发复杂度；缓存与哈希稳定。  
怎么用：字段 final + 无 setter；构造完成即有效。  
关键细节：内部数组 defensive copy；确保深层不可变。  
面试提示：举值对象示例（Money、UserId）。

### 13.2 Defensive Copy
是什么：复制输入或输出防止外部篡改内部状态。  
为什么：保护封装与线程安全。  
怎么用：Collections.unmodifiableList + copy；返回新对象。  
关键细节：Date、数组等引用类型风险高。  
面试提示：说明未复制导致的真实 Bug。

### 13.3 equals/hashCode 合约
是什么：相等对象必须同 hash；hash 不必唯一。  
为什么：影响 HashMap/HashSet 正常工作。  
怎么用：基于不可变字段实现；重写时保持对称传递性。  
关键细节：浮点 NaN；BigDecimal equals 与 compareTo 差异。  
面试提示：描述错误实现导致元素丢失案例。

---

## 14. 设计与 API 可用性 [2]

### 14.1 语义清晰
是什么：命名与参数表达业务意图。  
为什么：提高可维护性减少误用。  
怎么用：方法名使用动词 + 领域对象；避免 boolean flag 抽象成策略对象。  
关键细节：重载导致歧义；拒绝“doProcess”泛化。  
面试提示：讲一次重构命名改善理解。

### 14.2 参数验证
是什么：前置条件校验保护方法不被非法参数调用。  
为什么：失败快速暴露错误而非静默失败。  
怎么用：Objects.requireNonNull / Preconditions；抛出合理异常。  
关键细节：避免晚期 NullPointer；区分 IllegalArgument vs IllegalState。  
面试提示：强调前置校验+清晰异常提高排障效率。

---

## 15. 资源管理与 AutoCloseable [3]

### 15.1 try-with-resources
是什么：编译器生成自动关闭语句的结构。  
为什么：避免资源泄漏与重复模板。  
怎么用：多个资源分号分隔；只针对实现 AutoCloseable。  
关键细节：关闭顺序逆序；异常抑制 suppressed。  
面试提示：对比手动 finally 漏关风险。

### 15.2 Cleaner 取代 finalize
是什么：Cleaner 提供更可控的回收回调替代 finalize。  
为什么：finalize 不确定性与性能差。  
怎么用：注册清理逻辑释放 native 资源。  
关键细节：仍非实时；优先显式关闭。  
面试提示：说明弃用 finalize 的原因。

---

## 16. 内存与对象模型基础 [3]

### 16.1 对象头
是什么：Mark Word（锁状态、哈希、年龄）+ Klass Pointer；压缩指针减少空间。  
为什么：理解锁升级与监控工具输出。  
怎么用：排查锁竞争、对象哈希分布。  
关键细节：偏向锁 → 轻量级 → 重量级升级；sync 竞争路径。  
面试提示：解释锁状态转换。

### 16.2 对齐与填充
是什么：对象按 8 字节对齐；字段可能填充。  
为什么：提高访问性能但浪费少量空间。  
怎么用：结构紧凑字段排序（大类型靠前可减少填充）。  
关键细节：@Contended 避免伪共享；过度填充浪费。  
面试提示：说明伪共享与填充关系。

### 16.3 装箱与缓存
是什么：基本类型包装对象与缓存范围（Integer[-128,127]）。  
为什么：频繁装箱产生垃圾与性能退化。  
怎么用：使用原始类型或 LongAdder；避免在热路径频繁自动装箱。  
关键细节： == 比较包装对象需谨慎；拆箱 NPE 风险。  
面试提示：展示装箱热点对 GC 的影响。

---

## 常见面试抽象问题速答
1. ConcurrentHashMap 扩容机制：多线程协助迁移 + 标记节点，避免全表锁。  
2. parallelStream 退化原因：数据小、阻塞 IO、共享状态、拆分不均、竞争 ForkJoinPool。  
3. 原生序列化风险：性能差 + 版本不兼容 + Gadget 反序列化 RCE。  
4. ZonedDateTime vs OffsetDateTime：前者含时区规则与 DST，后者仅固定偏移。  
5. ? super vs ? extends：Producer Extends / Consumer Super 原则确保读写方向。  

---

## 学习与提升建议
- 源码阅读：HashMap、ConcurrentHashMap、ArrayList、Stream、ForkJoinPool。  
- 性能基准：JMH 验证集合扩容 & Stream vs for、装箱成本。  
- 异常治理：建立统一异常层次与日志策略。  
- 时间迁移：使用 java.time 替换 Date/Calendar，统一 UTC 存储策略。  
- 安全序列化：统一 ObjectMapper 配置，禁用默认多态与限制深度。  

## 重要度汇总
- 集合框架 [5]  
- 泛型与类型擦除 [5]  
- 异常体系 [4]  
- I/O / NIO [4]  
- 函数式与 Stream [4]  
- 字符串与编码 [4]  
- 反射与注解 [4]  
- 不可变对象设计 [4]  
- 序列化与 JSON [3]  
- 日期时间 API [3]  
- Optional [3]  
- BigDecimal 与数值 [3]  
- 资源管理 (AutoCloseable) [3]  
- 内存与对象模型 [3]  
- 新特性 (Record/Sealed/Pattern Matching) [3]  
- API 可用性 [2]  

(完)