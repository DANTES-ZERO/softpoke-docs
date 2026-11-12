# Touch events

> SoftPoke 1.0.0 | [Index](Index.md) | [Component reference](ComponentReference.md#softpoketouchsensor)

Touch events are the "nerve" of a character. While you touch the character with the hand rig, the plugin reports **where** (a named body part), **how hard**, **for how long** and **what kind of touch** (poke, press, grab, squeeze, stroke, knead, tap) as small JSON messages. A program of yours (a local LLM, a voice assistant, a game script) reads the messages and lets the character react.

* The messages leave the game over a **WebSocket** (`ws://127.0.0.1:9764`) and/or an HTTP **webhook** (POST). Both are **off by default**. Nothing is written to disk.
* There is **no Blueprint interface** for this feature on purpose: no events, no callable functions. The settings are plain properties.
* The plugin only describes contact. What the character says or does is your decision and your responsibility. SoftPoke is for adult characters only.

## Quick start

1. Add the component **SoftPokeTouchSensor** to the character actor, next to the `SoftPokeHandRig` (Add Component in the Details panel or the Blueprint editor).
2. **Project Settings > Plugins > SoftPoke > Touch events**: tick **Touch Web Socket**.
3. Run the game, touch the character, and in a console:

```
python <Project>/Plugins/SoftPoke/Tools/TouchEvents/receive.py
```

```
   12.43 touch_start  #7   right cheek_left         poke     0.08  right hand starts poking the left cheek
   12.63 touch_update #7   right cheek_left         press    0.42  right hand pressing the left cheek firmly for 0.4 s
   14.55 touch_end    #7   right cheek_left         press    0.00  right hand stopped pressing the left cheek after 2.1 s (peak: firm)
```

`receive.py` needs only Python 3.8+. `ws_client.py` is the same with the `websockets` package, `webhook_server.py` receives the webhook, `llm_example.py` shows how to feed a local LLM (see [Using it with an LLM](#using-it-with-an-llm)).

A sensor with both transports off (the default) does no work at all: the rig extracts no contact rows and nothing is formatted. The work starts when a transport is switched on or `bLogEvents` / `softpoke.Touch.Log` is on.

Without a rig that tracks a hand there is nothing to report: the events come from the contact of the hand the limiter ("physics hand") holds on the skin. They follow what the solver shows, not the raw tracker.

## What is sent

One event is one UTF-8 JSON object (WebSocket text frame, or the body of a POST). The keys are English and have units. **Receivers must ignore keys they do not know.**

| Event | When |
|---|---|
| `hello` | first message to a new WebSocket client: schema, clock, rates |
| `touch_start` | a hand has been on a part for two ticks (a single noisy tick is ignored) |
| `touch_update` | every `1 / UpdateHz` seconds (default 5 Hz) while the touch lasts, and at once when the gesture changes (at most 10 Hz) |
| `touch_end` | the hand let go, moved on to another part, or tracking stopped. Every `touch_start` gets exactly one `touch_end` |
| `state` | every `1 / StateHz` seconds (default 1 Hz) while at least one touch is active: all touches in one message |

| Key | Events | Meaning |
|---|---|---|
| `v` | all | schema version, `1` |
| `t` | all | seconds since the sensor started (monotonic) |
| `seq` | all | `+1` per event of any kind. A gap means events were dropped (update / state first, see [Delivery](#delivery)) or filtered out for a webhook |
| `event`, `entity` | all | event name; `EntityName` of the sensor (actor label by default) |
| `touch_id` | touch | a number that stays the same while the same hand stays on the same part; a new number after letting go or moving to another part. Never reused |
| `hand` | touch | `"left"` / `"right"`: the hand that touches |
| `part`, `part_group` | touch | part key and its group, see [Parts](#parts) |
| `bone`, `region` | touch | skeleton bone with the largest skin weight at the contact, and the baked region |
| `gesture` | touch | `poke`, `press`, `grab`, `squeeze`, `stroke`, `knead`, `tap` (provisional in `touch_start`) |
| `intensity` | touch | 0..1: depth of the dent divided by the depth the limiter allows at that place (see below) |
| `peak_intensity` | touch | the largest `intensity` so far |
| `depth_mm` | touch | depth of the deepest finger sample into the skin, smoothed |
| `duration_s` | touch | seconds since the first contact (0 in `touch_start`, the total in `touch_end`) |
| `fingers` | touch | the fingers in contact in the last 150 ms: `thumb index middle ring little palm` |
| `speed_cm_s` | touch | speed of the contact point along the skin (0 in `touch_start`) |
| `travel_cm`, `reason` | `touch_end` | path along the skin; `released`, `moved`, `hand_lost`, `shutdown`, `disabled` |
| `text` | touch, state | a short English description for an LLM; left out when `bIncludeText` is off |
| `touches` | `state` | the active touches: `touch_id hand part part_group gesture intensity duration_s fingers` |

```json
{"v":1,"t":12.431,"seq":104,"event":"touch_start","entity":"Sample","touch_id":7,"hand":"right","part":"cheek_left","part_group":"face","bone":"head","region":"face","gesture":"poke","intensity":0.08,"peak_intensity":0.08,"depth_mm":0.6,"duration_s":0.00,"fingers":["index"],"speed_cm_s":0.0,"text":"right hand starts poking the left cheek"}
{"v":1,"t":14.552,"seq":126,"event":"touch_end","entity":"Sample","touch_id":7,"hand":"right","part":"cheek_left","part_group":"face","bone":"head","region":"face","gesture":"press","intensity":0.00,"peak_intensity":0.55,"depth_mm":0.0,"duration_s":2.12,"fingers":["index","middle"],"speed_cm_s":0.0,"travel_cm":0.8,"reason":"released","text":"right hand stopped pressing the left cheek after 2.1 s (peak: firm)"}
```

The other two messages:

```json
{"v":1,"t":0.000,"seq":0,"event":"hello","entity":"Sample","schema":"softpoke.touch/1","server_time_unix":1794300000.123,"update_hz":5,"state_hz":1}
{"v":1,"t":15.000,"seq":131,"event":"state","entity":"Sample","touches":[{"touch_id":8,"hand":"left","part":"upper_arm_right","part_group":"arm","gesture":"squeeze","intensity":0.61,"duration_s":1.20,"fingers":["thumb","index","middle","ring"]}],"text":"left hand squeezing the right upper arm firmly for 1.2 s (thumb + 3 fingers)"}
```

**Connecting in the middle of a touch.** Nothing is replayed: a client that connects while a touch is open sees `touch_update` and `touch_end` events (and the touch in the next `state`) without a `touch_start`. A receiver should treat an unknown `touch_id` as "a touch that began before I connected"; `state` (every second, all open touches) is the way to resynchronise.

A JSON Schema is in `Tools/TouchEvents/touch_event.schema.json`.

**Intensity** is not a force. It is how deep the finger is in the skin relative to what the limiter allows there (a few millimetres over a bone, centimetres over a soft area). Once the limiter holds the hand back, a hard push and a firm push look alike (both near 1). The word in `text` is `gently` below 0.33, `firmly` below 0.66, `hard` above (with a small hysteresis).

**Timing.** The contact comes from the hand the limiter holds on the skin, which follows your tracked hand through the One-Euro filter of the rig. A finger that leaves the skin quickly therefore lets go of the touch a few tenths of a second later (the filter releases slowly near the skin). `touch_end.duration_s` counts until the last tick that still had contact; a quick poke is reported by its short intensity peak and is a `poke` even if the touch lasts 0.5 s.

## Parts

Parts are named neutrally and anatomically. The body is split by the **dominant skin-weight bone** at the contact (nearest baked skin anchors), the trunk further into front, back and side; the face is split into zones measured on the skin (the face is one bone).

| `part_group` | `part` |
|---|---|
| `face` | `forehead`, `temple_left/right`, `eye_left/right` (eye area), `nose`, `cheek_left/right`, `cheekbone_left/right`, `lips`, `mouth_corner_left/right`, `chin`, `jaw_left/right`, `ear_left/right`, `face` (fallback) |
| `head` | `head` (above the hairline / behind the ear, only where a region reaches) |
| `neck` | `neck` |
| `torso` | `shoulder_left/right`, `chest`, `abdomen`, `upper_back`, `lower_back`, `flank_left/right` (side of the torso), `pelvis`, `hip_left/right`, `gluteal_left/right` |
| `arm` | `upper_arm_left/right`, `forearm_left/right` |
| `hand` | `hand_left/right` (palm, back, all fingers) |
| `leg` | `thigh_left/right`, `lower_leg_left/right` |
| `foot` | `foot_left/right` |
| `other` | `other` (bone not classifiable) |

`left` / `right` are the character's left and right. The trunk is deliberately coarse (no finer split of the chest or the pelvis); you can suppress any part with `ExcludedParts`. The names and the text templates are fixed: nothing in the plugin builds free text.

Face zones are measured on the character's face (glabella and chin give the face height, `K = height / 12`), so they scale with the face. They are calibrated for adult proportions; a stylised face may need `BonePartOverrides` or an `ExcludedParts` entry. **Not covered:** places without a baked region (feet and scalp in the sample body) give no contact at all and so no event. On a rig with three spine bones the border between `abdomen` and `chest` sits higher than on a five-bone rig.

## Gestures

| Gesture | Rule (defaults; the main thresholds are `Tuning` properties) |
|---|---|
| `poke` | a short contact: the intensity rises and falls again (below 82 % of its peak) within 0.7 s |
| `tap` | several such peaks in one contact, or a poke on the same part within 0.7 s after the previous touch ended |
| `press` | a contact that stays: longer than 0.3 s with at least 0.3 s at 80 % of its peak intensity (or longer than 0.9 s) |
| `stroke` | the contact point moves along the skin faster than 2 cm/s (at least 1.5 cm in half a second) and nothing holds the part; the strength does not matter |
| `grab` | the part is held: the thumb and a finger push against each other (their contact normals point against each other), or the thumb and at least two more fingers touch the part together, or two different fingers / the palm clamp it from opposite sides (normals against each other, at least 2 cm apart); for at least 0.12 s |
| `squeeze` | a `grab` whose intensity reaches 0.5 or whose thumb-finger distance closes by 0.4 cm within 0.5 s |
| `knead` | a hold (or three or more fingers) whose intensity swings by at least 0.12 with two full cycles of 0.25 to 1.6 s within 2 s |

Priority: knead > squeeze > grab > stroke > poke/tap > press. Every change has to hold for 0.12 s, and each state has its own exit rule so a hand hovering around a threshold does not flip. `touch_start` carries a provisional gesture (`poke`, or `tap` for a series); `touch_end.gesture` is the gesture of the touch (a poke or tap stays one; a touch that was a `knead` for at least 1 s ends as `knead`, because the rhythm needs time to be seen; otherwise the one that lasted longest).

A hold is judged per hand over **all parts of one `part_group`**: when the thumb is on one part and the fingers on a neighbouring part of the same group (a fold of the abdomen that lies across the border of two parts, the thigh next to the hip), they are one touch on the part with the deepest contact. A thumb on the hand and fingers on the thigh stay two touches.

> **Experimental:** `grab`, `squeeze` and `knead` are verified with scripted contact rows (unit tests) and with a simulated hand in the editor that closes round a wrist, an upper arm, a thigh or a fold of the abdomen (thumb and three to four fingers in contact: always recognised as a hold, never as `press`). They have **not** been verified with a real tracked hand in VR. Which of the three it reports is a judgement from intensity and rhythm: a firm hold is a `squeeze`, a `knead` needs about 1.5 s of rhythmic change before it is seen. `poke`, `tap`, `press` and `stroke` were measured on a simulated hand only; on thin tissue over bone (the face) a `stroke` is often reported as `poke` or `press`.

What the labels can and cannot tell:

* The classifier sees only the contacts of the hand on one body part, not the pose of the hand. `grab`, `squeeze` and `knead` need contacts of the thumb and fingers (or the palm) on opposite sides of a part (a hand closed round an arm, a pinched fold). A hand that only presses with the palm, with one thumb or with the fingertips is a `press` (or a `stroke` when it moves), even if you meant to hold. Squeezing a thick thigh with one hand usually looks like a `press`.
* `intensity` saturates at the depth the limiter allows. On thin tissue over bone (forehead, chin) every firm touch is close to 1, so a poke, a tap and a press differ by their shape in time, not by strength; a `stroke` over such a place is recognised by its speed alone.
* The contact point is tracked on the mesh, so a swinging arm adds apparent motion.
* Take the labels as hints for a character, not as measurements.

## Settings

**Project Settings > Plugins > SoftPoke > Touch events** (`USoftPokeSettings`, `DefaultGame.ini`)

| Property | Default | Meaning |
|---|---|---|
| `bTouchWebSocket` | off | start the WebSocket server |
| `TouchWebSocketBindAddress` | `127.0.0.1` | address to listen on; anything else is reachable from the network and logs a warning |
| `TouchWebSocketPort` | `9764` | port (1024-65535) |
| `TouchAuthToken` | empty | optional secret of the WebSocket: `Authorization: Bearer <token>` or `?token=<token>` on the URL. Used for the WebSocket only |
| `TouchWebhookAuthToken` | empty | optional secret sent as `Authorization: Bearer <token>` with every webhook request. A different secret from the WebSocket one on purpose; the request is plain `http://`, so use it with a receiver on this PC or on a network you trust |
| `TouchAllowedOrigins` | empty | extra browser origins that may open the WebSocket |
| `TouchMaxClients` | 8 | simultaneous WebSocket clients |
| `bTouchWebhook` | off | send HTTP POSTs |
| `TouchWebhookUrls` | empty | up to four `http://host:port/path` URLs |
| `TouchWebhookEvents` | start, end | which events are posted: `start`, `update`, `end`, `state` |
| `bTouchAllowRemoteWebhook` | off | allow URLs that are not on this PC |
| `TouchWebhookTimeoutMs` | 1500 | time limit of one request |

Editing `DefaultGame.ini` by hand: a value that contains `//` must be written in double quotes, because an unquoted `//` starts a comment in an Unreal ini file (the Project Settings window writes the quotes for you). For example:

```ini
[/Script/SoftPokeRuntime.SoftPokeSettings]
bTouchWebSocket=True
TouchWebSocketPort=9764
bTouchWebhook=True
+TouchWebhookUrls="http://127.0.0.1:9765/touch"
+TouchWebhookEvents=start
+TouchWebhookEvents=end
+TouchAllowedOrigins="https://my-app.example"
```

An URL without the quotes is cut at `//` (the log says `webhook URL 'http:' is not a valid http:// URL`). The log line `settings read: ...` after the start of the session shows what was loaded.

**`SoftPokeTouchSensor`** (component, category `SoftPoke|Touch`)

| Property | Default | Meaning |
|---|---|---|
| `EntityName` | actor label | `entity` in the JSON |
| `UpdateHz` / `StateHz` | 5 / 1 | rates (`StateHz` 0 = no state events) |
| `bIncludeText` | on | add the `text` key |
| `Tuning` | see Gestures | thresholds of the tracker and the gesture rules |
| `ExcludedParts` | empty | part keys that never produce events |
| `BonePartOverrides` | empty | skeleton bone name to part key; replaces the rule for that bone and its child bones |
| `RigActor` | this actor | actor with the `SoftPokeHandRig`; empty = this actor, else the rig whose Body Actor is this actor |
| `bLogEvents` | off | log every event |

**Console variables:** `softpoke.Touch.Enable` (0 = kill switch for every sensor), `softpoke.Touch.WebSocket` and `softpoke.Touch.Webhook` (-1 follow the settings, 0 force off, 1 force on), `softpoke.Touch.UpdateHz` (0 = follow the component), `softpoke.Touch.Log`. Test aids that exist in every build **except Shipping** (they can override the security settings, so they are not in a shipped game): `softpoke.Touch.Token` / `WebhookToken` / `BindAddress` / `WebhookUrls` / `WebhookEvents` / `AllowRemoteWebhook` (replace the project setting of the same name), `softpoke.Touch.RowLog` (record the contact rows to `Saved/SoftPoke/touch_rows.jsonl`) and the commands `ProbePart`, `DumpParts` and `Flood`.
**Console commands:** `softpoke.Touch.Status` (counters, queue depths, drops, expired start/end events as `expired_high`, game-thread time; also written to `Saved/SoftPoke/touch_status.json`); in non-Shipping builds also `softpoke.Touch.ProbePart x y z` (world point to nearest surface node: region, bone, part), `softpoke.Touch.DumpParts file.csv` (every surface node with its bone and part, for tuning) and `softpoke.Touch.Flood <count> [block]` (synthetic events, to test slow receivers).

## Delivery

* The game thread never waits for a socket. Events are formatted on the game thread (a few microseconds) and put into a bounded queue; separate threads send them. Each webhook URL has its own thread, so a hanging receiver cannot delay the WebSocket or the other URLs.
* If a receiver is slow, old `touch_update` events are dropped first, then `state`; `touch_start` and `touch_end` are kept (a WebSocket client may be 512 events / 1 MB behind with them). A WebSocket client that is so far behind that a start or end would have to go is **disconnected** (`clients_dropped` in `softpoke.Touch.Status`) instead of getting a stream with a silent hole; it can reconnect and resynchronise with `state`. `seq` shows gaps where events were dropped.
* The WebSocket server is read only: whatever a client sends is ignored. At most 8 clients, one ping every 20 s, a client that is silent for 40 s is dropped. Pings, pongs and the final Close frame are sent between event frames, never inside one, and no event follows a Close.
* Webhooks: one POST per event with `Content-Type: application/json`, `X-SoftPoke-Event`, `X-SoftPoke-Seq` and optionally `Authorization: Bearer` (the webhook token). 2xx = delivered; 3xx (redirects are not followed) and 4xx except 408 / 429 = dropped (the receiver does not want it); timeouts, 408, 429 and 5xx are retried once, then with a back-off of 0.5, 1, 2, 4, 8 s. Updates and states older than 5 s are discarded, start/end after 60 s (counted as `expired_high` in the status). `https://` is not supported in 1.0.0 (use a local relay).
* Webhook delivery is **at least once**: after a timeout the same event can arrive twice. Use `seq` (unique, increasing, the same in the `X-SoftPoke-Seq` header) to drop duplicates. The events of one URL are sent one at a time in order (a retry delays the ones behind it).
* When the game stops (or you change the settings) the webhook threads first try to deliver the queued `touch_start` / `touch_end` events (the touches that were open end with the reason `shutdown`) for at most 0.35 s per URL on this PC (0.6 s for a remote one), in parallel, then give up. A receiver that hangs therefore delays the end of the game session by that time at most.
* While the game is paused no event is produced: a touch stays open until the game resumes; treat a long silence of an active touch as stale.

## Security

* Both transports are off by default. The WebSocket listens on `127.0.0.1` only. The port is opened without address re-use, so no other program can take it over.
* A browser page cannot read the stream: connections whose `Origin` is not `localhost` / `127.0.0.1` / `[::1]` (or listed in `TouchAllowedOrigins`) get `403`. Programs without an `Origin` header (Python, native clients) are accepted. With a token set, a wrong or missing token gets `401`.
* A webhook URL must point to this PC unless `bTouchAllowRemoteWebhook` is on (then the plugin warns in the log). A host name is resolved on the webhook's own thread (never on the game thread) when the first event is sent, and again every 5 s while it does not resolve.
* The events say where somebody touches a character. Treat them as private data, keep the default binding, and do not forward them without the player knowing.

## Using it with an LLM

`Tools/TouchEvents/llm_example.py` listens to the WebSocket and asks an OpenAI-compatible server (a local `llama.cpp` / LM Studio / Ollama endpoint) for one short in-character sentence. The rules that make it work:

* The model is far slower than the stream. Call it on `touch_start`, on a gesture change and on `touch_end`, **never per `touch_update`**; wait at least 1.5 s between calls. Do not read the socket from the thread that waits for the model (the example reads in a thread of its own and hands the model what happened since the last call; it never drops a `touch_end`, merges away a start that a later event has overtaken, and skips start events that are older than a few seconds).
* Send the `text` field as the user message; it is already a short English sentence.
* Put the character, the tone and the rules ("adult character, friendly, respect stop") in the system prompt. The plugin gives no judgement about consent or comfort.

## Cost

The sensor works after the hand rig on the game thread, from the contact samples the limiter has already computed (no extra penetration queries). Measured on an RTX 4090 PC in the sample scene (two hands pressing the cheeks, 60 fps and uncapped, two WebSocket clients and two webhook endpoints at 30 Hz updates):

* **Both transports off (the default):** the sensor does nothing, the rig extracts no contact rows. The frame was within +/-0.2 ms of the scene without the component (inside the noise of the measurement, the sign changes from run to run).
* **Transports on, two hands in contact:** the sensor tick takes about 12 microseconds on average (median 5, 99th percentile 30). The whole game-thread frame time was **0.2 to 0.35 ms longer** than without the component (this includes the rig's extraction of the contact rows and the four test receivers running on the same PC). Treat about 0.3 ms as the price of the feature while a hand touches the character.
* **One-time hitch:** the first tick after a transport starts (thread creation, socket binding) takes a few milliseconds, up to about 9 ms measured. It happens once, at the start of the session (or when you change the transport settings), not at the first touch.

Formatting and queuing one event costs a few microseconds; sending happens on other threads. `softpoke.Touch.Status` shows the average, 99th percentile and maximum of your own scene. Without a sensor component (or with `softpoke.Touch.Enable 0`) the rig does no extra work and the output is the same as without the feature.

## If nothing arrives

* **"waiting for the game"** in `receive.py`: the game is not running, the WebSocket is not ticked in the settings, or the port is taken (the log says `WebSocket server cannot listen on ...`; it retries every 5 s).
* **Connected, no touch events:** is a `SoftPokeHandRig` tracking a hand (Status text of the rig), is the sensor on the same character (`softpoke.Touch.Status` shows `"rig":true`), does the hand reach a baked region?
* **Windows firewall prompt:** only if the bind address is not `127.0.0.1`.
* **Wrong part:** `softpoke.Touch.DumpParts` and `ProbePart` show the part of every node; correct a bone with `BonePartOverrides`.
