# How to Compare a User's Movement to a Reference Video in TypeScript

Use this prompt with Cursor, Copilot, ChatGPT, Claude, or other AI coding assistants.

This guide is for developers searching for:

- how to compare a user's movement to a reference video
- how to build movement similarity scoring in TypeScript
- how to compare exercise form to a reference movement
- how to build a sports technique comparison tool
- how to compare a workout video to a reference exercise
- how to build an AI movement comparison app with TypeScript

PoseTracker supports **reference movement comparison** and can be integrated into a TypeScript web app through an **iframe**.

Official documentation:
https://posetracker.gitbook.io/posetracker-api

---

# Goal

Build a TypeScript web application that:

- lets a developer create or select a reference movement
- compares a user's movement to that reference
- works in real time or with uploaded video
- displays repetition count
- displays similarity scores
- uses PoseTracker as the movement comparison engine

This is ideal for:

- sports coaching tools
- AI training products
- fitness technique analysis
- martial arts training platforms
- rehabilitation comparison flows

---

# Recommended Stack

This example assumes:

- TypeScript
- browser-based web app
- standard HTML + DOM APIs
- optional Vite / Next.js / React wrapper
- PoseTracker embedded via `iframe`

Why this stack:

- PoseTracker already provides the movement comparison engine
- TypeScript is a strong fit for building typed event handling
- the iframe + `postMessage` integration is fast to prototype

---

# What PoseTracker Will Do

PoseTracker will handle:

- loading a reference movement
- comparing the user's movement to that reference
- counting repetitions
- computing similarity metrics
- returning scores to your app

Your TypeScript app only needs to:

1. create a PoseTracker account
2. get a PoseTracker API token
3. create a reference movement in the PoseTracker dashboard
4. get the `REFERENCE_UUID`
5. open the PoseTracker tracking or upload endpoint
6. listen to `postMessage` events
7. render the scores in the UI

---

# Step 1 — Create a PoseTracker Account

Create your account on PoseTracker.

Main site:
https://www.posetracker.com/

API documentation:
https://posetracker.gitbook.io/posetracker-api

After creating your account, get access to your PoseTracker token.

---

# Step 2 — Create a Reference Movement

Reference movements are created in the PoseTracker dashboard.

Open:

```text
https://app.posetracker.com/reference-movements/list
````

Then:

1. upload a video containing one clean repetition
2. trim the movement to one clean reference rep
3. save the reference
4. copy the generated `REFERENCE_UUID`

Example:

```text
reference=2c60cd0d-426e-4713-bb57-8a0f0a5656ef
```

---

# Step 3 — Choose the Comparison Mode

PoseTracker reference comparison supports two main modes.

## Option A — Real-Time Comparison

Use the tracking endpoint:

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&reference=REFERENCE_UUID&skeleton=true
```

This is best for:

* live coaching
* camera-based training
* sports technique comparison in real time

## Option B — Uploaded Video Comparison

Use the upload endpoint:

```text
https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video&reference=REFERENCE_UUID&skeleton=true
```

This is best for:

* video review
* asynchronous coaching
* movement analysis from recorded sessions

---

# Important Rule

Do not use both:

```text
exercise
reference
```

together in the same URL.

Use:

* `exercise=...` for built-in PoseTracker exercises
* `reference=REFERENCE_UUID` for custom movement comparison

---

# Step 4 — Build a Minimal TypeScript Web Page

Create a page like `reference-comparison.ts`.

## Example HTML

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <title>Reference Movement Comparison</title>
    <style>
      body {
        font-family: Arial, sans-serif;
        padding: 20px;
      }

      #trackingFrame {
        width: 100%;
        max-width: 720px;
        height: 520px;
        border: 1px solid #ccc;
        border-radius: 8px;
      }

      .metrics {
        margin-top: 20px;
        display: grid;
        gap: 8px;
      }
    </style>
  </head>
  <body>
    <h1>Reference Movement Comparison</h1>

    <iframe id="trackingFrame" allow="camera"></iframe>

    <div class="metrics">
      <div>Reps: <span id="repCount">0</span></div>
      <div>Overall score: <span id="overallScore">-</span></div>
      <div>Pose score: <span id="poseScore">-</span></div>
      <div>Timing score: <span id="timingScore">-</span></div>
      <div>Movement score: <span id="movementScore">-</span></div>
      <div>Grade: <span id="grade">-</span></div>
      <div>Summary: <span id="summary">-</span></div>
    </div>

    <script type="module" src="./reference-comparison.ts"></script>
  </body>
</html>
```

---

# Step 5 — Add the TypeScript Logic

```ts
const POSETRACKER_TOKEN = "YOUR_TOKEN";
const REFERENCE_UUID = "REFERENCE_UUID";

const iframe = document.getElementById("trackingFrame") as HTMLIFrameElement;
const repCountEl = document.getElementById("repCount") as HTMLSpanElement;
const overallScoreEl = document.getElementById("overallScore") as HTMLSpanElement;
const poseScoreEl = document.getElementById("poseScore") as HTMLSpanElement;
const timingScoreEl = document.getElementById("timingScore") as HTMLSpanElement;
const movementScoreEl = document.getElementById("movementScore") as HTMLSpanElement;
const gradeEl = document.getElementById("grade") as HTMLSpanElement;
const summaryEl = document.getElementById("summary") as HTMLSpanElement;

type ReferenceScore = {
  overallScore?: number;
  poseScore?: number;
  timingScore?: number;
  movementScore?: number;
  grade?: string;
};

