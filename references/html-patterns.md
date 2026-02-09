# HTML Patterns for Walkthrough Generation

Reference for generating interactive walkthrough HTML files using React (UMD) + Shiki (ESM) + Tailwind CDN + Mermaid.

## Architecture: `<script type="module">` (No Babel)

- **Native ES modules** — template literals, top-level `await`, ESM imports all work natively
- **Shiki via ESM** — `import { createHighlighter } from 'shiki'`
- **React/ReactDOM as UMD globals** — `window.React`, `window.ReactDOM`
- **No JSX** — use `React.createElement()` everywhere
- All examples below show JSX for readability — generated HTML must use `React.createElement()`

## Design Principles

1. **Always dark mode** — Black bg, white text, purple accents. Set `<html color-scheme: dark>`, `<body bg-wt-bg>`
2. **Quick mental model** — Readable in <2 min, not a code reference
3. **TL;DR first** — Summary card above diagram
4. **Full-size diagram** — Mermaid at natural size, never squished
5. **Pan + zoom** — Scroll zooms toward cursor, drag pans, auto-fit on load
6. **Floating detail overlay** — Right side, close (×), description + code snippet
7. **Shiki highlighting** — `vitesse-dark` theme, every node has a code snippet
8. **React + Tailwind CDN** — Declarative components, utility-first styling

## Color Palette & Tailwind Config

| Token | Hex | Use |
|-------|-----|-----|
| `wt-bg` | `#000000` | Page background |
| `wt-surface` | `#0a0a0a` | Panels, overlays |
| `wt-raised` | `#141414` | Hover states |
| `wt-border` | `#2a2a2a` | Borders, dividers |
| `wt-fg` | `#ffffff` | Primary text |
| `wt-muted` | `#a0a0a0` | Secondary text |
| `wt-accent` | `#a855f7` | Purple accent |
| `wt-file` | `#c084fc` | File paths |
| `wt-red` | `#ef4444` | Close button hover |

### Node Type Colors

| Type | Fill | Stroke | Text |
|------|------|--------|------|
| component | `#a855f7` | `#c084fc` | white |
| composable | `#7c3aed` | `#a78bfa` | white |
| utility | `#6d28d9` | `#8b5cf6` | white |
| external | `#525252` | `#737373` | white |
| event | `#d8b4fe` | `#e9d5ff` | black |
| data | `#9333ea` | `#a855f7` | white |

### Tailwind Config Block

```html
<script>
  tailwind.config = {
    theme: { extend: { colors: {
      wt: {
        bg: '#000000', surface: '#0a0a0a', raised: '#141414',
        border: '#2a2a2a', fg: '#ffffff', muted: '#a0a0a0',
        accent: '#a855f7', file: '#c084fc', red: '#ef4444',
      },
      node: {
        component: '#a855f7', composable: '#7c3aed', utility: '#6d28d9',
        external: '#525252', event: '#d8b4fe', data: '#9333ea',
      },
    }}},
  };
</script>
```

### Mermaid classDef

```
classDef component fill:#a855f7,stroke:#c084fc,color:#fff
classDef composable fill:#7c3aed,stroke:#a78bfa,color:#fff
classDef utility fill:#6d28d9,stroke:#8b5cf6,color:#fff
classDef external fill:#525252,stroke:#737373,color:#fff
classDef event fill:#d8b4fe,stroke:#e9d5ff,color:#000
classDef data fill:#9333ea,stroke:#a855f7,color:#fff
```

## CDN Dependencies

```html
<script src="https://cdn.tailwindcss.com"></script>
<script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/mermaid@11/dist/mermaid.min.js"></script>
```

No Babel. Shiki loaded via ESM `import` inside `<script type="module">`.

## Minimal CSS

```css
.mermaid-wrap svg { max-width: none !important; height: auto !important; }

/* Node hover (flowchart + ER) */
.mermaid-wrap .node { cursor: pointer; }
.mermaid-wrap .node:hover rect,
.mermaid-wrap .node:hover polygon,
.mermaid-wrap .node:hover circle,
.mermaid-wrap .node:hover .label-container { filter: brightness(1.3); transition: filter .15s; }
.mermaid-wrap .er.entityBox { cursor: pointer; }
.mermaid-wrap g:has(.er.entityBox):hover .er.entityBox { filter: brightness(1.3); transition: filter .15s; }

/* Detail panel body */
.dt-body p { color:#a0a0a0; font-size:.88rem; line-height:1.65; margin-bottom:10px; }
.dt-body p code {
  background:rgba(168,85,247,.12); padding:1px 6px; border-radius:4px;
  font-family:'SF Mono','Fira Code',monospace; font-size:.82rem; color:#c084fc;
}

/* Shiki + fallback code blocks (shared base) */
.dt-body .shiki, .dt-body pre.code-fallback {
  background: #000000; border: 1px solid #2a2a2a; border-radius: 8px;
  padding: 14px 16px; overflow-x: auto; margin: 8px 0 14px;
}
.dt-body .shiki { background: #000000 !important; }
.dt-body .shiki code, .dt-body pre.code-fallback code {
  font-family: 'SF Mono','Fira Code',monospace; font-size: .78rem; line-height: 1.55;
  background: none; padding: 0; border-radius: 0;
}
.dt-body .shiki code { color: inherit; }
.dt-body pre.code-fallback code { color: #e0e0e0; }
```

