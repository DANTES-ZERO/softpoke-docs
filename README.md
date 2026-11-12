# SoftPoke documentation

Documentation for **SoftPoke 1.0.0**, an Unreal Engine 5.8 code plugin by DANTES-ZERO, sold on Fab.

**This repository contains documentation only.** It has no plugin source code, no assets and no binaries. It is not the plugin.

## What SoftPoke is

SoftPoke makes the skin of a skinned character dent, bulge and slide under your hand in real time. A baked finite-element response is solved on the GPU every frame and applied through a Deformer Graph. You touch the character with your own hand: tracked by a webcam (the tracker is built into the plugin, no Python needed) or, experimentally, by OpenXR hand tracking in VR. Full-body baking is supported. Adult humanoid characters only.

## Requirements (summary)

* Unreal Engine 5.8 only.
* Windows 10/11, 64-bit only.
* DirectX 12 with Shader Model 6. NVIDIA RTX is the validated GPU vendor; AMD and Intel GPUs are not tested.
* A webcam for the built-in tracker (not needed for VR hand tracking or if you drive the hand yourself).
* A C++ toolchain is needed to package a game, and whenever the plugin is compiled on your PC.
* Baking your own character needs an internet connection and 64-bit Python 3.12 for the **Set up** button.

The full list is in [Install and Requirements](docs/InstallRequirements.md).

## Where to get SoftPoke

On Fab: [FAB LISTING URL]

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
