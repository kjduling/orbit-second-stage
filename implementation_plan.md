# Implementation Plan — Orbit: Second Stage

## Overview

This document defines the step-by-step implementation of **Orbit: Second Stage**, a Babylon.js / TypeScript / Vite space combat simulator. The project is an inspired re-creation of the 1999 freeware game *Orbit* by Steve Belczyk — not a port, but a fresh implementation using modern web technologies. The first playable build recreates the Original Orbit experience (flight, combat, missions, solar system). Second Stage content (new missions, ships, weapons, story) is added incrementally after the core engine is solid.

**Key architectural decisions:**
- **Babylon.js full** (not Lite) — WebXR support requires the full module set
- **Z-up internal coordinate system** converted to Y-up at the Babylon.js rendering boundary
- **JSON mission files** replacing the original text-based `.msn` format — enables web serving, future mission editor tooling, and easier parsing
- **Interface-segregated architecture** — every game system has an `I*` interface with single-player and network variants from day one, even if network implementations are stubs initially
- **Electron as primary native target** — mature `steamworks.js` integration, established IPC patterns for Babylon.js ↔ Steam SDK bridge; Tauri deferred as secondary target
- **Placeholders first** — colored spheres, boxes, and basic geometries for early phases; real GLTF models swapped in later
- **Multiplayer-ready from day one** — interfaces designed for networked play, but implementations deferred to Phase 10

---

## Types

### Core Types

