# RTLPlayground local device notes (the four switches in this fork)

| Language | Document |
|---|---|
| 中文 | [README.md](README.md) |
| **English** | [README.en.md](README.en.md) (this page) |

---

> **AI assistance declaration**
> This document and the board support it describes were prepared with the help of an
> **AI coding assistant (DeepSeek Harness)**. Every change was reviewed, built and tested
> on the **actual hardware** by the author; images were verified before writing, and the
> results below come from measurements rather than guesses. Anything not yet verified is
> marked as such.
>
> **AI 辅助声明**
> 本文档以及其中描述的板级适配工作，由 **AI 编码助手（DeepSeek Harness）**协助完成。
> 所有改动均由作者在**真实设备**上复核、编译并实测；固件在写入前经过校验，
> 结论均来自实机现象而非推测，未验证的部分在文中明确标注。

---

This document records the devices I have actually ported in `tarly/RTLPlayground`, what was
measured, and the traps that cost time. Boards that are already upstream are marked; the rest
stay local.

> This branch (`local/device-notes`) holds **documentation only**; the code lives on `main`,
> which stays in sync with upstream and can always be fast-forwarded.
> The original upstream README is kept as [README.upstream.md](README.upstream.md).

## 1. Overview

| PCB | Brand / model | Managed | Ports | SoC | Flash | Build target | Status |
|---|---|---|---|---|---|---|---|
| `MD09HT` / `V30` | Xiaomi SW-106T-A | No | 4 x 2.5G + 2 x SFP+ | RTL8372N | stock 1 MiB cFeon `Q80C-104HIP`; the tested unit has 4 MiB fitted | `XIAOMI_SW_106T_A` | **merged upstream** (PR #966) |
| `PCB-SWTG024AS-V1.0-16584` | ZX SWTG024AS | No | 4 x 2.5G + 2 x SFP+ | RTL8372 | stock 1 MiB; the tested unit has 4 MiB fitted | `ZX-SWTG024AS` | **merged upstream** (PR #1028) |
| `2G5F4_10G2_V1.04` | hongyavision HYWS-SGT0204S | No | 4 x 2.5G + 2 x SFP+ | RTL8372 | stock 512 KiB `T25S40`; the tested unit has 4 MiB fitted | `HONGYAVISION_HYWS_SGT0204S_V1_04` | local, PR not opened yet |
| `SE106 Pro 1.0.0 R` (board no. `2088501190`) | MERCURY SE106 Pro | **Yes** (stock web UI, RSA-signed updates) | 4 x 2.5G + 2 x SFP+ | RTL8372N | 4 MiB XMC `XM25QH32C` | not named yet | **unfinished** (image verified, board does not start) |

What the four share: 4 x 2.5GBASE-T + 2 x SFP+ (1G/2.5G/10G), a 512 KiB image, and - for the
three unmanaged ones - no factory MAC.

## 2. Per-device notes

### 2.1 Xiaomi SW-106T-A (`MD09HT` V30)

```text
SoC       RTL8372N (regget 4 = 0x83727000) with external RTL8224N PHYs
Ports     physical RJ45 1-4 -> logical 4-7; left cage (SDS1, detect GPIO38) -> logical 8;
          right cage (SDS0, detect GPIO37) -> logical 3
Flash     stock 1 MiB cFeon Q80C-104HIP, plus an unpopulated second SOIC-8 footprint;
          the tested unit has 4 MiB fitted
MAC       no factory MAC, mac_flash_offset = 0, address derived from the chip UUID (06: prefix)
MDI       copper pairs are mirror-image: register 0xa90 has to be 0x03
          (with the reset default 0x0c physical ports 1 and 2 never link)
LEDs      copper: green = 2.5G, amber = 10M/100M/1G.
          Each SFP+ cage has a single white lamp (looks slightly blue) shared by all speeds
```

```text
make MACHINE=XIAOMI_SW_106T_A VERSION=<version>
```

**Measured:** all four copper ports at 1G and 2.5G, both cages at 10G, every LED correct,
Web UI / CLI / VLAN / configuration save and reboot all working.

**Note:** this is the only board here that needs a change outside the board file -
`struct machine` gained `mdi_cfg`, written as
`MDI_CFG(MDI_SWAPPED, MDI_SWAPPED, MDI_STRAIGHT, MDI_STRAIGHT)`; the macros live in `machine.h`.

### 2.2 ZX SWTG024AS (`PCB-SWTG024AS-V1.0-16584`, dated 2023-09-13)

```text
SoC       RTL8372 (non-N)
Ports     physical RJ45 1-4 -> logical 4-7; left cage (J4, SDS1, detect GPIO30) -> logical 8;
          right cage (J2, SDS0, detect GPIO50) -> logical 3
Flash     stock 1 MiB; the tested unit has 4 MiB fitted
MAC       no factory MAC, derived as above
LEDs      copper: green = 2.5G, amber = the other speeds; each SFP+ cage has a single green
          lamp, so every speed and the blink share one led-set slot
Reset     footprint unpopulated, reset_pin = GPIO_NA
```

```text
make MACHINE=ZX-SWTG024AS VERSION=<version>
```

**Measured:** four copper ports at 2.5G and 1G, both cages at 10G, every LED correct,
Web UI / CLI / VLAN / configuration working.

**Shape of the change:** board files only - **no shared code touched**, the diff is `+131/-0`.

### 2.3 hongyavision HYWS-SGT0204S (`2G5F4_10G2_V1.04`, dated 2024.09.19)

```text
SoC       RTL8372 with external RTL8224 PHYs, two PPT PSM-4881 2.5G port modules
Ports     physical RJ45 1-4 -> logical 4-7; SFP1 (left, logical 8, SDS1, detect GPIO38);
          SFP2 (right, logical 3, SDS0, detect GPIO37)
Flash     stock 4 Mbit / 512 KiB, needs single-IO reads (flash_sio = 1);
          the tested unit has 4 MiB fitted
MAC       no factory MAC, derived as above
LEDs      copper: white = 2.5G, blue = 1000M link/activity; one lamp per SFP+ cage;
          the front panel also has a PWR lamp
Serial    TTL header present; RTLPlayground 115200 8N1, stock firmware 9600 8N1
```

```text
make MACHINE=HONGYAVISION_HYWS_SGT0204S_V1_04 VERSION=<version> CONFIG=<config.txt>
```

**Relation to the upstream V1.01 profile:** `2G5F4_10G2_V1.01` is already in the tree, but the
two revisions **use different SFP detect pins** (V1.01 uses GPIO30/GPIO50, this board uses
GPIO38/GPIO37). With the V1.01 pins the `sfp` command reports both cages empty and neither
passes traffic. Per the naming rules, a second revision of the same product carries a `_V1_04`
suffix.

**Known problem:** three of the thirteen lamps cannot be driven; every port, the remaining
lamps and the Web UI are fine. Comparing against the measured V1.01 configuration
(`high_leds` masks, `port_led_set`, `led_sets` layout) is in progress.

### 2.4 MERCURY SE106 Pro (`SE106 Pro 1.0.0 R`, board no. `2088501190`)

```text
SoC       RTL8372N (regget 4 = 0x83727000)
Ports     4 x 2.5G RJ45 + 2 x SFP+
Flash     4 MiB XMC XM25QH32C (32 Mbit, 3.3 V, DIO)
Stock     managed web UI, updates are RSA-signed -> external programmer is the only way in
Stock boot  two stages: boot stage 0x0-0x19000 (entry 0x0dc3, rt_header with magic, header
           checksum, payload checksum and length), runtime kernel at 0x1c000,
           configuration at 0x1FB000
```

**Status: unfinished.** The image is definitely written - reading the flash back gives a
byte-identical first 512 KiB - yet the board only lights its power LED, all twelve port LEDs stay
dark and no port links.

Assumptions already ruled out:

| Assumption | Evidence |
|---|---|
| image header is wrong | stock `00 40 02 0d c1`, ours `00 40 02 00 a8` - both "prefetch 0x4000 + LJMP" |
| flash timing parameters are wrong | the stock code programs `CONFIG=9 / DIV=4 / RCMD=0xbb / TCONF=0x19`, identical to this tree |
| the write failed | read-back verification passes |

Three possibilities remain, indistinguishable from the outside (everything dark):
**A** the boot ROM never hands over, **B** the image runs but dies at the first mailbox write,
**C** the port lamps are not on GPIO 0-27, so toggling a GPIO proves nothing. The decisive tool
is the serial console (a TTL header exists, its settings are not measured yet).

## 3. Building

```bash
git checkout main && git pull            # Xiaomi and ZX are upstream now
make MACHINE=XIAOMI_SW_106T_A VERSION=0.2.8
make MACHINE=ZX-SWTG024AS VERSION=0.2.4

git checkout local/device-notes          # HYWS still lives on this branch
make MACHINE=HONGYAVISION_HYWS_SGT0204S_V1_04 VERSION=0.2.12 CONFIG=config.txt
```

The image is always 512 KiB. `CI=1` turns warnings into errors, `make machine_check` compiles
every registered machine.

## 4. Things worth knowing

1. **Unmanaged boards have no MAC**: leave `mac_flash_offset` at 0 and let the address come from
   the chip UUID; to pin it, use the `mac` command and save the startup configuration.
2. **A bigger flash does not mean firmware changes**: the same 512 KiB image runs on 1 MiB and
   4 MiB parts. What actually matters is `flash_sio`, because some SPI NOR parts cannot do
   Dual-I/O reads.
3. **A single-lamp SFP cage gets one led-set slot**: putting the lower speeds into a second slot
   drives a lamp that does not exist, so a 1G/2.5G module never lights.
4. **`RTL837X_REG_LED_GLB_IO_EN` bit 6 must be set in `machine_custom_init()`**, otherwise the
   "solid" conditions do not drive the lamps at all and only activity blinking works.
5. **Several PCB revisions of one product need a version suffix** (`_V1_04`), otherwise the
   machine entry collides with the revision already in the tree.

## 5. References

- Upstream: <https://github.com/RTLPlayground/firmware>
- Measured notes for the V1.01 sibling of the HYWS board (LED-ID mapping, `FLASH_NO_DIO`
  experience): <https://github.com/jzhou404gmail/RTLPlayground>
- GPIO and mux table: upstream `doc/gpio.md` (`GPIO31` = UART TX, useful when hunting a console)


## 6. Token consumption (as of 2026-11-28 23:45)

How this is measured: the usage recorded step by step in the local DSH session logs
(`~/.dsh/sessions/`) is summed over **the four devices plus the project-wide coordination session, six
sessions in total**; sessions of other projects on the same machine are excluded. Cost uses
DeepSeek's `deepseek-flash` (V4.1-Flash-1017) **peak** rates: cache hit $0.006/M, cache miss
$0.30/M, output $1.20/M.

| Device / role | Sessions | Steps | Input, cache miss | Output | Input, cache hit | Total tokens | Peak cost |
|---|---|---|---|---|---|---|---|
| Project-wide coordination, sweeps and PRs | 1 | 915 | 10,438,539 | 785,585 | 315,663,360 | 326,887,484 | $5.97 |
| Xiaomi SW-106T-A | 2 | 1808 | 6,939,322 | 1,632,369 | 781,152,256 | 789,723,947 | $8.72 |
| HYWS-SGT0204S | 1 | 354 | 3,208,905 | 395,540 | 114,015,744 | 117,620,189 | $2.12 |
| MERCURY SE106 Pro | 1 | 258 | 2,252,454 | 279,982 | 62,684,800 | 65,217,236 | $1.39 |
| ZX SWTG024AS | 1 | 18 | 108,680 | 17,573 | 811,008 | 937,261 | $0.06 |
| **Total** | **6** | **3353** | **22,947,900** | **3,111,049** | **1,274,327,168** | **1,300,386,117** | **$18.26** |

Breakdown:

```text
input, cache miss   22,947,900  x $0.30/M  =  $6.88
input, cache hit  1,274,327,168 x $0.006/M =  $7.65
output              3,111,049  x $1.20/M  =  $3.73
---------------------------------------------
total at peak rates                        $18.26
off-peak equivalent                         $9.13
```

Notes:

- **The Xiaomi board accounts for about half the cost** ($8.72 of $18.26): 98% of that comes
  from the first session, which took the stock flash apart with binwalk to work out the GPIO
  layout - 1776 steps.
- **Project-wide coordination, sweeps and PRs is the second largest** ($5.97): it re-reads a large context on
  every turn, so it has the highest cache-miss input of the six.
- **ZX cost $0.06** (18 steps) - its board file was a copy-and-adjust of the Xiaomi one, which
  shows how cheap another device becomes once the pattern exists.
- **Cache hits are 98% of the tokens but 42% of the cost**, because a hit is 1/50 the price of
  a miss.
