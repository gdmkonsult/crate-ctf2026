# Förhäxande Konst — English

**Source reviewed:** `ctf-agent11.md`.

## Step-by-step method

1. Inspect the supplied PNG as a grid of colored tiles. The writeup records an image size of 1602×322 pixels, 80×80-pixel tiles, 20 tiles per row, and 63 tiles in total.
2. Visit the center pixel of each tile in row-major order, starting at the top-left tile. For tile column `c` and row `r`, the sample point is `(c*80+40, r*80+40)`. Apply this formula to all 63 tile positions; the source does not specify any separate treatment for the two unused tile slots in the 20-column layout.
3. Append each sampled pixel's red byte, then green byte, then blue byte before moving to the next tile. The resulting sequence is described as ASCII Base64 ending in `==`, followed by one NUL byte. The source does not document whether its Base64 decoder stopped at or ignored that trailing byte; preserve that as an implementation detail to resolve from the original artifact rather than assume a specific decoder behavior.
4. Base64-decode the ASCII data. The source reports 35 decimal values, each in the range 130–160. These are decimal byte values to invert individually, not a single decimal number.
5. Recover each character by subtracting its decimal value from 255 and interpreting the result as a character: `chr(255 - n)`.
6. As a consistency check, the source notes that a final value of 130 gives `255 - 130 = 125`, the closing-brace character. It reports that the decoded sequence forms the flag. The flag and all of its characters are omitted.

## Tools and source limitations

The source names Python and Pillow, but does not include the image or a complete script. The exact tile traversal is specified above; the actual sampled byte sequence and decoded output are not reproduced. It advises recognizing ASCII/Base64 and testing a simple byte inversion against the expected flag structure. The writeup does not identify a Base64 decoder invocation or explain its NUL handling. No platform-acceptance claim is made.

**Decoded output:** `<FLAG REDACTED>` — the source says it is obtained after applying `chr(255 - n)` to the Base64-decoded decimal values.
