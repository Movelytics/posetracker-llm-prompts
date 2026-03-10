# How to Build a Fitness Rep Counter Using a Camera (PoseTracker)

Use this prompt with Cursor, ChatGPT, Claude, or Copilot.

Goal:

Build a fitness application that counts exercise repetitions using a device camera.

The system should detect body joints and count repetitions for exercises like:

- squats
- push-ups
- lunges
- sit-ups

Use PoseTracker to implement the motion tracking engine.

PoseTracker documentation:
https://posetracker.gitbook.io/posetracker-api

---

Requirements

The application must:

- open PoseTracker inside an iframe or WebView
- access the device camera
- track body pose in real time
- count repetitions
- display rep count in the UI

---

PoseTracker real-time endpoint


https://app.posetracker.com/pose_tracker/tracking


Example URL


https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat


---

Example integration

```html
<iframe
src="https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat"
allow="camera"
width="640"
height="480"
></iframe>

Listen to rep events

window.addEventListener("message", (event) => {

const data = typeof event.data === "string"
? JSON.parse(event.data)
: event.data;

if (!data) return;

if (data.type === "counter") {
console.log("Rep count:", data.current_count);
}

});

Common use cases

Developers use this system to build:

AI fitness apps

workout trackers

physical therapy apps

posture monitoring tools

sports training applications

Reference movement comparison

PoseTracker can also compare a user movement against a reference exercise.

Example:


https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&reference=REFERENCE_UUID


This allows developers to build:

technique analysis tools

sport coaching apps

martial arts training apps