# Tidpunkt — Detaljerad rekonstruktion

## Källor som lästs
- `writeups/tidpunkt/ctf-agent11.md`
- `writeups/tidpunkt/ctf-agent21.md`

## Olösta motsägelse
En källa identifierar Peabprojekt i Djurgården, Linköping. Den andra identifierar Villa Österlen och Note Design Studio i Österlen, Skåne. Företag/plats är därför inte entydigt löst.

## Steg 1 — Extrahera EXIF
Båda använder bildfilen:

```text
challenges/tidpunkt/distfiles/image.jpg
```

Första källan:

```bash
exiftool -DateTimeOriginal challenges/tidpunkt/distfiles/image.jpg
```

Andra källan:

```bash
exiftool -a challenges/tidpunkt/distfiles/image.jpg | grep -i date
```

Gemensamt datum: `2026:07:30`. Klocktid: `xx:xx:xx` eller okänd. Helt datum är därför `2026-07-30`, inte en verifierad timmes.

## Steg 2 — Första källans metod
1. Matcha JYSK- och Nordic Wellness-skyltarna.
2. Sök `Åsmestadsgatan 2`, `Åsmestadsgatan 6`, `Kv Jägarsoldaten i stadsdelen Djurgården i Linköping` och `Peab bygger hyreslägenheter och vårdcentral i Linköping`.
3. Identifiera Peab/Stångåstaden-projektet med 176 bostäder i första etappen.
4. Använd nordorienterad satellitbild centrerad på `58.3859044,15.5888956`.
5. Spåra korridoren mot JYSK, källa anger bäring cirka `155°`.
6. Tolka solen som något till höger om korridoren.
7. Jämför solpositionerna:

```text
12:00 CEST: 156.7°
12:30 CEST: 167.4°
13:00 CEST: 178.4°
```

8. Källan säger att observationen passar 12:00 CEST.

## Steg 3 — Andra källans metod
1. Sök `Villa Österlen` i Dezeen, ArchDaily och Architonic.
2. Identifiera Villa Österlen i Österlen, Skåne.
3. Ange Note Design Studio som arkitekt/företag och Erik Lefvander som fotograf.
4. Analysera ljus med PIL/NumPy:

```text
medel 128.7
himmel 170
mark 100
```

5. Tolkningen är att himlen är 70 % ljusare och solståndet indikerar middag.
6. Skrivningen uppskattar 12:00 men nämner även 13:00 och 14:00 som alternativ.

## Steg 4 — Utvärdera konflikt
Källorna pekar inte på samma plats eller företag. Agent21:s Österlen/Note Design
Studio-svar är fel.

## Utgång
Operatören bekräftade rätt svar: företag **Peab**, datum **2026-07-30**, timme **12**
(Linköping/Stångåstaden, agent11). Viktig detalj: företaget skrivs med **gemener,
`peab`**, i flaggan (format `cratectf{<företag>_<yyyy-mm-dd>_<hh>}` → `peab` /
`2026-07-30` / `12`), trots att firman normalt skrivs "Peab". Den wrappade literala
flaggan återges inte här; använd gemener.
