# Surah Al-Hajj — Memorization Campaign Spec

**Purpose:** Build a slick, game-inspired, memory-only single-file HTML app for memorizing Surah Al-Hajj through time-aware retention, spaced review, recall scoring, section progression, and long-term preservation.

**Target file:** `surah-al-hajj-memorization.html`  
**Format:** One standalone HTML file with embedded CSS and JavaScript.  
**Storage:** Browser `localStorage`, with optional JSON export/import backup.  
**Scope for MVP:** Memorization layer only. No tafsir, no i'rab, no Lisan al-Arab, no translation panels, no audio.

---

## 1. Product Vision

The app should feel like a premium memorization campaign, not a school worksheet.

The user opens the file and enters a polished game-style dashboard where Surah Al-Hajj is treated as a long-term mastery campaign. The goal is not to memorize everything in one day, collect all points, and stop. The goal is to keep returning until each ayah and block proves stable over time.

The interface should borrow from games:

- campaign map
- section zones
- locked and unlocked stages
- retention bars
- glowing progress rings
- missions due today
- at-risk reviews
- rank and streak display
- mastery badges
- recall challenge screens
- subtle unlock animations

But the underlying logic should remain serious and memory-focused.

The app rewards only memorization recall over time.

---

## 2. Non-Negotiable Constraints

### Must include

- One standalone `.html` file.
- Full time-aware behavior using JavaScript `Date`.
- Progress saved in `localStorage`.
- Optional export/import of progress as JSON.
- No backend.
- No account/login.
- No audio controls or audio placeholders.
- No external framework required.
- Memory-only MVP.
- Points / score / retention only tied to actual recall.
- Section unlocks based only on prior section retention percentage.
- Slick, premium game-like interface.

### Must not include in MVP

- Tafsir.
- I'rab.
- Lisan al-Arab.
- Translation study cards.
- Audio.
- Voice recognition.
- Heavy dependency stack.
- Review blocking based on memory debt.
- Complex recovery mode.
- Forced limitation on how many ayat the user can learn inside an unlocked section.

---

## 3. Surah Structure

The app should use six major sections as the main campaign zones.

| Section | Name | Ayat | Unlock Rule |
|---|---|---:|---|
| 1 | The Tremor | 1-13 | Unlocked by default |
| 2 | The Division | 14-24 | Unlocks when Section 1 reaches target retention |
| 3 | The House | 25-37 | Unlocks when Section 2 reaches target retention |
| 4 | The Defense | 38-48 | Unlocks when Section 3 reaches target retention |
| 5 | The Test | 49-66 | Unlocks when Section 4 reaches target retention |
| 6 | The Surrender | 67-78 | Unlocks when Section 5 reaches target retention |

### Recommended section unlock threshold

Default threshold: **60% section retention**.

The threshold should be configurable in a single JavaScript config object.

```js
const CONFIG = {
  sectionUnlockThreshold: 60
};
```

### First 3 sections for the mockup/MVP demo

The initial mockup should focus on:

1. **Section 1 — The Tremor** — Ayat 1-13
2. **Section 2 — The Division** — Ayat 14-24
3. **Section 3 — The House** — Ayat 25-37

Section 1 starts unlocked. Sections 2 and 3 appear visually on the map, but Section 2 is locked until Section 1 reaches 60%, and Section 3 is locked until Section 2 reaches 60%.

---

## 4. Memorization Blocks

The app should support both ayah-level recall and block-level recall.

### Full Surah block structure

| Section | Blocks |
|---|---|
| Section 1 — The Tremor | 1-7, 8-13 |
| Section 2 — The Division | 14-18, 19-24 |
| Section 3 — The House | 25-29, 30-37 |
| Section 4 — The Defense | 38-41, 42-48 |
| Section 5 — The Test | 49-57, 58-66 |
| Section 6 — The Surrender | 67-72, 73-78 |

### Why blocks matter

Individual ayah memory is not enough. Qur'an memorization also depends on flow from one ayah to the next. The app therefore tracks:

