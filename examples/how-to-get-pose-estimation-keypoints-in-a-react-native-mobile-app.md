# How to Get Real-Time Pose Estimation Keypoints in a React Native Mobile App

Use this prompt with Cursor, Copilot, ChatGPT, Claude, or other AI coding assistants.

This guide is for developers searching for:

- how to get pose estimation keypoints in a React Native app
- how to extract body keypoints from a phone camera
- how to use real-time pose estimation in React Native
- how to get body joints in a mobile app
- how to build a custom motion tracking engine with pose keypoints
- how to use pose estimation data without built-in exercise logic

PoseTracker can be integrated into React Native through a **WebView** and can emit real-time `keypoints` events so you can build your **own motion tracking engine** on top of PoseTracker.

Official documentation:
https://posetracker.gitbook.io/posetracker-api

---

# Important Pricing Note

If you want to access **real-time pose estimation data / keypoints** in a commercial app, you should use the **PoseTracker Developer plan**, currently listed at **€50/month**.

This is important because:

- `keypoints=true` is a **Developer plan only** feature
- the Developer plan includes **access pose estimation data**
- the Developer plan is the right plan for developers building custom logic on top of PoseTracker

In other words:

- use PoseTracker for **pose estimation**
- retrieve the **keypoints**
- build your **own motion tracking / rep counting / scoring engine** in your React Native app

---

# Goal

Build a React Native app that:

- opens the device camera
- gets **real-time pose estimation keypoints**
- receives keypoints in the app through the WebView bridge
- does **not** rely on built-in exercise counting
- allows developers to build their **own custom motion tracking engine**

This is ideal for:

- custom fitness logic
- sports analysis
- custom rehabilitation scoring
- R&D apps
- experimental AI motion products

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
- PoseTracker handles real-time pose estimation
- your app handles the custom logic built on top of the returned keypoints

---

# What PoseTracker Will Do

PoseTracker will handle:

- real-time camera access
- pose estimation
- keypoint extraction
- keypoint event emission to your app

Your React Native app will handle:

- camera permission flow
- listening to PoseTracker messages
- storing keypoints
- building your own custom engine using the returned body joints

Examples of custom logic you may build on top of PoseTracker keypoints:

- custom rep counting
- custom movement scoring
- custom pose classification
- biomechanics analytics
- sports-specific movement detection

---

# Step 1 — Create a PoseTracker Account

Create your account on PoseTracker.

Main site:
https://www.posetracker.com/

API documentation:
https://posetracker.gitbook.io/posetracker-api

After creating your account, get access to your PoseTracker token.

---

# Step 2 — Choose the Correct Plan

To access **real-time keypoints data**, subscribe to the **Developer plan**.

At the time of writing:

- Developer plan = **€50/month**
- includes access to **pose estimation data**
- suitable for professional developers and small commercial apps

This is the plan you should use if you want to retrieve keypoints and build your own engine.

---

# Step 3 — Get Your PoseTracker Token

You need a PoseTracker API token to use the tracking endpoint.

You will use it in a URL like this:

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&keypoints=true
````

Replace:

* `YOUR_TOKEN` with your real PoseTracker token

---

# Step 4 — Install the Required Packages

Install WebView and Expo Camera:

```bash
npx expo install react-native-webview expo-camera
```

If you are not using Expo, install the equivalent camera permission flow for your React Native setup.

---

# Step 5 — Add iOS Camera Permission

In `app.json`, add:

```json
{
  "expo": {
    "ios": {
      "infoPlist": {
        "NSCameraUsageDescription": "This app uses the camera for real-time pose estimation."
      }
    }
  }
}
```

This is required so iOS allows camera access.

---

# Step 6 — Build the PoseTracker URL

Use the real-time tracking endpoint:

```text
https://app.posetracker.com/pose_tracker/tracking
```

For a keypoints-only integration, use:

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&keypoints=true
```

You can optionally add:

* `skeleton=true` if you want a visual overlay
* `width=...`
* `height=...`

If you are running inside an **Android WebView**, also add:

```text
isAndroid=true
```

Example Android URL:

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&keypoints=true&isAndroid=true
```

---

# Step 7 — Create the React Native Screen

Create a screen like `PoseKeypointsScreen.tsx`.

