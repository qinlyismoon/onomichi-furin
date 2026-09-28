# AGENTS.md — Onomichi Furin Wall (尾道 · 風鈴)

An interactive web wish wall of glass wind chimes (風鈴 / furin). Visitors swipe or blow to make the chimes swing and ring, read the wishes on the paper strips, and hang a wish of their own.

Read this file before changing anything. It records intent that the code alone does not show.

## Origin

Inspired by the wind-chime wish wall at **Senkō-ji (千光寺)** in Onomichi, Hiroshima, Japan, during its **Fukurin Matsuri (福鈴まつり)**. Visitors write a wish on a paper strip (短冊 / tanzaku), tie it under a glass chime, and hang it on bamboo poles overlooking the Onomichi Channel. See `reference/senkoji.png` for the photo that started the project. Use it as the visual anchor whenever you tune the scene.

## Project shape

- **Single file.** Everything lives in `index.html`: markup, CSS, and one inline script. No build step, no npm dependencies, no asset files.
- Fonts load from Google Fonts: `Ma Shan Zheng` (Chinese brush), `Yuji Syuku` (Japanese brush), `Noto Sans SC` (UI). Every font has a fallback stack.
- Sound is synthesized live with the Web Audio API. There are no audio files.
- Run locally: `python3 -m http.server 5173`, then open `http://localhost:5173`. The microphone only works in a secure context (localhost or https), not from `file://`.

Keep it a single file unless the user asks otherwise. If you split it, keep a zero-build setup that opens with a static server.

## Design principles (do not drift from these)

The visual direction is **Mondrian's color + Wu Guanzhong's line**. The colors should feel **warm, diffuse, and liquid**.

1. **Color carries warmth; line carries structure.** Keep the two in separate layers.
   - *Color layer* (`#bg` canvas): Mondrian-style rectangles, rendered at very low resolution (160px wide), then CSS-blurred and slightly oversized. Block edges breathe slowly with noise. This layer is never crisp.
   - *Line layer* (`#main` canvas): ink strokes with varying width, tapered ends, and dry-brush bristle gaps ("flying white"). Bamboo poles, ridge lines, shoreline, chime strings, and posts are all drawn this way. This layer is never blurred.
2. **The color blocks are an abstracted Onomichi view**, from top to bottom: sky with a yellow and a red block, grey-blue hills, the blue channel, a town strip of small blocks, and the dark tiled wall in the foreground. Preserve that reading.
3. **Palette** (primaries shifted warm):
   - vermilion `#D9462E`
   - yellow `#F0B23A`
   - ultramarine `#2D5CA4`
   - warm ground `#EFE3CD`
   - paper `#F8F2E7`
   - ink `#221D19`
   - wall `#3B2F29`

   Do not add new hues to the UI or to the chimes. Neutrals lean warm. The one exception is the time-of-day grade (see "Time and atmosphere"): it may tint the environment with apricot, coral, misty blue, or muted indigo, but only on environment layers.
4. **Wu Guanzhong dots.** Small colored dots are scattered through the town and hills. The glass bells carry brush arcs or dots.
5. **UI is quiet.** Use square corners (2px radius), 1.5px ink borders, and paper backgrounds. The logo mark is a tiny Mondrian grid (red / yellow over paper / blue, columns 3:2, rows 2:3); the favicon is the same mark as an inline SVG. The only filled accent is the red "Hang a wish" button, which sits in the header to the right of the title (it wraps below the title at phone width). Do not add emoji or gradients on UI chrome. The main page is the source of truth for UI geometry: angular, graphic, poster- and paper-like. Dialogs (`.panel`) are paper sheets in the same system, not rounded app cards. Corner radii step down from the outside in:
   - Sheet: 0px, with one small cut corner at the top left (`--cut`, 14px), drawn with `clip-path`. The border is an inset line plus a thin diagonal on the cut. The shadow is a soft `drop-shadow` that follows the cut.
   - Dialog buttons: 0px. Hang it stays the red primary; Cancel stays paper with an ink border.
   - Glass color cards: 5px.
   - Text inputs: 3px.
   
   Never use large radii (12px+), pill buttons, or heavy shadows. A faint seigaiha watermark sits in the lower-left of the sheet. The glass color picker keeps its illustrated bells, drawn by the same `paintBell()` as the wall with seigaiha etched into the lower glass, on a neutral paper-tinted card. The selected card gets a thin dark border and a faint tint in its own glass color, never a double outline or strong shadow.
