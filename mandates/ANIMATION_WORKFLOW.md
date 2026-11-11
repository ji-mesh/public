# ANIMATION WORKFLOW & CONCEPTS — JiMesh Landing

## 1. PHILOSOPHIE: „Planung vor Implementation“

Jede Animation beginnt **nicht im Code**, sondern in einem **Screen-Plan**.
Der Code ist nur die Ausführung eines fertigen Plans.

---

## 2. SCREEN-PLAN (Vorlage)

Für jede Slide / jeden Screen wird **vorher** ein Markdown-File angelegt:

```markdown
# Screen Plan: [Name] — Slide [Nummer]

## Meta
- **Slide-ID**: z.B. `agents_mesh` (aus animation-states.json)
- **Render-Modus**: `ortho` | `comic3d` | `dom-overlay`
- **Breakpoints**: mobile (<640) | tablet (640-1023) | desktop (≥1024)
- **Android-Tiers**: mobile | tablet | desktop (für Fiber)

## Assets
| Asset | URL | Critical? | viewBox | Größe |
|-------|-----|-----------|---------|-------|
| Cube | /brand/jimesh_cube_transparent.svg | ✅ | 0 0 200 200 | 8KB |
| Agent-Orchestrator | /brand/agents/orchestrator_agent.svg | ✅ | 0 0 100 100 | 4KB |
| ... | ... | ... | ... | ... |

## Layout-Tiers (pro Breakpoint)
| Tier | navbarHeight | buttonScale | fontScale | spacingScale | canvasScale |
|------|--------------|-------------|-----------|--------------|-------------|
| mobile | 56 | 0.875 | 0.875 | 0.75 | 1.0 |
| tablet | 64 | 0.9375 | 0.9375 | 0.875 | 1.0 |
| desktop | 72 | 1.0 | 1.0 | 1.0 | 1.0 |

## Animation-Choreo (Timeline)
| Phase | Zeit (ms) | Element | Aktion | Easing | Trigger |
|-------|-----------|---------|--------|--------|---------|
| intro | 0-280 | Cube | scale 0.15→1 + fade | smoothstep | SVG loaded |
| intro | 280-560 | Rings | scale 0.15→1 + spin | spin.speed | SVG loaded |
| ... | ... | ... | ... | ... | ... |

## Responsive Behavior
| Element | Mobile | Tablet | Desktop |
|---------|--------|--------|---------|
| Cube Position | center, scale 0.8 | center, scale 0.9 | right, scale 1.0 |
| Text Layout | stacked | side-by-side | side-by-side |
| Navbar | hamburger | full | full |

## SVG-Load Dependencies
- **Critical**: Cube, Agent-Icons → müssen VOR Animation geladen sein
- **Deferred**: Background-Layer, Decorative → können nachladen

## Resize/Zoom Behavior
- Canvas neu berechnet bei Resize/Zoom via ResizeObserver
- Animation **retriggert** (neue Timeline) bei Tier-Wechsel
- Keine CSS-Media-Queries für Canvas-Layout

## Android / Touch
- Touch-Targets ≥ 48px (mobile), 44px (tablet), 40px (desktop)
- `touch-action: manipulation` auf Buttons
- Kein Hover-Only auf Touch-Geräten

---

## 3. WORKFLOW: VON PLAN ZU CODE

### Schritt 1: Screen-Plan erstellen
```
docs/screens/[slide-name].md
```
→ Review → Approval → **erst dann** Code schreiben.

### Schritt 2: SVG-Inventory prüfen
- Alle URLs im Plan existieren unter `/public/brand/...`?
- `viewBox` / Größe in SVG-Datei konsistent?
- Critical-Flag gesetzt?

### Schritt 3: Animation-States JSON pflegen
`packages/site-ui/src/components/blogdemo/animation-states.json`
- `slides[].layer_files` für M6-Layer
- `slides[].position` per Breakpoint
- `transition.exit_ms` = 420, `cooldown_ms` = 1600
- `intro.start_delay_ms` = 280 (Intro-Divisor!)

### Schritt 4: Code-Implementation
**Ordnung:**
1. `RootCanvasWrapper` (einmal pro Page)
2. `SVGStore` preload (einmal pro App)
3. Loading Bar (zentral, oberer Rand)
4. `HubCanvas` / `ComicCubeHero` / etc. **warten auf SVG-Ready**
5. Choreo exakt nach Plan (Timing, Easing, Trigger)

### Schritt 5: Responsive Test
- Chrome DevTools: Mobile / Tablet / Desktop
- Zoom testen (50% - 200%)
- Android Chrome (Remote Debug) → Touch-Targets prüfen

### Schritt 6: Animation-Mandates Checkliste (M1–M21)
- M1: States in JSON?
- M2: Kein Overlap?
- M3: smoothstep exit 420ms?
- M4: GLSL Background je Page?
- M5: SVGs 1:1, SMIL lebt?
- M6: Element-wise Choreo?
- M7: Text-Position?
- M8: Kein rotierender Brand-Cube?
- M9: reduced-motion?
- M10: Checkpoint-Commit?
- M11: Tool-Entscheidung korrekt?
- M12: SMIL-Redraw verstanden?
- M13: A/B/C-Weg klar?
- M14: Choreo-Motor Parameter?
- M15: Advanced-Shit-Checkliste?
- M16: RootCanvasWrapper?
- M17: SVG-Store + localStorage?
- M18: Loading Bar + Wait-for-SVG?
- M19: Android Tiers?
- M20: Animation NACH SVG-Load?
- M21: Async/Loading-Coroutines?

---

## 4. CONCEPTS: KOMPONENTEN-ARCHITEKTUR

### 4.1 RootCanvasWrapper (M16)
**Einmal pro Page**, kapselt alles Canvas-Layout.
- `ResizeObserver` auf Wrapper + echtem `<canvas>`
- Liefert `CanvasMetrics` (width, height, dpr, tier, styleTiers)
- Style Tiers: mobile / tablet / desktop + Android
- `useCanvasMetrics()` Hook für Consumer
- **Keine** CSS-Media-Queries für Canvas-Layout

### 4.2 SVGStore (M17, M21)
**Zentraler Zustand** (Zustand + persist + localStorage)
- `loadSVG(url)` — Promise, cached in localStorage
- `loadSVGBatch(urls[])` — Promise.allSettled
- `preloadCritical()` — alle Critical-SVGs parallel
- `getProgress()` — {loaded, total, progress} für Loading Bar
- Persistenz: `localStorage` via Zustand `persist`
- Async/await, Lambdas, Coroutines — **alles non-blocking**

### 4.3 Loading Bar (M18)
- Zentrale Bar (oberer Rand, fixiert)
- Zeigt `progress` von SVGStore
- **Erst bei 100%** → `RootCanvasWrapper` + Canvas rendern
- Danach Loading Bar ausblenden (fade out 300ms)

### 4.4 Component Integration (M20)
Jede Canvas-Komponente (`ComicCubeHero`, `AgentsMeshLayer`, `TeamCards`)
- **Wartet** auf `isLoaded(criticalUrl)` oder `preloadCritical()`
- Timeline startet **erst nach** SVG-Ready
- `useEffect` mit Dependency auf `svgReady`

### 4.5 Android / Scale Tiers (M19)
| Tier | canvasScale | uiScale | touchTarget |
|------|-------------|---------|-------------|
| mobile | 1.0 | 0.875 | 48px |
| tablet | 1.0 | 0.9375 | 44px |
| desktop | 1.0 | 1.0 | 40px |

Komponenten lesen `metrics.tier` und `metrics.styleTiers[metrics.tier]`
**Keine** `window.innerWidth` in Komponenten.

---

## 5. RESPONSIVE CANVAS PATTERN

### Falsch (alt):
```tsx
// In Komponente
const width = window.innerWidth
if (width < 640) { ... }
```

### Richtig (neu):
```tsx
// Im RootCanvasWrapper
<RootCanvasWrapper>
  {({ tier, styleTiers, width, height, dpr }) => (
    <Canvas>
      <MyComponent tier={tier} styles={styleTiers[tier]} canvasSize={{ width, height }} />
    </Canvas>
  )}
