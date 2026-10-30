# Client default ranged-action selection (3.29.3b)

Default Attack does not require a different extracted ActionTypes name for a bow.
The native action factory lookup has a concrete weapon-type branch that selects
the second registry variant for `Bow` or `Wand`.
This supplement establishes that selector and its Actor provider; it does not
yet establish which of two registration sets wins during ordinary startup, or
an end-to-end default-bow numerical damage calculation.

## Identity and evidence

The input is the same original `Client/Game.exe` as
[contact production](client-hit-production.md): SHA256
`78a0e227189c263658a7f65a02f086428b859bfc114ae8e7ce8aab721ebd48d5`.
Addresses are preferred-image VAs with base `0x140000000`.
All findings are offline original-byte observations, using unwind ownership,
Capstone 5.0.3 and pefile 2024.8.26. No client/server execution, new traffic,
archive modification or new shared decompiler session was used.

Ignored records under `Tools/re/out/client-damage-audit/` are:

- `r3-default-bow-key-refs/audit.json`: the bounded 852/853 candidate scan.
- `r3-default-bow-provider/audit.json`: 12 function records covering the
  selector, provider construction/parser, enum resolver, setter and candidate
  action registration/construction.
- `r3-default-bow-registration-bound/audit.json`: four records for the exact
  provider leaf, configuration defaults and alternative registrations.
  The eight-byte leaf is explicitly bounded, rather than assigned a guessed
  unwind owner.
- `r3-default-bow-type-enum.json`: all 103 literal enum entries in the resolver's
  searched span, preserving names, values and entry addresses.
- `r3-default-bow-verify.py/json`: independent instruction, slot, enum and
  function-range rechecks against the original PE.

These are SHA-bound local audit records, not REA Evidence IDs or execution traces.

## Skill identity and conditional factory lookup

The extracted-data identity remains GrantedEffects row 468 `PlayerMelee` →
ActiveSkills row 108 `melee` → ActionTypes row 47 `Melee`, hash `0xb188`.
Stat-set row 476 includes native keys 852 `skill_can_fire_arrows` and 853
`skill_can_fire_wand_projectiles`; their presence alone does not select a factory.

Lookup `0x141941c90` performs a different concrete test:

1. It tries `0x141ee88f0` using the supplied skill and source Entity's world;
   when that resolves an Entity it obtains its Actor with `0x14013d8c0`.
   Otherwise it uses the source component vector/index at `+0x10`.
2. At `0x141941cea`, it calls the Actor's virtual `+0x148`.
3. A missing Actor/provider selects variant 0. Otherwise provider `+0x14c`
   equal to 14 or 9 selects variant 1. Value 37 selects variant 1 only when
   provider `+0x150` is 9. All other cases select variant 0.
4. It searches with `{action hash, variant}` at `0x141941d35` through
   `0x1419774d0`, then calls the found record's `+8` factory at `0x141941d5c`.
   The factory receives the preserved source context, skill and original
   fourth-argument byte; the latter is not the variant selector.

The numeric labels are backed by the native literal resolver, not by assumed
JSON row numbers. `0x1423f1bb0` searches 103 entries of 24 bytes beginning at
`0x142fc8fd0`, compares UTF-16 names and returns the entry's dword `+0x10`.

| Entry VA | Literal | Returned enum |
|---|---|---:|
| `0x142fc90c0` | `Bow` | 14 |
| `0x142fc98a0` | `Unarmed` | 37 |
| `0x142fc9948` | `Wand` | 9 |

Therefore the selector does positively distinguish bow/wand weapon state.
It does not read keys 852/853 at this branch.

## Provider identity and updates

The two audited derived Actor primary tables `0x1435daa70` and `0x14346e558`
bind `+0x148` to `0x141651830`, whose exact eight bytes implement
`return Actor + 0x598`.
The base Actor table `0x1435aa5d0` instead binds the null-return leaf
`0x1400ecd70`; the derived binding must not be generalized to every Actor.

Constructor `0x142192860`, called by the concrete client factory
`0x1416640c0`, constructs the embedded provider at Actor `+0x598` and installs
table `0x1435db980` at `0x1421929b0`.
It copies config `+0x88/+0x8c` to provider `+0x14c/+0x150` at
`0x142192955/0x14219295b`.
Config parser `0x141e8ac40` binds those fields to literal keys
`main_hand_unarmed_type` and `off_hand_unarmed_type`, invoking the enum resolver
at `0x141e8b1d4/0x141e8b24f`.
Config constructor `0x141e8bf20` defaults them to 37 and 104 respectively.
These are initialization inputs; they do not establish the equipped weapon
state of a later live action.

Provider table `+8` is setter `0x142199200`.
It compares and writes both fields, calls `0x141ee7fb0` for each retained active
skill, and checks for changes in skill eligibility bytes `+0xb0/+0xb1`.
That proves a mutable weapon-state provider and a skill-refresh consequence.
The equipment-to-setter caller has not been bound in this pass.

## Registry candidates and the remaining boundary

Registrar `0x141941bb0` gives its fourth argument a precise meaning: 1 inserts
variant 0, 2 inserts variant 1, and 0 inserts both.
The values are registration coverage, not evidence of client/server mode.

| Registration | Argument | Variant | Factory |
|---|---:|---:|---|
| `0x14196a070`, literal `Melee` | 1 | 0 | `0x1419bbb70` |
| `0x14196a040`, literal `Melee` | 2 | 1 | `0x1419bbca0` |
| `0x141262a00`, literal `Melee` | 1 | 0 | `0x1412caec0` |
| `0x1412629d0`, literal `Melee` | 2 | 1 | `0x1412cb0d0` |

For the first registration pair, the ordinary variant-1 factory allocates
`0x520` bytes and calls `0x141ac9dc0`. That constructor calls projectile-family
base `0x141b6eb70`, then installs final table `0x143273730`.
Its selected slots are `+0x618 → 0x141bb13a0` (the shared flight/contact path),
`+0x570 → 0x141b6f950` (accuracy path) and `+0x530 → 0x1412e34e0`.
These are a concrete candidate ranged continuation, not proof that an ordinary
default-bow session chooses this registration rather than the alternative.

Two next questions remain explicit:

- Bind the equipment producer to the provider's setter, including update order
  relative to action creation. The configuration's unarmed defaults are insufficient.
- Bind initialization and replacement order of both registration sets, then follow
  the winning ordinary bow action's contact continuation toward numerical hit data.

The selector/provider portion is established; ordinary startup selection and the
Hit-to-resource-loss chain remain Partial. A Fireball or MonsterMortarAttack
continuation cannot substitute for those missing links.
