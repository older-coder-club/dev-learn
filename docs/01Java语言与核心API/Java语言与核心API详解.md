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
- 是什么：阻塞模式调用阻塞线程直到数据可用；非阻塞配合 Selector 轮询准备就绪通道；内核层仍可能阻塞在少数系统调用。  
- 为什么：阻塞一连接一线程，线程堆栈+调度+上下文切换成本高；非阻塞减少线程、提升可扩展性（十万长连接）。  
- 怎么用：单 Reactor（少量 Selector 线程）分发事件 → 业务工作线程池; 更高并发可用多 Reactor（主接收 + 子处理）。  
- 关键细节：非阻塞 read/write 返回 0/部分读；需循环直到读完或返回 -1；粘包/半包需协议边界（长度字段/分隔符/固定头）。  
- 常见陷阱：忙轮询（未正确处理 select 阻塞与兴趣集合更新）；错误假设一次 read 得到完整消息；线程在 Selector 回调中执行长逻辑阻塞其他连接。  
- 面试提示：举“10 万连接：阻塞模式需近似 10 万线程 vs 非阻塞几十线程”并说明内存与上下文切换差异。  
- 案例：改造阻塞 BIO 聊天服务器 → NIO 后峰值线程从 6000 降到 96，CPU 利用率下降 35%。

### 3.2 Buffer 与 Channel
- 是什么：Channel 抽象数据源/目的（FileChannel, SocketChannel）；Buffer 封装内存区并维护 position/limit/capacity 状态。  
- 为什么：明确状态转移避免隐式复制；允许直接内存（DirectByteBuffer）减少内核交互开销。  
- 怎么用：读入后 flip → 消费 → compact/clear；写出前确保 position 指向有效数据起点；大块传输选 DirectBuffer。  
- 关键细节：clear 重置 position=0 limit=capacity；compact 保留未读数据移动至前部；DirectBuffer 分配昂贵（避免频繁创建，池化）；忘记 flip 导致空读。  
- 常见陷阱：混淆 limit 与 capacity；重复 flip 破坏边界；DirectBuffer 未释放（等待 GC + Cleaner 导致延迟）。  
- 面试提示：说明“flip = 写模式→读模式状态切换”与最典型 Bug。  
- 案例：日志队列使用堆缓冲频繁拷贝 → 改 DirectByteBuffer 池化吞吐提升 18%。

### 3.3 Selector 机制
- 是什么：单线程轮询注册 Channel 的兴趣事件(OP_READ/OP_WRITE/OP_ACCEPT/OP_CONNECT) 并分发处理。  
- 为什么：少量线程即可处理大量就绪事件，提高可扩展性与资源利用效率。  
- 怎么用：select()/selectTimeout→遍历 selectedKeys→处理→移除 key（或调用 iterator.remove）；写事件只在“待写且缓冲未清空”时注册。  
- 关键细节：旧 epoll 空轮询 bug（需 JDK 修复或 select 降级策略）；wakeup() 跨线程唤醒阻塞 select；处理过程中避免阻塞（业务线程池分离）。  
- 常见陷阱：长期保持 OP_WRITE 导致空转；selectedKeys 不移除造成重复处理；在 Selector 线程执行耗时数据库调用。  
- 面试提示：强调“Selector 线程只负责 IO 就绪分发”。  
- 案例：支付网关曾在 Selector 回调直接调用外部服务导致整体吞吐骤降，拆分后恢复。

### 3.4 零拷贝与高性能传输
- 是什么：sendfile / transferTo / mmap 让数据在内核缓冲与网卡之间传递，减少用户态复制与上下文切换。  
- 为什么：大文件/日志分发场景显著降低 CPU 消耗，提高吞吐。  
- 怎么用：FileChannel.transferTo(socketChannel); 索引/只读文件 mmap 建立映射直接访问。  
- 关键细节：TLS 情况下传统 sendfile 可能退化为常规路径；mmap 内存映射文件被截断可能抛 SIGBUS；PageCache 命中对性能关键。  
- 常见陷阱：误将 mmap 当通用优化（小文件不一定收益）；忽略文件大小变化；不关闭映射对象（引用未释放）。  
- 面试提示：画数据路径对比：read+write(4 次拷贝) vs sendfile(2 次)。  
- 案例：文件分发系统由循环 read/write 改 transferTo，CPU 下降 30%，吞吐提升 25%。

### 3.5 Scatter/Gather 与协议解析
- 是什么：Scatter 将数据流分散读入多个 Buffer（如头部+体）；Gather 将多个 Buffer 合并写出。  
- 为什么：避免手动拆分/拼接，减少数组复制，提高解析清晰度。  
- 怎么用：channel.read(new ByteBuffer[]{header, body}); 写出时 channel.write(buffers)。  
- 关键细节：适合固定/可预知头部长度；维护各 Buffer 的 position/remaining；部分写需循环继续。  
- 常见陷阱：假设一次 write 全部发送；未处理半写导致协议断裂。  
- 面试提示：描述“头部+体分离减少额外数组合并”。  
- 案例：二进制协议加入长度字段+Scatter读取，序列化开销下降。

### 3.6 NIO.2 文件 API
- 是什么：Path/Files 取代旧 File；提供原子移动、符号链接、权限；WatchService 监听变化。  
- 为什么：改进错误处理与跨平台一致性。  
- 怎么用：Files.move(temp, target, ATOMIC_MOVE, REPLACE_EXISTING); WatchService 监听配置目录。  
- 关键细节：WatchService 事件合并与丢失可能（需批处理与防抖）；文件权限 POSIX API 在不同平台差异；大目录遍历使用 Files.walk。  
- 常见陷阱：监听大量目录未关闭 WatchKey；递归遍历忘记关闭 Stream。  
- 面试提示：说明监听局限与防抖策略。  
- 案例：配置中心使用 WatchService 结合校验 + 延迟 200ms 合并事件。

### 3.7 AIO (Asynchronous IO)
- 是什么：CompletionHandler / Future 模式，IO 完成后回调或通知，不阻塞等待。  
- 为什么：进一步减少线程等待；但生态成熟度与性能优势有限。  
- 怎么用：AsynchronousSocketChannel.connect / read / write with CompletionHandler。  
- 关键细节：回调嵌套复杂度高；需自建错误处理链；背压控制困难。  
- 常见陷阱：回调中直接发起下一层深度形成“回调地狱”；忽略异常路径导致连接泄漏。  
- 面试提示：解释“在主流高性能服务中仍多选 Netty(NIO)”。  
- 案例：尝试 AIO 后复杂度 > NIO，无明显收益回退。

### 3.8 Netty 模型简述
- 是什么：事件驱动异步网络框架，封装 Selector、Pipeline、ByteBuf。  
- 为什么：减少样板及错误，提供高性能内存管理与编解码。  
- 怎么用：Bootstrap+EventLoopGroup；ChannelPipeline 添加编解码器与业务处理器。  
- 关键细节：ByteBuf 引用计数避免内存泄漏；单线程事件循环保证 handler 内无需锁（除跨线程共享）；可定制内存池。  
- 常见陷阱：未释放 ByteBuf（LEAK 报告）；阻塞 EventLoop 导致心跳延迟。  
- 面试提示：说明“为什么不要在 handler 里调用长阻塞操作”。  
- 案例：心跳延迟报警 → 将耗时计算移出 EventLoop 后恢复正常。

### 3.9 背压与流控
- 是什么：在生产者-消费者速率不匹配时限制上游速度避免堆积。  
- 为什么：无背压会导致内存膨胀/延迟激增。  
- 怎么用：限制待写队列大小；监控 channel.isWritable()；结合应用级令牌/窗口协议。  
- 关键细节：写缓冲高水位/低水位回调；拒绝或丢弃策略需记录度量。  
- 常见陷阱：只监控 CPU 忽略积压队列；无降级策略强行积压。  
- 面试提示：说明“背压 = 控制输入速率 + 缩短排队时间”。  
- 案例：增加写队列水位监控后避免内存突增 OOM。

### 3.10 文件与网络缓冲调优
- 是什么：合理设置 OS 与 JVM 层缓冲减少系统调用与上下文切换。  
- 为什么：缓冲过小频繁调用，过大浪费内存。  
- 怎么用：SocketChannel 套接字缓冲（SO_SNDBUF/SO_RCVBUF）；DirectBuffer 池化；JVM -XX:MaxDirectMemorySize 控制上限。  
- 关键细节：Linux rmem/wmem 上限可能限制套接字缓冲；避免与应用层重复过度缓存。  
- 常见陷阱：盲目调巨大缓冲导致 GC 或内存占用增加；忽略平台限制导致设置无效。  
- 面试提示：提到“测量再调优”与工具 netstat / ss / perf。  
- 案例：调整 SO_SNDBUF 后批量发送延迟下降 12%。

### 3.11 传输层常用参数
- 是什么：TCP_NODELAY、SO_KEEPALIVE、SO_REUSEADDR、SO_LINGER。  
- 为什么：控制延迟、连接生存与端口重用。  
- 怎么用：短小延迟敏感请求关闭 Nagle：setTcpNoDelay(true)；长连接打开 KeepAlive；快速重启用 SO_REUSEADDR。  
- 关键细节：Nagle+Delayed ACK 组合导致双倍 RTT；KeepAlive 默认间隔大（需 OS sysctl 调整）。  
- 常见陷阱：错误开启 SO_LINGER 导致关闭阻塞；忽略 TIME_WAIT 导致端口耗尽。  
- 面试提示：说明“小包 RPC → 关闭 Nagle”。  
- 案例：关闭 Nagle 后 p99 延迟由 40ms 降到 27ms。

