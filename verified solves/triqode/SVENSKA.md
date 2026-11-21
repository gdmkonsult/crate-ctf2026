# Triqode — Detaljerad rekonstruktion

## Källor som lästs
- `writeups/triqode/ctf-agent11.md`
- `writeups/triqode/ctf-agent13.md`
- `writeups/triqode/ctf-agent21.md`

## Källfil
- `challenges/triqode/distfiles/data.csv`
- 1343 3D-punkter
- Koordinater i `[0,1]`
- Målgrid: `22 × 22 × 22`

## Steg 1 — Läs CSV
Exempel i första källan:

```python
src = np.loadtxt('challenges/triqode/distfiles/data.csv', delimiter=',')
```

Andra källan läser med `csv.reader` och kontrollerar att raderna har tre numeriska koordinater.

## Steg 2 — Kvantisera
Omvandla varje koordinat:

```text
index = round(21 × koordinat)
```

Kodformer i källorna:

```python
pts = np.rint(21 * src).astype(int)
points_int = np.round(points * 21).astype(int)
```

## Steg 3 — Bygg projektioner
Skapa 22×22-matriser för:

- xy: kolumn `(0,1)`;
- yz: kolumn `(1,2)`;
- xz: kolumn `(0,2)`.

Markera union av alla projektioner. Källan beskriver alltså inte en densitetssumma utan närvaro av minst en punkt i cellen.

## Steg 4 — Förbered bilder
Första källan lägger 4 pixel padding och gör 8× uppskalning. Andra källan använder 6 pixel padding och 8× uppskalning.

`yz` och `xz` kan behöva invertera ljus/mörker innan skanning, och alla tre kan behöva rotation/transponering. Quietyta och uppskalning gör symbolerna läsbara för barcodeavkodaren.

## Steg 5 — Avkoda
Använd `zxingcpp.read_barcodes()`/`zxing-cpp` och prova även rotationer.

Källorna anger:

- yz: Data Matrix
- xz: QR
- xy: Aztec

Fragment (literala värden redigerade — avkoda de tre koderna själv; varje
projektion ger en bit):

```text
yz = dots + <fragment 1 — inleder flaggan>
xz = <fragment 2 — mitten>
xy = <fragment 3 — avslutar flaggan> + dots
```

## Steg 6 — Konkatenera
Sammanfoga i ordning:

```text
yz + xz + xy
```

Ta bort paddingpunkter:

```python
combined = (yz_text + xz_text + xy_text).replace('.', '')
```

Literal värde publiceras inte. Använd `FLAG_PLACEHOLDER`.

## Luckor
Fyra kontra sex paddingpixlar och rotationsvägen skiljer sig. Råa avkodade flaggan och plattformsacceptans saknas.
