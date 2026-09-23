# Utopia-pvp UI demo

当前版本：`utopia-pvp-demo-gits-v2.png`，使用项目更新后的 `utopia-mr-ui` skill 与内置 image_gen 制作。完整提示词见 `prompt-gits-v2.txt`；上一张黑黄图 `utopia-pvp-demo.png` 与 `prompt.txt` 保留为历史版本，不再代表当前主题。

## 依据与布局

- 目标仓库：https://github.com/innovation-shenzhen/Utopia-pvp
- 已核对版本：`d8fa65e33486bd66a057d84390d07d19abc94156`。
- 当时源码为红蓝方块、双人局域网、单摇杆移动、竖屏；README 的纸箱怪物设计没有作为本次依据。
- 四屏：准备、附近房间与 PIN、扫描对齐、游戏 HUD。选房与 PIN 合并在一屏为布局建议。
- 保留真实红蓝身份色，不新增攻击、血量、技能、积分或横屏规则。

## 当前视觉规范

以《攻壳机动队》1995 剧场版冷峻气质为默认设计起点，使用冷黑 #080F12、青灰面板 #101D23、电子绿 #83C7A1、青色信息线 #6DAAB1 与冷白 #DCE7E3。中文黑体、等宽数字、精密机械细线与功能性信息分层。色值为项目提炼，非官方色表；示意实景保持自然色。旧黑黄海报仅辅助构图。

## 状态与接入边界

准备动作以真实追踪有效状态启用；开始须房主且双方准备。未发现房间保留手输 IP；PIN 错误有说明；相机拒绝提供设置/返回；追踪失效清空摇杆输入并提示重新扫描。退出显示真实断连影响，不将确认框视为全局暂停。

后续对应现有 CreateRoom、OpenJoin、Ready、StartRound、Rescan、JoystickControl 与 Leave 语义，本次没有接入或修改目标工程。正式字体、图标、相机与真实 marker 仍需工程资源。

## 验证

新版生成图已目视检查：四屏布局、主要中文、红蓝身份、单摇杆与冷色科技方向保留。它是静态概念稿，房间码、计时、实景与识别图均为示意，图内 marker 不能用于真实 AR 定位。Unity Editor、Android 真机与 iPhone 真机均未验证。没有提交或推送。
