# How BWAPI Gets Into Brood War and Runs Bots

## Overview

BWAPI (Brood War Application Programming Interface) uses **DLL injection** to insert itself into the StarCraft: Brood War process at runtime. Once inside, it hooks into the game's internal loop, reads game state from memory, and forwards that state to your bot code through a well-defined callback interface. There are two distinct execution modes: **Module mode** (in-process DLL) and **Client mode** (out-of-process via shared memory).

---

## The Injection Chain

### Step 1: Chaoslauncher Launches StarCraft

The process starts with **Chaoslauncher** (`Chaoslauncher/Chaoslauncher.exe`), a third-party launcher originally built for loading "BW Launcher" plugins. It doesn't run StarCraft normally. Instead, it:

1. Starts `StarCraft.exe` in a **suspended state** (using `CREATE_SUSPENDED` in the Windows `CreateProcess` call).
2. Injects one or more `.bwl` plugin DLLs into the StarCraft process before it begins executing.
3. Resumes the process.

The key plugin is **`BWAPI_PluginInjector.bwl`** (found in `Chaoslauncher/Plugins/`). This plugin is responsible for loading the core `BWAPI.dll` into the StarCraft process.

### Step 2: BWAPI.dll Hooks the Game Engine

Once `BWAPI.dll` (`Starcraft/bwapi-data/BWAPI.dll`) is loaded inside the StarCraft process, it **patches the game's binary in memory** to intercept key functions. Specifically, BWAPI:

- **Hooks the game loop** to gain control once per logical frame.
- **Reads the game's internal data structures** directly from memory (unit tables, player data, map tiles, fog of war state, etc.) using hardcoded offsets for StarCraft version 1.16.1.
- **Intercepts game commands** so it can issue orders (move, attack, build) on behalf of the bot.

This is why BWAPI is locked to StarCraft version **1.16.1** -- the memory offsets are specific to that binary.

### Step 3: BWAPI Reads `bwapi.ini` Configuration

On initialization, BWAPI reads its config file at `Starcraft/bwapi-data/bwapi.ini`. The critical settings:

```ini
[ai]
ai     = bwapi-data/AI/ExampleAIModule.dll    ; which bot DLL to load
ai_dbg = bwapi-data/AI/ExampleAIModuled.dll   ; debug build variant

[config]
shared_memory = ON                             ; enable client-mode server

[auto_menu]
auto_menu = OFF                                ; can automate menu navigation
race = Terran                                  ; which race to play
enemy_count = 1                                ; number of AI opponents
```

The `ai` field tells BWAPI which bot DLL to load. The `shared_memory` field controls whether the BWAPI server (for client-mode bots) is enabled.

---

## Two Execution Modes

### Mode 1: Module Mode (In-Process DLL)

This is the primary mode used by `ExampleAIModule`. Your bot compiles to a **DLL** that BWAPI loads directly into the StarCraft process.

**How it works:**

1. BWAPI reads the `ai` path from `bwapi.ini`.
2. It calls `LoadLibrary()` to load your bot DLL into the StarCraft process.
3. It calls two **exported C functions** from your DLL:
   - `gameInit(Game* game)` -- passes the BWAPI `Game` pointer to your module
   - `newAIModule()` -- factory function that returns an instance of your `AIModule` subclass

These exports are defined in `ExampleAIModule/Source/Dll.cpp`:

```cpp
extern "C" __declspec(dllexport) void gameInit(BWAPI::Game* game) {
    BWAPI::BroodwarPtr = game;
}

extern "C" __declspec(dllexport) BWAPI::AIModule* newAIModule() {
    return new ExampleAIModule();
}
```

The `gameInit` call sets the global `BroodwarPtr`, which is what the `Broodwar` wrapper object uses internally. This gives your bot access to the entire game state through the `Game` interface.

4. Once loaded, BWAPI calls your `AIModule` virtual methods from inside the game loop:
   - `onStart()` when a match begins
   - `onFrame()` every logical game frame (~24 FPS at normal speed)
   - `onEnd(bool isWinner)` when the match ends
   - Various unit lifecycle callbacks (`onUnitCreate`, `onUnitDestroy`, `onUnitComplete`, etc.)

**Pros:** Lowest latency -- your code runs in the same process as StarCraft, in the same thread as the game loop. No IPC overhead.

**Cons:** A crash in your bot crashes StarCraft. You're limited to C++ (or languages that can produce compatible DLLs).

### Mode 2: Client Mode (Out-of-Process via Shared Memory)

This mode lets your bot run as a **separate process** that communicates with BWAPI through shared memory and named pipes.

**How it works:**

1. BWAPI (inside StarCraft) acts as a **server**. When `shared_memory = ON` in `bwapi.ini`, it creates:
   - A **shared memory region** named `Local\bwapi_shared_memory_<PID>` containing a `GameData` struct
   - A **named pipe** at `\\.\pipe\bwapi_pipe_<PID>` for synchronization
   - A **game table** at `Local\bwapi_shared_memory_game_list` that lists active game instances

