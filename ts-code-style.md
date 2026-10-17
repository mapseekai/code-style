# TypeScript Code Style

本规范适用于 TypeScript 项目的代码编写、重构与审查，与全局通用规范共同使用，优先保证正确性、可读性、一致性和可维护性。

## 格式与命名

- 使用 Prettier 统一格式，本规范建议的项目默认配置为 2 空格缩进、`singleQuote: true`、`semi: true`、`printWidth: 100`；行宽是排版目标，不是强制拆分代码的依据。
- 变量、参数、函数、方法和属性使用 `camelCase`；类型、接口、类和类型参数使用 `PascalCase`。
- 模块级固定常量使用 `UPPER_SNAKE_CASE`，普通 `const` 绑定仍使用 `camelCase`，不将所有不可重新赋值的变量都命名为常量。
- 缩写按单词参与驼峰命名，使用 `userId`、`parseUrl`、`HttpClient`；平台已有名称和外部协议字段保持原样。
- 普通文件默认使用 `kebab-case.ts`，如 `user-service.ts`；JSX 使用 `.tsx`，框架要求及 `.d.ts`、`.mts`、`.cts` 等特殊文件另按实际用途命名。
- 名称表达实际职责；布尔值优先使用 `is`、`has`、`can` 等语义前缀，不给接口统一添加 `I` 前缀。
- 优先沿用领域术语和标准库命名，避免自造缩写、重复上下文和无意义的 `Impl`、`Manager`、`Helper` 后缀。

