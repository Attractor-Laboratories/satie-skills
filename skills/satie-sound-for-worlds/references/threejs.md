# Three.js

## Three.js maintained adapter

Install: npm install satie-runtime@0.4.4



Import SatieScene and manifestFrom from satie-runtime/three. Never copy an implementation of the bridge into the project. For dense scenes, import SatieEngine from satie-runtime and pass an engine configured with loopVoiceBudget: { maxRealVoices: 32, silenceThresholdDb: -60, hysteresisDb: 3 } to the scene loader. This is an opt-in rendering budget for continuous voices, not a limit on discovered objects or their state. Inspect engine.loopVoiceReport() and verify audioContext.currentTime progresses with the host monotonic clock under the worst scene load. A smooth visual frame rate does not prove the audio renderer is keeping up. Stop timing/listening validation if the clocks diverge; do not wait for three slow audio seconds while the game runs for twenty real seconds. Choose a budget on the target device and preserve explicit virtualization evidence. Interactive triggers bypass this loop budget.

Release listening loop: call satie_listen on isolated suspect takes AND recorded gameplay WAVs (up to 30s each). Include the timed route and intended foreground/background hierarchy. Reject recognizable but harsh synthetic imitations; do not turn them down and call them fixed. Listen at spawn, nearest accessible sources, far points, boundary crossings, footsteps and collisions. A healthy runtime trace does not establish audible quality. satie_realize.reviewRequired and satie_integrate needs_review mean missing or failed listening evidence, never approval. Preserve local exact-hash rejections and concrete reasons, feed them into the next compose brief, and call satie_review_release with the actual asset hashes and route measurements. Human and agent judgments stay distinct; contribution still needs separate consent.

External extended sources: a mesh origin is not a river. Use satie.bindLine(id, object, localPoints, width) for a measured strip and a line voice. The optional CharacterMotor (satie-runtime or /integration) accepts collision bounds/velocities and material/depth queries; games with physics already should use their resolved contacts. Update listener first; use satie.event('step', resolvedFootPosition, surface) when contact geometry is available. satie.step(surface) remains the listener-position convenience. A `within` radius is measured from the voice's own position: give one only to an `on` voice that is `at` a host or placed. A hostless `on` voice (a lantern lighting, a win chime) and player-local steps answer the event wherever it happens and must not carry one; a radius there was measured from the world's origin. Immersion, grounded movement and collision events need real host state and prepared sound banks. Keep a fixed levelAnchorDb selected from route auditions; never boost the scene based on the spawn pose.

Expose the maintained sound editor when the developer wants to mix: mountMixer(audio, { scene, camera, renderer, script, samples, levelFaders: true, mixGroups, context: () => audio.authoringContext() }) from satie-runtime/three (the panel with its world sound layer) and the satie-runtime/vite plugin. Group stable source IDs into meaningful controls such as footsteps, engines, tires, birds and environmental textures. The automation-preserving level mode changes SPL; the default volume override would remove a volume-follow binding. Verify zero throttle/power-off still silences propulsion after a fader edit, and Save survives contract-checked reload. Distinguish authored off-screen ecology and diffuse city wash from observed game objects; a background cannot substitute for the actual nearby vehicles.

Mix from the chosen listener perspective. A vehicle-bound listener can put its own engine centimetres away while traffic is tens of metres away; switching to a chase camera changes that balance and the overall level. Select a fixed reference from quiet walking, a real nearby traffic pass, coasting and acceleration routes. Check NPC engines against their own tire layers, and occupied-car versus environment contribution. Preserve real source positions and identical vehicle response rules. Correct relative source levels as well as the overall reference; a global boost alone cannot fix masking. Report actual recorded output separately from modeled received SPL, and label any pre-master/direct-path meter scope. Higher or more distant camera modes should retain their actual geometry unless the brief explicitly calls for a presentation compensation.

Contact Foley craft: use distinct edited performances per material, preserve heel/weight-transfer/scuff and release, and audition the repeated gameplay sequence as well as isolated takes. A two-take no-repeat bank deterministically alternates A-B-A-B; pitch jitter cannot repair bad recordings. Template triggers honor variants N (minimum two for compatibility) in both realization cost and playback. Larger banks exclude the most recent ceil(N/2) performances, capped at N-1. Six to eight usable contacts are a starting authoring choice, not a quality guarantee. Never loop a recording of someone walking as the player's gait. Use physical/animation contacts for visible feet. A distance fallback is explicitly labelled and cannot claim measured contact or animation sync. Pace/impact should drive authored dynamics; place contacts at the feet and judge foreground level over the quietest ambience. Trim and measure take leading silence: firing an event immediately does not make a 400ms silent lead-in responsive.

Material routing must reach the actual selected buffer. Declaring event surfaces or forwarding a surface string is insufficient if every contact still plays one hard-floor bank. Resolve material at each planted foot's contact point, respecting finite mesh boundaries and overlaid paving, then preload a distinct recorded/generated bank for each declared surface through an on-step template containing {text}. Verify real boundary-crossing contacts against the selected buffer key and asset hash, not only the event label. Keep any unavailable-material fallback explicit. Check sibling loudness after the runtime normalizer, and judge timbre and shoe consistency through listening. An overall positive scene judgment does not approve every material bank.

Water needs several phenomena: the river's extended flow, individual entry/foot displacement/release, and a quiet body-local movement texture controlled by actual speed and immersion with a short settling release. Standing still or pushing a wall must not keep displacing water. Prepare natural recordings with distinct regions; do not truncate every take at the same arbitrary quarter-second. The repository's scripts/prepare-foley-bank.mts reads an explicit source/hash/license/region recipe and writes independent edited WAVs, provenance and an audition montage; it never generates, publishes or approves audio. Check sample counts, edit bounds and silence durations numerically. Listening models are fallible at counting segments; retain conflicting judgments, concrete source rejections and before/after route PCM rather than treating a model pass as human approval.

