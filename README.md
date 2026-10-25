# VirusTarget · 病毒靶子

> 把「病毒行为」做成一个**可控靶子 App**，用来验证「游龙（youlong-security ）」能否清除它。
> 本仓库由 `AdminTest` 的 `virus-target` 分支独立搬出，代码同源。

## 设定：文官 vs 武将 vs 病毒

| 角色 | 仓库 | 职责 |
|---|---|---|
| **文官**（检测） | `device-check` | 发现异常、定位 |
| **武将**（处置） | `youlong-security`（游龙护盾） | 清除病毒、拦截 shell、关键词扫描 |
| **病毒**（靶子） | **本仓库 `VirusTarget`** | 模拟真实病毒行为，给武将当靶子打 |

目标：验证武将（游龙）能不能识别并清除本仓库打出的这个"病毒"。

## 这个 App 会做什么（模拟病毒行为）

点「▶ 开始测试」后，按**固定序列**执行（顺序是关键）：

1. **锁机**（`lockNow`，设备管理员）→ 先让机器进入"被管"状态
2. **弹悬浮窗**（`OverlayService`，`TYPE_APPLICATION_OVERLAY` + `NOT_FOCUSABLE`）→ 隐藏核心，承载红色「☠ 自毁」逃生按钮
3. **最后才砍游龙**（`pm disable-user` 游龙双包）→ 放在最后，躲开游龙早期阶段的反制

> 顺序考量：前两步得手后（机已锁 + 悬浮窗已常驻），游龙砍不砍到都已被锁，避免"先砍游龙被它反咬"。

## 关键对抗点（游龙会检测什么）

| 游龙行为 | 本仓库对策 |
|---|---|
| 扫 shell 命令里的 `youlong` 关键词 | `ShizukuShell` 里包名**拆开存、运行时 `joinToString` 拼**，命令里不出现完整 `com.youlong.*` |
| 抓 shizuku 在跑的 shell 行为 | 砍游龙放在序列**最后**，且包名拼接 |
| dumpsys 前台检测 | 悬浮窗用 `NOT_FOCUSABLE`，不抢焦点 |

## 组件清单

| 文件 | 作用 |
|---|---|
| `MainActivity.kt` | 主界面 + 测试序列编排 |
| `ShizukuShell.kt` | Shizuku shell 封装；`disableYoulong()` 包名拆分拼接 |
| `OverlayService.kt` | 隐藏核心悬浮窗 + 自毁逃生按钮 |
| `TestDeviceAdminReceiver.kt` | 设备管理员（锁机前提）|
| `KeepAliveService.kt` | 保活前台服务（明文可见）|
| `BootReceiver.kt` | 开机自启 |
| `GuardAccessibility.kt` | 无障碍占位（待补，当前不参与编译）|

## 逃生口（重要）

测试跑起来后，屏幕角落那颗**红色「☠ 自毁」**按钮：
点一下 → 先尽量解锁 → `killProcess` 杀掉自己（**不卸载、不动系统**）。

手动恢复游龙（被砍后）：
```bash
pm enable com.youlong.tool
pm enable com.youlong.hd
```

## 构建

推到 `main` 后，Actions 自动编译并出 Release（带 APK）：
```bash
# 本地等价
gradle assembleDebug
# 产物：app/build/outputs/apk/debug/app-debug.apk
```

> 注意：旧包签名（`AdminTest`）与新仓库 debug 签名不同，**覆盖安装前需先 `pm uninstall com.demo.admintest`**。

## License

仅用于安全研究与自有设备/虚拟机测试。请勿用于未授权设备。
