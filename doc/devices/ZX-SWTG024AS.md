# ZX-SWTG024AS

| Field | Value |
|---|---|
| Model | ZX-SWTG024AS |
| PCB | PCB-SWTG024AS-V1.0-16584 |
| PCB date | 2023-09-13 |
| SoC | RTL8372 |
| Copper | 4 × 2.5G RJ45 |
| Uplink | 2 × SFP+ |
| Stock firmware | not available |
| UART | present, not required for flashing |
| Tested firmware | V0.2.1, 512 KB |

## Ports

- physical 1–4 → logical 4–7
- SFP5 → logical 8 / SDS1
- SFP6 → logical 3 / SDS0

## LED

- 2.5G: green
- 10/100/1000M: amber
- SFP5/6: single green

Measured pads:

```text
SFP6 green   9
1 green     12
2 green     15
2 amber     17
3 green     18
3 amber     20
4 green     21
4 amber     23
SFP5 green  24
1 amber      not confirmed
```

Brightness of 1 amber and 2/4 green is weak on this unit; hardware.

## Flash

- use external programmer
- 512 KB image
- no 4 MB padding

## Profile

Based on upstream RTLPlayground `SWTG024AS` 4+2 unmanaged profile.
