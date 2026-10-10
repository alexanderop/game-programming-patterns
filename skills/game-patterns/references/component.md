# Component

Source: https://gameprogrammingpatterns.com/component.html

Split an entity into one component per domain (input, physics, rendering, audio). The entity becomes a thin container that holds them.

## Use when

- One class touches several domains that should stay decoupled, such as input, physics, and drawing.
- A class has grown too large to edit safely.
- You need to mix capabilities more freely than inheritance allows. A decoration has graphics, a trigger zone has physics, a crate has both.

## Skip when

- You have a handful of simple entities. Wiring components adds cost for a problem you do not have yet.
- The inner loop is hot and the extra indirection shows up in a profiler. Consider a data-oriented layout instead.

## Sketch

```ts
interface Component {
  update(self: Entity, dt: number): void
}

class Entity {
  x = 0
  y = 0
  readonly #components: readonly Component[]

  constructor(components: readonly Component[]) {
    this.#components = components
  }

  update(dt: number): void {
    for (const c of this.#components) c.update(this, dt)
  }
}

class GravityComponent implements Component {
  update(self: Entity, dt: number): void {
    self.y += 9.8 * dt
  }
}

class SpriteComponent implements Component {
  readonly #sprite: string
  constructor(sprite: string) {
    this.#sprite = sprite
  }
  update(self: Entity): void {
    drawSprite(this.#sprite, self.x, self.y)
  }
}

const crate = new Entity([new GravityComponent(), new SpriteComponent('crate')])
```

Position lives on the entity, so components share it without knowing each other. Components are passed in from outside, which keeps the entity free of concrete component types.

## Pitfalls

- Update order starts to matter once components share state on the container. Bugs appear when someone reorders them.
- Cross-component communication is the main cost. Pick a mechanism deliberately instead of letting each pair invent its own.
- Message passing through the container can create feedback loops.
- Entities become clusters that must be created, wired, and cleaned up together.

## Decisions

- Who assembles components:
  - The entity builds its own. It is always valid, but hard to reconfigure.
  - Outside code supplies them. It is flexible, but callers can wire it wrong.
- How components talk:
  - Shared container state keeps them decoupled but bloats the container and makes order critical.
  - Direct references are simple and fast but couple the pair.
  - Messages relayed by the container keep siblings apart and the container simple, with the loop risk above.
- A common mix is shared state for universal data like position, direct references for tight pairs like animation and rendering, and messages for fire-and-forget events like a collision sound.

## Seen in

Unity's GameObject, entity component systems in many engines, XNA's GameComponent.