## Mermaid Initialization

### Base Theme Variables (shared by all diagram types)

```js
const MERMAID_THEME = {
  primaryColor: '#0a0a0a', primaryTextColor: '#ffffff', primaryBorderColor: '#2a2a2a',
  lineColor: '#a0a0a0', background: '#000000',
};
```

### Flowchart Config

```js
mermaid.initialize({
  startOnLoad: false, theme: 'dark', securityLevel: 'loose',
  themeVariables: {
    ...MERMAID_THEME,
    secondaryColor: '#000000', tertiaryColor: '#000000',
    mainBkg: '#0a0a0a', nodeBorder: '#2a2a2a',
    clusterBkg: 'rgba(10,10,10,0.8)', clusterBorder: '#7c3aed',
    titleColor: '#ffffff', edgeLabelBackground: 'transparent',
  },
  flowchart: { useMaxWidth: false, htmlLabels: true, curve: 'basis' },
});
```

### ER Diagram Config

For database-focused walkthroughs using `erDiagram` syntax:

```js
mermaid.initialize({
  startOnLoad: false, theme: 'dark', securityLevel: 'loose',
  themeVariables: {
    ...MERMAID_THEME,
    entityBkg: '#0a0a0a', entityBorder: '#7c3aed', entityTextColor: '#ffffff',
    attributeBackgroundColorEven: '#0a0a0a', attributeBackgroundColorOdd: '#141414',
    labelColor: '#a0a0a0', relationColor: '#a855f7',
  },
  er: { useMaxWidth: false, layoutDirection: 'TB' },
});
```

**Critical**: `useMaxWidth: false` (natural SVG size), `securityLevel: 'loose'` (click callbacks). ER `layoutDirection`: `'TB'` or `'LR'`.

## Data Structures

### SUMMARY (TL;DR card)

```js
const SUMMARY = "2-3 sentence overview. What does this system do at the highest level? Plain text, no formatting.";
```

### NODES

```js
const NODES = {
  nodeId: {
    title: "Drawing Interaction",
    description: "1-2 plain-text sentences. What is this? Why does it exist?",
    files: ["app/features/tools/useDrawingInteraction.ts"],
    code: `const element = createElement(tool, startPoint, currentPoint)`,
    lang: "typescript",  // default; set explicitly for vue, json, css, etc.
  },
};
```

- `description`: plain text, rendered as `<p>` (not `dangerouslySetInnerHTML`)
- `code`: **required** — every node must have a useful snippet (1-5 lines). Pick the most representative piece: a key function call, type definition, config, or core algorithm. Use template literals for multi-line.
- `lang`: required for every node (default `"typescript"`). Set explicitly for vue, json, css, etc.
- `files`: array of `"path"` or `"path:lines"` strings

### LEGEND

```js
const LEGEND = [
  { label: 'Component', color: 'bg-node-component' },
  // ...
];
```

### Shiki Languages

| ID | Use |
|----|-----|
| `typescript` | `.ts` (default) |
| `vue` | `.vue` SFC |
| `vue-html` | Vue templates |
| `json` | Config files |
| `css` / `javascript` / `bash` | As needed |

## React Component Architecture

```
App
├── Header (fixed, gradient fade)
├── Summary (TL;DR card below header)
├── DiagramViewport (full screen, pan/zoom via usePanZoom)
│   └── MermaidDiagram (SVG into ref)
├── ZoomControls (fixed bottom-left)
├── Legend (fixed bottom-center)
├── DetailPanel (fixed right, conditional)
│   ├── Close (×), Title, Description (<p>)
│   ├── CodeBlock (Shiki or fallback — every node has one)
│   └── Files list
└── KeyboardHint (fixed bottom-right)
```

### usePanZoom Hook

