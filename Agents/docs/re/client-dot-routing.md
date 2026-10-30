# Client DoT formulas, routing and arithmetic boundaries (3.29.3b)

## Scope and evidence

This is a static supplement to [client damage calculation](client-damage-calculation.md), sections 4–5.
It establishes conditional formula bodies and their registered keys, rather than execution on every network actor.
No client process, game UI, server, hook, attachment or runtime experiment was used.
These results do not establish official-server arithmetic or an independent local hit resolver.

The executable SHA256 is `78a0e227189c263658a7f65a02f086428b859bfc114ae8e7ce8aab721ebd48d5`.
The Stats table SHA256 is `91c2ed42f6d8094a2704f56aea32f6f16243881b04aadbe4fdbbdd3a566f05da`.
Addresses are preferred-image VAs with image base `0x140000000`.
Stat key `k` means zero-based Stats JSON row `k-1`, field `Id`.

The local evidence batches under ignored `Tools/re/out/client-damage-audit/` are:

| Batch | Contents |
|---|---|
| `r2-registration-discovery` | Immediate-key reference scan; 2,539 registration mappings recovered in unwind-owned functions below `0x140150000` |
| `r2-typed-routes` | 191 original-byte function records: typed mitigation, taken-as, typed routes, aggregates and registration functions |
| `r2-route-guards` | 52 records: resistance/max-resistance, bypass, darkness and supporting predicates |
| `r2-routing-support` | 14 records: Mana output, physical/elemental reduction, generic MoM, chaos ES percentage and healing predicate |
| `r2-base-rates` | Raw/self-rate formulas and action constructor candidate; 29 direct callers of the contribution adaptor |
| `r2-source-discovery` | Bounded immediate raw-key/direct-native-contribution reference scan, 127 verified instruction candidates |
| `r2-source-writers` | Contribution adaptor callers and its native object selection helper |
| `r2-source-builder-refs` / `r2-source-builder-bodies` | Eight direct references to the skill contribution builder and their four owning bodies |
| `r2-buff-network` | AddBuff reader, Buffs component handler, parameter copier, Buff constructor and tick; selected BuffDefinitions rows |
| `r2-buff-bindings-v2` | Buff insertion, allocation and replacement |
| `r2-buff-runtime-factory` / `r2-buff-factory-refs` | Id2 backend lookup/factory, registry singleton and eight direct references to it |
| `r2-buff-registry-callers` | Seven additional direct-call owners, including the two named registration routines |
| `r2-rf-backend` / `r2-rf-buff-ctor` / `r2-rf-buff-body` | RighteousFireBuff factory, constructor, action lookup, transform and object-creation bodies |
| `r2-buff-expiry-refs` / `r2-buff-expiry-owner` | Direct duration-tick callers and their three owning bodies |
| `r2-buff-component` / `r2-buff-vtable` | Component constructor, event dispatcher and original-byte vtable entries |
| `r2-buff-removal` / `r2-buff-destructor` | Message-driven removal, secondary backend tick, component/Buff destructors; exact reviewed five-byte tick thunk |

Each `audit.json` retains original-file offsets, SHA256s, PE-unwind roots/chains and every decoded instruction.
Selected `decompilations.jsonl` files come from Ghidra 12.1.4, JDK 21, `-noanalysis`, bounded bodies only.
The `.compact.c` files are analyst-generated reading aids, **not primary evidence**; original `.asm`, original-byte records
and unmodified decompilation JSONL decide ambiguous data flow.
Reproduce byte records with [damage_audit.py](../../../scripts/re/damage_audit.py) and selected decompilation with
[BoundedFunctions.java](../../../scripts/re/ghidra/BoundedFunctions.java).

Registration mapping requires an actual `call 0x142513630`, the output key loaded into EDX and callback address
loaded with RIP-relative `lea r8`. A raw function pointer or matching integer alone is insufficient.
For example registration `0x1400ad8c0` binds key 3357 to `0x1426db5c0`;
`0x1400ad940` binds 3358 to `0x1426dc1b0`; `0x1400ad9b0` binds 3359 to `0x1426dcdb0`.
Shared callback bindings are checked separately: 9609, 9622, 9629 and 14383 all bind `0x14281bd60`.

## Numeric notation

