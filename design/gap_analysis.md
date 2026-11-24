# Gap Analysis: Original Orbit GDD vs Orbit: Second Stage Implementation Plan

> Generated: 2026-11-23 | Updated: 2026-11-23 (Gap Analysis Round 2) | Status: Planning Phase — No Source Code Yet

---

## Executive Summary

The original Orbit game design document describes a **simple, self-hosted TCP multiplayer** architecture with straightforward file-based saves and native platform assets (.tri/.ac models, PPM textures, WAV audio). The Orbit: Second Stage implementation plan introduces a more complex **hybrid network architecture** (Steam + WebRTC), interface-segregated design, and ECS-lite patterns — but leaves critical gaps in protocol bridging, NAT traversal, asset creation, and platform-specific behavior.

Orbit: Second Stage is an inspired re-creation of the original game mechanics. It is not a port of the original C code, and no copyrighted material from the 1999 original is used. The original GDD serves as a reference for game design inspiration only.

**Current Project State:** Design/planning phase only. No source code exists. Kanbn board has ~67 tasks across all phases, all in Backlog. Three design documents exist: GDD (Orbit.md), implementation plan, and this gap analysis.

This document identifies all gaps between the two documents, categorizes them by severity, and records decisions made to close them.

---

## 1. Critical Gaps (Decisions Required)

### 1.1 Network Provider Architecture — 🔴 CLOSED

| Aspect                  | GDD Description              | Plan Description                           | Gap                                          |
| ----------------------- | ---------------------------- | ------------------------------------------ | -------------------------------------------- |
| **Native builds**       | Self-hosted TCP on port 2061 | `TCPNetwork.ts` + `Server.ts` (custom TCP) | Plan uses custom TCP, not Steam              |
| **Browser builds**      | Not addressed                | `WebRTCNetwork.ts` (data channels)         | WebRTC is browser-only                       |
| **Cross-platform play** | Not addressed                | No bridge between TCP and WebRTC           | **Cannot play together**                     |
| **NAT traversal**       | Not addressed (LAN only)     | Not addressed                              | **Browser clients behind NAT can't connect** |
| **Relay server**        | Not addressed                | Not addressed                              | **Needed for web↔native bridging**           |

**Decision:** Steam Networking Sockets for native builds (Electron with full SDK), WebRTC/WebSocket for browsers. Hybrid approach with relay/bridge server that exposes web-compatible transport while using Steam SDK internally.

**Implications for Plan:**

- Electron: Full Steamworks SDK via `steamworks.js` (Rust N-API wrapper) — lobbies, SDR, overlay, auth tickets
- Browser: WebSocket (`wss://`) or WebRTC connection to dedicated server's web transport layer
- Dedicated server: Initializes Steam SDK (`SteamGameServer_Init`) for Master Server registration + exposes WebSocket/WebRTC alongside Steam game port
- Authentication: Steam OpenID 2.0 in browser → Steam Web API `ISteamUser/CheckAppOwnership` for verification
- Server discovery: Server queries `IGameServersService/GetServerList` via HTTP, passes JSON list to browser clients
- Add `NetworkBridge.ts` — protocol translation between Steam SDR (for native clients) and WebSocket/WebRTC (for browser clients)

**Status:** Decision recorded. Implementation tasks exist on Kanbn board (phase-0-18 through phase-0-21, phase-11-2).

### 1.2 Save System — 🔴 CLOSED

| Aspect                  | GDD Description                                    | Plan Description                    | Gap                        |
| ----------------------- | -------------------------------------------------- | ----------------------------------- | -------------------------- |
| **Single-player saves** | Plain text file (`missions/train01.msn timestamp`) | `SaveManager.ts` (no platform spec) | Web platform not addressed |
| **Server saves**        | Not addressed                                      | Not addressed                       | **Missing entirely**       |
| **Cross-sync**          | N/A                                                | N/A                                 | **Not specified**          |

**Decision:** Single-player games store saves locally. Dedicated servers store saves on the server. Local saves cannot be uploaded to the server. Server saves cannot be downloaded to save locally.

