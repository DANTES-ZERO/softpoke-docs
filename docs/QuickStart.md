# Quick Start

> SoftPoke 1.0.0 | [Index](Index.md) | [Glossary](Glossary.md)

This page has four separate parts. Do only the part you need; each one is complete on its own. Steps are numbered inside each part: "A9" means step 9 of Part A, "B3" step 3 of Part B.

* **Part A: the sample character (about 10 minutes of hands-on time).** Try SoftPoke with the character and the two demo maps in the package. Nothing to bake.
* **Part B: your own MetaHuman.** Bake a MetaHuman of your project.
* **Part C: any other character of your own.** The short route, then [Bake Your Own Character](BakeYourOwnCharacter.md).
* **Part D: VR with controllers (experimental).**

**Can I use my own character?** Only after a bake. SoftPoke cannot deform a character that has not been baked, and the baked data of the sample character fits the sample mesh only. Before you start: the skin must be a separate, watertight mesh without clothes, hair, eyes or teeth (a game character with its clothes built into one mesh needs its skin separated in a 3D tool such as Blender first); the bake needs 64-bit Python 3.12 and a one-time download of about 1.5 GB; one limb in *Draft* quality takes about 10 minutes and a whole body up to several hours; the MetaHuman helper of Part B has not been run by the publisher on a default MetaHuman. If this is too much work, use Part A with the sample character.

Unfamiliar words (SPB, region, quality gate, Groom, NNE, PIE, ...) are explained in the [Glossary](Glossary.md). **Adult humanoid characters only.**

---

## Part A: the sample character

The package includes one **sample character** (an adult woman, already baked) and two demo maps. You need Unreal Engine 5.8 on Windows, a GPU that runs DirectX 12 with Shader Model 6 ([requirements](InstallRequirements.md)), and a webcam (or a video file of your hand).

**How long it takes.** About **10 minutes of hands-on time, plus 5-20 minutes of one-time shader compiling** (and 5-15 minutes more if the plugin itself is compiled on your PC) that you only wait for:

| Step | Time | Note |
|---|---|---|
| Compile the plugin on first enable | 5-15 minutes, once | Only if the plugin is compiled on your PC: when you copied the plugin source into your project's `Plugins` folder (this also needs a C++ toolchain, see [Install](InstallRequirements.md#install)), or when the editor asks to compile a plugin that you installed from Fab / the Epic Games Launcher. If the launcher delivers a built plugin there is no such wait. (Packaging a game needs a C++ toolchain in any case: see [Build tools for packaging](InstallRequirements.md#build-tools-for-packaging).) |
| Compile shaders on the first Play | 5-20 minutes, once | The editor can look frozen. Look at the bottom-right corner of the editor: it says **Compiling Shaders** and shows the progress. Wait until it is gone. |
| Camera set-up | 2-5 minutes | Steps A9 and A10. |

### Steps

