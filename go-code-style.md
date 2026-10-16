# Go Code Style

本规范适用于 Go 项目的代码编写、重构与审查，与全局通用规范共同使用，优先保证正确性、可读性、一致性和可维护性。

## 格式与命名

- 使用 `gofmt` 统一格式，不手动规定空格缩进或固定行宽；使用 `goimports` 时保持相同格式基线。
- 标识符使用 `MixedCaps` 或 `mixedCaps`，按是否导出决定首字母大小写；常量同样使用 `DefaultTimeout`、`maxRetries`，不使用全大写下划线形式。
- 普通包名使用简短的小写名称，不用下划线或驼峰；避免职责不明的 `util`、`common`、`helpers` 等包名。
- 文件名默认使用小写单词和下划线，如 `user_service.go`；平台及测试后缀遵循 Go 工具链约定。
- 保持惯用缩写大小写一致，使用 `ID`、`URL`、`HTTP`，例如 `userID`、`ServeHTTP`，不写成 `userId`、`ServeHttp`。
- 名称与作用域相称，局部允许 `i`、`err`、`ctx` 等惯用短名；避免重复包名、无意义后缀和自造缩写。
- 构造函数按上下文使用 `New` 或 `NewType`；简单取值方法不机械添加 `Get` 前缀。
- 导入先标准库，再其他包，以空行分组；避免无必要的别名和点导入，副作用导入仅用于明确的注册需求。

