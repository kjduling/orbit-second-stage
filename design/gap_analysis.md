# Gap Analysis: Orbit.md (GDD) vs implementation_plan.md

> Generated: 2026-11-23 | Status: Decisions Made — Plan Update Pending

---

## Executive Summary

The Game Design Document (Orbit.md) describes a **simple, self-hosted TCP multiplayer** architecture with straightforward file-based saves and native platform assets (.tri/.ac models, PPM textures, WAV audio). The implementation plan (implementation_plan.md) introduces a more complex **hybrid network architecture** (Steam + WebRTC), interface-segregated design, and ECS-lite patterns — but leaves critical gaps in protocol bridging, NAT traversal, data conversion, and platform-specific behavior.

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

**Decision:** Steam Networking Sockets for native builds (Electron/Tauri with full SDK), WebRTC/WebSocket for browsers. Hybrid approach with relay/bridge server that exposes web-compatible transport while using Steam SDK internally.

**Implications for Plan:**

- Electron/Tauri: Full Steamworks SDK via `steamworks.js` (Rust N-API wrapper) — lobbies, SDR, overlay, auth tickets
- Browser: WebSocket (`wss://`) or WebRTC connection to dedicated server's web transport layer
- Dedicated server: Initializes Steam SDK (`SteamGameServer_Init`) for Master Server registration + exposes WebSocket/WebRTC alongside Steam game port
- Authentication: Steam OpenID 2.0 in browser → Steam Web API `ISteamUser/CheckAppOwnership` for verification
- Server discovery: Server queries `IGameServersService/GetServerList` via HTTP, passes JSON list to browser clients
- Add `NetworkBridge.ts` — protocol translation between Steam SDR (for native clients) and WebSocket/WebRTC (for browser clients)

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

### 1.3 Data Conversion Pipeline — 🔴 CLOSED

| Aspect       | GDD Description             | Plan Description                        | Gap                         |
| ------------ | --------------------------- | --------------------------------------- | --------------------------- |
| **Textures** | PPM format (SGI "rgb")      | Mentions "PNG/WebP" but no pipeline     | **No conversion tool/task** |
| **Models**   | .tri and .ac (AC3D) formats | Mentions "GLTF/BABYLON" but no pipeline | **No conversion tool/task** |
| **Audio**    | WAV files                   | `SoundManager.ts` interface only        | **Format not specified**    |

**Decision:** One-time conversion task. Convert PPM→PNG, .tri/.ac→GLTF. May replace original models later.

**Implications for Plan:**

- Add Phase 0.12: Data Conversion Pipeline (PPM→PNG, .tri→GLTF, .ac→GLTF)
- Add build script `npm run convert-assets`
- Audio: Keep WAV for native, convert to Ogg/MP3 for web

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
- Electron/Tauri: Full Steamworks SDK via `steamworks.js` (Rust N-API wrapper) — lobbies, SDR, overlay, auth tickets

### 2.2 Mission File Access — Platform Differences

| Aspect              | Native (Electron/Tauri)           | Web                                      | Gap                                     |
| ------------------- | --------------------------------- | ---------------------------------------- | --------------------------------------- |
| **File I/O**        | Direct filesystem access          | No filesystem access                     | **Web can't load .msn files from disk** |
| **Mission loading** | Read from `data/missions/`        | Must use URL fetch or drag-and-drop      | **Not addressed in plan**               |
| **Include files**   | Resolved relative to mission file | Same issue — relative paths break in web | **Not addressed**                       |

**Required additions to Plan:**

- Web: Mission files served as static assets from `data/missions/`
- Web: Drag-and-drop support for custom missions
- MissionParser: Support both filesystem and URL-based include resolution

### 2.3 Audio Format Strategy

| Aspect                | GDD                               | Plan                        | Gap                                       |
| --------------------- | --------------------------------- | --------------------------- | ----------------------------------------- |
| **Format**            | WAV                               | Not specified               | **Web needs Ogg/MP3, native can use WAV** |
| **3D audio**          | Not in original                   | Not addressed               | **BabylonJS supports 3D audio — add?**    |
| **Platform handling** | System audio (platform-dependent) | `SoundManager.ts` interface | **Web Audio API vs native audio**         |

**Recommended decision:**

- Convert WAV→Ogg for web builds (better compression, browser support)
- Keep WAV for Electron/Tauri (native audio API supports it directly)
- Use BabylonJS `AudioEngine` for platform-agnostic playback
- Consider adding 3D positional audio for web (BabylonJS supports it natively)

### 2.4 Model Conversion Pipeline

| Aspect              | GDD                        | Plan                           | Gap                                         |
| ------------------- | -------------------------- | ------------------------------ | ------------------------------------------- |
| **.tri loader**     | Simple vertex format       | Phase 1 mentions `.tri` loader | **OK — simple to implement**                |
| **.ac loader**      | AC3D format with materials | Not addressed                  | **AC3D is complex — needs conversion tool** |
| **Texture binding** | PPM textures on models     | Mentions "GLTF/BABYLON"        | **Conversion pipeline needed**              |

