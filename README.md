# ALWAYS UP

An endless vertical climber built with **Babylon.js** that plays **inside a post on X
(Twitter)** — the "player card" pattern.

- **Play / share page:** https://agent-marsha.github.io/always-up/
- **Player page (the framed game):** https://agent-marsha.github.io/always-up/play/

## How the X embed works

A game can run inside an X post because X supports **player cards**. Post the URL of
`index.html`; X reads its `<head>`, shows a thumbnail with a play button, and loads the
`twitter:player` URL in a frame **inside the post**, so the game runs in the timeline.

`index.html` carries:

```html
<meta name="twitter:card"   content="player">
<meta name="twitter:title"  content="ALWAYS UP">
<meta name="twitter:image"  content="https://agent-marsha.github.io/always-up/preview.png">
<meta name="twitter:player" content="https://agent-marsha.github.io/always-up/play/">
<meta name="twitter:player:width"  content="480">
<meta name="twitter:player:height" content="600">
```

Rules that matter (learned from a working deployment):

1. **The tags must be in the server-sent HTML** — X does not run your JavaScript when it
   reads the card. `curl` the URL to confirm the tags are in the raw markup.
2. **Two pages.** A share page (card tags) and a bare player page (`play/`) that is *only*
   the game, filling the frame. No header, no footer beyond a slim way back.
3. **It must be framable.** GitHub Pages sends no `X-Frame-Options`, so `x.com` can iframe it.
4. **Show something within the first second.** A dark background + HUD render before the
   1.4 MB engine bundle finishes, so the frame never looks broken.
5. **X caches the card per URL.** If you change the tags or the player page, post a URL with
   something new appended (`?v=2`) or X will keep the old card.

Where it plays: on **x.com** the game runs inline in the post. The X **mobile app** may open
the player page in its own in-app browser instead. X decides which links get a player and can
change that at any time — there is no public "games API" for this; it rides the existing card
system.

## Game

- One action: **tap the left/right half** of the frame (or `←` / `→`, `A` / `D`).
- The climber bounces on its own; grey platforms are normal, **teal** pads launch you higher,
  lighter platforms slide side to side.
- **Score = altitude in metres.** Fall off the bottom and the run ends. Best height is kept
  in `localStorage` (wrapped in try/catch — it is blocked in sandboxed iframes).
- Idle before the first input: the climber bobs on the home platform and cannot die.
- `R` is not used; **tap anywhere after a run** to restart instantly.

## Build

Source is `src/game.js` (tree-shaken `@babylonjs/core` ES modules). Rebuild the bundle:

```bash
cd /root/always-up
npm --prefix _build install            # first time only
./_build/node_modules/.bin/esbuild src/game.js \
  --bundle --minify --format=iife --target=es2019 --outfile=play/game.js
```

Output: ~1.4 MB raw / **~357 KB gzipped** (vs ~8.7 MB / 1.9 MB gz for the full UMD build).

## Layout

```
index.html        share page — the URL you post (card tags live here)
preview.png       1200x1200 card thumbnail
play/index.html   bare player page (the game fills the frame)
play/game.js      built bundle (Babylon.js + game)
src/game.js       source
_build/           esbuild + @babylonjs/core (build-time only, not published)
```

---

A Human–Technology Interaction Lab (HTIL) prototype, Dept. of Computer Science,
The University of Alabama.
