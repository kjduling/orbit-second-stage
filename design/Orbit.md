# ORBIT — Game Design Document

> A comprehensive analysis of the original C/OpenGL game by Steve Belczyk (1999), intended as the reference for a BabylonJS/TypeScript/Vite remake.

---

## 1. Game Overview

### 1.1 Concept

**ORBIT** is a freeware space combat simulator where you play as "Sparky," humanity's top space fighter pilot, defending the solar system against an alien invasion. The game features:

- **Single-player mission-based campaigns** with progressive difficulty
- **Multiplayer network support** (up to 16 clients) for competitive/cooperative play
- **Newtonian or arcade flight physics** — two distinct flight models
- **Realistic planetary system** based on NASA data with gravitational physics
- **Extensible mission editor** via text-based mission files

### 1.2 Core Loop

1. Load a mission (training → campaign → victory/defeat)
2. Fly your ship through the solar system using Newtonian or arcade physics
3. Engage enemy targets in space combat
4. Complete mission objectives (destroy enemies, reach waypoints, survive)
5. Progress through mission chains (campaigns)

### 1.3 Setting

The game takes place in our **solar system** with:

- The Sun (center, light source)
- 8 major planets (Mercury through Neptune)
- ~20 moons (Phobos, Deimos, Io, Europa, Titan, Triton, etc.)
- Planetary rings on Jupiter, Saturn, Uranus, and Neptune
- A background starfield with ~65,000+ stars

---

## 2. Core Gameplay Mechanics

### 2.1 Player Ship

#### 2.1.1 Attributes

| Attribute          | Value/Range        | Description                                    |
| ------------------ | ------------------ | ---------------------------------------------- |
| **Name**           | "Sparky" (default) | Player's callsign, customizable via Ctrl+N     |
| **Shields**        | 0–100 (max)        | Regenerates at SHIELD_REGEN per second         |
| **Max Shields**    | 100.0              | Maximum shield capacity                        |
| **Score**          | Integer            | Accumulated from destroyed targets             |
| **Weapon**         | 0–3                | Currently selected weapon                      |
| **Flight Model**   | Newtonian / Arcade | Toggleable via 'f' key                         |
| **Position**       | double[3]          | X, Y, Z in world coordinates                   |
| **Velocity**       | double[3]          | Current velocity vector                        |
| **View Direction** | double[3]          | Forward vector                                 |
| **Up Vector**      | double[3]          | Up vector (maintains orthonormal basis)        |
| **Right Vector**   | double[3]          | Right-hand vector (cross product of up × view) |

#### 2.1.2 State Machine

```
STATE_NORMAL     → Active gameplay
STATE_DEAD1      → Just died, 5-second countdown
STATE_DEAD2      → Countdown over, press fire to respawn
STATE_INIT       → Initialization phase (first frame only)
STATE_LOADGAME   → Loading saved mission
```

### 2.2 Flight Models

#### 2.2.1 Newtonian Physics (`FLIGHT_NEWTONIAN`)

- **Velocity is persistent** — your ship maintains momentum
- **Acceleration**: `deltav = view_direction × DELTAV × deltaT` per second
- **DELTAV constant**: 0.2 units/second² (thrust acceleration)
- **Warp speed**: When holding forward at full throttle, velocity approaches warp
  - Normal warp: `deltaT × (move_forward × DELTAV + WARP_COEFF × |velocity|)`
  - Superwarp (single-player only): `100.0 × move_forward × DELTAV × DELTAV`
- **WARP_COEFF**: 0.2 (coefficient of current velocity added during warp)

#### 2.2.2 Arcade Physics (`FLIGHT_ARCADE`)

- **Velocity is set directly** by throttle — no momentum
- **Throttle range**: 0 to `MAX_THROTTLE` (10,000 km/s converted to game units)
- When warpspeed enabled: throttle can reach `MAX_WARP_THROTTLE` (1,000,000 km/s)
- **Full stop** (`SPACE` key): velocity = 0 in arcade mode

#### 2.2.3 Rotation (Both Models)

- **Theta rate**: 1.6 radians/second at full turn rate
- **Roll/pitch/yaw** via keyboard or joystick:
  - Left/Right: Yaw (rotate around up vector)
  - Up/Down: Pitch (rotate around right vector)
  - Pitch left/right: Bank (rotate around view vector)
- Rotation uses `RotateAbout()` — axis-angle rotation on the view/up/right vectors

### 2.3 Movement Controls

#### Keyboard

