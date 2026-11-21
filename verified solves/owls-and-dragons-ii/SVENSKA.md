# Owls and Dragons II — Svenska

**Kategori:** Forensik · **Indata:** `owl_king.png` (distfilen från Del I; Del II har inga nya filer)

Del II har ingen ny nedladdning. "Valvet" som ska öppnas är en JPEG gömd inuti Del I-bilden,
och dess `steghide`-lösenfras är Cardan-grille-frasen från Del I. Hela kedjan:

```
owl_king.png (zTXt-chunkar) -> rekonstruerad SVG -> inbäddad PNG+PDF + gömd 7z (grille)
                                              \-> trailer efter </svg>, XOR 0x13 -> grid.jpg (JPEG)
grille-fras = "TOYS FOR THE DRAGONS"  ==>  steghide-lösenfras TOYSFORTHEDRAGONS
steghide extract -sf grid.jpg -p TOYSFORTHEDRAGONS  ->  PNG som innehåller flaggan
```

## 1. Rekonstruera SVG:n ur PNG:ens `zTXt`-chunkar

`owl_king.png` bär sex stora `zTXt`-chunkar med nycklarna `SVG0`…`SVG5` efter IDAT-datan. För
varje: hoppa över PNG:ens 8-byte-signatur, gå igenom chunkar enligt
`längd (4) + typ (4) + data + CRC (4)` (stega `12 + längd`), behåll chunkar av typ `zTXt`
vars nyckel börjar med `SVG`, dela vid nyckel-NUL, hoppa över 1-byte-flaggan för
komprimeringsmetod och kör `zlib.decompress` på resten. Sammanfoga de sex delarna i
nyckelordning → en ~5,7 MB SVG (`full.svg`).

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
        parts.append(zlib.decompress(chunk[k+2:]))   # hoppa NUL + metodbyte
svg = b"".join(parts)
pathlib.Path("full.svg").write_bytes(svg)
```

## 2. Carva ut de tre payloaderna ur SVG:n

`full.svg` innehåller tre saker:

- **En base64-PNG** i en `data:...;base64,`-URI. Avkoda den → `nested.png` (1024×768). En PDF
  är tillagd **efter** den PNG:ens `IEND` (PNG IEND på offset 1488564; `%PDF` börjar direkt
  efter). Carva PDF:en från första `%PDF`-markören → `nested.pdf` (~338 KB).
- **En gömd `<g id="huh" display="none">`** som innehåller ett base64-kodat **7z**-arkiv.
  Avkoda den → `archive.7z` (dess headers är krypterade).
- **En trailer efter `</svg>`** som är XOR-obfuskerad med den enda byten `0x13`. XOR:a tillbaka
  → en JFIF-**JPEG, 5333×4000** → spara som `grid.jpg`. (Grille-bokstäverna finns i den här
  bilden.)

```python
import re, base64, pathlib
svg = pathlib.Path("full.svg").read_bytes()

# (a) base64-PNG (+ tillagd PDF)
s = svg.index(b"base64,") + 7
e = re.compile(rb"[A-Za-z0-9+/=\n]*").match(svg, s).end()
blob = base64.b64decode(re.sub(rb"\s", b"", svg[s:e]) + b"==")
pathlib.Path("nested.png").write_bytes(blob)
pdf = blob[blob.index(b"%PDF"):]                      # PDF tillagd efter PNG IEND
pathlib.Path("nested.pdf").write_bytes(pdf)

# (c) XOR-0x13-trailer efter </svg>  ->  JPEG (grid.jpg)
t = svg[svg.index(b"</svg>") + 6:]
pathlib.Path("grid.jpg").write_bytes(bytes(c ^ 0x13 for c in t))
```

Obs: `file` kan feltolka den råa (före-XOR) trailern som en "OpenPGP key"; det är en falsk
positiv — efter XOR 0x13 är magic-byten `FF D8 FF E0` (JPEG).

## 3. Hämta 7z-lösenordet ur PDF:en, extrahera grillen

Extrahera PDF:ens synliga text — det är 7z-lösenordet `YHSAFNBTE`:

```bash
gs -q -dSAFER -dBATCH -dNOPAUSE -sDEVICE=txtwrite -sOutputFile=- nested.pdf
#   ->  YHSAFNBTE
7z x -pYHSAFNBTE archive.7z        # -> thing.png
```

`thing.png` är en 1024×768 gråskala+alfa **Cardan-grille**: en mestadels svart mask med några
genomskinliga "hål" (alfa-255-begränsningsbox ≈ x237–817, y292–450).

## 4. Läs grille-frasen (= steghide-lösenfrasen)

Lägg grille-hålen över det svaga, lutande 5-radiga bokstavsrutnätet i `grid.jpg`:

1. Beskär rutnätsregionen: box `(4400, 1800, 5200, 2400)`.
2. Autokontrast (cutoff 1) och rotera cirka **−29° till −29,5°** (expand) för att räta upp
   rutnätet.
3. Skala grillens hålmask (nearest-neighbor) över den inpassade rutnätsboxen och läs
   bokstäverna som syns genom hålen, rad för rad i läsordning.

Hålen stavar **`T O | Y S | F O R T H E | D R A | G O N S`** → **`TOYS FOR THE DRAGONS`**.
Lösenfrasen (versaler, inga mellanslag) är:

```text
TOYSFORTHEDRAGONS
```

## 5. Öppna steghide-valvet

Den rekonstruerade JPEG:en `grid.jpg` är en `steghide`-behållare; grille-frasen är dess
lösenfras:

```bash
steghide extract -sf grid.jpg -p TOYSFORTHEDRAGONS -xf steg_out.png
file steg_out.png        # PNG 274x272 RGBA
```

`steghide` avslutar med 0 och skriver `steg_out.png` (en 274×272 RGBA-bild med kattleksaker /
kattungar). Bilden innehåller synligt flaggan:

```text
cratectf{...}            # flagga dold
```

Öppna/zooma `steg_out.png` (eller OCR:a den) för att läsa `cratectf{...}`-flaggtexten.

## Anteckningar / återvändsgränder

- Tidigare försök behandlade grille-frasen i sig som flaggan (t.ex. `toysforthedragons`,
  med/utan ett extra "E"). Det var Del I:s svar; **Del II:s svar är texten inuti den
  steghide-extraherade PNG:en**, inte grille-frasen.
- Exakt grille-inpassning (rotationsvinkel, hålskalning) är petig; verifiera de fem raderna
  oberoende om frasen är otydlig.

## Verifiering

Plattformsbekräftat löst (arrangörens logg bekräftar frasen om "drakarnas favoritleksaker").
`steghide extract`-kommandot ovan ger en PNG vars synliga text är den riktiga
`cratectf{...}`-flaggan, dold här för publicering.
