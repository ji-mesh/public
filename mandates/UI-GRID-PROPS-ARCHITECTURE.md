# UI-GRID-PROPS-ARCHITECTURE.md — JiMesh AgentBoard UI, Grid & Widget SSOT Architecture

**Status:** BINDING MANDATE  
**Applies to:** All JiMesh Frontend Components, AgentBoard Widgets, OpenCode Agent Hub, and Layout Engines  
**Language:** English ONLY (per AGENTS.md §3)  

---

## 1. Executive Intent & Architecture Principles

To prevent frontend entropy, duplicate styling sheets, uncoordinated resize loops, and invisible agent widgets, all UI components rendered in the JiMesh Browser Hub (`jimesh-browser-hub-ui` :3090) and agent interfaces must adhere to this unified specification.

### Core Tenets:
1. **Single Source of Truth (SSOT)**: All wire payloads, widget manifests, telemetry structures, and layout modes MUST be defined in `@jimesh/schemas` and imported.
2. **Zero-Slop UI & Unified Token Sheet**: Never hardcode disconnected colors, random font declarations, or ad-hoc SVGs. Use standard CSS custom properties defined in `:root`.
3. **No Hidden iFrames**: External or agent-provided services MUST register an explicit `WidgetManifest` with declared permissions, telemetry profiling, and standardized container boundaries (mandates/FRAME-IFRAME-RULE.md).
4. **Resilient Dual-Rendering**: Every agent widget MUST support both client-side interactive mounting and Server-Side Rendering (SSR) via `/api/widgets/:id/render` for headless previews, screenshot gating, and iframe sandboxes.

---

## 2. BaseProps Contract & Responsive Canvas Lifecycle

All interactive canvas views (Camoufox WebRTC viewport, OpenCode terminal, AgentBoard Grid) must subscribe to the `BaseProps` lifecycle:

```typescript
export interface BaseProps {
  /** Root canvas width in CSS pixels */
  canvasWidth: number;
  /** Root canvas height in CSS pixels */
  canvasHeight: number;
  /** Device pixel ratio (1.0 - 3.0) */
  dpr: number;
  /** Active breakpoint mode */
  breakpoint: 'mobile' | 'tablet' | 'desktop' | 'ultrawide';
  /** Global color scheme */
  theme: 'dark' | 'light';
  /** Active layout configuration */
  layoutMode: 'split' | 'browser' | 'term' | 'grid';
  /** Unique tracing identifier for performance auditing */
  traceId: string;
}
```

### 2.1 ResizeObserver Lifecycle
- All canvas containers must use an explicit `ResizeObserver` debounced at 16ms (60 fps) to avoid layout thrashing.
- Canvas resizing triggers coordinate re-calibration in `browser-agent-bridge.js` so agent clicks match visual targets.

---

## 3. Multi-Layout Switcher Specification

The Hub dashboard provides four operational layouts selectable via `#layout-switcher` or URL parameter `?layout=<mode>`:

| Mode | Selector Class | Left Pane | Right Pane | Use Case |
| :--- | :--- | :--- | :--- | :--- |
| **`split`** | `.layout-split` | Camoufox Browser (50%) | OpenCode PTY/Web (50%) | Default pair-programming and supervised browsing |
| **`browser`** | `.layout-browser` | Camoufox Browser (100%) | Hidden / Minimized Dock | High-focus operator manual navigation & video playback |
| **`term`** | `.layout-term` | Hidden / Minimized Dock | Full Terminal (100%) | Deep shell workflows, build monitoring & container debug |
| **`grid`** | `.layout-grid` | 3-Column AgentBoard Canvas | Chat & HUD Dock | Autonomous multi-agent coordination & widget dashboard |

---

## 4. AgentBoard Widget Standards & Profiling Contract

### 4.1 Widget Manifest Definition
Every widget component in the store (`/api/widgets/store`) must provide:
- `id`: Globally unique slug (e.g. `widget-sentiment-analyzer`).
- `name`: Human-readable display label.
- `category`: `chat | dashboard | crm | media | custom`.
- `meta.bigONotation`: Declared computational complexity (`O(1)`, `O(log N)`, `O(N)`, `O(N^2)`).
- `meta.cpuPercent`: Expected CPU consumption under active execution.
- `meta.memoryMb`: Memory footprint limit.
- `meta.errorRatePct`: Rolling error budget.

### 4.2 Server-Side Rendering (SSR) Endpoint
- **URL**: `GET /api/widgets/:id/render`
- **Output**: Clean, standalone `text/html; charset=utf-8` document.
- **Contract**: Renders current telemetry badges, status pills, and action buttons without external JavaScript framework dependencies.

### 4.3 Vector Preview Endpoint
- **URL**: `GET /api/widgets/:id/preview.svg`
- **Output**: Scalable `image/svg+xml` visual card.
- **Contract**: Allows agents and tools to verify widget visual layout and telemetry metrics directly in terminal or headless environments without running a browser.

---

## 5. Agent Hub CLI (`hub_tool.py`) Operational Workflow

Agents operating inside `jimesh-opencode-runner` (:4090) interface with the Hub using `hub_tool.py`:

```bash
# 1. Inspect active peer sessions and squad channels
python3 hub_tool.py list-agents
python3 hub_tool.py list-threads

# 2. Register agent-authored widget
python3 hub_tool.py register-widget \
  --id widget-sentiment-analyzer \
  --name "Sentiment Analyzer" \
  --big-o "O(log N)" \
  --cpu 1.8

# 3. Preview & verify widget layout before publishing
python3 hub_tool.py preview-widget --id widget-sentiment-analyzer --output /tmp/widget.svg
python3 hub_tool.py render-widget --id widget-sentiment-analyzer --output /tmp/widget.html

# 4. Post interactive widget card into squad thread
python3 hub_tool.py post-chat \
  --thread thread-squadmesh-main \
  --body "Deployed sentiment analyzer v1.0" \
  --widget-id widget-sentiment-analyzer

# 5. Request visual verification screenshot
python3 hub_tool.py request-screenshot \
  --thread thread-squadmesh-main \
  --reason "Verify sentiment card rendering"
```

---

## 6. Definition of Done & Compliance Audit
- [ ] No inline style declarations violating the CSS variable token sheet.
- [ ] All new widgets registered in `widgetStore` and verified via `/api/widgets/:id/render`.
- [ ] Telemetry profiler (`POST /api/widgets/:id/profile`) validates Big-O complexity.
- [ ] No un-gated screenshot requests executed without operator approval or explicit `autoApprove`.
