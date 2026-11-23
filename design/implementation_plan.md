# ORBIT — Implementation Plan

> BabylonJS / TypeScript / Vite remake of Steve Belczyk's 1999 space combat simulator.
> Architecture-first approach: multiplayer-ready from day one, Electron + Tauri support early, high code coverage from the start.

---

## Table of Contents

1. [Architecture Principles](#1-architecture-principles)
2. [Project Structure](#2-project-structure)
3. [Build & Test Infrastructure](#3-build--test-infrastructure)
4. [Phase 0 — Foundation](#4-phase-0--foundation)
5. [Phase 1 — Eye Candy (Starfield + Ship)](#5-phase-1--eye-candy-starfield--ship)
6. [Phase 2 — Flight Physics & Controls](#6-phase-2--flight-physics--controls)
7. [Phase 3 — Planets & Solar System](#7-phase-3--planets--solar-system)
8. [Phase 4 — Mission Parser](#8-phase-4--mission-parser)
9. [Phase 5 — Combat & Weapons](#9-phase-5--combat--weapons)
10. [Phase 6 — AI Targets](#10-phase-6--ai-targets)
11. [Phase 7 — HUD & UI](#11-phase-7--hud--ui)
12. [Phase 8 — Events & Game Flow](#12-phase-8--events--game-flow)
13. [Phase 9 — Save System & Polish](#13-phase-9--save-system--polish)
14. [Phase 10 — Multiplayer Layer](#14-phase-10--multiplayer-layer)
15. [Phase 11 — Electron / Tauri Native Builds](#15-phase-11--electron--tauri-native-builds)
16. [Testing Strategy](#16-testing-strategy)
17. [Risk Register](#17-risk-register)

---

## 1. Architecture Principles

### 1.1 Core Design Decisions

| Principle                 | Decision                                                                                 | Rationale                                                  |
| ------------------------- | ---------------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| **Multiplayer-ready**     | Event-driven ECS-lite architecture with interfaces for all game systems                  | Network layer plugs in without refactoring core logic      |
| **Deterministic physics** | Fixed-timestep accumulator pattern (1/60s tick)                                          | Reproducible across platforms; essential for netcode later |
| **No framework coupling** | BabylonJS only for rendering/input; everything else is pure TS                           | Easy to swap renderers, test in Node, run headless server  |
| **Mission-first data**    | All game content driven by `.msn` files, not hardcoded                                   | Faithful to original; enables modding                      |
| **Interface segregation** | Every system has a `I*` interface implemented by both single-player and network variants | Clean separation of concerns                               |
| **Hybrid networking**     | Steam SDR for native (Electron/Tauri via steamworks.js), WebSocket for browsers          | Full Steam features on desktop, browser-compatible fallback |

### 1.2 Network-Ready Architecture Pattern

```txt
                    ┌─────────────┐
                    │  IGame      │  ← All game logic interfaces
                    │  Interface  │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
        ┌─────▼─────┐ ┌───▼────┐ ┌────▼─────┐
        │ SPGame     │ │ NetGame│ │ Headless │
        │ (impl)     │ │ (impl) │ │ Server   │
        └────────────┘ └────────┘ └──────────┘
```

Every game system follows this pattern:

- `IWeaponSystem` → `SinglePlayerWeaponSystem` + `NetworkedWeaponSystem`
- `IAIManager` → `SinglePlayerAIManager` + `ServerAIManager` (server-side AI)
- `IPlayerController` → `KeyboardPlayerController` + `JoystickPlayerController` + `NetworkPlayerProxy`

### 1.3 Hybrid Network Architecture

**Critical: Browser and native clients use different transport layers.**

```txt
                    ┌─────────────────────────────────┐
                    │     Relay Server (Phase 11)      │
                    │                                  │
         ┌──────────┼──────────┐              ┌───────▼───────┐
         │          │          │              │               │
    ┌────▼───┐  ┌───▼───┐  ┌──▼──┐   WebSocket│  Dedicated   │
    │ Steam  │  │ Steam │  │ ... │────────────│  Server      │
    │ Client │  │ Lobby │  │     │   (wss://) │  (Steam SDR  │
    │ (SDR)  │  │       │  │     │            │   + WebSocket│
    └────────┘  └───────┘  └─────┘            │   dual port)│
                                              └─────────────┘
         ┌─────────────────────────────────────────────────────┐
         │                                                      │
    ┌────▼───┐     Steamworks SDK     ┌────────────────────────┐
    │ Electron│ ← steamworks.js (Rust) │  Browser Client        │
    │ / Tauri │                        │ (WebSocket/wss://)     │
    └─────────┘                        └────────────────────────┘
         ▲                                   ▲
         │ Full Steam SDK access             │ OpenID + Web API auth
         │ (lobby, overlay, SDR, auth)       │ (SteamID64 verification)
```

**Platform-specific networking:**

| Platform | Transport | Auth | Features |
|----------|-----------|------|----------|
| **Electron/Tauri** | Steam SDR via `steamworks.js` | Session ticket | Lobbies, overlay, matchmaking, NAT punch |
| **Browser** | WebSocket (`wss://`) to relay | OpenID 2.0 → Web API | Server-verified ownership, no Steam overlay |
| **Dedicated Server** | Steam SDR + WebSocket dual port | GSLT | Master Server registration, bridges both worlds |

**Authentication flow (browser clients):**
1. Browser → Steam OpenID 2.0 (`steamcommunity.com/openid/login`)
2. Steam returns verified `SteamID64` to server
3. Server calls `ISteamUser/CheckAppOwnership` with Publisher Key
4. If owned, grant access; otherwise reject

**Server discovery (browser clients):**
- Server queries `IGameServersService/GetServerList` via Steam Web API
- Returns JSON list of active server IPs/ports to browser clients
- Browser connects directly via WebSocket to selected server

### 1.4 Coordinate System

**Critical: BabylonJS uses Y-up, original game uses Z-up.**

All position/orientation data from mission files and physics is stored in a **Z-up internal coordinate system**, then converted to Y-up for BabylonJS rendering only at the boundary layer.

```txt
Internal (Z-up)     →     BabylonJS (Y-up)
  X → X                  X → X
  Y → Z                  Y → Z
  Z → -Y                 Z → -Y
```

Conversion helper: `toBabylonVec3(x, y, z)` → `(x, -z, y)`
Inverse: `fromBabylonVec3(v)` → `(v.x, v.z, -v.y)`

---

## 2. Project Structure

```txt
orbit/
├── design/                    # GDD, architecture docs
│   ├── Orbit.md               # Game Design Document (completed)
│   └── implementation_plan.md # This file
├── src/
│   ├── index.ts               # Web entry point (boot Babylon scene)
│   ├── main.ts                # Node/desktop entry point
│   │
│   ├── core/                  # Foundation — no game logic
│   │   ├── Vector3.ts         # Z-up internal vector math
│   │   ├── Matrix4.ts         # 4x4 matrix operations
│   │   ├── Timer.ts           # Fixed-timestep accumulator
│   │   ├── EventBus.ts        # Event system (pub/sub)
│   │   ├── RNG.ts             # Seeded PRNG (Mulberry32)
│   │   └── Logger.ts          # Structured logging
│   │
│   ├── player/                # Player ship system
│   │   ├── IShip.ts           # Ship interface
│   │   ├── Ship.ts            # Core ship state (SP)
│   │   ├── FlightModel.ts     # Newtonian / Arcade models
│   │   ├── Camera.ts          # BabylonJS camera wrapper
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
│   │   ├── RingSystem.ts      # Planetary rings
│   │   └── Starfield.ts       # 65K+ star rendering
│   │
│   ├── missions/              # Mission parsing & execution
│   │   ├── IMissionParser.ts
│   │   ├── MissionParser.ts   # Tokenizer + brace parser
│   │   ├── Mission.ts         # Parsed mission data
│   │   ├── Event.ts           # Event entity
│   │   ├── Trigger.ts         # Trigger types enum
│   │   ├── Action.ts          # Action types enum
│   │   ├── EventHandler.ts    # Event evaluation engine
│   │   └── cursor/            # Cursor management
│   │       └── CursorStack.ts
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
│   │   ├── SpaceJunk.ts       # Motion reference polygons
│   │   └── SoundManager.ts    # Audio abstraction
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
│   │   ├── NetworkBridge.ts   # Protocol translation: Steam SDR ↔ WebSocket
│   │   ├── Packet.ts          # Packet format definitions
│   │   └── Server.ts          # Headless dedicated server (Steam SDR + WebSocket)
│   │
│   └── game/                  # Game orchestration
│       ├── IGame.ts           # Core game interface
│       ├── SinglePlayerGame.ts
│       ├── NetworkedGame.ts   # Client-side networked game
│       ├── GameState.ts       # State machine (enum)
│       └── GameFactory.ts     # Creates correct game mode
│
├── electron/                  # Electron app
│   ├── main.js                # Electron main process
│   ├── preload.js             # Security bridge
│   └── package.json           # Electron-specific deps
│
├── tauri/                     # Tauri app (Rust backend)
│   ├── src/                   # Rust Tauri handlers
│   ├── tauri.conf.json        # Tauri config
│   └── Cargo.toml             # Rust dependencies
│
├── tests/                     # Test suite (mirrors src/)
│   ├── core/
│   │   ├── vector3.test.ts
│   │   ├── matrix4.test.ts
│   │   ├── timer.test.ts
│   │   └── rng.test.ts
│   ├── player/
│   │   ├── flight-model.test.ts
│   │   └── ship.test.ts
│   ├── weapons/
│   │   ├── weapon-defs.test.ts
│   │   └── missile-pool.test.ts
│   ├── targets/
│   │   ├── strategy.test.ts
│   │   ├── turn-toward.test.ts
│   │   ├── move-toward.test.ts
│   │   ├── find-enemy.test.ts
│   │   └── aim.test.ts
│   ├── planets/
│   │   └── solar-system.test.ts
│   ├── missions/
│   │   ├── parser.test.ts           # Tokenizer + brace parsing
│   │   ├── cursor-stack.test.ts
│   │   ├── event-evaluator.test.ts
│   │   └── action-processor.test.ts
│   ├── effects/
│   │   └── explosion.test.ts
│   ├── save/
│   │   └── save-manager.test.ts
│   ├── network/
│   │   ├── packet.test.ts
│   │   └── server.test.ts
│   └── integration/
│       ├── mission-load.test.ts     # Full mission load → state check
│       └── combat-flow.test.ts      # Fire missile → hit → damage
│
├── data/                    # Game data (from original)
│   ├── missions/            # .msn and .inc files
│   ├── models/              # .tri and .ac model files
│   ├── maps/                # Planet texture PPMs
│   └── sounds/              # WAV files
│
├── dist/                    # Build output
├── coverage/                # Vitest coverage reports
├── docs/                    # Developer documentation
├── package.json
├── tsconfig.json
├── tsconfig.test.json
├── vite.config.ts
├── vitest.config.ts
├── eslint.config.js
├── .prettierrc.js
└── .gitignore
```

---

## 3. Build & Test Infrastructure

### 3.1 Package.json Scripts

```json
{
  "scripts": {
    "dev": "vite --open",
    "build": "tsc && vite build",
    "build:electron": "npm run build && cd electron && npm run build",
    "build:tauri": "npm run build && tauri build",
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
    "electron:start": "npm run build && cd electron && npm start",
    "tauri:dev": "tauri dev",
    "tauri:build": "tauri build"
  }
}
```

### 3.2 Vitest Configuration (mirrors CharGen)

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

### 3.3 Coverage Thresholds (CharGen-standard)

| Metric     | Threshold | Notes                                              |
| ---------- | --------- | -------------------------------------------------- |
| Lines      | 90%       | Core domain logic must be thoroughly tested        |
| Branches   | 85%       | AI strategies, event evaluation have many branches |
| Functions  | 90%       | Every public method must be called in tests        |
| Statements | 90%       | Consistent with line coverage                      |

### 3.4 Electron Setup (Early)

**Why early:** Native build testing catches platform-specific issues (file I/O, audio, input) from day one. Also the primary vehicle for full Steam SDK integration.

**Phase 0 setup:**

- `electron/main.js` — minimal window, loads Vite dev server in dev mode
- `electron/preload.js` — context bridge for IPC
- `electron/package.json` — depends on `@electron/remote`, `electron`, `steamworks.js`
- Dev mode: Electron connects to Vite HMR
- Prod mode: Electron loads built files from `dist/`
- **Steam integration:** `steamworks.js` loaded in Electron main process (Rust N-API wrapper for Steamworks SDK)
  - Full access to: Steam lobbies, SDR networking, overlay, session tickets, matchmaking
  - Bridge IPC between renderer (BabylonJS) and main process (Steam SDK)

### 3.5 Tauri Setup (Early)

**Why early:** Tauri provides a smaller footprint than Electron and is the preferred native target for performance-conscious builds.

**Phase 0 setup:**

- `tauri/tauri.conf.json` — basic config, app ID `orbit.space`
- `tauri/Cargo.toml` — Rust deps: `tauri`, `steamworks.js` (via N-API)
- Tauri backend serves the Vite-built static files
- Rust handlers for native file I/O and audio (if needed)
- **Steam integration:** Same `steamworks.js` approach as Electron — loads Steamworks SDK in Rust backend

### 3.6 Headless Dedicated Server

**Architecture:** The server is a pure Node.js process with no rendering dependencies. Dual transport layer.

```txt
Server:
  - No BabylonJS dependency
  - Loads missions via MissionParser
  - Runs AI targets, physics, events
  - Steam SDR port: Accepts native clients (via Steamworks SDK)
  - WebSocket port: Accepts browser clients (wss://)
  - NetworkBridge: Translates between Steam SDR and WebSocket protocols
  - Authoritative game state
```

**Files:**
- `src/network/Server.ts` — Pure Node.js server, no DOM required
- `src/network/NetworkBridge.ts` — Protocol translation layer
- Server initializes Steamworks SDK (`SteamGameServer_Init`) for Master Server registration + GSLT auth

---

## 4. Phase 0 — Foundation

**Goal:** Project scaffolding, build system, testing infrastructure, core math library.
**Duration:** ~1-2 days
**Visual gratification:** None yet (pure infrastructure).

### 4.1 Tasks

| #    | Task                                                                  | Deliverable                                                                             | Tests         |
| ---- | --------------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ------------- |
| 0.1  | Initialize project: `package.json`, `tsconfig.json`, `vite.config.ts` | Working `npm run dev`                                                                   | N/A           |
| 0.2  | Create `vitest.config.ts` with coverage thresholds (90/85/90/90)      | `npm run test:coverage` passes                                                          | N/A           |
| 0.3  | Set up ESLint + Prettier config                                       | `npm run lint` + `npm run format` clean                                                 | N/A           |
| 0.4  | Implement `Vector3` (Z-up internal)                                   | `tests/core/vector3.test.ts` — dot, cross, normalize, add, subtract, scale, distance    | ≥90% coverage |
| 0.5  | Implement `Matrix4` (column-major, 4x4)                               | `tests/core/matrix4.test.ts` — identity, rotation, translation, multiply, transformVec3 | ≥90% coverage |
| 0.6  | Implement `Timer` (fixed-timestep accumulator)                        | `tests/core/timer.test.ts` — advance, getDeltaT, reset, interpolation factor            | ≥90% coverage |
| 0.7  | Implement `EventBus` (pub/sub with typed events)                      | `tests/core/event-bus.test.ts` — subscribe, emit, unsubscribe, one-time listeners       | ≥90% coverage |
| 0.8  | Implement `RNG` (Mulberry32 seeded PRNG)                              | `tests/core/rng.test.ts` — seed determinism, range bounds, statistical distribution     | ≥90% coverage |
| 0.9  | Implement `Logger` (structured, log levels)                           | `tests/core/logger.test.ts` — log levels, output formatting                             | ≥90% coverage |
| 0.10 | Initialize Electron app (`electron/`)                                 | `npm run electron:start` opens window                                                   | N/A           |
| 0.11 | Initialize Tauri app (`tauri/`)                                       | `npm run tauri:dev` opens window                                                        | N/A           |
| 0.12 | Set up data conversion pipeline (PPM→PNG, .tri/.ac→GLTF)              | `npm run convert-assets` converts all legacy assets                                     | N/A           |
| 0.13 | Scaffold relay server (`RelayServer.ts`)                              | Dual transport: Steam SDR + WebSocket listeners                                         | N/A           |
| 0.14 | Integrate `steamworks.js` for Electron/Tauri                          | Full Steam SDK access in native builds (lobby, SDR, overlay)                             | N/A           |
| 0.15 | Implement Steam OpenID auth flow                                      | Browser → OpenID login → SteamID64 verification via Web API                              | N/A           |
| 0.16 | Implement server discovery (`IGameServersService/GetServerList`)      | JSON server list endpoint for browser matchmaking                                       | N/A           |

### 4.2 Key Interfaces (Phase 0)

```typescript
// Vector3 — Z-up internal coordinate system
interface Vector3 {
  x: number; y: number; z: number;
  static zero(): Vector3;
  static fromBabylon(v: BABYLON.Vector3): Vector3;
  toBabylon(): BABYLON.Vector3;
  add(other: Vector3): Vector3;
  subtract(other: Vector3): Vector3;
  scale(scalar: number): Vector3;
  dot(other: Vector3): number;
  cross(other: Vector3): Vector3;
  magnitude(): number;
  magnitudeSquared(): number;
  normalize(): Vector3;
  clone(): Vector3;
}

// Timer — fixed-timestep accumulator
interface Timer {
  tick(now: number): void;
  getDeltaT(): number;           // Fixed timestep (1/60)
  getAccumulator(): number;      // Raw accumulator
  reset(): void;
}

// RNG — deterministic seeded PRNG
interface RNG {
  seed(value: number): void;
  next(): number;                // [0, 1)
  nextInt(min: number, max: number): number;
  nextFloat(min: number, max: number): number;
}

// EventBus — typed pub/sub
interface EventBus<TEvents extends Record<string, unknown>> {
  subscribe<K extends keyof TEvents>(event: K, handler: (data: TEvents[K]) => void): void;
  emit<K extends keyof TEvents>(event: K, data: TEvents[K]): void;
}
```

---

## 5. Phase 1 — Eye Candy (Starfield + Ship)

**Goal:** Something visible and interactive on screen. Starfield rendering + ship model + basic camera control.
**Duration:** ~2-3 days
**Visual gratification:** 🌟🌟🌟 Stars, ship, camera orbit — immediate payoff!

### 5.1 Tasks

| #   | Task                                                        | Deliverable                                      | Tests                                                                              |
| --- | ----------------------------------------------------------- | ------------------------------------------------ | ---------------------------------------------------------------------------------- |
| 1.1 | Initialize BabylonJS scene                                  | Scene renders black screen                       | N/A                                                                                |
| 1.2 | Implement `Starfield` (65K+ stars, two density levels)      | Visible starfield with depth sorting             | `tests/planets/starfield.test.ts` — star count, brightness mapping, culling        |
| 1.3 | Create `Sun` mesh (glowing sphere, light source)            | Glowing sun at origin                            | N/A                                                                                |
| 1.4 | Load and render a ship model (`light2.tri` or equivalent)   | Ship mesh visible in scene                       | `tests/core/tri-loader.test.ts` — vertex parsing, bounding box, normal computation |
| 1.5 | Implement `InputManager` (keyboard + joystick abstraction)  | `IInputManager` interface + `KeyboardInput` impl | `tests/player/input-manager.test.ts` — key state tracking, axis mapping            |
| 1.6 | Implement `Camera` wrapper around BabylonJS ArcRotateCamera | Camera responds to input                         | N/A                                                                                |
| 1.7 | Wire camera to ship orientation (view lock mode)            | Camera orbits ship with pitch/yaw                | N/A                                                                                |
| 1.8 | Add space junk reference polygons (motion cues)             | Subtle rotating polygons in background           | N/A                                                                                |

### 5.2 Key Interfaces (Phase 1)

```typescript
interface IInputManager {
  getKeyState(key: string): number;    // bit7=down, bit0=pressed-this-frame
  getAxis(axis: string): number;        // -1 to +1 analog value
  isKeyPressed(key: string): boolean;
  isKeyDown(key: string): boolean;
}

interface IStarfield {
  create(scene: BABYLON.Scene): void;
  setDensity(density: 'sparse' | 'dense'): void;
  updateCamera(camera: BABYLON.Camera): void;
}

interface IShipModel {
  load(modelPath: string, scene: BABYLON.Scene): BABYLON.Mesh;
  getBoundingInfo(): BABYLON.BoundingInfo;
  setOrientation(viewDir: Vector3, upDir: Vector3): void;
}
```

### 5.3 .tri Model Loader

The `.tri` format is simple enough to load without external dependencies:

```
v1x v1y v1z  v2x v2y v2z  v3x v3y v3z  0xRRGGBB
```

One triangle per line. Parse into BabylonJS `DynamicMesh` with vertex colors.

**Test:** Load a `.tri` file, verify vertex count, bounding box, and color extraction.

---

## 6. Phase 2 — Flight Physics & Controls

**Goal:** Full ship movement with both Newtonian and Arcade flight models.
**Duration:** ~3-4 days
**Visual gratification:** 🌟🌟🌟🌟 Fly around the solar system!

### 6.1 Tasks

| #   | Task                                                    | Deliverable                                     | Tests                                                                           |
| --- | ------------------------------------------------------- | ----------------------------------------------- | ------------------------------------------------------------------------------- |
| 2.1 | Implement `FlightModel` base interface + Newtonian impl | `IFlightModel` + `NewtonianFlightModel`         | `tests/player/flight-model.test.ts` — delta-v accumulation, warp speed, gravity |
| 2.2 | Implement Arcade flight model                           | `ArcadeFlightModel`                             | Same test suite (polymorphic)                                                   |
| 2.3 | Implement `Ship` state machine                          | `Ship` class with position/velocity/orientation | `tests/player/ship.test.ts` — state transitions, shield regen, death timer      |
| 2.4 | Wire keyboard input to flight controls                  | All keys functional (WASDQEZX + arrows)         | `tests/player/input-mapping.test.ts` — key to axis mapping                      |
| 2.5 | Implement joystick input support                        | Joystick axes mapped to yaw/pitch/roll/throttle | N/A                                                                             |
| 2.6 | Implement gravitational physics engine                  | Gravity affects player, missiles, targets       | `tests/player/gravity.test.ts` — force calculation, singularity clamping        |
| 2.7 | Implement full stop behavior (Newtonian)                | SPACE key stops ship in Newtonian mode          | N/A                                                                             |
| 2.8 | Implement warp speed (both models)                      | Warp acceleration / superwarp (SP only)         | `tests/player/warp.test.ts` — warp coefficient, superwap formula                |

### 6.2 Key Interfaces

```typescript
interface IFlightModel {
  type: 'newtonian' | 'arcade';
  update(ship: Ship, deltaT: number, input: IInputManager): void;
  applyGravity(ship: Ship, gravity: Vector3): void;
}

interface IShip {
  pos: Vector3;
  vel: Vector3;
  viewDir: Vector3;
  upDir: Vector3;
  rightDir: Vector3;
  shields: number;
  maxShields: number;
  score: number;
  weapon: number;
  state: ShipState;
  flightModel: IFlightModel;
  update(deltaT: number, input: IInputManager): void;
  fireMissile(weaponIndex: number): Missile | null;
  takeDamage(amount: number): boolean; // true if died
  regenerateShields(deltaT: number): void;
}

interface IGravityEngine {
  addBody(position: Vector3, mass: number): void;
  removeBody(position: Vector3): void;
  computeForce(position: Vector3): Vector3; // Total gravitational force at position
}
```

### 6.3 Physics Constants (from original)

| Constant            | Value          | Description                |
| ------------------- | -------------- | -------------------------- |
| `THETA`             | 1.6 rad/s      | Max rotation rate          |
| `DELTAV`            | 0.2 units/s²   | Thrust acceleration        |
| `WARP_COEFF`        | 0.2            | Warp velocity coefficient  |
| `MAX_THROTTLE`      | 10,000 km/s    | Max throttle (arcade)      |
| `MAX_WARP_THROTTLE` | 1,000,000 km/s | Max warp throttle (arcade) |
| `SHIELD_REGEN`      | configurable   | Shield regeneration rate   |
| `DEAD_TIME`         | 5.0 s          | Pre-respawn countdown      |

---

## 7. Phase 3 — Planets & Solar System

**Goal:** All planets, moons, rings, and orbital mechanics.
**Duration:** ~3-4 days
**Visual gratification:** 🌟🌟🌟🌟🌟 Full solar system!

### 7.1 Tasks

| #   | Task                                                       | Deliverable                                 | Tests                                                                        |
| --- | ---------------------------------------------------------- | ------------------------------------------- | ---------------------------------------------------------------------------- |
| 3.1 | Define `Planet` data structure + initial solar system data | 30+ planets/moons with NASA-based positions | `tests/planets/solar-system.test.ts` — planet count, positions, masses       |
| 3.2 | Implement planet rendering (distance-based LOD)            | Textured spheres → points based on distance | N/A                                                                          |
| 3.3 | Implement planetary ring systems (4 rings)                 | Jupiter, Saturn, Uranus, Neptune rings      | `tests/planets/ring-system.test.ts` — inner/outer radius, texture bounds     |
| 3.4 | Implement orbital animation                                | Planets orbit in XZ plane around primary    | `tests/planets/orbital-mechanics.test.ts` — angular velocity, orbit position |
| 3.5 | Load planet textures from PPM files                        | Textured planets with correct colors        | N/A                                                                          |
| 3.6 | Implement planet name labels (billboarded text)            | Names appear above planets at distance      | N/A                                                                          |
| 3.7 | Implement planet hiding/unhiding (mission events)          | `HidePlanet` / `UnhidePlanet` actions work  | `tests/missions/action-processor.test.ts`                                    |

### 7.2 Key Interfaces

```typescript
interface IPlanet {
  name: string;
  pos: Vector3;
  vel: Vector3;
  radius: number;
  mass: number;
  color: [number, number, number];
  texturePath?: string;
  isMoon: boolean;
  primary: IPlanet | null;
  rings?: IRingSystem;
  hidden: boolean;
  theta: number; // Orbital angle
  update(deltaT: number): void;
  render(scene: BABYLON.Scene, distanceSquared: number): void;
}

interface IRingSystem {
  innerRadius: number;
  outerRadius: number;
  texturePath: string;
  sectors: number;
  create(scene: BABYLON.Scene): void;
  update(parentPos: Vector3, oblicity: number): void;
}

interface ISolarSystem {
  planets: IPlanet[];
  sun: IPlanet;
  addPlanet(planet: IPlanet): void;
  getGravityAt(position: Vector3): Vector3;
  update(deltaT: number): void;
  render(scene: BABYLON.Scene, cameraDist: number): void;
}
```

---

## 8. Phase 4 — Mission Parser

**Goal:** Parse `.msn` and `.inc` mission files, build game state from them.
**Duration:** ~3-4 days
**Visual gratification:** 🌟🌟 Load a mission → see it render!

### 8.1 Tasks

| #    | Task                                                      | Deliverable                                       | Tests                                                                                   |
| ---- | --------------------------------------------------------- | ------------------------------------------------- | --------------------------------------------------------------------------------------- |
| 4.1  | Implement tokenizer (whitespace-separated, comment-aware) | `tokenize(missionText): string[]`                 | `tests/missions/parser.test.ts` — comment stripping, whitespace handling, include paths |
| 4.2  | Implement brace-delimited block parser                    | `parseBlock(tokens, depth): Block`                | Same test file — nested blocks, unclosed brace errors                                   |
| 4.3  | Implement `CursorStack` (position accumulation)           | Named cursors with relative offsets               | `tests/missions/cursor-stack.test.ts` — push/pop, named lookup, offset math             |
| 4.4  | Implement `MissionParser` (top-level token routing)       | Parse full `.msn` → `Mission` object              | Integration test: load `train01.msn` → verify mission data                              |
| 4.5  | Implement `Player` block parsing                          | Player start position, model, name                | Same integration test                                                                   |
| 4.6  | Implement `Object` block parsing                          | Target definitions with strategy, weapon, shields | Same integration test                                                                   |
| 4.7  | Implement `Event` block parsing                           | Triggers, actions, conditions                     | `tests/missions/event-parser.test.ts` — trigger/action enum validation                  |
| 4.8  | Implement `Briefing` text extraction                      | Multi-line briefing text preserved                | Same integration test                                                                   |
| 4.9  | Implement `Planet` modification parsing                   | `Reposition`, `Map`, `Hidden`, `Oblicity`         | `tests/missions/planet-mod.test.ts`                                                     |
| 4.10 | Implement `Include` file resolution                       | 16-level deep includes work                       | `tests/missions/include-resolution.test.ts` — circular include detection                |

### 8.2 Mission File Parser Architecture

```typescript
// Tokenizer: raw text → token stream
function tokenize(input: string): string[] {
  // Strip /* ... */ comments
  // Split on whitespace
  // Handle "quoted strings" for briefing text
}

// Brace parser: token stream → AST
interface Block {
  type: string;           // "Player", "Object", "Event", etc.
  properties: Map<string, string>;
  children: Block[];
}

function parseBlocks(tokens: string[], start: number): { block: Block; end: number } {
  // Track brace depth
  // Accumulate properties until closing brace
}

// Cursor management
class CursorStack {
  private stack: { name?: string; pos: Vector3 }[] = [];
  
  push(name: string | undefined, offset: Vector3): void { /* ... */ }
  pop(): void { /* ... */ }
  getPosition(name?: string): Vector3 { /* ... */ }
}

// Mission parser: tokens → game data
class MissionParser {
  private cursor = new CursorStack();
  
  parse(text: string): Mission {
    const tokens = tokenize(text);
    let i = 0;
    
    while (i < tokens.length) {
      const token = tokens[i];
      switch (token) {
        case 'Cursor': this.cursor.push(...); i++; break;
        case 'Player': this.parsePlayer(tokens, i); break;
        case 'Object': this.parseObject(tokens, i); break;
        case 'Event': this.parseEvent(tokens, i); break;
        case 'Briefing': this.parseBriefing(tokens, i); break;
        case 'Include': this.resolveInclude(tokens[i+1]); break;
        case 'Planet': this.parsePlanetMod(tokens, i); break;
      }
    }
    
    return this.buildMission();
  }
}
```

### 8.3 Mission Data Structure

```typescript
interface IMission {
  name: string;
  briefing: string[];
  playerStart: {
    position: Vector3;
    model: string;
    name: string;
  };
  targets: ITargetDef[];
  events: IEventDef[];
  waypoints: Vector3[];
  planets: IPlanetMod[]; // Planet modifications from mission
}

interface ITargetDef {
  name: string;
  model: string;
  score: number;
  strategy: Strategy;
  friendly: boolean;
  hidden: boolean;
  weapon: number;
  maxShields: number;
  shieldRegen: number;
  turnRate: number;
  speed: number;
  position: Vector3; // Resolved from cursor
}

interface IEventDef {
  name?: string;
  trigger: Trigger;
  triggerValue: number | string; // Distance, score, target name, seconds
  enabled: boolean;
  action: Action;
  actionValue: number | string; // Points, target name, text, event name
}
```

---

## 9. Phase 5 — Combat & Weapons

**Goal:** Fire weapons, missiles fly, targets take damage, explosions happen.
**Duration:** ~3-4 days
**Visual gratification:** 🌟🌟🌟🌟🌟💥 BOOM!

### 9.1 Tasks

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

### 9.2 Key Interfaces

```typescript
interface IWeaponDef {
  name: string;
  speed: number;           // units/s
  yield: number;           // damage on hit
  idleTime: number;        // cooldown seconds
  lifetime: number;        // projectile lifetime seconds
  renderer: number;        // 0-4 (which visual)
  color: [number, number, number];
}

interface IMissile {
  pos: Vector3;
  vel: Vector3;
  weaponDef: IWeaponDef;
  owner: string;           // "player" or target name
  age: number;
  alive: boolean;
  update(deltaT: number): void;
}

interface IWeaponSystem {
  fire(owner: string, weaponIndex: number, origin: Vector3, direction: Vector3): IMissile | null;
  updateAll(deltaT: number): void;
  getActiveMissiles(): IMissile[];
  checkCollisions(targets: ITarget[]): CollisionResult[];
}

interface CollisionResult {
  missile: IMissile;
  target: ITarget | null;
  planet: IPlanet | null;
  hit: boolean;
}
```

### 9.3 Weapon Definitions (from GDD)

| Index | Name      | Speed    | Yield | Idle | Lifetime | Renderer    | Color   |
| ----- | --------- | -------- | ----- | ---- | -------- | ----------- | ------- |
| 0     | Laser     | 4000/KMU | 20    | 0.2  | 1.0      | Cross       | (1,1,0) |
| 1     | PhotonRay | 5000/KMU | 30    | 0.3  | 0.75     | Ray         | (0,0,1) |
| 2     | IonGun    | 4000/KMU | 30    | 0.5  | 1.5      | Tetrahedron | (0,1,0) |
| 3     | Disruptor | 3000/KMU | 60    | 1.0  | 1.5      | Loop        | (1,1,0) |
| N+    | Spare     | 6000/KMU | 15    | 2.0  | 1.0      | None        | (1,0,0) |

---

## 10. Phase 6 — AI Targets

**Goal:** Enemy and friendly targets with 9 AI strategies.
**Duration:** ~3-4 days
**Visual gratification:** 🌟🌟🌟 Targets that actually fight back!

### 10.1 Tasks

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

### 10.2 Key Interfaces

```typescript
enum Strategy {
  DoNothing = 'STRAT_DONOTHING',
  Sit1 = 'STRAT_SIT1',   // Turns toward player, shoots poorly
  Sit2 = 'STRAT_SIT2',   // Aims at future position, shoots poorly
  Sit3 = 'STRAT_SIT3',   // Finds nearest enemy, turns toward it
  Sit4 = 'STRAT_SIT4',   // Finds nearest enemy, predictive aim, shoots well
  Hunt1 = 'STRAT_HUNT1', // Moves toward player, shoots
  Hunt2 = 'STRAT_HUNT2', // Moves toward aiming point, shoots
  Hunt3 = 'STRAT_HUNT3', // Moves toward closest enemy, shoots poorly
  Hunt4 = 'STRAT_HUNT4', // Moves toward aiming point, shoots well
}

interface ITarget {
  name: string;
  pos: Vector3;
  vel: Vector3;
  viewDir: Vector3;
  upDir: Vector3;
  strategy: Strategy;
  friendly: boolean;
  shields: number;
  maxShields: number;
  weapon: number;
  score: number;
  hidden: boolean;
  invisible: boolean;
  model: string;
  turnRate: number;
  speed: number;
  age: number;
  lastFireTime: number;
}

interface ITargetManager {
  addTarget(target: ITarget): void;
  removeTarget(name: string): void;
  updateAll(deltaT: number, playerPos: Vector3, playerVel: Vector3): void;
  getTargets(): ITarget[];
  getAliveTargets(): ITarget[];
}
```

---

## 11. Phase 7 — HUD & UI

**Goal:** All HUD elements rendered on screen.
**Duration:** ~2-3 days
**Visual gratification:** 🌟🌟🌟 Full game UI!

### 11.1 Tasks

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

### 11.2 Key Interfaces

```typescript
interface IHUD {
  update(player: IShip, targets: ITarget[], lockedTarget: ITarget | null): void;
  render(scene: BABYLON.Scene): void;
}

interface IRadar {
  center: Vector3;
  range: number;
  render(targets: ITarget[], planets: IPlanet[]): void;
}

interface IMessageOverlay {
  messages: MessageEntry[];
  addMessage(text: string, type: 'briefing' | 'console' | 'event'): void;
  clear(): void;
  render(scene: BABYLON.Scene): void;
}
```

---

## 12. Phase 8 — Events & Game Flow

**Goal:** Mission events trigger actions, game state transitions work.
**Duration:** ~2-3 days
**Visual gratification:** 🌟🌟🌟 Events fire, missions progress!

### 12.1 Tasks

| #   | Task                                           | Deliverable                           | Tests                                                                                                                        |
| --- | ---------------------------------------------- | ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| 8.1 | Implement `EventEvaluator` (trigger checking)  | All trigger types evaluated per frame | `tests/missions/event-evaluator.test.ts` — Approach, Depart, True, Score, Destroy, Alarm                                     |
| 8.2 | Implement `ActionProcessor` (action execution) | All action types executed             | `tests/missions/action-processor.test.ts` — Message, Hide, Unhide, Destroy, Score, Enable, Disable, LoadMission, Boom, Flash |
| 8.3 | Implement game state machine                   | `GameState` enum with transitions     | `tests/game/game-state.test.ts` — valid transitions only                                                                     |
| 8.4 | Implement death/respawn sequence               | STATE_DEAD1 → STATE_DEAD2 → respawn   | `tests/game/death-respawn.test.ts` — timer countdown, explosion                                                              |
| 8.5 | Implement mission chain progression            | LoadMission action chains missions    | Integration test: train01 → train02                                                                                          |
| 8.6 | Implement briefing display on mission load     | Orange text auto-wraps                | N/A                                                                                                                          |
| 8.7 | Implement victory/defeat screens               | win.msn / lose.msn                    | `tests/game/end-states.test.ts`                                                                                              |

### 12.2 Game State Machine

```typescript
enum GameState {
  INIT = 'STATE_INIT',
  NORMAL = 'STATE_NORMAL',
  DEAD1 = 'STATE_DEAD1',      // Just died, 5s countdown
  DEAD2 = 'STATE_DEAD2',      // Countdown over, press fire to respawn
  LOADGAME = 'STATE_LOADGAME', // Loading saved mission
}

interface IGame {
  state: GameState;
  deltaT: number;
  absT: number;
  compression: number;
  paused: boolean;
  
  update(deltaT: number): void;
  getState(): GameState;
  isPaused(): boolean;
  togglePause(): void;
}
```

---

## 13. Phase 9 — Save System & Polish

**Goal:** Save/load missions, polish visuals and controls.
**Duration:** ~2-3 days
**Visual gratification:** 🌟🌟 Save your progress!

### 13.1 Tasks

| #   | Task                                   | Deliverable                                 | Tests                                                        |
| --- | -------------------------------------- | ------------------------------------------- | ------------------------------------------------------------ |
| 9.1 | Implement `LocalSaveManager` (SP)      | File I/O (native) / IndexedDB (web)         | `tests/save/local-save-manager.test.ts` — write, read, max entries |
| 9.2 | Implement `ServerSaveManager`          | Server-side file storage                    | `tests/save/server-save-manager.test.ts` — server save/load  |
| 9.3 | Enforce local/server save separation   | No cross-sync mechanism                     | `tests/save/separation.test.ts` — upload/download rejected   |
| 9.4 | Implement save menu UI                 | Numbered list with mission name + timestamp | N/A                                                          |
| 9.5 | Implement screen flash effects         | Red on damage, white on events              | N/A                                                          |
| 9.6 | Implement planet texture customization | `Map` action changes planet textures        | `tests/missions/action-processor.test.ts`                    |
| 9.7 | Implement field of view adjustment     | > / < keys change FOV                       | N/A                                                          |
| 9.8 | Implement clipping plane adjustment    | { } ( ) keys adjust near/far clip           | N/A                                                          |
| 9.9 | Polish starfield rendering             | Depth sorting, brightness mapping           | N/A                                                          |
| 9.10 | Add sound effects integration          | Phaser, explosion, comm sounds              | `tests/effects/sound-manager.test.ts`                        |

### 13.2 Save System Architecture — Strict Separation

**Critical: Single-player saves and server saves are completely isolated.**

| Scenario | Where Saves Are Stored | Can Upload to Server? | Can Download from Server? |
|----------|----------------------|----------------------|--------------------------|
| **Single-player (web)** | Browser IndexedDB | ❌ No | ❌ No |
| **Single-player (Electron/Tauri)** | Local filesystem (`~/.orbit/saves/`) | ❌ No | ❌ No |
| **Multiplayer session** | Server-side file only | N/A | N/A |

**Rationale:**
- Prevents save manipulation via server interaction
- Keeps architecture simple — no sync complexity
- Server saves are ephemeral (tied to server session)
- Local saves are persistent (player-owned)

### 13.3 Save File Format

```
missions/train01.msn 1699900800
missions/jupiter1.msn 1699987200
```

One entry per line: `mission_filename unix_timestamp`. Most recent at top. Max entries configurable (`NSAVES`).

---

## 14. Phase 10 — Multiplayer Layer

**Goal:** Networked multiplayer with headless server support.
**Duration:** ~5-7 days
**Visual gratification:** 🌟🌟🌟🌟 Multiplayer!

### 14.1 Tasks

| #     | Task                                               | Deliverable                            | Tests                                                                    |
| ----- | -------------------------------------------------- | -------------------------------------- | ------------------------------------------------------------------------ |
| 10.1  | Define `INetworkLayer` interface                   | All network operations abstracted      | `tests/network/packet.test.ts` — packet format, serialization            |
| 10.2  | Implement `SPNetworkLayer` (no-op)                 | SP mode unaffected by network layer    | N/A                                                                      |
| 10.3  | Implement binary packet protocol                   | Magic byte 0x56, size-prefixed packets | `tests/network/packet.test.ts` — all packet types                        |
| 10.4  | Implement `SteamNetwork.ts` (native client)        | Steam SDR via steamworks.js            | `tests/network/steam-network.test.ts` — connect, send/receive packets    |
| 10.5  | Implement `WebSocketNetwork.ts` (browser client)   | wss:// connection to relay server      | `tests/network/websocket-network.test.ts` — connect, message handling    |
| 10.6  | Implement `NetworkBridge.ts`                       | Protocol translation: SDR ↔ WebSocket  | `tests/network/bridge.test.ts` — bidirectional translation               |
| 10.7  | Implement `NetworkedGame` (client)                 | Client-side networked game             | Integration test: connect → sync → play                                  |
| 10.8  | Implement `NetworkedAIManager` (server)            | Server-side AI for all targets         | Same as SP tests (polymorphic)                                           |
| 10.9  | Implement position sync protocol                   | Urgent/regular/long position reports   | `tests/network/position-sync.test.ts` — update frequency, lag handling   |
| 10.10 | Implement client management                        | Join/leave, slot management, roll call | `tests/network/client-management.test.ts`                                |

### 14.2 Network Packet Format

```typescript
// Binary packets (all prefixed with magic byte 0x56)
interface IPacket {
  magic: 0x56;
  type: PacketType;
  size: number;
  data: Uint8Array;
}

enum PacketType {
  POSS = 0x80,   // Short position report
  POSL = 0x81,   // Long position report
  POSU = 0x82,   // Urgent position report
  POSN = 0x83,   // Regular position report
  PLAN = 0x84,   // Planet position data
  VCNT = 0x85,   // Vacancy report
  PING = 0x86,   // Latency ping
}

// ASCII packets (for commands)
type ASCIICommand = 
  | `FIRE ${string}`
  | `CHAT ${string}`
  | `CMSG ${string}: ${string}`
  | `DROP ${string}`
  | `NAME ${string}`;
```

### 14.3 Client Architecture — Platform-Specific

**Native client (Electron/Tauri):**
```typescript
class SteamNetwork implements INetworkLayer {
  // Uses steamworks.js in Electron main process
  private steam: SteamworksJS;
  
  async connect(serverSteamID: SteamID): Promise<void> {
    // Connect via Steam SDR to server
    this.steam.networking.connectBySteamID(serverSteamID);
  }
  
  sendPacket(packet: IPacket): void {
    this.steam.networking.sendPacket(packet.data, packet.size);
  }
  
  onPacket(callback: (packet: IPacket) => void): void {
    this.steam.networking.onIncomingPacket(callback);
  }
}
```

**Browser client:**
```typescript
class WebSocketNetwork implements INetworkLayer {
  private ws: WebSocket;
  
  async connect(serverUrl: string): Promise<void> {
    // Connect to relay server via secure WebSocket
    this.ws = new WebSocket(`wss://${serverUrl}`);
    await this.authenticate(); // Steam OpenID token
  }
  
  sendPacket(packet: IPacket): void {
    this.ws.send(JSON.stringify({ type: 'packet', data: btoa(String.fromCharCode(...packet.data)) }));
  }
  
  onPacket(callback: (packet: IPacket) => void): void {
    this.ws.onmessage = (e) => {
      const msg = JSON.parse(e.data);
      if (msg.type === 'packet') {
        const data = Uint8Array.from(atob(msg.data), c => c.charCodeAt(0));
        callback({ magic: 0x56, type: data[1], size: data.length, data });
      }
    };
  }
}
```

### 14.4 Relay Server Architecture (Phase 11)

The relay server bridges Steam SDR clients and WebSocket clients:

```txt
RelayServer.ts:
  - SteamGameServer_Init() → Master Server registration
  - Steam SDR port: Accepts native clients
  - WebSocket port: Accepts browser clients
  - NetworkBridge: Translates packets between transports
  - Authoritative game state (same as headless server)
```

**Phase 11 additions to relay server:**
- Full dual-transport implementation
- `ServerSaveManager.ts` — server-side save system
- Steam OpenID verification endpoint (`CheckAppOwnership`)
- Server admin commands (DROP, roll call)

---

## 15. Phase 11 — Electron / Tauri Native Builds

**Goal:** Production-ready native builds for macOS, Windows, Linux.
**Duration:** ~2-3 days
**Visual gratification:** 🌟🌟🌟🌟🌟 Ship it!

### 15.1 Tasks

| #    | Task                                        | Deliverable                                     | Tests                                |
| ---- | ------------------------------------------- | ----------------------------------------------- | ------------------------------------ |
| 11.1 | Configure Electron production build         | `npm run build:electron` produces distributable | N/A                                  |
| 11.2 | Configure Tauri production build            | `npm run tauri:build` produces distributable    | N/A                                  |
| 11.3 | Implement native file I/O bridge (Electron) | Mission loading from disk                       | `tests/electron/file-bridge.test.ts` |
| 11.4 | Implement native file I/O bridge (Tauri)    | Mission loading via Tauri commands              | `tests/tauri/file-bridge.test.ts`    |
| 11.5 | Add platform-specific audio handling        | Native audio API fallbacks                      | N/A                                  |
| 11.6 | Configure code signing (macOS)              | notarized builds                                | N/A                                  |
| 11.7 | Configure Windows installer                 | NSIS installer                                  | N/A                                  |
| 11.8 | Configure Linux AppImage/DEB                | AppImage + DEB output                           | N/A                                  |
| 11.9 | Implement Steam overlay integration         | In-game Steam overlay (native builds)           | `tests/electron/steam-overlay.test.ts` |
| 11.10| Implement Steam lobby system                | Create/join lobbies via steamworks.js           | `tests/network/steam-lobby.test.ts`  |
| 11.11| Build relay server with dual transport      | Full RelayServer.ts (Steam SDR + WebSocket)     | `tests/network/relay-server.test.ts` |
| 11.12| Implement Steam OpenID verification         | Server-side `CheckAppOwnership` endpoint        | `tests/network/openid-verify.test.ts`|

---

## 16. Testing Strategy

### 16.1 Test Categories (mirrors CharGen)

| Category              | Location                                               | Purpose                               | Coverage Target     |
| --------------------- | ------------------------------------------------------ | ------------------------------------- | ------------------- |
| **Unit tests**        | `tests/core/`, `tests/player/`, `tests/weapons/`, etc. | Individual functions/classes          | ≥90% all metrics    |
| **Integration tests** | `tests/integration/`                                   | Multi-system workflows                | Key paths only      |
| **Invariant suite**   | `tests/domain/invariant-suite.test.ts`                 | Property-based testing for physics/AI | Critical invariants |

### 16.2 Test File Naming Convention

```
tests/<module>/<feature>.test.ts
```

Examples:

- `tests/player/flight-model.test.ts` — Newtonian and Arcade flight models
- `tests/missions/parser.test.ts` — Tokenizer, brace parser, cursor stack
- `tests/targets/turn-toward.test.ts` — AI turn primitive
- `tests/effects/explosion.test.ts` — Explosion generation and pooling

### 16.3 Key Test Scenarios

| Test                                  | Module                    | What It Verifies                |
| ------------------------------------- | ------------------------- | ------------------------------- |
| `vector3.dot-cross-normalize`         | core/vector3              | Vector math correctness         |
| `matrix4.transformVec3`               | core/matrix4              | Matrix-vector multiplication    |
| `timer.fixed-accumulator`             | core/timer                | Fixed timestep with accumulator |
| `rng.deterministic-seed`              | core/rng                  | Same seed → same sequence       |
| `flight-model.newtonian-deltav`       | player/flight-model       | Delta-v accumulation formula    |
| `flight-model.arcade-throttle`        | player/flight-model       | Direct velocity setting         |
| `flight-model.warp-coefficient`       | player/warp               | Warp speed calculation          |
| `gravity.compute-force`               | player/gravity            | Gravitational force at position |
| `weapon-defs.all-defined`             | weapons/weapon-defs       | All 5 weapons have valid defs   |
| `missile-pool.recycle-overflow`       | weapons/missile-pool      | Oldest-first eviction           |
| `strategy.sit1-turns-toward-player`   | targets/strategy          | Sit1 behavior                   |
| `strategy.hunt4-predictive-aim`       | targets/strategy          | Hunt4 predictive aiming         |
| `parser.tokenize-comments`            | missions/parser           | Comment stripping               |
| `parser.nested-blocks`                | missions/parser           | Brace depth handling            |
| `cursor-stack.relative-offsets`       | missions/cursor-stack     | Position accumulation           |
| `event-evaluator.approach-trigger`    | missions/event-evaluator  | Distance-based trigger          |
| `action-processor.load-mission-chain` | missions/action-processor | Mission progression             |
| `save-manager.write-read`             | save/save-manager         | Save/load roundtrip             |
| `local-save-manager.indexeddb`        | save/local-save-manager   | Browser IndexedDB saves          |
| `server-save-manager.file-io`         | save/server-save-manager  | Server-side file storage         |
| `save-separation.upload-rejected`     | save/separation           | No client→server upload          |
| `save-separation.download-rejected`   | save/separation           | No server→client download        |
| `packet.serialize-deserialize`        | network/packet            | Binary packet format            |
| `steam-network.connect-send-receive`  | network/steam-network     | Steam SDR client connectivity   |
| `websocket-network.auth-packet`       | network/websocket-network | WebSocket auth + packet flow    |
| `bridge.translate-sdr-to-ws`          | network/bridge            | Protocol translation            |
| `relay-server.dual-transport`         | network/relay-server      | Server bridges both transports  |
| `openid-verify.check-ownership`       | network/openid-verify     | Steam Web API verification      |
| `integration.train01-load`            | integration/mission-load  | Full mission load from file     |
| `integration.combat-flow`             | integration/combat-flow   | Fire → hit → damage chain       |

### 16.4 Property-Based Tests (Invariant Suite)

```typescript
// tests/physics/invariant-suite.test.ts
describe('Physics Invariants', () => {
  it('velocity magnitude never negative', () => {
    const ship = new Ship();
    for (let i = 0; i < 1000; i++) {
      ship.update(1/60, mockInput);
      expect(ship.vel.magnitude()).toBeGreaterThanOrEqual(0);
    }
  });
  
  it('shield value stays within bounds', () => {
    const ship = new Ship();
    for (let i = 0; i < 1000; i++) {
      ship.takeDamage(50);
      ship.regenerateShields(1/60);
      expect(ship.shields).toBeGreaterThanOrEqual(-50); // Allow brief negative before death
      expect(ship.shields).toBeLessThanOrEqual(100);
    }
  });
});
```

---

## 17. Risk Register

| Risk                               | Impact                           | Mitigation                                                                             |
| ---------------------------------- | -------------------------------- | -------------------------------------------------------------------------------------- |
| **BabylonJS Y-up conversion bugs** | All positions/orientations wrong | Dedicated `toBabylonVec3`/`fromBabylonVec3` boundary functions; test with known values |
| **Mission file parser complexity** | Can't load original missions     | Start with simplest mission (train01), add features incrementally                      |
| **.tri model loading**             | Ships/stations don't render      | Implement basic .tri loader first, AC3D later                                          |
| **Performance with 65K stars**     | FPS drops                        | Distance culling, LOD at 200² and 5000² thresholds                                     |
| **Electron/Tauri build conflicts** | Can't test native builds         | Keep Electron and Tauri configs minimal; merge Vite output into both                   |
| **Network code complexity**        | Multiplayer takes too long       | Phase 10 is last; start with interfaces only, implement later                          |
| **Headless server in Node**        | BabylonJS dependency leak        | Server.ts has zero BabylonJS imports; pure data processing                             |
| **PPM texture loading**            | Planet textures fail             | Implement basic PPM reader; fallback to color if texture fails                         |
| **steamworks.js compatibility**    | Steam SDK won't load in Electron | Test early in Phase 0; have fallback to manual Steam auth via Web API                  |
| **Relay server complexity**        | Dual transport too hard          | Start with WebSocket-only, add Steam SDR as optional module                            |
| **OpenID auth flow issues**        | Browser players can't verify     | Fallback: simple password + SteamID64 check; upgrade to full OpenID later              |
| **Steam Web API rate limits**      | Server discovery fails           | Cache server list with TTL; retry with exponential backoff                             |

---

## Summary: Phase Dependencies

```
Phase 0 (Foundation)
    │
    ├──→ Phase 1 (Eye Candy) ──→ Phase 2 (Flight Physics) ──→ Phase 3 (Planets)
    │                              │                                │
    │                              ▼                                ▼
    │                         Phase 5 (Combat) ←── Phase 6 (AI)
    │                              │
    ├──→ Phase 4 (Mission Parser) ──→ Phase 8 (Events & Flow)
    │                                      │
    ├──→ Phase 7 (HUD) ──────────────────┘
    │
    ├──→ Phase 9 (Save & Polish) ──→ Phase 10 (Multiplayer) ──→ Phase 11 (Native Builds)
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
