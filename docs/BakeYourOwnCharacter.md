# Bake Your Own Character

> SoftPoke 1.0.0 | [Index](Index.md)

SoftPoke deforms a character from **baked data**: for every body region (an upper arm, a hand, ...) a finite-element model of the soft tissue is computed once on your PC and stored in compact `.spk` files. The bake runs in an external Python program that the **SoftPoke Bake** panel starts for you. This page walks through the panel, the input your character must meet, the quality gate, the **Set up** button, and the extra steps for VRoid avatars.

**Adult humanoid characters only.** The quality gate is calibrated on adult anthropometrics and rejects child-like or heavily stylised proportions; the product does not support them.

Short route: [check the input](#input-checklist) -> [**Set up**](#the-set-up-button) (once) -> [panel](#the-bake-panel) -> [read the gate](#the-quality-gate) -> Play.

## Regions

SoftPoke supports baking the **full body**. A character is split into regions (arms, legs, hands and further body regions); the right side is created from the left by mirroring unless you turn mirroring off. The bake tool prints the name of every region it finds and bakes.

* **Regions** = `all` (the default; an empty field means `all`) bakes every region the tool detects, including `face` and `neck_collar` when a head with a face is found. This is the setting for a finished character. `list` only prints which regions this body offers and bakes nothing. Add `--no-face` (for example `all --no-face`) to leave `face` and `neck_collar` out of `all`; or give names, for example `upperarm_L,forearm_L,face`.
* For a first test, type a short list, for example `forearm_L` or `forearm_L,hand_L` (the names the tool prints, such as `upperarm_L`, `forearm_L`, `thigh_L`, `shin_L`, `hand_L`). A single limb region in *Draft* quality takes minutes.

* **`face`** and **`neck_collar`** are part of `all` when the skin mesh contains a head with a face: see [Face and neck](#face-and-neck). On the command line, `--regions list` prints the regions a mesh offers; `--no-face` leaves the two out of `all`.

## Face and neck

Two regions cover the head and the shoulder girdle:

* **`face`**: the front of the head from just under the chin up to the top of the head, in front of the ears. The open features of a face are left out on purpose: eyes, nose, mouth and lips, nostrils, and ears (a 14 mm margin around them, plus 6 mm from any mesh opening), because the soft-tissue model cannot answer a poke on a thin lid or a lip.
* **`neck_collar`**: the neck, throat, nape, collarbones, the notch between them, the top of the shoulders (trapezius) and the upper chest and back, out to the shoulder joints.

They are baked from the same skin mesh, with the same panel and the same command line as every other region. `Regions` = `all` includes them; to bake only these two, enter `face,neck_collar`.

What the mesh must contain:

* The **head with the face**: nose, mouth and ears in the skin mesh (eyes may be modelled as open holes, as VRoid does, or as eyelids). No hair, hat or glasses in the baked slots; the skin slot filter (**Material slots**) takes care of that for a VRoid avatar.
* The rig roles `head`, `neck`, `clavicle` and `upperarm` (the automatic bone map finds them for MetaHuman-style, Mixamo and VRM `J_Bip` rigs).
* The face must look to the character's front. The head may carry open eyes, mouth, nostrils and ear canals and welded lips: the quality gate does not count these (code `head_details`, information only) and the automatic repair leaves them alone.

You do not supply any landmark data. The tool measures the face itself and reports in the gate item `face_landmarks` how each landmark was found: the **nose tip** (the most forward point of the head), the **chin bottom** (where the profile of the mid-plane section turns back under the chin), the **mouth** (a mouth opening if the mesh has one, otherwise the densest line of the mesh between nose and chin), the **eyes** (a mirrored pair of eye openings if the mesh has them, otherwise the height of the rig's eye bones, otherwise head proportions), the **ears** (dense ear geometry, otherwise half the head depth) and the **skull width**. `mesh` = measured, `bone` = taken from a rig bone, `proportion` = estimated from head proportions. An estimate only moves the thickness map and the keep-out margins by a few millimetres. If the head cannot be found at all (the mesh ends below the head, the face does not look to the front, hair is part of the mesh) the two regions are skipped with the reason, in the log, in the gate item and in `manifest.json`, and the rest of the body is baked as usual. Asking for them by name in that case reports an error for exactly those two regions.

What you get:

* A **static** face and neck: the dent of the resting expression. The regions are attached to the `head` bone (face) and the `neck` bone (neck_collar) and follow those bones; they do not follow the jaw or facial expressions. VRoid expressions are blend shapes, which the joint-frame following cannot see, so a VRoid face dents correctly in its neutral expression only. A MetaHuman-style face rig (face joints) is likewise not followed by this bake.
* The surfaces around the eyes, nose, lips and ears **cannot be poked** (they are left out). Pokes on cheeks, jaw line, chin, forehead and temples, and on the neck, collarbones, notch and shoulder tops work. Skin of the neck, chin and cheeks is softer than the forearm values used for the limbs; the thickness maps are estimates (see [Behaviour and limits](BehaviourAndLimits.md#face-and-neck)).
* Time: on the reference PC, CPU path, Standard quality, the two regions together took about 11 minutes for a MetaHuman-style full-body mesh (659 s, 2500 and 1400 patch vertices) and about 5 minutes for a VRoid avatar (294 s, the `_SKIN` slot). The compression is fixed (face: rank 192, near radius 30 mm; neck_collar: rank 384, 30 mm) instead of being tuned, which is why these two bake faster than a limb. The data is about 16 MB for the face and 20 MB for the neck on a full-resolution head (on the VRoid test avatar: about 17 MB and 6 MB).

## Input checklist

Go through this before the first bake. A "no" is a fix in your modelling tool or in Unreal, or a reason to stop.

**Character**

* [ ] Adult humanoid proportions (an adult head-to-body ratio, normal limb length).
* [ ] Real-world size: the skeleton is roughly 1.2 to 2.3 m tall. The gate warns outside 1.2-2.3 m and fails outside 0.9-2.8 m.

**Skin mesh (the part that is baked)**

* [ ] A **body-only skin surface**. No clothes, hair, accessories, shoes, eyes, teeth, or inner underlay shells in the same mesh or in the material slots you select. Anything included is treated as flesh and gives wrong thickness or a failed gate. (Clothes are fine in other material slots or other meshes; you choose the skin slot in the panel.)
* [ ] A closed (watertight) surface; the only openings are wrists, ankles and neck (and, in the head, the eyes, mouth, nostrils and ear canals, which the gate does not count). No overlapping or duplicate shells, no zero-area faces.
* [ ] Edge length in the contact areas: median 15 mm or less. Sparser meshes (some VRoid exports have about 18 mm) still bake but dent poorly; see [the VRoid route](#the-vroid-route-and-display-subdivision).
* [ ] Normals point outward.
* [ ] A-pose or T-pose with arms and legs clear of the body and of each other (thighs not touching).

**Skeleton and weights**

* [ ] One humanoid skeleton with the usual roles: pelvis, spine, neck, head, upper arm, lower arm, hand, thigh, calf, foot, ball/toe, both sides, and the finger chains of both hands (hands are built from them). UE5 / MetaHuman, Mixamo, Rigify, VRM (`J_Bip`) names are recognised; others need a bone-map file (below).
* [ ] Left and right bones on the correct sides.
* [ ] No scale on bones, no non-uniform scale on the mesh.
* [ ] Every skin vertex has weights, normalised, at most 4-8 influences per vertex (the gate warns above 12 and fails above 24).
* [ ] Smooth weights, not proximity-only steps at the joints.

**Export and time**

* [ ] LOD 0 is baked and shown. The data is tied to the vertex order of LOD 0 of the **display** mesh; if you change that mesh later, import again.
* [ ] 1-3 GB of work space per body; 90-125 MB for the final data of a whole body.
* [ ] Start with one limb region in *Draft* quality to test the pipeline and the gate.

The same list, with the gate code for each line, is in `Extras/AgentSkills/softpoke-import-avatar/reference/input-checklist.md`.

## The Bake panel

**Window > SoftPoke Bake** (or type the console command `SoftPoke.BakePanel` into the **Cmd** box of the Output Log). The panel stays usable while a job runs. From top to bottom:

| Field | What to enter |
|---|---|
| **Python** + **Check** | The bake Python (`python.exe`). Empty = the environment variable `SOFTPOKE_PYTHON`, then `python` on PATH. **Check** verifies that numpy, scipy, pypardiso and pytetwild import. After **Set up** this is filled in for you. |
| **Base Python**, **Venv folder**, **GPU packages**, **Set up** | See [The Set up button](#the-set-up-button). |
| **Bake tool folder** | Folder with `ab_pipeline.py`. Empty = `<plugin>/Tools/Bake`. |
| **Skeletal mesh to bake** | The skeletal mesh asset to bake. For a VRoid avatar: the original import, not the subdivided display copy. |
| **Material slots** | Comma-separated pieces of material slot names to bake, for example `Body_00_SKIN` or `_SKIN`. Empty = the whole mesh (only if it is body-only). Look the names up in the mesh asset's Material Slots. |
| **Regions** | The list from [Regions](#regions). |
| **Bone map** | `auto` (detects UE5/MetaHuman, Mixamo/Rigify-style names, and VRM/VRoid `J_Bip`), `regex`, `vrm`, or the path of a JSON bone-map file. |
| **Quality** | *Draft* (minutes: no jiggle, coarse display mesh), *No jiggle* (everything but the jiggle modes), *Standard* (the validated default). |
| **GPU (risky with the editor open)** | Use the CUDA path: much faster, but a long CUDA job while the editor is open can reset the display driver and crash the editor. **Default is CPU.** The GPU path also needs the optional GPU packages. |
| **Accept gate warnings** | Bake although the quality gate gave warnings. Errors are never overridden. |
| **Work folder** | Where the bake writes. Empty = `<Project>/Saved/SoftPokeTools/Bakes/<mesh>_<time>` (1-3 GB). |
| **Target actor** + **Use selected** | The label of the level actor that shows the character. Its skinned mesh is the **display mesh**: the data is mapped onto it and a `SoftPokeComponent` is added to the actor. Leave it empty to import without assigning. |
| **Content folder** | Where the assets are created, for example `/Game/SoftPokeBaked`. |
| **Deformer graph** | For example `/Game/SoftPoke/DG_SoftPoke`; built when missing. |
| **Bake**, **Cancel**, **Open work folder** | Cancel stops only the bake's own process. |

A job runs five stages: export the mesh, bake (the long stage, a child process), write the compact `.spk` files, import them as assets, assign the body to the actor. The panel shows the stage, a log, and the quality-gate result.

The same operations are available to scripts and AI agents: `unreal.SoftPokeBakeLibrary` (functions `set_bake_param`, `start_python_check`, `start_setup`, `start_bake`, `cancel_bake`, `is_bake_running`, `get_bake_status_json`, `import_baked_body`, `assign_body_to_actor`, `export_skeletal_mesh_to_sp_raw`) and the console verbs `bake_panel_set key=value ...`, `bake_panel_check`, `bake_panel_setup`, `bake_panel_run`, `bake_panel_status`, `bake_panel_cancel`. Keys of `set_bake_param`:

`source`, `slots`, `out`, `quality` (`draft`, `nojiggle`, `standard`), `regions`, `bonemap`, `gpu` (0/1), `force` (0/1), `lod`, `actor`, `package`, `body`, `deformer`, `python`, `tools`, and for Set up `basepython`, `venv`, `setup_gpu` (0/1). See the [component reference](ComponentReference.md#editor-python-and-blueprint-library-usoftpokebakelibrary).

```python
import unreal, json
lib = unreal.SoftPokeBakeLibrary
for k, v in dict(source='/Game/Characters/SK_Body', slots='Body_00_SKIN', quality='draft',
                 regions='forearm_L', bonemap='auto', gpu='0', actor='MyCharacter',
                 package='/Game/SoftPokeBaked', body='MyBody',
                 deformer='/Game/SoftPoke/DG_SoftPoke').items():
    lib.set_bake_param(k, v)
lib.start_bake()
st = json.loads(lib.get_bake_status_json(40))   # poll until st['busy'] is False
print(st['state'], st['status'], st['gate'], st['body_asset'])
```

States: `Idle`, `Checking`, `SettingUp`, `Exporting`, `Baking`, `WritingSpk`, `Importing`, `Assigning`, `Done`, `Blocked` (the gate warned: nothing was baked), `Failed`, `Cancelled`.

### How long does it take?

Plan generously, and always test one limb first.

* **One limb in *Draft* quality:** about 10 minutes.
* **One limb in *Standard* quality (CPU):** about 15 minutes for an upper arm on the reference PC (925 s); a forearm plus a hand took about 20 minutes together.
* **A whole body in *Standard* quality:** up to **several hours on the CPU** (the default; the editor stays usable but busy). The GPU path (RTX 4090) took about 11 minutes, but a long GPU job while the editor is open can reset the display driver and crash the editor: close other GPU programs and save your work first.
* Times depend strongly on your CPU, and a second run of the same region can be slower than the first. Close other heavy programs while baking.

## The quality gate

Before anything is baked, the tool runs a **quality gate** on your input and tries to repair what can be repaired automatically (small holes, cracks, duplicate faces, fused fingertips). The panel shows one of:

* **PASS** - the body was baked.
* **WARN** - the tool found something it is not sure about and **has not baked anything yet**. What to do: read each message in the panel (every message has a code, an explanation and a suggested fix). If you can fix the cause in your character (for example smooth the weights or close a hole), do that and bake again. If the message is harmless for your purpose (a VRoid body always gives a `density` warning), tick **Accept gate warnings** and bake again.
* **FAIL** - errors. They are never overridden. Fix the cause.

The command-line tool reports the same as its exit code: `0` baked, `10` warnings only (nothing baked, re-run with `--force`), `20` errors. In the work folder, `quality_report.json` / `.txt` is the result that decided, `quality_report_input.json` the first run on the raw input, `repair_report.json` / `.txt` what the repair changed, and `manifest.json` the status per region.

Every message has a code (`non_manifold`, `bone_map`, `skeleton_height`, `density`, `weight_leak`, ...), an explanation and a suggested fix. The full list, with causes, fixes to try and when a body is simply unsupported, is the **gate catalog**: `Extras/AgentSkills/softpoke-bake-troubleshoot/reference/gate-catalog.md`. It is written for AI agents but reads fine for people; the Bake-troubleshoot skill (`SKILL.md`) explains the whole procedure. A few that you will meet often:

| Code | In one line |
|---|---|
| `bone_map` | Bone names could not be mapped to roles. Write a bone-map JSON (format in `Extras/AgentSkills/softpoke-import-avatar/reference/bone-map.md`) and enter its path in **Bone map**. |
| `skeleton_height`, `limb_lengths` | Scale or proportions outside the adult range. Fix the import scale; do not move joints to pass. Child-like proportions are unsupported. |
| `open_boundary`, `non_manifold`, `inner_shell` | The skin is not a clean closed surface: holes, overlapping shells, an underlay mesh. The automatic repair handles small cases; otherwise fix the mesh or exclude the extra shells with **Material slots**. |
| `density` | The mesh is too coarse. For VRoid: accept the warning and use a subdivided display mesh. |
| `weight_leak`, `weight_mottled`, `joint_blend` | Skin weights are rough. Smooth them in your modelling tool. |
| `head_details` | Information: how many open edges, non-manifold edges and duplicate faces were found in the head and not counted (eyes, mouth, nostrils, ears, closed lips). |
| `face_landmarks` | Information: whether the `face` and `neck_collar` regions can be baked from this mesh and which cue placed each landmark (`mesh`, `bone`, `proportion`), or why they are not available. |
| `winding_normalized` | Information: before the checks the loader turns every shell's faces outward and flips faces that disagree with their neighbours (geometry unchanged), so a mesh exported with flipped faces or a flipped shell is judged by its shape, not by its winding. The bake uses the same corrected faces. Value: faces and shells changed. |
| `isolated_fixed_nodes` | Reported **after** the face / neck bake (in `manifest.json` under the region's `bake_checks` and in the `result` line of `--json`), not by the input gate. Information with value 0 when fine; a warning when some skin nodes in the middle of the patch are clamped. A poke on such a node gives a small pinned spot. It never blocks the bake. |

## Options for hands and friction (command line)

These are options of the command-line tools; the panel's defaults are unchanged. Pass them to `ab_pipeline.py` (they are recorded in `manifest.json`, and `ab_spk.py` reads them from there) or directly to `ab_spk.py`:

| Option | What it does |
|---|---|
| `--mu 0.5` / `--mu hand=0.3` | Friction coefficient written into the `.spk` metadata. A bare number applies to every region; `name=value` to a region (`thigh_L=0.4`) or a kind (`hand` = `hand_L`, and `hand_R` which shares its file). Repeatable. `0` is frictionless; `unset` (or a negative number) writes "not set" and the importer uses 0.5. Default: 0.5 for every region. |
| `--pad-pole 24` | Hand regions: the rest normals of the finger-pad side within 24 mm of every fingertip are blended toward that finger's pad pole (the robust mean of the palm-side normals around the pad), per finger. Weight 1 inside 14 mm, fading to 0 at 24 mm. A finger whose pole is more than 35 degrees from the palm normal is left alone. |
| `--palm-thk-smooth 14` | Hand regions: palm soft-tissue thickness becomes the local minimum over a 14 mm radius, so one thin spot of a coarse bone proxy no longer caps the whole palm. |
| `--rig-raw FILE.npz` | The raw skeleton the hand options read (default: the `raw_*.npz` next to the region files; `ab_pipeline.py` records it). |

**Recommended for stylised and AI-generated hands** (thin fingertips, noisy normals): `--mu hand=0.3 --pad-pole 24 --palm-thk-smooth 14`. On the sample character this took the index pad from 2.5 to 0.9 of the allowed depth and removed the thumb flips. The hand options are **off by default**: on our three test hands (a MetaHuman body, a second photoreal body and a VRoid) the finger poles are sound (3 to 21 degrees from the palm normal), but the thumb pole is about 40 degrees off on the first two, so the thumb is skipped there, and we have not poke-tested these hands with the options on. Try them, compare, keep what looks better.

The same pad pole is the normal of the standard poke site for a fingertip: the node nearest to the pad target point (0.55 of the last phalanx beyond the last joint) among the palm-side nodes, with the pole as its normal (`ab_handtune.pad_sites`). Use it for poke tests, so a thin tip is tested from the direction a real finger touches it.

## The Set up button

The bake needs Python 3.12 with scientific packages. **Nothing is bundled with the plugin.** The **Set up** button builds a private environment for you with `pip`:

1. Install a **64-bit Python 3.12** from python.org on your PC (the Microsoft Store stub does not count). Other Python versions are not supported.
2. In the panel, leave **Base Python** empty (the first `python.exe` on PATH is used) or enter the path of the Python 3.12 `python.exe`. **Venv folder** empty = `<Project>/Saved/SoftPokeBake/venv`. Tick **GPU packages** only if you want the optional CUDA path (NVIDIA GPU, about 2 GB more).
3. Press **Set up**. The panel downloads the packages listed in `Tools/Bake/requirements.lock` from the Python package index (internet required; about 1.5 GB for the CPU packages), installs them into the venv, and runs a self-check (imports, a sparse-solver test, and a GPU report). Nothing is installed outside the venv folder.
4. On success the venv's `python.exe` becomes the bake Python and the **Python** field shows it. Press **Set up** again at any time: an existing venv is reused and pip resumes.

On the reference PC the CPU set-up took about a minute (internet speed dominates). If it fails, the log shows why: exit code 30 = the base Python is not 3.12 / 64-bit, 31 = the venv could not be created, 32 = pip failed (no network or proxy, disk full, a locked file, no wheel for this Python), 33 = the self-check failed, 34 = bad arguments or a missing lock file. A GPU result of "cupy not installed" is not an error; it only means the GPU path is not set up.

Licence note: the packages are downloaded to your PC by you, under their own licences (see [Licences](Licences.md)); SoftPoke does not redistribute them.

## The VRoid route and display subdivision

VRoid Studio avatars are stylised and consist of several parts (body skin, face, hair, clothes) in one mesh. They work with SoftPoke if you follow these steps. Import a `.vrm` into Unreal with a VRM importer plugin that you install yourself (SoftPoke does not ship one; the route was tested with the community plugin VRM4U).

1. **Export from VRoid Studio in tight-fitting clothes.** With loose clothes the body underneath is cut away and incomplete; with tight clothes the body under them is complete. The clothes stay separate parts and are excluded by the material-slot filter.
2. **Make a display copy with subdivided skin.** VRoid skin is coarse (about 18 mm edges), so a fingertip dent covers a handful of vertices. Run the plugin's helper with the bake Python:

   ```
   "<bake python>" "<plugin>/Tools/Bake/vrm_prepare_display.py" in.vrm out.vrm
   ```

   It subdivides the skin materials of the copy only: body skin **and face skin** (`Body_NN_SKIN` two levels, `Face_NN_SKIN` one level, smooth; the face is already finer, and one level matches the bake's surface so the dent shades round, `--face-levels` changes it). Eyes, eyelashes, brow and eye lines, the mouth interior, teeth, hair, clothes and every transparent (BLEND) material are never subdivided. The face keeps its expression blend shapes (they are subdivided with the same weights, so expressions still work), its UV seams stay sharp, and its outline stays fixed, so it still meets the body at the neck and the eyes and mouth without a gap. Options: `--no-subdiv` to copy as is, `--no-face` for the body skin only, `--select <regex>` if the skin materials have other names, `--levels`, `--scheme loop|keep|iloop`, `--validate` (also runs `vrm_subdiv_validate.py` and prints how far the new surface deviates from the original). The original file and the baked data are not changed, so no re-bake is needed when you only change the display mesh.
3. Import the **original** `.vrm` with the VRM importer and the **subdivided** `out.vrm` as a second skeletal mesh.
4. In the panel: **Skeletal mesh to bake** = the original import; **Material slots** = the skin slot (a name containing `_SKIN`); **Regions** as above; **Bone map** = `auto` (finds `J_Bip`); **Target actor** = the actor that shows the **subdivided** copy (assign that mesh to the actor first). Expect a `density` warning on the original (about 18 mm edges): read the gate report, then enable **Accept gate warnings**.
5. Bake. The result is mapped onto the subdivided display mesh.

Notes and limits of this route:

* Fingertips: the automatic repair welds vertices closer than 0.5 mm (never more than 0.1 times the shortest edge at that point) so that fingertips do not fuse. If tips still look pinched, the command-line tool `ab_repair.py --weld 0.1` can be run separately (see the bake-troubleshoot skill).
* Facial expressions on VRM avatars are blend-shape driven and are **not** followed. The `face` and `neck_collar` regions are baked from the VRoid skin like any other region and dent the neutral face and the neck; see [Face and neck](#face-and-neck). The VRoid head is baked from the original (not subdivided) mesh as well; eyes modelled as holes and a mouth slit are detected and kept out of the patch.
* A thin, strongly curved fingertip can be poked less reliably than a flat area.
* Clothes do not deform with the skin (see [Behaviour and limits](BehaviourAndLimits.md)).
* VRM models have licence terms of their own. Check that the model's licence permits modification and your use.

### The dent overlay (toon characters)

On flat-shaded (toon) materials a geometric dent can be hard to see. The optional **dent overlay** tints the dented area.

1. In the editor run the helper once (it creates the overlay material and instances under `/Game/SoftPokeToon`, skipping assets that exist): **Tools > Execute Python Script**, or the console command `py "<Project>/Plugins/SoftPoke/Tools/ue_build_toon_dent_materials.py"`.
2. Build the mask deformer: `SoftPoke.Cmd build_deformer /Game/SoftPoke/DG_SoftPoke_Mask 1 mask=1 range=12`.
3. On the `SoftPokeComponent`: `bDentOverlay` on, `DentOverlayMaterial` = `M_SoftPoke_DentOverlay` (or the instance `MI_SoftPoke_DentOverlay_Anime` / `_Soft`), `DentMaskDeformer` = `DG_SoftPoke_Mask`, `DentScaleMm` = 12 (it **must equal** `range=`), `DentOverlaySlotNames` = the skin slots (empty = every slot except translucent ones).
4. Colour: for MToon-style materials set `ShadeColorSourceParam` to `mtoon_ShadeColor`; lower `ShadeColorScale` (default 0.85) for a darker dent; call `MatchShadeColors()` after you change the character's materials.
5. **Feather.** Parameters of the overlay material instance: `Feather` 1 (default; the tint fades out smoothly at the rim) or 0 (hard stepped cel edge, instance `MI_SoftPoke_DentOverlay_Hard`); `FeatherHiMm` should be close to the deepest dent of that body part (about 5.5 mm on a forearm, 2.8 mm on a shin; default 6); `FeatherLoMm` 0.1. A hard edge with a thin dark outline means `Feather` is 0 or the instance descends from an older overlay material that has no `Feather` parameter (the helper script skips assets that exist, so an old `M_SoftPoke_DentOverlay` stays as it is and the script warns about it): run the script again with `--rebuild` (`py "<Project>/Plugins/SoftPoke/Tools/ue_build_toon_dent_materials.py" --rebuild`). It rebuilds the overlay material in place (same asset path, so its instances keep their values and nothing has to be re-assigned) and creates instances that are missing. `--rebuild-force` rebuilds it even when `Feather` exists.
6. **Different depth per slot.** `DentOverlaySlotOverrides` maps a material slot name to `Feather` and `FeatherHiMm` and is written onto that slot's overlay instance. A face dent is deeper than a forearm dent, so for a VRoid character use for example the face slot `FeatherHiMm` 5 to 6 and the body slot 5.5, or leave it empty and use one value.

The overlay is off by default. A mesh that stays untinted means the mask deformer is not assigned. Lighting matters: a low light from the side shows dents far better than flat frontal light. The Tune skill (`Extras/AgentSkills/softpoke-tune`) has the full procedure.

## After the bake

* The panel has created assets in the content folder: `SPB_<name>` (the body) and, optionally, `SPF_<name>` (skin fill) directly in it, and the regions in a sub-folder named after the body: `<name>/SPK_<region>` and `<name>/SPKP_<region>` (regions and their payloads). Every body therefore has its own regions, and you can bake several characters into the same content folder. The `SPB_` prefix is added for you: a body name typed as `SPB_Hero` gives the same assets as `Hero`. Bodies imported by older versions keep their regions next to the body asset and keep working.
* Check them: `SoftPoke.Cmd list_regions /Game/SoftPokeBaked/SPB_<name>` lists regions, attach bones and mirrored partners; `SoftPoke.Cmd map_stats /Game/SoftPokeBaked/SPB_<name>` prints the render-map statistics (the CRC must say MATCH). When a body is loaded for play the log also prints one line per region, `SoftPoke load: render map of <region> ... CRC32 ... MATCH`, for every region at once (console variable `softpoke.LogRenderMapCrcAtLoad`: 0 off, 1 recorded values only, 2 also checks the blob in memory; default 2 in the editor, 1 in a game build).
* **Region seams.** The importer blends neighbouring regions so that their weights add up to 1 along every seam (seam blend, on by default). The import log has one line `seam blend: ... vertices between regions raised ...`; a few hundred vertices left below 0.95 are normally outer edges (wrists, ankles, crotch, the top of the neck) or skin that no region maps (wider than 15 mm). If a dent looks cut along a line between two regions, check that the line is covered by both regions in the bake (overlapping patches are best); `import_folder ... seamblend=0` gives the weights of earlier versions for comparison.
* **Save** the level and the mesh asset: assigning marks packages dirty but does not save.
* Press Play and poke each region. For the look and feel, see [Behaviour and limits](BehaviourAndLimits.md#tuning).

### Checking the display mesh in Unreal

* **Textures stay old after a re-import.** When you re-import a changed skeletal mesh with Replace, Unreal keeps the existing material instances, and they can still point at the textures of the previous version (and at the previous texture files). After a re-import, open every material instance of the mesh and check its texture parameters, or re-assign them with a script. The SoftPoke data are not affected: the render map follows vertex order, and the importer checks the vertex count.
* **AI-generated hair is often one-sided.** Meshes from image-to-3D generators can have hair that is a single surface layer. With the default one-sided material the fringe edges look like a black jagged line. Enable **Two Sided** on the hair material instance (an override on the instance is enough).
* **Check textures in Blender with the Standard view transform.** Blender's default AgX (or Filmic) view transform lowers contrast and hides seams, steps and colour jumps in a texture, which then first show up in Unreal. Set Colour Management > View Transform to **Standard** when you inspect a texture, and compare with the Unreal viewport.

If something went wrong, the log tail in the panel, `LogSoftPoke` in the Output Log, and the files in the work folder (`manifest.json`, `quality_report.json`) tell the story. An AI agent with the skills from `Extras/AgentSkills/` can read these for you.