| Key                | Action                                        |
| ------------------ | --------------------------------------------- |
| **W / Up Arrow**   | Move forward (thrust)                         |
| **S / Down Arrow** | Move backward (reverse thrust)                |
| **A**              | Yaw left                                      |
| **D**              | Yaw right                                     |
| **Q**              | Pitch up                                      |
| **E**              | Pitch down                                    |
| **Z**              | Roll left                                     |
| **X**              | Roll right                                    |
| **SPACE**          | Full stop (Newtonian) / Throttle off (Arcade) |
| **Tab**            | Fire weapon                                   |
| **1-4**            | Select weapon                                 |
| **U**              | Lock onto nearest target                      |
| **Y**              | Cycle through visible targets                 |
| **F**              | Toggle flight model (Newtonian/Arcade)        |
| **V**              | Toggle view lock on locked target             |
| **B**              | Show briefing again                           |
| **M**              | Clear message from screen                     |
| **O**              | Toggle planet orbits display                  |
| **P / Ctrl+P**     | Pause game                                    |
| **L**              | Load saved game                               |
| **Ctrl+L**         | Type mission filename to load                 |
| **Ctrl+N**         | Change player name                            |
| **T**              | Chat (type message)                           |
| **S**              | Become server (if single-player)              |
| **C**              | Connect to server / type server address       |
| **U**              | Show connected clients                        |
| **F1**             | Show clients (same as U)                      |
| **> / <**          | Increase/decrease field of view               |
| **K / K**          | Increase/decrease time compression            |
| **F**              | Toggle FPS display                            |
| **o**              | Toggle orbit lines display                    |
| **[ / ]**          | Previous/Next waypoint                        |
| **{ / } / ( / )**  | Adjust near/far clipping planes               |
| **Q / Esc**        | Quit game                                     |

#### Joystick

- **X, Y**: Stick position → yaw/pitch
- **R**: Stick twist → roll
- **Z**: Throttle position → forward thrust
- Deadzone configurable in prefs (default ~0.1)
- Throttle mapping configurable (default 50-100%)

### 2.4 View Locking

Three lock modes (cycled with 'l'):

1. **LOCK_ENEMY** — Lock onto nearest enemy target
2. **LOCK_FRIENDLY** — Lock onto nearest friendly target
3. **LOCK_PLANET** — Lock onto nearest planet

When locked:

- Ship automatically aims toward the locked object
- HUD shows lock indicator and distance
- Locking enables targeting for weapons

---

## 3. Weapons System

### 3.1 Weapon Definitions

| Index | Name          | Speed (units/s) | Yield (damage) | Idle Time (s) | Lifetime (s) | Renderer        | Color          |
| ----- | ------------- | --------------- | -------------- | ------------- | ------------ | --------------- | -------------- |
| 0     | Laser         | 4000/KMU        | 20             | 0.2           | 1.0          | 1 (cross)       | Yellow (1,1,0) |
| 1     | PhotonRay     | 5000/KMU        | 30             | 0.3           | 0.75         | 2 (elongated)   | Blue (0,0,1)   |
| 2     | IonGun        | 4000/KMU        | 30             | 0.5           | 1.5          | 3 (tetrahedron) | Green (0,1,0)  |
| 3     | Disruptor     | 3000/KMU        | 60             | 1.0           | 1.5          | 4 (random loop) | Orange (1,1,0) |
| N+    | Spare (enemy) | 6000/KMU        | 15             | 2.0           | 1.0          | 0 (none)        | Red (1,0,0)    |

**KMU** = `KM_TO_UNITS1` — conversion factor from kilometers to game units.

### 3.2 Weapon Mechanics

- **Firing**: Press Tab or hold fire button
- **Idle timer**: Each weapon has a cooldown; must wait `idle` seconds between shots
- **Projectile lifetime**: Projectiles expire after `expire` seconds
- **Range**: Computed as `speed × expire` (squared for collision checks)
- **Velocity inheritance**: Missile velocity = `weapon_speed × direction + parent_velocity`
- **Friendly fire**: Friendly missiles don't damage friendly targets

### 3.3 Weapon Renderers

| Renderer | Visual      | Description                       |
| -------- | ----------- | --------------------------------- |
| 0        | None        | No renderer (enemy weapons)       |
| 1        | Cross       | Three perpendicular line segments |
| 2        | Ray         | Elongated quadrilateral/ray shape |
| 3        | Tetrahedron | Small spinning tetrahedron        |
| 4        | Random loop | Random hexagonal wireframe        |

### 3.4 Missile Pool

- **NMSLS** missiles allocated (fixed pool, recycled)
- Oldest missile reclaimed when pool exhausted
- Missiles destroyed on: expiration, planet impact, target hit, player hit

---

## 4. Combat & Damage

### 4.1 Shield System

- Player and targets have **shields** that regenerate over time
- Shield regen rate: `SHIELD_REGEN` per second (configurable)
- When shields drop below 0 on player → **death**
- Shields auto-capped at `maxshields`
- Visual indicator on HUD (green > 66%, yellow > 33%, red < 33%)

### 4.2 Damage Application

1. Missile hits target or player
2. Shield reduced by weapon's `yield` value
3. Screen flashes red (palette_flash = 2)
4. If shields < 0 and vulnerable → player dies
5. On death: explosion at player position, mission restart

### 4.3 Collision Detection

#### Missile vs Target (Axis-Aligned Bounding Box)

For each active target:

1. Transform missile position to target's local space (dot product with view/right/up)
2. Check if within model's `lobound` and `hibound` on all three axes
3. If within bounds on all axes → **hit**

