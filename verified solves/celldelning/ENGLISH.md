# Celldelning

## Source writeups

This translation reconciles:

- `crate-ctf-swarm/writeups/celldelning/ctf-agent11.md`
- `crate-ctf-swarm/writeups/celldelning/ctf-agent13.md`
- `crate-ctf-swarm/writeups/celldelning/ctf-agent17.md`
- `crate-ctf-swarm/writeups/celldelning/ctf-agent21.md`

All four agree on the same broad heap exploit chain: stale-slot UAF, largebin pointer write, fake FILE in a victim chunk, and shell recovery of the `FLAG` environment variable.

## 1. Inspect the binary interface

The source code/protocol facts are:

- the menu provides six append-only cell slots;
- allocations range from `0x40f` or `0x410` through `0x510`;
- create, edit, show, delete, and exit/win functions are menu operations;
- deleted slots retain pointers;
- edit checks the slot and requested bounds but not whether the slot is occupied;
- edit consumes `len+1` bytes because it also consumes protocol newline/input;
- the bundled libc is glibc `2.39`;
- the process exits with `_exit()`, so normal `exit`/FILE flush paths are not relied upon.

The source bin is PIE and unstripped, but ASLR/heap values are still recovered at runtime.

## 2. Establish the allocation layout

Agent17 describes:

1. allocate a `0x428` chunk;
2. allocate a guard-sized `0x418` chunk;
3. allocate a second `0x418` chunk;
4. delete the first chunk;
5. recreate an allocation of `0x428`.

The recreated chunk reuses slot state and shows freed-heap metadata to stale `show(3)` access.

The four writeups use slightly different requested sizes and names, but agree on a 0x428/0x418/0x418 guard layout followed by largebin sorting.

## 3. Leak libc

Freeing and recreating the first chunk places it in an unsorted-bin state.

A stale `show` read from the freed chunk exposes an unsorted-bin pointer.

The source reports:

```text
libc = unsorted_leak - 0x203b20
```

The value `0x203b20` is the agent's glibc `2.39` unsorted-bin anchor offset.

Agent11 warns not to strip leading spaces from output because byte parsing is used as a little-endian pointer and leading spaces in the response can change conversion.

## 4. Prepare and leak heap/largebin state

The source frees the stale first chunk again and requests a `0x510` allocation. This sorts the chunk through the largebin logic and creates the layout used for subsequent pointer leaks.

Then it:

1. creates or reuses the stale slot;
2. writes a 16-byte marker into index `0`;
3. reads the stale/reused slot;
4. identifies the following largebin/heap pointer as `chunk_a`;
5. restores `fd`, `bk`, `fd_nextsize`, and `bk_nextsize` metadata before allocator operations continue.

The `chunk_a` pointer is the heap address needed to locate the fake FILE structure.

## 5. Build the victim relationship

The source uses a guard and a victim chunk of the same allocator size. It calls the victim `B` and calculates:

```text
B_header = chunk_a + 0x850
```

This relationship is derived from the writeups' same-size chunk layout and the location of `B` in the allocations.

The exact heap addresses are runtime-dependent and are not reproduced in this document.

## 6. Place the shell text

Before freeing `B`, the source writes shell text into its usable layout. Agent13 describes using guard offset `0x410` so the command lands in `B.prev_size` and leaves `B.size` untouched.

Agent17 says that the guard has the same 0x420 actual chunk size as `B`, allowing a command like shell startup text to land in the previous-size field.

This phase establishes the fake FILE and command argument before the pointer-write trigger.

## 7. Construct the fake wide FILE

The writeups agree on these high-level structures:

- `B_header` is the fake FILE root;
- `_IO_wfile_jumps` is used as the top-level vtable;
- `wide_data` points to a structure inside `B_header`;
- the wide-data structure's `_wide_vtable` points back into `B_header`;
- the wide-vtable `__doallocate` callback is set to `system`;
- lock data points to writable memory in the chunk layout.

Exact runtime addresses are derived from libc and `chunk_a`. The source records the key offsets:

```text
libc + 0x202228  # _IO_wfile_jumps
libc + 0x58750   # __doallocate = system
libc + 0x2046a8  # stdout variable
```

One writeup places the wide-vtable and wide-data structure at offsets within the 0x418 victim area. Another describes those fields generally rather than by every byte.

These offsets are glibc-build facts, not portable values.

## 8. Poison the largebin victim

The source edits the stale first chunk's metadata. Agent17 identifies the relevant stale field as `bk_nextsize` at offset `0x18`.

The intended pointer target is the libc `stdout` variable:

```text
stdout_variable = libc + 0x2046a8
```

The source poisons `bk_nextsize` to point at a location that is `0x20` before the intended target. Because this is a heap exploit, exact packed payload bytes and overwrite sequence are intentionally omitted.

The smaller-victim largebin insertion then writes the victim chunk address into the `stdout` variable. The source reports:

```text
stdout variable -> B_header
```

This is the `stdout` variable, not the complete FILE structure.

## 9. Trigger FILE dispatch

The source frees `B`, performs the poisoned smaller-victim insertion through another menu create operation, and waits for the following menu output.

The next menu `puts`/`printf` dispatches through the redirected output object:

```text
puts/printf
  -> wide FILE handling
  -> _IO_wdoallocbuf
  -> __doallocate
  -> system(shell command)
```

The writeups agree that `_IO_list_all` and exit-flush triggers are not used because `_exit()` is the process's actual termination path.

Because this is a pwn/heap exploit, exact corruption payloads and socket-send code are omitted.

## 10. Recover the flag through the shell

After the shell greeting appears on the same socket, send one bounded command:

```sh
printf '__CELL_RESULT_%s__\n' "$FLAG"; exit
```

The shell variable `FLAG` is read by this command. Marker text identifies the result in the socket response.

The source local-run test values and remote output differ. Agent11 states that it did a local dummy `FLAG` sanity test and one remote run. Agent13 says the same protocol recovered a remote environment value. Agent17 reports three independent remote captures and an NFS event. Agent21 reports five local callback runs and one bounded remote attempt.

## Recovered result

The shell prints the challenge environment variable.

**Placeholder:** `<CHALLENGE_FLAG>`

It is obtained from `"$FLAG"` after the trigger opens the shell, not from a static file or direct `win()` function call.

## Gaps and conflicts

- The reports agree on glibc 2.39, the stale UAF, largebin write, fake FILE, and `system` callback.
- Agent11 gives the most complete script and local/remote protocol details.
- Agent13 gives a concise structural explanation.
- Agent17 gives a more detailed reproducible layout and notes that several alternative triggers are dead.
- Agent21 gives the shortest overview and no complete remote script.
- No source reproduces the literal candidate value.
- No CTF-platform acceptance event is provided.