Uses `useRef` for transform state (not `useState`) to avoid re-renders on every frame:

```jsx
function usePanZoom() {
  const viewportRef = useRef(null), canvasRef = useRef(null), zoomDisplayRef = useRef(null);
  const st = useRef({ zoom: 1, panX: 0, panY: 0 });
  const drag = useRef({ on: false, lx: 0, ly: 0 });

  const apply = useCallback(() => {
    const { zoom, panX, panY } = st.current;
    if (canvasRef.current)
      canvasRef.current.style.transform = `translate(${panX}px,${panY}px) scale(${zoom})`;
    if (zoomDisplayRef.current)
      zoomDisplayRef.current.textContent = Math.round(zoom * 100) + '%';
  }, []);

  const fitToScreen = useCallback(() => {
    const svg = canvasRef.current?.querySelector('svg');
    const vp = viewportRef.current;
    if (!svg || !vp) return;
    const s = st.current;
    const vw = vp.clientWidth, vh = vp.clientHeight;
    const sw = svg.getBoundingClientRect().width / s.zoom;
    const sh = svg.getBoundingClientRect().height / s.zoom;
    const fit = Math.max(0.15, Math.min(2, Math.min((vw - 80) / sw, (vh - 80) / sh)));
    s.zoom = fit; s.panX = (vw - sw * fit) / 2; s.panY = (vh - sh * fit) / 2;
    apply();
  }, [apply]);

  useEffect(() => {
    const vp = viewportRef.current;
    if (!vp) return;
    const onWheel = (e) => {
      e.preventDefault();
      const r = vp.getBoundingClientRect();
      const mx = e.clientX - r.left, my = e.clientY - r.top, s = st.current;
      const f = e.deltaY < 0 ? 1.12 : 1 / 1.12;
      const nz = Math.min(4, Math.max(0.15, s.zoom * f)), sc = nz / s.zoom;
      s.panX = mx - sc * (mx - s.panX); s.panY = my - sc * (my - s.panY); s.zoom = nz;
      apply();
    };
    const onDown = (e) => {
      if (e.target.closest('.node')) return;
      drag.current = { on: true, lx: e.clientX, ly: e.clientY };
      vp.setPointerCapture(e.pointerId);
    };
    const onMove = (e) => {
      const d = drag.current; if (!d.on) return;
      st.current.panX += e.clientX - d.lx; st.current.panY += e.clientY - d.ly;
      d.lx = e.clientX; d.ly = e.clientY; apply();
    };
    const onUp = () => { drag.current.on = false; };
    vp.addEventListener('wheel', onWheel, { passive: false });
    vp.addEventListener('pointerdown', onDown);
    vp.addEventListener('pointermove', onMove);
    vp.addEventListener('pointerup', onUp);
    vp.addEventListener('pointercancel', onUp);
    window.addEventListener('resize', fitToScreen);
    return () => { /* removeEventListeners */ };
  }, [apply, fitToScreen]);

  const zoomIn = useCallback(() => { st.current.zoom = Math.min(4, st.current.zoom * 1.25); apply(); }, [apply]);
  const zoomOut = useCallback(() => { st.current.zoom = Math.max(0.15, st.current.zoom / 1.25); apply(); }, [apply]);
  return { viewportRef, canvasRef, zoomDisplayRef, zoomIn, zoomOut, fitToScreen };
}
```

### MermaidDiagram (Flowchart)

**Critical**: After `innerHTML = svg`, you MUST call `bindFunctions?.(ref.current)` — without it, click handlers are lost.

```jsx
function MermaidDiagram({ onNodeClick }) {
  const ref = useRef(null);
  useEffect(() => {
    window.nodeClickHandler = onNodeClick;
    mermaid.initialize({ /* see flowchart config */ });
    mermaid.render('walkthrough-diagram', DIAGRAM).then(({ svg, bindFunctions }) => {
      if (ref.current) { ref.current.innerHTML = svg; bindFunctions?.(ref.current); }
    });
    return () => { delete window.nodeClickHandler; };
  }, [onNodeClick]);
  return <div ref={ref} className="mermaid-wrap" />;
}
```

### MermaidERDiagram (Database walkthroughs)

ER diagrams don't support `click` callback syntax — attach handlers manually to `.entityLabel` groups:

