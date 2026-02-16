a fork of shamus young's project frontier, with its dependencies bundled in.


This codebase is a C++/OpenGL world-simulation game organized as a mostly flat set of engine/gameplay modules under Terrain/, with a clear runtime pipeline:

    Boot + module init (Terrain/Main.cpp)

    Main loop with per-frame time budget (Main.cpp + Sdl.cpp)

    Procedural world model (World.cpp + Terraform.cpp)

    On-demand terrain paging/cache (Cache.cpp + CPage.cpp)

    Rolling scene grids for renderable chunks (Scene.cpp + CGrid.cpp)

    Render/environment/avatar/player systems (Render.cpp, Env.cpp, Avatar.cpp, Player.cpp)

1) Structure design (how it is shaped)
Core architectural layers

    Simulation model layer

        Terrain/World.h/.cpp: global World planet state (seed, region grid, noise tables, climate map).

        Terrain/Terraform.cpp: all high-level world generation passes (oceans, coasts, climate, rivers/lakes, zones, flora, colors).

    Data realization layer (lazy materialization)

        Terrain/CPage.cpp + Terrain/Cache.cpp: converts abstract world rules into concrete per-cell terrain/elevation/surface/color/tree data only when needed and stores pages.

    Spatial streaming/render prep layer

        Terrain/CGrid.cpp + Terrain/Scene.cpp: manages moving windows of terrain/forest/grass/brush/particle objects around player position.

    Presentation/gameplay layer

        Terrain/Render.cpp, Env.cpp, Avatar.cpp, Player.cpp, Input.cpp, Sdl.cpp.

This is effectively a procedural streaming world engine where persistent data is compact (rules + seed), and heavy geometry/material data is generated and evicted dynamically.
2) Where it optimizes

Key optimization points are explicit and practical:

    Time-sliced update loops

        Main.cpp: each frame targets ~15ms and passes a stop timestamp to systems.

        SceneUpdate(stop) and CacheUpdate(stop) do bounded work until time budget is consumed.

    Lazy generation + eviction

        CachePointAvailable() (Cache.cpp) creates CPage only on demand.

        CacheUpdate() periodically garbage-collects expired pages (PAGE_EXPIRE) and saves/deletes them.

    Staged page build pipeline

        CPage uses build stages (PAGE_STAGE_*) for position → normals → surface → color → trees → save, allowing incremental progression instead of one long stall.

    Distance-priority streaming

        CGrid.cpp precomputes sorted radial offsets (distance_list) and updates nearest chunks first.

    Rule-based compression of world state

        World stores region rules and noise, not full-resolution world geometry for all cells at once.

    Work fairness between systems

        Scene.cpp rotates update priority across terrain/grass/forest/brush each tick to avoid starvation.

3) Novel ideas

Most notable design ideas (for its era/style):

    “Region-as-rule” world model
    World stores per-region parameters (climate, bias, detail, colors, flags), then computes terrain values procedurally via interpolation and modifiers. This separates authoring intent from realized geometry elegantly.

    Cross-region dithering/blending

        World.cpp uses a dithermap and blend distances to avoid hard seams and reduce visible tiling at region boundaries.

    Two-tier geography generation

        Terraform creates macro geography (coastline, climate, rivers, biomes), while CPage realizes micro data locally (surfaces, normals, trees) only where observed.

    Deterministic procedural reproducibility

        Seeded noise + stored region map enables stable replayability and cheap save semantics.

4) Load-bearing structures (if changed, many systems break)

These are the architectural “beams”:

    struct World + region grid contract (Terrain/World.h)

        Feeds climate, water, tree, color, elevation logic across cache/render/gameplay.

        Any schema change ripples through Terraform, CPage, Scene, and save/load.

    CPage cell layout and stage machine (Terrain/Cpage.h, Terrain/CPage.cpp)

        Core format for all realized terrain data.

        Used by cache queries (CacheElevation, CacheSurface, normals, trees) and geometry generation.

    Cache page table indexing + lifecycle (Terrain/Cache.cpp)

        World-space to page-space mapping (PAGE_GRID, PAGE_SIZE) is fundamental to streaming correctness and memory behavior.

    GridManager update semantics (Terrain/CGrid.cpp)

        Coordinates all rolling render grids; dictates responsiveness and locality for terrain/foliage systems.

    Main loop ordering and budget semantics (Terrain/Main.cpp)

        System order (input/game/avatar/player/env/scene/cache/render) and time slicing preserve consistency and frame pacing.

Quick quality read

    The architecture is cohesive and purpose-built for large procedural terrain on constrained hardware.

    It favors predictable CPU budgeting and deterministic generation over modern ECS abstractions.

    Biggest risk area is tight coupling through globals/statics, but the core world-streaming pattern is solid and clear.
