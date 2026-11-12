# Glossary

> SoftPoke 1.0.0 | [Index](Index.md)

Short explanations of the words used in this documentation, in alphabetical order.

| Term | Meaning |
|---|---|
| **Bake** | The one-time calculation (on your PC, about 10 minutes for one limb in Draft quality, hours for a whole body) that turns your character's skin mesh into SoftPoke data. It is done from the **Window > SoftPoke Bake** panel. The sample character is already baked. |
| **Bone map** | The table that tells the bake tool which bone of your skeleton is the upper arm, the head and so on. `auto` works for UE5 / MetaHuman, Mixamo / Rigify-style and VRM names. |
| **Deformer Graph** | An engine asset (plugin *Deformer Graph*) that lets a graph of GPU programs change a skeletal mesh every frame. SoftPoke's graph is `DG_SoftPoke`; it is assigned to the mesh and does the actual denting. The Bake panel creates it when it is missing. |
| **Display mesh** | The skeletal mesh you see in the level. The baked data is mapped onto it. It can be the baked mesh itself or a finer copy of it. |
| **FEM** | Finite-element method: the way the bake calculates how skin, fat and muscle give way. You never run it yourself; the bake does. |
| **Gate** | See *Quality gate*. |
| **Groom** | The engine asset type for strand hair (and its component, the Groom component). "HairStrands" is the name of the engine plugin that renders and simulates a Groom. This documentation says **Groom** for the asset and the feature, and **HairStrands** only for the plugin name or the console variables (`r.HairStrands.*`). |
| **Hand rig** (`SoftPokeHandRig`) | The component on the character's actor that receives the hand from a *hand source* and gives it to SoftPoke. |
| **Hand source** | Where the hand comes from: `SoftPokeHandSourceAuto` (the default: OpenXR hand tracking, else VR controllers, else the fallback below), `SoftPokeHandSourceController` (VR controllers), `SoftPokeHandSourceNative` (the built-in webcam tracker, the normal choice), `SoftPokeHandSourceUdp` (the optional Python tracker, see [Webcam tracker guide](WebcamTracker.md)), `SoftPokeHandSourceOpenXR` (VR, experimental), or your own ([API and extension](ApiAndExtension.md)). |
| **Hand Size Cm** | A setting of the native hand source: the distance from the wrist crease to the base of the middle finger, in centimetres. `0` means an average adult hand. The webcam cannot measure the depth of a hand by itself, so it uses this size. |
| **Mochi / stylize** | An optional soft rim around the hand and ridges between the fingers that makes the flesh look softer. Off at `StylizeStrength` 0. |
| **NNE / NNERuntimeORT** | Unreal's Neural Network Engine and its ONNX Runtime plugin. The built-in webcam tracker runs its hand models through it. In UE 5.8 the plugin is marked experimental / beta by Epic. |
| **PIE** | Play In Editor: the **Play** button in the editor toolbar. The solver runs in PIE, Simulate and game builds, not in the plain editor viewport. Changes you make to objects during PIE are lost when you stop. |
| **Proxy hand** | The hand SoftPoke actually uses for contact. A tracked hand can go through the body, so the *physics hand* limiter gives the solver a proxy hand that never sinks deeper than the flesh allows. A visible hand mesh should follow the proxy hand ([SoftPokeHandMeshDriver](ComponentReference.md#softpokehandmeshdriver)). |
| **Quality gate** | The check the bake tool runs on your mesh before baking (closed surface, scale, weights, bone names, ...). Result: **PASS** (baked), **WARN** (warnings, nothing baked yet: read them, fix or accept), **FAIL** (errors you must fix). See [Bake Your Own Character](BakeYourOwnCharacter.md#the-quality-gate). |
| **Recenter** | Telling the tracker "my hand is at its resting place now" (press **C** in the demo maps, or call `Recenter()`). Do it with the hand where it should rest in front of the camera. |
| **Region** | A part of the body that is baked on its own (`forearm_L`, `thigh_L`, `hand_L`, `face`, ...). The right side is made by mirroring the left. The bake tool lists the regions it finds. |
| **SPB** (`SPB_<name>`) | The **body asset**: the main baked asset of a character. It lists the regions and points to the other files. The `SoftPokeComponent` on the actor references it. |
| **SPF** (`SPF_*`) | The **skin-fill** asset: extra baked data that lets the flesh rise around the fingers. |
| **SPK** (`SPK_*`) | The **region asset**: the baked response of one region (the compressed result of the FEM calculation). |
| **SPKP** (`SPKP_*`) | The **payload** of a region: the large data block that belongs to one SPK region asset. SPK and SPKP always come as a pair. |
| **Status** | A read-only text on many SoftPoke components and sources that tells you what they found or what went wrong (camera format, missing plugin, ...). It is the first thing to read when something does not work. |
| **`SoftPoke.Cmd`** | The editor command with SoftPoke's verbs (`list_regions`, `prepare_metahuman`, `assign_body`, ...). Type it into the **Cmd** box at the bottom of the **Output Log** window (not into the Python console). |
| **Target actor** | In the Bake panel, the level actor that shows the character. The result is assigned to it. |
| **Watertight** | A mesh without holes: every edge belongs to exactly two faces. The skin you bake must be watertight, apart from the natural openings (wrists, ankles, neck and, in the head, eyes, mouth, nostrils, ears). |
