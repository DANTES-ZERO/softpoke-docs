# Hair comb (optional): the hand pushes strand hair

> SoftPoke 1.0.0 | [Index](Index.md) | [Glossary](Glossary.md)

> **Optional and off by default.** Nothing on this page is needed to use SoftPoke. The feature needs the engine's HairStrands plugin (shown as **Groom** in **Edit > Plugins**) in YOUR project; SoftPoke does not depend on it (see [How it stays optional](#how-it-stays-optional)).

With the hand mesh driver ([SoftPokeHandMeshDriver](ComponentReference.md#softpokehandmeshdriver)) the visible hand follows the contact-resolved proxy hand. `USoftPokeHairCollision` gives that same hand to the Groom hair simulation as a collision body, so fingers and palm part and push strand hair, and the hair falls back afterwards.

## Try it first: the demo map

Open `L_DemoHair` (**SoftPoke Content > SoftPokeSample > Maps**), press Play, set up the camera as in [Quick Start](QuickStart.md) (Part A), bring your hand to the **side of the head** and move it **downward**, like a comb. The sample character already has the Groom, the hands and the collision set up.

## Setting it up on your own character: about 1-3 hours the first time

Expect roughly **1-3 hours** the first time: a Groom with simulation enabled, body colliders for the head and shoulders, visible hand meshes and testing. Steps, in order:

1. **Have a Groom (strand hair) on your character**, and the Groom (HairStrands) plugin enabled in your project. Cards-only hair is not combed.
2. **Render settings.** The default project settings work; see [Rendering settings](InstallRequirements.md#rendering-settings) for what changes in forward shading, and [Recommended project settings](#recommended-project-settings) below.
3. **Groom asset, simulation on.** Open the Groom asset, tab *Physics*: *Enable Simulation* on for every group; *Collision Constraint*: *Solve Collision* and *Project Collision* on; *Collision Radius* about 0.1 to 0.12 cm. In the *LOD* tab no LOD may have *Simulation = Disable*. A Groom Binding asset refers to its Groom: after copying a Groom, create a new binding for the copy. To change simulation settings from Python use the editor helper in [Enabling simulation from Python](#enabling-simulation-from-python-or-c-editor), not your own array replacement.
4. **A visible hand on each side.** Add a `UPoseableMeshComponent` per hand with a hand skeletal mesh, and a `USoftPokeHandMeshDriver` for it (right mesh with `Side = Right`, left with `Left`). The easiest hands are the sample's own, `SKM_NatHand2_R` / `SKM_NatHand2_L` in `SoftPokeSample/Character/HandNatural`, which you may use in your own project (see [Hand meshes](#hand-meshes)). The mesh may also be hidden: the hair collision follows its pose.
5. **Hand input.** The built-in webcam tracker (`SoftPokeHandSourceNative`, [Quick Start](QuickStart.md), [Built-in webcam tracker](NativeTracking.md)) or OpenXR hand tracking ([VR](VR.md)). The Python tracker over UDP also works ([Webcam tracker guide](WebcamTracker.md), advanced).
6. **The hair collision component.** Add `SoftPokeHairCollision` to any actor (for example the one that holds the hand drivers). Set:
   * `HandDrivers`: the hand mesh drivers (empty = every driver on the owner),
   * `TargetActors`: the actor(s) that carry the Groom component (empty = the owner and the actors attached to it),
   * `BodyActor` and `BodyColliders` (strongly recommended, see [Body colliders](#body-colliders-head-neck-shoulders)),
   * `bEnabled` = true.
7. **Body colliders** for head, neck and shoulders (see below). This is the longest step.
8. **Play and test.** The simulation runs in Play In Editor and in a packaged / `-game` run, **not** in the editor viewport. Tune with [Settings to tweak](#settings-to-tweak).

At BeginPlay the component builds, for each hand, a runtime Physics Asset (capsules per finger segment and a row of palm capsules, radii from the SoftPoke contact model times `HairRadiusScale`), puts it on an invisible `USkeletalMeshComponent` that takes its pose from the hand mesh, and registers that component with every target Groom (`UGroomComponent::AddCollisionComponent`; the list is not saved, so it is rebuilt every run). `Status` says what was found.

## Hand meshes

* **Sample hands (recommended).** `SKM_NatHand2_R` / `SKM_NatHand2_L` in `SoftPokeSample/Character/HandNatural` are the sample character's hand with a short rounded forearm, on a hand-only skeleton with one opaque skin material. They look natural and need no stub. The demo maps use `MI_VisibleHand_R_*` / `MI_VisibleHand_L_*` (parent `M_SoftPokeVisibleHand`): the character's own skin textures, a knuckle / crease / pore normal map generated from the hand mesh (`T_NatHand2_N_R` / `_L`, made by the publisher, no third-party data) and a dithered fade along the forearm (`FadeStartCm` 3 to `FadeEndCm` 8 behind the wrist, so the cut end of the forearm is never seen). The older plain looks `MI_HairCombHand_Skin` and `MI_HairCombHand_LightGlove` are in the same folder `SoftPokeSample/Character/HandLook`. These are the publisher's own assets and contain no Epic content.
* **VR template hands (optional).** `SKM_MannyXR_left/right` or `SKM_QuinnXR_left/right` of Epic's VR template (`Content/XRMannequins/Meshes`) use the default bone names. SoftPoke does not ship Epic content: migrate them from a VR template project. They are cut at the wrist and show an open face from behind: attach your own forearm or sleeve to the wrist bone, or frame the camera so that the wrist leaves the picture. They keep their own dark material unless you override it with an opaque skin-tone material of your own (roughness about 0.55, "Used with Skeletal Meshes" on).
* **Default bone names** are the UE5 / XR mannequin / MetaHuman hand names (or set `BoneMapOverride` on the driver).
* **A hand mesh from your own character.** (1) The skeleton contains only the hand bones (the wrist bone as root), authored in centimetres: a parent chain with a unit-conversion scale (100 or 0.01 on a root bone, typical for FBX from a DCC in metres) collapses all finger bones onto the wrist. (2) Do not name the mesh object like a bone (a mesh called `Hand_R` made the importer rename the bone `hand_r1`: "wrist bone not found"). (3) Re-weight the thumb: heavily blended character weights tear the thumb into a thin rod when it rotates. (4) The contact radii scale with the wrist-to-middle-knuckle length of the input hand, so a smaller mesh hand does not get thinner collision capsules than the input.
* **Combing that shows:** the fingers must be inside the hair, near the scalp, and spread a little; then each finger parts the strands. A hand outside the hair only pushes the outer layer. Keep the hand more than about 15 cm from the view, or the camera's near plane cuts through it.
* **Mirrored input** is a common cause of a "broken" hand: if `USoftPokeHandMeshDriver::Status` says the input hand looks mirrored, fix the tracker's handedness / mirror setting (`bMirrored` on the native source).

## Recommended project settings

These are recommendations for YOUR project (`Config/DefaultEngine.ini`, section `[SystemSettings]`); SoftPoke does not change your project configuration.

```ini
[SystemSettings]
; Removes horizontal light / dark bands in strand hair (hair voxel page allocation). No extra memory.
r.HairStrands.Voxelization.Virtual.Adaptive=0
```

If the hair shows rectangular bands of lost shadow in rows, this is the cause and the setting above is the cure. Do not set `r.HairStrands.Voxelization 0`: it crashes the editor in the light pass.

For **VR** no extra hair setting is needed: the default deferred settings with TSR worked in a headset test (instanced stereo off). Forward shading with MSAA made the sample's strand hair much darker in our tests, so it is not recommended for the sample. If your VR project uses **Instanced Stereo**, check the hair in the headset: under TSR it was drawn in the wrong place in `-emulatestereo` (see [Rendering settings](InstallRequirements.md#rendering-settings)). These settings are meant for the HairStrands plugin of UE 5.8 and were tested on an RTX 4090 only.

## Body colliders (head, neck, shoulders)

The Groom solver only knows the Physics Assets you give it. Without a body, a hand that presses hair towards the head pushes the strands **into the skull** (they then pop out as stray spikes). `BodyColliders` is a short list of spheres and capsules (bone name, start and end in the component space of the character's rest pose in cm, radius in cm; start = end is a sphere). Put a few spheres inside the skull just under the scalp (the hair roots), a capsule in the neck and one per shoulder. About a dozen are enough. `BodyActor` is the character (empty = the skeletal mesh the Groom is attached to). A bone with a scale (for example 100 on an FBX in metres) is handled. Turn on `bDrawDebug` to see the colliders (orange) and check that they sit inside the head.

## Collider stabiliser: no thrashing of combed hair

Strand hair is simulated against **kinematic** capsules. If the capsules jump between two frames (a tracking glitch, a tracker that finds the hand again somewhere else, a hand that is hidden by putting it far away), the Groom solver sees an enormous collider velocity and throws the strands around. With `bStabilizeCollider` (default **on**) the hair collider is therefore not rigidly attached to the visible hand any more. Each hand carries a separate copy of the hand pose that the stabiliser moves; the visible hand is not touched, only what the hair feels:

1. **Low-pass and speed limit.** The wrist position of the collider follows the hand with `a = 1 - exp(-dt / ColliderSmoothingSeconds)` (0.03 s), and the move of one frame is clamped to `ColliderMaxSpeedCmPerSec * dt` (100 cm/s). The same step is applied to every capsule of the hand (they ride on the wrist). Combing at up to about 60 cm/s is not changed.
2. **Glitch rejection.** A wrist sample more than `GlitchRejectCm` (5 cm) away from the last accepted one is ignored for up to `GlitchRejectMaxFrames` (3) consecutive frames; the next one is believed.
3. **Tracking loss.** The collider holds for `LossHoldSeconds` (0.1 s), then retreats `LossRetreatCm` (30 cm) away from the head at the speed limit. When the hand is found again it stays retreated `ReacquireHoldOffSeconds` (0.2 s) and then approaches the new position at the speed limit. At the start of Play the collider is parked the same way, above the head.
4. **Radii and head.** The contact radii are unchanged (`HairRadiusScale` 1.0). `bAddHeadSphere` adds a sphere (`HeadSphereRadiusCm` 7) inside the skull when `BodyColliders` has no collider on the head bone; the 100-primitive limit is respected.

Measured in the sample (L_DemoHair, synthetic hand sweeping 23 cm down the side of the head with 3 mm tracker noise; hair motion = changed pixels per frame of a head capture, peak, two runs each): a clean sweep and a fast sweep (about 190 cm/s) moved the hair the same with and without the stabiliser (about 2.1k to 2.5k px, within the run-to-run spread of 20 %); the wrist speed seen by the Groom solver fell from 199 to 100 cm/s. A tracking loss of 0.8 s that returns 13 cm farther along (the hand mesh is hidden and parked far away by the driver) threw the hair across the face without the stabiliser (peak 10.9k to 11.2k px, collider speed 3.3 million cm/s) and left it calm with it (1.4k to 1.5k px, 100 cm/s). Short glitches and drop-outs (10 cm for two frames, 0.15 s loss) were capped at 100 cm/s too, with a hair motion change inside the spread. A slower, steadier hand is always better: the speed limit is a safety net, not a reason to wave the hand at 3 m/s. Set `bStabilizeCollider` to false to get the old rigid attachment (for example when your tracker already is perfectly smooth and you want zero lag: the stabiliser adds about 0.03 s).

The Groom asset settings of the sample hair were **not** changed for this (see the Groom physics notes in [Settings to tweak](#settings-to-tweak)).

## Settings to tweak

| Setting | Where | Effect |
|---|---|---|
| `HairRadiusScale` | component | Multiplier of the contact radii (fingers 8.5 to 10.5 mm, thumb 10 to 12, palm 8.5 to 12 at the reference hand of 87.7 mm wrist to middle knuckle, scaled with the hand). 0.8 lets the hair part closer to the visible finger, 1.3 parts it earlier. Fixed when the simulation starts: set it before Play. |
| `PalmStationStride` | component | The palm is 23 capsules in the contact model; every Nth is used (default 3 = 9). The Groom solver accepts 100 primitives in total (all collision sources of a Groom together). |
| `DistalLengthFraction` | component | The hand mesh has no fingertip bone: the last segment is this fraction of the middle one (default 0.8). |
| Guide count | Groom asset *Interpolation* | About 500 generated guides is roughly one guide per square centimetre: a single thin finger moves few guides. More guides cost more simulation time. |
| Gravity, bend stiffness, friction, viscosity | Groom asset *Physics* | With gravity -981 the ends of a bob droop; gravity 0 keeps the shape. Low friction lets strands slide off the fingers; a high strand viscosity keeps neighbouring strands together, so combed hair parts in clumps instead of fraying into thin wisps (sample hair: friction 0.02, viscosity 0.85, bend stiffness 0.027-0.039 with damping 0.02; before: 0.05 / 0.3 / 0.009-0.013, which frayed). |
| `bStabilizeCollider` and the stabiliser values | component | See [Collider stabiliser](#collider-stabiliser-no-thrashing-of-combed-hair): `ColliderSmoothingSeconds` 0.03, `ColliderMaxSpeedCmPerSec` 100, `GlitchRejectCm` 5, `GlitchRejectMaxFrames` 3, `LossHoldSeconds` 0.1, `LossRetreatCm` 30, `ReacquireHoldOffSeconds` 0.2. |
| `bDrawDebug` | component | Draws the hair capsules (cyan: hands, orange: body) and logs a few positions every two seconds. |

`ExportCapsuleJson(Path)` (or `CapsuleJsonPath` for an export at BeginPlay) writes the radii in cm as JSON (`softpoke_hair_capsules_v1`: per capsule the two keypoints, `radius_cm` for the reference hand and `radius_per_palm_length`), so a DCC preview can use the same sizes.

## Enabling simulation from Python or C++ (editor)

Replacing the whole `HairGroupsPhysics` array from Python does not rebuild the Groom's derived data, and saving then crashes. The editor module provides `USoftPokeHairEditorLibrary`:

```python
import unreal
g = unreal.load_asset('/Game/MyChar/Groom/g_hair_copy')
err = unreal.SoftPokeHairEditorLibrary.set_groom_simulation(g, True, True, True, -1.0, -1, -1)  # enable, solve, project, radius, substeps, iterations (negative = keep)
print(err or 'ok', unreal.SoftPokeHairEditorLibrary.describe_groom_simulation(g))
unreal.EditorAssetLibrary.save_asset('/Game/MyChar/Groom/g_hair_copy', only_if_is_dirty=False)
```

It sets the properties by reflection and raises the property-changed event for `EnableSimulation`, which makes the Groom asset rebuild its derived data the way the Groom editor does.

## How it stays optional

* `SoftPoke.uplugin` has no dependency on HairStrands, and no SoftPoke module links against a HairStrands module. (A plugin dependency marked `Optional` would still switch the plugin on in every project, so it is not used.)
* The Groom is reached by reflection: `/Script/HairStrandsCore.GroomComponent` is looked up by name and `AddCollisionComponent` is called through `ProcessEvent`. Without the HairStrands plugin the class is not found and the component only sets `Status` to "no Groom class: enable the HairStrands plugin".
* The Groom asset helper in the editor module is reflection only too.

## Hair LOD for VR (optional, off by default)

Strand hair is the expensive part of hair combing in VR; the simulation of a few hundred guides is not. A **cards** (or reduced-strand) LOD of the same Groom costs a fraction of the strands. The Groom's own LOD system can switch to cards by distance, but a hand that combs the hair needs the **simulated strands**. `USoftPokeHairCollision` can therefore manage the LOD for you:

* by default the Groom keeps its own LOD selection, or, with `HairLodDefaultIndex >= 0` (for example 1 = cards), that LOD is forced at every distance, so the strands render only while a hand is near,
* while any hand is within `HairLodNearCm` (default 25 cm) of the head, LOD 0 (`HairLodForcedIndex`) is forced on every target Groom,
* the forced LOD is released only when every hand has been farther than `HairLodFarCm` (default 35 cm, hysteresis) for `HairLodReleaseDelay` seconds (default 0.75 s).

The sample hair is **strands only** and has no cards, so `bManageHairLOD` is **off** in the demo map. With a strand-only Groom the manager is harmless.

Enable it with `bManageHairLOD = true` on the `SoftPokeHairCollision` component (combing itself still needs `bEnabled`). The state is readable in `HairLodStatus`, `bHairLodForced` and `HairLodHandDistanceCm`, and `SetHairForcedLOD(-1 / 0 / 1 ...)` forces a LOD by hand. The head position is the bone `HairLodHeadBone` (default `head`) of `BodyActor`'s skeletal mesh; if that bone does not exist the centre of the Groom's bounds is used. The LOD is forced through the engine's `ILODSyncInterface`, which `UGroomComponent` implements; there is no HairStrands dependency.

### Preparing the Groom asset

1. **A Groom with LODs**: LOD 0 = strands, LOD 1 / 2 = cards (or a strand LOD with a lower decimation). In the Groom asset's *LOD* tab set *LOD Mode* to **Manual** and the screen sizes of LOD 1 / 2. Cards need a cards mesh per hair group and a hair-cards material; the engine can generate cards from the Groom (*Cards* tab).
2. **Let the manager choose the LOD** (set `HairLodDefaultIndex`, for example 1 = cards); do not rely on the engine's screen-size switch, which uses the bounds of the whole body when the Groom is attached to the skeletal mesh. Keep the Groom attached to the skeletal mesh component as usual.
3. **VR thresholds.** For a VR eye with a 90 degree field of view the engine switches at `screen size = 2 * R / d` (R = radius of the Groom's bounds, d = distance to the head). A screen size just under 1.0 gives cards at any distance except when a hand forces strands.
4. Keep the rules for combing: simulation on, collision on, the guides unchanged.

Known limits: (1) the strands-to-cards switch pops (UE 5.8 has no dithered transition for Grooms); the cards are a little paler and flatter than the strands. (2) While a hand is near, LOD 0 is forced and the strands cost more than a strands-only Groom that reduces strands with distance: the saving is for the time the hand is away from the head. (3) With Instanced Stereo on, strand hair under TSR was drawn in the wrong place in `-emulatestereo` (not checked on a headset). (4) The hair LOD manager was tested with `-emulatestereo` only, not with a headset.

## Limits

* The Niagara Physics Asset interface reads `USkeletalMeshComponent` only (a poseable mesh cannot be registered), and it fixes capsule radii when the simulation starts; this is why the component adds an invisible skeletal mesh follower and why `HairRadiusScale` needs a restart of Play.
* 100 primitives in total per Groom.
* The palm capsules are rigid with the wrist bone (the contact model moves the knuckles).
* Hair that lies flat against the scalp (thin fringes on the forehead) has nothing to push: only strands that can move away from the head react.
* Cost: the hair simulation (about 500 guides x 16 points) was 0.65 to 0.7 ms on an RTX 4090; strand rendering is the larger part. Use the engine's GPU profiler (`ProfileGPU`) in your project.
