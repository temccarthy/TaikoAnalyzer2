# Taiko Analyzer

> Turn taiko drum audio into readable, editable notation, with every stage of
> the pipeline visible and measurable.

## Why this attempt is different

The previous attempt failed because it was too open-ended and the ML was opaque.
This time:

- **Start small.** One drum, two sounds (don, ka), one clean clip.
- **Transparent by default.** Every stage produces output we can see and hear.
- **Incremental.** Build the simplest working version of one component, verify
  it against real audio, then move to the next. Don't perfect a stage before the
  next exists; don't build the next before the current one is checked.
- **Measurable.** Each stage gets a score. Metrics are defined when the stage
  that uses them is built, not up front.

## Audience

Personal tool for now. Public deployment is not a goal and may never be; don't
build for multiple users, accounts, or hosting until that changes.

## Starting scope

- Single drum, clean recording
- Only two hit types: **don** (center) and **ka** (rim)
- One clip (`clips/haruka_part2/haruka_part2.mp4`, ~2 min video). Work on short
  sections of it for now, not the whole clip.

Later (not now): soft notes (tsu/tsuku), flams, tempo changes, multiple drums,
ensemble recordings with shouts/fue/chappa.

## Pipeline

Each stage declares which earlier outputs it reads and adds only its own fields.
Contracts are agreed before a stage is built, so stages can be tested and
swapped in isolation.

| # | Stage | Reads | Simplest version | Output (adds) |
|---|-------|-------|------------------|---------------|
| 0 | Load & view | clip | Extract audio from video, select a section, show waveform + spectrogram, playback | audio, sample rate, section bounds |
| 1 | Onset detection | 0 | Spectral flux (optionally band-weighted toward drum frequencies) + peak picking | `[{time_s, strength}]` |
| 2 | Tempo & beats | 0, 1 | Standard beat tracker (librosa / madmom); no ML needed initially | beat times, tempo |
| 3 | Quantization | 1, 2 | Snap hits to nearest subdivision of the beat grid | `beat_index`, `subdivision` |
| 4 | Hit classification | 0, 3 | A few visible features (low-band energy, spectral centroid) + simple model (k-NN / logistic regression) | `label: don\|ka`, `confidence` |
| 5 | Notation | 3, 4 | Kuchi shōka grid (don / do-ko / ka / ka-ra on a beat grid) | readable artifact |
| 6 | Editing & feedback | all | Fix mistakes in the artifact; corrections become labeled training data | updated labels |

Notes:
- Beat tracking: try the standard tracker first and evaluate how it does on
  taiko before considering alternatives (e.g. a hand-set grid).
- Training starts on one clip. Retrain as clips and complexity are added.
- Frequency filtering emphasizes drum energy; it is not the main isolation step.
  Onset detection is.
- ML effort goes where it pays off: hit classification (stage 4).
- Western staff notation (MusicXML / LilyPond) is a possible later output.

## Minimal labeling

- Never label from scratch: correct the pipeline's guesses instead.
- Never label by hand-editing JSON. Labeling happens in the viewer.
- Every correction in the editor feeds back into the classifier (active learning).
- How to label, and when to build ground truth, is deferred (see Open questions).
  Until then, stages are checked by eye and ear.

## Visualization & verification

- **Per-stage output files** (JSON / arrays) so any stage can be inspected alone.
- **One synced viewer** with stacked tracks tied to audio playback:
  waveform → spectrogram → onset curve → detected hits → beat grid → labels.
- **Classifier scatter plot**: every hit as a point in feature space, so don/ka
  clusters and misclassifications are visible.
- **Sonification**: play the original with clicks / synthesized don-ka at each
  detected hit. The ear catches errors fast.
- **Per-stage metrics**: to be defined when each stage is built.

## Tooling

- **Web viewer** is the primary interface for visualization, ground-truth
  labeling, and editing. It grows into the stage-6 editor and a possible website.
- **All pipeline logic lives in a Python package.** The viewer only calls it
  and displays results.
- **Architecture:** local Python server (e.g. FastAPI) runs the pipeline and
  serves stage outputs as JSON; browser frontend renders, plays audio, and
  sends edits back. Harder to deploy than a static site, accepted in exchange
  for Python's audio/ML tooling.
- **Frontend:** React + TypeScript + Vite. Timeline (waveform, spectrogram,
  markers) is canvas-rendered (e.g. wavesurfer.js); React owns everything
  around it. Playback-rate updates (cursor) stay out of React state.
- No notebooks; no Sonic Visualiser (not flexible enough for tool-specific
  features later).

## Known risks

- Soft hits falling below the onset threshold
- Flams merging into a single hit
- Beat tracker locking to double/half tempo
- Taiko pieces often accelerate; fixed-tempo assumptions will break
- Ensemble audio (later) is a much harder problem than solo

## First steps

1. **Stage 0** — extract audio from the clip, select a section, view waveform +
   spectrogram with playback.
2. **Stage 1** — onset detection, checked by eye and ear (sonification).

## Open questions

- How to label and build ground truth (method, UI, when)
- Exact data contract format between stages
- Recording conditions of the source clip (solo vs. ensemble, noise)
