# Vandraren — Detaljerad rekonstruktion

## Källor som lästs
- `writeups/vandraren/ctf-agent16.md`
- `writeups/vandraren/ctf-agent20.md`

## Fråga och artefakt
- Frågan är: “Vandraren är hungrig, var är han påväg?”
- Video: `vandraren.mp4`
- Varaktighet: ungefär 20 sekunder
- Båda källorna använder den laminerade utskrift med vandringsrutt som syns mot slutet.

## Steg 1 — Extrahera bilder utan att minska användbar upplösning
`ctf-agent16` anger exakt:

```bash
ffmpeg -ss 17.5 -i challenges/vandraren/distfiles/vandraren.mp4 -vf 'crop=850:850:430:190' -frames:v 1 kort.png
```

- `-ss 17.5` söker till 17,5 sekunder.
- `crop=850:850:430:190` tar 850×850 pixlar från position `430:190`.
- `-frames:v 1` skriver en bild.
- `kort.png` är utfilnamnet.

`ctf-agent20` extraherar ett intervall:

```bash
ffmpeg -ss 16.8 -i vandraren.mp4 -t 1.2 -vf "fps=5,crop=700:850:500:150" q_%02d.png
```

- start: 16,8 sekunder;
- längd: 1,2 sekunder;
- fem bildrutor per sekund;
- crop: 700×850 vid `500:150`;
- utdata: `q_01.png` och följande nummer.

Båda intervallen täcker utskriften runt 17 sekunder.

## Steg 2 — Läs utskriften
Utskriften är en satellitbild med en blå vandringslinje. Den är inte norrorienterad. Källorna identifierar följande terräng:

- jordbruksfält i startområdet;
- skog längs rutten;
- en långsträckt sjö;
- en rak kraftledningsgata längs sjön;
- byggnader vid sjöstranden.

Jämför alla ledtrådar mot utskriftens geometri; identifiera inte platsen endast utifrån ett sjönamn.

## Steg 3 — Identifiering enligt källorna
`ctf-agent16` jämför utsikten och lunchserveringen med Lidas egen webbplats. Källan kopplar samman restaurang, lunchservering och utsikt över Getaren. Namnet ska representeras med understreck i stället för mellanslag och behålla svensk `ä`.

`ctf-agent20` använder en bredare sökning:

1. Extrahera svenska serveringsställen från Geofabriks `sweden-latest.osm.pbf`.
2. Tolka objekt med `pyosmium`.
3. Filtrera namn som följer 4+8-bokstavsmönster.
4. Jämför cirka 164 kandidatnamn mot terrängen.
5. Identifiera Lida Värdshus på `59.16292, 17.87958` i Botkyrka.

Andra källan anger att platsen matchar byggnaderna vid stranden, den långa sjön, kraftledningsgatan och jordbruksfälten vid startpunkten.

## Steg 4 — Slutnamn och flaggformat
Båda källorna konvergerar mot Lida/Lida friluftsområde och Getaren eller Lida Värdshus. Ingen av källorna publicerar den bokstavliga flaggan.

Använd `FLAG_PLACEHOLDER` i stället för något utmaningsberoende hemligt svar.

## Källuckor
- Första källan använder främst Lidas webbplats; andra använder OSM och namnmönster.
- `ctf-agent16` anger uttryckligen att kandidaten inte provades på tävlingsplattformen.
- Ingen ger bokstavlig verifierad flagga eller plattformsacceptans.
