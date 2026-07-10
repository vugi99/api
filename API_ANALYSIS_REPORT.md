# Nanos World API Analysis Report

Analysis of the bleeding-edge API JSON files for nanos world modding API.

---

## 1. Typos and Misspellings

| File | Location | Issue |
|---|---|---|
| `Classes/Player.json:239` | `SetCameraSocketOffset` efficiency | `"fat"` should be `"fast"` |
| `Structs/Vector.json:235` | `Size` return description | `"The lenght of the vector"` should be `"length"` |
| `Classes/BasePawn.json:207` | `RemoveStaticMeshAttached` description | `"from this enitity"` should be `"entity"` |
| `Classes/BaseActor.json:797` | `DimensionChange` event | `"changes it's dimension"` should be `"its dimension"` (possessive, no apostrophe) |
| `Classes/Player.json:777` | `DimensionChange` event | `"changes it's dimension"` — same apostrophe error |
| `Classes/Character.json:1716` | `ViewModeChange` event | `"changes it's View Mode"` — same error |
| `Classes/BasePickable.json:55` | `AddSkeletalMeshAttached` param desc | `"copying it's animation"` should be `"its"` |
| `Classes/BaseDamageable.json:95` | `Respawn` description | `"fullying it's Health"` should be `"filling its Health"`, and `"moving it to it's Initial Location"` should be `"its Initial Location"` |
| `Classes/Melee.json:3` | Class description | `"Charactes can hold it"` should be `"Characters"` |
| `Classes/Character.json:592` | `SetImpactDamageTaken` description | `"when being roamed by things"` should be `"run over by things"` |
| `Classes/WebUI.json:38` | `auto_resize` param desc | `"when screen changes it's size"` should be `"its"` |
| `Classes/Gizmo.json:116` | `Transform` event description | `"has it's transform updated"` should be `"its"` |
| `Classes/BaseVehicle.json:56` | `AddSkeletalMeshAttached` param desc | `"copying it's animation"` should be `"its"` |
| `Classes/BasePawn.json:56` | `AddSkeletalMeshAttached` param desc | `"copying it's animation"` should be `"its"` |
| `Classes/BasePawn.json:833` | `MoveComplete` event description | `"reaches it's destination"` should be `"its"` |
| `Classes/BaseDamageable.json:176` | `HealthChange` event description | `"When Entity has it's Health changed"` should be `"its"` |
| `StaticClasses/Server.json:396` | `KickByAccountID` description | `"by it's Account ID"` should be `"its"` |
| `StaticClasses/Server.json:412` | `BanByAccountID` description | `"by it's Account ID"` should be `"its"` |
| `StaticClasses/Server.json:427` | `Unban` description | `"by it's account ID"` should be `"its"` |
| `StaticClasses/Assets.json:76` (and others) | Get assets description | `"it's metadata"` should be `"its"` (multiple instances in file) |

---

## 2. Inconsistent or Incorrect Types

| File | Location | Issue |
|---|---|---|
| `Classes/BaseEntity.json:566` | `BroadcastRemoteInRadiusEvent` | Parameter `radius` uses type `"number"` — every other function in the API uses `"float"` or `"integer"`. Should be `"float"` for consistency. |
| `Classes/Character.json:1306` | `SetPhysicsAsset` | Parameter type is `"Other"` — not a valid nanos type. Should likely be `"PhysicsAssetPath"` or similar asset path type. |
| `Classes/CharacterSimple.json:73` | `SetPhysicsAsset` | Parameter type is `"Other"` — not a valid nanos type. Should likely be `"PhysicsAssetPath"` or similar asset path type. |
| `Enums.json:47-48` | `AssetType` enum values | Values are bare numbers (`2`, `4`, `8`...) while most other enums use strings (`"0"`, `"1"`...). Inconsistent format. |
| `Enums.json:267` | `CollisionChannel.All` | Value is `"(1 << 32) - 1"` — in Lua this overflows a 32-bit integer. Should likely be `"(1 << 23) - 1"` or similar. |
| `Classes/Sound.json:335` | `GetSoundType` return | Return type is `"float"` but should be `"SoundType"` (the enum). |
| `Classes/Character.json:498` | `SetDeathSound` parameter | Takes `type: "string"` but constructor takes `SoundPath`. Should be `SoundPath` for consistency. |
| `Enums.json` | `ConstraintMotion` entries | Uses `"Description"` (capital D) instead of `"description"` (lowercase) — inconsistent with all other enums in the file. |

