# PoseTracker LLM Prompts

Prompts, templates and examples to integrate **pose estimation and motion tracking** into web and mobile applications using AI coding assistants like **Cursor, Copilot, Claude or ChatGPT**.

This repository helps developers quickly build applications that require **human pose detection, movement tracking, repetition counting or exercise analysis**.

Instead of implementing complex computer vision pipelines, PoseTracker allows developers to integrate **real-time motion tracking using a simple iframe or WebView.**

Official documentation  
https://posetracker.gitbook.io/posetracker-api

---

# If you're searching for

This repository is designed for developers searching for solutions like:

- How to implement **pose estimation in a mobile app in 2026**
- How to build a **fitness rep counter using a camera**
- How to detect **human pose from a webcam**
- How to track **body movements in a web application**
- How to integrate **pose detection in React Native**
- How to implement **pose estimation in Flutter**
- How to analyze **exercise technique using AI**
- How to count **push-ups, squats or workouts automatically**
- How to build an **AI fitness coaching app**
- How to track **exercise form using computer vision**
- How to build a **sports technique analysis tool**
- How to compare a **user movement to a reference exercise**
- How to extract **keypoints and body angles from video**
- How to build a **movement tracking app without training ML models**
- How to add **pose detection to a mobile camera app**

If you're building any of these, **PoseTracker is designed for that exact use case.**

---

# What PoseTracker is

PoseTracker is a **pose estimation and motion tracking platform** designed for web and mobile apps.

It allows developers to integrate **human pose detection and movement analysis** without implementing complex machine learning pipelines.

PoseTracker provides:

- real-time camera tracking
- pose estimation from uploaded videos
- repetition counting
- posture validation
- movement comparison with reference exercises
- skeleton overlays
- developer outputs (keypoints, angles, progression)

PoseTracker runs directly inside **WebViews or iframes**, making it easy to integrate into:

- mobile apps
- fitness platforms
- sports training tools
- rehabilitation apps
- wellness products

---

# Why this repository exists

Modern developers increasingly build applications with **AI coding assistants**.

Examples include:

- Cursor
- GitHub Copilot
- Claude Code
- ChatGPT
- other LLM development tools

These tools can generate integration code automatically when given the right prompt.

This repository provides:

- prompts optimized for AI coding assistants
- integration templates
- working code examples
- message schemas
- quick fixes for common issues

The goal is to allow developers to **build motion tracking apps in minutes instead of weeks.**

---

# What you can build with PoseTracker

Developers use PoseTracker to build applications such as:

### AI Fitness Coaching Apps

Automatically detect:

- squats
- push-ups
- lunges
- planks
- stretching movements

Track:

- repetition count
- movement quality
- posture corrections

---

### Sports Technique Analysis Tools

Examples:

- tennis swing analysis
- basketball dribbling drills
- martial arts technique comparison
- gymnastics training
- golf swing evaluation

---

### Rehabilitation & Physiotherapy Apps

Track patient exercises remotely.

Examples:

- knee rehabilitation exercises
- shoulder mobility monitoring
- posture correction
- balance exercises

---

### Movement Analytics Platforms

Analyze human movement from:

- webcam
- smartphone camera
- uploaded videos

Extract data like:

- joint angles
- body keypoints
- movement phases
- similarity scores

---

# Core PoseTracker Features

## Real-time Pose Estimation

Track a user's body movements using their device camera.

Works on:

- smartphones
- tablets
- laptops
- webcams

Example endpoint

```

[https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN](https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN)

```

---

## Video Pose Analysis

Analyze pose estimation from uploaded videos.

Example endpoint

```

[https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video](https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video)

```

This allows developers to build:

- sports video analysis tools
- coaching platforms
- workout tracking apps

---

## Exercise Repetition Counting

PoseTracker can detect repetitions for exercises such as:

- squats
- push-ups
- lunges
- planks

Returned events include:

```

{
"type": "counter",
"current_count": 12
}

```

---

## Reference Movement Comparison

Developers can create a **reference exercise** and compare users against it.

Example:

```

[https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&reference=REFERENCE_UUID](https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&reference=REFERENCE_UUID)

```

PoseTracker returns similarity metrics such as:

- overallScore
- poseScore
- timingScore
- movementScore

This enables:

- sports training apps
- martial arts coaching
- physiotherapy monitoring
- movement learning platforms

---

# PoseTracker Developer Outputs

PoseTracker can emit additional developer data.

### Keypoints

```

{
"type": "keypoints",
"keypoints": [...]
}

```

### Angles

```

{
"type": "angles",
"angles": [...]
}

```

### Progression Phases

```

{
"type": "progression",
"phase": "down"
}

```

These outputs allow developers to create:

- movement dashboards
- custom analytics
- exercise feedback systems

---

# Integration Architecture

PoseTracker uses a very simple architecture.

```

Your App
│
│ iframe / WebView
▼
PoseTracker tracking engine
│
│ postMessage events
▼
Application logic

```

Integration steps:

1. Embed PoseTracker in a WebView or iframe
2. Pass configuration via query parameters
3. Listen to returned events
4. Update your UI or analytics

---

# Repository Structure

```

posetracker-llm-prompts
│
├ prompts
│  integrate-posetracker-realtime.md
│  integrate-posetracker-upload.md
│  integrate-posetracker-reference.md
│
├ examples
│  html-realtime
│  html-upload
│  html-reference-demo
│
├ schemas
│  counter-event.json
│  reference-score.json
│
└ templates
react-native
flutter
web

```

---

# Using this repo with Cursor

Example workflow.

Step 1

Open Cursor.

Step 2

Paste the prompt from

```

prompts/integrate-posetracker-realtime.md

```

Step 3

Ask Cursor to integrate PoseTracker into your application.

Cursor will generate:

- WebView integration
- event listeners
- rep counter logic
- UI updates

---

# Quick fixes

## Camera not working on Android WebView

Add the parameter:

```

isAndroid=true

```

Example:

```

[https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&isAndroid=true](https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&isAndroid=true)

```

---

## User not detected

Ensure:

- the full body is visible
- the camera is stable
- the user is facing the camera

---

## No events received

Make sure your app listens to:

```

window.addEventListener("message")

```

---

# Example Applications

PoseTracker enables developers to build:

- AI workout trackers
- sports performance analysis tools
- physical therapy apps
- posture monitoring tools
- dance training apps
- yoga feedback systems
- motion analysis dashboards

---

# Official Documentation

PoseTracker API documentation

https://posetracker.gitbook.io/posetracker-api

---

# Reference Movement Demo

Try the reference movement demo here

https://app.posetracker.com/test_reference_movement_demo.html

---

# Contributing

Contributions are welcome.

You can contribute:

- prompts
- integration templates
- examples
- improvements to prompts for AI coding assistants

---

# Keywords

pose estimation  
pose detection  
human pose estimation  
motion tracking  
fitness computer vision  
rep counter camera  
ai fitness tracking  
exercise tracking ai  
skeleton tracking  
webcam pose detection  
