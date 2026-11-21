# Celldelning

## Källskrivningar

Denna översättning sammanväger:

- `crate-ctf-swarm/writeups/celldelning/ctf-agent11.md`
- `crate-ctf-swarm/writeups/celldelning/ctf-agent13.md`
- `crate-ctf-swarm/writeups/celldelning/ctf-agent17.md`
- `crate-ctf-swarm/writeups/celldelning/ctf-agent21.md`

Alla fyra är överens om huvudkedjan: stale-slot UAF, largebin-pekar skrivning, fejkad FILE i victim-chunk och skalåterställning av miljövariabeln `FLAG`.

## 1. Inspektera binärens gränssnitt

Källfakta:

- menyn har sex append-only cell-slotar;
- allokeringar går från `0x40f` eller `0x410` till `0x510`;
- create, edit, show, delete och exit/win är menyoperationer;
- borttagna slotar behåller pekare;
- edit kontrollerar slot och requested bounds men inte om sloten är ledig;
- edit konsumerar `len+1` byte eftersom protokollnewlinjen/indata också tas emot;
- medföljande libc är glibc `2.39`;
- processen avslutas med `_exit()`, så vanliga exit/FILE-flushvägar används inte.

Binären är PIE och ostrippad, men ASLR/heapvärden tas fortfarande fram vid körning.

## 2. Fastställ allokeringslayouten

Agent17 beskriver:

1. allokera en `0x428`-chunk;
2. allokera en vaktchunk på `0x418`;
3. allokera en andra `0x418`-chunk;
4. frigör den första chunken;
5. återskapa en allokering på `0x428`.

Den återskapade chunken återanvänder slotläge och visar frigiven heapmetadata för stale `show(3)`-åtkomst.

De fyra skrivningarna använder något olika begärda storlekar och namn men är överens om layouten 0x428/0x418/0x418 och därefter largebin-sortering.

## 3. Läcka libc

Att frigöra och återskapa den första chunken placerar den i unsorted-bin-läge.

En stale `show`-läsning från den frigivna chunken avslöjar en unsorted-bin-pekare.

Källan anger:

```text
libc = unsorted_leak - 0x203b20
```

Värdet `0x203b20` är agenternas glibc `2.39`-offset för unsorted-bin-ankaret.

Agent11 varnar för att inte ta bort ledande mellanslag från utskriften, eftersom ledande mellanslag kan påverka pointertolkning.

## 4. Förbered och läck heap/largebin-läget

Källan frigör den stale första chunken igen och begär en `0x510`-allokering. Det sorterar chunken genom largebin-logiken och skapar layouten för efterföljande heapläckage.

Sedan:

1. skapas eller återanvänds stale-sloten;
2. skrivs en 16-byte-markör i index `0`;
3. läses den stale/återanvända sloten;
4. identifieras följande largebin/heappointer som `chunk_a`;
5. återställs `fd`, `bk`, `fd_nextsize` och `bk_nextsize` innan allocatoroperationer fortsätter.

Pekaren `chunk_a` är heapadressen som används för att placera den fejkade FILE-strukturen.

## 5. Bygg victimrelationen

Källan använder en vakt och en victim-chunk av samma allocatorstorlek. Den kallar victimen `B` och beräknar:

```text
B_header = chunk_a + 0x850
```

Denna relation härleds från skrivningarnas likstorlekslayout och positionen för `B`.

Exakta heap-adresser är runtimeberoende och återges inte här.

## 6. Placera skaltexten

Före frigivning av `B` skriver källan skaltext i den användbara layouten. Agent13 beskriver att vaktoffset `0x410` används så att kommandot hamnar i `B.prev_size` och `B.size` lämnas orört.

Agent17 säger att vakten har samma faktiska 0x420-chunkstorlek som `B`, vilket gör att en kommandotext kan hamna i previous-size-fältet.

Detta steg etablerar den fejkade FILE-strukturen och kommandargumentet före triggningsfasen.

