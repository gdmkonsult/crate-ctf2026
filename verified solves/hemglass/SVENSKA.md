# Hemglass — Svenska

**Kategori:** Webb / Reversing · **Endpoint:** `http://hemglass.challs.crate.nu/verify`

Utmaningen är en webbaserad speldosa som emulerar den klassiska Hemglass-slingans hårdvara
(en MOS 6581 "SID"-ljudkrets styrd av en Z80). `POST /verify` tar emot en enda melodisträng
och returnerar flaggan endast om melodin exakt matchar den kanoniska Hemglass-melodin.

## 1. Hitta melodins källa (OSINT)

1. Melodisträngens format är `<längd><ton><oktav>` per not (t.ex. `8D4`), mellanslagsseparerat.
   Du behöver den exakta kanoniska notsekvensen och timingen.
2. Granska den senaste ändringshistoriken (november 2026) för den svenska Wikipedia-artikeln
   om *Hemglass*. En nyligen tillagd referens pekar på Svenska Commodoreklubbens
   rekonstruktion: `https://www.commodore.se/hemglassmelodin-en-rekonstruktion/`.
3. Artikeln dokumenterar speldosans hårdvara (Z80 + SID 6581 + 2 kB EPROM) och länkar ett
   källkodsarkiv, `hemglass_mjukvara_z80.zip`. Ladda ner och packa upp det.

## 2. Läs ut melodin ur disassemblingen

Öppna den disassemblerade `hemglass.asm` ur arkivet. Melodisektionen innehåller exakt 16
noter, var och en med en av tre loop-fördröjningskonstanter i förhållandet
`DELAY_SHORT : DELAY_LONG : DELAY_VLONG = 1 : 2 : 3`.

| # | Ton | Fördröjningsklass |
|---|-----|-------------------|
| note01 | D4 | SHORT |
| note02 | B4 | SHORT |
| note03 | D5 | SHORT |
| note04 | B4 | SHORT |
| note05 | G4 | LONG |
| note06 | D4 | SHORT |
| note07 | B4 | SHORT |
| note08 | D5 | SHORT |
| note09 | B4 | SHORT |
| note10 | G4 | LONG |
| note11 | D4 | SHORT |
| note12 | C5 | LONG |
| note13 | C5 | SHORT |
| note14 | A4 | LONG |
| note15 | A4 | SHORT |
| note16 | G4 | VLONG |

## 3. Mappa fördröjningsklasser till webbgränssnittets längdnotation

Webbappen använder vanliga notvärdessiffror där ett högre tal är en kortare not:

- `SHORT`  → `8`  (åttondel)
- `LONG`   → `4`  (fjärdedel = 2× åttondel)
- `VLONG`  → `4.` (punkterad fjärdedel = 3× åttondel)

Skriv varje not som `längd` omedelbart följt av `ton` och `oktav`, mellanslagsseparerat.
Den resulterande kanoniska melodisträngen blir:

```text
8D4 8B4 8D5 8B4 4G4 8D4 8B4 8D5 8B4 4G4 8D4 4C5 8C5 4A4 8A4 4.G4
```

## 4. Skicka till `/verify`

Skicka strängen som JSON-fältet `melody` med `Content-Type: application/json`:

```bash
curl -s -X POST http://hemglass.challs.crate.nu/verify \
  -H 'Content-Type: application/json' \
  -d '{"melody":"8D4 8B4 8D5 8B4 4G4 8D4 8B4 8D5 8B4 4G4 8D4 4C5 8C5 4A4 8A4 4.G4"}'
```

Tjänsten svarar:

```json
{"ok": true, "flag": "cratectf{...}"}
```

Flaggan är värdet i `flag`-fältet (redigerat bort här — en människa som kör kommandot ovan
får ut det direkt).

## Verifiering

Plattformsbekräftat löst. `curl`-anropet ovan returnerar `{"ok": true, "flag": ...}` med
den riktiga flaggan; det bokstavliga värdet är dolt för publicering.
