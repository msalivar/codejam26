# Kitboga Code Jam 2026 — TypeRacer Skip

## What is this?

Clicking "Skip Ad" doesn't skip the ad. Instead, it opens a typing challenge at the bottom of the screen.

A **Skip** progress bar and an **Ad** progress bar race across the screen. To skip the ad, the user must type a passage fast enough to get their bar ahead of the ad's bar. The passage is infinite — it keeps appending new text as you approach the end, so you can never just wait it out.

## The catch

The difficulty adjusts automatically to keep the skip bar hovering just behind the ad bar. Type too fast and it slows you down. Fall too far behind and it helps you catch up — but only a little.

You also can't win too early. Skipping is only possible after at least 30% of the ad has played (capped at 30 seconds on longer ads). Before that, no matter how fast you type, the skip bar is hard-capped just below the ad bar.

As the ad gets close to the end (70%, 80%, 90%), the typing gets a small speed boost to give the user a fighting chance.

## Secret hotkeys

These only work once the typing panel is open:

| Key | Effect |
|-----|--------|
| `F8` | Cheat mode — removes all restrictions and lets the skip bar progress faster |
| `F10` | Inserts a deliberately difficult string just ahead in the passage (lots of apostrophes, semicolons, long words) |
