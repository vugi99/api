# New Classes Proposal

Proposals for new classes to expand the nanos world API. Each entry covers class name, description, purpose, and parent classes. No detailed function signatures — just the concept.

---

## 1. Projectile

**Description:** A fast-moving entity representing a bullet, arrow, rocket, or any physics-driven projectile with trail effects, lifetime, and impact behavior.

**Purpose:** The current weapon system fires hitscan bullets with no visible projectile. Many sandbox games need visible projectiles for gameplay variety — arrows, thrown objects, rocket launchers, grenade launchers, slow-moving energy balls, etc. A Projectile class gives modders a first-class entity for this.

**Parent Classes:** Entity → Actor → Paintable

**Key Concepts:**
- Configurable velocity, gravity, drag, lifetime
- Trail particle attachment
- On-hit damage with configurable falloff
- Bounce/ricochet behavior
- Explosion on impact option
- Owner/instigator tracking

---

## 2. DamageVolume

**Description:** An invisible (or visible) volume that applies damage to any Damageable entity that enters or stays inside it.

**Purpose:** Essential for sandbox environments — lava pools, poison gas, electric fences, radiation zones, fire areas, water drowning zones. Currently modders must use Triggers + custom Lua timers to replicate this, which is inefficient and inconsistent.

**Parent Classes:** Entity → Actor

**Key Concepts:**
- Configurable shape (box/sphere/capsule)
- Damage per second, damage type
- Overlap filter (class whitelist)
- Begin/End overlap events
- Optional visual effects (particles, decal)
- Can be toggled on/off at runtime

---

## 3. SpawnPoint

**Description:** A configurable spawn location entity that manages where players and entities respawn, with team-based and random spawn support.

**Purpose:** Currently spawn points are only defined in map TOML files as static data. Runtime spawn points are essential for game modes — king of the hill, team deathmatch, capture the flag, round-based games. Modders need to create, destroy, and query spawn points dynamically.

**Parent Classes:** Entity → Actor

**Key Concepts:**
- Location and rotation
- Team assignment
- Priority/weight for random selection
- Enable/disable at runtime
- Spawn delay/cooldown
- Maximum occupancy
- Associated effects (spawn particles, sounds)
- Events: Spawn, Occupied, Freed

---

## 4. AreaEffect

**Description:** A persistent visual/audio effect entity that exists in the world at a location — smoke clouds, fire patches, gas clouds, healing auras, shield bubbles, etc.

**Purpose:** Distinct from Particle (which is typically one-shot or looping emitter). AreaEffect is a gameplay-meaningful persistent zone with visual representation, optional gameplay effects, and lifecycle management. Think of it as a Trigger + Particle combined with a meaningful gameplay purpose.

**Parent Classes:** Entity → Actor

**Key Concepts:**
- Attached particle/sound assets
- Radius of effect
- Duration with fade-in/fade-out
- Optional overlap gameplay effects (slow, damage, heal, buff)
- Stacking behavior (can multiple overlap the same target?)
- Visual scale animation
- Events: Begin, End, Tick

---

## 5. Constraint

**Description:** A generic physics constraint entity that can connect any two actors with configurable linear/angular limits, drives, and motors — more flexible than Cable.

**Purpose:** Cable combines physics constraint with visual rope rendering. Many gameplay scenarios need pure physics constraints without visuals — hinges for doors, sliders for elevators, springs for traps, ragdoll bone constraints, vehicle attachment points, ragdoll-to-skeleton blending, etc.

**Parent Classes:** Entity → Actor

**Key Concepts:**
- Attach Actor A to Actor B (with bone support)
- Linear limits (X/Y/Z motion: Free/Limited/Locked)
- Angular limits (Swing1, Swing2, Twist)
- Linear/angular drives (position + velocity targets)
- Motor settings (strength, target)
- Break threshold (force/torque that breaks the constraint)
- Events: Broken, ConstraintBroken

---

## 6. Spline

**Description:** A path entity defined by a series of points with interpolation, useful for defining roads, patrol routes, camera paths, projectile trajectories, and animated movement paths.

**Purpose:** No path system exists in the API. Spline paths are fundamental for AI patrol routes, animated camera sequences, road networks, race tracks, projectile arc visualization, particle emission paths, and many other sandbox use cases.

**Parent Classes:** Entity → Actor

