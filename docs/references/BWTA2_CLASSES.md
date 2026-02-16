# BWTA2 Reference

BWTA2 (Brood War Terrain Analyzer 2) analyzes StarCraft: Brood War maps and decomposes them into regions, chokepoints, and base locations. It provides pathfinding, distance calculations, and spatial queries on top of this terrain graph. Call `BWTA::readMap()` in `onStart()`, then `BWTA::analyze()` (typically on a background thread) before using any query functions.

Include: `#include <BWTA.h>`

---

## Lifecycle Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `readMap()` | `void` | Reads raw map data from BWAPI. Call in `onStart()` before `analyze()` |
| `analyze()` | `void` | Performs full terrain analysis (regions, chokepoints, bases). Expensive — run on a background thread |
| `computeDistanceTransform()` | `void` | Computes distance transform map (distance from nearest unwalkable tile for every tile) |
| `balanceAnalysis()` | `void` | Analyzes map balance between start locations |
| `cleanMemory()` | `void` | Frees all BWTA2 allocated memory. Call when analysis data is no longer needed |

## Query Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `getRegions()` | `const std::set<Region*>&` | All analyzed regions on the map |
| `getChokepoints()` | `const std::set<Chokepoint*>&` | All chokepoints on the map |
| `getBaseLocations()` | `const std::set<BaseLocation*>&` | All base locations (mineral/gas clusters) |
| `getStartLocations()` | `const std::set<BaseLocation*>&` | Subset of base locations that are player start positions |
| `getUnwalkablePolygons()` | `const std::set<Polygon*>&` | All unwalkable terrain polygons |
| `getStartLocation(Player player)` | `BaseLocation*` | Start location for a specific player |
| `getMaxDistanceTransform()` | `int` | Maximum value in the distance transform map |
| `getDistanceTransformMap()` | `RectangleArray<int>*` | The full distance transform grid |

## Spatial Lookup Functions

All spatial lookups have overloads for `(int x, int y)`, `(TilePosition)`, and `(Position)` unless noted.

| Function | Returns | Description |
|----------|---------|-------------|
| `getRegion(int x, int y)` | `Region*` | Region containing the given tile coordinates |
| `getRegion(TilePosition)` | `Region*` | Region containing the given tile position |
| `getRegion(Position)` | `Region*` | Region containing the given pixel position |
| `getNearestChokepoint(int x, int y)` | `Chokepoint*` | Closest chokepoint to tile coordinates |
| `getNearestChokepoint(TilePosition)` | `Chokepoint*` | Closest chokepoint to tile position |
| `getNearestChokepoint(Position)` | `Chokepoint*` | Closest chokepoint to pixel position |
| `getNearestBaseLocation(int x, int y)` | `BaseLocation*` | Closest base location to tile coordinates |
| `getNearestBaseLocation(TilePosition)` | `BaseLocation*` | Closest base location to tile position |
| `getNearestBaseLocation(Position)` | `BaseLocation*` | Closest base location to pixel position |
| `getNearestUnwalkablePolygon(int x, int y)` | `Polygon*` | Closest unwalkable polygon to tile coordinates |
| `getNearestUnwalkablePolygon(TilePosition)` | `Polygon*` | Closest unwalkable polygon to tile position |
| `getNearestUnwalkablePosition(Position)` | `Position` | Nearest unwalkable pixel position (note: returns Position, not Polygon) |

## Pathfinding Functions

| Function | Returns | Description |
|----------|---------|-------------|
| `isConnected(int x1, int y1, int x2, int y2)` | `bool` | Whether two tile positions are ground-connected |
| `isConnected(TilePosition a, TilePosition b)` | `bool` | Whether two tile positions are ground-connected |
| `getGroundDistance(TilePosition start, TilePosition end)` | `double` | Ground walking distance between two tiles |
| `getNearestTilePosition(TilePosition start, const std::set<TilePosition>& targets)` | `std::pair<TilePosition, double>` | Nearest target tile and its ground distance |
| `getGroundDistances(TilePosition start, const std::set<TilePosition>& targets)` | `std::map<TilePosition, double>` | Ground distance from start to each target |
| `getGroundDistanceMap(TilePosition start, RectangleArray<double>& distanceMap)` | `void` | Fills a grid with ground distances from start to every tile |
| `getGroundWalkDistanceMap(int walkx, int walky, RectangleArray<double>& distanceMap)` | `void` | Same as above but using walk tile coordinates |
| `getShortestPath(TilePosition start, TilePosition end)` | `std::vector<TilePosition>` | Tile-by-tile shortest ground path |
| `getShortestPath(TilePosition start, const std::set<TilePosition>& targets)` | `std::vector<TilePosition>` | Shortest path from start to nearest target in set |

## HPA* Functions

Higher-level pathfinding using chokepoint graph nodes. Call `buildChokeNodes()` once after `analyze()`.

