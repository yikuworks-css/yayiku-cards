# Yayiku Cards

A drag-and-drop card board. Make cards, drag one onto another, and they fuse into a
bundle — a single container holding both. Cards you leave alone stay alone.

**Live:** https://yikuworks-css.github.io/yayiku-cards/

No build step, no dependencies, no server, no account. One HTML file. Open it and it runs.

---

## What it does

- **Create cards** with a title and a description
- **Drag a card onto another card** → they become a bundle, auto-named from their initials (`A` + `B` = `AB`)
- **Drag a card onto a bundle** → it joins that bundle
- **Drag a bundle onto a bundle** → they merge into one flat bundle
- **Drag a card out to empty space** → it leaves the bundle and goes free
- **Bundles dissolve automatically** when only one card is left
- **Collapse** a bundle to a compact stack, **rename** it, or **unpack** it entirely
- **Export / Import** the whole board as a JSON file
- Everything **auto-saves** to `localStorage`

## Running it

Double-click `index.html`. That's it.

Or serve it, if you prefer:

```bash
python3 -m http.server 4173
```

Then open <http://localhost:4173>.

---

## The interesting part: how the drag feels

This project exists mainly to explore drag-and-drop, so most of the effort went into
motion rather than features.

**Pointer Events, not HTML5 drag-and-drop.** The native DnD API gives you no control
over the drag image, fires inconsistently across browsers, and doesn't work on touch.
Everything here is built on `pointerdown` / `pointermove` / `pointerup`, which behaves
identically for mouse, trackpad, and finger.

**A 5px threshold before a drag starts.** Without it, every click registers as a
micro-drag and buttons inside cards become impossible to press.

**The card tilts with your velocity.** Horizontal pointer speed is smoothed with a
running average (`vx = vx * 0.78 + dx * 0.22`) and mapped to a rotation clamped to ±9°.
This is the single detail that makes a dragged card feel like an object with weight
rather than a rectangle following the cursor.

**A placeholder holds the gap.** When a card lifts, it switches to `position: fixed` and
a dashed placeholder of identical size takes its place, so the layout doesn't collapse
and snap back underneath you.

**FLIP for every layout change.** Before each re-render, every element's rect is
recorded; after the re-render, the difference is applied as a transform and animated
back to zero. Cards slide to new positions instead of teleporting — including the card
you just dropped, which flies from where you released it into its final slot.

**Drop targets announce themselves.** Hovering a valid target outlines it, pulses it,
and shows a floating label saying exactly what will happen — `Bundle with A`,
`Add to ABC`. You should never have to guess what releasing the mouse will do.

`prefers-reduced-motion` disables all of it.

## Data model

Deliberately flat — bundles never nest inside bundles:

```js
Card   { id, kind: 'card',   title, desc }
Bundle { id, kind: 'bundle', name, collapsed, renamed, children: [Card] }
```

Board state is one ordered array of these, in `localStorage` under `yayiku-cards-v1`.

## Known limits

- State lives in one browser on one machine. Clearing site data wipes it — use **Export**
  to keep a real backup.
- Bundles are one level deep by design. Dragging a bundle onto a bundle flattens them.
- No sync and no accounts. Firebase may come later; the state layer is deliberately small
  and isolated so swapping the persistence backend is a contained change.

## License

MIT — see [LICENSE](LICENSE).
