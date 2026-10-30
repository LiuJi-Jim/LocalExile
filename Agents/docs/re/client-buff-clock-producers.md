# Client Buff clock guard producers (3.29.3b)

A concrete external producer of the Buffs advancement guard is now bound:
the selected **Life component → retained Buffs component → virtual `+0xc8`
→ Buffs `+0x17a`** path. Life initialization with nonpositive current and
its nonpositive transition callback set the guard. A paired Life callback
clears it. This closes the receiver identity missing from the
[clock-update supplement](client-buff-clock-updates.md); it does not close
owner-stat-change recalculation of an existing instance's speed.

The input is the same original `Client/Game.exe`, SHA256
`78a0e227189c263658a7f65a02f086428b859bfc114ae8e7ce8aab721ebd48d5`,
with preferred image base `0x140000000`. All observations are offline original
bytes with chained PE-unwind ownership, Capstone 5.0.3 and pefile 2024.8.26.
No client execution, traffic acquisition or server changes were used.

## The retained receiver is positively identified as Buffs

Life configuration resolver `0x141e66ec0` looks up literal `Buffs`
(`0x14359cdc8`) in its component-name map at
`0x141e66f1e..0x141e66f40` and stores the returned component index in
configuration `+0xf0` at `0x141e66f4c`.

Life constructor `0x141e678d0` retains that configuration at `+0x160` and
its owning Entity at `+8`. It uses configuration `+0xf0` to index that
Entity's component vector `+0x10`, then stores the resulting pointer at
**Life `+0x170`** (`0x141e67971..0x141e67980`). This is the receiver used
by the guard producers below. It is not an inference from the shared
virtual offset `+0xc8` alone.

The constructor installs Life primary table `0x1435a7340` at
`0x141e67930..0x141e67937`. That table binds initialization slot
`+8 → 0x141e67b40`, transition slot `+0xe8 → 0x141e6c710`, and paired
slot `+0x100 → 0x141e6c760`. The previously bound Buffs factory installs
table `0x14359f5f8`, whose `+0xc8` is the exact seven-byte leaf
`0x141e0fbc0`: `mov byte ptr [rcx+0x17a],dl; ret`.

These bindings apply to the selected Life/Buffs component contracts. They do
not establish that every object sharing either virtual offset has these types.

## Concrete guard writes

| Producer | Bound path |
|---|---|
| Life initialization `0x141e67b40` | Tests Life current at `+0x1a8` (the embedded Life pool `+0x178`, current `+0x30`). The positive branch skips the write. In the nonpositive branch, after its other initialization work, `0x141e67c0b..0x141e67c17` loads retained Buffs `+0x170`, supplies `DL=1`, and calls its `+0xc8`. |
| Life transition `0x141e6c710` | Updates another retained component's state byte, invokes Life `+0x120`, then loads Life `+0x170` at `0x141e6c73c`, supplies `DL=1` at `0x141e6c743`, and tail-calls Buffs `+0xc8` at `0x141e6c74d`. |
| Paired Life callback `0x141e6c760` | Updates that other state byte in the opposite direction, supplies zero at `0x141e6c77b`, loads Life `+0x170`, and calls Buffs `+0xc8` at `0x141e6c787`. Its complete inspected body establishes the clear operation; its external invocation condition remains unbound in this pass. |

The transition has a positive numerical-resource caller as well as a table
binding. In the inspected loss reducer `0x141e6a8a0`, the branch through
`0x141e6ac55` calls the pool-loss helper, tests resulting current at
`0x141e6ac5a`, and skips the transition when current is positive. Otherwise,
after the change callback, it dispatches the pool's owning component
`+0xe8` at `0x141e6ac83`. For the embedded Life pool and installed Life
table above, this reaches the true-setting guard producer.

The independently established
[`ClientOnlyKillSelf` path](client-hit-resource-boundary.md#a-concrete-native-numeric-write-clientonlykillself)
also explicitly invokes Life `+0xe8` after setting current to zero at
`0x141819f11..0x141819f22`. This is a reuse of that existing resource-write
finding, not evidence that an ordinary contact program invokes it.

## What the guard changes

The consumer `0x141e10520` reads the Buff definition at instance `+8` and
tests its byte `+0xfc` at `0x141e105a8`. If that byte is zero, the newly
bound guard does not stop this path. If it is nonzero, a nonzero Buffs
`+0x17a` branches to cleanup at `0x141e105b8`, before both:

- remaining-duration subtraction/store at `0x141e105dd..0x141e105ea`;
- this invocation's backend `+0x28` callback at `0x141e10601`.

Thus the effect is a **conditional advancement/callback guard** for the
selected tick invocation. Clearing it removes this particular guard; the
other native mode, backend, instance-state and duration conditions still
apply. It is not a universal pause of every Buff callback.

None of these producers writes instance speed `+0x20` or invokes the
insertion-time owner-Stats clock calculator `0x141f79160`. A halted instance
clock does not by itself prove that a native incoming DoT rate stops, changes
or is removed. The definition-byte semantics and a damage-source consequence
require their own positive bindings.

## Evidence and remaining boundaries

Ignored original-byte audits under `Tools/re/out/client-damage-audit/`:

| Batch | Scope |
|---|---|
| `r4-buff-clock-producers-discovery` | Selected Life bodies, bounded direct/pointer discovery and virtual `+0xc8` candidates. |
| `r4-buff-clock-producers-life-guard` | Configuration-to-Buffs receiver binding, true/false guard writers, exact setter leaf and timer consumer; reused native self-kill caller. |
| `r4-buff-clock-producers-transition-search` | Life initialization/loss bodies and virtual `+0x100` candidates. |
| `r4-buff-clock-producers-reset-candidate` | Selected alternative receivers examined separately, plus virtual `+0xe8` candidates. |

`r4-buff-clock-producers-verify.py/json` independently rechecks 45 instruction
anchors, four table pointers, the exact setter leaf and literal, and all
23 byte ranges across these 15 function records against the original PE.

The displacement scans retain 81 `+0xc8`, 34 `+0x100`, and 67 `+0xe8`
owned candidates respectively; they do not resolve their receivers. The
latter two scans also retain six and two unowned candidates. These are
search inventories, not a claim that every candidate was classified.
Register-indirect calls, dynamically selected slots, unsupported prefixes,
unwind-less paths and inlined alternatives are outside that scanner's proof.
The selected `+0x100` alternatives did not supply a bound invoker of the
false-setting Life callback; no whole-program absence claim follows.

Still open: owner Stats notification → recalculation and speed write on an
existing Buff; the paired clear callback's external trigger; other possible
guard producers; and any connection from these clock changes to a native
damage contribution. The new positive result is the concrete Life-to-Buffs
guard chain above.
