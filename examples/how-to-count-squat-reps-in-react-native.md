# How to Count Squat Reps in a React Native App

Use this prompt with Cursor, Copilot, ChatGPT, Claude, or other AI coding assistants.

This guide is for developers searching for:

- how to count squat reps in a React Native app
- how to build a squat counter with React Native
- how to detect squats using a phone camera
- how to build a fitness rep counter in React Native
- how to use pose estimation in a React Native workout app
- how to build an AI squat tracker in Expo

PoseTracker is a motion tracking and pose estimation solution that can be integrated into React Native through a **WebView**.

Official documentation:
https://posetracker.gitbook.io/posetracker-api

---

# Goal

Build a React Native app that:

- opens the device camera
- tracks squat repetitions in real time
- displays the rep counter in the UI
- optionally displays posture feedback
- uses PoseTracker as the tracking engine

This is ideal for:

- fitness apps
- AI personal trainer apps
- workout trackers
- home training apps
- bodyweight exercise apps

---

# Recommended Stack

This example assumes:

- React Native
- Expo
- `react-native-webview`
- `expo-camera`

Why this stack:

- `react-native-webview` is the easiest way to embed PoseTracker
- `expo-camera` is useful to request camera permission before opening the PoseTracker screen
- PoseTracker handles the actual motion tracking logic

---

# What PoseTracker Will Do

PoseTracker will handle:

- real-time pose estimation
- squat repetition counting
- posture validation
- progression events
- optional recommendations
- optional keypoints / angles

Your React Native app only needs to:

1. create a PoseTracker account
2. get an API token
3. open the PoseTracker tracking URL in a WebView
4. listen to events returned by PoseTracker
5. update the UI

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

If you are using bare React Native, install the corresponding camera permission setup for your stack.

---

# Step 4 — Add iOS Camera Permission

In `app.json`, add:

```json
{
  "expo": {
    "ios": {
      "infoPlist": {
        "NSCameraUsageDescription": "This app uses the camera for squat tracking."
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

For squat counting, use:

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&skeleton=true&recommendations=true&progression=true
```

Useful parameters:

* `token=YOUR_TOKEN`
* `exercise=squat`
* `skeleton=true`
* `recommendations=true`
* `progression=true`

If you are running inside an **Android WebView**, you must also add:

```text
isAndroid=true
```

Example Android URL:

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&skeleton=true&recommendations=true&progression=true&isAndroid=true
```

---

# Step 6 — Create the React Native Screen

Create a screen like `SquatTrackerScreen.tsx`.

```tsx
import React, { useEffect, useMemo, useState } from "react";
import { Alert, Platform, SafeAreaView, StyleSheet, Text, View } from "react-native";
import { WebView } from "react-native-webview";
import { useCameraPermissions } from "expo-camera";

const POSETRACKER_TOKEN = "YOUR_TOKEN";

export default function SquatTrackerScreen() {
  const [permission, requestPermission] = useCameraPermissions();
  const [repCount, setRepCount] = useState(0);
  const [feedback, setFeedback] = useState("Get ready");
  const [phase, setPhase] = useState("idle");

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
        <Text>Camera permission is required to track squats.</Text>
        <Text onPress={requestPermission} style={styles.link}>
          Grant camera access
        </Text>
      </SafeAreaView>
    );
  }

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Squat Counter</Text>
        <Text style={styles.metric}>Reps: {repCount}</Text>
        <Text style={styles.metric}>Phase: {phase}</Text>
        <Text style={styles.feedback}>{feedback}</Text>
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
              // Adapt this if you want to map exact PoseTracker progression fields
              setPhase(data.phase || data.state || "moving");
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
    gap: 6,
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

PoseTracker will send events to your app through the WebView bridge.

The most important ones for a squat counter are:

### `counter`

Used to update the squat count.

Example:

```json
{
  "type": "counter",
  "current_count": 8
}
```

### `recommendations`

Used to show posture or movement feedback.

Example:

```json
{
  "type": "recommendations",
  "message": "Go lower"
}
```

### `progression`

Used to track movement phase, for example going down or up.

Example:

```json
{
  "type": "progression",
  "phase": "down"
}
```

---

# Step 8 — Expected POC Result

At the end of this implementation, your React Native app should:

* open a live camera screen
* track squats in real time
* increment a rep counter
* display basic coaching feedback
* work as a simple squat counting POC

This is enough for:

* internal demos
* MVPs
* beta testing
* validating fitness use cases

---

# Step 9 — Useful Improvements

After the first POC, you can improve the app by adding:

* a start screen
* a workout summary screen
* sound feedback on each rep
* calories / training analytics
* session duration
* posture history
* premium feedback using angles or keypoints

You can also enable:

* `angles=true`
* `keypoints=true`

if you want to build more advanced analytics.

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

## No reps detected

Check:

* the full body is visible
* lighting is good
* the user is facing the camera
* the correct `exercise=squat` parameter is used

---

# Final Prompt for Cursor / Claude / Copilot

Use this exact prompt with your coding assistant:

```text
Build a React Native squat rep counter using PoseTracker.

Stack:
- React Native
- Expo
- react-native-webview
- expo-camera

Requirements:
- request camera permission before opening the tracking screen
- embed PoseTracker in a WebView
- use the real-time tracking endpoint
- use the built-in squat exercise
- display squat rep count in the UI
- display posture feedback from recommendations events
- display progression phase if available
- use isAndroid=true on Android
- use mediaCapturePermissionGrantType="grant" on iOS
- do not implement local pose estimation
- use PoseTracker as the real-time tracking engine

Use this endpoint pattern:
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=squat&skeleton=true&recommendations=true&progression=true

Deliver:
1. a full `SquatTrackerScreen.tsx`
2. required package install commands
3. camera permission setup
4. a minimal UI showing rep count and feedback
5. robust WebView event handling
```

# Resources

PoseTracker docs:
[https://posetracker.gitbook.io/posetracker-api](https://posetracker.gitbook.io/posetracker-api)

PoseTracker real-time tracking:
[https://app.posetracker.com/pose_tracker/tracking](https://app.posetracker.com/pose_tracker/tracking)

Reference movement demo:
[https://app.posetracker.com/test_reference_movement_demo.html](https://app.posetracker.com/test_reference_movement_demo.html)
