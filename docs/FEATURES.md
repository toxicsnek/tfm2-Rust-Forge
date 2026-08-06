# TFM2 Rust Forge Features

TFM2 Rust Forge is a Windows desktop editor for creating Teamfight Manager 2 stable Rust mods. It generates the Rust champion code, mod assets, and managed workspace used to build the native mod DLL.

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

Adds a damage-absorbing shield layer to the current target using a formula. Shield duration is configured in ticks; newly created Shield effects default to 360 ticks. (6 seconds)

### Remove Shield

Removes every shield layer from the current target.

### Add Buff

Applies a named champion buff to the current target.    *Buffs defined under Identity - Buffs.*

### Remove Buff

Removes a named buff from the current target.   *Buffs defined under Identity - Buffs.*

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

### Set Persistent Flag

Sets a named boolean flag that persists through death for the current match. Flags are useful for one-time transitions such as switching modes after a counter reaches a threshold.

### Move To

Moves the caster using the ability's casting input:

- Targeting moves relative to the selected entity.
- Position moves relative to the cast position.
- Direction moves along the cast direction.

Move To supports minimum distance, maximum distance, signed X/Y landing offsets, a Move Away option, and an optional Duration Ticks value. Duration `0` moves immediately; positive durations interpolate the movement over that many simulation ticks and cancel when movement-blocking CC is applied. Nested Move To effects use the first target in the current target list.

### Spawn Unit

Creates a named unit at the current target position with configurable duration and count. **not implemented yet**

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

### Queue Effect

Queues a registered native effect by name after the configured delay.

### Draw Sprite

**not implemented yet** SDK does not properly support this.

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
