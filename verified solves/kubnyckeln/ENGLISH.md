# Kubnyckeln — English

**Sources reviewed:** `ctf-agent11.md`, `ctf-agent13.md`, and `ctf-agent21.md`.

## Step-by-step method

1. Read `challenges/kubnyckeln/distfiles/nyckel.txt` and `challenges/kubnyckeln/distfiles/chiffertext.txt`. Agent 13 reads the key as nine rows, each containing three entries matched by `### <A-Z or @> ### <A-Z or @> ### <A-Z or @> ###`; the expected total is 27 symbols. Agent 21 transcribes the boxes as:
   - BOX1: `B O X / R A D / K L U`
   - BOX2: `M N C / E F G / H I J`
   - BOX3: `P Q S / T V W / Z Y @`
   
   Agent 13 says the file contains three 3×3 boxes and the 27th symbol `@`, with symbol order box-major then row-major. The ciphertext is `QEBAWJEAHE`.
2. Assign one-based coordinates `(box, row, column)` by enumerating boxes 1–3, rows 1–3, and columns 1–3. For each of the ten ciphertext letters, look up its key-cube coordinate. Agent 13 treats all ten letters as one Trifid period (`n = 10`).
3. Use Agent 13's decryption procedure: append each ciphertext letter's three coordinates to a flat list, then split the 30 values into `a = flat[:10]`, `b = flat[10:20]`, and `c = flat[20:30]`. The plaintext coordinate for position `i` is `(a[i], b[i], c[i])`; look up each triple in the reverse coordinate map to obtain a plaintext character.
4. The standard alphabet cube used for decoding is:
   - BOX1: `A B C / D E F / G H I`
   - BOX2: `J K L / M N O / P Q R`
   - BOX3: `S T U / V W X / Y Z @`
5. To perform Agent 13's round-trip check, map each plaintext symbol back to its `(box,row,column)` triple; collect all box values, then all row values, then all column values. Read the resulting 30-value list in consecutive triples, map those triples through the same reverse cube, and compare the result with `QEBAWJEAHE`. Agent 13 reports `roundtrip ok: True`; its decoded token is `<PLAINTEXT TOKEN REDACTED>`.
6. Agent 21 describes a different route: for each ciphertext character it forms three lists (`layers`, `rows`, `cols`), uses those lists as a 3×10 matrix, and reads each column as a triple into the standard cube. It reports a ten-character intermediate result and takes the first nine characters for a candidate. The literal intermediate and candidate are withheld because the source results conflict.

## Conflicts and unresolved result

The three sources do not agree on the plaintext. Agent 11 reports a nine-character uppercase result and explicitly says casing is unverified. Agent 13 reports a nine-character token and a successful round trip, but does not publish the token. Agent 21 reports a different, ten-character decode and says to use its first nine characters; its described coordinate handling differs from Agent 13's flat-list split. The plaintext, flag, and any candidate characters are withheld, and the sources do not provide enough consistent evidence to resolve the discrepancy. No platform-acceptance claim is made.

**Result:** `<FLAG REDACTED; EXACT CANDIDATE UNRESOLVED>` — the sources say it is derived by decrypting `QEBAWJEAHE` with the cube, but disagree on the plaintext.

## Platform-confirmed answer

`TRIFIDKLAR` (uppercase, agents 11, 13) is **platform-confirmed correct** ("Correct
but you already solved this"). Agent21's `TMAEXRMEP` was wrong.
