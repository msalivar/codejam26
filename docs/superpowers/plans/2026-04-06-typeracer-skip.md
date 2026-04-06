# TypeRacer Skip Ad — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a deceptive "Skip Ad" overlay that hides a TypeRacer-style typing challenge behind a normal-looking skip button, with a rubber-band difficulty system that keeps the user tantalizingly close to winning until 30 seconds have passed.

**Architecture:** Single `submission/submission.html` file. A flat state machine controls two phases (idle → typing). A `keydown` listener drives the typing engine. Shell `postMessage` events drive video progress. A rubber-band controller adjusts `progressScale` per word to keep skip progress trailing ad progress at ~80% until t≥30s.

**Tech Stack:** Vanilla HTML, CSS, JavaScript (ES5-compatible). No build tools. No external dependencies.

---

## File Map

| File | Role |
|------|------|
| `submission/submission.html` | Entire submission — HTML structure, CSS styles, JS logic |

---

### Task 1: HTML structure & CSS scaffold

**Files:**
- Modify: `submission/submission.html` (replace entirely)

- [ ] **Step 1: Replace submission.html with the full HTML/CSS scaffold (no JS behavior yet)**

```html
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <title>Skip Ad</title>
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <style>
    html, body {
      background: transparent;
      margin: 0;
      padding: 0;
      font-family: sans-serif;
    }

    #overlay-container {
      width: 960px;
      height: 540px;
      position: relative;
    }

    /* Phase 1 — skip button */
    #skip-btn {
      display: none;
      position: absolute;
      bottom: 60px;
      right: 0;
      padding: 10px 18px;
      background: rgba(0, 0, 0, 0.6);
      border: 1px solid rgba(255, 255, 255, 0.2);
      border-right-width: 0;
      font-size: 1rem;
      border-radius: 4px 0 0 4px;
      color: white;
      cursor: pointer;
      font-family: sans-serif;
    }

    #skip-btn:hover {
      background: rgba(0, 0, 0, 0.8);
    }

    /* Phase 2 — typing panel */
    #typing-panel {
      display: none;
      position: absolute;
      bottom: 0;
      left: 0;
      right: 0;
      height: 130px;
      background: rgba(0, 0, 0, 0.78);
      padding: 10px 14px 8px;
      box-sizing: border-box;
      flex-direction: column;
      gap: 6px;
    }

    #typing-panel.visible {
      display: flex;
    }

    /* Progress bars */
    #bars-container {
      display: flex;
      flex-direction: column;
      gap: 10px;
    }

    .bar-row {
      display: flex;
      align-items: center;
      gap: 8px;
    }

    .bar-label {
      color: rgba(255, 255, 255, 0.55);
      font-size: 11px;
      width: 28px;
      flex-shrink: 0;
    }

    .bar-track {
      flex: 1;
      height: 8px;
      background: rgba(255, 255, 255, 0.12);
      border-radius: 4px;
      position: relative;
      overflow: visible;
    }

    .bar-fill {
      height: 100%;
      border-radius: 4px;
      width: 0%;
    }

    #ad-bar  { background: rgba(255, 255, 255, 0.55); }
    #skip-bar { background: #4caf50; }

    .race-icon {
      position: absolute;
      top: -18px;
      font-size: 15px;
      transform: translateX(-50%);
      pointer-events: none;
      left: 0%;
    }

    /* Passage text */
    #passage-container {
      flex: 1;
      overflow: hidden;
      position: relative;
      display: flex;
      align-items: center;
    }

    #passage-inner {
      white-space: nowrap;
      font-family: monospace;
      font-size: 14px;
      line-height: 1.4;
      position: absolute;
      left: 0;
    }

    .char-correct { color: #4caf50; }
    .char-error   { color: #f44336; background: rgba(244, 67, 54, 0.25); border-radius: 2px; }
    .char-current { color: white; border-bottom: 2px solid white; }
    .char-pending { color: rgba(255, 255, 255, 0.45); }

    /* Prompt */
    #prompt-label {
      color: rgba(255, 255, 255, 0.45);
      font-size: 11px;
      text-align: center;
      transition: opacity 1.5s ease;
    }
  </style>
</head>
<body>
  <div id="overlay-container">

    <button id="skip-btn">Skip Ad ›</button>

    <div id="typing-panel">
      <div id="bars-container">
        <div class="bar-row">
          <span class="bar-label">Ad</span>
          <div class="bar-track">
            <div id="ad-bar" class="bar-fill"></div>
            <span id="flag-icon" class="race-icon">🏁</span>
          </div>
        </div>
        <div class="bar-row">
          <span class="bar-label">Skip</span>
          <div class="bar-track">
            <div id="skip-bar" class="bar-fill"></div>
            <span id="car-icon" class="race-icon">🚗</span>
          </div>
        </div>
      </div>

      <div id="passage-container">
        <div id="passage-inner"></div>
      </div>

      <div id="prompt-label">Type to skip this ad</div>
    </div>

  </div>
  <script>
    // JS goes here in later tasks
  </script>
</body>
</html>
```