### 3.12 性能度量与工具
- 是什么：定位 IO 瓶颈与分配热点工具链。  
- 怎么用：iostat / vmstat / pidstat / perf / async-profiler / JFR / strace。  
- 关键细节：区分 CPU vs IO wait；火焰图看阻塞堆栈；JFR 低入侵采样。  
- 常见陷阱：将 IO wait 误判为 CPU 计算；只看平均不关注 p99。  
- 面试提示：给出“指标→采样→火焰图→改代码”路径。  
- 案例：识别频繁小包写 → 批量聚合传输吞吐提升。

### 3.13 高频问答速记
| 问题 | 速答 |
| ---- | ---- |
| 为什么非阻塞扩展性好 | 减少一连接一线程 → 降低调度与栈内存成本 |
| flip 的作用 | 将写模式转换为读模式：limit=position, position=0 |
| 零拷贝价值 | 减少用户态复制与上下文切换，传输大文件吞吐提升 |
| 半包处理策略 | 协议长度字段 / 定界符 / 累积缓冲区检测完整性 |
| Selector 线程能否执行业务 | 不建议，会阻塞其他 IO 事件分发 |
| DirectBuffer 优劣 | 访问快减少拷贝 / 分配慢需池化管理 |
| 为什么 sendfile 与 TLS 有时退化 | 加密需要用户态处理导致无法完全绕过复制 |
| 为什么 OP_WRITE 不常驻 | 仅在写缓冲未清空时注册，避免空轮询 |

### 3.14 最佳实践清单
- Selector 线程只做 IO 分发与轻量编解码。  
- 使用长度字段或分隔符解决粘包半包。  
- 池化并重用 DirectByteBuffer；监控直接内存上限。  
- 注册 OP_WRITE 时机：写缓冲未清空才注册，完成后取消。  
- 零拷贝优先用于大文件与日志分发；小数据不必强求。  
- 参数调优基于度量：先抓指标再调整缓冲 / TCP 选项。  
- 有背压：检测队列与写水位，拒绝或限速而不是无限排队。  
- 统一编码与解码层（ByteBuf/Buffer 管理），避免复制。  
- 文件事件监听处理防抖与补偿。  
- 生产基准：记录 p99/p999 延迟与吞吐，持续回归。  

---

## 4. 函数式与 Stream API [4]

### 4.1 Lambda 与闭包
- 是什么：以简洁语法表示单一函数接口实例；捕获外围“有效 final”变量形成闭包。
- 为什么：减少匿名内部类样板；提高抽象层次与可组合性；支持行为参数化（策略、回调）。
- 怎么用：集合处理 map/filter/sort；替换 Comparator/Predicate/Function；异步回调 (CompletableFuture.thenApply)；策略表 (Map<String, Runnable>).
- 关键细节：
  - 捕获变量必须“有效 final”，否则编译失败。
  - 捕获引用而非值；被捕获对象后续修改可见（需关注并发）。
  - 底层 invokedynamic + LambdaMetaFactory，非匿名类生成，减少类爆炸。
  - 对性能敏感场景避免频繁分配装箱函数式对象（可缓存或改用方法引用）。
- 常见陷阱：在热循环创建大量临时 lambda；捕获大对象导致意外延长生命周期；混用复杂链可读性差。
- 面试提示：说明“行为参数化”与 invokedynamic 优势。
- 案例：将策略 if-else 替换为 Map<String, Function<Input,Output>> 精简 40 行分支代码。
- 速记：捕获=引用延寿；方法引用减少样板；避免热点装箱分配。

### 4.2 流式管道惰性
- 是什么：中间操作（map/filter/peek/sorted/distinct）构建流水线描述；终止操作（collect/reduce/forEach/anyMatch/findFirst）触发执行。
- 为什么：支持短路（anyMatch/findFirst/limit）；避免不必要计算（惰性）；可融合优化（map→map 合并）。
- 怎么用：Stream.of(...) → 中间操作链 → 终止操作 collect(toList())；使用 limit/skip 进行分页；anyMatch 提前结束。
- 关键细节：
  - 有状态中间操作：sorted/distinct/limit/skip 在并行时需额外同步或缓存。
  - 多次终止操作需重新生成流（流一旦消费失效）。
  - peek 调试仅在终止操作触发时运行，不保证打印顺序（并行）。
  - 避免深层链；必要时拆分阶段提高可读性与定位。
- 常见陷阱：重复复用已消耗流抛 IllegalStateException；在 peek 中改变状态产生副作用；distinct 大数据集内存膨胀。
- 面试提示：强调“惰性 + 短路 = 性能收益”与有状态操作成本。
- 案例：使用 anyMatch 替换全表扫描 reduce，查询耗时从 40ms 降至 7ms。
- 速记：中间搭框架，终止才执行；有状态操作需缓存。

### 4.3 并行流陷阱
- 是什么：通过 parallel() 使用公共 ForkJoinPool 将数据分区并行处理。
- 为什么：在恰当场景提升吞吐（CPU 密集 + 大规模数据 + 无共享状态）；错误使用导致线程阻塞、竞争、性能下降。
- 怎么用：list.parallelStream().map(...).reduce(...); 阻塞操作（IO、锁）请使用自定义池：ForkJoinPool custom = new ForkJoinPool(n).
- 关键细节：
  - 拆分依据 Spliterator 特征；非 SIZED/SUBSIZED 数据源拆分质量差。
  - 公共池大小 = CPU 核心数，阻塞会耗尽线程导致“假死”。
  - 有序流 (ORDERED) 保序成本高；unordered() 可提升性能（统计类操作）。
  - 避免共享可变结构（ArrayList add）→ 使用线程安全收集器或并行友好的 Collector。
- 常见陷阱：对小集合并行产生额外拆分开销；混入 synchronized/Atomic 操作抵消收益；并行中使用 Random 共享实例。
- 面试提示：给出判断公式：数据量阈值 / 纯 CPU / 无共享 / 合适拆分。
- 案例：对 1000 元素并行变慢；对 5,000,000 元素纯计算并行 p95 降 45%。
- 速记：四条件：数据大 + CPU 纯 + 无共享 + 拆分好；否则别 parallel。

### 4.4 Collector 与自定义聚合
- 是什么：封装可变汇总过程的抽象（supplier / accumulator / combiner / finisher / characteristics）。
- 为什么：扩展 reduce 能力，支持复杂分组、聚合、转换，并行安全组合。
- 怎么用：Collectors.groupingBy(Classifier, downstream)、partitioningBy(Predicate)、mapping、reducing；自定义 Collector.of(supplier, accumulator, combiner, finisher, characteristics).
- 关键细节：
  - combiner 用于并行合并部分结果；顺序流中仍可能调用。
  - characteristics 包含 CONCURRENT / UNORDERED / IDENTITY_FINISH 决定优化路径。
  - toMap 需提供 key 合并函数避免 IllegalStateException（重复键）。
  - 自定义收集器需确保线程安全或限制为顺序使用。
- 常见陷阱：忽略并行下 combiner 行为（累计器非线程安全）；错误标记 CONCURRENT 导致竞态；使用 mutable state 在 map/filter 中泄漏。
- 面试提示：描述收集器五要素与并行合并注意事项。
- 案例：自定义统计 (min,max,sum,count) 一次遍历替代多重聚合，性能提升 30%。
- 速记：五组件一特征；并行需安全 combiner；toMap 合并函数。

### 4.5 Spliterator
- 是什么：可拆分迭代器，支持 tryAdvance(单步) / trySplit(拆分) + 特征标志 (SIZED, SUBSIZED, ORDERED, SORTED, DISTINCT, CONCURRENT, IMMUTABLE, NONNULL).
- 为什么：决定并行拆分质量与负载均衡，影响并行流性能。
- 怎么用：自定义数据源实现 estimateSize、characteristics、trySplit；对不可准确计数数据估算大小。
- 关键细节：
  - trySplit 返回新 Spliterator 保证大致对半拆分，过偏导致倾斜。
  - SUBSIZED 说明后续拆分也有准确大小，有利并行。
  - ORDERED 保序需要额外成本；无序统计可调用 unordered()。
  - 不可变 / 并发标志影响内部优化与安全性。
- 常见陷阱：返回 null 过早阻止并行；错误 size 估算导致拆分策略退化；忽略特征造成不必要排序。
- 面试提示：强调“拆分均衡性”与特征选择影响并行效率。
- 案例：自定义日志块 Spliterator 优化并行解析，处理时间减半。
- 速记：拆分均衡=并行效率；特征引导优化。

### 4.6 Optional 使用规范
- 是什么：显式表示“可能缺失”值的容器，避免裸 null。
- 为什么：提高语义与安全（编译器提示链式处理）。
- 怎么用：service.findUser(id).map(User::getName).filter(...).orElse("default"); orElseThrow() 抛定制异常。
- 关键细节：
  - 不用于成员字段 / 集合元素（增加序列化、嵌套复杂）。
  - ofNullable vs of：of(null) 抛 NPE。
  - orElse 总是求值参数；orElseGet 延迟求值。
  - flatMap 用于嵌套 Optional 展开。
