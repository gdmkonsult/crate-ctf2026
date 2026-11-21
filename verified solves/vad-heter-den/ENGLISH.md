# Vad Heter Den? — Detailed reconstruction

## Sources read
- `writeups/vad-heter-den/ctf-agent11.md`
- `writeups/vad-heter-den/ctf-agent21.md`

## Challenge prompt
- Category: OSINT
- Provided image: `image.png`
- Prompt: identify the stone building.
- Flag format: `cratectf{xxxxx_xxxxxxxx}`
- Both sources interpret the format as lowercase: five letters, underscore, eight letters.

## Step 1 — Check the image for hidden metadata
The first source runs:

```bash
exiftool challenges/vad-heter-den/distfiles/image.png
```

The second source records the image as 1887×1220 PNG, RGB+alpha, and runs the same EXIF class of inspection. Neither source reports hidden EXIF metadata that identifies the building.

## Step 2 — Perform visual analysis
The image features agreed in both writeups are:

- a round stone tower;
- green copper roof;
- surrounding stone walls/ramparts;
- Göta/Nordre älv nearby;
- the town of Kungälv in the background.

The second source adds that the fortress is on Bagaholmen and the image matches publicly available aerial/architectural views of a medieval fortress.

## Step 3 — Identify the building
Both sources identify the building as Bohus Fortress.

The Swedish name used in the writeups is:

```text
Bohus fästning
```

Independent written evidence given by the second source:

- official Norwegian name: Bohus festning;
- built in 1308 under King Haakon V;
- located on Bagaholmen near the Göta river, north of Gothenburg.

The first source relies on visual identification and proposes “bohus” plus “fästning” directly.

## Step 4 — Translate to flag format
Count the characters in each word:

- `bohus` = 5 letters;
- `fästning` = 8 letters;
- Swedish `ä` remains encoded as UTF-8;
- source format requires an underscore between the words.

Candidate spelling:

```text
bohus_fästning
```

with the challenge prefix/suffix: `cratectf{<fortress>_<swedish word for fortress>}` (the fortress is Bohus; flag value redacted). Keep `ä` as `ä`; do not replace it with `a` or omit the prefix/suffix.

The exact challenge value is redacted as:

```text
FLAG_PLACEHOLDER
```

## Source gaps
- The first source explicitly says `ä` versus `a` must be verified with organizers.
- The second source assumes UTF-8 and claims high confidence, but neither records an independent platform acceptance.
- The exact literal flag is absent from both writeups.

## Platform-confirmed answer

`bohus_fästning` (**with `ä`**, lowercase) is **platform-confirmed correct**
("Correct but you already solved this"). The plain-`a` spelling was not needed.