- [ ] **Step 2: Visually verify the scaffold**

Open `index.html` in a browser. Before clicking Play:
- Page loads without errors (check console)

Temporarily add `style="display:block"` to `#skip-btn` in DevTools — verify it appears bottom-right.
Temporarily add class `visible` to `#typing-panel` in DevTools — verify the bottom strip appears, two labeled bar rows are visible, passage area and prompt are present.

Remove the temporary overrides when done.

---

### Task 2: State machine, paragraph pool & shell listeners

**Files:**
- Modify: `submission/submission.html` — replace the `// JS goes here` comment with the code below

- [ ] **Step 1: Add state, paragraph pools, pick helper, and shell listener**

Replace the script comment with:

```javascript
(function(window, document) {

  // ---- Paragraph pools ----
  var POOL_A = [
    "The quick brown fox jumps over the lazy dog. Pack my box with five dozen liquor jugs. How vexingly quick daft zebras jump.",
    "Sphinx of black quartz, judge my vow. The five boxing wizards jump quickly. Jackdaws love my big sphinx of quartz.",
    "A wizard's job is to vex chumps quickly in fog. Blowzy red vixens fight for a quick jump. The jay, pig, fox, zebra and my wolves quack.",
    "Jinxed wizards pluck ivy from the big quilt. Pack my box with five dozen liquor jugs. Grumpy wizards make toxic brew for the evil queen and jack.",
  ];

  var POOL_B = [
    "I hereby confirm that I have watched this advertisement in its entirety and found it both informative and entertaining. I understand that advertising makes free content possible and I am grateful for the opportunity to engage with this message.",
    "By typing these words, I acknowledge that I have read and agreed to all applicable terms and conditions, including but not limited to the right of this platform to show me additional advertisements at any time and for any duration.",
    "I am a real human person who enjoys watching advertisements. I do not skip ads. I find them enriching. I have never once clicked away from an ad before it finished. This is a true statement that I am typing of my own free will.",
    "I confirm that I am not attempting to circumvent this advertisement and that my intention to skip is genuine and heartfelt. I promise to think fondly of this brand and consider their products during my next shopping experience.",
  ];

  function pick(arr) {
    return arr[Math.floor(Math.random() * arr.length)];
  }

  // ---- State ----
  var state = {
    phase: 'waiting',          // 'waiting' | 'idle' | 'typing' | 'done'
    passage: '',
    typedIndex: 0,
    hasError: false,
    wordsTyped: 0,
    progressScale: 1.0,
    skipProgress: 0,
    videoProgress: 0,
    currentTime: 0,
    videoDuration: null,
    baseGain: null,
  };

  // ---- Element refs ----
  var skipBtn       = document.getElementById('skip-btn');
  var typingPanel   = document.getElementById('typing-panel');
  var adBar         = document.getElementById('ad-bar');
  var skipBarEl     = document.getElementById('skip-bar');
  var flagIcon      = document.getElementById('flag-icon');
  var carIcon       = document.getElementById('car-icon');
  var passageInner  = document.getElementById('passage-inner');
  var promptLabel   = document.getElementById('prompt-label');

  // ---- Helpers ----
  function buildPassage() {
    var a = pick(POOL_A);
    var b = pick(POOL_B);
    return Math.random() < 0.5 ? a + ' ' + b : b + ' ' + a;
  }

  function computeBaseGain() {
    var totalWords = state.passage.split(' ').length;
    state.baseGain = 100 / totalWords;
  }

  // ---- Shell message listener ----
  window.addEventListener('message', function(event) {
    if (!event.data || !event.data.type) return;

    switch (event.data.type) {
      case 'adStarted':
        state.passage = buildPassage();
        computeBaseGain();
        renderPassage();
        state.phase = 'idle';
        skipBtn.style.display = 'block';
        break;

      case 'adFinished':
        if (state.phase !== 'done') {
          state.phase = 'done';
          window.top.postMessage({ type: 'fail' }, '*');
        }
        break;

      case 'timeupdate':
        var ct = event.data.currentTime;
        var dur = event.data.duration;
        state.currentTime = ct;
        if (dur && !state.videoDuration) {
          state.videoDuration = dur;
        }
        if (dur && dur > 0) {
          state.videoProgress = (ct / dur) * 100;
        }
        updateBars();
        checkWin();
        break;
    }
  });

  // Stubs — filled in later tasks
  function renderPassage() {}
  function updateBars() {}
  function checkWin() {}

})(window, document);
```

- [ ] **Step 2: Verify shell listener wiring**