- 常见陷阱：API 返回 Optional<List<T>> 与 List<Optional<T>> 混淆；链条过长影响可读性；在性能热点创建大量临时 Optional。
- 面试提示：说明边界：只用于返回值，不做持久化字段。
- 案例：将多层 null 检查改链式 Optional，缺陷率下降。
- 速记：返回值用 / 字段不用 / 惰性用 orElseGet。

### 4.7 典型高频问答速记
| 问题 | 速答 |
| ---- | ---- |
| 惰性优势 | 延迟执行、短路节省计算 |
| 并行流何时用 | 数据大+CPU密集+无共享+良好拆分 |
| distinct 内存风险 | 大量元素需维护哈希结构缓存 |
| toMap 键冲突处理 | 提供合并函数 (k,v1,v2) |
| trySplit 作用 | 平衡并行任务，返回新 Spliterator |
| Optional 字段是否合适 | 不合适，增加复杂与序列化成本 |
| orElse vs orElseGet | 前者总执行，后者惰性 |

### 4.8 面试答题骨架示例（“解释并行流为何在某些场景变慢”）
1. 是什么：parallelStream 使用 ForkJoinPool 并行拆分。  
2. 原因：数据量小拆分开销 > 计算；操作阻塞占满池；共享状态竞争；有序/有状态操作增加同步。  
3. 优化：仅在大数据 CPU 密集使用；改用自定义池隔离阻塞；消除共享写入。  
4. 关键细节：公共池大小≈CPU核数；拆分质量决定效率。  
5. 案例：小列表并行慢→改顺序快 3 倍。  

### 4.9 最佳实践清单
- 使用方法引用 (Class::method) 提高可读性与减少匿名开销。
- 控制流长度：将超长流链拆分阶段性变量命名。
- 对需要多次终止操作的数据源先收集为集合再创建多个流。
- 避免在流操作中引入副作用（外部可变集合 add），改用收集器。
- 并行前：度量顺序 vs 并行性能，确认收益。
- 自定义收集器：确保 combiner 逻辑与线程安全。
- Optional：API 边界返回值使用；内部逻辑可以继续用 null 避免对象额外分配。
- 不在日志禁用级别中构建字符串（参数化日志）。

### 4.10 常见陷阱对照
| 场景 | 问题 | 规避 |
| ---- | ---- | ---- |
| 已消费流再次使用 | IllegalStateException | 重新生成流 |
| 并行流写共享集合 | 竞争/数据错乱 | 使用 concurrent 收集器 / 收集后串行操作 |
| peek 改变元素状态 | 难调试副作用 | 仅调试输出，不修改 |
| toMap 键重复 | IllegalStateException | 提供合并函数 |
| Optional.get 直接获取 | NPE | 使用 orElse / orElseThrow |
| trySplit 返回 null 过早 | 并行低效 | 实现平衡拆分逻辑 |
| parallel IO 操作 | 阻塞线程池 | 改串行或自定义池 |


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

### 7.1 String 不可变性
- 是什么：java.lang.String 为 final；内部 value（JDK 9 起为 byte[]）与 coder 均 final，不提供原地修改方法。  
- 为什么：
  - 线程安全共享：不可变对象多线程读无需同步。
  - 安全性：防止 URL、文件路径、类名等被篡改（ClassLoader、反射场景）。
  - 缓存优化：hashCode 计算后缓存（字段 hash），重复使用 O(1) 获取。
  - 结构稳定：作为 Map/Set key 时值不变，避免哈希不一致。  
- 怎么用：直接共享 String；在构造安全敏感对象时复制外部可变源（如 char[] → new String()）。  
- 关键细节：
  - new String(char[]) 会拷贝数组（防止后续数组被修改）。
  - 与 StringBuilder 不同（后者可变）。
  - equals 比较逐字符；intern 可复用同 literal 引用。  
- 常见陷阱：
  - 过度创建 new String("abc") 无意义（与字面量相同）。
  - 误将 StringBuilder 暴露为字段造成逃逸修改风险。  
- 面试提示：说明“不可变性 = 并发安全 + 作为集合键稳定 + hash 缓存提升频繁查找性能”。
- 案例：缓存层以 URL 作为 key，使用 String 避免后续被外部修改导致缓存命中率下降。

### 7.2 存储与 Compact Strings（JDK 9）
- 是什么：JDK 9 以后 value 由 char[] 改为 byte[] + coder（0=LATIN1, 1=UTF16）。  
- 为什么：大量业务字符串为 ASCII/LATIN1（英文、数字）可减半内存；提升缓存命中率与减少 GC 压力。  
- 怎么用：开发者无需改代码，JVM 自动判定编码。  
- 关键细节：
  - coder 决定解码路径；LATIN1 单字节，UTF16 双字节。
  - 影响：String.length() 在 LATIN1 下仍返回代码单元数（同原语义）。
  - substring（JDK 7+）复制新数组，避免“共享大父数组”导致内存泄漏。  
- 常见陷阱：
  - 误以为可通过手工 API 强制 LATIN1；这是内部实现细节不可控。
  - 忽略多语言内容混用导致判断为 UTF16 内存仍较大。  
- 面试提示：说明 Compact Strings 降低对堆与 GC 的压力，适用于大量英文标识场景。
- 案例：电商日志字段（orderId、userId）迁移至 JDK 11 后堆使用下降 ~8%。

