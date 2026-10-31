# Delivery notes — La Fanfare des Monstres (v1.0.0, 31 October 2026)

One-off mission for the Z Code workshop (spec supplied by Cédric): a
four-lane rhythm game where the player turns a monster rehearsal into a
concert. This repository is the deliverable; publication on LOB7 follows the
usual checks and quotas under the Z Code account.

## Design decisions (interpretations of the spec)

- **The player IS the band**: instrument notes sound on successful hits
  (pitch taken from the chart); a soft pad + quiet beat pulse plays
  automatically so *the music continues after mistakes* — only your part
  drops out, plus a short flub on real errors.
- **Easy mode** (spec: "slows the patterns or widens the windows") does
  **both**: the whole chart runs at 85% speed (times scaled exactly,
  t_easy = t_normal / 0.85) and windows widen from ±45/±90/±135 ms to
  ±70/±120/±170 ms (values from a Kimi K3.1 Pro balance consult,
  31/10/2026, saved in the workshop).
- **Errors** (misses, late hits) break the chain and reset the multiplier;
  the music never stops. Stray taps near an upcoming note break the chain in
  Normal; per the same Kimi review, in Easy they **do not** break the chain
  and instead lock the lane 0.30 s (anti-mash; a genuinely missed note still
  breaks it). Taps far from any note are free warm-up (no penalty).
- **Late-by-a-hair** presses (between good and 2×good after the note)
  consume that note as a single miss — never a double penalty.
- Pass threshold **60% accuracy** is announced on the song-select screen;
  medals 60/75/90 (bronze/silver/gold). Accuracy is weighted
  (Perfect 100% / Great 70% / Good 40%).
- Audio-clock sync: the song time is `AudioContext.currentTime − startT`;
  rAF only draws. A wall-clock fallback covers autoplay-blocked contexts, and
  `ctx.suspend()` freezes time during pause, so no audio-visual drift can
  accumulate. The player offset (−150…+150 ms, 8-click calibration)
  compensates judgement exactly (verified by a dedicated test).

## What was actually tested

**Node tests — 20/20 passing against the real shipped `index.html`** (run in
a VM; `games/fanfare-monstres-dev/tests/` in the workshop):

- charts: 3 pieces, tempo/density ordering (1.52 < 2.42 < 3.28 notes/s),
  no same-lane note closer than 2× the good window, exactly one intended
  chord in pieces 1–2 (the finale), ≥4 in piece 3, all four instruments
  present, percussion lane, playable frequencies, backing within bounds;
- Easy mode: same notes, exact 1/0.85 scaling, wider windows, longer
  approach and duration;
- engine: window boundaries (Perfect/Great/Good at exact edges,
  floating-point-safe), nearest-note selection per lane, exact
  score = points × multiplier with steps 8/16/24 (cap ×4), misses break the
  chain while play continues, stray vs free taps, Easy nudge + 0.30 s lane
  lock, late-hit consumption (single penalty);
- results: weighted accuracy to the digit (0.69 case), pass at 60%,
  medals 90/75/60, perfect play ⇒ accuracy 1.0 / gold / pass on all three
  pieces in both difficulties, systematic-latency compensation test, and a
  jittered "average player" run that terminates deterministically;
- i18n key parity EN/FR/ES, self-contained package (no external loads),
  size.

**Real browser session (Chromium/IAB over http://127.0.0.1:8765), played
through the game's own input handlers:**

- home screen renders (paper/curtain/monster palette verified by pixel
  probes at expected geometry), song select shows the three pieces with
  note counts, durations, difficulty pills and the announced 60% threshold;
- **full playthrough of piece 1 (Normal)** with a page-side autopilot
  dispatching *real KeyboardEvents* at human-jittered times (±0–60 ms) plus
  3 deliberately skipped notes: final counts 45 Perfect / 16 Great / 3 Miss,
  accuracy 87.8% → **silver medal, pass**, max combo 23, score 9660 —
  matching the engine's math exactly;
- **full playthrough of piece 3 (Easy) muted** with presses offset +55 ms
  (compensated by the player offset setting) + a 4-press mash burst:
  110P/15G/2M, accuracy 94.9% → **gold**, combo 67, ×4 reached; the mash
  counted 1 stray **without breaking the chain** (combo trajectory shows the
  only reset came from the missed note) and follow-up presses hit the lane
  lock;
- countdown (3 audible clicks + big numerals), judgement popups, combo ×N
  and multiplier HUD, progress bar, results screen (counts, score,
  accuracy, medal, pass/fail banner), retry;
- pause (`P`) freezes song time, resume continues; settings: sound off
  (muted visual cues still pop), offset slider with live label,
  calibration screen counts taps (1/8 verified); language switch FR and ES
  verified on screen texts; mobile portrait 390×740 layout verified
  (monsters/lanes recomputed, no overflow);
- screenshots in the workshop's `evidence/fanfare-monstres/` (01 home,
  02 gameplay, 03 results, 04 muted-easy, 05 mobile-ES).

## Known limits (honest list)

- Menu clicks were driven via DOM `.click()` (the test pane's input routing
  is unreliable); in-game inputs were real `KeyboardEvent`s at precise
  times. Real mouse/finger presses on menus were not replayed this session.
- Touch lane taps (pointer events) share the same handler path as keys but
  were not replayed on a real touch device; no phone, no gamepad tested.
- Audio design was verified mechanically (context runs, scheduling
  lookahead, per-instrument synth calls) but not critically listened to
  end-to-end by a human ear; volume balance is a first guess.
- The IAB pane throttles rAF when unfocused; a watchdog interval advances
  the game state from the audio clock, and pause-on-blur protects the
  player — verified logically, not by pulling focus mid-song.
- No accessibility mode beyond mute visuals; the calibration minimum is a
  steady hand (average of 8 taps).

## Tooling

Made with Z-Code (automated). Balance consult: Kimi K3.1 Pro (`kimi -p`,
31/10/2026, session 72e94a62 — recommended windows and Easy-mode
anti-frustration rules, applied). Cover generated with a local Pillow
script. No external assets, fonts bundled by fallback stacks only.
