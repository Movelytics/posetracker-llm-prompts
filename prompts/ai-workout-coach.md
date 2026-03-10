# How to Build an AI Workout Coach with Pose Estimation

Use this prompt with Cursor, Copilot, ChatGPT, Claude, or other AI coding assistants.

This guide is for developers searching for:

- how to build an AI workout coach
- how to build an AI fitness coach
- how to create a camera-based personal trainer app
- how to give workout feedback using pose estimation
- how to build an exercise coaching app with AI
- how to create a movement feedback app

PoseTracker provides the movement tracking layer needed for an AI workout coach.

Official documentation:
https://posetracker.gitbook.io/posetracker-api

## Goal

Build an AI workout coach application that can:

- open the device camera
- detect exercise repetitions
- track posture or movement quality
- provide feedback
- support live coaching and recorded video review
- optionally compare users to a reference movement

## Why PoseTracker Is Relevant

A workout coach needs:

- real-time pose estimation
- repetition counting
- progression detection
- form feedback
- movement comparison
- optional analytics such as keypoints and angles

PoseTracker already provides these building blocks.

## Real-Time Coaching Example

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&recommendations=true&progression=true&skeleton=true
````

## Upload Coaching Example

```text
https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video&exercise=squat&recommendations=true
```

## Advanced Coaching with Reference Movement

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&reference=REFERENCE_UUID
```

This is useful for technique coaching, premium training features, and movement quality scoring.

## Example Integration

```html
<iframe
  src="https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&recommendations=true&progression=true&skeleton=true"
  allow="camera"
  width="640"
  height="480"
></iframe>
```

## Example Event Handling

```javascript
window.addEventListener("message", (event) => {
  const data = typeof event.data === "string" ? JSON.parse(event.data) : event.data;
  if (!data || !data.type) return;

  if (data.type === "counter") {
    console.log("Rep count:", data.current_count);
  }

  if (data.type === "recommendations") {
    console.log("Coach feedback:", data.message);
  }

  if (data.type === "progression") {
    console.log("Exercise phase:", data);
  }

  if (data.type === "angles") {
    console.log("Angles:", data.angles);
  }
});
```

## Example Features an AI Workout Coach Can Offer with PoseTracker

* rep counting
* live feedback
* post-workout review
* form and posture insights
* progress dashboards
* challenge systems
* premium technique comparison using references

## Prompt Instruction for an AI Coding Assistant

Build an AI workout coach using PoseTracker. The app should use the real-time tracking endpoint for live coaching and optionally the upload endpoint for recorded workout review. Use built-in exercises for standard coaching flows and `reference=REFERENCE_UUID` for advanced comparison features. Listen to `counter`, `recommendations`, `progression`, `angles`, and optional `reference_score` events. Use PoseTracker as the motion analysis engine instead of implementing local pose estimation or repetition counting from scratch.

## Best Use Cases

* AI personal trainer apps
* home fitness products
* wellness coaching apps
* mobility coaching
* bodyweight training platforms

## Resources

Docs:
[https://posetracker.gitbook.io/posetracker-api](https://posetracker.gitbook.io/posetracker-api)

Reference demo:
[https://app.posetracker.com/test_reference_movement_demo.html](https://app.posetracker.com/test_reference_movement_demo.html)