---

## 3. Missing Descriptions / Name Fields

| File | Location | Issue |
|---|---|---|
| `Classes/BaseActor.json:490` | `IsInWater` return | Missing `name` field in return value |
| `Classes/BaseActor.json:676` | `HasNetworkAuthority` return | Missing `name` field |
| `Classes/Weapon.json:666` | `GetAmmoToReload` | Missing `description` field |
| `Classes/Weapon.json:808` | `GetCanHoldUse` | Missing `description` field |
| `Classes/Weapon.json:820` | `GetHoldReleaseUse` | Missing `description` field |
| `Classes/Prop.json:93` | `SetPhysicsDamping` params | `linear_damping` and `angular_damping` have empty descriptions `""` |
| `Enums.json` | `AnimationSlotType`, `LightType` | Missing `description` field on the enum definition itself |

---

## 4. Incorrect Authority / Logic Errors

| File | Location | Issue |
|---|---|---|
| `StaticClasses/Client.json` | `GetActorsInRadius` | Listed as `"authority": "server"` but this is in the **Client** static class. Should be `"authority": "client"`. |
| `Classes/BaseEntity.json:239` | `HasAuthority` | Description says `"spawned by the client side"` but the return description says `"false if it was spawned by the Server or true if it was spawned by the client"`. The semantics are inverted/confusing — `HasAuthority` should return `true` if the entity was spawned by the server (i.e. the caller has authority). |

---

## 5. Missing `network_distribution` Field

The following base classes are missing the `network_distribution` field, which other entities explicitly define:

| Class | Expected Value | Notes |
|---|---|---|
| `Sound` | `"none"` or `"client"` | Has `"authority": "client"` but no `network_distribution`. Compare to `Light` and `Trigger` which set `"none"`. |
| `Pawn` (BasePawn) | `"enabled"` | Base class for Character which has `"enabled"`. Missing from base. |
| `Pickable` (BasePickable) | `"enabled"` | Base class for Weapon/Grenade/Melee which have `"enabled"`. Missing from base. |
| `Paintable` (BasePaintable) | `"enabled"` | Base class, but no distribution info. |
| `Damageable` (BaseDamageable) | `"enabled"` | Base class, but no distribution info. |
| `Vehicle` (BaseVehicle) | `"enabled"` | Base class for VehicleWheeled/VehicleWater. Missing from base. |

---

## 6. Copy-Paste Errors (Wrong Class Name in Descriptions)

| File | Location | Issue |
|---|---|---|
| `Classes/BaseVehicle.json:83` | `AddStaticMeshAttached` description_long | `"Attaches a StaticMesh to this Pickable"` — should say **"Vehicle"** |
| `Classes/BaseVehicle.json:161` | `RemoveSkeletalMeshAttached` description | `"from this Pickable"` — should say **"Vehicle"** |
| `Classes/BaseVehicle.json:207` | `RemoveStaticMeshAttached` description | `"from this Pickable"` — should say **"Vehicle"** |

---

## 7. Inconsistent Constructor Parameters

| Class | Issue |
|---|---|
| `Melee` | Constructor includes `handling_mode` defaulting to `"HandlingMode.Torch"` — this seems wrong for a melee weapon. Should likely be `"HandlingMode.SingleHandedMelee"` or `"HandlingMode.DoubleHandedMelee"`. |
| `Grenade` | Constructor missing `HandlingMode` parameter entirely, unlike `Melee`. |
| `Weapon` | Constructor missing `HandlingMode` — set separately via `SetHandlingMode()`. Inconsistent with `Melee` which takes it in constructor. |

