# Subclass Sandbox

Source: https://gameprogrammingpatterns.com/subclass-sandbox.html

A base class provides protected operations and one abstract method. Each subclass implements that method using only the provided operations.

## Use when

- One base class has many derived classes, such as hundreds of superpowers, enemy behaviors, or spells.
- The base class can offer every operation those subclasses need.
- The subclasses share behavior you want to reuse.
- You want subclasses coupled to the base class and not to audio, particles, physics, and other engine systems.

## Skip when

- There are only a few subclasses. The shared layer costs more than it saves.
- The behavior is simple enough to describe as data. Complex imperative logic is the case that resists data, so a sandbox method fits there.
- The base class would have to expose nearly the whole engine. Split it up first.

## Sketch

```ts
interface Services {
  move(entity: number, x: number, y: number): void
  playSound(id: string): void
  spawnParticles(kind: string, x: number, y: number): void
}

abstract class Superpower {
  constructor(readonly services: Services) {}

  abstract activate(user: number): void

  protected move(user: number, x: number, y: number): void {
    this.services.move(user, x, y)
  }

  protected playSound(id: string): void {
    this.services.playSound(id)
  }

  protected spawnParticles(kind: string, x: number, y: number): void {
    this.services.spawnParticles(kind, x, y)
  }
}

class SkyLaunch extends Superpower {
  activate(user: number): void {
    this.playSound('whoosh')
    this.spawnParticles('dust', 0, 0)
    this.move(user, 0, 100)
  }
}
```

`SkyLaunch` never imports an audio or particle module. Engine changes land in `Superpower`, and the dozens of subclasses stay untouched. Rules like queuing sounds can also be enforced in one place.

## Pitfalls

- The base class grows. It couples to every system any subclass touches, and becomes the brittle base class problem.
- Calling that base class a place to dump helpers produces a god object. Move groups of operations into helper objects, as in the Component pattern.
- State-changing operations deserve more thought than read-only ones. Check thread safety and determinism, especially in networked games.
- Thin forwarding wrappers add little unless they hide private state.
- Hierarchies stay shallow and wide. Deep inheritance trees undo the benefit.

## Decisions

- Which operations to provide:
  - More operations lower subclass coupling and raise base class coupling.
  - Skip operations only one or two subclasses use. Centralize ones that change state.
- Direct methods or contained objects:
  - Direct methods are simple but fatten the base class.
  - A getter returning a helper such as a sound player keeps the base class small and easier to maintain.
- How the base class gets its state:
  - Constructor argument. This is safe, but every subclass must forward it.
  - Two-stage init with an `init` method. Hide it behind a factory so nobody forgets to call it.
  - Static state. This saves memory and has singleton-style problems.
  - Service locator. The base class fetches what it needs and callers have no setup to do.

## Seen in

Update methods that double as sandbox methods, engine facades that hide the engine from gameplay code, and the reverse of Template Method, which puts the algorithm in the base class and primitives in subclasses.
