# Restaurant Website CSS Techniques
*Pure HTML/CSS signature techniques for premium local restaurant demo sites*

## Overview
These are the CSS craft techniques used to make restaurant websites look custom-designed rather than templated. Applied across 3 restaurant builds (Smoky Stacks, BGM Burgerman, Chapo's Burger). No frameworks — pure CSS only.

---

## Signature Techniques

### 1. Grain Texture Overlay (body-level)
Adds subtle film grain across entire page — elevates from flat to premium instantly.
```css
body::before {
  content: '';
  position: fixed;
  inset: 0;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)'/%3E%3C/svg%3E");
  opacity: 0.035;
  pointer-events: none;
  z-index: 9999;
}
```

### 2. Outline / Ghost Text
Large decorative text using stroke only, no fill. Creates depth layers.
```css
.ghost-text {
  -webkit-text-stroke: 2px var(--accent);
  color: transparent;
  font-size: clamp(4rem, 20vw, 28rem);
  opacity: 0.06;
  pointer-events: none;
  user-select: none;
}
```

### 3. Fluid Typography with clamp()
Headings that scale smoothly across all screen sizes — no breakpoints needed.
```css
h1 {
  font-size: clamp(3rem, 10vw, 10rem);
}
/* Chapo's hero: clamp(6rem, 16vw, 18rem) */
```

### 4. Frosted Glass Nav
Sticky nav that becomes translucent as user scrolls over content.
```css
nav {
  position: sticky;
  top: 0;
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  background: rgba(8, 8, 8, 0.85);
  border-bottom: 1px solid rgba(255, 255, 255, 0.06);
  z-index: 100;
}
```

### 5. Ember / Glow Blobs (ambient radial gradients)
Background atmosphere — replaces plain dark backgrounds with depth.
```css
.hero::before {
  content: '';
  position: absolute;
  top: 20%;
  left: 30%;
  width: 600px;
  height: 400px;
  background: radial-gradient(ellipse, rgba(255, 94, 0, 0.12), transparent 70%);
  pointer-events: none;
}
```

### 6. Diagonal Clip-Path Panels
For split hero layouts — the right panel gets a diagonal edge.
```css
.hero-right::before {
  content: '';
  position: absolute;
  left: -60px;
  top: 0;
  width: 120px;
  height: 100%;
  background: var(--surface);
  clip-path: polygon(60px 0, 100% 0, 40px 100%, 0 100%);
}
```

### 7. Collage Cards (slight rotation)
Menu cards with micro-rotation give a hand-placed, editorial feel.
```css
.card:nth-child(odd) { transform: rotate(0.3deg); }
.card:nth-child(even) { transform: rotate(-0.5deg); }
.card:nth-child(3n) { transform: rotate(0.8deg); }
```

### 8. CSS Ticker / Marquee
Infinite scrolling text strip — brand name, taglines, etc.
```css
@keyframes ticker {
  from { transform: translateX(0); }
  to { transform: translateX(-50%); }
}
.ticker-track {
  display: flex;
  animation: ticker 20s linear infinite;
  white-space: nowrap;
}
/* Duplicate content in HTML to loop seamlessly */
```

### 9. Card Glow on Hover
Colored border + outer glow that matches brand accent color.
```css
.card:hover {
  border-color: var(--accent);
  box-shadow: 0 0 30px rgba(255, 94, 0, 0.2);
  transform: translateY(-4px);
}
```

### 10. Float Animation (hero image)
Subtle up-down float on food photography — adds life without being distracting.
```css
@keyframes float-burger {
  0%, 100% { transform: translateY(0px) rotate(2deg); }
  50% { transform: translateY(-20px) rotate(2deg); }
}
.hero-img {
  animation: float-burger 6s ease-in-out infinite;
  filter: drop-shadow(0 30px 60px rgba(255, 94, 0, 0.4));
}
```

### 11. Vertical Text (rotated side labels)
Side labels using writing-mode — good for about sections, section labels.
```css
.side-label {
  writing-mode: vertical-rl;
  transform: rotate(180deg);
  font-size: 0.65rem;
  letter-spacing: 0.3em;
  text-transform: uppercase;
  opacity: 0.5;
}
```

### 12. Dark Hero with Background Image
Full-screen hero using a food photo at very low brightness — text reads over it perfectly.
```css
.hero {
  position: relative;
  min-height: 100vh;
  background: url('food-photo.jpg') center/cover no-repeat;
}
.hero::after {
  content: '';
  position: absolute;
  inset: 0;
  background: rgba(0, 0, 0, 0.82);
  /* Or use filter: brightness(0.18) on an <img> inside */
}
```

---

## 12-Column Bento Grid (Menu Layout)
```css
.menu-grid {
  display: grid;
  grid-template-columns: repeat(12, 1fr);
  gap: 1.5rem;
}
/* Cards span varied columns: 5, 4, 3, 7, 5 etc. */
.card-wide { grid-column: span 5; }
.card-mid { grid-column: span 4; }
.card-narrow { grid-column: span 3; }
```

---

## Three Restaurant Themes Built

### Smoky Stacks — Dark Ember
| Role | Hex |
|------|-----|
| Background | `#080808` |
| Accent | `#FF5E00` |
| Surface | `#111111` |
| Border | `rgba(255,94,0,0.15)` |

- Fonts: Bebas Neue (display), Oswald (headings), Inter (body)
- Layout: 3-col equal menu grid, ambient glow hero, floating burger
- Effect: ember radial glow blobs, float animation

### BGM Burgerman — Bold Red/Yellow
| Role | Hex |
|------|-----|
| Primary | `#D62B2B` |
| Secondary | `#FFB800` |
| Background | `#111111` |
| Cream | `#F5F0E8` |

- Fonts: Bebas Neue (display), Barlow Condensed (headings), Barlow (body)
- Layout: split 55/45 hero, 12-col bento menu, diagonal clip on right panel
- Effect: color blocking, sticker badge (rotate -8deg)

### Chapo's Burger — Dark + Neon Green
| Role | Hex |
|------|-----|
| Background | `#0D0D0D` |
| Accent | `#BEFF00` |
| Cream | `#F5F0E8` |
| Red | `#FF1744` |

- Fonts: Anton (display), Space Grotesk (body)
- Layout: full-bleed dark hero, collage menu with micro-rotations
- Effect: grain texture, `-webkit-text-stroke` outline text, neon glow blob

---

## Google Fonts Used
- `Bebas+Neue` — ultra-condensed, all-caps display
- `Oswald` — condensed, strong headings
- `Barlow+Condensed` + `Barlow` — versatile pair
- `Anton` — bold display, similar to Bebas
- `Space+Grotesk` — geometric, modern body
- `Inter` — neutral, clean body

---

## Projects Using This
- Smoky Stacks: `D:/2025_project_portfolio/smokyweb/index.html`
- BGM Burgerman: `D:/2025_project_portfolio/bgmburgerman/index.html`
- Chapo's Burger: `D:/2025_project_portfolio/chaposburger/index.html`

---
*Documented: 2026-05-03*
