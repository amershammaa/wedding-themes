# ZAFFA THEME FACTORY — the repeatable pipeline

The exact workflow used to ship beach, voyage-2, qasr-2, jardin-2, noir-2. One theme ≈ 1–2 hours end-to-end. Repo: `github.com/amershammaa/wedding-themes` (GitHub Pages).

---

## 0. The recipe card (decide these 5 things first)

| Slot | Example (noir) |
|---|---|
| **World** | Art-Deco midnight ballroom |
| **Palette** (9 CSS vars: em, em-deep, paper, card, ink, gold, gold-soft, bronze, lines) | champagne on black |
| **Type pair** (display + body; AR pair stays El Messiri + Almarai) | Poiret One + Cormorant |
| **Gate** (always: full-bleed "doors" art that swings open in 3D) | deco theatre doors |
| **Video moment** (always: sketch/etching → painted/illuminated life) | linework ignites, chandelier glitters |

**Content is FIXED** (the eventocards set): cover → looping video hero with full invitation → countdown → celebration + map → hotels + map links → Liste de Mariage with copy-IBAN → tailored per-guest RSVP (`?gid=`) + wishes + seal/heart → footer. Never redesign the content, only the skin + signature.

## 1. Base file

Copy the newest proven theme (currently `qasr-2/index.html` — it has the full engine: i18n AR/EN, door gate, video loop w/ paper-wash, reveals, motes, magnetic buttons, copy toast, chips, watchdog, fallbacks). Transform with find/replace only:
- palette = swap the 9 `:root` var VALUES (selectors never change)
- default `lang`, `<title>`, couple names (both dicts + static HTML), venue/city, bank beneficiary, localStorage key
- display font: add Google Font + override the 3 `html[dir=ltr]` display rules
- dark themes: `mix-blend-mode:multiply` → `screen` on art

## 2. Art — kie.ai gpt-image-2 (free, ~60s/batch)

API: `POST api.kie.ai/api/v1/jobs/createTask` `{model:"gpt-image-2-text-to-image", input:{prompt, aspect_ratio}}`, poll `/jobs/recordInfo?taskId=`.
4 images per theme, ONE style block reused in every prompt:
1. **doors** (2:3) — "tall closed [gate concept] filling the frame edge to edge … light glowing through the center seam"
2. **q_vid_start** (9:16) — the world "almost entirely UNPAINTED — pure line art, scene in the lower two thirds, top third bare"
3. **q_court** (16:9) — the world, partly washed, for the Celebration section
4. **q_couple** (1:1) — couple in the world's register, continuous-line sketch

Style block template: *"Fine [ink] line work with [palette] washes on [paper], [aesthetic register] wedding stationery style, refined, generous negative space, no text, no watermark, no logos, no border."*

## 3. Video — runway on kie (~8 credits, 5s, 720p 9:16)

Upload start frame to tmpfiles.org (needs a long filename), then
`POST api.kie.ai/api/v1/runway/generate {prompt, duration:5, quality:"720p", aspectRatio:"9:16", imageUrl}` → poll `/api/v1/runway/record-detail?taskId=`.
Prompt formula: *"The [sketch/etching] slowly comes to life: [washes bloom / lines illuminate], [3 specific magic details], camera locked off, no camera movement, gentle painterly transformation, [register] aesthetic."*
The page loops it with the paper-wash crossfade automatically.

## 4. Optimize (NEVER SKIP — the 3G rule)

- Images → WebP, max 960px wide, q78 (PIL: `save(webp, quality=78, method=6)`). 2.5MB → 50–150KB.
- Video → `ffmpeg -crf 28 -preset medium -vf scale=720:-2 -movflags +faststart -an`. ~5MB → ~1–1.6MB.
- `loading="lazy" decoding="async"` on below-fold art; `<link rel="preload" as="image">` on the doors; `preload="metadata"` on video.
- Budget: whole page ≤ 2.5MB.

## 5. Verify (checklist)

`node --check` the extracted inline script · open via headless: tap gate opens, guests personalized, hotels/ledger built, no horizontal overflow at 320px, video loops · file ends with `</html>` (OneDrive truncates — always work in /tmp, verify tail before commit).

## 6. Ship

Work in a local clone of the repo (never stage through OneDrive). `git add/commit/push` → GitHub Pages builds (~2–10 min; if a big-media build errors, push an empty commit to retrigger). Add the theme card to root `index.html`. Live at `amershammaa.github.io/wedding-themes/<slug>/?gid=<guestId>`.


## 7. Music (shared across all themes)

- One shared track at repo root: `music.m4a` — referenced by every theme as `../music.m4a`.
- Wiring (already in every theme): `<audio id="bgm" loop preload="none">` + round `.bgmbtn` toggle. Playback starts on the gate tap (the user gesture browsers require); `preload="none"` = zero cost until then.
- **Generating a new song:** use Suno / Higgsfield `generate_audio` — prompt formula: *"instrumental [oud & strings / piano & strings / oriental orchestral] wedding processional, warm, elegant, gentle build, loopable, no vocals, 30-60 seconds"*. Then normalize + fade + compress:
  `ffmpeg -i song.wav -af "afade=t=in:d=1.5,afade=t=out:st=<len-2.5>:d=2.5,loudnorm" -c:a aac -b:a 96k music.m4a`  (target ≤ 500KB)
- Per-theme songs: put `music.m4a` inside the theme's `art/` and change the src to `art/music.m4a`.

## 8. Performance targets (updated)

- Video: `-crf 30 -vf scale=540:-2 -an +faststart` → 330–710KB per 5s clip.
- Idle buffering: on `window load` (+800ms) the page flips the video to `preload="auto"` — it buffers silently while the guest reads the cover, so tap→play is instant even on 3G.
- Page budget: ≤ 1.5MB before video, video ≤ 700KB, music ≤ 500KB (loads only after tap).

## Taste rules (client-locked)
- **Avoid gold** as a UI/palette accent — client dislikes it. Prefer ivory, champagne-cream (only when the ART demands it, e.g. deco), sage, powder blue, blush, emerald, terracotta. Never flat gold text/buttons.
- Demo guests are always neutral placeholders (Sami & Lina Haddad) — never real people's names.
- No thick ornate Arabic display (Aref Ruqaa banned) — El Messiri or refined Naskh only.

## Hard-won rules (do not relearn these)
1. **No Lenis / no scroll-hijack libraries.** Native scroll only. (Broke the site three ways.)
2. **OneDrive lies** — sandbox reads/writes of big files truncate mid-sync. Build in /tmp, verify `</html>` tail + node --check before any commit.
3. gsap `.set` of a pre-open CSS state gets baked into scrub tweens — hide hero children, not the container.
4. `inset:0` + `left:50%` halves a fixed gate's width — use `top/bottom + left:50% + width:100%`.
5. Video with baked-in intro? Skip to the good frame or let the DOM do the reveal — never both.
6. cdnjs doesn't host everything (no lenis); check before referencing. `async` any non-critical script so a dead CDN can never block boot.
7. Probing "success-shaped" API payloads can create real (paid) jobs — probe with invalid models only.
8. kie CDN result URLs expire in hours — download immediately.
9. Every open() needs the 3.5s watchdog; every theme needs the no-GSAP + reduced-motion fallbacks.