`S(k)` below is the integer stat value obtained by the callback's evaluation-cache/raw-parent lookup.
`B(k)` is specifically the nonzero predicate returned by `0x1425135a0`; it is not a numeric getter.
`f(x)` denotes signed-integer-to-binary32 conversion. Every shown floating operation is binary32 with its native order.
The readable equations are not a replacement for that order.

```text
I(x) = max(0, f(add32(x,100)) / 100)
P(x) = max(0, f(x) / 100)
Q(x) = max(0, f(sub32(100,x)) / 100)
R(x) = CVTTSS2SI(binary32(x + (x < 0 ? -0.5 : +0.5)))
C(x) = x >= binary32(2^31) ? INT_MAX : R(x)
C0(x) = max(0,C(x))
split(d,p) = signed_min(d,R(binary32(binary32(f(p)*f(d))/100)))
```

`add32`/`sub32` identify ordinary 32-bit instructions, with no widening or saturation.
`C` is an upper saturation followed by signed rounding; it has no independent lower saturation.
`C0` adds nonnegative clamping.
The threshold at `0x1436a48a8` is **2,147,483,648**, not exactly represented `INT_MAX`.
The signed half constants are `+0.5` at `0x1436a2f80` and `-0.5` at `0x1436a49a0`.
Invalid/out-of-range signed conversions and NaNs must retain x86 conversion semantics; real-number algebra cannot describe them faithfully.

## Typed mitigation

All five lethal typed outputs return zero when 19670 `cannot_be_damaged` or 15468
`cannot_take_damage_over_time` is nonzero. Each also has its own immunity key.
With those guards clear, their readable equations are:

```text
common(t) = I(virtual_t_damage_taken_over_time_+%)
            * I(t_damage_taken_over_time_+%_final) * I(S(15460))
level = I(S(8173))

physical = C0(common(physical) * f(S(9563)) * (1-f(max(0,S(448)))/100) * level
              + common(physical) * f(S(9568)) * (1-f(max(0,S(448)))/100))
fire      = C0(Kfire * f(S(9564)) * E * level + Kfire * f(S(9569)) * E)
cold      = C0(Kcold * f(S(9565)) * E * level + Kcold * f(S(9571)) * E)
lightning = C0(Klightning * f(S(9566)) * E * level + Klightning * f(S(9572)) * E)
chaos     = C0(Kchaos * f(S(9567)) * level + Kchaos * f(S(9573)))

E = 1-f(S(20993))/100
Kfire = (1-f(S(17170))/100) * common(fire)
Kcold = (1-f(S(18922))/100) * common(cold)
Klightning = (B(12061) ? 1 : 1-f(S(18923))/100) * common(lightning)
Kchaos = (1-f(S(5955))/100) * common(chaos)
```

| Output | Callback | Immunity | Increase / final keys | Upper compare anchor |
|---|---|---|---|---|
| 3355 physical | `0x1426d9550` | 719 | 10734 / 14340 | `0x1426d9f9a` |
| 3356 fire | `0x1426da000` | 4522 | 10737 / 14341 | `0x1426dab8e` |
| 3357 cold | `0x1426db5c0` | 4523 | 10735 / 14342 | `0x1426dc12e` |
| 3358 lightning | `0x1426dc1b0` | 4524 | 10736 / 14343 | `0x1426dcd2e` |
| 3359 chaos | `0x1426dcdb0` | 4525 | 10738 / 14344 | `0x1426dd7fa` |

Fire additionally returns zero when **both** 19157 and 19158 are nonzero.
The same fire guards apply to nonlethal fire 3360 at `0x1426dac10`.
Its input is 9570, multiplied by fire resistance, elemental reduction and the same three taken factors,
without the level factor. Nonlethal chaos 9593 at `0x1426dd860` uses 9574, chaos resistance and chaos
taken factors, with 19670/15468/4525 zero guards and no level or elemental-reduction factor.
Both use `C0` at the end.

Physical's **local** reduction guard is `max(0,S(448))`, at `0x1426d9918..0x1426d991c`; no local 90 clamp occurs.
Its upstream callback `0x14274b370` combines many ordinary integer products/sums and then selects the lesser
of that total and 12270 `virtual_maximum_physical_damage_reduction_%`.
Elemental reduction callback `0x14274d840` ends with ordinary integer products/additions and a lower zero clamp;
the typed elemental callbacks themselves do not impose an upper percentage clamp.

### Resistance asymmetries