1. manifestFrom(root, { unitsPerMeter: 1 }) measures named meshes in metres. Mark sound-relevant objects with stable names; userData.satieId overrides identity, userData.satieMotion='moves' marks dynamic hosts, userData.occludes=false marks porous objects. Add known rooms, events, states and signals (with units) to the manifest. Screenshots supplement measured geometry.
2. Load the returned scene.contract.json. Call SatieScene.load('/satie/scene.satie', '/satie/samples.json', { contract, seed: 7, engine: new SatieEngine({ loopVoiceBudget: { maxRealVoices: 32 } }) }). The optional engine budget retains logical source identity/state while rendering the most audible continuous sources. It rejects failed fetches/decodes and verifies hashes when present.
3. Bind each host with audio.bind(id, object), or audio.bindAll(root, optionalNameMap). Missing/duplicate objects fail loudly. A sound plays from the centre of the bound object's geometry (merged or instanced meshes included). EVERY object a sound is `at` must be placed from the start: bind it, or call setHostPosition every frame while it exists; a creature or vehicle placed only when an event happens has no position until then (audio.bindWarnings lists the ones you missed). An event's signals are read at the moment it fires: set a signal the event changes AFTER firing it, or a sound gated by it plays at zero. Choose the requested ears explicitly: audio.bindListener(camera) uses the actual view position and orientation; audio.bindListener(player, { offset: [0, 1.65, 0], orientation: camera }) instead uses character position with camera orientation. Reassign on possession only if that is the chosen policy. Verify actual world position AND forward direction across every camera mode, orbit and enter/exit transition. Call audio.update(camera) after physics and world transforms each frame. All adapter positions use engine units; conversion to metres happens once.
4. Wait for load completion, enable Start, and call audio.start() directly in the click handler. Never await network loading before requesting audio activation.
5. Forward authoritative callbacks through HostAudioBridge (example below). audio.event/signal remain available; a bare event call has no causal timing evidence. Use per-object orthogonal state groups for power/damage/occupancy; audio.state is the global scene mode, not each object's state machine. Undeclared names, missing surfaces and out-of-range controls fail at the boundary.
6. Call audio.unbind(id) on entity removal and audio.dispose() on scene unload. Only registered objects produce bindings; an object can remain silent by design.
7. audio.beginCapture()/endCapture() produces local satie.playthrough/v2 evidence: revision, seed, assigned listener/hosts/source identities, signal and orthogonal object states, timestamped physical actions, scheduled starts plus PCM onset estimates, selected takes, resolved emitter distances and modeled levels. Run satie_audit_playthrough with the current script and scene. It is not a microphone recording and is never automatically uploaded for training.

## Causal host bridge (candidate API)

The same bridge applies to machines, creatures, doors, tools, characters and vehicles. Names below are project choices, not built-in car assumptions. The host supplies actual physics and state transitions; Satie owns validation, deduplication, audio response and evidence.

```ts
import { HostAudioBridge } from 'satie-runtime/integration';

const machine = new HostAudioBridge(audio, {
  bodyId: 'machine_1',
  events: { impact: { event: 'machine_impact', sourceId: 'machine_impact_sound', authority: 'physics-impact' } },
  controls: {
    rotation: { signal: 'machine_rpm', unit: 'rpm', min: 0, max: 9000 },
    motor: { signal: 'machine_on', unit: 'normalized', min: 0, max: 1 },
    cooling: { signal: 'machine_cooling', unit: 'normalized', min: 0, max: 1 },
  },
  states: {
    power: { states: { off: { motor: 0 }, running: { motor: 1 } } },
    damage: { states: { intact: { cooling: 0 }, venting: { cooling: 1 } } },
  },
  onDetach: { rotation: 0, motor: 0, cooling: 0 },
});

// Inside the resolved physics callback, in the same host monotonic clock as audio:
machine.event({ id: contact.occurrenceId, channel: 'impact',
  position: contact.point, occurredAt: contact.monotonicSeconds });
// After authoritative simulation advances:
machine.control('rotation', body.rpm);
machine.state('power', body.powerState);
machine.state('damage', body.damageState);
// On removal: machine.dispose(); audio.unbind('machine_1');
```

Every state in one group must reset the same set of controls. Independent groups cannot own the same control. A loop gated by a damage signal can therefore remain audible while the motor is off, without treating all objects as one global state. Declare these states and the host-code symbols that define them in behaviorContract.inventory. Give each parsed voice its exact `source_id` and `at` host. Unknown states are integration failures, never silent fallbacks.

For validation, hold listener/source pose and other controls fixed while varying one control; then approach/retreat with source controls and state held fixed. An accelerating source changing loudness is not proof of distance attenuation. Match the source waveform window too: a louder wave or gust can confound separate near/far clips even at fixed controls. For stable loops, compare complete native playback periods; evaluate direction against actual finite source geometry rather than its host origin. Exercise every inventoried state, including intended silence. Compare physical occurrence to source-specific onset timing; PCM onset estimates remain estimates until measured at output. Quiet preparation noise can precede the intended contact by hundreds of milliseconds, so review the actual cue rather than trusting the first energy crossing. Check sibling take levels after the runtime's real normalization and peak protection; a gain cap may leave a noise-like take effectively inaudible. Reject or replace that take, rather than weakening the gain guard. Keep exact source and edit hashes, and submit actual route recordings for listening and repetition checks.

Run the returned scripts/satie-vendor.mjs before release to pin asset bytes and use local samples. Keep the lockfile and runtime version. Gameplay requires neither Satie credentials nor generation calls.

