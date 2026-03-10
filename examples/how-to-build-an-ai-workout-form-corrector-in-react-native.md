# How to Build an AI Workout Form Corrector in React Native

Use this prompt with Cursor, Copilot, ChatGPT, Claude, or other AI coding assistants.

This guide is for developers searching for:

- how to build an AI workout form corrector in React Native
- how to build an exercise form correction app with React Native
- how to detect bad squat form in a mobile app
- how to build a pose estimation fitness app in React Native
- how to create an AI personal trainer app with camera feedback
- how to give workout posture feedback in a React Native app

PoseTracker can be integrated into React Native through a **WebView** to provide real-time exercise tracking, posture feedback, angles, and progression events.

Official documentation:
https://posetracker.gitbook.io/posetracker-api

---

# Goal

Build a React Native app that:

- opens the device camera
- analyzes a user performing an exercise in real time
- provides posture or form feedback
- displays repetition count
- optionally displays joint angles
- uses PoseTracker as the exercise form correction engine

This is ideal for:

- AI personal trainer apps
- workout coaching apps
- home fitness products
- posture correction tools
- digital physiotherapy apps

---

# Recommended Stack

This example assumes:

- React Native
- Expo
- `react-native-webview`
- `expo-camera`

Why this stack:

- `react-native-webview` is the simplest way to embed PoseTracker
- `expo-camera` is useful to request camera permission before loading PoseTracker
- PoseTracker handles the movement analysis logic

---

# What PoseTracker Will Do

PoseTracker will handle:

- real-time pose estimation
- repetition counting
- posture validation
- recommendation events
- progression events
- optional angle computation

Your React Native app only needs to:

1. create a PoseTracker account
2. get a PoseTracker API token
3. open the PoseTracker tracking URL in a WebView
4. enable the right parameters
5. listen to PoseTracker events
6. update the UI with feedback

---

# Step 1 — Create a PoseTracker Account

Create your account on PoseTracker.

Main site:
https://www.posetracker.com/

API documentation:
https://posetracker.gitbook.io/posetracker-api

After creating your account, get access to your PoseTracker token.

---

# Step 2 — Get Your PoseTracker Token

You need a PoseTracker API token to use the tracking endpoint.

You will use it in a URL like this:

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat
````

Replace:

* `YOUR_TOKEN` with your real PoseTracker token

---

# Step 3 — Install the Required Packages

Install WebView and Expo Camera:

```bash
npx expo install react-native-webview expo-camera
```

---

# Step 4 — Add iOS Camera Permission

In `app.json`, add:

```json
{
  "expo": {
    "ios": {
      "infoPlist": {
        "NSCameraUsageDescription": "This app uses the camera for exercise form analysis."
      }
    }
  }
}
```

This is required so iOS allows camera access.

---

# Step 5 — Build the PoseTracker URL

Use the real-time tracking endpoint:

```text
https://app.posetracker.com/pose_tracker/tracking
```

For a squat form correction flow, use:

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&skeleton=true&recommendations=true&angles=true&progression=true
```

Useful parameters:

* `token=YOUR_TOKEN`
* `exercise=squat`
* `skeleton=true`
* `recommendations=true`
* `angles=true`
* `progression=true`

If you are running inside an **Android WebView**, you must also add:

```text
isAndroid=true
```

Example Android URL:

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&skeleton=true&recommendations=true&angles=true&progression=true&isAndroid=true
```

---

# Step 6 — Create the React Native Screen

Create a screen like `WorkoutFormCorrectorScreen.tsx`.

