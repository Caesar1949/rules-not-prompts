---
paths:
  - "**/*.swift"
---

# Swift 通用规则

> 本文件通过 frontmatter 的 `paths` 自动激活：只要工作路径匹配 `**/*.swift` 就加载。
> 换语言时按 `docs/ADAPTING.md` 改写，或替换为等价的语言规则文件。

## 写前先读

- 修改任何文件前，先完整读取该文件，理解上下文
- 不要在未读取原文件的情况下直接修改

## 精准修改

- 只改目标代码，不动正交区域
- 改完立即编译验证（K9）

## 简洁性优先

- 优先复用现有模式，不引入新文件 / 新类型
- 避免过度工程化

## 错误处理

- 禁止强制解包 `!` 与强制类型转换 `as!`
- 用 `guard let` / `if let` / `as?`
- 错误要按来源分级：网络失败、服务端拒绝、数据缺失是三种不同的事，文案不能统一归因

## ForEach

- 数据模型必须遵循 `Identifiable`
- 禁止 `ForEach(..., id: .self)`

## 并发隔离

- UI 相关类型标注 `@MainActor`
- **会被系统在后台线程回调的注册型 API**（后台任务调度、CloudKit 订阅、通知中心 delegate 等），承载它的类型必须 nonisolated
- 隔离闭包传给非隔离参数时，编译器会插入**运行时隔离检查**，编译期完全无感，只在真机路径触发崩溃

## 视图辅助属性

- 需要局部变量时不要用 `@ViewBuilder`：用 `some View { let x = ...; return ... }`
- 只有纯视图拼接（无 `let` / `var`）时才用 `@ViewBuilder`

```swift
// ❌ result builder 不允许内嵌 let
@ViewBuilder
private var banner: some View {
    let shadow = DesignTokens.Shadows.card
    return HStack { ... }.shadow(color: shadow.color)
}

// ✅ 去掉 @ViewBuilder
private var banner: some View {
    let shadow = DesignTokens.Shadows.card
    return HStack { ... }.shadow(color: shadow.color)
}
```

- 阴影 / 颜色等常量放 `static let`，不要每次渲染重算

## 兼容性

- 版本差异用 `if #available` / `#if os()` 就地分发
- 不要建推测性的"版本适配层"目录——零调用的适配层比没有更糟

---

*来源：真实项目中被反复违反后固定下来的规则。案例见 `docs/CASEBOOK.md`。*
