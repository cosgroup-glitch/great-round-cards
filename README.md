# Great Round Cards

The editable card catalog for [Great Round](https://great-round.onrender.com). Only card definitions live here; the game source remains separate.

## Easiest way to add a card

1. Open the [Card Builder](https://great-round.onrender.com/card-builder.html).
2. Enter a unique name, effect text/colors, and stat changes. Click **ADD CARD TO CATALOG**, then **COPY UPDATED CATALOG**.
3. Open [cards.json in the GitHub editor](https://github.com/cosgroup-glitch/great-round-cards/edit/main/cards.json), replace the full file with the copied catalog, and **commit to main**.

That's it. The next newly hosted lobby/new match/rematch checks this repository automatically. No game code changes, server rebuild, or game deployment are needed. Existing matches retain their original catalog, including when extended with F5. Reload the builder before starting another editing session so you include the latest cards.

Only repository collaborators can commit updates. Other people can suggest changes through pull requests. Making the catalog visible does not give visitors permission to change the game.

## Edit JSON directly

Copy [card-template.json](card-template.json), paste it into the `cards` array in [cards.json](cards.json), and change the values. Use a comma between cards. Set `version` to a new label such as `alpha-2`; content changes are also detected by their hash even if the version label is unchanged.

```json
{
  "id": "iron-will",
  "name": "IRON WILL",
  "effects": [{ "text": "+25% maximum HP", "kind": "bonus" }],
  "modifiers": [{ "stat": "MaxHealth", "operation": "Multiply", "value": 1.25 }]
}
```

`id` must be unique and lowercase, using letters, numbers, and hyphens. `name` must also be unique. Effect lines are display text; modifiers are the gameplay changes. Keep them consistent.

### Colors

| kind | Appearance |
| --- | --- |
| `bonus` | Green |
| `new` | Yellow |
| `drawback` | Red |

### Operations

| operation | Example |
| --- | --- |
| `Add` | Ammo +3: `value: 3` |
| `Multiply` | +25%: `value: 1.25`; -25%: `value: 0.75` |
| `Override` | Set a stat to `value` |
| `ClampMin` | Keep a stat at least `value` for this modifier |
| `ClampMax` | Keep a stat at most `value` for this modifier |

Modifiers run in their listed order. Cards stack in the order they were picked. A card may contain several modifiers and 1?8 colored display lines. Optional `icon` selects a stat's existing icon; if omitted it uses the first modifier's stat.

### Supported stats

`MaxHealth`, `MoveSpeed`, `JumpImpulse`, `Damage`, `FireRate`, `ReloadTime`, `Ammo`, `BulletSpeed`, `BulletLifetime`, `BulletBounces`, `BulletSize`, `Spread`, `Recoil`, `BlockCooldown`, `BlockDuration`, `BodySize`, `ProjectileCount`, `RemoteControl`, `DrillDistance`, `BuckshotStacks`.

Movement, jump, bullet speed, bullet size, and body size use multipliers relative to their base. Time values use seconds. Fire rate is shots per second; spread is radians. Whole-number counts should use whole-number values. Remote control enables the existing mouse-steering/no-gravity mechanic when above zero. BuckshotStacks enables the existing randomized pellet mechanic; also set ProjectileCount/Spread as shown in the included Buckshot card.

You can combine mechanics the engine already supports. Entirely new mechanics still need engine code; JSON does not execute scripts or add custom abilities.

### A limit that other cards can bypass

Quick Guard has this optional field:

```json
"minimums": { "BlockCooldown": 1 }
```

It limits that card's reduction to one second without raising a cooldown another card already lowered. Other cards remain free to go below the limit. Do not use `ClampMin` for that behavior: that operation would raise an already lower value.

## Validation and fallback

[cards.schema.json](cards.schema.json) describes the format. The game also validates unique IDs/names, supported stats/operations, finite numbers, required fields, and valid base player stats. An invalid catalog is rejected as a whole; the server keeps its last valid catalog. If GitHub is unavailable on first startup, the original 14 cards are available as a fallback. Check [catalog status](https://great-round.onrender.com/api/cards) for version, card count, and validation errors.

[Card Builder](https://great-round.onrender.com/card-builder.html) only prepares a JSON file. Nothing is published until you commit it to GitHub.

## Card sets and per-lobby exclusions

Every card can specify `setId`; older entries default to `alpha`. Declare available sets in the root `sets` array using `id`, `name`, optional `description`, and optional `enabledByDefault` (default true). IDs must be unique. A card cannot refer to a missing set.

Hosts choose sets from **Edit Match Settings -> CARD SETS**, or **CARDS** in the lobby. Inside each set, hosts can disable individual cards. Guests can view these choices. Choices apply to that lobby and its rematches, and cannot change during an active match. At least one card must remain available to start. Main-menu **CARDS** shows the default card pool before joining a lobby.

### BUGTEST (off by default)

| Card | Test value |
| --- | --- |
| 99 AMMO | Magazine capacity 99 |
| AMMO 1000000 | Magazine capacity 1000000 |
| FIRE RATE 1000 | FireRate 1000 shots/second |
| 99 BOUNCES | 99 ricochets |
| HP 10000 | 10000 max HP |
| DAMAGE 10000 | 10000 total shot damage |
| RELOAD 0.1s | 0.1 second reload |
| ZERO BLOCK COOLDOWN | No cooldown between fresh block presses |
| BULLET SPEED X10 | 10x normal bullet speed |
| BULLET SIZE X8 | 8x normal bullet size |
| BULLET SIZE X0.1 | 1/10 normal bullet size |
| 32 PELLETS | 32 randomized pellets, total damage split across them |

These cards **set** values using `Override`; they do not add 99 ammo or multiply the current fire rate by 1000. High fire rates schedule multiple shots per simulation tick, so FIRE RATE 1000 produces its requested cadence. The 99-ammo test uses an actual 99-round magazine. There are no fixed gameplay ceilings for ammo or pellet count; counts must fit the engine's numeric representation and practical performance depends on the host. Large magazines use numeric ammo indicators rather than drawing one dot per round. BUGTEST is excluded from new lobbies unless the host enables it.