- individual ayah retention
- full block recall retention
- section retention calculated from both

---

## 5. Memory Science Basis

The app should be built around two proven principles:

1. **Retrieval practice:** active recall strengthens later retention more than passive rereading.
2. **Distributed practice / spacing:** review sessions spread over time are better than massed repetition in one sitting.

Useful references for later documentation:

- Roediger & Karpicke, 2006 — testing effect / retrieval practice: https://pubmed.ncbi.nlm.nih.gov/16507066/
- Cepeda et al., 2006 — distributed practice review: https://pubmed.ncbi.nlm.nih.gov/16719566/
- Ebbinghaus forgetting curve background / replication discussion: https://pmc.ncbi.nlm.nih.gov/articles/PMC4492928/

The app does not need to implement a complex scientific model. It should use a practical review ladder inspired by these principles.

---

## 6. Retention Ladder

Each ayah and each block moves through retention milestones.

The user cannot reach 100% in one day. Higher retention stages require coming back after real time has passed.

| Stage | Timing Requirement | Retention % |
|---|---:|---:|
| First Lock | immediate clean recall | 15% |
| Same-Day Hold | 30-60 minutes later | 25% |
| 1-Day Hold | 24 hours later | 40% |
| 3-Day Hold | 3 days later | 55% |
| 7-Day Hold | 7 days later | 70% |
| 14-Day Hold | 14 days later | 80% |
| 30-Day Hold | 30 days later | 90% |
| Deep Retention | 60 or 90 days later | 100% |

### Important timing rule

The app should not count “tomorrow” by calendar date alone.

Example: if the user memorizes at 11:50pm and opens the app at 12:10am, only 20 minutes passed. That should not unlock a 1-day review.

Use real elapsed time:

```js
const elapsedMs = Date.now() - lastSuccessfulRecallAt;
```

---

## 7. Recall Result Buttons

After every recall attempt, the user self-rates honestly.

### Response options

| Button | Meaning | Effect |
|---|---|---|
| Perfect | recalled without mistake | can advance if timing requirement is met |
| Minor Hesitation | small hesitation, self-corrected | can preserve current stage, but should not advance |
| Needed Cue | needed help/hint | no advancement; may mark for review |
| Failed | could not recall | can drop stage or flag as weak |

### Advancement rule

Only **Perfect** can advance the stage.

Minor hesitation may maintain the stage, but does not unlock the next milestone.

---

## 8. Decay Logic

The retention score should not remain fixed forever.

If the user misses the expected review window, the ayah or block can become overdue and eventually decay.

### Review windows

Each stage has:

- target due time
- grace period
- overdue state
- decay threshold

Example:

| Stage | Next Due | Grace Period | Decay if ignored |
|---|---:|---:|---|
| 25% | 24h | +12h | drop to 15% |
| 40% | 3d | +1d | drop to 25% |
| 55% | 7d | +2d | drop to 40% |
| 70% | 14d | +3d | drop to 55% |
| 80% | 30d | +7d | drop to 70% |
| 90% | 60/90d | +14d | drop to 80% |

### Decay behavior

Decay should feel visible but not punishing.

Visual states:

- normal: calm blue/gold
- due soon: cyan pulse
- due now: gold glow
- overdue: orange glow
- decayed: red/orange alert

The main point: the score represents current retention health, not historical effort.

---

## 9. Section Retention Formula

Section retention should combine ayah memory and block flow memory.

Recommended formula:

```txt
Section Retention = (Average Ayah Retention × 70%) + (Average Block Recall Retention × 30%)
```

Why: a person may know individual ayat, but still fail transitions. Full-block recall should matter.

### Section unlock rule

A section unlocks the next section when:

```txt
Current Section Retention >= Section Unlock Threshold
```

Default threshold: 60%.

No other unlock rule should be added for MVP.

---

## 10. Time-Aware Data Model

### Main storage key

```js
const STORAGE_KEY = "surahAlHajjMemorizationCampaign_v1";
```

### Suggested app state shape

