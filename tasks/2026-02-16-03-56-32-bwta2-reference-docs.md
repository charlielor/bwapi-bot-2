## Goal
Create a BWTA2 reference document in `docs/references/BWTA2_CLASSES.md`, following the same format as the existing `BWAPI_CLASSES.md`, to give Claude Code agents a quick reference for all BWTA2 classes and functions.

## BWAPI Functions / Types Involved
None directly — this is a documentation task for the BWTA2 library that wraps on top of BWAPI types (`Position`, `TilePosition`, `Player`, `Unitset`).

## Steps
1. Create `docs/references/BWTA2_CLASSES.md` with the following sections:
   - **Overview** — one-paragraph summary of what BWTA2 does (terrain analysis: regions, chokepoints, base locations, pathfinding)
   - **Lifecycle Functions** — table documenting `readMap()`, `analyze()`, `computeDistanceTransform()`, `balanceAnalysis()`, `cleanMemory()` with descriptions and when to call them
   - **Query Functions** — table documenting `getRegions()`, `getChokepoints()`, `getBaseLocations()`, `getStartLocations()`, `getUnwalkablePolygons()`, `getStartLocation(Player)`
   - **Spatial Lookup Functions** — table documenting `getRegion()`, `getNearestChokepoint()`, `getNearestBaseLocation()`, `getNearestUnwalkablePolygon()`, `getNearestUnwalkablePosition()` with their overloads (x/y, TilePosition, Position)
   - **Pathfinding Functions** — table documenting `isConnected()`, `getGroundDistance()`, `getNearestTilePosition()`, `getGroundDistances()`, `getGroundDistanceMap()`, `getGroundWalkDistanceMap()`, `getShortestPath()` overloads
   - **HPA\* Functions** — table documenting `buildChokeNodes()`, `getShortestPath2()`, `getGroundDistance2()`
   - **Classes** section with sub-tables for each class:
     - `BaseLocation` — all 13 methods (`getPosition`, `getTilePosition`, `getRegion`, `minerals`, `gas`, `getMinerals`, `getStaticMinerals`, `getGeysers`, `getGroundDistance`, `getAirDistance`, `isIsland`, `isMineralOnly`, `isStartLocation`)
     - `Chokepoint` — all 4 methods (`getRegions`, `getSides`, `getCenter`, `getWidth`)
     - `Region` — all 6 methods (`getPolygon`, `getCenter`, `getChokepoints`, `getBaseLocations`, `isReachable`, `getReachableRegions`, `getMaxDistance`)
     - `Polygon` — all 6 methods (`getArea`, `getPerimeter`, `getCenter`, `isInside`, `getNearestPoint`, `getHoles`)
     - `RectangleArray<T>` — brief mention as internal utility (not typically used directly by bot code)
   - **Usage Pattern** — brief code snippet showing the typical `readMap()` → background thread `analyze()` → query pattern, referencing how the bot currently uses it

2. Add a link to the new doc from `CLAUDE.md` in the existing "BWAPI Classes Reference" section

## Assumptions
- Follow the same table-based format used in `BWAPI_CLASSES.md`
- Document is intended as a quick-reference for Claude Code agents, not exhaustive API docs
- `RectangleArray` gets minimal coverage since it's an internal data structure, not typically called directly
- Method signatures include return types and parameter types for clarity
