# Conway's Game of Life — Two Experiments

Two implementations of Conway's Game of Life that got out of hand.

---

## `stacked-life.html ` — 100×100 Grid, 20 Layers

20 independent 2D Life simulations stacked in 3D space. Each layer runs B3/S23 simultaneously. Rendered as flush instanced cubes with speed tiers — bottom layers tick 4× faster than top layers, creating interference patterns where the fast and slow planes meet. A cross-layer rule can be toggled: dead cells ignite when the same position is alive in both the layer above and below.

Orbit, zoom, toggle cross-layer interaction.

**Stack:** Three.js, vanilla JS.

---

## `game_of_life_that_sings-pentatonic.html & game_of_life_that_sings-schumann.html` — 10,000 Universes with Sound

A 100×100 outer Game of Life grid where every cell *contains* a running 12×12 Life simulation.

- **Alive outer cell** → inner universe runs each tick
- **Dead outer cell** → inner universe is frozen. Time stops inside it.

The outer grid's rules decide which inner universes get to experience time. Click any alive cell to zoom in and watch its universe full screen. If the cell dies while you're inside, the grid freezes mid-pattern.

### Sound

Every birth and death in the outer grid fires a tone. The sound system maps **change only** — not state, not density, only the moment a cell transitions. Silence means nothing changed. Sound means the simulation made a decision.

Two versions of the audio exist — you can switch between them by editing `colToFreq()` in the source:

---

### Version A — Pentatonic (the one that sounds like a melody)

```js
function colToFreq(c) {
  const pentatonicRatios = [1, 9/8, 5/4, 3/2, 5/3];
  const octave   = Math.floor(c / (OUTER_N / 3));
  const degree   = Math.floor((c % (OUTER_N / 3)) / (OUTER_N / 15));
  const baseFreq = 110 * Math.pow(2, octave);
  return baseFreq * pentatonicRatios[degree % 5];
}
```

Column position maps to a note in A pentatonic (110hz root, 3 octaves). Because pentatonic contains no dissonant intervals, any combination of notes from the simulation sounds consonant. The rhythm and melodic movement are entirely produced by the simulation — Conway's oscillators create periodicity, gliders move spatially across the stereo field producing directed phrases.

**Why it sounds composed:** B3/S23 has strong spatial preferences. Gliders travel diagonally, which means births and deaths cluster in columns rather than distributing randomly. Repeated column = repeated pitch = melody. Blinkers flip on a period-2 cycle = rhythm. The pentatonic scale doesn't manufacture the music — it removes the wrong notes so what's left is audible.

**The arc:** starts as dense polyphonic chaos, finds structure in the middle, resolves to a held tone or near-silence as the simulation stabilizes. You are hearing the thermodynamic trajectory of the system — high entropy to low entropy — expressed as music. The "sine wave" at the end is the last surviving oscillator, playing its note forever.

**This is the honest version of: the music was always inside the math.**

---

### Version B — Schumann + 432hz

```js
function colToFreq(c) { return 216; }         // births: 432hz / 2
const SCHUMANN_AUDIBLE = 7.83 * 16;           // deaths: 125.28hz (audible)
const SCHUMANN_RAW     = 7.83;                // deaths: 7.83hz sub layer (infrasound)
```

Every birth in the entire simulation fires at **216hz** (one octave below 432hz). Every death fires at **125.28hz** (7.83 × 16, the nearest audible octave of the Schumann resonance) with a second oscillator at the literal **7.83hz** — Earth's electromagnetic cavity resonance frequency, the standing wave between the ground and the ionosphere.

The 7.83hz oscillator is infrasound — below the threshold of hearing. Most laptop speakers won't reproduce it. On a subwoofer or studio monitors it moves air at that frequency. You won't hear it. You'll feel it as pressure.

**The difference from Version A:** Version A has melody because the pentatonic scale makes any combination of notes sound consonant, and Life's spatial structure creates pitch movement. Version B has no melody — only two notes, the conversation between creation (432) and dissolution (Schumann). What you hear instead is rhythm and density. When the simulation is active: rapid interplay between the two frequencies. As it stabilizes: they settle into a slow alternating pulse. The last oscillator produces a two-note pattern at a fixed interval — 432 then 125, 432 then 125 — which is close enough to a perfect interval that it sounds like a drone chord fading to silence.

**Why 432 and 7.83:** 432hz / 7.83hz ≈ 55.17 — they are close to a 55:1 ratio, meaning they sit near a harmonic relationship in the overtone series. This isn't coincidence — 432hz tuning was historically derived from natural harmonic relationships. Together they represent one way of thinking about "natural" or "resonant" frequency: 432 as the pitch that relates most directly to physical harmonic series, 7.83 as the frequency the planet itself resonates at.

**The honest caveat:** The Schumann resonance is real and measurable. 432hz tuning is a choice — a different choice than the modern concert standard of 440hz, with legitimate historical and mathematical grounding, but still a choice. This version sounds more like ritual than music.

---

### Stereo Field (both versions)

- **Left → Right** maps to **top → bottom rows** of the outer grid
- Each event pans to the spatial position of the cell that fired it
- A glider moving across the grid produces a tone that moves across the stereo field

### Envelopes

- **Births:** sine wave, sharp attack (15ms), 1.2s decay
- **Deaths:** triangle wave, slightly slower attack, glides down a minor third over 500ms, 700ms decay
- **Reverb:** 1.5s convolution reverb (synthesized noise impulse response)
- **Max events per tick:** 24 (evenly sampled across all births/deaths to avoid mud)

---

## Running

Open either HTML file directly in a browser. No build step, no dependencies to install.

For the sound version: click **♪ Sound: On** after the simulation starts. Web Audio requires a user gesture to initialize.

**Stack:** Canvas 2D API, Web Audio API, vanilla JS.
