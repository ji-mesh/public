# ANIMATION SCHEMA SPECIFICATION — JiMesh Landing

## ÜBERBLICK

Alle Animation-States liegen als **YAML** im Ordner `animation-schemas/`, strukturiert nach Slides. 
Die JSON in `animation-states.json` wird **aus den YAMLs generiert** (Build-Zeit oder Dev-Script).

---

## ORDNER-STRUKTUR

```
${HOME}/projects/...
├── animation-schemas/
│   ├── _config.yml                    # Globale Defaults
│   ├── slides/
│   │   ├── 00_hub.yml                 # Slide 0: Hub / Landing
│   │   ├── 01_agents_mesh.yml         # Slide 1: Agents Mesh
│   │   ├── 02_secure_cloud.yml        # Slide 2: Secure Cloud
│   │   ├── 03_ai_gateway.yml          # Slide 3: AI Gateway
│   │   ├── 04_connectors.yml          # Slide 4: Connectors
│   │   ├── 05_managed_services.yml    # Slide 5: Managed Services
│   │   ├── 06_observability.yml       # Slide 6: Observability
│   │   ├── 07_ozone_vault.yml         # Slide 7: Ozone Vault
│   │   ├── 08_threat_response.yml     # Slide 8: Threat Response
│   │   ├── 09_jimesh_cube.yml         # Slide 9: JiMesh Cube
│   │   ├── 10_mock_preview.yml        # Slide 10: Mock Preview
│   │   └── 11_agent_chat_preview.yml  # Slide 11: Agent Chat Preview
│   ├── layers/
│   │   ├── _shared.yml                # Shared Layer-Defaults
│   │   ├── hub/
│   │   │   ├── glow.yml
│   │   │   ├── box.yml
│   │   │   ├── rings.yml
│   │   │   ├── cube.yml
│   │   │   └── pods/
│   │   ├── agents_mesh/
│   │   │   ├── glow.yml
│   │   │   ├── box.yml
│   │   │   ├── rings.yml
│   │   │   ├── cube.yml
│   │   │   └── pods/
│   │   └── secure_cloud/
│   ├── shaders/
│   │   ├── _shared.glsl               # Shared GLSL functions
│   │   ├── bg-frag.glsl               # Background Fragment Shader
│   │   ├── ring-frag.glsl             # Ring Shader
│   │   ├── toon-frag.glsl             # Toon Shader
│   │   ├── hull-frag.glsl             # Hull Shader
│   │   ├── drop-frag.glsl             # Drop Shader
│   │   ├── glitch-sprite-frag.glsl    # Glitch Sprite Shader
│   │   ├── glitch-mask-frag.glsl      # Glitch Mask Shader
│   │   ├── ring-reveal-frag.glsl      # Ring Reveal Shader
│   │   └── comic-bg-frag.glsl         # Comic BG Shader (with splitAmt)
│   ├── text/
│   │   ├── scramble.yml
│   │   ├── fade-in.yml
│   │   ├── fade-up.yml
│   │   └── loop-stages.yml
│   └── transitions/
│       ├── exit.yml
│       ├── cooldown.yml
│       └── intro.yml
```

---

## YAML-SCHEMA DEFINITIONEN

### 1. `_config.yml` — Globale Defaults

```yaml
# animation-schemas/_config.yml
version: "1.0"
project: "JiMesh Landing"
generated: "2026-10-23"

defaults:
  transition:
    exit_ms: 420
    cooldown_ms: 1600
    intro_divisor: 280
    easing: "cubic-bezier(0.22, 1, 0.36, 1)"
    smoothstep: true
  
  redraw:
    layer_ms: 50      # ~20fps für Layer (Ringe, Pods)
    full_ms: 600      # ~6fps für Vollraster
  
  breakpoints:
    mobile: 640
    tablet: 1024
    desktop: 1920

  canvas:
    logical_width: 1920
    logical_height: 1080
    ortho_half_h: 560
    near: -600
    far: 600

  loading:
    critical_svgs:
      - /brand/jimesh_cube_transparent.svg
      - /brand/agents/orchestrator_agent.svg
      - /brand/agents/coding_agent.svg
      - /brand/agents/browser_agent.svg
      - /brand/agents/ninja_agent.svg
      - /brand/agents/red_team_agent.svg
      - /brand/agents/figma_agent.svg
      - /brand/agents/fin_agent.svg
      - /brand/agents/voice_agent.svg
      - /brand/humans/man1.svg
      - /brand/humans/woman1.svg
```

