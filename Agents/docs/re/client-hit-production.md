# Client contact production and hit decisions (3.29.3b)

The concrete Fireball and Melee actions contain local contact and defense-hit
decisions. Fireball's projectile path queries the spatial grid and tests spell
dodge; the selected Melee path tests accuracy/evasion and changes the target's
hit entropy. These are positive local calculations, even though successful
ordinary startup disables the audited action-local native context builder in
[stat evaluation](client-stat-evaluation.md).
They do **not** establish a local numerical damage roll, critical roll,
mitigation calculation, or Hit-to-resource-loss chain.

The concrete continuation matters: Melee's tested-contact dispatch slot is empty
on the selected flag-0 factory object. Fireball's corresponding continuation
consumes already queued action feedback. A mode-byte reference in projectile
termination also gates visual effects; it is not a blanket exclusion of local
hit tests or proof of damage resolution.

## Identity, evidence and limits

| Input | Identity |
|---|---|
| `Client/Game.exe` | SHA256 `78a0e227189c263658a7f65a02f086428b859bfc114ae8e7ce8aab721ebd48d5` |
| Preferred image base | `0x140000000`; addresses are preferred-image VAs, not live ASLR addresses |
| `data/tables/Stats.json` | SHA256 `91c2ed42f6d8094a2704f56aea32f6f16243881b04aadbe4fdbbdd3a566f05da` |
| Stats convention | 23,346 rows; getter key `k` maps to JSON row `k−1` |
| Method | Original PE bytes, unwind/chained-function ranges, Capstone 5.0.3 and pefile 2024.8.26 |

This pass continued the static action/factory binding in
[stat evaluation](client-stat-evaluation.md) and the queue consumers in
[hit feedback](client-hit-feedback.md). It did not execute, launch, attach,
hook, or control the client or server. No new shared REA session or decompiler
project was opened. Decompiled signatures are not needed for the call/field
claims below; the argument forwarding follows instructions.

Ignored evidence under `Tools/re/out/client-damage-audit/` includes:

- `r3-contact-{action,dispatch,resolution,sites,geometry,accuracy,hit-request}/audit.json`:
  bounded function bodies, original-byte range hashes, callsites and selected references.
- `r3-contact-producers-verified/audit.json`: spatial-cell contact processing,
  Melee splash setup, and Fireball's queue continuation.
- `r3-contact-spatial/audit.json`: the grid walker, projectile placement and
  additional concrete table-installation references.
- `r3-contact-{flight,update}/audit.json`: the Fireball-bound flight producer,
  its contact query, and concrete constructors/factory final table stores.
- `r3-contact-impact/audit.json`: the non-piercing projectile continuation and
  its concrete termination callbacks.
- `r3-contact-tables.json`: selected table pointer locations/bytes, with the
  flag-0 Melee table bounded before its adjacent table at offset `0x590`.
- `r3-contact-verify.json`: independently rechecked table pointers, leaf bytes,
  key callsites, selected mode-byte references and function-byte hashes.
- `r4-contact-leaf-coverage/audit.json`: those six reviewed leaf/thunk spans
  re-exported as explicit function roots, preserving reference-only leaf ownership
  when the consolidated evidence manifest rechecks instruction boundaries.
- `r3-contact-entropy-{lifecycle/audit,verify}.json`: Stats component construction,
  its exact countdown leaf, and the predictive/queued-frame component tick sites;
  six function/range hashes and 15 instruction checks independently reverified.

These are original-byte audit records, not REA Evidence IDs. Raw pointers alone
do not establish object ownership. The factory/constructor stores and call
arguments below supply the concrete binding. Static reachability does not show
that a live session traversed a branch, nor establish prediction/lockstep mode
selection for every action variant.

## Concrete action binding