Open `index.html`, click Play, wait 2 seconds. Open DevTools console and run:

```javascript
document.getElementById('skip-btn').style.display
```

Expected: `"block"` — confirms `adStarted` fired and the skip button was shown.

---

### Task 3: Phase 1 — skip button click

**Files:**
- Modify: `submission/submission.html` — add inside the IIFE, after the shell listener block

- [ ] **Step 1: Add skip button click handler**

Add after the `window.addEventListener('message', ...)` block:

```javascript
  skipBtn.addEventListener('click', function() {
    if (state.phase !== 'idle') return;
    state.phase = 'typing';
    skipBtn.style.display = 'none';
    typingPanel.classList.add('visible');
    setTimeout(function() {
      promptLabel.style.opacity = '0';
    }, 3000);
    document.addEventListener('keydown', onKeyDown);
  });

  function onKeyDown(e) {
    // filled in Task 5
  }
```

- [ ] **Step 2: Verify phase transition**

Open `index.html`, click Play, wait for skip button to appear. Click it.

Expected:
- Skip button disappears
- Bottom typing panel slides into view
- "Type to skip this ad" label is visible
- After 3 seconds the label fades out
- No console errors

---

### Task 4: Passage text rendering & scroll

**Files:**
- Modify: `submission/submission.html` — replace the `function renderPassage() {}` and `function updateBars() {}` stubs

- [ ] **Step 1: Implement renderPassage, updatePassageDisplay, scrollToCurrentChar**

Replace `function renderPassage() {}` with:

```javascript
  function renderPassage() {
    passageInner.innerHTML = '';
    for (var i = 0; i < state.passage.length; i++) {
      var span = document.createElement('span');
      span.textContent = state.passage[i];
      passageInner.appendChild(span);
    }
    updatePassageDisplay();
  }

  function updatePassageDisplay() {
    var spans = passageInner.children;
    for (var i = 0; i < spans.length; i++) {
      var sp = spans[i];
      if (i < state.typedIndex) {
        sp.className = 'char-correct';
      } else if (i === state.typedIndex) {
        sp.className = state.hasError ? 'char-error' : 'char-current';
      } else {
        sp.className = 'char-pending';
      }
    }
    scrollToCurrentChar();
  }

  function scrollToCurrentChar() {
    var container = document.getElementById('passage-container');
    var currentSpan = passageInner.children[state.typedIndex];
    if (!currentSpan) return;
    var containerWidth = container.offsetWidth;
    var charLeft = currentSpan.offsetLeft;
    var shift = Math.max(0, charLeft - containerWidth / 2);
    passageInner.style.transform = 'translateX(-' + shift + 'px)';
  }
```

- [ ] **Step 2: Verify passage renders correctly**

Open `index.html`, click Play, click Skip Ad. The typing panel opens.

Expected:
- Full passage text is visible in the passage area, in gray/white monospace
- First character has a white underline (char-current)
- No console errors

---

### Task 5: Typing engine

**Files:**
- Modify: `submission/submission.html` — replace `function onKeyDown(e) {}` stub and add `onWordCompleted`

- [ ] **Step 1: Implement onKeyDown**

Replace `function onKeyDown(e) { // filled in Task 5 }` with:

```javascript
  function onKeyDown(e) {
    if (state.phase !== 'typing') return;

    if (e.key === 'Backspace') {
      if (state.hasError) {
        state.hasError = false;
        updatePassageDisplay();
      }
      return;
    }

    if (e.key.length !== 1) return;  // ignore Tab, Enter, F-keys, etc.
    if (state.hasError) return;       // must backspace to clear error first

    var expected = state.passage[state.typedIndex];

    if (e.key === expected) {
      state.typedIndex++;
      state.hasError = false;

      // Word completed: on space, or on last character
      if (expected === ' ' || state.typedIndex >= state.passage.length) {
        onWordCompleted();
      }

      if (state.typedIndex >= state.passage.length) {
        // Passage fully typed — remove listener, let timeupdate trigger win
        document.removeEventListener('keydown', onKeyDown);
        updatePassageDisplay();
        return;
      }
    } else {
      state.hasError = true;
    }

    updatePassageDisplay();
  }
```

- [ ] **Step 2: Add onWordCompleted stub (progress logic comes in Task 6)**

Add after `onKeyDown`:

```javascript
  function onWordCompleted() {
    state.wordsTyped++;
    // rubber-band logic added in Task 6
  }
```

- [ ] **Step 3: Verify typing mechanics**

Open `index.html`, click Play, click Skip Ad. Type the first few characters.

Expected:
- Correctly typed characters turn green
- Current character has white underline and advances right
- Typing a wrong character turns it red; no further advancement
- Pressing Backspace clears the red and restores the white underline
- Text scrolls left as cursor advances past the center
- No console errors