</RootCanvasWrapper>

// In Komponente
function MyComponent({ tier, styles, canvasSize }) {
  const scale = styles.canvasScale
  const buttonSize = 48 * styles.buttonScale
  // ...
}
```

---

## 6. ANIMATION-TRIGGER PATTERN

### Falsch (alt):
```tsx
useEffect(() => { startAnimation() }, [])
```

### Richtig (neu):
```tsx
const svgReady = useSVGStore(s => s.actions.isLoaded(CRITICAL_URL))

useEffect(() => {
  if (!svgReady) return
  const cleanup = startAnimation()
  return cleanup
}, [svgReady])
```

---

## 7. SVG PRELOAD PATTERN

```tsx
// In Page / Root Layout
useEffect(() => {
  useSVGStore.getState().actions.preloadCritical()
}, [])

// Loading Bar
const { loaded, total, progress } = useSVGProgress()

return (
  <>
    <LoadingBar progress={progress} />
    {progress === 1 && <RootCanvasWrapper>...</RootCanvasWrapper>}
  </>
)
```

---

## 8. CHECKLISTE PRO SLIDE (Definition of Done)

- [ ] Screen-Plan existiert (`docs/screens/...md`)
- [ ] Alle SVGs im Plan → Critical/Deferred markiert
- [ ] SVG-URLs existieren, `viewBox` konsistent
- [ ] `animation-states.json` gepflegt (M1, M14)
- [ ] `RootCanvasWrapper` genutzt (M16)
- [ ] `SVGStore.preloadCritical()` aufgerufen (M17)
- [ ] Loading Bar sichtbar bis 100% (M18)
- [ ] Animation wartet auf SVG-Ready (M20)
- [ ] Android Tiers in Komponenten genutzt (M19)
- [ ] Responsive getestet: Mobile / Tablet / Desktop / Zoom
- [ ] Touch-Targets ≥ 48/44/40px
- [ ] reduced-motion getestet (M9)
- [ ] M1–M21 Checkliste abgehakt
- [ ] Checkpoint-Commit (M10)

---

## 9. DATEI-STRUKTUR (Target)

```
${HOME}/projects/...
├── ANIMATION_MANDATES.md          # Bindende Regeln (M1-M21)
├── ANIMATION_WORKFLOW.md          # Dieses Dokument
├── docs/
│   └── screens/
│       ├── hub.md                 # Slide 0 Plan
│       ├── agents_mesh.md         # Slide 1 Plan
│       ├── secure_cloud.md        # Slide 2 Plan
│       └── ...
└── source/jimeshIO/frontend/
    ├── public-site/
    │   └── src/app/page.tsx       # Landing: LoadingBar + RootCanvasWrapper + HubCanvas
    └── packages/site-ui/
        ├── src/lib/
        │   ├── svg-store.ts       # Zustand Store (M17, M21)
        │   └── screen-plans.ts    # Types für Screen-Plans
        └── src/components/
            ├── RootCanvasWrapper.tsx  # M16
            ├── LoadingBar.tsx           # M18
            ├── blogdemo/
            │   ├── three/
            │   │   ├── hub-canvas.tsx   # nutzt RootCanvasWrapper, wartet auf SVG
            │   │   ├── comic-cube.tsx   # wartet auf SVG-Ready
            │   │   ├── agents-mesh-layer.tsx
            │   │   └── team-cards.tsx
            │   └── ...
            └── ...
```

---

## 10. NAMING CONVENTIONS

| Art | Pattern | Beispiel |
|-----|---------|----------|
| Screen Plan | `docs/screens/[slide-id].md` | `hub.md` |
| SVG Asset | `/brand/[category]/[name].svg` | `/brand/agents/orchestrator_agent.svg` |
| Layer File | `/brand/slideshow/layers/[slide]_[name].svg` | `amh_pod0.svg` |
| Animation State | `animation-states.json` | — |
| Style Tiers | `CanvasStyleTiers` Type | — |
| Component Props | `CanvasMetrics` + `tier` + `styles` | — |

---

## 11. REGEL: KEIN CODE OHNE PLAN

> **„Wer anfängt zu coden, bevor der Screen-Plan steht, baut Slop."**

Jede neue Animation / jeder neue Screen:
1. Plan schreiben (`docs/screens/...md`)
2. Review (selbst oder Peer)
3. **Erst dann** Code schreiben

---

*Version 1.0 — 2026-10-23 — bindend für alle Animation-Arbeiten*