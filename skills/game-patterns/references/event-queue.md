# Event Queue

Source: https://gameprogrammingpatterns.com/event-queue.html

Store messages in a buffer so the sender and the processor do not have to act at the same moment. The sender enqueues and moves on, and the receiver drains the queue on its own schedule.

## Use when

- You need to decouple in time, not just in identity. If you only need to decouple sender from receiver, Observer or Command is simpler.
- The receiver wants to delay, batch, merge, or drop requests. Example: an audio system that plays at most a few sounds per frame.
- Work arrives from a place where blocking is not acceptable, such as the middle of a physics step.

## Skip when

- The sender needs an answer right now. A queue gives up that control.
- A direct call or observer would do. A queue hides control flow and is harder to debug.

## Sketch

```ts
class RingBuffer<T> {
  readonly #items: (T | undefined)[]
  #head = 0 // oldest item, read here
  #tail = 0 // next free slot, write here

  constructor(readonly capacity: number) {
    this.#items = new Array<T | undefined>(capacity + 1).fill(undefined)
  }

  get isEmpty(): boolean {
    return this.#head === this.#tail
  }

  enqueue(item: T): boolean {
    const next = (this.#tail + 1) % this.#items.length
    if (next === this.#head) return false // full, caller decides what to drop
    this.#items[this.#tail] = item
    this.#tail = next
    return true
  }

  dequeue(): T | undefined {
    if (this.isEmpty) return undefined
    const item = this.#items[this.#head]
    this.#items[this.#head] = undefined
    this.#head = (this.#head + 1) % this.#items.length
    return item
  }
}

type AudioRequest = { readonly kind: 'play'; readonly soundId: string; readonly volume: number }
const pending = new RingBuffer<AudioRequest>(32)
```

The array holds one spare slot so that full and empty stay distinguishable (head equals tail only means empty). Both indices wrap with modulo, so nothing shifts and nothing allocates after construction.

## Pitfalls

- A single central queue acts as a global. It carries the same coupling risks as global state.
- The world changes between enqueue and processing. Put the data the handler needs in the message, and check at handling time that the target entity is still alive.
- Feedback loops hide behind the delay. A synchronous version would overflow the stack, but a queue just keeps growing. Drain only the events present when the drain starts, so events enqueued by handlers wait for the next frame, and log cycles in debug builds.
- A fixed ring can fill. Decide whether to drop the new item, drop the oldest, or grow.

## Decisions

- What goes in: events (something happened, many listeners) or requests (work wanted, one processor).
- Who reads: a single consumer is simple. Broadcast reaches all listeners but drops events if none are registered. A work queue gives each item to one of several workers and needs a scheduling rule.
- Who writes: one writer is like Observer. Many writers need a sender field in the message and make cycles easier.
- Lifetime: with a garbage collector, entries live as long as referenced. Reusing slots, as above, avoids allocation churn.
- Merging duplicates: scan pending requests at enqueue and combine, for example two identical sounds into one at the higher volume. A map keyed by request type avoids the linear scan.

## Seen in

Audio request queues, input event buffers, deferred gameplay events processed at a fixed point in the frame, GUI message loops.
