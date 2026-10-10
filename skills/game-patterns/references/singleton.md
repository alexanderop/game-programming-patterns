# Singleton

Source: https://gameprogrammingpatterns.com/singleton.html

Guarantee one instance of a class and give it a global access point. The book argues this usually does more harm than good, and recommends the alternatives below first.

## Use when

- Exactly one instance must exist and you can enforce that with a runtime check, such as an assertion on a flag in the constructor.
- A few well-known services need global reach, and you would rather configure them through a service locator than hard-code `Thing.instance`.

## Skip when

Skip it by default. Pick one of these instead.

- The class only operates on other objects. Move the behavior into those objects, for example `bullet.move()` instead of `BulletManager.move(bullet)`.
- You only need one instance. Enforce that without global access.
- A dependency is core to a function. Pass it in as a parameter or a context object.
- Many subclasses need the same service. Expose it through a protected accessor on their base class.
- A top-level object already exists. Let `Game` own the systems and hand them out.
- No polymorphism is needed. A module of plain functions is simpler than `instance()`.

## Sketch

```ts
class Logger {
  static #created = false
  readonly #sink: (line: string) => void

  constructor(sink: (line: string) => void) {
    if (Logger.#created) throw new Error('Logger already created')
    Logger.#created = true
    this.#sink = sink
  }

  write(line: string): void {
    this.#sink(line)
  }
}

class Game {
  readonly log = new Logger(console.log)
  readonly physics = new Physics(this.log)
}
```

Single instantiation is enforced at runtime, but there is no global handle. Code receives the logger through the owner or a constructor argument, so tests can pass a fake and a second instance stays possible later.

## Pitfalls

- A singleton is a global variable in a class. It brings the same hidden coupling and shared mutable state.
- Easy access invites unrelated modules to call each other, such as physics calling audio. Hard access is a useful signal that they should not.
- Call sites like `Log.instance().write()` lock in the one-instance design. Moving to several instances later touches every caller.
- Lazy initialization runs expensive setup at an unpredictable moment, which causes frame hitches. Initialize at a known point such as startup.
- Module-level exports in TypeScript are singletons too, and carry the same costs.

## Decisions

- Cross-cutting concerns like logging clutter every signature if passed explicitly. A locator or an owned object is the reasonable compromise.
- Accept a few globals, but keep the count small.

## Seen in

Logging, audio, and asset systems, which are the usual cases where a global handle is still defended.