**Implications for Plan:**

- `SaveManager.ts` splits into:
  - `LocalSaveManager.ts` — file I/O (native) / IndexedDB (web)
  - `ServerSaveManager.ts` — server-side file storage
- No save transfer mechanism between local and server
- Server saves are isolated to server sessions only

**Status:** Decision recorded. Tasks exist on Kanbn board (phase-9-1, phase-9-2).

### 1.3 Asset Creation — 🔴 CLOSED

| Aspect       | GDD Description             | Plan Description                        | Gap                         |
| ------------ | --------------------------- | --------------------------------------- | --------------------------- |
| **Textures** | PPM format (SGI "rgb")      | PNG/WebP textures, all original         | **No legacy assets needed** |
| **Models**   | .tri and .ac (AC3D) formats | GLTF/GLB models, all original           | **No conversion pipeline**  |
| **Audio**    | WAV files                   | WAV/Ogg/MP3, all original               | **Format strategy defined** |

**Decision:** All assets created from scratch. No licensing concerns. Use BabylonJS-native GLTF/GLB for models, PNG/WebP for textures, Ogg/MP3 for audio.

**Asset sources:**

- Models: Blender (free), Kenney asset packs (CC0), OpenGameArt
- Textures: Krita/Photoshop (original), Kenney (CC0), procedural generation
- SFX: Freesound.org (CC0), BFXR, original compositions
- Music: Incompetia (free), Kenney (CC0), original compositions

**Status:** Decision recorded. Tasks exist on Kanbn board (phase-0-14 through phase-0-17).

### 1.4 Dual Game Mode — ✅ DECIDED

| Aspect              | Decision                                              | Rationale                                          |
| ------------------- | ----------------------------------------------------- | -------------------------------------------------- |
| **Game modes**      | Original Orbit + Second Stage (sequel)                | Play the classic first; your additions come later  |
| **Mission sets**    | `data/missions/original/` and `data/missions/second-stage/` | Clean separation, independent save tracking     |
| **Mode selector**   | Main menu toggle: Original / Second Stage             | Simple UX, both modes share core engine            |
| **Story context**   | Second Stage set after original events                | Future work — placeholder missions can be added incrementally |

**Decision:** Build Original Orbit first as the foundation. Second Stage layers on top as an expansion with new missions, ships, weapons, and story content. Both modes share the same game engine, physics, and systems.

**Status:** Decision recorded. Tasks exist on Kanbn board (phase-0-12, phase-0-13).

---

## 2. Medium Gaps (Should Be Clarified)

### 2.1 Web vs Native Multiplayer Protocol Compatibility

| Aspect              | Current State                                     | Gap                                               |
| ------------------- | ------------------------------------------------- | ------------------------------------------------- |
| **Steam clients**   | Steam Networking Sockets (encrypted, NAT-punched) | Native only                                       |
| **Web clients**     | WebRTC data channels (browser-only)               | Browser only                                      |
| **Protocol bridge** | Not implemented                                   | **Missing entirely**                              |
| **Session sharing** | Not addressed                                     | Web and native can't share sessions without relay |

**Required additions to Plan:**

- `RelayServer.ts` — Node.js relay that:
  - Initializes Steam SDK (`SteamGameServer_Init`) for Master Server registration
  - Listens on Steam game port (SDR) for native clients
  - Listens on WebSocket port (`wss://`) for browser clients
  - Bridges game state between both transport layers
- `NetworkBridge.ts` — Protocol translation: Steam SDR packets ↔ WebSocket JSON messages
- Authentication flow: Browser → Steam OpenID 2.0 → Steam Web API `CheckAppOwnership`
- Server discovery: `IGameServersService/GetServerList` HTTP API → JSON list to browser clients
- Electron: Full Steamworks SDK via `steamworks.js` (Rust N-API wrapper) — lobbies, SDR, overlay, auth tickets

**Status:** Decision recorded. Tasks exist on Kanbn board (phase-0-18 through phase-0-21).

