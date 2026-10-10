# State

Source: https://gameprogrammingpatterns.com/state.html

Model an entity as a finite state machine. It is in exactly one state, inputs trigger transitions, and behavior depends on the current state.

## Use when

- Behavior depends on an internal mode, and the modes form a small discrete set. Examples are player movement, AI behavior, menus, and network handshakes.
- You see boolean flags like `isJumping` and `isDucking` combining into invalid states.
- The entity reacts to a stream of inputs over time.

## Skip when

- Behavior is not tied to discrete modes. Complex AI outgrows a plain state machine, so consider behavior trees or planning.
- Two or three branches cover it. A single enum and a `switch` is the simplest form and makes invalid states unrepresentable.

## Sketch

```ts
type Input = 'press-jump' | 'press-duck' | 'release-duck' | 'land'

type PlayerState =
  | { readonly kind: 'standing' }
  | { readonly kind: 'jumping' }
  | { readonly kind: 'ducking'; readonly chargeMs: number }

function transition(state: PlayerState, input: Input): PlayerState {
  switch (state.kind) {
    case 'standing':
      if (input === 'press-jump') return { kind: 'jumping' }
      if (input === 'press-duck') return { kind: 'ducking', chargeMs: 0 }
      return state
    case 'jumping':
      return input === 'land' ? { kind: 'standing' } : state
    case 'ducking':
      return input === 'release-duck' ? { kind: 'standing' } : state
  }
}

class Player {
  #state: PlayerState = { kind: 'standing' }

  handle(input: Input): void {
    const next = transition(this.#state, input)
    if (next !== this.#state) this.#enter(next)
  }

  #enter(next: PlayerState): void {
    this.#state = next
    setSprite(next.kind) // entry action lives in one place
  }
}
```

A discriminated union lets a state carry its own data, such as the charge timer, which a bare enum cannot. Routing every change through `#enter` gives entry actions one home instead of copies in each transition.

## Pitfalls

- Per-entity data in a state means a new object per transition. Stateless states can be shared (see flyweight).
- Cramming independent concerns into one machine causes a state explosion. Doing and carrying would need n times m states.
- Plain machines forget history, so returning to "whatever I was doing" needs extra structure.

## Decisions

- Enum, union, or classes: an enum for labels, a union when states carry data, classes for heavy behavior.
- Concurrent machines: run one machine per independent aspect, such as movement and weapon, for n plus m states. Interactions between them usually need ad hoc checks.
- Hierarchical machines: let a substate pass unhandled inputs to a superstate, so jumping and ducking share "on ground" behavior.
- Pushdown automaton: keep a stack of states. Push a temporary state like firing, then pop to resume whatever was underneath.
- Entry and exit actions: run setup and teardown in the state being entered or left, not at the transition site.

## Seen in

Character controllers, game AI, input handling, menu navigation, text parsing, and network protocols.
