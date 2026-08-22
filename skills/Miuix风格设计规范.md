---
name: Miuix风格设计规范
description: 编写基于 Miuix（top.yukonga.miuix.kmp，Compose Multiplatform 的 HyperOS 风格 UI 库）的应用设计规范文档与落地指引。覆盖 MiuixTheme 主题系统（深浅色 + Monet 动态取色 + keyColor）、colorScheme 官方色值、textStyles 字阶、miuix-squircle 连续曲率圆角、miuix-blur 背景模糊、组件映射（Scaffold/TopAppBar/NavigationBar/Card/Switch/Overlay 弹窗等）。触发词：Miuix、MIUI X、HyperOS 风格、写设计规范、Compose 小米风格、squircle、连续曲率圆角、澎湃OS风格UI。
---

# Miuix 风格设计规范

## 适用范围

把"应用类型 + 页面范围 + 强调色偏好"转换为一份**基于 Miuix 组件库落地**的设计规范文档（`DESIGN-SPEC.md`）与实现指引。

Miuix 是 Compose Multiplatform 的 HyperOS 风格 UI 库：组名 `top.yukonga.miuix.kmp`，仓库 `compose-miuix-ui/miuix`，严格遵循 Xiaomi HyperOS 设计规范，提供完整主题系统（`MiuixTheme`）、官方色值与字阶、squircle 连续曲率圆角、背景模糊与几十个现成组件。

**核心准则**：产出以"用 Miuix 现成组件、主题与令牌落地"为标准，不重复发明颜色 / 组件 / 动效数值；Miuix 已覆盖的能力直接映射，超出库范围的部分才给出补充规范。

触发信号词：Miuix、MIUI X、HyperOS 风格、写设计规范、Compose 小米风格、squircle、连续曲率圆角、澎湃OS风格UI。

## 判定标准（以 Miuix 官方为准）

规范中所有数值、组件、API 引用自 Miuix 官方文档与实际实现，不编造。Miuix 为实验性库，API 可能变更，引用以当前版本文档为准。

### 主题系统（硬性要求）
- 统一使用 `MiuixTheme`，配色由 `ThemeController` 控制：`MiuixTheme(controller = controller) { }`
- 配色模式 `ColorSchemeMode`：`System` / `Light` / `Dark` / `MonetSystem` / `MonetLight` / `MonetDark`
- 深浅色跟随系统：`System`、`MonetSystem`（自动跟随系统深色模式）
- **动态取色（Monet）**：`Monet` 系列模式启用；提供 `keyColor` 则从该种子色生成配色；`keyColor = null`（默认）时使用系统壁纸（Android Material You）
- 调色板：`paletteStyle`（如 `ThemePaletteStyle.Vibrant`）、`colorSpec`（`ThemeColorSpec.Spec2025` / `Spec2021`；Spec2025 仅被 TonalSpot / Neutral / Vibrant / Expressive 支持，其余降级 Spec2021）
- 强制深浅：`isDark = true/false` 覆盖系统设置
- 品牌色：直接传自定义 `lightColors = ...` / `darkColors = ...` 给 `ThemeController`；或不用 Controller 时 `MiuixTheme(colors = ...)`
- 通过 `MiuixTheme.colorScheme.<令牌>` 访问颜色，`MiuixTheme.textStyles.<样式>` 访问文本样式

### 颜色（以 colorScheme 官方令牌为准）
- 优先使用 `MiuixTheme.colorScheme` 官方令牌与默认深浅双色值；不另起一套私有色值
- 官方默认配色恰好匹配 MIUI：`primary` 浅 `#FF3482FF` / 深 `#FF277AF7`（经典蓝），`background` 浅 `#FFFFFFFF` / 深 `#FF242424`，`surface` 浅 `#FFF7F7F7` / 深 `#FF000000`，`surfaceContainer` 浅 `#FFFFFFFF` / 深 `#FF242424`，`dividerLine` 浅 `#FFE0E0E0` / 深 `#FF393939`，`windowDimming` 浅 `#4D000000` / 深 `#99000000`
- 需品牌强调色：用 `keyColor`（动态取色种子）或自定义 `lightColors/darkColors` 覆盖，并说明修改了哪些字段

### 强调色策略
- 默认推荐 `Monet` 动态取色（跟随壁纸或 keyColor）
- 固定强调色 = 选定 `keyColor` 种子色，示例预设：
  - 经典蓝（Miuix 默认主色）`#3482FF`
  - 初音绿 `#39C5BB`
  - 紫灰 `#7E78C8`
  - 玫瑰粉 `#D4868A`
  - 警示红 `#FF453A`
- 一次只用一套定位；结合 `paletteStyle` / `colorSpec` 控制生成效果

### 圆角（连续曲率 = Miuix squircle）
- MIUI/HyperOS 的"G2 连续曲线圆角"对应 `miuix-squircle`（连续曲率，区别于 `RoundedCornerShape` 的纯四分之一圆弧）
- 用法：`Modifier.squircleBackground(color, cornerRadius)`（仅填充）、`squircleSurface(...)`（填充+裁剪）、`squircleClip(...)`、`squircleBorder(...)`；底层 `Path.addSquircleRect`
- `cornerRadius` 推荐档位（具体值写入规范）：大卡 16-20dp、列表容器 12-16dp、底栏容器 28-32dp、图片 8-16dp
- 平台：Android 基于 shader 的 modifier 需 API 33+，更低版本**自动回退** `RoundedCornerShape`，无需手动分支；`miuix-ui` 已传递包含 squircle