| Output / callback | Verified branch/formula |
|---|---|
| Fire 17170 / `0x14271a3f0` | 18912 returns zero; 18064 returns zero while 20408 is false; otherwise `min(max(17171,-200), selectedMax)` |
| Cold 18922 / `0x14271d360` | `min(max(18924,-200), selectedMax)`; no fire-specific zero predicates |
| Lightning 18923 / `0x142720020` | 12099 first returns 12098 directly; otherwise `min(max(18925,-200), selectedMax)` |
| Chaos 5955 / `0x1427224d0` | `min(6217,152)`; **no -200 floor in this body** |

For elemental types, selectedMax is that type's maximum (146/148/150), or 15619 when flag 15620 is active.
Lightning's 12099/12098 shortcut precedes the ordinary maximum/minimum branch.
Lightning mitigation's separate 12061 is `lightning_resistance_does_not_apply_to_lighting_damage`.
Neither the cold nor chaos mitigation body has that shortcut.

The maximum-resistance callbacks contain their own boundaries:
fire 146 at `0x142716ca0` returns nonzero override 7700 directly, otherwise caps its ordinary sum at 90;
cold 148 at `0x142716fa0`, lightning 150 at `0x142717810`, and chaos 152 at `0x142718080` also contain
ordinary integer component sums and 90 comparisons. These upstream caps do not justify inserting a second,
unconditional 90 clamp into the final DoT-resistance callback or ignoring an earlier override.

## ES bypass, healing and Mana-before-Life

Each typed channel first decides whether damage heals. Fire 2784, cold 2785, lightning 2786 and chaos 17902
return zero in **both** ES-bypassing and ES-damaging callbacks when their healing predicate is active.
Physical and darkness have no corresponding healing gate in the audited split bodies.

For physical/fire/cold/lightning/darkness, with the healing gate clear:

```text
bypass = split(typedRate,bypassPercent)
eligibleForES = sub32(typedRate,bypass)
manaBypass = split(bypass,manaPercent)
lifeBypass = sub32(bypass,manaBypass)
manaES = split(eligibleForES,manaPercent)
lifeES = sub32(eligibleForES,manaES)
```

Chaos has a different first split: `eligibleForES = split(chaosRate,S(3014))`, then
`bypass = sub32(chaosRate,eligibleForES)`. It does not compute an elemental-style bypass percentage.
Nonlethal fire and nonlethal chaos repeat their respective type's six-stage routing with rates 3360/9593.
The following columns are keys, in the precise order of those equations.

| Type | Rate | Percentage | Bypass | ES | Mana bypass | Life bypass | Mana ES | Life ES | Mana % |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| Physical | 3355 | 3015 | 9603 | 9604 | 9607 | 9605 | 9608 | 9606 | 9648 |
| Fire | 3356 | 9609 | 9610 | 9611 | 9614 | 9612 | 9615 | 9613 | 9649 |
| Nonlethal fire | 3360 | 9609 | 9616 | 9617 | 9620 | 9618 | 9621 | 9619 | 9649 |
| Cold | 3357 | 9622 | 9623 | 9624 | 9627 | 9625 | 9628 | 9626 | 9650 |
| Lightning | 3358 | 9629 | 9630 | 9631 | 9634 | 9632 | 9635 | 9633 | 9651 |
| Chaos | 3359 | 3014 (ES) | 9636 | 9637 | 9640 | 9638 | 9641 | 9639 | 9652 |
| Nonlethal chaos | 9593 | 3014 (ES) | 9642 | 9643 | 9646 | 9644 | 9647 | 9645 | 9652 |
| Darkness | 8487 | 14383 | 14386 | 14387 | 14384 | 14385 | 10021 | 10020 | 1694 |

The full typed route callbacks occupy `0x1426dea90..0x1426e53f0` (end exclusive for darkness routes).
This table was checked against each registered body, including the distinct chaos ordering and nonlethal bodies.
Example original arithmetic anchors: physical percentage conversion `0x1426dec6c..0x1426decbc`, physical Life remainder
`sub ebx,edi` at `0x1426df311`, chaos ES split callback `0x1426e2c70`, chaos bypass remainder callback `0x1426e2ec0`.

### Percentage producers

The shared elemental/darkness bypass callback `0x14281bd60` returns 100 for 9587 or 9586;
otherwise it clamps 9677 to 0..100. Physical bypass `0x14281b960` has the same forced-100 branches but also
adds 50 conditionally from 3013/2980 and adds 18017 before clamping 0..100.
Non-chaos bypass 9677 at `0x14281b8e0` is an ordinary sum of 16304 plus 50 when 9585 is nonzero.