### 2. Slide-Schema — z.B. `slides/01_agents_mesh.yml`

```yaml
# animation-schemas/slides/01_agents_mesh.yml
slide:
  id: "agents_mesh"
  index: 1
  render_mode: "ortho"           # ortho | comic3d | dom-overlay
  split: true
  mock: true
  
  intro:
    start_delay_ms: 0
    stagger_ms: 50
    element_offset: 100
    divisor: 280

  position:
    desktop: { x: -160, y: -300, scale: 0.82 }
    tablet:  { x: -100, y: -220, scale: 0.60 }
    phone:   { x: 0,    y: 80,   scale: 0.90 }

  exit_ms: 420
  cooldown_ms: 1600

  layer_spec:
    - selector: "amh_glow"
      delay: 40
      anim: "fade"
      dist: 0
      fromScale: 0.12
      
    - selector: "amh_box"
      delay: 100
      anim: "fade"
      dist: 0
      
    - selector: "amh_rings"
      delay: 160
      anim: "scale"
      from: [0, 0]
      dist: 0
      fromScale: 0.15
      spin:
        speed: -0.12
      orbit:
        cx: 0
        cy: 0
        rx: 130
        ry: 190
        speed: 0
        phase: 0

    - selector: "amh_cube"
      delay: 220
      anim: "slide"
      from: [-1, 0]
      dist: 260
      fromScale: 1.0
      exit_shrink: 0.55

    - selector: "amh_pod*"
      delay_base: 300
      delay_step: 18
      anim: "scale"
      fromScale: 0.3
      spin:
        speed: 0.12
      orbit:
        cx: 0
        cy: 0
        rx: 340
        ry: 15
        speed: -0.12
        phase: 0
      count: 27
      index_range: [0, 25, 32]

  transition:
    exit_ms: 420
    cooldown_ms: 1600
    easing: "cubic-bezier(0.22, 1, 0.36, 1)"
```

### 3. Layer-Schema — z.B. `layers/agents_mesh/rings.yml`

```yaml
# animation-schemas/layers/agents_mesh/rings.yml
layer:
  id: "amh_rings"
  file: "amh_rings.svg"
  type: "vector"                 # vector | raster | smil
  
  geometry:
    x: 0
    y: 0
    w: 700
    h: 700
    
  animation:
    anim: "scale"
    delay: 160
    fromScale: 0.15
    dist: 0
    spin:
      speed: -0.12
    orbit:
      cx: 130
      cy: 190
      rx: 0
      ry: 0
      speed: 0
      phase: 0
      
  exit:
    scale_to_fromScale: true
    fade_quadratic: true
    shrink_cube: 0.55
    
  smil:
    has_animations: false
    redraw_ms: 50
    
  shader:
    type: "ring"
    fragment: "ring-frag.glsl"
    uniforms:
      - uTime
    spin_controlled_by_layer: true
```

### 4. Shader-Schema — z.B. `shaders/glitch-sprite-frag.glsl`

```yaml
# animation-schemas/shaders/glitch-sprite-frag.yml
shader:
  name: "glitch-sprite-frag"
  file: "glitch-sprite-frag.glsl"
  type: "fragment"
  
  uniforms:
    - name: "uMap"
      type: "sampler2D"
      description: "Input texture (SVG raster)"
    - name: "uTime"
      type: "float"
      description: "Global time in seconds"
    - name: "uReveal"
      type: "float"
      description: "0 = fully glitched/hidden, 1 = clean"
  
  features:
    - horizontal_block_displacement
    - rgb_split
    - digital_noise
    - scanline
    - progressive_pixel_reveal
    - flicker
  
  parameters:
    block_displacement:
      row_height: 16.0
      time_speed: 10.0
      threshold: 0.55
      max_offset: 0.28
    rgb_split:
      base_amount: 0.018
      time_speed: 22.0
      red_weight: 1.0
      green_weight: -0.5
      blue_weight: 1.5
    noise:
      frequency: 90.0, 70.0
      time_speed: 35.0
      amplitude: 0.3
    scanline:
      frequency: 320.0
      time_speed: 18.0
      amplitude: 0.12
      base: 0.88
    pixel_reveal:
      hash_scale: 41.0
      hash_offset: 7.0
      reveal_multiplier: 1.18
    flicker:
      time_speed: 24.0
      threshold: 0.65
      amplitude: 0.35
      
  glsl: |
    // ... GLSL code embedded or referenced
```

