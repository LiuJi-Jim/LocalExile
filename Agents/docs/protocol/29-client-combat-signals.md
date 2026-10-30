# 29. Client combat signals and resource authority (research)

This page separates incoming combat feedback, resource totals and stat inputs.
It documents client 3.29.3b readers and consumers, not an official-server damage algorithm.
See [client damage calculation](../re/client-damage-calculation.md) for artifact hashes, formula evidence and open tasks.
No client execution or new packet capture was performed for this static audit.

## Message roles and established limits

| S→C message | Established client path | Not established by that message alone |
|---|---|---|
| `0x1af` (decimal 431) | Actor result record, conditional source/target feedback signals | Per-hit damage amount, complete miss/block/crit taxonomy, local numerical pre-deduction |
| `0x1b1` (decimal 433) | Assign current/reserved/flags for Life, Mana or ES | Which hit/source caused the difference, full rollback/replay, server tick cadence |
| `0x1b3` (decimal 435) | Replace explicit stat values through the network override getter | Raw-input recomputation of every omitted derived output |

An incoming Life difference is a resource observation, not hit attribution or kill credit.
A Render or sound callback is a presentation observation, not proof of a client damage roll.
The generated [schema](schema.md) can leave these factories unresolved; that is missing generator coverage.
It does not invalidate a separately audited concrete reader.

## `0x1af`: read sequence

Reader `0x141fddba0` calls common header reader `0x141f92000`, then tail-jumps to `0x1419bd6f0`
with the decoded message's `+0x20` record as its destination.
`0x1419bd6f0` first calls `0x14195dde0` on record `+0x40`.
The sequence below excludes the opcode and connection framing.
Multi-byte scalar fields are big-endian on the wire.
Names such as `record+0x36` are **decoded memory offsets**, not wire offsets or recovered semantic names.

| Read order | Wire field | Decoded destination / condition |
|---|---|---|
| 1 | u32 entity | Common world-message header |
| 2 | u32 unnamed | Common header `+0x14` |
| 3 | u16 unnamed | Common header `+0x18` |
| 4 | u16 | record `+0x40` |
| 5 | u8 | record `+0x43` |
| 6 | u8 | record `+0x42` |
| 7 | u32 | record `+0x44` |
| 8 | u16 mask | record `+0x36` |
| 9 | u8 wire flags | bit 0 → record `+0x48`; bit 1 → record `+0x49` |
| 10 | u16 | Only if wire bit 1 and mask `0x400`; record `+0x34` |
| 11 | u8 | Only if wire bit 1; record `+0x39` |
| 12 | u8 | Only if wire bit 1; record `+0x3a` |
| 13 | u16 | Only if wire bits 1 and 2; record `+0x3c` |
| 14 | u16 | Only if wire bits 1 and 3; record `+0x3e` |

The higher wire bits do not independently add fields in this reader.
Do not label the u16 mask a damage amount or a critical flag without establishing its consumer semantics.
The record's action/entity-related fields and guard conditions must be followed separately.

## `0x1af`: dispatch and feedback binding

Actor group dispatcher `0x1421940a0`, through `0x1421952a0`, reaches `0x141e90f50`.
Record `+0x48` selects immediate action-associated processing or an `0x88`-byte queued representation.
The immediate path invokes Actor virtual slot `+0x230`.
The audited Actor tables bind it to `0x141e91580`.
The queued path's full lifecycle remains unresolved.

The downstream consumer resolves the record's entity reference and applies relationship/stat guards.
Its source branches use Actor signals `+0x2e8` and `+0x368`; guarded target branches use `+0x3e8`.
These have a concrete subscriber binding:

| Registration | Receiver callback | Established scope |
|---|---|---|
| Render constructor `0x1415f5d50` → source Actor `+0x2e8` | `0x141600c10` | Render callback; record is argument 2 |
| Same constructor → source Actor `+0x368` | `0x141600aa0` | Prepares `Audio/Sound Effects/WeaponSwinging/MissSG` |
| Same constructor → target Actor `+0x3e8` | `0x141609f40` | Target Render callback; record is argument 3 |

The different record argument positions follow the registered thunk ABI.
Decompiler-inferred signatures can omit forwarded parameters; the instruction audit takes precedence.
The MissSG cue does not identify every server miss/block outcome or prove a sound played.
Other subscribers and the numerical consequences of all mask bits remain research tasks.
No complete accuracy/critical/armour/pre-deduction chain is established by this dispatch.

## `0x1b1`: absolute resource assignment

The [existing wire description](10-chat-and-commands.md) documents the resource message under Monsters.
The world handler `0x142187480` routes it to `0x141e69fb0`:

| Pool selector | Pool within Life | Call instruction |
|---|---|---|
| 0: Life | `+0x178` | `0x1421880e6` |
| 1: Mana | `+0x1c8` | `0x1421880cb` |
| 2: ES | `+0x210` | `0x1421880b7` |

The setter copies decoded current into pool `+0x30`, reserved into `+0x10`, and flags into `+0x14`.
The setter itself does not reset fractional carry `+0x28` or replay input history.
Death/revive handling in the surrounding dispatcher is distinct from the pool assignment.

Between such assignments, the native Life tick can integrate continuous recovery and loss using stat rates.
That supports continuous extrapolation plus incoming value correction.
It does not establish that the client independently predicts numerical hit damage.
See [Stats update](10-chat-and-commands.md#3-sc-0x1b3-stats-update) for explicit-zero overrides and type fallback.

## Remaining protocol questions

- Resolve the full queued `0x1af` consumer and remaining subscribers without assuming a numerical damage role.
- Establish concrete meanings of the action-related fields and mask bits, preserving version and object guards.
- Determine whether another message or local action path supplies a real pre-deduction input.
- Trace how rate changes, fractional state and incoming resource assignment are ordered in the client loop.
- Keep official-server claims separate from any future, separately authorized LocalExile runtime experiment.