### 2.2 Mission File Access — Platform Differences

| Aspect              | Native (Electron)           | Web                                      | Gap                                     |
| ------------------- | --------------------------------- | ---------------------------------------- | --------------------------------------- |
| **File I/O**        | Direct filesystem access          | No filesystem access                     | **Web can't load .msn files from disk** |
| **Mission loading** | Read from `data/missions/`        | Must use URL fetch or drag-and-drop      | **Not addressed in plan**               |
| **Include files**   | Resolved relative to mission file | Same issue — relative paths break in web | **Not addressed**                       |

**Required additions to Plan:**

- Web: Mission files served as static assets from `data/missions/`
- Web: Drag-and-drop support for custom missions
- MissionParser: Support both filesystem and URL-based include resolution

**Status:** Not yet addressed in implementation plan. Should be added to Phase 4 tasks.

### 2.3 Audio Format Strategy

| Aspect                | GDD                               | Plan                        | Gap                                       |
| --------------------- | --------------------------------- | --------------------------- | ----------------------------------------- |
| **Format**            | WAV                               | Not specified               | **Web needs Ogg/MP3, native can use WAV** |
| **3D audio**          | Not in original                   | Not addressed               | **BabylonJS supports 3D audio — add?**    |
| **Platform handling** | System audio (platform-dependent) | `SoundManager.ts` interface | **Web Audio API vs native audio**         |

**Recommended decision:**

- Convert WAV→Ogg for web builds (better compression, browser support)
- Keep WAV for Electron (native audio API supports it directly)
- Use BabylonJS `AudioEngine` for platform-agnostic playback
- Consider adding 3D positional audio for web (BabylonJS supports it natively)

**Status:** Not yet addressed. Should be added to Phase 9 tasks.

### 2.4 Original Asset Pipeline

| Aspect              | GDD                        | Plan                           | Gap                                         |
| ------------------- | -------------------------- | ------------------------------ | ------------------------------------------- |
| **Model format**    | .tri and .ac (AC3D)        | GLTF/GLB (BabylonJS native)    | **No conversion needed — all original**     |
| **Texture format**  | PPM                        | PNG/WebP                       | **No conversion needed — all original**     |
| **Audio format**    | WAV                        | WAV/Ogg/MP3                    | **Multi-format strategy defined**           |

**Recommended approach:**

- Use BabylonJS's built-in GLTF/GLB loader (no custom loader needed)
- PNG/WebP textures via `BABYLON.Texture` — no PPM reader needed
- Audio: WAV for Electron, Ogg/MP3 for web via `BABYLON.Sound`
- All assets stored in `data/models/`, `data/textures/`, `data/sounds/`, `data/music/`

**Status:** Decision recorded. Tasks exist on Kanbn board (phase-0-14 through phase-0-17).

---

## 3. Minor Gaps (Nice to Have)

### 3.1 Joystick & Mouse Configuration

| Detail               | GDD                                                 | Plan                             | Gap                     |
| -------------------- | --------------------------------------------------- | -------------------------------- | ----------------------- |
| **Deadzone**         | 0.1 default                                         | Not specified in `IInputManager` | Add to config           |
| **Throttle mapping** | 50-100% configurable                                | Not specified                    | Add to config           |
| **Mouse control**    | `mouse_control`, `mouse_flipx`, `mouse_flipy` prefs | Not mentioned in plan            | Add mouse input handler |

**Status:** Low priority. Should be added to Phase 2 tasks (2.4, 2.5).

### 3.2 Starfield Data Source

| Detail                | GDD                                    | Plan              | Gap                                   |
| --------------------- | -------------------------------------- | ----------------- | ------------------------------------- |
| **Source**            | Raul Alonso's ssystem stellar database | "65K+ stars" only | **No data source or conversion plan** |
| **Magnitude mapping** | Magnitudes → brightness                | Not addressed     | Add brightness mapping logic          |

**Status:** Medium priority. Should be added to Phase 1 tasks (1.2).

