# RTLPlayground 本地机型说明（tarly 的 4 台设备）

| 语言 | 文档 |
|---|---|
| **中文** | [MY-DEVICES.md](MY-DEVICES.md)（本页） |
| English | [MY-DEVICES.en.md](MY-DEVICES.en.md) |

---

> **AI 辅助声明**
> 本文档以及其中描述的板级适配工作，由 **AI 编码助手（DeepSeek Harness）**协助完成。
> 所有改动均由作者在**真实设备**上复核、编译并实测；固件在写入前经过校验，
> 结论均来自实机现象而非推测，未验证的部分在文中明确标注。
>
> **AI assistance declaration**
> This document and the board support it describes were prepared with the help of an
> **AI coding assistant (DeepSeek Harness)**. Every change was reviewed, built and tested
> on the **actual hardware** by the author; images were verified before writing and the
> results below come from measurements, not guesses. Anything not verified is marked as such.

---

这份文档记录我在 `tarly/RTLPlayground` 上实际适配过的设备、实测结果和踩过的坑。
上游已经合并的机型会标注出来，剩下的留在本地。

> 本文件放在 `local/device-notes` 分支，**不覆写上游 `README.md`**，也不进 `main`，
> 这样 `main` 仍可对上游做 fast-forward。

## 一、总览

| PCB | 品牌 / 型号 | 管理 | 端口 | 主控 | 闪存 | 构建 target | 状态 |
|---|---|---|---|---|---|---|---|
| `MD09HT` / `V30` | Xiaomi SW-106T-A | 否 | 4 × 2.5G + 2 × SFP+ | RTL8372N | 原厂 1 MiB cFeon `Q80C-104HIP`；实测机换 4 MiB | `XIAOMI_SW_106T_A` | **已合并上游**（PR #966） |
| `PCB-SWTG024AS-V1.0-16584` | ZX SWTG024AS | 否 | 4 × 2.5G + 2 × SFP+ | RTL8372 | 原厂 1 MiB；实测机换 4 MiB | `ZX-SWTG024AS` | **已合并上游**（PR #1028） |
| `2G5F4_10G2_V1.04` | hongyavision HYWS-SGT0204S | 否 | 4 × 2.5G + 2 × SFP+ | RTL8372 | 原厂 512 KiB `T25S40`；实测机换 4 MiB | `HONGYAVISION_HYWS_SGT0204S_V1_04` | 本地，待提 PR |
| `SE106 Pro 1.0.0 R`（板号 `2088501190`） | 水星 MERCURY SE106 Pro | **是**（原厂 Web，RSA 签名升级） | 4 × 2.5G + 2 × SFP+ | RTL8372N | 4 MiB XMC `XM25QH32C` | 未命名 | **未完成**（镜像写入正确但不启动） |

四台的共同点：都是 4 × 2.5GE + 2 × SFP+（1G/2.5G/10G），都用 512 KiB 镜像，
三台无管理机型没有出厂 MAC。

## 二、逐台说明

### 1. Xiaomi SW-106T-A（`MD09HT` V30）

```text
主控      RTL8372N（regget 4 = 0x83727000），外挂 RTL8224N PHY
端口      物理 RJ45 1-4 → 逻辑 4-7；左笼（SDS1，检测 GPIO38）→ 逻辑 8；右笼（SDS0，检测 GPIO37）→ 逻辑 3
闪存      原厂 1 MiB cFeon Q80C-104HIP，背面还有一处空置 SOP-8 位；我这台换了 4 MiB
MAC       无出厂 MAC，mac_flash_offset = 0，由芯片 UUID 派生（06: 前缀）
MDI       铜口线序镜像：0xa90 必须写 0x03（默认 0x0c 时物理 1、2 口不 link）
LED       铜口 绿=2.5G、橙=10M/100M/1G；每个 SFP 笼单灯（白，肉眼偏蓝），全速率共用
```

```text
make MACHINE=XIAOMI_SW_106T_A VERSION=<版本>
```

**实测**：四个铜口 1G/2.5G 通、两个 SFP 笼 10G 通、灯全对、Web/CLI/VLAN/配置保存重启正常。

**注意**：这台是本项目唯一需要改共享代码的机型——`struct machine` 新增 `mdi_cfg`，
用 `MDI_CFG(MDI_SWAPPED, MDI_SWAPPED, MDI_STRAIGHT, MDI_STRAIGHT)` 表达，宏在 `machine.h`。

### 2. ZX SWTG024AS（`PCB-SWTG024AS-V1.0-16584`，日期 2023-09-13）

```text
主控      RTL8372（非 N）
端口      物理 RJ45 1-4 → 逻辑 4-7；左笼（J4，SDS1，检测 GPIO30）→ 逻辑 8；右笼（J2，SDS0，检测 GPIO50）→ 逻辑 3
闪存      原厂 1 MiB；我这台换了 4 MiB
MAC       无出厂 MAC，派生同上
LED       铜口 绿=2.5G、橙=其余速率；每个 SFP 笼单灯（绿），全速率共用一个槽位
复位键    未贴片，reset_pin = GPIO_NA
```

```text
make MACHINE=ZX-SWTG024AS VERSION=<版本>
```

