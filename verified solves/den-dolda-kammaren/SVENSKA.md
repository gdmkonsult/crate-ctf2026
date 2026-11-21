# Den Dolda Kammaren — Svenska

**Kategori:** Webb (uppvärmning). **Poäng:** 1000.
**Mål:** `http://dendoldakammaren.challs.crate.nu` (nginx)
**Flaggformat:** `cratectf{...}` (det bokstavliga värdet är redigerat — du återskapar
det själv).

Flaggan är gömd i JavaScript på klientsidan: en Base64-sträng som avobfuskeras av en
inline-rutin (en stegbaserad additiv förskjutning följd av en multiplikation med
modulär invers). Återskapa den rutinen lokalt för att skriva ut flaggan.

## 1. Extrahera den obfuskerade datan från sidan

```bash
curl -s http://dendoldakammaren.challs.crate.nu/
```

HTML:en innehåller ett dolt element (`<div style="display:none">`) med en knapp "Visa
flaggan" och ett inline-`<script>` som innehåller:

* Base64-strängen `OBFUSCATED`, och
* de fyra avkodningskonstanterna.

Från den live-sidan (bekräfta värdena i din egen hämtning — de är challengens egna
konstanter, inte per-session-hemligheter):

```
OBFUSCATED = "J//3S4FKYLVHIMJkDsCBnwtv+jCrfBZMapDs8612bQCikuKvtn/zvF4THiUdM48="
KEY     = 0x2A   (42)
MUL     = 31
MUL_INV = 223    (modulär invers av 31 mod 256)
STEP    = 7
```

Grep:a för att hitta dem snabbt:

```bash
curl -s http://dendoldakammaren.challs.crate.nu/ \
  | grep -nE 'OBFUS|KEY|STEP|MUL|atob|base64'
```

## 2. Avkodningsalgoritmen

För varje byte `b` på index `i` i den Base64-avkodade strängen:

1. subtrahera nyckeln och en stegförskjutning per index:
   `c = ((b - KEY - STEP*i) mod 256)`, normaliserat till `0..255`;
2. multiplicera med den modulära inversen: `out[i] = (c * MUL_INV) mod 256`.

Behandla utdata-byten som Latin-1-tecken — flaggan innehåller `ö`/`ä` (byte `0xF6`
m.fl.), så avkoda som Latin-1, inte strikt ASCII/enbart-UTF-8.

## 3. Återskapa lokalt

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

Kör det:

```bash
python3 decode.py
```

## Vad utdatan visar

Skriptet skriver ut flaggan i formatet `cratectf{...}` (den innehåller svenska tecken
som `ö`, vilket är varför Latin-1-avkodning spelar roll). Att klicka på sidans knapp
"Visa flaggan" kör exakt samma rutin i webbläsaren och visar samma sträng i det tidigare
dolda `display:none`-elementet.

## Verifiering

Plattformsbekräftat löst. Att Base64-avkoda den inline `OBFUSCATED`-strängen och
tillämpa `out = ((b - 0x2A - 7*i) mod 256) * 223 mod 256` per byte ger flaggan i
formatet `cratectf{...}` (oberoende reproducerad av flera lösare).