6. **Language.** Wish text on the strips may be Chinese, Japanese, or English. All UI copy (CTAs, labels, hints, toasts) is English. Place and festival names stay in the original Japanese: 尾道, 風鈴, 千光寺, 福鈴まつり. The brush glyph 風 on the mic button is Japanese for the same reason. Project docs (`AGENTS.md`, `README.md`) are written in English too, using Japanese only for those names.
7. **Depth.** There are three tiers of bamboo poles, back to front (`TIERS`). Back tiers are smaller, lighter, and quieter. They are drawn back to front.
8. **Four layers, one place.** Wish = traces left by people. Wind = movement and interaction. Time = the changing atmosphere of the place. Cats = subtle signs of life. New features should deepen the sense of place, not add decoration.

## Time and atmosphere

The wall follows the local time in Onomichi (`Asia/Tokyo`, never the device timezone). The four phases are the same physical place under different light, not four themes.

- **Logic** (`timeState`, `onomichiNow`, `sunTimes`, `phaseStarts`): returns the current phase and blend weights. Phases cross-fade over `PHASE_FADE` (40 min). `sunTimes()` currently returns a fixed sunrise 06:00 and sunset 18:00; replace it with a solar calculation for Onomichi (34.41°N, 133.20°E) to follow the seasons. Nothing else needs to change.
  - 朝 Morning: sunrise − 45 min → sunrise + 150 min
  - 昼 Day: → sunset − 90 min
  - 夕 Sunset: → sunset + 45 min
  - 夜 Night: → next morning
- **Styling** (`ATMOS`, `blendAtmos`): one parameter set per phase (sky gradient, multiply grade, sun or moon glow, water glint, town lights, chime glow, glass sheen direction, particles, cat shadow). Every phase must define the same keys, because they are blended numerically. Day is the baseline and leaves the original scene untouched.
- **What it may change:** the `#bg` canvas grade, glow behind bells, a light sheen on the glass, atmospheric motes, cat shadows. **What it must not change:** UI chrome, paper strips, text, and the red / yellow / blue / clear glass identity.
- `updateTime()` runs every 15 s and also updates the clock label next to Sound on. The label is one soft box holding two parts, `{kanji}` and `{English phase} · {HH:MM} · 尾道`, e.g. `夜` | `Night · 02:45 · 尾道`; phase and time always follow Onomichi local time. The kanji comes first, in the brush font at 22px like the 風 on the mic button, so the visitor reads the scene at a glance; the reading beside it is the plain-language explanation. Both parts share the same pale translucent background; only a short hairline divider separates them. **It must look different from the buttons**, because it is information, not a control: no solid ink border, no solid paper fill, `pointer-events:none`. It has the same 44px height as the Sound on button, but a soft-edged box looks shorter than a bordered one, so it uses a 1px hairline edge and overshoots 1px top and bottom (46px, `margin:-1px 0`). Keep that optical compensation if you change the border or height. At phone width it moves to the bottom-right, on the same row as the mic button, because there is no room under Sound on.
- **Preview:** add `?time=HH:MM` to the URL to force a local time, e.g. `?time=18:42`.

## Cats

Cats are ambient inhabitants of the scene, not mascots (`CAT_SCENES`, `drawCat`). They are ink silhouettes drawn on `#main` right after the landscape ink and before the chimes, so chimes and strips always sit on top. They live on the foreground wall, below the strips. They are never hit-tested, so they cannot block swipes, taps, or controls.

- At most two cats per phase: morning (sitting and stretching by the post, plus one peeking over the wall), day (sleeping plus sitting), sunset (one walking slowly along the wall plus sitting), night (a quiet silhouette with faint warm eyes plus one peeking).
- Motion is minimal: tail sway, breathing, an occasional stretch, blinking, slow walking. `prefers-reduced-motion` stops all of it.
- A new phase fades its cats in over about 1.6 s.