### 3.3 CI/CD & Deployment

| Aspect                  | GDD           | Plan                                        | Gap                                     |
| ----------------------- | ------------- | ------------------------------------------- | --------------------------------------- |
| **CI pipeline**         | Not addressed | Not addressed                               | **Missing entirely**                    |
| **Web deployment**      | Not addressed | Not addressed                               | **Missing entirely**                    |
| **Native distribution** | Not addressed | Phase 11 covers builds but not distribution | **Missing store/distribution strategy** |

**Status:** Low priority for MVP. Should be added as a separate phase or post-MVP task.

### 3.4 Security & Anti-Cheat

| Aspect                | GDD                    | Plan                                 | Gap                              |
| --------------------- | ---------------------- | ------------------------------------ | -------------------------------- |
| **Server authority**  | Implied (server hosts) | "Authoritative game state" mentioned | **Not detailed**                 |
| **Packet validation** | Not addressed          | Not addressed                        | **Missing entirely**             |
| **Anti-cheat**        | Not addressed          | Not addressed                        | **Missing for competitive play** |
| **Encryption**        | None (plain TCP)       | Steam = encrypted, WebRTC = DTLS     | **Mixed security posture**       |

**Status:** Low priority for MVP. Should be added to Phase 10 tasks.

### 3.5 Coordinate System — GDD vs Plan Alignment

| Aspect                   | GDD                        | Plan                                | Gap                  |
| ------------------------ | -------------------------- | ----------------------------------- | -------------------- |
| **Internal coords**      | Z-up (implied from C code) | Z-up internal, Y-up for BabylonJS   | Well addressed       |
| **Conversion functions** | Not specified              | `toBabylonVec3` / `fromBabylonVec3` | Good                 |

**Status:** Well addressed in both plan and Kanbn tasks (phase-0-4).

---

## 4. Architectural Decisions in Plan Not in GDD

These are additions in the implementation plan that have no counterpart in the GDD. They are valid design decisions but should be explicitly acknowledged:

| Decision                               | Rationale                                  | Risk                                 |
| -------------------------------------- | ------------------------------------------ | ------------------------------------ |
| **ECS-lite architecture**              | Network layer plugs in without refactoring | Over-engineering for a space shooter |
| **Interface segregation (I* pattern)** | Clean SP/Net separation                    | More files, more indirection         |
| **Fixed-timestep accumulator**         | Deterministic physics for netcode          | Slightly more complex game loop      |
| **Headless dedicated server**          | No rendering dependencies                  | Node.js performance vs C++ original  |
| **EventBus pub/sub**                   | Decoupled systems                          | Debugging complexity                 |
| **Hybrid networking (Steam + WebRTC)** | Full Steam features on desktop, web fallback | Complex relay server architecture  |

---

## 5. Missing Features in Plan (Present in GDD)

| Feature                       | GDD Section | Plan Coverage                        | Gap Level | Notes                                  |
| ----------------------------- | ----------- | ------------------------------------ | --------- | -------------------------------------- |
| **Joystick deadzone config**  | §2.3        | Not detailed                         | Low       | Add to Phase 2                         |
| **Mouse control with flip**   | §2.3        | Not mentioned                        | Low       | Add to Phase 1/2                       |
| **Superwarp (SP only)**       | §2.2.1      | Mentioned but not flagged as SP-only | Medium    | Should be explicitly SP-only           |
| **Weapon renderer visuals**   | §3.3        | Not detailed                         | Low       | Add to Phase 5                         |
| **Starfield data source**     | §9.4        | "65K+ stars" only                    | Medium    | Need ssystem conversion or alternative |
| **Briefing text auto-wrap**   | §8.1        | Not specified                        | Low       | Add to Phase 7                         |
| **Console message scrolling** | §8.1        | Not specified                        | Low       | Add to Phase 7                         |
| **Time compression (K key)**  | §2.3        | `compression` pref mentioned         | Low       | Add to Phase 9                         |
| **Field of view adjustment**  | §2.3        | `fov` pref mentioned                 | Low       | Add to Phase 9                         |
| **Clipping plane adjustment** | §2.3        | Not mentioned                        | Low       | Add to Phase 9                         |
| **Space junk reference**      | §9.5        | Phase 1.8 covers it                  | Covered   |                                        |
| **Planet ring systems**       | §6.4        | Phase 3.3 covers it                  | Covered   |                                        |
| **Gravitational physics**     | §6.5        | Phase 2.6 covers it                  | Covered   |                                        |