| Named registration | Factory / construction | Final primary table |
|---|---|---|
| Fireball, `0x141973cd0` | `0x141988dd0` → constructor `0x141a8b150`; table LEA `0x141a8b16c`, store `0x141a8b17e`; action type `0xaa4e` | `0x143335d80` |
| Melee, registration flag 0, `0x14196a070` | `0x1419bbb70` calls base constructor `0x141c98150`, then **overwrites** its table at `0x1419bbc67`; action type `0xb188` | `0x143517ba0` |
| Melee, registration flag 1, `0x14196a040` | `0x1419bbca0` → constructor `0x141ac9dc0`; table LEA `0x141ac9ddc` | `0x143273730` |

The registration flag's semantic name remains unresolved. It must not be
silently renamed “server/client” or “predictive/lockstep.” In particular, the
temporary Melee base table is not the final flag-0 object's table. The
CallAura/CastAura table from earlier broad action triage is not used as a proxy
for these named actions.

## Fireball produces spatial contacts

Fireball table `0x143335d80` slot `+0x618` points to `0x141bb13a0`.
That method calls `0x141ba99d0` at `0x141bb1dc8`. The latter's contact-query tail
preserves the action in `RDI` and the projectile's Positioned object in `RBX`:

1. `0x141baa544..0x141baa556` obtains the source Entity's world and its virtual
   `+0x180` spatial object.
2. `0x141baa55c..0x141baa57f` captures `{action, projectile Positioned}` and passes
   `Positioned+0x294` as the query position.
3. `0x141baa585` calls `0x141bb83c0`, which clamps the query bounds to the grid,
   iterates `0x38`-byte grid cells, and calls `0x141bbaf70` at `0x141bb8589`.
4. `0x141bbaf70` walks the cell's `0x10`-byte entries. It compares candidate
   position `+0x294/+0x298` against the query, subtracts the candidate radius at
   `+0x1e3`, checks target state and relationships, requires an existing
   `action+0x328` projectile Entity, and excludes the captured projectile itself.
5. At `0x141bbb0eb`, it passes `RCX=action` and `RDX=candidate Positioned` to
   `0x141ba7b30`. The target Entity is `candidate Positioned+8`.

This supplies a spatial producer and a concrete receiver; it is stronger than
finding a packet receiver or an unbound helper. There is another retained path
through `0x141bad450`, including its calls at `0x141bada4d/0x141baddb4`, but its
upstream callback lifetime is not required for the grid path above.

`0x141ba7b30` has further projectile/target guards before reaching the defense
test. At `0x141ba865e` it invokes action slot `+0x570` with the candidate
Positioned. On Fireball this resolves to `0x1419eee20`.

The projectile's later behavior also uses local probability:

| Site | Inputs and consequence |
|---|---|
| `0x141ba865e` | Fireball defense test; a true “avoided” result invokes `+0x4c0` and exits this contact |
| `0x141ba86bd..0x141ba86e2` | Reads target keys 6292 `avoid_chained_projectile_%_chance` when chained, plus 5070 `avoid_projectiles_%_chance`; source key 17224 `projectiles_cannot_be_avoided` bypasses this check |
| `0x141ba875e` → `0x141ba8770` | Seeds a local generator through `0x1419c29b0`, draws integer 0..99 through `0x1404d03d0`, and compares with that avoidance sum |
| `0x141ba8a15` | Invokes `+0x678 → 0x141ba96a0`, which decides piercing using pierce counts/flags and stat keys; its optional `projectile_chance_to_not_pierce_%` branch calls `0x140e3b450` |
| `0x141ba8a2e` | When piercing is allowed, calls `0x141baf5f0(action, 2, target Entity, false)`, then increments action `+0x454` |

The eight 32-bit words supplied to `0x1419c29b0` at the avoidance call are:
action sequence `+0x11a`, sub-action `+0x11c`, source Entity ID `+0x88`,
action `+0x2d0`, action byte `+0x2b8`, target Entity ID `+0x88`, phase byte
`+0x120`, and action `+0x448`. These inputs identify an avoidance roll; their
existence does not turn it into a damage-amount or critical roll.

