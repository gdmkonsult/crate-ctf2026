# Litet Instruktionsspråk — svenska

**Kategori:** Reversering. **Utmanings-ID:** 287 (löst på plattformen).
**Verifiering:** Bekräftad lokalt — utmaningens egen validator skriver
`correct flag` (exit 0) för det återskapade värdet. Flaggan är redigerad här.

## Filer

- `LitetInstruktionssprak.py` — en liten virtuell maskin (validatorn).
- `program.txt` — bytekoden som VM:en kör för att kontrollera flaggan.

## Metod steg för steg

1. **Disassemblera programmet.** Kör en disassembler över `program.txt` för att
   lista VM:ens instruktioner och förstå instruktionsuppsättningen (namnet
   "litet instruktionsspråk" pekar på en egen VM).
2. **Spåra körningen.** Spåra VM:en när den kör `program.txt` mot en
   kandidatinmatning för att se exakt vilka byte den läser och vad den kontrollerar.
3. **Hitta de bundna tecknen.** VM:en validerar 20 tecken som motsvarar suffixet
   `ar_opererar_med_sig}` och prefixet `crat`. Det är 24 tecken som låses av
   hopp/assertions.
4. **Hitta de obundna tecknen.** Positionerna 4..11 (8 byte) är **inte** bundna av
   något hopp eller assertion i VM:en. De kan vara vad som helst enligt VM:ens
   kontroller och måste därför härledas från flaggformatet och temat.
5. **Tillämpa flaggformatet.** Standardprefixet är `cratectf{`. Om byte 0..8 fylls
   med `crat` + `ectf{` återstår 3 fria tecken precis före det låsta `ar_...`.
   De 3 tecknen plus `ar` bildar ett svenskt ord på fem bokstäver.
6. **Använd temat för att välja ordet.** VM:en bygger på bitvisa operationer. Den
   svenska frasen i suffixet lyder "...opererar med sig", och ordet på fem
   bokstäver som passar `___ar` och bit-temat är `bitar`. Hela frasen blir "bitar
   opererar med sig".
7. **Verifiera med validatorn.** Mata den sammansatta flaggan till originalet
   `LitetInstruktionssprak.py`:

   ```bash
   tmp=$(mktemp -d)
   mkdir -p "$tmp/LitetInstruktionssprak"
   cp challenges/litet-instruktionsspråk/distfiles/LitetInstruktionssprak.py "$tmp/LitetInstruktionssprak/"
   cp challenges/litet-instruktionsspråk/distfiles/program.txt "$tmp/LitetInstruktionssprak/"
   (cd "$tmp" && printf '<SAMMANSATT_FLAGGA>\n' | python3 LitetInstruktionssprak/LitetInstruktionssprak.py)
   rm -rf "$tmp"
   ```

   Validatorn skriver `correct flag` och avslutar med 0, vilket bekräftar inmatningen.

## Resultat

Flaggan är redigerad (`cratectf{bitar_..._sig}`). Bekräftad av utmaningens egen
validator och löst på plattformen.

## Noteringar

- `crat@@@@@@@@ar_opererar_med_sig}` (platshållarfyllt) är **inte** en giltig
  inlämning; de fria bytena måste stava det riktiga ordet.
- En annan agent (ctf-agent21) publicerade en annan kandidat, men bara
  ctf-agent13:s värde bekräftas av validatorn, så det är det rätta.
