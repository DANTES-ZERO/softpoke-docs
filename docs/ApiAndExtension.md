# API and extension

> SoftPoke 1.0.0 | [Index](Index.md)

Everything named here is listed with types and defaults in the [component reference](ComponentReference.md).

## Driving the body from your own code

`USoftPokeComponent::SetCapsules(const TArray<FSoftPokeCapsule>& Capsules)` (Blueprint-callable) is the single entry point of the solver. Give it the hand as capsules in the **component space of the target mesh, in cm**: two end points `A`, `B`, a `Radius` (cm) and a `FingerGroup` (0 palm, 1 index, 2 middle, 3 ring, 4 pinky, 5 thumb). Call it every frame you want contact; the solver uses the capsules of the latest call. Capsules are the only contact objects. There is no fixed capsule count. `ResetSolver()` clears the state.

If you call `SetCapsules` yourself, do not put a `SoftPokeHandRig` on the same actor (the rig would overwrite your capsules every frame). The rig adds filtering and the penetration limiter; calling `SetCapsules` directly bypasses both.

## A custom hand source

Derive from `USoftPokeHandSource` (C++ or Blueprint; it is abstract, instanced and editable inline) and select your class with `SourceClass` (or assign an instance to `Source`) on the `SoftPokeHandRig`. The rig calls these overridable functions:

| Function | Purpose |
|---|---|
| `Start(UWorld* World)` | Called when the rig begins play. Open sockets, devices. |
| `Stop()` | Called at end of play. |
| `Poll(float DeltaSeconds)` | Once per rig tick, before `GetHand`. Read pending input. |
| `GetHand(ESoftPokeHandSide Side, FSoftPokeHandFrame& Out)` | Fill `Out` and return true when that hand is currently tracked. |
| `Describe()` | A short status string shown as the rig's `Status`. |
| `GetHead(FSoftPokeHeadFrame& Out)` | Optional: the tracked head (network sources). |

`FSoftPokeHandFrame` carries `bTracked`, `Positions` (21 joints, **world space, cm**, MediaPipe order, see below), optional `Rotations` (21 world quaternions; leave empty if you have none) and `Confidence`. The joint order is the MediaPipe hand order:

`0` wrist | `1-4` thumb (CMC, MCP, IP, tip) | `5-8` index (MCP, PIP, DIP, tip) | `9-12` middle | `13-16` ring | `17-20` pinky.

Add the module dependencies your source needs to your own module's `Build.cs`; `SoftPokeRuntime` exports its API (`SOFTPOKERUNTIME_API`).

## Blueprint and Python entry points

* On the hand rig: `GetProxyJoints`, `GetProxyKeypoints`, `GetTargetJoints`, `ResetHands`, `GetHeadFrame`, `GetHeadBlendshapes`, `GetHeadProxyCapsules`.
* On the UDP source: `GetHeadFrame`, `GetBlendshapes`, `GetBlendshape`.
* On the component: `SetCapsules`, `ResetSolver`, `RecreateSolvers`, `PrefetchRegions`, `EvictRegions`, `AreRegionsReady`, `ResetJiggle`, `SetDentOverlayEnabled`, `MatchShadeColors`.
* On the face driver: `SetStaticValue`, `ClearStaticValues`, `GetCurrentCurves`.
* Editor: `USoftPokeBakeLibrary` (`unreal.SoftPokeBakeLibrary` in Python) and the console command `SoftPoke.Cmd` (typed into the **Cmd** box of the Output Log).

## UDP protocol (version 1)

The webcam tracker and any other sender talk to `USoftPokeHandSourceUdp` with this protocol.

**Transport.** One UDP datagram per frame. The payload is **UTF-8 JSON**, at most 65507 bytes. Default port **9763**. The plugin listens on the address in `BindAddress` of the source, else on `UdpBindAddress` in the project settings, else `127.0.0.1` (this PC only). The receiver reads up to 256 datagrams per tick.

**Structure.** The root object is either one hand object, or a multi-hand object, optionally with a `head` object:

```json
{"v":1,"t":12.3456,"hands":[{"hand":"right","conf":0.93,"space":"ue_world_cm","lm":[[x,y,z], ...21...]},
                            {"hand":"left", "conf":0.90,"space":"ue_world_cm","lm":[[x,y,z], ...21...]}],
 "head":{"conf":0.9,"pos_ue_cm":[x,y,z],"quat_ue":[x,y,z,w],"scale":1.06,"bs":[...51 numbers...]}}
```

Single-hand form: the hand fields sit directly at the root next to `v` (and `t`).

