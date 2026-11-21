# Owls and Dragons I — svenska

**Granskade källor:** `ctf-agent11.md`, `ctf-agent20.md` och `ctf-agent21.md`.

## Extraktion och analys steg för steg

1. Börja med `challenges/owls-and-dragons-i/distfiles/owl_king.png` (cirka 6,1 MB enligt Agent 21). Gå igenom PNG-blocken efter den åtta byte långa signaturen. Agenterna 11 och 21 rapporterar sex stora `zTXt`-block efter `IDAT`, med nycklarna `SVG0`–`SVG5`. För varje block: dela vid nyckelns NUL-byte, hoppa över byten för komprimeringsmetod, zlib-avkoda återstående data och sätt ihop de sex byteföljderna i numerisk nyckelordning. Resultatet är cirka 5,7 MB SVG-markering. Agent 20:s parser går vidare med `blocklängd + 12` och stannar vid `IEND`.
2. Granska den återskapade SVG:n. Den innehåller Base64-data-URI:er och en dold grupp `<g id="huh">` (Agent 21 anger `display="none"`) med ett 7z-arkiv. Agent 20 tar den första `data:image/png;base64,...`-URI:n, Base64-avkodar den, kapar PNG-filen efter den första `IEND`-markören (inklusive markörens åtta byte) och öppnar bilden på 1024×768. Enligt den källan innehåller det översta bandet självt flaggan; detta motsäger gallerivägen nedan.
3. För gallerivägen granskas den avkodade PNG-datan efter dess `IEND`: enligt källorna ligger en PDF där. Läs PDF-texten för att få `<LÖSENORD REDIGERAT>` till det krypterade 7z-arkivet. Denna writeup anger inget kommando för PDF-textextraktion. Arkivet innehåller `thing.png`, en Cardan-gallerimask med svarta täckande områden och mellanrum/hål. Lösenordet ska inte avslöjas.
4. SVG-data efter `</svg>` beskrivs också som ett blobblock som XOR-avkodas byte för byte med `0x13`; resultatet är en JPEG på 5333×4000. Agent 21 kallar en kopia från en annan agent `trailer.bin`. Det svaga femradiga bokstavsrutnätet ligger nära högra kanten. Agent 21:s exakta beskärning är `(4400, 1800, 5200, 2400)` (800×600); Agent 11 anger ungefär samma beskärning.
5. Konvertera beskärningen till gråskala, använd autokontrast med `cutoff=1` och rotera sedan `-29.5` grader med expansion aktiverad och svart fyllning, enligt Agent 21. I `thing.png` används raderna 292–450, kolumnerna 237–817, grön kanal index 1, och värde 255 betyder hål. Skala masken med närmaste granne till `(815-168, 539-353)` = 647×186. Applicera den på raderna 353–539 och kolumnerna 168–815 i den roterade bilden; mörka pixlar under hålen genom att multiplicera dem med `0.1`. Beskär resultatet till `(150,335,830,555)`, dubbla båda dimensionerna med Lanczos och spara som `overlay.png`. Agent 11 säger bara att masken och rutnätet ska riktas in med enhetlig skalning, inte dessa gränser.
6. Gallerikällorna säger att bokstäverna genom hålen, lästa i ordning, ger `<SVARSFRAS REDIGERAD>`, som omsluts av utmaningens flaggformat. Agent 20 säger i stället att det inbäddade PNG-bandet visar flaggan direkt och kallar JPEG-/gallerivägen en villospårslösning. Den extraherade flaggan är `<FLAGGA REDIGERAD>`; beroende på avsedd väg säger källorna att den läses från bandet eller galleriresultatet.

## Motsägelser och upplösning

Agenterna 11 och 21 beskrev PDF-/lösenords-/7z-/JPEG-/Cardan-gallerivägen som gav
en "toys for the dragons"-fras. Agent 20 sa att flaggan finns direkt på den
inbäddade PNG-bildens band. **Lagets Discord-logg avgör: agent 20 har rätt.**
owl_king-lösningen renderade den bifogade PDF-sidan, som visar
`Flag for OwlsAndDragons I:` följt av igelkotts-frasen (den "hedgehog sage" som
nämns i uppgiften). "toys for the dragons" är temat för **del II**, inte del I, så
konsensusen från agent 11/21 är fel för den här utmaningen.

**Löst resultat:** redigerad — del I-flaggan är "amazing hedgehog"-frasen som läses
från den renderade PDF-bannern (notera den exakta blandade skiftningen i
"hEdGehog"), inte gallrets "toys for the dragons"-fras. Galler-/7z-materialet hör
till del II.
