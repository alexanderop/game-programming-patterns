# Observer

Source: https://gameprogrammingpatterns.com/observer.html

A subject announces that something happened without knowing who listens. Listeners register themselves and react.

## Use when

- Two unrelated systems need a thin link, such as physics reporting a fall and achievements reacting to it.
- Listeners come and go at runtime and the sender must not change.
- The notification is rare compared to the work around it. Walking a list is cheap, but not free.

## Skip when

- Both sides must be understood together. A direct call is easier to trace than a runtime listener list.
- The communication stays inside one cohesive feature.
- The code sits in a hot loop, or in a heavily threaded engine where a queued event is safer.

## Sketch

```ts
type Listener<E> = (event: E) => void

class Signal<E> {
  readonly #listeners = new Set<Listener<E>>()

  subscribe(fn: Listener<E>): () => void {
    this.#listeners.add(fn)
    return () => this.#listeners.delete(fn)
  }

  emit(event: E): void {
    for (const fn of [...this.#listeners]) fn(event)
  }
}

type FallEvent = { readonly entityId: number; readonly height: number }

class Physics {
  readonly onFall = new Signal<FallEvent>()
  // ...calls this.onFall.emit({ entityId, height }) during simulation
}

const stop = physics.onFall.subscribe((e) => achievements.unlock('FELL_OFF_BRIDGE', e.entityId))
// later, when the owner is destroyed: stop()
```

Subscribe returns an unsubscribe function, so cleanup is one call. Emit iterates a copy so a listener can unsubscribe during delivery without corrupting the loop.

## Pitfalls

- The lapsed listener problem. A subject's reference keeps an abandoned listener alive, so a closed UI screen keeps reacting. Garbage collection does not fix this. Always unsubscribe when the owner dies.
- Delivery is synchronous. A slow listener blocks the subject, so push heavy work to a queue.
- Listeners must not depend on each other's order.
- The call graph only exists at runtime, so debugging and code navigation get harder.
- Changing the listener list during delivery needs a deliberate policy, such as the copy above.

## Decisions

- Interface or function: plain functions are lighter and more flexible than a listener class.
- Pass the subject: include a sender in the payload when one listener watches several subjects.
- Subject or event object: a separate signal per event type is usually cleaner than inheriting from a base subject.
- Notify all or stop early: letting a listener halt delivery turns this into a chain of responsibility.

## Seen in

DOM and Node event listeners, C# events, Java's observer types, MVC model-to-view updates, and data binding in UI frameworks.
