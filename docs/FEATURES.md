# TFM2 Rust Forge Features

TFM2 Rust Forge is a Windows desktop editor for creating Teamfight Manager 2 stable Rust mods. It generates the Rust champion code, mod assets, and managed workspace used to build the native mod DLL.

## Implementation Modes

- **Rust-based** champions use the existing Rust ability graph and export generated native Rust champion code.
- **Data-based** champions use an independent typed Data ability/effect graph and export a `.data_champion` file.
- Rust and Data graphs are intentionally independent; changing implementation mode does not convert one graph into the other.
- Data champions may reference champion-scoped Native Effects when a mechanic requires native Rust behavior.

## Data Champions

Data champions can be imported with the `Import Data` toolbar action or created by selecting `Data-based` under Champion Identity. The Data editor supports Attack, Skill, Skill2, and Ultimate actions with recursive child effects.

Supported Data effect groups include:

- Damage and sustain: `Attack`, `ApAttack`, `FixedAttack`, `Heal`, `Shield`.
- Crowd control: `Stun`, `Bind`, `Airborne`, `Knockback`, `Grab`, `Pull`, `Fear`, `Charm`, `Taunt`, blocking, invisibility, and banish.
- Movement: rush, teleport, directional teleport, move-back, move-to, and rush-behind effects.
- Projectiles and areas: target and linear projectiles, delayed areas, periodic areas, range effects, parabolic projectiles, and shrinking barriers.
- Buffs and composition: add/remove buffs, casted effects, combine, delayed, self-targeting, random target, and switch branches.
- Native and presentation effects: native effect references, view effects, animations, and sound effects.

For Range and Line Range Projectiles, `Apply ticks` is exported as the SDK numeric `apply` field. `AroundTarget` and similar values belong to the `RangeEffect` `apply_type` field, not projectile `apply`.

Data export writes files to `champion/<champion-id>.data_champion` and validates Native effect references before writing.

## Projects And Champions

- Create a new Forge project or import an existing mod.
- Configure the stable SDK root and Teamfight Manager 2 mods root.
- Define champion identity, category, tags, icons, base stats, and growth stats.
- New champions default to base stats of Attack 100, HP 1000, Defence 25, Magic Resistance 25, and Move Speed 1100, with growth stats of Attack 20, HP 100, Defence 5, Magic Resistance 5, and Move Speed 10. Other default stat values are zero.
- Configure attack, skill, skill2, ultimate, and passive ability graphs.
- Export generated Rust source and shared mod assets.
- Build the exported managed workspace through Cargo.

## Ability Settings

Each ability can define:

- Description
- Range and growth range
- Start timing
- Total duration
- Cooldown
- Charges
- Casting type: Targeting, Position, Direction, or None
- Casting target
- Cancelable behavior
- Whether the ability can be used while moving

## Buffs

Champions can define reusable named buffs. Buff definitions include stat changes, multipliers, combat modifiers, crowd-control immunity, undying behavior, wall ignoring, duration, and permanent/timed behavior.

Add Buff, Remove Buff, and buff conditions use the champion's named buff definitions.

## Effects

### Deal Damage

Deals physical, magical, and/or fixed damage using formulas. Formula terms can reference caster or target stats.

### Heal

Heals the current target using a formula. Optional controls support healing the caster and healing enemies.

### Shield

Adds a damage-absorbing shield layer to the current target using a formula. Shield duration is configured in ticks; newly created Shield effects default to 360 ticks (6 seconds).

### Remove Shield

Removes every shield layer from the current target.

### Add Buff

Applies a named champion buff to the current target. Buffs are defined under **Identity → Buffs**.

### Remove Buff

Removes a named buff from the current target. Buffs are defined under **Identity → Buffs**.

### Apply CC

Applies a crowd-control effect such as stun, airborne, bind, fear, charm, taunt, or animation CC for the configured duration.

### Play Animation

Applies an Animation CC to the caster. Configure the animation name and duration in ticks. The animation name is truncated to the SDK's supported name capacity.

### Clear CC

Removes all crowd-control effects from the current target.

### Modify Stats

Changes the caster's base stat block. Non-stack values are calculated from the champion's base stats plus the configured value. Stack values accumulate from the current stack.

### Modify Persistent Counter

Adds or subtracts from a named champion counter stored outside the normal stat block. Counters persist through death for the current match.

The target defaults to the first current target. Enable **Target Is Caster** to modify the caster's counter instead.

### Reset Counter

