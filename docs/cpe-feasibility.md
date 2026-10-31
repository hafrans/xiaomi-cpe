# 小米 Android 手机作为 CPE 的可行性调研

> 后续实测更新：本机通过 GhostLock + Shizuku 在锁定 Bootloader 的状态下取得了本次启动周期的临时 Root，已用 `su -c id` 验证。具体过程与重启后的限制见 [Root 操作记录](civi2-root-log.md)。下方免解锁可行性部分记录的是操作前的调查结论。

## 实机只读检查（2026-11-01）

已通过 USB 调试连接实际设备：型号 `2209129SC`、代号 `ziyi`，Android 15 / API 35，构建号 `OS3.0.5.0.VLLCNXM`，内核 `5.10.236-android12-9-00003-gfb24cf99ad97-ab14313284`。`ro.boot.flash.locked=1`、`ro.boot.vbmeta.device_state=locked`，verified boot 为 `green`：设备报告 Bootloader 锁定。ADB shell 是普通 shell 用户，PATH 中未发现 `su`，已安装包列表未匹配常见 Magisk 名称；不足以排除所有隐藏或非常规 Root，但不能认定已有 Root。OEM 解锁开关不代表 Bootloader 已解锁。

可读的 `/proc/config.gz` 显示 `CONFIG_NET_NS=y`、`CONFIG_VETH=y`、`CONFIG_BRIDGE=y`、`CONFIG_NETFILTER=y`、`CONFIG_NF_NAT=y`、`CONFIG_IP_NF_IPTABLES=y`、`CONFIG_IP_NF_NAT=y`，同时 `CONFIG_USER_NS`、`CONFIG_BRIDGE_NETFILTER` 和 `CONFIG_NF_TABLES` 未启用。前者只是具备部分基础，不能证明有权限或满足容器全部条件；缺少 nftables 是使用依赖 nftables 的现代 OpenWrt 防火墙的重要障碍，需另行验证可用版本与方案，不能承诺规则正常运行。

Wi-Fi 系统历史能力日志报告 2.4 GHz、5 GHz 信道及最大客户端数 32；这是系统报告的能力，并非并发或稳定性实测。蜂窝网络接口可见。尚未测试热点是否开启、SIM 数据共享、客户端上网、实际流量路径、外网入站和长期稳定性；未开启热点、Root、刷机或更改手机设置。

> 当前需求：Xiaomi Civi 2、HyperOS 3.0.5.0，作为单机 CPE，不增加外部路由器；用户接受 Root，已打开 OEM 解锁开关。ADB 显示 Bootloader 仍锁定。本文较早的通用对比保留作背景，单机专项结论见文末。

## 免解锁 Root 补充检查（2026-11-01）

已通过授权 ADB 只读确认：`ziyi`、`OS3.0.5.0.VLLCNXM`、内核 `5.10.236-android12-9-00003-gfb24cf99ad97-ab14313284`、arm64、4096 字节页；系统安全补丁标记为 2026-07-01、vendor 补丁标记为 2026-02-01，SELinux 为 Enforcing。Bootloader 报告锁定，当前 ADB 为 uid 2000。安全补丁标记不是漏洞是否存在的证明。