**Recommended decision:**

- Convert .ac files to GLTF using Blender or AC3D export tool (one-time)
- Keep .tri loader in code (simple enough to parse at runtime)
- Store converted assets in `data/models/gltf/` and `data/textures/png/`

---

## 3. Minor Gaps (Nice to Have)

### 3.1 Joystick & Mouse Configuration

| Detail               | GDD                                                 | Plan                             | Gap                     |
| -------------------- | --------------------------------------------------- | -------------------------------- | ----------------------- |
| **Deadzone**         | 0.1 default                                         | Not specified in `IInputManager` | Add to config           |
| **Throttle mapping** | 50-100% configurable                                | Not specified                    | Add to config           |
| **Mouse control**    | `mouse_control`, `mouse_flipx`, `mouse_flipy` prefs | Not mentioned in plan            | Add mouse input handler |

### 3.2 Starfield Data Source

| Detail                | GDD                                    | Plan              | Gap                                   |
| --------------------- | -------------------------------------- | ----------------- | ------------------------------------- |
| **Source**            | Raul Alonso's ssystem stellar database | "65K+ stars" only | **No data source or conversion plan** |
| **Magnitude mapping** | Magnitudes → brightness                | Not addressed     | Add brightness mapping logic          |

### 3.3 CI/CD & Deployment

| Aspect                  | GDD           | Plan                                        | Gap                                     |
| ----------------------- | ------------- | ------------------------------------------- | --------------------------------------- |
| **CI pipeline**         | Not addressed | Not addressed                               | **Missing entirely**                    |
| **Web deployment**      | Not addressed | Not addressed                               | **Missing entirely**                    |
| **Native distribution** | Not addressed | Phase 11 covers builds but not distribution | **Missing store/distribution strategy** |

### 3.4 Security & Anti-Cheat

| Aspect                | GDD                    | Plan                                 | Gap                              |
| --------------------- | ---------------------- | ------------------------------------ | -------------------------------- |
| **Server authority**  | Implied (server hosts) | "Authoritative game state" mentioned | **Not detailed**                 |
| **Packet validation** | Not addressed          | Not addressed                        | **Missing entirely**             |
| **Anti-cheat**        | Not addressed          | Not addressed                        | **Missing for competitive play** |
| **Encryption**        | None (plain TCP)       | Steam = encrypted, WebRTC = DTLS     | **Mixed security posture**       |

### 3.5 Coordinate System — GDD vs Plan Alignment

| Aspect                   | GDD                        | Plan                                | Gap                  |
| ------------------------ | -------------------------- | ----------------------------------- | -------------------- |
| **Internal coords**      | Z-up (implied from C code) | Z-up internal, Y-up for BabylonJS   | ✅ **Well addressed** |
| **Conversion functions** | Not specified              | `toBabylonVec3` / `fromBabylonVec3` | ✅ **Good**           |

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

---

## 5. Missing Features in Plan (Present in GDD)

| Feature                       | GDD Section | Plan Coverage                        | Gap Level |
| ----------------------------- | ----------- | ------------------------------------ | --------- |
| **Joystick deadzone config**  | §2.3        | Not detailed                         | Low       |
| **Mouse control with flip**   | §2.3        | Not mentioned                        | Low       |
| **Superwarp (SP only)**       | §2.2.1      | Mentioned but not flagged as SP-only | Medium    |
| **Weapon renderer visuals**   | §3.3        | Not detailed                         | Low       |
| **Starfield data source**     | §9.4        | "65K+ stars" only                    | Medium    |
| **Briefing text auto-wrap**   | §8.1        | Not specified                        | Low       |
| **Console message scrolling** | §8.1        | Not specified                        | Low       |
| **Time compression (K key)**  | §2.3        | `compression` pref mentioned         | Low       |

---

## 6. Decision Log

| #    | Decision                                                                      | Date       | Rationale                                                                                                                                               |
| ---- | ----------------------------------------------------------------------------- | ---------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| D001 | Steam Networking Sockets for native builds (Electron/Tauri via steamworks.js) | 2026-11-23 | Best-in-class NAT traversal, cross-platform, integrates with Steam overlay. Full SDK accessible via Rust N-API wrapper in Electron main process.        |
| D002 | WebSocket/WebRTC for browser builds                                           | 2026-11-23 | Only viable option for browser multiplayer; no plugin required. Connects to server's web transport layer.                                               |
| D003 | Relay/bridge server with dual transport (Steam SDR + WebSocket/WebRTC)        | 2026-11-23 | Server initializes Steam SDK for Master Server registration, exposes web-compatible transport for browser clients. Bridge translates between protocols. |
| D004 | Single-player saves: local only (file I/O native, IndexedDB web)              | 2026-11-23 | Clean separation; no sync complexity                                                                                                                    |
| D005 | Server saves: server-side only                                                | 2026-11-23 | Server authority; no client tampering                                                                                                                   |
| D006 | No cross-sync between local and server saves                                  | 2026-11-23 | Prevents save manipulation; keeps architecture simple                                                                                                   |
| D007 | One-time data conversion pipeline (PPM→PNG, .tri/.ac→GLTF)                    | 2026-11-23 | Convert once, use forever. May replace models later anyway.                                                                                             |

