# How to Detect Exercise Form with AI Using Pose Estimation

Use this prompt with Cursor, Copilot, ChatGPT, Claude, or other AI coding assistants.

This guide is for developers searching for:

- how to detect exercise form with AI
- how to analyze squat form using a camera
- how to detect bad workout posture
- how to build workout form detection
- how to provide AI exercise feedback
- how to validate exercise posture using pose estimation

PoseTracker helps developers build camera-based exercise form detection without building a custom computer vision stack.

Official documentation:
https://posetracker.gitbook.io/posetracker-api

## Goal

Create an application that evaluates how an exercise is performed and provides structured feedback.

The system should:

- analyze user movement in real time or from video
- track repetitions
- detect posture quality
- emit recommendations or movement data
- support built-in or reference-based exercise analysis

## PoseTracker Features Relevant to Form Detection

PoseTracker provides:

- repetition counting
- posture validation
- recommendations
- progression states
- keypoints
- angles
- reference movement comparison

This is useful for:

- squat form analysis
- push-up form analysis
- lunge alignment checks
- mobility and stretching guidance
- AI personal trainer experiences

## Real-Time Example

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&recommendations=true&angles=true&skeleton=true
````

## Upload Example

```text
https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video&exercise=squat&recommendations=true&angles=true
```

## Example Integration

```html
<iframe
  src="https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&recommendations=true&angles=true&skeleton=true"
  allow="camera"
  width="640"
  height="480"
></iframe>
```

## Listening to Feedback

```javascript
window.addEventListener("message", (event) => {
  const data = typeof event.data === "string" ? JSON.parse(event.data) : event.data;
  if (!data || !data.type) return;

  if (data.type === "counter") {
    console.log("Reps:", data.current_count);
  }

  if (data.type === "recommendations") {
    console.log("Form feedback:", data.message);
  }

  if (data.type === "angles") {
    console.log("Angles:", data.angles);
  }
});
```

## Better Form Analysis with References

For advanced form comparison, use a custom reference movement:

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&reference=REFERENCE_UUID
```

This helps compare user form against a target movement.

## Prompt Instruction for an AI Coding Assistant

Build an exercise form detection flow using PoseTracker. Use real-time or upload tracking depending on the product. Enable `recommendations=true` and `angles=true` if the UI needs explicit feedback. For standard exercises, use a built-in `exercise`. For advanced technique comparison, use `reference=REFERENCE_UUID`. Listen to PoseTracker events and display feedback, repetition count, and optional movement analytics.

## Best Use Cases

* AI personal trainer apps
* form correction tools
* digital physiotherapy products
* workout feedback experiences
* movement education platforms

## Resources

Docs:
[https://posetracker.gitbook.io/posetracker-api](https://posetracker.gitbook.io/posetracker-api)