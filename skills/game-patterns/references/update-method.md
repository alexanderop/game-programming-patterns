# Update Method

Source: https://gameprogrammingpatterns.com/update-method.html

Give each object an `update` method that simulates one frame of its behavior, and let the game loop call it on every object once per frame.

## Use when

- The game has many live entities such as enemies, projectiles, and NPCs that must all advance together.
- Each entity's behavior is mostly independent of the others.
- Entities change over time and keep running behavior across frames.

## Skip when

- The game is abstract and turn-based, like chess. Animation may still use it.
- Behavior reads best as straight-line code and has no per-frame life.

## Sketch

```ts
interface Entity {
  update(dt: number): void
  readonly dead: boolean
}

class Patroller implements Entity {
  dead = false
  #x = 0
  #direction: 1 | -1 = 1

  readonly #left: number
  readonly #right: number

  constructor(left: number, right: number) {
    this.#left = left
    this.#right = right
  }

  update(dt: number): void {
    this.#x += this.#direction * 40 * dt
    if (this.#x >= this.#right) this.#direction = -1
    if (this.#x <= this.#left) this.#direction = 1
  }
}

class World {
  readonly #entities: Entity[] = []

  add(entity: Entity): void {
    this.#entities.push(entity)
  }

  update(dt: number): void {
    const count = this.#entities.length
    for (let i = 0; i < count; i++) this.#entities[i]!.update(dt)
    for (let i = this.#entities.length - 1; i >= 0; i--) {
      if (this.#entities[i]!.dead) this.#entities.splice(i, 1)
    }
  }
}
```

The loop caches `count` so entities spawned during this frame start acting next frame, and removal happens in a separate backward pass so no entity gets skipped. The patrol direction lives in a field because the entity has to remember where it was heading between frames.

## Pitfalls

- Entities do not run at the same instant. Later ones see earlier ones' new state, so order affects results. Use Double Buffer if that matters.
- Adding to or removing from the list during iteration skips or double-updates entities. Cache the length, or mark dead and purge afterward.
- Splitting behavior into one-frame slices is harder than writing it as a sequence. Keep state explicit, or use a state machine or generators for long sequences.
- With a variable timestep, scaled movement can overshoot a boundary. Handle the overshoot.
- Inactive entities in the same list waste time and cache space.

## Decisions

- Where `update` lives:
  - On the entity class. This is simplest, but subclass trees get brittle as behaviors multiply.
  - On components. Each part of an entity updates on its own.
  - On a delegate such as a swappable state object. Behavior changes without new subclasses.
- Dormant objects:
  - Keep everything in one list. This is simple, but you pay for idle objects.
  - Keep a separate active list. It saves time, but you must keep both lists in sync.
- Time:
  - Fixed step keeps `update` simple.
  - Variable step passes `dt` and requires care around overshoot.

## Seen in

Unity's `MonoBehaviour.Update`, XNA's `Game` and `GameComponent`, and most hand-rolled entity systems, usually paired with a game loop and components.