**Key Concepts:**
- Array of control points with tangents
- Linear/Catmull-Rom/Bezier interpolation
- Get position/rotation/tangent at distance or percentage
- Total length calculation
- Closest point on spline query
- Visual debugging (draw spline in world)
- Optional mesh along spline (road, fence)
- Events: PointAdded, PointMoved, Recalculated

---

## 7. TriggerVolume

**Description:** An enhanced version of Trigger that supports complex shapes, multiple overlapping zones, layer-based filtering, and persistent overlap tracking.

**Purpose:** Trigger is functional but limited for complex sandbox scenarios. Games need layered triggers (entry zones, safe zones, capture zones), multi-shape support, and overlap state tracking without polling.

**Parent Classes:** Entity → Actor

**Key Concepts:**
- Multiple shape support (box, sphere, capsule, convex)
- Layer/priority system for overlapping volumes
- Persistent overlap list (get all currently overlapping actors)
- Enter/Exit/Stay events with tick rate control
- Volume group tagging
- Visual mode for debugging
- Query: IsActorInside(actor)
- Query: GetOverlappingActors()

---

## 8. NavModifierVolume

**Description:** A volume that modifies the navigation mesh at runtime — creating obstacles, modifying walkability, blocking paths, or carving areas in the navmesh.

**Purpose:** Currently the Navigation static class provides pathfinding but no way to modify the navmesh at runtime. Dynamic obstacles, closing/opening doors that affect AI paths, temporary barricades, and player-built structures that block AI all need navmesh modification.

**Parent Classes:** Entity → Actor

**Key Concepts:**
- Box/Sphere/Capsule volume shape
- Modifiers: Block, Walkable, Navlink, Null
- Runtime enable/disable
- Area cost modification
- Temporary vs permanent modification
- Events: NavMeshUpdated

---

## 9. Interactable

**Description:** A base concept class for any object that can be interacted with by characters — doors, switches, levers, chests, buttons, computers, etc. Provides standardized interaction input, UI prompts, and state management.

**Purpose:** Currently Prop and Pickable handle interaction ad-hoc. Many sandbox objects need a standardized interaction framework — open/close doors, activate switches, pick up items, use computers, read signs. This provides the common interaction contract.

**Parent Classes:** Entity → Actor → Paintable

**Key Concepts:**
- Interaction prompt text and icon
- Interaction range
- Cooldown between interactions
- State machine (Closed/Open, Off/On, Locked/Unlocked)
- Require line-of-sight for interaction
- Multi-use vs single-use
- Events: InteractStart, InteractEnd, StateChange, Locked, Unlocked

---

## 10. ProjectilePool

**Description:** A server-side utility entity that manages a pool of pre-spawned Projectile entities for efficient bullet/projectile reuse, avoiding constant spawn/destroy overhead.

**Purpose:** For games with high fire rates (automatic weapons, turrets, artillery), spawning and destroying projectiles every frame is expensive. Object pooling reuses projectile entities, dramatically improving performance.

**Parent Classes:** Entity

**Key Concepts:**
- Pre-warm pool size
- Get/release projectile from pool
- Auto-recycle after lifetime expires
- Pool statistics (active, available, total)
- Overflow behavior (spawn new or ignore)
- Per-pool configuration (mesh, speed, damage)

---

## 11. Radar

**Description:** A client-side entity that renders a minimap/radar HUD element showing nearby entities, with configurable filters, ranges, and visual styles.

**Purpose:** Minimap/radar is essential for many sandbox game modes — battle royale, open world, survival, team games. Currently modders must build this entirely from scratch using Canvas + Tick events. A Radar class provides the core rendering with modders only needing to provide data.

**Parent Classes:** Entity

**Key Concepts:**
- Radar shape (circle, rectangle)
- Range/distance configuration
- Entity filter (show only specific classes/teams)
- Blip styling per entity type
- FOV cone display
- Ping/marker system
- Smooth rotation with player
- Zoom levels
- Events: BlipAdded, BlipRemoved, PingRequested

---

## 12. Inventory

**Description:** A server-side entity that represents a container/inventory system for managing items — backpacks, chests, vehicle trunks, weapon slots, etc.

**Purpose:** Inventory is a core sandbox mechanic (survival, RPG, looter shooters) with no built-in support. Currently modders implement this entirely in Lua tables. An Inventory class provides slot-based storage with networking, serialization, and events.

**Parent Classes:** Entity

