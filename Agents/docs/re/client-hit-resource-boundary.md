# Hit-to-resource boundary: receiver audit (3.29.3b)

The concrete numerical resource-loss receivers are identified, but an independent
**Hit → these receivers → deduction** path remains unresolved. In particular,
an indirect call with displacement `+0x48` is not enough to identify resource loss.
The Life **component** and its embedded **resource pools** use different tables
and assign different meanings to that same displacement.

This is an independent static supplement to
[client damage calculation](client-damage-calculation.md#why-a-direct-call-negative-result-is-insufficient).
It does not establish official-server execution, live client prediction, or
global absence of hit calculation. No client or server was executed, attached,
hooked or controlled for this pass.

## 1. Identity and method

| Input | Identity |
|---|---|
| `Client/Game.exe` | SHA256 `78a0e227189c263658a7f65a02f086428b859bfc114ae8e7ce8aab721ebd48d5` |
| Preferred image base | `0x140000000`; addresses are not live ASLR addresses |
| `data/tables/Stats.json` | SHA256 `91c2ed42f6d8094a2704f56aea32f6f16243881b04aadbe4fdbbdd3a566f05da` |
| Decoding | Capstone 5.0.3, pefile 2024.8.26; PE unwind roots plus chained ranges |
| Leaf exception | Explicit reviewed range `[0x1400f0f00,0x1400f0f03)`, bytes `c20000` (`ret 0`) |

The pass reused [damage_audit.py](../../../scripts/re/damage_audit.py).
Generated evidence stays under ignored `Tools/re/out/client-damage-audit/`:

- `r3-loss-receiver-boundary/audit.json`: 13 decoded bodies, range hashes,
  25 direct reducer references, four constructor/table LEAs and three raw reducer pointers.
- `r3-loss-vtables.json`: four exact table spans and leaf previews.
- `r3-loss-candidates.json`: preceding 55 linear instructions for the existing
  667 displacement-`0x48` candidates; this is triage, not reaching-definitions analysis.
- `r3-loss-materialized48-records.json`: 53 additional register-target candidates,
  load/dispatch bytes, context and source ranges/hashes for 43 owners.
- `r3-loss-{peek,vtables,materialized48,records}.py`: local analysis helpers.

These are original-byte audit records, not REA Evidence IDs. No new native REA
session was opened. The action/event follow-up below used a separate offline
Ghidra project, with selected decompilation completing for all four requested
bodies; its temporary project was deleted after export. Decompiled signatures
are provisional; the ABI claims below follow decoded instructions. The audit
checked instruction boundaries; it did not perform whole-program CFG/value-flow
analysis.

## 2. Proven table and object binding

Constructor `0x141e678d0` stores the owning Entity at component `+8`, then installs
the Life primary table at `0x141e67930..0x141e67937`. Its three calls to the pool
base constructor `0x141e65df0` pass the Life component as `RDX` and the embedded
pool address as `RCX`; each is immediately followed by its concrete table store.

| Receiver | Address relative to Life | Installed table | Installation | Relevant slot |
|---|---|---|---|---|
| Life component | `+0` | `0x1435a7340` | `0x141e67937` | `+0x40 → 0x141e6a190`; `+0x48 → 0x1400f0f00` |
| Life pool | `+0x178` | `0x1435a5da0` | `0x141e679c1` | `+0x48 → 0x141e6a8a0` |
| Mana pool | `+0x1c8` | `0x1435a5b68` | `0x141e67a0b` | `+0x48 → 0x141e6ae00` |
| Energy Shield pool | `+0x210` | `0x1435a6218` | `0x141e67a4c` | `+0x48 → 0x141e6a780` |

The loss functions consume `RCX=pool`, `XMM1=delta`, `R8b=reason`,
`R9b=nonlethal`. Their common numerical reducer is `0x141e6a6b0`.
Its integer-current/fractional-carry behavior and the continuous step functions
are described in the main report. The table installation and reducer bodies
establish this ABI; matching an arbitrary float argument does not establish it.

A raw eight-byte-pointer search finds each concrete reducer exactly once, at
its table's `+0x48`: `0x1435a5de8`, `0x1435a5bb0`, `0x1435a6260`.
The boundary-checked RIP-LEA search for the four tables finds the four constructor
sites above. This supports this binding; it does not rule out dynamically copied
tables, computed addresses, different instruction forms or another object implementation.

The Life primary table also assigns `+0xf8` and `+0x168` to `0x1400f0f00`.
Thus the Life reducer's virtual callbacks at those slots are empty on the object
constructed with this table. The `reserve_life_instead_of_loss_from_damage_for_x_ms`
branch alone does not prove that a meaningful deferred-damage callback executes
on that object. A different table installation would need separate evidence.

## 3. Concrete indirect receiver examples

Three attractive candidates pass a float in `XMM1` and call `+0x48`, but the
receiver is loaded directly from an Entity's component vector at `+0x10..+0x18`:

| Callsite / owner | Actual receiver and input |
|---|---|
| `0x14167a111` / `0x14167a040` | `RCX = Entity.components[index]`; `XMM1` is a selected float from the enclosing iteration, or zero |
| `0x142184b81` / `0x142184ae0` | `RCX = Entity.components[index]`; `XMM1` is selected through the two input pointers after world/entity flag tests |
| `0x141769857` / `0x141769210` | `RCX = Entity.components[index]`; requires `[RCX+8] == Entity`, then explicitly zeros `XMM1` |

When the element is the Life component constructed above, these dispatches land
in `ret 0`, not any pool-loss reducer. The vectors may contain other components;
this finding does not resolve all their dynamic element types or claim that
every component's slot is empty. It does prove why component iteration must not
be mistaken for a call on `Life+0x178`, `Life+0x1c8` or `Life+0x210`.
The ordinary Life integration function instead occupies component slot `+0x40`.

Other selected float-nearby candidates also fail to establish a pool receiver:

| Callsite / owner | Observed boundary |
|---|---|
| `0x14163dc18` / `0x14163dbc0` | Incoming object at `RCX`; decreases its `+0x18` float using a rate at `+0x1c`, invokes `+0x48` on expiry, then sets a flag and invokes `+0x18`. No embedded-pool provenance is established. |
| `0x141eb74e1`, `0x141eb754f` / `0x141eb7470` | Incoming object's `+0x18` float and `+0x1c` state drive `+0x48`; transitions to state 4 and an object-removal-style flag/slot-`+0x18` sequence. The first float load is a threshold comparison, not proved resource delta. |
| `0x141c0cbba` / `0x141c0c6b0` | Receiver is a vector element with a tag check `[receiver+8]==0x10`; preceding `XMM1` computes a sampling/index ratio, then is converted to an integer. Return `RAX` is tested and dereferenced at `+0xc`; no pool binding is established. |
| `0x141fe6306` / `0x141fe5fd0` | Receiver is the incoming object, whose owner Entity is at `+0x10`; preceding work traverses spatial coordinates/terrain thresholds. No embedded-pool receiver is established. |

The expiry/state observations are structural descriptions, not recovered class names.

## 4. Direct numerical ingress is bounded and known

The fresh E8/E9 instruction-boundary scan of the common reducer and three pool
reducers finds **25 callsites in seven owners**, with no unowned candidate for
those exact direct target encodings:

| Owner | Direct destinations and callsites |
|---|---|
| Spending `0x141e6a370` | Common at `0x141e6a4b4`, `0x141e6a4f0`, `0x141e6a517`, `0x141e6a533`; Life at `0x141e6a4ce` |
| ES reducer `0x141e6a780` | Common at `0x141e6a7b6`; Mana at `0x141e6a809`, `0x141e6a84c`; Life at `0x141e6a86f` |
| Life reducer `0x141e6a8a0` | Common at `0x141e6abb0`, `0x141e6ac55` |
| Mana reducer `0x141e6ae00` | Common at `0x141e6ae2c`; Life at `0x141e6ae51` |
| Life step `0x141e6ae70` | Life at `0x141e6afa1`, `0x141e6afda`, `0x141e6b010`, `0x141e6b04a` |
| ES step `0x141e6b480` | ES at `0x141e6b54a`, `0x141e6b566`; Common at `0x141e6b588` |
| Mana step `0x141e6cba0` | Common at `0x141e6cc9e`, `0x141e6cce8`, `0x141e6cd4b`; Life at `0x141e6cd06`, `0x141e6cd6a` |

These are continuous integration, spending and overflow routes. The audited
instructions establish numerical deduction, but no independent Hit input or
hit-result value has been connected to one of these calls.

## 5. Coverage and unresolved questions

The original FF /2,/4 memory-indirect scan contains **667 sites in 448 owners**,
plus **21 unowned candidate byte positions**. All 667 received a preceding-55-
instruction context pass; this did not resolve 667 receivers. In that window,
144 sites have a syntactic `XMM1` write and 165 have syntactic writes to both
reason/nonlethal registers. Writes can be stale across calls or branches.
Only 17 windows contain `+0x178`, `+0x1c8` or `+0x210`; these are not proof that
the base register is Life. The three component examples above were reviewed by
actual operand flow rather than these counts.

An additional scan covers a previously omitted form: opcode `8b` loads from
memory displacement `+0x48`, followed within 16 linear decoded instructions by
`call reg` or `jmp reg` using the loaded register. It stops at an intervening
call, branch, return or detected destination write. It found **53 sites in 43
owners**, none in the Life-local range `[0x141e60000,0x141e70000)`.
The **560 unowned byte candidates** in that pass are load candidates, not calls.
Inspected contexts include packed integer color arguments, stack/object pointer
arguments, global callback tables and function-pointer fields directly inside
objects. No new embedded-pool receiver was established. A target load from an
object's own `+0x48` field is not necessarily a virtual-table access at all.

Both scans remain bounded: no cross-block reaching definitions, callee summaries,
complete pointer-escape analysis, computed table addresses, arbitrary target copies,
complete prefix coverage or unwind-less-leaf discovery. The register-target
helper does not fully normalize GPR subregister aliases; retained candidates
therefore require manual inspection. No absence claim follows from the linear
`XMM1` ranking or the zero Life-local candidate count.

| Research question | Status |
|---|---|
| Which concrete objects own loss slot `+0x48`? | Answered for the three constructor-installed embedded pool tables. |
| Do the three component-vector calls enter those pool reducers for Life? | No; on the identified Life component table they invoke an empty function. |
| Are direct numerical loss callers accounted for? | Answered for the stated E8/E9 target scan: 25 sites, seven owners. |
| Is any indirect caller proved to supply Hit loss to an embedded pool? | Unresolved. |
| Does every possible indirect path exclude a Hit calculation? | Unresolved; the scans are not a whole-program proof. |

Closing R3 still requires a concrete input/result producer, defender identity,
receiver provenance to an embedded pool (or another numerical setter), and the
value flow for amount/reason/nonlethal. RNG, mitigation and routing must be linked
to that same path before calling it independent client hit prediction.

## 6. Reproducible source anchors

The following exact original-file spans support the new receiver distinctions.
End addresses are exclusive. Chained function portions are named as portions;
they are not silently widened into complete functions.

| Purpose / VA span | File offset | SHA256 |
|---|---:|---|
| Life constructor `[0x141e678d0,0x141e67ac4)` | 31878352 | `2edc7dd34048b12b220f92aefd34a6b9854aeebe6764aa151142a5f19e32e0fb` |
| Empty leaf `[0x1400f0f00,0x1400f0f03)` | 983808 | `b6cbececf1b07e74f37d57710c68c31f0b92cb08d51093252b321b07542e177b` |
| Component loop portion `[0x14167a062,0x14167a14a)` | 23565410 | `3ca461fb7596b3a1d95f770d5fee05ea9528bdbb56f804f41ac28e9e1d59750c` |
| Component loop `[0x142184ae0,0x142184bad)` | 35143392 | `4d2083116fe16776ccdfb6576d740220b19d789153befa9a3ade7c9f1c6f9c51` |
| Zero-float component dispatch portion `[0x14176969e,0x1417698eb)` | 24545950 | `45181755975b8b176bb99dda67c684289365e03006ab6796ea681260357d6cff` |
| Expiry candidate `[0x14163dbc0,0x14163df4b)` | 23318464 | `27c92c6dd6f4bb9131eef4a206a0744145a615350e2a82460095b7832b69f485` |
| State candidate `[0x141eb7470,0x141eb756e)` | 32204912 | `e0236416a67c4dd08ff91a14923af2391253448f8b7b518ddd49b87e08243f98` |
| Sampling candidate `[0x141c0c6b0,0x141c0cf0c)` | 29407920 | `ec3c94c8393b1e09a2cc12b64546a2eb1bacdc05392a07841d906d7887d4d105` |
| Life primary table `[0x1435a7340,0x1435a74b8)` | 56256320 | `64d77b7db7b79b12fe30e106e2b04a4c5d95bc3f4c9add54e07cd8152a4b1aef` |
| Life pool table span `[0x1435a5da0,0x1435a5e28)` | 56250784 | `7a93ee6a7effd8130bb2ef5c44ea1d2abee1ddea2cd283bfbc40dfc307ae6955` |
| Mana pool table span `[0x1435a5b68,0x1435a5bf0)` | 56250216 | `0cea5d7c1cbab2f5f0f2e60749d8328776de8ab512daf2bc0aca91788cf4ec78` |
| ES pool table span `[0x1435a6218,0x1435a62a0)` | 56251928 | `51334c88ad76507fe58236218f347aad9391d32ef007390ba1b4fe0ab4927ca4` |

The table spans are selected byte ranges, not claims that the following address
starts another complete object. Full chained ranges for every audited body and
the 43 additional candidate owners are retained in the local records named above.

## 7. Concrete action callbacks: Fireball and Melee

The result consumer `0x141e91580` calls action slots `+0x2d0` at
`0x141e92053` and `+0x2d8` at `0x141e920ae`, with `RDX=result` and
`R8=resolved target Entity`. Both require a matching action and nonzero
`result+0x49`. A base Action table with empty slots cannot settle the concrete
dispatch. Named registrations and the final factory stores give these bindings:

| Action / registry flag | Named registration → factory | Final table / installation | `+0x2d0` | `+0x2d8` |
|---|---|---|---|---|
| Fireball / 0 and 1 | `0x141973cd0 → 0x141988dd0` | `0x143335d80` in constructor `0x141a8b150`, store at `0x141a8b17e` | `ret 0` | `ret 0` |
| Melee / 0 | `0x14196a070 → 0x1419bbb70` | `0x143517ba0`, final factory store at `0x1419bbc67` | `ret 0` | `0x14136f210` |
| Melee / 1 | `0x14196a040 → 0x1419bbca0` | `0x143273730` in constructor `0x141ac9dc0`, store at `0x141ac9de3` | `ret 0` | `ret 0` |

Here `ret 0` is the exact leaf `0x1400f0f00`. The selected factory branches
are the ordinary non-mine/trap/totem cases; wrapper branches through
`0x1419e2e50`, `0x1419e2ef0` and `0x1419e2fa0` remain unexamined. Melee flag 0
temporarily installs `0x14355e290` in constructor `0x141c98150`; its later
factory store is essential. No RTTI class name is inferred from these tables.

The data references are zero-based: GrantedEffects row 0 `Fireball` points to
ActiveSkills row 84 `fireball`, which selects ActionTypes row 36 `Fireball`,
`HASH16=43598 (0xaa4e)`. GrantedEffects row 468 `PlayerMelee` points to
ActiveSkills row 108 `melee`, which selects ActionTypes row 47 `Melee`,
`HASH16=45448 (0xb188)`. The constructors pass those exact action identifiers.
GrantedEffects row 406 `Melee` also selects ActiveSkills row 108.

Lookup `0x141ee97f0 → 0x141941c90` uses the skill identifier at `+0x40`
and action hash at `+0x42`, searches the registry through `0x1419774d0`, and
invokes the selected entry's `+8` factory. Registrar `0x141941bb0` hashes the
supplied ASCII name with multiply-then-XOR FNV starting at zero and folds its
high/low 16-bit halves. Mode 0 inserts both flag values, mode 1 only flag 0,
and mode 2 only flag 1. The lookup derives the flag from actor virtual
`+0x148` data fields `+0x14c/+0x150`; its semantic name is not established here.

**Correction to the older generic spell-table example:** table `0x1434adb70`
is concretely installed by factory `0x141991bc0`, whose registration
`0x141972290` supplies the name `CastAura`. It must not be used as the concrete
Fireball binding; the named Fireball registration above selects another table.

### Melee's nonempty callback

`0x14136f210` preserves the target Entity, calls action virtual `+0x4b8`, then
tail-jumps to `0x1419e4620` with `RCX=action`, `RDX=target`. For final table
`0x143517ba0`, slot `+0x4b8` is `0x141b55cb0`: it returns `1.0` for a skill
data flag at `+0xa5`; otherwise it obtains stat context through `0x14193b350`,
selects main/offhand attack-speed key 294/295 using action `+0x311`, and returns
`1 / (1 + speed/100)`. The raw callback does not explicitly copy that returned
float into another XMM argument; a guessed decompiler signature is insufficient
to label it a damage amount.

The following helper reads `enable_test_hit_epks` (9464 / `0x24f8`). Its enabled
branch resolves target Render through `0x14013de80`, accesses Render `+0x260`,
checks the target's Player component, and calls `0x141f66ba0` with variant tag 1.
It also conditionally reads `disable_test_hit_animation_slow_down`
(9465 / `0x24f9`). The inspected callback/helper bodies do not call a pool
reducer or setter or write pool current/carry.

The positive effect-data chain continues: `0x141f66ba0` selects an effect record
(tag 1/value 1 selects row 0 through `0x14248c4f0`), reads path fields at
`+8/+0x2d`, converts time fields `+0x10/+0x14` by dividing by 1000, and reads
flags at `+0x2c/+0x35`. At `0x141f66d23` it invokes global service
`0x1445bf970` virtual `+8`. That service's concrete binding remains open, so
this is evidence of the effect route, not an exhaustive transitive exclusion
of numerical side effects.

## 8. Event dispatch reaches a programmable VM

The same consumer sends key `0x6876` at `0x141e921ba` through
`0x141e95620`, also gated by a matching action and `result+0x49`. Its captured
callable table `0x1435aa4a8`, invoke slot `+0x10`, selects leaf `0x141ea08a0`.
The leaf supplies `0x141de5ec0` with the selected event entry's payload, the
captured script context, `sourceActor.ownerEntity+0x28`, and the resolved
`targetEntity+0x28` when present. The target came from `0x141ebefd0` and was
stored at consumer stack `[rbp+0x168]` at `0x141e91639`. These offsets identify
passed interfaces; they do not independently recover their concrete classes.
The folded hash of ASCII `hit` equals `0x6876`, but this pass has not bound that
string as this event's registration name.

The numerical side-effect boundary is now more precise:

```text
result consumer 141e91580
  → event key 6876 / 141e95620
  → captured callable 1435aa4a8+10 / 141ea08a0
  → event program iteration 141de5ec0
  → program runner 141df49e0
  → bytecode interpreter 141ded710
  → opcode 13 / descriptor-selected callable 141dfa9f0
  → [callable.vtable+10]                 ← concrete target unresolved
```

`0x141de5ec0` checks for a nonempty program list and an allowed source interface
through `0x141de5a60`, installs source/target values through
`0x141de8600/0x141de5bc0`, and iterates **0x60-byte program records**. At
`0x141de5fbe` it calls `0x141df49e0`. The runner requires context `+0x58`
nonzero and `+0x59` zero, saves/restores current program `+0x68`, and at
`0x141df4ab5` invokes the interpreter with the program's byte stream.

The interpreter's audited unwind span contains 6,565 decoded instructions and
123 distinct direct callees. It dispatches opcode bytes `0..0x65` through the
102-entry jump table at `0x141df47b0`. The audited body has no direct call to
the four numerical reducers or the absolute pool setter; this bounded direct
observation excludes neither its callees nor its program-selected callbacks.
Its four direct virtual calls use the passed context interface's `+0x28` at
`0x141def09f`, `0x141defc59`, `0x141defcfe` and `0x141defda3`.

Opcode **13** starts at `0x141dee1d8`. It reads argument-count and function-index
bytes, selects a **72-byte descriptor** from program `+0x30` with
`0x141dfaa20`, requires descriptor `+0x40 == 2` through `0x141df49a0`, and
builds an argument range from the VM stack. Callsite `0x141dee2bf` enters
`0x141dfa9f0`; that bridge loads the callable at descriptor `+0x38` and invokes
its virtual `+0x10` at `0x141dfaa05`. **The actual descriptor and callable for
the Actor's selected `0x6876` event program have not been recovered.** This is
an explicit remaining route for numerical effects, rather than evidence that
the event is presentation-only.

There are also positive script-state writes. Opcodes **40, 41 and 42** start at
`0x141defc41`, `0x141defce6` and `0x141defd8b`, resolve a target context through
interface `+0x28`, and invoke keyed typed-value helpers. `0x141de8c70`
replaces/assigns typed values, `0x141de9a90` loops additions through
`0x141de9380`, and `0x141dea100` compares/removes values then stores through
`0x141de8c70`. Their observed containers use 40-byte values and 88-byte keyed
entries. No binding from these containers to Life/Mana/ES current or fractional
carry has been shown.

The ordinary startup guard on native action-context clones documented in
[stat evaluation](client-stat-evaluation.md) does not establish a no-op event
VM. Conversely, this executable VM does not prove independent client hit loss.
R3 remains open until a concrete selected event program/callable or another
result path carries a numerical amount into a proved resource receiver/setter.

## 9. Action/event evidence and additional byte anchors

The follow-up evidence uses ignored batches `r3-loss-action-{seeds,factory,
registry,registration,ctors,overrides}`, `r3-loss-concrete-actions`,
`r3-loss-factories-bound`, `r3-loss-melee-{followup,slot4b8,test-effect}`,
`r3-loss-script-receivers` and `r3-loss-script-descriptors-bound`.
`r3-loss-concrete-binding.json` retains the selected data rows, four table
spans/slots and the complete interpreter jump table. The partial failed batch
`r3-loss-script-call-descriptors` is not used as a completed audit.

Offline Ghidra export `r3-loss-vm-decompilations.jsonl` completed all four
requested bodies (`0x141ded710`, `0x141df49e0`, `0x141de5ec0`, `0x14136f210`),
SHA256 `fecf7160393b11c14e7620988534e10f68955597d98aee7944cf5d603c6c1645`.
The separately extracted raw interpreter text is
`r3-loss-0x141ded710.c` (145,439 bytes), SHA256
`6385df7410a0316973cd267c9d670f20a086a4dd529a4d4e3787037a7060a874`.
Local `r3-loss-vm-sidecars.json` records these and the three other extracted C
file hashes; these generated files remain uncommitted.

| Data table | SHA256 |
|---|---|
| ActionTypes | `eef7b004b13a3f32e487d5c102c41362b62131f83249f45194ece05b8518d9d6` |
| ActiveSkills | `baadc60b03734004624100d6aeab1a7b3a97728a4c625761d2b01c04b1404c75` |
| GrantedEffects | `ee02ccde977a7a7c444b3fb329867b65a931dda3ab89f204dfc96b15571a6b3b` |

| Purpose / VA span | File offset | SHA256 |
|---|---:|---|
| Fireball registration `[0x141973cd0,0x141973cfd)` | 26685648 | `cecc0a5bb0190fc31c48a3e0333b37b2404a6ca186009b4fc0fd33eba968bce6` |
| Melee flag 0 registration `[0x14196a070,0x14196a0a0)` | 26645616 | `2ab03a1b2a04e7f7b9e5623021dfd6aeda1a51e1cc2ceba2d94175b1d66b48e0` |
| Melee flag 1 registration `[0x14196a040,0x14196a070)` | 26645568 | `a5bcc8e8a911c25472f4d96ceb1449984f6201125dd7435ae0e0ebabe30021a9` |
| Fireball factory `[0x141988dd0,0x141988eb0)` | 26771920 | `56a1c63f13ac81fb4e96d98bf0795bfd432c281c2360106455c573e6c4f3ae02` |
| Melee flag 0 factory `[0x1419bbb70,0x1419bbc9d)` | 26980208 | `a01eef4cb4b070439a07068202f7786b67491af7985602d299313238333c3526` |
| Melee flag 1 factory `[0x1419bbca0,0x1419bbd80)` | 26980512 | `4a0e76ad393c380687d5bdefb4695448208be15bbfea77f092b38dff1bcb5a9d` |
| Fireball constructor `[0x141a8b150,0x141a8b1c9)` | 27829584 | `9513b412dc56bce9fafb57591ae1bdf291ccacb18b09b153e8303ba79602610b` |
| Melee flag 1 constructor `[0x141ac9dc0,0x141ac9e27)` | 28086720 | `dae44d0ef4dff2b3c66cd9db5d4f2e41021526e55808866223f12d9838587097` |
| Melee callback `[0x14136f210,0x14136f23e)` | 20375056 | `ad0e1a90356eb6b50c77a66def2a6bcfef7c00214b0087c6bfce11e0464c09ec` |
| Test-hit helper `[0x1419e4620,0x1419e4743)` | 27146784 | `39ca8955622dbe3d3fac7c62a3660fa716efa0b8f97834248e31b63fd1cdd8fb` |
| Speed getter `[0x141b55cb0,0x141b55d56)` | 28659888 | `af644618cefa95908c970a89ccca7e5249615c1d0f085dacf31bba6289f16673` |
| Event iteration `[0x141de5ec0,0x141de5ff3)` | 31347392 | `e2f65137bf6d8446055d0a96d100099972f7f2e95f735daa9ae9f28177705a38` |
| Interpreter `[0x141ded710,0x141df4948)` | 31378192 | `63ca6a3796bae102c0d4fb50017bb2ce9010d86a5e13dee9d62e75c809df0b89` |
| Callable bridge `[0x141dfa9f0,0x141dfaa17)` | 31432176 | `f6164daf10a8378c0920d45ae7cc6a7be0f2c82f8a7cea1b66146cf015d47401` |
| Fireball table `[0x143335d80,0x143336060)` | 53694848 | `de83e6fabedf0aad31f1dda07d424033903894929b2094d3c3582b210afcaa59` |
| Melee flag 0 table `[0x143517ba0,0x143518060)` | 55668640 | `bbccd48123f94edcf4292c0f7672aa3be55c7390ee6ee5ad9448b33c29af071f` |
| Melee flag 1 table `[0x143273730,0x143273a10)` | 52898608 | `775641251ecc13dbac9d9066cf0b01797da58180ac0a4d6d6e5a62516bd2431f` |
