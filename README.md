# ZX-SWTG024AS - RTLPlayground firmware

Firmware and board support for the **ZX-SWTG024AS**
(PCB `PCB-SWTG024AS-V1.0-16584`), a RTL8372 4 x 2.5G + 2 x SFP+ switch.

## Status

| Item | Value |
|---|---|
| Model | `ZX-SWTG024AS` |
| PCB | `PCB-SWTG024AS-V1.0-16584` |
| SoC | RTL8372 |
| Ports | 4 x 2.5G RJ45 + 2 x SFP+ |
| Firmware | `V0.2.2` |
| Image | 512 KB, no 4 MB padding |

## Firmware

`firmware/ZX-SWTG024AS-V0.2.2.bin`

SHA256: `38b0f0082c5d8cfd8753e9c740ef2c17d033d9e571047104ff59e2b704034339`

Default:

```text
IP:      192.168.10.247
Netmask: 255.255.255.0
Gateway: 192.168.10.1
User:    admin
Pass:    admin
```

All ports VLAN1.

## LEDs

Enclosure definition:

```text
2.5G              -> green
10 / 100 / 1000M  -> amber
SFP5/6            -> single green
```

Known hardware note: 1 amber and 2/4 green are dim on this unit. Brightness is
not individually controllable from the LED engine.

## Build

```sh
make MACHINE=ZX-SWTG024AS \
     CONFIG=config/default.txt \
     VERSION=0.2.2
```

The image is exactly 524288 bytes.

## Flash

Use an external programmer, write the 512 KB image, verify, then power on.

## Documentation

- [Board notes](docs/boards/ZX-SWTG024AS.md)
- [V0.2.2 release](docs/iterations/V0.2.2.md)
- [Validation](docs/validation/test-results.md)

## CI

`.github/workflows/build-zx.yml` builds the image and checks the 512 KB size.

## Chinese

[README.zh-CN.md](README.zh-CN.md)
