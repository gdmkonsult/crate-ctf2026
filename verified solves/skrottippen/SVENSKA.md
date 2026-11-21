# Skrottippen — Detaljerad reproduktion

## Källor som lästs
- `writeups/skrottippen/ctf-agent11.md`
- `writeups/skrottippen/ctf-agent21.md`

## Artefakt
- Arkiv: `skrottippen.zip`
- Känt URL-värde: `https://ctf.crate.foi.se/files/e62a8dc3058e177a462894e342d58f94/skrottippen.zip`
- Källan anger 1 GiB okomprimerad logg och 170 MB komprimerad arkivstorlek.
- Format: `ts | station | action | id | item | kg | material | op | note`

## Steg 1 — Hämta arkivet
Första källan:

```bash
curl -o skrottippen.zip https://ctf.crate.foi.se/files/e62a8dc3058e177a462894e342d58f94/skrottippen.zip
```

Andra källan:

```bash
curl -s -m 300 -o skrottippen.zip   'https://ctf.crate.foi.se/files/e62a8dc3058e177a462894e342d58f94/skrottippen.zip'
```

## Steg 2 — Strömma utan uppackning
Primärt kommando:

```bash
unzip -p skrottippen.zip | LC_ALL=C grep -a -n 'hårddisk'
```

- `unzip -p` skriver arkivets fil till stdout.
- `grep -a` behandlar binära/oväntade byte som sökbart textflöde.
- `LC_ALL=C` ger deterministisk bytebaserad matchning.
- `-n` visar radnummer.

Alternativ breddning:

```bash
unzip -p skrottippen.zip | grep -a -n -i 'lapp\|hårddisk\|cratectf'
```

## Steg 3 — Använd ledtråden
Sök på den utmaningsspecifika beskrivningen `gammal hårddisk med lapp`, inte bara ordet flagga. Ignorera lockposter som `ingen flagga här`.

Den avvikande posten har:

- station `KRAN-2`;
- handling `KASSERAD`;
- ID `SKROT-0FE4E78D`;
- vikt `0.6 kg`;
- material `elektronik`;
- operator `kjell`.

## Steg 4 — Bekräfta rad
Båda källorna anger rad `5107584` i den okomprimerade loggen.

Första källa:

```text
... | gammal hårddisk med lapp: cratectf{[REDACTED]} | 0.6 kg | elektronik | op=kjell
```

Andra källa:

```text
2022-04-26 18:56:39 | KRAN-2 | KASSERAD | id=SKROT-0FE4E78D | gammal hårddisk med lapp: <FLAG> | 0.6 kg | elektronik | op=kjell
```

## Steg 5 — Extrahera flaggan
Läs endast texten inom `cratectf{...}` i den avvikande posten. Bokstavligt värde är okänt i källorna och ersätts av `FLAG_PLACEHOLDER`.

## Luckor
Båda nämner privat NFS-publicering men visar ingen faktiskt värdepost eller plattformsacceptans. Rå flagga saknas därför.
