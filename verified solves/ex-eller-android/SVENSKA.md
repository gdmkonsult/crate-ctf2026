# Ex Eller Android

## Källskrivningar

Denna översättning sammanväger:

- `crate-ctf-swarm/writeups/ex-eller-android/ctf-agent11.md`
- `crate-ctf-swarm/writeups/ex-eller-android/ctf-agent21.md`

Båda beskriver Android-reverse engineering av ett hårdkodat XOR-värde.

## 1. Skaffa APK-filen

Källan hänvisar till `app.apk`.

## 2. Dekompilera APK-filen

Exakt kommando:

```bash
apktool d -f app.apk -o app_decompiled
```

`-f` tvingar utdatan även om målkatalogen finns. `-o app_decompiled` anger dekompileringskatalogen.

Applikationspaketet är `com.example.exellervad`.

## 3. Lokalisera applikationens smali

Gå till:

```text
app_decompiled/smali_classes2/com/example/exellervad/AuthManager.smali
```

Källan identifierar även vilseledande värden i:

```text
FakeFlags.smali
```

Skrivningarna säger att `FakeFlags.smali` ska ignoreras; `AuthManager.smali` innehåller den aktiva kontrollen.

## 4. Lokalisera den krypterade arrayen

I `AuthManager.smali` lokaliseras:

```text
ENCRYPTED_FLAG
```

Källan anger exakt 30 byte.

De bokstavliga krypterade byten är utmaningsberoende hemligt material och ersätts här med:

```text
<30 KRYPTERADE FLAGGBYTE>
```

## 5. Lokalisera och härled nyckeln

Källan hittar:

```text
XOR_KEY_STR
```

och anger att den innehåller en hårdkodad nyckel. Den bokstavliga nyckeln är en utmaningshemlighet och visas som:

```text
<XOR-NYCKEL>
```

Hämta strängen från `XOR_KEY_STR` i `AuthManager.smali`.

För varje teckenposition `i` i nyckelsträngen:

```text
derived[i] = ((ord(tecken[i]) + i) ^ 0x5a) & 0xff
```

Ingen ytterligare nyckelderivering eller extern hemlighetskälla anges.

## 6. XOR-dekryptera

För varje krypterad byteposition `i`:

```text
klartext[i] = krypterad[i] ^ härledd[i % len(härledd)]
```

Den härledda nyckeln upprepas enligt nyckellängdens modulo.

Skrivningarnas pseudokod är:

```python
encrypted = bytes(<30 KRYPTERADE FLAGGBYTE>)

xor_key_str = "<XOR-NYCKEL>"
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

`& 0xff`/bytekonverteringen är viktig eftersom Java Smalis signerade bytebeteende måste matchas av Python 8-bitars bytes.

## 7. Tolka resultatet

Avkoda de 30 byten som UTF-8. Den resulterande klartexten är det värde som `AuthManager.checkFlag` jämför.

Skrivningen innehåller inte den bokstavliga dekrypterade strängen.

## Återvunnet resultat

**Platshållare:** `<UTMANINGSFLAGGA>`

## Kända luckor

- Den bokstavliga krypterade arrayen utelämnas eftersom den innehåller utmaningsberoende data.
- Den hårdkodade nyckeln utelämnas eftersom den är en utmaningshemlighet.
- Källan rapporterar lyckad återställning men inte bokstavlig output.
- Ingen CTF-plattformsgodkännande ges.