## 7. Konstruera den fejkade wide FILE

Skrivningarna är överens om:

- `B_header` är roten för fejkad FILE;
- `_IO_wfile_jumps` används som toppnivå-vtable;
- `wide_data` pekar på en struktur i `B_header`;
- wide-datastrukturen `_wide_vtable` pekar tillbaka i `B_header`;
- wide-vtablens `__doallocate` sätts till `system`;
- lockdata pekar på skrivbart minne i chunklayouten.

Exakta runtime-adresser härleds från libc och `chunk_a`. Källan anger centrala offset:

```text
libc + 0x202228  # _IO_wfile_jumps
libc + 0x58750   # __doallocate = system
libc + 0x2046a8  # stdout-variable
```

En skrivning placerar wide-vtable och wide-data inom 0x418-victimområdet. En annan beskriver fälten mer generellt än med varje byteoffset.

Dessa offset är glibc-buildspecifika och inte portabla.

## 8. Förgifta largebin-victimen

Källan redigerar stale-chunken i metadatafältet. Agent17 identifierar relevant fält som `bk_nextsize` på offset `0x18`.

Avsett pekarmål är libc-variabeln `stdout`:

```text
stdout_variable = libc + 0x2046a8
```

Källan förgifter `bk_nextsize` så det pekar på en plats `0x20` före avsett mål. Eftersom detta är heap-exploatering utelämnas exakta packade payloadbyte och skrivningsordning.

Den mindre victim-insättningen i largebin skriver därefter victim-chunkens adress till `stdout`-variabeln. Källan anger:

```text
stdout-variabel -> B_header
```

Det är `stdout`-variabeln, inte hela FILE-strukturen.

## 9. Trigga FILE-dispatch

Källan frigör `B`, utför den förgiftade mindre victim-insättningen via en ny create-menyoperation och väntar på nästa menyutskrift.

Nästa `puts`/`printf` dispachar via den omdirigerade outputobjektet:

```text
puts/printf
  -> wide FILE-hantering
  -> _IO_wdoallocbuf
  -> __doallocate
  -> system(skalkommando)
```

Skrivningarna är överens om att `_IO_list_all` och exit-flush inte används, eftersom processen avslutas via `_exit()`.

Eftersom detta är en pwn/heap-utnyttjandebeskrivning utelämnas exakta korruptionspayload och socket-send-kod.

## 10. Hämta flaggan via skalet

När skalprompten visas på samma socket skickas ett begränsat kommando:

```sh
printf '__CELL_RESULT_%s__\n' "$FLAG"; exit
```

Skalvariabeln `FLAG` läses av detta kommando. Markörtext identifierar resultatet i socketsvaret.

Lokala testvärden och fjärrutdata skiljer sig. Agent11 beskriver en lokal dummy `FLAG`-kontroll och en fjärrkörning. Agent13 säger att samma protokoll återvann ett fjärrmiljövärde. Agent17 rapporterar tre oberoende fjärrfångster och en NFS-händelse. Agent21 rapporterar fem lokala callback-körningar och ett begränsat fjärrförsök.

## Återvunnet resultat

Skalet skriver ut utmaningens miljövariabel.

**Platshållare:** `<UTMANINGSFLAGGA>`

Den hämtas från `"$FLAG"` efter att triggen startar skalet, inte från en statisk fil eller direkt `win()`-anrop.

## Luckor och konflikter

- Rapporterna är överens om glibc 2.39, stale UAF, largebin-skrivning, fejkad FILE och `system`-callback.
- Agent11 ger mest komplett skript och lokala/fjärrdetaljer.
- Agent13 ger en kort strukturell förklaring.
- Agent17 ger en mer detaljerad layout och konstaterar att flera alternativa triggers är döda.
- Agent21 ger kortast översikt och inget komplett fjärrskript.
- Ingen källa återger kandidatens bokstavliga värde.
- Ingen CTF-plattformsgodkännande finns.