2. Your bot runs as a standalone `.exe` (like `ExampleAIClient` or `AIModuleLoader`) and uses the `BWAPIClient` library to connect:
   - Opens the game table shared memory to find an available game instance
   - Opens that instance's shared memory region to get a pointer to `GameData`
   - Opens the named pipe for frame-by-frame synchronization

From `BWAPIClient/Source/Client.cpp`:

```cpp
// Find the game table
this->gameTableFileHandle = OpenFileMappingA(..., "Local\\bwapi_shared_memory_game_list");

// Map the game data
mapFileHandle = OpenFileMappingA(..., sharedMemoryName.str().c_str());
data = static_cast<GameData*>(MapViewOfFile(mapFileHandle, ...));

// Connect via named pipe
pipeObjectHandle = CreateFileA(communicationPipe.str().c_str(), ...);
```

3. Each frame, the client and server **synchronize via the pipe**:
   - The server writes game state into shared memory, then signals the client (writes `2` to the pipe)
   - The client reads state, runs bot logic, writes commands back to shared memory, then signals the server (writes `1` to the pipe)
   - The server reads commands and executes them in the game

4. The `AIModuleLoader` variant adds another layer: it's a client-mode `.exe` that dynamically loads a bot DLL (via `LoadLibrary` + `GetProcAddress`), letting you use the Module-style `AIModule` interface while running out-of-process.

**Pros:** Bot crashes don't crash StarCraft. Enables non-C++ languages (Java, Python, etc.) via wrappers around the shared memory protocol.

**Cons:** Higher latency due to IPC. One frame of lag between reading state and commands taking effect.

---

## The GameData Shared Memory Layout

The `GameData` struct (defined in `include/BWAPI/Client/GameData.h`) is the core data contract between BWAPI and client-mode bots. It's a single flat C struct (~several MB) containing:

| Section | Contents |
|---------|----------|
| `players[12]` | All player data (resources, supply, race, upgrades) |
| `units[10000]` | All unit data (type, position, HP, orders, state flags) |
| `bullets[100]` | Active bullet/projectile data |
| `isWalkable[1024][1024]` | Walk-grid terrain passability |
| `isBuildable[256][256]` | Build-grid terrain |
| `isVisible[256][256]` | Current fog of war state |
| `events[10000]` | Game events (unit created, destroyed, etc.) |
| `commands[20000]` | Bot commands back to the server |
| `unitCommands[20000]` | Unit-specific commands |
| `shapes[20000]` | Debug drawing commands |

The entire game state is serialized into this struct every frame by the BWAPI server, and commands are read back from the `commands`/`unitCommands` arrays.

---

## The AIModule Callback Interface

Regardless of mode, your bot implements `BWAPI::AIModule` (defined in `include/BWAPI/AIModule.h`). The key callbacks:

```
onStart()                        -- match initialization
onFrame()                        -- called every game frame (main logic goes here)
onEnd(bool isWinner)             -- match finished
onUnitCreate(Unit)               -- a unit was created
onUnitDestroy(Unit)              -- a unit was destroyed
onUnitComplete(Unit)             -- a unit finished building/training
onUnitMorph(Unit)                -- a unit changed type (Zerg morphing, siege mode, etc.)
onUnitShow(Unit) / onUnitHide(Unit) -- visibility changes
onSendText(string)               -- user typed a chat message
onNukeDetect(Position)           -- nuclear launch detected
```

In Module mode, BWAPI calls these directly. In Client mode, the `BWAPIClient.update()` call reads events from shared memory and dispatches them to your callbacks.

---

## The Full Lifecycle

```
User launches Chaoslauncher
    |
    v
Chaoslauncher starts StarCraft.exe (suspended)
    |
    v
BWAPI_PluginInjector.bwl injects BWAPI.dll into StarCraft
    |
    v
StarCraft resumes, BWAPI hooks the game loop
    |
    v
BWAPI reads bwapi.ini
    |
    +--> [Module mode] LoadLibrary(bot.dll) -> gameInit() -> newAIModule()
    |
    +--> [Client mode] Creates shared memory + named pipe, waits for client
    |
    v
Game match starts
    |
    v
Each frame:
    BWAPI reads StarCraft memory -> populates game state
        |
        +--> [Module mode] Calls bot->onFrame() directly
        |
        +--> [Client mode] Writes to shared memory, signals pipe,
        |                   waits for client to process and signal back
        |
        v
    Bot issues commands (attack, move, build, train)
        |
        v
    BWAPI translates commands -> writes to StarCraft memory
    |
    v
Game match ends -> onEnd() callback
```

---

## Key Takeaways

1. **BWAPI is a memory hack.** It works by patching and reading the StarCraft 1.16.1 binary at runtime, which is why it requires that exact game version.

2. **Chaoslauncher is the entry point.** It handles the DLL injection that gets BWAPI into the StarCraft process.

3. **Two bot architectures exist.** Module mode (in-process, lower latency) and Client mode (out-of-process, more robust). Both use the same `AIModule` callback interface.

4. **The bot DLL contract is simple.** Export `gameInit()` and `newAIModule()`, subclass `AIModule`, implement callbacks. BWAPI handles everything else.

5. **Shared memory is the IPC mechanism.** A flat `GameData` struct holds the entire game state, synchronized frame-by-frame through Windows named pipes.
