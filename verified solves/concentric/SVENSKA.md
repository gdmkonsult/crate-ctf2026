# Concentric — svenska

**Kategori:** Kryptografi. **Uppgift:** "Vad står det i cirklarna?"
**Status:** Löst och bekräftad (laget postade hela writeupen; organisatören godkände).
Flaggan är redigerad här; den är Dante/Malebolge-frasen i gemener, bekräftad på plattformen.

## Översikt

Bilden `challenge.png` består av sex cirkulära symboler som ser ut som
labyrinter. Det är de inte. Varje svart ring är en bokstav, och ringarna läses
inifrån och ut. Titeln "Concentric" (koncentriska cirklar) är ledtråden.

## Återvändsgränder (dokumenterade så de inte upprepas)

- **Steganografi:** PNG-filen har exakt tre färger (vit, ljusgrå, mörkgrå) och
  ingen metadata. Inget är gömt i filens byte.
- **Labyrintlösning:** att söka en väg från mitten och ut ger ingenting.
- **Bitar per ring:** bågarna börjar och slutar på multiplar av 45°, så det är
  frestande att läsa 8 bitar per ring som ASCII — men det ger bara skräp.

## Struktur

- Varje symbol har eget centrum. De ljusgrå hjälplinjerna visar centrumen vid
  `x = 162, 487, 782, 1032, 1312, 1622`, alla vid `y = 177`.
- Ringarna ligger på radierna `10, 25, 40, 55, …` med 15 px mellanrum.
- En bokstav består av:
  - en båge på ringen som täcker vissa 45°-sektorer, och
  - radiella streck inåt från ringen vid 0°, 90°, 180° eller 270°.

Genom att "rulla ut" varje ring för sig och rita dess inåtgående streck blir
bokstäverna läsbara (lösarens `render_rings.py` ger `rings.png`).

## Alfabetet (ringform + inåtstreck → bokstav)

| Ringform | Inåtstreck | Bokstav |
|---|---|---|
| vänsterhalva `(` | — | C |
| vänsterhalva `(` | vid 9 | E |
| högerhalva `)` | — | D |
| högerhalva `)` | vid 3 | F |
| högerhalva `)` | vid 12 och 6 | B |
| `∩` (öppen nedåt) | vid 3 och 9 | A |
| `∩` med streck från mitten | vid 12 | M |
| öppen nere till höger | vid 3 och 6 | R |
| öppen uppe till höger | — | L |
| öppen uppe till höger | vid 3 | G |
| båge upptill + nedtill | — | T |
| båge upptill + nedtill | vid 12 och 6 | I |
| båge vänster + höger | vid 3 och 9 | H |
| underhalva `∪` | — | U |
| hel ring | — | O |
| två motstående kvartar | vid 3 och 9 | S |
| rak vågrät korda | — | `_` |
| rak lodrät korda (vänster/höger) | — | `{` / `}` |

Kordorna ligger på samma radieavstånd som en ring och fungerar därför som egna
"ringar" i läsordningen.

## Avkodning (inifrån och ut)

| Symbol | Ringar | Text |
|---|---|---|
| 1 | C R A T E C T F + lodrät korda | `cratectf{` |
| 2 | M A L E B O L G E + korda | `malebolge_` |
| 3 | E I G H T H + korda | `eighth_` |
| 4 | C I R C L E + korda | `circle_` |
| 5 | F R A U D + korda + T H E | `fraud_the` |
| 6 | korda + B O L G I A S + lodrät korda | `_bolgias}` |

Temat bekräftar läsningen: i Dantes Inferno är Malebolge helvetets åttonde
cirkel, där bedrägeri (fraud) straffas, indelad i tio bolgias. Skiftläget syns
inte i bilden; flaggformatet är gemener.

## Resultat

Flaggan är redigerad (gemener `cratectf{malebolge_...bolgias}`). Bekräftad
godkänd på plattformen.