---

## 6. Kanbn Board Alignment Check

### 6.1 Task Coverage Analysis

| Implementation Plan Section | Expected Tasks | Kanbn Tasks Found | Aligned? | Notes                          |
| --------------------------- | -------------- | ----------------- | -------- | ------------------------------ |
| Phase 0 (Foundation)        | 21 tasks       | 21 tasks          | Yes      | All tasks present              |
| Phase 1 (Eye Candy)         | 8 tasks        | 6 tasks           | Partial  | Tasks 1.6-1.8 combined into one Kanbn task |
| Phase 2 (Flight Physics)    | 8 tasks        | 8 tasks           | Yes      | All tasks present              |
| Phase 3 (Planets)           | 7 tasks        | 7 tasks           | Yes      | All tasks present              |
| Phase 4 (Mission Parser)    | 10 tasks       | 4 tasks           | Partial  | Tasks 4.2-4.10 combined into one Kanbn task |
| Phase 5 (Combat)            | 9 tasks        | 2 tasks           | Partial  | Tasks 5.2-5.9 combined into one Kanbn task |
| Phase 6 (AI Targets)        | 8 tasks        | 2 tasks           | Partial  | Tasks 6.2-6.8 combined into one Kanbn task |
| Phase 7 (HUD)               | 10 tasks       | 2 tasks           | Partial  | Tasks 7.2-7.10 combined into one Kanbn task |
| Phase 8 (Events & Flow)     | 7 tasks        | 2 tasks           | Partial  | Tasks 8.3-8.7 combined into one Kanbn task |
| Phase 9 (Save & Polish)     | 10 tasks       | 2 tasks           | Partial  | Tasks 9.3-9.10 combined into one Kanbn task |
| Phase 10 (Multiplayer)      | 10 tasks       | 2 tasks           | Partial  | Tasks 10.4-10.10 combined into one Kanbn task |
| Phase 11 (Native Builds)    | 12 tasks       | 2 tasks           | Partial  | Tasks 11.3-11.12 combined into one Kanbn task |
| Phase 12 (WebXR/VR)         | 11 tasks       | 2 tasks           | Partial  | Tasks 12.2-12.11 combined into one Kanbn task |

**Total:** ~108 expected tasks, ~67 Kanbn tasks found. Many multi-subtask items are combined in the Kanbn board.

### 6.2 Missing Kanbn Tasks

| Missing Task ID | Description                                    | Priority |
| --------------- | ---------------------------------------------- | -------- |
| Phase 1         | Space junk reference polygons (1.8)            | Low      |
| Phase 4         | Individual subtasks (4.2-4.10) are combined    | Medium   |
| Phase 5         | Individual subtasks (5.2-5.9) are combined     | Medium   |
| Phase 6         | Individual subtasks (6.2-6.8) are combined     | Medium   |
| Phase 7         | Individual subtasks (7.2-7.10) are combined    | Medium   |
| Phase 8         | Individual subtasks (8.3-8.7) are combined     | Medium   |
| Phase 9         | Individual subtasks (9.3-9.10) are combined    | Medium   |
| Phase 10        | Individual subtasks (10.4-10.10) are combined  | Medium   |
| Phase 11        | Individual subtasks (11.3-11.12) are combined  | Medium   |
| Phase 12        | Individual subtasks (12.2-12.11) are combined  | Medium   |

**Recommendation:** Break combined Kanbn tasks into individual subtasks for better tracking and granularity.

---

## 7. Decision Log