参考：[Go Code Review Comments](https://go.dev/wiki/CodeReviewComments) · [Effective Go — Names](https://go.dev/doc/effective_go#names) · [gofmt](https://pkg.go.dev/cmd/gofmt)

## 包与文件组织

- 按职责和依赖边界组织包，相关类型、方法与辅助函数就近放置。
- 默认使用未导出标识符，仅开放实际需要的 API；使用 `internal` 限制不应对外开放的包。
- 多个可执行入口可放入 `cmd/<name>`；不强制所有项目采用相同目录层级或创建 `pkg` 目录。
- 不以文件行数或类型数量机械拆包；不通过共享包掩盖职责混乱或循环依赖。
- 不为每个结构体单独建包，不按 `interfaces`、`types` 等技术标签拆散同一职责。
- 配置加载和依赖组装优先显式完成；避免在 `init()` 中执行 I/O、读取运行配置或启动 goroutine。

参考：[Organizing a Go Module](https://go.dev/doc/modules/layout) · [Google Go Style Guide](https://google.github.io/styleguide/go/guide) · [Uber — Avoid init](https://github.com/uber-go/guide/blob/master/style.md#avoid-init)

## 文件布局

代码按照自上而下的阅读顺序组织，优先展示主要概念和公开入口，再展开实现细节。

1. 构建约束、包注释、`package` 声明及导入。
2. 常量和必要的包级变量。
3. 主要类型、接口及核心数据结构。
4. 构造函数、公开函数和方法。
5. 私有辅助函数与内部实现。

- 同一类型的方法集中组织，按构造函数、公开方法、私有方法排列。
- 从属类型靠近对应主类型；同一 API 的相关类型按请求、成功响应、错误排列。
- 同一职责内，调用者优先放在被调用者之前；主体辅助逻辑在前，简单末端工具在后。
- 测试文件按导入、测试常量、测试用例、辅助函数排列，不在生产文件中放置测试。
- 上述阅读顺序以代码内聚性为先，不为机械排序破坏包初始化顺序或构建约束；业务初始化显式组织，避免依赖跨文件排列。

参考：[Uber — Function Grouping and Ordering](https://github.com/uber-go/guide/blob/master/style.md#function-grouping-and-ordering) · [Go Command — Build Constraints](https://pkg.go.dev/cmd/go#hdr-Build_constraints)

## 类型与 API 设计

- 使用明确的结构体和命名类型表达数据；需要区分不同领域概念时，使用类型定义，而不是仅创建类型别名。
- 状态使用命名类型和类型化常量；连续取值可用 `iota`，协议或持久化中的枚举值须保持稳定，优先显式指定，避免插入或重排改变旧值；外部数值仍需校验，不假定编译器会穷举检查。
- 优先提供有意义的零值；需要建立不变量或注入依赖时使用构造函数。
- 复用 `io.Reader`、`io.Writer`、`error` 等已有契约；不为相同语义另造接口。
- 接口通常由使用方定义，只包含实际需要的方法；公共协议本身是产品时，可以在协议包定义接口。
- 不为每个结构体预设接口，也不单纯为 Mock 创建完整的实现侧接口。
- 构造函数通常返回具体类型；确有封装或公共协议需要时，才返回接口。
- 泛型用于真实的类型参数化需求；不以 `any`、反射或大量类型断言替代明确类型，也不为单一实现强加泛型。
- 参数和返回值保持一致语义；简单查找可用 `(value, ok)`，需要说明失败原因时使用末尾的 `error` 返回值。
- 配置结构体和 Functional Options 按实际构造需求使用，不为简单对象增加 Builder 或多层工厂。
- 变更公开接口的方法、可识别的错误类别或序列化契约时，检查对现有调用方、实现方及存量数据的兼容性。

参考：[Google Go Style Decisions — Interfaces](https://google.github.io/styleguide/go/decisions#interfaces) · [When To Use Generics](https://go.dev/blog/when-generics) · [Go Specification — Type Declarations](https://go.dev/ref/spec#Type_declarations)

## 值、指针与集合

- 按修改语义、复制成本和可空契约选择值或指针，不为“可能更快”把所有参数改成指针。
- 接口通常直接传值，不传接口指针；字符串、切片和 map 也不因担心复制底层数据而机械改为指针。
- 明确切片与 map 的共享行为；需要独立数据时进行适当复制，不把赋值或浅拷贝视为深拷贝。
- 不复制已使用的 `sync.Mutex`、`sync.WaitGroup` 等同步对象；包含这些字段的结构体同样遵守其复制限制。
- 没有特殊契约时，空切片优先使用零值；需要区分 JSON `null` 与 `[]` 时，按接口要求处理。
- 写入 map 前完成初始化；需要区分缺失与合法零值时，使用 `value, ok := m[key]`。
- 不依赖 map 的遍历顺序；需要稳定输出时显式排序。
- 字符串索引用于字节处理；按 Unicode 码点遍历时使用 `range`，不混淆字节偏移与码点。

参考：[Go Specification — Representation of Values](https://go.dev/ref/spec#Representation_of_values) · [sync](https://pkg.go.dev/sync) · [Go Code Review Comments — Empty Slices](https://go.dev/wiki/CodeReviewComments#declaring-empty-slices)

## 控制流与惯用写法

- 优先处理错误和特殊情况并提前返回，使正常路径保持较少缩进。
- 前一分支已经 `return`、`continue` 或 `break` 时，不再增加无必要的 `else`。
- 变量作用域保持最小，避免无意遮蔽外层变量，特别是需要后续读取的 `err`。
- 能用零值、`range`、`append`、`copy` 和标准库直接表达的逻辑，不增加自定义包装。
- 不先检查切片是否为 `nil` 再调用本来就支持 nil 切片的 `len`、`range` 或 `append`。
- 在信任边界校验外部数据的长度、范围及整数转换的可表示性，读取或分配前落实资源上限；仅在约束已可靠建立且持续有效时省略重复校验，不删除必要的权限和资源保护。
- 不通过默认值、无依据的类型转换或丢弃返回值掩盖错误与缺失数据。

参考：[Effective Go — Control Structures](https://go.dev/doc/effective_go#control-structures) · [Go Specification — Built-in Functions](https://go.dev/ref/spec#Built-in_functions)

## 错误处理

- 可恢复失败返回 `error`，通常放在返回值末尾；不使用 `panic` 处理普通输入错误或业务失败。
- 错误需要处理、传播或明确说明为何可以忽略，不使用 `_` 静默丢弃有意义的失败。
- 使用 `errors.Is` 判断错误类别、`errors.As` 提取错误类型，不通过比较错误字符串决定业务分支。
- 只有允许调用方识别底层错误时才使用 `%w`；跨抽象边界按公开契约转换错误，不无条件暴露底层实现。
- 错误上下文说明失败操作和必要对象，不重复堆叠同义包装；同一失败避免逐层记录日志。
- 错误变量采用 `ErrNotFound` 或 `errNotFound`，自定义错误类型采用 `ParseError` 等名称；英文错误信息通常小写开头且不加句末标点。
- 成功时直接返回 `nil`，不返回装有 nil 指针的非 nil `error` 接口值。
- `recover` 仅用于明确的隔离边界或内部协议，不把程序错误转换成成功；库代码不使用 `os.Exit` 或致命日志终止进程。

参考：[Working with Errors in Go](https://go.dev/blog/go1.13-errors) · [Google — Error Best Practices](https://google.github.io/styleguide/go/best-practices#errors) · [Go FAQ — Nil Errors](https://go.dev/doc/faq#nil_error)

## 函数与方法

- 函数聚焦明确职责，名称表达行为和副作用；同类操作保持一致的参数顺序与返回语义。
- 接收者使用类型相关的简短名称，如 `c`、`srv`，同一类型保持一致，不使用 `this`、`self`。
- 需要修改接收者、避免复制大对象或包含不可安全复制字段时，使用指针接收者；小型值语义类型可使用值接收者。
- 同一类型的接收者选择保持一致，不随意混用以制造难以理解的方法集或复制行为。
- 命名返回值用于澄清语义或支持必要的延迟处理；较长函数显式写出返回值，不为省字使用裸返回。
- 默认提供同步接口，让调用方决定并发；异步接口必须明确结果、错误和任务生命周期。
- 删除无价值的纯转发与重复实现，但保留承担真实契约、适配或隔离职责的包装。

参考：[Google Go Style Decisions — Receiver Type](https://google.github.io/styleguide/go/decisions#receiver-type) · [Go Code Review Comments — Synchronous Functions](https://go.dev/wiki/CodeReviewComments#synchronous-functions)

## Context 与并发

- 需要取消、超时或请求级信息的函数显式接收 `ctx context.Context`，放在第一个参数；不传 `nil`。
- 沿调用链传递或派生 Context，不用 `context.Background()` 随意切断上游取消和截止时间。
- 不将请求 Context 保存在长期对象中；`WithValue` 只用于跨 API 传递的请求级信息，不代替业务参数或依赖注入。
- 创建可取消或超时 Context 后，确保调用对应 `cancel`；取消只发出信号，不代表后台工作已经停止。
- 启动 goroutine 前明确谁负责等待、退出和错误处理；避免脱离管理的后台任务。
- 并发数量和队列容量应有依据；为阻塞的发送、接收和 I/O 提供相应的取消或截止机制。
- channel 的关闭由能够确认不再发送的一方负责；通过同步防止重复关闭和关闭后发送。
- 保护共享可变状态，按语义选择锁、channel 或原子操作；不以固定 `time.Sleep` 代替同步。
- 缩小锁的作用范围，避免持锁执行无必要的慢 I/O 或未知回调；选择满足需求的最简单并发模型。

参考：[context](https://pkg.go.dev/context) · [Go Concurrency Patterns — Pipelines and Cancellation](https://go.dev/blog/pipelines) · [sync](https://pkg.go.dev/sync)

## 性能与资源

- 性能优化以 Benchmark、`pprof`、执行追踪和真实负载为依据，优先检查复杂度、分配、复制和锁竞争。
- 容量可合理预估时预分配，热点路径避免重复转换、重复计算和无必要的中间集合。
- 文件、连接、响应体等资源明确归属；成功获取后及时安排清理，注意循环中的 `defer` 会延迟到函数返回。
- 对影响结果的 `Flush`、`Close`、提交等错误进行处理；数据库结果迭代结束后检查 `Rows.Err()`。
- HTTP 客户端、Transport 和数据库连接池按其生命周期复用，不在每次操作中重复创建。
- `http.Client.Do` 不会仅因非 2xx 状态返回错误；收到响应后按接口契约检查状态码。
- `sync.Pool` 仅用于可丢弃、可重建的临时对象，不承担持久缓存或对象生命周期管理。
- 反射、`unsafe` 和复杂对象池必须有明确必要性；优化不得破坏类型安全、数据契约和资源上限。

参考：[Go Diagnostics](https://go.dev/doc/diagnostics) · [net/http](https://pkg.go.dev/net/http) · [database/sql](https://pkg.go.dev/database/sql) · [sync.Pool](https://pkg.go.dev/sync#Pool)

## 注释与文档

- 注释解释设计意图、关键约束和非显而易见的原因，不机械复述代码。
- 导出的类型、函数、方法和包提供文档注释；声明注释通常以对应名称开头，普通包注释以 `Package <name>` 开头，命令包注释通常以程序名起句。
- 公共接口说明必要的零值行为、错误条件、并发安全性、取消语义及资源归属。
- 对不可复制的类型、共享切片或 map、特殊生命周期及 `unsafe` 使用前提明确说明。
- 为重要或易误用的公共 API 提供 `Example`；需要由 `go test` 执行验证时，包含 `Output:` 或 `Unordered output:` 注释，否则示例只编译、不执行；保持注释、示例与实现同步，删除失效说明。

参考：[Go Doc Comments](https://go.dev/doc/comment) · [Google — Documentation](https://google.github.io/styleguide/go/best-practices#documentation)

## 测试与验证

- 普通单元测试与源码同目录，默认 `foo.go` 对应 `foo_test.go`；同一源码的测试集中维护，平台、构建标签或独立测试包确有需要时再拆分。
- 包内测试使用同名包；仅通过公开 API 验证时可使用 `<package>_test`，不为测试随意扩大生产 API。
- 测试名称遵循 `TestXxx`、`BenchmarkXxx`、`FuzzXxx`、`ExampleXxx` 等约定，准确表达被测行为。
- 多组输入与预期采用表驱动测试及 `t.Run`；复杂场景独立组织，不强行放入万能测试表。
- 测试辅助函数使用 `t.Helper()`；使用 `t.Cleanup()`、`t.TempDir()` 管理测试资源。
- 失败信息说明场景、实际值与预期值；`t.Fatal`、`t.FailNow` 由测试 goroutine 调用，子 goroutine 的结果交回测试入口处理。
- 仅在数据和资源互不干扰时使用 `t.Parallel()`；避免依赖遍历顺序、固定休眠或不受控的外部环境。
- 覆盖正常路径、重要边界和错误路径；缺陷修复补充回归测试，解析与编解码逻辑按需增加 Fuzz 测试。
- 并发代码执行 Race Detector；测试通过不等于证明所有执行路径都没有数据竞争。

参考：[testing](https://pkg.go.dev/testing) · [Go Test Comments](https://go.dev/wiki/TestComments) · [Data Race Detector](https://go.dev/doc/articles/race_detector)

## 工具与检查

在单个 Go module 根目录执行基础检查；`go fmt` 调用 `gofmt` 并会修改源码。

```bash
go fmt ./...
go build ./...
go vet ./...
go test ./...

# 在支持 Race Detector、已满足其 cgo 和编译器要求的环境执行。
go test -race ./...
```

- CI 检查格式差异，不静默改写源码；使用 `gofmt -l` 时，还须判断输出是否为空及命令是否执行成功。
- 语言特性和标准库 API 的使用须符合项目 `go.mod` 的 `go` 版本基线及支持矩阵；构建标签、cgo 与平台组合由项目确定，多模块仓库分别验证，不假定一次 `./...` 覆盖全部模块。
- `goimports`、Staticcheck 等工具固定与项目工具链兼容的版本；按项目约定运行，不因升级而无依据禁用检查。
- 规则例外限定到必要范围并说明原因；不通过删除断言、跳过测试或关闭检查换取通过。
- 验证结果以实际执行为准；无法运行的检查说明原因和未验证范围。

参考：[Go Command](https://pkg.go.dev/cmd/go) · [gofmt](https://pkg.go.dev/cmd/gofmt) · [Staticcheck](https://staticcheck.dev/docs/) · [Race Detector Requirements](https://go.dev/doc/articles/race_detector#Requirements)

---

## 延伸阅读

| 参考资料 | 重点内容 |
|---|---|
| [Go Code Review Comments](https://go.dev/wiki/CodeReviewComments) | 命名、错误、接口、接收者与审查惯例 |
| [Effective Go](https://go.dev/doc/effective_go) | 基础语言惯用法；不作为新特性与新库的唯一依据 |
| [Google Go Style Guide](https://google.github.io/styleguide/go/guide) | 清晰、简单、简洁、可维护与一致性原则 |
| [Google Go Style Decisions](https://google.github.io/styleguide/go/decisions) | 命名、接口、泛型、接收者和具体写法 |
| [Google Go Best Practices](https://google.github.io/styleguide/go/best-practices) | API、错误、文档与测试实践 |
| [Uber Go Style Guide](https://github.com/uber-go/guide/blob/master/style.md) | 文件顺序、初始化、并发与工程惯例；按适用范围采用 |
| [Go Language Specification](https://go.dev/ref/spec) | 类型、值、控制流和语言语义 |
| [Go Diagnostics](https://go.dev/doc/diagnostics) | 性能剖析、执行追踪与运行诊断 |
