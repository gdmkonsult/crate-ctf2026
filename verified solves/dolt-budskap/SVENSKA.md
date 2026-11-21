# Dolt Budskap

## Källskrivning

Detta dokument översätter:

- `crate-ctf-swarm/writeups/dolt-budskap/ctf-agent11.md`

Källan anger en komplett forensisk arbetsgång.

## 1. Inspektera MP3-metadata

Kör:

```bash
exiftool file.mp3
```

Källan rapporterar två kommentarer:

```text
The waveform isn't the whole picture
compressed in the end under pressure
```

Den första kommentaren flyttar fokus från vanlig vågformsdata. Den andra pekar på ett arkiv komprimerat/bifogat i slutet av filen.

## 2. Skapa ett spektrogram

Kör källans exakta kommando:

```bash
ffmpeg -i file.mp3 -lavfi showspectrumpic=s=1600x800:legend=0:scale=log spec.png
```

Parameterförklaring:

- `-i file.mp3` väljer indatan;
- `-lavfi` tar emot ett filtergraf;
- `showspectrumpic` skapar spektrogrambilden;
- `s=1600x800` gör utdatan `1600 × 800`;
- `legend=0` inaktiverar legend;
- `scale=log` använder logaritmisk frekvensskala;
- `spec.png` är outputfilen.

Öppna `spec.png`. Källan anger att bilden synligt visar:

```text
password:<ZIP-LÖSENORD>
```

Det bokstavliga lösenordet är utmaningsberoende och redigeras här.

Lösenordet används för det bifogade AES-arkivet. Det är inte själva svaret och ska inte förväxlas med flaggan.

## 3. Skanna efter inbäddade filsignaturer

Kör:

```bash
binwalk file.mp3
```

Källan anger att `binwalk` identifierar ett AES-krypterat ZIP-arkiv vid absolut offset:

```text
363267
```

Offset är signaturens plats i `file.mp3`. Därför börjar extraktionen med bytet efter offset `363267`.

## 4. Extrahera endast det bifogade arkivet

Använd `tail` för att kopiera varje byte efter offset `363267`:

```bash
tail -c +363268 file.mp3 > h.zip
```

Anledningen till `+363268` är exakt: om ZIP-signaturen börjar på offset `363267`, startar `tail -c +N` vid det källangivna ZIP-dataoffsetet med indexvärdet `363268`.

Utdatan heter `h.zip`.

## 5. Dekryptera och extrahera ZIP-filen

Använd 7-Zip eftersom källan anger att ZIP-filen är AES-krypterad:

```bash
7z x -p<ZIP-LÖSENORD> h.zip
```

Ersätt `<ZIP-LÖSENORD>` med lösenordet som läses visuellt från `spec.png`. Ersätt inte med flaggan eller ett gissat lösenord.

Extraktionen återställer källans `flag.txt`.

Källan anger uttryckligen att vanlig `unzip` misslyckas på detta AES-arkiv, så 7-Zip/AES-stöd krävs.

## 6. Läs den återställda filen

Kör:

```bash
cat flag.txt
```

Filen innehåller utmaningens flagga i textform.

## Återvunnet resultat

Det bokstavliga lösenordet och flaggan utelämnas.

**Platshållare:**

- lösenord: `<ZIP-LÖSENORD>`
- flagga: `<UTMANINGSFLAGGA>`

Flaggan läses från `flag.txt` efter att `7z` extraherar `h.zip`.

## Kända luckor

- Källan ger ingen bildkopia av `spec.png`.
- Källan anger inget bokstavligt ZIP-lösenord.
- Källan ger inget oberoende plattformsgodkännande.
