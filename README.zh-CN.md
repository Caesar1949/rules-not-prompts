# rules-not-prompts

**我不会写 Swift。一年前我用 AI 交付了一个 62,299 行的 iOS App。**

不是靠学会了 Swift，是靠给 AI 写了一套规则——每次改动之前，它必须先读完。

```
6 条红线        —— 专门拦那些「会静默失败」的错误：编译正常，测试全绿，用户手机上出事
13 条执行约束    —— 一次改动怎么做、怎么验证、怎么提交
按路径激活       —— 专项规则只在你改到匹配文件时才加载
案例集          —— 每条规则背后那次真实的事故
```

App 不会开源。这套规则会。

**English → [README.md](README.md)**

---

## 为什么是规则，不是提示词

所有用 AI 写代码的人都会撞上同一堵墙：你写了一大段指示，它还是做错了。

提示词在请求，规则在约束。

差别在于**可验证性**。一条好的规则必须能被检验：

- 能跑的命令（构建、测试）
- 能看见的产物（提交、diff）
- 能指认的失败（编译错误、崩溃日志）

三条都做不到的规则，只是 Agent 读完就忘的一段散文。

这个仓库里的每条规则，都附带了产生它的那次事故。这才是重点——[看案例集](docs/CASEBOOK.md)，你就知道一条坏规则要付多少代价。

---

## 为什么只有一份规则文件

这是整个仓库最重要的设计决定。

`CLAUDE.md` 和 `AGENTS.md` 都会被 Agent 自动加载。如果两边各存一份规则：

1. **同一套规则加载两遍**——白白浪费上下文预算
2. **两份副本必然漂移**——改了一处忘了另一处，Agent 读到哪一份纯属运气
3. **漂移之后，没人知道哪份是对的**

所以：`AGENTS.md` 是唯一真值。`CLAUDE.md` 只有一行：

```markdown
@AGENTS.md
```

需要给不同工具喂不同文件时，用导入或软链，**不要复制内容**。

---

## 为什么规则按路径加载

`.claude/rules/` 下的文件通过 frontmatter 的 `paths` 激活：

```yaml
---
paths:
  - "**/*.swift"
---
```

改 Swift 文件时才加载 Swift 规则，改工程文件时才加载依赖规则，其余一律不进上下文。

常驻规则会互相稀释。Agent 读 300 行规则，真正生效的往往只有最前面几条。

**一个值得记住的坑**：`paths` 的每一项都是文件 glob，必须匹配至少一个真实存在的文件。写成符号名或已删除的文件名，规则会**永不激活，而且不报错**。

---

## 安装

```bash
git clone https://github.com/<你>/rules-not-prompts
cd rules-not-prompts

# 复制进你的项目
cp -r .claude  /path/to/your-project/
cp AGENTS.md  /path/to/your-project/
cp CLAUDE.md  /path/to/your-project/

# 找出所有占位符
grep -rn "⟨" /path/to/your-project/
```

然后把 ⟨尖括号⟩ 里的占位符换成你自己的路径、构建命令和测试命令。

**先写三条规则，不是十三条。** 跑一周。规则越多，Agent 越容易把「没提到」当成「允许」。

---

## 目录

```
rules-not-prompts/
├── README.md                    ← 英文主文档
├── README.zh-CN.md              ← 你在这里
├── AGENTS.md                    ★ 规则唯一真值（规则 + 项目手册）
├── CLAUDE.md                    一行导入指针
├── .claude/
│   └── rules/
│       ├── swift.md             按 **/*.swift 激活
│       ├── frontend.md          按你的 UI 源码目录激活
│       ├── security.md          按源码 + 测试目录激活
│       └── dependencies.md      按你的工程文件激活
├── docs/
│   ├── RULES.md                 编号体系、来源分类、规则生命周期
│   ├── ADAPTING.md              换到 TypeScript / Python / Web 怎么改
│   └── CASEBOOK.md              8 个真实事故
└── promo/
    └── content-calendar-30d.md  30 天内容日历
```

---

## 六条红线

每一条针对的都是**会静默失败**的错误：编译通过、测试通过、用户在手机上出事。

| # | 禁止 | 为什么 |
|---|---|---|
| R001 | 遍历框架的私有导航结构 | 私有类型无稳定 API，升级即崩 |
| R002 | 在展示层做业务决策 | 逻辑散落，测试无法覆盖 |
| R003 | 混用多个包管理器 | 同一依赖两份解析结果 |
| R004 | 手改工程文件的内部字段 | 工具链会静默丢弃你的改动 |
| R005 | 引入未经批准的第三方 SDK | 供应链与合规风险不可控 |
| R006 | 在真实服务里嵌调试逻辑 | 调试代码随发布一起上线 |

完整编号与生命周期见 [docs/RULES.md](docs/RULES.md)；每条规则背后的事故见 [docs/CASEBOOK.md](docs/CASEBOOK.md)。

---

## 跨技术栈

骨架与语言无关；`rules/` 下四条是 Swift/iOS 示例，换栈时按 [docs/ADAPTING.md](docs/ADAPTING.md) 改写——里面有 TypeScript/React、Python、通用工程文件的现成改法。

| 工具 | 读取 |
|---|---|
| Claude Code | `CLAUDE.md`（导入 `AGENTS.md`）+ `.claude/rules/` |
| Codex / Cursor / Grok Build 等 | `AGENTS.md` |

---

## 许可

MIT。拿走、改掉、用起来都行，不需要署名。

唯一请求：如果你用它拦住了一次真实事故，回来开个 issue 告诉我。那比 star 有用。

---

*抽取自一个真实的单人 iOS 项目，已脱敏。抽取时刻：1,111 次提交 / 333 个 Swift 文件 / 62,299 行 / 96 个测试文件 / 0 个第三方依赖。*
