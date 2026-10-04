## Changelog
- port to 26.3 NeoForge (built against NeoForge `26.3.0.38-beta`)
- sync upstream Litematica `26.3-0.29.1` (sakura-ryoko/litematica)
- require MaFgLib `0.6.0+mc26.3` (malilib `0.30.2`) and FoxifiedClassTweaker
- networking: Servux payload now uses NeoForge's native payload API via MaFgLib 0.6.0 (registered during mod construction)
- fix: schematic block entities not being prepared on NeoForge (`extractVisibleBlockEntities` now targets the NeoForge overload that is actually called)
- mods.toml: `logoFile` -> `iconFile`
