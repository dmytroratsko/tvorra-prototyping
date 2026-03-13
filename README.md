# Tvorra — Prototypes

Clickable HTML/CSS/JS prototypes for the Tvorra iOS app. Each screen flow is a self-contained `index.html` in its own subfolder.

## Prototypes

| Folder | Description | Live |
|--------|-------------|------|
| `onboarding/` | Full onboarding flow (splash → gender → effect → preview → paywall → home) | [Open](https://dmytroratsko.github.io/tvorra-prototyping/onboarding/) |

---

## Adding a new prototype

1. Create a new folder: `mkdir new-screen-name`
2. Add `index.html` inside it
3. Push to GitHub — it's live at `dmytroratsko.github.io/tvorra-prototyping/new-screen-name/`
4. Add a row to the table above

---

## Onboarding Prototype

Single file (`onboarding/index.html`, ~1800 lines), no dependencies, no build step.

**Live:** https://dmytroratsko.github.io/tvorra-prototyping/onboarding/

---

## Screens & Flow

```
s-splash → s-gender → s-effect → s-preview → s-loading → s-paywall → s-home
```

| Screen ID | Description |
|-----------|-------------|
| `s-splash` | Intro screen — animated photo mosaic bg, stars badge, "1M+ Videos Created", CTA |
| `s-gender` | Gender select — CSS photo-card icons (female / male / other), auto-advances on tap |
| `s-effect` | Effect type select — 7 rows with real photo thumbnails + sticky CTA |
| `s-preview` | 3-slide carousel per effect — fullscreen Unsplash photo + CSS deco shapes |
| `s-loading` | "Preparing..." — 4-step animated checklist before paywall |
| `s-paywall` | Subscription screen — 3-photo collage with smooth fade, plans, benefits, sticky CTA |
| `s-home` | Personalised feed card — blurred bg, effect photo, metadata, "Try Now" |

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
CSS-generated person placeholders — `::before` body + `::after` head + `.photo-hair`.
When a `.pc-img` child is present, CSS silhouette auto-hides via `:has(.pc-img)` selector.

### Real photo layer (`.pc-img`)
Unsplash CDN image injected inside `.photo-card`:
```css
.pc-img {
  position:absolute; inset:0; z-index:1;
  background-size:cover; background-position:top center;
  border-radius:inherit;
}
.photo-card:has(.pc-img)::before,
.photo-card:has(.pc-img)::after { opacity:0; }
.photo-card:has(.pc-img) .photo-hair { opacity:0; }
```

---

## Screen Details

### s-splash
- Animated photo mosaic grid (rotated, drifting) with real Unsplash photos via `buildMosaic()`
- Overlay gradient fades mosaic into dark bottom
- Badge: `⭐⭐⭐⭐⭐` + laurel SVG branches + "1M+ Videos Created"
- Title: "Welcome to Tvorra 🔥", subtitle, gradient CTA button → `s-gender`

### s-gender
- 3 list rows: Female / Male / Other with CSS photo icons + chevron
- Tap → highlights row, 200ms delay → auto-advances to `s-effect`

### s-effect
- 7 scrollable rows: AI Kisses / Viral Dances / Hugs Nostalgia / TikTok Trends / AI Videos / Text To Video / None of These
- Each row: real Unsplash photo thumbnail (56×66px) via `buildEffectThumbs()` + tag + name + desc + chevron
- Effect keys: `kisses`, `dances`, `hugs`, `tiktok`, `aivid`, `t2v`, `none`
- Tap → sets `_effect` global → enables "See Your Content" sticky CTA
- CTA button calls `showPreviews()` → builds slides → navigates to `s-preview`

### s-preview
- `showPreviews()` builds 3 slides per `_effect` from `PV[]` data + `PV_PHOTOS[]` images
- Each slide: fullscreen Unsplash photo scene + glow + CSS deco shapes + overlay + title/desc
- Slides swipe via `preview-track` translateX
- "Next" → next slide; last slide → `s-loading`

### s-loading
- `startLoading()` called on nav to this screen
- 4 steps animate: pending → active (spinning border) → done (white checkmark)
- Step timing: ~800–1100ms each
- After all done → auto-navigates to `s-paywall`

### s-paywall
- Structure: `flex-column` — `pw-x` close btn + `pw-scroll` + `pw-sticky-cta`
- `pw-scroll`: 3-photo collage (370px, smooth 260px gradient fade) + title + countdown timer + plans + benefits + featured card + social proof mosaic + reviews + guarantee
- Collage built by `buildPwCollage()` with real Unsplash photos
- Social proof mosaic built by `buildPwMosaic()` with real Unsplash photos
- Countdown timer: 9:59, ticks every second, loops
- Plans toggle: yearly (selected by default, "Save 92%") / weekly
- `pw-x` close → `s-home`; Continue → `s-home`

### s-home
- Blurred photo-grid background (`home-bg`) via `buildHomeBg()`
- Dark overlay with `backdrop-filter:blur(14px)`
- Floating sheet card (`home-sheet`) with:
  - X button (→ `s-splash`) + heart like toggle
  - Big photo card with real Unsplash photo per effect + label overlay
  - Effect name, category, generations count
  - "Try Now" white CTA (→ `s-splash`)
- Content personalised to `_effect` via `_homeData[]` and `HOME_PHOTOS[]`

---

## Navigation

```js
nav('screen-id')   // navigate forward
goBack()           // navigate back (uses _hist stack)
```

Transitions: CSS `opacity` + `translateX`, classes `.active` / `.exit`.

Special hooks in `nav()`:
- `s-loading` → calls `startLoading()`
- `s-paywall` → calls `startTimer()`
- `s-home` → calls `showHome()`

---

## Photo System

```js
const UP = (id, w, h) =>
  `https://images.unsplash.com/photo-${id}?w=${w}&h=${h}&fit=crop&auto=format&q=75`;
```

| Object | Purpose | Size |
|--------|---------|------|
| `POOL[12]` | Splash mosaic + paywall collage/mosaic + home bg | 300×600 |
| `PV_PHOTOS` | Preview slide fullscreen photos per effect | 393×700 |
| `HOME_PHOTOS` | Home screen big card photo per effect | 400×600 |
| `EFF_PHOTOS` | Effect picker row thumbnails per effect | 112×132 |

Effects covered in all photo objects: `kisses`, `dances`, `hugs`, `tiktok`, `aivid`, `t2v`, `none`

---

## Preview Slide Data

Each effect has 3 slides in `PV` object:
```js
PV = {
  kisses: [ { sc, body, head, hair, gc, str, badge, title, desc, decos:[] }, ... ],
  dances: [ ... ],
  hugs:   [ ... ],
  tiktok: [ ... ],
  aivid:  [ ... ],
  t2v:    [ ... ],
  none:   [ ... ],
}
```

Each deco:
```js
{ t:'deco-heart', c:'rgba(255,80,120,0.75)', x:'10%', y:'12%',
  fs:'26px', op:.5, d:'3s', dl:'0s' }
```

---

## How to Update & Deploy

Edit the file locally, then from Terminal:

```bash
cd /Users/d.ratsko/Downloads/tvorra-onboarding
git add onboarding/index.html
git commit -m "describe change"
GIT_SSH_COMMAND="ssh -i ~/.ssh/github_key -o StrictHostKeyChecking=no" git push origin main
```

Site updates at https://dmytroratsko.github.io/tvorra-prototyping/ within ~1 min.

SSH key is configured at `~/.ssh/github_key` (ed25519).

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
| Progress bar + skip removed | Removed all skip buttons and progress bar from preview |
| s-home screen added | Personalised feed card after paywall close/continue |
| Real photos added | `POOL`, `PV_PHOTOS`, `HOME_PHOTOS` via Unsplash CDN + `.pc-img` system |
| Flow updated | `s-success` removed from flow; paywall now → `s-home` |
| Effect picker redesigned | 4 cards → 7 scrollable rows with photo thumbnails |
| Effect thumb real photos | `buildEffectThumbs()` injects `.pc-img` into each `.eff-thumb` |
| Paywall collage fade | Gradient 150px → 260px multi-stop for smooth photo-to-content transition |
| PV_PHOTOS data gap | Added `dances/hugs/t2v/none` keys (were falling back to `kisses`) |
| _homeData / HOME_PHOTOS gap | Added `dances/hugs/t2v/none` entries |
| pw-mosaic real photos | `buildPwMosaic()` now uses real Unsplash photos |