## Physics and sound (the core mechanic)

Each chime is a **coupled double pendulum**:

- `a1/v1` is the string + glass bell.
- `a2/v2` is the clapper + paper strip.

The mechanism works like this:

- **Wind pushes mainly the strip**, which has the larger area (`F*1.6` on the strip vs `F*0.35` on the bell).
- The strip drags the clapper.
- **Sound happens only when the clapper hits the bell rim.** The relative angle `a2 - a1` is clamped to `±MAXREL`. When the clapper hits the rim with closing velocity, `strike()` fires with that velocity as the impact.

This matches how a real furin rings. Its rhythm is emergent, not scheduled. **Never trigger sound on a timer or directly from input, bypassing this model.** The one exception is a direct flick of the glass: a tap on a bell, or a swipe through one.

Other details:

- The integrator is semi-implicit Euler at a fixed 120 Hz step (`step(h)`), with at most 8 substeps per frame.
- **Wind** comes from `windAt(x, t)`. It combines a gentle ambient breeze (so the wall is never still) with a `blow` level from 0 to 1. The blow level comes only from the mic. There is no hold-to-blow button or keyboard shortcut; without a mic, visitors swipe.
- **Mic blow detection** uses an FFT (2048 bins). The level is weighted low-band energy (<700 Hz) plus mid-band energy (700–3500 Hz), minus an adaptive noise floor. The mic is never connected to the audio output.
- **Glass tone** in `ding()`:
  - Partials follow inharmonic ratios `[1, 1.0019, 2.32, 4.25, 6.63]` with exponential decays.
  - A stereo panner places each chime by x position.
  - A compressor sits on the master bus, plus a generated convolution reverb.
  - Partials above 15 kHz are skipped.
  - Voice count is capped.
- **Pitch** comes from a pentatonic-style `SCALE` over D6 (1174.66 Hz). Bigger bells get lower pitches.
- **Visual sound.** Each strike brightens the bell (`ring`) and emits a thin ink ripple circle.

## Interaction

| Input | Effect |
|---|---|
| Press and swipe on the canvas | Chimes whose strip or bell the swipe segment crosses get an impulse in the swipe direction. The swipe leaves a light "wind trace" (see below). |
| Tap a strip | Opens the wish card and reads the text. |
| Tap a bell | Flicks the bell and rings it. |
| **風 Blow into mic** toggle | Live mic blowing. The 風 glyph leans with the blow level while the mic is on. If the mic is unavailable, a toast points the visitor to swiping. |
| **Hang a wish** | Opens the form: wish text (max 24 chars), glass color (red / yellow / blue / clear), optional name (max 12 chars), and a quiet `Share this wish with visitors` checkbox (default from `WISH_CONFIG.shareByDefault`). The new chime drops onto the front pole with an ease-out-back animation, then rings. |
| Sound on / off | Mute toggle. |

Audio unlocks on the first user gesture. Browsers require this.

## Pointer feedback: the wind trace

The pointer's visual feedback is purely visual and separate from the physics. The pointer handlers call `windOrigin`, `windDirection`, and `windDissipate` right next to `sweep()` / `hitTest()`, but nothing in the trace feeds back into wind, chimes, or sound. It follows three states:

- **Origin (pointer down):** a soft glow plus two thin rings that expand and vanish in under a second. No dot or center marker. It should read as "wind begins here", not "you clicked here".
- **Direction (drag):** small translucent dots scattered loosely around the path, at irregular spacing, with varied size and opacity and a faint halo on each. They pick up a little of the drag velocity, then drift slowly upward and sideways. Two very thin curved wind lines sit slightly offset on either side of the path. They are secondary to the dots.
- **Dissipation (pointer up):** no new dots. Existing dots keep drifting and fade out over 1–2 s, each with its own lifetime. The wind lines widen, soften, and fade.

