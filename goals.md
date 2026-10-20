# Taiko Analyzer — Goals

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
- **Measurable.** Each stage has a score against hand-checked ground truth.

## Audience

Personal tool for now. Public deployment is not a goal and may never be; don't
build for multiple users, accounts, or hosting until that changes.

## Starting scope

- Single drum, clean recording
- Only two hit types: **don** (center) and **ka** (rim)
- One short clip (15–30 s) with a clear pattern

Later (not now): soft notes (tsu/tsuku), flams, tempo changes, multiple drums,
ensemble recordings with shouts/fue/chappa.

## Pipeline

Each stage reads the previous stage's output and adds only its own fields.
Contracts are agreed before a stage is built, so stages can be tested and
swapped in isolation.

| # | Stage | Simplest version | Output (adds) |
|---|-------|------------------|---------------|
| 0 | Load & view | Load audio, show waveform + spectrogram, playback | audio, sample rate |
| 1 | Onset detection | Spectral flux (optionally band-weighted toward drum frequencies) + peak picking | `[{time_s, strength}]` |
| 2 | Hit classification | A few visible features (low-band energy, spectral centroid) + simple model (k-NN / logistic regression) | `label: don\|ka`, `confidence` |
| 3 | Tempo & beats | Standard beat tracker (librosa / madmom); no ML needed initially | beat times, tempo |
| 4 | Quantization | Snap hits to nearest subdivision of the beat grid | `beat_index`, `subdivision` |
| 5 | Notation | Kuchi shōka grid (don / do-ko / ka / ka-ra on a beat grid) | readable artifact |
| 6 | Editing & feedback | Fix mistakes in the artifact; corrections become labeled training data | updated labels |

Notes:
- Frequency filtering emphasizes drum energy; it is not the main isolation step.
  Onset detection is.
- ML effort goes where it pays off: hit classification (stage 2).
- Western staff notation (MusicXML / LilyPond) is a possible later output.

## Minimal labeling

- Never label from scratch: correct the pipeline's guesses instead.
- Ground truth for the first clip is made by correcting auto-detected onsets.
- Every correction in the editor feeds back into the classifier (active learning).

## Visualization & verification

- **Per-stage output files** (JSON / arrays) so any stage can be inspected alone.
- **One synced viewer** with stacked tracks tied to audio playback:
  waveform → spectrogram → onset curve → detected hits → beat grid → labels.
- **Classifier scatter plot**: every hit as a point in feature space, so don/ka
  clusters and misclassifications are visible.
- **Sonification**: play the original with clicks / synthesized don-ka at each
  detected hit. The ear catches errors fast.
- **Per-stage metrics** against ground truth (e.g. onset precision/recall/F1,
  classification accuracy, quantization error).

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

1. **Stage 0** — load one clip, view waveform + spectrogram with playback.
2. **Ground truth** — mark true hits and don/ka labels for that clip.
3. **Stage 1** — onset detection, scored against ground truth, checked by ear.

## Open questions

- Source clip: format, length, recording conditions?
- Exact data contract format between stages