### 模糊（MIUI X 毛玻璃）
- 对应 `miuix-blur`（背景模糊 / 颜色混合 / 纹理效果）
- Android 需 `minSdk 33`（依赖 `RuntimeShader`）；低于 33 用能力检查门控：`isRenderEffectSupported()` / `isRuntimeShaderSupported()`
- 三步用法：创建 `LayerBackdrop` 捕获背景 → 内容容器 `Modifier.layerBackdrop()` → 模糊表面 `Modifier.textureBlur()`
- 顶栏 / 底栏 / 悬浮胶囊底栏模糊均由此实现

### 字体（textStyles 官方字阶）
- 使用 `MiuixTheme.textStyles`：`title1` 32sp、`title2` 24sp、`title3` 20sp、`title4` 18sp、`headline1` 17sp、`headline2` 16sp、`main`/`paragraph` 17sp、`body1` 16sp、`body2` 14sp、`button` 17sp、`subtitle` 14sp Bold、`footnote1` 13sp、`footnote2` 11sp
- 颜色由 `onBackground` 统一设置；需要定制用 `defaultTextStyles(...)` 覆盖后传入 `MiuixTheme(textStyles = ...)`
- 不建议自建字阶，除非明确超出库范围

### 组件（以 Miuix 现成组件为准）
- 页面组件优先用 Miuix 现成组件（清单与映射见 `references/miuix-components-map.md`），不自行实现
- `OverlayDialog` / `OverlayBottomSheet` / `OverlayDropdownMenu` 等弹窗类组件基于 `Scaffold` 提供弹窗容器，**必须被 `Scaffold` 包裹**
- 自定义组件基于 `BasicComponent` 扩展

### 动效
- 使用 Miuix 组件自带动画与 Compose 标准动效（如 `FastOutSlowInEasing` 等），**不编造官方未定义的动效曲线数值**
- 需要补充的转场/反馈动效，在规范中标注"补充规范"而非宣称 Miuix 内置

## 执行流程（六步）

1. **确认设计范围**：应用类型、页面范围、是否深浅双色（推荐都支持）、强调色方案（默认 Monet 动态取色，或选 keyColor 预设）——**需用户确认**
   - 完成标准：应用类型、页面范围、强调色方案已确定
2. **搭建主题基线**：给出 `MiuixTheme` + `ThemeController` 的具体配置（根据深浅 / 动态取色 / keyColor 选择），说明颜色如何通过 `colorScheme` 访问
   - 完成标准：主题配置代码骨架可用、深浅双色与强调色策略明确
3. **页面与组件选型**：按页面范围映射 Miuix 组件（导航 / 容器 / 输入 / 选择 / 反馈 / 弹窗 / Preference），注明容器（Scaffold）约束与配置要点
   - 完成标准：每个页面有明确组件清单与映射
4. **令牌与效果落地**：圆角（squircle 及数值）、模糊（miuix-blur 用法）、字体（textStyles 指定）、图标（miuix-icons）具体落到 DESIGN-SPEC
   - 完成标准：各视觉令牌有 Miuix 对应 API 与数值
5. **输出 DESIGN-SPEC.md**：设计原则 + 主题配置 + 组件映射 + 令牌落地 + 适配说明（minSdk / 平台差异 / 低版本回退），**完成后请用户确认再交付**
   - 完成标准：五部分齐全，全部以 Miuix 为准、数值具体
6. **交付与说明**：给出 Gradle 依赖（`miuix-ui` 及可选模块）、`MiuixTheme` 骨架、关键组件调用示例
   - 完成标准：用户已收到规范与落地指引

## 期望输出

- `DESIGN-SPEC.md`：五部分（设计原则 / 主题与颜色 / 组件映射 / 令牌落地 / 适配说明），全部以 Miuix 为准、数值具体
- 实现指引：Gradle 依赖写法、`MiuixTheme` + `ThemeController` 骨架、关键组件与 squircle/blur 调用示例
- 依赖清单（按需）：`miuix-ui`（必含 squircle）、`miuix-preference`、`miuix-icons`、`miuix-blur`（minSdk 33）、`miuix-nav`

汇报格式：规范文档路径 → 覆盖范围 → Miuix 映射摘要（主题/组件/圆角/模糊）→ 依赖与代码骨架 → 待确认项。

## 边界

- **以 Miuix 官方为准**：数值、组件、API 引用自官方文档与实际实现，不编造、不魔改既有默认值
- **不重复造轮子**：Miuix 已覆盖的能力（主题、颜色、字阶、squircle、模糊、组件）直接映射使用
- **确认门**：应用类型与强调色方案未确认不进入主题配置；`DESIGN-SPEC.md` 输出后请用户确认再交付
- **注明库状态**：Miuix 为实验性库，API 可能变更；规范标注所依据的版本文档
- **不生成完整应用代码**：产出是设计规范与实现指引
- **平台差异如实说明**：如 miuix-blur 需 minSdk 33、squircle 低版本自动回退
