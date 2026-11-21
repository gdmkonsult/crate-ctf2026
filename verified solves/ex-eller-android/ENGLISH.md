# Ex Eller Android

## Source writeups

This translation reconciles:

- `crate-ctf-swarm/writeups/ex-eller-android/ctf-agent11.md`
- `crate-ctf-swarm/writeups/ex-eller-android/ctf-agent21.md`

Both describe Android reverse engineering of a hardcoded XOR check.

## 1. Acquire the APK

The source refers to `app.apk`.

## 2. Decompile the APK

The exact command is:

```bash
apktool d -f app.apk -o app_decompiled
```

`-f` forces output even if the target directory exists. `-o app_decompiled` sets the decompiled output directory.

The application package is `com.example.exellervad`.

## 3. Locate application smali

Navigate to:

```text
app_decompiled/smali_classes2/com/example/exellervad/AuthManager.smali
```

The source also identifies decoy values in:

```text
FakeFlags.smali
```

The writeups say to ignore `FakeFlags.smali`; `AuthManager.smali` contains the active check.

## 4. Locate the encrypted array

In `AuthManager.smali`, locate:

```text
ENCRYPTED_FLAG
```

The source reports exactly 30 bytes.

The literal encrypted bytes are challenge-derived secret material and are replaced here with:

```text
<30 ENCRYPTED FLAG BYTES>
```

## 5. Locate and derive the key

The source locates:

```text
XOR_KEY_STR
```

and reports that it contains a hardcoded key. The literal key is a challenge credential and is represented as:

```text
<XOR_KEY>
```

Obtain the string from `XOR_KEY_STR` in `AuthManager.smali`.

For each character position `i` in that key string:

```text
derived[i] = ((ord(character[i]) + i) ^ 0x5a) & 0xff
```

No additional key derivation or external secret source is reported.

## 6. XOR-decrypt

For every encrypted byte position `i`:

```text
plaintext[i] = encrypted[i] ^ derived[i % len(derived)]
```

The derived key repeats by modulo key length.

The pseudocode in the sources is:

```python
encrypted = bytes(<30 ENCRYPTED FLAG BYTES>)

xor_key_str = "<XOR_KEY>"
derived_key = bytes([
    (ord(c) + i) ^ 0x5a
    for i, c in enumerate(xor_key_str)
])

flag = bytes([
    encrypted[i] ^ derived_key[i % len(derived_key)]
    for i in range(len(encrypted))
])

print(flag.decode("utf-8"))
```

The `& 0xff`/byte conversion is important because Java Smali's signed-byte behavior must match the Python 8-bit bytes.

## 7. Interpret the result

Decode the 30 bytes as UTF-8. The resulting plaintext is the value compared by `AuthManager.checkFlag`.

The writeup does not include a literal decrypted string.

## Recovered result

**Placeholder:** `<CHALLENGE_FLAG>`

## Known gaps

- The literal encrypted array is omitted because it contains challenge-derived data.
- The hardcoded key is omitted because it is a challenge credential.
- The source reports successful recovery but does not provide a literal output.
- No CTF-platform acceptance evidence is provided.
