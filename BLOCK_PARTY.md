# Block Party — project document (v3)

A visual instrument for composing in rhythm layers. Own project, built from scratch.
This document is the working reference: what exists today, how it's built, and what's next.
Replaces `vision_block_party.md` v0.1, which started from a different framing.

## 1. Files

| File | What it's for |
|---|---|
| `index.html` | **The source code.** The only file that gets edited. Needs the assets next to it. |
| `Block Party v3 (single file).html` | Packaged version: everything inside, opens on its own. **Not edited by hand.** |
| `tools/build_single.py` | Generates the single file from `index.html` + assets. |
| 8 `.mp3`, 10 images, `MuseoModerno-Black.ttf` | Assets (trumpet and bamboos already tuned; originals in `originales/`). |
| `Block Party - melody lab.html` | Demo comparing the 3 ways of generating a melody (A/B/C). |

Workflow: edit `index.html` → test → `python tools/build_single.py` → share the single file.
That way the two versions never drift out of sync.

## 2. Measured specs

| | Value |
|---|---|
| Tempo / meter | 110 BPM, 4 bars of 4/4 = 16 beats |
| Loop length | 8.727279 s = 384,873 samples at 44.1 kHz (exact across all 8 files) |
| One beat | 0.545 s · one bar: 2.182 s · one sixteenth note: 0.136 s |
| Audio | mp3 192 kbps stereo, 44.1 kHz |
| Grid | 24 × 13 modules of side `m` = min(width/24, height/13) |
| Stack | Canvas2D + Web Audio, no libraries |
| Mix peak with all 8 layers in phase | 0.65 (no clipping) |

## 3. Palette

| Name in code | Hex | Used for |
|---|---|---|
| PAPER | `#e8d6bd` | background |
| INK | `#281847` | button panel, dark blocks |
| BLUE | `#3337f6` | bass, right column, bottom base |
| LAVENDER | `#fb89ce` | pink: left column, drums, trumpet |
| ORANGE | `#fd5a0f` | clap, bamboos, volume bar |
| SAGE | `#adc2c7` | shakers block background |

## 4. Layer map

Coordinates in grid modules (x, y).

| Key | Layer | Base vol. | Button | Always-visible animation | Passive view (if playing) |
|---|---|---|---|---|---|
| Q | Strings | 1.0 | (6,5) | flower sliding down the left pink column (x=2.5) | flower at (5,6) |
| S | Shakers | 0.5 | (7,6) | random dots in the sage block (5,1)–(7,4) | grid of blue dots (8–12, 4–8) |
| X | Bass | 1.0 | (7,7) | fan of 10 lines over the blue block (7,1)–(18,4) | 5 horizontal lines in the center |
| F | Drums | 0.7 | (9,6) | pink diamond growing at (19.5, 2.5) | blue column + ink diamonds on the left |
| W | Chords | 0.7 | (7,5) | gradient at (21,1) + flower spinning at (22.5, 2.5) | rotated gradient at (10,6) |
| D | Bamboos | 0.5 | (8,6) | 6 bars rising from the bottom at x = 18–23 | orange rectangles below |
| C | Trumpet | 1.0 | (8,7) | 3 stacked pink ellipses, centered x=15, y = 4–13 | orange block + rounded pink shape |
| A | Clap | 0.5 | (6,6) | two orange bars closing in at y=11 | pink and blue rectangles below |

Other controls: **space** = full silence · **↑ ↓** = overall volume (in steps of 0.1) · **click** = toggle button panel / passive view.

## 5. How the code is put together

Draw order each frame (`draw`):

1. PAPER background
2. `drawBajoBase` — blue block + bass fan
3. `botonera()` **or** `vistaPasiva()` — the center (5,4)–(12,11)
4. `disenoEstatico()` — the fixed blocks of the composition
5. `animacion()` — everything that reacts to audio
6. `drawVolumeBar()` — orange strip at the top

Audio: each layer = `BufferSource` (loop) → `Gain` → `Analyser` → output.
The animations read the **RMS** of each `Analyser` every frame.

## 6. How it sounds (v2)

**Shared clock.** The first layer sets beat 1. Every new layer enters on the **next beat** (max. 0.55 s) and starts at the loop position that matches that instant, so it always stays in phase. To switch to bar-quantized: `QUANT = 'bar'` (or `'sixteenth'`) in `index.html`.
- Turning a layer off also waits for the next beat. **Space** cuts everything instantly.
- Pressing a key twice before its beat arrives cancels it.
- While waiting, the button turns **orange**.
- If nothing is playing, the clock resets and the layer starts immediately.