```js
const appState = {
  version: 1,
  createdAt: "2026-05-28T18:00:00.000Z",
  updatedAt: "2026-05-28T18:30:00.000Z",
  settings: {
    sectionUnlockThreshold: 60,
    deepRetentionDays: 90
  },
  streak: {
    current: 0,
    best: 0,
    lastCompletedReviewDate: null
  },
  sections: {
    "1": {
      id: 1,
      name: "The Tremor",
      ayahRange: [1, 13],
      unlocked: true,
      retention: 0
    },
    "2": {
      id: 2,
      name: "The Division",
      ayahRange: [14, 24],
      unlocked: false,
      retention: 0
    },
    "3": {
      id: 3,
      name: "The House",
      ayahRange: [25, 37],
      unlocked: false,
      retention: 0
    }
  },
  ayat: {
    "2": {
      ayahNumber: 2,
      sectionId: 1,
      blockId: "1-7",
      retention: 55,
      stageIndex: 3,
      stageName: "3-Day Hold",
      firstLockedAt: "2026-05-28T18:00:00.000Z",
      lastSuccessfulRecallAt: "2026-05-31T18:00:00.000Z",
      nextDueAt: "2026-06-07T18:00:00.000Z",
      lastResult: "perfect",
      attempts: []
    }
  },
  blocks: {
    "1-7": {
      id: "1-7",
      sectionId: 1,
      ayahRange: [1, 7],
      retention: 55,
      stageIndex: 3,
      lastSuccessfulRecallAt: null,
      nextDueAt: null,
      attempts: []
    }
  }
};
```

### Attempt object

```js
const attempt = {
  targetType: "ayah", // "ayah" or "block"
  targetId: "2",
  result: "perfect", // perfect | minor | cue | failed
  attemptedAt: "2026-05-28T19:00:00.000Z",
  previousRetention: 40,
  newRetention: 55,
  advanced: true
};
```

---

## 11. Retention Stage Config

Use a config array for easy tuning.

```js
const RETENTION_STAGES = [
  { name: "First Lock", minDelayMs: 0, retention: 15, nextDelayMs: 30 * 60 * 1000 },
  { name: "Same-Day Hold", minDelayMs: 30 * 60 * 1000, retention: 25, nextDelayMs: 24 * 60 * 60 * 1000 },
  { name: "1-Day Hold", minDelayMs: 24 * 60 * 60 * 1000, retention: 40, nextDelayMs: 3 * 24 * 60 * 60 * 1000 },
  { name: "3-Day Hold", minDelayMs: 3 * 24 * 60 * 60 * 1000, retention: 55, nextDelayMs: 7 * 24 * 60 * 60 * 1000 },
  { name: "7-Day Hold", minDelayMs: 7 * 24 * 60 * 60 * 1000, retention: 70, nextDelayMs: 14 * 24 * 60 * 60 * 1000 },
  { name: "14-Day Hold", minDelayMs: 14 * 24 * 60 * 60 * 1000, retention: 80, nextDelayMs: 30 * 24 * 60 * 60 * 1000 },
  { name: "30-Day Hold", minDelayMs: 30 * 24 * 60 * 60 * 1000, retention: 90, nextDelayMs: 90 * 24 * 60 * 60 * 1000 },
  { name: "Deep Retention", minDelayMs: 90 * 24 * 60 * 60 * 1000, retention: 100, nextDelayMs: null }
];
```

---

## 12. Core Algorithms

### A. Due status

```js
function getDueStatus(item, now = Date.now()) {
  if (!item.nextDueAt) return "not_started";

  const due = new Date(item.nextDueAt).getTime();
  const diff = due - now;

  if (diff > 24 * 60 * 60 * 1000) return "future";
  if (diff > 0) return "due_soon";

  const overdueBy = Math.abs(diff);
  if (overdueBy < 24 * 60 * 60 * 1000) return "due_now";
  if (overdueBy < 3 * 24 * 60 * 60 * 1000) return "overdue";
  return "decay_risk";
}
```

### B. Perfect recall advancement