[GhostLock](https://github.com/YuKongA/ghostlock-app) 利用内核漏洞获取临时 Root，要求匹配内核配置；其[主项目支持表](https://github.com/YuKongA/ghostlock-app/blob/main/docs/kernel_profiles/SUPPORTED_DEVICES.md)未列出本机完整内核，且有人为同内核 Civi 2 提出[适配请求](https://github.com/YuKongA/ghostlock-app/issues/301)，该请求不是成功报告。[diting 分支](https://github.com/Haoyu-UT/ghostlock-app-diting/releases)宣称在其他型号实测通过与本机完全相同的内核字符串，但未列 Civi 2；型号、厂商内核模块和系统配置差异意味着字符串匹配不证明本机可用。该分支报告利用过程可能导致内核死锁或重启；临时 Root 重启消失，不能自动满足 CPE 断电恢复要求。

另有 [MQSAS 路线](https://github.com/xunchahaha/mi_nobl_root)针对另一款 Xiaomi 15 和 6.6 内核，不能直接用于 Civi 2。设备的 `miui.mqsas.IMQSNative` 服务存在，但仅有服务名称不证明漏洞存在或可用。没有运行任何漏洞代码、安装 Root 工具或修改手机；目前结论是存在值得研究的同内核候选，尚未证明本设备可以免解锁 Root。即使 Root 成功，缺少 `CONFIG_NF_TABLES` 以及实际热点接管仍需独立解决。

调研日期：2026-11-01。范围是让手机通过蜂窝网络上网、向其他设备共享连接，并提供状态面板。这里没有连接、检查或修改任何手机；补充机型与系统版本见文末，运营商和目标网络环境仍未知。下文所引 Android 平台能力不代表任意小米机型均已实现或向普通应用开放。

## 结论

**作为基础蜂窝网关有条件可行；作为可完全编程控制的专业 CPE 尚不能保证。** Android 的共享网络模块支持 Wi-Fi、USB、蓝牙和以太网客户端，并包含共享网络资格检查和转发组件；具体支持哪些连接方式取决于手机、系统和外设。[AOSP：Tethering](https://source.android.com/docs/core/ota/modular-system/tethering)

最稳妥的起点是使用手机系统设置提供的热点，由独立面板读取公开的、获授权的状态。若需要网线或更完整的 DHCP、端口转发、防火墙和客户端管理，可以评估手机 USB 共享网络接一台支持该模式的外部路由器，让路由器承担局域网功能；路由器、线缆、供电和手机的兼容性仍须实测。上述为架构建议，不是对特定设备功能的保证。

## OpenWrt 放在哪里

| 路线 | 能做到什么 | 主要代价 |
| --- | --- | --- |
| 原生 Android 热点 + 自定义面板 | 复用手机的蜂窝网络与热点，先验证基础 CPE 和展示效果 | 高级路由与自动控制受 ROM 和权限限制。 |
| 手机 + 外部 OpenWrt 路由器 | 手机通过 USB 共享蜂窝网络，路由器提供 OpenWrt、Wi-Fi/LAN 和策略；手机屏幕可作为面板 | 需要额外设备和供电；USB 模式、断电恢复需实测；可能存在双重 NAT，蜂窝侧还可能有运营商 NAT。[OpenWrt 手机 USB 共享指南](https://openwrt.org/docs/guide-user/network/wan/smartphone.usb.tethering) |
| 手机 Root + OpenWrt 用户空间 | 保留 Android，尝试运行 OpenWrt 用户空间并接管实际数据通路 | Root 或运行容器本身不等于 OpenWrt 接管热点：还要验证内核能力、网络命名空间、权限、接口、NAT 和系统组件冲突。硬件转发可能绕过应用处理器上的部分路径；逐设备验证规则和计数是否生效。[AOSP 热点硬件卸载](https://source.android.com/docs/core/data/tethering-offload) |
| 直接刷成 OpenWrt | 理论上需要针对具体手机移植内核、蜂窝调制解调器、Wi-Fi、屏幕和电源等 | 工程量及兼容风险最高；不知道设备型号前不能断言存在可用固件，也不能承诺电话、5G 或显示驱动可用。 |

手机的系统总流量、蜂窝流量和热点客户端流量是不同口径。Android 的接口计数可能是部分统计，硬件卸载也会改变观测路径，因此实时速率应标注来源，并用客户端传输实测核对，而非将系统总量直接标为热点速率。[TrafficStats](https://developer.android.com/reference/android/net/TrafficStats)

视觉界面可做成全屏 Android 应用或桌面，以及充电/停靠时的交互式屏保；[DreamService](https://developer.android.com/reference/android/service/dreams/DreamService) 提供屏保机制，实际触发和后台保活仍须在目标小米系统上验证。建议屏保使用低亮度、位置微移和自动息屏，减少长期显示负担。

解锁与 Root 应放在机型调查及备份之后：小米官方说明解锁条件因地区和设备而异，且解锁可能清除数据；不要把其他地区的规则直接套用于国行设备。[小米 Bootloader 官方说明](https://www.mi.com/uk/support/faq/details/KA-07238/)

## 能力和边界

| 需求 | 官方依据和判断 | 待验证 |
| --- | --- | --- |
| 蜂窝网络共享 | AOSP 共享网络支持 Wi-Fi、USB、蓝牙、以太网；运营商资格检查可能参与启用流程。[Tethering](https://source.android.com/docs/core/ota/modular-system/tethering) | SIM 套餐是否允许共享；当前 ROM 是否暴露相应选项；USB 网络模式及接收端兼容性。以太网共享不等于所有手机 USB 网卡均可使用。 |
| 热点设置与客户端 | AOSP Soft AP 可支持 SSID、安全方式、频段、客户端上限、允许/阻止列表和自动关闭等；可用性由设备决定，部分 API 仅系统应用可用。[Soft AP](https://source.android.com/docs/core/connect/wifi-softap) | 小米系统实际提供哪些设置、并发数、频段、重启恢复和空闲关闭策略。小米的 MIUI 14 文档列出无客户端时自动关闭的选项，并说明关闭后需手动重新开启；不能推断其他版本行为相同。[小米支持](https://www.mi.com/uk/support/article/KA-36314/) |
| 信号和小区 | `TelephonyManager` 提供信号和小区数据，但小区信息需要精确位置权限；Android 10 起缓存刷新受限且不保证更新。信号值可能为空或过时。[TelephonyManager](https://developer.android.com/reference/android/telephony/TelephonyManager) | 实测 SIM、双卡、网络制式和权限下能够取得哪些字段。5G NR 的 SS-RSRP、SS-RSRQ、SS-SINR 等 API 支持返回 `UNAVAILABLE`，不应显示为零或保证都有值。[CellSignalStrengthNr](https://developer.android.com/reference/android/telephony/CellSignalStrengthNr) |
| 流量 | `NetworkStatsManager` 可查历史汇总，查询整机或其他应用需使用情况访问授权；结果可包含共享网络流量，同时也包含系统和其他用途，不能直接视为某个客户端的流量。统计桶粒度约为小时，不适合细粒度实时速率。[NetworkStatsManager](https://developer.android.com/reference/android/app/usage/NetworkStatsManager) | 实际统计口径、运营商计费差异、双卡归属和是否有可用的每客户端数据。 |
| 热点开关和配置 | 有些 Soft AP 接口受系统权限保护；`TetheringManager` 的公开文档包含版本和权限限制，不能假定普通第三方应用在所有 Android 版本上可静默开关热点。[Soft AP](https://source.android.com/docs/core/connect/wifi-softap)；[TetheringManager](https://developer.android.com/reference/android/net/TetheringManager) | 对目标 API 版本、ROM 和安装身份逐项验证；不可用时跳转系统设置，由用户操作。 |
| 持续运行 | Android 对后台启动前台服务有限制；针对 Android 15 的应用，部分前台服务类型不能从开机广播启动，`dataSync` 等类型还有运行时限。[后台启动限制](https://developer.android.com/develop/background-work/services/fgs/restrictions-bg-start)；[Android 15 行为变化](https://developer.android.com/about/versions/15/behavior-changes-15)；[超时](https://developer.android.com/develop/background-work/services/fgs/timeout) | 目标 ROM 电池策略、用户授权、断电启动及实际运行时长。不能以一个通用后台服务保证 24 小时无人值守。 |

普通应用不应预设具备 root、系统签名权限、运营商特权或修改 APN、频段锁定、基站锁定、端口映射的能力。即使某设备可通过特殊权限实现，也应单独论证维护和安全成本。

## 建议方案

1. **先验证数据通路**：手机系统热点连接一个测试客户端，检查蜂窝上网、重连、速度、延迟与长时间负载；若需要有线局域网，再验证 USB 共享到指定路由器。先确定运营商允许共享及套餐流量计费方式。
2. **实现只读面板**：在手机上使用普通 Android 应用读取实际可获得的网络状态、信号和电量；使用明确单位、采集时间及“不可用/权限未授予”状态，不把缓存值当实时值。按需取得权限；若通过局域网访问面板，需设计认证、加密和访问范围。
3. **按需求增加控制**：优先使用系统设置中用户可操作的功能；对自动开启热点、终端列表、限速和端口转发分别做设备与权限验证。需要稳定可控的局域网功能时，优先评估外接路由器。
4. **做运行验收**：覆盖重启、断电恢复、SIM 或网络中断、空闲超时、后台应用被结束、温度、充电和持续负载。记录手机型号、系统版本、运营商、外设、环境和时间；未经测试不承诺无人值守可靠性。温升和电池状态出现异常时应停止测试并排查。

## 尚需提供或现场核实的信息

- 核实 Civi 2 的地区版、完整构建号、Android 版本、解锁状态，以及是否允许改变系统或使用特权。
- SIM 运营商、套餐共享限制、5G 覆盖与所需持续流量；是否需要公网入站访问（蜂窝网络可能使用运营商 NAT，应实测地址和可达性）。
- 客户端数量、Wi-Fi 或有线接口、上/下行和稳定性目标、供电与放置条件。
- 面板需要的数据和控制项：例如实时速率、每客户端流量、信号指标、短信、热点开关、DHCP 或端口转发。每项单独验证，而非根据平台文档推断均可用。

## Xiaomi Civi 2 专项补充

以下以用户提供的 Xiaomi Civi 2 和 HyperOS 3.0.5.0 为工作前提，覆盖上文调查时机型未知的表述；尚未检查手机，完整版本号、地区及 Android 版本均待设备确认。仅凭 `3.0.5.0` 无法确定固件分支、官方发布状态或可用的刷机包，不应使用其他机型的同号版本推断。小米明确指出不同机型、地区和系统版本的功能可能不同。[小米 HyperOS 3 说明](https://www.mi.com/global/hyperos?boardId=566822)

小米的产品页确认 Civi 2 使用第一代骁龙 7 移动平台、6.55 英寸 AMOLED 屏幕和 4500mAh 电池，提供 67W 有线充电；这些规格适合探索蜂窝共享加本机面板，但不能证明长期插电散热、热点稳定性或外接天线能力。[小米 Civi 2 产品页](https://www.mi.com/xiaomicivi2?masid=2875.0157) 验证应从系统自带热点开始：保持 Android 和手机本身作为蜂窝上游与 Wi-Fi 接入点，安装普通只读面板；逐项测试热点频段、可连接客户端数量、信号数据、充电负载、断线恢复及开机后行为。单机并不需要外接 OpenWrt；如果要求 DHCP、端口转发、逐客户端计量等完整路由功能，现有证据不足以承诺原生 Android 或普通应用能提供。

**解锁和 Root 是可研究选项，尚无针对这台设备当前固件的可行性证明。** 小米官方称 HyperOS 的解锁资格受审核限制，解锁存在清除数据等风险；该英国支持页面不能替代中国大陆设备的现行规则。[小米 Bootloader 说明](https://www.mi.com/uk/support/faq/details/KA-07238/) Magisk 官方安装说明要求先解锁，并根据设备实际启动结构选择匹配固件镜像；其明确警告不要刷他人修补的镜像。[Magisk 安装说明](https://github.com/topjohnwu/Magisk/blob/master/docs/install.md) 在核对完整版本号、地区、官方解锁资格、备份与恢复路径之前，不能判断本机能否解锁、使用哪个分区或保证 OTA 兼容。Root 也不自动提供内核网络能力或让 OpenWrt 接管热点。

**直接刷 OpenWrt 不作为当前可交付方案。** 查阅 [OpenWrt 支持设备说明](https://openwrt.org/supported_devices) 和 [小米设备索引](https://openwrt.org/toh/xiaomi/start) 时，未找到 Civi 2 的官方设备条目或可验证固件；这只是当前检索结果，不证明不存在第三方实验移植。完整移植还须解决蜂窝调制解调器、Wi-Fi AP、显示、充电、电源管理及固件等，不能用骁龙芯片或其他小米机型的固件推断兼容性。保留 Android 的 Root + OpenWrt 用户空间同样属于实验路线，须先证明容器与内核支持、接口接管和转发路径能工作。

下一步从“关于手机”记录**完整**系统版本字符串、Android 版本，并在开发者选项查看设备解锁状态，同时观察热点选项；然后用客户端测试共享和连续运行。以上均不要求解锁、Root 或修改系统分区，本次也没有连接或修改手机。

## Xiaomi Civi 2 / HyperOS 3.0.5.0：单机方案补充

调研日期：2026-11-01。这里以提供的机型和版本为待核实前提；没有连接或修改手机，也没有核实完整构建号、地区、Android 版本、Bootloader 状态或运营商。小米产品页称 Civi 2 使用第一代骁龙 7 移动平台；这只能确认产品平台，不能证明该 ROM 的 Wi-Fi 驱动支持第三方 AP、内核支持容器或存在可刷入的 OpenWrt 镜像。[小米 Civi 2 产品页](https://www.mi.com/xiaomicivi2)

**判断：单机原生热点是首先验证的基线；单机 OpenWrt 用户空间是可研究的实验路线，尚无该机该版本可用的证据。** 保留 Android 管理蜂窝连接和电源，客户端通过手机 Wi-Fi 接入；若高级路由是硬要求，下面的 VirtualAP + Droidspaces 方案值得逐项验证。无需外接路由器，但不能因为同为 Linux 就假设手机能运行或可靠转发。直接替换 Android 的原生 OpenWrt 需要设备专用移植与蜂窝、Wi-Fi、显示和电源驱动；本次未找到该机可用的官方固件证据，不应刷入其他机型镜像。

### 可研究的单机 OpenWrt 路径

[VirtualAP 项目](https://github.com/woyounai/VirtualAPForOldDevices)自行声明：其 managed 模式需要 root、Droidspaces 和运行中的 OpenWrt 容器。其[设计文档](https://github.com/woyounai/VirtualAPForOldDevices/blob/main/Documentation/How-It-Works.md)描述，`hostapd` 使用手机 Wi-Fi 的虚拟 AP 接口；该接口经网桥和 veth 连接容器，由 OpenWrt 提供 DHCP、DNS、NAT 和防火墙。Android 内核和蜂窝上行仍是基础；这不是刷机成独立 OpenWrt。项目宣称 LuCI、规则和限速等功能，但这些是项目能力描述，不是 Civi 2 实测结论。[Droidspaces](https://github.com/ravindu644/Droidspaces-OSS) 明确需要 root 和足够的内核配置，并称多数原厂内核缺少所需选项；特权容器中的不可信 root 进程也可能触及宿主机。

关键未知包括：驱动是否允许创建及桥接 AP 接口、hostapd 能否接管无线设备、网络命名空间/veth/bridge/netfilter 和容器所需选项是否可用、Android 的路由及 SELinux 是否冲突、蜂窝重连后上行是否恢复。虚拟接口名称和实现不能照搬示例。项目当前说明默认空闲 10 分钟关闭热点、默认客户端上限为 10，实际驱动可能更低；具体版本的默认值和行为应复核。[VirtualAP README](https://github.com/woyounai/VirtualAPForOldDevices)

原生 Android 热点与此第三方路径的数据通路不同。AOSP 说明系统共享可使用硬件卸载，也可使用 eBPF 加速；因此在任一具体路径上，必须以实际数据包和规则计数证明防火墙、限速及统计确实覆盖预期流量，不可仅凭规则加载成功断言生效。[AOSP 硬件卸载](https://source.android.com/docs/core/data/tethering-offload)；[AOSP Tethering](https://source.android.com/docs/core/ota/modular-system/tethering)

### Root 与验证门槛

- 先记录完整构建号、地区、Android 版本、当前解锁状态、备份和可恢复的匹配官方固件。小米的 HyperOS 解锁条件按设备、地区和账号审核，国际版说明不等于本机有资格；解锁可能清除数据并影响安全、更新与保修，须先核实当地现行政策。[小米 Bootloader 说明](https://www.mi.com/global/support/faq/details/KA-07238/)
- Magisk 官方安装文档要求已解锁 Bootloader，并根据实际设备选择含 ramdisk 的映像；不能从 HyperOS 版本推断应操作哪个分区。其文档警告不要刷别人提供的已修补映像。OTA 的官方说明仅在 Pixel 上验证，不能保证本机升级后保留 root。[Magisk 安装](https://topjohnwu.github.io/Magisk/install)；[Magisk OTA](https://topjohnwu.github.io/Magisk/ota.html)
- 基线测试：系统热点连接两个客户端，记录 DHCP、DNS、IPv4/IPv6、吞吐、延迟、断网重连、空闲及重启后行为；对照目标客户端数和运营商共享政策。对公网入站需从外部网络实际测试，不把本机端口转发视为穿透运营商 NAT 的保证。
- 若已具备授权和可恢复的 root 环境，再检查容器内核能力与 AP 驱动，逐步验证启动、客户端租约、双向流量、隔离和限速。用客户端生成流量并比对计数；测试 DNS、IPv6、VPN 断连与切换时是否绕过规则，验证 LuCI 仅在预期 LAN 可访问且已设置认证。
- 连续负载和无人值守测试覆盖温度、电池和供电、长时间上下行、断电及重启、蜂窝中断和应用/容器退出。记录版本、环境、指标与失败时间；异常发热或电池异常时停止。上述均为验收条件，未经实机测试不能承诺 24 小时可靠性。