**Click-free loop.** The padding the mp3 adds gets trimmed (in case the browser doesn't do it already), and the last 3 ms are smoothed so they splice cleanly into the start. Bass: jump from 0.072 → 0.012. Chords: 0.082 → 0.019.

**Fades.** 4 ms fade-in and ~15 ms fade-out; overall volume moves on a ramp. No clicks.

**Mix recalibrated by loudness (measured LUFS):**

| Layer | v1 | v2 | v2 vol. |
|---|---|---|---|
| Bass | −22.6 | −25.5 | 0.72 |
| Drums | −27.3 | −25.5 | 0.86 |
| Chords | −29.0 | −29.5 | 0.66 |
| Trumpet | −29.8 | −29.5 | 1.04 |
| Strings | −31.6 | −30.5 | 1.14 |
| Bamboos | −33.6 | −31.5 | 0.64 |
| Shakers | −36.2 | −33.0 | 0.72 |
| Clap | **−42.1** | −33.0 | 1.43 |

The animations still move the same way as in v1 (`vizGain`). Peak with all 8 layers: 0.69.

**Shakers on the grid.** The dots refresh on every sixteenth note, not every frame.

## 6b. How it sounds (v3)

**Tuning.** Trumpet was 40 cents flat and bamboos 46 cents flat relative to the rest (which sits at A440, G major). Corrected offline with Rubber Band (+27 and −41 cents), processing the loop 3 times and keeping the middle pass so the splice closes cleanly; the processing delay was compensated (final offset ±1 sample). Result: trumpet −13 c, bamboos +10 c.

**Measured harmonic map** (per beat, repeats every 2 bars): Am (bass A–C) → G (bass G) → G/F# → G/D → Am anticipated on beat 4 of bar 2.

**Ducking.** The kick hits on all 16 beats (onset +2 ms, decays to 25% in ~110 ms). With Bass and Drums playing, the bass drops −5 dB on each hit and returns with a 25 ms time constant after 40 ms. Scheduled against the clock (not a compressor).

**Stereo.** Trumpet +0.30, Chords +0.10, Strings −0.18, Bamboos −0.35. Bass, drums, clap and shakers stay centered.

**Shared reverb.** A 1.6 s room generated in the browser, energy-normalized, with a 280 Hz high-pass on the send. Sends: bamboos 0.33, trumpet 0.30, clap 0.24, strings 0.15, chords 0.09. Sits ~18 dB below the dry mix.

**v3 levels** (re-measured): strings 1.14 · shakers 0.73 · bass 0.72 · drums 0.87 · chords 0.66 · bamboos 0.70 · trumpet 1.18 · clap 1.45. Peak with all 8 layers + reverb: 0.675.

**Visual.** 16-beat indicator in the top left (4 groups = 4 bars). `titulo.jpg` removed.

## 6c. Generative melodies (proposal)

Three methods, compared in the lab and in `escuchar/M1–M3`:

| | Notes | Rhythm | Clashing notes (360 melodies) |
|---|---|---|---|
| A · random | chromatic | probability | 51% |
| B · pentatonic + euclidean | G A B D E | E(k,16) | 0% |
| C · chord-guided | strong beat = chord note; rest pentatonic, by steps | E(k,16) question / E(k+1,16) answer | 0% |

C is the recommended one: besides never clashing, it has direction (contour), call-and-response phrases, and resolves on G. Natural controls for Block Party: density (k) and rotation.

## 7. Backlog

### Sound (priority)
1. **Re-export the stems as WAV or OGG** at the exact length (384,873 samples). The splice fix is a patch; ideally the loop comes perfect straight from the DAW.
2. ~~**Bass vs. kick.**~~ (done in v3) 55% of the mix's energy sits below 120 Hz: the bass (62% there) and the kick (88%) collide. The kick lands on all 16 exact beats, so the bass can be ducked ~4 dB for ~80 ms on each hit (scheduled against the clock). Gives more punch and clarity.
3. ~~**Stereo.**~~ (done in v3) Bass, drums and trumpet are mono; shakers and clap are wide. Gentle panning: trumpet to the right, bamboos to the left, chords opened up slightly.
4. ~~**Space.**~~ (reverb done in v3; still missing an output compressor) A shared reverb (send) for trumpet, bamboos and clap, plus a gentle compressor on the output to glue it together.
5. **Variation.** The 4 bars loop identically. Some layers could have 2 versions (A/B) and alternate, or fills every 8 bars.

### Visual / interaction
6. ~~Visual pulse~~ (16-beat indicator in v3) on every beat (something subtle marking the clock).
7. Buttons colored by their layer when they're on, to read at a glance which key drives which shape.
8. Touch on screen (phone/tablet) and a fixed zone to switch views.
9. ~~`titulo.jpg` loads without being used~~ (removed in v3) (~300 KB). `L.level` unused.
10. **Generative melody layer (method C)** on a new key, with density and rotation, riding the same clock.

## 8. Version log

- **v3** — Trumpet and bamboos tuned, bass/kick ducking, panning, shared reverb, beat indicator, re-measured levels, melody lab.

- **v2** — Shared clock with next-beat entry, orange button while waiting, click-free loop, fades, LUFS-based mix, shakers on the grid.
- **v1** — Working base. Same behavior as the uploaded version. Title, comments, packaging script, and this document.