```jsx
function MermaidERDiagram({ onEntityClick }) {
  const ref = useRef(null);
  useEffect(() => {
    mermaid.initialize({ /* see ER config */ });
    mermaid.render('walkthrough-diagram', DIAGRAM).then(({ svg, bindFunctions }) => {
      if (!ref.current) return;
      ref.current.innerHTML = svg;
      bindFunctions?.(ref.current);
      ref.current.querySelectorAll('.entityLabel').forEach(label => {
        const name = label.textContent?.trim(), group = label.closest('g');
        if (!name || !group) return;
        group.style.cursor = 'pointer';
        group.addEventListener('click', () => onEntityClick(name));
      });
    });
  }, [onEntityClick]);
  return <div ref={ref} className="mermaid-wrap" />;
}
```

### Summary (Accordion — collapsed by default)

The summary is a collapsible accordion. **It starts collapsed** so the diagram is the first thing users see. Clicking toggles between a compact pill and the full summary card.

```jsx
function Summary({ collapsed, onToggle }) {
  if (collapsed) {
    return (
      <div onClick={onToggle}
        className="fixed top-16 left-6 z-10 px-3 py-1.5 bg-wt-surface/60 backdrop-blur border border-wt-border rounded-full shadow-lg cursor-pointer hover:bg-wt-surface/80 transition-all duration-300">
        <span className="text-xs text-wt-muted font-medium">ℹ︎ Summary</span>
      </div>
    );
  }
  return (
    <div onClick={onToggle}
      className="fixed top-16 left-6 z-10 max-w-lg px-4 py-3 bg-wt-surface/80 backdrop-blur border border-wt-border rounded-lg shadow-lg cursor-pointer transition-all duration-300">
      <p className="text-sm text-wt-muted leading-relaxed">{SUMMARY}</p>
      <p className="text-[0.65rem] text-wt-muted/50 mt-1.5">Click to dismiss</p>
    </div>
  );
}
```

In `App`, initialize the summary as **collapsed** (`true`):