```typescript
// src/core/Vector3.ts
interface Vector3 {
  x: number; y: number; z: number;
}

// src/core/Matrix4.ts
interface Matrix4 { data: Float32Array; } // column-major, 16 elements

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

## Files

### New Files (Phase 0 — Foundation)

| File | Purpose |
|------|---------|
| `package.json` | Project manifest with Vite, TypeScript, Vitest, Electron, Babylon.js dependencies |
| `tsconfig.json` | TypeScript config: ES2020, strict mode, moduleResolution bundler |
| `vite.config.ts` | Vite config: dev server on port 5173, build to dist/, Electron preload bridge |
| `vitest.config.ts` | Vitest config: node environment, globals, coverage thresholds (90/85/90/90) |
| `eslint.config.js` | ESLint flat config: TypeScript parser, recommended rules |
| `.prettierrc` | Prettier config: 2-space indent, semicolons, single quotes |
| `electron/main.js` | Electron main process: window creation, preload bridge to renderer, Steam SDK init |
| `electron/preload.js` | Context bridge: exposes `window.electronAPI` to renderer with IPC handlers |
| `electron/package.json` | Electron-specific dependencies: electron, @electron/remote, steamworks.js |

### New Files (Core Library — Phase 0)

| File | Purpose |
|------|---------|
| `src/core/Vector3.ts` | Z-up internal vector math: add, subtract, dot, cross, normalize, scale, distance |
| `src/core/Matrix4.ts` | Column-major 4x4 matrix: identity, rotation (axis-angle), translation, multiply, transformVec3 |
| `src/core/Timer.ts` | Fixed-timestep accumulator: tick(), getDeltaT(), getAccumulator(), reset() |
| `src/core/EventBus.ts` | Typed pub/sub event system: subscribe(), emit(), unsubscribe() |
| `src/core/RNG.ts` | Mulberry32 seeded PRNG: seed(), next(), nextInt(), nextFloat() |
| `src/core/Logger.ts` | Structured logging with levels: debug(), info(), warn(), error(), setLevel() |
| `src/core/game-mode.ts` | GameMode enum + mode selector: getActiveMode(), setMode(), getMissionPath(mode) |

### New Files (Player System — Phase 1-2)

| File | Purpose |
|------|---------|
| `src/player/IShip.ts` | Ship interface: position, velocity, orientation, shields, state machine |
| `src/player/Ship.ts` | Core ship state implementation (single-player): update(), takeDamage(), regenerateShields() |
| `src/player/FlightModel.ts` | IFlightModel interface + NewtonianFlightModel + ArcadeFlightModel implementations |
| `src/player/Camera.ts` | Babylon.js ArcRotateCamera wrapper: updateFromShip(), setViewLock() |
| `src/player/InputManager.ts` | IInputManager interface + KeyboardInput + JoystickInput implementations |

### New Files (Solar System — Phase 1-3)

| File | Purpose |
|------|---------|
| `src/planets/Planet.ts` | Planet entity: position, velocity, radius, mass, orbit angle, rendering state |
| `src/planets/SolarSystem.ts` | ISolarSystem interface + SolarSystem implementation: init(), update(), getGravity() |
| `src/planets/Starfield.ts` | Starfield rendering: 65K+ stars with two density levels, distance-based culling |

### New Files (Weapons & Combat — Phase 5)

| File | Purpose |
|------|---------|
| `src/weapons/WeaponDef.ts` | Weapon definition data type + default weapon definitions |
| `src/weapons/Missile.ts` | Missile entity: position, velocity, lifetime, weaponDef, owner |
| `src/weapons/MissilePool.ts` | Object pool for missiles: acquire(), release(), updateAll() |
| `src/weapons/IWeaponSystem.ts` | IWeaponSystem interface: fireMissile(), updateWeapons(), checkCollisions() |
| `src/weapons/SinglePlayerWeaponSystem.ts` | SP weapon system implementation |
| `src/weapons/NetworkedWeaponSystem.ts` | Networked weapon system (stub for Phase 10) |

### New Files (AI Targets — Phase 6)

| File | Purpose |
|------|---------|
| `src/targets/Target.ts` | Target entity state: position, velocity, strategy, shields, AI state |
| `src/targets/Strategy.ts` | AIFStrategy enum + strategy registry |
| `src/targets/TurnToward.ts` | AI primitive: rotate target's viewDir toward a point |
| `src/targets/MoveToward.ts` | AI primitive: translate target toward a point respecting distance thresholds |
| `src/targets/FindEnemy.ts` | AI primitive: find closest enemy target by squared distance |
| `src/targets/Aim.ts` | AI primitive: predictive aiming — compute future player position |
| `src/targets/ITargetManager.ts` | ITargetManager interface: updateTargets(), getTargets(), fireMissiles() |
| `src/targets/SinglePlayerAIManager.ts` | SP AI manager implementation |
| `src/targets/NetworkedAIManager.ts` | Networked AI manager (stub for Phase 10) |

### New Files (Mission System — Phase 4)

| File | Purpose |
|------|---------|
| `src/missions/MissionParser.ts` | JSON mission file parser: parseMission(json), validateMission(data) |
| `src/missions/Mission.ts` | Parsed mission data structure (matches MissionFile type above) |
| `src/missions/MissionLoader.ts` | Mission loading from URL (web) or filesystem (Electron): loadMission(url) |

### New Files (HUD & UI — Phase 7)

| File | Purpose |
|------|---------|
| `src/hud/IHUD.ts` | IHUD interface: update(), render() |
| `src/hud/SinglePlayerHUD.ts` | SP HUD implementation using Babylon.js GUI or DOM overlay |
| `src/hud/Radar.ts` | Circular radar overlay: renderTargets(), renderPlayer() |
| `src/hud/ShieldBar.ts` | Shield status bar with color coding (green > 66%, yellow > 33%, red < 33%) |
| `src/hud/Crosshair.ts` | Center crosshair rendering |
| `src/hud/MessageOverlay.ts` | Briefing text and console message display |

### New Files (Effects — Phase 5)

| File | Purpose |
|------|---------|
| `src/effects/Explosion.ts` | Explosion entity: position, size, color, lifetime, icosahedron mesh |
| `src/effects/ExplosionPool.ts` | Object pool for explosions |
| `src/effects/SpaceJunk.ts` | Motion reference polygons (subtle rotating shapes in background) |

### New Files (Save System — Phase 9)

| File | Purpose |
|------|---------|
| `src/save/ISaveManager.ts` | ISaveManager interface: saveGame(entry), loadGame(filename), listSaves() |
| `src/save/SaveManager.ts` | SP save/load: IndexedDB for web, file I/O for Electron |
| `src/save/SaveEntry.ts` | Save data structure: mission, ship state, score, timestamp |

### New Files (Network Layer — Phase 10)

| File | Purpose |
|------|---------|
| `src/network/INetworkLayer.ts` | INetworkLayer interface: connect(), disconnect(), sendPacket(), onMessage() |
| `src/network/SPNetworkLayer.ts` | No-op network layer for single-player (returns immediately) |
| `src/network/Packet.ts` | Packet type definitions and serialization helpers |
| `src/network/SteamNetwork.ts` | Steam SDR network layer (Phase 10+) |
| `src/network/WebSocketNetwork.ts` | WebSocket network layer for browser clients (Phase 10+) |

### New Files (Game Orchestration — Phase 8)

| File | Purpose |
|------|---------|
| `src/game/IGame.ts` | Core game interface: init(), update(deltaT), render(), getState() |
| `src/game/GameState.ts` | GameState enum: MENU, BRIEFING, PLAYING, PAUSED, DEAD1, DEAD2, VICTORY, DEFEAT, LOADGAME |
| `src/game/SinglePlayerGame.ts` | Single-player game orchestrator: wires all systems together |
| `src/game/NetworkedGame.ts` | Networked game client (Phase 10+) |
| `src/game/GameFactory.ts` | Creates correct game mode based on GameMode enum |

### New Files (XR — Phase 1.7, 12)

| File | Purpose |
|------|---------|
| `src/xr/WebXRInline.ts` | Inline XR session manager: enterInlineXR(), exitInlineXR() — Phase 1.7 |
| `src/xr/WebXRManager.ts` | Full immersive VR session manager: enterXR(), exitXR() — Phase 12 |
| `src/xr/XRCameraSwap.ts` | Standard ↔ XR camera swap — Phase 12 |
| `src/xr/VRControllerInput.ts` | VR controller input mapping — Phase 12 |

### Prototype Files (Phase 1.0)

| File | Purpose |
|------|---------|
| `src/prototype/QuickScene.ts` | Quick visual prototype: sun, colored planets, ship wireframe, star points |

### Entry Points

| File | Purpose |
|------|---------|
| `src/index.ts` | Web entry point: boot Babylon.js scene, initialize game loop |
| `src/main.ts` | Electron/desktop entry point: same as index.ts but with Electron IPC bridge |

### Data Directory

| Path | Purpose |
|------|---------|
| `data/missions/original/` | Original Orbit missions in JSON format (initially: train01.json, combat01.json) |
| `data/missions/second-stage/` | Second Stage missions directory (empty initially, populated later) |
| `data/models/placeholder/` | Placeholder models: ship-box.glb, station-box.glb (swapped for real GLTF later) |
| `data/textures/placeholder/` | Placeholder textures: solid colors for planets |

### Test Files (mirrors src/ structure)

| File | Purpose |
|------|---------|
| `tests/core/vector3.test.ts` | Vector3: dot, cross, normalize, add, subtract, scale, distance, toBabylon, fromBabylon |
| `tests/core/matrix4.test.ts` | Matrix4: identity, rotation, translation, multiply, transformVec3 |
| `tests/core/timer.test.ts` | Timer: advance, getDeltaT, reset, interpolation factor |
| `tests/core/event-bus.test.ts` | EventBus: subscribe, emit, unsubscribe, one-time listeners |
| `tests/core/rng.test.ts` | RNG: seed determinism, range bounds, statistical distribution |
| `tests/core/logger.test.ts` | Logger: log levels, output formatting |
| `tests/core/game-mode.test.ts` | GameMode: getActiveMode, setMode, mission path resolution per mode |
| `tests/player/flight-model.test.ts` | FlightModel: delta-v accumulation, warp speed, gravity |
| `tests/player/ship.test.ts` | Ship: state transitions, shield regen, death timer |
| `tests/planets/starfield.test.ts` | Starfield: star count, brightness mapping, culling |
| `tests/missions/parser.test.ts` | MissionParser: JSON parsing, validation, missing fields |
| `tests/weapons/weapon-defs.test.ts` | WeaponDef: default definitions, speed/yield calculations |
| `tests/weapons/missile-pool.test.ts` | MissilePool: acquire/release, pool exhaustion |
| `tests/targets/strategy.test.ts` | Strategy: enum values, registry lookup |
| `tests/targets/turn-toward.test.ts` | TurnToward: rotation toward point, dot product checks |
| `tests/targets/move-toward.test.ts` | MoveToward: distance thresholds, forward/backward movement |
| `tests/targets/find-enemy.test.ts` | FindEnemy: closest target by squared distance, faction filtering |
| `tests/effects/explosion.test.ts` | Explosion: creation, lifetime, size range |
| `tests/save/save-manager.test.ts` | SaveManager: save/load cycle, IndexedDB fallback |
| `tests/xr/webxr-inline.test.ts` | WebXRInline: session start/stop, WebXR unsupported fallback |

---

## Functions

### Core Library Functions

```typescript
// src/core/Vector3.ts
function vec3(x: number, y: number, z: number): Vector3;
function vec3Add(a: Vector3, b: Vector3): Vector3;
function vec3Subtract(a: Vector3, b: Vector3): Vector3;
function vec3Scale(v: Vector3, s: number): Vector3;
function vec3Dot(a: Vector3, b: Vector3): number;
function vec3Cross(a: Vector3, b: Vector3): Vector3;
function vec3Magnitude(v: Vector3): number;
function vec3MagnitudeSquared(v: Vector3): number;
function vec3Normalize(v: Vector3): Vector3;
function vec3Distance(a: Vector3, b: Vector3): number;
function vec3ToBabylon(v: Vector3): BABYLON.Vector3;  // Z-up → Y-up: (x, -z, y)
function vec3FromBabylon(v: BABYLON.Vector3): Vector3; // Y-up → Z-up: (x, z, -y)

