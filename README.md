# CMD LedBoard — MediaPipe Pose Detection in Max/MSP

Real-time pose detection using [MediaPipe](https://ai.google.dev/edge/mediapipe/solutions/vision/pose_landmarker) inside a Max/MSP `jweb` object. Detects body landmarks from a webcam feed and sends the data into Max as a dictionary for further processing (e.g. driving LED boards, visuals, or sound).

Based on [jweb-mediapipe](https://github.com/robtherich/jweb-mediapipe) by Rob Ramirez.

## How it works

1. **Max/MSP** loads `pose.html` inside a `jweb` object (Chromium-based browser embedded in Max).
2. **pose.html** loads the MediaPipe Vision bundle from CDN and initializes the `PoseLandmarker` model.
3. A webcam feed is captured via `getUserMedia` and processed at ~15 fps.
4. Detected pose landmarks (left, right, neutral body parts) are written to a Max dictionary (`posedict`) each frame, and an `update` message is sent to the `jweb` outlet so downstream Max objects can react.

## Project structure

```
OpenDag.maxpat          — Main Max/MSP patch
pose.html               — HTML page loaded by jweb
css/mesh-style.css      — Styles for the overlay canvas
js/
  pose.js               — Core pose detection logic (the main script)
  pose-landmarks-index.js — Named landmark index mappings (left/right/neutral)
  face-landmarker.js    — Face landmark detection (separate module)
  facemesh.js           — Face mesh detection
  handpose.js           — Hand pose detection
  hands-gesture-recognizer.js — Hand gesture recognition
  hands-landmarks-index.js    — Hand landmark index mappings
  object-detection.js   — Object detection module
```

## Max → JS communication

The `jweb` object sends messages to `pose.js` via bound inlets:

| Message | Description |
|---|---|
| `set_mediadevice <label>` | Switch to a specific webcam by device label |
| `get_mediadevices` | Outputs a list of available video devices |
| `set_image <path>` | Switch to single-image mode and detect on a still image |
| `draw_image <0/1>` | Toggle drawing the camera feed on the canvas |
| `draw_landmarks <0/1>` | Toggle drawing pose landmark points |
| `draw_connectors <0/1>` | Toggle drawing skeleton connections |

## JS → Max communication

| Output | Description |
|---|---|
| `update` | Sent each frame after landmarks are written to the dictionary |
| `dictionary posedict` | Pose data structured as `{left: {...}, right: {...}, neutral: {...}}` |
| `mediadevices <list>` | List of available video input device labels |
| `error <message>` | Error messages |

## Requirements

- **Max/MSP 8.x+** with `jweb` support
- Internet connection (MediaPipe model and libraries are loaded from CDN)
- A webcam

## Author

Developed by **Mark Meeuwenoord** for the [CMD Breda](https://www.cmd-breda.nl) (Communication & Multimedia Design, Avans University of Applied Sciences) open dagen — interactive events where prospective students can experience what CMD has to offer. This project drives a real-time LED board installation that reacts to visitors' body movements.

## Credits

- Original MediaPipe jweb integration: [robtherich/jweb-mediapipe](https://github.com/robtherich/jweb-mediapipe)
- [MediaPipe Pose Landmarker](https://ai.google.dev/edge/mediapipe/solutions/vision/pose_landmarker) by Google