### Fireball selects deterministic spell dodge

`0x1419eee20` invokes action slot `+0x770` at `0x1419eee34`.
Fireball's table points that slot at the exact false leaf `0x1400ed4a0`
(`xor al,al; ret`). Its selected branch therefore:

- gets the action's stat provider through `0x14193b350`, indexed by `+0x174`;
- supplies target Entity, sequence `+0x11a`, sub-action `+0x11c` and phase `+0x120`;
- passes **false** as the final random-mode flag;
- calls `0x141ebd2f0` at `0x1419eeeb2`, then negates its result to return “avoided.”

`0x141ebd2f0` reads source key 18751
`cannot_be_blocked_or_dodged_or_suppressed`, target key 15559
`dodge_all_the_things`, target key 838 `chance_to_dodge_spells_%`, source key
2596 `reduce_enemy_dodge_%`, and target key 5368 `extra_dodge_rolls`.
It derives the effective chance as an integer truncation of the target chance
times `(100−source reduction)/100`, with early bypasses for ineligible targets
and nonpositive chance. The target-provider selection can redirect through key
19294 `shaman_voodoo_pole_target_id`.

The final flag selects two different implementations. True calls
`0x140e3b310` repeatedly until a value is below 100. False, selected above,
packs the sequence/sub-action/phase and applies arithmetic with constants
127773, 16807, 2836 and 2147483647 before reducing to 0..99. Extra dodge rolls
are clamped to −1..1 and select the larger/smaller of two results as appropriate.
The returned boolean compares the resulting roll with effective dodge chance.
This is a concrete local deterministic defense roll.

## Melee produces an accuracy decision and contact records

Flag-0 Melee table `0x143517ba0` binds `+0x520 → 0x141c98700`.
That method compares the selected target and source positions/radii and attack
range; the in-range branch calls `0x141c9fe00` at `0x141c98a2b`.
`0x141c9fe00` retains the target at action `+0x348`, performs a source/target
spatial line test through `0x141d87160`, then calls `0x141c9abb0` at
`0x141c9fee3`. Additional stored targets take the same path at `0x141ca0086`.

`0x141c9abb0` calls `0x141c9aa80` at `0x141c9abe1`. The latter checks target
eligibility, including water-sphere ownership and trap/mine exclusions, then:

| Site | Dispatch / data |
|---|---|
| `0x141c9ab33` | action `+0x528 → 0x141c9a970`, passing the target Entity |
| `0x141c9aa69` | `0x141c9a970` calls `0x14193a8c0` with hand byte `action+0x311` and false final random-mode flag |
| `0x14193a928` | forwards source Entity, action provider and target to `0x141ebcff0` |
| `0x141ebd14f` | invokes `0x141ebc990`, the accuracy/evasion probability calculation, with a captured decision callback |
| `0x141c9ab81` | action `+0x530`, with target, contact context, saved accuracy boolean and flags |

`0x141c9aa80` returns its saved accuracy boolean even though the following
`+0x530` call is empty on this concrete table. `0x141c9abb0` retains the target
and writes that boolean at `+8` of an `0x10`-byte target record. Its own return
is the splash-path result, so its return must not be confused with that stored
accuracy bit. The outer caller stores groups of these results in action-owned
`0x20`-byte containers.

`0x141ebc990` reads hand-selected source accuracy keys 292/293, cannot-miss keys
688/689, target evasion 284 and cannot-evade key 2366, plus conditional modifiers.
It calls `0x1424ff510` at `0x141ebce43` and clamps the resulting hit probability.
Target overrides in `0x141ebcff0` include 19046 `attacks_never_hit_you`,
19045 `attacks_always_hit_you`, and 15559 `dodge_all_the_things`.
These are actual stat-provider calls, not immediate-value name matches.

### The selected Melee branch updates target entropy