| Field | Where | Type | Meaning |
|---|---|---|---|
| `v` | root | integer | Protocol version. Must be `1`; if absent it counts as 1. Any other value rejects the datagram. |
| `t` | root or hand or head | number | Sender clock in seconds. Statistics only: the receiver timestamps arrivals with its own clock. |
| `hands` | root | array of hand objects | Multi-hand form. Non-object entries are ignored. |
| `hand` | hand | string | `"left"` or `"right"`: the user's **anatomical** hand. A value starting with `l` or `L` is left; anything else is right. **Required**: a hand object without it is skipped (error `missing 'hand'`). |
| `conf` | hand, head | number | Tracker confidence. Default 1. A hand with `conf` below the source's `MinConfidence` (default 0.3) is treated as not tracked. For the head the value is clamped to 0..1. |
| `space` | hand | string | `"ue_world_cm"` (default) or `"mp_world_m"`. Any other value skips the hand. |
| `lm` | hand | 21 x [x,y,z] | Landmarks in MediaPipe order. Exactly 21 entries, each exactly three finite numbers; otherwise the hand is skipped. |
| `wrist_ue_cm` | hand | [x,y,z], optional | If present (3 numbers), the whole hand is shifted so that landmark 0 sits at this UE world position (cm), after the space conversion. |
| `head` | root | object, optional | Head pose and expression (below). |

**Spaces.**

* `ue_world_cm`: positions are already in Unreal world coordinates, centimetres (x forward, y right, z up). They are used as they are.
* `mp_world_m`: MediaPipe *world landmarks*: metres, origin at the hand centre, x to the right in the image, y down, z away from the camera. The receiver converts a point `(X, Y, Z)` to UE as `(x, y, z) = (100 Z, 100 X, -100 Y)`; if `bMirrorLateral` is set it then negates `y`; then it applies the source's `SpaceToWorld` transform; then it applies `wrist_ue_cm` if present. Prefer `ue_world_cm` in production.

**The head object.**

| Field | Type | Meaning |
|---|---|---|
| `pos_ue_cm` | [x,y,z], required | Head origin in UE world cm. |
| `quat_ue` | [x,y,z,w], required | Head rotation, UE world. The head frame is x = nose direction, y = the subject's right, z = up. The receiver normalises the quaternion; a zero quaternion or any non-finite value rejects the head. |
| `conf` | number | 0..1, default 1. Below `MinConfidence` the head counts as not tracked. |
| `scale` | number, optional | Head size relative to the canonical face model (1 = canonical, eye span 6.3 cm). Used if finite and > 0, capped at 5. |
| `bs` | array of 51 numbers, or an object | The 51 blendshape scores, 0..1 (clamped), in the order below. As an object: `{name: value}` using the names below, missing names are 0. An array of any other length is ignored. |
| `t` | number | Sender time; falls back to the root `t`. |

Blendshape order (ARKit names, the **subject's** left / right; MediaPipe's list without `_neutral`): `browDownLeft browDownRight browInnerUp browOuterUpLeft browOuterUpRight cheekPuff cheekSquintLeft cheekSquintRight eyeBlinkLeft eyeBlinkRight eyeLookDownLeft eyeLookDownRight eyeLookInLeft eyeLookInRight eyeLookOutLeft eyeLookOutRight eyeLookUpLeft eyeLookUpRight eyeSquintLeft eyeSquintRight eyeWideLeft eyeWideRight jawForward jawLeft jawOpen jawRight mouthClose mouthDimpleLeft mouthDimpleRight mouthFrownLeft mouthFrownRight mouthFunnel mouthLeft mouthLowerDownLeft mouthLowerDownRight mouthPressLeft mouthPressRight mouthPucker mouthRight mouthRollLower mouthRollUpper mouthShrugLower mouthShrugUpper mouthSmileLeft mouthSmileRight mouthStretchLeft mouthStretchRight mouthUpperUpLeft mouthUpperUpRight noseSneerLeft noseSneerRight`.

**Head-only datagrams.** A datagram whose root has a valid `head` object and no `hand` field and no `hands` array is a head-only datagram: it is accepted and counted in `HeadPacketsReceived`. A `head` next to hands does not change how the hands are parsed.

**Receiver behaviour.**

* A datagram is accepted when at least one hand parses or the head is valid; it is then counted in `PacketsReceived`. Otherwise it is counted in `PacketsRejected` and the first five reasons are logged (`LogSoftPoke`).
* A hand or the head is lost when no datagram updated it for `TimeoutSeconds` (default 0.25 s). To release a hand, stop sending it (or send `conf` 0).
* Send rates are free: the rig filters once per new sample and the limiter runs every tick on the held joints. 30 Hz webcam data in a 90 Hz game is fine.
* A hand packet is about 600 bytes; two hands about 1.2 KB.
* There is no authentication. Keep the default bind address `127.0.0.1` unless the sender runs on another machine.

### Minimal sender (sketch)

```python
import json, socket, time

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
def send(landmarks_ue_cm, hand="right", conf=1.0):
    # landmarks_ue_cm: 21 [x, y, z] lists in Unreal world cm, MediaPipe order
    pkt = {"v": 1, "t": time.time(), "hand": hand, "conf": conf,
           "space": "ue_world_cm", "lm": landmarks_ue_cm}
    sock.sendto(json.dumps(pkt, separators=(",", ":")).encode("utf-8"), ("127.0.0.1", 9763))
```

This sketch follows the specification above; it is an example and is not part of the plugin.

