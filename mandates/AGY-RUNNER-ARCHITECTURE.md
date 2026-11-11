# AGY-RUNNER-ARCHITECTURE.md — Antigravity (AGY) Runner & Host Daemon Architecture

**Status:** BINDING MANDATE  
**Applies to:** Hub Gateway (:3090), AGY Wrapper Daemon (:4095), OpenCode Runner (:4090), Data Hub UI (`#/data`), and `@jimesh/schemas`  
**Language:** English ONLY (per AGENTS.md §3)  

---

## 1. Executive Summary & Intent

To provide JiMesh operators with high-capacity autonomous coding and browsing assistance while avoiding container credential confinement, the JiMesh architecture establishes a dual-runner topology:

1. **OpenCode Central (`:4096` / `:4090` / Hub `:3090/api/opencode/*`)**:
   - Containerized multi-tenant runner designed for cloud provider APIs (Claude, OpenAI, DeepSeek).
   - Relies on ephemeral per-session working directories and containerized PTY lanes.
2. **Antigravity CLI Runner (`agy-wrapper.js` `:4095` / Hub `:3090/api/agy/*`)**:
   - Host-native daemon that wraps `${HOME}/.local/bin/agy` (Google DeepMind Antigravity CLI).
   - Resolves host DBus/Secret-Service and Gemini keyring credentials inaccessible inside isolated Docker containers.
   - Provides live VT100/xterm.js bidirectional terminal streaming over WebSockets.
   - Operates in both autonomous non-interactive execution (`-p` / `--print`) and interactive TUI sessions (`-i` / `--prompt-interactive`).

Both engines expose standardized REST envelopes and stream their session metadata into the unified Session Catalog (`/api/session-catalog` / `@jimesh/schemas/sessions.ts`).

---

## 2. Dual Runner Architectural Topology

```
                                  ┌───────────────────────────────┐
                                  │      Operator Browser         │
                                  │   http://localhost:3090/#/    │
                                  └──────────────┬────────────────┘
                                                 │
                                                 │ HTTP / WebSocket
                                                 ▼
┌─────────────────────────────────────────────────────────────────────────────────────────────────┐
│ JiMesh Gateway & Browser Hub (server.js :3090)                                                  │
│                                                                                                 │
│  /api/session-catalog       Unified Session Catalog (hub + opencode + pty + agy)                │
│  /api/opencode/*            Relay to OpenCode Runner (:4090)                                    │
│  /api/agy/*                 Relay to Host AGY Daemon (:4095)                                    │
│  /terminal?agent=agy        WebSocket Upgrade -> ws://127.0.0.1:4095/terminal                  │
│  /terminal                  WebSocket Upgrade -> ws://127.0.0.1:4090/terminal (opencode PTY)    │
└───────────────────────┬────────────────────────────────────────┬────────────────────────────────┘
                        │                                        │
                        │ HTTP / WS                              │ HTTP / WS
                        ▼                                        ▼
┌──────────────────────────────────────────────┐       ┌──────────────────────────────────────────┐
│ Docker: jimesh-opencode-runner (:4090)       │       │ Host Process: agy-wrapper.js (:4095)     │
│  - opencode central (:4096, Basic Auth)      │       │  - Native spawn of ${HOME}/.local/bin/agy│
│  - in-memory PTY lanes (node-pty)            │       │  - Direct DBus / Gnome Keyring access    │
│  - /workspace ephemeral trees                │       │  - stdio ring buffer (128KB)             │
└──────────────────────────────────────────────┘       └───────────────────┬──────────────────────┘
                                                                           │
                                                                           ▼
                                                       ┌──────────────────────────────────────────┐
                                                       │ Google Gemini & Antigravity Cloud APIs   │
                                                       │  - Models: gemini-3.8-flash-high,        │
                                                       │            gemini-4-argon-high,          │
                                                       │            claude-sonnet-5-5-high        │
                                                       └──────────────────────────────────────────┘
```

---

## 3. Comparison Matrix: OpenCode vs. AGY

