# Validation

| Test | Result |
|---|---|
| 1–4 port 2.5G link | pass |
| 1–4 port 2.5G ping | pass |
| 1G copper | amber |
| 2.5G copper | green |
| SFP5 | single green |
| SFP6 | single green |
| Web UI | pass |
| Default IP .10.247 | pass |
| Reboot | pass |

Known: 1 amber and 2/4 green are dim; hardware.

## Pad map

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
