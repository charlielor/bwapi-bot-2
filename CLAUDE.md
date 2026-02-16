# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a BWAPI (Brood War API) bot project for StarCraft: Brood War, designed as a platform for developing AI bots with Claude Code agents. It uses BWTA2 for map analysis and contains the supporting BWAPI libraries needed to build bots that play StarCraft autonomously.

## Build System

- **Solution:** `ExampleProjects.sln` — Visual Studio 2015, targeting v120 (Visual Studio 2013) toolset, Win32 only
- **Build:** Open in Visual Studio 2015, build Debug|Win32 or Release|Win32
- **No tests or linter** — this is a game AI project; testing is done by running the bot in StarCraft

## Architecture

**The active development area is `ExampleAIModule/`.** Other directories (ExampleAIClient, BWAPIClient, BWAPILIB, AIModuleLoader, ExampleTournamentModule, Shared, Chaoslauncher) are supporting infrastructure — treat them as read-only context.

### ExampleAIModule (DLL) — `ExampleAIModule/`
- Compiles to a DLL loaded directly into StarCraft via Chaoslauncher
- `ExampleAIModule` inherits from `BWAPI::AIModule` and implements event callbacks
- `Dll.cpp` exports `newAIModule()` (factory) and `gameInit()` (sets the Broodwar game pointer)
- Currently implements a simple Terran strategy: train SCVs, build Supply Depots and Barracks, produce Marines
- Depends on BWAPILIB (static lib); BWAPI headers are in `include/BWAPI/`

### Reference Directories
- **include/BWAPI/** — All BWAPI headers; `include/BWAPI.h` is the single-include entry point
- **Starcraft/bwapi-data/** — Runtime directory with `bwapi.ini` config, BWAPI DLLs, and map data

## BWAPI Callback Model

All bot logic flows through virtual methods on `BWAPI::AIModule`. The most important:
- `onStart()` — game initialization (set flags, analyze map)
- `onFrame()` — called every game frame; this is where the main AI logic lives
- `onEnd(bool isWinner)` — match complete
- `onUnitCreate/Destroy/Complete(Unit)` — unit lifecycle events

## Configuration

`Starcraft/bwapi-data/bwapi.ini` controls BWAPI behavior:
- `[ai] ai = bwapi-data/AI/ExampleAIModule.dll` — which AI DLL to load
- `[auto_menu]` — automated game setup (race, map, enemy count/race)
- `[config] shared_memory = ON` — required for client-mode bots

## Running a Bot

1. Build ExampleAIModule (Release)
2. Copy the output DLL to `Starcraft/bwapi-data/AI/`
3. Ensure `bwapi.ini` points to the DLL
4. Launch StarCraft via Chaoslauncher with the BWAPI injector plugin enabled
5. Start a game — the bot takes control automatically

## BWAPI Classes Reference
See [BWAPI_CLASSES.md](./docs/BWAPI_CLASSES.md) for the full BWAPI classes reference.
