Voici **le contenu complet du fichier suivant**, que je te conseille de nommer :

```text
examples/how-to-analyze-an-uploaded-workout-video-in-flutter.md
```

````markdown
# How to Analyze an Uploaded Workout Video in Flutter

Use this prompt with Cursor, Copilot, ChatGPT, Claude, or other AI coding assistants.

This guide is for developers searching for:

- how to analyze an uploaded workout video in Flutter
- how to build video pose analysis in Flutter
- how to detect exercise reps from a recorded video
- how to build a Flutter fitness video analysis app
- how to analyze workout form from video in Flutter
- how to use pose estimation on uploaded video in a mobile app

PoseTracker can be integrated into a Flutter app through a **WebView** to analyze uploaded workout videos.

Official documentation:
https://posetracker.gitbook.io/posetracker-api

---

# Goal

Build a Flutter app that:

- opens a PoseTracker upload flow inside a WebView
- lets the user select a workout video
- analyzes the video with PoseTracker
- displays repetition count
- displays summary results
- optionally shows keypoints or angles
- uses PoseTracker as the video pose analysis engine

This is ideal for:

- workout review apps
- AI coaching apps
- fitness video analysis tools
- asynchronous sports technique review
- rehabilitation monitoring apps

---

# Recommended Stack

This example assumes:

- Flutter
- `webview_flutter`

Why this stack:

- `webview_flutter` is the simplest way to embed PoseTracker in Flutter
- PoseTracker handles the upload UI and analysis flow
- Flutter only needs to load the URL and listen to returned messages

---

# What PoseTracker Will Do

PoseTracker will handle:

- video upload
- pose estimation on the uploaded video
- repetition counting
- optional exercise summary
- optional keypoint and angle outputs

Your Flutter app only needs to:

1. create a PoseTracker account
2. get a PoseTracker API token
3. open the PoseTracker upload endpoint in a WebView
4. pass the correct query parameters
5. listen to PoseTracker events
6. update the UI with results

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

You need a PoseTracker API token to use the upload endpoint.

You will use it in a URL like this:

```text
https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video&exercise=squat
````

Replace:

* `YOUR_TOKEN` with your real PoseTracker token

---

# Step 3 — Install the Required Package

Install the Flutter WebView package:

```bash
flutter pub add webview_flutter
```

---

# Step 4 — Build the Upload Tracking URL

Use the upload endpoint:

```text
https://app.posetracker.com/pose_tracker/upload_tracking
```

For a squat video analysis flow, use:

```text
https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video&exercise=squat&skeleton=true&recommendations=true
```

Useful parameters:

* `token=YOUR_TOKEN`
* `source=video`
* `exercise=squat`
* `skeleton=true`
* `recommendations=true`

You can also enable:

* `angles=true`
* `keypoints=true`

for more advanced analytics.

For a custom reference comparison workflow, use:

```text
https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video&reference=REFERENCE_UUID&skeleton=true
```

Important:

* use `exercise=...` for built-in PoseTracker exercise logic
* use `reference=REFERENCE_UUID` for custom movement comparison
* do not use both together

---

# Step 5 — Create the Flutter Screen

Create a screen like `workout_video_analysis_screen.dart`.

```dart
import 'dart:convert';

import 'package:flutter/material.dart';
import 'package:webview_flutter/webview_flutter.dart';

class WorkoutVideoAnalysisScreen extends StatefulWidget {
  const WorkoutVideoAnalysisScreen({super.key});

  @override
  State<WorkoutVideoAnalysisScreen> createState() =>
      _WorkoutVideoAnalysisScreenState();
}

