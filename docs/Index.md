# SoftPoke documentation

> SoftPoke 1.0.0 is for Unreal Engine 5.8 (Windows 64). VR is experimental (the controller path was checked on one headset setup, see [VR](VR.md)) and the built-in webcam tracker runs on an experimental engine plugin; see the [changelog and known issues](ChangelogFaq.md).

SoftPoke makes the skin of a skinned character dent, bulge and slide under your hand in real time. A baked finite-element response is solved on the GPU every frame and applied through a Deformer Graph. You touch the character with your own hand: tracked by a webcam (the tracker is built into the plugin, no Python needed) or, experimentally, by OpenXR hand tracking in VR. Full-body baking is supported. **Adult humanoid characters only.**

SoftPoke is sold as one edition. The Fab licence tier you choose (**Personal** or **Professional**) decides the price; the plugin and its features are the same.

| Page | What is in it |
|---|---|
| [Quick Start](QuickStart.md) | Four parts: (A) the 10-minute path with the sample character and demo maps, (B) your own MetaHuman, (C) any other own character, (D) VR with controllers; "if nothing happens" |
| [Glossary](Glossary.md) | What SPB, SPK, region, bake, quality gate, Groom, NNE, PIE, recenter, proxy hand and the other terms mean |
| [Install and Requirements](InstallRequirements.md) | Engine, platform, GPU, disk, plugin dependencies, project settings |
| [Built-in webcam tracker](NativeTracking.md) | **The webcam input**: `SoftPokeHandSourceNative`, camera set-up, calibration, threads, accuracy and speed, privacy |
| [Webcam tracker guide](WebcamTracker.md) | **Advanced, optional:** the separate Python webcam hand tracker (`Tools/Webcam`): set-up, running, calibration, recording, following the game view, `SoftPokeTrackerLink` |
| [VR (experimental)](VR.md) | OpenXR hand tracking, Valve Index controllers, motion controllers (Quest Touch, Vive, WMR): set-up and what is not yet tested |
| [Bake Your Own Character](BakeYourOwnCharacter.md) | The Bake panel, input checklist, quality gate, the **Set up** button, the VRoid route, the dent overlay |
| [Touch events](TouchEvents.md) | **The "nerve" of a character**: where, how hard, how long and what kind of touch, as JSON over a local WebSocket or a webhook for a local LLM or a script; parts, gestures, settings, security |
| [Hair comb (optional)](HairComb.md) | The hand pushes strand hair (Groom): set-up, settings, body colliders, hair LOD for VR, recommended project settings, how it stays independent of the HairStrands plugin ("Groom" is the asset type, HairStrands the engine plugin) |
| [Component reference](ComponentReference.md) | Every property and function of the components, generated from the source |
| [Behaviour and limits](BehaviourAndLimits.md) | What the limiter does, what is not simulated, tuning, performance |
| [Licences](Licences.md) | Where the licence texts are; the Fab licence tiers |
| [Changelog, known issues, FAQ](ChangelogFaq.md) | Release notes, open issues, answers, support |
| [API and extension](ApiAndExtension.md) | Your own hand source, Blueprint and Python entry points, the UDP protocol v1 specification |

**Optional components (all off by default):** `SoftPokeTouchSensor` (touch events for a local LLM or script, see [Touch events](TouchEvents.md)), `SoftPokeHandMeshDriver` (a visible hand that follows the contact-resolved hand), `SoftPokeHairCollision` (the hand pushes strand hair, hair LOD for VR), `SoftPokeTrackerLink` and `SoftPokeDesktopFly` (view follow, recenter and a free-fly viewpoint for desktop use without a headset). See the [component reference](ComponentReference.md).

AI agent skills that can install, bake and tune for you are in `Extras/AgentSkills/` (README).

## Links

* Publisher: DANTES-ZERO - https://github.com/DANTES-ZERO
* Online documentation: https://github.com/DANTES-ZERO/softpoke-docs
* Support: https://github.com/DANTES-ZERO/softpoke-docs/issues