Chaos ES-damaging percentage 3014 at `0x14281bec0` returns zero for 12570,
100 for 5298, otherwise starts at 15383, conditionally adds 50 using 3013/2980, then clamps 0..100.
These branches cannot be substituted by one universal “chaos bypasses ES” rule.

| Mana percentage | Callback | Formula before final clamp 0..100 |
|---|---|---|
| 9648 physical | `0x1427ec000` | `add32(1694,9331)` |
| 9649 fire / 9650 cold | `0x1427ec220` | 1694 |
| 9651 lightning | `0x1427ec340` | `add32(1694,4663)` |
| 9652 chaos | `0x1427ec560` | `add32(1694,10739)` |

1694 itself at `0x1427eb8b0` is an ordinary sum of conditional 7280/7335 and 8611/8663 contributions,
1971, `7*3297`, and 40 when 8383 is nonzero. Darkness uses this unclamped generic input directly in its split.

## Resource totals and overflow shares

Define lethal `LB=9595`, `LE=9594`, `MB=9597`, `ME=9596`; the same nonlethal channels are
`NLB=9599`, `NLE=9598`, `NMB=9601`, `NME=9600`.
Let `protectsMana = B(5338)`.

```text
3361 ES   = C(f(ME) + (protectsMana ? 0 : f(LE)))
3362 Life = C(f(LB) + (protectsMana ? f(LE) : 0))
6270 Mana = MB
3363 nonlethal ES   = protectsMana ? NME : add32(NME,NLE)
3364 nonlethal Life = protectsMana ? add32(NLB,NLE) : NLB
9602 nonlethal Mana = NMB
9960 overflowManaPermyriad = ME==0 ? 0 : clamp(R(f(ME)/f(add32(ME,LE))*10000),0,10000)
9961 overflowManaPermyriad = NME==0 ? 0 : clamp(R(f(NME)/f(add32(NME,NLE))*10000),0,10000)
```

| Output | Callback | Arithmetic boundary |
|---|---|---|
| 9595 / 9594 / 9597 / 9596 | `0x1426e53f0` / `0x1426e59e0` / `0x1426e5fd0` / `0x1426e65d0` | Six lethal typed channels (including darkness), individually converted to float, summed, then `C` |
| 9599 / 9598 / 9601 / 9600 | `0x1426e6bd0` / `0x1426e6de0` / `0x1426e6ff0` / `0x1426e7200` | Fire plus chaos through ordinary 32-bit addition; no saturation |
| 3361 / 3362 | `0x1426e7410` / `0x1426e7d60` | Float addition, `C`; 5338 selects Life-directed ES input |
| 3363 / 3364 | `0x1426e78e0` / `0x1426e7fc0` | Selected ordinary integer addition; no saturation |
| 6270 / 9602 | `0x1426e81e0` / `0x1426e82e0` | Forward 9597 / 9601 |
| 9960 / 9961 | `0x1426e7670` / `0x1426e7b00` | Ordinary 32-bit denominator addition before float conversion |
| 3365 healing | `0x1426e83d0` | Ordinary sum of heal-enabled fire3356/nonlethal3360/cold3357/lightning3358/chaos3359/nonlethal9593 |

Exact ordinary-add anchors include `0x1426e6db7` (9599), `0x1426e7ada` (3363),
`0x1426e787a` (9960 denominator), and `0x1426e8ea6..0x1426e8ec0` (healing six-channel sum).
The pool reducer spill behavior remains documented separately in section 5 of the main page.
These rate formulas do not create fresh hits on a spill recipient.

### Why a universal final cap is incorrect

The taken-as outputs 9563..9574 use intermediate `C` boundaries; typed mitigated 3355..3360/9593 use `C0`;
the four lethal route aggregates use `C`; final lethal Life/ES use `C` again.
The nonlethal route aggregates, final nonlethal Life/ES, healing sum and overflow denominators use ordinary additions.
Subtraction remainders are also ordinary 32-bit operations.

