# How to Analyze a Video with Pose Estimation (2026)

This guide shows how to integrate **PoseTracker video pose estimation** into a web or mobile application.

PoseTracker allows developers to analyze **human movement from uploaded videos or images** using pose estimation.

This is commonly used for:

- AI fitness apps
- sports technique analysis
- physiotherapy apps
- posture analysis tools
- movement analytics platforms
- coaching and training software

---

# If you're searching for

This guide answers common developer questions such as:

- How to implement **pose estimation from a video upload**
- How to build a **fitness rep counter from a recorded workout**
- How to analyze **sports movement from a video**
- How to detect **human pose from a recorded video**
- How to extract **body keypoints from an image**
- How to track **exercise repetitions from a video**
- How to build an **AI fitness coach using video uploads**
- How to analyze **movement quality from a workout video**
- How to detect **human skeleton keypoints in JavaScript**

PoseTracker provides a **simple API and WebView integration** for these use cases.

Official documentation  
https://posetracker.gitbook.io/posetracker-api

---

# What PoseTracker does

PoseTracker automatically performs:

- human pose detection
- body keypoint extraction
- joint angle computation
- repetition counting
- movement progression detection
- posture validation

All computation runs **directly inside the PoseTracker iframe**, making integration extremely simple.

---

# Upload Tracking Endpoint

PoseTracker provides a dedicated endpoint to analyze uploaded media.

```

[https://app.posetracker.com/pose_tracker/upload_tracking](https://app.posetracker.com/pose_tracker/upload_tracking)

```

Example:

```

[https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video](https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video)

```

---

# Supported Media

PoseTracker supports two types of media inputs.

---

# 1. Video Movement Analysis

```

source=video

```

This mode analyzes **movement across time**.

Typical use cases:

- analyzing workout videos
- detecting repetitions from recorded exercises
- reviewing sports technique
- evaluating rehabilitation exercises
- tracking dance movements

Example:

```

[https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video&exercise=squat](https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video&exercise=squat)

```

---

# 2. Image Pose Detection

```

source=image

```

This mode extracts **pose keypoints from a single frame**.

Typical use cases:

- posture analysis
- skeleton detection
- pose classification
- biomechanics research

Example:

```

[https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=image](https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=image)

```

---

# Important Query Parameters

## token

Your PoseTracker API key.

```

token=YOUR_API_KEY

```

---

## source

Defines the uploaded media type.

```

source=video
source=image

```

---

## exercise

Use PoseTracker built-in exercise tracking.

Examples:

```

exercise=squat
exercise=push_up
exercise=lunge
exercise=plank

```

Example URL

```

[https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video&exercise=push_up](https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video&exercise=push_up)

```

---

## reference

Compare the uploaded video to a **reference movement**.

```

reference=REFERENCE_UUID

```

Example:

```

[https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video&reference=REFERENCE_UUID](https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video&reference=REFERENCE_UUID)

```

This enables:

- technique comparison
- movement similarity scoring
- sports training feedback
- AI coaching tools

Important:

```

exercise and reference cannot be used together

```

---

# Optional Developer Outputs

PoseTracker can emit additional data.

## Skeleton Overlay

```

skeleton=true

```

Displays a skeleton overlay on the analyzed video.

---

## Keypoints

```

keypoints=true

```

Returns body keypoints.

Example output:

```

type: "keypoints"

```

---

## Angles

```

angles=true

```

Returns computed joint angles.

Example output:

```

type: "angles"

```

---

# Example HTML Integration

PoseTracker can be integrated using a simple iframe.

```

<iframe
  src="https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video&skeleton=true"
  width="640"
  height="480"
></iframe>
```

---

# Listening to PoseTracker Events

Your application receives pose data via `postMessage`.

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

  if (data.type === "exercise_summary") {
    console.log("Exercise summary:", data);
  }

  if (data.type === "keypoints") {
    console.log("Keypoints:", data.keypoints);
  }

  if (data.type === "angles") {
    console.log("Angles:", data.angles);
  }

});
```

---

# Typical Outputs

Counter example

```
{
 "type": "counter",
 "current_count": 8
}
```

Exercise summary example

```
{
 "type": "exercise_summary",
 "counter": 12
}
```

Keypoints example

```
{
 "type": "keypoints",
 "keypoints": [...]
}
```

---

# Common Developer Use Cases

Developers use PoseTracker upload tracking to build:

### AI Fitness Coaching Apps

Users upload their workout videos.

The system detects:

* repetitions
* posture quality
* training progress

---

### Sports Technique Analysis

Analyze athlete movements from recorded videos.

Examples:

* tennis swing analysis
* basketball shooting analysis
* martial arts technique review

---

### Rehabilitation Monitoring

Patients upload rehabilitation exercises.

The system analyzes movement quality.

---

### Motion Analytics Platforms

Extract pose data from videos for research or biomechanics analysis.

---

# Official Documentation

Full documentation

[https://posetracker.gitbook.io/posetracker-api](https://posetracker.gitbook.io/posetracker-api)