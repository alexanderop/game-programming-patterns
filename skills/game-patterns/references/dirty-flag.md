# Dirty Flag

Source: https://gameprogrammingpatterns.com/dirty-flag.html

Mark derived data stale when its inputs change. Recompute it only when someone reads it.

## Use when

- Inputs change more often than the derived result is read. Example: a node moves several times per frame but renders once.
- The result is expensive to recompute and hard to update step by step. If a running total is easy to keep, keep the running total instead.
- A profiler shows the recomputation costs real time. This is an optimization pattern.

## Skip when

- Every write is followed by a read. The flag check then adds cost and saves nothing.
- The value is cheap. A branch per read can cost more than a small calculation.

## Sketch

```ts
class SceneNode {
  readonly children: SceneNode[] = []
  #local = Mat4.identity()
  #world = Mat4.identity()
  #dirty = true

  setLocal(m: Mat4): void {
    this.#local = m
    this.#dirty = true
  }

  render(parentWorld: Mat4, parentDirty = false): void {
    const dirty = this.#dirty || parentDirty
    if (dirty) {
      this.#world = parentWorld.multiply(this.#local)
      this.#dirty = false
    }
    draw(this.#world)
    for (const child of this.children) child.render(this.#world, dirty)
  }
}
```

`parentDirty` passes staleness down during the traversal, so moving a parent never has to visit and flag every descendant. This works only because `render` is the single reader of `#world`. Add a second reader and it must walk the parent chain itself.

## Pitfalls

- A missed write leaves stale data that is hard to trace. Keep the input private and route every write through one setter that sets the flag.
- Deferred work can land all at once and cause a frame spike when the result is finally read.
- Deferred saves or network syncs lose data if the process dies before the flush.
- The cache stays in memory. The pattern trades memory for CPU time.

## Decisions

- When to clean the flag:
  - On read. This does no work if nobody reads, but a slow recompute stalls the reader.
  - At a checkpoint such as a loading screen or level transition. The player doesn't see the pause, but the work waits until the player reaches it.
  - On a background timer. You can tune how often it runs, but it needs safe concurrent access and may redo work.
- Granularity: one flag per small unit (per tile) recomputes only what changed. One flag per large unit (per chunk) uses less memory and less per-flag overhead but redoes unchanged data.

## Seen in

Scene graph transforms, physics engines that skip resting bodies until an impulse wakes them, UI frameworks that re-render only components with changed state.