---

### Task 6: Progress bars, rubber-band controller & win condition

**Files:**
- Modify: `submission/submission.html` — replace `updateBars`, `checkWin` stubs and expand `onWordCompleted`

- [ ] **Step 1: Implement updateBars**

Replace `function updateBars() {}` with:

```javascript
  function updateBars() {
    adBar.style.width    = state.videoProgress + '%';
    skipBarEl.style.width = state.skipProgress + '%';
    flagIcon.style.left  = state.videoProgress + '%';
    carIcon.style.left   = state.skipProgress  + '%';
  }
```

- [ ] **Step 2: Implement checkWin**

Replace `function checkWin() {}` with:

```javascript
  function checkWin() {
    if (state.phase !== 'typing') return;
    if (state.skipProgress > state.videoProgress && state.currentTime >= 30) {
      state.phase = 'done';
      window.top.postMessage({ type: 'success' }, '*');
    }
  }
```

- [ ] **Step 3: Implement rubber-band difficulty in onWordCompleted**

Replace `function onWordCompleted() { state.wordsTyped++; }` with:

```javascript
  function onWordCompleted() {
    state.wordsTyped++;

    var targetSkip = state.videoProgress * 0.8;
    var gap = targetSkip - state.skipProgress; // positive = user below target

    if (state.currentTime < 30) {
      // Hard ceiling: skipProgress must never exceed videoProgress before 30s
      if (state.skipProgress >= state.videoProgress) {
        state.progressScale = Math.max(0.1, state.progressScale - 0.08);
      } else if (gap > 10) {
        // User falling below 80% target — boost
        state.progressScale = Math.min(1.4, state.progressScale + 0.03);
      } else if (gap < -5) {
        // User exceeding 80% target — slow
        state.progressScale = Math.max(0.3, state.progressScale - 0.02);
      }
      // Within ±5% of target: leave progressScale unchanged
    } else {
      // After 30s: ceiling lifted, gentle boost if far behind
      if (state.skipProgress < state.videoProgress && gap > 20) {
        state.progressScale = Math.min(1.5, state.progressScale + 0.04);
      }
    }

    var gain = state.baseGain * state.progressScale;
    state.skipProgress = Math.min(100, state.skipProgress + gain);

    updateBars();
    checkWin();
  }
```

- [ ] **Step 4: Verify the race**

Open `index.html`, click Play, click Skip Ad. Type quickly.

Expected:
- Ad bar (gray) and skip bar (green) both advance
- 🏁 flag tracks the ad bar fill; 🚗 car tracks the skip bar fill
- Skip bar hovers visibly behind the ad bar
- Before 30s: no matter how fast you type, skip bar does not overtake ad bar
- After 30s: if you type past the ad bar's position, the page reloads with a success banner
- If video ends before you win, fail banner appears and page reloads
- No console errors

---

### Task 7: Final polish — icon positioning fix & edge case

**Files:**
- Modify: `submission/submission.html`

- [ ] **Step 1: Clamp icon positions to avoid overflow at 0% and 100%**

The icons can overflow the bar edges at extremes. Update `updateBars` to clamp:

```javascript
  function updateBars() {
    var adPct   = Math.min(99, Math.max(1, state.videoProgress));
    var skipPct = Math.min(99, Math.max(1, state.skipProgress));
    adBar.style.width     = state.videoProgress + '%';
    skipBarEl.style.width = state.skipProgress  + '%';
    flagIcon.style.left   = adPct   + '%';
    carIcon.style.left    = skipPct + '%';
  }
```

- [ ] **Step 2: Guard against winning immediately on adStarted if timeupdate fires before typing begins**

`checkWin` already guards `state.phase !== 'typing'`, and `state.skipProgress` starts at 0 so the condition `skipProgress > videoProgress` won't be true at the start. No extra change needed — verify this is the case by confirming `state.skipProgress` starts at 0 and `state.videoProgress` starts at 0, so `0 > 0` is false.

- [ ] **Step 3: Verify full end-to-end flow**

Open `index.html`. Run through the full experience:

1. Click Play — skip button appears bottom-right. Looks normal.
2. Click Skip Ad — typing panel appears, prompt fades after 3s.
3. Type correctly — green characters, car advances, stays behind flag.
4. Type incorrectly — character turns red, typing is blocked until Backspace.
5. At ~30s, type past the flag — success banner appears, page reloads.
6. Reload and let the video finish without typing — fail banner appears.

Verify no console errors in any of these flows.

- [ ] **Step 4: Test with different example ads**

Use the dropdown in the shell to switch between the 4 examples. Verify:
- Each loads a fresh passage (random selection)
- `baseGain` re-calibrates (longer/shorter passages feel appropriately paced)
- No state leaks between loads (page reloads on success/fail, so this is handled automatically)