#### Missile vs Player

- Distance check: `|missile_pos - player_pos| < TARGDIST` (target distance threshold)

#### Player/Target vs Planet

- Check if position is inside any planet's radius
- If inside → **crash death** (explosion + respawn at start)

### 4.4 Explosion System

- **NBOOMS** explosions allocated (fixed pool)
- Explosion shape based on icosahedron vertices (12 directions)
- Each explosion has randomized color and size
- Explosions spawn a point light that fades over time
- Size parameter controls visual scale (0.01–1.0 range)

---

## 5. AI & Target Strategies

### 5.1 Strategy Types

| Strategy            | Behavior                                                  | Difficulty |
| ------------------- | --------------------------------------------------------- | ---------- |
| **STRAT_DONOTHING** | Does nothing, sits still                                  | —          |
| **STRAT_SIT1**      | Turns toward player, shoots poorly                        | Easy       |
| **STRAT_SIT2**      | Aims at player's future position, shoots poorly           | Medium     |
| **STRAT_SIT3**      | Finds nearest enemy, turns toward it, shoots poorly       | Easy       |
| **STRAT_SIT4**      | Finds nearest enemy, aims at future position, shoots well | Hard       |
| **STRAT_HUNT1**     | Moves toward player, shoots                               | Medium     |
| **STRAT_HUNT2**     | Moves toward aiming point, shoots                         | Hard       |
| **STRAT_HUNT3**     | Moves toward closest enemy, shoots poorly                 | Easy       |
| **STRAT_HUNT4**     | Moves toward aiming point, shoots well                    | Hard       |

### 5.2 AI Thinking System

#### Distance Thresholds

| Constant      | Value      | Purpose                             |
| ------------- | ---------- | ----------------------------------- |
| THINK_CUTOFFA | 20,000/KMU | Ignore targets beyond this distance |
| THINK_CUTOFFB | 2,000/KMU  | Pursue targets within this distance |
| THINK_CUTOFFC | 500/KMU    | Maintain minimum distance           |

#### TurnToward Logic

1. Compute vector from target to desired position
2. If beyond THINK_CUTOFFA → stop moving (too far)
3. Normalize direction vector
4. Project onto target's right/up/view axes using dot products
5. Set turn rate based on which direction the target is
6. If facing target (dot(view, to_target) > 0.9) AND in range → **fire missile**

#### MoveToward Logic (in addition to TurnToward)

1. If beyond THINK_CUTOFFB → move forward at `maxvel`
2. If closer than THINK_CUTOFFC → stop (maintaining distance)
3. If facing away (dot(view, to_target) < 0) → move backward

#### FindEnemy Logic

1. Loop through all active targets
2. Filter: age > 0, not hidden, different faction (friendly != target_friendly)
3. Find closest by squared distance
4. Non-friendly targets also check against player position

### 5.3 Target Firing

- Targets fire when: facing target (theta > 0.9) AND within weapon range
- Uses same `FireMissile()` as player
- Target's weapon: `NPLAYER_WEAPONS` (spare/enemy weapon type) by default
- Target shield regen configurable per target

---

## 6. Planets & Solar System

### 6.1 Planet Structure

```c
struct {
    char name[32];        // Planet name (e.g., "Earth", "Mars")
    double pos[3];        // Position in world space
    double vel[3];        // Velocity for orbiting
    double up[3];         // Orientation
    double view[3];       // Viewing direction
    double right[3];      // Right-hand vector
    double radius;        // Physical radius (in game units)
    double radius2;       // radius² (precomputed)
    double mass;          // Gravitational mass (radius³)
    double color[3];      // Default color (when no texture)
    double dist;          // Distance from primary body
    double angvel;        // Angular velocity (degrees/s)
    double theta;         // Current orbital angle
    int oblicity;         // Axial tilt in degrees
    int is_moon;          // True if this is a moon
    int primary;          // Index of parent planet
    char texfname[128];   // Texture file path
    int custom;           // True if texture was customized
    int hidden;           // True if planet is hidden
    int list320,list80,list20; // Display lists (distance-based LOD)
    int orbitlist;        // Display list for orbit path
    double range2, absrange2; // Distance from player (squared)
} planet[NPLANETS];
```

### 6.2 Planet Rendering (Distance-Based LOD)

| Distance (r²) | Rendering Method                |
| ------------- | ------------------------------- |
| < 5²          | Textured sphere (high detail)   |
| < 25²         | Textured sphere (medium detail) |
| < 200²        | Textured sphere (low detail)    |
| < 5000²       | Colored point (2px)             |
| < 50000²      | Colored point (1px)             |

### 6.3 Planetary Orbits

- Planets orbit in the XZ plane around their primary
- Orbital speed: `angvel` degrees/second
- Orbit path drawn as line loop when `draw_orbits` is enabled
- Moon orbits drawn relative to parent planet position
- Oblicity (axial tilt) applied to both planet and its moons

### 6.4 Planetary Rings

Four ring systems: Jupiter, Saturn, Uranus, Neptune