The captured callback table is `0x1435abdd8`; its `+0x10` entry is
`0x141ee4920`. That exact leaf loads the probability from `RDX` into `XMM1`,
adjusts `this` by eight bytes, and tail-jumps to `0x141ebd1a0`.

The false final flag selected by the concrete Melee caller takes the entropy
branch, rather than the `0x140e3b310` branch:

1. The captured pointer resolves to the target Stats component from
   `0x14013de10`; its `+0x28` is the entropy integer and `+0x30` the initialization flag.
2. When `+0x30` is set, the seed combines target Entity ID times 436 with action
   sequence, applies the same 127773/16807/2836/2147483647 arithmetic, reduces
   modulo 100, stores `+0x28`, and clears `+0x30`.
3. It writes float 3.0 at component `+0x2c`, then adds
   `trunc(hitProbability×100)` to entropy `+0x28`.
4. If the resulting unsigned value is at least 100, it subtracts 100 and returns
   true; otherwise it returns false.

This is a positive state-changing hit decision on the target Stats component.
It does not mutate the Life component's embedded Life/Mana/ES pools.

### Entropy initialization and expiry use the actual component tick

The target getter `0x14013de10` resolves the literal component name `Stats`
at `0x1435a396c` and returns its Entity component-vector element. The ordinary
network Stats component is constructed by `0x1421c2ad0`, which installs table
`0x1435dd4e8` at `0x1421c2b28` and initializes the same fields used above:

| Instruction | Initial state |
|---|---|
| `0x1421c2b36` | integer entropy `+0x28 = 0` |
| `0x1421c2b3d` | float countdown `+0x2c = 3.0` (`0x40400000`) |
| `0x1421c2b44` | word `+0x30 = 1`: needs-initialization byte `+0x30 = 1`, neighboring byte `+0x31 = 0` |

This installed component table binds `+0x40 → 0x141e40bd0` and
`+0x48 → 0x1400f0f00` (`ret 0`). The exact tick leaf is
`[0x141e40bd0,0x141e40bf1)`. When byte `+0x30` is already set it returns
immediately. Otherwise it subtracts the supplied `XMM1` time from float
`+0x2c`, stores the result, and sets byte `+0x30 = 1` when the result is
nonpositive. **Expiry does not zero or replace entropy `+0x28`.** The next
eligible entropy decision observes the flag and re-seeds that integer.

This is connected to the actual world update, not just a timer-shaped leaf.
Predictive Entity updater `0x1421845b0` loads each component from Entity vector
`+0x10..+0x18`, passes its selected time in `XMM1`, and invokes `+0x40` at
`0x142184651`. The queued-frame paths invoke that same slot at
`0x14218482a` and `0x1421849b3`. For this Stats object those calls land in the
countdown leaf, while the separate `+0x48` phase is empty. The
[timing report](client-damage-timing.md) establishes how those paths obtain
seconds and select original/effective time according to world/entity guards.
The constructor, leaf and these three selected component-loop bodies contain
no reference to global byte `0x1445b38cd`.

Every execution of the entropy decision branch, whether it returns hit or miss,
refreshes the countdown to 3.0. Consequently the reset condition is expiration
after accumulated **selected simulation time** since the last such decision,
followed by re-seeding on the next such decision. A zero selected time does
not advance it. Early-return target/cannot-miss/always-hit paths can bypass the
decision callback. This does not establish three wall-clock seconds, a fixed
180-tick interval, or one reset per attack action.

Flag-1 Melee has a different `+0x570 → 0x141b6f950` path. Its actual `+0x778`
slot is the false leaf, selecting the accuracy helper with hand byte `+0x518`
and false final random-mode flag. Its neighboring `+0x770 → 0x141b6f810` is a
float speed getter, not the boolean slot used by that test. The two concrete
tables must not be combined by treating all similarly placed slots as booleans.

## What continues, and what the mode byte proves

