# Det Finns Ett Hus I

## Source writeups

This translation reconciles:

- `crate-ctf-swarm/writeups/det-finns-ett-hus-i/ctf-agent11.md`
- `crate-ctf-swarm/writeups/det-finns-ett-hus-i/ctf-agent13.md`
- `crate-ctf-swarm/writeups/det-finns-ett-hus-i/ctf-agent21.md`

The sources conflict on location. Their differences are preserved below rather than merged into one unsupported conclusion.

## 1. Establish the challenge format

All three sources identify the `cratectf{XXXXXXX}` shape or a similar seven-character format.

The submitted inner answer is therefore a building name represented by exactly seven letters.

## 2. Compare the beach photograph

Use `strand.jpg` and identify:

- beach and promenade geometry;
- the row of colored cabins;
- palm placement;
- distant shoreline/headland geometry;
- the direction of the view;
- any building immediately visible at the first line.

Agent11 says the distant built-up headland and curved bay fit Platja de Sant Antoni in Cullera.

Agent11 explicitly rejects the straight beachfront at Hotel Neptuno in Valencia because that view does not show the same court strip and headland.

Agent21 instead says the image perspective is Valencia's Paseo de la Neptuno, Las Arenas/Malvarrosa. It cites Mediterranean beachfront geometry but gives no metadata source for the location beyond the photo and camera details.

## 3. Agent11: use aerial and OpenStreetMap evidence

Agent11 gives an ArcGIS World Imagery export URL with:

```text
bbox=-0.2418,39.157,-0.239,39.165
bboxSR=4326
size=450,1300
format=jpg
f=image
```

The source says to look for colored cabins near image coordinates `(98,474)`, with palms immediately south and a long landscaped promenade strip.

Agent11 then proposes OpenStreetMap way:

```text
https://www.openstreetmap.org/way/238656596
```

Its name is seven letters and the source proposes that the building name, uppercased, is the flag candidate.

## 4. Agent13: perform a systematic OSM extraction

Agent13 uses this OSM data bounding box:

```text
-0.2432,39.1605,-0.239,39.1627
```

The source writes the result to:

```text
/tmp/cullera.osm
```

The process:

1. download the OSM map response for the bounding box;
2. parse XML;
3. build a node map from node `id`, `lat`, and `lon`;
4. inspect each `way`;
5. collect its tags;
6. ignore unnamed ways;
7. collect coordinates for tagged nodes;
8. keep a name only if it has exactly seven characters and the `building` tag exists;
9. uppercase the name;
10. print it inside `cratectf{}`.

The exact source script concept is:

```python
#!/usr/bin/env python3
import urllib.request
import xml.etree.ElementTree as ET

bbox = "-0.2432,39.1605,-0.239,39.1627"
url = f"https://api.openstreetmap.org/api/0.6/map?bbox={bbox}"
out = "/tmp/cullera.osm"
urllib.request.urlretrieve(url, out)

root = ET.parse(out).getroot()
nodes = {
    n.attrib["id"]: (float(n.attrib["lat"]), float(n.attrib["lon"]))
    for n in root.findall("node")
}

candidates = []
for w in root.findall("way"):
    tags = {t.attrib["k"]: t.attrib["v"] for t in w.findall("tag")}
    name = tags.get("name", "")
    if not name:
        continue
    pts = [nodes[nd.attrib["ref"]] for nd in w.findall("nd") if nd.attrib["ref"] in nodes]
    if not pts:
        continue
    if len(name) == 7 and tags.get("building"):
        candidates.append(name.upper())

for c in sorted(set(candidates)):
    print(f"cratectf{{{c}}}")
```

The script returns the known candidate:

```text
ARENSOL
```

It is a reproducer for a seven-character building name in this bounding box. It does not establish that every output candidate is geographically correct.

## 5. Agent21: alternate Valencia candidate set

Agent21 identifies:

- Hotel Neptuno;
- Hotel Balandret;
- Apartaments Perlamar;
- Hotel Miramar.

It assigns an address and relative position to the candidate buildings along Paseo de la Neptuno.

Agent21 chooses Hotel Neptuno as the westernmost first-line property. It then formats the building name in uppercase.

This conflicts with agent11/agent13 because:

- it places the photograph in Valencia;
- it favors a 13-letter candidate name;
- its candidate exceeds the stated seven-character inner format.

The candidate is therefore not reconciled as valid in this document.

## Recovered result

There is no independently accepted answer.

Known conflicting candidates:

- `ARENSOL`
- `HOTEL NEPTUNO`

The literal flag is represented as:

**Placeholder:** `<CHALLENGE_FLAG>`

## Known unresolved conflict

- Agent11 and agent13 support Cullera/Platja de Sant Antoni and `Arensol`.
- Agent21 supports Valencia/Paseo de la Neptuno and Hotel Neptuno.
- The source writeups do not agree on the exact geographic location.
- Agent11 says exact balcony/platform acceptance has not been verified.
- No source provides a CTF-platform accepted submission.

## Platform-confirmed answer

`ARENSOL` (agents 11, 13) is **platform-confirmed correct** ("Correct but you
already solved this"). The format is `cratectf{XXXXXXX}` (7 letters, uppercase);
agent21's `HOTEL NEPTUNO` was wrong.