- Each has inner radius (`r1`), outer radius (`r2`), texture file
- Rendered as textured quad strips with alpha blending
- Distance culling at `MAX_RING_RANGE`
- Oblicity inherited from parent planet

### 6.5 Gravitational Physics

- **Gravity constant G**: Configurable (default enabled via `gravity` flag)
- **Formula**: `dv = G × mass / r² × direction_vector`
- Applied per-frame: `deltav += Σ(G × planet[i].mass / |dp|² × dp/|dp|)` for each planet within 200×radius
- Minimum distance clamped to `RMIN` to prevent singularity
- Affects player, missiles, and targets (in network games)

---

## 7. Mission System

### 7.1 Mission File Format

Text-based mission files with brace-delimited blocks and token-based parsing. Supports comments (`/* ... */`) and include files (up to 16 levels deep).

#### Top-Level Tokens

| Token               | Description                                                       |
| ------------------- | ----------------------------------------------------------------- |
| `Cursor { ... }`    | Sets the reference point for subsequent Object/Player definitions |
| `Player { }`        | Defines player starting position relative to cursor               |
| `Object { ... }`    | Defines a target/object in the mission                            |
| `Briefing { ... }`  | Mission briefing text displayed to player                         |
| `Waypoint { }`      | Creates a waypoint at current cursor position                     |
| `Event { ... }`     | Defines a game event with trigger and actions                     |
| `Planet { Name X }` | Modifies an existing planet's properties                          |
| `Weapon { ... }`    | Custom weapon definition                                          |
| `Verbose` / `Terse` | Enable/disable verbose mission loading logs                       |
| `Include filename`  | Include another mission file                                      |

#### Cursor Syntax

```
Cursor { Name ±X ±Y ±Z }
Cursor { Name ±X }      // Only X offset
Cursor { Name }         // Just reference a named planet
```

### 7.2 Object Definition

```
Object {
    Name <name>
    Model <model_file.tri>
    Score <value>
    Strategy <strategy_name>
    Friendly
    Hidden
    Weapon <weapon_index>
    MaxShields <value>
    ShieldRegen <value>
    TurnRate <value>
    Speed <value>
    Invisible
}
```

#### Object Properties

| Property        | Type   | Description                                    |
| --------------- | ------ | ---------------------------------------------- |
| **Name**        | string | Unique identifier for targeting/events         |
| **Model**       | string | .tri or .ac model file from models/ directory  |
| **Score**       | int    | Points awarded on destruction                  |
| **Strategy**    | enum   | AI behavior (Sit1-4, Hunt1-4, DoNothing)       |
| **Friendly**    | bool   | True = allied, False = enemy                   |
| **Hidden**      | bool   | Initially hidden from view                     |
| **Weapon**      | int    | Weapon index (defaults to spare/enemy weapons) |
| **MaxShields**  | float  | Maximum shield capacity                        |
| **ShieldRegen** | float  | Shield regeneration rate                       |
| **TurnRate**    | float  | Rotation speed multiplier                      |
| **Speed**       | float  | Maximum forward velocity                       |
| **Invisible**   | bool   | Not shown on radar or with name labels         |

### 7.3 Event System

#### Event Structure

```
Event {
    Name <name>              // Optional identifier
    Trigger <type>           // What triggers this event
    Value <value>            // Parameter for trigger
    Enabled / Disabled       // Initial state
    Action <action_type>     // What happens when triggered
    Value <value>            // Parameter for action
}
```

#### Triggers

| Trigger    | Parameter    | Condition                                                             |
| ---------- | ------------ | --------------------------------------------------------------------- |
| `Approach` | distance     | Player comes within distance of event position                        |
| `Depart`   | distance     | Player moves beyond distance from event position                      |
| `True`     | —            | Always triggers (one-shot)                                            |
| `Score`    | score_value  | Player's score reaches or exceeds value                               |
| `Destroy`  | target_name  | Named target is destroyed                                             |
| `Alarm`    | seconds      | Timer expires after N seconds                                         |
| `StopNear` | distance     | Player velocity near zero AND within distance of event position       |
| `Shields`  | shield_value | Target's shields drop to or below value (checked in MissileHitTarget) |

#### Actions

| Action         | Parameter   | Effect                              |
| -------------- | ----------- | ----------------------------------- |
| `Message`      | text string | Display message to player           |
| `Hide`         | target_name | Hide named target                   |
| `Unhide`       | target_name | Show hidden target                  |
| `Destroy`      | target_name | Destroy named target                |
| `Score`        | points      | Add points to player score          |
| `Enable`       | event_name  | Enable a disabled event             |
| `Disable`      | event_name  | Disable an enabled event            |
| `Stop`         | —           | Set player velocity to zero         |
| `LoadMission`  | filename    | Load and start a new mission        |
| `Boom`         | size        | Create explosion at event position  |
| `Flash`        | —           | Screen flash effect                 |
| `MoveObject`   | target_name | Move named target to event position |
| `MovePlayer`   | —           | Move player to event position       |
| `MovePlanet`   | planet_name | Move planet to event position       |
| `HidePlanet`   | planet_name | Hide planet                         |
| `UnhidePlanet` | planet_name | Show hidden planet                  |
| `Betray`       | target_name | Toggle target's friendly status     |

