# Concentric — English

**Category:** Cryptography. **Prompt:** "What do the circles say?"
**Status:** Solved and confirmed (team posted the full writeup; organizer accepted it).
Flag redacted here; it is the lowercase Dante/Malebolge phrase, confirmed on the platform.

## Overview

The image `challenge.png` shows six circular symbols that look like mazes. They
are not mazes. Each black ring is a letter, and the rings are read from the
inside out. The title "Concentric" is the hint.

## Dead ends (documented so they are not repeated)

- **Steganography:** the PNG has exactly three colors (white, light grey, dark
  grey) and no metadata. Nothing is hidden in the file bytes.
- **Maze solving:** tracing a path from the center outward yields nothing.
- **Bits per ring:** arcs start and end on multiples of 45°, so reading 8 bits
  per ring as ASCII is tempting, but it only produces garbage.

## Structure

- Each symbol has its own center. The light-grey guide lines mark the centers at
  `x = 162, 487, 782, 1032, 1312, 1622`, all at `y = 177`.
- Rings sit on radii `10, 25, 40, 55, …`, spaced 15 px apart.
- A letter is built from:
  - an arc on the ring, covering certain 45° sectors, and
  - radial strokes inward from the ring, at 0°, 90°, 180°, or 270°.

Unrolling each ring on its own and drawing its inward strokes makes the letters
readable (the solver's `render_rings.py` produces `rings.png`).

## Alphabet (ring shape + inward strokes → letter)

| Ring shape | Inward stroke | Letter |
|---|---|---|
| left half `(` | — | C |
| left half `(` | at 9 | E |
| right half `)` | — | D |
| right half `)` | at 3 | F |
| right half `)` | at 12 and 6 | B |
| `∩` (open downward) | at 3 and 9 | A |
| `∩` with stroke from center | at 12 | M |
| open lower-right | at 3 and 6 | R |
| open upper-right | — | L |
| open upper-right | at 3 | G |
| arc top + bottom | — | T |
| arc top + bottom | at 12 and 6 | I |
| arc left + right | at 3 and 9 | H |
| lower half `∪` | — | U |
| full ring | — | O |
| two opposite quarters | at 3 and 9 | S |
| straight horizontal chord | — | `_` |
| straight vertical chord (left/right) | — | `{` / `}` |

The chords sit at the same radial spacing as rings, so they act as their own
"rings" in the reading order.

## Decoding (inner to outer)

| Symbol | Rings | Text |
|---|---|---|
| 1 | C R A T E C T F + vertical chord | `cratectf{` |
| 2 | M A L E B O L G E + chord | `malebolge_` |
| 3 | E I G H T H + chord | `eighth_` |
| 4 | C I R C L E + chord | `circle_` |
| 5 | F R A U D + chord + T H E | `fraud_the` |
| 6 | chord + B O L G I A S + vertical chord | `_bolgias}` |

The theme confirms the reading: in Dante's Inferno, Malebolge is the eighth
circle of Hell where fraud is punished, divided into ten bolgias. The case is not
visible in the image; the flag format is lowercase.

## Result

Flag redacted (lowercase `cratectf{malebolge_...bolgias}`). Confirmed accepted on
the platform.
