# Webcam tracker guide

> **Advanced, optional.** Most users do not need this program: the demo maps and the Quick Start use the built-in tracker.
>
> The built-in tracker `SoftPokeHandSourceNative` tracks the hand inside Unreal without Python ([Built-in webcam tracker](NativeTracking.md)). Use the Python tracker for head / face tracking, recording takes, or the LiDAR / view calibration tools.

> SoftPoke 1.0.0 | [Index](Index.md)

The webcam hand tracker is a **separate program** that runs next to Unreal Engine. It reads your camera, finds your hands (and, optionally, your head and facial expression) with Google's MediaPipe, and sends one small UDP message per camera frame to the plugin's `SoftPokeHandSourceUdp` source. The message format is public: [UDP protocol v1](ApiAndExtension.md#udp-protocol-version-1), so you can also write your own sender.

The tracker is part of the plugin package, in the folder `Tools/Webcam` (next to `Tools/Bake`). It is plain Python source that you set up once on your own computer.

## What it needs

* Windows, a webcam (or a phone used as a virtual webcam), and 64-bit Python 3.12 from python.org (the pinned packages are built for it).
* An internet connection for the one-time set-up. Nothing is shipped inside the plugin: `pip` downloads the Python packages listed in `requirements.lock` (`mediapipe`, `opencv-python`, `opencv-contrib-python`, `numpy` and their dependencies) into a folder you choose, and `fetch_models.py` downloads the MediaPipe model files (about 11 MB, checksum-verified) from Google's public model storage. Their licences are listed in the [licence index](Licences.md). Nothing in the tracker uses the GPU.

## Set up

1. **Copy `Tools/Webcam` to a folder you can write to**, for example `C:\SoftPokeTracker\Webcam`. The tracker stores `profile.json` (hand size, mirror check) and `workspace.json` next to `tracker.py`, and an engine-wide plugin folder is often read-only.
2. In that folder run (the bake panel's **Set up** uses the same script, with the bake packages instead):

```
python setup_env.py --venv C:\SoftPokeTracker\venv --lock requirements.lock --selfcheck tracker_selfcheck.py --post-script fetch_models.py
```

`python` must be the 64-bit Python 3.12. The script creates the venv folder, installs `requirements.lock` into it, runs `tracker_selfcheck.py` (imports the packages and checks the MediaPipe API) and then runs `fetch_models.py` inside the venv. Nothing is installed outside the two folders. It takes about a minute. The exit code is 0 when ready; 30 means the base Python is not suitable, 32 pip failed (check the internet connection or proxy; run it again, the venv is reused), 33 the self-check failed, 35 the model download failed. `--json status.jsonl` writes a machine-readable log (the same format as the bake Set up).

Without `setup_env.py` you can do the same by hand:

```
py -3.12 -m venv C:\SoftPokeTracker\venv
C:\SoftPokeTracker\venv\Scripts\python -m pip install --only-binary=:all: -r requirements.lock
C:\SoftPokeTracker\venv\Scripts\python fetch_models.py
```

`fetch_models.py --hands-only` skips the two face files (only `--face` needs them). `tracker_selfcheck.py --cameras` also tests which camera indices open.

## Run it

1. In Unreal: put a `SoftPokeHandRig` on the character's actor and set its **Source Class** to `SoftPokeHandSourceUdp` (port 9763, bind address `127.0.0.1`). Press Play.
2. Find your camera: `C:\SoftPokeTracker\venv\Scripts\python tracker.py --list-cameras` prints the indices that open, with delivered size and measured frame rate. (Below, `tracker.py` stands for that command line.)
3. Start tracking with a preview window:

```
tracker.py --camera 0 --width 1280 --height 720 --fps 30 --preview
```

No camera, or the camera is busy? `tracker.py --video clip.mp4 --realtime` plays a recorded video at its own frame rate and sends the same packets (`--images <folder>` does the same for still photos). This is also the quickest way to check the whole chain.

The tracker opens the camera through DirectShow first, then Media Foundation, and asks for MJPG (most webcams need it for 30 fps at 720p; `--fourcc ""` turns that off). It keeps only the newest frame, so no delay builds up. Preview keys: **Esc twice within two seconds** quits (a single key such as `q` does not, so that a game's movement keys cannot close the tracker by accident), `n` sets the neutral hand position (and `h` / `v` the head neutral and camera tilt with `--face`).

Useful options: `--host`/`--port` (default `127.0.0.1:9763`), `--space ue_world_cm|mp_world_m` (use `ue_world_cm`), `--mirrored auto|yes|no`, `--hfov-deg`, `--hand-size-cm`, `--face` and `--send-head` (head pose and 51 blendshapes), `--face-every N` (run the face model on every Nth frame), `--print-every N`, `--log timing.csv`. It also runs headless without `--preview`, and `--max-seconds N` / `--max-frames N` stop it by itself.

### Recorded videos, recording takes and replaying them

* **Video file as the camera:** `tracker.py --video clip.mp4` (any frame rate; add `--realtime` to pace it at the file's frame rate, otherwise it is tracked as fast as the machine allows). `--start 2.5 --end 8` uses only that part of the clip. The time stamps in the packets are the *media* time of the frame, so the result does not depend on the speed of the run. Use `--flip-input` (and `--mirrored`) for a selfie-view clip, as with a live camera.
* **Record the landmark stream:** `--record-landmarks take.jsonl` writes one line per video / camera frame (media time and the UDP packet, `null` when no hand was found); `--no-send` records without sending to Unreal. In a live session, `--record-dir takes --preview` lets you start and stop takes with the SPACE key in the preview window (files `takes/take_<time>_<n>.jsonl`) while the hand is still sent to Unreal, so you see yourself in the viewport.
* **Replay a recording:** `replay_landmarks.py take.jsonl` sends the stored packets over UDP at their original times (`--speed 0.5`, `--loop`, `--start`, `--end`, `--mirror` for the other hand). It needs no camera and no MediaPipe: use it for offline, repeatable tests and for rendering the same motion several times. For frame-locked rendering (a capture tool that steps through the recording and waits for the solver before every picture), import `replay_landmarks` and send one frame per step with `send_frame`.

### Following the game view, recenter, and the SoftPokeTrackerLink component

By default the hand workspace is fixed in the world (`anchor_ue_cm` and `yaw_deg` in `workspace.json`). For a free viewpoint (a desktop "VR view" where you walk around the character) start the tracker with `--follow-port 9799`: it then listens on that UDP port (127.0.0.1) for the view pose sent by the game as text `x y z yaw_deg`, and keeps the hand 40 cm in front of and 25 cm below the eye, facing the viewing direction. The follow-derived anchor and yaw are never written to `workspace.json`. The same port accepts the text `recenter` (the current hand position becomes the neutral position, like `n` in the preview; only `neutral_cam_m` is saved) and `rec` (start / stop a take, like SPACE; needs `--record-dir`). All of it works with a plain webcam or a video file. Any program can send these messages; the plugin component for it is **`SoftPokeTrackerLink`**:

1. Add `SoftPokeTrackerLink` (and, for a keyboard / mouse viewpoint without a headset, `SoftPokeDesktopFly`) to an actor and tick `bEnabled`. Both are off by default.
2. Start the tracker yourself with `--follow-port 9799`, **or** let the component do it: tick `bLaunchTracker`, set `TrackerPython` (the `python.exe` of the tracker's venv) and `TrackerFolder` (the folder with `tracker.py`), and adjust `TrackerArguments` (camera, size, `--face --send-head` ...). The component adds `--follow-port` and `--preview` itself, stops the tracker when the game ends, and restarts it after a crash (not after Esc Esc). Do not run two trackers at once.
3. In the game window: **C** recenters (hold your hand in the neutral spot first), **G** toggles the hand debug drawing, **T** shows or hides the preview window (restarts a tracker that the component launched). The keys are properties of the component (`RecenterKey`, `DebugDrawKey`, `PreviewKey`) and work while the game window has the focus. The `Recenter()` function can be called from a Blueprint.
4. The tracker's preview window has the focus when you click it: then keys go to the tracker, not to the game.

Recording takes (SPACE in the preview window, or `rec`) stays a tracker feature; see the section above. `SoftPokeTrackerLink` sends nothing but the view pose and these short commands, and only to `FollowHost` (default 127.0.0.1).

## Calibrate (five minutes, once per camera)

| Step | Command | Why |
|---|---|---|
| Left/right | `tracker.py mirror-check --camera 0`, raising only your **right** hand | Decides and stores whether your camera image is mirrored. A wrong answer swaps the hands. Verify later in the preview: raise the right hand, the label must say RIGHT and the character's right hand must move. |
| Hand size | `tracker.py calibrate-hand --size-cm 9.4` (ruler: wrist crease to the base of the middle finger) or `calibrate-hand --camera 0 --distance-cm 40` (open hand, palm to the camera, at a measured distance) | Depth is proportional to the registered hand size: a 10 % size error gives a 10 % depth error. |
| Camera field of view | `--hfov-deg` (default 70, the horizontal field of view of the delivered image) | Phone cameras used as webcams are often cropped. Put your hand at 40 cm and 60 cm and adjust until `--print-every 1` shows the measured distance. A 10 % field-of-view error gives roughly a 12-14 % depth error. |
| Workspace | In the preview, hold your hand where it should touch the character and press `n` | Stores the hand position that maps to the anchor point in front of the character in `workspace.json`. Edit `anchor_ue_cm` (where in the level), `yaw_deg` (which way the character faces) and `gain` (how far a hand movement travels). `--preset first_person` is available for a character facing you. |
| Face (optional) | `tracker.py calibrate-face --ipd-cm <value>` | Head size for the head collider. |

The left/right rule in short: MediaPipe labels a hand as it **appears in the image**. If your camera image is already mirrored (many phone-camera apps and OBS "mirror" options do that), the tracker swaps the labels back, so the packet always carries the user's anatomical hand.

## Accuracy and latency

* The depth of the hand comes from the hand size and the camera geometry, not from a depth sensor. In the tracker's offline simulation (realistic landmark error, 1280x720, 70 degree field of view) the position error is about 1.3-1.9 % of the distance (about 4 mm at 20 cm, 14 mm at 80 cm).
* MediaPipe's own depth is flattened, so the **hand orientation** is less accurate than the position. The webcam path is meant for casual, direct interaction, not for precision.
* The software delay from camera frame to the UDP send is of the order of tens of milliseconds; the camera and driver add their own, and so does the rendering. The end-to-end delay has not been measured and published yet.

## Troubleshooting

| Problem | What to check |
|---|---|
| `setup_env.py` stops with exit 30 / 32 / 35 | 30: use 64-bit Python 3.12 from python.org. 32 and 35: no internet or a proxy (add `--pip-args "--proxy http://host:3128"` for pip); run the same command again, the venv is reused. The last lines of the output name the failing package or download. |
| `--list-cameras` shows no camera, or the preview is black | Another program (a video call, OBS) may hold the camera: close it. Phone cameras need the phone's virtual-webcam app running first. Meanwhile `--video clip.mp4 --realtime` lets you test the rest. |
| The hand does not appear in Unreal | The rig's source is `SoftPokeHandSourceUdp`; the same port on both sides; Play is running. On the UDP source `PacketsReceived` must grow. If `PacketsRejected` grows, the log (`LogSoftPoke`) names the reason of the first rejected packets. |
| The wrong hand moves | Run `mirror-check` again. |
| The hand drifts away or the depth is wrong | Re-run the hand-size calibration and check `--hfov-deg`. |
| The hand jitters | Hold still and watch the wrist position with `--print-every`. On the rig, a lower `MinCutoffHz` smooths more when the hand is still; a higher `Beta` reduces the lag when it moves fast. |
| Frame rate is low | Use MJPG at 1280x720 and 30 fps; close other camera apps; `--list-cameras` shows the delivered rate. |
| Tracker on another PC | Set the tracker's `--host` to the PC with Unreal, and in Project Settings > Plugins > SoftPoke set `UdpBindAddress` to `0.0.0.0`; allow UDP 9763 in the firewall on a private network. |

## Privacy

The tracker processes camera images on your computer. It sends only joint coordinates (and, with `--face`, head pose and blendshape values) over UDP to the address you give it; it does not send images anywhere, and it stores frames only if you pass `--save-annotated`.
