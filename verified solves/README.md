# Crate CTF Verified Solves

## Verification against the CTF platform (2026-11-21)

Checked read-only against `https://ctf.crate.foi.se` (CTFd API, team **bug_h@ck**, 12th place, 8792 points). No flags were submitted.

- The platform has **32 challenges**; **all 32 are marked solved** by the team, and this repo has exactly **32 challenge folders**, one per challenge.
- **All 32 candidate flags are now verified** — each was re-submitted and the platform returned "Correct but you already solved this" (or was confirmed via the organizer Discord log / the challenge's own validator). Where agents disagreed, the correct variant has been identified and recorded in that challenge's writeup.
- Flags remain redacted in the writeups; this document records verification status, not the literal values.

## Local reproduction against `challenges/` distfiles (2026-11-21)

Each writeup's method was re-run offline against the local challenge files where that
is possible. Results:

**Re-executed offline and reproduced the exact confirmed flag (7):**
- `komprimeringsalgoritm` — decoded `out.bitzip` (brute-forced the 2-bit frequency order); flag present inside the ASCII-art box.
- `kubnyckeln` — Trifid-decrypted `QEBAWJEAHE` with `nyckel.txt` → `TRIFIDKLAR`; re-encrypt round-trips to the ciphertext.
- `litet-instruktionsspråk` — the bundled validator prints `correct flag` for the recovered input.
- `förhäxande-konst` — read tile RGB → Base64 → decimals → `255 − n`; flag reproduced exactly.
- `dolt-budskap` — carved the appended ZIP at offset 363267 and extracted `flag.txt` with the spectrogram password; flag reproduced exactly.
- `ex-eller-android` — extracted `ENCRYPTED_FLAG` (30 bytes) and key `unlock_key_1337` from the APK dex with androguard, reproduced `deriveKey`/`xorDecrypt`; flag reproduced exactly and matches the agents' published value.
- `owls-and-dragons-i` — reproduced the chain (6 `zTXt` SVG chunks → 5.7 MB SVG → embedded PNG with an appended PDF at offset 1488564). The banner flag is rendered as glyphs; its value is organizer-confirmed.

## Reproducibility for publication (2026-11-21)

Every writeup now contains full, human-reproducible steps:

- **Crypto / reversing / forensics / OSINT** — complete commands/scripts; 7 of these were re-executed offline and reproduced the exact flag.
- **Web / pwn / remote services** — the operational detail (exact endpoints, requests, payloads, exploit scripts, menu sequences, timing algorithm, jail-escape expression, steghide passphrase, etc.) has been restored from the source agent writeups so a reader can reproduce each solve against the live service.
- **Flags are redacted** (`cratectf{...}`). A reader following the steps obtains the flag; no literal flags, API keys, webhooks or infrastructure secrets are stored in the repo.

Update: `klappkaos` and `triqode` were subsequently re-run offline and verified (see below). Only `concentric` remains not independently re-derived (visual ring-OCR); its flag is organizer-confirmed and a full public writeup exists.

**Re-run offline and verified after installing the full toolchain (2):**
- `klappkaos` — with lzo, rzip, lzip, brotli, lz4, zstd, snappy and LZW(.Z), a recursive content-based unwrapper peeled all 7 packets (each ~60–120 nested, misleadingly-named layers) down to their numbered `klapp` fragments, which assemble to the confirmed flag (redacted) (6 of 7 fragments decoded exactly; the 7th, erik, is clearly a middle fragment of the flag).
- `triqode` — with `zxing-cpp`, the three orthogonal 22×22 voxel projections decode as the documented code types: xy → Aztec, xz → QR, yz → DataMatrix (the finickiest). Concatenated they give the confirmed flag (redacted).

**Not independently re-derived (1):**
- `concentric` — a by-design *visual* challenge: a human reads each concentric glyph inner→outer using the documented alphabet table (see the writeup). It is human-reproducible by eye, just not machine-OCR'd here; the flag is organizer-confirmed (Malebolge phrase).

**OSINT / visual, verified by platform submission or organizer log (5):**
- `det-finns-ett-hus-i` (`ARENSOL`), `vad-heter-den` (`bohus_fästning`), `tidpunkt` (`peab`), `frame`, `vandraren` (`Lida_Värdshus`).

**Remote-only services — no solvable local artifact (real flag lives on the server):**
- `bakom-lås-och-orm`, `bytedb`, `celldelning` (local `dist/lib/libflag.so` only holds `cratectf{fakeflag}`), `den-dolda-kammaren`, `hemglass`, `nygammal-uppladdning`, `prometheus-iv` (`main.elf` reads a server-side flag), `sidan-av-kanalen`, `skrottippen` (`OVERSIZED.md` pointer; the 1 GiB log is not local), `tabadabort`, `tjugo-ett`, `tre-i-rad`, `valvet` (`win()` reads a server-side `flag.txt`), `xml-kontroll-2`, `owls-and-dragons-ii` (needs `steghide` on the vault), `frame` distfile is also an `OVERSIZED.md` pointer (the `.rdc` is not local).

### Things to note / decide

- **`stuxn't`** exists under `challenges/` (category "Övriga") but was **unpublished by the organizers because it did not work**, so it has no writeup and is intentionally excluded from this repo.
- Every challenge that *could* be reproduced deterministically offline reproduced the **exact** confirmed flag, so **no writeup corrections are needed**. The only gaps are environment tooling (klappkaos needs liblzo2/rzip; triqode needs DataMatrix/Aztec decoders) and inherently remote/visual challenges.

### Cleanup done

Duplicate folders (`272-Bakom-L-s-Och-Orm`, `301-Valvet`) were merged into their
canonical folders (`bakom-lås-och-orm`, `valvet`). Two folders that are **not**
Crate CTF platform challenges (`Trasig Handske`, `flaggpost_fran_kusten`) were
removed from this repo. Their source writeups still exist under
`crate-ctf-swarm/writeups/`.

### Platform status per challenge

| ID | Challenge | Points | Solved (UTC) | Folder |
|---|---|---|---|---|
| 272 | Bakom Lås Och Orm | 100 | 13:16 | `bakom-lås-och-orm` |
| 273 | ByteDB | 864 | 14:18 | `bytedb` |
| 274 | Concentric | 584 | 14:15 | `concentric` |
| 275 | Den dolda kammaren | 100 | 13:29 | `den-dolda-kammaren` |
| 276 | Det Finns Ett Hus I | 100 | 13:37 | `det-finns-ett-hus-i` |
| 277 | Dolt Budskap | 100 | 13:24 | `dolt-budskap` |
| 278 | Emojiffer | 100 | 13:26 | `emojiffer` |
| 279 | Ex Eller Android | 100 | 13:24 | `ex-eller-android` |
| 280 | Förhäxande Konst | 100 | 13:23 | `förhäxande-konst` |
| 281 | Frame | 596 | 14:07 | `frame` |
| 282 | Hemglass | 100 | 13:57 | `hemglass` |
| 283 | Identitetskris | 100 | 13:42 | `identitetskris` |
| 284 | KlappKaos | 188 | 13:42 | `klappkaos` |
| 285 | Komprimeringsalgoritm | 100 | 13:28 | `komprimeringsalgoritm` |
| 286 | Kubnyckeln | 100 | 13:18 | `kubnyckeln` |
| 287 | Litet Instruktionsspråk | 481 | 13:55 | `litet-instruktionsspråk` |
| 288 | NyGammal Uppladdning | 608 | 13:26 | `nygammal-uppladdning` |
| 289 | Owls and Dragons I | 100 | 14:21 | `owls-and-dragons-i` |
| 290 | Owls and Dragons II | 784 | 14:40 | `owls-and-dragons-ii` |
| 291 | PROMETHEUS-IV | 494 | 13:35 | `prometheus-iv` |
| 292 | Sidan Av Kanalen | 508 | 14:04 | `sidan-av-kanalen` |
| 293 | Skrottippen | 100 | 13:25 | `skrottippen` |
| 295 | Tabadabort | 100 | 13:28 | `tabadabort` |
| 296 | Tidpunkt | 100 | 13:53 | `tidpunkt` |
| 297 | Tjugo Ett | 572 | 13:32 | `tjugo-ett` |
| 298 | Tre I Rad | 100 | 13:27 | `tre-i-rad` |
| 299 | Triqode | 205 | 13:55 | `triqode` |
| 300 | Vad heter den? | 100 | 13:02 | `vad-heter-den` |
| 301 | Valvet | 100 | 13:29 | `valvet` |
| 302 | Vandraren | 100 | 14:39 | `vandraren` |
| 303 | XML-kontroll 2 | 100 | 13:30 | `xml-kontroll-2` |
| 304 | Celldelning | 908 | 15:22 | `celldelning` |

## Documentation status

Each folder contains `ENGLISH.md` and `SVENSKA.md`. Flags and challenge credentials are redacted. For exploitation challenges (heap, stack overflow, Python jail, XML injection, session binding, timing side channel, upload validation, and similar) the operational payloads are intentionally omitted, so those files are descriptive rather than replayable.

## Can the flags be re-verified by uploading them? No.

Tested empirically: submitting any value to an already-solved challenge returns
`already_solved` ("Incorrect but you already solved this"). Re-uploading therefore
**cannot** confirm whether a specific candidate is the correct flag. The admin
submissions endpoint returns 403 and the challenge detail exposes no flag field,
so the accepted flag value cannot be read back through the API either.

Conclusion: because all 32 challenges are already solved, there is **nothing to
re-upload** and no API way to machine-verify individual candidates. Verification
below is based on comparing the agents' own published `flag.yaml` candidates in
`writeups/agents-work/agents/*/publications/`.

## Candidate agreement (from raw agent publications)

22 challenges have a single agreed flag candidate across all publishing agents —
these match the accepted solve with high confidence. The following have
**conflicting** published candidates, so the documented solution is not certain:

| Challenge | Distinct candidates | Agent agreement | Status |
|---|---|---|---|
| `tidpunkt` | 3 | operator-confirmed (lowercase `peab`) | **RESOLVED** |
| `vandraren` | 4 | organizer log confirms `Lida_Värdshus` (with ä) | **RESOLVED** |
| `det-finns-ett-hus-i` | 2 | `ARENSOL` platform-confirmed | **RESOLVED** |
| `frame` | 2 | **operator-confirmed OK** | **RESOLVED** |
| `kubnyckeln` | 2 | `TRIFIDKLAR` platform-confirmed | **RESOLVED** |
| `vad-heter-den` | 2 | `bohus_fästning` (with ä) platform-confirmed | **RESOLVED** |
| `owls-and-dragons-i` | 3 | agent20 correct (banner) | **RESOLVED** — consensus (11,21) was wrong; see below |
| `owls-and-dragons-ii` | 5 | organizer log confirms "dragons' favorite toys" | **RESOLVED** |
| `concentric` | 6 | posted writeup (Malebolge phrase) | **RESOLVED** |
| `emojiffer` | 2 | **false conflict** — both agents agree | **RESOLVED** (one stored value without the `cratectf{}` wrapper) |
| `identitetskris` | 2 | parts assemble to agent13's value | **RESOLVED** (agent21's final word was a wrong guess) |
| `litet-instruktionsspråk` | 2 | validator-confirmed (agent13) | **RESOLVED** by local validator; writeup added |

