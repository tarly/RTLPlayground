# ZX-SWTG024AS - RTLPlayground 固件

适用于 **ZX-SWTG024AS**，PCB `PCB-SWTG024AS-V1.0-16584`，RTL8372，
4 x 2.5G RJ45 + 2 x SFP+。

## 状态

| 项目 | 值 |
|---|---|
| 型号 | `ZX-SWTG024AS` |
| PCB | `PCB-SWTG024AS-V1.0-16584` |
| 主控 | RTL8372 |
| 端口 | 4 x 2.5G + 2 x SFP+ |
| 固件 | `V0.2.2` |
| 镜像 | 512 KB，不做 4 MB padding |

## 固件

`firmware/ZX-SWTG024AS-V0.2.2.bin`

SHA256：`38b0f0082c5d8cfd8753e9c740ef2c17d033d9e571047104ff59e2b704034339`

默认配置：

```text
IP:      192.168.10.247
Netmask: 255.255.255.0
Gateway: 192.168.10.1
用户名: admin
密码:   admin
```

所有端口 VLAN1。

## LED

外壳定义：

```text
2.5G              -> 绿灯
10 / 100 / 1000M  -> 橙灯
SFP5/6            -> 单色绿灯
```

本机 1 橙、2/4 绿偏弱；LED 引擎不能单独调亮度，不是固件问题。

## 构建

```sh
make MACHINE=ZX-SWTG024AS \
     CONFIG=config/default.txt \
     VERSION=0.2.2
```

镜像固定 524288 字节。

## 刷写

使用编程器写 512 KB 镜像，校验后上电。

## 文档

- [板级说明](docs/boards/ZX-SWTG024AS.md)
- [V0.2.2 发布](docs/iterations/V0.2.2.md)
- [验证结果](docs/validation/test-results.md)

## CI

`.github/workflows/build-zx.yml` 会构建并检查 512 KB。

## 英文

[README.md](README.md)