```js
function applyPerfectRecall(item, now = Date.now()) {
  const currentStage = RETENTION_STAGES[item.stageIndex ?? -1];
  const nextStageIndex = Math.min((item.stageIndex ?? -1) + 1, RETENTION_STAGES.length - 1);
  const nextStage = RETENTION_STAGES[nextStageIndex];

  const lastSuccess = item.lastSuccessfulRecallAt
    ? new Date(item.lastSuccessfulRecallAt).getTime()
    : null;

  const canAdvance = !lastSuccess || now - lastSuccess >= nextStage.minDelayMs;

  if (canAdvance) {
    item.stageIndex = nextStageIndex;
    item.stageName = nextStage.name;
    item.retention = nextStage.retention;
    item.lastSuccessfulRecallAt = new Date(now).toISOString();
    item.nextDueAt = nextStage.nextDelayMs
      ? new Date(now + nextStage.nextDelayMs).toISOString()
      : null;
    return { advanced: true, item };
  }

  // Perfect recall too early: record success, but do not advance.
  item.lastSuccessfulRecallAt = new Date(now).toISOString();
  return { advanced: false, item };
}
```

### C. Minor hesitation

```js
function applyMinorHesitation(item, now = Date.now()) {
  item.lastResult = "minor";
  item.lastAttemptAt = new Date(now).toISOString();
  // No advancement. No drop by default.
  return item;
}
```

### D. Needed cue

```js
function applyNeededCue(item, now = Date.now()) {
  item.lastResult = "cue";
  item.lastAttemptAt = new Date(now).toISOString();
  // No advancement. Optionally mark weak.
  item.weak = true;
  return item;
}
```

### E. Failed recall

```js
function applyFailedRecall(item, now = Date.now()) {
  item.lastResult = "failed";
  item.lastAttemptAt = new Date(now).toISOString();

  // Optional: drop one stage, but do not go below First Lock if already started.
  if ((item.stageIndex ?? -1) > 0) {
    item.stageIndex -= 1;
    const stage = RETENTION_STAGES[item.stageIndex];
    item.stageName = stage.name;
    item.retention = stage.retention;
    item.nextDueAt = new Date(now + stage.nextDelayMs).toISOString();
  }

  return item;
}
```

### F. Section retention calculation

```js
function calculateSectionRetention(sectionId, state) {
  const sectionAyat = Object.values(state.ayat).filter(a => a.sectionId === sectionId);
  const sectionBlocks = Object.values(state.blocks).filter(b => b.sectionId === sectionId);

  const avg = arr => arr.length
    ? arr.reduce((sum, x) => sum + (x.retention || 0), 0) / arr.length
    : 0;

  const ayahAverage = avg(sectionAyat);
  const blockAverage = avg(sectionBlocks);

  return Math.round((ayahAverage * 0.7) + (blockAverage * 0.3));
}
```

### G. Section unlock calculation

```js
function updateSectionUnlocks(state) {
  const threshold = state.settings.sectionUnlockThreshold;

  for (let i = 1; i <= 5; i++) {
    const current = state.sections[String(i)];
    const next = state.sections[String(i + 1)];

    if (current && next && current.retention >= threshold) {
      next.unlocked = true;
    }
  }

  return state;
}
```

---

## 13. UI Architecture

The single HTML page should behave like a mini app with views/tabs.

### Main views

1. Dashboard
2. Surah Map
3. Section View
4. Block View
5. Recall Challenge
6. Achievements
7. Export / Import

### MVP can implement these as panels in one page

No route system required. Use hidden/visible sections:

```html
<section id="dashboardView"></section>
<section id="sectionView"></section>
<section id="recallView"></section>
```

---

## 14. Visual Design Direction

### Overall style

Premium sacred-tech game dashboard.

Not childish. Not cartoonish. Not school-like.

### Mood

- dark midnight blue / obsidian background
- luminous gold accents
- cyan glow for active memory systems
- orange/red for overdue/decay risk
- emerald/green for stable and mastered states
- glassmorphism panels
- subtle star/particle field
- thin ornamental borders
- cinematic glow and depth
- refined Arabic-friendly typography