| #    | Decision                                                                      | Date       | Rationale                                                                                                                                               |
| ---- | ----------------------------------------------------------------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| D001 | Steam Networking Sockets for native builds (Electron via steamworks.js) | 2026-11-23 | Best-in-class NAT traversal, cross-platform, integrates with Steam overlay. Full SDK accessible via Rust N-API wrapper in Electron main process.        |
| D002 | WebSocket/WebRTC for browser builds                                           | 2026-11-23 | Only viable option for browser multiplayer; no plugin required. Connects to server's web transport layer.                                               |
| D003 | Relay/bridge server with dual transport (Steam SDR + WebSocket/WebRTC)        | 2026-11-23 | Server initializes Steam SDK for Master Server registration, exposes web-compatible transport for browser clients. Bridge translates between protocols. |
| D004 | Single-player saves: local only (file I/O native, IndexedDB web)              | 2026-11-23 | Clean separation; no sync complexity                                                                                                                    |
| D005 | Server saves: server-side only                                                | 2026-11-23 | Server authority; no client tampering                                                                                                                   |
| D006 | No cross-sync between local and server saves                                  | 2026-11-23 | Prevents save manipulation; keeps architecture simple                                                                                                   |
| D007 | Source/create all original assets: models, textures, SFX, music               | 2026-11-23 | All assets in `data/` as GLTF/PNG/Ogg. No legacy assets, no licensing concerns. Use CC0 sources or original compositions.                             |

---

## 8. Recommended Plan Updates

### 8.1 Phase 4 Additions (Mission Parser)

Add platform-specific mission loading to Phase 4:

- **Task 4.11:** Implement web mission loader (URL fetch from static assets)
- **Task 4.12:** Implement drag-and-drop for custom missions (web)
- **Task 4.13:** Support URL-based include resolution in MissionParser

### 8.2 Phase 7 Additions (HUD)

Add missing HUD features from GDD:

- **Task 7.11:** Implement briefing text auto-wrap (GDD §8.1)
- **Task 7.12:** Implement console message scrolling (GDD §8.1)
- **Task 7.13:** Implement weapon selection UI (fire control, ammo management)

### 8.3 Phase 9 Additions (Save & Polish)

Add missing polish features from GDD:

- **Task 9.11:** Implement time compression display (K key, GDD §2.3)
- **Task 9.12:** Implement field of view adjustment UI (> / < keys)
- **Task 9.13:** Implement clipping plane adjustment UI ({ } ( ) keys)

### 8.4 Phase 10 Additions (Multiplayer)

Add security features:

- **Task 10.11:** Implement packet validation (server-side input sanitization)
- **Task 10.12:** Implement server-side anti-cheat checks (speed/hack detection)

### 8.5 Data Directory Structure Update

```
data/
├── missions/          # .msn and .inc files (unchanged)
│   ├── original/      # Classic Orbit missions
│   └── second-stage/  # Expansion missions
├── models/            # Original GLTF/GLB ship/station/starfield models
├── textures/          # PNG/WebP planet/space/background textures
├── sounds/            # Original SFX (WAV for native, Ogg for web)
└── music/             # Original music tracks (Ogg/MP3)
```

---

## 9. Steam + Browser Architecture Research (2026-11-23)

> Research sourced from Gemini analysis. Documents the technical approach for connecting browser clients to Steam infrastructure.

### 9.1 Core Technical Hurdles

**Transport Layer:** Steamworks networking (`ISteamNetworkingSockets` / SDR) runs over raw UDP. Browsers block raw UDP — limited to HTTP/HTTPS, WebSockets (TCP), and WebRTC (UDP with DTLS/SRTP).

**Native SDK Access:** Browser cannot load `steam_api64.dll` / `libsteam_api.so`, so no local Steam overlay, no binary auth tickets via C++, no direct P2P relay queries.

### 9.2 Solution Architecture

#### A. Dual-Transport Dedicated Server

Server initializes Steamworks SDK (`SteamGameServer_Init`) → registers on Steam Master Server, tracks tags/player counts, uses GSLTs.

