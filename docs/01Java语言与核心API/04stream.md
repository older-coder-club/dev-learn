## 函数式与 Stream API [4]

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