class _WorkoutVideoAnalysisScreenState
    extends State<WorkoutVideoAnalysisScreen> {
  static const poseTrackerToken = 'YOUR_TOKEN';

  int repCount = 0;
  String status = 'Upload a workout video to begin';
  String feedback = '-';
  String summary = '-';

  late final WebViewController controller;

  @override
  void initState() {
    super.initState();

    final url =
        'https://app.posetracker.com/pose_tracker/upload_tracking'
        '?token=$poseTrackerToken'
        '&source=video'
        '&exercise=squat'
        '&skeleton=true'
        '&recommendations=true';

    controller = WebViewController()
      ..setJavaScriptMode(JavaScriptMode.unrestricted)
      ..addJavaScriptChannel(
        'PoseTrackerBridge',
        onMessageReceived: (JavaScriptMessage message) {
          try {
            final data = jsonDecode(message.message);

            if (data == null || data['type'] == null) return;

            if (data['type'] == 'initialization' && data['message'] != null) {
              setState(() {
                status = data['message'].toString();
              });
            }

            if (data['type'] == 'counter') {
              setState(() {
                repCount = (data['current_count'] ?? 0) as int;
              });
            }

            if (data['type'] == 'recommendations' && data['message'] != null) {
              setState(() {
                feedback = data['message'].toString();
              });
            }

            if (data['type'] == 'exercise_summary') {
              final counter = data['counter']?.toString() ?? '0';
              final avgRepScore = data['avg_rep_score']?.toString() ?? '-';
              final avgSimilarity = data['avg_similarity']?.toString() ?? '-';

              setState(() {
                summary =
                    'reps=$counter, avg_rep_score=$avgRepScore, avg_similarity=$avgSimilarity';
              });
            }

            if (data['type'] == 'error') {
              setState(() {
                status = data['message']?.toString() ?? 'Unknown error';
              });
            }
          } catch (_) {
            // Ignore malformed events during early integration
          }
        },
      )
      ..setNavigationDelegate(
        NavigationDelegate(
          onPageFinished: (_) async {
            await controller.runJavaScript('''
              window.addEventListener("message", function(event) {
                try {
                  const payload = typeof event.data === "string"
                    ? event.data
                    : JSON.stringify(event.data);
                  PoseTrackerBridge.postMessage(payload);
                } catch (e) {}
              });
            ''');
          },
        ),
      )
      ..loadRequest(Uri.parse(url));
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: const Text('Workout Video Analysis'),
      ),
      body: Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(16),
            child: Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: [
                Text('Status: $status'),
                const SizedBox(height: 8),
                Text('Reps: $repCount'),
                const SizedBox(height: 8),
                Text('Feedback: $feedback'),
                const SizedBox(height: 8),
                Text('Summary: $summary'),
              ],
            ),
          ),
          const Divider(height: 1),
          Expanded(
            child: WebViewWidget(controller: controller),
          ),
        ],
      ),
    );
  }
}
```

---

# Step 6 — Understand the Main Events

The most important events for uploaded video analysis are:

### `initialization`

Used to display analysis status.

Example:

```json
{
  "type": "initialization",
  "message": "Your video is processing..."
}
```

### `counter`

Used to display repetition count.

Example:

```json
{
  "type": "counter",
  "current_count": 8
}
```

### `exercise_summary`

Used to display final summary data.

Example:

```json
{
  "type": "exercise_summary",
  "counter": 12,
  "avg_similarity": 0.76,
  "avg_rep_score": 0.81
}
```

### `recommendations`

Used to display workout or posture feedback when available.

Example:

```json
{
  "type": "recommendations",
  "message": "Try to go lower"
}
```

---

# Step 7 — Expected POC Result

At the end of this implementation, your Flutter app should:

* load the PoseTracker upload screen
* let the user select a workout video
* process the video inside PoseTracker
* display repetition count
* display summary results
* optionally display feedback

This is enough for:

* internal demos
* MVPs
* beta products
* fitness video review features
* asynchronous coaching tools

---

# Step 8 — Useful Improvements

After the first POC, you can improve the app by adding:

* exercise selection before loading the WebView
* upload history
* saved workout reports
* support for `reference=REFERENCE_UUID`
* a result card after analysis
* a score breakdown per session
* premium review features for coaching products

You can also enable:

```text
angles=true
keypoints=true
```

if you want richer analytics.

---

# Common Fixes

## No reps detected

Check:

* the correct `exercise` is used
* the full body is visible in the video
* the movement is clearly visible
* the workout video is stable and well lit

## No summary returned

Check:

* `source=video` is present
* the video analysis completed
* your Flutter message bridge is forwarding `postMessage` data correctly

## Wrong comparison mode

Use:

* `exercise=...` for built-in exercise logic
* `reference=REFERENCE_UUID` for custom movement comparison

---

# Final Prompt for Cursor / Claude / Copilot

Use this exact prompt with your coding assistant:

```text
Build a Flutter screen that analyzes an uploaded workout video using PoseTracker.

Stack:
- Flutter
- webview_flutter

Requirements:
- embed PoseTracker in a WebView
- use the upload tracking endpoint
- set source=video
- use a built-in exercise such as squat
- display the upload flow inside the Flutter app
- listen to postMessage events from PoseTracker
- display repetition count in the UI
- display exercise_summary when available
- display recommendations feedback if returned
- do not implement local pose estimation
- use PoseTracker as the uploaded video analysis engine

Use this endpoint pattern:
https://app.posetracker.com/pose_tracker/upload_tracking?token=YOUR_TOKEN&source=video&exercise=squat&skeleton=true&recommendations=true

Deliver:
1. a full Flutter screen
2. required package install command
3. a JavaScript bridge from the WebView to Flutter
4. a minimal UI showing status, reps, feedback, and summary
5. robust event parsing
```

# Resources

PoseTracker docs:
[https://posetracker.gitbook.io/posetracker-api](https://posetracker.gitbook.io/posetracker-api)

Upload tracking:
[https://app.posetracker.com/pose_tracker/upload_tracking](https://app.posetracker.com/pose_tracker/upload_tracking)
