# Flyweight

Source: https://gameprogrammingpatterns.com/flyweight.html

Share the data that many similar objects have in common, and keep only the per-instance data in each object. It is purely a memory and transfer optimization.

## Use when

- You have thousands of objects that share most of their data, such as trees, tiles, or particles.
- Memory or per-object GPU upload is the bottleneck. Instanced rendering is the hardware form of this idea.
- You are tempted by a large enum plus `switch` for tile or unit kinds. A shared object keeps data and behavior together.
- State objects in a state machine hold no per-entity fields, so one instance can serve every machine.

## Skip when

- The shared part would need to differ per instance. A change to a shared object shows up everywhere at once.
- You have not measured a problem. The extra indirection can hurt cache behavior in tight loops.

## Sketch

```ts
class Terrain {
  readonly movementCost: number
  readonly isWater: boolean
  readonly texture: string

  constructor(movementCost: number, isWater: boolean, texture: string) {
    this.movementCost = movementCost
    this.isWater = isWater
    this.texture = texture
  }
}

const GRASS = new Terrain(1, false, 'grass.png')
const HILL = new Terrain(3, false, 'hill.png')
const RIVER = new Terrain(2, true, 'river.png')

class World {
  readonly #width: number
  readonly #tiles: readonly Terrain[]

  constructor(width: number, tiles: readonly Terrain[]) {
    this.#width = width
    this.#tiles = tiles
  }

  tileAt(x: number, y: number): Terrain {
    return this.#tiles[y * this.#width + x]!
  }
}
```

Each cell stores a reference, not a copy of the terrain data. Callers ask `world.tileAt(x, y).movementCost`, so the world needs no knowledge of terrain rules.

## Pitfalls

- Mutable flyweights cause bugs far from the cause. Mark every field `readonly`.
- Position and other per-instance data must stay outside the shared object. Mixing them in defeats the sharing.
- Unlike Type Object, flyweight is about efficiency, not about reducing the number of classes. Do not reach for it to model kinds.

## Decisions

- Creation:
  - Build all flyweights up front when the set is known. This keeps ownership simple.
  - Build them on demand behind a factory that looks up an existing identical instance first.
- Storage: hide the grid or array behind a small type, so you can change layout later for cache reasons.

## Seen in

Instanced forest rendering, tile maps with shared terrain objects, and stateless state objects reused across state machines.
