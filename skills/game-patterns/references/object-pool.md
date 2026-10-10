# Object Pool

Source: https://gameprogrammingpatterns.com/object-pool.html

Preallocate a fixed set of objects and reuse them instead of creating and destroying them. In a garbage-collected language, the goal is to avoid allocation churn and GC pauses.

## Use when

- Objects spawn and die constantly and are about the same size. Example: bullets, particles, sound instances.
- A profiler shows GC pauses or allocation cost causing dropped frames.
- Each object wraps something expensive to build that is worth reusing.

## Skip when

- Objects are created rarely. A pool only holds memory and adds reset bugs.
- You have no frame-time problem. Modern engines handle short-lived objects well, so measure first.
- Objects vary widely in size or shape.

## Sketch

```ts
class Bullet {
  x = 0
  y = 0
  vx = 0
  vy = 0
  life = 0
  next: Bullet | null = null // free list link, only meaningful while unused
}

class BulletPool {
  readonly #items: readonly Bullet[]
  #free: Bullet | null = null

  constructor(size: number) {
    this.#items = Array.from({ length: size }, () => new Bullet())
    for (const b of this.#items) this.release(b)
  }

  acquire(x: number, y: number, vx: number, vy: number): Bullet | null {
    const b = this.#free
    if (b === null) return null
    this.#free = b.next
    b.next = null
    b.x = x; b.y = y; b.vx = vx; b.vy = vy; b.life = 2
    return b
  }

  release(b: Bullet): void {
    b.next = this.#free
    this.#free = b
  }

  update(dt: number): void {
    for (const b of this.#items) {
      if (b.life <= 0) continue
      b.life -= dt
      b.x += b.vx * dt
      b.y += b.vy * dt
      if (b.life <= 0) this.release(b)
    }
  }
}
```

The free list threads unused objects together, so `acquire` and `release` are O(1) with no scan. `life <= 0` doubles as the in-use flag, which saves a separate field.

## Pitfalls

- Reused objects keep old state. Reset every field in `acquire`, or a bullet inherits the last one's damage.
- Pooled objects that hold references to other objects keep them alive. Clear those references on release.
- Releasing the same object twice corrupts the free list and hands one object to two owners.
- A pool sized too small silently drops spawns. A pool sized too large wastes memory.
- Code that keeps a reference after release sees the object change under it. A homing missile then chases whatever reused its target. Give each slot a generation counter that `release` increments, store `{ index, generation }` handles, and treat a mismatch as dead.

## Decisions

- When the pool is full. Fail the spawn, evict the oldest or least important object, or grow the pool. Dropping a particle is usually fine, dropping an enemy is not.
- Who tracks in-use state. The object can carry it (simple, may reuse an existing field) or the pool can track it separately (works for any type).
- Who initializes. The pool can take constructor-like arguments, or return a blank object and let the caller fill it, which forces the caller to handle the empty case.

## Seen in

Bullet and particle systems, audio voice pools, enemy spawners, network buffer pools.