Color rules: never use black or dark grey. The trace samples the `#bg` canvas under the pointer every 70 ms. On darker areas (blue channel, sunset, night) it uses ivory and cream, with an occasional pale coral or pale blue dot. On pale cream and yellow areas it blends toward muted warm sand and coral so it stays faintly visible. Keep opacity low; it must stay secondary to the chimes and strips. It should read as air moving, never as sparkles, stars, confetti, or a neon cursor trail.

## Rendering notes

- Bells and strips are pre-rendered **sprites** (`makeBell` → `paintBell`, `makeStrip`). They are rebuilt on resize and after fonts load. Do not draw gradients and shadows per frame.
- Strip text is vertical. CJK text is set one character per row, and falls to two columns (right to left) when long. Latin-only text is rotated 90°. Text that contains kana uses the Japanese font first.
- Wishes added by visitors get warm paper and a **mini Mondrian block at the strip's foot**, so they stand out from the pre-hung ones.
- `U` is the global size unit, derived from the viewport. All sizes scale by `U × tier scale`.
- `prefers-reduced-motion` stills the background, softens the ambient wind, freezes particles and cats, and stops the swatch sway.

## Data

The wall is one place fed by three conceptual layers, but there is **one unified wish record** and the layers are derived by filtering. Never build separate walls, tabs, sections, or color coding for seed / community / mine.

```
{ id, text, color: red|yellow|blue|clear, name: string|null, source: 'seed'|'user',
  visibility: 'shared'|'private', ownerId: string|null, createdAt: ISO string, nx?: 0..1 }
```

- **Seed layer:** `SEED_WISHES`, the 55 pre-hung wishes (`seed-01`…`seed-55`), `source:'seed'`, `visibility:'shared'`, `ownerId:null`. They are generated once from `mulberry32(20260928)` with the same draw order as the old on-screen generator, so every seed keeps the exact text and glass color it always had. `buildBase()` keeps drawing from that same stream for layout only.
- **Community layer:** wishes visitors chose to share (`visibility:'shared'`).
- **Personal layer:** the current visitor's wishes, shared and private (`ownerId === Identity.currentId()`).
- `nx` is only a placement hint so a chime does not move when its visibility changes.

**Visibility rule** (in `WishRepository.visible()`): `source === 'seed' || visibility === 'shared' || ownerId === me`. It is applied in the data layer. The renderer only ever receives allowed wishes, and another visitor's private wish never reaches it.

**Architecture:** `UI → WishRepository → stores → renderer`. The chime system does not know or care whether a wish is seed / shared / private.

- `Identity`: anonymous persistent visitor id (UUID in `localStorage`, key `onomichi-furin-visitor`). Never shown in the UI. Replace this object with real auth later.
- `WishRepository`: `subscribe(fn)`, `create({text,color,name,share,nx})`, `setVisibility(id, vis)` (owner only), `synced()`, `attachCommunity(store)`, `detachCommunity()`. The rest of the page never touches `localStorage` directly.
- Stores: `makeLocalStore` is used for **private** wishes (`onomichi-furin-private-v1`, never leaves the device) and for a **mock community** (`onomichi-furin-community-mock-v1`, `synced:false`: NOT synchronized between visitors). `makeDbCommunity` runs on the claude.ai artifact db (`window.claude.use("db")`, collection `wishes`, real sync) and replaces the mock when the db is present. Private wishes are never written to the shared db. Db docs store a numeric `createdAt` so ordering stays consistent with older docs; `normalizeWish()` accepts the old `{tint, numeric createdAt}` shape and treats old docs as shared and ownerless.
- The mock never pretends to sync. When `synced()` is false, the toast after sharing says sharing is not live yet. To go online later, add another store with the same `{list, put, remove, onChange, synced}` shape (e.g. Supabase or Firebase) and `attachCommunity()` it.
- `WISH_CONFIG.shareByDefault` (currently `true`) sets the default of the Share checkbox. Limits also live there.
- Legacy: wishes from the old local-only version (`onomichi-furin-wishes`) are migrated once into the mock community as this visitor's shared wishes.

Changing visibility updates the same record (same `id`, `text`, `color`, `createdAt`); only `visibility` and the store holding it change.

