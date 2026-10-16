# Rust Code Style

本规范适用于 Rust 项目的代码编写、重构与审查。遵循 Rust 官方约定及惯用实践，优先保证正确性、可读性、一致性和可维护性。

必须／不得表示硬性约束；优先／避免表示默认建议，可根据实际需求权衡。

## 格式与命名

- 使用 `rustfmt` 统一格式，默认 4 空格缩进、100 字符行宽。
- 模块、文件、函数、变量使用 `snake_case`；类型、Trait、枚举变体使用 `UpperCamelCase`；常量和静态变量使用 `SCREAMING_SNAKE_CASE`。
- 名称准确表达职责，优先沿用标准库和 Rust 生态惯用术语。
- 布尔值优先使用 `is_`、`has_`、`can_` 等能明确表达含义的名称。
- 类型转换遵循 `as_`、`to_`、`into_` 约定，迭代方法遵循 `iter`、`iter_mut`、`into_iter` 约定。
- 避免无意义缩写、重复上下文、类型名称冗余和不一致的术语。
- 导入路径清晰一致，避免不必要的通配符导入。

参考：[Rust Style Guide](https://doc.rust-lang.org/style-guide/) · [Rust API Naming](https://rust-lang.github.io/api-guidelines/naming.html)

## 模块与文件组织

- 按职责和抽象边界组织模块，保持高内聚、低耦合。
- 文件名表达主要职责，相关类型及实现优先就近组织。
- 默认保持最小可见性，根据 API 边界使用 `pub`、`pub(crate)` 等。
- crate 根模块主要负责模块声明、公共入口和必要的重新导出。
- 优先复用已有实现，不引入职责重叠的模块、平行抽象或重复封装。
- 不按固定行数或类型数量机械拆分文件。
- 模块之间通过明确的接口协作，避免不必要的跨层依赖。
- 仅在边界清晰且有实际复用需求时提取公共模块。

参考：[rust-analyzer Style Guide](https://rust-analyzer.github.io/book/contributing/style.html) · [Rust Modules](https://doc.rust-lang.org/reference/items/modules.html)

## 文件布局

代码按照自上而下的阅读顺序组织，优先展示主要概念和公开入口，再展开实现细节。

1. 模块文档、属性及导入。
2. 常量、类型、Trait 和核心数据结构。
3. 主要公开函数及核心实现。
4. 私有辅助函数和内部实现。
5. 测试模块。

- 相关 `impl`、Trait 实现与辅助函数尽量靠近对应类型。
- 主要业务入口应易于定位，避免被大量内部实现淹没。
- 遵循标准库、外部依赖和本地模块的导入分组约定。
- 文件布局以职责和阅读逻辑为准，不为机械排序破坏代码内聚性。

参考：[rust-analyzer Style Guide](https://rust-analyzer.github.io/book/contributing/style.html)

## 类型与 API 设计

- 优先使用类型系统表达约束，使非法状态尽可能无法构造。
- 使用 `enum` 表达互斥状态，使用 `struct` 表达组合数据。
- 使用 Newtype 区分具有不同语义的基础类型。
- 优先复用标准 Trait，仅在具有明确抽象需求时定义自定义 Trait。
- 有明确接收者的操作优先设计为方法，保持关联函数与普通函数的职责清晰。
- API 参数和返回值应表达真实的所有权与失败语义。
- 公共 API 不暴露不必要的内部实现细节。
- 泛型、关联类型和 Trait Bound 以实际复用需求为依据。
- 复杂对象根据实际构造需求使用 Builder，不为简单结构增加额外抽象。
- 避免仅为未来假设的扩展场景设计复杂接口。

参考：[Rust API Guidelines](https://rust-lang.github.io/api-guidelines/) · [Type-Driven Design](https://rust-unofficial.github.io/patterns/patterns/behavioural/newtype.html)

## 所有权与内存

- 优先借用，仅在确实需要独立所有权时移动或克隆数据。
- 字符串和切片输入优先使用 `&str`、`&[T]`，避免不必要的 `&String`、`&Vec<T>`。
- 减少不必要的 `clone()`、`to_owned()`、`to_string()` 和 `collect()`。
- 避免不必要的堆分配和引用计数，根据对象大小、生命周期及性能测量选择存储与所有权方式。
- 使用 `Arc`、`Mutex`、`RwLock` 等共享机制时明确所有权和同步语义。
- 仅在确有需要时引入显式生命周期、内部可变性或动态分发。
- 对已知规模的集合合理预分配容量，适当复用缓冲区。
- 优先使用安全 Rust，必要的 `unsafe` 应限制在最小可审查范围。
- 安全 API 必须通过类型约束、内部不变量或运行时检查等机制保证内存安全，不得仅依赖调用方遵守文档约定。

参考：[Ownership](https://doc.rust-lang.org/book/ch04-00-understanding-ownership.html) · [Rustonomicon](https://doc.rust-lang.org/nomicon/) · [Unsafe Code Guidelines](https://rust-lang.github.io/unsafe-code-guidelines/)

## 控制流与惯用写法

- 优先使用表达式直接产生结果，减少不必要的可变状态和重复赋值。
- 优先使用模式匹配表达状态分支，选择最清晰的 `match`、`if let` 或 `let else`。
- 尽可能穷举处理枚举状态，避免无意义的 `_` 分支掩盖新状态。
- 集合过滤、转换和聚合优先考虑 Iterator，复杂控制流可使用显式循环。
- 优先使用提前返回减少嵌套，避免冗长的条件判断。
- 合理使用 `const`、模式匹配和标准库组合子表达逻辑。
- 避免不必要的可变全局状态。
- 不重复验证已由类型系统、构造过程或明确边界保证的条件。
- 在简洁性与可读性冲突时，优先选择更清晰的实现。

参考：[Rust Cheat Sheet — Coding Guides](https://cheats.rs/) · [Rust Idioms](https://rust-unofficial.github.io/patterns/idioms/index.html)

## 错误处理

- 可恢复错误使用 `Result`，可选值使用 `Option`。
- 优先使用 `?` 传播错误，避免重复展开错误处理分支。
- 错误类型应准确表达失败原因，必要时附加有意义的上下文。
- 调用方需要分类处理时提供清晰的错误分类，包装底层错误时保留错误来源链。
- 仅在不变量具有明确保证时使用 `unwrap()` 或 `expect()`。
- `panic!` 用于不可恢复的程序错误或违反不变量的情况，不代替普通错误返回。
- 避免无意义的错误包装、重复检查及无差别捕获错误。
- 同类接口保持一致的错误类型、上下文和传播方式。

参考：[Rust Error Handling](https://doc.rust-lang.org/book/ch09-00-error-handling.html) · [Rust API Guidelines — Error Types](https://rust-lang.github.io/api-guidelines/interoperability.html)

## 函数与实现

- 函数聚焦明确职责，名称表达真实行为和副作用。
- 优先采用直接、易读的实现，不为简单逻辑引入额外间接层。
- 避免单纯转发调用的包装函数和缺少价值的中间抽象。
- 同类操作保持一致的参数顺序、返回类型和处理模式。
- 相关逻辑就近组织，重复逻辑提取为具有明确职责的共享实现。
- 保持单一事实来源，避免多个位置维护相同状态或规则。
- 不按固定行数拆分函数，以职责、复杂度和可读性决定边界。
- 优先使用标准库现有能力，避免重新实现成熟的基础功能。

参考：[Rust Compiler Conventions](https://rustc-dev-guide.rust-lang.org/conventions.html) · [rust-analyzer Style Guide](https://rust-analyzer.github.io/book/contributing/style.html)

## 异步与并发

- 按异步等待或接口契约需要使用 `async`，避免无意义的异步包装。
- 避免在异步执行路径中直接执行长时间阻塞操作。
- 不跨 `.await` 持有不必要的锁、借用或其他受限资源。
- 明确任务所有权、生命周期、取消、超时及资源清理语义。
- 优先使用有界并发和背压机制，避免无控制地创建任务。
- 减少不必要的共享状态，优先选择满足需求的简单并发模型。
- 同步原语与任务调度策略应符合实际执行环境和负载特征。
- 异步操作应考虑取消安全性，确保关键状态与资源得到正确处理。

参考：[Async Book](https://rust-lang.github.io/async-book/) · [Tokio Tutorials](https://tokio.rs/tokio/tutorial) · [Rust Cheat Sheet — Async](https://cheats.rs/)

## 性能与资源

- 性能优化以 Benchmark、Profiling 和真实工作负载为依据。
- 优先关注算法复杂度、内存分配、数据复制及访问模式。
- 热路径减少不必要的中间集合、动态分发和重复计算。
- 根据数据规模合理预分配、批量处理或复用缓冲区。
- 根据数据访问模式选择结构布局，必要时评估缓存局部性和 SIMD。
- 并发性能同时考虑锁竞争、任务开销及数据共享成本。
- 避免缺乏测量依据的微优化和过早优化。
- 优化不得破坏正确性、API 契约或必要的可维护性。

参考：[Rust Performance Book](https://nnethercote.github.io/perf-book/) · [Rust Cheat Sheet — Performance Tips](https://cheats.rs/)

## 注释与文档

- 注释解释设计意图、关键约束和非显而易见的原因，不重复描述代码行为。
- 公共 API 使用 `///` 文档注释，模块使用 `//!` 说明职责。
- 公共接口根据需要说明参数语义、错误条件、Panic 条件和使用示例。
- 需要调用方额外保证安全前提的函数使用 `unsafe fn`，需要实现方额外保证安全前提的 Trait 使用 `unsafe trait`，并在 `# Safety` 文档中说明相应契约；Trait 中方法的调用是否需要 `unsafe`，由方法自身的声明决定。
- 内部每个 `unsafe` 操作都必须满足其安全前提与不变量，并使用 `// SAFETY:` 说明局部依据。
- 优先编写可运行的文档示例，保持文档与实际实现一致。
- 删除过时、重复和无法提供额外信息的注释。

参考：[Rustdoc Book](https://doc.rust-lang.org/rustdoc/) · [Rust API Documentation Guidelines](https://rust-lang.github.io/api-guidelines/documentation.html)

## 测试与验证

- 单元测试优先就近放置在对应模块的 `#[cfg(test)] mod tests` 中。
- 集成测试放在 `tests/`，围绕公共行为和实际使用场景组织。
- 测试名称准确描述被测行为和预期结果。
- 测试覆盖正常路径、重要边界条件及有意义的错误路径。
- 优先使用简洁、稳定、可复现的测试数据。
- 复用测试辅助实现，避免重复搭建相同测试环境。
- 公共 API 的使用示例可通过文档测试验证。
- 使用 `rustfmt`、Clippy 和 Cargo Test 进行持续验证。

参考：[Rust Testing](https://doc.rust-lang.org/book/ch11-00-testing.html) · [Clippy](https://doc.rust-lang.org/clippy/) · [Cargo Test](https://doc.rust-lang.org/cargo/commands/cargo-test.html)

## 工具与检查

项目使用标准 Rust 工具链保持代码规范一致。

```bash
cargo fmt --all --check
cargo clippy --workspace --all-targets --all-features -- -D warnings
cargo test --workspace --all-features
```

- Rust Edition、工具链版本及 Feature 组合由项目规范确定。
- 格式规则优先交给 `rustfmt`，惯用写法与静态检查优先交给 Clippy。
- 项目级 Clippy 配置应明确、稳定；`clippy.toml` 仅调整规则支持的配置参数。
- Clippy 规则例外限定在最小必要作用域，指定具体 lint 并说明原因，避免无依据地全局禁用；工具链支持时可使用 `#[expect(clippy::具体规则, reason = "...")]`。
- Feature 存在互斥关系时，按项目定义的有效组合分别验证。
- 测试与检查结果必须以实际执行结果为准。

参考：[Rustfmt](https://github.com/rust-lang/rustfmt) · [Clippy Lints](https://rust-lang.github.io/rust-clippy/master/)

---

## 延伸阅读

| 参考资料 | 重点内容 |
|---|---|
| [Rust Language Cheat Sheet](https://cheats.rs/dl/rust_cheat_sheet_a4.pdf) | Rust 语言机制、惯用写法、性能与异步 |
| [rust-analyzer Style Guide](https://rust-analyzer.github.io/book/contributing/style.html) | 代码组织、文件布局、模块边界 |
| [Rust API Guidelines](https://rust-lang.github.io/api-guidelines/) | 命名、类型、Trait、API 设计 |
| [Rust Compiler Development Guide](https://rustc-dev-guide.rust-lang.org/conventions.html) | 编译器项目编码约定 |
| [Pragmatic Rust Guidelines](https://microsoft.github.io/rust-guidelines/) | 生产级 Rust 工程实践 |
| [Rust Design Patterns](https://rust-unofficial.github.io/patterns/) | Idioms、设计模式与反模式 |
| [The Rust Performance Book](https://nnethercote.github.io/perf-book/) | 性能测量与优化 |
| [The Rustonomicon](https://doc.rust-lang.org/nomicon/) | Unsafe Rust、内存与安全约束 |