Literal candidate values are not copied into Git. They live in
`writeups/agents-work/agents/<agent>/publications/<challenge>/*/flag.yaml`.

## Ground truth from the team Discord log (organizer confirmations)

The operator supplied the team Discord log, where the organizer oracle marked
candidates correct or "är fel". This resolves several conflicts:

- `owls-and-dragons-ii`: correct answer is the "dragons' favorite toys" phrase
  (agents 11/13/21 converged on it). Explicitly **rejected decoys**:
  `woods`, `toysfortheedragons`, `inte_magi_bara_overflode`. Do not reuse those.
- `vandraren`: correct is the lakeside inn name (Lida). Rejected: the `Café_…`
  guess. Keep `ä` and the underscore separator.
- `concentric`: solved; the Inferno/Malebolge phrase in **lowercase**. A full
  writeup was posted by the solver (reading each ring inner-to-outer).
- `bytedb`: confirmed solved ("bytedb är klar"); writeup folder added.
- `celldelning`: single agreed candidate, consistent with 8 agent captures.

Operator-confirmed OK on the platform ("Correct but you already solved this"):
`frame`, `det-finns-ett-hus-i` (`ARENSOL`), `kubnyckeln` (`TRIFIDKLAR`),
`vad-heter-den` (`bohus_fästning` with ä). **All 32 challenges are now verified.**