**实测**：四铜口 2.5G/1G、双笼 10G、灯全对、Web/CLI/VLAN/配置正常。

**特点**：纯板级新增，**没有改任何共享代码**，diff 只有 `+131/−0`。

### 3. hongyavision HYWS-SGT0204S（`2G5F4_10G2_V1.04`，日期 2024.09.19）

```text
主控      RTL8372（外挂 RTL8224 PHY），两颗 PPT PSM-4881 2.5G 端口模块
端口      物理 RJ45 1-4 → 逻辑 4-7；SFP1（左，逻辑 8，SDS1，检测 GPIO38）；SFP2（右，逻辑 3，SDS0，检测 GPIO37）
闪存      原厂 4 Mbit / 512 KiB，必须单线读（flash_sio = 1）；我这台换了 4 MiB
MAC       无出厂 MAC，派生同上
LED       铜口 白=2.5G、蓝=1000M L/A；每个 SFP 笼单灯；前面板另有 PWR
串口      板上有 TTL 插针；RTLPlayground 115200 8N1，原厂 9600 8N1
```

```text
make MACHINE=HONGYAVISION_HYWS_SGT0204S_V1_04 VERSION=<版本> CONFIG=<config.txt>
```

**与上游 V1.01 的关系**：`2G5F4_10G2_V1.01` 已在树里，但**两版 SFP 检测脚不同**
（V1.01 用 GPIO30/GPIO50，本板是 GPIO38/GPIO37）；用 V1.01 的脚位时 `sfp` 命令两个笼都报空、
且都不通流量。按规范，多版本必须带 `_V1_04` 后缀。

**已知问题**：13 颗灯里有 **3 颗无法驱动**，其余灯位与端口、Web UI 都正常。
正在借用同为该系列的 V1.01 实测配置做对照试验（`high_leds` 掩码、`port_led_set`、`led_sets` 结构）。

### 4. 水星 MERCURY SE106 Pro（`SE106 Pro 1.0.0 R`，板号 `2088501190`）

```text
主控      RTL8372N（regget 4 = 0x83727000）
端口      4 × 2.5G RJ45 + 2 × SFP+
闪存      4 MiB XMC XM25QH32C（32 Mbit，3.3 V，DIO）
原厂      带管理的 Web，升级校验 RSA 签名 → 只能编程器整片写入
原厂启动  两段式：启动段 0x0–0x19000（入口 0x0dc3，带 rt_header 的 magic/头校验/负载校验/长度），
          运行内核 0x1c000，配置区 0x1FB000
```

**状态：未完成。** 镜像写入确认正确（回读前 512 KiB 与源文件逐字节相同），但板子只有电源灯亮，
12 颗端口灯全黑、任何口都不 link。

已经排除的假设：

| 假设 | 依据 |
|---|---|
| 镜像头格式不对 | 原厂 `00 40 02 0d c1`，我们 `00 40 02 00 a8`，都是「预取 0x4000 + LJMP」 |
| 闪存时序参数不对 | 原厂代码写的 `CONFIG=9 / DIV=4 / RCMD=0xbb / TCONF=0x19` 与本源码一致 |
| 写入失败 | 回读校验通过 |

尚未分辨的三选一（外部表现完全一样）：**A** 启动 ROM 没交权、**B** 跑了但死在第一次邮箱写、
**C** 端口灯不在 GPIO 0–27 上（点 GPIO 证明不了什么）。决定性手段是串口（板上有 TTL 插针，参数未测）。

## 三、构建

```bash
git checkout main && git pull            # 小米、ZX 已在上游，直接从这里构建
make MACHINE=XIAOMI_SW_106T_A VERSION=0.2.8
make MACHINE=ZX-SWTG024AS VERSION=0.2.4

git checkout local/device-notes          # HYWS 还在本地分支上
make MACHINE=HONGYAVISION_HYWS_SGT0204S_V1_04 VERSION=0.2.12 CONFIG=config.txt
```

镜像固定 512 KiB；`CI=1` 会把警告当错误，`make machine_check` 编译全部机型。

## 四、共通经验

1. **无管理机没有 MAC**：`mac_flash_offset` 留 0，地址由 UUID 派生；要固定就用 `mac` 命令写进启动配置。
2. **换更大闪存不等于需要改固件**：512 KiB 镜像在 1 MiB 和 4 MiB 上都一样跑；真正影响的是
   `flash_sio`（有些 SPI NOR 不吃 Dual-I/O）。
3. **单灯 SFP 只能用一个 led-set 槽位**：把低速塞进第二个槽位等于驱动一个不存在的灯。
4. **`RTL837X_REG_LED_GLB_IO_EN` bit 6 要在 `machine_custom_init()` 置位**，否则「常亮」条件不起作用，
   只有活动闪烁有效。
5. **同一机型多 PCB 版本必须带版本后缀**（`_V1_04`），否则与已有条目冲突。

## 五、参考

- 上游仓库：<https://github.com/RTLPlayground/firmware>
- 同系列 V1.01 的实测记录（LED-ID 映射、`FLASH_NO_DIO` 经验）：
  <https://github.com/jzhou404gmail/RTLPlayground>
- GPIO/复用全表：上游 `doc/gpio.md`（`GPIO31` = UART TX，找串口时有用）