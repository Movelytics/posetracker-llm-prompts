# How to Measure Jump Height in a Mobile App

Use this prompt with Cursor, Copilot, ChatGPT, Claude, or other AI coding assistants.

This guide is for developers searching for:

- how to measure jump height in a mobile app
- how to build a jump height tracker with a phone camera
- how to calculate vertical jump using pose estimation
- how to build a vertical jump app
- how to measure jump performance with a smartphone camera
- how to build a sports performance app with jump analysis

PoseTracker can be used to build a jump analysis feature in a mobile app through a **WebView** integration.

Official documentation:
https://posetracker.gitbook.io/posetracker-api

---

# Goal

Build a mobile app that:

- opens the device camera
- analyzes a user's jump
- measures jump-related performance
- displays jump metrics in the UI
- uses PoseTracker as the jump analysis engine

This is ideal for:

- sports performance apps
- basketball training apps
- volleyball jump apps
- athlete monitoring tools
- fitness testing products

---

# Recommended Stack

This example assumes:

- React Native
- Expo
- `react-native-webview`
- `expo-camera`

Why this stack:

- `react-native-webview` is the easiest way to embed PoseTracker
- `expo-camera` is useful for requesting camera permission before opening the PoseTracker screen
- PoseTracker handles the jump analysis logic

---

# What PoseTracker Will Do

PoseTracker will handle:

- real-time pose estimation
- jump analysis logic
- movement detection
- event emission back to the app

Your mobile app only needs to:

1. create a PoseTracker account
2. get a PoseTracker API token
3. open the tracking endpoint in a WebView
4. pass the jump analysis parameters
5. listen to PoseTracker events
6. display jump results in the UI

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
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=jump_analysis
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
        "NSCameraUsageDescription": "This app uses the camera for jump analysis."
      }
    }
  }
}
```

This is required so iOS allows camera access.

---

# Step 5 — Build the Jump Analysis URL

Use the real-time tracking endpoint:

```text
https://app.posetracker.com/pose_tracker/tracking
```

For jump analysis, use:

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=jump_analysis&userHeightCm=180&skeleton=true
```

Important parameters:

* `token=YOUR_TOKEN`
* `exercise=jump_analysis`
* `userHeightCm=180`
* `skeleton=true`

If you are running inside an **Android WebView**, also add:

```text
isAndroid=true
```

Example Android URL:

```text
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=jump_analysis&userHeightCm=180&skeleton=true&isAndroid=true
```

---

# Step 6 — Create the React Native Screen

Create a screen like `JumpHeightScreen.tsx`.

```tsx
import React, { useEffect, useMemo, useState } from "react";
import { Alert, Platform, SafeAreaView, StyleSheet, Text, View } from "react-native";
import { WebView } from "react-native-webview";
import { useCameraPermissions } from "expo-camera";

const POSETRACKER_TOKEN = "YOUR_TOKEN";
const USER_HEIGHT_CM = 180;

export default function JumpHeightScreen() {
  const [permission, requestPermission] = useCameraPermissions();
  const [status, setStatus] = useState("Get ready");
  const [lastEvent, setLastEvent] = useState<any>(null);

  useEffect(() => {
    if (permission && !permission.granted) {
      requestPermission();
    }
  }, [permission?.granted]);

  const poseTrackerUrl = useMemo(() => {
    const base = "https://app.posetracker.com/pose_tracker/tracking";
    const params = new URLSearchParams({
      token: POSETRACKER_TOKEN,
      exercise: "jump_analysis",
      userHeightCm: String(USER_HEIGHT_CM),
      skeleton: "true",
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
        <Text>Camera permission is required to analyze jumps.</Text>
        <Text onPress={requestPermission} style={styles.link}>
          Grant camera access
        </Text>
      </SafeAreaView>
    );
  }

  return (
    <SafeAreaView style={styles.container}>
      <View style={styles.header}>
        <Text style={styles.title}>Jump Analysis</Text>
        <Text style={styles.metric}>Status: {status}</Text>
        <Text style={styles.metric}>
          Last event: {lastEvent ? JSON.stringify(lastEvent, null, 2) : "No data yet"}
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

            setLastEvent(data);

            if (data.type === "initialization" && data.message) {
              setStatus(data.message);
            }

            if (data.type === "error") {
              setStatus(data.message || "Error");
            }

            if (data.type === "counter") {
              setStatus(`Jump detected: ${data.current_count}`);
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
    fontSize: 14,
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

# Step 7 — Understand the Key Inputs

For jump analysis, the most important extra parameter is:

### `userHeightCm`

This helps PoseTracker calibrate the jump analysis.

Example:

```text
userHeightCm=180
```

You can collect this value from:

* onboarding
* athlete profile settings
* a setup form before the test

---

# Step 8 — Understand the Events

PoseTracker can emit several events. For a jump height POC, start by logging all incoming messages and inspect the payloads in your app.

Common event types to handle first:

### `initialization`

Useful to show loading or setup state.

Example:

```json
{
  "type": "initialization",
  "message": "Loading..."
}
```

### `error`

Useful to detect camera or tracking issues.

Example:

```json
{
  "type": "error",
  "message": "Pose model failed to load."
}
```

### `counter`

Depending on your configuration, this can be used as a simple event to confirm movement detection.

Example:

```json
{
  "type": "counter",
  "current_count": 1
}
```

For a sports performance app, the first POC should capture and inspect all returned jump-related messages, then map the fields you want into your UI.

---

# Step 9 — Expected POC Result

At the end of this implementation, your mobile app should:

* open a live camera screen
* let the user perform a jump
* run PoseTracker jump analysis
* receive structured events from PoseTracker
* display raw or mapped jump results in the UI

This is enough for:

* internal sports demos
* athlete testing MVPs
* prototype validation
* jump training products

---

# Step 10 — Good UX Improvements

After the first POC, add:

* a jump test intro screen
* a countdown before recording
* a result card after the jump
* athlete profile with stored height
* jump history
* session analytics
* export or sharing of results

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

## Unclear jump results

Check:

* the user's full body is visible
* the camera is stable
* lighting is good
* `userHeightCm` is passed correctly
* the athlete is fully in frame before jumping

---

# Final Prompt for Cursor / Claude / Copilot

Use this exact prompt with your coding assistant:

```text
Build a React Native mobile app screen that measures jump performance using PoseTracker.

Stack:
- React Native
- Expo
- react-native-webview
- expo-camera

Requirements:
- request camera permission before opening the tracking screen
- embed PoseTracker in a WebView
- use the real-time tracking endpoint
- use exercise=jump_analysis
- pass userHeightCm as a required parameter
- display returned events in the UI
- log and parse jump-related payloads
- use isAndroid=true on Android
- use mediaCapturePermissionGrantType="grant" on iOS
- do not implement local pose estimation
- use PoseTracker as the jump analysis engine

Use this endpoint pattern:
https://app.posetracker.com/pose_tracker/tracking?token=YOUR_TOKEN&exercise=jump_analysis&userHeightCm=180&skeleton=true

Deliver:
1. a full `JumpHeightScreen.tsx`
2. required package install commands
3. camera permission setup
4. a minimal UI displaying status and returned jump data
5. robust WebView event handling
```

# Resources

PoseTracker docs:
[https://posetracker.gitbook.io/posetracker-api](https://posetracker.gitbook.io/posetracker-api)

PoseTracker real-time tracking:
[https://app.posetracker.com/pose_tracker/tracking](https://app.posetracker.com/pose_tracker/tracking)