```jsx
const [summaryCollapsed, setSummaryCollapsed] = useState(true);
// ...
<Summary collapsed={summaryCollapsed} onToggle={() => setSummaryCollapsed(c => !c)} />
```
```

### DetailPanel

Pass both `nodeId` (for `HIGHLIGHTED` lookup) and `node` (data object):

```jsx
function DetailPanel({ nodeId, node, onClose }) {
  useEffect(() => {
    const onKey = (e) => { if (e.key === 'Escape') onClose(); };
    document.addEventListener('keydown', onKey);
    return () => document.removeEventListener('keydown', onKey);
  }, [onClose]);
  const codeHtml = HIGHLIGHTED[nodeId];
  return (
    <div className="fixed top-4 right-4 bottom-4 w-[560px] z-30 bg-wt-surface border border-wt-border rounded-xl shadow-2xl flex flex-col overflow-hidden">
      <button onClick={onClose}
        className="absolute top-3 right-3 z-10 w-7 h-7 rounded-md border border-wt-border bg-wt-raised text-wt-muted flex items-center justify-center text-lg hover:bg-wt-red hover:border-wt-red hover:text-white transition-colors">
        &times;
      </button>
      <div className="flex-1 overflow-y-auto p-5">
        <h2 className="text-lg font-bold text-wt-fg mb-3 pr-9">{node.title}</h2>
        <div className="dt-body"><p>{node.description}</p></div>
        {node.code && (
          <div className="dt-body">
            {codeHtml
              ? <div dangerouslySetInnerHTML={{ __html: codeHtml }} />
              : <pre className="code-fallback"><code>{node.code}</code></pre>}
          </div>
        )}
        {node.files?.length > 0 && (
          <div className="mt-4 pt-3 border-t border-wt-border">
            <div className="text-[0.7rem] uppercase tracking-wider text-wt-muted font-semibold mb-1.5">Files</div>
            <code className="text-sm text-wt-file font-mono leading-relaxed">
              {node.files.map((f, i) => <span key={i}>{f}<br/></span>)}
            </code>
          </div>
        )}
      </div>
    </div>
  );
}
```

### App (top-level wiring)

```jsx
function App() {
  const [activeId, _setActiveId] = useState(null);
  const [summaryCollapsed, setSummaryCollapsed] = useState(true); // collapsed by default
  const pz = usePanZoom();

  const setActiveNode = useCallback((nodeId) => {
    _setActiveId(nodeId);
    document.querySelectorAll('.mermaid-wrap .node').forEach(n => { n.style.opacity = nodeId ? '0.4' : '1'; });
    if (nodeId) {
      const el = document.querySelector(`.mermaid-wrap .node[id*="${nodeId}"]`);
      if (el) el.style.opacity = '1';
    }
  }, []);
  const closeDetail = useCallback(() => {
    _setActiveId(null);
    document.querySelectorAll('.mermaid-wrap .node').forEach(n => { n.style.opacity = '1'; });
  }, []);
  useEffect(() => { setTimeout(pz.fitToScreen, 600); }, [pz.fitToScreen]);

  return (
    <>
      <header className="fixed top-0 inset-x-0 z-10 px-6 py-3.5 bg-gradient-to-b from-wt-bg to-transparent pointer-events-none">
        <h1 className="text-base font-semibold text-wt-fg">{"TITLE_HERE"}</h1>
        <p className="text-sm text-wt-muted mt-0.5">{"SUBTITLE_HERE"}</p>
      </header>
      <Summary collapsed={summaryCollapsed} onToggle={() => setSummaryCollapsed(c => !c)} />
      <div ref={pz.viewportRef} className="w-full h-screen overflow-hidden cursor-grab active:cursor-grabbing">
        <div ref={pz.canvasRef} className="origin-top-left will-change-transform inline-block p-[80px_60px_60px]">
          <MermaidDiagram onNodeClick={setActiveNode} />
        </div>
      </div>
      <ZoomControls zoomDisplayRef={pz.zoomDisplayRef} onZoomIn={pz.zoomIn} onZoomOut={pz.zoomOut} onFit={pz.fitToScreen} />
      <div className="fixed bottom-5 left-1/2 -translate-x-1/2 z-20 flex gap-4 px-4 py-2 bg-wt-surface border border-wt-border rounded-lg shadow-xl">
        {LEGEND.map(l => (
          <span key={l.label} className="flex items-center gap-1.5 text-xs text-wt-muted">
            <span className={`w-2 h-2 rounded-full ${l.color}`} />{l.label}
          </span>
        ))}
      </div>
      {activeId && NODES[activeId] && <DetailPanel nodeId={activeId} node={NODES[activeId]} onClose={closeDetail} />}
      <div className="fixed bottom-5 right-5 z-20 text-xs text-wt-muted opacity-50">
        <kbd>Scroll</kbd> zoom · <kbd>Drag</kbd> pan · Click nodes
      </div>
    </>
  );
}
ReactDOM.createRoot(document.getElementById('root')).render(<App />);
```

## Complete Script Block Order

```js
// 1. Import Shiki
import { createHighlighter } from 'https://cdn.jsdelivr.net/npm/shiki@3.22.0/+esm'
// 2. Destructure React globals
const { useState, useEffect, useRef, useCallback } = React;
// 3. SUMMARY, 4. DIAGRAM, 5. NODES, 6. LEGEND
// 7. Shiki init + pre-highlight
const langs = [...new Set(Object.values(NODES).map(n => n.lang).filter(Boolean))];
if (langs.length === 0) langs.push('typescript');
let highlighter = null;
try { highlighter = await createHighlighter({ themes: ['vitesse-dark'], langs }); }
catch (e) { console.warn('Shiki failed:', e); }
const HIGHLIGHTED = {};
for (const [id, node] of Object.entries(NODES)) {
  if (node.code && highlighter) {
    try {
      HIGHLIGHTED[id] = highlighter.codeToHtml(node.code, { lang: node.lang || 'typescript', theme: 'vitesse-dark' });
    } catch (e) { console.warn(`Highlight failed for ${id}:`, e); }
  }
}
// 8. React components (usePanZoom, MermaidDiagram, Summary, DetailPanel, ZoomControls, App)
//    using React.createElement() — NOT JSX
// 9. Mount
ReactDOM.createRoot(document.getElementById('root')).render(React.createElement(App));
```

## Critical Rules

1. **bindFunctions is mandatory** — after `innerHTML = svg`, call `bindFunctions?.(ref.current)` or clicks won't work
2. **Node ID consistency** — Mermaid node ID, `click` binding ID, and `NODES` key must match exactly
3. **Set `window.nodeClickHandler` before `mermaid.render()`** so click bindings resolve
4. **Shiki graceful degradation** — always try/catch `createHighlighter()`. If undefined, DetailPanel renders `<pre class="code-fallback">`
5. **Languages auto-collected** from NODES data — never hardcode the Shiki language list
6. **Keep diagrams small** — 5-12 nodes max, 2-4 subgroups
7. **`useRef` for pan/zoom** — update DOM directly, no React re-renders during drag
8. **Auto-fit on load** — `setTimeout(pz.fitToScreen, 600)` waits for Mermaid render
9. **Node clicks vs pan** — check `e.target.closest('.node')` in pointerdown to let clicks through
10. **Plain text descriptions** — use `description` (string) not `content` (HTML), render as `<p>`