### 7.4 Planet Modification in Missions

```
Planet {
    Name <planet_name>       // Select planet to modify
    NewName <new_name>       // Rename the planet
    Reposition               // Move to cursor position
    Hidden                   // Hide this planet
    Map <texture_file.ppm>   // Change texture map
    Oblicity <degrees>       // Set axial tilt
    Radius <km_value>        // Set radius (in km, auto-converted)
}
```

### 7.5 Campaign Structure

The game features several campaign chains:

**Training Chain:**

- `train01.msn` → Earth orbit to Moon and back
- `train02.msn` → Continued training
- `train03.msn` → Advanced training
- `train04.msn` → Combat basics (destroy 4 platforms)

**Jupiter Campaign:**

- `jupiter1.msn` → Proceed to Io station
- `jupiter2.msn` → Continue Jupiter missions
- `jupiter3.msn` → Continue
- `jupiter4.msn` → Continue
- `jupiter5.msn` → Final Jupiter mission

**Saturn Campaign:**

- `saturn01.msn` through `saturn04.msn`

**Titan Campaign:**

- `titan01.msn` through `titan03.msn`

**Mars Campaign:**

- `mars1.msn` through `mars5.msn`

**End States:**

- `win.msn` — Victory (museum of all models)
- `lose.msn` — Defeat (Earth destroyed)

---

## 8. HUD (Heads-Up Display)

### 8.1 HUD Elements

| Element                | Position         | Description                                                |
| ---------------------- | ---------------- | ---------------------------------------------------------- |
| **Radar**              | Center           | Circular radar showing targets, planets, friendly units    |
| **Throttle Bar**       | Left of radar    | Vertical bar showing current throttle level                |
| **Velocity Readout**   | Left side        | Current velocity in km/s (green = forward, red = backward) |
| **Shields Bar**        | Right of radar   | Player shield status (color-coded)                         |
| **Target Shields Bar** | Right of shields | Target's shield status (when locked)                       |
| **Target Name**        | Right side       | Name and distance to locked target                         |
| **Target Range**       | Below name       | Distance in km (when within weapon range)                  |
| **Waypoint Info**      | Below range      | Waypoint number and distance                               |
| **Weapon Name**        | Bottom right     | Current weapon name (purple when ready, gray when cooling) |
| **Crosshair**          | Screen center    | Small bitmap crosshair                                     |
| **Briefing Message**   | Center screen    | Orange text, auto-wrapping, multi-line                     |
| **Console Messages**   | Bottom           | Green scrolling log messages                               |
| **FPS Counter**        | Top (optional)   | Frames per second display                                  |

### 8.2 Radar Details

- Circular radar centered on screen
- Shows all targets (colored by faction: red=enemy, green=friendly)
- Shows planets (blue for planets, cyan for moons)
- Range ring at configurable distance
- Player position always at center
- Target icons move relative to player's position

---

## 9. Models & Visuals

### 9.1 Model Formats

#### Triangle Files (.tri)

```
v1x v1y v1z  v2x v2y v2z  v3x v3y v3z  hex_color
```

- One triangle per line
- Vertices in counter-clockwise order (exterior view)
- Color as 24-bit hex (0xRRGGBB)
- Coordinates divided by 100.0 for final scale
- Normal computed from cross product of edge vectors

#### AC3D Files (.ac)

- Full AC3D model format support
- Materials and textures supported
- SGI "rgb" texture format only
- Based on Steve Baker's ac-to-gl converter

### 9.2 Model Types Available

| Model File                            | Type                 | Notes            |
| ------------------------------------- | -------------------- | ---------------- |
| `light1.tri`, `light2.tri`            | Light fighters       | Small, fast      |
| `medium1.tri`                         | Medium fighters      | Balanced         |
| `heavy1.ac`                           | Heavy ships          | AC3D model       |
| `fight1.ac`                           | Fighter              | AC3D model       |
| `freight1.ac`, `freight1.tri`         | Freighters           | Large            |
| `station1.tri` through `station4.tri` | Space stations       | Various designs  |
| `base1.tri`, `base2.tri`              | Bases                |                  |
| `platform.tri`                        | Platforms            | Training targets |
| `solar1.ac`, `solar1.tri`             | Solar arrays         |                  |
| `solar2.tri`                          | Solar panels         |                  |
| `comm1.tri`                           | Communications array |                  |
| `artifact1.ac`, `artifact1.tri`       | Alien artifact       | Key story object |
| `portal1.ac`                          | Portal               | Story object     |
| `eject1.ac`                           | Ejection module      |                  |
| `dest1.tri`                           | Destroyed ship       |                  |
| `null.tri`                            | Null model           | Placeholder      |

### 9.3 Model Rendering

