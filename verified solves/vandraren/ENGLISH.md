# Vandraren — Detailed reconstruction

## Sources read
- `writeups/vandraren/ctf-agent16.md`
- `writeups/vandraren/ctf-agent20.md`

## Prompt and artifact
- Prompt: “Vandraren är hungrig, var är han påväg?”
- Video: `vandraren.mp4`
- Duration: roughly 20 seconds
- The only useful clue in both sources is a laminated walking-route print held up near the end of the video.

## Step 1 — Extract frames without losing useful resolution
`ctf-agent16` provides an exact `ffmpeg` crop command:

```bash
ffmpeg -ss 17.5 -i challenges/vandraren/distfiles/vandraren.mp4   -vf 'crop=850:850:430:190' -frames:v 1 kort.png
```

Interpretation:

- `-ss 17.5` seeks to 17.5 seconds;
- `-i` supplies the source video;
- `crop=850:850:430:190` means an 850×850 crop at position `430:190`;
- `-frames:v 1` writes one frame;
- `kort.png` is the output name.

`ctf-agent20` instead extracts a range:

```bash
ffmpeg -ss 16.8 -i vandraren.mp4 -t 1.2   -vf "fps=5,crop=700:850:500:150" q_%02d.png
```

Interpretation:

- start at 16.8 seconds;
- duration `1.2` seconds;
- 5 frames per second;
- 700×850 crop at position `500:150`;
- output sequential files `q_01.png`, `q_02.png`, etc.

The source ranges overlap and both aim at the 17-second map view.

## Step 2 — Inspect the printout
The print is a satellite image with a blue walking route. It is not north-up.

Terrain clues in the route:

- the route begins near or through farmland;
- it passes through forest;
- it follows or crosses an elongated lake;
- a straight power-line clearing is visible beside the lake;
- buildings are visible on the lake shore.

Do not infer a place solely from a lake name or route endpoint; compare all terrain features to the print.

## Step 3 — Search and identify by source
`ctf-agent16` identifies Lida's own service area. It describes a restaurant, lunch service and view over Getaren. The business must be written with an underscore for a space and preserve Swedish `ä`.

The source candidate name is therefore based on:

```text
Lida friluftsområde
```

and it associates the destination with the restaurant/Lida Värdshus at Getaren.

`ctf-agent20` performs broader candidate filtering:

1. Extract Swedish food-place names from OpenStreetMap Geofabrik data `sweden-latest.osm.pbf`.
2. Parse objects with `pyosmium`.
3. Filter names fitting a four-letter-word plus eight-letter-word pattern.
4. Compare the 164 candidate names against satellite terrain, not only distance to lakes.
5. Identify Lida Värdshus at `59.16292, 17.87958` in Botkyrka.

The second source also says this specific location matches:

- shore buildings;
- the elongated lake;
- the diagonal/straight power-line clearing;
- farmland at the route start.

## Step 4 — Resolve name format
The sources converge on Lida/Lida friluftsområde and Getaren or Lida Värdshus. The exact flag format is not reproduced, and neither source includes its final literal value.

Represent the unresolved answer as:

```text
FLAG_PLACEHOLDER
```

## Source gaps
- The sources differ in how they search: one uses the venue's own website, while the other uses OSM and a 4+8 name filter.
- `ctf-agent16` explicitly says its candidate was not tested on the CTF platform.
- Neither source publishes a verified literal flag.