**Wall vs. wish record:** the wall shows what the viewport has room for. Seeds fill the poles in order, so at phone width only about 35 of the 55 seeds are hung. The `#count` label is the number of chimes actually hung (`{n} chimes`, no breakdown).

**Ownership marker:** wishes owned by the current visitor, shared or private, get a tiny vermilion knot with two short tails at the string hole of the strip (`makeStrip`, `ch.mine`). It is the only visual sign of ownership; do not add badges, avatars, or recolor the chime. Privacy is not visible from the wall.

**Wish card** by case: seed → text and `{Color} chime · hung earlier` (no system wording); another visitor's shared wish → text and `— Name` or `Shared by a visitor`, plus the date; my shared wish → `Your wish` / `Shared with visitors` and a quiet `Keep private` link; my private wish → `Your wish` / lock + `Only visible to you` and a `Share with visitors` link. Never show `ownerId` or any technical metadata.

Visitor-hung wishes keep the warm paper and the mini Mondrian foot; seeds keep the grey-green paper. That difference predates the data layers and is not an ownership cue.

## Code map (`index.html` script, in order)

1. **helpers** — clamp, noise `nz`, PRNG, color utils, palette, `TIERS`, `BLOCKS`, `TOWN_LIGHTS`, `DROPS`
2. **brush strokes** — `ribbon`, `brush`, `beamY`, `buildInk` (static landscape ink), `buildBeams`
3. **sprites** — `seigaiha`, `makeBell`, `paintBell`, `drawStripText`, `makeStrip`
4. **chimes** — `layoutChime`, `baseCounts`, `buildBase`, `makeUserChime`, `regroup` (the seed records, `SEED_WISHES`, sit with the constants at the top)
5. **audio** — `unlockAudio`, `ding`, `strike`
6. **wind & blowing** — `windAt`, `updateBlow`
7. **time of day** (logic) — `onomichiNow`, `sunTimes`, `PHASES`, `phaseStarts`, `timeState`
8. **atmosphere** (styling) — `ATMOS`, `blendAtmos`, `makeGlow`, `updateTime`
9. **particles** — `updateMotes`, `drawMotes`
10. **cats** — `CAT_SCENES`, `catSit` / `catSleep` / `catWalk`, `catShadow`, `drawCat`, `drawCats`
11. **physics** — `step`
12. **render** — `drawBG` (with the time grade), `drawChime` (with glow and sheen), `render`
13. **resize**
14. **wind trace** (pointer visuals only) — `fxSample`, `windOrigin`, `windDirection`, `windDissipate`, `updateWindTrace`, `drawWave`, `drawWindTrace`
15. **pointer** — `sweep`, `hitTest`, tap vs drag
16. **UI** — mic, sound, wish card, add-wish form, `drawSwatches`, seigaiha CSS tile, `pickSlot`
17. **wish data layer** — `WISH_CONFIG`, `Store`, `Identity`, `normalizeWish`, stores, `WishRepository`, `applyWishes`, form submit, `connectCommunity`
18. **fonts & main loop**

## Constraints and gotchas

- In the claude.ai artifact viewer, `getUserMedia`, `alert/confirm/prompt`, and `window.print` are blocked. Keep the in-page fallbacks (toasts instead of dialogs; swiping when the mic is blocked).
- Google CJK fonts load in unicode-range slices. Canvas does not trigger those loads, so the code calls `document.fonts.load(font, text)` with the actual text and then rebuilds the strips. Do the same for any new text drawn on canvas.
- Keep the page working at phone width (~390px). Controls stay above the safe-area inset.
- Chime count scales with viewport width, capped at 55 base chimes (about 35 at phone width). Resize rebuilds the base chimes, which resets their motion. That is intentional.

## Ideas not yet built

- A real backend behind `WishRepository` for a public shared wall (the community store is currently a mock or the artifact db)
- Phone gyroscope: tilting or shaking the phone makes wind
- Seasonal sunrise and sunset: replace the fixed `sunTimes()` with a solar calculation
- A 15-second demo recording for the portfolio
