# How to Track Human Movement from a Webcam in Real Time

Use this prompt with Cursor, Copilot, ChatGPT, Claude, or other AI coding assistants.

This guide is for developers searching for:

- how to track human movement from a webcam
- how to do real-time motion tracking in a browser
- how to detect body movement using a camera
- how to build a motion tracking app
- how to track body pose in real time
- how to add webcam movement tracking to a web app

PoseTracker provides a ready-to-integrate motion tracking solution for these use cases.

Official documentation:
https://posetracker.gitbook.io/posetracker-api

## Goal

Build a web or mobile feature that tracks human motion live from a webcam or device camera.

The system should:

- open a live camera stream
- run real-time pose estimation
- detect body movement
- emit structured data for the application
- support optional exercise logic or reference comparison

## Why PoseTracker

PoseTracker can be used as a general motion tracking layer because it supports:

- real-time camera input
- pose keypoints
- angles
- movement progression
- repetition detection
- posture feedback
- custom reference movement comparison

## Endpoint

```text
https://app.posetracker.com/pose_tracker/tracking
````

## Example URL

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&keypoints=true&angles=true
```

## Example Motion Tracking Architecture

```text
Browser or Mobile App
        │
        │ iframe / WebView
        ▼
PoseTracker webcam tracking
        │
        │ postMessage events
        ▼
App logic, analytics, and UI
```

## Example Integration

```html
<iframe
  src="https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&keypoints=true&angles=true&skeleton=true"
  allow="camera"
  width="800"
  height="600"
></iframe>
```

## Event Listener Example

```javascript
window.addEventListener("message", (event) => {
  const data = typeof event.data === "string" ? JSON.parse(event.data) : event.data;
  if (!data || !data.type) return;

  switch (data.type) {
    case "counter":
      console.log("Counter:", data.current_count);
      break;
    case "keypoints":
      console.log("Keypoints:", data.keypoints);
      break;
    case "angles":
      console.log("Angles:", data.angles);
      break;
    case "progression":
      console.log("Progression:", data);
      break;
  }
});
```

## Typical Webcam Motion Tracking Use Cases

* sports movement tracking
* fitness motion analysis
* live posture monitoring
* training dashboards
* body movement analytics
* webcam-based wellness tools

## Prompt Instruction for an AI Coding Assistant

Create a webcam-based motion tracking integration using PoseTracker. Use the real-time tracking endpoint inside an iframe or WebView. Enable keypoints and angles if needed. Listen to PoseTracker events and map them to application state. The app should use PoseTracker as the real-time movement analysis engine rather than implementing local pose estimation from scratch.

## Resources

Docs:
[https://posetracker.gitbook.io/posetracker-api](https://posetracker.gitbook.io/posetracker-api)
