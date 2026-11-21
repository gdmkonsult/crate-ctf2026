# Tidpunkt — Detailed reconstruction

## Sources read
- `writeups/tidpunkt/ctf-agent11.md`
- `writeups/tidpunkt/ctf-agent21.md`

## Source conflict
The sources disagree substantially. `ctf-agent11` identifies a Peab project in Djurgården, Linköping. `ctf-agent21` identifies Villa Österlen and Note Design Studio in Österlen, Skåne. The exact expected company/time answer is therefore unresolved from the available writeups.

## Step 1 — Extract EXIF metadata
Both sources use EXIF date extraction on:

```text
challenges/tidpunkt/distfiles/image.jpg
```

The command recorded by `ctf-agent11` is:

```bash
exiftool -DateTimeOriginal challenges/tidpunkt/distfiles/image.jpg
```

The command recorded as a date listing in `ctf-agent21` is:

```bash
exiftool -a challenges/tidpunkt/distfiles/image.jpg | grep -i date
```

Both records agree on the date:

```text
2026:07:30
```

Both redact or conceal the clock portion:

```text
xx:xx:xx
```

Thus only the date `2026-07-30` is a complete source-grounded fact.

## Step 2 — First source's complete visual and project sequence
`ctf-agent11` performs visual matching:

1. Match JYSK storefront signage and Nordic Wellness signage in the photograph.
2. Search for the addresses/places:
   - `Åsmestadsgatan 2`;
   - `Åsmestadsgatan 6`;
   - `Kv Jägarsoldaten i stadsdelen Djurgården i Linköping`;
   - `Peab bygger hyreslägenheter och vårdcentral i Linköping`.
3. The source identifies a Stångåstaden project built by Peab: 176 homes in the first phase, together with a health centre.
4. North-up satellite imagery is centered at `58.3859044,15.5888956`.
5. The storefront corridor from Nordic Wellness toward JYSK is traced and recorded as bearing about `155°`.
6. The photograph is interpreted as showing the sun slightly to the right of the corridor.
7. NOAA solar-position values in the source are:

```text
12:00 CEST: 156.7°
12:30 CEST: 167.4°
13:00 CEST: 178.4°
```

8. The sun offset is interpreted to fit 12:00 CEST.

The source's candidate flag format is `cratectf{företag_yyyy-mm-dd_hh}`. The exact company and hour are absent/redacted.

## Step 3 — Second source's independent sequence
`ctf-agent21` proposes a different research path:

1. Search architecture/news databases using `Villa Österlen`.
2. Candidate databases named by the source are Dezeen, ArchDaily and Architonic.
3. The candidate building is Villa Österlen, in Österlen, Skåne.
4. The source identifies Note Design Studio as the architect/company and Erik Lefvander as photographer.
5. It opens or analyzes `image.jpg` with PIL and NumPy; the source records these calculations:

```text
mean brightness: 128.7
sky region mean: 170
ground region mean: 100
```

The source's interpretation is that the sky is 70% brighter than the ground and that this indicates a high midday sun.
6. It concludes an estimated time of `12:00`.
7. The source explicitly says the contractor/builder was not publicly documented and suggests `Österlen Bygg` or other times such as `13:00` or `14:00` as alternatives.

The source's candidate flag format is `cratectf{company_yyyy-mm-dd_hh}`.

## Step 4 — Evaluate the conflict
The two analyses do not identify the same project or company:

- `ctf-agent11`: Linköping, Stångåstaden, Peab, solar azimuth 156.7° at 12:00.
- `ctf-agent21`: Österlen, Villa Österlen, Note Design Studio, brightness-based noon estimate.

The second source also admits no public builder documentation and no confirmed flag. Therefore it is not valid to choose one answer as proven without additional source evidence.

## Step 5 — Resolved answer

The operator confirmed the correct answer: company **Peab**, date **2026-07-30**,
hour **12**, i.e. the Linköping/Stångåstaden interpretation (ctf-agent11). Agent21's
Österlen/Note Design Studio answer is wrong. Important detail: the company is spelled
**lowercase `peab`** in the flag (format `cratectf{<company>_<yyyy-mm-dd>_<hh>}` →
`peab` / `2026-07-30` / `12`), even though the firm is normally written "Peab". The
wrapped literal flag is not reproduced here.

## Gaps
- `tidpunkt/ctf-agent21.md` gave a conflicting place and company; it is incorrect.
- The flag uses lowercase `peab`; do not capitalize it.
