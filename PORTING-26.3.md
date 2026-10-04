# Forgematica 26.3 port notes

Unofficial port of Forgematica (Litematica for NeoForge) to Minecraft 26.3 / NeoForge 26.3.0.38-beta.

- Base: `CagayakeGirls/litematica-neoforge` branch `neoforge/26.2` (Forgematica 0.5.0+mc26.2)
- Merged: `sakura-ryoko/litematica` branch `26.3` (Litematica 0.29.1)
- Library: `CagayakeGirls/malilib-neoforge` branch `neoforge/26.3` @ `5de33c93` (MaFgLib 0.6.0 = malilib 0.30.2)

This port was produced without being able to compile it (the Minecraft and NeoForge
mavens were unreachable from the environment it was made in). Everything below was
checked statically against the NeoForge 26.3.x patch set and MaFgLib 0.6.0 sources.

## What changed vs. upstream Litematica 0.29.1

| Area | Change | Why |
|---|---|---|
| Build | Architectury Loom (no-remap, NeoForge platform), MaFgLib instead of malilib, Iris API compile-only | Same setup as Forgematica 26.2 / MaFgLib 26.3 |
| Entry point | `team.cagayakegirls.forgematica.Forgematica` (`@Mod`) calls `Litematica.onInitialize()` and registers the config screen | NeoForge has no Fabric entrypoints |
| Networking | `ServuxLitematicaHandler` uses `IPayloadContext` instead of Fabric `ClientPlayNetworking.Context` | MaFgLib 0.6.0 replaced Fabric networking with NeoForge's payload API |
| Networking | Servux payload is registered in the mod constructor; `EntityDataManager.onGameInit()` only registers as a fallback | NeoForge only accepts payload types queued before `RegisterPayloadHandlersEvent`; `onGameInit()` runs too late |
| Rendering | `MixinLevelExtractor`: block-entity hook targets `extractVisibleBlockEntities(Camera, float, LevelRenderState, Frustum)` | NeoForge reroutes `extract()` to this overload; the vanilla 3-arg method is a dead delegate on NeoForge |
| Rendering | `MixinLevelRenderer`, `MixinRenderPipelines`, `MixinBindGroupLayouts` taken from upstream unchanged | Upstream 26.3 hooks `prepareChunkRenders*`, `executeOit`, `executeOutline`, `<clinit>`; NeoForge 26.3 does not change those signatures |
| Metadata | `neoforge.mods.toml`: `iconFile`, deps on `mafglib >=0.6`, `foxified_classtweaker`, MC `[26.3,26.4)` | 26.3 metadata changes, runtime access widener |
| CI | `.github/workflows/build.yml` builds MaFgLib 0.6.0 from source, then Forgematica; `release.yml` removed | MaFgLib 0.6.0 is not published; release.yml published to the original project's Modrinth/CurseForge IDs |

## Mixin review against NeoForge 26.3 patches

Every Litematica mixin target was cross-checked against the NeoForge 26.3.x patches.
Hits that were reviewed and need no change:

- `LevelExtractor.extract` `popPush` ordinal 2: NeoForge only adds a `popPush` at the end of the method.
- `LevelRenderer.render` (`addMainPass` INVOKE, `@Local terrainMatrix/profiler`): call and locals unchanged.
- `ChestBlock.mirror`: NeoForge adds its own MC-134110 fix; Litematica's HEAD-cancel still wins when its option is on.
- `LevelChunk.setBlockState` `isClientSide` ordinal 0: NeoForge appends a condition after the same call.
- `BlockStateModel.collectParts(random, parts)`: deprecated on NeoForge but still works (delegates with an empty level context), same output as vanilla for vanilla models.
- `DepthStencilState` 2-arg constructor: kept by NeoForge.
- All 167 `fi.dy.masa.malilib.*` imports resolve in MaFgLib 0.6.0.

## Known limitations / things to test first

1. Never compiled before handoff: expect possible compile errors on the first CI run.
2. Modded blocks with NeoForge context-sensitive models (connected textures etc.) render in schematics without world context.
3. Servux entity/block-entity sync (on servers running Servux) uses the new networking path: test it.
4. Iris/shaders: pipeline assignment path is new upstream code, untested on NeoForge Iris.

## Install (client)

- NeoForge 26.3.0.38-beta (or newer 26.3)
- `mafglib-0.6.0+mc26.3.jar` (from the CI artifact)
- FoxifiedClassTweaker 0.1.1 (Modrinth: `foxifiedclasstweaker`)
- `forgematica-0.6.0+mc26.3.jar`
