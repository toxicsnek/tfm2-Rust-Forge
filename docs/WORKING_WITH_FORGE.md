# Working With TFM2 Rust Forge

## 1. Launch Forge

Run `tfm2_rust_forge.exe`. The editor itself does not require Rust to launch.

## 2. Configure Paths

Set the following paths in the project settings:

- Stable SDK root, normally:
  `C:\Program Files (x86)\Steam\steamapps\common\Teamfight Manager2\mod-sdk-stable`
- Teamfight Manager 2 mods root, normally:
  `C:\Program Files (x86)\Steam\steamapps\common\Teamfight Manager2\mods`

The SDK root must contain `mod-api-stable`.

## 3. Create Or Import A Project

Use a new project for a new mod or import an existing mod folder. Keep the project and exported workspace paths accessible to the current Windows user.

## 4. Configure The Champion

In the Identity and Stats sections:

- Set the champion ID and display name.
- Choose category and tags.
- Set icons and other identity assets.
- Configure base stats and growth stats.
- Define reusable named buffs under **Identity → Buffs**.
- Define named persistent counters and flags for mechanics that must survive death.

New champions start with base Attack 100, HP 1000, Defence 25, Magic Resistance 25, and Move Speed 1100. Their default growth is Attack 20, HP 100, Defence 5, Magic Resistance 5, and Move Speed 10. Other default stat values are zero.

Buffs should be defined before building effects that add, remove, or check them.

Persistent state names should be defined before using `Modify Persistent Counter`, `Set Persistent Flag`, `Persistent Counter`, or `Persistent Flag`.

`Modify Persistent Counter` and its related conditions use simulation-local Forge state. Do not use them for gameplay-critical values that must transfer from rush/background simulation into gameplay; use host-visible state such as persisted buffs instead.

Use the **Refresh Stacks** effect in Rust graphs or Native Effects when a data champion needs to reclaim missing persisted buffs or ModifyStats stacks after a host-side state reset. It does not add state that is already present and does not modify persistent counters.

### Aseprite Sprite Sources

Set the champion sprite source mode to **Aseprite**, then choose a `.ase` or `.aseprite` file with **Pick Aseprite**. Create frame tags in Aseprite; each tag becomes an animation key. Forge copies the source file directly during Build/Export, and the game prepares its runtime assets from the Aseprite user-data metadata. The source file is not modified.

Every tag must have a unique non-empty name. The game interprets the Aseprite metadata, frame ranges, playback modes, and transparent frames directly.

## 5. Configure An Ability

Select Attack, Skill, Skill2, Ultimate, or Passive and configure:

- Casting type
- Casting target
- Range
- Start timing
- Duration
- Cooldown
- Charges
- Description

Use Position casting for effects that need a cast location and Direction casting for effects that need a cast direction.

## 6. Configure Modify Stats

`Modify Stats` applies its deltas to the champion's level-scaled base stats. Each supported stat has an optional **Stat Scales With Stacks** checkbox and a **Per Stack** multiplier. A multiplier of `1` adds one point per current stack; a multiplier of `300` adds 300 points per current stack.

The `Stack` stat itself remains a direct delta and cannot scale from stacks. The same level and stack behavior is generated for normal champion effects and Native Effects.

## 7. Choose Rust Or Data Implementation

Under Champion Identity, choose the implementation for the selected champion:

- `Rust-based` edits the existing Rust ability graphs and generates native Rust champion code.
- `Data-based` edits a separate Data ability graph and exports `champion/<id>.data_champion`.

The two graph types are independent. Forge does not convert the Rust graph into the Data graph or vice versa. Use the `Import Data` toolbar action to bring an existing `.data_champion` file into the project.

Data effect branches are edited recursively. Missing branches are created with their `Add Effect` button rather than being created just by drawing the editor. Native Data effects must reference a Native Effect defined under **Identity → Native Effects**. Area Effects expose a `Target` dropdown that maps to the exported Data `target` field.

When importing a `.data_champion`, Forge searches the source mod root for extensionless asset references using `asset/<namespace>/...` paths. It can bring in local sprites, skill icons, visual assets, and `.mp3`, `.wav`, `.ogg`, or `.flac` SFX files. It also imports the source champion's localization entries and required `mod.override_info` remappings. Base-game assets remain runtime references and do not need to exist in the source mod folder.

For `RangeProjectile` and `LineRangeProjectile`, set `Apply ticks` to the numeric duration of the area application. `apply_type` values such as `AroundTarget` are only used by `RangeEffect`.

## 8. Configure Data Visuals

Under Champion Identity, define named visual assets in:

- **Effect Visuals**: `Animation` or `LoopAnimation` definitions exported as `view_effects`.
- **Projectile Visuals**: `Animated`, `Sprite`, or `ThreePhase` definitions exported as `view_projectiles`.
- **Buff Visuals**: `Animated` or `ThreePhase` definitions exported as `view_buffs`.

Data `ViewEffect`, `CasterViewEffect`, and projectile effect names select these definitions from dropdowns. Use the file picker for `Animation Path` or `Sprite Path` instead of entering game asset references manually.

Effect Visuals support `is_follow`. Projectile Visuals default `repeat` to `true`. ThreePhase visuals expose pre, loop, and remove tags plus optional Z ordering.

Forge converts source paths into SDK asset references during export. Animated Effect Visuals, Projectile Visuals, and Buff Visuals accept `.ase` and `.aseprite` files, which are copied directly into the exported mod. The game reads their Aseprite user-data metadata and prepares the related `#sheet` and `#anim` assets. If an animation file is selected as `name#anim.fanim`, Forge also looks beside it for `name#sheet.png` and copies both files into the exported mod's matching folder. Physical Sprite files are copied with their extensions, but Sprite references in `.data_champion` omit the image extension.

