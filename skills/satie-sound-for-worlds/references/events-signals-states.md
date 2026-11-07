# Events, signals and states

How the game tells the sound what is happening, in the shape the middleware that ships games uses. Each rule names the satie-runtime call that implements it (satie-runtime/three, 0.4.5); the Unity and Unreal line says what the same thing is called there, so a developer who knows one recognises the other. Until satie-runtime 0.4.5 is on npm, these calls (`setHostSurface`, `hostEvent`, `signalTarget`, `bindTransient`, the edit log) come from the tarball at `https://www.satie.world/runtime/0.4.5/satie-runtime.tgz`; the install line above names the published version.

**One surface map, set on the emitting host before the event fires.** Footsteps, landings and drops from one object read one surface, the material under it now, not a surface guessed per sound. Resolve it at the contact point (the ground material under the planted foot, the wheel, the body) and set it on the host before the event; the engine reads a host's surface at the moment the event fires, so an event fired first and a surface set after plays the old bank. Declare the surfaces on the event in the manifest, preferably by their canonical names (concrete, stone, flagstone, gravel, sand, dirt, grass, mud, wood, metal, carpet, tile, water, snow, glass, plastic, fabric, or `other:<name>`): a canonical name shares its bank across projects and keys hosts.json the same way in every engine; free text still works and the runtime matches the exact text you declared.
- satie-runtime: `audio.setHostSurface(id, surface)` then `audio.hostEvent(id, 'step')` (fires at the host's geometry centre, with its surface as the event text and the host as the body); `audio.hostEvent(id, 'land')` and `audio.hostEvent(id, 'drop')` read the same map; `{ text }` overrides for one fire. `audio.event(name, point, surface)` remains for a contact point that is not a host.
- Unity: the physic material hit by the foot raycast, read into a switch before `PostEvent`. Unreal: the physical material from the hit result, set as the surface switch on the component before the AkEvent posts (Lyra's footstep helper does exactly this).

**The player's own sounds bind to the listener, not the body.** Steps, breath, cloth, the click of their own weapon play where the ears are, whatever camera the game renders from; in third person the body is metres from the listener and a step placed there pans and attenuates wrongly. Mark those events `by: "<player id>"` in the manifest: Satie writes them as hostless `on` voices that answer wherever the event happens, and the runtime places the player's steps at the listener. Sounds the player's THING makes (the engine of the car they drive) stay `at` it.
- satie-runtime: `audio.bindListener(player, { offset: [0, 1.65, 0], orientation: camera })` (or `bindListener(camera)`); `audio.step(surface)` fires the player's step at the listener; hostless `on` voices need no bind and must carry no `within`.
- Unity: the player's own emitters on the AudioListener's object. Unreal: post the local player's self-sounds on the listener component, not the pawn mesh (Lyra posts them "at listener").

**Transient per-event hosts release themselves.** A thrown bottle, a shell casing, a spawned spark exists for its sound and then must go: a host that is never unbound leaks a slot and keeps sounding where it died. Bind it when it spawns, fire from it, and let it release when nothing at it is playing or waiting to play; keep the object until the release callback, or hand it a detached object.
- satie-runtime: `audio.bindTransient(id, object, { graceS: 0.5, onRelease })`: bound like any host, unbound on its own once no voice at it plays or is pending (after `graceS`, so a sound fired the next frame counts); `audio.releaseTransient(id)` releases now. Long-lived objects keep `bind` and `unbind` on despawn.
- Unity: an AudioSource on a pooled object returned to the pool on `!isPlaying`. Unreal: a one-shot AkComponent with "stop when owner destroyed" and auto-destroy on end (Lyra's transient emitter).

**Signals carry a target; the runtime smooths and publishes on change.** A continuous control (speed, engine load, tension, crowd) is set every frame by the game but should reach the sound as a target value, eased engine-side so a physics tick does not zipper the gain, and published only when it moved enough to matter, so a hundred frames of 0.5001 cost nothing. Defaults: smoothing time 0.1 s (exponential), send threshold 1 percent of the signal's declared range (0.01 when it declares none). Use a shorter smoothing for a control the player feels (throttle, 0.05 s) and a longer one for a mood (tension, 0.5 s). Declare min, max and unit on every signal: the threshold comes from the range, and an out-of-range value is a boundary error, not a clamp.
- satie-runtime: `audio.signalTarget('engine_load', load, { smoothS: 0.1, threshold })` each frame or on change; `audio.update(camera)` does the easing and publishes; `audio.signal(name, value)` publishes at once and cancels the easing. An event reads the signals that gate it at the moment it fires, so set a signal the event changes AFTER firing it.
- Unity: a float parameter on the mixer or FMOD/Wwise RTPC with interpolation time, set from a damped value. Unreal: an RTPC with the project's interpolation (Lyra's RTPC helper keeps target, smoothing time and a change threshold before `SetRTPCValue`).

**States are a separate discrete group, with terminal states.** A state is a mode the game is in (calm, alert, combat; indoors, underwater; alive, dead), not a number and not an event: it changes rarely, it gates whole groups of voices (`zone`), and it has no smoothing. Keep each state group orthogonal (power is not damage), make every member of a group set the same controls, and name the terminal ones (destroyed, dead, extracted): a terminal state is the last the host enters before it is unbound, and nothing after it may revive a layer. The scene mode is one global group; each object's own groups live on its host bridge.
- satie-runtime: `audio.state('alert')` and `audio.state(null)` for the scene mode (declared in `states`, answered by `zone` voices; `state(name, isolation)` softens the cut); per-object groups through `HostAudioBridge({ states: { power: {...}, damage: {...} } })` with `machine.state('damage', 'venting')`; on a terminal state set it, then `dispose()` the bridge and `unbind` the host when the object leaves.
- Unity: a snapshot or state parameter on the mixer per group. Unreal: a Wwise State group per axis (Lyra keeps game-phase states apart from the RTPCs, and never drives a state from a float).

**A short three.js example** (the glue after `SatieScene.load` and `bindAll`):

```ts
audio.bindListener(player, { offset: [0, 1.65, 0], orientation: camera });

// the character: one surface map, read before each contact
feet.onContact(contact => {                       // your resolved foot contact (FootContactTracker or physics)
  audio.setHostSurface('player', groundMaterialAt(contact.point));
  audio.step(audio.hostSurface('player')!);       // the player's own step: at the listener
});
crate.onLand(point => { audio.setHostSurface('crate', groundMaterialAt(point)); audio.hostEvent('crate', 'drop'); });

// a thing that exists for its sound
function throwBottle(mesh: Object3D) {
  const id = `bottle_${mesh.id}`;
  audio.bindTransient(id, mesh, { onRelease: () => scene.remove(mesh) });
  mesh.userData.onBreak = (point: Vector3) => { audio.setHostSurface(id, groundMaterialAt(point)); audio.hostEvent(id, 'shatter'); };
}

// signals as targets; states as modes
function frame(dt: number) {
  audio.signalTarget('engine_load', car.load);                      // 0.1 s, 1 percent of range
  audio.signalTarget('tension', director.tension, { smoothS: 0.5 });
  if (director.phase !== lastPhase) { audio.state(director.phase); lastPhase = director.phase; }
  audio.update(camera);                                             // eases, publishes, releases
}
```

## The host map: public/satie/hosts.json

`satie_integrate` returns `hostMap` (also as the file `public/satie/hosts.json` in its bundle), schema `satie.hosts/v1`: top level `{ v, scene, contractId, updatedAt, hosts, removed?, report }`; one entry per host `{ host, object, clipPaths, surfaces, contractId, updatedAt, status? }` where `object` is the engine object, prefab name or path the host is bound to (it starts as the host id; correct it to the real path when they differ), `clipPaths` the samples.json keys it plays, and `surfaces` its banks keyed by canonical surface name (`{ gravel: [clipPath, ...], "other:bone": [...] }`). Hostless voices (the player's own steps, a chime) sit under `"(unhosted)"`.

Save it next to scene.satie exactly as returned (Satie never writes your files; you write it like samples.json) and pass the file back as `hostMap` on the next `satie_integrate`. A re-run starts from it: a host in the map but not in the new manifest is reported as "renamed or removed, was <object>" in `report` and listed under `removed`, a new host is `status: "new"`, a host whose clips or banks moved is `"changed"`. Show the report to the developer; a renamed mesh is fixed by renaming the host in the manifest (or correcting `object`), never by leaving its sounds unbound. New team members read it to see which object each sound is bound to.

## The developer's decisions: .satie/authoring/edits.jsonl

While the developer mixes in their own game, the `satie-runtime/vite` plugin writes every decision as one line of `satie.edits/v1` to `.satie/authoring/edits.jsonl` (contract: packages/satie-runtime/EDITS.md): `{ v, at, scene, voice, host, op, value, by }` with ops level, mute, unmute, solo, unsolo, move, swap, trim, remove, add, set, label, note. Read the file, or `GET http://localhost:<port>/__satie/edits?scene=public/satie/scene.satie` while the dev server runs (loopback and same-origin only), then fold it into the script with `satie_apply_edits { script, edits }` (free) instead of re-composing: the developer's levels, mutes, moves and swaps become authored lines, one summary line per edit. Keep `.satie/` in .gitignore; the folder is private.
