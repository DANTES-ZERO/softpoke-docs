# Install and Requirements

> SoftPoke 1.0.0 | [Index](Index.md)

## Requirements

| Item | Requirement |
|---|---|
| Engine | Unreal Engine **5.8** only (the plugin uses engine-internal Deformer Graph and render-graph interfaces that change between versions). |
| Platform | **Windows 10/11, 64-bit** only (`SupportedTargetPlatforms`: Win64). No Linux, Mac, console or mobile. |
| Graphics API | **DirectX 12 with Shader Model 6.** All 57 compute kernels are compiled for SM6 PC platforms only. With DirectX 11 or SM5 the solver switches itself off with one warning and the mesh stays undeformed (no crash). |
| GPU | **NVIDIA RTX** is the validated vendor. AMD and Intel GPUs: **not tested (expected to work on DX12 SM6 GPUs, not validated)**. |
| Engine plugins | `Deformer Graph` and `Compute Framework` (declared dependencies; enabling SoftPoke enables them). For OpenXR hand input also `OpenXR` and `OpenXR Hand Tracking` ([VR page](VR.md)). For the built-in webcam tracker: `NNERuntimeORT` (the engine's ONNX Runtime plugin for the Neural Network Engine, NNE; it is **experimental / beta in UE 5.8**, declared as an optional dependency of SoftPoke and enabled with it; without it the rest of SoftPoke works but the built-in webcam tracker does not). For the optional `py` console commands that these docs use to drive the editor from Python: `Python Editor Script Plugin` (SoftPoke's own code does not use it). |
| Webcam input | A webcam. The built-in tracker needs nothing else, **no Python** ([Built-in webcam tracker](NativeTracking.md)). Advanced option: a separate Python tracker ([Webcam tracker guide](WebcamTracker.md)). Not needed for VR hand tracking or if you drive the hand yourself. |
| C++ toolchain | **Needed to package a game, always, also for a Blueprint-only project** (a project that enables a code plugin such as SoftPoke is built with a small temporary C++ target). Visual Studio or Build Tools with a UE 5.8-accepted MSVC compiler and a Windows SDK: see [Build tools for packaging](#build-tools-for-packaging). **Also needed** if you copy the plugin source into your project's `Plugins` folder. **Whether the editor needs one after a Fab / Epic Games Launcher install depends on what the launcher delivers:** Fab builds code plugins for each engine version, but whether a built copy or the source reaches your PC was not verified by the publisher. **It is needed** if the editor reports missing modules or asks to compile the plugin: see [Install](#install). |
| Baking your own character | An internet connection and a 64-bit **Python 3.12** from python.org for the **Set up** button; about 1.5 GB for the CPU packages, plus about 2 GB for the optional GPU packages; 1-3 GB of work space per body. See [Bake Your Own Character](BakeYourOwnCharacter.md). |

**Minimum hardware:** not yet defined. Only one GPU model (an RTX 4090) has been measured; other GPUs have not been tested. Treat any other card as untested.

## VRAM and disk

* The baked data of one character is roughly 90-125 MB of `.spk` data; the runtime keeps regions resident on demand within `VramBudgetMB` (default 640 MB; the sample body needs 340 MB when every region is resident).
* A bake needs 1-3 GB of scratch space per body in the work folder.
* The first start compiles the SoftPoke shaders (several minutes in the editor); the result is cached.

## Install

**Which way did you get the plugin?** Fab builds code plugins from source. What you need on your PC depends on how you install it.

| Way | What happens | C++ toolchain on your PC |
|---|---|---|
| **From Fab / the Epic Games Launcher** ("Install to Engine") | Fab builds code plugins for each engine version and the launcher installs the result. What exactly reaches your PC (a built copy or the source) was not verified by the publisher. | **Not needed if the editor loads the plugin without compiling. Needed if the editor reports missing modules or asks to compile the plugin. Needed to package a game** (last row). |
| **Copy the plugin source** into `<YourProject>/Plugins/SoftPoke` | The editor asks to compile the plugin the first time the project opens (5-15 minutes). | **Needed:** Visual Studio or Build Tools with a UE 5.8-accepted MSVC compiler, see [Build tools for packaging](#build-tools-for-packaging). A Blueprint-only project becomes a C++-capable project this way. |
| **Package a game** (Platforms > Windows > Package Project), any project, also Blueprint-only | The build compiles a small temporary C++ target for the project (see [Build tools for packaging](#build-tools-for-packaging)). | **Always needed.** |

**Where is the plugin folder?** These docs write `Plugins/SoftPoke/...` for the SoftPoke plugin folder. For a copy it is `<YourProject>/Plugins/SoftPoke`. For a launcher install it is a folder that the launcher chose (under the engine or under your project; the exact location was not verified): if you cannot find it, search your drives for `SoftPoke.uplugin`. A plugin folder inside the engine is often read-only: copy files that you want to run or edit (for example `Tools/...`) to a writable folder.

Steps:

1. Install the plugin by one of the ways above. After a launcher install the plugin appears in **Edit > Plugins** in every UE 5.8 project; for a copy, open the project and let the editor compile when asked.
2. **Edit > Plugins**, enable **SoftPoke**, restart.
3. Check the log: `LogSoftPoke` lines at start-up and no module load error.

## Rendering settings

This is the one place that describes the render settings; the other pages refer to it.

* **The demo maps work in a default project** (deferred shading with TSR and Lumen). They do **not** require forward shading. You do not have to change any render setting to try them.
* **Demo maps are self-contained:** each map has an unbound **PPV_Exposure** post process volume with manual exposure (exposure compensation -2.6, no physical camera exposure), no dynamic global illumination, no reflection method, local exposure 0.8, no ambient occlusion and no motion blur, so the look does not depend on your project's defaults (auto exposure, Lumen, the luminance range setting). The lights are movable: a sky light from an engine cubemap, a soft directional sun, a soft key point light **F_Key** (shadows), a frontal **SoftPokeFillLight** without shadows and **HairFill** for the hair (lighting channel 1 only, 6 cd; the hair materials have a darker copper base colour and little back-light, so the hair does not wash out to pale orange or white). A grey floor (**DemoFloor**, `MI_DemoFloor`) and a sky without a sun disc complete the set. The same maps look alike in a blank deferred project and in a forward-shaded VR-style project.
* **Strand hair (Groom) and forward shading.** The project the demo maps were made in uses forward shading with MSAA (`r.ForwardShading=True`, as VR projects do). With forward shading, strand hair is lit **only by shadow-casting lights**, and at most 4 of them can reach the head (count the sun too); a light without shadows never lights the hair. In a default deferred project the hair is lit by all lights, so it may look slightly different from the screenshots (shading and shadows of the strands). Nothing needs to be fixed.
* Lumen is not available together with forward shading (an engine limit). If you want Lumen, use the default deferred settings. For VR the default deferred settings with TSR work: in a headset test (instanced stereo off) the sample hair looked right. Forward shading with MSAA is not required, and in our tests it made the sample's strand hair much darker (the back of the head nearly black). With **Instanced Stereo** on, strand hair under TSR was drawn in the wrong place in `-emulatestereo`; this was not checked on a headset.
* Optional hair settings for either mode: see [Recommended project settings](HairComb.md#recommended-project-settings).

## Project settings

* **Rendering API.** Project Settings > Platforms > Windows > **Default RHI = DirectX 12**; the targeted shader format must include SM6. `-dx12` on the command line also forces it. `-dx11` gives an undeformed mesh by design.
* **SoftPoke** (Project Settings > Plugins > SoftPoke): `UdpBindAddress`, default `127.0.0.1` - the plugin listens for the tracker on this PC only. Use `0.0.0.0` only if the tracker runs on another machine; then UDP port 9763 is reachable from the network, so use a private network and allow it in the Windows firewall.
* **SoftPoke Bake Tool** (Project Settings > Plugins > SoftPoke Bake Tool; stored per user): `PythonExecutable`, `SetupPythonExecutable`, `BakeToolDirectory`, `WorkDirectory`. The **Set up** button fills these in.
* **Bone influences.** Characters with many bone influences per vertex (rich face rigs) can need `r.GPUSkin.Support16BitBoneIndex=1` and `r.GPUSkin.UnlimitedBoneInfluences=1`. Changing them recompiles shaders.

## Packaging a game

Cook for **Windows, DirectX 12 (SM6)**. Before cooking: the skeletal mesh asset must list the SoftPoke deformer in `TargetMeshDeformers` (the `assign_body` command does this); set the default map in Project Settings > Maps & Modes (a Shipping build ignores `-map`); the level's actor carries a `SoftPokeComponent` with its body asset. Packaging needs a C++ compiler on your PC, also for a Blueprint-only project: see [Build tools for packaging](#build-tools-for-packaging). A packaged game reads the cooked `.uasset` files. The only files outside them are the two ONNX hand models of the built-in webcam tracker (`Resources/Models`, staged next to the game by the plugin's build rules; a packaged run with a physical webcam is not yet tested, see [Built-in webcam tracker](NativeTracking.md#status-of-testing)). The plugin's `Config/Game.ini` adds `/SoftPoke/Input` (the controller input assets, which are loaded by path) to the project's directories to always cook. To package a demo map as a first test: set **Game Default Map** (and **Editor Startup Map**) to `L_DemoPoke` in Project Settings > Maps & Modes (the map is plugin content: if the picker does not list it, enable the plugin-content filter of the picker, or copy the map into your project's `Content` folder), then **Platforms > Windows > Package Project** with a Development build first. The skill `softpoke-ops` in `Extras/AgentSkills/` has the full checklist.

## Build tools for packaging

**When a compiler is needed.** **Packaging a game always needs one**, also for a project that only uses Blueprints: a project that enables a code plugin outside the engine's default set (SoftPoke, OpenXR) is built with a small temporary C++ target when it is packaged. For use in the editor after a Fab / launcher install a compiler is needed only if the editor reports "Missing Modules" for SoftPoke or asks to compile the plugin. In that case install the components below, restart the editor and let it compile. If the message stays, a plugin folder inside the engine is often read-only: copy the plugin folder to `<YourProject>/Plugins/SoftPoke`, open the project and compile it there; if that also fails, send us `Saved/Logs/<Project>.log` (see Support in the [changelog page](ChangelogFaq.md#support)).

**Manual steps (Visual Studio Installer).**

1. Open the **Visual Studio Installer**, or download **Build Tools for Visual Studio 2022** or **Visual Studio 2022 / 2026** from Microsoft.
2. Click **Modify** (or **Install**) and tick the workload **Desktop development with C++**.
3. On the **Individual components** tab, tick:
   * `MSVC v143 - VS 2022 C++ x64/x86 build tools (v14.44-17.14)`
   * `Windows 11 SDK (10.0.22621.0)` (Unreal Engine 5.8 accepts any Windows SDK from 10.0.19041.0 up)
4. Install. You do not need to install .NET separately; Unreal Engine 5.8 ships its own.

**Import the component list instead.** The plugin ships a Visual Studio configuration file with these two components: `Extras/BuildTools/SoftPoke.vsconfig` (in the SoftPoke plugin folder). In the Visual Studio Installer choose **More > Import configuration**, pick that file, then install. We have not verified that these two components alone are enough on a clean PC, so if packaging reports a missing component, also tick the workload **Desktop development with C++** as in the manual steps.

**Compiler version rule.** Unreal Engine 5.8 checks the version of the MSVC compiler, not only the Visual Studio version. It needs at least 14.38.33130 and refuses MSVC 14.39 to 14.43, 14.44.0 to 14.44.35210 and 14.50.0 to 14.50.35722 (known compiler bugs). The 14.44 component above is the one Epic recommends for Visual Studio 2022.

**Visual Studio 2026 18.0 to 18.2:** the default compiler in these versions (MSVC 14.50.35717) is refused by Unreal Engine 5.8. Either add the 14.44 component above, or update Visual Studio to a version that includes MSVC 14.50.35723 or newer.

**License.** Visual Studio Community and Build Tools have license terms from Microsoft. Check that they cover your use before installing (<https://visualstudio.microsoft.com/vs/community/>, <https://visualstudio.microsoft.com/downloads/>).

**First package.** When you package a game with SoftPoke enabled, the build compiles a small stub and links the game, and it may also compile the plugin's C++ code. In our test (plugin in the project's `Plugins` folder) this build step took about one minute. This is normal. If the build stops with an error about Visual Studio, the MSVC toolchain or the Windows SDK, check your versions against the rule above.

## What the plugin starts or opens

* The Bake panel (editor only) starts the `python.exe` that you set in Project Settings > SoftPoke Bake Tool, for **Set up** and for bakes; pip then downloads packages ([Bake Your Own Character](BakeYourOwnCharacter.md)).
* `SoftPokeTrackerLink` can start the optional Python webcam tracker with the Python that you give it when the game starts, and stops it when the game ends. Off by default (`bLaunchTracker`).
* The plugin listens for hand data on UDP `127.0.0.1:9763` (`UdpBindAddress`). Only if you switch the touch events on it also runs a WebSocket server on `127.0.0.1:9764` and/or posts a webhook to this PC ([Touch events](TouchEvents.md)).
* The plugin never connects to the internet by itself ([FAQ](ChangelogFaq.md#faq)).

## Uninstall

Remove the plugin folder (or disable the plugin) and delete the assets that reference it: `SPB_*`, `SPK_*`, `SPKP_*`, `SPF_*`, `DG_SoftPoke*`. The Python environment the **Set up** button created lives in `<Project>/Saved/SoftPokeBake/venv` (or the folder you chose) and can be deleted at any time.