## 9. Build The Effect Graph

Add timed groups to control when effects occur. Add effects inside each group and use nested child editors to build branches.

Common patterns:

- Use `Target Self` to apply child effects to the caster.
- Use `Area` to create Friendly and Enemy target lists.
- Use `If` to filter or select targets for True and False branches.
- Use `Find Nearest With Buff` when a branch should find buffed allies or enemies independently of the current target.
- Use `Move To` after a target-producing effect to move toward the first selected target.
- Use `Play Animation` to apply a named animation to the caster for a configured duration.

`Spawn Unit` captures each returned unit ID and can track units by summoner and unit name. `Get Target` can select those live spawned units for child effects. Spawned units can optionally have `Block Attacks` or `Invincible` applied for their lifetime, including permanent protection when duration is `0`. Their selected Native Effect is queued once; put a `Queue Effect` targeting the spawned unit inside that Native Effect when it should repeat. `Draw Sprite` remains unavailable.

Place `Consume Timer Target` inside the successful child branch of a target selector such as `Enemies Around Target`. It stops queued effects only when that branch has a target. **Destroy Spawn On Trigger** defaults to false; enable it to call `entity_set_hp(spawn_id, 0)` and remove the spawned unit from tracking.

Use `Trigger On Initial Target` when a selector changes the target list but a final effect must operate on the original target, such as consuming or destroying a spawned timer target after nearby enemies were found.

## 10. Configure Formulas

Damage, healing, shields, and similar numeric effects use formula editors.

- Set a flat Base value.
- Add terms with `+ Term`.
- Choose Caster or Target as the term source.
- Choose the stat used by the term.
- Set the percentage contribution.

The `Stacks` stat can be selected as a Caster or Target term and scales like any other stat term.

Rust `If` conditions also support **Stat A >= B**. Configure one formula for Stat A and one for Stat B using flat values and Caster/Target stat terms. The comparison is calculated at runtime for the current target and respects **Target Caster** and **Negate**.

Rust `If` conditions support **Level >=**. Set the minimum level in the condition editor. It checks the current target by default; enable **Target Caster** to check the caster. **Negate** reverses the comparison.

## 11. Configure Move To

Move To reads its target source from the ability's casting type:

- `Targeting`: selected entity
- `Position`: cast position
- `Direction`: cast direction

Use:

- Duration Ticks to move immediately at `0`, or interpolate the movement over the specified number of ticks.
- Minimum Distance to force at least that much travel.
- Maximum Distance to stop short of a farther destination.
- Offset X/Y to adjust the final landing point.
- Move Away to reverse the travel direction.

## 12. Configure Play Animation

Add `Play Animation` from the `Status / Other` effect menu. Set:

- Animation Name
- Duration Ticks

The effect applies the Animation CC to the caster, not the current target.

## 13. Configure Projectiles

For Spawn Projectile:

1. Choose `Target` or `Linear` Move Kind.
2. Set Radius and Speed.
3. Configure the hit attack/casting types and target filter.
4. Enable On Caster if the projectile should start at the caster.
5. Enable Penetrate if it should continue through hits.
6. Set Max Targets when Penetrate is enabled; use `0` for uncapped behavior.
7. Give the projectile a unique Projectile Name.
8. Add effects under Effects On Hit.

On-hit effects apply to the entity struck by the projectile, not the parent effect's previous target.

The SDK currently supports exact one-hit and uncapped projectile behavior. A reliable per-projectile cap above one is not available because projectile hit callbacks do not expose projectile identity.

## 14. Export The Project

Use the project export workflow to generate the managed Rust workspace and mod assets. Forge clears and regenerates managed workspaces automatically. Managed export roots are also cleared before packaging to prevent stale generated files; imported/source-backed mods use conservative cleanup. Aseprite sources are copied without Forge-side extraction or rewriting. Verify that the selected SDK path and export/mod paths are correct before exporting.

## 15. Install Rust For Building

Rust is not required just to run Forge, but it is required to export/build mods.

1. Download and run `rustup-init.exe` from `https://www.rust-lang.org/tools/install`.
2. Choose the default installation and the MSVC toolchain.
3. Open a new PowerShell window.
4. Verify the installation:

   ```powershell
   rustc --version
   cargo --version
   ```

If Cargo reports a missing linker or Windows SDK, install Visual Studio Build Tools from `https://visualstudio.microsoft.com/visual-cpp-build-tools/` and select the `Desktop development with C++` workload.

## 16. Build The Mod

Forge copies the generated `Cargo.toml` and `src/` into the target mod folder before running Cargo there. This ensures the deployed DLL is built from the source files shipped with that mod. To build manually, run Cargo from the configured target mod folder:

```powershell
cargo build --release
```

The target PC must have Teamfight Manager 2, the stable SDK, and the configured mods directory available.

## 17. Deploy And Test

Copy or deploy the generated mod package/DLL to the configured Teamfight Manager 2 mods directory, then launch the game and test the champion.

If behavior is unexpected, first confirm that Forge generated and deployed the current managed workspace and DLL. Temporary generated diagnostic logging is disabled in normal builds.

## 18. VFX Assets

Projectile VFX assets can be selected with `Sprite Path` and copied into the exported mod. `Draw Sprite` remains unsupported because the SDK does not properly support it.