### Suggested typefaces

Use system-safe fonts for offline simplicity:

```css
font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
```

For Arabic or Qur'anic text later, use fallback:

```css
font-family: "Amiri", "Scheherazade New", serif;
```

Since MVP is memory dashboard only, Arabic full ayah rendering can be minimal or omitted in first prototype.

### Visual states

| State | Color / feel |
|---|---|
| locked | dark, dimmed, padlock icon |
| unlocked | soft blue/gold glow |
| active | strong gold border |
| due soon | cyan pulse |
| due now | gold pulse |
| overdue | orange warning |
| decay risk | red/orange glow |
| mastered | emerald/gold prestige glow |

---

## 15. First 3 Sections Mockup Requirements

The first implementation should look like a polished mockup for these sections.

### Header

Show:

- `Surah Al-Hajj — Memorization Campaign`
- Overall Retention
- Streak
- Rank
- Time-aware retention tag

Example:

```txt
Overall Retention: 32%
Streak: 6 days
Rank: Seeker
Time-aware retention system
```

### Left panel: Surah Map

Show three section cards:

1. Section 1 — The Tremor — Ayat 1-13 — Active — retention %
2. Section 2 — The Division — Ayat 14-24 — Locked/Unlocked depending on Section 1
3. Section 3 — The House — Ayat 25-37 — Locked/Unlocked depending on Section 2

Each section card should have:

- section number
- icon/visual motif
- section name
- ayah range
- retention ring
- lock/unlock status

### Center panel: Selected Section

When Section 1 is selected:

Show:

- section title
- section retention
- block cards
- selected block overview
- ayah nodes
- selected ayah recall panel

### Block cards

For Section 1:

- Block 1 — Ayat 1-7
- Block 2 — Ayat 8-13

Each block shows:

- retention %
- stage name
- next review due time
- `Enter Recall` button

### Ayah nodes

For selected block, show ayah nodes.

Each ayah node should display:

- ayah number
- retention state ring
- next due time
- status icon

### Selected ayah panel

Example for Ayah 2:

```txt
Ayah 2
Current Retention: 55%
Stage: Early Mid-Term
Last Recall: Perfect
Next Review: 18h
Decay Risk: Low
[Start Recall Challenge]
```

After recall:

```txt
How was your recall?
[Perfect] [Minor Hesitation] [Needed Cue] [Failed]
```

### Right panel

Show:

- Today's Missions
- At Risk
- Next Milestone
- Achievements

Example missions:

```txt
Review Ayah 2
Review Ayah 5
Recite Block 1 from memory
```

Example milestone:

```txt
Reach 60% retention in Section 1 to unlock Section 2.
```

---

## 16. Recall Challenge Screen

The challenge screen should feel immersive.

### Before recall

Show:

- target type: Ayah or Block
- target number/range
- current stage
- current retention
- next milestone if successful
- optional cue mode

### Cue modes for memorization

Because MVP is memory-only, cue modes can exist without study content.

Possible cue modes:

1. Ayah number only
2. First-word cue
3. First words of internal segments
4. Blank recall

For now, only implement:

- ayah number only
- first-word cue if ayah data is available

### After recall

User clicks one:

- Perfect
- Minor Hesitation
- Needed Cue
- Failed

Then display:

- whether stage advanced
- new retention %
- next due date/time
- section retention update
- unlock event if triggered

---

## 17. Unlock Animation

When Section 1 reaches threshold and Section 2 unlocks:

Show a short visual event:

```txt
SECTION UNLOCKED
Section 2 — The Division
Ayat 14-24 now available
```

Visual behavior:

- locked card glows
- padlock opens
- gold/cyan pulse
- section card becomes clickable

This is purely section-to-section progression.

No other unlock logic.

---

## 18. Achievements

Achievements should be cosmetic and retention-based.

Examples:

| Badge | Requirement |
|---|---|
| First Lock | first ayah reaches 15% |
| Same-Day Hold | first ayah reaches 25% |
| 1-Day Hold | first ayah reaches 40% |
| 3-Day Hold | first ayah reaches 55% |
| 7-Day Hold | first ayah reaches 70% |
| 30-Day Hold | first ayah reaches 90% |
| Deep Retention | first ayah reaches 100% |
| First Block Stabilized | one block reaches 70% |
| First Section Unlock | Section 2 unlocked |

Badges should not give points. They are visual recognition only.

---

## 19. Rank System

Rank is based on overall retention, not speed.

Suggested rank ladder:

| Rank | Overall Retention |
|---|---:|
| Seeker | 0-14% |
| Initiated | 15-24% |
| Holder | 25-39% |
| Stabilizer | 40-54% |
| Preserver | 55-69% |
| Fortified | 70-79% |
| Anchored | 80-89% |
| Guardian | 90-99% |
| Deep Retention | 100% |

Overall retention should be calculated across the unlocked and started content, with clear UI wording.

---

## 20. Today's Missions

Mission list should be generated from time-aware due items.

Priority:

1. overdue ayat
2. due-now ayat
3. due-soon ayat
4. due blocks
5. next suggested new recall if no reviews are due

Missions should not award separate points. Completing a mission means performing the relevant recall attempt.

---

## 21. Streak System

Streak should count only days where the user completes at least one due recall successfully.

A day should be based on local date:

```js
new Date().toLocaleDateString()
```

Streak should not increase from simply opening the app.

---

## 22. Export / Import

Because `localStorage` is device/browser-based, add backup tools.

### Export

- Button: `Export Progress`
- Downloads `surah-al-hajj-progress.json`

### Import

- Button: `Import Progress`
- User selects previous JSON
- App replaces current state after confirmation

### Reset

- Button: `Reset Campaign`
- Requires confirmation.

---

## 23. Single-File Implementation Plan

### File sections

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Surah Al-Hajj — Memorization Campaign</title>
  <style>
    /* all CSS here */
  </style>
</head>
<body>
  <div id="app"></div>
  <script>
    // all JavaScript here
  </script>
</body>
</html>
```

### JavaScript modules inside same script

Use sections/comments:

```js
// CONFIG
// DATA MODEL
// STORAGE
// TIME HELPERS
// RETENTION ENGINE
// UNLOCK ENGINE
// RENDERING
// EVENT HANDLERS
// EXPORT IMPORT
// INIT
```

---

## 24. Acceptance Criteria

The MVP is successful if:

- Opening the HTML displays a slick game-like dashboard.
- Section 1 is unlocked by default.
- Sections 2 and 3 display as locked until retention threshold is met.
- User can select Section 1 and see blocks 1-7 and 8-13.
- User can select an ayah and start recall.
- User can rate recall as Perfect / Minor Hesitation / Needed Cue / Failed.
- Perfect recall updates retention only when the timing rule allows it.
- The app saves progress in `localStorage`.
- Closing and reopening the HTML preserves progress.
- The app detects that time has passed and updates due/overdue status.
- Section 2 unlocks when Section 1 reaches the configured threshold.
- No audio appears anywhere.
- No study layer appears in MVP.
- Export/import works or is at least scaffolded clearly.

---

## 25. Development Notes for Codex

Build the first version as a polished interactive prototype, not a static mockup.

Prioritize:

1. visual polish
2. time-aware retention logic
3. clean localStorage state
4. section unlock by percentage
5. recall result workflow

Do not overbuild:

- no backend
- no external database
- no audio
- no tafsir/study layer
- no React unless explicitly needed later

A plain HTML/CSS/JS version is preferred for the first pass.

---

## 26. Final Product Summary

This app is a **single-file, game-inspired Qur'an memorization campaign** for Surah Al-Hajj.

It is not a study notebook.
It is not a tafsir app.
It is not an audio player.
It is not a one-day points game.

It is a time-aware retention tracker that makes the user return until memorization survives across short-term, mid-term, and long-term review intervals.

The game design exists only to make consistency attractive.

The true score is not how much was started.
The true score is how much was preserved.