The split helper pattern has **no** `2^31` guard. In physical bypass, `CVTTSS2SI` at `0x1426decb3`
is followed by signed compare/min at `0x1426decb7..0x1426decbc`.
At `d=INT_MAX,p=100`, converting `d` to binary32 yields `2^31`; multiplication/division can still yield `2^31`;
with masked invalid exceptions the signed conversion yields integer indefinite `0x80000000`, which signed-min selects.
This is an instruction-level boundary case, not an observed game state or proven official-server exploit.
Similarly, overflowing the integer overflow-share denominator may change its sign before division.
Input reachability, exception state and live actor evaluation eligibility remain separate obligations.

## Base-rate computations and producer boundary

There are genuine local native computations upstream of taken-as; the raw fire forwarding callback alone was
too narrow a view. Their inputs remain stats, rather than a recovered list of timed damage instances.

| Output | Callback | Verified upstream arithmetic |
|---|---|---|
| 1864 physical | `0x1426cc060` | `add32(mul32(3902,3903),356)`; no float saturation |
| 2022 fire | `0x1426cce60` | Forward 2023 |
| 2024 cold | `0x1426ce8b0` | Forward 2025 |
| 2026 lightning | `0x1426cef90` | Forward 2027 |
| 352 chaos | `0x1426cf780` | `add32(mul32(516,2006),456)`; no float saturation |
| 8179 physical unaffected | `0x1426cc330` | Self-rate terms from Life247, ES249, 2614/3113, 5338, 7951/7955/13579 and ordinary stack product7471*7455; float `C`, then ordinary add of raw13573 |
| 8180 fire unaffected | `0x1426ccf50` | Native Life+ES percentage/self-burn terms, conditional ignite8215, additional state contributions, final `C`; raw13574 is booleanized, see below |
| 8183 chaos unaffected | `0x1426cfa50` | Native self-rate terms, conditionally transferred ignite8215, poison8216 and self-chaos8217; float `C` |

Physical unaffected is particularly important: `0x1426cce4a` adds raw13573 **after** its self-term saturation.
Its result therefore does not inherit a universal upper bound from the earlier float comparison.
The fire unaffected body calls `B(13574)` at `0x1426cda13`, zero-extends AL at `0x1426cda18`,
and adds that 0/1 at `0x1426cda25`; replacing this with a numeric 13574 add would contradict the instructions.
Ignite8215 is suppressed in fire when 13492 `take_chaos_damage_from_ignite_instead` is active,
and appears conditionally in chaos under the complementary branch.
These are stat-to-rate computations, not a recovered ignite/poison instance/snapshot algorithm.

The generic native contribution adaptor `0x14193c850` is now established beyond a pointer guess:
nonzero contribution R8D is retained in ESI; stat key EDX is retained in R14D;
it lazily resolves a native destination at `adaptor+0x10`, then calls `0x1424fc480`
with the same key, value and source token `adaptor+0x24` at `0x14193c8c0`/`0x14193c8f3`.
When its selector at `+8` is zero, it iterates action-owned native objects at owner `+0x1b8..+0x1c0`;
otherwise it uses its selected native object. The destination is not the network override vector.

Builder `0x14193b540` records `(key,value)` pairs at action `+0x1a8` and invokes that adaptor at action `+0x178`,
when global byte `0x1445b38cd` is zero and the contribution is nonzero.
Replay `0x14193bbe0` re-applies those pairs. This proves a real skill/native-contribution mechanism.
Its audited direct callers include fixed offence/state keys and dynamic pairs; none currently closes a specific
ignite/poison instance → raw8215/8216 or source effect → raw2023 edge.

Factory `0x1412c40e0` still does **not** close that edge. Immediate 2023 is passed to `0x1419e7100`,
through `0x1419e2a50`, then stored as a 16-bit action field at `0x141937d84` (`action+0x118`).
It does not flow into EDX of the contribution adaptor in that constructor path.
Calling it an RF raw-fire producer on the basis of that equal integer is unsupported.

### Network Buff boundary

BuffDefinitions SHA256 is `3a23c6fdf7ba29fb8508e5c9560a44e5292345e45dd2f886ef7126b8427fd659`.
The table proves the following declared associations, with GrantedStats rows converted to native keys:

| Zero-based row / Id | Granted native keys | IsRecovery | Id2 backend name |
|---|---|---|---|
| 0 / ignited | 8215 | false | empty |
| 10 / poison | 8216 | false | empty |
| 12 / righteous_fire | 882, 21656, 26, 881, 1948, 19861 | false | empty |
| 13 / righteous_fire_aura | 2023 | false | RighteousFireBuff |

