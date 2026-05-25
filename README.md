# Arm69

> A key-driven riffle shuffle cipher with positional encoding.

---

## Overview

**Arm69** is a symmetric cipher built around the mechanics of a card riffle shuffle. The name comes from the rifle — a firearm — since a riffle *is* a rifle shuffle. The `69` refers to the cipher's full encoding space: 69 unique character codes covering every key on a standard US keyboard.

Arm69 is not a black-box cipher. Every step is transparent, deterministic, and reversible — but only with the correct key.

---

## How It Works

### Step 1 — Encode

Every character in the input string is converted to a two-digit numeric code.

| Range   | Characters                        |
|---------|-----------------------------------|
| `01–26` | `a–z` (lowercase)                 |
| `01–26` | `!01–!26` uppercase (with `!` prefix) |
| `27`    | ` ` space                         |
| `28–37` | `1234567890`                      |
| `38–69` | Special characters (by frequency) |

**Special character map (38–69), ordered most to least used:**

```
38=.  39=,  40=?  41=!  42='  43="  44=-  45=(  46=)  47=:
48=;  49=@  50=/  51=\  52=*  53=_  54=+  55==  56=[  57=]
58={  59=}  60=<  61=>  62=#  63=%  64=&  65=^  66=$  67=~
68=`  69=|
```

**Example:**

```
Input:   Hello World
Encoded: !08 05 12 12 15 27 !23 15 18 12 04
```

---

### Step 2 — Force Even Length

The riffle shuffle requires an even number of tokens. If the encoded token list is odd in length, the middle token is split into two tokens that sum to the original value.

Split tokens are encoded using key index references:

```
!XXxYY
```

Where `XX` and `YY` are indices into the key such that `key[XX] + key[YY]` equals the original token value. The index width scales with key length:

| Key Length  | Index Format   | Example       |
|-------------|----------------|---------------|
| 1–9         | `!XxY`         | `!2x3`        |
| 10–99       | `!XXxYY`       | `!02x03`      |
| 100–999     | `!XXXxYYY`     | `!002x003`    |
| 1000–9999   | `!XXXXxYYYY`   | `!0002x0003`  |

The decoder recovers the original value by looking up both indices in the key and summing them. Without the key, the split cannot be resolved.

---

### Step 3 — Riffle Shuffle (N passes)

The token list is riffled **N times**, where N equals the number of tokens after the even-length fix.

Each pass:
1. Split the list into a left half and right half
2. Interleave them — one token from left, one from right, repeat

```
Left:   A  B  C
Right:  D  E  F

Result: A  D  B  E  C  F
```

**Key-driven split offset:**

To prevent the shuffle from cycling back to the original (a known weakness of perfect riffle shuffles), the split point is offset each pass using the corresponding key digit:

```
Pass i → split at center ± key[i mod keyLength]
```

This means every pass performs a different split, breaking the cycle. Without the key, the split sequence is unknown and the shuffle cannot be reversed by brute force.

---

## Decoding

Decoding is the exact reverse of encoding:

1. **Un-riffle** — run each pass in reverse order, using the same key digits to recover the split point and un-interleave
2. **Restore split tokens** — find all `!XXxYY` tokens, resolve them using the key, and recombine
3. **Decode** — convert each numeric code back to its character

---

## Key Requirements

- The key must contain digit characters
- For a key of length L, index width is `len(str(L))`
- The key must be long and varied enough that some pair of indices `key[X] + key[Y]` can reach any value from `01` to `69`
- Short or low-value keys may fail the even-length fix if no valid pair of indices sums to the middle token's value

---

## Security Properties

| Property              | Notes                                                  |
|-----------------------|--------------------------------------------------------|
| Key-dependent decode  | `!XXxYY` tokens are unresolvable without the key      |
| No fixed shuffle cycle | Key-offset split prevents period cycling              |
| Character obfuscation | Every character becomes a 2-digit numeric code        |
| Case encoding         | Uppercase/lowercase distinction preserved via `!`     |
| Full keyboard support | All 69 US keyboard characters are encodable           |

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

**Encoded (42 tokens after even-fix):**
```
!08 05 12 12 15 27 !23 15 18 12 04 27 !20 08 05 27 !19 05 03 18 !02 !03 20 27 !11 05 25 27 !09 19 47 27 28 29 30 28 29 30 31 32 30 29
```

**After 42 riffle passes:**
```
!08 27 !03 27 05 28 20 !20 12 29 27 08 12 30 !11 05 15 28 05 27 27 29 25 !19 !23 30 27 05 15 31 !09 03 18 32 19 18 12 30 47 !02 04 29
```

---

## Name

**Arm** — from *firearm*. A rifle is a firearm. The cipher is built on a riffle.  
**69** — the size of the encoding map. Every keyboard character, one code.

---

*Arm69 is an original cipher design. It is not intended as a replacement for production-grade cryptographic standards such as AES. Use for educational, experimental, or game development purposes.*
