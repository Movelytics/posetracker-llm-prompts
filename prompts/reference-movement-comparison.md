# How to Compare a User’s Movement to a Reference Exercise

Use this prompt with Cursor, Copilot, ChatGPT, Claude, or other AI coding assistants.

This guide is for developers searching for:

- how to compare a user's movement to a reference exercise
- how to analyze movement similarity with AI
- how to compare sports technique to a reference
- how to build a movement similarity scoring tool
- how to compare a workout to a reference video
- how to build an AI coaching app with reference movement analysis

PoseTracker supports custom reference movement comparison in real time and from uploaded videos.

Official documentation:
https://posetracker.gitbook.io/posetracker-api

## Goal

Build an application that compares a user's movement to a predefined reference movement.

The application should:

- let developers create reference movements
- use a `REFERENCE_UUID`
- compare users to the reference live or from uploaded video
- return similarity scores
- count repetitions

## Step 1 — Create a Reference Movement

Reference movements are created in the dashboard:

```text
https://app.posetracker.com/reference-movements/list
````

Developers upload one clean repetition and receive a `REFERENCE_UUID`.

## Step 2 — Real-Time Comparison

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&reference=REFERENCE_UUID
```

## Step 3 — Upload Comparison

```text
https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video&reference=REFERENCE_UUID
```

## Important Rule

Do not use `exercise` and `reference` together.

Use:

* `exercise=...` for built-in PoseTracker exercise logic
* `reference=REFERENCE_UUID` for custom movement comparison

## Example Integration

```html
<iframe
  src="https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&reference=REFERENCE_UUID&skeleton=true"
  allow="camera"
  width="640"
  height="480"
></iframe>
```

## Listening to Similarity Scores

```javascript
window.addEventListener("message", (event) => {
  const data = typeof event.data === "string" ? JSON.parse(event.data) : event.data;
  if (!data || !data.type) return;

  if (data.type === "counter" && data.reference_score) {
    console.log("Rep:", data.current_count);
    console.log("Overall:", data.reference_score.overallScore);
    console.log("Pose:", data.reference_score.poseScore);
    console.log("Timing:", data.reference_score.timingScore);
    console.log("Movement:", data.reference_score.movementScore);
    console.log("Grade:", data.reference_score.grade);
  }

  if (data.type === "exercise_summary") {
    console.log("Summary:", data);
  }
});
```

## Example Reference Score

```json
{
  "type": "counter",
  "current_count": 1,
  "reference_score": {
    "overallScore": 0.87,
    "poseScore": 0.82,
    "timingScore": 0.91,
    "movementScore": 0.88,
    "grade": "A"
  }
}
```

## Typical Use Cases

* sports technique analysis
* martial arts training
* skill learning apps
* gymnastics training
* dance coaching
* physiotherapy comparison workflows

## Prompt Instruction for an AI Coding Assistant

Build a movement comparison feature using PoseTracker reference tracking. First explain how the developer creates a reference movement in the PoseTracker dashboard. Then integrate either the real-time or upload endpoint using `reference=REFERENCE_UUID`. Listen to `counter` events containing `reference_score`, and if available, `exercise_summary`. Use the similarity scores to drive UI, analytics, or feedback. Do not build a custom movement matching algorithm locally; use PoseTracker as the comparison engine.

## Resources

Docs:
[https://posetracker.gitbook.io/posetracker-api](https://posetracker.gitbook.io/posetracker-api)

Reference dashboard:
[https://app.posetracker.com/reference-movements/list](https://app.posetracker.com/reference-movements/list)

Reference demo:
[https://app.posetracker.com/test_reference_movement_demo.html](https://app.posetracker.com/test_reference_movement_demo.html)