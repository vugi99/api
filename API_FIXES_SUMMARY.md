# Nanos World API Fixes Summary

A comprehensive summary of the fixes applied to the nanos world scripting API schemas (JSON files) according to `API_ANALYSIS_REPORT.md` and user instructions.

## 1. Proposals Cleaned
In [NEW_CLASSES_PROPOSAL.md](file:///c:/Users/alexa/Desktop/nanos/api/NEW_CLASSES_PROPOSAL.md):
- Removed **`Radar`** class proposal (minimaps can be built via WebUI or widgets).
- Removed **`Fog`** class proposal (fog effects can be achieved using Particles or Sky settings).
- Renumbered the remaining proposals (1 to 14) and updated the **Summary Table** accordingly.

## 2. API Bug Fixes and Consistency
We updated the following API files:

### Typos and Misspellings
- **`Classes/Player.json`**: Fixed `SetCameraSocketOffset` efficiency (`"fat"` -> `"fast"`) and `DimensionChange` event description (`"changes it's"` -> `"changes its"`).
- **`Structs/Vector.json`**: Fixed `Size` return description (`"lenght"` -> `"length"`).
- **`Classes/BasePawn.json`**: Fixed `RemoveStaticMeshAttached` description (`"from this enitity"` -> `"entity"`), `AddSkeletalMeshAttached` parameter description (`"it's"` -> `"its"`), and `MoveComplete` event description (`"it's"` -> `"its"`).
- **`Classes/BaseActor.json`**: Fixed `DimensionChange` event description (`"changes it's"` -> `"changes its"`).
- **`Classes/Character.json`**: Fixed `ViewModeChange` event description (`"changes it's"` -> `"changes its"`) and `SetImpactDamageTaken` description (`"when being roamed by things"` -> `"run over by things"`).
- **`Classes/BasePickable.json`**: Fixed `AddSkeletalMeshAttached` description (`"copying it's"` -> `"copying its"`).
- **`Classes/BaseDamageable.json`**: Fixed `Respawn` description (`"fullying it's"` -> `"filling its"`, `"it's Initial"` -> `"its Initial"`) and `HealthChange` event description (`"it's"` -> `"its"`).
- **`Classes/Melee.json`**: Fixed class description (`"Charactes"` -> `"Characters"`).
- **`Classes/WebUI.json`**: Fixed `auto_resize` parameter description (`"it's"` -> `"its"`).
- **`Classes/Gizmo.json`**: Fixed `Transform` event description (`"it's"` -> `"its"`).
- **`Classes/BaseVehicle.json`**: Fixed `AddSkeletalMeshAttached` parameter description (`"it's"` -> `"its"`).
- **`StaticClasses/Server.json`**: Fixed typos (`"it's"` -> `"its"`) in `KickByAccountID`, `BanByAccountID`, and `Unban` descriptions.
- **`StaticClasses/Assets.json`**: Fixed all 9 occurrences of `"it's metadata"` -> `"its metadata"`.

### Type Corrections
- **`Classes/BaseEntity.json`**: Fixed parameter `radius` in `BroadcastRemoteInRadiusEvent` using type `"number"` -> `"float"`.
- **`Classes/Character.json` & `Classes/CharacterSimple.json`**: Fixed `SetPhysicsAsset` parameter using invalid type `"Other"` -> `"PhysicsAssetPath"`.
- **`Enums.json`**:
  - Converted `AssetType` numeric values (`2`, `4`, `8`, etc.) into string values (`"2"`, `"4"`, `"8"`, etc.) for consistency with other enums.
  - Corrected `CollisionChannel.All` value from `"(1 << 32) - 1"` -> `"(1 << 23) - 1"` to prevent integer overflow in Lua.
  - Lowercased `"Description"` -> `"description"` keys under `ConstraintMotion` enum entries.
  - Corrected `LightType.React` key name -> `Rect`.
- **`Classes/Sound.json`**: Changed `GetSoundType` return parameter type from `"float"` -> `"SoundType"`.
- **`Classes/Character.json`**: Corrected `SetDeathSound` parameter type from `"string"` -> `"SoundPath"`.

### Missing Descriptions and Names
- **`Classes/BaseActor.json`**: Added missing `name` return fields for `IsInWater` (`"in_water"`) and `HasNetworkAuthority` (`"has_network_authority"`).
- **`Classes/Weapon.json`**: Added missing descriptions to `GetAmmoToReload`, `GetCanHoldUse`, and `GetHoldReleaseUse`.
- **`Classes/Prop.json`**: Added missing descriptions for parameters `linear_damping` and `angular_damping` in `SetPhysicsDamping`.

### Logic, Authority & Structure
- **`StaticClasses/Client.json`**: Corrected `GetActorsInRadius` authority from `"server"` -> `"client"`.
- **`Classes/BaseEntity.json`**: Clarified `HasAuthority` description and return text to clearly denote caller authority on client vs. server contexts.
- **`Classes/BaseEntity.json`**: Fixed a leading stray comma at the array item declaration before `BroadcastRemoteInRadiusEvent`.

### Missing `network_distribution` Properties
Added `"network_distribution": "enabled"` (or `"none"` where appropriate) to:
- `Classes/Sound.json` (`"none"`)
- `Classes/BasePawn.json` (`"enabled"`)
- `Classes/BasePickable.json` (`"enabled"`)
- `Classes/BasePaintable.json` (`"enabled"`)
- `Classes/BaseDamageable.json` (`"enabled"`)
- `Classes/BaseVehicle.json` (`"enabled"`)

### Copy-Paste Errors
- **`Classes/BaseVehicle.json`**: Corrected 3 descriptions in `AddStaticMeshAttached`, `RemoveSkeletalMeshAttached`, and `RemoveStaticMeshAttached` to refer to `"Vehicle"` instead of `"Pickable"`.

### Inconsistent Constructor Defaults
- **`Classes/Melee.json`**: Corrected default `handling_mode` parameter in the constructor from `"HandlingMode.Torch"` -> `"HandlingMode.SingleHandedMelee"`.

---