The ordinary successful startup writes `1` to `0x1445b38cd` at `0x140101e50`;
the native action-context builder `0x14193be90` returns early when it is nonzero.
That previously established guard does not dominate every local contact helper.
The selected spatial and defense-decision bodies above contain no reference to
that byte. This is a bounded observation, not a whole-program reachability proof.

| Concrete continuation | Established operation / boundary |
|---|---|
| Flag-0 Melee `+0x530` | Pointer at `0x1435180d0` is `0x1400f0f00`, exact bytes `c20000` (`ret 0`). The tested contact does not dispatch a numerical resolver through this slot. |
| Fireball `+0x530` | Pointer at `0x1433362b0` is `0x1412e34e0`. It invokes `0x14193a170`, and optionally invokes it again under `+0x498`; this advances the action phase and consumes its matching **existing queued result range** through `0x141e91330`, as established in [hit feedback](client-hit-feedback.md). |
| Fireball `+0x4c0` | `0x141bae0c0` inserts the target's integer Entity ID into a sorted vector under `action+0x438`, suppressing duplicates. This is target bookkeeping, not pool loss. |
| Fireball non-piercing continuation | Contact processing retains the target at action `+0x3a8`; flight code calls `+0x4d0` at `0x141baa5ed`. This is the reviewed leaf `0x141ba68b0`, which clears `R8d` and jumps to `0x141ba68c0`. That function chooses projectile split/fork/chain/return/termination branches and calls `0x141baf5f0` with behavior codes 0..7. Its default termination calls that helper at `0x141ba6e63`, then invokes `+0x590` at `0x141ba6e9a`; Fireball's concrete `+0x590` is also `ret 0`. |
| Projectile `0x141baf5f0` mode check at `0x141baf662` | Selects an optional target effect Entity according to a packed definition mode at `+0x4b`: value 3 bypasses comparison, 5 excludes it, otherwise `(mode==1)` must equal the global byte. The selected block constructs an Entity and configures position/speed. This alone is not a numerical hit resolver. |
| Same function at `0x141baf89f` | Requires nonzero global byte, action `+0x598`, a false input flag, source MTX key 16486 `mtx_hyrris_ire_alternate_arrow_visuals`, and no target. Fireball's `+0x598` is false. This is an alternate-arrow visual branch, not exclusion of Fireball defense testing. |

Melee splash is not silently omitted from this boundary. `0x141c9abb0` can
call `0x141c98a60` after a successful accuracy result and a splash-enabled flag.
That function reads splash modifiers, creates a temporary action-stat context,
and calls `0x141c99480`; the latter queries nearby targets through
`0x141ca5ad0`. Temporary native-stat mutation is conditional on a context
obtained through guarded `0x14193be90`. This identifies contact expansion and
stat setup; the pass does not prove that every spatial callback/record consumer
is empty. In another follow-on record path, `0x141c99a20` invokes `+0x548`,
optional splash feedback, and `+0x580`. On the selected flag-0 table,
`+0x548 → 0x141c999b0` checks target eligibility, passes its Entity ID to
`0x141939f40` and thus consumes an existing queued feedback record;
`+0x580` is `ret 0`. The splash target's `+0x538` callback is the reviewed
leaf `0x141c99a10`, which forwards to that same `+0x548` slot. These add further
concrete feedback boundaries; they do not close every spatial callback or
alternative table.

## Remaining closure criteria

This pass establishes **contact → local defense decision** and two concrete
continuation boundaries. It leaves **accepted contact → numerical damage
amount/critical/mitigation → typed embedded resource-pool loss** unresolved.
The [resource receiver audit](client-hit-resource-boundary.md) identifies the
typed pool reducers but does not supply that missing edge.

A further positive claim must carry a candidate action/target from the paths
above into an amount-producing calculation, retain its actual context/provider
binding and mode guards, then show the amount reaching the typed pool receiver.
An RNG call used for dodge, projectile avoidance, piercing or trajectory is
insufficient. An empty slot on one final table is likewise insufficient to claim
that every action, splash callback, or alternate mode lacks numerical damage.