- Models loaded into OpenGL display lists
- Bounding box computed for collision detection
- In network games, models scaled to uniform radius (0.030 units)
- Triangle strips generated for efficient rendering
- Material colors from hex values in .tri files

### 9.4 Starfield

- Two levels: dense (~65K stars) and sparse (~6.5K stars)
- Stars have magnitudes mapped to brightness
- Dense/sparse selectable via `starfield` preference (0=sparse, 2=dense)
- Stars rendered as points with depth sorting (brightest last)
- Based on Raul Alonso's ssystem stellar database

### 9.5 Visual Effects

- **Explosions**: Icosahedron-based particle explosions with point lights
- **Space junk**: Subtle rotating reference polygons for motion cues
- **Palette flash**: Screen flash on damage (red for hit, white for event flash)
- **Lighting**: Two-point lighting setup (ambient + directional)
- **Depth of field**: Near/far clipping adjustable via keys
- **Field of view**: Adjustable 5°–180° (default configurable)

---

## 10. Audio System

### 10.1 Sound Effects

| Sound              | File               | Trigger                      |
| ------------------ | ------------------ | ---------------------------- |
| Phaser/Weapon fire | `phaser.wav`       | Firing any weapon            |
| Explosion          | `explosion1.wav`   | Target destroyed, player hit |
| Communications     | `communicator.wav` | Messages displayed           |

### 10.2 Audio Configuration

- Sound enabled/disabled via `sound` preference
- Uses system audio (platform-dependent)
- No positional/3D audio in original implementation

---

## 11. Network / Multiplayer

### 11.1 Architecture

- **Server-authoritative**: One player hosts as server, others connect as clients
- **Up to 16 clients** (`NCLIENTS = 16`)
- **Port**: 2061 (configurable)
- Binary packets for position data, ASCII for commands

### 11.2 Network Packets

#### Binary Packet Format

```
[0x56] [size_byte] [packet_data...]
```

Magic byte `0x56` marks start of packet.

#### Packet Types

| Type       | Byte | Description                               |
| ---------- | ---- | ----------------------------------------- |
| `PKT_POSS` | 0x80 | Short position report                     |
| `PKT_POSL` | 0x81 | Long position report                      |
| `PKT_POSU` | 0x82 | Urgent position report                    |
| `PKT_POSN` | 0x83 | Regular position report                   |
| `PKT_PLAN` | 0x84 | Planet position data                      |
| `PKT_VCNT` | 0x85 | Vacancy report (server slot availability) |
| `PKT_PING` | 0x86 | Latency ping                              |

#### ASCII Packet Types

| Command               | Format                           | Description |
| --------------------- | -------------------------------- | ----------- |
| `FIRE <weapon>`       | Player fired a weapon            |
| `CHAT <text>`         | Chat message                     |
| `CMSG <name>: <text>` | Chat message from another player |
| `DROP <client_id>`    | Drop a client (server only)      |
| `NAME <name>`         | Change player name               |

### 11.3 Network Timing

| Constant            | Value | Purpose                   |
| ------------------- | ----- | ------------------------- |
| `MAXCLIENTIDLE`     | 60s   | Drop idle clients         |
| `MAXSERVERIDLE`     | 120s  | Server timeout            |
| `PINGINTERVAL`      | 20s   | Latency ping interval     |
| `CLIENTPOSINTERVAL` | 0.33s | Position report interval  |
| `ROLLCALLINTERVAL`  | 30s   | Client roll call interval |

### 11.4 Position Reporting

- **Urgent reports**: When player transitions from still to moving or vice versa
- **Regular reports**: At `posninterval` rate (configurable per client)
- Position data includes: position, velocity, orientation vectors, movement flags
- Targets sync player position to their own target list

### 11.5 Network Game Flags

| Flag                 | Value | Description        |
| -------------------- | ----- | ------------------ |
| `FLAG_GRAVITY`       | 0x01  | Gravity enabled    |
| `FLAG_FLIGHTMODEL`   | 0x02  | Flight model type  |
| `FLAG_FULLSTOP`      | 0x04  | Full stop behavior |
| `FLAG_REALDISTANCES` | 0x08  | Real distance mode |
| `FLAG_ORBIT`         | 0x10  | Orbiting enabled   |

---

## 12. Save System

### 12.1 Save File Format

```
missions/train01.msn 1699900800
missions/jupiter1.msn 1699987200
```

- One entry per line: `mission_filename timestamp`
- Most recent save at top of file
- Max entries: `NSAVES` (configurable)

### 12.2 Loading Saves

- Press 'L' to bring up save menu
- Numbered list with mission name and timestamp
- Press corresponding digit key to load
- Loading resets game state and reads new mission

---

## 13. Configuration & Preferences

### 13.1 prefs.txt Format

