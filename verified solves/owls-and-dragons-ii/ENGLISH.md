# Owls and Dragons II — English

**Category:** Forensics · **Input:** `owl_king.png` (the Part I distfile; Part II ships no new files)

Part II has no new download. The "vault" to open is a JPEG hidden inside the Part I image,
and its `steghide` passphrase is the Cardan-grille phrase from Part I. Full chain:

```
owl_king.png (zTXt chunks) -> reconstructed SVG -> embedded PNG+PDF + hidden 7z (grille)
                                              \-> trailer after </svg>, XOR 0x13 -> grid.jpg (JPEG)
grille phrase = "TOYS FOR THE DRAGONS"  ==>  steghide passphrase TOYSFORTHEDRAGONS
steghide extract -sf grid.jpg -p TOYSFORTHEDRAGONS  ->  PNG containing the flag
```

## 1. Reconstruct the SVG from the PNG's `zTXt` chunks

`owl_king.png` carries six large `zTXt` chunks keyed `SVG0`…`SVG5` after the IDAT data. For
each: skip the PNG 8-byte signature, walk chunks by `length (4) + type (4) + data + CRC (4)`
(advance `12 + length`), keep chunks of type `zTXt` whose keyword starts with `SVG`, split at
the keyword NUL, skip the 1-byte compression-method flag, and `zlib.decompress` the rest.
Concatenate the six parts in keyword order → one ~5.7 MB SVG (`full.svg`).

```python
import struct, zlib, pathlib
png = pathlib.Path("owl_king.png").read_bytes()
assert png[:8] == b"\x89PNG\r\n\x1a\n"
i, parts = 8, []
while i < len(png):
    ln = struct.unpack(">I", png[i:i+4])[0]
    ctype = png[i+4:i+8]
    chunk = png[i+8:i+8+ln]
    i += 12 + ln
    if ctype == b"zTXt" and chunk.startswith(b"SVG"):
        k = chunk.find(b"\x00")
        parts.append(zlib.decompress(chunk[k+2:]))   # skip NUL + method byte
svg = b"".join(parts)
pathlib.Path("full.svg").write_bytes(svg)
```

## 2. Carve the three payloads out of the SVG

`full.svg` contains three things:

- **A base64 PNG** in a `data:...;base64,` URI. Decode it → `nested.png` (1024×768). A PDF is
  appended **after** that PNG's `IEND` (PNG IEND at offset 1488564; `%PDF` starts right
  after). Carve the PDF from the first `%PDF` marker → `nested.pdf` (~338 KB).
- **A hidden `<g id="huh" display="none">`** containing a base64 **7z** archive. Decode it →
  `archive.7z` (its headers are encrypted).
- **A trailer after `</svg>`** that is XOR-obfuscated with the single byte `0x13`. XOR it
  back → a JFIF **JPEG, 5333×4000** → save as `grid.jpg`. (The grille letters live in this
  image.)

```python
import re, base64, pathlib
svg = pathlib.Path("full.svg").read_bytes()

# (a) base64 PNG (+ appended PDF)
s = svg.index(b"base64,") + 7
e = re.compile(rb"[A-Za-z0-9+/=\n]*").match(svg, s).end()
blob = base64.b64decode(re.sub(rb"\s", b"", svg[s:e]) + b"==")
pathlib.Path("nested.png").write_bytes(blob)
pdf = blob[blob.index(b"%PDF"):]                      # PDF appended after PNG IEND
pathlib.Path("nested.pdf").write_bytes(pdf)

# (c) XOR-0x13 trailer after </svg>  ->  JPEG (grid.jpg)
t = svg[svg.index(b"</svg>") + 6:]
pathlib.Path("grid.jpg").write_bytes(bytes(c ^ 0x13 for c in t))
```

Note: `file` may misidentify the raw (pre-XOR) trailer as an "OpenPGP key"; that is a
false positive — after XOR 0x13 the magic bytes are `FF D8 FF E0` (JPEG).

## 3. Get the 7z password from the PDF, extract the grille

Extract the PDF's visible text — it is the 7z password `YHSAFNBTE`:

```bash
gs -q -dSAFER -dBATCH -dNOPAUSE -sDEVICE=txtwrite -sOutputFile=- nested.pdf
#   ->  YHSAFNBTE
7z x -pYHSAFNBTE archive.7z        # -> thing.png
```

`thing.png` is a 1024×768 grayscale+alpha **Cardan grille**: a mostly black mask with a few
transparent "holes" (alpha-255 bounding box ≈ x237–817, y292–450).

## 4. Read the grille phrase (= the steghide passphrase)

Overlay the grille holes onto the faint, tilted 5-row letter grid in `grid.jpg`:

1. Crop the grid region: box `(4400, 1800, 5200, 2400)`.
2. Autocontrast (cutoff 1) and rotate about **−29° to −29.5°** (expand) to level the grid.
3. Scale the grille's hole mask (nearest-neighbor) onto the registered grid box and read the
   letters visible through the holes, row by row in reading order.

The holes spell **`T O | Y S | F O R T H E | D R A | G O N S`** → **`TOYS FOR THE DRAGONS`**.
The passphrase (uppercase, no spaces) is:

```text
TOYSFORTHEDRAGONS
```

## 5. Open the steghide vault

The reconstructed JPEG `grid.jpg` is a `steghide` container; the grille phrase is its
passphrase:

```bash
steghide extract -sf grid.jpg -p TOYSFORTHEDRAGONS -xf steg_out.png
file steg_out.png        # PNG 274x272 RGBA
```

`steghide` exits 0 and writes `steg_out.png` (a 274×272 RGBA image of cat toys / kittens).
The image visibly contains the flag:

```text
cratectf{...}            # flag redacted
```

Open/zoom `steg_out.png` (or OCR it) to read the `cratectf{...}` flag text.

## Notes / dead ends

- Earlier attempts treated the grille phrase itself as the flag (e.g. `toysforthedragons`,
  with/without an extra "E"). That was the Part I answer; **Part II's answer is the text
  inside the steghide-extracted PNG**, not the grille phrase.
- Exact grille registration (rotation angle, hole scaling) is fiddly; verify the five rows
  independently if the phrase is unclear.

## Verification

Platform-confirmed solved (organizer log confirms the "dragons' favorite toys" phrase). The
`steghide extract` command above yields a PNG whose visible text is the real `cratectf{...}`
flag, redacted here for publication.
