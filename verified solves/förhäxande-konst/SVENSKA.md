# Förhäxande Konst — svenska

**Granskad källa:** `ctf-agent11.md`.

## Lösningsmetod steg för steg

1. Granska den bifogade PNG-bilden som ett rutnät av färgade plattor. Writeupen anger bildstorleken 1602×322 pixlar, plattor på 80×80 pixlar, 20 plattor per rad och totalt 63 plattor.
2. Läs mittpixeln i varje platta i radvis ordning med start i plattan längst upp till vänster. För kolumn `c` och rad `r` är provpunkten `(c*80+40, r*80+40)`. Använd formeln för alla 63 plattpositioner; källan anger ingen särskild hantering av de två oanvända platserna i rutnätet med 20 kolumner.
3. Lägg till den samplade pixelns röda byte, sedan gröna och sedan blå byte innan nästa platta behandlas. Sekvensen beskrivs som ASCII-kodad Base64 som slutar med `==` följt av en NUL-byte. Källan anger inte om Base64-avkodaren stannade vid eller ignorerade den avslutande byten; behandla det som en implementeringsdetalj som måste avgöras från originalartefakten i stället för att anta ett visst avkodarbeteende.
4. Base64-avkoda ASCII-datan. Källan anger 35 decimala värden, vart och ett i intervallet 130–160. Dessa är decimala bytevärden som inverteras var för sig, inte ett enda decimalt tal.
5. Återskapa tecknen genom att subtrahera varje decimalt värde från 255 och tolka resultatet som ett tecken: `chr(255 - n)`.
6. Som rimlighetskontroll nämner källan att slutvärdet 130 ger `255 - 130 = 125`, alltså tecknet för avslutande klammerparentes. Den uppger att den avkodade sekvensen bildar flaggan. Flaggan och samtliga tecken i den utelämnas.

## Verktyg och begränsningar

Källan nämner Python och Pillow men innehåller inte bilden eller ett komplett skript. Traverseringen av plattorna anges ovan; de faktiska samplade byten och avkodade resultatet återges inte. Källan rekommenderar att känna igen ASCII/Base64 och pröva en enkel byteinvertering mot förväntad flaggstruktur. Writeupen anger inte något Base64-avkodningsanrop eller hur NUL-byten hanteras. Inget påstående görs om godkännande på plattformen.

**Avkodat resultat:** `<FLAGGA REDIGERAD>` — enligt källan fås det genom att tillämpa `chr(255 - n)` på de Base64-avkodade decimaltalen.
