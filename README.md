# Phlosion VFX

Phlosion VFX is the reusable authored-effects library for Phlosion Engine. It
owns effect simulation, renderer-neutral authored batches, reusable renderer
submission, preview controllers, and an optional stock effect catalog.

It intentionally does not own game rules, Pokemon instances, battle events,
board state, proprietary source captures, extracted assets, or private cooked
content.

## Dependency direction

```text
Game -> Phlosion VFX -> Phlosion Engine
Game ----------------> Phlosion Engine
```

Phlosion VFX never depends on a game repository. Games bind an effect to their
own events, actors, sockets, and asset IDs.

## Build

Requirements match Phlosion Engine: CMake, a C++20 compiler, vcpkg through
`VCPKG_ROOT`, and the Vulkan SDK. A sibling `../PhlosionEngine` checkout is
used automatically. Set `PHLOSION_VFX_ENGINE_SOURCE_DIR` to an empty value to
exercise the exact pinned Git dependency instead.

```powershell
cmake --preset vs2026
cmake --build --preset debug
ctest --preset debug
```

Consumers can add the repository as a CMake subdirectory and link only the
surface they need:

```cmake
add_subdirectory(path/to/PhlosionVFX)
target_link_libraries(MyGame PRIVATE PhlosionVFX::Runtime)
```

Available targets are:

- `PhlosionVFX::Core`
- `PhlosionVFX::Runtime`
- `PhlosionVFX::Preview`
- `PhlosionVFX::PhlosionVFX`

## Content boundary

The current compatibility effects retain asset paths supplied by consuming
projects while the `.phlo` VFX-prefab pipeline is implemented. No referenced
asset payload is stored in this repository. Original and redistributable stock
content may be added later; proprietary source evidence and derived payloads
remain in a private asset depot.
