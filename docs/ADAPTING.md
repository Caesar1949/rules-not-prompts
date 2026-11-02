# 换技术栈怎么改

> 本模板的骨架与语言无关；`.claude/rules/` 下的四条示例是 Swift/iOS 专用。
> 这份文件说明哪些能留、哪些必须重写。

---

## 一、直接保留（与技术栈无关）

| 文件 / 章节 | 原因 |
|---|---|
| `AGENTS.md` 第四、五节（R / K 规则） | 禁止项与执行方式不依赖语言 |
| `AGENTS.md` 第七节（分层约定） | View / ViewModel / Service / Model 的分层是通用模式 |
| `AGENTS.md` 第十二节（流程与权限） | 提交、踩坑、权限是人的纪律 |
| `CLAUDE.md`（导入指针） | 只有一行 |
| `docs/RULES.md` | 编号体系与生命周期 |

---

## 二、必须重写

| 内容 | 换成 |
|---|---|
| 构建 / 测试命令 | 你项目的真实命令，一条能直接复制的 |
| 目录结构 | 真实目录，不要写计划中的目录 |
| `.claude/rules/swift.md` | 换成 `<语言>.md`（TypeScript / Python / Kotlin / Rust…） |
| `.claude/rules/frontend.md` | 换成你的 UI 层规则（Web / Compose / SwiftUI…） |
| `.claude/rules/arkts.md` | 只有做 HarmonyOS 才需要；不用可删 |
| `.claude/rules/i18n.md` | **任何多语言项目都保留**，它与语言无关 |
| `.claude/rules/dependencies.md` | 换成你的工程文件路径（`package.json` / `pyproject.toml` / `build.gradle`） |

> **`arkts.md` 是可选的。** 它只在 HarmonyOS 项目里有意义——Swift / Web 项目直接删掉这个文件即可，不要留着当参考。
>
> **`i18n.md` 是通用的。** 它讲的是「文案与语言变体」这件事，与具体技术栈无关，任何要出多语言版本的项目都该保留。 |

---

## 三、按技术栈替换的规则示例

### 3.1 Web 前端（TypeScript / React）

`rules/typescript.md`：

- 禁止 `any` 与 `as` 强转（用类型守卫）
- 禁止在渲染函数里做副作用
- 列表渲染必须有稳定 `key`，禁止用索引
- 状态更新走不可变写法

`rules/frontend.md`：

- 路由状态只读路由库公开 API，不解析内部对象
- 全局状态只在应用入口创建一次
- 禁止用 `key` 强制重建整棵组件树做主题切换
- 样式走设计令牌，禁止内联硬编码色值

`rules/dependencies.md`（paths: `package.json` / `package-lock.json`）：

- 锁文件不自动升级
- 新增依赖前先确认能否用平台能力替代
- 不手改 lockfile 内部结构

### 3.2 Python

`rules/python.md`：

- 禁止裸 `except:`，按异常类型捕获
- 禁止可变默认参数（`def f(x=[])`）
- 类型标注必须覆盖公共函数
- 同步 IO 不要混进 async 路径

### 3.3 HarmonyOS / ArkTS

`.claude/rules/arkts.md` 已经写好，直接可用。核心要点：

- ArkTS 是 TypeScript 的**受限子集**，不是 TypeScript——`unknown` 中转、索引访问、无类型对象字面量都会被 linter 拦下
- `ClickEvent` 没有 `stopPropagation`，嵌套可点元素必须改成 `Stack` 里的同级兄弟
- 可反复打开的组件禁止在 `aboutToAppear` 里 `new` 订阅对象
- DI 容器只创建不初始化，单例的 `initialize()` 必须由页面显式调
- 废弃 API 清单写在规则文件里，不要每次重新查

### 3.4 通用工程文件

`rules/dependencies.md`（paths 换成你的构建文件）：

- 依赖清单人工维护
- 构建文件可读不可手改内部字段
- 改完必须校验 + 完整构建一次

---

## 四、改写的三条原则

**1. 规则要能验证。**
写"代码要清晰"没用。写"禁止 `any`，用类型守卫"能被 review 抓到。

**2. 每条规则附一次事故。**
从你的 `ERRORS.md` 里挑。没有事故支撑的规则，先别写。

**3. 先写三条，跑一周再扩。**
规则越多，Agent 越容易把"没提到"当成"允许"。

---

## 五、迁移检查清单

- [ ] 所有 ⟨占位符⟩ 已替换（`grep -rn "⟨" .`）
- [ ] `paths` 每一项都匹配真实文件（否则规则永不激活）
- [ ] 构建命令能直接复制运行
- [ ] 测试命令能直接复制运行
- [ ] `CLAUDE.md` 仍只有导入指针，没有内容
- [ ] 没有 `~~已停用~~` 这类失效条目残留
- [ ] 跑一次真实任务，确认规则被读到

---

*最后一项最容易漏：规则写完不代表生效，要实际跑一次改动验证。*