// src/core/Matrix4.ts
function mat4Identity(): Matrix4;
function mat4Translation(x: number, y: number, z: number): Matrix4;
function mat4RotationAxis(axis: Vector3, angle: number): Matrix4;
function mat4Multiply(a: Matrix4, b: Matrix4): Matrix4;
function mat4TransformVec3(m: Matrix4, v: Vector3): Vector3;

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

## Classes

### Core Classes

```typescript
// src/core/Vector3.ts
class Vector3 {
  x: number; y: number; z: number;
  constructor(x: number, y: number, z: number);
  static zero(): Vector3;
  static fromBabylon(v: BABYLON.Vector3): Vector3;
  toBabylon(): BABYLON.Vector3; // Z-up → Y-up: (x, -z, y)
  add(other: Vector3): Vector3; subtract(other: Vector3): Vector3;
  scale(scalar: number): Vector3; dot(other: Vector3): number; cross(other: Vector3): Vector3;
  magnitude(): number; magnitudeSquared(): number; normalize(): Vector3; clone(): Vector3;
}

// src/core/Matrix4.ts
class Matrix4 {
  data: Float32Array; // column-major, 16 elements
  constructor();
  static identity(): Matrix4;
  static translation(x: number, y: number, z: number): Matrix4;
  static rotationAxis(axis: Vector3, angle: number): Matrix4;
  multiply(other: Matrix4): Matrix4; transformVec3(v: Vector3): Vector3;
}

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

## Dependencies

### NPM Packages

| Package | Version | Purpose |
|---------|---------|---------|
| `babylonjs` | ^7.x | Core 3D engine (full version, not Lite) |
| `@babylonjs/core` | ^7.x | Babylon.js core modules (tree-shakeable) |
| `@babylonjs/loaders` | ^7.x | GLTF/GLB model loader |
| `@babylonjs/gui` | ^7.x | 2D/3D GUI for HUD elements |
| `typescript` | ^5.x | TypeScript compiler |
| `vite` | ^6.x | Build tool + dev server |
| `vitest` | ^3.x | Test runner (mirrors CharGen setup) |
| `@vitest/coverage-v8` | ^3.x | Coverage reporting |
| `electron` | ^33.x | Desktop app wrapper |
| `@electron/remote` | ^2.x | Electron renderer ↔ main process bridge |
| `steamworks.js` | ^0.9.x | Steam SDK via Rust N-API (Electron only) |
| `eslint` | ^9.x | Linting |
| `@typescript-eslint/eslint-plugin` | ^8.x | TypeScript ESLint rules |
| `prettier` | ^3.x | Code formatting |
| `typedoc` | ^0.27.x | API documentation generation |

### Key Dependency Decisions

- **Babylon.js full** — WebXRManager requires the full module set; Lite strips it out
- **Electron over Tauri** — mature `steamworks.js` integration, established IPC patterns; Tauri deferred as secondary target
- **Tree-shaking via Vite** — only imported Babylon.js modules end up in the bundle
- **No Babylon.js dependency in Server.ts** — headless server is pure Node.js

---

## Testing

### Testing Approach

- **Vitest** with `node` environment for all tests (Babylon.js rendering tested via visual inspection + integration tests)
- **Coverage thresholds:** 90% lines, 85% branches, 90% functions, 90% statements
- **Core library tests** run in Node (no DOM required) — Vector3, Matrix4, Timer, EventBus, RNG, Logger
- **Game logic tests** run in Node — Ship, FlightModel, Target AI, MissionParser, WeaponDef
- **Babylon.js integration tests** require DOM/JSDOM — scene setup, camera, mesh creation, rendering
- **XR tests** use JSDOM + mocked `navigator.xr` — session lifecycle, support detection

### Key Test Scenarios

| Test | Module | What It Verifies |
|------|--------|------------------|
| Vector3 Z-up ↔ Y-up conversion | `vector3.test.ts` | `toBabylon(0,1,0)` = `(0,0,1)`, `fromBabylon(0,0,1)` = `(0,1,0)` |
| Newtonian delta-v accumulation | `flight-model.test.ts` | Velocity persists across frames, warp coefficient applied correctly |
| Arcade velocity set directly | `flight-model.test.ts` | No momentum — velocity = throttle × direction |
| Ship state machine transitions | `ship.test.ts` | NORMAL → DEAD1 (shields < 0) → DEAD2 (timer expires) → NORMAL (respawn) |
| Shield regen rate | `ship.test.ts` | Shields increase at SHIELD_REGEN per second, capped at maxShields |
| Take damage reduces shields | `ship.test.ts` | Damage applied correctly, death triggered when shields < 0 |
| Mission JSON parsing | `parser.test.ts` | Valid mission parsed correctly, missing required fields rejected |
| Cursor position resolution | `parser.test.ts` | Named cursor resolves to planet position, offset applied correctly |
| AI turn toward target | `turn-toward.test.ts` | Target's viewDir rotates toward desired position at correct rate |
| Find closest enemy | `find-enemy.test.ts` | Returns closest non-friendly target, handles empty target list |
| Predictive aiming | `aim.test.ts` | Accounts for missile speed and target velocity |
| Missile collision with target | `missile-pool.test.ts` | AABB check in target's local space |
| Explosion object pooling | `explosion.test.ts` | Pool reuse, lifetime expiry, size clamping |
| Save/load round-trip | `save-manager.test.ts` | Ship state preserved across save/load cycle |
| WebXR support detection | `webxr-inline.test.ts` | Returns false when navigator.xr undefined, true when available |
| Game mode mission path resolution | `game-mode.test.ts` | ORIGINAL → 'data/missions/original/', SECOND_STAGE → 'data/missions/second-stage/' |

---

## Implementation Order

### Phase 0 — Foundation (Week 1)

1. **Project scaffolding** — `package.json`, `tsconfig.json`, `vite.config.ts`, `vitest.config.ts`, ESLint, Prettier
2. **Core math library** — `Vector3` (Z-up), `Matrix4` (column-major) with full test suite
3. **Core utilities** — `Timer` (fixed-timestep accumulator), `EventBus` (typed pub/sub), `RNG` (Mulberry32), `Logger`
4. **Game mode infrastructure** — `GameMode` enum, mode selector, mission path resolution per mode
5. **Electron setup** — `electron/main.js`, `electron/preload.js`, IPC bridge, `steamworks.js` integration scaffold
6. **Test infrastructure** — Vitest config with coverage thresholds, first core tests passing

### Phase 1 — Eye Candy (Week 2)

7. **Quick visual prototype** — `src/prototype/QuickScene.ts`: sun sphere, 8 colored planets, ship wireframe, 500 star points
8. **Babylon.js scene initialization** — `src/index.ts`: Engine, Scene, render loop, resize handler
9. **Starfield rendering** — `Starfield` class with distance-based culling (65K stars, two density levels)
10. **Ship model loading** — GLTF loader for placeholder ship mesh (box → real model swap later)
11. **Input manager** — `IInputManager` interface + `KeyboardInput` implementation (all original keys mapped)
12. **Camera wrapper** — `ArcRotateCamera` wrapper responding to input
13. **Inline WebXR session** — `WebXRInline` class: enter/exit inline XR, support detection, unit tests

### Phase 2 — Flight Physics & Controls (Week 3)

14. **Flight model base interface** — `IFlightModel` with Newtonian and Arcade implementations
15. **Newtonian physics** — delta-v accumulation, warp speed, superwarp (SP only)
16. **Arcade physics** — direct velocity setting, throttle range, full stop
17. **Ship state machine** — Ship class with INIT/NORMAL/DEAD1/DEAD2 states, shield regen, death timer
18. **Input axis mapping** — keyboard keys → yaw/pitch/roll/throttle axes, deadzone handling
19. **Gravity physics** — gravitational force calculation, singularity clamping, planet influence radius

### Phase 3 — Planets & Solar System (Week 4)

20. **Planet data structure** — `Planet` class with all properties from GDD
21. **Solar system initialization** — 8 planets + ~20 moons with NASA-based orbital data
22. **Planet rendering** — textured spheres with Babylon.js materials, distance-based LOD
23. **Planetary orbits** — angular velocity, orbit path drawing, moon parent relationships
24. **Gravitational physics integration** — gravity affects player, missiles, and targets
25. **Sun rendering** — emissive material + point light at origin

### Phase 4 — Mission System (Week 5)

26. **JSON mission parser** — `MissionParser.parse()`, validation, cursor resolution
27. **Mission loader** — URL fetch for web, file I/O for Electron, mode-based path resolution
28. **Mission data structures** — `MissionFile`, `CursorDef`, `ObjectDef`, `EventDef`, etc.
29. **Initial missions** — `data/missions/original/train01.json`, `combat01.json` (placeholder content)
30. **Mission directory structure** — `data/missions/original/`, `data/missions/second-stage/`

### Phase 5 — Combat & Weapons (Week 6-7)

31. **Weapon definitions** — Laser, PhotonRay, IonGun, Disruptor, Spare with all properties
32. **Missile entity** — position, velocity, lifetime, weaponDef, owner
33. **Missile pool** — object pooling, acquisition, release, oldest-reclaimed exhaustion
34. **Weapon system interface** — `IWeaponSystem` with SP and Network variants (network stubbed)
35. **SP weapon system** — fire missile, update weapons, collision detection (AABB in local space)
36. **Collision detection** — missile vs target (AABB), missile vs player (distance), player vs planet (radius)
37. **Explosion system** — icosahedron-based explosions with point light, object pooling
38. **Space junk** — motion reference polygons in background

### Phase 6 — AI Targets (Week 8-9)

39. **Target entity** — `Target` class with all properties from GDD
40. **AI strategy enum** — DO_NOTHING, SIT1-4, HUNT1-4
41. **TurnToward primitive** — rotate target's viewDir toward desired position
42. **MoveToward primitive** — translate target respecting distance thresholds (THINK_CUTOFFA/B/C)
43. **FindEnemy primitive** — closest non-friendly target by squared distance, faction filtering
44. **Aim primitive** — predictive aiming: compute future player position based on velocity
45. **AI manager** — `SinglePlayerAIManager` running strategies for all targets each frame
46. **Target rendering** — mesh instantiation from model files, hidden/invisible handling

### Phase 7 — HUD & UI (Week 10)

47. **HUD interface** — `IHUD` with SP and Network variants
48. **Circular radar** — target positions relative to player, friendly/enemy color coding
49. **Shield bar** — green/yellow/red color coding based on shield percentage
50. **Crosshair** — center screen crosshair rendering
51. **Message overlay** — briefing text, console messages with auto-scroll and duration
52. **Velocity display / throttle bar / score readout** — flight data overlays

### Phase 8 — Events & Game Flow (Week 11)

53. **Event evaluator** — trigger evaluation: Approach, Depart, True, Score, Destroy, Alarm, Shields
54. **Action processor** — action execution: Message, Hide, Unhide, Destroy, Score, Enable/Disable, Boom, Flash, LoadMission
55. **Game state machine** — GameState enum transitions: MENU → BRIEFING → PLAYING → DEAD1 → DEAD2 → VICTORY/DEFEAT
56. **Death/respawn sequence** — 5-second countdown in DEAD1, press fire to respawn in DEAD2
57. **Mission progression** — victory/defeat conditions, mission chain loading
58. **Game orchestration** — `SinglePlayerGame` wiring all systems together

### Phase 9 — Save System & Polish (Week 12)

59. **Save data structure** — `SaveEntry` with mode, mission, ship state, score, timestamp
60. **Save manager** — IndexedDB for web, file I/O for Electron, save/load/list operations
61. **Settings persistence** — difficulty selection, flight model preference, graphics settings
62. **Mission select UI** — browse and load missions from `data/missions/original/`
63. **Screen flash effects** — palette flash on damage, victory/defeat screen transitions
64. **Sound manager** — audio abstraction layer (Babylon.js Sound API)

### Phase 10 — Multiplayer Layer (Week 13-15)

65. **Network interface** — `INetworkLayer` with SP no-op and network variants
66. **SP network layer** — no-op implementation that resolves immediately
67. **Packet format** — `NetworkPacket` type definitions, serialization helpers
68. **Steam network layer** — Steam SDR via `steamworks.js`, lobby creation/joining
69. **WebSocket network layer** — browser client connection to relay server
70. **Player sync** — ship state broadcast, target state broadcast
71. **Server authority** — dedicated server with authoritative game state
72. **Matchmaking** — server discovery, lobby browser UI

### Phase 11 — Electron Production Build (Week 16)

73. **Electron production config** — packaging, code signing, auto-updater
74. **Steam overlay integration** — in-game Steam overlay for native builds
75. **Native launcher UI** — platform-specific window management, app menu

### Phase 12 — WebXR / VR Support (Week 17-18)

76. **WebXRManager integration** — immersive-vr session entry point with HMD tracking
77. **XRCameraSwap** — seamless standard ↔ XR camera swap
78. **VR controller input mapping** — thumbstick → yaw/pitch, trigger → thrust/fire, grip → roll
79. **VR flight model** — rotation-only (no artificial locomotion), warp via button combo
80. **XR DOM Overlay HUD panels** — 3D floating UI panels in headset
81. **Pointer-based HUD interaction** — controller ray casts to select buttons
82. **Solar system scale compression** — proportional distances readable at headset FOV
83. **VR comfort features** — vignette during warp, motion sickness prevention
84. **XR session lifecycle** — enter/exit gracefully, state preservation
85. **Multi-headset support** — Oculus Touch, Valve Index, Vive, Quest

---

> **Refer to `@design/Orbit.md` for the full game design document, `@design/implementation_plan.md` for the phased breakdown with Kanbn task mappings, and `@design/gap_analysis.md` for architectural decisions and risk mitigations.**
