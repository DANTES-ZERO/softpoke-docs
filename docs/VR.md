# VR (experimental)

> SoftPoke 1.0.0 | [Index](Index.md)

> **Experimental.** The OpenXR hand-tracking path and the motion-controller path are implemented. The motion-controller path has been run on **one headset setup**: a packaged Development build of the demo map `L_DemoPoke` (Windows, DirectX 12) with a Pimax 8KX at 75 Hz through SteamVR as the OpenXR runtime, Valve Index controllers and an RTX 4090. Touching, the contact ripple and the Esc quit were checked there, and SteamVR reported 0 dropped and 0 reprojected frames in every run (2 to 5 minutes each). **The OpenXR hand-tracking path without controllers (bare hands) and other headsets and runtimes have not been tested.** The penetration limiter, the joint limits and the hand-to-capsule conversion were tested with recorded and synthetic hand data, and the controller path also with synthetic controller poses and trigger / grip values injected through a test hook; a physical hand driven over UDP works. Please treat VR as "supported, experimental": it may need adjustments on your hardware (see [what to check](#what-to-check-on-your-hardware)). The webcam path ([Webcam tracker guide](WebcamTracker.md)) is the one that has been used in practice.

## Which input do you get

| Hardware | Input | How |
|---|---|---|
| Valve Index controllers (SteamVR as the OpenXR runtime) | The finger-curl skeleton that SteamVR derives from the controllers' finger sensors, through OpenXR hand tracking (`XR_EXT_hand_tracking`) | `Auto` (default) uses it as an OpenXR hand. Valve Index controllers via SteamVR OpenXR hand tracking: supported, **not yet tested on a headset**. If SteamVR does not report hands for the controllers, `Auto` uses the controller path below. |
| Quest Touch, Vive wands, WMR controllers, Index without hand tracking, any OpenXR controller | A hand **synthesized** from the controller pose and the trigger / grip / thumb-touch values | `Auto` (default) falls back to `SoftPokeHandSourceController`; or set the rig's **Source Class** to it. See [Motion controllers](#motion-controllers). |
| Optical hand tracking (Quest hands, Leap, ...) | OpenXR hand tracking | `Auto` or `SoftPokeHandSourceOpenXR`. |
| No headset | Webcam / video / UDP | Set **Source Class** to `SoftPokeHandSourceNative`, or leave `Auto` and set **Auto Fallback Class** to it. |

`SoftPokeHandSourceAuto` (the default **Source Class** of the rig) chooses per hand: OpenXR hand tracking if it is valid, else the motion controller of that hand if it is tracked, else the fallback source. The rig's `Status` text names the source of each hand, for example `Auto: L controller, R controller | OpenXR: 1 hand tracker(s), left -, right - | controller: left tracked index_controller (trig 0.00 grip 1.00 thumb 0.00), ... [input: Enhanced Input]`. A hand changes its source only after the better one has been tracked for `SwitchDebounceSeconds` (0.25 s), and the rig restarts that hand's filter and limiter at the switch.

The engine does not tell SoftPoke whether OpenXR joints come from optical tracking or from the finger sensors of a controller (UE 5.8 does not request `XR_EXT_hand_tracking_data_source`). Nothing in SoftPoke rejects either: the joints are used the same way. With an Index profile the status text adds "(Valve Index: finger-curl joints from SteamVR)" as a hint.
>
> Release hardware statement: Windows 64, DirectX 12, NVIDIA RTX validated; AMD / Intel: not tested (expected to work on DX12 SM6 GPUs, not validated).

## How VR input works

`SoftPokeHandSourceOpenXR` (used by `Auto`, or on its own) reads the 26 hand joints that the engine's hand-tracking feature provides (`IHandTracker`, OpenXR extension `XR_EXT_hand_tracking`), converts them to the 21-joint hand model used by SoftPoke, and the rig turns those into capsules for the solver. With SteamVR, finger joints are derived from the Valve Index controllers' finger sensors through the same extension. The plugin links to no OpenXR SDK; it only uses the engine's modular feature.

## Set up

1. Engine plugins: enable **OpenXR** and **OpenXR Hand Tracking** (Edit > Plugins) and restart.
2. Make your headset's runtime the active OpenXR runtime (for SteamVR: SteamVR settings > Developer > "Set SteamVR as OpenXR runtime"). Start the runtime and put the headset on or wake it.
3. Add a `SoftPokeHandRig` to the character's actor. Leave **Source Class** at `SoftPokeHandSourceAuto` (OpenXR hands, else controllers), keep **Left Hand** and **Right Hand** on. To use one input only, set **Source Class** to `SoftPokeHandSourceOpenXR` or `SoftPokeHandSourceController`.
4. Play in **VR Preview**. The rig's `Status` text starts with the source's state, for example `OpenXR: 1 hand tracker(s), left tracked, right -` (a dash = that hand is not tracked), followed by `| L free R -` (`free` = tracked and not limited, `limited` = the limiter is holding the hand back). `OpenXR: 0 hand tracker(s)` means that no runtime provides hand tracking: check steps 1 and 2 and whether the runtime sees your fingers.
5. Turn on `bDrawDebug` on the rig to see the tracked capsules (blue) and the limited capsules (green; only where they differ).

Source option: `bAcceptUntracked` keeps the last known joints while the runtime reports "not tracked" (off = the hand is dropped at once).

## Motion controllers

`SoftPokeHandSourceController` takes the **grip pose** of each controller (`IXRTrackingSystem::GetMotionControllerState`, world space) and synthesizes the 21-joint hand from the controller inputs:

| Input | Hand |
|---|---|
| Trigger (analog) | Index finger curl |
| Grip (analog; a digital grip click counts as 0 / 1) | Middle, ring and little finger curl |
| Trigger released **and** grip held | The index finger points (stretched): poke with it |
| Thumb touch: thumbstick, trackpad, A / B (X / Y on the left Touch controller) touch | Thumb curl (a small share also follows the grip) |

The fingers move to the commanded curl at `FingerCurlSpeed` (curl units per second), not in one frame. The synthesized hand has the proportions of an average adult hand (`HandScale` scales it) and stays inside the same anatomical joint limits as a tracked hand (checked over the whole input range: at most 1 mm of correction, then the rig applies the limits to it again). The palm centre of the virtual hand sits at the grip pose origin; `ProfileTweaks` (per OpenXR interaction profile, matched by a part of the profile path) and `GlobalTweak` shift and rotate it relative to the grip pose. **The defaults are zero offsets: they have not been calibrated on a headset.** If the virtual hand floats off your real hand, adjust `PalmOffsetCm` / `RotationOffset` there.

From the rig on, the hand behaves exactly like a tracked hand: poke, grab, squeeze and comb, the penetration limiter, the joint limits, the visible hand mesh (`SoftPokeHandMeshDriver`) and the hair collider. The data of controllers is clean, so the rig's One-Euro filter (made for webcam jitter) is skipped for this source (`bRigFilter` = on to filter anyway). **The fingers are not tracked**, only the controller is: finger motion that is not on the trigger / grip / thumb touch is not reproduced.

**Input assets.** The inputs come from Enhanced Input actions shipped in the plugin's content (`SoftPoke Content > Input`; enable **Show Plugin Content**): `IA_SoftPoke_Trigger_L / _R`, `IA_SoftPoke_Grip_L / _R`, `IA_SoftPoke_ThumbTouch_L / _R` (Axis 1D), and one mapping context per OpenXR interaction profile: `IMC_SoftPoke_ValveIndex`, `IMC_SoftPoke_OculusTouch`, `IMC_SoftPoke_Vive`, `IMC_SoftPoke_WMR`, plus `IMC_SoftPoke_Generic` (the union of the four; use it instead of the four if you prefer one context). The source adds the four profile contexts to the first local player when it starts (priority `MappingPriority`); edit the assets to rebind, or assign your own actions / contexts on the source (`TriggerAction`, `GripAction`, `ThumbTouchAction`, `MappingContexts`). If the assets or Enhanced Input are not available (for example a project that uses the legacy input classes), the same keys are read raw from the player controller (`bFallbackToRawKeys`); the status text says `[input: Enhanced Input]` or `[input: raw keys]`. Controllers of other profiles (PICO, Reverb G2, ...) are tracked, but their buttons drive the fingers only if their keys are mapped to the six actions. Quest controllers used without hand tracking go through the Oculus Touch profile. These assets are loaded by path at run time, so nothing in your project references them; the plugin's `Config/Game.ini` adds `/SoftPoke/Input` to **Project Settings > Packaging > Additional Asset Directories to Cook** for you, and a packaged game contains them without any step on your side. If a packaged game still logs "the Enhanced Input assets ... were not found" (a project that clears that array in its own `DefaultGame.ini`), add `/SoftPoke/Input` to the list yourself.

**Haptics.** With `bHapticsOnContact` (default on) the controller gets a short, weak pulse (`HapticPulseSeconds` 0.06 s; amplitude `HapticBaseAmplitude` 0.08 plus `HapticDepthAmplitude` 0.25 times the press depth as a fraction of the allowed depth, never above `HapticMaxAmplitude` 0.35) when the hand touches the skin, and again when it presses clearly deeper. It uses `APlayerController::SetHapticsByValue`, re-issued every frame for the length of the pulse (OpenXR plays one call for one frame only). With OpenXR hand tracking active (for example Valve Index in SteamVR), the pulse also goes to the controller held in that hand. `Status` shows `haptic pulses L n R m` so you can see whether a contact fired. The strength is not tuned across headsets; adjust the amplitudes to taste.

**Tracking-to-world.** The poses are in the engine's world space, so the character, the camera and the VR origin must share it: use a pawn with a camera (as the VR Template's pawn does) and move the tracking origin by moving that pawn. The demo maps do this for you with the [demo VR pawn](#demo-vr-pawn) (run on one headset setup, see the note at the top).

## Demo VR pawn

The two demo maps (`L_DemoPoke`, `L_DemoHair`) use `BP_SoftPokeDemoGameMode` (class `SoftPokeDemoGameMode`). When a headset is enabled (the engine's HMD is active: VR Preview, or a VR game launch with a running OpenXR runtime) it spawns `BP_SoftPokeDemoVRPawn` (class `SoftPokeDemoVRPawn`, SoftPoke Content > SoftPokeSample > Demo); otherwise it spawns the desktop pawn as before. The console variable `softpoke.Demo.VR` overrides this: `-1` automatic (default), `0` always the desktop pawn, `1` always the VR pawn (testing without a headset: the camera sits 160 cm above the floor origin).

| Part | What it does |
|---|---|
| `VROrigin` > `Camera` | The tracking space. The camera is locked to the headset. Tracking origin: **floor** (`LocalFloor`; `bFloorTrackingOrigin`), so your real floor is the demo floor. |
| `MotionControllerLeft` / `MotionControllerRight` | `MotionControllerComponent`s with motion source `Left` / `Right` (use them to attach your own things; the hands come from the character's `SoftPokeHandRig`: OpenXR hand tracking, else the synthesized controller hands, via `SoftPokeHandSourceAuto`). |
| Start | Floor level, `VRReachDistanceCm` (65 cm) from the character, facing her; set on the game mode (`FocusLocation`, `VRReachDistanceCm`, `bPlaceVRStart`). Face, cheek and the outside of the thigh are within reach without walking. |
| Left thumbstick / trackpad | Walk (`MoveSpeedCmPerSec` 110, `MoveDeadzone` 0.2), relative to where you look. |
| Right thumbstick / trackpad, left-right | Snap turn (`SnapTurnDegrees` 30, threshold 0.7, re-arm 0.3) about your head. A gamepad's sticks work the same (for testing). |

In a packaged or standalone run of the demo maps (not in the editor, where PIE and VR Preview already end on Esc) the Esc key quits the game; turn this off with `bEscapeQuitsGame` on the demo game mode. A keyboard key only reaches the game while its desktop window has focus, so click the game window first if Esc does nothing. Keys are read raw from the player controller (Oculus Touch, Valve Index, Windows MR and Vive thumbstick / trackpad axes), so no input asset is needed; the Vive trackpad moves you while it is touched. The desktop fly component of the sample (WASD) is switched off while a headset is used. There is no teleport and no comfort vignette: if smooth walking is uncomfortable lower `MoveSpeedCmPerSec` or set `bThumbstickMove` off and use snap turns only. In your own map: set the game mode to `SoftPokeDemoGameMode` (or a Blueprint of it), fill `VRPawnClass` and `DesktopPawnClass`, or put `BP_SoftPokeDemoVRPawn` into the level and possess it. **Render budget.** Only the demo game mode limits the render resolution (the plugin never changes a game's own settings). With a headset, if the headset's ideal render target (the runtime's recommended size, including the supersampling set in SteamVR or the Oculus app) is larger than `MaxVRMegapixelsPerEye` on the game mode (default 3.6 MP per eye), it lowers `xr.SecondaryScreenPercentage.HMDRenderTarget` to fit, never raises a value that is already lower, and restores it when play ends. The Output Log says `SoftPokeDemoGameMode: VR ideal render target ...`. The console variable `softpoke.Demo.VRMaxMegapixelsPerEye` overrides the property (`0` = no cap). Warning for your own project: high supersampling on a wide-FOV headset can ask for tens of megapixels per eye (one test asked for 41 MP per eye) and stall the GPU until the driver resets it. In UE 5.8 the working control is `xr.SecondaryScreenPercentage.HMDRenderTarget` (a percentage); `vr.PixelDensity` is not read any more. On GPUs with 8 GB of video memory or less the demo also applies a [low-VRAM profile](#low-vram--quest-3-pc-vr).

Not tested on a headset: the stick key names on each runtime, the start position and height with your play-space, the walking speed, snap turn feel.

## Showing a hand

SoftPoke ships a sample hand (`SKM_NatHand2_R` / `_L`), and `SoftPokeHandMeshDriver` poses it from the contact-resolved hand; the actor `SAMPLE` in `L_DemoPoke` shows the set-up. To show another hand set `VisibleHandMeshOverrideRight` / `VisibleHandMeshOverrideLeft` on the driver, or tick `bAutoDetectVrTemplateHands` in a project made from the **VR Template** (see the [component reference](ComponentReference.md#softpokehandmeshdriver)). The deformation itself needs no visible hand. To drive a hand of your own without the driver, read the limited pose that the rig exposes:

* `GetProxyKeypoints(Side, OutKeypoints26)` returns the limited hand as 26 world-space transforms in the order of the engine's `EHandKeypoint`.
* `GetProxyJoints(Side, ...)` returns the 21 joints with rotations; `GetTargetJoints(Side, ...)` the tracked, unlimited joints.

To get the VR Template hands into an existing project, add the Virtual Reality / XR template pack (**Content Browser > Add > Add Feature or Content Pack**; the exact menu path depends on the engine version) and keep its folder as it is: the meshes reference other assets of the pack, so a single mesh copied to another path does not load. The same hands work in webcam-only projects.

## Performance

Cost grows with the number of regions that are awake near your hands. VR has been measured on one headset so far (Pimax 8KX at 75 Hz, RTX 4090, the demo map): normal poking holds 75 fps up to 3.6 MP per eye, the demo's default render budget. Sustained deep contact with both hands costs more GPU time: both hands pressing the upper arms of the sample wake 7 regions, and the solver then takes about 4.8 ms of GPU time per frame on an RTX 4090 (desktop measurement; the awake regions the hands do not touch take a short path that costs almost nothing, `softpoke.IdleSkip`). At 75 Hz such a two-hand press used to drop below the frame rate while it lasted (in an early headset test the view faded for a moment while kneading close to the body with both hands, the solver then took 4 to 7 ms and up to 18 ms in single frames). The component now keeps the solves within a GPU budget while the engine renders in stereo (`SolveGpuBudgetMs`, automatic 3 ms): the regions that do not fit in a frame show their last result and are solved in the next one, so they update at half the frame rate while both hands knead several regions. Region data also stays resident now (`VramBudgetMB` 640 MB holds the whole sample body; with 400 MB the regions evicted and reloaded each other, and each upload added 1.5 to 7 ms of GPU time in its frames). Other headsets, refresh rates and GPUs have not been measured. A single dropped frame when a hand comes close to your eyes (for example while combing the hair) can come from the visible hand switching to its see-through material: in the `AutoFade` mode of `SoftPokeHandMeshDriver` the hand turns see-through near the camera, and on the headset the first switch after a while cost the render thread 10 to 20 ms in one frame. If that matters, set `VisibilityMode` to `Opaque` (never see-through) or `Ghost` (always see-through); key H cycles the modes in the demo maps. See [Behaviour and limits](BehaviourAndLimits.md#performance) for what can be turned off (stylize, jiggle, dent overlay) and how to read the timings.

## Low VRAM / Quest 3 PC VR

SoftPoke targets **Windows PC VR**: the game renders on the PC (DirectX 12) and the headset shows it. A Meta Quest 3 is expected to work this way through Quest Link (cable), Air Link, Virtual Desktop or Steam Link; this has **not been tested** (see "Not checked" below). Use the controllers: bare-hand tracking is untested too. The headset's runtime must be the active OpenXR runtime of the PC (the Meta Quest Link app, Virtual Desktop and SteamVR each have a setting for it). **Standalone Quest (an Android build running on the headset) is not a target** of the plugin.

Many PCs used with a Quest 3 have a GPU with 8 GB of video memory. With Epic scalability and the demo's default render budget (3.6 MP per eye) a packaged build of the demo maps needs about 5.6 GB of video memory at its peak (measured, see below), and the VR runtime's compositor needs its own memory on top (about 2.6 GB with SteamVR on a Pimax 8KX). That does not fit in 8 GB next to the streaming app and the desktop. The render features that use the most memory in VR are the transient render buffers of the hair strands (voxel texture, visibility buffer, one-pass shadow projection buffers), the TSR history and the virtual shadow map page pool.

**What the demo does.** When `SoftPokeDemoGameMode` runs with a headset, it reads the dedicated video memory of the GPU when play starts and picks a profile: below 7 GB the **6 GB profile**, below 11 GB the **8 GB profile**, otherwise none (also none when the memory size is unknown). It applies the profile when play starts, only ever lowers a setting (a value that is already cheaper is kept), never replaces a value you set from the console (or in `ConsoleVariables.ini` or on the command line), and gives the previous values back when play ends (with their previous priority, so scalability changes made later still work), so an editor session is not left changed. The Output Log says `SoftPokeDemoGameMode: VR low-VRAM profile ...` with the dedicated video memory in MB, the profile, whether it was chosen automatically or forced, and each setting it changed. Desktop play is never changed. Like the render budget, this is done by the demo game mode only, never by the plugin in your own game mode.

The console variable `softpoke.Demo.VRLowVram` overrides the choice; it is read when play starts:

| Value | Profile |
|---|---|
| `-1` (default) | Automatic from the dedicated video memory |
| `0` | Off |
| `1` | 8 GB profile |
| `2` | 6 GB profile |

**8 GB profile** (`1`):

| Setting | Value | What it costs you |
|---|---|---|
| `r.TSR.History.ScreenPercentage` | `100` | Epic scalability keeps the TSR history at 200 %; at 100 % fine details are a little less stable. |
| `r.TSR.Resurrection` | `0` | Details hidden for a few frames are rebuilt instead of resurrected from older history. |
| `r.Shadow.Virtual.MaxPhysicalPages` | `2048` | Smaller virtual shadow map page pool (no page pool overflow in the demo maps). |
| `r.Shadow.Virtual.ResolutionLodBiasDirectional` | `0` | Directional light shadows at normal resolution instead of the higher Epic resolution. |
| `r.Shadow.Virtual.ResolutionLodBiasDirectionalMoving` | `0` | The same for a moving directional light. |
| `r.HairStrands.Visibility.MSAA.SamplePerPixel` | `2` | Fewer hair visibility samples per pixel; no visible loss in close portraits of the demo character. |
| `r.Shadow.Virtual.OnePassProjection` | `0` | Virtual shadow maps of local lights are projected light by light instead of in one pass. The image is the same; the one-pass buffers (hair transmittance and shadow mask textures, several hundred MB) are not allocated, and the frame takes a little more GPU time (about 0.2 ms in the demo). |
| `r.HairStrands.Voxelization.Virtual.VoxelPageCountPerDim` | `8` | The hair voxel texture holds 8 x 8 x 8 pages instead of the engine's 14 x 14 x 14 (about 73 MB instead of 399 MB). A very large amount of hair close to the camera may get coarser voxels; no visible change on the demo character. A value that is already 8 or lower is kept. |

**6 GB profile** (`2`): everything of the 8 GB profile, plus:

| Setting | Value | What it costs you |
|---|---|---|
| `r.Shadow.Virtual.MaxPhysicalPages` | `1024` (instead of 2048) | One "Virtual Shadow Map Page Pool overflow" warning can appear in the first frames. |
| `r.HairStrands.Visibility.MSAA.MeanSamplePerPixel` | `0.5` | Less memory for the hair visibility samples. |
| Render budget (`MaxVRMegapixelsPerEye` / `softpoke.Demo.VRMaxMegapixelsPerEye`) | `2.5` MP per eye | Lowers the game mode's `MaxVRMegapixelsPerEye` when it is uncapped (`0`) or higher than 2.5; a lower budget is kept, and so is a budget you set with `softpoke.Demo.VRMaxMegapixelsPerEye`. Lower render resolution. |

**In your own project.** Copy the settings into your project's `Config/DefaultEngine.ini`, for example for the 8 GB profile:

```ini
[SystemSettings]
r.TSR.History.ScreenPercentage=100
r.TSR.Resurrection=0
r.Shadow.Virtual.MaxPhysicalPages=2048
r.Shadow.Virtual.ResolutionLodBiasDirectional=0
r.Shadow.Virtual.ResolutionLodBiasDirectionalMoving=0
r.HairStrands.Visibility.MSAA.SamplePerPixel=2
r.Shadow.Virtual.OnePassProjection=0
r.HairStrands.Voxelization.Virtual.VoxelPageCountPerDim=8
```

For the 6 GB profile use `r.Shadow.Virtual.MaxPhysicalPages=1024` and add `r.HairStrands.Visibility.MSAA.MeanSamplePerPixel=0.5`; the render budget of the demo has no project setting, so set `xr.SecondaryScreenPercentage.HMDRenderTarget` yourself (see [Demo VR pawn](#demo-vr-pawn), render budget). Settings in `[SystemSettings]` apply to every platform and to desktop play too; to limit them to some machines, set them from your own code at runtime (as the demo game mode does) or in a device profile.

**Nanite streaming pool.** The editor allocates the Nanite streaming pool (about 514 MB) when it starts. A packaged game allocates it only when the first Nanite mesh is drawn, and the demo maps contain no Nanite mesh: the packaged demo measured 0.6 MB for it, so the numbers below need no Nanite setting. A project with Nanite meshes gets the pool in a packaged game too; `r.Nanite.Streaming.StreamingPoolSize` in `[SystemSettings]` makes it smaller. That setting was not measured with Nanite meshes, and a pool that is too small makes meshes lose detail or stream in visibly.

**Measured.** Demo map `L_DemoPoke`, Epic scalability, a hands-on tour of the whole body (72 s, both hands, driven over UDP). The numbers are from a packaged Development build (Windows, DirectX 12, OpenXR through SteamVR) on an RTX 4090 (24 GB) with a Pimax 8KX, at the render budget per eye listed in the table. Video memory is the dedicated video memory of the game process (the Windows GPU Process Memory counter), the peak of the tour; repeated runs of the same profile differed by less than 10 MB. GPU time is the median with the hands off / during the tour. The headset was not worn during the runs, so frame times are not meaningful, but video memory and GPU time are. **The VR runtime's compositor needs its own video memory on top of this: measured here about 2.6 GB with SteamVR for a Pimax 8KX (1.8 GB for the compositor plus 0.7 GB for the server); other runtimes differ**, and the desktop needs some more. With that SteamVR figure the total is about 8.2 GB for the profile off, 5.3 GB for the 8 GB profile and 4.9 GB for the 6 GB profile. This was not measured on a GPU with 8 GB.

| Profile | Render budget per eye | Video memory (game process) | GPU time |
|---|---|---|---|
| Off | 3.6 MP | 5.6 GB | 7.3 / 9.9 ms |
| 8 GB (`1`) | 3.6 MP | 2.7 GB | 6.9 / 9.5 ms |
| 6 GB (`2`) | 2.5 MP | 2.3 GB | 5.7 / 8.2 ms |

**Checked on a headset.** The 8 GB and the 6 GB profile were forced on the 24 GB card (`softpoke.Demo.VRLowVram` `1` / `2`) and viewed on the Pimax 8KX with the setup described at the top. Hair, shadows and moving skin were judged acceptable in both, and so was the lower resolution (2.5 MP per eye) of the 6 GB profile; no extra ghosting was seen on the moving skin. This is one person's visual check, not a measurement. **Not checked:** a real 8 GB or 6 GB graphics card, and a Quest 3 through Link, Air Link or Virtual Desktop.

### Optional: DLSS

SoftPoke does not ship or require DLSS. NVIDIA's DLSS plugin for UE 5.8 (from NVIDIA's developer site; DLSS Super Resolution, needs an RTX GPU) was tried once with the demo in VR, in the editor VR Preview and in a packaged build. To enable it: enable the plugin "DLSS" in the project, then set `r.NGX.DLSS.Enable=1`, `r.TemporalAA.Upscaler=1` and `r.ScreenPercentage=66.7` (the plugin picks the quality mode from the screen percentage).

What was seen: it ran in both eyes at Quality (internal 1393x1153 per eye upscaled to 2088x1728). No ghosting on the moving skin or the hair was seen and the image looked clean. The application GPU time that SteamVR reported was about 5.9 ms with DLSS and about 8.8 ms without (single runs of the same scene, not a controlled benchmark). Caveats: one headset, one GPU; the effect on video memory in a packaged game was not measured; while DLSS replaces TSR, the TSR entries of the low-VRAM profile (`r.TSR.History.ScreenPercentage`, `r.TSR.Resurrection`) have no effect. Treat this as something to try, not as a tested guarantee.

## What to check on your hardware

00. Demo maps: start `L_DemoPoke` in VR Preview. The VR pawn must be the one you control (`softpoke.Demo.VR` is -1; the pawn is `BP_SoftPokeDemoVRPawn`). You should stand on the floor about an arm's length from the character, facing her, with your virtual floor level with your real floor. Walk with the left stick, snap turn with the right stick. Check that your hand reaches the cheek and the outside of the thigh without walking; if the start is too close or too far change `VRReachDistanceCm` on `BP_SoftPokeDemoGameMode`. Do the same in `L_DemoHair`.
0. Controllers: read `Status` (it should say `controller: left tracked ... (trig ...)`); pull the trigger and squeeze the grip: the numbers change and the visible hand closes. Release the trigger with the grip held: the index points. Hold the virtual hand next to your real hand: is it where your hand is? If not, set `ProfileTweaks` (or `GlobalTweak`).
1. Grip a limb slowly: the dent should follow the fingers; blue and green capsules coincide while the grip is shallow.
2. Press hard with one finger: the green (limited) capsule stays outside the tissue at its thickness limit while blue goes deeper.
3. Move a hand quickly towards and away from the character, and lose tracking (hand out of view): the deformation must fade out about a quarter of a second later (`TimeoutSeconds` of UDP sources; for OpenXR as soon as the runtime reports the hand lost).
4. Watch `MaxViolationMm` in `LeftStatus` / `RightStatus`: values that stay at 0-2 mean the limiter holds.
5. If you use DLSS: look at moving skin and hair for ghosting (see [Optional: DLSS](#optional-dlss)).
6. Report what you see; see [Changelog, known issues, FAQ](ChangelogFaq.md#support).
