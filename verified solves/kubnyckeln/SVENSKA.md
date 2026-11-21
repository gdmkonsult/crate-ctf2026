# Kubnyckeln — svenska

**Granskade källor:** `ctf-agent11.md`, `ctf-agent13.md` och `ctf-agent21.md`.

## Lösningsmetod steg för steg

1. Läs `challenges/kubnyckeln/distfiles/nyckel.txt` och `challenges/kubnyckeln/distfiles/chiffertext.txt`. Agent 13 läser nyckeln som nio rader, där varje rad matchas som `### <A-Z eller @> ### <A-Z eller @> ### <A-Z eller @> ###`; det förväntade totalantalet är 27 symboler. Agent 21 skriver av rutorna så här:
   - BOX1: `B O X / R A D / K L U`
   - BOX2: `M N C / E F G / H I J`
   - BOX3: `P Q S / T V W / Z Y @`
   
   Agent 13 säger att filen innehåller tre 3×3-rutor och den 27:e symbolen `@`, med symbolordning ruta-för-ruta och sedan rad-för-rad. Chiffertexten är `QEBAWJEAHE`.
2. Tilldela enbaserade koordinater `(ruta, rad, kolumn)` genom att räkna rutor 1–3, rader 1–3 och kolumner 1–3. Slå upp varje av de tio chiffertextbokstäverna i nyckelkuben. Agent 13 behandlar alla tio bokstäver som en Trifid-period (`n = 10`).
3. Följ Agent 13:s dekrypteringsmetod: lägg varje chifferbokstavs tre koordinater i en platt lista och dela sedan de 30 värdena i `a = flat[:10]`, `b = flat[10:20]` och `c = flat[20:30]`. Klartextkoordinaten på position `i` är `(a[i], b[i], c[i])`; slå upp varje sådan trippel i omvända koordinatkartan för att få klartexttecken.
4. Standardalfabetkuben som används vid avkodningen är:
   - BOX1: `A B C / D E F / G H I`
   - BOX2: `J K L / M N O / P Q R`
   - BOX3: `S T U / V W X / Y Z @`
5. För Agent 13:s rundturstest mappas varje klartextsymbol tillbaka till sin `(ruta,rad,kolumn)`-trippel; samla först alla rutvärden, därefter alla radvärden och sedan alla kolumnvärden. Läs listans 30 värden i på varandra följande tripplar, slå upp dem i samma omvända kub och jämför resultatet med `QEBAWJEAHE`. Agent 13 rapporterar `roundtrip ok: True`; dess avkodade token är `<KLARTEXTTOKEN REDIGERAD>`.
6. Agent 21 beskriver en annan metod: för varje chiffertecken skapas tre listor (`layers`, `rows`, `cols`), listorna används som en 3×10-matris och varje kolumn läses som en trippel i standardkuben. Källan anger ett mellanresultat på tio tecken och använder de första nio som kandidat. Det bokstavliga mellanresultatet och kandidaten undanhålls eftersom källornas resultat skiljer sig.

## Motsägelser och olöst resultat

De tre källorna är inte överens om klartexten. Agent 11 anger ett resultat med nio versaler och säger uttryckligen att skiftläget är obekräftat. Agent 13 anger en token med nio tecken och en lyckad rundtur men publicerar inte token. Agent 21 anger en annan avkodning med tio tecken och säger att dess första nio ska användas; den beskrivna koordinathanteringen skiljer sig från Agent 13:s uppdelning av den platta listan. Klartexten, flaggan och eventuella kandidatbokstäver undanhålls, och källorna innehåller inte tillräckligt samstämmiga belägg för att lösa motsägelsen. Inget påstående görs om godkännande på plattformen.

**Resultat:** `<FLAGGA REDIGERAD; EXAKT KANDIDAT OLÖST>` — källorna säger att det härleds genom att dekryptera `QEBAWJEAHE` med kuben, men de är oense om klartexten.

## Plattformsbekräftat svar

`TRIFIDKLAR` (versaler, agent 11, 13) är **plattformsbekräftat korrekt** ("Correct
but you already solved this"). Agent21:s `TMAEXRMEP` var fel.
