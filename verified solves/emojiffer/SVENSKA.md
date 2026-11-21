# Emojiffer

## Källskrivningar

Denna översättning sammanväger:

- `crate-ctf-swarm/writeups/emojiffer/ctf-agent11.md`
- `crate-ctf-swarm/writeups/emojiffer/ctf-agent21.md`

Båda identifierar en monoalfabetisk substitutionschiffer där varje emoji representerar en fast klartextbokstav eller mellanslag.

## 1. Läs och ta bort filhuvudet

Öppna `ciphertext.txt` och lokalisera:

```text
Language=English, Text=
```

Använd texten efter `Text=` som chiffertext. Bevara emoji-gränserna exakt.

## 2. Räkna symbolmängden

Källan anger 27 olika emojisymboler:

- 26 representerar det lilla alfabetet;
- en representerar mellanslag.

Denna fasta ett-till-ett-mappning är den monoalfabetiska substitutionsmodellen.

## 3. Identifiera mellanslag

Agent21 anger att `🤟` förekommer `3 359` gånger, omkring `16 %` av texten. Därför avbildas `🤟` till mellanslag.

Mappningen följer ordfördelningen och stämmer med källans orduppdelning.

## 4. Identifiera det första vanliga ordet

Den frekventa tre-symbolssekvensen:

```text
🐹🍰🦁
```

förekommer `243` gånger och identifieras som `the`.

Det fastställer:

```text
🐹 -> t
🍰 -> h
🦁 -> e
```

## 5. Identifiera det första ämnesordet

Källan avbildar:

```text
🎁🦊🌟🐶🐹🍇🍔🦊🦓🐶🍰🌟
```

till:

```text
cryptographer
```

Därför:

```text
🎁 -> c
🦊 -> r
🌟 -> y
🐶 -> p
🍇 -> o
🍔 -> g
🦓 -> a
```

Sekvensen `🐹` är redan känd som `t`, och `🍰`/`🌟` som `h`/`y`.

## 6. Utöka mappningen via vanliga ord

Agent21 anger denna kompletta mappningsmängd:

```text
🤟 -> mellanslag
🐹 -> t
🍰 -> h
🦁 -> e
🎁 -> c
🦊 -> r
🌟 -> y
🐶 -> p
🍇 -> o
🍔 -> g
🦓 -> a
🍓 -> i
🌭 -> m
🎈 -> n
🥗 -> f
🍎 -> v
✨ -> b
🍕 -> k
🐭 -> l
🍜 -> d
🐻 -> u
🎂 -> x
🐱 -> q
🍣 -> z
🍌 -> j
```

Den återstående omappade symbolen, om någon, är källans återstående bokstav `s` eller `w`, men agent11 och agent21 presenterar ingen entydig mappning.

Källan visar ordbekräftelser:

- `🍓` förekommer i `cipher`;
- `🌭` förekommer i `common` och sammansättningar;
- `🎈` förekommer i `from`/`known`;
- `🥗` avbildas till `f`;
- `🍎` avbildas till `v`;
- `✨` avbildas till `b`;
- `🍕` avbildas till `k`;
- `🐭` avbildas till `l`;
- `🍜` avbildas till `d`;
- `🐻` avbildas till `u`;
- `🎂` avbildas till `x`;
- `🐱` avbildas till `q`;
- `🍣` avbildas till `z`;
- `🍌` avbildas till `j`.

## 7. Oberoende hill-climb-metod

Agent11 beskriver en oberoende lösningsväg:

1. börja med 27 olika symboler;
2. bevara mellanslaget som symbol med högst frekvens;
3. använd quadgram-logprobabilities;
4. använd Python `pydoc`-ämnestext som språkkorpus;
5. byt par av symbol-till-bokstav-mappningar;
6. acceptera byten som förbättrar n-grampoängen;
7. upprepa tills texten blir läsbar;
8. extrahera den inskjutna svenska texten.

Källan kallar detta monoalfabetisk hill climb. Den ger inte korpus-kod eller slutnyckel.

## 8. Dekryptera texten

För varje emoji i texten ersätts den med mappad klartextbokstav. Källan anger att klartexten är en historik över kryptografi.

Agent21:s substitutionsloop är:

```python
plaintext = ''.join(mapping.get(c, '?') for c in emoji_text)
```

`?` markerar en symbol som saknas i den partiella mappningen. Det är inte bevis på att slutklartexten är okänd: källan använder mappningen för att verifiera igenkännbara ord.

## 9. Extrahera den gömda flaggan

Källan anger att en inskjuten svensk fras finns:

```text
unsuited to pen and paper
flaggan ar ...
```

Den exakta saknade texten ersätts här. Källan anger att den är den 15-tecken långa inre flaggan som matchar:

```text
cratectf{XXXXXXXXXXXXXXX}
```

Den inskjutna frasen beskrivs som “flaggan är …” på svenska.

## Återvunnet resultat

**Platshållare:** `<UTMANINGSFLAGGA>`

## Kända luckor

- Agent11 ger ingen full nyckel eller bokstavlig klartext.
- Agent21 ger mappningar men ingen exakt slutnyckel eller bokstavlig flagga.
- Avkodningen är inte entydig om de exakta `u`/`w`/`s`-mappningarna och ger ingen oavkortad slutnyckel.
- Ingen CTF-plattformsgodkännande ingår.

## Upplösning (från agenternas work-foldrar)

Båda agenternas publicerade `flag.yaml` innehåller **samma** flaggvärde; den
tidigare "konflikten" berodde bara på att den ena lagrade värdet utan
`cratectf{}`-wrappen. Flaggan är en kort svensk fras ("massor med emojis"); det
literala värdet är redigerat här. Utmaningen är löst på plattformen.