### 5. Text-Animation Schema — z.B. `text/scramble.yml`

```yaml
# animation-schemas/text/scramble.yml
animation:
  name: "scramble"
  type: "text"
  description: "Matrix-style decode: chars sit in final slots, scramble briefly, lock in shuffled order"
  
  parameters:
    duration_ms: 700
    scramble_chars: "!@#$%^&*()_+-=[]{}|;':\",./<>?0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZ"
    lock_order: "shuffled"
    width_lock: true
    fast_mode: true
    
  timing:
    initial_delay: 0
    scramble_duration: 350
    lock_duration: 350
    hold_duration: 0
    
  css:
    font_family: "JetBrains Mono"
    font_size: "clamp(1.5rem, 4vw, 3rem)"
    line_height: 1.2
```

---

## PARSEN & GENERIEREN

### Build-Script: `scripts/generate-animation-states.ts`

```typescript
// scripts/generate-animation-states.ts
import * as fs from 'fs'
import * as path from 'path'
import * as yaml from 'yaml'

const SCHEMA_DIR = 'animation-schemas'
const OUTPUT_FILE = 'packages/site-ui/src/components/blogdemo/animation-states.json'

function loadYaml(file: string) {
  return yaml.parse(fs.readFileSync(file, 'utf-8'))
}

function generate() {
  const config = loadYaml(path.join(SCHEMA_DIR, '_config.yml'))
  const slidesDir = path.join(SCHEMA_DIR, 'slides')
  const slideFiles = fs.readdirSync(slidesDir).filter(f => f.endsWith('.yml')).sort()
  
  const slides = slideFiles.map(file => {
    const slide = loadYaml(path.join(slidesDir, file))
    return slide.slide
  })
  
  const output = {
    meta: config,
    background: config.defaults.background,
    transition: config.defaults.transition,
    slides,
  }
  
  fs.writeFileSync(OUTPUT_FILE, JSON.stringify(output, null, 2))
  console.log(`Generated ${OUTPUT_FILE} with ${slides.length} slides`)
}

generate()
```

---

## BESTEHENDE MUSTER IM INTERNET (Referenzen)

| System | Format | Besonderheit |
|--------|--------|--------------|
| **Lottie / Bodymovin** | JSON | After Effects → Web, weit verbreitet |
| **Rive** | .riv (binary) + JSON | State Machines, Interaktivität |
| **Framer Motion** | JS Objects | React-first, Variants, Layout Animations |
| **GSAP** | JS Timeline | Professionell, Plugins (ScrollTrigger, MorphSVG) |
| **Theater.js** | JSON + Editor | Visual Editor, Timeline-based |
| **Motion One** | WAAPI Wrapper | Klein, Web Animations API |
| **Anime.js** | JS Objects | Leicht, flexible Easing |
| **CSS Motion Path** | CSS | Native, `offset-path` |
| **SVG SMIL** | XML | Native SVG, `<animate>` Tags |
| **CSS `@keyframes`** | CSS | Native, performant |

**Empfehlung für JiMesh:**
- **YAML für Schema** (menschenlesbar, versionierbar, diffbar)
- **GLSL separat** (Syntax-Highlighting, Linting)
- **Build-Time Generation** → JSON für Runtime (schnell, typsicher)
- **TypeScript Types** aus Schema generieren (`zod` oder `ts-yaml`)

---

## IMPLEMENTIERUNGS-PRIORITÄT

1. **Phase 1**: `_config.yml` + `slides/00_hub.yml` + `slides/01_agents_mesh.yml`
2. **Phase 2**: `layers/` + `shaders/` + `text/` + `transitions/`
3. **Phase 3**: Build-Script + JSON-Generierung + Type-Generierung
4. **Phase 4**: Runtime-Integration (HubCanvas liest JSON, Shader aus Dateien)

---

*Stand: 2026-10-23 — bindet an M16–M21*