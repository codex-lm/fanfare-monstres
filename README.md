# La Fanfare des Monstres

A paper-cut rhythm game for browsers. Four little monsters — Boubou (tuba),
Pico (trumpet), Zaza (xylophone) and Momo (drum) — want to turn their chaotic
rehearsal into a real concert. **You are the conductor**: press each monster's
key exactly when its note lands on the stage pad, and the band plays.

Play it: open `index.html` — everything (game, art and sounds) is in that one
file. No network, no dependencies.

- 🎮 **Jouez** : ouvrez `index.html` — tout est dans ce fichier unique.
- 🎮 **Juega**: abre `index.html` — todo está en ese único archivo.

## Rules

- Notes fall down four lanes, one per monster/instrument.
- Press the matching key (or tap the lane) when the note reaches its pad:
  **Perfect** ±45 ms, **Great** ±90 ms, **Good** ±135 ms.
- A chain of hits raises the multiplier (**×2 at 8, ×3 at 16, ×4 at 24**),
  wakes the audience and lights the stage. Misses break the chain — but the
  backing music keeps playing.
- The piece is a **success at 60% accuracy**; medals at 60% (bronze),
  75% (silver), 90% (gold).
- Three hand-written pieces with rising difficulty: *Sock Parade* (96 BPM
  march), *Cave Boogie* (112 BPM), *Monster Carnival* (132 BPM, chords).
- **Easy mode** slows every pattern to 85% speed and widens the windows
  (±70/±120/±170 ms); stray taps no longer break the chain (they briefly lock
  the lane instead).

## Controls

| Monster | Instrument | Key | Touch |
|---|---|---|---|
| Boubou | Tuba | `D` | tap lane 1 |
| Pico | Trumpet | `F` | tap lane 2 |
| Zaza | Xylophone | `J` | tap lane 3 |
| Momo | Drum | `K` | tap lane 4 |

`P` / `Esc` pause · `Enter` confirm · sound starts only after you click.

## Engineering notes

- **Audio-clock synced**: the song timeline is driven by
  `AudioContext.currentTime` (wall-clock fallback if audio is blocked);
  requestAnimationFrame only draws, so visuals cannot drift from the music.
- **Player timing offset**: adjustable (−150…+150 ms) with a built-in
  8-click calibration mode; the offset compensates judgement, exactly.
- **All sounds are synthesized locally** (WebAudio oscillators/noise) — the
  four instruments, the backing pad, percussion pulse, flubs and cheers.
  Muting keeps full visual feedback (bigger judgement popups, stronger lane
  pulses on the beat).
- **Trilingual** EN/FR/ES (English default), persisted.
- `?debug=1` shows FPS/timing, and `window.__fanfare` exposes a read-mostly
  QA hook (`charts`, engine functions, `debug.step/pressAt/state`).

## Tests

20/20 Node tests run the **real shipped `index.html`** in a VM: chart
validation (no same-lane collisions, one instrument per lane, progressive
density), difficulty scaling, exact scoring/multiplier/accuracy/medal math,
window boundaries (floating-point safe), late-hit consumption, Easy-mode
anti-mash locks, full-song perfect-play and jittered-play simulations, i18n
key parity, and package self-containment. See `DELIVERY.md` for the honest
full report (what was and wasn't tested in a real browser).

## Credits & license

- Concept and design specification: **Cédric Bernadac**.
- Implementation, art, synthesized audio and tests: **Z Code**
  (automated production with Z-Code; balance review consulted from
  **Kimi K3.1 Pro** — windows and Easy-mode anti-frustration rules).
- Code and assets: **MIT** (see `LICENSE`).
