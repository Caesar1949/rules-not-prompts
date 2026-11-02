---
paths:
  - "**/*.ets"
  - "**/*.ts"
---

# ArkTS / ArkUI 规则

> 适用于 HarmonyOS（ArkTS / ArkUI / DevEco Studio）项目。
> 换栈见 `docs/ADAPTING.md`。

## 严格模式与类型系统

ArkTS 不是 TypeScript——它是**带静态检查的受限子集**。TypeScript 习惯在这里会直接被 linter 拦下。

| 禁止 | 替代 |
|---|---|
| `unknown` 中转、`as unknown as T` | 用具体类型 + 类型守卫 |
| 具体类型字段用索引访问 `obj['field']` | 直接点号访问 |
| 无显式 interface/class 的对象字面量 | 先声明 interface，再构造 |
| `Record<string, Object>` 当万能容器 | 声明具体 shape |
| 枚举与数字字面量直接比较（`state === 1`） | 用枚举成员 |
| `Object \| undefined` 直接 cast | 先 guard，让类型系统收窄 |

**最常见的一类**：把路由参数、配置对象在 helper 函数里拆成 `url` / `params` 两个形参。对象字面量一旦离开显式 interface 上下文，就触发 `arkts-no-untyped-obj-literals`。**helper 收整个 options 对象，别拆。**

## 组件与属性

- `@Component` 的自定义属性名**不得撞内建通用属性**（`height` / `width` / `borderColor` / `padding` …）
- 抽组件前先查 SDK 的 `CustomComponent` 成员列表
- 撞名了就改名避让：`height` → `coverHeight`

## 事件

**`ClickEvent` 没有 `stopPropagation`，`BaseEvent` 也没有。**

嵌套的可点元素无法阻止冒泡。唯一可靠的做法是**改成同级兄弟**：

```
Stack {
    Image(...)      .onClick(pickImage)      // 兄弟，不是子元素
    Text('✕')      .onClick(removeCover)    // 兄弟，不是子元素
}
```

点击只命中栈顶那个。这是 ArkUI 与 SwiftUI 最容易踩的差异之一——SwiftUI 里 `.onTapGesture` 默认不冒泡。

## 生命周期与订阅

- **可反复打开的组件**（弹窗、Sheet）禁止在 `aboutToAppear` 里 `new` 一个订阅对象——每次打开都注册一条永不退订的回调，还持有已销毁实例
- 订阅对象必须提供 `dispose()`；匿名内联回调无法退订
- 需要换刷新目标时，用**模块级共享订阅 + 每次打开 attach 目标**，不要每次 new

## 单例与依赖注入

- DI 容器只负责**创建**，不负责**初始化**。单例 ViewModel 必须由页面显式调 `initialize()`
- 容器 `clear()` 若重建了单例，模块级常量引用与 `getInstance()` 会指向**两个不同实例**
- 「注册服务」必须排在「存储 await」**之前**，否则启动期存在服务未注册的窗口

## 废弃 API 迁移

HarmonyOS 每次大版本升级都会废弃一批 API。迁移时**逐条核对替代方案**，不要凭直觉改。

| 废弃 | 替代 |
|---|---|
| `AlertDialog.show()` | `UIContext.showAlertDialog()` |
| `AppStorage` | `LocalStorage` |
| `getContext(this)` | `this.getUIContext()` |
| 全局 `px2vp` | `this.getUIContext().px2vp()` |
| 全局 `animateTo` | `this.getUIContext().animateTo()` |
| `PixelMap.packing` | `image.createImagePacker().packToData()`（用完必须 release） |
| 模块级 `router.pushUrl` 等 | `this.getUIContext().getRouter()`，统一走项目自己的路由封装 |
| `ColorMode` 取法 | `ConfigurationConstant.ColorMode` from `@kit.AbilityKit` |

**废弃 API 清单必须写进规则文件，不要每次重新查。**

## 布局

- `Stack` 内 `height('100%')` 解析为 **Scroll 视口高度**，不是父容器内容高
- 需要撑满父容器时，用单层 `Column` + 绝对定位，而不是叠 `Stack`

## 模块导入

- 命名空间导出的模块不是 default export：`DocumentViewPicker` 要用命名空间方式引入
- 枚举成员名要核对：`OpenMode.TRUNC`（不是 `TRUNCATE`）

## 编码

- 落盘统一 UTF-8。`String.fromCharCode` 逐字节拼串是 latin1 解码，中文与 emoji 会乱码
- 用项目统一的读写封装，别在业务代码里手写文件 IO

## 测试

- host 测试的 source root 与 `src/main/` 隔离，测试必须放在 `src/ohosTest/ets/test/`
- **新增测试文件必须在测试清单里注册**，否则它从未参与编译——「测试文件存在」的假象会掩盖编译级错误

---

*来源：一次跨多个版本的 SDK 升级适配。案例见 `docs/CASEBOOK.md`。*