```tsx
import React, { useEffect, useMemo, useState } from "react";
import { Alert, Platform, SafeAreaView, StyleSheet, Text, View } from "react-native";
import { WebView } from "react-native-webview";
import { useCameraPermissions } from "expo-camera";

const POSETRACKER_TOKEN = "YOUR_TOKEN";

export default function WorkoutFormCorrectorScreen() {
  const [permission, requestPermission] = useCameraPermissions();
  const [repCount, setRepCount] = useState(0);
  const [feedback, setFeedback] = useState("Start when ready");
  const [phase, setPhase] = useState("idle");
  const [angles, setAngles] = useState<any>(null);

  useEffect(() => {
    if (permission && !permission.granted) {
      requestPermission();
    }
  }, [permission?.granted]);

  const poseTrackerUrl = useMemo(() => {
    const base = "https://app.posetracker.com/pose_tracker/tracking";
    const params = new URLSearchParams({
      token: POSETRACKER_TOKEN,
      exercise: "squat",
      skeleton: "true",
      recommendations: "true",
      angles: "true",
      progression: "true",
    });

    if (Platform.OS === "android") {
      params.set("isAndroid", "true");
    }

    return `${base}?${params.toString()}`;
  }, []);

  if (!permission) {
    return (
      <SafeAreaView style={styles.centered}>
        <Text>Loading camera permission...</Text>
      </SafeAreaView>
    );
  }

  if (!permission.granted) {
    return (
      <SafeAreaView style={styles.centered}>
        <Text>Camera permission is required to analyze exercise form.</Text>
        <Text onPress={requestPermission} style={styles.link}>
          Grant camera access
        </Text>
      </SafeAreaView>
    );
  }

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>AI Workout Form Corrector</Text>
        <Text style={styles.metric}>Reps: {repCount}</Text>
        <Text style={styles.metric}>Phase: {phase}</Text>
        <Text style={styles.feedback}>Feedback: {feedback}</Text>
        <Text style={styles.angles}>
          Angles: {angles ? JSON.stringify(angles) : "No angle data yet"}
        </Text>
      </View>

      <WebView
        source={{ uri: poseTrackerUrl }}
        javaScriptEnabled
        domStorageEnabled
        originWhitelist={["*"]}
        allowsInlineMediaPlayback
        mediaPlaybackRequiresUserAction={false}
        onMessage={(event) => {
          try {
            const data =
              typeof event.nativeEvent.data === "string"
                ? JSON.parse(event.nativeEvent.data)
                : event.nativeEvent.data;

            if (!data || !data.type) return;

            if (data.type === "counter") {
              setRepCount(data.current_count ?? 0);
            }

            if (data.type === "recommendations" && data.message) {
              setFeedback(data.message);
            }

            if (data.type === "progression") {
              setPhase(data.phase || data.state || "moving");
            }

            if (data.type === "angles") {
              setAngles(data.angles || data);
            }
          } catch (error) {
            console.log("PoseTracker message parse error:", error);
          }
        }}
        onError={() => {
          Alert.alert("Error", "Failed to load PoseTracker.");
        }}
        style={styles.webview}
        {...(Platform.OS === "ios" && {
          mediaCapturePermissionGrantType: "grant" as const,
        })}
      />
    </SafeAreaView>
  );
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: "#fff",
  },
  centered: {
    flex: 1,
    alignItems: "center",
    justifyContent: "center",
    padding: 24,
  },
  header: {
    padding: 16,
    gap: 8,
  },
  title: {
    fontSize: 24,
    fontWeight: "700",
  },
  metric: {
    fontSize: 16,
  },
  feedback: {
    fontSize: 15,
    color: "#444",
  },
  angles: {
    fontSize: 13,
    color: "#666",
  },
  link: {
    marginTop: 12,
    color: "#2563eb",
    fontWeight: "600",
  },
  webview: {
    flex: 1,
  },
});
```

---

# Step 7 — Understand the Main Events

The most useful events for an exercise form corrector are:

### `counter`

Used to update the repetition count.

Example:

```json
{
  "type": "counter",
  "current_count": 6
}
```

### `recommendations`

Used to display form feedback.

Example:

```json
{
  "type": "recommendations",
  "message": "Keep your back straighter"
}
```

### `progression`

Used to identify movement phase.

Example:

```json
{
  "type": "progression",
  "phase": "down"
}
```

### `angles`

Used to display or log joint angle data.

Example:

```json
{
  "type": "angles",
  "angles": {
    "left_knee": 92,
    "right_knee": 89
  }
}
```

---

# Step 8 — Expected POC Result

At the end of this implementation, your React Native app should:

* open a live camera screen
* track the exercise in real time
* show repetition count
* show posture feedback
* show optional angle data
* work as a basic AI workout form correction POC

This is enough for:

* internal demos
* MVPs
* investor demos
* beta testing
* validating AI coaching use cases

---

# Step 9 — Useful Improvements

After the first POC, you can improve the app by adding:

* exercise selection
* a feedback history panel
* audio coaching
* a workout summary
* angle visualizations
* session analytics
* premium comparison against a reference movement

For advanced analysis, you can also switch from built-in `exercise=squat` to a custom reference:

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&reference=REFERENCE_UUID
```

This is useful when you want to compare the user against a target movement rather than only using built-in exercise logic.

---

# Common React Native / WebView Fixes

## Android camera issue

Always add:

```text
isAndroid=true
```

when PoseTracker is loaded inside an Android WebView.

## iOS repeated camera prompt

On iOS, when using `react-native-webview`, you can use:

```tsx
{...(Platform.OS === "ios" && {
  mediaCapturePermissionGrantType: "grant",
})}
```

and make sure native camera permission is requested before loading the WebView.

## No useful feedback returned

Check:

* the full body is visible
* lighting is good
* the user is facing the camera
* `recommendations=true` is enabled
* the correct built-in `exercise` is used

---

# Final Prompt for Cursor / Claude / Copilot

Use this exact prompt with your coding assistant:

```text
Build a React Native AI workout form corrector using PoseTracker.

Stack:
- React Native
- Expo
- react-native-webview
- expo-camera

Requirements:
- request camera permission before opening the tracking screen
- embed PoseTracker in a WebView
- use the real-time tracking endpoint
- use a built-in exercise such as squat
- enable recommendations=true
- enable angles=true
- enable progression=true
- display rep count in the UI
- display feedback from recommendations events
- display progression phase
- display angle data if available
- use isAndroid=true on Android
- use mediaCapturePermissionGrantType="grant" on iOS
- do not implement local pose estimation
- use PoseTracker as the motion analysis engine

Use this endpoint pattern:
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&skeleton=true&recommendations=true&angles=true&progression=true

Deliver:
1. a full `WorkoutFormCorrectorScreen.tsx`
2. required package install commands
3. camera permission setup
4. a minimal UI showing reps, feedback, phase, and angles
5. robust WebView event handling
```

# Resources

PoseTracker docs:
[https://posetracker.gitbook.io/posetracker-api](https://posetracker.gitbook.io/posetracker-api)

PoseTracker real-time tracking:
[https://app.posetracker.com/pose_tracker/tracking](https://app.posetracker.com/pose_tracker/tracking)