```
mission train01.msn
screenwidth 640
screenheight 480
fullscreen 0
gamemode no
name Sparky
model light2.tri
vulnerable 1
fov 60.0
joy_throttle 50
deadzone 0.1
starfield 2
drawhud 1
showfps 0
gravity 1
draw_orbits 0
orbit 0
compression 1.0
junk 1
sound 1
show_names 1
screen_shot_num 0
rings 1
ring_sectors 64
textures 1
realdistances 0
weapon 0
mouse_control 0
mouse_flipx 0
mouse_flipy 0
flightmodel 0
fullstop 1
superwarp 1
port 2061
posx -15000.0
posy 0.0
posz 0.0
slices0 32
stacks0 16
slices1 32
stacks1 16
slices2 32
stacks2 16
```

### 13.2 Key Preferences

| Preference      | Type   | Default | Description                       |
| --------------- | ------ | ------- | --------------------------------- |
| `gravity`       | int    | 1       | Enable gravitational physics      |
| `orbit`         | int    | 0       | Enable planetary orbits animation |
| `flightmodel`   | int    | 0       | 0=Newtonian, 1=Arcade             |
| `starfield`     | int    | 2       | 0=sparse, 2=dense                 |
| `drawhud`       | int    | 1       | Show HUD                          |
| `show_names`    | int    | 1       | Show planet/target names          |
| `junk`          | int    | 1       | Show space junk reference         |
| `rings`         | int    | 1       | Show planetary rings              |
| `textures`      | int    | 1       | Use texture maps                  |
| `realdistances` | int    | 0       | Real distance mode                |
| `superwarp`     | int    | 1       | Enable superwarp speed            |
| `fullstop`      | int    | 1       | Full stop behavior                |
| `compression`   | double | 1.0     | Time compression multiplier       |
| `fov`           | double | 60.0    | Field of view in degrees          |

---

## 14. Data Structures Summary

### 14.1 Core Constants

| Constant            | Value       | Description                 |
| ------------------- | ----------- | --------------------------- |
| `VERSION`           | "1.01"      | Game version                |
| `THETA`             | 1.6         | Rotation rate (rad/s)       |
| `DELTAV`            | 0.2         | Max delta-v per second      |
| `WARP_COEFF`        | 0.2         | Warp velocity coefficient   |
| `MAX_THROTTLE`      | 10000/KMU   | Max throttle (km/s)         |
| `MAX_WARP_THROTTLE` | 1000000/KMU | Max warp throttle (km/s)    |
| `SCREENWIDTH`       | 640         | Default screen width        |
| `SCREENHEIGHT`      | 480         | Default screen height       |
| `DEAD_TIME`         | 5.0         | Seconds before respawn      |
| `NPLANETS`          | ~30         | Number of planets/moons     |
| `NTARGETS`          | ~64         | Max simultaneous targets    |
| `NMSLS`             | ~128        | Max simultaneous missiles   |
| `NBOOMS`            | ~32         | Max simultaneous explosions |
| `NWEAPONS`          | ~16         | Total weapon definitions    |
| `NPLAYER_WEAPONS`   | 4           | Player-accessible weapons   |
| `NMODELS`           | ~64         | Max loaded models           |
| `NWAYPOINTS`        | 32          | Max waypoints               |
| `NEVENTS`           | ~32         | Max simultaneous events     |
| `ACTIONS_PER_EVENT` | ~8          | Max actions per event       |
| `NSTARS`            | ~65536      | Star count                  |
| `NCLIENTS`          | 16          | Max network clients         |
| `NRINGS`            | 4           | Ring systems                |
| `ORBIT_SECTORS`     | 64          | Orbit display resolution    |

### 14.2 Key Global State Variables

```c
int state;                    // Current game state
double deltaT;                // Time since last frame
double absT;                  // Absolute time
double compression;           // Time compression multiplier
int paused;                   // Game paused flag
int vulnerable;               // Player can take damage
int gravity;                  // Gravity enabled
int orbit;                    // Planets orbiting
int draw_orbits;              // Show orbit paths
int show_names;               // Show name labels
int drawhud;                  // Show HUD
int showfps;                  // Show FPS counter
int starfield;                // Star density (0=sparse, 2=dense)
int junk;                     // Show space junk
int sound;                    // Sound enabled
int rings;                    // Rings enabled
int textures;                 // Textures enabled
int mouse_control;            // Mouse control enabled
int fov;                      // Field of view
double clipnear, clipfar;     // Clipping planes
int palette_flash;            // Screen flash (0=none, 1=white, 2=red)
```

---

## 15. Game Flow

### 15.1 Initialization Sequence

1. `main()` → seed RNG, init joystick, timer, player
2. `InitStuff()` → initialize all subsystems
3. `OpenWindow()` → create OpenGL window, set up GL state
4. First frame: `CreateUniverse()` → init keyboard, mouse, starfield, planets, textures, HUD, sound, network
5. Load default or specified mission

### 15.2 Per-Frame Loop (DrawScene)