---

## 7. Recommended Plan Updates

### 7.1 Phase 0 Additions

- **Task 0.12:** Data conversion pipeline (PPM→PNG, .tri→GLTF, .ac→GLTF)
- **Task 0.13:** Relay server scaffolding (`RelayServer.ts`) — dual transport (Steam SDR + WebSocket)
- **Task 0.14:** Steamworks.js setup for Electron/Tauri (Rust N-API wrapper)
- **Task 0.15:** Steam OpenID authentication flow for browser clients
- **Task 0.16:** Server discovery via `IGameServersService/GetServerList` HTTP API

### 7.2 New Phase — Network Infrastructure

Consider a dedicated "Phase 9.5" between Save System and Multiplayer:

- Relay/bridge server implementation
- Steam networking client for native
- WebRTC client for browser
- Protocol bridge between the two
- NAT traversal (STUN/TURN)

### 7.3 Phase 10 Restructuring

Rename to "Phase 10 — Multiplayer Clients" and focus on:

- `SteamNetwork.ts` — Native multiplayer client (via steamworks.js in Electron main process)
- `WebSocketNetwork.ts` — Browser multiplayer client (wss:// to relay server)
- `NetworkedGame` — Client-side game logic
- Position sync, packet handling

### 7.4 Phase 11 Addition

New phase for "Phase 11 — Dedicated Server":

- `RelayServer.ts` — Full dual-transport relay:
  - Steam SDR port for native clients (via Steamworks SDK)
  - WebSocket port for browser clients
  - Game state bridging between both transport layers
- `NetworkBridge.ts` — Protocol translation layer
- `ServerSaveManager.ts` — Server-side save system
- Steam OpenID verification endpoint (`CheckAppOwnership`)
- Server admin commands (DROP, roll call)
- Server configuration

### 7.5 Data Directory Structure Update

```
data/
├── missions/          # .msn and .inc files (unchanged)
├── models/
│   ├── tri/           # Original .tri files (kept for reference)
│   └── gltf/          # Converted .gltf/.glb files (used at runtime)
├── textures/
│   ├── ppm/           # Original PPM files (kept for reference)
│   └── png/           # Converted PNG files (used at runtime)
└── sounds/
    ├── wav/           # Original WAV files (used by Electron/Tauri)
    └── ogg/           # Converted OGG files (used by web)
```

---

## 8. Files to Modify in Implementation Plan

| File                                    | Changes Required                                                                                              |
| --------------------------------------- | ------------------------------------------------------------------------------------------------------------- |
| `implementation_plan.md` §1.1           | Add Steam/WebRTC architecture decision table                                                                  |
| `implementation_plan.md` §2             | Update project structure with Steam, RelayServer, NetworkBridge                                               |
| `implementation_plan.md` §3.4–3.5       | Update Electron/Tauri to include Steam SDK setup                                                              |
| `implementation_plan.md` §4 (Phase 0)   | Add tasks 0.12–0.15 (data conversion, relay server, Steam SDK)                                                |
| `implementation_plan.md` §13 (Phase 9)  | Split into SP save (local) and server save                                                                    |
| `implementation_plan.md` §14 (Phase 10) | Restructure: Steam client (via steamworks.js), WebSocket client, NetworkedGame                                |
| `implementation_plan.md` §15 (Phase 11) | New phase: RelayServer (dual transport), NetworkBridge, ServerSaveManager, OpenID auth, Steam API integration |
| `implementation_plan.md` §16            | Add network security tests                                                                                    |
| `implementation_plan.md` §17            | Add risks: Steamworks.js compatibility, relay server complexity, OpenID flow, Steam Web API rate limits       |

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

When Vite + Babylon.js is packaged in Electron (or Tauri):

- Can load native Steamworks DLLs via `steamworks.js` (Rust-based N-API wrapper)
- Full access to: Steam lobbies, native auth session tickets, Steam Overlay, `ISteamNetworkingSockets` relays
- Same feature parity as traditional C++ or Unity builds

### 9.4 Architecture Decision Impact

This research confirms our hybrid approach:

- **Electron/Tauri:** Full Steamworks SDK via `steamworks.js` — best experience, full Steam features
- **Browser:** WebSocket to relay server — limited but functional, no Steam overlay/lobby integration
- **Dedicated Server:** Dual transport — bridges both worlds

---

*End of Gap Analysis.*
