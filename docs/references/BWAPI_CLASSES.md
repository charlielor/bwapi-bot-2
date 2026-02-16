# BWAPI Classes Reference

### Core Interfaces

| Class | Purpose |
|-------|---------|
| **`Game`** (`Broodwar`) | Main game interface — query map/units/players, issue global commands, draw debug info |
| **`Unit`** | Individual unit — get state (HP, position, orders), issue commands (attack, move, build, gather) |
| **`Player`** | Player info — resources, supply, unit counts, tech/upgrade status |
| **`AIModule`** | Base class your bot inherits — event callbacks (`onFrame`, `onStart`, `onUnitCreate`, etc.) |

### Type/Data Classes (static info lookups)

| Class | Purpose |
|-------|---------|
| **`UnitType`** | Unit stats — cost, HP, weapons, build requirements, capabilities (`isWorker()`, `isBuilding()`, `canAttack()`) |
| **`TechType`** | Researchable abilities — cost, energy cost, what researches it, what units use it |
| **`UpgradeType`** | Passive upgrades — cost per level, max level, what building upgrades it |
| **`WeaponType`** | Weapon stats — damage, cooldown, range, splash, targeting (air/ground) |
| **`Race`** | Race defaults — `getWorker()`, `getResourceDepot()`, `getRefinery()`, `getSupplyProvider()` |
| **`Order`** | Current unit action enum (Attack, Move, MiningMinerals, Train, etc.) |
| **`UnitSizeType`** | Small/Medium/Large — affects damage calculations |
| **`DamageType`** | Normal/Concussive/Explosive — interacts with unit size |
| **`BulletType`** | Projectile visual types |
| **`Error`** | Why a command failed (Insufficient_Minerals, Unbuildable_Location, etc.) |
| **`Color`** | Drawing colors for debug overlay |

### Collections

| Class | Purpose |
|-------|---------|
| **`Unitset`** | Set of units with batch commands (`attack()`, `move()`, `gather()`) and spatial queries (`getClosestUnit()`, `getUnitsInRadius()`) |
| **`Playerset`** | Set of players — `getUnits()`, `getRaces()` |

### Geometry

| Class | Purpose |
|-------|---------|
| **`Position`** | Pixel coordinates (1x1) |
| **`WalkPosition`** | Walk tile coordinates (8x8 pixels) |
| **`TilePosition`** | Build tile coordinates (32x32 pixels) |

### Filters (`BWAPI::Filter`)

Predefined predicates for filtering Unitsets: `IsWorker`, `IsBuilding`, `IsIdle`, `IsCompleted`, `CanAttack`, `IsFlyer`, `IsDetector`, `IsEnemy`, `IsAlly`, `HP`, `GetType`, etc.

### Supporting Classes

| Class | Purpose |
|-------|---------|
| **`Region`** | Map region — pathfinding, neighbors, terrain info |
| **`Force`** | Team/alliance grouping |
| **`Bullet`** | Active projectile — source, target, position |
| **`UnitCommand`** | Command object — encapsulates an action issued to a unit |
| **`Event`** | Game event from `Game::getEvents()` |
| **`Flag`** | `CompleteMapInformation`, `UserInput` — enable via `Broodwar->enableFlag()` |
