# PROMETHEUS-IV — Svenska

**Kategori:** Reverse engineering · **Tjänst:** `nc challs.crate.nu 32557` · **Binär:** `main.elf` (ej strippad)

En fjärrstyrd, menydriven satellitkonsol "AETHERION / PROMETHEUS-IV". Vinstvillkoret är ett
logikfel: `quit()` läser och skriver ut den serversidiga flaggan när ett internt
"kalibreringsresultat" har blivit **negativt**. För att nå det tillståndet måste menyn drivas
genom en specifik sekvens med telemetri-missmatchning + ohälsosam sensor som tvingar
programmet in i sin panikväg.

## 1. Reversa vinstvillkoret

`file` / `strings` / `objdump -d main.elf` visar fullständiga symboler: `apply()`,
`valid_telemetry_state()`, `panic_mode()`, `compute_calibration()`, `apply_calibration()`,
`read_flag()`, `satellite_failure_sequence()`, `quit()`.

- **`quit()`** — om flaggan "kalibrering tillämpad" är satt *och* det lagrade
  kalibreringsresultatet vid `STATE+0x20` är **negativt**, skrivs `CRITICAL ERROR` ut och
  `satellite_failure_sequence()` → `read_flag()` (läser `/home/ctf/flag.txt`) → skrivs ut
  till klienten, varpå programmet avslutas.
- **`apply()`** (Calibration-menyn) anropar `valid_telemetry_state()`, som jämför den
  översta byten ("sequence") i det **committade** orbit-ordet (`STATE+0x18`) mot det
  committade attitude-ordet (`STATE+0x1c`). Om sekvenserna **inte matchar** *och*
  `sensor_healthy()` är falskt (`STATE+0x9 == 0`), anropas `panic_mode()`.
- **`panic_mode()`** skriver över telemetrin med hårdkodade magiska värden och räknar om en
  "drift" från fasta datumkonstanter (senaste uppdatering `99-12-31` vs nuvarande
  `0000-01-02` → drift ≈ −3649 dagar/10, alltid ett stort negativt tal). `compute_calibration()`
  har en 32→64-bitars nollförlängningsbugg på den negativa driften, vilket ger en enorm
  positiv 64-bitars multiplikator som spiller över `0x7fffffff`-kontrollen (INT32_MAX) och
  lagrar **−1** i `STATE+0x20`. Det negativa värdet garanterar att `quit()` tar grenen som
  skriver ut flaggan.

## 2. Bygg det nödvändiga tillståndet via menyval

Två förvillkor måste gälla när Calibration→Apply körs:

**(a) Sensorn ohälsosam.** `Diagnostics → Sensor Diagnostics` markerar sensorn som ohälsosam
endast om den är ONLINE *och* en intern **degraderingsräknare > 2**. Räknaren ökar bara vid
en OFF→ON-övergång i `Status`-menyn, så du måste först växla sensorn av/på **tre gånger**
(och avsluta i ONLINE).

**(b) Missmatchad orbit-/attitude-sekvens.**
1. `Navigation → Update Orbit` committar den *nuvarande* telemetrisekvensen (sekvens 0).
2. `Diagnostics → Memory Diagnostics → Clean data? Yes` skriver om den levande telemetrin med
   en ny, inkrementerad sekvensbyte.
3. `Navigation → Update Attitude` committar den *nya* sekvensen — nu annorlunda än
   orbit-sekvensen från steg 1.

Därefter utlöser `Calibration → Apply and update? → Yes` `apply()` → missmatch + ohälsosam →
`panic_mode()` → negativ kalibrering → avslut via `quit()` → flaggan skrivs ut.

## 3. Exakt menysekvens (tillståndsövergångar)

Varje par nedan är `meny, val` skickat som radseparerade rader:

```text
1,1   1,2   1,1   1,2   1,1   1,2    # Status: 3× sensor OFF sedan ON (degradation -> 3, ONLINE)
3,1,3                                # Navigation: Update Orbit, Back        (commit orbit seq 0)
2,2,3                                # Diagnostics: Sensor Diagnostics, Back  (-> sensor UNHEALTHY)
2,1,1,3                              # Diagnostics: Memory Diagnostics -> Clean? Yes, Back (ny seq)
3,2,3                                # Navigation: Update Attitude, Back     (commit ny seq)
4,1                                  # Calibration: Apply and update? Yes    (-> panic_mode)
```

## 4. Automatiserad exploit

```python
#!/usr/bin/env python3
import socket, sys

HOST, PORT = "challs.crate.nu", 32557

choices = [
    "1","1", "1","2", "1","1", "1","2", "1","1", "1","2",  # 3× sensor av/på-cykler
    "3","1","3",          # Navigation: Update Orbit, Back
    "2","2","3",          # Diagnostics: Sensor Diagnostics (-> ohälsosam), Back
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
recv_some(20)   # låt panic_modes nedräknings-sleeps bli klara
sys.stdout.buffer.write(transcript)
s.close()
```

Körningen skriver ut nedräkningsblocket (`PROMETHEUS-IV drifting` / `shutting down` /
`not responding`) följt direkt av flaggraden `cratectf{...}` i PANIC MODE-utskriften. Vägen
är deterministisk — ingen brute force.

## Verifiering

Plattformsbekräftat löst, och oberoende reproducerat genom att spela upp den identiska
valsekvensen mot `challs.crate.nu:32557`. Den bokstavliga flaggan är dold för publicering.
