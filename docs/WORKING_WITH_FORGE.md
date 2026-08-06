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

## 6. Choose Rust Or Data Implementation

Under Champion Identity, choose the implementation for the selected champion:

- `Rust-based` edits the existing Rust ability graphs and generates native Rust champion code.
- `Data-based` edits a separate Data ability graph and exports `champion/<id>.data_champion`.

The two graph types are independent. Forge does not convert the Rust graph into the Data graph or vice versa. Use the `Import Data` toolbar action to bring an existing `.data_champion` file into the project.

Data effect branches are edited recursively. Missing branches are created with their `Add Effect` button rather than being created just by drawing the editor. Native Data effects must reference a Native Effect defined under **Identity → Native Effects**.

For `RangeProjectile` and `LineRangeProjectile`, set `Apply ticks` to the numeric duration of the area application. `apply_type` values such as `AroundTarget` are only used by `RangeEffect`.

## 7. Build The Effect Graph

Add timed groups to control when effects occur. Add effects inside each group and use nested child editors to build branches.

Common patterns:

- Use `Target Self` to apply child effects to the caster.
- Use `Area` to create Friendly and Enemy target lists.
- Use `If` to filter or select targets for True and False branches.
- Use `Find Nearest With Buff` when a branch should find buffed allies or enemies independently of the current target.
- Use `Move To` after a target-producing effect to move toward the first selected target.
- Use `Play Animation` to apply a named animation to the caster for a configured duration.

`Spawn Unit` and `Draw Sprite` are not implemented yet. The SDK does not properly support the required behavior.

## 8. Configure Formulas

Damage, healing, shields, and similar numeric effects use formula editors.

- Set a flat Base value.
- Add terms with `+ Term`.
- Choose Caster or Target as the term source.
- Choose the stat used by the term.
- Set the percentage contribution.

The `Stacks` stat can be selected as a Caster or Target term and scales like any other stat term.

## 9. Configure Move To

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

## 10. Configure Play Animation

Add `Play Animation` from the `Status / Other` effect menu. Set:

- Animation Name
- Duration Ticks

The effect applies the Animation CC to the caster, not the current target.

## 11. Configure Projectiles

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

## 12. Export The Project

Use the project export workflow to generate the managed Rust workspace and mod assets. Forge clears and regenerates managed workspaces automatically. Managed export roots are also cleared before packaging to prevent stale generated files; imported/source-backed mods use conservative cleanup. Verify that the selected SDK path and export/mod paths are correct before exporting.

## 13. Install Rust For Building

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

## 14. Build The Mod

Forge runs Cargo in the exported managed workspace. Build from that workspace with:

```powershell
cargo build --release
```

The target PC must have Teamfight Manager 2, the stable SDK, and the configured mods directory available.

## 15. Deploy And Test

Copy or deploy the generated mod package/DLL to the configured Teamfight Manager 2 mods directory, then launch the game and test the champion.

If behavior is unexpected, first confirm that Forge generated and deployed the current managed workspace and DLL. Temporary generated diagnostic logging is disabled in normal builds.

## 16. VFX Assets

Projectile VFX assets can be selected with `Pick PNG`. `Draw Sprite` is not implemented yet because the SDK does not properly support it.
