# CMD LedBoard — MediaPipe Pose Detection in Max/MSP

An interactive installation for the CMD Breda MediaLab that drives a large LED board with real-time body tracking. Visitors step in front of a camera—their pose is detected using [MediaPipe](https://ai.google.dev/edge/mediapipe/solutions/vision/pose_landmarker) running inside a Max/MSP `jweb` object—and the resulting visuals are mapped onto a 61×31 LED board (1891 LEDs) via [Protopixel](https://www.protopixel.io).

Based on [jweb-mediapipe](https://github.com/robtherich/jweb-mediapipe) by Rob Ramirez.

## How it works

1. **Max/MSP** loads `pose.html` inside a `jweb` object (Chromium-based browser embedded in Max).
2. **pose.html** loads the MediaPipe Vision bundle from CDN and initializes the `PoseLandmarker` model.
3. A webcam feed is captured via `getUserMedia` and processed at ~15 fps.
4. Detected pose landmarks (left, right, neutral body parts) are written to a Max dictionary (`posedict`) each frame, and an `update` message is sent to the `jweb` outlet so downstream Max objects can react.
5. Max renders 3D visuals driven by the pose data using `jit.gl.model` and sends the output to **Protopixel** via Syphon.
6. **Protopixel** maps the video output onto the LED board (61×31, 1891 LEDs total, split across 16 matrix panels) over ArtNet.

## Project structure

```
OpenDag.maxpat          — Main Max/MSP patch
CMD_OpenDag.ppxproj     — Protopixel project (LED panel mapping)
emo/                    — Emoji/image assets (unzip emo.zip after cloning)
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

## Protopixel LED mapping

The `CMD_OpenDag.ppxproj` file configures [Protopixel](https://www.protopixel.io) to map the visual output onto the LED board:

- **1891 LEDs** in a 61×31 grid, split across **16 matrix panels** in snake wiring
- **Input**: Syphon feed from Max (`jit.gl.syphonserver`), server name `Max`
- **Output**: ArtNet to 4 controllers at `10.0.0.2`–`10.0.0.5`, universes 1/5/9/13

To use: open `CMD_OpenDag.ppxproj` in [Protopixel Mapping Tool](https://www.protopixel.io/product/mapping-tool) and make sure the Max patch is running with Syphon output enabled.

## Requirements

- **Max/MSP 8.x+** with `jweb` and Syphon support
- **[Protopixel Mapping Tool](https://www.protopixel.io/product/mapping-tool)** for LED panel mapping
- Internet connection (MediaPipe model and libraries are loaded from CDN)
- A webcam
- LED panels with ArtNet controllers (for the full installation)

## Author

Developed by **Mark Meeuwenoord** for the [CMD Breda](https://www.cmd-breda.nl) (Communication & Multimedia Design, Avans University of Applied Sciences) open dagen — interactive events where prospective students experience what CMD has to offer. The installation runs on a large LED board in the CMD MediaLab, reacting to visitors' body movements in real time.

## Credits

- Original MediaPipe jweb integration: [robtherich/jweb-mediapipe](https://github.com/robtherich/jweb-mediapipe)
- [MediaPipe Pose Landmarker](https://ai.google.dev/edge/mediapipe/solutions/vision/pose_landmarker) by Google
