# TypeRacer Skip Ad — Design Spec
**Date:** 2026-04-06  
**Project:** Kitboga Code Jam 2026 — "Unskippable Ad"

---

## Overview

A deceptive skip mechanic: a normal-looking "Skip Ad" button that, when clicked, reveals a TypeRacer-style typing challenge. The user must type two paragraphs correctly to advance a progress bar that races against the video. Invisible difficulty scaling ensures the user never wins too easily.

---

## UI Layout

### Phase 1 — Idle
A single "Skip Ad" button, bottom-right corner of the iframe. Styled to match the existing template button — neutral, unsuspicious.

### Phase 2 — Typing Active
The skip button is hidden. A compact semi-transparent bottom strip (~130px tall, `rgba(0,0,0,0.75)` background) appears, containing top to bottom:

1. **Dual progress bars** — two stacked thin bars (8px each), labeled "Ad" and "Skip". Ad bar reflects `videoProgress` (neutral color, unmodified). Skip bar reflects `skipProgress` (green). The visual race between them is the core tension — the user can see their bar hovering just behind the ad bar.
   - **Car icon** (`🚗` or ASCII `[>]`) floats just above the skip bar's fill point, tracking with `skipProgress`. This replaces the static skip button in Phase 2.
   - **Flag icon** (`🏁` or ASCII `⚑`) floats just above the ad bar's fill point, tracking with `videoProgress`. Represents the moving target the user must overtake.
2. **Passage text** — the joined two-paragraph block rendered as flowing text. Characters are color-coded:
   - Green = correctly typed
   - Red = current error (must backspace to continue)
   - White/gray = untyped
   - Font: monospace. Text is displayed as a single long line that scrolls so the current cursor position stays centered/visible. The full passage is rendered but the container clips with `overflow: hidden`.
3. **Prompt** — small gray "Type to skip this ad" label, fades out after a few seconds.

The video remains mostly visible — the strip covers only the bottom ~24% of the 960×540 iframe.

---

## Paragraph Pool

Two categories. Each session randomly picks **one from each**, joined into one continuous passage.

### Category A — Neutral / Classic TypeRacer
Standard typing practice sentences. Familiar, unremarkable. Examples:
- "The quick brown fox jumps over the lazy dog. Pack my box with five dozen liquor jugs."
- "How vexingly quick daft zebras jump. The five boxing wizards jump quickly."

### Category B — Ironic / First-Person
Mildly funny passages written in first person, as if the user is agreeing to something. Examples:
- "I hereby confirm that I have watched this advertisement in its entirety and found it both informative and entertaining. I understand that advertising makes free content possible and I am grateful for the opportunity to engage with this message."
- "By typing these words, I acknowledge that I have read and agreed to all applicable terms and conditions, including but not limited to the right of this platform to show me additional advertisements at any time and for any duration."
- "I am a real human person who enjoys watching advertisements. I do not skip ads. I find them enriching. I have never once clicked away from an ad before it finished. This is a true statement that I am typing of my own free will."

---

## State Machine

```
state = {
  phase: 'idle' | 'typing' | 'done',
  passage: string,          // two paragraphs joined, computed at adStarted
  typedIndex: number,       // current character position
  hasError: boolean,        // blocks advancement until backspace clears it
  wordsTyped: number,       // total correctly completed words
  progressScale: number,    // starts 1.0, quietly decays
  skipProgress: number,     // 0–100, mirrors videoProgress scale
  videoProgress: number,    // 0–100, (currentTime / duration) * 100
  currentTime: number,      // seconds, updated each timeupdate
  videoDuration: number,    // set from first timeupdate event
  baseGain: number,         // computed once duration is known
}
```

---

## Typing Engine

- Keystrokes captured via `document.addEventListener('keydown')`. No `<input>` element.
- **Correct character:** advance `typedIndex`, clear `hasError`, update `skipProgress`, increment `wordsTyped` on space/end.
- **Wrong character:** set `hasError = true`, flash character red. No further advancement until `Backspace` is pressed.
- **Backspace:** clears `hasError`. Does not retreat `typedIndex` (can't un-type correct characters).
- Special characters (spaces, punctuation, apostrophes) must be typed correctly.

---

## Progress & Rubber-Band Difficulty

### Win condition
The user wins when `skipProgress > videoProgress` AND `currentTime >= 30`. Before 30 seconds, the user cannot win regardless of typing speed. After 30 seconds, exceeding the ad's progress triggers success.

### baseGain
Computed from the first `timeupdate` event (which provides `duration`). Until `duration` is known, defaults to `100 / totalWords` (assumes parity with a 60s video). Recalculated once real duration arrives.

```
totalWords = passage.split(' ').length
baseGain = 100 / totalWords  // one full bar if every word typed at video pace
```

### Dynamic progressScale (rubber-band controller)
After each correctly completed word, `progressScale` is adjusted to keep `skipProgress` close to but slightly behind `videoProgress`. Target gap = `skipProgress` should trail `videoProgress` by ~8%.

```
// Target: skipProgress ≈ 0.8 * videoProgress before 30s
targetSkip = videoProgress * 0.8
gap = targetSkip - skipProgress   // positive = user is below target, negative = ahead of target

if (currentTime < 30) {
  // Hard ceiling: skipProgress must never exceed videoProgress before 30s
  if (skipProgress >= videoProgress) progressScale = Math.max(0.1, progressScale - 0.08)  // pull back hard
  else if (gap > 10) progressScale = Math.min(1.4, progressScale + 0.03)  // below 80% target, boost
  else if (gap < -5) progressScale = Math.max(0.3, progressScale - 0.02)  // above 80% target, slow
  // within ±5% of 80% target: leave progressScale unchanged
} else {
  // After 30s: ceiling lifted, allow natural progression, gentle boost if behind
  if (skipProgress < videoProgress && gap > 20) progressScale = Math.min(1.5, progressScale + 0.04)
}
```

All adjustments are small per-word nudges. The user's bar will visibly hover just behind the ad bar — tantalisingly close — with no indication anything is being controlled.

### Skip Progress update
```
gain = baseGain * progressScale
skipProgress = Math.min(100, skipProgress + gain)
```

---

## Win / Loss

- `skipProgress > videoProgress` AND `currentTime >= 30` → `window.top.postMessage({ type: 'success' }, '*')`
- `adFinished` event received → `window.top.postMessage({ type: 'fail' }, '*')`

---

## Shell Events Used

| Event | Usage |
|-------|-------|
| `adStarted` | Transition to idle phase, show skip button |
| `adFinished` | Call `fail` |
| `timeupdate` | Update `videoDuration` (first event only), drive ad progress reference |
