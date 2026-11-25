# ORBIT: SECOND STAGE — Consolidated Implementation Plan

> An inspired re-creation of the classic Orbit space combat simulator. Built with Babylon.js / TypeScript / Vite. Not a port of the original C code — a fresh implementation of the core game mechanics, with no copyrighted material from the 1999 original.
> Architecture-first approach: multiplayer-ready from day one, Electron support early, high code coverage from the start.

---

## Table of Contents

1. [Overview & Key Architectural Decisions](#1-overview--key-architectural-decisions)
2. [Architecture Principles](#2-architecture-principles)
3. [Project Structure](#3-project-structure)
4. [Build & Test Infrastructure](#4-build--test-infrastructure)
5. [Types](#5-types)
6. [Files](#6-files)
7. [Functions](#7-functions)
8. [Classes](#8-classes)
9. [Constants, Definitions & Game Data](#9-constants-definitions--game-data)
10. [Dependencies](#10-dependencies)
11. [Testing Strategy](#11-testing-strategy)
12. [Implementation Order — Phases 0–12](#12-implementation-order--phases-012)
13. [Risk Register](#13-risk-register)
14. [Phase Dependencies](#14-phase-dependencies)

---

## 1. Overview & Key Architectural Decisions

This document defines the step-by-step implementation of **Orbit: Second Stage**, a Babylon.js / TypeScript / Vite space combat simulator. The project is an inspired re-creation of the 1999 freeware game *Orbit* by Steve Belczyk — not a port, but a fresh implementation using modern web technologies. The first playable build recreates the Original Orbit experience (flight, combat, missions, solar system). Second Stage content (new missions, ships, weapons, story) is added incrementally after the core engine is solid.

### Key Architectural Decisions

| Decision                                                                                   | Rationale                                                                                                                                         |
| ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Babylon.js full** (not Lite)                                                             | WebXR support requires the full module set; Lite strips it out                                                                                    |
| **Y-up world coordinate system** | Babylon.js native Y-up — we use `@babylonjs/core/Maths` (Vector3, Matrix4) directly throughout; Z-up data from the original game is converted to Y-up during mission JSON generation as a prep step |
| **JSON mission files** replacing the original text-based `.msn` format                     | Enables web serving, future mission editor tooling, and easier parsing                                                                            |
| **Interface-segregated architecture**                                                      | Every game system has an `I*` interface with single-player and network variants from day one, even if network implementations are stubs initially |
| **Electron as native target**                                                              | Mature `steamworks.js` integration, established IPC patterns for Babylon.js ↔ Steam SDK bridge                                                    |
| **Placeholders first**                                                                     | Colored spheres, boxes, and basic geometries for early phases; real GLTF models swapped in later                                                  |
| **Multiplayer-ready from day one**                                                         | Interfaces designed for networked play, but implementations deferred to Phase 10                                                                  |
| **Deterministic physics**                                                                  | Fixed-timestep accumulator pattern (1/60s tick) — reproducible across platforms, essential for netcode later                                      |
| **Babylon.js for math and rendering** | `@babylonjs/core/Maths` (Vector3, Matrix4) used throughout — tree-shaken to minimal bundle; Babylon.js only for rendering/input; everything else is pure TypeScript — easy to test in Node, run headless server               |

---

## 2. Architecture Principles

### 2.1 Dual Game Modes

| Mode               | Description                                            | Scope                                                                  | Story Context                                                                   |
| ------------------ | ------------------------------------------------------ | ---------------------------------------------------------------------- | ------------------------------------------------------------------------------- |
| **Original Orbit** | Faithful recreation of the 1999 game mechanics         | Core gameplay loop: flight, combat, missions, solar system exploration | The events of the original Orbit — you are the pilot defending the solar system |
| **Second Stage**   | Additional features, ships, weapons, and story content | Layers on top of Original — unlocks after completing core content      | Set after the events of the original — new threats, new tech, expanded universe |

Both modes share the same core game engine, physics, and systems. Mode selection determines which mission sets, ships, weapons, and HUD elements are available. Save system tracks progress per mode independently.

### 2.2 Network-Ready Architecture Pattern

Every game system follows this pattern:

```txt
IGame Interface (all game logic interfaces)
├── SPGame impl
├── NetGame impl
└── Headless Server
```

Examples:

- `IWeaponSystem` → `SinglePlayerWeaponSystem` + `NetworkedWeaponSystem`
- `IAIManager` → `SinglePlayerAIManager` + `ServerAIManager` (server-side AI)
- `IPlayerController` → `KeyboardPlayerController` + `JoystickPlayerController` + `NetworkPlayerProxy`

### 2.3 Hybrid Network Architecture

**Platform-specific networking:**

| Platform             | Transport                       | Auth                 | Features                                        |
| -------------------- | ------------------------------- | -------------------- | ----------------------------------------------- |
| **Electron**         | Steam SDR via `steamworks.js`   | Session ticket       | Lobbies, overlay, matchmaking, NAT punch        |
| **Browser**          | WebSocket (`wss://`) to relay   | OpenID 2.0 → Web API | Server-verified ownership, no Steam overlay     |
| **Dedicated Server** | Steam SDR + WebSocket dual port | GSLT                 | Master Server registration, bridges both worlds |

**Authentication flow (browser clients):**

1. Browser → Steam OpenID 2.0 (`steamcommunity.com/openid/login`)
2. Steam returns verified `SteamID64` to server
3. Server calls `ISteamUser/CheckAppOwnership` with Publisher Key
4. If owned, grant access; otherwise reject

**Server discovery (browser clients):**

- Server queries `IGameServersService/GetServerList` via Steam Web API
- Returns JSON list of active server IPs/ports to browser clients
- Browser connects directly via WebSocket to selected server

### 2.4 Coordinate System

The game world uses **Y-up** — Babylon.js native coordinate system. All position/orientation data is stored in Y-up throughout the runtime codebase. We use `@babylonjs/core/Maths` (`Vector3`, `Matrix4`) directly for all vector and matrix operations.

Any Z-up data from the original game's documentation or specs is converted to Y-up during mission JSON generation as a prep step. The conversion formula: `(x, y, z)_Zup → (x, -z, y)_Yup`. This happens once when building mission files — never at runtime.

> **Rule:** No custom Vector3/Matrix4 implementations. Use `BABYLON.Vector3` and `BABYLON.Matrix` from `@babylonjs/core/Maths` everywhere — client, server, tests. Tree-shaking ensures minimal bundle impact.

### 2.5 Save System — Strict Separation

| Scenario                     | Where Saves Are Stored                            | Can Upload to Server? | Can Download from Server? |
| ---------------------------- | ------------------------------------------------- | --------------------- | ------------------------- |
| **Single-player (web)**      | Browser IndexedDB                                 | ❌ No                  | ❌ No                      |
| **Single-player (Electron)** | Local filesystem (`~/.orbit-second-stage/saves/`) | ❌ No                  | ❌ No                      |
| **Multiplayer session**      | Server-side file only                             | N/A                   | N/A                       |

Prevents save manipulation via server interaction. Keeps architecture simple — no sync complexity.

---

## 3. Project Structure

```txt
orbit-second-stage/
├── design/                    # Architecture docs, implementation plan
│   └── implementation_plan_consolidated.md  # This file
├── src/
│   ├── index.ts               # Web entry point (boot Babylon.js scene)
│   ├── main.ts                # Node/desktop entry point
│   │
│   │   ├── core/                  # Foundation — no game logic (uses @babylonjs/core/Maths for Vector3/Matrix4)
│   │   ├── Timer.ts           # Fixed-timestep accumulator
│   │   ├── EventBus.ts        # Event system (pub/sub)
│   │   ├── RNG.ts             # Seeded PRNG (Mulberry32)
│   │   ├── Logger.ts          # Structured logging
│   │   └── game-mode.ts       # GameMode enum + mode selector
│   │
│   ├── player/                # Player ship system
│   │   ├── IShip.ts           # Ship interface
│   │   ├── Ship.ts            # Core ship state (SP)
│   │   ├── FlightModel.ts     # Newtonian / Arcade models
│   │   ├── Camera.ts          # Babylon.js camera wrapper
│   │   └── InputManager.ts    # Keyboard + joystick abstraction
│   │
│   ├── weapons/               # Weapon system
│   │   ├── IWeaponSystem.ts
│   │   ├── SinglePlayerWeaponSystem.ts
│   │   ├── NetworkedWeaponSystem.ts
│   │   ├── WeaponDef.ts       # Weapon definitions (from missions)
│   │   ├── Missile.ts         # Missile entity
│   │   └── MissilePool.ts     # Object pool for missiles
│   │
│   ├── targets/               # AI target system
│   │   ├── ITargetManager.ts
│   │   ├── SinglePlayerAIManager.ts
│   │   ├── NetworkedAIManager.ts
│   │   ├── Target.ts          # Target entity state
│   │   ├── Strategy.ts        # Strategy enum + registry
│   │   ├── TurnToward.ts      # AI: rotate toward target
│   │   ├── MoveToward.ts      # AI: translate toward target
│   │   ├── FindEnemy.ts       # AI: closest-target search
│   │   └── Aim.ts             # AI: predictive aiming
│   │
│   ├── planets/               # Solar system
│   │   ├── ISolarSystem.ts
│   │   ├── SolarSystem.ts     # SP implementation
│   │   ├── Planet.ts          # Planet entity
│   │   └── Starfield.ts       # 65K+ star rendering
│   │
│   ├── missions/              # Mission parsing & execution
│   │   ├── MissionParser.ts   # JSON mission parser
│   │   ├── Mission.ts         # Parsed mission data
│   │   └── MissionLoader.ts   # Mission loading from URL/filesystem
│   │
│   ├── hud/                   # HUD rendering
│   │   ├── IHUD.ts
│   │   ├── SinglePlayerHUD.ts
│   │   ├── Radar.ts           # Circular radar overlay
│   │   ├── ShieldBar.ts       # Shield status bars
│   │   ├── Crosshair.ts       # Center crosshair
│   │   └── MessageOverlay.ts  # Briefing / console messages
│   │
│   ├── effects/               # Visual/audio effects
│   │   ├── Explosion.ts       # Icosahedron explosion
│   │   ├── ExplosionPool.ts   # Object pool
│   │   └── SpaceJunk.ts       # Motion reference polygons
│   │
│   ├── save/                  # Save system
│   │   ├── ISaveManager.ts
│   │   ├── SaveManager.ts     # SP save/load
│   │   └── SaveEntry.ts       # Save data structure
│   │
│   ├── network/               # Multiplayer layer (interfaces + impls)
│   │   ├── INetworkLayer.ts   # Network interface
│   │   ├── SPNetworkLayer.ts  # No-op for SP mode
│   │   ├── SteamNetwork.ts    # Native multiplayer (Steam SDR via steamworks.js)
│   │   ├── WebSocketNetwork.ts# Browser multiplayer (wss:// to relay server)
│   │   ├── Packet.ts          # Packet format definitions
│   │   └── Server.ts          # Headless dedicated server
│   │
│   ├── game/                  # Game orchestration
│   │   ├── IGame.ts           # Core game interface
│   │   ├── SinglePlayerGame.ts
│   │   ├── NetworkedGame.ts   # Client-side networked game
│   │   ├── GameState.ts       # State machine (enum)
│   │   └── GameFactory.ts     # Creates correct game mode
│   │
│   ├── xr/                    # WebXR / VR support
│   │   ├── WebXRInline.ts     # Inline XR session manager (Phase 1.7)
│   │   ├── WebXRManager.ts    # Full immersive VR session manager (Phase 12)
│   │   ├── XRCameraSwap.ts    # Standard ↔ XR camera swap (Phase 12)
│   │   └── VRControllerInput.ts # VR controller input mapping (Phase 12)
│   │
│   └── prototype/             # Quick visual prototypes
│       └── QuickScene.ts      # Phase 1.0: sun, planets, ship wireframe
│
├── electron/                  # Electron app
│   ├── main.js                # Electron main process
│   ├── preload.js             # Security bridge
│   └── package.json           # Electron-specific deps
│
├── tests/                     # Test suite (mirrors src/)
│   ├── core/
│   │   ├── vector3.test.ts
│   │   ├── matrix4.test.ts
│   │   ├── timer.test.ts
│   │   ├── event-bus.test.ts
│   │   ├── rng.test.ts
│   │   ├── logger.test.ts
│   │   └── game-mode.test.ts
│   ├── player/
│   │   ├── flight-model.test.ts
│   │   └── ship.test.ts
│   ├── planets/
│   │   └── starfield.test.ts
│   ├── missions/
│   │   └── parser.test.ts
│   ├── weapons/
│   │   ├── weapon-defs.test.ts
│   │   └── missile-pool.test.ts
│   ├── targets/
│   │   ├── strategy.test.ts
│   │   ├── turn-toward.test.ts
│   │   ├── move-toward.test.ts
│   │   └── find-enemy.test.ts
│   ├── effects/
│   │   └── explosion.test.ts
│   ├── save/
│   │   └── save-manager.test.ts
│   └── xr/
│       └── webxr-inline.test.ts
│
├── data/                    # Game assets (all original, no licensing concerns)
│   ├── missions/
│   │   ├── original/              # Classic Orbit missions (JSON format)
│   │   │   ├── train01.json
│   │   │   └── combat01.json
│   │   └── second-stage/          # Expansion missions (empty initially)
│   ├── models/placeholder/        # Placeholder models: ship-box.glb, station-box.glb
│   └── textures/placeholder/      # Placeholder textures: solid colors for planets
│
├── dist/                    # Build output
├── coverage/                # Vitest coverage reports
├── docs/                    # Developer documentation
├── package.json
├── tsconfig.json
├── vite.config.ts
├── vitest.config.ts
├── eslint.config.js
├── .prettierrc
└── .gitignore
```

---

## 4. Build & Test Infrastructure

### 4.1 Package.json Scripts

```json
{
  "scripts": {
    "dev": "vite --open",
    "build": "tsc && vite build",
    "build:electron": "npm run build && cd electron && npm run build",
    "preview": "vite preview",
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage",
    "test:watch": "vitest watch",
    "clean": "rm -rf dist coverage",
    "lint": "eslint src/**/*.ts tests/**/*.ts",
    "lint:fix": "eslint src/**/*.ts tests/**/*.ts --fix",
    "format": "prettier --write src/**/*.ts tests/**/*.ts",
    "docs": "npx typedoc",
    "server:start": "node dist/network/Server.js",
    "electron:start": "npm run build && cd electron && npm start"
  }
}
```

### 4.2 Vitest Configuration

```typescript
// vitest.config.ts
import { defineConfig } from 'vitest/config'

export default defineConfig({
  test: {
    coverage: {
      enabled: true,
      reporter: ['text', 'lcov'],
      include: ['src/**/*'],
      exclude: ['src/index.ts', 'src/main.ts'],
      lines: 90,
      branches: 85,
      functions: 90,
      statements: 90
    },
    environment: 'node',
    globals: true,
    testMatch: ['tests/**/*.test.ts']
  }
})
```

### 4.3 Coverage Thresholds (project-standard)

| Metric     | Threshold | Notes                                              |
| ---------- | --------- | -------------------------------------------------- |
| Lines      | 90%       | Core domain logic must be thoroughly tested        |
| Branches   | 85%       | AI strategies, event evaluation have many branches |
| Functions  | 90%       | Every public method must be called in tests        |
| Statements | 90%       | Consistent with line coverage                      |

### 4.4 Testing Approach

- **Vitest** with `node` environment for all tests (Babylon.js rendering tested via visual inspection + integration tests)
- **Core library tests** run in Node (no DOM required) — Vector3, Matrix4, Timer, EventBus, RNG, Logger
- **Game logic tests** run in Node — Ship, FlightModel, Target AI, MissionParser, WeaponDef
- **Babylon.js integration tests** require DOM/JSDOM — scene setup, camera, mesh creation, rendering
- **XR tests** use JSDOM + mocked `navigator.xr` — session lifecycle, support detection

### 4.5 Electron Setup (Early)

- `electron/main.js` — minimal window, loads Vite dev server in dev mode
- `electron/preload.js` — context bridge for IPC (`window.electronAPI`)
- `electron/package.json` — depends on `@electron/remote`, `electron`, `steamworks.js`
- Dev mode: Electron connects to Vite HMR
- Prod mode: Electron loads built files from `dist/`
- **Steam integration:** `steamworks.js` loaded in Electron main process (Rust N-API wrapper for Steamworks SDK)
  - Full access to: Steam lobbies, SDR networking, overlay, session tickets, matchmaking

### 4.6 Headless Dedicated Server

The server is a pure Node.js process with no rendering dependencies. Dual transport layer.

- `src/network/Server.ts` — Pure Node.js server, no DOM required
- Server initializes Steamworks SDK (`SteamGameServer_Init`) for Master Server registration + GSLT auth
- No Babylon.js dependency — headless server is pure Node.js

---

## 5. Types

### Core Types

```typescript
// Babylon.js math — used throughout (client + server + tests)
// import { Vector3, Matrix } from '@babylonjs/core/Maths'

// src/core/RNG.ts
type RNGState = { seed: number };

// src/core/Logger.ts
enum LogLevel { DEBUG = 0, INFO = 1, WARN = 2, ERROR = 3 }
interface LogEntry { level: LogLevel; message: string; timestamp: number; context?: Record<string, unknown> }

// src/core/Timer.ts
interface TimerState {
  accumulator: number;
  fixedDeltaT: number; // typically 1/60
  interpolationFactor: number; // 0..1 for rendering between ticks
}

// src/core/EventBus.ts
type EventHandler<T> = (data: T) => void;
```

### Game Types

```typescript
// src/player/Ship.ts
enum ShipState { INIT, NORMAL, DEAD1, DEAD2, LOADGAME }
interface ShipStateData {
  position: Vector3; velocity: Vector3; viewDir: Vector3; upDir: Vector3; rightDir: Vector3;
  shields: number; maxShields: number; score: number; selectedWeapon: number;
  flightModel: 'newtonian' | 'arcade'; state: ShipState; deathTimer: number;
}

// src/player/FlightModel.ts
interface FlightModelConfig {
  deltax: number; warpCoeff: number; maxThrottle: number; maxWarpThrottle: number; thetaRate: number;
}

// src/weapons/WeaponDef.ts
enum WeaponRenderer { NONE = 0, CROSS = 1, RAY = 2, TETRAHEDRON = 3, RANDOM_LOOP = 4 }
interface WeaponDef {
  name: string; speed: number; yield_: number; idleTime: number; lifetime: number;
  renderer: WeaponRenderer; color: [number, number, number];
}

// src/targets/Target.ts
enum AIFStrategy { DO_NOTHING = 0, SIT1 = 1, SIT2 = 2, SIT3 = 3, SIT4 = 4, HUNT1 = 5, HUNT2 = 6, HUNT3 = 7, HUNT4 = 8 }
interface TargetState {
  name: string; position: Vector3; velocity: Vector3; viewDir: Vector3; upDir: Vector3; rightDir: Vector3;
  modelBounds: { lobound: Vector3; hibound: Vector3 }; strategy: AIFStrategy; friendly: boolean;
  shields: number; maxShields: number; shieldRegen: number; weaponIndex: number;
  turnRate: number; maxSpeed: number; score: number; hidden: boolean; invisible: boolean; age: number;
}

// src/planets/Planet.ts
interface PlanetData {
  name: string; position: Vector3; velocity: Vector3; upDir: Vector3; viewDir: Vector3; rightDir: Vector3;
  radius: number; radiusSquared: number; mass: number; color: [number, number, number];
  distanceFromPrimary: number; angularVelocity: number; theta: number; oblicity: number;
  isMoon: boolean; primaryIndex: number; texturePath: string; customTexture: boolean; hidden: boolean;
}

// src/missions/Mission.ts (JSON-based, replaces original .msn text format)
interface MissionFile {
  cursor?: CursorDef[]; player?: PlayerDef; objects?: ObjectDef[]; briefing?: string;
  waypoints?: WaypointDef[]; events?: EventDef[]; planets?: PlanetModDef[]; weapons?: WeaponDef[]; verbose?: boolean;
}
interface CursorDef { name: string; offsetX?: number; offsetY?: number; offsetZ?: number; refPlanet?: string }
interface PlayerDef { name: string; position: [number, number, number]; velocity?: [number, number, number] }
interface ObjectDef {
  name: string; model: string; score: number; strategy: AIFStrategy; friendly?: boolean; hidden?: boolean;
  weapon?: number; maxShields?: number; shieldRegen?: number; turnRate?: number; speed?: number; invisible?: boolean;
}
interface WaypointDef { name: string }
interface EventDef { name?: string; trigger: string; triggerValue: number | string; enabled?: boolean; action: string; actionValue: number | string }
interface PlanetModDef { name: string; newName?: string; reposition?: boolean; hidden?: boolean; texture?: string; oblicity?: number; radius?: number }

// src/game/GameState.ts
enum GameState { MENU, BRIEFING, PLAYING, PAUSED, DEAD1, DEAD2, VICTORY, DEFEAT, LOADGAME }

// src/core/GameMode.ts
enum GameMode { ORIGINAL = 'original', SECOND_STAGE = 'second-stage' }

// src/network/Packet.ts (interface for future multiplayer)
enum PacketType { PLAYER_STATE, TARGET_STATES, WEAPON_FIRE, WEAPON_HIT, SCORE_UPDATE, MISSION_EVENT }
interface NetworkPacket { type: PacketType; timestamp: number; data: unknown }
```

---

## 6. Files

### New Files (Phase 0 — Foundation)

| File                    | Purpose                                                                            |
| ----------------------- | ---------------------------------------------------------------------------------- |
| `package.json`          | Project manifest with Vite, TypeScript, Vitest, Electron, Babylon.js dependencies  |
| `tsconfig.json`         | TypeScript config: ES2020, strict mode, moduleResolution bundler                   |
| `vite.config.ts`        | Vite config: dev server on port 5173, build to dist/, Electron preload bridge      |
| `vitest.config.ts`      | Vitest config: node environment, globals, coverage thresholds (90/85/90/90)        |
| `eslint.config.js`      | ESLint flat config: TypeScript parser, recommended rules                           |
| `.prettierrc`           | Prettier config: 2-space indent, semicolons, single quotes                         |
| `electron/main.js`      | Electron main process: window creation, preload bridge to renderer, Steam SDK init |
| `electron/preload.js`   | Context bridge: exposes `window.electronAPI` to renderer with IPC handlers         |
| `electron/package.json` | Electron-specific dependencies: electron, @electron/remote, steamworks.js          |

### New Files (Core Library — Phase 0)

| File                    | Purpose                                                                                        |
| ----------------------- | ---------------------------------------------------------------------------------------------- |
| `src/core/Timer.ts`     | Fixed-timestep accumulator: tick(), getDeltaT(), getAccumulator(), reset()                     |
| `src/core/EventBus.ts`  | Typed pub/sub event system: subscribe(), emit(), unsubscribe()                                 |
| `src/core/RNG.ts`       | Mulberry32 seeded PRNG: seed(), next(), nextInt(), nextFloat()                                 |
| `src/core/Logger.ts`    | Structured logging with levels: debug(), info(), warn(), error(), setLevel()                   |
| `src/core/game-mode.ts` | GameMode enum + mode selector: getActiveMode(), setMode(), getMissionPath(mode)                |

### New Files (Player System — Phase 1-2)

| File                         | Purpose                                                                                     |
| ---------------------------- | ------------------------------------------------------------------------------------------- |
| `src/player/IShip.ts`        | Ship interface: position, velocity, orientation, shields, state machine                     |
| `src/player/Ship.ts`         | Core ship state implementation (single-player): update(), takeDamage(), regenerateShields() |
| `src/player/FlightModel.ts`  | IFlightModel interface + NewtonianFlightModel + ArcadeFlightModel implementations           |
| `src/player/Camera.ts`       | Babylon.js ArcRotateCamera wrapper: updateFromShip(), setViewLock()                         |
| `src/player/InputManager.ts` | IInputManager interface + KeyboardInput + JoystickInput implementations                     |

### New Files (Solar System — Phase 1-3)

| File                         | Purpose                                                                             |
| ---------------------------- | ----------------------------------------------------------------------------------- |
| `src/planets/Planet.ts`      | Planet entity: position, velocity, radius, mass, orbit angle, rendering state       |
| `src/planets/SolarSystem.ts` | ISolarSystem interface + SolarSystem implementation: init(), update(), getGravity() |
| `src/planets/Starfield.ts`   | Starfield rendering: 65K+ stars with two density levels, distance-based culling     |

### New Files (Weapons & Combat — Phase 5)

| File                                      | Purpose                                                                    |
| ----------------------------------------- | -------------------------------------------------------------------------- |
| `src/weapons/WeaponDef.ts`                | Weapon definition data type + default weapon definitions                   |
| `src/weapons/Missile.ts`                  | Missile entity: position, velocity, lifetime, weaponDef, owner             |
| `src/weapons/MissilePool.ts`              | Object pool for missiles: acquire(), release(), updateAll()                |
| `src/weapons/IWeaponSystem.ts`            | IWeaponSystem interface: fireMissile(), updateWeapons(), checkCollisions() |
| `src/weapons/SinglePlayerWeaponSystem.ts` | SP weapon system implementation                                            |
| `src/weapons/NetworkedWeaponSystem.ts`    | Networked weapon system (stub for Phase 10)                                |

### New Files (AI Targets — Phase 6)

| File                                   | Purpose                                                                      |
| -------------------------------------- | ---------------------------------------------------------------------------- |
| `src/targets/Target.ts`                | Target entity state: position, velocity, strategy, shields, AI state         |
| `src/targets/Strategy.ts`              | AIFStrategy enum + strategy registry                                         |
| `src/targets/TurnToward.ts`            | AI primitive: rotate target's viewDir toward a point                         |
| `src/targets/MoveToward.ts`            | AI primitive: translate target toward a point respecting distance thresholds |
| `src/targets/FindEnemy.ts`             | AI primitive: find closest enemy target by squared distance                  |
| `src/targets/Aim.ts`                   | AI primitive: predictive aiming — compute future player position             |
| `src/targets/ITargetManager.ts`        | ITargetManager interface: updateTargets(), getTargets(), fireMissiles()      |
| `src/targets/SinglePlayerAIManager.ts` | SP AI manager implementation                                                 |
| `src/targets/NetworkedAIManager.ts`    | Networked AI manager (stub for Phase 10)                                     |

### New Files (Mission System — Phase 4)

| File                            | Purpose                                                                   |
| ------------------------------- | ------------------------------------------------------------------------- |
| `src/missions/MissionParser.ts` | JSON mission file parser: parseMission(json), validateMission(data)       |
| `src/missions/Mission.ts`       | Parsed mission data structure (matches MissionFile type above)            |
| `src/missions/MissionLoader.ts` | Mission loading from URL (web) or filesystem (Electron): loadMission(url) |

### New Files (HUD & UI — Phase 7)

| File                         | Purpose                                                                    |
| ---------------------------- | -------------------------------------------------------------------------- |
| `src/hud/IHUD.ts`            | IHUD interface: update(), render()                                         |
| `src/hud/SinglePlayerHUD.ts` | SP HUD implementation using Babylon.js GUI or DOM overlay                  |
| `src/hud/Radar.ts`           | Circular radar overlay: renderTargets(), renderPlayer()                    |
| `src/hud/ShieldBar.ts`       | Shield status bar with color coding (green > 66%, yellow > 33%, red < 33%) |
| `src/hud/Crosshair.ts`       | Center crosshair rendering                                                 |
| `src/hud/MessageOverlay.ts`  | Briefing text and console message display                                  |

### New Files (Effects — Phase 5)

| File                           | Purpose                                                             |
| ------------------------------ | ------------------------------------------------------------------- |
| `src/effects/Explosion.ts`     | Explosion entity: position, size, color, lifetime, icosahedron mesh |
| `src/effects/ExplosionPool.ts` | Object pool for explosions                                          |
| `src/effects/SpaceJunk.ts`     | Motion reference polygons (subtle rotating shapes in background)    |

### New Files (Save System — Phase 9)

| File                       | Purpose                                                                  |
| -------------------------- | ------------------------------------------------------------------------ |
| `src/save/ISaveManager.ts` | ISaveManager interface: saveGame(entry), loadGame(filename), listSaves() |
| `src/save/SaveManager.ts`  | SP save/load: IndexedDB for web, file I/O for Electron                   |
| `src/save/SaveEntry.ts`    | Save data structure: mission, ship state, score, timestamp               |

### New Files (Network Layer — Phase 10)

| File                              | Purpose                                                                     |
| --------------------------------- | --------------------------------------------------------------------------- |
| `src/network/INetworkLayer.ts`    | INetworkLayer interface: connect(), disconnect(), sendPacket(), onMessage() |
| `src/network/SPNetworkLayer.ts`   | No-op network layer for single-player (returns immediately)                 |
| `src/network/Packet.ts`           | Packet type definitions and serialization helpers                           |
| `src/network/SteamNetwork.ts`     | Steam SDR network layer (Phase 10+)                                         |
| `src/network/WebSocketNetwork.ts` | WebSocket network layer for browser clients (Phase 10+)                     |

### New Files (Game Orchestration — Phase 8)

| File                           | Purpose                                                                                  |
| ------------------------------ | ---------------------------------------------------------------------------------------- |
| `src/game/IGame.ts`            | Core game interface: init(), update(deltaT), render(), getState()                        |
| `src/game/GameState.ts`        | GameState enum: MENU, BRIEFING, PLAYING, PAUSED, DEAD1, DEAD2, VICTORY, DEFEAT, LOADGAME |
| `src/game/SinglePlayerGame.ts` | Single-player game orchestrator: wires all systems together                              |
| `src/game/NetworkedGame.ts`    | Networked game client (Phase 10+)                                                        |
| `src/game/GameFactory.ts`      | Creates correct game mode based on GameMode enum                                         |

### New Files (XR — Phase 1.7, 12)

| File                          | Purpose                                                                |
| ----------------------------- | ---------------------------------------------------------------------- |
| `src/xr/WebXRInline.ts`       | Inline XR session manager: enterInlineXR(), exitInlineXR() — Phase 1.7 |
| `src/xr/WebXRManager.ts`      | Full immersive VR session manager: enterXR(), exitXR() — Phase 12      |
| `src/xr/XRCameraSwap.ts`      | Standard ↔ XR camera swap — Phase 12                                   |
| `src/xr/VRControllerInput.ts` | VR controller input mapping — Phase 12                                 |

### Prototype Files (Phase 1.0)

| File                          | Purpose                                                                   |
| ----------------------------- | ------------------------------------------------------------------------- |
| `src/prototype/QuickScene.ts` | Quick visual prototype: sun, colored planets, ship wireframe, star points |

### Entry Points

| File           | Purpose                                                                     |
| -------------- | --------------------------------------------------------------------------- |
| `src/index.ts` | Web entry point: boot Babylon.js scene, initialize game loop                |
| `src/main.ts`  | Electron/desktop entry point: same as index.ts but with Electron IPC bridge |

### Data Directory

| Path                          | Purpose                                                                         |
| ----------------------------- | ------------------------------------------------------------------------------- |
| `data/missions/original/`     | Original Orbit missions in JSON format (initially: train01.json, combat01.json) |
| `data/missions/second-stage/` | Second Stage missions directory (empty initially, populated later)              |
| `data/models/placeholder/`    | Placeholder models: ship-box.glb, station-box.glb (swapped for real GLTF later) |
| `data/textures/placeholder/`  | Placeholder textures: solid colors for planets                                  |

### Test Files (mirrors src/ structure)

| File                                 | Purpose                                                                                |
| ------------------------------------ | -------------------------------------------------------------------------------------- |
| `tests/core/timer.test.ts`           | Timer: advance, getDeltaT, reset, interpolation factor                                 |
| `tests/core/event-bus.test.ts`       | EventBus: subscribe, emit, unsubscribe, one-time listeners                             |
| `tests/core/rng.test.ts`             | RNG: seed determinism, range bounds, statistical distribution                          |
| `tests/core/logger.test.ts`          | Logger: log levels, output formatting                                                  |
| `tests/core/game-mode.test.ts`       | GameMode: getActiveMode, setMode, mission path resolution per mode                     |
| `tests/player/flight-model.test.ts`  | FlightModel: delta-v accumulation, warp speed, gravity                                 |
| `tests/player/ship.test.ts`          | Ship: state transitions, shield regen, death timer                                     |
| `tests/planets/starfield.test.ts`    | Starfield: star count, brightness mapping, culling                                     |
| `tests/missions/parser.test.ts`      | MissionParser: JSON parsing, validation, missing fields                                |
| `tests/weapons/weapon-defs.test.ts`  | WeaponDef: default definitions, speed/yield calculations                               |
| `tests/weapons/missile-pool.test.ts` | MissilePool: acquire/release, pool exhaustion                                          |
| `tests/targets/strategy.test.ts`     | Strategy: enum values, registry lookup                                                 |
| `tests/targets/turn-toward.test.ts`  | TurnToward: rotation toward point, dot product checks                                  |
| `tests/targets/move-toward.test.ts`  | MoveToward: distance thresholds, forward/backward movement                             |
| `tests/targets/find-enemy.test.ts`   | FindEnemy: closest target by squared distance, faction filtering                       |
| `tests/effects/explosion.test.ts`    | Explosion: creation, lifetime, size range                                              |
| `tests/save/save-manager.test.ts`    | SaveManager: save/load cycle, IndexedDB fallback                                       |
| `tests/xr/webxr-inline.test.ts`      | WebXRInline: session start/stop, WebXR unsupported fallback                            |

---

## 7. Functions

### Core Library Functions

```typescript
// Babylon.js math: BABYLON.Vector3 and BABYLON.Matrix used directly — no wrapper functions needed.

// src/core/Timer.ts
function createTimer(fixedDeltaT: number): Timer;
function timerTick(timer: Timer, now: number): void;
function timerGetDeltaT(timer: Timer): number;
function timerGetAccumulator(timer: Timer): number;
function timerReset(timer: Timer): void;

// src/core/EventBus.ts
function createEventBus<TEvents extends Record<string, unknown>>(): EventBus<TEvents>;
function eventBusSubscribe<T>(bus: EventBus<unknown>, event: string, handler: EventHandler<T>): void;
function eventBusEmit<T>(bus: EventBus<unknown>, event: string, data: T): void;
function eventBusUnsubscribe<T>(bus: EventBus<unknown>, event: string, handler: EventHandler<T>): void;

// src/core/RNG.ts
function createRNG(seed: number): RNG;
function rngSeed(rng: RNG, seed: number): void;
function rngNext(rng: RNG): number;
function rngNextInt(rng: RNG, min: number, max: number): number;
function rngNextFloat(rng: RNG, min: number, max: number): number;

// src/core/Logger.ts
function createLogger(minLevel: LogLevel): Logger;
function loggerDebug(logger: Logger, message: string, context?: Record<string, unknown>): void;
function loggerInfo(logger: Logger, message: string, context?: Record<string, unknown>): void;
function loggerWarn(logger: Logger, message: string, context?: Record<string, unknown>): void;
function loggerError(logger: Logger, message: string, context?: Record<string, unknown>): void;
function loggerSetLevel(logger: Logger, level: LogLevel): void;

// src/core/game-mode.ts
function getActiveMode(): GameMode;
function setMode(mode: GameMode): void;
function getMissionPath(mode: GameMode): string; // returns 'data/missions/original/' or 'data/missions/second-stage/'
```

### Player System Functions

```typescript
// src/player/Ship.ts
function createShip(): ShipState;
function shipUpdate(ship: ShipState, deltaT: number, input: IInputManager, flightModel: IFlightModel): void;
function shipTakeDamage(ship: ShipState, damage: number): boolean; // returns true if died
function shipRegenerateShields(ship: ShipState, deltaT: number): void;
function shipReset(ship: ShipState): void;

// src/player/FlightModel.ts
function createNewtonianFlightModel(config: FlightModelConfig): NewtonianFlightModel;
function createArcadeFlightModel(config: FlightModelConfig): ArcadeFlightModel;
function flightModelUpdate(model: IFlightModel, ship: ShipState, deltaT: number, input: IInputManager): void;
function flightModelApplyGravity(model: IFlightModel, ship: ShipState, gravity: Vector3): void;

// src/player/Camera.ts
function createCameraWrapper(scene: BABYLON.Scene, ship: ShipState): CameraWrapper;
function cameraUpdate(cam: CameraWrapper, deltaT: number, input: IInputManager): void;
function cameraSetViewLock(cam: CameraWrapper, lockMode: LockMode, lockedTarget: TargetState | null): void;

// src/player/InputManager.ts
function createKeyboardInput(): KeyboardInput;
function createJoystickInput(): JoystickInput;
function inputGetKeyState(input: IInputManager, key: string): number; // bit7=down, bit0=pressed-this-frame
function inputGetAxis(input: IInputManager, axis: string): number; // -1 to +1
function inputIsKeyPressed(input: IInputManager, key: string): boolean;
function inputIsKeyDown(input: IInputManager, key: string): boolean;
```

### Solar System Functions

```typescript
// src/planets/Planet.ts
function createPlanet(data: PlanetData): Planet;
function planetUpdate(planet: Planet, deltaT: number): void; // advances theta, updates position
function planetGetGravity(planet: Planet, position: Vector3): Vector3 | null;

// src/planets/SolarSystem.ts
function createSolarSystem(planets: Planet[]): SolarSystem;
function solarSystemUpdate(sys: SolarSystem, deltaT: number): void;
function solarSystemGetGravity(sys: SolarSystem, position: Vector3): Vector3;
function solarSystemGetPlanetAtPosition(sys: SolarSystem, position: Vector3): Planet | null;

// src/planets/Starfield.ts
function createStarfield(scene: BABYLON.Scene, count: number, density: 'sparse' | 'dense'): Starfield;
function starfieldUpdateCamera(sf: Starfield, camera: BABYLON.Camera): void;
```

### Mission System Functions

```typescript
// src/missions/MissionParser.ts
function parseMission(json: Record<string, unknown>): MissionFile;
function validateMission(mission: MissionFile): string[]; // returns error messages, empty if valid
function resolveCursorPosition(cursors: CursorDef[], planet: Planet | null): Vector3;

// src/missions/MissionLoader.ts
function loadMission(url: string): Promise<MissionFile>; // fetch JSON from URL (web) or Electron IPC path
function loadMissionFromFile(path: string): Promise<MissionFile>; // Electron-only file I/O
```

### Weapons Functions

```typescript
// src/weapons/WeaponDef.ts
function getDefaultWeaponDefs(): WeaponDef[]; // returns Laser, PhotonRay, IonGun, Disruptor, Spare

// src/weapons/Missile.ts
function createMissile(owner: ShipState | TargetState, weaponDef: WeaponDef, position: Vector3, direction: Vector3, parentVelocity: Vector3): Missile;
function missileUpdate(missile: Missile, deltaT: number): boolean; // returns true if expired

// src/weapons/MissilePool.ts
function createMissilePool(size: number): MissilePool;
function missilePoolAcquire(pool: MissilePool): Missile | null; // null if pool exhausted (oldest reclaimed)
function missilePoolRelease(pool: MissilePool, missile: Missile): void;
function missilePoolUpdateAll(pool: MissilePool, deltaT: number): void;

// src/weapons/SinglePlayerWeaponSystem.ts
function weaponSystemFire(weaponSys: SinglePlayerWeaponSystem, shooter: ShipState | TargetState, weaponIndex: number): void;
function weaponSystemUpdate(weaponSys: SinglePlayerWeaponSystem, deltaT: number): void;
function weaponSystemCheckCollisions(weaponSys: SinglePlayerWeaponSystem, targets: TargetState[], planets: Planet[]): void;
```

### AI Functions

```typescript
// src/targets/TurnToward.ts
function turnToward(target: TargetState, desiredPosition: Vector3, deltaT: number): void;

// src/targets/MoveToward.ts
function moveToward(target: TargetState, desiredPosition: Vector3, deltaT: number): void;

// src/targets/FindEnemy.ts
function findEnemy(targets: TargetState[], myFaction: boolean): TargetState | null;

// src/targets/Aim.ts
function aimAtPlayer(target: TargetState, player: ShipState, missileSpeed: number): Vector3;

// src/targets/SinglePlayerAIManager.ts
function aiManagerUpdate(ai: SinglePlayerAIManager, deltaT: number): void;
function aiManagerFireMissiles(ai: SinglePlayerAIManager, weaponPool: MissilePool): void;
```

### HUD Functions

```typescript
// src/hud/Radar.ts
function radarRender(radar: Radar, targets: TargetState[], player: ShipState): void;

// src/hud/ShieldBar.ts
function shieldBarUpdate(bar: ShieldBar, ship: ShipState): void;

// src/hud/MessageOverlay.ts
function messageOverlayShow(msg: MessageOverlay, text: string, duration?: number): void;
function messageOverlayClear(msg: MessageOverlay): void;
```

### Effect Functions

```typescript
// src/effects/Explosion.ts
function createExplosion(position: Vector3, size: number, color: [number, number, number]): Explosion;
function explosionUpdate(exp: Explosion, deltaT: number): boolean;

// src/effects/ExplosionPool.ts
function createExplosionPool(size: number): ExplosionPool;
function explosionPoolAcquire(pool: ExplosionPool): Explosion | null;
function explosionPoolRelease(pool: ExplosionPool, exp: Explosion): void;
function explosionPoolUpdateAll(pool: ExplosionPool, deltaT: number): void;
```

### Game Orchestration Functions

```typescript
// src/game/SinglePlayerGame.ts
function createSinglePlayerGame(mode: GameMode): SinglePlayerGame;
function gameInit(game: SinglePlayerGame, mission: MissionFile): void;
function gameUpdate(game: SinglePlayerGame, deltaT: number): void;
function gameRender(game: SinglePlayerGame): void;
function gameGetState(game: SinglePlayerGame): GameState;
function gameTransitionState(game: SinglePlayerGame, newState: GameState): void;

// src/game/GameFactory.ts
function createGame(mode: GameMode): IGame; // returns SinglePlayerGame or NetworkedGame
```

### Prototype Functions (Phase 1.0)

```typescript
// src/prototype/QuickScene.ts
function createQuickScene(scene: BABYLON.Scene): void;
// Creates: sun sphere, 8 colored planet spheres, ship wireframe box, 500 star points, point light at sun
```

### XR Functions (Phase 1.7)

```typescript
// src/xr/WebXRInline.ts
function createWebXRInline(scene: BABYLON.Scene): WebXRInline;
function webXREnterInline(xr: WebXRInline): Promise<void>;
function webXRExitInline(xr: WebXRInline): void;
function webXRIsSupported(): boolean; // checks navigator.xr availability
```

---

## 8. Classes

### Core Classes

```typescript
// Babylon.js math: BABYLON.Vector3 and BABYLON.Matrix used directly — no custom classes needed.

// src/core/Timer.ts
class Timer {
  private accumulator: number; private fixedDeltaT: number;
  constructor(fixedDeltaT: number);
  tick(now: number): void; getDeltaT(): number; getAccumulator(): number;
  getInterpolationFactor(): number; reset(): void;
}

// src/core/EventBus.ts
class EventBus<TEvents extends Record<string, unknown>> {
  private handlers: Map<keyof TEvents, Set<EventHandler<any>>>;
  subscribe<K extends keyof TEvents>(event: K, handler: EventHandler<TEvents[K]>): void;
  emit<K extends keyof TEvents>(event: K, data: TEvents[K]): void;
  unsubscribe<K extends keyof TEvents>(event: K, handler: EventHandler<TEvents[K]>): void;
}

// src/core/RNG.ts
class RNG {
  private state: number; // Mulberry32 internal state
  constructor(seed: number);
  seed(value: number): void; next(): number; nextInt(min: number, max: number): number; nextFloat(min: number, max: number): number;
}

// src/core/Logger.ts
class Logger {
  private minLevel: LogLevel;
  constructor(minLevel: LogLevel);
  debug(message: string, context?: Record<string, unknown>): void;
  info(message: string, context?: Record<string, unknown>): void;
  warn(message: string, context?: Record<string, unknown>): void;
  error(message: string, context?: Record<string, unknown>): void;
  setLevel(level: LogLevel): void;
}
```

### Player Classes

```typescript
// src/player/Ship.ts
class Ship {
  position: Vector3; velocity: Vector3; viewDir: Vector3; upDir: Vector3; rightDir: Vector3;
  shields: number; maxShields: number; score: number; selectedWeapon: number;
  flightModel: 'newtonian' | 'arcade'; state: ShipState; deathTimer: number;
  constructor();
  update(deltaT: number, input: IInputManager, flightModel: IFlightModel): void;
  takeDamage(damage: number): boolean; // returns true if died (shields < 0)
  regenerateShields(deltaT: number): void; reset(): void;
}

// src/player/FlightModel.ts
interface IFlightModel {
  type: 'newtonian' | 'arcade';
  update(ship: Ship, deltaT: number, input: IInputManager): void;
  applyGravity(ship: Ship, gravity: Vector3): void;
}
class NewtonianFlightModel implements IFlightModel { /* ... */ }
class ArcadeFlightModel implements IFlightModel { /* ... */ }

// src/player/Camera.ts
class CameraWrapper {
  private camera: BABYLON.ArcRotateCamera; private ship: Ship;
  constructor(scene: BABYLON.Scene, ship: Ship);
  update(deltaT: number, input: IInputManager): void;
  setViewLock(lockMode: LockMode, lockedTarget: TargetState | null): void;
}

// src/player/InputManager.ts
interface IInputManager {
  getKeyState(key: string): number; // bit7=down, bit0=pressed-this-frame
  getAxis(axis: string): number; // -1 to +1 analog value
  isKeyPressed(key: string): boolean; isKeyDown(key: string): boolean;
}
class KeyboardInput implements IInputManager { /* ... */ }
class JoystickInput implements IInputManager { /* ... */ }
```

### Solar System Classes

```typescript
// src/planets/Planet.ts
class Planet {
  name: string; position: Vector3; velocity: Vector3; upDir: Vector3; viewDir: Vector3; rightDir: Vector3;
  radius: number; radiusSquared: number; mass: number; color: [number, number, number];
  distanceFromPrimary: number; angularVelocity: number; theta: number; oblicity: number;
  isMoon: boolean; primaryIndex: number; texturePath: string; customTexture: boolean; hidden: boolean;
  mesh?: BABYLON.Mesh;
  constructor(data: PlanetData);
  update(deltaT: number): void; getGravity(position: Vector3): Vector3 | null;
}

// src/planets/SolarSystem.ts
class SolarSystem {
  private planets: Planet[];
  constructor(planets: Planet[]);
  update(deltaT: number): void;
  getGravity(position: Vector3): Vector3; // sum of all planet gravities
  getPlanetAtPosition(position: Vector3): Planet | null; // collision check
}

// src/planets/Starfield.ts
class Starfield {
  private particles: BABYLON.ParticleSystem | BABYLON.PointsSystem;
  constructor(scene: BABYLON.Scene, count: number, density: 'sparse' | 'dense');
  updateCamera(camera: BABYLON.Camera): void; // parallax update
}
```

### Mission Classes

```typescript
// src/missions/MissionParser.ts
class MissionParser {
  static parse(json: Record<string, unknown>): MissionFile;
  static validate(mission: MissionFile): string[];
  static resolveCursorPosition(cursors: CursorDef[], planetAtCursor: Planet | null): Vector3;
}

// src/missions/MissionLoader.ts
class MissionLoader {
  static async loadFromURL(url: string): Promise<MissionFile>; // fetch + parse JSON
  static async loadFromFile(path: string): Promise<MissionFile>; // Electron IPC → file read
}
```

### Weapon Classes

```typescript
// src/weapons/Missile.ts
class Missile {
  position: Vector3; velocity: Vector3; weaponDef: WeaponDef;
  owner: ShipState | TargetState; lifetime: number;
  constructor(owner: ShipState | TargetState, weaponDef: WeaponDef, position: Vector3, direction: Vector3, parentVelocity: Vector3);
  update(deltaT: number): boolean; // returns true if expired
}

// src/weapons/MissilePool.ts
class MissilePool {
  private pool: Missile[]; private active: Set<Missile>;
  constructor(size: number);
  acquire(): Missile | null; release(missile: Missile): void; updateAll(deltaT: number): void;
}

// src/weapons/SinglePlayerWeaponSystem.ts
class SinglePlayerWeaponSystem {
  private missilePool: MissilePool; private activeMissiles: Set<Missile>; private weaponDefs: WeaponDef[];
  constructor(missilePoolSize: number, weaponDefs: WeaponDef[]);
  fireMissile(shooter: Ship | TargetState, weaponIndex: number): void;
  update(deltaT: number): void; checkCollisions(targets: TargetState[], planets: Planet[]): void;
}

// src/weapons/NetworkedWeaponSystem.ts (stub)
class NetworkedWeaponSystem implements IWeaponSystem { /* delegates to SP impl, sends packets */ }
```

### AI Classes

```typescript
// src/targets/Target.ts
class Target {
  name: string; position: Vector3; velocity: Vector3; viewDir: Vector3; upDir: Vector3; rightDir: Vector3;
  modelBounds: { lobound: Vector3; hibound: Vector3 }; strategy: AIFStrategy; friendly: boolean;
  shields: number; maxShields: number; shieldRegen: number; weaponIndex: number;
  turnRate: number; maxSpeed: number; score: number; hidden: boolean; invisible: boolean; age: number;
  mesh?: BABYLON.Mesh;
}

// src/targets/SinglePlayerAIManager.ts
class SinglePlayerAIManager {
  private targets: Target[]; private weaponPool: MissilePool;
  constructor(targets: Target[], weaponPool: MissilePool);
  update(deltaT: number): void; fireMissiles(): void;
}

// src/targets/NetworkedAIManager.ts (stub)
class NetworkedAIManager implements ITargetManager { /* delegates to SP impl */ }
```

### HUD Classes

```typescript
// src/hud/Radar.ts
class Radar { private canvas: HTMLCanvasElement; render(targets: TargetState[], player: ShipState): void; }

// src/hud/ShieldBar.ts
class ShieldBar { private element: HTMLElement; update(ship: ShipState): void; }

// src/hud/MessageOverlay.ts
class MessageOverlay {
  private messages: Array<{ text: string; timestamp: number; duration: number }>;
  show(text: string, duration?: number): void; clear(): void; render(): void;
}
```

### Effect Classes

```typescript
// src/effects/Explosion.ts
class Explosion {
  position: Vector3; size: number; color: [number, number, number]; lifetime: number; elapsed: number;
  mesh?: BABYLON.Mesh; light?: BABYLON.PointLight;
}

// src/effects/ExplosionPool.ts
class ExplosionPool {
  private pool: Explosion[]; private active: Set<Explosion>;
  constructor(size: number);
  acquire(position: Vector3, size: number, color: [number, number, number]): Explosion | null;
  release(explosion: Explosion): void; updateAll(deltaT: number): void;
}
```

### Save Classes

```typescript
// src/save/SaveManager.ts
class SaveManager {
  private storage: Storage; // IndexedDB for web, Electron IPC for native
  async saveGame(filename: string, entry: SaveEntry): Promise<void>;
  async loadGame(filename: string): Promise<SaveEntry | null>;
  async listSaves(): Promise<string[]>;
}

// src/save/SaveEntry.ts
class SaveEntry { mode: GameMode; mission: string; ship: ShipStateData; score: number; timestamp: number; }
```

### Network Classes (Phase 10)

```typescript
// src/network/SPNetworkLayer.ts (stub)
class SPNetworkLayer implements INetworkLayer {
  connect(): Promise<void>; disconnect(): void; sendPacket(packet: NetworkPacket): void; onMessage(callback: (packet: NetworkPacket) => void): void;
}

// src/network/SteamNetwork.ts (Phase 10+) — class SteamNetwork implements INetworkLayer { /* ... */ }
// src/network/WebSocketNetwork.ts (Phase 10+) — class WebSocketNetwork implements INetworkLayer { /* ... */ }
```

### Game Orchestration Classes

```typescript
// src/game/SinglePlayerGame.ts
class SinglePlayerGame implements IGame {
  private state: GameState; private ship: Ship; private flightModel: IFlightModel;
  private solarSystem: SolarSystem; private aiManager: SinglePlayerAIManager;
  private weaponSystem: SinglePlayerWeaponSystem; private hud: SinglePlayerHUD;
  private explosionPool: ExplosionPool; private mission: MissionFile; private mode: GameMode;
  constructor(mode: GameMode);
  init(mission: MissionFile): void; update(deltaT: number): void; render(): void;
  getState(): GameState; transitionState(newState: GameState): void;
}

// src/game/GameFactory.ts
class GameFactory { static create(mode: GameMode): IGame; }
```

### XR Classes

```typescript
// src/xr/WebXRInline.ts (Phase 1.7)
class WebXRInline {
  private scene: BABYLON.Scene; private xrManager: BABYLON.WebXRManager | null;
  constructor(scene: BABYLON.Scene);
  static isSupported(): boolean; async enterInline(): Promise<void>; exitInline(): void;
}

// src/xr/WebXRManager.ts (Phase 12)
class WebXRManager {
  private scene: BABYLON.Scene; private xrManager: BABYLON.WebXRManager;
  private standardCamera: BABYLON.Camera;
  constructor(scene: BABYLON.Scene, standardCamera: BABYLON.Camera);
  async enterVR(): Promise<void>; exitVR(): void;
}

// src/xr/XRCameraSwap.ts (Phase 12)
class XRCameraSwap {
  private scene: BABYLON.Scene; private standardCamera: BABYLON.Camera; private xrCamera: BABYLON.WebXRCamera;
  swapToXR(): void; swapToStandard(): void;
}

// src/xr/VRControllerInput.ts (Phase 12)
class VRControllerInput implements IInputManager { /* maps XR controller inputs */ }
```

### Prototype Class (Phase 1.0)

```typescript
// src/prototype/QuickScene.ts
class QuickScene { static create(scene: BABYLON.Scene): void; }
// Creates: sun sphere (yellow, emissive), 8 colored planet spheres, ship wireframe box at origin,
// point light at sun, 500 star points, ArcRotateCamera
```

---

## 9. Constants, Definitions & Game Data

### Physics Constants

| Constant            | Value          | Description                |
| ------------------- | -------------- | -------------------------- |
| `THETA`             | 1.6 rad/s      | Max rotation rate          |
| `DELTAV`            | 0.2 units/s²   | Thrust acceleration        |
| `WARP_COEFF`        | 0.2            | Warp velocity coefficient  |
| `MAX_THROTTLE`      | 10,000 km/s    | Max throttle (arcade)      |
| `MAX_WARP_THROTTLE` | 1,000,000 km/s | Max warp throttle (arcade) |
| `SHIELD_REGEN`      | configurable   | Shield regeneration rate   |
| `DEAD_TIME`         | 5.0 s          | Pre-respawn countdown      |

### Weapon Definitions

| Index | Name      | Speed    | Yield | Idle | Lifetime | Renderer    | Color   |
| ----- | --------- | -------- | ----- | ---- | -------- | ----------- | ------- |
| 0     | Laser     | 4000/KMU | 20    | 0.2  | 1.0      | Cross       | (1,1,0) |
| 1     | PhotonRay | 5000/KMU | 30    | 0.3  | 0.75     | Ray         | (0,0,1) |
| 2     | IonGun    | 4000/KMU | 30    | 0.5  | 1.5      | Tetrahedron | (0,1,0) |
| 3     | Disruptor | 3000/KMU | 60    | 1.0  | 1.5      | Loop        | (1,1,0) |
| N+    | Spare     | 6000/KMU | 15    | 2.0  | 1.0      | None        | (1,0,0) |

### AI Strategies

| Strategy  | Description                                      |
| --------- | ------------------------------------------------ |
| DoNothing | Does nothing, passive                            |
| Sit1      | Turns toward player, shoots poorly               |
| Sit2      | Aims at future position, shoots poorly           |
| Sit3      | Finds nearest enemy, turns toward it             |
| Sit4      | Finds nearest enemy, predictive aim, shoots well |
| Hunt1     | Moves toward player, shoots                      |
| Hunt2     | Moves toward aiming point, shoots                |
| Hunt3     | Moves toward closest enemy, shoots poorly        |
| Hunt4     | Moves toward aiming point, shoots well           |

### Event Triggers

Approach, Depart, True, Score, Destroy, Alarm, Shields

### Event Actions

Message, Hide, Unhide, Destroy, Score, Enable, Disable, LoadMission, Boom, Flash

---

## 10. Dependencies

### NPM Packages

| Package                            | Version | Purpose                                  |
| ---------------------------------- | ------- | ---------------------------------------- |
| `babylonjs`                        | ^7.x    | Core 3D engine (full version, not Lite)  |
| `@babylonjs/core`                  | ^7.x    | Babylon.js core modules (tree-shakeable) |
| `@babylonjs/loaders`               | ^7.x    | GLTF/GLB model loader                    |
| `@babylonjs/gui`                   | ^7.x    | 2D/3D GUI for HUD elements               |
| `typescript`                       | ^5.x    | TypeScript compiler                      |
| `vite`                             | ^6.x    | Build tool + dev server                  |
| `vitest`                           | ^3.x    | Test runner                              |
| `@vitest/coverage-v8`              | ^3.x    | Coverage reporting                       |
| `electron`                         | ^33.x   | Desktop app wrapper                      |
| `@electron/remote`                 | ^2.x    | Electron renderer ↔ main process bridge  |
| `steamworks.js`                    | ^0.9.x  | Steam SDK via Rust N-API (Electron only) |
| `eslint`                           | ^9.x    | Linting                                  |
| `@typescript-eslint/eslint-plugin` | ^8.x    | TypeScript ESLint rules                  |
| `prettier`                         | ^3.x    | Code formatting                          |
| `typedoc`                          | ^0.27.x | API documentation generation             |

### Key Dependency Decisions

- **Babylon.js full** — WebXRManager requires the full module set; Lite strips it out
- **Electron** — mature `steamworks.js` integration, established IPC patterns for Babylon.js ↔ Steam SDK bridge
- **Tree-shaking via Vite** — only imported Babylon.js modules end up in the bundle
- **No Babylon.js dependency in Server.ts** — headless server is pure Node.js

---

## 11. Testing Strategy

### Test Categories

| Category              | Location                                               | Purpose                      | Coverage Target  |
| --------------------- | ------------------------------------------------------ | ---------------------------- | ---------------- |
| **Unit tests**        | `tests/core/`, `tests/player/`, `tests/weapons/`, etc. | Individual functions/classes | ≥90% all metrics |
| **Integration tests** | `tests/integration/`                                   | Multi-system workflows       | Key paths only   |

### Key Test Scenarios

| Test                              | Module                 | What It Verifies                                                                   |
| --------------------------------- | ---------------------- | ---------------------------------------------------------------------------------- |
| Newtonian delta-v accumulation    | `flight-model.test.ts` | Velocity persists across frames, warp coefficient applied correctly                |
| Arcade velocity set directly      | `flight-model.test.ts` | No momentum — velocity = throttle × direction                                      |
| Ship state machine transitions    | `ship.test.ts`         | NORMAL → DEAD1 (shields < 0) → DEAD2 (timer expires) → NORMAL (respawn)            |
| Shield regen rate                 | `ship.test.ts`         | Shields increase at SHIELD_REGEN per second, capped at maxShields                  |
| Take damage reduces shields       | `ship.test.ts`         | Damage applied correctly, death triggered when shields < 0                         |
| Mission JSON parsing              | `parser.test.ts`       | Valid mission parsed correctly, missing required fields rejected                   |
| Cursor position resolution        | `parser.test.ts`       | Named cursor resolves to planet position, offset applied correctly                 |
| AI turn toward target             | `turn-toward.test.ts`  | Target's viewDir rotates toward desired position at correct rate                   |
| Find closest enemy                | `find-enemy.test.ts`   | Returns closest non-friendly target, handles empty target list                     |
| Predictive aiming                 | `aim.test.ts`          | Accounts for missile speed and target velocity                                     |
| Missile collision with target     | `missile-pool.test.ts` | AABB check in target's local space                                                 |
| Explosion object pooling          | `explosion.test.ts`    | Pool reuse, lifetime expiry, size clamping                                         |
| Save/load round-trip              | `save-manager.test.ts` | Ship state preserved across save/load cycle                                        |
| WebXR support detection           | `webxr-inline.test.ts` | Returns false when navigator.xr undefined, true when available                     |
| Game mode mission path resolution | `game-mode.test.ts`    | ORIGINAL → 'data/missions/original/', SECOND_STAGE → 'data/missions/second-stage/' |

---

## 12. Implementation Order — Phases 0–12

### Phase 0 — Foundation (Week 1)

**Goal:** Project scaffolding, build system, testing infrastructure.
**Duration:** ~1-2 days
**Visual gratification:** None yet (pure infrastructure).

| #    | Task                                                                            | Deliverable                                                                             | Tests                                                          |
| ---- | ------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | -------------------------------------------------------------- |
| 0.1  | Initialize project: `package.json`, `tsconfig.json`, `vite.config.ts`           | Working `npm run dev`                                                                   | N/A                                                            |
| 0.2  | Create `vitest.config.ts` with coverage thresholds (90/85/90/90)                | `npm run test:coverage` passes                                                          | N/A                                                            |
| 0.3  | Set up ESLint + Prettier config                                                 | `npm run lint` + `npm run format` clean                                                 | N/A                                                            |
| 0.4  | Use `@babylonjs/core/Maths` for Vector3/Matrix4 (no custom implementations)       | `tests/core/babylon-math.test.ts` — verify BABYLON.Vector3 and BABYLON.Matrix work as expected in tests | ≥90% coverage                                                  |
| 0.5  | Implement `Timer` (fixed-timestep accumulator)                                  | `tests/core/timer.test.ts` — advance, getDeltaT, reset, interpolation factor            | ≥90% coverage                                                  |
| 0.6  | Implement `EventBus` (pub/sub with typed events)                                | `tests/core/event-bus.test.ts` — subscribe, emit, unsubscribe, one-time listeners       | ≥90% coverage                                                  |
| 0.7  | Implement `RNG` (Mulberry32 seeded PRNG)                                        | `tests/core/rng.test.ts` — seed determinism, range bounds, statistical distribution     | ≥90% coverage                                                  |
| 0.8  | Implement `Logger` (structured, log levels)                                     | `tests/core/logger.test.ts` — log levels, output formatting                             | ≥90% coverage                                                  |
| 0.9  | Initialize Electron app (`electron/`)                                           | `npm run electron:start` opens window                                                   | N/A                                                            |
| 0.10 | Define `GameMode` enum + mode selector infrastructure                           | `src/core/game-mode.ts` — Original / Second Stage toggle, save tracking                 | `tests/core/game-mode.test.ts`                                 |
| 0.11 | Create mission directory structure (`data/missions/original/`, `second-stage/`) | Mission loader resolves correct set based on active mode                                | `tests/core/mission-loader.test.ts` — path resolution per mode |

### Phase 1 — Eye Candy (Starfield + Ship) (Week 2)

**Goal:** Something visible and interactive on screen. Starfield rendering + ship model + basic camera control.
**Duration:** ~2-3 days
**Visual gratification:** 🌟🌟🌟 Stars, ship, camera orbit — immediate payoff!

| #   | Task                                                         | Deliverable                                      | Tests                                                                       |
| --- | ------------------------------------------------------------ | ------------------------------------------------ | --------------------------------------------------------------------------- |
| 1.0 | Quick visual prototype: sun, colored planets, ship wireframe | Recognizable solar system in browser             | N/A (visual inspection)                                                     |
| 1.1 | Initialize Babylon.js scene                                  | Scene renders black screen                       | N/A                                                                         |
| 1.2 | Implement `Starfield` (65K+ stars, two density levels)       | Visible starfield with depth sorting             | `tests/planets/starfield.test.ts` — star count, brightness mapping, culling |
| 1.3 | Create `Sun` mesh (glowing sphere, light source)             | Glowing sun at origin                            | N/A                                                                         |
| 1.4 | Load and render a placeholder ship model                     | Ship mesh visible in scene                       | N/A                                                                         |
| 1.5 | Implement `InputManager` (keyboard + joystick abstraction)   | `IInputManager` interface + `KeyboardInput` impl | `tests/player/input-manager.test.ts` — key state tracking, axis mapping     |
| 1.6 | Implement `Camera` wrapper around Babylon.js ArcRotateCamera | Camera responds to input                         | N/A                                                                         |
| 1.7 | Wire camera to ship orientation (view lock mode)             | Camera orbits ship with pitch/yaw                | N/A                                                                         |
| 1.8 | Add space junk reference polygons (motion cues)              | Subtle rotating polygons in background           | N/A                                                                         |

### Phase 2 — Flight Physics & Controls (Week 3)

**Goal:** Full ship movement with both Newtonian and Arcade flight models.
**Duration:** ~3-4 days
**Visual gratification:** 🌟🌟🌟🌟 Fly around the solar system!

| #   | Task                                                    | Deliverable                                     | Tests                                                                           |
| --- | ------------------------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------------------------- |
| 2.1 | Implement `FlightModel` base interface + Newtonian impl | `IFlightModel` + `NewtonianFlightModel`         | `tests/player/flight-model.test.ts` — delta-v accumulation, warp speed, gravity |
| 2.2 | Implement Arcade flight model                           | `ArcadeFlightModel`                             | Same test suite (polymorphic)                                                   |
| 2.3 | Implement `Ship` state machine                          | `Ship` class with position/velocity/orientation | `tests/player/ship.test.ts` — state transitions, shield regen, death timer      |
| 2.4 | Wire keyboard input to flight controls                  | All keys functional (WASDQEZX + arrows)         | `tests/player/input-mapping.test.ts` — key to axis mapping                      |
| 2.5 | Implement joystick input support                        | Joystick axes mapped to yaw/pitch/roll/throttle | N/A                                                                             |
| 2.6 | Implement gravitational physics engine                  | Gravity affects player, missiles, targets       | `tests/player/gravity.test.ts` — force calculation, singularity clamping        |
| 2.7 | Implement full stop behavior (Newtonian)                | SPACE key stops ship in Newtonian mode          | N/A                                                                             |
| 2.8 | Implement warp speed (both models)                      | Warp acceleration / superwarp (SP only)         | `tests/player/warp.test.ts` — warp coefficient, superwarp formula               |

### Phase 3 — Planets & Solar System (Week 4)

**Goal:** All planets, moons, rings, and orbital mechanics.
**Duration:** ~3-4 days
**Visual gratification:** 🌟🌟🌟🌟🌟 Full solar system!

| #   | Task                                                       | Deliverable                                 | Tests                                                                        |
| --- | ---------------------------------------------------------- | ------------------------------------------- | ---------------------------------------------------------------------------- |
| 3.1 | Define `Planet` data structure + initial solar system data | 30+ planets/moons with NASA-based positions | `tests/planets/solar-system.test.ts` — planet count, positions, masses       |
| 3.2 | Implement planet rendering (distance-based LOD)            | Textured spheres → points based on distance | N/A                                                                          |
| 3.3 | Implement planetary ring systems (4 rings)                 | Jupiter, Saturn, Uranus, Neptune rings      | `tests/planets/ring-system.test.ts` — inner/outer radius, texture bounds     |
| 3.4 | Implement orbital animation                                | Planets orbit in XZ plane around primary    | `tests/planets/orbital-mechanics.test.ts` — angular velocity, orbit position |
| 3.5 | Load planet textures from PNG/WebP files                   | Textured planets with correct colors        | N/A                                                                          |
| 3.6 | Implement planet name labels (billboarded text)            | Names appear above planets at distance      | N/A                                                                          |
| 3.7 | Implement planet hiding/unhiding (mission events)          | `HidePlanet` / `UnhidePlanet` actions work  | `tests/missions/action-processor.test.ts`                                    |

### Phase 4 — Mission System (Week 5)

**Goal:** Parse JSON mission files, build game state from them.
**Duration:** ~3-4 days
**Visual gratification:** 🌟🌟 Load a mission → see it render!

| #   | Task                              | Deliverable                                                                  | Tests                                                                      |
| --- | --------------------------------- | ---------------------------------------------------------------------------- | -------------------------------------------------------------------------- |
| 4.1 | Implement JSON mission parser     | `MissionParser.parse()`, validation, cursor resolution                       | `tests/missions/parser.test.ts` — JSON parsing, validation, missing fields |
| 4.2 | Implement mission loader          | URL fetch for web, file I/O for Electron                                     | Integration test: load `train01.json` → verify mission data                |
| 4.3 | Implement mission data structures | `MissionFile`, `CursorDef`, `ObjectDef`, `EventDef`, etc.                    | Same integration test                                                      |
| 4.4 | Initial missions                  | `data/missions/original/train01.json`, `combat01.json` (placeholder content) | N/A                                                                        |

### Phase 5 — Combat & Weapons (Week 6-7)

**Goal:** Fire weapons, missiles fly, targets take damage, explosions happen.
**Duration:** ~3-4 days
**Visual gratification:** 🌟🌟🌟🌟🌟💥 BOOM!

| #   | Task                                                       | Deliverable                                   | Tests                                                                               |
| --- | ---------------------------------------------------------- | --------------------------------------------- | ----------------------------------------------------------------------------------- |
| 5.1 | Define `WeaponDef` (all 5 weapons + spare)                 | Weapon constants from GDD                     | `tests/weapons/weapon-defs.test.ts` — speed, yield, idle, lifetime                  |
| 5.2 | Implement `Missile` entity                                 | Position, velocity, weapon type, lifetime     | `tests/weapons/missile.test.ts` — update, expire, hit detection                     |
| 5.3 | Implement `MissilePool` (object pool)                      | Recycle missiles, oldest-first eviction       | `tests/weapons/missile-pool.test.ts` — allocation, recycling, overflow              |
| 5.4 | Implement `SinglePlayerWeaponSystem`                       | Fire, update, collision detection             | `tests/weapons/single-player-weapon-system.test.ts` — fire cooldown, hit resolution |
| 5.5 | Implement AABB missile-target collision                    | Transform to target local space, check bounds | Same test — bounding box overlap                                                    |
| 5.6 | Implement missile-planet collision                         | Distance vs planet radius                     | Same test                                                                           |
| 5.7 | Implement `Explosion` (icosahedron-based)                  | Visual explosion + point light                | `tests/effects/explosion.test.ts` — vertex generation, color randomization, fade    |
| 5.8 | Implement `ExplosionPool`                                  | Object pool for explosions                    | `tests/effects/explosion-pool.test.ts`                                              |
| 5.9 | Implement weapon renderers (cross, ray, tetrahedron, loop) | Distinct visual per weapon type               | N/A                                                                                 |

### Phase 6 — AI Targets (Week 8-9)

**Goal:** Enemy and friendly targets with 9 AI strategies.
**Duration:** ~3-4 days
**Visual gratification:** 🌟🌟🌟 Targets that actually fight back!

| #   | Task                                                    | Deliverable                                   | Tests                                                                     |
| --- | ------------------------------------------------------- | --------------------------------------------- | ------------------------------------------------------------------------- |
| 6.1 | Implement `Target` entity                               | Position, velocity, strategy, weapon, shields | `tests/targets/target.test.ts` — update, state management                 |
| 6.2 | Implement `TurnToward` AI primitive                     | Rotate to face target direction               | `tests/targets/turn-toward.test.ts` — axis-angle rotation, theta clamping |
| 6.3 | Implement `MoveToward` AI primitive                     | Move toward target with distance thresholds   | `tests/targets/move-toward.test.ts` — THINK_CUTOFFA/B/C behavior          |
| 6.4 | Implement `FindEnemy` AI primitive                      | Closest-target search by faction              | `tests/targets/find-enemy.test.ts` — distance filtering, faction check    |
| 6.5 | Implement `Aim` (predictive aiming)                     | Lead target to future position                | `tests/targets/aim.test.ts` — time-to-impact calculation                  |
| 6.6 | Implement all 9 strategies (Sit1-4, Hunt1-4, DoNothing) | Strategy registry + dispatch                  | `tests/targets/strategy.test.ts` — each strategy's behavior               |
| 6.7 | Implement `SinglePlayerAIManager`                       | Manage all targets' AI                        | `tests/targets/single-player-ai-manager.test.ts`                          |
| 6.8 | Implement `NetworkedAIManager` interface                | Server-side AI (deferred to Phase 10)         | N/A (interface only)                                                      |

### Phase 7 — HUD & UI (Week 10)

**Goal:** All HUD elements rendered on screen.
**Duration:** ~2-3 days
**Visual gratification:** 🌟🌟🌟 Full game UI!

| #    | Task                                            | Deliverable                                    | Tests                                                          |
| ---- | ----------------------------------------------- | ---------------------------------------------- | -------------------------------------------------------------- |
| 7.1  | Implement `IRadar` (circular radar overlay)     | Targets, planets shown on radar                | `tests/hud/radar.test.ts` — position mapping, range ring       |
| 7.2  | Implement `ShieldBar` (player + target)         | Color-coded shield bars                        | `tests/hud/shield-bar.test.ts` — color thresholds (>66%, >33%) |
| 7.3  | Implement `Crosshair` (center bitmap)           | Crosshair rendered at screen center            | N/A                                                            |
| 7.4  | Implement `MessageOverlay` (briefing + console) | Orange briefing text, green scrolling messages | `tests/hud/message-overlay.test.ts` — auto-wrap, multi-line    |
| 7.5  | Implement velocity readout                      | Green/red directional display                  | N/A                                                            |
| 7.6  | Implement throttle bar                          | Vertical bar showing throttle level            | N/A                                                            |
| 7.7  | Implement target name + range display           | Shows locked target info                       | N/A                                                            |
| 7.8  | Implement waypoint info display                 | Waypoint number + distance                     | N/A                                                            |
| 7.9  | Implement weapon status display                 | Purple (ready) / gray (cooling)                | `tests/hud/weapon-display.test.ts`                             |
| 7.10 | Implement FPS counter (toggleable)              | Top-of-screen FPS display                      | N/A                                                            |

### Phase 8 — Events & Game Flow (Week 11)

**Goal:** Mission events trigger actions, game state transitions work.
**Duration:** ~2-3 days
**Visual gratification:** 🌟🌟🌟 Events fire, missions progress!

| #   | Task                                           | Deliverable                           | Tests                                                                                                                        |
| --- | ---------------------------------------------- | ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| 8.1 | Implement `EventEvaluator` (trigger checking)  | All trigger types evaluated per frame | `tests/missions/event-evaluator.test.ts` — Approach, Depart, True, Score, Destroy, Alarm                                     |
| 8.2 | Implement `ActionProcessor` (action execution) | All action types executed             | `tests/missions/action-processor.test.ts` — Message, Hide, Unhide, Destroy, Score, Enable, Disable, LoadMission, Boom, Flash |
| 8.3 | Implement game state machine                   | `GameState` enum with transitions     | `tests/game/game-state.test.ts` — valid transitions only                                                                     |
| 8.4 | Implement death/respawn sequence               | STATE_DEAD1 → STATE_DEAD2 → respawn   | `tests/game/death-respawn.test.ts` — timer countdown, explosion                                                              |
| 8.5 | Implement mission chain progression            | LoadMission action chains missions    | Integration test: train01 → train02                                                                                          |
| 8.6 | Implement briefing display on mission load     | Orange text auto-wraps                | N/A                                                                                                                          |
| 8.7 | Implement victory/defeat screens               | win.msn / lose.msn                    | `tests/game/end-states.test.ts`                                                                                              |

### Phase 9 — Save System & Polish (Week 12)

**Goal:** Save/load missions, polish visuals and controls.
**Duration:** ~2-3 days
**Visual gratification:** 🌟🌟 Save your progress!

| #   | Task                                 | Deliverable                                 | Tests                                                              |
| --- | ------------------------------------ | ------------------------------------------- | ------------------------------------------------------------------ |
| 9.1 | Implement `SaveManager` (SP)         | File I/O (native) / IndexedDB (web)         | `tests/save/local-save-manager.test.ts` — write, read, max entries |
| 9.2 | Enforce local/server save separation | No cross-sync mechanism                     | `tests/save/separation.test.ts` — upload/download rejected         |
| 9.3 | Implement save menu UI               | Numbered list with mission name + timestamp | N/A                                                                |
| 9.4 | Implement screen flash effects       | Red on damage, white on events              | N/A                                                                |
| 9.5 | Add sound effects integration        | Phaser, explosion, comm sounds              | `tests/effects/sound-manager.test.ts`                              |

### Phase 10 — Multiplayer Layer (Week 13-15)

**Goal:** Networked multiplayer with headless server support.
**Duration:** ~5-7 days
**Visual gratification:** 🌟🌟🌟🌟 Multiplayer!

| #    | Task                                             | Deliverable                            | Tests                                                                  |
| ---- | ------------------------------------------------ | -------------------------------------- | ---------------------------------------------------------------------- |
| 10.1 | Define `INetworkLayer` interface                 | All network operations abstracted      | `tests/network/packet.test.ts` — packet format, serialization          |
| 10.2 | Implement `SPNetworkLayer` (no-op)               | SP mode unaffected by network layer    | N/A                                                                    |
| 10.3 | Implement binary packet protocol                 | Magic byte 0x56, size-prefixed packets | `tests/network/packet.test.ts` — all packet types                      |
| 10.4 | Implement `SteamNetwork.ts` (native client)      | Steam SDR via steamworks.js            | `tests/network/steam-network.test.ts` — connect, send/receive packets  |
| 10.5 | Implement `WebSocketNetwork.ts` (browser client) | wss:// connection to relay server      | `tests/network/websocket-network.test.ts` — connect, message handling  |
| 10.6 | Implement position sync protocol                 | Urgent/regular/long position reports   | `tests/network/position-sync.test.ts` — update frequency, lag handling |
| 10.7 | Implement client management                      | Join/leave, slot management, roll call | `tests/network/client-management.test.ts`                              |

### Phase 11 — Electron Production Build (Week 16)

**Goal:** Production-ready native builds for macOS, Windows, Linux.
**Duration:** ~2-3 days
**Visual gratification:** 🌟🌟🌟🌟🌟 Ship it!

| #     | Task                                        | Deliverable                                     | Tests                                  |
| ----- | ------------------------------------------- | ----------------------------------------------- | -------------------------------------- |
| 11.1  | Configure Electron production build         | `npm run build:electron` produces distributable | N/A                                    |
| 11.3  | Implement native file I/O bridge (Electron) | Mission loading from disk                       | `tests/electron/file-bridge.test.ts`   |
| 11.5  | Add platform-specific audio handling        | Native audio API fallbacks                      | N/A                                    |
| 11.6  | Configure code signing (macOS)              | notarized builds                                | N/A                                    |
| 11.7  | Configure Windows installer                 | NSIS installer                                  | N/A                                    |
| 11.8  | Configure Linux AppImage/DEB                | AppImage + DEB output                           | N/A                                    |
| 11.9  | Implement Steam overlay integration         | In-game Steam overlay (native builds)           | `tests/electron/steam-overlay.test.ts` |
| 11.10 | Implement Steam lobby system                | Create/join lobbies via steamworks.js           | `tests/network/steam-lobby.test.ts`    |

### Phase 12 — WebXR / VR Support (Stretch Goal)

**Goal:** Play Orbit: Second Stage in a VR headset with full 360° visibility and VR controller flight controls.
**Duration:** ~3-4 days (stretch goal, after core gameplay solid)
**Visual gratification:** 🌟🌟🌟🌟🌟🌟 Put on the headset and fly the Defiant!

| #     | Task                                           | Deliverable                                     | Tests                                                  |
| ----- | ---------------------------------------------- | ----------------------------------------------- | ------------------------------------------------------ |
| 12.1  | Integrate `WebXRManager` in Babylon.js scene   | XR session entry point, HMD tracking active     | `tests/xr/webxr-init.test.ts` — session start/stop     |
| 12.2  | Implement `XRCameraSwap`                       | HMD replaces standard camera                    | `tests/xr/camera-swap.test.ts` — XR vs standard modes  |
| 12.3  | Map VR controller inputs to flight controls    | Thumbstick → yaw/pitch, trigger → thrust/fire   | `tests/xr/controller-mapping.test.ts` — all inputs     |
| 12.4  | Implement VR flight model (rotation-only)      | No artificial locomotion; only rotate + warp    | Same as Phase 2 tests (polymorphic)                    |
| 12.5  | Create XR DOM Overlay HUD panels               | 3D floating UI panels in headset                | `tests/xr/hud-panels.test.ts` — panel rendering, scale |
| 12.6  | Implement pointer-based HUD interaction        | Controller ray casts to select buttons          | `tests/xr/hud-interaction.test.ts` — hitbox detection  |
| 12.7  | Compress solar system scale for VR readability | Proportional distances, readable at headset FOV | `tests/xr/scale-compression.test.ts`                   |
| 12.8  | Implement VR comfort features                  | Vignette, motion sickness prevention            | `tests/xr/comfort.test.ts` — vignette toggle           |
| 12.9  | Add XR session lifecycle management            | Enter/exit XR gracefully, state preservation    | `tests/xr/session-lifecycle.test.ts`                   |
| 12.10 | Implement multi-headset support                | Oculus Touch, Valve Index, Vive, Quest          | Manual test matrix per headset                         |

---

## 13. Risk Register

| Risk                                 | Impact                                        | Mitigation                                                                             |
| ------------------------------------ | --------------------------------------------- | -------------------------------------------------------------------------------------- |
| **Mission JSON Z→Y conversion errors** | All positions/orientations wrong in-game | Dedicated JSON generation script with known-value test cases; verify with visual inspection |
| **Mission file parser complexity**   | Can't load original missions                  | Start with simplest mission (train01), add features incrementally                      |
| **GLTF model loading**               | Ships/stations don't render                   | Test GLTF loader early in Phase 1; use Babylon.js's built-in GLTF loader               |
| **Performance with 65K stars**       | FPS drops                                     | Distance culling, LOD at 200² and 5000² thresholds                                     |
| **Electron build conflicts**         | Can't test native builds                      | Keep Electron config minimal; merge Vite output into both                              |
| **Network code complexity**          | Multiplayer takes too long                    | Phase 10 is last; start with interfaces only, implement later                          |
| **Headless server in Node**          | Babylon.js dependency leak                    | Server.ts has zero Babylon.js imports; pure data processing                            |
| **steamworks.js compatibility**      | Steam SDK won't load in Electron              | Test early in Phase 0; have fallback to manual Steam auth via Web API                  |
| **Relay server complexity**          | Dual transport too hard                       | Start with WebSocket-only, add Steam SDR as optional module                            |
| **OpenID auth flow issues**          | Browser players can't verify                  | Fallback: simple password + SteamID64 check; upgrade to full OpenID later              |
| **Steam Web API rate limits**        | Server discovery fails                        | Cache server list with TTL; retry with exponential backoff                             |
| **WebXRManager integration**         | XR session crashes on some headsets           | Test early with Quest browser; fallback to standard camera for non-XR                  |
| **Motion sickness in VR**            | Players can't tolerate VR                     | No artificial locomotion; vignette during warp; comfort settings toggle                |
| **VR controller mapping complexity** | Different headsets have different controllers | Abstract input layer; test on Oculus Touch, Index, Vive simultaneously                 |

---

## 14. Phase Dependencies

```txt
Phase 0 (Foundation) → Phase 1 (Eye Candy) → Phase 2 (Flight Physics)
                                                            ↓
                                                    Phase 3 (Planets)
                                                            ↓
Phase 4 (Missions) ──┐                                    Phase 5 (Combat)
Phase 7 (HUD)   ─────┤                                     ↓
                     └──→ Phase 8 (Events & Flow) ←────── Phase 6 (AI)
                              ↓
                        Phase 9 (Save & Polish)
                              ↓
                       Phase 10 (Multiplayer)
                              ↓
                      Phase 11 (Native Builds)
                              ↓
                     Phase 12 (WebXR / VR)
```

**Parallelizable paths:**

- Phases 1, 4, 7 can run in parallel after Phase 0
- Phase 5 depends on Phase 2
- Phase 6 depends on Phase 2
- Phase 8 depends on Phases 4, 5, 6, 7
- Phase 9 depends on Phase 8
- Phase 10 depends on Phase 8 (interfaces can be designed earlier)
- Phase 11 depends on all previous phases

**MVP scope (single player, web-only):** Phases 0–9
**Full scope:** All phases
**Multiplayer-ready MVP:** Design interfaces in Phase 0, implement Phase 10 later

---

> **Refer to `@design/Orbit.md` for the full game design document and `@design/gap_analysis.md` for architectural decisions and risk mitigations.**
