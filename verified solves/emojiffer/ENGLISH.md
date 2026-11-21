# Emojiffer

## Source writeups

This translation reconciles:

- `crate-ctf-swarm/writeups/emojiffer/ctf-agent11.md`
- `crate-ctf-swarm/writeups/emojiffer/ctf-agent21.md`

Both identify a monoalphabetic substitution cipher in which every emoji represents one fixed plaintext letter or space.

## 1. Read and strip the file header

Open `ciphertext.txt` and locate:

```text
Language=English, Text=
```

Use the text after `Text=` as the ciphertext. Preserve emoji boundaries exactly.

## 2. Count the symbol set

The source reports 27 distinct emoji symbols:

- 26 represent the lowercase alphabet;
- one represents space.

This fixed one-to-one mapping is the monoalphabetic substitution model.

## 3. Identify space

Agent21 reports that `🤟` occurs `3,359` times, about `16%` of the text. It therefore maps `🤟` to space.

That assignment follows word distribution and is consistent with the source's word-splitting approach.

## 4. Identify the first common word

The frequent three-symbol sequence:

```text
🐹🍰🦁
```

occurs `243` times and is identified as `the`.

This establishes:

```text
🐹 -> t
🍰 -> h
🦁 -> e
```

## 5. Identify the first sentence/topic word

The source maps:

```text
🎁🦊🌟🐶🐹🍇🍔🦊🦓🐶🍰🌟
```

to:

```text
cryptographer
```

Therefore:

```text
🎁 -> c
🦊 -> r
🌟 -> y
🐶 -> p
🍇 -> o
🍔 -> g
🦓 -> a
```

The sequence `🐹` is already known as `t`, and `🍰`/`🌟` are known as `h`/`y`.

## 6. Extend the mapping through common words

Agent21 gives this complete mapping set:

```text
🤟 -> space
🐹 -> t
🍰 -> h
🦁 -> e
🎁 -> c
🦊 -> r
🌟 -> y
🐶 -> p
🍇 -> o
🍔 -> g
🦓 -> a
🍓 -> i
🌭 -> m
🎈 -> n
🥗 -> f
🍎 -> v
✨ -> b
🍕 -> k
🐭 -> l
🍜 -> d
🐻 -> u
🎂 -> x
🐱 -> q
🍣 -> z
🍌 -> j
```

The remaining unmapped symbol, if any, is the source's remaining letter `s` or `w`, but agent11 and agent21 do not present one unambiguously.

The source confirms the mapping from recognizable words:

- `🍓` appears in `cipher`;
- `🌭` appears in compound words and `common`;
- `🎈` appears in `from`/`known`;
- `🥗` maps to `f`;
- `🍎` maps to `v`;
- `✨` maps to `b`;
- `🍕` maps to `k`;
- `🐭` maps to `l`;
- `🍜` maps to `d`;
- `🐻` maps to `u`;
- `🎂` maps to `x`;
- `🐱` maps to `q`;
- `🍣` maps to `z`;
- `🍌` maps to `j`.

## 7. Independent hill-climb method

Agent11 describes an independent resolution route:

1. start with 27 distinct symbols;
2. preserve space because it has the highest-frequency symbol;
3. use quadgram log-probabilities;
4. use Python `pydoc` topic text as the language corpus;
5. swap pairs of symbol-to-letter assignments;
6. accept swaps that improve the n-gram score;
7. repeat until the text is readable;
8. extract the inserted Swedish text.

The source calls this a monoalphabetic hill climb. It does not provide the corpus code or final key.

## 8. Decrypt the text

For every emoji in the text, replace it with the mapped plaintext character. The source reports plaintext as a history of cryptography article.

Agent21's substitution loop is:

```python
plaintext = ''.join(mapping.get(c, '?') for c in emoji_text)
```

The `?` marks a symbol not found in the partial mapping. It is not evidence that the final plaintext is unknown: the source uses the mapping to verify recognizable words.

## 9. Extract the hidden flag

The source reports that an inserted Swedish phrase is present:

```text
unsuited to pen and paper
flaggan ar ...
```

The exact missing text is replaced here. The source says it is the 15-character inner flag matching:

```text
cratectf{XXXXXXXXXXXXXXX}
```

The inserted phrase is described as “flaggan är …” in Swedish.

## Recovered result

**Placeholder:** `<CHALLENGE_FLAG>`

## Known gaps

- Agent11 gives no complete key or literal plaintext.
- Agent21 gives mappings but no exact final recovered key and no exact literal flag.
- The sources do not unambiguously document the exact `u`/`w`/`s` assignments or provide an unredacted final key.
- No CTF-platform acceptance result is included.

## Resolution (from agent work folders)

Both agents' published `flag.yaml` files hold the **same** flag value; the earlier
"conflict" was only because one stored the value without the `cratectf{}` wrapper.
The flag is a short Swedish phrase (“massor med emojis”); the literal value is
redacted here. The challenge is solved on the platform.
