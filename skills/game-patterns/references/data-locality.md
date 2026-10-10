# Data Locality

Source: https://gameprogrammingpatterns.com/data-locality.html

Arrange data in memory in the order the hot loop reads it, so the CPU cache loads useful bytes instead of stalling on misses.

## Use when

- A profiler shows a per-frame loop over thousands of similar items (particles, bullets, boids) dominates frame time.
- The loop touches only a few fields of each item. Example: position and velocity, not name or loot table.
- You can store the items in flat arrays and drop per-item object identity in the hot path.

## Skip when

- The code runs rarely or the item count is small. The result is less flexible and harder to read for no visible gain.
- You have no measurement. Cache effects depend on hardware, so guessing is unreliable.
- Items are heterogeneous and the loop needs polymorphic behavior per item.

## Sketch

```ts
class Particles {
  readonly #x: Float32Array
  readonly #y: Float32Array
  readonly #vx: Float32Array
  readonly #vy: Float32Array
  #count = 0

  constructor(readonly capacity: number) {
    this.#x = new Float32Array(capacity)
    this.#y = new Float32Array(capacity)
    this.#vx = new Float32Array(capacity)
    this.#vy = new Float32Array(capacity)
  }

  spawn(x: number, y: number, vx: number, vy: number): boolean {
    if (this.#count === this.capacity) return false
    const i = this.#count++
    this.#x[i] = x; this.#y[i] = y; this.#vx[i] = vx; this.#vy[i] = vy
    return true
  }

  update(dt: number): void {
    for (let i = 0; i < this.#count; i++) {
      this.#x[i] += this.#vx[i] * dt
      this.#y[i] += this.#vy[i] * dt
    }
  }

  kill(i: number): void {
    const last = --this.#count
    this.#x[i] = this.#x[last]; this.#y[i] = this.#y[last]
    this.#vx[i] = this.#vx[last]; this.#vy[i] = this.#vy[last]
  }
}
```

This is struct-of-arrays, with live items packed at the front. `kill` swaps the last item into the gap, so the loop never skips dead entries. Callers that iterate and kill must revisit index `i` after a swap.

## Pitfalls

- JS gives less control than C++. Plain object arrays may already be laid out well by the engine, and typed arrays add bounds checks. Benchmark both before committing.
- Swap-remove changes indices. Anything holding an index to a moved item now points at the wrong one, so use handles if outside code refers to items.
- Float32Array rounds values. Do not store anything that needs double precision, such as large world coordinates or timestamps.
- Splitting into many arrays makes code noisier. Keep the layout behind one class so callers never see it.

## Decisions

- Layout. Struct-of-arrays suits loops that touch a few fields. Array-of-objects suits code that reads most fields of one item at a time.
- Hot and cold split. Keep per-frame fields in the packed arrays and rarely used data (names, drop tables) in a parallel array or map, so cache lines carry only useful bytes.
- Dead items. Swap-remove keeps the array packed but reorders items. A flag with a skip branch keeps order but wastes iterations.
- Polymorphism. Prefer one array per type, or a type-id field, over a mixed list of subclass instances in hot code.

## Seen in

Particle systems, entity component systems with per-component arrays, physics and boids simulations, WebGL and WebGPU vertex buffers.