**Alongside standard Steam game port**, server exposes:

- Secure WebSocket (`wss://`) for browser clients
- OR WebRTC data channel for browser clients

#### B. Authentication via Steam OpenID + Web API

1. Browser → Steam OpenID 2.0 login flow (`steamcommunity.com/openid/login`)
2. Steam returns verified `SteamID64` to backend/server
3. Server uses Steam Web API `ISteamUser/CheckAppOwnership` with Publisher Key to verify game ownership

#### C. Server Discovery (Matchmaking)

Browser cannot query Steam Master Server via UDP → server (or lightweight web backend) queries:

- `IGameServersService/GetServerList` HTTP Web API using Publisher Key
- Returns JSON list of active server IPs/ports to browser clients

### 9.3 Electron / Desktop Wrapper — Full Steam Access

When Vite + Babylon.js is packaged in Electron :

- Can load native Steamworks DLLs via `steamworks.js` (Rust-based N-API wrapper)
- Full access to: Steam lobbies, native auth session tickets, Steam Overlay, `ISteamNetworkingSockets` relays
- Same feature parity as traditional C++ or Unity builds

### 9.4 Architecture Decision Impact

This research confirms our hybrid approach:

- **Electron:** Full Steamworks SDK via `steamworks.js` — best experience, full Steam features
- **Browser:** WebSocket to relay server — limited but functional, no Steam overlay/lobby integration
- **Dedicated Server:** Dual transport — bridges both worlds

### 9.5 VR / WebXR Stretch Goal

**Goal:** Play Orbit: Second Stage in a VR headset with full 360° visibility and VR controller flight controls.

#### A. BabylonJS WebXR Support

BabylonJS has one of the most mature WebXR implementations in any game engine:

- **WebXRManager** — single entry point for all XR functionality
- **WebXRCamera** — replaces standard camera; handles HMD tracking natively
- **XR Input Sources** — abstracts VR controllers (Oculus Touch, Valve Index, Vive, etc.)
- **XR DOM Overlay** — renders HTML UI as 3D meshes in the headset
- **Teleportation** — built-in teleportation system for locomotion
- **Hand tracking** — experimental support via WebXR Hand Tracking API

#### B. Flight Controls in VR

| Input | VR Controller Mapping |
|-------|----------------------|
| Yaw/Pitch | Left thumbstick or HMD orientation |
| Roll | Right thumbstick or grip trigger |
| Thrust | Grip trigger (squeeze to accelerate) |
| Warp | Button combo (e.g., trigger + grip) |
| Fire | Index finger trigger |
| HUD interaction | Pointer ray + select button |

#### C. HUD Redesign for VR

2D overlay HUD doesn't work in XR. Must become:

- **3D floating panels** — DOM Overlay API renders HTML as textured planes in-world
- **Gaze-based or pointer-based selection** — controller ray casts to interact with buttons
- **Scale considerations** — panels at comfortable reading distance (~1-2m virtual)

#### D. Comfort & Scale Considerations

| Issue | Solution |
|-------|----------|
| Motion sickness from artificial locomotion | No forward movement — only rotation + warp; player stays stationary, ship orbits in place |
| Scale of solar system | Compress distances for VR readability; keep proportions but reduce scale factor |
| Starfield at extreme distance | Stars rendered as billboard sprites on inner sphere; outer space = procedural shader |

#### E. Technical Hurdles

1. **BabylonJS WebXRManager integration** — swap camera, configure input sources, handle session lifecycle
2. **Controller mapping** — abstract across headset types (Oculus, Vive, Index, Quest via Link/AirLink)
3. **VR-safe UI** — DOM Overlay panels must scale to headset FOV; button hitboxes need to be large enough for pointer rays
4. **Performance** — VR requires 72-90 FPS (varies by headset); must maintain frame budget with BabylonJS rendering pipeline

#### F. Architecture Decision Impact