---

## 8. Inconsistent Authority Patterns

The API mixes `"authority": "authority"` and `"authority": "network-authority"` without clear documentation of when to use which:

| Pattern | Used By | Meaning |
|---|---|---|
| `"authority"` | `SetGravityEnabled`, `SetCollision`, `SetVisibility` | Requires full authority (server-side entity owner) |
| `"network-authority"` | `AddImpulse`, `SetForce`, `TranslateTo` | Only requires network authority (can be delegated to a client) |

Functions that conceptually do similar things use different authority levels, e.g.:
- `SetGravityEnabled` = `"authority"` but `AddImpulse` = `"network-authority"`

---

## 9. Struct Constructor Defaults

| Struct | Issue |
|---|---|
| `Rotator` | Constructor defaults: `yaw` defaults to `pitch`, `roll` defaults to `pitch`. So `Rotator(90)` creates `Rotator(90, 90, 90)`. This is unintuitive — most math libraries default unset components to 0. Not documented. |
| `Vector` | Constructor defaults: `Y` defaults to `X`, `Z` defaults to `X`. So `Vector(5)` = `Vector(5, 5, 5)`. Same issue. |

---

## 10. Missing Getters/Setters for Constructor Parameters

| Class | Constructor Param | Missing Function |
|---|---|---|
| `Light` | `source_radius` | No `SetSourceRadius` / `GetSourceRadius` |
| `Light` | `visible` | No `SetVisible` / `IsVisible` |
| `Light` | `max_draw_distance` | No `SetMaxDrawDistance` / `GetMaxDrawDistance` |
| `Light` | `use_inverse_squared_falloff` | No setter/getter |
| `Light` | `cone_angle` | No setter/getter (Spot-only) |
| `Light` | `inner_cone_angle_percent` | No setter/getter (Spot-only) |

---

## 11. Miscellaneous Issues

| File | Issue |
|---|---|
| `Enums.json` | `LightType.React` — likely should be `"Rect"` (for rectangular/rect light in UE5). `"React"` is not a standard UE light type. |
| `Classes/BaseEntity.json:554` | Stray trailing comma between `BroadcastRemoteEvent` and `BroadcastRemoteInRadiusEvent` array entries. Valid in JSON5 but may break strict JSON parsers. |
| `Classes/BaseActor.json:195` | `GetSocketTransform` return is a `table` with `Location`/`Rotation` but no `Scale` — other transform functions return full transforms. |
| `Classes/Character.json:357` | `SetAirControl` parameter `boost_multiplier` has default `"512"` but description says "Final result is clamped at 1" — a multiplier of 512 seems excessive for something clamped to 1. |
| `Classes/Prop.json` | `SetInteractionToolTipText` has `"authority": "client"` — this is a client-side visual but there's no way to sync it from server. If set on server, it won't appear on clients. |

---

## Summary of Critical Fixes Needed

1. **`Player.json:239`** — `"fat"` → `"fast"` (typo in efficiency)
2. **`Client.json` `GetActorsInRadius`** — Wrong authority (`"server"` → `"client"`)
3. **`Character.json:1306` and `CharacterSimple.json:73` `SetPhysicsAsset`** — Invalid type `"Other"`
4. **`Enums.json` `CollisionChannel.All`** — Overflow value `(1 << 32) - 1`
5. **`BaseVehicle.json`** — Three functions reference "Pickable" instead of "Vehicle"
6. **`Sound.json:335` `GetSoundType`** — Return type `"float"` should be `"SoundType"`
7. **`Melee.json` constructor** — Default `HandlingMode.Torch` seems incorrect
8. **`Enums.json` `LightType.React`** — Likely should be `"Rect"`