1. `DeltaTime()` — compute delta time
2. `DoNetwork()` — handle network I/O
3. `Keyboard()` / `Joystick()` — read input
4. `DoMouse()` — handle mouse input
5. `MovePlanets()` — update planet positions if orbiting
6. `MovePlayer()` — update player physics
7. `UpdatePlayer()` — shields, weapon cooldowns, dead timer
8. `DoEvents()` — check event triggers
9. `ThinkTargets()` — AI decision making
10. `MoveTargets()` — update target physics
11. `MoveMissiles()` — update missile positions
12. `DrawScene()` → render:
    a. Clear buffers, set camera
    b. Draw starfield
    c. Draw planets (with LOD)
    d. Draw rings
    e. Draw models (targets, stations)
    f. Draw missiles
    g. Draw explosions
    h. Draw space junk
    i. Draw names
    j. Draw HUD
    k. Draw message/console
    l. Swap buffers

### 15.3 Death & Respawn

1. Player shields < 0 → `state = STATE_DEAD1`
2. Dead timer counts down from 5 seconds
3. Explosion at player position
4. After timer: `state = STATE_DEAD2`
5. Press fire → respawn, reset mission
6. On planet crash → same death sequence

---

## 16. Implementation Notes for BabylonJS Port

### 16.1 Architecture Recommendations

#### Core Systems

```
src/
├── core/           # Game engine foundation
│   ├── Vector3.ts
│   ├── Matrix4.ts
│   ├── Timer.ts
│   └── Physics.ts
├── player/         # Player ship
│   ├── Ship.ts
│   ├── FlightModel.ts
│   └── Camera.ts
├── weapons/        # Weapon system
│   ├── Weapon.ts
│   ├── Missile.ts
│   └── WeaponPool.ts
├── targets/        # AI targets
│   ├── Target.ts
│   ├── AI.ts
│   └── Strategy.ts
├── planets/        # Solar system
│   ├── Planet.ts
│   ├── RingSystem.ts
│   └── SolarSystem.ts
├── missions/       # Mission system
│   ├── MissionParser.ts
│   ├── Mission.ts
│   ├── Event.ts
│   └── EventHandler.ts
├── hud/            # HUD rendering
│   ├── Radar.ts
│   ├── ShieldBar.ts
│   └── HUD.ts
├── effects/        # Visual/audio effects
│   ├── Explosion.ts
│   ├── Starfield.ts
│   └── AudioManager.ts
├── network/        # Multiplayer (optional MVP)
│   ├── NetworkManager.ts
│   └── Packet.ts
├── save/           # Save system
│   └── SaveManager.ts
└── main.ts         # Entry point
```

#### Key BabylonJS Adaptations

- **Coordinate system**: BabylonJS uses Y-up (original uses Z-up). Need to convert all positions/orientations.
- **Rendering**: Replace OpenGL display lists with BabylonJS meshes/materials
- **Physics**: Implement custom physics engine (BabylonJS has built-in physics but Newtonian space flight may need custom implementation)
- **Input**: Replace GLUT keyboard/joystick with Web InputManager
- **UI**: Build HUD using BabylonGUI or HTML overlay
- **Models**: Convert .tri/.ac formats to GLTF/BABYLON format
- **Textures**: Convert PPM textures to PNG/WebP

### 16.2 Mission File Parser

The mission file format is a simple token-based parser with brace-delimited blocks. This should be straightforward to port:

- Token stream extraction (whitespace-separated, comment-aware)
- Brace-depth tracking for nested blocks
- `Cursor` position accumulation
- Include file resolution
- Event/action processing

### 16.3 AI Strategy Port

The AI system is pure function-based with clear state machines. Each strategy maps directly to a class or function:

- `TurnToward()` → compute rotation toward target
- `MoveToward()` → add movement toward target
- `FindEnemy()` → closest-target search
- `Aim()` → predictive aiming calculation

### 16.4 Gravitational System

- Newtonian gravity is the core physics feature
- Formula: `F = G × m₁ × m₂ / r²`
- Each planet contributes to total gravitational force on player/targets/missiles
- Configurable via `gravity` flag

---

## Appendix A: Campaign Mission Chain Map

```
Training → Jupiter → Saturn → Titan → Mars → Win/Lose

train01 → train02 → train03 → train04 → titan01
                                                        ↓
jupiter1 → jupiter2 → jupiter3 → jupiter4 → jupiter5 → (varies)
                                                       ↓
mars1 → mars2 → mars3 → mars4 → mars5 → win.msn

saturn01 → saturn02 → saturn03 → saturn04

titan01 → titan02 → titan03
```

## Appendix B: Planet List (from maps/ directory)

Sun, Mercury, Venus, Earth, Moon, Mars, Phobos, Deimos, Jupiter, Io, Europa, Ganymede, Callisto, Saturn, Mimas, Enceladus, Tethys, Dione, Rhea, Titan, Iapetus, Hyperion, Uranus, Miranda, Ariel, Umbriel, Titania, Oberon, Neptune, Triton, Pluto, Charon

## Appendix C: Original Game Credits

- **Creator**: Steve Belczyk (<orbit@genesis.nred.ma.us>)
- **Version**: 1.01
- **License**: GNU GPL v2+
- **Engine**: OpenGL + GLUT
- **Model format**: AC3D / custom .tri
- **Data sources**: NASA planetary images, Raul Alonso ssystem star data