- **WebXR is a browser feature** — works in both web and native builds (Electron supports WebXR via flags)
- **No separate multiplayer integration needed** — VR is a single-player mode; can play in multiplayer sessions
- **Phase placement:** Inline XR validated early in Phase 1.7 (no headset needed). Immersive VR in Phase 12 depends on Phase 2 (flight physics) and Phase 1 (scene), NOT on multiplayer. This means you can test XR integration weeks/months before multiplayer is done.
- **Early XR path:** Phase 1.0 (visual prototype) → Phase 1.7 (inline XR test) → Phase 12 (immersive VR with headset). You don't need to wait for core gameplay to be solid to start testing XR.

---

## 10. New Gaps Identified in This Round (2026-11-23)

### 10.1 Kanbn Task Granularity — MEDIUM

The Kanbn board has ~67 tasks for an implementation plan with ~108 individual tasks. Many phases combine multiple subtasks into single Kanbn entries, making progress tracking difficult.

**Recommendation:** Break combined tasks into individual subtasks. For example:
- Phase 4 (Mission Parser): Split the single "4.2-4.10" task into 9 individual tasks
- Phase 5 (Combat): Split the single "5.2-5.9" task into 8 individual tasks
- Same pattern for Phases 6-12

### 10.2 Missing Platform-Specific Mission Loading — MEDIUM

The implementation plan does not address how web clients will load mission files, since browsers have no filesystem access. The GDD assumes direct file I/O.

**Required additions:**
- Web: Static asset serving for `data/missions/` directory
- Web: Drag-and-drop support for custom .msn files
- MissionParser: URL-based include resolution (not just filesystem-relative)

### 10.3 Missing HUD Polish Features — LOW

Several GDD-specified HUD features are not explicitly covered in the implementation plan:
- Briefing text auto-wrap (GDD §8.1)
- Console message scrolling (GDD §8.1)
- Weapon selection UI with fire control and ammo management

### 10.4 Missing Polish Features — LOW

Several GDD-specified preference/key features are not explicitly covered:
- Time compression display (K key, GDD §2.3)
- Field of view adjustment UI (> / < keys)
- Clipping plane adjustment UI ({ } ( ) keys)

### 10.5 Security & Anti-Cheat — LOW

The implementation plan mentions "authoritative game state" but does not detail:
- Server-side packet validation
- Speed/hack detection for multiplayer
- Anti-cheat measures for competitive play

---

## 11. Files to Modify in Implementation Plan

| File                                    | Changes Required                                                                                              |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `implementation_plan.md` §4 (Phase 4)   | Add tasks 4.11-4.13: web mission loading, drag-and-drop, URL-based includes                                   |
| `implementation_plan.md` §7 (Phase 7)   | Add tasks 7.11-7.13: briefing auto-wrap, console scrolling, weapon selection UI                               |
| `implementation_plan.md` §9 (Phase 9)   | Add tasks 9.11-9.13: time compression display, FOV/clip UI                                                    |
| `implementation_plan.md` §10 (Phase 10) | Add tasks 10.11-10.12: packet validation, anti-cheat                                                          |
| `implementation_plan.md` §17            | Update testing strategy to include platform-specific tests (web mission loading, IndexedDB saves)             |

---

## 12. Summary of Gap Closure Status

| Category         | Total Gaps | Closed | In Progress | Open |
| ---------------- | ---------- | ------ | ----------- | ---- |
| Critical         | 4          | 4      | 0           | 0    |
| Medium           | 4          | 1      | 0           | 3    |
| Minor            | 5          | 2      | 0           | 3    |
| New (Round 2)    | 5          | 0      | 0           | 5    |
| **Total**        | **18**     | **7**  | **0**       | **11** |

**Next Steps:**
1. Update implementation plan with new tasks (Section 10 gaps)
2. Break combined Kanbn tasks into individual subtasks
3. Begin Phase 0 implementation (foundation tasks)
4. Add platform-specific mission loading to Phase 4
5. Consider CI/CD and deployment strategy post-MVP

---

*End of Gap Analysis.*
