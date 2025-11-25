
## 11. Optional 最佳实践 [3]

### 11.1 语义定位
是什么：表示“可能缺失”的返回值。  
为什么：替代返回 null 的潜在风险，提高表达力。  
怎么用：服务层返回 Optional；链式处理。  
关键细节：不作为字段、序列化成本高；不用于集合元素。  
面试提示：强调其使用边界与正确场景。

### 11.2 链式操作
是什么：map/flatMap/filter 对值操作组合。  
为什么：避免嵌套 if null。  
怎么用：opt.filter(...).map(...).orElse(...); orElseGet 惰性。  
关键细节：避免 Optional.of(null) 导致 NPE；使用 ofNullable。  
面试提示：区分 orElse vs orElseGet 计算时机。

### 11.3 性能与替代
是什么：包装对象带开销。  
为什么：热点循环中大量创建影响 GC。  
怎么用：内部逻辑可用注释 + null；对外 API 用 Optional。  
关键细节：统一规范避免混乱；不要链式过度复杂。  
面试提示：说明性能权衡。


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