# PROMETHEUS-IV — English

**Category:** Reverse engineering · **Service:** `nc challs.crate.nu 32557` · **Binary:** `main.elf` (not stripped)

A remote, menu-driven "AETHERION / PROMETHEUS-IV" satellite-control console. The win
condition is a logic flaw: `quit()` reads and prints the server-side flag when an internal
"calibration result" has become **negative**. Reaching that state requires driving the menu
through a specific telemetry-mismatch + unhealthy-sensor sequence that forces the program
into its panic path.

## 1. Reverse the win condition

`file` / `strings` / `objdump -d main.elf` reveal full symbols: `apply()`,
`valid_telemetry_state()`, `panic_mode()`, `compute_calibration()`, `apply_calibration()`,
`read_flag()`, `satellite_failure_sequence()`, `quit()`.

- **`quit()`** — if the "calibration applied" flag is set *and* the stored calibration result
  at `STATE+0x20` is **negative**, it prints `CRITICAL ERROR` and calls
  `satellite_failure_sequence()` → `read_flag()` (reads `/home/ctf/flag.txt`) → prints it to
  the client, then exits.
- **`apply()`** (Calibration menu) calls `valid_telemetry_state()`, which compares the top
  byte ("sequence") of the **committed** orbit word (`STATE+0x18`) vs the committed attitude
  word (`STATE+0x1c`). If the sequences **mismatch** *and* `sensor_healthy()` is false
  (`STATE+0x9 == 0`), it calls `panic_mode()`.
- **`panic_mode()`** overwrites telemetry with hardcoded magic values and recomputes a
  "drift" from fixed date constants (last update `99-12-31` vs current `0000-01-02` →
  drift ≈ −3649 days/10, always a large negative number). `compute_calibration()` has a
  32→64-bit zero-extension bug on that negative drift, producing a huge positive 64-bit
  multiplier that overflows the `0x7fffffff` (INT32_MAX) check and stores **−1** into
  `STATE+0x20`. That negative value guarantees `quit()` takes the flag-printing branch.

## 2. Build the required state via menu options

Two preconditions must hold when Calibration→Apply runs:

**(a) Sensor unhealthy.** `Diagnostics → Sensor Diagnostics` marks the sensor unhealthy only
if it is ONLINE *and* an internal **degradation counter > 2**. That counter increments only on
an OFF→ON transition in the `Status` menu, so you must toggle the sensor off/on **three
times** first (ending ONLINE).

**(b) Orbit/attitude sequence mismatch.**
1. `Navigation → Update Orbit` commits the *current* telemetry sequence (sequence 0).
2. `Diagnostics → Memory Diagnostics → Clean data? Yes` rewrites the live telemetry with a
   new, incremented sequence byte.
3. `Navigation → Update Attitude` commits that *new* sequence — now different from the orbit
   sequence committed in step 1.

Then `Calibration → Apply and update? → Yes` triggers `apply()` → mismatch + unhealthy →
`panic_mode()` → negative calibration → exit via `quit()` → flag printed.

## 3. Exact menu sequence (state transitions)

Each pair below is `menu, option` sent as newline-separated lines:

```text
1,1   1,2   1,1   1,2   1,1   1,2    # Status: 3× sensor OFF then ON (degradation -> 3, ONLINE)
3,1,3                                # Navigation: Update Orbit, Back        (commit orbit seq 0)
2,2,3                                # Diagnostics: Sensor Diagnostics, Back  (-> sensor UNHEALTHY)
2,1,1,3                              # Diagnostics: Memory Diagnostics -> Clean? Yes, Back (new seq)
3,2,3                                # Navigation: Update Attitude, Back     (commit new seq)
4,1                                  # Calibration: Apply and update? Yes    (-> panic_mode)
```

## 4. Automated exploit

```python
#!/usr/bin/env python3
import socket, sys

HOST, PORT = "challs.crate.nu", 32557

choices = [
    "1","1", "1","2", "1","1", "1","2", "1","1", "1","2",  # 3× sensor off/on cycles
    "3","1","3",          # Navigation: Update Orbit, Back
    "2","2","3",          # Diagnostics: Sensor Diagnostics (-> unhealthy), Back
    "2","1","1","3",      # Diagnostics: Memory Diagnostics -> Clean? Yes, Back
    "3","2","3",          # Navigation: Update Attitude, Back
    "4","1",              # Calibration: Apply and update? Yes
]

s = socket.create_connection((HOST, PORT), timeout=15)
transcript = b""

def recv_some(t=1.5):
    global transcript
    s.settimeout(t)
    try:
        while True:
            data = s.recv(4096)
            if not data:
                break
            transcript += data
    except socket.timeout:
        pass

recv_some(2)
for c in choices:
    s.sendall((c + "\n").encode())
    recv_some(1.2)
recv_some(20)   # allow panic_mode's countdown sleeps to finish
sys.stdout.buffer.write(transcript)
s.close()
```

Running this prints the countdown block (`PROMETHEUS-IV drifting` / `shutting down` /
`not responding`) followed directly by the flag line `cratectf{...}` in the PANIC MODE
output. The path is deterministic — no brute force.

## Verification

Platform-confirmed solved, and independently reproduced by replaying the identical choice
sequence against `challs.crate.nu:32557`. The literal flag is redacted for publication.
