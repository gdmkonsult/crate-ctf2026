# Triqode — Detailed reconstruction

## Sources read
- `writeups/triqode/ctf-agent11.md`
- `writeups/triqode/ctf-agent13.md`
- `writeups/triqode/ctf-agent21.md`

## Source artifact
- Data file: `challenges/triqode/distfiles/data.csv`
- Rows: 1343 normalized 3D points
- Coordinate interval: `[0,1]`
- Grid after quantization: `22 × 22 × 22`
- Output: one Aztec, one Data Matrix and one QR code across orthogonal projections

## Step 1 — Parse the CSV
Use a CSV or NumPy reader to obtain one `(x,y,z)` row per line:

```bash
python3
```

One source loads the file by:

```python
src = np.loadtxt('challenges/triqode/distfiles/data.csv', delimiter=',')
```

The verification source uses Python's `csv.reader` and explicitly accepts only rows that contain three numeric fields. All sources agree on 1343 points.

## Step 2 — Quantize coordinates
Scale each coordinate from `[0,1]` to an integer grid index:

```text
0 through 21 = round(21 × coordinate)
```

The primary solver commands and parameter are:

```python
pts = np.rint(21 * src).astype(int)
```

The verification source uses:

```python
points_int = np.round(points * 21).astype(int)
```

Both are equivalent rounding operations apart from implementation details. Do not assume a grid wider or narrower than 22: the lower bound is `0` and upper bound is `21`.

## Step 3 — Build three projections
Build 22×22 occupancy matrices:

- xy: project points using axes `(0,1)`;
- yz: project points using axes `(1,2)`;
- xz: project points using axes `(0,2)`.

Mark a cell as present if at least one point projects to that coordinate. All sources agree this union-operation creates the barcode images.

The exact occupancy examples differ in implementation but not in geometry:

```python
p = np.zeros((22, 22), dtype=bool)
p[pts[:, b], pts[:, a]] = 1
```

or

```python
grid[idx0, idx1] = 255
```

## Step 4 — Prepare the images for scanning
The first source adds 4-pixel padding, then 8× upscaling:

```text
22×22 grid
+ 4 pixels on each side
repeat each pixel 8 times in each dimension
```

The verification source uses 6-pixel padding and 8× upscaling. It is explicit that the present area is white and absent area is black before the final inversion variants.

The primary solver inverts `yz` and `xz` occupancy before rendering and uses `~img`/255 conversion. The sources explain this is necessary because the barcode decoder depends on the exact light/dark convention. For reproducibility, try the original and inverse image plus `np.rot90()` variants.

## Step 5 — Decode each projection
Use `zxingcpp.read_barcodes()` or `zxing-cpp`.

All source examples report:

- `yz`: Data Matrix;
- `xz`: QR, potentially after transposition or rotation;
- `xy`: Aztec.

Recorded decoded fragments (literal fragment text redacted — decode the three
codes yourself; each projection yields one piece):

```text
yz: dots, then <fragment 1 — begins the flag>
xz: <fragment 2 — middle>
xy: <fragment 3 — ends the flag>, then dots
```

The source also differs in the padding text:

- `ctf-agent11` writes the first fragment preceded by dot padding;
- `ctf-agent13` and `ctf-agent21` use dot padding conceptually and then strip dots;
- `ctf-agent13` states xz can require `rot90`.

## Step 6 — Concatenate the fragments
Use the order:

```text
yz text + xz text + xy text
```

Then remove padding dots:

```python
combined = (yz_text + xz_text + xy_text).replace('.', '')
```

The result is a `cratectf{...}`-shaped challenge value. Replace the literal value with `FLAG_PLACEHOLDER`.

## Source gaps
- The writeups do not preserve the literal decoded flag or all raw barcode images.
- Two sources use 4-pixel and 6-pixel quiet zones; the exact successful crop and contrast settings are not identical.
- OpenCV is mentioned in one source only as a limitation for QR and zxing-cpp as the more complete decoder.
- No platform acceptance evidence is included.
