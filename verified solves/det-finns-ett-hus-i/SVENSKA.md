# Det Finns Ett Hus I

## Källskrivningar

Denna översättning sammanväger:

- `crate-ctf-swarm/writeups/det-finns-ett-hus-i/ctf-agent11.md`
- `crate-ctf-swarm/writeups/det-finns-ett-hus-i/ctf-agent13.md`
- `crate-ctf-swarm/writeups/det-finns-ett-hus-i/ctf-agent21.md`

Källorna är oense om platsen. Skillnaden bevaras nedan i stället för att blandas till en slutsats som saknar stöd.

## 1. Fastställ utmaningsformatet

Alla tre källorna anger formatet `cratectf{XXXXXXX}` eller ett liknande sju-tecken-format.

Det inskickade svarsvärdet är därför ett byggnadsnamn representerat av exakt sju bokstäver.

## 2. Jämför strandfotografiet

Använd `strand.jpg` och identifiera:

- strand- och promenadgeometri;
- raden av färgade strandhus;
- placeringen av palmträd;
- avlägsen strand/uddgeometri;
- synhållningens riktning;
- eventuell byggnad direkt vid först linjen.

Agent11 säger att den avlägsna bebyggda udden och den böjda viken passar Platja de Sant Antoni i Cullera.

Agent11 avfärdar uttryckligen den raka strandlinjen vid Hotel Neptuno i Valencia eftersom den vyn inte visar samma sport-/planremsa och udde.

Agent21 säger i stället att bildvinkeln är Valencia/Paseo de la Neptuno, Las Arenas/Malvarrosa. Den hänvisar till Medelhavets strandgeometri men anger ingen metadataförankring bortom foto och kameradetaljer.

## 3. Agent11: använd flygbild och OpenStreetMap

Agent11 anger en ArcGIS World Imagery-export-URL med:

```text
bbox=-0.2418,39.157,-0.239,39.165
bboxSR=4326
size=450,1300
format=jpg
f=image
```

Källan säger att leta efter färgade strandhus kring bildkoordinaten `(98,474)`, med palmer omedelbart söderut och en lång, trädplanterad promenadremsa.

Agent11 föreslår därefter OpenStreetMap-way:

```text
https://www.openstreetmap.org/way/238656596
```

Dess namn har sju bokstäver, och källan föreslår att byggnadsnamnet i versaler är flaggkandidaten.

## 4. Agent13: utför en systematisk OSM-extraktion

Agent13 använder denna OSM-bounding-box:

```text
-0.2432,39.1605,-0.239,39.1627
```

Källan skriver resultatet till:

```text
/tmp/cullera.osm
```

Arbetsgången är:

1. hämta OSM-kartsvaret för bounding boxen;
2. tolka XML;
3. bygg en nodkarta från node `id`, `lat` och `lon`;
4. inspektera varje `way`;
5. samla dess taggar;
6. ignorera namnlösa way;
7. samla koordinater för taggade noder;
8. behåll namnet endast om det har exakt sju tecken och `building`-taggen finns;
9. gör namnet till versaler;
10. skriv ut det inom `cratectf{}`.

Det exakta källskriptets koncept är:

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

Skriptet returnerar den kända kandidaten:

```text
ARENSOL
```

Detta reproducerar ett sju-tecken-bygga namn inom bounding boxen. Det visar inte att alla utskrivna kandidater är geografiskt korrekta.

## 5. Agent21: alternativ Valenciakandidat

Agent21 identifierar:

- Hotel Neptuno;
- Hotel Balandret;
- Apartaments Perlamar;
- Hotel Miramar.

Den kopplar adress och relativ position till byggnaderna längs Paseo de la Neptuno.

Agent21 väljer Hotel Neptuno som den västligaste byggnaden vid strandlinjen. Därefter görs byggnadsnamnet till versaler.

Detta strider mot agent11/agent13 eftersom:

- det placerar fotografiet i Valencia;
- det favoriserar ett 13-teckens kandidatnamn;
- kandidaten överskrider det angivna formatet med sju inre tecken.

Kandidaten anses därför inte bekräftad i detta dokument.

## Återvunnet resultat

Ingen självständigt godkänd lösning finns.

Kända motstridiga kandidater:

- `ARENSOL`
- `HOTEL NEPTUNO`

Den bokstavliga flaggan representeras som:

**Platshållare:** `<UTMANINGSFLAGGA>`

## Kända olösta konflikter

- Agent11 och agent13 stödjer Cullera/Platja de Sant Antoni och `Arensol`.
- Agent21 stödjer Valencia/Paseo de la Neptuno och Hotel Neptuno.
- Skrivningarna är inte överens om fotografiets exakta geografiska plats.
- Agent11 säger att exakt balkong-/plattformsgodkännande inte är verifierat.
- Ingen källa ger en CTF-plattformsgodkänd inlämning.

## Plattformsbekräftat svar

`ARENSOL` (agent 11, 13) är **plattformsbekräftat korrekt** ("Correct but you
already solved this"). Formatet är `cratectf{XXXXXXX}` (7 versaler); agent21:s
`HOTEL NEPTUNO` var fel.
