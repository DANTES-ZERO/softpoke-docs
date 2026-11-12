# SoftPoke documentation

Documentation for **SoftPoke 1.0.0**, an Unreal Engine 5.8 code plugin by DANTES-ZERO, sold on Fab.

**This repository contains documentation only.** It has no plugin source code, no assets and no binaries. It is not the plugin.

## Overview

**Poke, grab and knead a character's skin with your own hand: a real-time soft-tissue deformation plugin for Unreal Engine 5.8.**

SoftPoke makes the skin of a skinned adult humanoid character dent, bulge and slide under a hand, in real time. A baked finite-element response (skin, fat, muscle) is solved on the GPU every frame and applied through a Deformer Graph. The hand comes from a webcam (built-in tracker, no Python), a video file, your own code, or, in VR (experimental), OpenXR hand tracking or motion controllers.

### What you can do

* **Poke:** a fingertip makes a soft dimple (cheek, forehead, arm, leg).
* **Grab, squeeze, knead and drag:** a whole hand sinks into soft tissue such as a thigh, and the skin slides and follows the hand, with friction.
* **Bone-aware:** skin over bone (kneecap, shin, elbow, collarbone, cheekbone) barely moves while soft tissue next to it gives. Depths are textbook adult averages, found on your character's skeleton at load time.
* Contact ripple and jiggle (optional): a push or a release makes the touched region ring for about 1 second; the wave is small (about 0.5-4 mm at 15-20 cm from the touch on the sample body) and shows in motion.
* **Physics hand:** the hand the solver uses (a proxy hand) is kept within the allowed tissue depth. A visible hand mesh follows it; the visible fingertips can still sink a little in a deep grab.
* Optional hair combing with Groom strand hair; optional touch events (which body part, how hard, what kind of touch) for a local script or LLM.

### What is in the box

* **The SoftPoke code plugin:** C++ runtime, editor and native-tracking modules, GPU compute shaders, Deformer Graph assets.
* **Sample content:** one adult sample character (already baked), natural hand meshes, strand hair, two demo maps (L_DemoPoke, L_DemoHair), desktop and VR pawns.
* **Built-in webcam hand tracker:** two Google MediaPipe hand models (Apache-2.0, ONNX form) run on the engine's NNE.
* Bake panel (Window > SoftPoke Bake) for your own adult characters, optional Groom hair combing, touch events as JSON over localhost (off by default).
* 15 documentation pages ([docs/Index.md](docs/Index.md)) and five AI-agent skills.
* **Not included:** MetaHuman meshes, textures or face poses (the sample skeleton uses MetaHuman-compatible bone names), Epic VR Template content, any third-party character.

### Requirements

* Unreal Engine 5.8 only. Windows 10/11, 64-bit only. DirectX 12 with Shader Model 6.
* **GPU:** NVIDIA RTX is the validated vendor, but only an RTX 4090 was measured. Every other card, NVIDIA or not, is untested.
* **C++ toolchain:** packaging a game always needs one, also for a Blueprint-only project (MSVC v14.44 build tools and Windows 11 SDK 22621; a Visual Studio component list is included). Whether the editor also needs one after a Fab install depends on what Fab delivers to your PC, which was not verified.
* Your own character needs a bake first. The skin must be a separate, watertight mesh without clothes, hair, eyes or teeth. The bake needs 64-bit Python 3.12 and a one-time download of about 1.5 GB; one limb in Draft quality takes about 10 minutes, a whole body up to several hours. The sample character is already baked and the demo maps work without a bake.

The full list of requirements is in [Install and Requirements](docs/InstallRequirements.md).

### VR and video memory

VR is experimental. Valve Index controllers were used on one headset setup (Pimax 8KX at 75 Hz, SteamVR as the OpenXR runtime, RTX 4090, packaged Development build). Which hand input the plugin used there, SteamVR's finger-curl skeleton or the synthesized controller hand, was not recorded. Bare-hand OpenXR tracking, other headsets and a Quest 3 are untested.

Video memory of the game process in the packaged VR demo, peak of a tour: 5.6 GB with the low-VRAM profile off (3.6 MP per eye), 2.7 GB with the 8 GB profile (3.6 MP per eye), 2.3 GB with the 6 GB profile (2.5 MP per eye); the SteamVR compositor needed about 2.6 GB more on the Pimax 8KX (other runtimes differ). The low-VRAM profiles apply to the demo maps only. Not verified on a real 8 GB or 6 GB card. DLSS is optional and not shipped.