参考：[Google TypeScript Style Guide — Naming](https://google.github.io/styleguide/tsguide.html#naming) · [Prettier Options](https://prettier.io/docs/options)

## 模块与文件组织

- 按职责和依赖边界组织模块，相关类型、函数与实现就近放置；不按文件行数或类型数量机械拆分。
- 默认使用 ES Modules，优先具名导出；框架入口或外部契约要求默认导出时遵循其约定，不随意混用 ESM 与 CommonJS。
- 仅导出模块外确实需要的契约，不为测试或调用方便暴露内部实现，不导出无约束的共享可变状态。
- 仅用于类型的依赖使用 `import type`、`export type`；运行时需要的类、值及副作用使用真实导入。
- 导入按运行时内建模块、第三方依赖、项目模块和相对路径分组；保持路径大小写一致，不为排序改变必要的副作用顺序。
- `index.ts` 用于明确的公共入口，不为每层目录机械创建聚合导出；内部模块避免绕经自身聚合入口形成循环依赖。
- 普通业务模块不在导入时启动请求、定时器或后台任务；初始化显式完成，不用 `namespace` 或纯静态容器类重复包装模块作用域。

参考：[TypeScript Modules Reference](https://www.typescriptlang.org/docs/handbook/modules/reference.html) · [verbatimModuleSyntax](https://www.typescriptlang.org/tsconfig/verbatimModuleSyntax.html) · [Google TypeScript Style Guide — Modules](https://google.github.io/styleguide/tsguide.html#modules)

## 文件布局

代码按照自上而下的阅读顺序组织，优先展示主要概念和公开入口，再展开实现细节。

1. 必要的文件指令、模块说明及导入。
2. 文件级常量。
3. 主要类型、接口及核心数据结构。
4. 公开函数、类及核心实现。
5. 私有辅助函数与内部实现。

- 从属类型靠近对应主类型；同一 API 的相关类型按请求、成功响应、错误排列。
- 类内先展示字段，再按构造函数、公开方法、私有方法排列；关联的访问器和方法保持接近。
- 同一职责内，调用者优先放在被调用者之前；主体辅助逻辑在前，简单末端工具在后。
- 测试文件按导入、测试常量、测试用例、辅助函数排列，相关测试按场景分组。
- 不为机械排序破坏初始化顺序、框架指令或变量可用性；`const`、类和测试数据必须在实际执行到使用处之前完成初始化。

本节延续全局规范的阅读布局，是工程约定，不是 TypeScript 的语法要求。

## 类型与 API 设计

- 使用 `string`、`number`、`boolean` 等基础类型，不以 `String`、`Number`、`Boolean` 包装类型声明普通数据。
- 普通对象契约默认使用 `interface`；联合、元组、映射类型及其他类型组合使用 `type`，不对同一用途反复切换写法。
- 对外函数和方法显式表达参数与返回类型；局部变量及上下文明确的回调利用类型推断，不重复标注明显类型。
- 使用可辨识联合表达互斥状态，不用多个布尔字段和可选字段拼出大量无效组合；在分支中进行穷举检查。
- 有限状态优先使用字面量联合；需要运行时常量时可使用 `as const` 对象，不为简单状态额外引入枚举机制。
- 泛型用于表达真实的类型关系，优先复用 `Pick`、`Omit`、`Partial`、`Record` 等标准工具类型，不为单一类型制造复杂条件类型。
- 普通类型别名不会建立名义隔离；确需区分相同底层表示的领域标识时，再使用品牌类型及受控构造，不为所有字段增加包装。
- API 明确输入是否会被修改；只读输入使用 `readonly` 属性或 `readonly T[]`，动态字典准确表达键可能缺失，不把稀疏数据声明为完整记录。

参考：[Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html) · [Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html) · [Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)

## 类型安全与数据边界

- 开启 `strict`；未知输入优先使用 `unknown`，先收窄再使用。与不完整的第三方类型互操作时，将必要的 `any` 限制在适配边界。
- 网络响应、文件、配置和消息数据通过明确的解析或校验入口进入业务层；类型注解、`as T` 和泛型参数不能代替运行时校验。
- 不用 `as unknown as T`、非空断言 `!` 或空对象断言掩盖建模错误；确有类型系统无法表达的不变量时，限定范围并说明依据。
- 检查对象是否满足目标结构时，优先考虑类型注解或 `satisfies`；`satisfies` 只检查编译期兼容性，不转换或验证运行时数据。
- 区分属性缺失、`undefined`、`null`、空集合和失败；`field?: T` 不等于必需字段 `field: T | undefined`，外部协议中的空值语义不得擅自改变。
- 类型守卫和断言函数必须真实检查其承诺的条件；只有约束已可靠建立且持续有效时，才省略下游重复检查。
- `readonly`、`Readonly<T>` 和 `as const` 不会冻结运行时对象，`Readonly<T>` 也不是深度只读；共享数据仍需明确修改权限和复制边界。

参考：[Everyday Types — Type Assertions](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions) · [The satisfies Operator](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html#the-satisfies-operator) · [Object Types](https://www.typescriptlang.org/docs/handbook/2/objects.html) · [exactOptionalPropertyTypes](https://www.typescriptlang.org/tsconfig/exactOptionalPropertyTypes.html)

## 控制流与惯用写法

- 默认使用 `const`，需要重新赋值时使用 `let`，不使用 `var`；保持变量作用域最小。
- 优先处理错误和特殊情况并提前返回，正常路径保持较少嵌套；避免嵌套三元表达式和过度链式调用。
- 默认使用 `===`、`!==`；判断缺失时不误用真值判断，将合法的 `0`、`false` 或空字符串当作不存在。
- 仅为空值提供默认值时使用 `??`；只有所有假值都应触发回退时才使用 `||`，不机械互换。
- `?.` 只用于契约允许缺失的访问，不用连续可选链将必需数据缺失变成静默成功。
- 数组遍历使用 `for...of` 或合适的数组方法，不用 `for...in`；转换使用 `map`，复杂状态逻辑使用清楚的循环，不为简短强行使用 `reduce`。

参考：[TypeScript Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html) · [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html)

## 错误处理

- 抛出 `Error` 或有意义的子类，不抛出字符串、数字或任意对象；预期业务失败可按项目契约使用可辨识结果类型。
- 将捕获值视为 `unknown`，收窄后再读取错误属性；不假定所有第三方代码都抛出 `Error`。
- 在能够恢复、转换接口或决定结果的位置处理错误；其他位置保留原因传播，不空 `catch`、不把失败替换成成功默认值。
- 包装错误应增加操作上下文，运行环境支持时通过 `cause` 保留原始原因；不逐层重复包装或记录同一失败。
- 用明确的错误类别或稳定错误码判断分支，不依赖面向人的错误文案；跨进程、Worker 或序列化边界的错误按传输契约处理。
- 在 `try` 内需要捕获异步拒绝，或需要等待操作结束再清理资源时，使用 `await`；不机械删除具有这些语义的 `return await`。
- 重试、回退和错误忽略必须有明确适用场景，不以简化为由删除权限、隔离和资源保护。

参考：[useUnknownInCatchVariables](https://www.typescriptlang.org/tsconfig/useUnknownInCatchVariables.html) · [typescript-eslint — return-await](https://typescript-eslint.io/rules/return-await/) · [Google TypeScript Style Guide — Exceptions](https://google.github.io/styleguide/tsguide.html#exceptions)

## 函数与类

- 模块级普通函数优先使用函数声明，回调优先使用箭头函数；需要词法 `this` 时显式选择箭头函数。
- 优先用函数和普通对象表达简单能力；有状态、生命周期或明确实例语义时再使用类，不为每个类预设接口或基类。
- 参数和返回值保持一致语义；多个易混淆的同类型参数可改为具名选项对象，不为简单调用引入 Builder 或多层工厂。
- 能清楚表达相同调用语义时使用联合参数；只有输入与返回类型存在需要保留的对应关系时才增加重载。
- 字段在声明或构造过程中建立有效状态，不用无依据的确定赋值断言 `!` 绕过初始化；覆盖继承成员时使用 `override`。
- 明确 `private` 是类型检查层面的限制，`#field` 提供运行时私有性；按封装需求和目标环境选择，不把静态可见性当作安全隔离。
- 保留承担真实契约、适配或隔离职责的包装，删除没有价值的纯转发；方法作为回调传递时处理好 `this` 绑定。

参考：[More on Functions](https://www.typescriptlang.org/docs/handbook/2/functions.html) · [Classes](https://www.typescriptlang.org/docs/handbook/2/classes.html)

## 异步与并发

- 异步接口明确返回 `Promise<T>`，不让同一契约随路径返回同步值或 Promise；已有外部接口允许两者时在适配边界处理。
- 创建的 Promise 必须被等待、返回给调用方或安排拒绝处理；`void promise` 只表示不等待，不会捕获拒绝。
- 不使用 `forEach(async ...)` 等方式冒充已等待的批处理；串行使用 `for...of` 配合 `await`，并发任务显式聚合。
- 所有任务成功才继续时使用 `Promise.all`，需要逐项结果时使用 `Promise.allSettled`；不把前者的提前拒绝当作其他任务已经取消。
- 并发数量和排队容量必须受控，不对不受控数据直接启动全部请求；复用项目已有调度能力。
- 需要取消的 API 接收并传递 `AbortSignal`；自定义操作处理已取消状态、取消事件和清理，避免切断上游取消链。
- 超时或取消信号不代表任务已停止；操作必须实际响应，结束后释放监听器、计时器及其他资源。
- `async` 不会自动并行执行同步计算；耗时计算按目标环境选择分批处理或 Worker，异步读改写共享状态时仍需处理竞争。

参考：[no-floating-promises](https://typescript-eslint.io/rules/no-floating-promises/) · [no-misused-promises](https://typescript-eslint.io/rules/no-misused-promises/) · [ECMAScript — Promise.all](https://tc39.es/ecma262/multipage/control-abstraction-objects.html#sec-promise.all) · [Node.js — AbortController](https://nodejs.org/api/globals.html#class-abortcontroller)

## 性能与资源

- 优化以基准、性能剖析和真实负载为依据，优先检查算法复杂度、内存保留、重复计算和事件循环阻塞。
- 按访问模式选择数组、`Map`、`Set` 等结构；热路径避免重复扫描和无必要的中间集合，不为猜测性能重写清楚的代码。
- 对象展开和数组复制通常只复制一层；需要独立数据时明确复制深度，不用 JSON 往返充当通用深拷贝。
- 文件、流、连接、事件订阅、计时器及 Worker 明确归属，在成功、失败和取消路径中完成必要清理。
- 需要作用域清理时使用 `try/finally`；项目工具链、运行时及资源协议支持时可使用 `using`、`await using`，不以垃圾回收代替确定性释放。
- 缓存有明确键、容量及失效策略，避免长期对象和闭包无意保留大数据；优化不改变 API 契约和必要的资源上限。

参考：[Node.js — Don't Block the Event Loop](https://nodejs.org/learn/asynchronous-work/dont-block-the-event-loop) · [TypeScript — Explicit Resource Management](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-5-2.html#using-declarations-and-explicit-resource-management)

## 注释与文档

- 注释解释设计意图、关键约束和非显而易见的原因，不机械复述代码。
- 公共 API 使用 `/** ... */` 文档注释；根据需要说明空值、异常或拒绝、取消、副作用和资源归属。
- TypeScript 声明已经表达的类型不在注释中重复维护；使用项目支持的 TSDoc 标签描述必要语义和示例。
- 生成声明和代码通过源定义维护；修改实现时同步更新示例与文档，删除过时说明。

参考：[TSDoc](https://tsdoc.org/) · [TypeScript Documentation](https://www.typescriptlang.org/docs/)

## 测试与验证

- 每个源码文件的单元测试必须与源码同目录且只保留一个对应文件：`foo.ts` 对应 `foo.test.ts`，`foo.tsx` 对应 `foo.test.tsx`；同一源码的不同场景必须合并，不得拆为多个单元测试文件。
- 已有项目统一使用 `.spec.ts`／`.spec.tsx` 时可一致沿用，但同一源码仍只对应一个单元测试文件。
- 跨组件或外部系统协作的集成测试必须放在项目 `test/` 目录，按同类场景归组；端到端和类型测试按实际边界独立组织。
- 测试名称描述行为和预期结果，覆盖正常路径、重要边界与错误路径；修复缺陷补充回归测试。
- 异步测试等待任务及断言完成，测试结束清理订阅、计时器和 Mock；不以固定休眠代替同步。
- 测试数据和替身遵守真实契约，不用 `as any` 或大范围断言把不完整对象伪装成有效输入。
- 公共泛型及类型契约按需增加类型测试；类型测试与运行时测试分别执行，不把代码转译成功或测试通过当作完整类型检查。

参考：[Vitest — Testing Types](https://vitest.dev/guide/testing-types) · [TypeScript Compiler Options](https://www.typescriptlang.org/docs/handbook/compiler-options.html)

## 工具与检查

新项目以以下类型安全选项为默认基线；这是共享配置片段，不是完整的运行环境配置：

```json
{
  "compilerOptions": {
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true
  }
}
```

- `noUncheckedIndexedAccess`、`exactOptionalPropertyTypes`、`noImplicitOverride` 均不包含在 `strict` 开关中，需独立开启；存量项目按明确范围迁移，不以新增断言消除暴露的问题。
- `target`、`lib`、`module`、`moduleResolution`、路径别名及 JSX 配置匹配实际运行与构建环境；不为 Node、浏览器和库发布复制同一套配置。
- 默认使用 ESLint 与 typescript-eslint 的类型感知检查，以 `recommendedTypeChecked` 为起点，按工具版本配置 `projectService`；重点检查悬空 Promise、Promise 误用和不安全的 `any` 传播。
- 格式交给单一格式器，类型与语义交给编译器和 linter；配置与兼容的工具版本纳入版本管理，不并行维护相互冲突的规则。
- 必要的类型抑制限定到具体位置并说明原因，优先使用能检查抑制是否仍必要的 `@ts-expect-error`；不全文件 `@ts-nocheck` 或无依据禁用检查。

在依赖已安装、工具已配置的单个项目根目录执行；以下以 pnpm 为例：

```bash
pnpm exec prettier . --check
pnpm exec eslint . --max-warnings 0
pnpm exec tsc --noEmit -p tsconfig.json
```

- 测试与构建运行项目定义的非监听脚本；项目引用和多包仓库按实际项目分别检查，不假定根目录一次 `tsc --noEmit` 覆盖全部源码和测试。
- CI 检查而不静默改写源码；实际通过、失败和未执行分开记录，不以跳过测试或削弱断言换取通过。

参考：[TSConfig Reference](https://www.typescriptlang.org/tsconfig/) · [Choosing Compiler Options](https://www.typescriptlang.org/docs/handbook/modules/guides/choosing-compiler-options.html) · [Typed Linting](https://typescript-eslint.io/getting-started/typed-linting/) · [Prettier CLI](https://prettier.io/docs/cli) · [ban-ts-comment](https://typescript-eslint.io/rules/ban-ts-comment/)

---

## 延伸阅读

| 参考资料 | 重点内容 |
|---|---|
| [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html) | 类型、收窄、函数、类和语言机制 |
| [TSConfig Reference](https://www.typescriptlang.org/tsconfig/) | 严格检查、模块解析及编译选项 |
| [Google TypeScript Style Guide](https://google.github.io/styleguide/tsguide.html) | 命名、模块与工程写法；按适用范围采用 |
| [TypeScript Contributor Coding Guidelines](https://github.com/microsoft/TypeScript/wiki/Coding-guidelines) | 编译器仓库的内部约定，不作为整个社区的强制标准 |
| [typescript-eslint](https://typescript-eslint.io/) | 类型感知 lint、异步误用和类型安全检查 |
| [Prettier](https://prettier.io/docs/) | 格式化、项目配置和 CI 检查 |
