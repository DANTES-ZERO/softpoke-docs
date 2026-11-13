# SoftPoke: third-party notices

Status of this file: SoftPoke 1.0.0, reviewed 2026-11-02, updated 2026-11-12.

## What the plugin package contains

The package (`Source/SoftPokeRuntime`, `Source/SoftPokeEditor`, `Shaders/`, `Content/`) is original code. The audit of 2026-10-13 found:

* No third-party source code, headers, libraries, binaries or fonts inside the plugin, except the ufbx library source in `Tools/Bake/tools/ufbx_dump/` (MIT or public domain, see "ufbx" below) and the two MediaPipe hand models listed below (Apache-2.0, data files). The sample content in `Content/SoftPokeSample/` (a sample character with hands and hair, baked data and demo maps) has its own provenance note: `SAMPLE_PROVENANCE.md` (includes the AI-content disclosure and the permission of the author of Charakuru).
* No `ThirdParty/` folder and no `.lib` / `.dll` shipped (the plugin is distributed as source; Fab builds code plugins for each engine version, or the engine tooling does on the user's machine). **No executable is shipped** (no `.exe`). The FBX reader of the bake tool, `ufbx_dump`, is shipped as source only and is built by the user (see "ufbx" below).
* No MetaHuman meshes, textures or face poses. The sample skeleton was re-authored with Unreal Mannequin / MetaHuman-compatible bone names (its root bone is named `metahuman_base_skel`), but it contains no MetaHuman asset data. The face driver outputs ARKit-named animation curves; connecting them to a face rig is done in the user's own project with the user's own assets.
* No data derived from GPL / AGPL / LGPL software.

## Unreal Engine

The plugin is compiled against Unreal Engine 5.8 modules (Core, Engine, RenderCore, RHI, Renderer, OptimusCore / DeformerGraph, ComputeFramework, DeveloperSettings, Sockets, Json, HeadMountedDisplay, InputCore, and in the editor UnrealEd and related modules). Engine code is governed by the Unreal Engine EULA and is not redistributed with the plugin. The plugin requires the engine plugins `DeformerGraph` and `ComputeFramework`, and optionally `NNERuntimeORT` (the built-in webcam tracker; experimental / beta in UE 5.8: check the engine's own `.uplugin`).

## Interfaces and conventions (no code copied)

* Hand landmark order: the 21-point hand model (wrist, thumb, index ... pinky) follows the numbering used by Google MediaPipe Hands (Apache License 2.0). The index order is used as a data convention in the UDP protocol and the hand skeleton.
* The built-in tracker (`Source/SoftPokeNativeTracking`) is original code. It re-implements the pre- and post-processing of the MediaPipe Hand Landmarker graph (anchors, detection decoding, NMS, ROI and landmark projection formulas) from the published behaviour; no MediaPipe source code is copied.

## Bundled model files (Apache License 2.0)

* `Resources/Models/hand_detector.onnx` and `Resources/Models/hand_landmarks_detector.onnx`: Google MediaPipe Hand Landmarker models (palm detection `palmdetector_full_192x192_2021_09_30_v0`, hand landmarks `handskeleton_handflag_handedness_full_2021_10_07_v0`, taken from `hand_landmarker.task`), Copyright Google LLC, Apache License 2.0. **Modified:** converted from TensorFlow Lite to ONNX with tf2onnx 1.17.0 (opset 17); weights unchanged. Licence text: `Licenses/LICENSE_Apache-2.0_MediaPipe_models.txt`; notice: `Resources/Models/NOTICE.txt`. Source: https://ai.google.dev/edge/mediapipe/solutions/vision/hand_landmarker
* They are run by the engine plugin NNERuntimeORT (ONNX Runtime and DirectML ship with Unreal Engine under Epic's terms); SoftPoke ships no ONNX Runtime or DirectML binaries. Camera capture uses Windows Media Foundation (part of Windows).
* ARKit blendshape names (52 facial coefficients, 51 tracked): names are used as animation curve names. No Apple software is included.
* OpenXR hand tracking is accessed through the engine's `IHandTracker` modular feature. No OpenXR SDK code is included.

## Separate downloads (not part of the plugin package)

If the webcam tracker or the bake tool is distributed next to the plugin, they have their own notice files and are installed by the user (for example through `pip`). They are NOT covered by this file. Their dependencies, with the licences found during the 2026-10-13 review: MediaPipe (Apache-2.0), OpenCV wheels (Apache-2.0, bundled FFmpeg is LGPL), NumPy / SciPy / PyPardiso (BSD-3), Numba / llvmlite (BSD-2), fTetWild / pytetwild (MPL-2.0), Intel MKL / oneTBB runtime (Intel Simplified Software License / Apache-2.0), Python (PSF). Nothing in this list is linked into the Unreal plugin binaries. The plugin package ships the bake tool sources in Tools/Bake, including the source of the ufbx library (see "ufbx" below); the packages above are installed by the user (pip) and are not shipped.

## ufbx (third-party source included in the package)

* `Tools/Bake/tools/ufbx_dump/ufbx.c` and `ufbx.h`: the ufbx FBX loader library, Copyright (c) 2020 Samuli Raivio, MIT or public domain (your choice). Unmodified third-party source. Licence text: `Tools/Bake/tools/ufbx_dump/LICENSE_ufbx`. Source: https://github.com/ufbx/ufbx
* `Tools/Bake/tools/ufbx_dump/ufbx_dump.c`: SoftPoke's own small command line program (original code) that uses ufbx to write the raw body file of the bake tool.
* No executable is shipped. The reader is only used to bake from a raw `.fbx` file outside Unreal (`ab_pipeline.py --raw body.fbx`); the user builds it once with one compiler command, see `Tools/Bake/tools/ufbx_dump/BUILD.md`. Baking from a skeletal mesh asset in Unreal or from a VRM file does not use it. It is not part of the Unreal plugin binaries.

## Webcam tracker (Tools/Webcam): packages and models fetched on the user's PC

The package ships the tracker's own source (`tracker.py`, `core.py`, `udp_packet.py`, `setup_env.py`, `tracker_selfcheck.py`, `fetch_models.py`, `replay_landmarks.py`, `requirements.lock`). Nothing below is shipped or bundled: `setup_env.py` has pip download the pinned wheels into a virtual environment the user creates, and `fetch_models.py` downloads the model files from the public Google locations (checksum-verified). Licences as declared in the package metadata of the pinned versions (checked 2026-10-14); no GPL or LGPL package is required.

* mediapipe 1.0.1: Apache-2.0. Model files `hand_landmarker.task`, `face_landmarker.task` and `canonical_face_model.obj` (Google, Apache-2.0).
* opencv-python / opencv-contrib-python 5.0.0.93: Apache-2.0 (OpenCV; the wheels' packaging scripts are MIT). The Windows wheels contain an FFmpeg library under LGPL-2.1, loaded dynamically by OpenCV for video files. It is inside the wheel that pip downloads, unmodified; SoftPoke does not ship, link or modify it. This is the only LGPL component and it is not required for camera input.
* numpy 2.5.3: BSD-3-Clause (with 0BSD, MIT, Zlib and CC0 parts for bundled components); its wheels carry OpenBLAS (BSD-3) and compiler runtime libraries (GPL-3 with the GCC runtime library exception).
* matplotlib 3.11.2 (matplotlib licence, PSF-based), contourpy (BSD-3), cycler (BSD), kiwisolver (BSD-3), fonttools (MIT), pyparsing (MIT), python-dateutil (BSD / Apache-2.0), six (MIT), packaging (Apache-2.0 or BSD-2), pillow 12.3.0 (MIT-CMU / HPND): dependencies of mediapipe.
* absl-py (Apache-2.0), flatbuffers (Apache-2.0), sounddevice (MIT), cffi (MIT-0), pycparser (BSD-3): dependencies of mediapipe.
* certifi 2026.10.19: MPL-2.0 (file-level copyleft, used unmodified).
* The optional developer input of the source tree (`--record3d`, needs the LGPL-2.1 package `record3d`) is not part of the package: its module is not shipped, and the tracker reports a clear error if the flag is used.

## Sample content (Content/SoftPokeSample)

See `SAMPLE_PROVENANCE.md`. In short: an original adult sample character made with Charakuru (by Nano; permission of 2026-10-28), hands from MakeHuman (CC0 1.0), strand hair made by the publisher, AI-generated source images disclosed there. No MetaHuman data, no Epic template content, no hair cards.

## Publisher

Copyright (c) DANTES-ZERO. Distributed under the Fab End User License Agreement (**Personal** or **Professional** licence tier as selected on the listing).
