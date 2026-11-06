# Behavior discovery and the host lifecycle

## Discover behavior in host code

Discover behavior in actual host code, not screenshots alone. Inventory every sound-producing object, its physical contact callbacks and continuous controls, all orthogonal state groups (power, damage, occupancy, immersion, etc.), and its despawn/reset lifecycle. scene.behaviorContract records the source file/symbol references, unresolved semantics, exact source ids, event authority, all audible/silent states, listener anchor and measurable latency/distance cases. A missing or silent state needs a concrete reason; do not assume a parked car is off or visible smoke means fire. Keep each actual moving source spatially attached to its own object.

## Host lifecycle

Publish listener/object transforms before starting. A missing host or excess class-count voice stays silent until its own slot exists; unbind despawned objects. Keep manifest clip keys unchanged (including Audio/). Load first, unlock/resume AudioContext inside the input gesture, and handle re-entry/tab suspension separately from asset readiness. If audio is essential, do not unlock gameplay while it is still loading.
