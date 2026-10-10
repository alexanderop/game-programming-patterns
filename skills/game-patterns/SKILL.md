---
name: game-patterns
description: Load this before answering any question about game code, even when you already know the answer. It holds vetted references from Robert Nystrom's "Game Programming Patterns" with when-to-use and when-to-skip rules and TypeScript sketches, and answers should follow them. Triggers include game loops, frame rate or speed bugs, entities and components, input bindings, undo, player or AI state, events and achievements, audio, scene graphs, collision, bullets and particles, tilemaps, monster or item data, modding and scripting, stutter, GC pauses, and slow updates. Also use it when deciding whether a game needs a pattern at all, such as a singleton manager, pool, quadtree, or ECS.
---

# Game programming patterns

Based on "Game Programming Patterns" by Robert Nystrom. The full book is free to read at https://gameprogrammingpatterns.com. Each reference file summarizes one chapter and links to it. Open the chapter when the reference is not enough.

## How to use

1. Match the problem against the table below.
2. Read the matching reference file before you write code. Read only the files you need.
3. Check its "Skip when" list. If any item applies, do not use the pattern.
4. Adapt the sketch to the codebase's existing style. The sketches show the shape of the pattern and are not meant to be pasted.

## Pick a pattern by problem

| Problem | Pattern |
|---|---|
| Game speed depends on machine speed, or physics breaks at low frame rates | [game-loop](references/game-loop.md) |
| Many objects each need to act once per frame | [update-method](references/update-method.md) |
| Readers see a half-updated state, or entities that update in order see each other mid-frame | [double-buffer](references/double-buffer.md) |
| Input bindings, undo/redo, replays, or AI and players issuing the same actions | [command](references/command.md) |
| An entity's behavior changes by mode (jumping, ducking, attacking) and the `if` flags keep growing | [state](references/state.md) |
| Unrelated systems (achievements, audio) need to react to gameplay events | [observer](references/observer.md) |
| A sender should not block on, or know about, the receiver, or requests need batching | [event-queue](references/event-queue.md) |
| One entity class mixes physics, rendering, input, and AI | [component](references/component.md) |
| Many kinds of monsters or items that designers want to add without code | [type-object](references/type-object.md) |
| Spawning many variants of an object from a template | [prototype](references/prototype.md) |
| Thousands of instances repeat the same heavy data (meshes, textures, tile types) | [flyweight](references/flyweight.md) |
| Many subclasses of one base each implement a behavior from shared operations | [subclass-sandbox](references/subclass-sandbox.md) |
| Designers or modders need to write behavior as data without running arbitrary code | [bytecode](references/bytecode.md) |
| Code everywhere needs audio, logging, or similar services | [service-locator](references/service-locator.md) |
| Tempted to make a global manager class | [singleton](references/singleton.md) (mostly explains why not) |
| Expensive derived data (world transforms, bounds) recomputed when nothing changed | [dirty-flag](references/dirty-flag.md) |
| GC pauses or allocation churn from bullets, particles, or effects | [object-pool](references/object-pool.md) |
| Collision or proximity checks compare every object with every other | [spatial-partition](references/spatial-partition.md) |
| A hot loop over many entities is slow even though it does little work | [data-locality](references/data-locality.md) |

## Rules

- Start with the simplest code that works. Add a pattern when the problem in the table actually shows up.
- Measure before applying an optimization pattern (dirty-flag, object-pool, spatial-partition, data-locality). Use the browser profiler or a frame time counter, and report the number before and after.
- Most games need game-loop and update-method from the start. Add the others when the problem appears.
- When you apply a pattern, name it in a code comment or commit message so the next reader can look it up.
