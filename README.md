# Tvorra — Onboarding Prototype

Clickable HTML/CSS/JS prototype of the Tvorra iOS app onboarding flow.
Single file (`index.html`, ~1544 lines), no dependencies, no build step.

**Live:** https://dmytroratsko.github.io/tvorra-prototyping/

---

## Screens & Flow

```
s-splash → s-gender → s-effect → s-preview → s-loading → s-paywall → s-success
```

| Screen ID | Description |
|-----------|-------------|
| `s-splash` | Intro screen — collage bg, stars badge, "1M+ Videos Created", CTA |
| `s-gender` | Gender select — CSS photo-card icons (female / male / other) |
| `s-effect` | AI effect type select — Kisses / Runway / Glam / Viral |
| `s-preview` | 3-slide preview carousel per effect — fullscreen photo scene + decos |
| `s-loading` | "Preparing..." loader — 4 animated checklist steps before paywall |
| `s-paywall` | Subscription screen — collage, plans, benefits, sticky CTA |
| `s-success` | Success confirmation — star orb animation |

---

## Design Tokens

```css
--bg:      #08080f   /* deep dark background */
--accent:  #C94BE0   /* purple primary */
--accent2: #7040D8   /* purple secondary */
--grad:    linear-gradient(135deg, #C94BE0, #7040D8)
```

iPhone shell: `393×852px`, `transform:scale()` for viewport fit.

---

## Key CSS Systems

### Gender icons (`.gi-icon`)
CSS-only person silhouettes, 42×42px rounded squares.
Classes: `.gi-female`, `.gi-male`, `.gi-other`
Uses `--gi-scene`, `--gi-body`, `--gi-head`, `--gi-hair`, `--gi-light` custom props.

### Deco shapes (`.ps-deco`)
CSS clip-path floating decorations on preview slides.
Types: `.deco-spark` (4-point star), `.deco-heart`, `.deco-circle`, `.deco-diamond`
Color passed via `--dc` CSS custom property.

### Photo cards (`.photo-card`)
Full-scene person placeholders — `::before` body + `::after` head + `.photo-hair`.
Used in paywall collage and preview slides.

---

## Screen Details

### s-splash
- Background: `#080810` full viewport
- Badge: `⭐⭐⭐⭐⭐` + laurel SVG branches + "1M+ Videos Created" (2 lines, bold)
- Title: large white + accent gradient text
- CTA: gradient pill button → navigates to `s-gender`

### s-gender
- 3 list rows with CSS photo icons + label + chevron
- Tap selects row, 200ms delay → auto-advances to `s-effect`

### s-effect
- 4 effect cards in 2×2 grid
- Each card: photo-card bg + label
- Effects: Kisses / Runway / Glam / Viral
- Tap → sets `_effect` global → navigates to `s-preview`

### s-preview
- `showPreviews()` renders 3 slides per effect
- Each slide: fullscreen photo scene + CSS deco shapes + title + description
- Sticky "Next" / "Skip" button at bottom
- Skip → `s-loading`, last slide Next → `s-loading`

### s-loading
- `startLoading()` called on nav to this screen
- 4 steps animate: pending → active (spinner) → done (checkmark)
- Step timing: ~900–1300ms each
- After all done → auto-navigates to `s-paywall`

### s-paywall
- Structure: `flex-column` — close btn + `pw-scroll` + `pw-sticky-cta`
- `pw-scroll`: collage (photo cards) + offer + plans (yearly/weekly) + benefits + social proof + guarantee
- `pw-sticky-cta`: single "Continue" button pinned at bottom with gradient fade above
- Countdown timer: 8 min, ticks every second
- Plans toggle: yearly (selected by default) / weekly

### s-success
- SVG star orb with glow animation
- "You're all set!" title

---

## Navigation

```js
nav('screen-id')   // navigate forward
goBack()           // navigate back
```

Transitions: CSS `opacity` + `translateX`, classes `.active` / `.exit`.

---

## Preview Slide Data

Each effect has 3 slides defined in `pvData` object:
```js
pvData = {
  kisses: { slides: [...], persons: [...] },
  runway: { ... },
  glam:   { ... },
  viral:  { ... }
}
```

Each slide:
```js
{
  title: 'Slide Title',
  desc: 'Description text',
  decos: [
    { t:'deco-heart', c:'rgba(255,80,120,0.75)', x:'10%', y:'12%',
      fs:'26px', op:.5, d:'3s', dl:'0s' },
  ]
}
```

---

## How to Update & Deploy

Edit `index.html` locally, then from Terminal:

```bash
cd /Users/d.ratsko/Downloads/tvorra-onboarding
git add index.html
git commit -m "describe change"
git push
```

Site updates at https://dmytroratsko.github.io/tvorra-prototyping/ within ~1 min.

SSH key `tvorra-prototype` is configured at `~/.ssh/github_key`

---

## Fix History

| Fix | Detail |
|-----|--------|
| Gender emoji → CSS icons | `👩👨🧑` replaced with `.gi-icon` CSS photo-card system |
| Preview deco emoji → CSS shapes | `❤️✨` replaced with `.deco-heart .deco-spark` clip-path shapes |
| Paywall collage overlap | Added `padding-top:54px` to `.pw-scroll` |
| Preview content hidden by button | `.ps-content` padding-bottom `18px` → `168px` |
| Success screen emoji | `🎉` replaced with SVG star in `.suc-orb` |
| Body background | `#111` → `#080810` (deep dark blue-purple) |
| Paywall 3 buttons → 1 sticky | All `pw-cta` inside scroll removed; single `pw-sticky-cta` outside scroll |
| Loading screen added | `s-loading` with animated checklist before paywall |
