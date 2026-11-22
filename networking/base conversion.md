#clankerSlop #networking #ccna

Networking uses three number systems: **decimal** (base 10) for how humans write [[IP|IPv4]] addresses, **binary** (base 2) for how devices actually process them, and **hexadecimal** (base 16) for [[MAC]] addresses, [[IPv6]] addresses, and protocol fields like EtherType. [[subnetting|Subnetting]] is just binary math, so being fast at converting between bases is one of the most useful CCNA skills.

## Binary

Each binary digit (a **bit**) is either 0 or 1. Eight bits make an **octet** (byte). Each position is worth twice the one to its right.

| Bit position | 8th     | 7th    | 6th    | 5th    | 4th   | 3rd   | 2nd   | 1st   |
| ------------ | ------- | ------ | ------ | ------ | ----- | ----- | ----- | ----- |
| Power of 2   | 2⁷      | 2⁶     | 2⁵     | 2⁴     | 2³    | 2²    | 2¹    | 2⁰    |
| **Value**    | **128** | **64** | **32** | **16** | **8** | **4** | **2** | **1** |

> [!tip] Memorize this row
> **128 64 32 16 8 4 2 1**. Write it at the top of your scratch paper before any subnetting question. An octet's maximum value is the sum of all of them: **255**.

### Binary → Decimal

Add up the values of every position that holds a `1`.

```
  1   1   0   0   0   0   0   0
128 +64                           = 192

  1   0   1   0   1   0   0   0
128     +32     +8                = 168
```

### Decimal → Binary (subtraction method)

Go left to right. If the position value fits into what's left, write `1` and subtract it; otherwise write `0`.

> [!example] Convert 172 to binary
> | 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |
> |---|---|---|---|---|---|---|---|
> | 1 | 0 | 1 | 0 | 1 | 1 | 0 | 0 |
> 172 − 128 = 44 → 64 doesn't fit → 44 − 32 = 12 → 16 doesn't fit → 12 − 8 = 4 → 4 − 4 = 0 → rest 0
> **172 = `10101100`**

### Values you'll see constantly

These are the [[subnetting#Valid mask octet values|subnet mask octets]]: each one adds the next bit from the left.

| Binary     | Decimal | Binary     | Decimal |
| ---------- | ------- | ---------- | ------- |
| `10000000` | 128     | `11111000` | 248     |
| `11000000` | 192     | `11111100` | 252     |
| `11100000` | 224     | `11111110` | 254     |
| `11110000` | 240     | `11111111` | 255     |

## Hexadecimal

Hex uses 16 symbols: `0–9` then `A–F` (A = 10 … F = 15). **One hex digit = exactly 4 bits** (a *nibble*), which is why hex is such a convenient shorthand for binary.

| Dec | Hex | Binary | Dec | Hex | Binary |
| --- | --- | ------ | --- | --- | ------ |
| 0   | 0   | `0000` | 8   | 8   | `1000` |
| 1   | 1   | `0001` | 9   | 9   | `1001` |
| 2   | 2   | `0010` | 10  | A   | `1010` |
| 3   | 3   | `0011` | 11  | B   | `1011` |
| 4   | 4   | `0100` | 12  | C   | `1100` |
| 5   | 5   | `0101` | 13  | D   | `1101` |
| 6   | 6   | `0110` | 14  | E   | `1110` |
| 7   | 7   | `0111` | 15  | F   | `1111` |

### Hex ↔ Binary

Convert each hex digit to 4 bits on its own (and back).

```
Hex     2    0    0    1    :    0    D    B    8
Binary  0010 0000 0000 0001 :    0000 1101 1011 1000
```

### Hex ↔ Decimal

A two-digit hex number = (first digit × 16) + second digit.

| Hex      | Calculation | Decimal                   |
| -------- | ----------- | ------------------------- |
| `0x1A`   | 1×16 + 10   | 26                        |
| `0xC0`   | 12×16 + 0   | 192                       |
| `0xFF`   | 15×16 + 15  | 255                       |
| `0x0800` | 8×256       | 2048 (EtherType for IPv4) |

To go decimal → hex: divide by 16; the quotient is the first digit, the remainder is the second. 200 ÷ 16 = 12 r 8 → `C8`.

## Where Each Base Shows Up

| Base    | Where you see it                                                    | Example                                                |
| ------- | ------------------------------------------------------------------- | ------------------------------------------------------ |
| Decimal | IPv4 addresses, masks, VLAN IDs, port numbers                       | `192.168.1.1`, `255.255.255.0`                         |
| Binary  | Subnetting, [[wildcard mask\|wildcard masks]], header flags, ANDing | `11111111.11111111.11111111.11000000`                  |
| Hex     | [[MAC]] addresses, [[IPv6]], EtherType, TPID, config register       | `00:1A:2B:3C:4D:5E`, `2001:db8::1`, `0x8100`, `0x2102` |

> [!question] Quick glossary
> | Term | Meaning |
> |---|---|
> | **Bit** | One binary digit, 0 or 1 |
> | **Octet / byte** | 8 bits, values 0–255 |
> | **Nibble** | 4 bits = one hex digit |
> | **`0x` prefix** | Marks a number as hexadecimal |
> | **MSB** | [[most significant bits\|Most significant bit]], the leftmost (128) bit |
