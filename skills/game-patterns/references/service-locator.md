# Service Locator

Source: https://gameprogrammingpatterns.com/service-locator.html

Give code global access to a service through an abstract interface. A locator holds the current provider, so callers know neither the concrete class nor how it was chosen.

## Use when

- A system is cross-cutting and effectively singular, such as audio, logging, or asset loading, and passing it through every signature would clutter the code.
- You want to swap the implementation at runtime or per build. Example: real audio in release, silent audio in tests.

## Skip when

- You can pass the object in. Explicit parameters make dependencies visible and are usually clearer.
- Only one or two callers need it. The locator hides dependencies that a constructor argument would show.

## Sketch

```ts
interface Audio {
  play(soundId: string): void
}

class NullAudio implements Audio {
  play(): void {}
}

class LoggedAudio implements Audio {
  readonly #inner: Audio
  constructor(inner: Audio) {
    this.#inner = inner
  }
  play(soundId: string): void {
    console.log(`audio: play ${soundId}`)
    this.#inner.play(soundId)
  }
}

class Locator {
  static readonly #fallback: Audio = new NullAudio()
  static #audio: Audio = Locator.#fallback

  static get audio(): Audio {
    return Locator.#audio
  }

  static provide(audio: Audio | undefined): void {
    Locator.#audio = audio ?? Locator.#fallback
  }
}

Locator.provide(new LoggedAudio(new WebAudio()))
Locator.audio.play('jump')
```

The null service means callers never check for a missing provider, and registering the fallback up front keeps a branch out of the accessor. The logging decorator wraps any provider without touching callers.

## Pitfalls

- Dependencies become invisible. A reader cannot see what a class needs without reading its body.
- The service may be unregistered when first requested. Decide the missing case on purpose.
- Anyone can call the service from anywhere, so it must behave correctly in every context, not only the one its author imagined.
- A silent null service can hide a wiring bug. Have it log when used in development builds.

## Decisions

- How the service is located:
  - External registration is fast and simple, allows runtime data like a server address, and allows swapping mid-game. Code must rely on someone registering first.
  - Compile-time binding costs nothing at runtime and guarantees presence, but needs a rebuild to change.
  - Runtime configuration through a config file lets non-programmers change it, and costs complexity and lookup time.
- When it is not found:
  - Return null and let callers handle it. Every call site must check, and one miss crashes.
  - Assert and halt. Callers stay simple and the bug is loud. Most shipped games choose this.
  - Return a null service. The game keeps running while features are unfinished, but unintended gaps are harder to spot.
- Scope: global for widely used services like logging, restricted to a class hierarchy for domain-specific ones, which limits coupling.

## Seen in

Audio and logging services in engines, test setups that swap in fakes, headless builds that stub out rendering and sound.