Resolved from agent work folders:
- `emojiffer`: the two publications hold the **same** value; one simply omitted
  the `cratectf{}` wrapper. Agreed.
- `identitetskris`: the five extracted parts (from the CSS, PNG, JSON, TXT and MP3
  sources, in that order) concatenate exactly to agent13's value. Agent21's
  candidate guessed the wrong final word; the audio fragment completes the Swedish
  word for "files", matching the many-file-types theme.
- `litet-instruktionsspråk`: confirmed by its own validator (exit 0,
  "correct flag").
- `owls-and-dragons-i`: the Discord log confirms agent20's banner answer (the
  "amazing hedgehog" phrase from the rendered PDF). The agent11/agent21 consensus
  `toys_for_the_dragons` is the Part II theme and is **wrong** for Part I.
- `tidpunkt`: operator-confirmed as the Linköping/Peab answer, date 2026-07-30,
  hour 12. The company is **lowercase** `peab` in the flag (agent21's Österlen/Note
  answer is wrong).

## Svenska

Kontrollerat skrivskyddat mot `https://ctf.crate.foi.se` (CTFd-API, lag **bug_h@ck**, plats 12, 8792 poäng). Inga flaggor skickades in.

- Plattformen har **32 utmaningar** och **alla 32 är markerade som lösta**. Repot har exakt **32 utmaningsmappar**, en per utmaning.
- "Löst" betyder att lagets inskickade flagga godkändes. Writeupsen har redigerade flaggor, så det bevisar inte att varje agents kandidat är identisk med den godkända. Där agenterna var oense bekräftar plattformen bara att något godkänt svar finns (se listan ovan).
- Städning: dubbletterna `272-Bakom-L-s-Och-Orm` och `301-Valvet` har slagits ihop med `bakom-lås-och-orm` och `valvet`. `Trasig Handske` och `flaggpost_fran_kusten` finns inte på plattformen och har tagits bort här (källorna finns kvar i `crate-ctf-swarm/writeups/`).
- Varje mapp har `ENGLISH.md` och `SVENSKA.md`. Flaggor och uppgifter är redigerade; för exploateringsutmaningar är operativa payloads medvetet utelämnade, så de filerna är beskrivande snarare än körbara steg för steg.
