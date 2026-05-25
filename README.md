<div align="center">

# Arm69

**A riffle shuffle cipher for Roblox and beyond**

[![Version](https://img.shields.io/badge/version-1.0.0-6C3EF4?style=flat-square&logoColor=white)](https://github.com/Kr3ativeKrayon/Arm69)
[![Stability](https://img.shields.io/badge/stability-experimental-f59e0b?style=flat-square)](https://github.com/Kr3ativeKrayon/Arm69)
[![License](https://img.shields.io/badge/license-MIT-6C3EF4?style=flat-square)](https://github.com/Kr3ativeKrayon/Arm69/blob/main/LICENSE)
[![Luau](https://img.shields.io/badge/luau-typed-6C3EF4?style=flat-square)](https://luau-lang.org)
[![Built by Plinko Labs](https://img.shields.io/badge/built%20by-Plinko%20Labs-6C3EF4?style=flat-square)](https://github.com/Kr3ativeKrayon)

</div>

---

Arm69 is a symmetric cipher built on repeated riffle shuffles. Every character maps to a two-digit numeric code. The key can be any string -- it is encoded the same way as the message before use. The key drives a unique split offset on every shuffle pass, preventing cycle attacks. Split tokens are indexed back into the encoded key so decoding is impossible without it.

---

## Table of Contents

* [How It Works](#how-it-works)
  * [Step 1 -- Encode](#step-1--encode)
  * [Step 2 -- Force Even Length](#step-2--force-even-length)
  * [Step 3 -- Riffle Shuffle](#step-3--riffle-shuffle)
* [Decoding](#decoding)
* [Key Requirements](#key-requirements)
* [Character Map](#character-map)
* [Security Properties](#security-properties)
* [Example](#example)
* [Contact](#contact)

---

## How It Works

### Step 0 -- Encode the Key

Before anything else, the key is encoded using the same character map as the message. This means the key can be any string -- letters, symbols, numbers, or a mix.

```
Key:    SecretKey!
Encoded: !19 05 03 18 05 20 !11 05 25 41
```

All subsequent steps use these encoded values, not the raw key string. Encoded key values range from `01` to `69`, which gives split offsets far more variation than raw digits alone, and guarantees that any two indices can sum to cover the full token value range.

---

### Step 1 -- Encode

Every character in the input string converts to a two-digit numeric code. Uppercase letters get a `!` prefix.

```
Input:   Hello World
Output:  !08 05 12 12 15 27 !23 15 18 12 04
```

See the full [Character Map](#character-map) below.

---

### Step 2 -- Force Even Length

The riffle requires an even number of tokens. If the token list is odd in length, the middle token is split into two key-indexed tokens that sum back to the original value.

**Format:**

```
!XXxYY
```

`XX` and `YY` are indices into the key. The decoder sums `key[XX] + key[YY]` to recover the original token. Without the key, the split cannot be resolved.

Index width scales with key length:

| Key Length | Index Format | Example |
|------------|--------------|---------|
| 1-9 | `!XxY` | `!2x3` |
| 10-99 | `!XXxYY` | `!02x03` |
| 100-999 | `!XXXxYYY` | `!002x003` |
| 1000+ | `!XXXXxYYYY` | `!0002x0003` |

---

### Step 3 -- Riffle Shuffle

The token list is riffled **N times**, where N equals the token count after the even-length fix.

Each pass splits the list into a left half and right half, then interleaves them:

```
Left:   A  B  C
Right:  D  E  F

Result: A  D  B  E  C  F
```

**Key-driven split offset:**

A perfect riffle on a fixed-length list cycles back to its original order after enough passes. To break this, each pass offsets the split point using the current encoded key value, clamped to stay in bounds:

```
offset = encodedKey[i mod keyLength] mod (tokenCount / 2)
split  = center + offset
```

Every pass performs a different split. Without the key the split sequence is unknown and the cipher cannot be reversed by brute force.

---

## Decoding

Decoding runs every step in reverse:

1. **Un-riffle** -- iterate passes from last to first, recover each split point from the key, un-interleave
2. **Restore split tokens** -- resolve all `!XXxYY` tokens by summing key indices and recombining
3. **Decode** -- convert each numeric code back to its original character

---

## Key Requirements

- The key must contain digit characters
- Index width is determined by `len(tostring(#key))`
- The key must be varied enough that some pair of indices sums to any value between `01` and `69`
- Keys that are too short or too uniform may fail the even-length fix step

---

## Character Map

**Letters (01-26)**

Lowercase is the plain code. Uppercase uses a `!` prefix.

```
a=01  b=02  c=03  d=04  e=05  f=06  g=07  h=08  i=09  j=10
k=11  l=12  m=13  n=14  o=15  p=16  q=17  r=18  s=19  t=20
u=21  v=22  w=23  x=24  y=25  z=26
```

**Space and Digits (27-37)**

```
 =27  1=28  2=29  3=30  4=31  5=32  6=33  7=34  8=35  9=36  0=37
```

**Special Characters (38-69), ordered by frequency**

```
38=.  39=,  40=?  41=!  42='  43="  44=-  45=(  46=)  47=:
48=;  49=@  50=/  51=\  52=*  53=_  54=+  55==  56=[  57=]
58={  59=}  60=<  61=>  62=#  63=%  64=&  65=^  66=$  67=~
68=`  69=|
```

---

## Security Properties

| Property | Notes |
|----------|-------|
| Key-dependent decode | `!XXxYY` tokens are unresolvable without the key |
| No fixed shuffle cycle | Key-offset split breaks periodic cycling |
| Full keyboard support | All 69 standard keyboard characters are encodable |
| Case preservation | Uppercase and lowercase are distinct via `!` prefix |
| Character obfuscation | Every character becomes a uniform two-digit code |

---

## Example

**Input:**
```
Hello World The Secret Key Is: 1231234532
```

**Key:**
```
1231234532
```

**Encoded (42 tokens after even-length fix):**
```
!08 05 12 12 15 27 !23 15 18 12 04 27 !20 08 05 27 !19 05 03 18 !02 !03 20 27 !11 05 25 27 !09 19 47 27 28 29 30 28 29 30 31 32 30 29
```

**After 42 riffle passes:**
```
!08 27 !03 27 05 28 20 !20 12 29 27 08 12 30 !11 05 15 28 05 27 27 29 25 !19 !23 30 27 05 15 31 !09 03 18 32 19 18 12 30 47 !02 04 29
```

---

## Contact

| Platform | Handle |
|----------|--------|
| Roblox | [Kr3ativeKrayon](https://www.roblox.com/users/1911367519/profile) |
| YouTube | [TotallyKr3ative](https://www.youtube.com/channel/UCpNZQoKVclQ74Pk5GmzdQDA) |
| X (Twitter) | [TotallyNotKr3ative](https://x.com/TheRealKr3ative) |
| Email | [TheRealKr3ative@gmail.com](mailto:TheRealKr3ative@gmail.com) |

---

*Built by Plinko Labs*

*Last Updated: May 2026*