| Function | Returns | Description |
|----------|---------|-------------|
| `buildChokeNodes()` | `void` | Builds the chokepoint node graph for HPA* pathfinding |
| `getShortestPath2(TilePosition start, TilePosition target)` | `std::list<Chokepoint*>` | Shortest path as a sequence of chokepoints to traverse |
| `getGroundDistance2(TilePosition start, TilePosition end)` | `int` | Approximate ground distance using the chokepoint graph |

---

## Classes

### BaseLocation

Represents a viable base expansion site — a cluster of mineral patches and/or gas geysers.

| Method | Returns | Description |
|--------|---------|-------------|
| `getPosition()` | `Position` | Center pixel position of the base |
| `getTilePosition()` | `TilePosition` | Top-left build tile for a resource depot at this base |
| `getRegion()` | `Region*` | The region this base is located in |
| `minerals()` | `int` | Total remaining mineral count across all patches |
| `gas()` | `int` | Total remaining gas count across all geysers |
| `getMinerals()` | `const Unitset&` | Current mineral patch units (updates as patches mine out) |
| `getStaticMinerals()` | `const Unitset&` | Original mineral patch units at game start |
| `getGeysers()` | `const Unitset&` | Vespene geyser units at this base |
| `getGroundDistance(BaseLocation* other)` | `double` | Ground walking distance to another base |
| `getAirDistance(BaseLocation* other)` | `double` | Straight-line air distance to another base |
| `isIsland()` | `bool` | True if base is not ground-reachable from any start location |
| `isMineralOnly()` | `bool` | True if base has minerals but no gas geysers |
| `isStartLocation()` | `bool` | True if this is a player start position |

### Chokepoint

A narrow passage between two regions. Useful for wall-ins, defensive positioning, and army movement.

| Method | Returns | Description |
|--------|---------|-------------|
| `getRegions()` | `const std::pair<Region*, Region*>&` | The two regions connected by this chokepoint |
| `getSides()` | `const std::pair<Position, Position>&` | The two edge positions defining the choke opening |
| `getCenter()` | `Position` | Center pixel position of the chokepoint |
| `getWidth()` | `double` | Width of the chokepoint in pixels |

### Region

A contiguous walkable area of the map, bounded by chokepoints and map edges.

| Method | Returns | Description |
|--------|---------|-------------|
| `getPolygon()` | `const Polygon&` | The polygon boundary of this region |
| `getCenter()` | `const Position&` | Center pixel position of the region |
| `getChokepoints()` | `const std::set<Chokepoint*>&` | Chokepoints on the border of this region |
| `getBaseLocations()` | `const std::set<BaseLocation*>&` | Base locations within this region |
| `isReachable(Region* region)` | `bool` | Whether another region is ground-reachable from this one |
| `getReachableRegions()` | `const std::set<Region*>&` | All regions ground-reachable from this one |
| `getMaxDistance()` | `int` | Maximum distance transform value within this region |

### Polygon

A geometric boundary shape. Extends `std::vector<Position>` — iterate it directly for the vertex positions.

| Method | Returns | Description |
|--------|---------|-------------|
| `getArea()` | `double` | Area of the polygon |
| `getPerimeter()` | `double` | Perimeter length of the polygon |
| `getCenter()` | `Position` | Centroid of the polygon |
| `isInside(Position p)` | `bool` | Whether a position is inside the polygon |
| `getNearestPoint(Position p)` | `Position` | Closest point on the polygon boundary to the given position |
| `getHoles()` | `const std::vector<Polygon>&` | Interior holes (unwalkable areas within the polygon) |

### RectangleArray\<T\>

A 2D grid template used internally by BWTA2 for distance transform maps and ground distance maps. Not typically used directly in bot code — you interact with it when calling `getGroundDistanceMap()` or `getDistanceTransformMap()`. Access elements with `array[x][y]`, query dimensions with `getWidth()` and `getHeight()`.

---

## Usage Pattern

```cpp
// In onStart():
BWTA::readMap();
analyzed = false;
CreateThread(NULL, 0, (LPTHREAD_START_ROUTINE)AnalyzeThread, NULL, 0, NULL);

// Background thread:
DWORD WINAPI AnalyzeThread() {
    BWTA::analyze();
    analyzed = true;
    return 0;
}

// In onFrame(), after analysis completes:
if (analyzed) {
    // Query base locations
    for (auto& base : BWTA::getBaseLocations()) {
        TilePosition pos = base->getTilePosition();
        bool island = base->isIsland();
    }

    // Find chokepoints near your base
    BaseLocation* myStart = BWTA::getStartLocation(Broodwar->self());
    Region* myRegion = myStart->getRegion();
    for (auto& choke : myRegion->getChokepoints()) {
        double width = choke->getWidth();
        Position center = choke->getCenter();
    }

    // Pathfinding
    double dist = BWTA::getGroundDistance(startTile, targetTile);
    std::vector<TilePosition> path = BWTA::getShortestPath(startTile, targetTile);
}
```
