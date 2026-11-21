# Den Dolda Kammaren — English

**Category:** Web (warmup). **Points:** 1000.
**Target:** `http://dendoldakammaren.challs.crate.nu` (nginx)
**Flag format:** `cratectf{...}` (literal value redacted — you recover it yourself).

The flag is hidden in client-side JavaScript: a Base64 blob that is deobfuscated by an
inline routine (a step-based additive offset followed by a modular-inverse multiply).
Reproduce that routine locally to print the flag.

## 1. Extract the obfuscated data from the page

```bash
curl -s http://dendoldakammaren.challs.crate.nu/
```

The HTML contains a hidden element (`<div style="display:none">`) with a "Visa flaggan"
("Show the flag") button and an inline `<script>` that holds:

* the Base64 `OBFUSCATED` string, and
* the four decode constants.

From the live page (confirm these values in your own fetch — they are the challenge's own
constants, not per-session secrets):

```
OBFUSCATED = "J//3S4FKYLVHIMJkDsCBnwtv+jCrfBZMapDs8612bQCikuKvtn/zvF4THiUdM48="
KEY     = 0x2A   (42)
MUL     = 31
MUL_INV = 223    (modular inverse of 31 mod 256)
STEP    = 7
```

Grep to locate them quickly:

```bash
curl -s http://dendoldakammaren.challs.crate.nu/ \
  | grep -nE 'OBFUS|KEY|STEP|MUL|atob|base64'
```

## 2. The decode algorithm

For each byte `b` at index `i` of the Base64-decoded blob:

1. subtract the key and a per-index step offset: `c = ((b - KEY - STEP*i) mod 256)`,
   normalised into `0..255`;
2. multiply by the modular inverse: `out[i] = (c * MUL_INV) mod 256`.

Treat the output bytes as Latin-1 characters — the flag contains `ö`/`ä` (byte `0xF6`
etc.), so decode as Latin-1, not strict ASCII/UTF-8-only.

## 3. Reproduce locally

```python
import base64

KEY, MUL_INV, STEP = 0x2A, 223, 7
OBFUSCATED = "J//3S4FKYLVHIMJkDsCBnwtv+jCrfBZMapDs8612bQCikuKvtn/zvF4THiUdM48="

raw = base64.b64decode(OBFUSCATED)
out = []
for i, b in enumerate(raw):
    c = ((b - KEY - STEP * i) % 256 + 256) % 256
    out.append((c * MUL_INV) % 256)
print(bytes(out).decode("latin-1"))
```

Run it:

```bash
python3 decode.py
```

## What the output shows

The script prints the flag in `cratectf{...}` format (it includes Swedish characters
such as `ö`, which is why Latin-1 decoding matters). Clicking the page's "Visa flaggan"
button runs the exact same routine in the browser and displays the same string in the
previously hidden `display:none` element.

## Verification

Platform-confirmed solved. Base64-decoding the inline `OBFUSCATED` string and applying
`out = ((b - 0x2A - 7*i) mod 256) * 223 mod 256` per byte yields the flag in
`cratectf{...}` format (reproduced independently by multiple solvers).
