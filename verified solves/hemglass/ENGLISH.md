# Hemglass — English

**Category:** Web / Reversing · **Endpoint:** `http://hemglass.challs.crate.nu/verify`

The challenge is a web-based music box that emulates the classic Hemglass ice-cream-truck
jingle hardware (a MOS 6581 "SID" sound chip driven by a Z80). `POST /verify` accepts a
single melody string and returns the flag only if the melody exactly matches the canonical
Hemglass tune.

## 1. Discover the melody source (OSINT)

1. The melody string format is `<duration><note><octave>` per note (e.g. `8D4`), space
   separated. You need the exact canonical note sequence and timing.
2. Check the recent edit history (November 2026) of the Swedish Wikipedia article for
   *Hemglass*. A recently added reference points to the Swedish Commodore Club's
   reconstruction: `https://www.commodore.se/hemglassmelodin-en-rekonstruktion/`.
3. That article documents the player hardware (Z80 + SID 6581 + 2 kB EPROM) and links a
   source archive `hemglass_mjukvara_z80.zip`. Download and unzip it.

## 2. Read the melody out of the disassembly

Open the disassembled `hemglass.asm` from the archive. The melody section contains exactly
16 notes, each referencing one of three loop-delay constants with the ratio
`DELAY_SHORT : DELAY_LONG : DELAY_VLONG = 1 : 2 : 3`.

| # | Pitch | Delay class |
|---|-------|-------------|
| note01 | D4 | SHORT |
| note02 | B4 | SHORT |
| note03 | D5 | SHORT |
| note04 | B4 | SHORT |
| note05 | G4 | LONG |
| note06 | D4 | SHORT |
| note07 | B4 | SHORT |
| note08 | D5 | SHORT |
| note09 | B4 | SHORT |
| note10 | G4 | LONG |
| note11 | D4 | SHORT |
| note12 | C5 | LONG |
| note13 | C5 | SHORT |
| note14 | A4 | LONG |
| note15 | A4 | SHORT |
| note16 | G4 | VLONG |

## 3. Map delay classes to the web interface's duration notation

The web app uses standard note-value numbers where a larger number is a shorter note:

- `SHORT`  → `8`  (eighth note)
- `LONG`   → `4`  (quarter note = 2× eighth)
- `VLONG`  → `4.` (dotted quarter = 3× eighth)

Write each note as `duration` immediately followed by `note` and `octave`, space-separated.
The resulting canonical melody string is:

```text
8D4 8B4 8D5 8B4 4G4 8D4 8B4 8D5 8B4 4G4 8D4 4C5 8C5 4A4 8A4 4.G4
```

## 4. Submit to `/verify`

Send the string as JSON field `melody` with `Content-Type: application/json`:

```bash
curl -s -X POST http://hemglass.challs.crate.nu/verify \
  -H 'Content-Type: application/json' \
  -d '{"melody":"8D4 8B4 8D5 8B4 4G4 8D4 8B4 8D5 8B4 4G4 8D4 4C5 8C5 4A4 8A4 4.G4"}'
```

The service responds:

```json
{"ok": true, "flag": "cratectf{...}"}
```

The flag is the value of the `flag` field (redacted here — a human running the command
above obtains it directly).

## Verification

Platform-confirmed solved. The `curl` request above returns `{"ok": true, "flag": ...}`
with the real flag; the literal value is redacted for publication.
