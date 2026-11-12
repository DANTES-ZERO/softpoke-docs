# Built-in webcam tracker (no Python)

> SoftPoke 1.0.0 | [Index](Index.md)

`SoftPokeHandSourceNative` tracks your hand with a webcam **inside Unreal**: no Python, no separate program, no UDP. This is the webcam input of SoftPoke and the one the demo maps use. A separate [Python webcam tracker](WebcamTracker.md) is included as an **advanced, optional** alternative (head / face tracking, recording takes, calibration tools); you do not need it.

How it works: the camera is read with Windows Media Foundation on a capture thread (only the newest frame is kept). A second thread runs Google MediaPipe's two hand models (palm detector + 21-point hand landmarks, Apache-2.0, converted to ONNX) through Unreal's Neural Network Engine (NNE: ONNX Runtime on the CPU, or with DirectML on the GPU). The palm detector runs only while a hand is missing; a tracked hand needs only the landmark model. The hand depth is solved from the image and your hand size (the same translation-only PnP as the Python tracker), and the hand is placed in the world with the same workspace rules. The game thread only copies the newest result.

## Requirements

* Windows 64, a webcam (USB or built-in; 720p at 30 fps recommended).
* The engine plugin **NNERuntimeORT** (ONNX Runtime for the Neural Network Engine, NNE). It ships with the engine but is **experimental / beta in UE 5.8**: it can change or have problems that SoftPoke cannot fix. SoftPoke declares it as an optional dependency and enables it with itself. If you disable it, the native source reports "NNERuntimeORTCpu is not available" and the rest of SoftPoke works as before.
* For `Backend = DirectML`: the D3D12 RHI (SoftPoke needs it anyway).

## Set up (two minutes)

1. On the character's actor, select the **SoftPokeHandRig** and set **Source Class** to `SoftPokeHandSourceNative` (or create the source in the rig's **Source** field to edit its settings).
2. Choose the camera. **Camera Index** 0 is the first camera Windows lists; or type part of the camera's name into **Camera Name** (used when not empty). `ListCameras()` (Blueprint) lists the devices and their formats. **Virtual cameras** (EpocCam, OBS Virtual Camera, NVIDIA Broadcast, ...) are often device 0: read `Status` for the device that was opened and set Camera Name or Camera Index if it is the wrong one. Before the first Play check Windows **Settings > Privacy & security > Camera > Let desktop apps access your camera = On**, and close other programs that use the camera. Keep your hand 40-80 cm from the camera, in even light, against a plain background.
3. Enter your **Hand Size Cm**: wrist crease to the base of the middle finger, measured with a ruler (adults: typically 8.5-11 cm). The depth of the hand is proportional to this value.
4. Play. Hold your hand where it should rest in front of the camera, then press **C** (with a `SoftPokeTrackerLink` in the level, see below) or call `Recenter()`. (The first Play compiles shaders once; see the [Quick Start](QuickStart.md).) The neutral position is stored in `Saved/SoftPoke/NativeTracking.json` and loaded next time (`bRememberCalibration`).
5. Optional: place the hand in the level with **Workspace** (`AnchorCm` = world position of the hand at the neutral position, `YawDeg`, `Gain`, `CameraPitchDeg`, `bFirstPerson`).

No camera at hand? Set **Video File** to an `.mp4` of your hand: the source plays it in a loop at its own frame rate instead of the camera.

### Following the view and recenter (SoftPokeTrackerLink)

