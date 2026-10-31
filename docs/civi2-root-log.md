# 小米 Civi 2 免解锁 Root 操作记录

日期：2026-11-01（中国标准时间）。目标：不解锁 Bootloader，尝试临时 Root，并安装配套 Root 管理器。Root 不代表 OpenWrt 路由能力已验证。

## 设备与初始状态

- 设备：Xiaomi Civi 2，型号 2209129SC，代号 `ziyi`，芯片 SM7450（系统报告）。
- 系统：HyperOS `OS3.0.5.0.VLLCNXM`，Android 15，安全补丁标记 2026-07-01，vendor 补丁标记 2026-02-01。
- 内核：`5.10.236-android12-9-00003-gfb24cf99ad97-ab14313284`，arm64，页大小 4096。
- 初始状态：`ro.boot.flash.locked=1`，`ro.boot.vbmeta.device_state=locked`，verified boot `green`，SELinux Enforcing；ADB shell uid 2000。未在 PATH 中发现 su。
- 用户已启用 USB 调试并授权本电脑，也开启 USB 安装。OEM 解锁开关已打开，但 Bootloader 实际仍锁定。

## 工具和来源

1. [GhostLock diting 5.10 r7](https://github.com/Haoyu-UT/robust-ghostlock-app/releases/tag/diting-5.10-r7)：`GhostLock-diting-5.10-r7-arm64-debug.apk`，SHA-256 `d338d0ccd896a1f1ded098a48d67f0ab5820090a96d49f49c4b3534aeb58c257`。SHA-256 与 GitHub 发布元数据相符；一致性不代表已审计软件或保证安全。该分支在其他机型上测试过相同内核，Civi 2 没有预先证实的成功案例。
2. [BakaSU / ReSukiSU v4.2.0-rc3](https://github.com/Baka-SU/BakaSU/releases/tag/v4.2.0-rc3)：`ReSukiSU_v4.2.0-rc3_35171-arm64-v8a-release.apk`，SHA-256 `25657bc449439687608fffa04b4b586de90fc405e3dc6217bd997fc71ba0a0a1`。这是 KernelSU 衍生 Root 管理器，为 GhostLock 分支说明要求的配套版本；它是预发布版本。
3. [Android SDK Platform Tools](https://developer.android.com/tools/releases/platform-tools)：临时下载 Google 官方 macOS 包，ADB 37.0.1，路径 `/tmp/xiaomicpe-tools/platform-tools/adb`，临时路径可能在重启或清理后失效。
4. [Shizuku v13.6.0](https://github.com/RikkaApps/Shizuku/releases/tag/v13.6.0)：`shizuku-v13.6.0.r1086.2650830c-release.apk`，本地 SHA-256 `6e273ab0e991c4e79bc8b1bbb9b9dd739ccac1a8712a541a214078886b7b790f`；发布元数据未提供可独立对比的 digest。包名 `moe.shizuku.privileged.api`。

## 实际操作

1. 通过 USB 和授权 ADB 读取设备状态；未解锁 BL、未刷写分区。
2. 从上述发布页下载两个 APK，分别核对 SHA-256。经 ADB 安装 BakaSU 和 GhostLock，安装均返回 `Success`；应用包名分别为 `com.resukisu.resukisu` 和 `com.ghostlock.app`。
3. 根据 GhostLock r7 发布说明，首次安装后重启手机一次；用户解锁手机。ADB 再次连接，Android 启动完成。
4. 打开 GhostLock：初始加载时短暂显示未匹配；加载完成后显示“受支持 / 配置已就绪”，识别 Civi 2 和上述内核，默认 CPU 组为 4、5；未开启 Shizuku 执行。
5. 点击“执行”进行一次临时 Root 尝试。日志先显示 W1 阶段及一次 `route_done ... success=1`，随后进入 W3 阶段，出现 `PI route did not produce a verified write` 并执行内部重试；这些是中间状态，不是 Root 成功的证据。
6. 执行期间 ADB 连接关闭、设备短暂离线，随后重连。重连时 `/proc/uptime` 约 43 秒，说明设备发生过重启；无法仅凭现有日志断定重启的底层原因。Android 报告启动完成，Bootloader 仍锁定，SELinux Enforcing，ADB shell 仍为 uid 2000、PATH 中无 su。首次尝试未建立可验证的 Root。
7. 重启后 GhostLock 提示上次失败发生在 W3 seccomp bypass 阶段，建议通过 Shizuku 的 shell 权限跳过该阶段。安装并打开 Shizuku，用应用“查看指令”提供的本机 ADB 命令启动服务，在 GhostLock 提示中启用 Shizuku 并允许其访问；应用显示“Shizuku 已就绪”。按提示启动第二次临时 Root 尝试。Shizuku 的 shell 权限本身不是 Root，重启后需要重新启动服务。
8. 第二次尝试后 ReSukiSU 自动打开并显示“工作中 / LKM / 越狱模式”，版本 `v4.2.0-rc3-239e1e88@ReSukiSU (35171/4)`，当时显示 0 个超级用户、0 个模块。随后从 ADB 执行 `su -c 'id; getenforce; getprop ro.boot.flash.locked'`，返回 `uid=0(root) gid=0(root) ... context=u:r:ksu:s0`、`Enforcing`、`1`；单独执行 `su -c 'id -u'` 返回 `0`。`/proc/modules` 可见 `kernelsu` 模块；`ro.boot.vbmeta.device_state=locked`、`ro.boot.verifiedbootstate=green`。这些检查证明当前启动周期取得 Root，并非永久或重启后自动恢复的证明。

## 当前状态

三款应用已安装，第二次尝试在本次启动周期取得并验证 Root。Bootloader 仍锁定且 SELinux 当前为 Enforcing。未安装额外 Root 模块、未刷写启动分区；未测试设备重启后的 Root 恢复。ADB shell 调用 `su` 已成功，应妥善保护已授权的调试电脑与手机上的授权配置。

## 边界与恢复

该方法是运行时漏洞利用；即使成功，通常重启后 Root 消失，可能出现重启或死锁。项目的匹配配置基于其他机型，相同内核字符串不等于硬件全兼容。未授权刷机或 BL 解锁，也不应在锁定 BL 时刷入未验证的 boot 镜像。若系统异常，先正常重启，检查 Root、SELinux 和网络状态；无法启动时不要随意刷机或恢复出厂，应先确认备份与本机可用恢复方案。安装应用会留在手机上，重启不会自动卸载。