### Limits

* **Version 1.0.0:** interfaces and documentation can still change.
* **Webcam:** a phone used as a webcam was tested, a physical USB webcam was not; depth is approximate; the engine's NNERuntimeORT plugin is experimental in UE 5.8.
* A small-deformation model, not a medical simulation. Separate clothes do not deform with the skin; only the hand touches the skin; the face follows the head bone only.
* Adult humanoid characters only.
* The full list of what was and was not verified is in [Changelog, known issues, FAQ](docs/ChangelogFaq.md).

### AI-generated content

Created with AI. The sample character (reference images, 3D mesh, textures) was made with generative AI tools (image generation, then image-to-3D with Tripo inside Charakuru, with the author's written permission to bundle it) and cleaned and re-rigged by the publisher. The plugin's code and documentation were developed with AI assistance. The preview images and videos were captured in a test project with scripted, synthetic hand motion; some images are composites and one is a diagram. Details: [Licenses/SAMPLE_PROVENANCE.md](Licenses/SAMPLE_PROVENANCE.md) and [Licenses/THIRD_PARTY_NOTICES.md](Licenses/THIRD_PARTY_NOTICES.md).

### Content policy

The sample character is a clothed adult; SoftPoke is for adult humanoid characters only. Support: see the Support section below.

## Where to get SoftPoke

Available on Fab (search for SoftPoke by DANTES-ZERO).

## Documentation

Start with [docs/Index.md](docs/Index.md).

| Page | What is in it |
|---|---|
| [Index](docs/Index.md) | Overview and links to all pages |
| [Quick Start](docs/QuickStart.md) | The 10-minute path with the sample character, your own MetaHuman, any other own character, VR with controllers, "if nothing happens" |
| [Glossary](docs/Glossary.md) | What SPB, SPK, region, bake, quality gate, Groom, NNE, PIE, recenter, proxy hand and the other terms mean |
| [Install and Requirements](docs/InstallRequirements.md) | Engine, platform, GPU, disk, plugin dependencies, project settings |
| [Built-in webcam tracker](docs/NativeTracking.md) | The webcam input: set-up, calibration, threads, accuracy and speed, privacy |
| [Webcam tracker guide](docs/WebcamTracker.md) | Advanced, optional: the separate Python webcam hand tracker |
| [VR (experimental)](docs/VR.md) | OpenXR hand tracking, controllers, set-up and what is not yet tested |
| [Bake Your Own Character](docs/BakeYourOwnCharacter.md) | The Bake panel, input checklist, quality gate, the Set up button, the dent overlay |
| [Touch events](docs/TouchEvents.md) | Where, how hard, how long and what kind of touch, as JSON over a local WebSocket or a webhook |
| [Hair comb (optional)](docs/HairComb.md) | The hand pushes strand hair (Groom): set-up, settings, body colliders, hair LOD for VR |
| [Component reference](docs/ComponentReference.md) | Every property and function of the components |
| [Behaviour and limits](docs/BehaviourAndLimits.md) | What the limiter does, what is not simulated, tuning, performance |
| [Licences](docs/Licences.md) | Where the licence texts are; the Fab licence tiers |
| [Changelog, known issues, FAQ](docs/ChangelogFaq.md) | Release notes, open issues, answers, support |
| [API and extension](docs/ApiAndExtension.md) | Your own hand source, Blueprint and Python entry points, the UDP protocol v1 specification |

The documents in `Licenses/` and `Extras/BuildTools/` are copies of files that ship with the plugin, included so that the links in the documentation resolve. Some pages also name other files by their path inside the plugin package (for example `Extras/AgentSkills/`); those files are not in this repository.

## Support

Use the Issues tab of this repository. Please include:

* the SoftPoke and Unreal Engine versions,
* your GPU and driver,
* the headset, if the problem is about VR,
* an excerpt of `Saved/Logs/<Project>.log` (the `LogSoftPoke` lines), and for bake problems the files `manifest.json` and `quality_report.json` from the bake work folder.

Do not attach models you have no right to share. Support covers the unmodified plugin and the supported matrix (Windows 64, UE 5.8, D3D12 SM6, NVIDIA RTX validated, adult humanoid characters). It does not include custom bakes. See also [SUPPORT.md](SUPPORT.md).

## Copyright

Copyright (c) 2026 DANTES-ZERO. All rights reserved.

This documentation is not open-licensed. No licence to copy, modify or redistribute it is granted. The plugin itself is licensed through Fab; see [Licences](docs/Licences.md).
