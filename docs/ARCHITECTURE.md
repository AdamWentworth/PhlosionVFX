# Phlosion VFX Architecture

Status: Active

Phlosion VFX sits between engine rendering infrastructure and game-specific
presentation.

## Repository ownership

Phlosion Engine owns particle primitives, render backends, resource loading,
materials, shader interfaces, and the generic preview host.

Phlosion VFX owns reusable effect simulation, authored draw-pass schemas,
renderer submission adapters, reusable preview controllers, and optional
original stock effects.

Games own event-to-effect mappings, actor and socket binding, gameplay
configuration, and game-specific presentation. Pokemon-specific types and
board concepts are forbidden dependencies here.

Private asset depots own proprietary captures, extracted source payloads,
uncleared derived content, and cooked copies of that content.

## Runtime resources

An assembled reusable effect is represented as a `.phlo` with root type
`VfxPrefab`. It may depend on `.phmesh`, `.phmat`, KTX2 textures, backend
shader payloads, curves, and emitter definitions. Game code addresses the
effect by stable asset ID rather than a physical source path.

The existing path-driven authored effects are a compatibility boundary. New
generic APIs must accept asset IDs or caller-supplied resolvers, and source-
specific defaults must migrate into game configuration or private cook
recipes.
