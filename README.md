# Crate CTF 2026 — Team `bug_h@ck`

Writeups for **every challenge we solved** at Crate CTF 2026 — **32 solves**,
finishing **12th**

Each challenge has its own folder under [`verified solves/`](./verified%20solves)
with a writeup in **English** (`ENGLISH.md`) and **Swedish** (`SVENSKA.md`). Flags
are redacted — follow the steps and you'll recover them yourself.

## How we solved it 🤖🐝

This was a team effort with a twist: most of the heavy lifting was done by a
**swarm of autonomous CTF agents** working side by side with the humans.

- A fleet of agents each picked up challenges independently and worked them in
  parallel across all categories — web, pwn, reversing, crypto, forensics, OSINT
  and misc.
- They coordinated over a **shared NFS channel**: every agent wrote its current
  hypothesis and findings there, and published flag candidates as small,
  immutable "events" the rest of the swarm could see.
- Nothing counted until it was **independently reproduced by a different agent**
  from the written steps — and a **human always made the final submission**.
- Humans jumped in where machines shouldn't decide alone: visual/audio
  steganography, OSINT judgment calls, and the occasional "is that an `ä` or an
  `a`?" moment.

That's the gist — we're keeping the exact orchestration, plumbing and secret
sauce to ourselves. 😉

Every solution here was re-checked after the event: all 32 flags were confirmed
against the platform, and the deterministic challenges were re-run end-to-end
offline to make sure the writeups actually reproduce.

## Team

| Member | Score |
|---|---|
| [bug_h@ck](https://ctf.crate.foi.se/users/2549) (Captain) | 1222 |
| [crone](https://ctf.crate.foi.se/users/2550) | 2307 |
| [ctf-agent23](https://ctf.crate.foi.se/users/3259) | 934 |
| [allo](https://ctf.crate.foi.se/users/2909) | 884 |
| [malalalal](https://ctf.crate.foi.se/users/3442) | 767 |
| [Oerten](https://ctf.crate.foi.se/users/2551) | 0 |
| [st4ck_arr0w](https://ctf.crate.foi.se/users/2552) | 0 |
| [williamhglund](https://ctf.crate.foi.se/users/2553) | 0 |
| [Bern Hard](https://ctf.crate.foi.se/users/3257) | 0 |
| [gdmkeno](https://ctf.crate.foi.se/users/3260) | 0 |
| [Paka](https://ctf.crate.foi.se/users/3348) | 0 |

## Solves

| Challenge | Category | Value | Time (2026-11-21) |
|---|---|---:|---|
| [Celldelning](https://ctf.crate.foi.se/challenges#Celldelning-304) | Exploatering | 834 | 16:22 |
| [Owls and Dragons II](https://ctf.crate.foi.se/challenges#Owls%20and%20Dragons%20II-290) | Forensik | 676 | 15:40 |
| [Vandraren](https://ctf.crate.foi.se/challenges#Vandraren-302) | OSINT | 100 | 15:39 |
| [Owls and Dragons I](https://ctf.crate.foi.se/challenges#Owls%20and%20Dragons%20I-289) | Forensik | 100 | 15:21 |
| [ByteDB](https://ctf.crate.foi.se/challenges#ByteDB-273) | Exploatering | 784 | 15:18 |
| [Concentric](https://ctf.crate.foi.se/challenges#Concentric-274) | Kryptografi | 467 | 15:15 |
| [Frame](https://ctf.crate.foi.se/challenges#Frame-281) | Forensik | 0 | 15:07 |
| [Sidan Av Kanalen](https://ctf.crate.foi.se/challenges#Sidan%20Av%20Kanalen-292) | Övriga | 100 | 15:04 |
| [Hemglass](https://ctf.crate.foi.se/challenges#Hemglass-282) | Webb | 100 | 14:57 |
| [Triqode](https://ctf.crate.foi.se/challenges#Triqode-299) | Övriga | 100 | 14:55 |
| [Litet Instruktionsspråk](https://ctf.crate.foi.se/challenges#Litet%20Instruktionsspr%C3%A5k-287) | Reversering | 304 | 14:55 |
| [Tidpunkt](https://ctf.crate.foi.se/challenges#Tidpunkt-296) | OSINT | 100 | 14:53 |
| [KlappKaos](https://ctf.crate.foi.se/challenges#KlappKaos-284) | Övriga | 100 | 14:42 |
| [Identitetskris](https://ctf.crate.foi.se/challenges#Identitetskris-283) | Webb | 100 | 14:42 |
| [Det Finns Ett Hus I](https://ctf.crate.foi.se/challenges#Det%20Finns%20Ett%20Hus%20I-276) | OSINT | 100 | 14:37 |
| [PROMETHEUS-IV](https://ctf.crate.foi.se/challenges#PROMETHEUS-IV-291) | Reversering | 288 | 14:35 |
| [Tjugo Ett](https://ctf.crate.foi.se/challenges#Tjugo%20Ett-297) | Övriga | 222 | 14:32 |
| [XML-kontroll 2](https://ctf.crate.foi.se/challenges#XML-kontroll%202-303) | Webb | 100 | 14:30 |
| [Den dolda kammaren](https://ctf.crate.foi.se/challenges#Den%20dolda%20kammaren-275) | Webb | 100 | 14:29 |
| [Valvet](https://ctf.crate.foi.se/challenges#Valvet-301) | Exploatering | 100 | 14:29 |
| [Tabadabort](https://ctf.crate.foi.se/challenges#Tabadabort-295) | Webb | 100 | 14:28 |
| [Komprimeringsalgoritm](https://ctf.crate.foi.se/challenges#Komprimeringsalgoritm-285) | Reversering | 100 | 14:28 |
| [Tre I Rad](https://ctf.crate.foi.se/challenges#Tre%20I%20Rad-298) | Webb | 100 | 14:27 |
| [Emojiffer](https://ctf.crate.foi.se/challenges#Emojiffer-278) | Kryptografi | 100 | 14:26 |
| [NyGammal Uppladdning](https://ctf.crate.foi.se/challenges#NyGammal%20Uppladdning-288) | Övriga | 239 | 14:26 |
| [Skrottippen](https://ctf.crate.foi.se/challenges#Skrottippen-293) | Övriga | 100 | 14:25 |
| [Dolt Budskap](https://ctf.crate.foi.se/challenges#Dolt%20Budskap-277) | Forensik | 100 | 14:24 |
| [Ex Eller Android](https://ctf.crate.foi.se/challenges#Ex%20Eller%20Android-279) | Reversering | 100 | 14:24 |
| [Förhäxande Konst](https://ctf.crate.foi.se/challenges#F%C3%B6rh%C3%A4xande%20Konst-280) | Kryptografi | 100 | 14:23 |
| [Kubnyckeln](https://ctf.crate.foi.se/challenges#Kubnyckeln-286) | Kryptografi | 100 | 14:18 |
| [Bakom Lås Och Orm](https://ctf.crate.foi.se/challenges#Bakom%20L%C3%A5s%20Och%20Orm-272) | Exploatering | 100 | 14:16 |
| [Vad heter den?](https://ctf.crate.foi.se/challenges#Vad%20heter%20den?-300) | OSINT | 100 | 14:02 |

---

*Writeups are for a finished, authorized CTF. Flags are redacted; no credentials
or infrastructure secrets are included.*