Clears a named persistent counter. Clearing a counter that does not exist is harmless. The target defaults to the first current target; enable **Target Is Caster** to clear the caster's counter instead.

### Set Persistent Flag

Sets a named boolean flag that persists through death for the current match. Flags are useful for one-time transitions such as switching modes after a counter reaches a threshold.

Enable **Target Is Caster** to set the caster's flag instead of the first current target's flag.

### Move To

Moves the caster using the ability's casting input:

- Targeting moves relative to the selected entity.
- Position moves relative to the cast position.
- Direction moves along the cast direction.

Move To supports minimum distance, maximum distance, signed X/Y landing offsets, a Move Away option, and an optional Duration Ticks value. Duration `0` moves immediately; positive durations interpolate the movement over that many simulation ticks and cancel when movement-blocking CC is applied. Nested Move To effects use the first target in the current target list.

### Spawn Unit

**Not yet implemented.**

### Spawn Projectile

Creates a projectile with configurable:

- Target or Linear movement
- Radius
- Speed
- Attack type
- Casting type and target filter
- Target entity or target position
- Caster-origin spawning
- Penetration
- Maximum targets when penetration is enabled
- Effects applied to projectile-hit targets

When On Caster is enabled, the projectile starts at the caster and aims toward the selected target. Both Target and Linear movement use the Speed field.

Projectile names are used as native hit-effect registration names and should be unique within the mod.

The stable SDK supports exact one-hit and uncapped behavior. Reliable per-projectile caps above one require projectile identity support that the current SDK does not expose.

### Area

Finds targets inside a circle, rectangle, or cone and applies Friendly and Enemy child effects to the selected target lists.

### Target Self

Replaces the current target context with the caster for its child effects.

### If

Evaluates conditions and sends selected targets to True or False child branches. Maximum Targets limits each branch independently. A value of 0 disables both branches.

Conditions that read persistent state support **Target Caster**, which evaluates the caster's state instead of the current target's state.

### Get Target

Builds a target list for child effects. Selectors include nearest, farthest, lowest HP percentage, highest HP percentage, random, allies around the caster, enemies around the caster, and enemies around the target. Selectors support team filtering, champion-only filtering, range checks, and delayed child groups. Towers are excluded from selector results.

Maximum-target limiting is currently disabled for Get Target. Selectors that inherently choose one target, such as nearest or lowest HP percentage, still return one target.

### Periodic Effect

Registers a timed effect on each selected target. Configure interval, duration, and whether the first pulse is immediate. Periodic effects support stacking, maximum stacks, and refresh-on-reapplication behavior. Their pulses are dispatched from generated `on_update` logic, and the instance ends when its target dies.

### Native Effects

Defines reusable champion-scoped effects under **Identity -> Native Effects**. Each Native Effect contains timed child groups and is emitted as a stable native effect implementation and registration.

Use **Queue Effect** to select a defined Native Effect from a dropdown and queue it for a target after the configured delay. Native Effect names must be unique within the mod and are converted to valid generated Rust type names.

### Queue Effect

Queues a registered native effect by name after the configured delay.

### Draw Sprite

**Not implemented yet.** The SDK does not properly support this.

## Conditions

### Has Buff

Checks whether the current target has a named buff, optionally requiring a minimum count.

### Distance To

Checks the distance between caster and current target using an at-most or greater-than comparison.

### Below Equal HP Percent

Checks whether the current target's HP percentage is at or below the configured threshold.

### Is Enemy

Checks whether the current target's team differs from the caster's team. Missing entities fail the condition.

### Is Champion

Checks whether the current target is a champion.

### Is Tower

Checks whether the current target is a tower.

### Is Crowd Controlled

Checks whether the current target has active crowd control.

### Is Isolated

Checks whether the current target has no living ally within the configured range.

### Has Shield

Checks whether the current target has a shield.

### Has Stacks

Checks whether the current target's Stacks stat is greater than or equal to the configured Count.

### Persistent Counter

Checks whether a named persistent counter is greater than or equal to the configured Count.

### Persistent Flag

Checks whether a named persistent flag is set.

### Find Nearest With Buff

Finds the nearest living, targetable enemy or ally with a named buff and passes the selected targets to the True branch. Max Targets defaults to 1 and limits how many matching targets are selected.

## Target Flow

Effects can operate on target lists instead of only one target. Area and selector conditions create target lists, If branches partition them, and normal target effects apply to each selected target. Move To intentionally uses only the first selected target.