1. **Install the plugin** as described in [Install and Requirements](InstallRequirements.md#install) and open your project.
2. **Enable it.** **Edit > Plugins**, search "SoftPoke", tick **Enabled**. It also enables the engine plugins *Deformer Graph* and *Compute Framework*. For the hair map check that the **Groom** plugin (the engine's HairStrands) is enabled. For the webcam check that **NNERuntimeORT** is enabled (it comes with SoftPoke as an optional dependency; Epic marks it experimental / beta in UE 5.8). Restart the editor when asked.
3. **Use DirectX 12.** **Project Settings > Platforms > Windows**: **Default RHI** = **DirectX 12**. A new UE 5.8 project usually has this already: then change nothing. If you change it, restart the editor.
4. **Check that the plugin loaded.** Open **Window > Output Log** (Developer Tools), type `SoftPoke.Cmd list_regions` into the **Cmd** box at the bottom of the Output Log and press Enter. A usage warning from SoftPoke proves that the commands are registered.
5. **Show plugin content.** In the Content Browser open the **Settings** menu (gear button) and tick **Show Plugin Content**.
6. **Open the demo map.** **SoftPoke Content > SoftPokeSample > Maps > `L_DemoPoke`**. Both demo maps show a visible natural hand that follows your tracked hand. They work in a default project (deferred shading, TSR, Lumen): see [Rendering settings](InstallRequirements.md#rendering-settings). You do not have to change any render setting: each map brings its own exposure, lights and floor, so it looks the same in any project.
7. **Press Play.** The first Play compiles shaders (see the table above). Click once inside the game window. The map brings its own desktop view: **W A S D** move, **Q / E** down / up, the arrow keys or the right mouse button turn, **Shift** is fast. These keys (and **C** below) only reach the game while the game view has keyboard focus: if nothing happens, click inside the game view first.
8. **See `Status` and set your hand size.** While playing, free the mouse: press **F8** (eject) or **Shift+F1**. In the **Outliner** select the actor **SAMPLE**. In **Details** open the component **SoftPokeHandRig** and expand **Source**: **Status** shows the camera name, format, frame rate and any error; **Hand Size Cm** is your hand size. `0` means an average adult hand and is fine for a first test. For better depth, measure from your wrist crease to the base of your middle finger and enter that. **Changes made during Play are lost when Play stops.** To keep a value, stop Play, select **SAMPLE** in the level, set it there and save the level (the demo maps are plugin content: if the editor will not save the map in place, use **File > Save Current Level As** to save a copy in your project's `Content` folder and work in that copy). Then click into the game window again to go on.
9. **Set up the camera.**
   * Windows **Settings > Privacy & security > Camera**: **Let desktop apps access your camera** must be **On**.
   * Close other programs that use the camera (video calls, OBS, the Python tracker).
   * **Virtual cameras** (EpocCam, OBS Virtual Camera, NVIDIA Broadcast, ...) are often device 0 and are picked first. The device that was opened is shown in `Status` (step 8). To use another camera, type part of its name into **Camera Name**, or its number into **Camera Index** on the same **Source** (category *SoftPoke > Native > Camera*; index 0 is the first device; a non-empty Camera Name wins over the index; the Blueprint function `ListCameras()` lists the devices).  Set them on the **SAMPLE** actor in the level (not the Play copy) to keep them.
   * Put your hand **40-80 cm** from the camera, in even light, in front of a plain background, palm or back of the hand towards the camera.
   * Use **one hand in front of the webcam and the other on the keyboard and mouse** for walking.
10. **Recenter.** Hold your hand where it should rest in front of the camera and press **C**. Press **C** again whenever the hand drifts away.
11. **Touch the character.**
    * In **`L_DemoPoke`**: the view starts about **50 cm** in front of the face (the horizon is in the middle of the frame and the face is in its lower half). Your hand rests about **40 cm in front of and 25 cm below the view** (after **C**); a reach forward (10-15 cm) and up touches the face. If your hand does not reach, click into the game view and walk closer with **W** (back with **S**; **Q / E** down / up); for a forearm or a thigh walk there with **W A S D**. Move your hand in towards the skin: the skin dents and the flesh rises around the fingers. The face has a **static expression** (it does not follow blend shapes), and eyes, nose, lips and ears do not dent (a fingertip stops on them).
    * When your hand comes straight at the face, the visible hand turns see-through (a single layer, no fingers showing through the palm) so you can see the contact, and the end of its wrist stub fades away. **H** cycles the visible hands through *opaque*, *auto-fade* (default) and *ghost* (always see-through).
    * In **`L_DemoHair`**: spread your fingers, bring the fingertips **into the hair close to the scalp** (on top or at the side of the head) and draw them **backward or downward**, as if combing. The strands part around the fingers and fall back when the hand leaves. A hand that only hovers outside the hair pushes the outer strands but does not part them. More in [Hair comb](HairComb.md).

No webcam? Stop Play, set **Video File** on the same **Source** to an `.mp4` of your hand, and play again: it is played in a loop instead of the camera. Press **C** once the hand is visible. With the sample clip's hands the recorded hands reach the face region from the start position; if your own clip's hand stays short of the face, walk closer with **W** (about 60 cm per second, **Shift** is faster): a clip cannot reach forward on its own, so you bring the character to the hand. A relative **Video File** path is looked up in the **project folder** first (for example `Content/Movies/hand.mp4` or `Saved/hand.mp4`), then in the project's `Content` folder (so `Movies/hand.mp4` works too), then in the working directory of the editor (the engine's `Binaries\Win64`); an absolute path always works. If the file is not found, `Status` lists the full paths that were tried.

If the hand does not follow or the skin does not deform, see [If nothing happens](#if-nothing-happens). What is in the sample, where it comes from and the AI-content disclosure: `Licenses/SAMPLE_PROVENANCE.md`. Character created with Charakuru (by Nano).

---

## Part B: your own MetaHuman

**Supported:** MetaHumans made with the MetaHuman plugin / MetaHuman Creator for UE 5.8, in your own project. SoftPoke contains no MetaHuman meshes, textures or face poses. The bake tool recognises the MetaHuman skeleton names.

**The MetaHuman licence is yours to check.** SoftPoke bakes data from your MetaHuman's body mesh and that data stays in your project; whether you may use or ship a MetaHuman is governed by Epic's MetaHuman terms, which SoftPoke cannot interpret for you. Please read them.

**What to expect.**

* The one-call helper in step B4 has not yet been run by the publisher on a default MetaHuman. If it reports a problem, make the same settings by hand ([Bake Your Own Character](BakeYourOwnCharacter.md)).
* The face is baked with a **static expression**: it dents, but it does not follow blend shapes or face joints.
* **Time.** The one-time **Set up** of the bake tool takes a few minutes (about 1.5 GB download). One limb in *Draft* quality takes about 10 minutes. A whole body in *Standard* quality takes up to **several hours on the CPU** (the default; the editor stays usable). The GPU path is faster, but a long GPU job while the editor is open can reset the display driver: close other GPU programs and read the warning on the panel's **GPU** option. Do the one-limb test first.

### Steps

1. **Enable the plugin and DirectX 12** as in Part A, steps A2 and A3, and check the Output Log command (A4).
2. **Put your MetaHuman Blueprint into the level** and select it. Note its label in the Outliner.
3. **Install Python 3.12** (64-bit, from python.org; the Microsoft Store version does not work). It is needed for the bake only, **not** for the webcam.
4. **Prepare the actor.** In the **Cmd** box of the Output Log type `SoftPoke.Cmd prepare_metahuman <actor label>`. It finds the `Body` mesh, adds a `SoftPokeComponent` and a `SoftPokeHandRig` to the actor and fills in the Bake panel. Then select the **SoftPokeHandRig** and set **Source Class** to `SoftPokeHandSourceNative` if it is not set already (the built-in webcam tracker). Camera set-up and **Status**: as in A8 to A10.
5. **Open the Bake panel:** **Window > SoftPoke Bake**. Press **Set up** once (private Python environment; internet needed).
6. **Test with one limb:** type `forearm_L` into **Regions**, set **Quality** to *Draft*, press **Bake** (about 10 minutes). The panel creates the deformer graph (`DG_SoftPoke`) by itself if it is missing.
7. **Read the quality gate result** in the panel.
   * **PASS**: the body was baked, imported and assigned to your actor.
   * **WARN**: the tool found something it is not sure about and **baked nothing yet**. Read each message. Either fix the cause in your character and bake again, or, if the message is harmless for you, tick **Accept gate warnings** and press **Bake** again.
   * **FAIL**: an error that cannot be overridden. Fix the cause ([the quality gate](BakeYourOwnCharacter.md#the-quality-gate)).
8. **Save** the level and the mesh asset, press **Play**, move your hand towards the left forearm.
9. **Whole body:** set **Regions** to `all` and **Quality** to *Standard*, and start the bake when you can leave the PC for hours.
10. **Visible hand (optional):** the actor `SAMPLE` in the sample map `L_DemoPoke` shows the set-up of a `SoftPokeHandMeshDriver` with the sample hands `SKM_NatHand2_R` / `_L`; see also the [component reference](ComponentReference.md#softpokehandmeshdriver). Visible hand mesh: the bundled hand is used by default. To use another hand, set `VisibleHandMeshOverrideRight` / `VisibleHandMeshOverrideLeft` on the driver (any skeletal mesh with the UE mannequin / MetaHuman hand bone names; the driver swaps in its own poseable mesh at Begin Play, draws it with `SkinMaterialForOverride` unless `bUseSourceMeshMaterials` is on, and falls back to the bundled hand with a `LogSoftPoke` warning if the mesh has no skeleton or no wrist / finger bones). If your project was made from the VR Template, tick `bAutoDetectVrTemplateHands` to use its MannyXR hands (`/Game/Characters/MannequinsXR/Meshes/SKM_MannyXR_right` / `_left`); SoftPoke never ships Epic's meshes. The MannyXR hands read as a white glove with panel seams, which is why this is opt-in. For hair combing see [Hair comb](HairComb.md).

Troubleshooting only: if the deformer graph is ever missing, create it by hand with `SoftPoke.Cmd build_deformer /Game/SoftPoke/DG_SoftPoke 1`.

---

## Part C: any other character of your own

1. **Check your character** against the [input checklist](BakeYourOwnCharacter.md#input-checklist). The key points: the **skin must be a separate, watertight (closed) mesh** without clothes, hair, eyes or teeth; clean skin weights; a humanoid skeleton with recognisable bone names; adult proportions. A typical game character with its clothes built into one mesh does **not** pass until the skin is separated, which needs a 3D tool such as Blender.
2. **Enable the plugin and DirectX 12** (A2, A3). Install Python 3.12 (B3).
3. **Bake:** **Window > SoftPoke Bake**, then follow [Bake Your Own Character](BakeYourOwnCharacter.md): **Set up** once, test with one limb in *Draft*, read the gate, then bake the whole body. The times are those of Part B.
4. **Add a hand:** add a **SoftPokeHandRig** to the character's actor, set **Source Class** to `SoftPokeHandSourceNative`, and use A8 to A10.

---

## Part D: VR with controllers (experimental, checked on one headset setup)

For Valve Index controllers with SteamVR, Quest Touch, Vive wands, WMR controllers and any other OpenXR controller. Details and the list of what is not verified: [VR](VR.md).

1. **Engine plugins.** Enable **OpenXR** and **OpenXR Hand Tracking** (Edit > Plugins), and **Enhanced Input** (on by default). Restart the editor. Make your headset's runtime the active OpenXR runtime (SteamVR: Settings > Developer > Set SteamVR as OpenXR runtime). Plugin content: tick **Show Plugin Content** (A5).
2. **Play the demo maps with a headset.** Nothing to build: the demo maps' game mode (`BP_SoftPokeDemoGameMode`) spawns the ready-made VR pawn `BP_SoftPokeDemoVRPawn` (SoftPoke Content > SoftPokeSample > Demo) when a headset is enabled, and the desktop pawn otherwise. Play in **VR Preview** (or launch the map as a VR game). You start standing at floor level, an arm's length from the character, looking at her. **Left thumbstick / trackpad**: walk (relative to where you look); **right thumbstick / trackpad left-right**: snap turn (30 degrees) about your head. No headset but you want to see the VR pawn: console `softpoke.Demo.VR 1` before Play (`0` = always the desktop pawn). In your own map use the same pair (`SoftPokeDemoGameMode` + `SoftPokeDemoVRPawn`), or any pawn with a camera (the VR Template's pawn): the controllers' world poses follow the pawn's tracking origin. Details: [VR](VR.md#demo-vr-pawn).
3. **The hand rig.** Add a `SoftPokeHandRig` (the sample's actor `SAMPLE` has one) and leave **Source Class** at `SoftPokeHandSourceAuto`. It uses OpenXR hand tracking if the runtime reports hands (Valve Index controllers through SteamVR: the finger-curl skeleton), else the controllers, else the source you set in **Auto Fallback Class** (the sample sets `SoftPokeHandSourceNative`, the webcam, which starts only when no VR hand is tracked for a second).
4. **Play in VR Preview.** Read the rig's **Status**: `Auto: L controller, R controller | ...`. Pull the trigger and squeeze the grip: the fingers of the visible hand follow. Release the trigger with the grip held: the index finger points. Touch the thumbstick / trackpad / face buttons: the thumb curls.
5. **Touch the character** as in A11: poke with the pointing index finger, grab and squeeze a limb with the grip, comb hair with an open hand. The controller vibrates lightly when the hand touches the skin (`bHapticsOnContact` on the controller source).
6. **If the virtual hand is not where your hand is**, open the `Controller` sub-object of the Auto source (or use `SoftPokeHandSourceController` as the Source Class) and set `PalmOffsetCm` / `RotationOffset` in `GlobalTweak` or the entry of your controller in `ProfileTweaks`. If the fingers do not react, check that the Status says `[input: Enhanced Input]` or `[input: raw keys]` and that the trig / grip numbers change; otherwise your controller profile is not one of the mapped ones ([VR](VR.md#motion-controllers)).

---

## If nothing happens

| Symptom | Check |
|---|---|
| The skin does not deform | In Play, select the actor and read `SoftPokeComponent > Status` and `RegionSummary`. "Requires DirectX 12 / Shader Model 6" means: fix the RHI (A3). Is the Deformer Graph assigned to the mesh and is `BodyAsset` set? Re-run `SoftPoke.Cmd assign_body <actor label> <SPB asset> /Game/SoftPoke/DG_SoftPoke`, then save the level and the mesh asset. |
| The mesh is invisible or exploded | Open `DG_SoftPoke` and compile it; read `LogSoftPoke` and `ComputeFramework` errors; clear the mesh deformer on the component to confirm the mesh itself is fine. |
| No hand with a headset | Read the rig's `Status`: `Auto: L none, R none` means neither OpenXR hands nor controllers are tracked (runtime not running, controllers asleep, or the wrong OpenXR runtime). See [VR](VR.md). |
| No hand, no movement | Read the native source's `Status` (A8): camera name and format, fps, errors such as "NNERuntimeORTCpu is not available" (enable the NNERuntimeORT plugin). See [Built-in webcam tracker](NativeTracking.md#troubleshooting). |
| The wrong camera is used | A virtual camera was picked first: set **Camera Name** or **Camera Index** (A9). |
| The editor looks frozen on the first Play | It is compiling shaders: look at the bottom-right corner of the editor. |
| Works in the editor, not in a packaged build | Packaged builds must use DirectX 12 (SM6). The mesh asset needs the deformer registered in its `TargetMeshDeformers` (`assign_body` does this), and a Shipping build ignores `-map`: set the default map. |
| Screen flicker or editor freeze during a bake | A long GPU job reset the display driver. Bake on the CPU while the editor is open (the panel's default). |

More: [Install and Requirements](InstallRequirements.md), [Bake Your Own Character](BakeYourOwnCharacter.md), [Known issues](ChangelogFaq.md#known-issues).

## Ask your AI agent (optional)

SoftPoke ships five ready-made skills for AI coding agents (Claude Code and other agents that read the open Agent Skills format): set-up and "nothing deforms" diagnosis, importing your own character, reading quality-gate messages, tuning the look and feel, and packaging and support reports. Copy or link the folders from `Plugins/SoftPoke/Extras/AgentSkills/` into your project's `.claude/skills/` folder (other agents: point them at `AGENTS.md` in that folder), restart the agent session, and describe your problem in plain words. The agent works through scripts, console commands and log files, asks before long jobs, and cannot click editor buttons. Instructions and limits: `Extras/AgentSkills/README.md`.