Add a `SoftPokeTrackerLink` component (any actor) and turn on `bEnabled`. When a rig uses the native source, the link drives it directly: the hand workspace follows the view (`FollowForwardCm` 40 cm in front of it, `FollowDownCm` 25 cm below, turned with the view's yaw) and the **C** key recenters. Nothing is sent over the network and no tracker program is started; the link's tracker-launch options only concern the Python tracker.

### Using your Python tracker calibration

`ImportProfileJson` / `ImportWorkspaceJson` read the Python tracker's `profile.json` (hand size, field of view, mirroring, intrinsics) and `workspace.json` (anchor, neutral, gain, yaw, camera tilt `view_from_cam`, preset) when the source starts.

### Measuring the hand size with the camera

Hold ONE open hand, palm to the camera, with the hand centre at a measured distance (for example 40 cm from the camera plane) and call `CalibrateHandSize(40)`. It takes the median of the last 30 frames, applies the size at once and stores it.

## Settings that matter

| Setting | Default | Notes |
|---|---|---|
| `Width` / `Height` / `Fps` | 1280 / 720 / 30 | The closest format the camera offers is used (size first, then rate, then MJPG > NV12 > YUY2; `PreferredFormat` overrides). |
| `bMirrored` | off | Turn on if the camera driver delivers a mirrored (selfie) image. Handedness and depth are corrected. |
| `HorizontalFovDeg` | 70 | The camera's horizontal field of view. A wrong value scales the depth; use `Intrinsics` if you have a calibration. |
| `Backend` | CPU | `DirectML` runs the models on the GPU (about 1.5 ms per model on an RTX 4090; the models are created in the background in about 1 s when the source starts). CPU is the default so the tracker does not compete with rendering. |
| `MaxHands` | 2 | 1 saves the second landmark pass when only one hand is used. |
| `DetectIntervalFrames` | 8 | While one hand is tracked, the palm detector looks for the second hand every N frames (1 = every frame, like MediaPipe's VIDEO mode). |
| `TranslationAlpha` | 1 (off) | Optional extra smoothing of the hand position. The rig's One-Euro filter already smooths every source. |
| `bPreview` | off | Keeps a small camera image with the landmarks in a texture (`GetPreviewTexture()`), for example for a UMG image. |
| `Status`, `Stats` | | Camera format, camera / tracker fps, inference time, capture-to-keypoint latency. |

### CPU threads (important for packaged games)

ONNX Runtime's thread count is a **project** setting of the NNERuntimeORT plugin (Project Settings > Plugins > NNERuntimeORT, restart required), not a setting of the source. The engine default is all cores in the editor but **one thread in a packaged game**. With one thread a tracked hand costs about 9 ms and the palm detector about 15 ms on a Ryzen 7 7800X3D: this still fits a 30 fps camera but leaves little margin on slower CPUs. We recommend 2-4 threads; add to your project's `Config/DefaultEngine.ini`:

```ini
[/Script/NNERuntimeORT.NNERuntimeORTSettings]
GameThreadingOptions=(bUseGlobalThreadPool=False,IntraOpNumThreads=3,InterOpNumThreads=1,ExecutionMode=SEQUENTIAL)
```

## Accuracy and speed (measured on the developer PC)

Ryzen 7 7800X3D + RTX 4090, UE 5.8, 1280x720 H.264 test clips, compared frame by frame with the Python tracker (MediaPipe 1.0.1, VIDEO mode) on the same clips:

| | Native vs Python tracker |
|---|---|
| Landmarks (2D) | mean 0.74 px, median 0.25 px (0.36 % of the hand size) |
| Hand depth | RMS difference 0.44 cm (0.82 %) |
| Hand orientation | mean 0.67 deg |
| Joints in the world | mean 0.21 cm |

| Per frame | CPU, 4 threads | CPU, all cores (editor) | CPU, 1 thread (game default) | DirectML |
|---|---|---|---|---|
| One tracked hand (landmarks) | 3.9 ms | 3.1 ms | 8.9 ms | 1.5 ms |
| Palm detector (only while a hand is missing) | 7.2 ms | 6.6 ms | 15.0 ms | 1.3 ms |

Frame arrival to keypoints ready: about 6 ms (CPU, two hands) in the threaded source; add the time until your next game tick. For comparison, the Python tracker needs about 38 ms from capture to the UDP packet.

## Privacy

The camera image is processed on your computer only. It is not stored, recorded or sent anywhere. The models run locally; no network connection is used.

## Troubleshooting

| Symptom | Check |
|---|---|
| `Status` says "NNERuntimeORTCpu is not available" | Enable the engine plugin NNERuntimeORT (Edit > Plugins) and restart. |
| "no video capture device found" / "camera index out of range" | `ListCameras()`. Close other programs that use the camera (video calls, OBS, the Python tracker). |
| "cannot activate camera" | The camera is in use, or the Windows camera privacy settings block desktop apps (Settings > Privacy & security > Camera > Let desktop apps access your camera). |
| The wrong camera is used (a virtual camera) | Set **Camera Name** or **Camera Index**; check `Status` for the device name. |
| The hand is too near / too far | `HandSizeCm` and `HorizontalFovDeg`. |
| Left and right hands are swapped | `bMirrored`. |
| Low tracker fps in a packaged game | Raise the NNERuntimeORT thread count (above), or `MaxHands = 1`, or `Backend = DirectML`. |

## Status of testing

This feature is new. It was tested on the developer PC with an EpocCam virtual camera and with recorded hand takes sent over UDP. A physical USB webcam is not among the cameras it was tested with, and it has **not yet been tested with a real, physical webcam in a cooked (packaged) build**. Please report camera formats that fail (the `Status` line shows the format that was chosen). The tracker also rests on the engine's experimental NNE plugin (see Requirements).

## Licence of the models

`Resources/Models/hand_detector.onnx` and `hand_landmarks_detector.onnx` are Google MediaPipe Hand Landmarker models (Apache License 2.0), converted from TensorFlow Lite to ONNX without changing the weights. See `Resources/Models/NOTICE.txt`, `Licenses/LICENSE_Apache-2.0_MediaPipe_models.txt` and `Licenses/THIRD_PARTY_NOTICES.md`.