AddBuff reader `0x141fdf870` sizes and zero-initializes its granted-value vector at packet `+0x48`
using the table's GrantedStats count (`row+0xdc`, `0x141fdfca4..0x141fdfcaf`).
It reads numeric values into that vector only when `row+0x7c` IsRecovery is true;
the false branch at `0x141fdfcbc` skips to `0x141fdfcf5`.
The four selected rows therefore do not receive their damage-rate magnitudes through that recovery-only payload.
The reader separately reads an unconditional counted u32 vector at packet `+0x88`.
It is a distinct field, not another name for the granted-value vector.

Copier `0x141e07220` carries packet `+0x48` to parameters `+0x10`, and packet `+0x88` to parameters `+0x80`.
Constructor `0x141e06290` carries those to Buff `+0x130` and Buff `+0x80` respectively;
it also keeps the table row at `+8` and duration/rate state at `+0x18/+0x1c/+0x20`.
The constructor checks the granted-value count against the table count and pads zeros on mismatch.
These copied fields establish an instance representation, without establishing publication into native actor stats.
Handler `0x141e11720` finds instances by their `+0x9c` key, updates/replaces an existing instance or allocates
and inserts a new one through `0x141e27e30` / `0x141e11dc0`.

Insertion calls registry singleton `0x141e06110` and backend factory `0x141e05fa0` at
`0x141e11e55..0x141e11e65`, then stores the result at Buff `+0x148`.
The factory reads the UTF-16 Id2 string from the table row at `+0x74`, computes its FNV64 hash,
looks up a registry entry and invokes its factory vtable slot zero at `0x141e06081`.
No matching entry returns a null backend. An empty table Id2 alone does not prove which registry entries exist.

There is a positive RF-aura backend binding: registration body `0x1415be830` sets up vtable `0x1434652b0`,
inserts the literal `RighteousFireBuff` at `0x1415c0085..0x1415c00a1`, and stores its factory object
in entry `+0x30` at `0x1415c00b0`. The vtable's first pointer is `0x1415d1010`, which allocates an object
and invokes constructor `0x14179d280`. That constructor keeps the Buff pointer, checks Buff `+0x30`,
and consumes the first value of Buff `+0x80` as an identifier for lookup `0x14179da90`.
The lookup builds a `0xb2` query with callback `0x141937ad0` and calls `0x141f70f30`.
Its precise resolved-object ownership remains unclassified here.

The audited constructor then calls `0x14179d7c0` and `0x14179d4e0`.
The first computes a scale from the resolved object's `+0x24` and Buff `+0x30`, builds a 64-byte transform
through `0x140cea160`, and passes it to `0x141dc4950` when an object exists.
The second creates an object through `0x141da6360` and sends that object plus the transform to `0x141dc31c0`.
This is evidence of a transform/object presentation path; these bodies do not publish raw2023 through
the audited native contribution adaptor or native-add entry.
It is not evidence that the backend locally derives RF damage magnitude.

The remaining producer boundary is consequently explicit: the inspected reader/copy/instance/backend path
does not close `ignite instance → raw8215`, `poison instance → raw8216`, or `RF aura instance → raw2023`.
Other consumers of the counted vector, Buff events, indirect backend methods and dynamically registered handlers
remain possible and were not exhausted. This bounded result must not become a claim that Buffs never affect stats.

### Duration ticking and message-driven removal

Constructor `0x141e0fc40` installs Buffs component vtable `0x14359f5f8` at `0x141e0fca6..0x141e0fcb3`,
stores the owning actor at component `+8`, and initializes the Buff vector at `+0x160..+0x170`.
The installed vtable's `+0x40` points to `0x141e10620`; its `+0x48` points to the exact five-byte
jump thunk `0x141e10690 → 0x141e106a0`. This constructor binding supplies ownership beyond a raw pointer match.

`0x141e10620` iterates component `+0x160..+0x168` and invokes duration tick `0x141e10520` for each instance
at `0x141e1065b`. A second direct caller, `0x142148880`, obtains one Buff through `0x141e092a0`
and invokes the same tick at `0x142148ee5`; this does not change what the tick body does.
Tick skips instances with Buff `+0xd0` bit `0x10` set. Its native/network-mode-dependent early skip is:

```text
global[0x1445b38cd] != 0
and byte[component.+158 + 0x68] == 0
and Buff.+148 backend == null
and byte[Buff.row + 0x100] == 0
and float[resolved row + 0xda] == 0
and float[resolved row + 0xde] == 0
```

