# Unity 接入与验收

本文件是可复用的实现约束，不是已编译的 Unity 组件包。官方链接于 2026-09-22 核对；包文档举例不等于要求安装该版本。

## 开始编码前

读取 `ProjectSettings/ProjectVersion.txt`、`Packages/manifest.json`、相关场景/Prefab、输入与本地化配置，记录实际 `6000.x`、AR/定位 SDK、UI 技术和渲染管线。沿用项目已有 uGUI 或 UI Toolkit。没有工程时保留这些为未确认，不凭目录名称推测。

全新工程可优先评估 uGUI + TextMesh Pro 作为该触摸战斗 HUD 的起点；这是实现建议，不是用户已选型，也不要求迁移已有 UI Toolkit。不能笼统声称“Unity 6 的 UI Toolkit 无法做世界空间 UI”；能力会随小版本变化，需要查对应版本并验证。[Unity UI 系统比较](https://docs.unity3d.com/6000.0/Documentation/Manual/UI-system-compare.html) / [6000.6 世界空间 UI](https://docs.unity3d.com/6000.6/Documentation/Manual/ui-systems/create-world-space-ui.html)

## 建议职责划分

- 视觉主题：将 JSON 数值映射到项目的 ScriptableObject/主题对象或 USS 变量；此 JSON 不可声称直接导入 Unity 即生效。
- 页面与 HUD：只呈现状态、发送操作意图。
- 定位适配器：将所用 SDK 的权限、追踪、匹配、失效与绑定事件转换为 UI 可读状态。不得通过动画或固定定时器造成功。
- 对局适配器：读取真实生命值、回合、技能可用性与比赛结果，接收 UI 动作请求；不在 UI 中新增伤害和连招规则。
- 操作模块：拥有各自布局与输入绑定，共享 HUD 可独立工作和验收。

角色和决斗场是世界内容；生命条、提示、菜单和常用按钮是屏幕 UI。相机视野中的世界标签需处理离屏、相机背后、遮挡和尺寸，不能将所有标记永远夹在屏幕边缘。Apple 对手持 AR 也建议把文本与可操作 UI 置于屏幕空间以利阅读。[Apple 手持 AR 设计](https://developer.apple.com/videos/play/wwdc2022/10131/)

## 朝向、坐标和触摸

准备与战斗使用分别编排的纵横布局，共享数据与场景对象。可采用不同页面根节点/Prefab/模板，不把竖屏界面整体横向拉伸。允许横屏左/右时分别检查刘海与手势区；是否锁某一侧由产品决定。

关键控件在 `Screen.safeArea` 内，背景可铺满。窗口尺寸和方向改变时更新；多窗口/渲染目标情形以实际项目坐标换算为准。`Screen.safeArea` 是 Player 窗口像素，原点在左下，不是 UI Toolkit 的左上坐标或 Canvas 本地坐标。[Screen.safeArea](https://docs.unity3d.com/6000.5/Documentation/ScriptReference/Screen-safeArea.html)

uGUI 的全屏根 Canvas 若使用 Screen Space 模式且根区域与整个 Player 窗口一致，可按窗口宽高将安全矩形归一化设置子节点 anchors，并检查 offset；相机局部 viewport、RenderTexture 和非全屏根不应直接套该算法。UI Toolkit 按面板缩放、坐标原点及实际版本映射安全区，避免重复应用。

Canvas Scaler 可按设计参考分辨率缩放，但 1080×1920 和 1920×1080 是两套构图基准，不要求在旋转帧内硬切参考分辨率。选择适合现有项目的缩放/布局策略，等待尺寸更新完成，避免旋转时短暂出现巨型按钮、错位射线或重复输入。[Canvas Scaler](https://docs.unity3d.com/Packages/com.unity.ugui@2.6/manual/script-CanvasScaler.html)

iOS 至少 44×44 pt、Android 至少 48×48 dp 的触摸区域作为设计下限参考；这不是 44/48 个 Canvas 单位。命中区可以大于图标，且不可与邻近动作重叠。按 UI 缩放、平台屏幕度量与真机检查校准；`Screen.dpi` 可能不可用或不准，不单独作为物理尺度保证。[Apple 布局提示](https://developer.apple.com/design/tips/) / [Android 无障碍触摸区域](https://developer.android.com/guide/topics/ui/accessibility/apps) / [Android 密度单位](https://developer.android.com/training/multiscreen/screendensities)

装饰图像关闭 raycast 命中，避免拦截按钮。若实现直接操作模块，按 pointer/touch ID 处理同时按压和取消，旋转/禁用/失焦/定位丢失时清空持续输入。设置页和暂停页也要在真机触摸可用，不能依赖 hover 或桌面键盘。

## AR 与重定位

AR Foundation 只是可能的接入路径。若使用它，`SessionTracking` 代表设备位姿追踪成功，不代表外部重定位服务已经匹配场景；项目自己的场景绑定状态必须单独保存。若使用其他定位栈，映射其真实事件，不强装 AR Foundation。[AR Session 状态](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.4/manual/features/session/arsession.html)

不要将“识别平面”或“点一下放置”擅自替换用户的已登记场景重定位流程。定位算法参数、匹配阈值和稳定时间由实际 SDK/项目负责；UI 只消费结果和可解释原因。

相机拒绝、设备不支持、AR 服务需要更新要有明确恢复路径；iOS 已拒绝权限后，重复启动会话不等于再次弹系统授权。ARCore 项目按实际 Required/Optional 策略检查支持与服务安装状态。[Apple 相机未授权错误](https://developer.apple.com/documentation/arkit/arerror/code/cameraunauthorized) / [ARCore 支持检查](https://developers.google.com/ar/develop/unity-arf/enable-arcore)

追踪不可用时如有真实原因则给对应提示；没有原因不要编造“光线太暗”。后台恢复需重新校验定位结果是否仍有效。[NotTrackingReason](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.4/api/UnityEngine.XR.ARSubsystems.NotTrackingReason.html) / [Apple AR 体验指导](https://developer.apple.com/design/human-interface-guidelines/augmented-reality)

深度、环境遮挡、光照估计按实际 provider 和设备探测后启用。提供无深度时可读的场地边界/接触阴影等表现候选，不承诺其等同真实遮挡。[AR Foundation 平台支持](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.4/manual/index.html) / [ARCore 深度能力](https://developers.google.com/ar/develop/unity-arf/depth/developer-guide)

## 资源与性能

可复用资产优先为少量面板、切角/角框、警示纹理、图标及中文字体。使用 9-slice 时检查切角是否被拉伸；使用程序绘制时检查多分辨率边线。不要为每个控件导出一整张大背景。随包海报只作设计参考，不是待导入客户端的 UI 贴图。

字体需随 Unity 资产流程打包，配置中文字符和 fallback；测试动态角色名、标点、省略、长提示和缺字，不只验证示例中的几个汉字。数字不要因比例字宽抖动。图集大小和动态字形成本按目标设备测量，不默认预烘全部 CJK。

将频繁更新的 HUD 与静态装饰按框架实际性能特征隔离；避免每帧重建全部布局或更新未变化文字。控制透明覆盖、大面积模糊、遮罩和持续装饰动画，用实际 Profiler/Frame Debugger 数据决定优化，不承诺未测的 FPS。[Unity UI 性能指导](https://docs.unity3d.com/Manual/best-practice-guides/ui-toolkit-for-advanced-unity-developers/optimizing-performance.html)

## 验收记录

只检查当前改动相关内容；下面是交付完整 MR UI 流程时的验收范围。不要为纯文案调整启动全套真机回归。

| 层级 | 能证明什么 | 至少记录 |
|---|---|---|
| 静态/浏览器设计预览 | 视觉方向与部分布局 | 文件、页面/尺寸、实际检查过的状态；不能证明 Unity 渲染或定位 |
| Unity Editor / 模拟 | 工程编译、绑定、布局、部分输入与状态流 | Unity/包版本、场景、测试动作、结果；模拟定位单独标注 |
| Android 真机 | 该设备实际触摸、相机、定位及运行表现 | 机型/系统、构建、方向、光照、操作路径与性能记录 |
| iPhone 真机 | 该设备实际触摸、安全区域、权限和恢复 | 同上；不能用 Android 结果替代 |

完整路径：首次进入 → 相机授权 → 定位 → 场地绑定 → 横置 → 对战 → 位置失效/恢复 → 结算 → 返回准备。再验证权限拒绝、定位失败、场地绑定失败、失焦恢复和旋转期间的输入取消。操作未定时只验收共享 HUD 与接口，完整可玩对战仍待定。

明/暗/复杂相机背景、16:9 与长屏、左右刘海、安全区域、中文长文案和多人触控（若选直接操作）应按实际支持范围检查。帧率、运行时长、温升和最低机型由项目约定后实测，未约定/未运行就标记待确认/未验证。XR Simulation 不能替代设备测试。[Unity XR Simulation 限制](https://docs.unity3d.com/Packages/com.unity.xr.arfoundation@6.3/manual/xr-simulation/simulation-overview.html)
