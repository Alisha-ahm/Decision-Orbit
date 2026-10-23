# 🪐 Decision Orbit

A single-file, dependency-free decision-making tool. Describe a problem, add
options as orbiting planets, score them on 5 criteria, and hit **Solve** to see
which option gravitates closest to the sun — with a particle-burst celebration
and a result card explaining why it won.

Everything (HTML + CSS + vanilla JS) lives in **`index.html`**. No build step,
no libraries, no backend — open the file directly in any browser.

## Features
- **Problem input** → updates the glowing central "sun" label live.
- **Add option** → creates an orbiting planet with the option's name.
- **Click a planet** → slide-in panel with 5 sliders (1–10): Cost, Effort,
  Time, Benefit, Risk.
- **Scoring formula**: `raw = Benefit×2 − (Cost + Effort + Time + Risk)`,
  normalized from its theoretical [-38, 16] range to **0–100**.
  Benefit is the only positive driver; Cost, Effort, Time and Risk all
  subtract. Higher score → smaller orbit radius (closer to the sun) and a
  brighter glow.
- **Solve button** → pauses all orbits, tweens the winning planet inward with
  a strong sun pulse and a 30-particle confetti burst, then slides in a
  result card: *"Best choice: X — Score N/100"* plus an auto-generated reason
  based on which criteria stood out.
- **Delete option** (🗑 inside the panel, or the × badge on hover) and a
  **Reset** button that clears everything (with confirmation).
- **Mood Mode** — Calm 🌙 / Focus 🎯 / Energy ⚡ — swaps the neon palette and
  the orbital speed multiplier (Calm = slow & soft, Energy = fast & vivid),
  persisted alongside your session.
- **Persistence** — problem, options, scores and mood are saved to
  `localStorage` (`decisionOrbit.v1`) and restored automatically on refresh.
- Fully responsive: side panel becomes a bottom sheet on narrow screens,
  orbit stage and header reflow for mobile.

## How the logic works
**Scoring** (`computeScore`): each option stores five 1–10 slider values.
`raw = benefit*2 - (cost + effort + time + risk)`. The best possible raw value
is `2*10 - (1+1+1+1) = 16`; the worst is `2*1 - (10+10+10+10) = -38`. We
linearly map `[-38, 16] → [0, 100]` and round, so the score is always a clean
0–100 number regardless of slider combination.

**Orbit math** (`scoreToRadius`): the stage computes a `minR`/`maxR` band from
its own size. `radius = maxR - (score/100) * (maxR - minR)` — a 100-score
option sits at `minR` (tight orbit, closest to the sun), a 0-score option
sits at `maxR` (far drift). Each planet keeps its own angle + speed in a
`runtime` map and advances `angle += speed * dt * moodSpeedMult` every
animation frame (`requestAnimationFrame`), so orbits are smooth and
independent. Glow (box-shadow blur + color) and planet size also scale with
score. On **Solve**, each planet's radius is temporarily overridden and
tweened with an eased animation: the winner tweens to a tiny radius near the
sun, everyone else tweens slightly outward and dims; closing the result card
tweens every radius back to its natural score-based value.

## Project structure
```
index.html   ← everything: HTML structure, CSS (with :root variables +
                per-mood variable overrides), and commented vanilla JS
                (state/persistence, scoring, orbit math, rendering,
                animation loop, solve/particles, panel, mood, events).
README.md    ← this file
```

## Running locally
Just open `index.html` in any modern browser — no server, build step, or
install required.

## Deployment
This repo is a static site, so it can be hosted anywhere that serves static
files as-is (GitHub Pages, Vercel, Netlify, Cloudflare Pages, etc.) with zero
configuration — just point the host at `index.html`.

### Deploy to Vercel
1. Push this repo to GitHub (see below).
2. Go to https://vercel.com/new, choose **Import Git Repository**, and select
   this repo.
3. Framework preset: **Other** (static). Build command: *(leave empty)*.
   Output directory: *(leave empty / root)*.
4. Click **Deploy** — Vercel serves `index.html` at the generated
   `*.vercel.app` URL immediately, no config file required.

Alternatively with the Vercel CLI from the project folder:
```bash
npm i -g vercel
vercel --prod
```

### Push to GitHub (`Alisha-ahm/Decision-Orbit`)
```bash
git init
git add index.html README.md
git commit -m "Decision Orbit: single-file orbit-based decision tool"
git branch -M main
git remote add origin https://github.com/Alisha-ahm/Decision-Orbit.git
git push -u origin main
```

> Note: this assistant's tools can write files and push to *this* project's
> own linked Git-backed preview/Hosted pipeline, but they have no credentials
> for third-party accounts — they cannot authenticate to your personal GitHub
> account or your Vercel account to push/deploy on your behalf. Use the
> commands above (or GitHub's "upload files" web UI + Vercel's "Import Git
> Repository" flow) to finish those two deployments yourself; both take under
> a minute once the files are in hand.

## Data model (localStorage key `decisionOrbit.v1`)
```json
{
  "problem": "Which laptop should I buy?",
  "mood": "focus",
  "options": [
    { "id": "id-...", "name": "MacBook Air", "cost": 7, "effort": 3,
      "time": 4, "benefit": 8, "risk": 2 }
  ]
}
```

## Possible next steps
- Export/import a session as JSON for sharing.
- Weighted-criteria mode (let the user set importance per criterion).
- Multiple saved decisions with a switcher.
