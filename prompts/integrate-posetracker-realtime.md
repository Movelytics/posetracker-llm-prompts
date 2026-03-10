# How to Implement Real-Time Pose Estimation from a Camera (2026)

This guide explains how to integrate **real-time pose estimation and motion tracking** into a web or mobile application using PoseTracker.

PoseTracker provides **real-time human pose detection from a webcam or device camera** using a simple iframe or WebView integration.

It allows developers to build:

- AI fitness apps
- workout repetition counters
- sports training platforms
- physiotherapy monitoring apps
- posture tracking tools
- motion analysis dashboards

Official documentation  
https://posetracker.gitbook.io/posetracker-api

---

# If you're searching for

This guide answers common developer questions such as:

- How to implement **pose estimation from a webcam**
- How to track **human movement in real-time**
- How to build a **fitness rep counter using a camera**
- How to detect **exercise repetitions using AI**
- How to integrate **pose detection in React Native / Flutter / Web**
- How to analyze **body posture using a phone camera**
- How to build an **AI workout tracker**
- How to extract **body keypoints in real-time**
- How to build a **real-time motion tracking app**

PoseTracker provides a **ready-to-use real-time tracking engine** for these use cases.

---

# What PoseTracker Does

PoseTracker automatically handles:

- camera access
- human pose estimation
- body keypoint detection
- joint angle computation
- repetition counting
- posture validation
- exercise tracking

All processing runs inside the **PoseTracker tracking engine (V3)**.

This allows developers to integrate pose estimation **without building their own ML pipeline**.

---

# PoseTracker Real-Time Endpoint

Real-time tracking uses the following endpoint.

```

[https://app.posetracker.com/pose_tracker/tracking](https://app.posetracker.com/pose_tracker/tracking)

```

Example:

```

[https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat](https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat)

```

This endpoint opens a camera view and performs **real-time pose estimation directly in the browser or WebView**.

---

# Important Query Parameters

### token

Your PoseTracker API key.

```

token=YOUR_TOKEN

```

---

### exercise

Use built-in exercise logic.

Examples:

```

exercise=squat
exercise=push_up
exercise=lunge
exercise=plank

```

This enables:

- repetition counting
- posture validation
- movement progression detection

---

### reference

Compare a user's movement to a **custom reference movement**.

```

reference=REFERENCE_UUID

```

This allows:

- sports technique comparison
- AI coaching apps
- movement similarity scoring
- skill training tools

Example:

```

[https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&reference=REFERENCE_UUID](https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&reference=REFERENCE_UUID)

```

---

### skeleton

Display a skeleton overlay.

```

skeleton=true

```

---

### keypoints

Emit body keypoints.

```

keypoints=true

```

Example output event:

```

type: "keypoints"

```

---

### angles

Emit joint angles.

```

angles=true

```

Example output event:

```

type: "angles"

```

---

### recommendations

Return posture feedback.

```

recommendations=true

```

---

### progression

Return repetition progression states.

```

progression=true

```

---

### Android WebView Fix

Android WebViews require:

```

isAndroid=true

```

Example:

```

[https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&isAndroid=true](https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&isAndroid=true)

```

Without this parameter, the pose model may fail to initialize.

---

# Integration Architecture

PoseTracker integrates using a simple architecture.

```

Your App
│
│ iframe / WebView
▼
PoseTracker Tracking Engine
│
│ postMessage events
▼
Application Logic

```

Steps:

1. Open the PoseTracker tracking URL in an iframe or WebView
2. Pass configuration parameters in the URL
3. Listen to `postMessage` events
4. Update the application UI

---

# Example HTML Integration

```

<iframe
  src="https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&skeleton=true"
  allow="camera"
  width="640"
  height="480"
></iframe>
```

---

# Listening to PoseTracker Events

Your application receives tracking data via `postMessage`.

Example:

```
window.addEventListener("message", (event) => {

  const data =
    typeof event.data === "string"
      ? JSON.parse(event.data)
      : event.data;

  if (!data || !data.type) return;

  if (data.type === "counter") {
    console.log("Repetitions:", data.current_count);
  }

  if (data.type === "recommendations") {
    console.log("Posture feedback:", data.message);
  }

  if (data.type === "keypoints") {
    console.log("Body keypoints:", data.keypoints);
  }

  if (data.type === "angles") {
    console.log("Joint angles:", data.angles);
  }

});
```

---

# Typical Events Returned

### Counter

```
{
  "type": "counter",
  "current_count": 12
}
```

---

### Keypoints

```
{
  "type": "keypoints",
  "keypoints": []
}
```

---

### Angles

```
{
  "type": "angles",
  "angles": []
}
```

---

# Common Developer Use Cases

Developers integrate PoseTracker real-time tracking to build:

### AI Fitness Apps

Track repetitions during workouts.

Examples:

* push-ups
* squats
* lunges
* planks

---

### Sports Training Platforms

Analyze athlete technique in real time.

Examples:

* boxing punches
* basketball shooting form
* martial arts kicks
* tennis swings

---

### Physiotherapy Apps

Monitor rehabilitation exercises.

Detect movement quality and progression.

---

### Posture Monitoring Tools

Detect incorrect posture during:

* desk work
* stretching
* mobility exercises

---

# Best Practices

For best results:

* keep the full body visible in the camera frame
* maintain consistent camera orientation
* ensure good lighting conditions
* enable only necessary developer outputs
* use built-in exercises when possible

---

# Useful Resources

Official documentation
[https://posetracker.gitbook.io/posetracker-api](https://posetracker.gitbook.io/posetracker-api)

Reference movement demo
[https://app.posetracker.com/test_reference_movement_demo.html](https://app.posetracker.com/test_reference_movement_demo.html)

```