### 7.3 拼接与性能
- 是什么：字符串拼接存在编译期与运行期差异：常量折叠 vs 动态构建。  
- 为什么：String 不可变，频繁 + 生成大量中间对象增加 GC。  
- 怎么用：
  - 循环内使用 StringBuilder / StringBuffer（线程安全但慢）；
  - Java 15+：使用 Text Blocks（"""）改善多行可读性；
  - 大规模构建：预估容量 new StringBuilder(expectedLength)。  
- 关键细节：
  - 编译器将 a + b + c 优化为单个 StringBuilder（非循环）。
  - 日志框架使用占位符减少无谓拼接：logger.info("id={} name={}", id, name)。
  - format() 可读性高但性能差（格式解析）。  
- 常见陷阱：
  - 在日志级别关闭时仍进行字符串拼接（缺少 isDebugEnabled 判断或使用参数化日志）。
  - 大循环内 String += 触发多次扩容与复制。  
- 面试提示：说明“循环中使用 + 导致指数级对象创建”并给具体优化对比。
- 案例：某批处理将 1e6 行 CSV 构造改为 StringBuilder 预分配，GC 次数从 12 次降到 2 次。

### 7.4 字符串常量池与 intern
- 是什么：JVM 维护字符串常量池（字面量与第一次调用 intern() 的结果）。  
- 为什么：避免重复常量对象；提升比较通过引用（==）判定的可能性（慎用）。  
- 怎么用：
  - 仅对少量高度重复值（枚举名称、状态码）调用 intern。
  - 避免对大量动态内容（如用户输入）intern。  
- 关键细节：
  - JDK 7 起常量池转入堆，不再受 PermGen 限制但仍可能导致堆膨胀。
  - intern 返回池内引用，若未存在则添加。
  - 不推荐用 == 比较业务字符串。  
- 常见陷阱：
  - 滥用 intern 导致内存不可回收热点膨胀。
  - 在缓存场景通过 intern 期望获得 Map 减少 equals 成本收益微弱。  
- 面试提示：说明 intern 是空间换引用复用的特定场景工具而非通用性能优化。
- 案例：将支付状态码（<50 种）intern 后节省重复对象但未显著影响整体性能（展示理性评估）。

### 7.5 编码与 Charset 管理
- 是什么：字符→字节转换需显式 charset；默认平台编码不可靠。  
- 为什么：
  - 避免跨平台（Windows GBK vs Linux UTF-8）出现乱码。
  - 网络协议、消息传输需统一编码保障一致性。  
- 怎么用：
  - 始终使用 StandardCharsets.UTF_8；
  - new String(bytes, StandardCharsets.UTF_8)；
  - Files.newBufferedReader(path, UTF_8)。  
- 关键细节：
  - 不要省略 charset 参数（默认编码可变）；
  - BOM（UTF-8 BOM）可能影响某些解析器；
  - InputStreamReader 默认平台编码。  
- 常见陷阱：
  - JSON/CSV 读写未指定编码，线上与测试环境不一致；
  - 使用 String.getBytes() 无参导致潜在不可预测字节序列。  
- 面试提示：举一个“测试环境 UTF-8 → 生产 GBK 导致消息解析失败”的案例。
- 案例：接口签名校验使用 bytes 不同编码导致签名不一致，修复方式：强制 UTF-8。

### 7.6 Unicode、代码点与代理对 (Surrogate Pair)
- 是什么：Java 内部使用 UTF-16 代码单元；部分字符（emoji、罕见汉字）占两个 char（代理对）。  
- 为什么：处理长度、截取、展示时需基于代码点避免截断非法。  
- 怎么用：
  - str.codePointCount(0, str.length());
  - 遍历：str.offsetByCodePoints(index, 1)；
  - 正确截取：使用 BreakIterator 或基于 codePoint API。  
- 关键细节：
  - charAt 返回单个代码单元，可能不是完整字符；
  - emoji 长度显示与代码点数量不同；
  - 盲目 substring 可能截断代理对产生乱码。  
- 常见陷阱：
  - length() 作为“字符数”用于限制昵称长度不准确；
  - UI 截断导致显示“残缺方框”。  
- 面试提示：说明对“字符”与“代码点”区分与业务长度校验正确方式。
- 案例：昵称限制 10 “字符”实际使用 length 导致允许 10 个代理对（20 code units）→ 风险：过长展示溢出。

### 7.7 Locale、大小写与正规化
- 是什么：不同语言存在大小写与排序差异（例：土耳其语 I ↔ ı）。  
- 为什么：错误 Locale 导致认证、搜索、比较失败。  
- 怎么用：
  - toLowerCase(Locale.ROOT) 用于稳定逻辑（标识规范化）；
  - Collator 排序；Normalizer 进行 Unicode 规范化（NFC/NFD）。  
- 关键细节：
  - 默认 Locale 随系统；不适合用于协议/标识转换；
  - 正规化避免组合字符差异（é vs e + ´）。  
- 常见陷阱：
  - 使用默认 toLowerCase 导致跨区行为差异；
  - 未正规化造成缓存键不一致。  
- 面试提示：举例“土耳其语用户登录失败”场景说明 Locale 重要性。
- 案例：国际站对产品 SKU 进行 lowerCase 未指定 Locale 导致少数地区异常。

### 7.8 安全与边界处理
- 是什么：字符串输入需防止注入与资源耗尽（超长、异常编码）。  
- 为什么：避免 SQL/LDAP/命令注入；防止超长 payload 造成内存压力。  
- 怎么用：
  - 长度限制 + 白名单字符（正则）；
  - 参数化语句避免拼接；
  - 验证编码后再入库。  
- 关键细节：
  - 正则需预编译避免重复编译；
  - 过滤后保持失败原因透明（返回统一错误码）。  
- 常见陷阱：
  - 直接拼接 SQL："... where name = '" + userInput + "'"；
  - 未限制上传文件名长度造成日志污染。  
- 面试提示：描述“统一输入校验 → 安全基线”的策略。
- 案例：过滤日志中控制字符防止终端逃逸攻击。

### 7.9 性能度量与工具
- 是什么：针对字符串操作热点使用基准验证真实开销。  
- 为什么：避免凭感觉优化或过度设计。  
- 怎么用：
  - JMH：基线对比 + vs StringBuilder；
  - Java Flight Recorder：识别高频分配；
  - Allocation profiling：async-profiler 观察分配火焰图。  
- 关键细节：
  - 预热迭代确保 JIT 优化生效；
  - 分离不同长度、不同字符集场景。  
- 常见陷阱：
  - 忽略逃逸分析使得优化判断失真；
  - 在未隔离 GC 噪声条件下测量。  
- 面试提示：强调“以基准数据证明优化有效”，展示量化能力。
- 案例：JMH 测试 1e5 次拼接：String += 耗时 320ms → StringBuilder 90ms。

### 7.10 高频问答速记
| 问题 | 速答 |
| ---- | ---- |
| String 为什么不可变 | 并发安全 + 安全性 + hash 缓存 + 作为 Map key 稳定 |
| JDK9 改成 byte[] 意义 | 降低 ASCII / LATIN1 场景内存占用 |
| intern 适用场景 | 少量高频重复字面值；避免动态海量值 |
| length() 与真实字符数差异 | 遇代理对（emoji）length() > codePointCount |
| 编码必须显式指定原因 | 默认编码不可靠导致跨平台乱码与签名失败 |
| 循环中 + 低效原因 | 产生多余中间对象与扩容复制 |
| Locale.ROOT 用途 | 稳定大小写规范化避免地区差异 |

### 7.11 面试答题骨架示例（“如何优化大量字符串拼接”）
1. 是什么：循环内频繁 String += 构建大文本。  
2. 问题：不可变导致多次分配与复制，GC 压力增大。  
3. 优化：使用单个 StringBuilder 预估容量；日志参数化；必要时使用 char[] 聚合。  
4. 数据：基准对比前后耗时与分配次数。  
5. 风险：过度 micro-optimization；维护复杂度提升。  
6. 收益：延迟下降 X%、GC 次数减少 Y%。  

---

## 8. 泛型与类型擦除 [5]

### 8.1 擦除机制
- 是什么：编译期做类型检查与插入强制转换；运行时参数化类型被擦除成其上界（默认 Object）。  
- 为什么：与旧 JVM 字节码兼容，不需修改运行时。  
- 怎么用：通过 Class<T> 或 Type/ParameterizedType 获取结构；无法直接获得泛型实参运行时类（List<String> 只见到 List）。  
- 关键细节：
  - instanceof 不能含泛型实参：if (list instanceof List<String>) 非法。
  - 运行时 List<String> 与 List<Integer> 的 getClass() 相同。  
  - 桥接方法保证多态（编译器合成）。  
- 常见陷阱：试图反射获取 T 的真实类型；滥用强制转换隐藏类型错误。  
- 面试提示：一句话示例“擦除后容器只保留原始结构，元素类型安全靠编译期”。  
- 案例：某工具库内部反射期望 List<String> 元信息失败，改用 TypeToken 解决。

### 8.2 通配符与 PECS 原则
- 是什么：? extends T（生产者只读） / ? super T（消费者写入）。  
- 为什么：平衡泛型协变/逆变不可用的限制。  
- 怎么用：只读数据源 List<? extends Number>；写入聚合 List<? super Number>。  
- 关键细节：? super 读出退化为 Object；? extends 不允许 add（除 null）。  
- 常见陷阱：误在 ? extends 集合中 add 导致编译错误；? super 读取后错误强转。  
- 面试提示：快速写出：copy(List<? extends T> src, List<? super T> dst)。  
- 案例：实现通用 copy 方法减少重复。

### 8.3 上界与多重约束
- 是什么：<T extends A & B & C> 要求 T 同时具有多个接口能力（类必须放首位且最多一个）。  
- 为什么：算法对能力集合有要求（排序+克隆+序列化）。  
- 怎么用：public <T extends Comparable<T> & Serializable> void sortAndStore(List<T> list)。  
- 关键细节：多重界定后访问受限于交集；避免过度复杂导致理解困难。  
- 常见陷阱：界定链过长暴露实现细节。  
- 面试提示：指出“类在最前+多个接口”。  
- 案例：限制缓存键必须既可序列化又可自然排序。

### 8.4 数组协变 vs 泛型不协变
- 是什么：数组协变（Sub[] → Super[]）；泛型不协变需通配符。  
- 为什么：数组保留运行时类型信息能抛出 ArrayStoreException；泛型擦除无法安全协变。  
- 怎么用：避免创建参数化类型数组：List<String>[] → 使用 List<?>[] 或 List<List<String>>。  
- 关键细节：new T[] 不允许；@SuppressWarnings("unchecked") 仅在无法避免时局部使用。  
- 常见陷阱：使用参数化数组导致堆污染。  
- 面试提示：给出“Object[] arr= new String[1]; arr[0]=Integer”抛异常示例。  
- 案例：重构框架内部 T[] 缓冲改用 List<T> 减少警告。

### 8.5 桥接方法 (Bridge Method)
- 是什么：编译器生成合成方法维持泛型与擦除后签名一致。  
- 为什么：保证多态：子类重写返回更具体类型仍能被父类引用正确调用。  
- 怎么用：理解反射时 isSynthetic() 过滤。  
- 关键细节：影响 AOP/反射扫描（需跳过）。  
- 常见陷阱：统计方法数出现重复。  
- 面试提示：举例 Comparator<String>#compare 与原始类型兼容。  
- 案例：反射框架过滤 synthetic 方法避免重复注册。

### 8.6 设计原则
- 是什么：保持泛型 API 简洁、类型安全、意图明确。  
- 为什么：过度嵌套降低可读与可维护性。  
- 怎么用：拆分复杂 Map<String, List<Map<K,V>>> → 定义值对象；对外暴露窄泛型。  
- 关键细节：倾向具体类型返回；通配符用于入参增加适配性。  
- 常见陷阱：统一使用通配符导致调用方推断困难。  
- 面试提示：说明“返回具体，接收泛化”策略。  
- 案例：简化分页 API：Page<T> 而非 Map<String,Object>。

### 8.7 TypeToken 与反射泛型信息
- 是什么：通过匿名子类捕获泛型参数（Guava TypeToken / 自定义）。  
- 为什么：运行时获取 List<String> 中 String 类型信息用于序列化、转换。  
- 怎么用：new TypeToken<List<String>>(){} 获取 ParameterizedType。  
- 关键细节：只能在创建时立即捕获；需缓存避免重复解析。  
- 常见陷阱：在非匿名子类或擦除路径中失去信息。  
- 面试提示：说明“绕过擦除以获取实际类型参数”。  
- 案例：JSON 反序列化泛型集合使用 TypeToken 精确还原元素类型。

### 8.8 泛型类型推断与菱形语法
- 是什么：<> 让编译器推断构造时实参类型（List<String> list = new ArrayList<>();）。  
- 为什么：减少冗余。  
- 怎么用：在复杂嵌套结构中使用菱形语法提升可读。  
- 关键细节：类型推断基于赋值目标；过度推断可能退化为 Object。  
- 常见陷阱：返回值推断不佳导致原始类型警告。  
- 面试提示：指出“菱形语法仅在 Java 7+”。  
- 案例：Map<String,List<Integer>> m = new HashMap<>(); 清晰。

### 8.9 通配符捕获 (Capture Conversion)
- 是什么：编译器在内部为 ? 创建隐式类型变量以支持安全操作。  
- 为什么：允许在方法内部转换 List<?> → List<T> 受控使用。  
- 怎么用：私有辅助方法 <T> void process(List<T> list) 接受 List<?> 调用。  
- 关键细节：调用者仍保持 ? 协议安全。  
- 常见陷阱：试图直接强转 (List<String>) list<?>。  
- 面试提示：说明捕获转换帮助局部泛化。  
- 案例：validate(List<?> l) 内部使用辅助泛型方法消除警告。

### 8.10 堆污染与 @SafeVarargs
- 是什么：泛型类型不一致引用写入不安全数据。  
- 为什么：导致后续读取 ClassCastException。  
- 怎么用：避免参数化数组；使用不可变集合；在 final / static / private 可变参数泛型方法上加 @SafeVarargs。  
- 关键细节：@SafeVarargs 仅声明“方法内部不执行不安全操作”。  
- 常见陷阱：错误标注 @SafeVarargs 掩盖真实风险。  
- 面试提示：解释“为什么泛型 varargs 会有警告”。  
- 案例：combine(List<String>... lists) 误加入 List<Integer> 产生隐藏问题。

### 8.11 原始类型 (Raw Type) 风险
- 是什么：未参数化使用泛型类，如 List raw = new ArrayList();。  
- 为什么：绕过类型检查引入运行时失败。  
- 怎么用：始终加参数；迁移旧代码逐步泛型化。  
- 关键细节：原始类型与擦除后不同（编译器施加 unchecked 警告）。  
- 常见陷阱：公共 API 暴露 Raw Type 导致传播。  
- 面试提示：说明“Raw Type = 放弃泛型安全”。  
- 案例：旧工具方法返回 raw Map 在调用层出现强转异常。

### 8.12 泛型与性能 / JIT
- 是什么：泛型仅编译期存在，不增加运行时对象；类型转换仍需在边界执行。  
- 为什么：性能影响主要在额外的强制转换与无法特化原始类型（不存在 List<int>)。  
- 怎么用：对性能敏感的数值集合可使用 Trove/FastUtil（特化）或原始数组。  
- 关键细节：装箱/拆箱成本高于泛型本身；JIT 可消除部分边界转换。  
- 常见陷阱：将性能问题归因于“使用泛型”而非装箱。  
- 面试提示：强调“泛型本身开销可忽略，关注装箱与分配”。  
- 案例：改 List<Long> 计数为 LongAdder 提升吞吐。

### 8.13 典型问答速记
| 问题 | 速答 |
| ---- | ---- |
| 为何需要擦除 | 与旧 JVM 兼容，减少运行时复杂性 |
| 如何获取泛型实参 | 使用 Type/ParameterizedType 或 TypeToken 捕获 |
| PECS 原则含义 | 生产者 extends 只读，消费者 super 可写 |
| 泛型数组为何危险 | 运行时类型信息丢失导致堆污染风险 |
| 桥接方法作用 | 保持泛型重写后多态一致性 |
| @SafeVarargs 使用条件 | final/static/private 泛型可变参且内部安全 |
| Raw Type 风险 | 跳过编译期检查导致运行时 ClassCastException |
| 不能写入 ? extends 集合原因 | 类型不确定，写入破坏类型安全 |
| 泛型性能问题本质 | 装箱与转换，而非泛型语法本身 |

### 8.14 答题骨架示例（“解释 PECS 原则并举例”）
1. 是什么：协变与逆变在 Java 泛型不可直接表达，通过通配符分离读/写。  
2. 为什么：保证类型安全又提升灵活性。  
3. 示例：copy(List<? extends T> src, List<? super T> dst)。  
4. 关键：? extends 禁止写入（无法证明安全），? super 读取退化 Object。  
5. 陷阱：在 ? extends 集合 add 导致编译错误；误强转 ? super 读取类型。  
6. 案例：通用集合复制与归并 API。

### 8.15 最佳实践清单
- 返回值尽量使用具体类型（List<User>），入参可用通配符增加兼容（Collection<? extends User>）。  
- 避免 Raw Type；出现 unchecked 警告先审查再有选择 @SuppressWarnings。  
- 避免参数化类型数组；若必须使用，限制作用域并说明理由。  
- 泛型工具方法加入合理上界（<T extends Comparable<? super T>> 排序）。  
- 缓存反射得到的 Type 元信息（避免重复解析）。  
- 利用 TypeToken 捕获复杂嵌套类型用于序列化/转换。  
- 可变参数泛型方法审查堆污染后标注 @SafeVarargs。  
- 不滥用通配符造成阅读困难；保持 API 简明单一职责。  
- 文档中明确泛型参数含义（T = 元素类型、K/V = 键值）。  


---

## 9. 反射与注解机制 [4]

### 9.1 ClassLoader 体系与双亲委派
- 是什么：
  - 类加载过程（加载→验证→准备→解析→初始化）；
  - 层次：Bootstrap(核心) → Platform(旧 Extension) → Application(System) → 自定义（URLClassLoader、模块 Layer）；
  - 双亲委派：优先向上委派避免核心类被篡改。
- 为什么：
  - 隔离不同模块版本；
  - 支持插件与热替换；
  - 安全（防止伪造 java.lang.*）。
- 怎么用：
  - 自定义：继承 URLClassLoader 或使用 ModuleLayer 定义模块隔离；
  - SPI：ServiceLoader 依赖 Thread Context ClassLoader 查找 META-INF/services；
  - 热加载：专用 Loader + 丢弃引用以便 GC。
- 关键细节：
  - 同名类由不同 ClassLoader 加载 ⇒ 类型不兼容 (ClassCastException)；
  - 内存泄漏：Web/App 容器中静态缓存持有自定义 Loader 导致卸载失败；
  - ContextClassLoader：框架（JDBC、JAXP、ServiceLoader）在回调线程中查找应用类；
  - getResource 与 getResourceAsStream 依加载器搜索范围；
  - 模块系统强封装，需 --add-opens 打开包给反射。
- 常见陷阱：
  - 直接使用 Class.forName 而非线程上下文导致 SPI 失败；
  - 忘记关闭 URLClassLoader 持有 Jar 文件句柄；
  - Mixed Delegation 破坏安全（未调用父加载器）。
- 面试提示：能说明“为什么需要上下文 ClassLoader + 一个真实的泄漏案例（线程本地缓存持 Loader）”。

### 9.2 反射使用与性能优化
- 是什么：运行时检查/访问类型结构（Class、Field、Method、Constructor）；setAccessible 打破封装。
- 为什么：框架通用化（DI、序列化、ORM、AOP）；动态适配未知类型。
- 怎么用：
  - 启动阶段一次性扫描 + 缓存元数据（Method、Field、PropertyDescriptor）；
  - 访问字段优先 MethodHandle / VarHandle；
  - 批量构造对象：预热构造器句柄避免重复查找。
- 关键细节：
  - setAccessible 在 Java 17+ 强封装模块里需 --add-opens；
  - 频繁反射阻碍 JIT 内联（invokeExact 可优化）；
  - 安全：反射可绕过 private，需最小化暴露；
  - VarHandle 提供更轻量字段访问 + 原子/内存语义操作；
  - AccessibleObject.canAccess 代替旧 isAccessible 检查。
- 常见陷阱：
  - 每次调用通过 Class.getDeclaredXxx 查找未缓存；
  - 大量字段遍历导致启动慢；
  - 反射异常被吞（InvocationTargetException cause 丢失）。
- 面试提示：描述“缓存反射元数据让启动从 2.3s 降到 1.1s”的案例 & 替换为 MethodHandle 的收益。

### 9.3 MethodHandle / VarHandle / invokedynamic
- 是什么：
  - MethodHandle：可优化的可组合调用句柄；
  - VarHandle：统一变量访问抽象（普通字段、数组元素、ByteBuffer）含原子/内存屏障；
  - invokedynamic：JVM 指令延迟绑定（Lambda、字符串拼接、动态语言）。
- 为什么：降低反射成本，提供更接近 JVM 层的优化入口。
- 怎么用：
  - Lookup lookup = MethodHandles.lookup();
  - findVirtual/findStatic/findConstructor；
  - LambdaMetaFactory 结合 invokedynamic 生成函数式对象；
  - VarHandle 获取并执行 getAndSet/compareAndExchange。
- 关键细节：
  - MethodHandle 类型签名严格（invokeExact vs invoke）；
  - 组合器：filterArguments、filterReturnValue、dropArguments；
  - VarHandle 支持 acquire/release/opaque/volatile 访问模式；
  - Lambda 表达式字节码不生成匿名类，延迟引导提升性能。
- 常见陷阱：
  - 忘记缓存 MethodHandle 重复 lookup；
  - 使用 invoke 而非 invokeExact 失去编译期签名校验；
  - 误解 VarHandle 与 Unsafe：前者更安全。
- 面试提示：说明“为何框架迁移到 MethodHandle + JIT 内联优势”。

### 9.4 注解模型与元注解
- 是什么：注解 = 元数据；元注解：@Retention、@Target、@Documented、@Inherited、@Repeatable。
- 为什么：增强可读性与工具处理能力，自描述配置。
- 怎么用：
  - SOURCE：代码生成、静态检查；
  - CLASS：保留到字节码，运行时不读取；
  - RUNTIME：反射读取框架处理；
  - @Repeatable 通过容器注解实现多值。
- 关键细节：
  - @Inherited 仅对类继承有效，对接口/方法无效；
  - 注解元素默认可以有 default 值，类型受限（基本类型、String、Enum、Class、注解、数组）；
  - 运行时获取：method.getAnnotations() / getDeclaredAnnotations()；
  - 合成注解（Spring）通过别名与组合整合配置。
- 常见陷阱：
  - 忘记 Retention 导致运行时读取不到；
  - 滥用 RUNTIME 增加扫描开销；
  - default 值语义不清晰；缺少文档说明。
- 面试提示：能列举自定义注解：@Target(ElementType.TYPE) @Retention(RUNTIME) @Documented，并解释选择 RUNTIME 的原因。

### 9.5 编译期注解处理与 APT
- 是什么：注解处理器 (JSR 269) 在编译阶段生成代码（MapStruct、AutoService、Lombok(借助内部 hook)）。
- 为什么：减少运行时反射与启动时间；编译期失败更早反馈。
- 怎么用：
  - 实现 AbstractProcessor；
  - 支持 @SupportedAnnotationTypes 与 @SupportedSourceVersion；
  - 利用 Filer 生成源文件或资源。
- 关键细节：
  - 增量编译需正确声明支持；
  - 多轮处理 (rounds) 直到无新文件产生；
  - 需处理元素模型（TypeElement、ExecutableElement）。
- 常见陷阱：
  - 生成代码包名/路径错误；
  - 忽略异常导致处理器静默失败；
  - 未处理重复注解导致冲突。
- 面试提示：比较“运行时扫描 vs 编译期生成”性能与失败定位差异。

### 9.6 注解扫描与启动优化
- 是什么：框架通过类路径或包扫描收集注解（Spring、Jakarta）。
- 为什么：自动配置、依赖注入、AOP 织入。
- 怎么用：
  - 限定基础包；
  - 预构建索引（Spring AOT、Quarkus Jandex）；
  - 延迟初始化非关键 Bean。
- 关键细节：
  - I/O 与反射叠加启动成本；
  - Native Image 需配置反射元数据；
  - JAR 内部扫描需处理多 ClassLoader 场景。
- 常见陷阱：
  - 无包限制全盘扫描；
  - 过度使用 ClassPath 遍历导致冷启动慢；
  - Multi-release JAR 兼容性忽略。
- 面试提示：给出“扫描包从 30 个减到 5 个启动缩短 40%”的量化。

### 9.7 动态代理技术栈对比
- 是什么：JDK Proxy（接口） vs CGLIB（继承） vs ByteBuddy（字节码 DSL） vs Javassist（源级/字节码混合）。
- 为什么：实现横切关注点（事务、缓存、日志、限流、监控）。
- 怎么用：
  - JDK：Proxy.newProxyInstance + InvocationHandler；
  - CGLIB：Enhancer.setSuperclass + MethodInterceptor；
  - ByteBuddy：new ByteBuddy().subclass(...).method(ElementMatchers...).intercept(...).make()。
- 关键细节：
  - final 类/方法无法 CGLIB 代理；
  - equals/hashCode/toString 建议在代理层处理；
  - 自调用绕过代理 ⇒ 事务/缓存失效；
  - 代理链顺序影响执行（metrics → retry → circuit breaker）。
- 常见陷阱：
  - 代理对象泄漏（缓存原始与代理混用）；
  - 过度嵌套代理导致调用栈深、性能下降；
  - InvocationHandler 中捕获异常后吞掉导致业务静默失败。
- 面试提示：说明“如何解决 @Transactional 自调用：拆分服务或注入自身代理”。

### 9.8 反射与安全 / 模块封装
- 是什么：Java 9+ 模块系统限制未开放包的深反射访问。
- 为什么：提高封装与安全边界。
- 怎么用：
  - 在启动参数添加 --add-opens module/package=ALL-UNNAMED；
  - 尽量使用公共 API 或 MethodHandle。
- 关键细节：
  - IllegalAccessException 在强封装下更常见；
  - 最小开放原则：仅对需要的包开放；
  - 禁止滥用 --illegal-access（已逐步移除）。
- 常见陷阱：
  - 升级 JDK 后反射失效，未调整模块开放；
  - 使用 Unsafe 绕过封装造成兼容风险。
- 面试提示：说明“迁移到 Java 17 遇到的反射失败处理步骤”。

### 9.9 反射内存与 GC 影响
- 是什么：反射缓存元数据与生成的代理 / 额外类增加元数据与元空间使用。
- 为什么：大量动态生成类（CGLIB、ByteBuddy）可能导致 Metaspace 增长。
- 怎么用：
  - 合理复用代理类；
  - 限制动态类生成数量；
  - 监控 Metaspace（JFR / jcmd VM.native_memory）。
- 关键细节：
  - 旧 PermGen 已被 Metaspace 替代；
  - ClassLoader 引用链导致整批类无法卸载。
- 常见陷阱：
  - 热加载循环生成新类未清理；
  - 将代理类放入静态集合永久持有。
- 面试提示：提供一次“Metaspace OOM”排查过程。

### 9.10 典型面试问答速记
| 问题 | 要点速答 |
| ---- | -------- |
| 双亲委派意义 | 防止核心类被篡改，保证类型一致性与安全 |
| 为什么使用 MethodHandle | 更接近 JVM，JIT 可内联，低于反射开销 |
| @Inherited 作用范围 | 仅类继承层次，不影响方法/字段/接口 |
| 运行时扫描优化手段 | 限定包 + 预索引 + 缓存元数据 + 延迟加载 |
| 事务自调用失效原因 | 直接方法调用绕过代理，未触发切面逻辑 |
| VarHandle 优势 | 标准化内存语义访问，替代 Unsafe 某些场景 |
| 注解编译期处理价值 | 更早失败、减少运行时反射、生成高效代码 |

### 9.11 答题骨架示例（“为什么选择 MethodHandle 而非反射”）
1. 是什么：JVM 层直接可组合调用句柄。  
2. 为什么：更低调用开销、JIT 优化空间更大。  
3. 怎么用：启动阶段 lookup + 缓存句柄；invokeExact 保持签名。  
4. 关键细节：签名严格、可组合过滤器、对性能敏感路径适用。  
5. 风险与陷阱：未缓存导致重复 lookup；滥用 invoke 丢编译期检查。  
6. 案例：反射调用 QPS 下降 → 改 MethodHandle 提升 15%。  

### 9.12 实战最佳实践清单
- 扫描：限定基础包，避免全类路径遍历。  
- 缓存：字段/方法元数据缓存为轻量结构（name→MethodHandle）。  
- 降级：能用直接调用/接口聚合时不用反射。  
- 可测试性：封装反射访问到单独 Utility，便于 Mock 与替换。  
- 模块化：升级 JDK 时审查 --add-opens 配置，逐步依赖公开 API。  
- 动态类：合并拦截逻辑减少多层代理（使用组合切面顺序）。  
- 安全：仅开放必要包；禁止在公共库中暴露 setAccessible 滥用。  

### 9.13 常见陷阱对照
| 场景 | 问题 | 规避策略 |
| ---- | ---- | -------- |
| 热加载 | ClassLoader 泄漏 | 避免静态持 Loader；使用弱引用缓存 |
| 大量反射 | 启动慢 | 预索引 + 缓存 + 编译期生成 |
| 代理嵌套 | 调用栈深 | 合并横切逻辑、顺序链管理 |
| setAccessible 滥用 | 封装破坏/升级失效 | 改 MethodHandle / 公共 API |
| 未处理 InvocationTargetException | cause 丢失 | 捕获 unwrap 记录 |
| 事务自调用 | 失效 | 结构拆分 / 注入自身代理 |

### 9.14 推荐深度学习路径
- 源码：ClassLoader、ServiceLoader、MethodHandles、LambdaMetaFactory、Annotation Processing、ByteBuddy。  
- 工具：javap -c 查看 invokedynamic、JFR 事件分析反射热点、jdeps 分析模块依赖。  
- 实验：基准比较反射 vs MethodHandle vs 直接调用；扫描范围减少对启动影响测试。 


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

## 11. 语言新特性[3]

### 11.1. Record（Java 16+）
- 是什么：不可变数据载体语法糖，自动生成构造器、equals/hashCode/toString、组件访问器。
- 为什么：减少样板，鼓励值对象语义，提升可读性。
- 怎么用：record User(Long id, String name) {}；在 compact constructor 内加入校验。
- 关键细节：字段隐式 final；可定义静态成员与实例方法；不能继承其他类（隐式 extends Object）；可实现接口。
- 常见陷阱：在 equals/hashCode 中再手写逻辑重复；将可变集合作为组件未做 defensive copy。
- 面试提示：强调“领域值对象 + 不变性”。
- 案例：DTO 类 120 行 → Record 12 行。
- 速记：Record = 精简不可变数据容器。

### 11.2. Sealed Classes（Java 17）
- 是什么：通过 permits 限定允许的子类集合。
- 为什么：穷举模式匹配更安全；限制扩展面。
- 怎么用：sealed interface Shape permits Circle, Rectangle {}
- 关键细节：子类必须是 final / sealed / non-sealed；模块/包可见性影响封闭性；与模式匹配 switch 协同简化分支。
- 常见陷阱：忘记给子类声明 final/sealed/non-sealed 导致编译出错；跨包扩展受限。
- 面试提示：说明“可控继承层次 + 模式匹配优化”。
- 案例：支付状态枚举扩展迁移到 sealed 层次，避免被外部添加非法子类。
- 速记：Sealed = 受控扩展集合。

### 11.3. Pattern Matching for instanceof / switch（Java 16+ / 21）
- 是什么：instanceof 自动绑定变量；switch 支持类型/模式匹配（含守护条件）。
- 为什么：减少显式强转与冗长分支；提高可读性。
- 怎么用：if (obj instanceof User u) { u.name(); }；switch (shape) { case Circle c -> ... ; case Rectangle r -> ... }
- 关键细节：匹配顺序自上而下；封闭层次（sealed）可启用穷举检查；guard (when) 提升表达力。
- 常见陷阱：重复模式导致不可达；在 switch 中遗漏 default 导致编译警告（非穷举）。
- 面试提示：突出“减少样板 + 增强穷举保障”。
- 案例：旧 instanceof 链 30 行 → 模式 switch 12 行。
- 速记：Pattern Matching = 自动绑定 + 穷举安全。

### 11.4. Switch Expressions（Java 14）
- 是什么：switch 作为表达式返回值；箭头语法减少 fall-through。
- 为什么：消除样板 break；更函数化。
- 怎么用：int score = switch(level) { case HIGH -> 100; case LOW -> 10; default -> 50; };
- 关键细节：多个标签逗号分隔；yield 用于代码块内返回值；表达式分支不可隐式贯穿。
- 常见陷阱：误认为箭头允许隐式共享状态；忽略 default。
- 面试提示：说明“更安全不再意外贯穿”。
- 案例：枚举映射优化。
- 速记：新 switch = 值表达 + 安全标签。

### 11.5. Text Blocks（Java 15）
- 是什么：""" 多行字符串，提高可读性（SQL/JSON）。
- 为什么：减少转义与拼接。
- 怎么用：String sql = """SELECT * FROM user WHERE id = ?""";
- 关键细节：自动去除公共缩进；末尾换行处理规则；仍需参数化防注入。
- 常见陷阱：缩进误差导致不期望空格；直接拼接用户输入。
- 面试提示：强调“维护成本降 + 安全仍需校验”。
- 案例：SQL 片段行数减少与错误率下降。
- 速记：""" 优化多行字面量。

### 11.6. Virtual Threads（Java 21 LTS / Project Loom）
- 是什么：用户模式轻量线程，调度由 JVM 管理，阻塞调用不占 OS 线程长期资源。
- 为什么：降低并发编程复杂度（保留同步风格）同时扩展到大量并发（百万级）。
- 怎么用：Thread.startVirtualThread(() -> handler()); Executors.newVirtualThreadPerTaskExecutor()
- 关键细节：阻塞 I/O 自动挂起；不适合长期占用 CPU 紧循环；ThreadLocal 行为需关注（频繁创建）；栈深回收与调度策略仍需度量。
- 常见陷阱：期望 CPU 密集加速（瓶颈仍在核心数）；忽略同步结构竞争；过度依赖 ThreadLocal 增成本。
- 面试提示：回答“区别于传统线程：调度+栈在堆分片可切换”。
- 案例：从 500 线程池迁移虚拟线程 → 并发连接提升 20x，代码保持阻塞风格。
- 速记：虚拟线程 = 轻阻塞并发。

### 11.7. Structured Concurrency（Java 21 Incubator）
- 是什么：对并发任务形成结构化生命周期（类似作用域），统一取消与聚合异常。
- 为什么：避免“分叉后遗忘”线程/任务；资源与错误集中管理。
- 怎么用：try (var scope = new StructuredTaskScope.ShutdownOnFailure()) { scope.fork(...); scope.join(); scope.throwIfFailed(); }
- 关键细节：不同策略 Scope（ShutdownOnFailure / ShutdownOnSuccess）；join 等待所有；聚合异常简化处理。
- 常见陷阱：混用自定义线程池与 scope 导致取消不一致。
- 面试提示：说明“结构化=可见边界 + 统一错误处理”。
- 案例：并发远程聚合改造减少孤儿任务。
- 速记：Structured = 任务生命周期显式。

### 11.8. Scoped Values（Java 21 Preview）
- 是什么：线程栈上下文替代部分 ThreadLocal 场景，更轻量可安全传递。
- 为什么：减少不可控 ThreadLocal 泄漏与复杂清理。
- 怎么用：ScopedValue.runWhere(SOME_KEY, value, () -> logic());
- 关键细节：只读语义；嵌套覆盖；虚拟线程适配良好。
- 常见陷阱：尝试存可变共享对象并修改；当成写缓存。
- 面试提示：对比 ThreadLocal：无显式 remove，生命周期作用域清晰。
- 案例：请求上下文迁移减少清理代码。
- 速记：Scoped = 受控上下文只读。

### 11.9. Foreign Function & Memory API（Java 22+ / Panama）
- 是什么：安全访问本地内存与调用 native 函数替代 JNI（MemorySegment, Linker）。
- 为什么：降低 JNI 样板与开销；高性能与安全边界。
- 怎么用：MemorySegment.allocateNative(...); Linker.link(…).
- 关键细节：生命周期/作用域控制（Arena）；内存布局（ValueLayout）；安全检查可启用/禁用。
- 常见陷阱：错误作用域释放后仍访问；未对齐布局。
- 面试提示：强调“性能 + 安全比 JNI 更可维护”。
- 案例：解析二进制协议使用本地内存减少复制。
- 速记：FFM = 现代化安全替代 JNI。


### 11.10. LTS 演进对比速查
| 版本 | 主要语言/语法特性 | 价值 |
| ---- | ---------------- | ---- |
| 8 | Lambda, Stream, Optional | 行为参数化 |
| 11 | var 局部推断(10+), 新 HttpClient | 减样板 / 标准化客户端 |
| 17 | Sealed, Pattern Matching instanceof, Records, Text Blocks | 不变值对象 + 模式增强 |
| 21 | Virtual Threads, Structured Concurrency, Pattern Matching Switch | 轻量并发 + 安全穷举 |

### 11.11 面试高频速答
| 问题 | 速答 |
| ---- | ---- |
| Record vs Lombok | 原生语义 + 自动 equals/hashCode，无需注解处理器 |
| Sealed 好处 | 限制继承集合，模式匹配可穷举 |
| 虚拟线程优势 | 阻塞风格保留，极大提升并发数量 |
| Structured Concurrency 解决什么 | 避免孤儿任务，集中错误与取消 |
| Scoped Value 对比 ThreadLocal | 受限作用域只读，无手动清理 |
| Pattern Matching switch 何时安全省略 default | 在 sealed 完全穷举子类时 |
| Foreign Memory API 替代 JNI 好处 | 更少样板 + 安全检查 + 性能 |

### 11.12. 答题骨架示例（“虚拟线程与线程池差异”）
1. 是什么：JVM 管理调度的超轻量线程（百万级）。  
2. 为什么：阻塞 I/O 不再占用昂贵平台线程，简化异步回调。  
3. 怎么用：newVirtualThreadPerTaskExecutor / Thread.startVirtualThread。  
4. 关键细节：ThreadLocal 使用成本；CPU 密集仍受核心限制；阻塞点可安全挂起。  
5. 陷阱：误将其用于忙循环；滥用 ThreadLocal。  
6. 案例：连接数扩展 20x。  

### 11.13. 最佳实践清单
- 使用 Record 表达纯数据，不加入复杂可变逻辑。
- Sealed + switch 模式组合实现穷举业务状态。
- 并发场景优先评估虚拟线程替代复杂回调。
- Scoped Value 替代易泄漏 ThreadLocal 场景。
- Pattern Matching 提升类型分发清晰度，避免 instanceof 链。
- 充分验证预览/孵化特性在生产启用前的稳定性与兼容性。
- Text Blocks 用于多行常量，仍保持参数化防注入。
- FFM API 替代 JNI 时明确生命周期与对齐策略。
- 度量虚拟线程迁移收益（吞吐、阻塞栈分布）而非盲目替换。


---

## 12. 数值与精度 (BigDecimal) [3]

### 12.1 精度与构造
- 是什么：BigDecimal 提供任意精度十进制表示（值 + scale）；避免二进制浮点误差。
- 为什么：金额、利率、结算、税率等需精确不可舍入误差。
- 怎么用：BigDecimal.valueOf(0.1) 或 new BigDecimal("0.10")；避免 new BigDecimal(double)。
- 关键细节：
  - value 与 scale：1.23 = value=123 scale=2；equals 比较同时考虑 scale，compareTo 忽略 scale。
  - 推荐用字符串或 valueOf；valueOf(double) 内部使用 Double.toString 保留语义。
  - 常量缓存：BigDecimal.ZERO/ONE/TEN。
  - 不可变；复合运算链避免中间对象过多可使用局部变量。
- 常见陷阱：
  - new BigDecimal(0.1) → 0.100000000000000005551...
  - equals 判断 1.0 与 1 不相等；集合 key 使用需统一 scale。
  - 直接除法未指定 RoundingMode 抛 ArithmeticException（除不尽）。
- 面试提示：给浮点误差例子并说明 compareTo 与 equals 差异。
- 案例：统一金额存储 scale=2 避免缓存命中差异。
- 速记：构造用字符串 / scale 影响 equals / compareTo 忽略 scale。

### 12.2 舍入与 scale
- 是什么：RoundingMode 定义除法与格式化的舍入策略；scale 决定小数位。
- 为什么：财务合规与统计准确性（HALF_EVEN 减少累积偏差）。
- 怎么用：amount.setScale(2, RoundingMode.HALF_EVEN)；divide(x, scale, mode)。
- 关键细节：
  - 常用模式：HALF_UP（传统四舍五入），HALF_EVEN（银行家），DOWN（截断），FLOOR（向负无穷）。
  - compareTo 忽略 scale：new BigDecimal("1.0").compareTo(new BigDecimal("1")) == 0。
  - 建议集中常量定义：static final int MONEY_SCALE = 2。
- 常见陷阱：未指定舍入导致除不尽异常；混用不同 scale 聚合后排序异常。
- 面试提示：解释为何财务统计偏好 HALF_EVEN。
- 案例：切换 HALF_UP→HALF_EVEN，累计误差降低。
- 速记：统一 scale + 明确 RoundingMode。

### 12.3 性能
- 是什么：高精度运算分配频繁；影响 GC 与热路径。
- 为什么：循环内大量创建 BigDecimal 成为热点。
- 怎么用：常量缓存；乘以放大因子用 long 表示“分”；必要时预聚合。
- 关键细节：
  - Map key 使用 BigDecimal 需规范 scale；否则 equals 不同导致重复条目。
  - 使用 long cents = amount.movePointRight(2).longValueExact() 再格式化输出。
  - 频繁除法/幂运算昂贵。
- 常见陷阱：随意转换 long↔BigDecimal 造成装箱与对象风暴；不同 scale key 导致缓存失效。
- 面试提示：强调“用整数表示最小货币单位”。
- 案例：批量结算改 long 分存储性能提升 30%。
- 速记：热路径用 long；BigDecimal 用于边界转换。

---

## 13. 对象与不可变设计 [4]

### 13.1 不可变好处
- 是什么：对象创建后状态不再变化；所有字段 final 且无暴露可变内部结构。
- 为什么：并发安全（无同步读）；缓存哈希稳定；降低推断复杂度。
- 怎么用：值对象（Money、UserId、Range）；构造完成即有效；禁止 setter。
- 关键细节：内部集合/数组 defensive copy；避免可变 Date → 使用 Instant/LocalDate。
- 常见陷阱：仅 final 引用但内部 List 可变；暴露内部数组引用。
- 面试提示：列举线程安全与缓存优势。
- 案例：订单金额值对象减少并发条件竞争。
- 速记：final + 无泄漏 + 深不可变。

### 13.2 Defensive Copy
- 是什么：对传入/返回的可变对象进行复制。
- 为什么：防止外部修改内部状态破坏封装。
- 怎么用：new ArrayList<>(inputList)；返回 Collections.unmodifiableList(copy)。
- 关键细节：对多层嵌套需深复制；时间类用不可变替代；注意性能权衡。
- 常见陷阱：直接返回内部 List；构造保存传入数组引用。
- 面试提示：举真实“修改外部集合影响对象内部” Bug。
- 案例：日志过滤配置被外部线程改写导致安全策略失效。
- 速记：入参复制 / 出参包装。

### 13.3 equals/hashCode 合约
- 是什么：equals 定义逻辑相等；hashCode 保证相等对象哈希一致。
- 为什么：HashMap/HashSet 正确性及查找性能。
- 怎么用：基于关键不可变字段生成；保持自反、对称、传递、一致。
- 关键细节：浮点 NaN；使用 Objects.hash 性能与稳定性；BigDecimal equals 与 compareTo 差异。
- 常见陷阱：重写 equals 未同步修改 hashCode；使用可变字段参与哈希后被修改导致定位失败。
- 面试提示：强调不可变字段构成哈希。
- 案例：使用可变 list 参与 equals 造成集合中“幽灵”元素。
- 速记：equals 与 hashCode 同步；只用不可变关键属性。

---

## 14. 设计与 API 可用性 [2]

### 14.1 语义清晰
- 是什么：命名与签名直观表达意图。
- 为什么：降低认知负担与误用风险。
- 怎么用：方法名动词+对象；避免 boolean 标志重载；使用枚举策略。
- 关键细节：过度重载导致歧义；参数顺序不一致影响理解。
- 常见陷阱：doProcess、handleStuff 模糊；多个 boolean 标志失语义。
- 面试提示：提简化前后差异。
- 案例：rename calculate(a,b,true,false) → calculateWithTaxAndDiscount(a,b)。
- 速记：命名自解释；少标志多对象。

### 14.2 参数验证
- 是什么：前置校验确保方法边界。
- 为什么：快速失败易定位；避免后续链路污染。
- 怎么用：Objects.requireNonNull；自定义 Preconditions；异常含上下文。
- 关键细节：IllegalArgument vs IllegalState；错误码标准化。
- 常见陷阱：延迟 NPE；吞异常返回 null。
- 面试提示：强调“早失败 + 明确异常”。
- 案例：参数校验减少 30% 运行时模糊错误。
- 速记：入口校验统一工具。

---

## 15. 资源管理与 AutoCloseable [3]

### 15.1 try-with-resources
- 是什么：编译器生成自动关闭结构；逆序关闭。
- 为什么：避免泄漏与模板代码。
- 怎么用：try (InputStream in = ...; BufferedReader br = ...) { ... }
- 关键细节：suppressed 异常 getSuppressed(); 自动关闭仅针对 AutoCloseable。
- 常见陷阱：在 finally 再重复关闭；忽略 suppressed 导致排查困难。
- 面试提示：比较手动 finally 漏关风险。
- 案例：数据库连接未关闭引起池耗尽通过迁移修复。
- 速记：try-with-resources = 自动+逆序+suppressed。

### 15.2 Cleaner 取代 finalize
- 是什么：Cleaner 注册回调在对象不可达后执行；替代不确定 finalize。
- 为什么：finalize 性能差且不可预测；可能延迟释放关键资源。
- 怎么用：Cleaner.create(); cleaner.register(obj, runnable)。
- 关键细节：非实时；优先显式 close；仅兜底。
- 常见陷阱：依赖 Cleaner 处理必须及时释放的资源。
- 面试提示：说明 finalize 弃用理由（不可预期+安全风险）。
- 案例：native 内存泄漏通过显式 close 修复。
- 速记：Cleaner=兜底，不代替显式关闭。

---

## 16. 内存与对象模型基础 [3]

### 16.1 对象头
- 是什么：Mark Word(锁/年龄/哈希) + Klass Pointer；可含压缩指针。
- 为什么：理解锁状态升级与调优。
- 怎么用：分析同步竞争；观察对象哈希变化。
- 关键细节：偏向→轻量→重量级；hashCode 计算后写入 Mark Word；JOL 可打印布局。
- 常见陷阱：误解偏向锁始终优势（高冲突场景反而频繁撤销）。
- 面试提示：阐述锁升级路径与触发条件。
- 案例：关闭偏向锁启动加快（低锁争用系统）。
- 速记：Mark Word 存锁与哈希；升级阶梯三段。

### 16.2 对齐与填充
- 是什么：对象 8 字节对齐；字段排列可能出现填充；缓存行为影响伪共享。
- 为什么：提升访问效率；减少总线争用。
- 怎么用：字段聚类；@Contended 避免共享缓存行伪共享。
- 关键细节：伪共享导致多线程写同缓存行反复失效；@Contended 默认需 -XX:-RestrictContended。
- 常见陷阱：误用 @Contended 滥占内存；忽略数组元素伪共享。
- 面试提示：描述伪共享症状（CPU 高、吞吐低）。
- 案例：统计计数分片加 @Contended 提升 2x。
- 速记：对齐提升访问；伪共享=缓存行竞争。

### 16.3 装箱与缓存
- 是什么：基本类型包装对象；缓存小范围（Integer [-128,127] 等）。
- 为什么：装箱产生额外对象与潜在 GC；频繁拆箱可能抛 NPE。
- 怎么用：性能敏感使用原始类型；计数采用 LongAdder；避免在热循环自动装箱。
- 关键细节：== 比较包装仅对缓存区内可能同引用；外部区间需 equals；自动拆箱 null → NPE。
- 常见陷阱：以 == 判断 Integer 超过缓存区；在集合中频繁装箱拆箱。
- 面试提示：说明“热点使用原始类型减少分配”。
- 案例：Long 计数改 LongAdder 增吞吐。
- 速记：装箱=对象+GC；缓存=小区间复用。

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