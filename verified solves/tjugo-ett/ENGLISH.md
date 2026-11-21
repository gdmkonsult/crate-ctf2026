# Tjugo Ett — Detailed reconstruction

## Sources read
- `writeups/tjugo-ett/ctf-agent11.md`
- `writeups/tjugo-ett/ctf-agent13.md`

## Target
- Service: `challs.crate.nu`, port `30178`
- Game: Blackjack/Tjugo Ett
- Goal recorded by source: reach a balance of at least `$100 000`
- Menu: `1` Start, `2` Show next card, `3` Simple output
- Hand actions: `1` Hit, `2` Stand, `3` Double down

## Step 1 — Establish the oracle behavior
The source writeups report that `Show next card` returns the next card and advances or exposes one card from a predictable deck. The first source says 60 previews showed a fixed 52-card cyclic order identical across connections. The second source says 52 calls to `Show next card` reveal the complete cycle and that each preview consumes a card.

The exact reconciliation is:

- a complete cycle contains 52 unique cards;
- after 52 cards, the sequence repeats;
- use suits and card values because cards are not indistinguishable in the normal view;
- preview behavior consumes a position, so full-cycle extraction must account for that consumption before hands or hand synchronization.

## Step 2 — Understand deal order and state
Both sources give or imply this order:

1. Player receives first card.
2. Dealer receives a face-up card.
3. Player receives second card.
4. Dealer receives a hidden second card.
5. Additional cards are delivered through Hit.

Dealer behavior: stand on a total of at least `17`; do not apply a player strategy to dealer actions.

## Step 3 — Synchronize a hand to the known deck
The first source says to parse visible cards and find their position in the 52-card sequence.

The second source gives a more exact normal-output synchronization method:

- dealer line shape: `Dealer: ( d1,XX )`;
- player line shape: `Player: ( p1,p2 )`;
- suit-bearing card strings are matched against extracted sequence entries;
- find an index `i` such that:

```text
sequence[i] = p1
sequence[i+1] = d1
sequence[i+2] = p2
```

The first visible player/dealer cards are sufficient to locate the starting position only because the cards are unique in the recovered 52-card cycle.

## Step 4 — Calculate every legal outcome
The source writeups describe deterministic completion rather than a random-strategy guess:

1. Convert card values to totals, counting aces as `11` initially.
2. Reduce any ace from `11` to `1` until the player or dealer total is no longer above `21`, as applicable.
3. Resolve Stand by finishing the dealer with the known future cards and comparing totals.
4. Resolve Hit recursively: if the player exceeds `21`, the hand loses; otherwise continue until the desired terminal state is tested.
5. Resolve Double down by applying the double action and resolving against the dealer and known next card.

The dealer must continue drawing while its adjusted total is below `17`. No source provides a complete public payout table, so only the source-level outcome is documented.

## Step 5 — Choose the action
The first source says to calculate a winning line, play that line, and choose Double down whenever it wins. The second source implements the same concept: compare Stand, Hit and Double down results and select the action whose normalized result is highest.

The exact source-provided ordinary game menu tokens are preserved:

- `1` Hit
- `2` Stand
- `3` Double down

The source first agent describes betting always 50% of the money. The second agent's source code is built around fixed actions but does not restate a 50%-bet policy as a separate explanation.

## Step 6 — Repeat until threshold
The first source reports reaching `$115200`, at which point the server prints the flag. The second source loops up to 250 hands and waits for the server's flag response. These are two different stopping observations, not independent proof of the exact hand path.

The source writeups report the flag text format only as `cratectf{...}` and redact it. Represent it as `FLAG_PLACEHOLDER`.

## Source gaps/conflicts
- The 60-preview source and 52-card-extraction source describe the card oracle with similar conclusions but somewhat different extraction accounting.
- The exact 52-card order is not published.
- The full payout/split/surrender rules are not present.
- No complete platform transcript proves acceptance.

## Cyber-safety boundary
No operational bot, socket script, card-prediction automation or remote action loop is reproduced. The reconstruction is limited to the source-proven deck model and deterministic action evaluation.