| Feature | OpenCode Central Engine | Antigravity (AGY) Runner |
| :--- | :--- | :--- |
| **Port & Protocol** | `:4096` HTTP (Basic Auth), `:4090` PTY WS | `:4095` HTTP REST + WS `/terminal` |
| **Gateway Prefix** | `/api/opencode/*` | `/api/agy/*` |
| **Terminal Protocol** | Raw PTY JSON framing via `opencode-runner` | Direct VT100/ANSI stream over WebSocket |
| **Execution Context** | Docker container (`jimesh-opencode-runner`) | Host daemon (`agy-wrapper.js`) |
| **Credentials** | Environment variables (`OPENCODE_SERVER_PASSWORD`) | Host Secret-Service / DBus / `~/.gemini` |
| **Execution Modes** | REST message turn loop | Autonomous (`-p`) & Interactive TUI (`-i`) |
| **Target Directories** | `/workspace/data/<id>` | `${HOME}/projects/...` |
| **Catalog Kind** | `kind: 'opencode'` | `kind: 'agy'` |

---

## 4. SSOT Wire Schemas (`@jimesh/schemas/agy.ts`)

All wire interactions between UI, gateway, and daemon adhere strictly to `@jimesh/schemas`:

```typescript
export interface AgyHealthResponse {
  status: 'ok' | 'degraded' | 'error';
  service: 'agy-wrapper';
  binary: string;
  binaryExists: boolean;
  activeSessions: number;
  totalSessions: number;
  ts: string;
}

export interface AgySessionRecord {
  id: string;
  status: 'running' | 'exited' | 'error';
  createdAt: string;
  lastActive: number; // Unix epoch ms
  exitCode: number | null;
  dir: string;
  prompt: string;
  model?: string;
  effort?: 'low' | 'medium' | 'high' | 'xhigh' | 'max';
  mode?: 'print' | 'interactive';
}

export interface AgySpawnRequest {
  sessionId?: string;
  prompt: string;
  model?: string;
  effort?: 'low' | 'medium' | 'high' | 'xhigh' | 'max';
  mode?: 'print' | 'interactive';
  continueSession?: boolean;
}
```

---

## 5. Gateway REST API Reference

All routes are proxied from the hub gateway (`:3090`) to the host daemon (`:4095`):

### `GET /api/agy/health`
Returns runner readiness, verified binary path, and active session counts.
```json
{
  "status": "ok",
  "service": "agy-wrapper",
  "binary": "${HOME}/.local/bin/agy",
  "binaryExists": true,
  "activeSessions": 2,
  "totalSessions": 2,
  "ts": "2026-11-07T03:06:22.547Z"
}
```

### `GET /api/agy/sessions`
Returns an array of all active and terminated AGY sessions:
```json
{
  "ok": true,
  "sessions": [
    {
      "id": "default",
      "status": "running",
      "createdAt": "2026-11-07T02:56:49.866Z",
      "lastActive": 1794020210152,
      "exitCode": null,
      "dir": "/workspace/data/default",
      "prompt": "Inspect CHANGES_2026-11-06_session_log.md and complete Reddit onboarding"
    }
  ]
}
```

### `POST /api/agy/sessions/spawn`
Spawns a new session with custom mission prompt and model parameters:
```json
{
  "sessionId": "reddit_onboarding",
  "prompt": "Configure FreeDNS subdomain forwarding and complete Reddit verification OTP",
  "model": "gemini-3.8-flash-high",
  "effort": "high",
  "mode": "print"
}
```

### `POST /api/agy/sessions/:id/input`
Relays an operator input line into the session's active `stdin`.

### `DELETE /api/agy/sessions/:id`
Sends `SIGTERM` to the underlying AGY process and evicts it from the session map.

---

## 6. Frontend UI Integration (`http://localhost:3090/#/data`)

The Data Hub interface exposes two AGY controls:

1. **Launcher Modal (`[🚀 AGY Agent]`)**:
   - Location: Sidebar header next to `📁 /workspace/data`.
   - Allows operators to launch a targeted task against any directory with choice of Gemini/Claude models.
2. **Live Terminal Pane (`[⚡ AGY Terminal]`)**:
   - Location: File preview action bar.
   - Slides down an xterm.js instance connecting to `ws://localhost:3090/terminal?agent=agy&sessionId=<id>`.
   - Supports live ANSI color streaming, keyboard input, and re-attaching to existing sessions.

---

## 7. Verification & Health Check Commands

```bash
# 1. Check daemon health via gateway
curl -s http://localhost:3090/api/agy/health | jq .

# 2. Query active sessions
curl -s http://localhost:3090/api/agy/sessions | jq .

# 3. Test runtime schema validation
node --experimental-strip-types --no-warnings -e "
import { validateAgyHealthResponse } from './schemas/src/agy.ts';
fetch('http://127.0.0.1:4095/health').then(r => r.json()).then(d => console.log('Validated:', validateAgyHealthResponse(d)));
"
```
