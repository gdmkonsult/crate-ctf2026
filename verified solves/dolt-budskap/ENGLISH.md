# Dolt Budskap

## Source writeup

This document translates:

- `crate-ctf-swarm/writeups/dolt-budskap/ctf-agent11.md`

The source provides one end-to-end forensics procedure.

## 1. Inspect MP3 metadata

Run:

```bash
exiftool file.mp3
```

The source reports two comments:

```text
The waveform isn't the whole picture
compressed in the end under pressure
```

The first comment directs attention away from ordinary waveform content. The second directs attention to an archive embedded/compressed at the end of the file.

## 2. Render a spectrogram

Run the exact source command:

```bash
ffmpeg -i file.mp3 -lavfi showspectrumpic=s=1600x800:legend=0:scale=log spec.png
```

Break down the parameters:

- `-i file.mp3` selects the input;
- `-lavfi` supplies a filter graph;
- `showspectrumpic` creates the spectrogram image;
- `s=1600x800` makes the output `1600 × 800`;
- `legend=0` disables the legend;
- `scale=log` uses a logarithmic frequency scale;
- `spec.png` is the output filename.

Open `spec.png`. The source reports that the image visibly shows:

```text
password:<ZIP_PASSWORD>
```

The literal password is challenge-derived and is redacted here.

This password is used for the appended AES archive. It is not a direct answer and must not be confused with the recovered flag.

## 3. Scan for embedded file signatures

Run:

```bash
binwalk file.mp3
```

The source reports that `binwalk` identifies an AES-encrypted ZIP at absolute offset:

```text
363267
```

The offset is the signature location in `file.mp3`. It is why the extraction starts with the byte immediately after offset `363267`.

## 4. Extract only the appended archive

Use `tail` to copy every byte after offset `363267`:

```bash
tail -c +363268 file.mp3 > h.zip
```

The reason for `+363268` is exact: if a ZIP signature begins at offset `363267`, byte indexing starts one byte later than that offset for `tail -c +N`, so `363268` starts with the ZIP data in the source's interpretation.

The output file is named `h.zip`.

## 5. Decrypt and extract the ZIP

Use 7-Zip because the source reports an AES-encrypted ZIP:

```bash
7z x -p<ZIP_PASSWORD> h.zip
```

Replace `<ZIP_PASSWORD>` with the password visually recovered from `spec.png`. Do not substitute the flag or a guessed password.

The extraction is where the source's `flag.txt` is restored.

The source explicitly says that plain `unzip` fails on this AES-encrypted archive, so 7-Zip/AES support is required.

## 6. Read the recovered file

Run:

```bash
cat flag.txt
```

The file contains the challenge flag in text form.

## Recovered result

The literal password and literal flag are omitted.

**Placeholders:**

- password: `<ZIP_PASSWORD>`
- flag: `<CHALLENGE_FLAG>`

The flag is read from `flag.txt` after `7z` extracts `h.zip`.

## Known gaps

- The source gives no screenshot copy of `spec.png`.
- The source gives no literal ZIP password.
- The source gives no independent platform acceptance.