The global comparison is at `0x141e10554`; the final equality branches to the return/cleanup path
at `0x141e105a2`. This is a **duration-tick guard**, not a native stat-publication guard.
When ticking is allowed, additional row/component flag checks, positive remaining duration and Buff `+0x118 != 4`
are required. The body subtracts `deltaTime * (row.+73 ? 1 : Buff.+20)` from Buff `+0x1c`
at `0x141e105e2..0x141e105ea`, then invokes backend vtable `+0x28` when one exists.
It does not clamp the new duration to zero or call the removal routine after crossing zero.
The secondary tick `0x141e106a0` only invokes backend vtable `+0x30` for unremoved instances
whose remaining duration is positive. Neither audited component tick removes the vector entry.

There is a separate positive removal entry: message handler `0x141e11720` selects opcode `0x1c2`
at `0x141e11749..0x141e11754`, loads the instance key from packet `+0x20`, and calls
`0x141e11fb0` at `0x141e11761`. Opcode `0x1c1` takes its add/update branch instead.
The removal routine finds the active instance by Buff `+0x9c`, erases and compacts its vector entry,
decrements component `+0x168` by eight, and sets Buff `+0xd0` bit `0x10` at `0x141e12090`.
It then invokes backend removal vtable `+0x18` at `0x141e120a6`, dispatches the instance's `+0x150`
event through `0x14019a240`, and performs additional object/token cleanup.
At `0x141e121ab..0x141e121b2` it dispatches the component event through `0x141690920`
with thunk `0x1401d81f4` (listener vtable `+0x10`), followed by `0x141de6000` notifications.
Finally it invokes Buff destructor `0x141e06650` and frees the instance.

The component event dispatcher `0x141690920` visits two listener collections, calls the supplied thunk,
and compacts removed listeners. Dispatch alone does not classify those listeners as network stat wrappers
or native formula producers. The destructor clears the granted-value and counted vectors, invokes backend
destruction and releases retained objects. Its indirect cleanup methods and event subscribers are not
an exhaustive source-contribution analysis.

Thus the audited network lifecycle distinguishes local duration state and backend callbacks from explicit
message-driven removal. It does not establish that local timer expiry removes a damage source or subtracts
an ignite/poison/RF rate from native actor stats. The generic action-contribution route has its own
`0x14193be90` mode guard at `0x14193bebc..0x14193bec3`, and builder `0x14193b540` is separately guarded;
successful-startup binding is documented in [stat evaluation](client-stat-evaluation.md#5-action-local-native-mutation-is-guarded-out-by-ordinary-startup).
Those guards apply to their identified action routes. A Buff-specific route into them has not been recovered,
so they cannot be generalized into a proof that every Buff damage source is disabled.

## Coverage and remaining work

Typed mitigation, ES/Mana/Life splits, healing exclusions, lethal/nonlethal aggregates, overflow ratios and their
local arithmetic boundaries are established as conditional static formulas for this exact executable.
The raw/self-rate callbacks show more local computation than forwarding alone, but the following are still open:

- Concrete ailment/effect/source instance creation, duration, stacking and snapshot contribution into raw base rates.
- Whether those source contributions target each actual network actor's fallback object, and when eligible entries evaluate.
- Reachable ranges of extreme inputs and whether the native conversion/ordinary-overflow boundary cases occur in play.
- Exhaustive transitive taken-as percentage producers, every upstream maximum-resistance override, and every self-rate state term.
- Official-server arithmetic, correction cadence and source authority.

The bounded source search covered immediate `mov edx,key` references for 356/456/2023/2025/2027/8215/8216/8217/13573/13574,
direct E8/E9 references to native add `0x1424fc480`, the 29 verified direct calls to adaptor `0x14193c850`,
and eight verified direct calls to builder `0x14193b540`.
It excludes indirect calls, register/data-driven keys, unwind-less leaves and arbitrary data-table producers.
A lack of a producer in that search is not proof that no such producer exists.
The Buff follow-up adds the reader-to-instance path, eight direct registry-singleton references and the named
RF-aura factory chain described above. It still does not establish source-instance damage snapshots or publication.
The lifecycle increment additionally closes component ownership of both ticks and the explicit remove-message
path, while retaining timer-to-rate removal and subscriber-to-native-publication as unresolved edges.
