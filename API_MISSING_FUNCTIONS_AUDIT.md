# Nanos World API — Missing Functions Audit

Audit of existing classes to identify missing getters/setters, missing events, missing utility functions, and gaps that would make the API more complete for a sandbox game on Unreal Engine 5.7 with Chaos Vehicles.

This document only covers **existing classes** — no new classes or base class refactors are proposed.

---

## Table of Contents

1. [VehicleWheeled (Chaos Vehicles)](#1-vehiclewheeled-chaos-vehicles)
2. [Vehicle (Base)](#2-vehicle-base)
3. [VehicleWater](#3-vehiclewater)
4. [Character](#4-character)
5. [CharacterSimple](#5-charactersimple)
6. [Pawn (Base)](#6-pawn-base)
7. [Player](#7-player)
8. [Actor (Base)](#8-actor-base)
9. [Entity (Base)](#9-entity-base)
10. [Prop](#10-prop)
11. [Pickable (Base)](#11-pickable-base)
12. [Weapon](#12-weapon)
13. [Melee](#13-melee)
14. [Grenade](#14-grenade)
15. [Light](#15-light)
16. [Sound](#16-sound)
17. [Particle](#17-particle)
18. [Trigger](#18-trigger)
19. [Cable](#19-cable)
20. [StaticMesh](#20-staticmesh)
21. [InstancedStaticMesh](#21-instancedstaticmesh)
22. [Billboard](#22-billboard)
23. [Text3D](#23-text3d)
24. [TextRender](#24-textrender)
25. [Decal](#25-decal)
26. [Canvas](#26-canvas)
27. [WebUI](#27-webui)
28. [Widget](#28-widget)
29. [Widget3D](#29-widget3d)
30. [SceneCapture](#30-scenecapture)
31. [Gizmo](#31-gizmo)
32. [Blueprint](#32-blueprint)
33. [Damageable (Base)](#33-damageable-base)
34. [Paintable (Base)](#34-paintable-base)
35. [Structs](#35-structs)
36. [Static Classes](#36-static-classes)
37. [Enums](#37-enums)

---

## 1. VehicleWheeled (Chaos Vehicles)

The Wheeled Vehicle class has extensive `Set*` functions for configuration but almost no corresponding getters. For a Chaos Vehicles system, modders need to read back vehicle state at runtime for HUDs, AI, diagnostics, etc.

### Missing Getters (Setters exist, no getter)

| Missing Function | Description | Notes |
|---|---|---|
| `GetEngineSetup()` | Returns current engine configuration (max_torque, max_rpm, idle_rpm, brake_effect, rev_up_moi, rev_down_rate, torque_curve) | Set by `SetEngineSetup` |
| `GetSteeringSetup()` | Returns current steering configuration (steering_type, angle_ratio, steering_curve) | Set by `SetSteeringSetup` |
| `GetTransmissionSetup()` | Returns current transmission configuration (final_ratio, change_up_rpm, change_down_rpm, gear_change_time, efficiency, forward_gear_ratios, reverse_gear_ratios) | Set by `SetTransmissionSetup` |
| `GetDifferentialSetup()` | Returns current differential configuration (differential_type, front_rear_split) | Set by `SetDifferentialSetup` |
| `GetAerodynamicsSetup()` | Returns aerodynamics config (mass, drag_coefficient, chassis_width, chassis_height, downforce_coefficient, center_of_mass_override) | Set by `SetAerodynamicsSetup` |
| `GetWheelSetup(index)` | Returns all configuration for a specific wheel by index | Set by `SetWheel` — no way to read back a wheel's configuration |
| `GetSteeringWheelSetup()` | Returns steering wheel location, radius, and rotation | Set by `SetSteeringWheelSetup` |
| `GetHeadlightsSetup()` | Returns headlights location and color | Set by `SetHeadlightsSetup` |
| `GetTaillightsSetup()` | Returns taillights location | Set by `SetTaillightsSetup` |
| `GetHornSound()` | Returns the horn sound asset path | Set by `SetHornSound` |
| `GetAutoStartEngine()` | Returns if engine auto-starts on enter | Set by `SetAutoStartEngine` |
| `GetCameraOffset()` | Returns the camera offset | Set by `SetCameraOffset` |
| `GetAutoUnflip()` | Returns if auto-unflip is enabled | Constructor param, no getter |

### Missing Runtime State Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetSpeed()` | Returns current forward speed in MPH or km/h | `GetRPM` and `GetGear` exist but no speed getter — essential for HUDs |
| `GetSpeedKPH()` | Returns current speed in km/h | Convenience overload |
| `GetThrottle()` | Returns current throttle input value (0-1) | Useful for HUDs and AI |
| `GetBrake()` | Returns current brake input value (0-1) | Useful for HUDs |
| `GetHandbrake()` | Returns if handbrake is active | Useful for HUDs |
| `GetSteeringAngle()` | Returns current steering angle | Useful for HUDs and wheel visualization |
| `GetEngineStarted()` | Returns if the engine is currently running | Set by `SetEngineStarted`, no getter |
| `IsHornActive()` | Returns if the horn is currently sounding | `Horn()` function exists, no state getter |
| `GetWheelCount()` | Returns the number of configured wheels | Useful for iteration |
| `IsTireFlat(wheel_index)` | Returns if a specific tire is flat | Set by `SetTireFlat`, no getter |
| `GetForwardDirection()` | Returns the vehicle's forward direction vector | Useful for effects, AI |
| `GetCurrentGearRatio()` | Returns the effective gear ratio including final ratio | Useful for diagnostics |
| `GetRPMNormalized()` | Returns RPM as a 0-1 normalized value | Useful for HUD gauges |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `EngineStart` | Triggered when engine is started | No event when engine state changes |
| `EngineStop` | Triggered when engine is stopped | No event when engine state changes |
| `GearChange` | Triggered when gear changes | Arg: old_gear, new_gear — essential for audio/HUD |
| `SpeedChange` | Triggered when speed changes significantly | Arg: old_speed, new_speed — useful for game logic |
| `Brake` | Triggered when brakes are applied | Arg: is_braking |
| `Handbrake` | Triggered when handbrake is toggled | Arg: is_active |
| `TirePuncture` | Triggered when a tire goes flat | Arg: wheel_index — gameplay event |
| `VehicleFlip` | Triggered when the vehicle is flipped (for auto-unflip) | Useful for gameplay logic |
| `Hit` (server-side) | The `Hit` event exists on `both` authority but there's no server-side equivalent for applying gameplay damage | Currently only `both` |

### Missing Setters

| Missing Function | Description | Notes |
|---|---|---|
| `SetMaxSteeringAngle(steering_angle)` | Convenience to set max steering angle for all wheels | Without modifying each wheel individually |
| `SetSuspensionMaxRaise(wheel_index, value)` | Adjust suspension at runtime per-wheel | Tuning gameplay vehicles |
| `SetSuspensionMaxDrop(wheel_index, value)` | Adjust suspension at runtime per-wheel | Tuning gameplay vehicles |
| `SetBrakeTorque(wheel_index, torque)` | Override brake torque per wheel at runtime | Useful for damage systems (e.g. broken brake) |
| `SetHandbrakeTorque(wheel_index, torque)` | Override handbrake torque per wheel at runtime | Useful for damage systems |
| `SetFrictionForceMultiplier(wheel_index, multiplier)` | Adjust tire grip at runtime | Useful for surface effects (ice, mud) |
| `SetCorneringStiffness(wheel_index, stiffness)` | Adjust cornering at runtime | Useful for gameplay |
| `SetHeadlightsEnabled(enabled)` | Toggle headlights on/off | Headlights are set up but no runtime toggle |
| `SetTaillightsEnabled(enabled)` | Toggle taillights on/off | No runtime toggle |
| `SetDriftMode(enabled)` | Adjust friction/slip thresholds for drifting | Common gameplay feature |

---

## 2. Vehicle (Base)

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetMesh()` already exists | — | Good |
| `GetDoors()` already exists | — | Good |
| `IsDoorOpen(seat_index)` | Returns if a specific door is open | Doors can be configured but no state query |
| `GetDriver()` | Returns the Character driving (seat 0) | `GetPassenger(0)` exists but no semantic alias |
| `GetSeatCount()` | Returns the number of seats | Useful for UI |
| `GetExplosionSettings()` | Returns current explosion settings | Set by `SetExplosionSettings` |

### Missing Functions

| Missing Function | Description | Notes |
|---|---|---|
| `RemoveDoor(seat_index)` | Removes a configured door | `SetDoor` exists but no removal |
| `SetSeatLocation(seat_index, location, rotation)` | Adjust seat position at runtime | Useful for vehicle customization |
| `GetSeatLocation(seat_index)` | Gets the seat position | No way to query seat position |
| `SetEngineLocation(location)` | Set the engine/effect location | For custom vehicles |
| `Explode()` | Force the vehicle to explode | Characters have damage, vehicles don't have a manual explode |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `EngineStart` | Engine was started | No event |
| `EngineStop` | Engine was stopped | No event |
| `GearChange` | Gear changed | No event |
| `DoorOpen` | A door was opened | No event for door state |
| `DoorClose` | A door was closed | No event for door state |
| `PassengerChange` | Any passenger enters or leaves | Aggregated event for all seats |

---

## 3. VehicleWater

Very minimal API. For a water vehicle (boats, jet skis) much more is expected.

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetEngineOffset()` | Returns the engine offset | Set by `SetEngineOffset` |
| `GetThrustStrength()` | Returns the thrust strength | Set by `SetThrustStrength` |
| `GetSpeed()` | Returns current speed | No way to query speed |
| `GetCurrentThrottle()` | Returns current throttle | No way to query input |
| `IsEngineRunning()` | Returns engine state | No state query |

### Missing Setters

| Missing Function | Description | Notes |
|---|---|---|
| `SetMaxSpeed(max_speed)` | Limit boat speed | No speed control |
| `SetSteeringAngle(angle)` | Set rudder/steering angle | AI cannot steer boats |
| `SetBuoyancy(location, force)` | Configure buoyancy point | Physics tuning |
| `SetWaterDrag(drag)` | Configure water drag | Physics tuning |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `EngineStart` | Engine started | No event |
| `EngineStop` | Engine stopped | No event |
| `SpeedChange` | Speed changed | No event |
| `Splash` | Vehicle splashes into water | Gameplay event |

---

## 4. Character

The Character class is the most feature-rich but still has gaps.

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetMeshAsset()` | Returns the SkeletalMesh asset path | `SetMesh` exists but `GetMesh` is on Pawn base — not clear if it returns the asset or the component |
| `GetDeathSound()` | Returns the death sound asset | Set by `SetDeathSound`, no getter |
| `GetPainSound()` | Returns the pain sound asset | Set by `SetPainSound`, no getter |
| `GetFOVMultiplier()` | Returns the FOV multiplier | Set by `SetFOVMultiplier`, no getter |
| `GetHighFallingTime()` | Returns high falling time threshold | Set by `SetHighFallingTime`, no getter |
| `GetRagdollStandUpCooldown()` | Returns ragdoll stand-up cooldown | Set by `SetRagdollStandUpCooldown`, no getter |
| `GetRadialDamageToRagdoll()` | Returns radial damage ragdoll threshold | Set by `SetRadialDamageToRagdoll`, no getter |
| `GetFootstepVolumeMultiplier()` | Returns footstep volume | Set by `SetFootstepVolumeMultiplier`, no getter |
| `GetPunchDamage()` | Already exists | Good |
| `GetJumpZVelocity()` | Already exists | Good |
| `GetCanJump()` | Returns if character can jump | `SetCanJump` exists, no getter |
| `GetCanDive()` | Returns if character can dive | `SetCanDive` exists, no getter |
| `GetCanDeployParachute()` | Returns if parachute deploy is allowed | `SetCanDeployParachute` exists, no getter |
| `GetCanSprint()` | Already exists | Good |
| `GetCurrentAimMode()` | Returns current aim mode | `GetWeaponAimMode` exists, but separate from character aim |
| `GetAnimationBlueprint()` | Returns the current animation blueprint path | No getter for animation blueprint |

### Missing Setters

| Missing Function | Description | Notes |
|---|---|---|
| `SetCapsuleHalfHeight(height)` | Individual capsule dimension control | `SetCapsuleSize` sets all three at once |
| `SetCapsuleRadius(radius)` | Individual capsule dimension control | No fine-grained control |
| `SetViewDistance(distance)` | Control when character is rendered | No view distance control |
| `SetMovementSpeed(walking, sprinting, crouching)` | Convenience to set all movement speeds at once | Currently scattered across multiple functions |
| `SetMaxAccelerationSettings(walking, falling, swimming, etc.)` | Already exists as `SetAccelerationSettings` | — |
| `SetMorphTargetsFromTable(morphs_table)` | Batch set multiple morph targets | Currently one-at-a-time only |
| `SetAimSpreadMultiplier(multiplier)` | Control weapon spread from character | No aim spread control |
| `SetCameraShake(intensity, duration)` | Trigger camera shake | No camera shake function |
| `SetNightVision(enabled)` | Toggle night vision post-process | Common FPS feature |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `Damage` | A more specific damage event with full hit info | `TakeDamage` exists on Damageable but not Character-specific |
| `Revive` | When character is revived from death | No revive event |
| `AimStart` | When character starts aiming | Only `WeaponAimModeChange` exists |
| `AimStop` | When character stops aiming | Only `WeaponAimModeChange` exists |
| `SprintStart` | When character starts sprinting | Only `GaitModeChange` exists |
| `SprintStop` | When character stops sprinting | Only `GaitModeChange` exists |
| `CrouchStart` | When character starts crouching | Only `StanceModeChange` exists |
| `CrouchStop` | When character stops crouching | Only `StanceModeChange` exists |
| `JumpStart` | When character starts jumping | No event (only `FallingModeChange`) |
| `Land` | When character lands from a fall | No landing event — `FallingModeChange` covers it but a dedicated Land event would be cleaner |
| `SwimStart` | When character enters water | Only `SwimmingModeChange` |
| `SwimStop` | When character exits water | Only `SwimmingModeChange` |
| `VehicleEnter` | When character enters a vehicle | `EnterVehicle` exists as event but `CharacterEnter` on Vehicle is different |
| `VehicleLeave` | When character leaves a vehicle | `LeaveVehicle` exists as event |
| `MeleeAttack` | When character performs a melee attack (punch) | `Punch` event exists but no detailed melee attack info |

---

## 5. CharacterSimple

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetMaxAcceleration()` | Returns max acceleration | Set by `SetMaxAcceleration` |
| `GetSpringArmSettings()` | Returns spring arm configuration | Set by `SetSpringArmSettings` |
| `GetRotationSettings()` | Returns rotation settings | Set by `SetRotationSettings` |
| `GetSpeedSettings()` | Returns speed settings | Set by `SetSpeedSettings` |
| `GetPawnSettings()` | Returns pawn rotation settings | Set by `SetPawnSettings` |
| `GetAirControl()` | Returns air control settings | Set by `SetAirControl` |
| `GetAnimationBlueprint()` | Returns animation blueprint path | Set by `SetAnimationBlueprint` |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `Death` | When character dies | Not inherited from Damageable? (may need explicit re-declaration) |
| `TakeDamage` | When character takes damage | Same concern |
| `HealthChange` | When health changes | Same concern |

---

## 6. Pawn (Base)

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetJumpZVelocity()` | Returns jump velocity | Set by `SetJumpZVelocity`, no getter |
| `GetGravityScale()` | Already exists | Good |
| `GetFlyingMode()` | Already exists | Good |
| `GetCapsuleSize()` | Already exists | Good |
| `GetMesh()` | Already exists | Good |
| `GetBrakingSettings()` | Returns braking configuration | Set by `SetBrakingSettings`, no getter |
| `GetAIAvoidanceSettings()` | Returns RVO avoidance settings | Set by `SetAIAvoidanceSettings`, no getter |
| `GetMeshSettings()` | Returns mesh settings | Set by `SetMeshSettings`, no getter |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `Jump` | When character jumps | No event on Pawn base |
| `Land` | When character lands | No event on Pawn base |
| `CrouchStart` / `CrouchEnd` | Crouch state changes | Not present on Pawn |
| `SprintStart` / `SprintEnd` | Sprint state changes | Not present on Pawn |

---

## 7. Player

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetCameraFOV()` | Returns current camera FOV | `SetCameraFOV` exists, no getter |
| `GetCameraSocketOffset()` | Returns camera socket offset | `SetCameraSocketOffset` exists, no getter |
| `GetCameraSpeedSettings()` | Returns camera speed settings | `SetCameraSpeedSettings` exists, no getter |
| `GetVOIPLocalSetting()` | Returns local VOIP setting | `SetVOIPLocalSetting` exists, no getter |
| `GetVOIPGlobalChannelSetting(channel)` | Returns global VOIP channel setting | `SetVOIPGlobalChannelSetting` exists, no getter |
| `GetVOIPLocalMaxDistance()` | Returns local VOIP max distance | `SetVOIPLocalMaxDistance` exists, no getter |
| `GetVOIPLocalVolume()` | Returns local VOIP volume | `SetVOIPLocalVolume` exists, no getter |
| `GetVOIPGlobalVolume()` | Returns global VOIP volume | `SetVOIPGlobalVolume` exists, no getter |
| `GetVOIPGlobalHighPassFilter()` | Returns high-pass filter | `SetVOIPGlobalHighPassFilter` exists, no getter |
| `GetVOIPGlobalLowPassFilter()` | Returns low-pass filter | `SetVOIPGlobalLowPassFilter` exists, no getter |
| `IsCameraFading()` | Returns if camera is currently fading | `StartCameraFade` exists, no state query |
| `GetPing()` | Already exists | Good |
| `GetControlledPawn()` | Alias for `GetControlledCharacter` that returns Pawn type | More semantically correct |

### Missing Setters

| Missing Function | Description | Notes |
|---|---|---|
| `SetMouseSensitivity(sensitivity)` | Control mouse sensitivity | Common modding need |
| `SetInputMode(mode)` | Switch between game/UI/both input modes | Common for menus |
| `LockMouseCursor(lock)` | Lock/unlock mouse cursor | Common for FPS |
| `SetHUDVisible(visible)` | Toggle HUD visibility | Common gameplay need |
| `SetCrosshairVisible(visible)` | Toggle crosshair | Common gameplay need |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `Chat` | When player sends a chat message | No chat event |
| `Command` | When player executes a console command | No command event |
| `InputAction` | Generic input action event | Only `Input.BindAction` exists |
| `死亡` (Death) | When the player dies | Not on Player, only on Damageable |
| `Respawn` | When the player respawns | Not on Player, only on Damageable |
| `DamageTaken` | When player takes damage | Not on Player |

---

## 8. Actor (Base)

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetRootComponentType()` | Returns the root component type (StaticMesh, SkeletalMesh, etc.) | Useful for debugging |
| `GetOwner()` | Returns the owning entity | UE concept — useful for attachment chains |
| `IsSimulatingPhysics()` | Returns if physics simulation is active | No physics state query |
| `IsAttachedTo(other)` | Returns if this actor is attached to a specific other | `GetAttachedTo` exists but no direct check |
| `GetDistanceTo(other)` | Returns distance to another actor | Utility function, currently requires manual location math |
| `GetDirectionTo(other)` | Returns normalized direction to another actor | Utility function |
| `IsOverlapping(other)` | Returns if this actor overlaps with another | Useful for custom collision logic |
| `GetAttachedSocketName()` | Returns which socket/bone this actor is attached to | `AttachTo` takes a bone_name but no getter |

### Missing Functions

| Missing Function | Description | Notes |
|---|---|---|
| `SetActorTickEnabled(enabled)` | Enable/disable actor ticking | Performance optimization |
| `SetActorTickInterval(interval)` | Set the tick interval | Performance optimization |
| `SetActorScale(scale)` | Already exists as `SetScale` | — |
| `K2_SetActorLocationAndRotation(location, rotation, sweep)` | Set both in one call | Currently separate calls |
| `MakeVisible()` / `MakeInvisible()` | Shorthand for `SetVisibility(true/false)` | Convenience |
| `TeleportTo(location, rotation)` | Instant teleport (no physics) | `SetLocation`/`SetRotation` exist but Teleport is cleaner |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `Collision` | When this actor collides with another | No generic collision event on Actor |
| `Overlap` | When this actor overlaps with another | Only on Trigger |
| `Tick` | Per-frame update on this actor | No per-actor tick event |

---

## 9. Entity (Base)

### Missing Functions

| Missing Function | Description | Notes |
|---|---|---|
| `GetAllValues()` | Returns all values as a table | `GetAllValuesKeys` exists but not the values themselves |
| `HasValue(key)` | Check if a value exists without getting it | Useful for conditional logic |
| `RemoveValue(key)` | Remove a specific value | No way to remove entity values |
| `GetPackage()` | Returns the package that spawned this entity | Useful for multi-package debugging |
| `GetSpawnTime()` | Returns when this entity was spawned | Useful for age-based logic |
| `GetAge()` | Returns how long this entity has existed | Convenience |
| `IsValid()` already exists | — | Good |
| `IsDestroyed()` | Returns if entity is being destroyed | `IsBeingDestroyed` exists but naming is clearer |

---

## 10. Prop

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetPhysicsDamping()` | Returns linear and angular damping | Set by `SetPhysicsDamping`, no getter |
| `GetCenterOfMass()` | Returns center of mass location | Useful for physics tuning |
| `IsGrabbed()` | Returns if this prop is currently grabbed | `GetHandler` exists but boolean check is cleaner |

### Missing Setters

| Missing Function | Description | Notes |
|---|---|---|
| `SetLinearDamping(damping)` | Set linear damping individually | `SetPhysicsDamping` sets both |
| `SetAngularDamping(damping)` | Set angular damping individually | `SetPhysicsDamping` sets both |
| `SetSimulatePhysics(enabled)` | Toggle physics simulation | No physics toggle |
| `SetCollisionEnabled(enabled)` | Enable/disable collision | No collision toggle (only SetCollision on Actor) |
| `SetCollisionResponse(channel, response)` | Per-channel collision response | Fine-grained collision control |
| `SetCustomGravity(gravity)` | Override gravity for this prop | `SetGravityEnabled` on/off only |
| `SetMaxAngularVelocity(velocity)` | Limit angular velocity | Useful for gameplay |
| `SetMaxLinearVelocity(velocity)` | Limit linear velocity | Useful for gameplay |
| `SetPhysicsLinearVelocity(velocity)` | Set velocity directly | Only impulse available, no direct velocity set |
| `SetPhysicsAngularVelocity(velocity)` | Set angular velocity directly | Only impulse available |
| `SetEnableGravity(enabled)` | Already exists on Actor | — |
| `SetVisibilityDamping(factor)` | Dampen visibility changes (for UI) | Not applicable |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `Destroy` | When prop is destroyed | Inherited from Entity |
| `GrabStart` | When grab begins | `Grab` event exists |
| `GrabEnd` | When grab ends | `UnGrab` event exists |

---

## 11. Pickable (Base)

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetAttachmentSettings()` | Returns attachment settings (location, rotation, socket) | Set by `SetAttachmentSettings`, no getter |
| `GetCrosshairMaterial()` | Returns the crosshair material path | Set by `SetCrosshairMaterial`, no getter |
| `IsPickable()` | Returns if this pickable can be picked up | Set by `SetPickable`, no getter |
| `CanUse()` | Returns if this pickable can be used | Set by `SetCanUse`, no getter |
| `IsInUse()` | Returns if this pickable is currently being used | Useful for UI |

---

## 12. Weapon

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetFireMode()` | Returns the current fire mode (auto/semi/burst) | No fire mode getter |
| `GetMaxSpread()` | Returns max spread | Set by `SetSpread`, but no max spread concept |
| `GetMaxRecoil()` | Returns max recoil | Set by `SetRecoil` |
| `GetWallbangSettings()` | Returns wallbang configuration | Set by `SetWallbangSettings`, no getter |
| `GetDamageSettings()` | Returns base damage | `GetDamage` exists |
| `IsReloading()` | Returns if weapon is currently reloading | No state query |
| `IsFiring()` | Returns if weapon is currently firing | No state query |
| `GetFireMode()` | Returns fire mode | Not exposed |
| `GetCurrentSpread()` | Returns current dynamic spread (not base) | Base spread is set, current spread changes during firing |
| `GetCurrentRecoil()` | Returns current accumulated recoil | Base recoil is set, current accumulates |
| `GetBarrelLocation()` | Returns barrel location for muzzle flash | Useful for effects |
| `GetMuzzleLocation()` | Alias for barrel location | Common term |

### Missing Setters

| Missing Function | Description | Notes |
|---|---|---|
| `SetFireMode(mode)` | Set fire mode (auto/semi/burst) | No fire mode control |
| `SetBurstCount(count)` | Number of shots per burst | No burst fire |
| `SetMaxSpread(max_spread)` | Limit maximum spread | Only base spread exists |
| `SetSpreadIncreasePerShot(increase)` | Control spread increase rate | No spread dynamics |
| `SetSpreadRecoveryRate(rate)` | Control spread recovery | No spread recovery |
| `SetRecoilIncreasePerShot(increase)` | Control recoil accumulation | No recoil dynamics |
| `SetRecoilRecoveryRate(rate)` | Control recoil recovery | No recoil recovery |
| `SetBulletPenetration(penetration)` | Enable/disable bullet penetration | `SetWallbangSettings` exists but no simple toggle |
| `SetTracerFrequency(frequency)` | How often tracers appear | No tracer control |
| `SetInfiniteAmmo(enabled)` | Disable ammo consumption | No infinite ammo toggle |
| `SetOneHandedDamageMultiplier(multiplier)` | Damage multiplier when one-handed | No handling-based damage |
| `SetHeadshotMultiplier(multiplier)` | Headshot damage multiplier | No headshot control |
| `SetFallOffDamage(distance, multiplier)` | Damage fall-off by distance | No distance-based damage |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `FireStart` | When firing begins (first shot) | Only `Fire` per shot |
| `FireEnd` | When firing ends | No end-of-fire event |
| `DryFire` | When weapon clicks empty | No dry fire event |
| `MagazineEmpty` | When magazine runs out | No magazine empty event |
| `AmmoChange` | When total ammo changes | `AmmoBagChange` exists |
| `ReloadStart` | When reload begins | Only `Reload` when complete |
| `ReloadEnd` | When reload finishes | `Reload` event exists |
| `AimStart` | When aiming begins | No event |
| `AimEnd` | When aiming ends | No event |
| `BulletImpact` | When bullet hits something | `BulletHit` exists on client only |
| `Overheat` | When weapon overheats (if applicable) | No overheat system |

---

## 13. Melee

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetDamageSettings()` | Returns damage start time and duration | Set by `SetDamageSettings`, no getter |
| `GetImpactSounds()` | Returns all impact sounds | Set by `SetImpactSound`, no getter |

### Missing Setters

| Missing Function | Description | Notes |
|---|---|---|
| `SetAttackRange(range)` | Set melee reach distance | No range control |
| `SetAttackSpeed(speed)` | Set attack speed multiplier | No speed control |
| `SetKnockbackForce(force)` | Set knockback on hit | No knockback |
| `SetBlockDamageMultiplier(multiplier)` | Damage multiplier when blocking | No block system |
| `SetStaggerDuration(duration)` | Stagger duration on hit | No stagger |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `UseStart` | When use/attack begins | Only `Attack` on hit |
| `UseEnd` | When use/attack ends | No end event |
| `Block` | When melee is blocked | No block event |
| `HitEntity` | When melee hits an entity with detailed info | `Attack` exists but no hit location/bone info |

---

## 14. Grenade

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetExplosionParticles()` | Returns explosion particle asset | No getter (constructor param) |
| `GetExplosionSound()` | Returns explosion sound asset | No getter (constructor param) |
| `IsArmed()` | Returns if grenade is armed (timer running) | No state query |
| `GetRemainingTime()` | Returns time until explosion | No countdown getter |
| `IsCooking()` | Returns if grenade is being cooked (timer started) | No state query |

### Missing Setters

| Missing Function | Description | Notes |
|---|---|---|
| `SetExplosionParticles(particle)` | Change explosion particle | No setter |
| `SetExplosionSound(sound)` | Change explosion sound | No setter |
| `SetCanCook(enabled)` | Allow/prevent cooking | No cooking control |
| `SetFuseLength(length)` | Alias for `SetTimeToExplode` | Convenience |
| `SetExplosionRadius(radius)` | Set combined explosion radius | Must use `SetDamage` with multiple params |
| `SetBounceForce(force)` | Control bounce behavior | No physics tuning |
| `SetMassOverride(mass)` | Override grenade mass | No mass control |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `Arm` | When grenade fuse starts | No arm event |
| `Bounce` | When grenade bounces off a surface | No bounce event |
| `PinPulled` | When pin is pulled (if applicable) | No pin event |
| `FuseEnd` | When fuse timer ends (just before explosion) | No pre-explosion event |

---

## 15. Light

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetSourceRadius()` | Returns source radius | Constructor param, no getter |
| `IsVisible()` | Returns visibility state | Constructor param `visible`, no getter |
| `GetMaxDrawDistance()` | Returns max draw distance | Constructor param, no getter |
| `GetUseInverseSquaredFalloff()` | Returns falloff mode | Constructor param, no getter |
| `GetConeAngle()` | Returns spot light cone angle | Constructor param, no getter |
| `GetInnerConeAngle()` | Returns inner cone angle | Constructor param, no getter |
| `GetLightProfile()` | Returns the light profile | Set by `SetTextureLightProfile`, no getter |
| `GetLightType()` | Returns the light type | Constructor param, no getter |

### Missing Setters

| Missing Function | Description | Notes |
|---|---|---|
| `SetSourceRadius(radius)` | Set source radius at runtime | No setter |
| `SetVisible(visible)` | Toggle visibility | No setter (only on Actor base) |
| `SetMaxDrawDistance(distance)` | Set max draw distance at runtime | No setter |
| `SetUseInverseSquaredFalloff(use)` | Toggle inverse squared falloff | No setter |
| `SetConeAngle(angle)` | Set spot light cone angle | No setter |
| `SetInnerConeAngle(angle)` | Set inner cone angle | No setter |
| `SetOuterConeAngle(angle)` | Set outer cone angle | No setter |
| `SetAttenuationFunction(function)` | Change attenuation function | No setter |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `TurnOn` | When light is turned on | No event |
| `TurnOff` | When light is turned off | No event |
| `Flicker` | Programmatic flicker effect | No built-in flicker |

---

## 16. Sound

### Missing Setters

| Missing Function | Description | Notes |
|---|---|---|
| `SetSoundAsset(asset)` | Change the sound being played | No way to change sound at runtime |
| `SetLooping(loop)` | Change loop mode at runtime | `SoundLoopMode` in constructor only |
| `SetAttenuationSettings(...)` | Change 3D sound attenuation | Constructor params only |
| `SetStereoPan(pan)` | Set stereo panning (-1 left, 1 right) | No pan control |
| `SetSubmixSend(submix, send)` | Route to submix | No submix routing |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `Finished` | When sound finishes playing | No event — must poll `IsPlaying` |
| `Looped` | When sound loops | No event |

---

## 17. Particle

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetAsset()` | Returns the particle asset path | No getter |
| `GetParameterFloat(name)` | Read back a float parameter | Set exists, no getter |
| `GetParameterVector(name)` | Read back a vector parameter | Set exists, no getter |
| `GetParameterColor(name)` | Read back a color parameter | Set exists, no getter |

### Missing Functions

| Missing Function | Description | Notes |
|---|---|---|
| `SetVisibility(visible)` | Toggle particle visibility | No visibility control |
| `SetLocalSpace(local_space)` | Toggle local/world space | No space control |
| `SetAutoDestroy(auto_destroy)` | Change auto-destroy at runtime | Constructor param only |
| `SetLODLevel(level)` | Set LOD level for performance | No LOD control |
| `SetMaxDistance(distance)` | Cull particles beyond distance | No distance culling |

---

## 18. Trigger

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetExtent()` | Returns the trigger extent/size | Set by `SetExtent`, no getter |
| `GetColor()` | Returns the trigger color | Set by `SetColor`, no getter |
| `GetOverlapOnlyClasses()` | Returns the class filter | Set by `SetOverlapOnlyClasses`, no getter |
| `GetOverlappingActors()` | Returns all currently overlapping actors | No way to query current overlaps |
| `IsOverlapping(actor)` | Returns if a specific actor is overlapping | No individual overlap check |

### Missing Functions

| Missing Function | Description | Notes |
|---|---|---|
| `SetIsVisible(visible)` | Toggle trigger visibility | Constructor param only |
| `SetTriggerType(type)` | Change between Sphere/Box at runtime | Constructor param only |

---

## 19. Cable

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetCableSettings()` | Returns cable settings (length, segments, etc.) | Set by `SetCableSettings`, no getter |
| `GetForces()` | Returns forces applied | Set by `SetForces`, no getter |
| `GetRenderingSettings()` | Returns rendering settings | Set by `SetRenderingSettings`, no getter |
| `GetAngularLimits()` | Returns angular limits | Set by `SetAngularLimits`, no getter |
| `GetLinearLimits()` | Returns linear limits | Set by `SetLinearLimits`, no getter |

---

## 20. StaticMesh

### Missing Functions

| Missing Function | Description | Notes |
|---|---|---|
| `SetVisibility(visible)` | Toggle visibility | No visibility control |
| `SetCastShadow(cast)` | Toggle shadow casting | No shadow control |
| `SetCollisionResponse(channel, response)` | Per-channel collision | No fine-grained collision |
| `SetSimulatePhysics(enabled)` | Enable physics | No physics toggle |
| `SetCastHiddenShadow(hidden)` | Cast shadow when hidden | No shadow control |
| `SetRenderCustomDepth(enabled)` | Custom depth for post-process | No post-process control |
| `SetCustomDepthStencilValue(value)` | Custom stencil value | No stencil control |
| `SetMinDrawDistance(distance)` | Minimum draw distance | No draw distance control |
| `SetMaxDrawDistance(distance)` | Maximum draw distance | No draw distance control |
| `SetTranslucentSortPriority(priority)` | Sort priority for translucency | No sort control |
| `SetLightmapResolution(resolution)` | Lightmap resolution override | No lightmap control |
| `SetLodBias(bias)` | LOD bias | No LOD control |
| `SetBoundsScale(scale)` | Bounds scale | No bounds control |

---

## 21. InstancedStaticMesh

### Missing Functions

| Missing Function | Description | Notes |
|---|---|---|
| `SetInstanceVisibility(index, visible)` | Toggle individual instance visibility | No per-instance visibility |
| `SetInstanceColor(index, color)` | Per-instance color tint | No per-instance color |
| `SetCastShadow(enabled)` | Toggle shadow casting | No shadow control |
| `SetMesh(mesh)` | Change the mesh asset at runtime | No mesh change |
| `GetInstances()` | Returns all instance transforms | No bulk getter |
| `SetInstancesTransforms(instances)` | Batch update transforms | Only per-instance update |
| `RemoveInstances(indices)` | Batch remove instances | Only per-instance remove |
| `FindInstanceByTransform(transform)` | Find instance at location | No search |

---

## 22. Billboard

### Missing Functions

| Missing Function | Description | Notes |
|---|---|---|
| `SetSize(size)` | Change billboard size | No setter (constructor only) |
| `GetSize()` | Returns billboard size | No getter |
| `SetMaterial(material)` | Change material | No setter |
| `SetSizeInScreenSpace(screen_space)` | Toggle screen/world space | Constructor param only |
| `SetVisibility(visible)` | Toggle visibility | No visibility control |
| `SetFacingCamera(facing)` | Change facing mode | No facing control |
| `SetRotation(rotation)` | Set billboard rotation | No rotation control |

---

## 23. Text3D

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetText()` | Returns current text | Set by `SetText`, no getter |
| `GetFont()` | Returns current font | Set by `SetFont`, no getter |
| `GetGlyphSettings()` | Returns glyph settings | Set by `SetGlyphSettings`, no getter |
| `GetTextSettings()` | Returns text settings | Set by `SetTextSettings`, no getter |
| `GetMaxSize()` | Returns max size | Set by `SetMaxSize`, no getter |

---

## 24. TextRender

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetAlignment()` | Returns alignment settings | Set by `SetAlignment`, no getter |
| `GetSpacingAdjust()` | Returns spacing | Set by `SetSpacingAdjust`, no getter |
| `GetFont()` | Returns font asset | Set by `SetFont`, no getter |
| `IsShadowCasting()` | Returns shadow casting state | Constructor param, no getter |

### Missing Setters

| Missing Function | Description | Notes |
|---|---|---|
| `SetWorldSize(size)` | Set text world size | `SetWordSize` exists but different from world size |
| `SetRenderType(type)` | Change rendering type at runtime | Constructor param only |
| `SetHorizontalSpacing(spacing)` | Set horizontal spacing individually | Only combined `SetSpacingAdjust` |
| `SetVerticalSpacing(spacing)` | Set vertical spacing individually | Only combined `SetSpacingAdjust` |
| `SetShadowEnabled(enabled)` | Toggle shadow | Constructor param only |
| `SetCastShadow(enabled)` | Toggle shadow casting | Constructor param only |

---

## 25. Decal

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetSize()` | Returns decal size | Set by `SetSize`, no getter |
| `GetMaterial()` | Returns decal material | Constructor param, no getter |
| `GetLifespan()` | Returns lifespan | Constructor param, no getter |
| `IsFadingIn()` | Returns if currently fading in | No state query |
| `IsFadingOut()` | Returns if currently fading out | No state query |

### Missing Functions

| Missing Function | Description | Notes |
|---|---|---|
| `SetMaterial(material)` | Change decal material at runtime | No runtime material change |
| `SetFadeScreenSize(size)` | Change fade screen size | Constructor param only |
| `SetLifespan(lifespan)` | Change remaining lifespan | No lifespan control |
| `SetVisibility(visible)` | Toggle visibility | No visibility control |

---

## 26. Canvas

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetClearColor()` | Returns clear color | Constructor param, no getter |
| `IsAutoResizing()` | Returns if auto-resize is on | Constructor param, no getter |
| `GetScreenPosition()` | Returns screen position | Set by `SetScreenPosition`, no getter |
| `IsVisible()` | Returns visibility | Set by `SetVisibility`, no getter |
| `GetAutoRepaintRate()` | Returns repaint rate | Set by `SetAutoRepaintRate`, no getter |

### Missing Functions

| Missing Function | Description | Notes |
|---|---|---|
| `DrawCircle(position, radius, color)` | Draw a circle | Only polygon with N sides |
| `DrawArc(position, radius, start_angle, end_angle, color)` | Draw an arc | No arc drawing |
| `DrawTriangle(points, color)` | Draw a triangle | No triangle drawing |
| `DrawGradientRect(position, size, color_top, color_bottom)` | Gradient rectangle | No gradient drawing |
| `DrawTextWithBackground(text, position, text_color, bg_color)` | Text with background | No text background |
| `MeasureText(text, font, size)` | Measure text dimensions | No text measurement |
| `SetClearColor(color)` | Change clear color | Constructor param only |
| `SetShouldClearBeforeUpdate(clear)` | Toggle clear before update | Constructor param only |

---

## 27. WebUI

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetURL()` | Returns current URL | No URL getter |
| `IsFocused()` | Returns if this WebUI has focus | No focus state query |
| `GetCurrentZoom()` | Returns current zoom level | No zoom getter |
| `GetLoadProgress()` | Returns page load progress | No progress query |

### Missing Setters

| Missing Function | Description | Notes |
|---|---|---|
| `SetZoom(zoom)` | Set zoom level | No zoom control |
| `SetBackgroundColor(color)` | Change background color | Constructor param only |
| `SetAutoResize(enabled)` | Toggle auto-resize | Constructor param only |
| `GoBack()` | Navigate back | No navigation |
| `GoForward()` | Navigate forward | No navigation |
| `Reload()` | Reload current page | No reload |
| `Stop()` | Stop loading | No stop loading |
| `SetAudioMuted(muted)` | Mute WebUI audio | No audio control |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `URLChange` | When URL changes | No navigation event |
| `LoadProgress` | During page loading | No progress event |
| `ConsoleMessage` | JavaScript console.log output | No console capture |
| `ScriptError` | JavaScript error occurred | No error event |
| `ContextMenu` | Right-click context menu | No context menu event |
| `Tooltip` | Tooltip shown | No tooltip event |

---

## 28. Widget

### Missing Functions

| Missing Function | Description | Notes |
|---|---|---|
| `RemoveChild(child)` | Remove a specific child widget | `AddChild` exists, no removal |
| `RemoveAllChildren()` | Remove all children | No bulk removal |
| `GetChildCount()` | Returns number of children | No child count |
| `GetChild(index)` | Returns child at index | No child access by index |
| `GetParent()` | Returns parent widget | No parent access |
| `SetPosition(position)` | Set widget position | No position control |
| `GetPosition()` | Get widget position | No position getter |
| `SetSize(size)` | Set widget size | No size control |
| `GetSize()` | Get widget size | No size getter |
| `SetRotation(rotation)` | Set widget rotation | No rotation control |
| `SetOpacity(opacity)` | Set widget opacity | No opacity control |
| `GetOpacity()` | Get widget opacity | No opacity getter |
| `SetToolTip(tool_tip)` | Set tooltip text | No tooltip |
| `SetCursor(cursor_type)` | Set cursor type when hovering | No cursor control |
| `SetIsEnabled(enabled)` | Enable/disable widget | No enabled state |
| `IsEnabled()` | Returns if widget is enabled | No enabled check |
| `SetRenderOpacity(opacity)` | Set render opacity | No opacity control |
| `SetRenderTransform(transform)` | Set render transform | No transform control |
| `SetPadding(padding)` | Set widget padding | No padding control |
| `SetForegroundColor(color)` | Set foreground color | No color control |
| `SetFont(font)` | Set font | No font control |
| `SetTextColor(color)` | Set text color (TextBlock) | No text color |
| `SetText(text)` | Set text (TextBlock) | No text setter |
| `GetText()` | Get text (TextBlock) | No text getter |
| `SetValue(value)` | Set value (Slider/ProgressBar) | No value setter |
| `GetValue()` | Get value (Slider/ProgressBar) | No value getter |
| `SetChecked(checked)` | Set checked state (CheckBox) | No checked setter |
| `GetChecked()` | Get checked state (CheckBox) | No checked getter |
| `SetColorAndOpacity(color)` | Set color and opacity | No color control |
| `SetBrushResource(image)` | Set image resource (Image) | No brush control |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `OnClicked` | When widget is clicked (Button) | No click event |
| `OnHovered` | When widget is hovered | No hover event |
| `OnUnhovered` | When widget is unhovered | No unhover event |
| `OnPressed` | When widget is pressed | No press event |
| `OnReleased` | When widget is released | No release event |
| `OnValueChanged` | When value changes (Slider/CheckBox) | No value change event |
| `OnTextChanged` | When text changes (EditableText) | No text change event |
| `OnTextCommitted` | When text is committed (EditableText) | No text commit event |
| `OnSelectionChanged` | When selection changes (ComboBox) | No selection event |
| `OnScroll` | When scroll position changes | No scroll event |

---

## 29. Widget3D

### Missing Functions

| Missing Function | Description | Notes |
|---|---|---|
| `SetWorldLocation(location)` | Set 3D position | No position control |
| `SetWorldRotation(rotation)` | Set 3D rotation | No rotation control |
| `SetWorldScale(scale)` | Set 3D scale | No scale control |
| `SetDrawSize(size)` | Set draw size | No size control |
| `SetPivot(pivot)` | Set pivot point | No pivot control |
| `SetSpace(space)` | Toggle World/Screen space | No space control |
| `SetVisibility(visible)` | Toggle visibility | No visibility control |
| `SetAttachedToActor(actor)` | Attach to actor | No attachment |
| `SetAttachedToSocket(actor, socket)` | Attach to actor socket | No socket attachment |

---

## 30. SceneCapture

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetFOVAngle()` | Returns FOV | Set by `SetFOVAngle`, no getter |
| `GetRenderRate()` | Returns render rate | Set by `SetRenderRate`, no getter |
| `IsFrozen()` | Returns if frozen | Set by `SetFreeze`, no getter |
| `GetSize()` | Returns texture size | No size getter |
| `IsDistanceOptimizationEnabled()` | Returns optimization state | Set by `SetDistanceOptimizationEnabled`, no getter |
| `GetViewDistance()` | Returns view distance | Constructor param, no getter |
| `GetRenderActors()` | Returns render-only actors list | No list getter |

---

## 31. Gizmo

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetTransformMode()` | Returns current transform mode | Set by `SetTransformMode`, no getter |
| `GetAlignSpace()` | Returns align space | Set by `SetAlignSpace`, no getter |
| `GetSnapSettings()` | Returns snap settings | Set by `SetSnapSettings`, no getter |
| `GetLocation()` | Returns current gizmo location | No position getter |
| `GetRotation()` | Returns current gizmo rotation | No rotation getter |
| `GetScale()` | Returns current gizmo scale | No scale getter |
| `GetTransform()` | Returns full transform | No transform getter |

### Missing Functions

| Missing Function | Description | Notes |
|---|---|---|
| `SetTargetActor(actor)` | Set which actor this gizmo controls | No target actor |
| `SetSnappingEnabled(enabled)` | Toggle snapping | No snap toggle |
| `SetAxisSnap(axis, enabled)` | Toggle snap per axis | No per-axis snap |
| `SetColor(color)` | Set gizmo color | No color control |
| `SetLineThickness(thickness)` | Set line thickness | No thickness control |
| `SetSphereRadius(radius)` | Set interaction sphere radius | No radius control |

---

## 32. Blueprint

### Missing Functions

| Missing Function | Description | Notes |
|---|---|---|
| `GetBlueprintAsset()` | Returns the blueprint asset path | No getter |
| `SetStaticMesh(mesh)` | Change the mesh if Blueprint has one | No mesh control |
| `SetMaterial(index, material)` | Set material on Blueprint | Already on Paintable |

---

## 33. Damageable (Base)

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetHealthPercent()` | Returns health as percentage (0-1) | Useful for HUDs |
| `GetLastDamageInfo()` | Returns info about the last damage taken | No damage history |
| `GetLastInstigator()` | Returns the player who last damaged this | No instigator tracking |
| `GetDamageMultipliers()` | Returns all bone damage multipliers | No bulk getter |
| `IsAlive()` | Alias for `!IsDead()` | Convenience |
| `GetRespawnLocation()` | Returns initial spawn location | No getter |
| `GetRespawnRotation()` | Returns initial spawn rotation | No getter |

### Missing Events

| Missing Event | Description | Notes |
|---|---|---|
| `Damaged` | More detailed damage event with hit bone, direction, etc. | `TakeDamage` exists but `Death` event includes more info |
| `Healed` | When entity is healed (health increased) | `HealthChange` exists but no specific heal event |
| `Revive` | When entity is revived from death | No revive event |

---

## 34. Paintable (Base)

### Missing Getters

| Missing Function | Description | Notes |
|---|---|---|
| `GetMaterial(index, attachable_id)` | Returns the material at an index | No getter for applied material |
| `GetPhysicalMaterial()` | Returns the physical material | Set by `SetPhysicalMaterial`, no getter |
| `GetMaterialCount()` | Returns number of material slots | No material count |
| `GetAllMaterialParameters(index)` | Returns all parameters of a material | No bulk parameter getter |

---

## 35. Structs

### Vector

| Missing Function | Description | Notes |
|---|---|---|
| `Lerp(other, alpha)` | Linear interpolation | Fundamental for animations/movement |
| `Slerp(other, alpha)` | Spherical interpolation | For direction interpolation |
| `ClampSize(max)` | Clamp vector magnitude | Common utility |
| `ClampSizeMin(min)` | Clamp minimum magnitude | Common utility |
| `GetClampedSize(max)` | Returns clamped copy | Non-mutating version |
| `ProjectOnTo(other)` | Project vector onto another | Common for physics |
| `MirrorOnTo(normal)` | Mirror vector on a plane | Common for reflections |
| `RotateBy(quat)` | Rotate by quaternion | Common for orientation |
| `AngleBetween(other)` | Angle between two vectors (degrees) | Common utility |
| `ToVector2D()` | Convert to 2D (drop Z) | Common conversion |
| `ToString()` | String representation | Debugging |
| `Min_components()` | Returns component-wise minimum | Utility |
| `Max_components()` | Returns component-wise maximum | Utility |
| `ComponentMin(other)` | Component-wise min | Utility |
| `ComponentMax(other)` | Component-wise max | Utility |
| `ComponentClamp(min, max)` | Component-wise clamp | Utility |
| `ComponentAbs()` | Absolute value of each component | Utility |

### Rotator

| Missing Function | Description | Notes |
|---|---|---|
| `Lerp(other, alpha)` | Linear interpolation | Fundamental for smooth rotation |
| `Slerp(other, alpha)` | Spherical interpolation | Better rotation interpolation |
| `Clamp()` | Clamp to [-180, 180] | Common utility |
| `GetClamped()` | Returns clamped copy | Non-mutating version |
| `Add(other)` | Add two rotators | Operator exists but explicit method |
| `Subtract(other)` | Subtract two rotators | Operator exists but explicit method |
| `Multiply(scalar)` | Scale rotator | Operator exists but explicit method |
| `Equals(other, tolerance)` | Equality check with tolerance | Utility |
| `GetAngle()` | Get total angle magnitude | Utility |
| `GetAxis()` | Get rotation axis | Utility |
| `ContainsNaN()` | Check for NaN components | Debugging |
| `ToString()` | String representation | Debugging |
| `ToEulerDegrees()` | Convert to Euler angles (degrees) | Convenience |
| `ToEulerRadians()` | Convert to Euler angles (radians) | Convenience |
| `GetForwardVector()` already exists | — | Good |
| `GetRightVector()` already exists | — | Good |
| `GetUpVector()` already exists | — | Good |

### Color

| Missing Function | Description | Notes |
|---|---|---|
| `Lerp(other, alpha)` | Linear interpolation | Fundamental for color transitions |
| `ToHex()` | Convert to hex string | Common utility |
| `FromHex(hex)` | Static: create from hex string | Common utility |
| `ToVector()` | Convert to Vector (RGB) | Utility |
| `ToRotator()` | Convert to Rotator (HSL-like) | Utility |
| `WithAlpha(alpha)` | Returns copy with different alpha | Common utility |
| `WithR(r)` | Returns copy with different R | Utility |
| `WithG(g)` | Returns copy with different G | Utility |
| `WithB(b)` | Returns copy with different B | Utility |
| `Desaturate(amount)` | Desaturate color | Visual effects |
| `GetLuminance()` | Get perceived luminance | UI accessibility |
| `ToLinear()` | Convert from sRGB to linear | Rendering |
| `ToSRGB()` | Convert from linear to sRGB | Rendering |
| `ToString()` | String representation | Debugging |
| `IsEqual(other, tolerance)` | Approximate equality | Utility |

### Vector2D

| Missing Function | Description | Notes |
|---|---|---|
| `Lerp(other, alpha)` | Linear interpolation | Common utility |
| `ClampSize(max)` | Clamp magnitude | Common utility |
| `GetClampedSize(max)` | Returns clamped copy | Non-mutating |
| `Distance(other)` | Distance between points | Common utility |
| `DistanceSquared(other)` | Squared distance | Optimization |
| `Dot(other)` | Dot product | Common utility |
| `Cross(other)` | 2D cross product (scalar) | Common utility |
| `Normalize()` | Normalize in-place | Common utility |
| `GetSafeNormal()` | Returns normalized copy | Common utility |
| `Size()` | Returns magnitude | Common utility |
| `SizeSquared()` | Returns squared magnitude | Optimization |
| `IsNearlyZero(tolerance)` | Check if near zero | Common utility |
| `IsZero()` | Check if exactly zero | Common utility |
| `RotateBy(angle)` | Rotate by angle (degrees) | Common utility |
| `ToVector()` | Convert to 3D Vector (Z=0) | Common conversion |
| `ToString()` | String representation | Debugging |
| `ClampMin(min)` | Clamp minimum per component | Utility |
| `ClampMax(max)` | Clamp maximum per component | Utility |
| `ComponentMin(other)` | Component-wise min | Utility |
| `ComponentMax(other)` | Component-wise max | Utility |

### Matrix

| Missing Function | Description | Notes |
|---|---|---|
| `Multiply(other)` | Matrix multiplication | Core operation |
| `TransformVector(vector)` | Transform a vector by matrix | Core operation |
| `TransformPosition(position)` | Transform a position by matrix | Core operation |
| `Inverse()` | Returns inverse matrix | Core operation |
| `Determinant()` | Returns determinant | Core operation |
| `Transpose()` | Returns transpose | Core operation |
| `GetTranslation()` | Extract translation | Core operation |
| `GetRotation()` | Extract rotation as Rotator | Core operation |
| `GetScale()` | Extract scale | Core operation |
| `SetTranslation(vector)` | Set translation component | Core operation |
| `ToString()` | String representation | Debugging |
| `Identity()` | Static: identity matrix | Core operation |
| `FromRotatorAndPosition(rotator, position)` | Create from transform | Core operation |
| `ToTransform()` | Convert to Location+Rotation+Scale | Core operation |

### Quat

| Missing Function | Description | Notes |
|---|---|---|
| `Multiply(other)` | Quaternion multiplication | Core operation |
| `Inverse()` | Returns inverse quaternion | Core operation |
| `Normalize()` | Normalize quaternion | Core operation |
| `GetNormalized()` | Returns normalized copy | Core operation |
| `Dot(other)` | Dot product | Core operation |
| `Size()` | Returns magnitude | Core operation |
| `SizeSquared()` | Returns squared magnitude | Core operation |
| `RotateVector(vector)` | Rotate vector by quaternion | Core operation |
| `UnrotateVector(vector)` | Inverse rotate vector | Core operation |
| `ToRotator()` | Convert to Rotator | Core conversion |
| `FromRotator(rotator)` | Static: create from Rotator | Core conversion |
| `Slerp(other, alpha)` | Spherical interpolation | Core animation |
| `Log()` | Logarithm | Core math |
| `Exp()` | Exponent | Core math |
| `GetAxisX()` | Get X axis | Core utility |
| `GetAxisY()` | Get Y axis | Core utility |
| `GetAxisZ()` | Get Z axis | Core utility |
| `GetForwardVector()` | Get forward direction | Core utility |
| `ToString()` | String representation | Debugging |
| `IsNormalized()` | Check if normalized | Debugging |

---

## 36. Static Classes

### Server

| Missing Function | Description | Notes |
|---|---|---|
| `GetGameMode()` | Returns the current game mode name | No game mode query |
| `GetAllPlayers()` | Returns all connected players | Must use `Player.GetAll()` |
| `GetPlayerBySteamID(steam_id)` | Returns player by Steam ID | Exists on Player class, not Server |
| `GetPlayerByAccountID(account_id)` | Returns player by Account ID | No equivalent on Server |
| `SetTickRate(tick_rate)` | Change server tick rate at runtime | No runtime tick rate change |
| `SetLogLevel(level)` | Change log level at runtime | No setter |
| `BanIP(ip, reason)` | Ban by IP address | Only account-based ban |
| `UnbanIP(ip)` | Unban by IP | Only account-based unban |
| `IsIPBanned(ip)` | Check if IP is banned | No ban check |
| `GetBannedPlayers()` | Returns list of banned players | No ban list |
| `SendHTTPRequest(url, method, headers, body, callback)` | HTTP request from server | `HTTP` class exists but not on Server |
| `GetPlayerCount()` | Returns connected player count | `GetConnectionCount` exists |
| `GetPlayerByIndex(index)` | Get player by connection index | No index-based access |

### Client

| Missing Function | Description | Notes |
|---|---|---|
| `GetScreenWidth()` | Returns screen width | `GetSize` not available |
| `GetScreenHeight()` | Returns screen height | No screen dimensions |
| `GetMousePosition()` | Returns mouse position | No mouse position query |
| `SetMousePosition(x, y)` | Set mouse position | No mouse position set |
| `IsMouseVisible()` | Returns if mouse is visible | No mouse visibility query |
| `SetMouseVisible(visible)` | Toggle mouse visibility | No mouse visibility set |
| `GetClipboardText()` | Read clipboard | `CopyToClipboard` exists but no read |
| `PasteFromClipboard()` | Paste from clipboard | No paste |
| `TakeScreenshot(format)` | Take a screenshot | No screenshot function |
| `SetFullscreen(mode)` | Set fullscreen mode | No fullscreen control |
| `IsFullscreen()` | Returns fullscreen state | No fullscreen query |
| `SetVSync(enabled)` | Toggle VSync | No VSync control |
| `SetFPSLimit(limit)` | Set FPS limit | No FPS limit |
| `GetFPS()` | Returns current FPS | No FPS getter |
| `GetGPUName()` | Returns GPU name | No GPU info |
| `GetCPUName()` | Returns CPU name | No CPU info |
| `GetMemoryUsage()` | Returns memory usage | No memory info |

### Events

| Missing Function | Description | Notes |
|---|---|---|
| `SubscribeOnce(event_name, callback)` | Subscribe to event only once | Must manually unsubscribe |
| `UnsubscribeAll()` | Unsubscribe from all events in package | Must call per event |

### Timer

| Missing Function | Description | Notes |
|---|---|---|
| `SetInterval(callback, milliseconds)` | Create repeating timer | Only `SetTimeout` exists |
| `ClearInterval(id)` | Clear a repeating timer | Must use `ClearTimeout` |
| `SetImmediate(callback)` | Execute on next tick | Must use `SetTimeout(0)` |
| `PauseTimer(id)` | Pause a timer | No pause |
| `ResumeTimer(id)` | Resume a paused timer | No resume |
| `GetTimerTimeLeft(id)` | Get remaining time | No time-left query |
| `IsTimerActive(id)` | Check if timer is active | No active check |

### HTTP

| Missing Function | Description | Notes |
|---|---|---|
| `DownloadFile(url, file_path, callback)` | Download file to disk | No file download |
| `UploadFile(url, file_path, method, callback)` | Upload file from disk | No file upload |
| `SetTimeout(timeout)` | Set request timeout | No timeout config |
| `SetHeader(name, value)` | Set default headers | No header config |

### Trace

| Missing Function | Description | Notes |
|---|---|---|
| `SweepSphere(location, radius, direction, distance, options)` | Sphere sweep | Only line traces |
| `SweepBox(location, extent, rotation, direction, distance, options)` | Box sweep | Only line traces |
| `SweepCapsule(location, radius, half_height, direction, distance, options)` | Capsule sweep | Only line traces |
| `OverlapSphere(location, radius, options)` | Sphere overlap test | No overlap tests |
| `OverlapBox(location, extent, rotation, options)` | Box overlap test | No overlap tests |
| `GetPhysicalMaterialAtLocation(location)` | Get surface type at location | No surface query |

### Assets

| Missing Function | Description | Notes |
|---|---|---|
| `GetAssetInfo(asset_path)` | Returns metadata about an asset | No asset metadata |
| `GetAssetDependencies(asset_path)` | Returns asset dependencies | No dependency query |
| `IsAssetLoaded(asset_path)` | Check if asset is loaded | No load status check |
| `LoadAssetAsync(asset_path, callback)` | Async asset loading | No async loading |
| `GetMeshBounds(asset_path)` | Returns mesh bounds | No bounds query |
| `GetMeshVertexCount(asset_path)` | Returns vertex count | No mesh info |
| `GetMaterialProperties(asset_path)` | Returns material properties | No material info |

### Input

| Missing Function | Description | Notes |
|---|---|---|
| `UnbindAction(action_name)` | Unbind a previously bound action | No unbind |
| `GetKeyState(key)` | Returns if a key is currently pressed | No key state query |
| `GetMouseState()` | Returns mouse position and buttons | No mouse state |
| `IsActionPressed(action_name)` | Check if action is currently pressed | No action state query |
| `IsActionJustPressed(action_name)` | Check if action was just pressed | No action state query |
| `IsActionJustReleased(action_name)` | Check if action was just released | No action state query |
| `GetAxisValue(axis_name)` | Returns current axis value | No axis value query |
| `SetGamepadConnected(connected)` | Simulate gamepad connection | No gamepad simulation |

### Navigation

| Missing Function | Description | Notes |
|---|---|---|
| `FindPathSync(start, end, options)` | Synchronous pathfinding | Only async with callback |
| `ProjectPointToNavMesh(location, query_extent)` | Project point to navmesh | No projection |
| `GetRandomPointInNavMesh(center, radius)` | Get random navigable point | No random point |
| `GetRandomReachablePointInRadius(location, radius)` | Get random reachable point | No random reachable point |
| `GetNavMeshBounds()` | Returns navmesh bounds | No bounds query |
| `IsPointOnNavMesh(location)` | Check if point is navigable | No point-on-mesh check |
| `GetNavMeshHeight(location)` | Get navigation height at location | No height query |

### Chat

| Missing Function | Description | Notes |
|---|---|---|
| `SendMessage(player, message)` | Send message to specific player | No targeted message |
| `SendAnnouncement(message, type)` | Send server announcement | No announcement |
| `ClearChat()` | Clear chat for a player | No clear |
| `GetChatHistory()` | Returns recent chat messages | No history |
| `SetChatEnabled(enabled)` | Enable/disable chat | No chat toggle |

### Level

| Missing Function | Description | Notes |
|---|---|---|
| `GetLevelInfo()` | Returns level metadata | No level info |
| `GetLevelBounds()` | Returns level bounds | No bounds query |
| `GetSpawnPoints()` | Returns all spawn points in level | No spawn point query |
| `GetLevelTimeOfDay()` | Returns current time of day | No time query |
| `SetLevelTimeOfDay(time)` | Set time of day | No time set |
| `GetWeather()` | Returns current weather | No weather query |
| `SetWeather(weather)` | Set weather | No weather set |
| `GetFogSettings()` | Returns fog configuration | No fog query |
| `SetFogSettings(settings)` | Set fog configuration | No fog set |
| `GetPostProcessSettings()` | Returns post-process settings | No post-process query |

### Debug

| Missing Function | Description | Notes |
|---|---|---|
| `DrawSphere(location, radius, color, duration)` | Draw debug sphere | No 3D debug draw |
| `DrawBox(location, extent, rotation, color, duration)` | Draw debug box | No 3D debug draw |
| `DrawCapsule(location, radius, half_height, rotation, color, duration)` | Draw debug capsule | No 3D debug draw |
| `DrawString(location, text, color, duration)` | Draw debug string in 3D | No 3D debug text |
| `FlushDebug()` | Clear all debug drawings | No flush |
| `GetMemoryStats()` | Returns memory statistics | No memory stats |
| `GetPerformanceStats()` | Returns performance stats | No performance stats |
| `SetProfilingEnabled(enabled)` | Toggle profiling | No profiling toggle |

### Steam

| Missing Function | Description | Notes |
|---|---|---|
| `GetPlayerAvatar(steam_id, size)` | Get player avatar as texture | No avatar query |
| `GetPlayerName(steam_id)` | Get player's Steam name | No name query |
| `GetPlayerCount()` | Get current player count on server | No count query |
| `IsOverlayEnabled()` | Check if Steam overlay is enabled | No overlay check |
| `OpenOverlay(url)` | Open Steam overlay to URL | No overlay control |
| `GetBuildID()` | Get current build ID | No build info |
| `IsDLCInstalled(app_id)` | Check if DLC is installed | No DLC check |
| `GetNumOwnedApps()` | Get number of owned apps | No ownership query |

### Discord

| Missing Function | Description | Notes |
|---|---|---|
| `GetPartySize()` | Returns current party size | No party query |
| `GetPartyMax()` | Returns max party size | No party query |
| `IsDiscordEnabled()` | Returns if Discord integration is enabled | No state query |

### PostProcess

| Missing Function | Description | Notes |
|---|---|---|
| `GetSettings()` | Returns all post-process settings | No settings getter |
| `SetVignetteIntensity(intensity)` | Set vignette intensity | No vignette control |
| `SetChromaticAberration(intensity)` | Set chromatic aberration | No aberration control |
| `SetGrainIntensity(intensity)` | Set film grain intensity | No grain control |
| `SetMotionBlur(amount)` | Set motion blur | No motion blur control |
| `SetAmbientOcclusion(amount)` | Set AO | No AO control |
| `SetBloomIntensity(intensity)` | Set bloom | No bloom control |
| `SetDOF(focal_distance, aperture)` | Set depth of field | No DOF control |
| `SetColorTemperature(temperature)` | Set color temperature | No temperature control |
| `SetAutoExposure(enabled)` | Toggle auto exposure | No exposure control |

### Sky

| Missing Function | Description | Notes |
|---|---|---|
| `GetTimeOfDay()` | Returns current time of day | No time query |
| `SetTimeOfDay(time)` | Set time of day | No time set |
| `GetSunDirection()` | Returns sun direction vector | No sun direction |
| `GetMoonDirection()` | Returns moon direction vector | No moon direction |
| `SetSunDirection(direction)` | Set sun direction | No sun control |
| `SetSunBrightness(brightness)` | Set sun brightness | No brightness control |
| `SetMoonBrightness(brightness)` | Set moon brightness | No brightness control |
| `GetCloudDensity()` | Returns cloud density | No cloud query |
| `SetCloudDensity(density)` | Set cloud density | No cloud control |
| `GetStarIntensity()` | Returns star intensity | No star query |
| `SetStarIntensity(intensity)` | Set star intensity | No star control |
| `GetCloudSpeed()` | Returns cloud movement speed | No cloud speed query |
| `SetCloudSpeed(speed)` | Set cloud movement speed | No cloud speed control |
| `GetWindSpeed()` | Returns wind speed | No wind query |
| `SetWindSpeed(speed)` | Set wind speed | No wind control |
| `GetWindDirection()` | Returns wind direction | No wind direction query |
| `SetWindDirection(direction)` | Set wind direction | No wind direction control |

---

## 37. Enums

### Missing Enums

| Enum Name | Description | Notes |
|---|---|---|
| `VehicleFireMode` | Fire modes: Auto, Semi, Burst | Weapon has no fire mode concept |
| `MovementMode` | Walking, Flying, Swimming, Falling, etc. | CharacterSimple uses raw integers |
| `InputMode` | Game, UI, GameAndUI | Player has no input mode concept |
| `FullscreenMode` | Windowed, Fullscreen, Borderless | Client has no fullscreen control |
| `ChatMessageType` | Say, Team, System, Announcement | Chat has no message types |
| `WeatherPreset` | Clear, Cloudy, Rain, Snow, etc. | Level has no weather system |

### Missing Enum Values

| Enum | Missing Value | Notes |
|---|---|---|
| `CollisionChannel` | `Visibility` | UE5 visibility channel |
| `CollisionChannel` | `Camera` | UE5 camera channel |
| `CollisionChannel` | `Destructible` | UE5 destructible channel |
| `CollisionChannel` | `VehicleBreakable` | Chaos vehicle breakable |
| `CollisionChannel` | `PawnMesh` | Pawn mesh specific channel |
| `CollisionType` | `QueryOnly` | Only query, no physical collision |
| `CollisionType` | `PhysicsOnly` | Only physics, no query |
| `LogType` | `Verbose2` | Additional verbosity level |

---

## Summary of Priority Gaps

### Critical (Core Gameplay)

1. **VehicleWheeled missing speed getter** — No `GetSpeed()` despite having RPM/Gear getters
2. **VehicleWheeled missing events** — No `GearChange`, `EngineStart/Stop`, `Brake` events
3. **VehicleWater extremely minimal** — Only 2 functions, no speed/events
4. **Prop missing physics setters** — No direct velocity set, no physics toggle
5. **Weapon missing fire mode** — No fire mode concept (auto/semi/burst)
6. **Timer missing SetInterval** — Only `SetTimeout` exists, no repeating timer
7. **Structs missing Lerp/Slerp** — Fundamental for all game math

### High (Quality of Life)

8. **VehicleWheeled missing all getters** — 14+ setters with no corresponding getters
9. **Light missing getters for constructor params** — 8 params with no getters
10. **Player missing mouse/input control** — No sensitivity, cursor lock, input mode
11. **Input missing key state queries** — No way to check if key is pressed
12. **Character missing convenience getters** — Death/pain sound, FOV, ragdoll settings

### Medium (Completeness)

13. **Widget missing value getters/setters** — No text/value/color/opacity control
14. **WebUI missing navigation** — No back/forward/reload
15. **StaticMesh missing physics/rendering control** — Very minimal API
16. **Billboard missing all runtime control** — Constructor-only API
17. **Decal missing runtime material change** — No material setter

### Low (Nice to Have)

18. **Debug missing 3D drawing** — No sphere/box/capsule drawing
19. **Sky missing sun/moon control** — Limited sky manipulation
20. **PostProcess missing per-effect control** — No bloom/DOF/grain control
