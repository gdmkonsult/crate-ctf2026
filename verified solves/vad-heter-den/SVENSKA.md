# Vad Heter Den? — Detaljerad rekonstruktion

## Källor som lästs
- `writeups/vad-heter-den/ctf-agent11.md`
- `writeups/vad-heter-den/ctf-agent21.md`

## Steg 1 — Kontrollera metadata
Första källan kör:

```bash
exiftool challenges/vad-heter-den/distfiles/image.png
```

Andra källan anger 1887×1220 RGB/alpha PNG. Ingen användbar dold metadata hittas.

## Steg 2 — Visuell analys
Bildens gemensamma kännetecken:

- runt stenbygge;
- grönt koppartak;
- murar/ramparts;
- Göta/Nordre älv;
- Kungälv i bakgrunden.

## Steg 3 — Identifiera byggnaden
Båda källorna identifierar Bohus fästning i Kungälv. Andra källan anger byggstart 1308 under kung Haakon V och placering på Bagaholmen vid Göta älv.

## Steg 4 — Formatera
Formatet är `cratectf{xxxxx_xxxxxxxx}`:

- `bohus` har 5 bokstäver;
- `fästning` har 8 bokstäver;
- svensk `ä` ska behållas som UTF-8;
- orden separeras med understreck.

Kandidaten är därmed `bohus_fästning` som slutliga intern text. Bokstavlig flagga ersätts av `FLAG_PLACEHOLDER`.

## Luckor
Första källan säger att ä/a-bokstaven behöver verifieras. Andra källan antar UTF-8 men ingen plattformsacceptans visas.

## Plattformsbekräftat svar

`bohus_fästning` (**med `ä`**, gemener) är **plattformsbekräftat korrekt**
("Correct but you already solved this"). Varianten med vanligt `a` behövdes inte.