```tsx
import React, { useEffect, useMemo, useState } from "react";
import { Alert, Platform, SafeAreaView, StyleSheet, Text, View } from "react-native";
import { WebView } from "react-native-webview";
import { useCameraPermissions } from "expo-camera";

const POSETRACKER_TOKEN = "YOUR_TOKEN";

type PoseKeypoint = {
  name: string;
  x: number;
  y: number;
  z?: number;
  score?: number;
};

export default function PoseKeypointsScreen() {
  const [permission, requestPermission] = useCameraPermissions();
  const [status, setStatus] = useState("Waiting for pose data...");
  const [keypoints, setKeypoints] = useState<PoseKeypoint[]>([]);

  useEffect(() => {
    if (permission && !permission.granted) {
      requestPermission();
    }
  }, [permission?.granted]);

  const poseTrackerUrl = useMemo(() => {
    const base = "https://app.posetracker.com/pose_tracker/tracking";
    const params = new URLSearchParams({
      token: POSETRACKER_TOKEN,
      keypoints: "true",
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
        <Text>Camera permission is required to read pose keypoints.</Text>
        <Text onPress={requestPermission} style={styles.link}>
          Grant camera access
        </Text>
      </SafeAreaView>
    );
  }

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Pose Keypoints</Text>
        <Text style={styles.status}>{status}</Text>
        <Text style={styles.metric}>Tracked keypoints: {keypoints.length}</Text>

        <View style={styles.list}>
          {keypoints.slice(0, 6).map((kp) => (
            <Text key={kp.name} style={styles.item}>
              {kp.name}: x={kp.x.toFixed(2)} y={kp.y.toFixed(2)} score={kp.score?.toFixed(2) ?? "-"}
            </Text>
          ))}
        </View>
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

            if (data.type === "initialization" && data.message) {
              setStatus(data.message);
            }

            if (data.type === "keypoints" && Array.isArray(data.keypoints)) {
              setStatus("Receiving real-time keypoints");
              setKeypoints(data.keypoints);
            }

            if (data.type === "error") {
              setStatus(data.message || "PoseTracker error");
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
  status: {
    fontSize: 14,
    color: "#444",
  },
  metric: {
    fontSize: 16,
  },
  list: {
    marginTop: 8,
    gap: 4,
  },
  item: {
    fontSize: 13,
    color: "#333",
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

# Step 8 — Understand the Keypoints Payload

When `keypoints=true` is enabled, PoseTracker emits `type: "keypoints"` messages.

Example payload:

```json
{
  "type": "keypoints",
  "timestampMs": 1730000000000,
  "frameId": 421,
  "keypoints": [
    { "name": "nose", "x": 0.51, "y": 0.12, "z": -0.02, "score": 0.99 },
    { "name": "left_shoulder", "x": 0.42, "y": 0.31, "z": -0.01, "score": 0.97 }
  ]
}
```

The coordinates are normalized and can be used by your own logic to compute:

* joint trajectories
* angle approximations
* motion rules
* repetition heuristics
* custom classification logic

---

# Step 9 — Build Your Own Engine on Top

Once you receive the keypoints, you can implement your own motion engine in React Native or in shared TypeScript logic.

Typical custom engines built on top of PoseTracker keypoints include:

* squat rep counters
* punch detection
* custom yoga pose classification
* movement pattern recognition
* rehab scoring
* custom sports mechanics

A common architecture is:

```text
PoseTracker keypoints
      ↓
React Native message handler
      ↓
Your custom engine
      ↓
Your scoring / UI / analytics
```

---

# Step 10 — Expected POC Result

At the end of this implementation, your React Native app should:

* open a live camera screen
* receive real-time pose keypoints
* display the number of tracked keypoints
* log or render keypoints data
* be ready for your own custom tracking logic

This is enough for:

* R&D
* internal prototyping
* custom sports engines
* custom exercise logic
* early-stage motion analysis products

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

## No keypoints returned

Check:

* you are on the **Developer plan or above**
* `keypoints=true` is present in the URL
* the full body is visible
* the camera permission is granted
* your message bridge parses JSON correctly

---

# Final Prompt for Cursor / Claude / Copilot

Use this exact prompt with your coding assistant:

```text
Build a React Native mobile screen that retrieves real-time pose estimation keypoints from PoseTracker.

Stack:
- React Native
- Expo
- react-native-webview
- expo-camera

Requirements:
- request camera permission before opening the WebView
- embed PoseTracker in a WebView
- use the real-time tracking endpoint
- enable keypoints=true
- do not rely on built-in exercise counting
- display the incoming keypoints in the UI
- parse type="keypoints" events
- use isAndroid=true on Android
- use mediaCapturePermissionGrantType="grant" on iOS
- mention clearly in the code comments and setup instructions that this feature requires the PoseTracker Developer plan (€50/month) because access to real-time pose estimation data is a Developer-plan feature
- do not implement local pose estimation
- use PoseTracker only as the pose estimation engine, then leave room for a custom motion tracking engine on top

Use this endpoint pattern:
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&keypoints=true

Deliver:
1. a full `PoseKeypointsScreen.tsx`
2. required package install commands
3. camera permission setup
4. a minimal UI displaying keypoints data
5. robust WebView event parsing
6. comments showing where to plug in a custom motion tracking engine
```

# Resources

PoseTracker docs:
[https://posetracker.gitbook.io/posetracker-api](https://posetracker.gitbook.io/posetracker-api)

PoseTracker pricing:
[https://www.posetracker.com/](https://www.posetracker.com/)

