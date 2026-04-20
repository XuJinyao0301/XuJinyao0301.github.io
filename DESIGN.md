# DESIGN.md

## 1. Visual Theme & Atmosphere
- Theme name: Orbital Editorial
- Design philosophy: 把“技术博客”做成“可探索的数字星图”，首页强调空间感与方向感，阅读区保持克制与高可读。
- Atmosphere keywords: deep-space, calm, precise, exploratory, premium
- One-line tone: `在宇宙尺度里保持工程秩序。`

## 2. Color Palette & Roles
```css
:root {
  --orb-bg-0: #040812;
  --orb-bg-1: #081327;
  --orb-bg-2: #0f1f3e;
  --orb-panel: rgba(9, 17, 34, 0.72);
  --orb-panel-strong: rgba(12, 22, 45, 0.88);
  --orb-line: rgba(153, 197, 255, 0.28);
  --orb-line-soft: rgba(153, 197, 255, 0.14);
  --orb-text: #d8e4ff;
  --orb-text-muted: #9fb2d9;
  --orb-title: #f4f8ff;
  --orb-accent: #72c3ff;
  --orb-accent-2: #4de2c0;

  --orb-bg-0-rgb: 4, 8, 18;
  --orb-bg-1-rgb: 8, 19, 39;
  --orb-panel-rgb: 9, 17, 34;
  --orb-accent-rgb: 114, 195, 255;
  --orb-accent-2-rgb: 77, 226, 192;
}
```
- Roles:
- `--orb-bg-*`: global background depth
- `--orb-panel*`: cards and floating controls
- `--orb-line*`: borders and orbit trajectories
- `--orb-accent*`: interactive highlights and CTA

## 3. Typography Rules
- Google Fonts:
- `https://fonts.googleapis.com/css2?family=Space+Grotesk:wght@400;500;700&family=Noto+Sans+SC:wght@400;500;700&display=swap`
- Font stack:
- Headings/UI: `"Space Grotesk", "Noto Sans SC", "Avenir Next", sans-serif`
- Body: `"Noto Sans SC", "PingFang SC", "Hiragino Sans GB", sans-serif`
- Scale:
- Hero title: `clamp(2.1rem, 6vw, 5rem)` / 700 / letter-spacing `-0.02em`
- H2: `clamp(1.4rem, 3vw, 2rem)`
- Body: `1rem` to `1.08rem`, line-height `1.7-1.85`
- Meta: `0.82rem-0.9rem`
- Forbidden fonts: Inter, Roboto, Arial, Times New Roman

## 4. Component Stylings
```css
.orb-btn {
  border: 1px solid rgba(var(--orb-accent-rgb), 0.42);
  background: linear-gradient(130deg, rgba(var(--orb-accent-rgb), 0.95), rgba(var(--orb-accent-2-rgb), 0.9));
  color: #041122;
}
.orb-btn:hover { transform: translateY(-2px); filter: brightness(1.05); }
.orb-btn:active { transform: translateY(0); filter: brightness(0.96); }
.orb-btn:focus-visible { outline: 2px solid rgba(var(--orb-accent-rgb), 0.8); outline-offset: 2px; }
.orb-btn:disabled { opacity: 0.5; cursor: not-allowed; }

.orb-ghost {
  border: 1px solid var(--orb-line);
  background: rgba(var(--orb-panel-rgb), 0.56);
  color: var(--orb-text);
}
.orb-ghost:hover { border-color: rgba(var(--orb-accent-rgb), 0.5); background: rgba(var(--orb-panel-rgb), 0.82); }
.orb-ghost:active { transform: translateY(0); }
.orb-ghost:focus-visible { outline: 2px solid rgba(var(--orb-accent-rgb), 0.72); outline-offset: 2px; }
.orb-ghost:disabled { opacity: 0.45; }

.orb-card {
  background: linear-gradient(165deg, var(--orb-panel-strong), var(--orb-panel));
  border: 1px solid var(--orb-line-soft);
}
.orb-card:hover { border-color: var(--orb-line); transform: translateY(-4px); }
.orb-card:active { transform: translateY(-1px); }
.orb-card:focus-within { border-color: rgba(var(--orb-accent-rgb), 0.62); }
.orb-card[aria-disabled='true'] { opacity: 0.5; }
```

## 5. Layout Principles
- Max width: `1240px`
- Hero layout: desktop two-column (`minmax(0,1.05fr)` + `minmax(380px,0.95fr)`), mobile single-column
- Spacing scale: `0.25, 0.5, 0.75, 1, 1.5, 2, 3, 4rem`
- Floating module spacing: at least `24px`
- Avoid full-width hard blocks on hero; keep breathing margins via `padding-inline: clamp(1rem, 4vw, 3rem)`

## 6. Depth & Elevation
- Elevation 1 (base cards): `0 14px 36px rgba(0,0,0,0.28)`
- Elevation 2 (hero panel): `0 24px 66px rgba(2,8,20,0.5)`
- Elevation 3 (interactive controls): `0 26px 72px rgba(2,10,28,0.62)`
- Glass usage: `backdrop-filter <= 14px` only on nav / floating controls

## 7. Animation & Interaction
- Interaction tier: **L3** (immersive)
- Motion set:
- Hero reveal: title/paragraph/buttons stagger in `480-680ms`
- Earth sphere: continuous axial rotate (`16s linear infinite`)
- Orbit rings: counter-rotation layers for depth
- Toolbar: pointer drag updates `--orbital-angle`; releasing keeps inertia-like eased settle
- Hover states: only transform + opacity + border color
- JS constraints:
- pointermove always throttled by `requestAnimationFrame`
- no global scroll-jacking
- no persistent heavy WebGL loop (CSS-only Earth)

## 8. Do's and Don'ts
- Do: keep primary copy concise and mission-oriented
- Do: maintain high contrast for text on deep backgrounds
- Do: route resume to About instead of exposing full CV on home
- Do: keep feature interactions discoverable with visible hint text
- Do: preserve post list readability as editorial surface
- Do: keep animations reversible and lightweight
- Do: ensure touch-first behavior on drag module fallback
- Do: respect reduced-motion users
- Don't: use purple-dominant gradients
- Don't: stack blur filters on moving elements
- Don't: hide critical nav actions inside hover-only UI
- Don't: make drag interaction mandatory for navigation
- Don't: overload home with full resume details
- Don't: break archive/article page visual consistency

## 9. Responsive Behavior
- Breakpoints:
- Desktop: `>= 1200px`
- Tablet: `768px - 1199px`
- Mobile: `<= 767px`
- Mobile behavior:
- Hero converts to vertical stack
- Orbital toolbar shrinks and allows horizontal pan fallback
- CTA buttons full width if viewport `< 420px`
- Accessibility:
- minimum tap target `44px`
- focus ring always visible for keyboard users
- `prefers-reduced-motion: reduce` disables continuous rotate and long transitions
