# Tjugo Ett — Detaljerad rekonstruktion

## Källor som lästs
- `writeups/tjugo-ett/ctf-agent11.md`
- `writeups/tjugo-ett/ctf-agent13.md`

## Mål
- Tjänst: `challs.crate.nu`, port `30178`
- Meny: `1` Start, `2` Show next card, `3` Simple output
- Hand: `1` Hit, `2` Stand, `3` Double down
- Källan anger målbalans minst `$100 000`.

## Steg 1 — Kartlägg kortoraklet
En källa använde 60 preview-anrop. Den andra säger att 52 anrop till `Show next card` ger hela cykeln och att varje preview förbrukar ett kort.

Gemensam slutsats:
- kortleken är 52 kort;
- sekvensen är cyklisk;
- sekvensen är identisk mellan anslutningar;
- kort och suit gör varje kort unikt.

## Steg 2 — Dealordning
1. spelare får kort;
2. dealer får synligt kort;
3. spelare får andra kort;
4. dealer får dolt kort;
5. additional cards kommer via Hit.

Dealern stannar på 17 eller mer.

## Steg 3 — Synka handen
Båda källorna hittar kortens position i återvunna sekvensen. Normalutgången är i en källa:

```text
Dealer: ( d1,XX )
Player: ( p1,p2 )
```

Matcha index så att:

```text
sequence[i] = p1
sequence[i+1] = d1
sequence[i+2] = p2
```

Därmed är nästa kort kända i den återvunna cykeln.

## Steg 4 — Beräkna utfall
För varje möjlig handling:

1. omvandla kort till totaler;
2. räkna ess som 11 först;
3. minska ess från 11 till 1 tills totalen inte överstiger 21 när det behövs;
4. låt dealern dra kort tills minst 17;
5. jämför Hit, Stand och Double down mot nästa förutsägbara kort;
6. välj handling med högst utfall.

## Steg 5 — Spela
Källan beskriver att välja vinnande linje och använda Double down när det vinner. Första källan anger också 50 % av saldo som satsning.

Andra källan återger en loop på upp till 250 rundor och söker serverns successresultat. När målet är nått skriver servern källornas återgivna flaggtext; första källan anger observationsbalans `$115200`.

## Luckor
Kortordningen publiceras inte, 60-preview- och 52-card-källorna räknar consumption olika och exakt full payout-regel saknas.

**Säkerhetsgräns:** Ingen kortbot eller socketautomation återges.