**Key Concepts:**
- Configurable slot count and slot types
- Item stack management (count, max stack)
- Slot categories (weapon, ammo, consumable, misc)
- Transfer between inventories
- Weight/capacity limits
- Drop all items
- Serialize to/from table (for database storage)
- Events: ItemAdded, ItemRemoved, ItemMoved, SlotChanged, InventoryFull, InventoryEmpty

---

## 13. Team

**Description:** A server-side entity that represents a team in the game — managing members, scores, colors, and team-based gameplay logic.

**Purpose:** Many sandbox game modes are team-based but there's no first-class Team entity. Currently modders use raw integer team IDs on Characters. A Team class provides richer team management with scores, rosters, colors, and events.

**Parent Classes:** Entity

**Key Concepts:**
- Team name, color, logo
- Member list (add/remove players)
- Team score
- Max members (optional)
- Ally/enemy team relationships
- Team chat channel
- Respawn logic (team spawn points)
- Events: MemberAdded, MemberRemoved, ScoreChanged, TeamWon

---

## 14. GameMode

**Description:** A base entity class for defining game mode logic — round management, scoring rules, win conditions, player state management, and match lifecycle.

**Purpose:** Game modes are currently implemented as loose Lua scripts with no standardized structure. A GameMode class provides the common contract for round-based, timed, or free-roam modes with built-in networking and lifecycle events.

**Parent Classes:** Entity

**Key Concepts:**
- Match state machine (Waiting, Starting, InProgress, RoundEnd, MatchEnd)
- Round timer
- Score limit
- Player ready system
- Team assignment logic
- Spectator management
- Pause/resume
- Events: MatchStart, MatchEnd, RoundStart, RoundEnd, PlayerScored, PlayerDied, StateChanged

---

## 15. Fog

**Description:** A client-side entity that creates a localized fog volume in the world — for atmospheric effects, smoke screens, underwater haze, magical auras, etc.

**Purpose:** Global fog is controlled through Sky settings, but localized fog volumes are needed for gameplay — smoke grenades, gas zones, magical effects, underwater areas, cave entrances, atmospheric storytelling.

**Parent Classes:** Entity → Actor

**Key Concepts:**
- Box/Sphere/Capsule volume shape
- Fog density, color, falloff
- Priority for overlapping fog volumes
- Fade in/out
- Wind influence
- Optional particle attachment
- Events: FogEnter, FogExit (on overlapping actors)

---

## 16. PhysicsField

**Description:** A client-side entity that creates a localized physics force field — attracting, repelling, or applying custom forces to nearby physics objects within a radius.

**Purpose:** Force fields are common in sandbox games — gravity guns, black holes, repulsor fields, wind tunnels, anti-gravity zones, tractor beams. Currently modders must manually apply forces via Tick events, which is inefficient and lacks proper networking.

**Parent Classes:** Entity → Actor

**Key Concepts:**
- Force type: attract, repel, custom direction
- Force strength and falloff curve
- Radius of effect
- Affected entity types (filter)
- Rotation of force field
- Events: ActorEntered, ActorExited, ForceApplied

---

## Summary Table

| # | Class Name | Purpose | Parent |
|---|---|---|---|
| 1 | `Projectile` | Visible physics-driven projectile | Entity → Actor → Paintable |
| 2 | `DamageVolume` | Volume that applies damage on overlap | Entity → Actor |
| 3 | `SpawnPoint` | Runtime spawn location management | Entity → Actor |
| 4 | `AreaEffect` | Persistent gameplay zone with visuals | Entity → Actor |
| 5 | `Constraint` | Generic physics constraint (no visuals) | Entity → Actor |
| 6 | `Spline` | Path entity with interpolation | Entity → Actor |
| 7 | `TriggerVolume` | Enhanced trigger with complex shapes | Entity → Actor |
| 8 | `NavModifierVolume` | Runtime navmesh modification | Entity → Actor |
| 9 | `Interactable` | Standardized interaction framework | Entity → Actor → Paintable |
| 10 | `ProjectilePool` | Object pooling for projectiles | Entity |
| 11 | `Radar` | Minimap/radar HUD rendering | Entity |
| 12 | `Inventory` | Item container/slot management | Entity |
| 13 | `Team` | Team management and scoring | Entity |
| 14 | `GameMode` | Match lifecycle and rules | Entity |
| 15 | `Fog` | Localized fog volume | Entity → Actor |
| 16 | `PhysicsField` | Localized physics force field | Entity → Actor |