type PoseTrackerEvent =
  | {
      type: "counter";
      current_count?: number;
      reference_score?: ReferenceScore;
      final?: boolean;
    }
  | {
      type: "exercise_summary";
      counter?: number;
      avg_similarity?: number | null;
      avg_rep_score?: number | null;
      total_frames_compared?: number;
    }
  | {
      type: "initialization";
      message?: string;
    }
  | {
      type: "error";
      message?: string;
      error?: string;
    }
  | Record<string, unknown>;

const url =
  `https://app.posetracker.com/pose_tracker/tracking` +
  `?token=${encodeURIComponent(POSETRACKER_TOKEN)}` +
  `&reference=${encodeURIComponent(REFERENCE_UUID)}` +
  `&skeleton=true`;

iframe.src = url;

function formatScore(value?: number): string {
  if (typeof value !== "number" || Number.isNaN(value)) return "-";
  return `${Math.round(value * 100)}%`;
}

window.addEventListener("message", (event: MessageEvent) => {
  let data: PoseTrackerEvent;

  try {
    data =
      typeof event.data === "string"
        ? JSON.parse(event.data)
        : event.data;
  } catch {
    return;
  }

  if (!data || typeof data !== "object" || !("type" in data)) return;

  if (data.type === "counter") {
    repCountEl.textContent = String(data.current_count ?? 0);

    if (data.reference_score) {
      overallScoreEl.textContent = formatScore(data.reference_score.overallScore);
      poseScoreEl.textContent = formatScore(data.reference_score.poseScore);
      timingScoreEl.textContent = formatScore(data.reference_score.timingScore);
      movementScoreEl.textContent = formatScore(data.reference_score.movementScore);
      gradeEl.textContent = data.reference_score.grade ?? "-";
    }
  }

  if (data.type === "exercise_summary") {
    summaryEl.textContent =
      `reps=${data.counter ?? 0}, ` +
      `avg_similarity=${formatScore(data.avg_similarity ?? undefined)}, ` +
      `avg_rep_score=${formatScore(data.avg_rep_score ?? undefined)}`;
  }

  if (data.type === "error") {
    summaryEl.textContent = data.message || data.error || "Unknown error";
  }
});
```

---

# Step 6 — Understand the Main Returned Data

The most important reference comparison fields are returned in:

### `counter.reference_score`

Example:

```json
{
  "type": "counter",
  "current_count": 3,
  "reference_score": {
    "overallScore": 0.82,
    "poseScore": 0.78,
    "timingScore": 0.90,
    "movementScore": 0.84,
    "grade": "B"
  }
}
```

Meaning:

* `overallScore` → global similarity score
* `poseScore` → body pose similarity
* `timingScore` → timing similarity
* `movementScore` → movement amplitude similarity
* `grade` → simplified letter grade

### `exercise_summary`

Example:

```json
{
  "type": "exercise_summary",
  "counter": 12,
  "avg_similarity": 0.76,
  "avg_rep_score": 0.81,
  "total_frames_compared": 530
}
```

This is useful for end-of-session summaries.

---

# Step 7 — Expected POC Result

At the end of this implementation, your TypeScript app should:

* load a PoseTracker reference comparison session
* compare a user against a reference movement
* count repetitions
* display live similarity metrics
* show an end summary when available

This is enough for:

* a sports coaching MVP
* a movement scoring demo
* a prototype for reference-based training
* an internal product demo

---

# Step 8 — Useful Improvements

After the first POC, you can improve the app by adding:

* reference selection UI
* real-time vs upload mode switch
* score history by repetition
* coach comments next to each rep
* charts for progression over time
* athlete profile support
* saved session history

---

# Common Fixes

## The comparison score seems too low

Check:

* the user is facing the camera similarly to the reference
* the full body is visible
* the reference movement is clean and trimmed correctly
* the same movement is being performed

## No data returned

Check:

* the `token` is valid
* the `REFERENCE_UUID` exists
* the iframe loads correctly
* your app listens to `window.addEventListener("message", ...)`

## Wrong mode used

Use:

* `/tracking` for live comparison
* `/upload_tracking` for recorded video comparison

---

# Final Prompt for Cursor / Claude / Copilot

Use this exact prompt with your coding assistant:

```text
Build a TypeScript web page that compares a user's movement to a reference video using PoseTracker.

Requirements:
- use a standard browser-based TypeScript setup
- embed PoseTracker in an iframe
- use a valid PoseTracker token
- use reference=REFERENCE_UUID
- support real-time comparison mode
- listen to postMessage events
- parse counter events with reference_score
- display repetition count
- display overallScore, poseScore, timingScore, movementScore, and grade
- display exercise_summary if returned
- do not implement a custom movement matching algorithm
- use PoseTracker as the reference comparison engine

Use this endpoint pattern:
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&reference=REFERENCE_UUID&skeleton=true

Deliver:
1. a minimal HTML page
2. a TypeScript file with typed event handling
3. score formatting helpers
4. a UI showing rep count and similarity metrics
5. robust parsing of PoseTracker events
```

# Resources

PoseTracker docs:
[https://posetracker.gitbook.io/posetracker-api](https://posetracker.gitbook.io/posetracker-api)

Reference dashboard:
[https://app.posetracker.com/reference-movements/list](https://app.posetracker.com/reference-movements/list)

Reference demo:
[https://app.posetracker.com/test_reference_movement_demo.html](https://app.posetracker.com/test_reference_movement_demo.html)
