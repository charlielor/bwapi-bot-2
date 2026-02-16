# bwapi-bot-2

A StarCraft: Brood War bot built with BWAPI and BWTA2, designed as a platform for developing AI strategies with Claude Code agents.

## Build

- **IDE:** Visual Studio 2015, targeting v120 (Visual Studio 2013) toolset
- **Solution:** `ExampleProjects.sln` — build Debug|Win32 or Release|Win32
- **Dependencies:** BWAPI 4.x, BWTA2 (included)

## Running

1. Build ExampleAIModule (Release)
2. Copy the output DLL to `Starcraft/bwapi-data/AI/`
3. Ensure `Starcraft/bwapi-data/bwapi.ini` points to the DLL
4. Launch StarCraft via Chaoslauncher with the BWAPI injector plugin enabled
