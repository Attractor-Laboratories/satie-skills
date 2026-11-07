# Behavior discovery and the host lifecycle

## Discover behavior in host code

Discover behavior in actual host code, not screenshots alone. Inventory every sound-producing object, its physical contact callbacks and continuous controls, all orthogonal state groups (power, damage, occupancy, immersion, etc.), and its despawn/reset lifecycle. scene.behaviorContract records the source file/symbol references, unresolved semantics, exact source ids, event authority, all audible/silent states, listener anchor and measurable latency/distance cases. A missing or silent state needs a concrete reason; do not assume a parked car is off or visible smoke means fire. Keep each actual moving source spatially attached to its own object.

## Filling the contract

Say what the game does; never relabel a sound to pass a rule. The inspector reads these fields:

- `kind`: `contact` for feet and wheels meeting a surface, `impact` for one thing striking another once (a knock, a slam, a dropped cup), `continuous` for a mechanism or texture that runs while its object is in a state (fire, engine, clock), `state` for a sound that follows a transition (door opens, lamp lit: give it the `event`) or a signal range (`stateCases`), `environment` for weather, room tone and the distant world.
- `authority`: what decides the moment. `physics-contact`, `physics-impact` and `animation-contact` are resolved collisions and foot-down markers. `distance-fallback` is metres travelled, one step per stride, for games without a physics engine (the runtime's CharacterMotor, or your own counter). `state-transition` is the object changing state; `game-state` is the script deciding (a scripted knock, a cutscene). A timer or the render loop is never an authority for a contact.
- `host`: the object; an `environment` behavior may name a room instead, and the script answers it with `in <room>` or a `bed`.
- `spatial`: the near and far levels the playthrough audit compares. Not needed for the listener's own sounds (the player's steps) or for `environment`.
- `signals`: needed for a `continuous` sound only when something drives it; a fire that burns while `lit` is gated by its inventory state alone.
- Inventory: a state silent at rest may list transition sounds next to its `silentReason` (a closed door that gets knocked on). Transition sounds and the weather need not appear in any state; running sounds must.

The contract is optional. Without it the scene composes and gameplay audio stays unverified until `satie_audit_playthrough`; an honest partial contract beats a complete invented one.

## Host lifecycle

Publish listener/object transforms before starting. A missing host or excess class-count voice stays silent until its own slot exists; unbind despawned objects. Keep manifest clip keys unchanged (including Audio/). Load first, unlock/resume AudioContext inside the input gesture, and handle re-entry/tab suspension separately from asset readiness. If audio is essential, do not unlock gameplay while it is still loading.
