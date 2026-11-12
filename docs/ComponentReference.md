# SoftPoke component reference

> SoftPoke 1.0.0. Property names, types, defaults and ranges on this page were generated from the plugin's header files (`Source/SoftPokeRuntime/Public`, `Source/SoftPokeEditor/Public`) and checked against them. Defaults are the values in the C++ declaration; a Blueprint or level can override them.

Contents: [SoftPokeComponent](#softpokecomponent) | [SoftPokeHandRig](#softpokehandrig) | [Limiter settings](#limiter-settings-fsoftpokelimitersettings) | [Hand sources](#hand-sources) | [SoftPokeFaceDriver](#softpokefacedriver) | [SoftPokeTouchSensor](#softpoketouchsensor) | [Project settings](#project-settings) | [Data assets and small types](#data-assets-and-small-types) | [Editor Python / Blueprint library](#editor-python-and-blueprint-library-usoftpokebakelibrary) | [Console](#console-commands-and-variables) | [Bake tool settings](#bake-tool-settings-usoftpokebakesettings)

In editor Python, a UFUNCTION `FooBar` is `unreal.<Class>.foo_bar`, and a UPROPERTY `bFoo` / `FooBar` is `get_editor_property('foo')` / `('foo_bar')` (the `b` prefix of booleans is dropped).

Read-only properties show the state of a running component (Play-In-Editor or a packaged game) in the Details panel. Properties marked "Development" exist for diagnostics; leave them at their defaults.

## SoftPokeComponent

`USoftPokeComponent` (class group Rendering, spawnable in Blueprints). Add **one per skinned body** to the actor that carries the skeletal mesh. It receives hand capsules every frame, solves all baked regions near the hand on the GPU and adds the result to the one displacement buffer that the Deformer Graph reads. Without a valid body asset it does nothing (or runs a simple CPU placeholder if `bUseGpuSolver` is off).

Typical setup: `BodyAsset` = the imported body, a Deformer Graph with the SoftPoke data interface assigned to the skeletal mesh component (`SoftPoke.Cmd assign_body` does both). Everything else has working defaults.

**Category `SoftPoke`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `BodyAsset` | `USoftPokeBodyAsset` |  |  | Baked body (all regions) made by the import step (`SPB_<name>`). Takes precedence over `BakeAsset`. |
| `BakeAsset` | `USoftPokeBakeAsset` |  |  | A single baked region. Used only when `BodyAsset` is empty. |
| `TargetMesh` | `USkinnedMeshComponent` |  |  | Skinned mesh to deform. Empty = the first skinned mesh on the owning actor. |
| `TargetMeshName` | `FName` |  |  | Name of the skinned mesh component to deform. Wins over `TargetMesh` and survives Play-In-Editor duplication; filled by `assign_body` / `assign_face`. |
| `SolverRateHz` | `float` | `0.f` | >= 0.0 | Solve rate cap in Hz. 0 = one solve per `SetCapsules` call. |

**Category `SoftPoke|Multi`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bSecondaryComponent` | `bool` | `false` |  | Marks the second component of a split body + face character. The hand rig drives the primary component and feeds every secondary one with the same capsules. |
| `bTickAfterTargetMesh` | `bool` | `false` |  | Tick after `TargetMesh` so that the bend following reads the pose that mesh has just evaluated (face meshes with their own animation blueprint). |

**Category `SoftPoke|Face`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `ExclusionRegion` | `FName` | `"face"` |  | Face component: region whose contacts are removed under the chin while the mouth is open (see `SetUnderChinExcluded`). |
| `ExclusionMaxZCm` | `float` | `161` |  | Face component: height (cm, rest pose, component space) below which the skin may count as the fold under the chin. |
| `ExclusionNormalZ` | `float` | `-0.9` |  | Face component: rest-normal Z that the nearest skin point must be below to count as the fold under the chin (-0.9 = only the true fold). |
| `ExclusionFadeSec` | `float` | `0.15` | >= 0.0 | Seconds in which an excluded capsule shrinks to zero radius (0 = switch at once). |
| `NumExcludedCapsules` | `int32` | `0` |  | Capsules parked by the under-chin exclusion in the last solve (read only). |

**Category `SoftPoke|Stylize`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `StylizeStrength` | `float` | `1` | 0.0 .. 2.0 | Emphasis knob `s` of the dent-and-bulge look. 0 = off (pure physical result, no extra GPU pass), 1 = recommended, 2 = maximum. |
| `StylizeSpillScale` | `float` | `1` | 0.0 .. 2.0 | Multiplier of `s` for the raised rim around the hand. |
| `StylizeRidgeScale` | `float` | `1` | 0.0 .. 2.0 | Multiplier of `s` for the ridges between the fingers. |
| `StylizeRound` | `float` | `0.9` | 0.0 .. 1.0 | Roundness of the rim profile (0 = plain hill, 1 = half-round). Earlier default: 0.6. |
| `StylizeVolume` | `float` | `1` | 0.0 .. 1.0 | Volume compensation: share of the added volume taken back from the broad far bulge. |
| `StylizeRimHeightMm` | `float` | `4` | 0.0 .. 15.0 | Height of the rim hill at `s` = 1 (mm). Earlier default: 5 (a whole grabbing hand made a hard ring). |
| `StylizeRidgeHeightMm` | `float` | `10` | 0.0 .. 20.0 | Height of the finger ridges above the rest surface at `s` = 1 (mm). |
| `StylizeSmoothIterations` | `int32` | `0` | 0 .. 16 | Smoothing passes of the lift (0 = automatic from the mesh density). |
| `StylizeSmoothWeight` | `float` | `0.6` | 0.0 .. 1.0 | Weight of one smoothing pass. |
| `StylizeResolveSmoothIterations` | `int32` | `0` | -1 .. 16 | Smoothing passes of the stylize result **after** the push-out of the skin from the hand. The push-out moves single display nodes out of the hand by their full overlap; on a deep grab (whole hand 2-3 cm in the thigh) that showed as faceted, capsule-shaped patches and a hard crease line around the hand. -1 = off (the previous behaviour), 0 = automatic (16), otherwise this count (rounded up to even). More passes = rounder bulges; the skin then hugs a single fingertip a little less (forearm poke peak 16.8 -> 12.8 mm in our test). |
| `StylizeResolveSmoothWeight` | `float` | `0.8` | 0.0 .. 1.0 | Weight of one such pass. |
| `StylizeShoulder` | `float` | `0.25` | 0.0 .. 1.0 | Share of the dent next to the hand that is lifted to soften the step at the edge of the dent. |
| `bStylizeAntiFold` | `bool` | `true` |  | Re-clamps the lift after smoothing so that narrow grooves do not fold through themselves. |
| `StylizeRidgeRound` | `float` | `0.3` | 0.0 .. 1.0 | Roundness of the finger ridges. |
| `StylizeLowPassMs` | `float` | `60` | 0.0 .. 300.0 | Time constant (ms) of the temporal low-pass on the stylize output while the hand moves (removes shimmer from tracker jitter). 0 = off. |
| `StylizeLowPassHoldMs` | `float` | `150` | 0.0 .. 500.0 | Time constant (ms) of the same low-pass while the hand is still. |
| `NumStyleRegions` | `int32` | `0` |  | Regions with baked style data (read only). |
| `AvgStylizeGpuMs` | `float` | `-1.f` |  | Average GPU time of the stylize passes per active region (ms, -1 = none yet; read only). |

**Category `SoftPoke|Bend`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bBendFollow` | `bool` | `true` |  | Regions follow all the bones that skin them, not only their attach bone (needs the skin data in the baked files; other regions are unaffected). |
| `BendSigmaMm` | `float` | `20` | 2.0 .. 100.0 | Kernel width (mm) of the anchor weights used to map the hand into the rest pose. |
| `BendReachMm` | `float` | `40` | 5.0 .. 200.0 | Reach (mm) beyond the nearest anchor. |
| `NumBendRegions` | `int32` | `0` |  | Regions that have skin data for the bend following (read only). |
| `NumBendMapped` | `int32` | `0` |  | Regions mapped with the bend frames in the last solve (read only). |
| `LastBendCpuMs` | `float` | `-1.f` |  | Game-thread time of the last bend mapping (ms; read only). |
| `MaxBendCpuMs` | `float` | `0.f` |  | Largest bend mapping time so far (ms; read only). |

**Category `SoftPoke|Jiggle`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bJiggle` | `bool` | `true` |  | Switch of the jiggle layer (modal soft-tissue dynamics on top of the contact result). |
| `JiggleAmount` | `float` | `1` | 0.0 .. 1.3 | Scales the whole jiggle layer. 0 = off (no extra pass), 1 = as baked, up to 1.3. |
| `JiggleDamping` | `float` | `1` | 0.5 .. 10.0 | Multiplies the damping ratio of every mode (0.5 = bouncier, 2 = tight). |
| `JiggleFreqScale` | `float` | `1` | 0.85 .. 4.0 | Multiplies the mode frequencies (raise it for stiffer-feeling tissue). |
| `JiggleImpact` | `float` | `1` | 0.0 .. 2.0 | Multiplies the kick a fast-loading hand (a slap) gives the tissue. 0 = off. |
| `JiggleModes` | `int32` | `0` | 0 .. 512 | Maximum number of modes used (0 = all baked modes). |
| `JiggleModesIdle` | `int32` | `64` | 0 .. 512 | Mode cap for regions that move but are not touched. |
| `JiggleInputLowpassHz` | `float` | `50` | 0.0 .. 240.0 | Cutoff (Hz) of the low-pass on the bone-motion input (0 = off). |
| `JiggleWakeThresholdMmS2` | `float` | `200` | >= 0.0 | A region is excited when its bone acceleration exceeds this (mm/s^2; 9810 = 1 g) or a hand touches it. |
| `JiggleHoldSec` | `float` | `0.25` | >= 0.0 | Seconds a region keeps running after the last excitation. |
| `JiggleSleepAmpMm` | `float` | `0.02` | >= 0.0 | A region goes to sleep when its displayed jiggle amplitude is below this (mm). |
| `JiggleMaxRunSec` | `float` | `15` | >= 0.5 | Hard cap of the quiet running time after the last excitation (s). |
| `bJigglePrefetchMovingRegions` | `bool` | `true` |  | Request loading of regions whose bone moves, so that they can jiggle (on-demand mode). |
| `NumJiggleRegions` | `int32` | `0` |  | Regions with baked jiggle data (read only). |
| `NumJiggleRunning` | `int32` | `0` |  | Regions running a jiggle pass in the last solve (read only). |
| `AvgJiggleGpuMs` | `float` | `-1.f` |  | Average GPU time of the jiggle passes (ms; read only). |
| `LastJiggleGpuMs` | `float` | `-1.f` |  | Last GPU time of the jiggle passes (ms; read only). |
| `MaxJiggleInputMmS2` | `float` | `0.f` |  | Largest bone-motion input of the last tick (mm/s^2); useful for tuning `JiggleWakeThresholdMmS2` (read only). |

**Category `SoftPoke|Ripple`** (contact ripple; needs `bJiggle`)

A push, release or knead makes the whole touched region wave, out to its base and far side (item 1: the physical dynamic part plus the stylised wave), a wide soft bulge rises around a dent (item 2) and, optionally, the wave crosses region borders (item 3). The wave and the bulge are stylised, display-only effects: `RippleWaveGain` 0 and `RippleBulgeFraction` 0 give the physical result of the baked model. They never feed the contact solve, so the dent next to the hand does not change. The wave follows the soft part that is touched, not only the region: a belly touch settles gradually around the solar plexus instead of running into the chest, a chest or buttock touch fills its whole mound up to the base (`bRippleWaveAnatomy`, `bRippleWaveCarry`). With `bRipple` off, or the console variable `softpoke.Ripple 0`, the output and the GPU cost are those of a build without the feature (apart from a 16 byte dummy buffer per solve).

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bRipple` | `bool` | `true` |  | Master switch of the contact ripple. |
| `bRippleContact` | `bool` | `true` |  | Item 1: the dynamic (wave) part of the displacement is not cut off by the far-field limit (`bLimitFarField`). The static dent is unchanged. |
| `RippleGain` | `float` | `1` | 0.0 .. 3.0 | Display gain of the wave far from the hand (1 = as the physics gives it). It never feeds the contact solve and is 1 at the contact. |
| `bRippleWave` | `bool` | `true` |  | Item 1b, the wave: every push, release or knead kicks the low, wide modes of the touched region, so the whole region (out to its base and its far side) wobbles for about a second and settles. It follows the RATE of the contact load (nothing moves in a steady hold), is display only and is faded near the hand. |
| `RippleWaveGain` | `float` | `1` | 0.0 .. 3.0 | Strength of the wave (0 = off = physical wave only). The size grows less than linearly with the gain because the wave is bounded (state limit and `RippleWaveMaxMm`): on the thigh press 3 gives about 1.7 times the 15-20 cm motion of 1 (median 0.70 to 1.20 mm), see [Behaviour and limits](BehaviourAndLimits.md). |
| `RippleWaveMaxMm` | `float` | `5` | 0.5 .. 10.0 | Soft limit (mm) of the displayed wave: linear up to 30 % of the limit, then a smooth tanh knee towards it (no flat plateau). Also limited to half the local soft-tissue thickness. |
| `RippleWaveShapeGainMax` | `float` | `6` | 0.0 .. 100.0 | The wave is divided by the local size of the baked mode shapes so that it has the same size everywhere in a region. Where the shapes barely move (next to a fixed region edge, for example the top of the chest under the collarbone) that gain gets very large and the wave piled up at its soft limit at the region rim. Above this gain the normalisation grows only with `RippleWaveShapeGainExponent`. 0 = no limit (the previous behaviour). Console override `softpoke.RippleWaveShapeGainMax`. |
| `RippleWaveShapeGainExponent` | `float` | `0.5` | 0.0 .. 0.9 | Above `RippleWaveShapeGainMax` the gain grows with this power of the unlimited gain (0 = hard limit: the wave fades with the mode shapes; 0.5 = it fades with the square root of their size). Console override `softpoke.RippleWaveShapeGainExponent`. |
| `RippleWaveSeamBandCm` | `float` | `4` | 0.0 .. 10.0 | Width (cm) of the seam band of a touched region: its nodes closer than this to another region show the wave of its own interior (where the mode shapes fade out near a region rim they would show almost nothing), and the neighbouring regions copy that interior wave too, so the wave crosses a region seam at the size it has inside the mound. Needs `bRippleWaveCarry`. 0 = off (the previous behaviour). Console override `softpoke.RippleCarrySeamBandCm`. |
| `RippleWaveMoundBlurCm` | `float` | `1` | 0.25 .. 6.0 | Smoothing radius (cm) of the mound membership. Console override `softpoke.RippleAnatBlurCm`. |
| `RippleWaveChestFadeCm` | `float` | `6` | 0.0 .. 15.0 | Distance (cm) over which the chest fades out beyond its base. 0 = half of `RippleWaveZoneBlendCm` (the previous behaviour, 4 cm; the buttocks keep it). With 6 the wave of a chest press runs on over the collarbone and fades out there instead of stopping at the top of the chest. Rebuilds the anatomy on a worker thread when changed. |
| `RippleWaveCutHz` | `float` | `9` | 2 .. 30 | Corner frequency of the modes that carry the wave (4th order low pass of the excitation). Lower = wider and slower, higher = more local detail. |
| `RippleWaveStrokeMs` | `float` | `300` | 50 .. 1000 | Length of the stroke whose load change gives `RippleWaveGain` times the static response. A longer value makes every stroke ring harder. |
| `RippleWaveReachCm` | `float` | `40` | 5 .. 200 | Reach of the wave. The wave field is normalised by the local size of the modal shapes, so it has the same size everywhere in the region (the baked modes alone fall off within a hand width). It is zero within about 2 cm of a contact centre, rises to full size at 5 cm, is full up to 8 cm and then dies off exponentially with this e-folding length: 80 % at 16 cm, 45 % at 45 cm (the base of a thigh). The wave of all points moves together (a standing wobble of the region), it is not a travelling wave. |
| `RippleWaveDamping` | `float` | `0` | 0.0 .. 20.0 | Extra damping rate (1/s) of the wave on top of the modal damping: higher = fewer visible cycles after a release. |
| `RippleHandClearMm` | `float` | `15` | 0.0 .. 60.0 | The wave and the bulge fade out within this distance of the hand capsules (smoothstep of the distance of the skin to the nearest capsule), and are never allowed to move a skin point by more than half its distance to the nearest capsule, so the extra motion can not carry the skin into the hand. 0 = no fade and no limit. |
| `bRippleWaveAnatomy` | `bool` | `true` |  | The wave follows the touched soft part instead of the region only. Three soft masses are found once at load time from the rest nodes alone (no bake data, no skin weights): the chest and the left and right buttock, as the places where the skin bulges out of a 3.5 cm neighbourhood; the chest / abdomen ramp comes from the skeleton heights (`spine_03` to `spine_04`, centred at 40 % of the way: the solar plexus). A contact is classified by its position (a mound, the belly, or none), latched while the wave rings and classified again when the contact moves more than 8 cm; a change of the soft part under a held contact is cross-faded over about 0.2 s (`softpoke.RippleAnatFadeMs`, default 60 ms time constant, 0 = at once). The wave is scaled by 1 inside the touched soft part and by about `RippleWaveOtherZoneGain` outside it (exactly: the floor divided by the envelope at the contact centre, so a contact on the border of a mound still gets the full wave), with a smooth transition. Display only (after the soft limit), nothing reaches the contact solve. Limbs, face and the back are not touched by it. Off = the wave of the previous version (same constants and passes; a 16 byte dummy buffer is still created per solve). Needs a recognised skeleton (`pelvis`, `spine_03`, `spine_04` or their VRoid / Mixamo equivalents, thighs, foot; one log line says when it is not and the wave then keeps the shape of the previous version). Chest, belly and buttock are found on the sample body only (convexity thresholds tuned on one character): another character may need `RippleWaveChestFullAt` or `RippleWaveZoneBlendCm` adjusted. |
| `RippleWaveOtherZoneGain` | `float` | `0.1` | 0.0 .. 1.0 | Wave gain outside the touched soft part (the chest wave on the belly, the buttock wave on the thigh and the back); 1 = no shaping. |
| `RippleWaveChestFullAt` | `float` | `0.7` | 0.2 .. 1.0 | Membership of the chest at which the wave of a chest touch reaches full size. 1 = the wave grows with the membership and is about half size at the base of the chest; 0.5 = full size up to the base, then the fade-out beyond it. Lower = a stronger wave under the breast and towards the armpit, a little more of it past the base and a steeper step where the chest ends at the collarbone. The buttocks do not use it (their wave is already full size at the base). Takes effect with the next contact. |
| `RippleWaveZoneBlendCm` | `float` | `8` | 2.0 .. 20.0 | Width of the transition: the chest / abdomen ramp is this wide (centred at the solar plexus), a buttock fades out over half of it beyond its base (the chest: `RippleWaveChestFadeCm`). A change rebuilds the anatomy on a worker thread (about 0.3 s on the sample body, nothing is blocked). |
| `bRippleWaveCarry` | `bool` | `true` |  | A mound is split over several regions (a buttock: glute, thigh, torso back and front), and a region that is not touched has no wave. The touched region publishes its wave on the nodes the neighbours need (one small pass of about 3 us) and the other regions that own a part of the same mound add it on their own nodes while it rings, so the wave reaches the base of the mound. Render only, no state, no solve: a region without a plan of its own gets one extra render pass (about 6 to 9 us GPU for the carry kernel plus the region's render pass) for about a second, nothing otherwise; a region that is awake but has no contact takes the idle short path and then the same carrier passes. Only regions that own at least 40 nodes of the mound carry. The cost on the sample body is +0.1 to +0.2 ms frame GPU time with both hands kneading the chest or the buttocks (see [Behaviour and limits](BehaviourAndLimits.md)). A region that is not loaded cannot carry (no region is loaded for it). Excludes `bRippleCoupling` (the old seam coupling shares the exchange buffer). |
| `bRippleBulge` | `bool` | `true` |  | Item 2: the flesh pushed away by a dent rises as a wide, low bulge around it. Needs the style data of the region (regions without it have no bulge). |
| `RippleBulgeFraction` | `float` | `3` | 0.0 .. 8.0 | Volume of the wide bulge as a fraction of the displayed dent volume (the physical value is 0.1 to 0.25, so the default is stylised: a 1 mm ring over 3 to 8 cm needs about 4 times the dent volume). Default result: +0.4 to +0.8 mm mean rise over the 3-8 cm ring on the thigh, upper arm and chest, nothing on the belly (stiff, 4 mm dent). |
| `RippleBulgeMaxMm` | `float` | `3` | 0.0 .. 8.0 | Highest point of the bulge (mm); also limited to half the local soft-tissue thickness. |
| `RippleBulgeNearMm` | `float` | `40` | 15 .. 60 | Distance from the contact (mm) where the bulge ring starts to rise (full height at about 1.3 times this). |
| `RippleBulgeFarMm` | `float` | `80` | 40 .. 300 | The bulge fades out between 0.47 and 1.47 times this distance from the contact (mm). |
| `RippleBulgeTauMs` | `float` | `80` | 20 .. 300 | Time constant of the bulge following the dent, for press and release alike. |
| `bRippleCoupling` | `bool` | `false` | | Item 3 (opt-in): the motion at the border of a region excites the modes of the neighbouring regions. Costs GPU time while a region next to a touched one is awake (see [Behaviour and limits](BehaviourAndLimits.md)). |
| `RippleCouplingGain` | `float` | `0.25` | 0.0 .. 0.6 | Strength of the seam transfer. The effective transfer (gain x (1 - `RippleHopLoss`)) is capped at 0.11, above which coupled regions can keep a small limit cycle. |
| `RippleHopLoss` | `float` | `0.6` | 0.2 .. 1.0 | Share of the energy lost at every region border. |
| `RippleSeamDepthMm` | `float` | `25` | 5 .. 60 | Depth of the receiving strip inside the neighbouring region (mm). |
| `RippleSeamRadiusMm` | `float` | `8` | 3 .. 20 | Pairing radius (mm) used to find the border between two regions from their rest meshes. |
| `RippleSeamMinSamples` | `int32` | `8` | 1 .. 64 | Fewer node pairs than this = the two regions are not neighbours. |
| `RippleMaxMm` | `float` | `3` | 0.5 .. 8.0 | Safety limit (mm) of the wave amplitude a neighbour can receive through a seam. |
| `RippleRegionScale` | `TMap<FName,float>` | empty |  | Strength multiplier per region name (gain, bulge, coupling). Regions not listed use the built-in table: face, head, neck, hands and feet 0.4, everything else 1. |
| `NumRippleSeams` | `int32` | `0` |  | Neighbour strip nodes found at run time, summed over the regions (read only). |
| `NumRippleInfluenced` | `int32` | `0` |  | Regions currently excited through a seam (read only). |
| `NumRippleMoundNodes` | `int32` | `0` |  | Nodes of the three soft masses (chest, left and right buttock) above half membership (read only, 0 until the anatomy is built). |
| `NumRippleCarriers` | `int32` | `0` |  | Regions that carried the wave of a neighbour in the last tick (read only). |
| `RippleAnatomyBuildMs` | `float` | `0` |  | Build time of the last anatomy on the worker thread (ms, read only). |

**Category `SoftPoke|Solver`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bUseGpuSolver` | `bool` | `true` |  | Use the GPU solver. Off = a simple CPU placeholder (capsule push-out) that does not use the baked data. |
| `bSkinFill` | `bool` | `true` |  | Vertices that no region maps follow their nearest mapped neighbours smoothly (needs the fill data created by the import). |
| `bSolveInEditor` | `bool` | `false` |  | Run the solver in the editor viewport too. Off by default so that opening a level does not start the GPU solver. Play-In-Editor, Simulate and game worlds always solve. |
| `NormalIterations` | `int32` | `20` | 1 .. 200 | Iterations of the normal-force solve. |
| `FrictionIterations` | `int32` | `80` | 1 .. 400 | Iterations of the friction solve. |
| `ContactBandMm` | `float` | `3` |  | Contact band (mm): skin nodes closer to the hand than this join the contact set. |
| `NormalOmega` | `float` | `0.7` | 0.3 .. 1.0 | Under-relaxation of the normal load update (lower = steadier contact, 1 = none). |
| `NormalRelaxTauMs` | `float` | `0` | 0.0 .. 500.0 | > 0 makes the under-relaxation independent of the solve rate (time constant in ms). 0 = fixed `NormalOmega`. |
| `NormalReleaseTauMs` | `float` | `0` | 0.0 .. 500.0 | Time constant (ms) of a slower release of a vanishing contact load. 0 = off (default). |
| `CandidateRangeMm` | `float` | `30` |  | Range (mm) around the hand in which skin nodes are considered for contact. |
| `FrictionMuOverride` | `float` | `-1` |  | Friction coefficient. <= 0 uses the value baked into the asset. |
| `MaxContactNodes` | `int32` | `3072` |  | Upper bound of contact nodes per region. |
| `MaxLoadedNodes` | `int32` | `2048` |  | Upper bound of loaded nodes per region. |
| `DisplacementKneeMm` | `float` | `40` |  | Safety clamp of the displacement applied to a vertex: unchanged below this (mm), smooth saturation above. |
| `DisplacementMaxMm` | `float` | `70` |  | Maximum displacement the safety clamp saturates to (mm). |
| `bThicknessClamp` | `bool` | `true` |  | Thickness-aware clamp: a region with baked soft-tissue thickness gets a tighter clamp (knee and maximum scaled from its thickness), so thin tissue over bone (jaw, chin, cheekbone) cannot be pushed far, also where the penetration limiter cannot see the contact (open patch rims, e.g. under the jaw). Never loosens the two values above. |
| `ThicknessKneeFraction` | `float` | `0.5` |  | Knee of the thickness clamp as a fraction of the region thickness (and, per vertex, of the vertex's own thickness). 0.6 -> 0.5. |
| `ThicknessMaxFraction` | `float` | `0.75` |  | Maximum of the thickness clamp as a fraction of the thickness. Default 0.75, so a dent stays within what the penetration limiter allows (0.7 x thickness); a deep forearm press no longer crushes two thirds of the tissue. |
| `ThicknessPercentile` | `float` | `0.9` |  | Which thickness of the region (percentile of its baked per-node thickness) sets the clamp. |
| `bPerVertexThicknessClamp` | `bool` | `true` |  | Part of `bThicknessClamp`: every render vertex is also clamped by its own soft-tissue thickness `t` (knee `ThicknessKneeFraction x t`, maximum `ThicknessMaxFraction x t`), so the kneecap, shin, elbow, ankle, collarbone or cheekbone barely move while the thigh or cheek next to them still give. Applied when a region loads. |
| `bAnatomicalThickness` | `bool` | `true` |  | Caps the thickness used by the penetration limiter and the per-vertex clamp with adult soft-tissue depths at bony landmarks (patella, tibia, malleoli, olecranon, epicondyles, wrist, clavicle, acromion, sternum line, spinous processes, zygoma, mandible border, chin, brow, forehead ...). The landmarks are found on the skin of the loaded regions from the skeleton (UE5 / MetaHuman, VRoid, Mixamo bone names), at load time, without a rebake. A skeleton that is not recognised keeps the baked thickness (see `AnatomySummary`). |
| `AnatomyTissueScale` | `float` | `1.0` | 0.3 .. 4.0 | Multiplies every landmark tissue depth (1 = adult defaults; higher for a softer / heavier character, lower for a very lean one). Applied when the regions load. |
| `bLimitFarField` | `bool` | `true` |  | Locality: the displacement of a region fades out between `FarFieldRadiusMm` and `FarFieldRadiusMm + FarFieldFalloffMm` from the hand's contact points (up to four per region and tick). Removes the small far-field motion of the linear tissue model (a deep thigh poke moved skin 15-22 cm away by up to 1.9 mm). Regions that only jiggle are not affected. |
| `FarFieldRadiusMm` | `float` | `90` | 20 .. 500 | Full displacement inside this distance (mm) from a contact point. |
| `FarFieldFalloffMm` | `float` | `60` | 10 .. 500 | Width (mm) of the fade beyond `FarFieldRadiusMm`. |
| `bBoneShadow` | `bool` | `true` |  | Needs `bLimitFarField`. Skin behind the bone under a contact does not move: per contact point, a render vertex deeper (along the inward skin normal) than the soft-tissue thickness at the contact plus `BoneShadowStartMm` fades out over 14 mm. Without it the far side of a forearm moved about 2 mm under a deep press. |
| `BoneShadowStartMm` | `float` | `8` | 0 .. 100 | Extra depth (mm) below the tissue thickness at the contact before the bone shadow starts. Larger = the far side of a limb follows more. |
| `AnatomySummary` | `FString` |  |  | Read only: the landmarks the anatomical model found and how many limiter nodes they capped. |
| `bSkipWhenHandFar` | `bool` | `true` |  | Skip regions (after clearing their state once) while no capsule is near them. |

**Category `SoftPoke|Diagnostics`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `NumFillVertices` | `int32` | `0` |  | Fill vertices of the body (0 = no fill data; read only). |
| `bDumpEachSolve` | `bool` | `false` |  | Development: write the displacement of every solve to disk (`DumpDirectory`). Leave off. |
| `bDumpContactState` | `bool` | `false` |  | Development: also write the contact state with `bDumpEachSolve`. Leave off. |
| `DumpFrameLimit` | `int32` | `64` |  | Development: budget of dumped solve ticks. |
| `DumpTag` | `FString` | `"ue"` |  | Development: file name tag of the dumps. |
| `DumpDirectory` | `FString` |  |  | Development: dump folder (empty = `<Project>/Saved/SoftPoke`). |
| `GpuTimingLogInterval` | `int32` | `10` |  | Log "SoftPoke GPU ... ms" every N solves (0 = off). |
| `bGpuSolverActive` | `bool` | `false` |  | True while the GPU solver runs (read only). |
| `Status` | `FString` |  |  | Why the solver is running or not (for example the DirectX 12 / Shader Model 6 message; read only). |
| `SolveCount` | `int32` | `0` |  | Solves so far (read only). |
| `LastGpuMs` | `float` | `-1.f` |  | GPU time of the solver passes summed over the regions solved in a frame: last (ms, -1 = none yet; read only). |
| `AvgGpuMs` | `float` | `-1.f` |  | The same, average (ms; read only). |
| `MaxGpuMs` | `float` | `0.f` |  | The same, maximum (ms; read only). |
| `LastFrameSolveGpuMs` | `float` | `-1.f` |  | GPU time from the first to the last region solve of a frame, gaps between the solves included (RHI timestamps, a few frames old; ms, read only). Also the `csvprofile` column `SoftPoke/SolveFrameGpuMs`. |
| `MaxFrameSolveGpuMs` | `float` | `0.f` |  | The same, maximum since play started (ms; read only). |
| `NumDeferredSolves` | `int32` | `0` |  | Region solves deferred by `SolveGpuBudgetMs` in the last frame (read only; `csvprofile` column `SoftPoke/SolvesDeferred`). |
| `TotalDeferredSolves` | `int32` | `0` |  | The same, total since play started (read only). |
| `LastPredictedSolveMs` | `float` | `0.f` |  | Predicted GPU cost of the solves that ran in the last frame (ms, from the measured cost of each region; read only; `csvprofile` column `SoftPoke/SolvePredictedMs`). |
| `GpuTimeByActiveCount` | `FString` |  |  | Average GPU time per frame by number of active regions, as text (read only). |

**Category `SoftPoke|Regions`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `ActivationMarginMm` | `float` | `15` | >= 0.0 | A region wakes up when a capsule is closer than the candidate range plus this margin (mm). |
| `ActivationHysteresisMm` | `float` | `25` | >= 0.0 | A region sleeps again only when every capsule is farther than the activation distance plus this (mm). |
| `bForceAllRegionsActive` | `bool` | `false` |  | Diagnostics: solve every region every frame regardless of the hand position. |
| `SolveGpuBudgetMs` | `float` | `-1` | >= -1.0 | GPU budget of the region solves per frame (ms). When the predicted cost of this frame's solves (each region's own measured cost) is above it, the solves that do not fit wait one frame: such a region shows its last result for that frame and is solved first in the next one, never two frames in a row, and its next step covers the skipped time (jiggle and wave keep their speed, the friction and approach-side state see the whole hand motion). Two hands on several regions then cost about the budget instead of a dropped headset frame; the regions that waited update at half the frame rate while the budget is exceeded. -1 = automatic: 3 ms while the engine renders in stereo (VR headset), unlimited otherwise. 0 = unlimited. Not applied while dumping (`bDumpEachSolve`) or without `bLazyRegionLoading`. `GetEffectiveSolveGpuBudgetMs()` returns the value in effect. Console override `softpoke.SolveBudgetMs`. |
| `NumRegions` | `int32` | `0` |  | Regions of the body (read only). |
| `NumActiveRegions` | `int32` | `0` |  | Regions solved in the last frame (read only). |
| `ActiveRegionNames` | `FString` |  |  | Names of the regions solved in the last frame (read only). |
| `RegionSummary` | `FString` |  |  | One line per region: name, bone, mapped render vertices, mirror, solver buffers (read only). |

**Category `SoftPoke|Streaming`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bLazyRegionLoading` | `bool` | `true` |  | Load region data on demand (a region is resident only while a hand is near it) within `VramBudgetMB`. |
| `VramBudgetMB` | `float` | `640.f` | >= 16.0 | VRAM budget of the resident region data (MB). Regions beyond it are evicted. The default holds the whole sample body (340 MB) with room to spare: with two hands nearly every region is wanted at some point, and with a budget below the body's total the regions evict and reload each other (each reload is an upload of 10 to 100 MB). A body larger than the budget still works (least recently used regions are evicted first). |
| `EvictCooldownSec` | `float` | `8.f` | >= 0.0 | A resident region is evicted only when no capsule was near it for this long (s). |
| `IdleUnloadSec` | `float` | `0.f` | >= 0.0 | Evict a region after this many idle seconds even under budget (0 = only when over budget). |
| `WakeAheadMarginMm` | `float` | `60.f` | >= 0.0 | Prefetch starts when the predicted hand distance to a region is below the solve reach plus this margin (mm). |
| `PrefetchLookaheadSec` | `float` | `0.5` | >= 0.0 | How far ahead (s) the hand distance is predicted for prefetching. |
| `UploadMBPerFrame` | `float` | `16.f` | >= 1.0 | GPU upload budget per frame (MB). |
| `FadeInSec` | `float` | `0.1` | >= 0.0 | A region that becomes ready while the hand is already in range fades its displacement in over this time (s). |
| `DebugEvictAllEverySec` | `float` | `0.f` | >= 0.0 | Soak test: evict every idle region at this interval (s, 0 = off). |
| `NumResidentGroups` | `int32` | `0` |  | Resident units (region + mirror partner; read only). |
| `ResidentMB` | `float` | `0.f` |  | VRAM used by resident regions (MB; read only). |
| `NumLoadingGroups` | `int32` | `0` |  | Units being loaded (read only). |
| `TotalLoads` | `int32` | `0` |  | Loads so far (read only). |
| `TotalEvictions` | `int32` | `0` |  | Evictions so far (read only). |
| `LastLoadLatencyMs` | `float` | `-1.f` |  | Latency from load request to ready, last (ms; read only). |
| `MaxLoadLatencyMs` | `float` | `0.f` |  | Latency from load request to ready, maximum (ms; read only). |
| `MaxStreamingTickMs` | `float` | `0.f` |  | Longest game-thread time one streaming tick spent (ms; read only). |
| `CreateHitchMs` | `float` | `-1.f` |  | Game-thread time of the last solver creation (the start-up hitch, ms; read only). |
| `NumLateContactFrames` | `int32` | `0` |  | Contacts that found their region data not ready yet (read only). |
| `NumDegradedContact` | `int32` | `0` |  | Contact units that could not be loaded within the VRAM budget (read only). |

**Category `SoftPoke|ToonDent`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bDentOverlay` | `bool` | `false` |  | Toon dent overlay (a tint that makes the dent readable on flat-shaded characters). Off by default. |
| `DentScaleMm` | `float` | `12` | 1.0 .. 100.0 | Full scale of the dent mask (mm). Must equal the `range=` the mask deformer was built with (default 12). |
| `DentOverlayMaterial` | `UMaterialInterface` |  |  | Overlay material (`M_SoftPoke_DentOverlay` or an instance of it). One dynamic instance per slot is made from it. |
| `DentMaskDeformer` | `UMeshDeformer` |  |  | Deformer graph that writes the dent mask while the overlay is on (build with `mask=1`). |
| `DentOverlaySlotNames` | `TArray<FName>` |  |  | Material slots that get the overlay. Empty = every slot except translucent or additive ones. |
| `DentOverlaySlotOverrides` | `TMap<FName, FSoftPokeDentSlotOverride>` |  |  | Per-slot overrides of the overlay feathering: slot name to (`Feather` 0..1, `FeatherHiMm`), written onto that slot's overlay instance. Slots not listed keep the values of `DentOverlayMaterial`. |
| `ShadeColorSourceParam` | `FName` | `"mtoon_ShadeColor"` |  | Colour parameter copied from each slot material into the overlay (by name). |
| `ShadeColorDestParam` | `FName` | `"ShadeColor"` |  | Vector parameter of the overlay material that receives the copied colour. |
| `ShadeColorScale` | `float` | `0.85` | 0.0 .. 2.0 | The copied colour is multiplied by this so that the dent is darker than the lit skin. |
| `DentOverlayMaxDrawDistance` | `float` | `0` | >= 0.0 | Overlay maximum draw distance (cm, 0 = engine default). |

**Functions**

| Function | Signature | Description |
|---|---|---|
| `SetUnderChinExcluded` | `void SetUnderChinExcluded(bool bExcluded)` | Switches the under-chin exclusion on or off (the face driver does this from the jaw-open value). (BlueprintCallable) |
| `IsUnderChinExcluded` | `bool IsUnderChinExcluded()` | True while the under-chin exclusion is on. (BlueprintPure) |
| `SetVirtualBoneMotion` | `void SetVirtualBoneMotion(FName Bone, const FTransform& OffsetComponentCm)` | Test hook: displaces the motion input of the regions attached to a bone without moving the skeleton. (BlueprintCallable) |
| `RemoveVirtualBoneMotion` | `void RemoveVirtualBoneMotion(FName Bone)` | Removes the virtual motion of one bone. (BlueprintCallable) |
| `ClearVirtualBoneMotions` | `void ClearVirtualBoneMotions()` | Removes all virtual bone motions. (BlueprintCallable) |
| `ResetJiggle` | `void ResetJiggle()` | Zeroes the modal state of every region (all jiggle stops; the next excitation starts from rest). (BlueprintCallable) |
| `SetCapsules` | `void SetCapsules(const TArray<FSoftPokeCapsule>& InCapsules)` | Replaces the current hand: capsules in the `TargetMesh` component space, cm. Called by the hand rig every frame; call it yourself to drive the body from your own code. (BlueprintCallable) |
| `ResetSolver` | `void ResetSolver()` | Zeroes the solver state (for example when the scene resets). (BlueprintCallable) |
| `ResetDumpBudget` | `void ResetDumpBudget()` | Development: restart the dump budget. (BlueprintCallable) |
| `HasPenetrationSurfaces` | `bool HasPenetrationSurfaces()` | True when the skin surfaces for the penetration limiter are available. (BlueprintPure) |
| `RecreateSolvers` | `void RecreateSolvers()` | Destroys and recreates the region solvers (after assigning a new `BodyAsset` at run time). (BlueprintCallable) |
| `PrefetchRegions` | `int32 PrefetchRegions(FName RegionName)` | Starts loading a region and its mirror partner (`NAME_None` = all). Returns the number of units requested. (BlueprintCallable) |
| `EvictRegions` | `int32 EvictRegions(FName RegionName, bool bIgnoreCooldown)` | Evicts resident regions that are not in contact (`NAME_None` = all). Returns the number evicted. (BlueprintCallable) |
| `AreRegionsReady` | `bool AreRegionsReady(FName RegionName)` | True when every region (or the named one) is resident. (BlueprintPure) |
| `SetDentOverlayEnabled` | `void SetDentOverlayEnabled(bool bEnable)` | Turns the overlay on or off at run time. (BlueprintCallable) |
| `IsDentOverlayApplied` | `bool IsDentOverlayApplied()` | True while the overlay is applied to the target mesh. (BlueprintPure) |
| `MatchShadeColors` | `int32 MatchShadeColors()` | Re-copies the shade colour from the slot materials into the overlay instances. Returns the number of slots updated. (BlueprintCallable) |
| `CopyColorParameterByName` | `static bool CopyColorParameterByName(UMaterialInterface* Source, FName SrcParam, UMaterialInstanceDynamic* Dest, FName DstParam, float Scale = 1.0f)` | Helper: reads a vector parameter by name from a material and writes it, scaled, to a dynamic instance. (BlueprintCallable) |

## SoftPokeHandRig

`USoftPokeHandRig` (class group Rendering, spawnable). Put it on the body actor next to the `SoftPokeComponent`. Data flow: hand source (OpenXR, UDP or your own) -> One-Euro filter -> penetration limiter ("physics hand") -> capsules -> `USoftPokeComponent::SetCapsules`. It runs in Play-In-Editor and game worlds. The limited ("proxy") hand is exposed in world space so that a visible hand mesh can follow it.

**Category `SoftPoke|Input`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `BodyActor` | `AActor` |  |  | Actor with the body mesh and the `SoftPokeComponent`. Default: the owner. |
| `SecondaryActors` | `TArray<TObjectPtr<AActor>>` |  |  | Extra actors whose secondary SoftPoke components are fed with the same hand (for example a separate head actor). |
| `Source` | `USoftPokeHandSource` |  |  | The joint source instance. Empty = created from `SourceClass` at Begin Play. If `SourceClass` names another class (not the Auto default), `SourceClass` wins at Begin Play and a warning is logged. |
| `SourceClass` | `TSubclassOf<USoftPokeHandSource>` | `SoftPokeHandSourceAuto` |  | Class of the joint source to create: `SoftPokeHandSourceAuto` (default: OpenXR hand tracking, else motion controllers, else `AutoFallbackClass`), `SoftPokeHandSourceOpenXR`, `SoftPokeHandSourceController` (synthesized hand from motion controllers), `SoftPokeHandSourceNative` (built-in webcam tracker), `SoftPokeHandSourceUdp` (Python tracker, advanced), or your own subclass of `SoftPokeHandSource`. |
| `AutoFallbackClass` | `TSubclassOf<USoftPokeHandSource>` | empty |  | Used when `SourceClass` is `SoftPokeHandSourceAuto`: the source that drives a hand when neither OpenXR hand tracking nor a motion controller tracks it (for example `SoftPokeHandSourceNative` for the webcam). Started only when needed. The sample maps set `SoftPokeHandSourceNative`. |
| `bLeftHand` | `bool` | `true` |  | Use the left hand. |
| `bRightHand` | `bool` | `true` |  | Use the right hand. |

**Category `SoftPoke|Filter`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bFilter` | `bool` | `true` |  | One-Euro filter on the joints (once per new sample). |
| `MinCutoffHz` | `float` | `0.3` | >= 0.05 | One-Euro minimum cutoff (Hz). Smaller = smoother when the hand is still. |
| `Beta` | `float` | `1` | >= 0.0 | One-Euro speed coefficient. Larger = less lag when moving fast. |
| `DerivativeCutoffHz` | `float` | `0.8` | >= 0.05 | Cutoff (Hz) of the low-pass on the speed estimate. |
| `SpeedFloorCmPerSec` | `float` | `4` | >= 0.0 | Per-joint speed (cm/s) below which a joint does not open the cutoff (suppresses jitter). |
| `HandSpeedFloorCmPerSec` | `float` | `2.5` | >= 0.0 | The same for the common motion of the whole hand (cm/s). |
| `bContactFilter` | `bool` | `true` |  | Contact-aware filtering: while the raw tracked hand is below the skin the filter is stiffer along the contact normal. |
| `ContactCutoffScale` | `float` | `0.6` | 0.05 .. 1.0 | Cutoff scale while in contact. |
| `ContactDeadBandMm` | `float` | `0.5` | 0.0 .. 3.0 | Half width (mm) of the dead band along the contact normal (0 = off). |
| `ContactReleaseSec` | `float` | `0.15` | 0.0 .. 1.0 | The contact state is held this long (s) after contact ends (no flicker at the skin line). |
| `ThinTissueAllowedMm` | `float` | `10` | 0 .. 30 | Over bone: while the allowed depth at the contact is below this value (kneecap about 3.5 mm), the cut-off along the contact normal falls toward `ThinTissueCutoffScale` and the dead band grows toward `ThinTissueDeadBandMm`, linearly in the allowed depth. Tracker jitter of a few mm is as large as the whole dent over a bone. 0 = off. |
| `ThinTissueCutoffScale` | `float` | `0.12` | 0.02 .. 1.0 | Cut-off scale along the normal with bone right under the skin. |
| `ThinTissueDeadBandMm` | `float` | `2.5` | 0 .. 6 | Dead band half width (mm) along the normal with bone right under the skin. |

**Category `SoftPoke|Limiter`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `Limiter` | `FSoftPokeLimiterSettings` |  |  | Settings of the penetration limiter ("physics hand"); see `FSoftPokeLimiterSettings` below. |
| `JointLimits` | `FSoftPokeHandJointLimits` | knuckle 90 / 25 / 20, middle joint 105 / 5, end joint 85 / 15, thumb 70 / 85 |  | anatomical finger joint ranges in degrees (flexion / hyperextension / sideways), used by `Limiter.bAnatomicalJointLimits` and by the hand mesh driver. |

**Category `SoftPoke|Head`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bHeadCollider` | `bool` | `false` |  | Adds capsules for the user's head, taken from the `head` field of the tracker packets, so that the hand-side body can be touched by the head (default off). |
| `HeadCapsuleCount` | `int32` | `2` | 1 .. 3 | Head capsules: 1 = cranium, 2 = + face/jaw, 3 = + neck. |
| `HeadPivotOffsetCm` | `FVector` | `FVector(-4.5, 0.0, 4.5)` |  | Skull centre relative to the tracker's head origin, in the head frame (x nose, y right, z up), cm at head scale 1. |
| `HeadSizeMultiplier` | `float` | `1` | 0.2 .. 3.0 | Extra size factor on the head capsules. |

**Category `SoftPoke|Debug`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `HeadStatus` | `FSoftPokeHandStatus` |  |  | Diagnostics of the head collider (same struct as the hand status; read only). |
| `bDrawDebug` | `bool` | `false` |  | Draw the tracked capsules (blue) and the limited capsules (green, only where they differ). |
| `Status` | `FString` |  |  | Source status text (for example "OpenXR: 0 hand tracker(s)" or the UDP counters; read only). |
| `LeftStatus` | `FSoftPokeHandStatus` |  |  | Limiter diagnostics of the left hand (read only). |
| `RightStatus` | `FSoftPokeHandStatus` |  |  | Limiter diagnostics of the right hand (read only). |

**Functions**

| Function | Signature | Description |
|---|---|---|
| `GetNumSecondaryComponents` | `int32 GetNumSecondaryComponents()` | Number of secondary components being driven (0 = single component). (BlueprintPure) |
| `GetHeadFrame` | `bool GetHeadFrame(FSoftPokeHeadFrame& Out)` | Latest head frame (world cm). False without a head. (BlueprintCallable) |
| `GetHeadBlendshapes` | `bool GetHeadBlendshapes(TMap<FName, float>& OutBlendshapes)` | Latest blendshapes as ARKit name -> value, for driving an avatar's face yourself. (BlueprintCallable) |
| `GetHeadProxyCapsules` | `bool GetHeadProxyCapsules(TArray<FVector>& OutA, TArray<FVector>& OutB, TArray<float>& OutRadius)` | The head capsules the solver sees (after the limiter), world space. (BlueprintCallable) |
| `GetProxyJoints` | `bool GetProxyJoints(ESoftPokeHandSide Side, TArray<FVector>& OutPositions, TArray<FQuat>& OutRotations)` | The limited ("proxy") joints and rotations in the plugin skeleton (21), world cm. False if the hand is not tracked. (BlueprintCallable) |
| `GetProxyKeypoints` | `bool GetProxyKeypoints(ESoftPokeHandSide Side, TArray<FTransform>& OutKeypoints26)` | The same as 26 `EHandKeypoint` transforms in world space, to draw a hand mesh at the limited pose. (BlueprintCallable) |
| `GetTargetJoints` | `bool GetTargetJoints(ESoftPokeHandSide Side, TArray<FVector>& OutPositions)` | The tracked, filtered, not limited joints (world cm). (BlueprintCallable) |
| `ResetHands` | `void ResetHands()` | Clears the filter and limiter state of both hands. (BlueprintCallable) |

### Limiter settings (`FSoftPokeLimiterSettings`)

The `Limiter` property of the hand rig. The allowed compression depth at a skin point is `min(MaxDepthMm, MaxThicknessFraction * T, max(ThicknessFraction * T, MinDepthMm))`, where `T` is the soft-tissue thickness there (the baked value, capped at the bony landmarks when the component's `bAnatomicalThickness` is on). Openings of a region patch (the eye / nose / mouth opening of the face region) are closed for the limiter with a shallow dome, so a hand stops there (`softpoke.LimiterCapHoles`). This is a tuned look-and-feel limiter, not a physical constant (see [Behaviour and limits](BehaviourAndLimits.md)).

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bEnabled` | `bool` | `true` |  | Master switch of the proxy hand. Off = the solver gets the filtered tracked hand as it is. |
| `Strength` | `float` | `1` | 0.0 .. 1.0 | How hard the proxy is pushed out of the tissue (0 = off, 1 = full). |
| `ThicknessFraction` | `float` | `0.7` | 0.05 .. 1.0 | Allowed compression as a fraction of the local tissue thickness. |
| `MaxThicknessFraction` | `float` | `0.85` | 0.1 .. 1.0 | The allowed depth never exceeds this fraction of the thickness (stops tunnelling on thin spots). |
| `MinDepthMm` | `float` | `2` | >= 0.0 | Guard for degenerate thickness (mm). |
| `MaxDepthMm` | `float` | `28` | >= 1.0 | Absolute maximum allowed depth (mm). |
| `SoftMargin` | `float` | `0.75` | 0.3 .. 0.98 | The limiter is inactive below this fraction of the allowed depth and bends smoothly towards it above. |
| `RelaxSeconds` | `float` | `0.12` | >= 0.01 | Time constant (s) with which the proxy returns to the tracked hand (no popping). |
| `Iterations` | `int32` | `10` | 1 .. 30 | Maximum solver iterations of the limiter per frame. |
| `MaxRotationDeg` | `float` | `20` | 0.0 .. 60.0 | Largest rotation of the proxy hand about the wrist (degrees). |
| `RimGuardMm` | `float` | `8` | 0.0 .. 40.0 | Guard band (mm) at the open end of a baked patch (for example the wrist end of the forearm). |
| `RimRampMm` | `float` | `10` | 0.0 .. 40.0 | Ramp length (mm) for samples that never came down onto the skin near an open end. |
| `RateTMmPerSec` | `float` | `150` | 0.0 .. 2000.0 | Speed limit (mm/s) of the proxy translation while the open-end field is involved. |
| `RateWDegPerSec` | `float` | `90` | 0.0 .. 720.0 | Speed limit (deg/s) of the proxy rotation while the open-end field is involved (0 = off). |
| `bPerFingerRelax` | `bool` | `true` |  | Every finger of the proxy hand relaxes towards the tracked finger on its own, so a hand that is held on the skin keeps its dent while the tracker jitters. |
| `MaxFingerOffsetMm` | `float` | `45` | 0.0 .. 200.0 | Largest distance (mm) of a proxy finger joint from the tracked finger (0 = no cap): the proxy never folds a finger back without limit. |
| `bAnatomicalJointLimits` | `bool` | `true` | | keep every finger joint of the tracked and the proxy hand inside `JointLimits` (rig property), applied after the input filter and after every push-back step, bone lengths kept; the push-back may only bend a joint back relative to the tracked finger, never curl it further or sideways. See [Behaviour and limits](BehaviourAndLimits.md). |
| `AxialPushRigidBelowMm` | `float` | `2` | 0.0 .. 30.0 | where the allowed depth of a contact is below this (mm; fading out at twice the value), the part of the push that runs along a finger moves the whole proxy hand back instead of folding that finger: a finger pressed head-on against a rigid part (the nose, about 1.2 mm allowed) stops straight. Bone (4-6 mm allowed) and soft tissue keep the per-finger compliance. 0 = off. |
| `bRigidPushAlongHand` | `bool` | `true` | | the rigid push-back of the palm samples moves the whole proxy hand back against the direction of the hand (wrist to middle knuckle) instead of along the contact normal, so the hand does not slide over a sloped surface and a held press under tracker noise stays steadier. Falls back to the normal at grazing angles. Off = along the normal (r5). |
| `PushAlongHandMinCos` | `float` | `0.3` | 0.0 .. 0.9 | `bRigidPushAlongHand` applies only where the cosine between the retreat direction and the contact normal is above this (fully 0.25 above it). |

### Hand status (`FSoftPokeHandStatus`)

`LeftStatus`, `RightStatus` and `HeadStatus` of the hand rig.

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bTracked` | `bool` | `false` |  | The hand is tracked. |
| `bProxyActive` | `bool` | `false` |  | The limiter is currently moving the proxy away from the tracked hand. |
| `MaxViolationMm` | `float` | `0.f` |  | Largest remaining violation of the proxy after the iterations (mm). |
| `MaxJointOffsetMm` | `float` | `0.f` |  | Largest distance between a tracked and a proxy joint (mm). |
| `TranslationMm` | `float` | `0.f` |  | Translation of the proxy relative to the tracked hand (mm). |
| `RotationDeg` | `float` | `0.f` |  | Rotation of the proxy relative to the tracked hand (degrees). |
| `DeepestTargetDepthMm` | `float` | `0.f` |  | Deepest sample of the tracked hand below the skin (mm, 0 when outside). |
| `ProxyDepthMm` | `float` | `0.f` |  | Deepest sample of the limited hand below the skin (mm). |
| `AllowedDepthMm` | `float` | `0.f` |  | Allowed depth at that sample (mm). |

## SoftPokeHandMeshDriver

Opt-in (`bEnabled` is false by default; nothing is moved). Poses a hand skeletal mesh on a `UPoseableMeshComponent` from the **contact-resolved proxy hand** (`USoftPokeHandRig::GetProxyKeypoints`), the hand the deformation actually uses, instead of the raw tracked hand, so the visible fingers and palm do not sink into the skin. Per tick the proxy keypoints are matched to the mesh at the wrist (wrist, middle knuckle, index and little knuckles), then the bones are walked wrist -> metacarpal -> proximal -> intermediate -> distal in the **parent frame** (forward kinematics): each bone starts at the **end of its parent bone** (the metacarpals and the thumb's first bone at their place in the rigid palm), starts from its parent's corrected frame (rest local rotation, or the tracked local rotation) and gets the minimal swing that aims it at its proxy child keypoint. Its length follows the proxy segment through an **axial bone scale** clamped to `MinBoneScale` .. `MaxBoneScale` of the mesh's own bone at the fitted size, so the joints of the mesh always meet: no keypoint configuration a tracker or the limiter can produce stretches the skin between two bones or pushes it into a fold. A final check rejects a pose with a non-finite value, a bone scale outside the range or joints that do not meet and shows the last good pose instead. Roll and the bend plane stay intact. The size is fitted to the palm and finger lengths of the proxy hand (`bFitScaleToFingers`). The mesh is a hand only: frame the camera so the wrist leaves the picture, or attach your own forearm / sleeve to the wrist bone.

| Property | Type | Default | Description |
|---|---|---|---|
| `bEnabled` | `bool` | `false` | Pose the mesh every tick. |
| `HandRig` | `USoftPokeHandRig*` | none | The rig whose proxy hand is shown (empty = the rig on the owner). |
| `PoseableMesh` | `UPoseableMeshComponent*` | none | The hand mesh component. The driver moves and scales it. |
| `Side` | `ESoftPokeHandSide` | `Right` | Which proxy hand; the mesh must be that hand. |
| `RollSource` | `ESoftPokeHandRollSource` | `HandFrame` | `HandFrame`: roll from the hand frame (works with every source). `Tracked`: start every bone from the tracked local rotation (OpenXR). |
| `BoneMapOverride` | `TArray<FName>` | empty | 26 names for the `EHandKeypoint` entries (0 palm, 1 wrist, 2-5 thumb, 6-10 index, 11-15 middle, 16-20 ring, 21-25 little; None = not driven). Empty = the UE5 / XR mannequin / MetaHuman naming (`hand_r`, `thumb_01_r`, `index_metacarpal_r`, `index_01_r` ..., `_l` for the left hand). |
| `ScaleMultiplier` | `float` | `1` | Relative to the automatic fit. |
| `HandOffsetCm` | `FVector` | `0` | Offset of the whole mesh in the wrist frame (cm), e.g. to lift it off the skin. |
| `bAlignRoll` | `bool` | `false` | Experimental: after the swing, twist each bone about its own axis so that its rest side reference follows the hand's side axis (or the bend-plane normal). No visible difference with correct input in our tests. |
| `bDrawDebug` | `bool` | `false` | Draw the proxy keypoint segments (green) and the posed bone segments (magenta) in the foreground and fill `DebugReport`. |
| `DebugReport` | `FString` | | Per bone: angle between the posed bone segment and the proxy segment (degrees; 0 by construction) and the distance of the bone origin to its keypoint (cm). |
| `MinBoundsScale` | `float` | `6` | A poseable mesh is culled by its rest-pose bounds; the driver raises the mesh's `BoundsScale` to at least this value and updates the bounds after every pose, so a hand posed far from rest never vanishes. 1 = leave the component alone. |
| `bHideWhenNotTracked` | `bool` | `true` | Hide the mesh (hidden in game, and parked far away) while the hand is not tracked. The mesh is made visible when its hand is tracked, so you can leave the component's **Visible** off in the level: an idle hand then never overlaps your character in the editor (the demo maps do this). |
| `bGuardInput` | `bool` | `true` | Input guard. Non-finite keypoints or a degenerate palm keep the last good pose for up to `HoldLastGoodSeconds` (then the mesh is hidden); every finger phalanx is clamped to `MinSegmentRatio` .. `MaxSegmentRatio` of the mesh's own segment at the fitted scale; a finger joint never bends more than `MaxJointBendDeg`; a finger that jumps more than `OutlierJumpCm` relative to the palm in one tick is held for up to 4 ticks; the fitted scale is smoothed. `GuardReport` counts what it did. Off = the raw proxy keypoints. |
| `MinSegmentRatio` / `MaxSegmentRatio` | `float` | `0.25` / `1.6` | Allowed length of a finger phalanx relative to the mesh's segment at the fitted scale (the synthesised palm segments are not clamped). The minimum is low on purpose: webcam depth is flattened, so a curled finger seen end-on is legitimately short. |
| `MaxJointBendDeg` | `float` | `150` | Largest bend at a finger joint: only a finger folded back onto itself is corrected (a curled webcam fist has 100-130 deg at single joints). |
| `OutlierJumpCm` | `float` | `6` | One-tick jump of a finger relative to the palm that counts as a tracker glitch. |
| `HoldLastGoodSeconds` | `float` | `0.5` | How long an unusable frame keeps the last good pose. |
| `GuardReport` | `FString` | | Read only: frames, frames with clamped segments / bends, outlier fingers held, bad frames held. |
| `bAnatomicalJointLimits` | `bool` | `true` | every finger bone stays inside the rig's `JointLimits` (no sideways bend at the middle / end joints, limited hyperextension, thumb cones). `LastJointClamps` counts the bones clamped in the last frame. |
| `MinBoneScale` / `MaxBoneScale` | `float` | `0.8` / `1.25` | range of the axial bone scale (posed bone length / the mesh's bone at the fitted size). A proxy segment outside it is not followed further; the next bone starts at the end of the clamped one. |
| `TipLengthRatio` | `float` | `0.9` | Length of the fingertip beyond the last finger bone (to the tip keypoint) as a fraction of the preceding bone (a skeleton has no tip bone). |
| `bFitScaleToFingers` | `bool` | `true` | Size the mesh by the palm and finger chain lengths of the proxy hand (four fingers); off = wrist to middle knuckle only (before r5). |
| `bShortenForearmTowardCamera` | `bool` | `true` | Materials of the mesh with the scalar parameter `SoftPokeForearmFade` get 0.3 while the forearm points into the camera's half space and 1 when it points away (75 -> 105 deg). The driver also sets the vector parameters `SoftPokeHandBasisX` / `SoftPokeHandBasisY` (the hand's rotation from rest) every frame, which a material needs to turn a rest-space offset into world space. The sample's `M_SoftPokeVisibleHand` uses both to fold the forearm into a short closed end. |
| `VisibilityMode` | `ESoftPokeHandVisibility` | `AutoFade` | `Opaque`, `AutoFade` (see-through while close to the camera or between the camera and the contact) or `Ghost` (always see-through). Key **H** of `SoftPokeTrackerLink` cycles every hand in the world. |
| `VisibleHandMeshOverrideRight` / `VisibleHandMeshOverrideLeft` | `TSoftObjectPtr<USkeletalMesh>` | empty | show this skeletal mesh (UE mannequin / MetaHuman hand bone names) instead of the PoseableMesh's own hand. At Begin Play the driver swaps in its own `UPoseableMeshComponent` with that mesh and takes over the old component's settings. A mesh with no skeleton, no wrist / middle finger bone or fewer than 10 bones is refused with a `LogSoftPoke` warning and the bundled hand stays (`Status`). `ActiveOverrideMesh` shows what is in use. |
| `bAutoDetectVrTemplateHands` | `bool` | `false` | With no override set, use the VR Template's `/Game/Characters/MannequinsXR/Meshes/SKM_MannyXR_right` / `_left` (or `/Game/XRMannequins/Meshes/...` in a migrated copy) when they exist. Epic's meshes are never shipped with SoftPoke; they read as a white glove with panel seams, so this is opt-in. |
| `bUseSourceMeshMaterials` / `SkinMaterialForOverride` | `bool` / material | `false` / empty | An override mesh is drawn with `SkinMaterialForOverride` (empty = the sample's `MI_VisibleHand_Generic` when present, else the mesh's own materials). The material needs the parameters of `M_SoftPokeVisibleHand` for the fade (`SoftPokeForearmFade`, `HandOpacity` on the ghost variant). |
| `bOcclusionFade` | `bool` | `true` | `AutoFade` only: fade while the hand is closer to the camera than `FadeStartDistanceCm`, or while it lies on the line from the camera to the fingertip in contact and its forearm points toward the camera (closer than 35 .. 60 deg; a grab seen from the side stays opaque). Smooth over about 0.15 s. |
| `FadeStartDistanceCm` | `float` | `25` | Distance from the camera at which the hand starts to fade (fully faded at half of it). |
| `MinOpacity` | `float` | `0.25` | Opacity of the faded hand (a subtle fresnel outline). The finger in contact keeps `ContactFingerOpacity`. |
| `ContactFingerOpacity` | `float` | `0.85` | Opacity within 3 cm of the fingertip in contact while faded. |
| `OcclusionFadeMaterial` | `UMaterialInterface*` | none | Translucent material used while faded (the opaque materials come back at full opacity, so an unfaded hand has no translucency sorting). Each slot gets a dynamic instance with the parameters of the slot's material instance, plus `HandOpacity`, `ContactOpacity`, `ContactRadiusCm`, `ContactPos`. While faded the driver adds an invisible, opaque copy of the hand (a second `UPoseableMeshComponent`: same pose, the opaque slot materials, not drawn in the main or the depth pass, custom depth on) and the translucent material compares its pixel depth with that custom depth (`FrontToleranceCm`), so only the front-most layer of the hand is drawn; the project's *Custom Depth-Stencil Pass* must not be *Disabled* for this (the default is *Enabled*). The driver also sets `StubFade` (0..1, the sample material fades the wrist stub with it; in `AutoFade` it follows how close the forearm points at the camera, 60 .. 35 deg, in `Ghost` it is 1, in `Opaque` there is no translucent material) and the fade switches on for the stub alone, with the hand fully opaque. `SoftPokeHandBasisX.w` carries the hand's world scale. Empty = no fade. |
| `CurrentOpacity` | `float` | | Read only. |
| `PoseReport`, `LastMinBoneScale`, `LastMaxBoneScale`, `LastMaxJointGapCm`, `LastForearmViewAngleDeg`, `NumRejectedPoses` | | | Read only: the pose check of the last frame and the number of rejected poses. |

Bones of the mesh above the wrist that are not in the bone map (the root / forearm bone of a hand-only mesh such as `SKM_NatHand2_*`) always follow the wrist rigidly, so the forearm turns with the hand.
| `UpdatePose()` | function | | Pose now; returns false and sets `Status` when the hand is not tracked or the setup is incomplete. |
| `Status` | `FString` | | Also says "input hand looks MIRRORED" when the thumb of the incoming hand lies on the other side of the palm plane than the mesh's thumb (a tracker that sends the other hand or a mirrored webcam image): the fingers then curl backwards and the thumb breaks. Check the source's handedness / mirror setting. |

Known limitation: when the palm is turned sideways (supination) the posed mesh can bend backwards at the wrist or fingers, particularly with webcam input (its depth is flattened); the deformation is not affected because it uses the proxy hand, not the mesh. Per-bone scaling and an automatic mirror correction were tried and removed: they were unstable under contact.

The VR template's hand meshes (`SKM_QuinnXR_left/right`, `SKM_MannyXR_left/right` in `Content/XRMannequins/Meshes`) use the default bone names. They belong to the VR template project; SoftPoke does not ship them.

## SoftPokeHairCollision

Opt-in (`bEnabled` is false by default; nothing is created or registered). Optional feature: needs the engine's HairStrands plugin in your project (SoftPoke does not depend on it; the Groom is reached by reflection). Lets the contact-resolved hand push strand hair: builds a runtime Physics Asset of finger and palm capsules (radii of the SoftPoke contact model times `HairRadiusScale`) on an invisible `USkeletalMeshComponent` that takes its pose from the hand mesh of a `USoftPokeHandMeshDriver`, and registers it with the target Groom components through `UGroomComponent::AddCollisionComponent`. Optional head / neck / shoulder colliders keep the hair from being pushed into the body. See [Hair comb](HairComb.md). Optional hair LOD manager for VR (`bManageHairLOD`, off by default): forces the Groom's LOD 0 (strands) while a hand is within `HairLodNearCm` of the head and releases it with hysteresis (`HairLodFarCm`, `HairLodReleaseDelay`), so the Groom's own cards / reduced-strand LODs apply the rest of the time; reached through the engine's `ILODSyncInterface`, no HairStrands dependency. See [Hair LOD for VR](HairComb.md#hair-lod-for-vr-optional-off-by-default).

| Property | Type | Default | Description |
|---|---|---|---|
| `bEnabled` | `bool` | `false` | Build and register the collision at BeginPlay. |
| `HandDrivers` | `TArray<USoftPokeHandMeshDriver*>` | empty | One driver per hand (empty = every driver on the owner). |
| `TargetActors` | `TArray<AActor*>` | empty | Actors whose Groom components receive the collision (empty = the owner and its attached actors). |
| `bIncludeAttachedActors` | `bool` | `true` | Also search actors attached to each target actor. |
| `BodyActor` | `AActor*` | none | Character for `BodyColliders` (empty = the skeletal mesh the first Groom is attached to). |
| `BodyColliders` | `TArray<FSoftPokeHairBodyCollider>` | empty | Sphere (Start = End) or capsule: `Bone`, `Start`, `End` (component space of the rest pose, cm), `RadiusCm`. |
| `HairRadiusScale` | `float` | `1` | Multiplier of the contact radii. Fixed when the simulation starts. |
| `PalmStationStride` | `int32` | `3` | Use every Nth of the 23 palm capsules. The Groom solver accepts 100 primitives in total. |
| `DistalLengthFraction` | `float` | `0.8` | Fingertip segment length as a fraction of the intermediate segment. |
| `bKeepPoseUpdating` | `bool` | `true` | Keep the hand mesh posing when hidden or off screen and refresh its bones right after the driver. |
| `CapsuleJsonPath` | `FString` | empty | If set, the reference capsule radii (cm) are written here at BeginPlay. |
| `bDrawDebug` | `bool` | `false` | Draw the hair capsules and log a few positions. |
| `bStabilizeCollider` | `bool` | `true` | Drive the hair collider from a separate, rate-limited copy of the hand pose (low-pass, speed limit, glitch rejection, tracking-loss retreat). Off = the collider is attached rigidly to the hand mesh. See [Collider stabiliser](HairComb.md#collider-stabiliser-no-thrashing-of-combed-hair). |
| `ColliderSmoothingSeconds` | `float` | `0.03` | Low-pass time constant of the collider wrist, `a = 1 - exp(-dt / tau)`. |
| `ColliderMaxSpeedCmPerSec` | `float` | `100` | Speed limit of the collider; per frame the move is clamped to `MaxSpeed * dt`, the same step for all capsules of the hand. |
| `GlitchRejectCm` | `float` | `5` | A wrist sample that jumps farther than this in one frame from the last accepted one is ignored ... |
| `GlitchRejectMaxFrames` | `int32` | `3` | ... for at most this many consecutive frames; the next sample is believed. |
| `LossHoldSeconds` | `float` | `0.1` | Tracking lost: the collider holds its position this long ... |
| `LossRetreatCm` | `float` | `30` | ... then retreats this far away from the head at the speed limit (never a teleport). |
| `ReacquireHoldOffSeconds` | `float` | `0.2` | After tracking returns the collider stays retreated this long, then approaches at the speed limit. |
| `bAddHeadSphere` | `bool` | `true` | Add a sphere inside the skull when `BodyColliders` has none on the head bone (`HairLodHeadBone`); skipped when the 100-primitive budget is used up. |
| `HeadSphereRadiusCm` | `float` | `7` | Radius of that sphere. |
| `HeadSphereOffsetCm` | `FVector` | `(0, 0, 8)` | Centre of that sphere relative to the head bone (rest pose, character component space, cm). |
| `ColliderState`, `ColliderPeakSpeedCmPerSec`, `RawPeakSpeedCmPerSec`, `GlitchesRejected` | read only | | State of the stabiliser per hand, peak speed (cm/s) of the collider wrist and of the raw hand wrist, and the number of rejected samples since `ResetColliderStats()`. |
| `ResetColliderStats()` | function | | Zero the peak speeds and the glitch counter. |
| `Setup()` | function | | Build and register now (also done at BeginPlay and retried once a second until a Groom is found). Returns the number of Groom components registered. |
| `ExportCapsuleJson(Path)` | function | | Write the capsule radii as JSON (cm). |
| `GetNumCapsules()` | function | | Capsules of all Physics Assets built. |
| `Status` | `FString` | | What was found / built. |

Editor module helper (Python: `unreal.SoftPokeHairEditorLibrary`): `SetGroomSimulation(GroomAsset, bEnable, bSolveCollision, bProjectCollision, CollisionRadiusCm, SubSteps, IterationCount)` (negative = keep; returns an error text or empty), `DescribeGroomSimulation(GroomAsset)`, `AddInstanceComponent(Actor, ComponentClass, Name)` (adds a component to a level actor so that it is saved with the level).

## SoftPokeTrackerLink

Opt-in (`bEnabled` is false by default: nothing is sent, no key is read, no process is started). The link between the game and the webcam tracker program (`Tools/Webcam/tracker.py`) for desktop use. It sends the view pose (the camera of player 0, or `ViewComponent`) as UDP text `x y z yaw` to the tracker's `--follow-port`, so the tracked hand workspace follows the view; sends `recenter` on request; reads three keys from the first player controller (only while the game window has the focus; Enhanced Input is not needed); and can start the tracker when the game starts and stop it when the game ends. No editor, no Python inside Unreal, messages go to `127.0.0.1` by default. Recording takes is a tracker feature (SPACE in the preview window). See [Webcam tracker guide](WebcamTracker.md#following-the-game-view-recenter-and-the-softpoketrackerlink-component). **Built-in tracker:** when a `SoftPokeHandRig` in the level uses `SoftPokeHandSourceNative`, the view pose and Recenter go to that source directly (no UDP, no process); see [Built-in webcam tracker](NativeTracking.md).

| Property | Type | Default | Description |
|---|---|---|---|
| `bEnabled` | `bool` | `false` | Master switch. |
| `bSendViewPose` | `bool` | `true` | Send the view pose every tick. |
| `FollowHost` | `FString` | `127.0.0.1` | Address of the tracker's follow port. |
| `FollowPort` | `int32` | `9799` | The tracker's `--follow-port`. |
| `ViewComponent` | `USceneComponent*` | none | The view (for example a camera). Empty = the camera of player 0. |
| `bEnableKeys` | `bool` | `true` | Read the keys below. |
| `RecenterKey` | `FKey` | `C` | Recenter: hold your hand in the neutral spot, then press. |
| `DebugDrawKey` | `FKey` | `G` | Toggle `bDrawDebug` of every `SoftPokeHandRig` and `SoftPokeHandMeshDriver` in the world. |
| `PreviewKey` | `FKey` | `T` | Toggle the tracker's preview window (restarts a tracker that this component launched; ignored for one you started yourself). |
| `HandVisibilityKey` | `FKey` | `H` | cycle every `SoftPokeHandMeshDriver` in the world through opaque / auto-fade / ghost (`CycleHandVisibility()`). |
| `bLaunchTracker` | `bool` | `false` | Start `tracker.py` at BeginPlay and stop it at EndPlay. Do not also start a tracker yourself. |
| `TrackerPython` | `FString` | empty | Full path of the `python.exe` of the tracker's virtual environment. |
| `TrackerFolder` | `FString` | empty | Folder with `tracker.py` (working directory of the process). |
| `TrackerArguments` | `FString` | `--camera 0 --width 1280 --height 720 --fps 30` | Options after `tracker.py`. `--follow-port` and `--preview` are added from the settings here. |
| `bTrackerPreview` | `bool` | `true` | Add `--preview`. |
| `RestartDelaySeconds` | `float` | `5` | A tracker that exits with an error is restarted after this delay; one that exits normally (Esc Esc) stays off until the next game start. 0 = never. |
| `bDebugDrawOn`, `Status` | read only | | Current debug-draw state; what the component is doing or why not. |
| `Recenter()` | function | | Send `recenter`. Returns false when the component is not running. |
| `ToggleDebugDraw()`, `SetDebugDraw(bOn)` | function | | The debug-draw toggle. |
| `StartTracker()`, `StopTracker()`, `SetTrackerPreview(bShow)`, `IsTrackerRunning()` | function | | The launched tracker process. |

## SoftPokeDesktopFly

Opt-in (`bEnabled` is false by default). A desktop "VR viewpoint" for testing without a headset: moves and turns an actor (`TargetActor`, default the pawn of player 0) like a free-fly camera, and sets the player's control rotation. Use it with `SoftPokeTrackerLink`. Do not combine it with a pawn that already moves with the same keys, and do not use it with a headset (the HMD sets the view). Keys are read from the first player controller, so they work only while the game window has the focus.

| Property | Type | Default | Description |
|---|---|---|---|
| `bEnabled` | `bool` | `false` | Master switch. |
| `TargetActor` | `AActor*` | none | The actor that moves. Empty = the pawn of player 0 (or the owner). |
| `SpeedCmPerSec` | `float` | `60` | Speed. |
| `FastMultiplier` | `float` | `3` | Speed factor while `FastKey` is held. |
| `TurnSpeedDeg`, `PitchSpeedDeg` | `float` | `90`, `40` | Arrow-key turn and look speed (deg/s). |
| `MouseSensitivity` | `float` | `0.2` | Degrees per mouse count while `MouseLookKey` is held. |
| `MaxPitchDeg` | `float` | `80` | Largest look up / down angle. |
| `ForwardKey`, `BackKey`, `LeftKey`, `RightKey`, `UpKey`, `DownKey` | `FKey` | `W S A D E Q` | Move (forward / back / left / right on the ground plane in the viewing direction, up / down). |
| `FastKey` | `FKey` | `LeftShift` | Fast movement. |
| `TurnLeftKey`, `TurnRightKey`, `LookUpKey`, `LookDownKey` | `FKey` | `Left Right Up Down` | Turn and look. |
| `MouseLookKey` | `FKey` | `RightMouseButton` | Hold and move the mouse to look around. |

## SoftPokeDemoGameMode and SoftPokeDemoVRPawn

Game mode and pawn of the demo maps (see [VR](VR.md#demo-vr-pawn)). `SoftPokeDemoGameMode` (Blueprint `BP_SoftPokeDemoGameMode`): `DesktopPawnClass` (empty = Default Pawn Class), `VRPawnClass` (`BP_SoftPokeDemoVRPawn`), `FocusLocation` (0,0,0), `VRReachDistanceCm` (65), `bPlaceVRStart` (true), `FloorZFallback` (0); static `ShouldUseVRPawn()`; console variable `softpoke.Demo.VR` (-1 automatic, 0 desktop, 1 VR). `SoftPokeDemoVRPawn`: components `VROrigin`, `Camera`, `LeftController`, `RightController`; `bFloorTrackingOrigin` (true), `bThumbstickMove` (true), `MoveSpeedCmPerSec` (110), `MoveDeadzone` (0.2), `bSnapTurn` (true), `SnapTurnDegrees` (30), `SnapTurnThreshold` (0.7), `SnapTurnRearm` (0.3), `NoHmdEyeHeightCm` (160), `bDisableDesktopFlyInVR` (true); `SnapTurnBy(deg)`, `GetStatusText()`, `SetInputOverride(move right, move forward, turn)` (test hook).

## SoftPokeTouchSensor

`USoftPokeTouchSensor` (class group Rendering, spawnable in Blueprints, **no Blueprint events or functions**). Add it to the character next to the `SoftPokeHandRig`. It turns the contacts of the hand rig into touch events (where, how hard, how long, what kind of touch) and gives them to the WebSocket / webhook transports that are switched on in the project settings (both off by default). Everything about the events, parts, gestures and security is on the page [Touch events](TouchEvents.md). Without the component, or with `softpoke.Touch.Enable 0`, the plugin does no touch work.

**Category `SoftPoke|Touch`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `EntityName` | `FString` | actor label |  | Name sent as `entity` in every event. |
| `UpdateHz` | `float` | `5.f` | 1.0 - 30.0 | Rate of `touch_update` per active touch. |
| `StateHz` | `float` | `1.f` | 0.0 - 10.0 | Rate of the `state` event; 0 = off. |
| `bIncludeText` | `bool` | `true` |  | Add the short English description (`text`) to every event. |
| `Tuning` | `FSoftPokeTouchTuning` |  |  | Thresholds: `StartDepthMm` 0.25, `KeepDepthMm` 0.08, `ReleaseHoldSec` 0.12, `MergeRadiusCm` 3, `PartSwitchDwellSec` 0.25, `GestureDwellSec` 0.12, `PressMinSec` 0.3, `StrokeEnterCmPerSec` 2, `StrokeExitCmPerSec` 1, `OpposingDot` -0.2, `SqueezeIntensity` 0.5, `KneadAmplitude` 0.12, `TapGapSec` 0.7. |
| `ExcludedParts` | `TArray<FName>` | empty |  | Part keys that never produce events (for example `nose`). |
| `BonePartOverrides` | `TMap<FName, FName>` | empty |  | Skeleton bone name to part key; replaces the rule for that bone and its child bones. |
| `RigActor` | `AActor` | this actor |  | Actor with the `SoftPokeHandRig`; empty = this actor, else the rig whose Body Actor is this actor. |
| `bLogEvents` | `bool` | `false` |  | Log every event at Display level. |

## Hand sources

Subclasses of `USoftPokeHandSource` (abstract; `Start`, `Stop`, `Poll`, `GetHand`, `GetHead`, `Describe` - see [API and extension](ApiAndExtension.md)). Select one with the rig's `SourceClass`.

### `USoftPokeHandSourceAuto` (default, experimental)

Source mode **Auto**: per hand, OpenXR hand tracking if it is valid, else the hand's motion controller, else the fallback source. See [VR](VR.md#which-input-do-you-get).

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bUseOpenXRHands` | `bool` | `true` |  | Allow OpenXR hand tracking (also the finger-curl skeleton of Valve Index controllers through SteamVR). |
| `bUseControllers` | `bool` | `true` |  | Allow the motion-controller source. |
| `FallbackSourceClass` | `TSubclassOf<USoftPokeHandSource>` | empty |  | Third choice (webcam / video / UDP). Empty = the rig's `AutoFallbackClass`; both empty = no fallback. |
| `SwitchDebounceSeconds` | `float` | `0.25` | 0 .. 5 | A better source must be tracked this long before the hand switches to it. When the current source is lost the next one takes over at once. |
| `FallbackStartDelaySeconds` | `float` | `1.0` | 0 .. 30 | The fallback is started when no OpenXR hand and no controller was tracked for this long. 0 = start at once. |
| `OpenXR` | `USoftPokeHandSourceOpenXR` |  |  | Read-only sub-object. |
| `Controller` | `USoftPokeHandSourceController` |  |  | Read-only sub-object: its properties are those of the next section (open the `Controller` sub-object in Details). |

Functions: `GetActiveSource(Side)` (`None`, `OpenXRHands`, `Controller`, `Fallback`), `GetFallbackSource()`.

### `USoftPokeHandSourceController` (experimental, checked on one headset setup)

Synthesizes the hand from the grip pose of a motion controller and its trigger / grip / thumb-touch values (trigger = index curl, grip = middle / ring / little curl, trigger released with the grip held = pointing index, thumb touch = thumb curl). See [VR](VR.md#motion-controllers) and [Behaviour and limits](BehaviourAndLimits.md#motion-controllers-synthesized-hand).

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bUseEnhancedInput` | `bool` | `true` |  | Read the inputs through Enhanced Input actions (adds the mapping contexts to the first local player). Off = raw key states. |
| `MappingContexts` | `TArray<UInputMappingContext*>` | empty |  | Contexts added at Start. Empty = the shipped `IMC_SoftPoke_ValveIndex`, `_OculusTouch`, `_Vive`, `_WMR` (plugin content `/SoftPoke/Input`). |
| `MappingPriority` | `int32` | `0` |  | Priority of the added contexts. |
| `TriggerAction[2]`, `GripAction[2]`, `ThumbTouchAction[2]` | `UInputAction*` | empty |  | Axis 1D actions, [0] left, [1] right. Empty = the shipped `IA_SoftPoke_Trigger_L / _R`, `IA_SoftPoke_Grip_L / _R`, `IA_SoftPoke_ThumbTouch_L / _R`. |
| `bFallbackToRawKeys` | `bool` | `true` |  | When the Enhanced Input actions are not available, read the same keys from the player controller. |
| `bAcceptInertialOnly` | `bool` | `false` |  | Also use controllers whose position is lost (orientation only). |
| `HandScale` | `float` | `1` | 0.5 .. 1.5 | Size of the virtual hand relative to an average adult hand (wrist to middle knuckle 8.6 cm). |
| `InputDeadzone` | `float` | `0.04` | 0 .. 0.5 | Analog values below this are 0. |
| `FingerCurlSpeed` | `float` | `8` | 1 .. 60 | Curl units per second at which the fingers follow the inputs. |
| `ThumbGripShare` | `float` | `0.15` | 0 .. 1 | Share of the thumb curl that comes from the grip. |
| `ProfileTweaks` | `TArray<FSoftPokeControllerProfileTweak>` | 4 entries, all zero |  | Per OpenXR interaction profile (`ProfileContains` = part of the profile path, case-insensitive): `PalmOffsetCm` (grip frame: x forward, y right, z up) and `RotationOffset` of the virtual hand relative to the grip pose. The y offset and the yaw / roll are mirrored for the left hand. Not calibrated on a headset. |
| `GlobalTweak` | `FSoftPokeControllerProfileTweak` | zero |  | Applied after the profile entry to every controller. |
| `bRigFilter` | `bool` | `false` |  | Controller data is clean: the rig's One-Euro filter is skipped for this source. On = filter it anyway. |
| `bHapticsOnContact` | `bool` | `true` |  | Short, weak haptic pulse when the hand touches the skin and when it presses clearly deeper. |
| `HapticBaseAmplitude`, `HapticDepthAmplitude`, `HapticMaxAmplitude` | `float` | `0.08`, `0.25`, `0.35` | 0 .. 1 | Pulse amplitude = base + depth term x (press depth / allowed depth), capped by the maximum. |
| `HapticPulseSeconds`, `HapticFrequency` | `float` | `0.06`, `0.5` | 0.01 .. 0.5; 0 .. 1 | Pulse length (s) and the normalised frequency passed to `SetHapticsByValue`. |

Functions: `GetInputs(Side)` (smoothed trigger, grip, thumb touch as a vector), `GetInteractionProfile(Side)`, `IsHandTracked(Side)`. In editor builds only: `SetTestInput(Side, GripWorld, Trigger, Grip, ThumbTouch)` replaces a controller (pose and input values) by an injected one, `SetTestPose(Side, GripWorld)` injects only the pose (the inputs are read through Enhanced Input / raw keys as usual), `ClearTestInput(Side)` returns to the real controller (test hooks used by the BuyerSim harness; not compiled into a game).

### `USoftPokeHandSourceOpenXR` (experimental)

Hand tracking through the engine's `IHandTracker` modular feature (the engine plugins OpenXR and OpenXR Hand Tracking, `XR_EXT_hand_tracking`). With SteamVR this includes the finger-curl skeleton of Valve Index controllers. See [VR (experimental)](VR.md). `GetInteractionProfile(Side)` returns the controller profile the engine reports.

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bAcceptUntracked` | `bool` | `false` |  | OpenXR: accept the last known joints while the runtime reports "not tracked" or its hand tracking state is not valid. Off = the hand is lost immediately. |
| `ReferenceJointRadiusCm` | `float` | `0.75` | 0.0 .. | mean finger joint radius (cm) of the reference hand that the runtime's `HandKeyRadii` are compared with. The ratio scales the visible hand within 0.85 .. 1.15 of its fitted size; 0 = ignore the radii. See [Behaviour and limits](BehaviourAndLimits.md). |

### `USoftPokeHandSourceUdp`

Receives the webcam tracker's datagrams ([UDP protocol v1](ApiAndExtension.md#udp-protocol-version-1)).

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `Port` | `int32` | `9763` |  | UDP port (default 9763). |
| `BindAddress` | `FString` |  |  | Local address to listen on. Empty = Project Settings > Plugins > SoftPoke > `UdpBindAddress` (default 127.0.0.1, this PC only). |
| `TimeoutSeconds` | `float` | `0.25` | >= 0.02 | A hand or head is lost when no datagram arrived for this long (s). |
| `MinConfidence` | `float` | `0.3` | 0.0 .. 1.0 | Packets with a lower confidence are ignored. |
| `SpaceToWorld` | `FTransform` | `FTransform::Identity` |  | Where the tracker space `mp_world_m` (hand-centred metres) sits in the world; `ue_world_cm` packets are not transformed. |
| `bMirrorLateral` | `bool` | `false` |  | Mirror the lateral axis of `mp_world_m` packets (for selfie-flipped images). |
| `PacketsReceived` | `int32` | `0` |  | Datagrams accepted (read only). |
| `PacketsRejected` | `int32` | `0` |  | Datagrams rejected (read only). |
| `HeadPacketsReceived` | `int32` | `0` |  | Datagrams that carried a valid `head` (read only). |

**Functions**

| Function | Signature | Description |
|---|---|---|
| `GetHeadFrame` | `bool GetHeadFrame(FSoftPokeHeadFrame& Out)` | Latest head frame; false when none arrived within `TimeoutSeconds`. (BlueprintCallable) |
| `GetBlendshapes` | `bool GetBlendshapes(TMap<FName, float>& OutBlendshapes)` | Latest 51 blendshapes as ARKit name -> value (the subject's left / right). (BlueprintCallable) |
| `GetBlendshape` | `float GetBlendshape(FName Name)` | One blendshape by ARKit name (for example `jawOpen`); 0 if unknown. (BlueprintCallable) |

Static helpers (C++ only): `GetBlendshapeNames()` returns the 51 blendshape names in packet order; `ParsePacket(...)` decodes one datagram.

### `USoftPokeHandSourceNative` (recommended webcam input)

Module `SoftPokeNativeTracking` (Win64; needs the engine plugin NNERuntimeORT). Built-in webcam hand tracker: Media Foundation capture, MediaPipe hand models through NNE, PnP depth, workspace mapping. Guide: [Built-in webcam tracker](NativeTracking.md).

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `CameraName` | `FString` |  |  | The first camera whose name contains this text. Empty = `CameraIndex`. |
| `CameraIndex` | `int32` | `0` | >= 0 | Camera by index (see `ListCameras`). |
| `Width` / `Height` / `Fps` | `int32` / `int32` / `float` | `1280` / `720` / `30` |  | Requested format; the closest the camera offers is used. |
| `PreferredFormat` | `ESoftPokeCameraFormat` | `Auto` |  | Auto = MJPG, then NV12, then YUY2. |
| `bMirrored` | `bool` | `false` |  | The camera image is mirrored (selfie view). |
| `HorizontalFovDeg` | `float` | `70` | 20 .. 150 | Horizontal field of view, used when `Intrinsics.CalibWidth` is 0. |
| `Intrinsics` | `FSoftPokeCameraIntrinsics` |  |  | fx fy cx cy at CalibWidth x CalibHeight (scaled to the delivered size). |
| `VideoFile` | `FFilePath` |  |  | Play a video file instead of the camera (tests, no webcam). |
| `bLoopVideo` | `bool` | `true` |  | Loop the video file. |
| `HandSizeCm` | `float` | `0` | 0 .. 15 | Wrist crease to middle-finger base (cm). 0 = MediaPipe's average hand. |
| `Workspace` | `FSoftPokeHandWorkspace` |  |  | `AnchorCm`, `NeutralCamM`, `Gain`, `YawDeg`, `bFirstPerson`, `CameraPitchDeg`, `bUseViewFromCam` / `ViewFromCam` (same law as the Python tracker's workspace.json). |
| `FollowForwardCm` / `FollowDownCm` | `float` | `40` / `25` |  | Follow mode: neutral in front of / below the view. |
| `ImportProfileJson` / `ImportWorkspaceJson` | `FFilePath` |  |  | Read the Python tracker's profile.json / workspace.json at start. |
| `bRememberCalibration` | `bool` | `true` |  | Store neutral and hand size in `Saved/SoftPoke/NativeTracking.json`. |
| `Backend` | `ESoftPokeNativeBackend` | `Cpu` |  | `Cpu` or `DirectML`. |
| `MaxHands` | `int32` | `2` | 1 .. 2 | Hands tracked. |
| `DetectIntervalFrames` | `int32` | `8` | 1 .. 60 | Palm detector interval while fewer than `MaxHands` are tracked (every frame while none is). |
| `MinDetectionConfidence` / `MinPresenceConfidence` | `float` | `0.5` / `0.5` |  | MediaPipe thresholds. |
| `MinConfidence` | `float` | `0.3` | 0 .. 1 | Hands with a lower confidence (handedness score x depth fit) are not reported. |
| `TranslationAlpha` | `float` | `1` | 0.05 .. 1 | EMA of the hand position (1 = off). |
| `TimeoutSeconds` | `float` | `0.25` | >= 0.02 | A hand is lost after this long without a detection. |
| `bPreview` / `PreviewWidth` | `bool` / `int32` | `false` / `640` |  | Preview texture with landmarks. |
| `Status` / `Stats` | `FString` / `FSoftPokeNativeStats` |  |  | Read only: format, fps, ms, latency. |

**Functions**

| Function | Signature | Description |
|---|---|---|
| `ListCameras` | `static TArray<FString> ListCameras()` | Capture devices with their native formats. (BlueprintCallable) |
| `Recenter` | `bool Recenter()` | Current hand position becomes the neutral position (right hand first; mean of 15 frames). (BlueprintCallable) |
| `CalibrateHandSize` | `float CalibrateHandSize(float DistanceCm)` | Hand size from an open hand at a known distance (median of 30 frames). (BlueprintCallable) |
| `SetFollowView` | `void SetFollowView(const FVector& ViewLocation, float ViewYawDeg)` | Follow-the-view workspace (also driven by `SoftPokeTrackerLink`). (BlueprintCallable) |
| `GetPreviewTexture` | `UTexture2D* GetPreviewTexture()` | Preview texture or null. (BlueprintCallable) |
| `IsRunning` | `bool IsRunning()` | Capture and tracker threads are running. (BlueprintPure) |

### Frames

`FSoftPokeHandFrame` (hand joints) and `FSoftPokeHeadFrame` (head pose and blendshapes), both Blueprint types:

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bTracked` | `bool` | `false` |  | The hand is tracked in this frame. |
| `Positions` | `TArray<FVector>` |  |  | Joint positions (21, world cm). |
| `Rotations` | `TArray<FQuat>` |  |  | Joint rotations (21, world) if the source provides them (OpenXR); empty for position-only sources (network). |
| `Confidence` | `float` | `1.f` |  | Tracker confidence 0..1. |
| `HandSizeRatio` | `float` | `0` |  | hand size relative to the reference hand from the source's joint radii (OpenXR); 0 = none. |

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `bTracked` | `bool` | `false` |  | The head is tracked in this frame. |
| `Position` | `FVector` | `FVector::ZeroVector` |  | Head origin, world cm. |
| `Rotation` | `FQuat` | `FQuat::Identity` |  | Head rotation, world. |
| `Confidence` | `float` | `1.f` |  | Tracker confidence 0..1. |
| `Scale` | `float` | `0.f` |  | Head size relative to the canonical face model (0 = not sent). |
| `Blendshapes` | `TArray<float>` |  |  | The 51 blendshape scores 0..1 in the order of `GetBlendshapeNames()`; empty if the packet had none. |

`ESoftPokeHandSide`: `Left`, `Right`.

## SoftPokeFaceDriver

`USoftPokeFaceDriver` (class group Rendering, spawnable; implements `ICurveSourceInterface`). Turns the tracker's 51 blendshapes into animation curves with ARKit names for a face animation blueprint (a Curve Source node bound to `CurveSourceBindingName`), and switches the under-chin exclusion of the face component from the jaw-open value. It does not drive any character itself: connecting the curves to a face rig happens in your project, with your own assets.

`ESoftPokeFaceMode`: `Tracker`, `Cycle`, `Static`, `Off`.

**Category `SoftPoke|Face`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `Mode` | `ESoftPokeFaceMode` | `Tracker` |  | Where the face values come from: `Tracker` (webcam blendshapes), `Cycle` (canned jaw / smile / brow loop), `Static` (values you set), `Off`. |
| `CurveSourceBindingName` | `FName` | `"SoftPokeFace"` |  | Name the face animation blueprint's Curve Source node binds to (its Source Binding). |
| `HandRig` | `USoftPokeHandRig` |  |  | Tracker mode: the hand rig whose UDP source supplies the blendshapes. Empty = the rig on this actor. |
| `RigActor` | `AActor` |  |  | Actor that holds the hand rig when it is not on this actor. |
| `FaceMesh` | `USkinnedMeshComponent` |  |  | The face skeletal mesh component (the one with the face animation blueprint). Empty = the target mesh of `FaceSoftPoke`. |
| `FaceMeshName` | `FName` |  |  | Name of the face mesh component on this actor (wins over `FaceMesh`). |
| `FaceSoftPokeName` | `FName` |  |  | Name of the face's SoftPoke component on this actor (wins over `FaceSoftPoke`). |
| `FaceSoftPoke` | `USoftPokeComponent` |  |  | The face's SoftPoke component (secondary). Empty = the first secondary SoftPoke component of this actor. |
| `bForceFaceLod0` | `bool` | `true` |  | Force LOD 0 on the face mesh while the driver runs (the face region is baked for LOD 0). |
| `Intensity` | `float` | `1` | 0.0 .. 1.0 | Global scale of all values (0..1), applied before the filter. |
| `bMirrorLeftRight` | `bool` | `false` |  | Swap left and right (a tracker image that is not selfie-mirrored). |
| `bFilter` | `bool` | `true` |  | One-Euro filter on every channel. |
| `MinCutoffHz` | `float` | `4` | >= 0.05 | One-Euro minimum cutoff (Hz). |
| `Beta` | `float` | `2` | >= 0.0 | One-Euro speed coefficient. |
| `ReleaseSeconds` | `float` | `0.25` | >= 0.01 | Seconds in which a lost tracker returns the face to neutral. |
| `bExcludeUnderChin` | `bool` | `true` |  | Enable the under-chin exclusion of `FaceSoftPoke` while the jaw is open. |
| `JawExcludeOn` | `float` | `0.30` | 0.0 .. 1.0 | jawOpen (filtered, 0..1) above which the under-chin fold is excluded. |
| `JawExcludeOff` | `float` | `0.20` | 0.0 .. 1.0 | jawOpen below which the fold is supported again (hysteresis). |

**Category `SoftPoke|Face|Debug`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `CyclePeriodSeconds` | `float` | `8` | >= 1.0 | Cycle mode: length of one loop (s). |
| `StaticValues` | `TMap<FName, float>` |  |  | Static mode: ARKit name -> value. |

**Category `SoftPoke|Face|Status`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `Status` | `FString` |  |  | The mode in effect, the filtered jawOpen and packets used (read only). |
| `JawOpenValue` | `float` | `0.f` |  | Filtered jaw-open value (read only). |
| `bUnderChinExcludedNow` | `bool` | `false` |  | True while the under-chin exclusion is on (read only). |

**Functions**

| Function | Signature | Description |
|---|---|---|
| `SetStaticValue` | `void SetStaticValue(FName ArkitName, float Value)` | Sets one Static value (and switches to Static mode). Name: ARKit name, for example `jawOpen`. (BlueprintCallable) |
| `ClearStaticValues` | `void ClearStaticValues()` | Clears the Static values (neutral). (BlueprintCallable) |
| `GetCurrentCurves` | `void GetCurrentCurves(TMap<FName, float>& OutCurves)` | The curve values currently supplied (ARKit name -> value), for Blueprint or debugging. (BlueprintCallable) |

## Project settings

Project Settings > Plugins > **SoftPoke** (`USoftPokeSettings`, stored in the project):

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `UdpBindAddress` | `FString` | `"127.0.0.1"` |  | Address the UDP hand source listens on when the source's own `BindAddress` is empty. `127.0.0.1` = this PC only. Use `0.0.0.0` only if the tracker runs on another machine. |
| `bTouchWebSocket` | `bool` | `false` |  | Touch events: start the WebSocket server (read only). See [Touch events](TouchEvents.md). |
| `TouchWebSocketBindAddress` | `FString` | `"127.0.0.1"` |  | Address the touch WebSocket listens on. Anything but `127.0.0.1` is reachable from the network (warning in the log). |
| `TouchWebSocketPort` | `int32` | `9764` | 1024 - 65535 | Port of the touch WebSocket. |
| `TouchAuthToken` | `FString` | empty |  | Optional shared secret of the WebSocket (`Authorization: Bearer` or `?token=`). WebSocket only. |
| `TouchWebhookAuthToken` | `FString` | empty |  | Optional secret sent as `Authorization: Bearer` with every webhook request (not the WebSocket token). |
| `TouchAllowedOrigins` | `TArray<FString>` | empty |  | Extra browser origins allowed to open the WebSocket (localhost pages always are). |
| `TouchMaxClients` | `int32` | `8` | 1 - 32 | Simultaneous WebSocket clients. |
| `bTouchWebhook` | `bool` | `false` |  | Touch events: send HTTP POSTs to `TouchWebhookUrls`. |
| `TouchWebhookUrls` | `TArray<FString>` | empty |  | Up to four `http://host:port/path` URLs; only this PC unless `bTouchAllowRemoteWebhook`. |
| `TouchWebhookEvents` | `TArray<FString>` | `start`, `end` |  | Events that are posted: `start`, `update`, `end`, `state`. |
| `bTouchAllowRemoteWebhook` | `bool` | `false` |  | Allow webhook URLs that are not on this PC. |
| `TouchWebhookTimeoutMs` | `int32` | `1500` | 200 - 10000 | Time limit of one webhook request. |

## Data assets and small types

Created by the import step; you normally do not edit them.

* `USoftPokeBodyAsset` (`SPB_<name>`): the list of region assets of one body.

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `Regions` | `TArray<TObjectPtr<USoftPokeBakeAsset>>` |  |  | The region assets of the body. |
| `BuildInfo` | `FString` |  |  | Import summary (read only). |
| `Fill` | `TSoftObjectPtr<USoftPokeFillAsset>` |  |  | Skin fill data of the body (read only). |

* `USoftPokeBakeAsset` (`SPK_<region>`): one baked region. Only these properties are meant to be edited; the rest is bookkeeping written by the importer (solver frame, mirror data, render-map statistics, payload reference).

**Category `Region`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `RegionName` | `FName` |  |  | Name of the region (set by the importer). |
| `AttachBone` | `FName` | `"thigh_l"` |  | Bone whose transform the region follows (set by the importer from the baked file). |

**Category `Solver`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `FrictionMu` | `float` | `0.5` |  | Friction coefficient baked with the matrix (used by the solver). 0 = frictionless. The importer copies the value of the .spk meta blob when it is >= 0 (a negative value there means "not set": 0.5). |
| `TipContact` | `ESoftPokeTipContact` | `Auto` |  | Fingertip contact mode of the region: `Auto` (on for hand and foot regions), `On`, `Off`. |

`ESoftPokeTipContact`: `Auto`, `On`, `Off`.

* `FSoftPokeCapsule` (Blueprint struct, one hand capsule):

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `A` | `FVector` | `FVector::ZeroVector` |  | First end point of the capsule axis (component space, cm). |
| `B` | `FVector` | `FVector::ZeroVector` |  | Second end point of the capsule axis (component space, cm). |
| `Radius` | `float` | `0.5` |  | Capsule radius (cm). |
| `FingerGroup` | `uint8` | `0` |  | 0 palm, 1 index, 2 middle, 3 ring, 4 pinky, 5 thumb. |

## Editor Python and Blueprint library (`USoftPokeBakeLibrary`)

Editor module (not in packaged games). A thin layer under the Bake panel: whatever the panel does, these calls do. In Python: `unreal.SoftPokeBakeLibrary.<snake_case_name>`; out-parameters (`OutError`, `OutMessage`) come back as the return value. See [Bake Your Own Character](BakeYourOwnCharacter.md) for the workflow and parameter keys.

| Function | Signature | Description |
|---|---|---|
| `ExportSkeletalMeshToSPRaw` | `static bool ExportSkeletalMeshToSPRaw(USkeletalMesh* SkeletalMesh, const FString& OutputFile, FString& OutMessage, int32 LodIndex = 0, bool bPreferMeshDescriptionWeights = true, FName SkinWeightProfile = NAME_None)` | Writes the bake input (`SPRAW1`) from a skeletal mesh LOD. Python returns the statistics text, or `None` on failure (the error is in `LogSoftPoke`). (BlueprintCallable) |
| `SetBakeParam` | `static bool SetBakeParam(const FString& Key, const FString& Value, FString& OutError)` | Sets one bake parameter (key list in "Bake Your Own Character"). Returns false + error text for an unknown key or bad value. (BlueprintCallable) |
| `StartPythonCheck` | `static bool StartPythonCheck(FString& OutError)` | Verifies the configured Python (short non-blocking process; read the result with `GetBakeStatusJson`). (BlueprintCallable) |
| `StartSetup` | `static bool StartSetup(FString& OutError)` | The **Set up** button: creates the bake venv and installs the pinned packages with pip. Non-blocking. (BlueprintCallable) |
| `StartBake` | `static bool StartBake(FString& OutError)` | Starts the whole job: export, bake (child process), compact files, import, assign. Non-blocking. (BlueprintCallable) |
| `CancelBake` | `static void CancelBake()` | Stops the running bake process (only its own children). (BlueprintCallable) |
| `IsBakeRunning` | `static bool IsBakeRunning()` | True while a bake or setup job runs. (BlueprintCallable, BlueprintPure) |
| `GetBakeStatusJson` | `static FString GetBakeStatusJson(int32 LogTail = 40)` | JSON status: state, status text, busy flag, progress, output folder, body asset, quality gate items, setup result, log tail. (BlueprintCallable, BlueprintPure) |
| `ImportBakedBody` | `static bool ImportBakedBody(const FString& SpkFolder, const FString& PackageDir, USkeletalMesh* DisplayMesh, const FString& BodyName, FString& OutMessage, FString& OutBodyAssetPath)` | Imports a folder of compact `.spk` files as a body asset on a display mesh (step 4 alone). (BlueprintCallable) |
| `AssignBodyToActor` | `static bool AssignBodyToActor(const FString& ActorLabel, const FString& BodyAssetPath, const FString& DeformerPath, FString& OutMessage)` | Adds or finds the `SoftPokeComponent` of a level actor and sets its body asset and deformer (step 5 alone). Marks packages dirty; saves nothing. (BlueprintCallable) |
| `PrepareMetaHumanActor` | `static bool PrepareMetaHumanActor(const FString& ActorLabel, FString& OutMessage)` | One call for a MetaHuman (or any MetaHuman-skeleton character) placed in the level: finds the body mesh (the component named `Body`, else one with the bones `pelvis`, `thigh_l`, `hand_r`), adds a `SoftPokeComponent` targeting it and a `SoftPokeHandRig` with the UDP source when missing, and fills the bake parameters `source`, `actor`, `bonemap=auto`. Bakes nothing. Python: `msg = unreal.SoftPokeBakeLibrary.prepare_meta_human_actor('<label>')` (the message, or `None` on failure; the failure text is in the Output Log). |

`ESoftPokeBakeState` (the `state` field of the status JSON): `Idle`, `Checking`, `SettingUp`, `Exporting`, `Baking`, `WritingSpk`, `Importing`, `Assigning`, `Done`, `Blocked`, `Failed`, `Cancelled`. `ESoftPokeBakeQuality` (`quality` parameter, lower case): `Draft`, `NoJiggle`, `Standard`.

## Bake tool settings (`USoftPokeBakeSettings`)

Project Settings > Plugins > **SoftPoke Bake Tool**. Stored per user (not in the project), because the paths are machine specific.

**Category `Python`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `PythonExecutable` | `FFilePath` |  |  | python.exe used for baking. Empty = environment variable `SOFTPOKE_PYTHON`, then `python` on PATH. **Set up** writes the venv's python.exe here. |
| `SetupPythonExecutable` | `FFilePath` |  |  | python.exe (3.12, 64-bit, from python.org) that **Set up** builds the venv from. Empty = environment variable `SOFTPOKE_BASE_PYTHON`, then the first python.exe on PATH. |
| `BakeToolDirectory` | `FDirectoryPath` |  |  | Folder with the bake scripts. Empty = `<plugin>/Tools/Bake`. |

**Category `Bake`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `WorkDirectory` | `FDirectoryPath` |  |  | Parent folder of the per-bake work folders (default `<Project>/Saved/SoftPokeTools/Bakes`; 1-3 GB per body). |

**Category `Import`**

| Property | Type | Default | Range | Description |
|---|---|---|---|---|
| `RenderMapSmoothing` | `float` | -1 | -1 to 5 | Smoothed render map: kernel radius of the import, in node spacings. -1 = the importer default (1.8), 0 = off (barycentric map of earlier versions), 1 to 3 useful. Used by the Bake panel and `import_baked_body`; the console `import_folder ... smooth=<factor>` overrides it per import. |
| `RenderMapSmoothingFace` | `float` | -1 | -1 to 5 | The same for the `face` and `neck_collar` regions (wide, shallow dents). -1 = the importer default (2.5). |
| `RenderMapSmoothingHand` | `float` | -1 | -1 to 5 | The same for the `hand*` and `finger*` regions. -1 = the importer default (0: no smoothing there, `RenderMapFormat` 1). Hands are finely meshed and their dents small; the kernel is expensive on them (see the cost table below). |
| `RenderMapSmoothingMaxNodes` | `int32` | -1 | -1 or 8 to 64 | Cap of display nodes one smoothed render vertex reads. -1 = the importer default (32). |
| `RenderMapSmoothingOrder` | `int32` | -1 | -1 to 2 | Polynomial order the smoothing reproduces exactly (0 plain average, 1 linear, 2 quadratic). -1 = the importer default (1). |
| `bRenderMapSeamBlend` | `bool` | true | | Seam blend of the import: the region weights add up to 1 along every seam between regions (see `bSeamBlend` below). false = the weights of earlier versions. |

### Import options (render map)

Every render vertex of the display mesh gets its displacement from the display nodes of the region through the **render map**, which the importer builds from positions. Since this version the map can be **smoothed** (`RenderMapFormat` 2): instead of interpolating the 3 nodes of the closest display triangle (piecewise linear, so the roughness of the baked solid's surface, about 4 degrees of normal noise at the 90th percentile, shows as wavy or blotchy dent outlines on toon shading), the vertex takes a weighted sum over the display nodes within `SmoothRadiusFactor` times the local node spacing. The weights are a Wendland C2 kernel with a moving-least-squares correction (partition of unity; with `SmoothOrder` 1 a linear displacement field and with 2 a quadratic one is reproduced exactly), computed along the display mesh surface (geodesic distance, so a neighbouring finger or the far side of a thin part never contributes), within the region, and capped at `SmoothMaxNodes` (default 32) nodes per vertex. It costs nothing at run time beyond reading the extra nodes (the map is larger: about 10 nodes per vertex at factor 1.8, 20 at 2.5, 30 at 3). The `hand*` and `finger*` regions are not smoothed by default (`SmoothRadiusFactorHand` 0, format 1).

| Option | Where | Default | Effect |
|---|---|---|---|
| `SmoothRadiusFactor` | `FSoftPokeImportOptions`, `import_folder smooth=`, `RenderMapSmoothing` | 1.8 | Kernel radius in local node spacings (a node spacing is 2.5 to 3.2 mm on the shipped bakes) for all regions except face and neck. 0 = off. Larger = rounder dents and a lower peak (see below). |
| `SmoothRadiusFactorFace` | `FSoftPokeImportOptions`, `import_folder smoothface=`, `RenderMapSmoothingFace` | 2.5 | The same for regions whose name starts with `face` or `neck`: their dents are wide, and the toon outline is where roundness matters most. < 0 = same as `SmoothRadiusFactor`. |
| `SmoothRadiusFactorHand` | `FSoftPokeImportOptions`, `import_folder smoothhand=`, `RenderMapSmoothingHand` | 0 | The same for regions whose name starts with `hand` or `finger`. Default 0 = no smoothing (format 1 map): the fingers are finely meshed and the dents there are small, and the kernel costs a lot on a dense hand. < 0 = same as `SmoothRadiusFactor`. |
| `SmoothMaxNodes` | `FSoftPokeImportOptions`, `import_folder smoothcap=`, `RenderMapSmoothingMaxNodes` | 32 | Hard cap of display nodes per smoothed render vertex (the farthest are dropped, the weights renormalised), 8 to 64. At the default factors only a few face / neck vertices exceed 32 nodes (the body stays under 21), so the cap changes nothing visible there (peak depth within 1 percent); it bounds memory and GPU time when you raise a factor (at `smooth=3` about 30 nodes on average, up to 62). |
| `SmoothOrder` | `FSoftPokeImportOptions`, `import_folder smoothorder=`, `RenderMapSmoothingOrder` | 1 | 0 = plain kernel average (smoothest per radius, loses the most depth), 1 = linear precision, 2 = quadratic precision (keeps the depth of a parabolic dent but smooths less). |
| `USoftPokeBakeAsset::RenderMapSmoothRadius` | read only | | The factor a map was built with (0 = barycentric). |
| `bSeamBlend` | `FSoftPokeImportOptions`, `import_folder seamblend=`, `bRenderMapSeamBlend` | true | Partition of unity along every seam between regions. Each region's weight fades to 0 at the rim of its patch; where two patches overlap by less than twice the fade (25 mm) or only touch rim to rim the sum used to fall to about 0 (no dent, no wave on that line). The missing weight is now shared by the regions near the seam: a region is extended past its free rim (new map entries at the closest rim point, so the rim displacement continues unchanged) and the regions crossfade continuously by their own taper weight; a region near a fixed (clamped) rim gives way to the others. Vertices where only one region is near the rim that causes the deficit (true outer edges) are not changed. The render mesh may consist of separate unwelded pieces: their rims are joined to nearby vertices of other pieces for this step. Sample body: 17,328 vertices raised, 16,390 new entries, 0.6 s. |
| `SeamReachMm` | `FSoftPokeImportOptions`, `import_folder seamreach=` | 40 | How far a region may be extended past its free rim (mm, along the render mesh); its share fades to 0 there. |
| `SeamGapMm` | `FSoftPokeImportOptions`, `import_folder seamgap=` | 15 | Skin that no region maps counts as part of a seam only in strips up to this wide (thin uncovered lines between rims). Wider unmapped skin (the back of the neck of the sample, the crotch, the scalp) is left as it is. |

Trade-off (VRoid face cheek, 6 mm poke):

| `smooth=` / `smoothorder=` | peak vs old map | render normal roughness (p90) |
|---|---|---|
| 0 (old map) | 100 % | 4.7 degrees |
| 2.0 / 0 | 96 % | 4.6 |
| 2.0 / 1 | 96 % | 4.5 |
| **2.5 / 1 (face default)** | **93 %** | **4.0** |
| 3.0 / 1 | 88 % | 3.8 |
| 3.0 / 2 | 101 % | 4.8 |
| 4.0 / 2 | 98 % | 4.5 |
| 5.0 / 2 | 94 % | 4.5 |

At the defaults the measured peak render displacement changes as follows (old map to new map): VRoid face cheek 7.7 to 7.1 mm (-7 percent; normal roughness p90 4.7 to 4.0 degrees, p99 10.2 to 8.3), VRoid forearm 11.4 to 11.3 mm (-1 percent), VRoid shin 4.8 to 4.7 mm (-1.5 percent), MetaHuman body 1 forearm (7 mm fingertip, 9 mm deep) 11.7 to 10.8 mm (-8 percent), MetaHuman face cheek 5.8 to 5.6 mm (-3.5 percent; roughness p90 8.4 to 7.7). A larger factor on the body rounds more and costs more: `smooth=2.5` takes the body 1 forearm to 9.6 mm (-18 percent), so it is not the body default. Roughness is the angle between the normal of a displaced vertex and the normal of the same displacement after two Laplacian smoothing passes, over the vertices that move by more than 0.3 mm.
Cost of the smoothing (measured with per-pass GPU timestamps, median of the `RenderDispSparse` pass while a fingertip holds the contact; map size = sum of the region blobs):

| Part | format 1 (`smooth=0`) | smoothed, 1.8 (face 2.5) | note |
|---|---|---|---|
| MetaHuman body 1, hand (84,700 render vertices): pass | 38 us | 230 us (450 us at `smooth=2.5 smoothorder=2`) | why hands are not smoothed by default |
| MetaHuman body 1, hand: map | 1.3 MiB | 9.0 MiB (15.9 MiB at 2.5 / 2) | 10.7 nodes per vertex on average, at most 21 |
| MetaHuman body 1, whole body: map | 7.1 MiB | 46.6 MiB, 32.1 MiB with the hands left at format 1 (default) | |
| VRoid body, whole body: map | 1.2 MiB | 8.9 MiB, 7.1 MiB with the hands at format 1 (default) | |
| VRoid, hand (10,400 render vertices): pass | 12 us | 53 us, 14 us at format 1 (default) | |
| VRoid, cheek contact (face + neck + torso): pass sum | 124 us | 460 us | face / neck use factor 2.5 (20 nodes) |

Re-import the body (`import_folder` or the panel) to change the setting; old assets keep working unchanged (format 1).

## Console commands and variables

**`SoftPoke.Cmd <verb> <arguments>`** (editor). The result is written to the Output Log, category `LogSoftPoke`.

| Verb | Arguments | Purpose |
|---|---|---|
| `build_deformer` | `<package path> [normals 0\|1] [mask=0\|1] [range=<mm>]` | Creates the Deformer Graph asset, for example `/Game/SoftPoke/DG_SoftPoke`. `mask=1 range=12` builds the variant that writes the dent mask for the toon overlay. |
| `import_folder` | `<folder> <package dir> <skeletal mesh object path> [body name] [region=bone ...] [fine=0] [lazy=0] [creases=0] [clampedrim=0] [rimtaper=<mm>] [fill=0] [fillreach=<mm>] [fillk=<n>] [smooth=<factor>] [smoothface=<factor>] [smoothhand=<factor>] [smoothcap=<n>] [smoothorder=0\|1\|2]` | Imports every `.spk` of a folder: one asset per region (plus mirrored partners) and the body asset `SPB_<body name>`. Use the display mesh as a full object path (`/Game/Dir/Name.Name`). `smooth=` / `smoothface=` set the smoothed render map (see [Import options](#import-options-render-map)): kernel radius in node spacings, default 1.8 (face and neck regions 2.5, hand and finger regions 0 = not smoothed), `0` = the old barycentric map. `smoothhand=` sets the hand / finger factor, `smoothcap=` the node cap per vertex (default 32). |
| `import` | `<spk file> <package path> <skeletal mesh object path> [attach bone] [region name]` | Imports a single `.spk`. |
| `assign_body` | `<actor label> <body asset path> [deformer asset path]` | Adds or finds the `SoftPokeComponent`, sets `BodyAsset`, sets the mesh deformer (also registered in the mesh asset's `TargetMeshDeformers`, which cooked builds need). Marks packages dirty; saves nothing. |
| `prepare_metahuman` | `<actor label>` | The console form of `PrepareMetaHumanActor` (see above). |
| `assign_face` | `<actor label> <face mesh component name> <body asset path> [deformer asset path] [driver 0\|1]` | Second `SoftPokeComponent` for the face mesh of a split body + face actor. |
| `map_stats` | `<body asset path>` | Render-map statistics and CRC check of every region. |
| `list_regions` | `<body asset path>` | Regions, bones and mirrored partners. |
| `export_raw` | `<skeletal mesh object path> <out file> [lod]` | Writes the bake input from a skeletal mesh. |
| `bake_panel_open`, `bake_panel_set key=value ...`, `bake_panel_check`, `bake_panel_setup`, `bake_panel_run`, `bake_panel_status`, `bake_panel_cancel` | | Scriptable Bake panel (same controller as the panel). |

**Console variables** (`softpoke.*`). Variables marked "dev" are hidden and read-only in a Shipping build.

| Variable | Default | Purpose |
|---|---|---|
| `softpoke.Touch.Enable` | 1 | Touch events kill switch: 0 = every `SoftPokeTouchSensor` is inactive (no contact rows, no events). |
| `softpoke.Touch.WebSocket` / `softpoke.Touch.Webhook` | -1 | -1 = follow the project settings, 0 = force the transport off, 1 = force it on. |
| `softpoke.Touch.UpdateHz` | 0 | `touch_update` rate for every sensor; 0 = follow the component. |
| `softpoke.Touch.Log` | 0 | 1 = log every touch event. |
| `softpoke.Touch.Token`, `softpoke.Touch.WebhookToken`, `softpoke.Touch.BindAddress`, `softpoke.Touch.WebhookUrls`, `softpoke.Touch.WebhookEvents`, `softpoke.Touch.AllowRemoteWebhook` | empty / -1 | Test aids (not in Shipping builds): non-empty (or 0 / 1) replaces the same project setting (`none` = no token / no URLs). |
| `softpoke.Touch.Flood` | | Command, test aid (not in Shipping builds): `softpoke.Touch.Flood <count> [block]` publishes synthetic events to test slow receivers. |
| `softpoke.Touch.RowLog` | 0 | 1 = record the contact rows of every tick to `Saved/SoftPoke/touch_rows.jsonl` (development aid, not in Shipping builds). |
| *commands* | | `softpoke.Touch.Status` (counters, queues, drops, game-thread time; also `Saved/SoftPoke/touch_status.json`), `softpoke.Touch.ProbePart x y z`, `softpoke.Touch.DumpParts file.csv` (the last two not in Shipping builds). |
| `softpoke.LimiterParallelMinPairs` | 64 | The limiter's penetration query, the second hand and the per-region capsule preparation run on worker threads; the query does so when at least this many (region, hand sample) pairs need the surface search. 0 = everything on the game thread. The result does not depend on it. |
| `softpoke.HeadCollider` | -1 | -1 = use `bHeadCollider` of the rig, 0 = force off, 1 = force on. |
| `softpoke.LimiterCapHoles` | 1 | The penetration limiter closes the openings of a region patch (the eye / nose / mouth opening of the face region) with a shallow dome so a hand stops there. 0 = off. Read when the regions load. |
| `softpoke.FaceMode` (dev) | -1 | Mode of every face driver: -1 = the component property, 0 = Tracker, 1 = Cycle, 2 = Static, 3 = Off. |
| `softpoke.FaceSet <name> <value> ...` (dev, command) | | Sets static ARKit values on every face driver and switches it to Static mode. `clear` = neutral. |
| `softpoke.HeadDump` (command) | | Prints the latest head pose and non-zero blendshapes of every UDP source once. |
| `softpoke.HeadLog <seconds>` (dev) | 0 | Logs the same every N seconds (0 = off). |
| `softpoke.StylizeLowPass` | 1 | 1 = the temporal low-pass of the stylize output is on, 0 = off. |
| `softpoke.ContactNormalMode` (dev) | -1 | -1 = per region (fingertip contact mode for regions flagged for it), 0 = the plain contact model everywhere, 2 = fingertip contact mode everywhere. |
| `softpoke.SolveGate` (dev) | 1 | 1 = a region is solved only while a capsule can produce a contact candidate. |
| `softpoke.UploadPath` | 1 | Upload path of the static region data (1 = direct RHI upload, 0 = legacy path). |

Further `softpoke.*` variables are developer tuning knobs and are not part of the supported interface.

| Variable | Default | Purpose |
|---|---|---|
| `softpoke.Ripple` | `-1` | Contact ripple: -1 follows `bRipple`, 0 forces it off (A/B tests: same output and cost as a build without the feature), 1 forces it on. |
| `softpoke.SolveBudgetMs` | `-2` | Override of `SolveGpuBudgetMs` for every component: -2 = use the property, -1 = automatic, 0 = unlimited, > 0 = budget in ms. |
| `softpoke.RippleWaveShapeGainMax`, `softpoke.RippleWaveShapeGainExponent`, `softpoke.RippleCarrySeamBandCm`, `softpoke.RippleAnatBlurCm` | `-1` | Overrides of the properties of the same meaning for every component (-1 = use the property). |
| `softpoke.RippleAnatFadeMs` | `60` | Time constant (ms) of the cross-fade of the anatomical wave envelope when the soft part of a held contact changes (0 = at once). |
| `softpoke.RippleCarryExportFirst` | `0` | Diagnostics: 1 = a touched region publishes its own wave before the carry adds a neighbour's wave (single hop). |
| `softpoke.RippleLog` | `0` | 1 = read back the bulge state of every contact solve and log it (development only). |
