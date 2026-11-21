# Skrottippen — Detailed reproduction

## Sources read
- `writeups/skrottippen/ctf-agent11.md`
- `writeups/skrottippen/ctf-agent21.md`

## Target artifact
- Compressed archive: `skrottippen.zip`
- Download URL in both sources: `https://ctf.crate.foi.se/files/e62a8dc3058e177a462894e342d58f94/skrottippen.zip`
- The first source calls the uncompressed log `skrottippen.txt` and says it is 1 GiB.
- The second source calls the compressed archive 170 MB.
- Source log format:

```text
ts | station | action | id | item | kg | material | op | note
```

## Step 1 — Download
The first source downloads with:

```bash
curl -o skrottippen.zip https://ctf.crate.foi.se/files/e62a8dc3058e177a462894e342d58f94/skrottippen.zip
```

The second source uses a quiet request with an explicit timeout:

```bash
curl -s -m 300 -o skrottippen.zip   'https://ctf.crate.foi.se/files/e62a8dc3058e177a462894e342d58f94/skrottippen.zip'
```

The exact URL is not challenge-derived secret material, so both are preserved.

## Step 2 — Stream rather than extract
The first source's primary command is:

```bash
unzip -p skrottippen.zip | LC_ALL=C grep -a -n 'hårddisk'
```

The flags mean:

- `unzip -p` writes the archived file to stdout instead of creating it on disk.
- `grep -a` treats the 1 GiB text as text even if it contains unusual byte values.
- `LC_ALL=C` makes matching deterministic for the byte-oriented search.
- `-n` includes the 1-based source line number.

The second source broadens the search:

```bash
unzip -p skrottippen.zip | grep -a -n -i 'lapp\|hårddisk\|cratectf'
```

Here:

- `-i` makes matching case-insensitive.
- `lapp` matches the hint word "note".
- `hårddisk` matches "hard drive".
- `cratectf` finds direct flag-prefix occurrences.

## Step 3 — Apply the description hint
Do not search only for the generic string `flag`. Both writeups state that the description explicitly hints at the item `gammal hårddisk med lapp`.

The broad matches may include decoys such as `ingen flagga här`; the anomalous entry is the one combining:

- station `KRAN-2`;
- action `KASSERAD`;
- item text `gammal hårddisk med lapp`;
- ID `SKROT-0FE4E78D`;
- material `elektronik`;
- operator `kjell`;
- mass `0.6 kg`.

## Step 4 — Confirm the line
Both sources place the relevant entry at line `5107584` in the uncompressed text.

The first source shows a normalized entry:

```text
... | gammal hårddisk med lapp: cratectf{[REDACTED]} | 0.6 kg | elektronik | op=kjell
```

The second source gives the fuller timestamp and station context:

```text
2022-04-26 18:56:39 | KRAN-2 | KASSERAD | id=SKROT-0FE4E78D | gammal hårddisk med lapp: <FLAG> | 0.6 kg | elektronik | op=kjell
```

The source writeups use different exact presentation for the surrounding fields, but both agree on line, item, ID, weight, material and operator.

## Step 5 — Extract the flag
Extract the text inside the `cratectf{...}` braces from the anomalous entry. Do not copy a line such as `ingen flagga här`, and do not invent a flag value.

The result is represented exactly as `FLAG_PLACEHOLDER`.

## Source gaps
- The exact raw flag is absent from both writeups.
- The private NFS publication mentioned in the sources is not an available source record and does not count as platform acceptance.
- The first source gives only `KASSERAD` line content without a complete header; the second gives a fuller entry.
